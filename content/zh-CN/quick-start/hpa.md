---
title: "设置弹性伸缩" 
keywords: 'kubernetes, docker, helm, jenkins, istio, prometheus'
description: ''
---

[Pod 弹性伸缩 (HPA)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) 是高级版新增的功能，应用的资源使用率通常都有高峰和低谷的时候，如何动态地根据资源使用率来削峰填谷，提高集群的平台和集群资源利用率，让 Pod 副本数自动调整呢？这就有赖于 Horizontal Pod Autoscaling 了，顾名思义，能够使 Pod 水平自动伸缩，也是最能体现 KubeSphere 之于传统运维价值的地方，用户无需对 Pod 手动地水平扩缩容 (Scale out/in)。HPA 仅适用于创建部署 (Deployment) 时或创建部署后设置，支持根据集群的监控指标如 CPU 使用率和内存使用量来设置弹性伸缩，当业务需求增加时，KubeSphere 能够无缝地自动水平增加 Pod 数量，提高系统的稳定性。

## 弹性伸缩工作原理

HPA 在 Kubernetes 中被设计为一个 Controller，可以在 KubeSphere 中通过简单的设置或通过 UI 的 `kubectl autoscale` 命令来创建。HPA Controller 默认每 `30 秒` 轮询一次，检查工作负载中指定的部署 (Deployment) 的资源使用率，如 CPU 使用率或内存使用量，同时与创建部署时设定的值和指标做比较，从而实现 Pod 副本数自动伸缩的功能。

在部署中创建了 HPA 后，Controller Manager 将会访问 Metrics-server，获取用户定义的资源中每一个容器组的利用率或原始值 (取决于指定的目标类型) 的平均值，然后，与 HPA 中设置的指标进行对比，同时计算部署中的 Pod 需要弹性伸缩的具体值并实现操作。在底层 Kubernetes 中的 Pod 的 CPU 和内存资源，实际上还分为 limits 和 requests 两种情况，在调度的时候，kube-scheduler 将会根据 requests 的值进行计算。因此，当 Pod 没有设置资源请求值 (request) 时，弹性伸缩功能将不会工作。

![弹性伸缩工作原理](/hpa.svg)

## 目的

本示例演示创建一个设置了弹性伸缩的应用，通过另外创建的多个 Pod 循环向该应用发送无限的查询请求访问应用的服务，相当于手动增加 CPU 负载，即模拟多个用户同时访问该服务，演示其弹性伸缩的功能，详细说明 HPA 的工作原理以及如何在部署中设置 Pod 水平自动伸缩。

## 预估时间

约 25 分钟。

## 前提条件

