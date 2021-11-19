---
lab:
    title: '랩: Azure Stack Hub를 사용하여 ARM(Azure Resource Manager) 템플릿 유효성 검사'
    module: '모듈 2: 서비스 제공'
---

# 랩 - Azure Stack Hub를 사용하여 ARM(Azure Resource Manager) 템플릿 유효성 검사
# 학생 랩 매뉴얼

## 랩 종속성

- 없음

## 예상 소요 시간

30분

## 랩 시나리오

여러분은 Azure Stack Hub 환경의 운영자입니다. 기존 ARM(Azure Resource Manager) 템플릿을 사용하여 Azure Stack Hub 리소스 배포를 자동화해야 합니다. 

## 목표

이 랩을 완료하면 다음을 수행할 수 있습니다.

- Azure Stack Hub 배포에 사용 가능한지 ARM 템플릿 유효성 검사

## 랩 환경

이 랩에서는 AD FS(Active Directory Federation Services)와 통합된 ASDK 인스턴스(ID 공급자로 백업된 Active Directory)를 사용합니다. 

랩 환경의 구성은 다음과 같습니다.

- 다음 액세스 지점을 사용하여 **AzS-HOST1** 서버에서 실행되는 ASDK 배포:

  - 관리자 포털: https://adminportal.local.azurestack.external
  - 관리자 ARM 엔드포인트: https://adminmanagement.local.azurestack.external
  - 사용자 포털: https://portal.local.azurestack.external
  - 사용자 ARM 엔드포인트: https://management.local.azurestack.external

- 관리자:

  - ASDK 클라우드 운영자 사용자 이름: **CloudAdmin@azurestack.local**
  - ASDK 클라우드 운영자 암호: **Pa55w.rd1234**
  - ASDK 호스트 관리자 사용자 이름: **AzureStackAdmin@azurestack.local**
  - ASDK 호스트 관리자 암호: **Pa55w.rd1234**

이 랩을 진행하면서 PowerShell을 통해 Azure Stack Hub를 관리하는 데 필요한 소프트웨어를 설치합니다. 


### 연습 1: Azure Stack Hub를 사용하여 ARM 템플릿 유효성 검사

이 연습에서는 Azure Stack Hub를 사용하여 ARM 템플릿의 유효성을 검사합니다.

1. 클라우드 기능 파일 작성 
1. 템플릿 유효성 검사 정상 실행
1. 실패하는 템플릿 유효성 검사 실행
1. 템플릿 문제 수정


#### 작업 1: 클라우드 기능 파일 작성

이 작업에서는 다음을 수행합니다.

- 클라우드 기능 파일 작성

1. 필요한 경우 다음 자격 증명을 사용하여 **AzS-HOST1**에 로그인합니다.

    - 사용자 이름: **AzureStackAdmin@azurestack.local**
    - 암호: **Pa55w.rd1234**

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 관리자로 PowerShell 7을 시작합니다.
1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 신뢰할 수 있는 리포지토리로 PowerShell 갤러리를 구성합니다.

    ```powershell
    Set-PSRepository -Name 'PSGallery' -InstallationPolicy Trusted
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 PowerShellGet을 설치합니다.

    ```powershell
    Install-Module PowerShellGet -MinimumVersion 2.2.3 -Force
    ```

    >**참고**: 사용 중인 모듈 관련 경고 메시지는 무시하세요.

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 Azure Stack Hub용 PowerShell Az 모듈을 설치합니다.

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Install-Module -Name Az.BootStrapper -Force -AllowPrerelease -AllowClobber
    Install-AzProfile -Profile 2019-03-01-hybrid -Force
    Install-Module -Name AzureStack -RequiredVersion 2.0.2-preview -AllowPrerelease
    ```

    >**참고**: 이미 사용 가능한 명령 관련 오류 메시지는 무시하세요.

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 Azure Stack Hub 운영자 PowerShell 환경을 등록합니다.

    ```powershell
    Add-AzEnvironment -Name 'AzureStackAdmin' -ArmEndpoint 'https://adminmanagement.local.azurestack.external' `
       -AzureKeyVaultDnsSuffix adminvault.local.azurestack.external `
       -AzureKeyVaultServiceEndpointResourceId https://adminvault.local.azurestack.external

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 새로 등록한 **AzureStackAdmin** 환경에 로그인합니다.

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureStackAdmin'
    ```

