# [User role controlled by request parameter](https://portswigger.net/web-security/access-control/lab-user-role-controlled-by-request-parameter)

## **`الهدف`**
الدخول إلى لوحة تحكم الإدارة وحذف المُستخدم `carlos.`
## **`خطوات الحل`**
نقوم بالدخول إلى حساب `wiener:peter` ونقوم باعتراض الـ `respone` الخاص بتسجيل الدخول 
```HTTP
HTTP/2 302 Found
Location: /my-account?id=wiener
Set-Cookie: Admin=False; Secure; HttpOnly
Set-Cookie: session=ZLs9QrwdSxNZDsFuQ00Ft9zvF2UPyYSK; 
```
ونقوم بتعديل `Admin=False` إلى `Admin=True` ثم نمرر الـ `response` لنلاحظ وجود زر `Admin panel` وبالضغط عليه نقوم بالدخول إلى لوحة التحكم ونقوم بحذف المُستخدم `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`الترقيع هو جزء كبير من ترقيع لاب`** [Unprotected admin functionality](./Unprotected-admin-functionality.md)
