# export skill

## 역할
`.app-artifacts/{session-id}/` 하위에 생성된 풀스택 앱 산출물을 사용자가 지정한 `target-dir`로 내보낸다. 파일을 복사(cp -r)하며 원본은 유지한다.

## 호출 방법

`orchestrator-agent`를 Task로 호출하며 아래 인자를 전달한다.

```
mode=export
session_id={session-id}
target={target-dir 절대 경로}
```

orchestrator-agent는 `mode=export` 인자를 받으면 Bash `cp -r`로 지정된 디렉토리와 파일을 `target-dir`로 복사한다. 삭제 작업은 수행하지 않는다.

**호출 방향 원칙**: export skill → orchestrator-agent 순서로 **정방향 호출**만 수행한다. 복사 절차(cp 명령)는 orchestrator-agent에 정의되어 있으며 본 skill에 중복 서술하지 않는다.

## 전제 조건

아래 조건이 모두 충족되어야 한다.

1. STAGE 2 완료 — `backend/`, `frontend/`, `tests/`가 존재
2. STAGE 3 완료 — `docs/`와 `README.md`가 존재
3. `target-dir`의 **상위 디렉토리**가 이미 존재 (orchestrator가 확인)

## 기대 산출물

`{target-dir}/` 하위에 아래 파일이 복사된다.

```
{target-dir}/
├── backend/         # 백엔드 소스 트리
├── frontend/        # 프론트엔드 소스 트리
├── tests/           # 테스트 트리
├── docs/            # 배포 문서 (Dockerfile, docker-compose.yml, ci.yml, .env.example, DEPLOY.md)
└── README.md        # 앱 사용·실행 안내
```

`intake.json`, `plan.md`, `arch.md`, `api-contract.json`, `review.md` 등 하네스 내부 산출물은 복사하지 않는다 (앱 코드와 무관한 메타 파일). 필요 시 인자로 포함 여부를 조정할 수 있다.

## 참고

내보내기 후 `target-dir`에서 앱을 실행하는 방법은 복사된 `README.md`를 참고한다. 내부 복사 절차는 `agents/orchestrator-agent.md`에 정의되어 있다. 본 skill 파일에 중복 서술하지 않는다 (책임 분리 원칙).
