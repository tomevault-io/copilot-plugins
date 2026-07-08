## geomag-site-selection

> > 이 파일을 읽고 작업에 임한다. 새로운 변경이 생기면 즉시 이 파일을 업데이트한다.

# 지구자기장 측정 입지 선정 프로젝트 — Claude 작업 규칙

> 이 파일을 읽고 작업에 임한다. 새로운 변경이 생기면 즉시 이 파일을 업데이트한다.

---

## 1. 프로젝트 개요

대한민국 지구자기장 총자력 측정 최적 입지를 자동 선정하는 Python 시스템.
- 인공 자기 간섭 요소(송전탑·철도·도시·지질) 제외 후 다기준 점수 산정
- Folium 기반 대화형 HTML 지도 생성 → GitHub Pages 배포

---

## 2. 주요 스크립트 및 실행 순서

```
① python geomag_site_selection.py   # 분석 + 지도 생성 → output/ 저장
② python build_docs.py              # output/ → docs/ 변환 (GeoJSON 단순화)
③ python create_methodology_doc.py  # 방법론 Word 문서 생성 → docs/output/ 저장
```

| 스크립트 | 역할 | 출력 |
|---|---|---|
| `geomag_site_selection.py` | 메인 분석·지도 생성 | `output/` |
| `build_docs.py` | 배포 빌드 (GeoJSON 단순화) | `docs/` |
| `create_methodology_doc.py` | 방법론 Word 문서 생성 | `docs/output/` |

---

## 3. 파일 경로 구조

```
/output/                        ← 중간 산출물 (git 제외, .gitignore)
  geomag_site_selection.html    ← 생성된 지도 HTML
  candidate_sites.csv
  data/
    candidates_p1/p2/p3.geojson
    gradient_low/mid_low/optimal/caution.geojson  ← 자기이상 등급별
    zone_*.geojson

/docs/                          ← GitHub Pages 배포 폴더 (git 커밋 대상)
  index.html                    ← build_docs.py가 output/HTML을 복사
  data/
    *.geojson                   ← build_docs.py가 단순화하여 복사
  output/
    YYYYMMDD_HHMMSS_*.docx      ← 방법론 문서
    *.csv, *.png 등 산출물
```

### 저장 경로 원칙
- 모든 산출물은 `docs/output/`에 저장 (git 커밋 대상)
- `docs/output/` 경로: `Path(__file__).parent / "docs" / "output"` (절대경로 하드코딩 금지)
- 로컬 백업: `C:/LG_gram_backup_users/LX/2026_geomag/docs/output/`
  - 두 경로가 동일한 경우(로컬 직접 실행 시) 복사 생략 (`resolve()` 비교)
- 루트 `/output/` 단독 저장 금지

### 파일명 규칙
- 날짜/시간 포함 파일: `YYYYMMDD_HHMMSS_설명.확장자`
  - 예: `20260514_153745_site_selection_methodology_geology.docx`

---

## 4. Git 워크플로우

### 커밋 대상
- `docs/` 폴더 전체 (HTML, GeoJSON, output 산출물, Word 문서)
- `output/`(루트)는 커밋하지 않는다 (`.gitignore` 처리됨)

### 표준 커밋 순서 (분석 결과 업데이트 시)
```powershell
python geomag_site_selection.py
python build_docs.py
git add docs/
git commit -m "분석 결과 업데이트"
git push origin HEAD:main
```

### Push 규칙
- 항상 `git push origin HEAD:main` 사용 (브랜치명 불일치 방지)
- non-fast-forward 오류 시: `git pull origin main --rebase` 후 재시도
- 워크트리가 뒤처진 경우도 동일하게 rebase

### .gitignore 핵심 규칙
- `/output/` — 루트 output만 제외 (`output/`로 쓰면 `docs/output/`도 차단되므로 주의)
- `docs/output/data/` — 대용량 원본 GeoJSON이 잘못 생성되는 폴더
- `data/*.json`, `data/*.csv`, `data/*.dat` — Overpass API 캐시, 대용량 원본
- `data/수치지질도_25만축척_전국/` — 대용량 shapefile
- `data/zone_cache/` — 제외구역 유니온 WKB 캐시 (생성된 바이너리, 커밋 불필요)

---

## 5. 입지 점수 산정 체계 (현재 구현 상태)

| # | 항목 | 배점 | 상태 | 산정 방식 |
|---|---|---|---|---|
| ① | 격자 데이터 희소성 | 25 | ✅ | KDTree K=5 최근접 이웃 평균 거리 정규화 |
| ② | 지형적 대표성 | 15 | ✅ | Open-Elevation API 경사도 (1 - slope/30°) × 15 |
| ③ | 전력/철도 이격도 | 15 | ✅ | log(dist) 함수 기반 정규화 |
| ④ | 인구 밀집 이격도 | 15 | ✅ | log(dist) 함수 기반 정규화 |
| ⑤ | 자기이상·지질경계 모델 기여도 | 10 | ✅ | KIGAM 반경 0.05° P90-P10 구간별 점수화 |
| ⑥ | 암상 적합성 | 5 | ✅ | 수치지질도(1:250,000) 자성암종·단층 이격거리 log 정규화 |
| ⑦ | 부지 지속성 | 10 | ❌ | 현장 확인 (미구현) |
| ⑧ | 관리 접근성 | 5 | ❌ | 현장 확인 (미구현) |

- **최대 가용 점수**: 85점 (①~⑥) → 100점 정규화
- **우선순위 등급**: P66 이상 → 1등급, P33~P66 → 2등급, P33 미만 → 3등급

