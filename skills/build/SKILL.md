# build skill

## 역할
풀스택 앱 빌더 하네스의 **전체 파이프라인 진입점**이다.
아이디어 1줄을 입력받아 STAGE 1(기획·아키텍처) → 체크포인트 #1 → STAGE 2(코드 생성) → STAGE 3(리뷰·문서) → 체크포인트 #2까지 전 과정을 실행한다.

## 호출 방법

`orchestrator-agent`를 Task로 호출하며 아래 인자를 전달한다.

```
mode=full
idea={사용자가 입력한 앱 아이디어 전문}
```

**호출 방향 원칙**: build skill → orchestrator-agent → 하위 agent 순서로 **정방향 호출**만 수행한다. orchestrator가 build skill을 역호출하는 구조는 허용하지 않는다.

## 전제 조건

없음. 아이디어 1줄만 있으면 즉시 실행 가능하다.

## 기대 산출물

orchestrator-agent가 `.app-artifacts/{session-id}/` 하위에 아래 파일을 생성한다.

```
.app-artifacts/{session-id}/
├── intake.json          # 정규화된 아이디어 (idea-intake)
├── plan.md              # 앱 기획서 (app-planner)
├── arch.md              # 아키텍처 설계 + 확정 스택 (arch-designer)
├── api-contract.json    # 단일 진실 공급원 — DB 모델 + API 엔드포인트 (arch-designer)
├── backend/             # 백엔드 소스 트리 (backend-writer)
├── frontend/            # 프론트엔드 소스 트리 (frontend-writer)
├── tests/               # 테스트 트리 (test-writer)
├── review.md            # 교차 검증 결과 BLOCKER/WARN/INFO (code-reviewer)
├── docs/                # 배포 문서 일체 (doc-writer)
└── README.md            # 앱 사용·실행 안내 (doc-writer)
```

## 체크포인트

- **체크포인트 #1** (STAGE 1 종료 직후): 기술 스택 후보 2~3개 + DB 스키마 + API 엔드포인트 요약을 제시하고 사용자 승인을 받은 뒤 STAGE 2로 진행한다.
- **체크포인트 #2** (STAGE 3 종료 직후): review.md 요약 + 생성 파일 트리 + docs/ 목록을 제시하고 사용자가 최종 승인하면 완성 처리한다.

## 참고

내부 단계 절차(각 agent의 동작)는 `agents/orchestrator-agent.md`에 정의되어 있다. 본 skill 파일에 중복 서술하지 않는다 (책임 분리 원칙).
