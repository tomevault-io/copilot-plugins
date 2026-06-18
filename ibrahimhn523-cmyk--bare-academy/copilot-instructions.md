## bare-academy

> > ملف مرجعي يصف المشروع بشكل دقيق ليستخدمه Claude في الجلسات القادمة. يُحدَّث عند تغيير الهيكلة أو اتخاذ قرارات معمارية.

# CLAUDE.md — دليل المشروع لـ Claude Code

> ملف مرجعي يصف المشروع بشكل دقيق ليستخدمه Claude في الجلسات القادمة. يُحدَّث عند تغيير الهيكلة أو اتخاذ قرارات معمارية.

---

## ١. بنية المشروع الحالية

مشروع **بوابة أكاديمية بارع** — منصة ويب لإدارة برامج أكاديمية رياضية تعليمية في بريدة، السعودية.

### الملفات وأدوارها

```
bare-academy/
├── index.html            (568 سطر) — الصفحة العامة + نموذج التسجيل
├── dashboard.html        (744 سطر) — لوحة الإدارة
├── dashboard.css         (365 سطر) — تنسيقات لوحة الإدارة
├── dashboard.js          (2454 سطر) — منطق لوحة الإدارة
├── portal.html           (920 سطر) — بوابة الموظفين متعددة الصلاحيات
├── portal.css            (1025 سطر) — تنسيقات البوابة
├── portal.js             (3092 سطر) — منطق البوابة
├── leaderboard.html      (408 سطر) — لوحة صدارة عامة (HTML + JS مدمج)
├── tournament-view.html  (567 سطر) — عرض البطولة العام (HTML + JS مدمج)
├── README.md             (170 سطر) — توثيق عام (⚠️ متقادم: يذكر Google Sheets)
├── image/                — الشعارات (bare-logo.png, bare-logo-blue.png)
├── fonts/                — خط TheYearofHandicrafts (5 أوزان)
└── .claude/
    └── settings.local.json — صلاحيات Claude Code المحلية
```

### التطبيقات الأربعة

| التطبيق | الجمهور | الوصف |
|---|---|---|
| `index.html` | العامة | صفحة هبوط + نموذج تسجيل طالب → يكتب في جدول `students` |
| `dashboard.html` | المدير العام (كلمة سر واحدة) | إدارة الطلاب، البرامج، الاشتراكات، الرسوم، التواصل، الإحصائيات، السجل |
| `portal.html` | موظفون متعددون (users + permissions) | تحضير، نقاط، مسابقات ثقافية، بطولات رياضية، صدارة، إدارة المستخدمين |
| `leaderboard.html` + `tournament-view.html` | العامة (قراءة فقط) | عرض صدارة برنامج معيّن وعرض بطولة (يُمرَّر `?prog=` أو `?id=`) |

### ملاحظة عن الملفات المفقودة المذكورة في README
- `registration.html` — **غير موجود**؛ نموذج التسجيل مدمج في `index.html`.
- `google-sheets-api.js` — **غير موجود**؛ تمت الهجرة إلى Supabase.
- `package.json` — **غير موجود**؛ لا يوجد build step.

---

## ٢. الـ Stack المستخدم

### Frontend
- **Vanilla HTML / CSS / JavaScript** — بدون framework، بدون build step، بدون bundler.
- **CDN خارجي:**
  - SheetJS `xlsx-0.20.1` (في `dashboard.html`) — لاستيراد/تصدير Excel.
  - Font Awesome 6.4.0 (في `index.html` فقط).
- **خط محلي:** `TheYearofHandicrafts` (5 أوزان) في مجلد `fonts/`.

### Backend / Data
- **Supabase** (PostgREST API):
  - URL: `https://oytfhgqhibbcsqbnvwyv.supabase.co/rest/v1`
  - مفتاح: `anon` JWT — مكشوف في الكود client-side في كل الملفات.
- **النداء مباشر** من المتصفح إلى Supabase REST عبر `fetch()` — لا يوجد خادم تطبيق وسيط.
- **localStorage** للـ session (`portal_user`) ولقوالب WhatsApp (`wa_templates`).

### الجداول في Supabase
| الجدول | الاستخدام |
|---|---|
| `students` | الطلاب |
| `programs` | البرامج التعليمية |
| `subscriptions` | اشتراكات الطلاب في البرامج |
| `payments` | الدفعات |
| `settings` | إعدادات عامة |
| `logs` | سجل العمليات (آخر 150) |
| `users` | مستخدمو البوابة (مع `password`, `role`, `permissions`) |
| `attendance`, `attendance_log` | التحضير |
| `points`, `point_reasons` | نظام النقاط |
| `cultural_competitions`, `cultural_participants` | المسابقات الثقافية |
| `sports_tournaments`, `sports_teams`, `sports_matches`, `sports_stats` | البطولات الرياضية |

### الاستضافة
- **غير محددة في الكود** — لا `vercel.json`، لا `netlify.toml`، لا Dockerfile.
- يعمل كـ **static site** على أي خادم HTTP بسيط (Python, Node http-server حسب README).

### Google Sheets
- **مذكور في README فقط — غير مستخدم في الكود.**
- المشروع كان مرتبطاً بـ Google Sheets سابقاً ثم هاجر إلى Supabase. كلمة "sheet" في الكود تشير لـ Excel sheets فقط (مكتبة SheetJS).

