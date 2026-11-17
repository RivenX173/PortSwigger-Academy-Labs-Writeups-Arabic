# [File path traversal, validation of start of path](https://portswigger.net/web-security/file-path-traversal/lab-validate-start-of-path)

## **`الهدف`**
الحصول على محتويات ملف `/etc/passwd`
## **`خطوات الحل`**
**`خطوات حل هذا اللاب هي نفسها خطوات حل لاب`** [File path traversal, simple case](./File-path-traversal,-simple-case.md)

باختلاف واحد فقط وهو أننا سنقوم بتغيير الجزئية الموجودة في أول سطر من الـ `request` إلى `/var/www/images/../../../etc/passwd` 
ونقوم بتمرير، لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [File path traversal, simple case](./File-path-traversal,-simple-case.md)
