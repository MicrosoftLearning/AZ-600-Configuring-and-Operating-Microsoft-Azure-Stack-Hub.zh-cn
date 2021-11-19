---
lab:
    title: '랩: Azure Stack Hub에서 공용 IP 주소 관리'
    module: '모듈 5: 인프라 관리'
---

# 랩 - Azure Stack Hub에서 공용 IP 주소 관리
# 학생 랩 매뉴얼

## 랩 종속성

- 없음

## 예상 소요 시간

30분

## 랩 시나리오

여러분은 Azure Stack Hub 환경의 운영자입니다. 공용 IP 주소 리소스를 관리해야 합니다. 

## 목표

이 랩을 완료하면 다음을 수행할 수 있습니다.

- 공용 IP 주소 리소스 관리

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

이 랩을 진행하면서 PowerShell을 통해 Azure Stack Hub를 관리하는 데 필요한 소프트웨어를 설치합니다. 그리고 사용자 계정을 추가로 만듭니다.

## 지침

### 연습 0: 랩 준비

이 연습에서는 이 랩에서 사용할 Active Directory 사용자 계정을 만듭니다.

1. 사용자 계정 만들기(클라우드 운영자 역할)

#### 작업 1: 사용자 계정 만들기(클라우드 운영자 역할)

이 작업에서는 다음을 수행합니다.

- 사용자 계정 만들기(클라우드 운영자 역할)

1. 필요한 경우 다음 자격 증명을 사용하여 **AzS-HOST1**에 로그인합니다.

    - 사용자 이름: **AzureStackAdmin@azurestack.local**
    - 암호: **Pa55w.rd1234**

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

>**검토**: 이 연습에서는 이 랩에서 사용할 Active Directory 계정을 만들었습니다.


### 연습 1: 제안 만들기(클라우드 운영자 역할)

이 연습에서는 클라우드 운영자 역할을 맡아 먼저 공용 IP 주소 사용량을 검토한 다음 네트워크 서비스가 포함된 요금제, 그리고 이 요금제가 포함된 제안을 만듭니다. 그런 다음 사용자가 해당 제안을 기준으로 구독을 만들 수 있도록 제안을 공개합니다. 이 연습에서는 다음 작업을 수행합니다.

1. 공용 IP 주소 사용량 검토(클라우드 운영자 역할)
1. 네트워크 서비스가 포함된 요금제 만들기(클라우드 운영자 역할)
1. 요금제를 기준으로 제안을 만들고 공용으로 설정(클라우드 운영자 역할)


#### 작업 1: 공용 IP 주소 사용량 검토(클라우드 운영자 역할)

이 작업에서는 다음을 수행합니다.

