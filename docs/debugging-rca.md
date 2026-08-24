# Debugging, RCA & Engineering Depth — SDET Interview Prep

> **Format:** samjhaane ke liye Hinglish, aur jo aap interviewer ke saamne **bolenge**
> wo English mein — `> **Interview answer:**` blockquotes mein.
>
> Ye doc cover karta hai: **Debugging + RCA**, **Web/System fundamentals**,
> **Security testing**, **Performance testing**, aur **Observability**.
>
> Interviewer ke liye ye sabse revealing area hai — kyunki yahan pata chalta hai ki
> aap sirf test likhte ho ya **problem solve** karte ho.

---

## Contents

| # | Topic |
|---|---|
| 1 | Debugging mindset — RCA ka poora flow |
| 2 | Chrome DevTools — QA ka primary weapon |
| 3 | Logs aur stack traces padhna |
| 4 | Web fundamentals — request lifecycle |
| 5 | HTTP, cookies, sessions, CORS |
| 6 | Infrastructure — DNS, load balancer, CDN, cache |
| 7 | Security testing |
| 8 | Performance testing |
| 9 | Observability |
| 10 | Senior scenario questions |
| 11 | Quick revision |

---

# 1. Debugging mindset — RCA ka poora flow

## 1.1 Debugging aur "bug dhoondhna" alag cheez hai

Junior QA: *"Ye kaam nahi kar raha"* → dev ko de diya.
Senior QA: *"Ye kaam nahi kar raha **kyunki** X, aur uska proof ye raha — `file:line`"*

Farak sirf effort ka nahi hai. Dev ko jo cheez chahiye wo hai **reproduction + narrowing**.
Aap jitna narrow karke doge, fix utna jaldi hoga.

## 1.2 RCA ka layered flow — ye yaad rakho

```
BUG dikha
  ↓
1. REPRODUCE          — kya ye consistently hota hai? kis condition mein?
  ↓
2. BROWSER CONSOLE    — koi JS error? warning?
  ↓
3. NETWORK TAB        — request gayi? kya bheja? kya aaya? status?
  ↓
4. API DIRECT         — UI hataao, seedha API call karo. Wahan bhi hota hai?
  ↓
5. BACKEND LOGS       — us request ka trace, exception, stack
  ↓
6. DATABASE           — actual data kya hai? UI jo dikha raha hai wo sach hai?
  ↓
7. ROOT CAUSE         — file:line tak pahuncho
  ↓
8. FIX VERIFICATION   — fix ke baad wahi steps dobara
  ↓
9. REGRESSION         — is fix se aur kya toot sakta tha? wo bhi test karo
```

**Har layer ek sawaal ka jawab deti hai:** "problem is layer tak pahunchi bhi thi ya nahi?"

Agar network tab mein request hi nahi gayi → problem **frontend** mein hai.
Agar request gayi, 200 aaya, par UI galat → problem **rendering ya DTO mapping** mein hai.
Agar API ne hi galat diya → problem **backend** mein hai.
Agar backend theek hai par DB mein data galat → problem **write path** mein hai.

> **Interview answer:**
> "I work down the stack in layers, and each layer answers one question: did the problem
> reach this layer or not. If the network tab shows no request, it's a frontend problem and
> I stop there. If the request went out and came back 200 but the screen is wrong, it's
> rendering or DTO mapping. If the API itself returned wrong data, I go to the backend logs
> and then the database. The point of the sequence is that each step eliminates half the
> stack, so by the time I file the bug the developer has a narrowed location rather than a
> screenshot."

## 1.3 Reproduce nahi ho raha — kya karein

Ye interview mein poochha jaata hai aur zyadatar log kamzor jawab dete hain.

**Variables jo alag ho sakte hain:**

| Variable | Kaise check karein |
|---|---|
| User / role / permissions | Same user se try karo |
| Data state | Us specific record pe try karo, naye pe nahi |
| Browser / version | Same browser, same version |
| Screen size | Responsive breakpoint pe bug ho sakta hai |
| Timing / race | Slow network throttle karke try karo |
| Cache / stale session | Hard reload, incognito |
| Feature flag | Us user pe flag on hai? |
| Timezone / locale | Date bugs yahan chhupte hain |
| Concurrency | Do tab kholke saath mein karo |
| Environment | Staging pe nahi, prod pe ho sakta hai |

**Aur sabse important — evidence collect karo:**
- Session replay (PostHog) — user ne actually kya kiya
- Error tracking (Sentry/Rollbar) — kitni baar hua, kaunse users pe
- Server logs us timestamp ke aas-paas
- Correlation ID agar hai

> **Interview answer:**
> "First I stop assuming it's not real — 'not reproducible' usually means 'I haven't found the
> variable yet'. I go back to the reporter for the exact user, record, timestamp and browser,
> because those four resolve most cases. Then I work through the variables that differ between
> their run and mine: data state, permissions, feature flags, timezone, screen size, and
> concurrency. If I still can't reproduce it, I don't close it — I add instrumentation.
> Session replay and error tracking usually show me what I couldn't reproduce by hand, and an
> error-rate count tells me whether it's one user or five hundred."

> **[REAL]** Aapke Project Sales verification mein ek "page khaali hai" jaisa case tha — page
> actually theek tha, sirf wait 9 second kam tha. Lesson: **"kaam nahi kar raha" ko verify karo,
> maan mat lo.** Pehle DOM dump kiya, buttons list kiye, network calls dekhe — tab pata chala.

## 1.4 Narrowing techniques

**Binary search on the flow** — 12-step flow mein bug hai? Step 6 pe check karo. Theek hai to
7-12 mein hai, warna 1-5 mein. 12 steps → 4 checks.

**Binary search on commits** — `git bisect`. Kaunsa commit toota, 100 commits mein 7 checks.

**Isolate the layer** — UI hata ke API se karo. API theek hai → UI ka bug.

**Minimize the repro** — 15-step repro ko 3-step banao. Aksar minimize karte-karte hi root
cause mil jaata hai.

