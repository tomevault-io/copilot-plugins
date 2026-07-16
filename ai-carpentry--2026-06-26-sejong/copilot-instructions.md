## 2026-06-26-sejong

> 세종대학교 2026-06-26 워크숍 사례로 정리한 The Carpentries 공식 `workshop-template` 기반 웹사이트 구축 절차.

# Software Carpentry 워크숍 웹사이트 구축 가이드

세종대학교 2026-06-26 워크숍 사례로 정리한 The Carpentries 공식 `workshop-template` 기반 웹사이트 구축 절차.

---

## 1. 개요

- **템플릿 출처**: https://github.com/carpentries/workshop-template
- **배포**: GitHub Pages (`gh-pages` 브랜치)
- **정적 사이트 생성기**: Jekyll (github-pages gem 사용)
- **저장소 명명 규칙**: `YYYY-MM-DD-사이트명` (예: `2026-06-26-sejong`)

---

## 2. 사전 요구사항

| 도구 | 최소 버전 | 비고 |
|------|----------|------|
| Ruby | 3.0+ | `ffi` gem 호환을 위해 2.7은 권장하지 않음 |
| Bundler | 2.x | `gem install bundler` |
| Git | 2.x | |
| GitHub 계정 | — | `gh-pages` 브랜치 자동 배포 |
| Eventbrite 계정 | — | 참가 등록 (선택) |

rbenv 사용자는 `.ruby-version` 확인 후 필요 시 최신 버전 설치.

---

## 3. 저장소 초기 세팅

### 3.1 템플릿에서 새 저장소 생성
1. https://github.com/carpentries/workshop-template 에서 **Use this template** → 저장소명을 `YYYY-MM-DD-장소` 형식으로 생성 (예: `2026-06-26-sejong`).
2. 로컬 클론:
   ```bash
   git clone git@github.com:<org>/2026-06-26-sejong.git
   cd 2026-06-26-sejong
   ```
3. 기본 브랜치를 `gh-pages`로 설정 (GitHub Pages가 이 브랜치에서 자동 빌드).

### 3.2 Bundler 설치
```bash
bundle config set --local path .vendor/bundle
bundle install
```

**Ruby 버전 호환 문제 발생 시** (예: `ffi requires ruby >= 3.0`):
```bash
# 현재 플랫폼을 lock 파일에 추가한 뒤 호환 버전으로 다운그레이드 재해결
bundle lock --add-platform x86_64-darwin
bundle lock --update=ffi
bundle install
```
근본 해결은 Ruby 3.0 이상 설치.

---

## 4. `index.md` front matter 구성

워크숍 정보는 `index.md` 상단 YAML front matter에 기술한다. 세종대 사례:

```yaml
---
layout: workshop                                           # 고정값
venue: "Sejong University"                                 # 기관명 (주소 제외)
address: "서울특별시 광진구 능동로 209(군자동) 율곡관 201호"  # 상세 주소
country: "kr"                                              # ISO 3166-1 alpha-2 소문자
language: "ko"                                             # ISO 639-1 소문자
latitude: "37.550106"                                      # https://www.latlong.net/
longitude: "127.073171"
humandate: "Jun 26-27, 2026"                               # ⚠ 3글자 월 형식 필수
humantime: "10:00 am - 5:00 pm KST (1:00 am - 8:00 am UTC)"
startdate: 2026-06-26                                      # ISO 8601
enddate: 2026-06-27
instructor: ["Kwangchun Lee"]
helper: ["HwanHee Hyung"]
email: ["kwangchun.lee.7@gmail.com"]
collaborative_notes: https://pad.carpentries.org/2026-06-26-sejong
eventbrite: "1988050060247"                                # 문자열로 감싸기
---
```

### 4.1 주의할 필드

- **`humandate`**: `make workshop-check`가 **3글자 월 + 4자리 연도** 형식만 통과시킴. `"June 26-27, 2026"` → **실패**, `"Jun 26-27, 2026"` → 통과.
- **`humantime`**: KST 기준과 UTC 환산을 병기. KST = UTC+9.
- **`language`**: 한국어 페이지는 `"ko"`로 지정. 하지만 기본 `_layouts/workshop.html`은 `<html lang="en">`이 하드코딩되어 있어 별도 수정 필요 (§7 참고).
- **`eventbrite`**: 숫자형으로 두면 Liquid가 정수로 처리할 수 있으므로 **반드시 따옴표로 감쌀 것**.

