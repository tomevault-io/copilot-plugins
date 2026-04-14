## yeonsu-library-web

> 공통 규칙: 모든 이미지에 alt 속성 필수 적용


(Figma MCP 사용 시)

---

이미지 관리

공통 규칙: 모든 이미지에 alt 속성 필수 적용

[ 이미지 ]
public\assets\images -> 이동 후 이미지 생성 후 경로 연결
<img src="./assets/images/파일" alt="파일" /> <-- 여기로 경로연결

[ 아이콘 이미지 ]
public\assets\images\icon -> 이동 후 이미지 생성 후 경로 연결
<img src="./assets/images/icon/파일" alt="파일" /> <-- 여기로 경로연결

[ svg ]
public\assets\images\icon -> 이동 후 svg 생성 후 경로 연결
<img src="./assets/images/icon/파일" alt="파일" /> <-- 여기로 경로연결

---

---

기존 UI 컴포넌트 먼저 확인 및 활용

UiButton - 버튼 요소
UiInput - 입력 필드
기타 Ui\_\_\_ 형태의 컴포넌트들

---

---

스타일

style 은 페이지에 scss로 생성

이렇게 큰 페이지 관련 scss 네임으로 감싸서 만들어줘

<style is:inline lang="scss">
.페이지명영어-page {
    max-width: 976px;
    margin: 0 auto;
    padding: 0 20px;

    .section-1 {
        .subsection-1-1 {
            .element-1-1-1 {
                // 스타일 작성
            }
        }
    }

    .section-2 {
        // 다른 섹션 스타일
    }
}

// 이 설정은 사용하지 않음
font-family: 'Pretendard GOV', sans-serif;

@mixin font($size, $weight, $color, $line-height: 1, $text-align: null) {
    font-size: $size;
    font-weight: $weight;
    color: $color;
    line-height: $line-height;
    text-align: $text-align;
}

// 사용 예시
.title {
    @include font(24px, 700, #333, 1.4, center);
}
------------------------------------------------------------------------

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/box3101) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
