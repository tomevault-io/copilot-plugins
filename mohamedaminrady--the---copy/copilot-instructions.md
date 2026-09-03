## the-copy

> هذه الوثيقة تُعد المرجع الرسمي الذي يُوجّه وكيل الترميز المسئول عن تطوير وصيانة مشروع **theeeecopy**. يجب الالتزام بجميع البنود هنا بدقة لضمان جودة الكود، أمان المنظومة، وسلاسة العمل الجماعي.

# AGENTS.md - التوجيهات التنفيذية لوكيل الترميز

<div dir="rtl">

## 🎯 الهدف

هذه الوثيقة تُعد المرجع الرسمي الذي يُوجّه وكيل الترميز المسئول عن تطوير وصيانة مشروع **theeeecopy**. يجب الالتزام بجميع البنود هنا بدقة لضمان جودة الكود، أمان المنظومة، وسلاسة العمل الجماعي.

## 🗂️ نطاق التوجيهات

- تنطبق هذه التعليمات على كامل المستودع ما لم تُقدَّم ملفات `AGENTS.md` أكثر تخصيصًا داخل مجلدات فرعية.
- أي تضارب بين هذه التعليمات وأخرى أكثر تعمقًا يُحسم لصالح الملف الأقرب للمكان الذي تعمل فيه.
- تبقى أوامر النظام أو العميل أو المطوّر أعلى أولوية من هذه الوثيقة.

---

## فهرس المحتوى

