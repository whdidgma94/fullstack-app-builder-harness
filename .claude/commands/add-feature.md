`add-feature` skill을 실행한다.

이미 생성된 앱(arch.md + api-contract.json 존재)에 새 기능을 추가한다.
자연어 기능 설명을 입력받아 feature-spec 계획(체크포인트 AF-1) → 기능별 코드 병렬 생성 →
보안·코드 리뷰 → 최종 승인(체크포인트 AF-2)까지 실행한다.
신규 코드는 features/{feature-slug}/ 에 격리 생성되며 기존 파일은 수정하지 않는다.

ARGUMENTS: $ARGUMENTS
