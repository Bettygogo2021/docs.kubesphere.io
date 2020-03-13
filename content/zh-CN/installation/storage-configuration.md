---
title: "存储配置说明"
keywords: ''
description: ''
---

目前，Installer 支持以下类型的存储作为存储服务端，为 KubeSphere 提供持久化存储 (更多的存储类型持续更新中)：

- QingCloud 云平台块存储
- QingStor NeonSAN
- Ceph RBD
- GlusterFS
- NFS
- NFS in Kubernetes (仅限 multi-node 部署测试使用)
- Local Volume (仅限 all-in-one 部署测试使用)

同时，Installer 集成了 [QingCloud 云平台块存储 CSI 插件](https://github.com/yunify/qingcloud-csi/blob/master/README_zh.md) 和 [QingStor NeonSAN CSI 插件](https://github.com/wnxn/qingstor-csi/blob/master/docs/install_in_k8s_v1.12_zh.md)，仅需在安装前简单配置即可对接 QingCloud 云平台块存储或 NeonSAN 作为存储服务，前提是需要有操作 [QingCloud 云平台](https://console.qingcloud.com/login) 资源的权限或已有 NeonSAN 服务端。Installer 也集成了 NFS、GlusterFS 或 Ceph RBD 这类存储的客户端，用户需提前准备相关的存储服务端，然后在 `vars.yml` 配置对应的参数即可对接相应的存储服务端。

Installer 对接的开源存储服务端和客户端，以及 CSI 插件，已测试过的版本如下：

| **名称** | **版本** | **参考** | 
| ----------- | --- |---|
| Ceph RBD Server | v0.94.10 |若用于测试部署可参考 [部署 Ceph 存储服务端](/docs/v2.0/zh-CN/appendix/ceph-ks-install/)，如果是正式环境搭建请参考 [Ceph 官方文档](http://docs.ceph.com/docs/master/)|
| Ceph RBD Client | v12.2.5 | 在安装 KubeSphere 前仅需在 `vars.yml` 配置相应参数即可对接其存储服务端，参考 [Ceph RBD](../storage-configuration/#ceph-rbd)|
| GlusterFS Server | v3.7.6 |若用于测试部署可参考 [部署 GlusterFS 存储服务端](/docs/v2.0/zh-CN/appendix/glusterfs-ks-install/)， 如果是正式环境搭建请参考 [Gluster 官方文档](https://www.gluster.org/install/) 或 [Gluster Docs](http://gluster.readthedocs.io/en/latest/Install-Guide/Install/) ，并且需要安装 [Heketi 管理端 (v3.0.0)](https://github.com/heketi/heketi/tree/master/docs/admin)|
|GlusterFS Client |v3.12.10|在安装 KubeSphere 前仅需在 `vars.yml` 配置相应参数即可对接其存储服务端，配置详见 [GlusterFS](../storage-configuration/#glusterfs)|
|NFS Server in Kubernetes| v1.0.9 |配置详见 [NFS Server 配置](../storage-configuration/#nfs) |
|NFS Client | v3.1.0 | 在安装 KubeSphere 前仅需在 `vars.yml` 配置相应参数即可对接其存储服务端，详见 [NFS Client](../storage-configuration/#nfs)  |
| QingCloud-CSI|v0.2.0.1|在安装 KubeSphere 前仅需在 `vars.yml` 配置相应参数，详见 [QingCloud CSI](../storage-configuration/#qingcloud-云平台块存储)|
| NeonSAN-CSI|v0.3.0|在安装 KubeSphere 前仅需在 `vars.yml` 配置相应参数，详见 [Neonsan-CSI](../storage-configuration/#qingstor-neonsan) |

> 说明：
> 集群中不可同时存在两个默认存储类型，若要指定默认存储类型前请先确保当前集群中无默认存储类型。

## 配置文件释义

准备了满足要求的存储服务端后，只需要参考以下表中的参数说明，在 `conf/vars.yml` 中，根据您存储服务端所支持的存储类型，首先在 `conf/vars.yml` 的相应部分参考示例或注释修改对应参数并保存，然后再执行安装程序，即可完成集群存储类型的配置。

以下对 `vars.yml` 存储相关的参数配置做简要说明 (参数详解请参考 [storage classes](https://kubernetes.io/docs/concepts/storage/storage-classes/) )。

> 特别注意：`vars.yml` 中默认配置了 Local 类型的存储作为集群默认的存储类型，若配置其他类型的存储，则首先需要将 Local 相关的参数配置都修改为 **false**，然后再根据您的存储服务端类型，参考如下说明修改 `vars.yml` 中对应的存储相关部分的参数配置。

### QingCloud 云平台块存储

KubeSphere 支持使用 QingCloud 云平台块存储作为平台的存储服务，如果希望体验动态分配 (Dynamic Provisioning) 方式创建存储卷，推荐使用 [QingCloud 云平台块存储](https://docs.qingcloud.com/product/storage/volume/)，平台已集成 [QingCloud-CSI](https://github.com/yunify/qingcloud-csi/blob/master/README_zh.md) 块存储插件支持对接块存储，仅需简单配置即可使用 QingCloud 云平台各种性能的块存储服务。

[QingCloud-CSI](https://github.com/yunify/qingcloud-csi/blob/master/README_zh.md) 块存储插件实现了 CSI 接口，并且支持 KubeSphere 使用 QingCloud 云平台的存储资源。块存储插件部署后，用户可创建访问模式 (Access Mode) 为 **单节点读写（ReadWriteOnce）** 的基于 QingCloud 的超高性能型 (超高性能型硬盘只能用在超高性能型主机)、性能型 (性能型硬盘只能用在性能型主机) 或容量型硬盘的存储卷并挂载至工作负载。在安装 KubeSphere 时配置 QingCloud-CSI 插件的参数说明如下。

> 注意：在 KubeSphere 集群内使用到性能型、超高性能型、企业型或基础型硬盘时，集群的主机类型也应与硬盘的类型保持一致。

|**QingCloud-CSI** | **Description**|
| --- | ---|
| qingcloud\_csi\_enabled|是否使用 QingCloud-CSI 作为持久化存储，是：true； 否：false |
| qingcloud\_csi\_is\_default\_class|是否设定为默认的存储类型， 是：true；否：false <br/> 注：系统中存在多种存储类型时，只能设定一种为默认的存储类型|
| qingcloud\_access\_key\_id ， <br> qingcloud\_secret\_access\_key|通过[QingCloud 云平台控制台](https://console.qingcloud.com/login) 的右上角账户图标选择 **API 密钥** 创建密钥获得|
|qingcloud\_zone| zone 应与 Kubernetes 集群所在区相同，CSI 插件将会操作此区的存储卷资源。例如：zone 可以设置为 sh1a（上海一区-A）、sh1b（上海1区-B）、 pek2（北京2区）、pek3a（北京3区-A）、gd1（广东1区）、gd2a（广东2区-A）、ap1（亚太1区）、ap2a（亚太2区-A）|
| qingcloud\_type | QingCloud 云平台块存储的类型，0 代表性能型硬盘，1 或 2（根据集群所在区不同而参数不同）代表容量型硬盘，3 代表超高性能型硬盘，详情见 [QingCloud 官方文档](https://docs.qingcloud.com/product/api/action/volume/create_volumes.html)|
| qingcloud\_maxSize, qingcloud\_minSize | 限制存储卷类型的存储卷容量范围，单位为 GiB|
| qingcloud\_stepSize | 设置用户所创建存储卷容量的增量，单位为 GiB|
| qingcloud\_fsType | 存储卷的文件系统，支持 ext3, ext4, xfs. 默认为 ext4|

### QingStor NeonSAN

NeonSAN-CSI 插件支持对接青云自研的企业级分布式存储 [QingStor NeonSAN](https://www.qingcloud.com/products/qingstor-neonsan/) 作为存储服务，若您准备好 NeonSAN 服务端后，即可在 `conf/vars.yml` 配置 NeonSAN-CSI 插件对接其存储服务端。详见 [NeonSAN-CSI 参数释义](https://github.com/wnxn/qingstor-csi/blob/master/docs/reference_zh.md#storageclass-%E5%8F%82%E6%95%B0)。

| **NeonSAN** | **Description** |
| --- | --- |
| neonsan\_csi\_enabled | 是否使用 NeonSAN 作为持久化存储，是：true；否：false |
| neonsan\_csi\_is\_default\_class | 是否设定为默认存储类型，是：true；否：false <br/> 注：系统中存在多种存储类型时，只能设定一种为默认存储类型 |
| neonsan\_csi\_protocol | NeonSAN 服务端传输协议，如 TCP 或 RDMA|
| neonsan\_server\_address |NeonSAN 服务端地址 |
| neonsan\_cluster\_name| NeonSAN 服务端集群名称|
| neonsan\_server\_pool|Kubernetes 插件从哪个 pool 内创建存储卷，默认值为 kube|
| neonsan\_server\_replicas|NeonSAN image 的副本个数，默认为 1|
| neonsan\_server\_stepSize|用户所创建存储卷容量的增量，单位为 GiB，默认为 1|
| neonsan\_server\_fsType|存储卷的文件系统格式，默认为 ext4|


### Ceph RBD

[Ceph RBD](https://ceph.com/) 是一个分布式存储系统，在 `conf/vars.yml` 配置的释义如下。

| **Ceph\_RBD** | **Description** |
| --- | --- |
| ceph\_rbd\_enabled | 是否使用 Ceph RBD 作为持久化存储，是：true；否：false |
| ceph\_rbd\_storage\_class | 存储类型名称 |
| ceph\_rbd\_is\_default\_class | 是否设定为默认存储类型， 是：true；否：false <br/> 注：系统中存在多种存储类型时，只能设定一种为默认存储类型 |
| ceph\_rbd\_monitors | 根据 Ceph RBD 服务端配置填写，可参考 [Kubernetes 官方文档](https://kubernetes.io/docs/concepts/storage/storage-classes/#ceph-rbd) |
| ceph\_rbd\_admin\_id | 能够在存储池中创建的客户端 ID ，默认: admin，可参考 [Kubernetes 官方文档](https://kubernetes.io/docs/concepts/storage/storage-classes/#ceph-rbd) |
| ceph\_rbd\_admin\_secret | Admin_id 的 secret，安装程序将会自动在 kube-system 项目内创建此 secret，可参考 [Kubernetes 官方文档](https://kubernetes.io/docs/concepts/storage/storage-classes/#ceph-rbd) |
| ceph\_rbd\_pool | 可使用的 Ceph RBD 存储池，可参考 [Kubernetes 官方文档](https://kubernetes.io/docs/concepts/storage/storage-classes/#ceph-rbd) |
| ceph\_rbd\_user\_id | 用于映射 RBD  的 ceph 客户端 ID 默认: admin，可参考 [Kubernetes 官方文档](https://kubernetes.io/docs/concepts/storage/storage-classes/#ceph-rbd) |
| ceph\_rbd\_user\_secret | User_id 的 secret，需注意在所使用 rbd image 的项目内都需创建此 Secret，可参考 [Kubernetes 官方文档](https://kubernetes.io/docs/concepts/storage/storage-classes/#ceph-rbd) |
| ceph\_rbd\_fsType | 存储卷的文件系统，kubernetes 支持 fsType，默认：ext4，可参考 [Kubernetes 官方文档](https://kubernetes.io/docs/concepts/storage/storage-classes/#ceph-rbd) |
| ceph\_rbd\_imageFormat | Ceph RBD  格式，默认："1"，可参考 [Kubernetes 官方文档](https://kubernetes.io/docs/concepts/storage/storage-classes/#ceph-rbd) |
|ceph\_rbd\_imageFeatures| 当 `ceph_rbd_imageFormat` 字段不为 1 时需填写此字段，可参考 [Kubernetes 官方文档](https://kubernetes.io/docs/concepts/storage/storage-classes/#ceph-rbd)|

> 注： 存储类型中创建 secret 所需 ceph secret 如 `ceph_rbd_admin_secret` 和 `ceph_rbd_user_secret` 可在 ceph 服务端通过以下命令获得：

```bash
$ ceph auth get-key client.admin
```

### GlusterFS 

[GlusterFS](https://www.gluster.org/) 是一个开源的分布式文件系统，配置时需提供 heketi 所管理的 glusterfs 集群，在 `conf/vars.yml` 配置的释义如下。

| **GlusterFS**| **Description** |
| --- | --- |
| glusterfs\_provisioner\_enabled | 是否使用 GlusterFS 作为持久化存储，是：true；否：false |
| glusterfs\_provisioner\_storage\_class | 存储类型的名称 |
| glusterfs\_is\_default\_class | 是否设定为默认存储类型，是：true；否：false <br/> 注：系统中存在多种存储类型时，只能设定一种为默认存储类型| --- | --- |glusterfs\_provisioner\_resturl | Heketi 服务 url，参数配置请参考 [Kubernetes 官方文档](https://kubernetes.io/docs/concepts/storage/storage-classes/#glusterfs) | glusterfs\_provisioner\_clusterid | Heketi 服务端输入 heketi-cli cluster list 命令获得，参数配置请参考 [Kubernetes 官方文档](https://kubernetes.io/docs/concepts/storage/storage-classes/#glusterfs) |
| glusterfs\_provisioner\_restauthenabled | Gluster 启用对 REST 服务器的认证，参数配置请参考 [Kubernetes 官方文档](https://kubernetes.io/docs/concepts/storage/storage-classes/#glusterfs) |
| glusterfs\_provisioner\_resturl | Heketi 服务端 url，参数配置请参考 [Kubernetes 官方文档](https://kubernetes.io/docs/concepts/storage/storage-classes/#glusterfs) |
| glusterfs\_provisioner\_clusterid | Gluster 集群 id，登录 heketi 服务端输入 heketi-cli cluster list 得到 Gluster 集群 id，参数配置请参考 [Kubernetes 官方文档](https://kubernetes.io/docs/concepts/storage/storage-classes/#glusterfs) |
| glusterfs\_provisioner\_restuser | 能够在 Gluster pool 中创建 volume 的 Heketi 用户，参数配置请参考 [Kubernetes 官方文档](https://kubernetes.io/docs/concepts/storage/storage-classes/#glusterfs) |
| glusterfs\_provisioner\_secretName | secret 名称，安装程序将会在 kube-system 项目内自动创建此 secret，参数配置请参考 [Kubernetes 官方文档](https://kubernetes.io/docs/concepts/storage/storage-classes/#glusterfs) |
| glusterfs\_provisioner\_gidMin | glusterfs\_provisioner\_storage\_class 中 GID 的最小值，参数配置请参考 [Kubernetes 官方文档](https://kubernetes.io/docs/concepts/storage/storage-classes/#glusterfs) |
| glusterfs\_provisioner\_gidMax | glusterfs\_provisioner\_storage\_class 中 GID 的最大值，参数配置请参考 [Kubernetes 官方文档](https://kubernetes.io/docs/concepts/storage/storage-classes/#glusterfs) |
| glusterfs\_provisioner\_volumetype | Volume 类型，参数配置请参考 [Kubernetes 官方文档](https://kubernetes.io/docs/concepts/storage/storage-classes/#glusterfs) |
| jwt\_admin\_key | heketi 服务器中 `/etc/heketi/heketi.json` 的 jwt.admin.key 字段 |

> 注： Glusterfs 存储类型中所需的 `glusterfs_provisioner_clusterid` 可在 glusterfs 服务端通过以下命令获得：

```bash
$ export HEKETI_CLI_SERVER=http://localhost:8080
$ heketi-cli cluster list
```

### NFS

> 注意：**NFS** 与 **NFS in Kubernetes** 是两种不同类型的存储类型，在 vars.yml 中配置时仅需配置其中一种作为默认的存储类型即可。

[NFS](https://kubernetes.io/docs/concepts/storage/volumes/#nfs) 即网络文件系统，它允许网络中的计算机之间通过 TCP/IP 网络共享资源。需要预先准备 NFS 服务端，本方法可以使用 QingCloud 云平台 [vNAS](https://www.qingcloud.com/products/nas/) 作为 NFS 服务端。在 `conf/vars.yml` 配置的释义如下。

关于在安装前如何配置 QingCloud vNas，本文档在 [常见问题 - 安装前如何配置 QingCloud vNas](../../faq/#安装前如何配置-qingcloud-vnas) 给出了一个详细的示例供参考。

| **NFS** | **Description** |
| --- | --- |
| nfs\_client\_enable | 是否使用 NFS 作为持久化存储，是：true；否：false |
| nfs\_client\_is\_default\_class | 是否设定为默认存储类型，是：true；否：false <br/> 注：系统中存在多种存储类型时，只能设定一种为默认存储类型 |
| nfs\_server | 允许其访问的 NFS 服务端地址，可以是 IP 或 Hostname |
| nfs\_path | NFS 共享目录，即服务器上共享出去的文件目录，可参考 [Kubernetes 官方文档](https://kubernetes.io/docs/concepts/storage/volumes/#nfs) |


### NFS in Kubernetes（仅限 multi-node 部署测试使用）

NFS 即网络文件系统，它允许网络中的计算机之间通过 TCP/IP 网络共享资源。本安装方法将会在 Kubernetes 集群内安装 [容器化的 NFS 服务端](https://github.com/helm/charts/tree/master/stable/nfs-server-provisioner)，要求 Kubernetes 节点有足够的硬盘空间。在 `conf/vars.yml` 配置的释义如下。

| **NFS** | **Description** |
| --- | --- |
| nfs\_in\_k8s\_enable | 是否部署 NFS Server 到当前集群作为存储服务端，是：true；否：false | 
|nfs\_in\_k8s\_is\_default\_class | 是否设定 NFS 为默认存储类型，是：true；否：false <br/> 注：系统中存在多种存储类型时，只能设定一种为默认存储类型 |

### Local Volume（仅限 all-in-one 部署测试使用）

[Local Volume](https://kubernetes.io/docs/concepts/storage/volumes/#local) 表示挂载的本地存储设备，如磁盘、分区或目录，仅支持在 all-in-one 模式安装时使用 Local Volume，由于该类型暂不支持动态分配，因此仅建议用于测试部署，方便初次安装和体验。在 `conf/vars.yml` 配置的释义如下。

| **Local Volume** | **Description** |
| --- | --- |
| local\_volume\_provisioner\_enabled | 是否使用 local volume 作为持久化存储，  是：true；否：false |
| local\_volume\_provisioner\_storage\_class | 存储类型的名称，   默认：local |
| local\_volume\_is\_default\_class | 是否设定为默认存储类型， 是：true；否：false <br/> 注：系统中存在多种存储类型时，只能设定一种为默认的存储类型 |

> 说明： 在您配置好 Local Volume (只有 all-in-one 支持这类存储) 并成功安装 KubeSphere 后，可参阅 [Local Volume 使用方法](../../storage/local-volume)。
