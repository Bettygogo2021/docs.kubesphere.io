---
title: "在 Kubernetes 在线部署 KubeSphere"
keywords: 'kubernetes, docker, helm, jenkins, istio, prometheus'
description: '将 KubeSphere 部署在 Kubernetes 之上'
---

KubeSphere 支持在已有 Kubernetes 集群之上在线安装 [KubeSphere](/zh-CN/)。在安装之前，请确认您的环境满足以下 [前提条件](../prerequisites)：


> - `Kubernetes` 版本： `1.13.0 ≤ K8s version < 1.16`；
> - `Helm`，版本 `>= 2.10.0`（不支持 helm 2.16.0 [#6894](https://github.com/helm/helm/issues/6894)），且已安装了 Tiller，参考 [如何安装与配置 Helm](https://devopscube.com/install-configure-helm-kubernetes/)；
> - 集群的可用 CPU > 1 C，可用内存 > 2 G；
> - 集群已有存储类型（StorageClass），若还没有存储类型可参考 [安装 OpenEBS 创建 LocalPV 存储类型](../../appendix/install-openebs)；
> - 集群能够访问外网。

可参考 [前提条件](../prerequisites) 验证，若待安装的环境满足以上条件则可以开始部署 KubeSphere。

## 视频教程

**视频教程以 QingCloud 云平台先安装 K8s 集群和 StorageClass 来准备安装示例的前提条件（即 K8s 集群），然后再在 K8s 集群一键安装 KubeSphere，与其它云平台安装步骤类似，具体以其官方文档为准。**

<video controls="controls" style="width: 100% !important; height: auto !important;">
  <source type="video/mp4" src="https://kubesphere-docs.pek3b.qingstor.com/website/%E5%85%A5%E9%97%A8%E6%95%99%E7%A8%8B/KSInstall_200P004C202002_install-kubesphere-on-k8s.mp4">
</video>

## 最小化安装 KubeSphere

1. 最小化安装仅需要一条命令，即可安装在 Kubernetes 之上。

```yaml
$ kubectl apply -f https://raw.githubusercontent.com/kubesphere/ks-installer/master/kubesphere-minimal.yaml
```

2. 查看安装日志，等待安装成功。

```bash
$ kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
```

3. 通过 `kubectl get pod --all-namespaces` 查看 kubesphere 的 namespace 下所有 Pod 状态是否为 Running。确认 Pod 都正常运行后，可使用 `IP:30880` 访问 KubeSphere UI 界面，默认的集群管理员账号为 `admin/P@88w0rd`。

![](https://pek3b.qingstor.com/kubesphere-docs/png/20191020153911.png)


## 完整安装指南

完整安装以及参数释义，请参考 [ks-installer GitHub](https://github.com/kubesphere/ks-installer/tree/master)。
