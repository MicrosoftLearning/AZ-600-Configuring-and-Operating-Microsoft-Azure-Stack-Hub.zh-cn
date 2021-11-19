---
lab:
    title: '랩: Azure Gallery Packager를 사용하여 사용자 지정 Marketplace 항목 추가'
    module: '모듈 2: 서비스 제공'
---

# 랩 - Azure Gallery Packager를 사용하여 사용자 지정 Marketplace 항목 추가
# 학생 랩 매뉴얼

## 랩 종속성

- 없음 

## 예상 소요 시간

45분

## 랩 시나리오

여러분은 Azure Stack Hub 환경의 운영자입니다. Azure Gallery Packager 도구를 사용하여 사용자 지정 Azure Stack Marketplace 항목을 만들어야 합니다.

## 목표

이 랩을 완료하면 다음을 수행할 수 있습니다.

- Azure Gallery Packager를 사용하여 사용자 지정 Azure Stack Hub Marketplace 항목 만들기

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

이 랩을 진행하면서 사용자 지정 Azure Stack Marketplace 항목을 만드는 데 필요한 소프트웨어를 다운로드하여 설치합니다.

### 연습 1: Azure Stack Hub Marketplace 항목 사용자 지정 및 게시

이 연습에서는 Azure Gallery Packager 도구를 사용하여 Azure Marketplace 항목을 사용자 지정하고 게시합니다.

1. Azure Gallery Packager 도구 및 샘플 패키지 다운로드
1. 기존 Azure Gallery Packager 패키지 수정
1. Azure Stack Hub 스토리지 계정에 패키지 업로드 
1. Azure Stack Hub Marketplace에 패키지 게시
1. 게시한 Azure Stack Hub Marketplace 항목의 사용 가능 여부 확인

#### 작업 1: Azure Gallery Packager 도구 및 샘플 패키지 파일 다운로드

이 작업에서는 다음을 수행합니다.

- Azure Gallery Packager 도구 및 샘플 패키지 파일 다운로드