### ⑤ 자기이상 점수 구간
| P90-P10 | 점수 |
|---|---|
| < 30 nT | 5점 |
| 30~150 nT | 8점 |
| 150~400 nT | 10점 (최고) |
| 400~800 nT | 7점 (현장 검증 필수) |
| > 800 nT | 조건부 제외 |
| 데이터 미취득 | 6점 |

---

## 6. 제외 구역 필터 (사전 제거)

| # | 조건 | 반경 |
|---|---|---|
| [1] | 고압철탑·송전탑 | 1.0 km |
| [2] | 철도 | 5.0 km |
| [3a] | 핵산도심·산업 | 500 m |
| [3b] | 주거·하락 | 300 m |
| [4] | 파이프라인 | 0.5 km |
| [5] | 풍신탑·기지국 | 0.5 km |
| [6] | 풍력발전기 | 0.5 km |
| [7] | 채석장·광산 | 1.0 km |
| [8] | 자기이상 P90-P10 >800 nT | 조건부 제외 |
| [지질] | 자성 암종 폴리곤 내부 | 직접 제외 |
| [지질] | 단층선 | 500 m |

---

## 7. 지도 레이어 구성

### 자기이상·지질경계 모델 기여도 레이어 (기본 비표시, 레이어 컨트롤에서 활성화)
| 레이어 | 색상 | 파일 |
|---|---|---|
| < 30 nT (복잡 작용) | 회색 #B0B0B0 | `docs/data/gradient_low.geojson` |
| 30~150 nT (지-중) | 하늘 #88CCEE | `docs/data/gradient_mid_low.geojson` |
| 150~400 nT (최적) ★ | 녹색 #2E8B57 | `docs/data/gradient_optimal.geojson` |
| 400~800 nT (검증 필요) | 황색 #DDAA00 | `docs/data/gradient_caution.geojson` |

### 팝업에 표시되는 점수 컬럼
`s1`(희소성) · `s2`(지형) · `s3`(전력철도) · `s4`(인구) · `s5`(모델기여도) · `s6`(암상)
- `vn()` 함수로 NaN → `-` 처리
- `s6`은 `rename_map`에 `"s6_암상": "s6"` 포함되어야 GeoJSON에 저장됨

---

## 8. 방법론 문서 (`create_methodology_doc.py`)

- 저장: `docs/output/YYYYMMDD_HHMMSS_site_selection_methodology_geology.docx`
- 날짜: `datetime.now().strftime("%Y년 %m월")` 자동 생성 (하드코딩 금지)
- 로컬 백업: `Path("C:/LG_gram_backup_users/LX/2026_geomag/docs/output")`
  - 동일 경로 복사 방지: `if docs_path.resolve() != local_path.resolve()`
- 문서 내용이 바뀌면 재실행 후 `git add docs/output/` → 커밋

---

## 9. Python 환경

- 인터프리터: `C:\Users\YOONS\anaconda3\python.exe`
- 실행 예시: `& "C:\Users\YOONS\anaconda3\python.exe" geomag_site_selection.py`
- 주요 패키지: geopandas, shapely, folium, numpy, pandas, requests, scipy, python-docx
- 좌표계: WGS84 (EPSG:4326) 입출력, UTM Zone 52N (EPSG:32652) 공간 분석

---

## 10. 성능 최적화 — 제외구역 캐시

`build_exclusion_zones()` 내 `unary_union`은 주거 1.17M · 수계 1.44M 피처로 약 9분 소요.
두 번째 실행부터 `data/zone_cache/` WKB 캐시를 사용해 수 초로 단축.

### 캐시 작동 방식
- `_load_zone_cache(name, source_paths, buffer_key)`: WKB + meta JSON 확인 → 소스 파일 mtime 비교 → 유효하면 geometry 반환
- `_save_zone_cache(name, geom, source_paths, buffer_key)`: WKB + meta JSON 저장
- `build_exclusion_zones()` 내 `_cached()` 클로저가 자동으로 캐시 시도 → miss 시 계산 → 저장

### 캐시 무효화 조건
- 소스 JSON 파일 수정 시간이 캐시보다 최신인 경우
- `buffer_key` (버퍼 반경 또는 임계값 문자열) 변경 시
- `data/zone_cache/*.wkb` 수동 삭제 시 (재계산 강제)

### 캐시 경로
`data/zone_cache/` — `.gitignore`에 포함, 커밋하지 않음

---

## 11. 자주 발생하는 문제 및 해결

| 문제 | 원인 | 해결 |
|---|---|---|
| `git push` non-fast-forward | 브랜치 불일치 | `git pull origin main --rebase` 후 재push |
| `docs/output` ignored 경고 | `.gitignore`의 `output/` 패턴이 하위 경로도 차단 | `/output/`로 루트 한정 |
| `create_methodology_doc.py` PermissionError | 동일 경로 자기복사 | `resolve()` 비교 후 동일하면 복사 생략 |
| push 타임아웃 (HTTP 408) | 대용량 파일 포함 | `docs/output/data/` 경로 확인 및 제거 |
| 팝업에 암상 `-` 표시 | `rename_map`에 `s6_암상` 누락 | `"s6_암상": "s6"` 추가 확인 |
| gradient 레이어 컨트롤 미표시 | `anomaly_tiers_available=False` | KIGAM 데이터 존재 확인 후 재실행 |

---
> Source: [Yoonsoo-ji/geomag-site-selection](https://github.com/Yoonsoo-ji/geomag-site-selection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
