---
lab:
    title: '랩: Azure Stack Hub에서 권한 있는 엔드포인트 액세스'
    module: '모듈 5: 인프라 관리'
---

# 랩 - Azure Stack Hub에서 권한 있는 엔드포인트 액세스
# 학생 랩 매뉴얼

## 랩 종속성

- 없음

## 예상 소요 시간

30분

## 랩 시나리오

여러분은 Azure Stack Hub 환경의 운영자입니다. 권한 있는 엔드포인트에 액세스하는 방법을 확인해야 합니다.

## 목표

이 랩을 완료하면 다음을 수행할 수 있습니다.

- Azure Stack Hub 권한 있는 엔드포인트 액세스 

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


### 연습 1: 권한 있는 엔드포인트를 통해 Azure Stack Hub 관리

이 연습에서는 권한 있는 엔드포인트로의 PowerShell 원격 세션을 설정하여 해당 원격 세션을 통해 액세스 가능한 Windows PowerShell cmdlet을 실행합니다. 이 연습에서는 다음 작업을 수행합니다.

1. Windows PowerShell을 통해 권한 있는 엔드포인트에 연결
1. 권한 있는 엔드포인트를 통해 사용 가능한 기능 검토
1. 권한 있는 엔드포인트로의 연결을 닫고 세션 기록 수집

#### 작업 1: Windows PowerShell을 통해 권한 있는 엔드포인트에 연결

이 작업에서는 다음을 수행합니다.

- Windows PowerShell을 통해 권한 있는 엔드포인트에 연결

1. 필요한 경우 다음 자격 증명을 사용하여 **AzS-HOST1**에 로그인합니다.

    - 사용자 이름: **AzureStackAdmin@azurestack.local**
    - 암호: **Pa55w.rd1234**

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 관리자로 PowerShell ISE를 시작합니다.
1. **관리자: Windows PowerShell ISE** 콘솔에서 다음 명령을 실행하여 권한 있는 엔드포인트를 실행 중인 인프라 VM의 IP 주소를 확인합니다. 

    ```powershell
    $ipAddress = (Resolve-DnsName -Name AzS-ERCS01).IPAddress
    ```

1. **관리자: Windows PowerShell ISE** 창에서 다음 명령을 실행하여 권한 있는 엔드포인트를 실행 중인 인프라 VM의 IP 주소를 WinRM 신뢰할 수 있는 호스트 목록에 추가합니다(모든 호스트가 이미 허용되어 있는 경우는 제외).

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

1. **관리자: Windows PowerShell ISE** 창에서 다음 명령을 실행하여 변수에 Azure Stack Hub 관리자 자격 증명을 저장합니다.

    ```powershell
    $adminUserName = 'CloudAdmin@azurestack.local'
    $adminPassword = 'Pa55w.rd1234' | ConvertTo-SecureString -Force -AsPlainText
    $adminCredentials = New-Object PSCredential($adminUserName,$adminPassword)
    ```

1. **관리자: Windows PowerShell ISE** 창에서 다음 명령을 실행하여 권한 있는 엔드포인트에 대한 PowerShell 원격 세션을 설정합니다.

    ```powershell
    Enter-PSSession -ComputerName $ipAddress -ConfigurationName PrivilegedEndpoint -Credential $adminCredentials
    ```

1. PowerShell 원격 세션이 정상적으로 설정되었는지 확인합니다. 세션이 정상적으로 설정되면 권한 있는 엔드포인트를 실행 중인 인프라 VM의 IP 주소로 시작하는 프롬프트가 Windows PowerShell ISE 창의 콘솔 창에서 대괄호 안에 표시되어야 합니다.


#### 작업 2: 권한 있는 엔드포인트를 통해 사용 가능한 기능 검토

이 작업에서는 다음을 수행합니다.

- 권한 있는 엔드포인트를 통해 사용 가능한 기능 검토

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내의 PowerShell 원격 세션에 표시된 **관리자: Windows PowerShell ISE** 창의 콘솔 창에서 다음 명령을 실행하여 사용 가능한 모든 PowerShell cmdlet을 확인합니다.

    ```powershell
    Get-Command
    ```

1. **관리자: Windows PowerShell ISE** 창의 PowerShell 원격 세션에서 다음 명령을 실행하여 현재 클라우드 관리 사용자 계정을 확인합니다.

    ```powershell
    Get-CloudAdminUserList
    ```

    >**참고**: 반환되는 목록에는 CloudAdmin과 AzureStackAdmin의 2개 계정만 포함되어 있습니다.

