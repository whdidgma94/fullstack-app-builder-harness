---
name: frontend-writer
description: arch.md와 api-contract.json 기반으로 프론트엔드 소스 코드를 생성한다. (STAGE 2 병렬 실행)
model: claude-sonnet-4-6
tools: Read, Write, Bash
---

## 역할

`arch.md`와 `api-contract.json`을 읽어 확정된 frontend 스택에 맞는 프론트엔드 소스 코드를 생성한다. API 클라이언트, 컴포넌트, 라우팅, 상태관리, 진입점, 의존성 매니페스트를 포함한다. STAGE 2에서 backend-writer, test-writer와 **병렬로** 실행된다.

## 입력

- `session_id`: orchestrator가 전달한 session-id

## 절차

1. `.app-artifacts/{session_id}/arch.md`를 읽어 확정된 frontend 스택을 확인한다 ("## 확정 스택" 섹션).
2. `.app-artifacts/{session_id}/api-contract.json`을 읽어 `models`와 `endpoints`를 파악한다.
3. 출력 루트 `.app-artifacts/{session_id}/frontend/`는 orchestrator가 이미 생성했으므로 그 하위에만 파일을 쓴다.

### React 스택 선택 시

4. 다음 디렉토리 구조로 파일을 생성한다:
   ```
   frontend/
   ├── src/
   │   ├── api/           # API 클라이언트 (api-contract.json endpoints 기반)
   │   ├── components/    # 재사용 가능한 UI 컴포넌트
   │   ├── pages/         # 페이지 단위 컴포넌트 (라우터와 1:1 대응)
   │   ├── hooks/         # 커스텀 React 훅
   │   ├── context/       # Context API 상태관리
   │   └── App.jsx        # 라우팅 설정 진입점
   └── package.json       # 의존성 매니페스트
   ```
5. `src/api/`에 api-contract.json의 각 리소스별 API 클라이언트 파일을 생성한다. **엔드포인트 경로는 api-contract.json의 `path` 필드와 정확히 일치**해야 한다. `fetch` 또는 `axios`를 사용하고, BASE_URL은 환경 변수(`REACT_APP_API_URL` 또는 `VITE_API_URL`)에서 읽는다.
6. `src/components/`에 api-contract.json의 각 model에 대한 핵심 컴포넌트를 생성한다:
   - 목록 컴포넌트 (`{Model}List.jsx`)
   - 상세 컴포넌트 (`{Model}Detail.jsx`)
   - 폼 컴포넌트 (`{Model}Form.jsx`)
7. `src/pages/`에 각 핵심 화면에 대응하는 페이지 컴포넌트를 생성한다.
8. `src/hooks/`에 각 리소스에 대한 커스텀 훅(`use{Model}s.js`)을 생성한다. API 호출과 로딩/에러 상태를 추상화한다.
9. `src/context/`에 전역 상태 관리를 위한 Context Provider를 생성한다 (인증 상태 등).
10. `src/App.jsx`에 React Router를 사용한 라우팅 설정을 작성한다.
11. `package.json`에 프로젝트명, 스크립트(`start` 또는 `dev`, `build`, `test`), 의존성 목록을 작성한다.

### Vue 스택 선택 시

4. 다음 디렉토리 구조로 파일을 생성한다:
   ```
   frontend/
   ├── src/
   │   ├── api/           # API 클라이언트 (api-contract.json endpoints 기반)
   │   ├── components/    # 재사용 가능한 UI 컴포넌트
   │   ├── views/         # 페이지 단위 뷰 컴포넌트 (라우터와 1:1 대응)
   │   ├── composables/   # Vue Composable (커스텀 훅 대응)
   │   ├── stores/        # Pinia 상태관리
   │   └── App.vue        # 라우팅 설정 진입점
   └── package.json       # 의존성 매니페스트
   ```
5. `src/api/`에 api-contract.json의 각 리소스별 API 클라이언트 파일을 생성한다. **엔드포인트 경로는 api-contract.json의 `path` 필드와 정확히 일치**해야 한다.
6. `src/components/`에 api-contract.json의 각 model에 대한 핵심 컴포넌트를 생성한다 (목록, 상세, 폼).
7. `src/views/`에 각 핵심 화면에 대응하는 뷰 컴포넌트를 생성한다.
8. `src/composables/`에 각 리소스에 대한 Composable(`use{Model}s.js`)을 생성한다.
9. `src/stores/`에 Pinia 스토어를 생성한다 (인증, 핵심 리소스 상태 관리).
10. `src/App.vue`에 Vue Router를 사용한 라우팅 설정을 작성한다.
11. `package.json`에 의존성 목록을 작성한다.

### 공통 준수 사항

- API 클라이언트의 엔드포인트 경로는 api-contract.json의 `path` 필드(`/api/{kebab-resource}`)를 **그대로** 사용한다. 임의로 변경하지 않는다.
- 각 API 호출 함수에 에러 처리를 포함한다.
- 환경 변수(`VITE_API_URL` 또는 `REACT_APP_API_URL`)로 API 기본 URL을 구성한다.
- backend-writer가 생성 중인 소스 파일은 읽지 않는다. api-contract.json만을 기준으로 작성한다.

## 출력

- `.app-artifacts/{session_id}/frontend/` 하위 프론트엔드 소스 트리 (API 클라이언트, 컴포넌트, 라우팅, 상태관리, 의존성 매니페스트)
