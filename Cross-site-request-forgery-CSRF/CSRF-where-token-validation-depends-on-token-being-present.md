# [CSRF where token validation depends on token being present](https://portswigger.net/web-security/csrf/bypassing-token-validation/lab-token-validation-depends-on-token-being-present)

## **`الهدف`**
إنشاء كود `HTML` يقوم بعمل هجوم `CSRF` بحيث يغير الـ `email` الخاص بالضحية.
## **`خطوات الحل`**
**`نفس خطوات حل لاب`** [CSRF vulnerability with no defenses](./CSRF-vulnerability-with-no-defenses.md)

لأن هذا اللاب لا يتحقق من وجود الـ `CSRF Token` فعند حذف المعلم تمامًا (وليس القيمة فقط) فإن الـ `request` يمر بنجاح.
## **`ترقيع الثغرة`**
**`بالإضافة إلى جزء كبير من ترقيع لاب`** [CSRF vulnerability with no defenses](./CSRF-vulnerability-with-no-defenses.md)

فيجب تعديل الكود البرمجي في الـ `Backend` ليفرض الوجود الإلزامي للتوكن، أي أنه يجب إضافة شرط برمجي يتحقق أولاً من وجود معلم `CSRF Token`، وإذا وصل الـ `Request` بدونه نهائياً، فيجب أن يرفض السيرفر الطلب فوراً.
