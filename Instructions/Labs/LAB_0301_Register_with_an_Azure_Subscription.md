---
lab:
    title: '랩: Azure 구독에 Azure Stack Hub 등록'
    module: '모듈 3: 데이터 센터 통합 구현'
---

# 랩 - Azure 구독에 Azure Stack Hub 등록
# 학생 랩 매뉴얼

## 랩 종속성

- 없음

## 예상 소요 시간

30분

## 랩 시나리오

여러분은 Azure Stack Hub 환경의 운영자입니다. Azure Marketplace 항목을 다운로드하고 Azure에 대한 데이터 보고를 설정하기 위해 Azure Stack Hub를 Azure 구독에 등록해야 합니다. 

## 목표

이 랩을 완료하면 다음을 수행할 수 있습니다.

- Azure 구독에 Azure Stack Hub 등록 

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


### 연습 1: Azure 구독에 Azure Stack Hub 등록

이 연습에서는 Azure 구독에 Azure Stack Hub를 등록합니다.

1. Azure Stack Hub 리소스 공급자 등록
1. Azure Stack Hub 등록 수행
1. Azure Stack Hub 등록 확인

#### 작업 1: Azure Stack Hub 리소스 공급자 등록

이 작업에서는 다음을 수행합니다.

- 대상 Azure 구독에 Azure Stack Hub 리소스 공급자를 등록합니다.

1. 필요한 경우 다음 자격 증명을 사용하여 **AzS-HOST1**에 로그인합니다.

    - 사용자 이름: **AzureStackAdmin@azurestack.local**
    - 암호: **Pa55w.rd1234**

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 관리자로 PowerShell 7을 시작합니다.
1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 Azure Stack Hub용 PowerShell Az 모듈을 설치합니다.

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Install-Module -Name Az.BootStrapper -Force -AllowPrerelease -AllowClobber
    Install-AzProfile -Profile 2019-03-01-hybrid -Force
    Install-Module -Name AzureStack -RequiredVersion 2.0.2-preview -AllowPrerelease
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 이 랩에서 사용할 Azure 구독에 인증을 합니다.

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureCloud'
    ```

1. 메시지가 표시되면 이 랩에서 사용할 Azure 구독에서 Contributor 역할이 있는 Azure Active Directory(Azure AD) 사용자의 자격 증명을 사용하여 로그인합니다.
1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 Azure Stack 리소스 공급자를 등록합니다.

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.AzureStack
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 등록이 완료되었는지 확인합니다.

    ```powershell
    Get-AzResourceProvider -ProviderNamespace Microsoft.AzureStack | Where-Object {$_.RegistrationState -eq 'Registered'}
    ```

    >**참고**: 등록이 완료될 때까지 기다립니다. 등록 상태를 확인하려면 **Get-AzResourceProvider** cmdlet을 다시 실행합니다.


#### 작업 2: Azure Stack Hub 등록 수행

이 작업에서는 다음을 수행합니다.

- Azure Stack Hub 등록 수행

1. **AzSHOST-1**에 연결된 원격 데스크톱 세션 내의 **관리자:  C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 PEP(권한 있는 엔드포인트) 세션을 설정합니다.

    ```powershell
    Enter-PSSession -ComputerName AzS-ERCS01 -ConfigurationName PrivilegedEndpoint
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창의 권한 있는 엔드포인트에 대해 설정된 PowerShell 원격 세션 내에서 다음 명령을 실행하여 Azure Stack Hub 스탬프의 요약 구성을 표시합니다.

    ```powershell
    Get-AzureStackStampInformation
    ```

1. 이전 단계에서 실행한 명령의 출력에서 **CloudId** 속성의 값을 확인하여 적어 둡니다.
1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창의 권한 있는 엔드포인트에 대해 설정된 PowerShell 원격 세션 내에서 다음 명령을 실행하여 세션을 종료합니다.

    ```powershell
    Exit-PSSession
    ```

    >**참고**: 일반적으로는 **Exit_PSSession**을 사용하여 권한 있는 엔드포인트에 대해 설정된 세션을 종료해서는 안 되며, 대신 **Close-PrivilegedEndpoint** cmdlet을 사용해야 합니다. 하지만 이 랩에서는 이 방식을 따르지 않습니다. 세션 기록 로그를 호스트하는 파일 공유를 설정할 필요 없이 작업을 간편하게 수행할 수 있도록 하기 위해서입니다.

    >**참고**: **Cloud ID** 속성의 스탬프 값은 Azure Stack Hub 관리자 포털의 **로컬 \| 속성** 블레이드에서도 확인할 수 있습니다.

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 Azure Stack 인스턴스를 대상으로 하는 Azure Stack PowerShell 환경을 등록합니다(`[cloud_ID]` 자리 표시자는 **Get-AzureStackStampInformation** cmdlet의 출력에서 확인한 **CloudId** 속성의 값으로 바꿔야 함).

    ```powershell
    Import-Module .\Registration\RegisterWithAzure.psm1
    $RegistrationName = "[cloud_ID]"
    Set-AzsRegistration `
       -PrivilegedEndpointCredential $adminCredentials `
       -PrivilegedEndpoint 'AzS-ERCS01' `
       -BillingModel 'Development' `
       -RegistrationName $RegistrationName `
       -UsageReportingEnabled:$true
    ```

1. 메시지가 표시되면 Azure 구독의 Contributor 역할이 지정된 Azure AD 사용자 계정을 사용하여 로그인합니다.

    > **참고:** 등록이 완료될 때까지 기다립니다. 등록은 20분 정도 걸릴 수 있습니다.


#### 작업 3: Azure Stack Hub 등록 확인

이 작업에서는 다음을 수행합니다.

- Azure 구독에 Azure Stack Hub가 등록되었는지 확인

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 [Azure Stack Hub 관리자 포털](https://adminportal.local.azurestack.external/)이 표시된 웹 브라우저 창을 열고 CloudAdmin@azurestack.local로 로그인합니다.
1. Azure Stack Hub 관리자 포털이 표시된 웹 브라우저의 **대시보드** 블레이드에서 **지역 관리** 타일을 선택합니다.
1. **로컬** 블레이드에서 **속성**을 선택합니다. 
1. **로컬 \| 속성** 블레이드에서 **등록 상태**가 **등록됨**으로 표시되는지 확인합니다. 

    > **참고:** 상태는 **등록됨**, **등록되지 않음** 또는 **만료됨** 중 하나일 수 있습니다.

>**검토**: 이 연습에서는 Azure 구독에 Azure Stack Hub를 등록했습니다.
