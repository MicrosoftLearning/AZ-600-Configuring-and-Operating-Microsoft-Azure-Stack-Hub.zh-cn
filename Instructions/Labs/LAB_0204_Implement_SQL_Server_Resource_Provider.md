---
lab:
    title: '랩: Azure Stack Hub에서 SQL Server 리소스 공급자 구현'
    module: '모듈 2: 서비스 제공'
---

# 랩 - Azure Stack Hub에서 SQL Server 리소스 공급자 구현
# 학생 랩 매뉴얼

## 랩 종속성

- 없음

## 예상 소요 시간

150분

## 랩 시나리오

여러분은 Azure Stack Hub 환경의 운영자입니다. 테넌트가 SQL Server 데이터베이스를 배포하도록 허용해야 합니다. 

## 목표

이 랩을 완료하면 다음을 수행할 수 있습니다.

- Azure Stack Hub에서 SQL Server 리소스 공급자 구현

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


### 연습 1: Azure Stack Hub에서 SQL Server 리소스 공급자 설치

이 연습에서는 Azure Stack Hub에서 SQL Server 리소스 공급자를 설치합니다.

1. SQL Server 리소스 공급자 바이너리 다운로드 
1. SQL Server 리소스 공급자 설치
1. SQL Server 리소스 공급자 설치 확인

>**참고**: 이 연습을 최대한 빠른 시간 내에 끝낼 수 있도록 Azure Stack Hub SQL Server 리소스 공급자를 설치하려면 수행해야 하는 다음을 비롯한 일부 작업은 이미 완료된 상태입니다.

- Azure Marketplace 신디케이션 구현
- Azure Marketplace에서 **Microsoft AzureStack 추가 기능 RP Windows Server** 다운로드

#### 작업 1: SQL Server 리소스 공급자 바이너리 다운로드

이 작업에서는 다음을 수행합니다.

- SQL Server 리소스 공급자 바이너리 다운로드