---

## 5. Eventbrite 등록 연동

### 5.1 Eventbrite 이벤트 생성
1. https://eventbrite.com → **Create Event**
2. 기본 정보 입력:
   - 이벤트명: `Software Carpentry Workshop at Sejong University`
   - 날짜/시간: 워크숍 일정 (KST)
   - 장소: 대면이면 실주소, 온라인이면 URL
3. 티켓 설정 (무료/유료, 정원 등) 후 **Publish**.
4. 이벤트 URL 끝자리의 숫자 ID 복사 (예: `https://www.eventbrite.com/e/...-1988050060247` → `1988050060247`).
5. `index.md` front matter에 `eventbrite: "1988050060247"` 입력.

### 5.2 ⚠ 레거시 iframe 엔드포인트 문제

`workshop-template` 기본값은 다음 iframe을 렌더링:
```html
<iframe src="https://www.eventbrite.com/tickets-external?eid=...&ref=etckt">
```

**문제**: 이 `tickets-external` 경로는 deprecated. Eventbrite의 CloudFront WAF가 리퍼러·지역·브라우저 조건에 따라 **403 Forbidden**을 반환하는 사례 빈번. 한국 IP에서 특히 잦음.

**해결**: `index.md`의 iframe 블록을 공식 최신 위젯 + 직접 링크 fallback으로 교체.

```html
{% if page.eventbrite %}
<div id="eventbrite-widget-container-{{page.eventbrite}}" style="margin: 1em 0;"></div>
<p class="text-center">
  <a id="eventbrite-widget-modal-trigger-{{page.eventbrite}}"
     href="https://www.eventbrite.com/e/{{page.eventbrite}}"
     class="btn btn-success btn-lg"
     target="_blank" rel="noopener">
    Register on Eventbrite
  </a>
</p>
<script src="https://www.eventbrite.com/static/widgets/eb_widgets.js"></script>
<script type="text/javascript">
  window.EBWidgets && window.EBWidgets.createWidget({
    widgetType: 'checkout',
    eventId: '{{page.eventbrite}}',
    iframeContainerId: 'eventbrite-widget-container-{{page.eventbrite}}',
    iframeContainerHeight: 425,
    modal: true,
    modalTriggerElementId: 'eventbrite-widget-modal-trigger-{{page.eventbrite}}'
  });
</script>
{% endif %}
```

아울러 `_includes/javascript.html`에서 **구식 iframe 높이 검사 훅도 제거** — 더 이상 의미 없음.

---

## 6. 스케줄 편집

`index.md` 하단의 Day 1 / Day 2 테이블을 실제 커리큘럼에 맞게 편집. 세종대 10:00-17:00 사례:

```html
<tr> <td>10:00</td>  <td>Automating Tasks with the Unix Shell</td> </tr>
<tr> <td>11:30</td>  <td>Morning break</td> </tr>
<tr> <td>12:00</td>  <td>Automating Tasks with the Unix Shell (Continued)</td> </tr>
<tr> <td>13:00</td>  <td>Lunch break</td> </tr>
<tr> <td>14:00</td>  <td>R for Reproducible Scientific Analysis</td> </tr>
<tr> <td>15:30</td>  <td>Afternoon break</td> </tr>
<tr> <td>16:00</td>  <td>R for Reproducible Scientific Analysis (Continued)</td> </tr>
<tr> <td>16:30</td>  <td>Wrap-up</td> </tr>
<tr> <td>17:00</td>  <td>END</td> </tr>
```

`humantime`의 시작·종료 시각과 **스케줄 테이블의 마지막 END 행을 반드시 일치**시킬 것.

---

## 7. 한국어 페이지 i18n 패치

기본 `_layouts/workshop.html:22`가 `<html lang="en">`로 하드코딩되어 스크린리더·검색엔진이 한국어를 영어로 오인식. front matter의 `language`를 읽도록 수정:

```diff
- <html lang="en">
+ <html lang="{{ page.language | default: site.lang | default: 'en' }}">
```

---

## 8. 로컬 개발 서버

```bash
bundle exec jekyll serve --host 127.0.0.1 --port 4000
```

