# [Blind SQL injection with time delays](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays)

## **`الهدف`**
قم بتنفيذ هجوم `SQLi` يتسبب في تأخير `10` ثواني.
## **`خطوات الحل`**
**`ملاحظة:`** البورتسويجر يستعمل في هذ اللاب قاعدة بيانات من نوع `PostgreSQL`

سنقوم بتكرار نفس خطوات حل [اللاب السابق](./Blind-SQL-injection-with-conditional-responses.md) ولكن سنقوم بحقن أمر استعلام التالي في الـ `TrackingId` داخل `Inspector`
```sql
'||pg_sleep(10)--
```
لنتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](./SQL-injection-vulnerability-in-WHERE-clause-allowing-retrieval-of-hidden-data.md)

ولكن بتعديل بسيط في النقطة الثانية من الترقيع حيث ستصبح:  
ويجب التحقق من مدخلات المستخدم ومطابقتها مع قائمة بيضاء محددة مسبقا قبل تمريرها إلى قاعدة البيانات، وبما أن قيمة الـ `TrackingId` داخل الـ `Cookie` تكون معروفة الهيكلة وتتكون فقط من حروف وأرقام `Alphanumeric` وبطول ثابت، فيجب ألا يتم قبول أي مدخل يخرج عن هذا النطاق مثل احتوائه على علامات اقتباس أو رموز خاصة أو بطول أكبر من المُحدد، ويقوم التطبيق برفض الطلب فوراً إذا كان المدخل غير صالح.
