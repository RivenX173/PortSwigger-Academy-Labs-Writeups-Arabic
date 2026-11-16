# [Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-template-literal-angle-brackets-single-double-quotes-backslash-backticks-escaped)

## **`الهدف`**
قم بتنفيذ هجوم `XSS` يستدعي دالة `alert` داخل قالب نصي `template string`.
## **`خطوات الحل`**
نقوم بحقن الأمر التالي فى خانة البحث ونضغط `Search` لننتهي من حل اللاب.
```JS
${alert(document.domain)}
```
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Reflected XSS into HTML context with nothing encoded](./Reflected-XSS-into-HTML-context-with-nothing-encoded.md)

ويجب على الترميز أن يكون `Context-Aware` وتحديدًا `JavaScript Escaping` مثل تحويل `$` إلى `\u0024`.

وبجب تجنب حقن بيانات المستخدم مباشرة داخل أكواد الجافاسكريبت، وتمريرها بدلاً من ذلك كبيانات منفصلة (مثلاً داخل عنصر `HTML` مخفي أو كـ `JSON`) ثم قراءتها برمجياً.
