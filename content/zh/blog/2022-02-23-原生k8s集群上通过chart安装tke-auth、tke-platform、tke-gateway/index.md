---
layout: blog
title: "原生k8s集群上通过chart安装tke-auth、tke-platform、tke-gateway"
date: 2022-02-23
# slug of this blog url
slug: tkestack-installer-chart
---

**Author**: wl-chen

[TKEStack](https://github.com/tkestack/tke) 是一个开源项目，它为在生产环境中部署容器的组织提供了一个容器管理平台。TKEStack 让您可以轻松地在任何地方运行 Kubernetes、满足 IT 需求并为 DevOps 团队赋能。
本文将介绍如何使用 [helm](https://helm.sh/zh/docs/) 在原生 k8s 集群上，以 chart 的形式安装 TKEStack 中的核心组件 tke-auth、tke-platfor、tke-gateway，实现 TKEStack 的轻量化安装。 

## 前置要求

本文介绍的内容是建立在已经有一个正常运行的 k8s 集群的基础上，并且下面的操作需要在 master 节点上进行操作。如果没有现有的 k8s 集群，可以通过 kind 创建本地集群并进行下面的操作。
本文介绍的内容需要通过 helm 安装一些实验用途的组件，可参考[安装Helm](https://helm.sh/zh/docs/intro/install/)进行安装。

## 创建指定 namespace

tke-auth、tke-platform、tke-gateway 三个 chart 需要运行在指定的 namespace 下，执行如下命令：

```sh
kubectl create namespace tke
```

## 安装 chart

本文提供了二进制可执行程序来生成 tke-auth、tke-platform、tke-gateway 三个 chart 的 values 文件

执行如下命令拉取 TKEStack 项目代码

```sh
git clone https://github.com/tkestack/tke.git
```

在 TKEStack 项目的`charts/bin`目录放置了可执行文件`bin`和需要填写的yaml文件`customConfig.yaml`。`customConfig.yaml`文件中一些注释“必填”的参数，需要填写，其余的参数可根据需要选填，选填部分为空时会自动填充默认值。`customConfig.yaml`内容如下：

```yaml
# 必填，etcd访问地址，形式如https://172.19.0.2:2379
etcd:
  host: https://172.18.0.2:2379 
# 必填，服务器ip，数组形式
serverIPs:
  - 172.18.0.2
# 访问的域名，数组形式
dnsNames:
  - tke.gateway
# 必填，集群front-proxy-ca.crt文件地址，默认位置为/etc/kubernetes/pki/front-proxy-ca.crt
frontProxyCaCrtAbsPath: /etc/kubernetes/pki/front-proxy-ca.crt
# 必填，集群etcd的ca.crt文件地址，默认位置为/etc/kubernetes/pki/etcd/ca.crt
etcdCrtAbsPath: /etc/kubernetes/pki/etcd/ca.crt
# 必填，集群etcd的ca.key文件地址，默认位置为/etc/kubernetes/pki/etcd/ca.key
etcdKeyAbsPath: /etc/kubernetes/pki/etcd/ca.key
tke-auth:
  api:
    # 必填
    replicas: 1
    # 必填
    image: tkestack/tke-auth-api-amd64:74592a3bceb5bebca602bea21aaebf78007a3bb2
    # 必填，数组形式，auth的重定向访问地址，包括集群服务器ip地址（必填）、tke-gateway的域名（可选）、集群高可用的VIP地址（可选）和集群的公共可访问域名（可选）
    redirectHosts: 
      - 172.18.0.2
    enableAudit: 
    # tke-auth-api组件在node上的对外暴露端口，默认31138
    nodePort: 
    # tke集群的租户id，默认default
    tenantID: 
    # OIDC认证方式的secret，默认自动生成
    oIDCClientSecret: 
    # authentication用户名，默认为admin
    adminUsername: 
  controller:
    # 必填
    replicas: 1
    # 必填
    image: tkestack/tke-auth-controller-amd64:74592a3bceb5bebca602bea21aaebf78007a3bb2
    # tke集群的用户名，默认为admin
    adminUsername: 
    # tke集群的密码，默认自动生成
    adminPassword: 
tke-platform:
  # 必填 VIP，或者公网可访问的集群IP
  publicIP:
  metricsServerImage: metrics-server:v0.3.6
  addonResizerImage: addon-resizer:1.8.11
  api:
    # 必填
    replicas: 1
    # 必填
    image: tkestack/tke-platform-api-amd64:bc48bed59bff2022d87db5e1484481715357ee7c
    enableAuth: true
    enableAudit: 
    # OIDC认证方式客户端id，默认为default
    oIDCClientID: 
    # OIDC认证方式的issuer_url，默认为https://tke-auth-api/oidc
    oIDCIssuerURL: 
    # 是否开启OIDC认证，默认不开启，值为空
    useOIDCCA:
  controller:
    # 必填
    replicas: 1
    # 必填
    providerResImage: tkestack/provider-res-amd64:v1.21.4-1
    # 必填
    image: tkestack/tke-platform-controller-amd64:bc48bed59bff2022d87db5e1484481715357ee7c
    # 默认为docker.io
    registryDomain:
    # 默认为tkestack
    registryNamespace:
    # 监控存储类型，默认为influxdb
    monitorStorageType: 
    # 监控存储地址，为tke集群master ip地址加8086端口
    monitorStorageAddresses:
tke-gateway:
  # 必填
  image: tkestack/tke-gateway-amd64:bc48bed59bff2022d87db5e1484481715357ee7c
  # 默认为docker.io
  registryDomainSuffix:
  # tke集群的租户id，默认default
  tenantID:
  # OIDC认证方式的secret，默认自动生成
  oIDCClientSecret:
  # 是否开启自签名，默认为true
  selfSigned: true
  # 第三方cert证书，在selfSigned为false时需要填值
  serverCrt:
  # 第三方certKey，在selfSigned为false时需要填值
  serverKey:
  enableAuth: true
  enableBusiness:
  enableMonitor:
  enableRegistry:
  enableLogagent:
  enableAudit:
  enableApplication:
  enableMesh:

```

`customConfig.yaml`文件中的参数填写完毕后，执行`bin`,会在同级目录生成`auth-chart-values.yaml`、`platform-chart-values.yaml`、`gateway-chart-values.yaml`三个yaml文件，分别对应三个chart（tke-auth、tke-platform、tke-gateway）在安装时需要的`values.yaml`文件

切换到项目的`charts/`目录，接下来进行chart的安装：

```sh
# tke-auth的安装
helm install -f bin/auth-chart-values.yaml tke-auth tke-auth/
```

```sh
# tke-platform的安装
helm install -f bin/platform-chart-values.yaml tke-platform tke-platform/
```

```sh
# tke-gateway的安装
helm install -f bin/gateway-chart-values.yaml tke-gateway tke-gateway/
```

通过如下命令如果能查询到三个组件对应的 `api-resources`，则表示`chart`安装成功

```sh
kubectl api-resources | grep tke
```

chart安装完成后，可以查询到以下信息，如图所示：
<img alt="" width="100%" src="auth-api-resources.png">

<img alt="" width="100%" src="platform-api-resources.png">

<img alt="" width="100%" src="svc.png">

## 修改集群 apiserver 配置

修改 k8s 集群中`/etc/kubernetes/mainfest/kube-apiserver.yaml`的内容，在`spec.containers.command`字段增加以下两条：

```yaml
# 如果已有这两个参数，则将其按照以下内容修改
- --authorization-mode=Node,RBAC,Webhook
- --authorization-webhook-config-file=/etc/kubernetes/pki/tke-authz-webhook.yaml
```

在参数`authorization-webhook-config-file`对应的目录`/etc/kubernetes/pki/`下新建文件`tke-authz-webhook.yaml`，文件内容如下（其中`cluster.server`参数中的IP地址需要修改为master的IP地址）：

```yaml
apiVersion: v1
kind: Config
clusters:
  - name: tke
    cluster:
      server: https://172.19.0.2:31138/auth/authz
      insecure-skip-tls-verify: true
users:
  - name: admin-cert
    user:
      client-certificate: /etc/kubernetes/pki/webhook.crt
      client-key: /etc/kubernetes/pki/webhook.key
current-context: tke
contexts:
- context:
    cluster: tke
    user: admin-cert
  name: tke
```

将二进制执行文件生成的`webhook.crt`和`webhook.key`（位置在二进制执行文件同级目录`/data`内）同时放到对应位置`/etc/kubernetes/pki/`

### 创建独立集群

访问地址`http://{master节点ip}/tkestack`,出现如下登陆界面，输入之前设置的用户名`adminusername`和密码`adminpassword`,如无设置，默认用户名为`admin`，密码为`YWRtaW4=`。

<img alt="" width="100%" src="登陆界面.png">

登陆后，点击集群管理的新建独立集群：

<img alt="" width="100%" src="新建独立集群.png">

具体的集群创建信息可参考文档[集群创建](https://tkestack.github.io/web/zh/docs/user-guide/platform-console/cluster-mgmt/#%E6%96%B0%E5%BB%BA%E7%8B%AC%E7%AB%8B%E9%9B%86%E7%BE%A4)

创建集群完成后，可以在页面端看到如下状态

<img alt="" width="100%" src="新建独立集群-成功.png">

并且在创建集群的master节点上可以查询到相关集群信息

<img alt="" width="100%" src="cluster信息.png">
