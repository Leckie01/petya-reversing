# Petya 랜섬웨어 상세 분석 가이드

## 📚 목차

- [Petya 랜섬웨어 개요](#-petya-랜섬웨어-개요)
- [분석 접근 방법](#-분석-접근-방법)
- [주요 분석 포인트](#-주요-분석-포인트)
- [복구 가능성 분석](#-복구-가능성-분석)
- [탐지 및 방어 전략](#-탐지-및-방어-전략)

---

## 🦠 Petya 랜섬웨어 개요

### 발견 시기 및 배경

**Petya (Green Petya)**
- **발견 시기**: 2016년 3월
- **발견자**: G DATA Security Labs
- **명명**: 영화 "007 골든아이"의 위성 무기 'Petya'에서 유래
- **특징**: MBR(Master Boot Record) 암호화 랜섬웨어의 선구자

**역사적 타임라인:**
```
2016년 3월  - Petya (Green Petya) 최초 발견
2016년 4월  - Mischa 변종 등장 (파일 암호화 추가)
2016년 5월  - GoldenEye 변종
2017년 6월  - NotPetya (ExPetr) 대규모 공격
```

### Petya vs NotPetya (ExPetr) 차이점

| 구분 | Petya (2016) | NotPetya (2017) |
|------|--------------|-----------------|
| **목적** | 금전 취득 | 데이터 파괴 (위장 랜섬웨어) |
| **복구 가능** | 가능 (키 지불 시) | 불가능 |
| **전파 방식** | 이메일, Exploit Kit | EternalBlue, PsExec, WMI |
| **랜섬 노트** | Bitcoin 지갑 제공 | 가짜 Bitcoin 지갑 |
| **암호화 키** | 저장됨 (복구 가능) | 덮어쓰여짐 (복구 불가) |
| **피해 규모** | 중간 | 매우 큼 (전 세계) |
| **공격 대상** | 일반 기업/개인 | 우크라이나 및 다국적 기업 |

⚠️ **중요**: 이 가이드는 주로 원본 Petya (2016)를 다루지만, NotPetya의 차이점도 설명합니다.

### 공격 방식 및 감염 경로

#### 초기 감염 벡터

**1. 이메일 기반 (Dropbox 링크)**
```
공격 흐름:
1. 피싱 이메일 수신 (구직 지원서로 위장)
2. Dropbox 링크 클릭
3. 자가 압축 실행 파일 다운로드
4. 사용자가 실행
```

**이메일 예시:**
```
제목: Job Application - John Smith
내용: 
Hello,
Please find my CV attached via Dropbox:
[Dropbox Link]

Best regards,
John Smith
```

**2. Exploit Kit (Neutrino EK)**
```
- 웹사이트 방문 시 자동 감염
- 브라우저/플러그인 취약점 악용
- Drive-by download
```

**3. NotPetya 추가 전파 방식**
```
- EternalBlue (MS17-010): SMBv1 취약점
- PsExec: 관리 공유를 통한 실행
- WMI: Windows Management Instrumentation
- 자격 증명 탈취 (Mimikatz 유사)
```

### MBR 암호화 메커니즘

**MBR (Master Boot Record)이란?**
```
┌──────────────────────────────────┐
│         물리 디스크               │
├──────────────────────────────────┤
│  Sector 0: MBR (512 bytes)       │ ← Petya가 공격!
│  - Boot Code (446 bytes)         │
│  - Partition Table (64 bytes)    │
│  - Signature (2 bytes: 0x55AA)   │
├──────────────────────────────────┤
│  Partition 1 (C:\)               │
│  - Boot Sector                   │
│  - File System (NTFS/FAT32)      │
│  - Files and Directories         │
└──────────────────────────────────┘
```

**Petya의 MBR 공격 과정:**
```
1단계: 권한 상승
   - UAC 우회 시도
   - SeDebugPrivilege 획득
   
2단계: MBR 백업 및 수정
   - 원본 MBR 읽기 (DeviceIoControl)
   - 악성 부트로더로 교체
   - 파티션 테이블 손상
   
3단계: 강제 재부팅
   - Blue Screen 유발 (ntdll!NtRaiseHardError)
   - 또는 예약된 작업으로 재부팅
   
4단계: 부팅 시 실행
   - Petya 부트로더 실행
   - 가짜 CHKDSK 화면 표시
   - MFT (Master File Table) 암호화
   
5단계: 랜섬 노트 표시
   - Skull ASCII art 표시
   - Bitcoin 지불 요구
   - 복구 키 입력 화면
```

**암호화 대상:**
- **MBR**: 부팅 코드 교체
- **MFT**: NTFS의 마스터 파일 테이블 (파일 메타데이터)
- **실제 파일 데이터는 암호화하지 않음!** (Mischa 변종 제외)

### 피해 규모 및 영향

#### Petya (2016)
```
- 피해 국가: 주로 유럽 (독일, 프랑스 등)
- 피해 규모: 수천 대의 PC
- 산업: 기업, 개인 사용자
- 경제적 손실: 수백만 달러
```

#### NotPetya (2017)
```
- 피해 국가: 우크라이나, 러시아, 미국, 유럽 전역
- 피해 규모: 수만 대 이상
- 주요 피해 기업:
  * Maersk (해운): $300M+ 손실
  * Merck (제약): $870M+ 손실
  * FedEx: $400M+ 손실
  * 우크라이나 정부 기관 다수
- 총 경제적 손실: $10B+ (역대 최대)
```

**영향:**
- 글로벌 공급망 마비
- 병원, 공항 운영 중단
- 국가 인프라 타격
- 사이버 전쟁 도구로 인식

---

## 🔬 분석 접근 방법

### 정적 분석 단계별 가이드

#### 1. 기본 정보 수집

**파일 정보 확인:**
```bash
# Windows
certutil -hashfile petya.exe MD5
certutil -hashfile petya.exe SHA256

# Linux
md5sum petya.exe
sha256sum petya.exe
file petya.exe
```

**예상 출력:**
```
petya.exe: PE32 executable (GUI) Intel 80386, for MS Windows
MD5: [해시값]
SHA256: [해시값]
Size: ~72 KB (원본 Petya)
```

**VirusTotal 검색:**
```
1. virustotal.com 방문
2. 해시값으로 검색
3. ⚠️ 샘플 업로드는 금지 (공개됨)
4. 탐지 결과 및 행위 정보 확인
```

#### 2. 문자열 분석

**문자열 추출:**
```bash
# Windows
strings.exe petya.exe > strings.txt

# Linux
strings petya.exe > strings.txt
```

**주목할 문자열:**
```
- URL 및 IP 주소
- Bitcoin 지갑 주소
- 레지스트리 키
- 파일 경로
- 에러 메시지
- API 함수 이름
- 암호화 관련 문자열 (base64, hex 등)
```

**Petya 특징적 문자열:**
```
- "Misscha" (변종 이름)
- "\\.\PhysicalDrive0" (디스크 직접 접근)
- "MBR" 관련 문자열
- Bitcoin 관련 문자열
- Tor 주소 (.onion)
```

#### 3. PE 구조 분석

**PEStudio 사용:**
```
1. PEStudio 실행
2. petya.exe 드래그 앤 드롭
3. 분석 항목:
   - indicators: 의심스러운 특성 자동 표시
   - libraries: Import된 DLL 목록
   - imports: 사용된 API 함수
   - strings: 문자열 (형식별 분류)
   - resources: 내장된 리소스
   - sections: 섹션 정보 및 엔트로피
```

**주요 확인 사항:**
```
✓ 컴파일 시간 (Timestamp): 실제인지 조작되었는지
✓ Entry Point: 표준 위치인지 확인
✓ Sections: 비정상적인 섹션 있는지
✓ Imports:
  - CreateFileA/W (파일 접근)
  - WriteFile (파일 쓰기)
  - DeviceIoControl (디스크 직접 접근) ⚠️
  - CryptAcquireContext (암호화) ⚠️
  - NtRaiseHardError (강제 재부팅) ⚠️
✓ Entropy: 높은 엔트로피 = 암호화/패킹 가능성
```

#### 4. Ghidra 디스어셈블

**프로젝트 생성 및 분석:**
```
1. Ghidra 실행
2. File > New Project
   - Non-Shared Project
   - 프로젝트명: PetyaAnalysis
3. File > Import File > petya.exe
4. 더블클릭하여 CodeBrowser 열기
5. Analyze 클릭 (기본 옵션)
6. 분석 완료 대기 (1-2분)
```

**초기 분석 포인트:**
```
1. Entry Point 찾기:
   - Symbol Tree > Functions > entry
   - 또는 Search > Program Text > "entry"

2. 주요 함수 식별:
   - WinMain 또는 main 함수
   - 문자열 참조를 통한 함수 추적
   
3. Import 함수 확인:
   - Symbol Tree > Imports
   - DeviceIoControl, CryptEncrypt 등 찾기
   
4. 문자열 기반 분석:
   - Search > For Strings
   - "PhysicalDrive", "MBR" 등 검색
   - XREF 추적 (Ctrl+Shift+F)
```

### 동적 분석 단계별 가이드

⚠️ **경고**: MBR을 손상시킬 수 있으므로 **반드시 격리된 VM**에서 수행!

#### 1. 사전 준비

**스냅샷 생성:**
```
VirtualBox/VMware:
- 현재 상태를 "BeforePetyaAnalysis"로 저장
- 언제든 복원 가능하도록
```

**모니터링 도구 실행:**
```
1. Process Monitor 시작 (관리자 권한)
   - Filter: Process Name is petya.exe
   - Capture: Ctrl+E

2. Process Hacker 시작
   - 프로세스 트리 관찰 준비

3. Wireshark 시작 (필요 시)
   - 네트워크 활동 캡처
```

#### 2. 디버거에서 실행

**x64dbg 설정:**
```
1. x64dbg 실행 (x32 또는 x64 버전 선택)
2. File > Open > petya.exe
3. Options > Preferences > Events:
   - Entry Breakpoint 활성화
4. Debug > Run (F9)
5. Entry Point에서 중단됨
```

**초기 중단점 설정:**
```
# API 호출에 중단점
- DeviceIoControl (디스크 접근)
- WriteFile (파일 쓰기)
- CryptEncrypt (암호화)
- NtRaiseHardError (블루스크린)
- CreateFileA/W (파일 열기)
- RegSetValueEx (레지스트리 쓰기)

방법:
1. Ctrl+G: Go to Expression
2. DeviceIoControl 입력
3. F2: Breakpoint 설정
```

#### 3. 단계별 실행

**실행 흐름 추적:**
```
F7: Step Into - 함수 내부로 진입
F8: Step Over - 함수 건너뛰기
F9: Run - 다음 중단점까지 실행
Ctrl+F9: Run until return - 현재 함수 끝까지
```

**관찰 포인트:**
```
1. 레지스터 값:
   - EAX: 반환값 또는 연산 결과
   - ESP: 스택 포인터
   - EIP: 다음 실행 명령어
   
2. 스택:
   - 함수 파라미터
   - 지역 변수
   - 반환 주소
   
3. 메모리:
   - 문자열 버퍼
   - 데이터 구조
   - 암호화 키 (가능한 경우)
```

#### 4. 행위 분석

**Process Monitor 분석:**
```
분석 항목:
✓ File System:
  - CreateFile: 어떤 파일을 여는가?
  - WriteFile: 어떤 파일을 쓰는가?
  - DeleteFile: 파일 삭제 여부
  - \\.\PhysicalDrive0 접근 확인! ⚠️

✓ Registry:
  - RegSetValue: 어떤 키를 설정하는가?
  - Run 키 생성 확인 (지속성)
  
✓ Process/Thread:
  - 새 프로세스 생성 여부
  - 인젝션 시도 확인
  
✓ Network:
  - TCP/UDP 연결 시도
  - DNS 쿼리
```

**Process Hacker 분석:**
```
1. petya.exe 프로세스 선택
2. 속성 확인:
   - Handles: 열린 파일/레지스트리
   - Memory: 메모리 영역
   - Threads: 스레드 목록
3. 메모리 검색:
   - Memory > Strings
   - 암호화되지 않은 데이터 찾기
```

#### 5. 네트워크 분석

**Wireshark 캡처:**
```
필터:
- ip.addr == [샘플 연결 IP]
- http or https
- dns

확인 사항:
- C&C 서버 주소
- Bitcoin 결제 게이트웨이
- Tor 네트워크 사용
```

**NotPetya의 경우:**
```
- SMB 트래픽 (EternalBlue)
- 로컬 네트워크 스캔
- 자격 증명 전파 시도
```

---

## 🔍 주요 분석 포인트

### 초기 실행 흐름

**1. Entry Point 분석**

```c
// 의사 코드 (Ghidra 디컴파일 결과 예시)
int entry() {
    // 1. 명령줄 인자 확인
    if (argc > 1) {
        // 특정 파라미터로 다른 동작 수행
    }
    
    // 2. 관리자 권한 확인
    if (!IsAdmin()) {
        // UAC 우회 시도 또는 권한 상승
        ElevatePrivileges();
    }
    
    // 3. 메인 악성 루틴 시작
    return MaliciousMain();
}
```

**2. 안티 분석 체크**

```c
// 디버거 탐지
if (IsDebuggerPresent()) {
    ExitProcess(0);
}

// VM 탐지 (일부 변종)
if (IsVirtualMachine()) {
    ExitProcess(0);
}

// 샌드박스 탐지
if (CheckSandbox()) {
    Sleep(600000);  // 10분 대기
}
```

### 파일 드롭 및 권한 상승

**드롭된 파일:**
```
위치: %TEMP% 또는 현재 디렉토리
- [random].tmp: 임시 실행 파일
- [random].dat: 암호화된 데이터
```

**권한 상승 기법:**
```
1. UAC 우회:
   - COM Elevation Moniker (Mischa)
   - eventvwr.exe hijacking
   
2. SeDebugPrivilege 획득:
   - OpenProcessToken
   - AdjustTokenPrivileges
   
3. 드라이버 설치 (일부 변종):
   - 커널 레벨 접근
```

### MBR 덮어쓰기 과정

**코드 흐름:**

```c
// 1. 물리 디스크 열기
HANDLE hDisk = CreateFileA(
    "\\\\.\\PhysicalDrive0",
    GENERIC_READ | GENERIC_WRITE,
    FILE_SHARE_READ | FILE_SHARE_WRITE,
    NULL,
    OPEN_EXISTING,
    0,
    NULL
);

if (hDisk == INVALID_HANDLE_VALUE) {
    // 권한 부족 - 권한 상승 시도
    return;
}

// 2. 원본 MBR 백업 (복구용 - Petya만 해당)
BYTE originalMBR[512];
DWORD bytesRead;
ReadFile(hDisk, originalMBR, 512, &bytesRead, NULL);

// 3. 암호화된 원본 MBR 저장
EncryptAndStore(originalMBR, 512);

// 4. 악성 부트로더 작성
BYTE maliciousBootloader[512];
PrepareBootloader(maliciousBootloader);
SetFilePointer(hDisk, 0, NULL, FILE_BEGIN);
WriteFile(hDisk, maliciousBootloader, 512, NULL, NULL);

// 5. 추가 악성 코드를 다른 섹터에 작성
for (int sector = 1; sector < 63; sector++) {
    WriteFile(hDisk, maliciousCode[sector], 512, NULL, NULL);
}

// 6. 파티션 테이블 손상 (NotPetya)
// OverwritePartitionTable();

CloseHandle(hDisk);
```

**섹터 레이아웃:**
```
Sector 0:    악성 부트로더
Sector 1-33: 악성 코드 (암호화 루틴 등)
Sector 34+:  암호화된 원본 MBR 및 설정 데이터
```

### 암호화 알고리즘 식별

**Salsa20 암호화**

Petya는 Salsa20 스트림 암호를 사용합니다.

**식별 방법:**
```
1. Import 테이블 확인:
   - CryptAcquireContext
   - CryptGenRandom (난수 생성)
   
2. 코드 패턴 인식:
   - Salsa20 특유의 상수값:
     * "expand 32-byte k"
   - Quarter-round 함수
   - 20번의 라운드 (Salsa20의 '20')
```

**Ghidra에서 식별:**
```c
// Salsa20 특징적인 코드
void salsa20_core(uint32_t *output, uint32_t *input) {
    // 상수값 "expa", "nd 3", "2-by", "te k"
    uint32_t x[16];
    
    // 20 라운드
    for (int i = 0; i < 10; i++) {
        quarterround(x, 0, 4, 8, 12);
        quarterround(x, 5, 9, 13, 1);
        quarterround(x, 10, 14, 2, 6);
        quarterround(x, 15, 3, 7, 11);
        // ...
    }
}
```

**암호화 키 관리:**
```
Petya (원본):
- 키 생성: CryptGenRandom으로 난수 생성
- 키 저장: 디스크의 특정 섹터에 저장
- 복구: 올바른 비밀번호 입력 시 키 복호화

NotPetya:
- 키 생성: 난수 생성
- 키 파괴: 생성 직후 덮어쓰기
- 복구: 불가능 (의도적 파괴)
```

### C&C 통신 분석

**Petya (원본):**
```
- Tor 네트워크 사용 (.onion 주소)
- Bitcoin 결제 확인 API
- 복구 키 검증 서버

통신 흐름:
1. 감염자 ID 생성
2. Tor를 통해 C&C 연결
3. Bitcoin 지불 확인 요청
4. 복구 키 수신 (지불 시)
```

**분석 방법:**
```
1. 문자열에서 .onion 주소 찾기
2. Bitcoin 지갑 주소 추출
3. Wireshark로 네트워크 트래픽 캡처
   - Tor 연결 시도 (포트 9050, 9051)
   - HTTPS 트래픽
```

**NotPetya:**
```
- C&C 통신 없음 (Wiper로 설계)
- 가짜 Bitcoin 주소 표시
- 이메일 주소도 가짜 (폐쇄됨)
```

### 지속성 메커니즘

**Petya의 지속성:**
```
⚠️ Petya는 전통적인 지속성을 사용하지 않음
   - MBR 감염이 곧 지속성
   - 재부팅 시 항상 실행됨
   - 제거하려면 MBR 복구 필요
```

**Mischa 변종의 지속성:**
```
레지스트리:
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
- 값: [랜덤명]
- 데이터: [실행파일 경로]

예약된 작업:
- schtasks를 통한 작업 생성
- 부팅 시 또는 로그인 시 실행
```

### 랜섬노트 표시 방법

**부팅 시 화면:**
```
┌─────────────────────────────────────────────┐
│         ☠ PETYA RANSOMWARE ☠                │
│                                             │
│   Your MBR has been encrypted!              │
│   To decrypt, pay 0.99 Bitcoin to:         │
│   [Bitcoin Address]                         │
│                                             │
│   Infection ID: [고유 ID]                   │
│                                             │
│   After payment, enter your key:           │
│   [_________________________________]       │
│                                             │
│   ⚠️ DO NOT turn off your computer!        │
└─────────────────────────────────────────────┘
```

**부트로더 코드 분석:**
```asm
; 16비트 리얼 모드 코드
start:
    ; 비디오 모드 설정
    mov ax, 0x0003      ; 80x25 텍스트 모드
    int 0x10
    
    ; 스컬 ASCII art 출력
    mov si, skull_art
    call print_string
    
    ; 랜섬 메시지 출력
    mov si, ransom_msg
    call print_string
    
    ; 키보드 입력 대기
    call get_input
    
    ; 입력된 키 검증
    call verify_key
    jz key_valid
    
    ; 무효한 키
    jmp start
    
key_valid:
    ; MBR 복호화 시작
    call decrypt_mbr
    ; 원본 부트로더로 점프
    jmp 0x7C00
```

---

## 🔓 복구 가능성 분석

### Petya (원본) 복구

**복구 가능:**
✅ 예 - 키가 있다면 가능

**복구 방법:**
```
1. 부팅 시 Petya 화면에서 키 입력
2. Petya가 MBR 복호화
3. Windows 정상 부팅

키가 없는 경우:
- Genetic algorithm을 사용한 브루트포스
- leostone의 Petya 복호화 도구 사용
- 약한 키 생성 알고리즘 악용
```

**복호화 도구:**
```
- leostone/petya_recovery
  https://github.com/leostone/petya_recovery
  
사용법:
1. 감염된 디스크의 섹터 덤프
2. 도구로 키 생성 시도
3. 키 입력하여 복구
```

### NotPetya (ExPetr) 복구

**복구 불가능:**
❌ 아니오 - 의도적으로 설계됨

**이유:**
```
1. 암호화 키가 생성 직후 파괴됨
2. 살라미 ID가 무작위 (복구 불가)
3. 결제해도 키가 없음
4. Wiper로 설계됨 (랜섬웨어가 아님)
```

**유일한 복구 방법:**
```
- 백업에서 복원
- 디스크 포렌식 (부분 복구 가능성)
- 전문 복구 업체 (비용 대비 효과 낮음)
```

### MBR 수동 복구 (키 없이)

**Windows 복구 환경 사용:**
```
1. Windows 설치 미디어로 부팅
2. "컴퓨터 복구" 선택
3. 명령 프롬프트 실행
4. MBR 복구 명령어:

   bootrec /fixmbr     # MBR 재작성
   bootrec /fixboot    # 부트 섹터 복구
   bootrec /rebuildbcd # BCD 재구성
```

**Linux에서 복구:**
```bash
# dd를 사용한 MBR 백업 복원 (사전 백업이 있는 경우)
sudo dd if=mbr_backup.img of=/dev/sda bs=512 count=1

# testdisk를 사용한 파티션 테이블 복구
sudo testdisk /dev/sda
```

**⚠️ 주의**: MBR 복구 후에도 MFT가 손상된 경우 파일 복구 어려움

---

## 🛡️ 탐지 및 방어 전략

### 탐지 방법

#### 1. 행위 기반 탐지

**의심스러운 행위:**
```
✓ PhysicalDrive0 직접 접근
✓ MBR 영역 쓰기 시도
✓ NtRaiseHardError API 호출
✓ 권한 상승 시도
✓ 대량의 파일 암호화
```

**SIEM 규칙 예시:**
```
rule DetectMBRWrite {
    condition:
        process.file.path != "C:\\Windows\\System32\\*" AND
        (
            file.device.bus_type == "disk" AND
            file.device.access == "\\.\PhysicalDrive0" AND
            file.operation == "write"
        )
}
```

#### 2. 시그니처 기반 탐지

**YARA 규칙:**
```yara
rule Petya_Ransomware {
    meta:
        description = "Detects Petya Ransomware"
        author = "Security Researcher"
        date = "2024-01-01"
        
    strings:
        $mbr = "\\\\.\\PhysicalDrive0" ascii
        $salsa = "expand 32-byte k" ascii
        $bitcoin = /[13][a-km-zA-HJ-NP-Z1-9]{25,34}/ ascii
        $skull = {1F 1F 0F 00 00 0F 1F 1F}  // Skull pattern
        
        $api1 = "NtRaiseHardError" ascii
        $api2 = "DeviceIoControl" ascii
        $api3 = "CryptGenRandom" ascii
        
    condition:
        uint16(0) == 0x5A4D and  // PE file
        filesize < 200KB and
        $mbr and
        2 of ($api*) and
        ($salsa or $bitcoin)
}
```

#### 3. 네트워크 탐지

**IDS/IPS 규칙 (Snort):**
```
# Petya Bitcoin 결제 트래픽
alert tcp any any -> any $HTTP_PORTS (
    msg:"Possible Petya Ransomware Payment Request";
    flow:established,to_server;
    content:"POST";
    content:"bitcoin";
    classtype:trojan-activity;
    sid:1000001;
)

# EternalBlue 공격 (NotPetya)
alert tcp any any -> any 445 (
    msg:"Possible EternalBlue SMB Exploit";
    flow:established,to_server;
    content:"|ff|SMB|75|";
    content:"|00 00 00 00|";
    classtype:attempted-admin;
    sid:1000002;
)
```

#### 4. 엔드포인트 탐지

**Sysmon 설정:**
```xml
<Sysmon schemaversion="4.50">
  <EventFiltering>
    <!-- Device 파일 접근 탐지 -->
    <FileCreate onmatch="include">
      <TargetFilename condition="contains">\\.\ </TargetFilename>
    </FileCreate>
    
    <!-- 원시 디스크 접근 -->
    <RawAccessRead onmatch="include">
      <Device condition="is">PhysicalDrive0</Device>
    </RawAccessRead>
    
    <!-- 권한 상승 -->
    <ProcessAccess onmatch="include">
      <GrantedAccess>0x1F0FFF</GrantedAccess>
    </ProcessAccess>
  </EventFiltering>
</Sysmon>
```

### 방어 전략

#### 1. 사전 예방

**시스템 강화:**
```
✓ Windows 업데이트 적용 (특히 MS17-010)
✓ SMBv1 비활성화
✓ 불필요한 관리 공유 비활성화
✓ UAC 최고 수준 설정
✓ PowerShell 실행 정책 강화
```

**명령어:**
```powershell
# SMBv1 비활성화
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol

# 관리 공유 비활성화 (레지스트리)
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name "AutoShareWks" -Value 0 -PropertyType DWORD

# PowerShell Constrained Language Mode
$ExecutionContext.SessionState.LanguageMode = "ConstrainedLanguage"
```

#### 2. 네트워크 세그먼트화

**네트워크 격리:**
```
┌─────────────────┐
│  DMZ (Public)   │ ← 방화벽 ← 인터넷
└────────┬────────┘
         │
    ┌────▼────┐
    │ 방화벽  │
    └────┬────┘
         │
┌────────▼─────────┐
│   내부 네트워크   │
│  ┌─────┐ ┌─────┐ │
│  │ IT  │ │ HR  │ │ ← VLAN 분리
│  └─────┘ └─────┘ │
└──────────────────┘
```

**방화벽 규칙:**
```
# SMB 포트 차단 (외부)
iptables -A INPUT -p tcp --dport 445 -s ! 192.168.0.0/16 -j DROP
iptables -A INPUT -p tcp --dport 139 -s ! 192.168.0.0/16 -j DROP

# WMI 포트 제한
iptables -A INPUT -p tcp --dport 135 -s ! 192.168.0.0/16 -j DROP
```

#### 3. 백업 전략

**3-2-1 규칙:**
```
3: 최소 3개의 복사본
2: 2개의 다른 미디어
1: 1개는 오프사이트(외부)

구현:
- 온사이트: 로컬 NAS
- 오프사이트: 클라우드 백업
- 오프라인: 테이프 또는 외장 HDD (주기적 연결)
```

**백업 검증:**
```powershell
# 정기적 백업 테스트
Test-Path \\backup-server\backups\latest.zip
Get-ChildItem \\backup-server\backups | 
    Where-Object { $_.LastWriteTime -gt (Get-Date).AddDays(-1) }
```

#### 4. 접근 제어

**최소 권한 원칙:**
```
- 일반 사용자: 관리자 권한 없음
- 관리자 계정: 별도 계정, MFA 필수
- 서비스 계정: 최소 권한만 부여
```

**그룹 정책 설정:**
```
Computer Configuration > Windows Settings > Security Settings >
Local Policies > User Rights Assignment:

- "Debug programs": Administrators만
- "Load and unload device drivers": Administrators만
- "Shut down the system": 제한
```

#### 5. 모니터링 및 로깅

**필수 로그:**
```
✓ Windows Event Logs
  - Security: 로그인, 권한 변경
  - System: 디스크 접근, 드라이버 로드
  
✓ Sysmon
  - 프로세스 생성
  - 네트워크 연결
  - 파일 생성
  - 레지스트리 변경
  
✓ 방화벽/IDS 로그
  - 네트워크 트래픽
  - 차단된 연결
```

**SIEM 통합:**
```
모든 로그를 중앙 SIEM으로 전송:
- Splunk
- Elastic Stack (ELK)
- Microsoft Sentinel
- IBM QRadar
```

#### 6. 사용자 교육

**피싱 방어:**
```
✓ 이상한 이메일 주의
✓ 첨부 파일 실행 전 확인
✓ Dropbox/WeTransfer 링크 주의
✓ IT 부서에 즉시 보고
```

**정기 훈련:**
```
- 모의 피싱 테스트
- 보안 인식 교육
- 랜섬웨어 대응 훈련
```

#### 7. 인시던트 대응 계획

**랜섬웨어 대응 플레이북:**
```
1. 탐지
   - 경보 확인
   - 피해 범위 파악

2. 격리
   - 감염된 시스템 네트워크 차단
   - 관련 계정 비활성화

3. 조사
   - 악성코드 샘플 수집
   - IOC 추출
   - 침투 경로 분석

4. 제거
   - 감염된 시스템 재이미징
   - MBR 복구 (필요 시)

5. 복구
   - 백업에서 데이터 복원
   - 서비스 재개

6. 사후 분석
   - 근본 원인 분석
   - 재발 방지 조치
```

### IOC (Indicators of Compromise)

**파일 IOC:**
```
MD5 해시:
- Petya (Green): [실제 샘플 해시]
- NotPetya: 027cc450ef5f8c5f653329641ec1fed9

파일명:
- perfc.dat
- [random].exe
- %TEMP%\[random].tmp

경로:
- C:\Windows\perfc.dat
- %TEMP%\
```

**네트워크 IOC:**
```
IP 주소:
- [C&C 서버 IP]

도메인:
- [.onion 주소]

Bitcoin 지갑:
- 1Mz7153HMuxXTuR2R1t78mGSdzaAtNbBWX (예시)
```

**레지스트리 IOC:**
```
HKLM\SYSTEM\CurrentControlSet\Services\[랜덤명]
HKCU\Software\Microsoft\Windows\CurrentVersion\Run\[랜덤명]
```

**Yara 기반 IOC:**
```
- MBR 수정 패턴
- Salsa20 상수
- 특정 문자열
```

---

## 📚 추가 자료

### 관련 문서
- [리버싱 기본 개념](reversing-basics.md)
- [실습 환경 설정](setup-environment.md)
- [단계별 분석 체크리스트](analysis-steps.md)

### 외부 리소스
- Kaspersky Lab: Petya 분석 보고서
- Cisco Talos: NotPetya 상세 분석
- SANS: Petya 랜섬웨어 디코더

---

**⚠️ 면책 조항**: 이 문서는 교육 목적으로만 사용되어야 합니다. 실제 악성코드 분석은 항상 격리된 환경에서 수행하세요.
