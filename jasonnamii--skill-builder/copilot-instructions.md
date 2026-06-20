## skill-builder

> 스킬을 생성·수정·검증·패키징하는 1턴 완결 게이트키퍼. 호스트 캐시 직접 편집·VM/호스트 이중환경 분기·N사본 동기·9룰 베놈 강제·.skill 패키징을 한 흐름으로 처리한다.


# skill-builder v5.0

**1턴 완결.** ⓪ 이중환경 게이트 → ① 호스트 캐시 직접 편집 → ② N사본 동기 → ③ 검증 → ④ (옵션) 패키징.

**v5.0 이중환경 모델 (2026-05-24):** v4.0의 "캐시-직접" 명제는 유지하되 **VM/호스트 환경 분리 가정 사고를 정정**. 실측 5건: ① VM Bash로 `/var/folders/...` 호스트 캐시 = **접근 불가**(VM 안에서 안 보임) ② VM에 보이는 건 `/sessions/.../mnt/.claude/skills/` bindfs **read-only** 마운트 1곳 ③ "7사본·sha 3종 혼재" 검증 = **VM에서 실행 불가**(N사본 다 못 봄) ④ bindfs 권한으로 cp·zip 직접 쓰기 `Operation not permitted` ⑤ zip `-x "_archive/*"` 패턴 무력화. 따라서 **편집 = 호스트 Edit 도구·검증·패키징 = VM Bash** 도구 분리가 §⓪ 진입 첫 게이트로 박힌다.

**v4.0 (archived):** "캐시 = 형 하드 동일파일" 명제 자체는 유지. 다만 그 캐시를 **어느 도구로 접근하느냐**가 환경별로 다르다는 분기 미명시 → v5.0에서 보강.

**v3.1 베놈 유지:** 모든 신규·수정 스킬에 **9룰 강제 적용**. SKILL.md 골격에 Prereq·NextPhase·OutputPath·RefIndex·Boundaries·WhenToUse·FailureModes 7섹션 강제 + metadata 표준화 + description 첫문장 평문화 강제.

## ⓪ 이중환경 분기 게이트 (v5.0 신설 · 진입 첫 단계)

**원칙**: 본 스킬의 모든 명령은 **두 환경 중 하나**에서 실행됨. 어느 환경에서 무엇을 할지 명시 안 하면 § ① 코드블록 전체가 빈 값을 반환하거나 read-only 권한 오류로 실패.

| 작업 | 환경 | 도구 | 경로 |
|------|------|------|------|
| **SKILL.md 편집** | 호스트(macOS) | Edit·Write·Read (호스트 도구) | `/var/folders/.../T/claude-hostloop-plugins/{HASH}/skills/{X}/SKILL.md` |
| **검증·sha256·grep** | VM(Linux 샌드박스) | bash (mcp__workspace__bash) | `/sessions/{SID}/mnt/.claude/skills/{X}/` (read-only bindfs) |
| **패키징(zip)** | VM | bash | 스테이징=`/tmp/`·결과는 `cat $ZIP > $OUT` redirect |
| **N사본 동기** | 환경 분기 (아래 §② 참조) | — | — |

**진입 게이트 절차**:
1. 형 발화에서 스킬명 추출
2. **호스트 캐시 경로 확인**(호스트 Read로 `/var/folders/.../skills/{X}/SKILL.md` 첫 5줄) — 성공 = 편집 가능
3. **VM mount 경로 확인**(VM bash `ls /sessions/*/mnt/.claude/skills/{X}/SKILL.md`) — 성공 = 검증·패키징 가능
4. 한쪽이라도 실패 시 STOP + 진단 메시지(스킬명 오타 / 플러그인 미설치 / mount 안 됨)

**금지**:
- VM bash로 `/var/folders/...` 접근 시도 ✗ (안 보임)
- 호스트 Edit으로 `/sessions/...` 접근 시도 ✗ (호스트에 없음)
- 호스트 캐시를 VM cp로 staging 시도 ✗ (bindfs 권한 거부)

## Skill Boundaries

