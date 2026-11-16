# [Bypassing GraphQL brute force protections](https://portswigger.net/web-security/graphql/lab-graphql-brute-force-protection-bypass)

## **`الهدف`**
استخدام **[قائمة كلمات المرور](https://portswigger.net/web-security/authentication/auth-lab-passwords)** التي يوفرها الـ `PortSwigger` لعمل `brute force` لتسجيل الدخول لحساب `carlos`.
## **`خطوات الحل`**
بعد الدخول إلى اللاب نقوم بالذهاب إلى `HTTP history` وسنجد أحد الـ `requests` بـ `URl` إلى `/graphql/v1`.

لذلك نقوم بإرسال هذا الـ `request` إلى `repeater` ونضغط `Mouse left-click` داخل الـ `request` ونختار `Set introspection query` ونقوم بعمل إرسال للـ `request`، ومن ثم نقوم بالضغط على `Mouse left-click` داخل الـ `request` ونختار `GraphQL > Save GraphQL queries to site map`.

وبالتوجه إلى `Target > Site map` نقوم بفحص الاستعلامات الموجودة في مخطط `/graphql/v1` لنلاحظ وجود `Mutation` تقوم الدخول إلى الحساب.
```GraphQL
mutation($input: LoginInput) {
  login(input: $input) {
    token
    success
  }
}
```

لذلك سنقوم بنسخ كلمات المرور التى يوفرلها الـ `PortSwigger` في ملف ونسميه `PS-Passwords.txt` وبعد ذلك سنستخدم سكريبت بايثون بسيط يقوم بسحب كلمات المرور الـ `100` من ملف `PS-Passwords.txt` ويطبع `Mutation` الدخول `100` مرة كـ `alias` ويحفظهم في ملف نصي جديد لكي لا نحتاج إلى كتابته `100` مرة يدويًا.
```python
# 1. قراءة الكلمات من ملف PS-Passwords.txt
words = []
with open("PS-Passwords.txt", "r", encoding="utf-8") as file:
    for line in file:
        clean_line = line.strip()
        if clean_line:  # لتجاهل الأسطر الفارغة
            words.append(clean_line)

# 2. توليد النص واستبدال  word بالكلمات القادمة من الملف
output = "mutation {\n"
for i, word in enumerate(words, start=1):
    output += f'    login{i}: login(input: {{ username: "carlos", password: "{word}" }}) {{ token success }}\n'
output += "}"

# 3. حفظ الناتج في ملف result.txt
with open("result.txt", "w", encoding="utf-8") as file:
    file.write(output)

print("File result.txt created successfully!")
```

لذلك سنقوم بنسخ السكريبت داخل ملف نصي ونحفظه بصيغة `.py` نضعه فى نفس المسار الموجد به ملف `PS-Passwords.txt` ونقوم بتشغيله داخل الـ `Terminal` في `Linux` أو الـ `CMD/PowerShell` في `Windows` باختلاف أوامر التشغيل بينهم،

لينتج لدينا ملف `result.txt` به `100 alias` فنقوم بنسخها ونتوجه إلى الـ `requst` الموجود في الـ `repeater` ونضع به الـ `100` في خانة `GraphQl` ونقوم بإرساله.

==**`ملاحظة هامة:`**== الـ `PortSwigger` يوفر كود `JS` في جزئية `Tip` داخل صفحة اللاب لتوليد الـ `100 alias` بشكل تلقائي وطريقة أكثر سهولة.

بعد ذلك سنقوم باستعمال خاصية البحث في الاستجابة لنجد محاولة الدخول التى بها `success: true` فنقوم بمعرفة رقمها، كمثال `login4` 

ونذهب إلى الملف النصي الذي وضعنا به كلمات المرور `PS-Passwords.txt` وننسخ كلمة المرور رقم أربعة ونقوم بالدخول إلى حساب `carlos` بها لننتهي من حل اللاب.

وهناك طريقة أخر للحل حيث سنلاحظ أن الـ `token` الذي يتم طباعته في جميع حالات تسجيل الدخول هو نفسه يتكرر داخل الـ `response` الواحد فإذا كانت جميع المحاولات خاطئة فسنجد الـ `token` مكرر `100` مرة ولكن لا قيمة له في هذه الحالة.

لكن لو كانت إحدي المحاولات صحيحة فسنجد أن هذا الـ `Token` هو الخاص بجلسة حساب `carlos` الحالية لذلك يمكننا أن نقوم بنسخه ونتوجه إلى المتصفح ونضغط `F12` ثم  `Application` ونضعه في خانة الـ `Session` داخل الـ `Cookies` ثم نضغط `Enter`، بعد ذلك نقوم بالضغط على زر `Home` في اللاب ثم زر `My account` لنجد أننا داخل حساب `carlos` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب إيقاف خاصية الفحص الذاتي تمامًا في بيئة الإنتاج، وعلى الرغم من أن إغلاقها لن يمنع المهاجم من تخمين الحقول، ولكنه سيجعل الأمر أكثر صعوبة.

ويجب ضبط السيرفر ليمنع أي طلب يحتوي على عدد مبالغ فيه من الـ `aliases`.

ويجب تقييد الطلبات على مستوى التطبيق فبدلًا من حساب عدد طلبات الـ `HTTP` الواردة، يجب أن يقوم النظام بعد مرات استدعاء الدالة المسؤولة عن تسجيل الدخول `Login Resolver` نفسها، بحيث إذا قام نفس الـ `IP` أو المستخدم بتنفيذ عملية الدخول أكثر من الحد المسموح (مثلاً 5 مرات في الدقيقة)، يتم حظر العملية فوراً، حتى وإن كانت جميع المحاولات مجمعة داخل طلب `GraphQL` واحد.
