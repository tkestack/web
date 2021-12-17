---
layout: blog
title: "TKEStack 中的认证与鉴权"
date: 2021-12-18
# slug of this blog url
slug: tkestack-auth
---

**Author**: LeoRyu

_认证鉴权对于 K8s 而言可以说是最为复杂、灵活和关键的部分，本文将介绍 TKEStack 中认证鉴权机制是如何运作的。_


## 开始之前

由于认证鉴权涉及的相关知识很多，为了避免读者在下面探究 TKEStack 认证鉴权机制时查处相关名词知识，在开始之前先介绍一些本文涉及到的知识概念。

### K8s 中的一些身份认证方式

- X509 客户证书。通过给 API 服务器传递 `--client-ca-file=SOMEFILE` 选项，就可以启动客户端证书身份认证。 所引用的文件必须包含一个或者多个证书机构，用来验证向 API 服务器提供的客户端证书。 如果提供了客户端证书并且证书被验证通过，则 subject 中的公共名称（Common Name）就被 作为请求的用户名。

- 静态令牌文件。当 API 服务器的命令行设置了 `--token-auth-file=SOMEFILE` 选项时，会从文件中 读取持有者令牌。目前，令牌会长期有效，并且在不重启 API 服务器的情况下 无法更改令牌列表。

- 服务账号令牌。服务账号（Service Account）是一种自动被启用的用户认证机制，使用经过签名的 持有者令牌来验证请求。服务账号通常由 API 服务器自动创建并通过 ServiceAccount 准入控制器 关联到集群中运行的 Pod 上。 持有者令牌会挂载到 Pod 中可预知的位置，允许集群内进程与 API 服务器通信。 服务账号也可以使用 Pod 规约的 serviceAccountName 字段显式地关联到 Pod 上。

- OpenID Connect（OIDC）令牌。OpenID Connect 是一种 OAuth2 认证方式， 被某些 OAuth2 提供者支持，例如 Azure 活动目录、Salesforce 和 Google。 协议对 OAuth2 的主要扩充体现在有一个附加字段会和访问令牌一起返回， 这一字段称作 ID Token（ID 令牌）。详情可参考 https://openid.net/connect/ 。