- **하는 것** — SKILL.md 편집·references/scripts 갱신·9룰 강제 적용·N사본 동기·검증·(옵션) 패키징
- **안 하는 것** — 스킬 품질 진단(→ skill-doctor) · 자동변이/최적화(→ autoloop) · 플러그인 패키징(→ create-cowork-plugin) · UP 편집(→ up-manager) · git push (→ git-sync)

## When to Use

- 사용자가 "{스킬명} 수정해줘·고쳐줘·바꿔줘" — 단일 스킬 편집
- 신규 스킬 생성 요청 ("X 스킬 만들어줘")
- autoloop 루프 종료 후 handoff.json 감지 — 자동 패키징 모드
- skill-doctor 처방전 수신 — 처방대로 수정 후 N사본 동기
- **안 쓸 때** — 단순 질문, 스킬 진단만 원함(→ skill-doctor), 자동 최적화(→ autoloop)

## Prerequisites

본 스킬 진입 전 다음을 체크. 미충족 시 STOP + 안내.

| # | 체크 | 미충족 시 |
|---|------|-----------|
| 1a | **호스트 캐시 경로** 존재 (호스트 Read로 `/var/folders/.../T/claude-hostloop-plugins/*/skills/{X}/SKILL.md` 첫 5줄) | 호스트 캐시 없음 → 진단: ① 스킬명 오타? ② 플러그인 미설치? ③ 호스트 경로 hash 변경? (해시 폴더 ls로 재확인) |
| 1b | **VM mount 경로** 존재 (VM bash `ls /sessions/*/mnt/.claude/skills/{X}/SKILL.md`) | VM mount 없음 → 진단: ① 세션 재시작 필요? ② 플러그인 캐시 미동기? |
| 2 | 9룰 적용 의도 확인 (신규 스킬) | 9룰 골격 자동 삽입. 수정 모드는 누락 룰만 보강 |
| 3 | autoloop handoff.json 있나? (선택) | 있으면 ① 캐시 백업 단계로 핸드오프 사용 |
| 4 | 형 발화에 패키징 트리거 hit? (`패키징해줘·.skill로·줘·달라·제공해줘`) | hit = §④ 자동 진행 / miss = §① §② §③까지만 |

## ⛔ 절대 규칙 (8)

| # | 규칙 |
|---|------|
| 1 | **게이트키퍼** — 스킬 편집 전 본 스킬 발동. 미발동 = FAIL |
| 2 | **이중환경 분리** — 편집 = 호스트 Edit 도구·검증/패키징 = VM bash. 환경 혼동 시 빈 결과·권한 오류로 실패. §⓪ 게이트 우선 통과 |
| 3 | **호스트 캐시 직접 편집** — `/var/folders/.../T/claude-hostloop-plugins/{HASH}/skills/{X}/SKILL.md`를 **호스트 Edit·Write 도구**로 직접 수정. VM bash로 호스트 경로 접근 ✗ · VM staging·cp -r ✗ |
| 4 | **N사본 동기 강제(환경 분기)** — 호스트 캐시 편집 후 호스트 측 다중 plugin 디렉터리 동기는 별도 절차. VM 측은 read-only mount 1곳 자동 반영. §② 참조 |
| 5 | **9룰 베놈 강제** — 신규 스킬은 7섹션(Boundaries·WhenToUse·Prereq·OutputPath·RefIndex·NextPhase·FailureModes) 골격 자동 삽입. 수정 모드는 누락 룰 보강. 미적용 = FAIL |
| 6 | **편집 직후 sha256 검증 강제(VM bash)** — VM mount 경로에서 shasum -a 256. 호스트 다중 사본 동기 시는 별도 sha 비교 절차 |
| 7 | **패키징 권한 우회 강제** — bindfs로 outputs에 zip 직접 쓰기 ✗ → `/tmp/`에서 zip 생성 후 `cat $ZIP > $OUT` redirect |
| 8 | **패키징은 옵션** — 형이 "패키징해줘·.skill로·zip해줘·줘·달라·제공해줘" 명시 시만. 무명시 = §편집·§동기·§검증만으로 완료 |