**Differential debugging** — jo case kaam karta hai aur jo nahi, unke beech ka **exact farak**
dhoondho. Ek supplier pe kaam karta hai, doosre pe nahi? Un dono records ko compare karo.

---

# 2. Chrome DevTools — QA ka primary weapon

Ye poochha jaata hai: *"DevTools mein kya-kya use karte ho?"*

## 2.1 Network tab — sabse important

| Kya dekhein | Kyun |
|---|---|
| **Status** | 200/400/401/403/500 — layer identify karta hai |
| **Request payload** | Frontend ne actually kya bheja? Aksar bug yahin hota hai |
| **Response body** | Backend ne kya diya? UI se match karta hai? |
| **Request headers** | Auth token gaya? Content-Type sahi hai? |
| **Response headers** | Cache-Control, Set-Cookie, CORS headers |
| **Timing breakdown** | Queueing / DNS / TCP / TTFB / Download — kahan slow hai |
| **Initiator** | Ye request kis code se gayi |
| **Size** | Payload bada to nahi |

**Useful filters:**
```
Filter: XHR/Fetch        — sirf API calls, assets nahi
status-code:500          — sirf errors
larger-than:1M           — bade payloads
-.png -.jpg              — images hataao
```

**Throttling** — "Slow 3G" select karke loading states aur race conditions test karo.
Bahut saare timing bugs sirf yahan dikhte hain.

**Preserve log** — navigation ke baad bhi requests dikhengi. Redirect flows ke liye zaroori.

## 2.2 Console tab

```javascript
// Errors — red. Warnings — yellow.
// Filter se sirf errors dekho

// Useful console commands QA ke liye:
document.querySelectorAll("button").length          // locator verify karo
document.querySelector("#email").readOnly            // readOnly hai?
document.querySelector("#btn").disabled              // disabled hai?
getEventListeners(document.querySelector("#btn"))    // handler laga hai? (Chrome only)
localStorage                                          // storage dekho
document.cookie                                       // cookies
performance.getEntriesByType("navigation")[0]         // page timing
```

> **[REAL]** Aapke date-picker bug mein exactly yahi kiya tha — live page pe JS chalake
> pata kiya ki input `readOnly: true` hai aur calendar grid mein **11 duplicate day numbers**
> hain. Wo insight sirf DOM inspect karke mila, guess se nahi.

## 2.3 Application tab

- **Cookies** — value, domain, path, Expires, HttpOnly, Secure, SameSite
- **Local/Session Storage** — token yahan hai? kya-kya store ho raha hai?
- **Service Workers** — stale cache ka common culprit
- **Clear storage** — clean state se test karo

**Security angle:** agar auth token `localStorage` mein hai to wo **XSS se churaya ja sakta
hai**. `HttpOnly` cookie safer hai. Ye batana interview mein strong hai.

## 2.4 Sources aur Performance

- **Sources** — breakpoints laga sakte ho, `debugger` statement, call stack dekho
- **Performance** — page load profile, long tasks, layout thrashing
- **Lighthouse** — performance, accessibility, SEO, best practices ka automated audit

> **Interview answer:**
> "The network tab is where I spend most of my time — the request payload in particular,
> because a surprising number of 'backend bugs' turn out to be the frontend sending the wrong
> thing. I also use throttling deliberately: a lot of race conditions only appear on a slow
> connection, and if I can reproduce one on Slow 3G I've turned an intermittent report into a
> deterministic one. On the console side I use it to verify locator assumptions before I write
> them into a test — checking whether an element is actually readOnly or disabled takes two
> seconds and saves an hour of chasing a flaky test."

---

# 3. Logs aur stack traces padhna

## 3.1 Stack trace kaise padhein

```
com.ether.exception.BusinessLogicException: Cannot accept estimate
    at com.ether.sales.service.SalesEstimateService.approveEstimate(SalesEstimateService.kt:626)
    at com.ether.sales.service.OpenSalesEstimateService.processEstimateResponse(...:181)
    at com.ether.sales.controller.OpenSalesEstimateController.processEstimateResponse(...:71)
    ...
Caused by: org.springframework.dao.IncorrectResultSizeDataAccessException:
    Query { "org": ..., "emailAddress": "x@y.com" } returned non unique result.
    at org.springframework.data.mongodb.core.MongoTemplate.findOne(...)
```

**Padhne ka tareeka:**

1. **Sabse upar** — kya exception hua aur kya message
2. **Pehli line jo aapke code ki hai** — `SalesEstimateService.kt:626` ← yahan dekho
3. **"Caused by" sabse neeche** — **ye asli wajah hai**. Log aksar ye miss kar dete hain.
4. Framework ki lines (spring, jackson) skip karo — wo aam taur pe wajah nahi hoti

> **[REAL]** Ye actual trace aapke Project Sales bug ka hai. "Caused by" line ne bataya ki
> `findByOrgAndEmailAddress` `Optional` expect karta hai par duplicate contacts hain —
> yaani `(org, emailAddress)` pe koi unique constraint nahi hai. **Root cause ek line mein.**

## 3.2 Log levels

| Level | Kab | QA kya dekhe |
|---|---|---|
| ERROR | Kuch fail hua, action chahiye | Sabse pehle ye |
| WARN | Kuch galat hai par chal raha hai | Aksar bug ka early sign |
| INFO | Normal flow milestones | Trace follow karne ke liye |
| DEBUG | Detailed | Debugging ke waqt on karo |
| TRACE | Bahut detailed | Rarely |

**Structured logging** — modern apps JSON logs likhte hain:
```json
{"timestamp":"2026-08-23T10:15:00Z","level":"ERROR","logger":"PaymentService",
 "message":"Charge failed","orderId":"PO-001","correlationId":"abc-123","userId":"u-45"}
```
Isse aap **filter kar sakte ho** — `correlationId=abc-123` daalke ek request ka poora journey.

## 3.3 Correlation ID — ye concept zaroor jaanein

Ek request kai services se guzarti hai. Har service apne logs likhti hai. **Kaise pata chale
ki kaunsa log kis request ka hai?**

