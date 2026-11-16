# [Finding a hidden GraphQL endpoint](https://portswigger.net/web-security/graphql/lab-graphql-find-the-endpoint)

## **`الهدف`**
العثور على `endpoint` مخفية لحذف حساب `carlos`.
## **`خطوات الحل`**
نقوم بعمل تجربة عشوائية أو عن طريق `brute force` للوصول للايند بوينت الخاصة باللاب والتى ستكون `/api`، وبعد ذلك نضغط `Mouse left-click` داخل الـ `request` ونختار `Set introspection query` ولكن سنجد أن الـ `repsponse` يعرض رسال خطأ توضح أن الفحص الذاتي ممنوع بسبب أن الاستعلام يحتوي على كلمة `__schema` أو كلمة `__type`
```http
HTTP/2 200 OK
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 156

{
  "errors": [
    {
      "locations": [],
      "message": "GraphQL introspection is not allowed, but the query contained __schema or __type"
    }
  ]
}
```

لذلك كل ماعلينا فعله هو تعديل الـ `request` ووضح `\n` ملاصقة لكلمة `__schema` لتصبح `__schema\n` وسنجد أن المخطط قد تم استدعائه في الاستجابة، فنقوم بالضغط على `Mouse left-click` داخل الـ `request` ونختار `GraphQL > Save GraphQL queries to site map`.

وبالتوجه إلى `Target > Site map` نقوم بفحص الاستعلامات الموجودة في مخطط `/api` وسنلاحظ وجود  `query` يُستعمل  لتحديد الـ `ID`  لحساب مستخدم معين، ونلاحظ وجود `Mutation` يقوم بحذف المستخدم بناءًا على رقم الـ `ID`، فنقوم بإرسال كلامهما إلى `repeater`
```GraphQL
query($id: Int!) {
  getUser(id: $id) {
    id
    username
  }
}
```
 وبالتعديل على جزئية الـ `Variables` داخل `Burp Suite` في الـ `query` بتغيير الـ `ID` ونقوم بإرسال الـ `request` فسنكتشف أن حساب `carlos` هو صاحب `ID` رقم `3` .

فنقوم بالتعديل على جزئية الـ `Variables` داخل `Burp Suite` بتغيير الـ `ID` في الـ `Mutation` لنحذف حساب `carlos` ونقوم بإرسال الـ `request`.

```GraphQL
mutation($input: DeleteOrganizationUserInput) {
  deleteOrganizationUser(input: $input) {
    user {
      id
      username
    }
  }
}
```

ونتوجه إلى اللاب مرة أخرى لتظهر رسالة `solved` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
لا يجب أبدًا الاعتماد على الفلاتر النصية لتأمين `GraphQL` ، بل يجب أن يتم إيقاف خاصية الفحص الذاتي من جذورها عبر إعدادت مكتبة أو محرك `GraphQL` المستخددم في السيرفر لأن إغلاقها من الإعدادات الهيكلية يضمن أن السيرفر سيرفض استعلامات الفحص الذاتي تماماً مهما تم التلاعب بالمسافات أو الأسطر.

ويجب تطبيق آليات التحقق من الصلاحيات `Authorization` داخل الدوال البرمجية `Resolvers` الخاصة بالـ `Mutations`، وقبل أن ينفذ السيرفر أمر الحذف يجب أن يقرأ جلسة المستخدم` (Session/Token)` ويتأكد من أنه يمتلك صلاحيات إدارية `(Admin Privileges)`، وإذا كان المستخدم عادياً أو غير مسجل دخول، يجب أن يتم رفض الطلب فوراً.

ويجب تقييد الوصول لبيانات المستخدمين، فعندما يتم طلب استعلام لبيانات مستخدم فإذا لم يكن الشخص هو صاحب الحساب المطلوب بياناته أو لم يمتلك صلاحيات إدارية فيجب أن يتم رفض الطلب فورًا  لمنع المهاجمين من حصد وتسريب معلومات النظام `Data Enumeration`.
