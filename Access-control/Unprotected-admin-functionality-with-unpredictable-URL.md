# [Unprotected admin functionality with unpredictable URL](https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality-with-unpredictable-url)

## **`الهدف`**
الدخول إلى لوحة تحكم الإدارة وحذف المُستخدم `carlos.`
## **`خطوات الحل`**
نقوم باعتراض أي `response` لنلاحظ وجود هذا الكود والذي يفضح مسار لوحة التحكم.
```html
<script>
var isAdmin = false;
if (isAdmin) {
   var topLinksTag = document.getElementsByClassName("top-links")[0];
   var adminPanelTag = document.createElement('a');
   adminPanelTag.setAttribute('href', '/admin-jluvdv');
   adminPanelTag.innerText = 'Admin panel';
   topLinksTag.append(adminPanelTag);
   var pTag = document.createElement('p');
   pTag.innerText = '|';
   topLinksTag.appendChild(pTag);
}
</script>
```

لنتوجه بعد ذلك إلى مسار `/admin-jluvdv` ونقوم بحذف المُستخدم `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`بالإضافة إلى جزء كبير من ترقيع لاب` [Unprotected admin functionality](./Unprotected-admin-functionality.md)
فيجب على السيرفر أن يتحقق من هوية المُستخدم والـ `Session` الخاصة به قبل إرسال صفحة الـ `HTML` وإذا لم يكن المُستخدم مديرًا `Admin` فيجب على السيرفر أن يقوم بمسح كود الجافا سكريبت هذا تمامًا من الصفحة قبل خروجها من السيرفر للذهاب للمتصفح، لكي لا يستطيع المُستخدم العادي رؤية متغير `isAdmin` أو مسار لوحة التحكم `/admin-jluvdv` نهائيًا.
