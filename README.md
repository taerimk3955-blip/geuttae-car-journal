# 그때

말로 설명하기 어려운 차량의 소리·진동과 발생 상황을 남겨 정비소에 전달하는 차량 증상 기록 앱입니다.

## 제출 파일

- Android release APK: [`apk/GEUTTAE_그때_release.apk`](apk/GEUTTAE_그때_release.apk)
- 전체 소스: [`GEUTTAE_Source.zip`](GEUTTAE_Source.zip) — 압축을 풀면 아래 개발 구조가 들어 있습니다.
- 설치 없이 보는 기존 웹 앱: [`GEUTTAE_App.html`](GEUTTAE_App.html) — 파일을 내려받아 브라우저에서 열 수 있습니다. 녹음은 브라우저 보안 정책에 따라 localhost 실행이 필요합니다.
- 기획·화면 자료: [`GEUTTAE_Panel_A2.png`](GEUTTAE_Panel_A2.png)

APK의 파일명 앞부분 `GEUTTAE`는 프로젝트 식별명입니다. 앱 ID는 `kr.geuttae.journal`, 버전은 `1.0.0`입니다. 이 저장소에는 실제 사용자의 기록, 개인정보가 입력된 신청서, APK 서명용 개인키를 포함하지 않습니다.

## 실행 방법

### Android APK

1. Android 10(API 29) 이상 기기에서 APK를 내려받아 설치합니다.
2. 내려받은 경로의 앱 설치 허용을 요청받으면 확인합니다.
3. ‘그때’를 실행합니다. 별도 서버나 실제 계정 없이 사용할 수 있습니다.
4. 녹음 시에만 마이크 권한을 요청합니다. 권한을 거부해도 텍스트로 기록할 수 있습니다.

기존 React 앱을 Android WebView에 포함해 실행합니다. 앱 화면과 글꼴은 APK 안에 들어 있어 네트워크 연결 없이도 열립니다. 녹음·사진 선택·파일 저장·인쇄/PDF는 Android 기능과 연결했습니다. 위치 및 차량 데이터 접근 권한은 요청하지 않습니다.

### 웹 소스 실행

소스 ZIP을 압축 해제한 폴더에서 Node.js 22 이상과 Python 3을 준비합니다.

```sh
npm ci
npm run build:web
npm run preview
```

브라우저에서 `http://localhost:4173`을 엽니다. `web/src/App.tsx`가 앱 화면과 기능, `web/src/main.tsx`가 로컬 저장 어댑터입니다. `web/index.html`은 빌드된 단일 HTML입니다.

### Android 재빌드

Android SDK platform 35, build-tools 35.0.0, JDK 17 이상을 준비합니다. APK에는 개발용 debug 서명 대신 프로젝트용 release 키를 사용했습니다. 키 파일과 비밀번호는 저장소에 올리지 않습니다. 재빌드할 때는 본인의 키를 환경변수로 지정합니다.

```sh
export ANDROID_HOME=/path/to/android-sdk
export GEUTTAE_KEYSTORE=/private/path/release.jks
export GEUTTAE_STORE_PASSWORD='your-store-password'
export GEUTTAE_KEY_PASSWORD='your-key-password'
export GEUTTAE_KEY_ALIAS=geuttae
export GEUTTAE_TEAM=GEUTTAE
npm ci
npm run build:web
bash scripts/build-apk.sh
```

`build-apk.sh`는 aapt2 → Java 컴파일 → d8 → zipalign → apksigner 순으로 release APK를 만듭니다. Gradle 프로젝트도 `android/`에 포함했습니다. Android Studio에서 열거나 해당 폴더에서 `./gradlew assembleRelease`를 실행할 수 있습니다. Gradle 빌드에도 같은 서명 환경변수가 필요합니다.

## 기능과 데이터 보관

- 차량 등록·수정, 증상 기록 추가·수정·삭제, 검색·유형 필터
- 음성 녹음, 사진 첨부, 녹음 재생
- 증상별 집계·반복 조건 확인
- 정비 질문과 답변, 정비 결과 메모·완료/재발 관리
- 정비 전달 보고서, 텍스트 저장, 인쇄/PDF, JSON 백업·가져오기
- 로컬 체험용 프로필과 마이페이지

기록과 프로필은 localStorage, 첨부 파일은 IndexedDB에 저장합니다. 앱을 삭제하거나 앱 데이터를 지우면 함께 삭제됩니다. JSON 백업은 기록과 첨부 메타데이터를 담으며 음성·사진 원본 자체는 포함하지 않습니다. 첨부 원본을 다른 기기로 복구하는 기능은 구현되지 않았습니다.

로그인 화면은 로컬 프로필 체험이며 실제 계정 인증·클라우드 동기화가 아닙니다. 기어·속도·외기온은 명시적으로 표시한 예시 데이터입니다. 고장 진단을 하지 않습니다. WebView에서 음성 인식 API를 제공하지 않으면 음성 받아쓰기 대신 직접 메모를 입력합니다.

## 테스트 환경과 필요한 설정

- 빌드: Linux x86_64, OpenJDK 21 런타임, Eclipse ECJ 3.38(Java 8 target), Android platform 35 / build-tools 35.0.0
- 웹: `npm run build:web` 및 TypeScript 타입 검사
- APK: 패키지·실행 Activity·최소 API·release debuggable 플래그 확인, zipalign 및 apksigner 서명 검증
- Android System WebView는 최신 버전을 권장합니다. 오디오 권한 및 시스템 파일 선택기가 필요합니다.
- 일반 Android 에뮬레이터 실행 시험 결과는 `TESTING.md`에 기록합니다. 실물 차량과 실기기 마이크 검증은 별도입니다.

### Pleos Connect 에뮬레이터 버전과 화면 규격

**Pleos Connect 에뮬레이터는 아직 사용하지 않았으며, 해당 버전과 차량 화면에서의 실행을 검증하지 않았습니다.** Android APK로 패키징한 현재 앱과 향후 Vehicle/STT/TTS SDK 연동 계획을 구분합니다. Pleos 검증을 완료했다고 주장하지 않습니다.

차량용 화면 시안은 1920×1080 가로 환경을 고려했고, 기존 모바일 화면은 390×844 기준으로 구성했습니다. 이는 UI 설계 기준이며 Pleos 에뮬레이터의 실제 테스트 규격을 뜻하지 않습니다. 실제 차량 화면·주행 중 조작 제한·차량 권한 연동은 후속 검증 범위입니다.

## 소스 구조

소스 ZIP을 압축 해제하면 다음 폴더가 있습니다.

- `web/src/`: 기존 앱의 React·TypeScript 소스
- `android/`: Android 호스트, 마이크 권한·선택 파일·내보내기·인쇄 연결
- `scripts/`: 웹 및 release APK 빌드 스크립트
- `server-source/`: 초기 웹 앱의 API·데이터베이스 구현. 기존 인증 환경과 D1/R2 바인딩이 필요한 별도 소스이며 APK의 실행 의존성이 아닙니다.
- `docs/`: 판넬, 로고, 라이선스

## 라이선스

앱에 포함된 SUIT 글꼴은 SIL Open Font License, React는 MIT, Lucide는 ISC 라이선스를 따릅니다. 관련 고지는 `THIRD_PARTY_NOTICES.md`와 소스 ZIP에 포함했습니다. 별도의 프로젝트 오픈소스 라이선스는 지정하지 않았습니다.
