# fullstack-app-builder-harness

**아이디어 한 줄 → 풀스택 앱 스캐폴드 자동 생성**

Claude Code 기반 멀티 에이전트 하네스. 자연어 아이디어 한 줄을 입력하면 기획서·아키텍처·백엔드/프론트엔드/테스트 코드·배포 문서까지 6종 산출물을 자동으로 만들어 냅니다.

---

## 개요

이 하네스는 다음 흐름으로 동작합니다.

```
아이디어 1줄  →  [STAGE 1] 기획·아키텍처  →  체크포인트 #1(기술 스택 선택)
             →  [STAGE 2] 코드 생성(병렬)  →  [STAGE 3] 리뷰·문서화
             →  체크포인트 #2(최종 검토)   →  완성된 풀스택 앱 스캐폴드
```

한 번의 `/build` 명령으로 아래 **6종 산출물**이 생성됩니다.

| # | 산출물 | 담당 agent |
|---|--------|-----------|
| 1 | 앱 기획서 (`plan.md`) | app-planner |
| 2 | 아키텍처 설계 (`arch.md`, `api-contract.json`) | arch-designer |
| 3 | 백엔드 코드 (`backend/`) | backend-writer |
| 4 | 프론트엔드 코드 (`frontend/`) | frontend-writer |
| 5 | 테스트 코드 (`tests/`) | test-writer |
| 6 | 배포 문서 (`docs/`, `README.md`) | doc-writer |

---

## 빠른 시작

```bash
# 1. 하네스 클론
git clone <repo-url>
cd fullstack-app-builder-harness

# 2. Claude Code로 실행
claude

# 3. 전체 파이프라인 실행
/build "팀 단위 할 일 관리 앱"
```

두 번의 체크포인트(기술 스택 선택, 최종 검토)에만 사용자 입력이 필요하고, 나머지는 자동으로 진행됩니다.

---

## 커맨드 레퍼런스

### `/build "앱 아이디어"`
전체 파이프라인(STAGE 1~3)을 실행합니다. 체크포인트 2회에서만 사용자 입력을 요청합니다.

- **인자**: 자연어 앱 아이디어 (한 줄)
- **전제조건**: 없음
- **동작**:
  1. STAGE 1 — idea-intake → app-planner → arch-designer 순차 실행
  2. 체크포인트 #1 — 기술 스택 후보 제시 및 사용자 선택 대기
  3. STAGE 2 — backend-writer / frontend-writer / test-writer 병렬 실행
  4. STAGE 3 — code-reviewer → doc-writer 순차 실행
  5. 체크포인트 #2 — 최종 산출물 검토 및 승인 대기
- **산출물**: `.app-artifacts/{session-id}/` 하위 전체

---

### `/plan-app "앱 아이디어"`
STAGE 1(기획 + 아키텍처)만 실행하고 체크포인트 #1에서 정지합니다.

- **인자**: 자연어 앱 아이디어 (한 줄)
- **전제조건**: 없음
- **동작**: idea-intake → app-planner → arch-designer → 체크포인트 #1(기술 스택 선택) 후 정지
- **산출물**: `intake.json`, `plan.md`, `arch.md`, `api-contract.json`
- **다음 단계**: 기술 스택 승인 후 `/generate`로 코드 생성

---

### `/generate`
STAGE 2(코드 생성)만 실행합니다.

- **인자**: 없음
- **전제조건**: 승인된 `arch.md`와 `api-contract.json`이 존재해야 합니다 (`/plan-app` 선행 필요)
- **동작**: backend-writer / frontend-writer / test-writer를 병렬 실행
- **산출물**: `backend/`, `frontend/`, `tests/`

---

### `/export {target-dir}`
생성된 코드를 지정한 디렉토리로 내보냅니다.

- **인자**: 복사할 대상 디렉토리 경로
- **전제조건**: STAGE 2 이상 완료 상태
- **동작**: `backend/`, `frontend/`, `tests/`, `docs/`, `README.md`를 `target-dir`로 복사 (`cp -r`)
- **예시**: `/export ~/projects/my-todo-app`

