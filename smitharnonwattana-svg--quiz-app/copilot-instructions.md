## quiz-app

> Single-page web app (SPA) for exam practice, targeting students preparing for ม.1 entrance exams.

# My Exam Academia — Claude Development Rules
## Project Overview
Single-page web app (SPA) for exam practice, targeting students preparing for ม.1 entrance exams.
All logic lives in one self-contained HTML file — no build step, no frontend server.
Backend: Firebase Cloud Functions (Node.js 22, asia-southeast1) สำหรับส่ง LINE notify เท่านั้น

## Version Update Rule (IMPORTANT)
**Every time any HTML file is modified or generated**, you MUST:
1. **Increment the version number** — bump the minor version (e.g. v45.2 → v45.3, v45.3 → v45.4)
2. **Update all THREE version locations** in the HTML:
   - `<head>` comment: `<!-- APP_VERSION: v{new_version} -->`
   - Login page display (near bottom of `#page-login`): `>v{new_version}</div>`
   - Version check script (near bottom of `</body>`): `const CURRENT_VERSION = 'v{new_version}';`
3. **Update version.json** at repo root:
   ```json
   { "version": "v{new_version}", "updatedAt": "<current ISO timestamp>" }
   ```
   ⚠️ version.json และ APP_VERSION ใน index.html
   ต้องตรงกันเสมอ — ห้าม bump แค่ไฟล์เดียว
4. **Save a local versioned copy** named `index_v{new_version}.html` (e.g. `index_v45.3.html`)
   - Keep `index.html` as the main working file (update it in place)
   - Save a local copy named `index_v{new_version}.html` — ไฟล์นี้ถูก .gitignore ไว้ ไม่ push ขึ้น GitHub

### Version locations in index.html (ต้องอัพเดททั้ง 3 ที่ทุกครั้ง)
- Line ~30: `<!-- APP_VERSION: v48.2 -->`
- Line ~1072: `>v48.2</div>`  (login page display)
- Line ~13350: `const CURRENT_VERSION = 'v48.2';`  (version check script)

### Current version: v48.2
When making the next change, bump to v48.3.
⚠️ โน้ตนี้อาจตกยุค — ให้ grep `APP_VERSION` ใน index.html เพื่อยืนยันเลขปัจจุบันก่อน bump ทุกครั้ง

## Regression Tests (สำคัญ — รันก่อน push ทุกครั้งที่แก้ index.html)
```bash
bash tests/run.sh
```
- ชุด Playwright checks รวมจากบั๊กที่เคยแก้ (v47.93→v48.0) ต้องเขียวทั้งชุดก่อน push
- แก้บั๊กใหม่เมื่อไหร่ ให้เพิ่ม check ใหม่เข้า tests/regression.mjs ด้วยเสมอ (กันบั๊กเดิมกลับมา)
- ใช้เวลา ~1 นาที รายละเอียด/ข้อจำกัดอยู่ใน tests/README.md
- ไฟล์ tests/ อยู่นอกตัวแอป — กฎ single-file ใช้กับตัวแอปเท่านั้น

## GitHub Push (Deploy)
Token และข้อมูล deploy เก็บอยู่ใน `.claude-local` (gitignored — ไม่ push ขึ้น GitHub)
วิธี push ทุกครั้ง:
```bash
# อ่าน token จาก .claude-local แล้ว push
PAT=$(grep GITHUB_PAT /home/user/Quiz-app/.claude-local | cut -d= -f2)
git remote set-url origin https://${PAT}@github.com/smitharnonwattana-svg/Quiz-app.git
git push origin main
git push -u origin main
git remote set-url origin https://github.com/smitharnonwattana-svg/Quiz-app.git
```

## Deployment
- Production URL: https://smitharnonwattana-svg.github.io/Quiz-app/
- Platform: GitHub Pages (deploy จาก branch main เท่านั้น)
- Firebase project nanont-exam: ใช้สำหรับ Cloud Functions เท่านั้น (ไม่ใช้ Firebase Hosting)
- _headers: deprecated ไม่ได้ใช้แล้ว

## PWA Rules
- App ติดตั้งได้บน iPad/iPhone ผ่าน Safari → Add to Home Screen
- manifest.json และ icon.svg อยู่ที่ repo root
- version.json ใช้สำหรับ auto-detect version update
- ทุก deploy ต้อง update version.json พร้อมกับ index.html เสมอ
- Version check script ใน index.html fetch version.json
  ทุก 5 นาที และตอน app กลับมา foreground

## App Structure
- Navigation: `navigate('page-name')` function
- All pages: `div.page` elements, shown via `.page.active` CSS class
- Data storage: localStorage (client-side, no database)
- LINE notify: Firebase Cloud Functions → LINE Messaging API (Secrets: LINE_TOKEN, LINE_USER_ID)
- PDF viewer: custom canvas-based renderer using pdf.js
- Language: Thai (th)

## Important files / paths
- index.html — frontend ทั้งหมด (อย่าสร้างไฟล์ JS/CSS แยก)
- functions/index.js — Cloud Function lineNotify
  ⚠️ ทุกครั้งที่แก้ functions/index.js ต้อง deploy ผ่าน Cloud Shell:
  1. เปิด https://console.cloud.google.com → กด >_ Cloud Shell
  2. git stash && git pull origin main
  3. npx firebase-tools deploy --only functions --project nanont-exam
  4. รอ Deploy complete! แล้วทดสอบใน DevTools
  Claude Code deploy functions ตรงไม่ได้ — หลังแก้ functions/index.js ให้แจ้ง user ทุกครั้งว่าต้อง deploy เองผ่าน Cloud Shell
