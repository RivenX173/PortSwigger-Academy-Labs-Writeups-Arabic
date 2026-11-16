# [File path traversal, traversal sequences blocked with absolute path bypass](https://portswigger.net/web-security/file-path-traversal/lab-absolute-path-bypass)

## **`الهدف`**
الحصول على محتويات ملف `/etc/passwd`
## **`خطوات الحل`**
**`خطوات حل هذا اللاب هي نفسها خطوات حل لاب` [File path traversal, simple case](./File-path-traversal,-simple-case.md)

باختلاف واحد فقط وهو أننا سنستعمل المسار المطلق حيث سنقوم بتغيير الجزئية الموجودة في أول سطر من الـ `request` إلى `/etc/passwd` ونقوم بتمريره، لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [File path traversal, simple case](./File-path-traversal,-simple-case.md)
