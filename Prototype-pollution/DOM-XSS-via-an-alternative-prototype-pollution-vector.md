# [DOM XSS via an alternative prototype pollution vector](https://portswigger.net/web-security/prototype-pollution/client-side/lab-prototype-pollution-dom-xss-via-an-alternative-prototype-pollution-vector)

## **`الهدف`**
قم بالعثور على مصدر يمكنك استخدامه لإضافة خصائص عشوائية إلى `Object.prototype`، ثم قم بتحديد خاصية من خصائص الأداة `Gadget` التي تسمح لك بتنفيذ جافا سكريبت عشوائية، ثم قم بدمجهم لاستدعاء `alert()`.
## **`خطوات الحل`**
==`ملاحظة هامة:`==لقد تم حل هذا اللاب بإستخدام `DOM Invader` لذلك لا يوجد خطوات يدوية.

ولكن بعد الضغط على `Exploit` في الخطوة الأخيرة، ستجد أن الاستغلال الخاص بالـ `DOM Invader` لا يعمل لذلك ستحتاج لإضافة علامة `-` في نهاية رابط الصفحة الجديدة التي فُتحت تلقائيًا لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [DOM XSS via client-side prototype pollution](./DOM-XSS-via-client-side-prototype-pollution.md)