Jawab: request ke saath ek unique ID chalti hai — `X-Correlation-ID` ya `X-Request-ID` header.
Har service usko apne har log line mein likhti hai.

**QA ke liye ye gold hai:**
```python
# Test se ek ID bhejo
correlation_id = f"test-{uuid.uuid4().hex[:8]}"
api.post("/api/v1/orders", json=payload, headers={"X-Correlation-ID": correlation_id})

# Fail hone pe log mein wahi ID search karo — poora backend journey mil jayega
```

> **Interview answer:**
> "I send a correlation ID header from my tests, generated per test run. When something fails,
> I search the backend logs for that ID and get the complete server-side journey of that exact
> request across services — instead of guessing which of a thousand log lines was mine.
> It costs one header and it turns log searching from archaeology into a lookup."

---

# 4. Web fundamentals — request lifecycle

Ye "senior-level thinking" wala section hai. Interviewer dekhta hai ki aap system samajhte ho
ya sirf UI.

## 4.1 URL type karne se page dikhne tak — poora flow

```
1. URL parse                  https://app.merlinai.co/orders
                                ↓
2. DNS LOOKUP                 app.merlinai.co → 52.14.x.x
   browser cache → OS cache → router → ISP resolver → root → TLD → authoritative
                                ↓
3. TCP HANDSHAKE              SYN → SYN-ACK → ACK  (3-way)
                                ↓
4. TLS HANDSHAKE (https)      cert verify, key exchange, cipher agree
                                ↓
5. HTTP REQUEST               GET /orders HTTP/1.1
                              Host, Cookie, Authorization, Accept...
                                ↓
6. [CDN?]                     static assets edge se aa sakte hain
                                ↓
7. [LOAD BALANCER]            kis server pe bhejein
                                ↓
8. SERVER                     app → business logic → DB query → response
                                ↓
9. HTTP RESPONSE              200, headers, HTML body
                                ↓
10. BROWSER RENDER            HTML parse → DOM
                              CSS parse → CSSOM
                              DOM + CSSOM → Render tree
                              Layout (reflow) → Paint → Composite
                                ↓
11. JS EXECUTE                bundles download, parse, execute
                              SPA: API calls → data → re-render
                                ↓
12. INTERACTIVE               user click kar sakta hai
```

**QA ke liye har step pe kya toot sakta hai:**

| Step | Possible failure | Kaise pakdein |
|---|---|---|
| DNS | Galat record, propagation delay | `nslookup`, `dig` |
| TLS | Expired cert, wrong domain | Browser warning, `openssl s_client` |
| CDN | Stale cache | Hard reload, cache headers dekho |
| Load balancer | Ek server unhealthy | Baar-baar refresh — kabhi kaam kabhi nahi |
| Server | 5xx, timeout | Logs, status code |
| Render | Layout shift, blank screen | Console errors, Lighthouse |
| JS | Bundle fail, race | Network tab, console |

> **[REAL]** Aapke supplier portal tests mein `wait_until="commit"` use hota hai, comment ke
> saath: *"app's heavy JS bundles block domcontentloaded for 60s+"*. Ye step 11 ka direct
> impact hai — JS bundle itna bada hai ki normal load event bahut der se aata hai.

## 4.2 Client-server architecture

```
CLIENT (browser)                    SERVER
  │                                   │
  │  ── HTTP request ──────────────→  │
  │                                   │  business logic
  │                                   │  DB query
  │  ←────────────── HTTP response ── │
  │                                   │
  stateless: har request independent
  server ko yaad nahi ki aap kaun ho
  → isliye cookie/token har request mein bhejna padta hai
```

**Monolith vs Microservices:**

| | Monolith | Microservices |
|---|---|---|
| Deploy | Ek unit | Har service alag |
| Failure | Sab down | Ek service down, baaki chal sakti hain |
| Testing | E2E aasan | E2E mushkil — contract testing chahiye |
| Debugging | Ek log file | Distributed tracing chahiye |

---

# 5. HTTP, cookies, sessions, CORS

## 5.1 HTTP versions

| Version | Kya naya | QA impact |
|---|---|---|
| HTTP/1.1 | Persistent connections | Head-of-line blocking, browser 6 connections/domain — isliye "domain sharding" hack |
| HTTP/2 | Multiplexing, header compression, server push | Ek connection pe kai requests — sharding ab anti-pattern |
| HTTP/3 | QUIC over UDP | Packet loss pe behtar — mobile networks |

## 5.2 Cookies — poora

```
Set-Cookie: session=abc123; Domain=.merlinai.co; Path=/; Expires=...;
            HttpOnly; Secure; SameSite=Lax
```

| Attribute | Matlab | Security impact |
|---|---|---|
| `Domain` | Kaunse domain pe bheji jaye | Bahut broad = leak risk |
| `Path` | Kaunse path pe | |
| `Expires` / `Max-Age` | Kab tak | Missing = session cookie, tab band = gayab |
| **`HttpOnly`** | JS access nahi kar sakta | **XSS se token chori nahi hoga** |
| **`Secure`** | Sirf HTTPS pe | HTTP pe plaintext nahi jayegi |
| **`SameSite`** | Cross-site request pe bheji jaye? | **CSRF protection** |

**SameSite values:**
- `Strict` — cross-site pe kabhi nahi. Sabse safe, par external link se aane pe logout dikhta hai.
- `Lax` — top-level GET navigation pe jaayegi. Default aajkal.
- `None` — hamesha jayegi, par `Secure` mandatory.

**QA test cases:**
- Logout ke baad cookie invalidate hui? (sirf browser se delete karna kaafi nahi — server side bhi)
- `HttpOnly` set hai auth cookie pe?
- `Secure` set hai?
- Expiry sensible hai?

## 5.3 Session vs Token auth

