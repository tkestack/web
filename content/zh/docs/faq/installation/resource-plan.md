---
title: "如何规划部署资源"
linkTitle: "如何规划部署资源"
weight: 2
description: >
  如何规划部署资源
---

TKEStack支持使用物理机或虚拟机部署，采用kubernetes on kubernetes架构部署，在主机上只拥有一个物理机进程kubelet，其他kubernetes组件均为容器。架构上分为global集群和业务集群。global集群，运行整个TKEStack平台自身所需要的组件，业务集群运行用户业务。在实际的部署过程中，可根据实际情况进行调整。

安装TKEStack，需要提供两种角色的 Server：

Installer server 1台，用以部署集群安装器，安装完成后可以回收。

Global server，若干台，用以部署 Globa 集群，常见的部署模式分为三种：

1. **All in one 模式**，1台server部署 Global集群，global集群同时也充当业务集群的角色，即运行平台基础组件，又运行业务容器。global集群会默认设置taint不可调度，使用此模式时，需要手工在golbal集群【节点管理】-【更多】-【编辑Taint】中去除不可调度设置。\(关于taint，[了解更多](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)\)。由于此种模式不具有高可用能力，不建议在生产环境中使用。
2. **Global 与业务集群混部的高可用模式**，3台Server部署global集群，global集群同时也充当业务集群的角色，即运行平台基础组件，又运行业务容器。global集群会默认设置taint不可调度，使用此模式时，需要手工在golbal集群【节点管理】-【更多】-【编辑Taint】中去除不可调度设置。\(关于taint，[了解更多](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)\)。由于此种模式有可能因为业务集群资源占用过高而影响global集群，不建议在生产环境中使用。
3. **Global 与业务集群分别部署的高可用模式**，3台Server部署global集群，仅运行平台自身组件，业务集群单独在TKEStack控制台上创建（建议3台以上），此种模式下，业务资源占有与平台隔离，建议在生产环境中使用此种模式。

集群节点主机配置，请参考[资源需求](../../../installation/environment-requirement)。