## ① 호스트 캐시 직접 편집 (호스트 Edit 도구 전용)

**호스트 캐시 경로 패턴**:
```
/var/folders/{2글자}/{랜덤}/T/claude-hostloop-plugins/{HASH}/skills/{SKILL_NAME}/SKILL.md
```

**경로 확인 (호스트 Read 도구로 — VM bash ✗)**:
- 호스트 Read 도구로 첫 5줄 읽기 시도
- 성공 = 편집 가능. 실패 = Prerequisites 1a로 돌아가 진단

**편집 방식 (전부 호스트 도구)**:
- **Edit** — 1~3곳 부분수정. 가장 단순·안전 (디폴트)
- **Write** — 전면 재작성 시만 (신규 스킬·v숫자 마이그레이션). 부분수정에 Write 사용 = FAIL
- **morph edit_file** — 다중수정·반복치환·구조보정 (UP 모프 모드 hit 시만)

**편집 시 주의**:
- `replace_all: false`가 디폴트. 동일 문자열 다중 hit 시 명시 트리거 필요
- 큰 블록 교체보다 작은 단위 다발이 안전(섹션별 Edit 1콜씩)
- 9룰 7섹션 누락 시 Edit로 섹션 추가·기존 본문 보존

**금지**:
- `mcp__workspace__bash`로 `/var/folders/...` 접근 ✗ (VM에서 호스트 캐시 안 보임. exit 1만 반환)
- VM 안에서 `cp -r` 캐시 → `/tmp/` staging ✗ (read-only bindfs·권한 거부)
- `python3 << EOF` 류 heredoc으로 호스트 캐시 직접 쓰기 ✗ (VM 안에서 호스트 경로 없음)

상세 — `→ references/edit-protocol.md`.

## ② N사본 동기 (환경 분기)

### VM 환경 (현 세션 디폴트)

- VM에 보이는 SKILL.md = `/sessions/{SID}/mnt/.claude/skills/{X}/SKILL.md` **1곳뿐**(read-only bindfs)
- 호스트 캐시 편집 시 이 mount는 **자동 반영**(마스터 = 호스트 캐시)
- **VM 안에서는 추가 동기 작업 불필요**. 검증만 진행(§③)

### 호스트 환경 (다중 plugin 디렉터리 존재 시)

호스트에는 plugin UUID × user UUID 조합마다 별개 사본 존재 가능. 다음 위치들 확인:

```
~/Library/Application Support/Claude/local-agent-mode-sessions/skills-plugin/.../skills/{X}/
~/Library/Application Support/Claude/local-agent-mode-sessions/{SID}/.../cache/.../skills/{X}/
/var/folders/.../T/claude-hostloop-plugins/{HASH}/skills/{X}/   ← 마스터
```

**호스트 동기 절차** (호스트 쉘 직접 접근 필요):
- 본 스킬 범위 밖. `git-sync` 또는 형 직접 작업으로 위임
- VM 안 skill-builder는 마스터(호스트 캐시) 편집까지만 책임

### 단순화 원칙 (v5.0)

- 캐시 1곳(호스트 마스터) 편집 = VM mount 자동 반영 → 현 세션 즉시 발동 OK
- 다른 워크스페이스·재기동 후 옛 버전 발동 우려 시 → `git-sync` 또는 패키징 후 재배포

## ③ 검증 (VM bash 1콜)

VM mount 경로에서 검증. 호스트 캐시 편집이 mount에 반영됐는지·9룰 누락 없는지 확인.

