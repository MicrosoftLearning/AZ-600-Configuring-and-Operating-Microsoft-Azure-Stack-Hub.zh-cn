---
lab:
    title: '랩: Azure Stack Hub 인프라 백업 구성'
    module: '모듈 5: 인프라 관리'
---

# 랩 - Azure Stack Hub 인프라 백업 구성
# 학생 랩 매뉴얼

## 랩 종속성

- 없음

## 예상 소요 시간

30분

## 랩 시나리오

여러분은 Azure Stack Hub 환경의 운영자입니다. 인프라 백업을 수행할 수 있도록 환경을 준비해야 합니다. 

## 목표

이 랩을 완료하면 다음을 수행할 수 있습니다.

- Azure Stack Hub 인프라 백업 구성

## 랩 환경 

이 랩에서는 AD FS(Active Directory Federation Services)와 통합된 ASDK 인스턴스(ID 공급자로 백업된 Active Directory)를 사용합니다. 

랩 환경에는 다음과 같은 구성 요소가 포함됩니다.

- 다음 액세스 지점을 사용하여 **AzSHOST-1** 서버에서 실행되는 ASDK 배포:

  - 관리자 포털: https://adminportal.local.azurestack.external
  - 관리자 ARM 엔드포인트: https://adminmanagement.local.azurestack.external
  - 사용자 포털: https://portal.local.azurestack.external
  - 사용자 ARM 엔드포인트: https://management.local.azurestack.external

- 관리자:

  - ASDK 클라우드 운영자 사용자 이름: **CloudAdmin@azurestack.local**
  - ASDK 클라우드 운영자 암호: **Pa55w.rd1234**
  - ASDK 호스트 관리자 사용자 이름: **AzureStackAdmin@azurestack.local**
  - ASDK 호스트 관리자 암호: **Pa55w.rd1234**

이 랩을 진행하면서 PowerShell을 통해 Azure Stack Hub를 관리하는 데 필요한 소프트웨어를 설치합니다. 그리고 사용자 계정을 추가로 만듭니다.

### 연습 1: Azure Stack Hub 인프라 백업 구성

이 연습에서는 Azure Stack Hub 인프라 백업 구성을 준비합니다.

1. 백업 사용자 만들기
1. 백업 공유 만들기
1. 암호화 키 생성
1. 백업 컨트롤러 구성

#### 작업 1: 백업 사용자 만들기

이 작업에서는 다음을 수행합니다.

- 백업 사용자 만들기

1. 필요한 경우 다음 자격 증명을 사용하여 **AzSHOST-1**에 로그인합니다.

    - 사용자 이름: **AzureStackAdmin@azurestack.local**
    - 암호: **Pa55w.rd1234**

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 **시작**을 클릭하고 시작 메뉴에서 **Windows 관리 도구**를 클릭합니다. 그런 다음 관리 도구 목록에서 **Active Directory 관리 센터**를 두 번 클릭합니다.
1. **Active Directory 관리 센터** 콘솔에서 **azurestack(로컬)** 을 클릭합니다.
1. 세부 정보 창에서 **사용자** 컨테이너를 두 번 클릭합니다.
1. **작업** 창의 **사용자** 섹션에서 **새로 만들기 -> 사용자**를 클릭합니다.
1. **사용자 만들기** 창에서 다음 설정을 지정하고 **확인**을 클릭합니다. 

    - 전체 이름: **AzS-BackupOperator**
    - 사용자 UPN 로그온: **AzS-BackupOperator@azurestack.local**
    - 사용자 SamAccountName: **azurestack\AzS-BackupOperator**
    - 암호: **Pa55w.rd**
    - 암호 옵션: **기타 암호 옵션 -> 암호 사용 기간 제한 없음**

#### 작업 2: 백업 공유 만들기

이 작업에서는 다음을 수행합니다.

