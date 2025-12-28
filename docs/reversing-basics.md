# 리버싱 기본 개념

## 📚 목차

- [리버스 엔지니어링이란?](#-리버스-엔지니어링이란)
- [정적 분석 vs 동적 분석](#-정적-분석-vs-동적-분석)
- [필수 도구 소개](#-필수-도구-소개)
- [x86/x64 어셈블리 기초](#-x86x64-어셈블리-기초)
- [PE 파일 구조](#-pe-파일-구조)
- [디컴파일러 사용법](#-디컴파일러-사용법)
- [안티 디버깅 기법](#-안티-디버깅-기법)

---

## 🔍 리버스 엔지니어링이란?

### 정의

**리버스 엔지니어링(Reverse Engineering)**은 완성된 제품이나 프로그램을 분해하고 분석하여 그 구조, 기능, 동작 원리를 이해하는 과정입니다.

소프트웨어 리버싱은 주로 **실행 파일(바이너리)**을 대상으로 하며, 소스 코드 없이 프로그램의 동작을 파악합니다.

### 목적

리버싱은 다양한 목적으로 수행됩니다:

1. **악성코드 분석** 🦠
   - 랜섬웨어, 트로이목마, 바이러스 등의 동작 분석
   - 공격 기법 및 취약점 파악
   - 탐지 및 제거 방법 개발

2. **취약점 연구** 🔓
   - 소프트웨어의 보안 취약점 발견
   - 패치되지 않은 버그 찾기
   - 익스플로잇 개발 및 방어

3. **호환성 분석** 🔄
   - 레거시 시스템 이해
   - API 및 프로토콜 분석
   - 상호운용성 확보

4. **지적 재산권 보호** ⚖️
   - 불법 복제 탐지
   - 코드 도용 확인
   - 라이선스 위반 조사

5. **학습 및 연구** 📖
   - 프로그래밍 기법 학습
   - 알고리즘 이해
   - 최적화 방법 연구

### 합법성

⚠️ **중요**: 리버싱의 합법성은 **목적과 대상**에 따라 달라집니다.

**합법적인 경우:**
- 본인이 소유한 소프트웨어 분석
- 상호운용성을 위한 분석 (EU, 미국 일부 주)
- 보안 연구 목적 (책임있는 공개 전제)
- 교육 목적의 학습

**불법일 수 있는 경우:**
- 라이선스 약관 위반
- DRM 우회 목적
- 지적 재산권 침해
- 악의적 목적의 분석

---

## ⚖️ 정적 분석 vs 동적 분석

악성코드나 프로그램을 분석할 때는 크게 두 가지 접근 방법을 사용합니다.

### 정적 분석 (Static Analysis)

**프로그램을 실행하지 않고** 코드 자체를 분석하는 방법입니다.

#### 특징

✅ **장점:**
- 안전함 (악성 코드가 실행되지 않음)
- 전체 코드 흐름 파악 가능
- 숨겨진 기능 발견 가능
- 실행 조건이 없어도 분석 가능

❌ **단점:**
- 난독화/패킹된 코드 분석 어려움
- 시간이 많이 소요됨
- 전문 지식 필요
- 동적 생성 코드 파악 어려움

#### 주요 기법

1. **파일 정보 수집**
   ```bash
   # 파일 타입 확인
   file malware.exe
   
   # 해시 계산
   md5sum malware.exe
   sha256sum malware.exe
   ```

2. **문자열 추출**
   ```bash
   # Windows
   strings.exe malware.exe
   
   # Linux
   strings malware.exe
   ```

3. **PE 구조 분석**
   - PE 헤더 검사
   - Import/Export 테이블 확인
   - 섹션 분석

4. **디스어셈블**
   - IDA Pro, Ghidra 사용
   - 어셈블리 코드 분석
   - 함수 흐름 추적

#### 정적 분석 도구

| 도구 | 용도 | 라이선스 |
|-----|------|---------|
| **Ghidra** | 디스어셈블러/디컴파일러 | 무료 (NSA) |
| **IDA Free** | 디스어셈블러 | 무료 (제한적) |
| **Binary Ninja** | 디스어셈블러 | 상용/개인용 |
| **PEStudio** | PE 분석 | 무료 |
| **PE-bear** | PE 분석 | 무료 |
| **Detect It Easy** | 패커/컴파일러 탐지 | 무료 |

### 동적 분석 (Dynamic Analysis)

**프로그램을 실제로 실행**하면서 동작을 관찰하는 방법입니다.

#### 특징

✅ **장점:**
- 실제 동작 확인 가능
- 난독화 우회 가능
- 빠른 분석 가능
- 숨겨진 기능 발견

❌ **단점:**
- 위험함 (격리 환경 필수)
- 모든 코드 경로 실행 어려움
- 안티 디버깅 기법에 방해받음
- 실행 조건 필요 (네트워크, 파일 등)

#### 주요 기법

1. **디버깅**
   - 중단점(Breakpoint) 설정
   - 단계별 실행(Step Into/Over)
   - 레지스터/메모리 검사

2. **시스템 모니터링**
   ```
   - 파일 시스템 변경 추적
   - 레지스트리 변경 모니터링
   - 네트워크 연결 감시
   - 프로세스 생성 감시
   ```

3. **API 후킹**
   - 함수 호출 가로채기
   - 파라미터 로깅
   - 반환값 수정

4. **메모리 분석**
   - 메모리 덤프 생성
   - 언패킹된 코드 추출
   - 암호화 키 검색

#### 동적 분석 도구

| 도구 | 용도 | 플랫폼 |
|-----|------|--------|
| **x64dbg** | 디버거 | Windows |
| **OllyDbg** | 디버거 | Windows |
| **WinDbg** | 커널 디버거 | Windows |
| **GDB** | 디버거 | Linux |
| **Process Monitor** | 시스템 모니터링 | Windows |
| **Process Hacker** | 프로세스 분석 | Windows |
| **Wireshark** | 네트워크 분석 | 멀티플랫폼 |

### 통합 분석 접근

실제로는 **정적 분석과 동적 분석을 병행**하는 것이 가장 효과적입니다.

```
┌─────────────────────────────────────────┐
│          종합 분석 워크플로우              │
└─────────────────────────────────────────┘

1. [정적] 파일 정보 수집 및 문자열 추출
2. [정적] PE 구조 분석 및 Import 확인
3. [동적] 첫 실행 및 행위 관찰
4. [정적] 관심 함수 디스어셈블
5. [동적] 중단점 설정 후 상세 분석
6. [정적] 알고리즘 이해 및 문서화
7. [동적] 추가 검증 및 IOC 수집
```

---

## 🛠️ 필수 도구 소개

### 디스어셈블러 (Disassembler)

기계어를 어셈블리 언어로 변환하는 도구입니다.

#### 1. Ghidra

![Ghidra Logo](https://ghidra-sre.org/)

**특징:**
- NSA가 개발하고 오픈소스로 공개
- 강력한 디컴파일러 내장
- 다양한 아키텍처 지원
- 무료

**장점:**
- 고품질 디컴파일 결과
- 스크립팅 지원 (Python, Java)
- 협업 기능
- 크로스 플랫폼

**단점:**
- 느린 실행 속도
- 높은 메모리 사용량
- 초기 학습 곡선

**사용 예:**
```bash
# Ghidra 프로젝트 시작
ghidraRun

# 명령줄 분석
analyzeHeadless /project ProjectName -import malware.exe -postScript analyze.py
```

#### 2. IDA Pro / IDA Free

**특징:**
- 업계 표준 디스어셈블러
- 풍부한 플러그인 생태계
- 강력한 분석 기능

**IDA Free 제약사항:**
- 64비트 PE 파일 분석 불가
- 일부 고급 기능 제한
- 상업적 사용 불가

**장점:**
- 직관적인 UI
- 빠른 성능
- 풍부한 플러그인

**단점:**
- 고가 (Pro 버전)
- 디컴파일러 별도 구매 (Hex-Rays)

#### 3. Binary Ninja

**특징:**
- 현대적인 UI/UX
- 중간 언어(IL) 제공
- 활발한 개발

**장점:**
- 우수한 가격 대비 성능
- 강력한 API
- 빠른 업데이트

### 디버거 (Debugger)

프로그램을 실행하며 단계별로 제어하고 관찰하는 도구입니다.

#### 1. x64dbg

**특징:**
- Windows용 오픈소스 디버거
- x86 및 x64 지원
- OllyDbg의 현대적 대체

**주요 기능:**
- 그래픽 UI
- 플러그인 지원
- 스크립팅
- 메모리 맵 시각화

**사용법:**
```
1. x64dbg 실행
2. File > Open > 분석할 파일 선택
3. F7: Step Into
4. F8: Step Over
5. F9: Run
6. F2: Breakpoint 설정
```

#### 2. OllyDbg

**특징:**
- 클래식 Windows 디버거
- 32비트 전용
- 간단한 사용법

**제약사항:**
- 더 이상 개발되지 않음
- 64비트 미지원
- Windows 10에서 일부 문제

#### 3. WinDbg

**특징:**
- Microsoft 공식 디버거
- 커널 모드 디버깅 지원
- 강력하지만 복잡함

**사용 사례:**
- 드라이버 분석
- 블루스크린 분석
- 시스템 레벨 디버깅

### PE 분석 도구

#### 1. PEStudio

**기능:**
- 자동 위협 평가
- Import/Export 분석
- 문자열 추출
- 시그니처 검증

#### 2. PE-bear

**기능:**
- PE 구조 시각화
- 섹션 편집
- Import 재구성

#### 3. CFF Explorer

**기능:**
- 종합 PE 편집기
- Import/Export 편집
- 리소스 편집

### 시스템 모니터링 도구

#### 1. Process Monitor (ProcMon)

**기능:**
- 실시간 파일 시스템 모니터링
- 레지스트리 변경 추적
- 네트워크 활동 기록
- 프로세스/스레드 활동

**사용법:**
```
1. Procmon 실행 (관리자 권한)
2. Ctrl+E: 캡처 시작/중지
3. 필터 설정: Process Name is malware.exe
4. 이벤트 확인 및 분석
```

#### 2. Process Hacker

**기능:**
- 프로세스 트리 보기
- 메모리 검색 및 덤프
- 네트워크 연결 확인
- 핸들/DLL 조회

### 네트워크 분석 도구

#### 1. Wireshark

**기능:**
- 패킷 캡처 및 분석
- 프로토콜 디코딩
- 필터링 및 검색

**사용 예:**
```
# 특정 IP로의 트래픽 필터
ip.dst == 192.168.1.100

# HTTP 트래픽만 보기
http

# TCP 포트 443 트래픽
tcp.port == 443
```

#### 2. Fiddler

**기능:**
- HTTP/HTTPS 프록시
- 웹 트래픽 디버깅
- 요청/응답 수정

---

## 💻 x86/x64 어셈블리 기초

### 레지스터 (Registers)

레지스터는 CPU 내부의 초고속 저장 공간입니다.

#### 범용 레지스터 (x86/x64)

| 64비트 | 32비트 | 16비트 | 8비트 | 용도 |
|--------|--------|--------|-------|------|
| **RAX** | EAX | AX | AL/AH | 누산기 (Accumulator) |
| **RBX** | EBX | BX | BL/BH | 베이스 (Base) |
| **RCX** | ECX | CX | CL/CH | 카운터 (Counter) |
| **RDX** | EDX | DX | DL/DH | 데이터 (Data) |
| **RSI** | ESI | SI | SIL | 소스 인덱스 (Source Index) |
| **RDI** | EDI | DI | DIL | 목적지 인덱스 (Destination Index) |
| **RBP** | EBP | BP | BPL | 베이스 포인터 (Base Pointer) |
| **RSP** | ESP | SP | SPL | 스택 포인터 (Stack Pointer) |

#### 특수 레지스터

- **RIP/EIP**: 명령어 포인터 (Instruction Pointer) - 다음 실행할 명령어 주소
- **RFLAGS/EFLAGS**: 플래그 레지스터 - CPU 상태 정보
  - ZF (Zero Flag): 결과가 0일 때
  - CF (Carry Flag): 캐리 발생
  - SF (Sign Flag): 결과가 음수
  - OF (Overflow Flag): 오버플로우

### 기본 명령어

#### 데이터 이동

```asm
; MOV - 데이터 복사
MOV EAX, 0x10       ; EAX = 0x10
MOV EBX, EAX        ; EBX = EAX
MOV [0x401000], EAX ; 메모리[0x401000] = EAX

; LEA - 주소 로드 (Load Effective Address)
LEA EAX, [EBX+4]    ; EAX = EBX + 4 (주소 계산)

; XCHG - 교환
XCHG EAX, EBX       ; EAX <-> EBX
```

#### 스택 조작

```asm
; PUSH - 스택에 푸시
PUSH EAX            ; ESP -= 4, [ESP] = EAX

; POP - 스택에서 팝
POP EBX             ; EBX = [ESP], ESP += 4

; 함수 프롤로그
PUSH EBP            ; 이전 베이스 포인터 저장
MOV EBP, ESP        ; 현재 스택을 베이스로 설정

; 함수 에필로그
MOV ESP, EBP        ; 스택 복원
POP EBP             ; 이전 베이스 포인터 복원
RET                 ; 반환
```

#### 산술 연산

```asm
; ADD - 덧셈
ADD EAX, 5          ; EAX = EAX + 5

; SUB - 뺄셈
SUB EBX, 2          ; EBX = EBX - 2

; MUL - 곱셈 (unsigned)
MUL EBX             ; EDX:EAX = EAX * EBX

; DIV - 나눗셈 (unsigned)
DIV ECX             ; EAX = EDX:EAX / ECX, EDX = 나머지

; INC - 증가
INC EAX             ; EAX = EAX + 1

; DEC - 감소
DEC ECX             ; ECX = ECX - 1
```

#### 논리 연산

```asm
; AND - 논리곱
AND EAX, 0xFF       ; EAX = EAX & 0xFF

; OR - 논리합
OR EBX, 0x80        ; EBX = EBX | 0x80

; XOR - 배타적 논리합
XOR EAX, EAX        ; EAX = 0 (자주 사용하는 최적화)

; NOT - 논리 부정
NOT EBX             ; EBX = ~EBX

; TEST - AND 연산 후 플래그만 설정 (값 변경 안 함)
TEST EAX, EAX       ; ZF 설정, EAX가 0이면 ZF=1
```

#### 비교 및 분기

```asm
; CMP - 비교 (뺄셈 후 플래그만 설정)
CMP EAX, EBX        ; EAX - EBX 계산, 플래그 설정

; JMP - 무조건 점프
JMP 0x401000        ; 0x401000으로 점프

; 조건부 점프
JE  label           ; Jump if Equal (ZF=1)
JNE label           ; Jump if Not Equal (ZF=0)
JZ  label           ; Jump if Zero (ZF=1)
JNZ label           ; Jump if Not Zero (ZF=0)
JG  label           ; Jump if Greater (signed)
JL  label           ; Jump if Less (signed)
JA  label           ; Jump if Above (unsigned)
JB  label           ; Jump if Below (unsigned)
```

#### 함수 호출

```asm
; CALL - 함수 호출
CALL 0x401000       ; PUSH EIP+5, JMP 0x401000

; RET - 함수 반환
RET                 ; POP EIP
RET 0x10            ; POP EIP, ESP += 0x10 (파라미터 정리)
```

### 호출 규약 (Calling Conventions)

#### cdecl (C Declaration)
- 파라미터: 오른쪽부터 왼쪽으로 스택에 푸시
- 정리: 호출자가 스택 정리
- 반환값: EAX

```asm
; int add(int a, int b)
PUSH 2              ; b
PUSH 1              ; a
CALL add
ADD ESP, 8          ; 호출자가 정리
```

#### stdcall (Standard Call)
- 파라미터: cdecl과 동일
- 정리: 피호출자가 스택 정리 (RET n)
- 반환값: EAX
- Windows API에서 사용

```asm
PUSH 2
PUSH 1
CALL MessageBoxA    ; 함수 내부에서 RET 8
```

#### fastcall
- 처음 2개 파라미터: ECX, EDX
- 나머지: 스택
- 빠른 호출

#### x64 호출 규약 (Microsoft x64)
- 처음 4개 파라미터: RCX, RDX, R8, R9
- 나머지: 스택
- 반환값: RAX
- Shadow space: 32바이트 예약

### 스택 프레임 이해

```asm
; 함수 진입
PUSH EBP            ; 이전 베이스 포인터 저장
MOV EBP, ESP        ; 현재 스택을 베이스로 설정
SUB ESP, 0x20       ; 지역 변수 공간 할당

; 스택 레이아웃:
; [EBP+12] - 2번째 파라미터
; [EBP+8]  - 1번째 파라미터
; [EBP+4]  - 반환 주소
; [EBP]    - 이전 EBP
; [EBP-4]  - 1번째 지역 변수
; [EBP-8]  - 2번째 지역 변수
; ...

; 함수 종료
MOV ESP, EBP        ; 스택 복원
POP EBP             ; 이전 베이스 포인터 복원
RET                 ; 반환
```

---

## 📦 PE 파일 구조

**PE (Portable Executable)**는 Windows 실행 파일 형식입니다.

### PE 파일 구조 개요

```
┌─────────────────────────┐
│     DOS Header          │ ← MZ 시그니처
├─────────────────────────┤
│     DOS Stub            │ ← "This program..."
├─────────────────────────┤
│     PE Header           │ ← PE 시그니처 (PE\0\0)
│  - Signature            │
│  - File Header          │
│  - Optional Header      │
├─────────────────────────┤
│   Section Headers       │ ← .text, .data, .rdata...
├─────────────────────────┤
│     .text Section       │ ← 실행 코드
├─────────────────────────┤
│     .data Section       │ ← 초기화된 데이터
├─────────────────────────┤
│     .rdata Section      │ ← 읽기 전용 데이터
├─────────────────────────┤
│     .rsrc Section       │ ← 리소스 (아이콘, 문자열 등)
└─────────────────────────┘
```

### DOS Header

```c
typedef struct _IMAGE_DOS_HEADER {
    WORD e_magic;      // MZ 시그니처 (0x5A4D)
    // ... (생략)
    LONG e_lfanew;     // PE 헤더 오프셋
} IMAGE_DOS_HEADER;
```

- **e_magic**: 항상 "MZ" (0x5A4D) - Mark Zbikowski의 이니셜
- **e_lfanew**: PE 헤더의 파일 오프셋

### PE Header

#### NT Headers

```c
typedef struct _IMAGE_NT_HEADERS {
    DWORD Signature;                    // PE\0\0 (0x50450000)
    IMAGE_FILE_HEADER FileHeader;
    IMAGE_OPTIONAL_HEADER OptionalHeader;
} IMAGE_NT_HEADERS;
```

#### File Header

```c
typedef struct _IMAGE_FILE_HEADER {
    WORD  Machine;              // 0x014C (x86), 0x8664 (x64)
    WORD  NumberOfSections;     // 섹션 개수
    DWORD TimeDateStamp;        // 컴파일 시간
    DWORD PointerToSymbolTable;
    DWORD NumberOfSymbols;
    WORD  SizeOfOptionalHeader;
    WORD  Characteristics;      // 파일 속성
} IMAGE_FILE_HEADER;
```

**Characteristics 플래그:**
- `0x0002`: IMAGE_FILE_EXECUTABLE_IMAGE (실행 가능)
- `0x0020`: IMAGE_FILE_LARGE_ADDRESS_AWARE
- `0x0100`: IMAGE_FILE_32BIT_MACHINE
- `0x2000`: IMAGE_FILE_DLL (DLL 파일)

#### Optional Header

```c
typedef struct _IMAGE_OPTIONAL_HEADER {
    WORD  Magic;                    // 0x010B (x86), 0x020B (x64)
    BYTE  MajorLinkerVersion;
    BYTE  MinorLinkerVersion;
    DWORD SizeOfCode;
    DWORD SizeOfInitializedData;
    DWORD SizeOfUninitializedData;
    DWORD AddressOfEntryPoint;      // 진입점 (EP) - 중요!
    DWORD BaseOfCode;
    DWORD ImageBase;                // 로드될 기본 주소
    DWORD SectionAlignment;         // 메모리 정렬
    DWORD FileAlignment;            // 파일 정렬
    // ... (생략)
    DWORD SizeOfImage;              // 메모리에서 이미지 크기
    DWORD SizeOfHeaders;
    DWORD CheckSum;
    WORD  Subsystem;                // GUI, Console 등
    // ...
    IMAGE_DATA_DIRECTORY DataDirectory[16];  // 중요 테이블들
} IMAGE_OPTIONAL_HEADER;
```

**중요 필드:**
- **AddressOfEntryPoint**: 프로그램이 시작되는 주소 (RVA)
- **ImageBase**: 기본 로드 주소 (보통 0x00400000)
- **DataDirectory**: Import, Export, Resource 등의 테이블 위치

### Data Directories

```c
#define IMAGE_DIRECTORY_ENTRY_EXPORT         0
#define IMAGE_DIRECTORY_ENTRY_IMPORT         1
#define IMAGE_DIRECTORY_ENTRY_RESOURCE       2
#define IMAGE_DIRECTORY_ENTRY_EXCEPTION      3
#define IMAGE_DIRECTORY_ENTRY_SECURITY       4
#define IMAGE_DIRECTORY_ENTRY_BASERELOC      5
#define IMAGE_DIRECTORY_ENTRY_DEBUG          6
#define IMAGE_DIRECTORY_ENTRY_ARCHITECTURE   7
#define IMAGE_DIRECTORY_ENTRY_GLOBALPTR      8
#define IMAGE_DIRECTORY_ENTRY_TLS            9
#define IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG   10
#define IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT  11
#define IMAGE_DIRECTORY_ENTRY_IAT           12
#define IMAGE_DIRECTORY_ENTRY_DELAY_IMPORT  13
#define IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR 14
```

### Section Headers

```c
typedef struct _IMAGE_SECTION_HEADER {
    BYTE  Name[8];              // 섹션 이름 (예: .text)
    DWORD VirtualSize;          // 메모리에서 크기
    DWORD VirtualAddress;       // 메모리에서 RVA
    DWORD SizeOfRawData;        // 파일에서 크기
    DWORD PointerToRawData;     // 파일 오프셋
    // ... (생략)
    DWORD Characteristics;      // 섹션 속성
} IMAGE_SECTION_HEADER;
```

**Characteristics 플래그:**
- `0x20000000`: IMAGE_SCN_MEM_EXECUTE (실행 가능)
- `0x40000000`: IMAGE_SCN_MEM_READ (읽기 가능)
- `0x80000000`: IMAGE_SCN_MEM_WRITE (쓰기 가능)

### 주요 섹션

#### .text
- **용도**: 실행 코드
- **속성**: 읽기, 실행
- **내용**: 컴파일된 기계어

#### .data
- **용도**: 초기화된 전역/정적 변수
- **속성**: 읽기, 쓰기
- **내용**: 초기값이 있는 데이터

#### .rdata
- **용도**: 읽기 전용 데이터
- **속성**: 읽기
- **내용**: 문자열 상수, Import 테이블

#### .bss
- **용도**: 초기화되지 않은 데이터
- **속성**: 읽기, 쓰기
- **내용**: 파일에 없고 메모리에만 존재

#### .rsrc
- **용도**: 리소스
- **속성**: 읽기
- **내용**: 아이콘, 문자열, 대화상자 등

### Import Table

프로그램이 사용하는 외부 함수(DLL)를 나열합니다.

```c
typedef struct _IMAGE_IMPORT_DESCRIPTOR {
    DWORD OriginalFirstThunk;  // INT (Import Name Table)
    DWORD TimeDateStamp;
    DWORD ForwarderChain;
    DWORD Name;                // DLL 이름 RVA
    DWORD FirstThunk;          // IAT (Import Address Table)
} IMAGE_IMPORT_DESCRIPTOR;
```

**분석 포인트:**
- 어떤 DLL을 사용하는가?
- 어떤 API를 호출하는가?
- 의심스러운 API 사용 여부

**예시:**
```
kernel32.dll
  - CreateFileA
  - WriteFile
  - CreateProcessA

user32.dll
  - MessageBoxA
  - GetWindowTextA

ws2_32.dll  ← 네트워크 사용!
  - socket
  - connect
  - send
```

### Export Table

DLL이 제공하는 함수를 나열합니다.

```c
typedef struct _IMAGE_EXPORT_DIRECTORY {
    DWORD Characteristics;
    DWORD TimeDateStamp;
    WORD  MajorVersion;
    WORD  MinorVersion;
    DWORD Name;                // DLL 이름
    DWORD Base;                // Ordinal 기준
    DWORD NumberOfFunctions;
    DWORD NumberOfNames;
    DWORD AddressOfFunctions;    // 함수 주소 배열
    DWORD AddressOfNames;        // 함수 이름 배열
    DWORD AddressOfNameOrdinals; // Ordinal 배열
} IMAGE_EXPORT_DIRECTORY;
```

### RVA와 File Offset 변환

**RVA (Relative Virtual Address)**: 메모리에서 ImageBase로부터의 상대 주소

**File Offset**: 파일에서의 위치

**변환 공식:**
```
FileOffset = RVA - SectionVirtualAddress + SectionPointerToRawData
```

---

## 🔧 디컴파일러 사용법

### Ghidra 기본 사용법

#### 1. 프로젝트 생성

```
1. Ghidra 실행
2. File > New Project
3. Non-Shared Project 선택
4. 프로젝트 위치 및 이름 지정
```

#### 2. 파일 임포트 및 분석

```
1. File > Import File
2. 분석할 파일 선택
3. Format: Portable Executable (PE) 확인
4. OK 클릭
5. 자동 분석 시작 (Analyze 클릭)
6. 분석 옵션 확인 후 Analyze 클릭
```

#### 3. 인터페이스 구성

- **Program Trees**: 섹션 구조
- **Symbol Tree**: 함수, 변수 목록
- **Listing**: 디스어셈블리 코드
- **Decompile**: 디컴파일된 C 코드
- **Console**: 로그 및 스크립트 출력

#### 4. 주요 기능

**함수 찾기:**
```
- Symbol Tree에서 Functions 폴더 확인
- Entry Point부터 시작 (symbol.entry)
- Search > For Strings로 문자열 검색
- 문자열 더블클릭 > XREF (Cross Reference) 확인
```

**주석 추가:**
```
- 코드에서 ; 키: EOL 주석
- Ctrl+Shift+; : Pre 주석
- Ctrl+; : Post 주석
- Ctrl+Alt+; : Plate 주석
```

**함수 시그니처 변경:**
```
- 함수명에서 우클릭 > Edit Function Signature
- 반환 타입, 파라미터 수정
```

**변수 이름 변경:**
```
- 변수에서 L 키
- 새 이름 입력
```

**데이터 타입 변경:**
```
- 데이터에서 우클릭 > Data > Choose Data Type
- 구조체 정의: Window > Data Type Manager
```

#### 5. 단축키

| 키 | 기능 |
|----|------|
| **G** | Go to Address |
| **L** | 이름 변경 |
| **;** | 주석 추가 |
| **D** | 데이터로 정의 |
| **C** | 코드로 정의 |
| **F** | 함수 생성 |
| **Ctrl+Shift+E** | 참조 찾기 |
| **Space** | 그래프 뷰 전환 |

### IDA 기본 사용법

#### 1. 파일 열기

```
1. IDA Pro/Free 실행
2. New > 파일 선택
3. PE 형식 확인
4. OK 클릭하여 분석 시작
```

#### 2. 주요 뷰

- **IDA View**: 디스어셈블리
- **Hex View**: 16진수 뷰
- **Functions**: 함수 목록
- **Strings**: 문자열 목록
- **Imports/Exports**: Import/Export 테이블

#### 3. 네비게이션

```
- Entry Point: 자동으로 시작
- Jump to Address: G 키
- 문자열 창: Shift+F12
- 함수 창: Shift+F3
- Cross Reference: X 키
```

#### 4. 분석 팁

**문자열 기반 분석:**
```
1. Shift+F12로 문자열 창 열기
2. 의심스러운 문자열 찾기
   - URL, IP 주소
   - 레지스트리 키
   - 파일 경로
3. 더블클릭하여 해당 위치 이동
4. X 키로 어디서 참조하는지 확인
```

**함수 흐름 분석:**
```
1. 함수 진입 (Entry Point 또는 관심 함수)
2. Space 키로 그래프 뷰 전환
3. 조건 분기 확인
4. F5 키로 디컴파일 (Hex-Rays 있는 경우)
```

### 디컴파일 결과 해석

#### 예제 1: 간단한 함수

**어셈블리:**
```asm
push    ebp
mov     ebp, esp
sub     esp, 10h
mov     [ebp-4], 0
mov     eax, [ebp-4]
add     eax, 5
mov     esp, ebp
pop     ebp
retn
```

**디컴파일:**
```c
int function() {
    int var = 0;
    return var + 5;
}
```

#### 예제 2: 조건문

**어셈블리:**
```asm
cmp     eax, ebx
jle     short loc_401020
mov     eax, 1
jmp     short loc_401025
loc_401020:
mov     eax, 0
loc_401025:
retn
```

**디컴파일:**
```c
int function(int a, int b) {
    if (a > b) {
        return 1;
    } else {
        return 0;
    }
}
```

#### 예제 3: 반복문

**어셈블리:**
```asm
mov     ecx, 0
loc_loop:
cmp     ecx, 10
jge     short loc_end
; ... (루프 내용)
inc     ecx
jmp     short loc_loop
loc_end:
```

**디컴파일:**
```c
for (int i = 0; i < 10; i++) {
    // ...
}
```

---

## 🛡️ 안티 디버깅 기법

악성코드는 분석을 방해하기 위해 다양한 안티 디버깅 기법을 사용합니다.

### 1. IsDebuggerPresent

**기법:**
```c
if (IsDebuggerPresent()) {
    ExitProcess(0);  // 디버거 탐지 시 종료
}
```

**우회:**
```
- 디버거에서 중단점 설정
- IsDebuggerPresent 호출 직후
- EAX (반환값)를 0으로 변경
```

### 2. PEB (Process Environment Block) 확인

**기법:**
```asm
mov eax, fs:[30h]      ; PEB 주소
movzx eax, byte ptr [eax+2]  ; BeingDebugged 플래그
test eax, eax
jnz being_debugged
```

**우회:**
```
- PEB의 BeingDebugged 플래그 수동 변경
- 또는 해당 명령어 NOP으로 패치
```

### 3. CheckRemoteDebuggerPresent

**기법:**
```c
BOOL bDebugged;
CheckRemoteDebuggerPresent(GetCurrentProcess(), &bDebugged);
if (bDebugged) {
    // 디버거 탐지
}
```

**우회:**
- 함수 후킹하여 항상 FALSE 반환

### 4. Timing 체크

**기법:**
```c
DWORD start = GetTickCount();
// ... (일부 코드)
DWORD end = GetTickCount();
if (end - start > 1000) {
    // 너무 느림 = 디버깅 중
}
```

또는:
```asm
rdtsc           ; 타임스탬프 카운터 읽기
mov ebx, eax
; ... (일부 코드)
rdtsc
sub eax, ebx
cmp eax, 10000h
ja  being_debugged
```

**우회:**
- 타이밍 체크 코드 NOP으로 패치
- 또는 조건 분기 반전

### 5. 하드웨어 브레이크포인트 탐지

**기법:**
```c
CONTEXT ctx;
ctx.ContextFlags = CONTEXT_DEBUG_REGISTERS;
GetThreadContext(GetCurrentThread(), &ctx);
if (ctx.Dr0 || ctx.Dr1 || ctx.Dr2 || ctx.Dr3) {
    // 하드웨어 BP 탐지
}
```

**우회:**
- 해당 체크 루틴 패치
- 또는 GetThreadContext 후킹

### 6. Exception 기반

**기법:**
```c
__try {
    // INT 3 또는 기타 예외 발생
    __asm int 3;
} __except(EXCEPTION_EXECUTE_HANDLER) {
    // 디버거 없으면 여기 도달
}
```

**우회:**
- 예외 발생 시 디버거 동작 조정
- Pass exception to application

### 7. 프로세스 및 창 탐지

**기법:**
```c
// 디버거 프로세스 확인
CreateToolhelp32Snapshot();
Process32First() / Process32Next();
// "ollydbg.exe", "x64dbg.exe" 등 검색

// 디버거 창 확인
FindWindow("OLLYDBG", NULL);
FindWindow("WinDbgFrameClass", NULL);
```

**우회:**
- 디버거 프로세스/창 이름 변경
- 또는 해당 API 후킹

### 안티 디버깅 우회 전략

#### 1. 정적 패치
```
- 안티 디버깅 코드를 NOP으로 교체
- 조건 분기를 무조건 분기로 변경
- 파일에 직접 패치하여 저장
```

#### 2. 동적 우회
```
- 디버거에서 실시간으로 값 변경
- API 반환값 조작
- 플래그 레지스터 수정
```

#### 3. 플러그인 사용
```
- ScyllaHide (x64dbg 플러그인)
- Anti-Anti-Debug 플러그인
- 자동으로 다양한 안티 디버깅 우회
```

#### 4. 커널 디버거 사용
```
- WinDbg 커널 모드 디버깅
- 대부분의 유저 모드 안티 디버깅 우회
- 복잡하지만 강력함
```

---

## 📚 추가 학습 자료

### 다음 단계

이제 기본 개념을 이해했다면, 다음으로 진행하세요:

1. 📦 [실습 환경 설정](setup-environment.md)
2. 📋 [단계별 분석 체크리스트](analysis-steps.md)
3. 🔍 [Petya 분석 가이드](petya-analysis-guide.md)

### 연습 문제

기본 개념을 익히기 위한 실습:

1. **간단한 프로그램 분석**
   - "Hello World" 프로그램 디스어셈블
   - Entry Point 찾기
   - MessageBox 호출 추적

2. **PE 구조 분석**
   - PEStudio로 실행 파일 열기
   - Import 테이블 확인
   - 섹션 특성 분석

3. **어셈블리 읽기**
   - 간단한 함수의 어셈블리 코드 읽기
   - 스택 프레임 이해
   - 함수 호출 추적

4. **디버깅 실습**
   - x64dbg로 프로그램 로드
   - Breakpoint 설정
   - Step Into/Over 연습
   - 레지스터 값 관찰

---

## 🔗 관련 문서

- [도구 목록](../resources/tools.md)
- [참고 자료](../resources/references.md)
- [분석 로그 템플릿](../notes/analysis-log-template.md)

---

**💡 팁**: 리버싱은 많은 연습이 필요합니다. 간단한 프로그램부터 시작하여 점차 복잡한 프로그램으로 나아가세요!
