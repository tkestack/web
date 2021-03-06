---
title: "ConfigMap"
linkTitle: "ConfigMap"
weight: 1
description: >
  ConfigMap
---

通过 ConfigMap 您可以将配置和运行的镜像进行解耦，使得应用程序有更强的移植性。ConfigMap 是有 key-value 类型的键值对，您可以通过控制台的 Kubectl 工具创建对应的 ConfigMap 对象，可以通过挂载数据卷、环境变量或在容器的运行命令中使用 ConfigMap。 ConfigMap 有两种使用方式，创建负载时做为数据卷挂载到容器和作为环境变量映射到容器。

## ConfigMap 控制台操作指引

### 创建 ConfigMap

* 登录TKEStack，切换到【业务管理】控制台，选择左侧导航栏中的【应用管理】。
* 选择需要创建ConfigMap的【业务】下相应的【命名空间】，展开【配置管理】列表，进入ConfigMap管理页面。 
* 单击【新建】，进入 “新建ConfigMap” 页面。如下图所示：

![&#x65B0;&#x5EFA;ConfigMap](../../../../../../images/new-config-map.png)

* 根据实际需求，设置 ConfigMap 参数。关键参数信息如下：
  * 名称：自定义。
  * 命名空间：根据实际需求进行选择命名空间类型
  * 定义变量名和变量值。
* 单击【创建ConfigMap】，完成创建。

### 更新 ConfigMap

1. 登录TKEStack，切换到业务管理控制台，选择左侧导航栏中的【应用管理】。
2. 选择需要创建ConfigMap的业务下相应的命名空间，展开配置管理列表，进入ConfigMap管理页面。 
3. 在需要更新 YAML 的 ConfigMap 行中，单击【编辑YAML】，进入更新 ConfigMap 页面。
4. 在 “更新ConfigMap” 页面，编辑 YAML，单击【完成】，即可更新 YAML。

   > 如需修改 key-values，编辑 YAML 中 data 的参数值，单击【完成】，即可完成更新。

## Kubectl 操作 ConfigMap 指引

### YAML 示例

```yaml
apiVersion: v1
data:
  key1: value1
  key2: value2
  key3: value3
kind: ConfigMap
metadata:
  name: test-config
  namespace: default
```

* data：ConfigMap 的数据，以 key-value 形式呈现。
* kind：标识 ConfigMap 资源类型。
* metadata：ConfigMap 的名称、Label等基本信息。
* metadata.annotations：ConfigMap 的额外说明，可通过该参数设置腾讯云 TKE 的额外增强能力。

### 创建 ConfigMap

#### 方式一：通过 YAML 示例文件方式创建

1. 参考 [YAML 示例](../configmap#YAMLSample)，准备 ConfigMap YAML 文件。
2. 安装 Kubectl，并连接集群。操作详情请参考 [通过 Kubectl 连接集群](https://cloud.tencent.com/document/product/457/8438)。
3. 执行以下命令，创建 ConfigMap YAML 文件。

   ```text
   kubectl create -f ConfigMap YAML 文件名称
   ```

   例如，创建一个文件名为 web.yaml 的 ConfigMap YAML 文件，则执行以下命令：

   ```text
   kubectl create -f web.yaml
   ```

4. 执行以下命令，验证创建是否成功。

   ```text
   kubectl get configmap
   ```

   返回类似以下信息，即表示创建成功。

   ```text
   NAME          DATA      AGE
   test          2         39d
   test-config   3         18d
   ```

#### 方式二：通过执行命令方式创建

执行以下命令，在目录中创建 ConfigMap。

```text
kubectl create configmap <map-name> <data-source>
```

* &lt;map-name&gt;：表示 ConfigMap 的名字。
* &lt;data-source&gt;：表示目录、文件或者字面值。

更多参数详情可参见 [Kubernetes configMap 官方文档](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#create-a-configmap)。

### 使用 ConfigMap

#### 方式一：数据卷使用 ConfigMap 类型

YAML 示例如下：

```yaml
apiVersion: v1
 kind: Pod
 metadata:
   name: nginx
 spec:
   containers:
     - name: nginx
       image: nginx:latest
       volumeMounts:
        name: config-volume
        mountPath: /etc/config
   volumes:
        name: config-volume
        configMap:
          name: test-config ## 设置 ConfigMap 来源
          ## items:  ## 设置指定 ConfigMap 的 Key 挂载
          ##   key: key1  ## 选择指定 Key
          ##   path: keys ## 挂载到指定的子路径
   restartPolicy: Never
```

#### 方式二：环境变量中使用 ConfigMap 类型

YAML 示例如下：

```yaml
apiVersion: v1
 kind: Pod
 metadata:
   name: nginx
 spec:
   containers:
     - name: nginx
       image: nginx:latest
       env:
         - name: key1
           valueFrom:
             configMapKeyRef:
               name: test-config ## 设置来源 ConfigMap 文件名
               key: test-config.key1  ## 设置该环境变量的 Value 来源项
   restartPolicy: Never
```

