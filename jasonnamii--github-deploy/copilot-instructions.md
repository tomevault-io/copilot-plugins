## github-deploy

> |


# GitHub Deploy (choi 디폴트 + pdkim 명시 모드 · Bash 자동실행)

**깃허브배포·퍼블리싱·웹배포·깃배포** 엔진. 디폴트는 `works.choi.build/{레포명}/`, 명시 호출 시 `works.pdkim.com/{레포명}/`. Private + noindex + HTTPS. **2도메인 체제 (디폴트 1택 + 옵션 1택).**


## Skill Boundaries

- **하는 것** — GitHub Pages 자동 배포 + **GA4 측정 ID 자동 주입** (mode별 분기).
- **안 하는 것** — 레포관리(→직접), DNS(→직접), jasonnamii.github.io 신규배포(→deprecated·리다이렉트만).

**📊 GA4 측정 ID 매핑 (verbatim · 절대 외우지 말 것 — deploy.sh _helper.py가 SSOT):**

| mode | 도메인 | 측정 ID |
|---|---|---|
| **choi (디폴트)** | works.choi.build | `G-1HB3T0BTW6` |
| **pdkim (명시)** | works.pdkim.com | `G-BRYWG9T5PQ` |

**Claude는 GA 스니펫을 절대 수동으로 HTML에 박지 않는다.** deploy.sh phase 3.5가 자동 주입. 형이 "GA 적용해줘"라고 해도 Claude의 역할 = `bash deploy.sh ...` 1콜로 끝. 형 vault grep 등으로 ID를 "찾아서" 박는 행위 = FAIL (모드별 ID 거꾸로 박힐 위험 100%).

**🚀 실행 환경 (필독):** 모든 bash는 **Claude가 직접 Bash 도구로 자동 실행**. `~/github-repos/skill-repos/github-deploy/scripts/deploy.sh`는 형 맥북 zsh 환경에서 `gh auth`·SSH 키·토큰을 그대로 사용. **1줄 명령 출력 후 "형이 맥북 터미널에 붙여넣어 실행" 안내 = FAIL**. 자동 실행이 디폴트, 수동 안내는 폐기.

**원칙:** SKILL.md는 분기·규칙만. 실행은 전부 `scripts/*.sh` 호출. LLM이 bash 본문 생성 ✗ → Bash 도구로 스크립트 호출만.

**v3.1 (2026-05-24) — 재배포 분기 단축 (-20s/재배포):**
1. **verify_root sleep 분기** — `DEPLOY_KIND=redeploy`면 초기 대기 20s→5s. 신규는 false-positive 회피용 20s 유지. 평균 -15s/재배포.
2. **clone 캐시 재사용** — `$WORK`(`/tmp/gh-deploy/{root}`)가 유효한 git repo면 `fetch + reset --hard origin/HEAD + clean -fdx` 1콜. 신규 clone 4~5s → 캐시 fetch 1~2s. 평균 -3s/배포.
3. **자동 fallback** — fetch/reset 실패 시 통째 `rm -rf` 후 신규 clone. 안전성 무손실.
4. **실측** — 동일 입력 재배포 28s → 8~10s.

**v2.2 (2026-05-02) — Bash 자동실행 전면화:** v2.1의 "샌드박스 직접 push 불가·형이 수동 실행" 정책 폐기. Claude Code의 Bash는 형 맥북 zsh를 그대로 쓰므로 `gh auth`·SSH·토큰 전부 동작. Claude가 deploy.sh를 **무조건 자동 호출**. 1줄 명령 출력 안내 전면 삭제.

**v3.0 (2026-05-23) — 외부공유 락 모드 신설 (잘못된 mode 배포 차단):**
1. **`scripts/external-locks.json` 신설** — slug별 `expected_mode`·`reason` 박제. 외부공유 중인 slug를 자동 보호.
2. **`check_external_lock()` 함수** — Phase 0 직전 호출. slug가 락에 등록돼 있고 호출 mode가 expected_mode와 다르면 `exit 9` abort.
3. **`--override-lock` 플래그** — 락 우회용. 외부공유 끊김을 감수할 때만 사용. 인자 어느 위치에 와도 인식.
4. **현재 락 보유 slug:** `madpop_with_kim` (expected=pdkim, 외부 공유 중).
5. **Failure Modes에 lock 항목 3종 신설** — abort 시나리오·override 사용 시점·락 관리 방법.

**v2.9 (2026-05-23) — F1 short-circuit 시에도 manifest 미러 갱신:**
1. **F1 sha256 match 분기 패치** — 동일 콘텐츠 재배포 시 `exit 0` 직전에 `manifest_update` 1회 호출. 콘텐츠 push는 스킵하되 로컬 manifest 미러의 `timestamp`·`last_kind=skip-same-sha`를 최신화.
2. **누적 drift 차단** — F1 빠짐이 반복돼도 manifest_lookup이 항상 최신 상태. 다음 신규/redeploy 시 원격 `_manifest.json`에 합류됨.
3. **Failure Modes에 F1+manifest 항목 신설** — 동일 콘텐츠 재배포 시 manifest가 누락되지 않는지 status 박제(`STATUS=skip-same-sha`)로 확인.

