# [Client-side prototype pollution via browser APIs](https://portswigger.net/web-security/prototype-pollution/client-side/browser-apis/lab-prototype-pollution-client-side-prototype-pollution-via-browser-apis)

## **`الهدف`**
قم بالعثور على مصدر يمكنك استخدامه لإضافة خصائص عشوائية إلى `Object.prototype`، ثم قم بتحديد خاصية من خصائص الأداة `Gadget` التي تسمح لك بتنفيذ جافا سكريبت عشوائية، ثم قم بدمجهم لاستدعاء `alert()`.
## **`خطوات الحل`**
==`ملاحظة هامة:`==لقد تم حل هذا اللاب بإستخدام `DOM Invader` لذلك لا يوجد خطوات يدوية.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [DOM XSS via client-side prototype pollution](./DOM-XSS-via-client-side-prototype-pollution.md)

ويجب عند استخدام دالة `Object.defineProperty()` لحماية خصائص الكائنات `Properties` وجعلها غير قابلة للتعديل، أن يتم تعريف وصف القيمة `value` صراحة داخل إعدادات الدالة `Descriptors`، لأنه في حال إغفال تمريرها، سيقوم محرك اللغة تلقائياً بالبحث عن هذه القيمة المفقودة في سلسلة الوراثة `Prototype Chain`، مما يسمح للمهاجم بتلويث القيمة الموروثة عبر النموذج الأولي الأساسي `Object.prototype` وتمرير الـ `Payload` الخبيث الخاص به، لذا يجب دائماً تعيين القيمة بشكل مباشر وصريح لقطع الطريق على أي وراثة خبيثة.
