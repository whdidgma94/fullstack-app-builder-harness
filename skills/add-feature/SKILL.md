`add-feature` skill을 실행한다.

## 역할

이미 `/build`로 생성된 앱에 **새 기능을 추가**하는 진입점이다. 기존 session(arch.md + api-contract.json 존재)을 대상으로, 자연어 기능 설명을 받아 feature-spec 계획 → 체크포인트 AF-1 → 기능별 코드 병렬 생성 → 보안·코드 리뷰 → 체크포인트 AF-2까지 실행한다.

## 전제 조건

- 대상 session에 `.app-artifacts/{session-id}/arch.md`와 `api-contract.json`이 존재해야 한다.
- 존재하지 않으면 먼저 `/build`를 실행한다.

## 호출

`orchestrator-agent`를 Task로 호출하며 아래 인자를 전달한다.

```
mode=add-feature
session_id={대상 앱 session-id, 미지정 시 orchestrator가 후보 제시}
feature_description={추가할 기능 자연어 설명 전문}
```

**호출 방향**: add-feature skill → orchestrator-agent → 하위 agent (정방향 호출만 수행).

## 기대 산출물

```
.app-artifacts/{session-id}/
├── api-contract.json                          # 신규 엔드포인트·모델 APPEND 반영 (기존 항목 보존)
└── features/{feature-slug}/
    ├── feature-spec.json                      # feature-planner
    ├── backend/                               # backend-writer (feature 모드)
    ├── frontend/                              # frontend-writer (feature 모드)
    ├── tests/                                 # test-writer (feature 모드)
    ├── review.md                              # code-reviewer (feature 범위)
    └── security.md                            # security-reviewer (feature 범위)
```

## 체크포인트

- **AF-1** (feature-planner 직후): feature_slug + 신규 엔드포인트/모델/컴포넌트 건수 + 기존 파일 수정 필요 항목을 제시하고 승인받는다.
- **AF-2** (리뷰 직후): 생성 파일 트리 + review/security 요약 + 수동 통합 필요 항목을 제시하고 최종 승인받는다.

내부 단계 절차는 `agents/orchestrator-agent.md`의 "ADD-FEATURE 모드" 섹션에 정의되어 있다.