- http://127.0.0.1:4000 에서 확인.
- 파일 변경 시 자동 리빌드 (Auto-regeneration).
- **로컬 한정 현상 (무시 가능)**:
  - 네비바 "Improve this page" 링크가 `/edit//index.md` 같은 깨진 URL 생성 → `_includes/gh_variables.html`이 `jekyll.environment == "development"`일 때 `repo_url`을 빈 문자열로 두기 때문. GitHub Pages 프로덕션에서는 `site.github.repository_url`이 자동 채워져 정상.

---

## 9. 검증

```bash
bundle exec make workshop-check
```

- 종료 코드 0 = 통과.
- 주요 체크 항목: `humandate` 포맷, 날짜 범위, 필수 필드 유무, country/language 코드 유효성.

사이트 빌드만 확인하려면:
```bash
bundle exec make site
```

---

## 10. 배포 (GitHub Pages)

```bash
git add index.md _layouts/workshop.html _includes/javascript.html
git commit -m "Configure workshop metadata and registration"
git push origin gh-pages
```

- GitHub Pages가 `gh-pages` 브랜치를 감지해 자동 빌드 (보통 1–2분).
- 공개 URL: `https://<org>.github.io/2026-06-26-sejong/`
- 배포 상태: GitHub 저장소 → **Actions** 탭의 *pages build and deployment* 워크플로우에서 확인.

---

## 10-A. Setup 섹션 YouTube 동영상 비공개 이슈 (⚠ 2026-04 현재 필수 대응)

### 10-A.1 증상
Setup 섹션의 Windows/macOS 탭에 임베드된 "Video Tutorial" 재생 영역이 **"비공개 동영상"** 메시지를 띄우며 재생 불가. 브라우저 개발자 도구로 확인하면 YouTube가 `playabilityStatus: "LOGIN_REQUIRED"` + `reason: "비공개 동영상"` 을 반환.

### 10-A.2 원인
`workshop-template` 초기 버전에 박힌 다음 6개 YouTube ID는 모두 **업로더가 비공개(private)로 전환**된 상태:

| 비디오 ID | 사용처 |
|-----------|--------|
| `339AEqk9c-8` | `_includes/install_instructions/shell.html` (Windows Git Bash) |
| `9LQhwETCdwY` | `shell.html`, `git.html`, `editor.html` (macOS Terminal) — 3곳 중복 |
| `xxQ0mzZ8UvA` | `python.html` (Windows Python) |
| `TcSAln46u9U` | `python.html` (macOS Python) |
| `q0PjTAylwoU` | `r.html` (Windows R) |
| `5-ly3kyxwEg` | `r.html` (macOS R) |

확인 명령:
```bash
for id in 9LQhwETCdwY xxQ0mzZ8UvA TcSAln46u9U q0PjTAylwoU 5-ly3kyxwEg 339AEqk9c-8; do
  curl -sL -A "Mozilla/5.0" -H "Cookie: CONSENT=YES+cb" \
    "https://www.youtube.com/watch?v=$id" \
    | grep -oE '"reason":\{"simpleText":"[^"]+"'
done
# 모두 "비공개 동영상" 출력
```

### 10-A.3 업스트림 공식 대응
- 이슈: https://github.com/carpentries/workshop-template/issues/450 ("Update video tutorials", 2025-04 갱신)
- **결론: 업스트림은 모든 install_instructions 파일에서 비디오 iframe을 완전히 제거**
- 검증:
  ```bash
  for f in shell git python r editor; do
    curl -sL "https://raw.githubusercontent.com/carpentries/workshop-template/gh-pages/_includes/install_instructions/$f.html" \
      | grep -c "iframe\|youtube"
  done
  # 모두 0
  ```

### 10-A.4 권장 해결 방법 (3가지 중 택1)

#### (A) 업스트림 방식 — 비디오 블록 완전 제거 **[권장]**

각 파일에서 다음 패턴의 블록을 삭제:
```html
<h4[^>]*>Video Tutorial</h4>
<div class="yt-wrapper2">
<div class="yt-wrapper">
<iframe ... youtube-nocookie.com/embed/... ></iframe>
</div>
</div>
```

추가로 `shell.html` 본문에 있는 다음 문장도 함께 제거:
> "See the Git installation [video tutorial](#shell-macos-video-tutorial) for an example on how to open the Terminal."

대상 파일 5개:
- `_includes/install_instructions/shell.html`
- `_includes/install_instructions/git.html`
- `_includes/install_instructions/python.html`
- `_includes/install_instructions/r.html`
- `_includes/install_instructions/editor.html`

