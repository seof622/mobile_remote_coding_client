# mobile_remote_coding_client

Windows 데스크탑에서 실행되는 `desktop_remote_coding_agent`에 연결하여, 스마트폰에서 **Codex App Server 기반 개발 작업을 제어하는 모바일 클라이언트**다.

이 앱은 모바일 IDE가 아니라 **Codex Remote Control / Review Client**를 목표로 한다.

> 핵심 UX: 명령 → 관찰 → 승인 → 검토

---

## 핵심 아키텍처

```text
┌──────────────────────┐
│ Mobile Client        │
│ Flutter / Android    │
└──────────┬───────────┘
           │
           │ Tailscale
           │ REST / WebSocket
           ▼
┌──────────────────────────────┐
│ Desktop Remote Coding Agent  │
│                              │
│ Auth / Events / Approvals    │
│ Git / Build / Test           │
└──────────────┬───────────────┘
               │
               │ JSON-RPC
               ▼
┌──────────────────────────────┐
│ Codex App Server             │
│ Thread → Turn → Item         │
└──────────────────────────────┘
```

모바일 앱은 Codex App Server에 직접 연결하지 않는다.

Desktop Agent가 Codex 프로토콜과 네트워크 연결을 관리하고 모바일에는 안정적인 애플리케이션 API를 제공한다.

---

## 핵심 목표

스마트폰에서 다음 Workflow를 수행한다.

```text
Desktop 연결
    ↓
프로젝트 선택
    ↓
Codex Thread 선택/생성
    ↓
개발 요청
    ↓
실시간 작업 진행 확인
    ↓
필요한 Command/File/Permission 승인
    ↓
작업 완료
    ↓
Diff / Test 결과 검토
```

---

## 지원 환경

초기:

- Android
- Flutter + VS Code 권장

향후:

- iOS

---

## 핵심 데이터 모델

모바일은 Codex의 다음 개념을 UI 모델로 사용한다.

```text
Project
  └── Thread
       └── Turn
            ├── User Message
            ├── Agent Message
            ├── Command Execution
            ├── File Change
            └── Approval
```

---

## 주요 화면

```text
Connection
   ↓
Project List
   ↓
Project
   ├── Chat
   ├── Activity
   ├── Changes
   ├── Tests
   └── Terminal/Tools
```

---

## 기능 요구사항

### FR-M01. Desktop 등록

저장 정보:

- 이름
- Tailscale IP / MagicDNS hostname
- Port
- Client Authentication Token

예:

```text
My Desktop
desktop.tailnet-name.ts.net:8080
```

### FR-M02. 연결 상태

- Connected
- Connecting
- Disconnected
- Desktop Offline
- Reconnecting

### FR-M03. 자동 재연결

Wi-Fi ↔ LTE/5G 변경 등 네트워크 전환 후 자동 재연결한다.

재연결 후 서버에서 현재 상태를 다시 동기화한다.

### FR-M04. 프로젝트 목록

예:

```text
SmartHome

feature/matter
3 files changed

Codex: Working
1 approval pending
```

### FR-M05. Thread 목록

프로젝트에 연결된 Codex Thread를 확인하고 선택할 수 있어야 한다.

- 새 Thread 생성
- 기존 Thread Resume
- 최근 작업 시간
- 현재 상태
- 마지막 메시지 요약

### FR-M06. Codex Chat

자연어 요청을 입력한다.

```text
Matter pairing 기능을 구현해줘.
현재 architecture는 유지하고 테스트까지 작성해.
```

Desktop Agent가 이를 Codex `turn/start`로 전달한다.

### FR-M07. Streaming 응답

Agent Message delta를 실시간 표시한다.

긴 메시지를 모두 받은 뒤 한 번에 표시하지 않는다.

### FR-M08. Activity Timeline

Codex 작업을 단순 텍스트 로그가 아닌 Item 단위로 보여준다.

예:

```text
✓ Read src/auth/AuthService.kt

● Running
  ./gradlew test

⚠ Approval required
  npm install axios

✓ Updated
  PairingManager.kt
```

### FR-M09. Command Approval

Desktop Agent에서 Command Approval 요청이 오면 즉시 사용자에게 표시한다.

```text
실행 승인 필요

npm install axios

cwd
/home/user/project

Reason
Network access required

[거절] [이번만 승인] [세션 동안 승인]
```

서버가 제공한 `availableDecisions`를 기준으로 버튼을 구성한다.

### FR-M10. File Change Approval

파일 변경 승인이 필요한 경우:

- 대상 파일
- 변경 요약
- 승인 사유
- 가능하면 Diff Preview

를 표시한다.

사용자는 승인/거절할 수 있어야 한다.

### FR-M11. Permission Approval

지원되는 경우 다음 권한 요청을 표시한다.

- Network access
- 추가 filesystem access

요청 범위를 사용자가 명확히 확인할 수 있어야 한다.