### المصادقة
- **لا يوجد Supabase Auth.**
- المصادقة يدوية: جدول `users` مع `password` plaintext، فحص بمقارنة نصية، حفظ في `localStorage`.
- `dashboard.html` يستخدم كلمة مرور واحدة ثابتة (موثّقة سابقاً في README).

---

## ٣. المشاكل الحرجة المعروفة

### 🔴 C1 — كلمات المرور مخزّنة plaintext
- **المكان:** [portal.js:144](portal.js:144)
- **الوصف:** `if (u.password !== password)` — مقارنة نصية، يعني الـ DB تحتوي passwords بدون hashing/salting.
- **الأثر:** أي تسرّب لـ DB = كشف كل كلمات المرور دفعة واحدة. كثير من المستخدمين يكررون كلمات المرور في حسابات أخرى.

### 🔴 C2 — مصادقة client-side عبر localStorage فقط
- **المكان:** [portal.js:146-162](portal.js:146)
- **الوصف:** بعد `login()` يُحفظ كائن المستخدم (مع `role` و `permissions`) في `localStorage`. عند إعادة التحميل يُقرأ منه مباشرة بدون أي تحقق من السيرفر.
- **الأثر:** أي مستخدم يفتح Console ويعدّل `localStorage.portal_user` ليصير `role:'super_admin'`، ثم يحدّث الصفحة → صلاحية كاملة بدون استدعاء سيرفر.

### 🔴 C3 — Anon key مكشوف للعمليات الحساسة
- **المكان:** [dashboard.js:5](dashboard.js:5), [portal.js:6](portal.js:6), [index.html:519](index.html:519), [leaderboard.html:219](leaderboard.html:219), [tournament-view.html:206](tournament-view.html:206)
- **الوصف:** نفس الـ anon JWT يُستخدم لـ INSERT/UPDATE/DELETE على كل الجداول من المتصفح.
- **الأثر:** كل الأمان يعتمد على Supabase **RLS policies** (غير مرئية في الكود). لو RLS غير محكمة → أي زائر يقدر يعمل `DELETE /students` أو `UPDATE /payments` مباشرة.

### 🔴 C4 — لا rate limiting على login
- **المكان:** [portal.js:133-151](portal.js:133)
- **الوصف:** `login()` يقبل عدداً لا نهائياً من المحاولات.
- **الأثر:** brute force سهل، خصوصاً مع كلمات مرور بسيطة.

### 🔴 C5 — صلاحيات client-side غير ملزمة على السيرفر
- **المكان:** [portal.js:165-168](portal.js:165), [portal.js:301](portal.js:301)
- **الوصف:** `hasPermission()` يفحص كائن `_user.permissions` المحلي فقط. كل العمليات الفعلية تذهب لـ Supabase REST بنفس anon key.
- **الأثر:** الواجهة تخفي الأزرار للمستخدم العادي، لكنه يقدر يستدعي API مباشرة. الصلاحيات "زينة بصرية" بدون enforcement حقيقي.

### مشاكل حرجة إضافية (موثّقة في تقرير المراجعة الكامل)
- جدول `users` قابل للقراءة من العميل (يعرّض كل passwords).
- لا CSRF protection على عمليات الكتابة.
- كلمة المرور الإدارية كانت مكشوفة في README سابقاً.

---

## ٤. Tech Debt مرتب بالأولوية

### 🔴 Critical (يمنع الانتقال للإنتاج)
| # | البند | الموقع |
|---|---|---|
| TD-1 | Hashing لكلمات المرور (أو الانتقال لـ Supabase Auth) | [portal.js:144](portal.js:144) |
| TD-2 | استبدال localStorage auth بـ JWT موقّع من السيرفر | [portal.js:146-162](portal.js:146) |
| TD-3 | إنشاء طبقة API (Vercel/Cloudflare Functions) للعمليات الحساسة | جميع `fetch` المباشرة لـ Supabase |
| TD-4 | فحص وقفل RLS policies على كل الجداول | في Supabase (خارج الـ repo) |
| TD-5 | rate limiting على login | [portal.js:133](portal.js:133) |
| TD-6 | فرض صلاحيات على السيرفر بناءً على `auth.uid()` | RLS + portal.js permissions |

### 🟡 Major (يحسّن جودة وصيانة الكود)
| # | البند | الملاحظة |
|---|---|---|
| TD-7 | استخراج الـ Supabase helpers المشتركة بين dashboard.js و portal.js | ازدواجية ~10 دوال بالحرف |
| TD-8 | تقسيم الملفات الضخمة (portal.js: 3092 سطر، dashboard.js: 2454 سطر) | حسب الـ domain |
| TD-9 | استبدال inline `onclick=` (226 موقع) بـ event listeners | يكسر مع CSP صارمة |
| TD-10 | نقل قوالب WhatsApp من localStorage إلى DB | [dashboard.js:1408](dashboard.js:1408) |
| TD-11 | إصلاح 8 catch blocks فارغة (silent error swallowing) | [portal.js:161, 202-203](portal.js:161) وغيرها |
| TD-12 | تحسين `addLog`: استخدام Postgres trigger بدل INSERT+SELECT+DELETE في كل عملية | [dashboard.js:60-74](dashboard.js:60) |
| TD-13 | إضافة Subresource Integrity (SRI) لـ CDN scripts | [dashboard.html:8](dashboard.html:8), [index.html:8](index.html:8) |
| TD-14 | تحويل validation للسيرفر (CHECK constraints + RLS validation) | [index.html:444-481](index.html:444) |
| TD-15 | حقل كلمة المرور في modal إنشاء المستخدم نوعه `text` | [portal.html:709](portal.html:709) |
| TD-16 | revalidate state بعد بعض mutations لتجنب stale UI | متفرق |
| TD-17 | تقليل State العام (~25 متغير `let _xxx`) | بداية portal.js و dashboard.js |