가장 안전한 방법은 업스트림 `gh-pages` 브랜치에서 해당 5개 파일을 통째로 가져오는 것:
```bash
for f in shell git python r editor; do
  curl -sL -o "_includes/install_instructions/$f.html" \
    "https://raw.githubusercontent.com/carpentries/workshop-template/gh-pages/_includes/install_instructions/$f.html"
done
```
단, 업스트림 파일에 다른 변경사항이 포함되어 있을 수 있으므로 `git diff`로 확인 후 커밋.

#### (B) 공개된 대체 영상으로 교체
Carpentries 공식 YouTube 채널(https://www.youtube.com/@TheCarpentries) 또는 Mike Renfro의 공개 업로드(https://vps.mike.renf.ro/)에서 설치 영상을 찾아 ID를 교체. 다만 **업스트림이 영상을 완전히 제거한 방향이므로 유지보수 부담이 큼** — 권장하지 않음.

#### (C) 자체 녹화 영상 링크
세종대 환경(한국어 UI)에 맞춘 자체 녹화 영상을 YouTube에 공개 업로드 후 ID 교체. 한국어 학습자에게 유용하지만 워크숍 1회용으로는 과도한 비용.

### 10-A.5 세종대 사례 적용 권고
- 단기: **방법 (A)** 로 비디오 블록 제거 → 즉시 "비공개 동영상" 에러 해소
- 중기(여유 있으면): 방법 (C) 로 한국어 설치 가이드 2-3분짜리 자체 영상 촬영·공개 → 학습자 접근성 향상

---

## 10-B. GitHub Actions CI 워크플로우 유지보수 (⚠ 오래된 템플릿 필수 패치)

`workshop-template`에서 포크한 레포는 `.github/workflows/website.yml`, `template.yml`이 수년 전 버전으로 고정되어 있어 GitHub 플랫폼 변경에 따라 **실행 자체가 막히는** 두 가지 이슈가 연쇄 발생합니다. Pages 자체 배포(`pages-build-deployment`)는 별개 경로라 사이트 가용성에는 영향 없지만, Actions 탭이 빨간색으로 남아있어 관리상 반드시 수정.

### 10-B.1 문제 1 — `ubuntu-20.04` 러너 폐지 (2025-04-15)

**증상**: Actions 탭에서 Website 잡이 무한 `queued`, 메시지 *"Waiting for a runner to pick up this job..."*.

**원인**: `runs-on: ubuntu-20.04` 는 폐지된 이미지. 매칭 러너 없음.

**해결 — 두 파일에서 동시 교체**:
```yaml
# 변경 전 → 변경 후
runs-on: ubuntu-20.04          →  runs-on: ubuntu-22.04
RSPM: ".../__linux__/focal/..."  →  RSPM: ".../__linux__/jammy/..."
remotes::system_requirements("ubuntu", "20.04")  →  ("ubuntu", "22.04")
```

스턱된 run은 `gh run cancel <id>` 로 정리.

### 10-B.2 문제 2 — `actions/cache@v2` 등 deprecated (2024-12-05 발표)

**증상**: 잡 시작 즉시 실패, 메시지:
> Error: This request has been automatically failed because it uses a deprecated version of `actions/cache: v2`.

**원인**: GitHub이 Node 16 기반 `actions/cache` v1/v2 및 구형 toolkit을 폐지. 관련 가이드: https://github.blog/changelog/2024-12-05-notice-of-upcoming-releases-and-breaking-changes-for-github-actions/

**해결 — 두 파일에서 일괄 업그레이드**:
| 액션 | 변경 전 | 변경 후 |
|------|--------|--------|
| `actions/cache` | `@v2` | `@v4` (필수) |
| `actions/checkout` | `@master` | `@v4` (major pin) |
| `actions/setup-python` | `@v2` | `@v5` |
| `ruby/setup-ruby` | `@v1` | 유지 (rolling major) |
| `r-lib/actions/setup-r` | `@v2` | 유지 |

### 10-B.3 참고 — 이 워크플로우는 꼭 필요한가?

- `Website` 워크플로우는 **lesson 레포의 RMD 렌더링·사이트 빌드 검증용**.
- 워크숍 레포는 RMD가 없으므로 대부분의 step이 skip되어 실질 효용 낮음.
- GitHub Pages 자체 빌드가 `gh-pages` 브랜치를 감지해 독립적으로 배포하므로 **워크플로우를 통째로 삭제해도 사이트 가용성에는 영향 없음**.
- 유지 결정: 커뮤니티 관행에 맞춰 존치하되 위 두 패치는 반드시 적용.

---

## 11. 체크리스트 (배포 전 최종 점검)

- [ ] `index.md` front matter 모든 필드 채움 (특히 `humandate` 3글자 월)
- [ ] `eventbrite` 필드는 따옴표로 감싼 문자열
- [ ] Eventbrite 이벤트 **Publish** 상태이며 ID 일치
- [ ] iframe 대신 `eb_widgets.js` 사용
- [ ] `_layouts/workshop.html`의 `<html lang>` 패치 적용
- [ ] 스케줄 표의 시작/종료 시각이 `humantime`과 일치
- [ ] Setup 섹션의 비공개 YouTube 영상 블록 제거 (§10-A)
- [ ] `.github/workflows/*.yml`의 러너·액션 버전 최신화 (§10-B)
- [ ] `bundle exec make workshop-check` 통과 (exit 0)
- [ ] 로컬 서버에서 주요 페이지 수동 확인: `/`, `/aio/`, setup 섹션, Code of Conduct
- [ ] `gh-pages` 브랜치에 push
- [ ] GitHub Pages 배포 후 등록 버튼 클릭 테스트

---

## 12. 세종대 사례 참고 링크

- 저장소: https://github.com/statkclee/2026-06-26-sejong
- 공개 사이트: https://statkclee.github.io/2026-06-26-sejong/
- Eventbrite: https://www.eventbrite.com/e/1988050060247
- 협업 노트: https://pad.carpentries.org/2026-06-26-sejong
- AMY Self-Organised 등록 폼 (공식 워크숍 승인용): https://amy.carpentries.org/forms/self-organised/
- The Carpentries 워크숍 호스팅 가이드: https://carpentries.org/workshops/host-workshop/

---

## 13. 자주 부딪히는 문제

| 증상 | 원인 | 해결 |
|------|------|------|
| `bundle install` 실패: `ffi requires ruby >= 3.0` | Ruby 2.7 사용 | Ruby 3.x 설치 또는 `bundle lock --update=ffi` |
| Eventbrite 등록 박스 자리에 CloudFront 403 | 레거시 `tickets-external` iframe | §5.2의 최신 위젯으로 교체 |
| `make workshop-check` 실패: humandate invalid | 3글자 월 형식 아님 | `"June"` → `"Jun"` |
| 네비바 "Improve this page" 404 (로컬) | 개발 환경에서 `repo_url` 공백 | 무시 — 프로덕션에서 정상 |
| Setup 섹션 Video Tutorial "비공개 동영상" | workshop-template 초기의 6개 YouTube ID가 업로더에 의해 비공개 전환 | §10-A — 업스트림 따라 비디오 블록 제거 |
| Actions 탭의 `Website` 워크플로우가 수시간째 `queued` ("Waiting for a runner…") | `.github/workflows/*.yml`의 `runs-on: ubuntu-20.04` — GitHub이 2025-04-15에 폐지한 러너 이미지 | `ubuntu-22.04` 로 교체 + RSPM URL `focal` → `jammy` + `system_requirements("ubuntu", "22.04")`. 스턱 run은 `gh run cancel <id>` 로 정리. GitHub Pages 자체 배포는 별개로 영향 없음 |
| 빌드 즉시 실패: `This request has been automatically failed because it uses a deprecated version of actions/cache: v2` | GitHub이 2024-12-05 발표로 `actions/cache` v1/v2 + Node 16 기반 액션을 폐지 | 두 워크플로우에서 `actions/cache@v2→v4`, `actions/checkout@master→v4`, `actions/setup-python@v2→v5` 일괄 업그레이드 |
| `/aio/` 페이지 공백 | `_episodes_rmd/`만 존재, `_episodes/` 미빌드 | `make lesson-rmd` (R 환경 필요) 또는 레슨 비통합 사용 |
| 한국어 스크린리더 발음 이상 | `<html lang="en">` 하드코딩 | §7 패치 |

---
> Source: [ai-carpentry/2026-06-26-sejong](https://github.com/ai-carpentry/2026-06-26-sejong) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