```
SESSION (stateful)                    TOKEN / JWT (stateless)
─────────────────                     ──────────────────────
login → server session banata hai     login → server signed token deta hai
session ID cookie mein                token client store karta hai
server memory/redis mein state        server kuch store nahi karta
                                      
logout = server se delete             logout = ??? token valid rehta hai
                                      → isliye short expiry + refresh token
                                      → ya blocklist rakhni padti hai
                                      
scale = shared session store          scale = koi shared state nahi
```

**JWT anatomy:**
```
eyJhbGciOiJIUzI1NiJ9  .  eyJzdWIiOiJyaXRpayJ9  .  SflKxwRJSMeKKF2QT4f
      HEADER                    PAYLOAD                  SIGNATURE
   {"alg":"HS256"}       {"sub":"ritik","exp":...}   HMAC(header.payload, secret)
```

**Ye samajhna zaroori hai:** payload **base64 hai, encrypted nahi**. Koi bhi padh sakta hai.
Isliye usme kabhi password ya sensitive data mat daalo. Signature guarantee karta hai ki
**tamper nahi hua** — par confidentiality nahi deta.

**QA test cases (aapke project se relevant):**
- Payload badalke bhejo (e.g. `role: admin`) → signature fail hona chahiye → 401
- Expired token → 401
- Doosre user ka valid token → 403/404
- `alg: none` attack → reject hona chahiye
- Token replay (single-use hona chahiye tha?) → reject

> **[REAL]** `ProjectSaleConfirmIntegrationTest` mein exactly ye saare cases hain — forged
> signature, wrong sale binding, email mismatch, single-use replay, expiry, cross-org.
> Ye interview mein batana strong hai: *"our acceptance-token tests cover forged signature,
> replay, expiry, cross-binding and cross-org, because that endpoint is publicly reachable."*

## 5.4 CORS — ye zaroor samjho

**Problem:** browser security policy kehti hai ki `app.merlinai.co` ka JavaScript
`api.merlinai.co` ko call nahi kar sakta — **different origin**.

**Origin = scheme + host + port.** Teenon match karne chahiye.

```
https://app.merlinai.co        →  https://api.merlinai.co     ✗ different host
https://app.merlinai.co        →  http://app.merlinai.co      ✗ different scheme
https://app.merlinai.co:443    →  https://app.merlinai.co:8080 ✗ different port
```

**Solution:** server batata hai ki kaunse origins allowed hain.

```
Access-Control-Allow-Origin: https://app.merlinai.co
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Authorization, Content-Type
Access-Control-Allow-Credentials: true
```

**Preflight** — "simple" requests ke alawa browser pehle ek `OPTIONS` request bhejta hai
poochhne ke liye. Ye tab hota hai jab:
- Method `PUT`/`DELETE`/`PATCH` ho
- Custom headers hoN (jaise `Authorization`)
- Content-Type `application/json` ho

```
Browser                              Server
   │  OPTIONS /api/v1/orders  ──────→ │   (preflight)
   │  Origin: https://app...          │
   │  Access-Control-Request-Method: POST
   │                                  │
   │  ←──── 204 + Allow-* headers ─── │
   │                                  │
   │  POST /api/v1/orders  ─────────→ │   (actual)
```

**QA ke liye:**
- CORS error hamesha **browser** mein aata hai, Postman/curl mein nahi — kyunki ye browser
  ki policy hai, server ki nahi. **Ye interview mein poochha jaata hai.**
- `Allow-Origin: *` + `Allow-Credentials: true` — ye combination browser reject karta hai,
  aur ye ek security smell hai
- Naya header add karne pe preflight allow-list update karni padti hai — common bug

> **Interview answer:**
> "CORS is a browser-enforced policy, not a server one — which is why a request that fails in
> the browser will succeed from Postman or curl. That distinction matters when triaging,
> because 'it works in Postman' doesn't mean the endpoint is fine. What I test is the preflight
> response: whether the methods and headers the frontend actually uses are in the allow-list,
> because adding one new header on the client side silently breaks the call until the server
> allow-list is updated. And I check that a wildcard origin isn't combined with credentials —
> browsers reject that combination and it usually indicates a misconfiguration."

---

# 6. Infrastructure — DNS, load balancer, CDN, cache

## 6.1 DNS

```
app.merlinai.co → ?
  1. Browser cache
  2. OS cache        (hosts file yahan override kar sakte ho — testing ke liye useful)
  3. Router cache
  4. ISP resolver
  5. Root → TDL (.co) → authoritative nameserver
  → 52.14.x.x
```

**QA relevance:** DNS propagation mein waqt lagta hai (TTL). Naya environment banne ke baad
kuch der tak purana IP resolve ho sakta hai. `nslookup` / `dig` se verify karo.

**Testing trick:** `/etc/hosts` mein entry daalke traffic kisi aur IP pe bhej sakte ho —
staging backend ke saath prod frontend test karna ho to kaam aata hai.

## 6.2 Load balancer

```
                    ┌─ Server 1
Client → LB ────────┼─ Server 2
                    └─ Server 3
```

**Algorithms:** round-robin, least-connections, IP-hash (sticky).

**QA ke liye kya matter karta hai:**

| Issue | Symptom | Test |
|---|---|---|
| **Session affinity** | Kabhi logged in, kabhi logout | Baar-baar refresh karo |
| Ek server unhealthy | Intermittent 500 | Health check verify karo |
| Mixed versions during deploy | Kabhi naya feature dikhta, kabhi nahi | Rolling deploy ke dauraan test |

> Ye "kabhi kaam karta hai kabhi nahi" wale bugs ka **classic** source hai — aur log ise
> "flaky test" samajh lete hain.

## 6.3 CDN aur caching

```
User (Bangalore) → CDN edge (Mumbai)  → cache HIT  → turant
                                      → cache MISS → origin server → cache karke bhejo
```

**Cache headers:**
```
Cache-Control: max-age=3600, public        # 1 ghanta cache karo
Cache-Control: no-store                    # kabhi cache mat karo (sensitive data)
Cache-Control: must-revalidate
ETag: "abc123"                             # content ka fingerprint
If-None-Match: "abc123"  →  304 Not Modified  (body nahi bhejta — bandwidth bachta hai)
```

