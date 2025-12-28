# 리버싱 도구 목록

## 📚 목차

- [디스어셈블러/디컴파일러](#-디스어셈블러디컴파일러)
- [디버거](#-디버거)
- [PE 분석 도구](#-pe-분석-도구)
- [네트워크 분석 도구](#-네트워크-분석-도구)
- [시스템 모니터링](#-시스템-모니터링)
- [문자열 및 바이너리 분석](#-문자열-및-바이너리-분석)
- [온라인 분석 서비스](#-온라인-분석-서비스)
- [기타 유틸리티](#-기타-유틸리티)

---

## 🔍 디스어셈블러/디컴파일러

### Ghidra

**설명**: NSA에서 개발한 오픈소스 리버스 엔지니어링 프레임워크

**특징**:
- 강력한 디컴파일러 내장
- 다양한 프로세서 아키텍처 지원
- Python 및 Java 스크립팅
- 무료 및 오픈소스

**다운로드**: https://ghidra-sre.org/

**라이선스**: Apache License 2.0

**필요 사항**: Java JDK 17+

---

### IDA Pro / IDA Free

**설명**: 업계 표준 대화형 디스어셈블러

**특징**:
- 직관적인 UI
- 풍부한 플러그인 생태계
- 크로스 플랫폼 지원
- 강력한 그래프 뷰

**다운로드**: https://hex-rays.com/ida-free/

**라이선스**:
- IDA Free: 무료 (64비트 PE 미지원)
- IDA Pro: 상용 (~$1,800+)
- Hex-Rays Decompiler: 별도 구매

**추천 플러그인**:
- Hex-Rays Decompiler (디컴파일러)
- IDA Python (스크립팅)
- BinDiff (바이너리 비교)

---

### Binary Ninja

**설명**: 현대적인 UI를 가진 리버스 엔지니어링 플랫폼

**특징**:
- 중간 언어(IL) 제공
- 빠른 분석 속도
- 강력한 API
- 활발한 개발

**다운로드**: https://binary.ninja/

**라이선스**:
- Personal: $299
- Commercial: $399/year
- Free: 개인 비상업용 (제한적)

**장점**:
- 합리적인 가격
- 우수한 디컴파일러
- 플러그인 아키텍처

---

### Radare2 / Cutter

**설명**: 오픈소스 리버스 엔지니어링 프레임워크

**특징**:
- 명령줄 기반 (radare2)
- GUI 버전 (Cutter)
- 스크립팅 지원
- 완전 무료

**다운로드**:
- Radare2: https://rada.re/
- Cutter: https://cutter.re/

**라이선스**: LGPL v3

**학습 곡선**: 높음 (명령줄 복잡)

---

### Hopper Disassembler

**설명**: macOS/Linux용 디스어셈블러

**특징**:
- 간단한 UI
- 디컴파일러 내장
- ARM 지원 우수

**다운로드**: https://www.hopperapp.com/

**라이선스**: 상용 (~$99)

**플랫폼**: macOS, Linux

---

## 🐛 디버거

### x64dbg

**설명**: Windows용 오픈소스 x86/x64 디버거

**특징**:
- OllyDbg의 현대적 대체
- 32/64비트 지원
- 플러그인 생태계
- 활발한 개발

**다운로드**: https://x64dbg.com/

**라이선스**: GPL v3

**추천 플러그인**:
- ScyllaHide (안티 디버깅 우회)
- xAnalyzer (정적 분석)
- ret-sync (IDA/Ghidra 동기화)

**단축키**:
- F2: Breakpoint
- F7: Step Into
- F8: Step Over
- F9: Run

---

### OllyDbg

**설명**: 클래식 Windows 디버거

**특징**:
- 간단한 UI
- 32비트 전용
- 플러그인 지원

**다운로드**: http://www.ollydbg.de/

**라이선스**: 무료

**상태**: 더 이상 개발되지 않음 (x64dbg 사용 권장)

---

### WinDbg / WinDbg Preview

**설명**: Microsoft 공식 디버거

**특징**:
- 커널 모드 디버깅
- 크래시 덤프 분석
- 강력하지만 복잡함

**다운로드**:
- WinDbg: Windows SDK에 포함
- Preview: Microsoft Store

**라이선스**: 무료

**사용 사례**:
- 드라이버 분석
- BSOD 분석
- 시스템 레벨 디버깅

---

### GDB (GNU Debugger)

**설명**: Linux/Unix용 표준 디버거

**특징**:
- 명령줄 기반
- 강력한 스크립팅
- 원격 디버깅

**다운로드**: 대부분의 Linux 배포판에 포함

**라이선스**: GPL

**프론트엔드**:
- GEF (GDB Enhanced Features)
- PEDA (Python Exploit Development Assistance)
- pwndbg

---

### EDB (Evan's Debugger)

**설명**: Linux용 그래픽 디버거

**특징**:
- OllyDbg와 유사한 UI
- 플러그인 지원
- 리눅스 네이티브

**다운로드**: https://github.com/eteran/edb-debugger

**라이선스**: GPL v2

---

## 📦 PE 분석 도구

### PEStudio

**설명**: PE 파일 정적 분석 도구

**특징**:
- 자동 위협 평가
- Import/Export 분석
- 의심 지표 강조
- 무료

**다운로드**: https://www.winitor.com/

**라이선스**: 프리웨어

**강점**:
- 초보자 친화적
- 자동 분석
- 빠른 스캔

---

### PE-bear

**설명**: PE 구조 시각화 및 편집 도구

**특징**:
- PE 구조 트리 뷰
- 섹션 편집
- Import 재구성

**다운로드**: https://github.com/hasherezade/pe-bear

**라이선스**: 무료 (오픈소스)

**개발자**: hasherezade (멀웨어 연구자)

---

### CFF Explorer

**설명**: PE 편집기 (Explorer Suite 일부)

**특징**:
- 종합 PE 편집
- Import/Export 편집
- 리소스 편집
- 프로세스 뷰어

**다운로드**: https://ntcore.com/?page_id=388

**라이선스**: 프리웨어

---

### Detect It Easy (DIE)

**설명**: 패커 및 컴파일러 탐지 도구

**특징**:
- 패커/프로텍터 식별
- 컴파일러 탐지
- 엔트로피 계산
- 시그니처 데이터베이스

**다운로드**: https://github.com/horsicq/Detect-It-Easy

**라이선스**: MIT

**버전**:
- die.exe (GUI)
- diec.exe (콘솔)

---

### PEiD

**설명**: 레거시 패커 탐지 도구

**특징**:
- 방대한 시그니처 DB
- 플러그인 지원

**다운로드**: https://www.aldeid.com/wiki/PEiD

**상태**: 더 이상 개발되지 않음 (DIE 사용 권장)

---

### PE Tools

**설명**: PE 분석 명령줄 도구

**다운로드**: https://github.com/petoolse/petools

**라이선스**: MIT

---

## 🌐 네트워크 분석 도구

### Wireshark

**설명**: 네트워크 프로토콜 분석기

**특징**:
- 패킷 캡처 및 분석
- 700+ 프로토콜 지원
- 강력한 필터링
- 크로스 플랫폼

**다운로드**: https://www.wireshark.org/

**라이선스**: GPL v2

**필수 기술**:
- 디스플레이 필터
- 캡처 필터
- 프로토콜 디코딩

---

### Fiddler

**설명**: HTTP/HTTPS 디버깅 프록시

**특징**:
- 웹 트래픽 캡처
- SSL/TLS 복호화
- 요청/응답 수정
- 스크립팅

**다운로드**: https://www.telerik.com/fiddler

**라이선스**: 무료

**플랫폼**: Windows, macOS, Linux

---

### Burp Suite

**설명**: 웹 애플리케이션 보안 테스트 도구

**특징**:
- HTTP 프록시
- Scanner (Pro)
- Intruder
- Repeater

**다운로드**: https://portswigger.net/burp

**라이선스**:
- Community: 무료 (제한적)
- Professional: 상용

---

### tcpdump

**설명**: 명령줄 패킷 분석기

**특징**:
- 경량
- 스크립팅 친화적
- Unix/Linux 표준

**다운로드**: 대부분의 시스템에 기본 포함

**라이선스**: BSD

**Windows 버전**: WinDump

---

### NetworkMiner

**설명**: 네트워크 포렌식 도구

**특징**:
- 패킷에서 파일 추출
- 호스트 정보 수집
- 이미지/영상 재구성

**다운로드**: https://www.netresec.com/?page=NetworkMiner

**라이선스**:
- Free: 무료 (제한적)
- Professional: 상용

---

## 📊 시스템 모니터링

### Process Monitor (ProcMon)

**설명**: 실시간 파일/레지스트리/프로세스 모니터

**특징**:
- 시스템 활동 캡처
- 강력한 필터링
- 이벤트 스택 추적
- Sysinternals Suite 일부

**다운로드**: https://docs.microsoft.com/sysinternals/downloads/procmon

**라이선스**: 무료

**개발자**: Microsoft (Mark Russinovich)

---

### Process Hacker

**설명**: 고급 프로세스 뷰어

**특징**:
- 프로세스 트리
- 메모리 검색/덤프
- 네트워크 연결
- 핸들/DLL 조회

**다운로드**: https://processhacker.sourceforge.io/

**라이선스**: GPL v3

**장점**:
- Task Manager보다 강력
- 플러그인 지원
- 오픈소스

---

### Process Explorer

**설명**: 고급 프로세스 관리 도구

**특징**:
- 프로세스 계층 구조
- DLL 및 핸들 뷰
- 시스템 정보
- Sysinternals Suite

**다운로드**: https://docs.microsoft.com/sysinternals/downloads/process-explorer

**라이선스**: 무료

---

### Autoruns

**설명**: 자동 시작 프로그램 관리

**특징**:
- 시작 프로그램 목록
- 레지스트리 Run 키
- 예약된 작업
- 서비스

**다운로드**: https://docs.microsoft.com/sysinternals/downloads/autoruns

**라이선스**: 무료

**용도**: 지속성 메커니즘 탐지

---

### TCPView

**설명**: 네트워크 연결 모니터

**특징**:
- 실시간 연결 표시
- 프로세스별 연결
- 연결 종료 기능

**다운로드**: https://docs.microsoft.com/sysinternals/downloads/tcpview

**라이선스**: 무료

---

### Regshot

**설명**: 레지스트리 비교 도구

**특징**:
- 스냅샷 생성
- 변경 사항 비교
- HTML 리포트

**다운로드**: https://sourceforge.net/projects/regshot/

**라이선스**: GPL

**사용법**:
1. 1st shot (실행 전)
2. 샘플 실행
3. 2nd shot (실행 후)
4. Compare

---

## 📝 문자열 및 바이너리 분석

### Strings (Sysinternals)

**설명**: 바이너리에서 문자열 추출

**특징**:
- ASCII/Unicode 지원
- 최소 길이 설정
- 오프셋 표시

**다운로드**: https://docs.microsoft.com/sysinternals/downloads/strings

**라이선스**: 무료

**사용법**:
```cmd
strings.exe -n 4 sample.exe > output.txt
strings.exe -u sample.exe > unicode.txt
```

---

### HxD

**설명**: Hex 편집기

**특징**:
- 빠른 성능
- 대용량 파일 지원
- 파일 비교
- 체크섬 계산

**다운로드**: https://mh-nexus.de/en/hxd/

**라이선스**: 프리웨어

---

### 010 Editor

**설명**: 전문 hex 편집기

**특징**:
- 바이너리 템플릿
- 스크립팅
- 파일 구조 파싱

**다운로드**: https://www.sweetscape.com/010editor/

**라이선스**: 상용 (~$49)

---

### binwalk

**설명**: 펌웨어 분석 도구

**특징**:
- 파일 시그니처 검색
- 파일 추출
- 엔트로피 분석

**다운로드**: https://github.com/ReFirmLabs/binwalk

**라이선스**: MIT

**설치**:
```bash
# Linux
sudo apt install binwalk

# Python
pip install binwalk
```

---

### ExeinfoPE

**설명**: PE 정보 및 패커 탐지

**특징**:
- 컴파일러 탐지
- 패커 식별
- 플러그인 지원

**다운로드**: http://www.exeinfo.pe.hu/

**라이선스**: 프리웨어

---

## 🌐 온라인 분석 서비스

### VirusTotal

**설명**: 다중 안티바이러스 스캔 서비스

**특징**:
- 70+ AV 엔진
- 행위 분석
- 네트워크 관계
- YARA 스캔

**웹사이트**: https://www.virustotal.com/

**API**: 무료 (제한적)

**⚠️ 주의**: 업로드한 샘플은 공개됩니다!

---

### Hybrid Analysis

**설명**: 무료 자동 악성코드 분석

**특징**:
- 샌드박스 실행
- 상세 보고서
- IOC 추출
- MITRE ATT&CK 매핑

**웹사이트**: https://www.hybrid-analysis.com/

**API**: 무료

---

### Any.run

**설명**: 대화형 온라인 샌드박스

**특징**:
- 실시간 인터랙션
- 네트워크 트래픽
- 프로세스 모니터링
- 스크린샷

**웹사이트**: https://any.run/

**라이선스**:
- 무료: 제한적 (공개 제출)
- 유료: 비공개 분석

---

### Joe Sandbox

**설명**: 고급 악성코드 분석 시스템

**특징**:
- 심층 행위 분석
- 안티 회피 기술
- 상세 보고서

**웹사이트**: https://www.joesandbox.com/

**라이선스**: 상용 (무료 티어 있음)

---

### Malware Bazaar

**설명**: 악성코드 샘플 공유 플랫폼

**특징**:
- 샘플 다운로드
- 태그 및 분류
- API 접근

**웹사이트**: https://bazaar.abuse.ch/

**운영**: abuse.ch

---

### URLhaus

**설명**: 악성 URL 데이터베이스

**특징**:
- 악성 URL 추적
- 페이로드 다운로드
- 블랙리스트

**웹사이트**: https://urlhaus.abuse.ch/

---

## 🔧 기타 유틸리티

### YARA

**설명**: 악성코드 패턴 매칭 도구

**특징**:
- 규칙 기반 탐지
- 강력한 패턴 언어
- 스크립팅 통합

**다운로드**: https://virustotal.github.io/yara/

**라이선스**: BSD

**사용법**:
```bash
yara rule.yar sample.exe
yara -r rules/ directory/
```

---

### Capa

**설명**: 악성코드 능력 탐지 도구

**특징**:
- 행위 기반 탐지
- MITRE ATT&CK 매핑
- 규칙 기반

**다운로드**: https://github.com/mandiant/capa

**라이선스**: Apache 2.0

**개발자**: Mandiant (FireEye)

---

### PEfile (Python)

**설명**: Python PE 파일 파싱 라이브러리

**특징**:
- PE 구조 파싱
- Import/Export 분석
- 스크립팅 친화적

**설치**:
```bash
pip install pefile
```

**라이선스**: MIT

---

### Volatility

**설명**: 메모리 포렌식 프레임워크

**특징**:
- 메모리 덤프 분석
- 프로세스/네트워크 추출
- 플러그인 아키텍처

**다운로드**: https://www.volatilityfoundation.org/

**라이선스**: GPL

**버전**:
- Volatility 2: Python 2
- Volatility 3: Python 3

---

### Scylla

**설명**: Import 재구성 도구 (언패킹)

**특징**:
- IAT 재구성
- OEP 찾기
- 덤프 복구

**다운로드**: https://github.com/NtQuery/Scylla

**라이선스**: GPL v3

**통합**: x64dbg 플러그인으로 사용 가능

---

### UnpackMe / Unipacker

**설명**: 자동 언패킹 도구

**특징**:
- 다양한 패커 지원
- 자동 덤프

**다운로드**: https://github.com/unipacker/unipacker

**라이선스**: GPL

---

### ILSpy

**설명**: .NET 디컴파일러

**특징**:
- C# 코드 생성
- IL 뷰
- 어셈블리 탐색

**다운로드**: https://github.com/icsharpcode/ILSpy

**라이선스**: MIT

**용도**: .NET 악성코드 분석

---

### dnSpy

**설명**: .NET 디버거 및 에디터

**특징**:
- 디컴파일 및 디버깅
- 코드 편집
- 패치

**다운로드**: https://github.com/dnSpy/dnSpy

**라이선스**: GPL v3

**상태**: 아카이브됨 (유지보수 중단)

---

### JD-GUI

**설명**: Java 디컴파일러

**특징**:
- JAR/CLASS 파일 분석
- GUI 인터페이스

**다운로드**: https://java-decompiler.github.io/

**라이선스**: GPL v3

---

## 📦 도구 설치 팁

### Windows 패키지 관리자

**Chocolatey 사용:**
```powershell
# Chocolatey 설치
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

# 도구 설치
choco install sysinternals
choco install wireshark
choco install python
```

### Linux 패키지 관리자

**APT (Debian/Ubuntu):**
```bash
sudo apt update
sudo apt install gdb radare2 binwalk
```

**Kali Linux (대부분 사전 설치):**
```bash
sudo apt install kali-tools-reverse-engineering
```

---

## 🎓 학습 리소스

### 도구 튜토리얼

- **Ghidra**: NSA 공식 문서, YouTube 강의
- **x64dbg**: 공식 문서, 커뮤니티 가이드
- **IDA Pro**: Hex-Rays 튜토리얼

### 실습 환경

- **Flare-VM**: 악성코드 분석 VM (Windows)
- **REMnux**: 리버싱 Linux 배포판

다운로드:
- Flare-VM: https://github.com/mandiant/flare-vm
- REMnux: https://remnux.org/

---

## 🔗 관련 문서

- [리버싱 기본 개념](../docs/reversing-basics.md)
- [실습 환경 설정](../docs/setup-environment.md)
- [참고 자료](references.md)

---

**💡 팁**: 모든 도구를 설치할 필요는 없습니다. 자신에게 맞는 도구를 선택하여 숙련도를 높이세요!
