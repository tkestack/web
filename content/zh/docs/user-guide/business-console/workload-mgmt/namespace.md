---
title: "命名空间"
linkTitle: "命名空间"
weight: 1
description: >
  命名空间
---

Namespaces 是 Kubernetes 在同一个集群中进行逻辑环境划分的对象， 您可以通过 Namespaces 进行管理多个团队多个项目的划分。在 Namespaces 下，Kubernetes 对象的名称必须唯一。您可以通过资源配额进行可用资源的分配，还可以进行不同 Namespaces 网络的访问控制。

## 使用方法

* 通过 TKEStack 控制台使用：TKEStack 控制台提供 Namespaces 的增删改查功能。
  * 【业务管理】平台下不支持对命名空间的直接操作，需在【平台管理】下[【业务管理】](../../../platform-console/business-mgmt)中指定业务通过“创建业务下的命名空间”来实现。
* 通过 Kubectl 使用：更多详情可查看 [Kubernetes 官网文档](https://kubernetes.io/docs/tasks/administer-cluster/namespaces/)。

## 相关知识

### 通过 ResourceQuota 设置 Namespaces 资源的使用配额

一个命名空间下可以拥有多个 ResourceQuota 资源，每个 ResourceQuota 可以设置每个 Namespace 资源的使用约束。可以设置 Namespaces 资源的使用约束如下：

* 计算资源的配额，例如 CPU、内存。
* 存储资源的配额，例如请求存储的总存储。
* Kubernetes 对象的计数，例如 Deployment 个数配额。

不同的 Kubernetes 版本，ResourceQuota 支持的配额设置略有差异，更多详情可查看 [Kubernetes ResourceQuota 官方文档](https://kubernetes.io/docs/concepts/policy/resource-quotas/)。 ResourceQuota 的示例如下所示：

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-counts
  namespace: default
spec:
  hard:
    configmaps: "10"  ## 最多10个 ConfigMap
    replicationcontrollers: "20" ## 最多20个 replicationcontroller
    secrets: "10" ## 最多10个 secret
    services: "10" ## 最多10个 service
    services.loadbalancers: "2"  ## 最多2个 Loadbanlacer 模式的 service
    cpu: "1000" ## 该 Namespaces 下最多使用1000个 CPU 的资源
    memory: 200Gi ## 该 Namespaces 下最多使用200Gi的内存
```

