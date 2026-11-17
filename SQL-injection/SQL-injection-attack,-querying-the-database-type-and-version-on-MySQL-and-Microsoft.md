# [SQL injection attack, querying the database type and version on MySQL and Microsoft](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-mysql-microsoft)

## **`الهدف`**
قم بتنفيذ هجوم `UNION SQLi` ليعرض إصدار قاعدة البيانات.
## **`خطوات الحل`**
**`ملاحظة:`** البورتسويجر يستعمل في هذ اللاب قاعدة بيانات من نوع `MySQL` أو `Microsoft`

سنقوم بتكرار نفس خطوات حل [هذا اللاب](./SQL-injection-UNION-attack,-determining-the-number-of-columns-returned-by-the-query.md) لتحديد عدد الأعمدة بإستخدام `NULL` وسنجد أنها عمودين
```sql
' UNION SELECT NULL,NULL-- 
``` 
**`ملاحظة هامة:`** في `MySQL`، يجب أن يتبع تسلسل الشرطتين المتتاليتين مسافة، وإلا سيفشل الهجوم، ويمكن بدلاً من ذلك، استخدام رمز الهاش/الشباك `#` للدلالة على التعليق.

بعد ذلك سنقوم بمحاولة لتغيير كل `NULL` بأى سلسلة نصية مثل: 'A' وسنكتشف أن كلا العمودين يدعموا النصوص، بعد ذلك سنقوم بحقن أيًا من هذين الأمرين لأن كلامهما سيفعل نفس الشئ، بما أن كلا العمودين يدعموا النصوص.
```sql
Gifts' UNION SELECT NULL,@@version-- 
أو
Gifts' UNION SELECT @@version,NULL-- 
```
لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](./SQL-injection-vulnerability-in-WHERE-clause-allowing-retrieval-of-hidden-data.md)
