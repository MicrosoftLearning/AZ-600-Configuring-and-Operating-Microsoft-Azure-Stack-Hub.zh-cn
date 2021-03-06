---
lab:
  title: 实验室：在 Azure Stack Hub 中委托套餐管理
  module: 'Module 4: Manage Identity and Access'
---

# <a name="lab---delegate-offer-management-in-azure-stack-hub"></a>实验室 - 在 Azure Stack Hub 中委托套餐管理
# <a name="student-lab-manual"></a>学生实验室手册

## <a name="lab-dependencies"></a>实验室依赖项

- 无

## <a name="estimated-time"></a>预计用时

45 分钟

## <a name="lab-scenario"></a>实验室方案

你是 Azure Stack Hub 环境的操作员。 你需要将套餐管理委托给内部技术支持人员。

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 委托 Azure Stack Hub 套餐。

## <a name="lab-environment"></a>实验室环境 

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

你将在本实验室过程中创建其他用户帐户。

### <a name="exercise-0-prepare-for-the-lab"></a>练习 0：实验室准备

在本练习中，你将创建将在本实验室中使用的 Active Directory 用户帐户：

1. 创建委派管理员用户帐户（以云操作员的身份）
1. 创建租户用户帐户（以云操作员的身份）


#### <a name="task-1-create-a-delegated-admin-user-account-as-a-cloud-operator"></a>任务 1：创建委派管理员用户帐户（以云操作员的身份）

在此任务中，你将：

- 创建委派管理员用户帐户（以云操作员的身份）

1. 如果需要，请使用以下凭据登录到 AzS-HOST1：

    - 用户名： **AzureStackAdmin@azurestack.local**
    - 密码：Pa55w.rd1234

1. 在与 AzS-HOST1 的远程桌面会话中，单击“开始”，在“开始”菜单中，单击“Windows 管理工具”，然后在管理工具列表中，双击“Active Directory 管理中心”   。
1. 在“Active Directory 管理中心”控制台中，单击“azurestack(本地)” 。
1. 在“详细信息”窗格中，双击“用户”容器。
1. 在“任务”窗格的“用户”部分中，单击“新建”->“用户”  。
1. 在“创建用户”窗口中，指定以下设置，然后单击“确定” ： 

    - 全名：DP1
    - 用户 UPN 登录名：dp1@azurestack.local
    - 用户 SamAccountName：azurestack\dp1
    - 密码：Pa55w.rd
    - 密码选项：“其他密码选项”->“密码永不过期”

1. 重复步骤 5-6，使用以下设置创建另一个用户帐户： 

    - 全名：T1U1
    - 用户 UPN 登录名：t1u1@azurestack.local
    - 用户 SamAccountName：azurestack\t1u1
    - 密码：Pa55w.rd
    - 密码选项：“其他密码选项”->“密码永不过期”

### <a name="exercise-1-designate-the-delegated-provider-as-a-cloud-operator"></a>练习 1：指定委托的提供商（以云操作员的身份）

在本练习中，你将充当云操作员，并创建一个包含订阅服务的计划，以及一个相应的套餐。 接下来，你将创建包含新套餐的订阅，并将委托的提供商指定为其订阅者：

1. 创建一个包含订阅服务的计划（以云操作员的身份）
1. 根据该计划创建一个套餐（以云操作员的身份）
1. 创建一个包含套餐的新订阅（以委托的提供商的身份）


#### <a name="task-1-create-a-plan-consisting-of-the-subscription-service-as-a-cloud-operator"></a>任务 1：创建一个包含订阅服务的计划（以云操作员的身份）

在此任务中，你将：

- 创建一个包含订阅服务的计划（以云操作员的身份）