```bash
# VM bash 실행 (mcp__workspace__bash)
SKILL_NAME=skill-name
CACHE_VM=$(ls /sessions/*/mnt/.claude/skills/$SKILL_NAME/SKILL.md 2>/dev/null | head -1)

# 1. 파일 존재·라인수
wc -l "$CACHE_VM"

# 2. sha256 (VM mount = 호스트 마스터 반영분)
shasum -a 256 "$CACHE_VM"

# 3. 9룰 7섹션 grep
for SECTION in "Skill Boundaries" "When to Use" "Prerequisites" "절대 규칙\|Output" "Reference Index" "Next Phase" "Failure Modes\|Gotchas"; do
  grep -q "$SECTION" "$CACHE_VM" && echo "✓ $SECTION" || echo "✗ MISSING: $SECTION"
done

# 4. description ≤1024 (대략 — 정확 측정은 호스트 Read로 frontmatter 직접 확인)
DESC=$(awk '/^description:/,/^[a-z]+:/' "$CACHE_VM" | head -100 | wc -c)
[ $DESC -le 1024 ] && echo "✓ desc≈$DESC/1024" || echo "✗ desc≈$DESC 초과 의심·호스트 Read로 정확 확인"
```

**컨펌 1줄:** `편집 N곳 · sha=XX · 9룰=OK·desc≈N/1024 → §편집·§검증 완료. 패키징 필요?`

## ④ 패키징 (VM bash·옵션·형 명시 시만)

**v5.0 권한 우회 패턴 강제** — bindfs로 outputs에 zip 직접 쓰기 `Operation not permitted` 발생. `/tmp/`에서 zip 만든 후 `cat redirect`.

```bash
# VM bash (mcp__workspace__bash)
SKILL_NAME=skill-name
SRC=/sessions/dazzling-trusting-ritchie/mnt/.claude/skills/$SKILL_NAME    # VM mount(read-only)
STAGE=/tmp/skill-stage-$$
ZIP_TMP=/tmp/$SKILL_NAME.skill
OUT=/sessions/dazzling-trusting-ritchie/mnt/outputs/$SKILL_NAME.skill

# 스테이징(읽기는 가능)
mkdir -p $STAGE
cp -r $SRC $STAGE/$SKILL_NAME

# /tmp에서 zip 생성 (zip 내부 경로 명시 — exclude 패턴 활성화)
rm -f $ZIP_TMP
cd $STAGE
zip -qr $ZIP_TMP $SKILL_NAME \
  -x "$SKILL_NAME/_archive/*" \
     "$SKILL_NAME/__pycache__/*" \
     "$SKILL_NAME/*.pyc" \
     "$SKILL_NAME/.DS_Store" \
     "$SKILL_NAME/evals/*"

# outputs로 cat redirect (cp 대신·bindfs 우회)
cat $ZIP_TMP > $OUT

# 검증
ls -la $OUT
shasum -a 256 $ZIP_TMP $OUT   # 두 sha 동일해야 OK
unzip -l $OUT | head -30      # _archive 제외 확인
```

**핵심 우회 패턴**:
| 함정 | 우회 |
|------|------|
| `cp $ZIP $OUT` → bindfs 권한 거부 | `cat $ZIP > $OUT` redirect |
| `zip -x "_archive/*"` 무력화 | zip 내부 경로 명시 `-x "$SKILL_NAME/_archive/*"` |
| outputs에 0바이트 파일 잔존 | sha256 비교로 사이즈 일치 확인 |

**제공**: `mcp__cowork__present_files` 또는 `[다운로드](computer://...)` 1줄. **git push 제안**: 옵션·답 없으면 ✗.

**언제 패키징 필요한가:**
- 형이 명시 요청 ("패키징해줘·zip해줘·.skill로 줘")
- 다른 머신·다른 사람에게 공유
- 백업·git 레포에 추가 (git-sync에 위임)

**캐시 직접 편집만으로 충분한 경우:**
- 트리거 추가·1~3곳 수정·블록 신설 등 일반 작업
- 현 세션·다음 세션에서 즉시 발동 필요한 경우
- §②(N사본 동기)까지 끝나면 형 하드에 완전 반영

## Output Path (산출물 경로 명시)

