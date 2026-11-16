# [Authentication bypass via OAuth implicit flow](https://portswigger.net/web-security/oauth/lab-oauth-authentication-bypass-via-oauth-implicit-flow)

## **`الهدف`**
الدخول إلى حساب `carlos` مع ملاحظة أن البريد الإلكتروني الخاص به هو `carlos@carlos-montoya.net`.
## **`خطوات الحل`**
نقوم بتسجيل الدخول إلى حساب `wiener:peter` عن طريق اكمال خطوات تسجيل الدخول بـ `OAuth` بعد ذلك نتوجه إلى `Proxy > HTTP history` ثم نقوم بالبحث عن الـ `request` التالي ونقوم بتحويله إلى `repeater`
```http
POST /authenticate HTTP/2
Host: 0a2900f80365361380b4038800b000de.web-security-academy.net
Cookie: session=RKyTTblc0E1e0kgpEwXlLcVhj4h90uWP

{"email":"wiener@hotdog.com","username":"wiener","token":"vRQH906F0N4mX1qDwXdl2MrloTFKL4i9O2rmELTR3X9"}
```

بعد ذلك نقوم بتغير البريد الإلكتروني إلى `carlos@carlos-montoya.net`، ونقوم بالضغط على `Mouse left-click` داخل الـ `request` ثم `Request in browser ثم In original session` ثم نقوم بنسخ الرابط وفتحه في المتصفح لنجد أننا قد دخلنا إلى `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب على السيرفر ألا يثق نهائيًا في مدخلات المستخدم، وألا يعتمد إطلاقاً على حقلي `email` أو `username` المرسلة في الـ `Request` القادم من المتصفح لمعرفة هوية الشخص، ويجب أن يهتم فقط بالـ `authorization code`، وبمجرد أن يستلمه يجب أن يقوم بإنشاء اتصال آمن وخفي (في الخلفية) مع مزود خدمة الـ `OAuth` ليستكمل باقي الخطوات بأمان بعيدًا عن المتصفح.