- functions/package.json — dependencies ของ Cloud Functions
- firebase.json — config deploy Cloud Functions (ไม่ใช่ Firebase Hosting)
- .firebaserc — Firebase project: nanont-exam
- _headers — deprecated ไม่ต้องแก้
- .claude-local — เก็บ GITHUB_PAT (gitignored ห้าม commit)
- index_v*.html — version backups (gitignored)
- version.json — PWA version manifest (ต้อง push ทุกครั้งพร้อม index.html)

## Session Setup (ทำทุกครั้งที่เริ่ม session ใหม่)
ก่อนทำงานทุกครั้ง ให้รันตามลำดับนี้:

1. git pull origin main
   (sync โค้ดล่าสุดจาก GitHub)

2. ตรวจว่า .claude-local มีอยู่ไหม
   ถ้าไม่มี → สร้างจาก GITHUB_PAT ใน ~/.claude/settings.json:
   echo "GITHUB_PAT=$(node -e "console.log(JSON.parse(require('fs').readFileSync(process.env.HOME+'/.claude/settings.json','utf8')).env.GITHUB_PAT)")" > /home/user/Quiz-app/.claude-local

3. ทดสอบ push ได้เลยถ้าต้องการ deploy
   (ใช้ deploy script ใน ## GitHub Push section)

หมายเหตุ: ถ้า ~/.claude/settings.json ไม่มี GITHUB_PAT
→ แจ้งผู้ใช้ว่า "ต้องสร้าง .claude-local ก่อน กรุณาให้ GITHUB_PAT"

---

## AI Coding Guidelines (Karpathy-inspired)

### 1. Think Before Coding
- State assumptions explicitly before implementing
- If multiple interpretations exist, present them — don't pick silently
- If a simpler approach exists, say so and push back when warranted
- If something is unclear, STOP — name what's confusing and ask

### 2. Simplicity First
- Minimum code that solves the problem. Nothing speculative.
- No features beyond what was asked
- No abstractions for single-use code
- No "flexibility" or "configurability" that wasn't requested
- If you write 200 lines and it could be 50, rewrite it

### 3. Surgical Changes
- Touch only what you must. Clean up only your own mess.
- Don't "improve" adjacent code, comments, or formatting
- Don't refactor things that aren't broken
- Match existing style, even if you'd do it differently
- If you notice unrelated dead code, MENTION it — don't delete it
- Remove imports/variables/functions only if YOUR change made them orphans

### 4. Verify Before Claiming Done
- Re-read your output before saying it's complete
- Check that every requirement from the spec is addressed
- If you skipped something, say so explicitly — don't silently omit

---

## Skills ที่ควรเรียกใช้

ระหว่าง session ให้เรียก skill เหล่านี้เมื่อ context ตรง (proactive ไม่ต้องรอ user ขอ)

### โปรเจกต์นี้ — ใช้บ่อย
- **single-file-spa** — ก่อนเพิ่มฟีเจอร์ใหม่ที่มีอยากแยกไฟล์ JS/CSS (Quiz-app เป็น single-file ตาม design ห้ามแตก)
- **legacy-data-fallback** — เมื่อเพิ่ม field ใหม่ใน schema (`localStorage`, attempt, question, exam) — ใช้ `||` fallback ที่ read time เสมอ (เคย apply: `examType||'mc'`, `q.points||1`, `att.weighted===true`)
- **learning-analytics-design** — เมื่อออกแบบ/ปรับเมตริก (calibration, mood, anxiety markers, difficulty, weakness recovery) ก่อนเขียนโค้ด
- **classlist-toggle-pattern** — เมื่อต้องเปลี่ยน 1 CSS class บน element ที่มี class อื่นด้วย (ใช้ `classList.toggle/add/remove` ไม่ใช่ `className=`)
- **reactive-state-persistence** — เมื่อแก้ฟังก์ชันที่ Firestore listenDoc อาจ re-trigger (มี `_statsState`, `_dashState` แล้ว — อย่าให้ filter reset ตอน listener ยิงซ้ำ)
- **pwa-version-polling** — เมื่อแก้ระบบเวอร์ชัน/auto-reload (Quiz-app ใช้ pattern นี้แล้วผ่าน `version.json`)

### On-demand — เรียกตามจังหวะ
- **scrutinize** — ก่อน wrap up session ที่ทำหลาย versions ติด (≥5 commits) → ตรวจ "ทำ simpler ได้ไหม" + verify end-to-end (ใช้ก่อน user ไปทดสอบ)
- **debug-mantra** — เจอบัคที่หา root cause ไม่เจอใน 2-3 hypothesis → บังคับ reproduce reliably ก่อนเดา (ตัวอย่าง: ถ้าใช้ตอน pinch zoom รอบ 1 อาจไม่ iterate 5 รอบ)

### Skip — ไม่เหมาะกับ context นี้
- post-mortem (commit message ละเอียดอยู่แล้ว, solo dev ไม่ต้อง ceremony)
- management-talk (ไม่มี VP/exec audience)

---
> Source: [smitharnonwattana-svg/Quiz-app](https://github.com/smitharnonwattana-svg/Quiz-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