### FR-M12. Approval 상태 처리

Approval 상태:

```text
Pending
Accepted
Declined
Cancelled
Resolved
```

이미 resolve된 요청의 버튼은 비활성화한다.

### FR-M13. Turn 중단

작업 중:

```text
[Stop]
```

을 누르면 Desktop Agent를 통해 Codex Turn interrupt를 요청한다.

### FR-M14. Git Status

- 현재 Branch
- Modified
- Added
- Deleted
- staged / unstaged
- Ahead / Behind

### FR-M15. Diff Viewer

- 파일별 Diff
- 추가/삭제 Line 구분
- staged/unstaged 구분
- 대용량 Diff 접기

### FR-M16. Build / Test

- Build 실행
- Test 실행
- 진행 상태
- stdout/stderr
- 성공/실패
- 실행 시간

Test 결과 예:

```text
42 tests

✓ 41 passed
✕ 1 failed
```

### FR-M17. 작업 완료 요약

```text
Task Completed

Changed
- PairingManager.kt
- MatterRepository.kt

Added
- PairingManagerTest.kt

Tests
✓ 42 passed

Git
3 files changed
```

### FR-M18. Pending Approval 복구

앱이 백그라운드로 이동하거나 연결이 끊긴 동안 승인 요청이 발생할 수 있다.

재연결 시 Pending Approval 목록을 다시 가져와 사용자에게 표시해야 한다.

---

## 핵심 UX

이 앱은 IDE를 축소해서 휴대폰에 넣는 것이 아니다.

```text
지시
 ↓
진행 상황 관찰
 ↓
위험 작업 승인
 ↓
결과 검토
```

모바일에서 직접 코드를 타이핑하는 것은 보조 기능이다.

---

## Project 화면 예시

```text
SmartHome
feature/matter

Codex ● Working

────────────────────

You

Matter pairing 구현하고
관련 테스트까지 작성해줘.

────────────────────

Codex

기존 Repository 구조를 확인하고 있습니다.

────────────────────

Activity

✓ Read MatterRepository.kt
✓ Read PairingManager.kt
● Running tests...

────────────────────

[ Stop ]
```

---

## Approval 화면 예시

```text
⚠ Command Approval

Codex wants to run:

git push origin feature/matter

Working Directory
/home/user/projects/smart-home

[Decline]

[Accept Once]

[Accept for Session]
```

---

## 하단 Navigation

```text
[Chat] [Activity] [Changes] [Tests]
```

### Chat

Codex Thread 대화.

### Activity

Command, File Change, Approval 등 실시간 Item Timeline.

### Changes

Git Status / Diff.

### Tests

Build / Test 실행과 결과.

Terminal은 MVP 핵심 기능에서 제외하고 추후 Tools 화면으로 확장한다.

---

## 네트워크

```text
Android
   │
   │ Tailscale
   ▼
Desktop Agent
```

- Port Forwarding 불필요
- 공용 Internet Endpoint 불필요
- REST API로 상태/명령 전달
- WebSocket으로 실시간 이벤트 수신

---

## 보안 요구사항

- Desktop Token은 Android Keystore 등에 안전하게 저장
- Token 평문 파일 저장 금지
- Approval request ID 검증
- resolve된 Approval 재실행 금지
- Desktop Agent가 제공하지 않은 임의 명령 API 노출 금지
- Tailscale 외부 접속 기본 차단

---

## 백그라운드 동작

초기 버전은 항상 연결을 유지하려고 하지 않는다.

앱이 foreground일 때 WebSocket을 유지하고, background에서는 필요 시 연결을 종료할 수 있다.

향후 Push Notification을 추가하여 다음 이벤트를 알릴 수 있다.

- Approval required
- Codex task completed
- Test failed
- Agent error

---

## MVP

### Phase 1 — Remote Codex

- Desktop 연결
- 프로젝트 목록
- Thread 목록
- Thread Start / Resume
- Prompt 입력
- Agent Message Streaming
- Activity Timeline
- Turn Stop

### Phase 2 — Remote Approval

- Command Approval
- File Change Approval
- Permission Approval
- Accept / Accept for Session / Decline
- Pending Approval 복구

### Phase 3 — Review

- Git Status
- Diff Viewer
- Build
- Test
- 작업 완료 Summary

---

## Non-Goals

초기 버전에서는 다음을 목표로 하지 않는다.

- 완전한 모바일 IDE
- VS Code 원격 화면 복제
- 모든 Shell command를 자유롭게 실행하는 SSH client
- Codex App Server에 모바일이 직접 연결
- 공용 인터넷에 Desktop을 노출

---

## 향후 확장

- Push Notification
- 여러 Desktop
- 여러 Thread 병렬 관리
- Git commit / push UI
- GitHub Issue / PR 연동
- CI 확인
- 작업 Queue
- 음성 명령
- 파일 첨부
- 사진 첨부
- Codex 작업 요약
