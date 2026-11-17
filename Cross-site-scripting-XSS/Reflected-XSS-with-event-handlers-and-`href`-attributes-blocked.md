# [Reflected XSS with event handlers and `href` attributes blocked](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-event-handlers-and-href-attributes-blocked)

## **`الهدف`**
قم بتنفيذ هجوم `XSS` عندما يتم الضغط عليه يستدعي دالة `alert`.
## **`خطوات الحل`**
نقوم بحقن الأمر التالي فى خانة البحث ونضغط `Search` لننتهي من حل اللاب.
```html
<svg><a><animate attributeName=href values=javascript:alert(1) /><text x=20 y=20>Click me</text></a>
```

وسبب نجاح هذا الـ `Payload`:

- الـ `<svg>`: هو عنصر داخل الـ `HTML`، ووظيفته هنا توفير بيئة عمل تدعم وسوم التحريك والروابط الخاصة بـ `SVG`.

- الـ `<a>`: وسم الرابط داخل الـ `SVG`؛ يختلف عن رابط `HTML` العادي بأنه يعتمد على معيار `SVG/XLink`، ومسؤول عن توجيه المستخدم عند النقر.
- الـ `<animate>`: وسم تحريك وظيفته تعديل خصائص العناصر تلقائياً.

- الـ `attributeName=href`: يحدد الخاصية المراد تعديلها، وهي رابط الزر `href`.

- الـ `values=javascript:alert(1)`: القيمة التي سيتم حقنها داخل الـ `href`، وهي كود جافاسكريبت.

- الـ `<text x=20 y=20>Click me</text>`: هو النص الظاهر للمستخدم للنقر عليه.

وبمجرد أن يعرض المتصفح هذا العنصر، يقوم وسم `<animate>` فوراً وبشكل ديناميكي بتعيين الخاصية `href` لعنصر `<a>` لتصبح قيمتها `javascript:alert(1)`. عندما يقوم الضحية بالنقر على النص `Click me`، يُشغل المتصفح كود الجافاسكريبت ويظهر صندوق التنبيه.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [Reflected XSS into HTML context with nothing encoded](./Reflected-XSS-into-HTML-context-with-nothing-encoded.md)
