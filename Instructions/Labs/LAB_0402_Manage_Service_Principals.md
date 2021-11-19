---
lab:
    title: '랩: Azure Stack Hub에서 서비스 주체 관리'
    module: '모듈 4: ID 및 액세스 관리'
---

# 랩 - Azure Stack Hub에서 서비스 주체 관리
# 학생 랩 매뉴얼

## 랩 종속성

- 없음

## 예상 소요 시간

30분

## 랩 시나리오

여러분은 Azure Stack Hub 환경의 운영자입니다. 내부에서 개발한 애플리케이션을 사용하여 Azure Stack Hub를 관리하려고 합니다. 애플리케이션이 인증을 할 수 있도록 서비스 주체를 만들어 기본 공급자 구독의 Contributor 역할에 할당해야 합니다.

>**참고**: Azure Resource Manager를 통해 리소스를 배포하거나 구성해야 하는 애플리케이션은 자체 ID로 표시되어야 합니다. 사용자가 보안 주체인 사용자 계정으로 표시되는 것처럼, 앱은 서비스 주체로 표시됩니다. 서비스 주체는 앱용 ID를 제공하므로 앱에 필요한 권한만 위임할 수 있습니다.

앱은 인증 중에 자격 증명을 제공해야 합니다. 이 인증 과정에서는 두 가지 요소가 사용됩니다.

- 애플리케이션 ID - 클라이언트 ID라고도 합니다. GUID - Active Directory 테넌트에서 앱 등록을 고유하게 식별하는 데 사용됩니다.
- 애플리케이션 ID와 연결된 비밀. 클라이언트 암호 문자열(암호에 해당함)을 생성할 수도 있고 X509 인증서를 지정할 수도 있습니다.