更多认证相关信息可参考 [用户认证](https://kubernetes.io/zh/docs/reference/access-authn-authz/authentication/)[1]。

### K8s 中的一些鉴权方式

- Node。一个专用鉴权组件，根据调度到 kubelet 上运行的 Pod 为 kubelet 授予权限。
- ABAC。基于属性的访问控制（ABAC）定义了一种访问控制范型，通过使用将属性组合 在一起的策略，将访问权限授予用户。策略可以使用任何类型的属性（用户属性、资源属性、 对象，环境属性等）。
- RBAC。基于角色的访问控制（RBAC）是一种基于企业内个人用户的角色来管理对 计算机或网络资源的访问的方法。在此上下文中，权限是单个用户执行特定任务的能力， 例如查看、创建或修改文件。
- Webhook。WebHook 是一个 HTTP 回调：发生某些事情时调用的 HTTP POST； 通过 HTTP POST 进行简单的事件通知。实现 WebHook 的 Web 应用程序会在发生某些事情时 将消息发布到 URL。
- AlwaysAllow。允许所有请求。仅在你不需要 API 请求 的鉴权时才使用。
- AlwaysDeny。阻止所有请求。仅用于测试。

更多认证相关信息可参考 [鉴权概述](https://kubernetes.io/zh/docs/reference/access-authn-authz/authorization/)[2]。

### Casbin
Casbin 是一款轻量级的开源访问控制框架，支持 PERM 模式（Policy，Effect，Request，Matchers），相关概念如下：

- Request：定义了请求的元组对象，至少包含访问实体（sub）、访问的资源（obj）和行为（Action），例如：r={sub, obj,act}。
- Policy：定义了鉴权规则各字段的名称和顺序，至少包含访问实体（sub）、访问的资源（obj）和行为（Action），例如：p={sub, obj, act}。
- Matchers：定义了 Request 和 Policy 的匹配规则。
- Effect：用于对所有 Matchers 匹配后的结果，再进行一次逻辑组合判断（例如：e = some(where (p.eft == allow)) && !some(where (p.eft == deny))，这个例子组合的逻辑含义是：如果有匹配出结果为 alllow 的策略，并且没有匹配出结果为 deny 的策略，则结果为真，如果有任何deny，都为假）。

关于本项目详细信息可以参考 [Casbin 概述](https://casbin.org/docs/zh-CN/overview)[3]。

## 两种认证鉴权场景

TKEStack 扩展 K8s API 的方式是通过 aggregated apiserver[4] 实现的，这种方式与 CRD 的方式不同，每一个扩展组件都可以被视为一个独立的 APIServer，这使得 TKEStack 的认证鉴权相较之下更加灵活，但也更加复杂。为了更加方便和清晰地展示认证鉴权的流程，这里我们分为两种场景分别展开：第一种是用户通过浏览器由 tke-gateway 入口访问各个组件的资源；第二种是用户通过 kubectl 由 kube-apiserver 入口访问各个组件的资源。

## 由浏览器发起的访问

### 认证

用户通过浏览器登陆访问的用户信息并不是 K8s 集群可以识别的用户，无法由集群本身进行认证流程，需要外部的 OIDC provider 进行认证。在 TKEStack 中承担此工作的是 tke-auth 组件，tke-auth 中是通过 [dex](https://github.com/dexidp/dex) 实现的 OIDC 服务。

在由浏览器发起的访问场景下，认证流程的链路大致分为两步骤：

1. `用户浏览器中登录 -> tke-gateway -> tke-auth返回认证信息凭证 -> 设置访问凭证到浏览器`
2. `已有访问凭证的前端 -> tke-gateway -> tke-xxxx -> tke-auth校验认证凭证`

整个认证过程中数据传输都是通过浏览器或是 TKEStack 系统组件完成的，没有 kube-apiserver 的参与。

### 鉴权

当前版本 TKEStack 系统组件的鉴权模式默认是以 [SubjectAccessReview](https://kubernetes.io/docs/reference/access-authn-authz/authorization/) 继承了 kube-apiserver 的鉴权模式，而 kube-apiserver 中鉴权模式配置了 webhook 指向了 tke-auth 组件。

tke-auth 中的鉴权是基于 [casbin](https://github.com/casbin/casbin) 实现的，基于其 PERM 模型 TKEStack 完成了一套 RBAC 模型的实现：

```
[request_definition]
r = sub, dom, obj, act

[policy_definition]
p = sub, dom, obj, act, eft

[role_definition]
g = _, _, _

[policy_effect]
e = some(where (p.eft == allow)) && !some(where (p.eft == deny))

[matchers]
m = g(r.sub, p.sub, r.dom) && keyMatchCustom(r.obj, p.obj) && keyMatchCustom(r.act, p.act)
```

- request_definition：定义了模型的 Request
- policy_definition：定义了模型的 Policy
- role_definition：定义了模型的用户和角色的映射关系， 三个空缺分别表示用户, 角色, 域，TKEStack 使用的域为业务 ID
- policy_effect：定义了模型的 Effect
- matchers：定义了模型的匹配方式，keyMatchCustom 为自定义匹配函数，支持以正则匹配和通配符的方式，匹配 Role 和 Policy 的字段

用户通过浏览器访问场景下 TKEStack 的鉴权流程整个链路如下：

`用户浏览器发起请求 -> tke-gateway -> tke-xxx -> kube-apiserver webhook -> tke-auth返回鉴权结果`

## 集群内由 kubectl 发起的请求

### 认证

集群内通过 kubectl 发起请求场景下的认证流程与 K8s 本身的认证流程基本无异，因此整个链路也非常简单：

`kubectl 读取 kubeconfig 中的认证信息 -> kube-apiserver 检验认证信息`

由于此场景下的用户信息是可以被 K8s 识别的，且 kubectl 是直接连接 kube-apiserver 的，认证流程在请求到达 TKEStack 系统组件之前就完成了。

### 鉴权

之前在由浏览器发起访问场景中已经提到 TKEStack 系统组件的鉴权模式是以 [SubjectAccessReview](https://kubernetes.io/docs/reference/access-authn-authz/authorization/) 继承了 kube-apiserver 的鉴权模式，这种模式也同样适用于集群内由 kubectl 发起请求的场景，只不过在 kube-apiserver 中生效的鉴权模式不再是指向 tke-auth 的 webhook，而是 K8s 的 RBAC/ABAC 模式。

整个鉴权流程的链路如下：

`kubectl 发起请求 -> kube-apiserver -> tke-xxx -> kube-apiserver 返回 RBAC/ABAC 鉴权结果`

[1] Kubernetes 用户认证 [https://kubernetes.io/zh/docs/reference/access-authn-authz/authentication/]

[2] Kubernetes 鉴权概述 [https://kubernetes.io/zh/docs/reference/access-authn-authz/authorization/]

[3] Casbin 概述 [https://casbin.org/docs/zh-CN/overview]

[4] Kubernetes API Aggregation Layer [https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/apiserver-aggregation/]
