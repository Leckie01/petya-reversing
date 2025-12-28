# 단계별 분석 체크리스트

## 📚 목차

- [개요](#-개요)
- [Step 1: 초기 파일 분석](#-step-1-초기-파일-분석)
- [Step 2: PE 헤더 분석](#-step-2-pe-헤더-분석)
- [Step 3: 정적 코드 분석](#-step-3-정적-코드-분석)
- [Step 4: 동적 분석](#-step-4-동적-분석)
- [Step 5: 분석 결과 정리](#-step-5-분석-결과-정리)

---

## 📋 개요

이 문서는 악성코드 분석을 체계적으로 수행하기 위한 **단계별 체크리스트**입니다. 각 단계를 순서대로 따라가며 분석을 진행하세요.

### 사용 방법

1. **순서대로 진행**: 각 단계를 건너뛰지 말고 순서대로 수행
2. **체크리스트 활용**: 각 항목을 완료하면 체크 표시
3. **결과 기록**: 각 단계의 결과를 분석 로그에 기록
4. **유연한 적용**: 샘플에 따라 일부 단계는 생략 가능

### 분석 시간 예상

```
Step 1: 초기 파일 분석      - 15-30분
Step 2: PE 헤더 분석        - 20-40분
Step 3: 정적 코드 분석      - 1-3시간
Step 4: 동적 분석           - 1-2시간
Step 5: 분석 결과 정리      - 30분-1시간

총 소요 시간: 약 3-7시간 (샘플 복잡도에 따라 다름)
```

---

## 🔍 Step 1: 초기 파일 분석

### 1.1 파일 해시 확인

**목적**: 파일의 고유 식별자를 생성하고 기존 데이터베이스와 비교

**체크리스트:**
```
[ ] MD5 해시 계산
[ ] SHA1 해시 계산
[ ] SHA256 해시 계산
[ ] 해시값 기록
```

**명령어 (Windows):**
```cmd
certutil -hashfile sample.exe MD5
certutil -hashfile sample.exe SHA1
certutil -hashfile sample.exe SHA256
```

**명령어 (Linux/macOS):**
```bash
md5sum sample.exe
sha1sum sample.exe
sha256sum sample.exe
```

**PowerShell:**
```powershell
Get-FileHash sample.exe -Algorithm MD5
Get-FileHash sample.exe -Algorithm SHA1
Get-FileHash sample.exe -Algorithm SHA256
```

**결과 기록:**
```
MD5:    [해시값]
SHA1:   [해시값]
SHA256: [해시값]
```

### 1.2 파일 타입 및 크기 확인

**목적**: 파일의 기본 속성과 실제 타입 확인

**체크리스트:**
```
[ ] 파일 크기 확인
[ ] 파일 타입 확인 (PE32/PE32+)
[ ] 아키텍처 확인 (x86/x64)
[ ] 컴파일 시간 확인
```

**명령어 (Windows):**
```cmd
dir sample.exe
```

**명령어 (Linux):**
```bash
file sample.exe
ls -lh sample.exe
stat sample.exe
```

**결과 예시:**
```
파일명: malware.exe
크기: 147,456 bytes (144 KB)
타입: PE32 executable (GUI) Intel 80386, for MS Windows
아키텍처: x86 (32-bit)
```

### 1.3 문자열 추출 및 분석

**목적**: 파일에 포함된 텍스트를 분석하여 단서 찾기

**체크리스트:**
```
[ ] ASCII 문자열 추출
[ ] Unicode 문자열 추출
[ ] 의심스러운 문자열 식별
[ ] 주요 문자열 분류 및 기록
```

**명령어 (Windows):**
```cmd
# Sysinternals Strings 사용
strings.exe sample.exe > strings_output.txt
strings.exe -u sample.exe > strings_unicode.txt

# 또는 PowerShell
Get-Content sample.exe -Encoding Byte | 
    ForEach-Object { [char]$_ } | Out-File strings.txt
```

**명령어 (Linux):**
```bash
strings sample.exe > strings_ascii.txt
strings -e l sample.exe > strings_unicode.txt  # little-endian UTF-16
```

**분석할 문자열 카테고리:**
```
✓ URL 및 도메인:
  - http://, https://, ftp://
  - .com, .net, .onion
  
✓ IP 주소:
  - 정규식: \d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}
  
✓ 파일 경로:
  - C:\, %TEMP%, %APPDATA%
  - /etc/, /tmp/, /var/
  
✓ 레지스트리 키:
  - HKEY_LOCAL_MACHINE
  - HKEY_CURRENT_USER
  - Software\Microsoft\Windows\CurrentVersion\Run
  
✓ API 함수 이름:
  - CreateFile, WriteFile, RegSetValue
  - socket, connect, send
  
✓ 암호화 관련:
  - "AES", "RSA", "key", "encrypt"
  - Base64 인코딩 문자열
  
✓ 에러 메시지 및 디버그 문자열:
  - "Error", "Failed", "Debug"
  - 개발자 경로 (C:\Users\Dev\...)
```

**필터링 팁:**
```bash
# URL 찾기
strings sample.exe | grep -i "http"

# IP 주소 찾기
strings sample.exe | grep -E "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b"

# 레지스트리 키 찾기
strings sample.exe | grep -i "HKEY"

# Base64 패턴 찾기
strings sample.exe | grep -E "^[A-Za-z0-9+/]{20,}={0,2}$"
```

### 1.4 패커/프로텍터 탐지

**목적**: 파일이 패킹되어 있는지 확인

**체크리스트:**
```
[ ] Detect It Easy (DIE)로 스캔
[ ] PEiD 시그니처 확인
[ ] 엔트로피 분석
[ ] 패커 유형 식별
[ ] 언패킹 필요 여부 결정
```

**도구 사용:**

**Detect It Easy (DIE):**
```cmd
# GUI 모드
die.exe sample.exe

# 콘솔 모드
diec.exe sample.exe
```

**결과 해석:**
```
패커 없음:
- Compiler: Microsoft Visual C++ 8.0
- Linker: Microsoft Linker
- Entropy: 낮음 (< 6.0)

패킹됨:
- Packer: UPX 3.96
- Entropy: 높음 (> 7.0)
- Sections: 비정상적 (.upx0, .upx1)
```

**엔트로피 분석:**
```
엔트로피 값 해석:
0.0 - 3.0: 매우 낮음 (텍스트 파일)
3.0 - 5.0: 낮음 (일반 실행 파일)
5.0 - 7.0: 중간 (일부 압축/암호화)
7.0 - 8.0: 높음 (강한 압축/암호화)

⚠️ 엔트로피 > 7.0 → 패킹 또는 암호화 의심
```

**언패킹:**
```
일반적인 패커:
- UPX: upx -d sample.exe
- ASPack: Generic unpacker 사용
- Themida: 수동 언패킹 필요

언패킹 후:
- 다시 Step 1부터 분석 반복
```

### 1.5 온라인 샌드박스 조회

**목적**: 기존 분석 결과 확인 (샘플 업로드는 금지)

**체크리스트:**
```
[ ] VirusTotal 해시 검색
[ ] Hybrid Analysis 검색
[ ] Any.run 검색
[ ] 기존 보고서 확인
```

**VirusTotal 검색:**
```
1. virustotal.com 방문
2. Search 탭 선택
3. 해시값 입력 (MD5, SHA1, SHA256)
4. ⚠️ 파일 업로드하지 않기 (공개됨)
```

**확인 사항:**
```
✓ 탐지율: X/Y (X개 엔진이 탐지)
✓ 첫 제출 시간
✓ 파일 이름 (다른 샘플과 연관성)
✓ 행위 정보 (Behavior 탭)
✓ 네트워크 통신 (Relations 탭)
✓ YARA 규칙 매치
```

**주의사항:**
```
⚠️ 미공개 샘플은 온라인에 업로드하지 마세요
⚠️ 조직 정보가 포함된 샘플 주의
⚠️ VirusTotal은 샘플을 공개적으로 공유합니다
```

---

## 📦 Step 2: PE 헤더 분석

### 2.1 PE 구조 분석 도구 사용

**목적**: PE 파일 구조를 시각화하고 이상 징후 탐지

**체크리스트:**
```
[ ] PEStudio로 스캔
[ ] PE-bear로 헤더 검사
[ ] CFF Explorer로 상세 분석
```

**PEStudio 분석:**
```
1. PEStudio 실행
2. sample.exe 드래그 앤 드롭
3. 좌측 메뉴 순서대로 확인:
   
   [ ] indicators - 의심 지표 자동 표시
   [ ] file-header - 기본 파일 정보
   [ ] optional-header - 선택 헤더
   [ ] sections - 섹션 정보
   [ ] imports - Import 테이블
   [ ] exports - Export 테이블 (DLL인 경우)
   [ ] resources - 리소스 (아이콘, 문자열 등)
   [ ] version - 버전 정보
   [ ] strings - 문자열 (형식별)
```

### 2.2 Import Table 분석

**목적**: 사용된 API 함수를 통해 기능 추정

**체크리스트:**
```
[ ] Import된 DLL 목록 확인
[ ] 각 DLL의 함수 목록 확인
[ ] 의심스러운 API 식별
[ ] API 카테고리별 분류
```

**주요 DLL 및 기능:**

**kernel32.dll (시스템 기본 기능):**
```
파일 조작:
- CreateFileA/W: 파일 생성/열기
- ReadFile, WriteFile: 파일 읽기/쓰기
- DeleteFileA/W: 파일 삭제
- MoveFileA/W: 파일 이동
- FindFirstFileA/W, FindNextFileA/W: 파일 검색

프로세스/스레드:
- CreateProcessA/W: 프로세스 생성
- CreateThread: 스레드 생성
- TerminateProcess: 프로세스 종료
- OpenProcess: 프로세스 핸들 획득

메모리:
- VirtualAlloc: 메모리 할당
- VirtualProtect: 메모리 보호 속성 변경
- WriteProcessMemory: 다른 프로세스 메모리 쓰기 ⚠️

기타:
- LoadLibraryA/W: DLL 동적 로드
- GetProcAddress: 함수 주소 획득
- Sleep: 대기
```

**advapi32.dll (고급 API):**
```
레지스트리:
- RegOpenKeyExA/W: 레지스트리 키 열기
- RegSetValueExA/W: 레지스트리 값 설정 ⚠️
- RegQueryValueExA/W: 레지스트리 값 읽기
- RegCreateKeyExA/W: 레지스트리 키 생성
- RegDeleteKeyA/W: 레지스트리 키 삭제

서비스:
- CreateServiceA/W: 서비스 생성 ⚠️
- StartServiceA/W: 서비스 시작
- OpenSCManagerA/W: 서비스 관리자 열기

암호화:
- CryptAcquireContextA/W: 암호화 컨텍스트 획득
- CryptEncrypt, CryptDecrypt: 암호화/복호화
- CryptGenRandom: 난수 생성
```

**user32.dll (UI):**
```
- MessageBoxA/W: 메시지 박스 표시
- GetForegroundWindow: 활성 창 획득
- SetWindowsHookExA/W: 키로거용 후킹 ⚠️
- GetAsyncKeyState: 키 입력 감지 ⚠️
```

**ws2_32.dll (네트워크):**
```
- socket: 소켓 생성 ⚠️
- connect: 연결 ⚠️
- send, recv: 데이터 송수신 ⚠️
- WSAStartup: 윈속 초기화
- gethostbyname: 호스트 이름 조회
```

**wininet.dll (인터넷):**
```
- InternetOpenA/W: 인터넷 세션 시작 ⚠️
- InternetConnectA/W: 서버 연결 ⚠️
- HttpOpenRequestA/W: HTTP 요청 생성
- HttpSendRequestA/W: HTTP 요청 전송
- InternetReadFile: 데이터 다운로드
```

**ntdll.dll (시스템 저수준):**
```
- NtCreateFile: 파일 생성 (저수준)
- NtWriteFile: 파일 쓰기 (저수준)
- NtRaiseHardError: 블루스크린 유발 ⚠️⚠️
- RtlAdjustPrivilege: 권한 조정
```

**⚠️ 표시된 API는 악성 행위에 자주 사용됨**

**의심스러운 API 조합:**
```
랜섬웨어 패턴:
- CryptAcquireContext + CryptEncrypt
- FindFirstFile + FindNextFile (파일 검색)
- WriteFile (암호화된 데이터 쓰기)
- DeleteFile (원본 삭제)

키로거 패턴:
- SetWindowsHookEx + GetAsyncKeyState
- CreateFile (로그 파일 생성)

다운로더 패턴:
- InternetOpen + InternetConnect
- InternetReadFile + WriteFile

프로세스 인젝션:
- VirtualAllocEx + WriteProcessMemory
- CreateRemoteThread
```

### 2.3 Export Table 분석

**목적**: DLL이 제공하는 함수 확인 (DLL인 경우)

**체크리스트:**
```
[ ] Export 함수 목록 확인
[ ] Ordinal 번호 확인
[ ] 함수 이름 패턴 분석
```

**분석 방법:**
```
PEStudio > exports 탭:
- 함수 이름
- Ordinal 번호
- RVA (상대 가상 주소)
```

**일반 DLL vs 악성 DLL:**
```
정상 DLL:
- 의미 있는 함수 이름
- 문서화된 API
- 일관된 명명 규칙

악성 DLL:
- 난독화된 이름 (a, b, c, func1)
- 숨겨진 기능
- 비표준 ordinal 사용
```

### 2.4 리소스 섹션 확인

**목적**: 내장된 리소스 (아이콘, 문자열, 데이터) 분석

**체크리스트:**
```
[ ] 리소스 타입 확인
[ ] 아이콘 추출 및 확인
[ ] 문자열 테이블 분석
[ ] 추가 PE 파일 확인 (드롭퍼)
[ ] 의심스러운 데이터 확인
```

**리소스 타입:**
```
RT_ICON: 아이콘
RT_BITMAP: 비트맵 이미지
RT_STRING: 문자열 테이블
RT_RCDATA: 사용자 정의 데이터 ⚠️
RT_DIALOG: 대화상자
RT_MANIFEST: 애플리케이션 매니페스트
```

**분석 도구:**
```
CFF Explorer:
1. Resource Editor 섹션
2. 각 리소스 타입 확장
3. 의심스러운 항목 더블클릭
4. 내용 확인 또는 추출

Resource Hacker:
1. sample.exe 열기
2. 트리 구조로 리소스 탐색
3. 저장 (Save Resource)
```

**의심 징후:**
```
⚠️ 비정상적으로 큰 RCDATA
⚠️ PE 시그니처(MZ)가 있는 리소스 → 드롭퍼
⚠️ 암호화된 데이터 (높은 엔트로피)
⚠️ 압축된 실행 파일
```

### 2.5 엔트로피 분석

**목적**: 각 섹션의 엔트로피를 계산하여 암호화/패킹 탐지

**체크리스트:**
```
[ ] 각 섹션의 엔트로피 확인
[ ] 높은 엔트로피 섹션 식별
[ ] 비정상적인 섹션 조사
```

**PEStudio에서 확인:**
```
sections 탭:
- Name: 섹션 이름
- Size: 섹션 크기
- Entropy: 엔트로피 값
- Characteristics: 속성 (읽기/쓰기/실행)
```

**정상 범위:**
```
.text (코드):   5.5 - 6.5 (중간)
.data (데이터): 3.0 - 5.0 (낮음)
.rdata (읽기):  4.0 - 6.0 (낮음-중간)
.rsrc (리소스): 5.0 - 7.0 (중간-높음, 압축된 이미지)

⚠️ > 7.0: 암호화 또는 강한 압축 의심
```

**비정상적인 섹션:**
```
⚠️ 실행 권한이 있는 .data 섹션
⚠️ 쓰기 권한이 있는 .text 섹션
⚠️ 표준이 아닌 이름 (.upx, .aspack, .petite)
⚠️ 크기가 0인 섹션
⚠️ RawSize와 VirtualSize가 크게 다름
```

---

## 🔬 Step 3: 정적 코드 분석

### 3.1 Ghidra/IDA로 디스어셈블

**목적**: 기계어를 어셈블리 및 C 코드로 변환하여 분석

**체크리스트:**
```
[ ] Ghidra 프로젝트 생성
[ ] 파일 임포트
[ ] 자동 분석 수행
[ ] Entry Point 확인
```

**Ghidra 워크플로우:**
```
1. File > New Project
   - Project Name: Sample_Analysis
   - Project Directory: C:\Analysis\Projects

2. File > Import File
   - sample.exe 선택
   - Format: Portable Executable (PE) 자동 감지
   
3. 더블클릭하여 CodeBrowser 열기

4. Analysis 대화상자:
   ✓ Analyze Now 선택
   ✓ 기본 옵션 유지
   ✓ 분석 시작 (1-5분 소요)
```

### 3.2 주요 함수 식별

**목적**: 악성 행위를 수행하는 핵심 함수 찾기

**체크리스트:**
```
[ ] Entry Point 분석
[ ] WinMain 또는 main 함수 찾기
[ ] 문자열 참조로 함수 추적
[ ] Import 함수 호출 추적
```

**Entry Point 찾기:**
```
Ghidra:
1. Symbol Tree > Functions > entry
2. 더블클릭하여 이동
3. Decompile 창에서 C 코드 확인
```

**WinMain 찾기:**
```
entry 함수에서:
- _WinMain@16 또는 WinMain 호출 찾기
- CALL 명령어 추적
- F 키로 함수 생성 (필요 시)
```

**문자열 기반 함수 추적:**
```
1. Search > For Strings
2. 필터: minimum length 4, encoding: all
3. 의심스러운 문자열 찾기:
   - URL, IP 주소
   - 파일 경로
   - 레지스트리 키
4. 문자열 더블클릭
5. XREF (Ctrl+Shift+F) 확인
6. 참조하는 함수로 이동
```

**함수 명명:**
```
분석하면서 함수에 의미 있는 이름 부여:
- L 키: 함수/변수 이름 변경
- 예시:
  * FUN_004010a0 → EncryptFile
  * FUN_00401200 → ConnectToC2
  * FUN_00401500 → DropPayload
```

### 3.3 코드 흐름 분석

**목적**: 프로그램의 실행 흐름과 로직 이해

**체크리스트:**
```
[ ] 제어 흐름 그래프 확인
[ ] 조건 분기 분석
[ ] 루프 식별
[ ] 함수 호출 관계 추적
```

**제어 흐름 그래프 (CFG):**
```
Ghidra:
1. 함수 선택
2. Window > Function Graph
   또는 Space 키
3. 그래프 모드로 전환

분석 포인트:
- 시작 블록 (초록색)
- 조건 분기 (다이아몬드 형태)
- 루프 (역방향 화살표)
- 종료 블록 (빨간색)
```

**조건 분기 분석:**
```asm
; 예시 어셈블리
CMP EAX, 0
JE  loc_success    ; Jump if Equal (ZF=1)
JNE loc_fail       ; Jump if Not Equal (ZF=0)

loc_success:
    ; 성공 경로
    ...
    JMP end

loc_fail:
    ; 실패 경로
    ...
    
end:
    RET
```

**C 디컴파일 결과:**
```c
if (result == 0) {
    // 성공 경로
} else {
    // 실패 경로
}
```

**루프 식별:**
```c
// for 루프 패턴
for (int i = 0; i < 100; i++) {
    // 반복 작업
}

// while 루프 패턴
while (condition) {
    // 조건 만족 시 반복
}

// 파일 검색 루프 (악성코드에서 흔함)
HANDLE hFind = FindFirstFile("*.*", &fd);
if (hFind != INVALID_HANDLE_VALUE) {
    do {
        // 각 파일 처리
    } while (FindNextFile(hFind, &fd));
}
```

### 3.4 암호화 루틴 찾기

**목적**: 암호화 알고리즘 및 키 관리 방식 파악

**체크리스트:**
```
[ ] 암호화 API 호출 찾기
[ ] 암호화 상수 식별
[ ] 키 생성 루틴 분석
[ ] 암호화 알고리즘 추정
```

**암호화 API 추적:**
```
Imports에서 찾기:
- CryptAcquireContext
- CryptGenKey
- CryptEncrypt
- CryptDecrypt

또는:
- BCryptOpenAlgorithmProvider (CNG API)
- BCryptEncrypt
```

**암호화 상수 식별:**
```
AES:
- S-box 상수: 0x63, 0x7c, 0x77...
- Rcon 배열

RSA:
- 큰 소수 (1024/2048비트)
- 지수 e = 0x10001 (65537)

Salsa20/ChaCha20:
- "expand 32-byte k" 문자열
- "expand 16-byte k"

RC4:
- 256바이트 S-box 초기화
```

**Ghidra에서 상수 검색:**
```
1. Search > For Scalars
2. 값 입력 (예: 0x61707865 - Salsa20)
3. 결과에서 코드 위치 확인
4. 주변 코드 분석
```

**키 생성 루틴:**
```c
// 예시: 약한 키 생성
void GenerateKey(BYTE *key, DWORD keyLen) {
    // 시간 기반 시드 (약함!)
    srand(GetTickCount());
    for (int i = 0; i < keyLen; i++) {
        key[i] = rand() % 256;
    }
}

// 강한 키 생성
void GenerateKeyStrong(BYTE *key, DWORD keyLen) {
    HCRYPTPROV hProv;
    CryptAcquireContext(&hProv, NULL, NULL, 
                        PROV_RSA_FULL, 0);
    CryptGenRandom(hProv, keyLen, key);
    CryptReleaseContext(hProv, 0);
}
```

### 3.5 문자열 참조 추적

**목적**: 문자열 사용 위치 파악 및 기능 추정

**체크리스트:**
```
[ ] 주요 문자열 위치 확인
[ ] 각 문자열의 XREF 확인
[ ] 함수 기능 추정
```

**XREF (Cross Reference) 확인:**
```
Ghidra:
1. 문자열 더블클릭하여 데이터 위치 이동
2. 문자열 주소에서 우클릭
3. References > Show References to Address
   또는 Ctrl+Shift+F
4. 리스트에서 참조 위치 확인
5. 더블클릭하여 코드 이동
```

**예시 분석:**
```
문자열: "C:\\Windows\\System32\\cmd.exe"

XREF 위치: FUN_00401234

디컴파일 결과:
CreateProcessA(NULL, 
               "C:\\Windows\\System32\\cmd.exe",
               ...);

→ 이 함수는 명령 프롬프트를 실행하는 기능
→ 함수명을 LaunchCommandPrompt로 변경
```

### 3.6 API 호출 분석

**목적**: 각 API의 사용 목적 파악

**체크리스트:**
```
[ ] Import 함수 호출 위치 찾기
[ ] 파라미터 분석
[ ] 반환값 사용 방식 확인
```

**API 호출 추적:**
```
Ghidra:
1. Symbol Tree > Imports > [DLL] > [함수명]
2. 더블클릭
3. XREF로 호출 위치 확인
```

**파라미터 분석:**
```c
// CreateFileA 예시
HANDLE hFile = CreateFileA(
    "C:\\secret.txt",        // lpFileName
    GENERIC_WRITE,           // dwDesiredAccess
    0,                       // dwShareMode
    NULL,                    // lpSecurityAttributes
    CREATE_ALWAYS,           // dwCreationDisposition
    FILE_ATTRIBUTE_NORMAL,   // dwFlagsAndAttributes
    NULL                     // hTemplateFile
);

분석:
- 파일명: C:\\secret.txt
- 접근: 쓰기 전용
- 생성: 항상 새로 생성 (기존 파일 덮어쓰기)
→ 데이터를 파일에 쓰는 기능
```

---

## 🖥️ Step 4: 동적 분석

⚠️ **경고**: 격리된 VM에서만 수행!

### 4.1 디버거 설정 및 첫 실행

**목적**: 실행 흐름을 제어하며 관찰

**체크리스트:**
```
[ ] x64dbg 실행 및 파일 로드
[ ] 초기 중단점 설정
[ ] 모니터링 도구 시작
```

**x64dbg 설정:**
```
1. x64dbg 실행 (x32/x64 선택)
2. File > Open > sample.exe
3. Options > Preferences:
   - Events > Entry Breakpoint: ✓
   - Exception > Ignore all exceptions: ✗
4. Debug > Run (F9)
5. Entry Point에서 중단
```

**초기 중단점 설정:**
```
Ctrl+G (Go to Expression):
- CreateFileA
- WriteFile
- InternetOpenA
- RegSetValueExA
- VirtualAlloc
- CreateProcessA

각 API에서 F2 (Toggle Breakpoint)
```

### 4.2 중단점 설정 전략

**목적**: 효과적인 디버깅을 위한 중단점 배치

**체크리스트:**
```
[ ] API 중단점 설정
[ ] 조건부 중단점 설정
[ ] 메모리 중단점 설정 (필요 시)
```

**중단점 종류:**

**1. 일반 중단점 (Software BP):**
```
- F2 키 또는 클릭
- 특정 주소에서 실행 중단
- 코드 패치 (INT 3)
```

**2. 하드웨어 중단점:**
```
우클릭 > Breakpoint > Hardware, Execute
- CPU 레지스터 사용 (DR0-DR3)
- 최대 4개까지
- 안티 디버깅에 탐지될 수 있음
```

**3. 조건부 중단점:**
```
중단점 우클릭 > Edit:
- Condition: EAX == 1
- Log: "{eax}" (레지스터 값 로깅)
- Command: 자동 명령 실행
```

**4. 메모리 중단점:**
```
메모리 뷰에서 우클릭 > Breakpoint > Memory, Access
- 특정 메모리 접근 시 중단
- 암호화 키 접근 추적에 유용
```

### 4.3 API 호출 모니터링

**목적**: 실행 중 API 호출 관찰

**체크리스트:**
```
[ ] 각 API 호출 시 파라미터 확인
[ ] 반환값 확인
[ ] 로그 기록
```

**API 중단점에서 확인:**

**CreateFileA 예시:**
```
중단 시 스택 확인:
ESP+0x04: lpFileName (파일명 포인터)
ESP+0x08: dwDesiredAccess
ESP+0x0C: dwShareMode
...

메모리 뷰:
- lpFileName 주소로 이동
- 문자열 확인: "C:\\target.txt"

로그:
CreateFileA("C:\\target.txt", GENERIC_WRITE, ...)
```

**WriteFile 예시:**
```
ESP+0x04: hFile (파일 핸들)
ESP+0x08: lpBuffer (버퍼 포인터)
ESP+0x0C: nNumberOfBytesToWrite

버퍼 내용 확인:
- 메모리 뷰에서 lpBuffer 이동
- 쓰여질 데이터 확인
- 암호화된 데이터인지 확인 (엔트로피)
```

### 4.4 파일 시스템 변경 추적

**목적**: 파일 생성/수정/삭제 모니터링

**체크리스트:**
```
[ ] Process Monitor 시작
[ ] 필터 설정
[ ] 실행 후 이벤트 분석
```

**Process Monitor (ProcMon) 사용:**
```
1. Procmon.exe 실행 (관리자 권한)
2. Ctrl+L: 필터 설정
   - Process Name is sample.exe
   - Include
3. Ctrl+E: 캡처 시작
4. 디버거에서 샘플 실행
5. Ctrl+E: 캡처 중지
```

**이벤트 분석:**
```
Operation 종류:
- CreateFile: 파일 열기/생성
- WriteFile: 파일 쓰기 ⚠️
- ReadFile: 파일 읽기
- SetDispositionInformationFile: 삭제 표시 ⚠️
- QueryDirectory: 디렉토리 검색 ⚠️

필터 설정:
- Operation is WriteFile
- Path contains .txt (확장자 필터)
```

**주목할 패턴:**
```
랜섬웨어:
1. QueryDirectory (파일 검색)
2. CreateFile (원본 파일 열기)
3. ReadFile (데이터 읽기)
4. CreateFile (새 파일 생성, .encrypted)
5. WriteFile (암호화된 데이터 쓰기)
6. SetDispositionInformationFile (원본 삭제)
```

### 4.5 레지스트리 변경 추적

**목적**: 레지스트리 수정 모니터링 (지속성 확인)

**체크리스트:**
```
[ ] ProcMon으로 레지스트리 작업 필터
[ ] 변경된 키 확인
[ ] 지속성 메커니즘 식별
```

**레지스트리 필터:**
```
ProcMon:
- Operation begins with Reg
- Include

주요 Operation:
- RegOpenKey
- RegSetValue ⚠️
- RegCreateKey ⚠️
- RegQueryValue
```

**지속성 확인 위치:**
```
Run 키:
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\Software\Microsoft\Windows\CurrentVersion\Run

RunOnce 키:
HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce

서비스:
HKLM\SYSTEM\CurrentControlSet\Services\[서비스명]

예약된 작업:
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tasks
```

### 4.6 네트워크 활동 모니터링

**목적**: 네트워크 통신 감지 및 분석

**체크리스트:**
```
[ ] Wireshark 캡처 시작
[ ] 샘플 실행
[ ] 트래픽 분석
[ ] C&C 서버 식별
```

**Wireshark 사용:**
```
1. Wireshark 시작
2. 캡처 인터페이스 선택 (VM 네트워크)
3. 캡처 시작 (Capture > Start)
4. 샘플 실행
5. 캡처 중지
```

**필터:**
```
기본 필터:
ip.addr == [샘플 IP]
tcp
http
dns

프로토콜별:
http.request      # HTTP 요청
dns.qry.name      # DNS 쿼리
tcp.port == 80    # HTTP 트래픽
tcp.port == 443   # HTTPS 트래픽
```

**분석 항목:**
```
✓ DNS 쿼리: 어떤 도메인을 조회?
✓ TCP 연결: 어느 IP/포트로 연결?
✓ HTTP 요청: GET/POST 데이터는?
✓ 사용자 에이전트: 정상인지 위장인지?
✓ Payload: 업로드/다운로드 데이터
```

**Process Hacker 네트워크:**
```
1. sample.exe 프로세스 선택
2. 우클릭 > Properties
3. Network 탭
4. 활성 연결 확인:
   - Remote Address
   - Remote Port
   - State (ESTABLISHED, LISTEN)
```

### 4.7 메모리 덤프 및 분석

**목적**: 런타임 메모리 상태 캡처 및 분석

**체크리스트:**
```
[ ] 프로세스 메모리 덤프
[ ] 메모리에서 문자열 추출
[ ] 암호화 키 검색
[ ] 언패킹된 코드 추출
```

**메모리 덤프 생성:**

**x64dbg:**
```
1. Plugins > Scylla
2. Dump > Full Dump
3. 저장 위치 지정
```

**Process Hacker:**
```
1. sample.exe 우클릭
2. Properties > Memory
3. Save All Modules
4. 개별 영역 덤프: 우클릭 > Save
```

**덤프 분석:**
```bash
# 문자열 추출
strings memdump.dmp > strings_mem.txt

# 암호화 키 패턴 검색
strings -n 16 memdump.dmp | grep -E "^[A-Fa-f0-9]{32,}$"

# PE 파일 찾기
binwalk memdump.dmp
```

**메모리에서 키 찾기 (x64dbg):**
```
Memory Map:
1. Ctrl+M: 메모리 맵 열기
2. Search > Pattern
3. 패턴 입력:
   - ASCII: "key", "password"
   - Hex: 암호화 상수
4. 결과에서 주변 메모리 확인
```

---

## 📊 Step 5: 분석 결과 정리

### 5.1 IOC 추출

**목적**: 탐지 및 대응에 사용할 지표 수집

**체크리스트:**
```
[ ] 파일 해시 기록
[ ] IP 주소 및 도메인 추출
[ ] 레지스트리 키 목록
[ ] 파일 경로 목록
[ ] Mutex/이벤트 이름
```

**IOC 카테고리:**

**1. 파일 IOC:**
```
해시:
- MD5: [해시]
- SHA1: [해시]
- SHA256: [해시]

파일명:
- sample.exe
- dropped.dll
- config.dat

경로:
- C:\Windows\Temp\malware.exe
- %APPDATA%\Microsoft\payload.bin
- C:\ProgramData\config.dat

크기:
- 147,456 bytes
```

**2. 네트워크 IOC:**
```
IP 주소:
- 192.168.1.100 (C&C 서버)
- 10.0.0.5 (내부 전파)

도메인:
- malicious-domain.com
- example.onion

URL:
- http://malicious-domain.com/gate.php
- https://example.com/download.exe

User-Agent:
- Mozilla/5.0 (특이한 패턴)
```

**3. 레지스트리 IOC:**
```
키:
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
- 값: "Updater"
- 데이터: "C:\Users\Public\malware.exe"

HKLM\SOFTWARE\[악성 소프트웨어명]
- 설정 데이터

HKLM\SYSTEM\CurrentControlSet\Services\[악성 서비스]
```

**4. Mutex/Object IOC:**
```
Mutex 이름:
- Global\MalwareMutex
- Local\UniqueID_12345

Event 이름:
- Global\SyncEvent

Named Pipe:
- \\.\pipe\malware_pipe
```

**5. 행위 IOC:**
```
- MBR 쓰기 시도
- PhysicalDrive0 직접 접근
- 대량 파일 암호화
- 특정 포트로 연결 (8080, 9050)
```

### 5.2 행위 요약 작성

**목적**: 악성코드의 전체 동작을 요약

**체크리스트:**
```
[ ] 감염 벡터 설명
[ ] 초기 실행 동작
[ ] 지속성 메커니즘
[ ] 주요 악성 행위
[ ] C&C 통신 (존재 시)
[ ] 피해 영향
```

**행위 요약 템플릿:**
```markdown
## 행위 요약

### 감염 벡터
- 방법: 피싱 이메일, Dropbox 링크
- 대상: 기업 사용자

### 초기 실행
1. UAC 우회 시도
2. 관리자 권한 획득
3. %TEMP%에 임시 파일 생성

### 지속성
- 레지스트리 Run 키 생성
- HKCU\Software\Microsoft\Windows\CurrentVersion\Run
- 값: "SystemUpdate" = "C:\Users\Public\malware.exe"

### 악성 행위
1. 파일 검색 (*.doc, *.xls, *.pdf)
2. Salsa20 암호화 적용
3. 확장자 .encrypted 추가
4. 원본 파일 삭제

### C&C 통신
- 서버: 192.168.1.100:8080
- 프로토콜: HTTP POST
- 데이터: 감염 ID, 파일 목록

### 피해 영향
- 파일 암호화로 인한 데이터 손실
- 업무 중단
- 복구 비용
```

### 5.3 탐지 시그니처 생성

**목적**: IDS/IPS, AV 등에서 사용할 탐지 규칙 생성

**체크리스트:**
```
[ ] YARA 규칙 작성
[ ] Snort 규칙 작성 (네트워크)
[ ] Sigma 규칙 작성 (로그)
```

**YARA 규칙 예시:**
```yara
rule Malware_Sample_Detected {
    meta:
        description = "Detects Sample Malware"
        author = "Analyst Name"
        date = "2024-01-15"
        hash = "abc123..."
        
    strings:
        $s1 = "C:\\Windows\\Temp\\malware.exe" ascii
        $s2 = "SystemUpdate" wide
        $s3 = { 55 8B EC 83 EC 20 }  // 함수 프롤로그
        $api1 = "CreateFileA" ascii
        $api2 = "CryptEncrypt" ascii
        
    condition:
        uint16(0) == 0x5A4D and  // MZ header
        filesize < 500KB and
        2 of ($s*) and
        all of ($api*)
}
```

**Snort 규칙 예시:**
```
alert tcp any any -> any 8080 (
    msg:"Malware C&C Communication Detected";
    flow:established,to_server;
    content:"POST";
    http_method;
    content:"/gate.php";
    http_uri;
    content:"infection_id=";
    http_client_body;
    classtype:trojan-activity;
    sid:1000001;
    rev:1;
)
```

**Sigma 규칙 예시:**
```yaml
title: Malware Registry Persistence
id: 12345678-1234-1234-1234-123456789012
description: Detects malware persistence via Run key
status: experimental
author: Analyst Name
date: 2024/01/15
logsource:
    product: windows
    service: sysmon
    definition: 'Sysmon Event ID 13'
detection:
    selection:
        EventID: 13
        TargetObject|contains: 
            - 'CurrentVersion\Run'
        Details|contains:
            - 'malware.exe'
    condition: selection
falsepositives:
    - Legitimate software updates
level: high
```

### 5.4 방어 방안 제안

**목적**: 조직이 취할 수 있는 대응 방안 제시

**체크리스트:**
```
[ ] 즉각 대응 조치
[ ] 시스템 강화 방안
[ ] 탐지 개선 방안
[ ] 사용자 교육 방안
```

**방어 방안 템플릿:**
```markdown
## 방어 방안

### 즉각 대응
1. IOC 기반 차단
   - IP 192.168.1.100 방화벽 차단
   - 도메인 malicious-domain.com DNS 싱크홀
   
2. 엔드포인트 검사
   - 레지스트리 키 확인 및 제거
   - 파일 시스템 검사
   - 프로세스 모니터링

### 시스템 강화
1. 패치 적용
   - MS17-010 (EternalBlue)
   - 최신 보안 업데이트
   
2. 설정 강화
   - SMBv1 비활성화
   - PowerShell 실행 정책 제한
   - 매크로 비활성화
   
3. 접근 제어
   - 최소 권한 원칙
   - 관리자 계정 분리
   - MFA 강제

### 탐지 개선
1. EDR 배포
   - 엔드포인트 행위 모니터링
   - 자동 대응 기능
   
2. SIEM 규칙 추가
   - IOC 기반 경보
   - 행위 기반 탐지
   
3. 네트워크 모니터링
   - IDS/IPS 규칙 업데이트
   - 네트워크 세그먼트화

### 사용자 교육
1. 피싱 인식 훈련
2. 첨부 파일 주의사항
3. 의심 활동 보고 절차
```

### 5.5 최종 보고서 작성

**목적**: 분석 결과를 종합하여 문서화

**체크리스트:**
```
[ ] 요약 (Executive Summary)
[ ] 상세 분석 내용
[ ] IOC 목록
[ ] 권장 사항
[ ] 첨부 자료 (스크린샷, 로그 등)
```

**보고서 구조:**
```markdown
# 악성코드 분석 보고서

## 1. 요약
- 샘플명: [이름]
- 유형: 랜섬웨어
- 위험도: 높음
- 분석 날짜: 2024-01-15
- 분석자: [이름]

## 2. 기본 정보
- 파일명: malware.exe
- 크기: 144 KB
- MD5: [해시]
- SHA256: [해시]

## 3. 상세 분석
### 3.1 정적 분석
- PE 구조
- Import 분석
- 문자열 분석

### 3.2 동적 분석
- 실행 행위
- 네트워크 통신
- 파일/레지스트리 변경

## 4. 주요 발견 사항
1. MBR 암호화 시도
2. Salsa20 암호화 사용
3. C&C 통신: 192.168.1.100

## 5. IOC
[IOC 목록]

## 6. 권장 사항
[방어 방안]

## 7. 참고 자료
- 스크린샷
- PCAP 파일
- 메모리 덤프
```

---

## 📝 분석 로그 작성

분석 중 발견한 내용은 [분석 로그 템플릿](../notes/analysis-log-template.md)을 사용하여 기록하세요.

---

## 🔗 관련 문서

- [리버싱 기본 개념](reversing-basics.md)
- [실습 환경 설정](setup-environment.md)
- [Petya 분석 가이드](petya-analysis-guide.md)
- [도구 목록](../resources/tools.md)
- [분석 로그 템플릿](../notes/analysis-log-template.md)

---

**💡 팁**: 이 체크리스트를 프린트하거나 별도 파일로 복사하여 실제 분석 시 항목을 체크하며 사용하세요!
