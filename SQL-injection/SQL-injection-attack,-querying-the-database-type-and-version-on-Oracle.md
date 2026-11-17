# [SQL injection attack, querying the database type and version on Oracle](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-oracle)

## **`الهدف`**
قم بتنفيذ هجوم `UNION SQLi` ليعرض إصدار قاعدة البيانات.
## **`خطوات الحل`**
**`ملاحظة:`** البورتسويجر يستعمل في هذ اللاب قاعدة بيانات من نوع `Oracle`

سنقوم بتكرار نفس خطوات حل [هذا اللاب](./SQL-injection-UNION-attack,-determining-the-number-of-columns-returned-by-the-query.md) لتحديد عدد الأعمدة بإستخدام `NULL` وسنجد أنها عمودين.
```sql
' UNION SELECT NULL,NULL FROM DUAL--
``` 
**`ملاحظة هامة:`** بما أن قاعدة البيانات من نوع `Oracle` فيجب أن تستخدم كل استعلامات `SELECT` الكلمة المفتاحية `FROM` وأن تحدد جدولًا صالحًا، ويوجد جدول مدمج `built-in` في أوراكل `Oracle` يُسمى `dual` يمكن استخدامه لهذا الغرض.

بعد ذلك سنقوم بمحاولة لتغيير كل `NULL` بأى سلسلة نصية مثل: 'A' وسنكتشف أن كلا العمودين يدعموا النصوص، بعد ذلك سنقوم بحقن أيًا من هذين الأمرين لأن كلامهما سيفعل نفس الشئ.
```sql
' UNION SELECT banner,NULL FROM v$version--
أو
' UNION SELECT NULL,banner FROM v$version--
```
لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](./SQL-injection-vulnerability-in-WHERE-clause-allowing-retrieval-of-hidden-data.md)
