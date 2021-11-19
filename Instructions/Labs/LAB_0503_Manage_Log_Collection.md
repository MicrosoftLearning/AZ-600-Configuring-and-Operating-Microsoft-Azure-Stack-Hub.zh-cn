---
lab:
    title: '랩: Azure Stack Hub에서 로그 수집 관리'
    module: '모듈 5: 인프라 관리'
---

# 랩 - Azure Stack Hub에서 로그 수집 관리
# 학생 랩 매뉴얼

## 랩 종속성

- 없음

## 예상 소요 시간

30분

## 랩 시나리오

여러분은 Azure Stack Hub 환경의 운영자입니다. 로그 수집을 수행할 방법을 확인해야 합니다.

## 목표

이 랩을 완료하면 다음을 수행할 수 있습니다.

- Azure Stack Hub 로그 수집 수행

## 랩 환경 

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


### 연습 1: Azure Stack Hub 진단 로그 수집 기능 살펴보기

이 연습에서는 진단 로그 수집 옵션 관리를 위한 여러 가지 옵션을 살펴봅니다.

1. 자동 관리 진단 로그 수집 사용
1. 요청 시 진단 로그 전송
1. 로컬 파일 공유에 진단 로그 복사

#### 작업 1: 자동 관리 진단 로그 수집 사용

이 작업에서는 다음을 수행합니다.

- 자동 관리 로그 수집 사용

>**참고**: 자동 관리 로그 수집에서는 지원 사례를 개설하기 전에 진단 로그를 자동 수집하여 Azure Stack Hub에서 Microsoft로 전송합니다. 이러한 로그는 시스템 상태 경고 발생 시에만 수집되며, 지원 사례 해결을 위해 필요한 상황에서 Microsoft 지원 담당자만 로그에 액세스합니다.