**QA test cases:**
- Deploy ke baad purana JS bundle serve ho raha hai? (cache busting hash lagta hai?)
- **Sensitive data cache ho raha hai?** — `Cache-Control: no-store` hona chahiye. Ye ek
  real security bug hota hai: user logout kare, back button dabaye, aur pichhla data dikh jaye.
- 304 sahi kaam kar raha hai?

## 6.4 Reverse proxy

```
Internet → Reverse Proxy (nginx) → internal services
```
Kaam: SSL termination, routing, rate limiting, compression, caching, request/response headers.

**QA relevance:** kabhi-kabhi error proxy se aata hai, app se nahi. `502 Bad Gateway` matlab
proxy backend tak nahi pahuncha. `504 Gateway Timeout` matlab backend ne der lagayi.
Ye distinction debugging mein bachata hai.

## 6.5 WebSockets

```
HTTP:       request → response, connection band
WebSocket:  handshake (HTTP Upgrade) → connection khuli rehti hai → dono taraf messages
```

**Kab:** real-time updates, chat, live dashboards, notifications.

**QA test cases:**
- Connection drop hone pe reconnect hota hai?
- Reconnect ke baad missed messages aate hain?
- Multiple tabs — har tab apna connection?
- Message order guaranteed hai?
- Auth — connection pe token verify hota hai?

```python
# Playwright se websocket monitor karo
def test_websocket_messages(page):
    messages = []
    page.on("websocket", lambda ws: ws.on("framereceived",
            lambda payload: messages.append(payload)))
    page.goto("/dashboard")
    page.wait_for_timeout(5000)
    assert any("order_updated" in str(m) for m in messages)
```

---

# 7. Security testing

Deep security engineer banna zaroori nahi — par **basic vulnerabilities pehchanna** aana
chahiye. Ye ab senior QA se expect kiya jaata hai.

## 7.1 Authentication vs Authorization

| | Authentication | Authorization |
|---|---|---|
| Sawaal | **Tum kaun ho?** | **Tumhe kya karne ki ijaazat hai?** |
| Fail hone pe | 401 | 403 |
| Example | Login | Viewer approve nahi kar sakta |

**RBAC (Role-Based Access Control):** user → role → permissions.

**QA ka test matrix:** har role × har action = ek cell. Har cell test karo.

```python
@pytest.mark.parametrize("role,action,expected", [
    ("admin",   "approve", 200),
    ("manager", "approve", 200),
    ("viewer",  "approve", 403),      # ← ye sabse important
    ("viewer",  "read",    200),
])
def test_rbac(api_for_role, role, action, expected):
    r = api_for_role(role).post(f"/api/v1/orders/123/{action}")
    assert r.status_code == expected
```

**Critical point:** UI mein button chhupa dena **security nahi hai**. API level pe test karo.

## 7.2 OWASP Top 10 — QA lens se

| # | Vulnerability | QA kaise test kare |
|---|---|---|
| 1 | **Broken Access Control** | IDOR — doosre user ka ID daalke try karo. Cross-org. Role bypass. |
| 2 | Cryptographic Failures | HTTPS everywhere? Password hashed? Sensitive data cache/logs mein? |
| 3 | **Injection** | SQL/NoSQL/command injection payloads |
| 4 | Insecure Design | Rate limiting hai? Business logic bypass ho sakta hai? |
| 5 | Security Misconfiguration | Default creds, debug mode on, verbose errors, directory listing |
| 6 | Vulnerable Components | `pip-audit`, `npm audit`, Dependabot |
| 7 | Auth Failures | Weak password allowed? Brute force protection? Session fixation? |
| 8 | Data Integrity Failures | Unsigned updates, insecure deserialization |
| 9 | **Logging Failures** | Security events log hote hain? PII log mein to nahi? |
| 10 | SSRF | Server ko internal URL fetch karwana |

## 7.3 IDOR — sabse common aur sabse easy to test

**Insecure Direct Object Reference** — aap apna ID badalke doosre ka data dekh sakte ho.

```python
def test_cannot_access_other_users_order(user_a_api, user_b_order_id):
    r = user_a_api.get(f"/api/v1/orders/{user_b_order_id}")
    assert r.status_code in (403, 404)
    # 404 aksar behtar hai — 403 bata deta hai ki resource EXIST karta hai


def test_cannot_access_other_org(org_a_api, org_b_order_id):
    """Multi-tenant apps mein ye sabse critical test hai."""
    r = org_a_api.get(f"/api/v1/orders/{org_b_order_id}")
    assert r.status_code in (403, 404)
```

> **[REAL]** Merlin ka backend multi-tenant hai — har query `orgId` se scoped honi chahiye.
> Unke apne rules mein likha hai: *"findAll() is a security violation in almost every context"*.
> Aur `ProjectSaleLifecycleIntegrationTest` mein ek test hai:
> *"a second org cannot read, close, or cancel this org's project sale"*. **Ye IDOR test hai.**

## 7.4 Injection

**SQL Injection:**
```
' OR '1'='1
'; DROP TABLE users; --
1 UNION SELECT username, password FROM users --
admin'--
```

**NoSQL Injection (MongoDB — aapke project pe relevant):**
```json
{"email": {"$ne": null}, "password": {"$ne": null}}    // login bypass
{"email": "admin@x.com", "password": {"$gt": ""}}
```

**Test karna:** in payloads ko har input field aur har API parameter mein daalo.
**Expected:** clean validation error, **na ki** DB error ya successful bypass.

**Red flag:** agar response mein SQL syntax error ya Mongo query dikh jaye — **ye do bugs
hain**: injection ki sambhavna, aur information disclosure.

> **[REAL]** Aapne ye exactly pakda tha — Project Sales ka accept endpoint 400 ke saath
> **raw Mongo query, org ObjectId samet** return kar raha tha, ek **public unauthenticated**
> endpoint pe. Interview mein ye batana bahut strong hai.

