---
title: "事件"
linkTitle: "事件"
weight: 6
description: >
  事件
---

日志针对的是容器。包括了 Kuberntes 集群的运行容器的日志情况和各类资源的调度情况，对维护人员日常观察资源的变更以及定位问题均有帮助。

## 日志控制台操作指引

* 登录 TKEStack，切换到【业务管理】控制台，点击【应用管理】，选择【日志】。
* 进入“日志”页面。 
* 可以按照不用的命名空间和资源类型进行筛选。

![](../../../../../images/image%20%2869%29.png)

> 注意：Kubernetes 默认只将最近一个小时的事件存储在 ETCD 中，若想实现事件的持久化存储和查询操作

> 您可以参考 [事件持久化](https://github.com/tkestack/tke/blob/master/hack/addon/readme/PersistentEvent.md) 将事件导入外部存储，实现事件的持久化存储

