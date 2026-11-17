# [HTTP/2 request splitting via CRLF injection](https://portswigger.net/web-security/request-smuggling/advanced/lab-request-smuggling-h2-request-splitting-via-crlf-injection)

## **`الهدف`**
قم بحذف حساب `Carlos` باستخدام تقنية تسميم قائمة انتظار الاستجابة للوصول إلى لوحة التحكم `/admin`.
## **`خطوات الحل`**
سنقوم بتحويل طلب الدخول إلى اللاب إلى `Repeater`
```http
GET / HTTP/2
Host: 0ab400a204e7ca808003993500920079.web-security-academy.net
Cookie: session=n0cyZSVZZJHgAdbq1aQ2uVX3FXdgYsIo
```

ثم سنقوم بتنفيذ `HTTP/2 request splitting` بخدعة `Kettled` وهذا يجب أن يتم من داخل `Inspector` عن طريق إضافة ترويسة بأي أسم مثلا `TEST1` وتكون قيمتها كاملة كالتالي:
```http
TEST2\r\n
\r\n
GET /test HTTP/1.1\r\n
Host: 0aea00c6034f88d080fc6c2900020015.web-security-academy.net\r\n
Cookie: session=2prPPWi4wLlF5eufpqUWfMVmCxbj19aZ
```
ولا يمكن أن يتم هذا يدويًا في التمثيل النصي لـ`HTTP/2`.
2. نقوم بتعديل مسار الطلب الأصلى أو قيمة حقل `:path` داخل `Inspector` إلى `/test` لأن وضع مسار زائف للطلب الأصلي والطلب المُهرب يساعدنا على التفرقة بين الاستجابات الأصلية والمسروقة.

**`ملاحظة هامة جدًا:`** الطلب السابق لن يظهر بهذا الشكل داخل `Burp Suite` بسبب حقن بيانات داخل قيمة ترويسة `TEST1` فلا يمكن عرضه بنظام ال `text` لذلك سيتغير الشكل قليًلا ولن يكون ظاهرًا إلا نص الـ `Body` فقط.

وسنقوم بإرساله مرة أو اثنين ثم ننتظر بضع ثوان ونعيد إرساله حتى نحصل على الاستجابة التالية:
```http
HTTP/2 302 Found
Location: /my-account?id=administrator
Set-Cookie: session=eYSFevbMAwFYfAGoqKVqf5tMQnSuaBT0; Secure; HttpOnly; SameSite=None
X-Frame-Options: SAMEORIGIN
Content-Length: 0
```

وقد نحتاج إلى تكرار هذه العملية عشرات المرات حتى نحصل على هذه الاستجابة السابقة، وإذا صادفنا فى وقت ما أن الاستجابات التى نحصل عليها أصبحت هي نفسها الاستجابات الأصلية الخاصة بنا فيجب علينا إرسال أى طلب أصلى عدة مرات لإعادة ضبط الاتصال ثم نعود ونكرر الفعل مرة أخرى حتى نحصل على الـ `Session` الخاصة بحساب `administrator`

بعد ذلك يمكننا إرسال طلب إلى مسار `/admin/delete?username=carlos` أو يمكننا  أن نتوجه إلى المتصفح ونضغط `F12` ثم  `Application` ونضعه في خانة الـ `Session` داخل الـ `Cookies` ثم نضغط `Enter` لنجد أننا داخل الحساب وندخل يدويًا إلى لوحة التحكم ونحذف حساب `Carlos` لننتهي من حل اللاب. 
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [Response queue poisoning via H2.TE request smuggling](./Response-queue-poisoning-via-H2.TE-request-smuggling.md)
