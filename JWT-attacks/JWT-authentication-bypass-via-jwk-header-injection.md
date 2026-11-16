# [JWT authentication bypass via jwk header injection](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jwk-header-injection)

## **`الهدف`**
قم بتعديل وتوقيع `JWT` للحصول على صلاحيات للوصول إلى لوحة التحكم في `/admin` ثم حذف حساب `carlos`.
## **`خطوات الحل`**
نقوم بالدخول إلى حساب `wiener:peter` بعد ذلك نذهب إلى مسار `/admin` ولكن سنقوم باعتراض هذا الـ `request` ونقله إلى `repeater`
```http
GET /admin HTTP/1.1
Host: 0a9e008803fc3388803cd58800d90072.web-security-academy.net
Cookie: session=eyJraWQiOiJkODEzZjc5ZS03MDYxLTQ1OTQtOTExOC1kMmE2YTJiYTVmNmIiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc4NTU3Njg0Nywic3ViIjoid2llbmVyIn0.vi_mK57B0cetfHzWwCx9hzlCzV8tv9DizS16GC5RBUY_bLIbKzDLbm2D3uz3IuzZKTi2Mnl-SCyCyRG29Gwz7U3gTALy2kvVBD8eJgjHmXx2LIU8dIvwTi6LNI-CsM-OSBCBMoSyOhePgJ_dIRm16GtOMXDQ3QXGq_ZDFd4cunY4RMYD2kZTnoJ5U3epP-CuOiP0dNQJ1Muxn8mab-bJSYwWWHdSMyP0YyHp4beYtYAizt2QYwNW0wzTBYFZd-q7wLGo-jFR41x_OKAm1k_wPW2GpedmKuiyYnxVKZobLP1fnR-ZqPbtIT8fY9GPnS3FA3gmpmldPtgxjx9LukSsug

```

وبعد ذلك سنذهب إلى `Burp Suite` ونذهب إلى إضافة `JWT Editior` ونقوم بإضافة `New RSA Key` ونضغط `Generate`

ثم نعود إلى الـ `request` الموجود في `repeater`، ونستعمل `Inspector` ونقوم بتظليل الجزء الخاص بالـ `Payload` في الـ `Session token` ونقوم بتعديل اسم `wiener` إلى `administrator`، ثم نضغط على إختيار `JSON Web Token` ونقوم بالضغط على `Attack` ثم `Embedded JWK` ونقوم بإرسال الـ `request` لنتأكد من صحة الـ `response.

ثم سنقوم بنسخ كامل الـ `Session token` بعد التعديل، ونقوم بالذهاب إلى المتصفح داخل اللاب والضغط على `F12` ووضعها بدلًا من القديمة وعمل `refresh` للصفحة، ونستطيع بعد ذلك الدخول مرة أخرى إلى `/admin` وحذف حساب `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب ألا يعتمد السيرفر أبداً على المفاتيح أو الروابط المرسلة في الـ `JWT Header` (مثل `jwk` أو `jku` أو `x5c`) للتحقق من صحة التوقيع.

يجب أن يقوم السيرفر بالتحقق من التوقيع باستخدام مفاتيح مخزنة محلياً أو مجلوبة من مصدر داخلي موثوق تماماً.

إذا كانت معمارية التطبيق تتطلب فعلياً الاعتماد على مفاتيح ديناميكية يتم تمريرها، فيجب على السيرفر التحقق من هذا المفتاح ومطابقته بقائمة بيضاء `Whitelist` صارمة تحتوي على المفاتيح المسموح بها فقط قبل استخدامه في عملية الـ `Verification`.

يجب على المبرمجين قراءة توثيق `Document` المكتبة المستخدمة وإيقاف تفعيل دعم `jwk` إذا لم تكن هناك حاجة ماسة له.