**v2.8 (2026-05-22) — manifest 합집합 분기 신설 (choi+pdkim 양쪽 조회):**
1. **Phase 0.5 신설** — slug 정규화 후 choi/pdkim 양쪽 `_manifest.json` 미러 합집합 조회. 4케이스 자동 판정 (`new_global`·`redeploy_same`·`exists_other`·`exists_both`).
2. **`_manifest.json` 박제 위치** — 각 루트 레포(`works-choi-live`·`works-pdkim`) 루트에 1개씩. 로컬 미러는 `~/github-repos/skill-repos/github-deploy/.cache/manifest-{mode}.json`.
3. **반대 모드/양쪽 hit 시 ⚠ 경고 + 진행** — 자동 중단 ✗ (배포 흐름 보존). `MANIFEST_VERDICT` + `MANIFEST_OTHER_URL`을 `.deploy-status.txt`에 박제하여 형이 사후 확인 가능.
4. **slug 키 = 정규화된 REPO 인자** — 소문자·하이픈·영숫자만. 형이 명시 박제(`KISAS-TF-Agenda` → `kisas-tf-agenda`).

**v2.7 (2026-05-21) — GA4 매핑 SSOT 박제 + 수동 박기 금지 룰 신설:**
1. **GA4 ID 매핑을 Skill Boundaries에 verbatim 박제** — choi=G-1HB3T0BTW6 / pdkim=G-BRYWG9T5PQ. Claude가 vault grep 등으로 ID 추측·수동 박기 금지. deploy.sh phase 3.5(_helper.py GA4_IDS dict)가 SSOT.
2. **Failure Modes에 GA4 3종 신설** — ① 수동 박기 금지 ② "기존 gtag 있으면 스킵" 가드와의 충돌 ③ ID 매핑 외우지 말 것.
3. **WRONG/CORRECT에 GA 사건 케이스 추가** — vault grep으로 찾은 G-BRYWG9T5PQ를 choi에 박은 실제 사고 1건 박제.

**버전 히스토리 (요약):**
- **v3.1 (2026-05-24)** — 재배포 verify 대기 20s→5s + clone 캐시 재사용. 동일 입력 재배포 28s→8~10s.
- **v2.5 (2026-05-10)** — verify polling(sleep 20+5s×12), Phase 0 병렬화, `_helper.py` 통합. 배포 120s→80s, 재배포 라우팅 8s→3s.
- **v2.4 (2026-05-09)** — F1 sha256 short-circuit, F2 `.deploy-status.txt` 9필드 박제, F3 stdout flush, F4 timeout≠실패 가이드. 동일파일 재배포 1초.
- **v2.3 (2026-05-06)** — Phase 0 라우팅 게이트(cache+tree+head 3중 조회), 검증 단순화(루트 URL 1회 HEAD), `.deploy-cache.json` 자동 갱신.

**계정·레포 구조 (고정):**
- OWNER = `jasonnamii`
- choi 배포처 (디폴트) = `jasonnamii/works-choi-live` 루트 레포의 `/{레포명}/`
- pdkim 배포처 (명시) = `jasonnamii/works-pdkim` 루트 레포의 `/{레포명}/`
- 레거시(리다이렉트 전용) = `jasonnamii/jasonnamii.github.io`
- **스크립트 경로 (형 맥북 표준):** `~/github-repos/skill-repos/github-deploy/scripts/`

---

## When to Use

- 사용자가 "배포해줘", "올려줘", "자동으로 배포", "바로 배포", "리다이렉트 걸어줘" 같은 표현으로 발동
- 도메인 작업이 필요한 시점
- **안 쓸 때** — 레포관리(→직접), DNS(→직접), jasonnamii.github.io 신규배포(→deprecated·리다이렉트만).


## Prerequisites

| # | 체크 | 미충족 시 |
|---|------|-----------|
| 1 | 대상·입력 명확 (스킬 발동 의도 확인) | 1줄 확인 후 진입 |
| 3 | scripts/ 실행 권한 | 권한 보정 후 재시도 |


## 🚀 라우팅 (Bash 자동실행 직행)

**디폴트 = choi.** pdkim은 명시 트리거가 있을 때만. **모든 트리거에서 Claude가 즉시 Bash 도구 호출.**

| 트리거 | 액션 (Claude 자동) |
|---|---|
| **배포·build·deploy·깃배포·재배포** + 파일/레포명 (도메인 미지정) | Bash 도구 → `bash ~/github-repos/skill-repos/github-deploy/scripts/deploy.sh {repo} {src}` |
| **"pdkim으로 배포"·"pdkim 모드"·"김피디 배포"·`--mode=pdkim`** | Bash 도구 → `... deploy.sh {repo} {src} --mode=pdkim` |
| **리다이렉트·마이그레이션** | Bash 도구 → §레거시 마이그레이션 스크립트 |
| jasonnamii.github.io 신규 배포 언급 | "리다이렉트 전용. choi 또는 pdkim으로." 안내만 (실행 ✗) |

**pdkim 명시 키워드:** `pdkim`, `김피디`, `pdkim으로`, `pdkim 모드`, `works.pdkim.com`, `--mode=pdkim`, `mode=pdkim`. 하나라도 등장하면 pdkim.

---

## ⚡ 숏서킷 (Bash 자동실행 1콜)