### 🟢 Minor (تحسينات تدريجية)
| # | البند |
|---|---|
| TD-18 | استخراج magic strings (`'مشترك'`, `'super_admin'`...) إلى ملف constants |
| TD-19 | إضافة `'use strict'` في بداية ملفات JS |
| TD-20 | توحيد أسماء المتغيرات (Arabic/English mix) |
| TD-21 | إضافة Service Worker للعمل offline (تطبيق إداري) |
| TD-22 | إعداد ESLint + Prettier |
| TD-23 | (اختياري) الانتقال إلى TypeScript |
| TD-24 | تحديث README (يذكر Google Sheets، registration.html، google-sheets-api.js — كلها متقادمة) |
| TD-25 | توثيق `--gold2` ومتغيرات CSS الأخرى |

---

## ٥. المتطلبات القادمة

> هذا القسم فارغ حالياً. سيُملأ عند تحديد ميزات/تحسينات جديدة بعد مرحلة Discovery.

<!-- مثال متوقع:
- [ ] ميزة كذا — السبب: ... — الأولوية: ...
-->

---

## ٦. قرارات معمارية

> سجل القرارات التقنية المهمة بصيغة خفيفة (ADR-lite). كل قرار: **التاريخ، السياق، القرار، البدائل، النتيجة**.

### ADR-001 — استخدام Supabase REST مباشرة من العميل (سابق)
- **التاريخ:** غير موثّق (موروث من بداية المشروع).
- **السياق:** الحاجة إلى MVP سريع بدون بنية backend.
- **القرار:** استدعاء Supabase REST مباشرة من المتصفح باستخدام anon key.
- **البدائل المتاحة وقت القرار:** Supabase JS Client، Vercel Functions، Express server.
- **النتيجة:** سرعة في التطوير لكن مع ثغرات أمنية حرجة (انظر §٣). **القرار يحتاج مراجعة قبل الإنتاج.**

### ADR-002 — هجرة من Google Sheets إلى Supabase (سابق)
- **التاريخ:** غير موثّق.
- **السياق:** الحاجة لقدرات DB حقيقية (joins, queries, indexing).
- **القرار:** ترك Google Sheets API الكاملة، استخدام Supabase Postgres.
- **النتيجة:** تحسّن كبير في الأداء والقدرات. README متقادم لم يُحدّث ليعكس هذا.

### ADR-004 — جدول `comm_log` لسجل رسائل التواصل
- **التاريخ:** 2026-04-30
- **السياق:** الحاجة لسجل دائم لكل رسالة واتساب مرسلة (للمراجعة، التدقيق، تتبّع التواصل مع أولياء الأمور).
- **القرار:** جدول مستقل `comm_log` (بدل توسيع جدول `logs` العام المحدود بـ 150 سطراً).
- **البدائل:**
  - (أ) إعادة استخدام `logs` مع حقل `label` منظّم — مرفوض: محدود بـ 150 ولا يدعم استعلامات منظمة.
  - (ب) جدول `whatsapp_log` خاص بواتساب — مرفوض: قد نضيف قنوات أخرى (SMS/إيميل) مستقبلاً.
- **النتيجة:** جدول عام للتواصل بحقول snapshot للطالب (studentName, phone) لتفادي فقدان السياق عند تعديل بيانات الطالب لاحقاً. الحقل `sentBy` مهيّأ مسبقاً لربطه بمصادقة حقيقية في S4. **DDL في commit `67a42c6` ضمن رسالة الـ commit.**

```sql
CREATE TABLE comm_log (
  id            BIGSERIAL PRIMARY KEY,
  "programId"   BIGINT,
  "studentId"   BIGINT,
  "studentName" TEXT NOT NULL,
  phone         TEXT,
  "templateName" TEXT,
  "messageText" TEXT,
  "sendType"    TEXT NOT NULL DEFAULT 'single',
  "sentBy"      TEXT NOT NULL DEFAULT 'admin',
  "sentAt"      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_comm_log_program ON comm_log("programId");
CREATE INDEX idx_comm_log_student ON comm_log("studentId");
CREATE INDEX idx_comm_log_sent_at ON comm_log("sentAt" DESC);
```

### ADR-003 — جدول `users` يدوي بدل Supabase Auth (سابق)
- **التاريخ:** غير موثّق.
- **السياق:** الرغبة في نظام صلاحيات granular مخصص.
- **القرار:** جدول `users` فيه `password`, `role`, `permissions` (JSON).
- **النتيجة:** plaintext passwords + auth client-side فقط. **يحتاج مراجعة عاجلة (TD-1, TD-2).**

### ADR-005 — التواريخ هجرية في الواجهة، ميلادية في DB
- **التاريخ:** 2026-05-01
- **السياق:** المستخدمون (المشرفون + إبراهيم) يفكرون ويتعاملون بالتقويم الهجري في السياق التعليمي السعودي. التواريخ المعروضة بالميلادي تُربك. ولكن DB لا يجب أن تتغير لتفادي migration للبيانات القائمة + للحفاظ على توافق مع أي نظام مستقبلي.
- **القرار:**
  - **التخزين:** Supabase يبقى على ميلادي `YYYY-MM-DD` (لا تغيير).
  - **الواجهة:** كل إدخال + كل عرض هجري `YYYY/MM/DD`.
  - **التحويل:** عند الحدود فقط (قراءة من DB → هجري للعرض، حفظ → ميلادي للـ DB).
  - **الإدخال:** تقويم منبثق هجري مخصّص (لا توجد إمكانية لجعل `<input type="date">` هجرياً).
