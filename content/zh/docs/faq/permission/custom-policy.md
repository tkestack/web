---
title: "如何设置自定义策略"
linkTitle: "如何设置自定义策略"
weight: 3
description: >
  如何设置自定义策略
---

TKEStack 策略（policy）用来描述授权的具体信息。核心元素包括操作（action）、资源（resource）以及效力（effect）。

## 操作（action）

描述允许或拒绝的操作。操作可以是 API（以 name 前缀描述）或者功能集（一组特定的 API，以 permid 前缀描述）。该元素是必填项。

## 资源（resource）

描述授权的具体数据。资源是用六段式描述。每款产品的资源定义详情会有所区别。有关如何指定资源的信息，请参阅您编写的资源声明所对应的产品文档。该元素是必填项。

## 效力（effect）

描述声明产生的结果是“允许”还是“显式拒绝”。包括 allow（允许）和 deny （显式拒绝）两种情况。该元素是必填项。

## 策略样例

该样例描述为：允许关联到此策略的用户，对cls-123集群下的工作负载deploy-123中的所有资源，有查看权限。

```javascript
{
  "actions": [
    "get*",
    "list*",
    "watch*"
  ],
  "resources": [
    "cluster:cls-123/deployment:deploy-123/*"
  ],
  "effect": "allow"
}
```

