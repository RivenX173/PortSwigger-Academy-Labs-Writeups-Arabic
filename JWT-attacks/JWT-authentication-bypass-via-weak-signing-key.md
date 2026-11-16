# [JWT authentication bypass via weak signing key](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-weak-signing-key)

## **`الهدف`**
قم بعمل `brute-force` للحصول على الـ `Secret Key` الخاص بالـ `JWT` ثم استعمله لتوقيع `JWT` مُعدل يُعطيك صلاحيات للوصول إلى لوحة التحكم في `/admin` ثم حذف حساب `carlos`.
## **`خطوات الحل`**
نقوم بالدخول إلى حساب `wiener:peter` بعد ذلك نذهب إلى مسار `/admin` ولكن سنقوم باعتراض هذا الـ `request` ونقله إلى `repeater`
```http
GET /admin HTTP/2
Host: 0ac8003c04cc6ab6803fe45d009a00dd.web-security-academy.net
Cookie: session=eyJraWQiOiJmNzk5NTFlYS1hYmM1LTQ0NzMtYWJlMi1hNWNlZDEzZTNkYmIiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc4NTU3MDI1Nywic3ViIjoiYWRtaW5pc3RyYXRvciJ9.JgYshuh97czVzgFE5x3KU9xl7ed_9QUln2aMypozixA
```

وبعد ذلك سنقوم بنسخ الـ `JWT` إلى ملف نصي بأي اسم عشوائي `jwt` ثم  نقوم بتحميل قائمة كلمات **[wordlist of well-known secrets](https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list)** ونضعها في ملف نصي بأي اسم عشوائي `list` ثم سنستعمل أداة `hashcat` ونقوم بمحاولة كسر المفتاح السري بالأمر التالي:
```Shell
hashcat -a 0 -m 16500 -D 1 E:\jwt E:\list
```
والنتيجة ستكون أن المفاتح السري هو `secret1`، وبعد ذلك سنذهب إلى `Burp Suite` ونذهب إلى إضافة `JWT Editior` ونقوم بإضافة `New Symmetric Key` ونضع `Specify secret: secret1`.

ثم نعود إلى الـ `request` الموجود في `repeater`، ونستعمل `Inspector` ونقوم بتظليل الجزء الخاص بالـ `Payload` في الـ `Session token` ونقوم بتعديل اسم `wiener` إلى `administrator`، ثم نضغط على إختيار `JSON Web Token` ونقوم بالضغط على `Sign`، ونقوم بإرسال الـ `request` لنتأكد من صحة الـ `response.

ثم سنقوم بنسخ كامل الـ `Session token` بعد التعديل، ونقوم بالذهاب إلى المتصفح داخل اللاب والضغط على `F12` ووضعها بدلًا من القديمة وعمل `refresh` للصفحة، ونستطيع بعد ذلك الدخول مرة أخرى إلى `/admin` وحذف حساب `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب على مفتاح التشفير ألا يقل طوله عن `256` بت (`32` بايت)، وألا يكون المفتاح كلمة ذات معني أو موجودة في القواميس، بل يجب أن يتم توليده عشوائيًا باستخدام مولد أرقام عشوائية آمن تشفيريًا `CSPRNG`.

بدلاً من استخدام خوارزمية `HS256` التي تعتمد على مفتاح واحد سري للتوقيع والتحقق، يُفضل استخدام خوارزميات مثل `RS256` أو `ES256` حيث تعتمد هذه الخوارزميات على زوج من المفاتيح: مفتاح خاص `Private Key` يتم الاحتفاظ به بسرية تامة على السيرفر ويُستخدم لتوقيع التوكن، ومفتاح عام `Public Key `يُستخدم فقط للتحقق من صحة التوقيع.

لا يجب الاعتماد على مفتاح سري واحد للأبد، بل يجب إعداد النظام لتغيير (تدوير) المفاتيح السرية بشكل دوري.
