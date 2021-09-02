---
layout: blog
title: "TKEStack 集群容器运行时迁移"
date: 2021-09-01
# slug of this blog url
slug: container-runtime-migraion
---

**Author**: LeoRyu

_Kubernetes宣布在1.20版本之后将弃用Docker作为容器运行时，在2021年末发布的1.23版本中将彻底移除dockershim组件。可能很多朋友不是很清楚容器运行时迁移工作要怎么做，本文将就TKEStack的global集群如何进行容器运行时迁移做详细介绍。同时，本文非TKEStack globa集群的容器运行时迁移也有很好的参考价值。_

## 限制条件

1. 集群迁移前请确认master节点不止一个，单master节点切换容器运行时只能通过备份etcd数据再恢复集群方式，本文提供方法并不适用。
2. 迁移前倾确认当前K8s集群支持以containerd作为容器运行时，这里建议集群版本为1.19+。


## 迁移流程

### 准备工作

1. 每个节点上修改 docker daemon.json: `vim /etc/docker/daemon.json`，将 `"live-restore"` 修改为 `false`。
2. 每个节点重启 dcoker 服务： `systemctl daemon-reload && systemctl restart docker.service`。
3. 修改galaxy ds：`kubectl edit -n kube-system ds galaxy-daemonset`，在`volumeMounts`和`volumes`中添加以下内容：
```sh
...
        volumeMounts:
        - name: containerd-run
          mountPropagation: Bidirectional
          mountPath: /var/run/netns/
...
      volumes:
      - name: containerd-run
        hostPath:
          path: /var/run/netns
```

### 非master0节点迁移

非global集群首个master节点的迁移步骤。

1. 封锁节点：`kubectl cordon xxx`。
2. 驱逐节点：`kubectl drain xxx --ignore-daemonsets --delete-local-data`，注意该操作会删除 `pod` 的本地临时数据。
3. 停止Kubelet服务: `systemctl stop kubelet`。
4. 停止Docker服务：`systemctl disable docker.service && systemctl stop docker.service && systemctl stop containerd.service`。
5. 下载containerd，这里从 `https://github.com/containerd/nerdctl/releases` 下载：`curl -LO https://github.com/containerd/nerdctl/releases/download/v0.11.1/nerdctl-full-0.11.1-linux-amd64.tar.gz`。
6. 安装containerd：`tar Cxzvvf /usr/local nerdctl-full-0.11.1-linux-amd64.tar.gz`。
7. 添加containerd自定义配置：
```sh
cat /etc/containerd/config.toml
version = 2
root = "/var/lib/containerd"
state = "/run/containerd"

[grpc]
  address = "/run/containerd/containerd.sock"
  gid = 0
  max_recv_message_size = 16777216
  max_send_message_size = 16777216
  uid = 0

[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
    sandbox_image = "registry.tke.com/library/pause:3.2"
    [plugins."io.containerd.grpc.v1.cri".cni]
      bin_dir = "/opt/cni/bin"
      conf_dir = "/etc/cni/net.d"
    [plugins."io.containerd.grpc.v1.cri".containerd]
      
      default_runtime_name="runc"
      
    [plugins."io.containerd.grpc.v1.cri".registry]
      [plugins."io.containerd.grpc.v1.cri".registry.configs]
        
        [plugins."io.containerd.grpc.v1.cri".registry.configs."registry.tke.com".tls]
          insecure_skip_verify=true
        
        [plugins."io.containerd.grpc.v1.cri".registry.configs."default.registry.tke.com".tls]
          insecure_skip_verify=true
```
8. 修改 `kubelet` 参数：
```sh
cat /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
Environment="KUBELET_RUNTIME_ARGS=--container-runtime=remote --container-runtime-endpoint=unix:///run/containerd/containerd.sock"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/sysconfig/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS $KUBELET_RUNTIME_ARGS
```

9. 激活 `containerd` 服务：`systemctl daemon-reload && systemctl enable containerd.service && systemctl start containerd.service`。

10. 启动kubelet服务：`systemctl daemon-reload && systemctl start kubelet.service`。

11. 解除节点封锁：`kubectl uncordon xxx`。

### master0节点迁移

如果你的集群不是TKEStack的global集群请忽略以下内容。如果你确定你当前的TKEStack没有使用内置的tke-registry作为镜像仓库，也可以忽略一下内容。

不确定哪个是master0节点的话，可以通过`kubectl get pod -n tke -owide | grep tke-registry-api`查看pod所在节点，该节点就是mater0节点。

master0节点迁移放在其他节点迁移完成后在进行，相较非master0迁移多出了一下步骤：

1. 在步骤`1. 封锁节点`前备份master0节点镜像到其他master节点：
```sh
## 使用docker在master0节点上执行：
docker save $(docker images | sed '1d' | awk '{print $1 ":" $2}') -o backup.tar

## 使用nerdctl在其他master节点上执行：
nerdctl --namespace=k8s.io load -i backup.tar
```
2. 在步骤`9. 激活 `containerd` 服务`之后，步骤`10. 启动kubelet服务`之前加载master0节点镜像：
```sh
## 使用nerdctl在master0节点上执行：
nerdctl --namespace=k8s.io load -i backup.tar
```