```python
def test_error_does_not_leak_internals():
    r = requests.post(f"{API}/api/v1/open/accept", json={"bad": "data"})
    body = r.text.lower()
    for leak in ["stack trace", "sql", "mongo", "org.$id", "$java",
                 "at com.", "exception", "/users/", "c:\\"]:
        assert leak not in body, f"Error response leaked: {leak}"
```

## 7.5 XSS aur CSRF

**XSS (Cross-Site Scripting)** — attacker ka JS aapke page pe chalta hai.

```html
<script>alert(1)</script>
<img src=x onerror=alert(document.cookie)>
"><svg onload=alert(1)>
javascript:alert(1)
```

**Types:** stored (DB mein save, har user ko), reflected (URL se), DOM-based (client-side).

**Test:** ye payloads har input mein daalo, phir jahan wo render hota hai wahan dekho.
**Expected:** escaped text dikhe, execute na ho.

**Defense:** output encoding, Content-Security-Policy header, `HttpOnly` cookies.

**CSRF (Cross-Site Request Forgery)** — attacker ki site se aapke logged-in session pe request.

**Defense:** CSRF token, `SameSite` cookie.
**Test:** CSRF token hata ke ya galat token se request bhejo → reject hona chahiye.

## 7.6 Sensitive data exposure — checklist

- [ ] Passwords hashed (bcrypt/argon2), plaintext nahi
- [ ] API responses mein `passwordHash`, internal IDs, debug fields nahi
- [ ] Logs mein PII, tokens, card numbers nahi
- [ ] Error messages mein stack trace nahi (production mein)
- [ ] `.git`, `.env`, backup files web se accessible nahi
- [ ] Source maps production mein nahi
- [ ] Credentials repo mein commit nahi

> **[REAL]** Aakhri point — aapke repo mein `application-test.properties` mein MongoDB Atlas
> credentials aur QuickBooks client secret **plaintext committed** hain. Ye batana interview
> mein strong hai: *"I found credentials committed in our test properties file and raised it —
> the remediation is moving them to the CI secret store and rotating them."*

> **Interview answer (security overall):**
> "I'm not a security engineer, but I treat a defined set of security checks as part of normal
> functional testing — mainly access control, because that's where QA can find real issues
> without specialist tooling. For every endpoint I ask three questions: does it work without a
> token, does it work with another user's token, and does it work with another tenant's token.
> In a multi-tenant product that third one is the highest-value test there is. I also check
> that error responses don't leak internals — I've found a public endpoint returning a raw
> database query including the org identifier, which is an information-disclosure issue on top
> of the functional bug."

---

# 8. Performance testing

## 8.1 Types — ye table yaad rakho

| Type | Kya karte hain | Kya pata chalta hai |
|---|---|---|
| **Load** | Expected traffic daalo | Normal load pe performance theek hai? |
| **Stress** | Badhate raho jab tak toot na jaye | Breaking point kahan hai |
| **Spike** | Achanak bahut saara traffic | Sudden surge handle karta hai? |
| **Endurance / Soak** | Normal load, 8-12 ghante | **Memory leak**, connection leak |
| **Volume** | Bahut saara DATA | Bade dataset pe queries slow to nahi |
| **Scalability** | Resources badhao | Linearly scale karta hai? |

## 8.2 Metrics — aur ek zaroori nuance

| Metric | Matlab |
|---|---|
| **Response time** | Ek request ka poora time |
| **Latency** | Network travel time (response time ka hissa) |
| **Throughput** | Requests per second |
| **Concurrent users** | Ek saath kitne active |
| **Error rate** | % requests jo fail huin |
| **p50 / p95 / p99** | Percentiles — **average se zyada important** |

**Average dhoka deta hai:**
```
100 requests:
  95 requests → 50ms
   5 requests → 3000ms

Average = (95×50 + 5×3000) / 100 = 197ms     ← "theek lag raha hai"
p95     = 3000ms                              ← 5% users ka experience KHARAB
```

> **Interview answer:**
> "I report p95 and p99, not the average, because the average hides the tail. If ninety-five
> percent of requests are fast and five percent take three seconds, the average looks
> acceptable while one in twenty users has a bad experience — and at scale that's thousands
> of people. p99 is what I'd hold a team to for a critical path."

## 8.3 JMeter — components

| Component | Kaam |
|---|---|
| **Thread Group** | Kitne users, ramp-up time, loop count |
| **Samplers** | Actual requests (HTTP Request) |
| **Listeners** | Results — *View Results Tree sirf debug ke liye, load mein OFF* |
| **Assertions** | Response validate karo — warna 500 bhi "pass" ho jayega |
| **Config Elements** | CSV Data Set (test data), HTTP Header Manager (auth) |
| **Timers** | Think time — real user simulate karo |
| **Pre/Post Processors** | Token extract karke agli request mein use karo |

```bash
# GUI mein test plan banao, chalao HAMESHA non-GUI mode mein
jmeter -n -t plan.jmx -l results.jtl -e -o report/

# -n non-GUI   -t plan   -l results   -e generate report   -o output dir
```

**Common galti:** GUI mode mein load test chalana. GUI khud resource khaata hai aur results
distort karta hai.

## 8.4 k6 — modern alternative (JS mein)

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '2m', target: 100 },   // ramp up
    { duration: '5m', target: 100 },   // stay
    { duration: '2m', target: 0 },     // ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],   // 95% under 500ms
    http_req_failed: ['rate<0.01'],     // <1% errors
  },
};

export default function () {
  const res = http.get('https://api.merlinai.co/api/v1/orders');
  check(res, { 'status 200': (r) => r.status === 200 });
  sleep(1);
}
```

**k6 ka fayda:** code hai, git mein rehta hai, CI mein aasan, thresholds built-in
(pass/fail automatic).

## 8.5 Bottleneck kaise dhoondhein

```
Slow response
  ↓
Kahan slow hai?
  ├─ Network        → CDN, payload size, compression
  ├─ Web server     → connection pool, worker count
  ├─ Application    → inefficient code, N+1 queries, sync blocking calls
  ├─ Database       → missing index, slow query, lock contention
  └─ External API   → third-party slow, no timeout