1. 필요한 경우 다음 자격 증명을 사용하여 **AzSHOST-1**에 로그인합니다.

    - 사용자 이름: **AzureStackAdmin@azurestack.local**
    - 암호: **Pa55w.rd1234**

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 [Azure Stack Hub 관리자 포털](https://adminportal.local.azurestack.external/)이 표시된 웹 브라우저 창을 열고 CloudAdmin@azurestack.local로 로그인합니다.
1. Azure Stack Hub 관리자 포털이 표시된 웹 브라우저 창의 허브 메뉴에서 **도움말 + 지원**을 클릭합니다.
1. **개요** 블레이드에서 **로그 수집**을 클릭합니다.
1. **개요 \| 로그 수집** 블레이드에서 **자동 관리 로그 수집 사용**을 클릭합니다.

    >**참고**: **자동 관리 로그 수집 사용** 옵션을 사용할 수 없으면 다음 단계로 바로 넘어가세요. 

1. **설정** 블레이드에서 다음 설정을 지정하고 **저장**을 클릭합니다.

    - 자동 관리 로그 수집: **사용**
    - 로그 위치: **Azure(권장)**

1. **개요 \| 로그 수집** 블레이드로 돌아온 다음 **설정**을 클릭하여 지정한 설정에 따라 자동 관리 로그 수집이 구성되었는지 확인합니다.


#### 작업 2: 요청 시 진단 로그 전송

이 작업에서는 다음을 수행합니다.

- Azure Stack Hub 관리 포털을 사용하여 요청 시 진단 로그 전송
- Azure Stack Hub PowerShell을 사용하여 요청 시 진단 로그 전송 

>**참고**: 이 기능의 명칭은 *지금 로그 보내기*입니다.

>**참고**: 먼저 Azure Stack Hub 관리 포털을 사용하여 요청 시 진단 로그를 전송합니다.

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내의 [Azure Stack Hub 관리자 포털](https://adminportal.local.azurestack.external/)이 표시된 웹 브라우저 창 허브 메뉴에서 **도움말 + 지원**을 클릭합니다.
1. **개요** 블레이드에서 **로그 수집**을 클릭합니다.
1. **개요 \| 로그 수집** 블레이드에서 **지금 로그 보내기**를 클릭합니다.
1. **지금 로그 보내기** 블레이드에서 다음 설정을 지정하고 **수집 + 업로드**를 클릭합니다.

    - 시작: 현재 날짜와 시간 - 3시간
    - 종료: 현재 날짜 및 시간

    >**참고**: 업로드가 완료될 때까지 기다리지 말고 다음 단계를 진행하세요. 이 로그 수집은 실패합니다. 랩 환경에서 대상 Azure 구독에 직접 액세스할 수 없으므로, 이 실패는 정상적인 현상입니다.

    >**참고**: 이제 Azure Stack Hub PowerShell을 사용하여 동일 기능을 구성합니다. 이 구성을 수행하려면 권한 있는 엔드포인트에 연결해야 합니다.

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

    >**참고**: PowerShell 원격 세션이 정상적으로 설정되었는지 확인합니다. 세션이 정상적으로 설정되면 권한 있는 엔드포인트를 실행 중인 인프라 VM의 IP 주소로 시작하는 프롬프트가 Windows PowerShell ISE 창의 콘솔 창에서 대괄호 안에 표시되어야 합니다.

1. **관리자: Windows PowerShell ISE** 창의 콘솔 창 내 PowerShell 원격 세션에서 다음 명령을 실행하여 요청 시 Azure Stack Hub 스토리지 진단 로그를 전송합니다.

    ```powershell
    Send-AzureStackDiagnosticLog -FilterByRole Storage
    ```

    >**참고**: 업로드가 완료될 때까지 기다리지 말고 다음 단계를 진행하세요. 이 로그 수집은 실패합니다. 랩 환경에서 대상 Azure 구독에 직접 액세스할 수 없으므로, 이 실패는 정상적인 현상입니다.

    >**참고**: 역할을 기준으로 로그를 필터링할 수도 있고, 로그를 수집할 기간을 지정할 수도 있습니다. 자세한 내용은 [진단 로그 수집](https://docs.microsoft.com/ko-kr/azure/azure-stack/azure-stack-diagnostics)을 참조하세요.


#### 작업 3: 로컬 파일 공유에 진단 로그 복사

이 작업에서는 다음을 수행합니다.

- Azure Stack Hub 관리 포털을 사용하여 로컬 파일 공유에 진단 로그 복사
- Azure Stack Hub PowerShell을 사용하여 로컬 파일 공유에 진단 로그 복사 

>**참고**: 먼저 로그를 저장할 파일 공유를 만듭니다.

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 파일 탐색기를 시작합니다. 
1. 파일 탐색기에서 새 폴더 **C:\\AzSHLogs**를 만듭니다.
1. 파일 탐색기에서 **AzSHLogs** 폴더를 마우스 오른쪽 단추로 클릭하고 오른쪽 클릭 메뉴에서 **속성**을 클릭합니다.
1. **AzSHLogs 속성** 창에서 **공유** 탭을 클릭한 다음 **고급 공유**를 클릭합니다.
1. **고급 공유** 대화 상자에서 **이 폴더 공유**를 클릭한 다음 **권한**을 클릭합니다.
1. **AzSHLogs에 대한 권한** 창에서 **모든 사용자** 항목이 선택되어 있는지 확인하고 **제거**를 클릭합니다.
1. **추가**를 클릭하고 **사용자, 컴퓨터, 서비스 계정 또는 그룹 선택** 대화 상자에 **CloudAdmins**를 입력한 후에 **확인**을 클릭합니다.
1. **CloudAdmins** 항목이 선택되어 있는지 확인하고 **허용** 열에서 **모든 권한** 체크박스를 선택합니다.
1. **추가**를 클릭하고 **사용자, 컴퓨터, 서비스 계정 또는 그룹 선택** 대화 상자에서 **위치**를 클릭합니다.
1. **위치** 대화 상자에서 로컬 컴퓨터를 나타내는 항목(**AzS-HOST1**)을 클릭하고 **확인**을 클릭합니다.
1. **선택할 개체 이름 입력** 텍스트 상자에 **Administrators**를 입력하고 **확인**을 클릭합니다.
1. **Administrators** 항목이 선택되어 있는지 확인하고 **허용** 열에서 **모든 권한** 체크박스를 클릭한 후에 **확인**을 클릭합니다.
1. **고급 공유** 대화 상자로 돌아와서 **확인**을 클릭합니다.
1. **AzSHLogs 속성** 창으로 돌아와서 **보안** 탭을 클릭한 다음 **편집**을 클릭합니다.
1. **추가**를 클릭하고 **사용자, 컴퓨터, 서비스 계정 또는 그룹 선택** 대화 상자에 **CloudAdmins**를 입력한 후에 **확인**을 클릭합니다.
1. **AzSHLogs에 대한 권한** 대화 상자의 **그룹 또는 사용자 이름** 창에 있는 항목 목록에서 **CloudAdmins**를 클릭합니다. 그런 다음 **CloudAdmins에 대한 권한** 창의 **허용** 열에서 **모든 권한**을 클릭하고 **확인**을 클릭합니다.
1. **AzSHLogs 속성** 창으로 돌아와서 **닫기**를 클릭합니다.

    >**참고**: 다음으로는 Azure Stack Hub 관리 포털에서, 또는 권한 있는 엔드포인트를 통해 파일 공유로의 진단 로그 수집을 구성할 수 있습니다.

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내의 [Azure Stack Hub 관리자 포털](https://adminportal.local.azurestack.external/)이 표시된 웹 브라우저 창 허브 메뉴에서 **도움말 + 지원**을 클릭합니다.
1. **개요** 블레이드에서 **로그 수집**을 클릭합니다.
1. **개요 \| 로그 수집** 블레이드에서 **설정**을 클릭합니다.
1. **설정** 블레이드에서 **로그 위치** 옵션을 **Azure(권장)** 에서 **로컬 파일 공유**로 변경하고 다음 설정을 지정합니다.

    - SMB 파일 공유 경로: **\\\\AzS-HOST1.azurestack.local\\AzSHLogs**
    - 사용자 이름: **AZURESTACK\\AzureStackAdmin**
    - 암호: **Pa55w.rd1234**

1. **설정** 블레이드에서 **저장**을 클릭합니다.
1. **개요 \| 로그 수집** 블레이드에서 **지금 로그 보내기**를 클릭합니다.
1. **지금 로그 보내기** 블레이드에서 다음 설정을 지정하고 **수집 + 업로드**를 클릭합니다.

    - 시작: 현재 날짜와 시간 - 3시간
    - 종료: 현재 날짜 및 시간

    >**참고**: 로그 수집 및 업로드 진행률을 확인하려면 **개요 \| 로그 수집** 블레이드에서 **새로 고침**을 클릭합니다.

    >**참고**: 업로드가 완료될 때까지 기다리지 말고 다음 단계를 진행하세요. 이번에는 로그 수집이 성공하지만 수집이 완료되려면 15분 정도 걸릴 수 있습니다. 

    >**참고**: 이제 Azure Stack Hub PowerShell을 사용하여 동일 기능을 구성합니다. 이 구성을 수행하려면 권한 있는 엔드포인트에 연결해야 합니다.

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 이전 작업에서 권한 있는 엔드포인트에 연결했던 **관리자: Windows PowerShell** 콘솔로 다시 전환합니다.
1. **관리자: Windows PowerShell** 창의 콘솔 창 내 PowerShell 원격 세션에서 다음 명령을 실행하여 로컬 파일 공유에 Azure Stack Hub 스토리지 진단 로그를 복사합니다.

    ```powershell
    Get-AzureStackLog -OutputSharePath '\\AzS-HOST1.azurestack.local\AzSHLogs' -OutputShareCredential $using:adminCredentials -FilterByRole Storage
    ```

1. cmdlet 실행이 완료될 때까지 기다렸다가 파일 탐색기에서 **C:\\AzSHLogs** 폴더의 내용을 검토합니다.

    >**참고**: 이 폴더에는 앞에서 시작한 개별 복사 작업에 해당하는 폴더가 포함되어 있어야 합니다. 폴더 이름의 형식은 **AzureStackLogs-*YYYYMMDDHHMMSS*-AZS-ERCS01**이어야 합니다. 여기서 ***YYYYMMDDHHMMSS***는 복사본의 타임스탬프를 나타냅니다.

1. **관리자: Windows PowerShell** 창의 PowerShell 원격 세션 프롬프트에서 다음 명령을 실행하여 세션을 닫습니다.

    ```powershell
    Close-PrivilegedEndpoint -TranscriptsPathDestination '\\AzS-HOST1.azurestack.local\AzSHLogs' -Credential $using:adminCredentials
    ```

>**검토**: 이 연습에서는 권한 있는 엔드포인트로의 PowerShell 원격 세션을 설정했으며 해당 엔드포인트에서 사용 가능한 PowerShell cmdlet을 사용하여 진단 로그를 수집했습니다.