- 백업 공유 만들기 

    >**참고**: 랩 이외의 시나리오에서는 이 공유가 Azure Stack Hub 배포 외부에 배치됩니다. 그러나 여기서는 작업을 간편하게 수행하기 위해 Azure Stack Hub 호스트에 공유를 직접 만듭니다.

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 파일 탐색기를 시작합니다. 
1. 파일 탐색기에서 새 폴더 **C:\\Backup**을 만듭니다.
1. 파일 탐색기에서 **Backup** 폴더를 마우스 오른쪽 단추로 클릭하고 오른쪽 클릭 메뉴에서 **속성**을 클릭합니다.
1. **Backup 속성** 창에서 **공유** 탭을 클릭한 다음 **고급 공유**를 클릭합니다.
1. **고급 공유** 대화 상자에서 **이 폴더 공유**를 클릭한 다음 **권한**을 클릭합니다.
1. **Backup에 대한 권한** 창에서 **모든 사용자** 항목이 선택되어 있는지 확인하고 **제거**를 클릭합니다.
1. **추가**를 클릭하고 **사용자, 컴퓨터, 서비스 계정 또는 그룹 선택** 대화 상자에 **AzS-BackupOperator**를 입력한 후에 **확인**을 클릭합니다.
1. **AzS-BackupOperator** 항목이 선택되어 있는지 확인하고 **허용** 열에서 **모든 권한** 체크박스를 선택합니다.
1. **추가**를 클릭하고 **사용자, 컴퓨터, 서비스 계정 또는 그룹 선택** 대화 상자에서 **위치**를 클릭합니다.
1. **위치** 대화 상자에서 로컬 컴퓨터를 나타내는 항목(**AzS-HOST1**)을 클릭하고 **확인**을 클릭합니다.
1. **선택할 개체 이름 입력** 텍스트 상자에 **Administrators**를 입력하고 **확인**을 클릭합니다.
1. **Administrators** 항목이 선택되어 있는지 확인하고 **허용** 열에서 **모든 권한** 체크박스를 클릭한 후에 **확인**을 클릭합니다.
1. **고급 공유** 대화 상자로 돌아와서 **확인**을 클릭합니다.
1. **Backup 속성** 창으로 돌아와서 **보안** 탭을 클릭한 다음 **편집**을 클릭합니다.
1. **추가**를 클릭하고 **사용자, 컴퓨터, 서비스 계정 또는 그룹 선택** 대화 상자에 **AzS-BackupOperator**를 입력한 후에 **확인**을 클릭합니다.
1. **Backup에 대한 권한** 대화 상자의 **그룹 또는 사용자 이름** 창에 있는 항목 목록에서 **AzS-BackupOperator**를 클릭합니다. 그런 다음 **AzS-BackupOperator에 대한 권한** 창의 **허용** 열에서 **모든 권한**을 클릭하고 **확인**을 클릭합니다. 
1. **Backup 속성** 창으로 돌아와서 **닫기**를 클릭합니다.

#### 작업 3: 암호화 키 생성

이 작업에서는 다음을 수행합니다.

- 암호화 키 생성 

    >**참고**: 모든 인프라 백업은 암호화해야 합니다. 따라서 인프라 백업을 구성하려면 암호화 키 쌍에 해당하는 인증서를 제공해야 합니다. 여기서는 Windows PowerShell을 사용하여 키를 생성합니다. 

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 관리자로 PowerShell 7을 시작합니다.
1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 암호화 키 쌍과 해당 인증서를 생성합니다.
    
    ```powershell
    $cert = New-SelfSignedCertificate `
               -DnsName "azsh.contoso.com" `
               -CertStoreLocation 'cert:\LocalMachine\My'

    New-Item -Path 'C:\' -Name 'CertStore' -ItemType 'Directory'

    Export-Certificate `
        -Cert $cert `
        -FilePath C:\CertStore\AzSHIBPK.cer 
    ```

    >**참고**: Azure Stack Hub에서는 인프라 백업용으로 자체 서명 인증서를 사용할 수 있습니다. 복구 중에 프라이빗 키를 제공해야 하므로 프로덕션 환경에서는 안전한 위치에 프로덕션 키를 저장해 두어야 합니다.


#### 작업 4: 백업 컨트롤러 구성

이 작업에서는 다음을 수행합니다.

- 백업 컨트롤러 구성

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 [Azure Stack Hub 관리자 포털](https://adminportal.local.azurestack.external/)이 표시된 웹 브라우저 창을 열고 CloudAdmin@azurestack.local로 로그인합니다.
1. Azure Stack Hub 관리자 포털에서 **모든 서비스**를 클릭합니다.
1. **모든 서비스** 블레이드에서 **관리**를 선택하고 **인프라 백업**을 선택합니다. 
1. **인프라 백업** 블레이드에서 **구성**을 클릭합니다.
1. **백업 컨트롤러 설정** 블레이드에서 다음 설정을 지정하고 **확인**을 클릭합니다.

    - 백업 스토리지 위치: **\\AzS-HOST1.azurestack.local\Backup**
    - 사용자 이름: **AzS-BackupOperator@azurestack.local**
    - 암호: **Pa55w.rd**
    - 암호 확인: **Pa55w.rd**
    - 백업 빈도(시간): **12**
    - 보존 기간(일): **7**
    - 인증서 .cer 파일: **C:\CertStore\AzSHIBPK.cer**

1. 인프라 백업이 사용하도록 설정되었는지 확인하려면 Azure Stack Hub 관리자 포털이 표시된 웹 브라우저 페이지를 새로 고치고 **인프라 백업** 블레이드로 다시 이동합니다.
1. **인프라 백업** 블레이드에서 백업 설정을 검토합니다.
1. 필요한 경우 **지금 백업**을 클릭하여 주문형 백업을 시작합니다.

    >**참고**: 백업 프로세스는 완료하려면 15분 이상 걸리며 일시 중지하거나 중지할 수 없습니다.

>**검토**: 이 연습에서는 Azure Stack Hub 인프라 백업을 호스트할 공유, 그리고 Azure Stack Hub 인프라 백업에서 해당 공유에 연결하는 데 사용할 Active Directory 사용자 계정을 만들었습니다. 또한 암호화 키를 생성하고 백업 컨트롤러를 구성했습니다. 
