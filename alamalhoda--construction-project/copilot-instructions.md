## construction-project

> قانون فعال‌سازی محیط مجازی برای پروژه construction_project


# قانون فعال‌سازی محیط مجازی

در این پروژه، قبل از اجرای هر دستور Python/Django:

## 1. همیشه ابتدا محیط مجازی را فعال کنم:
```bash
source env/bin/activate
```

## 2. سپس دستور اصلی را اجرا کنم:
```bash
python3 manage.py [دستور]
```

## 3. یا از اسکریپت‌های آماده استفاده کنم:
```bash
./run_django.sh [دستور]
python3 run_django.py [دستور]
```

## مثال‌های عملی:

**❌ اشتباه:**
```bash
python3 manage.py migrate  # ممکن است خطا دهد
```

**✅ درست:**
```bash
source env/bin/activate && python3 manage.py migrate
# یا
./run_django.sh migrate
```

## نکات مهم:

- این قانون فقط برای پروژه construction_project اعمال می‌شود
- محیط مجازی در پوشه `env` قرار دارد
- همیشه از پوشه اصلی پروژه اجرا کنم
- اگر محیط مجازی فعال نباشد، ابتدا آن را فعال کنم

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alamalhoda) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