| 산출물 | 경로 | 비고 |
|---|---|---|
| 편집된 SKILL.md (영구) | 캐시 경로 + N사본 7곳 | §①·§② 종료 시 자동 반영 |
| .skill 패키지 (옵션) | host outputs `~/Library/.../local_*/outputs/{SKILL}.skill` | §④ 호출 시에만 |
| 백업 (안전) | `/tmp/{SKILL}.SKILL.md.bak.{ts}` | §① 진입 시 자동 1회 |
| 리서치 결과 | `{VAULT}/_skills research/{SKILL}/{YYYY-MM-DD}_{topic}.md` | 세션 종료 시 소실 방지 |
| handoff (autoloop 연계) | `/sessions/{id}/autoloop-lab/{SKILL}/handoff.json` | 패키징 input |

## Reference Index

references/ 폴더의 어떤 파일을 언제 읽는지.

| 파일 | 내용 | 언제 |
|---|---|---|
| `references/edit-protocol.md` | 묶음 편집 heredoc 패턴 | ① 편집 시 |
| `references/new-skill-template.md` | 신규 스킬 골격 (9룰 포함) | 신규 생성 시 |
| `references/trigger-guide.md` | description 트리거 작성법 (P1~NOT) | description 작성·수정 시 |
| `references/9-rules-template.md` | 7섹션 골격 스니펫 | 베놈 모드 (9룰 보강) |
| `references/anti-bangbang.md` | v3.2 뱅뱅이 4대 RCA + 진단 출력 패턴 | python heredoc 작성·성능 패치 시 |

## 신규생성 (9룰 베놈 골격 자동 삽입)

```
skill-name/
├── SKILL.md          (≤500줄·목표 200~300·9룰 7섹션 강제)
├── references/       (도메인 지식 ≥1개)
├── scripts/          (결정적·반복 작업)
└── assets/           (출력 템플릿)
```

**신규 스킬 생성 위치 = 캐시 폴더에 직접 mkdir.** 그 다음 §② N사본 동기로 다른 워크스페이스에도 전파.

**SKILL.md 7섹션 강제 골격:**
1. `## Skill Boundaries` — 하는 것/안 하는 것
2. `## When to Use` — 상황 묘사 2~5줄 (P2 동사 + 시나리오)
3. `## Prerequisites` — 시작 전 체크 표
4. `## ⛔ 절대 규칙` — 5~7개 표
5. `## Output Path` — 산출물 경로 결정론
6. `## Reference Index` — references/ 폴더 표
7. `## Next Phase` + `## Failure Modes (Gotchas)` — 다음 스킬 추천 + 흔히 망치는 패턴

description ≤1024자·3인칭/명령형·**첫 문장은 평문 한 문장** (외부 LLM 호환). 표 우선·"왜" 필수·예시 1+·Gotchas 필수.

상세 — `→ references/new-skill-template.md`, `→ references/trigger-guide.md`, `→ references/9-rules-template.md`.

## 배치 (N개 동시)

```bash
# N개 스킬을 한 번에 편집 + 각각 N사본 동기
SKILLS="skill-A skill-B skill-C"
BASE="$HOME/Library/Application Support/Claude/local-agent-mode-sessions/skills-plugin"

for SK in $SKILLS; do
  CACHE=$(ls /var/folders/*/T/claude-hostloop-plugins/*/skills/$SK/SKILL.md 2>/dev/null | head -1)
  # 편집 (Edit/morph/python heredoc 중 택)
  # ...
  # N사본 동기
  find "$BASE" -path "*/skills/$SK/SKILL.md" -exec cp "$CACHE" {} \;
done

# 일괄 검증
for SK in $SKILLS; do
  COUNT=$(find "$BASE" -path "*/skills/$SK/SKILL.md" -exec shasum -a 256 {} \; | awk '{print $1}' | sort -u | wc -l)
  echo "$SK: sha종류=$COUNT (1이면 OK)"
done
```

## Next Phase

본 스킬 작업 후 자연스럽게 이어지는 흐름:

- **검증·진단** → `skill-doctor` (32셀 매트릭스로 수정 후 건강도 측정)
- **자동 최적화** → `autoloop` (eval 기반 변이 루프로 점수 상승)
- **레포 동기화** → `git-sync` (github-repos에 push·다른 사람 공유)
- **세션 마무리** → `session-briefing` (작업 결정·미결·다음을 VAULT 저장)

