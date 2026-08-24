# 05 — OOP, Framework Architecture & System Design (Senior SDET Interview Prep)

> **Kaise use karein:** Har concept ka structure fixed hai — **Kya hai (analogy pehle!) → Code → Automation mein kahan lagta hai → Interview answer (English) → Cross-question**.
> Explanation Hinglish mein hai taaki concept dimaag mein baithe. Lekin `> **Interview answer:**` wale blockquote — **wahi bolna hai, English mein**. Ratta nahi, structure yaad rakho.
>
> `> **[REAL]**` boxes = tumhare Merlin AI project ke actual examples. Interview mein sabse zyada weight inhi ka hai. OOP ki definition har candidate bolta hai; jo banda bolta hai "*maine apne framework mein ye isliye compose kiya, inherit nahi kiya*" — wahi senior lagta hai.
>
> **Ek baat pehle:** Senior SDET round mein OOP ka test "definition yaad hai kya" nahi hota. Test hota hai — **"kya tu design decision justify kar sakta hai?"** Isliye har section ke end mein *"kab NAHI use karna"* bhi diya hai. Wahi asli marks hain.

---

## Table of Contents

| # | Section |
|---|---|
| — | **PART 1 — OOP FROM ZERO** |
| 1 | [Class vs Object](#1-class-vs-object) |
| 2 | [Encapsulation](#2-encapsulation) |
| 3 | [Inheritance + MRO](#3-inheritance--mro) |
| 4 | [Polymorphism](#4-polymorphism) |
| 5 | [Abstraction (ABC)](#5-abstraction-abc) |
| 6 | [Composition vs Inheritance](#6-composition-vs-inheritance) |
| 7 | [Dunder / Magic Methods](#7-dunder--magic-methods) |
| 8 | [staticmethod vs classmethod vs instance method](#8-staticmethod-vs-classmethod-vs-instance-method) |
| 9 | [dataclass (+ NamedTuple, TypedDict, Pydantic)](#9-dataclass--namedtuple-typeddict-pydantic) |
| 10 | [SOLID Principles](#10-solid-principles) |
| 11 | [Design Patterns in Test Frameworks](#11-design-patterns-in-test-frameworks) |
| — | **PART 2 — FRAMEWORK ARCHITECTURE** |
| 12 | [Layered Architecture](#12-layered-architecture) |
| 13 | [Page Object Model](#13-page-object-model) |
| 14 | [Page Factory (aur Python ko kyun nahi chahiye)](#14-page-factory-aur-python-ko-kyun-nahi-chahiye) |
| 15 | [Base Page](#15-base-page) |
| 16 | [Fixtures](#16-fixtures) |
| 17 | [Utilities / Helpers — "One Canonical Way"](#17-utilities--helpers--one-canonical-way) |
| 18 | [Test Data Management](#18-test-data-management) |
| 19 | [Config Management](#19-config-management) |
| 20 | [Logging](#20-logging) |
| 21 | [Reporting](#21-reporting) |
| 22 | [Retry Strategy](#22-retry-strategy) |
| 23 | [Parallelisation](#23-parallelisation) |
| 24 | [Tagging & Suite Composition](#24-tagging--suite-composition) |
| 25 | [Environment Separation](#25-environment-separation) |
| — | **PART 3 — SYSTEM DESIGN FOR TESTERS** |
| 26 | [The 6-Step Answer Framework](#26-the-6-step-answer-framework) |
| 27 | [Design: Scalable Playwright Framework for 5000 Tests](#27-design-scalable-playwright-framework-for-5000-tests) |
| 28 | [Design: Automation Platform for 10000 Tests](#28-design-automation-platform-for-10000-tests) |
| 29 | [Design: QA for a Payment System](#29-design-qa-for-a-payment-system) |
| 30 | [Design: Test Data Management at Scale](#30-design-test-data-management-at-scale) |
| 31 | [Design: QA Process for a New Team (30/60/90)](#31-design-qa-process-for-a-new-team-306090) |
| 32 | [Microservices Testing](#32-microservices-testing) |
| 33 | [Observability for QA](#33-observability-for-qa) |
| 34 | [Trade-off Questions](#34-trade-off-questions) |
| 35 | [Senior Scenario Questions](#35-senior-scenario-questions) |
| 36 | [Red Flags — Ye Jawab Mat Dena](#36-red-flags--ye-jawab-mat-dena) |
| 37 | [Quick Revision Table](#37-quick-revision-table) |

---

# PART 1 — OOP FROM ZERO

> **Framing jo tumhe interview mein use karni hai:** OOP ka asli maqsad "4 pillars" nahi hai. Asli maqsad hai — **change ko ek jagah rok dena**. Jab UI badle to sirf ek file badle. Jab auth badle to sirf ek class badle. Jitni jagah change ka blast radius chhota, utna framework mature.
>
> Ye ek line poori PART 1 ka thesis hai. Har pillar isi ka ek tool hai:
>
> | Pillar | Kis cheez ko rokta hai |
> |---|---|
> | Encapsulation | Internal state ka misuse |
> | Abstraction | Implementation detail ka leak |
> | Inheritance | Common behaviour ka duplication |
> | Polymorphism | `if/elif` chains ka phailna |

---

# 1. Class vs Object

## 1.1 Kya hai — analogy pehle

**Class = naksha (blueprint). Object = ghar jo us naksha se bana.**

Ek architect ek hi naksha banata hai — "2BHK, 900 sq ft, ek balcony". Us naksha se 50 ghar ban sakte hain. Har ghar ka **address alag**, **rang alag**, lekin **structure same**. Naksha mein koi rehta nahi — rehte log ghar mein hain.

- **Class** = definition, template. Memory mein ek baar.
- **Object (instance)** = us class ka ek actual copy, apne **data ke saath**.

Automation mein: `LoginPage` ek class hai. Har test run mein jo `LoginPage(page)` banta hai — wo object hai, aur uske andar **us test ka apna `page`** hota hai.

```
        class LoginPage           <- blueprint, ek hi hai
              │
     ┌────────┼────────┐
     v        v        v
  obj1     obj2      obj3        <- 3 objects, alag-alag page/state
 (test_a)  (test_b)  (test_c)
```

## 1.2 Code

```python
class LoginPage:
    # class variable — SAARE objects ke beech SHARED, ek hi copy
    URL_PATH = "/login"
    login_attempts = 0            # <- ye gotcha ban sakta hai, aage dekho

    def __init__(self, page, base_url: str):
        # instance variables — har object ka APNA
        self.page = page
        self.base_url = base_url
        self.username_input = page.get_by_label("Username")
        self.password_input = page.get_by_label("Password")
        self.submit_button = page.get_by_role("button", name="Sign in")

    def open(self) -> "LoginPage":
        self.page.goto(f"{self.base_url}{self.URL_PATH}")
        return self

    def login(self, user: str, pwd: str) -> "DashboardPage":
        self.username_input.fill(user)
        self.password_input.fill(pwd)
        self.submit_button.click()
        LoginPage.login_attempts += 1      # class var explicitly update
        return DashboardPage(self.page)
```

**`__init__` kya hai?** Constructor nahi — technically **initializer** hai. Object ban chuka hota hai (`__new__` ne banaya), `__init__` sirf usme data bharta hai. Ye distinction cross-question mein pooch lete hain.

```python
class Demo:
    def __new__(cls, *a, **kw):
        print("__new__ chala — object allocate hua")
        return super().__new__(cls)

    def __init__(self, name):
        print("__init__ chala — object ko data mila")
        self.name = name

Demo("x")
# __new__ chala — object allocate hua
# __init__ chala — object ko data mila
```

## 1.3 Instance vs Class variable — the worked gotcha

Ye **sabse zyada pooche jaane wala Python OOP trap** hai. Mutable class variable saare instances mein share hota hai.

```python
class TestUser:
    roles = []                    # ❌ MUTABLE class variable — BOMB

    def __init__(self, name):
        self.name = name

    def add_role(self, role):
        self.roles.append(role)   # self.roles -> class ki hi list mil rahi hai!

a = TestUser("ritik")
b = TestUser("amit")

a.add_role("admin")
print(b.roles)         # ['admin']   <- b ne kuch nahi kiya, phir bhi admin hai
print(a.roles is b.roles)   # True   <- ek hi list object
print(TestUser.roles)       # ['admin']
```

**Kyun hua?** `self.roles.append(...)` mein koi **assignment nahi** hai. Python `self.roles` ko instance mein dhoondta hai, nahi milta, class tak jaata hai, wahi list return karta hai, aur usi ko mutate kar deta hai.

**Fix:**

```python
class TestUser:
    def __init__(self, name, roles=None):
        self.name = name
        self.roles = roles if roles is not None else []   # har object ki apni list
```

Same bug ka doosra roop — **mutable default argument**:

```python
def create_po(items=[]):          # ❌ default list function definition pe EK BAAR banti hai
    items.append("cement")
    return items

print(create_po())   # ['cement']
print(create_po())   # ['cement', 'cement']   <- leak!

def create_po(items=None):        # ✅
    items = items or []
    items.append("cement")
    return items
```

**Assignment vs mutation ka rule:**

```python
a = TestUser("ritik")
a.roles = ["qa"]        # ASSIGNMENT -> ab instance variable ban gaya, class wali chhup gayi
print(TestUser.roles)   # [] ya jo bhi tha — untouched
```

| | Instance variable | Class variable |
|---|---|---|
| Kahan define | `__init__` ke andar `self.x = ...` | class body mein `x = ...` |
| Kiska | Har object ka apna | Sab objects mein shared |
| Kab use | Test-specific state (page, user, data) | Constants (`URL_PATH`, `DEFAULT_TIMEOUT`), counters |
| Khatra | — | Mutable ho to cross-instance leak |

## 1.4 Automation mein kahan lagta hai

- `BASE_TIMEOUT = 5000` — class variable, sahi use.
- `self.page` — instance variable, har test ka apna browser context.
- **Parallel run mein class variable = shared mutable state = flaky test.** Agar tumne test count ya "last created PO id" class variable mein rakha, aur xdist se 8 workers chale — data race. (Actually alag processes hote hain xdist mein, isliye process-level to bach jaate ho, lekin threads/async mein nahi.)

> **[REAL]** Merlin framework mein page classes nahi hain — functional helper modules hain (`_helpers/interactions.py`). Wahan module-level constant hi effectively "class variable" ka role kar raha hai: `DEFAULT_TIMEOUT_MS = 5000`. Module-level mutable dict rakhna wahi hi bug dega jo upar dikhaya — isliye warning-collector fixture ne state ko **fixture ke andar** rakha hai, module-level list mein nahi. Ye conscious decision tha.

> **Interview answer:** A class is a blueprint that defines structure and behaviour; an object is a concrete instance of that blueprint holding its own state. In Python, `__init__` is not the constructor — `__new__` allocates the object and `__init__` initialises it. The distinction I actually care about day to day is instance versus class variables: class variables are shared across every instance, so a mutable class variable like a list is a classic source of cross-test contamination. If one page object appends to it, every other instance sees the change. I keep class variables strictly for immutable constants like default timeouts and URL paths, and everything stateful goes in `__init__` as an instance variable. The same trap shows up as mutable default arguments in function signatures.

**Cross-question: "`a.roles.append('x')` class variable ko change karta hai, lekin `a.roles = ['x']` nahi. Kyun?"**
> Because `append` is a mutation on the object Python resolved through the lookup chain — instance first, then class — so it mutates the class's list. Assignment is different: assignment always creates or rebinds a name on the instance, shadowing the class attribute. So mutation reaches through to the class, assignment stops at the instance.

**Cross-question: "Kya class variable parallel test execution mein safe hai?"**
> Within a single process, no — it's shared mutable state. With pytest-xdist each worker is a separate process so class variables are naturally isolated per worker, which hides the bug rather than fixing it. The moment you move to threads, asyncio, or a single-process parallel runner, it breaks. I treat shared mutable module or class state as a design smell regardless of the current runner.

---

# 2. Encapsulation

## 2.1 Kya hai — analogy pehle

**ATM machine.** Tum card daalte ho, PIN, amount — paisa nikal aata hai. Machine ke andar cash cassettes, counters, sensors sab hai, lekin **tumhe access nahi**. Tumhare paas sirf **buttons** hain (public interface). Andar ka mechanism (private state) chhupa hai.

Encapsulation = **data aur usko change karne wale methods ko ek jagah bandh karna, aur bahar wale ko sirf controlled interface dena.**

Do faayde:
1. **Invalid state impossible ho jaata hai** — tum object ko galat state mein nahi daal sakte.
2. **Internal implementation badal sakti hai bina caller toote.**

## 2.2 Python mein 3 levels

Python mein `private` keyword nahi hai. Convention + naam-mangling hai.

```python
class ApiSession:
    def __init__(self, base_url, token):
        self.base_url = base_url       # public      — kahin se bhi use karo
        self._retries = 3              # protected   — "andar ka hai, mat chhedo" (convention)
        self.__token = token           # private     — name mangling ho jaati hai

    def whoami(self):
        return self.__token[:6] + "..."

s = ApiSession("https://api.merlin", "tok_abc123xyz")
print(s.base_url)     # ok
print(s._retries)     # chalega — Python rokta nahi, sirf signal deta hai
print(s.__token)      # ❌ AttributeError
print(s._ApiSession__token)   # ✅ chal gaya — mangled naam
```

**Name mangling exactly kya hai?** Compile time pe Python `__token` ko `_ClassName__token` mein rename kar deta hai. Ye **security nahi** hai — ye **accidental override se bachne ka mechanism** hai.

Asli use case (ye bolna interview mein impress karta hai):

```python
class BasePage:
    def __init__(self):
        self.__timeout = 5000          # mangled -> _BasePage__timeout

class InvoicePage(BasePage):
    def __init__(self):
        super().__init__()
        self.__timeout = 30000         # mangled -> _InvoicePage__timeout

p = InvoicePage()
print(p.__dict__)
# {'_BasePage__timeout': 5000, '_InvoicePage__timeout': 30000}
# Dono zinda hain — subclass ne base ka state accidentally corrupt nahi kiya
```

Agar single underscore hota to subclass base ki value overwrite kar deta aur base ke methods galat timeout use karte.

| Prefix | Naam | Enforcement | Kab use |
|---|---|---|---|
| `name` | public | koi nahi | Jo API tum support karne ko taiyaar ho |
| `_name` | protected | convention only | Internal helper; subclass use kar sakta hai |
| `__name` | private | name mangling | Sirf isi class ka; subclass clash se bachao |

## 2.3 Getters/setters aur `@property`

Java-style getter/setter Python mein **anti-pattern** hai:

```python
class Bad:                       # ❌ Python mein aise mat likho
    def __init__(self): self._x = 0
    def get_x(self): return self._x
    def set_x(self, v): self._x = v
```

Python ka tarika — plain attribute se shuru karo, jab validation/computation chahiye tab `@property` mein badal do. **Caller ka code nahi badalta.** Ye Python ki killer feature hai — "uniform access principle".

**Worked example — `status` property jo page se padhti hai:**

```python
class PurchaseOrderPage:
    def __init__(self, page):
        self.page = page
        self._status_badge = page.get_by_test_id("po-status-badge")

    @property
    def status(self) -> str:
        """UI se live status padhta hai — cached nahi, har baar fresh."""
        return self._status_badge.inner_text().strip().upper()

    @property
    def is_approved(self) -> bool:
        return self.status == "APPROVED"

    @property
    def total_amount(self) -> float:
        raw = self.page.get_by_test_id("po-total").inner_text()
        return float(raw.replace("₹", "").replace(",", "").strip())

po = PurchaseOrderPage(page)
assert po.status == "PENDING_APPROVAL"     # method call jaisa nahi dikhta, attribute jaisa
assert po.total_amount == 125000.0
assert po.is_approved is False
```

**Setter ke saath validation:**

```python
class TestConfig:
    def __init__(self, env: str):
        self._env = None
        self.env = env             # setter se hokar jaayega — validation free mein

    @property
    def env(self) -> str:
        return self._env

    @env.setter
    def env(self, value: str):
        allowed = {"staging", "qa", "dev"}
        if value not in allowed:
            raise ValueError(
                f"env='{value}' invalid. Allowed: {sorted(allowed)}. "
                "Production is intentionally blocked here."
            )
        self._env = value

c = TestConfig("staging")     # ok
c.env = "production"          # ❌ ValueError — object kabhi invalid state mein nahi jaayega
```

**Ye guardrail pattern senior-level signal hai.** "Maine config class mein production ko structurally block kar diya, discipline pe bharosa nahi kiya."

## 2.4 Automation mein kahan lagta hai

- **Live UI state** ko property banao — stale value cache mat karo. `po.status` har call pe DOM se padhta hai.
- **Derived assertions** (`is_approved`) property banao — test readable ho jaata hai.
- **Locators ko `_protected`** rakho — test file ko raw locator nahi chhuna chahiye (POM rule, section 13).
- **Secrets ko `__private`** rakho aur `__repr__` mein mask karo (section 7).

> **[REAL]** Merlin ke `_helpers/config.py` mein env-driven config hai. Encapsulation ka spirit wahan module-level pe apply hua hai: `get_base_url()` function expose hai, raw `os.environ` reads test files mein nahi hain. Same principle — internal source (env var) chhupa, controlled accessor bahar. Agar kal config ka source env var se AWS Secrets Manager ban jaaye, test file ka ek line nahi badlega.

> **Interview answer:** Encapsulation means bundling data with the methods that operate on it and exposing only a controlled interface, so an object can never be put into an invalid state from outside. Python doesn't enforce access — a single underscore is a convention meaning "internal", and a double underscore triggers name mangling to `_ClassName__attr`. That mangling isn't security; its real purpose is preventing a subclass from accidentally clobbering a base class's attribute. Practically, I don't write Java-style getters and setters in Python. I start with a plain attribute, and if I later need validation or computation I convert it to a `@property` — the call sites don't change. In page objects I use properties for live UI state, so `page.status` re-reads the DOM instead of returning a stale cached value.

**Cross-question: "Agar `__` private nahi hai to point kya hai?"**
> The point is name collision avoidance, not access control. Python's philosophy is "we're all consenting adults" — you can reach `_ClassName__attr` if you really need to, and that explicitness is deliberate. What double underscore actually guarantees is that if a base class has `__timeout` and a subclass defines `__timeout`, they're stored under different keys and neither corrupts the other. I use it for attributes a base class depends on internally, and single underscore for everything else.

**Cross-question: "`@property` kab NAHI use karna?"**
> When the operation is expensive or has side effects. A property looks like an attribute access, so callers assume it's cheap. If reading it triggers a network call, a five-second wait, or mutates state, it should be an explicit method so the cost is visible at the call site. I'd also avoid it when the method takes arguments — that's just a method.

---

# 3. Inheritance + MRO

## 3.1 Kya hai — analogy pehle

**Family inheritance.** Bachche ko parents se surname, kuch traits, kuch property automatically mil jaati hai. Bachcha usme apna kuch add kar sakta hai, ya kuch traits badal bhi sakta hai.

Code mein: **child class ko parent ke saare attributes aur methods free mein mil jaate hain.** Child unhe use kar sakta hai, extend kar sakta hai, ya override kar sakta hai.

**Relationship ka naam: "is-a".** `AdminDashboardPage` **is a** `DashboardPage`. Agar ye vaakya galat lagta hai — inheritance mat use karo, composition use karo (section 6).

## 3.2 Types of inheritance

```
SINGLE                MULTI-LEVEL              HIERARCHICAL           MULTIPLE
  A                      A                        A                  A     B
  │                      │                     ┌──┼──┐                \   /
  B                      B                     B  C  D                  C
                         │
                         C
```

```python
# ---------- 1. SINGLE ----------
class BasePage:
    def __init__(self, page):
        self.page = page
    def wait_for_load(self):
        self.page.wait_for_load_state("networkidle")

class LoginPage(BasePage):
    def login(self, u, p): ...


# ---------- 2. MULTI-LEVEL ----------
class AuthenticatedPage(BasePage):
    def logout(self):
        self.page.get_by_role("button", name="Logout").click()

class DashboardPage(AuthenticatedPage):      # BasePage -> AuthenticatedPage -> DashboardPage
    def open_module(self, name): ...

d = DashboardPage(page)
d.wait_for_load()   # BasePage se
d.logout()          # AuthenticatedPage se
d.open_module("PO") # apna


# ---------- 3. HIERARCHICAL ----------
class InvoicePage(BasePage): ...
class PurchaseOrderPage(BasePage): ...
class SupplierPage(BasePage): ...            # teeno ka same parent


# ---------- 4. MULTIPLE ----------
class TableMixin:
    def row_count(self):
        return self.page.locator("table tbody tr").count()

class ExportMixin:
    def export_csv(self):
        self.page.get_by_role("button", name="Export").click()
        return self.page.wait_for_event("download")

class POListPage(BasePage, TableMixin, ExportMixin):   # 3 parents
    pass
```

**Mixin** = ek chhoti class jo sirf behaviour deti hai, akele instantiate nahi hoti. Multiple inheritance ka **sabse safe** use yahi hai. Naam ke end mein `Mixin` likhna convention hai.

## 3.3 `super()` — properly

```python
class BasePage:
    def __init__(self, page, timeout_ms=5000):
        self.page = page
        self.timeout_ms = timeout_ms
        print("BasePage.__init__")

class InvoicePage(BasePage):
    def __init__(self, page, invoice_id):
        super().__init__(page, timeout_ms=15000)   # parent ka init pehle chalao
        self.invoice_id = invoice_id               # phir apna
        print("InvoicePage.__init__")
```

**`super().__init__()` bhoolna = classic bug.** Parent ne jo attributes set karne the (`self.page`) wo set hi nahi honge → `AttributeError` 20 lines baad, jo debug karna painful hai.

`super()` ko "parent" mat samjho — wo **MRO mein agla** hai. Multiple inheritance mein ye difference matter karta hai.

## 3.4 MRO aur Diamond Problem

**Diamond problem:** D, B aur C dono se inherit karta hai; B aur C dono A se. Agar D pe `greet()` call karo aur B, C dono ne override kiya hai — kiska chalega?

```
        A
       / \
      B   C
       \ /
        D
```

```python
class A:
    def who(self): return "A"

class B(A):
    def who(self): return "B -> " + super().who()

class C(A):
    def who(self): return "C -> " + super().who()

class D(B, C):
    def who(self): return "D -> " + super().who()

print(D().who())
# D -> B -> C -> A          <- B ka super() C hai, A nahi!

print([c.__name__ for c in D.__mro__])
# ['D', 'B', 'C', 'A', 'object']
```

**Yahi wo cheez hai jo log galat bolte hain.** `B.who` ke andar `super()` **`A` nahi**, **`C`** hai — kyunki `super()` MRO ke hisaab se resolve hota hai, class definition ke hisaab se nahi. Isi wajah se `A.who` sirf **ek baar** chalta hai, do baar nahi.

**MRO kaise banta hai — C3 linearization.** Rules:
1. Class khud pehle.
2. Parents left-to-right order mein.
3. Har class apne parents se **pehle** aani chahiye.
4. Contradiction ho to Python `TypeError` de deta hai class definition pe hi.

```python
class X: pass
class Y: pass
class Bad(X, Y): pass
class Worse(Y, X): pass
class Impossible(Bad, Worse): pass
# TypeError: Cannot create a consistent method resolution order (MRO)
# for bases X, Y     <- Bad kehta hai X pehle, Worse kehta hai Y pehle
```

**Cooperative multiple inheritance ka real example (test framework mixin chain):**

```python
class BasePage:
    def __init__(self, page, **kw):
        self.page = page
        super().__init__(**kw)          # chain aage badhao — IMPORTANT

class LoggingMixin:
    def __init__(self, logger=None, **kw):
        self.logger = logger or print
        super().__init__(**kw)

class RetryMixin:
    def __init__(self, retries=2, **kw):
        self.retries = retries
        super().__init__(**kw)

class POPage(LoggingMixin, RetryMixin, BasePage):
    pass

p = POPage(page="PAGE", logger=None, retries=3)
print([c.__name__ for c in POPage.__mro__])
# ['POPage', 'LoggingMixin', 'RetryMixin', 'BasePage', 'object']
print(p.retries, p.page)   # 3 PAGE   <- sabka __init__ chala
```

Rule: cooperative inheritance mein har `__init__` `**kw` accept kare aur `super().__init__(**kw)` call kare. Ek link toota to chain toot jaati hai.

## 3.5 Automation mein kahan lagta hai

| Sahi use | Galat use |
|---|---|
| `BasePage` → sab pages (common wait, screenshot, nav) | `LoginPage(BaseTest)` — page test nahi hai |
| `BaseApiClient` → `POClient`, `InvoiceClient` | 5-level deep hierarchy jahan koi nahi jaanta method kahan se aa raha |
| Mixins for cross-cutting (table, export, pagination) | Sirf code reuse ke liye inherit karna, "is-a" nahi hai |

**Golden rule:** Inheritance tab jab **substitutability** chahiye (LSP, section 10). Sirf reuse chahiye to composition better hai.

> **[REAL]** Merlin framework ne deliberately deep inheritance avoid kiya — helper **modules** hain, class hierarchy nahi. Kyun? 6 e2e suites hain, 12–15 steps each. Inheritance tree ka cost tab justify hota hai jab 50+ pages hon aur genuine shared contract ho. 6 suites pe ek `BasePage` banake usme sab kuch daal dena — ISP violation ho jaata (section 10). Isliye flat functional helpers + explicit imports. Ye "maine inheritance nahi use kiya kyunki zaroorat nahi thi" wala answer interview mein bahut strong lagta hai, agar reason bata sako.

> **Interview answer:** Inheritance models an "is-a" relationship and lets a subclass reuse and specialise a base class. Python supports single, multi-level, hierarchical and multiple inheritance. The one thing people get wrong is `super()` — it doesn't mean "my parent", it means "the next class in the MRO". In a diamond, where D inherits B and C which both inherit A, calling `super()` inside B resolves to C, not A. That's what makes cooperative multiple inheritance work and ensures A's method runs exactly once. The MRO is computed by C3 linearisation and you can inspect it with `__mro__`. In frameworks I keep inheritance shallow — usually one BasePage level plus mixins for cross-cutting behaviour — because deep hierarchies make it impossible to tell where a method actually comes from.

**Cross-question: "Diamond problem Python mein hota hai kya?"**
> The ambiguity is resolved, not eliminated. C3 linearisation gives a deterministic order, and if a consistent order can't be built Python raises a TypeError at class-definition time rather than at runtime. The remaining hazard is `__init__` chains: if one class in the chain doesn't call `super().__init__()`, the classes after it in the MRO are silently skipped. That's a real bug I look for in mixin-heavy code.

**Cross-question: "Multiple inheritance vs interface — Python mein kya better?"**
> For behaviour sharing I use mixins with a strict rule that a mixin never has its own state requirements beyond what it declares, and never inherits from anything but object. For contracts I use ABCs, or increasingly `typing.Protocol`, which gives structural typing — a class satisfies the protocol just by having the right methods, no inheritance needed. Protocol is closer to what Go does and fits Python's duck typing better.

**Cross-question: "Inheritance kab NAHI use karoge?"**
> When the relationship is "has-a" rather than "is-a", when I only want code reuse, or when the subclass would need to weaken the base class's contract — that's an LSP violation and it will break polymorphic callers. My default is composition, and I reach for inheritance only when I genuinely need substitutability.

---

# 4. Polymorphism

## 4.1 Kya hai — analogy pehle

**"Start" button.** Car ka start button, washing machine ka start button, microwave ka start button — **same naam, alag behaviour**. Tumhe har device ke andar ka mechanism jaanne ki zaroorat nahi. Tum bas `start()` bolte ho.

Polymorphism = **ek interface, kai implementations.** Caller ko farq nahi padta ki andar kya hai.

Automation mein iska cash value: **`if/elif` chains khatam ho jaate hain.**

```python
# ❌ Bina polymorphism
def login(page, auth_type, creds):
    if auth_type == "form":
        ...
    elif auth_type == "sso":
        ...
    elif auth_type == "api_token":
        ...
    elif auth_type == "otp":       # har naye auth pe ye function edit karna padega
        ...
```

Ye OCP violation bhi hai (section 10). Polymorphism isko fix karta hai.

## 4.2 Method overriding (runtime polymorphism)

```python
class BasePage:
    def is_loaded(self) -> bool:
        return self.page.locator("body").is_visible()

class DashboardPage(BasePage):
    def is_loaded(self) -> bool:                 # override
        return self.page.get_by_test_id("kpi-cards").is_visible()

class POListPage(BasePage):
    def is_loaded(self) -> bool:                 # override
        return self.page.locator("table tbody tr").count() > 0

def wait_until_ready(pages: list[BasePage]):
    for p in pages:
        assert p.is_loaded(), f"{type(p).__name__} not loaded"
        # caller ko pata nahi kaunsa is_loaded chal raha — that's the point
```

**Override karte waqt parent ko bhi chalana ho:**

```python
class AuditedPage(BasePage):
    def is_loaded(self):
        base_ok = super().is_loaded()            # parent ka kaam
        return base_ok and self.page.get_by_test_id("audit-trail").is_visible()
```

## 4.3 Duck typing — the Pythonic one

> "If it walks like a duck and quacks like a duck, it's a duck."

Python **type nahi dekhta, capability dekhta hai.** Koi common base class ki zaroorat nahi.

```python
class SlackNotifier:
    def send(self, msg: str): print(f"[slack] {msg}")

class EmailNotifier:
    def send(self, msg: str): print(f"[email] {msg}")

class NoOpNotifier:                              # test double
    def send(self, msg: str): pass

def report_failure(notifiers, test_name):
    for n in notifiers:                          # koi inheritance nahi, koi ABC nahi
        n.send(f"FAILED: {test_name}")

report_failure([SlackNotifier(), EmailNotifier(), NoOpNotifier()], "test_po_approval")
```

Teeno unrelated classes hain. Kaam kar gaya kyunki teeno mein `send` hai. **Yahi wajah hai ki Python mein mocking itni aasan hai** — tumhe interface implement nahi karna padta, bas method banana hai.

**`typing.Protocol` — duck typing + type checking dono:**

```python
from typing import Protocol

class Notifier(Protocol):
    def send(self, msg: str) -> None: ...

def report_failure(notifiers: list[Notifier], test_name: str) -> None:
    for n in notifiers:
        n.send(f"FAILED: {test_name}")

# SlackNotifier ko Notifier inherit karne ki zaroorat NAHI —
# mypy structurally check kar lega ki send(msg: str) -> None maujood hai
```

Ye senior-level detail hai. Bolna: *"I prefer Protocol over ABC when I want a contract without forcing an inheritance relationship."*

## 4.4 Operator overloading (compile-time-ish / ad-hoc polymorphism)

Dunder methods se built-in operators ko apne objects pe kaam karwana.

```python
class Money:
    def __init__(self, amount: int, currency: str = "INR"):
        self.amount = amount           # paise mein — float NEVER for money
        self.currency = currency

    def __add__(self, other: "Money") -> "Money":
        if self.currency != other.currency:
            raise ValueError(f"Cannot add {self.currency} + {other.currency}")
        return Money(self.amount + other.amount, self.currency)

    def __sub__(self, other): return Money(self.amount - other.amount, self.currency)
    def __mul__(self, n: int): return Money(self.amount * n, self.currency)
    def __eq__(self, other): return (self.amount, self.currency) == (other.amount, other.currency)
    def __lt__(self, other): return self.amount < other.amount
    def __repr__(self): return f"Money({self.amount}, {self.currency!r})"
    def __str__(self):  return f"₹{self.amount / 100:,.2f}"

line1 = Money(125000)     # ₹1250.00
line2 = Money(75000)
total = line1 + line2
print(total)              # ₹2,000.00
print(repr(total))        # Money(200000, 'INR')
print(line1 > line2)      # True
```

Assertion ab domain language mein likhi jaati hai — `assert po.total == Money(200000)` — aur failure message readable hota hai.

## 4.5 Method overloading — Python mein kyun nahi hai

Java/C++ mein same naam ke multiple methods alag signature ke saath ho sakte hain. **Python mein nahi.** Baad wali definition pehli ko replace kar deti hai:

```python
class Search:
    def find(self, name): return f"by name: {name}"
    def find(self, name, role): return f"by name+role: {name},{role}"   # pehli wali gayab

s = Search()
s.find("ritik")           # ❌ TypeError: find() missing 1 required positional argument
```

**Kyun nahi hai?** Python dynamically typed hai — function object ek naam se bandha hota hai namespace dict mein. Same naam = same key = overwrite. Compile-time type-based dispatch ka concept hi nahi.

**Workarounds — teeno pata hone chahiye:**

```python
# 1. Default arguments
class Search:
    def find(self, name, role=None):
        return f"by name: {name}" if role is None else f"by name+role: {name},{role}"

# 2. *args / **kwargs
class Locator:
    def get(self, *args, **kwargs):
        if "test_id" in kwargs:
            return self.page.get_by_test_id(kwargs["test_id"])
        if len(args) == 1:
            return self.page.locator(args[0])
        raise TypeError("get() needs a selector or test_id=")

# 3. functools.singledispatchmethod — asli type-based dispatch
from functools import singledispatchmethod

class Assertion:
    @singledispatchmethod
    def check(self, expected):
        raise NotImplementedError(f"No check for {type(expected)}")

    @check.register
    def _(self, expected: str):
        print(f"string compare: {expected}")

    @check.register
    def _(self, expected: int):
        print(f"numeric compare: {expected}")

    @check.register
    def _(self, expected: list):
        print(f"list membership: {expected}")

a = Assertion()
a.check("APPROVED")     # string compare
a.check(42)             # numeric compare
a.check(["A", "B"])     # list membership
```

**Note:** `singledispatchmethod` **first argument ke runtime type** pe dispatch karta hai — ye still runtime polymorphism hai, compile-time overloading nahi. Aur `typing.overload` sirf type checker ke liye hai, runtime pe kuch nahi karta. Ye distinction bolna strong hai.

## 4.6 Automation mein kahan lagta hai

- Alag auth strategies — same `authenticate(page)` method (Strategy pattern, section 11).
- Alag environments — same `get_base_url()`.
- Test doubles — real client aur fake client, same interface (duck typing).
- Alag report formats — same `write(results)`.

> **[REAL]** `_helpers/interactions.py` mein `click(locator, *, timeout_ms=5000)` — ye polymorphic hai duck typing se: `locator` chahe Playwright `Locator` ho, chahe `FrameLocator` se nikla ho, chahe test double ho — jab tak usme `.click()` hai, kaam karega. Isi wajah se helper ko unit-test karna aasaan hai: ek fake locator object bana do, `.click()` record kar lo, done. Koi Playwright browser chahiye hi nahi.

> **Interview answer:** Polymorphism means one interface with many implementations, so calling code doesn't need to know the concrete type. In Python it mostly shows up as method overriding and duck typing. Duck typing is the Pythonic form — I don't need a shared base class, I just need the object to have the method I'm calling. That's exactly why test doubles are so easy in Python: a fake API client only needs the same method names, not an inheritance relationship. Python doesn't support compile-time method overloading because a function name is just a key in a namespace dict, so a second definition replaces the first. The workarounds are default arguments, `*args`/`**kwargs` dispatch, or `functools.singledispatch` for genuine type-based dispatch. Practically, polymorphism is what lets me delete `if/elif` chains — different auth strategies become different classes with the same `authenticate` method instead of a growing conditional.

**Cross-question: "Duck typing ka downside kya hai?"**
> Errors move from definition time to call time. If a class is missing a method, you find out when that line executes — possibly deep inside a long test. I mitigate that with `typing.Protocol` plus mypy in CI, which gives me structural typing checked statically, so I get the flexibility of duck typing with the safety of a declared contract.

**Cross-question: "Overloading aur overriding mein difference?"**
> Overriding is runtime polymorphism — a subclass replaces a base class method with the same signature, and dispatch happens on the object's actual type. Overloading is compile-time, multiple methods with the same name but different signatures, resolved by the compiler from argument types. Python has overriding but not overloading; `typing.overload` only informs the type checker and has no runtime effect.

---

# 5. Abstraction (ABC)

## 5.1 Kya hai — analogy pehle

**Gaadi ka steering.** Tum steering ghumate ho, gaadi mudti hai. Andar rack-and-pinion hai ya electric power steering — tumhe farq nahi padta. Interface same, implementation chhupi hui.

**Abstraction vs Encapsulation — ye difference pooch lete hain:**

| | Abstraction | Encapsulation |
|---|---|---|
| Kya karta hai | Implementation **detail chhupata** hai | **Data ko protect** karta hai |
| Focus | Design level — "kya expose karna hai" | Implementation level — "kaun chhu sakta hai" |
| Tool | ABC, interface, Protocol | private/protected, property |
| Ek line | *What* vs *how* | *Who* can touch |

## 5.2 `abc.ABC` + `@abstractmethod` — worked example

Problem: Har page ka apna "loaded" ka matlab hai. Agar `BasePage` mein default `is_loaded()` de doge, koi subclass usko override karna bhool jaayega aur test **jhooth bol dega** — wait pass ho jaayega jabki page load hi nahi hua.

Solution: **compiler-level majboori.**

```python
from abc import ABC, abstractmethod
from playwright.sync_api import Page

class BasePage(ABC):
    """Har page ko is_loaded() implement karna PADEGA."""

    def __init__(self, page: Page, base_url: str):
        self.page = page
        self.base_url = base_url

    # ---------- subclass ko force ----------
    @property
    @abstractmethod
    def path(self) -> str:
        """URL path, e.g. '/purchase-orders'"""

    @abstractmethod
    def is_loaded(self) -> bool:
        """Page-specific readiness check. NO default — har page apna decide kare."""

    # ---------- concrete, sabko free ----------
    def open(self):
        self.page.goto(f"{self.base_url}{self.path}")
        self.wait_until_loaded()
        return self

    def wait_until_loaded(self, timeout_ms: int = 15000):
        import time
        deadline = time.monotonic() + timeout_ms / 1000
        while time.monotonic() < deadline:
            if self.is_loaded():
                return self
            self.page.wait_for_timeout(250)
        raise TimeoutError(f"{type(self).__name__} not loaded in {timeout_ms}ms")

    def screenshot(self, name: str):
        self.page.screenshot(path=f"artifacts/{name}.png", full_page=True)


class POListPage(BasePage):
    path = "/purchase-orders"

    def is_loaded(self) -> bool:
        return self.page.get_by_test_id("po-table").is_visible()


class BrokenPage(BasePage):
    path = "/broken"
    # is_loaded implement nahi kiya

BrokenPage(page, "https://app.merlin")
# ❌ TypeError: Can't instantiate abstract class BrokenPage
#    with abstract method is_loaded
```

**Key insight:** Error **instantiation pe** aata hai, test failure ke 40 line baad nahi. Ye "fail fast" hai — abstraction ka asli business value.

## 5.3 ABC vs Protocol vs NotImplementedError

```python
# 1. NotImplementedError — weakest. Runtime pe pata chalta hai, wo bhi method call pe.
class BasePage:
    def is_loaded(self):
        raise NotImplementedError

# 2. ABC — instantiation pe hi block. Nominal typing (inherit karna padega).
class BasePage(ABC):
    @abstractmethod
    def is_loaded(self) -> bool: ...

# 3. Protocol — structural typing. Inherit nahi karna, mypy check karta hai.
from typing import Protocol, runtime_checkable

@runtime_checkable
class Loadable(Protocol):
    def is_loaded(self) -> bool: ...

def wait_all(pages: list[Loadable]): ...     # koi bhi class jisme is_loaded hai, chalega
```

| | NotImplementedError | ABC | Protocol |
|---|---|---|---|
| Error kab | Method call pe | Instantiation pe | mypy run pe (static) |
| Inheritance chahiye | Haan | Haan | Nahi |
| Third-party class adapt kar sakte | Nahi | Nahi | **Haan** |
| Kab use | Legacy / quick | Apni hierarchy | Contract without coupling |

## 5.4 Automation mein kahan lagta hai

- `BasePage.is_loaded()` — jaisa upar.
- `BaseApiClient.authenticate()` — har env ka apna auth.
- `BaseReporter.publish(results)` — HTML, Allure, Slack.
- `TestDataFactory.create()` / `cleanup()` — cleanup ko abstract rakhna forces har factory ko cleanup sochna. Ye **orphan test data** rokta hai.

```python
class DataFactory(ABC):
    def __init__(self): self._created: list[str] = []

    @abstractmethod
    def create(self, **kw) -> dict: ...

    @abstractmethod
    def delete(self, entity_id: str) -> None: ...

    def cleanup_all(self):
        for eid in reversed(self._created):     # reverse — FK dependencies
            try:
                self.delete(eid)
            except Exception as e:
                print(f"[cleanup] {eid} failed: {e}")   # cleanup kabhi test fail na kare
        self._created.clear()
```

> **[REAL]** Merlin framework mein ABC nahi hai — kyunki class hierarchy hi nahi hai. Lekin **wahi guarantee module-level pe achieve hui hai**: `_helpers/interactions.py` mein har action ka *ek hi* canonical function hai, aur alternatives ko comment mein explicitly forbidden mark kiya gaya hai with reason (`force=True` banned — kyunki ye actionability check bypass karta hai aur genuine UI bug ko chhupa deta hai). ABC "must implement" enforce karta hai; ye module "must not use" enforce karta hai — dono ka maqsad same: **galat raasta band karo.**

> **Interview answer:** Abstraction is about exposing what something does while hiding how it does it. In Python I enforce it with `abc.ABC` and `@abstractmethod`. The concrete value is fail-fast: if a subclass forgets to implement a required method, Python refuses to instantiate the class, so I get a clear error at construction time rather than a confusing failure deep in a test. A good example is a BasePage where `is_loaded()` is abstract — if I gave it a default implementation, some page would inherit the wrong readiness check and the wait would silently pass on an unloaded page, which is worse than failing. I also use `typing.Protocol` when I want a contract without forcing inheritance, especially for adapting third-party objects.

**Cross-question: "Abstract class mein concrete method ho sakta hai?"**
> Yes, and that's usually the point. An ABC typically mixes abstract methods that define the contract with concrete methods that provide shared behaviour built on top of that contract — that's the Template Method pattern. In my BasePage, `is_loaded()` is abstract but `wait_until_loaded()` is concrete and calls it in a loop.

**Cross-question: "Abstract class vs interface?"**
> Python has no separate interface keyword. An ABC with only abstract methods and no state is effectively an interface. The practical difference in other languages is that interfaces can't hold state or implementation and a class can implement many, while abstract classes can hold both but you inherit only one. In Python multiple inheritance blurs this, so I use ABCs for hierarchies I own and Protocols where I'd have used an interface.

---

# 6. Composition vs Inheritance

## 6.1 Kya hai — analogy pehle

- **Inheritance = "is-a".** Sports car **is a** car. Bachcha **is a** insaan.
- **Composition = "has-a".** Car **has an** engine. Car engine nahi hai — car ke andar engine hai.

Simple test: vaakya bolo. *"InvoicePage is a BasePage"* — sahi lagta hai, inheritance chalega. *"InvoicePage is a ApiClient"* — bakwaas hai. Page ke paas API client **hota hai**. Composition.

**Ek aur analogy jo interview mein bolna:** Inheritance = **paidaishi rishta** — badal nahi sakte, permanent hai, aur parent ke saare gun-dosh milte hain. Composition = **naukri pe rakhna** — jab chaaho badal do, jitna chaaho utna hi kaam do.

## 6.2 Same example, dono tarike se

**Requirement:** Ek `POPage` jo purchase order UI handle kare, aur usko API se PO create bhi karwana ho (setup ke liye), aur actions log bhi karne hon.

### Version A — Inheritance se (jaisa log pehle likhte hain)

```python
class ApiClient:
    def __init__(self, base_url, token):
        self.base_url, self.token = base_url, token
    def post(self, path, payload):
        print(f"POST {self.base_url}{path} {payload}")
        return {"id": "PO-1001"}
    def get(self, path):
        print(f"GET {self.base_url}{path}")
        return {"status": "DRAFT"}
    def delete(self, path):
        print(f"DELETE {self.base_url}{path}")


class Logger:
    def log(self, msg):
        print(f"[LOG] {msg}")


class BasePage:
    def __init__(self, page):
        self.page = page
    def goto(self, path):
        self.page.goto(path)


class POPage(BasePage, ApiClient, Logger):        # ❌ 3 parents
    def __init__(self, page, base_url, token):
        BasePage.__init__(self, page)
        ApiClient.__init__(self, base_url, token)

    def create_po_via_api(self, payload):
        self.log("creating PO via API")
        return self.post("/api/po", payload)       # apna hi method jaisa lagta hai

    def approve_in_ui(self):
        self.log("approving in UI")
        self.page.get_by_role("button", name="Approve").click()
```

**Kya toota:**
1. `POPage` **is a** `ApiClient`? Nahi. Semantically jhooth hai.
2. `po_page.delete("/api/po/1")` — page object pe HTTP delete expose ho gaya. Koi bhi test isko call kar sakta hai. **API surface phool gaya.**
3. `POPage` ke paas ab `get`, `post`, `delete`, `log`, `goto` — 5 public methods jo uske domain ke nahi hain. IntelliSense kachra.
4. Naam clash ka khatra: agar `BasePage` mein bhi `get()` hota (get element), MRO decide karta — silently galat method chal jaata.
5. `ApiClient` ko mock karna? Poora `POPage` mock karna padega. Testing hard.
6. `ApiClient` ka constructor signature badla → `POPage` toota. **Tight coupling.**

### Version B — Composition se (jaisa likhna chahiye)

```python
class POPage:
    def __init__(self, page, api: ApiClient, logger: Logger):
        self.page = page
        self._api = api            # HAS-A
        self._log = logger         # HAS-A

    # ---- domain language mein public API, HTTP verbs bahar nahi ----
    def seed_draft_po(self, supplier: str, amount: int) -> str:
        self._log.log(f"seeding draft PO for {supplier}")
        resp = self._api.post("/api/po", {"supplier": supplier, "amount": amount})
        return resp["id"]

    def open(self, po_id: str):
        self._log.log(f"opening PO {po_id}")
        self.page.goto(f"/purchase-orders/{po_id}")
        return self

    def approve(self):
        self.page.get_by_role("button", name="Approve").click()
        return self

    @property
    def status(self) -> str:
        return self.page.get_by_test_id("po-status").inner_text().strip()
```

**Kya theek hua:**

| Problem (inheritance) | Composition mein |
|---|---|
| `POPage is-a ApiClient` jhooth | `POPage has-a ApiClient` — sach |
| `post/get/delete` leak | `_api` private, sirf `seed_draft_po` expose |
| Mock karna mushkil | `POPage(page, FakeApi(), NoOpLogger())` — ek line |
| Constructor coupling | `ApiClient` badle to bas injection point badle |
| MRO clash | Koi MRO hi nahi |
| Runtime swap nahi | `po._api = staging_api` — runtime pe swap possible |

**Testing ka farq — ye demo dena interview mein:**

```python
class FakeApi:
    def __init__(self): self.calls = []
    def post(self, path, payload):
        self.calls.append((path, payload))
        return {"id": "PO-FAKE-1"}

class NoOpLogger:
    def log(self, msg): pass

def test_seed_uses_correct_payload():
    fake = FakeApi()
    po = POPage(page=None, api=fake, logger=NoOpLogger())
    po.seed_draft_po("ACME Cement", 125000)
    assert fake.calls == [("/api/po", {"supplier": "ACME Cement", "amount": 125000})]
    # Koi browser nahi, koi network nahi, 2ms mein chala
```

Inheritance wale version mein ye test likhna hi mushkil hai.

## 6.3 Kab kya

```
Ye sawaal poocho:
  "Kya X ko har jagah Y ki tarah use kiya ja sakta hai?"  (Liskov)
      │
   HAAN ────> Inheritance theek hai
      │
   NAHI ────> Composition
```

| Inheritance chuno jab | Composition chuno jab |
|---|---|
| Sach mein "is-a" hai | "has-a" / "uses-a" hai |
| Substitutability chahiye (polymorphic list) | Sirf reuse chahiye |
| Base class stable hai, badalti nahi | Behaviour runtime pe badalna hai |
| 1–2 level deep | 3+ level ban raha hai |
| Framework tumhara hai | Third-party class extend kar rahe ho |

**Industry rule of thumb:** *"Favour composition over inheritance"* — Gang of Four, 1994. Reason: inheritance **compile-time** pe fix ho jaata hai aur base class ke internals pe depend karta hai (fragile base class problem). Composition **runtime** pe flexible hai.

**Fragile base class problem — 1 line mein:** Base class ke andar ka change (jo public API nahi bhi hai) subclass ko silently tod sakta hai, kyunki subclass base ke implementation pe depend karta hai, sirf interface pe nahi.

## 6.4 Automation mein kahan lagta hai

- **Page + API client** — composition (upar wala example).
- **Page + wait strategy** — composition. Kuch pages ko network idle chahiye, kuch ko spinner-gone. Inheritance se karoge to `BaseNetworkIdlePage`, `BaseSpinnerPage` — class explosion.
- **BasePage → LoginPage** — inheritance theek hai, genuinely is-a.
- **Test class + fixtures** — pytest ka poora model composition hai. `def test_x(page, api_client, po_factory)` — ye dependency injection hai, inheritance nahi. Isi wajah se pytest unittest se behtar scale karta hai.

> **[REAL]** Merlin ka framework composition ka hi extreme version hai — **functions + explicit imports**, koi hierarchy nahi. `supplier_portal.py` ko `interactions.py` ki functions **use** karni hain, us se inherit nahi karni. Test file `from _helpers.interactions import click, fill` likhta hai — jo chahiye wahi aata hai, ek fat base class se sab kuch nahi. Ye ISP + composition dono satisfy karta hai. Interview mein bolna: *"I deliberately chose flat composable modules over a page-object hierarchy because with six suites the coupling cost of a base class outweighed the reuse benefit."*

> **Interview answer:** Inheritance models "is-a" and composition models "has-a". The test I apply is Liskov substitutability — if an object of the subclass can't be used everywhere the base is expected, it shouldn't inherit. Concretely: a POPage that needs to seed data over HTTP should hold an API client, not inherit from one. If it inherits, the page suddenly exposes `post`, `get` and `delete` to every test, semantics are wrong, and swapping in a fake client for a unit test means mocking the page itself. With composition I inject the client, expose one domain-level method like `seed_draft_po`, and testing it takes a five-line fake. The general rule is favour composition over inheritance, because inheritance is fixed at class-definition time and couples you to the base class's internals, while composition is swappable at runtime. I still use inheritance where it's genuinely an is-a with a stable base — a single BasePage level, for example.

**Cross-question: "Toh inheritance kabhi use hi nahi karoge?"**
> I use it, but deliberately and shallowly. One level of BasePage giving genuinely universal behaviour is fine — every page can be navigated to, screenshotted, and checked for readiness. What I avoid is inheritance used purely as a code-reuse mechanism, and anything beyond two levels, because at that depth nobody can tell where a method actually comes from without reading the whole chain.

**Cross-question: "Fragile base class problem kya hai?"**
> It's when a change inside a base class — even a change to a private detail that keeps the public API identical — breaks subclasses, because subclasses depend on the base's implementation, not just its interface. A classic example is a base class method that internally calls another of its own methods; a subclass overrides that second method, and now refactoring the base to stop calling it silently changes the subclass's behaviour. Composition avoids this because the collaborator is only reachable through its public interface.

---

# 7. Dunder / Magic Methods

## 7.1 Kya hai — analogy pehle

**Universal remote ke standard buttons.** Har device ka apna circuit hai, lekin sabme "power", "volume up", "mute" button same jagah hai. Tum ek hi remote se sab chala lete ho.

Dunder methods = Python ke **standard buttons**. `len(x)` internally `x.__len__()` call karta hai. `x == y` internally `x.__eq__(y)`. Agar tum apni class mein ye buttons laga do, **tumhari class built-in types jaisi behave karne lagti hai** — `for` loop, `sorted()`, `in`, `with`, sab free mein.

Ye Python ka "protocol" model hai — inheritance nahi, **method naam se contract**.

## 7.2 `__repr__` vs `__str__` — the classic question

| | `__repr__` | `__str__` |
|---|---|---|
| Kiske liye | **Developer** | **End user** |
| Kab chalta | `repr(x)`, REPL, debugger, list ke andar, **logs** | `str(x)`, `print(x)`, f-string |
| Goal | Unambiguous — ideally `eval()` karke object wapas ban jaaye | Readable |
| Fallback | — | Agar `__str__` nahi hai to `__repr__` use hota hai |
| Reverse fallback | `__str__` ho aur `__repr__` na ho → `repr()` default `<object at 0x...>` deta hai | — |

```python
class TestUser:
    def __init__(self, username, password, role):
        self.username, self.password, self.role = username, password, role

    def __repr__(self):
        # secret MASK — ye senior-level habit hai
        return f"TestUser(username={self.username!r}, role={self.role!r}, password='***')"

    def __str__(self):
        return f"{self.username} ({self.role})"

u = TestUser("ritik.qa", "Sup3rSecret!", "PROJECT_ADMIN")
print(u)          # ritik.qa (PROJECT_ADMIN)          <- __str__
print(repr(u))    # TestUser(username='ritik.qa', role='PROJECT_ADMIN', password='***')
print([u])        # [TestUser(username='ritik.qa', ...)]   <- list __repr__ use karti hai
```

**Ye kyun matter karta hai automation mein:** pytest assertion failure mein object ka **`__repr__`** dikhta hai. Agar `__repr__` nahi likha:

```
E   assert <tests.models.PurchaseOrder object at 0x10f3a2b50> == <tests.models.PurchaseOrder object at 0x10f3a2c10>
```

Ye useless hai. `__repr__` ke saath:

```
E   assert PurchaseOrder(id='PO-1001', status='DRAFT', total=125000) ==
E          PurchaseOrder(id='PO-1001', status='APPROVED', total=125000)
```

**Ab failure padhne se hi root cause dikh gaya.** Ye ek chhota kaam hai jo debugging time ghanton bachata hai.

**Rule:** `__repr__` hamesha likho. `__str__` sirf tab jab user-facing output alag chahiye. Aur `!r` use karo repr ke andar (`{self.username!r}`) taaki string quotes ke saath dikhe — `''` vs `None` ka farq pata chale.

## 7.3 `__eq__` + `__hash__` — ye saath kyun jaate hain

**Analogy:** Library mein book dhoondhne ke do tarike — **shelf number** (hash) se seedha jao, phir **title match** (eq) karke confirm karo. Agar do same books ke shelf number alag ho gaye, tum kabhi confirm hi nahi kar paoge ki wo same hai.

Python ka rule: **agar `a == b` hai, to `hash(a) == hash(b)` hona hi chahiye.** Warna `set` aur `dict` toot jaate hain.

```python
class PurchaseOrder:
    def __init__(self, po_id, status, total):
        self.po_id, self.status, self.total = po_id, status, total

    def __eq__(self, other):
        if not isinstance(other, PurchaseOrder):
            return NotImplemented          # ✅ NotImplemented, not False — Python reverse try karega
        return self.po_id == other.po_id

    def __repr__(self):
        return f"PurchaseOrder({self.po_id!r}, {self.status!r}, {self.total})"

a = PurchaseOrder("PO-1001", "DRAFT", 125000)
b = PurchaseOrder("PO-1001", "APPROVED", 125000)
print(a == b)        # True — same id
print({a, b})        # ❌ TypeError: unhashable type: 'PurchaseOrder'
```

**`__eq__` define karte hi Python `__hash__` ko `None` set kar deta hai.** Kyun? Kyunki default hash `id()` based hai (memory address), aur ab `a == b` hai lekin unke address alag hain → hash alag → set/dict corrupt. Python ne isko **silently todne ke bajaye loudly block** kar diya. Ye ek achha design decision hai jo interview mein bolna banta hai.

**Fix:**

```python
class PurchaseOrder:
    def __init__(self, po_id, status, total):
        self.po_id, self.status, self.total = po_id, status, total

    def __eq__(self, other):
        if not isinstance(other, PurchaseOrder): return NotImplemented
        return self.po_id == other.po_id

    def __hash__(self):
        return hash(self.po_id)            # SAME field jispe eq based hai

a, b = PurchaseOrder("PO-1001", "DRAFT", 1), PurchaseOrder("PO-1001", "APPROVED", 2)
print(len({a, b}))     # 1 — set ne dono ko same maana ✅
```

**Golden rules:**
1. `__hash__` hamesha **usi field(s)** pe based ho jispe `__eq__` hai.
2. Hash **immutable** field pe ho. Agar `po_id` badal diya set mein daalne ke baad — object "kho" jaayega, `in` check False dega.
3. Mutable object ko hashable mat banao. Isliye `frozen=True` dataclass (section 9) sweet spot hai.

**Set-based assertion — real use:**

```python
expected = {PurchaseOrder("PO-1001", ...), PurchaseOrder("PO-1002", ...)}
actual   = set(api.list_purchase_orders())
assert actual == expected, f"Missing: {expected - actual}, Unexpected: {actual - expected}"
# ye failure message ek order-independent diff deta hai — list comparison se bahut behtar
```

## 7.4 `__lt__` — sorting

Sirf `__lt__` chahiye `sorted()` ke liye. Baaki comparisons ke liye `functools.total_ordering`.

```python
from functools import total_ordering

@total_ordering
class TestResult:
    SEVERITY = {"CRITICAL": 0, "HIGH": 1, "MEDIUM": 2, "LOW": 3}

    def __init__(self, name, severity, duration_s):
        self.name, self.severity, self.duration_s = name, severity, duration_s

    def __eq__(self, other):
        return (self.severity, self.name) == (other.severity, other.name)

    def __lt__(self, other):
        # severity pehle, phir duration descending (slow test upar)
        return (self.SEVERITY[self.severity], -self.duration_s) < \
               (self.SEVERITY[other.severity], -other.duration_s)

    def __repr__(self):
        return f"{self.severity:<8} {self.duration_s:>6.1f}s  {self.name}"

results = [
    TestResult("test_login", "LOW", 4.2),
    TestResult("test_po_approval", "CRITICAL", 31.0),
    TestResult("test_invoice_pdf", "HIGH", 12.5),
    TestResult("test_po_reject", "CRITICAL", 9.0),
]
for r in sorted(results): print(r)
# CRITICAL   31.0s  test_po_approval
# CRITICAL    9.0s  test_po_reject
# HIGH       12.5s  test_invoice_pdf
# LOW         4.2s  test_login
```

Report generation ke liye ye exactly wahi chahiye — **critical failures upar, slow tests pehle.**

## 7.5 `__len__`, `__getitem__`, `__contains__` — container banana

```python
class TestSuite:
    def __init__(self, name, tests=None):
        self.name = name
        self._tests = list(tests or [])

    def __len__(self):                 # len(suite)
        return len(self._tests)

    def __getitem__(self, idx):        # suite[0], suite[1:3], aur for-loop bhi FREE
        return self._tests[idx]

    def __contains__(self, name):      # "test_x" in suite
        return any(t.name == name for t in self._tests)

    def __iter__(self):                # explicit iterator — __getitem__ se behtar
        return iter(self._tests)

    def __bool__(self):                # if suite:   -> khaali suite falsy
        return bool(self._tests)

    def __add__(self, other):          # smoke + regression
        return TestSuite(f"{self.name}+{other.name}", self._tests + other._tests)

suite = TestSuite("smoke", results)
print(len(suite))                      # 4
print(suite[0])                        # pehla test
print("test_login" in suite)           # True
for t in suite: pass                   # iterate — kuch extra likha nahi
if not suite: print("empty suite!")    # __bool__
combined = suite + TestSuite("reg", [])
```

**Gotcha:** agar sirf `__getitem__` ho aur `__iter__` na ho, Python phir bhi iterate kar leta hai (legacy protocol: index 0,1,2... jab tak IndexError na aaye). Lekin `__iter__` likhna explicit aur fast hai.

**`__bool__` ka trap:** agar `__bool__` nahi hai lekin `__len__` hai, to `if obj:` **`len(obj) != 0`** use karega. Matlab khaali container falsy ho jaayega — aksar ye chahiye hi hota hai, lekin agar nahi chahiye to `__bool__` explicitly define karo.

## 7.6 `__call__` — object ko function banana

```python
class RetryPolicy:
    def __init__(self, attempts=3, delay_ms=500, on=(TimeoutError,)):
        self.attempts, self.delay_ms, self.on = attempts, delay_ms, on

    def __call__(self, fn, *args, **kwargs):
        import time
        last = None
        for i in range(1, self.attempts + 1):
            try:
                return fn(*args, **kwargs)
            except self.on as e:
                last = e
                print(f"[retry] attempt {i}/{self.attempts} failed: {e}")
                time.sleep(self.delay_ms / 1000)
        raise last

retry_flaky_infra = RetryPolicy(attempts=3, delay_ms=1000)
result = retry_flaky_infra(api.get_purchase_order, "PO-1001")
```

Faayda vs plain function: **object mein config hai**, aur wo config inspect/change ho sakti hai. `retry_flaky_infra.attempts` padha ja sakta hai, log kiya ja sakta hai, test mein `attempts=1` set kiya ja sakta hai. Function closure mein ye nahi ho sakta.

`__call__` ke saath object **callable** ban jaata hai — matlab `callable(retry_flaky_infra)` True, aur wo kahin bhi jaa sakta hai jahan function expect ho.

## 7.7 `__enter__` / `__exit__` — context manager

**Analogy:** Ghar se nikalte waqt gas band karna. Kaam kitna bhi galat ho jaaye — gas band **hoga hi**. Context manager wahi guarantee deta hai: cleanup **hamesha** chalega, exception aaye ya na aaye.

```python
class TestDataScope:
    """Test ke andar bane saare records ko exit pe delete kar do."""

    def __init__(self, api):
        self.api = api
        self.created: list[tuple[str, str]] = []   # (entity_type, id)

    def __enter__(self):
        print("[scope] open")
        return self                       # `as` ko yahi milta hai

    def create_po(self, supplier, amount):
        po = self.api.post("/api/po", {"supplier": supplier, "amount": amount})
        self.created.append(("po", po["id"]))
        return po

    def __exit__(self, exc_type, exc_value, tb):
        # exc_type None hai to sab theek gaya
        for entity, eid in reversed(self.created):     # reverse — FK order
            try:
                self.api.delete(f"/api/{entity}/{eid}")
            except Exception as e:
                print(f"[scope] cleanup failed for {eid}: {e}")
        if exc_type is not None:
            print(f"[scope] test failed with {exc_type.__name__}: {exc_value}")
        return False       # ❗ False = exception ko aage badhne do (swallow mat karo)


with TestDataScope(api) as data:
    po = data.create_po("ACME", 125000)
    assert po["status"] == "DRAFT"
    raise AssertionError("boom")
# cleanup phir bhi chala, aur AssertionError test tak pahunchi
```

**`__exit__` ka return value — ye exactly wo trap hai jo tumhare project mein hua tha:**

| return | Matlab |
|---|---|
| `False` / `None` | Exception propagate hoga ✅ **default yahi rakho** |
| `True` | Exception **swallow** ho jaayega ❌ — test green dikhega jabki fail hua |

> **[REAL]** Tumhare project ka `try: fill(...) except: pass` wala bug **exactly yahi failure mode** hai — silent swallow. Function ne "success" report kiya jabki fill kabhi hua hi nahi, aur uske baad ki verification ek **jhooth pe** chali. Poora ek release cycle nikal gaya. `__exit__` mein `return True` likhna context-manager version of the same bug hai. Isliye rule: **`__exit__` mein kabhi bhi `return True` mat likho jab tak tumhara specific maqsad ek particular exception ko handle karna na ho — aur tab bhi `exc_type` check karke.**

**Simpler version — `contextlib`:**

```python
from contextlib import contextmanager

@contextmanager
def test_data_scope(api):
    created = []
    try:
        yield created            # jo `as` ko milega
    finally:
        for entity, eid in reversed(created):
            try: api.delete(f"/api/{entity}/{eid}")
            except Exception as e: print(f"cleanup failed: {e}")

with test_data_scope(api) as created:
    po = api.post("/api/po", {...}); created.append(("po", po["id"]))
```

`yield` ke pehle = `__enter__`, `finally` block = `__exit__`. **pytest fixtures exactly isi model pe bane hain** — `yield` ke baad ka code teardown hai.

## 7.8 Automation mein kahan lagta hai — summary

| Dunder | Automation use |
|---|---|
| `__repr__` | pytest assertion failures readable + secrets masked |
| `__eq__`/`__hash__` | Set-based comparison of API results vs expected |
| `__lt__` | Report sorting — critical failures first |
| `__len__`/`__getitem__` | Custom suite/collection objects |
| `__call__` | Configurable retry/timing policy objects |
| `__enter__`/`__exit__` | Test data scope, temporary feature flag, browser context |
| `__bool__` | `if not warnings:` on a warning-collector |

> **[REAL]** Merlin ka warning-collector fixture teardown pe toast/API warnings print karta hai. Agar wo ek class hota, `__len__` (kitni warnings), `__bool__` (koi warning hai kya), aur `__repr__` (readable dump) usko fixture ke andar se seedha assert-able bana dete: `assert not warnings, warnings`.

> **Interview answer:** Dunder methods are Python's protocol hooks — they let my own classes plug into built-in syntax like `len()`, `==`, `in`, `sorted()` and `with`. The two I care most about in test code are `__repr__` and the `__eq__`/`__hash__` pair. `__repr__` is developer-facing and unambiguous, `__str__` is user-facing and readable; pytest prints `__repr__` in assertion failures, so writing a good one turns an unreadable "object at 0x7f..." into a diff you can act on. `__eq__` and `__hash__` must be defined together because Python's contract is that equal objects hash equal — and in fact defining `__eq__` alone sets `__hash__` to None so the class becomes unhashable, which is Python failing loudly rather than corrupting your sets. I base the hash on the same immutable field the equality uses. `__enter__`/`__exit__` I use for scoped test data so cleanup runs even when the test fails, and I always return False from `__exit__` so exceptions propagate — swallowing them is how you get a suite that's green and lying.

**Cross-question: "`__str__` na ho to kya hota hai?"**
> `print()` and `str()` fall back to `__repr__`. The reverse isn't true — if you define only `__str__`, `repr()` still gives the default `<object at 0x...>`, which is what shows up in logs and assertion failures. So if I'm only going to write one, I write `__repr__`.

**Cross-question: "`__eq__` mein `NotImplemented` kyun return kiya, `False` kyun nahi?"**
> Returning `False` claims the two objects are definitely unequal. Returning `NotImplemented` tells Python "I can't decide", so it tries the reflected operation on the right-hand operand, and only if that also declines does it fall back to identity comparison. This matters for interoperability — if someone writes a comparison adapter class, returning False from my side would break it.

**Cross-question: "`__exit__` mein `True` return karne se kya hota hai?"**
> It suppresses the exception — the `with` block exits normally and the caller never sees the failure. In test code that's catastrophic: the test reports pass while the assertion actually failed. I've seen the equivalent bug from a bare `except: pass` around a fill operation — the helper reported success, and every verification after it was checking a state that was never set. It survived a full release cycle. So my rule is that suppression must be explicit, narrow, and justified in a comment, never a default.

---

# 8. staticmethod vs classmethod vs instance method

## 8.1 Kya hai — analogy pehle

Ek **restaurant** socho:

- **Instance method** = *"is table ka bill banao"* — specific table (object) ka data chahiye. `self` milta hai.
- **Classmethod** = *"is restaurant ki aaj ki total sale batao"* — kisi ek table se matlab nahi, **poore restaurant (class)** se matlab hai. `cls` milta hai.
- **Staticmethod** = *"18% GST calculate karo"* — na table se matlab, na restaurant se. Bas ek formula hai jo yahan **rakha** hua hai kyunki topically yahin belong karta hai. Na `self`, na `cls`.

## 8.2 Table

| | Instance method | `@classmethod` | `@staticmethod` |
|---|---|---|---|
| Pehla arg | `self` (object) | `cls` (class) | kuch nahi |
| Access kar sakta | instance + class state | sirf class state | kuch nahi (jo pass karo) |
| Call kaise | `obj.m()` | `Cls.m()` ya `obj.m()` | `Cls.m()` ya `obj.m()` |
| Inheritance mein `cls` | — | **subclass** milta hai (polymorphic) ✅ | — |
| Typical use | Object ka kaam | Alternate constructor, factory, class-level counter | Pure utility jo class ke topic se juda hai |
| Alternative | — | — | **Module-level function** (aksar behtar) |

## 8.3 Code — teeno ek saath

```python
class BrowserSession:
    _active_sessions = 0                       # class state

    def __init__(self, browser_name, headless, base_url):
        self.browser_name = browser_name
        self.headless = headless
        self.base_url = base_url
        BrowserSession._active_sessions += 1

    # ---------- INSTANCE METHOD ----------
    def describe(self) -> str:
        mode = "headless" if self.headless else "headed"
        return f"{self.browser_name} ({mode}) -> {self.base_url}"

    # ---------- CLASSMETHOD: alternate constructors ----------
    @classmethod
    def from_env(cls) -> "BrowserSession":
        """Env vars se session banao — CI ke liye."""
        import os
        return cls(
            browser_name=os.getenv("BROWSER", "chromium"),
            headless=os.getenv("HEADLESS", "true").lower() == "true",
            base_url=os.getenv("BASE_URL", "https://staging.merlin.app"),
        )

    @classmethod
    def from_config_file(cls, path: str) -> "BrowserSession":
        import json
        with open(path) as f:
            cfg = json.load(f)
        return cls(cfg["browser"], cfg.get("headless", True), cfg["base_url"])

    @classmethod
    def local_debug(cls) -> "BrowserSession":
        """Developer ke laptop ke liye — headed, localhost."""
        return cls("chromium", headless=False, base_url="http://localhost:3000")

    @classmethod
    def active_count(cls) -> int:
        return cls._active_sessions

    # ---------- STATICMETHOD ----------
    @staticmethod
    def is_supported(browser_name: str) -> bool:
        return browser_name.lower() in {"chromium", "firefox", "webkit"}


s1 = BrowserSession.from_env()
s2 = BrowserSession.local_debug()
print(s2.describe())                # chromium (headed) -> http://localhost:3000
print(BrowserSession.active_count())# 2
print(BrowserSession.is_supported("safari"))   # False
```

## 8.4 Classmethod as alternate constructor — kyun ye pattern important hai

Python mein **ek hi `__init__`** ho sakta hai (no overloading, section 4). Toh agar object banane ke 4 tarike hain — env se, file se, API response se, defaults se — to kya karoge?

**❌ Galat tarika — ek fat `__init__`:**

```python
class TestUser:
    def __init__(self, username=None, password=None, api_response=None,
                 from_env=False, config_path=None, role=None):
        if from_env:
            username = os.getenv("TEST_USER"); password = os.getenv("TEST_PASS")
        elif api_response:
            username = api_response["data"]["login"]; role = api_response["data"]["role"]
        elif config_path:
            ...
        # 40 lines of branching. Kaunsa combination valid hai? Koi nahi jaanta.
```

**✅ Sahi tarika — `__init__` simple, classmethods alag:**

```python
class TestUser:
    def __init__(self, username: str, password: str, role: str, user_id: str | None = None):
        """Dumb constructor — sirf assign karta hai, koi logic nahi."""
        self.username, self.password, self.role, self.user_id = username, password, role, user_id

    @classmethod
    def from_env(cls, prefix: str = "TEST") -> "TestUser":
        import os
        return cls(
            username=os.environ[f"{prefix}_USERNAME"],
            password=os.environ[f"{prefix}_PASSWORD"],
            role=os.getenv(f"{prefix}_ROLE", "PROJECT_ADMIN"),
        )

    @classmethod
    def from_api_response(cls, resp: dict) -> "TestUser":
        d = resp["data"]
        return cls(username=d["login"], password="<not-returned>", role=d["role"], user_id=d["id"])

    @classmethod
    def project_admin(cls) -> "TestUser":
        return cls("admin.qa@merlin.co", "Adm1n@123", "PROJECT_ADMIN")

    @classmethod
    def supplier(cls) -> "TestUser":
        return cls("supplier.qa@vendor.co", "Supp@123", "SUPPLIER")

    def __repr__(self):
        return f"TestUser({self.username!r}, role={self.role!r}, password='***')"

# Call sites ab SELF-DOCUMENTING hain
user  = TestUser.from_env()
admin = TestUser.project_admin()
vendor= TestUser.supplier()
```

Test file padhne wale ko turant pata chal jaata hai kya ho raha hai. Ye "named constructor" pattern hai.

## 8.5 `cls` polymorphic hota hai — ye sabse important detail

```python
class BasePage:
    def __init__(self, page): self.page = page

    @classmethod
    def create(cls, page):
        print(f"creating {cls.__name__}")
        return cls(page)                  # ❗ cls — hardcoded BasePage nahi

    @staticmethod
    def create_static(page):
        return BasePage(page)             # ❌ hardcoded — subclass ke liye galat


class InvoicePage(BasePage): pass

p1 = InvoicePage.create(page)             # creating InvoicePage
print(type(p1))                           # <class 'InvoicePage'>  ✅

p2 = InvoicePage.create_static(page)
print(type(p2))                           # <class 'BasePage'>     ❌ galat class!
```

**Yahi wajah hai ki factory methods hamesha `@classmethod` hote hain, `@staticmethod` nahi.** `cls` inheritance ke saath sahi kaam karta hai.

## 8.6 Staticmethod — kab actually use karein (aur kab module function behtar hai)

Honest truth jo interview mein bolna: **Python mein `@staticmethod` aksar zaroorat nahi hoti — module-level function behtar hota hai.**

```python
# Option A — staticmethod
class DateUtils:
    @staticmethod
    def to_ddmmyyyy(d): return d.strftime("%d/%m/%Y")

DateUtils.to_ddmmyyyy(today)      # class sirf namespace ban gayi

# Option B — module function (usually better)
# file: _helpers/dates.py
def to_ddmmyyyy(d): return d.strftime("%d/%m/%Y")

from _helpers import dates
dates.to_ddmmyyyy(today)          # module hi namespace hai
```

Java/C# mein sab kuch class mein hona padta hai, isliye wahan static methods common hain. Python mein **modules first-class namespace** hain.

**Staticmethod genuinely justified hai jab:**
1. Function class ke domain se strongly juda hai aur subclass usko override kar sakta hai (`@staticmethod` inheritable hai).
2. Class ke andar se hi mostly call hota hai aur bahar barely.
3. Ek library API surface consistent rakhni hai.

> **[REAL]** Merlin framework ne exactly Option B chuna — `_helpers/dates.py`, `_helpers/po_validators.py` jaise **modules**, na ki `DateUtils` / `POValidator` classes jinme sirf static methods hon. Wo "static-only class" pattern Java ka carry-over hai aur Python mein ek khaali wrapper hai. Interview mein ye bolna: *"I don't create utility classes with only static methods in Python — modules already give me namespacing, and a class with no state is just ceremony."*

> **Interview answer:** An instance method takes `self` and works on one object's state. A classmethod takes `cls` and works at the class level — the main use is alternate constructors, because Python allows only one `__init__` and no overloading. So instead of a constructor full of branching, I keep `__init__` dumb and add named constructors like `TestUser.from_env()` or `BrowserSession.local_debug()`, which makes call sites self-documenting. The critical detail is that `cls` is polymorphic: if a subclass calls an inherited classmethod, `cls` is the subclass, so the factory returns the right type — a staticmethod that hardcodes the base class name would silently return the wrong class. A staticmethod takes neither and is just a function namespaced inside the class. In Python I mostly avoid them, because a module-level function already gives me namespacing; a class containing only static methods is a Java habit, not a Python design.

**Cross-question: "Classmethod ko object pe call kar sakte hain?"**
> Yes — `obj.from_env()` works and `cls` is still bound to the class, not the instance. It's legal but confusing to read, so I always call classmethods on the class.

**Cross-question: "Singleton banane ke liye classmethod use karoge?"**
> You can — a classmethod holding a cached instance in a class variable is the usual `get_instance()` shape. But I'd question the singleton first. In a parallel test run a singleton is shared mutable state, and per-worker isolation is what you actually want. I'd rather use a pytest fixture with session scope, which gives me one instance per worker with automatic teardown, than a global singleton.

---

# 9. dataclass (+ NamedTuple, TypedDict, Pydantic)

## 9.1 Kya hai — analogy pehle

**Form ka printed template.** Pehle har form haath se likhna padta tha — naam ka column, address ka column, signature ki jagah. Ab pre-printed template hai — bas fields bharo. `@dataclass` wahi hai: Python khud hi `__init__`, `__repr__`, `__eq__` likh deta hai.

Dataclass ka ek hi maqsad hai: **data rakhne wali classes ka boilerplate khatam karna.**

## 9.2 Before / After

### ❌ Before — 30 lines

```python
class PurchaseOrder:
    def __init__(self, po_id, supplier, amount_paise, status="DRAFT",
                 line_items=None, created_by=None):
        self.po_id = po_id
        self.supplier = supplier
        self.amount_paise = amount_paise
        self.status = status
        self.line_items = line_items if line_items is not None else []
        self.created_by = created_by

    def __repr__(self):
        return (f"PurchaseOrder(po_id={self.po_id!r}, supplier={self.supplier!r}, "
                f"amount_paise={self.amount_paise!r}, status={self.status!r}, "
                f"line_items={self.line_items!r}, created_by={self.created_by!r})")

    def __eq__(self, other):
        if not isinstance(other, PurchaseOrder):
            return NotImplemented
        return (self.po_id, self.supplier, self.amount_paise, self.status,
                self.line_items, self.created_by) == \
               (other.po_id, other.supplier, other.amount_paise, other.status,
                other.line_items, other.created_by)
```

Aur ab socho: ek naya field add karna hai. **Teen jagah** edit karni padegi. Ek jagah bhoole → silent bug (`__eq__` mein field miss = do alag objects equal dikhenge).

### ✅ After — 8 lines

```python
from dataclasses import dataclass, field

@dataclass
class PurchaseOrder:
    po_id: str
    supplier: str
    amount_paise: int
    status: str = "DRAFT"
    line_items: list[str] = field(default_factory=list)
    created_by: str | None = None
```

`__init__`, `__repr__`, `__eq__` — teeno auto-generated. Naya field add karo, teeno khud update ho jaate hain.

```python
po = PurchaseOrder("PO-1001", "ACME Cement", 125000)
print(po)
# PurchaseOrder(po_id='PO-1001', supplier='ACME Cement', amount_paise=125000,
#               status='DRAFT', line_items=[], created_by=None)
print(po == PurchaseOrder("PO-1001", "ACME Cement", 125000))   # True
```

## 9.3 `field(default_factory=...)` — mutable default ka fix

Yaad hai section 1.3 ka mutable-default bug? Dataclass usko **error bana deta hai**:

```python
@dataclass
class Bad:
    items: list = []          # ❌ ValueError: mutable default <class 'list'> for field items
                              #    is not allowed: use default_factory
```

Python ne yahan bhi **loudly block** kiya, silently share nahi kiya. Sahi tarika:

```python
@dataclass
class TestScenario:
    name: str
    tags: list[str] = field(default_factory=list)
    metadata: dict = field(default_factory=dict)
    run_id: str = field(default_factory=lambda: __import__("uuid").uuid4().hex[:8])

a, b = TestScenario("t1"), TestScenario("t2")
a.tags.append("smoke")
print(b.tags)            # []  ✅ alag list
print(a.run_id, b.run_id)# alag run ids
```

`default_factory` ka callable **har instance ke liye alag** chalta hai. Isliye unique-per-test data (parallel safety, section 18) ke liye perfect hai.

## 9.4 `frozen=True` — immutability

```python
@dataclass(frozen=True)
class TestUser:
    username: str
    password: str
    role: str

u = TestUser("ritik.qa", "pass", "ADMIN")
u.role = "SUPER_ADMIN"
# ❌ dataclasses.FrozenInstanceError: cannot assign to field 'role'
```

**Do bade faayde:**

1. **Hashable free mein.** `frozen=True` + `eq=True` (default) → Python `__hash__` bhi generate kar deta hai. Ab `set` aur `dict` key mein daal sakte ho.
2. **Shared fixture corruption impossible.** Ye automation ka killer use case:

```python
@pytest.fixture(scope="session")
def admin_user():
    return TestUser("admin.qa@merlin.co", "Adm1n@123", "PROJECT_ADMIN")

def test_a(admin_user):
    admin_user.role = "SUPPLIER"     # ❌ FrozenInstanceError — test_b bach gaya
    ...

def test_b(admin_user):
    assert admin_user.role == "PROJECT_ADMIN"    # ye pehle silently fail ho jaata
```

Session-scoped fixture **saare tests share karte hain**. Agar mutable hai, ek test dusre ko corrupt kar dega — aur ye bug order-dependent hoga, matlab reproduce karna nightmare. `frozen=True` isko **compile-time-ish error** bana deta hai.

**Frozen object ko "modify" karna — `replace`:**

```python
from dataclasses import replace
supplier_user = replace(admin_user, role="SUPPLIER")   # naya object, original safe
```

## 9.5 Baaki useful parameters

```python
@dataclass(frozen=True, order=True, slots=True, kw_only=True)
class TestResult:
    priority: int              # order=True -> sorting isi order mein fields se
    name: str = field(compare=False)          # comparison se exclude
    duration_s: float = field(compare=False, repr=True)
    password: str = field(default="", repr=False)   # repr mein NAHI dikhega (secret!)
```

| Param | Kya karta hai | Kab use |
|---|---|---|
| `frozen=True` | Immutable | Shared fixtures, dict keys, config |
| `order=True` | `__lt__ __le__ __gt__ __ge__` generate | Sorting results |
| `slots=True` | `__dict__` hatakar memory bachata, typo pe error | Bade collections (10k+ objects) |
| `kw_only=True` | Sab args keyword-only | Bahut fields hon, positional confusing ho |
| `field(repr=False)` | Us field ko repr se hatao | **Passwords, tokens** |
| `field(compare=False)` | Equality se hatao | timestamps, ids jo har run mein badalte |
| `__post_init__` | init ke baad validation | Derived fields, invariants |

```python
@dataclass
class PurchaseOrder:
    po_id: str
    amount_paise: int
    line_items: list[dict] = field(default_factory=list)

    def __post_init__(self):
        if self.amount_paise < 0:
            raise ValueError(f"amount cannot be negative: {self.amount_paise}")
        if not self.po_id.startswith("PO-"):
            raise ValueError(f"malformed po_id: {self.po_id}")

    @property
    def amount_rupees(self) -> float:
        return self.amount_paise / 100
```

> **[REAL]** `kw_only=True` bilkul wahi philosophy hai jo tumne `def click(locator, *, timeout_ms: int = 5000)` mein use ki — `*` ke baad sab keyword-only. Call site padhne se hi pata chale ki 5000 ka matlab kya hai. `click(loc, 5000)` ambiguous hai; `click(loc, timeout_ms=5000)` self-documenting. Dataclass mein `kw_only=True` isi rule ko poori class pe apply kar deta hai. Interview mein ye connection banana — *"I apply the same keyword-only discipline in my helpers that `kw_only` dataclasses give you: at a call site, a bare number is a bug waiting to happen."*

## 9.6 dataclass vs NamedTuple vs TypedDict vs Pydantic vs plain class

```python
# ---------- 1. plain class ----------
class User:
    def __init__(self, name, role):
        self.name, self.role = name, role
# Behaviour-heavy classes ke liye. Data holder ke liye boilerplate.

# ---------- 2. NamedTuple ----------
from typing import NamedTuple
class Point(NamedTuple):
    x: int
    y: int
p = Point(1, 2)
print(p.x, p[0])            # dono kaam karte — tuple bhi hai
# a, b = p                  # unpacking free
# IMMUTABLE by default, lightweight, tuple-compatible

# ---------- 3. dataclass ----------
@dataclass
class Order:
    id: str
    total: int
# Mutable by default, methods add kar sakte, inheritance, __post_init__

# ---------- 4. TypedDict ----------
from typing import TypedDict
class POPayload(TypedDict):
    supplier: str
    amount: int
payload: POPayload = {"supplier": "ACME", "amount": 125000}
# Ye RUNTIME pe plain dict hi hai. Sirf type checker ke liye shape.
# JSON request/response bodies ke liye perfect.

# ---------- 5. Pydantic ----------
from pydantic import BaseModel, Field, field_validator
class POResponse(BaseModel):
    id: str
    supplier: str
    amount: int = Field(gt=0)
    status: str

    @field_validator("status")
    @classmethod
    def known_status(cls, v):
        allowed = {"DRAFT", "PENDING_APPROVAL", "APPROVED", "REJECTED"}
        if v not in allowed:
            raise ValueError(f"unknown status {v!r}; API contract changed?")
        return v

po = POResponse(**api_response.json())    # RUNTIME validation + coercion
```

| | Mutable | Runtime validation | Methods | Best for |
|---|---|---|---|---|
| plain class | Haan | Manual | Haan | Behaviour-heavy objects |
| `NamedTuple` | Nahi | Nahi | Haan (limited) | Chhote immutable value objects, tuple compat |
| `@dataclass` | Haan (`frozen` se nahi) | `__post_init__` se manual | Haan | **Default choice** — test data models, config |
| `TypedDict` | Haan (dict hai) | Nahi (static only) | Nahi | JSON payload shapes |
| Pydantic | Configurable | **Haan, built-in** | Haan | **API contract validation**, untrusted input |

**Decision rule jo bolna:**

```
Data bahar se aa raha (API/JSON/user)?  -> Pydantic (runtime validation chahiye)
Sirf dict ka shape document karna hai?  -> TypedDict
Immutable + tuple jaisa chahiye?        -> NamedTuple
Andar ka data model, methods bhi honge? -> dataclass  <- 90% cases
Behaviour hi behaviour, data kam?       -> plain class
```

## 9.7 Automation mein kahan lagta hai

```python
@dataclass(frozen=True)
class TestUser:                    # session fixtures — immutable, no cross-test corruption
    username: str
    password: str = field(repr=False)
    role: str

@dataclass
class POBuilder:                   # test data builder (section 11)
    supplier: str = "ACME Cement"
    amount_paise: int = 125000
    line_items: list = field(default_factory=list)

@dataclass(frozen=True)
class EnvConfig:                   # config — frozen so nobody mutates at runtime
    name: str
    base_url: str
    api_url: str
    timeout_ms: int = 5000

class POApiResponse(BaseModel):    # API response — Pydantic, contract validation
    id: str
    status: str
```

**Pydantic for API responses is a genuine contract test.** Agar backend ne field ka type badal diya (`amount: int` → `amount: str`), tumhara test **schema parse pe** fail hoga clear message ke saath, na ki 15 lines baad ek confusing `TypeError` pe.

> **Interview answer:** A dataclass removes the boilerplate for classes whose job is holding data — Python generates `__init__`, `__repr__` and `__eq__` from the field annotations, so adding a field updates all three instead of me editing three places and forgetting one. For mutable defaults I use `field(default_factory=list)`; a plain `[]` is actually rejected with an error, which is Python fixing the classic shared-mutable-default bug. I use `frozen=True` for anything shared — session-scoped fixture users, config objects — because a session fixture is shared across every test, and one test mutating it creates an order-dependent failure that's miserable to reproduce. Frozen also makes the class hashable, so it works as a dict key or in a set. As for the alternatives: TypedDict just documents a dict's shape for the type checker with no runtime effect, NamedTuple is an immutable tuple-compatible value object, and Pydantic adds runtime validation — which is what I want at the API boundary, because a schema mismatch then fails at parse time with a clear message instead of surfacing as a confusing error deep in the test.

**Cross-question: "dataclass inheritance mein koi gotcha?"**
> Yes — a subclass can't add a field without a default after a parent field that has one, because the generated `__init__` would have a non-default argument following a default one. The fix is `kw_only=True`, which removes positional ordering entirely, and I generally prefer keyword-only dataclasses anyway for readability.

**Cross-question: "`frozen=True` deep immutability deta hai?"**
> No, it's shallow. The field bindings are frozen but a mutable object inside a field is still mutable — a frozen dataclass holding a list means you can't reassign the list, but you can still append to it. For genuine immutability I use tuples instead of lists in frozen dataclasses, and I'd note that `frozen=True` also silently makes `__post_init__` mutations fail, so derived fields need `object.__setattr__`.

---

# 10. SOLID Principles

> **Framing:** SOLID definitions har koi ratta maar leta hai. Senior banne ke liye do cheezein chahiye — (a) **apne code se example**, (b) **"ye principle todne ka cost kya hai"**. Neeche har principle ka BAD aur GOOD dono test-automation ka hai, shapes/animals ka nahi.
>
> **Ek line mein poora SOLID:** *"Make change cheap and local."* Har principle isi ka ek angle hai.

| | Naam | Ek line | Todne ka symptom |
|---|---|---|---|
| **S** | Single Responsibility | Ek class, badalne ki **ek** wajah | Ek chhote change pe 8 test files touch karni padin |
| **O** | Open/Closed | Extension ke liye khula, modification ke liye band | Naya browser/env add karne pe purani working file edit karni padi |
| **L** | Liskov Substitution | Subclass base ki jagah bina toote chale | `isinstance()` checks phailne lage |
| **I** | Interface Segregation | Client ko wo method na jhelne pade jo use nahi karta | 40-method BasePage jisme se har page 4 hi use karta hai |
| **D** | Dependency Inversion | High-level module abstraction pe depend kare, concrete pe nahi | Unit test likhne ke liye real network chahiye |

---

## 10.1 S — Single Responsibility Principle

**Definition:** Ek class ke paas badalne ki **sirf ek wajah** honi chahiye. "Ek kaam" nahi — **ek stakeholder / ek reason to change.**

**Analogy:** Ek dukaan jahan chai, mobile repair aur photocopy teeno hote hain. Chai ka rate badla to poori dukaan band karni padegi. Teen alag dukaanein hoti to sirf ek band hoti.

### ❌ BAD — page object jo API bhi karta hai aur assert bhi

```python
class PurchaseOrderPage:
    def __init__(self, page, api_base, db_conn, slack_webhook):
        self.page = page
        self.api_base = api_base
        self.db = db_conn
        self.slack = slack_webhook

    def create_po_ui(self, supplier, amount):
        self.page.get_by_label("Supplier").fill(supplier)
        self.page.get_by_label("Amount").fill(str(amount))
        self.page.get_by_role("button", name="Submit").click()

    # ---- responsibility 2: HTTP ----
    def seed_po_via_api(self, supplier, amount):
        import requests
        return requests.post(f"{self.api_base}/po",
                             json={"supplier": supplier, "amount": amount},
                             timeout=30).json()

    # ---- responsibility 3: database ----
    def get_po_from_db(self, po_id):
        cur = self.db.cursor()
        cur.execute("SELECT status, total FROM purchase_orders WHERE id = %s", (po_id,))
        return cur.fetchone()

    # ---- responsibility 4: assertions ----
    def verify_po_approved(self, po_id):
        assert self.page.get_by_test_id("po-status").inner_text() == "APPROVED"
        assert self.get_po_from_db(po_id)[0] == "APPROVED"

    # ---- responsibility 5: reporting ----
    def notify_failure(self, msg):
        import requests
        requests.post(self.slack, json={"text": msg})

    # ---- responsibility 6: test data ----
    def random_supplier_name(self):
        import random, string
        return "SUP-" + "".join(random.choices(string.ascii_uppercase, k=6))
```

**Ye class kitni wajah se badlegi?** UI badla, API contract badla, DB schema badla, assertion policy badli, Slack ka format badla, data generation strategy badli — **6 alag teams isko touch karengi.** Merge conflicts guaranteed. Aur worst: iska koi bhi unit test likhne ke liye tumhe browser + network + database teeno chahiye.

### ✅ GOOD — har responsibility alag

```python
# ---- 1. UI only ----
class PurchaseOrderPage:
    def __init__(self, page):
        self.page = page

    def create(self, supplier: str, amount: int) -> "PurchaseOrderPage":
        self.page.get_by_label("Supplier").fill(supplier)
        self.page.get_by_label("Amount").fill(str(amount))
        self.page.get_by_role("button", name="Submit").click()
        return self

    @property
    def status(self) -> str:
        return self.page.get_by_test_id("po-status").inner_text().strip()


# ---- 2. HTTP only ----
class POApiClient:
    def __init__(self, session, base_url):
        self.session, self.base_url = session, base_url

    def create(self, supplier, amount) -> dict:
        r = self.session.post(f"{self.base_url}/po",
                              json={"supplier": supplier, "amount": amount}, timeout=30)
        r.raise_for_status()
        return r.json()


# ---- 3. DB only ----
class PORepository:
    def __init__(self, conn): self.conn = conn
    def status_of(self, po_id) -> str:
        with self.conn.cursor() as cur:
            cur.execute("SELECT status FROM purchase_orders WHERE id = %s", (po_id,))
            return cur.fetchone()[0]


# ---- 4. Assertions only (_helpers/invariant_assertions.py) ----
def assert_po_approved_everywhere(ui: PurchaseOrderPage, repo: PORepository, po_id: str):
    assert ui.status == "APPROVED", f"UI shows {ui.status}"
    assert repo.status_of(po_id) == "APPROVED", "DB not updated — UI/DB divergence"


# ---- 5. Test data only ----
def unique_supplier_name(prefix="SUP") -> str:
    import uuid
    return f"{prefix}-{uuid.uuid4().hex[:8].upper()}"


# ---- test file ab sirf orchestration hai ----
def test_po_approval(page, po_api, po_repo):
    supplier = unique_supplier_name()
    po = po_api.create(supplier, 125000)
    ui = PurchaseOrderPage(page)
    ui.page.goto(f"/purchase-orders/{po['id']}")
    ui.page.get_by_role("button", name="Approve").click()
    assert_po_approved_everywhere(ui, po_repo, po["id"])
```

**Ab kya badla:** UI change → sirf `PurchaseOrderPage`. DB schema change → sirf `PORepository`. Har class alag se test ho sakti hai.

> **[REAL]** Merlin ka framework SRP ko module level pe follow karta hai — `interactions.py` (kaise interact karna), `po_validators.py` (kya valid hai), `invariant_assertions.py` (kya hamesha sach hona chahiye), `api_clients.py` (HTTP), `config.py` (env). Ye separation deliberately kiya gaya. Interview mein bolna: *"My helper modules are split by reason-to-change, not by convenience. When our PO validation rules changed, exactly one file was touched."*

**SRP ka common galat interpretation:** "Ek class mein ek hi method ho." Nahi. `PurchaseOrderPage` mein 15 methods ho sakte hain — jab tak sab **UI interaction** hain, ek hi reason to change hai.

> **Interview answer:** Single Responsibility means a class should have one reason to change, defined by who asks for the change. The classic violation in test automation is a page object that also makes API calls, queries the database, contains assertions, and generates test data. That class now changes when the UI changes, when the API contract changes, when the schema changes, and when the assertion policy changes — four different teams touching one file, and you can't unit test any part of it without a browser and a network. I split it into a page object that only knows the UI, an API client that only knows HTTP, a repository that only knows the database, and separate assertion helpers. The test then just orchestrates them. The practical payoff is blast radius: when our PO validation rules changed, exactly one module was modified.

**Cross-question: "SRP follow karne se class explosion nahi ho jaayega?"**
> It can if you apply it dogmatically at method level. The unit of responsibility is the reason to change, not the method count — a page object with fifteen UI methods still has one reason to change. I only split when I can name two distinct change drivers with different owners. If I can't name them, splitting is premature.

---

## 10.2 O — Open/Closed Principle

**Definition:** Software entities extension ke liye **khuli**, modification ke liye **band** honi chahiye. Naya behaviour **naya code likhkar** aaye, purana kaam karta hua code edit karke nahi.

**Analogy:** Power strip. Naya device lagana hai to strip mein plug kar do — **wiring nahi kholni**. Agar har naye device ke liye deewar todni pade, ye OCP violation hai.

**Kyun matter karta hai:** Working code ko edit karna = regression risk. Naya code add karna = risk sirf naye code tak.

### ❌ BAD — naya browser add karne ke liye purana function edit karna

```python
def launch_browser(playwright, browser_name, headless=True):
    if browser_name == "chromium":
        return playwright.chromium.launch(headless=headless)
    elif browser_name == "firefox":
        return playwright.firefox.launch(headless=headless)
    elif browser_name == "webkit":
        return playwright.webkit.launch(headless=headless)
    elif browser_name == "edge":                        # naya requirement
        return playwright.chromium.launch(channel="msedge", headless=headless)
    elif browser_name == "chrome-mobile":               # aur naya
        ...
    else:
        raise ValueError(f"Unknown browser: {browser_name}")
```

Har naye browser pe ye **tested, working function** edit hota hai. Ek typo saare browsers tod deta hai. Function badhta jaata hai.

### ✅ GOOD — registry + strategy, purani file untouched

```python
from typing import Protocol, Callable
from playwright.sync_api import Browser, Playwright

# ---- core: ye file ab kabhi nahi badlegi ----
class BrowserLauncher(Protocol):
    def launch(self, pw: Playwright, *, headless: bool) -> Browser: ...

_REGISTRY: dict[str, BrowserLauncher] = {}

def register_browser(name: str):
    def deco(cls):
        _REGISTRY[name] = cls()
        return cls
    return deco

def launch_browser(pw: Playwright, name: str, *, headless: bool = True) -> Browser:
    try:
        return _REGISTRY[name].launch(pw, headless=headless)
    except KeyError:
        raise ValueError(f"Unknown browser {name!r}. Registered: {sorted(_REGISTRY)}") from None


# ---- extensions: naya browser = naya block, koi purana code touch nahi ----
@register_browser("chromium")
class ChromiumLauncher:
    def launch(self, pw, *, headless): return pw.chromium.launch(headless=headless)

@register_browser("firefox")
class FirefoxLauncher:
    def launch(self, pw, *, headless): return pw.firefox.launch(headless=headless)

@register_browser("webkit")
class WebkitLauncher:
    def launch(self, pw, *, headless): return pw.webkit.launch(headless=headless)

# ---- 6 mahine baad, naya requirement: sirf ye ADD hua ----
@register_browser("edge")
class EdgeLauncher:
    def launch(self, pw, *, headless):
        return pw.chromium.launch(channel="msedge", headless=headless)

@register_browser("chromium-ci")
class ChromiumCILauncher:
    """CI ke liye — sandbox off, shm size fix (docker)."""
    def launch(self, pw, *, headless):
        return pw.chromium.launch(headless=True,
                                  args=["--no-sandbox", "--disable-dev-shm-usage"])
```

`launch_browser` ka body **kabhi nahi badla**. Naya browser = naya class. Ye OCP hai.

**Same pattern ke aur automation examples:**
- Naya report format → naya `Reporter` class, `run_reports()` untouched.
- Naya environment → config dict mein entry, `get_config()` untouched.
- Naya assertion type → naya validator function, registry mein register.

**Warning — over-engineering ka risk:** OCP tab lagao jab **extension point genuinely predict ho**. Agar tumhe pata hai ki browsers add honge, registry banao. Agar ek hi browser hamesha rahega, `if/else` theek hai. **YAGNI vs OCP ka balance senior judgement hai.** Interview mein ye nuance bolna bahut achha lagta hai.

> **Interview answer:** Open/Closed means I should be able to add new behaviour by adding new code, not by editing code that already works and is already tested. The classic violation in a framework is a `launch_browser` function that's a growing if/elif chain — every new browser edits a function every test depends on, so a typo there breaks all of them. I replace it with a registry: a small protocol defining what a launcher does, a dictionary mapping name to implementation, and a decorator that registers new ones. Adding Edge or a CI-specific Chromium config becomes a new class in a new place, and the dispatch function never changes again. I'd add one caveat though — OCP has a cost in indirection, so I only introduce the extension point where I can name a concrete axis of change I actually expect. Applying it everywhere is speculative generality.

**Cross-question: "Registry pattern se debugging mushkil nahi ho jaati?"**
> It adds one level of indirection, yes. I mitigate it two ways: the error message on an unknown key lists every registered name, so a typo is self-diagnosing, and registration happens at import time in one obvious module so there's a single place to look. The trade is real — I accept it where the extension axis is real, and I don't where it isn't.

---

## 10.3 L — Liskov Substitution Principle

**Definition:** Agar `S` `T` ka subtype hai, to program mein `T` ke objects ko `S` se replace karne pe **program ka correctness nahi tootna chahiye**.

**Aasan bhasha:** Subclass ko base ka **vaada nibhana** padega. Aur zyada demand mat karo, aur kam deliver mat karo.

**Analogy:** Tumne debit card ke liye apply kiya. Bank ne card diya jo sirf Tuesday ko chalta hai. Technically "card" hai, lekin **contract tod diya**. Har wo jagah jahan card chalna chahiye tha, ab exception handle karna padega.

### ❌ BAD — subclass base ka contract todta hai

```python
class BasePage:
    def __init__(self, page):
        self.page = page

    def open(self) -> "BasePage":
        """Contract: page kholta hai aur SELF return karta hai. Kabhi exception nahi
        agar user logged in hai."""
        self.page.goto(self.path)
        return self

    def is_loaded(self) -> bool:
        """Contract: bool return karta hai. Kabhi throw nahi karta."""
        return self.page.locator("main").is_visible()


class ReportsPage(BasePage):
    path = "/reports"

    def open(self):
        raise NotImplementedError("Reports page needs a date range — use open_with_range()")
        # ❌ base ne kaha "open kaam karega". Subclass ne mana kar diya.

    def is_loaded(self):
        if not self.page.locator("#chart").is_visible():
            raise RuntimeError("chart missing")      # ❌ base ne kaha "bool return karega"
        return True

    def open_with_range(self, start, end):
        self.page.goto(f"{self.path}?from={start}&to={end}")
        return self


class ReadOnlyDashboardPage(BasePage):
    path = "/dashboard"
    def open(self):
        super().open()
        return None            # ❌ base ne self return karne ka vaada kiya tha
```

**Ab kya hota hai:**

```python
def warm_up_all_pages(pages: list[BasePage]):
    for p in pages:
        p.open().is_loaded()          # ye generic code ab RANDOMLY todta hai
```

Aur fix karne ke liye log ye likhte hain — **jo LSP violation ka classic symptom hai:**

```python
def warm_up_all_pages(pages):
    for p in pages:
        if isinstance(p, ReportsPage):            # ❌ smell
            p.open_with_range(default_start, default_end)
        elif isinstance(p, ReadOnlyDashboardPage):
            p.open()                               # return ignore karo
        else:
            p.open().is_loaded()
```

**`isinstance` checks phailna = LSP toota hua hai.** Polymorphism ka poora point khatam.

### ✅ GOOD — contract honestly model karo

```python
from abc import ABC, abstractmethod

class Page(ABC):
    """Contract: har page kholne layak hai. Parameters page-specific ho sakte hain,
    lekin har page ke paas ek 'sensible default' open() hona chahiye."""

    @abstractmethod
    def open(self) -> "Page": ...

    @abstractmethod
    def is_loaded(self) -> bool:
        """Kabhi throw nahi — sirf True/False."""


class ReportsPage(Page):
    path = "/reports"
    DEFAULT_RANGE = ("2026-01-01", "2026-12-31")

    def __init__(self, page, date_range=None):
        self.page = page
        self.date_range = date_range or self.DEFAULT_RANGE   # requirement ko OPTIONAL banao

    def open(self) -> "ReportsPage":
        start, end = self.date_range
        self.page.goto(f"{self.path}?from={start}&to={end}")
        return self                                   # contract poora

    def is_loaded(self) -> bool:
        return self.page.locator("#chart").is_visible()   # throw nahi, bool


class DashboardPage(Page):
    path = "/dashboard"
    def __init__(self, page): self.page = page
    def open(self) -> "DashboardPage":
        self.page.goto(self.path)
        return self
    def is_loaded(self) -> bool:
        return self.page.get_by_test_id("kpi-cards").is_visible()


def warm_up_all_pages(pages: list[Page]):
    for p in pages:
        assert p.open().is_loaded()     # ✅ koi isinstance nahi, sab predictable
```

### LSP ke 4 concrete rules (ye yaad rakho, ye poochte hain)

| Rule | Matlab | Violation |
|---|---|---|
| **Preconditions weaken ya same** | Subclass base se **kam** demand kare | Base `open()` bina arg chalta tha, subclass ne date range mandatory kar diya |
| **Postconditions strengthen ya same** | Subclass base se **zyada ya barabar** de | Base `self` return karta tha, subclass `None` deta hai |
| **Invariants preserve** | Base ke rules subclass mein bhi sach | Base guarantee: "`self.page` kabhi None nahi". Subclass ne None set kar diya |
| **No new exceptions** | Subclass naye checked exceptions na phenke | Base `bool` deta tha, subclass `RuntimeError` phenkta hai |

**Classic textbook example jo pooch lete hain — Square/Rectangle:** `Square` `Rectangle` se inherit kare, to `set_width(5); set_height(4); assert area == 20` toot jaata hai kyunki square dono ek saath badalta hai. Ye **behavioural** violation hai, structural nahi — isiliye compiler nahi pakadta.

> **Interview answer:** Liskov says a subclass must be usable anywhere the base class is expected without breaking correctness. The practical rules are: don't strengthen preconditions, don't weaken postconditions, preserve invariants, and don't throw new exception types. In a page hierarchy, the violation I've seen is a ReportsPage that inherits BasePage but makes `open()` raise because it needs a date range — now generic code that iterates over pages and calls `open()` breaks unpredictably. The tell-tale symptom is `isinstance` checks appearing in the calling code to work around specific subclasses; that means polymorphism has stopped working and the hierarchy is wrong. The fix isn't more branching — it's making the extra requirement optional with a sensible default so the base contract still holds, or admitting it isn't an is-a and using composition instead.

**Cross-question: "LSP violation compiler kyun nahi pakadta?"**
> Because it's about behaviour, not signatures. A subclass can have a perfectly type-compatible signature and still violate the contract — returning None where the base returned self, or raising where the base promised a bool. Type checkers catch some of it, like an incompatible return annotation, but semantic contracts like "never throws" or "always returns self" aren't in the type system. That's why I write them into docstrings and cover them with a shared contract test that runs against every subclass.

**Cross-question: "Contract test kaise likhoge?"**
> A parametrised test that takes every registered page class and asserts the base contract: `open()` returns an instance of the same class, `is_loaded()` returns a bool and doesn't raise, and the object is usable afterwards. It's about fifteen lines and it catches Liskov violations the moment someone adds a new subclass.

---

## 10.4 I — Interface Segregation Principle

**Definition:** Client ko aise interface pe depend karne pe majboor mat karo jo wo use nahi karta. **Kai chhote specific interfaces** > ek bada general interface.

**Analogy:** TV ka remote jisme 60 button hain lekin tum sirf 5 use karte ho. Har baar volume badhaane ke liye 60 buttons mein se dhoondhna padta hai. Chhota remote (5 button) actually behtar hai.

### ❌ BAD — fat BasePage

```python
class BasePage:
    """Sab kuch yahin daal do."""
    def __init__(self, page): self.page = page

    # navigation
    def open(self): ...
    def go_back(self): ...
    def refresh(self): ...
    # table
    def get_table_rows(self): ...
    def sort_by_column(self, col): ...
    def filter_table(self, **kw): ...
    def get_cell(self, row, col): ...
    def select_all_rows(self): ...
    # pagination
    def next_page(self): ...
    def prev_page(self): ...
    def go_to_page(self, n): ...
    def set_page_size(self, n): ...
    # export
    def export_csv(self): ...
    def export_pdf(self): ...
    def export_excel(self): ...
    # file upload
    def upload_file(self, path): ...
    def upload_multiple(self, paths): ...
    # modal
    def confirm_modal(self): ...
    def dismiss_modal(self): ...
    # form
    def submit_form(self): ...
    def reset_form(self): ...
    def validate_required_fields(self): ...
    # ... 20 aur


class LoginPage(BasePage):
    """Login page ke paas na table hai, na pagination, na export, na upload."""
    def login(self, u, p): ...
```

**Problems:**
1. `login_page.export_pdf()` — IDE suggest karta hai. Koi junior call kar dega. Runtime pe cryptic failure.
2. `LoginPage` ka public surface 40 methods hai jismein se 1 relevant hai.
3. Table logic mein change → `LoginPage` ke tests bhi re-run/re-verify karne padte hain (unnecessary coupling).
4. Kisi ne `get_table_rows` mein bug fix kiya → poora suite ka regression risk.

### ✅ GOOD — chhote focused mixins/protocols

```python
from typing import Protocol

# ---- capabilities, alag-alag ----
class Navigable(Protocol):
    def open(self): ...
    def is_loaded(self) -> bool: ...

class TableMixin:
    """Sirf un pages ke liye jinme table hai."""
    def rows(self):
        return self.page.locator("table tbody tr")
    def row_count(self) -> int:
        return self.rows().count()
    def sort_by(self, column: str):
        self.page.get_by_role("columnheader", name=column).click()
        return self
    def cell(self, row: int, col: str) -> str:
        return self.rows().nth(row).get_by_test_id(f"cell-{col}").inner_text()

class PaginationMixin:
    def next_page(self):
        self.page.get_by_role("button", name="Next").click()
        return self
    def set_page_size(self, n: int):
        self.page.get_by_label("Rows per page").select_option(str(n))
        return self

class ExportMixin:
    def export(self, fmt: str = "csv"):
        with self.page.expect_download() as dl:
            self.page.get_by_role("button", name="Export").click()
            self.page.get_by_role("menuitem", name=fmt.upper()).click()
        return dl.value

class UploadMixin:
    def upload(self, *paths: str):
        self.page.get_by_label("Attachments").set_input_files(list(paths))
        return self


# ---- minimal base: sirf wo jo SACH mein har page ke paas hai ----
class BasePage:
    def __init__(self, page):
        self.page = page
    def open(self):
        self.page.goto(self.path)
        return self
    def screenshot(self, name):
        self.page.screenshot(path=f"artifacts/{name}.png")


# ---- har page utna hi leta hai jitna chahiye ----
class LoginPage(BasePage):
    path = "/login"
    def login(self, u, p): ...
    # 3 methods total. Clean.

class POListPage(BasePage, TableMixin, PaginationMixin, ExportMixin):
    path = "/purchase-orders"

class InvoiceDetailPage(BasePage, UploadMixin):
    path = "/invoices"
```

Ab `LoginPage` ke paas `export_pdf` hai hi nahi. **Galat call likhna impossible ho gaya**, IDE hi nahi dikhayega.

> **[REAL]** Merlin ka functional-module approach ISP ka natural solution hai. Test file `from _helpers.interactions import click, fill` likhti hai — sirf wahi 2 functions import hote hain. Ek fat `BasePage` inherit karke 40 methods mil jaana ka ulta. Interview mein bolna: *"Explicit imports are interface segregation enforced by the language — a module that imports two helpers has a dependency surface of exactly two functions."*

> **Interview answer:** Interface Segregation says clients shouldn't be forced to depend on methods they don't use. In test frameworks the classic violation is the god BasePage — forty methods covering tables, pagination, export, upload, modals — inherited by every page including LoginPage, which has none of those things. The costs are real: autocomplete suggests `export_pdf()` on a login page so someone eventually calls it, the public surface is meaningless, and a change to table logic forces you to re-verify pages that have no table. My fix is a minimal base with only what's genuinely universal — navigation, screenshot, readiness — plus small focused mixins that pages opt into. A page then has exactly the capabilities it actually has, and calling something meaningless becomes impossible rather than merely discouraged.

**Cross-question: "Mixins bhi to multiple inheritance hai — MRO problem nahi aayegi?"**
> It can, so I constrain them: a mixin inherits only from object, never declares `__init__` state the page doesn't know about, and uses distinct method names. Under those rules the MRO is linear and predictable. If a mixin starts needing constructor participation, that's my signal it should be a composed collaborator instead of a mixin.

---

## 10.5 D — Dependency Inversion Principle

**Definition:**
1. High-level modules low-level modules pe depend na karein. **Dono abstraction pe depend karein.**
2. Abstractions details pe depend na karein. **Details abstraction pe depend karein.**

**Analogy:** Laptop charger. Laptop "220V Indian socket" pe depend nahi karta — wo **USB-C standard** pe depend karta hai. Isliye wahi laptop US, UK, Japan kahin bhi chalta hai — bas adapter badalta hai. Agar laptop directly Indian socket pe wired hota, har country ke liye naya laptop chahiye hota.

### ❌ BAD — test directly `requests` pe depend karta hai

```python
import requests

class POWorkflow:
    """High-level business logic."""
    def __init__(self, base_url, token):
        self.base_url = base_url
        self.token = token

    def create_and_approve(self, supplier, amount):
        # ❌ concrete library hardcoded
        r = requests.post(f"{self.base_url}/po",
                          headers={"Authorization": f"Bearer {self.token}"},
                          json={"supplier": supplier, "amount": amount},
                          timeout=30)
        r.raise_for_status()
        po = r.json()

        r2 = requests.post(f"{self.base_url}/po/{po['id']}/approve",
                           headers={"Authorization": f"Bearer {self.token}"}, timeout=30)
        r2.raise_for_status()
        return r2.json()
```

**Kya toota:**
- Is business logic ka unit test likhne ke liye **real network** chahiye, ya `requests` ko monkeypatch karna padega (fragile).
- `requests` se `httpx` ya `aiohttp` pe move karna ho → business logic edit.
- Retry, logging, correlation ID header add karna ho → har call site pe.
- Contract-test mode (recorded responses) impossible.

### ✅ GOOD — abstraction pe depend karo, implementation inject karo

```python
from typing import Protocol, Any

# ---- ABSTRACTION (high-level module ise OWN karta hai) ----
class ApiClient(Protocol):
    def post(self, path: str, json: dict | None = None) -> dict[str, Any]: ...
    def get(self, path: str) -> dict[str, Any]: ...


# ---- HIGH-LEVEL: sirf abstraction jaanta hai ----
class POWorkflow:
    def __init__(self, api: ApiClient):        # dependency INJECTED
        self.api = api

    def create_and_approve(self, supplier: str, amount: int) -> dict:
        po = self.api.post("/po", json={"supplier": supplier, "amount": amount})
        return self.api.post(f"/po/{po['id']}/approve")


# ---- LOW-LEVEL implementations: ye abstraction ko satisfy karte hain ----
class RequestsClient:
    def __init__(self, base_url, token, timeout=30):
        import requests
        self.s = requests.Session()
        self.s.headers.update({"Authorization": f"Bearer {token}"})
        self.base_url, self.timeout = base_url, timeout

    def post(self, path, json=None):
        r = self.s.post(f"{self.base_url}{path}", json=json, timeout=self.timeout)
        r.raise_for_status()
        return r.json() if r.content else {}

    def get(self, path):
        r = self.s.get(f"{self.base_url}{path}", timeout=self.timeout)
        r.raise_for_status()
        return r.json()


class PlaywrightApiClient:
    """Browser context ke cookies reuse karta hai — UI+API mixed tests ke liye."""
    def __init__(self, request_context, base_url):
        self.rc, self.base_url = request_context, base_url
    def post(self, path, json=None):
        return self.rc.post(f"{self.base_url}{path}", data=json).json()
    def get(self, path):
        return self.rc.get(f"{self.base_url}{path}").json()


class RecordedClient:
    """Contract tests — recorded fixtures se, network bilkul nahi."""
    def __init__(self, recordings: dict): self.recordings = recordings
    def post(self, path, json=None): return self.recordings[("POST", path)]
    def get(self, path): return self.recordings[("GET", path)]


class FakeClient:
    """Unit tests — in-memory, assertions on calls."""
    def __init__(self): self.calls, self.next_id = [], 1000
    def post(self, path, json=None):
        self.calls.append(("POST", path, json))
        if path == "/po":
            self.next_id += 1
            return {"id": f"PO-{self.next_id}", "status": "DRAFT"}
        return {"status": "APPROVED"}
    def get(self, path):
        self.calls.append(("GET", path)); return {}


# ---- ab dependency wiring conftest mein hoti hai ----
def test_workflow_calls_approve_after_create():
    fake = FakeClient()
    POWorkflow(fake).create_and_approve("ACME", 125000)
    assert [c[0:2] for c in fake.calls] == [
        ("POST", "/po"),
        ("POST", "/po/PO-1001/approve"),
    ]
    # 2ms, no network, no browser
```

**Dependency direction ka diagram:**

```
❌ BAD                              ✅ GOOD
                                     
POWorkflow                          POWorkflow
    │ depends on                        │ depends on
    v                                   v
requests (concrete)                 ApiClient (Protocol)  <- abstraction
                                        ^
                                        │ implements
                        ┌───────────────┼───────────────┐
                   RequestsClient  PlaywrightClient  FakeClient
                   
        Dependency arrow INVERT ho gaya — low-level ab abstraction ki taraf point karta hai
```

**"Inversion" ka matlab yahi hai** — arrow ki direction ulti ho gayi. Pehle high-level → low-level. Ab low-level → abstraction ← high-level.

**DIP vs Dependency Injection — difference (ye pooch lete hain):**
- **DIP** = principle. "Abstraction pe depend karo."
- **DI** = technique. "Dependency bahar se pass karo, andar mat banao."
- **IoC container** = tool jo DI automate karta hai. **pytest fixtures Python ka IoC container hain.**

```python
# conftest.py — ye framework ka composition root hai
@pytest.fixture
def api_client(request, config) -> ApiClient:
    mode = request.config.getoption("--api-mode", default="live")
    if mode == "live":
        return RequestsClient(config.api_url, config.token)
    if mode == "recorded":
        return RecordedClient(load_recordings())
    return FakeClient()

@pytest.fixture
def po_workflow(api_client) -> POWorkflow:
    return POWorkflow(api_client)         # injection

def test_po_approval(po_workflow):        # test ko pata hi nahi konsa client hai
    result = po_workflow.create_and_approve("ACME", 125000)
    assert result["status"] == "APPROVED"
```

> **[REAL]** Merlin ka `_helpers/api_clients.py` + `_helpers/config.py` combination isi shape ka hai — config env se aata hai, client us config se banta hai, aur test sirf client ko use karta hai. Agar kal API base URL ya auth mechanism badla, test files mein ek line nahi badlegi. Ye DIP ka business value hai: *"the change stayed in one module."*

> **Interview answer:** Dependency Inversion says high-level modules shouldn't depend on low-level ones — both should depend on an abstraction. Concretely, if my PO workflow imports `requests` and calls it directly, then the business logic is welded to an HTTP library: I can't unit test it without a network, I can't swap to Playwright's request context to reuse browser cookies, and adding a correlation-ID header means editing every call site. Instead I define an `ApiClient` protocol that the workflow owns, and inject an implementation. Now I have a real requests client, a Playwright-backed one, a recorded one for contract tests and a fake for unit tests — all satisfying the same protocol. Note the inversion: the low-level client now depends on an abstraction defined by the high-level module, so the dependency arrow points the other way. In pytest, fixtures are the injection mechanism — conftest is effectively my composition root, and the test itself never knows which implementation it got.

**Cross-question: "Isse over-abstraction nahi ho jaata? Har cheez ke liye interface?"**
> It would if I did it reflexively. My trigger is concrete: I introduce the abstraction when I have, or can clearly foresee, a second implementation — a fake for testing counts as a real second implementation, which is why the boundary at the network edge almost always earns it. For something with exactly one implementation forever, like a date formatting helper, an interface is pure ceremony and I skip it.

**Cross-question: "Mocking library use kar sakte the, abstraction kyun?"**
> Mocking a concrete library means patching module internals, so the test is coupled to how the code calls `requests`, not to what it does. Rename a keyword argument and the mock silently stops matching. With an injected protocol, the fake is coupled only to the contract I defined, so refactoring the HTTP layer doesn't touch the tests. I still use `unittest.mock` for third-party code I don't control, but for my own boundaries I prefer injection.

---

# 11. Design Patterns in Test Frameworks

> **Interview mein pattern ka naam bolna sasta hai.** Value tab aati hai jab tum bolo — *"maine ye pattern isliye use kiya, aur iska cost ye tha"*. Har pattern ke saath **kab NAHI** bhi diya hai.

| Pattern | Ek line | Test framework mein |
|---|---|---|
| Page Object | UI ka structure ek class mein | Locator change = 1 file |
| Factory | Object banane ka logic centralize | Test data creation |
| Builder | Step-by-step complex object | Fluent test data with many optional fields |
| Singleton | Ek hi instance | Config/driver — **parallel mein anti-pattern** |
| Strategy | Interchangeable algorithms | Auth: form / SSO / API token |
| Facade | Complex flow pe simple API | `create_approved_po()` = 12 steps |
| Decorator | Behaviour wrap karo | Retry, timing, logging |
| Observer | Event pe multiple listeners | pytest hooks, reporters |
| Fluent Interface | Method chaining | `page.open().login().goto_po()` |

---

## 11.1 Page Object

**Analogy:** Ek building ka **reception desk**. Tumhe andar ke kamre, corridors ka naksha nahi jaanna. Reception se bolo "HR se milna hai" — wo handle kar lega. Building ka andruni layout badle to sirf reception ko pata chalna chahiye.

```python
class LoginPage:
    PATH = "/login"

    def __init__(self, page):
        self.page = page
        # locators private — test file inhe kabhi nahi chhuegi
        self._username = page.get_by_label("Username")
        self._password = page.get_by_label("Password")
        self._submit   = page.get_by_role("button", name="Sign in")
        self._error    = page.get_by_test_id("login-error")

    def open(self) -> "LoginPage":
        self.page.goto(self.PATH)
        return self

    def login(self, username: str, password: str) -> "DashboardPage":
        self._username.fill(username)
        self._password.fill(password)
        self._submit.click()
        return DashboardPage(self.page)          # success -> agla page return

    def login_expecting_failure(self, username: str, password: str) -> "LoginPage":
        self._username.fill(username)
        self._password.fill(password)
        self._submit.click()
        return self                              # failure -> same page

    @property
    def error_message(self) -> str:
        return self._error.inner_text().strip()
```

Detail section 13 mein hai (POM built up from the duplication problem, 4 rules, mistakes).

---

## 11.2 Factory Pattern

**Analogy:** Restaurant ka kitchen. Tum "veg thali" order karte ho — kitchen decide karta hai konsi sabzi, kitni roti. Tumhe recipe nahi likhni padti.

**Factory = object banane ka logic ek jagah, caller ko sirf "kya chahiye" batana hai.**

```python
import uuid
from dataclasses import dataclass, field

@dataclass
class TestUser:
    username: str
    password: str
    role: str
    user_id: str | None = None


class UserFactory:
    """Users banata hai, aur unko cleanup ke liye track bhi karta hai."""

    ROLE_TEMPLATES = {
        "project_admin": {"role": "PROJECT_ADMIN", "permissions": ["po.create", "po.approve"]},
        "site_engineer": {"role": "SITE_ENGINEER", "permissions": ["po.create"]},
        "supplier":      {"role": "SUPPLIER",      "permissions": ["quote.submit"]},
        "finance":       {"role": "FINANCE",       "permissions": ["invoice.approve"]},
    }

    def __init__(self, api):
        self.api = api
        self._created: list[str] = []

    def create(self, kind: str, **overrides) -> TestUser:
        if kind not in self.ROLE_TEMPLATES:
            raise ValueError(f"Unknown user kind {kind!r}. Known: {sorted(self.ROLE_TEMPLATES)}")
        tpl = {**self.ROLE_TEMPLATES[kind], **overrides}
        # unique per test -> parallel safe
        uniq = uuid.uuid4().hex[:8]
        payload = {
            "username": tpl.get("username", f"qa.{kind}.{uniq}@merlin.test"),
            "password": tpl.get("password", "Test@12345"),
            "role": tpl["role"],
            "permissions": tpl["permissions"],
        }
        resp = self.api.post("/api/users", json=payload)
        self._created.append(resp["id"])
        return TestUser(payload["username"], payload["password"], tpl["role"], resp["id"])

    def cleanup(self):
        for uid in reversed(self._created):
            try:
                self.api.delete(f"/api/users/{uid}")
            except Exception as e:
                print(f"[cleanup] user {uid} failed: {e}")   # cleanup test ko fail na kare
        self._created.clear()


# ---- pytest fixture ke saath ----
import pytest

@pytest.fixture
def users(api_client):
    f = UserFactory(api_client)
    yield f
    f.cleanup()                # teardown — guaranteed

def test_supplier_cannot_approve_po(users, page):
    supplier = users.create("supplier")
    admin = users.create("project_admin")
    ...
```

**Faayde:**
- Test file mein user creation ka detail nahi — intent dikhta hai.
- Role template badla → ek jagah.
- Cleanup automatic aur guaranteed.
- Unique data har call pe → parallel safe.

**Factory ke variants (naam poochte hain):**

| Variant | Kya |
|---|---|
| **Simple Factory** | Ek method jo type ke hisaab se object banata (upar wala) |
| **Factory Method** | Subclass decide karta konsa object banega (`@classmethod create` with `cls`) |
| **Abstract Factory** | Related objects ka poora family — e.g. `MobileUIFactory` vs `DesktopUIFactory` jo alag-alag page objects deti hain |

> **Interview answer:** Factory centralises object creation so callers state what they want, not how it's built. In test automation my main use is test data: a `UserFactory` with role templates, so a test says `users.create("supplier")` instead of assembling a payload inline. Three things it buys me — the creation logic lives in one place so a schema change is a one-line fix, every created entity is tracked so a fixture teardown can clean it up, and each call generates unique identifiers so tests are parallel-safe. Without a factory, unique-data logic gets copy-pasted and eventually someone hardcodes a username and two workers collide.

**Cross-question: "Factory aur Builder mein difference?"**
> Factory answers "what to create" in one call — you pass a type or kind and get a finished object. Builder answers "how to configure" across several calls — you chain optional settings and then build. I use a factory when there are a few well-known variants, and a builder when there are many optional fields and tests need to vary one of them at a time.

---

## 11.3 Builder Pattern

**Analogy:** Subway sandwich. Bread chuno → veggies → sauce → toast karwao → **"make it"**. Har step optional hai, order flexible, aur final object last step pe banta hai.

**Problem jo builder solve karta hai — telescoping constructor:**

```python
# ❌ Ye padha nahi jaata
po = create_po("ACME", 125000, "SITE-A", True, False, "2026-09-01", None, 3, "URGENT", [])
#              kya hai True? False? 3?
```

```python
from dataclasses import dataclass, field, replace
from datetime import date, timedelta
import uuid


@dataclass(frozen=True)
class PurchaseOrder:
    supplier: str
    amount_paise: int
    site: str
    needs_approval: bool
    is_urgent: bool
    delivery_date: str
    line_items: tuple = ()
    reference: str = ""


class POBuilder:
    """Fluent builder — har method self ka NAYA copy return karta hai (immutable builder)."""

    def __init__(self):
        self._data = dict(
            supplier=f"SUP-{uuid.uuid4().hex[:6].upper()}",   # unique default
            amount_paise=100_000,
            site="SITE-DEFAULT",
            needs_approval=True,
            is_urgent=False,
            delivery_date=(date.today() + timedelta(days=30)).isoformat(),
            line_items=(),
            reference=f"REF-{uuid.uuid4().hex[:6].upper()}",
        )

    def _with(self, **kw) -> "POBuilder":
        b = POBuilder()
        b._data = {**self._data, **kw}
        return b

    # ---- fluent setters ----
    def for_supplier(self, name: str): return self._with(supplier=name)
    def with_amount(self, paise: int):  return self._with(amount_paise=paise)
    def at_site(self, site: str):       return self._with(site=site)
    def urgent(self):                   return self._with(is_urgent=True, needs_approval=True)
    def auto_approved(self):            return self._with(needs_approval=False)
    def delivered_on(self, d: str):     return self._with(delivery_date=d)
    def with_items(self, *items):       return self._with(line_items=tuple(items))

    # ---- named presets: intent-revealing shortcuts ----
    def as_high_value(self):
        return self._with(amount_paise=10_000_000, needs_approval=True)

    def as_expired(self):
        return self._with(delivery_date=(date.today() - timedelta(days=5)).isoformat())

    # ---- terminal ----
    def build(self) -> PurchaseOrder:
        d = self._data
        if d["amount_paise"] <= 0:
            raise ValueError("amount must be positive")
        if d["is_urgent"] and not d["needs_approval"]:
            raise ValueError("urgent PO must require approval — invalid combination")
        return PurchaseOrder(**d)


# ---- test mein kaisa dikhta hai ----
def test_high_value_po_requires_two_approvals(po_api):
    po = (POBuilder()
          .for_supplier("ACME Cement Pvt Ltd")
          .as_high_value()
          .at_site("SITE-MUMBAI-01")
          .with_items({"sku": "CEM-53", "qty": 500})
          .build())
    created = po_api.create(po)
    assert created["approval_levels_required"] == 2


def test_expired_delivery_date_is_rejected(po_api):
    po = POBuilder().as_expired().build()          # sirf ek cheez vary ki
    with pytest.raises(ValidationError):
        po_api.create(po)
```

**Ye kyun powerful hai:** har test mein **sirf wo field dikhta hai jo us test ke liye relevant hai**. Baaki sensible defaults. Test padhne wale ko turant pata chalta hai "ye test high-value ke baare mein hai".

**Builder mein `build()` mein validation** rakhna important hai — invalid combination ka test-time pe pata chal jaata hai, API 400 dene se pehle.

**Alternative — `dataclasses.replace` (simpler, aksar kaafi):**

```python
BASE_PO = PurchaseOrder("ACME", 100_000, "SITE-A", True, False, "2026-09-01")
high_value = replace(BASE_PO, amount_paise=10_000_000)
urgent     = replace(BASE_PO, is_urgent=True)
```

> **Interview answer:** Builder solves the telescoping-constructor problem — when an object has many optional fields, positional arguments become unreadable and every test repeats the full construction. I use a fluent builder for test data: sensible unique defaults for everything, chainable methods to override, and named presets like `.as_high_value()` or `.as_expired()` that encode domain intent. The payoff is that each test shows only the field it actually varies, so reading the test tells you what it's about. I make the builder immutable — each method returns a new builder — so a shared base builder can't be accidentally mutated by one test and leak into another. Validation of invalid combinations goes in `build()`, so a bad combination fails at construction with a clear message instead of as a 400 from the API.

**Cross-question: "Har test data ke liye builder banaoge?"**
> No — that's a lot of machinery. My rule is: fewer than four fields, use a factory function with keyword defaults; many optional fields with meaningful combinations, use a builder. And for simple variation on a frozen dataclass, `dataclasses.replace` gives most of the benefit with none of the code.

---

## 11.4 Singleton — aur parallel runs mein kyun anti-pattern hai

**Analogy:** Ghar ka **ek hi** main gas connection. Sahi hai jab ek hi family hai. Lekin agar building mein 8 flats ho jaayein aur sab ek hi connection share karein — koi bhi gas band kar de, sabka khana ruk jaata hai.

**Classic implementations:**

```python
# --- 1. __new__ override ---
class Config:
    _instance = None
    def __new__(cls, *a, **kw):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self, env="staging"):
        # ❗ GOTCHA: __init__ HAR baar chalta hai, __new__ nahi
        if getattr(self, "_initialised", False):
            return
        self.env = env
        self._initialised = True

a = Config("staging"); b = Config("production")
print(a is b, a.env)      # True staging  <- doosri call ka arg silently ignore hua!


# --- 2. Decorator based ---
def singleton(cls):
    instances = {}
    def get(*a, **kw):
        if cls not in instances:
            instances[cls] = cls(*a, **kw)
        return instances[cls]
    return get

@singleton
class DriverManager:
    def __init__(self): self.browser = None


# --- 3. Module-level (PYTHONIC — modules already singletons hain) ---
# file: _helpers/config.py
import os
from functools import lru_cache
from dataclasses import dataclass

@dataclass(frozen=True)
class EnvConfig:
    name: str
    base_url: str
    api_url: str

@lru_cache(maxsize=1)              # ye hi Python ka singleton hai
def get_config() -> EnvConfig:
    env = os.getenv("TEST_ENV", "staging")
    return EnvConfig(env, os.environ["BASE_URL"], os.environ["API_URL"])
```

### Kyun anti-pattern (ye poora bolna hai)

```
SEQUENTIAL RUN — theek                PARALLEL RUN — bomb
────────────────────                  ───────────────────
  test_1 ──> Config ──> staging        worker-1 ──┐
  test_2 ──> Config ──> staging        worker-2 ──┼──> Config (shared, thread-based)
  test_3 ──> Config ──> staging        worker-3 ──┘         │
                                                      test_2 ne env badla
                                                      test_1 aur test_3 galat env pe
```

| Problem | Detail |
|---|---|
| **Shared mutable global state** | Ek test ne badla, sab affected. Order-dependent, reproduce karna nightmare |
| **Hidden dependency** | Function signature mein nahi dikhta ki wo `Config` use kar raha hai. Padhne se pata nahi chalta |
| **Test isolation khatam** | Har test ke baad singleton reset karna padta hai — aur koi bhoolega |
| **Mock karna mushkil** | Global instance ko patch karna padega, injection nahi |
| **Parallel unsafe** | Threads/async mein race. xdist mein per-process copy hoti hai jo bug **chhupa** deti hai, fix nahi karti |
| **Lifecycle control nahi** | Kab bane, kab mare — pata nahi. Browser singleton = leaked processes |

### Sahi alternative — pytest fixture

```python
# ❌ Singleton driver — parallel mein disaster
@singleton
class DriverManager:
    def get_browser(self): ...

# ✅ Fixture — scope se lifecycle control, per-worker isolation, guaranteed teardown
@pytest.fixture(scope="session")
def browser(playwright):
    b = playwright.chromium.launch(headless=True)
    yield b
    b.close()                       # teardown guaranteed

@pytest.fixture
def page(browser):                  # har test ka apna context — isolation
    ctx = browser.new_context()
    p = ctx.new_page()
    yield p
    ctx.close()
```

**Session-scoped fixture aur singleton mein farq:** fixture ka scope **explicit** hai, teardown **guaranteed** hai, aur xdist mein har worker ko apni copy milti hai **by design**, accident se nahi. Aur dependency **signature mein dikhti hai** — `def test_x(browser)`.

> **Interview answer:** Singleton guarantees one instance globally. In Python you rarely need the classic implementation because a module is already a singleton — an `lru_cache`d `get_config()` function gives you the same thing more simply. The reason I treat it as an anti-pattern in test frameworks is that it's global mutable state with a hidden dependency: a function using the singleton doesn't declare it in its signature, so you can't tell by reading a test what it depends on, and one test mutating the config affects every other test. That produces order-dependent failures which are the worst class of flakiness to debug. Under parallel execution it's worse — with threads you get a genuine race, and with pytest-xdist each worker gets its own process so the bug is hidden rather than fixed, which means it reappears the day someone switches runner. My alternative is a session-scoped pytest fixture: explicit scope, guaranteed teardown, per-worker isolation by design, and the dependency is visible in the test signature. If the config genuinely must be global, I make it a frozen dataclass so at least it's immutable.

**Cross-question: "Toh singleton kabhi acceptable hai?"**
> Yes — when the state is immutable and the instance is genuinely process-wide, like a read-only config loaded once, or a connection pool where sharing is the point. The rule I apply is: immutable and expensive to create, singleton is fine; mutable, it's a fixture.

---

## 11.5 Strategy Pattern

**Analogy:** Google Maps ka route mode — car, bike, walk, train. **Same input (A se B), alag algorithm.** Tum runtime pe mode badal sakte ho, app ka baaki hissa same rehta hai.

**Problem:** auth ke 4 tarike hain aur `if/elif` phail raha hai (yaad hai section 4.1?).

```python
from typing import Protocol
from playwright.sync_api import Page

# ---- 1. Strategy interface ----
class AuthStrategy(Protocol):
    name: str
    def authenticate(self, page: Page, creds: dict) -> None: ...


# ---- 2. Concrete strategies ----
class FormLoginAuth:
    name = "form"
    def authenticate(self, page, creds):
        page.goto("/login")
        page.get_by_label("Username").fill(creds["username"])
        page.get_by_label("Password").fill(creds["password"])
        page.get_by_role("button", name="Sign in").click()
        page.wait_for_url("**/dashboard", timeout=15000)


class ApiTokenAuth:
    """Fastest — UI bypass. Un tests ke liye jo login ko test nahi kar rahe."""
    name = "api_token"
    def __init__(self, api): self.api = api
    def authenticate(self, page, creds):
        token = self.api.post("/api/auth/token", json=creds)["access_token"]
        page.context.add_cookies([{
            "name": "auth_token", "value": token,
            "domain": creds["domain"], "path": "/", "httpOnly": True,
        }])
        page.goto("/dashboard")


class StorageStateAuth:
    """Sabse fast — ek baar login karke storage state save, phir reuse."""
    name = "storage_state"
    def __init__(self, state_path: str): self.state_path = state_path
    def authenticate(self, page, creds):
        # context banate waqt hi state load hoti hai; yahan sirf navigate
        page.goto("/dashboard")


class SsoAuth:
    name = "sso"
    def authenticate(self, page, creds):
        page.goto("/login")
        page.get_by_role("button", name="Continue with SSO").click()
        page.wait_for_url("**/idp.okta.com/**")
        page.get_by_label("Email").fill(creds["username"])
        page.get_by_role("button", name="Next").click()
        page.get_by_label("Password").fill(creds["password"])
        page.get_by_role("button", name="Verify").click()
        page.wait_for_url("**/dashboard", timeout=30000)


# ---- 3. Context: strategy ko use karta hai, jaanta nahi konsi hai ----
class LoginService:
    def __init__(self, strategy: AuthStrategy):
        self._strategy = strategy

    def set_strategy(self, s: AuthStrategy):      # runtime pe badal sakte ho
        self._strategy = s

    def login(self, page, creds):
        print(f"[auth] using {self._strategy.name}")
        self._strategy.authenticate(page, creds)


# ---- 4. Selection: config-driven ----
STRATEGIES = {
    "form": lambda ctx: FormLoginAuth(),
    "api_token": lambda ctx: ApiTokenAuth(ctx["api"]),
    "storage_state": lambda ctx: StorageStateAuth(ctx["state_path"]),
    "sso": lambda ctx: SsoAuth(),
}

@pytest.fixture
def logged_in_page(page, config, api_client):
    mode = config.auth_mode                       # env var / CLI se
    strategy = STRATEGIES[mode]({"api": api_client, "state_path": ".auth/state.json"})
    LoginService(strategy).login(page, config.creds)
    return page
```

**Real business value:** login test ke liye `form`, baaki 500 tests ke liye `storage_state`. Har test se 6–10 second bacha. 500 tests × 8s = **~66 minutes CI time saved.**

**Strategy vs polymorphism (cross-question):** Strategy = polymorphism + **runtime selection + composition**. Sirf overriding polymorphism hai; strategy tab hai jab algorithm ek object mein encapsulated ho aur inject/swap ho sake.

> **Interview answer:** Strategy encapsulates interchangeable algorithms behind one interface so the caller picks at runtime. My canonical use is authentication. Naively you get a `login()` function with an if/elif over form login, SSO, API token and stored session state — it grows with every new auth mechanism, and every change risks the existing paths. As strategies, each mechanism is its own class with an `authenticate(page, creds)` method, selected from config. The concrete payoff is speed: the login suite runs the real form strategy because that's what it tests, and every other suite uses stored storage state, which skips six to ten seconds per test. Across five hundred tests that's over an hour of CI time. It's also the cleanest way to handle environments where staging uses form login and production-like envs use SSO.

**Cross-question: "Strategy pattern aur simple if/else — kab kaunsa?"**
> If there are two branches and no expectation of more, if/else is honest and cheaper to read. I move to Strategy when there are three or more, when the branches have their own state or setup, or when the selection has to be config-driven at runtime. The trigger I look for is a branch body growing beyond a few lines or needing its own dependencies — that's the point where the conditional starts hiding real complexity.

---

## 11.6 Facade Pattern

**Analogy:** Hotel ka **concierge**. Tum bolte ho "airport jaana hai" — wo taxi book karta hai, bill settle karta hai, luggage bhejta hai, driver ko brief karta hai. Tumhe 5 departments se baat nahi karni.

**Problem:** har test ke pehle 12 steps ka setup repeat ho raha hai.

```python
# ❌ BEFORE — har test mein 12 steps
def test_invoice_against_approved_po(page, api, config):
    # 1 login
    page.goto("/login"); page.get_by_label("Username").fill(...); ...
    # 2 create supplier
    supplier = api.post("/api/suppliers", json={...})
    # 3 approve supplier
    api.post(f"/api/suppliers/{supplier['id']}/approve")
    # 4 create site
    site = api.post("/api/sites", json={...})
    # 5 create PO
    po = api.post("/api/po", json={...})
    # 6 add line items
    api.post(f"/api/po/{po['id']}/items", json=[...])
    # 7 submit
    api.post(f"/api/po/{po['id']}/submit")
    # 8 switch to approver
    ...
    # 9 approve L1
    # 10 approve L2
    # 11 verify status
    # 12 navigate to invoice
    ...
    # ---- ab ASLI test shuru hota hai (line 60) ----
```

```python
# ✅ AFTER — Facade
class POWorkflowFacade:
    """Complex multi-service flows ke liye simple API."""

    def __init__(self, api, users_factory, config):
        self.api = api
        self.users = users_factory
        self.config = config

    def create_approved_po(
        self,
        *,                                  # keyword-only — call site self-documenting
        amount_paise: int = 125_000,
        supplier_name: str | None = None,
        site: str | None = None,
        approval_levels: int = 2,
    ) -> dict:
        """Ek fully approved PO deta hai. Andar 12 steps hain — bahar se ek call."""
        supplier = self._create_approved_supplier(supplier_name)
        site_obj = self._ensure_site(site)
        po = self.api.post("/api/po", json={
            "supplier_id": supplier["id"],
            "site_id": site_obj["id"],
            "amount_paise": amount_paise,
        })
        self.api.post(f"/api/po/{po['id']}/items", json=[
            {"sku": "CEM-53", "qty": 100, "rate_paise": amount_paise // 100}
        ])
        self.api.post(f"/api/po/{po['id']}/submit")
        for level in range(1, approval_levels + 1):
            approver = self.users.create("project_admin")
            self.api.post(f"/api/po/{po['id']}/approve",
                          json={"level": level}, as_user=approver)
        final = self.api.get(f"/api/po/{po['id']}")
        assert final["status"] == "APPROVED", (
            f"Facade contract broken: expected APPROVED, got {final['status']}. "
            "This is a setup failure, not a test failure."
        )
        return final

    def _create_approved_supplier(self, name):
        s = self.api.post("/api/suppliers", json={"name": name or unique_supplier_name()})
        self.api.post(f"/api/suppliers/{s['id']}/approve")
        return s

    def _ensure_site(self, site):
        return self.api.get(f"/api/sites/{site}") if site else \
               self.api.post("/api/sites", json={"name": f"SITE-{uuid.uuid4().hex[:6]}"})


# ---- test ab 3 line ka hai ----
def test_invoice_against_approved_po(page, po_flow, logged_in_page):
    po = po_flow.create_approved_po(amount_paise=500_000)
    invoice = InvoicePage(page).open().create_against(po["id"], amount_paise=500_000)
    assert invoice.status == "PENDING_FINANCE_APPROVAL"
```

**Facade ka critical rule — setup failure ko test failure se alag karo.** Upar wale code mein facade khud assert karta hai ki setup sach mein hua. Warna test "PENDING_FINANCE_APPROVAL nahi mila" bolega jabki asli baat ye thi ki PO approve hi nahi hua tha. **Ye distinction senior signal hai.**

> **[REAL]** `_helpers/supplier_portal.py` exactly yahi hai — supplier-side ka complex flow ek module mein wrap. Test file supplier portal ke andar ke 12 clicks nahi jaanti; wo bas domain-level function call karti hai. Ye facade hi hai, class ke bajaye module ke roop mein.

> **Interview answer:** Facade puts a simple interface over a complicated subsystem. In tests it's how I keep setup from drowning the test. Our purchase-order flow needs an approved supplier, a site, a PO with line items, submission and two levels of approval before you can even start testing invoicing — twelve steps that were repeated in every test, so the actual assertion started around line sixty. I wrapped it in a `create_approved_po()` facade with keyword-only options, so the test reads as three lines of intent. Two design details matter. First, the facade asserts its own contract at the end — if the PO isn't actually APPROVED it fails with a message saying this is a setup failure, not a test failure, so nobody debugs the wrong layer. Second, it does setup through the API rather than the UI, because setup is not the thing under test and UI setup is both slow and an extra source of flakiness.

**Cross-question: "Facade ke andar bug ho to?"**
> That's the real risk — a broken facade fails many tests at once and each failure message points at the wrong place. I handle it two ways: the facade validates its own postcondition and fails with an explicit "setup failure" message, and the facade itself has a small direct test that runs early in the suite. That way a facade regression shows up as one clearly-labelled failure rather than fifty confusing ones.

---

## 11.7 Decorator Pattern

**Analogy:** Gift wrapping. Gift wahi rehta hai, upar layer chadh jaati hai. Chaho to layers add karte jao — paper, ribbon, box. Andar ka gift kabhi nahi badalta.

**Decorator = existing function ka behaviour badle bina uske around functionality add karna.**

```python
import functools, time, logging

log = logging.getLogger("qa")


# ---------- 1. TIMING ----------
def timed(fn):
    @functools.wraps(fn)                    # ❗ metadata preserve — warna fn.__name__ = 'wrapper'
    def wrapper(*args, **kwargs):
        t0 = time.perf_counter()
        try:
            return fn(*args, **kwargs)
        finally:
            dt = time.perf_counter() - t0
            log.info("step=%s duration_ms=%.0f", fn.__name__, dt * 1000)
    return wrapper


# ---------- 2. RETRY (parametrised decorator = 3 levels) ----------
def retry(times: int = 3, delay_s: float = 1.0, on: tuple = (TimeoutError, ConnectionError)):
    def decorator(fn):
        @functools.wraps(fn)
        def wrapper(*args, **kwargs):
            last = None
            for attempt in range(1, times + 1):
                try:
                    return fn(*args, **kwargs)
                except on as e:
                    last = e
                    log.warning("retry %s attempt=%d/%d error=%s",
                                fn.__name__, attempt, times, e)
                    if attempt < times:
                        time.sleep(delay_s * (2 ** (attempt - 1)))   # exponential backoff
            raise last
        return wrapper
    return decorator


# ---------- 3. SCREENSHOT ON FAILURE ----------
def screenshot_on_failure(fn):
    @functools.wraps(fn)
    def wrapper(self, *args, **kwargs):
        try:
            return fn(self, *args, **kwargs)
        except Exception:
            path = f"artifacts/{fn.__name__}_{int(time.time())}.png"
            self.page.screenshot(path=path, full_page=True)
            log.error("step failed, screenshot=%s", path)
            raise                              # ❗ ALWAYS re-raise — swallow mat karo
    return wrapper


# ---------- 4. ALLURE-style step ----------
def step(description: str):
    def decorator(fn):
        @functools.wraps(fn)
        def wrapper(*args, **kwargs):
            log.info("STEP: %s", description.format(*args, **kwargs))
            return fn(*args, **kwargs)
        return wrapper
    return decorator


# ---------- usage ----------
class POPage:
    def __init__(self, page): self.page = page

    @step("Approve PO {1}")
    @timed
    @screenshot_on_failure
    def approve(self, po_id: str):
        self.page.goto(f"/purchase-orders/{po_id}")
        self.page.get_by_role("button", name="Approve").click()
        return self


class POApiClient:
    @retry(times=3, delay_s=0.5, on=(ConnectionError, TimeoutError))
    def get(self, po_id): ...
```

**Decorator stacking ka order — ye pooch lete hain:**

```python
@a
@b
@c
def f(): ...

# equivalent: f = a(b(c(f)))
# Execution order: a ka wrapper pehle chalta hai, phir b ka, phir c ka, phir asli f
```

Isliye upar wale example mein `step` sabse bahar (pehle log), `timed` uske andar, `screenshot_on_failure` sabse andar (function ke sabse paas) — kyunki screenshot ko asli failure ke sabse kareeb hona chahiye.

**`functools.wraps` kyun zaroori:** iske bina `fn.__name__` `'wrapper'` ban jaata hai, docstring gayab, aur **pytest test collection tak toot sakti hai**. Ye ek classic interview gotcha hai.

**⚠️ Retry decorator ka danger — ye bolna important hai:**

```python
@retry(times=3)              # ❌ ye ek real bug ko 3 baar chhupa dega
def click_approve(page):
    page.get_by_role("button", name="Approve").click()
```

UI action pe blanket retry = **flakiness ko chhupana**. Retry sirf **infra-level** exceptions pe (network reset, DNS, 502). Assertion failures aur UI actions pe kabhi nahi. Detail section 22 mein.

> **Interview answer:** The decorator pattern adds behaviour around a function without changing it, and Python's `@` syntax makes it first class. In frameworks I use it for cross-cutting concerns — timing each step, structured logging, capturing a screenshot on failure, and retrying genuinely transient infrastructure errors. Two implementation details matter. `functools.wraps` is mandatory: without it the wrapper replaces the function's name and docstring, which breaks introspection and can break pytest collection. And stacking order is `a(b(c(f)))` top-down, so I put the screenshot decorator innermost, closest to the real failure. The important judgement call is retry — I scope it narrowly to infrastructure exception types, never to assertions or UI actions, because a blanket retry on a click turns a real intermittent product bug into a green test.

**Cross-question: "Decorator pattern aur Python decorator same hai?"**
> They overlap but aren't identical. The GoF decorator pattern wraps an object to add behaviour while preserving its interface, usually via composition and a shared interface. Python's `@` syntax is a language feature for wrapping any callable, which is the same idea applied to functions. Python can also express the classic object version — a class that holds another object and forwards calls, adding behaviour — which is what a logging or retrying API client wrapper looks like.

---

## 11.8 Observer Pattern

**Analogy:** WhatsApp group. Ek banda message bhejta hai, saare members ko mil jaata hai. Sender ko nahi pata kaun-kaun members hain — group handle karta hai. Naya member add hone pe sender ka code nahi badalta.

**Observer = ek subject event emit karta hai, kai listeners react karte hain, subject ko listeners ka pata nahi.**

```python
from typing import Protocol, Callable
from dataclasses import dataclass
from enum import Enum


class Event(str, Enum):
    SUITE_START = "suite_start"
    TEST_START = "test_start"
    TEST_PASS = "test_pass"
    TEST_FAIL = "test_fail"
    TEST_SKIP = "test_skip"
    SUITE_END = "suite_end"


@dataclass
class TestEvent:
    kind: Event
    test_name: str
    duration_s: float = 0.0
    error: str | None = None
    metadata: dict | None = None


# ---- SUBJECT ----
class TestEventBus:
    def __init__(self):
        self._subscribers: dict[Event, list[Callable[[TestEvent], None]]] = {}

    def subscribe(self, kind: Event, handler: Callable[[TestEvent], None]):
        self._subscribers.setdefault(kind, []).append(handler)
        return self

    def emit(self, event: TestEvent):
        for h in self._subscribers.get(event.kind, []):
            try:
                h(event)
            except Exception as e:
                # ❗ ek observer ka crash baaki observers aur test run ko na rok de
                print(f"[eventbus] observer {h!r} failed: {e}")


# ---- OBSERVERS ----
class ConsoleReporter:
    ICONS = {Event.TEST_PASS: "PASS", Event.TEST_FAIL: "FAIL", Event.TEST_SKIP: "SKIP"}
    def __call__(self, e: TestEvent):
        print(f"[{self.ICONS.get(e.kind, e.kind.value):4}] {e.test_name} ({e.duration_s:.1f}s)")


class SlackNotifier:
    def __init__(self, webhook, only_critical=True):
        self.webhook, self.only_critical = webhook, only_critical
    def __call__(self, e: TestEvent):
        if self.only_critical and "critical" not in (e.metadata or {}).get("tags", []):
            return
        print(f"[slack->{self.webhook}] :red_circle: {e.test_name} failed\n{e.error}")


class MetricsCollector:
    def __init__(self): self.durations, self.failures = {}, []
    def __call__(self, e: TestEvent):
        self.durations[e.test_name] = e.duration_s
        if e.kind is Event.TEST_FAIL:
            self.failures.append(e.test_name)
    def slowest(self, n=10):
        return sorted(self.durations.items(), key=lambda kv: -kv[1])[:n]


class FlakyDetector:
    """Test history DB mein likhta hai — flaky detection ke liye (section 28)."""
    def __init__(self, db): self.db = db
    def __call__(self, e: TestEvent):
        self.db.insert("test_results", {
            "name": e.test_name, "status": e.kind.value,
            "duration_s": e.duration_s, "error": e.error,
        })


# ---- wiring ----
bus = TestEventBus()
metrics = MetricsCollector()
bus.subscribe(Event.TEST_PASS, ConsoleReporter())
bus.subscribe(Event.TEST_FAIL, ConsoleReporter())
bus.subscribe(Event.TEST_FAIL, SlackNotifier("https://hooks.slack/xxx"))
bus.subscribe(Event.TEST_PASS, metrics)
bus.subscribe(Event.TEST_FAIL, metrics)

bus.emit(TestEvent(Event.TEST_FAIL, "test_po_approval", 31.2, error="AssertionError: DRAFT != APPROVED",
                   metadata={"tags": ["critical"]}))
```

**pytest ka poora hook system Observer hai** — tumne already use kiya hai:

```python
# conftest.py
import pytest

def pytest_runtest_makereport(item, call):
    """pytest ne event emit kiya, hum observe kar rahe hain."""
    ...

@pytest.hookimpl(hookwrapper=True, tryfirst=True)
def pytest_runtest_makereport(item, call):
    outcome = yield
    rep = outcome.get_result()
    setattr(item, f"rep_{rep.when}", rep)      # fixtures ko result expose karo

@pytest.fixture(autouse=True)
def screenshot_on_failure(request, page):
    yield                                       # test chalta hai
    rep = getattr(request.node, "rep_call", None)
    if rep and rep.failed:
        page.screenshot(path=f"artifacts/{request.node.name}.png", full_page=True)
```

> **[REAL]** Ye exact pattern tumhare `conftest.py` mein hai — `pytest_runtest_makereport` hook test result ko fixture tak pahunchata hai, aur screenshot fixture us result ko **observe** karke act karta hai. Warning-collector fixture bhi observer hai — teardown pe collected warnings print karta hai. Interview mein bolna: *"My conftest uses pytest's hook system, which is an observer implementation — the makereport hook publishes the outcome and my screenshot and warning-collector fixtures subscribe to it. Neither knows about the other."*

> **Interview answer:** Observer lets a subject publish events to multiple subscribers without knowing who they are, so adding a new listener doesn't touch the emitting code. In test frameworks this is exactly how the runner's hook system works — pytest emits lifecycle events and reporters, screenshot handlers, metrics collectors and flaky-detection writers all subscribe independently. In my own conftest I use `pytest_runtest_makereport` as a hook wrapper to attach the outcome to the test item, and an autouse fixture then reads that outcome at teardown to capture a screenshot only on failure. The design property I care about is that adding Slack notification later meant adding a subscriber, not editing the reporting path. One implementation detail: I wrap each observer call in its own try/except, because a failing reporter should never take down the test run.

**Cross-question: "Observer ka downside?"**
> Two. Debugging gets harder because control flow is indirect — you can't see from the emit site who will run. And ordering between observers is implicit unless you make it explicit, which bites when one observer depends on another's side effect. I mitigate both by keeping observers independent and side-effect-free with respect to each other, and by logging which subscribers fired.

---

## 11.9 Fluent Interface (Method Chaining)

**Analogy:** Sentence bolna. *"Login karo, phir PO page pe jao, phir approve karo."* Ek hi saans mein, natural order mein. Vs. *"Login. Ab ek variable banao. Us variable pe PO page. Ab ek aur variable..."*

**Rule: har method `self` (ya agla object) return kare.**

```python
class LoginPage:
    def __init__(self, page): self.page = page

    def open(self) -> "LoginPage":
        self.page.goto("/login"); return self

    def with_username(self, u: str) -> "LoginPage":
        self.page.get_by_label("Username").fill(u); return self

    def with_password(self, p: str) -> "LoginPage":
        self.page.get_by_label("Password").fill(p); return self

    def submit(self) -> "DashboardPage":
        self.page.get_by_role("button", name="Sign in").click()
        return DashboardPage(self.page)          # PAGE TRANSITION -> naya object


class DashboardPage:
    def __init__(self, page): self.page = page

    def open_module(self, name: str) -> "DashboardPage":
        self.page.get_by_role("link", name=name).click(); return self

    def goto_purchase_orders(self) -> "POListPage":
        self.page.get_by_role("link", name="Purchase Orders").click()
        return POListPage(self.page)


class POListPage:
    def __init__(self, page): self.page = page

    def search(self, term: str) -> "POListPage":
        self.page.get_by_placeholder("Search POs").fill(term)
        self.page.keyboard.press("Enter"); return self

    def filter_by_status(self, status: str) -> "POListPage":
        self.page.get_by_label("Status").select_option(status); return self

    def open_first(self) -> "PODetailPage":
        self.page.locator("table tbody tr").first.click()
        return PODetailPage(self.page)


class PODetailPage:
    def __init__(self, page): self.page = page

    def approve(self) -> "PODetailPage":
        self.page.get_by_role("button", name="Approve").click(); return self

    @property
    def status(self) -> str:
        return self.page.get_by_test_id("po-status").inner_text().strip()


# ---- test padhne mein ek English sentence jaisa ----
def test_approve_pending_po(page):
    detail = (LoginPage(page)
              .open()
              .with_username("admin.qa@merlin.co")
              .with_password("Adm1n@123")
              .submit()                                # -> DashboardPage
              .goto_purchase_orders()                  # -> POListPage
              .filter_by_status("PENDING_APPROVAL")
              .open_first()                            # -> PODetailPage
              .approve())
    assert detail.status == "APPROVED"
```

**Page transition pe naya object return karna** — ye fluent POM ka best part hai. IDE tumhe sirf wahi methods dikhaayega jo **current page pe valid** hain. Galat page ka method call karna compile-time-ish error ban jaata hai.

**Downsides (ye bhi bolna — balanced answer senior lagta hai):**

| Problem | Detail |
|---|---|
| **Debugging** | 8-method chain mein failure — stack trace ek hi line number dikhata hai |
| **Assertions beech mein nahi** | Chain todni padti hai kisi intermediate check ke liye |
| **Discipline chahiye** | Ek method `return self` bhool gaya → `AttributeError: 'NoneType'` |
| **Over-chaining** | 15-method chain padhne layak nahi rehti |

**Mitigation:** chain ko 4–6 steps pe todo aur intermediate variable rakho jahan assertion chahiye.

> **Interview answer:** A fluent interface makes each method return an object so calls chain into something that reads like a sentence. In page objects I combine it with page transitions — a method that stays on the page returns `self`, and one that navigates returns the next page object. That gives me two things: the test reads as the user journey, and the IDE only offers methods valid on the page you're currently on, so calling an invoice method from the dashboard becomes hard to write rather than merely wrong. The trade-off is debugging: when an eight-call chain fails, the traceback points at one line spanning all of them, and you can't assert on an intermediate state without breaking the chain. So I cap chains at about five calls and break them wherever an intermediate assertion belongs.

**Cross-question: "Fluent interface aur Builder — same cheez?"**
> Builder usually is fluent, but fluency is just the calling style. Builder specifically constructs one object through accumulated configuration ending in a `build()` call. A fluent page object isn't building anything — it's performing navigation and actions, and the chained return value is a live object, not a deferred construction.

---

# PART 2 — FRAMEWORK ARCHITECTURE

> **Interview mein "apna framework explain karo" sabse common question hai.** Galat jawab: "Playwright hai, pytest hai, POM use kiya hai, Allure report banti hai." Ye tool ki list hai, architecture nahi.
>
> Sahi jawab ka structure: **layers → har layer ki responsibility → dependency rule → ek design decision jo tumne liya aur uska trade-off.** Neeche wahi hai.

---

# 12. Layered Architecture

## 12.1 Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│  LAYER 5 — REPORTING / OBSERVABILITY                                 │
│  Allure, JUnit XML, HTML, Slack, results DB, metrics                 │
│  Sochta hai: "run ka verdict kya tha, kya fail hua, kyun"            │
│  Jaanta hai: results ka shape. NAHI jaanta: business domain          │
└──────────────────────────────────────────────────────────────────────┘
                                 ^  (events / results)
┌──────────────────────────────────────────────────────────────────────┐
│  LAYER 1 — TEST LAYER              tests/                            │
│  test_po_approval.py, test_invoice_flow.py                           │
│  Sirf: Arrange -> Act -> Assert. Business language.                  │
│  ❌ Locator NAHI. ❌ sleep NAHI. ❌ HTTP call NAHI. ❌ if/else NAHI.  │
└──────────────────────────────┬───────────────────────────────────────┘
                               │ calls down only
                               v
┌──────────────────────────────────────────────────────────────────────┐
│  LAYER 2 — DOMAIN / PAGE LAYER     pages/  flows/  _helpers/          │
│  LoginPage, POPage, supplier_portal.py, po_validators.py             │
│  Business action -> UI/API steps ka translation                      │
│  Jaanta hai: locators, page structure, domain rules                  │
│  ❌ Assertion NAHI (validators alag). ❌ Test data creation NAHI.     │
└──────────────────────────────┬───────────────────────────────────────┘
                               │
                               v
┌──────────────────────────────────────────────────────────────────────┐
│  LAYER 3 — CORE LAYER              _helpers/interactions.py           │
│  click(), fill(), select(), wait_for(), safe_navigate()               │
│  api_clients.py, invariant_assertions.py                              │
│  "ONE CANONICAL WAY to do each action"                                │
│  Jaanta hai: Playwright API, HTTP. NAHI jaanta: koi business concept   │
└──────────────────────────────┬───────────────────────────────────────┘
                               │
                               v
┌──────────────────────────────────────────────────────────────────────┐
│  LAYER 4 — INFRA LAYER             conftest.py  config.py  ci/        │
│  Browser lifecycle, fixtures, env config, secrets, logging setup,     │
│  test data factories, DB/queue connections, docker, CI pipeline       │
│  Jaanta hai: environment. NAHI jaanta: tests ya pages                 │
└──────────────────────────────────────────────────────────────────────┘
```

## 12.2 Dependency rule — "layers only call downward"

```
  Test  ──> Page  ──> Core  ──> Infra          ✅ ALLOWED
  Test  ─────────────────────> Core            ⚠️  allowed but avoid (skip-level)
  Core  ──> Page                               ❌ FORBIDDEN (upward)
  Page  ──> Test                               ❌ FORBIDDEN
  Core  ──> Core (sibling)                     ⚠️  careful — circular import risk
```

**Kyun ye rule:** agar `interactions.py` kisi page ko import kare, to `interactions` ab business domain jaanta hai — matlab wo reusable nahi raha, aur circular imports shuru ho jaate hain. **Upward dependency = layer ka matlab khatam.**

**Practical test:** kya `_helpers/interactions.py` ko ek doosre product ke test suite mein copy kar sakte ho bina kuch badle? Agar haan — layer clean hai. Agar usme `PurchaseOrder` ka mention hai — leak ho gaya.

## 12.3 Har layer ki responsibility table

| Layer | Owns | Changes when | Should NOT contain |
|---|---|---|---|
| Test | Scenarios, assertions, business intent | Requirement badle | Locators, waits, HTTP, control flow |
| Domain/Page | Locators, page actions, domain flows | UI badle | Assertions, test data creation, env config |
| Core | Interaction primitives, HTTP client, generic assertions | Tooling badle (Playwright upgrade) | Any business concept |
| Infra | Fixtures, config, browser lifecycle, CI | Environment/pipeline badle | Test logic |
| Reporting | Result formatting, publishing, history | Reporting requirement badle | Business logic |

## 12.4 Ek concrete "leak" example

```python
# ❌ LEAK — test layer mein Layer-3 ka concern
def test_po_approval(page):
    page.goto("https://staging.merlin.app/purchase-orders")     # hardcoded URL (infra)
    page.wait_for_timeout(3000)                                  # raw wait (core)
    page.locator("//div[@class='po-row'][1]//button[2]").click() # raw locator (page layer)
    time.sleep(2)
    assert "APPROVED" in page.content()                          # weak assertion

# ✅ LAYERED
def test_po_approval(logged_in_page, po_flow):
    po = po_flow.create_approved_po(amount_paise=125_000)        # Layer 2 facade
    detail = PODetailPage(logged_in_page).open(po["id"]).approve()  # Layer 2
    assert detail.status == "APPROVED"                            # Layer 1 assertion
```

> **[REAL]** Merlin ka framework layered hai lekin **classes ke bajaye modules se**: `tests/` (L1) → `_helpers/supplier_portal.py`, `po_validators.py` (L2) → `_helpers/interactions.py`, `api_clients.py` (L3) → `conftest.py`, `config.py` (L4). Dependency rule enforce karne ka aasan tarika: `interactions.py` mein kabhi kisi domain module ka import nahi. Ye ek grep se verify ho sakta hai — CI mein ek chhota lint rule bana sakte ho.

> **Interview answer:** I structure the framework in layers with a strict rule that dependencies only point downward. The test layer contains only business intent — arrange, act, assert — with no locators, no waits, no HTTP. Below it is the domain layer, page objects and flow helpers, which translate a business action into concrete steps and own the locators. Below that is the core layer with interaction primitives, the API client and generic assertion helpers, which knows Playwright and HTTP but no business concept at all. At the bottom is infrastructure — fixtures, config, browser lifecycle, CI. Reporting sits to the side, consuming events. The rule matters because an upward dependency destroys the layer: if my `interactions` module imports a purchase-order page, it's no longer reusable and circular imports start. The check I actually apply is whether I could lift the core layer into a different product's suite unchanged — if it mentions a domain noun, something has leaked.

**Cross-question: "Layer skip kar sakte hain? Test seedha API client call kare?"**
> I allow it deliberately for setup — a test calling the API client directly to seed data is pragmatic and I'd rather that than a UI setup path. What I don't allow is a test reaching into interaction primitives, because that's where locators and waits creep back into tests. So: skipping down to a stable, business-agnostic layer for setup is fine; skipping down to the layer that couples you to the DOM is not.

---

# 13. Page Object Model

## 13.1 Kya hai — problem se shuru

**Analogy:** Ek company ka **single point of contact**. Client ko har department ka number nahi chahiye — ek account manager se baat karo. Company andar restructure ho jaaye, client ko farq nahi padta.

### Step 1 — problem: duplication

```python
def test_login_success(page):
    page.goto("https://staging.merlin.app/login")
    page.locator("#username-field").fill("admin@merlin.co")
    page.locator("#password-field").fill("Adm1n@123")
    page.locator("button.btn-primary.login-submit").click()
    assert page.url.endswith("/dashboard")

def test_login_wrong_password(page):
    page.goto("https://staging.merlin.app/login")
    page.locator("#username-field").fill("admin@merlin.co")
    page.locator("#password-field").fill("WRONG")
    page.locator("button.btn-primary.login-submit").click()
    assert page.locator(".error-toast").inner_text() == "Invalid credentials"

def test_login_locked_account(page):
    page.goto("https://staging.merlin.app/login")
    page.locator("#username-field").fill("locked@merlin.co")
    # ... same 4 lines phir se
```

Ab dev ne `#username-field` ko `#user-email` kar diya. **40 tests mein find-replace.** Kuch miss ho jaayenge. Kuch tests mein XPath tha, kuch mein CSS. **Ye POM ka origin story hai — isi se explanation shuru karo interview mein.**

### Step 2 — pehla attempt: constants

```python
USERNAME = "#username-field"
PASSWORD = "#password-field"
SUBMIT = "button.btn-primary.login-submit"
```

Better, lekin **actions abhi bhi duplicate** hain, aur ye constants global namespace pollute karte hain. Kaunsa constant kaunse page ka hai — pata nahi.

### Step 3 — POM

```python
from playwright.sync_api import Page, expect

class LoginPage:
    """Login page ka SOLE representation. Locator yahin, aur kahin nahi."""

    PATH = "/login"

    def __init__(self, page: Page):
        self.page = page
        # ---- locators: private, semantic (role/label based) ----
        self._username = page.get_by_label("Username")
        self._password = page.get_by_label("Password")
        self._submit   = page.get_by_role("button", name="Sign in")
        self._error    = page.get_by_test_id("login-error")
        self._remember = page.get_by_label("Remember me")

    # ---- navigation ----
    def open(self) -> "LoginPage":
        self.page.goto(self.PATH)
        expect(self._submit).to_be_visible()      # readiness — koi sleep nahi
        return self

    # ---- happy path: agla page return ----
    def login(self, username: str, password: str, remember: bool = False) -> "DashboardPage":
        self._username.fill(username)
        self._password.fill(password)
        if remember:
            self._remember.check()
        self._submit.click()
        return DashboardPage(self.page)

    # ---- negative path: same page return ----
    def login_expecting_failure(self, username: str, password: str) -> "LoginPage":
        self._username.fill(username)
        self._password.fill(password)
        self._submit.click()
        return self

    # ---- state exposure, NOT assertion ----
    @property
    def error_message(self) -> str:
        return self._error.inner_text().strip()

    @property
    def is_submit_enabled(self) -> bool:
        return self._submit.is_enabled()


# ---- tests ab intent-only hain ----
def test_login_success(page):
    dashboard = LoginPage(page).open().login("admin@merlin.co", "Adm1n@123")
    assert dashboard.is_loaded()

def test_login_wrong_password(page):
    login = LoginPage(page).open().login_expecting_failure("admin@merlin.co", "WRONG")
    assert login.error_message == "Invalid credentials"

def test_submit_disabled_when_fields_empty(page):
    assert LoginPage(page).open().is_submit_enabled is False
```

Ab locator change = **ek line, ek file**.

## 13.2 The 4 rules

| # | Rule | Kyun |
|---|---|---|
| **1** | **Page object mein assertion nahi** | Test decide karta hai kya sahi hai. PO sirf *state expose* karta hai. Ek assertion PO mein daali → wo PO sirf ek test ke liye reusable rahega |
| **2** | **Har method kuch return kare** — `self` ya agla page | Chaining possible, aur navigation explicit dikhta hai |
| **3** | **Raw locator expose mat karo** | `page_obj.username_field` public hua → test usko chhu legi → POM ka poora point khatam |
| **4** | **Page object business action expose kare, UI mechanics nahi** | `login()` haan; `click_submit_button()` nahi. Test ko "kaise" nahi, "kya" pata hona chahiye |

**Rule 1 ka nuance jo poochte hain:** *"Toh page object mein wait bhi assertion hai na?"* — Nahi. **Readiness wait ≠ test assertion.** `expect(self._submit).to_be_visible()` `open()` ke andar ek *precondition* hai (page use karne layak hai), test ka verdict nahi. Verdict test mein hi rehna chahiye.

## 13.3 Common mistakes

### ❌ Mistake 1 — assertions inside page object

```python
class LoginPage:
    def login(self, u, p):
        ...
        assert self.page.url.endswith("/dashboard"), "Login failed"   # ❌
```
**Kya toota:** ab negative test likhna impossible hai — wahi `login()` galat password ke saath call karoge to yahi assert fail ho jaayegi. PO ne test ka kaam apne haath mein le liya.

### ❌ Mistake 2 — kuch return nahi karna

```python
class LoginPage:
    def login(self, u, p):
        self._submit.click()          # ❌ None return

# test
LoginPage(page).open().login(...)     # AttributeError: 'NoneType' object has no attribute...
```

### ❌ Mistake 3 — raw locators public

```python
class LoginPage:
    def __init__(self, page):
        self.username_input = page.get_by_label("Username")   # ❌ public

# test file ab ye likh sakti hai:
LoginPage(page).username_input.fill("x")     # POM bypass ho gaya
```
**Fix:** `_username`. Convention se signal do.

### ❌ Mistake 4 — page object mein test data

```python
class LoginPage:
    VALID_USER = "admin@merlin.co"        # ❌ page ka kaam nahi
    VALID_PASS = "Adm1n@123"              # ❌ aur secret hardcoded!
```

### ❌ Mistake 5 — god page object

```python
class DashboardPage:
    # 62 methods, 400 lines — poore app ka wrapper
```
**Fix:** components mein todo — `HeaderComponent`, `SidebarComponent`, `POTableComponent`. Composition, inheritance nahi.

```python
class DashboardPage:
    def __init__(self, page):
        self.page = page
        self.header  = HeaderComponent(page)
        self.sidebar = SidebarComponent(page)
        self.po_table = POTableComponent(page.get_by_test_id("po-table"))

# test
dashboard.po_table.sort_by("Amount").row(0).open()
dashboard.header.switch_project("SITE-MUMBAI-01")
```

### ❌ Mistake 6 — `sleep` page object ke andar

```python
def open(self):
    self.page.goto(self.PATH)
    time.sleep(3)             # ❌ 3s har test mein, aur phir bhi flaky
```
**Fix:** Playwright ke auto-waiting + `expect()` web-first assertions.

## 13.4 Component objects (POM ka scale-up)

```python
class POTableComponent:
    """Reusable — jahan bhi PO table hai wahan use ho sakta hai."""

    def __init__(self, root):
        self._root = root                  # scoped locator — poore page pe nahi
        self._rows = root.locator("tbody tr")

    def row_count(self) -> int:
        return self._rows.count()

    def sort_by(self, column: str) -> "POTableComponent":
        self._root.get_by_role("columnheader", name=column).click()
        return self

    def find_row_by_po_id(self, po_id: str):
        return self._rows.filter(has_text=po_id)

    def open_po(self, po_id: str) -> "PODetailPage":
        self.find_row_by_po_id(po_id).get_by_role("link").click()
        return PODetailPage(self._root.page)
```

**Scoped root locator** important hai — agar page pe do tables hain, component apne root ke andar hi dhoondega. Ye "strict mode violation" flakiness khatam karta hai.

> **[REAL]** Tumhare project mein classic POM classes nahi hain, functional modules hain. Interview mein isko **weakness ki tarah mat pesh karo — decision ki tarah karo:** *"We use composable helper modules instead of page classes. With six end-to-end suites the indirection cost of a class hierarchy wasn't paying for itself, and functions compose more freely — a supplier-portal flow can use the same interaction primitives as a PO flow without any inheritance relationship. The POM goals still hold: locators live in one place, tests contain no selectors, and each action has exactly one canonical implementation. If we grew to fifty pages I'd introduce page classes, because at that size the namespacing and the IDE discoverability start earning their keep."*

> **Interview answer:** Page Object Model gives each page a class that owns its locators and exposes business actions, so tests describe intent and know nothing about the DOM. The problem it solves is duplication — before POM, changing one selector meant a find-and-replace across forty tests and you'd always miss a few. My four rules are: no assertions inside a page object, because the test decides what's correct and an assertion inside `login()` makes negative testing impossible; every method returns something, either `self` or the next page object, which makes navigation explicit and enables chaining; locators stay private so a test can't bypass the abstraction; and methods express business actions like `login()`, not UI mechanics like `click_submit_button()`. At scale I break large pages into component objects scoped to a root locator — a reusable table component, a header component — because a four-hundred-line god page object is just the duplication problem moved somewhere else.

**Cross-question: "Assertion page object mein bilkul nahi? Wait bhi to check hai."**
> I distinguish readiness from verdict. A wait inside `open()` that confirms the page is usable is a precondition — if it fails, the page object couldn't do its job, and the error should say so. A test assertion is a verdict about product behaviour and belongs in the test. The practical test I apply: if a negative test would need to call the same method and expect a different outcome, the check must not be inside the page object.

**Cross-question: "POM ka alternative kya hai?"**
> Screenplay pattern, which models actors performing tasks with abilities — it composes better for multi-actor scenarios and reads well, at the cost of more concepts. Or plain composable functions, which is what my current framework uses. The underlying goals are constant regardless of pattern: selectors in one place, tests free of DOM knowledge, one canonical way to perform each action.

---

# 14. Page Factory (aur Python ko kyun nahi chahiye)

## 14.1 Kya hai

Page Factory Selenium+Java ka ek **initialisation mechanism** hai. `@FindBy` annotation se field declare karo, `PageFactory.initElements()` un fields mein **lazy proxy** bhar deta hai.

```java
// Java / Selenium
public class LoginPage {
    @FindBy(id = "username")
    private WebElement username;              // abhi khaali proxy hai

    @FindBy(css = "button.login")
    private WebElement submit;

    public LoginPage(WebDriver driver) {
        PageFactory.initElements(driver, this);   // proxies inject
    }

    public void login(String u, String p) {
        username.sendKeys(u);      // <- YAHAN actual findElement chalta hai
        submit.click();
    }
}
```

**Ye kyun banaya gaya tha:** Java mein `driver.findElement()` **eager** hai — call karte hi DOM search hota hai. Agar constructor mein saare elements dhoondh loge aur page abhi load nahi hua, `NoSuchElementException`. Page Factory ne proxy diya jo **use ke waqt** resolve hota hai.

**Iske apne problems the:**
- `StaleElementReferenceException` — proxy purana element cache kar leta tha; page re-render hone pe stale.
- `@FindBy` sirf simple strategies support karta tha; dynamic locators ke liye nahi.
- `@CacheLookup` performance ke liye tha lekin staleness aur bigaad deta tha.
- Selenium ne khud isko de-emphasise kar diya (Selenium 4 mein Java `PageFactory` deprecated ho raha hai).

## 14.2 Python/Playwright mein kyun nahi chahiye

**Playwright ka `Locator` already lazy hai.** Wo ek **query ka description** hai, ek element reference nahi.

```python
class LoginPage:
    def __init__(self, page):
        self.page = page
        self._username = page.get_by_label("Username")     # DOM search ABHI NAHI hua
        self._submit = page.get_by_role("button", name="Sign in")

    def login(self, u, p):
        self._username.fill(u)     # <- YAHAN resolve hota hai, har baar FRESH
        self._submit.click()       # <- yahan bhi fresh — koi staleness nahi
```

| | Selenium `WebElement` | Playwright `Locator` |
|---|---|---|
| Kab resolve | `findElement()` pe (eager) | Har action pe (lazy, fresh) |
| Stale ho sakta | **Haan** — StaleElementReferenceException | **Nahi** — har baar re-query |
| Auto-wait | Nahi (explicit waits chahiye) | **Haan** — actionability checks built-in |
| Page Factory zaroori | Laziness ke liye haan | **Bilkul nahi** |

**Matlab:** Playwright ka Locator = Page Factory ka proxy, **plus** auto-waiting, **plus** staleness ka poora problem hi khatam.

**Aur ek layer:** Python mein `@FindBy` jaisi annotation-driven injection ki zaroorat isliye bhi nahi kyunki Python mein descriptors se ye trivially ho sakta hai — lekin phir bhi log nahi karte, kyunki plain attribute already lazy hai.

```python
# Agar koi zid kare ki declarative chahiye — Python mein aise hota (lekin zaroorat nahi):
class Locator:
    def __init__(self, how, what): self.how, self.what = how, what
    def __set_name__(self, owner, name): self.name = name
    def __get__(self, obj, objtype=None):
        return getattr(obj.page, self.how)(self.what)

class LoginPage:
    username = Locator("get_by_label", "Username")     # descriptor
    def __init__(self, page): self.page = page
# ...lekin ye plain `page.get_by_label("Username")` se better kuch nahi deta.
```

> **Interview answer:** Page Factory is a Selenium-Java mechanism where `@FindBy` annotations declare locators and `PageFactory.initElements` injects lazy proxies, so the element lookup happens at first use rather than in the constructor. It existed because Selenium's `findElement` is eager — resolving elements in a constructor before the page loaded threw NoSuchElementException. Its own weakness was stale element references, since the proxy could hold onto an element that the page had re-rendered. It's largely deprecated now. Python with Playwright doesn't need it because a Playwright `Locator` is already lazy by design — it's a description of a query, not a reference to an element, and it re-resolves on every action with built-in actionability waiting. So the two problems Page Factory addressed, laziness and staleness, are both solved at the API level. I'd say I understand what it was for, and that assigning locators in `__init__` in Playwright gives me the same laziness with none of the staleness.

**Cross-question: "Toh Playwright mein element kabhi stale nahi hota?"**
> The Locator itself never goes stale because it re-queries. What can still bite you is a locator that matches multiple elements after a re-render — Playwright's strict mode raises an error rather than silently picking the first, which I actually want. The other case is holding an `ElementHandle`, the older lower-level API, which can go stale exactly like a Selenium WebElement. That's why I use Locators and effectively never use ElementHandle.

---

# 15. Base Page

## 15.1 Kya andar aur kya bahar

```python
from abc import ABC, abstractmethod
from playwright.sync_api import Page, expect

class BasePage(ABC):
    """Sirf wo cheezein jo GENUINELY har page ke paas hain."""

    def __init__(self, page: Page, base_url: str):
        self.page = page
        self.base_url = base_url

    # ---- contract ----
    @property
    @abstractmethod
    def path(self) -> str: ...

    @abstractmethod
    def is_loaded(self) -> bool: ...

    # ---- universal behaviour ----
    def open(self, **query) -> "BasePage":
        qs = ("?" + "&".join(f"{k}={v}" for k, v in query.items())) if query else ""
        self.page.goto(f"{self.base_url}{self.path}{qs}")
        self.wait_until_loaded()
        return self

    def wait_until_loaded(self, timeout_ms: int = 15_000) -> "BasePage":
        expect(self.page).to_have_url(lambda u: self.path in u, timeout=timeout_ms)
        assert self.is_loaded(), f"{type(self).__name__} did not reach loaded state"
        return self

    def screenshot(self, name: str) -> str:
        path = f"artifacts/{name}.png"
        self.page.screenshot(path=path, full_page=True)
        return path

    def reload(self) -> "BasePage":
        self.page.reload()
        return self.wait_until_loaded()

    @property
    def current_url(self) -> str:
        return self.page.url

    @property
    def toast_messages(self) -> list[str]:
        return self.page.get_by_role("alert").all_inner_texts()
```

| ✅ BasePage mein rakho | ❌ BasePage mein mat rakho |
|---|---|
| Navigation (`open`, `reload`) | Table / pagination / export helpers (mixin banao) |
| Readiness contract (`is_loaded` abstract) | Login logic (wo LoginPage/auth strategy ka kaam) |
| Screenshot / artifact capture | Assertions about business rules |
| Universal page state (`current_url`, toasts) | API calls (composition se inject) |
| Very generic waits | Test data creation |
| | Anything only 2 out of 20 pages use |

**Ek line ka test:** *"Kya ye method har page ke liye meaningful hai?"* Agar 3 pages ke liye hai — mixin banao, base mein mat daalo. (ISP, section 10.4)

## 15.2 BasePage ka sabse bada risk

BasePage **magnet** hoti hai. Har banda "ye common lagta hai" sochkar apna method daal deta hai. 6 mahine mein 45 methods. Isliye:

- BasePage ka size **CI mein limit** karo (ek simple test: `assert len(public_methods(BasePage)) <= 12`).
- Naya method add karne ke liye PR mein justify karna pade.

> **Interview answer:** A base page holds only what is genuinely universal — navigation, readiness waiting, screenshot capture, and generic page state like current URL and toasts. I make `is_loaded()` abstract rather than giving it a default, because a default readiness check will silently pass on a page it wasn't written for, which is worse than failing. What I deliberately keep out is anything only some pages need — table helpers, pagination, export, upload — those go in mixins that pages opt into. The failure mode I actively guard against is the base page becoming a magnet: every developer adds "one more common thing" until it's forty-five methods that no single page uses more than five of. I've seen teams enforce this with a test that caps the base class's public method count, which sounds blunt but works.

**Cross-question: "BasePage mein `login()` daal sakte hain — har test ko login chahiye?"**
> No. Every test needs login, but not every page performs login — that's the distinction. Login belongs in an auth strategy invoked by a fixture before the page objects are used. Putting it in BasePage means every page exposes `login()`, which is meaningless on an invoice detail page and violates interface segregation.

---

# 16. Fixtures

## 16.1 Kya hai — analogy pehle

**Restaurant ka mise en place.** Chef khaana banane se pehle sab kuch cut, measure aur arrange karke rakhta hai. Order aane pe sirf cook karna hai. Aur shift ke end mein sab clean.

Fixture = **test ke liye pre-arranged setup + guaranteed cleanup.** Aur pytest mein ye **dependency injection** bhi hai.

## 16.2 Scope table

| Scope | Kab banta | Kab marta | Use case | Risk |
|---|---|---|---|---|
| `function` (default) | Har test | Test ke baad | Page, test data, per-test state | Slow agar mehnga setup |
| `class` | Har test class | Class ke baad | Related tests ka shared setup | Class-level coupling |
| `module` | Har file | File ke baad | File-level seed data | Cross-test leakage |
| `package` | Har package/dir | Package ke baad | Suite-level env | Rarely needed |
| `session` | Poore run mein ek baar | Run ke baad | Browser launch, config, auth token | **Shared mutable state** |

**Rule:** scope jitna bada, utni speed, utna hi isolation risk. **Default `function` rakho, upgrade karo sirf jab measure karke pata chale ki setup mehnga hai.**

## 16.3 Composition — fixtures ek doosre ko use karte hain

```python
# conftest.py
import pytest, uuid, logging
from playwright.sync_api import sync_playwright

log = logging.getLogger("qa")

# ---------- LAYER: session ----------
@pytest.fixture(scope="session")
def config():
    from _helpers.config import get_config
    cfg = get_config()
    log.info("=" * 70)
    log.info("ENVIRONMENT: %s  |  BASE_URL: %s", cfg.name.upper(), cfg.base_url)
    log.info("=" * 70)
    return cfg

@pytest.fixture(scope="session")
def playwright_instance():
    with sync_playwright() as pw:
        yield pw

@pytest.fixture(scope="session")
def browser(playwright_instance, config):
    b = playwright_instance.chromium.launch(
        headless=config.headless,
        args=["--no-sandbox", "--disable-dev-shm-usage"] if config.in_ci else [],
    )
    yield b
    b.close()

@pytest.fixture(scope="session")
def storage_state(browser, config, tmp_path_factory):
    """Ek baar login karke session save — har test 6-8s bachata hai."""
    path = tmp_path_factory.mktemp("auth") / "state.json"
    ctx = browser.new_context(base_url=config.base_url)
    page = ctx.new_page()
    page.goto("/login")
    page.get_by_label("Username").fill(config.username)
    page.get_by_label("Password").fill(config.password)
    page.get_by_role("button", name="Sign in").click()
    page.wait_for_url("**/dashboard")
    ctx.storage_state(path=str(path))
    ctx.close()
    return str(path)

# ---------- LAYER: function ----------
@pytest.fixture
def context(browser, config, storage_state, request):
    ctx = browser.new_context(
        base_url=config.base_url,
        storage_state=storage_state,
        viewport={"width": 1440, "height": 900},
        record_video_dir="artifacts/video" if config.record_video else None,
        extra_http_headers={"X-Correlation-Id": f"qa-{request.node.name}-{uuid.uuid4().hex[:8]}"},
    )
    ctx.tracing.start(screenshots=True, snapshots=True, sources=True)
    yield ctx
    trace_path = f"artifacts/trace/{request.node.name}.zip"
    ctx.tracing.stop(path=trace_path)
    ctx.close()

@pytest.fixture
def page(context):
    p = context.new_page()
    yield p
    p.close()

@pytest.fixture
def api_client(config):
    from _helpers.api_clients import RequestsClient
    return RequestsClient(config.api_url, config.token)

@pytest.fixture
def po_factory(api_client):
    from _helpers.factories import POFactory
    f = POFactory(api_client)
    yield f
    f.cleanup()
```

**Dependency graph:**

```
config ──┬──> browser ──> storage_state ──┐
         │       │                        │
         │       └────────────────────────┴──> context ──> page
         ├──> api_client ──> po_factory
         └──> (banner log)
```

pytest ye graph khud resolve karta hai. Tumne kabhi order nahi likha — bas dependency declare ki. **Ye IoC container hai.**

## 16.4 `autouse` — kab haan, kab naa

```python
@pytest.fixture(autouse=True)
def screenshot_on_failure(request, page):
    """Har test pe automatically — request karne ki zaroorat nahi."""
    yield
    rep = getattr(request.node, "rep_call", None)
    if rep is not None and rep.failed:
        path = f"artifacts/{request.node.name}.png"
        page.screenshot(path=path, full_page=True)
        print(f"\n[screenshot] {path}")

@pytest.fixture(autouse=True)
def collected_warnings(page):
    """Toast/console/API warnings collect karo, teardown pe print."""
    warnings = []
    page.on("console", lambda m: warnings.append(("console", m.text))
            if m.type in ("warning", "error") else None)
    page.on("response", lambda r: warnings.append(("http", f"{r.status} {r.url}"))
            if r.status >= 400 else None)
    yield warnings
    if warnings:
        print("\n--- WARNINGS COLLECTED DURING TEST ---")
        for kind, msg in warnings:
            print(f"  [{kind}] {msg}")
```

| autouse haan | autouse naa |
|---|---|
| Cross-cutting: logging, screenshots, warning collection | Anything test-specific |
| Environment guardrails (prod block) | Expensive setup — sab tests pe force ho jaayega |
| Cleanup jo bhoolna nahi chahiye | Anything jiska result test use karta hai (explicitly declare karo) |

**autouse ka downside:** dependency **chhup** jaati hai. Test padhne se pata nahi chalta ki kya-kya chal raha hai. Isliye sirf cross-cutting concerns ke liye.

## 16.5 Teardown ordering — ye poochte hain

```python
@pytest.fixture
def a():
    print("A setup"); yield "a"; print("A teardown")

@pytest.fixture
def b(a):
    print("B setup"); yield "b"; print("B teardown")

@pytest.fixture
def c(b):
    print("C setup"); yield "c"; print("C teardown")

def test_x(c): print("TEST")

# Output:
# A setup
# B setup
# C setup
# TEST
# C teardown      <- REVERSE order (LIFO)
# B teardown
# A teardown
```

**Teardown hamesha reverse order mein** — jo baad mein bana, pehle marta. Logical: `page` `context` se pehle band hona chahiye, `context` `browser` se pehle.

**Critical detail:** agar setup ke beech exception aa jaaye, **sirf un fixtures ka teardown chalega jo successfully setup ho gaye the.**

```python
@pytest.fixture
def broken(a):
    raise RuntimeError("setup failed")
    yield
# A ka teardown CHALEGA. broken ka nahi (kyunki yield tak pahuncha hi nahi).
```

**`yield` vs `addfinalizer`:**

```python
@pytest.fixture
def data(api, request):
    created = api.post("/api/po", json={...})
    request.addfinalizer(lambda: api.delete(f"/api/po/{created['id']}"))
    # addfinalizer TURANT register ho jaata hai — agar iske baad koi line fail ho,
    # cleanup phir bhi chalega. yield-based mein wo line yield se pehle fail hoti to
    # cleanup register hi nahi hota.
    api.post(f"/api/po/{created['id']}/submit")     # ye fail ho to bhi cleanup chalega
    return created
```

**Ye ek genuine senior-level detail hai.** Jab setup multi-step ho aur beech mein fail ho sakta ho, `addfinalizer` incremental cleanup registration deta hai.

## 16.6 `request` object

```python
@pytest.fixture
def po_factory(api_client, request):
    prefix = request.node.name[:30]                    # test ka naam
    marker = request.node.get_closest_marker("slow")   # markers padho
    cls = request.cls                                   # test class (agar hai)
    param = getattr(request, "param", None)             # indirect parametrization
    cfg_opt = request.config.getoption("--env")         # CLI options
    scope = request.scope                               # fixture ka scope

    f = POFactory(api_client, prefix=prefix)
    request.addfinalizer(f.cleanup)
    return f
```

**Parametrised fixture + indirect:**

```python
@pytest.fixture
def user(request, users_factory):
    return users_factory.create(request.param)

@pytest.mark.parametrize("user", ["supplier", "site_engineer", "finance"], indirect=True)
def test_role_cannot_approve_po(user, page):
    ...
# Teen alag users ke saath teen tests — fixture ko param mila
```

## 16.7 Fixture vs Factory — kab kya

| | Fixture | Factory fixture |
|---|---|---|
| Kitne objects | Ek fixed object | Test jitne chahe utne |
| Parameters | Fixed | Har call pe alag |
| Example | `admin_user` | `users.create("supplier")` |

```python
# ❌ Fixture jab tumhe 3 alag users chahiye
def test_approval_chain(admin_user, supplier_user, finance_user):  # 3 fixtures likhne padhe

# ✅ Factory fixture
def test_approval_chain(users):
    admin = users.create("project_admin")
    supplier = users.create("supplier")
    finance = users.create("finance")
    another_admin = users.create("project_admin")     # jitne chahe
```

> **[REAL]** Tumhare conftest ka `pytest_runtest_makereport` hook + screenshot fixture ka combination exactly wo pattern hai jo ye section describe karta hai: hook test outcome ko `item` pe attach karta hai (`rep_call`), aur autouse fixture teardown mein us outcome ko padhkar decide karta hai screenshot lena hai ya nahi. Bina us hook ke fixture ko pata hi nahi chalta ki test pass hua ya fail — **ye ek non-obvious pytest detail hai aur interview mein bolne pe achha impression banta hai.**

> **Interview answer:** Fixtures are pytest's dependency injection and lifecycle mechanism. A fixture declares what a test needs, pytest resolves the dependency graph and tears everything down in reverse order. Scope is the main design lever: function scope is my default because it gives maximum isolation, and I only widen it when I've measured that the setup is expensive — browser launch is session-scoped, browser context and page are function-scoped, so tests share the process but never share cookies or storage. I use autouse only for cross-cutting concerns like screenshot-on-failure and warning collection, because autouse hides the dependency from the test signature. Two details I rely on: teardown is LIFO and only runs for fixtures that completed setup, and `request.addfinalizer` registers cleanup immediately, so in a multi-step setup where a later step can fail, the earlier entities still get cleaned up — a yield-based fixture would never reach its yield in that case. And for test data I prefer a factory fixture over a plain fixture, because a test usually needs several entities with different parameters, not one fixed object.

**Cross-question: "Session-scoped fixture ka risk?"**
> Shared mutable state. Every test gets the same object, so one test mutating it creates order-dependent failures that only reproduce in a specific run order — the worst kind of flakiness. My mitigations are to make session-scoped objects immutable, usually a frozen dataclass, and to keep anything stateful at function scope. Also worth knowing: under pytest-xdist, session scope means once per worker, not once per run, so a session fixture that assumes global uniqueness — like seeding a fixed record — will collide across workers.

**Cross-question: "conftest.py hierarchy kaise kaam karti hai?"**
> pytest collects conftest files from the rootdir down to the test's directory, and fixtures defined closer to the test override ones defined higher up. So I put truly global fixtures — config, browser — in the root conftest, and suite-specific ones in that suite's directory. It's the same shadowing rule as scoping, which makes it easy to give one suite a specialised version of a fixture without affecting the rest.

---

# 17. Utilities / Helpers — "One Canonical Way"

## 17.1 Principle

**Analogy:** Hospital mein injection dene ka **ek** protocol hota hai. Har nurse apna tareeka nahi banati. Kyun? Kyunki jab kuch galat ho, tum jaanna chahte ho ki kya hua — aur agar 8 tareeke hain, root cause dhoondhna impossible hai.

**Rule: har action ka framework mein exactly EK canonical implementation ho. Baaki tareeke explicitly forbidden hon, with a documented reason.**

Kyun ye itna important hai:

| Bina canonical way | Canonical way ke saath |
|---|---|
| 8 alag click patterns codebase mein | 1 |
| Flakiness fix karne ke liye 8 jagah edit | 1 jagah edit, poora suite theek |
| Naya banda apna 9va pattern add karta hai | Naya banda existing function use karta hai |
| Code review mein "ye theek hai?" har baar | Review mein sirf "canonical use kiya?" |

## 17.2 Real implementation

```python
"""_helpers/interactions.py

ONE CANONICAL WAY to perform each UI action.

FORBIDDEN ALTERNATIVES — aur kyun:

  ❌ locator.click(force=True)
       Actionability checks bypass karta hai (visible, stable, enabled,
       receives-events). Agar element covered hai ya disabled hai, force
       click "safal" ho jaayega lekin app mein kuch nahi hoga. Ye ek
       GENUINE UI BUG ko green test mein badal deta hai.
       Agar element sach mein covered hai -> wo bug hai, report karo.

  ❌ page.wait_for_timeout(n) / time.sleep(n)
       Fixed wait. Slow machine pe kam pad jaata hai (flaky), fast machine pe
       waste hota hai (slow suite). Hamesha condition pe wait karo, time pe nahi.

  ❌ locator.evaluate("el => el.click()")
       DOM event dispatch karta hai bina real user interaction ke. React ke
       synthetic event handlers aksar isse trigger nahi hote. Test pass, app
       mein kuch nahi hua.

  ❌ page.query_selector() / ElementHandle
       Legacy eager API. Stale ho sakta hai. Locator use karo.

  ❌ try/except around an interaction with `pass`
       SILENT FAILURE. Ye framework ka sabse mehenga bug hai — humne ek poora
       release cycle ek jhoothi verification pe chalaya kyunki ek fill()
       kabhi hua hi nahi tha aur helper ne success report kiya.
       Agar failure expected hai, use expect_failure=True explicitly.
"""
from playwright.sync_api import Locator, Page, expect
import logging

log = logging.getLogger("qa.interactions")

DEFAULT_TIMEOUT_MS = 5_000


def click(locator: Locator, *, timeout_ms: int = DEFAULT_TIMEOUT_MS,
          description: str | None = None) -> None:
    """Canonical click.

    Keyword-only args after `*` — call site self-documenting rehta hai.
    click(btn, 10000) ambiguous hai; click(btn, timeout_ms=10000) nahi.
    """
    what = description or _describe(locator)
    log.debug("click: %s (timeout=%dms)", what, timeout_ms)
    expect(locator).to_be_visible(timeout=timeout_ms)
    expect(locator).to_be_enabled(timeout=timeout_ms)
    locator.click(timeout=timeout_ms)          # NO force. Ever.


def fill(locator: Locator, value: str, *, timeout_ms: int = DEFAULT_TIMEOUT_MS,
         verify: bool = True, description: str | None = None) -> None:
    """Canonical fill — aur verify karta hai ki value SACH MEIN gayi."""
    what = description or _describe(locator)
    log.debug("fill: %s = %r", what, _mask(value))
    expect(locator).to_be_visible(timeout=timeout_ms)
    expect(locator).to_be_editable(timeout=timeout_ms)
    locator.fill(value, timeout=timeout_ms)
    if verify:
        # ❗ Ye line wo bug rokti hai jo humein ek release cycle mehenga pada:
        #    fill "hua" lekin field khaali reh gayi (masked input / react controlled
        #    component / disabled overlay). Verification chal rahi thi ek jhooth pe.
        expect(locator).to_have_value(value, timeout=timeout_ms)


def select(locator: Locator, *, label: str | None = None, value: str | None = None,
           timeout_ms: int = DEFAULT_TIMEOUT_MS) -> None:
    if (label is None) == (value is None):
        raise ValueError("select() needs exactly one of label= or value=")
    expect(locator).to_be_visible(timeout=timeout_ms)
    locator.select_option(label=label, value=value, timeout=timeout_ms)


def _describe(locator: Locator) -> str:
    return str(locator)


def _mask(value: str) -> str:
    return value if len(value) < 4 else value[:2] + "*" * (len(value) - 2)
```

## 17.3 Do design decisions jo interview mein bolne layak hain

### (a) Keyword-only arguments

```python
def click(locator, *, timeout_ms: int = 5000): ...

click(approve_btn, 15000)                # ❌ TypeError — positional allowed nahi
click(approve_btn, timeout_ms=15000)     # ✅ padhne se pata chal raha hai
```

**Kyun:** ek bare number call site pe **kuch nahi batata**. Kya wo timeout hai? retry count? index? Keyword-only banane se **code review mein hi** clarity aa jaati hai, aur future mein naya parameter add karna backward-compatible rehta hai (kyunki koi positional pe depend nahi kar raha).

### (b) Silent failure ban

```python
# ❌ Wo bug jo hua tha
def fill_field(locator, value):
    try:
        locator.fill(value)
    except Exception:
        pass                     # <- yahan bug hai
    return True                  # <- aur yahan jhooth

# fill kabhi hua hi nahi. Function ne True return kiya.
# Uske baad ki saari verification ek FALSE PREMISE pe chali.
# Poora release cycle nikal gaya.
```

**Sikhne wali baat jo bolna hai:** *"A test that fails loudly costs you an hour. A test that passes falsely costs you a release."*

**Fix ke 3 layer:**
1. Bare `except: pass` ko **lint rule** se ban karo (ruff `E722`, `S110`).
2. Har `fill()` apni **postcondition verify** kare (`to_have_value`).
3. Exception ko **kabhi swallow mat karo** — agar handle karna hai to specific exception type, specific reason, aur comment.

> **Interview answer:** My rule is that the framework provides exactly one canonical way to do each action, and the forbidden alternatives are documented in the module with the reason each is forbidden. So `click` is one function that asserts visibility and enabledness and then clicks — `force=True` is banned because it bypasses actionability checks, which means if an element is covered by an overlay or disabled, the click "succeeds" while the app does nothing, turning a genuine UI bug into a green test. Fixed sleeps are banned because they're simultaneously flaky on slow machines and wasteful on fast ones. The payoff is that when we hit flakiness, the fix goes in one function and the whole suite improves — rather than hunting eight different click patterns. Two specific decisions I'd highlight. First, keyword-only arguments after a bare star, so `click(button, timeout_ms=15000)` is self-documenting at the call site where a bare `15000` tells you nothing. Second, we ban silent exception swallowing, because we had a `try: fill() except: pass` that reported success when the fill never happened — every verification after it ran on a false premise, and it survived a full release cycle. Now fill verifies its own postcondition with a value assertion, and bare excepts are a lint error.

**Cross-question: "`force=True` kabhi legitimate hai?"**
> Rarely, and never as a default. The honest cases are a known third-party widget that intercepts pointer events in a way Playwright can't model, or a deliberate test of an unusual interaction. Even then I'd want it isolated in one named helper with a comment linking to the reason, not scattered at call sites. My default reaction to needing force is that it's telling me something real about the UI — an overlay that shouldn't be there, a button enabled too early — and that's a bug worth reporting rather than working around.

**Cross-question: "Ye ek canonical function bottleneck nahi ban jaayega?"**
> It becomes a single point of change, which is exactly the point — but it does mean a bad change there breaks everything, so it needs the highest bar: its own unit tests against a fake locator, no browser needed, and careful review. In practice that's a good trade. The alternative — everyone writing their own interaction code — distributes the risk so widely that you can never fix flakiness systematically.

---

# 18. Test Data Management

## 18.1 Kya hai — analogy pehle

**Restaurant ka ingredient management.** Kuch cheezein hamesha stock mein rehti hain (namak, tel — *static seed*). Kuch roz subah aati hain (sabzi — *per-suite*). Kuch order pe banti hain (fresh juice — *per-test factory*). Aur jhoothi plate wapas nahi rakhi jaati — dho di jaati hai (*cleanup*).

## 18.2 Test data ke 4 layers

```
┌─────────────────────────────────────────────────────────────────┐
│ LAYER 1 — STATIC SEED (env ke saath aata hai, version controlled)│
│ Master data: currencies, tax codes, roles, permission matrix     │
│ Kab refresh: env rebuild pe. Kaun banata: infra/migration script │
│ Tests ise READ karte hain, kabhi MODIFY nahi                     │
├─────────────────────────────────────────────────────────────────┤
│ LAYER 2 — PER-SUITE (session/module fixture)                     │
│ Ek project, ek site, ek approved supplier — mehnga setup         │
│ Kab: suite start. Cleanup: suite end                             │
├─────────────────────────────────────────────────────────────────┤
│ LAYER 3 — PER-TEST FACTORY (function fixture)  <- 90% yahan      │
│ Har test apna PO, apna invoice, apna user banata hai             │
│ Unique names, guaranteed cleanup, parallel safe                  │
├─────────────────────────────────────────────────────────────────┤
│ LAYER 4 — FULL ISOLATION (per-test tenant/DB/schema)             │
│ Har test ka apna tenant ya DB schema. Mehnga lekin bulletproof   │
│ Kab: multi-tenant app, ya jab conflicts unmanageable ho jaayein  │
└─────────────────────────────────────────────────────────────────┘
```

**Rule:** jitna neeche jaoge, isolation utna behtar, cost utna zyada. **Default Layer 3 rakho.**

## 18.3 Factory with guaranteed cleanup

```python
# _helpers/factories.py
import uuid
from dataclasses import dataclass, field
from datetime import date, timedelta
import logging

log = logging.getLogger("qa.factory")


@dataclass
class CreatedEntity:
    kind: str
    id: str
    cleanup_path: str


class POFactory:
    def __init__(self, api, *, prefix: str = "QA"):
        self.api = api
        self.prefix = prefix
        self._created: list[CreatedEntity] = []

    # ---- unique-per-test naming: PARALLEL SAFETY ka core ----
    def _uniq(self, kind: str) -> str:
        return f"{self.prefix}-{kind}-{uuid.uuid4().hex[:10].upper()}"

    def supplier(self, *, approved: bool = True, **overrides) -> dict:
        payload = {"name": self._uniq("SUP"), "gstin": self._fake_gstin(),
                   "email": f"{uuid.uuid4().hex[:8]}@vendor.test", **overrides}
        s = self.api.post("/api/suppliers", json=payload)
        self._track("supplier", s["id"], f"/api/suppliers/{s['id']}")
        if approved:
            self.api.post(f"/api/suppliers/{s['id']}/approve")
        return s

    def purchase_order(self, *, supplier_id: str | None = None,
                       amount_paise: int = 125_000,
                       status: str = "DRAFT", **overrides) -> dict:
        supplier_id = supplier_id or self.supplier()["id"]
        payload = {"reference": self._uniq("PO"), "supplier_id": supplier_id,
                   "amount_paise": amount_paise,
                   "delivery_date": (date.today() + timedelta(days=30)).isoformat(),
                   **overrides}
        po = self.api.post("/api/po", json=payload)
        self._track("po", po["id"], f"/api/po/{po['id']}")
        if status != "DRAFT":
            po = self._advance_to(po["id"], status)
        return po

    def _advance_to(self, po_id: str, status: str) -> dict:
        chain = ["DRAFT", "PENDING_APPROVAL", "APPROVED"]
        if status not in chain:
            raise ValueError(f"Unknown target status {status!r}; known: {chain}")
        if status in ("PENDING_APPROVAL", "APPROVED"):
            self.api.post(f"/api/po/{po_id}/submit")
        if status == "APPROVED":
            self.api.post(f"/api/po/{po_id}/approve")
        final = self.api.get(f"/api/po/{po_id}")
        assert final["status"] == status, (
            f"FACTORY SETUP FAILURE: wanted {status}, got {final['status']}. "
            "This is not a test failure — the fixture could not build its precondition."
        )
        return final

    def _track(self, kind, eid, path):
        self._created.append(CreatedEntity(kind, eid, path))

    def cleanup(self):
        """Reverse order — child pehle, parent baad mein (FK constraints)."""
        for e in reversed(self._created):
            try:
                self.api.delete(e.cleanup_path)
                log.debug("cleaned %s %s", e.kind, e.id)
            except Exception as exc:
                # ❗ Cleanup NEVER fails the test — warna ek stale record poora
                #    suite red kar dega aur asli signal chhup jaayega
                log.warning("cleanup failed for %s %s: %s", e.kind, e.id, exc)
        self._created.clear()

    @staticmethod
    def _fake_gstin() -> str:
        return "27" + uuid.uuid4().hex[:10].upper() + "1Z5"


# conftest.py
@pytest.fixture
def po_data(api_client, request):
    f = POFactory(api_client, prefix=f"QA-{request.node.name[:20]}")
    yield f
    f.cleanup()
```

**Prefix mein test ka naam** — orphan record milne pe pata chal jaata hai kaunse test ne banaya. Ye debugging mein bahut kaam aata hai.

## 18.4 Parallel safety — unique data per test

```python
# ❌ PARALLEL MEIN TOOTEGA
def test_create_supplier(api):
    api.post("/api/suppliers", json={"name": "ACME Cement"})    # duplicate key error
                                                                 # jab 2 workers saath chale

# ❌ Timestamp bhi kaafi nahi — same second mein 2 workers
name = f"ACME-{int(time.time())}"

# ✅ UUID — collision practically impossible
name = f"ACME-{uuid.uuid4().hex[:10]}"

# ✅ Worker-id + counter (xdist) — readable aur unique
def worker_prefix(request):
    wid = getattr(request.config, "workerinput", {}).get("workerid", "master")
    return f"{wid}"          # gw0, gw1, gw2...
```

**Parallel-safe test ke 5 rules:**

| # | Rule | Violation ka symptom |
|---|---|---|
| 1 | Apna data khud banao, shared record use mat karo | Do tests same PO edit karte hain, race |
| 2 | Unique identifiers (UUID) | Duplicate key errors, random failures |
| 3 | Global state modify mat karo (feature flags, settings) | Ek test flag off karta, dusra fail |
| 4 | Fixed IDs assume mat karo (`PO-1`, first row) | Order-dependent failures |
| 5 | Cleanup guaranteed ho, aur idempotent ho | Data leak, env slowly bharta jaata |

**Global state ka example jo aksar bhoolte hain:**

```python
# ❌ Ye test poore env ke liye setting badal deta hai
def test_approval_threshold(api):
    api.put("/api/settings/approval_threshold", json={"value": 500000})
    ...
    # doosra worker isi waqt approval test chala raha hai -> galat threshold

# ✅ Test-scoped configuration, ya tenant-scoped
def test_approval_threshold(api, project):
    api.put(f"/api/projects/{project['id']}/settings/approval_threshold",
            json={"value": 500000})       # sirf is test ke project pe
```

## 18.5 Fixtures vs Factories

| | Fixture | Factory |
|---|---|---|
| Kitne | 1 fixed | N, parametrised |
| Best for | Env-level: config, browser, auth token | Entities: users, POs, invoices |
| Cleanup | Fixture teardown | Factory ka `cleanup()` |
| Pattern | `def test(admin_user)` | `def test(users)` phir `users.create(...)` |

**Behtareen combination:** factory ko fixture se wrap karo (upar `po_data` example). Factory ki flexibility + fixture ka guaranteed teardown.

## 18.6 Masking / PII

Production data test env mein laane ka **anonymisation** rule:

| Field | Masking strategy |
|---|---|
| Name | Deterministic fake (same input → same fake, taaki relationships bachein) |
| Email | `user_{hash}@test.invalid` — `.invalid` TLD kabhi deliver nahi hoga |
| Phone | Reserved test range |
| PAN/GSTIN/Aadhaar | Format-valid but checksum-invalid fake |
| Bank account | Fake, sandbox-only |
| Card number | **Kabhi copy mat karo** — tokenized reference ya sandbox test card |
| Amounts | Aksar preserve — distribution testing ke liye zaroori |

**Deterministic masking kyun:** agar `Ramesh Kumar` har jagah `Fake_A12` bane, to referential integrity bachi rehti hai aur joins theek chalte hain. Random masking se relationships toot jaate hain.

**Ye bolna important hai:** *"My default is synthesised data, not masked production data. Masking is a fallback when you need production-scale volume or realistic edge-case distributions, and it carries a compliance obligation — the masked copy is still regulated data until proven otherwise."*

> **[REAL]** Merlin mein tests **production pe** chalte hain — ye test data management ko critically important banata hai. Interview mein isko honestly aur strongly frame karna:
> *"Our end-to-end suites run against production, so test data management isn't a convenience — it's a safety requirement. Every test creates its own data with a QA-prefixed unique identifier, so it's identifiable and separable from real business data. Cleanup is guaranteed by fixture teardown and never fails the test, because a cleanup error shouldn't mask a real result. And crucially, our tests are additive and scoped — we create our own purchase orders rather than touching existing ones — so a test can never mutate a real customer's record."*

> **Interview answer:** I think about test data in four layers. Static seed data — currencies, tax codes, roles — ships with the environment and tests only read it. Per-suite data is expensive setup shared across a module, like a project or an approved supplier. Per-test factory data is where about ninety percent lives: each test creates its own entities through a factory fixture with unique UUID-based identifiers and guaranteed teardown. And full isolation, a per-test tenant or schema, is the escape hatch when conflicts become unmanageable. The single most important property is unique-data-per-test, because it's what makes parallel execution safe — a hardcoded supplier name means two workers collide on a duplicate key and you get failures that look random. I put the test name into the entity prefix so an orphaned record tells you which test created it. Cleanup runs in reverse creation order for foreign keys, and never fails the test — a cleanup error is logged, not raised, because a stale record shouldn't turn a real result red.

**Cross-question: "Cleanup fail ho jaaye to?"**
> I log it and let the test's own result stand, because raising in teardown replaces a real verdict with an infrastructure error. But logging alone leaks data over time, so I pair it with a scheduled sweeper job that deletes anything with the QA prefix older than 24 hours, and I track cleanup failure rate as a metric — a rising rate usually means an API contract changed or entities are gaining new dependencies that block deletion.

**Cross-question: "Production data copy karke test env mein use karna theek hai?"**
> Only with anonymisation, and I'd treat it as a fallback rather than the default. Synthesised data is safer and more targeted — I can build the exact edge case I want instead of hoping production contains it. When I do need production-shaped data, masking must be deterministic so referential integrity survives, emails go to a `.invalid` domain so nothing can be delivered, and payment instruments are never copied at all. And the masked copy is still regulated data unless the masking is provably irreversible, so it needs the same access controls.

---

# 19. Config Management

## 19.1 Precedence — defaults → file → env var → CLI

```
LOWEST                                                          HIGHEST
   │                                                                │
   v                                                                v
┌────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐
│  Defaults  │<─│ Config file  │<─│  Env vars    │<─│  CLI arguments   │
│  (code)    │  │ (.yaml/.toml)│  │ (CI secrets) │  │  (--env=staging) │
└────────────┘  └──────────────┘  └──────────────┘  └──────────────────┘
 Sensible       Per-environment    Secrets,           Ad-hoc override,
 fallbacks      profiles,          CI injection       local debugging
                version controlled
```

**Har layer ka rationale:**
- **Defaults** — framework kaam kare bina kisi setup ke (new joiner friendly).
- **File** — environment profiles, git mein, review ho sakte hain. **Secrets nahi.**
- **Env var** — CI secrets injection, 12-factor standard.
- **CLI** — ek run ke liye override, kuch persist nahi hota.

## 19.2 Implementation

```python
# _helpers/config.py
import os, json
from dataclasses import dataclass, field, replace
from functools import lru_cache
from pathlib import Path
import logging

log = logging.getLogger("qa.config")


@dataclass(frozen=True)      # frozen — koi runtime pe mutate na kar sake
class EnvConfig:
    name: str
    base_url: str
    api_url: str
    username: str
    password: str = field(repr=False)          # repr mein secret nahi
    token: str = field(default="", repr=False)
    timeout_ms: int = 5_000
    headless: bool = True
    record_video: bool = False
    auth_mode: str = "storage_state"
    in_ci: bool = False
    is_production: bool = False


DEFAULTS = dict(timeout_ms=5_000, headless=True, record_video=False,
                auth_mode="storage_state")

PROFILES = {
    "local":      dict(base_url="http://localhost:3000",
                       api_url="http://localhost:8000/api",
                       headless=False, record_video=True, auth_mode="form"),
    "staging":    dict(base_url="https://staging.merlin.app",
                       api_url="https://staging-api.merlin.app/api"),
    "qa":         dict(base_url="https://qa.merlin.app",
                       api_url="https://qa-api.merlin.app/api"),
    "production": dict(base_url="https://app.merlin.app",
                       api_url="https://api.merlin.app/api",
                       is_production=True, timeout_ms=15_000),
}


@lru_cache(maxsize=1)
def get_config() -> EnvConfig:
    env = os.getenv("TEST_ENV", "staging").lower()
    if env not in PROFILES:
        raise ValueError(f"Unknown TEST_ENV={env!r}. Known: {sorted(PROFILES)}")

    merged = {**DEFAULTS, **PROFILES[env], "name": env}

    # optional file layer
    cfg_file = Path(os.getenv("QA_CONFIG_FILE", f"config/{env}.json"))
    if cfg_file.exists():
        merged.update(json.loads(cfg_file.read_text()))

    # env var layer — secrets YAHIN se aate hain, file se KABHI nahi
    merged["username"] = os.environ["QA_USERNAME"]
    merged["password"] = os.environ["QA_PASSWORD"]
    merged["token"] = os.getenv("QA_API_TOKEN", "")
    if os.getenv("HEADLESS"):
        merged["headless"] = os.getenv("HEADLESS", "true").lower() == "true"
    merged["in_ci"] = os.getenv("CI", "").lower() in ("1", "true")

    cfg = EnvConfig(**merged)
    _print_banner(cfg)
    _guardrails(cfg)
    return cfg


def _print_banner(cfg: EnvConfig) -> None:
    """LOUD startup banner — kabhi confusion na ho ki kahan chal raha hai."""
    bar = "=" * 76
    warn = "  ⚠️  PRODUCTION  ⚠️" if cfg.is_production else ""
    print(f"\n{bar}")
    print(f"  TEST ENVIRONMENT : {cfg.name.upper()}{warn}")
    print(f"  BASE URL         : {cfg.base_url}")
    print(f"  API URL          : {cfg.api_url}")
    print(f"  USER             : {cfg.username}")
    print(f"  AUTH MODE        : {cfg.auth_mode}")
    print(f"  HEADLESS         : {cfg.headless}   TIMEOUT: {cfg.timeout_ms}ms")
    print(f"  CI               : {cfg.in_ci}")
    print(f"{bar}\n")


def _guardrails(cfg: EnvConfig) -> None:
    """Structural safety — discipline pe bharosa mat karo."""
    if cfg.is_production and not os.getenv("ALLOW_PRODUCTION_RUN"):
        raise RuntimeError(
            "Refusing to run against PRODUCTION.\n"
            "If this is intentional, set ALLOW_PRODUCTION_RUN=1 explicitly.\n"
            "This guard exists because an accidental prod run is unrecoverable."
        )
    if cfg.is_production and cfg.record_video:
        raise RuntimeError("Video recording on production may capture customer data.")
```

**Startup banner ka business value:** *"Kitni baar aisa hua hai ki koi ghante bhar ek failing test debug karta raha, aur asli baat ye thi ki wo galat environment pe chal raha tha?"* Banner isko impossible bana deta hai. Ye ek chhota feature hai jiska ROI bahut zyada hai.

## 19.3 Secrets

| ❌ Kabhi nahi | ✅ Karo |
|---|---|
| Config file mein password commit | Env var se inject |
| Code mein hardcoded token | Vault / AWS Secrets Manager / CI secret store |
| Screenshot/log mein secret | Log filter + `field(repr=False)` |
| Slack mein secret paste | Reference share karo |
| `.env` file git mein | `.env.example` commit karo, `.env` gitignore |

```python
class SecretFilter(logging.Filter):
    """Logs se secrets hataao — safety net, primary defence nahi."""
    def __init__(self, secrets: list[str]):
        super().__init__()
        self._secrets = [s for s in secrets if s and len(s) > 3]

    def filter(self, record):
        msg = str(record.getMessage())
        for s in self._secrets:
            msg = msg.replace(s, "***REDACTED***")
        record.msg, record.args = msg, ()
        return True

logging.getLogger().addFilter(SecretFilter([cfg.password, cfg.token]))
```

## 19.4 CLI options

```python
# conftest.py
def pytest_addoption(parser):
    parser.addoption("--env", default=None, help="Override TEST_ENV")
    parser.addoption("--headed", action="store_true", help="Run with visible browser")
    parser.addoption("--api-mode", default="live", choices=["live", "recorded", "fake"])
    parser.addoption("--allow-production", action="store_true")

@pytest.fixture(scope="session")
def config(request):
    if request.config.getoption("--env"):
        os.environ["TEST_ENV"] = request.config.getoption("--env")
    cfg = get_config()
    if request.config.getoption("--headed"):
        cfg = replace(cfg, headless=False)      # frozen dataclass -> replace
    return cfg
```

> **Interview answer:** Config resolves in a strict precedence order — code defaults, then a per-environment profile file in version control, then environment variables, then CLI arguments, with each layer overriding the one before. Defaults mean a new joiner can run the suite with no setup; profiles are reviewable in git; environment variables carry secrets and are how CI injects them; CLI is for a one-off override. Secrets never live in a file, and the config object is a frozen dataclass with `repr=False` on the password so it can't be printed accidentally, plus a logging filter that redacts known secret values as a safety net. Two things I'd call out. First, a loud startup banner printing the environment, base URL and user — because the most wasted debugging hour in QA is someone chasing a failure that was actually the wrong environment. Second, structural guardrails: if the resolved environment is production, the run refuses to start unless an explicit override variable is set. I don't want safety to depend on people remembering.

**Cross-question: "Config ko frozen kyun banaya?"**
> Because config is session-scoped and shared by every test. If it's mutable, one test can change the base URL or a timeout and every test after it silently runs differently, producing an order-dependent failure. Frozen makes that a hard error at the moment of mutation. When I genuinely need a variant — headed mode for local debugging — I use `dataclasses.replace` to derive a new object rather than mutating the shared one.

---

# 20. Logging

## 20.1 Setup

```python
# _helpers/logging_setup.py
import logging, sys, json, os
from datetime import datetime, timezone


class JsonFormatter(logging.Formatter):
    """Structured logs — machine parseable, ELK/Loki friendly."""
    def format(self, record):
        payload = {
            "ts": datetime.now(timezone.utc).isoformat(),
            "level": record.levelname,
            "logger": record.name,
            "msg": record.getMessage(),
            "test": getattr(record, "test_name", None),
            "correlation_id": getattr(record, "correlation_id", None),
        }
        if record.exc_info:
            payload["exc"] = self.formatException(record.exc_info)
        return json.dumps({k: v for k, v in payload.items() if v is not None})


def setup_logging(level=logging.INFO, json_output: bool | None = None):
    json_output = os.getenv("CI") if json_output is None else json_output
    root = logging.getLogger()
    root.setLevel(logging.DEBUG)
    root.handlers.clear()

    console = logging.StreamHandler(sys.stdout)
    console.setLevel(level)
    console.setFormatter(
        JsonFormatter() if json_output
        else logging.Formatter("%(asctime)s %(levelname)-7s %(name)-22s %(message)s",
                               datefmt="%H:%M:%S")
    )
    root.addHandler(console)

    # DEBUG sab kuch file mein — console pe INFO, file mein poori detail
    fh = logging.FileHandler("artifacts/test-run.log", mode="w")
    fh.setLevel(logging.DEBUG)
    fh.setFormatter(JsonFormatter())
    root.addHandler(fh)

    # third-party noise mute
    for noisy in ("urllib3", "asyncio", "PIL"):
        logging.getLogger(noisy).setLevel(logging.WARNING)
```

## 20.2 Levels — QA context mein

| Level | Kab | Example |
|---|---|---|
| `DEBUG` | Har interaction, locator, payload | `click: get_by_role('button', name='Approve')` |
| `INFO` | Test milestones, step boundaries | `STEP: Approving PO-1001` |
| `WARNING` | Retry hua, deprecated path, slow response | `retry attempt 2/3 for GET /api/po` |
| `ERROR` | Test fail, unexpected state | `Expected APPROVED, got DRAFT` |
| `CRITICAL` | Framework/infra toot gaya | `Browser launch failed — suite aborted` |

**Console pe INFO, file mein DEBUG.** Console readable rahe; jab debug karna ho to file mein poori detail milti hai.

## 20.3 Per-test log capture

```python
@pytest.fixture(autouse=True)
def test_logging(request, caplog):
    test_name = request.node.name
    correlation_id = f"qa-{uuid.uuid4().hex[:12]}"

    # har log record mein test naam aur correlation id inject
    class ContextFilter(logging.Filter):
        def filter(self, record):
            record.test_name = test_name
            record.correlation_id = correlation_id
            return True

    f = ContextFilter()
    logging.getLogger().addFilter(f)

    # per-test log file
    handler = logging.FileHandler(f"artifacts/logs/{test_name}.log", mode="w")
    handler.setFormatter(JsonFormatter())
    logging.getLogger().addHandler(handler)

    log.info("=== TEST START: %s (correlation_id=%s) ===", test_name, correlation_id)
    yield correlation_id
    log.info("=== TEST END: %s ===", test_name)

    logging.getLogger().removeHandler(handler)
    logging.getLogger().removeFilter(f)
    handler.close()
```

## 20.4 Correlation ID — backend logs se match karna

**Ye senior differentiator hai.** Jab test fail ho, tumhe sirf apna log nahi, **backend ka log bhi** chahiye — aur wo exactly usi request ka.

```python
@pytest.fixture
def context(browser, config, request):
    correlation_id = f"qa-{request.node.name[:40]}-{uuid.uuid4().hex[:8]}"
    ctx = browser.new_context(
        base_url=config.base_url,
        extra_http_headers={
            "X-Correlation-Id": correlation_id,     # har request pe jaayega
            "X-Test-Name": request.node.name,
            "X-Automated-Test": "true",             # backend ise filter/exclude kar sakta
        },
    )
    ctx.correlation_id = correlation_id
    yield ctx
    ctx.close()


@pytest.fixture
def api_client(config, request):
    cid = f"qa-{request.node.name[:40]}-{uuid.uuid4().hex[:8]}"
    return RequestsClient(config.api_url, config.token,
                          default_headers={"X-Correlation-Id": cid})
```

**Failure report mein correlation id daalo:**

```python
@pytest.fixture(autouse=True)
def attach_correlation_on_failure(request, context):
    yield
    rep = getattr(request.node, "rep_call", None)
    if rep and rep.failed:
        cid = context.correlation_id
        print(f"\n{'='*60}")
        print(f"CORRELATION ID : {cid}")
        print(f"Kibana         : https://logs.merlin.co/app/discover#/?_q=(query:'{cid}')")
        print(f"Trace          : https://traces.merlin.co/trace/{cid}")
        print(f"{'='*60}\n")
```

**Ab failure investigation ka flow:** test fail → report mein correlation id → Kibana mein paste → backend ne exactly kya kiya, kaunsa service fail hua, kaunsi query slow thi — sab dikhta hai. **QA report se root cause tak 30 second.**

**Backend team ke saath ye ek agreement hai** — unhe correlation id ko log context mein propagate karna hoga (har service mein). Ye conversation shuru karna hi seniority ka signal hai.

> **Interview answer:** For logging I set up Python's `logging` with two handlers — console at INFO with a human format, and a file at DEBUG with JSON, so the console stays readable while the full detail is available for debugging. Per test I attach a filter that injects the test name and a correlation ID into every record, plus a per-test log file, so a failure gives me exactly that test's log and nothing else. The part that changes investigation speed most is the correlation ID: I generate one per test, inject it as an `X-Correlation-Id` header on both the browser context and the API client, and print it in the failure output alongside a pre-built Kibana link. When a test fails I can go from the report to the backend's view of the same request in about thirty seconds — which service errored, which query was slow. That requires an agreement with backend to propagate the header through their services, and honestly, proposing that agreement is more valuable than any test I write that sprint. I also send `X-Automated-Test: true` so backend can exclude synthetic traffic from their business metrics.

**Cross-question: "print vs logging?"**
> `print` has no level, no timestamp, no structure, and no way to route output differently in CI versus local. Logging gives me all of that plus per-test capture and secret filtering in one place. The one exception is pytest's own reporting output — the failure summary that a human reads first — where a formatted `print` in a teardown fixture is fine because pytest captures and attaches it to the failure.

---

# 21. Reporting

## 21.1 Format comparison

| Format | Kiske liye | Strength | Weakness |
|---|---|---|---|
| **Console** | Developer, local | Instant | Ephemeral, scrollback mein kho jaata |
| **JUnit XML** | CI system | Universal — har CI parse karta hai | Insipid, no attachments |
| **pytest-html** | Team, quick share | Single file, zero infra | Basic, no history |
| **Allure** | Team + management | Steps, attachments, history, trends, categories | Needs a server/hosting |
| **Custom dashboard** | Org level | Exactly what you need, cross-suite trends | Build + maintain cost |

**Practical setup: teeno saath.**

```bash
pytest \
  --junitxml=artifacts/junit.xml \       # CI ko chahiye
  --html=artifacts/report.html --self-contained-html \   # quick share
  --alluredir=artifacts/allure-results   # rich report
```

## 21.2 Allure — steps, attachments, categories

```python
import allure
from allure_commons.types import Severity, AttachmentType


@allure.epic("Procurement")
@allure.feature("Purchase Order Approval")
@allure.story("Two-level approval for high-value POs")
@allure.severity(Severity.CRITICAL)
@allure.tag("smoke", "procurement")
@allure.link("https://jira.merlin.co/PROC-1234", name="PROC-1234")
class TestPOApproval:

    @allure.title("High-value PO requires two approvals")
    @allure.description("""
    Business rule: POs above ₹10,00,000 need L1 (Project Admin) and
    L2 (Finance) approval before they move to APPROVED.
    """)
    def test_high_value_requires_two_approvals(self, page, po_data, api_client):
        with allure.step("Given a submitted PO of ₹15,00,000"):
            po = po_data.purchase_order(amount_paise=150_000_000,
                                        status="PENDING_APPROVAL")
            allure.attach(json.dumps(po, indent=2), "PO payload", AttachmentType.JSON)

        with allure.step("When L1 approves"):
            detail = PODetailPage(page).open(po["id"]).approve()

        with allure.step("Then status is still PENDING_L2, not APPROVED"):
            assert detail.status == "PENDING_L2_APPROVAL", (
                f"Expected PENDING_L2_APPROVAL after single approval, got {detail.status}. "
                f"PO id={po['id']}"
            )


# conftest.py — failure pe artifacts attach
@pytest.fixture(autouse=True)
def allure_artifacts_on_failure(request, page, context):
    yield
    rep = getattr(request.node, "rep_call", None)
    if rep and rep.failed:
        allure.attach(page.screenshot(full_page=True), "screenshot", AttachmentType.PNG)
        allure.attach(page.content(), "dom", AttachmentType.HTML)
        allure.attach(page.url, "url", AttachmentType.TEXT)
        allure.attach(context.correlation_id, "correlation_id", AttachmentType.TEXT)
        log_file = f"artifacts/logs/{request.node.name}.log"
        if os.path.exists(log_file):
            allure.attach.file(log_file, "test log", attachment_type=AttachmentType.TEXT)
```

**Allure categories** — failures ko automatically classify karo (`allure-results/categories.json`):

```json
[
  {"name": "Product defects", "matchedStatuses": ["failed"]},
  {"name": "Test infrastructure",
   "matchedStatuses": ["broken"],
   "traceRegex": ".*(ConnectionError|TimeoutError: browser|net::ERR).*"},
  {"name": "Known flaky — quarantined",
   "matchedStatuses": ["failed"], "messageRegex": ".*FLAKY-QUARANTINE.*"},
  {"name": "Test data setup failures",
   "matchedStatuses": ["broken"], "messageRegex": ".*FACTORY SETUP FAILURE.*"}
]
```

**Ye kyun powerful hai:** report khud bata deta hai ki 12 failures mein se 8 infra hain aur 4 asli bugs. Manual triage ka time bach jaata hai.

## 21.3 Achha report kaisa dikhta hai

```
════════════════════════════════════════════════════════════════════
  VERDICT: ❌ DO NOT RELEASE
  Reason:  2 critical failures in payment-adjacent flows
════════════════════════════════════════════════════════════════════

  Environment : STAGING  (build 2026.8.14-rc3, commit a1b2c3d)
  Duration    : 14m 22s  (8 parallel workers)
  Executed    : 2026-08-23 09:14 IST

  RESULT
  ├─ Passed    : 487
  ├─ Failed    :   3   <- detail neeche
  ├─ Flaky     :   2   (passed on retry — investigate, don't ignore)
  ├─ Skipped   :   6   (4 known-issue, 2 env-gated)
  └─ Quarantine:   1   (PROC-1199, SLA expires 2026-08-28)

  ── FAILURES (root cause first) ─────────────────────────────────────
  1. test_invoice_exceeds_po_amount              [CRITICAL] [NEW]
     Expected: rejection with "Invoice exceeds PO"
     Actual  : invoice created in PENDING state
     Impact  : over-billing possible; finance control bypassed
     Evidence: screenshot, DOM, correlation qa-inv-8f3a2b1c
     Suspect : PR #4471 changed invoice validation service
     → Bug   : PROC-1450 (raised, blocker)

  2. test_po_approval_l2_threshold               [CRITICAL] [NEW]
     ... (same shape)

  3. test_supplier_bulk_upload                   [MEDIUM]  [KNOWN — PROC-1201]

  ── FLAKY (passed on rerun — these are debt) ────────────────────────
  - test_dashboard_kpi_load  (2/3 attempts) — investigating slow KPI API

  ── SUMMARY LINE ────────────────────────────────────────────────────
  487 passed. Full details: <allure link>
════════════════════════════════════════════════════════════════════
```

**Design principles:**

| Principle | Kyun |
|---|---|
| **Verdict sabse upar** | Manager ko 5 second mein decision chahiye |
| **Failures with root cause, pass count ek line** | 487 pass names likhne se koi value nahi |
| **NEW vs KNOWN alag** | Naya failure = attention. Purana = already tracked |
| **Flaky alag section** | Flaky ko "pass" mein chhupana debt chhupana hai |
| **Impact in business terms** | "over-billing possible" > "assertion failed" |
| **Evidence linked** | Screenshot, correlation id — investigate karne ke liye |
| **Suspect commit/PR** | Dev ka time bachta hai |

> **Interview answer:** I run three report formats at once because they serve different audiences — JUnit XML because every CI system parses it, a self-contained HTML file for quick sharing, and Allure for the rich view with steps, attachments and trend history. In Allure I structure tests with epic, feature and story, wrap each phase in a step, and on failure attach the screenshot, the DOM, the URL and the correlation ID. I also configure Allure categories with regexes that automatically classify failures into product defects, infrastructure problems and test-data setup failures, which removes most manual triage. But the format matters less than the content. A good report leads with the verdict — release or don't release, and why. Failures come first with root cause, business impact and evidence, and the pass count is one line, because listing 487 passing test names helps nobody. I keep flaky results in their own section rather than folding them into passes, because a test that passed on retry is debt, and hiding it is how suites rot.

**Cross-question: "Report kaun padhta hai aur kaise alag hona chahiye?"**
> Three audiences with different needs. Developers want the failing test, the stack trace, the screenshot and the correlation ID — that's Allure. The release manager wants one line: is it safe to ship, and what's blocking. Leadership wants trend — is quality improving, are we getting faster, is flakiness going down. I generate the same run into three views rather than writing one report that serves nobody well.

---

# 22. Retry Strategy

## 22.1 The core distinction

```
                 Test fails
                     │
        ┌────────────┴────────────┐
        │                         │
  Kya failure INFRA ka hai?   Kya failure APP/TEST ka hai?
        │                         │
   ✅ RETRY LEGITIMATE        ❌ RETRY HIDES A BUG
        │                         │
  - DNS resolution fail      - Assertion failed
  - Connection reset         - Element not found
  - 502/503/504 from LB      - Wrong data shown
  - Browser launch timeout   - Race condition in app
  - Docker/pod eviction      - Timing bug in test
  - Network blip             - Missing wait
```

**Ek line ka rule jo bolna hai:** *"Retry is acceptable when the failure is provably not about the system under test. Everything else, retry is a way of not fixing a bug."*

## 22.2 pytest-rerunfailures — aur uska danger

```bash
pytest --reruns 2 --reruns-delay 1        # blanket retry — ❌ ye mat karo
```

**Blanket retry kyun khatarnak hai:**

| Problem | Detail |
|---|---|
| Real intermittent bug chhup jaata hai | App mein 1-in-5 race hai. Retry se test green. Customer ko production mein milta hai |
| Suite time badh jaata hai | Har flaky test 3x chalta hai |
| Flakiness ka incentive khatam | "Retry laga do" easier hai "fix karo" se |
| Signal degrade | "Green" ka matlab hi badal jaata hai |

**Targeted retry — sahi tarika:**

```python
# 1. Sirf specific tests pe, aur reason documented
@pytest.mark.flaky(reruns=2, reruns_delay=1,
                   reason="PROC-1338: third-party PDF service returns 503 ~2% of calls")
def test_invoice_pdf_download(page, po_data): ...


# 2. Exception type ke basis pe — infra only
def pytest_collection_modifyitems(items):
    """Sirf un tests ko rerun karo jo infra exception se fail hue."""
    ...

# 3. Framework level — API client mein retry, test level pe nahi
class RequestsClient:
    RETRYABLE_STATUS = {502, 503, 504}
    RETRYABLE_EXC = (ConnectionError, ConnectionResetError)

    def _request(self, method, path, **kw):
        for attempt in range(1, 4):
            try:
                r = self.s.request(method, f"{self.base_url}{path}", **kw)
                if r.status_code in self.RETRYABLE_STATUS and attempt < 3:
                    log.warning("retryable %s from %s, attempt %d", r.status_code, path, attempt)
                    time.sleep(0.5 * 2 ** (attempt - 1))
                    continue
                r.raise_for_status()
                return r
            except self.RETRYABLE_EXC as e:
                if attempt == 3:
                    raise
                log.warning("retryable network error %s, attempt %d", e, attempt)
                time.sleep(0.5 * 2 ** (attempt - 1))
```

**Level ka rule:** retry ko **jitna neeche ho sake utna neeche** rakho. HTTP client mein retry karna better hai poore test ko rerun karne se — kyunki wahan tumhe pata hai ki kya retry ho raha hai aur kyun.

## 22.3 Quarantine with SLA

Flaky test mila. Options: (a) delete karo — coverage gayi, (b) ignore karo — noise, (c) **quarantine with a deadline**.

```python
# pytest.ini / pyproject.toml
[tool.pytest.ini_options]
markers = [
    "quarantine: flaky test excluded from the gating run — MUST have a ticket and SLA",
]

# test file
@pytest.mark.quarantine(
    ticket="PROC-1338",
    owner="ritik.chaturvedi",
    quarantined_on="2026-08-18",
    sla_days=10,
    reason="Intermittent 503 from PDF microservice; awaiting infra fix",
)
def test_invoice_pdf_download(page): ...
```

```python
# conftest.py — SLA enforce karo, warna quarantine ek kabristan ban jaata hai
def pytest_collection_modifyitems(config, items):
    from datetime import date, timedelta
    expired = []
    for item in items:
        m = item.get_closest_marker("quarantine")
        if not m:
            continue
        qdate = date.fromisoformat(m.kwargs["quarantined_on"])
        deadline = qdate + timedelta(days=m.kwargs.get("sla_days", 14))
        if date.today() > deadline:
            expired.append(f"{item.nodeid} (ticket {m.kwargs['ticket']}, "
                           f"owner {m.kwargs['owner']}, expired {deadline})")
    if expired:
        raise pytest.UsageError(
            "Quarantine SLA expired for:\n  " + "\n  ".join(expired) +
            "\nEither fix the test, or renew the quarantine with a written justification."
        )
```

**Quarantine ka poora process:**

```
Flaky detect (history se) ──> Quarantine marker + ticket + owner + SLA
                                        │
                     gating run se hata, nightly mein chalta rahe
                                        │
                          ┌─────────────┴─────────────┐
                     Fix ho gaya                SLA expire
                          │                           │
                 Quarantine hatao          CI FAIL — forced conversation
                                           (fix karo ya delete karo, decide)
```

**Ye "SLA" wala hissa senior signal hai.** Quarantine bina deadline ke ek kabristan ban jaata hai jahan 60 tests pade rehte hain aur koi nahi jaanta ki wo coverage gayab hai.

> **Interview answer:** My rule is that retry is legitimate only when the failure is provably not about the system under test — DNS failures, connection resets, a 502 from the load balancer, a browser launch timeout, a pod eviction in CI. Everything else — assertion failures, element-not-found, race conditions — retry is just a way of not fixing a bug, and it's expensive because a one-in-five race in the product will disappear from CI and reappear for a customer. So I don't use blanket `--reruns`. Instead I push retries as low as possible: the HTTP client retries specific status codes and network exceptions with exponential backoff, where I know exactly what's being retried and why. At test level, a rerun marker requires a ticket ID and a written reason. For genuinely flaky tests I quarantine rather than delete — the test leaves the gating run but keeps running nightly, and the quarantine marker carries an owner and an SLA date. If the SLA expires, collection itself fails with a message naming the test and owner, which forces the conversation. Without that deadline, quarantine becomes a graveyard where coverage quietly disappears.

**Cross-question: "Retry pe pass hua test — pass count mein daaloge?"**
> No. I report it in a separate flaky section. Folding it into passes hides debt and lets flakiness accumulate invisibly. I also track flaky rate as a first-class metric and treat a rising rate as a release risk signal, because a test that's intermittently failing is often telling you something real about the product's timing behaviour.

---

# 23. Parallelisation

## 23.1 xdist basics

```bash
pytest -n 8                      # 8 workers
pytest -n auto                   # CPU count ke barabar
pytest -n 8 --dist loadscope     # same class/module ek worker pe
pytest -n 8 --dist loadfile      # same file ek worker pe
pytest -n 8 --dist worksteal     # dynamic — fast worker slow se kaam le leta
```

| `--dist` mode | Kaise baantta | Kab use |
|---|---|---|
| `load` (default) | Test-by-test, jo worker free hai | Fully independent tests |
| `loadscope` | Class/module ek saath | Module/class-scoped fixture mehnga hai |
| `loadfile` | File ek saath | File-level shared setup |
| `worksteal` | Dynamic rebalancing | Test durations bahut alag hain |

**Ek important baat:** xdist mein har worker ek **alag process** hai. Matlab:
- Session fixtures **har worker mein ek baar** chalte hain, poore run mein ek baar nahi.
- Module-level global state naturally isolated hai (jo shared-state bugs ko **chhupa** deta hai).
- Workers ke beech koi memory sharing nahi.

## 23.2 Session fixture + xdist ka classic problem

```python
# ❌ 8 workers = 8 baar login = 8x waste, aur possibly rate-limit
@pytest.fixture(scope="session")
def storage_state(browser, config, tmp_path_factory):
    ...login karo...

# ✅ File lock se sirf ek worker login kare, baaki reuse karein
import filelock, json

@pytest.fixture(scope="session")
def storage_state(browser, config, tmp_path_factory, worker_id):
    if worker_id == "master":                 # -n use nahi ho raha
        return _do_login(browser, config)

    root_tmp = tmp_path_factory.getbasetemp().parent    # workers ke beech shared
    state_file = root_tmp / "storage_state.json"
    with filelock.FileLock(str(state_file) + ".lock"):
        if state_file.is_file():
            return str(state_file)            # kisi aur worker ne pehle bana diya
        path = _do_login(browser, config)
        state_file.write_text(open(path).read())
        return str(state_file)
```

`worker_id` fixture xdist deta hai (`gw0`, `gw1`, ... ya `master`).

## 23.3 Parallel-safe test ka checklist

| # | Requirement | Kaise verify karo |
|---|---|---|
| 1 | **No shared mutable data** | Har test apna data banata hai |
| 2 | **Unique identifiers** | UUID in every created entity name |
| 3 | **No fixed IDs** | `PO-1` ya "first row" pe depend mat karo |
| 4 | **No global config mutation** | Feature flags, settings — test-scoped ho |
| 5 | **No shared files** | `tmp_path` fixture use karo, hardcoded path nahi |
| 6 | **No fixed ports** | Random/ephemeral port |
| 7 | **Order independent** | `pytest -p no:randomly` hata ke, `--random-order` se verify |
| 8 | **Cleanup isolated** | "delete all POs" jaisa cleanup mat likho — sirf apne |

**Killer test:** `pytest --random-order -n 8` teen baar chalao. Agar teeno baar same result — parallel safe. Agar random failures — kuch shared hai.

**Aur ek diagnostic:**

```bash
pytest -n 8            # fail
pytest -n 1            # pass
# -> problem parallelism ka hai, test ka nahi. Shared resource dhoondo.
```

## 23.4 Sharding across CI machines

```yaml
# .github/workflows/e2e.yml
jobs:
  e2e:
    strategy:
      fail-fast: false
      matrix:
        shard: [1, 2, 3, 4, 5, 6]
    steps:
      - run: pytest --splits 6 --group ${{ matrix.shard }} -n 4 --junitxml=junit-${{ matrix.shard }}.xml
      # 6 machines × 4 workers = 24 parallel tests
  merge-reports:
    needs: e2e
    steps:
      - run: allure generate artifacts/*/allure-results -o allure-report
```

**Sharding vs xdist:**

| | xdist (`-n`) | Sharding (matrix) |
|---|---|---|
| Kya | Ek machine pe multiple processes | Multiple machines |
| Limit | CPU/RAM of one box | CI concurrency limit / budget |
| Best | Pehla 4–8x | Uske aage |

**Dono use karo:** `--splits 6 --group N -n 4`.

**Balanced sharding:** `pytest-split` `--durations-path` ke saath historical timings use karta hai, taaki har shard ka time barabar ho. Bina iske ek shard 3 minute mein khatam, doosra 18 minute chalta rehta — aur total time slowest shard ka hota hai.

## 23.5 Kitne workers?

```
Browser tests: har worker ko ~1 browser context chahiye ≈ 300-500 MB RAM
  8 GB machine  -> ~8 workers (RAM bound)
  4 vCPU        -> ~4-8 workers (CPU bound for JS-heavy pages)

Rule: min(CPU_count, RAM_GB × 1.5, backend_rate_limit / test_rps)
Aur phir MEASURE karo — theory se number nahi nikalta.
```

**Backend rate limit ko mat bhoolo** — 24 parallel tests backend pe 24x load daal sakte hain. Staging environment aksar isse gir jaata hai, aur phir "flaky tests" ki shikayat aati hai jabki asli baat backend saturation hai.

> **Interview answer:** I parallelise on two axes: pytest-xdist for processes within a machine, and CI matrix sharding across machines, combined — six shards times four workers gives twenty-four concurrent tests. The distribution mode matters: default `load` distributes test by test, `loadscope` keeps a class on one worker which you need when class-scoped setup is expensive, and `worksteal` rebalances dynamically when durations vary a lot. For sharding I use historical timing data so shards are balanced, because total time is the slowest shard — unbalanced shards mean one finishes in three minutes while another runs eighteen. What actually makes parallelism work is test independence: every test creates its own data with UUID-based identifiers, no test mutates global configuration, nothing depends on a fixed ID or the first row of a table, and cleanup deletes only what that test created. My verification is running the suite with random ordering at high concurrency several times — if results differ between runs, something is shared. One non-obvious constraint: session-scoped fixtures run once per worker, not once per run, so a session login means eight logins and possibly a rate limit. I solve that with a file lock so the first worker performs the login and the rest reuse the stored state.

**Cross-question: "Parallel mein test fail ho raha hai, serial mein pass. Kaise debug karoge?"**
> That result is itself the diagnosis — the test isn't self-contained. I'd bisect: run with two workers, then four, to find the concurrency at which it appears, and use `--dist loadfile` to see whether isolating a file fixes it, which points at file-level shared state. Then I look for the usual suspects in order: hardcoded entity names causing duplicate keys, a shared record two tests both modify, global settings or feature flags, and shared files or ports. The fix is almost never "reduce parallelism" — that just hides it until the day someone increases it again.

---

# 24. Tagging & Suite Composition

## 24.1 Marker setup

```toml
# pyproject.toml
[tool.pytest.ini_options]
markers = [
  # --- suite membership ---
  "smoke: critical path, must pass before anything else (<5 min)",
  "regression: full functional coverage",
  "sanity: quick post-deploy verification",
  # --- risk ---
  "critical: revenue or compliance impact; failure blocks release",
  "high: major functionality",
  "medium: standard functionality",
  # --- characteristics ---
  "slow: takes over 60 seconds",
  "flaky: known intermittent — must carry a ticket",
  "quarantine: excluded from gating run; requires ticket + SLA",
  "destructive: modifies shared state; cannot run in parallel",
  # --- domain ---
  "procurement: PO, supplier, quotation flows",
  "finance: invoice, payment, reconciliation",
  "supplier_portal: external supplier-facing flows",
  # --- environment gating ---
  "requires_sso: only runs where SSO is configured",
  "not_production: must never run against production",
]
addopts = "--strict-markers"      # ❗ typo'd marker = error, silently ignored nahi
```

`--strict-markers` **zaroori hai.** Iske bina `@pytest.mark.smoek` (typo) chup-chaap ignore ho jaata hai aur wo test kisi suite mein nahi aata.

## 24.2 Selection

```bash
pytest -m smoke                                    # smoke only
pytest -m "regression and not slow"                # fast regression
pytest -m "critical or high"                       # risk-based
pytest -m "procurement and not quarantine"         # domain, quarantine chhodkar
pytest -m "smoke and not requires_sso"             # env-gated
pytest tests/procurement -m critical -n 4          # path + marker + parallel
pytest --deselect tests/slow/test_bulk_upload.py   # ek specific test hatao
pytest -k "approval and not reject"                # naam se (marker nahi)
```

## 24.3 Suite composition — kaunsi suite kab

```
┌─────────────────────────────────────────────────────────────────────┐
│ PR / commit         │ smoke              │ ~40 tests  │ < 5 min      │
│                     │ -m "smoke"         │            │ blocking     │
├─────────────────────────────────────────────────────────────────────┤
│ Merge to main       │ smoke + critical   │ ~150 tests │ < 15 min     │
│                     │ -m "smoke or critical"                          │
├─────────────────────────────────────────────────────────────────────┤
│ Pre-deploy (staging)│ full regression    │ ~500 tests │ < 30 min     │
│                     │ -m "not quarantine and not slow"                │
├─────────────────────────────────────────────────────────────────────┤
│ Nightly             │ EVERYTHING         │ ~600 tests │ time no bar  │
│                     │ includes slow + quarantine (to track recovery)  │
├─────────────────────────────────────────────────────────────────────┤
│ Post-deploy (prod)  │ sanity             │ ~15 tests  │ < 3 min      │
│                     │ -m "sanity and not destructive"                 │
└─────────────────────────────────────────────────────────────────────┘
```

## 24.4 Markers se automatic behaviour

```python
# conftest.py — marker sirf selection ke liye nahi, behaviour ke liye bhi
def pytest_collection_modifyitems(config, items):
    env = os.getenv("TEST_ENV", "staging")
    skip_prod = pytest.mark.skip(reason="marked not_production")
    for item in items:
        # 1. production guardrail
        if env == "production" and item.get_closest_marker("not_production"):
            item.add_marker(skip_prod)
        # 2. slow tests ko timeout badhao
        if item.get_closest_marker("slow"):
            item.add_marker(pytest.mark.timeout(300))
        # 3. destructive tests ko ek hi worker pe
        if item.get_closest_marker("destructive"):
            item.add_marker(pytest.mark.xdist_group("serial"))
```

**Marker discipline ke rules:**
- Har test ke paas **exactly ek** suite marker (smoke/regression/sanity) ho.
- Har test ke paas **exactly ek** risk marker ho.
- `flaky` aur `quarantine` bina ticket ke allowed nahi (CI check se enforce).
- Naya marker add karne ke liye `pyproject.toml` mein register karna padega — `--strict-markers` isko force karta hai.

> **Interview answer:** Markers are how I compose different suites from the same test code. I keep four categories: suite membership like smoke or regression, risk level like critical or high, characteristics like slow or destructive, and domain tags. `--strict-markers` is on, so a typo in a marker name is an error rather than a test silently dropping out of every suite. Composition then maps to pipeline stages — smoke on every PR under five minutes because it must be a fast gate, smoke plus critical on merge, full regression before deploy, everything including slow and quarantined tests nightly so I can see when a quarantined test recovers, and a tiny sanity set post-deploy. I also use markers to drive behaviour, not just selection: a `not_production` marker auto-skips when the target environment is production, slow tests get a longer timeout, and destructive tests get grouped onto a single worker so they can't run concurrently with anything. The discipline that keeps this honest is requiring exactly one suite marker and one risk marker per test, enforced in CI.

**Cross-question: "Smoke suite mein kya hona chahiye — kaise decide karoge?"**
> I define it by business criticality plus reachability: if this breaks, is the product unusable, and does it gate everything else? For our ERP that's login, project selection, creating a purchase order, and the approval path — roughly forty tests. My constraint is a hard five-minute budget, because a gate people wait on is a gate people start bypassing. If a new candidate would push it over budget, something has to earn its way out, which forces a real conversation about what's actually critical rather than the suite growing by accretion.

---

# 25. Environment Separation

## 25.1 Profiles

| Env | Purpose | Data | Who runs | Suite |
|---|---|---|---|---|
| `local` | Development | Seeded/reset freely | Developer | Any, headed |
| `dev` | Integration of in-progress work | Volatile | CI on feature branch | smoke |
| `qa` | QA's controlled environment | QA owns, stable | CI + manual | full regression |
| `staging` | Production mirror | Prod-like volume, anonymised | CI pre-deploy | full regression |
| `production` | Live | Real customer data | CI post-deploy only | sanity, read-mostly |

## 25.2 Guardrails — discipline pe bharosa mat karo

```python
# conftest.py

PRODUCTION_HOSTS = {"app.merlin.app", "api.merlin.app"}

@pytest.fixture(scope="session", autouse=True)
def production_guardrails(config):
    """Structural safety. Har layer pe check."""
    if not config.is_production:
        return

    # 1. Explicit opt-in chahiye
    if not os.getenv("ALLOW_PRODUCTION_RUN"):
        pytest.exit("Refusing production run without ALLOW_PRODUCTION_RUN=1", returncode=2)

    # 2. Loud banner
    print("\n" + "!" * 76)
    print("!!  RUNNING AGAINST PRODUCTION — real customer data is present  !!")
    print("!!  Only additive, QA-prefixed data may be created              !!")
    print("!" * 76 + "\n")

    # 3. Video/trace off — customer data leak ka risk
    if config.record_video:
        pytest.exit("Video recording is forbidden on production", returncode=2)


@pytest.fixture(autouse=True)
def block_destructive_api_calls(config, monkeypatch, api_client):
    """Production pe DELETE sirf QA-prefixed entities pe allowed."""
    if not config.is_production:
        return
    original_delete = api_client.delete

    def guarded_delete(path, *a, **kw):
        if "/QA-" not in path and not _is_qa_owned(path):
            raise RuntimeError(
                f"BLOCKED: attempted DELETE on non-QA entity in production: {path}"
            )
        return original_delete(path, *a, **kw)

    monkeypatch.setattr(api_client, "delete", guarded_delete)
```

## 25.3 Per-env data strategy

```python
ENV_DATA_STRATEGY = {
    "local":      "seed_and_reset",     # DB reset allowed
    "dev":        "create_and_cleanup",
    "qa":         "create_and_cleanup",
    "staging":    "create_and_cleanup",
    "production": "create_only_prefixed_readonly_otherwise",
}
```

**Production pe testing ka honest framing (tumhare liye important, kyunki tumhare suites prod pe chalte hain):**

> **[REAL] Interview mein aise bolo:** *"Our six end-to-end suites run against production. That's not ideal and I'd be the first to say so — but it was the right call given the constraint that our staging environment didn't have representative data or the full integration surface, so staging-green told us very little. What we did was make it safe by design rather than by care. Every entity we create carries a QA prefix and a UUID, so our data is identifiable and separable. Tests are additive — we create our own purchase orders rather than modifying existing ones — so a test can't corrupt a real record. Deletes in production are guarded to only touch QA-prefixed entities. Video and trace recording are disabled there because they'd capture customer data. And we send an `X-Automated-Test` header so backend can exclude our traffic from business metrics. The upside is real: we catch integration and configuration issues that a staging suite structurally cannot. If I were building this again with more leverage, I'd invest in a production-like staging environment first — but I'd keep a small production sanity suite permanently, because post-deploy verification in the real environment catches a class of problem nothing else does."*

**Ye answer strong hai kyunki:** (a) trade-off acknowledge kiya, (b) risk ko structurally mitigate kiya, (c) business reason diya, (d) better alternative bhi jaanta hai.

> **Interview answer:** I separate environments by purpose and give each its own config profile, data strategy and suite. Local is disposable, QA is controlled and stable for regression, staging mirrors production for the pre-deploy gate, and production runs a small sanity suite post-deploy. The important part is guardrails, because environment mistakes are the ones that hurt most and relying on people remembering doesn't scale. So: the run refuses to start against production unless an explicit override variable is set, a loud startup banner names the environment and base URL, video and trace recording are disabled there because they capture customer data, deletes are blocked on anything without a QA prefix, and tests marked `not_production` are auto-skipped at collection. All of that is code, not documentation.

**Cross-question: "Production pe test karna galat nahi hai?"**
> It's a risk you take deliberately or not at all. What makes it defensible is that it's additive and scoped — you create your own prefixed data, never modify existing records, and destructive operations are structurally blocked. What makes it valuable is that it catches configuration, integration and data-shape problems that a non-representative staging environment cannot. What makes it indefensible is doing it without those controls, or using it as a substitute for having a decent pre-production environment. I'd always want a small production sanity suite; I'd never want production to be my only signal.

---

# PART 3 — SYSTEM DESIGN FOR TESTERS

> **Ye hi wo section hai jo ~1 year wale QA ko senior SDET se alag karta hai.** Coding round har koi crack kar leta hai. Design round mein 90% candidate seedha solution bolna shuru kar dete hain — aur wahin haar jaate hain.
>
> **Interviewer kya dekh raha hai:** kya tu **problem ko define kar sakta hai** solve karne se pehle? Kya tu **trade-off** bol sakta hai? Kya tu **scale ke saath kya badlega** samajhta hai?
>
> Yaad rakho: **design round mein "sahi jawab" nahi hota. Reasoning hi jawab hai.**

---

# 26. The 6-Step Answer Framework

## 26.1 Framework

```
┌──────────────────────────────────────────────────────────────────────┐
│ STEP 1 — CLARIFY (2-3 min)          ⭐ PEHLA SCORED POINT YAHIN HAI   │
│   Sawaal poocho. Assume mat karo. Interviewer deliberately vague hai. │
│   "Before I design, let me make sure I understand the problem."       │
├──────────────────────────────────────────────────────────────────────┤
│ STEP 2 — CONSTRAINTS & SCALE (2 min)                                  │
│   Numbers nikalo. Kitne tests? Kitna time budget? Kitni team?         │
│   Kitne env? Kya budget? Numbers bina design meaningless hai.         │
├──────────────────────────────────────────────────────────────────────┤
│ STEP 3 — RISK ANALYSIS (2-3 min)      ⭐ QA-SPECIFIC EDGE             │
│   "Kya toot sakta hai aur uska business cost kya hai?"                │
│   Dev system design mein ye step nahi hota. Tumhara differentiator.   │
├──────────────────────────────────────────────────────────────────────┤
│ STEP 4 — HIGH-LEVEL DESIGN (5-8 min)                                  │
│   Boxes + arrows. Components + responsibilities + data flow.          │
│   Whiteboard/ascii pe draw karo. Bolte waqt draw karo.                │
├──────────────────────────────────────────────────────────────────────┤
│ STEP 5 — DEEP DIVE (8-10 min)                                         │
│   1-2 areas chuno (ya interviewer bole). Code-level detail.           │
│   Yahan tumhari technical depth dikhti hai.                           │
├──────────────────────────────────────────────────────────────────────┤
│ STEP 6 — TRADE-OFFS & EVOLUTION (3-5 min)                             │
│   "Maine X chuna Y ke bajaye kyunki Z. Iska cost ye hai."             │
│   "10x scale pe kya pehle tootega."   ⭐ SENIOR SIGNAL                │
└──────────────────────────────────────────────────────────────────────┘
```

## 26.2 Step 1 ko seriously lo — ye khud ek scored point hai

Interviewer ka question **jaan-boojhkar** vague hota hai. *"Design a test automation framework."* Kitne tests? Web ya mobile? Team size? Kuch nahi bataya.

**Agar tum seedha solution bolna shuru karoge — tum ek point already haar gaye.** Wo dekh raha hai ki tum requirement gather karte ho ya assume karte ho. Kyunki job mein bhi yahi hoga: PM ek adhoori requirement dega, aur tumhe sawaal poochna hoga.

**Clarifying questions ka checklist (kisi bhi QA design question ke liye):**

| Category | Sawaal |
|---|---|
| **Scope** | Web, mobile, API — teeno? Kaunse browsers/devices? |
| **Scale** | Kitne tests aaj? 1 saal mein kitne expected? Kitne engineers likhenge? |
| **Speed** | CI feedback ka budget kya hai? PR pe kitna time acceptable? |
| **Team** | Kaun likhega — dedicated SDET, ya devs bhi? Unka skill level? |
| **Stack** | Existing stack kya hai? Team konsi language jaanti hai? |
| **Environments** | Kitne env? Data kaisa hai? Prod-like staging hai? |
| **Existing** | Zero se bana rahe hain ya kuch already hai? Migration hai? |
| **Constraints** | Budget? Cloud allowed? Compliance/data residency? |
| **Success** | Iska success kaise measure hoga? Kya problem solve kar rahe hain? |

**Sabse powerful sawaal:** *"What problem is this solving today — what's currently going wrong?"* Ye poochna dikhata hai ki tum tool nahi, **outcome** ke baare mein soch rahe ho.

## 26.3 Step 3 — risk analysis (tumhara QA edge)

Developer system design mein risk analysis nahi hota. QA mein ye **core** hai.

```
Har feature/component ke liye:
    Risk score = Likelihood of failure × Business impact of failure

  Business impact ki categories:
    - Revenue loss        (payment fail, order fail)
    - Compliance/legal    (audit trail, PII leak, tax calculation)
    - Data corruption     (irreversible — sabse bura)
    - User trust          (wrong data dikhna)
    - Convenience         (UI glitch — lowest)
```

**Ye bolna:** *"I'd allocate test depth by risk, not by feature count. Payment and approval flows get exhaustive coverage including partial-failure scenarios. A settings page that changes a display preference gets a smoke test. Uniform coverage is a false economy — it costs the same to maintain but the value is wildly different."*

## 26.4 Common mistakes

| ❌ Mistake | ✅ Fix |
|---|---|
| Seedha solution bolna | 2-3 min clarify karo |
| Sirf tools list karna ("Playwright, Allure, Jenkins") | Components + responsibilities + boundaries |
| Numbers nahi dena | "500 tests, 8 workers, 12 min — here's the math" |
| Trade-off nahi bolna | "I chose X over Y because Z; the cost is W" |
| Perfect design pesh karna | "This design breaks at 10x when the results DB becomes the bottleneck" |
| Chup rehna sochte waqt | Loudly socho — reasoning hi score hai |
| Sirf happy path | "What happens when a worker dies mid-test?" |

> **Interview answer:** My approach to any design question is six steps. First I clarify — scope, scale, team, existing stack, and most importantly what problem this is solving today, because the question is deliberately underspecified and jumping to a solution means designing for a problem I haven't confirmed. Second I establish constraints and numbers: how many tests now and in a year, what the CI time budget is, how many engineers contribute. Third — and this is where QA design differs from developer design — I do risk analysis: what can break and what does each failure cost the business, because that's how I allocate test depth. Fourth, the high-level design, components and responsibilities and data flow, drawn out. Fifth, a deep dive into one or two areas where the real complexity lives. And sixth, trade-offs and evolution: what I chose, what I gave up, and what breaks first at ten times the scale. I'd stress that the clarifying step isn't a warm-up — it's the part that most resembles the actual job, where a requirement arrives incomplete and the value is in the questions you ask before you build.

**Cross-question: "Agar interviewer bole 'assume whatever you want' to?"**
> Then I state my assumptions explicitly and out loud — five hundred tests, a team of six, a ten-minute PR budget, web only — and I say that if any of those change materially, specifically the scale or the number of contributors, I'd revisit specific decisions. That way the assumptions are visible and correctable, and the interviewer can steer me if I've assumed something they care about differently.

---

# 27. Design: Scalable Playwright Framework for 5000 Tests

## 27.1 Step 1 — Clarify (jo tum poochoge)

> *"Before I design, a few questions. Are these 5000 UI tests, or a mix of UI and API — because that changes the runtime by an order of magnitude. How many engineers contribute, and are they dedicated SDETs or also product developers? What's the CI feedback budget on a pull request versus pre-deploy? How many environments, and is there a production-like staging with representative data? And is this greenfield or are we scaling an existing suite that's already hurting?"*

**Maan lo interviewer ye bolta hai:** 5000 tests, ~70% API / 30% UI, 20 engineers contribute, PR budget 10 min, pre-deploy budget 30 min, 4 environments, existing suite hai jo abhi 3 ghante leti hai.

## 27.2 Step 2 — Constraints & the math

```
5000 tests
  UI  : 1500 × avg 25s = 37,500 s = 10.4 hours serial
  API : 3500 × avg 1.5s = 5,250 s = 1.5 hours serial
  TOTAL serial ≈ 12 hours

Target: 30 min pre-deploy
  => 12 h / 0.5 h = 24x parallelism minimum
  => realistically 32x (overhead, stragglers, unbalanced shards)

  8 shards × 4 workers = 32 concurrent
  Cost: 8 CI machines × ~30 min
```

**Aur ye important framing:** 5000 tests **ek hi run mein** chalane ki zaroorat hi nahi honi chahiye.

```
PR           : smoke (150)              ≈ 4 min     ← 95% of runs
Merge        : smoke + critical (600)   ≈ 9 min
Pre-deploy   : full regression (4200)   ≈ 28 min
Nightly      : everything (5000)        ≈ 40 min
```

**Ye bolna:** *"The first design decision isn't parallelism, it's not running all 5000 every time. Test selection is cheaper than compute."*

## 27.3 Step 3 — Risk analysis

| Area | Likelihood | Impact | Depth |
|---|---|---|---|
| PO approval / financial limits | Medium | Revenue + compliance | Exhaustive: UI + API + DB invariants + partial failure |
| Invoice / payment | Medium | Revenue, irreversible | Exhaustive + reconciliation checks |
| Supplier onboarding | High (many integrations) | Business blocking | Heavy API, thin UI |
| Reporting / dashboards | Medium | Trust, recoverable | API-level data correctness + one UI smoke |
| Settings / preferences | Low | Convenience | Smoke only |

## 27.4 Step 4 — High-level design

```
repo/
├── tests/                          ← LAYER 1: intent only
│   ├── procurement/
│   │   ├── api/        test_po_approval_rules.py
│   │   └── ui/         test_po_approval_journey.py
│   ├── finance/
│   ├── supplier_portal/
│   └── conftest.py                 ← suite-local fixtures
│
├── src/qa/
│   ├── pages/                      ← LAYER 2: UI domain
│   │   ├── base.py
│   │   ├── components/             table.py  header.py  modal.py
│   │   └── procurement/            po_list.py  po_detail.py
│   ├── flows/                      ← LAYER 2: multi-step facades
│   │   └── po_flows.py             create_approved_po(), etc.
│   ├── clients/                    ← LAYER 3: API clients per service
│   │   ├── base.py                 retry, correlation id, auth
│   │   ├── po_client.py
│   │   └── invoice_client.py
│   ├── core/                       ← LAYER 3: primitives
│   │   ├── interactions.py         ONE canonical way per action
│   │   ├── assertions.py           domain assertions
│   │   └── waits.py
│   ├── data/                       ← factories + builders
│   │   ├── factories.py
│   │   └── builders.py
│   └── infra/                      ← LAYER 4
│       ├── config.py               env profiles + guardrails
│       ├── auth.py                 auth strategies
│       └── logging_setup.py
│
├── conftest.py                     ← root fixtures
├── pyproject.toml                  ← markers, addopts, deps
└── ci/                             ← pipeline definitions
```

**Ownership model — 20 engineers ke liye ye critical hai:**

```
CODEOWNERS
/tests/procurement/     @procurement-team @ritik
/tests/finance/         @finance-team @ritik
/src/qa/core/           @qa-platform          ← core sirf platform team badal sakti
/src/qa/infra/          @qa-platform
/conftest.py            @qa-platform
```

**Rule:** tests **feature team** ke, framework **QA platform** ka. Ye "20 log contribute karte hain" wale problem ka structural jawab hai.

## 27.5 Step 5 — Deep dives

### (a) Auth strategy — sabse bada single speedup

```python
@pytest.fixture(scope="session")
def storage_states(browser, config, tmp_path_factory, worker_id):
    """Har ROLE ke liye ek baar login, saved state reuse."""
    import filelock, json
    root = tmp_path_factory.getbasetemp().parent
    states = {}
    for role in ("project_admin", "site_engineer", "finance", "supplier"):
        f = root / f"state_{role}.json"
        with filelock.FileLock(f"{f}.lock"):
            if not f.is_file():
                _login_and_save(browser, config, role, f)
        states[role] = str(f)
    return states

@pytest.fixture
def context(browser, config, storage_states, request):
    role = getattr(request.node.get_closest_marker("as_role"), "args", ("project_admin",))[0]
    return browser.new_context(base_url=config.base_url, storage_state=storage_states[role])
```

**Impact:** 1500 UI tests × 8s login saved = **3.3 hours** of compute removed. Ye single change hai jo sabse bada ROI deta hai.

### (b) Test data — parallel-safe by construction

- Har entity: `QA-{worker}-{test}-{uuid8}` prefix.
- Factory fixture per test, cleanup reverse-order, cleanup never fails the test.
- Nightly sweeper: `QA-*` older than 24h delete.
- Global state (feature flags, settings) **project-scoped**, global-scoped nahi.

### (c) Parallelism + balanced sharding

```yaml
strategy:
  matrix: { shard: [1,2,3,4,5,6,7,8] }
steps:
  - run: |
      pytest --splits 8 --group ${{ matrix.shard }} -n 4 \
             --durations-path .test_durations \
             -m "not quarantine" \
             --junitxml=junit-${{ matrix.shard }}.xml \
             --alluredir=allure-results
```

`.test_durations` ko nightly regenerate karke commit karo — warna sharding unbalanced ho jaayegi aur total time slowest shard ka hoga.

### (d) Flaky management at 5000 scale

```
Every run  ──> results DB (test, status, duration, shard, commit, error)
                       │
              Nightly job: last 20 runs
                       │
       flaky = same commit pe kabhi pass kabhi fail
                       │
        ┌──────────────┴──────────────┐
   flake rate > 2%              flake rate < 2%
        │                             │
   auto-quarantine +            monitor only
   ticket + owner + SLA
```

Manual flaky detection 5000 tests pe **impossible** hai. Ye automated hona hi chahiye — aur yahi 500 se 5000 ka sabse bada functional difference hai.

## 27.6 Step 6 — 500 vs 5000 vs 50000

| Dimension | 500 tests | 5000 tests | 50000 tests |
|---|---|---|---|
| **Runtime** | 20 min serial | 12 h serial | 5 days serial |
| **Parallelism** | `-n 4` on one box | 8 shards × 4 = 32 | Distributed platform, 200+ workers |
| **Selection** | Run everything | Tag-based suites | **Test impact analysis** — sirf affected tests |
| **Flaky detection** | Manual, notice ho jaata | Automated from history | ML-ish: pattern clustering, auto-quarantine |
| **Ownership** | 1-2 log, sab jaante hain | CODEOWNERS per domain | Platform team + per-team SDET embed |
| **Structure** | Flat `tests/` | Domain packages | Multi-repo ya monorepo with build graph |
| **Data** | Shared seed theek | Per-test factory mandatory | Per-test tenant/schema isolation |
| **Reporting** | pytest-html | Allure + trends | Custom dashboard + results warehouse |
| **Infra** | GitHub Actions runner | CI matrix + cache | Kubernetes, autoscaling, browser grid |
| **Cost** | Negligible | ~$500-2000/mo | ~$20-50k/mo — cost becomes a design input |
| **Bottleneck** | Test writing speed | CI compute + flakiness | Scheduling, artifact storage, human triage |
| **Biggest risk** | Coverage gaps | Flakiness eroding trust | Nobody knows what's covered |

**Ye table interview mein bahut strong hai** — dikhata hai ki tum sirf ek scale nahi, **scale ka gradient** samajhte ho.

**50000 pe kya fundamentally badal jaata hai:**
1. **Test impact analysis** — code coverage map se pata karo kaunse tests is commit se affected hain. Sab chalana economically impossible.
2. **Human triage bottleneck** — 50000 tests pe 1% failure = 500 failures. Koi insaan nahi dekh sakta. Auto-classification mandatory.
3. **Coverage visibility** — "kya cover hai" ka jawab dena ek product ban jaata hai.

> **Interview answer:** For five thousand tests my first design decision isn't parallelism, it's selection — nobody should run all five thousand on every pull request. I'd compose suites by marker: a hundred-and-fifty test smoke suite under five minutes gating PRs, smoke plus critical on merge, full regression pre-deploy, everything nightly. Then the math: five thousand tests at roughly twelve hours serial needs about thirty-two-way concurrency to hit a thirty-minute pre-deploy budget, which I'd get from eight CI shards times four xdist workers, sharded by historical durations so they're balanced — otherwise total time is the slowest shard. Structurally it's layered: tests contain only intent, page objects and flow facades own the UI, per-service API clients and interaction primitives sit below, and infra holds fixtures and config. The single biggest performance win is auth: log in once per role, save storage state, and reuse it, which removes several hours of compute across fifteen hundred UI tests. With twenty contributors, the thing that actually determines whether this survives is the ownership model — tests are owned by the feature teams via CODEOWNERS, while the core and infra layers are owned by a small platform group, because a shared core edited by twenty people degrades fast. And flaky management has to be automated at this size: every run writes to a results database, a nightly job computes flake rate per test over the last twenty runs, and anything above two percent gets auto-quarantined with a ticket, an owner and an SLA. At five hundred tests you'd notice flakiness by hand; at five thousand you won't, and by fifty thousand the whole model changes again — you need test impact analysis to select from the code diff, because running everything stops being economically possible.

**Cross-question: "Ek shard fail ho gaya — poora run fail?"**
> No, I set `fail-fast: false` so every shard completes and I get the full failure picture in one pass rather than discovering failures one shard at a time across three reruns. The merge job aggregates all the JUnit XML and Allure results and computes the overall verdict. I do distinguish a shard that failed because tests failed from a shard that died because the runner was evicted — the latter is an infrastructure retry, and I retry that shard specifically rather than the whole run.

**Cross-question: "20 engineers contribute kar rahe hain — quality kaise maintain karoge?"**
> Structurally, not socially. CODEOWNERS so framework changes need platform review while test changes need only team review. Lint rules that fail CI on the things that actually rot a suite — bare excepts, `time.sleep`, raw selectors inside `tests/`, unregistered markers. A template test and a short contributing guide so the default path is the right one. And a review checklist that's five items, not fifty. What I would not rely on is documentation and good intentions — with twenty contributors, anything not enforced by CI will drift within a quarter.

---

# 28. Design: Automation Platform for 10000 Tests

> **Ye sawaal alag hai.** Section 27 ek **framework** ka design tha (code kaise organise ho). Ye ek **platform** ka design hai (execution infrastructure as a product). Interviewer dekh raha hai ki tum distributed systems ki bhasha bol sakte ho ya nahi.

## 28.1 Step 1 — Clarify

> *"A few things first. Is this a platform serving multiple teams and multiple repos, or one team's suite that's grown? Are tests UI-heavy or API-heavy — that drives whether I need a browser grid at all. What's the required feedback time, and is that per-run or per-team? Do we control the infrastructure — Kubernetes available, or is this cloud-vendor only? And what's the budget envelope, because at this scale compute cost becomes a design constraint, not an afterthought."*

**Assume:** multi-team platform, 10000 tests (60% UI), 15 min target for a full run, Kubernetes available, ~$15k/month budget.

## 28.2 Architecture diagram

```
                          ┌──────────────────────────────┐
   git push / cron /      │        TRIGGER LAYER          │
   manual / API  ────────>│  CI webhook, scheduler, CLI   │
                          └───────────────┬──────────────┘
                                          │ run request
                                          v
        ┌──────────────────────────────────────────────────────────┐
        │                    ORCHESTRATOR / SCHEDULER               │
        │  - resolve test selection (tags, impact analysis)         │
        │  - fetch historical durations from Results DB             │
        │  - build BALANCED shards (bin-packing by duration)        │
        │  - create run record, emit N shard jobs                   │
        │  - track run state, handle straggler / worker death       │
        └───────────────┬───────────────────────────┬──────────────┘
                        │ enqueue shard jobs        │ reads
                        v                           v
        ┌───────────────────────────┐   ┌──────────────────────────┐
        │      QUEUE (SQS/Redis/    │   │  TEST METADATA STORE     │
        │      RabbitMQ/K8s Jobs)   │   │  test list, tags, owners,│
        │  visibility timeout,      │   │  historical durations,   │
        │  DLQ for poison shards    │   │  flake rates             │
        └───────────┬───────────────┘   └──────────────────────────┘
                    │ pull
        ┌───────────v────────────────────────────────────────────┐
        │            WORKER POOL  (K8s Deployment + HPA/KEDA)     │
        │  ┌────────┐ ┌────────┐ ┌────────┐        ┌────────┐    │
        │  │worker-1│ │worker-2│ │worker-3│  ....  │worker-N│    │
        │  │pytest  │ │pytest  │ │pytest  │        │pytest  │    │
        │  │ -n 4   │ │ -n 4   │ │ -n 4   │        │ -n 4   │    │
        │  └───┬────┘ └───┬────┘ └───┬────┘        └───┬────┘    │
        │      │ autoscale on queue depth (KEDA)       │         │
        └──────┼──────────┼──────────┼────────────────┼─────────┘
               │          │          │                │
      browser  │          │          │                │  results +
      sessions v          v          v                v  artifacts
        ┌────────────────────────────────┐   ┌─────────────────────────┐
        │        BROWSER GRID             │   │   RESULTS INGESTION     │
        │  Playwright server pods /       │   │   (API + queue buffer)  │
        │  Selenium Grid / cloud vendor   │   │            │            │
        │  - session pooling              │   │            v            │
        │  - per-session isolation        │   │   ┌──────────────────┐  │
        │  - video/trace capture          │   │   │  RESULTS DB      │  │
        └────────────────────────────────┘   │   │  (Postgres +     │  │
                    │ artifacts             │   │   partitioning)  │  │
                    v                       │   └────────┬─────────┘  │
        ┌────────────────────────────────┐  │            │            │
        │   ARTIFACT STORE (S3/GCS)      │  └────────────┼────────────┘
        │  screenshots, traces, videos,  │               │
        │  logs, HAR                     │               v
        │  lifecycle: 7d / 30d / 1y      │   ┌──────────────────────────┐
        └────────────────────────────────┘   │  ANALYTICS & REPORTING   │
                                             │  - run verdict API        │
        ┌────────────────────────────────┐   │  - Allure / dashboard     │
        │  PLATFORM OBSERVABILITY        │<──│  - flaky detector (nightly)│
        │  Prometheus + Grafana, traces  │   │  - Slack/PR notifier      │
        │  SLO: queue wait, run duration │   │  - cost attribution       │
        └────────────────────────────────┘   └──────────────────────────┘
```

## 28.3 Scheduling & dynamic sharding

**Naive sharding (alphabetical / round-robin) ka problem:**

```
Shard 1: [ 2s, 3s, 1s, 2s ]        = 8s     ← 14 min idle
Shard 2: [ 800s, 60s, 30s ]        = 890s   ← poora run 890s ka
Total run time = max(shards) = 890s
```

**Duration-aware bin packing (LPT — Longest Processing Time first):**

```python
def build_shards(tests: list[dict], shard_count: int) -> list[list[str]]:
    """tests: [{"id": ..., "p95_duration_s": ...}]  — history se
    LPT greedy: sabse lamba test sabse khaali shard mein daalo.
    Optimal ke ~4/3 ke andar rehta hai, aur O(n log n) hai."""
    ordered = sorted(tests, key=lambda t: -t["p95_duration_s"])
    shards = [{"tests": [], "total": 0.0} for _ in range(shard_count)]
    for t in ordered:
        target = min(shards, key=lambda s: s["total"])
        target["tests"].append(t["id"])
        target["total"] += t["p95_duration_s"]
    return [s["tests"] for s in shards]
```

**Kyun p95, average nahi:** average ek slow outlier ko chhupa deta hai. Agar ek test usually 5s leta hai lekin 5% baar 90s, average 9s dikhega aur shard planning galat hogi. p95 realistic worst case deta hai.

**Naye tests jinke paas history nahi:** unhe median duration assign karo, aur pehle run ke baad real data aa jaayega.

**Straggler handling:** agar 7 shards khatam ho gaye aur 1 abhi chal raha hai, orchestrator uske bache hue tests ko **re-shard** karke idle workers ko de sakta hai (work stealing). Ye advanced hai lekin bolne layak.

## 28.4 Worker autoscaling

```yaml
# KEDA — queue depth pe scale
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata: { name: test-workers }
spec:
  scaleTargetRef: { name: test-worker-deployment }
  minReplicaCount: 2            # warm pool — cold start avoid
  maxReplicaCount: 200          # budget ceiling
  cooldownPeriod: 300           # 5 min — thrashing avoid
  triggers:
    - type: aws-sqs-queue
      metadata:
        queueURL: https://sqs.../test-shards
        queueLength: "1"        # 1 shard per worker
```

**Design decisions jo bolne layak hain:**

| Decision | Reason |
|---|---|
| `minReplicaCount: 2` | Warm pool — pehle shard ko cold start (image pull ~60s) na jhelna pade |
| `maxReplicaCount` = budget ceiling | Runaway scaling se bill blowup na ho |
| `cooldownPeriod: 300` | Scale-down thrashing rokta hai; ek run ke steps ke beech pods na maren |
| Spot/preemptible instances | 60-70% saste. Tests idempotent hain, worker mar jaaye to shard requeue |
| Visibility timeout > max shard duration | Warna queue shard ko "lost" maankar duplicate kar degi |

**Worker death handling:** queue visibility timeout expire → message wapas visible → doosra worker uthata hai. Shard **idempotent** hona chahiye (results write mein `run_id + test_id` unique key). Poison shard (jo baar-baar worker maar raha hai) 3 attempts ke baad DLQ mein.

## 28.5 Browser grid options

| Option | Pros | Cons | Kab |
|---|---|---|---|
| **Browser in worker pod** | Simplest, no network hop, fastest | Har worker ko heavy image + RAM | **Default choice** — <200 workers |
| **Playwright server** (`playwright run-server`) | Browser pods alag scale ho sakte, worker light | Network hop, WS connection management | Worker/browser ka ratio alag chahiye |
| **Selenium Grid 4** | Multi-language, mature, hub/node model | Heavier, Playwright ke liye natural nahi | Mixed Selenium + Playwright estate |
| **Cloud (BrowserStack/LambdaTest/Sauce)** | Real devices, real browsers, zero ops | Expensive at 10k scale, network latency, less control | Cross-browser matrix, real-device needs |

**Ye bolna:** *"At ten thousand tests I'd default to browsers inside the worker pod — it's the simplest topology and avoids a network hop on every action, which matters when you have millions of actions. I'd move to a separate Playwright server tier only if I find worker CPU and browser memory scale differently, which is when decoupling actually pays. I'd use a cloud vendor for the cross-browser and real-device matrix specifically — running the full ten thousand there would be prohibitively expensive, but running a two-hundred-test cross-browser subset there is exactly right."*

## 28.6 Results DB schema

```sql
-- runs: ek CI run
CREATE TABLE runs (
  run_id          UUID PRIMARY KEY,
  started_at      TIMESTAMPTZ NOT NULL,
  finished_at     TIMESTAMPTZ,
  trigger         TEXT NOT NULL,          -- pr | merge | nightly | manual
  git_sha         TEXT NOT NULL,
  branch          TEXT NOT NULL,
  environment     TEXT NOT NULL,
  suite_selector  TEXT,                   -- "-m 'smoke or critical'"
  shard_count     INT,
  verdict         TEXT,                   -- pass | fail | infra_error
  total_cost_usd  NUMERIC(10,4)
);

-- test_results: ek test ka ek execution  (BIG table -> partition by month)
CREATE TABLE test_results (
  id              BIGSERIAL,
  run_id          UUID NOT NULL REFERENCES runs(run_id),
  test_id         TEXT NOT NULL,          -- "tests/procurement/test_po.py::test_approval"
  status          TEXT NOT NULL,          -- passed|failed|skipped|error|flaky
  duration_ms     INT NOT NULL,
  shard_index     INT,
  worker_id       TEXT,
  attempt         SMALLINT DEFAULT 1,
  error_type      TEXT,                   -- AssertionError | TimeoutError | ConnectionError
  error_message   TEXT,
  failure_class   TEXT,                   -- product_defect | infra | test_data | unknown
  correlation_id  TEXT,
  artifacts       JSONB,                  -- {"screenshot": "s3://...", "trace": "s3://..."}
  started_at      TIMESTAMPTZ NOT NULL,
  PRIMARY KEY (id, started_at)
) PARTITION BY RANGE (started_at);

CREATE INDEX ON test_results (test_id, started_at DESC);
CREATE INDEX ON test_results (run_id);
CREATE INDEX ON test_results (status, started_at DESC) WHERE status IN ('failed','error');

-- test_registry: har test ki current metadata
CREATE TABLE test_registry (
  test_id           TEXT PRIMARY KEY,
  owner_team        TEXT NOT NULL,
  tags              TEXT[],
  p50_duration_ms   INT,
  p95_duration_ms   INT,                  -- <- sharding isi se
  flake_rate_30d    NUMERIC(5,4),
  quarantined       BOOLEAN DEFAULT FALSE,
  quarantine_ticket TEXT,
  quarantine_expiry DATE,
  last_seen_at      TIMESTAMPTZ
);
```

**Scale ka concern:** 10000 tests × ~20 runs/day = **200k rows/day**, ~6M/month. Isliye `PARTITION BY RANGE (started_at)` — purane partitions detach karke cold storage mein bhej do. Raw rows 90 din, uske baad daily aggregate table.

## 28.7 Flaky detection from history

```sql
-- Same git_sha pe kabhi pass kabhi fail = definitionally flaky
WITH by_commit AS (
  SELECT tr.test_id, r.git_sha,
         COUNT(*) FILTER (WHERE tr.status = 'passed') AS passes,
         COUNT(*) FILTER (WHERE tr.status IN ('failed','error')) AS fails
  FROM test_results tr JOIN runs r USING (run_id)
  WHERE tr.started_at > now() - INTERVAL '14 days'
  GROUP BY 1, 2
)
SELECT test_id,
       COUNT(*) FILTER (WHERE passes > 0 AND fails > 0) AS flaky_commits,
       COUNT(*)                                          AS total_commits,
       ROUND(COUNT(*) FILTER (WHERE passes > 0 AND fails > 0)::numeric
             / COUNT(*), 4)                              AS flake_rate
FROM by_commit
GROUP BY test_id
HAVING COUNT(*) FILTER (WHERE passes > 0 AND fails > 0) > 0
ORDER BY flake_rate DESC;
```

**Ye definition strong hai kyunki:** same commit pe alag result = code ka farq nahi hai = test ya environment ka issue. Ye "failed then passed on retry" se behtar signal hai.

**Automated response pipeline:**

```
flake_rate > 5%   -> auto-quarantine + ticket + Slack to owner team + 14-day SLA
flake_rate 2-5%   -> flag in report, weekly digest to owner
flake_rate < 2%   -> monitor only
quarantine expired -> CI collection fails, forces decision
```

**Aur ek useful query — error clustering:** `failure_class` aur `error_type` pe group karke dekho ki 40 failures mein se 30 ek hi `ConnectionError` hain — matlab infra issue hai, 30 alag bugs nahi.

## 28.8 Artifact storage & retention

| Artifact | Size (avg) | Retention | Reason |
|---|---|---|---|
| JUnit XML / results JSON | 5 KB | 1 year | Trend analysis, cheap |
| Screenshot (failure only) | 300 KB | 30 days | Debugging window |
| Playwright trace | 3–8 MB | 7 days | Huge; only needed while investigating |
| Video (failure only) | 5–20 MB | 7 days | Huge |
| Logs (per test) | 50 KB | 30 days | Correlate with backend |

**Math:** 10000 tests × 2% failure × 20 runs/day × 8 MB trace ≈ **32 GB/day**. Bina lifecycle policy ke ye mahine mein ~1 TB. S3 lifecycle rules mandatory:

```
day 0-7   : S3 Standard
day 7-30  : S3 Infrequent Access (screenshots, logs only; traces/videos deleted)
day 30-365: Glacier (results JSON only)
day 365+  : delete
```

**Sirf failure pe capture karo** — `--tracing=retain-on-failure`. Har test ka trace rakhna 50x cost hai aur 99% kabhi nahi dekha jaata.

## 28.9 Platform observability (platform khud bhi ek product hai)

```
Platform SLIs                              SLO target
─────────────────────────────────────────────────────────
Queue wait time (p95)                      < 60 s
Run duration (p95, full regression)        < 15 min
Worker startup time (p95)                  < 45 s
Platform-caused failure rate               < 0.5 %
Result ingestion lag (p95)                 < 30 s
Artifact upload success rate               > 99.9 %
```

**Grafana dashboard rows:**
1. **Run health** — runs/hour, pass rate, verdict distribution
2. **Latency** — queue wait, run duration, shard skew (max/min shard time ratio)
3. **Capacity** — active workers, queue depth, worker utilisation %
4. **Reliability** — worker crash rate, DLQ depth, retry rate
5. **Quality** — flake rate trend, quarantine count, coverage-by-team
6. **Cost** — $/run, $/test, spend by team

**Critical distinction jo bolna hai:** *"I separate 'the tests failed' from 'the platform failed'. Platform-caused failures are an SLO breach and my responsibility; product failures are the platform working correctly. Conflating them is how teams stop trusting the platform — every infrastructure blip looks like a product regression, and eventually people ignore red."*

## 28.10 Cost

```
Assumption: 10000 tests, 20 runs/day, avg 15 min run, 100 workers peak

Compute (spot, 4 vCPU / 8 GB ≈ $0.05/hr):
  100 workers × 0.25 h × 20 runs × 30 days × $0.05  = $  750/mo
Storage (S3 with lifecycle):                          = $  200/mo
Results DB (RDS Postgres, medium):                    = $  300/mo
Grid / cloud browsers (cross-browser subset only):    = $1,500/mo
Observability (Grafana Cloud / Datadog):              = $  400/mo
                                                        ─────────
                                                       ~$3,150/mo
```

**Cost optimisation levers (priority order):**
1. **Don't run everything every time** — selection is 10x cheaper than any infra optimisation.
2. **Spot instances** — 60-70% saving; tests are idempotent so eviction is safe.
3. **Artifact lifecycle** — traces/videos only on failure, deleted after 7 days.
4. **Auth via storage state** — removes hours of compute.
5. **API-level tests where UI adds no signal** — 15x cheaper per test.
6. **Right-size workers** — measure; over-provisioned RAM is silent waste.

**Cost attribution per team** ek underrated feature hai: har team dekhti hai ki uske tests ka kitna kharcha hai. Ye behaviour badalta hai — team khud slow tests optimise karti hai.

> **Interview answer:** At ten thousand tests I stop designing a framework and start designing a platform. The core pipeline is: a trigger creates a run request; an orchestrator resolves test selection, pulls historical p95 durations from a results database, and builds balanced shards using longest-processing-time bin packing; shards go onto a queue; a Kubernetes worker pool autoscaled by queue depth pulls them; workers either embed browsers or connect to a browser grid; results and artifacts stream to a results database and object storage; and a reporting layer produces the verdict plus nightly flaky analysis. Several details matter. Sharding by p95 rather than mean, because a mean hides a slow outlier and total run time is the slowest shard. Queue visibility timeout longer than the maximum shard duration, and idempotent result writes, so a worker dying just requeues the shard — which is what lets me run on spot instances at sixty percent lower cost. Flaky detection defined as a test that both passes and fails on the same commit, because that isolates non-determinism from real regressions; above five percent it's auto-quarantined with a ticket, an owner and an SLA. Artifacts need aggressive lifecycle policies — traces are several megabytes each and at this volume that's tens of gigabytes a day, so traces and videos are failure-only and deleted after a week. And I'd instrument the platform itself with SLOs — queue wait, run duration, and specifically the platform-caused failure rate as distinct from product failures, because if people can't tell those apart they stop trusting red.

**Cross-question: "Worker mid-run mar gaya to?"**
> The queue's visibility timeout expires and another worker picks the shard up, so the work isn't lost. Two things make that safe: shards must be independently runnable, which they are because tests create their own data, and result writes must be idempotent, which I get from a unique key on run id plus test id plus attempt. I also cap retries — a shard that kills three workers is a poison shard and goes to a dead-letter queue with an alert, because it's usually an OOM or a crash loop, not bad luck. The run then reports as infrastructure-incomplete rather than silently missing a shard's worth of coverage, which is the failure mode I'd most want to avoid.

**Cross-question: "Ye platform banana justify kaise karoge? Build vs buy?"**
> By counting the cost of not having it. If a hundred engineers each lose twenty minutes a day to slow or untrustworthy CI, that's a large recurring cost that dwarfs the platform's build cost, and it compounds because slow feedback changes behaviour — people batch changes and stop running tests. That said, I'd buy before I build: a hosted CI with good matrix support plus a cloud browser vendor gets you most of the way at ten thousand tests, and the orchestration layer is the only piece that's genuinely hard to buy. I'd build the scheduler and results database, and rent the compute and the browsers.

---

# 29. Design: QA for a Payment System

> **Ye sawaal risk-first sochne ka test hai.** Agar tumne "main login, card entry, aur success page test karunga" bola — junior. Senior wahan se shuru karta hai jahan **paisa cut gaya lekin order nahi bana.**

## 29.1 Step 1 — Clarify

> *"Which parts do we own? Are we the merchant integrating a PSP like Stripe or Razorpay, or are we building the payment processor itself — that changes almost everything. Which payment methods: cards, UPI, netbanking, wallets? Do we store card data or is it tokenized by the gateway, because that determines our PCI scope. Are there refunds, partial refunds, subscriptions, and multi-currency? And what's the current pain — are we seeing failed payments, reconciliation mismatches, or chargebacks?"*

**Assume:** merchant side, PSP-integrated, cards + UPI, tokenized (no card storage), refunds and partial refunds exist, single currency.

## 29.2 Step 2 — Risk analysis FIRST (yahi answer ka core hai)

```
Payment system mein failure modes ko severity se sort karo:

  SEV-1  Money moved, system doesn't know
         → charge succeeded, order not created
         → customer charged, no goods; support ticket; chargeback; trust dead
         → IRREVERSIBLE without manual intervention

  SEV-1  Double charge (idempotency failure)
         → retry created a second charge
         → regulatory + reputational

  SEV-1  Wrong amount charged
         → currency/paise conversion bug, tax rounding

  SEV-2  Money not moved, system thinks it did
         → order fulfilled without payment → revenue leak

  SEV-2  Refund fails silently / refunds twice

  SEV-3  Payment declines that should succeed (conversion loss)

  SEV-4  UI issues on the payment page
```

**Ye bolna:** *"Notice that the highest-severity failures are all state-divergence between us and the payment provider — not UI bugs. So that's where I'd put the testing depth. The classic happy-path payment test has the lowest value per unit of effort in this whole system."*

## 29.3 Partial failure — the highest-value test

```
        ┌─────────┐   1. authorise    ┌──────────────┐
        │ Checkout│ ────────────────> │     PSP       │
        │ Service │ <──────────────── │  (Stripe/RP)  │
        └────┬────┘   2. charge OK    └──────────────┘
             │
             │ 3. create order   ┌──────────────┐
             └─────────────────> │   Order      │  ❌ FAILS HERE
                                 │   Service    │     (DB down, timeout,
                                 └──────────────┘      validation, deploy)
             
        RESULT: customer's money is gone. There is no order.
        The system has NO RECORD that anything is wrong.
```

**Ye scenario test karne ke tareeke:**

```python
def test_charge_succeeds_but_order_creation_fails(psp_sandbox, order_service, api):
    """SEV-1: partial failure must be detected and compensated."""
    # Order service ko deliberately fail karao
    with order_service.fault_injection(fail_on="POST /orders", error=500):
        resp = api.post("/checkout", json={
            "cart_id": cart["id"],
            "payment_method": psp_sandbox.token_for("4242424242424242"),
            "idempotency_key": idem_key,
        })

    # 1. Customer ko honest error mile — "success" nahi
    assert resp.status_code == 502
    assert resp.json()["code"] == "ORDER_CREATION_FAILED"

    # 2. Charge ko void/refund kar diya gaya ho (compensating action)
    charge = psp_sandbox.get_charge_for(idem_key)
    assert charge["status"] in ("voided", "refunded"), (
        f"MONEY STUCK: charge {charge['id']} is {charge['status']} with no order"
    )

    # 3. Ya, agar auto-compensation nahi hai — reconciliation queue mein entry ho
    pending = api.get("/internal/reconciliation/pending").json()
    assert any(p["idempotency_key"] == idem_key for p in pending), \
        "Orphan charge not queued for reconciliation — it will never be found"

    # 4. Audit trail mein poori kahani ho
    events = api.get(f"/internal/audit?idempotency_key={idem_key}").json()
    assert [e["type"] for e in events] == [
        "checkout.started", "payment.authorised", "payment.captured",
        "order.creation_failed", "payment.voided",
    ]
```

**Ye ek test poore payment QA ki philosophy dikhata hai.** Interview mein ye likhkar dikha sakte ho.

**Failure injection points jo test karne chahiye:**

| Kahan fail | Expected behaviour |
|---|---|
| PSP timeout after charge sent (response lost) | Idempotency key se status query karo, blind retry mat karo |
| Order service down after capture | Void/refund, ya reconciliation queue |
| Webhook never arrives | Polling fallback picks it up within SLA |
| Webhook arrives twice | Idempotent consumer — ek hi order |
| Webhook arrives out of order (captured before authorised) | State machine rejects invalid transition |
| Network partition mid-capture | Reconciliation catches within next cycle |
| Refund partially succeeds | Amounts reconcile; no double refund |

## 29.4 Idempotency

```python
def test_retry_with_same_idempotency_key_does_not_double_charge(api, psp_sandbox):
    key = f"idem-{uuid.uuid4()}"
    payload = {"cart_id": cart["id"], "amount_paise": 250_000,
               "payment_method": token, "idempotency_key": key}

    r1 = api.post("/checkout", json=payload)
    r2 = api.post("/checkout", json=payload)          # exact retry
    r3 = api.post("/checkout", json=payload)          # aur ek

    assert r1.json()["order_id"] == r2.json()["order_id"] == r3.json()["order_id"]
    charges = psp_sandbox.list_charges(idempotency_key=key)
    assert len(charges) == 1, f"DOUBLE CHARGE: {len(charges)} charges for one key"


def test_same_key_different_amount_is_rejected(api):
    """Idempotency key reuse with different payload = client bug. Must not silently
    return the old result — that would hide a real error."""
    key = f"idem-{uuid.uuid4()}"
    api.post("/checkout", json={**base, "amount_paise": 250_000, "idempotency_key": key})
    r = api.post("/checkout", json={**base, "amount_paise": 500_000, "idempotency_key": key})
    assert r.status_code == 422
    assert r.json()["code"] == "IDEMPOTENCY_KEY_REUSED_WITH_DIFFERENT_PAYLOAD"
```

**Idempotency ke 4 rules jo test karne hain:**
1. Same key + same payload → same result, ek hi side effect.
2. Same key + different payload → **error**, chup-chaap purana result nahi.
3. Key ka TTL hona chahiye (usually 24h) — aur expiry ke baad behaviour defined ho.
4. Concurrent requests same key ke saath → ek jeete, doosra wait kare ya conflict return kare (race test).

```python
def test_concurrent_requests_same_idempotency_key(api):
    """Race: 5 parallel requests, ek hi charge."""
    from concurrent.futures import ThreadPoolExecutor
    key = f"idem-{uuid.uuid4()}"
    with ThreadPoolExecutor(5) as ex:
        results = list(ex.map(lambda _: api.post("/checkout", json={**base, "idempotency_key": key}), range(5)))
    order_ids = {r.json().get("order_id") for r in results if r.ok}
    assert len(order_ids) == 1, f"Race produced {len(order_ids)} orders"
```

## 29.5 Reconciliation

**Reconciliation = roz ek job jo humara record aur PSP ka record compare karta hai.** Ye payment system ka **safety net** hai — aur QA ko ye khud test karna chahiye.

```
   Our DB (payments table)          PSP settlement report (daily file/API)
   ────────────────────────         ──────────────────────────────────────
   txn_1  ₹2500  CAPTURED           txn_1  ₹2500  captured     ✅ match
   txn_2  ₹1800  CAPTURED           (missing)                  ⚠️ we think captured, PSP doesn't
   (missing)                        txn_3  ₹4200  captured     🔴 ORPHAN CHARGE — customer charged,
                                                                  we have no record
   txn_4  ₹900   CAPTURED           txn_4  ₹950   captured     🔴 AMOUNT MISMATCH
   txn_5  ₹1200  REFUNDED           txn_5  ₹1200  captured     🔴 refund never reached PSP
```

```python
def test_reconciliation_detects_orphan_charge(psp_sandbox, recon_job, db):
    # PSP pe charge banao lekin humare DB mein record mat banao
    charge = psp_sandbox.create_charge(amount_paise=420_000, metadata={"synthetic": "qa"})

    report = recon_job.run(for_date=date.today())

    assert charge["id"] in [d["psp_reference"] for d in report["discrepancies"]]
    d = next(d for d in report["discrepancies"] if d["psp_reference"] == charge["id"])
    assert d["type"] == "ORPHAN_CHARGE"
    assert d["severity"] == "CRITICAL"
    assert d["alerted"] is True          # PagerDuty/Slack gaya


def test_reconciliation_detects_amount_mismatch(psp_sandbox, recon_job, db): ...
def test_reconciliation_detects_missing_refund(psp_sandbox, recon_job, db): ...
def test_reconciliation_is_idempotent(recon_job):
    """Do baar chalane se duplicate discrepancy records na banein."""
```

**Reconciliation ko test karna aksar bhool jaate hain** — aur wahi wo cheez hai jo production mein sabse zyada value deti hai. Ye bolna strong hai.

## 29.6 Audit trail

```python
def test_audit_trail_is_complete_and_immutable(api, db):
    order = complete_purchase(amount_paise=250_000)
    events = api.get(f"/internal/audit?order_id={order['id']}").json()

    # 1. Poora lifecycle capture hua
    assert [e["type"] for e in events] == [
        "checkout.started", "payment.authorised", "payment.captured",
        "order.created", "order.confirmed",
    ]
    # 2. Har event mein who/what/when/where
    for e in events:
        assert e["actor"] and e["timestamp"] and e["correlation_id"]
        assert "card_number" not in json.dumps(e)      # PII/PCI leak check
        assert "cvv" not in json.dumps(e).lower()
    # 3. Immutable — update/delete allowed nahi
    r = api.patch(f"/internal/audit/{events[0]['id']}", json={"type": "hacked"})
    assert r.status_code in (403, 405)
```

**Audit trail ka business reason:** chargeback dispute mein tumhe **prove** karna padta hai ki kya hua. Missing audit = dispute haar gaye. Ye compliance requirement hai, nice-to-have nahi.

## 29.7 PCI / tokenization

| Rule | Test |
|---|---|
| Card number kabhi humare server pe na aaye | Network capture — PAN sirf PSP domain pe jaana chahiye (iframe/hosted fields) |
| Logs mein PAN/CVV kabhi nahi | Log scanner test with a known test PAN |
| DB mein sirf token + last4 + brand | Schema check + data scan |
| Screenshots/traces mein card data nahi | Test artifacts scan; masked input verify |
| TLS everywhere | Reject plain HTTP; check HSTS |

```python
def test_card_number_never_reaches_our_backend(page, network_log):
    page.goto("/checkout")
    page.frame_locator("#psp-card-iframe").get_by_label("Card number").fill("4242424242424242")
    page.get_by_role("button", name="Pay").click()

    our_requests = [r for r in network_log if "merlin.app" in r.url]
    for r in our_requests:
        body = r.post_data or ""
        assert "4242424242424242" not in body, f"PAN leaked to our backend in {r.url}"
        assert not re.search(r"\b\d{13,19}\b", body), f"Possible PAN in {r.url}"
```

**Tokenization ka matlab:** card details **kabhi** tumhare server ko touch nahi karte — browser se seedha PSP ko jaate hain (iframe / hosted fields), aur tumhe ek token milta hai. Isse tumhara **PCI scope** SAQ-D se SAQ-A tak gir jaata hai — ye ek massive compliance cost saving hai. Ye jaanna business awareness dikhata hai.

## 29.8 Sandbox test cards

| Scenario | Typical test card (Stripe-style) |
|---|---|
| Success | 4242 4242 4242 4242 |
| Generic decline | 4000 0000 0000 0002 |
| Insufficient funds | 4000 0000 0000 9995 |
| Lost/stolen card | 4000 0000 0000 9987 |
| Expired card | 4000 0000 0000 0069 |
| Incorrect CVC | 4000 0000 0000 0127 |
| Processing error | 4000 0000 0000 0119 |
| 3DS required | 4000 0025 0000 3155 |
| 3DS failure | 4000 0000 0000 3220 |
| Fraud/risk block | 4100 0000 0000 0019 |
| Dispute/chargeback | 4000 0000 0000 0259 |

**Test matrix ka rule:** har decline code ke liye **user-facing message** verify karo. "Insufficient funds" aur "card expired" ka message alag hona chahiye — warna user ko pata nahi chalega kya karna hai, aur conversion girta hai. Ye QA se product-thinking ka signal hai.

**3DS/OTP flows** ko mat chhodo — India mein RBI mandate ki wajah se ye default path hai, exception nahi.

## 29.9 Production monitoring (test suite se zyada important)

```
Business metrics (alert on anomaly, not threshold):
  - payment success rate       — 5-min window vs 7-day baseline
  - success rate by method     — UPI alag, cards alag (ek gir sakta hai)
  - success rate by issuer/bank — ek bank ka outage detect ho jaata hai
  - avg time to capture
  - refund success rate

Integrity metrics (these are the SEV-1 detectors):
  - orphan charges (PSP has it, we don't)      → alert immediately, page on-call
  - orphan orders (we have it, PSP doesn't)
  - reconciliation discrepancy count & amount
  - stuck payments (>15 min in PENDING)
  - webhook processing lag
  - duplicate charge count                     → must be zero
```

**Synthetic monitoring:** har 5 minute mein production pe ek real ₹1 transaction (aur uska auto-refund). Ye tumhe **real user se pehle** pata chala deta hai ki payments down hain.

**Ye bolna:** *"For payments, production monitoring matters more than the test suite. A test suite tells you the code was correct when it shipped. Monitoring tells you money is moving correctly right now — and payment failures are usually caused by things a test suite can't see: an issuer outage, a PSP config change, a certificate expiry, a fraud rule tuned too aggressively."*

> **Interview answer:** I'd design payment QA risk-first rather than feature-first. The highest-severity failure isn't a broken UI — it's state divergence between us and the payment provider, and the worst case is the charge succeeding while order creation fails, because the customer's money is gone and the system has no record anything is wrong. So my highest-value test injects a fault into the order service after a successful capture and asserts three things: the customer gets an honest error rather than a success page, the charge is voided or queued for reconciliation, and the audit trail records the full sequence. Around that I'd test idempotency hard — same key and same payload returns the same result with exactly one charge, same key with a different payload is rejected rather than silently returning the old result, and concurrent requests with the same key produce one order. I'd test the reconciliation job itself by planting synthetic discrepancies at the provider and asserting the job detects orphan charges, amount mismatches and missing refunds, because reconciliation is the safety net and an untested safety net isn't one. On compliance, I'd verify card data never touches our backend by asserting no PAN appears in any request to our domain, since tokenization is what keeps our PCI scope small. And I'd argue that for payments, production monitoring outweighs the test suite: success rate by method and by issuer, stuck-payment counts, duplicate-charge count which must be zero, plus a synthetic one-rupee transaction every five minutes. Most payment incidents come from things a suite structurally can't see — an issuer outage, a fraud rule change, a certificate expiry.

**Cross-question: "Refund flow mein kya test karoge?"**
> Full and partial refunds, refunding more than the captured amount which must be rejected, refunding twice with the same idempotency key producing one refund, refunding a payment that was never captured, and a refund that fails at the provider — where the key question is whether our state stays consistent or we mark it refunded optimistically. That last one is the same partial-failure class as the checkout case, just in the opposite direction, and it leaks money instead of trapping it.

**Cross-question: "Production mein payment test kaise karoge — real paisa?"**
> Synthetic transactions with a real card at the smallest chargeable amount, immediately auto-refunded, tagged in metadata so reconciliation and business reporting exclude them. I'd run those from a dedicated monitoring account, not a customer account, and alert on the synthetic failing rather than on the amount. For anything beyond a smoke check, the provider's sandbox is the right place — the value of the production synthetic is proving the whole live chain works end to end, not covering scenarios.

---

# 30. Design: Test Data Management at Scale

## 30.1 Step 1 — Clarify

> *"How many environments and how many teams share them? Is the data volume the issue, the conflicts between teams, or the time to provision? Do we have production-scale data anywhere, and are we under any data-protection regulation? And is the current pain 'tests interfere with each other' or 'we can't reproduce a production bug because we don't have realistic data' — those need different solutions."*

## 30.2 The four-layer model

```
┌────────────────────────────────────────────────────────────────────────┐
│ L1  STATIC SEED — ships with the environment, version controlled        │
│     Master data: currencies, tax codes, roles, permissions, categories  │
│     Owned by: platform/infra, applied by migration                     │
│     Lifecycle: recreated on env rebuild                                 │
│     Rule: tests READ only. A test that mutates seed data is a bug.      │
│     Cost: cheap. Isolation: none (shared) — hence read-only.            │
├────────────────────────────────────────────────────────────────────────┤
│ L2  PER-SUITE — session/module fixture                                  │
│     Expensive shared setup: a project, a site, an approved supplier     │
│     Lifecycle: created at suite start, cleaned at suite end             │
│     Rule: read-mostly. If a test mutates it, it must restore it.        │
│     Cost: medium. Isolation: within suite only — parallel risk.         │
├────────────────────────────────────────────────────────────────────────┤
│ L3  PER-TEST FACTORY — function fixture       ← 90% of data lives here  │
│     Every test creates its own PO, invoice, user with UUID identifiers  │
│     Lifecycle: created in setup, deleted in teardown (reverse order)    │
│     Rule: never reference another test's data.                          │
│     Cost: a few API calls. Isolation: excellent. Parallel-safe.         │
├────────────────────────────────────────────────────────────────────────┤
│ L4  FULL ISOLATION — per-test tenant / schema / database                │
│     Each test gets its own tenant or schema; nothing is shared at all   │
│     Lifecycle: provisioned per test (or per worker), dropped after      │
│     Cost: high (seconds to minutes). Isolation: total.                  │
│     When: multi-tenant products, destructive tests, or when L3 conflicts│
│           become unmanageable.                                          │
└────────────────────────────────────────────────────────────────────────┘
```

**Decision rule:**

```
Data shared aur immutable?          -> L1
Setup mehnga aur read-mostly?       -> L2
Test isko modify karega?            -> L3   ← DEFAULT
Test destructive hai / tenant-level?-> L4
```

## 30.3 Provisioning strategies compared

| Strategy | Speed | Isolation | Realism | Best for |
|---|---|---|---|---|
| API factory (create via app's own API) | Fast (100-500ms) | Excellent | High — goes through real validation | **Default** |
| Direct DB insert | Fastest (~10ms) | Excellent | **Low — bypasses validation, can create impossible states** | Bulk perf data only |
| DB snapshot restore | Slow (seconds-minutes) | Total | High | Nightly reset, L4 |
| Copy-on-write DB clone (e.g. Neon/Aurora clones) | Fast (seconds) | Total | High | L4 done well |
| Anonymised production copy | Slow | N/A | Highest | Perf testing, bug repro |
| Docker/testcontainers per test | Medium | Total | Medium | Component/integration tests |

**Direct DB insert ka danger jo bolna hai:** *"I avoid seeding through the database because it bypasses the application's own validation, so you can create states the product could never actually reach — and then you're testing behaviour against data that will never exist in production. Worse, it silently breaks when the schema changes, because there's no contract. I use the API for setup precisely because it goes through the same validation the product does."*

## 30.4 Masking / anonymisation

| Field | Strategy | Why |
|---|---|---|
| Name | Deterministic pseudonym (hash → fake name) | Same input → same output, so joins and relationships survive |
| Email | `user_{hash}@example.invalid` | `.invalid` TLD is reserved — mail can never be delivered |
| Phone | Reserved test ranges | Can't dial a real person |
| GSTIN / PAN / Aadhaar | Format-valid, checksum-invalid | Passes format validation, provably fake |
| Bank account / IFSC | Synthetic | Never real |
| Card PAN | **Never copied at all** — replaced with a sandbox token | PCI scope |
| Address | Fake but geographically plausible | Shipping/tax logic still exercised |
| Amounts, dates, quantities | **Usually preserved** | Distribution matters for perf and edge cases |
| Free-text notes | Redact or drop | PII hides in free text — the most commonly missed field |

**Deterministic masking kyun:** agar `Ramesh Kumar` har table mein `Fake_A12` bane, referential integrity bachi rehti hai. Random masking se relationships toot jaate hain aur data useless ho jaata hai.

**Do baatein jo aksar miss hoti hain:**
1. **Free-text fields** — comments, notes, descriptions mein PII chhupi hoti hai. Automated regex scan (email, phone, PAN patterns) chalao masked dataset pe, aur wo scan **CI ka part** ho.
2. **Masked data abhi bhi regulated hai** jab tak masking provably irreversible na ho. Same access controls chahiye.

## 30.5 Refresh strategy

```
┌──────────────┬─────────────────┬──────────────────────────────────────┐
│ Environment  │ Refresh cadence │ Method                                │
├──────────────┼─────────────────┼──────────────────────────────────────┤
│ local        │ on demand       │ docker compose down -v && seed        │
│ dev          │ weekly          │ drop + migrate + seed (fast, small)   │
│ qa           │ nightly seed    │ seed top-up; full reset weekly        │
│ staging      │ monthly         │ anonymised prod snapshot + seed        │
│ production   │ never           │ additive QA-prefixed data only        │
└──────────────┴─────────────────┴──────────────────────────────────────┘

CONTINUOUS (all envs, hourly):
   sweeper job: DELETE entities WHERE name LIKE 'QA-%' AND created_at < now() - 24h
   → this is what actually keeps environments healthy; cleanup failures are
     inevitable and the sweeper is the backstop.
```

**Refresh ka hidden cost:** har refresh ke baad L2 setup dobara banana padta hai, aur kisi ne agar manually kuch configure kiya tha wo gayab. Isliye **sab kuch code se seed ho** — manual env setup ek anti-pattern hai kyunki wo refresh survive nahi karta.

## 30.6 Multi-team conflict prevention

```
Namespace by team + test:
  QA-{team}-{test_id}-{uuid8}      e.g. QA-procurement-po_approval-8f3a2b1c

Benefits:
  - Orphan record ka owner turant pata chalta hai
  - Team-wise cleanup possible
  - Team-wise data volume metrics
  - Ek team ka sweeper doosre ka data delete nahi karega
```

**Aur agar teams sach mein same records pe conflict karti hain:** L4 pe jao (per-team tenant), ya shared record ko read-only bana do aur mutation ko per-test copy pe force karo.

> **Interview answer:** I model test data in four layers by lifetime and isolation. Static seed data — currencies, tax codes, roles — ships with the environment through migrations and is strictly read-only; a test mutating seed data is a bug. Per-suite data is expensive setup like a project or an approved supplier, created once per module. Per-test factory data is where about ninety percent lives: every test creates its own entities through a factory fixture with UUID-based identifiers and guaranteed reverse-order teardown, which is what makes parallel execution safe. And full isolation — a tenant or schema per test — is the escape hatch for destructive tests or when conflicts become unmanageable. I strongly prefer creating data through the application's own API rather than direct database inserts, because the API enforces the same validation the product does; seeding through the database lets you build states the product could never reach, and it breaks silently when the schema changes. For anonymised production data, masking must be deterministic so referential integrity survives, emails go to a reserved invalid domain, card data is never copied at all, and free-text fields get scanned for PII patterns in CI because that's where personal data actually hides. And the thing that keeps environments healthy long-term isn't cleanup — cleanup will sometimes fail — it's a sweeper job that deletes anything with the QA prefix older than a day.

**Cross-question: "Test data ke liye seed script vs factory — kaunsa?"**
> They solve different problems and I use both. A seed script gives every environment the same reference data, which needs to be identical and stable — it's infrastructure. A factory gives one test the specific entity it needs with unique identifiers, which needs to be different every time — it's test code. The failure mode I look out for is factories being replaced by a big seed script, because then tests start depending on shared records and you lose parallel safety and independence at the same time.

---

# 31. Design: QA Process for a New Team (30/60/90)

> **Ye question leadership potential test karta hai.** Sabse common galti: din 1 se tooling ka plan bolna. *"Main Playwright framework banaunga, Allure lagaunga, CI mein integrate karunga."* — ye galat jawab hai, kyunki tumhe abhi pata hi nahi ki problem kya hai.
>
> **Sahi framing: pehle 30 din sirf samajhne aur trust banane ke hain.** Automation din 1 se likhna technically theek lagta hai lekin politically aur practically fail hota hai — kyunki tum aisi cheez automate kar rahe ho jiski value tumhe nahi pata, aur team ko abhi tak tumpe bharosa nahi hai.

## 31.1 Days 1–30 — Learn and earn trust

**Goal: samjho ki dard kahan hai, aur ek visible win do.**

| Week | Kya karo |
|---|---|
| 1 | Product ko **user ki tarah** use karo. Har flow manually. Notes banao — kya confusing hai, kya slow hai, kya toota hua lagta hai |
| 1 | Har role se 1:1 — devs, PM, support, ek sales/CS person. Sawaal: *"What breaks most often? What are you most afraid of shipping?"* |
| 2 | Last 6 mahine ke **production bugs** padho. Classify karo: kaunse area, kaunsa type, kaunse escape hue aur kyun |
| 2 | Existing tests (agar hain) chalao. Pass rate, runtime, flakiness note karo. **Kuch mat badlo abhi** |
| 3 | Release process observe karo — ek poora release dekho end to end. Kahan time jaata hai, kahan darr lagta hai |
| 3 | **Risk map banao** — feature × likelihood × business impact |
| 4 | **Findings present karo** — data ke saath, opinion ke saath nahi. Aur ek chhota concrete win deliver karo |

**Week 4 ka "small win" kya ho sakta hai:**
- Ek high-impact bug dhoondh ke report karo (manual exploratory se) — ye instantly credibility deta hai.
- Ek 30-minute manual smoke checklist banao jo release se pehle chal sake.
- Ek broken flaky test fix karo jo sabko pareshan kar raha tha.

**Ye phase ka anti-pattern:** din 3 pe bolna *"aapka process galat hai, main isko theek karunga."* Ye guaranteed resistance hai. Pehle sun'na, phir bolna.

**Deliverable at day 30:** ek 1-page document —
```
1. What I found (data, not opinion)
   - 43 production bugs in 6 months; 61% in procurement approval logic
   - Release takes 2 days of manual checking; 1 person is the bottleneck
   - No test data strategy; QA env is broken 3 days a week
2. Top 3 risks, ranked by business impact
3. What I propose for the next 60 days, and what I need
4. What I will NOT do yet, and why
```

Wo aakhri line — *"what I will NOT do yet"* — bahut powerful hai. Ye dikhata hai ki tum prioritise kar rahe ho, sab kuch promise nahi kar rahe.

## 31.2 Days 31–60 — Build the foundation

**Goal: highest-risk area ka safety net banao, aur process mein QA ko jaldi le aao.**

| Focus | Action |
|---|---|
| **Test strategy** | 1 page, not 30. Kya automate hoga, kya manual rahega, kaun likhega, kya definition of done hai |
| **Highest-risk area automate karo** | Poora regression nahi — sirf wo top-1 risk jo day-30 report mein tha. 20-40 tests |
| **CI mein daalo** | Chahe sirf smoke ho. Feedback loop banna zaroori hai |
| **Shift-left** | Refinement meetings mein jao. Requirement pe sawaal poochna shuru karo — *"what happens if the approver is deleted mid-approval?"* |
| **Bug process** | Severity/priority definitions, reproduction template, triage cadence |
| **Environment** | Agar QA env 3 din/hafta toota hai — ye #1 priority hai, tests se pehle |

**Framework decisions is phase mein:**
- Team ki language mein likho (Python agar devs Python jaante hain), apni favourite mein nahi.
- Layered structure day 1 se (section 12) — baad mein retrofit karna mehnga hai.
- One canonical way principle day 1 se (section 17).
- **Coverage ka peecha mat karo. Risk ka karo.**

**Metric jo track karna shuru karo (baseline ke liye):**
- Escaped defects per release
- Time from commit to feedback
- Release cycle time
- Flaky rate

## 31.3 Days 61–90 — Scale and hand over

**Goal: QA ko ek person se ek practice mein badlo.**

| Focus | Action |
|---|---|
| **Expand coverage** | Risk order mein next 2 areas |
| **Devs ko contribute karwao** | Unke saath pair karo. Ek dev ka pehla test merge hona **badi jeet** hai |
| **Quality gates** | PR pe smoke mandatory, pre-deploy pe regression. Gradually enforce, day 1 se nahi |
| **Reporting** | Verdict-first report, weekly quality digest to the team |
| **Flaky management** | Detection + quarantine with SLA (section 22) |
| **Document + train** | Contributing guide, template test, 1-hour workshop |
| **Retro** | 90-day review: kya kaam kiya, kya nahi, agla quarter kya |

**Sabse important cheez is phase mein:** *"quality is not the QA person's job"* wali culture shift shuru karo. Jab pehla developer khud test likhta hai bina bole — wo tumhara sabse bada success metric hai.

## 31.4 Timeline diagram

```
DAY 1 ──────────── 30 ──────────── 60 ──────────── 90 ─────────>
  │                 │                │                │
  │  LEARN & TRUST  │   FOUNDATION   │  SCALE & HAND  │
  │                 │                │      OVER      │
  ├─ use product    ├─ 1-page strat  ├─ expand coverage
  ├─ 1:1s           ├─ automate #1   ├─ devs contribute
  ├─ read bug hist  │  risk area     ├─ quality gates
  ├─ observe release├─ CI smoke      ├─ flaky process
  ├─ risk map       ├─ shift-left    ├─ reporting cadence
  └─ SMALL WIN      ├─ bug process   ├─ docs + training
                    └─ fix the env   └─ 90-day retro
                       if it's broken

  Trust curve:     ▁▂▄▆  ──>  ▆▇█  ──>  ████
  Automation:      ▁     ──>  ▃▄   ──>  ▆▇
```

**Note the ordering:** trust curve automation curve se **pehle** upar jaati hai. Ye deliberate hai.

> **Interview answer:** My first thirty days would have almost no automation in them, and that's deliberate. I'd use the product as a user across every flow, do one-to-ones with developers, the PM, support and someone customer-facing asking what breaks most and what they're most afraid of shipping, read six months of production bugs and classify them by area and escape reason, and watch a full release end to end. That gives me a risk map built on data rather than opinion. I'd close month one by presenting findings with numbers, proposing a plan, and — importantly — saying what I will not do yet and why. I'd also deliver one small concrete win, usually a real bug found through exploratory testing, because credibility comes from finding something that matters, not from a framework nobody asked for. Month two is foundation: a one-page test strategy, automation for the single highest-risk area only, and getting that into CI so the feedback loop exists. If the QA environment is broken half the week, fixing that comes before any test — an unreliable environment poisons every signal you'll produce afterwards. Month three is scale and handover: expand by risk, pair with developers so they contribute tests, introduce quality gates gradually, set up flaky detection and a reporting cadence, and run a ninety-day retro. The reason trust comes before tooling is practical — a framework built before you understand the risk automates the wrong things, and a process imposed before you've earned credibility gets quietly ignored. My real success metric at ninety days is a developer writing a test without being asked.

**Cross-question: "Agar management din 1 se automation maange to?"**
> I'd agree to a visible deliverable on their timeline while protecting the discovery work. Concretely: I'd commit to a working CI smoke suite by day thirty rather than a full framework, and I'd explain that the specific tests in it will be chosen from what I learn in weeks one and two, so the same effort covers what actually breaks. That's a negotiation about scope, not about whether to deliver. What I'd push back on is committing to a coverage number early, because a percentage target drives you to automate what's easy rather than what's risky.

**Cross-question: "Team automation ko resist kar rahi hai — kya karoge?"**
> First I'd find out why, because the reasons are usually legitimate: a previous suite was flaky and wasted their time, or tests block merges without giving useful failure messages, or they were told to write tests without being given time. Each of those has a different fix. Generally the thing that changes minds is making the suite obviously useful to them — fast, reliable, and with failure output that points at the problem rather than at a stack trace. When a test catches something real before review and the message is clear, adoption follows. Mandates without that produce compliance, not ownership.

---

# 32. Microservices Testing

## 32.1 Architecture

```
   ┌──────────┐
   │ Frontend │  React SPA
   └────┬─────┘
        │ HTTPS  (X-Correlation-Id generated here)
        v
   ┌─────────────────┐
   │  API GATEWAY    │  authn/authz, rate limit, routing, correlation propagation
   └──┬───────┬──────┘
      │       │
      v       v
┌──────────┐ ┌──────────────┐  sync HTTP/gRPC  ┌──────────────┐
│ Service A│ │  Service B    │ ───────────────> │  Service C   │
│  (PO)    │ │  (Invoice)    │                  │  (Supplier)  │
└────┬─────┘ └──────┬───────┘                  └──────┬───────┘
     │ own DB       │ own DB                           │ own DB
     v              v                                  v
  ┌──────┐      ┌──────┐                            ┌──────┐
  │ po_db│      │inv_db│                            │sup_db│
  └──────┘      └──────┘                            └──────┘
     │              │                                  │
     │  publish     │  publish                         │
     └──────┬───────┴──────────────┬───────────────────┘
            v                      v
      ┌───────────────────────────────────────┐
      │        KAFKA / EVENT BUS               │
      │  topics: po.events, invoice.events,    │
      │          supplier.events               │
      └───────┬───────────────┬────────────────┘
              │ consume       │ consume
              v               v
      ┌──────────────┐  ┌──────────────┐
      │ Notification │  │  Analytics   │
      │   Service    │  │   Service    │
      └──────────────┘  └──────────────┘
```

**Monolith se kya badla — testing ke liye:**

| | Monolith | Microservices |
|---|---|---|
| Transaction | ACID, ek DB | **Distributed — saga + compensation** |
| Consistency | Immediate | **Eventual** — test ko wait karna padega |
| Failure | Sab ya kuch nahi | **Partial failure** — half-done states real hain |
| Debugging | Ek stack trace | **Correlation ID chahiye**, warna andhere mein |
| Test env | Ek app chalao | 8 services + Kafka + 3 DBs |
| Contract | Compiler check karta hai | **Runtime pe pata chalta hai — isliye contract tests** |

## 32.2 Testing strategy across services

```
        ▲  fewer, slower, more confidence
        │
   ┌────┴─────────────────────────────────┐
   │  E2E (5%)                             │  real services, real flow
   │  Critical journeys only               │  slow, expensive, flaky-prone
   ├───────────────────────────────────────┤
   │  Contract tests (15%)                 │  ← THE microservices-specific layer
   │  Consumer-driven, per service pair    │  fast, catches integration breaks
   ├───────────────────────────────────────┤
   │  Component / service tests (30%)      │  one service, real DB, mocked deps
   │  Testcontainers, in-memory bus        │
   ├───────────────────────────────────────┤
   │  Unit tests (50%)                     │  fast, isolated
   └───────────────────────────────────────┘
```

**Key insight jo bolna hai:** *"In microservices, the expensive integration questions are answered by contract tests, not by end-to-end tests. E2E should verify a handful of critical journeys, not the compatibility of every service pair — that combinatorial explosion is what makes E2E suites unmaintainable."*

## 32.3 Service isolation — testing one service alone

```python
# Component test: real service + real DB, dependencies stubbed
import pytest
from testcontainers.postgres import PostgresContainer

@pytest.fixture(scope="session")
def po_db():
    with PostgresContainer("postgres:16") as pg:
        run_migrations(pg.get_connection_url())
        yield pg.get_connection_url()

@pytest.fixture
def supplier_service_stub(wiremock):
    """Supplier service ka stub — real service nahi chahiye."""
    wiremock.stub(
        method="GET", url_pattern=r"/suppliers/[\w-]+",
        response={"status": 200, "json": {"id": "SUP-1", "status": "APPROVED",
                                          "credit_limit_paise": 10_000_000}},
    )
    return wiremock

def test_po_creation_rejects_unapproved_supplier(po_service, supplier_service_stub):
    supplier_service_stub.stub(
        method="GET", url_pattern=r"/suppliers/SUP-BAD",
        response={"status": 200, "json": {"id": "SUP-BAD", "status": "PENDING"}},
    )
    r = po_service.post("/po", json={"supplier_id": "SUP-BAD", "amount_paise": 100_000})
    assert r.status_code == 422
    assert r.json()["code"] == "SUPPLIER_NOT_APPROVED"

def test_po_creation_when_supplier_service_is_down(po_service, supplier_service_stub):
    """Resilience: dependency down hone pe kya hota hai?"""
    supplier_service_stub.stub(method="GET", url_pattern=r"/suppliers/.*",
                               response={"status": 503})
    r = po_service.post("/po", json={"supplier_id": "SUP-1", "amount_paise": 100_000})
    # Design decision: fail closed (safe) ya degrade gracefully?
    assert r.status_code == 503
    assert r.json()["code"] == "SUPPLIER_SERVICE_UNAVAILABLE"
    assert r.headers.get("Retry-After") is not None
```

**Dependency-down testing** aksar chhoot jaati hai aur production mein wahi sabse zyada dukh deti hai. Har external dependency ke liye kam se kam 3 tests: **timeout, 5xx, malformed response.**

## 32.4 Contract testing (consumer-driven, Pact)

**Problem:** Provider ne response mein field ka naam badla. Provider ke apne tests pass. Consumer production mein toot gaya. E2E hi pakadta hai — aur E2E slow aur flaky hai.

**Solution:** consumer likhta hai ki usko provider se kya chahiye; provider us expectation ke against verify karta hai.

```
┌──────────────────────┐                    ┌──────────────────────┐
│  CONSUMER            │                    │  PROVIDER            │
│  (Invoice Service)   │                    │  (PO Service)        │
│                      │                    │                      │
│  1. Test likho with  │                    │                      │
│     mock provider    │                    │                      │
│  2. PACT FILE        │ ──── publish ───>  │  3. Pact broker se   │
│     generate hota hai│      to broker     │     pact fetch karo  │
│     (expectations)   │                    │  4. Real provider pe │
│                      │                    │     replay karo      │
│                      │ <─── verification  │  5. Pass/fail        │
│                      │      result        │                      │
└──────────────────────┘                    └──────────────────────┘

Provider ka CI FAIL ho jaata hai agar wo consumer ka contract tode — deploy se PEHLE.
```

**Consumer side:**

```python
# invoice_service/tests/test_po_client_pact.py
import pytest
from pact import Consumer, Provider, Like, Term

pact = Consumer("invoice-service").has_pact_with(Provider("po-service"),
                                                 pact_dir="./pacts")

def test_get_approved_po_contract(po_client):
    expected = {
        "id": Like("PO-1001"),
        "status": Term(r"APPROVED|PENDING_APPROVAL|DRAFT", "APPROVED"),
        "amount_paise": Like(125000),
        "supplier_id": Like("SUP-42"),
        "line_items": [{"sku": Like("CEM-53"), "qty": Like(100)}],
    }
    (pact
     .given("a purchase order PO-1001 exists and is approved")   # provider state
     .upon_receiving("a request for an approved purchase order")
     .with_request("get", "/api/po/PO-1001",
                   headers={"Accept": "application/json"})
     .will_respond_with(200, body=expected))

    with pact:
        po = po_client.get_purchase_order("PO-1001")
        assert po.status == "APPROVED"
        assert po.amount_paise == 125000
```

**Provider side:**

```python
# po_service/tests/test_pact_verification.py
def test_verify_invoice_service_pact(provider_app, pact_broker):
    verifier = Verifier(provider="po-service",
                        provider_base_url="http://localhost:8000")
    output, _ = verifier.verify_with_broker(
        broker_url=pact_broker.url,
        publish_verification_results=True,
        provider_version=os.environ["GIT_SHA"],
        # provider states: "a purchase order PO-1001 exists and is approved"
        # ko real data mein setup karna padega
        provider_states_setup_url="http://localhost:8000/_pact/provider_states",
    )
    assert output == 0
```

**Key concepts jo poochte hain:**

| Term | Matlab |
|---|---|
| **Consumer-driven** | Contract consumer likhta hai (usko jo chahiye), provider apna full API nahi |
| **Provider state** | "Given a PO exists and is approved" — provider ko ye state banani padegi verification se pehle |
| **Pact broker** | Central store; contracts + verification results + versions |
| **can-i-deploy** | Broker se poocho — "kya ye version deploy karna safe hai?" CI gate |
| **Matchers** (`Like`, `Term`) | **Type** match karo, exact value nahi — warna contract har data change pe toot jaayega |

**Contract testing kya NAHI karta (ye bolna important hai):** ye **shape aur compatibility** verify karta hai, **business correctness** nahi. Ye nahi batata ki approval logic sahi hai. Isliye ye E2E ko replace nahi karta — ye **integration ka wo hissa** replace karta hai jiske liye E2E use ho raha tha.

## 32.5 Test doubles taxonomy

> **Ye definitions interview mein exactly poochi jaati hain.** Gerard Meszaros ki taxonomy hai. Yaad rakhne ka tareeka: **kitna behaviour hai** aur **kya verify karte ho**.

```
     kam behaviour ────────────────────────────────> zyada behaviour
     
     DUMMY ──> STUB ──> SPY ──> MOCK ──> FAKE
     
     never    canned   records  expects  working
     used     answers  calls    calls    implementation
```

| Double | Kya karta hai | Verify kya karte ho | Example |
|---|---|---|---|
| **Dummy** | Kuch nahi — sirf parameter bharne ke liye | Kuch nahi | `create_po(supplier, logger=None)` mein `None` |
| **Stub** | Fixed/canned response deta hai | **State** — output kya aaya | `stub.get_supplier() -> {"status": "APPROVED"}` |
| **Spy** | Real ya stub + **calls record karta hai** | Baad mein calls check karte ho | `assert spy.calls == [("POST", "/po")]` |
| **Mock** | **Pehle se expectations set** + verify | **Behaviour** — sahi call hui ya nahi | `mock.expect_call("send_email").once()` |
| **Fake** | **Working implementation**, simplified | State, normally | In-memory DB, in-memory queue |

**Code mein farq:**

```python
# ---------- DUMMY: bas jagah bharne ke liye ----------
class DummyLogger:
    def log(self, *a, **kw): pass          # kabhi call hi nahi hoga, ya matter nahi karta

create_po(supplier="ACME", logger=DummyLogger())


# ---------- STUB: canned answers ----------
class StubSupplierClient:
    def get(self, supplier_id):
        return {"id": supplier_id, "status": "APPROVED", "credit_limit_paise": 10_000_000}

def test_po_allowed_within_credit_limit():
    result = POService(StubSupplierClient()).create(supplier_id="SUP-1", amount_paise=500_000)
    assert result.status == "DRAFT"        # STATE verify — output kya hai


# ---------- SPY: calls record karta hai ----------
class SpyNotifier:
    def __init__(self): self.calls = []
    def send(self, to, subject, body):
        self.calls.append({"to": to, "subject": subject})

def test_approval_notifies_requester():
    spy = SpyNotifier()
    POService(notifier=spy).approve("PO-1001")
    assert len(spy.calls) == 1
    assert spy.calls[0]["to"] == "requester@merlin.co"    # BEHAVIOUR verify (after the fact)


# ---------- MOCK: expectations pehle, verify baad mein ----------
from unittest.mock import Mock, call

def test_approval_notifies_exactly_once():
    notifier = Mock()
    POService(notifier=notifier).approve("PO-1001")
    notifier.send.assert_called_once_with(
        to="requester@merlin.co", subject="PO-1001 approved", body=ANY
    )                                       # BEHAVIOUR verify with pre-set expectation
    notifier.send_sms.assert_not_called()


# ---------- FAKE: working, simplified implementation ----------
class FakePORepository:
    """Real repo jaisa behaviour, lekin in-memory."""
    def __init__(self): self._store, self._seq = {}, 1000
    def save(self, po):
        self._seq += 1
        po_id = f"PO-{self._seq}"
        self._store[po_id] = {**po, "id": po_id}
        return self._store[po_id]
    def get(self, po_id):
        if po_id not in self._store: raise NotFound(po_id)
        return self._store[po_id]
    def find_by_supplier(self, sid):
        return [p for p in self._store.values() if p["supplier_id"] == sid]

def test_po_workflow_end_to_end_in_memory():
    repo = FakePORepository()
    svc = POService(repo=repo)
    po = svc.create(supplier_id="SUP-1", amount_paise=125_000)
    svc.submit(po["id"]); svc.approve(po["id"])
    assert repo.get(po["id"])["status"] == "APPROVED"     # asli logic chali, DB ke bina
```

**Stub vs Mock — the one-line distinction jo interviewer sunna chahta hai:**

> *"A stub is about the input to your code — it feeds canned data so you can test state. A mock is about the output of your code — it verifies that the right interactions happened. If your assertion is on a return value, you used a stub. If your assertion is on a call, you used a mock or spy."*

**Spy vs Mock:** spy **record karke baad mein poochta hai** (lenient, "loose"); mock **pehle expectation set karta hai aur verify karta hai** (strict). Practically Python ke `unittest.mock.Mock` dono roles nibhata hai — isliye conceptual difference bolna zaroori hai.

**Kab kaunsa — practical guidance:**
- Default **stub/fake** use karo. Ye implementation se kam coupled hain.
- **Mock** sirf tab jab side effect hi behaviour hai (email bheja, event publish hua, payment charge hua).
- **Over-mocking** ek known anti-pattern hai: agar test poora call sequence assert kar raha hai, wo refactor pe toot jaayega jabki behaviour sahi hai. *"Mock what you can't control; fake what you own."*

## 32.6 Service virtualization

**Service virtualization = ek poora fake service jo network pe chalta hai** (stub jo process ke andar hai, uska scale-up).

| Tool | Kya |
|---|---|
| **WireMock** | HTTP stubbing, record & replay, fault injection, latency simulation |
| **Mountebank** | Multi-protocol (HTTP, TCP, SMTP) |
| **Hoverfly** | Capture/simulate, proxy mode |
| **MockServer** | HTTP/HTTPS, expectations, verification |
| **Prism** | OpenAPI spec se mock server auto-generate |
| **Playwright route interception** | Browser level — frontend tests ke liye best |

```python
# Frontend test mein backend virtualize karna — Playwright
def test_dashboard_handles_slow_kpi_api(page):
    def slow_response(route):
        page.wait_for_timeout(6000)         # 6s latency inject
        route.fulfill(status=200, json={"kpis": []})
    page.route("**/api/dashboard/kpis", slow_response)
    page.goto("/dashboard")
    expect(page.get_by_test_id("kpi-skeleton")).to_be_visible()   # loading state dikhta hai?
    expect(page.get_by_test_id("kpi-timeout-message")).to_be_visible(timeout=10_000)

def test_dashboard_handles_api_500(page):
    page.route("**/api/dashboard/kpis", lambda r: r.fulfill(status=500))
    page.goto("/dashboard")
    expect(page.get_by_role("alert")).to_contain_text("Unable to load")
    expect(page.get_by_role("button", name="Retry")).to_be_visible()

def test_dashboard_handles_malformed_response(page):
    page.route("**/api/dashboard/kpis", lambda r: r.fulfill(status=200, body="not json"))
    page.goto("/dashboard")
    expect(page.get_by_role("alert")).to_be_visible()      # crash nahi hona chahiye
```

**Ye kyun powerful hai:** error states, timeouts, aur edge-case responses ko **deterministically** test kar sakte ho. Real backend se ye trigger karna ya to impossible hai ya bahut mehnga. **Aur ye tests bilkul flaky nahi hote**, kyunki network involved hi nahi hai.

**Trade-off jo bolna hai:** *"Virtualized responses can drift from reality — the stub says one thing and the real service returns another. That's exactly the gap contract testing closes, so I pair the two: virtualization for edge cases and error states, contract tests to guarantee the happy-path shape is still accurate."*

---

## 32.7 Event-driven testing — Kafka basics

**Analogy:** Kafka ek **akhbaar ka distribution system** hai. Publisher (service) akhbaar chhapta hai ek **topic** (edition) mein. Wo topic **partitions** (bundles) mein bantt jaata hai. **Consumer group** ek delivery team hai — team ke har member ko alag bundle milta hai, aur poori team milkar sab bundles cover karti hai. Doosri team (doosra consumer group) **wahi** akhbaar independently padh sakti hai.

```
   PRODUCER (PO Service)
        │ publish key=PO-1001
        v
  ┌──────────────────────────────────────────────────┐
  │ TOPIC: po.events                                  │
  │  ┌────────────┐ ┌────────────┐ ┌────────────┐    │
  │  │Partition 0 │ │Partition 1 │ │Partition 2 │    │
  │  │ [m1][m2][m3]│ │ [m4][m5]   │ │ [m6][m7]   │    │
  │  └────────────┘ └────────────┘ └────────────┘    │
  │   ordering guaranteed WITHIN a partition only     │
  └──────┬────────────────┬───────────────┬──────────┘
         │                │               │
   ┌─────v──────────────────────────┐  ┌──v─────────────────────────┐
   │ CONSUMER GROUP: notifications  │  │ CONSUMER GROUP: analytics  │
   │  consumer-1 -> P0              │  │  consumer-1 -> P0, P1, P2  │
   │  consumer-2 -> P1, P2          │  │  (own independent offsets) │
   │  (own offsets per partition)   │  │                            │
   └────────────────────────────────┘  └────────────────────────────┘
   
   Dono groups SAME messages padhte hain, independently.
   Ek group ke andar, har partition EXACTLY ek consumer ko jaata hai.
```

| Concept | Matlab | Testing implication |
|---|---|---|
| **Topic** | Named stream of events | Test ko topic name aur schema pata hona chahiye |
| **Partition** | Topic ka shard; parallelism ki unit | **Ordering sirf partition ke andar** guaranteed hai |
| **Key** | Partition decide karti hai (`hash(key) % partitions`) | Same key = same partition = ordering. Ye order-sensitive flows ke liye critical |
| **Offset** | Partition mein consumer ki position | Reset karke replay test kar sakte ho |
| **Consumer group** | Consumers ka set jo kaam baantte hain | Group ke andar ek partition ek hi consumer ko |
| **Rebalance** | Consumer aane/jaane pe partitions redistribute | Rebalance ke dauraan duplicate processing ho sakti hai |
| **Retention** | Message kitna time rakha jaata | Replay window |

**Delivery semantics — ye poocha jaata hai:**

| Semantic | Matlab | Reality |
|---|---|---|
| At-most-once | 0 ya 1 baar | Message loss possible |
| **At-least-once** | 1 ya zyada baar | **Kafka ka practical default** — duplicates possible |
| Exactly-once | Theek 1 baar | Kafka transactions se possible, lekin costly aur end-to-end guarantee mushkil |

**Isliye: consumers ko IDEMPOTENT hona hi padega.** Ye ek logical zaroorat hai, optional optimisation nahi.

```python
def test_duplicate_event_does_not_create_duplicate_invoice(kafka, invoice_db):
    """At-least-once delivery ka matlab hai duplicate AAYENGE. Consumer idempotent ho."""
    event = {"event_id": "evt-abc-123", "type": "po.approved",
             "po_id": "PO-1001", "amount_paise": 125_000}

    kafka.produce("po.events", key="PO-1001", value=event)
    kafka.produce("po.events", key="PO-1001", value=event)      # exact duplicate
    kafka.produce("po.events", key="PO-1001", value=event)      # aur ek

    wait_until(lambda: invoice_db.count(po_id="PO-1001") >= 1, timeout=30)
    time.sleep(3)                                                # settle hone do
    assert invoice_db.count(po_id="PO-1001") == 1, "Consumer is not idempotent"


def test_out_of_order_events_are_handled(kafka, po_db):
    """Alag partitions se events out of order aa sakte hain."""
    kafka.produce("po.events", key="PO-2001",
                  value={"type": "po.approved", "po_id": "PO-2001", "version": 2})
    kafka.produce("po.events", key="PO-2001",
                  value={"type": "po.submitted", "po_id": "PO-2001", "version": 1})

    wait_until(lambda: po_db.get("PO-2001")["status"] == "APPROVED", timeout=30)
    # Stale event ne state ko peeche nahi kheencha
    assert po_db.get("PO-2001")["version"] == 2


def test_poison_message_goes_to_dlq_and_does_not_block(kafka, dlq):
    """Ek bad message poori partition ko block nahi karna chahiye."""
    kafka.produce("po.events", key="PO-3001", value={"type": "po.approved"})  # missing fields
    kafka.produce("po.events", key="PO-3002",
                  value={"type": "po.approved", "po_id": "PO-3002", "amount_paise": 1000})

    wait_until(lambda: dlq.count() == 1, timeout=30)
    wait_until(lambda: invoice_db.exists(po_id="PO-3002"), timeout=30)  # aage wale chale
```

**Event schema evolution** — Schema Registry ke saath compatibility test:

```python
def test_new_optional_field_is_backward_compatible(schema_registry):
    """Producer naya field add kare to purane consumers na toote."""
    new_schema = load_schema("po_approved_v2.avsc")
    assert schema_registry.check_compatibility(
        subject="po.events-value", schema=new_schema, level="BACKWARD"
    ) is True
```

## 32.8 Async workflow testing — polling vs callback vs event

```
❌ NEVER:  time.sleep(10)   — slow AND flaky at the same time.
           Fast machine pe waste, slow machine pe fail. Worst of both.
```

| Strategy | Kaise | Pros | Cons | Kab |
|---|---|---|---|---|
| **Polling with timeout** | Har 500ms check karo, max 30s | Simple, universal | Latency (poll interval), load | **Default** |
| **Callback / webhook** | Test ek HTTP endpoint expose kare | No polling, exact timing | Test ko server chalana padega, networking | Webhook flows |
| **Event subscription** | Test khud Kafka consumer bane | Sabse accurate, event contract bhi verify hota | Kafka client setup | Event-driven systems |
| **DB polling** | DB directly check karo | Fast, precise | Internals pe coupling | Last resort |

```python
def wait_until(condition, *, timeout_s=30, interval_s=0.5, message="condition not met"):
    """Canonical polling helper — pehla poll TURANT, phir interval."""
    import time
    deadline = time.monotonic() + timeout_s
    last_exc = None
    while time.monotonic() < deadline:
        try:
            result = condition()
            if result:
                return result
        except Exception as e:
            last_exc = e            # dependency abhi ready nahi — retry karo
        time.sleep(interval_s)
    raise TimeoutError(f"{message} (waited {timeout_s}s). Last error: {last_exc}")


# ---- 1. POLLING ----
def test_invoice_generated_after_po_approval(api, po):
    api.post(f"/po/{po['id']}/approve")
    invoice = wait_until(
        lambda: api.get(f"/invoices?po_id={po['id']}").json().get("items") or None,
        timeout_s=45,
        message=f"No invoice generated for {po['id']}",
    )
    assert invoice[0]["amount_paise"] == po["amount_paise"]


# ---- 2. EVENT SUBSCRIPTION ----
def test_approval_publishes_correct_event(api, kafka_consumer, po):
    kafka_consumer.subscribe("po.events")
    api.post(f"/po/{po['id']}/approve")
    event = kafka_consumer.wait_for(
        lambda e: e["type"] == "po.approved" and e["po_id"] == po["id"], timeout_s=30
    )
    # Event ka CONTRACT bhi verify karo — downstream isi pe depend karta hai
    assert set(event) >= {"event_id", "type", "po_id", "amount_paise",
                          "approved_by", "occurred_at", "correlation_id"}
    assert event["amount_paise"] == po["amount_paise"]


# ---- 3. WEBHOOK / CALLBACK ----
@pytest.fixture
def webhook_receiver():
    from http.server import HTTPServer, BaseHTTPRequestHandler
    import threading, json, queue
    received = queue.Queue()

    class Handler(BaseHTTPRequestHandler):
        def do_POST(self):
            body = self.rfile.read(int(self.headers["Content-Length"]))
            received.put(json.loads(body))
            self.send_response(200); self.end_headers()
        def log_message(self, *a): pass

    server = HTTPServer(("0.0.0.0", 0), Handler)
    threading.Thread(target=server.serve_forever, daemon=True).start()
    yield type("R", (), {"url": f"http://{get_host_ip()}:{server.server_port}",
                         "queue": received})()
    server.shutdown()

def test_webhook_fired_on_approval(api, webhook_receiver, po):
    api.post("/webhooks", json={"url": webhook_receiver.url, "events": ["po.approved"]})
    api.post(f"/po/{po['id']}/approve")
    payload = webhook_receiver.queue.get(timeout=30)
    assert payload["po_id"] == po["id"]
```

**Timeout selection ka rule:** timeout = **p99 latency × 3**. Bahut chhota = flaky. Bahut bada = failure detect hone mein der (aur suite slow). Aur timeout ko **config se** aao, hardcode nahi — kyunki staging aur production ki latency alag hoti hai.

## 32.9 Distributed transactions & the Saga pattern

**Problem:** Order banane ke liye 4 services chahiye. Beech mein ek fail ho gaya. Rollback kaise? **Distributed 2-phase commit practically use nahi hota** — slow hai, availability maar deta hai.

**Solution: Saga — local transactions ki chain + compensating actions.**

```
   HAPPY PATH
   ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐
   │ 1. Reserve │->│ 2. Charge  │->│ 3. Create  │->│ 4. Notify  │
   │   Inventory│  │   Payment  │  │   Order    │  │  Supplier  │
   └────────────┘  └────────────┘  └────────────┘  └────────────┘

   FAILURE AT STEP 3 -> COMPENSATE IN REVERSE
   ┌────────────┐  ┌────────────┐  ┌────────────┐
   │ 1. Reserve │->│ 2. Charge  │->│ 3. Create  │ ❌ FAILS
   │   Inventory│  │   Payment  │  │   Order    │
   └─────▲──────┘  └─────▲──────┘  └────────────┘
         │               │
   ┌─────┴──────┐  ┌─────┴──────┐
   │ C1. Release│<-│ C2. Refund │      compensating transactions
   │  Inventory │  │   Payment  │      (reverse order)
   └────────────┘  └────────────┘
```

**Do styles:**

| | Choreography | Orchestration |
|---|---|---|
| Kaise | Har service event sunta hai aur react karta hai | Ek orchestrator steps call karta hai |
| Pros | Loose coupling, no single point | Flow ek jagah dikhta hai, debug aasan |
| Cons | **Flow kahin likha nahi hai** — samajhna mushkil | Orchestrator ek coupling point |
| Testing | Har service alag + E2E se poora flow | Orchestrator ki state machine seedha test ho sakti |

```python
def test_saga_compensates_when_order_creation_fails(services, fault):
    """Sabse important saga test — partial failure aur uska cleanup."""
    with fault.inject(service="order", endpoint="POST /orders", error=500):
        r = services.checkout.post("/checkout", json={"cart_id": cart["id"],
                                                      "payment_token": token})
    assert r.status_code == 502

    # Compensation complete hui?
    wait_until(lambda: services.payment.get(f"/charges?cart={cart['id']}")
                       .json()["items"][0]["status"] == "refunded", timeout_s=60)
    wait_until(lambda: services.inventory.get(f"/reservations?cart={cart['id']}")
                       .json()["items"] == [], timeout_s=60)

    # Saga state machine ne kya record kiya
    saga = services.saga.get(f"/sagas?cart_id={cart['id']}").json()["items"][0]
    assert saga["status"] == "COMPENSATED"
    assert [s["name"] for s in saga["steps"] if s["status"] == "COMPENSATED"] == \
           ["charge_payment", "reserve_inventory"]           # REVERSE order


def test_compensation_itself_failing_raises_an_alert(services, fault):
    """Compensation bhi fail ho sakta hai. Tab kya? Ye aksar untested rehta hai."""
    with fault.inject(service="order", endpoint="POST /orders", error=500), \
         fault.inject(service="payment", endpoint="POST /refunds", error=500):
        services.checkout.post("/checkout", json={"cart_id": cart["id"]})

    saga = wait_until(lambda: services.saga.get(f"/sagas?cart_id={cart['id']}").json()["items"][0])
    assert saga["status"] == "COMPENSATION_FAILED"
    assert saga["requires_manual_intervention"] is True
    assert services.alerts.was_fired("saga.compensation_failed")     # human ko pata chala?
```

**Compensating action ke properties jo test karne hain:**
1. **Idempotent** — do baar chalne pe do refund nahi.
2. **Eventually succeeds** — retry with backoff, warna manual queue.
3. **Semantically correct** — refund ≠ "transaction undo". Ledger mein charge aur refund **dono** dikhne chahiye.
4. **Order matters** — reverse order mein.
5. **Failure ka apna plan** — compensation fail hone pe alert + manual queue.

**Point 3 ek achha nuance hai:** *"Compensation isn't rollback. A rollback erases history; a compensating transaction adds a new fact that offsets the old one. The audit trail must show both the charge and the refund — that difference matters for accounting and for disputes."*

## 32.10 Correlation IDs & eventual consistency

**Correlation ID ka propagation:**

```
Frontend generates:  X-Correlation-Id: 7f3a2b1c-...
      │
      v
Gateway ──> Service A ──> Service B ──> DB
   │           │             │
   │           │ publishes event with correlation_id in header/payload
   │           v
   │      Kafka ──> Consumer ──> Service C
   │
   └──> every log line in every service carries correlation_id
   
   Result: ek query se poora request ka journey dikhta hai — 8 services ke paar.
```

```python
def test_correlation_id_propagates_across_services(api, logs, kafka_consumer):
    cid = f"qa-{uuid.uuid4().hex}"
    api.post("/po", json={...}, headers={"X-Correlation-Id": cid})

    # 1. Saare services ke logs mein wahi id
    entries = wait_until(lambda: logs.search(correlation_id=cid, min_count=3), timeout_s=30)
    assert {e["service"] for e in entries} >= {"gateway", "po-service", "supplier-service"}

    # 2. Published event mein bhi propagate hua
    event = kafka_consumer.wait_for(lambda e: e.get("correlation_id") == cid)
    assert event["type"] == "po.created"
```

**Eventual consistency testing — 3 cheezein test karo:**

```python
# 1. Convergence — end state eventually sahi hai
def test_read_model_eventually_reflects_write(api, po):
    api.post(f"/po/{po['id']}/approve")
    wait_until(lambda: api.get(f"/search/po?q={po['id']}").json()["items"][0]["status"]
                       == "APPROVED", timeout_s=30)

# 2. Convergence WINDOW — kitni der lagti hai, aur kya wo acceptable hai
def test_read_model_converges_within_sla(api, po):
    t0 = time.monotonic()
    api.post(f"/po/{po['id']}/approve")
    wait_until(lambda: search_shows_approved(po['id']), timeout_s=30)
    lag = time.monotonic() - t0
    assert lag < 5.0, f"Read model lag {lag:.1f}s exceeds 5s SLA"

# 3. INTERMEDIATE state acceptable hai — user ko kya dikhta hai beech mein?
def test_ui_shows_pending_state_during_propagation(page, po_api, po):
    po_api.post(f"/po/{po['id']}/approve")
    page.goto(f"/purchase-orders/{po['id']}")
    # Stale data dikhna theek hai, lekin honestly labelled hona chahiye
    expect(page.get_by_test_id("sync-indicator")).to_be_visible()
    expect(page.get_by_test_id("po-status")).to_have_text("APPROVED", timeout=30_000)
```

**Ye teesra test sabse valuable hai aur sabse kam likha jaata hai.** Eventual consistency ka asli QA sawaal ye nahi hai ki "kya ye eventually consistent ho jaata hai" — wo to design se hai. Sawaal ye hai: **"beech ke us window mein user ko kya dikhta hai, aur kya wo galat decision le sakta hai?"** Agar approve karne ke baad UI abhi bhi "Approve" button dikha raha hai, user dobara click karega — aur agar backend idempotent nahi hai, duplicate.

> **Interview answer:** Microservices change testing in four concrete ways. First, integration compatibility moves from the compiler to runtime, so I use consumer-driven contract tests with Pact — the consumer declares what it needs, that pact is published to a broker, and the provider's CI verifies against it before deploying. That catches breaking changes before release without needing a full end-to-end environment, and it's why E2E should stay at a handful of critical journeys rather than trying to cover every service pair. Second, partial failure becomes normal, so I test dependency failure explicitly — timeout, 5xx, and malformed response for every external call — and I test sagas by injecting a failure mid-flow and asserting the compensating actions ran in reverse order, plus the case where compensation itself fails and must alert a human. Third, everything is asynchronous, so I never sleep; I poll with a timeout derived from p99 latency, or better, subscribe to the event directly so I verify the event contract at the same time. And because Kafka gives at-least-once delivery, idempotent consumers aren't optional — I explicitly publish the same event three times and assert exactly one invoice is created. Fourth, debugging requires correlation IDs propagated through every service and into event payloads, and I have a test that asserts that propagation, because it's the thing that makes every future investigation possible.

**Cross-question: "Test doubles — stub aur mock mein exact difference?"**
> A stub is about input: it feeds canned data into the code under test so I can assert on resulting state. A mock is about output: it carries pre-set expectations and verifies that the code made the right calls. The quick test is what your assertion targets — if you assert on a return value you used a stub, if you assert on a call you used a mock or a spy. A spy sits between: it records calls and you inspect them afterwards, rather than declaring expectations upfront. A fake is a working simplified implementation, like an in-memory repository, and a dummy is a placeholder that's never actually used. My default is stubs and fakes because they're less coupled to implementation; I reach for mocks only when the interaction itself is the behaviour under test — an email sent, an event published, a payment charged.

**Cross-question: "Eventual consistency ko test karna itna mushkil kyun?"**
> Because there's no single moment when the system is "correct", so a naive assertion right after the write fails, and the usual fix — a sleep — is both slow and flaky. The right approach is polling with a timeout that reflects the actual SLA, and then treating the convergence window as a first-class thing to test: assert it converges, assert it converges within the SLA so a regression in lag is caught, and assert what the user sees during the window. That last one matters most, because if the UI shows a stale state without indicating it's stale, users take duplicate actions — and then you're relying on the backend being idempotent to save you.

---

# 33. Observability for QA

> **Kyun QA ko ye jaanna chahiye:** ek senior SDET sirf "test likhne wala" nahi hota — wo **quality signal ka owner** hota hai. Aur production ka signal test suite se zyada sach bolta hai. Jo QA production observability padhta hai, uska test backlog **real user pain** se aata hai, guesswork se nahi.

## 33.1 The three pillars

```
┌──────────────────┬───────────────────┬─────────────────────────────┐
│      LOGS        │      METRICS      │          TRACES              │
├──────────────────┼───────────────────┼─────────────────────────────┤
│ "kya hua"        │ "kitna / kitni    │ "ek request ka poora safar" │
│ discrete events  │  baar / kitna     │ causal chain across services │
│                  │  time"            │                             │
│                  │ aggregated numbers│                             │
├──────────────────┼───────────────────┼─────────────────────────────┤
│ High detail      │ Low detail        │ High detail                 │
│ High volume/cost │ Cheap, long       │ Sampled (usually 1-10%)     │
│                  │ retention         │                             │
├──────────────────┼───────────────────┼─────────────────────────────┤
│ ELK, Loki,       │ Prometheus +      │ Jaeger, Tempo, Zipkin,      │
│ CloudWatch, Splunk│ Grafana, Datadog │ Honeycomb, X-Ray            │
├──────────────────┼───────────────────┼─────────────────────────────┤
│ "Why did THIS    │ "Is the system    │ "Where did the 4 seconds    │
│  request fail?"  │  healthy overall?"│  go?"                       │
└──────────────────┴───────────────────┴─────────────────────────────┘
```

**Debugging ka natural flow:** Metrics batate hain ki **kuch galat hai** (error rate spike). Traces batate hain **kahan** (payment service, DB call). Logs batate hain **kyun** (connection pool exhausted).

**Ye teen-line flow interview mein bolna** — ye dikhata hai ki tum tools nahi, **workflow** samajhte ho.

## 33.2 Structured logging

```python
# ❌ Unstructured — grep karo aur pray karo
log.info(f"PO {po_id} approved by {user} in {ms}ms")
# "PO PO-1001 approved by admin@merlin.co in 342ms"
# Query: "average approval time last hour?" -> regex parsing, painful

# ✅ Structured — queryable
log.info("po.approved", extra={
    "event": "po.approved",
    "po_id": po_id,
    "user_id": user_id,
    "amount_paise": amount,
    "duration_ms": ms,
    "correlation_id": cid,
    "service": "po-service",
    "environment": "production",
})
# {"event":"po.approved","po_id":"PO-1001","duration_ms":342,...}
```

Ab query possible hai: `event:po.approved AND duration_ms > 1000 AND environment:production` — aur ye ek **dashboard** ban sakta hai.

**QA ke liye iska direct use:** production logs se hi tumhe pata chal jaata hai ki kaunse flows sabse zyada use hote hain, kaunse errors sabse zyada aate hain, aur kaunsa edge case tumne socha hi nahi tha.

## 33.3 Distributed tracing

```
Trace ID: 7f3a2b1c...          total 1,240 ms
│
├─ [gateway]        POST /checkout                        1240 ms  ████████████
│  ├─ [auth-svc]    validate token                          15 ms  ▌
│  ├─ [cart-svc]    GET /cart/abc                           45 ms  ▌
│  ├─ [payment]     POST /charge                           890 ms  █████████
│  │  ├─ [psp-api]  external POST stripe/charges           850 ms  ████████  ← BOTTLENECK
│  │  └─ [db]       INSERT payments                         12 ms  ▌
│  ├─ [order-svc]   POST /orders                           180 ms  ██
│  │  ├─ [db]       INSERT orders                           18 ms  ▌
│  │  └─ [db]       SELECT inventory (N+1: 12 queries)     150 ms  █▌  ← BUG
│  └─ [kafka]       publish order.created                    8 ms  ▌
```

| Term | Matlab |
|---|---|
| **Trace** | Ek request ka poora journey, saare services ke paar |
| **Span** | Us journey ka ek operation (ek service call, ek DB query) |
| **Trace ID** | Poore trace ka unique id — **yahi correlation ID hai** |
| **Span ID / parent** | Parent-child relationship — causality banata hai |
| **Baggage** | Context jo saath travel karta hai (e.g. `is_test=true`) |
| **Sampling** | Kitne % traces store karo — 100% mehnga hai |

**QA ke liye tracing ke 3 killer uses:**

1. **Root cause in seconds.** Test fail hua → trace ID → dikhta hai ki payment service ne 850ms liya PSP ke liye aur timeout ho gaya. Bina trace ke ye 2 ghante ka kaam hai.

2. **Performance regression detection as a test:**

```python
def test_checkout_has_no_n_plus_one_queries(api, tracing):
    with tracing.capture() as trace:
        api.post("/checkout", json={...})
    db_spans = [s for s in trace.spans if s["kind"] == "db"]
    assert len(db_spans) < 15, (
        f"{len(db_spans)} DB queries in checkout — likely N+1. "
        f"Queries: {[s['name'] for s in db_spans]}"
    )

def test_checkout_p95_latency_budget(api, tracing):
    durations = [run_checkout_and_get_duration(api, tracing) for _ in range(20)]
    p95 = sorted(durations)[18]
    assert p95 < 2000, f"checkout p95 = {p95}ms, budget 2000ms"
```

**Ye "performance assertions as functional tests" wala idea senior-level hai.** Performance ko ek alag phase mein dhakelne ke bajaye, budget ko test mein daal do — regression turant pakda jaata hai.

3. **Test ke aakhri failure ko backend se jodna** (section 20 ka correlation ID).

## 33.4 Error tracking (Sentry / Rollbar)

Logs se alag: error tracking **exceptions ko deduplicate aur group** karta hai, stack trace ke saath, aur trend dikhata hai.

```
Sentry issue view:
  TypeError: Cannot read property 'amount' of undefined
  po-frontend · InvoiceForm.tsx:142
  ├─ 1,284 events · 340 users affected · first seen 3 days ago
  ├─ Regression: reappeared after release v2026.8.14
  ├─ Browsers: Chrome 128 (89%), Safari 17 (11%)
  └─ Breadcrumbs: navigated /invoices → clicked "Create" → API 200 → crash
```

**QA ke liye ye ek test backlog hai, ready-made:**

| Sentry se signal | QA action |
|---|---|
| Top 10 errors by user count | Har ek ke liye ek regression test likho |
| Naya error ek release ke baad | Turant investigate — release regression |
| Ek specific browser pe hi error | Cross-browser gap in coverage |
| Error jo test env mein kabhi nahi aaya | **Test data ya config gap** — sabse valuable finding |

**Ye ek line bolna:** *"I review the top production errors weekly and turn each into a regression test. That's the cheapest possible test-selection algorithm — real users have already told you where the bugs are, so you're not guessing at risk."*

## 33.5 Dashboards, ELK, and what QA should watch

```
┌─────────────────── QUALITY DASHBOARD (Grafana) ────────────────────┐
│  RELEASE HEALTH                                                     │
│  error rate ▁▁▁▂▁▁█▃▂▁  ← spike at 14:20, deploy was 14:18         │
│  p95 latency ▂▂▂▂▂▃▃▄▅▅  ← creeping up over the week                │
│  ─────────────────────────────────────────────────────────────────  │
│  BUSINESS FUNNEL                                                    │
│  PO created → submitted → approved → invoiced                       │
│    1200         980         890        820      (drop at submit?)   │
│  ─────────────────────────────────────────────────────────────────  │
│  TEST SIGNAL                                                        │
│  suite pass rate  ████████░░ 94%    flaky rate ▂▂▃▂ 1.8%            │
│  CI duration      ▄▄▄▅▅▆▆▆   ← trending up, investigate             │
│  escaped defects  3 this release (target ≤2)                        │
└─────────────────────────────────────────────────────────────────────┘
```

**Stack ke naam jo poochte hain:**
- **ELK** = Elasticsearch (store/search) + Logstash (ingest/transform) + Kibana (visualise). Modern variant: Elasticsearch + **Filebeat** + Kibana, ya Grafana **Loki** (sasta, label-based).
- **Prometheus** = metrics DB, pull model, PromQL query language. **Grafana** = visualisation (kisi bhi source pe).
- **OpenTelemetry** = vendor-neutral standard for traces + metrics + logs. Ye **aaj ka default answer** hai — vendor lock-in avoid karta hai.

## 33.6 SLI / SLO / Error budget

| Term | Matlab | Example |
|---|---|---|
| **SLI** (Indicator) | Jo **maapte** ho | successful requests / total requests |
| **SLO** (Objective) | **Target** jo tum khud rakhte ho | 99.9% success over 30 days |
| **SLA** (Agreement) | Customer ke saath **contract**, penalty ke saath | 99.5% or refund |
| **Error budget** | 100% − SLO = kitna fail hone ki **ijazat** hai | 0.1% of 30 days = **43 minutes** |

**Error budget ka poora point:** ye engineering aur reliability ke beech **ek objective conversation** deta hai.

```
Budget remaining > 50%   ->  Ship fast. Take risks. Experiment.
Budget remaining < 25%   ->  Slow down. More testing. Smaller releases.
Budget exhausted         ->  FEATURE FREEZE. Only reliability work ships.
```

**QA ke liye ye transformative hai.** *"Should we release?"* ek opinion-based argument se ek **data-based decision** ban jaata hai.

```
QA-owned SLIs jo main propose karunga:
  - critical journey success rate  (synthetic monitoring se)
  - checkout / PO approval completion rate
  - p95 latency of top 5 user journeys
  - escaped defect rate per release
  - production error rate for new code paths
```

**Ye interview mein bolna:** *"I'd want QA to own a couple of SLIs, not just report test results. If the release decision is 'the suite is green', that's a proxy. If it's 'we have 68% of our error budget left and the critical-journey SLI is above target', that's a decision. It also removes the QA-as-gatekeeper dynamic — nobody argues with the budget."*

## 33.7 Production signals as test backlog

```
┌────────────────────────────────────────────────────────────────┐
│  PRODUCTION SIGNAL              →   QA ACTION                   │
├────────────────────────────────────────────────────────────────┤
│  Top Sentry errors (weekly)     →   regression test each        │
│  Support tickets by category    →   test the top 3 categories   │
│  Slow endpoints from traces     →   performance budget test     │
│  Funnel drop-off point          →   exploratory session there   │
│  Feature usage analytics        →   re-prioritise coverage      │
│    (nobody uses feature X)      →     stop maintaining X's tests│
│    (everyone uses feature Y)    →     deepen Y's coverage       │
│  Incidents / postmortems        →   a test for each root cause  │
│  Browser/device distribution    →   right cross-browser matrix  │
└────────────────────────────────────────────────────────────────┘
```

**Sabse underused signal: feature usage analytics.** Aksar test suite ka 30% un features pe hota hai jo 2% users use karte hain, aur core flow pe coverage patli hai. Ye data se hi pata chalta hai.

**Postmortem → test ka rule:** har production incident ke baad ek test likho jo us exact failure ko pakadta. Ye "never again" ka concrete implementation hai. Agar test likhna possible nahi — wo bhi ek finding hai (matlab wo failure mode observable nahi hai, aur usko observable banana chahiye).

> **Interview answer:** Observability has three pillars. Logs are discrete events answering "why did this specific request fail" — high detail, high cost. Metrics are aggregated numbers answering "is the system healthy" — cheap and long-retention. Traces show one request's path across services, answering "where did the time go". The debugging flow is metrics tell you something is wrong, traces tell you where, logs tell you why. For QA specifically there are three concrete uses. First, correlation IDs — I generate one per test and inject it as a header, so a failing test links straight to the backend's view of that exact request, which turns a two-hour investigation into a two-minute one. Second, traces let me write performance assertions as ordinary functional tests: assert the checkout flow makes fewer than fifteen database spans, which catches an N-plus-one at code review time rather than in a load test three months later. Third, and most valuable, production signals become my test backlog — the top Sentry errors by affected users, the support ticket categories, the funnel drop-offs, and a test for every incident root cause. That's a far better test-selection algorithm than my own guesses about risk, because real users have already found the bugs. I'd also want QA to own an SLI or two rather than just reporting pass rates, because "we have sixty-eight percent of our error budget left" is a decision, whereas "the suite is green" is a proxy for one.

**Cross-question: "QA ko production access dena chahiye?"**
> Read access to observability tools, yes — logs, dashboards, traces, error tracking. Write access to production data, no. The asymmetry is deliberate: reading production is how QA learns what actually breaks, and denying it means the test strategy is built on guesswork. In practice the teams where QA can't see production are the ones where the suite drifts furthest from real risk.

**Cross-question: "SLO aur SLA mein difference?"**
> An SLO is an internal target you set for yourself; an SLA is an external contract with a customer that has financial or legal consequences. The SLO is always stricter than the SLA — you want to breach your own target and react well before you breach the contract. And the useful artefact is the error budget derived from the SLO: at ninety-nine-point-nine percent over thirty days you have about forty-three minutes of allowed downtime, and how much of that remains is what should drive whether you ship aggressively or slow down.

---

# 34. Trade-off Questions

> **Ye sawaal deliberately binary poochhe jaate hain, aur unka jawab kabhi binary nahi hota.** Junior "X better hai" bolta hai. Senior bolta hai **"it depends on ___"** aur phir wo blank bharta hai — ye hi wo cheez hai jo score karti hai.
>
> **Answer ka template:**
> 1. *"It depends on \_\_\_"* — decision variable name karo
> 2. *"I'd choose A when \_\_\_, and B when \_\_\_"* — dono side justify karo
> 3. *"In my current context I chose \_\_\_ because \_\_\_"* — apna concrete example
> 4. *"The cost of that choice is \_\_\_"* — apni choice ka nuksaan bhi bolo

| Trade-off | It depends on... | Choose A when | Choose B when |
|---|---|---|---|
| **UI vs API tests** | Kaunsi layer mein risk hai | UI: user-visible behaviour, integration of frontend+backend, critical journeys | API: business logic, edge cases, data validation, anything combinatorial (15x faster, 10x more stable) |
| **E2E vs Integration** | Confidence needed vs feedback speed | E2E: critical revenue paths, cross-service flows | Integration: everything else — most bugs are within a service |
| **Speed vs Coverage** | Kis stage pe ho | Speed: PR gate (5 min budget, smoke only) | Coverage: pre-deploy, nightly |
| **Mocking vs Real dependency** | Kis cheez ko test kar rahe ho | Mock: your logic, error paths, third-party you can't control | Real: the integration itself, contract correctness |
| **Record/replay vs Hand-written tests** | Suite ki lifetime | Record: throwaway, one-off exploration, legacy characterisation | Hand-written: anything maintained beyond a month |
| **In-house framework vs Vendor tool** | Team skill + control needs | In-house: engineering team, custom needs, long horizon | Vendor: small/non-technical team, fast start, standard flows |
| **BDD (Cucumber) vs plain code** | Kya business stakeholders SACH mein Gherkin padhte hain | BDD: genuine three-amigos collaboration happening | Plain: only engineers read the tests (usually true — then BDD is pure overhead) |
| **Parallel vs Sequential** | Test independence ka level | Parallel: tests self-contained, data isolated | Sequential: shared state that can't be isolated (fix that instead) |
| **Retry vs Fix** | Failure ka source | Retry: provably infra (network, 502, pod eviction) | Fix: everything else — retry hides product bugs |
| **Page Object vs Screenplay** | Scenario complexity | POM: standard CRUD apps, familiar to everyone | Screenplay: multi-actor scenarios, complex role interactions |
| **Test in prod vs Staging only** | Staging ki representativeness | Prod: staging isn't representative, need real config/integrations | Staging: destructive tests, or when prod risk isn't mitigable |
| **Shift-left vs Shift-right** | Where the unknowns are | Left: requirements are the weak point, defects escape early stages | Right: production is where surprises live (config, scale, real data) — do both |
| **Coverage % target vs Risk-based** | Kya measure kar rahe ho | Coverage %: never actually — it optimises for easy tests | Risk-based: always. Depth follows business impact |
| **Monorepo vs Separate test repo** | Ownership model | Monorepo: tests version with the code, devs contribute easily | Separate: multiple products, dedicated QA org, independent release cadence |
| **Selenium vs Playwright vs Cypress** | Constraints, not preference | Selenium: multi-language teams, legacy, real-device grid | Playwright: new projects, multi-tab/origin, speed, tracing. Cypress: frontend-team-owned, DX-first |
| **Synchronous vs Async Playwright** | Team + integration | Sync: pytest, simpler mental model, most QA teams | Async: high concurrency in one process, async app code |
| **Flat helpers vs Class hierarchy** | Suite size + team size | Flat: small-to-medium suites, few contributors, composability | Classes: 50+ pages, IDE discoverability and namespacing start to pay |
| **Build platform vs Buy CI** | Scale + budget | Build: unique orchestration needs at very large scale | Buy: almost always at first — rent compute, build only the scheduler |
| **Strict lint gates vs Trust** | Number of contributors | Strict: 10+ contributors, anything not enforced drifts | Lighter: 2-3 people who talk daily |

**Do trade-offs jo tumhare project se seedha aate hain — inhe prepare karke jao:**

> **[REAL] "Production pe test karna vs staging only"** — section 25.3 mein poora answer hai. Framing: staging representative nahi tha, isliye prod chuna, aur risk ko structurally mitigate kiya (QA prefix, additive-only, guarded deletes, no video). Cost: prod risk, aur limited destructive testing.

> **[REAL] "Functional helper modules vs Page Object classes"** — 6 suites pe class hierarchy ka indirection cost reuse benefit se zyada tha; functions freely compose karte hain. Cost: IDE discoverability kam hai, aur naye joiner ko batana padta hai ki kaunsa helper kahan hai. 50 pages pe main classes pe move karunga.

> **Interview answer (generic template jo kisi bhi trade-off pe kaam karega):** I'd avoid answering that as a binary, because the right answer changes with context. The variable here is [X]. I'd choose [A] when [condition], because [reason], and [B] when [condition], because [reason]. In my current project I chose [A] — specifically because [concrete constraint]. The cost of that choice is [honest downside], and the signal that would make me revisit it is [trigger]. Being explicit about what would change my mind is usually the most useful part of the answer.

**Cross-question: "Tum kabhi galat choose kiye ho? Kya seekha?"**
> Yes — the silent-failure one. We had a helper that wrapped a fill in a try/except and swallowed the exception, and it reported success regardless. The intent was to make tests resilient to a flaky field, which sounded reasonable at the time. What it actually did was let every verification after that point run against a state that was never set, and it survived a full release cycle before we caught it. The lesson I took wasn't "don't use try/except" — it was that a test that fails loudly costs you an hour, and a test that passes falsely costs you a release. So now the framework verifies its own postconditions, `fill` asserts the value actually landed, and bare excepts are a lint error rather than a code-review conversation.

---

# 35. Senior Scenario Questions

---

## 35.1 "Tests are flaky and CI takes 90 minutes. What exactly will you change and why?"

> **Ye Level-4 engineering-judgement question hai.** Interviewer dekh raha hai ki tum **prioritise** kar sakte ho ya nahi, aur kya tum **expected impact** bata sakte ho. "Main flaky tests fix karunga aur parallel chalaunga" — ye jawab nahi hai. Jawab hai: **kaunsa pehle, kyun, aur kitna faayda.**

### Step 0 — Measure before changing anything (Week 1)

**Kuch bhi badalne se pehle, data chahiye. Bina data ke tum guess kar rahe ho.**

```
Instrument karo:
  1. pytest --durations=50               → kaunse tests slow hain
  2. Per-phase timing                    → setup vs execution vs teardown
  3. Last 30 runs ka history             → flake rate per test
  4. Failure classification              → product bug / infra / test bug / data
  5. CI stage timing                     → checkout, install, build, test, report
```

**Aksar jo milta hai (typical distribution):**

```
90 min breakdown — a very common real shape:
  ├─ 12 min  environment setup (docker pull, npm install, browser download)
  ├─ 55 min  test execution
  │           ├─ 22 min  login/auth (every test logs in through the UI)
  │           ├─ 18 min  test data setup through the UI
  │           └─ 15 min  actual assertions
  ├─  8 min  the 6 slowest tests (bulk upload, report generation)
  ├─ 10 min  serial execution — no parallelism at all
  └─  5 min  reporting + artifact upload

Flakiness: 40 flaky failures/week, but 28 of them are 5 tests.
```

**Ye insight critical hai:** 90 minute mein se sirf **15 minute** actual testing hai. Baaki setup aur auth hai. Aur 70% flakiness **5 tests** se aa rahi hai. **Pareto everywhere.**

### The prioritised plan (impact/effort order)

| # | Change | Effort | Expected impact | Why first |
|---|---|---|---|---|
| **1** | **Auth via storage state** — login once per role, reuse | 1 day | **90 → 70 min** (-22 min) | Biggest single win, zero risk, no test changes |
| **2** | **Parallelise: `-n 4`** | 1 day | **70 → 25 min** | Mechanical, huge; requires data isolation (see #3) |
| **3** | **Unique-data-per-test + factory cleanup** | 3 days | Enables #2 safely; kills ~30% of flakiness | Prerequisite for parallelism |
| **4** | **Fix the top 5 flaky tests** | 3 days | 40 → 12 flaky failures/week | 70% of flakiness from 5 tests |
| **5** | **API setup instead of UI setup** | 4 days | **25 → 14 min** | Removes 18 min of UI data creation |
| **6** | **Cache dependencies + prebuilt docker image** | 1 day | **14 → 8 min** | Setup 12 min → 3 min |
| **7** | **Split suites: smoke on PR, full pre-deploy** | 2 days | **PR feedback: 8 min → 4 min** | Changes the felt experience most |
| **8** | **Shard across 4 CI machines** | 2 days | Full run 8 → 4 min | Only after per-test cost is fixed |
| **9** | **Flaky detection + quarantine with SLA** | 3 days | Prevents regression of #4 | Makes the fix permanent |
| **10** | **Move combinatorial cases UI → API** | ongoing | Long-term suite health | Structural, not a quick win |

**Result:**
```
Before:  90 min, 40 flaky failures/week
After :  PR feedback 4 min, full regression ~12 min, ~5 flaky failures/week
Effort:  ~4 weeks of focused work
```

### Why this order specifically

**Ye reasoning hi asli jawab hai:**

1. **Auth first** kyunki ye highest impact-per-effort hai, kisi test ko change nahi karta, aur ek din ka kaam hai. **Momentum aur credibility** deta hai — team ko turant faayda dikhta hai.

2. **Data isolation before parallelism** — ulta karoge to parallelism flakiness **badha** degi, aur team bolega "parallel chalane se tests toot gaye, wapas karo". Ek failed attempt ke baad dobara permission milna mushkil hai. **Order matters politically, not just technically.**

3. **Flaky fix before more parallelism** kyunki flaky suite mein parallelism ka faayda dikhta hi nahi — log waise bhi rerun kar rahe hote hain.

4. **Sharding last** kyunki ye **paisa** kharch karta hai. Pehle per-test cost theek karo, phir hardware pe kharch karo. Warna tum inefficiency ko scale kar rahe ho.

5. **Suite splitting** ka impact numbers se zyada **felt experience** pe hai. Developer ko 90 min ka run nahi jhelna — usko PR pe 4 min ka feedback chahiye. Full regression 12 min hai, wo background mein chal sakta hai.

### Flakiness ka root-cause taxonomy (ye bolna depth dikhata hai)

| Root cause | % (typical) | Fix |
|---|---|---|
| Timing / implicit waits | 35% | `expect()` web-first assertions; ban `sleep`; wait on condition not time |
| Test data collision | 25% | Unique data per test; own-data-only |
| Test order dependency | 15% | Isolate state; verify with `--random-order` |
| Environment instability | 10% | Health check before suite; fail fast with a clear message |
| Genuine product race condition | 10% | **Report as a bug — do not "fix" the test** |
| Third-party / network | 5% | Targeted retry at the client layer |

**Wo 10% product race condition wala row sabse important hai.** *"Some of what gets labelled flaky is the product actually being non-deterministic. If a test intermittently sees a stale value after an update, that's the product's eventual-consistency window leaking into the UI — and users hit it too. I raise those as bugs rather than adding a wait, because the wait hides a real user-facing problem."*

### What I would NOT do

| ❌ | Why |
|---|---|
| Blanket `--reruns 2` | Hides the problem; masks real intermittent product bugs; makes the suite 3x more expensive |
| Delete the flaky tests | Loses coverage silently; the underlying issue stays |
| Add `time.sleep` to "stabilise" | Slower AND still flaky — worst of both |
| Buy more CI machines first | Scales the inefficiency; costs money; doesn't fix flakiness |
| Rewrite the framework | Months of no value; the problems are mostly not framework problems |

> **Interview answer:** I'd start by measuring for a week before changing anything, because the intuition about where ninety minutes goes is usually wrong. In my experience the breakdown looks like twelve minutes of environment setup, twenty-two minutes of every test logging in through the UI, eighteen minutes of creating test data through the UI, and only around fifteen minutes of actual assertions — plus no parallelism at all. And flakiness follows Pareto: roughly seventy percent of flaky failures come from about five tests. So my order is driven by impact per unit of effort, and by dependencies. First, authentication via saved storage state — log in once per role and reuse it. That's a day's work, changes no test code, and typically removes twenty minutes. Second, unique data per test with factory cleanup, because that's the prerequisite for parallelism; if I parallelise before isolating data, flakiness gets worse and the team concludes parallelism doesn't work, and I won't get a second attempt. Third, parallelism itself, which takes seventy minutes to about twenty-five. Fourth, fix the top five flaky tests, since that's seventy percent of the flakiness for three days of work. Fifth, move test data setup from the UI to the API, which removes most of the remaining time. Then dependency caching and a prebuilt image, then splitting into a four-minute smoke suite on PRs with full regression pre-deploy — that last one changes the felt experience more than any raw number, because developers care about PR feedback, not total suite time. Sharding across machines comes last, deliberately, because it costs money and I don't want to scale an inefficiency. Net effect is roughly four minutes of PR feedback and twelve minutes of full regression, in about four weeks. What I wouldn't do is add blanket reruns, because some of what's labelled flaky is the product genuinely being non-deterministic, and hiding that means shipping it to users.

**Cross-question: "Agar management bole ek hafte mein theek karo?"**
> Then I'd do items one and two only — storage-state auth and dependency caching — which are both low-risk, don't require touching tests, and together take ninety minutes to roughly forty-five. I'd deliver that, then show the measurement data and make the case that the remaining wins need data isolation work that can't be safely rushed. Rushing parallelism without isolation would give me a fast suite that fails randomly, which is worse than a slow reliable one, and I'd say that explicitly rather than agreeing and then delivering something that erodes trust.

**Cross-question: "Ek test 8 minute leta hai. Kya karoge?"**
> First I'd look at what it's actually doing — usually most of that is setup, not verification, and moving setup to the API collapses it. If it's genuinely one long user journey, I'd ask whether it's testing one thing or five, because a fifteen-step test is usually five tests wearing a trench coat, and splitting it gives better failure localisation as well as parallelism. If it's irreducibly slow — a bulk upload of ten thousand rows, say — I'd tag it `slow`, move it out of the PR gate into the nightly run, and cover the same logic with a fast API-level test at a smaller scale. The key point is that a slow test isn't automatically a bad test; a slow test in the wrong pipeline stage is.

---

## 35.2 "How would you migrate 2000 Selenium tests to Playwright?"

### Step 1 — Clarify aur challenge the premise

> *"Before planning the migration — what's the actual problem we're solving? If it's flakiness, a rewrite might not fix it, because flakiness usually comes from test design, not the tool. If it's speed and maintenance cost, Playwright genuinely helps. Also: how many of those two thousand still provide value? In most suites of that age, a meaningful fraction are duplicated, testing removed features, or permanently skipped."*

**Ye pehla sawaal hi seniority dikhata hai** — "migrate karo" ko blindly accept karne ke bajaye, purpose poochna.

### Step 2 — Audit first (2 weeks)

```
2000 tests ka audit:
  ├─ 300  duplicated / testing the same thing
  ├─ 200  permanently skipped or @Ignore'd
  ├─ 150  testing removed or deprecated features
  ├─ 400  low-value: UI tests for pure business logic → belong at API level
  └─ 950  genuinely valuable

  → Migrate 950, not 2000. Delete 650. Convert 400 to API tests.
```

**Ye sabse valuable insight hai:** *"The cheapest migration is the one you don't do. I'd expect to delete or downgrade roughly half — migrating a test that shouldn't exist is pure waste, and a migration is the rare moment when deleting tests is politically possible."*

### Step 3 — Strategy: Strangler Fig, not big bang

```
   Month 0        Month 3         Month 6         Month 9
   ┌────────┐    ┌────────┐      ┌────────┐      ┌────────┐
   │Selenium│    │Selenium│      │Selenium│      │        │
   │  2000  │    │  1400  │      │   500  │      │  gone  │
   ├────────┤    ├────────┤      ├────────┤      ├────────┤
   │        │    │Playwrgt│      │Playwrgt│      │Playwrgt│
   │        │    │  300   │      │   700  │      │  950   │
   └────────┘    └────────┘      └────────┘      └────────┘
   
   BOTH suites run in CI throughout. Selenium shrinks as Playwright grows.
   No moment where coverage drops.
```

**Rules during migration:**
1. **All new tests are written in Playwright from day 1.** Selenium suite ko freeze karo — sirf critical fixes.
2. **Dono suites CI mein chalein** — coverage kabhi gap na ho.
3. **Migrate by risk, high-value first** — agar migration beech mein ruk gaya (jo aksar hota hai), tumne sabse zaroori hissa kar liya.
4. **Rewrite, don't transliterate.** Line-by-line port karoge to Selenium ke saare anti-patterns saath aa jaayenge — explicit waits, `Thread.sleep`, brittle XPath. Playwright ka auto-waiting model alag hai.

### Step 4 — What actually changes in the code

| Selenium pattern | Playwright equivalent | Note |
|---|---|---|
| `WebDriverWait(...).until(EC.visibility_of(...))` | `expect(locator).to_be_visible()` | Auto-waiting built in — most explicit waits just disappear |
| `driver.find_element(By.XPATH, "//div[3]/button[2]")` | `page.get_by_role("button", name="Approve")` | **Biggest stability win.** Rewrite locators semantically |
| `Thread.sleep(3000)` | delete it | Almost never needed |
| `StaleElementReferenceException` handling | delete it | Locators re-resolve automatically |
| `PageFactory` / `@FindBy` | plain attributes in `__init__` | Locators already lazy (section 14) |
| `driver.switch_to.frame(...)` | `page.frame_locator("#f").get_by_role(...)` | Cleaner |
| `Actions(driver).moveToElement().click()` | `locator.hover()` then `locator.click()` | First-class |
| New tab handling | `context.expect_page()` | Much simpler |
| Network stubbing (needs a proxy) | `page.route(...)` | **New capability** — enables error-state testing |
| Grid + video setup | `context.tracing.start()` | Trace viewer is a step change for debugging |

**Locator rewrite is where the real value is:** 2000 XPath selectors → semantic role/label selectors. Ye alone flakiness mein sabse bada drop deta hai, tool change se independent.

### Step 5 — Team enablement (aksar bhoola jaata hai)

```
Week 1  : 2-hour workshop — Playwright's model, auto-waiting, locator strategy
Week 2  : pair-migrate 20 tests together, live
Week 3+ : each engineer migrates their own team's tests, with review
Ongoing : a "migration cookbook" — Selenium pattern → Playwright pattern, with real examples
          plus a #playwright-migration channel for questions
```

**Migration ka sabse bada risk technical nahi, human hai.** Agar team Playwright ka mental model nahi samajhti, wo Selenium-style Playwright likhegi — explicit waits, XPath, `force=True` — aur tumhe koi faayda nahi milega.

### Step 6 — Success criteria (pehle se define karo)

```
Not "2000 tests migrated" — that's an output, not an outcome.

  ✓ Suite runtime      : 90 min → under 20 min
  ✓ Flaky rate         : 8% → under 2%
  ✓ Debug time per failure: ~40 min → under 10 min (trace viewer)
  ✓ Tests deleted      : ~650 (this is a WIN, report it as one)
  ✓ Contributors       : 3 people → 12 people writing tests
```

> **Interview answer:** I'd start by challenging the premise: what problem is the migration solving? If it's flakiness, a tool swap alone won't fix it, because most flakiness comes from test design — brittle XPath, sleeps, shared data — and those port straight over. If it's speed, debuggability and maintenance cost, Playwright genuinely helps. Then I'd audit before migrating, and I'd expect to find that of two thousand tests, several hundred are duplicated, permanently skipped, or testing removed features, and a few hundred more are UI tests for pure business logic that belong at the API level. So the plan is migrate around nine hundred and fifty, delete six hundred and fifty, and convert four hundred — and a migration is the rare political moment when deleting tests is actually possible. The approach is strangler fig, not big bang: all new tests go in Playwright from day one, the Selenium suite freezes, both run in CI so coverage never dips, and I migrate by risk so that if the effort stalls partway — which is common — the most valuable tests are already done. Critically I'd rewrite rather than transliterate, especially the locators: moving from positional XPath to role and label based locators is where most of the stability gain comes from, independent of the tool. And I'd invest heavily in team enablement, because the real failure mode is engineers writing Selenium-style Playwright with explicit waits and forced clicks, which gets you all of the migration cost and none of the benefit. Success is measured as runtime, flake rate and time-to-debug — not as tests migrated.

**Cross-question: "Migration ke dauraan ek critical bug production mein aa gaya, aur wo test Selenium suite mein tha jo abhi migrate nahi hua. Kya karoge?"**
> Fix it in whichever suite currently covers it — that's the Selenium one — because coverage now matters more than migration purity, and that's exactly why both suites keep running throughout. Then I'd note it as a signal: if bugs are landing in an area that's still on the old suite, that area's risk is higher than my migration order assumed, and I'd move it up the queue. The migration order should be responsive to what's actually breaking, not fixed at the start.

---

## 35.3 "Two teams' tests conflict on shared test data. Fix it."

### Diagnose first

```
Symptom: Team A ke tests randomly fail, aur sirf tab jab Team B chal raha ho.

Root cause ke 4 possibilities:
  1. Dono ek hi FIXED record use kar rahe hain     (e.g. supplier "ACME Cement")
  2. Dono GLOBAL state modify kar rahe hain        (feature flag, org setting, threshold)
  3. Ek team ka cleanup DUSRE ka data delete kar raha hai  ("delete all POs")
  4. Ek team ka test COUNT pe depend kar raha hai  ("table has 5 rows")
```

**Diagnose karne ka tareeka:** dono suites ko akele chalao (pass), phir saath chalao (fail). Phir logs mein us entity ka id dhoondo jispe conflict hai — aur uske creation/mutation ke saare events dekho, correlation ID se.

### The fix — layered, in order

| Layer | Fix | Time to implement |
|---|---|---|
| **Immediate (day 1)** | Namespace all created data: `QA-{team}-{test}-{uuid8}` | Hours |
| **Immediate** | Ban "delete all X" cleanup — delete only tracked ids | Hours |
| **Short term (week 1)** | Every test creates its own data; no test references a fixed record | Days |
| **Short term** | Make shared seed data **read-only** — a test that mutates it is a bug caught by review/lint | Days |
| **Medium (week 2-3)** | Global settings become scoped — per project/tenant, not per environment | Depends on product |
| **Long term** | Per-team or per-test tenant isolation (L4, section 30) | Weeks |
| **Guardrail** | CI check: fail if a test creates an entity without a unique suffix | Days |

```python
# ❌ THE BUG — both teams use the same record
def test_po_approval(api):
    supplier = api.get("/suppliers?name=ACME Cement").json()["items"][0]
    api.post("/po", json={"supplier_id": supplier["id"], ...})
    # Team B ne isi waqt ACME ko deactivate kar diya → Team A fail

# ❌ THE OTHER BUG — cleanup that isn't scoped
def teardown():
    api.delete("/po/all?status=DRAFT")     # Team B ke drafts bhi gaye

# ✅ THE FIX
def test_po_approval(po_factory):
    supplier = po_factory.supplier(approved=True)     # QA-procurement-po_approval-8f3a2b1c
    po = po_factory.purchase_order(supplier_id=supplier["id"])
    # teardown deletes only the ids this factory created, in reverse order
```

### Global state — the harder half

```python
# ❌ Ye poore environment ko affect karta hai
api.put("/settings/approval_threshold", json={"value": 500_000})

# ✅ Option A: scope it to the test's own project
api.put(f"/projects/{project['id']}/settings/approval_threshold", json={"value": 500_000})

# ✅ Option B: agar product scoping support nahi karta — ye ek PRODUCT GAP hai.
#    Raise it as such. Meanwhile, serialise those tests:
@pytest.mark.xdist_group("global_settings")
@pytest.mark.destructive
def test_approval_threshold_change(api, restore_settings):
    ...
```

**Option B ka framing bolna:** *"If a setting can only be changed globally, that's a product limitation with a real cost — it means two teams can't test independently, and it also means a customer admin can't scope that setting. I'd raise it as a product gap with that business framing, not just as a test problem. Meanwhile I'd serialise those specific tests and restore the setting in teardown."*

### Process fix, not just technical

```
1. A shared "test data contract" both teams agree to, in the repo:
     - every created entity carries QA-{team}-{test}-{uuid}
     - nobody deletes what they didn't create
     - shared seed data is read-only
     - global settings changes require a `destructive` marker
2. CI enforcement: a lint rule that fails on unscoped deletes and un-suffixed creates
3. A shared dashboard: entity counts per team, orphan records, cleanup failure rate
4. One owner for the shared environment — otherwise nobody owns it
```

> **Interview answer:** First I'd confirm the diagnosis rather than assume it, by running each suite alone — both pass — then together, where they fail. Then I'd find the specific entity from the logs and trace its creation and mutation using correlation IDs. There are usually four causes: both teams using the same fixed record, both mutating a global setting, one team's cleanup deleting the other's data with something like "delete all drafts", or a test asserting on a row count that another team's data changes. The immediate fix is namespacing — every created entity gets a `QA-team-test-uuid` prefix, and cleanup deletes only tracked IDs, never a wildcard. That's hours of work and stops the bleeding. Then each test creates its own data instead of referencing a fixed record, and shared seed data becomes strictly read-only. The harder half is global state: if a setting can only be changed environment-wide, no amount of test hygiene fixes it. I'd scope it to a project or tenant if the product supports that; if it doesn't, I'd raise it as a product gap — because a customer admin probably can't scope it either — and meanwhile serialise those tests with a marker so they never run concurrently. Finally I'd make it a process, not a heroic fix: a short written data contract both teams agree to, a lint rule in CI that fails on unscoped deletes and un-suffixed entity names, and a dashboard showing orphan records per team. Without the enforcement, it regresses within a quarter.

**Cross-question: "Sabse simple fix — dono teams ko alag environment de do. Kyun nahi?"**
> It's a valid option and sometimes the right one, so I'd cost it out rather than dismiss it. The downsides are real: environments multiply cost and drift from each other, each needs seeding and maintenance, and it doesn't scale past a few teams — with eight teams you get eight environments nobody fully owns. It also hides the underlying problem, which is that tests aren't self-contained, and that same problem will bite you the moment you enable parallelism within a team. So I'd fix isolation first, and use separate environments where there's a genuine reason — different product versions, or destructive testing that can't coexist with anything.

---

## 35.4 "How do you decide framework tech stack for a new project?"

### The decision framework — in priority order

```
1. TEAM SKILL          ← weighted highest, and most often ignored
2. APPLICATION STACK
3. TESTING NEEDS
4. ECOSYSTEM & MATURITY
5. CI/INFRA FIT
6. LONG-TERM MAINTENANCE
   ─────────────────────
   Personal preference   ← weighted LAST
```

| Factor | Questions to ask | Why it matters |
|---|---|---|
| **Team skill** | Team kaunsi language rozana likhti hai? Kaun tests maintain karega? | A "better" tool nobody knows is a worse tool. Adoption beats capability |
| **App stack** | Frontend framework? Backend language? | Same language as devs = devs contribute tests. This is the single biggest multiplier |
| **Testing needs** | Web only? Mobile? Desktop? API? Cross-browser? Real devices? Visual? | Determines hard requirements |
| **Ecosystem** | Libraries, hiring pool, docs, community, longevity | You'll live with this for years |
| **CI/infra** | Docker support? Parallel model? Cloud grid? Licence cost? | An unrunnable framework in CI is worthless |
| **Maintenance** | Who owns it in 2 years? How easy to onboard? | Frameworks outlive their authors |

### Worked example — how I'd reason out loud

> *"Say it's a React SPA with a Python backend, three QA engineers who know Python, and eight developers split between TypeScript and Python. Cross-browser matters but real devices don't. CI is GitHub Actions with Docker.*
>
> *I'd pick Playwright with Python and pytest. Playwright because auto-waiting removes the biggest source of flakiness, the trace viewer cuts debugging time dramatically, and it handles multi-tab and multi-origin flows natively. Python because all three QA engineers already write it and half the backend team does too, which means devs can contribute tests — that's worth far more than any feature difference between tools. Pytest because fixtures give me dependency injection and lifecycle management for free, and the plugin ecosystem covers parallelism, retries and reporting.*
>
> *What I'd give up: TypeScript Playwright has slightly better docs and examples, and Cypress has a nicer developer experience for frontend engineers. If the frontend team owned the tests rather than QA, I'd genuinely reconsider Cypress or TS Playwright — the question 'who maintains this' should drive the language choice more than the tool comparison does.*
>
> *I'd validate with a spike: build ten real tests covering the three hardest flows — file upload, a multi-step approval, and an iframe-based third-party widget — before committing. A stack that looks great in a tutorial and fails on your actual hard cases is the expensive mistake."*

### What I'd NOT base it on

| ❌ | Why |
|---|---|
| "It's the newest / trending" | Trend ≠ fit. And you inherit early-stage bugs |
| "I like it" | You'll leave; the team stays |
| GitHub stars | Popularity for a different use case is not evidence for yours |
| A vendor benchmark | Always measured on the vendor's happy path |
| "It's what my last company used" | Different constraints, different team |

> **Interview answer:** I weight team skill highest and personal preference last, which is the opposite of how these decisions usually get made. The most important question is who will write and maintain these tests in two years, because a technically superior tool nobody on the team knows is a worse choice than a good-enough tool they're fluent in — adoption beats capability. Second is matching the application stack, specifically the language, because if tests are in the same language as the product, developers contribute, and that's the single biggest multiplier on a test suite's long-term health. Then hard requirements from the testing needs — mobile, real devices, cross-browser, visual — because those eliminate options quickly. Then ecosystem maturity and CI fit. For a React frontend with a Python backend and a QA team that writes Python, I'd choose Playwright with Python and pytest: Playwright for auto-waiting and the trace viewer, Python because the whole team already reads it, pytest for fixtures and the plugin ecosystem. And I'd be explicit about the trade-off — TypeScript Playwright has better docs and Cypress has better DX for frontend engineers, so if the frontend team were the owners rather than QA, I'd reconsider. Finally, I'd validate with a spike of ten tests on the three hardest flows in the product before committing, because tools differentiate on hard cases, not on the login form in the tutorial.

**Cross-question: "Team Java jaanti hai lekin tum Python chahte ho. Kya karoge?"**
> Choose Java. Selenium or Playwright with Java and JUnit is entirely capable of a good suite, and my preference isn't worth the cost of a team that can't maintain their own tests. What I'd bring is the design — layering, one canonical way per action, data isolation, fixture-style lifecycle management — which transfers across languages. The framework quality comes from those decisions, not from the language. The one case where I'd argue harder is if the team's Java is only from a very old codebase and the product is entirely Python, since then the "team skill" advantage is weaker than it looks and the developer-contribution argument points the other way.

---

## 35.5 "How would you make a test suite that 20 engineers contribute to stay maintainable?"

### The core insight

> **Anything not enforced by CI will drift within one quarter.** 20 logon ke saath documentation, conventions aur good intentions kaam nahi karte. **Structure, automation aur ownership** kaam karte hain.

### The five mechanisms

**1. Ownership — CODEOWNERS**

```
# Tests: owned by the feature teams (they know the domain)
/tests/procurement/     @procurement-team
/tests/finance/         @finance-team
/tests/supplier_portal/ @supplier-team

# Framework: owned by a small group (protects the shared core)
/src/qa/core/           @qa-platform
/src/qa/infra/          @qa-platform
/conftest.py            @qa-platform
/pyproject.toml         @qa-platform
```

**Rationale:** tests scale with contributors; the **core does not**. Ek shared core jise 20 log edit karte hain, 3 mahine mein bikhar jaata hai. Core changes ko ek chhoti team se guzarna chahiye — ye gatekeeping nahi, **consistency** hai.

**2. Automated enforcement — the rules that actually matter**

```toml
# ruff / flake8
[tool.ruff.lint]
select = ["E", "F", "B", "S", "T20"]
ignore = []
# E722  bare except            ← the silent-failure bug
# S110  try-except-pass        ← same
# T201  print found            ← use logging
```

```python
# tests/meta/test_suite_conventions.py — the suite tests ITSELF
import ast, pathlib, pytest

TEST_FILES = list(pathlib.Path("tests").rglob("test_*.py"))

@pytest.mark.parametrize("path", TEST_FILES, ids=str)
def test_no_sleep_in_tests(path):
    src = path.read_text()
    assert "time.sleep" not in src, f"{path}: use wait_until(), not sleep"
    assert "wait_for_timeout" not in src, f"{path}: wait on a condition, not a duration"

@pytest.mark.parametrize("path", TEST_FILES, ids=str)
def test_no_raw_selectors_in_tests(path):
    src = path.read_text()
    for bad in ('page.locator("', "page.query_selector", 'By.XPATH'):
        assert bad not in src, f"{path}: selectors belong in page objects, not tests"

@pytest.mark.parametrize("path", TEST_FILES, ids=str)
def test_no_force_click(path):
    assert "force=True" not in path.read_text(), f"{path}: force=True is banned (see interactions.py)"

def test_every_test_has_a_suite_and_risk_marker(pytestconfig):
    """Har test ke paas exactly ek suite marker aur ek risk marker ho."""
    ...

def test_base_page_has_not_grown(...):
    """BasePage ka public method count 12 se zyada na ho (ISP guardrail)."""
```

**Ye "meta tests" pattern bahut strong hai interview mein** — suite khud apne conventions enforce karti hai, aur violation ek **failing test** hai, ek code-review opinion nahi.

**3. Make the right path the easy path**

```
- A template test file that already has the correct structure
- A cookiecutter / generator: `make new-test suite=procurement name=po_approval`
- The one canonical helper per action (section 17), well documented, easy to find
- Good error messages in helpers that point at the right fix
```

**Ye sabse underrated mechanism hai.** Agar sahi tareeka galat tareeke se aasan hai, log sahi tareeka choose karenge — bina kisi enforcement ke.

**4. Review discipline — a five-item checklist, not fifty**

```
PR review checklist (kept deliberately short):
  □ Does the test name state the business behaviour, not the mechanics?
  □ Does the test create its own data, and clean it up?
  □ Are assertions in the test, not in the page object?
  □ Is there exactly one suite marker and one risk marker?
  □ Would the failure message tell you what broke, without opening the code?
```

Lambi checklist koi nahi padhta. **5 items jo actually matter** — wo padhi jaati hai.

**5. Visible health metrics**

```
Weekly automated digest to the team channel:
  - suite runtime trend                    (creeping up? investigate)
  - flake rate by owning team              ← per team makes it actionable
  - tests added / deleted this week
  - quarantined tests and days remaining on SLA
  - slowest 10 tests, with owner
  - cleanup failure rate
```

**Per-team attribution** critical hai. "Suite mein 3% flakiness hai" — koi kuch nahi karta. "Finance team ke 4 tests 40% flakiness contribute kar rahe hain" — wo team us hafte fix kar deti hai.

### The anti-patterns to name

| ❌ | What happens |
|---|---|
| A 40-page "test writing guidelines" doc | Read once at onboarding, never again |
| One person reviewing every test PR | Bottleneck, then burnout, then rubber-stamping |
| No ownership on the framework core | Everyone edits, nobody maintains, it rots |
| Coverage % as the team's target | Optimises for easy tests; risk coverage stays flat |
| Quarantine without an SLA | Becomes a graveyard; coverage disappears silently |

> **Interview answer:** With twenty contributors, my operating assumption is that anything not enforced by CI drifts within a quarter — so I'd rely on structure, automation and ownership rather than documentation and good intentions. Four mechanisms. First, ownership through CODEOWNERS: tests are owned by the feature teams who understand the domain, but the framework core and conftest are owned by a small platform group, because tests scale with contributors and a shared core doesn't — twenty people editing the same interaction helpers produces five ways to do everything within months. Second, automated enforcement of the rules that actually matter: lint failing on bare excepts and sleeps, plus meta-tests in the suite itself that assert no raw selectors appear in test files, no forced clicks, every test carries exactly one suite marker and one risk marker, and the base page hasn't grown past a method cap. Making a convention a failing test rather than a review comment is what makes it stick. Third, make the right path the easy path — a template test, a generator command, one obvious canonical helper per action with good error messages. That's the mechanism that works without any enforcement at all. And fourth, visible health metrics attributed per team: runtime trend, flake rate by owning team, quarantine SLA countdown, slowest tests with owners. Attribution is what makes it actionable — a global flake rate gets ignored, but a team seeing their own four tests causing most of it fixes them that week. What I'd avoid is a long guidelines document, and a single reviewer for every test PR, because that becomes a bottleneck and then a rubber stamp.

**Cross-question: "Ek engineer consistently kharab tests likh raha hai. Kya karoge?"**
> I'd treat it as a systems question before a person question, because if one person consistently gets it wrong, the framework is probably making it easy to get wrong. So first I'd pair with them on two or three tests to see where the friction is — usually it's not knowing which helper exists, or the right way being genuinely harder. If it's a knowledge gap, pairing plus a better template fixes it. If it persists after that, it becomes a direct, private conversation with specifics — this test, this problem, this is what good looks like — not a passive-aggressive review comment. And I'd add a CI check for whatever the recurring issue is, so the feedback comes from the machine at the moment of writing rather than from a person days later, which is both faster and much less personal.

---

# 36. Red Flags — Ye Jawab Mat Dena

> Ye wo jawab hain jo **turant junior label** laga dete hain. Har ek ke saath likha hai ki interviewer ke dimaag mein kya chalta hai, aur uski jagah kya bolna hai.

## 36.1 OOP ke red flags

| ❌ Mat bolna | Interviewer sochta hai | ✅ Iski jagah bolo |
|---|---|---|
| "OOP ke 4 pillars hain — encapsulation, inheritance, polymorphism, abstraction" (aur bas) | Ratta maara hai, apply nahi kiya | Definition + **apne code se ek example** + kab NAHI use kiya |
| "Inheritance code reuse ke liye use karte hain" | Composition nahi samajhta | "Inheritance is for substitutability. For pure reuse I compose" |
| "Python mein `__` private hota hai" | Half-truth — name mangling nahi jaanta | "It triggers name mangling; the purpose is avoiding subclass collisions, not access control" |
| "Method overloading Python mein hota hai" | Basic galat | "Python doesn't support it — a second definition replaces the first. Workarounds are defaults, `*args`, or `singledispatch`" |
| "Maine POM use kiya hai" (aur bas) | Sirf tutorial-level | POM ka **problem** batao, 4 rules, aur ek mistake jo tumne ki thi |
| "Singleton config ke liye achha pattern hai" | Parallel execution nahi socha | "It's global mutable state — under parallel runs it's order-dependent flakiness. I use a session fixture" |
| "SOLID follow karta hoon" | Buzzword | Ek principle chuno aur **apne code ka before/after** dikhao |
| "`__str__` aur `__repr__` same cheez hai" | Detail miss | "repr is for developers and shows in assertion failures; str is for users" |
| "Abstract class banate hain taaki code reuse ho" | Purpose galat samjha | "For a contract that fails fast — a subclass that forgets to implement can't even be instantiated" |

## 36.2 Framework ke red flags

| ❌ Mat bolna | Interviewer sochta hai | ✅ Iski jagah bolo |
|---|---|---|
| "Framework mein Playwright, pytest, Allure use kiya hai" | Tool list = architecture nahi | **Layers + responsibilities + dependency rule + ek design decision aur uska trade-off** |
| "Flaky test aane pe retry laga dete hain" | Bug chhupata hai | "Retry only for provably infra failures. Otherwise it hides real intermittent product bugs" |
| "Wait ke liye `sleep` use karta hoon jab kuch aur kaam na kare" | Fundamentals kamzor | "Always wait on a condition. If nothing works, the app has no observable ready state — that's a testability bug worth raising" |
| "Har test ke liye alag environment chahiye" | Isolation ka aasan raasta dhoondh raha | "I isolate at the data layer with unique-per-test entities. Separate environments don't scale past a few teams" |
| "Coverage 80% target rakhta hoon" | Metric ko outcome samajh liya | "I allocate depth by risk. A coverage percentage optimises for easy tests" |
| "Sab kuch automate karna chahiye" | Cost/value nahi socha | "I automate what's repeated, stable and valuable. Exploratory testing finds things automation structurally can't" |
| "Assertions page object mein daal deta hoon, convenient hai" | POM ka point nahi samjha | "Then negative tests become impossible — the same method can't expect a different outcome" |
| "`force=True` laga do, click nahi ho raha" | Bug chhupa raha hai | "Force bypasses actionability checks, so a covered or disabled element 'succeeds' while the app does nothing. If I need it, the UI is telling me something real" |
| "`try/except: pass` se test crash nahi hota" | **Sabse khatarnak** | "That's a silent failure — the function reports success on a lie. We lost a release cycle to exactly this" |
| "Fixtures ko session scope de do, fast ho jaayega" | Isolation ka risk nahi socha | "Session scope means shared mutable state. I widen scope only after measuring, and I make session objects frozen" |

## 36.3 System design ke red flags

| ❌ Mat bolna | Interviewer sochta hai | ✅ Iski jagah bolo |
|---|---|---|
| Turant solution bolna shuru karna | Requirement gather nahi karta | 2-3 minute clarifying questions — **ye khud ek scored point hai** |
| "Main ek framework banaunga aur sab automate karunga" | Scale/risk nahi socha | Numbers do, risk map do, phased plan do |
| "Ye design perfectly scale karega" | Limits nahi jaanta | "This breaks first at X — the results DB becomes the bottleneck around N runs/day" |
| "Best practice ye hai..." | Context-free | "It depends on X. I'd choose A when..., B when..." |
| Trade-off nahi bolna | Sirf ek side dekhta hai | Har choice ka **cost** bhi bolo |
| "Microservices testing E2E se karenge" | Combinatorial explosion nahi samjha | "Contract tests for compatibility, E2E only for a handful of critical journeys" |
| "Payment testing mein card entry aur success page test karenge" | Risk-first nahi socha | "The highest-value test is charge succeeded but order creation failed" |
| "Eventual consistency ke liye `sleep(10)`" | Async model nahi samjha | "Poll with a timeout from p99 latency, and assert what the user sees during the window" |
| "Day 1 se framework banana shuru karunga" (new team) | Trust-building nahi samjha | "First 30 days: understand risk from data, deliver one small win" |
| "Production pe kabhi test nahi karna chahiye" | Absolutist, nuance nahi | "It depends on mitigation — additive, prefixed, guarded deletes. And production monitoring is not optional either way" |

## 36.4 Behavioural / framing ke red flags

| ❌ | ✅ |
|---|---|
| "Ye dev ki galti thi" | "The process let it through — here's the gap I'd close" |
| "Mujhe pata nahi" (aur chup) | "I haven't worked with that directly. My understanding is X — how does it work here?" |
| "Humara framework perfect hai" | "Here's what I'd change now that I've seen it under load" |
| "Maine akele poora framework banaya" (jab nahi banaya) | Apna actual contribution honestly bolo — **jhooth cross-question mein pakda jaata hai** |
| Sirf tools ke naam ginana | Har tool ke saath **kaunsa problem solve kiya** |
| Weakness ko chhupana | "We ran on production, which isn't ideal — here's why, and here's how we made it safe" |

**Sabse bada meta red flag:** **confidence bina nuance ke.** Har jawab mein absolute certainty = tumne alternatives nahi soche. Senior log hamesha bolte hain *"it depends"*, *"the cost of that is"*, *"I'd revisit if"*.

---

# 37. Quick Revision Table

## 37.1 OOP — one-liners

| Concept | One-line answer |
|---|---|
| Class vs Object | Blueprint vs instance with its own state |
| `__init__` vs `__new__` | `__new__` allocates, `__init__` initialises |
| Class vs instance variable | Class var shared across instances — mutable one leaks across tests |
| Encapsulation | Bundle data + methods, expose a controlled interface |
| `_` vs `__` | Convention vs name mangling (`_Class__attr`) — collision avoidance, not security |
| `@property` | Attribute syntax with method behaviour; converts without changing call sites |
| Inheritance | "is-a" + substitutability. Keep it shallow |
| `super()` | Next class in the **MRO**, not "the parent" |
| MRO / C3 | Deterministic linearisation; `Class.__mro__`; TypeError if inconsistent |
| Diamond problem | Resolved by C3; base method runs exactly once |
| Polymorphism | One interface, many implementations — kills if/elif chains |
| Duck typing | Type doesn't matter, capability does. `Protocol` = duck typing + static check |
| No overloading in Python | Name is a dict key; second def replaces first. Use defaults / `singledispatch` |
| Abstraction | ABC + `@abstractmethod` → fails at instantiation, not deep in a test |
| ABC vs Protocol | Nominal (must inherit) vs structural (just match the shape) |
| Composition vs Inheritance | "has-a" vs "is-a". Default to composition |
| Fragile base class | Base internals change → subclasses break, because they depend on implementation |
| `__repr__` vs `__str__` | Developer/unambiguous (shows in assertion failures) vs user/readable |
| `__eq__` + `__hash__` | Must go together; defining `__eq__` alone makes the class unhashable |
| `__enter__`/`__exit__` | Guaranteed cleanup. **Never `return True`** — it swallows exceptions |
| staticmethod vs classmethod | No binding vs `cls` — and `cls` is polymorphic, which is why factories use it |
| dataclass | Auto `__init__`/`__repr__`/`__eq__`; `default_factory` for mutables; `frozen=True` for shared objects |
| dataclass vs Pydantic | Internal model vs runtime validation at the API boundary |

## 37.2 SOLID

| | Principle | Test-automation example |
|---|---|---|
| S | One reason to change | Page object doing UI + API + DB + assertions → split |
| O | Open for extension, closed for modification | Browser registry instead of a growing if/elif |
| L | Subclass usable as base | `isinstance` checks appearing = LSP is broken |
| I | No fat interfaces | 40-method BasePage → minimal base + opt-in mixins |
| D | Depend on abstractions | Inject an `ApiClient` protocol, not `requests` |

## 37.3 Design patterns

| Pattern | One line | Framework use |
|---|---|---|
| Page Object | Page's locators + actions in one class | Locator change = one file |
| Factory | Centralised creation | Test data with unique ids + cleanup |
| Builder | Fluent step-by-step construction | Test data with many optional fields |
| Singleton | One instance globally | **Anti-pattern under parallel runs** — use a fixture |
| Strategy | Interchangeable algorithms | Auth: form / SSO / token / storage state |
| Facade | Simple API over a complex flow | `create_approved_po()` = 12 steps |
| Decorator | Wrap behaviour | Retry, timing, screenshot-on-failure |
| Observer | Publish/subscribe events | pytest hooks; `pytest_runtest_makereport` |
| Fluent interface | Method chaining | `login().goto_po().approve()` |

## 37.4 Framework architecture

| Topic | Key point |
|---|---|
| Layers | Test → Domain/Page → Core → Infra. **Dependencies only point downward** |
| Layer test | Could the core layer be lifted into another product unchanged? |
| POM 4 rules | No assertions inside; return something; private locators; business actions not UI mechanics |
| Page Factory | Java/Selenium lazy proxies; Playwright Locators are already lazy → not needed |
| BasePage | Only universal behaviour; `is_loaded()` abstract; guard against it becoming a magnet |
| Fixture scope | Default function; widen only after measuring; session = shared mutable state |
| Teardown | LIFO; only runs for fixtures that completed setup; `addfinalizer` for multi-step setup |
| One canonical way | One implementation per action; forbidden alternatives documented with reasons |
| Test data | Four layers; per-test factory is ~90%; unique ids = parallel safety |
| Config | defaults → file → env var → CLI; secrets only from env; loud startup banner; frozen |
| Logging | Console INFO / file DEBUG; correlation ID per test linking to backend logs |
| Reporting | Verdict first; failures with root cause + impact; pass count is one line; flaky separate |
| Retry | Only provably infra; push it down to the HTTP client; quarantine with an SLA |
| Parallel | xdist + CI sharding; balance shards by p95 duration; verify with random order |
| Tagging | `--strict-markers`; one suite marker + one risk marker per test |
| Environments | Guardrails in code: refuse prod without opt-in, block unscoped deletes, no video |

## 37.5 System design

| Topic | Key point |
|---|---|
| 6-step framework | Clarify → constraints → **risk** → high-level → deep dive → trade-offs |
| First scored point | Asking clarifying questions before designing |
| 5000 tests | Selection before parallelism; storage-state auth; balanced shards; automated flaky detection |
| Scale gradient | 500: run everything · 5000: tag-based selection · 50000: test impact analysis |
| Platform | Orchestrator → queue → autoscaled workers → grid → results DB → reporting |
| Sharding | Bin-pack by **p95** duration, not mean — total time = slowest shard |
| Flaky definition | Passes and fails **on the same commit** |
| Artifacts | Failure-only; traces are megabytes; lifecycle policy is mandatory |
| Payments | Highest-value test = charge succeeded + order creation failed |
| Idempotency | Same key + same payload = one charge; different payload = reject, not replay |
| Reconciliation | Test the safety net itself — orphan charges, amount mismatches, missing refunds |
| New team | 30 days learn + one small win · 60 foundation · 90 scale and hand over |
| Contract testing | Consumer-driven (Pact); catches compatibility, **not** business correctness |
| Test doubles | Dummy (unused) · Stub (canned input) · Spy (records) · Mock (expects) · Fake (works) |
| Stub vs Mock | Assert on a value = stub. Assert on a call = mock |
| Kafka | Ordering only within a partition; at-least-once → **idempotent consumers mandatory** |
| Saga | Local transactions + compensating actions in reverse; compensation ≠ rollback |
| Async testing | Never sleep. Poll with timeout = p99 × 3, or subscribe to the event |
| Eventual consistency | Test convergence, the convergence window, **and what the user sees during it** |
| Observability | Metrics = something's wrong · Traces = where · Logs = why |
| SLI/SLO/budget | 99.9% over 30 days ≈ 43 min budget; budget drives ship-or-slow-down |
| Production signals | Top errors, support tickets, funnel drops, incidents → your test backlog |

## 37.6 The five sentences worth memorising

> 1. **"A test that fails loudly costs you an hour. A test that passes falsely costs you a release."**
> 2. **"Retry is acceptable only when the failure is provably not about the system under test."**
> 3. **"I don't chase coverage percentage — I allocate depth by business risk."**
> 4. **"Anything not enforced by CI will drift within a quarter."**
> 5. **"The framework's job is to make change cheap and local — when the UI changes, exactly one file should change."**

## 37.7 Your project, in four talking points

> **[REAL] Ye chaar cheezein har interview mein aani chahiye:**
>
> 1. **One canonical way** — `_helpers/interactions.py` has exactly one implementation per action, with forbidden alternatives documented **with the reason each is forbidden**. `force=True` is banned because it bypasses actionability checks and converts a genuine UI bug into a green test.
>
> 2. **Keyword-only arguments** — `def click(locator, *, timeout_ms: int = 5000)`. A bare number at a call site tells you nothing; keyword-only makes intent visible at review time and keeps future parameters backward-compatible.
>
> 3. **The silent-failure bug** — a `try: fill() except: pass` reported success when the fill never happened. Every verification after it ran on a false premise and it survived a full release cycle. Fix was three-layered: lint ban on bare excepts, postcondition verification inside `fill`, and never swallowing exceptions without a specific type and a written reason.
>
> 4. **Deliberate architecture choices** — functional helper modules instead of a page-object hierarchy, because at six suites the indirection cost exceeded the reuse benefit; and end-to-end suites running on production, made safe structurally through QA-prefixed unique data, additive-only tests, guarded deletes and disabled video — with the honest acknowledgement that a representative staging environment would be the better investment.

---

**Ek aakhri baat:** is document ka har `> **Interview answer:**` block English mein hai aur bolne ke liye likha gaya hai. Ratta mat maaro — **structure** yaad rakho (definition → problem it solves → your concrete example → the trade-off). Wo structure kisi bhi variant question pe kaam karega.

Aur har answer ke aakhir mein ek cheez zaroor jodo: **"...the cost of that choice was ___"**. Wahi ek line junior aur senior ka farq hai.