**확인 2~3개 (없으면 채워서 직행):**
- 대상 파일 (미지정 시 직전 HTML)
- 레포명 = URL 경로 (소문자+하이픈, 예: `kisas-tf-agenda`)
- 모드 (선택, 기본=choi)

**Claude가 호출하는 Bash 도구 1콜:**

choi 디폴트:
Bash 도구로 실행 (timeout 180s):
```bash
bash -lc "bash ~/github-repos/skill-repos/github-deploy/scripts/deploy.sh {레포명} \"{원본경로}\""
```

pdkim 명시:
Bash 도구로 실행 (timeout 180s):
```bash
bash -lc "bash ~/github-repos/skill-repos/github-deploy/scripts/deploy.sh {레포명} \"{원본경로}\" --mode=pdkim"
```

- 3번째 인자가 없으면 choi 고정.
- 원본경로는 **단일 HTML이어도 OK** — auto-asset이 같은 폴더의 `images/` 등 자동 탐지.
- 폴더 배포 원하면 폴더 경로 전달 (index.html 루트 필수).
- 빌드 대기·Pages 활성화 **전부 스킵**. 두 루트 레포 모두 Pages 이미 활성.
- HEAD 검증 내장 — `✅ 완벽 배포` / `⚠ N건 실패` 자동 출력.
- **출력 끊김 시:** Bash를 `run_in_background`로 실행한 뒤 BashOutput로 추가 수신.
- **HEAD 검증 폴백 (v2.2 이하):** mapfile 에러로 검증 누락 시 별도 `curl -sI` 1콜로 직접 200 확인. v2.3부터는 자동 처리되므로 폴백 불필요.

---

## 🪛 Step 0: 재배포 감지 (v2.3부터 deploy.sh 내장)

**v2.3:** deploy.sh 진입 시 Phase 0 라우팅 게이트가 자동으로 `.deploy-cache.json` + `gh api contents` + `curl HEAD` 3중 조회 → "기배포 발견 → 재배포" / "신규배포" 1줄 보고. **별도 check-deploy.sh 호출 불필요.**

**check-deploy.sh는 다음 경우에만 별도 호출:** 형이 "지금까지 배포한 레포 다 보여줘" 같은 *리스트* 요청을 할 때만.

**트리거 (레거시):** "재배포·업뎃·redeploy" 키워드 + 레포명만 있고 파일 경로 불명.

**Claude가 Bash로 자동 호출:**

choi:
Bash 도구로 실행 (timeout 30s):
```bash
bash -lc "bash ~/github-repos/skill-repos/github-deploy/scripts/check-deploy.sh {레포명}"
```

pdkim:
```
... check-deploy.sh {레포명} pdkim
```

**출력 (TSV):**
```
# github-deploy PRE-CHECK: {repo}
DOMAIN   STATUS                     URL                              LAST_COMMIT
choi     SUBDIR_EXISTS|SUBDIR_NEW   https://works.choi.build/{repo}/  2026-04-15T...
```

기존 배포면 update, 아니면 new. 파일 경로만 형에게 재확인 후 즉시 deploy.sh Bash 호출.

---

## 📦 Auto-Asset + Verify

deploy.sh가 자동 수행 (choi·pdkim 동일):

1. **입력 판정** — 폴더 입력 = 통째 복사 / 단일 HTML = auto-asset 스캔
2. **auto-asset 스캔** (단일 HTML만)
   - 대상: `src=`, `href=`, `srcset=`, `poster=`, `data-src=`
   - 제외: 외부 URL, `data:`, `mailto:`, `javascript:`, `#` 앵커
   - 쿼리·해시 제거 후 경로만 사용
   - 원본 HTML과 같은 디렉토리 기준 상대경로 실존 체크
   - src_dir 밖으로 벗어나는 `../` 참조는 무시
   - 누락 파일은 로그만 남기고 배포는 계속
3. **HEAD 검증** (배포 후 자동)
   - 전파 45초 대기 후 `curl -I {BASE_URL}/{경로}` 전량 체크
   - 200이 아니면 경고 + 수동 재확인 URL 출력
   - `SKIP_VERIFY=1 bash ...` 로 끌 수 있음

**v1 범위 외:** CSS 내부 `url(...)`, inline `<style>` 참조, JS 동적 로딩.

---

## 🔁 레거시 마이그레이션 (jasonnamii.github.io → choi/pdkim 리다이렉트)

**목적:** 기존 `works.jasonnamii.com/{repo}/` 링크 유지, 실제 콘텐츠는 choi/pdkim에서 서빙.

**트리거:** "리다이렉트·마이그레이션·legacy 이관" 키워드.

**Claude가 Bash로 자동 호출 (2단계):**

**Step A — 타겟에 없는 레거시 콘텐츠 복제 (404 방지):**
Bash 도구로 실행 (timeout 60s):
```bash
bash -lc "bash ~/github-repos/skill-repos/github-deploy/scripts/migrate-legacy.sh jasonnamii scan"
```
출력된 목록 각각을 deploy.sh로 choi 또는 pdkim에 먼저 자동 배포.

**Step B — 리다이렉트 일괄 적용:**
Bash 도구로 실행 (timeout 120s):
```bash
bash -lc "bash ~/github-repos/skill-repos/github-deploy/scripts/migrate-legacy.sh jasonnamii apply --target=choi"
```