## Failure Modes (Gotchas)

| 함정 | 대응 |
|---|---|
| 매 step `head·grep·ls` 재확인 | **v2.0 실패 본질.** 묶음 heredoc 1회로 편집·assert·len()·검증 통합 |
| 캐시 1곳만 고치고 끝 | **v4.0 핵심.** N사본 동기 누락 시 다른 워크스페이스·재기동 시 옛 버전 발동. §② 강제 |
| description 1024 초과 사후 적발 | 편집 heredoc 안 `len()` assert 즉시 측정 |
| 패키징 = 끝 보고 | 실제 발동 시 작동 안 할 수 있음. §③ sha256 동일성 1줄 검증 필수 |
| `.skill` 접미사 `-1`·`_copy` | 마운트 구파일 미삭제. §④에서 `rm -f` 선행 |
| zip 대신 tar | .skill = zip 전용 |
| autoloop handoff 무시 | `autoloop-lab/$SKILL/handoff.json` 감지 시 ① 백업 단계로 핸드오프 사용 |
| 리서치 outputs 저장 | 세션 종료 = 소실. VAULT만 |
| description ↔ 본문 불일치 | description = 발동 판단 유일 입력 |
| **9룰 7섹션 누락한 채 패키징** | 절대규칙 4 위반. 7섹션 grep PASS 후 진행 |
| **신규 스킬에 Boundaries·WhenToUse 누락** | 트리거 충돌 시 디스앰비게이트 불가. 골격 강제 삽입 |
| **Prerequisites 누락한 채 시작** | 의존 파일 없을 때 침묵 실패. 본문 상단에 체크 표 |
| **(v3.2) 묶음 heredoc 실패 시 진단 콜 폭증** | assert에 실패 태그·snippet 강제. heredoc 끝 `applied/skipped` 출력 강제. 형 진단 콜 0회 |
| **(v3.2) 디스크 풀로 파일 truncated** | 더 이상 staging 안 함 → /sessions 디스크 무관. 작성 직후 `wc -c` 검증만 |
| **(v4.0) VM staging 만들려는 시도** | **금지.** 캐시=형하드 동일파일. 복사·staging은 디스크 풀·bindfs 권한 거부 사고만 일으킴. 캐시 직접 편집 |
| **(v4.0) N사본 동기 시 의도적 변형본 덮어쓰기 우려** | cp 전에 6개 사본의 sha256·mtime 출력하여 확인. 모두 stale이면 진행, 변형 의심 시 형 컨펌 |
| **(v4.0) references/scripts 폴더 변경 후 SKILL.md만 동기** | rsync로 폴더째 동기 또는 find로 references/*도 cp |
| **(v5.0) VM bash로 `/var/folders/.../skills/` 접근 시도** | **금지.** VM에서 호스트 캐시 안 보임. exit 1만 반환. 호스트 Read·Edit·Write 도구로 직접 접근 |
| **(v5.0) "캐시 = 호스트하드 동일" 명제만 믿고 VM에서 편집 시도** | 캐시 위치 자체는 동일하나 **VM에서 그 경로 접근 불가**가 별개 사실. v4.0 RCA 누락 사고. §⓪ 이중환경 게이트 통과 강제 |
| **(v5.0) "7사본 sha256 sort -u = 1" VM에서 실행** | VM에 보이는 사본 = mount 1곳뿐. 호스트 N사본 검증은 별도 도구 영역(git-sync·형 직접) |
| **(v5.0) cp $ZIP $OUTPUTS_DIR/ → 권한 거부·0바이트 잔존** | bindfs 권한 분리. `cat $ZIP > $OUT` redirect로 우회. sha256으로 사이즈 일치 검증 |
| **(v5.0) `zip -x "_archive/*"` 미적용** | zip exclude는 zip 내부 경로 매칭. `-x "$SKILL_NAME/_archive/*"` 형태로 명시 |
| **(v5.0) description에 메타·버전이력 박제** | 자가위반. description은 평문 1문장 + P1~P5 트리거. 버전이력은 헤더 본문에만 |
| **(v5.0) Prerequisites "캐시 미존재 → 오타?" 일반론** | 진짜 원인은 환경 분리·hash 변경·plugin 미설치 등. Prereq 1a·1b 분리하여 호스트/VM 양쪽 체크 |

## ❌ WRONG vs ✅ CORRECT

```
❌ WRONG (v3.x): VM `/sessions/$SID/$SKILL/`로 cp -r → 편집 → mnt/outputs/staging → host sync
   사고 — 디스크 풀, bindfs 권한 거부, references/ 빈 폴더, 책임경계 모호 등 5건 발생
✅ CORRECT (v4.0): 캐시 경로 직접 Edit → find로 N사본 cp → sha256 sort -u 검증 1줄
```

```
❌ WRONG: 신규 스킬 생성 시 frontmatter + 본문 1섹션만 작성 (9룰 누락)
✅ CORRECT: 7섹션 골격(Boundaries·WhenToUse·Prereq·OutputPath·RefIndex·NextPhase·FailureModes) 자동 삽입 후 도메인 내용 채움
```

```
❌ WRONG: 캐시 1곳 고치고 "끝" 보고
✅ CORRECT: §② N사본 동기 → §③ sha256 동일성 검증 → "7사본 모두 동일 sha OK" 보고
```

```
❌ WRONG: 형 무명시인데도 자동으로 .skill zip 출고 → outputs 어지럽힘
✅ CORRECT: §① §② §③까지만 자동·§④ 패키징은 형 명시 트리거 시만
```

```
❌ WRONG (v3.2 안티-뱅뱅): 묶음 heredoc 실패 후 "어디까지 들어갔지?" 진단 콜 3회 (sed/grep/find 따로)
✅ CORRECT: assert에 실패 태그 + heredoc 끝 `applied/skipped` 출력 → 1콜로 진단 완료
```

```
❌ WRONG (v5.0): VM bash로 `ls /var/folders/.../skills/$X/SKILL.md` → 빈 결과 → 캐시 없다고 보고
✅ CORRECT: §⓪ 게이트에서 호스트 Read로 캐시 5줄 확인 + VM bash로 mount 확인. 두 경로 분리
```

```
❌ WRONG (v5.0): VM에서 호스트 캐시 cp → /tmp staging → 호스트 outputs로 cp
   사고 — bindfs 권한 거부·0바이트 파일·outputs 점유
✅ CORRECT: 호스트 Edit으로 캐시 직접 편집(staging ✗) + /tmp에서 zip → cat redirect로 outputs 작성
```

```
❌ WRONG (v5.0): description 첫 문장이 "v4.0 캐시-직접 모델 — VM staging·Host sync 폐기..."
   사고 — 자가위반·외부 LLM 호환 ✗·트리거 매칭 약함
✅ CORRECT: 평문 1문장 "스킬을 생성·수정·검증·패키징하는 1턴 완결 게이트키퍼" + P1~P5 표준 포맷
```

## 모프 적용 (v260524_08~)

- 사용 영역  → SKILL.md 9룰 7섹션(Prereq·NextPhase·OutputPath·RefIndex·Boundaries·WhenToUse·FailureModes) 일괄 삽입·references/scripts 다중 갱신·트리거 P1~P5 구조 보정
- 트리거    → "모프로·모프 모드·Morph로" hit 시에만 발동. 트리거 없으면 내장도구(Write·Edit·replace_all)로 진행
- 폴백      → Morph MCP 미연결·실패 시 즉시 내장도구로 전환. 동일 도구 1회 실패 시 재시도 ✗
- 검증      → 매 Morph edit_file 콜 직후 ① 캐시 경로 grep ② N사본 sha256 sort -u ③ wc -l ④ HTML이면 </script>·</body>·</html> 존재 ⑤ JS·TS·Python이면 문법검사. 누적검증 ✗

---
> Source: [jasonnamii/skill-builder](https://github.com/jasonnamii/skill-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
