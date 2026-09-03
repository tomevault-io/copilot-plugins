## mapleland-utils

> 여러 리포에 흩어진 메이플랜드 유틸을 `mapleland.myungwoo.kr` 한 도메인으로 모아

# 메이플랜드 유틸 모음 (집계 사이트)

여러 리포에 흩어진 메이플랜드 유틸을 `mapleland.myungwoo.kr` 한 도메인으로 모아
배포하는 리포. 이 리포에는 앱 코드가 없다. **하는 일은 조립**이다.

```bash
node scripts/build-site.mjs        # dist/ 에 랜딩 · 404 · 리다이렉트 · CNAME 생성
python3 -m http.server -d dist     # 랜딩만 확인 (유틸 카드 링크는 404 로 뜨는 게 정상)
```

유틸 본체까지 포함한 사이트 전체는 `.github/workflows/deploy.yml` 이 조립한다.
각 유틸 리포를 clone → `NEXT_PUBLIC_BASE_PATH=/<슬러그>` 로 빌드 → `dist/<슬러그>/` 에 복사.

## 깨뜨리면 안 되는 것

### 1. 도메인은 이 리포만 소유한다

GitHub Pages 는 **하나의 커스텀 도메인을 하나의 사이트에만** 붙일 수 있다. 유틸 리포에
`mapleland.myungwoo.kr` CNAME 을 추가하면 도메인이 그 리포로 넘어가면서 이 사이트가
죽는다. 유틸을 하위 경로로 붙이는 방법은 여기서 함께 빌드하는 것뿐이다.

커스텀 도메인은 **리포 설정(Settings → Pages)에** 저장된다. Actions 로 배포할 때는
산출물의 `CNAME` 파일이 도메인을 정하지 않는다(브랜치 배포 방식에서만 그렇다).
`dist/CNAME` 을 계속 만드는 이유는 배포 방식을 브랜치로 되돌리거나 다른 정적 호스팅에
올릴 때의 안전망이고, 지금 도메인의 실제 출처는 설정이다. 설정에서 도메인을 비우면
사이트는 `myungwoo.github.io/mapleland-utils` 로 돌아가고, 유틸들의 링크는 루트 기준
절대경로(`/damage/`)라서 전부 깨진다.

### 2. 유틸 목록의 단일 출처는 `utils.json`

랜딩 페이지 카드, 배포 대상, 리다이렉트가 모두 여기서 나온다. `dist/` 는 전부 산출물이니
`dist/index.html` 을 직접 고치지 말 것 — 다음 배포에 사라진다.

`scripts/build-site.mjs` 를 두는 이유가 이것이다. 목록을 HTML 과 워크플로 양쪽에 손으로
적으면 유틸을 추가할 때 한쪽만 고치고 끝나서 반드시 어긋난다.

### 3. 슬러그는 공개 주소다

`slug` 는 사람들이 공유하고 북마크하는 주소(`/damage`)다. 바꾸면 남이 공유해 둔 링크가
깨진다. 부득이하게 바꿀 때는 **옛 슬러그로 들어온 요청을 새 슬러그로 보내는 리다이렉트를
남긴다**. `build-site.mjs` 의 리다이렉트 생성부가 그 자리다(현재는 리포 이름 → 슬러그).

### 4. 유틸 리포에 요구되는 것

새 유틸을 붙이려면 그 리포가 이걸 만족해야 한다. 위 둘이 없으면 `out/` 이 없어서 빌드가
죽는다. 세 번째는 죽지 않고 **조용히 다른 유틸의 설정을 망가뜨린다**(§5).

- `basePath` 를 `NEXT_PUBLIC_BASE_PATH` 환경변수로 받는다.
- `output: "export"` 로 `out/` 을 내보낸다.
- localStorage / IndexedDB 키에 `ml:<슬러그>:` 접두어를 붙인다 (§5).

**함정:** `actions/configure-pages` 의 `static_site_generator: next` 를 쓰면, 그 액션이
`next.config.js` 를 새로 만들어 `output`/`basePath` 를 주입한다. Next 는 `next.config.js`
를 `next.config.ts` 보다 먼저 읽으므로 **리포의 `next.config.ts` 가 조용히 무시된다.**
`damage-calculator` 와 `mapleland-timer` 가 실제로 그 상태였다. 유틸 리포에서 설정이
안 먹는 것 같으면 이걸 먼저 의심할 것.

### 5. localStorage 키는 오리진을 공유한다

유틸 다섯 개가 `mapleland.myungwoo.kr` 하나를 쓴다. **localStorage 는 오리진 단위라
경로로 갈라지지 않는다** — `/damage` 와 `/hunt` 는 같은 저장소를 본다.
`myungwoo.github.io/<repo>` 들도 원래부터 같은 오리진이었다.

규칙:

- **앱 전용 값: `ml:<슬러그>:<이름>`** (`ml:damage:save:0`, `ml:hunt:records`,
  `ml:exp:roiLevel`, `ml:split:tabs:v1`)
- **일부러 공유하는 값: `ml:theme`** — 값은 `'light' | 'dark' | 'system'`.
  같은 사이트인데 앱마다 다크모드를 따로 기억하면 화면이 튀어서 공유한다.
  **모르는 값을 만나면 시스템 설정으로 보고, 절대 덮어쓰지 말 것.**
  데미지 계산기가 `'system'` 을 무효 값으로 보고 덮어써서 사냥 타이머의 선택을
  지우던 버그가 실제로 있었다. 공유 키의 값 집합을 앱마다 다르게 해석하면 그 꼴이 난다.