```

**QA ka sabse common find: N+1 queries.** Ek page 200 DB queries kar raha hai jab 2 kaafi
thi. Ye tab dikhta hai jab data badhta hai — 10 records pe theek, 1000 pe timeout.

> **[REAL]** Merlin ke backend rules mein `no-n-plus-one.md` ka poora file hai, aur ek real
> incident documented hai: *"116 seconds for 500 docs"* lazy DBRef proxies ki wajah se.
> Interview mein ye batana: *"one of the performance rules in our codebase came from an
> incident where lazy database references inside a loop turned one query into hundreds —
> 116 seconds for 500 documents."*

---

# 9. Observability

## 9.1 Teen pillars

```
LOGS                    METRICS                 TRACES
────                    ───────                 ──────
"kya hua"               "kitna/kitni baar"      "request ka poora journey"
discrete events         aggregated numbers      distributed spans
                        
ERROR: payment failed   error_rate: 2.3%        request abc-123:
order=PO-001            p95_latency: 340ms        gateway    12ms
                        requests_per_sec: 450     auth-svc    8ms
                                                  order-svc 210ms  ← yahan slow
                                                  db         45ms
```

**Kab kya use karein:**
- **Metrics** — "kuch galat hai" pata karne ke liye (alerting)
- **Traces** — "kahan galat hai" pata karne ke liye
- **Logs** — "kya galat hai" pata karne ke liye

## 9.2 Distributed tracing

```
Trace ID: abc-123  (poore request ka ek ID)
 ├─ Span: API Gateway          [12ms]
 │   └─ Span: Auth Service     [8ms]
 ├─ Span: Order Service        [210ms]  ← bottleneck yahan
 │   ├─ Span: DB query         [180ms]  ← actually yahan
 │   └─ Span: Cache lookup     [2ms]
 └─ Span: Notification Service [30ms]
```

Tools: OpenTelemetry (standard), Jaeger, Zipkin, Datadog.

## 9.3 SLI, SLO, SLA — ye poochha jaata hai

| Term | Matlab | Example |
|---|---|---|
| **SLI** | Service Level **Indicator** — measurement | "99.2% requests succeed" |
| **SLO** | Service Level **Objective** — internal target | "99.5% requests succeed" |
| **SLA** | Service Level **Agreement** — customer contract, penalty ke saath | "99% uptime ya refund" |

**Error budget:** agar SLO 99.9% hai, to 0.1% failure "allowed" hai. Wo aapka budget hai.
Budget khatam → naye features rok do, reliability pe kaam karo.

> **Interview answer:**
> "SLI is what you measure, SLO is the target you hold yourself to, and SLA is the contractual
> promise with financial consequences. The one that changes behaviour is the error budget that
> falls out of the SLO — if you've agreed 99.9%, you've also agreed that 0.1% of failures are
> acceptable, and when you've spent that budget the team stops shipping features and works on
> reliability. It turns 'quality' from an argument into a number."

## 9.4 QA ke liye observability ka asli use

**Production errors = aapki test gap list.**

```
Sentry mein error dikha
  ↓
Kya humare tests ne ye pakda?
  ├─ Haan → test theek hai, deploy process mein gap
  └─ Nahi → YE MERA AGLA TEST CASE HAI
