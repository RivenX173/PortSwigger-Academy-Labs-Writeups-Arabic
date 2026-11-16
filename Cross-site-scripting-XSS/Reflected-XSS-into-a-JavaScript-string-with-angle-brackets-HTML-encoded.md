# [Reflected XSS into a JavaScript string with angle brackets HTML encoded](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-angle-brackets-html-encoded)

## **`الهدف`**
قم بتنفيذ هجوم `XSS` يهرب من السلسلة النصية لـ `JS` ويستدعي دالة `alert`.
## **`خطوات الحل`**
نقوم بحقن الأمر التالي فى خانة البحث ونضغط `Search` لننتهي من حل اللاب.
```JS
'-alert(document.domain)-'
```
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Reflected XSS into HTML context with nothing encoded](./Reflected-XSS-into-HTML-context-with-nothing-encoded.md)
