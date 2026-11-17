# [Stored XSS into anchor `href` attribute with double quotes HTML-encoded](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-href-attribute-double-quotes-html-encoded)

## **`الهدف`**
قم بتنفيذ هجوم `Stored XSS` في `href attribute` في تعليق يستدعي دالة `alert`.
## **`خطوات الحل`**
نقوم بالدخول على أى تدوينة ثم نقوم بحقن الأمر التالي فى خانة `Website` ونملأ باقي الحقول بأى بيانات عشوائية ثم نضغط `Post Comment` لننتهي من حل اللاب.
```JS
javascript:alert(1)
```
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [Stored XSS into HTML context with nothing encoded](./Stored-XSS-into-HTML-context-with-nothing-encoded.md)

ويجب رفض أي رابط لا يبدأ صراحةً بـ `http://` أو `https://` في حقول الإدخال عن طريق `Protocol Whitelisting`.