---

## 파이프라인 흐름

```
                      /build "앱 아이디어 한 줄"
                                │
                                ▼
                    ┌───────────────────────┐
                    │   orchestrator-agent   │  (Opus, 메인 파이프라인 진행자)
                    └───────────┬───────────┘
                                │
╔═════════════════ STAGE 1 (sequential) ════════════════╗
║                               │                        ║
║  ┌──────────────┐  ┌──────────────┐  ┌─────────────┐  ║
║  │ idea-intake  │→ │ app-planner  │→ │arch-designer│  ║
║  │  (Opus)      │  │  (Opus)      │  │  (Opus)     │  ║
║  │ intake.json  │  │ plan.md      │  │ arch.md     │  ║
║  └──────────────┘  └──────────────┘  └──────┬──────┘  ║
║                                              │         ║
║         ◆ 체크포인트 #1: 기술 스택 선택 ◆            ║
╚══════════════════════════════╪════════════════════════╝
                                │ (승인 후)
╔═════════════════ STAGE 2 (parallel) ══════════════════╗
║  orchestrator가 mkdir -p로 출력 디렉토리 사전 생성     ║
║                               │                        ║
║          ┌────────────────────┼───────────────────┐   ║
║          ▼                    ▼                   ▼   ║
║  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐║
║  │backend-writer│   │frontend-     │   │ test-writer  │║
║  │  (Sonnet)    │   │writer(Sonnet)│   │  (Sonnet)    │║
║  │ backend/     │   │ frontend/    │   │ tests/       │║
║  └──────────────┘   └──────────────┘   └──────────────┘║
╚══════════════════════════════╪════════════════════════╝
                                │ (3개 모두 완료 후)
╔═════════════════ STAGE 3 (sequential) ════════════════╗
║                               │                        ║
║           ┌──────────────┐    ┌──────────────┐         ║
║           │ code-reviewer│ →  │  doc-writer  │         ║
║           │  (Opus)      │    │  (Sonnet)    │         ║
║           │ review.md    │    │ docs/,README │         ║
║           └──────────────┘    └──────┬───────┘         ║
║                                      │                  ║
║           ◆ 체크포인트 #2: 최종 산출물 검토·승인 ◆     ║
╚══════════════════════════════╪════════════════════════╝
                                │
                                ▼
                      완성된 풀스택 앱 스캐폴드
              (.app-artifacts/{session-id}/ 하위 전체)
```

---

## 산출물 구조

`/build "간단한 할 일 관리 앱"` 실행 후 생성되는 파일 트리 예시입니다.

```
.app-artifacts/20260601-todo-app/
├── intake.json
├── plan.md
├── arch.md
├── api-contract.json
├── backend/
│   ├── src/
│   │   ├── models/
│   │   │   └── Task.js
│   │   ├── routes/
│   │   │   └── tasks.js
│   │   └── services/
│   │       └── taskService.js
│   ├── app.js
│   └── package.json
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── TaskList.jsx
│   │   │   └── TaskItem.jsx
│   │   ├── pages/
│   │   │   └── Home.jsx
│   │   └── api/
│   │       └── tasks.js
│   └── package.json
├── tests/
│   ├── backend/
│   │   ├── unit/
│   │   │   └── taskService.test.js
│   │   └── integration/
│   │       └── tasks.api.test.js
│   └── frontend/
│       └── TaskList.test.jsx
├── review.md
├── docs/
│   ├── Dockerfile.backend
│   ├── Dockerfile.frontend
│   ├── docker-compose.yml
│   ├── .github/
│   │   └── workflows/
│   │       └── ci.yml
│   ├── .env.example
│   └── DEPLOY.md
└── README.md
```

---

## 체크포인트 설명

### 체크포인트 #1 — 기술 스택 선택

STAGE 1(기획·아키텍처) 완료 직후 나타납니다. arch-designer가 **기술 스택 후보 2~3개**를 제시하며, 사용자가 선택한 스택으로 api-contract.json을 확정한 뒤 STAGE 2가 시작됩니다.