1. 메시지가 표시되면 **CloudAdmin@azurestack.local** 계정으로 인증을 진행합니다.
1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 Azure Stack Tools를 다운로드합니다.

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Set-Location -Path 'C:\'
    Invoke-WebRequest https://github.com/Azure/AzureStack-Tools/archive/az.zip -OutFile az.zip
    Expand-Archive az.zip -DestinationPath . -Force
    Set-Location -Path '\AzureStack-Tools-az'
    ```

    >**참고**: 이 단계에서는 Azure Stack Hub 도구를 호스트하는 GitHub 리포지토리가 포함된 보관 파일을 로컬 컴퓨터에 복사한 다음 **C:\\AzureStack-Tools-master** 폴더로 확장합니다. 이 도구에는 Azure Stack Hub Resource Manager 템플릿 유효성 검사를 비롯한 폭넓은 기능을 제공하는 PowerShell 모듈이 포함되어 있습니다. 

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 AzureRM.CloudCapabilities PowerShell 모듈을 가져옵니다.

    ```powershell
    Import-Module .\CloudCapabilities\Az.CloudCapabilities.psm1 -Force
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 클라우드 기능 JSON 파일을 생성합니다. 

    ```powershell
    $path = 'C:\Templates'
    New-Item -Path $path -ItemType Directory -Force
    Get-AzCloudCapability -Location 'local' -OutputPath $path\AzureCloudCapabilities.Json -Verbose
    ```

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 파일 탐색기를 시작하고 **C:\\Templates** 폴더로 이동한 다음 **AzureCloudCapabilities.Json** 파일이 정상적으로 작성되었는지 확인합니다.

#### 작업 2: 템플릿 유효성 검사 정상 실행

이 작업에서는 다음을 수행합니다.

