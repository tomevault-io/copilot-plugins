## hooshex-website

> سلام Gemini! من یک توسعه‌دهنده Laravel هستم و یک پلتفرم آموزش آنلاین به نام **هوشکس** دارم. می‌خواهم معماری پروژه را به **بالاترین سطح استاندارد جهانی** برسانم.

# Prompt کامل برای Gemini - مشاوره معماری پروژه هوشکس

## 📋 دستورالعمل

سلام Gemini! من یک توسعه‌دهنده Laravel هستم و یک پلتفرم آموزش آنلاین به نام **هوشکس** دارم. می‌خواهم معماری پروژه را به **بالاترین سطح استاندارد جهانی** برسانم.

لطفاً فایل‌های زیر را بخوانید و سپس به سوالات من پاسخ دهید:

1. `ARCHITECTURE_ANALYSIS_FOR_GEMINI.md` - تحلیل کامل معماری فعلی
2. `GEMINI_ARCHITECTURE_CONSULTATION.md` - سوالات مشاوره

---

## 🎯 هدف اصلی

من می‌خواهم:
- معماری را به **world-class standard** برسانم
- **Best practices** برای Laravel 12 را پیاده‌سازی کنم
- یک **scalable** و **maintainable** architecture بسازم
- برای **future growth** آماده شوم

---

## 📊 خلاصه پروژه

### Stack:
- **Backend**: Laravel 12, PHP 8.2+
- **Frontend**: Livewire 3, Tailwind CSS v4, Alpine.js
- **Admin**: Filament 3
- **Database**: PostgreSQL, Redis, Meilisearch

### Architecture:
- **Modular Monolith** با **Domain-Driven Design** سبک
- **Domain Structure**: AiTools, Auth, Blog, Commerce, Core, Courses

### مشکلات فعلی:
1. برخی Domain‌ها خالی هستند (Auth, Core)
2. Services پراکنده
3. Controllers هنوز business logic دارند
4. Test coverage پایین (< 10%)
5. عدم وجود Repository Pattern
6. عدم وجود Action classes
7. Models در app/Models هستند (باید به Domains منتقل شوند)

---

## ❓ سوالات کلیدی

### 1️⃣ ساختار Domain و Repository Pattern
- آیا ساختار Domain فعلی مناسب است؟
- آیا باید Repository Pattern اضافه کنیم یا از Eloquent مستقیم استفاده کنیم؟
- بهترین practice برای Laravel 12 با Modular Monolith چیست؟
- چگونه باید Models را بین Domains و app/Models تقسیم کنیم؟

### 2️⃣ Service Layer Architecture
- چگونه باید Service classes را سازمان‌دهی کنیم؟
- آیا باید Domain Services و Application Services جدا داشته باشیم؟
- تفاوت بین Domain Services و Application Services در Laravel چیست؟

### 3️⃣ Action Classes vs Service Classes
- آیا Action classes بهتر از Service classes هستند؟
- چه زمانی از Action استفاده کنیم و چه زمانی از Service؟
- آیا Laravel Action packages توصیه می‌شود؟

### 4️⃣ Event-Driven Architecture
- چگونه Event-driven architecture را در Laravel پیاده‌سازی کنیم؟
- آیا برای پلتفرم آموزش آنلاین لازم است؟
- چه نوع Eventهایی باید داشته باشیم؟

### 5️⃣ API Design و Versioning
- آیا باید Laravel API Resources استفاده کنیم؟
- چگونه API versioning پیاده‌سازی کنیم؟
- آیا GraphQL بهتر از REST است؟

### 6️⃣ Caching Strategy
- بهترین caching strategy برای Laravel چیست?
- Query caching, Model caching, یا Response caching?
- چگونه cache invalidation را مدیریت کنیم?

### 7️⃣ Testing Strategy
- بهترین testing strategy برای Laravel + Livewire چیست؟
- چگونه Livewire components را test کنیم؟
- آیا TDD توصیه می‌شود؟

### 8️⃣ Frontend Architecture
- آیا ساختار CSS فعلی (base/components/utilities/pages/vendors) بهترین است؟
- چگونه JavaScript را code split کنیم؟
- آیا Storybook اضافه کنیم؟

### 9️⃣ Performance Optimization
- چگونه performance را optimize کنیم؟
- آیا باید eager loading استفاده کنیم؟
- چگونه database indexing را بهبود دهیم؟

### 🔟 Security Best Practices
- چه security measures اضافی نیاز داریم؟
- آیا باید rate limiting اضافه کنیم؟
- چگونه CSRF protection را بهبود دهیم؟

### 1️⃣1️⃣ Monitoring & Observability
- آیا Laravel Telescope کافی است؟
- چگونه logging strategy را بهبود دهیم؟
- آیا باید APM اضافه کنیم؟

### 1️⃣2️⃣ Scalability
- آیا Modular Monolith کافی است یا باید به Microservices فکر کنیم؟
- چگونه برای scalability آماده شویم؟
- چه زمانی باید scale کنیم?

---

## 🎯 درخواست

لطفاً:

1. ✅ **هر سوال را به تفصیل پاسخ دهید**
2. ✅ **Best practices** برای Laravel 12 را پیشنهاد دهید
3. ✅ **مثال‌های عملی** با کد PHP/Laravel ارائه دهید
4. ✅ **اولویت‌بندی** برای پیاده‌سازی پیشنهاد دهید
5. ✅ **Trade-offs** هر تصمیم را توضیح دهید
6. ✅ **References** و منابع مفید ارائه دهید

---

## 📝 نکات مهم

- من می‌خواهم **world-class architecture** بسازم
- **Maintainability** و **Scalability** برای من بسیار مهم است
- می‌خواهم برای **future growth** آماده شوم
- **Performance** و **Security** اولویت بالایی دارند

---

**متشکرم از کمک شما! 🙏**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sepehrbayat)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sepehrbayat)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