```

> **Interview answer:**
> "I treat production error tracking as the most honest input to my test backlog. My tests
> verify what we thought of; production tells me what we didn't. Every error class that shows
> up in Sentry and wasn't caught by a test becomes a test — and more usefully, I ask what
> *category* of thing we missed, because usually it's not one bug, it's a blind spot. That's
> how a bug fix becomes a process fix instead of just another test case."

---

# 10. Senior scenario questions

## Q1. "Production mein payment fail ho raha hai but API 200 return kar rahi hai. Debug kaise karoge?"

**Ye headline question hai.** Structured answer:

> **Interview answer:**
> "A 200 with a failed outcome tells me the failure is semantic, not transport — so the first
> thing I check is what the 200 actually contains. Plenty of APIs return 200 with a body like
> `{"status": "failed"}`, and a client that only checks the status code will report success.
>
> Then I work the flow in order. Was the charge actually created at the payment gateway? Their
> dashboard is the source of truth for that, not our database. If the gateway shows a
> successful charge, the money moved and our side lost it — that's the worst case and it's
> urgent. If the gateway shows no charge, we never called it, and the question moves upstream.
>
> Next I look at whether this is a partial failure. The classic shape is: charge succeeded,
> order creation threw, the exception was swallowed, and the handler still returned 200. So I
> check for an orphaned charge with no corresponding order, and whether a reversal or refund
> was issued. If neither exists, we have money taken and nothing recorded.
>
> Then I check the database directly — the payment row, its status, the order row, and whether
> the ledger entries balance. And I check whether it's a retry problem: if the client retried
> a non-idempotent POST, we may have double-charged.
>
> Finally I'd look at scope — one user or all users, one payment method or all, started at a
> particular deploy. Error tracking and the deploy timeline usually answer that in a minute.
>
> The reason I'd escalate this one immediately rather than finish investigating first is that
> every minute it continues is more affected customers."

**Ye jawab strong kyun hai:** aap layer-by-layer chal rahe ho, partial failure ka concept
samajhte ho, aur urgency ka business sense dikha rahe ho.

## Q2. "Ek bug 100 mein se 1 baar hota hai. Kaise pakdoge?"

> **Interview answer:**
> "One-in-a-hundred usually means a race condition, a data-dependent path, or an
> infrastructure issue like one unhealthy server behind the load balancer. I'd approach it in
> that order.
>
> For a race, I try to make it deterministic by changing timing — throttling the network,
> adding latency to a dependency, or running the action concurrently from two sessions. If
> slowing things down makes it reproducible, it's a race and I now have a reliable test.
>
> For data dependency, I compare a failing case with a passing one and look for the exact
> difference — a null field, a different status, a boundary value.
>
> For infrastructure, the giveaway is that it correlates with nothing in the application. I'd
> check whether failures cluster on one instance, which the load balancer or trace data will
> show.
>
> And I'd add instrumentation rather than keep guessing — a correlation ID and structured
> logging around the suspect path costs very little and turns the next occurrence into
> evidence instead of another anecdote."

## Q3. "Ek page slow hai. Investigate karo."

> **Interview answer:**
> "I'd separate the three places time can go: network, server, and rendering.
>
> The network tab's timing breakdown tells me immediately which one it is. If time-to-first-byte
> is high, the server is slow and the frontend is innocent. If TTFB is fine but the page takes
> seconds to become interactive, it's rendering or JavaScript.
>
> If it's the server, I'd look at the slowest API call and then at the backend trace for that
> request. The most common cause I've seen is N+1 queries — code that looks like one operation
> but issues a database call per row, which is fine with ten records and fatal with a thousand.
>
> If it's the frontend, I'd check bundle size, whether rendering is blocked on a sequential
> chain of API calls that could run in parallel, and whether there's a large uncompressed
> asset. Lighthouse gives a fast first read on that.
>
> And I'd check whether it's slow for everyone or slow at scale — a page that's fast in staging
> with fifty records and slow in production with fifty thousand is a data-volume problem, and
> that's a test gap as much as a performance one."

## Q4. "Aapko lagta hai bug hai, dev ko nahi. Deadlock. Kya karoge?"

> **Interview answer:**
> "I'd move it from opinion to evidence. First I check what the requirement or acceptance
> criteria actually says — if it's explicit, there's nothing to argue about. If it's silent,
> then this isn't a disagreement about a bug, it's an ambiguity in the requirement, and the
> right person to resolve it is the product owner, not either of us.
>
> I'd also frame the impact rather than the defect: what a user experiences, how many are
> affected, and what the support cost looks like. 'The date shows wrong' is arguable; 'a
> supplier sees a delivery date one day earlier than agreed and could ship late' isn't.
>
> If it's genuinely a judgement call and product decides it's acceptable, I'd ask for that
> decision to be recorded on the ticket, because otherwise the same argument happens again in
> three months. I'm not trying to win — I'm trying to make sure the decision was made
> deliberately and by the right person."

## Q5. "Aapne kaise pata lagaya ki ek 'flaky' test actually product bug tha?"

Ye aapki **real story** hai — poori taiyari Part 8 mein hai (Playwright doc).

> **Interview answer (short version):**
> "We had a supplier acknowledgement step that failed intermittently and had been labelled
> flaky with a retry on it. I read the component source and found the input was read-only in
> that library version, so our `fill()` could never have worked — it waited thirty seconds for
> the field to become editable and then threw. It wasn't intermittent at all; it failed
> identically every run and only looked flaky because a broad try/except swallowed the
> exception and fell through to a fallback path.
>
> The fallback was picking a day from the calendar by matching text and taking the first match
> — and the calendar pads its grid with days from adjacent months sharing the same CSS class,
> so eleven of the forty-two numbers were duplicated. On most days the first match was a
> disabled past date. That's why the failure depended on today's date, and that's what made it
> look random."

---

# 11. Quick revision

| Sawaal | 10-second jawab |
|---|---|
| RCA flow | Reproduce → console → network → API direct → logs → DB → root cause → verify → regression |
| Reproduce nahi ho raha | Variables identify karo: user, data, browser, timing, flag, timezone, concurrency. Phir instrument karo. |
| Stack trace | Sabse upar exception, apne code ki pehli line, **"Caused by" sabse neeche = asli wajah** |
| Correlation ID | Test se header bhejo, fail hone pe logs mein wahi ID search karo |
| 401 vs 403 | Kaun ho pata nahi vs pata hai par allowed nahi |
| CORS | **Browser** enforce karta hai, server nahi — isliye Postman mein error nahi aata |
| Cookie flags | HttpOnly (XSS), Secure (HTTPS), SameSite (CSRF) |
| JWT payload | base64 hai, **encrypted nahi** — sensitive data mat daalo |
| IDOR test | Doosre user ka / doosre org ka ID daalke try karo |
| Injection red flag | Response mein SQL/Mongo error dikhna = do bugs |
| p95 vs average | Average tail chhupa deta hai — 5% users ka experience |
| Load vs stress vs soak | Expected / breaking point / memory leak |
| N+1 | Loop ke andar DB call — 10 records pe theek, 1000 pe timeout |
| 502 vs 504 | Proxy backend tak nahi pahuncha vs backend ne der lagayi |
| Teen pillars | Logs (kya) / Metrics (kitna) / Traces (kahan) |
| SLI/SLO/SLA | Measurement / internal target / customer contract |
| Error budget | SLO ka bacha hissa — khatam to features ruko, reliability pe kaam |
| Production errors | **Aapki test gap list** — jo wahan aaya aur test mein nahi, wo agla test case |

---

## Red flags — ye jawab mat dena

| Galat jawab | Kyun galat |
|---|---|
| "Reproduce nahi hua to close kar diya" | Bug real ho sakta hai — instrument karo |
| "Ye backend ka issue hai, mera kaam nahi" | Narrowing hi aapka kaam hai |
| "Average response time 200ms hai, theek hai" | Tail chhupa rahe ho — p95 dekho |
| "Security testing security team ka kaam hai" | Access control QA ka core kaam hai |
| "Console mein error tha par test pass ho gaya" | Console errors bhi failures hain |
| "Logs dev dekhte hain" | Log padhna senior QA ki basic skill hai |
| "Performance testing ke liye tool nahi hai" | curl aur ek loop se bhi shuruat ho sakti hai |

---

## Aakhri baat

Is doc ka har technical jawab tab strong hota hai jab uske saath **aapka apna example** ho.

**Ratta:**
> "CORS is a browser security policy."

**Experience:**
> "CORS is browser-enforced, which is why 'it works in Postman' doesn't mean the endpoint is
> fine — I've seen that exact confusion send a team looking at the backend when the actual fix
> was one missing entry in the preflight allow-list."

Doosra jawab dene wale ko job milti hai.
