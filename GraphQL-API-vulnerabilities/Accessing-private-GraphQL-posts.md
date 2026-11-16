# [Accessing private GraphQL posts](https://portswigger.net/web-security/graphql/lab-graphql-reading-private-posts)

## **`الهدف`**
العثور على تدوينة مخفية واستعمال كلمة المرور السرية الموجودة داخلها كإجابة لحل اللاب.
## **`خطوات الحل`**
بعد الدخول إلى اللاب نقوم بالذهاب إلى `HTTP history` وسنجد أحد الـ `requests` بـ `URl` إلى `/graphql/v1` وبفحص الـ `response` الخاص به سنجد أنه لا يوجد مدونة بـ `ID` رقم `3` ولكن هناك `1,2,4,5`.

لذلك نقوم بإرسال هذا الـ `request` إلى `repeater` ونضغط `Mouse left-click` داخل الـ `request` ونختار `Set introspection query` ونقوم بعمل إرسال للـ `request` ونقوم بنسخ الـ `response` إلى موقع **[GraphQL Visualizer](https://nathanrandal.com/graphql-visualizer/)**، أو نسخ الـ `URL` إلى إضافة `InQL` المجانية داخل `Burp Suite`.

وبفحص الاستجابة التي تحتوى على المخطط سنكتشف أنه يجب علينا أن نعود إلى الـ `repeater` مرة أخرى فى نفس الـ `request` ونقوم بإرسال هذا  الـ `query`.
```GraphQL
query GetPostPasswordById {
  getBlogPost(id: 3) {
    id
    postPassword
  }
}
```

وسنجد كلمة المرور ظاهرة في الاستجابة `j6dn2cs8x4ijl3rzmcbvdov0qb556u5o` فنقوم بنسخها ونتوجه إلى اللاب ونضغط على `Submit Solution` وندخل كلمة المرور لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب إيقاف خاصية الفحص الذاتي تمامًا في بيئة الإنتاج، وعلى الرغم من أن إغلاقها لن يمنع المهاجم من تخمين الحقول، ولكنه سيجعل الأمر أكثر صعوبة.

ويجب أن يتحقق النظام أولاً مما إذا كان المطلوب هو شئ متاح للعامة أم لا، فإذا كانت مخفية، يجب أن يتحقق النظام من صلاحية الوصول `Authorization` كأن يكون مدير أو محرر أو يكون هو كاتب هذه التدوينة، فإذا لم يكن كذلك، يجب أن يتم رفض الطلب فورًا وإرجاع رسالة خطأ `(Unauthorized / Forbidden)`.
