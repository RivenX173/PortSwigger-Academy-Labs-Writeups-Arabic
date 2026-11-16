# [JWT authentication bypass via flawed signature verification](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-flawed-signature-verification)

## **`الهدف`**
تعديل الـ `Session Token` للوصول إلى لوحة التحكم في `/admin` ثم حذف حساب `carlos`.
## **`خطوات الحل`**
نقوم بالدخول إلى حساب `wiener:peter` بعد ذلك نذهب إلى مسار `/admin` ولكن سنقوم باعتراض هذا الـ `request` ونقله إلى `repeater`
```http
GET /admin HTTP/2
Host: 0a060039033426e080edb77f0060002f.web-security-academy.net
Cookie: session=eyJraWQiOiI5MjU0MjM5MS0zNTI3LTQwNGMtYTlhZi1lZDRjMDUxNTgxZDkiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc4NTQ4ODU2MSwic3ViIjoid2llbmVyIn0.Fle2p8OsUcKCYLkwGt4PwUB7k4B6pPD2FcnJYmlTlDMwm-76zM0XmR5xQUtuqfvXW3TycTx0bBxCUYKLR5BodgJXnfPKVdIkJreB7OUgQ9CjYqB-ztJnrfF5U4klgDO1yLdVkIxzfguaPXgAzbviF7tZpcrvsTflRy6NE1SINY0zxeD102Cy3e-G2BS_a1mY4K58AdGr9zToTlloVJcizo6xok-D-33pxQl4VaJJGvPGGvMk5ZSuJG453wmCdmHMd8TJqZ3eFpldOXRxqU5LcKCIDwyMXcCtE7bNFg7Mqy12x2GvrdCJfFHV63c6cewdKyDCascIZLQzfHujKBUMTQ
```

وبعد ذلك سنستعمل `Inspector` ونقوم بتظليل الجزء الخاص بالـ `Header` في الـ `Session token` ونقوم بتعديل نوع التشفير في `alg` إلى `none`، ثم تظليل الجزء الخاص بالـ`Payload` ونقوم بتعديل اسم `wiener` إلى `administrator`، ثم نقوم بتظليل الجزء الخاص بالـ `Signature` وحذفه نهائيًا مع مراعاة وجود نقطة بالنهاية، ونقوم بإرسال الـ `request` لنتأكد من صحة الـ `response.

ثم سنقوم بنسخ كامل الـ `Session token` بعد التعديل، ونقوم بالذهاب إلى المتصفح داخل اللاب والضغط على `F12` ووضعها بدلًا من القديمة وعمل `refresh` للصفحة، ونستطيع بعد ذلك الدخول مرة أخرى إلى `/admin` وحذف حساب `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب على السيرفر أن يرفض أي `JWT` يحتوي على `alg: none` فوراً.

ويجب إعداد السيرفر ليقبل فقط خوارزميات محددة مسبقاً (مثلاً `RS256` أو `HS256`). إذا أرسل المستخدم توكن يحتوي على أي خوارزمية أخرى، يجب أن يتم رفضه فوراً.
