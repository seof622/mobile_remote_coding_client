# mobile_remote_coding_client

## 프로젝트 개요

Windows Desktop에서 실행되는 Remote Coding Agent에 연결하여 스마트폰에서 Codex 기반 개발 작업을 제어하는 모바일 클라이언트다.

이 앱은 모바일 IDE를 만드는 것이 아니라, 스마트폰에서 Codex Agent에게 작업을 지시하고 진행 상황과 변경 사항을 검토하며 필요한 작업을 승인하는 UX를 목표로 한다.

---

## 핵심 목표

스마트폰만으로 다음 Workflow를 수행할 수 있어야 한다.

```text
프로젝트 선택
    ↓
Codex Session 선택
    ↓
개발 요청
    ↓
Codex 작업 진행
    ↓
결과 확인
    ↓
Diff 확인
    ↓
Test 확인
    ↓
Commit / Push 승인
```

모바일에서는 코드 작성 자체보다 Agent에게 작업을 지시하고 결과를 검토하는 경험을 우선한다.

---

## 지원 환경

초기 우선순위:

- Android

향후:

- iOS

구현 후보:

- Flutter
- Kotlin Compose

---

## 주요 화면

```text
Connection
     │
     ▼
Desktop
     │
     ▼
Project List
     │
     ▼
Project
     │
     ├── Agent Chat
     ├── Sessions
     ├── Git
     ├── Diff
     ├── Test
     └── Terminal
```

---

## 기능 요구사항

### FR-M01. Desktop 등록

등록 정보:

- 이름
- Tailscale IP / Hostname
- Port
- Authentication Token

예:

```text
My Desktop

100.x.x.x:8080
```

### FR-M02. 연결 상태 표시

상태:

- Connected
- Connecting
- Disconnected
- Desktop Offline

### FR-M03. 자동 재연결

Wi-Fi ↔ LTE/5G 변경 등 일시적인 연결 종료가 발생해도 자동 재연결해야 한다.

### FR-M04. 프로젝트 목록

프로젝트 카드에서 다음 정보를 표시:

```text
SmartHome

branch: feature/matter
3 files changed

Codex: Running
```

### FR-M05. 프로젝트 Dashboard

표시 정보:

- 현재 Branch
- Git 상태
- 실행 중인 Codex Session
- 최근 작업
- 테스트 상태

### FR-M06. Codex Chat

Chat 형태로 작업 요청.

예:

```text
User

Matter device pairing 부분 구현해줘.
기존 architecture는 유지해.
```

Codex 응답은 Streaming 방식으로 표시한다.

### FR-M07. 작업 상태 표시

예:

```text
Reading files...

Analyzing architecture...

Editing:
src/device/PairingManager.kt

Running tests...
```

### FR-M08. Session 관리

- Session 생성
- Session 조회
- Session 선택
- Session 종료
- 기존 Session 재접속

### FR-M09. Git Status

예:

```text
Modified

M PairingManager.kt
M MatterRepository.kt

Added

A PairingManagerTest.kt
```

### FR-M10. Diff Viewer

- 파일 단위 변경 내용 조회
- 추가/삭제 Line 시각적 구분

예:

```text
PairingManager.kt

+ suspend fun pairDevice(...)
+ {
+     ...
+ }
```

### FR-M11. Test 실행

예:

```text
Run Tests
```

결과:

```text
42 Tests

41 Passed
1 Failed
```

실패한 테스트 선택 시 상세 로그를 표시한다.

### FR-M12. Build 실행

- 프로젝트 Build 실행
- 진행 상태 표시
- Build 결과 표시

### FR-M13. Terminal

고급 사용자를 위한 간단한 Terminal 기능.

예:

```bash
git status
git log --oneline -5
```

전체 SSH Terminal보다 Desktop Agent Command API 사용을 우선한다.

### FR-M14. 작업 취소

Codex 실행 중 작업 Cancel 지원.

```text
Codex is working...

[Stop]
```

### FR-M15. Approval

Desktop Agent에서 위험 작업 승인 요청 시 모바일에서 승인 또는 거절.

```text
Codex wants to execute:

git push origin feature/matter

[Reject]      [Approve]
```

### FR-M16. Commit 생성

- Commit 요청
- Commit Message 직접 입력
- Codex를 통한 Commit Message 생성

### FR-M17. 작업 완료 요약

예:

```text
Task Completed

Changed
- PairingManager.kt
- MatterRepository.kt

Added
- PairingManagerTest.kt

Test
✓ 42 Passed

Git
3 files changed
```

---

## 핵심 UX

이 앱은 IDE의 모바일 버전을 목표로 하지 않는다.

핵심 UX:

```text
명령
↓
관찰
↓
검토
↓
승인
```

스마트폰에서는 코드 한 줄 한 줄을 작성하기보다 Codex Agent를 관리한다.

---

## 기본 화면 예시

### Project 화면

```text
SmartHome

feature/matter

Codex
● Working

────────────────────

You

Matter pairing 기능 구현해줘.

────────────────────

Codex

기존 구조를 확인했습니다.

현재 다음 파일을 수정하고 있습니다.

MatterRepository.kt
PairingManager.kt

────────────────────

[ Stop ]
```

---

## 하단 Navigation

초기 구조:

```text
[Chat] [Changes] [Tests] [Terminal]
```

### Chat

Codex와 작업 지시 및 대화.

### Changes

Git Status 및 Diff.

### Tests

Build / Test 결과.

### Terminal

직접 Command 실행.

---

## 알림

향후 Push Notification 지원.

알림 대상:

- Codex 작업 완료
- Build 완료
- Test 실패
- 사용자 승인 요청
- Agent 오류

예:

```text
Codex finished your task.

3 files changed
42 tests passed.
```

---

## 보안 요구사항

- 인증 Token은 Android Keystore 등에 안전하게 저장
- 사용자 승인 없는 위험 작업 실행 금지
- Desktop 연결 정보와 Credential 평문 저장 금지
- Tailscale Network 외부 연결 기본 차단

---

## 비기능 요구사항

### 모바일 UX

- 한 손 사용 가능한 인터페이스
- 주요 기능 1~2번의 터치로 접근

### 네트워크

- 모바일 네트워크의 일시적 연결 종료를 정상 상황으로 간주
- WebSocket 종료 시 자동 복구

### 배터리

- 백그라운드 지속 Polling 지양
- WebSocket 또는 Push Notification 우선

---

## MVP 범위

1차 버전:

- Desktop 연결
- 프로젝트 목록
- 프로젝트 선택
- Codex Session 생성
- Codex Prompt 입력
- Streaming 응답
- Session 재접속
- Git Status
- Git Diff
- Test 실행
- Codex 작업 Cancel

MVP 제외 후보:

- Push Notification
- Commit
- Push
- Approval
- Terminal
- 여러 Desktop 관리

---

## 향후 확장

- 여러 Desktop 등록
- 음성 Codex 명령
- 사진 / 문서 첨부
- GitHub Issue 연결
- Pull Request 확인
- CI 결과 확인
- 작업 Queue
- 여러 Agent 동시 작업
- Agent별 작업 진행률
- Codex 작업 비용 / Token 확인
- Desktop 화면 원격 확인
