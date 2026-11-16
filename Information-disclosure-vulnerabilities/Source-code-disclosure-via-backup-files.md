# [Source code disclosure via backup files](https://portswigger.net/web-security/information-disclosure/exploiting/lab-infoleak-via-backup-files)

## **`الهدف`**
قم بالحصول على كلمة مرور الـ `Database` المُضمنة في `Source Code` والذي يمكنك الوصول إليه عن طريق ملفات الـ `Backup` في مجلد مخفي، وقدمها كـ `Solution`. 
## **`خطوات الحل`**
سنقوم بالتوجه إلى مسار `/robots.txt`
```http
GET /robots.txt HTTP/2
Host: 0aa9002c033b14ab80395dad00bd00ab.web-security-academy.net
Cookie: session=oQEMo2Lin3AITfgWtYs38KEKujIYr1Ld
```

وسنجد في الاستجابة مسار ملفات الـ `Backup`
```http
HTTP/2 200 OK
Content-Type: text/plain; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 32

User-agent: *
Disallow: /backup
```

وعند التوجه إلى مسار `/backup` سنجد في الاستجابة مسار أخر `/backup/ProductTemplate.java.bak`
```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 435

<html>
    <head>
        <title>Index of /backup</title>
        <style>
            table { margin: 1em; }
            td { padding: 0.2em; }
        </style>
    </head>
    <body>
        <h1>Index of /backup</h1>
        <table>
            <tr><th>Name</th><th>Size</th></tr>
            <tr><td><a href='/backup/ProductTemplate.java.bak'>ProductTemplate.java.bak</a></td><td>1647B</td></tr>
        </table>
    </body>
</html>
```
وعند التوجه إلى مسار `/backup/ProductTemplate.java.bak` سنجد كلمة مرور قاعدة البيانات
```http
HTTP/2 200 OK
Content-Type: text/plain; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 1667

package data.productcatalog;

import common.db.JdbcConnectionBuilder;

import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.Serializable;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class ProductTemplate implements Serializable
{
    static final long serialVersionUID = 1L;

    private final String id;
    private transient Product product;

    public ProductTemplate(String id)
    {
        this.id = id;
    }

    private void readObject(ObjectInputStream inputStream) throws IOException, ClassNotFoundException
    {
        inputStream.defaultReadObject();

        ConnectionBuilder connectionBuilder = ConnectionBuilder.from(
                "org.postgresql.Driver",
                "postgresql",
                "localhost",
                5432,
                "postgres",
                "postgres",
                "n7pxxwowkjf3b4njxg2qgfbfglnxrnxx"
        ).withAutoCommit();
        try
        {
            Connection connect = connectionBuilder.connect(30);
            String sql = String.format("SELECT * FROM products WHERE id = '%s' LIMIT 1", id);
            Statement statement = connect.createStatement();
            ResultSet resultSet = statement.executeQuery(sql);
            if (!resultSet.next())
            {
                return;
            }
            product = Product.from(resultSet);
        }
        catch (SQLException e)
        {
            throw new IOException(e);
        }
    }

    public String getId()
    {
        return id;
    }

    public Product getProduct()
    {
        return product;
    }
}
```
ثم نقوم بنسخ كلمة المرور وهي `n7pxxwowkjf3b4njxg2qgfbfglnxrnxx` ونضغط على `Submit solution` ونضعه في الحقل المناسب ثم نضغط `OK` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب ألا يتم حفظ أي ملفات `Backup` أو `Source Code` أو ملفات تحكم بالإصدارت `Version Control` (مثل `.git`) في المجلدات التي يتم تقديمها للزوار، حيث يجب نقل هذه الملفات إلى مسارات آمنة خارج الـ `Web Root`، أو إزالتها تماماً من سيرفر الإنتاج، ويٌفضل إضافة قاعدة `Rule` في إعدادات سيرفر الويب تمنع بشكل قاطع تقديم أي ملفات تنتهي بهذه الامتدادات، حتى لو نسي المطور مسحها.

ويجب تعطيل خاصية استعراض المجلدات `Directory Listing`.

ويجب عدم استخدام `robots.txt` كأداة أمنية، حيث أن هذا الملف مخصص لعناكب محركات البحث النظيفة فقط، وإذا كان هناك مجلد حساس، يجب حمايته بآليات المصادقة والتصريح `Authentication & Authorization` وعدم ذكره في ملفات عامة.

ويجب ألا يتم كتابة أي كلمات مرور أو مفاتيح تشفير `API Keys` داخل الـ `Source Code` نهائياً.

وبما أن الـ `Source Code` في هذا اللاب يوجد به ثغرة `SQLi` فيجب أن يعتمد تطبيق الويب بشكل أساسي على الاستعلامات المجهزة `Prepared Statements` لفصل هيكل استعلام `SQL` عن البيانات المدخلة، بحيث لا يتم دمج مدخلات المستخدم مباشرة داخل الاستعلام، بل يتم إرسالها كبيانات منفصلة لكي تتعامل معها قاعدة البيانات كنص عادي وليس كأوامر برمجية قابلة للتنفيذ.