1. 필요한 경우 다음 자격 증명을 사용하여 **AzS-HOST1**에 로그인합니다.

    - 사용자 이름: **AzureStackAdmin@azurestack.local**
    - 암호: **Pa55w.rd1234**

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 웹 브라우저 창을 열고 [Azure Gallery Packager 도구 다운로드 페이지](https://aka.ms/azsmarketplaceitem)로 이동합니다. 그런 후에 **Microsoft Azure Stack Gallery Packaging Tool and Sample 3.0.zip** 보관 파일을 **Downloads** 폴더에 다운로드합니다.
1. 다운로드가 완료되면 **C:\\Downloads** 폴더(필요한 경우 폴더를 만드세요)에 zip 파일 내의 **Packager** 폴더 압축을 풉니다.


#### 작업 2: 기존 Azure Gallery Packager 패키지 수정

이 작업에서는 다음을 수행합니다.

- Windows Server 2019 이미지(Windows Server 2016 이미지 아님)를 사용하도록 기존 Azure Gallery Packager 패키지 수정

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내의 파일 탐색기에서 **C:\\Downloads\\Packager\\Samples for Packager** 폴더로 이동하여 **C:\\Downloads** 폴더에 **Sample.SingleVMWindowsSample.1.0.0.azpkg** 패키지를 복사하고 패키지의 확장명을 **zip**으로 바꿉니다.
1. **C:\\Downloads\\SamplePackage** 폴더(폴더를 먼저 만들어야 함)에 **Sample.SingleVMWindowsSample.1.0.0.zip** 보관 파일에 포함된 내용의 압축을 풉니다.
1. 파일 탐색기에서 **C:\\Downloads\\SamplePackage\\DeploymentTemplates** 폴더로 이동한 다음 메모장에서 **createuidefinition.json** 파일을 엽니다.
1. 앞에서 ASDK 인스턴스에 다운로드한 Windows Server 2019 Datacenter Core 이미지를 참조하도록 이 파일의 **imageReference** 섹션에 포함된 **sku** 매개 변수를 수정합니다.

    ```json
    "imageReference": {
      "publisher": "MicrosoftWindowsServer",
      "offer": "WindowsServer",
      "sku": "2019-Datacenter-Core-smalldisk"
    },
    ```

1. 변경 내용을 저장하고 메모장을 닫습니다.
1. 파일 탐색기에서 **C:\\Downloads\\SamplePackage\\strings** 폴더로 이동한 다음 메모장에서 **resources.resjson** 파일을 엽니다.
1. 키-값 쌍에서 다음 값을 설정하여 **resources.resjson** 파일의 내용을 수정합니다.

    - displayName: **Custom Windows Server 2019 Core VM**
    - publisherDisplayName: **Contoso**
    - summary: **Custom Windows Server 2019 Core VM (small disk)**
    - longSummary: **Custom Contoso Windows Server 2019 Core VM (small disk)**
    - description: **<p>Sample customized Windows Server 2019 Azure Stack Hub VM</p><p>Based on Azure Stack Hub Quickstart template</p>**
    - documentationLink: **Documentation**

    >**참고**: 그러면 다음 콘텐츠가 반환됩니다.

    ```json
    {
      "displayName": "Custom Windows Server 2019 Core VM",
      "publisherDisplayName": "Contoso",
      "summary": "Custom Windows Server 2019 Core VM (small disk)",
      "longSummary": "Custom Contoso Windows Server 2019 Core VM (small disk)",
      "description": "<p>Sample customized Windows Server 2019 Azure Stack Hub VM</p><p>Based on Azure Stack Hub Quickstart template</p>",
      "documentationLink": "Documentation"
    }
    ```

1. 변경 내용을 저장하고 메모장을 닫습니다.
1. 파일 탐색기에서 **C:\\Downloads\\SamplePackage** 폴더로 이동한 다음 메모장에서 **manifest.json** 파일을 엽니다.
1. 키-값 쌍에서 다음 값을 설정하여 **manifest.json** 파일의 내용을 수정합니다.

    - name: **CustomVMWindowsSample**
    - publisher: **Contoso**
    - version: **1.0.1**
    - displayName: **Custom Windows Server 2019 Core VM**
    - publisherDisplayName: **Contoso**
    - publisherLegalName: **Contoso Inc.**
    - uri of the documentation link: **https://docs.contoso.com**

    >**참고**: 그러면 다음 콘텐츠가 반환됩니다(파일의 줄 2부터 시작됨).

    ```json
        "$schema": "https://gallery.azure.com/schemas/2015-10-01/manifest.json#",
        "name": "CustomVMWindowsSample",
        "publisher": "Contoso",
        "version": "1.0.1",
        "displayName": "Custom Windows Server 2019 Core VM",
        "publisherDisplayName": "Contoso",
        "publisherLegalName": "Contoso Inc.",
        "summary": "ms-resource:summary",
        "longSummary": "ms-resource:longSummary",
        "description": "ms-resource:description",
        "longDescription": "ms-resource:description",
	    "uiDefinition": {
		"path": "UIDefinition.json"
	},
        "links": [
            { "displayName": "ms-resource:documentationLink", "uri": "https://docs.contoso.com/" }
        ], 
    ```

1. 변경 내용을 저장하고 메모장을 닫습니다.


#### 작업 3: 사용자 지정된 Azure Gallery Packager 패키지 생성

이 작업에서는 다음을 수행합니다.

- 새로 사용자 지정한 Azure Gallery Packager 패키지 다시 생성

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 **명령 프롬프트**를 시작합니다.

1. **명령 프롬프트**에서 다음 명령을 실행하여 현재 디렉터리를 변경합니다.

    ```cmd
    cd C:\Downloads\Packager
    ```

1. **명령 프롬프트**에서 다음 명령을 실행하여 이전 작업에서 수정한 콘텐츠를 기반으로 새 패키지를 생성합니다.

    ```cmd
    AzureStackHubGallery.exe package -m C:\Downloads\SamplePackage\manifest.json -o C:\Downloads\
    ```

1. **C:\\Downloads** 폴더에 **Contoso.CustomVMWindowsSample.1.0.1.azpkg** 패키지가 자동으로 저장되었는지 확인합니다.


#### 작업 4: Azure Stack Hub 스토리지 계정에 패키지 업로드

이 작업에서는 다음을 수행합니다.

- Azure Stack Hub 스토리지 계정에 Azure Stack Hub Marketplace 항목 패키지 업로드

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 [Azure Stack Hub 관리자 포털](https://adminportal.local.azurestack.external/)이 표시된 웹 브라우저 창을 열고 CloudAdmin@azurestack.local로 로그인합니다.
1. Azure Stack Hub 관리자 포털이 표시된 웹 브라우저 창에서 **+ 리소스 만들기**를 클릭합니다. 
1. **새로 만들기** 블레이드에서 **데이터 + 스토리지**를 클릭합니다.
1. **데이터 + 스토리지** 블레이드에서 **스토리지 계정**을 클릭합니다.
1. **스토리지 계정 만들기** 블레이드의 **기본** 탭에서 다음 설정을 지정합니다.

    - 구독: **기본 공급자 구독**
    - 리소스 그룹: 새 리소스 그룹 **marketplace-pkgs-RG**의 이름.
    - 이름: 소문자나 숫자 3~24자가 포함된 고유한 이름
    - 위치: **로컬**
    - 성능: **표준**
    - 계정 종류: **스토리지(범용 v1)**
    - 복제: **LRS(로컬 중복 스토리지)**

1. **스토리지 계정 만들기** 블레이드의 **기본** 탭에서 **다음: 고급 >** 을 클릭합니다.
1. **스토리지 계정 만들기** 블레이드의 **고급** 탭에서 기본 설정을 유지하고 **검토 + 만들기**를 클릭합니다.
1. **스토리지 계정 만들기**의 **검토 + 만들기** 탭에서 **만들기**를 클릭합니다.

    >**참고**: 스토리지 계정이 프로비전될 때까지 기다립니다. 1분 정도 걸립니다.

1. Azure Stack Hub 관리자 포털이 표시된 웹 브라우저 창의 허브 메뉴에서 **리소스 그룹**을 선택합니다.
1. **리소스 그룹** 블레이드의 리소스 그룹 목록에서 **marketplace-pkgs-RG** 항목을 클릭합니다.
1. **marketplace-pkgs-RG** 블레이드에서 새로 만든 스토리지 계정을 나타내는 항목을 클릭합니다.
1. 스토리지 계정 블레이드에서 **컨테이너**를 클릭합니다.
1. 컨테이너 블레이드에서 **+ 컨테이너**를 클릭합니다.
1. **새 컨테이너** 블레이드의 **이름** 텍스트 상자에 **gallerypackages**를 입력합니다. 그런 다음 **공용 액세스 수준** 드롭다운 목록에서 **Blob(Blob에 대한 익명 읽기 전용 액세스)** 를 선택하고 **만들기**를 클릭합니다.
1. 컨테이너 블레이드로 돌아와서 새로 만든 컨테이너를 나타내는 **gallerypackages** 항목을 클릭합니다.
1. **gallerypackages** 블레이드에서 **업로드**를 클릭합니다.
1. **Blob 업로드** 블레이드에서 **파일 선택** 텍스트 상자 옆의 폴더 아이콘을 클릭합니다. 
1. **열기** 대화 상자에서 **C:\Downloads** 폴더로 이동하여 **Contoso.CustomVMWindowsSample.1.0.1.azpkg** 패키지 파일을 선택하고 **열기**를 클릭합니다.
1. **Blob 업로드** 블레이드로 돌아와서 **업로드**를 클릭합니다.


#### 작업 5: Azure Stack Hub Marketplace에 패키지 게시

이 작업에서는 다음을 수행합니다.

- Azure Stack Hub Marketplace에 패키지 게시

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 관리자로 PowerShell 7을 시작합니다.
1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내의 **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 이 랩에 필요한 Azure Stack Hub PowerShell 모듈을 설치합니다. 

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Install-Module -Name Az.BootStrapper -Force -AllowPrerelease -AllowClobber
    Install-AzProfile -Profile 2019-03-01-hybrid -Force
    Install-Module -Name AzureStack -RequiredVersion 2.0.2-preview -AllowPrerelease
    ```

    >**참고**: 이미 사용 가능한 명령 관련 오류 메시지는 무시하세요.

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 Azure Stack Hub 운영자 환경을 등록합니다.

    ```powershell
    Add-AzEnvironment -Name 'AzureStackAdmin' -ArmEndpoint 'https://adminmanagement.local.azurestack.external' `
       -AzureKeyVaultDnsSuffix adminvault.local.azurestack.external `
       -AzureKeyVaultServiceEndpointResourceId https://adminvault.local.azurestack.external
    ```

1. **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 프롬프트에서 다음 명령을 실행하여 AzureStack\CloudAdmin 자격 증명으로 Azure Stack Hub 운영자 PowerShell 환경에 로그인합니다. Azure Stack Hub 관리자 포털이 실행 중인 기존 브라우저 세션을 활용하면 됩니다.

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureStackAdmin'
    ```

    >**참고**: 그러면 다른 브라우저 탭이 자동으로 열리고 인증 성공 관련 알림 메시지가 표시됩니다.

1. 브라우저 탭을 닫고 **관리자: C:\Program Files\PowerShell\7\pwsh.exe** 창으로 다시 전환하여 **CloudAdmin@azurestack.local**로 정상 인증되었는지 확인합니다.
1. 관리자: **관리자: C:\Program Files\PowerShell\7\pwsh.exe**에서 다음 명령을 실행하여 Azure Stack Hub Marketplace에 패키지를 게시합니다(여기서 `<storage_account_name>`은 이전 작업에서 할당한 스토리지 계정의 이름을 나타냄).

    ```powershell
    Add-AzsGalleryItem -GalleryItemUri `
    https://<스토리지 계정 이름>.blob.local.azurestack.external/gallerypackages/Contoso.CustomVMWindowsSample.1.0.1.azpkg -Verbose
    ```

1. **Add-AzsGalleryItem** 명령의 출력을 검토하여 명령이 정상적으로 완료되었는지 확인합니다.


#### 작업 6: 게시한 Azure Stack Hub Marketplace 항목의 사용 가능 여부 확인

이 작업에서는 다음을 수행합니다.

- 게시한 Azure Stack Hub Marketplace 항목의 사용 가능 여부 확인

1. **AzS-HOST1**에 연결된 원격 데스크톱 세션 내에서 [Azure Stack Hub 사용자 포털](https://portal.local.azurestack.external)이 표시된 웹 브라우저 창을 엽니다.
1. Azure Stack Hub 관리자 포털이 표시된 웹 브라우저 창에서 **Marketplace**를 클릭합니다. 
1. **Marketplace** 블레이드에서 **컴퓨팅**을 클릭한 다음 **자세히 보기**를 클릭합니다.
1. **Marketplace** 블레이드에서 선택 항목 목록에 **사용자 지정 Windows Server 2019 Core VM**이 표시되어 있는지 확인합니다. 

    >**참고**: 새 Azure Stack Hub Marketplace 항목이 정상 작동하도록 하려면 템플릿이 참조하는 OS 이미지 등 해당 항목의 배포에 필요한 모든 필수 구성 요소도 갖춰져 있는지를 확인해야 합니다. 이 랩에서는 해당 작업에 대해 다루지 않습니다.

>**검토**: 이 연습에서는 Azure Gallery Packager를 사용하여 사용자 지정 Azure Stack Hub Marketplace 항목을 만든 후 **Add-AzsGalleryItem** cmdlet을 사용하여 해당 항목을 게시했습니다.
