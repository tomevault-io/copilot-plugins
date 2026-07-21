## gsghwp

> 이 저장소를 설치·업데이트·제거해 달라는 요청을 받으면 이 문서와 `README.md`를 먼저 읽는다. 레지스트리를 직접 편집하거나 DLL을 임의 경로에 복사하지 않고 저장소의 스크립트만 사용한다.

# GSG HWP 설치 에이전트 지침

이 저장소를 설치·업데이트·제거해 달라는 요청을 받으면 이 문서와 `README.md`를 먼저 읽는다. 레지스트리를 직접 편집하거나 DLL을 임의 경로에 복사하지 않고 저장소의 스크립트만 사용한다.

## 변경 전 사용자 안내

실제 설치 전에 다음 내용을 사용자에게 분명히 알린다.

1. `%LOCALAPPDATA%\HancomDocumentAutomation\native\0.5.51\HancomLiveBridge.dll`이 복사 또는 교체된다.
2. `%LOCALAPPDATA%\GSG_HWP\security\FilePathCheckerModule.dll`이 복사 또는 교체된다.
3. HKCU의 다음 세 값이 변경된다.
   - `Software\HNC\HwpUserAction\Modules`
   - `Software\HNC\HwpUserAction\Modules\Uses`
   - `Software\HNC\HwpAutomation\Modules`의 `FilePathCheckerModule`
4. 기존 DLL 2개와 레지스트리 값의 존재 여부·종류·값은 `%LOCALAPPDATA%\GSG_HWP\backups`에 먼저 저장된다.
5. `uninstall.ps1 -AcceptChanges`로 설치 전 상태를 복원할 수 있다.
6. HKLM, `regsvr32`, 관리자 권한은 사용하지 않는다.

## 설치 절차

1. Windows, 한/글 2024, Git을 확인한다.
2. 한/글 프로세스가 실행 중이면 저장을 안내하고 모두 종료한다. 설치 목적으로 사용자의 열린 문서를 수정하거나 테스트 문서를 새로 열지 않는다.
3. 저장소 루트에서 `install.ps1`을 옵션 없이 실행한다. 종료 코드 2는 변경 없이 미리보기만 마친 정상 결과다.
4. 위 변경·백업·복원 내용을 안내했고 사용자가 설치를 요청한 상태라면 `install.ps1 -AcceptChanges`를 실행한다.
5. `uv`가 없다는 오류가 나면 사용자에게 알리고 `winget install --id astral-sh.uv -e`로 설치한 뒤 다시 실행한다. 설치기는 잠금된 `pyhwpx` 환경에서 파일 경로 보안 DLL을 가져와 해시를 검증한다.
6. 대상 앱에 맞게 MCP를 등록한다.

### Codex

```powershell
codex plugin marketplace add innae1121-bit/gsghwp --ref main
codex plugin add gsg-hwp@gsg-hwp
```

### Claude Code

저장소 루트에서 시작 스크립트의 절대 경로를 구해 사용자 범위 stdio MCP로 등록한다.

```powershell
$startMcp = (Resolve-Path ".\plugins\gsg-hwp\scripts\start-mcp.ps1").Path
claude mcp add --transport stdio --scope user gsg-hwp -- `
  powershell.exe -NoLogo -NoProfile -NonInteractive -ExecutionPolicy Bypass -File $startMcp
```

### 다른 MCP 클라이언트

`powershell.exe -NoLogo -NoProfile -NonInteractive -ExecutionPolicy Bypass -File <절대 경로>\plugins\gsg-hwp\scripts\start-mcp.ps1`을 stdio 서버로 등록한다.

7. 한/글과 에이전트 앱을 다시 시작하고 새 작업에서 `gsg-hwp` 연결과 `hwp_runtime_info`를 확인한다.

## 제거와 원상복구

1. 대상 앱에서 `gsg-hwp` MCP 또는 플러그인을 제거한다.
2. 한/글을 모두 종료한다.
3. `uninstall.ps1`을 옵션 없이 실행해 복원 예정 내용을 표시한다.
4. 안내 후 `uninstall.ps1 -AcceptChanges`를 실행한다.
5. 설치 전 네이티브/파일 경로 보안 DLL과 레지스트리 3개 값의 상태가 복원되었다는 스크립트 결과와 사용한 백업 파일 경로를 사용자에게 알린다.
6. 백업 폴더는 감사와 추가 복구를 위해 삭제하지 않는다.

Codex 제거 명령:

```powershell
codex plugin remove gsg-hwp@gsg-hwp
codex plugin marketplace remove gsg-hwp
```

Claude Code 제거 명령:

```powershell
claude mcp remove gsg-hwp
```

설치나 복원이 실패하면 수동 레지스트리 편집, `regsvr32`, 다른 DLL 복사 또는 UI 자동화로 우회하지 않는다. 오류와 백업 파일 경로를 보존해 사용자에게 보고한다.

## 기능 확장 요청

공개 도구에 원하는 기능이 없으면 `plugins/gsg-hwp/skills/automate-hancom-documents/resources/hancom_official_api_catalog_v1.json`과 1,448개 활성 네이티브 라우트를 먼저 조사한다. 대응 API가 있더라도 1,448개를 모두 개별 도구로 노출하지 않는다. 필요한 기능 하나를 작업 중심 도구 또는 recipe로 연결하고 대상·상태 검증, 결과 검수, 회귀 테스트와 `compatibility-manifest.json` 갱신을 함께 수행한다.

---
> Source: [innae1121-bit/gsghwp](https://github.com/innae1121-bit/gsghwp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
