## lyricsslide-creator

> 이 프로젝트는 PyQt6로 구축된 Python GUI 애플리케이션으로, 사용자가 노래 가사로 PowerPoint 프레젠테이션을 생성할 수 있도록 돕기 위해 설계되었습니다. 사용자는 Melon.com (대한민국의 온라인 음악 서비스)에서 노래를 검색하고, 가사와 앨범 아트를 검색하고, 가사를 편집한 다음, 제공된 템플릿을 기반으로 가사를 슬라이드 형식으로 구성한 `.pptx` 파일을 생성할 수 있습니다. 이 애플리케이션은 PyInstaller를 사용하여 Windows용 독립 실행형 실행 파일로 묶을 수 있습니다.

# Lyrics Slide Creator 프로젝트 개요

이 프로젝트는 PyQt6로 구축된 Python GUI 애플리케이션으로, 사용자가 노래 가사로 PowerPoint 프레젠테이션을 생성할 수 있도록 돕기 위해 설계되었습니다. 사용자는 Melon.com (대한민국의 온라인 음악 서비스)에서 노래를 검색하고, 가사와 앨범 아트를 검색하고, 가사를 편집한 다음, 제공된 템플릿을 기반으로 가사를 슬라이드 형식으로 구성한 `.pptx` 파일을 생성할 수 있습니다. 이 애플리케이션은 PyInstaller를 사용하여 Windows용 독립 실행형 실행 파일로 묶을 수 있습니다.

## 주요 기술

*   **Python**: 주 프로그래밍 언어.
*   **PyQt6**: 그래픽 사용자 인터페이스 구축에 사용됩니다.
*   **requests & beautifulsoup4**: Melon.com에서 노래 정보, 가사 및 앨범 아트를 웹 스크래핑하는 데 사용됩니다.
*   **python-pptx**: PowerPoint 프레젠테이션을 프로그래밍 방식으로 생성하고 조작하는 데 사용됩니다.
*   **PyInstaller**: 애플리케이션을 Windows용 독립 실행형 실행 파일로 패키징하는 데 사용됩니다.
*   **Pillow**: 특히 앨범 아트 처리를 위해 이미지 처리에 사용됩니다.

## 아키텍처

애플리케이션은 모듈식 구조를 따릅니다:

*   **진입점 (Entry Points)**:
    *   `run.py`: 개발 및 직접 실행을 위한 주요 진입점입니다. 전역 예외 처리를 설정합니다.
    *   `src/main.py`: PyInstaller로 애플리케이션이 번들될 때의 진입점이며, 동결된 실행 파일에 대한 특정 리소스 경로를 처리합니다.
*   **사용자 인터페이스 (UI)**:
    *   `src/ui/main_window.py`: 기본 애플리케이션 창(`QMainWindow`)을 정의합니다. 노래 검색 페이지와 가사 편집 페이지 간을 전환하며 애플리케이션의 UI 상태를 관리합니다. `QThread`를 사용하여 비동기 작업을 위해 백엔드 로직과의 상호 작용을 조정합니다.
    *   `src/ui/style.py`: PyQt6 애플리케이션의 현대적이고 깔끔한 시각적 테마를 위한 QSS(Quantum Style Sheets) 정의를 포함합니다.
*   **유틸리티/백엔드 로직**:
    *   `src/utils/melon_api.py`: Melon.com에서 노래 검색, 가사 검색 및 앨범 아트 URL을 포함하여 데이터를 가져오는 `MelonScraper` 클래스를 구현합니다.
    *   `src/utils/ppt_engine.py`: PowerPoint 프레젠테이션 생성을 위한 핵심 로직을 처리하는 `PPTBuilder` 클래스를 구현합니다. 여기에는 가사를 슬라이드 크기 청크로 분할하고, 템플릿을 적용하고, 글꼴 스타일을 유지하고, 프레젠테이션을 출력하는 것이 포함됩니다.
    *   `src/utils/helpers.py`: `resource_path`와 같은 유틸리티 함수를 제공하여 애플리케이션이 PyInstaller 번들 실행 파일로 실행될 때 리소스에 대한 올바른 파일 경로를 보장합니다.

## 빌드 및 실행

### 종속성

이 프로젝트에는 Python 3.x 및 `requirements.txt`에 지정된 라이브러리가 필요합니다.

### 설치

1.  **가상 환경 생성 및 활성화**:
    ```bash
    python -m venv venv
    # Windows:
    .\venv\Scripts\activate
    # macOS/Linux:
    source venv/bin/activate
    ```
2.  **필수 라이브러리 설치**:
    ```bash
    pip install -r requirements.txt
    ```

### 실행 (개발 모드)

소스 코드에서 애플리케이션을 직접 실행하려면:

```bash
python run.py
```

### 빌드 (Windows 실행 파일)

PyInstaller를 사용하여 Windows용 독립 실행형 `.exe` 파일을 생성하려면:

```bash
build_windows.bat
```

이 스크립트는 `dist` 폴더에 `LyricsSlideCreator.exe`를 생성합니다.

## 개발 컨벤션

*   **언어**: Python.
*   **GUI 프레임워크**: PyQt6.
*   **스타일링**: `src/ui/style.py`에 사용자 정의 QSS 정의.
*   **비동기 작업**: 네트워크 요청 및 과도한 처리 중에 UI 응답성을 유지하기 위해 PyQt의 `QThread`를 사용하여 처리됩니다.
*   **리소스 관리**: 개발 및 PyInstaller 환경 모두에서 리소스에 대한 올바른 자산 경로를 처리하기 위해 `resource_path`를 사용합니다.
*   **패키징**: `PyInstaller`는 독립 실행형 실행 파일을 생성하기 위해 선택된 도구입니다.
*   **오류 처리**: `run.py`에 처리되지 않은 Python 예외를 catch하기 위한 전역 예외 핸들러가 포함됩니다.
*   **PowerPoint 템플릿**: 애플리케이션은 새 프레젠테이션 생성을 위한 기본으로 `template.pptx`에 의존합니다. 이 파일은 PyInstaller 빌드에 특별히 포함되며 Git에서 무시되지 않습니다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/seungheony)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/seungheony)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
