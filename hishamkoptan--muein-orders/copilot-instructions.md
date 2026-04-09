## muein-orders

> 1. كل Repository Method يرجع Future<ApiResult<T>> فقط.


# قواعد التعامل مع الأخطاء (Global Error Handling Rules)

1. كل Repository Method يرجع Future<ApiResult<T>> فقط.
2. النجاح = ApiResult.success(data).
3. الفشل = ApiResult.failure(AppErrorHandler.toApiError(e, st)).
4. Bloc/UI لا يستخدم try/catch. يستخدم result.isSuccess أو result.when.
5. جميع الأخطاء تمر عبر AppErrorHandler.
6. ممنوع استخدام مكتبة dartz (Either, Left, Right, Option...).
7. ApiResult هو المصدر الوحيد للتعامل مع نتائج الـ API والـ Firebase.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HishamKoptaN)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/HishamKoptaN)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
