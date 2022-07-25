---
lab:
  title: 实验室：在 Azure Stack Hub 中管理公共 IP 地址
  module: 'Module 5: Manage Infrastructure'
ms.openlocfilehash: fd9c6f35024c4753fe27d86cef8fd47dd48764f3
ms.sourcegitcommit: fd0b6231a00e8c86b46d914b2b6c4d984bc19902
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/02/2022
ms.locfileid: "139256867"
---
# <a name="lab---manage-public-ip-addresses-in-azure-stack-hub"></a>实验室 - 在 Azure Stack Hub 中管理公共 IP 地址
# <a name="student-lab-manual"></a>学生实验室手册

## <a name="lab-dependencies"></a>实验室依赖项

- 无

## <a name="estimated-time"></a>预计用时

30 分钟

## <a name="lab-scenario"></a>实验室方案

你是 Azure Stack Hub 环境的操作员。 你需要管理公共 IP 地址资源。 

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 管理公共 IP 地址资源

## <a name="lab-environment"></a>实验室环境

本实验室使用与 Active Directory 联合身份验证服务 (AD FS) 集成的 ADSK 实例（将 Active Directory 备份为标识提供者）。 

实验室环境由以下部分组成：

- 在具有以下接入点的 AzS-HOST1 服务器上运行的 ASDK 部署：

  - 管理员门户： https://adminportal.local.azurestack.external
  - 管理员 ARM 终结点： https://adminmanagement.local.azurestack.external
  - 用户门户： https://portal.local.azurestack.external
  - 用户 ARM 终结点： https://management.local.azurestack.external

- 管理用户：

  - ASDK 云操作员用户名：CloudAdmin@azurestack.local
  - ASDK 云操作员密码：Pa55w.rd1234
  - ASDK 主机管理员用户名：AzureStackAdmin@azurestack.local
  - ASDK 主机管理员密码：Pa55w.rd1234

在本实验室课程中，你将安装通过 PowerShell 管理 Azure Stack Hub 所需的软件。 你还将创建其他用户帐户。

## <a name="instructions"></a>Instructions

### <a name="exercise-1-create-an-offer-as-a-cloud-operator"></a>练习 1：创建套餐（以云操作员的身份）

在本练习中，你将充当云操作员。 首先，你将查看公共 IP 地址的使用情况，然后创建一个包含网络服务的计划和一个包含该计划的套餐。 接下来，你将公开此套餐，允许用户基于此套餐创建订阅。 该练习由以下任务组成：

1. 查看公共 IP 地址的使用情况（以云操作员的身份）
1. 创建一个包含网络服务的计划（以云操作员的身份）。
1. 创建一个基于该计划的套餐并将其公开（以云操作员的身份）


#### <a name="task-1-review-public-ip-address-usage-as-a-cloud-operator"></a>任务 1：查看公共 IP 地址的使用情况（以云操作员的身份）

在此任务中，你将：

- 查看公共 IP 地址的使用情况（以云操作员的身份）。

