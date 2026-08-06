## patent-translation

> 이 프로젝트는 영어 특허 docx 문서를 한글로 번역하기 위한 준비 작업(기술분야 요약, 관련 특허 조사, 용어집 구축)을 자동화합니다.

# 특허 번역 프로젝트

이 프로젝트는 영어 특허 docx 문서를 한글로 번역하기 위한 준비 작업(기술분야 요약, 관련 특허 조사, 용어집 구축)을 자동화합니다.

## 업무 프로세스

1. 영어 docx 특허 문서를 읽고 기술분야를 간략히 요약한다.
2. 한국 특허청(KIPRIS)에 출원된 관련 특허 문서를 3건 조회하여 공통으로 사용된 기술 용어를 추출한다.
3. 추출한 용어를 CSV 포맷으로 정리한다. (source_term: 영어, target_term: 한글)
4. 추출한 용어를 /Users/bravesong/patent-translation/terms_en_new.csv로 저장한다. 이때, 
terms_en_new.csv이 이미 있는 경우, terms_en_new1.csv, terms_en_new2.csv 등으로 저장한다.
   - terms_en_new.csv는 이번 문서에서 새로 추출한 용어만 담는 기록용 파일이다(번역에 직접 쓰이지 않는다).
   - 다음 단계에서 기존 terms_en.csv와 합쳐 terms_en_add.csv로 저장하며, 실제 번역에는 이 terms_en_add.csv가 쓰인다.
5. 추출한 용어를 /Users/bravesong/patent-translation/terms_en.csv 파일의 기존 리스트에 중복 없이 추가하고, 업데이트된 전체 리스트는 /Users/bravesong/patent-translation/terms_en_add.csv 로 저장한다. (원본 terms_en.csv는 변경하지 않는다)
6. /Users/bravesong/patent-translation/patent_translator_merged.py 안에 있는 파이썬 코드를 읽고, 코드 내용을 먼저 보여준 뒤 실행한다.
   - 이 스크립트는 .docx를 직접 입력받는다(확장자로 자동 판별). docx → txt 사전 변환은 더 이상 필수가 아니다.
   - 단, 표가 있는 문서는 아래 "docx → txt 변환(선택)"을 참고한다.

## docx → txt 변환 (선택)

`patent_translator_merged.py`의 자체 .docx 추출(`extract_text_from_docx`)은 표 셀 텍스트를 행/열 구분 없이 줄 단위로만 이어붙인다.
표 구조(행당 한 줄, 셀은 탭 구분)를 보존해야 하는 문서라면 `docx_to_txt.py`로 먼저 변환해 .txt를 입력으로 넘긴다.

```
python /Users/bravesong/patent-translation/docx_to_txt.py X.docx > X.txt
```

- 문단과 표를 문서에 나타나는 순서대로 추출한다. 표는 행당 한 줄, 셀은 탭으로 구분한다.
- 변환된 .txt를 스크립트의 원본 경로로 넘긴다. 중간/최종 파일명이 이 이름을 기준으로 자동 생성된다.
- 이미지와 각주/미주, 머리말/꼬리말은 빠진다. 본문이 거기 들어 있는 문서면 별도 확인이 필요하다.
- 표 구조가 중요하지 않은 문서는 .docx를 스크립트에 바로 넘겨도 된다.

## 파일 경로

- 기존 용어집: /Users/bravesong/patent-translation/terms_en.csv
- 업데이트된 용어집(출력): /Users/bravesong/patent-translation/terms_en_add.csv
- 번역용 영문 용어집: /Users/bravesong/patent-translation/terms_en_add.csv (스크립트 DEFAULT_GLOSSARY_EN, [3단계]에서 적용)
- 번역용 한글 용어집: /Users/bravesong/patent-translation/terms_ko.csv (스크립트 DEFAULT_GLOSSARY_KO, [7단계]에서 적용)
- docx → txt 변환 스크립트(선택): /Users/bravesong/patent-translation/docx_to_txt.py
- 번역 실행 스크립트: /Users/bravesong/patent-translation/patent_translator_merged.py

## CSV 규칙

모든 용어집 공통:

- 컬럼명: source_term, target_term
- 인코딩: UTF-8 (BOM 유무 무관 — Excel 호환을 위해 BOM을 붙이고, `load_glossary()`가 `utf-8-sig`로 읽어 안전하게 처리한다)
- 기존 용어와 중복되는 term은 추가하지 않는다 (source_term 기준 비교)

KIPRIS 조사로 만드는 영문 용어집(terms_en.csv, terms_en_add.csv):

- 용어는 명사구 위주로 추출하고, 문장 단위 표현은 제외한다

## terms_ko.csv (한글 용어집)

번역된 한글 본문에 적용하는 한→한 대응표. 문체 변환(경어체→평서체), 용어 통일, 도면 표기 정리를 담당한다.
영문 용어집과 달리 문장 단위 표현을 담아도 된다. (예: `중요하지 않습니다,중요하지 않다`)

- [7단계](마지막 단계)에서 `apply_korean_glossary()`가 적용한다.
- 적용 순서는 source_term 길이 내림차순이다(`load_glossary`가 정렬). 파일의 행 순서는 결과에 영향을 주지 않으므로 가나다순으로 유지한다.
- 규칙은 자족적으로 쓴다. 다른 규칙이 먼저 적용되어야 성립하는 규칙은 만들지 않는다.
  - `합니다,한다`가 만든 "필요한다"를 뒤에서 고치는 대신, `필요합니다,필요하다`를 직접 등록한다.
  - `그림. 도,도`는 `그림,도`보다 길어 먼저 매칭되므로, "도. 도"를 청소하는 뒤처리 규칙이 필요 없다.
- 치환으로 받침이 바뀌면 뒤따르는 조사가 자동 교정된다. (`끝부분은` → `단부는`) 받침이 같으면 단순 치환한다.
- 문체 변환은 열거식이다. 등록되지 않은 어미는 그대로 남으므로, 새 문서에서 발견하면 그때 추가한다.
- `patent_translator_merged.py`의 [6단계] `apply_technical_style()`이 `terms_ko.csv`보다 먼저 실행되며, `합니다→한다`류의 경어체 변환 상당수를 코드에 하드코딩으로 이미 처리한다. `terms_ko.csv`에 같은 변환을 등록해도 그 시점엔 이미 바뀐 뒤라 무의미하니, `apply_technical_style()`의 `rules` 딕셔너리에 없는 표현만 `terms_ko.csv`에 추가한다.

## 주의사항

- terms_en.csv 원본 파일은 직접 덮어쓰지 않고, 항상 terms_en_add.csv로 별도 저장한다.
- terms_ko.csv에는 이 규칙이 적용되지 않는다. 파이프라인이 직접 읽는 파일이므로 원본을 그대로 편집한다.
- patent_translator_merged.py는 실행 전에 코드 내용(직전 실행 이후 바뀐 부분이 있으면 그 변경점도)을 사용자에게 보여준다. 다만 별도 승인을 기다리지 않고 이어서 전체를 실행한다.
- KIPRIS 관련 특허 조회 결과가 불충분하거나 애매하면, 추측하지 말고 사용자에게 확인을 요청한다.
- 기존 파일을 변경하기 전에 원본을 `backups/<날짜>_<작업명>/`에 원본 파일명 그대로 복사하고, 무엇의 원본인지 같은 폴더의 README.md에 남긴다. (인코딩 변환 없이 바이트 그대로 보관)

---
> Source: [bravesong2027-hash/Patent-Translation](https://github.com/bravesong2027-hash/Patent-Translation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