- Azure Stack Hub 빠른 시작 템플릿을 대상으로 템플릿 유효성 검사기 실행

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 웹 브라우저를 시작하고 Azure Stack Hub 빠른 시작 템플릿 리포지토리 [**MySql Server on Windows for AzureStack** 페이지](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/mysql-standalone-server-windows)로 이동합니다. 
1. **MySql Server on Windows for AzureStack** 페이지에서 **azuredeploy.json**을 클릭합니다.
1. [AzureStack-QuickStart-Templates/mysql-standalone-server-windows/azuredeploy.json](https://github.com/Azure/AzureStack-QuickStart-Templates/blob/master/mysql-standalone-server-windows/azuredeploy.json) 페이지에서 템플릿의 내용을 검토합니다.
1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창으로 전환한 후 다음 명령을 실행하여 azuredeploy.json 파일을 다운로드한 다음 **C:\\Templates** 폴더에 **sampletemplate1.json** 파일로 저장합니다.

    ```powershell
    Invoke-WebRequest -Uri 'https://raw.githubusercontent.com/Azure/AzureStack-QuickStart-Templates/master/mysql-standalone-server-windows/azuredeploy.json' -UseBasicParsing -OutFile $path\sampletemplate1.json
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 AzureRM.TemplateValidator PowerShell 모듈을 가져옵니다.

    ```powershell
    Set-Location -Path 'C:\AzureStack-Tools-az\TemplateValidator'
    Import-Module .\AzureRM.TemplateValidator.psm1 -Force
    ```
  
1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 템플릿의 유효성을 검사합니다.

    ```powershell
    Test-AzureRMTemplate `
        -TemplatePath $path `
        -TemplatePattern sampletemplate1.json `
        -CapabilitiesPath $path\AzureCloudCapabilities.json `
        -IncludeComputeCapabilities `
        -IncludeStorageCapabilities `
        -Report sampletemplate1validationreport.html `
        -Verbose
    ```

1. 유효성 검사의 출력을 검토하여 문제가 없는지 확인합니다.

    >**참고**: 출력은 다음과 같은 형식이어야 합니다.

    ```
    Validation Summary:
        Passed: 1
        NotSupported: 0
        Exception: 0
        Warning: 0
        Recommend: 0
        Total Templates: 1
    Report available at - C:\AzureStack-Tools-az\TemplateValidator\sampletemplate1validationreport.html
    ```

#### 작업 3: 실패하는 템플릿 유효성 검사 실행

이 작업에서는 다음을 수행합니다.

- Azure 빠른 시작 템플릿을 대상으로 템플릿 유효성 검사기 실행

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내의 Azure Stack 빠른 시작 템플릿 리포지토리가 표시된 웹 브라우저에서 Azure 빠른 시작 템플릿 리포지토리 [**MySQL Server 5.6 on Ubuntu VM** 페이지](https://github.com/Azure/azure-quickstart-templates/tree/master/mysql-standalone-server-ubuntu)로 이동합니다.
1. **MySQL Server 5.6 on Ubuntu VM** 페이지에서 **azuredeploy.json**을 클릭합니다.
1. [azure-quickstart-templates/mysql-standalone-server-ubuntu/azuredeploy.json](https://github.com/Azure/azure-quickstart-templates/blob/master/mysql-standalone-server-ubuntu/azuredeploy.json) 페이지에서 템플릿의 내용을 검토합니다.
1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창으로 전환한 후 다음 명령을 실행하여 azuredeploy.json 파일을 다운로드한 다음 **C:\\Templates** 폴더에 **sampletemplate2.json** 파일로 저장합니다.

    ```powershell
    Invoke-WebRequest -Uri 'https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mysql-standalone-server-ubuntu/azuredeploy.json' -UseBasicParsing -OutFile $path\sampletemplate2.json
    ```

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내의 웹 브라우저에서 [Virtual Machines](https://docs.microsoft.com/ko-kr/rest/api/compute/virtualmachines)의 REST API 참조로 이동하여 최신 Azure API 버전(이 콘텐츠 작성 시점에서는 **2020-12-01**)을 확인합니다. 
1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 메모장에서 **sampletemplate2.json** 파일을 엽니다.

    ```powershell
    notepad.exe $path\sampletemplate2.json
    ```

1. **sampletemplate2.json** 파일의 내용이 표시된 메모장 창에서 템플릿 `resources` 섹션의 가상 머신 리소스를 나타내는 섹션을 찾습니다. 이 섹션의 내용은 다음과 같은 형식입니다.

    ```json
    {
      "apiVersion": "2017-06-01",
      "type": "Microsoft.Compute/virtualMachines",
      "name": "[variables('vmName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
        "[concat('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
      ],
    ```

1. `apiVersion` 키의 값을 이 작업 앞부분에서 확인한 가상 머신의 최신 Azure REST API 버전으로 설정하고 변경 내용을 저장합니다. 파일은 계속 열어 둡니다.

    >**참고**: 파일을 변경하기 전에 원래 값을 적어 두세요. 다음 작업에서 원래 값으로 되돌려야 합니다.

    >**참고**: REST API 버전이 **2020-12-01**이라고 가정하는 경우 파일을 변경하면 다음과 같은 결과가 반환되어야 합니다.

    ```json
    {
      "apiVersion": "2020-12-01",
      "type": "Microsoft.Compute/virtualMachines",
      "name": "[variables('vmName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
        "[concat('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
      ],
    ```

    >**참고**: 여기에는 Azure Stack Hub에서 아직 지원되지 않는 구성이 의도적으로 포함되었습니다.

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 새로 수정한 템플릿의 유효성을 검사합니다.

    ```powershell
    Test-AzureRMTemplate `
        -TemplatePath $path `
        -TemplatePattern sampletemplate2.json `
        -CapabilitiesPath $path\AzureCloudCapabilities.json `
        -IncludeComputeCapabilities `
        -IncludeStorageCapabilities `
        -Report sampletemplate2validationreport.html `
        -Verbose
    ```

1. 유효성 검사의 출력을 검토하여 이번에는 지원 문제가 발생했음을 확인합니다.

    >**참고**: 출력은 다음과 같은 형식이어야 합니다.

    ```
    Validation Summary:
        Passed: 0
        NotSupported: 1
        Exception: 0
        Warning: 0
        Recommend: 0
        Total Templates: 1
    Report available at - C:\AzureStack-Tools-az\TemplateValidator\sampletemplate2validationreport.html
    ```

1. **C:\AzureStack-Tools-az\TemplateValidator\sampletemplate2validationreport.html** 파일을 열고 보고서를 검토합니다. 

    >**참고**: 보고서에는 다음 형식의 항목이 포함되어 있어야 합니다. **NotSupported: apiversion (Resource type: Microsoft.Compute/virtualMachines). Not Supported Values - 2020-12-01**.


#### 작업 4: 템플릿 문제 수정

이 작업에서는 다음을 수행합니다.

- 템플릿 문제 수정

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내의 파일 탐색기에서 **C:\\Templates** 폴더로 이동한 다음 **AzureCloudCapabilities.json** 파일을 엽니다.
1. **AzureCloudCapabilities.json** 파일에서 `"ResourceTypeName": "virtualMachines",` 섹션을 찾습니다. 이 섹션의 내용은 다음과 같은 형식입니다.

    ```json
    {
      "ProviderNamespace": "Microsoft.Compute",
      "ResourceTypeName": "virtualMachines",
      "Locations": [
        "local"
      ],
      "ApiVersions": [
        "2020-06-01",
        "2019-12-01",
        "2019-07-01",
        "2019-03-01",
        "2018-10-01",
        "2018-06-01",
        "2018-04-01",
        "2017-12-01",
        "2017-06-01",
        "2016-08-30",
        "2016-03-30",
        "2015-11-01",
        "2015-06-15"
      ],
      "ApiProfiles": [
        "2017-03-09-profile",
        "2018-03-01-hybrid"
      ]
    },
    ```

1. **sampletemplate2.json** 파일로 전환하여 이전 작업에서 수정했던 REST API 버전의 값을 원래 값으로 변경합니다. 이 버전이 위에 나와 있는 **AzureCloudCapabilities.json**의 API 버전 중 하나와 일치하는지 확인하고 변경 내용을 저장합니다.
1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 새로 수정한 템플릿의 유효성을 검사합니다.

    ```powershell
    Test-AzureRMTemplate `
        -TemplatePath $path `
        -TemplatePattern sampletemplate2.json `
        -CapabilitiesPath $path\AzureCloudCapabilities.json `
        -IncludeComputeCapabilities `
        -IncludeStorageCapabilities `
        -Report sampletemplate2validationreport.html `
        -Verbose
    ```

1. 유효성 검사의 출력을 검토하여 이번에는 문제가 없음을 확인합니다.

>**검토**: 이 연습에서는 클라우드 기능 파일을 만든 후 Azure Resource Manager 템플릿의 유효성을 검사하는 데 사용했습니다. 그리고 유효성 검사 결과를 기반으로 템플릿을 수정했습니다.
