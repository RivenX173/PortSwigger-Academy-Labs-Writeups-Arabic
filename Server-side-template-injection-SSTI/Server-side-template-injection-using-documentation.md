# [Server-side template injection using documentation](https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-using-documentation)

## **`الهدف`**
قم بتحديد محرك القوالب واستخدم الوثائق لمعرفة كيف تنفذ كود على السيرفر ثم قم بحذف ملف `morale.txt` الموجود في `Carlos's home directory`.
## **`خطوات الحل`**
سنقوم بتسجيل الدخول إلى حساب `content-manager:C0nt3ntM4n4g3r` وبعد ذلك سنتوجه إلى أى منتج وسنقوم بالضغط على `Edit template` وبعد ذلك نقوم بتعديل متغير من المتغيرات مثل تغيير `${product.name}` إلى `${test}` ثم نقوم بضغط `Save`

وسنجد رسالة خطأ طويلة توضح أن محرك القوالب المستخدم هو `freemaker`

وبعد قراءة معمقة في وثائق `freemaker` سنصل إلى الـ `Payload` التالي:
```JAVA
<#assign ex="freemarker.template.utility.Execute"?new()> ${ ex("rm /home/carlos/morale.txt") }
```

وسنقوم بالضغط مرة أخرى على `Edit template` ونقوم بحذف المتغير `${test}` ونضع مكانه الـ `Payload` ونضغط `Save` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب ضبط إعدادات محرك `FreeMarker` لمنع دالة `?new()` من استدعاء كلاسات الجافا أو حصرها في قائمة بيضاء صارمة.

وإذا كان النظام يُحتم على المستخدم تعديل القوالب حتى لو كان مدير المحتوى فقط، فيجب استبدال `FreeMarker` بمحركات قوالب مبسطة لا تدعم تنفيذ `RCE`، أو أن تتم عملية معالجة القالب `Template Rendering`في بيئة معزولة تمامًا مثل حاوية `Docker` أو `Sandbox` بحيث لا تملك صلاحية الوصول إلى ملفات النظام الحساسة أو الشبكة الداخلية.

وإذا لم يكن مدير المحتوى يحتاج حقًا إلى تعديل كود القالب نفسه `(HTML/Template Tags)` فيجب إعطاءه أدوات لتعديل المحتوى فقط مثل محررات النصوص أو دعم لغة `Markdown` بدلًا من منحهم صلاحية تعديل القالب المعماري للصفحة.

ويجب ألا يعمل تطبيق الويب بصلاحيات مستخدم `Root` ويعمل بصلاحيات مستخدم محدود مثل `www-data` بحيث لا يملك صلاحية التعديل أو الحذف إلا في مجلدات محددة خاصة بالتطبيق مثل `/var/www/html/uploads`.