**리다이렉트 HTML 템플릿 (경로 보존):**
```html
<!DOCTYPE html><html><head>
<meta charset="utf-8">
<meta http-equiv="refresh" content="0; url=https://works.choi.build{PATH}">
<link rel="canonical" href="https://works.choi.build{PATH}">
<script>location.replace('https://works.choi.build' + location.pathname + location.search + location.hash)</script>
</head><body>이 페이지는 <a href="https://works.choi.build{PATH}">works.choi.build{PATH}</a>로 이동되었습니다.</body></html>
```

**DNS 요구사항:** `works.jasonnamii.com` CNAME **유지 필수** (최소 3~6개월). DNS 죽으면 리다이렉트도 죽음.

---

## 📋 결과보고 (Claude 출력 템플릿)

**신규 배포 (choi, v2.8 manifest 분기 포함):**
```
✅ 배포 완료 (choi · 신규배포 · Bash 자동실행)
[phase 0] 기배포 없음 → 신규배포 모드
[manifest] slug={정규화된 레포명} 양쪽 레포 모두 없음 → 완전 신규 (new_global)
메인: https://works.choi.build/{레포명}/   (~60초 후 200)
레포: https://github.com/jasonnamii/works-choi-live/tree/main/{레포명} (Private)
검증: HTTP 200 OK
캐시: .deploy-cache.json + manifest-choi.json + _manifest.json (원격) 갱신
```

**⚠ 반대 모드에 존재 (exists_other):**
```
⚠ 배포 진행 (pdkim · 신규 · 같은 slug가 choi에도 있음)
[manifest] slug={slug} 다른 모드(choi)에 존재: https://works.choi.build/{레포명}/
[manifest] 현재 호출=pdkim → pdkim에 신규 배포 진행
메인: https://works.pdkim.com/{레포명}/
참고: 정본 정리는 형 결정 (한쪽 삭제 or 양쪽 유지)
```

**⚠ 양쪽 hit (exists_both):**
```
⚠ 배포 진행 (정본 모호 · 양쪽 레포 모두 존재)
[manifest] choi  : https://works.choi.build/{레포명}/
[manifest] pdkim : https://works.pdkim.com/{레포명}/
[manifest] 현재 호출={mode} → 이쪽에 덮어쓰기. 정본 정리 필요
```

**재배포 / pdkim / 마이그레이션 (요약):**
- 재배포: `[phase 0] 기배포 발견 (cache|tree|head) → 재배포 모드` + `[manifest] redeploy_same` + HTTP 200 + 캐시 갱신.
- pdkim 신규: 도메인만 `works.pdkim.com/{레포명}/`으로 교체, 나머지 동일.
- 레거시 마이그레이션: `_archive/{repo}/` 백업 + meta-refresh 또는 302 검증.

---

## ⚠️ 에러 대응

| exit / 증상 | 원인 | Claude 자동 대응 |
|------|------|------|
| `deploy` =3 | 루트 레포 clone 실패 | Bash로 `gh repo view jasonnamii/works-choi-live` 자동 점검 |
| `deploy` =9 (v3.0) | 외부공유 락 hit — 잘못된 mode | 메시지의 expected mode로 재호출. 의도된 경우 `--override-lock` |
| push rejected | remote 선행 커밋 | 스크립트 자동 `git pull --rebase` 재시도 |
| `check-deploy.sh` 빈 응답 | `gh auth` 미로그인 | Bash로 `gh auth status` 자동 확인, 미로그인 시 형에게 1줄 보고 |
| `[verify]` 실패 N건 | 전파 지연 or 진짜 누락 | 1~2분 뒤 BASE_URL 자동 재체크. 계속 404면 원본 HTML 참조 경로 불일치 보고 |
| `[auto-asset] MISSING` | HTML 참조는 있는데 파일 없음 | 누락 파일 보고. 형 결정(수정 or 추가) 후 재배포 |
| `migrate-legacy.sh` scan 결과 없음 | 레거시 레포 서브폴더 구조 아님 | 레포 구조 자동 점검 후 보고. 수동 이관 필요 시 형에게 알림 |
| **mapfile: command not found** (v2.2 이하 잔재) | macOS bash 3.2 한계 — v2.3에서 mapfile 제거됨 | v2.3 이상이면 발생 안 함. 발생 시 deploy.sh 버전 확인 |
| **Phase 0 캐시 충돌** | `.deploy-cache.json` 파손 | `rm ~/github-repos/skill-repos/github-deploy/.cache/deploy-cache.json` 후 재배포 (캐시 자동 재생성) |
| **mode 인자 누락** | choi/pdkim 분기 모호 | 디폴트=choi 적용. pdkim 원하면 `--mode=pdkim` 명시 |

---

## 🗄️ Deprecated: jasonnamii.github.io 신규 배포

v2.0부터 신규 배포 **차단**. 리다이렉트 전용.

