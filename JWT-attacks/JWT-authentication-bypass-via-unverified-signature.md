# [JWT authentication bypass via unverified signature](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-unverified-signature)

## **`الهدف`**
تعديل الـ `Session Token` للوصول إلى لوحة التحكم في `/admin` ثم حذف حساب `carlos`.
## **`خطوات الحل`**
نقوم بالدخول إلى حساب `wiener:peter` بعد ذلك نذهب إلى مسار `/admin` ولكن سنقوم باعتراض هذا الـ `request` ونقله إلى `repeater`
```http
GET /admin HTTP/2
Host: 0a4d00ae0300dedc804b5d3d00e200e2.web-security-academy.net
Cookie: session=eyJraWQiOiI1MGU1MzlmZC0zMDZmLTRhM2QtODU2Yy04Mzk1Mzc4Y2QyZWYiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc4NTQ4NjU2MSwic3ViIjoid2llbmVyIn0.zce2LUA_pUE__rJ5ZJ2BS2bfntt7oksbuK1_mF8ztaKO3bzhIb9K9bNIeLJW1QT-T6_S302QthUrUIEYiAldaopdRyoz6r5cGvWZWmE4ul5vEaaVE0tkVFWqFIiyw9cBRerF28IgH1X6VXOuf_7VNgg9oJqaqQAk3rH2YKcBmiIF_aIzBWnLr5_Bp05c3d-pbuOoKst2d6_ODeIxl4Zg6nrnVeqEqLlIwfA0q5mmI3c7vgtG_40LIlccr7adAavldpRf-NdM5fewf4nf03gOYfuxRm67l8Amr9rvQ9it-k9yPpGBjYm9gXbl0CksCXGR5iCAy6HRuxVD6qx2LEXGJw
```

وبعد ذلك سنستعمل `Inspector` ونقوم بتظليل الجزء الخاص بالـ `Payload` في الـ `Session token` ونقوم بتعديل اسم `wiener` إلى `administrator`، ونقوم بإرسال الـ `request` لنتأكد من صحة الـ `response.

ثم سنقوم بنسخ كامل الـ `Session token` بعد التعديل، ونقوم بالذهاب إلى المتصفح داخل اللاب والضغط على `F12` ووضعها بدلًا من القديمة وعمل `refresh` للصفحة، ونستطيع بعد ذلك الدخول مرة أخرى إلى `/admin` وحذف حساب `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب على السيرفر تنفيذ عملية فحص للتوقيع باستخدام المفتاح السري قبل قراءة أي معلومة داخل الـ `Payload` (مثل اسم المستخدم أو صلاحياته)، وإذا فشل الفحص أو لم يطابق الناتج، يُرفض الطلب فوراً برمز `401 Unauthorized`.
