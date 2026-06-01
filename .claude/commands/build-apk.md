`build-apk` skill을 실행한다.

생성된 프론트엔드 코드를 Capacitor로 래핑하여 Android APK를 빌드한다.
Java JDK와 Android SDK가 있으면 APK를 직접 생성하고,
없으면 설정 파일과 build-apk.sh 스크립트를 제공한다.
결과물은 .app-artifacts/{session-id}/apk/ 에 저장된다.

ARGUMENTS: $ARGUMENTS