이 랩에서는 암호를 사용합니다. 인증서를 사용하는 인증 관련 세부 정보는 [앱 ID를 사용하여 Azure Stack Hub 리소스 액세스](https://docs.microsoft.com/ko-kr/azure-stack/operator/azure-stack-create-service-principals?view=azs-2008&tabs=az1%2Caz2&pivots=state-disconnected)를 참조하세요.

## 목표

이 랩을 완료하면 다음을 수행할 수 있습니다.

- Azure Stack Hub AD FS 통합 시나리오에서 서비스 주체 만들기 및 관리

## 랩 환경 

이 랩에서는 AD FS(Active Directory Federation Services)와 통합된 ASDK 인스턴스(ID 공급자로 백업된 Active Directory)를 사용합니다. 

랩 환경에는 다음과 같은 구성 요소가 포함됩니다.

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

이 랩에서 설명하는 방식은 Azure Stack Hub AD FS 통합 시나리오용입니다. Azure Active Directory(Azure AD)를 ID 공급자로 사용하는 경우 서비스 주체 기반 인증을 구현하는 방식 관련 세부 정보는 [앱 ID를 사용하여 Azure Stack Hub 리소스 액세스](https://docs.microsoft.com/ko-kr/azure-stack/operator/azure-stack-create-service-principals?view=azs-2008&tabs=az1%2Caz2&pivots=state-connected)를 참조하세요.


### 연습 1: Azure Stack Hub에서 서비스 주체 만들기 및 구성

이 연습에서는 권한 있는 엔드포인트로의 PowerShell 원격 세션을 설정하여 서비스 주체를 만든 다음 Azure Stack Hub 관리자 포털을 사용하여 해당 서비스 주체에 Contributor 역할을 할당합니다. 이 연습에서는 다음 작업을 수행합니다.

1. 서비스 주체 만들기
1. 서비스 주체에 Contributor 역할 할당


#### 작업 1: 서비스 주체 만들기

이 작업에서는 다음을 수행합니다.

- PowerShell을 통해 권한 있는 엔드포인트에 연결하여 서비스 주체 만들기

1. 필요한 경우 다음 자격 증명을 사용하여 **AzS-HOST1**에 로그인합니다.

    - 사용자 이름: **AzureStackAdmin@azurestack.local**
    - 암호: **Pa55w.rd1234**

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 관리자로 PowerShell 7을 시작합니다.
1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 권한 있는 엔드포인트에 대한 PowerShell 원격 세션을 설정합니다.

    ```powershell
    $session = New-PSSession -ComputerName AzS-ERCS01 -ConfigurationName PrivilegedEndpoint
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 새 앱 등록(및 서비스 주체 개체)을 만든 다음 **$spObject** 변수에 앱 등록의 참조를 저장합니다.

    ```powershell
    $spObject = Invoke-Command -Session $session -ScriptBlock {New-GraphApplication -Name 'azsmgmt-app1' -GenerateClientSecret}
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 Azure Stack Hub 스탬프 정보를 검색한 다음 **$azureStackInfo** 변수에 해당 스탬프의 참조를 저장합니다.

    ```powershell
    $azureStackInfo = Invoke-Command -Session $session -ScriptBlock {Get-AzureStackStampInformation}
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 권한 있는 엔드포인트에 대한 PowerShell 원격 세션을 종료합니다.

    ```powershell
    $session | Remove-PSSession
    ```

    >**참고**: 일반적으로는 **Close-PrivilegedEndpoint** cmdlet을 사용하여 권한 있는 엔드포인트 세션을 닫아야 합니다. 하지만 이 랩에서는 이 방식을 따르지 않습니다. 세션 기록 로그를 호스트하는 파일 공유를 설정할 필요 없이 작업을 간편하게 수행할 수 있도록 하기 위해서입니다.

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 이 작업 앞부분에서 검색한 Azure Stack Hub 스탬프 정보를 사용해 서비스 주체를 구성하는 데 사용할 변수 값을 설정합니다. 이때 각 변수가 Azure Resource Manager 사용자 작업에 사용되는 Azure Stack Hub 엔드포인트, Graph API에 액세스하는 데 사용되는 OAuth 토큰을 가져올 대상 그룹, 그리고 ID 공급자의 GUID를 참조하도록 값을 설정해야 합니다.

    ```powershell
    $armUseEndpoint = $azureStackInfo.TenantExternalEndpoints.TenantResourceManager
    $graphAudience = "https://graph." + $azureStackInfo.ExternalDomainFQDN + "/"
    $tenantID = $azureStackInfo.AADTenantID
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 Azure Stack Hub 사용자 환경을 등록 및 설정합니다.

    ```powershell
    Add-AzEnvironment -Name 'AzureStackUser' -ArmEndpoint $armUseEndpoint
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 AzureStackUser 환경에 서비스 주체로 로그인합니다.

    ```powershell
    $securePassword = $spObject.ClientSecret | ConvertTo-SecureString -AsPlainText -Force
    $credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $spObject.ClientId, $securePassword
    $spUserSignIn = Connect-AzAccount -Environment 'AzureStackUser' -ServicePrincipal -Credential $credential -TenantId $tenantID
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 정상적으로 로그인되었는지 확인합니다.

    ```powershell
    $spUserSignIn
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 현재 인증 컨텍스트를 제거합니다.

    ```powershell
    Remove-AzAccount -Username $credential.UserName
    ```

    >**참고**: 이제 위 단계를 같은 순서로 반복하여 Azure Stack Hub 관리자 환경에 인증할 수 있는지 유효성을 검사합니다.

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 이 작업 앞부분에서 검색한 Azure Stack Hub 스탬프 정보를 사용해 서비스 주체를 구성하는 데 사용할 변수 값을 설정합니다. 이때 각 변수가 Azure Resource Manager 관리 작업에 사용되는 Azure Stack Hub 엔드포인트, Graph API에 액세스하는 데 사용되는 OAuth 토큰을 가져올 대상 그룹, 그리고 ID 공급자의 GUID를 참조하도록 값을 설정해야 합니다.

    ```powershell
    $armAdminEndpoint = $azureStackInfo.AdminExternalEndpoints.AdminResourceManager
    $graphAudience = "https://graph." + $azureStackInfo.ExternalDomainFQDN + "/"
    $tenantID = $azureStackInfo.AADTenantID
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 Azure Stack Hub 관리자 환경을 등록 및 설정합니다.

    ```powershell
    Add-AzEnvironment -Name 'AzureStackAdmin' -ArmEndpoint $armAdminEndpoint
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 AzureStackAdmin 환경에 서비스 주체로 로그인합니다.

    ```powershell
    $spAdminSignIn = Connect-AzAccount -Environment 'AzureStackAdmin' -ServicePrincipal -Credential $credential -TenantId $tenantID
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 정상적으로 로그인되었는지 확인합니다.

    ```powershell
    $spAdminSignIn
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 새 서비스 주체의 속성을 표시합니다.

    ```powershell
    $spObject
    ```

    >**참고**: 출력은 다음과 같은 형식이어야 합니다.

    ```
    ApplicationIdentifier : S-1-5-21-2657257302-3827180852-1812683747-1510
    ClientId              : 6a3fb4ad-838a-47b1-a93c-f3e4a1b683c8
    Thumbprint            :
    ApplicationName       : Azurestack-azsmgmt-app2-53508ec5-d7d9-4761-876a-3602542c2965
    ClientSecret          : 3fKPtUg37YraCk1IaFtdqeyTpVplXDqc25Dj3bUs
    PSComputerName        : AzS-ERCS01
    RunspaceId            : 6b142339-b67f-490e-a258-40983c0cd8ea
    ```

    >**참고**: **ApplicationName** 속성의 값을 적어 둡니다. 다음 작업에서 해당 값이 필요합니다. 또한 **ClientSecret** 속성도 별도로 기록하여 관리 애플리케이션을 구현하는 개발자에게 제공해야 합니다.


#### 작업 2: 서비스 주체에 Contributor 역할 할당

이 작업에서는 다음을 수행합니다.

- Azure Stack Hub 관리자 포털을 사용하여 서비스 주체에 Contributor 역할 할당

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 [Azure Stack Hub 관리자 포털](https://adminportal.local.azurestack.external/)이 표시된 웹 브라우저 창을 열고 CloudAdmin@azurestack.local로 로그인합니다.
1. Azure Stack Hub 관리자 포털이 표시된 웹 브라우저 창의 허브 메뉴에서 **모든 서비스**를 선택합니다. 
1. **모든 서비스** 블레이드에서 **일반**을 선택하고 서비스 목록에서 **구독**을 선택합니다.
1. **구독** 블레이드에서 **기본 공급자 구독**을 선택합니다.
1. **기본 공급자 구독** 블레이드에서 **IAM(액세스 제어)** 를 선택합니다.
1. **기본 공급자 구독 | 액세스 제어(IAM)** 블레이드에서 **+ 추가**를 클릭하고 드롭다운 메뉴에서 **역할 할당 추가**를 선택합니다.
1. **역할 할당 추가** 블레이드에서 다음 설정을 지정하고 **저장**을 클릭합니다.

    - 역할: **Contributor**
    - 다음에 대한 액세스 할당: **Azure AD 사용자, 그룹 또는 서비스 주체**
    - 선택 사항: 이전 작업에서 확인했던 서비스 주체의 **ApplicationName** 속성 값을 검색하여 선택합니다.

1. 역할이 정상적으로 할당되었음을 확인합니다.

>**검토**: 이 연습에서는 서비스 주체를 만들고 Contributor 역할을 할당했습니다. 