- **البدائل المرفوضة:**
  - (أ) عرض هجري + ميلادي معاً → ازدحام بصري.
  - (ب) input ميلادي + معاينة هجرية حية → لا يطابق التجربة المطلوبة.
  - (ج) تخزين هجري في DB → يُحدث migration معقد لبيانات قائمة + يربط الـ DB بتقويم ثقافي.
- **التحويل التقني:**
  - **G→H:** `Intl.DateTimeFormat` مع `calendar:'islamic-umalqura'` (دقيق، رسمي سعودي).
  - **H→G:** بحث ضيق على Intl نفسه (لا API قياسي عكسي). الدقة 100%، السرعة < 1ms.
- **النتيجة:** البيانات في DB تبقى قابلة للقراءة بأي أداة (ميلادية standard ISO). الواجهة تعرض/تستقبل بما يفهمه المستخدم. أي تكامل خارجي مستقبلي يجد بيانات قياسية. التطبيق على 4 مراحل (دوال → عرض → إدخال → لمسات) بـ commits منفصلة لاختبار آمن.

### ADR-007 — إعادة بناء التحضير بتصميم drill-down جديد (mock)
- **التاريخ:** 2026-05-02
- **السياق:** بعد حذف ميزة التحضير بالكامل (ADR-006)، تم اعتماد تصميم جديد عبر mockup مرفق. الحاجة: إعادة بناء بواجهة بسيطة، 3 شاشات drill-down (Programs → Groups → Attend)، قراءة من البيانات الحقيقية للسياق، وحالات حضور mock في الذاكرة (لا DB بعد).
- **القرار:**
  - **3 شاشات** متتابعة في `#section-attendance`، تُرسم ديناميكياً في `#att-body` بناءً على `_attScreen`.
  - **مصدر البيانات:** البرامج النشطة من `_progs`، المجموعات من `program.groups` (مفصول بـ `،`) أو من الاشتراكات، الطلاب من subscriptions الحقيقية. **حالات الحضور mock** في `_attData` + `localStorage('bare_att_mock')`.
  - **4 حالات** فقط: حاضر / متأخر / مستأذن / غائب (مبسّط من 5 السابقة).
  - **mobile-first:** breakpoints 480/600/900px، مع responsive للجدول الأسبوعي (sticky عمود الاسم، scroll أفقي).
  - **prefix `att-`** لكل CSS classes ودوال JS لتفادي التصادم.
- **البدائل المرفوضة:**
  - (أ) `<select>` لاختيار البرنامج (مثل قسم النقاط) — درج مع UX مختلف، يُربك في mobile.
  - (ب) فُل-mock بدون DB حقيقي — يضيع فرصة اختبار التكامل مع البيانات الفعلية.
  - (ج) ربط بـ Supabase الآن — مؤجل حتى تُحدّد متطلبات الـ schema (ADR قادم).
- **النتيجة:**
  - portal.html: +18 سطر (sidebar + section + permission chip)
  - portal.js: +400 سطر (state + 19 دالة + 8 تعديلات سياق)
  - portal.css: +280 سطر (mobile-first مع متغيرات لون الحالات)
  - **مجموع: ~700 سطر مضاف**
  - DB لم تُلمس. سيتم بناء جداول `attendance` و `attendance_log` لاحقاً (إن لزم).
  - localStorage key: `bare_att_mock` — يضيع عند مسح الكاش لكن يبقى عبر reload.
  - **مراحل قادمة محتملة:** ربط DB، إضافة سجل العمليات، تنقّل بين الأسابيع، تقارير شهرية.

### ADR-008 — Schema جداول التحضير وربطها بـ Supabase
- **التاريخ:** 2026-05-02
- **السياق:** بعد ADR-007 (إعادة بناء التحضير mock في localStorage)، حان وقت ربط الميزة بـ DB لتمكين عبر-أجهزة و persistent storage. السؤال: schema بسيط (جدول واحد) أم طبيعي (جلسات + سجلات)؟
- **القرار:**
  - **خيار أ — جدول واحد عريض:** `attendance` يخزن (programId, studentId, date, status) مع snapshot للاسم والمجموعة و audit metadata.
  - **جدول `attendance_log`** للـ audit (oldStatus, newStatus, changedBy, changedAt). يُكتب best-effort بعد كل تعديل (parallel، لا يعطّل التدفق إن فشل).
  - **`UNIQUE(programId, studentId, date)`** — يدعم upsert عبر `Prefer: resolution=merge-duplicates`.
  - **`CHECK (status IN ...)`** — حماية من قيم خاطئة على مستوى DB.
  - **Trigger `updatedAt`** — يحدّث تلقائياً عند UPDATE.
  - **RLS مؤقت:** anon full access (تطابق ADR-001). يُحكم في S4.
