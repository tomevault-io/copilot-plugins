## android-build

> 사용자 폴더가 한글(예: `그리고`)이면 터미널에서 경로가 깨질 수 있으므로 **반드시 `$env:USERPROFILE` 기반 경로**로 진행한다.

# Android 빌드 및 테스트 기기 설치 (한글 경로 대응)

사용자 폴더가 한글(예: `그리고`)이면 터미널에서 경로가 깨질 수 있으므로 **반드시 `$env:USERPROFILE` 기반 경로**로 진행한다.

## 1. 사전 조건

- **JAVA_HOME**: Android Studio JBR 사용. 미설정 시 아래 명령에서 설정.
- **Android Studio** 설치 및 SDK 설치 완료.
- 테스트 기기: USB 연결 또는 무선 디버깅(adb 연결) 완료.

## 2. 빌드 (PowerShell)

한 줄로 실행할 때:

```powershell
$env:JAVA_HOME = "C:\Program Files\Android\Android Studio\jbr"; $projectRoot = Join-Path $env:USERPROFILE "Desktop\MOVEIT"; $androidPath = Join-Path $projectRoot "android"; Set-Location $androidPath; .\gradlew.bat assembleDebug
```

- `$env:USERPROFILE` → `C:\Users\그리고` 등 한글 경로가 있어도 올바르게 해석됨.
- JAVA_HOME이 이미 시스템에 설정되어 있으면 앞의 `$env:JAVA_HOME = "..."` 생략 가능.
- 성공 시 APK 위치: `%USERPROFILE%\Desktop\MOVEIT\android\app\build\outputs\apk\debug\app-debug.apk`

## 3. 테스트 기기 설치 (PowerShell)

연결된 기기 한 대에 설치(덮어쓰기 `-r`):

```powershell
$adb = Join-Path $env:LOCALAPPDATA "Android\Sdk\platform-tools\adb.exe"; $projectRoot = Join-Path $env:USERPROFILE "Desktop\MOVEIT"; $apk = Join-Path $projectRoot "android\app\build\outputs\apk\debug\app-debug.apk"; & $adb install -r $apk
```

- 기기가 여러 대면 `& $adb -s <device_id> install -r $apk` 로 기기 지정.
- 기기 목록: `& $adb devices`

## 4. AAB 배포 (TotalManagements - Play 스토어 제출용)

한 줄로 실행할 때:

```powershell
$env:JAVA_HOME = "C:\Program Files\Android\Android Studio\jbr"; $projectRoot = Join-Path $env:USERPROFILE "Desktop\TotalManagements"; $androidPath = Join-Path $projectRoot "android"; Set-Location $androidPath; .\gradlew.bat bundleRelease
```

- 성공 시 AAB 위치: `%USERPROFILE%\Desktop\TotalManagements\android\app\build\outputs\bundle\release\app-release.aab`
- 스크립트 사용: `android\build-aab.ps1` 실행 (동일 경로 규칙 적용).

### 출시 모드 서명 (Play Console 업로드용)

디버그 서명 AAB는 업로드 불가. **업로드 키**로 서명해야 함. [앱 서명 가이드](https://developer.android.com/studio/publish/app-signing?hl=ko#generate-key) 참고.

1. **업로드 키 생성** (최초 1회): `android` 폴더에서 `.\setup-release-keystore.ps1` 실행 → 프롬프트에 키스토어/키 비밀번호, 이름 등 입력.
2. **keystore.properties 작성**: `keystore.properties.example` 을 복사해 `keystore.properties` 로 저장 후, `storePassword`, `keyPassword` 를 위에서 입력한 값으로 채움. `storeFile`, `keyAlias` 는 스크립트 기본값 그대로 사용 가능.
3. **AAB 재빌드**: `.\build-aab.ps1` → 생성된 AAB가 출시 서명으로 서명됨.

## 5. 주의사항

- **한글 경로 직접 사용 금지**: `cd "c:\Users\그리고\Desktop\MOVEIT"` 처럼 한글을 쓴 경로는 스크립트/셸에서 깨질 수 있음. 항상 `Join-Path $env:USERPROFILE "Desktop\..."` 형태 사용.
- **MoveitWebChromeClient** (MOVEIT): `Bridge` 참조는 부모 클래스가 private 이므로, 서브클래스에서 `private final Bridge bridge` 로 보관하고 `bridge.getWebView()` 등으로 사용한다. `getBridge()` 는 부모에 없음.
- 빌드 실패 시: `BUILD FAILED` 로그에서 `:app:compileDebugJavaWithJavac` 오류 위치를 확인하고, 위 Java 규칙을 지키는지 점검.

---
> Source: [tkaykim/totalmanagement](https://github.com/tkaykim/totalmanagement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
