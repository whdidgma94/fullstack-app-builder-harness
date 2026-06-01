# generate skill

## 역할
**STAGE 2 전용** 스킬이다. 이미 승인된 `arch.md`와 `api-contract.json`을 기반으로 백엔드·프론트엔드·테스트 코드를 **병렬**로 생성한다.

## 호출 방법

`orchestrator-agent`를 Task로 호출하며 아래 인자를 전달한다.

```
mode=stage2
session_id={session-id}
```

orchestrator-agent는 `mode=stage2` 인자를 받으면 병렬 호출 직전 `mkdir -p backend frontend tests`로 출력 디렉토리를 사전 생성한 뒤, `backend-writer`, `frontend-writer`, `test-writer`를 **동시에** Task로 호출한다. 3개 모두 완료되면 STAGE 3(리뷰·문서)로 넘어간다.

**호출 방향 원칙**: generate skill → orchestrator-agent → 하위 writer agent 순서로 **정방향 호출**만 수행한다.

## 전제 조건

아래 두 파일이 반드시 존재해야 한다.

- `.app-artifacts/{session-id}/arch.md` — 확정 기술 스택 포함
- `.app-artifacts/{session-id}/api-contract.json` — DB 모델 + API 엔드포인트 단일 진실 공급원

체크포인트 #1에서 사용자 승인이 완료된 상태여야 한다. `plan-app` skill 또는 `/plan-app` 커맨드로 STAGE 1을 먼저 실행해 위 파일을 생성한다.

## 기대 산출물

STAGE 2가 완료되면 아래 디렉토리가 `.app-artifacts/{session-id}/`에 생성된다.

```
.app-artifacts/{session-id}/
├── backend/             # 백엔드 소스 트리 (backend-writer, Sonnet)
├── frontend/            # 프론트엔드 소스 트리 (frontend-writer, Sonnet)
└── tests/               # 테스트 트리 — api-contract.json 기준 작성 (test-writer, Sonnet)
```

## 참고

- 병렬 agent 간 계약 불일치를 방지하기 위해 세 writer 모두 `api-contract.json`을 읽기 전용으로 참조한다. `test-writer`는 소스 파일을 직접 읽지 않는다.
- STAGE 2 완료 후 코드 리뷰가 필요하면 `review-code` skill을 사용한다.
- 내부 병렬 호출 절차는 `agents/orchestrator-agent.md`에 정의되어 있다. 본 skill 파일에 중복 서술하지 않는다 (책임 분리 원칙).