```
[체크포인트 #1] 기술 스택을 선택해 주세요:

  A. Node.js/Express + React + PostgreSQL
  B. Python/FastAPI + Vue + PostgreSQL
  C. Node.js/Express + Next.js + PostgreSQL

  ※ 미설치 도구 경고: docker가 감지되지 않았습니다.
     코드·문서 생성은 정상 진행되며, 로컬 실행 시 설치가 필요합니다.

선택 (A/B/C):
```

잘못된 기술 스택으로 코드를 생성해 낭비하지 않도록, 이 단계에서 반드시 스택을 확정합니다.

---

### 체크포인트 #2 — 최종 검토

STAGE 3(리뷰·문서화) 완료 직후 나타납니다. code-reviewer의 검토 결과(`review.md` 요약)와 생성된 전체 파일 목록을 제시합니다.

```
[체크포인트 #2] 최종 산출물을 확인해 주세요:

  리뷰 결과: BLOCKER 0건 / WARN 2건 / INFO 5건
  생성 파일: 총 35개 (backend 12, frontend 15, tests 8)
  배포 문서: docs/ (Dockerfile, docker-compose, CI/CD, .env.example, DEPLOY.md)

  WARN 항목:
    - [WARN] Task.dueDate 필드가 frontend에서 미사용 (frontend/src/api/tasks.js:23)
    - [WARN] .env.example에 DB_SSL 항목 누락

  승인하시겠습니까? (yes/no):
  승인 후 /export {target-dir} 로 코드를 내보낼 수 있습니다.
```

BLOCKER가 0건이면 승인을 권장합니다. WARN/INFO는 승인 후 수동 수정해도 무방합니다.

---

## 출력 예시 (샘플 실행 결과)

```
$ /build "간단한 할 일 관리 앱"

[STAGE 1] 기획 중...
✓ intake.json 생성 (앱 유형: web, 핵심 엔티티: User, Task)
✓ plan.md 생성 (요구사항 12개, 사용자 스토리 10개, MVP In-scope 8개)
✓ arch.md 생성

[체크포인트 #1] 기술 스택을 선택해 주세요:
  A. Node.js/Express + React + PostgreSQL
  B. Python/FastAPI + Vue + PostgreSQL
→ 사용자 선택: A

✓ api-contract.json 확정 (모델 2개, 엔드포인트 8개)

[STAGE 2] 코드 생성 중 (병렬)...
✓ backend/  생성 완료 (12개 파일)
✓ frontend/ 생성 완료 (15개 파일)
✓ tests/    생성 완료 (8개 파일)

[STAGE 3] 검토 및 문서화...
✓ review.md: BLOCKER 0건 / WARN 2건 / INFO 5건
✓ docs/ 생성 완료 (Dockerfile ×2, docker-compose.yml, ci.yml, .env.example, DEPLOY.md)
✓ README.md 생성 완료

[체크포인트 #2] 최종 산출물 확인...
  총 35개 파일 생성 | .app-artifacts/20260601-todo-app/
  승인 후 /export ~/projects/todo-app 으로 내보낼 수 있습니다.
→ 사용자 승인: yes

완료. 산출물 경로: .app-artifacts/20260601-todo-app/
```

### 샘플 api-contract.json (할 일 관리 앱)

```json
{
  "session_id": "20260601-todo-app",
  "stack": { "backend": "node-express", "frontend": "react", "db": "postgresql" },
  "models": [
    {
      "name": "Task",
      "fields": [
        { "name": "id",        "type": "uuid",      "pk": true },
        { "name": "title",     "type": "string",    "required": true },
        { "name": "done",      "type": "boolean",   "default": false },
        { "name": "createdAt", "type": "timestamp", "auto": true }
      ]
    }
  ],
  "endpoints": [
    { "method": "GET",    "path": "/api/tasks",     "returns": "Task[]" },
    { "method": "POST",   "path": "/api/tasks",     "body": "TaskCreate",  "returns": "Task" },
    { "method": "PATCH",  "path": "/api/tasks/:id", "body": "TaskUpdate",  "returns": "Task" },
    { "method": "DELETE", "path": "/api/tasks/:id", "returns": "void" }
  ]
}
```