- **البدائل المرفوضة:**
  - (أ) جلسات منفصلة (`att_sessions` + `att_records`) — تعقيد إضافي بدون حاجة حالية. متوسعة لاحقاً لو احتجنا notes per-session.
  - (ب) Schemaless (jsonb column) — صعوبة الفهرسة والاستعلام، RLS أصعب.
  - (ج) إبقاء mock في localStorage — لا cross-device، لا audit، لا scalable.
- **النتيجة:**
  - **الكتابة optimistic:** UI يُحدّث فوراً، DB في الخلفية، rollback تلقائي عند فشل.
  - **القراءة authoritative من DB** بعد Commit 3.
  - **Migration لمرة واحدة:** عند الدخول الأول للتحضير، يُنقل ما في `localStorage('bare_att_mock')` إلى DB إن أمكن (مع فلترة على الطلاب المعروفين فقط)، ثم يُمسح localStorage.
  - 3 commits متسلسلة: DDL في Supabase Studio (يدوي) → ربط القراءة → ربط الكتابة + migration + audit.

### ADR-011 — صفحة البطولة الجديدة (tournament.html) — React inline + DB موجود
- **التاريخ:** 2026-05-07
- **السياق:** بعد ADR-010 (حذف الرياضي القديم)، احتاج المشروع نظام بطولات أحدث وأكثر وظيفة. توفّر بروتوتايب React جاهز (~7700 سطر) يغطي wizard، dashboard، League/Bracket/Calendar/Stats، modals للمباريات وإحصائيات اللاعبين، public share عبر URL hash. الجداول الخمسة في DB كانت موجودة مسبقاً (`tournaments`, `tournament_teams`, `tournament_matches`, `tournament_events`, `tournament_ratings`).
- **القرار:**
  - **صفحة مستقلة:** `tournament.html` في root، مع كل الكود في `tournament/` subfolder. لا تتداخل مع portal.html / dashboard.html.
  - **React 18 + Babel inline من CDN** — يطابق نمط البروتوتايب، لا build step.
  - **Schema موجود مسبقاً** — لا DDL جديد. الكود يكيّف نفسه:
    - `sport` JSONB → نخزن full object {id,name,emoji,team}
    - `tournament_events`: حقول `team` (string 'home'/'away')، `player`, `scorer`, `assist` منفصلة (لا `playerId/teamId`)
    - `tournament_matches`: لا `phase` — استخدام `groupId`, `isBye`, `isThirdPlace` flags
    - `tournament_matches.winnerId` (مش `winnerTeamId`)
    - `tournament_ratings`: UNIQUE(matchId, playerId) → دعم upsert
  - **TweaksPanel محذوف** — كان edit-mode tool للمطورين (يسمح بتغيير اسم/شعار الأكاديمية والثيم في localStorage). إزالته من الإنتاج لمنع التلاعب من المستخدمين.
  - **الألوان:** البروتوتايب يستخدم نفس palette بارع (`#2D3651`, `#D4AF37`, `#F5E6D3`) — لا تعديل ألوان.
  - **Public share:** `tournament.html#/view/<id>` → readOnly mode (موجود في البروتوتايب).
  - **Participants:** يُسحبون من `subscriptions` حسب `programId` المحدد.
- **البدائل المرفوضة:**
  - (أ) ضمن portal.html — يكسر استقلالية الصفحة + يخلط React مع vanilla JS الموجود.
  - (ب) إعادة كتابة بـ vanilla JS — ضياع لـ ~7700 سطر جاهز ومُختبَر.
  - (ج) Build step (Vite/Webpack) — يخالف ADR-001 (المشروع بدون build).
  - (د) إنشاء جداول جديدة — موجودة مسبقاً، إعادة استخدام أفضل.
- **النتيجة:**
  - **4 commits متسلسلة:**
    1. UI port (~7100 سطر منسوخ + إزالة TweaksPanel)
    2. ADR-011 + F19 (هذا)
    3. DB integration (`tournament/supabase.jsx` + adapt mutations)
    4. Portal integration + Hijri dates + polish
  - **التواريخ هجرية** عبر تطبيق ADR-005 على `formatArabicDate` في `data.jsx` (commit 4).
  - **Babel inline** بطيء قليلاً عند التحميل الأول (compile 14 jsx) لكنه خيار صحيح للنطاق الحالي.
  - **Mobile-first:** البروتوتايب responsive من الأصل، لا تعديل مطلوب لأحجام الشاشات.

### ADR-010 — حذف القسم الرياضي القديم بالكامل
- **التاريخ:** 2026-05-07
- **السياق:** ميزة البطولات الرياضية (Sports Tournaments) كانت موجودة في portal كقسم رئيسي مع شاشتين (قائمة البطولات + الإحصائيات) و wizard من 4 خطوات و 4 modals، إضافة لـ tournament-view.html (صفحة عامة مستقلة) و 4 جداول في DB. المساحة الكلية ~1900 سطر. الميزة لم تُستخدم تشغيلياً ولم تُحقق العائد المتوقع. القرار: حذف نهائي مع نية إعادة بناء بتصميم جديد لاحقاً (نمط مماثل لـ ADR-006 للتحضير).
- **القرار:**
  - حذف كامل من portal.html / portal.js / portal.css.
  - حذف ملف `tournament-view.html` بالكامل.
  - تنظيف cascade في dashboard.js (لم يعد يحاول DELETE من جداول رياضية).
  - تنظيف فلاتر sports في leaderboard.html.
  - **DB:** الجداول الأربعة (`sports_tournaments`, `sports_teams`, `sports_matches`, `sports_stats`) تم DROP خارجياً قبل التنظيف البرمجي.