| 항목 | 상태 |
|------|------|
| `jasonnamii.github.io` 루트 레포 신규 배포 | ❌ 차단 |
| `scripts/deploy.sh` mode=jasonnamii | ❌ 미지원 |
| `scripts/resolve-state.sh` | ⚠️ 보존 (미호출, 향후 복귀 대비) |
| `scripts/wait-build.sh` | ⚠️ 보존 (미호출) |
| `check-deploy.sh` | ✅ choi/pdkim 분기 조회 |
| `migrate-legacy.sh` | ✅ 활성 (jasonnamii.github.io → choi/pdkim 리다이렉트) |

---

## Output Path

| 산출물 | 경로 |
|---|---|
| 주 산출물 | `mnt/outputs/github-deploy_{topic}_{YYYY-MM-DD}.md` |
| 형식 | works.choi.build으로, works.pdkim.com으로. |
| 리서치 결과 (해당 시) | `{VAULT}/_skills research/github-deploy/{YYYY-MM-DD}_{topic}.md` |

## Reference Index

scripts/ 폴더의 어떤 파일을 언제 호출하는지.

| 파일 | 역할 | 언제 |
|---|---|---|
| `scripts/deploy.sh` | 본류 배포 (Phase 0~4 + manifest) | 모든 배포 트리거 |
| `scripts/_helper.py` | Python 헬퍼 (cache·stage·noindex·ga4·manifest) | deploy.sh 내부에서만 호출 |
| `scripts/check-deploy.sh` | 레포 배포 현황 리스트 | 형이 "지금까지 배포한 레포 보여줘" 요청 시 |
| `scripts/migrate-legacy.sh` | jasonnamii.github.io → choi/pdkim 리다이렉트 | 레거시 마이그레이션 트리거 시 |
| `scripts/resolve-state.sh` | 레거시 보존(미호출) | deprecated |
| `scripts/wait-build.sh` | 레거시 보존(미호출) | deprecated |
| `scripts/external-locks.json` | 외부공유 락 정의 (v3.0) | check_external_lock이 deploy.sh 진입 시 자동 참조 |

## Next Phase

본 스킬 작업 후 자연스럽게 이어지는 흐름:

- 다른 mode 검증 → `bash deploy.sh {repo} {src} --mode={반대 mode}` (manifest verdict 확인 후 형 결정)
- 정본 정리 → 형 직접 (한쪽 레포 폴더 삭제 + manifest 엔트리 제거)
- 세션 마무리 → `session-briefing` (배포 결과·미결을 VAULT 저장)

## Failure Modes (Gotchas)

