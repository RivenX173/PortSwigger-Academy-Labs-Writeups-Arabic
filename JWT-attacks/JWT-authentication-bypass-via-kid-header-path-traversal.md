# [JWT authentication bypass via kid header path traversal](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-kid-header-path-traversal)

## **`الهدف`**
قم بتزوير `JWT` يعطيك صلاحيات للوصول إلى لوحة التحكم في `/admin` ثم حذف حساب `carlos`.
## **`خطوات الحل`**
نقوم بالدخول إلى حساب `wiener:peter` بعد ذلك نذهب إلى مسار `/admin` ولكن سنقوم باعتراض هذا الـ `request` ونقله إلى `repeater`
```http
GET /admin HTTP/2
Host: 0afa006803ef60a882b29709005c008f.web-security-academy.net
Cookie: session=eyJraWQiOiIyNDVjMDIxYi02YTAxLTRhYjEtOTMxNy0wMDNjMjRkYTM0NjciLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc4NTY1NzgxNywic3ViIjoid2llbmVyIn0.cKIQ8yWS7z-2RhFN5R2HmCQx_-dqEeHY-qbBBBZyPb4
```
وبعد ذلك سنذهب إلى إضافة `JWT Editior` ونقوم بإضافة `New Symmetric Key` وسنترك حقل `Specify secret` فارغ بدون شئ ثم نضغط `Generate`. ونقوم بتعديل مكونات المفتاح لتصبح كالتالي:
```JSON
{  
    "kty": "oct",  
    "kid": "\/..\/..\/..\/dev\/null",  
    "k": ""  
}
```

ثم نضغط `OK` لنحفظه، ونعود إلى الـ `request` الموجود في `repeater`، ونستعمل `Inspector` ونقوم بتظليل الجزء الخاص بالـ `Payload` في الـ `Session token` ونقوم بتعديل اسم `wiener` إلى `administrator`، ثم نضغط على إختيار `JSON Web Token` ونقوم بالضغط على `Sign` مع التأكيد على إختيار 
`Update/generate "alg", "typ" and "kid" parameters`
ونقوم بإرسال الـ `request` لنتأكد من صحة الـ `response.

ثم سنقوم بنسخ كامل الـ `Session token` بعد التعديل، ونقوم بالذهاب إلى المتصفح داخل اللاب والضغط على `F12` ووضعها بدلًا من القديمة وعمل `refresh` للصفحة، ونستطيع بعد ذلك الدخول مرة أخرى إلى `/admin` وحذف حساب `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة
يُفضل ألا يتم استخدام الـ `kid` كمسار لملف أو استعلام مباشر لقاعدة البيانات، وبدلاً من ذلك، يجب أن يكون الـ `kid` مجرد مُعرف `String أو Number`، ويقوم التطبيق بالبحث عنه داخل قائمة بيضاء معرفة مسبقاً في الكود للوصول إلى المفتاح الصحيح.

وإذا كان لا بد من قراءة المفاتيح من ملفات، يجب فحص قيمة الـ `kid` قبل استخدامها بحيث يكون هناك مسار ثابت وغير قابل للتغيير لإحضار المفتاح، ومنع أي رموز تُستخدم في التنقل بين مسارات الملفات مثل `../` أو `\` أو `/`، ويمكن تطبيق هذا عن طريق السماح فقط بالحروف والأبجدية والأرقام في حقل الـ `kid`.

ويجب أن يقوم السيرفر بفحص المفتاح السري بعد جلبه وقبل استخدامه في عملية التحقق بحيث إذا وجد أن المفتاح فارغ، أو طوله قصير جداً ولا يطابق معايير التشفير الآمنة، يجب أن يرفض العملية ويعتبر الـ `Token` غير صالح.
