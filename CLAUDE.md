# fullstack-app-builder-harness

## 1. 하네스 개요

### 목적
**아이디어 1줄 → 풀스택 앱 스캐폴드 자동 생성.**
사용자가 자연어로 앱 아이디어 한 줄(예: "팀 단위 할 일 관리 앱")을 입력하면, orchestrator-agent가 기획서·아키텍처 설계·백엔드/프론트엔드/테스트 코드·배포 문서까지 6종 산출물을 순차·병렬 혼합 파이프라인으로 자동 생성한다.

### 대상 사용자
- 아이디어를 빠르게 실행 가능한 코드 스캐폴드로 전환하고 싶은 1인 개발자 / 인디 해커
- MVP 프로토타입을 단시간에 구성해야 하는 스타트업 팀
- 기획·설계·구현의 일관된 베이스라인을 자동으로 확보하려는 학습자

---

## 2. 산출물 경로

모든 런타임 산출물은 `.app-artifacts/{session-id}/` 하위에 저장된다.

```
.app-artifacts/{session-id}/       ← 예: .app-artifacts/20260601-todo-app/
├── intake.json                    ← idea-intake: 정규화된 아이디어
├── plan.md                        ← app-planner: 앱 기획서
├── arch.md                        ← arch-designer: 아키텍처 + 확정 스택
├── api-contract.json              ← arch-designer: 단일 진실 공급원
├── backend/                       ← backend-writer: 백엔드 소스 트리
│   ├── src/models/
│   ├── src/routes/
│   ├── src/services/
│   ├── app.js
│   └── package.json
├── frontend/                      ← frontend-writer: 프론트엔드 소스 트리
│   ├── src/components/
│   ├── src/pages/
│   ├── src/api/
│   └── package.json
├── tests/                         ← test-writer: 테스트 트리
│   ├── backend/unit/
│   ├── backend/integration/
│   └── frontend/
├── review.md                      ← code-reviewer: BLOCKER/WARN/INFO 분류 리뷰
├── security.md                    ← security-reviewer: CRITICAL/HIGH/MEDIUM/LOW 보안 취약점 리뷰
├── docs/                          ← doc-writer: 배포 문서
│   ├── Dockerfile.backend
│   ├── Dockerfile.frontend
│   ├── docker-compose.yml
│   ├── .github/workflows/ci.yml
│   ├── .env.example
│   └── DEPLOY.md
├── README.md                      ← doc-writer: 앱 사용·실행 설명서
└── features/                      ← add-feature 실행 시 생성 (선택적)
    └── {feature-slug}/            ← 기능별 격리 디렉토리 (예: user-auth, payment-module)
        ├── feature-spec.json      ← feature-planner: 신규 엔드포인트·모델·컴포넌트 스펙
        ├── backend/               ← backend-writer (feature 모드)
        ├── frontend/              ← frontend-writer (feature 모드)
        ├── tests/                 ← test-writer (feature 모드)
        ├── review.md              ← code-reviewer (feature 범위)
        └── security.md            ← security-reviewer (feature 범위)
```

> **런타임 산출물**은 하네스 *실행 시* 생성되며, 하네스 자체 파일(`output/` 트리)과 구분된다.

---

## 3. 파이프라인 규칙

1. **orchestrator-agent가 전체 파이프라인을 조율한다.** 모든 하위 agent 호출, 병렬 실행, 체크포인트 제시를 orchestrator가 담당한다.
2. **session-id는 orchestrator가 단 한 번 결정한다.** 형식: `{YYYYMMDD}-{app-slug}` (예: `20260601-todo-app`). 하위 agent는 session-id를 독립 생성하지 않고 인자로 전달받아 사용한다.
3. **STAGE 2 병렬 실행 직전, orchestrator가 `mkdir -p`로 출력 디렉토리를 사전 생성한다.** `backend/`, `frontend/`, `tests/` 디렉토리를 일괄 생성하여 병렬 writer 간 경쟁 조건을 차단한다.
4. **api-contract.json은 backend-writer·frontend-writer·test-writer 세 agent의 단일 진실 공급원이다.** arch-designer가 생성하고, 세 writer는 읽기 전용으로 참조한다. test-writer는 소스 파일을 직접 읽지 않고 api-contract.json 기준으로만 테스트를 작성한다.
5. **체크포인트 #1(아키텍처 승인) 이후에만 STAGE 2(코드 생성)가 실행된다.** 잘못된 기술 스택으로 코드를 낭비하지 않기 위해, arch-designer가 제시한 스택 후보를 사용자가 선택·승인해야 한다.
6. **체크포인트 #2(최종 검토)에서 사용자 승인 후 완성 처리된다.** code-reviewer의 review.md 요약(BLOCKER/WARN/INFO 건수)과 security-reviewer의 security.md 요약(CRITICAL/HIGH/MEDIUM/LOW 건수), 생성 파일 트리를 함께 제시한다.
7. **코드 리뷰 심각도는 BLOCKER / WARN / INFO 3종으로 고정한다.** code-reviewer, review.md, 체크포인트 #2 안내문 모두 동일 라벨을 사용한다.
9. **add-feature는 새 session-id를 만들지 않는다.** 기존 session-id를 입력으로 받아 `.app-artifacts/{session-id}/features/{feature-slug}/` 하위에 신규 코드를 격리 생성한다.
10. **api-contract.json은 add-feature 시 append-only다.** 신규 엔드포인트·모델을 배열 끝에 추가만 하며, 기존 항목은 절대 삭제·변경하지 않는다.
11. **add-feature의 신규 코드는 기존 파일을 덮어쓰지 않는다.** 기존 `backend/`, `frontend/`, `tests/` 파일을 읽거나 수정하지 않는다. 기존 파일 통합이 필요한 항목은 `INTEGRATION.md`와 체크포인트 AF-2에서 수동 통합 지시로만 제공한다.
12. **add-feature 체크포인트는 AF-1(feature-spec 승인)·AF-2(코드 최종 승인) 2회다.** doc-writer는 add-feature에서 호출하지 않는다.
8-1. **보안 리뷰 심각도는 CRITICAL / HIGH / MEDIUM / LOW 4종으로 고정한다.** security-reviewer, security.md, 체크포인트 #2 안내문 모두 동일 라벨을 사용한다. OWASP Top 10 기준으로 인증·인젝션·민감 데이터·설정 오류·입력 검증·의존성 보안을 검토한다.
8. **harness-reviewer는 메타 하네스(`custom_harness_maker`) 측 `agents/harness-reviewer.md`를 reuse한다.** 이 하네스 `output/agents/`에는 `harness-reviewer.md`를 두지 않는다.

