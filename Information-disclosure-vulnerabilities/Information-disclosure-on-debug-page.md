# [Information disclosure on debug page](https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-on-debug-page)

## **`الهدف`**
قم بالحصول على متغير البيئة `SECRET_KEY` من خلال صفحة تصحيح الأخطاء وقدمه كـ `Solution`. 
## **`خطوات الحل`**
سنقوم بتحويل طلب الدخول إلى اللاب إلى `Repeater`
```http
GET / HTTP/2
Host: 0a2a0074032f104e80a83a9100b30078.web-security-academy.net
Cookie: session=wrKXsgQkNJrtDTGUqtmPrI8AbfGIeXhT
```

وبفحص الاستجابة الخاصة به سنجد بداخلها تعليق من المطور يشير إلى مسار ملف الـ `Debug`
```http 
<!-- <a href=/cgi-bin/phpinfo.php>Debug</a> -->
```
سنقوم بالتوجه إلى هذا المسار ثم سنقوم بنسخ قيمة متغير البيئة `SECRET_KEY` ونضغط على `Submit solution` ونضعه في الحقل المناسب ثم نضغط `OK` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب حذف ملفات تصحيح الأخطاء من بيئة الإنتاج.

ويجب إزالة التعليقات البرمجية الحساسة من كود الـ `HTML` الموجه للمستخدم، ويُفضل استخدام أدوات التحزيم `Build Tools` أو الـ `Minifiers` لحذف هذه التعليقات تلقائياً قبل نشر الكود.

ويجب إيقاف وضع التصحيح `Debug Mode` في بيئة الانتاج، والاعتماد بدلًا من ذلك على سجلات الأخطاء `Error Logs` الداخلية.

وإذا كانت هناك ضرورة ملحة لوجود صفحات تشخيصية لمراقبة حالة السيرفر فيجب حمايتها بآليات مصادقة `Authentication` قوية وتقييد الوصول إليها `IP Whitelisting` ليقتصر على الشبكة الداخلية الخاصة بفريق التطوير فقط.
