---
title: "应用仓库"
keywords: 'kubernetes, docker, helm, jenkins, istio, prometheus'
description: ''
---

KubeSphere 基于 [OpenPitrix](https://openpitrix.io) 构建了应用仓库服务，OpenPitrix 是由 [QingCloud](https://www.qingcloud.com) 主导开源的跨云应用管理平台，支持基于 Helm Chart 类型的 Kubernetes 应用。在应用仓库中，每个应用程序都是基础软件包存储库，如果要将 OpenPitrix 用于应用程序管理，则需要先创建存储库。应用程序管理器可以将包存储到 http / https 服务器、[minio 对象存储](https://docs.min.io/) 或 S3 对象存储。应用仓库是独立于 OpenPitrix 的外部存储，可以是 [minio 对象存储](https://docs.min.io/)、青云 QingCloud 的 QingStor 对象存储，也可以是 AWS 对象存储，里面存储的内容是开发者开发好的应用的配置包以及索引文件。注册好仓库后，存储的应用配置包会被自动索引成为可部署的应用。

## 准备应用仓库

[Helm 官方文档](https://helm.sh/docs/developing_charts/#the-chart-repository-guide) 已经提供了多种方式创建应用仓库，目前我们补充演示以下三种方式，包括基于本地的 Local Helm Repo 和基于 GitHub 准备应用仓库方便用户使用：

- [基于 Local Helm Repo 快速搭建应用仓库部署 Redis](../local-repo)（方便快速测试）
- [KubeSphere 官方应用仓库](../app-hosting-official)（极简操作，**推荐！**）
- [基于 GitHub 搭建自己的应用仓库](../app-hosting-github)


## 添加应用仓库

使用 `集群管理员` 账户登录 KubeSphere 管理控制台，点击左上角 **平台管理 → 平台设置**，进入列表页。

![添加应用仓库](https://pek3b.qingstor.com/kubesphere-docs/png/20190410020944.png)

1. 点击右上角 **添加应用仓库** 按钮。

2. 在弹出窗口填入应用仓库的基本信息后，点击 **验证** 按钮。

> 创建 QingStor 对象存储，详见 [QingStor 官方文档](https://docs.qingcloud.com/qingstor/guide/bucket_manage.html#%E5%88%9B%E5%BB%BA-bucket)，获取密钥详见 [Access Key](https://docs.qingcloud.com/qingstor/api/common/signature.html#%E8%8E%B7%E5%8F%96-access-key)。创建 AWS S3 对象存储获取 Access Key ID 和 Secret Access Key 详见 [官方文档](https://docs.aws.amazon.com/zh_cn/general/latest/gr/managing-aws-access-keys.html)。

- 应用仓库名称：为应用仓库起一个简洁明了的名称，便于用户浏览和搜索。
- 类型：支持 Helm Chart 类型的应用
- URL：支持以下三种协议
   - S3：支持将仓库内的应用部署到运行环境。QingStor 的 Bucket URL 是 http 开头，但是可以兼容 S3 协议，URL 按照 S3 风格 `s3.<zone-id>.qingstor.com/<bucket-name>/` 就可以使用 S3 接口访问 QingStor 服务。
   - HTTP：可读，不可写，仅支持获取该应用仓库 (对象存储) 中的应用，支持部署到运行环境，比如：`http://docs-repo.gd2.qingstor.com`，该示例包含一个 Nginx 示例应用，创建后将自动导入到平台中，可在应用模板中进行部署。
   - HTTPS：可读，不可写，仅支持获取该应用仓库 (对象存储) 中的应用，支持部署到运行环境。

- 描述信息：简单介绍应用仓库的主要特性，让用户进一步了解该应用仓库；

3. 验证通过后，点击 **确认** 按钮完成应用仓库的添加。当添加应用仓库后，KubeSphere 会自动加载此仓库下的所有应用模板。

![创建应用仓库](/repo-basic.png)

Google 有两个应用仓库可以试用，QingStor 对其中稳定的仓库做了一个 mirror (后续我们会开发商业版的应用仓库供企业使用)，用户可根据需要添加所需应用仓库：

> - QingStor Helm Repo: `https://helm-chart-repo.pek3a.qingstor.com/kubernetes-charts/`
> - Google Stable Helm Repo: `https://kubernetes-charts.storage.googleapis.com/`
> - Google Incubator Helm Repo: `https://kubernetes-charts-incubator.storage.googleapis.com/`

在企业内私有云场景下，用户可以基于 [Helm](https://helm.sh) 规范去构建自己的应用仓库，并且可以开发和上传满足企业业务需求的应用到自己的应用仓库中，然后基于 KubeSphere 完成应用的分发部署。

## 添加示例应用仓库


1、应用仓库一般仅集群管理员或拥有应用仓库权限的用户操作。以集群管理员 **cluster-admin** 登录 KubeSphere，选择 **平台管理 → 平台设置**，进入应用仓库页面，点击 「添加应用仓库」。

2、填写应用仓库的详细信息，如下添加一个 http 协议的示例仓库：`http://docs-repo.gd2.qingstor.com`，完成后点击 **确定**。

![应用仓库详细信息](/app-repo-basic.png)

![示例应用仓库](https://pek3b.qingstor.com/kubesphere-docs/png/20190428181651.png)

3、添加完成后，可以在顶部的 **应用模板** 中查看新添加仓库的应用。

![查看应用模板](/app-template-lists.png)