1. 在与 AzSHOST-1 的远程桌面会话中，打开显示 [Azure Stack Hub 管理员门户](https://adminportal.local.azurestack.external/)的 Web 浏览器窗口，并使用 CloudAdmin@azurestack.local 登录。
1. 在 Azure Stack Hub 管理员门户的“仪表板”页面上的“资源提供程序”磁贴中，单击“网络”  。 
1. 在“网络”边栏选项卡上，记下“公共 IP 池使用情况”图以及已使用和可用 IP 地址的数量 。

#### <a name="task-2-create-a-plan-consisting-of-the-network-services-as-a-cloud-operator"></a>任务 2：创建一个包含网络服务的计划（以云操作员的身份）

在此任务中，你将：

- 创建一个包含网络服务的计划（以云操作员的身份）。

1. 在显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口中，单击“+ 创建资源”。 
1. 在“新建”边栏选项卡上，单击“套餐 + 计划” 。
1. 在“套餐 + 计划”边栏选项卡上，单击“计划” 。
1. 在“新建计划”边栏选项卡的“基本信息”选项卡上，指定以下设置 ：

    - 显示名称：Network-plan1
    - 资源名称：network-plan1
    - 资源组：新资源组名称 network-plans-RG

1. 单击“下一步:服务 >”。
1. 在“新建计划”边栏选项卡的“服务”选项卡上，选中“Microsoft.Network”复选框  。
1. 单击“下一步:配额 >”。
1. 在“新建计划”边栏选项卡的“配额”选项卡上，选择“新建”  。
1. 在“创建网络配额”边栏选项卡上，指定以下设置，然后单击“确定” ：

    - 名称：Network-plan1-quota
    - 虚拟网络：**2**
    - 虚拟网络网关：**2**
    - 虚拟网络连接：**2**
    - 公共 IP：20
    - NIC：20
    - 负载均衡器：5
    - 网络安全组：20

1. 依次单击“查看 + 创建”、“创建”。 

    >备注：请等待部署完成。 这应该只需要几秒钟时间。


#### <a name="task-3-create-an-offer-based-on-the-plan-as-a-cloud-operator"></a>任务 3：根据该计划创建一个套餐（以云操作员的身份）

在此任务中，你将：

- 根据该计划创建一个套餐（以云操作员的身份）

1. 在 Azure Stack Hub 管理员门户的主菜单中，单击“+ 创建资源”。 
1. 在“新建”边栏选项卡上，单击“套餐 + 计划” 。
1. 在“套餐 + 计划”边栏选项卡上，单击“套餐” 。
1. 在“新建套餐”边栏选项卡的“基本信息”选项卡中，指定以下设置 ：

    - 显示名称：Network-offer1
    - 资源名称：network-offer1
    - 资源组：名为 network-offers-RG 的新资源组
    - 公开提供此套餐：**是**

1. 单击“下一步:基本计划 >”。 
1. 在“新建套餐”边栏选项卡的“基本计划”选项卡上，选中“Network-plan1”条目旁边的复选框  。
1. 单击“下一步:附加产品计划 >”。
1. 保留“附加产品计划”设置为默认值，单击“查看 + 创建”，然后单击“创建”  。

    >备注：请等待部署完成。 这应该只需要几秒钟时间。

>回顾：在本练习中，你创建了一个计划，并基于该计划创建了一个公共套餐。


### <a name="exercise-2-create-public-ip-address-resources-as-a-user"></a>练习 2：创建公共 IP 地址资源（以用户的身份）

在本练习中，你将充当注册了在第一个练习中创建的套餐，创建了新订阅并在该订阅中创建了公共 IP 地址资源的用户。 该练习由以下任务组成：

1. 注册套餐（以用户的身份）
1. 创建 IP 地址资源（以用户的身份）

#### <a name="task-1-sign-up-for-the-offer-as-a-user"></a>任务 1：注册套餐（以用户的身份）

在此任务中，你将：

- 注册套餐（以用户的身份）

1. 1. 在与 AzS-HOST1 的远程桌面会话中，启动 Web 浏览器的 InPrivate 会话。
1. 在 Web 浏览器窗口中，导航到 [Azure Stack Hub 用户门户](https://portal.local.azurestack.external)并使用 t1u1@azurestack.local 和密码 Pa55w.rd 登录 。
1. 在 Azure Stack Hub 用户门户中，单击“仪表板”上的“获取订阅”。
1. 在“获取订阅”边栏选项卡的“名称”文本框中，键入“T1U1-network-subscription1”  。
1. 在套餐列表中，选择“Network-offer1”，然后单击“创建” 。
1. 在“订阅已创建。必须刷新门户才能开始使用订阅”时，单击“刷新”。

#### <a name="task-2-create-a-public-ip-address-as-a-user"></a>任务 2：创建公共 IP 地址（以用户的身份）

在此任务中，你将：

- 使用 Azure Stack Hub 用户门户创建公共 IP 地址（以用户的身份）

1. 在 Azure Stack Hub 用户门户中，单击“仪表板”上的“+ 创建资源”。
1. 在“市场”边栏选项卡上，选择“公共 IP 地址”，然后在“公共 IP 地址”边栏选项卡上，选择“创建”   。
1. 在“创建公共 IP 地址”边栏选项卡上，指定以下设置并选择“创建”（将所有其他设置保留为默认值） ：

    - 名称：t1-pip1
    - IP 地址分配：**动态**
    - 空闲超时（分钟）：**4**
    - DNS 名称标签：t1-pip1
    - 订阅：T1U1-network-subscription1
    - 资源组：新资源组，名为 publicIPs-RG
    - 位置：本地

1. 等待 IP 地址资源预配完成。

>回顾：完成本练习后，你已在用户订阅中创建了公共 IP 地址资源。

### <a name="exercise-3-manage-public-ip-address-usage-as-a-cloud-operator"></a>练习 3：管理公共 IP 地址的使用情况（以云操作员的身份）

在本练习中，你将充当云操作员，查看和管理公共 IP 地址的使用情况。 该练习由以下任务组成：

1. 查看公共 IP 地址的使用情况
2. 添加公共 IP 地址池

#### <a name="task-1-review-public-ip-address-usage-as-a-cloud-operator"></a>任务 1：查看公共 IP 地址的使用情况（以云操作员的身份）

在此任务中，你将：

- 查看公共 IP 地址的使用情况（以云操作员的身份）。

1. 切换到显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口，以 CloudAdmin@azurestack.local 身份登录。
1. 在 Azure Stack Hub 管理员门户的主菜单中，单击“仪表板”，然后在“资源提供程序”磁贴上，单击“网络”  。
1. 在“网络”边栏选项卡上，再次查看“公共 IP 池使用情况”图以及已使用和可用 IP 地址的数量 。

    >**注意**：这些数字应该已更改，反映出你（以用户的身份）在用户订阅中创建的其他公共 IP 地址。

#### <a name="task-2-add-a-public-ip-address-pool-as-a-cloud-operator"></a>任务 2：添加公共 IP 地址池（以云操作员的身份）

在此任务中，你将：

- 添加公共 IP 地址池

1. 在显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口中，单击“网络”边栏选项卡上的“公共 IP 池使用情况”磁贴 。

    >**注意**：如果“公共 IP 池”边栏选项卡显示消息“排他操作‘启动’正在进行中 **。该操作运行过程中，将禁用添加节点和添加 IP 池操作。单击此处查看活动日志”，则需要等待“启动”操作完成，然后再进行下一步**。

1. 在“公共 IP 池”边栏选项卡中，单击“+ 添加 IP 池” 。 
1. 在“添加 IP 池”边栏选项卡上，指定以下设置并单击“添加” 。

    - 名称：公共池 1
    - 区域：本地
    - 地址范围（CIDR 块）：192.168.110.0/24

1. 等待更改生效，然后导航回“网络”边栏选项卡。
1. 查看“公共 IP 池使用情况”图，并记下可用 IP 地址的数量变化。

>回顾：在本练习中，你查看并配置了公共 IP 地址池。