1. **관리자: Windows PowerShell ISE** 창의 PowerShell 원격 세션에서 다음 명령을 실행하여 Azure Stack Hub의 업데이트 준비 유효성을 검사하고 결과를 검토합니다.

    ```powershell
    Test-AzureStack -Group UpdateReadiness 
    ```

    >**참고**: ASDK에서는 업데이트를 지원하지 않으므로 이 명령은 반드시 데모용으로만 사용해야 합니다. 

    >**참고**: **Test-AzureStack** 기능 소개 정보는 [Azure Stack Hub 시스템 상태 유효성 검사](https://docs.microsoft.com/ko-kr/azure-stack/operator/azure-stack-diagnostic-test?view=azs-2008)를 참조하세요.

    >**참고**: 지원 시나리오에서는 Microsoft 지원 엔지니어가 Azure Stack Hub 인프라의 내부 항목에 액세스하기 위해 권한 있는 엔드포인트 PowerShell 세션의 권한을 높여야 할 수도 있습니다. 이 프로세스를 권한 있는 엔드포인트 잠금 해제라고 합니다. 세션 권한 상승 프로세스는 사용자 2명이 조직 인증 2회를 진행하는 2단계 프로세스입니다. 항상 환경을 제어할 수 있는 Azure Stack Hub 운영자가 잠금 해제 절차를 시작합니다. 다음 작업에서 이 프로세스를 파악할 수 있는 에뮬레이트된 시나리오를 단계별로 진행할 예정입니다.

1. **관리자: Windows PowerShell ISE** 창의 PowerShell 원격 세션에서 다음 명령을 실행하여 권한 있는 엔드포인트가 현재 잠겨 있는지 유효성을 검사합니다.

    ```powershell
    Get-SupportSessionInfo
    ```

1. **관리자: Windows PowerShell ISE** 창의 PowerShell 원격 세션에서 다음 명령을 실행하여 지원 세션 토큰을 생성합니다.

    ```powershell
    Get-SupportSessionToken
    ```

    >**참고**: 지원 시나리오에서는 Microsoft 지원 엔지니어가 선택한 채팅, 이메일 등의 매체를 통해 요청 토큰을 엔지니어에게 전달합니다. 그러면 Microsoft 지원 엔지니어는 요청 토큰을 사용하여 지원 세션 인증 토큰을 생성한 다음 해당 값을 릴레이합니다. 다음으로는 동일 PowerShell 원격 세션에서 **Unlock-SupportSession** cmdlet을 실행하고 메시지가 표시되면 지원 세션 인증 토큰 값을 입력합니다. 그러면 PowerShell 원격 세션의 권한이 상승되어 인프라에 아무 제한 없이 연결할 수 있으며 모든 관리 기능을 사용할 수 있게 됩니다.


#### 작업 3: 권한 있는 엔드포인트로의 세션을 닫고 세션 기록 수집

이 작업에서는 다음을 수행합니다.

- 권한 있는 엔드포인트로의 세션을 닫고 세션 기록 수집

>**참고**: 권한 있는 엔드포인트에서는 모든 작업 및 해당 출력이 로깅됩니다. 로그를 수집하려면 **Close-PrivilegedEndpoint** cmdlet을 사용하여 세션을 닫습니다. 그러면 엔드포인트가 닫히고 로그 파일이 외부 파일 공유에 보존용으로 전송됩니다.

>**참고**: 먼저 권한 있는 엔드포인트 로그를 저장할 파일 공유를 만듭니다.

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 관리자로 다른 PowerShell ISE를 시작합니다.
1. **관리자: Windows PowerShell ISE** 콘솔에서 다음 명령을 실행하여 권한 있는 엔드포인트 세션 로그를 저장할 공유를 만들고 구성합니다.

    ```powershell
    $pepGroup = 'AZURESTACK\CloudAdmins'
    New-Item -Path 'C:\PEPLogs' -ItemType Directory -Force
    $pepShare = New-SmbShare -Name 'PEPLogs' -Description 'PEPLogs' -Path 'C:\PEPLogs'
    Grant-SmbShareAccess -Name $pepShare.Name -AccountName $pepGroup -AccessRight Full -Force
    Revoke-SmbShareAccess -Name $pepShare.Name -AccountName 'Everyone' -Force
    ```

1. **관리자: Windows PowerShell ISE** 창의 PowerShell 원격 세션으로 다시 전환한 후 다음 명령을 실행하여 권한 있는 엔드포인트 세션을 닫고 외부 파일 공유에 보존용으로 세션 로그 파일을 전송합니다.

    ```powershell
    Close-PrivilegedEndpoint -TranscriptsPathDestination '\\AzS-HOST1.azurestack.local\PEPLogs' -Credential $using:adminCredentials
    ```

1. cmdlet 실행이 완료될 때까지 기다렸다가 파일 탐색기에서 **C:\\PEPLogs** 폴더의 내용을 검토합니다.

>**검토**: 이 연습에서는 권한 있는 엔드포인트로의 PowerShell 원격 세션을 설정하여 해당 기능을 검토한 후 세션을 닫았습니다.
