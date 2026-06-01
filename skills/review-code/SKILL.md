# review-code skill

## 역할
`code-reviewer-agent`를 단독으로 실행하여 생성된 코드의 교차 검증 결과를 `review.md`로 산출한다. `/build` 파이프라인 내 STAGE 3에 포함되지만, 단독 실행도 가능하다 (재검토·디버깅 목적).

## 호출 방법

`code-reviewer-agent`를 Task로 **직접** 호출하며 아래 인자를 전달한다. orchestrator를 경유하지 않는다.

```
session_id={session-id}
```

`code-reviewer-agent`는 `.app-artifacts/{session-id}/` 하위의 `backend/`, `frontend/`, `tests/`, `api-contract.json`을 읽어 교차 검증을 수행하고 `review.md`를 작성한다.

**호출 방향 원칙**: review-code skill → code-reviewer-agent로 **직접 정방향 호출**한다.

## 전제 조건

아래 파일·디렉토리가 모두 존재해야 한다.

- `.app-artifacts/{session-id}/backend/` — 백엔드 소스 트리
- `.app-artifacts/{session-id}/frontend/` — 프론트엔드 소스 트리
- `.app-artifacts/{session-id}/tests/` — 테스트 트리
- `.app-artifacts/{session-id}/api-contract.json` — 단일 진실 공급원

STAGE 2(`generate` skill)가 완료된 뒤 실행한다.

## 기대 산출물

```
.app-artifacts/{session-id}/
└── review.md    # BLOCKER / WARN / INFO 심각도 라벨로 분류된 교차 검증 결과 (code-reviewer, Opus)
```

## 검증 항목 (code-reviewer-agent가 수행)

- `api-contract.json` 대비 엔드포인트 경로·메서드 일치 여부 (backend ↔ contract)
- `api-contract.json` 대비 API 클라이언트 호출 일치 여부 (frontend ↔ contract)
- DB 필드명·타입 일치 여부 (backend 모델 ↔ contract)
- 테스트 커버리지 — 엔드포인트별 테스트 존재 여부 (tests ↔ contract)
- 명명 일관성, 누락 파일, 명백한 결함

심각도 라벨은 **BLOCKER / WARN / INFO** 3종으로 고정한다.

## 참고

내부 검증 절차는 `agents/code-reviewer.md`에 정의되어 있다. 본 skill 파일에 중복 서술하지 않는다 (책임 분리 원칙).
