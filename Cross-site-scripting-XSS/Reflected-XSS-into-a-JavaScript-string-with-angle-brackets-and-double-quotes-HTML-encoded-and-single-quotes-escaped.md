# [Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-angle-brackets-double-quotes-encoded-single-quotes-escaped)

## **`الهدف`**
قم بتنفيذ هجوم `XSS` يهرب من السلسلة النصية لـ `JS` ويستدعي دالة `alert`.
## **`خطوات الحل`**
نقوم بحقن الأمر التالي فى خانة البحث ونضغط `Search` لننتهي من حل اللاب.
```JS
\';alert(document.domain)//
```
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Reflected XSS into HTML context with nothing encoded](./Reflected-XSS-into-HTML-context-with-nothing-encoded.md)

ويجب على الترميز أن يكون `Context-Aware` وتحديدًا `JavaScript Escaping` مثل تحويل `\` إلى `\\`، و `'` إلى `\x27` أو `\u0027`)
