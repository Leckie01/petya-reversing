# 실습 환경 설정 가이드

## 📚 목차

- [개요](#-개요)
- [가상 머신 설정](#-가상-머신-설정)
- [분석 도구 설치](#-분석-도구-설치)
- [안전한 악성코드 분석 환경 구축](#-안전한-악성코드-분석-환경-구축)
- [분석 전 체크리스트](#-분석-전-체크리스트)
- [문제 해결](#-문제-해결)

---

## 🎯 개요

악성코드 분석은 **반드시 격리된 환경**에서 수행해야 합니다. 이 가이드는 안전한 분석 환경을 구축하는 방법을 단계별로 설명합니다.

### 필요 사항

#### 하드웨어 요구사항
- **CPU**: Intel VT-x 또는 AMD-V 지원 (가상화 기술)
- **RAM**: 최소 8GB (권장 16GB 이상)
- **디스크**: 100GB 이상 여유 공간
- **네트워크**: 인터넷 연결 (초기 설정용)

#### 소프트웨어 요구사항
- **호스트 OS**: Windows 10/11, macOS, Linux
- **가상화 소프트웨어**: VirtualBox 또는 VMware
- **게스트 OS**: Windows 10 ISO (분석 대상에 따라)

### 환경 구축 개요

```
┌─────────────────────────────────────────┐
│          호스트 OS (물리 머신)             │
│  ┌─────────────────────────────────┐   │
│  │   가상화 소프트웨어 (VirtualBox)  │   │
│  │  ┌─────────────────────────┐    │   │
│  │  │  게스트 OS (Windows 10)  │    │   │
│  │  │  ┌─────────────────┐    │    │   │
│  │  │  │  분석 도구들     │    │    │   │
│  │  │  │  - Ghidra       │    │    │   │
│  │  │  │  - x64dbg       │    │    │   │
│  │  │  │  - ProcMon      │    │    │   │
│  │  │  └─────────────────┘    │    │   │
│  │  │                         │    │   │
│  │  │  ⚠️ 네트워크 격리       │    │   │
│  │  └─────────────────────────┘    │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

---

## 💻 가상 머신 설정

### Option 1: VirtualBox (무료, 추천)

#### 1. VirtualBox 다운로드 및 설치

**다운로드:**
- 웹사이트: https://www.virtualbox.org/
- Downloads > Windows hosts (또는 본인 OS)
- 최신 버전 다운로드 (예: 7.0.x)

**설치:**
```
1. 다운로드한 설치 파일 실행
2. "Next" 클릭하여 기본 설정으로 진행
3. 네트워크 어댑터 설치 경고 시 "Yes" 클릭
4. Install 클릭
5. 설치 완료 후 "Finish"
```

**Extension Pack 설치 (선택사항):**
```
1. VirtualBox 웹사이트에서 Extension Pack 다운로드
2. VirtualBox 실행 > Tools > Extensions
3. Install 버튼 > 다운로드한 .vbox-extpack 파일 선택
4. 라이선스 동의 후 설치
```

#### 2. Windows 10 ISO 다운로드

**정품 ISO 다운로드:**
```
1. https://www.microsoft.com/software-download/windows10
2. "Download tool now" 클릭
3. Media Creation Tool 실행
4. "Create installation media" 선택
5. 언어, 에디션, 아키텍처 선택
   - Language: Korean
   - Edition: Windows 10
   - Architecture: 64-bit (x64)
6. ISO file 선택
7. 저장 위치 지정 후 다운로드 대기
```

**대안 (평가판):**
- https://www.microsoft.com/en-us/evalcenter/
- Windows 10 Enterprise 90일 평가판

#### 3. 가상 머신 생성

**Step 1: 새 VM 생성**
```
1. VirtualBox 실행
2. "새로 만들기" (New) 클릭
3. 설정:
   - 이름: "MalwareAnalysis-Win10"
   - 폴더: 충분한 공간이 있는 위치
   - ISO 이미지: 다운로드한 Windows 10 ISO 선택
   - Type: Microsoft Windows
   - Version: Windows 10 (64-bit)
4. "다음" 클릭
```

**Step 2: 하드웨어 설정**
```
- Base Memory: 4096 MB (4GB) 이상 권장
- Processors: 2 CPU 이상 권장
- "다음" 클릭
```

**Step 3: 가상 하드 디스크**
```
- Create a Virtual Hard Disk Now
- Hard Disk Size: 60GB 이상
- "다음" 클릭
```

**Step 4: 요약 확인 및 완성**
```
- 설정 확인
- "완료" 클릭
```

#### 4. VM 추가 설정

**시스템 설정:**
```
1. VM 선택 > 설정 (Settings)
2. System > Motherboard:
   ✓ Floppy 체크 해제
   ✓ Boot Order: Hard Disk 최상위
   
3. System > Processor:
   ✓ Enable PAE/NX
   
4. System > Acceleration:
   ✓ Enable VT-x/AMD-V (필수)
   ✓ Enable Nested Paging
```

**디스플레이 설정:**
```
Display > Screen:
- Video Memory: 128 MB
- Graphics Controller: VBoxSVGA
✓ Enable 3D Acceleration (선택사항)
```

**저장소 설정:**
```
Storage:
- Controller: SATA에 ISO 이미지 연결 확인
- 설치 후 ISO 제거 가능
```

**네트워크 설정 (중요!):**
```
Network > Adapter 1:
- Enable Network Adapter: ✓
- Attached to: Host-only Adapter
  또는
- Attached to: Internal Network
  
⚠️ NAT나 Bridged는 사용하지 마세요!
   호스트와 외부 네트워크 접근을 차단합니다.
```

**공유 폴더 설정:**
```
Shared Folders:
- ⚠️ 공유 폴더는 비활성화 권장
- 샘플 전송은 ISO 이미지나 USB로
```

#### 5. Windows 10 설치

**VM 시작:**
```
1. VM 선택 > 시작 (Start)
2. ISO 부팅 대기
3. Windows 설치 화면 표시
```

**설치 진행:**
```
1. 언어, 시간, 키보드 설정
   - Language: 한국어
   - Time: (UTC+09:00) 서울
   - Keyboard: Korean

2. "지금 설치" 클릭

3. 제품 키:
   - "제품 키가 없습니다" 선택
   - 또는 평가판 키 입력

4. Windows 버전 선택:
   - Windows 10 Pro 권장

5. 라이선스 동의

6. 설치 유형:
   - "사용자 지정: Windows만 설치"

7. 파티션:
   - 전체 디스크 선택 > 다음
   - 자동 파티션 생성

8. 설치 진행 (10-20분 소요)

9. 재부팅 후 초기 설정:
   - 지역: 대한민국
   - 키보드: Microsoft 한글
   - 네트워크: "지금은 건너뛰기" (중요!)
   - 계정: 로컬 계정 생성 (Microsoft 계정 불필요)
   - 사용자 이름: analyst
   - 암호: (선택사항이지만 설정 권장)
   - 개인 정보 설정: 모두 끄기
```

**설치 완료 후:**
```
1. Windows 바탕화면 도달
2. Windows Update 일시 중지 (설정 > 업데이트)
3. Windows Defender 일시 중지 (분석 시 방해됨)
```

#### 6. VirtualBox Guest Additions 설치

**설치 목적:**
- 화면 해상도 자동 조정
- 복사/붙여넣기 기능
- 성능 향상

**설치 방법:**
```
1. VM 실행 중 상태에서
2. Devices > Insert Guest Additions CD image
3. VM 내부에서 CD 드라이브 열기
4. VBoxWindowsAdditions.exe 실행
5. 기본 설정으로 설치
6. 재부팅
```

#### 7. 스냅샷 생성

**중요: 깨끗한 상태 저장**

```
1. VM 실행 중 또는 종료 상태
2. Machine > Take Snapshot
3. 이름: "CleanInstall-Tools-Ready"
4. 설명: "Windows 10 설치 완료, 분석 도구 설치 전"
5. OK 클릭

이제 언제든 이 상태로 복원 가능!
```

**스냅샷 복원:**
```
1. VM 선택 > Snapshots
2. 복원할 스냅샷 선택
3. Restore 클릭
4. 현재 상태를 새 스냅샷으로 저장 여부 선택
```

**스냅샷 전략:**
```
Snapshot 1: "CleanInstall"
  ↓ (Windows 설치 완료)
Snapshot 2: "ToolsInstalled"
  ↓ (모든 분석 도구 설치 완료)
Snapshot 3: "BeforeAnalysis"
  ↓ (분석 시작 전)
[분석 수행]
  ↓ (문제 발생 시)
Restore to "BeforeAnalysis"
```

### Option 2: VMware Workstation Player (무료 for 개인)

#### 1. VMware 다운로드 및 설치

```
1. https://www.vmware.com/products/workstation-player.html
2. Download for Windows (또는 Linux)
3. 설치 파일 실행
4. 개인 사용 라이선스 선택
5. 기본 설정으로 설치
```

#### 2. VM 생성 (VirtualBox와 유사)

```
1. Create a New Virtual Machine
2. Installer disc image file (ISO): Windows 10 ISO 선택
3. Windows 제품 키: 건너뛰기
4. VM 이름 및 위치 지정
5. 디스크 크기: 60GB
6. Customize Hardware:
   - Memory: 4096 MB
   - Processors: 2
   - Network Adapter: Host-only
7. Finish > Power On
```

#### 3. VMware Tools 설치

```
1. VM > Install VMware Tools
2. Setup 실행
3. 기본 설정으로 설치
4. 재부팅
```

---

## 🛠️ 분석 도구 설치

분석 도구들을 VM 내부에 설치합니다.

### 필수 도구 설치 순서

#### 1. Windows 기본 설정

**Windows Defender 비활성화:**
```
1. 설정 > 업데이트 및 보안 > Windows 보안
2. 바이러스 및 위협 방지
3. 설정 관리
4. 실시간 보호 끄기
5. 클라우드 제공 보호 끄기
6. 자동 샘플 제출 끄기
```

**파일 확장자 표시:**
```
1. 파일 탐색기 열기
2. 보기 > 옵션
3. 보기 탭
4. "알려진 파일 형식의 파일 확장명 숨기기" 체크 해제
```

**숨김 파일 표시:**
```
파일 탐색기 > 보기:
✓ 숨김 항목
```

#### 2. Ghidra 설치

**필요 사항: Java JDK**

**JDK 설치:**
```
1. https://adoptium.net/
2. Temurin 17 (LTS) 다운로드
3. Windows x64 .msi 설치
4. 기본 경로로 설치: C:\Program Files\Eclipse Adoptium\
```

**Ghidra 설치:**
```
1. https://ghidra-sre.org/
2. Latest Release 다운로드 (예: ghidra_10.x_PUBLIC.zip)
3. C:\Tools\ghidra 에 압축 해제
4. ghidraRun.bat 실행하여 테스트
```

**첫 실행:**
```
1. ghidraRun.bat 더블클릭
2. 프로젝트 생성:
   - File > New Project
   - Non-Shared Project
   - 프로젝트 위치: C:\Analysis
3. Ghidra 사용 가능!
```

#### 3. x64dbg 설치

**다운로드:**
```
1. https://x64dbg.com/
2. Download Snapshot 선택
3. .zip 파일 다운로드
```

**설치:**
```
1. C:\Tools\x64dbg 에 압축 해제
2. x96dbg.exe 실행 (32/64비트 자동 선택)
   또는
   - x32\x32dbg.exe (32비트 전용)
   - x64\x64dbg.exe (64비트 전용)
```

**플러그인 설치 (ScyllaHide - 안티디버깅 우회):**
```
1. https://github.com/x64dbg/ScyllaHide
2. Release 페이지에서 최신 버전 다운로드
3. x64dbg 폴더에 압축 해제
4. x64dbg 재시작
5. Plugins > ScyllaHide > Options에서 활성화
```

#### 4. PEStudio 설치

**다운로드:**
```
1. https://www.winitor.com/download
2. PEStudio 최신 버전 다운로드
```

**설치:**
```
1. C:\Tools\pestudio 에 압축 해제
2. pestudio.exe 실행
3. 첫 실행 시 서명 데이터베이스 업데이트
```

#### 5. PE-bear 설치

**다운로드:**
```
1. https://github.com/hasherezade/pe-bear-releases
2. Releases에서 최신 버전 다운로드
```

**설치:**
```
1. C:\Tools\PE-bear 에 압축 해제
2. PE-bear.exe 실행
```

#### 6. Detect It Easy (DIE) 설치

**다운로드:**
```
1. https://github.com/horsicq/Detect-It-Easy
2. Releases에서 Windows 버전 다운로드
```

**설치:**
```
1. C:\Tools\die 에 압축 해제
2. die.exe 또는 diec.exe(콘솔) 실행
```

#### 7. Process Monitor 설치

**다운로드:**
```
1. https://docs.microsoft.com/sysinternals/downloads/procmon
2. Process Monitor 다운로드
```

**설치:**
```
1. Zip 파일 압축 해제
2. Procmon.exe 실행 (관리자 권한)
3. 드라이버 자동 설치
4. 처음 실행 시 라이선스 동의
```

#### 8. Process Hacker 설치

**다운로드:**
```
1. https://processhacker.sourceforge.io/
2. 설치 파일 다운로드
```

**설치:**
```
1. 설치 프로그램 실행
2. C:\Program Files\Process Hacker 2 에 설치
3. 관리자 권한으로 실행 설정
```

#### 9. Wireshark 설치

**다운로드:**
```
1. https://www.wireshark.org/download.html
2. Windows Installer (64-bit) 다운로드
```

**설치:**
```
1. 설치 프로그램 실행
2. 기본 설정으로 진행
3. Npcap 설치 (프롬프트 시)
4. 설치 완료
```

#### 10. CFF Explorer 설치

**다운로드:**
```
1. https://ntcore.com/?page_id=388
2. Explorer Suite 다운로드
```

**설치:**
```
1. 설치 프로그램 실행
2. C:\Program Files\NTCore\Explorer Suite 에 설치
3. CFF Explorer.exe 실행
```

#### 11. HxD (Hex Editor) 설치

**다운로드:**
```
1. https://mh-nexus.de/en/hxd/
2. 설치 파일 다운로드
```

**설치:**
```
1. 설치 프로그램 실행
2. 기본 경로로 설치
```

#### 12. 추가 유틸리티

**Python 설치 (스크립팅용):**
```
1. https://www.python.org/downloads/
2. Python 3.x 최신 버전
3. ✓ Add Python to PATH
4. Install
```

**유용한 Python 패키지:**
```cmd
pip install pefile
pip install capstone
pip install yara-python
pip install python-magic-bin
```

### 도구 폴더 구조 (권장)

```
C:\Tools\
├── ghidra\
├── x64dbg\
├── pestudio\
├── PE-bear\
├── die\
├── procmon\
├── strings\
└── [기타 도구들]

C:\Analysis\
├── samples\      (분석할 샘플)
├── dumps\        (메모리 덤프)
├── pcaps\        (네트워크 캡처)
└── notes\        (분석 노트)
```

---

## 🔒 안전한 악성코드 분석 환경 구축

### 1. 네트워크 격리 설정

#### VirtualBox 네트워크 설정

**Host-only 네트워크 생성:**
```
1. VirtualBox > 파일 > 호스트 네트워크 관리자
2. 만들기 버튼 클릭
3. 어댑터 설정:
   - IPv4 Address: 192.168.56.1
   - IPv4 Network Mask: 255.255.255.0
   - DHCP 서버: 비활성화 권장
4. 적용
```

**VM 네트워크 어댑터 설정:**
```
1. VM 선택 > 설정 > 네트워크
2. 어댑터 1:
   ✓ Enable Network Adapter
   - Attached to: Host-only Adapter
   - Name: 위에서 생성한 어댑터 선택
3. 어댑터 2, 3, 4: 비활성화
```

**완전 격리 (인터넷 차단):**
```
- Attached to: Not attached
또는
- Attached to: Internal Network
  Name: isolated
```

#### 네트워크 검증

VM 내부에서 테스트:
```cmd
# 외부 연결 테스트
ping google.com
> Request timed out (정상 - 차단됨)

# 호스트 연결 테스트
ping 192.168.56.1
> Reply from... (정상 - 호스트와만 통신)

# DNS 조회 테스트
nslookup google.com
> DNS request timed out (정상 - 차단됨)
```

### 2. 공유 폴더 비활성화

**중요**: 악성코드가 호스트로 전파되는 것을 방지

```
VM 설정 > 공유 폴더:
- 모든 공유 폴더 제거
- Auto-mount 비활성화
```

**파일 전송 대안:**
```
1. ISO 이미지 생성하여 마운트
2. 네트워크를 통한 SCP/FTP (일시적 활성화)
3. USB 패스스루 (주의해서 사용)
```

### 3. 클립보드 및 드래그앤드롭 비활성화

```
VM 설정 > 일반 > 고급:
- Shared Clipboard: Disabled
- Drag'n'Drop: Disabled
```

### 4. VM 스냅샷 전략

**분석 전:**
```
1. 깨끗한 상태 스냅샷: "BeforeAnalysis"
2. 도구 설정 완료 스냅샷: "ToolsReady"
```

**분석 중:**
```
- 각 샘플 분석 전 새 스냅샷 생성
- 이름 규칙: "Before-[샘플명]-[날짜]"
```

**분석 후:**
```
- 중요 발견이 있으면 스냅샷 보존
- 일반적으로는 초기 상태로 복원
```

### 5. 안전 수칙 체크리스트

**분석 시작 전 확인:**
```
✓ VM이 스냅샷으로 복원 가능한가?
✓ 네트워크가 격리되어 있는가?
✓ 공유 폴더가 비활성화되어 있는가?
✓ 호스트에 중요 데이터가 백업되어 있는가?
✓ Windows Defender가 비활성화되어 있는가?
✓ 분석 도구들이 정상 작동하는가?
```

**분석 중 주의사항:**
```
⚠️ VM 외부로 샘플을 복사하지 마세요
⚠️ 실제 계정 정보를 입력하지 마세요
⚠️ 중요한 문서를 VM에 저장하지 마세요
⚠️ VM과 호스트를 혼동하지 마세요
⚠️ 네트워크를 활성화하지 마세요
```

**분석 후 정리:**
```
1. 중요 발견사항을 호스트에 문서화
2. IOC를 별도로 저장
3. 스크린샷을 호스트에 복사
4. VM을 초기 스냅샷으로 복원
5. 또는 VM 삭제 후 재생성
```

---

## ✅ 분석 전 체크리스트

### 환경 준비

```
[ ] 가상 머신이 정상 부팅되는가?
[ ] 모든 분석 도구가 설치되어 있는가?
[ ] 도구들이 정상 실행되는가?
[ ] 깨끗한 상태의 스냅샷이 있는가?
```

### 네트워크 설정

```
[ ] VM 네트워크가 격리되어 있는가?
[ ] ping google.com이 실패하는가?
[ ] 호스트와 통신은 가능한가? (필요 시)
[ ] FakeNet 또는 INetSim이 준비되었는가? (고급)
```

### 보안 설정

```
[ ] 공유 폴더가 비활성화되어 있는가?
[ ] 클립보드 공유가 비활성화되어 있는가?
[ ] Drag & Drop이 비활성화되어 있는가?
[ ] Windows Defender가 비활성화되어 있는가?
```

### 분석 준비

```
[ ] 분석 노트 템플릿이 준비되었는가?
[ ] 스크린샷 도구가 있는가?
[ ] 분석할 샘플이 VM에 있는가?
[ ] 샘플의 해시가 기록되었는가?
```

### 도구 실행 테스트

```
[ ] Ghidra가 정상 실행되는가?
[ ] x64dbg가 정상 실행되는가?
[ ] PEStudio가 정상 실행되는가?
[ ] Process Monitor가 정상 실행되는가?
[ ] Wireshark가 정상 실행되는가?
```

---

## 🔧 문제 해결

### 일반적인 문제

#### 1. 가상화 기능이 비활성화됨

**증상:**
```
"VT-x/AMD-V 하드웨어 가속을 사용할 수 없습니다"
```

**해결:**
```
1. BIOS/UEFI 진입 (재부팅 시 F2, Del, F12 등)
2. Advanced > CPU Configuration
3. Intel Virtualization Technology 또는 AMD-V 활성화
4. 저장 후 재부팅
```

**Windows에서 Hyper-V와 충돌:**
```
관리자 권한 CMD:
bcdedit /set hypervisorlaunchtype off
재부팅
```

#### 2. VM이 느림

**원인:**
- RAM 부족
- CPU 코어 부족
- 디스크 I/O 병목

**해결:**
```
1. VM 설정에서 RAM 증가 (최소 4GB)
2. CPU 코어 2개 이상 할당
3. SSD에 VM 저장
4. 불필요한 서비스 비활성화 (VM 내부)
```

#### 3. 네트워크가 완전히 작동하지 않음

**Host-only 네트워크 문제:**
```
1. VirtualBox > 파일 > 호스트 네트워크 관리자
2. 어댑터 제거 후 재생성
3. VM 네트워크 어댑터 재설정
```

**Windows 방화벽 문제:**
```
호스트에서:
1. 제어판 > Windows Defender 방화벽
2. 고급 설정
3. 인바운드 규칙 > VirtualBox 허용
```

#### 4. Guest Additions가 설치되지 않음

**해결:**
```
1. VM 내부에서 Windows Update 수행
2. 재부팅
3. Devices > Insert Guest Additions CD
4. 수동으로 CD 열기
5. VBoxWindowsAdditions.exe 실행
6. 재부팅
```

#### 5. 도구가 실행되지 않음

**Ghidra가 실행 안 됨:**
```
- Java JDK 설치 확인
- 환경 변수 JAVA_HOME 설정
- 경로에 공백 없는지 확인
```

**x64dbg가 실행 안 됨:**
```
- VC++ Redistributable 설치
- https://aka.ms/vs/17/release/vc_redist.x64.exe
```

### 성능 최적화

**VM 성능 향상 팁:**
```
1. 3D 가속 비활성화 (안정성 향상)
2. 비디오 메모리 128MB
3. 고정 크기 가상 디스크 사용
4. 호스트 캐시 I/O 활성화
5. Paravirtualization: KVM (Linux) 또는 Hyper-V (Windows)
```

**Windows 최적화 (VM 내부):**
```
1. 시각 효과 끄기:
   - 시스템 > 고급 시스템 설정
   - 성능 옵션 > 최고 성능
   
2. 불필요한 서비스 비활성화:
   - Windows Search
   - Superfetch
   - Windows Update (일시)
   
3. 페이지 파일 최소화
```

---

## 📝 다음 단계

환경 설정을 완료했다면:

1. 📖 [리버싱 기본 개념](reversing-basics.md) 복습
2. 📋 [단계별 분석 체크리스트](analysis-steps.md) 확인
3. 🔍 [Petya 분석 가이드](petya-analysis-guide.md) 실습 시작

---

## 🔗 관련 리소스

- [도구 목록](../resources/tools.md)
- [참고 자료](../resources/references.md)
- [분석 로그 템플릿](../notes/analysis-log-template.md)

---

**💡 팁**: 항상 깨끗한 스냅샷을 유지하고, 분석 전후로 복원하는 습관을 들이세요!
