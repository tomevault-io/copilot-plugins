## dftr

> بعد أي تعديل على الكود — commit و push تلقائياً إلى origin


# Git: commit و push تلقائياً

- بعد إنهاء تعديلات المستخدم على هذا المشروع (`dftr-repo`)، **لا تنتظر** طلباً صريحاً للرفع.
- نفّذ مباشرةً (ما لم يطلب المستخدم عدم الرفع):
  1. `git status` للتحقق من الملفات
  2. `git add` للملفات المعدّلة ذات الصلة
  3. `git commit -m "رسالة وصفية قصيرة بالعربية أو الإنجليزية"`
  4. `git push` إلى الفرع الحالي (عادة `main`)
- إذا فشل الـ push (شبكة، مصادقة): أخبر المستخدم بالخطأ ولا تفترض أن الرفع تم.
- لا ترفع أسراراً (.env) ولا تلغي `.gitignore`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ahmedsami2hvuv)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Ahmedsami2hvuv)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
