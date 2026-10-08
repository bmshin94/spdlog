# spdlog 전수조사 분석 & 활용 전략 정리

> 이 문서는 [bmshin94/spdlog](https://github.com/bmshin94/spdlog) 저장소를 전수조사한 결과와,
> 그에 기반한 활용·수익화 전략을 정리한 한국어 가이드입니다.

| 항목 | 내용 |
|---|---|
| **현재 저장소** | https://github.com/bmshin94/spdlog |
| **원본(Upstream)** | https://github.com/gabime/spdlog |
| **공식 문서(Wiki)** | https://github.com/gabime/spdlog/wiki |
| **내장 의존성** | https://github.com/fmtlib/fmt (fmt 라이브러리 사본 포함) |
| **버전** | v1.17.0 |
| **라이선스** | MIT |
| **작성일** | 2026-10-08 |

---

## 목차

1. [정체 — 이게 뭐하는 건가](#1-정체--이게-뭐하는-건가)
2. [폴더 전수조사](#2-폴더-전수조사)
3. [핵심 기능 해부](#3-핵심-기능-해부)
4. [쉬운 설명 — 비유로 이해하기](#4-쉬운-설명--비유로-이해하기)
5. [설치 및 사용법](#5-설치-및-사용법)
6. [자주 묻는 질문 7가지](#6-자주-묻는-질문-7가지)
7. [수익화 전략](#7-수익화-전략)
8. [실행 로드맵](#8-실행-로드맵)
9. [라이선스 주의사항](#9-라이선스-주의사항)

---

## 1. 정체 — 이게 뭐하는 건가

### 한 줄 결론

**spdlog은 C++로 작성된 "초고속 로깅 라이브러리"입니다.**

AI 도구도, 플러그인도, 실행 프로그램도 아닙니다.
**다른 C++ 프로그램 안에 넣어서, 그 프로그램이 실행 기록(로그)을 남기게 해주는 코드 부품**입니다.

- 원본 프로젝트는 GitHub ⭐ 2.8만개 이상 — C++ 로깅 분야 사실상 표준
- 현재 저장소는 원본을 복사(fork)해 온 것
- MIT 라이선스 → 상업적 이용·수정·재배포 전부 자유 (저작권 표시만 유지)

### 코드 규모

| 구분 | 줄 수 |
|---|---|
| spdlog 자체 헤더 (bundled 제외) | 11,155줄 |
| 내장 fmt 라이브러리 | 16,228줄 |
| 컴파일 소스 (`src/`) | 207줄 |
| 테스트 (`tests/`) | 3,717줄 |
| Sink 헤더 | 28종 |

### ⚠️ `CLAUDE.md`에 대한 주의

저장소 루트의 `CLAUDE.md`에는 "초광속 시스템 비행 기록 장치", "지연시간제로로거" 같은
홍보성 문구가 있습니다. 이 파일은 **원본 spdlog에 없던 파일**이며, fork 후에 추가된
자동 생성 요약문입니다.

```
확인된 커밋: e6e591e "Merge PR #1: docs: add CLAUDE.md project guide"
             → CLAUDE.md 16줄 추가 (원본 gabime/spdlog에는 존재하지 않음)
```

틀린 말은 아니지만 과장이 섞여 있으므로, 실제 기능은 이 문서를 기준으로 파악하는 것이 정확합니다.

---

## 2. 폴더 전수조사

```
spdlog/
├── include/spdlog/     ← 라이브러리 본체 (108개 파일, 핵심)
│   ├── spdlog.h             전역 API 진입점 (spdlog::info(...) 등)
│   ├── logger.h             Logger 클래스 — 로그 한 줄의 출발점
│   ├── async_logger.h       비동기 Logger (성능의 핵심)
│   ├── common.h             로그 레벨 7단계 정의
│   ├── pattern_formatter.h  로그 출력 모양 결정 (%Y-%m-%d 등)
│   ├── tweakme.h            컴파일 시점 성능 튜닝 스위치 모음
│   ├── stopwatch.h          실행시간 측정 유틸
│   ├── mdc.h                스레드별 진단 정보 저장
│   ├── sinks/   (28개)      "로그를 어디에 쓸지" 모음
│   ├── details/ (24개)      내부 엔진 (큐, 스레드풀, OS 추상화)
│   ├── cfg/                 환경변수·명령행으로 로그레벨 제어
│   └── fmt/bundled/         fmt 라이브러리 내장 사본 (16,228줄)
├── src/                (7개 파일, 207줄)  컴파일 라이브러리용 진입점
├── example/            사용 예제 25종 (example.cpp, 14KB)
├── bench/              성능 벤치마크 4종 (처리량·지연시간 측정)
├── tests/              단위 테스트 33개 파일, 3,717줄
├── cmake/              빌드 스크립트 (CMake 패키지 설정)
├── scripts/            CI 보조 스크립트
├── .github/workflows/  Linux/Windows/macOS 자동 빌드 검증
├── CMakeLists.txt       빌드 설정 본문 (20KB, 옵션 30여 개)
├── README.md            공식 문서 (19KB)
└── CLAUDE.md            ⚠️ fork 후 추가된 홍보성 요약 (원본에 없음)
```

### 폴더별 쉬운 설명

| 폴더 | 쉬운 설명 |
|---|---|
| `include/spdlog/` | **상품 본체.** 이것만 복사해 넣으면 바로 사용 가능 (헤더온리) |
| `include/.../sinks/` | **"배송기사" 28명 명부.** 필요한 것만 골라서 사용 |
| `include/.../details/` | **엔진룸.** 직접 손댈 필요 없음. 큐·스레드풀·OS 호환 처리 |
| `include/.../fmt/bundled/` | **동봉 부품(fmt).** 따로 설치 안 해도 되게 포함해둔 것 |
| `src/` | **"미리 조립된 버전"용 공장.** 컴파일 시간 단축용 |
| `example/` | **사용 설명서 25개 예제.** 실무에선 여기 복붙부터 시작 |
| `tests/` | **품질검사 3,717줄.** 33개 항목을 매 수정마다 자동 검사 |
| `bench/` | **성능 측정기.** "느려졌나?" 자동 감시 |
| `cmake/`, `CMakeLists.txt` | **조립 설명서.** OS·컴파일러 상관없이 알아서 맞춰 빌드 |
| `.github/workflows/` | **자동 품질 관리원.** 코드 올릴 때마다 자동 검증 |

### CI 검증 매트릭스 (`.github/workflows/linux.yml`)

품질 수준을 보여주는 지표입니다:

```
gcc 9  (C++11, Release)
gcc 11 (C++17, Debug)
gcc 12 (C++20, Release)
gcc 12 (C++20, Debug + ASAN)   ← 메모리 오류 검사
clang 12 (C++17, Debug)
clang 15 (C++20, Release + TSAN) ← 스레드 경쟁 검사
```

여기에 Windows / macOS 워크플로가 추가로 존재합니다.
이것이 "믿을 수 있는 오픈소스"와 "취미 프로젝트"를 가르는 결정적 차이입니다.

---

## 3. 핵심 기능 해부

### 3.1 로그 레벨 7단계 (`include/spdlog/common.h:247`)

```cpp
trace → debug → info → warn → err → critical → off
```

심각도 순서입니다. `set_level(info)`로 두면 trace/debug는 아예 출력되지 않습니다.
**실행 중에도 바꿀 수 있고, 컴파일 시점에 아예 코드에서 제거**할 수도 있습니다 (후자가 성능상 핵심).

### 3.2 Sink — "로그를 어디에 쓸 것인가" 28종

`include/spdlog/sinks/` 전체 목록:

| 분류 | Sink | 용도 |
|---|---|---|
| **파일** | `basic_file_sink` | 파일 하나에 계속 기록 |
| | `rotating_file_sink` | 지정 용량 차면 새 파일로 교체, 오래된 건 삭제 |
| | `daily_file_sink` | 매일 지정 시각에 새 파일 생성 |
| | `hourly_file_sink` | 매 시간마다 새 파일 |
| **콘솔** | `stdout_sinks` / `stderr` | 터미널 출력 |
| | `ansicolor_sink` | Linux/macOS 컬러 출력 |
| | `wincolor_sink` | Windows 컬러 출력 |
| **OS 연동** | `syslog_sink` | Linux 시스템 로그 |
| | `systemd_sink` | systemd journal |
| | `win_eventlog_sink` | Windows 이벤트 뷰어 |
| | `msvc_sink` | Visual Studio 디버그 창 |
| | `android_sink` | Android logcat |
| **네트워크** | `tcp_sink` / `udp_sink` | 원격 서버로 로그 전송 |
| | `kafka_sink` | Kafka 메시지 큐로 |
| | `mongo_sink` | MongoDB에 직접 |
| | `loki_sink` | Grafana Loki로 |
| **특수** | `ringbuffer_sink` | 메모리에만 최근 N개 보관 |
| | `callback_sink` | 로그 발생 시 내 함수 호출 (Slack 알림 등) |
| | `dist_sink` | 여러 sink로 동시 분배 |
| | `dup_filter_sink` | 똑같은 로그 반복 시 묶어서 표시 |
| | `qt_sinks` | Qt GUI 창에 로그 표시 |
| | `null_sink` | 버리기 (성능 측정용) |

**한 Logger에 여러 Sink를 동시에 붙일 수 있고, Sink마다 레벨과 포맷을 다르게 줄 수 있습니다.**

### 3.3 비동기 모드 — 속도의 비밀

```
[내 코드] --log()--> [락프리 큐] --> [전용 백그라운드 스레드] --> [파일/네트워크]
   ↑                                       ↑
   즉시 반환 (수백 ns)              느린 디스크 I/O는 여기서 처리
```

관련 파일: `details/thread_pool.h`, `details/mpmc_blocking_q.h`

큐가 꽉 찼을 때의 정책 3가지 (`async_logger.h:22`):

| 정책 | 동작 | 특성 |
|---|---|---|
| `block` | 자리 날 때까지 대기 | 로그 손실 0, 느림 |
| `overrun_oldest` | 가장 오래된 로그 버림 | 빠름 |
| `discard_new` | 새 로그 버림 | 빠름 |

### 3.4 성능 실측치 (README 기준, i7-4770, 100만 건)

| 모드 | 처리량 |
|---|---|
| 동기 단일스레드 파일 (`basic_st`) | 초당 **577만 건** |
| 동기 10스레드 경쟁 (`basic_mt`) | 초당 **166만 건** |
| 비동기 (block 정책) | 초당 **58만 건** |
| 비동기 (overrun 정책) | 초당 **268만 건** |

체감 환산: 게임 서버 1프레임(16ms) 안에 로그 4만 건을 남겨도 프레임이 깨지지 않습니다.
즉 **로깅 때문에 느려질 걱정을 안 해도 되는 수준**입니다.

### 3.5 포맷팅 — printf 대신 Python 스타일

내장된 fmt 라이브러리(Python `.format()` 문법)를 사용합니다:

```cpp
spdlog::info("숫자 패딩 {:08d}", 12);            // 00000012
spdlog::critical("10진 {0:d}, 16진 {0:x}", 42);  // 42, 2a
spdlog::info("왼쪽정렬 {:<30}", "text");
spdlog::info("위치 인수 {1} {0}", "too", "supported");
```

**타입을 컴파일 시점에 검사**합니다. `printf("%d", "문자열")` 같은 실수가 애초에 컴파일되지 않습니다.

### 3.6 출력 패턴 플래그 (`pattern_formatter-inl.h` 전수조사)

```cpp
spdlog::set_pattern("[%Y-%m-%d %H:%M:%S.%e] [%n] [%^%l%$] [%t] %v");
```

| 플래그 | 의미 | 플래그 | 의미 |
|---|---|---|---|
| `%v` | 로그 본문 | `%Y %C` | 연(4자리/2자리) |
| `%n` | Logger 이름 | `%m %d` | 월, 일 |
| `%l` / `%L` | 레벨 / 레벨 약어 | `%H %M %S` | 시(24h), 분, 초 |
| `%t` | 스레드 ID | `%I %p` | 시(12h), AM/PM |
| `%P` | 프로세스 ID | `%e %f %F` | 밀리초, 마이크로초, 나노초 |
| `%@` | `파일명:줄번호` | `%E` | epoch 초 |
| `%s` / `%g` | 짧은/전체 파일명 | `%z` | 타임존 오프셋 |
| `%#` | 줄번호 | `%a %A` | 요일(약어/전체) |
| `%!` | 함수명 | `%b %B` | 월 이름(약어/전체) |
| `%^` `%$` | 컬러 시작 / 끝 | `%c %x %X %D %r %R %T` | 날짜·시각 조합 포맷 |
| `%&` | MDC 진단정보 | `%u %i %o %O` | 이전 로그와의 경과시간(ns/us/ms/s) |
| `%%` | `%` 문자 | `%*` | 사용자 정의 플래그 |
| `%+` | 기본 포맷 | `%-` `%=` | 정렬 지정 |

### 3.7 Backtrace — 가장 실용적인 기능

평소엔 상세 로그를 메모리 링버퍼에만 돌려 녹화하고, 사고 발생 시에만 쏟아냅니다.

```cpp
spdlog::enable_backtrace(32);   // 최근 32개만 메모리에 돌려 녹화

spdlog::debug("DB 커넥션 획득");      // 출력 안 됨, 녹화만 (= 거의 공짜)
spdlog::debug("쿼리 실행 SELECT...");  // 녹화만
spdlog::debug("응답 대기 중");         // 녹화만

if (error) {
    spdlog::dump_backtrace();   // 💥 직전 32단계 전부 출력
}
```

**CCTV 비유**: 24시간 전부 저장하진 않지만, 사고 직전 1분은 항상 가지고 있는 구조입니다.
→ 평소 성능 손실 거의 0, 사고 순간엔 상세 정황 전부 확보.

### 3.8 환경변수로 재컴파일 없이 레벨 변경 (`cfg/env.h`, `cfg/argv.h`)

```bash
export SPDLOG_LEVEL=warn              # 평소
export SPDLOG_LEVEL=info,mylogger=trace  # 특정 모듈만 상세
./myapp

# 또는 명령행 인수로
./myapp SPDLOG_LEVEL=debug
```

### 3.9 기타 유틸

| 기능 | 파일 | 설명 |
|---|---|---|
| **Stopwatch** | `stopwatch.h` | 구간 실행시간 측정 → `spdlog::info("{:.3}", sw)` |
| **MDC** | `mdc.h` | 스레드별 key-value 진단정보 (`%&`로 출력). 비동기 모드 미지원 |
| **bin_to_hex** | `fmt/bin_to_hex.h` | 바이너리 데이터를 hex dump로 출력 |
| **파일 이벤트 핸들러** | `common.h` | 로그 파일 열기/닫기 전후 콜백 |
| **커스텀 에러 핸들러** | `spdlog.h` | 로깅 자체가 실패할 때의 처리 지정 |
| **주기적 flush** | `spdlog.h` | `flush_every(3s)` — N초마다 디스크 반영 |

### 3.10 컴파일 시점 성능 튜닝 (`tweakme.h`)

```cpp
#define SPDLOG_CLOCK_COARSE       // 시계 정확도↓ 속도↑ (Linux)
#define SPDLOG_NO_SOURCE_LOC      // 파일명/줄번호 안 씀 → 더 빠름
#define SPDLOG_NO_THREAD_ID       // 스레드ID 조회 생략 → 더 빠름
#define SPDLOG_NO_ATOMIC_LEVELS   // 레벨 원자적 접근 생략 → 더 빠름
#define SPDLOG_NO_TLS             // 스레드 로컬 저장소 미사용 (fork 안전)
#define SPDLOG_PREVENT_CHILD_FD   // 자식 프로세스가 로그 fd 상속 방지
#define SPDLOG_WCHAR_FILENAMES    // Windows 유니코드 파일명
#define SPDLOG_LEVEL_NAMES {...}  // 레벨 이름 커스터마이즈
#define SPDLOG_ACTIVE_LEVEL SPDLOG_LEVEL_INFO  // debug 호출을 컴파일에서 삭제
```

마지막 것이 가장 강력합니다 — `SPDLOG_DEBUG(...)` 호출이 **바이너리에서 완전히 사라집니다**
(런타임 비용이 진짜 0).

---

## 4. 쉬운 설명 — 비유로 이해하기

### 4.1 "로그"가 뭔지부터

로그 = **프로그램이 쓰는 일기장**입니다.

```
[09:31:02] 서버 시작됨
[09:31:05] 사용자 'kim' 로그인 성공
[09:31:07] 결제 요청 금액=50000
[09:31:07] ⚠️ 결제 서버 응답 없음 (타임아웃 3초)
[09:31:10] ❌ 결제 실패, 사용자에게 오류 반환
```

프로그램은 눈에 보이지 않습니다. 사용자가 "결제가 안 돼요"라고 할 때,
개발자가 그 순간 서버 안에서 뭐가 일어났는지 알 수 있는 **유일한 단서**가 로그입니다.
비행기 블랙박스와 같습니다 — 사고 나기 전엔 아무도 안 보지만, 없으면 원인을 영원히 알 수 없습니다.

### 4.2 "printf로 하면 안 되나?" — 4가지 지옥

#### 지옥 1 — 너무 느림

```
printf 방식:  내 코드 ─[printf]─→ 💤 디스크에 다 쓸 때까지 대기 ─→ 다음 줄
spdlog 비동기: 내 코드 ─[log()]─→ 대기줄에 쪽지 던지고 ─→ 즉시 다음 줄
                                      ↓
                                전담 스레드가 알아서 디스크에 씀
```

**카페 비유**: 손님(내 코드)이 주문만 하고 자리로 돌아갑니다.
바리스타(백그라운드 스레드)가 커피를 만들어 가져다줍니다.
손님이 커피 만드는 걸 서서 기다리지 않습니다. → **약 200배 빠름**

#### 지옥 2 — 글자가 뒤섞임

멀티스레드에서 두 스레드가 동시에 로그를 쓰면:

```
기대:  사용자 kim 로그인
       사용자 lee 결제완료

현실:  사용자 k사용자 lee 결im 로그인
       제완료              ← 💀 읽을 수 없는 쓰레기
```

**spdlog**: `_mt`(multi-thread) Logger는 내부에서 순서를 정리해 **절대 섞이지 않게 보장**.
혼자 쓰는 게 확실하면 `_st`(single-thread)로 그 보호장치를 떼고 더 빠르게. → **선택권을 줍니다.**

#### 지옥 3 — 하드디스크가 꽉 참

로그 파일 하나에 계속 쓰면 1주일 뒤 50GB가 되고 서버가 멈춥니다.

```
rotating_file_sink:  5MB 차면 → log.1.txt로 밀고 새로 시작, 3개만 유지
                     (= 항상 최대 15MB로 고정, 오래된 건 자동 삭제)

daily_file_sink:     log-2026-10-08.txt
                     log-2026-10-09.txt   ← 매일 새 파일
```

**휴지통 비유**: 꽉 차면 알아서 가장 오래된 걸 버리고 새 걸 넣습니다.

#### 지옥 4 — 켜고 끌 수가 없음

개발 중엔 상세 로그가 필요하지만 서비스에선 쓰레기 + 성능 낭비입니다.

```
trace ─ debug ─ info ─ warn ─ error ─ critical ─ off
(수다)  (상세)  (보통) (경고)  (오류)   (대참사)   (침묵)
  ↑                      ↑
개발 중엔 여기       서비스는 여기
```

다이얼을 `warn`에 두면 trace/debug/info는 **코드에 그대로 있으면서 출력만 안 됩니다.**
문제 생기면 환경변수만 바꾸면 전부 다시 보입니다. **재컴파일 불필요.**

### 4.3 spdlog의 3단 구조 — 이것만 이해하면 끝

로그 한 줄을 **공장 라인**처럼 처리합니다.

```
   ①            ②              ③
 Logger  →  Formatter  →   Sink
(접수원)     (포장담당)     (배송기사)
```

#### ① Logger — "접수원"

내가 직접 말을 거는 창구입니다.

```cpp
logger->info("사용자 {} 로그인", "kim");
```

접수원의 역할: **"이거 지금 접수할 가치가 있나?"** 판단.
레벨이 `warn`인데 `info` 요청이 오면 → 그 자리에서 버립니다.
(이게 성능의 핵심 — 버릴 로그는 포장도 배송도 하지 않음)

#### ② Formatter — "포장담당"

```
입력:  "사용자 kim 로그인"
         ↓ 패턴 적용: "[%H:%M:%S] [%n] [%l] %v"
출력:  "[09:31:05] [auth] [info] 사용자 kim 로그인"
```

패턴 문자 40여 개를 레고처럼 조립해 **원하는 모양을 직접 설계**합니다.

#### ③ Sink — "배송기사" (28종)

완성된 로그를 목적지로 운반합니다. 결정적으로 — **기사를 여러 명 동시에 고용 가능:**

```cpp
Logger 하나
   ├─→ 콘솔 기사    (레벨: warn 이상만, 빨간색으로)
   ├─→ 파일 기사    (레벨: 전부, 날짜 포함 상세 포맷)
   ├─→ Slack 기사   (레벨: critical만, 즉시 알림)
   └─→ Kafka 기사   (레벨: 전부, 중앙 수집 서버로)
```

한 번의 `logger->critical("DB 폭발")` 호출로 → 터미널에 빨갛게 뜨고 + 파일에 기록되고
+ 폰에 Slack 알림 오고 + 중앙 모니터링에 집계됩니다. **코드는 한 줄.**

**이 3분리가 왜 중요한가**: 나중에 "로그를 MongoDB에도 저장해야 해"라는 요구가 오면,
**기존 코드를 한 줄도 고치지 않고** 시작할 때 Sink 하나만 추가하면 끝입니다.

### 4.4 어떨 때 쓰는가

| 상황 | spdlog가 하는 일 |
|---|---|
| **게임 서버** | 수천 명 접속 중 "누가 언제 어떤 행동을" — 서버 프레임 안 깨고 기록 |
| **금융/HFT** | 마이크로초 지연도 안 되는 주문 체결 기록 → 비동기 모드 |
| **임베디드/IoT** | 로그가 디스크 꽉 채우면 안 됨 → rotating sink로 자동 용량 제한 |
| **장시간 서비스** | 날짜별 로그 분리 → daily sink, 과거 추적 용이 |
| **운영 중 장애 분석** | backtrace로 사고 직전 상황 복구 |
| **분산 시스템** | kafka/loki/tcp sink로 중앙 로그 수집 |
| **GUI 앱** | qt_sink로 앱 화면 안에 로그 콘솔 |
| **크로스플랫폼 제품** | 하나의 코드로 Linux/Win/macOS/Android 전부 |

### 4.5 어떤 도움이 되는가 — 3가지 트랙

#### 트랙 A — C++ 개발자라면 (직접적)

오늘부터 바로 사용. 로깅을 직접 짜는 건 **바퀴 재발명 + 함정 밟기**입니다.
이미 10년치 함정이 메워진 걸 공짜(MIT)로 쓰는 게 합리적입니다.

#### 트랙 B — 다른 언어 개발자라면 (간접적, 그래도 큼)

**"잘 만든 라이브러리는 어떻게 생겼는가"의 교과서**입니다. 가져갈 것들:

- 기능을 Logger/Formatter/Sink로 쪼개는 **관심사 분리** 감각
- 빠른 경로를 먼저 걸러내는 **성능 설계** (레벨 체크를 맨 앞에)
- `_mt`/`_st` 같은 **선택권을 주는 API 설계** (안전 vs 속도를 사용자가 고르게)
- 락프리 MPMC 큐 (`details/mpmc_blocking_q.h`)
- 스레드풀 설계 (`details/thread_pool.h`)
- 헤더온리/컴파일 양쪽 지원 트릭 (`-inl.h` 패턴)
- 테스트/CI를 갖춘 **배포 가능한 프로젝트의 형태**

이 감각은 React든 PHP든 Python이든 **그대로 이식됩니다.**

#### 트랙 C — 콘텐츠·제품을 만든다면 (사업적)

⭐2.8만 유명 프로젝트 + MIT 라이선스 + 한국어 자료 부족
= **콘텐츠/서비스 소재로 좋은 조건** (→ 7장 참조)

---

## 5. 설치 및 사용법

### 5.1 방법 ① 헤더온리 — 가장 간단 (30초)

설치랄 것도 없습니다. 복사만 하면 끝.

```bash
git clone https://github.com/bmshin94/spdlog.git
cp -r spdlog/include/spdlog  내프로젝트/include/
```

```cpp
#include "spdlog/spdlog.h"

int main() {
    spdlog::info("안녕 spdlog!");
    spdlog::warn("경고 메시지");
    spdlog::error("오류 번호: {}", 42);
}
```

```bash
g++ -std=c++11 -I include main.cpp -o main && ./main
```

출력:
```
[2026-10-08 09:31:02.145] [info] 안녕 spdlog!
[2026-10-08 09:31:02.145] [warning] 경고 메시지
[2026-10-08 09:31:02.145] [error] 오류 번호: 42
```

- **장점**: 설치 0, 의존성 0, 어디든 바로
- **단점**: 매번 전부 컴파일 → 빌드가 느려짐 (파일 많으면 체감 큼)

### 5.2 방법 ② 컴파일 라이브러리 — 권장 (빌드 속도 대폭 향상)

```bash
git clone https://github.com/bmshin94/spdlog.git
cd spdlog && mkdir build && cd build
cmake ..
cmake --build .                  # libspdlog.a 생성
sudo cmake --install .           # 시스템에 설치 (선택)
```

사용 (`CMakeLists.txt`):
```cmake
find_package(spdlog REQUIRED)
add_executable(myapp main.cpp)
target_link_libraries(myapp PRIVATE spdlog::spdlog)
```

헤더온리로 연결할 경우:
```cmake
target_link_libraries(myapp PRIVATE spdlog::spdlog_header_only)
```

### 5.3 방법 ③ 패키지 매니저 — 한 줄 (실무 최다)

```bash
vcpkg install spdlog                        # Windows / 크로스플랫폼
conan install --requires=spdlog/[*]         # 범용
sudo apt install libspdlog-dev              # Ubuntu / Debian
brew install spdlog                         # macOS
sudo port install spdlog                    # MacPorts
pkg install spdlog                          # FreeBSD
dnf install spdlog                          # Fedora
pacman -S spdlog                            # Arch Linux
sudo zypper in spdlog-devel                 # openSUSE
emerge dev-libs/spdlog                      # Gentoo
apt-get install libspdlog-devel             # ALT Linux
conda install -c conda-forge spdlog         # Conda
```

### 5.4 지원 플랫폼

| 플랫폼 | 요구사항 |
|---|---|
| Linux, FreeBSD, OpenBSD, Solaris, AIX | gcc 4.8.1+ / clang 3.5+ |
| Windows | MSVC 2013+, cygwin |
| macOS | clang 3.5+ |
| Android | NDK |

최소 요구 표준: **C++11** (C++17/20도 지원, `SPDLOG_USE_STD_FORMAT`로 `std::format` 사용 가능)

### 5.5 실전 레시피

#### 레시피 1: 컬러 콘솔 + 파일 동시 기록 (가장 흔함)

```cpp
#include "spdlog/spdlog.h"
#include "spdlog/sinks/stdout_color_sinks.h"
#include "spdlog/sinks/rotating_file_sink.h"

void setup_logging() {
    auto console = std::make_shared<spdlog::sinks::stdout_color_sink_mt>();
    console->set_level(spdlog::level::info);     // 콘솔엔 info 이상

    auto file = std::make_shared<spdlog::sinks::rotating_file_sink_mt>(
        "logs/app.log", 1024*1024*10, 5);        // 10MB씩 5개 유지
    file->set_level(spdlog::level::trace);       // 파일엔 전부

    auto logger = std::make_shared<spdlog::logger>(
        "app", spdlog::sinks_init_list{console, file});
    logger->set_level(spdlog::level::trace);
    logger->flush_on(spdlog::level::err);        // 에러는 즉시 디스크에

    spdlog::set_default_logger(logger);          // 전역 등록
}

int main() {
    setup_logging();
    spdlog::info("서버 시작");    // 콘솔 + 파일 둘 다
    spdlog::debug("상세 정보");   // 파일에만
}
```

#### 레시피 2: 고성능 비동기

```cpp
#include "spdlog/async.h"
#include "spdlog/sinks/rotating_file_sink.h"

spdlog::init_thread_pool(8192, 1);   // 큐 8192칸, 작업 스레드 1개
auto logger = spdlog::rotating_logger_mt<spdlog::async_factory>(
    "async", "logs/fast.log", 1024*1024*10, 3);
logger->info("초고속 기록");
```

#### 레시피 3: 출력 모양 바꾸기

```cpp
spdlog::set_pattern("[%Y-%m-%d %H:%M:%S.%e] [%^%-8l%$] [%t] [%s:%#] %v");
// → [2026-10-08 09:31:02.145] [info    ] [12345] [main.cpp:42] 메시지
```

#### 레시피 4: Slack/이메일 알림 연동

```cpp
#include "spdlog/sinks/callback_sink.h"

auto alert = std::make_shared<spdlog::sinks::callback_sink_mt>(
    [](const spdlog::details::log_msg &msg) {
        send_slack(std::string(msg.payload.data(), msg.payload.size()));
    });
alert->set_level(spdlog::level::critical);   // critical만 알림
```

#### 레시피 5: 성능 측정 + Backtrace

```cpp
#include "spdlog/stopwatch.h"

spdlog::enable_backtrace(64);
spdlog::stopwatch sw;
heavy_work();
spdlog::info("처리 완료 {:.3}초", sw);
if (failed) spdlog::dump_backtrace();
```

### 5.6 주요 CMake 빌드 옵션

| 옵션 | 기본값 | 설명 |
|---|---|---|
| `SPDLOG_BUILD_SHARED` | OFF | 동적 라이브러리로 빌드 |
| `SPDLOG_BUILD_EXAMPLE` | ON(단독) | 예제 빌드 |
| `SPDLOG_BUILD_TESTS` | OFF | 테스트 빌드 |
| `SPDLOG_BUILD_BENCH` | OFF | 벤치마크 빌드 (google/benchmark 필요) |
| `SPDLOG_ENABLE_PCH` | OFF | 사전 컴파일 헤더로 빌드 가속 |
| `SPDLOG_FMT_EXTERNAL` | OFF | 외부 fmt 사용 (내장 사본 대신) |
| `SPDLOG_USE_STD_FORMAT` | OFF | `std::format` 사용 (C++20) |
| `SPDLOG_NO_EXCEPTIONS` | OFF | 예외 미사용 (`-fno-exceptions`) |
| `SPDLOG_SANITIZE_ADDRESS` | OFF | 메모리 오류 검사 |
| `SPDLOG_SANITIZE_THREAD` | OFF | 스레드 경쟁 검사 |
| `SPDLOG_DISABLE_DEFAULT_LOGGER` | OFF | 기본 Logger 생성 안 함 |
| `SPDLOG_PREVENT_CHILD_FD` | OFF | 자식 프로세스 fd 상속 방지 |

---

## 6. 자주 묻는 질문 7가지

### Q1. 설치 및 사용법?

→ **5장 참조.** 요약: 헤더 복사(30초) / CMake 빌드(권장) / 패키지 매니저(한 줄) 3가지 방법.

### Q2. 이거 플러그인이야? 스킬이야? MCP야?

#### 🔴 전부 아닙니다. spdlog는 "C++ 라이브러리"입니다.

| 구분 | 정체 | 누가 쓰나 | 어떻게 동작 | spdlog? |
|---|---|---|---|---|
| **라이브러리** | 내 프로그램에 **합쳐지는 코드 부품** | 개발자가 코드 작성 시 | 컴파일 때 내 프로그램 안으로 들어감 | ✅ **이것** |
| **플러그인** | 기존 앱에 끼워 넣는 **확장 기능** | 앱 사용자 | 앱이 실행 중에 불러옴 | ❌ |
| **스킬(Skill)** | AI에게 주는 **작업 지침서(문서)** | AI 에이전트 | AI가 읽고 따름 | ❌ |
| **MCP** | AI와 외부 도구를 잇는 **통신 규약** | AI 에이전트 | 서버로 떠서 AI와 대화 | ❌ |

혼동의 원인은 저장소에 있는 `CLAUDE.md` 파일입니다. 이 파일은 fork 후 추가된 메모이고
spdlog 자체와는 무관합니다.

> **비유**: 자동차(spdlog) 조수석에 "운전법 메모"(CLAUDE.md)를 넣어뒀다고
> 자동차가 AI 도구가 되는 건 아닙니다.

```
spdlog = C++ 라이브러리 (헤더온리 or 정적/동적 라이브러리)
        ≠ 플러그인  ≠ 스킬  ≠ MCP  ≠ 실행 프로그램  ≠ 서비스
```

단, **"spdlog를 MCP 서버로 감싸는 것"은 가능합니다** (→ Q4).

### Q3. API 토큰을 사용해야 돼?

#### 🟢 아니요. 전혀 필요 없습니다.

| 항목 | 필요 여부 |
|---|---|
| API 토큰 / 키 | ❌ 불필요 |
| 회원가입 / 로그인 | ❌ 불필요 |
| 인터넷 연결 | ❌ 불필요 (완전 오프라인 동작) |
| 사용료 / 구독 | ❌ 무료 (MIT) |
| 외부 서버 통신 | ❌ 없음 |
| 사용량 제한 | ❌ 없음 |

spdlog는 **내 컴퓨터 안에서만 도는 코드**입니다. 아무것도 수집하지 않습니다.

**예외 — 내가 외부로 보낼 때.** 이건 spdlog의 요구가 아니라 상대 서비스의 요구입니다:

| Sink | 필요한 것 |
|---|---|
| `tcp_sink` / `udp_sink` | 수신 서버 IP:포트 (인증 없음) |
| `kafka_sink` | Kafka 브로커 주소 (+ 설정 시 SASL 인증) |
| `mongo_sink` | MongoDB 접속 문자열 (ID/PW) |
| `loki_sink` | Grafana Loki URL (+ Grafana Cloud면 API 키) |

### Q4. AI 에이전트를 구축하는데 도움이 될까?

#### ❌ 직접적으로는 — 거의 도움 안 됩니다

AI 에이전트 구축에 필요한 것은 LLM API SDK(Python/TypeScript), MCP, 벡터DB,
오케스트레이션 프레임워크입니다. **이 목록에 C++ 로깅 라이브러리는 없습니다.**

#### ✅ 간접적으로는 — 3가지 실질적 가치

**(1) 에이전트 "관측성(Observability)"의 교과서** ⭐ 가장 큰 가치

AI 에이전트를 만들면 반드시 부딪히는 문제 — "에이전트가 왜 이런 판단을 했는지 모르겠다",
"어느 단계에서 비용이 폭발했는지 모르겠다". 이게 로깅/트레이싱 문제이고,
spdlog의 설계 개념이 그대로 적용됩니다:

| spdlog 개념 | 에이전트에서의 대응 |
|---|---|
| **레벨 7단계** | trace=모든 토큰 / debug=프롬프트 전문 / info=툴호출 / warn=재시도 / error=실패 |
| **다중 Sink** | 콘솔(개발) + 파일(감사) + 분석도구 + Slack(알림) 동시 전송 |
| **Backtrace** | 평소엔 프롬프트를 메모리에만, **실패한 순간에만 직전 32단계 전체 덤프** |
| **MDC** | `session_id`, `user_id`, `trace_id`를 모든 로그에 자동 부착 |
| **구조화 포맷** | JSON 로그로 내보내 LLM이 자기 로그를 읽고 자가 디버깅 |
| **비동기 큐** | 로깅이 LLM 응답 스트리밍을 막지 않게 |

→ **코드를 가져다 쓰는 게 아니라 아키텍처를 배끼는 것**이 실질적 가치입니다.

**(2) 고성능 C++ 에이전트 런타임이라면 — 직접 사용**

- 로컬 LLM 추론 엔진(llama.cpp 계열) 기반 에이전트
- 로봇/드론 자율 에이전트 (ROS2, C++ 필수)
- 초저지연 트레이딩 에이전트
- 게임 NPC AI (Unreal Engine)

**(3) AI로 코드를 읽는 연습 재료**

11,155줄 — AI에게 "코드베이스 분석"을 연습시키기에 이상적인 크기이며,
품질이 높아 AI의 헛소리를 검증하기 쉽습니다.

#### 🔧 spdlog를 MCP로 감쌀 수는 있습니다

```
MCP 서버 "log-analyzer"
  ├ tool: tail_log(file, n)         최근 n줄 읽기
  ├ tool: grep_log(pattern, level)  패턴 검색
  ├ tool: analyze_errors(timerange) 에러 패턴 집계
  └ tool: explain_backtrace(dump)   spdlog backtrace 덤프 해석

→ AI가 "서버 왜 죽었어?"라는 질문에 로그를 직접 읽고 답함
```

| 질문 | 답 |
|---|---|
| spdlog로 AI 에이전트를 만들 수 있나? | ❌ 용도가 다릅니다 |
| AI 에이전트에 spdlog를 넣나? | 🔶 C++ 에이전트면 ✅, Python/TS면 ❌ |
| spdlog에서 배울 게 있나? | ✅ **에이전트 관측성 설계는 거의 그대로 적용** |
| spdlog를 AI 도구로 만들 수 있나? | ✅ MCP 서버로 감싸면 가능 |

### Q5. 수익화할 만한 아이디어가 있어?

→ **7장에서 상세히 다룹니다.** 요약:
spdlog 자체는 팔 수 없지만(MIT), **주변의 결핍**을 팔 수 있습니다.
① 지식(강의/책/컨설팅) ② 편의(설정 생성기/뷰어) ③ 분석(AI 로그 분석 SaaS)

### Q6. 우리가 React나 PHP로 만들 수 있어?

질문을 두 갈래로 나눠야 합니다.

#### 갈래 A: "spdlog를 React/PHP로 포팅?" → ⚠️ 가능하지만 의미 없음

| 항목 | C++ (spdlog) | JavaScript | PHP |
|---|---|---|---|
| 실행 방식 | 기계어 직접 컴파일 | 인터프리터/JIT | 인터프리터 |
| 멀티스레드 | ✅ 진짜 스레드 | ❌ 단일 스레드 | ❌ 요청마다 독립 프로세스 |
| 메모리 제어 | ✅ 수동 (링버퍼 등) | ❌ GC가 관리 | ❌ GC가 관리 |
| 실측 성능 | 초당 **577만 건** | 초당 수십만 건 | 초당 수만 건 |

spdlog의 가치가 "C++의 저수준 제어로 얻은 속도"인데, 그 제어권이 없는 언어로 옮기면
**핵심이 사라집니다.** 게다가 각 생태계에 이미 훌륭한 대안이 있습니다:

| 언어 | 이미 있는 "그 언어의 spdlog" |
|---|---|
| JS/Node | **pino** (가장 빠름), winston, bunyan |
| 브라우저/React | **loglevel**, debug, consola |
| PHP | **Monolog** (사실상 표준, Laravel 기본) |
| Python | loguru, structlog |
| Go | zap, zerolog |
| Rust | tracing, log |

**Monolog이 PHP의 spdlog입니다.** 구조도 거의 같습니다 (Logger → Formatter → Handler = Sink).

#### 갈래 B: "React/PHP로 spdlog 관련 **제품**을?" → ✅✅ 이게 정답

여기가 진짜 기회입니다 (→ 7장 상세).

| 제품 | 기술 | 난이도 | 기간 |
|---|---|---|---|
| **설정 생성기** | React 단독 (백엔드 불필요) | ★☆☆ | 1~2주 |
| **로그 뷰어/대시보드** | PHP + React | ★★☆ | 1~2개월 |
| **AI 로그 분석기** | React + PHP + LLM API | ★★★ | 2~3개월 |

| 질문 | 답 |
|---|---|
| spdlog를 React/PHP로 포팅? | ⚠️ 가능하지만 **의미 없음** |
| React/PHP로 spdlog 주변 제품? | ✅✅ **가장 현실적이고 돈이 되는 방향** |
| 추천 1순위 | **설정 생성기** (1~2주, 즉시 가능, 검증용) |
| 추천 최대 기회 | **AI 로그 분석기** (2~3개월, SaaS 가능) |

### Q7. 유튜브 강의 영상으로 제작 가능할까?

#### 🟢 가능합니다. 단 "조건부"입니다.

#### ✅ 유리한 점

| 요소 | 근거 |
|---|---|
| **주제 신뢰도** | ⭐2.8만, 업계 표준. "듣보잡 소개"가 아님 |
| **라이선스** | MIT — 코드 인용, 화면 표시, 상업적 영상화 전부 합법 |
| **한국어 자료 공백** | 유튜브에 spdlog 한국어 심층 강의가 사실상 없음 |
| **시각적 표현 용이** | 로그는 화면에 바로 뜸. 색깔, 파일 회전, 성능 숫자 |
| **실습 진입장벽 낮음** | 헤더 복사 1번으로 첫 로그 출력. 시청자 이탈 적음 |
| **콘텐츠 분량 충분** | 28종 Sink, 40개 패턴, 비동기 구조, 테스트, CI → 10편 이상 |
| **명확한 "와" 순간** | "초당 577만 건" 실측, 비동기 전후 비교 → 썸네일 소재 |

#### ⚠️ 불리한 점

| 요소 | 현실 |
|---|---|
| **시장 크기** | 한국 C++ 유튜브 시청자층이 작음 (Python/JS 대비 1/20 수준) |
| **수익 직결성** | 조회수 광고 수익만으론 수지 안 맞을 가능성 높음 |
| **난이도 함정** | 비동기/락프리 큐 설명은 어렵고, 못하면 중간 이탈 큼 |

#### 🎯 해결 전략: "spdlog 강의"가 아니라 "C++ 로깅/성능 강의"로 포장

- ❌ 나쁜 제목: `spdlog 사용법 1강 - 설치하기`
- ✅ 좋은 제목:
  - `C++ 로그 때문에 서버가 느려진다? 200배 빠르게 만드는 방법`
  - `printf 쓰면 안 되는 4가지 이유 (실측 포함)`
  - `게임 서버는 초당 500만 건 로그를 어떻게 남기나`
  - `면접 단골 질문: 락프리 큐, 코드로 이해하기`

> "spdlog"를 검색하는 사람은 적지만, "C++ 성능", "서버 로그", "멀티스레드"를
> 검색하는 사람은 많습니다. 후자를 낚아서 spdlog로 해결하는 구조.

#### 커리큘럼 설계안 (10편 + 심화 4편)

| # | 제목 | 핵심 장면 | 길이 |
|---|---|---|---|
| 0 | printf의 4가지 지옥 | **4개 문제를 실제로 재현해 보여줌** | 12분 |
| 1 | 30초 설치 + 첫 로그 | 헤더 복사 → 컬러 출력 | 8분 |
| 2 | 레벨 7단계 — 로그 음량 조절기 | 레벨 바꾸며 출력 변화 | 10분 |
| 3 | 패턴 40개 완전 정복 | **설정 생성기 띄워 실시간 시연** | 15분 |
| 4 | Sink 28종 — 어디로든 보낸다 | 콘솔+파일+Slack 동시 시연 | 15분 |
| 5 | 파일 자동 관리 (회전/일별/시간별) | **파일이 교체되는 걸 화면 녹화** | 12분 |
| 6 | 비동기 모드 — 200배의 비밀 | **동기 vs 비동기 실측 비교** 🔥 | 18분 |
| 7 | Backtrace — 사고 직전 CCTV | 에러 순간 32단계 덤프 | 12분 |
| 8 | 실전: 서버 로깅 아키텍처 설계 | 처음부터 끝까지 구축 | 25분 |
| 9 | 성능 튜닝 `tweakme.h` 완전 분석 | 옵션별 벤치마크 실측 | 20분 |
| **S1** | 소스코드 읽기: 락프리 MPMC 큐 | `mpmc_blocking_q.h` 한 줄씩 | 30분 |
| **S2** | 소스코드 읽기: 스레드풀 설계 | `thread_pool.h` | 25분 |
| **S3** | 헤더온리 vs 컴파일 — `-inl.h` 트릭 | 라이브러리 설계 기법 | 20분 |
| **S4** | 3,717줄 테스트 + CI 설계 분석 | 실무 품질 관리 | 25분 |

> **심화 4편(S1~S4)이 진짜 차별점입니다.** "쓰는 법"은 README에 있지만,
> **"어떻게 만들었나"를 한국어로 해설한 영상은 없습니다.**
> C++ 중급자가 돈 내고 볼 콘텐츠이며, 면접 대비 수요와 겹칩니다.

#### 수익 구조 (다층화)

```
1층) 유튜브 광고       →  작음. "미끼"로 취급
2층) 멤버십/후원       →  소스코드 심화편 전용 공개
3층) 인프런/유데미 유료 →  💰 메인. "C++ 로깅 & 성능 완전정복"
4층) 설정 생성기 웹앱   →  영상 내 자연스러운 노출 + 트래픽 확보
5층) 전자책/PDF        →  영상 내용 정리본 판매
6층) 기업 사내교육      →  💰💰 단가 최고. 게임사/금융권 C++ 팀
```

#### 제작 실무 팁

- **첫 10초가 전부**: 영상 시작 즉시 "동기 0.6초 vs 비동기 0.003초" 숫자 제시
- **항상 실행 화면 포함**: C++ 콘텐츠의 최대 적은 "슬라이드만 읽기"
- **벤치마크를 라이브로**: `bench/` 폴더 벤치마크를 그 자리에서 실행 → 신뢰도 급상승
- **예제 코드 전부 GitHub 공개**: 영상 설명란 링크 → 구독 유입
- **에러를 숨기지 말기**: 컴파일 에러 장면을 그대로 두고 고치는 과정 → 초보 이탈 방지에 효과적

---

## 7. 수익화 전략

### 전제

> ⚠️ **spdlog 자체는 팔 수 없습니다.** MIT 라이선스라 누구나 공짜로 가져갑니다.
> ✅ **팔 수 있는 건 "spdlog 주변의 결핍"입니다.**

전수조사에서 확인한 **실제 결핍 5개** — 모든 아이디어의 근거:

| # | spdlog가 안 해주는 것 | 결과 |
|---|---|---|
| 1 | 설정이 복잡함 (Sink 28종 × 패턴 40개 × 옵션 30개) | 매번 README 뒤짐 |
| 2 | **로그를 읽는 도구가 없음** (쓰기만 함) | 터미널에서 `grep` |
| 3 | **한국어 자료가 거의 없음** | 영어 README + 위키뿐 |
| 4 | **로그 분석/해석을 안 해줌** | 사람이 눈으로 GB 단위 읽기 |
| 5 | **AI 도구와 연결점 없음** | 2026년 트렌드에서 단절 |

---

### 🥇 1순위: spdlog 설정 생성기 (Config Builder)

**근거: 결핍 #1** | **난이도 ★☆☆** | **기간 1~2주** | **투자금 거의 0**

웹에서 클릭으로 spdlog 설정을 만들고, 결과를 실시간 미리보기하고, C++ 코드를 복사해 가는 도구.

```
┌─────────────────────────────────────────────────┐
│  spdlog 설정 빌더                                │
│                                                 │
│  출력 대상  ☑ 콘솔(컬러)  ☑ 회전파일  ☐ Kafka    │
│  로그 레벨  [info ▾]                             │
│  모드       ◉ 비동기  ○ 동기                      │
│  패턴       [%Y-%m-%d %H:%M:%S.%e] [%l] %v       │
│                                                 │
│  ┌─ 실시간 미리보기 ──────────────────────────┐  │
│  │ [2026-10-08 09:31:02.145] [info] 메시지   │  │
│  └───────────────────────────────────────────┘  │
│                                                 │
│  ┌─ 생성된 C++ 코드 ──────────────────[복사]─┐  │
│  │ #include "spdlog/async.h"                 │  │
│  │ spdlog::init_thread_pool(8192, 1);        │  │
│  └───────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
```

#### 왜 1순위인가

- **존재하지 않습니다** (경쟁 없음)
- 수요가 **명확하고 반복적**입니다 (프로젝트 시작할 때마다 필요)
- React만으로 됩니다 → **서버비 0원**
- 1~2주에 완성 → **시장 반응을 빠르게 검증**
- 뒤이은 모든 제품의 **트래픽 입구**가 됨

#### 기능 명세

```
① Sink 선택       체크박스 (콘솔/회전/일별/syslog/TCP/Kafka...)
② 레벨 설정       드롭다운 + Sink별 개별 레벨
③ 동기/비동기     라디오 + 큐 크기/오버플로 정책
④ 패턴 빌더       플래그 40개를 드래그&드롭 + 설명 툴팁
⑤ 실시간 미리보기  패턴 바꾸는 즉시 가상 로그 출력 모양 갱신 ⭐핵심
⑥ 코드 생성       C++ 코드 + CMakeLists + tweakme.h 세트
⑦ 프리셋          "게임서버", "임베디드", "금융", "개발용" 원클릭
⑧ 공유 링크       URL에 설정 인코딩 → 팀에 공유
```

**⑤번이 승부처입니다.** 패턴 문자열을 바꿀 때 결과 로그 한 줄이 즉시 바뀌는 것 —
이것만으로 사람들이 북마크합니다.

#### 수익화

| 경로 | 예상 |
|---|---|
| 무료 + 후원 버튼 | 소액이지만 꾸준 |
| Pro ($5/월) | 프리셋 무제한 저장, 팀 공유, 설정 버전 관리 |
| 광고 | 개발자 대상 광고 단가 양호 |
| **간접 가치** | 포트폴리오 + 유튜브 유입 + 다음 제품 고객 확보 ⭐ |

#### 실행 순서

```
1주차: 패턴 미리보기 엔진 (JS로 플래그 40개 구현) ← 여기가 핵심
2주차: UI + 코드 생성 + 프리셋 → Vercel/Netlify 무료 배포
그 후:  Reddit r/cpp, HackerNews, spdlog Discussions에 공유
```

> 💡 spdlog 저장소 README에 링크 추가를 제안(PR)하면 공식 유입 경로가 생깁니다.

---

### 🥈 2순위: AI 로그 분석 SaaS 🔥 최대 기회

**근거: 결핍 #2+#4+#5** | **난이도 ★★★** | **MVP 3주 / 정식 2~3개월**

로그 파일을 넣으면 **AI가 "무슨 일이 일어났는지"를 사람 말로 설명**해주는 서비스.

#### 왜 유망한가

- 개발자가 로그 읽는 시간 = **직접적인 비용**. 가치 환산이 쉬움
- 2026년 AI 도구 수요가 최고점
- spdlog 사용자만이 아니라 **모든 로그 사용자**가 고객 (시장이 훨씬 큼)
- Q4의 **"spdlog × AI 에이전트" 교차점** — 유일하게 트렌드와 연결되는 아이디어

#### 동작 흐름

```
[업로드/연동]  로그 파일 드롭 or S3/파일경로 연동
      ↓
[파싱]        PHP/Python — spdlog 패턴 자동 인식해 구조화
      ↓
[압축]        에러 클러스터링 + 반복 제거 (토큰 비용 절감 핵심 ⭐)
      ↓
[AI 분석]     LLM API — 원인 추정 + 해결책 제시
      ↓
[리포트]      React — 타임라인/히트맵/원인/해결책
```

#### 리포트 예시

```
📋 분석 결과 (app.log, 2.4GB, 1,847만 줄)

🔴 핵심 문제
   DB 커넥션 풀 고갈 — 13:02:11 최초 발생, 13:05까지 지속

📊 타임라인
   13:00 ─────▁▁▃▇████▇▃▁───── 13:10
              에러 급증 구간 (정상 대비 340배)

🔍 추정 원인 (신뢰도 높음)
   1. 커넥션 반환 누락 — OrderService.cpp:218 예외 발생 시 close() 미호출
   2. 동시 요청 급증 — 13:02에 요청량 8배 (프로모션 시작 시각 일치)

💡 해결 방안
   - 즉시: 풀 크기 20 → 50 (임시 완화)
   - 근본: RAII 패턴으로 커넥션 래핑 (예외 안전)
   - 예방: 풀 사용률 80% 초과 시 critical 로그 + 알림 추가
```

#### 💰 핵심 기술 장벽: 토큰 비용 관리

GB 로그를 그대로 LLM에 넣으면 **원가가 매출을 초과합니다.**
이게 이 사업의 진짜 기술 장벽이자 **진입 방어막**입니다:

```
2.4GB 로그 → ① 레벨 필터 (error/critical만)       → 120MB
            → ② 템플릿 클러스터링 (동일 패턴 묶음)  → 3,400 그룹
            → ③ 대표 샘플 + 빈도 + 타임라인만 추출  → 180KB
            → ④ LLM 입력                           → 약 45K 토큰 ✅
```

→ 1회 분석 원가를 수십~수백 원 수준으로 맞출 수 있습니다. **여기를 잘 만드는 게 사업의 전부입니다.**

#### 가격 설계

| 플랜 | 가격 | 내용 |
|---|---|---|
| Free | $0 | 월 3회, 10MB 제한 (체험) |
| Solo | $9/월 | 월 50회, 500MB |
| Team | $49/월 | 무제한, 5GB, 실시간 연동, Slack 알림 |
| **Self-hosted** | $499/년 | 온프레미스 (금융/공공 — 로그 외부 전송 금지 고객) ⭐ |
| **MCP 서버판** | 별도 | AI 코딩 도구에 바로 붙이는 버전 🔥 |

> 마지막 두 개가 숨은 금맥입니다. **Self-hosted**는 "로그를 외부에 못 보내는"
> 금융/공공 수요(단가 높음), **MCP 서버판**은 AI 코딩 도구 사용자가 바로 설치해 쓰는 신규 채널입니다.

#### MCP 서버판 상세

```
MCP 서버 "log-insight"
 ├ analyze_log(path)           로그 분석 리포트
 ├ find_root_cause(error_id)   원인 추적
 ├ tail_and_watch(path)        실시간 감시
 └ explain_backtrace(dump)     spdlog backtrace 덤프 해석

→ 개발자가 AI 코딩 도구에서 "서버 왜 죽었어?" 물으면
  AI가 이 도구로 로그를 직접 읽고 답함
```

**spdlog의 backtrace 덤프를 AI가 해석해주는 것** — spdlog 특화 기능이라 차별점이 명확합니다.

---

### 🥉 3순위: 한국어 교육 콘텐츠 패키지

**근거: 결핍 #3** | **난이도 ★★☆** | **기간 2~3개월** | **현금화가 가장 빠름**

```
① 유튜브 무료 10편        → 신뢰 확보 + 유입 (Q7 커리큘럼)
② 인프런/유데미 유료 강의  → 💰 메인 수익
   "C++ 로깅 & 성능 완전정복" (심화편 S1~S4 포함, 10~15시간)
   가격: 59,000~99,000원
③ 전자책/PDF             → 영상 내용 정리 + 치트시트
   가격: 15,000~29,000원
④ 템플릿 번들            → 즉시 쓰는 설정 10종 + CMake 세트
   가격: 19,000원 or 강의 구매자 무료
⑤ 기업 사내교육           → 💰💰 단가 최고
   게임사/금융권 C++ 팀 대상, 반일~2일, 회당 100~500만원
```

#### 왜 현금화가 가장 빠른가

- **투자금 0원** (장비 외)
- 1편 만들어 올리면 **즉시 반응 측정** 가능
- ⑤ 기업교육은 **단 1건으로도 수개월 서버비를 커버**
- 그리고 **1순위 제품(설정 생성기)의 홍보 채널**이 됨

#### 차별화 원칙

> **"쓰는 법"은 README에 있습니다. 한국어로 아무도 안 한 건 "어떻게 만들었나"입니다.**

- 락프리 MPMC 큐를 한 줄씩 해부 (`mpmc_blocking_q.h`)
- 헤더온리/컴파일 양립 트릭 (`-inl.h` 패턴)
- `_mt`/`_st` 정책 기반 설계
- 3,717줄 테스트 + CI 6종 조합 설계 분석

---

### 4순위: 경량 로그 뷰어 (오픈소스 → 유료 호스팅)

**근거: 결핍 #2** | **난이도 ★★☆** | **기간 1~2개월**

```
오픈소스로 공개 (GitHub ⭐ 확보)
      ↓
유료 호스팅판 ($19/월) + 기업 지원 계약
```

**포지셔닝**: ELK는 무겁고 Datadog은 비쌉니다.
**"작은 팀이 Docker 한 줄로 띄우는 로그 뷰어"** 틈새.

- PHP 백엔드 (파일 파싱 + WebSocket tail) + React 프론트 (차트/필터/검색)
- spdlog 패턴 자동 인식을 **기본 지원** → spdlog 커뮤니티에서 먼저 퍼짐
- ⭐가 쌓이면 **2순위 SaaS의 프론트엔드로 재활용** 가능

---

### 5순위: 기타 틈새 수익원

| 아이디어 | 설명 | 수익 |
|---|---|---|
| **커스텀 Sink 개발 대행** | 특정 회사 내부 시스템용 Sink 제작 | 건당 50~300만원 |
| **성능 튜닝 컨설팅** | "로그 때문에 느려요" 문제 해결 | 시간당 10~30만원 |
| **spdlog 플러그인 팩** | 유료 Sink 모음 (OpenTelemetry, Sentry, ClickHouse) | $29 단품 |
| **마이그레이션 도구** | log4cpp/glog/printf → spdlog 자동 변환 CLI | $49 or 오픈소스+후원 |
| **치트시트 상품** | 패턴 40개 + Sink 28종 포스터/카드 | 소액 다건 |
| **GitHub Sponsors** | 위 도구들을 오픈소스로 내고 후원 받기 | 변동 |

---

## 8. 실행 로드맵

```
[0~2주]  ① 설정 생성기 React 앱 제작·배포           투자 0원
         → Reddit r/cpp, HN, spdlog Discussions 공유
         → 📊 반응 측정: 트래픽이 나오나?

[2~6주]  ② 유튜브 0편(문제제기) + 6편(비동기 실측) 업로드
         → 영상 안에서 ①번 도구 노출 (상호 유입)
         → 📊 반응 측정: C++ 한국 시장이 반응하나?

[6~12주] ③ 반응에 따라 분기
         ├ 교육이 잘 되면  → 유료 강의 제작 + 기업교육 영업 (💰 빠른 현금)
         └ 도구가 잘 되면  → ④ AI 로그 분석기 MVP 착수 (💰💰 큰 그림)

[3~6개월] ⑤ AI 로그 분석 SaaS 정식 출시
         + MCP 서버판 동시 출시 (AI 도구 채널 선점)
         + Self-hosted 판으로 기업 영업
```

### 핵심 원칙 3가지

1. **작게 먼저 검증하세요.** 설정 생성기(2주)로 "사람들이 관심 있나"를 먼저 확인.
   바로 SaaS에 3개월 쏟는 건 위험합니다.
2. **유튜브는 수익이 아니라 유입입니다.** 광고 수익을 기대하면 실망합니다.
   **신뢰 확보 → 유료 강의/기업교육/도구 유입**이 목적입니다.
3. **AI 결합이 유일한 큰 레버입니다.** 설정 생성기·강의는 안정적이지만 상한이 있습니다.
   **AI 로그 분석 + MCP**가 유일하게 큰 시장과 연결됩니다.

---

## 9. 라이선스 주의사항

spdlog는 **MIT 라이선스**입니다.

| 행위 | 가능 여부 |
|---|---|
| spdlog 코드를 제품에 포함 | ✅ 허용 (저작권 고지만 포함) |
| spdlog를 영상에 띄우고 해설 | ✅ 허용 |
| spdlog 기반 유료 강의 판매 | ✅ 허용 |
| spdlog 수정판을 제품에 넣어 판매 | ✅ 허용 |
| 소스 공개 의무 | ❌ 없음 (GPL과 다름) |
| **"spdlog 공식"이라고 표기** | ⚠️ **금지** (상표/혼동 유발) |
| **저작권 고지 삭제** | ⚠️ **라이선스 위반** |

> ✅ **"공식"이라고 하지 말고, LICENSE 파일을 같이 배포**하면 전부 안전합니다.

내장된 fmt 라이브러리도 별도 라이선스가 있습니다:
`include/spdlog/fmt/bundled/fmt.license.rst` 참조.

---

## 부록: 참고 링크

| 항목 | 링크 |
|---|---|
| 이 저장소 | https://github.com/bmshin94/spdlog |
| 원본 프로젝트 | https://github.com/gabime/spdlog |
| 공식 Wiki (상세 문서) | https://github.com/gabime/spdlog/wiki |
| 커스텀 포맷팅 가이드 | https://github.com/gabime/spdlog/wiki/Custom-formatting |
| 나만의 Sink 만들기 | https://github.com/gabime/spdlog/wiki/Sinks#implementing-your-own-sink |
| 최신 릴리스 | https://github.com/gabime/spdlog/releases/latest |
| fmt 라이브러리 | https://github.com/fmtlib/fmt |
| 벤치마크 소스 | [bench/bench.cpp](../bench/bench.cpp) |
| 사용 예제 25종 | [example/example.cpp](../example/example.cpp) |
| 성능 튜닝 스위치 | [include/spdlog/tweakme.h](../include/spdlog/tweakme.h) |

### 다른 언어의 대응 라이브러리

| 언어 | 라이브러리 | 링크 |
|---|---|---|
| Node.js | pino | https://github.com/pinojs/pino |
| PHP | Monolog | https://github.com/Seldaek/monolog |
| Python | loguru | https://github.com/Delgan/loguru |
| Go | zap | https://github.com/uber-go/zap |
| Rust | tracing | https://github.com/tokio-rs/tracing |

---

*분석일: 2026-10-08 · 대상 버전: spdlog v1.17.0*
