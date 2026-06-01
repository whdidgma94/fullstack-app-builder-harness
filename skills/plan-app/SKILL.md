# plan-app skill

## 역할
**STAGE 1 전용** 스킬이다. 코드 생성 없이 아이디어 정규화 → 앱 기획서 → 아키텍처 설계까지만 수행하고, 체크포인트 #1에서 사용자 승인을 기다리며 정지한다.

## 호출 방법

`orchestrator-agent`를 Task로 호출하며 아래 인자를 전달한다.

```
mode=stage1
idea={사용자가 입력한 앱 아이디어 전문}
```

orchestrator-agent는 `mode=stage1` 인자를 받으면 `idea-intake → app-planner → arch-designer` 순서로만 실행하고, 체크포인트 #1 제시 후 사용자 응답을 대기한다. STAGE 2(코드 생성)로는 진행하지 않는다.

**호출 방향 원칙**: plan-app skill → orchestrator-agent → 하위 agent 순서로 **정방향 호출**만 수행한다.

## 전제 조건

없음. 아이디어 1줄만 있으면 즉시 실행 가능하다.

## 기대 산출물

STAGE 1이 완료되면 아래 파일이 `.app-artifacts/{session-id}/`에 생성된다.

```
.app-artifacts/{session-id}/
├── intake.json          # 정규화된 아이디어 (idea-intake)
├── plan.md              # 앱 기획서 (app-planner)
├── arch.md              # 아키텍처 설계 + 스택 후보 2~3개 (arch-designer)
└── api-contract.json    # 단일 진실 공급원 — DB 모델 + API 엔드포인트 (arch-designer)
```

## 체크포인트

- **체크포인트 #1**: arch-designer가 제시한 기술 스택 후보 2~3개, DB 스키마, API 엔드포인트 요약을 표시하고 스택 선택을 요청한다. 사용자가 승인하면 `/generate` 또는 `/build` 로 STAGE 2를 이어 실행한다.

## 참고

STAGE 2 코드 생성을 바로 이어서 실행하려면 `generate` skill(또는 `/generate` 커맨드)을 사용한다. 내부 단계 절차는 `agents/orchestrator-agent.md`에 정의되어 있다. 본 skill 파일에 중복 서술하지 않는다 (책임 분리 원칙).
