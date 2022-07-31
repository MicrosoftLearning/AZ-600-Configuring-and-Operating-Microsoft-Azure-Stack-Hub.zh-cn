---
lab:
  title: 实验室：在 Azure Stack Hub 中访问特权终结点
  module: 'Module 5: Manage Infrastructure'
---

# <a name="lab---access-the-privileged-endpoint-in-azure-stack-hub"></a>实验室 - 在 Azure Stack Hub 中访问特权终结点
# <a name="student-lab-manual"></a>学生实验室手册

## <a name="lab-dependencies"></a>实验室依赖项

- 无

## <a name="estimated-time"></a>预计用时

30 分钟

## <a name="lab-scenario"></a>实验室方案

你是 Azure Stack Hub 环境的操作员。 你需要确定访问特权终结点的方法。

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 访问 Azure Stack Hub 特权终结点 

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


### <a name="exercise-1-manage-azure-stack-hub-via-the-privileged-endpoint"></a>练习 1：通过特权终结点管理 Azure Stack Hub

在本练习中，你将建立与特权终结点的 PowerShell 远程处理会话，以运行可通过远程处理会话访问的 Windows PowerShell cmdlet。 该练习由以下任务组成：

1. 通过 Windows PowerShell 连接到特权终结点
1. 查看可通过特权终结点使用的功能
1. 关闭与特权终结点的连接并收集会话脚本

#### <a name="task-1-connect-to-the-privileged-endpoint-via-windows-powershell"></a>任务 1：通过 Windows PowerShell 连接到特权终结点

在此任务中，你将：

- 通过 Windows PowerShell 连接到特权终结点

1. 如果需要，请使用以下凭据登录到 AzS-HOST1：

    - 用户名： **AzureStackAdmin@azurestack.local**
    - 密码：Pa55w.rd1234

1. 在与 AzS-HOST1 的远程桌面会话中，以管理员身份启动 PowerShell ISE。
1. 在“管理员: Windows PowerShell ISE”控制台中运行以下命令，确定运行特权终结点的基础结构 VM 的 IP 地址： 

    ```powershell
    $ipAddress = (Resolve-DnsName -Name AzS-ERCS01).IPAddress
    ```

1. 在“Administrator: Windows PowerShell ISE”窗口运行以下命令，将运行特权终结点的基础结构 VM 的 IP 地址添加到 WinRM 受信任主机列表中（除非已允许所有主机）：

    ```powershell
    $trustedHosts = (Get-Item -Path WSMan:\localhost\Client\TrustedHosts).Value
    If ($trustedHosts -ne '*') {
    If ($trustedHosts -ne '') {
        $trustedHosts += ",ipAddress"
    } else {
    $trustedHosts = "$ipAddress"
    }
    }
    Set-Item WSMan:\localhost\Client\TrustedHosts -Value $TrustedHosts -Force
    ```

1. 在“Administrator: Windows PowerShell ISE”窗口运行以下命令，将 Azure Stack Hub 管理员凭据存储在变量中：

    ```powershell
    $adminUserName = 'CloudAdmin@azurestack.local'
    $adminPassword = 'Pa55w.rd1234' | ConvertTo-SecureString -Force -AsPlainText
    $adminCredentials = New-Object PSCredential($adminUserName,$adminPassword)
    ```

1. 在“管理员: Windows PowerShell ISE”窗口运行以下命令，建立与特权终结点的 PowerShell 远程处理会话：

    ```powershell
    Enter-PSSession -ComputerName $ipAddress -ConfigurationName PrivilegedEndpoint -Credential $adminCredentials
    ```

1. 验证是否已成功建立 PowerShell 远程处理会话。 “Windows PowerShell ISE”窗口中的“控制台”窗格应显示用方括号括起来的提示，该提示以运行特权终结点的基础结构 VM 的 IP 地址开头。


#### <a name="task-2-review-the-functionality-available-via-the-privileged-endpoint"></a>任务 2：查看可通过特权终结点使用的功能

在此任务中，你将：

- 查看可通过特权终结点使用的功能。

1. 在与 AzS-HOST1 的远程桌面会话中，从“管理员: Windows PowerShell ISE”窗口的 PowerShell 远程处理会话中的“控制台”窗格运行以下命令，识别所有可用的 PowerShell cmdlet：

    ```powershell
    Get-Command
    ```