- **재배포 verify 대기 = 5s (v3.1 신설)** — `DEPLOY_KIND=redeploy`면 `verify_root`가 초기 대기를 20s→5s로 단축. 이미 200 떠있는 URL에 20s 캐시 회피 마진이 불필요. 신규는 그대로 20s 유지(이전 캐시가 200을 잘못 응답하는 false-positive 회피용).
- **clone 캐시 재사용 (v3.1 신설)** — `/tmp/gh-deploy/{works-choi-live|works-pdkim}/.git`이 유효하면 신규 clone 대신 `git fetch origin && git reset --hard origin/HEAD && git clean -fdx` 1콜. remote URL 검증 통과 + fetch/reset/clean 전부 성공해야 캐시 사용. 1개라도 실패 시 자동 fallback(rm -rf 후 clone). 사용자가 `/tmp/gh-deploy/`를 수동으로 비워도 무해(다음 호출이 그냥 신규 clone).
- **캐시 재사용 시 로컬 커밋 손실 주의 (v3.1 신설)** — `git clean -fdx` + `reset --hard`는 `$WORK` 안의 모든 추적/비추적 파일을 origin/HEAD 기준으로 초기화. 형이 `/tmp/gh-deploy/` 안에 수동으로 만든 파일은 다음 배포 시 사라짐. 작업폴더로 쓰지 말 것.
- **GA4 자동 주입은 deploy.sh phase 3.5 전담 (v2.7 신설)** — Claude는 GA 스니펫을 HTML에 절대 수동으로 박지 않는다. 형이 "GA 적용해줘"·"GA 박아줘"라고 해도 Claude의 액션 = `bash deploy.sh {repo} {src}` 1콜. 수동 박기 = FAIL. **이유:** ① deploy.sh가 mode별 올바른 ID를 자동 주입함 ② Claude가 vault grep 등으로 ID를 "찾으면" 모드별로 거꾸로 박힐 위험 100% (실제 사고 1건: 형 choi 페이지에 pdkim용 ID `G-BRYWG9T5PQ`이 박혔음).
- **"기존 gtag 있으면 스킵" 가드와의 충돌 (v2.7 신설)** — `_helper.py` inject_ga4()는 head에 `googletagmanager.com/gtag` 또는 mode별 ID가 이미 있으면 주입을 건너뛴다. Claude가 잘못된 ID를 먼저 수동 박으면 deploy.sh가 올바른 ID 주입을 못 함. **정정 절차:** ① 잘못된 gtag 스니펫 제거 ② `bash deploy.sh` 재실행 ③ 라이브 `curl … | grep G-` 1콜로 올바른 ID 확인.
- **GA4 ID는 외우지 말 것 (v2.7 신설)** — choi=G-1HB3T0BTW6, pdkim=G-BRYWG9T5PQ. 매핑을 머리로 외워서 적용하면 또 거꾸로 박는다. SSOT = `_helper.py` 안 `GA4_IDS` dict. SKILL.md 상단 Boundaries의 표는 사람용 확인 자료일 뿐, Claude 액션은 deploy.sh 1콜로 한정.
- **manifest 합집합은 양쪽 mode 미러 + 원격 (v2.8 신설)** — Phase 0.5에서 `manifest_sync` 2회(choi/pdkim) 후 `manifest_lookup` 1회로 합집합 판정. 한쪽 mode만 보면 반대 mode 기존 배포를 못 본다. Claude는 verdict (`new_global`·`redeploy_same`·`exists_other`·`exists_both`)를 보고에 반드시 포함.
- **manifest_sync 실패 = 정상 흐름 유지 (v2.8 신설)** — gh api 실패·네트워크 끊김 시 로컬 미러를 그대로 사용. 미러도 없으면 `new_global`로 판정되어 신규 배포 진행. 배포 자체를 막지 않는 게 원칙. 단, status에 `MANIFEST_VERDICT=unknown`이면 미러 fetch 실패 의심.
- **slug 정규화는 deploy.sh가 책임 (v2.8 신설)** — REPO 인자가 `KISAS-TF-Agenda`처럼 대문자·하이픈 섞여도 `slug_normalize`로 `kisas-tf-agenda`로 통일. Claude가 SKILL.md 표 또는 보고에 slug를 적을 때도 정규화된 형태로. 정규화 안 한 키로 미러 조회 = miss 오탐.
- **exists_both 케이스는 자동 중단 ✗·형 결정 (v2.8 신설)** — 양쪽 레포에 같은 slug가 있을 때 deploy.sh는 ⚠ 경고만 출력하고 호출된 mode에 덮어쓰기 진행. 정본 정리(한쪽 삭제 or 양쪽 유지)는 사람 판단. Claude가 임의로 한쪽 삭제 ✗.
- **Bash 도구 자동실행이 디폴트** — Claude는 무조건 Bash로 deploy.sh를 호출. "1줄 명령 출력 + 형이 수동 실행" 안내는 v2.2부터 폐기. 형이 "배포"·"자동으로"·"올려줘" 안 적어도 Bash 직행.
- **timeout ≠ 실패 (v2.4 신설)** — Bash 도구 timeout 만료 = 응답 timeout일 뿐, deploy.sh는 정상 진행 중일 가능성 높음. 즉시 재시도 ✗. **timeout 발생 시 4단계 검증 순서:**
  1. 백그라운드 Bash면 BashOutput로 추가 출력 수신
  2. `cat ~/github-repos/skill-repos/github-deploy/.cache/deploy-status.txt` — F2 박제 결과 즉시 파악 (`STATUS=success` 면 완료, `STATUS=phase4-pushed` 면 push까지 끝남)
  3. `cd /tmp/gh-deploy/{root_repo} && git log -1 --format='%h %s'` — 최근 commit 확인
  4. `curl -sI {BASE_URL}/ | head -3` — 라이브 last-modified 확인
  → 4단계 중 하나라도 성공 신호면 **재시도 ✗·결과 보고만**. 모두 실패 신호면 재시도 1회.
