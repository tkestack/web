---
title: "迁移步骤"
linkTitle: "迁移步骤"
date: 2020-07-14
description: >
  TKEStack 具体迁移步骤，注意事项
weight: 3
---

# 迁移步骤

## 迁移步骤

### 文档说明

   此文档用来指导用户如何将存量TKEStack集群从Docker手动迁移到Containerd。首先迁移worker节点，然后再迁移master节点，如果master、worker节点有多个，
请逐个节点进行迁移。

### 迁移步骤
#### 迁移Worker节点
1、首先cordon、drain worker节点，避免在迁移过程中pod调度到此节点
```
# kubectl cordon <node_name>
```
```
# kubectl drain <node_name> --ignore-daemonsets
``` 
2、停止Kubelet服务
```
# sudo systemctl stop kubelet
```
```
# sudo systemctl status kubelet
```
3、停止Docker服务
```
# sudo systemctl stop  docker
```
4、在节点上安装并且配置Containerd,
首先在Containerd官网上下载所需版本的安装包，如下以Containerd v1.5.2版本为例进行安装：
```
# tar -zvxf cri-containerd-cni-1.5.2-linux-amd64.tar.gz -C /
```
安装好后用户可以根据需求通过修改Containerd的配置文件对Containerd进行配置：
```
# mkdir -p /etc/containerd/
```
```
# vim /etc/containerd/config.toml
```
5、启动Containerd服务
```
# sudo systemctl enable containerd
```
```
# sudo systemctl start containerd
```
```
# sudo systemctl status containerd
```
6、修改Kubelet配置文件，使runtime为Containerd
```
# vim /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
```
在[Service]添加配置项：`KUBELET_RUNTIME_ARGS`并且添加到`ExecStart`配置中
```
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
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_RUNTIME_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
```
7、重新启动Kubelet服务，查看Kubelet进程是否使用Containerd服务正常启动：
```
# sudo systemctl daemon-reload 
```
```
# sudo systemctl restart kubelet
```
```
# ps -ef | grep kubelet | grep container-runtime-endpoint
```
8、通过查看node信息，可以看到此时运行时使用的是Containerd
```
# kubectl describe node <node_name>
```
```
# kubectl get nodes -o wide
```
9、uncordon节点使节点能够正常使用：
```
# kubectl uncordon <node_name>
```

### 迁移Mater节点
迁移Master节点按照迁移worker节点的的步骤逐个进行迁移。