- 공용 IP 주소 사용량 검토(클라우드 운영자 역할)

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 [Azure Stack Hub 관리자 포털](https://adminportal.local.azurestack.external/)이 표시된 웹 브라우저 창을 열고 CloudAdmin@azurestack.local로 로그인합니다.
1. Azure Stack Hub 사용자 포털의 **대시보드** 페이지 **리소스 공급자** 타일에서 **네트워크**를 클릭합니다. 
1. **네트워크** 블레이드에서 **공용 IP 풀 사용량** 그래프, 그리고 사용 중인 IP 주소 및 사용 가능한 IP 주소 수를 확인합니다.

#### 작업 2: 네트워크 서비스가 포함된 요금제 만들기(클라우드 운영자 역할)

이 작업에서는 다음을 수행합니다.

- 네트워크 서비스가 포함된 요금제 만들기(클라우드 운영자 역할)

1. Azure Stack Hub 관리자 포털이 표시된 웹 브라우저 창에서 **+ 리소스 만들기**를 클릭합니다. 
1. **새로 만들기** 블레이드에서 **제안+요금제**를 클릭합니다.
1. **제안+요금제** 블레이드에서 **요금제**를 클릭합니다.
1. **새 요금제** 블레이드의 **기본** 탭에서 다음 설정을 지정합니다.

    - 표시 이름: **Network-plan1**
    - 리소스 이름: **network-plan1**
    - 리소스 그룹: 새 리소스 그룹 **network-plans-RG**의 이름.

1. **다음: 서비스 >** 를 클릭합니다.
1. **새 요금제** 블레이드의 **서비스** 탭에서 **Microsoft.Network** 체크박스를 선택합니다.
1. **다음: 할당량>** 을 클릭합니다.
1. **새 요금제** 블레이드의 **할당량** 탭에서 **새로 만들기**를 선택합니다.
1. **네트워크 할당량 만들기** 블레이드에서 다음 설정을 지정하고 **확인**을 클릭합니다.

    - 이름: **Network-plan1-quota**
    - 최대 가상 네트워크 수: **2**
    - 최대 가상 네트워크 게이트웨이 수: **2**
    - 최대 네트워크 연결 수: **2**
    - 최대 공용 IP 수: **20**
    - 최대 NIC 수: **20**
    - 최대 부하 분산 장치 수: **5**
    - 최대 네트워크 보안 그룹 수: **20**

1. **검토 + 만들기**와 **만들기**를 차례로 클릭합니다.

    >**참고**: 배포가 완료될 때까지 기다립니다. 몇 초면 끝납니다.


#### 작업 3: 요금제를 기준으로 제안 만들기(클라우드 운영자 역할)

이 작업에서는 다음을 수행합니다.

- 요금제를 기준으로 제안 만들기(클라우드 운영자 역할)

1. Azure Stack Hub 관리자 포털에서 **+ 리소스 만들기**를 클릭합니다. 
1. **새로 만들기** 블레이드에서 **제안+요금제**를 클릭합니다.
1. **제안+요금제** 블레이드에서 **제안**을 클릭합니다.
1. **새 제안 만들기** 블레이드의 **기본** 탭에서 다음 설정을 지정합니다.

    - 표시 이름: **Network-offer1**
    - 리소스 이름: **network-offer1**
    - 리소스 그룹: 새 리소스 그룹 **network-offers-RG**
    - 이 제안을 공개로 설정: **예**

1. **다음: 기본 요금제 >** 를 클릭합니다. 
1. **새 제안 만들기** 블레이드의 **기본 요금제** 탭에서 **Network-plan1** 항목 옆의 체크박스를 선택합니다.
1. **다음: 추가 요금제 >** 를 클릭합니다.
1. **추가 요금제** 설정은 기본값으로 유지하고 **검토 + 만들기**를 클릭한 다음 **만들기**를 클릭합니다.

    >**참고**: 배포가 완료될 때까지 기다립니다. 몇 초면 끝납니다.

>**검토**: 이 연습에서는 요금제, 그리고 해당 요금제를 기반으로 하는 공용 제안을 만들었습니다.


### 연습 2: 공용 IP 주소 리소스 만들기(사용자 역할)

이 연습에서는 사용자 역할을 맡아 첫 번째 연습에서 만든 제안에 등록하고 새 구독을 만든 다음 해당 구독에서 공용 IP 주소 리소스를 만듭니다. 이 연습에서는 다음 작업을 수행합니다.

1. 제안에 등록(사용자 역할)
1. Azure Stack Hub 사용자 Azure Resource Manager 엔드포인트에 연결(사용자 역할)
1. IP 주소 리소스 만들기(사용자 역할)

#### 작업 1: 제안에 등록(사용자 역할)

이 작업에서는 다음을 수행합니다.

- 제안에 등록(사용자 역할)

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 웹 브라우저의 InPrivate 세션을 시작합니다.
1. 웹 브라우저 창에서 [Azure Stack Hub 사용자 포털](https://portal.local.azurestack.external)로 이동하여 **Pa55w.rd** 암호를 사용해 **t1u1@azurestack.local**로 로그인합니다.
1. Azure Stack Hub 사용자 포털의 대시보드에서 **구독 가져오기**를 클릭합니다.
1. **구독 가져오기** 블레이드의 **표시 이름** 텍스트 상자에 **T1U1-network-subscription1**을 입력합니다.
1. 제안 목록에서 **Network-offer1**을 선택하고 **만들기**를 클릭합니다.
1. **구독이 생성되었습니다. 구독을 사용해서 시작하려면 포털을 새로 고쳐야 합니다.** 메시지가 표시되면 **새로 고침**을 클릭합니다.


#### 작업 2: Azure Stack Hub Azure Resource Manager 사용자 엔드포인트에 연결(사용자 역할)

이 작업에서는 다음을 수행합니다.

- Azure Stack Hub Azure Resource Manager 사용자 엔드포인트에 연결(사용자 역할)

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 관리자로 PowerShell 7을 시작합니다.

    >**참고**: Azure Stack Hub로의 PowerShell 연결을 설정하는 자세한 지침은 **PowerShell을 통해 Azure Stack Hub에 연결** 랩의 지침을 참조하세요.

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 이 랩에 필요한 Azure Stack Hub PowerShell 모듈을 설치합니다.

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Install-Module -Name Az.BootStrapper -Force -AllowPrerelease -AllowClobber
    Install-AzProfile -Profile 2019-03-01-hybrid -Force
    Install-Module -Name AzureStack -RequiredVersion 2.0.2-preview -AllowPrerelease
    ```

    >**참고**: 이미 사용 가능한 명령 관련 오류 메시지는 무시하세요.

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 Azure Stack Hub 도구를 다운로드한 후 압축을 풉니다.

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Set-Location -Path 'C:\'
    Invoke-WebRequest https://github.com/Azure/AzureStack-Tools/archive/az.zip -OutFile az.zip
    Expand-Archive az.zip -DestinationPath . -Force
    Set-Location -Path '\AzureStack-Tools-az'
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 Azure Stack Hub 사용자 환경을 등록합니다.

    ```powershell
    Add-AzEnvironment -Name 'AzureStackUser' -ArmEndpoint 'https://management.local.azurestack.external'
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 브라우저 세션을 통해 **t1u1@azurestack.local** 사용자로 Azure Stack Hub 사용자 환경에 대한 인증을 시작합니다.

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureStackUser' -UseDeviceAuthentication
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창에 표시되는 메시지를 검토합니다. 그런 후에 InPrivate 모드에서 다른 웹 브라우저 창을 열고 [adfs.local.azurestack.external](https://adfs.local.azurestack.external/adfs/oauth2/deviceauth) 페이지로 이동하여 검토한 메시지에 포함된 코드를 입력합니다. 메시지가 표시되면 **t1u1@azurestack.local** 사용자로 다시 로그인합니다.
1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창으로 다시 전환하여 **t1u1@azurestack.local** 사용자로 정상 인증되었는지 확인합니다.


#### 작업 3: IP 주소 리소스 만들기(사용자 역할)

이 작업에서는 다음을 수행합니다.

- IP 주소 리소스 만들기(사용자 역할)

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 새로 프로비전한 구독을 사용 중인지 확인합니다. 

    ```powershell
    (Get-AzSubscription).Name
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 현재 구독 내에서 네트워크 리소스 공급자를 등록합니다.

    ```powershell 
    Register-AzResourceProvider -ProviderNamespace Microsoft.Network
    ```

    >**참고**: 리소스 공급자가 관리하는 리소스를 만들려면 해당 공급자를 등록해야 합니다.

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 공용 IP 주소 리소스를 호스트할 리소스 그룹을 만듭니다.

    ```powershell 
    $rg = New-AzResourceGroup -Name publicIPs-RG -Location local
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 공용 IP 주소 리소스를 만듭니다. 

    ```powershell
    1..5 | ForEach-Object {New-AzPublicIpAddress -Name "publicIP$_" -ResourceGroupName $rg.ResourceGroupName -AllocationMethod Static -Location local}
    ```

1. 모든 IP 주소 리소스가 프로비전될 때까지 기다립니다.

>**검토**: 이 연습에서는 사용자 구독에서 공용 IP 주소 리소스를 만들었습니다.


### 연습 4: 공용 IP 주소 사용량 관리(클라우드 운영자 역할)

이 연습에서는 클라우드 운영자 역할을 맡아 공용 IP 주소 사용량을 검토 및 관리합니다. 이 연습에서는 다음 작업을 수행합니다.

1. 공용 IP 주소 사용량 검토
2. 공용 IP 주소 풀 추가

#### 작업 1: 공용 IP 주소 사용량 검토(클라우드 운영자 역할)

이 작업에서는 다음을 수행합니다.

- 공용 IP 주소 사용량 검토(클라우드 운영자 역할)

1. Azure Stack Hub 관리자 포털이 표시되어 있으며 CloudAdmin@azurestack.local로 로그인되어 있는 웹 브라우저 창으로 전환합니다.
1. Azure Stack Hub 관리자 포털의 허브 메뉴에서 **대시보드**를 클릭하고 **리소스 공급자** 타일에서 **네트워크**를 클릭합니다.
1. **네트워크** 블레이드에서 **공용 IP 풀 사용량** 그래프, 그리고 사용 중인 IP 주소 및 사용 가능한 IP 주소 수를 다시 검토합니다.

    >**참고**: 사용자 구독(사용자 역할)에서 추가로 만든 공용 IP 주소 5개가 반영되어 IP 주소 수가 변경된 상태여야 합니다.

#### 작업 2: 공용 IP 주소 풀 추가(클라우드 운영자 역할)

이 작업에서는 다음을 수행합니다.

- 공용 IP 주소 풀 추가

1. Azure Stack Hub 관리자 포털이 표시된 웹 브라우저 창의 **네트워크** 블레이드에서 **공용 IP 풀 사용량** 타일을 클릭합니다.

    >**참고**: **공용 IP 풀** 블레이드에 **'시작' 단독 작업이 진행 중입니다. 해당 작업이 진행 중인 동안에는 노드 추가 및 IP 풀 추가 작업이 사용하지 않도록 설정됩니다. 활동 로그를 확인하려면 여기를 클릭하십시오.** 메시지가 표시되면 '시작' 작업이 완료될 때까지 기다렸다가 다음 단계를 진행하세요.

1. **공용 IP 풀** 블레이드에서 **+ IP 풀 추가**를 클릭합니다. 
1. **IP 풀 추가** 블레이드에서 다음 설정을 지정하고 **추가**를 클릭합니다.

    - 이름: **공용 풀 1**
    - 지역: **로컬**
    - 주소 범위(CIDR 블록): **192.168.110.0/24**

1. 변경 내용이 적용될 때까지 기다렸다가 **네트워크** 블레이드로 다시 이동합니다.
1. **공용 IP 풀 사용량** 그래프를 검토하여 사용 가능한 IP 주소 수가 변경되었음을 확인합니다.

>**검토**: 이 연습에서는 공용 IP 주소 풀을 검토 및 구성했습니다.