---

## 트러블슈팅

### `node` / `python3` / `docker`가 설치되지 않은 경우

**원인**: 런타임 도구가 시스템에 없어도 하네스 실행 자체는 가능합니다. 파이프라인은 파일 생성만 수행하므로 코드 생성은 정상적으로 완료됩니다. 단, 생성된 코드를 **로컬에서 실행**하려면 해당 도구가 필요합니다.

**해결책**:
- Node.js: https://nodejs.org/ 에서 LTS 버전 설치
- Python3: https://www.python.org/downloads/ 또는 `brew install python3`
- Docker: https://docs.docker.com/get-docker/ 에서 Docker Desktop 설치

orchestrator는 파이프라인 시작 시 자동으로 도구 유무를 확인하고, 체크포인트 #1에서 미설치 항목을 경고로 안내합니다.

---

### `api-contract.json`이 없는 상태에서 `/generate` 실행

**원인**: `/generate`는 STAGE 2(코드 생성)만 수행하며, 승인된 `arch.md`와 `api-contract.json`이 사전에 존재해야 합니다. 이 파일 없이 실행하면 아래 오류가 발생합니다.

```
오류: api-contract.json을 찾을 수 없습니다.
      먼저 /plan-app "앱 아이디어" 를 실행하고 체크포인트 #1에서 기술 스택을 승인하세요.
```

**해결책**: `/plan-app "앱 아이디어"`를 먼저 실행하고 체크포인트 #1에서 기술 스택을 선택·승인한 후 `/generate`를 실행하세요.

---

### `.app-artifacts/` 디렉토리 권한 오류

**원인**: 하네스 실행 루트의 `.app-artifacts/` 디렉토리에 쓰기 권한이 없으면 파일 생성에 실패합니다.

**해결책**:
```bash
chmod 755 .app-artifacts
# 또는 디렉토리가 없는 경우
mkdir -p .app-artifacts && chmod 755 .app-artifacts
```

---

### backend/frontend 코드와 api-contract.json 불일치 (code-reviewer BLOCKER)

**원인**: STAGE 2 생성 중 예외가 발생하거나, 수동 편집으로 인해 생성된 코드가 `api-contract.json`의 엔드포인트·모델 정의와 어긋날 수 있습니다. code-reviewer가 이를 BLOCKER로 분류합니다.

**해결책**:
1. **자동 재생성**: `/generate`를 재실행하여 STAGE 2 전체를 다시 수행합니다. api-contract.json을 기준으로 코드가 재생성됩니다.
2. **수동 수정**: review.md의 BLOCKER 항목을 참고하여 해당 파일을 직접 수정합니다. 수정 후 `/review-code`로 재검증할 수 있습니다.

---

### STAGE 2 도중 중단 후 재시작

**원인**: 네트워크 오류·Claude Code 재시작 등으로 STAGE 2(코드 생성) 중 파이프라인이 중단될 수 있습니다.

**해결책**: `/generate`를 재실행하세요. orchestrator가 `stage=2` 모드로 backend/frontend/test-writer를 다시 병렬 호출하여 STAGE 2를 재수행합니다. STAGE 1 산출물(`arch.md`, `api-contract.json`)은 그대로 유지되므로 기획·아키텍처를 다시 생성할 필요가 없습니다.

> 참고: 이미 생성된 파일이 있어도 writer agent가 덮어씁니다.

---

## 라이선스 / 기여

이 하네스는 **MIT License** 하에 공개됩니다. 자유롭게 사용·수정·배포할 수 있습니다.

버그 리포트, 기능 제안, 풀 리퀘스트 모두 환영합니다. 기여 전 `plan.md`의 파이프라인 규칙과 `docs/harness-patterns.yaml`의 검증된 패턴을 참고하세요.