- **البدائل المرفوضة:**
  - (أ) الإبقاء مع display:none — يبقي تعقيد صيانة لكود غير مستخدم.
  - (ب) إبقاء `tournament-view.html` كأرشيف — يصبح dead code يربط schema مفقود.
  - (ج) ترحيل البيانات للأرشيف — لا قيمة (ميزة لم تُستخدم).
- **النتيجة:**
  - portal.html: 711 → 541 سطر (−170)
  - portal.js: 3322 → 2399 سطر (−923)
  - portal.css: 850 → 770 سطر (−80)
  - dashboard.js: −12 سطر (cascade)
  - leaderboard.html: −6 سطر
  - tournament-view.html: محذوف بالكامل (−567 سطر)
  - **مجموع المحذوف: ~1758 سطر**
  - 5 commits متسلسلة (HTML / JS / CSS / outside-portal / docs).
  - النمط مطابق لـ ADR-006: حذف نظيف، DB قابلة للحذف خارجياً، جاهز لإعادة بناء بتصميم جديد لو احتجناها.

### ADR-009 — تبويب التحضير في dashboard (program-level)
- **التاريخ:** 2026-05-03
- **السياق:** بعد ADR-008 (التحضير في portal)، احتاج المدير في dashboard لرؤية وتعديل بيانات التحضير ضمن صفحة البرنامج بنفس مستوى التبويبات الموجودة (المشتركون، الرسوم، التواصل، الإحصائيات).
- **القرار:**
  - **تبويب جديد** "📅 التحضير" بين التواصل والإحصائيات.
  - **3 شاشات داخلية** (pills):
    - **ملخص إجمالي:** بطاقات + تنبيهات الغياب 3+ + جدول كامل + فلاتر + Excel
    - **تفاصيل يوم:** اختيار يوم + جدول حالات الطلاب
    - **تسجيل الحضور:** اختيار يوم + أزرار ح/ت/م/غ + حفظ تلقائي
  - **استخدام نفس الجدول** `attendance` المُنشأ في ADR-008 (لا duplication).
  - **القراءة/الكتابة:** نفس النمط (optimistic + audit log + rollback).
  - **التصدير:** Excel عبر SheetJS (موجود أصلاً في dashboard.html).
  - **الأيام:** كل أيام البرنامج من startDate إلى endDate حسب program.days، مع scroll أفقي على الموبايل (عدد قد يصل 32+).
- **البدائل المرفوضة:**
  - (أ) إعادة استخدام portal فقط — المدير يحتاج Overview تجميعي ليس متاحاً في portal.
  - (ب) Schema منفصل لـ dashboard — يكسر اتساق البيانات.
  - (ج) day picker لأسبوع واحد — المدير يحتاج رؤية الفصل كاملاً.
- **النتيجة:**
  - dashboard.html: +18 سطر (sidebar + section)
  - dashboard.js: +400 سطر (state + 12 دالة + ٣ تعديلات سياق)
  - dashboard.css: +130 سطر (متغيرات + classes)
  - **مجموع: ~550 سطر مضاف**
  - DB لم تُلمس (نفس جدولَي ADR-008).
  - مشاركة الحضور بين portal و dashboard: المشرف يسجّل في portal، المدير يراجع في dashboard.

### ADR-006 — حذف ميزة التحضير بالكامل
- **التاريخ:** 2026-05-02
- **السياق:** ميزة التحضير (Attendance) كانت موجودة في portal كقسم رئيسي مع 5 صفحات فرعية (programs/groups/matrix/stats/log) + شاشة "تحضير سريع" منفصلة. الميزة كبيرة (~1100 سطر عبر HTML/JS/CSS) لكنها **لم تُستخدم في التشغيل اليومي** كما كان متوقعاً، ولم تُحقق العائد التشغيلي اللازم لتبرير صيانتها.
- **القرار:** حذف الميزة بالكامل من الواجهة والكود. الإبقاء على جدولَي `attendance` و `attendance_log` في Supabase مؤقتاً (لا DROP).
- **البدائل المرفوضة:**
  - (أ) تعطيل العرض فقط (display:none) — يبقي تعقيد الكود ويُربك الجلسات القادمة.
  - (ب) DROP لجدولَي DB فوراً — مخاطرة فقدان أي بيانات تاريخية محتملة. الاحتياط: إبقاؤها متاحة للقراءة لو احتجنا أرشيف لاحقاً.
  - (ج) إبقاء الميزة لاستخدام مستقبلي — يخالف مبدأ "كود قليل أنظف". لو احتجناها لاحقاً، نعيد بناءها بناءً على متطلبات حقيقية.
- **النتيجة:**
  - portal.html: 920 → 702 سطر
  - portal.js: 3092 → 2490 سطر
  - portal.css: 1025 → 553 سطر
  - dashboard.js: −1 سطر (cleanup call)
  - dashboard.css: −5 سطر (CSS ميت)
  - **مجموع المحذوف: ~1110 سطر**
  - الجدولان `attendance` و `attendance_log` يبقيان في DB دون استخدام (لا تأثير).
  - لا أزرار/روابط مكسورة (تم التحقق في المتصفح).
  - `attachHijriPicker` (اسم يحتوي `att`) غير متأثر — منتقي تاريخ مستقل عن الميزة.

<!-- إضافة قرارات جديدة هنا بصيغة:
### ADR-XXX — العنوان
- **التاريخ:** YYYY-MM-DD
- **السياق:**
- **القرار:**
- **البدائل:**
- **النتيجة:**
-->

