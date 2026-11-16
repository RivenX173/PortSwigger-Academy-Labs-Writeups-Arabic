# [SQL injection UNION attack, finding a column containing text](https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text)

## **`الهدف`**
قم بتنفيذ هجوم `UNION SQLi` يقوم بإسترجاع/عرض القيمة النصية المطلوبة/العشوائية التي يتم توفيرها في الصفحة الرئيسية من اللاب.
## **`خطوات الحل`**
سنقوم بتكرار نفس خطوات حل [اللاب السابق](./SQL-injection-UNION-attack,-determining-the-number-of-columns-returned-by-the-query.md) لتحديد عدد الأعمدة بإستخدام `NULL` وسنجد أنها ثلاثة أعمدة
```sql
' UNION SELECT NULL,NULL,NULL--
``` 
بعد ذلك سنقوم بعدة محاولات لتغيير كل `NULL` بالنص الذي يتم توفير في هذا اللاب وهو `'4XX49U'` وستنجح المحاولة الثانية ويتم إسترجاع النص المطلوب.
```sql
' UNION SELECT NULL,'4XX49U',NULL--
```
لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](./SQL-injection-vulnerability-in-WHERE-clause-allowing-retrieval-of-hidden-data.md)