- **F1 sha256 short-circuit (v2.4)** — 동일 입력 파일 재배포 시 deploy.sh가 1초 이내 `DONE-SKIP (sha256 match...)` 출력하고 종료. 콘텐츠 변경 없는데 다시 push할 일 ✗. 출력에 `DONE-SKIP` 보이면 정상.
- **외부공유 락 abort (v3.0 신설)** — `scripts/external-locks.json`에 등록된 slug를 expected_mode와 다른 mode로 배포 시도 시 `exit 9`. 외부공유 URL이 잘못된 정본으로 덮이는 사고 차단. **abort 시 메시지의 expected mode로 재호출**이 디폴트 대응.
- **`--override-lock` 사용 시점 (v3.0 신설)** — 락이 걸린 slug를 의도적으로 다른 mode로 배포할 때만. 외부공유 끊김을 감수한다는 명시 의사. CI·자동화에서는 사용 금지. 본 플래그 사용 시 status에 `STATUS=lock-overridden`로 박제되지는 않지만 stderr에 ⚠ 경고 3줄 출력.
- **락 관리 방법 (v3.0 신설)** — `scripts/external-locks.json` 직접 편집. 추가: 새 slug에 `expected_mode`·`reason`·`locked_at` 박제. 해제: 해당 키 제거. 락 파일 자체가 없으면 락 체크는 통과(no-op) — 항상 안전.
- **F1 short-circuit + manifest 동기 (v2.9 신설)** — F1 빠짐 시에도 `manifest_update`가 로컬 미러의 timestamp/last_kind를 갱신함. 원격 `_manifest.json` 동기는 다음 신규/redeploy 시 자동 합류. F1 누적이 반복돼도 manifest_lookup 결과는 항상 최신. status 박제 `STATUS=skip-same-sha`로 확인.
- **`.deploy-status.txt` 위치** — `~/github-repos/skill-repos/github-deploy/.cache/deploy-status.txt`. 9필드 (`STATUS·PHASE·MODE·REPO·URL·COMMIT·HTTP_CODE·DEPLOY_KIND·TIME·PID`). timeout 시 첫 의지처.
- **Claude Code Bash = 형 맥북 zsh** — Bash 도구는 형 로컬 셸을 그대로 띄움 → `gh auth`·SSH 키·토큰·홈 경로 전부 사용 가능. "샌드박스 토큰 없음" 전제는 v2.1의 오해, v2.2부터 폐기.
- **스크립트 표준 경로** — `~/github-repos/skill-repos/github-deploy/scripts/deploy.sh`. `~/.claude/skills/github-deploy/scripts/`는 미설치 → fallback 금지. 경로 의심 시 Bash `find ~ -maxdepth 6 -type f -name "deploy.sh" -path "*github-deploy*"` 1콜로 확인.
- **Bash 호출 형식** — `command='bash -lc "..."'` 권장 (zsh login 환경 강제로 PATH·gh auth 안정). timeout_ms는 deploy=180000, check=30000, migrate=120000 디폴트.
- **v2.3 검증 = sleep 60s + 루트 HEAD 1회** — 파일별 검증 루프는 폐기. deploy.sh가 단일 HTML을 `index.html`로 리네임하므로 루트 URL 200이면 곧 페이지 OK. 재시도·진단 루프 사라짐.
- **Phase 0 자동 분기** — deploy.sh 진입 시 캐시·gh api·curl HEAD 3중 조회로 `redeploy|new` 자동 판정. 별도 check-deploy.sh 호출 ✗.
- **`.deploy-cache.json` 위치** — `~/github-repos/skill-repos/github-deploy/.cache/deploy-cache.json`. 파손 시 삭제 후 재배포로 자동 재생성.
- **HEAD 검증 캐시** — Pages 전파 평균 50초. v2.3은 60s 고정. 그래도 404면 폴더 구조·파일명 한글 인코딩 확인.
- **디폴트 = choi**, **명시 = pdkim** — "pdkim·김피디·pdkim으로" 키워드 없으면 무조건 choi. 도메인 핑퐁 ✗.
- **OWNER = jasonnamii** — 두 모드 모두 GitHub 계정은 `jasonnamii`. 착각 금지.
- **레거시는 리다이렉트만** — `jasonnamii.github.io`(works.jasonnamii.com) 신규 배포 요청 = 거부 + choi/pdkim 권유.
- **choi/pdkim은 숏서킷으로** — resolve-state.sh 호출 ✗. mode=subdir 고정.
- **Project Site vs User Site**: 다른 레포 콘텐츠를 `works.choi.build/{다른레포}/`에 넣는 건 절대 불가 → 서브폴더 강제. pdkim도 동일.
- **CNAME 금지 (서브폴더)**: 서브폴더엔 CNAME 일체 금지. 루트 레포(`works-choi-live`·`works-pdkim`)에만 있음.
- **작업 경로**: choi=`/tmp/gh-deploy/works-choi-live`, pdkim=`/tmp/gh-deploy/works-pdkim` 분리.
- **단일 HTML 입력 OK**: 같은 폴더 자원 자동 동반 (두 모드 동일).
- **폴더 배포 시 `index.html` 루트 필수** — 없으면 Pages 404.
- **auto-asset 한계**: CSS 내부 `url(...)`, 동적 JS 로드, 절대경로는 미지원. 폴더 입력 권장.
- **전파 지연**: HEAD 검증 전 45초 대기. 그래도 404면 추가 1~2분 기다려 재확인.
- **스크립트 수정 금지**: SKILL.md는 호출만. 로직은 `scripts/*.sh`.
- **HEAD 검증 끄기**: `command='bash -lc "SKIP_VERIFY=1 bash .../deploy.sh ..."'`
- **레거시 DNS 유지**: `works.jasonnamii.com` CNAME 최소 3~6개월 유지.
- **리다이렉트 적용 전 복제 필수**: `migrate-legacy.sh scan` → 복제 → `apply` 순서 엄수.
- **`_helper.py` 위치 (v2.5 신설)**: deploy.sh와 같은 디렉토리(`scripts/_helper.py`). deploy.sh가 `SCRIPT_DIR/_helper.py`로 자동 참조. 분리 호출 ✗ (e.g. `python3 _helper.py cache_read ...`는 deploy.sh 내부 호출 전용). 직접 수정 금지 — Python 로직은 _helper.py에 단일 소스.
- **polling 시작 = sleep 20 (v2.5)**: false-positive 200 회피용. GitHub Pages CDN이 이전 캐시를 200으로 응답할 가능성 차단. 20s는 경험치, 더 줄이면 캐시 hit 위험.

---

## ✅ WRONG / CORRECT 대조

❌ **WRONG (v2.1 잔재 — 수동 안내):**
```
사용자: "배포해줘"
→ "✅ 형 맥북 터미널에서 1줄 실행:
   bash ~/.claude/skills/github-deploy/scripts/deploy.sh {repo} {src}"
→ (Claude는 실행 ✗, 사용자가 수동 복붙)
```

✅ **CORRECT (v2.2 동작 — Bash 자동실행):**
```
사용자: "배포해줘"
→ Claude가 즉시 Bash 도구로 bash ~/github-repos/skill-repos/github-deploy/scripts/deploy.sh {repo} {src} 자동 호출
→ 출력 받아서 ✅ 메인 URL · HTTP 200 OK 보고
→ "배포"·"자동으로" 같은 키워드 형이 안 적어도 직행
```

