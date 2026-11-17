# [Multi-step process with no access control on one step](https://portswigger.net/web-security/access-control/lab-multi-step-process-with-no-access-control-on-one-step)

## **`الهدف`**
الدخول بحساب `wiener:peter` وترقية حسابك إلى مدير `(administrator).`
## **`خطوات الحل`**
نقوم بالدخول إلى حساب `administrator:admin` ومحاولة رفع صلاحيات `carlos` ونلاحظ أن هناك خطوة ثانية تأكيدية تظهر رسالة `Are you sure?` مع وجود خيارين `Yes` أو `No,take me back`، لذلك نضغط على `Yes` ونذهب إلى `Burp Suite` لنرى الـ `request` الخاص بهذه الخطوة لنفهم المكتوب بها ومما تتكون.

بعد ذلك نقوم بالخروج من الحساب والدخول إلى حساب `wiener:peter` ونقوم باعتراض أي `request` مثل الضغط على `My account`

```http
GET /my-account?id=wiener HTTP/2
Host: 0a3000a904c02213809aadfa00700082.web-security-academy.net
Cookie: session=Akb2VAD6jRkd08mrBMvL1v8ybrZe7zav
```

ونقوم بتغيير الـ `Method` إلى `POST` والمسار إلى `/admin-roles`، ونقوم بإضافة `action=upgrade&confirmed=true&username=wiener` في السطر الأخير
```http
POST /admin-roles HTTP/2
Host: 0a3000a904c02213809aadfa00700082.web-security-academy.net
Cookie: session=7GBtewhAcaRvAr4O7Gdt0BDYDjpeDGQi

action=upgrade&confirmed=true&username=wiener
```

ونقوم بتمريره لنجد أنه تم رفع صلاحيات الحساب إلى مدير وننتهي من حل اللاب.

وهناك حل أخر وهو الحل الرسمى للـ `PortSwigger` وهو أن ندخل إلى حساب `administrator:admin` ونقوم برفع صلاحيات `carlos` و اعتراض الـ `request` الخاص بالخطوة التأكيدية بعد الضغط على `Yes`
```http
POST /admin-roles HTTP/2
Host: 0a16001904d4c6f583a2f16f000d00b7.web-security-academy.net
Cookie: session=UYzw0eHC5VM4g2d88725sOhx0JmhhZ9L

action=upgrade&confirmed=true&username=carlos

```

وتوجيهه إلى الـ `repeater` وبعد ذلك نقوم بعمل تسجيل خروج وندخل إلى حساب `wiener:peter` ونقوم بنسخ الـ `Session`الخاصة بـ `wiener` ونضعها بدلًا من الـ `Session` الموجودة (الخاصة بحساب `administrator`) في الـ `request` الموجود مُسبقًا في الـ `repeater` ونغير اسم `carlos` إلى `wiener` في الـ `request` ونقوم بتمريره لنجد أنه تم رفع صلاحيات الحساب إلى مدير لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`بالإضافة إلى جزء كبير من ترقيع لاب`** [Unprotected admin functionality](./Unprotected-admin-functionality.md)

فيجب تطبيق ضوابط الوصول على كل الخطوات، فيجب على السيرفر أن يفحص الصلاحيات في كل خطوة وأن يقرأ الـ `Session` ويتأكد أن صاحبها هو `Administrator` بالفعل وليس مُستخدم عادي.

وبما أن العملية مُكونة من عدة خطوات، فيجب على السيرفر ألا يثق في أى `request` من أى مكان عشوائي يكون فيه `confirmed=true`، بل يجب عليه أن يتأكد أن المُستخدم قد مر بالخطوات في ترتيبها الصحيح` (Step 1 ثم Step 2`) عن طريق تخزين حالة مؤقتة في الـ `Session` على السيرفر وليس على المتصفح تُثبت ما إذا كان المُستخدم قد مر بالخطوات السابقة، وتقوم برفض الـ `request` فورًا بأمر `403 Forbidden` أو أمر إعادة التوجيه `302 Redirect` إذا لم يكن المُستخدم قد مر بالخطوات السابقة.