1. 필요한 경우 다음 자격 증명을 사용하여 **AzS-HOST1**에 로그인합니다.

    - 사용자 이름: **AzureStackAdmin@azurestack.local**
    - 암호: **Pa55w.rd1234**

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 [Azure Stack Hub 관리자 포털](https://adminportal.local.azurestack.external/)이 표시된 웹 브라우저 창을 열고 CloudAdmin@azurestack.local로 로그인합니다.
1. **AzSHOST-1**에 연결된 원격 데스크톱 세션 내의 Azure Stack 관리자 포털이 표시된 웹 브라우저 내 허브 메뉴에서 **모든 서비스**를 클릭합니다.
1. **모든 서비스** 블레이드에서 **Marketplace 관리**를 검색하여 해당 항목을 선택합니다.
1. Marketplace 관리 블레이드에서 사용 가능한 서비스 목록에 **Microsoft AzureStack 추가 기능 RP Windows Server**가 표시되어 있는지 확인합니다.
1. **AzSHOST-1**에 연결된 원격 데스크톱 세션 내에서 다른 웹 브라우저 창을 시작합니다. 그런 다음 (https://aka.ms/azshsqlrp11931) 에서 SQL 리소스 공급자 자동 압축 풀기 실행 파일을 다운로드하여 **C:\\Downloads\\SQLRP** 폴더에 해당 파일의 압축을 풉니다(폴더를 먼저 만들어야 함).


#### 작업 2: SQL Server 리소스 공급자 설치

이 작업에서는 다음을 수행합니다.

- SQL Server 리소스 공급자 설치

1. AzSHOST-1에 연결된 원격 데스크톱 세션 내에서 관리자로 Windows PowerShell을 시작합니다.

    > **참고:** 새 PowerShell 세션을 시작하세요.

1. **관리자: Windows PowerShell** 프롬프트에서 다음 명령을 실행하여 신뢰할 수 있는 리포지토리로 PowerShell 갤러리를 구성합니다.

    ```powershell
    Set-PSRepository -Name 'PSGallery' -InstallationPolicy Trusted
    ```

1. **관리자: Windows PowerShell** 프롬프트에서 다음 명령을 실행하여 SQL Server 리소스 공급자에 필요한 AzureRM.Bootstrapper 모듈 버전을 설치합니다.

    ```powershell
    Get-Module -Name Azs.* -ListAvailable | Uninstall-Module -Force -Verbose
    Get-Module -Name Azure* -ListAvailable | Uninstall-Module -Force -Verbose

    Install-Module -Name AzureRm.BootStrapper -RequiredVersion 0.5.0 -Force
    Install-Module -Name AzureStack -RequiredVersion 1.6.0
    ```

1. **관리자: Windows PowerShell** 프롬프트에서 다음 명령을 실행하여 신뢰할 수 있는 리포지토리로 PowerShell 갤러리를 구성합니다.

    ```powershell
    Add-AzureRmEnvironment -Name 'AzureStackAdmin' -ArmEndpoint 'https://adminmanagement.local.azurestack.external' `
       -AzureKeyVaultDnsSuffix adminvault.local.azurestack.external `
       -AzureKeyVaultServiceEndpointResourceId https://adminvault.local.azurestack.external
    ```

1. **관리자: Windows PowerShell** 프롬프트에서 다음 명령을 실행하여 현재 환경을 설정합니다.

    ```powershell
    Set-AzureRmEnvironment -Name 'AzureStackAdmin'
    ```

1. **관리자: Windows PowerShell** 프롬프트에서 다음 명령을 실행하여 핸재 환경에 인증합니다(메시지가 표시되면 암호로 **Pa55w.rd1234**를 사용해 **CloudAdmin@azurestack.local** 사용자로 로그인).

    ```powershell
    Connect-AzureRmAccount -EnvironmentName 'AzureStackAdmin'
    ```

1. **관리자: Windows PowerShell** 프롬프트에서 다음 명령을 실행하여 인증이 정상적으로 완료되었으며 해당 컨텍스트가 설정되었음을 확인합니다.

    ```powershell
    Get-AzureRmContext
    ```

1. **관리자: Windows PowerShell** 프롬프트에서 다음 명령을 실행하여 SQL Server 리소스 공급자를 설치하는 데 필요한 변수를 설정합니다.

    ```powershell
    $domain = 'azurestack.local'
    $privilegedEndpoint = 'AzS-ERCS01'
    $downloadDir = 'C:\Downloads\SQLRP'

    # AzureStack\AzureStackAdmin 자격 증명 설정
    $serviceAdmin = 'AzureStackAdmin@azurestack.local'
    $serviceAdminPass = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $serviceAdminCreds = New-Object System.Management.Automation.PSCredential ($serviceAdmin, $serviceAdminPass)

    # AzureStack\CloudAdmin 자격 증명 설정
    $cloudAdminName = 'AzureStack\CloudAdmin'
    $cloudAdminPass = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $cloudAdminCreds = New-Object PSCredential($cloudAdminName, $cloudAdminPass)

    # 새 리소스 공급자 VM 로컬 관리자 계정 설정
    $vmLocalAdminPass = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $vmLocalAdminCreds = New-Object System.Management.Automation.PSCredential ('sqlrpadmin', $vmLocalAdminPass)

    # Set a password that will protect the private key of a self-signed certificate generated to secure the SQL Server resource provider
    $pfxPass = ConvertTo-SecureString 'Pa55w.rd1234pfx' -AsPlainText -Force

    # SQL Server 리소스 공급자 모듈을 포함하도록 PowerShell 모듈 경로 환경 변수 업데이트
    $rpModulePath = Join-Path -Path $env:ProgramFiles -ChildPath 'SqlMySqlPsh'
    $env:PSModulePath = $env:PSModulePath + ';' + $rpModulePath 
    ```

1. **관리자: Windows PowerShell** 프롬프트에서 현재 디렉터리를 SQL Server 리소스 공급자 설치 파일의 압축을 풀었던 위치로 변경하고 DeploySQLProvider.ps1 스크립트를 실행합니다.

    ```powershell
    Set-Location -Path 'C:\Downloads\SQLRP'

    .\DeploySQLProvider.ps1 `
        -AzCredential $serviceAdminCreds `
        -VMLocalCredential $vmLocalAdminCreds `
        -CloudAdminCredential $cloudAdminCreds `
        -PrivilegedEndpoint $privilegedEndpoint `
        -DefaultSSLCertificatePassword $pfxPass
    ```

    > **참고:** 설치가 완료될 때까지 기다립니다. 1시간 정도 걸릴 수 있습니다.

#### 작업 3: SQL Server 리소스 공급자 설치 확인

이 작업에서는 다음을 수행합니다.

- SQL Server 리소스 공급자 설치 확인

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 Azure Stack 관리자 포털이 표시된 웹 브라우저 창으로 전환한 다음 허브 메뉴에서 **리소스 그룹**을 클릭합니다. 
1. **리소스 그룹** 블레이드에서 **system.local.sqladapter**를 클릭합니다.
1. **system.local.sqladapter** 블레이드에서 **배포** 항목을 검토하여 모든 배포가 정상적으로 완료되었는지 확인합니다.
1. Azure Stack 관리자 포털에서 **가상 머신**으로 이동하여 SQL 리소스 공급자 VM이 정상적으로 생성되어 실행되고 있는지 확인합니다.
1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 [Azure Stack Hub 테넌트 포털](https://portal.local.azurestack.external/)이 표시된 웹 브라우저 창을 열고 메시지가 표시되면 CloudAdmin@azurestack.local로 로그인합니다.
1. Azure Stack Hub 테넌트 포털의 허브 메뉴에서 **리소스 만들기**를 클릭합니다.
1. **새로 만들기** 블레이드에서 **데이터 + 스토리지**를 선택하고 사용 가능한 리소스 종류 목록에 **SQL 데이터베이스**가 표시되는지 확인합니다.

>**검토**: 이 연습에서는 Azure Stack Hub에서 SQL Server 리소스 공급자를 설치했습니다.


### 연습 2: Azure Stack Hub에서 SQL Server 리소스 공급자 구성

이 연습에서는 Azure Stack Hub에서 SQL Server 리소스 공급자를 구성합니다.

1. SQL Server 호스팅 서버용 요금제, 제안 및 구독 만들기(클라우드 운영자 역할)
1. SQL Server 호스팅 서버로 사용할 Azure Stack Hub VM 배포(클라우드 운영자 역할)
1. SQL 호스팅 서버 추가(클라우드 운영자 역할)
1. 사용자에게 SQL 데이터베이스 제공(클라우드 운영자 역할)
1. SQL 데이터베이스 만들기(사용자 역할)

>**참고**: 이 연습을 최대한 빠른 시간 내에 끝낼 수 있도록 Azure Stack Hub SQL Server 리소스 공급자 구성을 원활하게 진행하려면 수행해야 하는 다음을 비롯한 일부 작업은 이미 완료된 상태입니다.

- Azure Marketplace에서 SQL Server 이미지 다운로드
- Azure Marketplace에서 **Sql IaaS VM 확장** 다운로드


#### 작업 1: SQL Server 호스팅 서버용 요금제, 제안 및 구독 만들기(클라우드 운영자 역할)

이 작업에서는 다음을 수행합니다.

- SQL Server 호스팅 서버용 요금제, 제안 및 구독 만들기(클라우드 운영자 역할)

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 [Azure Stack Hub 관리자 포털](https://adminportal.local.azurestack.external/)이 표시된 웹 브라우저 창을 열고 CloudAdmin@azurestack.local로 로그인합니다.
1. Azure Stack Hub 관리자 포털이 표시된 웹 브라우저 창에서 **+ 리소스 만들기**를 클릭합니다.
1. **새로 만들기** 블레이드에서 **제안+요금제**를 클릭한 다음 **요금제**를 클릭합니다.
1. **새 요금제** 블레이드의 **기본** 탭에서 다음 설정을 지정합니다.

    - 표시 이름: **sql-server-hosting-plan1**
    - 리소스 이름: **sql-server-hosting-plan1**
    - 리소스 그룹: 새 리소스 그룹 **sql-server-hosting-plans-RG**의 이름.

1. **다음: 서비스 >** 를 클릭합니다.
1. **새 요금제** 블레이드의 **서비스** 탭에서 **Microsoft.Compute**, **Microsoft.Storage** 및 **Microsoft.Network** 체크박스를 선택합니다.
1. **다음: 할당량>** 을 클릭합니다.
1. **새 요금제** 블레이드의 **할당량** 탭에서 다음 설정을 지정합니다.

    - Microsoft.Compute: **기본 할당량**
    - Microsoft.Network: **기본 할당량**
    - Microsoft.Storage: **기본 할당량**

1. **검토 + 만들기**와 **만들기**를 차례로 클릭합니다.

    >**참고**: 배포가 완료될 때까지 기다립니다. 몇 초면 끝납니다.

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내의 Azure Stack Hub 관리자 포털이 표시된 웹 브라우저 창에서 새로 만들기 블레이드로 돌아와서 **제안**을 클릭합니다.
1. **새 제안 만들기** 블레이드의 **기본** 탭에서 다음 설정을 지정합니다.

    - 표시 이름: **sql-server-hosting-offer1**
    - 리소스 이름: **sql-server-hosting-offer1**
    - 리소스 그룹: **sql-server-hosting-offers-RG**
    - 이 제안을 공개로 설정: **아니요**

1. **다음: 기본 요금제 >** 를 클릭합니다. 
1. **새 제안 만들기** 블레이드의 **기본 요금제** 탭에서 **sql-server-hosting-plan1** 항목 옆의 체크박스를 선택합니다.
1. **다음: 추가 요금제 >** 를 클릭합니다.
1. **추가 요금제** 설정은 기본값으로 유지하고 **검토 + 만들기**를 클릭한 다음 **만들기**를 클릭합니다.

    >**참고**: 배포가 완료될 때까지 기다립니다. 몇 초면 끝납니다.

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내의 Azure Stack Hub 관리자 포털이 표시된 웹 브라우저 창에서 **새로 만들기** 블레이드로 돌아와서 **구독**을 클릭합니다.
1. **사용자 구독 만들기** 블레이드에서 다음 설정을 지정하고 **만들기**를 클릭합니다.

    - 이름: **sql-server-hosting-subscription1**
    - 사용자: **cloudadmin@azurestack.local**
    - 디렉터리 테넌트: **ADFS.azurestack.local**
    - 제안 이름: **sql-server-hosting-offer1**

    >**참고**: 배포가 완료될 때까지 기다립니다. 몇 초면 끝납니다.

1. Azure Stack Hub 관리자 포털 창은 열어 둡니다.


#### 작업 2: SQL Server 호스팅 서버로 사용할 Azure Stack Hub VM 배포(클라우드 운영자 역할)

이 작업에서는 다음을 수행합니다.

- SQL Server 호스팅 서버로 사용할 Azure Stack Hub VM 배포(클라우드 운영자 역할)

    >**참고**: 청구 가능한 사용자 구독에서 SQL Server 호스팅 서버로 작동하는 Azure Stack Hub VM을 만들어야 합니다.

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 [Azure Stack Hub 테넌트 포털](https://portal.local.azurestack.external/)이 표시된 웹 브라우저 창으로 전환합니다.
1. Azure Stack Hub 테넌트 포털의 허브 메뉴에서 **리소스 만들기**를 클릭합니다.
1. 새로 만들기 블레이드에서 컴퓨팅을 선택하고 사용 가능한 리소스 종류 목록에서 **{WS-BYOL} 무료 SQL Server 라이선스: Windows Server 2016의 SQL Server 2017 Express**를 선택합니다.
1. **가상 머신 만들기** 블레이드의 **기본 내용** 창에서 다음 설정을 지정하고 **확인**을 클릭합니다(나머지는 기본값을 그대로 유지).

    - 이름: **sql-host-vm0**
    - VM 디스크 유형: **프리미엄 SSD**
    - 사용자 이름: **sqladmin**
    - 암호: **Pa55w.rd**
    - 구독: **sql-server-hosting-subscription1**
    - 리소스 그룹: 새 리소스 그룹 **sql-server-hosting-RG**의 이름.
    - 위치: **로컬**

1. **크기 선택** 블레이드에서 **DS1_v2**를 선택하고 **선택**을 클릭합니다.
1. **가상 머신 만들기** 블레이드의 **설정** 창에서 **네트워크 보안 그룹** 설정을 **고급**으로 지정하고 **네트워크 보안 그룹(방화벽)** 을 클릭합니다.
1. **네트워크 보안 그룹 만들기** 블레이드에서 **+ 인바운드 규칙 추가**를 클릭합니다.
1. **인바운드 보안 규칙 추가** 블레이드에서 다음 설정을 지정하고 **추가**를 클릭합니다(나머지는 기본값을 그대로 유지).

    - 대상 포트 범위: **1433**
    - 프로토콜: **TCP**
    - 작업: **허용**
    - 이름: **SQL**

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

1. 배포가 완료되면 **sql-host-vm0** 가상 머신 블레이드로 이동하여 **개요** 섹션의 **DNS 이름** 레이블 바로 아래에 있는 **구성**을 클릭합니다.
1. **sql-host-vm0-ip \| 구성** 블레이드의 **DNS 이름 레이블(옵션)** 텍스트 상자에 **sql-host-vm0**을 입력하고 **저장**을 클릭합니다.

    >**참고**: 이렇게 하면 **sql-host-vm0.local.cloudapp.azurestack.external** DNS 이름을 통해 **sql-host-vm0**을 사용할 수 있게 됩니다.

1. **sql-host-vm0-ip \| 구성** 블레이드에서 **할당** 옵션을 **정적**으로 설정하고 **저장**을 클릭합니다.

    >**참고**: 이렇게 하면 **sql-host-vm0** 가상 머신 다시 시작이 트리거됩니다.


#### 작업 3: SQL 호스팅 서버 추가(클라우드 운영자 역할)

이 작업에서는 다음을 수행합니다.

- SQL 호스팅 서버 추가(클라우드 운영자 역할)

1. **AzSHOST-1**에 연결된 원격 데스크톱 세션 내의 Azure Stack 관리자 포털이 표시된 웹 브라우저에서 **모든 서비스**를 클릭하고 **관리 리소스** 섹션에서 **SQL 호스팅 서버**를 클릭합니다.

    > **참고:** Azure Stack 관리자 포털이 표시된 브라우저 페이지를 새로 고쳐야 **SQL 호스팅 서버** 리소스 종류가 표시될 수도 있습니다.

1. **SQL 호스팅 서버** 블레이드에서 **+ 추가**를 클릭합니다.
1. **SQL 호스팅 서버 추가** 블레이드에서 다음 설정을 지정합니다.

    - SQL Server 이름: **sql-host-vm0.local.cloudapp.azurestack.external**
    - 사용자 이름: **sqladmin**
    - 암호: **Pa55w.rd**
    - 호스팅 서버 크기(GB): **50**
    - Always On 가용성 그룹: 선택 취소
    - 구독: **기본 공급자 구독**
    - 리소스 그룹: 새 리소스 그룹 **sql.resources-RG**의 이름.
    - 위치: **로컬**

1. **SQL 호스팅 서버 추가** 블레이드에서 **SKU**를 클릭하고 **SKU** 블레이드에서 **새 SKU 만들기**를 클릭한 후에 **SKU 만들기** 블레이드에서 다음 설정을 지정합니다.

    - 이름: **MSSQL2017Exp**
    - 제품군: **SQL Server 2017**
    - 계층: **독립 실행형**
    - 버전: **Express**

1. **SKU 만들기** 블레이드에서 **확인**을 클릭하고 **SQL 호스팅 서버 추가** 블레이드로 돌아와서 **만들기**를 클릭합니다.

    > **참고:** 작업이 완료될 때까지 기다립니다. 1분도 걸리지 않습니다.

1. **SQL 호스팅 서버** 블레이드에서 **새로 고침**을 클릭하고 서버 목록에 **sqlhost1.local.cloudapp.azurestack.external**이 표시되는지 확인합니다.


#### 작업 4: 사용자에게 SQL 데이터베이스 제공(클라우드 운영자 역할)

이 작업에서는 다음을 수행합니다.

- 사용자에게 SQL 데이터베이스 제공(클라우드 운영자 역할)

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내의 Azure Stack Hub 관리자 포털이 표시된 웹 브라우저 창에서 **+ 리소스 만들기**를 클릭합니다.
1. **새로 만들기** 블레이드에서 **제안+요금제**를 클릭한 다음 **요금제**를 클릭합니다.
1. **새 요금제** 블레이드의 **기본** 탭에서 다음 설정을 지정합니다.

    - 표시 이름: **sql-server-2017-express-db-plan1**
    - 리소스 이름: **sql-server-2017-express-db-plan1**
    - 리소스 그룹: 새 리소스 그룹 **sqldb-plans-RG**의 이름.

1. **다음: 서비스 >** 를 클릭합니다.
1. **새 요금제** 블레이드의 **서비스** 탭에서 **Microsoft.SQLAdapter** 체크박스를 선택합니다.
1. **다음: 할당량>** 을 클릭합니다.
1. **새 요금제** 블레이드의 **할당량** 탭에서 **Microsoft.SQLAdapter** 항목 옆의 **새로 만들기**를 클릭합니다.
1. **할당량 만들기** 블레이드에서 다음 설정을 지정하고 **만들기**를 클릭합니다.

    - 할당량 이름: **sql-server-2017-express-db-quota1**
    - 모든 데이터베이스의 최대 크기(GB): **2**
    - 최대 데이터베이스 수: **20**

1. **검토 + 만들기**와 **만들기**를 차례로 클릭합니다.

    >**참고**: 배포가 완료될 때까지 기다립니다. 몇 초면 끝납니다.

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내의 Azure Stack Hub 관리자 포털이 표시된 웹 브라우저 창에서 **새로 만들기** 블레이드로 돌아와서 **제안**을 클릭합니다.
1. **새 제안 만들기** 블레이드의 **기본** 탭에서 다음 설정을 지정합니다.

    - 표시 이름: **sql-server-2017-express-db-offer1**
    - 리소스 이름: **sql-server-2017-express-db-offer1**
    - 리소스 그룹: **sqldb-offers-RG**
    - 이 제안을 공개로 설정: **예**

1. **다음: 기본 요금제 >** 를 클릭합니다. 
1. **새 제안 만들기** 블레이드의 **기본 요금제** 탭에서 **sql-server-2017-express-db-plan1** 항목 옆의 체크박스를 선택합니다.
1. **다음: 추가 요금제 >** 를 클릭합니다.
1. **추가 요금제** 설정은 기본값으로 유지하고 **검토 + 만들기**를 클릭한 다음 **만들기**를 클릭합니다.

    >**참고**: 배포가 완료될 때까지 기다립니다. 몇 초면 끝납니다.


#### 작업 5: SQL 데이터베이스 만들기(사용자 역할)

이 작업에서는 다음을 수행합니다.

- 테스트 사용자 계정 만들기
- 새로 만든 사용자 계정을 사용하여 SQL 데이터베이스 만들기

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
1. **구독 가져오기** 블레이드의 **이름** 텍스트 상자에 **t1u1-sqldb-subscription1**을 입력합니다.
1. 제안 목록에서 **sql-server-2017-express-db-offer1**을 선택하고 **만들기**를 클릭합니다.
1. **구독이 생성되었습니다. 구독을 사용해서 시작하려면 포털을 새로 고쳐야 합니다.** 메시지가 표시되면 **새로 고침**을 클릭합니다. 
1. Azure Stack Hub 테넌트 포털의 허브 메뉴에서 **모든 서비스**를 클릭합니다.
1. 서비스 목록에서 **SQL 데이터베이스**를 클릭합니다.
1. **SQL 데이터베이스** 블레이드에서 **+ 추가**를 클릭합니다.
1. **데이터베이스 만들기** 블레이드에서 다음 설정을 지정합니다.

    - 데이터베이스 이름: **sqldb1**
    - 데이터 정렬: **SQL_Latin1_General_CP1_CI_AS**
    - 최대 크기(MB): **200**
    - 구독: **t1u1-sqldb-subscription1**
    - 리소스 그룹: 새 리소스 그룹 **sqldb-RG**의 이름.
    - 위치: **로컬**
    - SKU: **MSSQL2017Exp**

    >**참고**: 새로 만든 SKU를 테넌트 포털에서 사용하려면 잠시 기다려야 할 수도 있습니다.

1. **데이터베이스 만들기** 블레이드에서 **로그인**을 클릭합니다.
1. **로그인 선택** 블레이드에서 **새 로그인 만들기**를 클릭합니다.
1. **새 로그인** 블레이드에서 다음 설정을 지정하고 **확인**을 클릭합니다.

    - 데이터베이스 로그인: **dbAdmin**
    - 암호: **Pa55w.rd**

1. **데이터베이스 만들기** 블레이드로 돌아와서 **만들기**를 클릭합니다.

    >**참고**: 배포가 완료될 때까지 기다립니다. 1분도 걸리지 않습니다. 

>**검토**: 이 연습에서는 SQL Server 호스팅 서버를 Azure Stack Hub에 추가하고 테넌트에 제공했으며, 테넌트 사용자로 SQL 데이터베이스를 배포했습니다.
