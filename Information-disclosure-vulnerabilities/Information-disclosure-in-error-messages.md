# [Information disclosure in error messages](https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-in-error-messages)

## **`الهدف`**
قم بالحصول على رقم إصدار الـ `Framework` المستخدم وقدمه كـ `Solution`. 
## **`خطوات الحل`**
سنقوم بالدخول إلى أى منتج في اللاب، ثم نتوجه لـ `HTTP history` ونقوم بتوجيه طلب الوصول لصفحة هذا المنتج إلى `Repeater`
```http
GET /product?productId=1 HTTP/2
Host: 0aae003d03bc3e1c81c148250022006c.web-security-academy.net
Cookie: session=8R3oxHbYblyO1I1IarbH3btyNt3TYOO2
```

بعد ذلك نقوم بتغيير قيمة معلم `productID` إلى أى نص مثل `test`
```http
GET /product?productId=test HTTP/2
Host: 0aae003d03bc3e1c81c148250022006c.web-security-academy.net
Cookie: session=8R3oxHbYblyO1I1IarbH3btyNt3TYOO2
```
وسنجد أن الاستجابة هي رسالة خطأ مطولة فيها رقم إصدار الـ `Framework`.
```http
HTTP/2 500 Internal Server Error
Content-Length: 1679

Internal Server Error: java.lang.NumberFormatException: For input string: "test"
	at java.base/java.lang.NumberFormatException.forInputString(NumberFormatException.java:67)
	at java.base/java.lang.Integer.parseInt(Integer.java:661)
	at java.base/java.lang.Integer.parseInt(Integer.java:777)
	at lab.w.i.h.e.W(Unknown Source)
	at lab.m.l.h.r.O(Unknown Source)
	at lab.m.l.a.k.t.M(Unknown Source)
	at lab.m.l.a.m.lambda$handleSubRequest$0(Unknown Source)
	at d.x.w.t.lambda$null$3(Unknown Source)
	at d.x.w.t.D(Unknown Source)
	at d.x.w.t.lambda$uncheckedFunction$4(Unknown Source)
	at java.base/java.util.Optional.map(Optional.java:260)
	at lab.m.l.a.m.B(Unknown Source)
	at lab.server.f.o.v.V(Unknown Source)
	at lab.m.l.e.X(Unknown Source)
	at lab.m.l.e.V(Unknown Source)
	at lab.server.f.o.d.r.z(Unknown Source)
	at lab.server.f.o.d.n.lambda$handle$0(Unknown Source)
	at lab.w.d.o.p.s(Unknown Source)
	at lab.server.f.o.d.n.X(Unknown Source)
	at lab.server.f.o.i.V(Unknown Source)
	at d.x.w.t.lambda$null$3(Unknown Source)
	at d.x.w.t.D(Unknown Source)
	at d.x.w.t.lambda$uncheckedFunction$4(Unknown Source)
	at lab.server.jc.N(Unknown Source)
	at lab.server.f.o.i.n(Unknown Source)
	at lab.server.f.h.a.I(Unknown Source)
	at lab.server.f.r.g(Unknown Source)
	at lab.server.f.d.g(Unknown Source)
	at lab.server.jg.X(Unknown Source)
	at lab.server.jg.O(Unknown Source)
	at lab.a.v.lambda$consume$0(Unknown Source)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1144)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:642)
	at java.base/java.lang.Thread.run(Thread.java:1583)

Apache Struts 2 2.3.31
```

ثم نقوم بنسخ رقم الإصدار وهو `Apache Struts 2 2.3.31` ونضغط على `Submit solution` ونضعه في الحقل المناسب ثم نضغط `OK` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب معالجة المُدخلات برمجياً باستخدام `try-catch` لاصطياد الأخطاء المتوقعة (مثل إدخال نص بدلًا من رقم) وإرجاع رسالة خطأ قياسية للمستخدم مثل `HTTP 400`.

ويجب إعداد السيرفر ليعترض أي خطأ داخلي مثل `500 Internal Server Error` ويعرض بدلاً منه صفحة `HTML` بسيطة للمستخدم لا تحتوي على أي تفاصيل تقنية مع التأكد من تسجيل هذه االأخطاء بالتفصيل بشكل آمن في ملفات السيرفر الداخلية `Logs` فقط.

ويجب التأكد من تعطيل وضع التطوير `Development Mode` لأنه عادة ما يظهر رقم إصدار الـ `Framework` لان التطبيق مايزال يعمل في وضع التطوير بدلًا من وضع الإنتاج.

ويجب تحديث `Apache Struts 2 2.3.31` إلى أحدث إصدار مستقر ومُصحح فوراً لتجنب ثغرات الـ `RCE` المعروفة في هذا الإصدار القديم.