---

## 4. 공유 데이터 계약

### session-id 형식
```
{YYYYMMDD}-{app-slug}
```
- 예: `20260601-todo-app`, `20260615-budget-saas`
- `app-slug`는 idea-intake가 아이디어에서 추출한 kebab-case 슬러그
- orchestrator-agent가 파이프라인 시작 시 단 한 번 결정하고 모든 하위 agent에 인자로 전달

### artifacts root 경로 규약
```
.app-artifacts/{session-id}/
```
하네스 실행 루트 기준의 상대 경로. 모든 중간·최종 산출물이 이 경로 하위에 위치한다.

### api-contract.json 스키마
arch-designer가 생성하고, backend/frontend/test-writer 세 agent가 공통으로 참조하는 핵심 계약 파일.

```json
{
  "session_id": "20260601-todo-app",
  "stack": {
    "backend": "node-express",
    "frontend": "react",
    "db": "postgresql"
  },
  "models": [
    {
      "name": "Task",
      "fields": [
        { "name": "id",    "type": "uuid",    "pk": true },
        { "name": "title", "type": "string",  "required": true },
        { "name": "done",  "type": "boolean", "default": false }
      ]
    }
  ],
  "endpoints": [
    { "method": "GET",  "path": "/api/tasks", "returns": "Task[]" },
    { "method": "POST", "path": "/api/tasks", "body": "TaskCreate", "returns": "Task" }
  ]
}
```

### 명명 규약 (단일 확정)
| 대상 | 규약 | 예시 |
|------|------|------|
| 모델명 | PascalCase | `Task`, `UserProfile` |
| API 엔드포인트 경로 | `/api/{kebab-resource}` | `/api/tasks`, `/api/user-profiles` |
| session-id / app-slug | kebab-case | `todo-app`, `budget-saas` |

---

## 5. 커맨드

```
/build "앱 아이디어"                      ← 전체 파이프라인 실행 (STAGE 1~3 + 체크포인트 2회)
/plan-app "앱 아이디어"                  ← STAGE 1만 실행 (기획 + 아키텍처 + 체크포인트 #1에서 정지)
/generate                               ← STAGE 2만 실행 (승인된 arch.md/api-contract.json 필요)
/export {target-dir}                    ← 생성된 코드를 target-dir로 내보내기
/security-review {session_id}           ← 보안 취약점 단독 검토 (security.md 생성)
/add-feature {session_id} "기능 설명"   ← 기존 앱에 새 기능 추가 (체크포인트 AF-1·AF-2)
```

### 사용 예시

```bash
# 전체 파이프라인 실행
/build "팀 단위 할 일 관리 앱"

# 기획·아키텍처만 먼저 확인
/plan-app "간단한 가계부 SaaS"

# 아키텍처 승인 후 코드만 생성
/generate

# 생성된 코드를 프로젝트 폴더로 내보내기
/export ~/projects/my-todo-app
```

---

## 6. harness-reviewer 참조

harness-reviewer는 메타 하네스(`custom_harness_maker`) 측 `agents/harness-reviewer.md`를 reuse한다. 이 하네스 `output/agents/`에는 `harness-reviewer.md`를 두지 않는다.

harness-reviewer는 `/build` 파이프라인 완료 후 격리된 세션에서 독립 리뷰를 수행하며, 체크포인트 #2에서 사용자에게 결과를 제시할 때 review.md와 함께 harness-reviewer 리뷰 요약도 포함한다.