1. 在“管理员: Windows PowerShell ISE”窗口的 PowerShell 远程处理会话中，运行以下命令来识别当前的云管理员用户帐户：

    ```powershell
    Get-CloudAdminUserList
    ```

    >**注意**：该列表应仅包含两个帐户 - CloudAdmin 和 AzureStackAdmin。

1. 在“管理员: Windows PowerShell ISE”窗口的 PowerShell 远程处理会话中，运行以下命令来验证 Azure Stack Hub 的更新准备情况并查看结果：

    ```powershell
    Test-AzureStack -Group UpdateReadiness 
    ```
    >**注意**：请记住，ASDK 不支持更新，因此这仅出于演示目的。 忽略报告的任何失败的测试。

    >**注意**：有关 Test-AzureStack 功能的简介，请参阅[验证 Azure Stack Hub 系统状态](https://docs.microsoft.com/en-us/azure-stack/operator/azure-stack-diagnostic-test?view=azs-2008)。

    >**注意**：在支持场景中，Microsoft 支持工程师可能需要提升特权终结点 PowerShell 会话才能访问 Azure Stack Hub 基础结构的内部项。 此过程称为解锁特权终结点。 PEP 会话提升过程包含两个步骤，涉及到两个人员和两个组织的身份验证。 解锁过程由 Azure Stack Hub 操作员启动，该操作员始终保持对其环境的控制。 接下来将逐步模拟场景，以说明此过程。

1. 在“管理员: Windows PowerShell ISE”窗口的 PowerShell 远程处理会话中，运行以下命令来验证特权终结点当前是否已锁定：

    ```powershell
    Get-SupportSessionInfo
    ```

1. 在“管理员: Windows PowerShell ISE”窗口的 PowerShell 远程处理会话中，运行以下命令来生成支持会话令牌：

    ```powershell
    Get-SupportSessionToken
    ```

    >**注意**：在支持场景中，你将通过 Microsoft 支持工程师选择的媒介（例如聊天、电子邮件）将请求令牌传递给他们。 然后，Microsoft 支持工程师将使用该请求令牌生成支持会话授权令牌，并将其值传递给你。 接下来，在同一 PowerShell 远程处理会话中，运行 Unlock-SupportSession cmdlet，并在出现提示时提供该支持会话授权令牌的值。 届时，PowerShell 远程处理会话将得到提升，具有完整的管理功能和对基础结构的完全可访问性。


#### <a name="task-3-close-the-session-to-the-privilege-endpoints-and-collect-the-session-transcript"></a>任务 3：关闭与特权终结点的会话并收集会话脚本

在此任务中，你将：

- 关闭与特权终结点的会话并收集会话脚本。

>**注意**：特权终结点记录每个操作及其输出。 若要收集日志，请使用 Close-PrivilegedEndpoint cmdlet 关闭会话。 此操作会关闭终结点并将日志文件传输到外部文件共享进行保留。

>**注意**：你将首先创建一个文件共享来存储特权终结点日志。

1. 在与 AzS-HOST1 的远程桌面会话中，以管理员身份启动另一个 PowerShell ISE。
1. 在“管理员: Windows PowerShell ISE”控制台运行以下命令，创建和配置将存储特权终结点会话日志的共享：

    ```powershell
    $pepGroup = 'AZURESTACK\CloudAdmins'
    New-Item -Path 'C:\PEPLogs' -ItemType Directory -Force
    $pepShare = New-SmbShare -Name 'PEPLogs' -Description 'PEPLogs' -Path 'C:\PEPLogs'
    Grant-SmbShareAccess -Name $pepShare.Name -AccountName $pepGroup -AccessRight Full -Force
    Revoke-SmbShareAccess -Name $pepShare.Name -AccountName 'Everyone' -Force
    ```

1. 切换回“管理员: Windows PowerShell ISE”窗口中的 PowerShell 远程处理会话，运行以下命令以关闭特权终结点会话，并将会话日志文件传输到外部文件共享进行保留：

    ```powershell
    Close-PrivilegedEndpoint -TranscriptsPathDestination '\\AzSHOST-1.azurestack.local\PEPLogs' -Credential $using:adminCredentials
    ```

    >**注意**：忽略任何警告消息。

1. 等待该 cmdlet 完成，然后在文件资源管理器中查看 C:\\PEPLogs 文件夹的内容。 

>回顾：在本练习中，你建立了与特权终结点的 PowerShell 远程处理会话，查看了其功能，并关闭了该会话。