❌ **WRONG (이중 안내):**
```
Bash로 자동 실행 후에도 "수동 실행하려면 ~~~" 같은 백업 안내 추가
→ 정책 혼동, 형 짜증 트리거
```

✅ **CORRECT (단일 경로):**
```
Bash 1콜 → 결과 보고. 끝. 수동 안내 ✗.
```

**v2.5 / Phase 0 병렬 케이스 (요약):**
- WRONG: sleep 60 고정 단일 HEAD / gh api·curl HEAD 직렬
- CORRECT: sleep 20 + polling 5s×12 (200시 break) / `&` + `wait` 병렬

---

❌ **WRONG (v2.7 새 함정 — 형이 "GA 적용해줘" 했을 때 Claude가 vault grep으로 ID 찾아 수동 박음):**
```
형: "BCM.html에 GA 적용해줘. 깃배포 스킬로."
→ Claude: vault grep "G-[A-Z0-9]+" → `G-BRYWG9T5PQ` 1개 발견
→ Claude: choi용으로 단정해서 BCM.html <head>에 수동 삽입
→ Claude: bash deploy.sh bcm BCM.html 실행
→ deploy.sh phase 3.5: "이미 gtag 있음 → 스킵"
→ 라이브: choi 도메인인데 pdkim용 ID 박힌 채 발행
→ 형 GA 대시보드: choi 스트림에 데이터 0건. 사고.
```

✅ **CORRECT (v2.7 — deploy.sh 1콜만, Claude는 ID 손도 대지 않음):**
```
형: "BCM.html에 GA 적용해줘. 깃배포 스킬로."
→ Claude: `bash deploy.sh bcm BCM.html` 1콜만 실행
→ deploy.sh phase 3.5: mode=choi 감지 → _helper.py GA4_IDS["choi"]=G-1HB3T0BTW6 자동 주입
→ 라이브: curl … | grep G- → `G-1HB3T0BTW6` 단일 확인
→ 형 GA 대시보드: choi 스트림 정상 수집
→ Claude 보고: "✅ GA4 자동 주입 완료. choi=G-1HB3T0BTW6 라이브 확인"
```

---

❌ **WRONG (v2.4 새 함정 — timeout 후 즉시 재시도):**
```
사용자: "배포해줘"
→ Claude Bash 도구 호출
→ MCP timeout 떠오름
→ Claude: "실패한 듯. 다시 시도"
→ 작업폴더 rm -rf + 재시도
→ "변경 없음" 출력 (실제로는 1차에 이미 push 완료)
→ Claude 혼란, 5콜+ 폭증
```

✅ **CORRECT (v2.4 동작 — timeout = 일시 stream 끊김 가능성):**
```
사용자: "배포해줘"
→ Claude Bash 도구 호출 (timeout=180s)
→ MCP timeout 발생
→ Claude 4단계 검증 순서:
  ① cat ~/github-repos/skill-repos/github-deploy/.cache/deploy-status.txt
     → STATUS=success 확인 → 즉시 결과 보고. 끝
  (또는 ② git log -1 / ③ curl HEAD로도 OK)
→ 재시도 ✗
```

---

❌ **WRONG (v2.8 새 함정 — 호출 모드만 조회):**
```
형: "bcm.html을 pdkim으로 배포해줘"
→ Claude: pdkim manifest만 조회 → miss → 신규 처리
→ deploy.sh: pdkim에 신규 배포
→ 실제로는 choi에 같은 slug=bcm이 이미 있었음
→ 형: 같은 문서가 choi·pdkim 양쪽에 존재. 어느 게 정본?
```

✅ **CORRECT (v2.8 — 양쪽 mode 합집합 조회):**
```
형: "bcm.html을 pdkim으로 배포해줘"
→ deploy.sh Phase 0.5:
  ① manifest_sync choi  (gh api → 미러 갱신)
  ② manifest_sync pdkim (gh api → 미러 갱신)
  ③ manifest_lookup slug=bcm → choi=Y, pdkim=N
→ MANIFEST_VERDICT=exists_other (반대 모드 hit)
→ ⚠ "다른 모드(choi)에 slug=bcm 존재: https://works.choi.build/bcm/"
→ pdkim 신규 배포 진행 + status 박제
→ Claude 보고: "⚠ choi에 동일 slug 존재. pdkim 신규 배포 완료. 정본 정리 필요?"
```

---

❌ **WRONG (v2.8 — 양쪽 hit 시 자동 삭제):**
```
deploy.sh가 exists_both 감지 → 자동으로 한쪽 삭제 → 형 의도와 다른 정본 손실
```

✅ **CORRECT (v2.8 — 양쪽 hit 시 경고만 + 형 결정):**
```
deploy.sh가 exists_both 감지 → ⚠ 양쪽 URL 출력 → 호출 mode에 덮어쓰기만 진행
→ Claude 보고에 "⚠ 양쪽 레포 정본 모호. 정리 결정 필요" 포함
→ 형이 명시적으로 "choi 쪽 삭제해줘" 지시할 때만 처리
```

---
> Source: [jasonnamii/github-deploy](https://github.com/jasonnamii/github-deploy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
