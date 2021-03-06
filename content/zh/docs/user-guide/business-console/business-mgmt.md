---
title: "业务管理"
linkTitle: "业务管理"
weight: 2
description: >
  业务管理
---

**这里用户可以管理业务。**

## 更改业务成员

1. 登录 TKEStack。
2. 切换至 【业务管理】控制台，点击【业务管理】。
3. 在“业务管理”页面中，可以看到已创建的业务列表。鼠标移动到要修改的业务上\(无需点击\)，成员列会出现修改图标按钮。如下图所示： ![&#x4FEE;&#x6539;&#x56FE;&#x6807;&#x6309;&#x94AE;](../../../../images/修改业务成员图标1.png)

   > 注意：修改业务成员仅限状态为Active的业务，这里可以新建和删除成员。

## 查看业务监控

1. 登录 TKEStack。
2. 切换至 【业务管理】控制台，点击【业务管理】。
3. 在“业务管理”页面中，可以看到已创建的业务列表。点击监控按钮，如下图所示：

   ![&#x76D1;&#x63A7;&#x6309;&#x94AE;](../../../../images/查看业务监控1.png)

4. 在右侧弹出窗口里查看业务监控情况，如下图所示：

   ![&#x4E1A;&#x52A1;&#x76D1;&#x63A7;&#x8BE6;&#x60C5;](../../../../images/业务监控详情1.png)

## 删除业务

1. 登录 TKEStack。
2. 切换至 【业务管理】控制台，点击【业务管理】。
3. 在“业务管理”页面中，可以看到已创建的业务列表。点击删除按钮，如下图所示：

   ![&#x5220;&#x9664;&#x4E1A;&#x52A1;](../../../../images/删除业务1.png)

   > 注意：删除业务成员仅限状态为Active的业务

## 对业务的操作

* 登录 TKEStack。
* 在【业务管理】控制台的【业务管理】中，单击【业务id】。如下图所示： 

![&#x4E1A;&#x52A1;id](../../../../images/businessid1.png)

a. **业务信息：** 在这里可以对业务名称、关联的集群、关联集群的资源进行限制等操作。

![&#x4E1A;&#x52A1;&#x4FE1;&#x606F;](../../../../images/业务信息1.png)

b. **成员列表：** 在这里可以对业务名称、关联的集群、关联集群的资源进行限制等操作。

![&#x4E1A;&#x52A1;&#x4FE1;&#x606F;](../../../../images/成员列表设置.png)

c. **子业务：** 在这里可以**新建本业务的子业务**或**通过导入子业务将已有业务变成本业务的子业务**

![&#x4E1A;&#x52A1;&#x4FE1;&#x606F;](../../../../images/子业务.png)

d. **业务下Namespace列表：** 这里可以管理业务下的Namespace

![&#x4E1A;&#x52A1;&#x4FE1;&#x606F;](../../../../images/业务Namespace列表.png)

​ 单击【新建Namespace】。在“新建Namespace”页面中，填写相关信息。如下图所示：

![&#x65B0;&#x5EFA;&#x7A7A;&#x95F4;&#x5217;&#x8868;](../../../../images/my-ns.png)

​ **名称**：不能超过63个字符，这里以`new-ns`为例

​ **集群**：`my-business`业务中的集群，这里以`global`集群为例

​ _资源限制\*_：这里可以限制当前命名空间下各种资源的使用量，可以不设置。