---

## ملاحظات للجلسات القادمة

- **اللغة:** المشروع بالكامل بالعربية (RTL). كل النصوص الموجهة للمستخدم بالعربي.
- **التاريخ:** يدعم Hijri + Gregorian (موجود في commits أخيرة في branch `claude/brave-chatterjee`).
- **branch الحالي:** `claude/brave-chatterjee` (worktree).
- **Main branch:** `main`.
- **عند العمل على ميزة جديدة:** تحقق أولاً من القرارات المعمارية أعلاه، ولا تكرّس قرارات قديمة (مثل توسيع المصادقة client-side) — الأفضل اقتراح الترقية.
- **عند المراجعة:** التقرير الكامل لمشاكل الأمان موثّق في الجلسة (لم يُحفظ كملف منفصل).

---

## ٧. السياق المؤسسي لأكاديمية بارع

> هذا القسم للفهم السياقي فقط — لا يؤثر على الكود مباشرة لكنه ضروري لفهم القرارات.

### من يستخدم البوابة؟
- **إبراهيم الحسين (أبو خليل):** راعي المشروع + مدير المشروع + القيادة التقنية. يشرف على الاستراتيجية والتطوير.
- **عبدالكريم المهيميد:** مدير البرنامج (شريك). يستخدم البوابة يومياً للتحضير، الرسوم، متابعة الطلاب.
- **المشرفون (3-4 أشخاص):** كل مشرف له صلاحيات محددة (ثقافة، رياضة، إعلام، نظام).

### طبيعة التشغيل
- **الأيام:** الأحد - الأربعاء فقط (لا خميس، جمعة، سبت).
- **وقت الذروة:** 4:15م - 8:00م (وقت الجلسة).
- **البرامج السنوية:** فصل أول، رمضاني، فصل ثاني، صيفي.
- **البرنامج الحالي:** الفصل الدراسي الثاني 1447هـ (29 مارس → 20 مايو 2026).
- **الفئة المستهدفة:** ذكور، رابع ابتدائي → ثالث متوسط.

### الأولويات التشغيلية (بالترتيب)
1. **الاستقرار أولاً** — البوابة تُستخدم خلال الجلسة مباشرة. أي عطل = إشكال ميداني.
2. **سهولة الاستخدام** — المشرفون ليسوا تقنيين. الواجهة تكون واضحة وسريعة.
3. **الدقة المالية** — تسجيل الدفعات والمتبقيات يجب أن يكون موثوقاً 100%.
4. **الأمان كافٍ للسياق** — المشروع داخلي (مو منصة عامة). المستخدمون معروفون. الأولوية تحسين الأمان تدريجياً لا إعادة بناء كاملة.

### قواعد التطوير مع إبراهيم
- **لا بناء قبل الإذن** — عرض الخطة أولاً، انتظار الموافقة، ثم التنفيذ.
- **خطوات صغيرة متسلسلة** — تغيير واحد في كل جلسة، مع اختبار قبل الانتقال للتالي.
- **لا كسر للميزات الموجودة** — البوابة شغّالة الآن وتُستخدم. أي تغيير يحافظ على ما يعمل.
- **توثيق القرارات** — أي قرار تقني مهم يُضاف كـ ADR في §٦.

### الهيكل المالي للمشروع (للسياق فقط)
- إيجار سنوي: 20,000 ريال (موزع على 4 برامج).
- الفصل الثاني: أول برنامج بتكاليف رأسمالية شبه صفر (الاستثمارات سُدِّدت في البرنامجين الأولين).
- الهدف طويل المدى: 5 فروع + مكتب إداري.

---

## ٨. خارطة العمل القادمة

> تُحدَّث هذه الخارطة في بداية كل جلسة عمل.

### المرحلة الحالية: Discovery + Fix
**الهدف:** تحديد المتطلبات الجديدة + إصلاح المشاكل الحرجة.

### تم إنجازه
| # | الإجراء | الحالة | التاريخ | Commit |
|---|---|---|---|---|
| F1 | فلتر "نصف مدة" في قائمة المشتركين (كل / قارب الانتهاء / منتهي) + إعادة تسمية UI من "جزئي" إلى "نصف مدة" | ✅ | 2026-04-29 | `9d588ca` |
| F2 | توحيد مصدر قوالب WhatsApp (📱 modal يقرأ من نفس localStorage كقسم التواصل) | ✅ | 2026-04-29 | `9d588ca` |
| F3 | حذف القوالب الافتراضية المضمّنة بالكامل (`COMM_DEFAULT_TEMPLATES` و `buildWhatsAppMsg`) | ✅ | 2026-04-29 | `9d588ca` |
| F5 | إصلاح ترميز الإيموجي في رابط واتساب (دالة `buildWaUrl` موحّدة + `normalize('NFC')` + تحذير URL طويل) | ✅ | 2026-04-30 | `d26dd92` |
| F6 | سجل التواصل: جدول `comm_log` جديد + تبويب 📜 السجل + التقاط اسم القالب لكل إرسال (فردي/جماعي) | ✅ | 2026-04-30 | `67a42c6` |
| F7 | متغيرات القالب كأزرار قابلة للضغط في مودال "قالب جديد" + helper عام `insertVarAt` + إصلاح ضمني لمتغيرَي التاريخ الهجري المفقودَين | ✅ | 2026-04-30 | `f13e58d` |
| F8 | إصلاح حقيقي لمشكلة الإيموجي: WhatsApp يقطع `text` عند ~1024 byte → � عند الإيموجي. الحل: clipboard fallback للرسائل الطويلة عبر `openWaLink()` | ✅ | 2026-04-30 | `ab839c2` |
| F9 | **التواريخ هجرية — المرحلة ١ (دوال فقط):** `toHijri`, `toGregorian`, `fmtHijri`, `parseHijriInput`, `dateInputToHijri` في dashboard.js + portal.js. اختبار 100/100 round-trip. ADR-005 موثّق | ✅ | 2026-05-01 | `b8ce79c` |

