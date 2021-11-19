---
lab:
    title: '랩: Azure Stack Hub에서 App Service 리소스 공급자 구현'
    module: '모듈 2: 서비스 제공'
---

# 랩 - Azure Stack Hub에서 App Service 리소스 공급자 구현
# 학생 랩 매뉴얼

## 랩 종속성

- Azure Stack Hub에서 SQL Server 리소스 공급자 구현

## 예상 소요 시간

4시간

## 랩 시나리오

여러분은 Azure Stack Hub 환경의 운영자입니다. 테넌트가 App Service 앱과 Azure 함수를 배포하도록 허용해야 합니다.

## 목표

이 랩을 완료하면 다음을 수행할 수 있습니다.

 - Azure Stack Hub에서 App Service 리소스 공급자 구현

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


### 연습 1: Azure Stack Hub에서 App Service 리소스 공급자 설치

이 연습에서는 Azure Stack Hub에서 App Service 리소스 공급자를 설치합니다.

1. SQL Server 호스팅 서버 프로비전
1. 파일 서버 프로비전
1. App Service 리소스 공급자 설치
1. App Service 리소스 공급자 설치 유효성 검사

>**참고**: 이 연습을 최대한 빠른 시간 내에 끝낼 수 있도록 Azure Stack Hub App Service 리소스 공급자를 설치하려면 수행해야 하는 다음을 비롯한 일부 작업은 이미 완료된 상태입니다.

- Azure Marketplace 신디케이션 구현
- 다음 Azure Marketplace 항목 다운로드:

  - **[smalldisk] Windows Server 2019 Datacenter Server Core-Bring your own license**
  - **Windows Server 2016 Datacenter-Bring your own license** 
  - **사용자 지정 스크립트 확장**


#### 작업 1: SQL Server 호스팅 서버 프로비전

이 작업에서는 다음을 수행합니다.

- SQL Server 호스팅 서버 프로비전

