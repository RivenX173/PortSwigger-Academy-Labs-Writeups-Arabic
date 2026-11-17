# [Reflected XSS into attribute with angle brackets HTML-encoded](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-attribute-angle-brackets-html-encoded)

## **`الهدف`**
قم بتنفيذ هجوم `XSS` يقوم بحقن سمة `attribute` تستدعي دالة `alert`.
## **`خطوات الحل`**
نقوم بحقن الأمر التالي فى خانة البحث ونضغط `Search` لننتهي من حل اللاب.
```html
"onmouseover="alert(1)
```
حيث أن هذا اللاب يقوم بطباعة أي كلمة بحث تكتبها بدال علامتي تنصيص `""`.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [Reflected XSS into HTML context with nothing encoded](./Reflected-XSS-into-HTML-context-with-nothing-encoded.md)