1. [هيكل المشروع](#هيكل-المشروع)
2. [المعايير العامة للكتابة](#المعايير-العامة-للكتابة)
3. [إرشادات TypeScript](#إرشادات-typescript)
4. [سير عمل Git](#سير-عمل-git)
5. [متطلبات الاختبار](#متطلبات-الاختبار)
6. [قواعد الأمان](#قواعد-الأمان)
7. [أدلة الأداء](#أدلة-الأداء)
8. [معايير التوثيق](#معايير-التوثيق)
9. [معالجة الأخطاء](#معالجة-الأخطاء)
10. [قائمة مراجعة الكود](#قائمة-مراجعة-الكود)
11. [إرشادات Sentry وتتبّع الأداء](#إرشادات-sentry-وتتبّع-الأداء)
12. [مُلحق: أوامر سريعة وأنماط شائعة](#مُلحق-أوامر-سريعة-وأنماط-شائعة)

---

## هيكل المشروع

### 1. نظرة عامة على المونوريبو

```
theeeecopy/
├── frontend/          # تطبيق Next.js 15
│   ├── src/
│   │   ├── app/      # صفحات App Router
│   │   ├── components/
│   │   ├── lib/      # الأدوات والمساعدات
│   │   ├── hooks/    # Hooks مخصّصة
│   │   └── types/    # تعريفات TypeScript
│   └── public/
├── backend/          # واجهات Express.js
│   ├── src/
│   │   ├── controllers/
│   │   ├── services/
│   │   ├── middleware/
│   │   ├── routes/
│   │   ├── models/
│   │   └── types/
│   └── tests/
├── docs/             # التوثيق
└── scripts/          # سكربتات البناء والأدوات
```

### 2. مكدس التقنيات

- **الواجهات (Frontend)**: Next.js 15.4.7، React 18.3.1، TypeScript 5.7.2، Tailwind CSS 4.1.16، Radix UI، TanStack Query 5.90.6، Zod 3.25.76.
- **الخلفية (Backend)**: Node.js 20+، Express.js 4.18.2، TypeScript 5.0+، Drizzle ORM 0.44.7، PostgreSQL (Neon Serverless)، Redis 5.9.0، BullMQ 5.63.0، Google Gemini AI.

---

## المعايير العامة للكتابة

### 1. قواعد حرجة

**ممنوع قطعًا**
- استخدام النوع `any` بدون مبرر واضح.
- تعطيل أخطاء TypeScript عبر `@ts-ignore` أو `@ts-nocheck`.
- الالتزام أو الدمج بكود مُعلَّق (commented out code).
- الدفع مباشرةً إلى فرع `main` أو `master`.
- دمج طلبات السحب بدون اجتياز CI/CD.
- تخطي كتابة الاختبارات للميزات الجديدة.
- تخزين بيانات حساسة داخل الكود.
- استخدام `var` بدلًا من `let` أو `const`.
- تعديل معاملات الدوال مباشرة.
- إنشاء اعتماديات دائرية.

**يجب دائمًا**
- كتابة كود مفهوم بذاته، والالتزام بـ DRY.
- استخدام أسماء متغيرات ودوال معبّرة.
- إضافة JSDoc للواجهات العامة.
- معالجة الأخطاء صراحةً والتحقق من مدخلات المستخدم.
- تغطية المسارات الحرجة بالاختبارات.
- تشغيل اللينتر قبل الالتزام وتحديث التوثيق عند الحاجة.

### 2. التسمية وتنظيم الملفات

- اتبع camelCase للمتغيرات والدوال، PascalCase للمكوّنات والأنواع، وSCREAMING_SNAKE_CASE للثوابت.
- التزم بنمط الملفات مثل `UserProfile.tsx` للمكوّنات و`useAuth.ts` للهوكس و`api-client.ts` للمرافق.
- لا تستخدم بادئة `I` للواجهات.

### 3. التنسيق والأدوات

- استخدم Prettier بالإعدادات التالية:

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "arrowParens": "always"
}
```

- التزم بقواعد ESLint المضمّنة، بما في ذلك منع `no-unused-vars`، وفرض `prefer-const`، وتحذير `@typescript-eslint/explicit-function-return-type`.
- نظم الاستيرادات حسب الفئات (React/Next، مكتبات خارجية، مسارات مطلقة داخلية، أنواع، أنماط).

- استخدم مسارات عليا (`@/`) كما هو مبيَّن في إعدادات `tsconfig.json`.

---

## إرشادات TypeScript

1. فعّل الوضع الصارم في `tsconfig.json` مع كل خيارات السلامة (`strict`, `noImplicitAny`, ...).
2. استخدم الواجهات لوصف الكائنات والأنواع للاتحادات أو البدائل البسيطة.
3. صرّح بأنواع العائد لكل الدوال العامة، وتجنّب الاعتماد على الاستنتاج الضمني عند تعقّد التواقيع.
4. عالج القيم `null` و`undefined` صراحةً باستخدام الاختبارات أو المشغل `??`.
5. أنشئ Type Guards مخصّصة عند الحاجة وحدد الاتحادات المميزة (Discriminated Unions) لمسارات التنفيذ المختلفة.
6. فضّل الكائنات المجمّدة بـ `as const` أو اتحادات السلاسل بدلًا من `enum` إلا عند الحاجة للتوافق.

---

## سير عمل Git

1. استخدم تسمية فروع واضحة (`feature/...`، `fix/...`، `hotfix/...`، إلخ).
2. التزم بمعيار **Conventional Commits** في رسائل الالتزام.
3. التزم بقالب العنوان والوصف لطلبات السحب (`[Type] الوصف المختصر`).
4. قبل طلب المراجعة يجب تنفيذ: `pnpm typecheck`، `pnpm lint`، `pnpm test`، `pnpm build` وإجراء مراجعة ذاتية وتحديث التوثيق.
5. حدّد وقت استجابة المراجعات: 4 ساعات للإصلاحات الحرجة، 24 ساعة للطلبات الاعتيادية، 48 ساعة للميزات الكبيرة.

---

## متطلبات الاختبار

1. حافظ على تغطية لا تقل عن 80٪ عامةً، مع أهداف أعلى للمسارات الحرجة.
2. استخدم Vitest وTesting Library في الواجهة، وPlaywright للاختبارات الشاملة، وSupertest لاختبارات REST.
3. التزم بنمط AAA (تهيئة، تنفيذ، تحقق) في الاختبارات.
4. قم بمحاكاة (Mock) التبعيات الخارجية، ولا تجرِ اتصالات حقيقية ضمن الاختبارات.
5. استخدم أسماء وصفية لحالات الاختبار.
6. وفّر سيناريوهات Playwright كاملة لتدفقات العمل الأساسية.

---

## قواعد الأمان

1. نفّذ التحقق من الهوية والصلاحيات في جميع نقاط الدخول الحساسة.
2. تحقّق من المدخلات باستخدام Zod أو مخططات مماثلة.
3. استخدم استعلامات Parameterized عبر Drizzle ORM لتجنّب حقن SQL.
4. امنع XSS عبر تعقيم المحتوى (DOMPurify) والاعتماد على الهروب التلقائي في React.
5. تحقّق من متغيرات البيئة عبر مخطط Zod ووفّر ملف `.env.example`.
6. لا تسرّب الأسرار في المستودع، واستعمل حدود المعدل (Rate Limiting) للواجهات.

---

## أدلة الأداء

1. استخدم الاستعلامات المجمّعة، والفهارس، وتجنّب نمط N+1.
2. فعّل التخزين المؤقت في Redis للعمليات المكلفة، وقم بإبطال (Invalidate) المفاتيح ذات الصلة بعد التحديث.
3. في React، استخدم `useMemo` و`useCallback` و`React.memo` لمنع إعادة التصيير غير الضرورية.
4. طبّق التجزئة الديناميكية (Dynamic Imports) للمكوّنات الثقيلة.
5. استخدم `next/image` لتحسين الصور، واستورد ما تحتاجه فقط من المكتبات لتقليل حجم الحزمة.

---

## معايير التوثيق

1. وثّق الواجهات العامة باستخدام JSDoc وأمثلة واضحة.
2. أضف تعليقات مبرِّرة للمنطق المعقّد، وتجنّب التعليقات التوضيحية البديهية.
3. حافظ على ملفات README للموديولات الهامة مع أقسام للاستخدام، وواجهة API، والمعمارية، والاختبارات.
4. استخدم تعليقات OpenAPI/Swagger لتوثيق REST API.

---

## معالجة الأخطاء

1. اعتمد فئات أخطاء مخصّصة (`AppError`, `ValidationError`, ...)، وأبلغ عنها باستخدام Sentry عند الحاجة.
2. تجنّب إسكات (Swallow) الاستثناءات؛ أعد الرمي بعد التسجيل أو أعد صياغة الخطأ برسالة واضحة.
3. فعّل Middleware موحّد لمعالجة الأخطاء في Express، واستخدم Error Boundaries في React.

---

## قائمة مراجعة الكود

### قبل إرسال PR
- [ ] جميع فحوصات النوع واللينتر والبناء والاختبارات ناجحة.
- [ ] لا توجد `console.log` أو أسرار أو كود معلّق.
- [ ] تم تحديث التوثيق وملفات البيئة والمُرحلات (migrations) عند الحاجة.
- [ ] اكتملت المراجعة الذاتية.

### للمراجعين
- التحقق من جودة الكود والتزامه بالمعايير، واكتمال الاختبارات، وسلامة الأمن، وكفاءة الأداء، ووضوح التوثيق.

---

## إرشادات Sentry وتتبع الأداء

1. استخدم `Sentry.captureException(error)` ضمن كتل try/catch.
2. أنشئ Spans بمعنى واضح عبر `Sentry.startSpan` للإجراءات المهمة (نقرات، استدعاءات API...).
3. خصّص أسماء العمليات (`op`) ووثّق السمات (Attributes) ذات الصلة داخل Span.
4. في Next.js، تقع تهيئة Sentry في ملفات `instrumentation-client.(js|ts)`، و`sentry.server.config.ts`، و`sentry.edge.config.ts` فقط.
5. عند تمكين التسجيل، استخدم `Sentry.consoleLoggingIntegration`، واستفد من `logger.fmt` لإنتاج سجلات مُهيكلة.
6. اتبع الأمثلة المرفقة لكيفية التقاط الاستثناءات، وإنشاء Spans، وتسجيل الأحداث.

---

## مُلحق: أوامر سريعة وأنماط شائعة

### أوامر PNPM الأساسية

```bash
# Frontend
cd frontend
pnpm dev
pnpm build
pnpm typecheck
pnpm lint
pnpm test
pnpm test:e2e

# Backend
cd backend
pnpm dev
pnpm build
pnpm typecheck
pnpm lint
pnpm test
pnpm db:push
pnpm db:studio

# Root
pnpm lint
pnpm typecheck
pnpm test
pnpm ci
```

### أنماط مكررة

```typescript
// استدعاء API مع معالجة للأخطاء
async function fetchData<T>(url: string): Promise<T> {
  try {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    return await response.json();
  } catch (error) {
    console.error('Fetch error:', error);
    throw error;
  }
}

// هوك React مع TypeScript
function useProject(projectId: string) {
  const { data, isLoading, error } = useQuery({
    queryKey: ['project', projectId],
    queryFn: () => apiClient.getProject(projectId),
    staleTime: 60000,
  });

  return { project: data, isLoading, error };
}

// التحقق باستخدام Zod
const projectSchema = z.object({
  title: z.string().min(1).max(200),
  description: z.string().max(2000).optional(),
  status: z.enum(['draft', 'published', 'archived']),
});

type Project = z.infer<typeof projectSchema>;
```

### مصادر مرجعية

- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Next.js Documentation](https://nextjs.org/docs)
- [React Documentation](https://react.dev)
- [Drizzle ORM Documentation](https://orm.drizzle.team)
- وثائق المشروع: `README.md`، `backend/BACKEND_DOCUMENTATION.md`، `docs/performance-optimization/README.md`

---

## ✅ الخلاصة

باتباع هذه التوجيهات، يضمن وكيل الترميز:
- جودة عالية قابلة للصيانة.
- أمانًا متينًا وأداءً محسنًا.
- اختبارات موثوقة وتوثيقًا واضحًا.
- تعاونًا فعّالًا مع بقية الفريق.

للاستفسارات أو الاقتراحات، افتح تذكرة على GitHub أو راجع الوثائق المساندة.

</div>

---
> Source: [mohamedaminrady/the...copy](https://github.com/mohamedaminrady/the...copy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
