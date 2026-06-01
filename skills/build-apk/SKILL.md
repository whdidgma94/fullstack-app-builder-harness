`build-apk` skill을 실행한다.

## 역할

생성된 프론트엔드 코드를 Capacitor로 래핑하여 Android APK를 생성한다. 빌드 도구(Java JDK, Android SDK)가 있으면 APK를 직접 빌드하고, 없으면 설정 파일과 빌드 스크립트를 제공한다.

## 전제 조건

- 대상 session에 `.app-artifacts/{session-id}/frontend/`가 존재해야 한다 (STAGE 2 완료 필요).
- 프론트엔드 스택이 React 또는 Vue여야 한다.

## 호출

`orchestrator-agent`를 Task로 호출하며 아래 인자를 전달한다.

```
mode=build-apk
session_id={대상 앱 session-id}
app_id={Android 패키지 ID, 선택}
app_name={앱 표시 이름, 선택}
api_url={백엔드 API URL, 선택}
```

## 기대 산출물

```
.app-artifacts/{session-id}/apk/
├── capacitor.config.json   # Capacitor 앱 설정
├── package.json            # Capacitor 의존성
├── .env.android            # API URL 환경변수
├── build-apk.sh            # 수동 빌드 스크립트
├── apk-build.md            # 빌드 결과 보고서
└── android/                # Android 프로젝트 (빌드 도구 있을 시)
    └── app/build/outputs/apk/debug/app-debug.apk
```

내부 빌드 절차는 `agents/apk-builder.md`에 정의되어 있다.