### مشروع التواريخ الهجرية (ADR-005) — مراحل قادمة

| # | المرحلة | الحالة |
|---|---|---|
| F10 | **المرحلة ٢ — العرض هجري:** دالتان جديدتان (`fmtHijriShort`, `fmtHijriDatetime`) + استبدال 19 موضع عرض في dashboard.js و portal.js | ✅ | 2026-05-01 | `c4f9084` |
| F11 | **المرحلة ٣ — الإدخال هجري:** ٣ sub-phases — (3a) منتقي popup + helpers `read/writeDateInput`، (3b) ١٠ inputs في dashboard + ٣٢ موضع `.value`، (3c) ٣ inputs في portal + ٨ مواضع `.value` | ✅ | 2026-05-01 | `f840d08, b33c7d4, 1761459` |
| F12 | **المرحلة ٤ — قوالب واتساب هجرية + تنظيف:** `{بداية/نهاية_الاشتراك}` تنتج هجري فقط، حذف `fmtDate`, `fmtDateBoth`, `fmtDateHijri`, `fmtDatetime` (دوال ميلادية غير مستخدمة)، حذف نسخة مكررة من `fmtDateShort`. توافق عكسي مع `_هجري` للقوالب القديمة | ✅ | 2026-05-01 | `3efa35c` |
| F13 | **حذف ميزة التحضير بالكامل** (ADR-006): ٤ commits متسلسلة — portal.html (-218 سطر) + portal.js (-602 سطر) + portal.css (-472 سطر) + dashboard cleanup (-6 سطر) | ✅ | 2026-05-02 | `34a2225, 88a3267, d504a98, a2edb4b` |
| F14 | **إعادة بناء التحضير** (ADR-007): تصميم drill-down جديد، 3 شاشات (Programs → Groups → Attend)، 4 حالات، mock في localStorage، mobile-first | ✅ | 2026-05-02 | `718c595` |
| F15 | **تحسينات التحضير:** عداد المشتركين على شاشة البرامج + تنقّل بين الأسابيع مع تقييد بمدى البرنامج + إصلاح TZ في تواريخ الأسبوع (UTC) + أيقونات عربية (ح ت م غ) + drag-drop لإعادة ترتيب الطلاب مع localStorage | ✅ | 2026-05-02 | `fb09fdd, f9f9a26, 084d85e, bdc692a, 3d7f7eb` |
| F16 | **ربط التحضير بـ Supabase** (ADR-008): DDL لـ `attendance` + `attendance_log`، قراءة من DB، كتابة optimistic مع rollback، audit log، migration لمرة واحدة من localStorage | ✅ | 2026-05-02 | `b4e3ffe, 72eb2c2` |
| F17 | **تبويب التحضير في dashboard** (ADR-009): 3 شاشات داخلية (ملخص/يوم/تسجيل) في صفحة البرنامج، تنبيهات غياب 3+، Excel export، نفس الـ schema مع portal | ✅ | 2026-05-03 | `2679b97` |
| F18 | **حذف القسم الرياضي القديم بالكامل** (ADR-010): 5 commits متسلسلة — portal.html (-170) + portal.js (-923) + portal.css (-80) + dashboard/leaderboard/tournament-view (-585) + ADR-010 | ✅ | 2026-05-07 | `d62de36, cc6bfef, 7cf1043, 8906737, 90dd7d8` |
| F19 | **صفحة البطولة الجديدة** (ADR-011): `tournament.html` مستقل + 16 ملف React inline + DB layer `tournament/supabase.jsx` + Hijri dates + رابط في portal sidebar | ✅ | 2026-05-07 | `9d3d954, 819ae5a, dbb4fc9, f034d6f` |

### الإصلاحات العاجلة المتبقية (أمنية — بالترتيب)
| # | الإجراء | الحالة |
|---|---|---|
| S1 | فحص RLS policies في Supabase | ⏳ |
| S2 | إصلاح حقل كلمة المرور في modal إنشاء المستخدم (type="text" → type="password") | ⏳ |
| S3 | إضافة rate limiting بسيط على login | ⏳ |
| S4 | الانتقال لـ Supabase Auth (يحل TD-1, TD-2, TD-3, TD-5, TD-6 دفعة واحدة) | 🔜 (يحتاج تخطيط) |

### المتطلبات الجديدة
> فارغة — ستُحدَّد في جلسة Discovery القادمة مع إبراهيم.

### القاعدة لكل جلسة
1. اقرأ §٨ أولاً (الخارطة).
2. أخبر إبراهيم بالحالة الحالية في 3 أسطر.
3. لا تبدأ أي تغيير قبل موافقته.

---
> Source: [ibrahimhn523-cmyk/bare-academy](https://github.com/ibrahimhn523-cmyk/bare-academy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