1. 필요한 경우 다음 자격 증명을 사용하여 **AzS-HOST1**에 로그인합니다.

    - 사용자 이름: **AzureStackAdmin@azurestack.local**
    - 암호: **Pa55w.rd1234**

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 [Azure Stack Hub 관리자 포털](https://adminportal.local.azurestack.external/)이 표시된 웹 브라우저 창을 열고 CloudAdmin@azurestack.local로 로그인합니다.
1. Azure Stack Hub 관리자 포털의 허브 메뉴에서 **리소스 만들기**를 클릭합니다.
1. **새로 만들기** 블레이드에서 **컴퓨팅**을 선택하고 사용 가능한 리소스 종류 목록에서 **{WS-BYOL} 무료 SQL Server 라이선스: Windows Server 2016의 SQL Server 2017 Express**를 선택합니다.
1. **가상 머신 만들기** 블레이드의 **기본 내용** 창에서 다음 설정을 지정하고 **확인**을 클릭합니다(나머지는 기본값을 그대로 유지).

    - 이름: **SqlHOST1**
    - VM 디스크 유형: **프리미엄 SSD**
    - 사용자 이름: **sqladmin**
    - 암호: **Pa55w.rd**
    - 구독: **기본 공급자 구독**
    - 리소스 그룹: 새 리소스 그룹 **sql.resources-RG**의 이름.
    - 위치: **로컬**

1. **크기 선택** 블레이드에서 **DS1_v2**를 선택하고 **선택**을 클릭합니다.
1. **가상 머신 만들기** 블레이드의 **설정** 창에서 **네트워크 보안 그룹** 설정을 **고급**으로 지정하고 **네트워크 보안 그룹(방화벽)** 을 클릭합니다.
1. **네트워크 보안 그룹 만들기** 블레이드에서 **+ 인바운드 규칙 추가**를 클릭합니다.
1. **인바운드 보안 규칙 추가** 블레이드에서 다음 설정을 지정하고 **추가**를 클릭합니다(나머지는 기본값을 그대로 유지).

    - 대상 포트 범위: **1433**
    - 프로토콜: **TCP**
    - 작업: **허용**
    - 우선 순위: **200**
    - 이름: **custom-allow-sql**

1. **네트워크 보안 그룹 만들기** 블레이드로 돌아와서 **확인**을 클릭합니다.
1. **가상 머신 만들기** 블레이드의 **설정** 창으로 돌아와서 다음 설정을 지정하고 **확인**을 클릭합니다(나머지는 기본값을 그대로 유지).

    - 부팅 진단: 사용 안 함
    - 게스트 OS 진단: 사용 안 함

1. **가상 머신 만들기** 블레이드의 **SQL Server 설정** 창에서 다음 설정을 지정하고 **확인**을 클릭합니다(나머지는 기본값을 그대로 유지).

    - SQL 연결: **공용(인터넷)**
    - 포트: **1433**
    - SQL 인증: **사용**
    - 로그인 이름: **SQLAdmin**
    - 암호: **Pa55w.rd**
    - 스토리지 구성: **일반**
    - 자동화된 패치: **사용 안 함**
    - 자동화된 백업: **사용 안 함**
    - Azure Key Vault 통합: **사용 안 함**

1. **가상 머신 만들기** 블레이드의 **요약** 창에서 **확인**을 클릭합니다.

    >**참고**: 배포가 완료될 때까지 기다립니다. 20분 정도 걸릴 수 있습니다.

1. 배포가 완료되면 **SqlHOST1** 가상 머신 블레이드로 이동하여 **개요** 섹션의 **DNS 이름** 레이블 바로 아래에 있는 **구성**을 클릭합니다.
1. **SqlHOST1-ip \| 구성** 블레이드의 **DNS 이름 레이블(옵션)** 텍스트 상자에 **sqlhost1**을 입력하고 **저장**을 클릭합니다.

    >**참고**: 이렇게 하면 **sqlhost1.local.cloudapp.azurestack.external** DNS 이름을 통해 **sqlhost1**을 사용할 수 있게 됩니다.

1. **sqlhost1-ip \| 구성** 블레이드에서 **할당** 옵션을 **정적**으로 설정하고 **저장**을 클릭합니다.

    >**참고**: 이렇게 하면 **sqlhost1** 가상 머신 다시 시작이 트리거됩니다. 다시 시작이 완료될 때까지 기다렸다가 다음 단계를 진행합니다.

1. **AzSHOST-1**에 연결된 원격 데스크톱 세션 내에서 **sqlhost1.local.cloudapp.azurestack.external**로 연결하는 원격 데스크톱 세션을 시작하고 메시지가 표시되면 다음 자격 증명을 사용하여 로그인합니다.

    - 사용자 이름: **SQLAdmin**
    - 암호: **Pa55w.rd**

1. **SqlHOST1**에 연결된 원격 데스크톱 세션 내에서 **시작**을 마우스 오른쪽 단추로 클릭하고 오른쪽 클릭 메뉴에서 **명령 프롬프트(관리자)** 를 선택합니다. 
1. **SqlHOST1**에 연결된 원격 데스크톱 세션 내의 **관리자:  명령 프롬프트**에서 다음 명령을 실행하여 로컬 SQL Server 인스턴스로의 SQLCMD 세션을 시작합니다.

    ```
    sqlcmd
    ```

1. **SqlHOST1**에 연결된 원격 데스크톱 세션 내의 **관리자: 명령 프롬프트**에서 다음 명령을 실행하여 SQL Server용으로 포함된 데이터베이스 인증을 사용하도록 설정합니다.

    ```
    sp_configure 'contained database authentication', 1;
    GO
    RECONFIGURE;
    GO
    ```

    > **참고:** 이 랩의 뒷부분에서 App Service 리소스 공급자를 구현할 때 이 호스팅 서버를 사용하려면 이 단계를 수행해야 합니다.

    > **참고:** **sqlhost1.local.cloudapp.azurestack.external**로 연결하는 원격 데스크톱 세션은 열어 둡니다. 이 랩 뒷부분에서 해당 세션을 사용할 것입니다.


#### 작업 2: 파일 서버 프로비전

이 작업에서는 다음을 수행합니다.

- 파일 서버 프로비전

1. **AzSHOST-1**에 연결된 원격 데스크톱 세션으로 전환하여 Azure Stack 관리자 포털의 허브 메뉴에서 **모든 서비스**를 클릭합니다.
1. 서비스 목록에서 **Marketplace 관리**를 클릭합니다.
1. **Marketplace 관리 - Marketplace 항목** 블레이드에서 **[smalldisk] Windows Server 2019 Datacenter Server Core-Bring your own license** 항목을 검색하여 해당 항목이 사용 가능한지 확인합니다.
1. **AzSHOST-1**에 연결된 원격 데스크톱 세션 내의 브라우저 창에서 새 탭을 열고 (https://aka.ms/appsvconmasdkfstemplate)으로 이동합니다.
1. **AzureStack-QuickStart-Templates / appservice-fileserver-standalone** 페이지에서 **azuredeploy.json**을 클릭한 다음 **Raw**를 클릭합니다.
1. 페이지의 전체 내용을 선택하여 클립보드에 복사합니다.
1. Azure Stack 관리자 포털로 다시 전환하여 **+ 리소스 만들기**를 클릭합니다.
1. **새로 만들기** 블레이드에서 **사용자 지정**을 클릭한 다음 **템플릿 배포**를 클릭합니다.
1. **사용자 지정 배포** 블레이드에서 **편집기에서 사용자 고유의 템플릿을 빌드합니다.** 를 선택합니다. 
1. **템플릿 편집** 블레이드에서 미리 작성된 템플릿을 클립보드의 내용으로 바꿉니다.
1. **사용자 지정 배포** 블레이드에서 **템플릿 편집**을 클릭합니다.
1. **템플릿 편집** 블레이드의 **매개 변수** 섹션에서 다음 값을 설정합니다.

    - **imageReference**의 **defaultValue**: **MicrosoftWindowsServer**로 설정 **| WindowsServer | 2019-Datacenter-Core-smalldisk | latest**
    - **imageReference**의 **allowedValue**: **MicrosoftWindowsServer**로 설정 **| WindowsServer | 2019-Datacenter-Core-smalldisk | latest**
    - **fileServerVirtualMachineSize**의 **defaultValue**: **Standard_A1_v2**로 설정
    - **fileServerVirtualMachineSize**의 **allowedValues**: **Standard_A1_v2**로 설정
    - **adminPassword**의 **defaultValue**: **Pa55w.rd1234**로 설정
    - **fileShareOwnerPassword**의 **defaultValue**: **Pa55w.rd1234**로 설정
    - **fileShareUserPassword**의 **defaultValue**: **Pa55w.rd1234**로 설정

    > **참고:** 그러면 **Parameters** 섹션의 콘텐츠가 다음과 같이 설정됩니다.

    ```json
      "parameters": {
        "fileServerVirtualMachineSize": {
          "type": "string",
          "defaultValue": "Standard_A1_v2",
          "allowedValues": [
            "Standard_A1_v2",
          ],
          "metadata": {
            "description": "Size of vm"
          }
        },
        "imageReference": {
          "type": "string",
          "defaultValue": "MicrosoftWindowsServer | WindowsServer | 2019-Datacenter-Core-smalldisk | latest",
          "allowedValues": [
            "MicrosoftWindowsServer | WindowsServer | 2019-Datacenter-Core-smalldisk | latest"
          ],
          "metadata": {
            "description": "Please ensure the image is available. publisher: MicrosoftWindowsServer | offer: WindowsServer | sku: 2016-Datacenter"
          }
        },
        "dnsNameForPublicIP": {
          "type": "string",
          "defaultValue": "appservicefileshare",
          "maxLength": 63,
          "metadata": {
            "description": "Unique DNS Name for the Public IP used to access the file share.It must be lowercase. It should match the following regular expression, or it will raise an error: ^[a-z][a-z0-9-]{1,61}[a-z0-9]$"
          }
        },
        "adminUsername": {
          "type": "string",
          "defaultValue": "fileshareowner",
          "metadata": {
            "description": "File server Admin user"
          }
        },
        "adminPassword": {
          "type": "securestring",
          "defaultValue": "Pa55w.rd1234",
          "metadata": {
            "description": "File server Admin password"
          }
        },
        "fileShareOwner": {
          "type": "string",
          "defaultValue": "fileshareowner",
          "metadata": {
            "description": "fileshare owner username"
          }
        },
        "fileShareOwnerPassword": {
          "type": "securestring",
          "defaultValue": "Pa55w.rd1234",
          "metadata": {
            "description": "fileshare owner password"
          }
        },
        "fileShareUser": {
          "type": "string",
          "defaultValue": "fileshareuser",
          "metadata": {
            "description": "fileshare user"
          }
        },
        "fileShareUserPassword": {
          "type": "securestring",
          "defaultValue": "Pa55w.rd1234",
          "metadata": {
            "description": "fileshare user password"
          }
        },
        "vmExtensionScriptLocation": {
          "type": "string",
          "defaultValue": "https://raw.githubusercontent.com/Azure/azurestack-quickstart-templates/master/appservice-fileserver-standalone",
          "metadata": {
            "description": "File Server extension script Url"
          }
        }
      },
    ```

1. **템플릿 편집** 블레이드의 **변수** 섹션에서 **"sku": "2016-Datacenter",** 를 **"sku": "2019-Datacenter-Core-smalldisk",** 로 바꾸고 **저장**을 선택합니다.
1. **사용자 지정 배포** 블레이드로 돌아와서 **구독** 드롭다운 목록에서 **기본 공급자 구독**을 선택하고 **리소스 그룹** 섹션에서 **sql.resources-RG**를 선택합니다.
1. **사용자 지정 배포** 블레이드에서 **검토 + 만들기**를 클릭한 다음 **만들기**를 클릭합니다.

    > **참고:** 배포가 완료될 때까지 기다립니다. 15분 정도 걸립니다.


#### 작업 3: App Service 리소스 공급자 설치

이 작업에서는 다음을 수행합니다.

- App Service 리소스 공급자 설치

1. **AzSHOST-1**에 연결된 원격 데스크톱 세션 내의 Azure Stack 관리자 포털 허브 메뉴에서 **모든 서비스**를 클릭합니다.
1. 서비스 목록에서 **Marketplace 관리**를 클릭합니다.
1. **Marketplace 관리 - Marketplace 항목** 블레이드에서 **Windows Server 2016 Datacenter-Bring your own license** 항목을 검색하여 해당 항목이 사용 가능한지 확인합니다.
1. **Marketplace 관리 - Marketplace 항목** 블레이드에서 **사용자 지정 스크립트 확장** 항목을 검색하여 해당 항목이 사용 가능한지 확인합니다.
1. **AzSHOST-1**에 연결된 원격 데스크톱 세션 내에서 웹 브라우저를 시작하고 (https://aka.ms/appsvconmasinstaller) 로 이동하여 **AppService.exe**를 다운로드합니다. 다운로드가 완료되면 **C:\\Downloads\\AppServiceRP** 폴더에 해당 파일을 복사합니다(필요하면 폴더를 만드세요).
1. 웹 브라우저에서 (https://aka.ms/appsvconmashelpers) 로 이동하여 **AppServiceHelperScripts.zip**을 다운로드합니다. 다운로드가 완료되면 **C:\\Downloads\\AppServiceRP** 폴더에 파일의 압축을 풉니다.
1. **AzSHOST-1**에 연결된 원격 데스크톱 세션 내에서 관리자로 Windows PowerShell을 시작합니다.
1. **관리자: Windows PowerShell** 창에서 다음 명령을 실행하여 Azure Stack에서 App Service에 필요한 인증서를 만듭니다.

    ```powershell
    Set-Location -Path C:\Downloads\AppServiceRP
    Get-ChildItem -Path '.\' -File -Recurse | Unblock-File

    $pfxPass = ConvertTo-SecureString 'Pa55w.rd1234pfx' -AsPlainText -Force

    .\Create-AppServiceCerts.ps1 `
	-pfxPassword $pfxPass `
	-DomainName 'local.azurestack.external'
    ```

1. **관리자: Windows PowerShell** 창에서 다음 명령을 실행하여 App Service 공급자를 설치하는 데 필요한 Azure Stack Hub용 Azure Resource Manager 루트 인증서를 만듭니다.

    ```
    $domain = 'azurestack.local'
    $privilegedEndpoint = 'AzS-ERCS01'

    # 권한 있는 엔드포인트 액세스에 필요한 클라우드 관리자 자격 증명을 추가합니다.
    $cloudAdminPass = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $cloudAdminCreds = New-Object System.Management.Automation.PSCredential ("CloudAdmin@$domain", $cloudAdminPass)

    .\Get-AzureStackRootCert.ps1 -PrivilegedEndpoint $privilegedEndpoint -CloudAdminCredential $cloudAdminCreds
    ```

    > **참고:** 이 스크립트는 Azure Stack Hub용 Azure Resource Manager 루트 인증서가 들어 있는 AzureStackCertificationAuthority.cer 파일을 로컬 폴더에 만듭니다.

1. **관리자: Windows PowerShell** 창에서 다음 명령을 실행하여 App Service 공급자를 설치하는 데 필요한 AD FS 앱을 만듭니다.

    ```
    $domain = 'azurestack.local'
    $privilegedEndpoint = 'AzS-ERCS01'
    $adminArmEndpoint = 'adminmanagement.local.azurestack.external'
    $certificateFilePath = 'C:\Downloads\AppServiceRP\sso.appservice.local.azurestack.external.pfx'
    $certificatePassword = ConvertTo-SecureString 'Pa55w.rd1234pfx' -AsPlainText -Force

    .\Create-ADFSIdentityApp.ps1 `
       -AdminArmEndpoint $adminArmEndpoint `
       -PrivilegedEndpoint $privilegedEndpoint `
       -CloudAdminCredential $cloudAdminCreds `
       -CertificateFilePath $certificateFilePath `
       -CertificatePassword $certificatePassword
    ```

1. 스크립트의 출력에서 생성된 AD FS 애플리케이션의 ID를 나타내는 GUID를 복사합니다. 

    > **참고:** 이 GUID를 적어 두세요. 이 작업 뒷부분에서 해당 GUID가 필요합니다.

1. **관리자: Windows PowerShell** 창의 **관리자: Windows PowerShell** 창에서 다음 명령을 실행하여 AppService.exe를 시작합니다.

    ```
    .\AppService.exe
    ```

    > **참고:** 이렇게 하면 Microsoft Azure App Service 설치 마법사가 시작됩니다.

1. **App Service를 배포하거나 최신 버전으로 업그레이드합니다.** 를 클릭합니다.
1. **Microsoft 소프트웨어 보조 사용 조건** 페이지에서 내용을 검토하고 **사용 조건을 읽고 이해했으며, 이에 동의합니다.** 체크박스를 클릭한 후에 **다음**을 클릭합니다.
1. 타사 사용 조건이 표시되는 페이지에서 내용을 검토하고 **사용 조건을 읽고 이해했으며, 이에 동의합니다.** 체크박스를 클릭한 후에 **다음**을 클릭합니다.
1. 관리자 및 테넌트 ARM 엔드포인트가 표시되는 페이지에서 정보가 정확한지 확인하고 **다음**을 클릭합니다.
1. Azure Stack App Service 클라우드 정보 페이지에서 **자격 증명** 옵션이 선택되어 있는지 확인하고 **연결**을 클릭합니다.
1. 메시지가 표시되면 암호 **Pa55w.rd1234**를 사용하여 **CloudAdmin@AzureStack.local**으로 로그인합니다.
1. Azure Stack App Service 클라우드 정보 페이지로 돌아와서 **Azure Stack 구독** 드롭다운 목록에서 **기본 공급자 구독**을 선택하고 **Azure Stack 위치** 드롭다운 목록에서는 **로컬**을 선택한 후 **다음**을 클릭합니다.
1. **가상 네트워크 구성**에서 기본 설정을 수락하고 **다음**을 클릭합니다.
1. 다음 페이지에서 아래 정보를 지정하고 **다음**을 클릭합니다.

    - 파일 공유 UNC 경로: **\\appservicefileshare.local.cloudapp.azurestack.external\websites**
    - 파일 공유 소유자: **fileshareowner**
    - 파일 공유 소유자 암호: **Pa55w.rd1234**
    - 파일 공유 사용자: **fileshareuser**
    - 파일 공유 사용자 암호: **Pa55w.rd1234**

1. 다음 페이지에서 이 작업 앞부분에서 생성한 애플리케이션 ID를 식별하는 설정을 지정하고 **다음**을 클릭합니다.

    - ID 애플리케이션 ID: 이 작업의 앞부분에서 복사한 GUID
    - ID 애플리케이션 인증서 파일(*.pfx): **C:\Downloads\AppServiceRP\sso.appservice.local.azurestack.external.pfx**
    - ID 애플리케이션 인증서 파일(*.pfx) 암호: **Pa55w.rd1234pfx**
    - ARM(Azure Resource Manager) 루트 인증서 파일(*.cer): **C:\Downloads\AppServiceRP\AzureStackCertificationAuthority.cer**

1. 다음 페이지에서 인증서 파일과 각 파일의 암호를 식별하는 설정을 지정합니다.

    - App Service 기본 SSL 인증서 파일(*.pfx): **C:\Downloads\AppServiceRP\_.appservice.local.azurestack.external.pfx**
    - App Service 기본 SSL 인증서(*.pfx) 암호: **Pa55w.rd1234pfx**
    - App Service API SSL 인증서 파일(*.pfx): **C:\Downloads\AppServiceRP\api.appservice.local.azurestack.external.pfx**
    - App Service API SSL 인증서(*.pfx) 암호: **Pa55w.rd1234pfx**
    - App Service 게시자 인증서 파일(*.pfx): **C:\Downloads\AppServiceRP\ftp.appservice.local.azurestack.external.pfx**
    - App Service 게시자 SSL 인증서(*.pfx) 암호: **Pa55w.rd1234pfx**

1. 다음 페이지에서 SQL Server 설정을 지정합니다.

    - SQL Server 이름: **sqlhost1.local.cloudapp.azurestack.external**
    - SQL sysadmin 로그인: **SQLAdmin**
    - SQL sysadmin 암호: **Pa55w.rd1234**

1. 다음 페이지에서 App Service 배포의 인스턴스 수와 SKU를 지정합니다.

    - 컨트롤러 역할: **2개 인스턴스 - Standard_A1_v2 - [1개 코어, 2048MB]**
    - 관리 역할: **1개 인스턴스 - Standard_A2_v2 - [2개 코어, 4096MB]**
    - 게시자 역할: **1개 인스턴스 - Standard_A1_v2 - [1개 코어, 2048MB]**
    - 프런트 엔드 역할: **1개 인스턴스 - Standard_A1_v2 - [1개 코어, 2048MB]**
    - 공유 작업자 역할: **1개 인스턴스 - Standard_A1_v2 - [1개 코어, 2048MB]**

1. **다음**을 클릭합니다.
1. 다음 페이지의 **플랫폼 이미지 선택** 드롭다운 목록에서 **2016 Datacenter - latest** 이미지를 선택하고 **다음**을 클릭합니다.
1. 다음 페이지에서 배포용으로 아래 관리자 자격 증명을 지정합니다.

    - 작업자 역할 가상 머신 관리자: **SAWorkerAdmin**
    - 작업자 역할 가상 머신 암호: **Pa55w.rd1234**
    - 암호 확인: **Pa55w.rd1234**
    - 기타 역할 가상 머신 관리자: **SAORoleAdmin**
    - 기타 역할 가상 머신 암호: **Pa55w.rd1234**
    - 암호 확인: **Pa55w.rd1234**

1. **다음**을 클릭합니다.
1. 요약 페이지에서 **배포를 시작하려면 선택하고 [다음]을 클릭하세요.** 체크박스를 클릭하고 배포를 시작하려면 **다음**을 클릭합니다.

    > **참고:** 설치가 완료될 때까지 기다립니다. 2~3시간 정도 걸릴 수 있습니다.

1. 설치가 완료되면 **끝내기**를 클릭합니다.


#### 작업 4: App Service 리소스 공급자 설치 유효성 검사

이 작업에서는 다음을 수행합니다.

- App Service 리소스 공급자 설치 유효성 검사

1. **AzSHOST-1**에 연결된 원격 데스크톱 내의 Azure Stack 관리자 포털이 표시된 웹 브라우저 내 허브 메뉴에서 **모든 서비스**를 선택합니다. 그런 다음 **모든 서비스** 블레이드에서 **관리**를 선택하고 서비스 목록에서 **App Service**를 클릭합니다. 

    > **참고:** 브라우저 페이지를 새로 고쳐야 **App Service** 항목이 사용 가능한 상태가 될 수도 있습니다.

1. **App Service** 블레이드의 **필수** 섹션에서 **상태** 레이블 아래에 **모든 역할이 준비됨** 메시지가 표시되는지 확인합니다.

    > **참고:** 모든 역할이 정상적으로 시작될 때까지 기다리세요. 이 경우 15~20분이 더 걸릴 수 있습니다.

>**검토**: 이 연습에서는 Azure Stack Hub에서 App Service 리소스 공급자를 설치했습니다.


### 연습 2: Azure Stack Hub에서 App Service 리소스 공급자의 관리 작업 살펴보기

이 연습에서는 Azure Stack Hub에서 App Service 리소스 공급자의 관리 작업을 살펴봅니다.

1. App Service 리소스의 확장 기능 살펴보기
1. App Service 리소스 공급자의 백업 설정 살펴보기


### 작업 1: App Service 리소스의 확장 기능 살펴보기

이 작업에서는 다음을 수행합니다.

- App Service 리소스의 확장 기능 검토
- App Service 리소스 공급자의 백업 설정 검토

1. **AzSHOST-1**에 연결된 원격 데스크톱 세션 내의 Azure Stack 관리자 포털이 표시된 웹 브라우저 내 **App Service** 블레이드에서 **역할**을 클릭합니다.
1. **App Service | 역할** 블레이드에서 역할 및 해당 인스턴스의 목록을 검토합니다.
1. **App Service | 역할** 블레이드의 **컨트롤러** 행에서 오른쪽 줄임표 기호를 클릭하고 드롭다운 목록에서 **가상 머신** 항목을 확인합니다.

    > **참고:** 컨트롤러 역할은 가상 머신을 사용하여 구현됩니다. 따라서 App Service 리소스 공급자를 설치할 때 컨트롤러 인스턴스 2개를 선택한 것입니다.

1. **App Service | 역할** 블레이드의 나머지 행에서 오른쪽 줄임표 기호를 클릭하고 드롭다운 목록에서 **ScaleSet** 항목을 확인합니다.

    > **참고:** 기타 모든 역할은 확장 집합을 사용하여 구현되므로 확장이 가능합니다.

1. **App Service | 역할** 블레이드에서 현재 **공유** 작업자 계층에는 웹 작업자 역할만 표시되어 있음을 확인합니다. 
1. **App Service | 역할** 블레이드 왼쪽의 세로 메뉴에서 **작업자 계층**을 선택합니다.
1. **App Service | 작업자 계층** 블레이드에서 **+ 추가**를 클릭합니다. 
1. **만들기** 블레이드에서 사용 가능한 옵션을 검토합니다. 사용 가능한 옵션으로는 **공유**와 **전용** 중 하나를 선택할 수 있는 **컴퓨팅 모드** 드롭다운 목록 등이 있습니다.

    > **참고:** 사용자 지정 소프트웨어가 포함된 다양한 크기의 가상 머신을 선택한 작업자 계층에서 가상 머신으로 배포할 수 있습니다.

1. 아무 항목도 변경하지 않고 **만들기** 블레이드를 닫습니다.

    > **참고:** 프로비전 프로세스는 1시간 넘게 걸릴 수도 있습니다.


### 작업 2: App Service 리소스 공급자의 백업 설정 살펴보기

이 작업에서는 다음을 수행합니다.

- App Service 리소스 공급자의 백업 설정 검토

> **참고:** Azure Stack Hub의 App Service 백업에는 다음과 같은 기본 구성 요소가 포함됩니다.

- 리소스 공급자 인프라
- 리소스 공급자 비밀 
- 계량 데이터베이스를 호스트하는 SQL Server 인스턴스
- App Service 파일 공유에 저장된 사용자 워크로드 콘텐츠

    > **참고:** App Service 복구 PowerShell cmdlet을 사용하면 복구 중에 백업에서 리소스 공급자 인프라 구성을 다시 만들 수 있습니다. 복구 프로세스 관련 세부 정보는 [Azure Stack Hub에서 App Service 복구](https://docs.microsoft.com/ko-kr/azure-stack/operator/app-service-recover?view=azs-2008)를 참조하세요.

1. **AzSHOST-1**에 연결된 원격 데스크톱 세션 내의 Azure Stack 관리자 포털이 표시된 웹 브라우저 내 **App Service** 블레이드에서 **비밀**을 클릭합니다.
1. **App Service \| 비밀** 블레이드에서 **비밀 다운로드**를 클릭한 다음 **저장**을 클릭합니다.
1. **AzSHOST-1**의 **Downloads** 폴더에 **SystemSecrets.json** 파일이 다운로드되었는지 확인합니다.

    > **참고:** **SystemSecrets.json** 파일을 안전한 위치에 복사하고 비밀을 교체할 때마다 이 프로세스를 반복해야 합니다. 

    > **참고:** **Appservice_hosting** 및 **Appservice_metering** 데이터베이스를 백업할 때는 Azure Backup Server의 SQL Server 유지 관리 계획을 사용하는 것이 좋습니다. 그러나 SQL Server PowerShell 모듈 cmdlet을 사용하여 이러한 데이터베이스를 백업할 수도 있습니다.

1. **AzSHOST-1**에 연결된 원격 데스크톱 세션 내에서 **sqlhost1.local.cloudapp.azurestack.external**로 연결하는 원격 데스크톱 세션으로 전환합니다. 
1. **SqlHOST1**에 연결된 원격 데스크톱 세션 내에서 관리자로 Windows PowerShell을 시작합니다.
1. **SqlHOST1**에 연결된 원격 데스크톱 세션 내의 **관리자: Windows PowerShell** 창에서 다음 명령을 실행하여 App Service 데이터베이스의 로컬 백업을 수행합니다.

    ```powershell
    $date = Get-Date -Format 'yyyyMMdd'
    New-Item -ItemType Directory -Path 'C:\Backups'
    Backup-SqlDatabase -ServerInstance 'localhost' -Database 'appservice_hosting' -BackupFile "C:\Backups\appservice_hosting_$date.bak" -CopyOnly
    Backup-SqlDatabase -ServerInstance 'localhost' -Database 'appservice_metering' -BackupFile "C:\Backups\appservice_metering_$date.bak" -CopyOnly
    ```

    > **참고:** App Service는 지정된 파일 공유에 테넌트 앱 정보를 저장합니다. 파일 공유를 백업할 때는 Azure Backup Server를 사용하는 것이 좋습니다. 그러나 어떤 파일 복사 유틸리티든 이러한 용도로 사용할 수 있습니다.

1. **AzSHOST-1**에 연결된 원격 데스크톱 세션으로 전환합니다. 그런 다음 **AzSHOST-1**에 연결된 원격 데스크톱 세션 내의 Azure Stack 관리자 포털이 표시된 웹 브라우저 내 **App Service** 블레이드에서 **시스템 구성**을 클릭합니다.
1. Azure Stack 관리자 포털이 표시된 웹 브라우저 내 **App Service \| 시스템 구성** 블레이드에서 **파일 공유**의 전체 경로(**\\\\appservicefileshare.local.cloudapp.azurestack.external\\websites**)를 확인합니다.
1. **AzSHOST-1**에 연결된 원격 데스크톱 세션으로 전환한 다음 **관리자:  Windows PowerShell** 창에서 다음 명령을 실행하여 App Service 파일 공유의 콘텐츠를 로컬 파일 시스템에 복사합니다.

    ```powershell
    $source = '\\appservicefileshare.local.cloudapp.azurestack.external\websites'
    $date = Get-Date -Format 'yyyyMMdd'
    $destination = "C:\Backups\FileShare\$date"
    New-Item -ItemType Directory -Path $destination -Force

    $fileshareusername = 'fileshareowner'
    $fileshareuserpassword = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $fileshareusercreds = New-Object System.Management.Automation.PSCredential ($fileshareusername, $fileshareuserpassword)
    New-PSDrive -Name 'S' -Root $source -PSProvider 'FileSystem' -Credential $fileshareusercreds
    robocopy $source $destination /e /r:1 /w:1
    Remove-PSDrive -Name 'S'
    ```

>**검토**: 이 연습에서는 Azure Stack Hub에서 App Service 리소스 공급자의 관리 작업을 살펴보았습니다.


### 연습 3: Azure Stack Hub에서 App Service 리소스 제공

이 연습에서는 Azure Stack Hub 사용자에게 App Service 리소스를 제공합니다.

1. 사용자에게 App Service 리소스 제공(클라우드 운영자 역할)
1. 웹앱 만들기(사용자 역할)


### 작업 1: 사용자에게 App Service 리소스 제공(클라우드 운영자 역할)

이 작업에서는 다음을 수행합니다.

- 사용자에게 App Service 리소스 제공(클라우드 운영자 역할)

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 [Azure Stack Hub 관리자 포털](https://adminportal.local.azurestack.external/)이 표시된 브라우저 창으로 전환합니다.
1. Azure Stack Hub 관리자 포털이 표시된 웹 브라우저 창에서 **+ 리소스 만들기**를 클릭합니다.
1. **새로 만들기** 블레이드에서 **제안+요금제**를 클릭한 다음 **요금제**를 클릭합니다.
1. **새 요금제** 블레이드의 **기본** 탭에서 다음 설정을 지정합니다.

    - 표시 이름: **app-service-plan1**
    - 리소스 이름: **app-service-plan1**
    - 리소스 그룹: 새 리소스 그룹 **app-service-plans-RG**의 이름.

1. **다음: 서비스 >** 를 클릭합니다.
1. **새 요금제** 블레이드의 **서비스** 탭에서 **Microsoft.Web** 체크박스를 선택합니다.
1. **다음: 할당량>** 을 클릭합니다.
1. **새 요금제** 블레이드의 **할당량** 탭에서 **새로 만들기**를 선택하고 **만들기** 블레이드에서 다음 설정을 지정한 후에 **확인**을 클릭합니다.

    - 이름: **app-service-quota1**
    - App Service 요금제: **사용자 지정** **20**
    - 공유 App Service 요금제: **사용자 지정** **10**
    - 전용 App Service 요금제: **사용자 지정** **10**
    - 가격 책정 SKU: **사용자 지정** **2개 선택됨**(**무료** 및 **공유**)
    - 사용량 요금제: **사용**

    >**참고**: 사용량 요금제 모델에서 Azure Functions를 제공하려면 공유 웹 작업자를 배포해야 합니다.

1. **새 요금제** 블레이드의 **할당량** 탭으로 돌아와서 **검토 + 만들기**를 클릭한 다음 **만들기**를 클릭합니다.

    >**참고**: 배포가 완료될 때까지 기다립니다. 몇 초면 끝납니다.

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내의 Azure Stack Hub 관리자 포털이 표시된 웹 브라우저 창에서 **새로 만들기** 블레이드로 돌아와서 **제안**을 클릭합니다.
1. **새 제안 만들기** 블레이드의 **기본** 탭에서 다음 설정을 지정합니다.

    - 표시 이름: **app-service-offer1**
    - 리소스 이름: **app-service-offer1**
    - 리소스 그룹: **app-service-offers-RG**
    - 이 제안을 공개로 설정: **예**

1. **다음: 기본 요금제 >** 를 클릭합니다. 
1. **새 제안 만들기** 블레이드의 **기본 요금제** 탭에서 **app-service-plan1** 항목 옆의 체크박스를 선택합니다.
1. **다음: 추가 요금제 >** 를 클릭합니다.
1. **추가 요금제** 설정은 기본값으로 유지하고 **검토 + 만들기**를 클릭한 다음 **만들기**를 클릭합니다.

    >**참고**: 배포가 완료될 때까지 기다립니다. 몇 초면 끝납니다.


#### 작업 2: 웹앱 만들기(사용자 역할)

이 작업에서는 다음을 수행합니다.

- 테스트 사용자 계정 만들기
- 웹앱 만들기(사용자 역할)

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 **시작**을 클릭하고 시작 메뉴에서 **Windows 관리 도구**를 클릭합니다. 그런 다음 관리 도구 목록에서 **Active Directory 관리 센터**를 두 번 클릭합니다.
1. **Active Directory 관리 센터** 콘솔에서 **azurestack(로컬)** 을 클릭합니다.
1. 세부 정보 창에서 **사용자** 컨테이너를 두 번 클릭합니다.
1. **작업** 창의 **사용자** 섹션에서 **새로 만들기 -> 사용자**를 클릭합니다.
1. **사용자 만들기** 창에서 다음 설정을 지정하고 **확인**을 클릭합니다. 

    - 전체 이름: **T1U1**
    - 사용자 UPN 로그온: **t1u1@azurestack.local**
    - 사용자 SamAccountName: **azurestack\t1u1**
    - 암호: **Pa55w.rd**
    - 암호 옵션: **기타 암호 옵션 -> 암호 사용 기간 제한 없음**

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 웹 브라우저의 InPrivate 세션을 시작합니다.
1. 웹 브라우저 창에서 [Azure Stack Hub 사용자 포털](https://portal.local.azurestack.external)로 이동하여 **Pa55w.rd** 암호를 사용해 **t1u1@azurestack.local**로 로그인합니다.
1. Azure Stack Hub 사용자 포털의 대시보드에서 **구독 가져오기** 타일을 클릭합니다.
1. **구독 가져오기** 블레이드의 **이름** 텍스트 상자에 **t1u1-app-service-subscription1**을 입력합니다.
1. 제안 목록에서 **app-service-offer1**을 선택하고 **만들기**를 클릭합니다.
1. **구독이 생성되었습니다. 구독을 사용해서 시작하려면 포털을 새로 고쳐야 합니다.** 메시지가 표시되면 **새로 고침**을 클릭합니다. 
1. Azure Stack Hub 테넌트 포털의 허브 메뉴에서 **+ 리소스 만들기**를 클릭합니다.
1. 서비스 목록에서 **웹 + 모바일**을 클릭한 다음 **웹앱**을 클릭합니다. 
1. **웹앱** 블레이드에서 다음 설정을 지정합니다.

    - 구독: **t1u1-app-service-subscription1**
    - 앱 이름: **t1u1webapp1**
    - 리소스 그룹: 새 리소스 그룹 **webapps-RG**의 이름.

1. **웹앱** 블레이드에서 **App Service 계획/위치**를 클릭하고 **App Service 계획** 블레이드에서 **+ 새로 만들기**를 클릭합니다. 
1. **새 App Service 계획** 블레이드에서 다음 설정을 지정합니다.

    - App Service 계획: **appserviceplan1**
    - 위치: **로컬**

1. **새 App Service 계획** 블레이드에서 **가격 책정 계층**을 클릭합니다.
1. **사양 선택기** 블레이드에서 **F1** 가격 책정 계층을 선택하고 **적용**을 클릭합니다.
1. **새 App Service 계획** 블레이드로 돌아와서 **확인**을 클릭합니다.
1. **웹앱** 블레이드로 돌아와서 **만들기**를 클릭합니다.

    >**참고**: 배포가 완료될 때까지 기다립니다. 1분도 걸리지 않습니다.

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내의 Azure Stack Hub 사용자 포털이 표시된 웹 브라우저 InPrivate 세션 내 허브 메뉴에서 **모든 리소스**를 클릭합니다.
1. **모든 리소스** 블레이드의 구독 필터 드롭다운 목록에서 **t1u1-app-service-subscription1** 항목을 선택하고 **새로 고침**을 클릭합니다.
1. **모든 리소스** 블레이드의 리소스 목록에서 **t1u1webapp1** 항목을 클릭합니다.
1. **t1u1webapp1** 블레이드에서 **찾아보기**를 클릭합니다.

    >**참고**: 그러면 다른 브라우저 탭이 열리고 새로 프로비전한 웹앱의 기본 홈 페이지가 표시됩니다.

>**검토**: 이 연습에서는 App Service를 사용자에게 제공했으며 테넌트 사용자로 웹앱을 만들었습니다.
