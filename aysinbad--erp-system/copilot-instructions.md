## erp-system

> Document Name:  AI Agent Instructions

# AI Agent Instructions

## Document Information
```
Document Name:  AI Agent Instructions
Version:        1.1.0
Status:         Approved
Classification: Source of Truth
Owner:          Solution Architecture Team
Approved-by:    Product Owner
Approved-date:  2026-07-18
Last-Updated:   2026-07-18
```

> يُطبَّق على أي أداة AI (Claude, Cursor, Codex, أو غيرها) تعمل على هذا المشروع، بلا استثناء.
> هذا الملف يحوي **القواعد الصلبة فقط** — تتغيّر نادراً. للسياق المتغيّر أسبوعياً (آخر ADR معتمد، الوحدة الجاري العمل عليها)، انظر `docs/09_AI/AI_CONTEXT.md`.

---

## Before doing any task — read in this exact order
1. `DEVELOPMENT_RULES.md`
2. `docs/00_Project/Current_State.md`
3. `docs/02_Governance/Decisions_Log.md` (فهرس) — ثم افتح ملف الـ ADR الفعلي المطلوب من `docs/02_Governance/ADR/ADR-XXX.md`
4. `docs/01_Standards/Glossary.md` — مصطلحات إلزامية، لا مرادفات مرفوضة
5. `docs/02_Governance/Known_Issues.md` — أخطاء معروفة لا تُنقل كسلوك مطلوب
6. `docs/09_AI/AI_CONTEXT.md` — السياق الحالي (آخر ADR، الوحدة قيد العمل)
7. The relevant file in `docs/03_Business_Logic/` for the module in scope (لو مش موجود، انتقل لقاعدة "التوقف والسؤال" أدناه)
8. عند الحاجة للرجوع للكود الفعلي (Rule Zero فقط): `reference/prototype/prototype_v2.html` — اقرأ `reference/prototype/README.md` أولاً

---

## Hard rules

1. **Never invent business logic.**
   لو المنطق غير موثّق أو غامض، المرجع النهائي هو كود الـ Prototype نفسه (`Rule Zero` — `docs/02_Governance/ADR/ADR-000.md`)، وليس افتراض "منطقي" من الـ AI. أي استنتاج من الكود يُذكر صراحةً كافتراض قبل الاستمرار، لا يُترك ضمنياً.

2. **Never change an approved workflow or ADR.**
   أي تغيير في سلوك معتمد يتطلب ADR جديد يُقترح كملف مستقل جديد في `docs/02_Governance/ADR/ADR-XXX.md` (الرقم التالي مباشرة بعد آخر ملف موجود)، لا تعديلاً مباشراً في ملف قائم. حالته الابتدائية دائماً `Proposed`.

3. **Port logic 1:1 — this is a rewrite, not a translation.**
   الهدف نقل نفس السلوك لتقنية جديدة (ASP.NET Core / React)، وليس "تحسين" أو إعادة تفسير منطق الأعمال أثناء النقل، حتى لو بدا الكود القديم غير أنيق.

4. **Use only approved Glossary terms** في الكود والتعليقات والتوثيق — لا مرادفات (مثال: "فاتورة مبدئية (Proforma)" لا "فاتورة أولية").

5. **Known Issues are not behavior to preserve.**
   أي بند في `docs/02_Governance/Known_Issues.md` (مثل: كلمات مرور نص صريح، تحقق صلاحيات على الواجهة فقط، تشفير XOR) يُعالَج بالحل الموثّق هناك عند البناء في الـ Backend الجديد — لا يُنسخ كما هو بحجة "الحفاظ على السلوك".

6. **Never resolve a code-vs-documentation conflict alone.**
   لو الكود يناقض التوثيق، التصرف الوحيد المسموح: توثيق التعارض في `docs/02_Governance/Known_Issues.md` أو ملف Business Logic ذي الصلة (قسم "Open Questions")، والتوقف لمراجعة بشرية — لا الحسم من طرف الـ AI.

7. **If documentation is missing, ask before coding.**
   الغموض سبب للتوقف والسؤال، مش سبب لملء الفراغ بافتراض.

8. **CompanyId (multi-tenant) from day one.**
   أي جدول أو Service جديد يُبنى بافتراض `CompanyId` من البداية — لا يُؤجَّل، حسب المبادئ الحاكمة في `docs/00_Project/Current_State.md`.

9. **The Prototype is never edited to "fix" behavior.**
   `reference/prototype/prototype_v2.html` مرجع تاريخي لمنطق الأعمال (ADR-000)، وليس الكود النهائي. أي تصحيح مطلوب (أمني، منطقي، أو غيره) يُنفَّذ في الـ Backend/Frontend الجديد فقط، بعد اعتماد ADR إن كان يمثّل تغيير سلوك (انظر قاعدة 2).

10. **Each Business Logic module lives in its own folder** under `docs/03_Business_Logic/{Module}/` (per `docs/03_Business_Logic/CRM/` as the reference pattern):
    - **`{Module}.md`** — يبدأ بـ **Module Dependencies** (Depends On — Business Modules / Shared Utilities، وProduces — مين بيستهلك مخرجات هذا الموديول) مباشرة بعد ترويسة المستند، **قبل** Purpose — عشان أي حد يفتح الملف يعرف فوراً إيه الموديولات اللازم يفهمها الأول. ثم: Purpose, Scope, Out of Scope, Actors, Glossary (لو الموديول محتاج مصطلحات خاصة), Entities, Status Flow, Business Rules, Business Invariants, Reports, Open Questions, Future Enhancements.
    - **`{Module}_Implementation_Guide.md`** — تفاصيل الدمج التقني: State Transition Matrix, Permissions, Notifications, Audit Rules, Domain Events, Standard Error Codes.
    - **`{Module}_RFC.md`** — نقاشات مفتوحة عميقة (خيارات/مقارنة/تريد أوف) لسؤال من Open Questions **قبل** ما ياخد شكل قرار جاهز. لو استقر، يترقّى لـ ADR رسمي في `02_Governance/ADR/` ويُعلَّم كمكتمل هنا (لا يُحذَف — للتتبع التاريخي).
    - **`{Module}_Test_Cases.md`** — حالات الاختبار. سيناريوهات حسابية (Given/When/Then) يمكن كتابتها فور استقرار القواعد؛ اختبارات آلية (Unit/Integration) تنتظر وجود كود فعلي.
    - أي تغيير في قاعدة عمل يُحدَّث في `{Module}.md` أولاً، ثم ينعكس أثره في باقي الملفات إن استلزم — لا العكس.

---
> Source: [aysinbad/ERP-System](https://github.com/aysinbad/ERP-System) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
