---
lab:
    title: '랩: PowerShell을 통해 Azure Stack Hub에 연결'
    module: '모듈 5: 인프라 관리'
---

# 랩 - PowerShell을 통해 Azure Stack Hub에 연결
# 학생 랩 매뉴얼

## 랩 종속성

- 없음

## 예상 소요 시간

30분

## 랩 시나리오

여러분은 Azure Stack Hub 환경의 운영자입니다. PowerShell을 사용하여 환경을 관리할 수 있어야 합니다. 그리고 경우에 따라서는 사용자로 PowerShell을 통해 Azure Stack Hub에 연결할 수 있어야 합니다. 

## 목표

이 랩을 완료하면 다음을 수행할 수 있습니다.

- PowerShell을 통해 ASDK 운영자 및 사용자 환경에 연결

## 랩 환경

이 랩에서는 AD FS(Active Directory Federation Services)와 통합된 ASDK 인스턴스(ID 공급자로 백업된 Active Directory)를 사용합니다. 

>**참고**: Azure Active Directory(Azure AD)와 통합된 Azure Stack Hub에 연결하는 방법 관련 세부 정보는 [PowerShell을 사용하여 Azure Stack Hub에 연결](https://docs.microsoft.com/ko-kr/azure-stack/operator/azure-stack-powershell-configure-admin?view=azs-2008&tabs=az1%2Caz2%2Caz3#connect-with-azure-ad)을 참조하세요.

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

## 지침

### 연습 1: Azure PowerShell을 통해 ASDK에 연결

이 연습에서는 PowerShell을 통해 ASDK에서 ASDK의 관리자 ARM 엔드포인트에 연결합니다.

1. Azure Stack Hub 호환 Az PowerShell 모듈 설치
1. Azure Stack Hub 도구 다운로드
1. PowerShell을 통해 Azure Stack Hub 운영자 환경 구성 및 해당 환경에 연결
1. PowerShell을 통해 Azure Stack Hub 사용자 환경 구성 및 해당 환경에 연결

#### 작업 1: Azure Stack Hub 호환 Azure PowerShell 모듈 설치

이 작업에서는 다음을 수행합니다.

- 기존 Azure 및 Az PowerShell 모듈 제거
- Azure Stack Hub 호환 Az PowerShell 모듈용 필수 구성 요소 설치 및 구성
- Azure Stack Hub 호환 Az PowerShell 모듈 설치

>**참고**: 모든 버전의 AzureRM(Azure Resource Manager) PowerShell 모듈은 최신 버전은 아니지만 계속 지원됩니다. 그러나 이제는 Azure 및 Azure Stack Hub와 상호 작용할 때 PowerShell 모듈로 Az PowerShell 모듈을 사용하는 것이 좋습니다. 

1. 필요한 경우 다음 자격 증명을 사용하여 **AzS-HOST1**에 로그인합니다.

    - 사용자 이름: **AzureStackAdmin@azurestack.local**
    - 암호: **Pa55w.rd1234**

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 관리자로 **Windows PowerShell**을 시작합니다.
1. **관리자: Windows PowerShell** 프롬프트에서 다음 명령을 실행하여 Azure PowerShell 및 Az PowerShell 모듈의 기존 버전을 모두 제거합니다.

    ```powershell
    Get-Module -Name Azure* -ListAvailable | Uninstall-Module -Force -Verbose -ErrorAction Continue
    Get-Module -Name Azs.* -ListAvailable | Uninstall-Module -Force -Verbose -ErrorAction Continue
    Get-Module -Name Az.* -ListAvailable | Uninstall-Module -Force -Verbose -ErrorAction Continue
    ```

    >**참고**: 사용 중인 모듈 관련 오류 메시지가 표시되면 Windows PowerShell 세션을 닫았다가 다시 연 후 위의 명령을 다시 실행합니다.

1. **관리자: Windows PowerShell** 프롬프트에서 다음 명령을 실행하여 **C:\\Program Files\\WindowsPowerShell\\Modules** 및 **C:\\Users\\AzureStackAdmin\\Documents\\PowerShell\\Modules\\** 폴더에서 이름이 **Azure**, **Az** 또는 **Azs**로 시작되는 폴더를 모두 삭제합니다.

    ```powershell
    Get-ChildItem -Path 'C:\Program Files\WindowsPowerShell\Modules' -Include 'Az*' -Recurse -Force | Remove-Item -Force -Recurse
    Get-ChildItem -Path 'C:\Users\AzureStackAdmin\Documents\PowerShell\Modules' -Include 'Az*' -Recurse -Force | Remove-Item -Force -Recurse
    ```

    >**참고**: 사용 중인 모듈 관련 오류 메시지가 표시되면 Windows PowerShell 세션을 닫았다가 다시 연 후 위의 명령을 다시 실행합니다.


1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 Microsoft Edge를 시작하여 [PowerShell 릴리스 페이지](https://github.com/PowerShell/PowerShell/releases/tag/v7.1.2)로 이동합니다. 
1. [PowerShell 릴리스 페이지](https://github.com/PowerShell/PowerShell/releases/tag/v7.1.2)에서 최신 PowerShell 릴리스를 다운로드하여 설치합니다. 
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


#### 작업 2: Azure Stack Hub 도구 다운로드 

이 작업에서는 다음을 수행합니다.

- GitHub에서 Azure Stack Hub 도구 다운로드

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에서 다음 명령을 실행하여 Azure Stack Tools를 다운로드합니다.

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Set-Location -Path 'C:\'
    Invoke-WebRequest https://github.com/Azure/AzureStack-Tools/archive/az.zip -OutFile az.zip
    Expand-Archive az.zip -DestinationPath . -Force
    Set-Location -Path '\AzureStack-Tools-az'
    ```

    >**참고**: 이 단계에서는 Azure Stack Hub 도구를 호스트하는 GitHub 리포지토리가 포함된 보관 파일을 로컬 컴퓨터에 복사한 다음 **C:\\AzureStack-Tools-master** 폴더로 확장합니다. 이 도구에는 폭넓은 기능을 제공하는 PowerShell 모듈이 포함되어 있습니다. 제공되는 기능으로는 Azure Stack Hub 기능 확인, Azure Stack Hub VM 인프라와 이미지 관리, Resource Manager 정책 구성, Azure에 Azure Stack Hub 등록, Azure Stack Hub 배포, Azure Stack Hub에 연결, Azure Stack Hub 테넌트 관리, Azure Stack Hub Resource Manager 템플릿 유효성 검사 등이 있습니다. 


#### 작업 3: PowerShell을 통해 Azure Stack Hub 운영자 환경 구성 및 해당 환경에 연결

이 작업에서는 다음을 수행합니다.

- PowerShell을 통해 Azure Stack Hub 운영자 환경 구성
- PowerShell을 통해 Azure Stack Hub 운영자 환경에 연결
- PowerShell을 통해 Azure Stack Hub 운영자 환경에 대한 연결 유효성 검사

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 Azure Stack Hub 운영자 PowerShell 환경을 등록합니다.

    ```powershell
    Add-AzEnvironment -Name 'AzureStackAdmin' -ArmEndpoint 'https://adminmanagement.local.azurestack.external' `
       -AzureKeyVaultDnsSuffix adminvault.local.azurestack.external `
       -AzureKeyVaultServiceEndpointResourceId https://adminvault.local.azurestack.external
    ```

    >**참고**: 위의 명령은 다음 출력을 반환합니다.

    ```powershell
    Name            Resource Manager Url                              ActiveDirectory Authority
    ----            --------------------                              -------------------------
    AzureStackAdmin https://adminmanagement.local.azurestack.external https://adfs.local.azurestack.external/adfs/
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 AzureStack\CloudAdmin 자격 증명을 사용해 Azure Stack Hub 운영자 PowerShell 환경에 로그인합니다.

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureStackAdmin' -UseDeviceAuthentication
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에 표시되는 메시지를 검토합니다. 그런 후에 웹 브라우저 창을 열고 [adfs.local.azurestack.external](https://adfs.local.azurestack.external/adfs/oauth2/deviceauth) 페이지로 이동하여 검토한 메시지에 포함된 코드를 입력하고 **계속**을 클릭합니다. 

1. 메시지가 표시되면 다음 자격 증명을 사용하여 로그인합니다.

    - 사용자 이름: **CloudAdmin@azurestack.local**
    - 암호: **Pa55w.rd1234**

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창으로 다시 전환하여 **CloudAdmin@azurestack.local**로 정상 인증되었는지 확인합니다.
1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 Azure Stack Hub 관리자 구독 목록을 표시합니다.

    ```powershell
    Get-AzSubscription
    ```

    >**참고**: 출력에 **기본 공급자 구독**, **계량 구독** 및 **사용 구독**이 포함되어 있음을 확인합니다.

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 해당 PowerShell 환경 컨텍스트를 확인합니다.

    ```powershell
    Get-AzContext
    ```


#### 작업 4: PowerShell을 통해 Azure Stack Hub 사용자 환경 구성 및 해당 환경에 연결

이 작업에서는 다음을 수행합니다.

- PowerShell을 통해 Azure Stack Hub 사용자 환경 구성
- PowerShell을 통해 Azure Stack Hub 사용자 환경에 연결
- PowerShell을 통해 Azure Stack Hub 운영자 사용에 대한 연결 유효성 검사

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 PowerShell 7을 시작합니다.
1. **C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 Azure Stack Hub 사용자 환경을 대상으로 Azure Resource Manager 환경을 등록합니다.

    ```powershell
    Add-AzEnvironment -Name 'AzureStackUser' -ArmEndpoint 'https://management.local.azurestack.external'
    ```

    >**참고**: 위의 명령은 다음 출력을 반환합니다.

    ```powershell
    Name            Resource Manager Url                              ActiveDirectory Authority
    ----            --------------------                              -------------------------
    AzureStackUser https://management.local.azurestack.external https://adfs.local.azurestack.external/adfs/
    ```

1. **C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 Azure Stack Hub PowerShell 환경ㄹ에 로그인합니다.

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureStackUser'
    ```

    >**참고**: 그러면 웹 브라우저 창이 자동으로 열리고 인증하라는 메시지가 표시됩니다.

1. 메시지가 표시되면 다음 자격 증명을 사용하여 로그인합니다.

    - 사용자 이름: **CloudAdmin@azurestack.local**
    - 암호: **Pa55w.rd1234**

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 Azure Stack Hub 관리자 구독 목록을 표시합니다.

    ```powershell
    Get-AzSubscription
    ```

    >**참고**: 출력에 **기본 공급자 구독**, **계량 구독** 및 **사용 구독**이 포함되어 있지 않음을 확인합니다.

>**검토**: 이 연습에서는 PowerShell을 통해 Azure Stack Hub 운영자 및 사용자 환경에 연결했습니다.