1. 在与 AzS-HOST1 的远程桌面会话中，打开显示 [Azure Stack Hub 管理员门户](https://adminportal.local.azurestack.external/)的 Web 浏览器窗口，并使用 CloudAdmin@azurestack.local 登录。
1. 在显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口中，单击“+ 创建资源”。 
1. 在“新建”边栏选项卡上，单击“套餐 + 计划” 。
1. 在“套餐 + 计划”边栏选项卡上，单击“计划” 。
1. 在“新建计划”边栏选项卡的“基本信息”选项卡上，指定以下设置 ：

    - 显示名称：co-subscription-plan
    - 资源名称：co-subscription-plan
    - 资源组：新资源组名称：co-subscription-RG

1. 单击“下一步:服务 >”。
1. 在“新建计划”边栏选项卡的“服务”选项卡上，选中“Microsoft.Subscriptions”复选框  。
1. 单击“下一步:配额 >”。
1. 在“新建计划”边栏选项卡的“配额”选项卡上的“Microsoft.Subscriptions”下拉列表中，选择“delegatedProviderQuota”   。 
1. 依次单击“查看 + 创建”、“创建”。 

    >备注：请等待部署完成。 这应该只需要几秒钟时间。

#### <a name="task-2-create-an-offer-based-on-the-plan-as-a-cloud-operator"></a>任务 2：根据该计划创建一个套餐（以云操作员的身份）

在此任务中，你将：

- 根据该计划创建一个套餐（以云操作员的身份）

1. 在与 AzS-HOST1 的远程桌面会话中，在显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口中，单击“+ 创建资源” 。 
1. 在“新建”边栏选项卡上，单击“套餐 + 计划” 。
1. 在“套餐 + 计划”边栏选项卡上，单击“套餐” 。
1. 在“新建套餐”边栏选项卡的“基本信息”选项卡中，指定以下设置 ：

    - 显示名称：cotodp-subscription-offer
    - 资源名称：cotodp-subscription-offer
    - 资源组：co-subscription-RG
    - 公开提供此套餐：**是**

1. 单击“下一步:基本计划 >”。 
1. 在“新建套餐”边栏选项卡的“基本计划”选项卡上，选中“co-subscription-plan”条目旁边的复选框  。
1. 单击“下一步:附加产品计划 >”。
1. 保留“附加产品计划”设置为默认值，单击“查看 + 创建”，然后单击“创建”  。

    >备注：请等待部署完成。 这应该只需要几秒钟时间。

#### <a name="task-3-create-a-new-subscription-containing-the-offer-and-add-the-delegated-provider-as-its-subscriber-as-a-cloud-operator"></a>任务 3：创建一个包含套餐的新订阅，并将委托的提供商添加为其订阅者（以云操作员的身份）

在此任务中，你将：

- 创建一个包含套餐的新订阅，并将委托的提供商添加为其订阅者（以云操作员的身份）

1. 在与 AzS-HOST1 的远程桌面会话中，在显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口中，单击“+ 创建资源” 。 
1. 在“新建”边栏选项卡上，单击“套餐 + 计划” 。
1. 在“套餐 + 计划”边栏选项卡上，单击“订阅” 。
1. 在“创建用户订阅”边栏选项卡上，指定以下设置：

    - 名称：dp1-subscription1
    - 用户：dp1@azurestack.local
    - 目录租户：ADFS.azurestack.local

1. 在公开套餐列表中，单击 cotodp-subscription-offer，然后单击“创建” 

    >备注：请等待部署完成。 这应该只需要几秒钟时间。

>回顾：在本练习中，你已指定委托的提供商。


### <a name="exercise-2-create-and-delegate-a-delegated-offer-to-a-delegated-provider-as-a-cloud-operator"></a>练习 2：创建委托套餐并将其委托给委托的提供商（以云操作员的身份）

在本练习中，你将充当云操作员，并创建一个包含服务的套餐，委托的提供商将为其租户提供该套餐：

1. 创建要委托的计划（以云操作员的身份）
1. 根据该计划创建一个套餐（以云操作员的身份）
1. 将套餐委托给委托的提供商（以云操作员的身份）

#### <a name="task-1-create-a-plan-to-delegate-as-a-cloud-operator"></a>任务 1：创建要委托的计划（以云操作员的身份）

在此任务中，你将：

- 创建要委托的计划（以云操作员的身份）

1. 在与 AzS-HOST1 的远程桌面会话中，在显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口中，单击“+ 创建资源” 。 
1. 在“新建”边栏选项卡上，单击“套餐 + 计划” 。
1. 在“套餐 + 计划”边栏选项卡上，单击“计划” 。
1. 在“新建计划”边栏选项卡的“基本信息”选项卡上，指定以下设置 ：

    - 显示名称：co-services1-plan
    - 资源名称：co-services1-plan
    - 资源组：新资源组名称：co-services-RG

1. 单击“下一步:服务 >”。
1. 在“新建计划”边栏选项卡的“服务”选项卡上，选中“Microsoft.Storage”复选框  。
1. 单击“下一步:配额 >”。
1. 在“新建计划”边栏选项卡的“配额”选项卡上，单击“Microsoft.Storage”下拉列表旁边的“新建”   。
1. 在“创建存储配额”边栏选项卡上，指定以下设置，然后单击“确定” ：

    - 名称：co-services1-storage-quota
    - 容量 (GB)：200
    - 存储帐户数量：**1**

1. 依次单击“查看 + 创建”、“创建”。 

    >备注：请等待部署完成。 这应该只需要几秒钟时间。


#### <a name="task-2-create-an-offer-based-on-the-plan-as-a-cloud-operator"></a>任务 2：根据该计划创建一个套餐（以云操作员的身份）

在此任务中，你将：

- 根据该计划创建一个套餐（以云操作员的身份）

1. 在与 AzS-HOST1 的远程桌面会话中，在显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口中，单击“+ 创建资源” 。 
1. 在“新建”边栏选项卡上，单击“套餐 + 计划” 。
1. 在“套餐 + 计划”边栏选项卡上，单击“套餐” 。
1. 在“新建套餐”边栏选项卡的“基本信息”选项卡中，指定以下设置 ：

    - 显示名称：cotodp-services1-offer
    - 资源名称：cotodp-services1-offer
    - 资源组：co-services-RG
    - 公开提供此套餐：**是**

1. 单击“下一步:基本计划 >”。 
1. 在“新建套餐”边栏选项卡的“基本计划”选项卡上，选中“co-services1-plan”条目旁边的复选框  。
1. 单击“下一步:附加产品计划 >”。
1. 保留“附加产品计划”设置为默认值，单击“查看 + 创建”，然后单击“创建”  。

    >备注：请等待部署完成。 这应该只需要几秒钟时间。


#### <a name="task-3-delegate-the-offer-to-a-delegated-provider-as-a-cloud-operator"></a>任务 3：将套餐委托给委托的提供商（以云操作员的身份）

在此任务中，你将：

- 将套餐委托给委托的提供商（以云操作员的身份）

1. 在与 AzS-HOST1 的远程桌面会话中，在显示 Azure Stack Hub 管理员门户的 Web 浏览器窗口中的中心菜单中，单击“套餐” 。 
1. 在“套餐”边栏选项卡上，单击“cotodp-services1-offer” 。
1. 在“cotodp-services1-offer”边栏选项卡上，单击“委托的提供商” 。
1. 在“cotodp-services1-offer \| 委托的提供商”边栏选项卡上，单击“+ 添加” 。
1. 在“委托的套餐”边栏选项卡上，指定以下内容，然后单击“委托” ：

    - 名称：cotodp-services1-offer
    - 选择委托的提供商订阅：dp1-subscription1

    >备注：请等待部署完成。 这应该只需要几秒钟时间。

>回顾：在本练习中，你创建了套餐并将其委托给委托的提供商。


### <a name="exercise-3-create-a-delegated-offer-to-a-tenant-as-a-delegated-provider"></a>练习 3：为租户创建委托套餐（以委托的提供商的身份）

在本练习中，你将充当委托的提供商，并创建租户可订阅的套餐（基于云操作员的套餐）：

1. 为租户创建委托的提供商套餐（以云操作员的身份）
1. 标识委托的提供商门户的 URL（以委托的提供商的身份）

#### <a name="task-1-create-a-delegated-provider-offer-for-a-tenant-as-a-delegated-provider"></a>任务 1：为租户创建委托的提供商套餐（以委托的提供商的身份）

在此任务中，你将：

- 根据委托的提供商的云操作员套餐为租户（以委托的提供商的身份）创建套餐

这将允许租户根据委托的提供商的套餐创建订阅

1. 在与 AzS-HOST1 的远程桌面会话中，启动 Web 浏览器的 InPrivate 会话。
1. 在 Web 浏览器窗口中，导航到 [Azure Stack Hub 用户门户](https://portal.local.azurestack.external)并使用 dp1@azurestack.local 和密码 Pa55w.rd 登录 。
1. 在 Azure Stack Hub 用户门户的中心菜单中，单击“所有服务”，然后在“所有服务”边栏选项卡上，单击“套餐”  。
1. 在“套餐”边栏选项卡上，单击“+ 添加” 。
1. 在“新建套餐”边栏选项卡上，指定以下设置：

    - 显示名称：dp1tot-services1-offer
    - 资源名称：dp1tot-services1-offer
    - 提供商订阅：dp1-subscription1
    - 资源组：新资源组名称：dp1-RG

1. 在“选择公开套餐”部分，单击“cotodp-services1-offer” 。
1. 返回到“新建套餐”边栏选项卡，单击“创建” 。

    >备注：请等待部署完成。 这应该只需要几秒钟时间。

1. 在 Azure Stack Hub 用户门户中，返回到“套餐”边栏选项卡，单击“刷新”，然后单击新创建的套餐“dp1tot-services1-offer”  。
1. 在“dp1tot-services1-offer”边栏选项卡上，单击“更改状态”，然后在下拉列表中单击“公共”  。


#### <a name="task-2-identify-the-url-of-the-delegated-provider-portal-as-a-delegated-provider"></a>任务 2：标识委托的提供商门户的 URL（以委托的提供商的身份）

在此任务中，你将：

- 标识委托的提供商门户的 URL（以委托的提供商的身份）

1. 在 Azure Stack Hub 用户门户中，使用 dp1@azurestack.local 登录后，在中心菜单中单击“所有服务”，然后在“所有服务”边栏选项卡上的服务列表中，单击“订阅”   。
1. 在“订阅”边栏选项卡上，单击“dp1-subscription1” 。
1. 在“dp1-subscription1”委托的提供商订阅边栏选项卡上，单击“属性” 。
1. 在属性边栏选项卡上，将“门户 URL”的值复制到剪贴板。 本实验室的下一个练习中需要使用该值。
1. 在 Azure Stack Hub 用户门户的右上角，单击用户头像图标，然后在下拉菜单中单击“注销”。

    >**注意**：租户需要通过委托的提供商门户 URL 订阅套餐。

>回顾：在本练习中，你创建了一个委托的提供商套餐。


### <a name="exercise-4-sign-up-for-a-delegated-providers-offer-as-a-tenant-user"></a>练习 4：注册委托的提供商套餐（以租户用户的身份）

在本练习中，你将充当租户用户，注册委托的提供商套餐并在新订阅中创建资源。 该练习由以下任务组成：

1. 注册委托的提供商套餐（以租户用户的身份）
1. 在新订阅中创建资源（以租户用户的身份）

#### <a name="task-1-sign-up-for-the-delegated-providers-offer-as-a-tenant-user"></a>任务 1：注册委托的提供商套餐（以租户用户的身份）

在此任务中，你将：

- 注册委托的套餐（以租户用户的身份）

1. 在与 AzS-HOST1 的远程桌面会话中，启动 Web 浏览器的 InPrivate 会话。
1. 在 Web 浏览器窗口中，导航到在上一个练习中确定的委托的提供商门户 URL，并使用 t1u1@azurestack.local 和密码 Pa55w.rd 登录 。

    >**注意**：在本例中，请勿使用标准 Azure Stack Hub 用户门户 URL。

1. 在 Azure Stack Hub 用户门户中，单击“获取订阅”磁贴。
1. 在“获取订阅”边栏选项卡的“名称”文本框中，键入“t1u1-subscription1”  。
1. 在“获取订阅”边栏选项卡上，选择“dp1tot-services1-offer”套餐，然后单击“创建”  。
1. 出现消息“订阅已创建 **。必须刷新门户才能开始使用订阅”后，单击“刷新”** 。

#### <a name="task-2-create-a-resource-in-the-new-subscription-as-a-tenant-user"></a>任务 2：在新订阅中创建资源（以租户用户的身份）

在此任务中，你将：

- 在新订阅中创建资源（以租户用户的身份）

1. 在与 AzS-HOST1 的远程桌面会话中，在 Azure Stack Hub 用户门户中，使用 t1u1@azurestack.local 登录后，请单击中心菜单中的“所有服务”，然后在“所有服务”边栏选项卡的服务列表中，单击“订阅”    。
1. 在“订阅”边栏选项卡上，单击“t1u1-subscription1” 。
1. 在“t1u1-subscription1”边栏选项卡上，单击“资源” 。
1. 在“t1u1-subscription1 \| 资源”边栏选项卡上，单击“+ 添加” 。
1. 在“新建”边栏选项卡上，单击“数据 + 存储”，然后在资源类型列表中，单击“存储帐户”  。
1. 在“创建存储帐户”边栏选项卡的“基本信息”选项卡中，指定以下设置 ：

    - 订阅：t1u1-subscription1
    - 资源组：新资源组名称 delegated-RG
    - 存储帐户名称：由 3 - 24 个小写字母或数字组成的唯一名称
    - 位置：本地
    - 性能：“标准”
    - 帐户类型：“存储(常规用途 v1)”
    - 复制：本地冗余存储 (LRS)

1. 在“创建存储帐户”边栏选项卡的“基本信息”选项卡上，单击“下一步:  高级 &gt;”。
1. 在“创建存储帐户”边栏选项卡的“高级”选项卡上，保留默认设置，然后单击“查看 + 创建”  。
1. 在“创建存储帐户”边栏选项卡的“查看 + 创建”选项卡上，单击“创建”  。

    >**注意**：等待存储帐户预配完成。 这大约需要一分钟。

1. 验证存储帐户是否已成功创建。

>回顾：在本练习中，你订阅了委托的提供商套餐，以这种方式创建了新订阅，并在新订阅中预预配了存储帐户。
