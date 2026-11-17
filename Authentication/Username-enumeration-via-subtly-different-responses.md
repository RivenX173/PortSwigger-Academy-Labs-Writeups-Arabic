# [Username enumeration via subtly different responses](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-subtly-different-responses)

## **`الهدف`**
معرفة اسم المستخدم الصالح من بين قائمة أسماء يوفرها موقع الـ `PortSwigger` ثم القيام بعمل `brute-force` علي كلمة المرور الخاصة بهذا المستخدم وأخيرًا تسجيل الدخول بنجاح إلى حسابه.
## **`خطوات الحل`**
**`ملاحظة:`** خطوات الحل الرسمي من الـ `PortSwigger` تختلف قليلًا عن الخطوات المكتوبة هنا لأنه يستعمل أداة `Burp Intruder`.

**`خطوات حل هذا اللاب تتشابه في جزء كبير مع خطوات حل لاب`**
[Username enumeration via subtly different responses](./Username-enumeration-via-subtly-different-responses.md)

ولكن هناك بعض الاختلافات في خطوات الحل هنا حيث أن ملاحظة الـ `response` المختلف  في هذا اللاب تعتمد على خانة في الـ `Turbo Intruder` تُسمى `Anomaly rank` وتعني `مقدار الانحراف/تصنيف الغرابة/الخروج عن المألوف.`

وسنلاحظ في النتائج عند اسم `acces` أن خانة الـ `Anomaly rank` مرتفعة بشكل ملحوظ حيث تبلغ `15310` مقارنة بالبقية الذين يتراوحون بين `654` و `3294`.

وبفحص الـ `response` الخاص بها مقارنة بأى `response` أخر من الموجودين في النتائج بأداة `Comparer` الموجودة في الـ `Burp Suite` سنلاحظ وجود اختلاف طفيف حيث ان جميع الـ `responses` بها هذا السطر

`<p class=is-warning>Invalid username or password.</p>`

ولكن في هذ الـ `response` الخاص باسم `acces` فالسطر لا يوجد به نقطة `(.)` بعد كلمة `password`

`<p class=is-warning>Invalid username or password </p>`

وبعد ذلك نكرر مافعلناه في اللاب السابق بتجربة قائمة كلمات المرور مع اسم `acces` لنلاحظ أن كلمة مرور `george` قد عادت برقم `302` في خانة الـ `Status code` وهذا يعنى أنها ناجحة عكس البقية الذين كانوا برقم `200`

لذلك نقوم بالعودة إلى اللاب وتسجيل الدخول إلى لحساب `acces:george` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`بالإضافة إلى جزء من ترقيع لاب`** [Username enumeration via subtly different responses](./Username-enumeration-via-subtly-different-responses.md)

فلا يجب الاعتماد على انتباه المُبرمج لتجنب الهفوات، بل يجب وضع قواعدة برمجية تمنع حدوثها من الأساس، فبدلاً من كتابة نص رسالة الخطأ يدوياً في عدة أماكن، يجب على المبرمج تعريف الرسالة مرة واحدة كمتغير ثابت في إعدادات التطبيق `Constants`، واستدعاء هذا المتغير `Constants` سواء فشل التحقق من اسم المستخدم أو فشل التحقق من كلمة المرور، يجب أن يقوم الكود بإرجاع هذا المتغير فقط. فحتى لو أخطأ المبرمج في صياغة الرسالة، فالرسالة ستُطبع كما هي في كلتا الحالتين، وستبقى الاستجابة متطابقة تماماً `Byte-for-Byte`، ولن يكون الـ `Anomaly rank` مرتفعًا ولن يجد الـ `Comparer` أي اختلاف.