- 已创建了企业空间、项目和普通用户 `project-regular` 账号，若还未创建请参考 [多租户管理快速入门](../admin-quick-start)；
- 使用项目管理员 `project-admin` 邀请项目普通用户 `project-regular` 加入项目并授予 `operator` 角色，参考 [多租户管理快速入门 - 邀请成员](../admin-quick-start/#邀请成员)。


## 操作示例

<!-- ### 示例视频

<video controls="controls" style="width: 100% !important; height: auto !important;">
  <source type="video/mp4" src="https://kubesphere-docsvideo.gd2.qingstor.com/demo5-hpa-quick-start.mp4">
</video> -->

### 创建 HPA

#### 第一步：创建部署

1、以项目普通用户 `project-regular` 登录 KubeSphere。在项目列表里点击之前创建好的项目 `demo-namespace`，进入项目；

在左侧菜单栏选择 **工作负载 → 部署**，点击 **创建部署**。

![创建部署](/demo5-create-deployment.png)

2、填写部署的基本信息如下：

- 名称：必填，起一个简洁明了的名称，便于用户浏览和搜索，如 `hpa-example`
- 别名：可选，更好的区分资源，并支持中文名称
- 更新策略：选择 RollingUpdate

点击 **下一步**；

![填写基本信息](/hpa-demo-1.png)

#### 第二步：配置弹性伸缩参数

点击 **弹性伸缩**，填写信息如下：

- 弹性伸缩 (HPA)
   - 最小副本数：弹性伸缩的容器组数量下限，此处设置 `1`
   - 最大副本数：弹性伸缩的容器组数量上限，此处设置 `10`
   - CPU Request Target(%) (CPU 目标值)：当 CPU 使用率超过或低于此目标值时，将相应地添加或删除副本，此处设置为 `50%`
   - Memory Request Target(Mi) (内存目标值)：当内存使用量超过或低于此目标值时，将添加或删除副本，本示例以增加 CPU 负载作为测试，内存暂不作限定

 > 注：容器组模板中可以设置 HPA 相关参数，以设置 CPU 目标值作为弹性伸缩的计算参考，实际上会为部署创建一个 `Horizontal Pod Autoscaler` 来调度其弹性伸缩。

![](https://pek3b.qingstor.com/kubesphere-docs/png/20190514011228.png)


#### 第三步：添加容器

1、点击 **添加容器**，填写容器的基本信息，容器名称可自定义，镜像填写 `mirrorgooglecontainers/hpa-example`，将默认拉取 **latest** 的镜像。CPU 和内存此处暂不作限定，将使用在创建项目时指定的默认请求值；

![](https://pek3b.qingstor.com/kubesphere-docs/png/20190514011315.png)

2、下拉至**服务设置**，自定义端口名称，如 port，默认 TCP 协议，端口号为 `80`。

![](https://pek3b.qingstor.com/kubesphere-docs/png/20190514011621.png)

3、点击 **保存**，然后点击 **下一步**；由于示例不会用到持久化存储，因此存储卷也不作设置，点击 **下一步**；标签和节点选择器无需设置，点击 **创建**，即可查看 HPA 示例的运行状态详情。

### 创建服务

#### 第一步：填写基本信息

在左侧菜单栏，点击 **网络与服务 → 服务** 进入服务列表页，点击 **创建**；

- 名称：此处填写 `hpa-example`，注意该服务名将用于被其它 Pod 访问
- 别名和描述信息：HPA 示例，帮助用户更好地了解该服务

点击 **下一步**；

![创建服务](/hpa-service.png)

#### 第二步：服务设置

1、由于是在集群内部访问该服务，因此服务设置选择第一项 **通过集群内部 IP 来访问服务 Virtual IP**，服务信息参考以下填写：

- 选择器：点击 **指定工作负载** ，在弹出的界面中指定上一步创建的部署 hpa-example；
- 端口：端口名称可自定义，服务的端口和目标端口都填写 `TCP` 协议的 `80` 端口；
- 会话亲和性：None

点击 **下一步**；

![](https://pek3b.qingstor.com/kubesphere-docs/png/20190514012105.png)

2、标签设置保留默认值 `app: hpa-example`，点击 **下一步**；

3、外网访问方式选择 `None`，即仅在集群内部访问该服务，点击 **创建** 即可创建成功。

![](https://pek3b.qingstor.com/kubesphere-docs/png/20190514012207.png)

### 创建 Load-generator

另外创建一个部署 (Deployment) 用于向上一步创建的服务不断发送查询请求，模拟增加 CPU 负载。

#### 第一步：基本信息

在左侧菜单栏选择 **工作负载 → 部署**，点击创建部署，填写部署的基本信息，其它项保持默认：


- 名称：必填，起一个简洁明了的名称，便于用户浏览和搜索，如 `load-generator`
- 别名：更好的区分资源，并支持中文名称，比如 `增加负载`
- 描述信息：简单介绍该部署


点击 **下一步**；

![填写基本信息](/hpa-load-basic.png)

#### 第二步：容器组模板  

1、点击 **添加容器**，设置参数如下：

- 镜像名称：填写 `busybox`；
- CPU 和内存：此处暂不作限定，将使用在创建项目时指定的默认请求值。

![](https://pek3b.qingstor.com/kubesphere-docs/png/20190514012524.png)

3、然后下滑至 **启动命令**，勾选 **启动命令**，在展开的运行命令和参数中填写用于对 hpa-example 服务增加 CPU 负载的命令和参数，其它设置暂无需配置。设置`运行命令`和`参数`如下：

> 注意：参数中服务的 http 地址应替换为您实际的服务和项目名称。例如，我们在创建 HPA 的服务时，服务名称为 hpa-example，当前的项目名称为 demo-namespace，那么该服务在内部的 http 地址为 `http://hpa-example.demo-namespace.svc.cluster.local`。

```
# 运行命令
sh
-c

# 参数 (http 地址参考：http://{$服务名称}.{$项目名称}.svc.cluster.local)
while true; do wget -q -O- http://hpa-example.demo-namespace.svc.cluster.local; done
```

![](https://pek3b.qingstor.com/kubesphere-docs/png/20190514012832.png)

4、完成填写后，点击 **保存**；点击 **下一步**；本示例暂未用到存储，点击 **下一步** 跳过存储卷设置；标签设置保留默认值即可，点击 **创建**；

至此，以上一共创建了两个部署 (分别是 hpa-example 和 load-generator ) 和一个服务 (hpa-example)。

![](https://pek3b.qingstor.com/kubesphere-docs/png/20190514014625.png)


### 验证弹性伸缩

#### 第一步：查看部署状态

在部署列表中，点击之前创建的部署 `hpe-example`，进入资源详情页，请重点关注此时容器组的弹性伸缩状态和当前的 CPU 使用率以及它的监控情况。

![](https://pek3b.qingstor.com/kubesphere-docs/png/20190514014254.png)

#### 第二步：查看弹性伸缩情况

待 `load-generator` 的所有副本的容器都创建成功并开始访问 `hpe-example` 服务时，如下图所示，刷新页面后可见 CPU 使用率明显升高，先快速上升至 5210 %，并且期望副本和实际运行副本数都变成了 4，这是由于我们之前设置的 Horizontal Pod Autoscaler 开始工作了，`load-generator` 循环地请求该 `hpa-example` 服务使得 CPU 使用率迅速升高，HPA 开始工作后使得该服务的后端 Pod 副本数迅速增加共同处理大量的请求，`hpa-example` 的副本数会随 CPU 的使用率升高而继续增加。

一分钟左右 CPU 使用率降低至 629 %，副本数增加至所设置 HPA 的最大值 10，正好也证明了弹性伸缩的工作原理。

![](https://pek3b.qingstor.com/kubesphere-docs/png/20190514014919.png)

理论上，从容器组的 CPU 监控曲线中可以看到最初创建的 1 个容器组的 CPU 使用量有一个明显的升高趋势，待 HPA 开始工作时可以发现 CPU 使用量有明显降低的趋势，最终趋于平稳，而此时新增的 Pod 上可以看到 CPU 使用量在增加。

![](https://pek3b.qingstor.com/kubesphere-docs/png/20190514015547.png)

> 说明：HPA 工作后，Deployment 最终的副本数量可能需要几分钟才能稳定下来，删除负载后 Pod 数量回缩至正常状态也需要几分钟。 由于环境的差异，不同环境中最终的副本数量可能与本示例中的数量不同。

#### 停止负载

1、在左侧菜单栏选择 **工作负载 → 部署**，在部署列表中选择 `load-generator`，点击界面上方的 **删除** （或将之前设置的循环请求命令删除），停止负载；

2、再次查看 `hpe-example` 的运行状况，可以发现几分钟后它的 CPU 利用率已缓慢降到 10 %，并且 HPA 将其副本数量最终减少至最小副本数 1，最终恢复了正常状态，从 CPU 使用量监控曲线反映的趋势也可以帮助我们进一步理解弹性伸缩的工作原理；

> 注意：在完成本示例后，请将工作负载 load-generator 删除，防止其一直访问该应用而造成 CPU 资源的不必要的消耗或集群因资源不足而出现问题。

![](https://pek3b.qingstor.com/kubesphere-docs/png/20190514020958.png)

3、在部署的详情页面，可以下钻到每个 Pod 的单个容器的监控详情，点击最初创建的容器组进入容器组详情页，选择 「监控」，查看该容器组的 CPU 使用量和网络流入、出速率监控曲线，与本示例的操作流程和 HPA 原理正好相符。

![](https://pek3b.qingstor.com/kubesphere-docs/png/20190514020208.png)

### 修改弹性伸缩

创建后若需要修改弹性伸缩的参数，可以在部署详情页，点击 **更多操作 → 弹性伸缩**，如下页面支持修改其参数：

![修改弹性伸缩](/update-hpa.png)

### 取消弹性伸缩

若该部署无需设置 HPA，则可以在当前的部署详情页中，点击弹性伸缩右侧的 **···**，然后选择 **取消**。

![取消 HPA](/cancel-hpa.png)

至此，您已经熟悉了如何在创建部署时设置弹性伸缩的基本操作。












