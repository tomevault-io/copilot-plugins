## braillify

> 한국어 텍스트를 한국 점자로 변환하는 라이브러리.

# Braillify

한국어 텍스트를 한국 점자로 변환하는 라이브러리.

## 프로젝트 구조

- `libs/braillify/` — Rust 핵심 변환 엔진
- `packages/node/` — Node.js WASM 바인딩
- `packages/python/` — Python 바인딩 (maturin)
- `apps/landing/` — Next.js 랜딩 페이지
- `test_cases/` — 점자 변환 테스트 케이스 (JSON)
- `docs/` — 2024 개정 한국 점자 규정 PDF
- `braillove-case-collector/` — 점자 내부표기 → 숫자/유니코드 변환기

## 빌드 & 테스트

```bash
bun install
cargo build --release -p braillify
bun test                    # 전체 테스트 (Rust + Bun + Python)
bun test test_cases/         # 테스트케이스 무결성 검증만
```

## 테스트 케이스 규칙

### 파일 구조

- `test_cases/korean/rule_{N}.json` — 한글 점자 제N항
- `test_cases/korean/rule_{N}_b1.json` — 제N항 붙임 1
- `test_cases/math/math_{N}.json` — 수학 점자 제N항
- 근거: `docs/2024 개정 한국 점자 규정.pdf`

### 엔트리 형식

```json
{
  "input": "입력 텍스트 (묵자 또는 LaTeX)",
  "note": "설명 (선택, 동일 input이 여럿이거나 맥락 필요 시에만)",
  "internal": "점자 내부표기",
  "expected": "브라유셀 인덱스 연결 문자열",
  "unicode": "점자 유니코드 문자열"
}
```

### internal → expected/unicode 변환

`braillove-case-collector/converter.py`의 패턴을 따른다:

```
pattern: " a1b'k2l@cif/msp"e3h9o6r^djg>ntq,*5<-u8v.%[$+x!&;:4\0z7(_?w]#y)="
braille: ⠀⠁⠂⠃⠄⠅⠆⠇⠈⠉⠊⠋⠌⠍⠎⠏⠐⠑⠒⠓⠔⠕⠖⠗⠘⠙⠚⠛⠜⠝⠞⠟⠠⠡⠢⠣⠤⠥⠦⠧⠨⠩⠪⠫⠬⠭⠮⠯⠰⠱⠲⠳⠴⠵⠶⠷⠸⠹⠺⠻⠼⠽⠾⠿
```

특수 매핑: `` ` ``→0, `{`→42, `}`→59, `~`→24, `|`→51

`expected`는 각 문자의 인덱스를 문자열로 이어붙인 것, `unicode`는 대응하는 점자 유니코드 문자를 이어붙인 것이다.

### 무결성 검증

`test_cases/testcase-integrity.test.ts`가 모든 엔트리의 internal → expected/unicode 일치를 검증한다. 대문자(수학 변수 A, B 등)를 포함한 internal은 기본 패턴 외이므로 skip된다.

### 테스트 케이스 작성 원칙

1. **PDF가 유일한 근거** — `docs/2024 개정 한국 점자 규정.pdf`에 없는 예제를 만들지 않는다.
2. **PDF 순서 준수** — 기호 정의 → 해당 예제 순서로, PDF에 나온 순서 그대로 배치한다.
3. **기호 단독 엔트리 필수** — 각 기호는 단독 엔트리로 먼저 등록하고, 그 뒤에 해당 기호를 사용하는 예제가 온다.
4. **note는 필요할 때만** — 동일 input이 다른 의미로 쓰일 때, 또는 맥락이 필요할 때만 추가한다. input을 반복하는 note는 쓰지 않는다.
5. **소속 정확히** — 각 엔트리는 해당 항 파일에만 존재한다. 다른 항의 예제를 섞지 않는다.

### LaTeX 입력

수학 수식은 LaTeX 형식의 input도 테스트한다. 기존 엔트리의 LaTeX 버전을 추가하는 방식이다:

- 형식: `$<LaTeX 수식>$` (앞에 `$`, 뒤에 `$`)
- 동일한 `internal`/`expected`/`unicode`를 공유
- `"note": "LaTeX"` 표기
- **기존 예제의 변환만** — 새로운 수식을 만들지 않는다

```json
{
  "input": "$\\frac{3}{4}$",
  "note": "LaTeX",
  "internal": "#d/#c",
  "expected": "6025129",
  "unicode": "⠼⠙⠌⠉"
}
```

주요 LaTeX 변환:

| 수식 | LaTeX |
|------|-------|
| 분수 | `$\frac{분자}{분모}$` |
| 근호 | `$\sqrt{x}$`, `$\sqrt[n]{x}$` |
| 위첨자 | `$x^{2}$` |
| 아래첨자 | `$x_{n}$` |
| 부등호 | `$\neq$`, `$\geq$`, `$\leq$` |
| 절댓값 | `$\|x\|$` |
| 무한대 | `$\infty$` |
| 적분 | `$\int f(x)dx$` |
| 집합 | `$\cup$`, `$\cap$`, `$\subset$`, `$\emptyset$` |
| 논리 | `$\land$`, `$\lor$`, `$\forall$`, `$\exists$` |

### 대문자 수학 변수

수학 점자에서 대문자 변수(A, B, N 등)를 사용하는 internal은 기본 64셀 패턴에 포함되지 않는다. 이런 엔트리는 `expected`/`unicode`가 빈 문자열이며, 무결성 테스트에서 자동으로 skip된다.

---
> Source: [dev-five-git/braillify](https://github.com/dev-five-git/braillify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
