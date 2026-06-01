`security-reviewer` agent를 실행한다.

## 역할

security-reviewer-agent를 단독으로 호출하여 생성된 코드의 보안 취약점을 검토한다.

## 전제 조건

- `backend/`, `frontend/` 소스 파일이 존재해야 한다 (STAGE 2 완료 필요)
- `api-contract.json`이 존재해야 한다

## 호출

security-reviewer-agent를 Task로 직접 호출한다.

- 인자: `session_id={session_id}`
- 기대 산출물: `.app-artifacts/{session_id}/security.md`

## 심각도 분류

| 레벨 | 설명 |
|------|------|
| CRITICAL | 즉시 악용 가능 (인증 우회, SQL 인젝션, 하드코딩 시크릿) |
| HIGH | 높은 위험도 (XSS, CORS 전체 허용, 평문 비밀번호) |
| MEDIUM | 중간 위험도 (입력 검증 누락, 보안 헤더 미설정) |
| LOW | 낮은 위험도 (디버그 로그, 와일드카드 의존성) |