- **키를 바꿀 때는 예전 키를 지우지 않는다.** 새 키가 비어 있을 때만 한 번 복사한다.
  배포를 되돌릴 일이 생겨도 사용자 데이터가 남아 있어야 하고, 그래야 마이그레이션이
  여러 번 돌아도 결과가 같다.
- IndexedDB 이름과 sessionStorage 도 같은 오리진을 쓴다. 새 유틸을 붙일 때 확인할 것.

새 유틸을 `utils.json` 에 추가하기 전에, 그 앱이 접두어 없는 키를 쓰는지 먼저 볼 것.
접두어 없는 키는 조용히 남의 설정을 덮어쓴다 — 에러가 아니라 "설정이 이상해졌다"로
나타나서 원인을 찾기 어렵다.

### 6. 파비콘은 이 사이트만 가진다

각 유틸은 자기 파비콘을 그대로 쓴다(버섯, 주문서, EXP, 코인). 랜딩만 `assets/favicon.svg`
의 M 마크를 쓰고, `build-site.mjs` 가 `assets/` 를 `dist/` 로 복사한다. 파일을 추가하면
자동으로 따라온다.

- `favicon.svg` — 최신 브라우저. 이게 원본이다.
- `favicon-32.png` — SVG 파비콘을 모르는 구형 브라우저 폴백.
- `apple-touch-icon.png` — iOS 홈 화면(180px). **모서리를 깎지 않은 정사각**이어야 한다.
  iOS 가 알아서 깎으므로, 둥근 모서리를 넣으면 두 번 깎여 이상해진다. 그래서 마크도
  0.84 로 줄여 안쪽 여백을 뒀다.

PNG 는 SVG 에서 뽑은 것이다. 마크를 고치면 두 PNG 도 다시 만들어야 한다(macOS 기준):

```bash
qlmanage -t -s 720 -o /tmp assets/favicon.svg     # SVG → PNG 래스터
python3 -c "from PIL import Image; Image.open('/tmp/favicon.svg.png').resize((32,32), Image.LANCZOS).save('assets/favicon-32.png')"
```

`apple-touch-icon.png` 은 정사각 바탕 + 0.84 스케일 마크로 따로 만든 SVG 를 180px 로
줄인 것이다. CI 에 래스터라이저가 없어서 PNG 를 커밋해 두는 쪽을 골랐다 — 빌드 때
만들려면 의존성이 붙는다.

### 7. 유틸 링크는 새 탭에서 연다

`build-site.mjs` 의 카드에 `target="_blank" rel="noopener"` 가 붙어 있다. 취향이 아니라
파비콘 때문이다: 같은 탭에서 열면 유틸의 파비콘이 탭에 남고, 뒤로 가기로 돌아와도
브라우저가 아이콘을 바로 되돌리지 않아 랜딩이 남의 아이콘을 달고 있다.
`rel="noopener"` 는 새 탭이 `window.opener` 로 이 페이지를 만지지 못하게 막는다.

문구도 `새 탭에서 열기 ↗` 다. 새 탭에서 열리는데 "바로 쓰기"라고만 적어 두면
동작과 말이 다르다.

### 8. 유틸이 갱신됐다는 사실은 알림으로만 들어온다

유틸이 업데이트돼도 이 리포에는 푸시가 없다. 그래서 각 유틸 리포의 배포 워크플로가
마지막에 `repository_dispatch`(`utils-updated`) 를 보내고, 이 리포는 그걸 받아 다시
빌드한다. 유틸 쪽 시크릿은 `MAPLELAND_UTILS_DISPATCH`(이 리포에 대한 Contents 쓰기
권한만 가진 fine-grained PAT)다.

cron(03:00 KST)은 안전망이지 주 경로가 아니다. **스케줄 워크플로는 리포에 60일간
활동이 없으면 GitHub 가 자동으로 끈다.** 이 리포는 조용하므로 cron 을 믿고 알림을
빼면, 두 달 뒤 아무도 모르게 사이트가 굳는다.

PAT 가 만료되면 알림이 조용히 끊긴다(유틸 쪽 워크플로는 시크릿이 없으면 로그만 남기고
넘어간다). 사이트가 오래된 것 같으면 유틸 리포의 최근 배포 로그에서 "Notify
mapleland-utils" 단계를 먼저 볼 것.

### 9. 유틸의 에셋 경로는 슬러그에 묶여 있다

각 유틸 안에서 `"/alert.mp3"` 같은 루트 절대경로를 쓰면 `basePath` 를 안 타서 슬러그가
바뀌는 순간 깨진다. 상대경로(`"./alert.mp3"`)나 `NEXT_PUBLIC_BASE_PATH` 를 붙여 쓸 것.
워크플로의 확인 단계가 `dist/<슬러그>/index.html` 의 `_next` 경로만 검사하므로,
그 외 에셋은 이 규칙으로 막는다.

### 10. 랜딩 페이지에 프레임워크를 들이지 않는다

지금 랜딩은 의존성 0, 빌드 0(노드 표준 라이브러리만)이다. 카드 다섯 장을 그리는 일에
번들러를 붙이면 이 리포의 CI 가 유틸 5개 빌드 + 자기 빌드로 늘어난다. 스타일은
`build-site.mjs` 안의 인라인 CSS 하나로 유지한다.

## 배포 한 번만 하는 설정

1. DNS: `mapleland` CNAME → `myungwoo.github.io`
2. Settings → Pages → Source: **GitHub Actions**, Custom domain: `mapleland.myungwoo.kr`
3. Enforce HTTPS

---
> Source: [myungwoo/mapleland-utils](https://github.com/myungwoo/mapleland-utils) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
