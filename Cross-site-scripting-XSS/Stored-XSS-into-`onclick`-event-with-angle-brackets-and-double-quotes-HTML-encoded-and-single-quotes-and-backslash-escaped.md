# [Stored XSS into `onclick` event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-onclick-event-angle-brackets-double-quotes-html-encoded-single-quotes-backslash-escaped)

## **`الهدف`**
قم بتنفيذ هجوم `Stored XSS` في تعليق يستدعي دالة `alert` عند الضغط على اسم صاحب التعليق.
## **`خطوات الحل`**
نقوم بالدخول على أى تدوينة ثم نقوم بحقن الأمر التالي فى خانة `Website` ونملأ باقي الحقول بأى بيانات عشوائية ثم نضغط `Post Comment` لننتهي من حل اللاب.
```URL
http://test?&apos;-alert(1)-&apos;
```

وسبب نجاح هذا الـ `Payload`:

- الـ `&apos;`: ويمثل علامة الاقتباس الفردية (`'`)، حيث يستخدم هذا الترميز لتجاوز الـ `WAF` أو فلاتر الإدخال التي تحظر رمز `'` الصريح، وإذا وُضع داخل خاصية `HTML`، مثل خصائص الأحداث: `onclick`، `onload`، أو سمات الـ `HTML`، فإن المتصفح يقوم بفك ترميز `&apos;` تلقائياً إلى `'` قبل إرسال الكود لمحرك الجافا سكريبت.
- الـ `-`: في جافا سكريبت، تُستخدم علامة الطرح (`-`) أو الجمع (`+`) كحيلة لتنفيذ الدالة `alert(1)` داخل سياق نصي قائم بالفعل، حيث يحاول المحرك حساب ناتج العملية فينفذ `alert(1)` أولاً دون إحداث خطأ في بناء الجملة `Syntax Error`.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Stored XSS into HTML context with nothing encoded](./Stored-XSS-into-HTML-context-with-nothing-encoded.md)

ويجب تجنب وضع بيانات المستخدم داخل خصائص مثل `onclick` تماماً.

ويجب على الترميز أن يكون `Context-Aware` وتحديدًا `JavaScript Escaping` مثل تحويل `\` إلى `\\`، و `'` إلى `\x27` أو `\u0027`).

وإذا كنت مضطراً لوضع البيانات داخل `onclick`، يجب عمل `JS Escaping` أولاً (تحويل `'` إلى `\x27` أو `\u0027`)، **ثم** عمل `HTML Encoding` للناتج بالكامل.
