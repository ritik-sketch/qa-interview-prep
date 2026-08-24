# 01 — QA Fundamentals (Senior SDET Interview Prep)

> **Kaise use karein:** Har concept ka structure fixed hai — **Kya hai → Kyun matter karta hai → Kab use hota hai → Interview answer (English) → Cross-question**.
> Explanation Hinglish mein hai taaki concept dimaag mein baithe. Lekin jo blockquote mein `> **Interview answer:**` likha hai — **wahi bolna hai, English mein, waise ka waisa**. Ratta mat maaro, structure yaad rakho.
>
> `> **[REAL]**` wale boxes tumhare Merlin AI project ke actual examples hain. **Interview mein sabse zyada weight inhi ka hai.** Definition to har candidate bolta hai; jo banda apne project se concrete example deta hai wahi senior lagta hai.

---

## Table of Contents

| # | Section |
|---|---|
| 1 | [SDLC — Software Development Life Cycle](#1-sdlc--software-development-life-cycle) |
| 2 | [STLC — Software Testing Life Cycle](#2-stlc--software-testing-life-cycle) |
| 3 | [Requirement Analysis & Ambiguity Hunting](#3-requirement-analysis--ambiguity-hunting) |
| 4 | [Test Planning & IEEE 829](#4-test-planning--ieee-829) |
| 5 | [Test Strategy vs Test Plan](#5-test-strategy-vs-test-plan) |
| 6 | [Test Scenarios vs Test Cases](#6-test-scenarios-vs-test-cases) |
| 7 | [Test Case Writing](#7-test-case-writing) |
| 8 | [Test Data Management](#8-test-data-management) |
| 9 | [Test Execution](#9-test-execution) |
| 10 | [Defect Life Cycle](#10-defect-life-cycle) |
| 11 | [Severity vs Priority](#11-severity-vs-priority) |
| 12 | [Smoke vs Sanity vs Regression vs Retesting](#12-smoke-vs-sanity-vs-regression-vs-retesting) |
| 13 | [Levels of Testing — Unit, Integration, System, UAT](#13-levels-of-testing) |
| 14 | [Exploratory Testing (Session-Based)](#14-exploratory-testing-session-based) |
| 15 | [Negative Testing](#15-negative-testing) |
| 16 | [Black-Box Test Design Techniques](#16-black-box-test-design-techniques) |
| 17 | [Risk-Based Testing](#17-risk-based-testing) |
| 18 | [Test Estimation](#18-test-estimation) |
| 19 | [Requirement Traceability Matrix](#19-requirement-traceability-matrix-rtm) |
| 20 | [Test Metrics](#20-test-metrics) |
| 21 | [Senior-Level Scenario Questions](#21-senior-level-scenario-questions) |
| 22 | [Red Flags — Ye Jawab Mat Dena](#22-red-flags--ye-jawab-mat-dena) |
| 23 | [Quick Revision Table](#23-quick-revision-table) |

---

# 1. SDLC — Software Development Life Cycle

## 1.1 Kya hai

SDLC ek **framework** hai jo batata hai ki software banne ka process kaise chalega — requirement se lekar maintenance tak. Ye koi ek cheez nahi hai, ye **models ka collection** hai. Har model batata hai ki phases kis **order** mein aayenge aur unke beech **feedback loop** kitna tight hoga.

6 universal phases har model mein hote hain (naam alag ho sakte hain):

```
  ┌──────────────┐
  │ Requirement  │  Kya banana hai? BA/PM + stakeholders
  │  Gathering   │
  └──────┬───────┘
         v
  ┌──────────────┐
  │   Design      │  HLD (architecture) + LLD (module level)
  └──────┬───────┘
         v
  ┌──────────────┐
  │Implementation │  Developers code likhte hain + unit tests
  └──────┬───────┘
         v
  ┌──────────────┐
  │   Testing     │  QA — integration, system, UAT
  └──────┬───────┘
         v
  ┌──────────────┐
  │  Deployment   │  Release to production
  └──────┬───────┘
         v
  ┌──────────────┐
  │ Maintenance   │  Bug fixes, patches, enhancements
  └──────────────┘
```

## 1.2 Kyun matter karta hai (interview mein)

Interviewer ye isliye poochta hai ki dekhe **tum apne aap ko kahan fit karte ho**. Junior QA bolta hai "testing phase mein QA aata hai". Senior SDET bolta hai "**QA requirement phase se hi involve hota hai** — kyunki requirement mein pakda hua defect sabse sasta hota hai." Ye **shift-left** ka core argument hai.

**Cost of defect — ye number yaad rakho:**

| Kahan pakda | Relative fix cost |
|---|---|
| Requirement phase | 1x |
| Design phase | 5x |
| Coding phase | 10x |
| Testing phase (QA) | 15–50x |
| Production (customer ne dekha) | 100x+ |

## 1.3 Models — sab, aur kab kaunsa

### A) Waterfall Model

**Kya hai:** Sequential. Ek phase 100% khatam, phir agla shuru. Peeche jaana allowed nahi (ya bahut mehnga).

```
Requirements ──> Design ──> Implementation ──> Testing ──> Deployment ──> Maintenance
     (no going back — har phase ek "gate" hai)
```

**Pros:** Documentation solid, milestones clear, contract/audit ke liye best, fixed-scope fixed-price projects ke liye predictable.
**Cons:** Testing sabse aakhir mein — matlab defect sabse mehnge stage pe milta hai. Requirement change accommodate nahi hota. Customer ko working software bahut late dikhta hai.

**Kab use hota hai:** Requirements 100% frozen aur clear hon; regulated domains (defence, aviation avionics, medical device firmware, banking core migrations) jahan har phase ka signed document compliance ke liye chahiye; short projects with well-understood tech.

### B) V-Model (Verification & Validation Model)

**Kya hai:** Waterfall ka hi extension, lekin har **development phase ke saamne uska corresponding test phase** hota hai — aur wo test phase ki **planning parallel mein hoti hai**, execution baad mein. Yahi V-model ka asli point hai jo log interview mein miss karte hain: **test design left side pe hota hai, execution right side pe.**

```
 Requirements ────────────────────────────────> UAT / Acceptance Testing
     \                                                      /
      \  BRD/SRS                          Acceptance test plan banti hai yahin
       v                                                    ^
    System Design ──────────────────────────> System Testing
        \                                                  /
         v                                                ^
      Architectural / HLD ─────────────────> Integration Testing
           \                                             /
            v                                           ^
         Module / LLD ──────────────────────> Unit Testing
              \                                        /
               v                                      ^
                └──────── CODING ────────────────────┘
```

**Left side = Verification** ("Are we building the product **right**?" — reviews, walkthroughs, static testing, no code execution).
**Right side = Validation** ("Are we building the **right** product?" — actual execution).

**Pros:** Testing early plan hoti hai, har deliverable ka ek test level map hota hai, traceability strong.
**Cons:** Still rigid — requirement change pura V hila deta hai. No early prototype.

**Kab use hota hai:** Medical devices (IEC 62304), automotive (ISO 26262), avionics (DO-178C), defence. Jahan regulator poochta hai "is requirement ka test evidence dikhao".

### C) Iterative / Incremental Model

**Kya hai:** Product ko chhote increments mein banate hain. Har increment ka apna mini-SDLC. Incremental = naye features add hote jaate hain. Iterative = same feature ko refine karte jaate hain.

**Kab:** Bada product jise chunks mein deliver karna hai, requirements broadly known hain lekin detail evolve hogi.

### D) Spiral Model

**Kya hai:** Risk-driven. Har loop ke 4 quadrants: **Determine objectives → Identify & resolve risks (prototype) → Develop & test → Plan next iteration.** Har spiral ke start pe risk analysis mandatory.

**Kab:** High-risk, high-cost, R&D-heavy projects. Naya unproven tech. Jahan galat direction ka cost bahut zyada ho.

### E) Agile (Scrum / Kanban)

**Kya hai:** Chhote time-boxed iterations (sprints, usually 1–3 weeks) mein potentially shippable increment. Requirements user stories mein. Continuous feedback.

**Scrum ceremonies aur QA ka role — ye poocha jaata hai:**

| Ceremony | QA kya karta hai |
|---|---|
| Backlog grooming / refinement | **Ambiguity yahin pakadta hai.** Acceptance criteria challenge karta hai, edge cases raise karta hai |
| Sprint planning | Test effort estimate deta hai, "testable hai ya nahi" batata hai, definition-of-done pe agree karta hai |
| Daily standup | Blockers raise karta hai (env down, build not deployed, data missing) |
| Sprint review / demo | Acceptance criteria ke against demo verify karta hai |
| Retrospective | Escaped defects / leakage discuss karta hai, process improvement |

**Kanban** — no fixed sprints, continuous flow, WIP limits. Support/maintenance teams ke liye better.

**Kab Agile use hota hai:** Requirements evolve ho rahi hain, product-market fit dhoondh rahe hain, customer feedback fast chahiye. **Startups aur SaaS** — jaise Merlin.

### F) DevOps / CI-CD (Agile ka natural extension)

**Kya hai:** Dev + Ops ki wall hatana. Automated build, automated test, automated deploy. QA ka kaam yahan **pipeline mein quality gate banana** hai, manual gatekeeper banna nahi.

## 1.4 Comparison table

| Aspect | Waterfall | V-Model | Iterative | Spiral | Agile |
|---|---|---|---|---|---|
| Requirement stability needed | Very high | Very high | Medium | Medium | Low (change welcome) |
| QA involvement starts | Testing phase | Requirement phase (planning) | Each increment | Each spiral | Day 1, every ceremony |
| Working software visible | Very late | Very late | Early-ish | Medium | Every sprint |
| Handles change | Poorly | Poorly | Moderately | Well | Very well |
| Documentation weight | Heavy | Heaviest | Medium | Heavy | Light (just enough) |
| Risk handling | Implicit | Implicit | Incremental | **Explicit, primary driver** | Continuous |
| Best for | Fixed-scope contract | Safety-critical / regulated | Large phased rollout | High-risk R&D | Product / SaaS / startup |

> **[REAL]** Merlin AI par hum **Agile** follow karte hain — Jira/Linear mein stories, sprint-based delivery. Mera role sirf "sprint ke end mein test karna" nahi hai. Jab Project Sales V1 ka verification aaya, maine **5 defects nikale jisme 3 blockers the** — aur unme se kam se kam do defects **requirement-level ambiguity** the, coding bug nahi. Frontend blank `customerId` bhej raha tha **ye assume karke ki backend derive kar lega**, aur backend usko simply null kar de raha tha. Ye contract ambiguity hai — agar refinement meeting mein hi ye sawaal pooch liya jaata ki "customerId ka source of truth kaun hai, FE ya BE?", to ye defect kabhi code hi nahi hota. Yahi shift-left ka real example hai.

> **Interview answer:**
> SDLC is the framework that defines how software moves from requirement to maintenance. The phases are broadly the same — requirements, design, implementation, testing, deployment, maintenance — what changes between models is the ordering and how tight the feedback loop is.
>
> Waterfall is strictly sequential and works only when requirements are frozen, typically in fixed-scope or contractual work. The V-model pairs every development phase with a corresponding test level, and the key point is that the test design happens on the left side in parallel with development artifacts, while execution happens on the right — that is why it is preferred in regulated domains like medical devices and automotive, where you must produce test evidence for every requirement. Spiral is risk-driven, with explicit risk analysis at the start of every loop, so it suits high-risk R&D. Agile delivers a shippable increment in short iterations and is the right fit when requirements are still evolving.
>
> The reason this matters to me as a tester is where QA enters. In Waterfall, QA enters at the testing phase, which is the most expensive place to find a defect — a requirement defect caught late costs orders of magnitude more than one caught during requirement review. In Agile, I am involved from backlog refinement onward. At Merlin we run Agile, and in a recent feature verification, two of the blocking defects I found were essentially requirement-level contract ambiguities — the frontend assumed the backend would derive a field, the backend assumed the frontend would send it, and nobody had written down who owned it. That is a defect that costs nothing to prevent in a refinement meeting and a lot to fix after release.

> **Cross-question: "You said Agile — so does that mean no documentation and no test plan?"**
>
> > No. Agile says "working software over comprehensive documentation" — it says *over*, not *instead of*. What changes is the form factor, not the existence. Instead of a fifty-page test plan document signed once, I maintain a living test strategy, acceptance criteria on each story, a regression suite that acts as executable documentation, and a traceability link from story to test case in Jira. The documentation that survives is the documentation that gets read. A test plan nobody opens after week one has negative value because it goes stale and misleads people.

> **Cross-question: "In your project, do you actually follow pure Scrum?"**
>
> > It is Scrum-flavoured rather than textbook Scrum. We work in sprints with stories tracked in Jira and Linear, we have refinement and review, but like most product startups we allow in-sprint scope changes when something customer-facing breaks. I would not claim it is by-the-book Scrum because interviewers can tell, and honestly the value we get is from the feedback loop, not from ceremony purity.

> **Cross-question: "Which model would you pick if you were starting Merlin from scratch today?"**
>
> > Agile for the product surface, but with one V-model borrowing: for the money-critical paths — purchase orders, invoices, anything that hits a financial record — I would insist that acceptance-level tests are designed at the same time the requirement is written, before the code exists. Those flows have real-world consequences for a construction company, so they deserve the "test designed on the left side" discipline even inside an Agile process.

---

# 2. STLC — Software Testing Life Cycle

## 2.1 Kya hai

STLC SDLC ke andar ka testing-specific lifecycle hai. **6 phases**, aur har phase ke **entry criteria, activities, deliverables aur exit criteria** hote hain. Interview mein log phases to bata dete hain, lekin **entry/exit criteria aur deliverable** nahi bata paate — wahi differentiator hai.

```
┌───────────────────┐
│ 1. Requirement    │  --> RTM (draft), Automation Feasibility Report
│    Analysis       │
└─────────┬─────────┘
          v
┌───────────────────┐
│ 2. Test Planning  │  --> Test Plan, Effort Estimation, Risk register
└─────────┬─────────┘
          v
┌───────────────────┐
│ 3. Test Case      │  --> Test cases, Test scripts, Test data requirements
│    Development    │
└─────────┬─────────┘
          v
┌───────────────────┐
│ 4. Test Env       │  --> Env ready + smoke passed, Test data seeded
│    Setup          │      (runs PARALLEL to phase 3)
└─────────┬─────────┘
          v
┌───────────────────┐
│ 5. Test Execution │  --> Execution report, Defect reports, Updated RTM
└─────────┬─────────┘
          v
┌───────────────────┐
│ 6. Test Cycle     │  --> Test Summary Report, Test Closure Report,
│    Closure        │      Metrics, Lessons learned
└───────────────────┘
```

## 2.2 Phase-by-phase (full detail)

### Phase 1 — Requirement Analysis

| | |
|---|---|
| **Entry criteria** | BRD / SRS / user stories available; acceptance criteria drafted; stakeholders available for clarification |
| **Activities** | Requirements ko testable aur non-testable mein classify karna; ambiguity list banana; clarification questions raise karna; automation feasibility assess karna; risk identify karna |
| **Deliverables** | **RTM (initial)**, Automation Feasibility Report, list of open questions/assumptions |
| **Exit criteria** | Saare open questions ya to answered hain ya documented assumption ban chuke hain; RTM baseline ho gaya; sign-off from BA/PM |

### Phase 2 — Test Planning (kuch jagah "Test Strategy" bhi bolte hain)

| | |
|---|---|
| **Entry criteria** | Requirements baselined; RTM draft ready |
| **Activities** | Scope define karna (in-scope / out-of-scope); test approach decide karna; effort estimate; resource plan; tool selection; entry-exit criteria define karna; risk & mitigation; schedule |
| **Deliverables** | **Test Plan document**, Effort estimation sheet, Risk register, Test schedule |
| **Exit criteria** | Test plan reviewed aur **approved/signed-off** by stakeholders; estimation agreed |

### Phase 3 — Test Case Development

| | |
|---|---|
| **Entry criteria** | Test plan approved; requirements stable |
| **Activities** | Test scenarios derive karna; test cases likhna (steps + expected result); test data identify karna; automation scripts likhna; **peer review** karana |
| **Deliverables** | Test cases, automation scripts, test data requirement doc, **updated RTM** (requirement → test case mapping) |
| **Exit criteria** | Test cases peer-reviewed aur baselined; RTM shows **100% requirement coverage**; automation scripts committed aur dry-run pass |

### Phase 4 — Test Environment Setup

| | |
|---|---|
| **Entry criteria** | Environment architecture/config defined; hardware/software available; build deployed |
| **Activities** | Env configure karna; test data seed karna; access/credentials setup; third-party stubs/mocks configure karna; **smoke test on the environment itself** |
| **Deliverables** | Ready environment, environment config document, smoke test result |
| **Exit criteria** | Env accessible; smoke test passed; test data available and valid |

> **Note:** Phase 3 aur 4 usually **parallel** chalte hain — ye cross-question aata hai. Env setup ka dependency development pe hai, test case writing ka dependency requirement pe. Isliye dono ek saath.

### Phase 5 — Test Execution

| | |
|---|---|
| **Entry criteria** | Test cases baselined; env ready & smoke passed; build deployed with release notes |
| **Activities** | Test cases execute karna; actual vs expected compare karna; defect log karna; failed cases retest karna; regression run karna; RTM update karna; daily status report |
| **Deliverables** | Test execution report (pass/fail/blocked/not-run), defect reports, updated RTM, daily/weekly status |
| **Exit criteria** | Planned test cases executed (ya consciously deferred, documented); no open Critical/High defects (ya waived with written approval); exit criteria from test plan met |

### Phase 6 — Test Cycle Closure

| | |
|---|---|
| **Entry criteria** | Execution complete; defects triaged/closed/deferred |
| **Activities** | Metrics compile karna (defect density, DRE, leakage, coverage); exit criteria verify karna; retrospective/lessons learned; test artifacts archive karna; reusable assets identify karna |
| **Deliverables** | **Test Summary Report**, **Test Closure Report**, metrics dashboard, lessons learned |
| **Exit criteria** | Summary report signed off; artifacts archived; action items raised for next cycle |

> **[REAL]** Merlin ke P2P automation ke context mein STLC concretely aisa dikhta hai. Requirement analysis mein maine PO ke teen item types identify kiye — material, trade, custom — kyunki inka flow same nahi hai, aur automation feasibility yahin decide hui. Test case development mein 6 end-to-end suites bane, har ek 12–15 steps ka: PO create → supplier acknowledge → receive → invoice. Environment setup mera **sabse constrained phase** hai kyunki ye suites **production par chalti hain**, isliye env "setup" ka matlab hai safe test data aur reversible actions — main naya environment spin nahi kar sakta. Execution mein wahi date-picker wala issue aaya jo pehle "flaky" label ho gaya tha. Aur closure phase mein maine ye maintain kiya ki kaunsa failure genuine product defect tha aur kaunsa automation defect — kyunki agar ye distinction na rakho to team automation pe trust khona shuru kar deti hai.

> **Interview answer:**
> STLC has six phases: requirement analysis, test planning, test case development, test environment setup, test execution, and test cycle closure. What actually matters in each phase is not the activity list but the entry criteria, the deliverable, and the exit criteria — because those are what let you say objectively whether a phase is done.
>
> Requirement analysis produces the initial traceability matrix and the automation feasibility assessment, and it exits when every ambiguity is either answered or recorded as an explicit assumption. Test planning produces the test plan and estimation, and exits on stakeholder sign-off. Test case development produces reviewed test cases and scripts, and exits when the RTM shows full requirement coverage. Environment setup runs in parallel with case development, and it exits only when a smoke test passes on the environment itself with valid seeded data. Execution exits when planned cases are executed and there are no open critical or high defects, unless they are formally waived. Closure produces the test summary report and the metrics — defect density, removal efficiency, leakage — and the lessons learned that feed the next cycle.
>
> The one I would emphasise is environment setup, because in my current project the end-to-end suites run against production. That changes the meaning of "environment ready" — I cannot spin up a fresh environment, so readiness means safe, reversible test data and non-destructive actions, and that constraint feeds all the way back into how I design the cases.

> **Cross-question: "Which STLC phase do most teams get wrong, and why?"**
>
> > Environment setup and closure. Environment gets treated as an IT ticket instead of a testing deliverable, so execution starts on an environment where the data is stale or a downstream service is stubbed differently from production — and then every failure costs an hour of triage before you even know if it is a product bug. Closure gets skipped entirely because the next sprint has already started, which means nobody ever computes defect leakage, so the team never learns which test gaps keep letting bugs through. Skipping closure is how a team stays at the same quality level for two years.

> **Cross-question: "Can phases overlap? Is STLC strictly sequential?"**
>
> > No, it is not strictly sequential in practice. Test case development and environment setup overlap because they depend on different upstream inputs — cases depend on requirements, environment depends on the build. In an Agile team, execution of already-stable stories overlaps with case design for stories still in development. What must not overlap is execution starting before the environment has passed its own smoke check, because then you are generating noise instead of signal.

> **Cross-question: "What is the difference between exit criteria and definition of done?"**
>
> > Exit criteria are about a test phase or cycle — for example, zero open critical defects and ninety-five percent of planned cases executed. Definition of done is about a single story or increment — for example, code reviewed, unit tests added, acceptance criteria verified, no new critical defects, documentation updated. Exit criteria are the gate for a release; definition of done is the gate for a story. In Agile the definition of done does more day-to-day work, but you still need release-level exit criteria, otherwise every story is done and the release is still not shippable.

---

# 3. Requirement Analysis & Ambiguity Hunting

## 3.1 Kya hai

Requirement analysis matlab requirement ko **padhna nahi — interrogate karna**. Har requirement statement mein kuch likha hota hai aur bahut kuch **nahi** likha hota. Testing ka asli kaam wahi shuru hota hai jo nahi likha.

Ek **testable requirement** ke 6 properties hoti hain:

| Property | Matlab | Fail hone pe kya hota hai |
|---|---|---|
| **Complete** | Saara zaroori info present hai | Dev apna assumption bhar dega |
| **Unambiguous** | Sirf ek interpretation possible | FE aur BE alag samjhenge → integration defect |
| **Consistent** | Doosri requirements se conflict nahi | Do features ek doosre ko todenge |
| **Verifiable/Testable** | Pass/fail objectively decide ho sake | "System should be fast" — kaise test karoge? |
| **Traceable** | Unique ID, upstream/downstream link | Coverage prove nahi kar paoge |
| **Feasible** | Given tech/time mein ban sakta hai | Sprint mid-way mein scope explode |

## 3.2 Ambiguity kaise dhoondhein — 7 systematic lenses

Ye ek **checklist** hai. Har requirement ko in 7 lenses se guzaro, questions apne aap nikal aayenge:

1. **Ambiguous words** — "fast", "user-friendly", "appropriate", "properly", "as needed", "etc.", "and/or", "if required", "handle gracefully". Ye sab red flags hain. **"Etc." ka matlab hai requirement writer ne soch nahi rakha.**
2. **Missing quantifiers** — kitne? kitni der? kitna bada? kitni baar? Number nahi hai to requirement testable nahi hai.
3. **Missing negative path** — agar fail ho gaya to? Error message kya? Retry? Rollback? Partial success?
4. **Missing actor/permission** — kaun kar sakta hai? Kaun nahi? Kya hoga agar unauthorized user try kare?
5. **Missing state/lifecycle** — entity ki states kya hain? Kaunsi transition allowed hai, kaunsi nahi? Kya iss action ke baad state change hota hai?
6. **Missing boundaries** — min/max, empty, null, zero, negative, very large, duplicate, unicode.
7. **Missing integration contract** — data kahan se aata hai, kahan jaata hai, kaun sa field kaun populate karta hai, timeout kya hai, idempotent hai ya nahi.

## 3.3 Worked example — ek line ki requirement, 20+ clarifying questions

**Requirement (jaisa actually milta hai):**

> *"User should be able to upload documents to a purchase order."*

Bas. Ek line. Ab isko interrogate karte hain.

### Lens 1 — Ambiguous words
1. "Documents" ka exactly matlab kya? Kaun se **file types** allowed hain — PDF, JPG, PNG, DOCX, XLSX, CSV, ZIP? Aur **explicitly blocked** kya hai — `.exe`, `.sh`, `.svg` (XSS vector), macro-enabled `.docm`?
2. "Upload" — sirf browse-and-select, ya drag-and-drop bhi, ya paste-from-clipboard bhi? Mobile pe camera capture?

### Lens 2 — Quantifiers
3. **Maximum file size** kitna hai? Per file, aur total per PO?
4. Ek PO pe **maximum kitne documents** attach ho sakte hain?
5. **Multiple files ek saath** select ho sakti hain ya ek-ek karke?
6. Filename ki **max length** kya hai? Special characters, spaces, unicode, emoji allowed?

### Lens 3 — Negative path
7. Agar upload **beech mein fail** ho jaaye (network drop) — partial file save hoti hai ya rollback? User ko kya dikhta hai? Resume support hai?
8. Agar file **size limit cross** kare — validation client-side hai ya server-side ya dono? Error message ka exact text kya hai?
9. Agar user **wrong file type** de — reject kaise karein? Extension check ya actual MIME/magic-byte check? (Ek `virus.exe` ko `invoice.pdf` rename karke upload kar diya to?)
10. **Duplicate filename** upload hua to — overwrite, reject, ya auto-rename `invoice(1).pdf`?
11. Agar **storage service down** ho — PO save hota hai document ke bina, ya poora transaction fail?

### Lens 4 — Actor / permission
12. **Kaun** upload kar sakta hai — PO creator, approver, site engineer, supplier? Supplier apne acknowledgement ke saath document attach kar sakta hai?
13. **Kaun dekh sakta hai** upload ki hui file? Kya supplier internal documents dekh sakta hai?
14. **Kaun delete** kar sakta hai? Delete hard hai ya soft? Audit trail rehta hai?
15. Kya **unauthenticated user** direct file URL se file access kar sakta hai? (Ye security question hai — signed URL hai ya public bucket?)

### Lens 5 — State / lifecycle
16. PO ki **kaun si states** mein upload allowed hai? Draft mein haan — lekin approved PO mein? Received PO mein? Cancelled/closed PO mein?
17. Agar PO **approved ho chuka** hai aur koi document add karta hai — kya PO re-approval chahiye? Kya ye audit event hai?
18. PO **delete/cancel** hone pe attached documents ka kya hota hai — orphan, cascade delete, ya retain for audit?

### Lens 6 — Boundaries
19. **0-byte file** upload karne pe kya hoga?
20. Exactly limit ke barabar file (agar 10 MB limit hai to exactly 10 MB) — accept ya reject? Limit inclusive hai ya exclusive?
21. 500-character filename? Filename with `../../etc/passwd` (path traversal)?

### Lens 7 — Integration contract
22. File **kahan store** hoti hai — S3, GridFS, local disk? Response mein kya return hota hai — file ID, signed URL, ya raw path?
23. Kya upload **synchronous** hai ya background job? Agar background hai to UI kya dikhata hai jab tak process ho raha hai?
24. **Virus scanning** hota hai? Agar haan to scan ke pehle file visible hoti hai?
25. Kya ye document **invoice matching** ya kisi downstream flow mein use hota hai? Agar haan to us contract ka format kya hai?

### Non-functional (bonus, senior signal)
26. **Response time expectation** kya hai 10 MB file ke liye?
27. **Concurrent uploads** — 50 users ek saath upload karein to?
28. **Accessibility** — file input keyboard se operable hai? Screen reader upload status announce karta hai?
29. **Audit/compliance** — retention period? GDPR-style delete request pe file bhi delete hogi?

**Ye 29 questions ek line ki requirement se aaye.** Interview mein 8–10 bol dena kaafi hai — lekin **categories bolna** important hai, taaki lage ki tumhare paas method hai, list nahi.

> **[REAL]** Merlin ke Project Sales V1 verification mein ye exact skill kaam aayi. Requirement level pe ye kabhi define nahi hua tha ki **`customerId` ka source of truth kaun hai**. Frontend ne blank `customerId` bheja ye maan kar ki backend context se derive kar lega; backend ne usko simply null set kar diya. Result — **sale create ho gaya bina kisi customer ke**. Ye "coding bug" nahi hai, ye ek **unwritten integration contract** hai. Agar refinement mein sirf ek sawaal poocha jaata — "is field ko kaun populate karta hai, aur agar blank aaya to backend ka behaviour reject hai ya derive hai?" — to ye defect exist hi nahi karta.
>
> Isi feature mein doosra ambiguity gap tha **lifecycle ka**: offer `DRAFT` state mein mint hota tha, lekin **kisi ne define hi nahi kiya tha ki usko publish kaun karega**. Requirement mein "offer generate hoga aur customer ko email jaayega" likha tha — beech ka publish step kisi ne likha hi nahi. Isliye link-builder null return karta tha, aur email chupchaap ek legacy tokenless URL pe fall back kar jaata tha jise accept gate reject kar deta tha. **Missing state transition = missing requirement.**

> **Interview answer:**
> Requirement analysis for me is not reading the requirement, it is interrogating it. I run every requirement through a fixed set of lenses so that the questions are systematic rather than dependent on how alert I am that day.
>
> First, ambiguous vocabulary — words like fast, appropriate, gracefully, or "etc." Every one of those means the author has not decided yet. Second, missing quantifiers — how many, how large, how long, how often; if there is no number, the requirement is not verifiable. Third, the negative path — what happens on failure, on partial failure, on timeout. Fourth, actors and permissions — who can do this, who cannot, and what an unauthorised attempt returns. Fifth, state and lifecycle — in which states of the entity is this action legal, and which transitions are explicitly invalid. Sixth, boundaries — empty, zero, null, maximum, one over maximum, duplicate, unicode. Seventh, the integration contract — which side populates which field, what the response shape is, whether the call is idempotent.
>
> To make that concrete: given a one-line requirement like "user should be able to upload documents to a purchase order," I would ask which file types are allowed and which are explicitly blocked, whether type validation is by extension or by actual content signature, what the maximum file size and maximum attachment count are, whether the size limit is inclusive, what happens on a mid-upload network failure, who is allowed to upload and who is allowed to view — specifically whether a supplier can see internal documents — in which purchase order states uploading is still permitted, whether adding a document to an already-approved purchase order triggers re-approval, what happens to the attachments when the purchase order is cancelled, and whether the stored file is served through a signed URL or is publicly reachable. That last one has caught real security issues.
>
> The reason I take this seriously is that in a recent feature verification, two of the three blocking defects I found were requirement gaps, not coding errors. In one, the frontend sent a blank customer ID assuming the backend would derive it while the backend simply nulled it, so records were created with no customer attached — nobody had ever written down which side owned that field. In another, an offer was created in draft state and no requirement said who was responsible for publishing it, so the downstream link builder returned null and the customer email silently fell back to a legacy URL that the acceptance gate then rejected. A missing state transition is a missing requirement, and you find it by asking about lifecycle at refinement time, not by testing harder later.

> **Cross-question: "The BA says the requirement is clear and you are overthinking. How do you handle it?"**
>
> > I stop arguing in the abstract and make it concrete. I write down the two or three different behaviours the sentence permits and ask which one they mean — "if the field arrives blank, does the backend reject the request, derive the value, or store null?" Once you present it as three specific outcomes rather than as "this is ambiguous," the conversation stops being about my judgement and becomes a decision they have to make. If they still say it is obvious, I record my interpretation as a written assumption in the ticket and move on. Then if it turns out wrong, the conversation afterwards is about a documented assumption, not about blame — and in my experience the assumption getting written down is often what makes someone finally read it properly.

> **Cross-question: "How do you test a non-functional requirement like 'the system should be fast'?"**
>
> > As written, I cannot — it is not verifiable, so my first job is to convert it into something that is. I would push for a statement with a metric, a percentile, a load level, and a scope: for example, "the purchase order list endpoint returns in under 800 milliseconds at the 95th percentile with 200 concurrent users and a tenant holding 50,000 purchase orders." Now it is testable, and I can build a JMeter scenario for exactly that. If the business genuinely does not know the number, I measure the current behaviour first and present it back — people find it much easier to react to "it is currently 2.3 seconds, is that acceptable?" than to invent a target from nothing.

> **Cross-question: "You are given a story with no acceptance criteria. What do you do?"**
>
> > I write the acceptance criteria myself, in Given-When-Then form, and put them on the ticket asking the product owner to confirm or correct them. That is far more efficient than asking them to write it, because reviewing a draft takes two minutes and writing from scratch takes an hour, so the draft actually gets read. It also surfaces disagreement immediately — the moment I write "given the field is blank, when the request is submitted, then the API returns a 400," someone will say "no, it should derive it," and that is exactly the conversation I wanted.

---

# 4. Test Planning & IEEE 829

## 4.1 Kya hai

Test Plan ek **project-specific document** hai jo batata hai: **kya test hoga, kaise hoga, kaun karega, kab karega, kaunse resources chahiye, kya risks hain, aur kab hum bolenge ki testing done hai.**

Ye ek **decision document** hai, essay nahi. Har section mein ek decision hona chahiye.

## 4.2 IEEE 829 Test Plan sections — poori list

IEEE 829 standard 16 sections define karta hai. Interview mein 8–10 bol do, lekin poori list pata honi chahiye:

| # | Section | Kya likha jaata hai |
|---|---|---|
| 1 | **Test plan identifier** | Unique ID + version (e.g. `MERLIN-P2P-TP-v1.2`) — traceability aur audit ke liye |
| 2 | **Introduction** | Product ka context, objective, related documents (BRD, SRS, design docs) ke references |
| 3 | **Test items** | Exactly kaunse modules/builds/versions test ho rahe hain — build number level pe |
| 4 | **Features to be tested** | In-scope features, requirement IDs ke saath |
| 5 | **Features NOT to be tested** | **Out-of-scope, with reason.** Ye sabse important aur sabse zyada skip kiya jaane wala section hai |
| 6 | **Approach** | Test levels, types, techniques, manual vs automation split, tooling |
| 7 | **Item pass/fail criteria** | Ek individual test item kab "pass" mana jaayega |
| 8 | **Suspension criteria & resumption requirements** | Kab testing rok denge (e.g. smoke fail, env down >4hrs), aur resume karne ke liye kya chahiye |
| 9 | **Test deliverables** | Test cases, scripts, defect reports, execution report, summary report, metrics |
| 10 | **Testing tasks** | Task breakdown with dependencies |
| 11 | **Environmental needs** | Hardware, software, network, data, third-party access, licences |
| 12 | **Responsibilities** | Kaun kya karega — QA, dev, DevOps, BA, product |
| 13 | **Staffing and training needs** | Kitne log, kaunsi skill, kya training chahiye |
| 14 | **Schedule** | Milestones, dates, dependencies |
| 15 | **Risks and contingencies** | Risk register + mitigation + fallback |
| 16 | **Approvals** | Kaun sign-off karega, naam aur date ke saath |

**Yaad rakhne ka trick:** *"I INTRODUCE test ITEMS, say what's IN and what's OUT, my APPROACH, PASS criteria, SUSPEND rules, DELIVERABLES, TASKS, ENVIRONMENT, RESPONSIBILITIES, STAFF, SCHEDULE, RISKS, APPROVALS."*

## 4.3 Do sections jinpe senior candidates score karte hain

### "Features NOT to be tested"
Ye section batata hai ki tumne **consciously scope decide kiya** hai. Bina iske, koi bhi escaped defect tumhari galti ban jaata hai. Iske saath, wo ek **documented, accepted risk** hai.

Example likhne ka tareeka:
> *Out of scope: Browser support below Chrome 100 and Safari 15 — analytics shows under 0.4% of tenant traffic. Risk accepted by Product on 2026-08-10.*

### "Suspension criteria"
Batata hai ki tumhare paas **stop rule** hai. Bina stop rule ke QA team ek broken build pe 3 din waste kar deti hai.

Example:
> *Testing will be suspended if: smoke suite fails on the deployed build; more than 30% of planned cases are blocked by a single defect; the test environment is unavailable for more than 4 continuous hours. Resumption requires a new build with the blocking defect verified fixed and a green smoke run.*

## 4.4 Ek chhota, real-shaped test plan skeleton

```
1. Identifier      : MERLIN-P2P-TP-v1.0
2. Introduction    : P2P flow — PO create -> supplier acknowledge -> receive -> invoice.
                     Refs: PRD-P2P-v3, API contract v2.1
3. Test items      : Web app build 4.12.0, PO service 2.3.1, Notification service 1.8.0
4. In scope        : PO creation (material / trade / custom item types), supplier
                     acknowledgement, goods receipt (full + partial), invoice
                     generation and matching, bid flow
5. Out of scope    : Payment gateway settlement (vendor-owned, covered by vendor
                     contract tests); IE11 support (0% traffic); mobile native app
                     (not in this release). Risk accepted by Product.
6. Approach        : Risk-based. Automated E2E (Playwright + Python) for the 6 core
                     flows; API-level checks via Postman/requests for contract
                     validation; exploratory charters for new UI; JMeter for the PO
                     list endpoint under tenant-scale data.
7. Pass/fail       : A case passes only if all steps pass AND the resulting DB/API
                     state matches expected. Partial pass is a fail.
8. Suspension      : Smoke fail on deploy; >30% cases blocked by one defect; env down
                     >4 hrs. Resume on new build + green smoke.
9. Deliverables    : Test cases (Jira), automation suite, defect reports, daily
                     execution report, test summary report, metrics.
10. Environment    : Runs against PRODUCTION. Requires isolated tenant/test supplier
                     accounts, reversible/non-destructive data, and documented cleanup.
11. Responsibilities: QA — design/execute/report. Dev — fix + unit coverage.
                     DevOps — deploy + env access. Product — scope + risk acceptance.
12. Risks          : (a) Production execution — data pollution risk. Mitigation:
                     dedicated test entities + naming convention + cleanup step.
                     (b) Single QA — bus factor. Mitigation: documented suites in repo.
                     (c) Third-party notification delivery latency. Mitigation: poll
                     with bounded wait, assert on API state not on inbox.
13. Approvals      : QA lead, Engineering manager, Product owner.
```

> **[REAL]** Mere test plan ka sabse unusual section **environment** hai, kyunki mere E2E suites **production par** chalte hain. Iska matlab test plan mein extra constraints likhne padte hain jo normal staging-based plan mein nahi hote: dedicated test supplier entities, ek naming convention taaki real data se distinguish ho sake, non-destructive actions, aur explicit cleanup. Aur risk section mein data pollution ek **first-class risk** hai mitigation ke saath — ye wo cheez hai jo interviewer ko turant batati hai ki maine sirf template copy nahi kiya, apne actual constraints ke hisaab se socha hai.

> **Interview answer:**
> A test plan is a project-specific decision document — it states what will be tested, how, by whom, on what environment, what the risks are, and crucially when we will declare testing complete. The IEEE 829 structure covers identifier, introduction, test items, features to be tested, features not to be tested, approach, pass/fail criteria, suspension and resumption criteria, deliverables, tasks, environmental needs, responsibilities, staffing, schedule, risks and contingencies, and approvals.
>
> Of those, the two sections I care most about are the ones people skip. "Features not to be tested" is what converts an untested area from my personal oversight into a consciously accepted, signed-off risk — if a defect escapes there, the conversation is about a decision we made together, not about something I missed. And suspension criteria matter because without an explicit stop rule, a team will burn days testing a build that was never viable. On my project I define suspension as smoke failing on the deployed build, more than thirty percent of planned cases blocked by a single defect, or the environment being unavailable for more than four hours, with resumption requiring a fresh build and a green smoke run.
>
> One thing specific to my current plan is the environment section, because my end-to-end suites run against production. That forces constraints into the plan that a staging-based plan never needs — dedicated test supplier entities, a naming convention that makes test data distinguishable, non-destructive actions, and an explicit cleanup step — and it puts data pollution in the risk register as a first-class risk with a stated mitigation.

> **Cross-question: "In Agile, do you still write a full test plan?"**
>
> > Not a sixteen-section document per sprint, no — that would be waste. I write it once per release or per major epic, keep it short, and treat it as living. What I do not drop is the decision content: scope and explicit out-of-scope, the approach, the environment and data constraints, the risks, and the exit criteria. Those decisions have to exist somewhere whether or not the artifact is called a test plan. What Agile removes is the ceremony of the document, not the thinking behind it.

> **Cross-question: "Who approves the test plan, and what if they never read it?"**
>
> > Formally, the QA lead, the engineering manager, and the product owner. Realistically, people skim it. So I do not rely on the document to communicate — I pull out the two or three items that actually need a decision, usually the out-of-scope list and the risk acceptances, and I walk through just those in a ten-minute conversation. Approval on the things that carry consequences is real approval. Approval on a document nobody opened is theatre, and it will not protect anyone when a defect escapes.

> **Cross-question: "What is the very first thing you would write in a test plan for a feature you have never seen?"**
>
> > The risks, and immediately after that the out-of-scope list. Everything else in the plan is derived from those — the approach, the depth of coverage, and the schedule all follow from what you are most afraid of and what you have consciously decided not to look at. If I start with the schedule I end up writing a plan that describes activity; if I start with risk I end up writing a plan that describes intent.

---

# 5. Test Strategy vs Test Plan

## 5.1 Kya hai

Ye **guaranteed** poocha jaata hai, aur log ghalti karte hain. Simple frame:

- **Test Strategy** = **organisation/product level**, long-lived, batata hai **"hum testing kaise karte hain, generally"**. Static-ish document. Approach aur standards.
- **Test Plan** = **project/release level**, short-lived, batata hai **"iss release mein specifically kya, kab, kaun"**. Dynamic document.

**Ek line mein:** *Strategy is the "how we test, as a company." Plan is the "what we test, this release."*

## 5.2 Full comparison

| Aspect | Test Strategy | Test Plan |
|---|---|---|
| Level | Organisation / product / program | Project / release / sprint |
| Scope | Broad, applies across projects | Narrow, one release |
| Lifespan | Long-lived, rarely changes | Changes every release |
| Written by | QA Manager / Head of Quality / Architect | Test Lead / Senior QA |
| Answers | *How* do we approach testing? | *What, when, who* for this release? |
| Contains | Test levels, types, entry/exit standards, automation policy, tooling standards, defect management process, risk approach, metrics definitions | Scope, in/out, schedule, resources, environment, specific risks, deliverables, exit criteria for this release |
| Changes when | Process/tooling changes | Every release |
| Derived from | Business/quality objectives | Test strategy + this release's requirements |

**Relationship:** Test Plan **strategy ko follow karta hai**. Strategy bolti hai "hum har release pe automated regression chalayenge aur zero open Sev-1 pe hi ship karenge." Plan bolta hai "iss release mein regression suite ka ye subset chalega, ye 3 features out of scope hain, aur 12 October ko sign-off hai."

## 5.3 Real content difference

**Strategy mein likha hoga:**
> *All customer-facing money flows must have automated end-to-end coverage. No release ships with an open Severity-1 defect. API contract changes require contract tests before merge. Automation is written in Playwright with Python; page objects are mandatory; no hard-coded waits.*

**Plan mein likha hoga:**
> *Release 4.12 covers PO creation for material, trade and custom item types plus the bid flow. Payment settlement is out of scope. Execution window 3–9 October, single QA resource, running against production with dedicated supplier accounts. Sign-off 10 October.*

> **[REAL]** Mere case mein strategy aur plan ka farq bilkul concrete hai. **Strategy-level rule** ye hai: har critical P2P money flow — PO create se lekar invoice tak — ka automated end-to-end coverage hona chahiye, Playwright + Python mein, aur locator strategy resilient honi chahiye — koi positional selector nahi, koi hard-coded wait nahi. Ye rule kisi ek release ka nahi hai, ye har release pe apply hota hai. **Plan-level baat** ye hai ki iss particular release mein kaunse item types cover ho rahe hain, kis window mein execute hoga, aur production execution ke liye kya data-safety constraints hain. Date-picker wale bug ne mera **strategy-level rule change** kiya — "text match pe `.first` mat use karo jab tak tumne uniqueness prove na ki ho" ab ek standing rule hai, sirf ek test ka fix nahi.

> **Interview answer:**
> A test strategy is an organisation or product level document that describes how we approach testing in general — which test levels and types we use, our automation policy and tooling standards, our entry and exit standards, how defects are managed, and how metrics are defined. It is long-lived and changes only when the process or tooling changes.
>
> A test plan is release or project level. It describes what specifically is in scope and out of scope for this release, the schedule, the resources, the environment, the release-specific risks, the deliverables, and the exit criteria for this particular sign-off. It changes every release, and it is written to comply with the strategy.
>
> The relationship is that the plan implements the strategy. For example, at the strategy level we hold that every critical money flow must have automated end-to-end coverage and that automation must not use positional selectors or hard-coded waits. At the plan level I state that this release covers purchase order creation for three item types plus the bid flow, that payment settlement is out of scope with product sign-off, and that execution runs against production with dedicated supplier accounts and an explicit cleanup step.
>
> A useful test of whether something belongs in the strategy is whether it would still be true next release. If yes, it is strategy. If it has a date or a build number in it, it is a plan.

> **Cross-question: "Can a project have a test plan without a test strategy?"**
>
> > It can, and small startups usually do — but it means the strategy exists implicitly in people's heads instead of on paper, which works exactly until the second QA joins or the first person leaves. What I would do in that situation is extract the strategy from the plans: after two or three releases, the rules that keep repeating in every plan are the strategy, and I would lift them out into a one-page document. Writing it up front from scratch in a startup usually produces a document that describes an aspiration rather than what the team actually does.

> **Cross-question: "Give me one example of something that people wrongly put in a strategy."**
>
> > Names and dates. The moment a document says "Ritik will test module X between the third and the ninth," it is a plan, not a strategy, because it expires. Similarly, listing specific test cases in a strategy is wrong — the strategy should say "boundary analysis is mandatory on all numeric inputs," and the plan or the test case repository holds the actual boundary values.

---

# 6. Test Scenarios vs Test Cases

## 6.1 Kya hai

- **Test Scenario** = ek **high-level "kya verify karna hai"** statement. One-liner. User ke perspective se. Steps nahi hote.
- **Test Case** = us scenario ko verify karne ke liye **detailed, executable steps** with exact data aur exact expected result.

**Ek scenario ke andar aam taur pe 3–15 test cases hote hain** (positive, negative, boundary variations).

```
Requirement
    │
    ├──> Scenario 1 ──┬──> Test Case 1.1 (positive, valid data)
    │                 ├──> Test Case 1.2 (negative, invalid data)
    │                 ├──> Test Case 1.3 (boundary, min value)
    │                 └──> Test Case 1.4 (boundary, max+1)
    │
    └──> Scenario 2 ──┬──> Test Case 2.1
                      └──> Test Case 2.2
```

## 6.2 Comparison

| Aspect | Test Scenario | Test Case |
|---|---|---|
| Level | High level, "what" | Detailed, "how" |
| Contains | One-line objective | Preconditions, steps, data, expected result, postconditions |
| Perspective | User / business | Tester / execution |
| Effort to write | Low | High |
| Count | Fewer | Many (3–15 per scenario) |
| Who reads | BA, PM, business stakeholder | QA executor, automation engineer |
| When written | Right after requirement analysis | After scenarios are agreed |
| Changes | Rarely | Often (data, UI labels) |

## 6.3 Scenarios kaise derive karein requirement se — 6 techniques

1. **User journeys / flows** — end-to-end path trace karo. "PO create karke supplier ko bhejna."
2. **Actors ke hisaab se** — har role ke liye alag scenario. Buyer, supplier, approver, admin.
3. **CRUD** — har entity pe Create, Read, Update, Delete, plus List/Search/Filter/Export.
4. **States ke hisaab se** — entity ki har state pe wahi action try karo. Draft PO edit, approved PO edit, cancelled PO edit.
5. **Business rules ke hisaab se** — har rule ek scenario. "PO amount > 5 lakh needs director approval."
6. **Failure modes** — har external dependency fail hone pe kya. "Notification service down ho to PO create hota hai?"

## 6.4 Worked example

**Requirement:** *"A buyer can create a purchase order, send it to a supplier, and the supplier can acknowledge it."*

**Derived scenarios:**

| ID | Scenario | Derivation lens |
|---|---|---|
| SC-01 | Verify buyer can create a PO with a material item and submit it to a supplier | User journey |
| SC-02 | Verify buyer can create a PO with a trade item | User journey / variation |
| SC-03 | Verify buyer can create a PO with a custom item | User journey / variation |
| SC-04 | Verify PO cannot be submitted with mandatory fields missing | Failure / negative |
| SC-05 | Verify supplier receives notification and can view the PO | Actor: supplier |
| SC-06 | Verify supplier can acknowledge a submitted PO | State transition |
| SC-07 | Verify supplier cannot acknowledge an already-acknowledged PO | **Invalid** state transition |
| SC-08 | Verify supplier cannot acknowledge a cancelled PO | **Invalid** state transition |
| SC-09 | Verify buyer cannot edit a PO after supplier acknowledgement | State-based permission |
| SC-10 | Verify a buyer from Tenant A cannot view a PO belonging to Tenant B | Security / authorisation |
| SC-11 | Verify PO amount above approval threshold routes for approval before submission | Business rule |
| SC-12 | Verify PO creation behaviour when the notification service is unavailable | Failure mode |
| SC-13 | Verify PO totals recalculate correctly when line quantity or rate is edited | Business rule / calculation |
| SC-14 | Verify PO list supports search, filter by status, and pagination at scale | CRUD / read |

**SC-06 ko test cases mein expand karo:**

| TC ID | Test case | Type |
|---|---|---|
| TC-06.1 | Supplier acknowledges a PO in SUBMITTED state with all lines accepted → status becomes ACKNOWLEDGED | Positive |
| TC-06.2 | Supplier acknowledges with partial quantity on one line → status and remaining quantity are correct | Positive variant |
| TC-06.3 | Supplier attempts acknowledgement with quantity greater than ordered → rejected with validation message | Negative |
| TC-06.4 | Supplier acknowledges with quantity zero on all lines → behaviour matches spec (reject vs decline) | Boundary |
| TC-06.5 | Acknowledgement is attempted twice rapidly (double-click / duplicate request) → only one state change occurs | Concurrency / idempotency |
| TC-06.6 | Buyer sees the acknowledged state and acknowledgement timestamp on the PO | Downstream verification |
| TC-06.7 | Acknowledgement by a user from a different supplier org → 403, no state change | Security |

> **[REAL]** Mere 6 end-to-end suites basically **scenarios** hain — har ek 12–15 steps ka full journey: PO create → supplier acknowledge → receive → invoice. Lekin scenario level pe main sirf ye kehta hoon "material item type ka full P2P cycle." **Test case level pe** decide hota hai ki exactly kaunsi quantity, kaunsa supplier, partial receive hai ya full, aur har step ke baad kaunsa exact state assert hona chahiye. Yahi wajah hai ki ek scenario ke andar bhi item type ke hisaab se **alag test cases** chahiye — material, trade aur custom ka flow same nahi hai, isliye ek "generic PO" test case likhna coverage ka illusion deta, coverage nahi.

> **Interview answer:**
> A test scenario is a one-line, high-level statement of what needs to be verified, written from the user or business perspective — for example, "verify that a supplier can acknowledge a submitted purchase order." A test case is the detailed, executable form of that: preconditions, exact steps, exact data, expected result, and postconditions. One scenario typically expands into somewhere between three and fifteen test cases covering the positive path, the negative paths, and the boundaries.
>
> The reason the distinction matters is that they serve different audiences. Scenarios are what I review with the business analyst and product owner, because they can read fourteen one-liners and tell me what is missing. Nobody from the business will review two hundred detailed test cases. So scenarios are my coverage-agreement artifact, and cases are my execution artifact.
>
> To derive scenarios I work through fixed lenses rather than brainstorming: end-to-end user journeys, then one pass per actor, then CRUD on each entity, then one pass over the entity's state machine including the transitions that must be rejected, then one scenario per business rule, and finally one per external dependency failure. That last two lenses are where most of the interesting scenarios come from — for instance, "supplier cannot acknowledge an already-acknowledged purchase order" and "purchase order creation behaviour when the notification service is unavailable" only appear if you deliberately look for invalid transitions and failure modes.

> **Cross-question: "If you are short on time, would you write scenarios or test cases?"**
>
> > Scenarios, without hesitation. Scenarios give me coverage visibility and let me have the "what are we not covering" conversation with product, which is the highest-value conversation available. Detailed cases mainly buy repeatability and hand-off ability. If I am the one executing and I am short on time, I write scenarios as charters and test exploratorily against them, then write up the cases for whatever turned out to matter. What I never do is write beautifully detailed cases for scenario one while scenarios eight through fourteen do not exist yet — that is optimising depth before coverage, and depth on the wrong area is worth nothing.

> **Cross-question: "How do you know your scenario list is complete?"**
>
> > Completeness in the absolute sense is not achievable, so I use several independent signals. First, traceability — every requirement and acceptance criterion maps to at least one scenario. Second, the derivation lenses — I can show I did an actor pass, a state pass, a business rule pass and a failure-mode pass, so the gaps are at least not systematic. Third, review — I get a developer and the product owner to read the list, because developers find missing technical edge cases and product finds missing business cases. Fourth, feedback from production — if a defect escapes, I ask which lens would have caught it and add that lens permanently. That last one is the only real measure, because it is measured against reality rather than against my own imagination.

---

# 7. Test Case Writing

## 7.1 Full template

| Field | Kya bharna hai | Example |
|---|---|---|
| **Test Case ID** | Unique, structured | `TC-PO-ACK-006` |
| **Module / Feature** | Kis area ka hai | Purchase Order → Supplier Acknowledgement |
| **Requirement ID** | Traceability link | `REQ-PO-14`, `US-482` |
| **Title / Objective** | Ek line, action + expected outcome | Supplier acknowledges a submitted PO with full quantity |
| **Preconditions** | Test start hone se pehle system kis state mein hona chahiye | A PO exists in SUBMITTED status with 1 material line, qty 100; supplier user has portal access |
| **Test Data** | Exact values, ya data reference | PO `PO-2026-0455`; supplier `Test Supplier A`; qty `100`; UOM `bags` |
| **Steps** | Numbered, ek action per step, unambiguous | 1. Log in as supplier user. 2. Open PO `PO-2026-0455`. 3. Enter acknowledged qty `100`. 4. Click **Acknowledge** |
| **Expected Result** | Observable, specific, **per step jahan zaroori ho** | PO status changes to `ACKNOWLEDGED`; acknowledgement timestamp is recorded; buyer's PO detail page reflects the same status; `GET /api/po/{id}` returns `status: ACKNOWLEDGED` |
| **Postconditions** | Test ke baad system state (cleanup ke liye zaroori) | PO remains in ACKNOWLEDGED state; no email queue backlog |
| **Priority** | P1/P2/P3 — execution order decide karta hai | P1 |
| **Type** | Functional / Negative / Boundary / Security / Regression | Functional |
| **Automation status** | Automated / Manual / Candidate | Automated (`test_po_material_flow.py::test_supplier_ack`) |
| **Actual Result** | Execution ke time bharte hain | — |
| **Status** | Pass / Fail / Blocked / Not Run / Skipped | — |
| **Defect ID** | Fail hone pe link | — |
| **Author / Date** | Ownership | Ritik C. / 2026-08-12 |

## 7.2 Achhe test case ke 10 properties

1. **Atomic** — ek case ek cheez verify kare. Agar ek case 4 cheezein check karta hai aur step 2 fail ho gaya, tumhe pata hi nahi chalega ki 3 aur 4 kaam karte hain ya nahi.
2. **Independent** — kisi doosre test case ke output pe depend na kare. Warna order badla to sab girega, aur parallel execution possible nahi hoga.
3. **Repeatable / Deterministic** — 10 baar chalao, 10 baar same result. Aaj ki date pe depend na kare, random data pe depend na kare.
4. **Specific expected result** — "should work correctly" nahi. "Status becomes `ACKNOWLEDGED` and `GET /api/po/{id}` returns `status: ACKNOWLEDGED`" — aisa.
5. **Clear preconditions** — koi bhi banda bina tumse poochhe execute kar sake.
6. **Traceable** — requirement ID linked ho.
7. **Self-cleaning** — apna data khud clean kare ya reversible ho (production pe ye **non-negotiable** hai).
8. **Right abstraction level** — "click the third button" nahi, "click **Acknowledge**". UI cosmetic change se case na tootey.
9. **Verifies state, not just UI** — sirf toast message dekhna kaafi nahi, underlying data verify karo.
10. **Reasonable length** — 10–15 steps se zyada matlab shayad ye actually 2 cases hain (E2E journeys exception hain, lekin waha bhi checkpoints chahiye).

## 7.3 Common mistakes — aur fix

| Mistake | Kyun bura hai | Fix |
|---|---|---|
| "Verify PO module works" | Testable hi nahi | Ek specific behaviour + specific expected result |
| Expected result: "should work" / "no error" | Fail/pass decide nahi kar sakte | Exact observable outcome likho |
| No preconditions | Doosra banda execute nahi kar payega | Starting state explicitly likho |
| Steps mein assertion mix | Step aur verification alag cheez hain | Steps = actions; expected result = separate column |
| Hardcoded volatile data (today's date, incrementing ID) | Kal fail karega | Relative/generated data ya explicit setup |
| Ek case mein 6 alag validations | Failure isolate nahi hota | Alag cases banao |
| Chained cases (TC-2 depends on TC-1) | Ek fail = cascade fail; parallel run impossible | Har case apna setup kare |
| UI-position-based steps ("click 3rd row") | UI change pe toot jaayega | Semantic identifiers use karo |
| Vague test data ("some valid value") | Reproducible nahi | Exact value likho |
| No negative cases | Half the risk untested | Har positive ke saath negative socho |

## 7.4 Do versions — bura aur achha

**Bura:**
```
TC-01: Test the date picker
Steps: 1. Open form. 2. Select a date. 3. Save.
Expected: Date should be saved properly.
```
Problems: kaunsa form? kaunsi date? "properly" kya hai? past date allowed hai? verify kahan karein?

**Achha:**
```
TC-PO-DATE-003
Requirement : REQ-PO-09 (delivery date must not be in the past)
Precondition: Logged in as buyer; PO create form open; today = D
Test data   : Attempt date = D-1 (yesterday)
Steps       : 1. Click the "Expected Delivery Date" field to open the calendar.
              2. Navigate to the current month.
              3. Attempt to select the day for D-1.
Expected    : The D-1 cell for the CURRENT month is rendered disabled
              (aria-disabled="true") and is not selectable; the field value
              remains unchanged; no API call is fired.
Postcondition: Form still in draft; no PO created.
Type        : Negative / Boundary
Priority    : P2
```

> **[REAL]** Date-picker wale bug ne mujhe test case writing ke baare mein sabse bada sabak diya. Purana automation code calendar mein **text match pe `.first`** use kar raha tha. Problem ye thi ki Mantine ka calendar **6x7 grid** render karta hai jo previous aur next month ke days se padded hota hai, aur wo padded days **wahi CSS class share karte hain** — matlab 42 cells mein se **11 day-numbers duplicate** hote hain. Upar se `minDate` ki wajah se past days disabled the. To `.first` aksar ek **disabled outside-month day** pick kar leta tha → 30 second timeout.
>
> Isse do rules nikle jo ab main har case pe apply karta hoon: **(1) kabhi bhi `.first` mat use karo jab tak tumne prove na kiya ho ki match unique hai** — locator ko current month aur enabled state se scope karo; aur **(2) agar test ka result aaj ki date pe depend karta hai to wo test deterministic nahi hai.** Ye failure "flaky" dikh raha tha lekin bilkul deterministic tha — ye purely aaj ke **day-of-month** pe depend karta tha. Ek flaky test aur ek date-dependent test alag cheez hain, aur unka fix bhi alag hai.

> **Interview answer:**
> A good test case has an ID, a link to the requirement for traceability, explicit preconditions, exact test data, numbered unambiguous steps, a specific observable expected result, and postconditions. But the properties that actually make it good are more important than the template.
>
> It must be atomic, so that a failure tells you exactly one thing. It must be independent, because chained cases mean one failure cascades and you cannot parallelise. It must be deterministic — if the result depends on today's date or on random data, it is not a test, it is a coin flip. The expected result must be specific and observable; "should work correctly" is not an expected result. And it should verify state, not just the UI response — a success toast on screen is not proof that the record was persisted correctly.
>
> The mistake I have personally learned the most from is non-determinism. In my project, an automated step selecting a date in a calendar looked flaky — sometimes it passed, sometimes it hit a thirty-second timeout. It was not flaky at all, it was fully deterministic; it just depended on today's day of the month. The calendar renders a six-by-seven grid padded with days from the previous and next month, and those padded cells share the same CSS class as real ones, so eleven of the forty-two day numbers are duplicated. The code matched on the day text and took the first match, which frequently landed on a disabled out-of-month cell. Two rules came out of that and I now apply them everywhere: never take the first match unless you have proved the match is unique, and if a result varies with the calendar, the test is not deterministic and must be fixed rather than retried.

> **Cross-question: "Do you write detailed test cases or do you prefer checklists?"**
>
> > It depends on who will execute them and how often. For a stable, regulated, or hand-off-heavy flow, detailed cases earn their cost because someone else must be able to run them identically. For a fast-moving feature I am testing myself this week, detailed cases go stale before they are executed a second time, so I prefer a checklist plus exploratory charters and I invest the saved time in automating whatever turns out to be worth repeating. The deciding question is: will this be executed by someone other than me, or more than a handful of times? If yes, detail it. If no, a checklist plus automation is a better use of the same hours.

> **Cross-question: "How do you keep test cases from going stale?"**
>
> > Three things. First, write them at the right abstraction — refer to the button by its label rather than its position, so a layout change does not invalidate the case. Second, automate the ones that get run repeatedly, because an automated case that goes stale fails loudly, whereas a manual document goes stale silently and nobody notices for a year. Third, prune deliberately — during closure I look for cases that have never failed and never covered a real risk, and I retire them. A suite that only grows eventually costs more to maintain than the defects it catches, and at that point people start ignoring it, which is worse than not having it.

> **Cross-question: "Should the expected result include the API response, or just the UI?"**
>
> > Both, when the case is verifying something that persists. The UI can show a success message while the write silently failed or stored the wrong value — I have seen exactly that shape of bug, where the flow appeared to succeed and the underlying record was created without a required reference. So for anything that creates or changes state, I assert on the persisted state through the API as well as on the UI signal. For purely presentational cases, UI alone is fine.

---

# 8. Test Data Management

## 8.1 Kya hai

Test data = wo inputs aur pre-existing records jo test ko execute karne ke liye chahiye. **Testing ka sabse underrated bottleneck yahi hai.** Zyadatar "flaky" tests actually **data problems** hote hain, code problems nahi.

## 8.2 Test data sources — 4 strategies

| Strategy | Kya hai | Pros | Cons | Kab use karein |
|---|---|---|---|---|
| **Production copy (raw)** | Prod DB ka dump | Realistic volume, realistic weirdness | **PII/compliance violation**, huge, slow | Almost never — legally risky |
| **Masked / anonymised production** | Prod data with PII scrambled | Realistic shape + volume, compliant | Masking pipeline maintain karna padta hai; referential integrity break ho sakti hai | Performance testing, migration testing, realistic regression |
| **Synthetic (generated)** | Faker/scripts se banaya gaya | Full control, no PII, unlimited volume, deterministic | Real-world weirdness miss karta hai (weird unicode names, legacy nulls) | Functional & automation testing — **default choice** |
| **Manually crafted "golden" data** | Handpicked edge-case records | Precise edge coverage | Doesn't scale, maintain karna padta hai | Boundary and negative cases |

**Best practice = combination:** synthetic for bulk + golden records for edge cases + masked prod subset for realism checks.

## 8.3 Masking techniques

| Technique | Kya karta hai | Example |
|---|---|---|
| **Substitution** | Real value ko realistic fake se replace | `Rajesh Kumar` → `Amit Sharma` |
| **Shuffling** | Column ke andar values ko shuffle | Salaries same column mein reshuffle |
| **Nulling / redaction** | Value hata do | `PAN: ABCDE1234F` → `XXXXXXXXXX` |
| **Encryption** | Reversible masking | Tokenised card number |
| **Number/date variance** | Value ko ±x% shift | DOB shifted by random days |
| **Format-preserving masking** | Format same, value fake | `9876543210` → `9123456789` (still valid mobile format) |

**Key rule:** masking **referential integrity preserve karni chahiye** — agar `customer_id` mask hua hai to orders table mein bhi consistently wahi mask apply hona chahiye, warna joins toot jaayenge aur test noise generate karenge.

## 8.4 Test data ke design principles (automation ke liye)

1. **Self-sufficient tests** — har test apna data khud banaye (setup fixture), doosre test ke residue pe depend na kare.
2. **Unique identifiers per run** — timestamp/UUID prefix use karo taaki parallel runs collide na karein. `PO-AUTOTEST-{uuid4}`.
3. **Cleanup / teardown** — jo banaya, wo hatao (ya at least mark karo).
4. **No shared mutable state** — ek shared "test PO" jise 5 tests modify karte hain = guaranteed flakiness.
5. **Data as code** — factories/builders code mein, spreadsheets mein nahi. Version-controlled.
6. **Seed reproducibility** — random data use karo to seed log karo, taaki failure reproduce ho sake.
7. **Boundary data explicitly designed** — empty string, max-length, unicode, zero, negative, very large number — ye accident se nahi aate, deliberately banane padte hain.

## 8.5 Data challenges — aur handle kaise karein

| Challenge | Handling |
|---|---|
| Data expires (e.g. valid-for-30-days record) | Setup mein freshly create karo, static mat rakho |
| Third-party dependency (real supplier email) | Dedicated test accounts + catch-all inbox, ya API-level assertion |
| Data volume for performance test | Synthetic bulk generation script, masked prod subset |
| Tests interfering with each other | Namespacing + isolated tenants + parallel-safe unique data |
| PII in logs/screenshots | Mask in reporting layer, not just in DB |
| State pollution across runs | Idempotent setup: "ensure exists" instead of "create" |

> **[REAL]** Mere case mein test data problem sabse tricky hai kyunki suites **production par** chalti hain. Iska matlab hai: main database wipe nahi kar sakta, main freely bulk data generate nahi kar sakta, aur mera har created record **real reporting mein dikh sakta hai**. Isliye mera approach hai — **identifiable, isolated, reversible**. Test entities clearly identifiable naming ke saath, dedicated test supplier accounts, aur non-destructive steps. Aur automation mein **har run apna PO khud create karta hai** — main kabhi bhi kisi pre-existing PO pe depend nahi karta, kyunki production mein wo PO kisi aur ne modify kar diya hoga aur mera test bina kisi product bug ke fail ho jaayega. Ye discipline "flaky suite" aur "trustworthy suite" ke beech ka farq hai.

> **Interview answer:**
> Test data is where most so-called flakiness actually originates, so I treat it as a design problem rather than a chore. There are four broad sources: raw production data, which I would avoid because of the privacy and compliance exposure; masked or anonymised production data, which is right when I need realistic volume and realistic messiness, for example in performance or migration testing; synthetic generated data, which is my default for functional and automated testing because it gives control, determinism and no personal data; and a small set of hand-crafted golden records for the specific boundary and negative cases that generators never produce.
>
> For masking, the techniques are substitution, shuffling, nulling, encryption, numeric or date variance, and format-preserving masking. The rule that matters most is that masking has to preserve referential integrity — if a customer identifier is masked in one table it has to be masked identically everywhere it is referenced, otherwise joins break and you spend your testing time debugging your own data.
>
> For automation specifically, my principles are: every test creates its own data in setup, every run uses a unique identifier so parallel runs cannot collide, there is no shared mutable fixture, data lives in code as factories rather than in spreadsheets, and if there is randomness the seed is logged so a failure can be reproduced.
>
> My own constraint is unusual and it forced this discipline on me: my end-to-end suites run against production. So I cannot reset a database or generate bulk data freely, and anything I create can appear in real reporting. My approach is that test data must be identifiable, isolated and reversible — dedicated test supplier accounts, a naming convention that distinguishes test records, non-destructive steps, and every run creating its own purchase order rather than relying on a pre-existing one. That last rule is important: if I depended on an existing record, someone else could modify it and my test would fail with no product defect behind it, which is exactly how a team learns to ignore its own test results.

> **Cross-question: "Is running automated tests on production a good idea?"**
>
> > It is not what I would choose if I had a production-like staging environment with realistic data. In my case it reflects a real constraint, and I have made it as safe as I can — isolated test entities, clearly identifiable naming, non-destructive actions, no bulk generation. And it does buy one genuine advantage: I am validating the real integrations and the real configuration, so I never get the failure mode where everything is green in staging and broken in production because of an environment difference. But I would still push for a production-like staging environment, and I would keep only a small, carefully chosen read-mostly smoke set running against production as a live health signal. The distinction I would make in an interview is between deliberately testing in production with guardrails and observability, which is a legitimate practice, and testing in production because there is nowhere else, which is a constraint to be improved.

> **Cross-question: "How do you handle test data for a flow that sends a real email?"**
>
> > I try to avoid asserting on the inbox at all, because email delivery introduces latency and a third-party dependency into a test that is not actually about email. My preference is to assert at the boundary I control — that the notification service was called with the right payload, or that the correct record was created with the correct link. If I genuinely must verify the email end to end, I use a dedicated test mailbox with an API, poll with a bounded wait rather than a fixed sleep, and keep exactly one such test rather than putting an inbox check into every flow. Verifying the link content matters more than verifying delivery, incidentally — in a recent feature the email did get delivered, it just quietly contained the wrong URL.

---

# 9. Test Execution

## 9.1 Execution process

```
Build deployed + release notes received
        │
        v
Verify build is testable  ──> SMOKE SUITE ──> FAIL ──> reject build, notify, stop
        │ PASS
        v
Execute by priority (P1 first, risk-based order)
        │
        ├──> PASS  ──> mark Pass, update RTM
        ├──> FAIL  ──> reproduce, isolate, log defect with evidence
        └──> BLOCKED ──> log blocker, note dependency, move to next case
        │
        v
Daily status report (executed / passed / failed / blocked / not-run + top risks)
        │
        v
Defect fixed ──> RETEST that case ──> plus impact-area REGRESSION
        │
        v
Exit criteria met? ──> No ──> next cycle
                    └─> Yes ──> Test Summary Report ──> sign-off
```

## 9.2 Test case statuses — aur unka exact matlab

| Status | Matlab | Kab lagta hai |
|---|---|---|
| **Pass** | Actual == expected, poora case | Sab steps aur assertions pass |
| **Fail** | Actual != expected | Product behaviour galat hai |
| **Blocked** | Execute hi nahi kar sake | Dependency missing, env down, prerequisite defect |
| **Not Run** | Time hi nahi mila | Cycle end pe bacha hua |
| **Skipped / N/A** | Consciously chhoda | Feature out of scope for this build |
| **In Progress** | Chal raha hai | Long E2E cases mein |

**Interview trap:** *"Fail aur Blocked mein farq?"* — **Fail** matlab test chala aur product ne galat behave kiya. **Blocked** matlab test chal hi nahi paaya. Ye distinction metrics ke liye critical hai: 40 Blocked cases ka matlab hai environment/dependency problem, 40 Failed ka matlab hai quality problem. Dono ka response bilkul alag hai.

## 9.3 Jab tum blocked ho — kya karo (ye poocha jaata hai)

**Galat jawab:** "Main developer ka wait karta hoon." Ye passive hai, senior nahi lagta.

**Sahi approach — 6 steps:**

1. **Blocker ko precisely document karo** — exact error, build number, environment, timestamp, steps, screenshot/log/HAR. Guessing nahi, evidence.
2. **Turant raise karo** — standup ka wait mat karo agar ye P1 blocker hai. Right person ko directly ping karo with the evidence.
3. **Impact quantify karo** — "iski wajah se 34 planned cases blocked hain, jisme 12 P1 hain" — ye ek casual complaint ko ek prioritisation input bana deta hai.
4. **Workaround dhoondho** — API level se bypass ho sakta hai? Direct DB seed? Feature flag? Different data path? Ek senior tester blocked hone ke baad ruk nahi jaata, doosra raasta dekhta hai.
5. **Parallel mein productive raho** — jo cases blocked nahi hain wo chalao; test cases likho; exploratory session lo unaffected areas pe; automation maintenance karo. **Idle time nahi hona chahiye.**
6. **Visibility banaye rakho** — daily status mein blocked count aur uska business impact dikhao, taaki ye "QA slow hai" na dikhe balki "hum ek dependency pe blocked hain" dikhe.

## 9.4 Daily status report ka format

```
Date: 2026-08-21 | Build: 4.12.0-rc3 | Cycle: P2P Regression, Day 3 of 5

Executed : 128 / 240 (53%)
Passed   : 111
Failed   : 9  (defects: MER-1204, MER-1207, MER-1211 ...)
Blocked  : 8  (all blocked by MER-1204 — supplier portal login fails for
               new supplier accounts)
Not run  : 112

Top risks:
  1. MER-1204 blocks the entire supplier acknowledgement path — 8 cases now,
     ~30 more downstream if not fixed by tomorrow.
  2. Invoice-matching cases not yet started; 2 days of buffer remaining.

Asks:
  - Need MER-1204 prioritised today to protect the invoice-matching window.
```

**Ye format kyun achha hai:** numbers hain, blockers ka business impact hai, aur **ek explicit ask** hai. Management ko report chahiye jo decision enable kare, sirf activity log nahi.

> **[REAL]** Execution ke dauraan mera sabse valuable habit ye ban gaya hai ki **automation failure ko blindly "flaky" label nahi karna**. Date-picker wala case perfect example hai: wo failure har run mein **30 second stall** karta tha aur phir fallback path mein chala jaata tha. Team ke liye ye "flaky test" tha. Actually wo **deterministic** tha — `.fill()` isliye stall karta tha kyunki Mantine v5 ka DatePicker input **readOnly** hai (component `allowFreeInput` pass hi nahi karta), aur fallback calendar path `.first` ki wajah se ek **disabled outside-month day** pakad leta tha. Agar maine "retry laga do" kar diya hota to root cause kabhi nahi milta aur 30 second har run mein waste hote rehte. Execution phase mein **failure ko investigate karna** utna hi important hai jitna test chalana.

> **Interview answer:**
> Execution starts with a gate, not with test cases. I verify the build is testable by running the smoke suite first, and if smoke fails I reject the build rather than spending the day generating noise. After that I execute in priority and risk order, highest-risk areas first, so that if the cycle gets cut short, what remains unexecuted is what mattered least.
>
> I distinguish carefully between failed and blocked, because they mean different things. Failed means the test ran and the product behaved incorrectly — that is a quality signal. Blocked means the test could not run at all — that is an environment or dependency signal. Forty failures and forty blocks demand completely different responses, and reporting them as one number hides that.
>
> When I am blocked, I do not wait. I document the blocker precisely with the exact error, build, environment and evidence; I raise it immediately to the right person rather than waiting for standup if it is a priority-one blocker; I quantify the impact — saying "this blocks thirty-four planned cases, twelve of them priority one" turns a complaint into a prioritisation input; I look for a workaround such as driving the flow at the API level or seeding the state directly; and I keep executing whatever is unaffected, doing exploratory sessions or automation maintenance, so there is no idle time.
>
> The habit I would most want to convey is that I investigate failures rather than labelling them. On my project a step was widely considered flaky because it stalled for thirty seconds and then took a fallback path. It was not flaky at all — the input was read-only by design in that component version, so the fill call could never succeed, and the fallback then matched an ambiguous element. Had I added a retry, the root cause would have stayed hidden and we would have paid thirty seconds every run forever. A retry that makes a test green without explaining the failure is not a fix, it is a way of losing information.

> **Cross-question: "Your automated suite has 30 failures this morning. What is your first move?"**
>
> > I triage before I investigate. First I look for a common signature — if all thirty failed at login or at the same first step, it is almost certainly one environmental or deployment cause, not thirty defects, and I would check whether the build actually deployed and whether the environment is healthy. Then I separate product failures from test failures. Then I check whether any failure is in a money-critical path, because that one gets investigated first regardless of count. What I would not do is start at the top of the list and work down, because thirty failures usually reduce to two or three causes, and finding those causes is much faster than debugging individually.

> **Cross-question: "How do you report status to a manager who wants a one-line answer?"**
>
> > I give a risk statement rather than a number. Something like: "We are on track for Friday except the supplier acknowledgement path, which is blocked by one defect; if it is not fixed today, invoice matching loses its buffer." A percentage figure sounds precise and tells them nothing actionable — fifty-three percent executed does not say whether we ship. What a manager needs is what could go wrong, by when, and what decision I need from them.

---

# 10. Defect Life Cycle

## 10.1 Kya hai

Defect life cycle = ek bug ke **states** aur unke beech **kaun transition karta hai**. Interview mein log states bata dete hain lekin **"kaun move karta hai"** bhool jaate hain — wahi maturity dikhata hai.

## 10.2 Full state diagram

```
                            ┌──────────┐
                            │   NEW    │  (QA logs the defect)
                            └────┬─────┘
                                 │ Dev lead / triage assigns
                                 v
                            ┌──────────┐
                  ┌─────────│ ASSIGNED │──────────┐
                  │         └────┬─────┘          │
                  │              │ Dev works      │ Dev/triage decides
                  │              v                │ it is not valid work
                  │         ┌──────────┐          │
                  │         │  OPEN /  │          │
                  │         │IN PROGRESS│         │
                  │         └────┬─────┘          │
                  │              │ Dev fixes      │
                  │              v                v
                  │         ┌──────────┐   ┌──────────────────────────┐
                  │         │  FIXED   │   │  REJECTED (not a bug)    │
                  │         │(Ready for│   │  DUPLICATE               │
                  │         │ retest)  │   │  DEFERRED / POSTPONED    │
                  │         └────┬─────┘   │  CANNOT REPRODUCE        │
                  │              │         └────────────┬─────────────┘
                  │              │ QA retests           │
                  │              v                      │ QA disagrees
                  │        ┌───────────┐                │ (adds evidence)
                  │        │ QA verify │                │
                  │        └─────┬─────┘                │
                  │      pass    │    fail              │
                  │        ┌─────┴──────┐               │
                  │        v            v               │
                  │  ┌──────────┐  ┌──────────┐         │
                  │  │  CLOSED  │  │ REOPENED │         │
                  │  └──────────┘  └────┬─────┘         │
                  │                     │               │
                  └─────────────────────┴───────────────┘
                     (back to ASSIGNED for rework)

  Special: CLOSED defect found again in a later release
           ──> REOPENED (same root cause) or NEW (different root cause)
```

## 10.3 Har state — detail table

| State | Matlab | **Kaun move karta hai** | QA ki responsibility |
|---|---|---|---|
| **New** | Defect abhi log hua, triage pending | QA logs it | Complete report: steps, actual, expected, env, build, evidence, severity, priority |
| **Assigned** | Owner assign ho gaya | Dev lead / triage committee | Ensure sahi team ko gaya hai |
| **Open / In Progress** | Dev actively kaam kar raha hai | Developer | Available rehna clarification ke liye |
| **Fixed / Resolved** | Dev bolta hai fix ho gaya, retest ready | Developer | Fix ka commit/build number note karo |
| **Retest / Ready for QA** | QA queue mein hai verification ke liye | QA picks up | Verify **on the build that contains the fix** |
| **Verified** | QA ne confirm kiya fix kaam karta hai | QA | Retest + impact regression |
| **Closed** | Defect band | QA (usually) | Regression case add karo agar zaroori |
| **Reopened** | Retest fail hua ya defect wapas aaya | QA | **Evidence dena mandatory** — kya alag hai |
| **Rejected / Not a Bug** | Dev/product bolte hain expected behaviour hai | Dev / Product | Requirement se verify karo; disagree karo to evidence ke saath |
| **Duplicate** | Already logged hai | Dev / triage | Original ID link karo, verify ki root cause sach mein same hai |
| **Deferred / Postponed** | Valid bug hai, lekin abhi fix nahi karenge | Product owner / release manager | Ensure ki decision documented hai + release notes / known issues mein hai |
| **Cannot Reproduce** | Dev ke paas reproduce nahi hua | Developer | Better repro: video, logs, exact data, env, frequency |

## 10.4 Nuances jo senior candidate se expect kiye jaate hain

### Reopen vs New — kab kya?
- **Reopen** — same root cause, fix kaam nahi kiya ya regress ho gaya. History ek jagah rehni chahiye.
- **New** — symptom similar hai lekin root cause alag hai. Reopen karoge to history confusing ho jaayegi aur metrics distort honge.

### Deferred ka sahi handling
Deferred ek **business decision** hai, QA decision nahi. QA ka kaam hai:
- Ensure karna ki **product owner** ne consciously decide kiya, dev ne convenience ke liye nahi.
- Deferred defect **known issues / release notes** mein jaaye.
- Ek **revisit date ya trigger condition** ho, warna "deferred" ka matlab "silently closed forever" ho jaata hai.

### Duplicate — verify before accepting
Bahut baar do defects same **symptom** dikhate hain lekin root cause alag hota hai. Agar tumne duplicate accept kar liya aur original fix ho gaya, tumhara wala bug **live chala jaayega**. Rule: duplicate tabhi accept karo jab root cause same ho, symptom same hone se nahi.

### Defect report ke mandatory fields
```
Title       : Short, specific, includes WHAT and WHERE
              BAD : "Sales page broken"
              GOOD: "Sale is created with null customer when customerId is
                     sent blank from the create-sale form"
Environment : Build/version, browser, OS, tenant, user role
Steps       : Numbered, minimal, from a clean state
Test data   : Exact values used
Actual      : What happened, with evidence
Expected    : What should have happened, with the requirement reference
Evidence    : Screenshot / video / console log / network HAR / API
              request-response / server log with correlation ID
Severity    : Technical impact
Priority    : Business urgency
Frequency   : Always / intermittent (x out of y attempts)
Workaround  : Yes/no, and what it is
```

> **[REAL]** Project Sales V1 ke defects ne mujhe sikhaya ki **defect report mein evidence hi sab kuch hai**. Ek defect tha jisme customer-facing offer endpoint `totalPrice: null` aur empty `lineItems` return kar raha tha, jabki contract ki real value thi. Agar main sirf itna likhta ki "offer page galat data dikha raha hai" to wo turant "cannot reproduce" ya "frontend issue" ban jaata. Isliye maine API response attach kiya — same contract jiski real value hai, uske customer-facing endpoint pe null total aur khaali line items. Ab ye argue karne layak nahi bacha.
>
> Isi tarah ek defect **security-flavoured** tha: accept endpoint duplicate contact email pe `400 "non unique result"` return karta tha **aur raw Mongo query leak kar deta tha — ek public unauthenticated endpoint pe**. Ye technically ek functional bug (duplicate handling) aur ek security bug (information disclosure) dono hai. Maine ise **do alag defects** ki tarah frame kiya, kyunki agar dev sirf duplicate-handling fix kar deta to error leak wala issue chhup jaata aur baaki endpoints pe bhi bacha rehta. **Ek ticket mein do root causes mat daalo.**

> **Interview answer:**
> The defect life cycle runs New, Assigned, Open or In Progress, Fixed, Retest, Verified, Closed — with the branches Rejected or not-a-bug, Duplicate, Deferred, Cannot Reproduce, and Reopened. What I think matters more than the state list is who is authorised to move each state, because that is what stops the process being abused.
>
> QA logs it as New. Triage or the dev lead assigns it. The developer moves it through In Progress to Fixed. Only QA moves it to Verified or Closed, and only QA reopens it — a developer closing their own defect removes the entire point of the loop. Deferred is not a QA or developer decision at all; it is a product owner decision, and my job there is to make sure it was consciously made rather than done for convenience, that it appears in the known-issues list, and that it has a revisit trigger, otherwise deferred quietly means closed forever.
>
> Two distinctions I am careful about. Reopen versus new: I reopen only when the root cause is the same, and I raise a new defect when the symptom looks similar but the cause is different, because collapsing them corrupts both the history and the metrics. And duplicate: I only accept a duplicate when the root cause is genuinely the same, not merely the symptom — otherwise the original gets fixed, my defect gets closed with it, and the actual bug ships.
>
> On report quality, the thing that decides whether a defect gets fixed or gets argued about is evidence. In a recent verification I raised a defect where a customer-facing endpoint returned a null total price and empty line items for a contract that had real values. Written as "the offer page shows wrong data" that becomes a cannot-reproduce within a day. Written with the API request and response attached against a named contract with known values, there was nothing to debate. I also split one finding into two defects deliberately — an endpoint returned a 400 on a duplicate contact email and leaked the raw database query in the error body on a public unauthenticated endpoint. Those are two root causes, a functional one and an information-disclosure one, and if I had filed them together the leak would have been hidden behind the functional fix and left in place on every other endpoint.

> **Cross-question: "Who should have the authority to close a defect?"**
>
> > QA, on the build that contains the fix. If developers can close their own defects, the verification step effectively disappears and you find out in production. The one legitimate exception is defects closed as duplicate or as won't-fix by product, but those are not really closures — those are decisions, and they should be visible as such rather than blended into the closed count.

> **Cross-question: "A defect is marked Cannot Reproduce. What do you do?"**
>
> > I treat it as my report being insufficient rather than as the developer being wrong, because that gets the fastest resolution. I go back and reproduce it myself first, noting the frequency — three out of ten attempts is a different investigation from always. Then I tighten everything that could differ: exact build, exact user and role, exact tenant, exact data, browser and version, and whether it depends on state left over from an earlier step. I attach a video, the console output, and the network trace with the correlation identifier so the developer can find the request in the server logs. Very often the missing variable is a precondition I did not realise I had — a specific role, a specific record state, or in one case on my project, the day of the month.

> **Cross-question: "How do you handle a defect that is reopened three times?"**
>
> > After the second reopen I stop treating it as a ticket problem and treat it as a communication problem. I sit with the developer and reproduce it in front of them, because three reopens almost always means we are not looking at the same thing — either my repro has an implicit precondition I never wrote down, or they are fixing a symptom while the cause is elsewhere. I also check whether the ticket is actually holding two different defects that keep swapping places. And I would raise it in the retrospective, because a defect that reopens three times has usually already cost more than the feature it belongs to.

---

# 11. Severity vs Priority

## 11.1 Kya hai

- **Severity** = defect ka **technical / functional impact** kitna bada hai. Ye **QA decide karta hai**. Objective-ish.
- **Priority** = ise **kitni jaldi fix karna hai**. Ye **business/product decide karta hai**. Subjective, business context pe depend karta hai.

**One-liner:** *Severity is "how badly it breaks." Priority is "how soon we fix it."* Ye **independent** dimensions hain — isliye chaaron combinations possible hain.

## 11.2 Severity levels

| Severity | Definition | Example |
|---|---|---|
| **S1 — Critical / Blocker** | System/major feature completely unusable, data loss/corruption, security breach, no workaround | App won't load; PO creation fails for all users; customer data exposed |
| **S2 — Major / High** | Major functionality broken but workaround exists, or wrong data in a core flow | Invoice total calculated wrong; supplier acknowledgement fails for one item type |
| **S3 — Minor / Medium** | Feature works but with noticeable deviation | Sorting incorrect on one column; validation message shown at the wrong moment |
| **S4 — Trivial / Low** | Cosmetic, no functional impact | Typo, misalignment, inconsistent spacing |

## 11.3 Priority levels

| Priority | Definition | Typical SLA |
|---|---|---|
| **P1 — Urgent** | Fix immediately, may need hotfix | Same day |
| **P2 — High** | Fix in this release | Before release |
| **P3 — Medium** | Fix in an upcoming release | Next sprint or two |
| **P4 — Low** | Fix when convenient / backlog | No commitment |

## 11.4 The 4-combination matrix — **ye guaranteed poocha jaata hai**

| Combination | Meaning | Real example |
|---|---|---|
| **High Severity + High Priority** | Core functionality broken and business is bleeding right now | Users cannot log in at all. Or: **a sale is created with no customer attached** — the record is functionally corrupt and it is happening on the main creation path. Fix now. |
| **High Severity + Low Priority** | Badly broken, but almost nobody hits it | App crashes on a legacy browser used by 0.1% of traffic. Or a data-corrupting bug in a report that only runs at financial year-end — severity is high, but there are eight months before anyone runs it. |
| **Low Severity + High Priority** | Technically trivial, business-critically visible | Company name misspelled on the homepage or on a customer-facing invoice PDF. Or a wrong currency symbol — the logic is fine, one character is wrong, but it goes to every customer. Fix before the demo. |
| **Low Severity + Low Priority** | Minor and can wait | Slight misalignment of a label on an internal admin settings page. Backlog. |

**Interview mein hamesha "High Sev + Low Priority" aur "Low Sev + High Priority" ke examples maange jaate hain** — kyunki wahi prove karta hai ki tumne concept samjha hai, ratta nahi maara.

## 11.5 Kaun decide karta hai — aur conflict kaise handle karein

- **Severity — QA.** Kyunki QA ne impact observe kiya hai. QA ko iska defend karna aana chahiye.
- **Priority — Product owner / business,** usually triage meeting mein, dev input ke saath.

**Agar product priority down-grade kar de aur tumhe lage galat hai:** argue mat karo severity pe. **Impact ko business language mein translate karo.** "S2 hai" mat bolo. Bolo: "Iss bug ki wajah se har naya sale record bina customer ke create ho raha hai — matlab jo bhi sales aaj create honge unhe manually reconcile karna padega, aur finance ke reports galat honge." Ab decision unka hai, lekin unke paas poori information hai.

> **[REAL]** Project Sales V1 ke 5 defects mein se **3 blockers** the, aur unko blocker call karne ki wajah severity thi, na ki mera irritation. Ek: frontend blank `customerId` bhejta tha aur backend usko null kar deta tha — **sale bina customer ke create ho jaata tha**. Ye High Severity + High Priority hai, kyunki ye data integrity ka issue hai main creation path pe, workaround nahi hai, aur galat data downstream reporting mein chala jaata hai.
>
> Doosra: tokenized offer link **404 deta tha kyunki page kisi bhi portal pe deployed hi nahi tha, production samet.** Ye bhi High/High — poora customer-facing flow dead hai. Lekin dhyan do, ye ek **deployment** issue hai code issue nahi — aur yahi wajah hai ki severity "kitna toota hai" pe base hoti hai, "fix kitna chhota hai" pe nahi. Fix ek deploy jitna chhota ho sakta hai, severity phir bhi critical rahegi.
>
> Aur error-leak wala defect (public unauthenticated endpoint raw Mongo query leak kar raha tha) — **functional severity medium lag sakti hai** kyunki user ek 400 dekhta hai, **lekin security severity high hai**, kyunki attacker ko internal query structure dikh raha hai bina login ke. Isliye maine ise security lens se frame kiya, functional lens se nahi. Jis lens se tum frame karte ho wahi decide karta hai ki bug fix hoga ya backlog mein sadega.

> **Interview answer:**
> Severity is the technical or functional impact of the defect — how badly it breaks the product — and it is assessed by QA. Priority is how soon the business wants it fixed, and it is decided by the product owner. They are independent dimensions, which is why all four combinations occur in practice.
>
> High severity with high priority is the obvious one: users cannot log in, or a record is created in a corrupt state on the main creation path. High severity with low priority is a badly broken behaviour that almost nobody encounters — for example a crash on a browser version with a fraction of a percent of traffic, or a data-corrupting bug in a year-end report that nobody will run for eight months. Low severity with high priority is the reverse: a spelling error in the company name on a customer-facing invoice, or a wrong currency symbol — trivially broken, but it goes in front of every customer, so it gets fixed before the release. Low severity with low priority is a misaligned label on an internal admin screen.
>
> When product downgrades a priority I disagree with, I do not argue about the severity label, because nobody outside QA cares about the label. I translate the impact into business terms: rather than saying it is a severity two, I say that every sale created today will have no customer attached, so finance will need to reconcile them manually and the revenue reports will be wrong. Then the decision is still theirs, but it is an informed one. And one more thing I am deliberate about — the framing lens changes the outcome. A defect where a public unauthenticated endpoint returns a database query in its error body looks like a medium-severity validation issue through a functional lens, and a high-severity information disclosure through a security lens. It is the same defect; only the second framing gets it fixed this week.

> **Cross-question: "Can severity change over time?"**
>
> > Severity should not change just because time passed or because someone got annoyed — it describes the impact, and the impact is a fact about the product. It legitimately changes when new information arrives: if I discover the defect also corrupts persisted data rather than merely displaying incorrectly, that is genuinely higher severity. Priority, by contrast, changes all the time, and it should — a defect in the reporting module jumps in priority the week before month-end close, without its severity moving at all.

> **Cross-question: "A cosmetic typo — would you ever block a release for it?"**
>
> > Yes, in specific cases. If the typo is in the company name, in a legal or compliance statement, in a currency or unit symbol, or on the checkout or invoice screen, then it is cosmetic in severity and severe in consequence. A wrong unit on a construction purchase order — tonnes instead of kilograms, for instance — is one character of code and a very expensive real-world outcome. So my rule is that I look at what the text influences, not at how much of the code is wrong.

> **Cross-question: "Who wins if QA and the developer disagree on severity?"**
>
> > Severity is my call as QA, and I should be able to defend it with the evidence — what breaks, whether a workaround exists, how many users or records are affected, whether data is corrupted. If the developer disagrees, usually one of us has information the other lacks, and the conversation resolves it. If it does not resolve, it goes to triage with both positions stated, and the product owner decides the priority, which is the number that actually determines what happens. I try not to spend energy defending a label when the outcome is decided by priority anyway.

---

# 12. Smoke vs Sanity vs Regression vs Retesting

## 12.1 Comparison table — yaad kar lo

| Aspect | **Smoke** | **Sanity** | **Regression** | **Retesting** |
|---|---|---|---|---|
| **Purpose** | Build testable hai ya nahi? | Specific fix/feature broadly kaam kar raha hai? | Naye change ne purana kuch toda to nahi? | Ye specific defect fix hua ya nahi? |
| **Scope** | Wide but shallow — critical paths only | Narrow but deeper — one area | Wide and deep — whole app or impacted area | Very narrow — the failed case only |
| **Depth** | Very shallow | Medium | Deep | Deep on one case |
| **When** | Immediately after every build/deploy | After a minor fix or a small build change | Before release, after significant changes, periodically | After a defect is marked Fixed |
| **Trigger** | New build | Small fix delivered | Any code change | Defect status = Fixed |
| **Scripted?** | Yes, fixed suite | Often unscripted/ad-hoc | Yes, maintained suite | Yes — the original failing case |
| **Automated?** | Almost always | Sometimes | Should be — highest ROI | Usually |
| **Duration** | 5–15 min | 15–60 min | Hours to days | Minutes |
| **If it fails** | **Reject the build**, stop testing | Send back, do not do deep testing | Raise defects, assess release risk | Reopen the defect |
| **Subset of** | Regression suite (a chosen slice) | — | — | — |

## 12.2 Deeper distinctions

### Smoke Testing
**Kya hai:** "Build Verification Test" bhi kehte hain. Ek chhoti, fixed suite jo verify karti hai ki application **launch hoti hai aur core paths chalte hain**. Ye quality prove nahi karti — ye prove karti hai ki **aage test karna worth hai**.

Naam kahan se aaya: hardware testing se — circuit ko power do, agar dhuan (smoke) nikla to aage test karne ka matlab hi nahi.

**Merlin-shaped smoke suite:**
```
1. App loads, login works for buyer role
2. Dashboard renders without console errors
3. PO list page loads and returns data
4. PO create form opens
5. Create a minimal PO and confirm it is persisted
6. Supplier portal login works
7. Notification/API health endpoint returns healthy
```
Agar in mein se koi bhi fail — build reject, koi regression mat chalao.

### Sanity Testing
**Kya hai:** Narrow aur deep. Ek specific area pe focused check ki fix ne kaam kiya aur us area mein kuch obviously toota nahi. Usually **unscripted**, aur usually **regression se pehle** hoti hai — agar sanity fail ho gayi to poori regression chalane ka time waste karne ka koi matlab nahi.

**Interview trap:** *"Smoke aur sanity mein difference?"*
> Smoke = **wide + shallow**, poore build pe, "kya build testable hai". Sanity = **narrow + deep**, ek area pe, "kya ye fix/feature theek se kaam kar raha hai". Smoke usually scripted aur automated hoti hai, sanity usually ad-hoc.

### Regression Testing
**Kya hai:** Verify karna ki **naya change ne existing working functionality nahi todi**. Ye QA ka sabse bada recurring cost hai — aur isiliye **automation ka sabse strong business case** yahi hai.

**Regression selection strategies:**

| Strategy | Kya hai | Kab |
|---|---|---|
| **Full regression** | Poori suite | Major release, big refactor, framework upgrade, before a long freeze |
| **Regional / impact-based** | Sirf impacted modules + unke immediate neighbours | Normal sprint release |
| **Risk-based** | Highest business-risk areas first | Time-boxed cycles |
| **Progressive** | Naye features ke cases regression suite mein add hote jaate hain | Continuous |
| **Corrective** | Suite same, koi change nahi (sirf config/data changed) | Env/config-only changes |

**Impact analysis kaise karein** (ye senior question hai): dev se poocho kaunsi files/modules change hue; shared components aur shared services trace karo; database schema changes dekho; API contract changes dekho; aur **jo defects pichle release mein iss area mein aaye the** unke cases zaroor chalao — defects cluster karte hain.

### Retesting (Confirmation Testing)
**Kya hai:** Sirf wo exact test case jo fail hua tha, **fix wale build pe**, **same data ke saath** dobara chalana.

**Retesting vs Regression — classic interview question:**

| | Retesting | Regression |
|---|---|---|
| Purpose | Confirm the specific fix works | Confirm nothing else broke |
| Scope | The failed case only | Related/whole area |
| Data | **Same data as the original failure** | Varied |
| Automation | Can be, but it's a targeted run | Should be automated |
| Can they run in parallel? | Retest first, then regression | Regression after retest passes |
| Always needed? | Yes, when a defect is fixed | Yes, when code changed |

**Golden rule:** Retest **hamesha regression se pehle**. Agar fix hi kaam nahi kiya to regression chalane ka koi point nahi.

> **[REAL]** Mere 6 end-to-end P2P suites practically **regression suite** hain — har release ke pehle ye confirm karti hain ki PO create → acknowledge → receive → invoice ka pura cycle intact hai across material, trade aur custom item types. Inka **smoke subset** bahut chhota hai: login + PO list load + PO create form open. Agar wo teen fail hote hain to poori 15-step suite chalane ka koi matlab nahi — 15 minutes waste honge aur 6 cascading failures milenge jo sab ek hi root cause ke hain.
>
> Aur date-picker wala episode isi principle ka reverse example hai: agar main har failure ko regression failure maan leta to main product bugs dhoondhta rehta, jabki wo **test-side** defect tha. Isliye main hamesha pehla sawaal ye poochta hoon — "kya ye failure product ka hai ya test ka?" Agar ye distinction clear nahi rakhoge, to regression suite pe team ka trust khatam ho jaata hai, aur ek regression suite jispe trust nahi hai wo hai hi nahi.

> **Interview answer:**
> Smoke testing checks whether the build is worth testing at all — it is wide but shallow, covering only the critical paths, it runs immediately after every deployment, it is scripted and automated, and if it fails I reject the build rather than continuing. Sanity testing is the opposite shape: narrow and deep, focused on one area after a specific fix, usually unscripted, and it runs before regression so that we do not spend hours on a full suite when the fix itself did not work.
>
> Regression testing verifies that a new change has not broken existing behaviour. It is the largest recurring cost in testing and therefore the strongest business case for automation. I do not always run it in full — I select. For a normal sprint release I do impact-based selection: which modules changed, what shared components and services they touch, whether the API contract or schema changed, and importantly, which areas produced defects in recent releases, because defects cluster. For a major refactor or a framework upgrade I run the full suite, because impact analysis is unreliable when the change is architectural.
>
> Retesting, or confirmation testing, is narrow — it is re-executing the exact failed case on the build containing the fix, using the same data that produced the failure. The order matters: retest first, then regression, because there is no point regressing a fix that did not work.
>
> On my project the six end-to-end purchase-to-pay suites are effectively the regression suite, and the smoke subset is deliberately tiny — login, list loads, create form opens. If those three fail there is no value in running fifteen-step flows, because I will get six cascading failures that all trace back to one cause.

> **Cross-question: "Is smoke testing a subset of regression testing?"**
>
> > In practice yes — the smoke suite is usually a curated slice of the regression suite, chosen to cover the critical paths in the shortest possible time. But they answer different questions. Regression asks whether anything broke; smoke asks whether the build is stable enough to be worth asking that. That is why the failure response differs: a regression failure produces a defect, while a smoke failure produces a rejected build.

> **Cross-question: "Your regression suite takes six hours and the team runs it once a week. How would you fix that?"**
>
> > I would attack it on three fronts. First, tiering — split it into a ten-minute smoke tier that runs on every commit, a forty-minute critical tier on every merge, and the full suite nightly. That gets fast feedback where it matters most without touching the total runtime. Second, parallelisation and data independence — six hours usually means tests are serialised because they share data, and making each test create its own data removes the constraint. Third, pruning — in a suite that old there are usually cases that have never failed and cases that duplicate each other, and I would retire them deliberately during closure. What I would not do is silently stop running it, which is what teams do by default when a suite gets slow.

> **Cross-question: "After a defect is fixed, is retesting enough?"**
>
> > No. Retesting only proves that specific case now passes. The fix itself is a code change, and every code change can break something else — so retesting confirms the fix, and impact-based regression around the changed area confirms the fix did not cost something. In my experience the regression after a fix is more valuable than people expect, because fixes are made under time pressure with less design thought than the original feature.

---

# 13. Levels of Testing

```
        ┌───────────────────────────────────────────────┐
        │  UAT / Acceptance  — does it solve the business │  Business users
        │                      problem?                   │
        ├───────────────────────────────────────────────┤
        │  System Testing    — does the WHOLE system meet │  QA
        │                      requirements, end to end?  │
        ├───────────────────────────────────────────────┤
        │  Integration       — do the modules talk to     │  QA / Dev
        │                      each other correctly?      │
        ├───────────────────────────────────────────────┤
        │  Unit Testing      — does this function behave? │  Dev
        └───────────────────────────────────────────────┘
```

## 13.1 Functional Testing

**Kya hai:** Verify karna ki system **kya karta hai** — requirement ke against behaviour check karna. Black-box. Internal implementation se koi matlab nahi, sirf input → output.

**Kya cover hota hai:** business rules, calculations, workflows, data validation, CRUD, permissions, error handling, integrations ke functional outcomes.

**Functional vs Non-functional:**

| Functional | Non-functional |
|---|---|
| **What** the system does | **How well** it does it |
| PO create hota hai | PO create 800 ms mein hota hai |
| Login works with correct credentials | Login handles 5000 concurrent users |
| Invoice total calculate hota hai | Invoice page WCAG AA accessible hai |
| Requirement-driven | Quality-attribute-driven |
| Types: unit, integration, system, UAT, smoke, sanity, regression | Types: performance, load, stress, security, usability, accessibility, compatibility, reliability, scalability, maintainability, portability |

## 13.2 Integration Testing — **sab approaches**

**Kya hai:** Individually working modules ko combine karke unke **interfaces** test karna. Bug yahan module ke andar nahi hota — **modules ke beech** hota hai. Data format mismatch, wrong assumptions, missing fields, timing, error propagation.

### Approaches

#### A) Big Bang Integration
Saare modules ek saath integrate karke test karo.

```
   [A] [B] [C] [D] [E]  ──all at once──> [ SYSTEM ] ──> test
```
- **Pros:** Simple, no stubs/drivers needed, fine for very small systems.
- **Cons:** **Fault isolation nahi hota.** Fail hua to pata hi nahi kahan. Late start — sab modules ready hone tak wait.
- **Kab:** Chhote systems, ya jab saare modules already ready hon.

#### B) Top-Down Integration
Upar se neeche. Pehle top-level control modules, phir neeche wale. Missing lower modules ke liye **stubs** (dummy called-modules) use hote hain.

```
            [ UI / Controller ]        <-- test first
                   │
            ┌──────┴──────┐
        [ STUB ]      [ STUB ]         <-- replaced by real modules progressively
```
- **Pros:** Major design flaws jaldi milte hain; early prototype/demo possible; UI-level issues jaldi dikhte hain.
- **Cons:** Lower-level (usually data/logic-heavy) modules late test hote hain; **stubs likhne padte hain**, aur stub-based testing real behaviour hide kar sakta hai.
- **Kab:** Jab architecture top-down clear ho aur UI/flow validation priority ho.

#### C) Bottom-Up Integration
Neeche se upar. Pehle lowest-level utility/data modules, phir upar. Missing upper modules ke liye **drivers** (dummy calling-modules) use hote hain.

```
        [ DRIVER ]                     <-- replaced by real modules progressively
             │
      ┌──────┴──────┐
   [ DAO ]      [ Utils ]              <-- test first
```
- **Pros:** Core logic aur data layer jaldi verify hoti hai; fault isolation better.
- **Cons:** UI/end-user perspective **bahut late** aata hai; koi working prototype late tak nahi milta; drivers likhne padte hain.
- **Kab:** Jab risk data/business-logic layer mein ho (jo aksar hota hai).

**Stub vs Driver — yaad rakhne ka trick:**
> **Stub** = **called** module ka fake (top-down mein use hota hai). **Driver** = **calling** module ka fake (bottom-up mein). *"S comes before D in the alphabet, Top comes before Bottom."* → Stub-Top, Driver-Bottom.

#### D) Sandwich / Hybrid Integration
Top-down aur bottom-up dono **ek saath**. Middle layer ko "target layer" maan kar upar se aur neeche se converge karte hain.

```
        [ UI ]  ─── top-down ───┐
                                 v
                        [ MIDDLE / SERVICE LAYER ]  <-- meet here
                                 ^
        [ DAO ]  ── bottom-up ──┘
```
- **Pros:** Fast, parallel effort, large systems ke liye practical.
- **Cons:** Stubs **aur** drivers dono chahiye — costly; middle layer khud thoda late test hoti hai.
- **Kab:** Bade layered systems, bade teams, tight timelines.

#### E) Incremental Integration (umbrella term)
Ek-ek module add karke test karte jaana. Top-down, bottom-up aur sandwich sab **incremental** ki hi flavours hain — inka opposite Big Bang hai.

**Interview clarification:** Incremental koi 5th separate technique nahi hai — ye **category** hai. Big Bang = non-incremental. Top-down / bottom-up / sandwich = incremental.

### Comparison

| Approach | Stubs? | Drivers? | Fault isolation | Early UI feedback | Early core-logic feedback | Best for |
|---|---|---|---|---|---|---|
| Big Bang | No | No | **Poor** | Late | Late | Very small systems |
| Top-Down | **Yes** | No | Good | **Early** | Late | UI-driven, design validation |
| Bottom-Up | No | **Yes** | **Good** | Late | **Early** | Logic/data-risk systems |
| Sandwich | Yes | Yes | Good | Early | Early | Large layered systems |

> **[REAL]** Integration testing ka sabse strong example mere paas Project Sales V1 se hai, aur ye interview mein bahut impact karta hai. Backend ke paas **57 integration tests the aur wo sab pass ho rahe the**, jabki flow **poori tarah toota hua tha** — sale bina customer ke create ho raha tha, offer draft mein atka hua tha, aur customer-facing page 404 de raha tha.
>
> Wajah simple hai: **har test apne inputs khud construct karta tha.** Test khud apna token mint karta tha, khud apna `customerId` pass karta tha. Matlab har test wo scenario verify kar raha tha jo test writer ne **assume** kiya tha, na ki wo scenario jo **frontend actually produce karta hai**. **Asli integration seam — frontend jo bhejta hai vs backend jo expect karta hai — kabhi exercise hi nahi hua.**
>
> Yahi wajah hai ki main "integration test" naam pe blindly trust nahi karta. Main poochta hoon: **ye test input kahan se laa raha hai?** Agar test khud apna input bana raha hai to wo integration test nahi hai, wo ek **badi unit test** hai. Real integration test wo hai jo asli producer ka output asli consumer ko feed kare — contract tests, ya E2E jo actual UI ya actual client se chale.

> **Interview answer:**
> Integration testing verifies the interfaces between modules that individually work. The defects live between components rather than inside them — mismatched data formats, differing assumptions about who populates a field, error propagation, and timing.
>
> There are four approaches. Big bang integrates everything at once; it needs no stubs or drivers but gives you almost no fault isolation, so it only suits very small systems. Top-down starts from the control or UI layer and uses stubs for the not-yet-built lower modules; it surfaces design and flow issues early but leaves the data and logic layers tested late. Bottom-up starts from the lowest utility and data modules and uses drivers to call them; it gives good fault isolation and early verification of core logic, but you get no end-user view until late. Sandwich or hybrid runs both directions simultaneously toward a middle target layer — it is what large layered systems actually use, at the cost of needing both stubs and drivers. Top-down, bottom-up and sandwich are all incremental approaches; big bang is the non-incremental one. And the memory hook for the two doubles: a stub replaces the module being called, a driver replaces the module doing the calling.
>
> The example I would give from my own work is about what integration testing is not. On a recent feature the backend had fifty-seven integration tests and every one of them passed, while the end-to-end flow was completely broken — records were being created without a customer reference, the offer stayed in draft with nothing to publish it, and the customer-facing page returned a 404. The reason all fifty-seven passed is that each test constructed its own inputs: it minted its own token and passed its own customer identifier. So each test verified the scenario its author imagined, not the scenario the frontend actually produces. The real integration seam — what one side sends versus what the other side expects — was never exercised.
>
> That taught me to ask one question of any test labelled integration: where does the input come from? If the test manufactures its own input, it is a large unit test, not an integration test. A genuine integration test feeds the real producer's output to the real consumer, which in practice means contract tests at the boundary or end-to-end tests driven through the actual client.

> **Cross-question: "If the unit and integration tests all pass, why do you still need end-to-end tests?"**
>
> > Because unit and integration tests verify the pieces against the assumptions written by the same people who built the pieces. End-to-end is the only level where the real producer feeds the real consumer through the real configuration and the real deployment. The example I lived through makes this concrete — fifty-seven green integration tests, and the customer-facing page was not deployed on any portal at all, including production. No amount of unit or integration coverage detects a page that does not exist, because those levels never make the request the customer would make. That is also the argument for keeping end-to-end tests few but real: they are the only ones that check the things nobody wrote an assumption about.

> **Cross-question: "So should we have written more integration tests?"**
>
> > More of the same kind would not have helped. What was needed was different in kind — a contract test at the seam, where the frontend's actual request shape is the input to the backend's validation, so that a blank customer identifier is exercised as the frontend genuinely sends it rather than as the test author imagined it. Plus a small number of true end-to-end tests through the real client. My rule of thumb is that a test's value comes from what it does not control. A test that controls all of its inputs can only confirm what you already believed.

## 13.3 System Testing

**Kya hai:** Poore, integrated system ko **end-to-end** test karna, **requirements ke against**, ek **production-like environment** mein. Black-box, QA-owned.

**Kya include hota hai:** end-to-end functional flows, plus non-functional aspects — performance, security, usability, compatibility, recovery, installation, configuration.

**Entry criteria:** integration testing complete; all modules integrated; environment production-like; test cases ready.
**Exit criteria:** all planned system test cases executed; no open S1/S2; requirements coverage complete via RTM.

**System testing vs Integration testing:**

| Integration | System |
|---|---|
| Interfaces between modules | The whole system's behaviour |
| Partial system | Complete, integrated system |
| Often needs stubs/drivers | No stubs — real system |
| Technical focus | Requirement/business focus |
| Dev + QA | QA |

## 13.4 UAT — User Acceptance Testing

**Kya hai:** **Business users / customers** verify karte hain ki system unka **real business problem** solve karta hai. Ye "kya code sahi hai" nahi poochta — ye poochta hai "**kya ye wo cheez hai jo humein chahiye thi**".

**Kaun karta hai:** actual end users, business analysts, product owners, client representatives. **QA nahi** — QA facilitate karta hai, execute nahi.

**QA ka role UAT mein:** UAT environment aur data ready karna, UAT scenarios/scripts prepare karna, users ko brief karna, session ke dauraan support dena, defects log karna aur triage karna, aur sign-off track karna.

**Entry criteria for UAT:**
- System testing complete, exit criteria met
- No open S1/S2 defects
- UAT environment ready with **realistic (production-like) data**
- UAT test scenarios prepared and agreed
- Business users identified, available and trained
- Release notes and known issues shared

**Exit criteria:** all UAT scenarios executed; no open critical business-blocking defects; formal **sign-off** from business.

### UAT ke types

| Type | Kaun | Kahan | Kya hota hai |
|---|---|---|---|
| **Alpha testing** | Internal staff/employees (not the dev team who built it) | **Developer's site / controlled environment** | Early real-usage feedback in a controlled setting; issues can be fixed fast |
| **Beta testing** | **Real external end users / customers** | **User's own environment**, real conditions | Real-world usage, real devices, real data, real network; feedback collected in the field |
| **Contract acceptance** | Client | Agreed environment | Verify contractual acceptance criteria are met |
| **Regulation acceptance** | Regulator / compliance body | As mandated | Verify legal/regulatory compliance |
| **Operational acceptance (OAT)** | Ops / SRE team | Production-like | Backup/restore, failover, monitoring, runbooks, disaster recovery, maintenance procedures |

**Alpha vs Beta — one-liner:** *Alpha = internal users, at our place, controlled. Beta = external users, at their place, real world.*

> **Interview answer:**
> The levels stack as unit, integration, system, and acceptance. Unit is owned by developers and verifies a single function or class in isolation. Integration verifies the interfaces between components. System testing takes the fully integrated system in a production-like environment and validates it end to end against the requirements, including non-functional aspects like performance, security, compatibility and recovery — that is QA-owned and black-box. Acceptance testing is business-owned and answers a different question entirely: not "is it built correctly" but "does it solve the problem we asked for."
>
> In UAT, the executors are actual business users, analysts and product owners — not QA. My role is to make UAT possible: prepare the environment with realistic data, write the scenarios in business language, brief the users, support them during sessions, and triage what they raise. The entry criteria I would insist on are that system testing has met its exit criteria, there are no open critical or high defects, the environment has production-like data, and the business users are identified and available — the last one is the one that actually derails UAT in practice, because the users have day jobs.
>
> Within acceptance, alpha testing is done by internal staff who did not build the product, in a controlled environment at our site, while beta testing puts it in front of real external users in their own environment with their own devices, data and network conditions. Beta finds the class of problems no internal environment ever produces — unusual devices, poor connectivity, and users doing things nobody on the team would think to do. There is also operational acceptance testing, which the operations team runs to verify backup and restore, failover, monitoring and runbooks — that one gets skipped constantly and is exactly what you wish you had done during the first real incident.

> **Cross-question: "If UAT finds a defect, whose failure is it?"**
>
> > It depends on the kind of defect, and the distinction matters more than the blame. If UAT finds a functional bug — something behaves incorrectly against a written requirement — that is a QA escape and it belongs in my defect leakage metric. If UAT finds that the requirement itself was wrong, that the workflow does not match how the business actually works, then that is exactly what UAT is for and it is a success, not a failure — though it does suggest we should have involved users earlier, at refinement. So my first question about any UAT defect is which of the two it is, because one means I need better coverage and the other means we need users in the room sooner.

---

# 14. Exploratory Testing (Session-Based)

## 14.1 Kya hai

Exploratory testing = **simultaneous learning, test design, and test execution**. Tum pehle se scripted cases follow nahi karte — tum product ko explore karte ho, aur jo seekhte ho uske base pe agla test decide karte ho.

**Ye ad-hoc testing NAHI hai.** Farq critical hai:

| Ad-hoc / Monkey testing | Exploratory testing |
|---|---|
| No structure, no plan | Structured by **charters**, time-boxed |
| Not documented | Documented in session notes |
| Not repeatable, not accountable | Reproducible findings, reportable coverage |
| No measurable output | Measurable: sessions, coverage areas, bugs, questions raised |
| Random | Directed by risk and by what you learn |

## 14.2 Session-Based Test Management (SBTM)

James Bach ka framework. 4 elements:

1. **Charter** — session ka mission. Ek short statement: kya explore karna hai, kaunse resources se, kaunsi information dhoondhni hai.
2. **Time box** — usually 60–120 minutes. Short session ~45 min, normal ~90 min, long ~120 min. **Uninterrupted.**
3. **Session notes** — kya test kiya, kya mila, kya questions uthe, kya blocked raha.
4. **Debrief** — session ke baad lead/peer ke saath 5–10 min review: kya cover hua, kya risk baaki hai, next charter kya hona chahiye.

**Session time ka breakdown (metrics ke liye):**
- **Test design & execution (T)** — actual testing
- **Bug investigation & reporting (B)** — jab bug mila, usko isolate aur report karna
- **Session setup (S)** — environment, data, tooling

Agar B + S 50% se zyada hai, to problem testing mein nahi hai — environment/data/product stability mein hai. **Ye ek bahut strong metric hai jo senior candidates use karte hain.**

## 14.3 Charter kaise likhein

**Template:**
> **Explore** (target: feature/area/component)
> **With** (resources: data, tools, personas, configurations)
> **To discover** (information: risks, behaviours, inconsistencies)

## 14.4 Full worked charter — Merlin-shaped

```
════════════════════════════════════════════════════════════════════
SESSION CHARTER
════════════════════════════════════════════════════════════════════
Charter ID   : ET-PO-DATE-01
Tester       : Ritik C.
Date         : 2026-08-21
Time box     : 90 minutes
Build        : 4.12.0-rc3

CHARTER
  Explore    : the delivery-date selection on the PO creation form
  With       : multiple system dates (start of month, end of month,
               a 31st, a leap-day, and a month where the 1st falls
               on a Sunday), keyboard-only input, paste input, and
               the browser locale set to a non-default value
  To discover: whether date selection is deterministic regardless of
               today's date, whether disabled dates can be selected
               by any route, and whether the value saved matches the
               value displayed

AREAS IN SCOPE
  - Calendar grid rendering and month padding
  - minDate / maxDate enforcement
  - Keyboard interaction and accessibility of the date field
  - Persistence: displayed value vs value stored vs value returned by API
  - Timezone handling on save and on reload

OUT OF SCOPE
  - Date formatting in exported PDF (separate charter)

════════════════════════════════════════════════════════════════════
SESSION NOTES
════════════════════════════════════════════════════════════════════
[00:00] Setup. Buyer login, PO create form open.                    (S)
[00:08] Opened the calendar. Grid is 6 rows x 7 columns = 42 cells.
        Cells from the previous and next month are rendered with the
        SAME class as in-month cells. Counted 11 day numbers that
        appear twice in the grid.                                   (T)
        --> RISK: any locator that matches on day text alone is
            ambiguous. Note for automation review.
[00:19] minDate is set to today, so past days render disabled.
        Confirmed a disabled day cannot be selected by mouse.        (T)
[00:26] Tried keyboard: tabbed to the field and typed a date
        directly. Nothing entered. Inspected the input --> it is
        readOnly.                                                    (T)
        --> QUESTION: is keyboard entry intended to be impossible?
            This is an accessibility concern, not only a usability one.
[00:41] Attempted to select an out-of-month day that is visually
        present but disabled --> no selection, no error, no feedback
        to the user.                                                 (T)
        --> RISK: silent no-op. A user does not learn why nothing
            happened.
[00:55] Changed system date to the 31st and repeated --> grid padding
        shifts, and the set of duplicated day numbers changes.       (T)
        --> CONFIRMS: behaviour varies with today's date.
[01:10] Saved a PO with a valid future date, reloaded, compared the
        displayed value with the API response.                       (T)
[01:22] Wrote up findings.                                           (B)

TIME BREAKDOWN
  Test design & execution (T) : 62 min  (69%)
  Bug investigation (B)       : 20 min  (22%)
  Setup (S)                   :  8 min  ( 9%)

════════════════════════════════════════════════════════════════════
FINDINGS
════════════════════════════════════════════════════════════════════
BUGS / ISSUES
  1. Date field is readOnly, so keyboard and paste entry are
     impossible. Accessibility impact for keyboard-only users.
  2. Clicking a disabled out-of-month day produces no feedback at all.

TEST-SIDE RISKS (not product defects)
  3. Calendar day text is not unique within the grid — 11 of 42 day
     numbers are duplicated by previous/next-month padding. Any
     automation matching on day text and taking the first match is
     non-deterministic with respect to today's date.

QUESTIONS FOR PRODUCT / DEV
  - Is read-only date entry an intentional design decision?
  - Should out-of-month days be rendered at all, given they are
    never selectable?

COVERAGE ASSESSMENT
  Calendar grid       : deep
  minDate enforcement : deep
  Timezone handling   : shallow — needs its own charter
  Locale variations   : not covered — ran out of time

NEXT CHARTER PROPOSED
  ET-PO-DATE-02: explore timezone and locale handling of the delivery
  date across save, reload, and API response.
════════════════════════════════════════════════════════════════════
```

## 14.5 Exploratory testing ke tours (heuristics)

Charter ke andar direction dene ke liye "tours" use hote hain:

| Tour | Kya karte ho |
|---|---|
| **Feature tour** | Har feature ko ek baar touch karo, seekhne ke liye |
| **Money tour** | Wo features jo revenue se directly juda hai — sabse zyada attention |
| **Landmark tour** | Key features ki list banao aur unke beech random order mein ghoomo |
| **Back-alley tour** | Sabse kam use hone wale features — waha coverage sabse kam hoti hai |
| **Data tour** | Data ko system mein follow karo — kahan enter hota hai, kahan dikhta hai, kahan store hota hai |
| **Configuration tour** | Settings badal-badal kar dekho |
| **Interruption tour** | Beech mein cancel, back, refresh, close, network drop |
| **Anti-social tour** | Wo karo jo koi normal user nahi karega — negative values, huge inputs, double clicks |

**Popular mnemonics:**
- **SFDIPOT** (San Francisco Depot) — Structure, Function, Data, Interfaces, Platform, Operations, Time. Product coverage ke liye.
- **CRUSSPIC STMPL** — quality criteria: Capability, Reliability, Usability, Scalability, Security, Performance, Installability, Compatibility / Supportability, Testability, Maintainability, Portability, Localisability.

> **[REAL]** Mera date-picker root cause **exactly aise hi ek exploratory session se** nikla. Automation "flaky" batayi ja rahi thi. Agar main sirf retry laga deta to root cause kabhi na milta. Maine ek focused session liya aur teen alag cheezein discover ki jo koi bhi scripted test case cover nahi karta: **(1)** input `readOnly` hai kyunki component `allowFreeInput` pass hi nahi karta, isliye `.fill()` kabhi succeed hi nahi kar sakta — wo 30 second timeout tak wait karta tha aur phir fallback path leta tha; **(2)** calendar ka 6x7 grid previous/next month ke days se padded hai jo **same CSS class** share karte hain, jiski wajah se 42 mein se 11 day numbers duplicate hain; aur **(3)** `minDate` past days ko disable karta hai, to `.first` aksar ek **disabled outside-month day** pick karta tha.
>
> Aur sabse important insight: **wo failure flaky nahi tha, deterministic tha** — wo purely aaj ke day-of-month pe depend karta tha. Ye exploratory testing ka classic value hai: scripted test tumhe batata hai ki tumne jo socha tha wo kaam kar raha hai ya nahi; exploratory tumhe batata hai ki tumne **kya sochne se hi miss kar diya**.

> **Interview answer:**
> Exploratory testing is simultaneous learning, test design and execution — I use what I discover to decide what to test next, rather than following a script written before I knew anything. It is not ad-hoc testing, and I would push back on anyone who conflates them, because exploratory testing is structured, time-boxed, documented and reportable.
>
> I run it as session-based test management. Each session has a charter stating what I am exploring, with what resources, and what information I am trying to discover. It is time-boxed, typically ninety uninterrupted minutes. I keep notes as I go, and I debrief afterwards on what was covered, what risk remains and what the next charter should be. I also split the session time into test execution, bug investigation, and setup — if setup and investigation exceed half the session, that itself is the finding: the problem is the environment or the product's stability, not my testing.
>
> To give direction inside a session I use tours — a money tour through revenue-critical paths, a data tour following a value from entry through display to storage, an interruption tour with cancel, back, refresh and network drops, and a back-alley tour through the least-used features where coverage is always thinnest.
>
> The most valuable session I have run was on a date picker that everyone had labelled flaky. In ninety minutes I found three things no scripted case would have covered: the input is read-only because the component never passes the option that allows free input, so the fill call could never succeed and simply waited out its timeout; the calendar renders a six-by-seven grid padded with previous and next month days that share the same CSS class, so eleven of forty-two day numbers are duplicated; and a minimum-date setting disables past days, so the first text match frequently landed on a disabled out-of-month cell. The conclusion was that the failure was not flaky at all — it was fully deterministic and varied only with today's date. That is exactly what exploratory testing is for: a scripted test tells you whether what you thought of works, and an exploratory session tells you what you never thought of.

> **Cross-question: "How do you report exploratory testing to management? It sounds unmeasurable."**
>
> > It is very measurable, just not in test-case counts. I report the number of sessions run, the areas each charter covered and at what depth, the issues and open questions raised, the coverage assessment per area — deep, shallow, or not covered — and the session time split between execution, investigation and setup. That gives a manager the same three things a scripted report gives: what was covered, what was found, and what risk remains. What it does not give is a percentage, and I would argue a percentage of scripted cases was never a coverage measure either — it measures how many things I thought of in advance, not how much of the product is actually exercised.

> **Cross-question: "How much of your testing should be exploratory versus scripted?"**
>
> > I would not fix a ratio, I would assign by purpose. Anything that must be repeated identically every release — regression, smoke, contract checks — should be scripted and preferably automated, because a human executing the same steps for the fortieth time is worse at it than a machine and is being wasted. Everything new, everything ambiguous, and everything where a defect just escaped is where exploratory time earns the most. A rough shape I have found workable is that new features get an exploratory pass first, and the cases worth keeping from that session get scripted afterwards — which is the right order, because you learn what is worth scripting by exploring, not before.

> **Cross-question: "Is exploratory testing only for manual testers? You are an SDET."**
>
> > No, and I would say it is more valuable for an SDET, not less. Automation can only assert on what someone already anticipated, so a fully automated suite is a permanent record of past thinking. Exploratory sessions are how I generate new assertions to add. In my case an exploratory session did more than find product bugs — it found a defect in my own automation strategy, the non-unique locator, and that produced a standing rule for the whole suite. Exploring the product also tells me which flows are stable enough to be worth automating, which saves me from automating something that is about to be redesigned.

---

# 15. Negative Testing

## 15.1 Kya hai

Negative testing = system ko **wo dena jo usko nahi milna chahiye**, aur verify karna ki wo **gracefully reject karta hai** — crash nahi karta, silently accept nahi karta, aur internal details leak nahi karta.

**Positive testing** proves karta hai ki jo kaam karna chahiye wo hota hai. **Negative testing** proves karta hai ki jo nahi hona chahiye wo **nahi hota** — aur production incidents usually doosri category se aate hain.

## 15.2 Negative test ideas — systematic categories

| Category | Ideas |
|---|---|
| **Empty / null** | Empty string, whitespace only, null, missing field entirely, empty array/object |
| **Type mismatch** | String where number expected, number where date expected, array where object expected, boolean as string |
| **Boundary violations** | Max+1, min−1, zero where positive required, negative amounts, extremely large numbers, floating point where integer expected |
| **Format violations** | Malformed email, invalid date `2026-02-30`, wrong phone format, invalid currency, wrong enum value |
| **Length** | 1 char, max+1 chars, 10,000 chars, very long single word (no spaces) |
| **Character sets** | Unicode, emoji, RTL text, combining characters, control characters, null bytes |
| **Injection** | `' OR 1=1 --`, `<script>alert(1)</script>`, `{{7*7}}`, `../../etc/passwd`, NoSQL operators like `{"$ne": null}`, XXE payloads |
| **Authorisation** | No token, expired token, malformed token, another tenant's ID, lower-privileged role, direct URL access to a restricted page |
| **State violations** | Acknowledge an already-acknowledged PO; cancel a completed PO; pay an already-paid invoice; use a consumed one-time token twice |
| **Concurrency** | Double-click submit, two users editing the same record, duplicate request with the same idempotency key |
| **Dependency failure** | Downstream service down, timeout, 500 from a dependency, slow response, malformed response from a third party |
| **Interruption** | Refresh mid-flow, back button, close tab, network drop mid-request, session expiry mid-form |
| **Duplicates** | Same email twice, same PO number, same file uploaded twice |

## 15.3 Achha negative test kya verify karta hai

Sirf "error aaya" dekhna **kaafi nahi**. Ek achha negative test 4 cheezein verify karta hai:

1. **Rejection hua** — invalid input accept nahi hua.
2. **Sahi error code/message aaya** — user-appropriate, actionable, aur **exact status code** (400 vs 401 vs 403 vs 404 vs 409 vs 422 — ye distinctions matter karte hain).
3. **State change nahi hua** — koi partial record create nahi hua, kuch aadha save nahi hua.
4. **Kuch leak nahi hua** — stack trace, SQL/Mongo query, internal path, framework version, ya database structure error response mein nahi honi chahiye.

> **[REAL]** Point 4 ka perfect real example mere paas hai. Project Sales V1 mein accept endpoint duplicate contact email pe **`400 "non unique result"`** return karta tha — aur uske saath **raw Mongo query bhi leak kar deta tha**, aur ye ek **public, unauthenticated endpoint** tha. Isme do alag failures hain: pehla ye ki duplicate contact email ka case handle hi nahi kiya gaya (functional), aur doosra ye ki error response internal query structure expose kar raha hai bina kisi authentication ke (security — information disclosure). Ek attacker ke liye ye free reconnaissance hai. Yahi wajah hai ki mera negative test sirf ye check nahi karta ki "error aaya" — wo check karta hai ki **error mein kya hai**.

> **Interview answer:**
> Negative testing means giving the system inputs and sequences it should refuse, and verifying that it refuses them cleanly. I work through fixed categories rather than improvising: empty and null values, type mismatches, boundary violations, format violations, extreme lengths, unusual character sets, injection payloads, authorisation violations such as another tenant's identifier or an expired token, invalid state transitions, concurrency cases like double submission, dependency failures and timeouts, interruptions like refresh or back mid-flow, and duplicates.
>
> The part I care about most is what counts as a pass. Seeing an error is not enough. A negative test should verify four things: that the input was actually rejected, that the response carries the correct status code and an actionable message — and the distinction between 400, 401, 403, 404, 409 and 422 does matter — that no partial state change occurred, and that nothing internal leaked in the error.
>
> That fourth check is not theoretical for me. On a recent feature, a public unauthenticated endpoint returned a 400 on a duplicate contact email and included the raw database query in the error body. That is two separate defects: a functional one, in that duplicate contacts were never handled, and an information-disclosure one, in that an unauthenticated caller learns the internal query structure. If my negative test had only asserted "an error is returned," it would have passed on that build. So my negative assertions check the shape and the content of the error, not merely its presence.

> **Cross-question: "How many negative tests is too many?"**
>
> > It stops paying off when you are testing the framework rather than your code. If validation is generated from a schema, testing thirty malformed variants of the same field is verifying the validation library, not the application, and one representative case per rule is enough. Where I invest heavily instead is anywhere the application wrote its own logic — custom business rules, state transitions, authorisation checks, and anything touching money or another tenant's data. So my split is roughly: one case per validation rule for schema-driven fields, and exhaustive coverage on hand-written rules and on authorisation.

---

# 16. Black-Box Test Design Techniques

**Kyun zaroori hain:** Exhaustive testing impossible hai. Ek 10-character password field ke liye possible inputs practically infinite hain. Ye techniques **systematic reduction** deti hain — minimum tests se maximum defect-detection. Aur interview mein ye **directly poochi jaati hain, worked example ke saath.**

| Technique | Kab use karo |
|---|---|
| Equivalence Partitioning | Input ranges/sets — kitne tests, kam karne ke liye |
| Boundary Value Analysis | Ranges ke edges — defects yahin cluster karte hain |
| Decision Table | Multiple conditions ka combination → different outcomes |
| State Transition | Entity ka lifecycle, allowed/invalid transitions |
| Pairwise / Orthogonal Array | Bahut saare configuration parameters |
| Error Guessing | Experience-based, techniques ke upar ek layer |
| Use Case Testing | End-to-end business flows |

---

## 16.1 Equivalence Partitioning (EP)

### Kya hai
Input domain ko aise **groups (partitions)** mein baanto ki ek partition ke **saare values ka behaviour same ho**. Phir har partition se **ek representative value** test karo. Logic: agar 25 kaam karta hai to 26 bhi kaam karega — dono same partition mein hain.

Partitions do tarah ke hote hain: **valid partitions** (accept hone chahiye) aur **invalid partitions** (reject hone chahiye).

### Worked example — Age field

**Requirement:** *"Age must be between 18 and 60 (both inclusive). Only whole numbers are allowed."*

**Partitions:**

| Partition | Range | Type | Representative value | Expected |
|---|---|---|---|---|
| P1 | age < 18 | **Invalid** | `10` | Rejected — "Age must be between 18 and 60" |
| P2 | 18 ≤ age ≤ 60 | **Valid** | `35` | Accepted |
| P3 | age > 60 | **Invalid** | `75` | Rejected |
| P4 | Non-numeric | **Invalid** | `abc` | Rejected — "Enter a valid number" |
| P5 | Decimal | **Invalid** | `25.5` | Rejected — "Whole numbers only" |
| P6 | Negative | **Invalid** | `-5` | Rejected |
| P7 | Empty | **Invalid** | `` (blank) | Rejected — "Age is required" |

**Tests: 7** — instead of testing every number from -infinity to +infinity.

### Ek zaroori nuance
**Ek test mein ek hi invalid partition test karo.** Agar tum `abc` aur blank ek saath doge to first validation error hi dikhega aur doosri validation **kabhi exercise hi nahi hogi** — ye "error masking" kehlata hai. Valid partitions ko combine karna theek hai, invalid ko nahi.

### Merlin-shaped EP example

**Requirement:** *"PO line quantity must be greater than 0 and not more than 10,000."*

| Partition | Values | Type | Rep. value |
|---|---|---|---|
| Quantity ≤ 0 | ..., -1, 0 | Invalid | `0` |
| 1 ≤ Quantity ≤ 10000 | 1..10000 | Valid | `500` |
| Quantity > 10000 | 10001, ... | Invalid | `50000` |
| Non-numeric | `abc`, `1e5` | Invalid | `abc` |
| Blank | `` | Invalid | `` |

---

## 16.2 Boundary Value Analysis (BVA)

### Kya hai
Defects **edges pe** hote hain, middle mein nahi — kyunki `>` vs `>=` ki galti, off-by-one error, aur loop condition sab boundary pe hi manifest karte hain. BVA EP ka natural companion hai: **EP batata hai kaunsi partitions hain, BVA batata hai unke edges kahan hain.**

### Do variants

**2-value BVA (standard):** har boundary pe **boundary value** aur **uske just outside wali value** test karo. Formula: boundary aur boundary±1.

**3-value BVA (robust):** har boundary pe **boundary−1, boundary, boundary+1** — teenon. Zyada thorough, zyada tests.

### Worked example — Age 18 to 60

**Boundaries:** lower = 18, upper = 60.

**2-value BVA:**

| Value | Position | Expected |
|---|---|---|
| `17` | Lower boundary − 1 | **Rejected** |
| `18` | Lower boundary | **Accepted** |
| `60` | Upper boundary | **Accepted** |
| `61` | Upper boundary + 1 | **Rejected** |

→ **4 tests**

**3-value BVA:**

| Value | Position | Expected |
|---|---|---|
| `17` | Lower − 1 | Rejected |
| `18` | Lower | Accepted |
| `19` | Lower + 1 | Accepted |
| `59` | Upper − 1 | Accepted |
| `60` | Upper | Accepted |
| `61` | Upper + 1 | Rejected |

→ **6 tests**

```
      REJECT          |            ACCEPT            |         REJECT
  ────────────────────┼──────────────────────────────┼────────────────────
                 17   18   19  ..............  59   60   61
                  ↑    ↑    ↑                   ↑    ↑    ↑
   2-value BVA:   ✓    ✓                             ✓    ✓
   3-value BVA:   ✓    ✓    ✓                   ✓    ✓    ✓
```

### EP + BVA combine karke — full test set for Age

| # | Value | Technique | Partition/Boundary | Expected |
|---|---|---|---|---|
| 1 | `10` | EP | Invalid (below) | Rejected |
| 2 | `35` | EP | Valid | Accepted |
| 3 | `75` | EP | Invalid (above) | Rejected |
| 4 | `17` | BVA | Lower − 1 | Rejected |
| 5 | `18` | BVA | Lower | Accepted |
| 6 | `60` | BVA | Upper | Accepted |
| 7 | `61` | BVA | Upper + 1 | Rejected |
| 8 | `abc` | EP | Invalid type | Rejected |
| 9 | `` | EP | Empty | Rejected |
| 10 | `-5` | EP | Negative | Rejected |
| 11 | `25.5` | EP | Decimal | Rejected |

**11 tests, complete coverage of a numeric field.** Ye exactly wo answer hai jo interviewer chahta hai.

### Non-obvious boundaries — senior-level thinking
Boundaries sirf numeric ranges mein nahi hote:

| Type | Boundaries |
|---|---|
| **String length** | 0, 1, max−1, max, max+1 |
| **Collections** | Empty list, 1 item, max items, max+1 items |
| **Dates** | Today, yesterday, tomorrow, **month end (28/29/30/31)**, year end, **leap day (29 Feb)**, DST change day |
| **Time** | 00:00:00, 23:59:59, midnight crossing, timezone boundary |
| **Files** | 0 bytes, 1 byte, exactly max size, max+1 byte |
| **Pagination** | Page 0, page 1, last page, last page + 1, exactly one full page, one item over a page |
| **Money** | 0.00, 0.01, smallest currency unit, rounding boundaries (0.005), maximum representable |
| **Concurrency** | 1 user, N users, N+1 users |

> **[REAL]** Date boundaries ka value mujhe first-hand pata chala. Merlin ka date-picker calendar ek **6x7 grid** render karta hai — matlab 42 cells — aur wo grid previous aur next month ke days se padded hota hai. Iska matlab hai ki grid ki **shape har month ke hisaab se badalti hai**, is baat pe depend karke ki month ka 1st tareekh kaunse weekday pe padta hai. Isliye "month ka pehla din", "month ka aakhri din", "31-day month", "28-day month", aur "leap day" sab **real boundaries** hain, theoretical nahi. Aur kyunki `minDate` past days ko disable karta hai, "aaj" bhi ek boundary hai — aaj se pehle ka din reject hona chahiye, aaj accept hona chahiye. Agar maine ye boundaries socha hi na hota to bug ka pattern kabhi samajh mein nahi aata, kyunki failure **today's day-of-month** ke hisaab se badalta tha.

> **Interview answer:**
> Equivalence partitioning and boundary value analysis go together. Partitioning divides the input domain into groups where every value should behave identically, so I test one representative per partition instead of every value. Boundary analysis then targets the edges of those partitions, because that is where off-by-one and greater-than versus greater-than-or-equal defects actually live.
>
> Take a field where age must be between 18 and 60 inclusive, whole numbers only. The partitions are: below 18, which is invalid; 18 through 60, which is valid; above 60, invalid; plus invalid partitions for non-numeric input, decimals, negatives and empty. That gives representatives like 10, 35, 75, "abc", 25.5, minus 5 and blank.
>
> For boundaries, two-value analysis tests each boundary and the value just outside it: 17, 18, 60 and 61 — four tests. Three-value or robust analysis tests one below, the boundary, and one above at each edge: 17, 18, 19, 59, 60 and 61 — six tests. Combining partitioning with two-value boundary analysis gives me around eleven tests that cover that field completely, versus an infinite input space.
>
> One rule I follow: I test only one invalid value per test case. If I submit a non-numeric value and leave a mandatory field blank in the same test, only the first validation fires and the second is never exercised — that is error masking, and it produces false confidence.
>
> The part I would add as a senior consideration is that boundaries are not only numeric. String length has boundaries at zero, one, maximum and maximum plus one. Collections have empty, one, and maximum. Dates have month-end, year-end, leap day and daylight-saving transitions. Pagination has page zero, exactly one full page, and one item over a page. Money has zero, the smallest unit, and rounding boundaries. In my own project the date boundaries turned out to be the real ones — the calendar renders a six-by-seven grid whose padding shifts depending on which weekday the first of the month falls on, so month length and start weekday are genuine boundaries, and a defect there presented differently depending on today's date.

> **Cross-question: "Is boundary analysis still relevant when validation is generated from a schema?"**
>
> > For the format checks, less so — if the minimum and maximum come from a declarative schema, I test one representative per rule rather than exhaustively, because beyond that I am testing the validation library. But boundary thinking stays relevant for everything the schema does not express: business rules that compare two fields, rounding behaviour on money, pagination edges, and date boundaries. Those are hand-written logic, and hand-written logic is where off-by-one lives.

> **Cross-question: "You have a field with min 1 and max 10000. Which boundaries do you actually test in a time-boxed cycle?"**
>
> > Two-value at both edges: 0, 1, 10000 and 10001. That is four tests and it catches essentially every off-by-one defect. I would add the three-value variant — 2 and 9999 — only if that field controls something expensive, for instance if the quantity drives a financial calculation or a bulk operation, because there the cost of a missed edge is high enough to justify the extra two tests.

---

## 16.3 Decision Table Testing

### Kya hai
Jab output **multiple conditions ke combination** pe depend karta hai, to decision table har possible combination ko systematically enumerate karta hai. Ye missing business rules dhoondhne ka **sabse achha tool** hai.

**Rules ki sankhya:** N binary conditions ke liye **2^N rules**. 3 conditions = **8 rules**.

### Structure
```
┌─────────────────┬────┬────┬────┬────┬────┬────┬────┬────┐
│                 │ R1 │ R2 │ R3 │ R4 │ R5 │ R6 │ R7 │ R8 │
├─────────────────┼────┴────┴────┴────┴────┴────┴────┴────┤
│ CONDITIONS      │                                        │
│  Condition 1    │  Y    Y    Y    Y    N    N    N    N  │
│  Condition 2    │  Y    Y    N    N    Y    Y    N    N  │
│  Condition 3    │  Y    N    Y    N    Y    N    Y    N  │
├─────────────────┼────────────────────────────────────────┤
│ ACTIONS         │                                        │
│  Action A       │  X                                     │
│  Action B       │       X    X    X                      │
└─────────────────┴────────────────────────────────────────┘
```

**Pattern likhne ka trick (8 rules ke liye):** Condition 1 → `Y Y Y Y N N N N` (4-4), Condition 2 → `Y Y N N Y Y N N` (2-2), Condition 3 → `Y N Y N Y N Y N` (1-1). Har condition ka block size aadha hota jaata hai.

### Full worked example — PO Approval

**Business rules:**
> *"A purchase order is auto-approved only if the amount is within the buyer's approval limit, the supplier is verified, and the budget for that project has sufficient balance. If the amount exceeds the limit but everything else is fine, it routes to a manager for approval. If the supplier is not verified, it is blocked regardless of anything else. If the budget is insufficient, it routes to finance."*

**Conditions:**
- **C1** — Amount within buyer's approval limit? (Y/N)
- **C2** — Supplier verified? (Y/N)
- **C3** — Budget has sufficient balance? (Y/N)

**Actions:**
- **A1** — Auto-approve PO
- **A2** — Route to manager for approval
- **A3** — Block PO with "supplier not verified"
- **A4** — Route to finance for budget review

**Full decision table — all 8 rules:**

| | **R1** | **R2** | **R3** | **R4** | **R5** | **R6** | **R7** | **R8** |
|---|---|---|---|---|---|---|---|---|
| **C1** Amount within limit | Y | Y | Y | Y | N | N | N | N |
| **C2** Supplier verified | Y | Y | N | N | Y | Y | N | N |
| **C3** Budget sufficient | Y | N | Y | N | Y | N | Y | N |
| | | | | | | | | |
| **A1** Auto-approve | **X** | | | | | | | |
| **A2** Route to manager | | | | | **X** | | | |
| **A3** Block — supplier unverified | | | **X** | **X** | | | **X** | **X** |
| **A4** Route to finance | | **X** | | | | **?** | | |

**Ab yahan asli value hai — jo sawaal ye table generate karta hai:**

| Rule | Combination | Question raised |
|---|---|---|
| **R4** | Within limit, unverified supplier, insufficient budget | Two problems at once. Does the user see **both** errors, or only the supplier one? What is the precedence? |
| **R6** | Over limit, verified, insufficient budget | Two routings needed — manager **and** finance. Is it **sequential** (finance then manager)? **Parallel**? Which one first? **Requirement does not say.** |
| **R7, R8** | Over limit **and** unverified | Does the block short-circuit, so the manager never even sees it? Or does it route and then get blocked? |
| **R3, R4, R7, R8** | Supplier unverified | Requirement says "blocked regardless" — confirm it truly overrides the limit check and the budget check. Is the error message the same in all four? |

> **Ye hi decision table ka asli purpose hai.** Ye sirf test cases generate nahi karta — ye **requirement ke gaps expose karta hai**. R6 ke liye requirement mein jawab hai hi nahi. Interview mein ye point banana **bahut strong** hai.

### Rule consolidation (optimisation)
Agar C2 = N hone pe hamesha A3 hota hai chahe C1 aur C3 kuch bhi ho, to R3, R4, R7, R8 ko ek rule mein collapse kiya ja sakta hai:

| | **R3–R8 (collapsed)** |
|---|---|
| C1 Amount within limit | **–** (don't care) |
| C2 Supplier verified | **N** |
| C3 Budget sufficient | **–** (don't care) |
| A3 Block | **X** |

→ 8 rules **5 tests** ban jaate hain. **Lekin:** collapse tabhi karo jab tumne **verify kar liya ho** ki behaviour sach mein independent hai. Pehle enumerate karo, phir collapse karo — ulta nahi. Warna tum wo combination skip kar doge jahan actually bug hai.

### Merlin-shaped mini example — Invoice matching

**Conditions:** C1 = Invoice quantity matches receipt quantity? C2 = Invoice rate matches PO rate? C3 = Invoice within tolerance window?

| | R1 | R2 | R3 | R4 | R5 | R6 | R7 | R8 |
|---|---|---|---|---|---|---|---|---|
| C1 Qty matches | Y | Y | Y | Y | N | N | N | N |
| C2 Rate matches | Y | Y | N | N | Y | Y | N | N |
| C3 Within tolerance | Y | N | Y | N | Y | N | Y | N |
| Auto-match & post | X | X | | | | | | |
| Hold — rate variance | | | X | X | | | | |
| Hold — qty variance | | | | | X | X | | |
| Hold — both variances | | | | | | | X | X |

**Question this raises:** agar quantity aur rate dono match karte hain, to C3 (tolerance) ka matter hi nahi karta — R1 aur R2 same action dete hain. To kya tolerance check sirf variance hone pe apply hota hai? Requirement clarify karni padegi.

> **Interview answer:**
> A decision table is what I use when the outcome depends on a combination of conditions rather than on a single input. With N binary conditions there are two to the power N rules, so three conditions produce eight, and enumerating all eight is what stops you from silently skipping a combination.
>
> Take purchase order approval, where a PO auto-approves only if the amount is within the buyer's limit, the supplier is verified, and the project budget has sufficient balance; over the limit routes to a manager; an unverified supplier blocks it regardless; and an insufficient budget routes to finance. The three conditions give eight rules, and I lay them out with the first condition alternating in blocks of four, the second in blocks of two, and the third alternating singly, so no combination can be missed.
>
> What makes this technique valuable is not the test cases it produces but the questions it forces. Rule six is over the limit, verified supplier, insufficient budget — that needs both a manager routing and a finance routing, and the requirement as written does not say whether they are sequential, parallel, or which comes first. Rule four has an unverified supplier and an insufficient budget together, and the requirement does not say whether the user sees both errors or only one. Rules seven and eight combine over-limit with unverified, and it is undefined whether the block short-circuits before the manager ever sees it. So an eight-row table turned one paragraph of business rules into four concrete questions for the product owner, and that is far more valuable than the eight tests.
>
> On optimisation: if a condition genuinely dominates — an unverified supplier always blocks regardless of the other two — then four rules collapse into one with don't-care markers, taking eight rules down to five tests. But I enumerate first and collapse afterwards, never the reverse, because collapsing early is how you assume independence that the code does not actually have.

> **Cross-question: "What if you have six conditions? Sixty-four rules is not practical."**
>
> > At that point I do three things. First, I check whether the conditions are truly independent — often two of them are really one business concept and can be merged. Second, I collapse using don't-care conditions where a dominant condition short-circuits the rest, which usually removes a large fraction of the rules. Third, if the combinations genuinely are independent, I switch to pairwise testing, which covers every pair of condition values in a much smaller set — typically somewhere around ten to fifteen cases for six binary conditions instead of sixty-four. Pairwise is justified because empirically most combinatorial defects are triggered by an interaction between two factors, not six. But I would still enumerate exhaustively for anything involving money or authorisation, because there the cost of the rare missed combination is not worth the saving.

> **Cross-question: "Have you actually used a decision table on your project?"**
>
> > The natural place for it in my product is invoice matching, where the outcome depends on whether the invoiced quantity matches the receipt, whether the rate matches the purchase order, and whether the variance sits inside a tolerance window. Laying that out as eight rules immediately raises the question of whether the tolerance check even applies when both quantity and rate match exactly, because two different rules then produce the same action — which usually means either a redundant condition or a rule nobody has specified. That is the kind of question I would take to a refinement session rather than guessing at in a test.

---

## 16.4 State Transition Testing

### Kya hai
Jab system ka behaviour uski **current state** pe depend karta ho, to state transition testing verify karti hai: valid transitions kaam karte hain, **invalid transitions reject hote hain**, aur har state se sahi actions available hain.

**4 components:** States, Events (transitions ke triggers), Transitions (state A → state B on event E), Actions (transition ke side effects).

**Ye technique tab lagti hai jab entity ka lifecycle ho** — order, PO, invoice, payment, ticket, subscription, user account, document approval.

### Worked example — Merlin PO state machine

**States:** DRAFT, SUBMITTED, ACKNOWLEDGED, PARTIALLY_RECEIVED, RECEIVED, INVOICED, CANCELLED

**State diagram:**

```
                          ┌─────────┐
                          │  DRAFT  │
                          └────┬────┘
                    submit     │            cancel
              ┌────────────────┤──────────────────────┐
              v                                       │
        ┌───────────┐                                 │
        │ SUBMITTED │────── cancel ───────────────────┤
        └─────┬─────┘                                 │
              │ supplier acknowledges                 │
              v                                       │
      ┌──────────────┐                                v
      │ ACKNOWLEDGED │───── cancel ──────────>  ┌───────────┐
      └───┬──────┬───┘                          │ CANCELLED │
          │      │                              └───────────┘
  partial │      │ full receipt                       ^
  receipt │      │                                    │
          v      v                                    │
 ┌──────────────────┐   full   ┌──────────┐           │
 │PARTIALLY_RECEIVED│─receipt─>│ RECEIVED │           │
 └──────────────────┘          └────┬─────┘           │
          │                         │                 │
          │ more partial            │ invoice raised  │
          └──(self loop)            v                 │
                              ┌──────────┐            │
                              │ INVOICED │────────────┘
                              └──────────┘         (NOT allowed —
                                                    see invalid list)
```

### State transition table — VALID transitions

| # | Current state | Event | Next state | Action / side effect |
|---|---|---|---|---|
| 1 | DRAFT | Submit | SUBMITTED | Notify supplier; PO number locked; buyer can no longer edit lines |
| 2 | DRAFT | Cancel | CANCELLED | No notification needed |
| 3 | SUBMITTED | Supplier acknowledges | ACKNOWLEDGED | Record acknowledgement timestamp; notify buyer |
| 4 | SUBMITTED | Cancel | CANCELLED | Notify supplier of cancellation |
| 5 | ACKNOWLEDGED | Receive full quantity | RECEIVED | Create goods receipt; update inventory |
| 6 | ACKNOWLEDGED | Receive partial quantity | PARTIALLY_RECEIVED | Create partial receipt; track remaining quantity |
| 7 | ACKNOWLEDGED | Cancel | CANCELLED | Notify supplier; requires cancellation reason |
| 8 | PARTIALLY_RECEIVED | Receive remaining quantity | RECEIVED | Close the receipt; remaining quantity = 0 |
| 9 | PARTIALLY_RECEIVED | Receive another partial | PARTIALLY_RECEIVED (self) | Accumulate received quantity |
| 10 | RECEIVED | Invoice raised | INVOICED | Match invoice against receipt and PO |

### **INVALID transitions — ye sabse important hai**

Yahi wo test cases hain jo log likhna bhool jaate hain, aur yahi wo bugs hain jo production mein data corrupt karte hain.

| # | Current state | Attempted event | Expected result |
|---|---|---|---|
| I1 | DRAFT | Supplier acknowledges | **Rejected** — supplier should not even be able to see a draft PO. Expect 403/404, not 400 (visibility matters) |
| I2 | DRAFT | Receive goods | **Rejected** — cannot receive against an unsubmitted PO |
| I3 | SUBMITTED | Receive goods | **Rejected** — supplier has not acknowledged yet. Business decision: is this truly blocked, or allowed? **Ask.** |
| I4 | ACKNOWLEDGED | Supplier acknowledges again | **Rejected / idempotent no-op** — must not create a second acknowledgement record or a second timestamp |
| I5 | ACKNOWLEDGED | Buyer edits line items | **Rejected** — PO is contractually agreed; any change must go through a revision flow |
| I6 | RECEIVED | Cancel | **Rejected** — goods have physically arrived; cancellation is no longer a valid state change |
| I7 | INVOICED | Cancel | **Rejected** — financial record exists |
| I8 | INVOICED | Receive more goods | **Rejected** — over-receipt after invoicing |
| I9 | CANCELLED | Any event (submit / acknowledge / receive / invoice) | **Rejected** — CANCELLED is a terminal state |
| I10 | RECEIVED | Receive again beyond ordered quantity | **Rejected** — over-receipt, unless over-receipt tolerance is an explicit business rule. **Ask.** |
| I11 | PARTIALLY_RECEIVED | Invoice raised | **Depends** — can you invoice a partial receipt? This is a real business question, not a technical one. **Ask.** |

### Coverage levels — ye interview mein poocha jaata hai

| Coverage | Kya cover hota hai | Effort |
|---|---|---|
| **0-switch (state coverage)** | Har state kam se kam ek baar visit ho | Lowest |
| **0-switch (transition coverage)** | Har **valid transition** kam se kam ek baar execute ho | Standard minimum |
| **1-switch** | Har **pair of consecutive transitions** cover ho (e.g. SUBMITTED→ACKNOWLEDGED→RECEIVED as one sequence) | Higher, catches sequence-dependent bugs |
| **N-switch** | N+1 consecutive transitions ke saare sequences | Exhaustive, usually impractical |
| **Invalid transition coverage** | Har state se har **invalid** event try karna | **Sabse zyada value, sabse zyada skip kiya jaata hai** |

**Practical rule:** valid transitions ka **100% transition coverage**, plus **invalid transitions** har terminal aur financially-significant state se.

> **[REAL]** Mere 6 end-to-end suites basically **state transition ke happy paths** hain — DRAFT → SUBMITTED → ACKNOWLEDGED → RECEIVED → INVOICED, teen alag item types ke liye. Ye 0-switch transition coverage deta hai valid transitions pe. Lekin asli senior-level gap **invalid transitions** mein hai — aur ye exactly wahi shape hai jo Project Sales V1 ke ek defect ki thi. Waha offer **DRAFT state mein mint hota tha aur usko publish karne wala koi step exist hi nahi karta tha.** Matlab state machine mein ek transition **missing** thi. Downstream mein link-builder null return karta tha, email chupchaap ek legacy tokenless URL pe fall back kar jaata tha, aur accept gate usko reject kar deta tha.
>
> Ye bug ek state transition diagram banane se **turant** dikh jaata — kyunki tum dekh lete ki DRAFT se koi outgoing edge hi nahi hai. Yahi wajah hai ki main ab kisi bhi lifecycle-wale entity ke liye pehle state machine draw karta hoon: **missing transition ek bug hai, aur diagram usko visible bana deta hai.**

> **Interview answer:**
> State transition testing applies whenever behaviour depends on the entity's current state — orders, invoices, tickets, subscriptions, approvals. I model four things: the states, the events that trigger movement, the transitions themselves, and the actions or side effects each transition produces.
>
> For a purchase order in my product the states are draft, submitted, acknowledged, partially received, received, invoiced and cancelled. The valid transitions include submit from draft, supplier acknowledgement from submitted, full or partial receipt from acknowledged, a self-loop on partially received as further quantities arrive, and invoicing from received — each with its side effects, such as recording an acknowledgement timestamp and notifying the buyer.
>
> The transitions I care most about are the invalid ones, because those are the tests people skip and they are where data corruption comes from. A supplier should not be able to acknowledge a draft — and I would expect a 403 or 404 rather than a 400, because a supplier should not even learn the purchase order exists. Acknowledging an already-acknowledged order must be rejected or idempotent, never creating a second acknowledgement record. A buyer must not edit line items after acknowledgement, because that is a contractually agreed document. A received or invoiced order must not be cancellable. And cancelled is terminal, so every event from it must be rejected.
>
> On coverage levels: zero-switch coverage means every valid transition is exercised at least once, which I treat as the minimum. One-switch coverage exercises every pair of consecutive transitions, which catches sequence-dependent defects. In practice I aim for full valid-transition coverage plus deliberate invalid-transition coverage from every terminal and financially significant state.
>
> I want to make one point about missing transitions, because I hit exactly that. On a recent feature, an offer was created in draft state and there was no step anywhere that published it — the state machine simply had no outgoing edge from draft. The consequences appeared far downstream: the link builder returned null, the email silently fell back to a legacy URL without a token, and the acceptance gate rejected that URL. Nobody could see the cause from the symptom. Drawing the state diagram would have exposed it in a minute, because a state with no way out is visibly wrong on a diagram and invisible in prose. So a missing transition is a defect, and the diagram is what makes it visible.

> **Cross-question: "How do you test that a transition is truly idempotent?"**
>
> > I fire the same event twice and check three things rather than one: that the state is still correct, that no duplicate record was created — no second acknowledgement row, no second timestamp, no second notification sent — and that the second response is sensible, either the same success or an explicit conflict. Then I do the harder version, firing both requests concurrently rather than sequentially, because sequential duplicates are usually handled by a state check while concurrent ones need a database constraint or a lock. The double-click on a submit button is the everyday version of this, and it is a very common source of duplicate records in production.

> **Cross-question: "Your suites cover the happy path transitions. How would you extend them?"**
>
> > Two directions. First, invalid transitions from the financially significant states — cancel after receipt, invoice after cancellation, receive after invoicing — because those are the ones that corrupt financial records rather than merely annoying a user, and they are cheap to test at API level without driving the UI. Second, the sequence dimension: one-switch coverage over the pairs, particularly around partial receipt, since the self-loop where quantities accumulate is exactly the kind of place where a running total goes wrong on the third iteration but not the first. I would drive both at API level rather than through the UI, because state machine testing does not need a browser and running it through a browser makes it slow enough that people stop running it.

## 16.5 Other techniques (brief but know them)

### Pairwise / All-Pairs Testing
**Kya hai:** Jab bahut saare independent parameters hon (browser x OS x role x language x currency), to saare combinations test karna impossible hai. Pairwise ensure karta hai ki **har pair of values** kam se kam ek test mein saath aaye.

**Example:** 4 browsers x 3 OS x 3 roles x 2 currencies = 72 combinations. Pairwise se ~12 test cases mein saare pairs cover ho jaate hain.

**Justification:** empirically zyadatar combinatorial defects **do factors ke interaction** se aate hain, chhe ke nahi.

### Error Guessing
**Kya hai:** Experience-based. Tester apne past defects, known weak spots, aur intuition se test design karta hai. Ye formal technique nahi hai, **techniques ke upar ek layer** hai.

**Common error guesses:** empty submit, double-click, browser back after submit, session expiry mid-form, copy-paste with trailing whitespace, very long name, apostrophe in a name (`O'Brien` — classic SQL breaker), zero, negative, leading zeros, dates in the past, refresh on a payment page.

### Use Case / Scenario Testing
**Kya hai:** Real business workflows ko end-to-end test karna, actor ke perspective se, main flow + alternate flows + exception flows ke saath. Mere P2P suites basically use case tests hain.

---

## 16.6 Technique selection — kaunsi technique kab

| Situation | Technique |
|---|---|
| Numeric or range input | EP + BVA |
| Dropdown / enum / set of options | EP (each option is its own partition) |
| Multiple conditions deciding an outcome | Decision table |
| Entity with a lifecycle / statuses | State transition |
| Many independent config parameters | Pairwise |
| End-to-end business workflow | Use case / scenario testing |
| Anything, as a final sweep | Error guessing |

> **Interview answer:**
> I do not pick a technique by preference, I pick it by the shape of the input. A numeric or range field gets equivalence partitioning to reduce the input space and boundary analysis on the edges. A field with a fixed set of options gets partitioning where each option is its own partition. When the outcome depends on a combination of conditions, I build a decision table, because that is what makes missing rules visible. When the entity has a lifecycle, I build a state transition model and then deliberately test the invalid transitions. When there are many independent configuration parameters, I use pairwise to keep the count practical. Use case testing covers the end-to-end business flows, and error guessing is the last sweep on top of all of it, driven by the defects this team has produced before.
>
> The reason I want to be explicit about selection is that most of the value in these techniques is not the tests they produce — it is that they are systematic, so my coverage does not depend on how alert I happen to be that morning.

---

# 17. Risk-Based Testing

## 17.1 Kya hai

Risk-based testing = testing effort ko **risk ke hisaab se allocate karna**, uniformly nahi. Har feature ko equal attention dena sabse common junior mistake hai — kyunki har feature ka business impact equal nahi hota.

**Risk = Probability (kitna likely hai fail hona) x Impact (fail hua to kitna nuksaan)**

## 17.2 Probability kaise assess karein — evidence, guesswork nahi

| Probability driver | Kyun |
|---|---|
| **Code churn** — kitna change hua hai | Naya/badla hua code sabse zyada defect-prone |
| **Complexity** | Zyada branches, zyada conditions = zyada defects |
| **Defect history** — pichle release mein yahan kitne bugs aaye | **Defects cluster karte hain.** Ye sabse strong predictor hai |
| **Number of integrations** | Har seam ek failure point |
| **Team familiarity** | Naya developer / naya tech = zyada risk |
| **Test coverage abhi kitna hai** | Uncovered area = unknown risk |
| **Time pressure on the change** | Hotfix aur last-minute change sabse risky |

## 17.3 Impact kaise assess karein

| Impact driver | Kyun |
|---|---|
| **Money** | Direct financial loss ya wrong financial record |
| **Data integrity** | Corrupt data ko baad mein theek karna sabse mehnga hai |
| **Number of users affected** | Core flow vs niche feature |
| **Legal / compliance / security** | Regulatory exposure, breach |
| **Reputation / customer trust** | Customer-facing visible failure |
| **Workaround availability** | Workaround hai to impact kam |
| **Recoverability** | Undo kar sakte hain ya nahi |

## 17.4 Worked matrix — Merlin P2P

**Scale:** Probability 1 (very unlikely) – 5 (very likely). Impact 1 (negligible) – 5 (catastrophic). **Risk score = P x I** (max 25).

| # | Area / risk | P | I | Score | Band | Test approach |
|---|---|---|---|---|---|---|
| 1 | Invoice total calculated incorrectly (qty x rate, tax, rounding) | 3 | 5 | **15** | High | Full boundary + decision table on matching rules; automated regression every build; rounding cases explicit |
| 2 | Sale/PO created with a missing mandatory reference (e.g. no customer) | 4 | 5 | **20** | **Critical** | Contract test at the FE-BE seam; API-level negative tests for blank/absent fields; assert persisted state, not UI toast |
| 3 | Customer-facing offer link broken or not deployed | 3 | 5 | **15** | High | Post-deploy smoke that actually requests the customer-facing URL on every portal including production |
| 4 | Public unauthenticated endpoint leaking internal details in errors | 3 | 4 | **12** | High | Negative tests asserting error body shape; add to security checklist for every public endpoint |
| 5 | Supplier acknowledgement double-submit creating duplicate records | 3 | 4 | **12** | High | Idempotency test — sequential and concurrent |
| 6 | Cross-tenant data exposure (Tenant A sees Tenant B's PO) | 2 | 5 | **10** | High | Authorisation tests on every read endpoint with another tenant's ID |
| 7 | PO list page slow at tenant scale (50k+ POs) | 3 | 3 | **9** | Medium | JMeter scenario at realistic data volume; measure p95 |
| 8 | Date picker selecting the wrong date | 4 | 3 | **12** | High | Deterministic locator scoped to current month + enabled; date-boundary cases (month end, leap day, today) |
| 9 | Notification email not delivered to supplier | 3 | 3 | **9** | Medium | Assert the notification call and payload rather than the inbox; one end-to-end mailbox check |
| 10 | Partial receipt accumulating quantity incorrectly | 2 | 4 | **8** | Medium | State transition self-loop test with three successive partial receipts |
| 11 | Sorting incorrect on a list column | 3 | 1 | **3** | Low | Covered in regression, not prioritised |
| 12 | Label misalignment on internal admin settings | 2 | 1 | **2** | Low | Exploratory sweep only |

**Visual matrix:**

```
        IMPACT →
        1        2        3        4        5
     ┌────────┬────────┬────────┬────────┬────────┐
   5 │        │        │        │        │        │
     ├────────┼────────┼────────┼────────┼────────┤
   4 │        │        │  #8    │  #5    │  #2    │  <- CRITICAL zone
P    ├────────┼────────┼────────┼────────┼────────┤
R  3 │  #11   │        │  #7 #9 │  #4    │ #1 #3  │
O    ├────────┼────────┼────────┼────────┼────────┤
B  2 │  #12   │        │        │  #10   │  #6    │
     ├────────┼────────┼────────┼────────┼────────┤
   1 │        │        │        │        │        │
     └────────┴────────┴────────┴────────┴────────┘

  Score 15-25 : Critical/High — deep testing, automate, run every build
  Score  8-14 : Medium        — standard coverage, run each release
  Score  1-7  : Low           — light coverage, exploratory sweep, accept risk
```

## 17.5 Risk response — 4 options

| Response | Matlab | Example |
|---|---|---|
| **Mitigate** | Testing badhao, controls add karo | Automate the invoice calculation regression |
| **Transfer** | Kisi aur ko de do | Payment settlement is vendor-owned; rely on their contract tests |
| **Avoid** | Feature/approach hi change kar do | Do not launch the risky flow this release |
| **Accept** | Consciously chhod do, documented | IE11 support not tested — 0% traffic, signed off by product |

**Accept karna galat nahi hai** — **undocumented accept karna galat hai.** Yahi "Features not to be tested" section ka poora point hai.

> **[REAL]** Mera highest-scoring risk **"record created with a missing mandatory reference"** hai (P=4, I=5, score 20), aur main ye isliye jaanta hoon kyunki **ye actually hua**. Frontend blank `customerId` bhej raha tha ye assume karke ki backend derive kar lega, backend usko null kar de raha tha, aur sale bina customer ke create ho jaata tha. Probability high hai kyunki ye ek **unwritten contract** pe depend karta hai, aur impact maximum hai kyunki ye **data integrity** ka issue hai jo silently downstream reporting corrupt karta hai — user ko koi error hi nahi dikhta.
>
> Iska risk response **mitigate** tha, aur specifically FE-BE seam pe: API-level negative tests jo blank aur absent field dono bhejte hain, aur **persisted state assert karte hain, UI toast nahi**. Ye distinction important hai — UI success dikha rahi thi jabki record galat ban raha tha.
>
> Doosra example risk assessment ke discipline ka: date-picker ka risk score maine 12 rakha (P=4, I=3). Probability high thi kyunki failure **har us din hota tha jab day-of-month grid mein duplicate ho** — matlab bahut frequent. Impact medium tha kyunki ye **test-side** issue tha, product user ko nahi rokta tha — lekin ye har run mein 30 second waste karta tha aur suite ka trust khatam kar raha tha. **Automation ki reliability bhi ek risk item hai**, sirf product nahi.

> **Interview answer:**
> Risk-based testing means allocating effort proportionally to risk rather than spreading it uniformly, and I score risk as probability multiplied by impact on a one-to-five scale each.
>
> For probability I use evidence rather than intuition: how much the code changed, how complex it is, how many integration seams it crosses, how familiar the team is with it, what its current test coverage is, and above all its defect history — defects cluster, so the area that produced bugs last release is the best predictor of where they will appear next. For impact I look at money, data integrity, number of users affected, legal or security exposure, whether a workaround exists, and whether the damage is recoverable.
>
> To make it concrete on my product: the highest item on my matrix is a record being created with a missing mandatory reference, probability four and impact five, giving twenty. Probability is high because it depends on an unwritten contract between frontend and backend, and impact is maximum because it is a silent data integrity failure — the user sees success while the record is wrong, so it is discovered later in reporting when it is expensive to correct. That gets a contract test at the seam plus API-level negative tests that assert the persisted state rather than the UI response. At the other end, a misaligned label on an internal admin page scores two, and it gets an exploratory sweep and nothing more.
>
> The other thing risk-based testing gives me is a defensible answer to "why did you not test that." There are four legitimate responses to a risk — mitigate, transfer, avoid, and accept — and accepting a risk is a perfectly valid engineering decision. What is not valid is accepting it silently. So anything I decide not to test goes in writing into the out-of-scope section with the reason and the person who accepted it.

> **Cross-question: "How do you convince a product owner to accept a risk?"**
>
> > I do not present it as a risk, I present it as a trade with numbers attached. Something like: "Testing this area properly costs two days. Based on our defect history it has produced one defect in the last four releases, and the worst realistic outcome is a display error on an internal screen. Those two days would otherwise go into invoice matching, where a defect corrupts financial records. I propose we skip the first and note it as accepted." That is a decision they can make in thirty seconds, and it is theirs to make. What does not work is asking for permission to skip something — that sounds like cutting corners. Framing it as reallocating a fixed budget to the higher-impact area is the same decision and it lands completely differently.

> **Cross-question: "How do you handle a risk that has high impact but very low probability?"**
>
> > Low probability does not mean low priority when the impact is catastrophic and irreversible. Cross-tenant data exposure is my example — I would rate it low probability because the authorisation layer is centralised and well-tested, but the impact is a data breach, which is unrecoverable and has legal consequences. So the score alone under-represents it. My rule is that for anything irreversible — data corruption, security exposure, financial posting — I test it regardless of the probability score, because probability estimates are the least reliable half of the calculation and being wrong about a reversible bug costs a fix while being wrong about an irreversible one costs the company.

---

# 18. Test Estimation

## 18.1 Kya hai

Estimation = kitna testing effort lagega, ye predict karna. Interview mein ye isliye poocha jaata hai ki dekhein tumne kabhi **commitment** diya hai ya sirf assigned kaam kiya hai.

**Golden rule:** Estimate hamesha **range** ya **confidence ke saath** do, ek single number nahi. "5 days" bolna trap hai. "4 to 6 days assuming the environment is stable and the API contract is final" — ye senior answer hai.

## 18.2 Techniques

### A) Work Breakdown Structure (WBS)
Kaam ko chhote tasks mein todo, har task estimate karo, sum karo, buffer add karo. **Sabse reliable, sabse time-consuming.**

```
P2P Regression cycle
├── Test design
│   ├── Review requirements + raise questions        : 0.5 d
│   ├── Write scenarios                              : 1.0 d
│   └── Write/update test cases                      : 2.0 d
├── Automation
│   ├── Update existing 6 suites for new fields      : 2.0 d
│   └── New suite for the bid flow                   : 3.0 d
├── Execution
│   ├── Smoke per build (7 builds x 15 min)          : 0.25 d
│   ├── Full regression run + triage                 : 2.0 d
│   └── Retest + impact regression on fixes          : 1.5 d
├── Exploratory
│   └── 4 charters x 90 min                          : 1.0 d
└── Reporting
    └── Daily status + summary report                : 0.5 d
                                              ─────────────
                                     Subtotal   : 13.75 d
                                     Buffer 20% :  2.75 d
                                     TOTAL      : ~16.5 d
```

### B) Three-Point Estimation (PERT)
Har task ke liye teen numbers: **O** (optimistic), **M** (most likely), **P** (pessimistic).

```
Expected (E)          = (O + 4M + P) / 6
Standard deviation SD = (P - O) / 6
```

**Worked example — "Automate the bid flow suite":**
- O = 2 days (everything cooperates, locators are clean)
- M = 3 days (normal — a couple of unstable selectors)
- P = 7 days (the flow needs test data setup nobody has built, environment is flaky)

```
E  = (2 + 4(3) + 7) / 6 = (2 + 12 + 7) / 6 = 21 / 6 = 3.5 days
SD = (7 - 2) / 6 = 0.83 days

So: 3.5 days ± 0.83  →  ~68% confidence in 2.7 – 4.3 days
                     →  ~95% confidence in 1.8 – 5.2 days (E ± 2SD)
```

**Ye interview mein bolne wala answer hai** — kyunki ye dikhata hai ki tum uncertainty ko quantify kar sakte ho.

### C) Test Case Point / Function Point based
Test cases ko complexity ke hisaab se classify karo, historical average time se multiply karo.

| Complexity | Definition | Avg design + execute time | Count | Total |
|---|---|---|---|---|
| Simple | 1–5 steps, single screen, no setup | 20 min | 60 | 20 h |
| Medium | 6–12 steps, multiple screens, some data setup | 45 min | 40 | 30 h |
| Complex | 13+ steps, multi-role, cross-module, heavy setup | 120 min | 15 | 30 h |
| | | | **Total** | **80 h ≈ 10 d** |

Isme buffer add karo → ~12 days.

### D) Percentage of Development Effort
Industry heuristic: testing effort usually development effort ka **25–50%** hota hai.

Agar dev effort 40 days hai to testing ~10–20 days. **Ye sirf sanity check ke liye achha hai**, primary estimate ke liye nahi — kyunki ye tumhare context ko ignore karta hai.

### E) Analogy / Historical Based
"Pichhli baar similar feature (material PO flow) ko automate karne mein 4 days lage. Trade flow similar complexity ka hai, plus ek extra approval step. So ~5 days."

**Sabse fast aur surprisingly accurate — agar tumhare paas historical data ho.** Isliye actual-vs-estimated track karna zaroori hai.

### F) Expert Judgement / Delphi / Planning Poker
Multiple experts independently estimate karte hain, phir discuss karte hain outliers pe, phir converge. Agile teams planning poker isi ka simplified version hai. **Independent estimate pehle** — warna pehla bolne wala sabko anchor kar deta hai.

## 18.3 Estimate mein kya include karna log bhool jaate hain

Ye list bolna interview mein **bahut strong** signal hai:

| Often forgotten | Typical cost |
|---|---|
| Test **environment setup** and access | Half a day to days |
| **Test data** creation and refresh | Often larger than execution itself |
| **Retesting** after fixes | ~1 defect fix cycle per 10 defects found |
| **Regression** after each fix | Often 2–3 rounds, not 1 |
| **Defect reporting** and triage meetings | 20–30 min per defect, properly done |
| **Automation maintenance** (not new writing) | 20–30% of automation time, ongoing |
| **Meetings**, standups, refinement, demos | ~10–15% of everyone's week |
| **Learning curve** on a new module | Real and often ignored |
| **Build rejections** — bad builds cost a cycle | Historically 1 in 5 builds |
| **Buffer for unknowns** | 15–25% |

## 18.4 Estimate kaise present karein

```
Estimate: Bid flow automation + regression cycle

  Most likely : 16 working days
  Range       : 14 – 20 working days

  Assumptions:
    - API contract is final; no field changes mid-cycle
    - Test supplier accounts available on day 1
    - Environment available at least 6 hrs/day
    - Defect fixes delivered within 1 working day

  Risks to the estimate:
    - If the bid flow needs new test data infrastructure, add 2–3 days
    - Each rejected build costs approximately half a day

  Excluded:
    - Performance testing of the bid listing endpoint (separate estimate)
```

**Assumptions likhna sabse important hai.** Agar assumption toot gaya to estimate automatically renegotiate hota hai, aur tum "slow" nahi lagte.

> **[REAL]** Mere estimates mein do cheezein aisi hain jo textbook estimation mein nahi hoti. Pehli — **production par run karne ka overhead**. Har suite ko data-safe hona chahiye, matlab har run apna PO khud create karta hai aur cleanup socha jaata hai; ye pure design time hai jo ek staging-based suite mein nahi lagta. Doosri — **automation maintenance**. Date-picker wala fix ek "test likhne" ka kaam nahi tha, wo root-cause investigation ka kaam tha, aur wo ek pura session le gaya. Agar main sirf naye test likhne ka time estimate karta aur maintenance ka nahi, to main har sprint mein under-estimate karta. Isliye main automation effort ka **20–30% maintenance ke liye explicitly** rakhta hoon.

> **Interview answer:**
> I use different techniques depending on how much I know. When I have a clear scope, I use a work breakdown structure — decompose into design, automation, execution, exploratory and reporting tasks, estimate each, then add a buffer. When there is real uncertainty, I use three-point estimation: optimistic, most likely and pessimistic, with the expected value as optimistic plus four times most likely plus pessimistic, all divided by six, and the standard deviation as pessimistic minus optimistic over six. For example, automating a new flow at two, three and seven days gives an expected three and a half days with a standard deviation of about point eight, so I would quote roughly three to four and a half days rather than a single number. When I have history, analogy-based estimation is fastest and surprisingly accurate — which is why I track estimated versus actual, because that history is what makes the next estimate credible.
>
> What I think separates a senior estimate is what gets included. People estimate execution and forget everything around it: environment setup, test data creation which is frequently larger than execution, retesting after fixes, the two or three rounds of regression rather than one, time spent writing and triaging defects, automation maintenance rather than automation writing, meetings, and the cost of rejected builds. In my own case I explicitly reserve twenty to thirty percent of automation effort for maintenance, because I have learned that fixing a single misbehaving locator can consume an entire session once you actually root-cause it instead of retrying it.
>
> And I always present an estimate as a range with stated assumptions — that the API contract is final, that test accounts exist on day one, that fixes come back within a day. If an assumption breaks, the estimate renegotiates itself automatically, and the conversation is about the assumption rather than about whether I am slow.

> **Cross-question: "Your manager says the estimate is too high and asks you to do it in half the time. What do you say?"**
>
> > I would not argue about the number, because that becomes a contest of wills. I would present what the halved timeline buys and what it drops. Something like: "In eight days I can cover the critical purchase order and invoice paths with full automation and one exploratory pass. What comes out is the bid flow regression and the boundary work on invoice matching, which is where our defect history is worst. If you are comfortable with that trade, I will start today." Now it is a scope decision rather than an effort dispute, they own the trade-off, and it is documented. What I would never do is silently agree and then quietly cut coverage, because that is how you end up personally owning an escaped defect that was actually a schedule decision.

> **Cross-question: "How accurate have your estimates been?"**
>
> > Honestly, my early ones under-estimated consistently, and the reason was always the same category: I estimated the testing and not the surrounding work — environment issues, data setup, and retest cycles. What corrected it was tracking actuals against estimates and noticing the pattern was systematic rather than random, which meant it was fixable. Now I estimate the surrounding work explicitly as line items rather than hoping the buffer absorbs it. I would rather say that plainly than claim I am always accurate, because anyone who has actually delivered under an estimate knows nobody is.

---

# 19. Requirement Traceability Matrix (RTM)

## 19.1 Kya hai

RTM ek **mapping document** hai jo requirements ko test cases (aur defects) se link karta hai. Ye do sawaalon ka jawab deta hai:

1. **Forward traceability:** Har requirement ka test hai? → **Coverage gaps** dikhata hai.
2. **Backward traceability:** Har test kisi requirement se linked hai? → **Scope creep / orphan tests** dikhata hai.

Dono milkar **bi-directional traceability** banti hai.

## 19.2 Kyun matter karta hai

- **Coverage proof** — "kya sab kuch test hua?" ka objective jawab.
- **Impact analysis** — requirement badla to kaunse test cases update karne padenge, ye instantly pata chalta hai.
- **Audit/compliance** — regulated domains mein mandatory.
- **Gap detection** — jo requirement kisi test se linked nahi hai wo **untested** hai, chahe tumhe kitna bhi confident lage.
- **Orphan detection** — jo test kisi requirement se linked nahi, ya to requirement missing hai ya test useless hai. Dono worth knowing.

## 19.3 Worked RTM — Merlin P2P

| Req ID | Requirement | Priority | Test Scenario | Test Case IDs | Automated? | Status | Defect IDs |
|---|---|---|---|---|---|---|---|
| REQ-PO-01 | Buyer can create a PO with material items | High | SC-01 | TC-01.1, TC-01.2, TC-01.3 | Yes | Pass | — |
| REQ-PO-02 | Buyer can create a PO with trade items | High | SC-02 | TC-02.1, TC-02.2 | Yes | Pass | — |
| REQ-PO-03 | Buyer can create a PO with custom items | High | SC-03 | TC-03.1, TC-03.2 | Yes | Pass | — |
| REQ-PO-04 | Mandatory fields must be validated before submit | High | SC-04 | TC-04.1 … TC-04.6 | Partial | Pass | — |
| REQ-PO-05 | Delivery date cannot be in the past | Medium | SC-04 | TC-PO-DATE-001…003 | Yes | Pass | MER-1188 (closed) |
| REQ-PO-06 | Supplier is notified when a PO is submitted | High | SC-05 | TC-05.1, TC-05.2 | Yes | Pass | — |
| REQ-PO-07 | Supplier can acknowledge a submitted PO | High | SC-06 | TC-06.1 … TC-06.7 | Yes | **Fail** | MER-1204 |
| REQ-PO-08 | Supplier cannot acknowledge twice | High | SC-07 | TC-07.1, TC-07.2 | No | **Not run** | — |
| REQ-PO-09 | Buyer cannot edit a PO after acknowledgement | High | SC-09 | TC-09.1 | No | **Not run** | — |
| REQ-PO-10 | Goods receipt supports full and partial quantities | High | SC-10 | TC-10.1 … TC-10.4 | Yes | Pass | — |
| REQ-PO-11 | Invoice is matched against receipt and PO | Critical | SC-11 | TC-11.1 … TC-11.8 | Partial | Pass | MER-1211 (open) |
| REQ-PO-12 | A user cannot view another tenant's PO | Critical | SC-12 | **— none —** | No | **NO COVERAGE** | — |

**Ye table padho aur dekho ki ye kya turant bata deta hai:**
- **REQ-PO-12 ka koi test hi nahi hai** — aur ye Critical priority hai. Ye RTM ki sabse badi value hai: **gap immediately visible ho jaata hai.** Bina RTM ke ye gap kisi ko dikhta hi nahi, kyunki "jo nahi hai" wo dashboards mein nahi dikhta.
- REQ-PO-08 aur REQ-PO-09 **not run** hain — ye invalid state transitions hain, aur exactly wahi cheez hai jo log skip karte hain.
- REQ-PO-11 Critical hai aur sirf partially automated hai — ye ek automation priority hai.

## 19.4 RTM kaise maintain karein practically

Alag spreadsheet maintain karna **fail ho jaata hai** — wo stale ho jaati hai. Practical approach:

- Test management tool (Jira + Xray/Zephyr, TestRail, Linear) mein **requirement se test case ko link** karo — RTM automatically generate hoti hai.
- Automated test ke naam ya docstring mein **requirement ID** rakho, taaki report se traceability derive ho sake:
  ```python
  def test_supplier_cannot_acknowledge_twice():
      """REQ-PO-08: acknowledging an already-acknowledged PO must not
      create a second acknowledgement record."""
  ```
- Defect ko requirement se bhi link karo, sirf test case se nahi — taaki defect density per requirement nikale.

> **[REAL]** Agar Project Sales V1 ke liye RTM maintain hoti to teen defects **pehle hi visible ho jaate**. Requirement level pe ye likha tha ki "customer ko offer email jaayega aur wo accept kar sakega." Ek proper RTM mein us requirement ke against ye test cases hone chahiye the: **offer publish hota hai**, **link build hota hai**, **link ka page actually load hota hai**, aur **accept endpoint work karta hai**. Actual mein backend ke paas **57 integration tests** the, sab pass — lekin unme se ek bhi test **customer-facing URL ko actually request nahi karta tha**. RTM ye immediately dikha deta: requirement "customer can open the offer link" ke saamne column **khaali** hota. Aur wo khaali cell hi wo defect hai jisme page **kisi bhi portal pe deployed hi nahi tha, production samet.**
>
> Yahi wajah hai ki main RTM ko paperwork nahi maanta. **RTM ka asli output "jo test hua" nahi hai — "jo test nahi hua" hai.**

> **Interview answer:**
> A requirement traceability matrix maps requirements to test scenarios, test cases and defects, and it gives you bi-directional traceability. Forward traceability answers whether every requirement has at least one test, which exposes coverage gaps. Backward traceability answers whether every test maps to a requirement, which exposes orphan tests and scope creep.
>
> Its real output is not the list of what was tested — that information is already in the execution report. Its real output is the empty cells, because an untested requirement is invisible everywhere else. A dashboard showing ninety-eight percent pass tells you nothing about the requirement that has no test at all; the RTM is the only artifact where that shows up as a blank row.
>
> To make that concrete, on a recent feature the requirement was that a customer receives an offer link by email and can accept it. The backend had fifty-seven passing integration tests. Not one of them actually requested the customer-facing URL. In an RTM, the row for "customer can open the offer link" would have had an empty test column, and that empty cell was precisely the defect — the page was not deployed on any portal, including production. Fifty-seven green tests could not surface that, because they were all testing things someone had thought of.
>
> Practically, I do not maintain the RTM as a separate spreadsheet, because separate spreadsheets go stale within two sprints. I link test cases to requirements inside the tracking tool so the matrix is generated, and I put the requirement identifier in the automated test's name or docstring so traceability can be derived from the test report itself.

> **Cross-question: "Is an RTM not overhead in Agile?"**
>
> > The document is overhead; the question it answers is not. In Agile the same function is served by linking tests to stories in the tracker and by acceptance criteria on each story, which gives you generated traceability for free. What I would resist is dropping the question itself — "which requirements have no test at all" is worth asking every release regardless of methodology, and if the tool cannot answer it in one click then something is wrong with how the tests are linked, not with the concept.

> **Cross-question: "What do you do when you find a requirement with no test coverage?"**
>
> > It depends on what the requirement is worth. If it is critical — cross-tenant access control, for instance — I write the test immediately and treat the gap as a defect in my own process. If it is low priority, I record it in the out-of-scope list with a reason rather than quietly leaving the cell empty, because an empty cell with no explanation looks identical to an oversight. And I ask why it happened, since a gap in a critical requirement is usually not a one-off — it is normally a whole category of thing I am not looking at, such as authorisation or deployment verification.

---

# 20. Test Metrics

## 20.1 Kya hai

Metrics = testing ko measurable banana, taaki decisions data pe hon, feelings pe nahi. Lekin **galat metric worse than no metric hai** — kyunki log usko game karte hain.

## 20.2 Core formulas — yaad karo

### Defect Density
```
Defect Density = Total defects found / Size of module
                 (size = KLOC, function points, or story points)
```
**Kis kaam ka:** Batata hai kaunsa module **relatively** buggy hai — targeting ke liye.

**Worked example:**
```
PO module        : 45 defects / 12 KLOC = 3.75 defects per KLOC
Invoice module   : 30 defects /  4 KLOC = 7.50 defects per KLOC
Reporting module : 20 defects / 10 KLOC = 2.00 defects per KLOC
```
Absolute count mein PO module sabse bura dikhta hai (45 defects). Lekin **density** batati hai ki **Invoice module double concentration** rakhta hai — matlab wahan deeper testing aur code review chahiye. **Ye exactly wo insight hai jo raw count chhupa deta hai.**

### Defect Leakage (Defect Escape Rate)
```
Defect Leakage = (Defects found in production / Total defects found) x 100
                  where Total = defects found in testing + defects found in production
```
**Kis kaam ka:** QA process ki effectiveness ka **sabse honest measure**. Ye tumhare kaam ko judge karta hai, developer ke kaam ko nahi.

**Worked example:**
```
Defects found during testing    : 180
Defects found in production     :  20
Total                           : 200

Leakage = (20 / 200) x 100 = 10%
```
Matlab 10 mein se 1 defect nikal gaya. **Trend matter karta hai, absolute number nahi** — agar ye 10% se 6% ja raha hai to process improve ho raha hai.

### Defect Removal Efficiency (DRE)
```
DRE = (Defects found before release / Total defects) x 100
    = (Defects found in testing / (Defects in testing + Defects in production)) x 100
```
**Ye leakage ka complement hai:** `DRE = 100 − Leakage`

**Worked example (same numbers):**
```
DRE = (180 / 200) x 100 = 90%
```
**Industry benchmark:** ~85% average, 95%+ good, 99% excellent (aur usually bahut mehnga).

### Test Case Effectiveness
```
Test Case Effectiveness = (Defects detected by test cases / Total defects detected) x 100
```
**Kis kaam ka:** Batata hai ki tumhare **designed test cases** kitne defects pakad rahe hain vs exploratory/production/other sources.

**Worked example:**
```
Defects found by scripted test cases : 130
Defects found by exploratory         :  40
Defects found by devs/other/prod     :  30
Total                                : 200

Test Case Effectiveness = (130 / 200) x 100 = 65%
```
Agar ye number bahut kam hai to tumhare test cases weak hain. Agar bahut zyada hai to shayad tum exploratory kam kar rahe ho.

### Test Coverage
```
Requirement Coverage = (Requirements with at least one test / Total requirements) x 100
Test Execution Coverage = (Test cases executed / Test cases planned) x 100
Code Coverage = (Lines/branches executed by tests / Total lines/branches) x 100
```
**Warning:** Code coverage **quality nahi** measure karta. 100% code coverage ke saath bhi tum har assertion galat likh sakte ho. Ye batata hai ki code **execute** hua, ye nahi ki behaviour **verify** hua.

### Defect Rejection / Invalid Defect Ratio
```
Defect Rejection Ratio = (Defects rejected / Total defects raised) x 100
```
**Kis kaam ka:** Tumhare defect reports ki quality. Agar 25% reject ho rahe hain to ya to tum galat samajh rahe ho requirement, ya tumhari reports mein evidence kam hai.

### Defect Age / Mean Time to Repair
```
Defect Age = Date closed − Date reported
MTTR = Sum of (fix time for each defect) / Number of defects
```
**Kis kaam ka:** Fix cycle kitna slow hai — process bottleneck dikhata hai.

### Defect Severity Index
```
DSI = Σ (number of defects at severity level x weight of that level) / Total defects

  weights: S1 = 4, S2 = 3, S3 = 2, S4 = 1
```
**Worked example:**
```
S1: 5 defects  x 4 = 20
S2: 20 defects x 3 = 60
S3: 50 defects x 2 = 100
S4: 25 defects x 1 = 25
Total defects = 100          Weighted sum = 205

DSI = 205 / 100 = 2.05
```
Ye batata hai ki release ki overall defect severity kya hai — 100 trivial defects aur 100 blockers ka count same hai, lekin DSI bilkul alag hoga.

### Test Automation Metrics
```
Automation Coverage    = (Automated test cases / Total test cases) x 100
Automation Pass Rate   = (Passed automated tests / Total automated tests run) x 100
Flakiness Rate         = (Tests with inconsistent results / Total tests) x 100
Automation ROI (rough) = (Manual execution time saved x number of runs)
                         − (Automation build + maintenance time)
```
**Flakiness rate** wo metric hai jo automation engineer ke liye sabse important hai — kyunki flaky suite ka trust zero hota hai, aur trust ke bina suite ki value zero hai.

## 20.3 Metrics dashboard — ek release ka snapshot

```
Release 4.12 — Test Summary Metrics
──────────────────────────────────────────────────────────
Requirements                    : 48
Requirements with coverage      : 46  (95.8%)
Requirements with NO coverage   :  2  <-- action item

Test cases planned              : 240
Executed                        : 232  (96.7%)
Passed                          : 214  (92.2% of executed)
Failed                          :  12
Blocked                         :   6
Not run                         :   8

Defects found in testing        :  58
  S1 Critical                   :   3   (all closed)
  S2 Major                      :  11   (10 closed, 1 deferred)
  S3 Minor                      :  28
  S4 Trivial                    :  16
Defect Severity Index           : 1.85

Defects rejected                :   4   (6.9% rejection ratio)
Mean time to repair             : 1.8 days

Automation coverage             : 61%  (146 of 240)
Automation flakiness rate       : 1.4% (2 of 146)  <-- target < 2%

Post-release (measured 30 days later)
Defects found in production     :   4
Defect leakage                  : 6.5%   (4 / 62)
Defect removal efficiency       : 93.5%
──────────────────────────────────────────────────────────
```

## 20.4 Vanity metrics — jinse bacho

| Metric | Kyun problematic hai |
|---|---|
| **Number of test cases written** | Zyada test cases = better coverage nahi. Log cases split karke number badha dete hain |
| **Number of defects found (by tester)** | Testers ko trivial defects log karne ka incentive milta hai; aur ye developers ke saath relationship kharab karta hai |
| **Pass percentage alone** | 100% pass ka matlab ho sakta hai ki tests weak hain, product achha nahi |
| **Code coverage alone** | Execute hona ≠ verify hona. Assertions ke bina bhi coverage 100% ho sakta hai |
| **Automation percentage alone** | Galat cheez automate karna ya flaky automation percentage badha deta hai aur value ghata deta hai |

**Rule:** metric ko **decision** drive karni chahiye. Agar koi metric dekh ke tum kuch alag nahi karoge, to wo metric report karne layak nahi hai.

> **[REAL]** Sabse relevant metric mere liye **defect leakage** hai, aur Project Sales V1 uska perfect case study hai. Waha backend ke **57 integration tests sab pass** the — matlab agar hum "test pass rate" ya "test count" ko metric maante to sab kuch **perfect** dikhta. Lekin flow completely broken tha. **Pass rate ne kuch nahi bataya, kyunki wo sirf ye measure kar raha tha ki jo tests likhe gaye the wo pass ho rahe hain — ye nahi ki sahi tests likhe gaye the.**
>
> Isliye main pass rate ko standalone metric ki tarah kabhi report nahi karta. Jo main track karta hoon wo hai: kitne defects **production mein** mile vs testing mein (leakage), aur har escaped defect ke liye ek specific sawaal — **"kaunsi test technique ise pakadti?"** Un 5 defects ke liye jawab tha: ek contract test FE-BE seam pe, aur ek post-deploy check jo customer-facing URL ko actually request karta. Wo dono ab permanent gaps hain jo main close karta hoon. **Leakage metric ka poora point yahi hai — number nahi, uske baad ka action.**

> **Interview answer:**
> The metrics I actually use fall into three groups: coverage, defect effectiveness, and process health.
>
> For defect effectiveness, defect density is defects divided by module size, and its value is that it corrects for size — a module with forty-five defects across twelve thousand lines is in better shape than one with thirty defects across four thousand, and the raw count hides that. Defect leakage is defects found in production divided by total defects found, times one hundred; if testing found one hundred and eighty and production found twenty, leakage is ten percent. Defect removal efficiency is the complement — defects found before release over total, so ninety percent in that example. Industry benchmark is roughly eighty-five percent average and above ninety-five percent good. Test case effectiveness is defects found by scripted cases over total defects found, which tells me whether my designed cases are earning their keep compared with exploratory work.
>
> I also track defect severity index, which weights defects by severity so that a hundred trivial issues and a hundred blockers do not report as the same number, and defect rejection ratio, which measures the quality of my own defect reports rather than the product.
>
> What I am careful about is vanity metrics. Test case count, defect count per tester, pass percentage on its own and code coverage on its own are all easy to move and easy to game. My own project taught me this directly: a backend feature had fifty-seven passing integration tests while the end-to-end flow was completely broken, because every one of those tests constructed its own inputs. Pass rate said one hundred percent and meant nothing, since it measured whether the tests we wrote pass, not whether we wrote the right tests.
>
> So the metric I lean on hardest is leakage, and specifically what I do with it. For every escaped defect I ask which technique would have caught it. For those five defects the answer was a contract test at the frontend-backend seam and a post-deploy check that actually requests the customer-facing URL. Those two became permanent additions. The number itself changes nothing — the question it forces is where the value is.

> **Cross-question: "Your leakage went up this quarter. Is your team performing worse?"**
>
> > Not necessarily, and I would want to look before concluding. Leakage can rise because production usage grew, because a new customer segment exercises paths we never see, because scope grew faster than test capacity, or because we consciously accepted risk to hit a date. It can also rise simply because we started tracking production defects more honestly, which looks like decline and is actually improvement. So I would break the escaped defects down by area and by cause before drawing any conclusion. What I would resist is treating leakage as a performance score for the QA team, because the fastest way to improve that number is to stop recording production defects properly, and that is exactly what happens when a metric becomes a judgement rather than a diagnostic.

> **Cross-question: "What is a good code coverage target?"**
>
> > I would not set a target number, because a target on code coverage produces tests written to raise coverage rather than to find defects — assertions get dropped and the number still goes up. What I use coverage for is finding uncovered areas and asking whether they matter. Zero coverage on a payment calculation is a real signal. Sixty percent versus eighty percent overall tells me very little. And I would point out that coverage measures that code executed, not that behaviour was verified — you can reach full coverage with a test suite that asserts nothing at all.

> **Cross-question: "Which single metric would you report to a CTO?"**
>
> > Defect leakage with its trend, plus the severity of what leaked. It is the only metric measured against reality rather than against our own plans, and it answers the question a CTO actually has, which is how much of what we ship breaks in front of customers. Everything else — case counts, pass rates, coverage percentages — measures our activity. Leakage measures our outcome.

---

# 21. Senior-Level Scenario Questions

> Ye section sabse important hai. In sawaalon mein interviewer **definition nahi** dhoondh raha — wo **structured thinking** dhoondh raha hai. Har jawab mein pehle **approach/framework** bolo, phir detail. Aur har jawab ke end mein **trade-off ya risk** mention karo — wahi senior signal hai.

---

## Q1. "How would you test a login feature?"

**Ye sabse zyada poocha jaane wala QA sawaal hai.** Junior candidate 5 cases bolta hai — valid, invalid, blank, wrong password, forgot password. Senior candidate **categories** bolta hai aur phir depth dikhata hai.

**Bolne ka structure:** *"Let me organise this into categories — functional, boundary and negative, security, API level, UI and usability, accessibility, performance, and compatibility. I will go through each."* Ye ek line hi tumhe alag kar deti hai.

### Category 1 — Functional (positive)
1. Valid username and valid password → login succeeds, correct landing page, correct role-based navigation
2. Login with email as identifier and login with username as identifier (if both are supported)
3. Login with correct credentials but different letter case in the email — email is normally case-insensitive, password is not
4. Leading and trailing whitespace in the email is trimmed and login still succeeds
5. "Remember me" checked → session persists after closing and reopening the browser
6. "Remember me" unchecked → session does not persist
7. Successful login after a password reset works with the new password
8. Successful login sets the session cookie with the correct attributes and creates a session record
9. Redirect behaviour: a user who was deep-linked to a protected page and then logged in lands on that page, not on the dashboard
10. Logout works, and after logout the back button does not restore an authenticated page
11. Multi-role users land on the correct default workspace for their role
12. Concurrent login from two devices — is it allowed, and is the older session invalidated?

### Category 2 — Negative
13. Valid username, wrong password → generic error, no login
14. Non-existent username → **the same generic error as wrong password** (see security note below)
15. Blank username, blank password, both blank → field-level validation
16. Whitespace-only username and whitespace-only password
17. Wrong password repeated up to and beyond the lockout threshold → account lockout behaviour
18. Login attempt on a deactivated, suspended or deleted account
19. Login attempt on an account that has not verified its email, if verification is required
20. Login with an expired password, if password expiry exists
21. Password reset link used twice → the second use must be rejected
22. Password reset link used after expiry → rejected
23. Login while the account is locked, then after the lockout window expires → succeeds again
24. Submitting the form twice rapidly (double-click) → only one session is created

### Category 3 — Boundary
25. Password at exactly the minimum length, and one character below it
26. Password at exactly the maximum length, and one character above it
27. Username / email at maximum allowed length
28. Email with the maximum-length local part and domain
29. Very long input — ten thousand characters in each field — no crash, no timeout, clean rejection
30. Single-character inputs in each field
31. Lockout at exactly the threshold: if the limit is five, the fifth failure and the sixth behave as specified

### Category 4 — Security (**ye category interview jitati hai**)
32. **Username enumeration** — the error for a non-existent user must be identical to the error for a wrong password, and the **response timing must also be similar**, otherwise the difference leaks which accounts exist
33. **SQL / NoSQL injection** in both fields — `' OR '1'='1`, and for a Mongo backend, operator injection such as `{"$ne": null}` in a JSON body
34. **XSS** — `<script>alert(1)</script>` in the username, reflected back in the error message
35. **Brute force protection** — rate limiting, account lockout, CAPTCHA after N failures, exponential backoff
36. **Credentials in transit** — HTTPS enforced, no HTTP fallback, HSTS present
37. **Password never appears** in the URL, in browser history, in application logs, in server logs, or in analytics events
38. **Password field masked** by default, and `autocomplete` behaviour is appropriate
39. **Session fixation** — the session identifier must be regenerated on successful login, not reused from before login
40. **Session cookie attributes** — `HttpOnly`, `Secure`, `SameSite` set correctly
41. **Session timeout** — idle timeout and absolute timeout both behave as specified
42. **Logout invalidates the session server-side**, not merely client-side — replaying the old token after logout must fail
43. **JWT specifics**, if used — expiry enforced, signature verified, algorithm not settable to `none`, token not accepted after logout if a revocation list exists
44. **Password storage** — hashed with a modern algorithm and salted; verify a password is never returned in any API response
45. **CSRF protection** on the login form if the app is cookie-based
46. **Response does not leak** stack traces, framework versions, or database errors on failure
47. **Authorisation after authentication** — a successfully logged-in user of one tenant cannot read another tenant's data by changing an ID
48. **Password reset flow security** — the token is single-use, expires, is not guessable, and is not leaked in the referrer header
49. **2FA / MFA**, if present — wrong code, expired code, reused code, backup codes, and bypass attempts by skipping straight to the post-2FA endpoint
50. **Clickjacking** — login page cannot be framed (`X-Frame-Options` / CSP `frame-ancestors`)

### Category 5 — API level
51. `POST /login` with valid credentials → 200 with the expected response shape and a token
52. Wrong password → **401**, not 400 and not 200 with an error body
53. Missing field entirely versus present-but-empty — both handled, correct status codes
54. Wrong content type, malformed JSON body → 400, no stack trace
55. Extra unexpected fields in the payload are ignored, not blindly persisted (mass assignment)
56. Case sensitivity of field names in the payload
57. Rate limiting returns **429** with a sensible `Retry-After`
58. Login endpoint is idempotent enough that a retried request does not create duplicate sessions
59. Response time of the login endpoint is measured and within the agreed threshold
60. Protected endpoints reject requests with no token, an expired token, and a malformed token — with **401** for missing/invalid authentication and **403** for insufficient permission

### Category 6 — UI / usability
61. Tab order flows logically: username → password → remember me → submit
62. Enter key submits the form from either field
63. Password visibility toggle works and does not persist the revealed state across page loads
64. Error messages appear near the relevant field and are not lost on scroll
65. The submit button shows a loading state and is disabled during submission to prevent double submits
66. Browser autofill populates the fields correctly and the form still submits
67. Copy-paste into the password field works — blocking paste breaks password managers and is a usability defect
68. Caps Lock warning, if specified
69. Links for forgot password, sign up and support are present and correct
70. The form retains the username after a failed attempt, but never the password

### Category 7 — Accessibility
71. Every input has an associated `<label>`, not just placeholder text
72. Entire flow is operable by keyboard alone, with a visible focus indicator
73. Error messages are announced to screen readers, typically via `aria-live` or `role="alert"`
74. Colour is not the only means of conveying an error state
75. Colour contrast on labels, errors and the button meets WCAG AA
76. The page has a sensible title and heading structure, and the form has an accessible name
77. Zoom to 200% keeps the form usable without horizontal scrolling

### Category 8 — Performance / reliability
78. Login response time under normal load, measured at p95, not average
79. Login under concurrent load — for example five hundred simultaneous logins — with error rate and latency measured
80. Behaviour when the authentication service or database is slow or unavailable: a clear error, no hang, no partial session
81. Lockout counters behave correctly under concurrent failed attempts from multiple sources
82. Session store behaviour when it fills or the cache is evicted

### Category 9 — Compatibility
83. Supported browsers and their previous major version
84. Mobile viewport: layout, virtual keyboard type (email keyboard for the email field), and the field not being obscured by the keyboard
85. Password manager compatibility on desktop and mobile
86. Behaviour with cookies disabled and with third-party cookies blocked
87. Behaviour on a slow or intermittent connection — submit while offline, then reconnect

**Total: 87 distinct test ideas across nine categories.** Interview mein saare mat bolo — **categories bolo, aur har category se 3–5 strongest examples do**, especially security wale.

> **[REAL]** Ek cheez jo main iss list mein apne experience se add karta hoon: **"authentication ke baad authorisation verify karo."** Mere project mein ek defect tha jahan ek **public, unauthenticated endpoint** duplicate contact email pe `400 "non unique result"` return karta tha aur uske saath **raw Mongo query bhi leak** kar deta tha. Ye login endpoint nahi tha, lekin sabak wahi hai: **auth-related endpoints pe error response ka content utna hi important hai jitna status code.** Isliye mere login negative tests sirf ye assert nahi karte ki "error aaya" — wo assert karte hain ki error mein **kya nahi hona chahiye**: koi stack trace nahi, koi query nahi, koi internal path nahi, aur non-existent user aur wrong password ke liye **bilkul same message**.

> **Interview answer:**
> I would organise it into nine categories — functional, negative, boundary, security, API level, UI and usability, accessibility, performance, and compatibility — and go deeper in the ones that carry the most risk, which for login is security.
>
> On the functional side: valid credentials landing on the correct role-based page, email treated as case-insensitive while the password is case-sensitive, whitespace trimmed on the identifier, remember-me persisting or not persisting the session correctly, login after a password reset, deep-link redirect returning the user to the page they originally requested, logout genuinely ending the session so the back button does not restore an authenticated page, and concurrent logins from two devices behaving according to the specified policy.
>
> Negative: wrong password, non-existent user, blank and whitespace-only fields, repeated failures crossing the lockout threshold, deactivated and unverified accounts, expired or already-used password reset links, and double submission creating only one session.
>
> Boundary: password at exactly the minimum and maximum lengths and one either side, maximum-length email, ten-thousand-character input rejected cleanly rather than crashing, and the lockout threshold tested at exactly N and N plus one.
>
> Security is where I would spend the most time. Username enumeration is the one people miss — the error for a non-existent account must be identical to the error for a wrong password, and the response timing must be similar too, because a measurable timing difference leaks which accounts exist. Then SQL and NoSQL injection in both fields, including operator injection like a not-equal object against a Mongo backend; cross-site scripting reflected through the error message; brute-force protection through rate limiting, lockout and backoff; HTTPS enforced with no downgrade; the password never appearing in a URL, in browser history or in any log; session identifier regenerated on login to prevent session fixation; cookies carrying HttpOnly, Secure and SameSite; logout invalidating the session server-side so a replayed token fails; and for JWT specifically, expiry enforced, signature verified, and the algorithm not settable to none. I would also verify authorisation immediately after authentication — that a logged-in user of one tenant cannot read another tenant's records by changing an identifier — because authentication passing is not the same as authorisation holding.
>
> At the API level I check the status codes precisely: 401 for wrong credentials rather than a 200 with an error body, 400 for a malformed payload, 429 with a Retry-After when rate limited, and no stack trace or database detail in any error response. On accessibility, real labels rather than placeholders, full keyboard operability with a visible focus ring, errors announced through an aria-live region, and contrast meeting WCAG AA. On performance, p95 latency under concurrent login load and graceful behaviour when the auth service is slow rather than a hang. On compatibility, mobile viewport with the correct keyboard type, password manager support, and behaviour with cookies disabled.
>
> If I had to pick the three I would not ship without: username enumeration through both message and timing, session invalidation on logout verified server-side, and authorisation checked immediately after authentication.

> **Cross-question: "You have thirty minutes to test login. What do you do?"**
>
> > I would run the highest-risk subset. Valid login for each role, wrong password, non-existent user checking that the message is identical to wrong password, blank submission, lockout at the threshold, session invalidation after logout verified by replaying the token, and one cross-tenant authorisation check after logging in. That is roughly eight checks, they cover the paths that break the product or leak data, and everything I skip is either cosmetic or already covered by regression. The categories I would consciously drop are accessibility and compatibility — not because they do not matter, but because their failures are visible and recoverable, whereas an authorisation failure is not.

> **Cross-question: "Would you automate all of these?"**
>
> > No. I would automate the functional happy paths per role, the core negative cases, the API-level status code checks and the session invalidation check, because those run every build and are cheap to assert. Security testing like injection and timing-based enumeration I would run as periodic dedicated passes with proper tooling rather than as part of every regression run, since they are slow and noisy in a build pipeline. Accessibility I would cover with an automated axe-style scan in the pipeline plus a manual keyboard-and-screen-reader pass per release, because automated scans catch roughly a third of real accessibility issues and confidently miss the rest.

> **Cross-question: "How do you test that logout actually works?"**
>
> > Not by observing that the UI returned to the login page, because that only proves the client forgot the token. I capture the session token or cookie before logout, perform the logout, and then replay a request to a protected endpoint using that captured token. It must return 401. If it still returns data, logout was purely cosmetic and the session lives on server-side — which matters enormously on a shared or public machine. I would run the same check for password change and for account deactivation, because those should invalidate existing sessions too and very often do not.

---

## Q2. "You have 1000 test cases and only 2 hours. What do you execute?"

**Ye prioritisation ka test hai. Galat jawab: "P1 cases chalaunga."** Ye adhoora hai — interviewer poochega "P1 bhi 400 hain, ab?"

**Framework — 5 filters, iss order mein:**

```
1000 cases
   │
   ├─ Filter 1: RISK        --> what breaks the business if it fails?
   ├─ Filter 2: CHANGE      --> what did this build actually touch?
   ├─ Filter 3: HISTORY     --> where have defects clustered before?
   ├─ Filter 4: SPEED       --> what gives the most signal per minute?
   └─ Filter 5: AUTOMATION  --> what can run in parallel while I test manually?
   │
   v
~60-80 cases executed + a documented statement of residual risk
```

> **Interview answer:**
> First, I would ask one clarifying question, because it changes the answer completely: what decision does this two hours support? If it is a go/no-go for a release, I optimise for finding blockers. If it is a smoke check after a hotfix, I optimise for the changed area and its blast radius. Those are different selections.
>
> Assuming it is a release decision, I apply five filters in order. First, risk — the flows where a failure costs money, corrupts data, or blocks every user. On my product that is purchase order creation, supplier acknowledgement, goods receipt and invoicing, because those are the revenue path. Second, change — I ask the developers what this build actually touched and pull in the cases covering those modules plus their immediate integration neighbours, because untouched code does not spontaneously break. Third, history — the areas that produced defects in the last two or three releases, since defects cluster and past defect location is the single best predictor of the next one. Fourth, speed — I prefer many shallow checks across breadth over one deep check in a corner, because in two hours breadth finds blockers and depth finds edge cases, and edge cases do not stop a release. Fifth, I kick off the entire automated suite in parallel on the first minute, so machine time and my time run concurrently rather than sequentially.
>
> That typically leaves me with sixty to eighty cases, and I would sequence them so the most critical run first, in case I get only ninety minutes instead of a hundred and twenty.
>
> The part I consider non-negotiable is what I deliver at the end. I would not just report pass and fail. I would state explicitly what was not executed and what risk that leaves — for example, "the bid flow and all reporting were not exercised; if a defect exists there we will find it in production." That converts my time constraint into an informed business decision rather than into silent exposure. In my experience that closing statement is what the release manager actually needs, and it is what makes the difference between QA being a gate and QA being an advisor.

> **Cross-question: "What if you have no idea what changed in the build?"**
>
> > Then getting that information is my first action, not an obstacle — five minutes asking a developer or reading the merged pull requests will save an hour of guessing. If it is genuinely unavailable, I fall back to risk and history alone, and I run the full automated regression in parallel to compensate for having no targeting information. I would also raise the absence of release notes as a process defect, because testing without knowing what changed means every cycle costs more than it needs to, and that is a fixable problem rather than a fact of life.

> **Cross-question: "You run your selection and everything passes. Do you sign off?"**
>
> > I would report that the executed subset passed and state clearly what was not covered, but I would not present that as a clean bill of health. Two hours against a thousand cases is roughly a seven percent sample, and a passing sample of that size mainly tells you there is no catastrophic failure on the paths you chose. The sign-off decision belongs to the release owner; my job is to make sure they are making it with an accurate picture of the residual risk rather than with a green tick.

---

## Q3. "A bug is not reproducible. What do you do?"

**Galat jawab: "Main close kar deta hoon."** Ye bahut common trap hai. Non-reproducible ka matlab "exist nahi karta" nahi hai — ye sabse zyada matlab hai **"ek variable jo maine record nahi kiya"**.

**Systematic approach — variables ki list:**

| Variable category | Kya check karo |
|---|---|
| **Environment** | Build/version, browser + version, OS, device, screen size, network speed |
| **User** | Role, permissions, tenant, account age, feature flags for that user |
| **Data** | Exact record IDs, record state, data volume, whether the record was pre-existing or newly created |
| **Timing** | Time of day, **day of month**, month boundary, timezone, DST, order of actions, elapsed time between steps |
| **State** | Cached data, cookies, local storage, previous session, browser back/forward, open tabs |
| **Concurrency** | Another user acting on the same record, background job running |
| **Frequency** | Truly random, or once every N attempts? Record the ratio |

> **Interview answer:**
> I treat non-reproducible as meaning there is a variable I did not capture, not as meaning the defect does not exist. Closing it is the one thing I would not do, because in my experience intermittent defects in production are the expensive ones.
>
> My process has four stages. First, I try to reproduce it myself several times and record the frequency precisely — three failures out of twenty attempts is a completely different investigation from one out of a hundred, and it also tells the developer whether they should expect to hit it. Second, I go back over the variable space systematically: build and browser version, user role and tenant and feature flags, the exact data record and whether it was pre-existing or freshly created, timing factors including time of day and day of month and timezone, cached state such as cookies and local storage, and whether another user or a background job was acting on the same record. Third, I go to the evidence rather than the reproduction — server logs around that timestamp, the correlation identifier from the request, error tracking, and any recorded session — because a defect that occurred once still left a trace, and the trace often identifies the cause faster than reproduction does. Fourth, if it remains unreproducible I do not close it; I log it with everything I have, tag it as intermittent, and add monitoring or logging around the suspected area so the next occurrence is captured with more context.
>
> I have a very specific reason for insisting on this. On my project a failure was reported as intermittent and flaky for a long time. It was not intermittent at all — it was completely deterministic, and it depended on today's day of the month. The calendar component renders a grid padded with days from adjacent months that share the same styling, so certain day numbers appear twice, and whether the failure occurred depended on where today's date fell in that grid. Anyone who had closed it as non-reproducible would have been closing a defect that fired reliably, just not on the day they tested. That case is why the first variable I now check on any intermittent report is the date.

> **Cross-question: "The developer says they cannot reproduce it either and wants it closed. What do you say?"**
>
> > I would agree not to block the release on it if the impact is low, but I would push back on closing. My suggestion would be to keep it open as a known intermittent issue and add targeted logging around the suspected path, so the next occurrence arrives with the context we are missing. That costs a developer perhaps thirty minutes and it converts an unsolvable ticket into a solvable one. If they still want it closed, I would ask that it be closed explicitly as "cannot reproduce" rather than as "not a defect", because those two mean different things to whoever reads the ticket in six months, and one of them makes the next person's investigation start from zero.

---

## Q4. "How do you test a feature with no documentation?"

**Galat jawab: "Main documentation ka wait karta hoon."** Reality mein documentation aksar hoti hi nahi — especially startups mein.

**Sources of truth, priority order:**

```
1. The people        --> PM, dev, designer, support, sales (5-min conversations)
2. The product       --> explore it; the current behaviour is a hypothesis, not a spec
3. The code / API    --> validation rules, enums, schemas, error strings, config
4. The tests         --> existing unit/integration tests encode intended behaviour
5. The tickets       --> Jira/Linear history, PR descriptions, commit messages
6. The customers     --> support tickets, PostHog analytics of real usage
7. Comparable products --> how does the rest of the industry solve this?
```

> **Interview answer:**
> I would not wait for documentation, because in most product teams it will not arrive and waiting makes me a bottleneck. Instead I reconstruct the specification from the sources that do exist, and I make my reconstruction visible so it can be corrected.
>
> I start with people, because a fifteen-minute conversation with the product owner and the developer usually beats a day of reverse engineering. I ask three questions specifically: what problem does this solve for the user, what is the one thing that must never happen, and what did you deliberately decide not to handle. That last question surfaces intentional limitations that would otherwise become defect reports and waste everyone's time.
>
> Then I explore the product itself, but I treat the current behaviour as a hypothesis rather than a specification — a common trap is deciding that whatever the software currently does is correct, which means you can never find a defect. Then I read the code and the API contract, because validation rules, enums, schemas and error strings encode the real constraints, and existing unit tests encode what the developer believed the behaviour should be. Then the ticket history, pull request descriptions and design files. Then support tickets and analytics, which tell me how the feature is actually used rather than how it was intended to be used.
>
> From all of that I write my understanding as acceptance criteria in Given-When-Then form and post it on the ticket asking for confirmation. That is the important step: reviewing a draft takes two minutes, so it actually gets read, and the moment someone disagrees I have learned the specification for free. Anything nobody confirms becomes a documented assumption, so if it turns out to be wrong the conversation afterwards is about a recorded assumption rather than about who is to blame.
>
> One thing I would add from experience: in the absence of documentation, integration seams are where the risk concentrates. When two teams build against an unwritten contract, each side assumes the other handles something. I have seen exactly that produce a record created with a missing mandatory reference, because the frontend assumed the backend would derive a field and the backend assumed the frontend would send it. So with no documentation, the questions I ask first are always about who owns which field and which state transitions exist.

> **Cross-question: "How do you know the current behaviour is a bug and not intended?"**
>
> > I check it against something other than my own opinion. First, does it violate a general principle — data integrity, security, consistency with the rest of the product? A record saved with a missing mandatory reference is wrong regardless of documentation, because no reasonable specification would ask for that. Second, is it inconsistent with how the same concept behaves elsewhere in the product? Inconsistency is nearly always unintentional. Third, would a user be surprised or harmed? If it is none of those, then it is a question rather than a defect, and I raise it as a question — because filing questions as defects damages your credibility, and credibility is what makes people act on the defects that are real.

---

## Q5. "Developer says 'it's not a bug, it's by design'. What now?"

**Galat jawab: "Main manager ko escalate karta hoon."** Escalation last resort hai, first move nahi.

> **Interview answer:**
> My first assumption is that they might be right, because sometimes they are, and starting from that position keeps the conversation productive. So I go and check the requirement, the acceptance criteria and any design documentation before responding.
>
> Then one of three things is true. If the behaviour genuinely matches a documented requirement, I was wrong, I close the defect and I update my test case — and I would say so plainly, because being wrong occasionally is what makes people trust you when you are certain. If the behaviour contradicts a documented requirement, I reply with the specific reference and the evidence, and there is usually nothing left to argue about. The interesting case is the third one, where the requirement simply does not cover it, which is the most common situation. That is not a QA-versus-developer dispute at all — it is an undecided product question, and neither of us has the authority to settle it.
>
> In that third case I stop discussing whether it is a bug, because that framing has no resolution, and I reframe it as impact. I take it to the product owner as: here is what the system currently does, here is what a user will expect, here is the consequence if we ship it as is. Then it is their decision. If they say it is intended, I close it as by design and — importantly — I ask that the behaviour be written down, so the next tester does not spend the same hour rediscovering it.
>
> I would escalate to a manager only if there is a genuine risk being dismissed rather than decided — a security exposure or a data integrity issue where someone is choosing not to look. Even then I escalate the risk, not the disagreement. Something like "this endpoint returns internal database detail to unauthenticated callers, and I want a decision recorded on whether we accept that" is a very different message from "the developer will not fix my bug", and only one of them gets acted on.

> **Cross-question: "What if the developer is dismissive and you are sure it is a real defect?"**
>
> > I stop using words and use evidence, because arguments about judgement go nowhere while artifacts do not. I attach the request and response, the persisted record showing the incorrect state, and a short recording. Then I state the consequence in business terms rather than technical ones — every record created through this path has no customer attached, so finance will reconcile them by hand and the revenue report will be wrong. At that point it is no longer my opinion against theirs; it is a documented consequence, and it goes to triage where the product owner decides the priority. And I keep the tone neutral, because I have to work with this person next sprint. The goal is the fix, not the argument.

---

## Q6. "How do you decide when testing is done?"

**Galat jawab: "Jab saare test cases pass ho jaayein."** Ye naive hai — testing kabhi complete nahi hoti, tum sirf **stop** karte ho.

**Exit criteria — multi-dimensional, ek number nahi:**

| Dimension | Criterion |
|---|---|
| **Coverage** | All planned cases executed or consciously deferred; RTM shows every requirement covered |
| **Defects** | Zero open S1/S2; open S3/S4 reviewed and accepted |
| **Trend** | Defect discovery rate has flattened; new defects are cosmetic, not structural |
| **Risk** | Every high-risk area from the risk matrix has been exercised |
| **Regression** | Regression suite green on the release candidate build |
| **Non-functional** | Performance and security checks completed for the release scope |
| **Process** | Retest complete on all fixes; no untriaged defects |
| **Business** | UAT sign-off obtained where applicable |

> **Interview answer:**
> Testing is never finished, so the real question is when we stop, and that has to be a defined decision rather than a feeling. I define exit criteria in the test plan up front, before execution starts, because deciding when to stop while under deadline pressure produces the answer the deadline wants.
>
> I look at several dimensions together. Coverage — every planned case executed or consciously deferred, and the traceability matrix showing no requirement without a test. Defects — no open critical or high severity issues, and the remaining lower-severity ones explicitly reviewed and accepted rather than merely unfixed. The defect discovery trend — this is the one people miss: if I am still finding new structural defects on the last day, the curve has not flattened and stopping is a gamble, whereas if the last twenty findings are all cosmetic, that is a genuine signal of stability. Risk — every high-risk area from my matrix has actually been exercised. Regression green on the release candidate specifically, not on an earlier build. And business sign-off where UAT applies.
>
> But I would add the honest part, because interviewers are testing for it: in a real product team, the decision to ship is a business decision, not a QA decision. My job is not to grant or withhold permission. My job is to make sure that when the business decides to ship, they know exactly what they are shipping — what was covered, what was not, what is open, and what could go wrong. If they choose to release with a known high-severity defect because a customer commitment matters more, that is a legitimate call, provided it is made with the information and recorded. What is not acceptable is shipping while believing everything was tested when it was not, and preventing that is entirely within my control.

> **Cross-question: "Management says ship tomorrow regardless. What do you do?"**
>
> > I make the risk explicit and put it in writing, and then I support the release rather than obstructing it. Specifically I would send a short summary: here is what is tested and passing, here is what is untested, here are the open defects with their severity and the realistic consequence of each. I would also propose mitigations rather than only objections, because objections without options get ignored — for example a feature flag so the risky flow can be disabled, extra monitoring on the affected path, a support team briefing, or a rollback plan we have actually verified. That turns me from a gate into someone helping them ship safely, and it means the decision is theirs with full information, which is where it should sit anyway.

---

## Q7. "You found a critical bug 1 hour before release. What do you do?"

**Ye judgement aur communication ka test hai. Panic nahi, process.**

```
0-10 min  : VERIFY   --> reproduce it, confirm it is real, check if it exists in prod already
10-20 min : ASSESS   --> blast radius, data impact, workaround, is it reversible?
20-25 min : COMMUNICATE --> tell the release owner immediately with facts, not alarm
25-40 min : OPTIONS  --> present 3-4 concrete options with trade-offs
40-60 min : SUPPORT  --> execute whichever option is chosen; verify it
```

> **Interview answer:**
> The first thing I do is not raise the alarm — it is to make sure I am right, because a false critical one hour before release costs enormous credibility and I will need that credibility the next time. So within the first ten minutes I reproduce it cleanly, confirm the exact steps, and check one specific thing: does this defect already exist in the current production build? If it does, it is not a release blocker at all — it is a pre-existing issue and the release does not make it worse, which completely changes the decision.
>
> Assuming it is new, I assess impact fast. How many users hit this path, does it corrupt data or is it merely visible, is there a workaround, and is the damage reversible? Data corruption is the category I treat most seriously, because a display bug can be fixed after release while corrupted records may need manual reconciliation and may not be recoverable at all.
>
> Then I communicate immediately, to the release owner, with facts rather than urgency. I would say what breaks, who it affects, whether there is a workaround, and what I recommend — not "we have a critical bug, we cannot ship."
>
> And I bring options rather than a veto, because at this point the decision is a business trade-off and my job is to make it a well-informed one. Typically four: delay the release and fix it properly; ship with the affected feature disabled behind a flag; ship as is with a documented known issue, support briefed and monitoring in place; or take a targeted hotfix now if the fix is genuinely small and verifiable in the time available — and I would be candid that a rushed fix carries its own risk, because a fix written in forty minutes without regression is how a critical bug becomes two.
>
> Then whatever they choose, I execute it and verify it rather than continuing to argue. If we ship with the known issue, I make sure it is written down, monitored, and communicated to support so the first customer report is not a surprise.
>
> The one case where I would push much harder is data integrity or a security exposure — a record being silently created in a corrupt state, or an unauthenticated endpoint leaking internal detail. Those are not reversible by a patch next week, because the damage accumulates from the moment you ship. For everything else I present the trade-off; for those I would state plainly that I recommend against releasing and ask for the decision to be recorded.

> **Cross-question: "The developer produces a fix in twenty minutes. Do you accept it?"**
>
> > Cautiously, and I would say so out loud. I would retest the specific defect, and then run impact-based regression around whatever the fix touched, because a change made in twenty minutes under deadline pressure has had less design thought than anything else in the release. If my automated regression around that area can run in the remaining time, that decides it. If it cannot, I would tell the release owner exactly that: the original defect is verified fixed, but the fix itself is unregressed, and here is what could break. Quite often the honest answer is that the rushed fix is riskier than the known bug, and saying that clearly is more useful than accepting a fix just because it arrived.

---

## Q8. "How would you test a search functionality?"

**Categories mein todo — yahan bhi wahi structure kaam karta hai.**

### Functional
1. Exact match returns the expected result
2. Partial match / substring search
3. Case-insensitive search ("STEEL" = "steel" = "Steel")
4. Multi-word search — is it AND or OR semantics? **This is a real requirement question**
5. Search across multiple fields (PO number, supplier name, item description)
6. Leading and trailing whitespace trimmed
7. Special characters in the query — hyphen, apostrophe (`O'Brien`), slash, ampersand
8. Unicode and non-English input
9. Numeric search — PO number, quantity, amount
10. Search with filters applied together — does search operate within the filter, or reset it?
11. Sorting applied on top of search results
12. Pagination of results — page 2 keeps the query
13. Clearing the search restores the unfiltered list
14. Search term persists (or does not) after navigating away and back — specified behaviour either way

### No-result / negative
15. No matches → a clear empty state, not a blank page or a spinner forever
16. Empty query submitted → all results, or a validation message — which is specified?
17. Whitespace-only query
18. Very long query (500+ characters)
19. Query consisting only of special characters
20. Query with SQL/NoSQL injection payloads and XSS payloads — results must not execute anything and errors must not leak query structure
21. Wildcards typed by the user (`*`, `%`, `_`) — treated literally or as wildcards?

### Relevance and behaviour
22. Most relevant result appears first — exact match should outrank partial match
23. Typo tolerance / fuzzy matching, if specified
24. Synonyms and stemming ("cement bag" vs "cement bags")
25. Search suggestions / autocomplete correctness and ordering
26. Recent searches, if the feature exists
27. Highlighting of the matched term in results

### Boundary and scale
28. Query of exactly one character — is a minimum length enforced?
29. Query at the maximum allowed length and one over
30. Result set with exactly one match
31. Result set with zero matches
32. Result set larger than one page, and exactly one page
33. Search across a large dataset — 50,000+ records — response time at p95

### Performance and UX
34. Debouncing on type-ahead — a request should not fire on every keystroke
35. Rapid typing then deletion does not leave a stale result set from an earlier request (race condition — a late response overwriting a newer one is a classic search bug)
36. Loading state shown during the request
37. Search request is cancellable / superseded correctly
38. Slow backend → graceful timeout message, not a hang

### Security and permissions
39. A user only sees results they are authorised to see — search must not become a data-leak channel by returning another tenant's records
40. Search does not reveal the existence of records the user cannot open

> **Interview answer:**
> I would cover it in six groups: functional matching, no-result and negative handling, relevance, boundary and scale, performance and UX, and security.
>
> Functionally: exact match, partial match, case insensitivity, and multi-word behaviour — and multi-word is a requirement question rather than a test, because AND semantics and OR semantics produce very different products and the requirement usually does not say. Then which fields are searched, how special characters like apostrophes and hyphens behave, unicode input, and how search interacts with filters, sorting and pagination — specifically whether page two retains the query, which is a common defect.
>
> For no results, I check that there is a proper empty state rather than a blank screen or a permanent spinner, and I check what an empty query does, since returning everything and returning a validation message are both defensible and only one is specified. Negative cases include very long queries, queries of only special characters, injection payloads, and user-typed wildcards being handled either literally or as wildcards but consistently.
>
> Relevance is where search quality actually lives — an exact match should rank above a partial match, and if fuzzy matching or stemming is claimed, I test it with real misspellings and plurals rather than assuming.
>
> On performance and UX, two things matter most. First, debouncing, so a request does not fire on every keystroke. Second, and this is the bug I would look for hardest: the race condition where a user types quickly, several requests are in flight, and a slower earlier response arrives last and overwrites the correct newer results. That produces results that do not match the query in the box, it is intermittent, and it is very often dismissed as flaky rather than investigated.
>
> Security last but not least: search must not become a data leakage channel. A user should only get results they are authorised to open, and search should not even reveal that records exist in another tenant. I would test that by searching for a term I know exists only in another tenant's data and confirming zero results rather than a permission error, because a permission error still confirms the record exists.

---

## Q9. "How would you test a file upload feature?"

### Functional
1. Upload a valid file of each supported type, and verify it is stored, listed and retrievable
2. Upload via browse, via drag and drop, and via paste if supported
3. Multiple files selected at once
4. Upload progress indicator is accurate and completes
5. Cancel an in-progress upload → no partial file is retained
6. Download the uploaded file and verify it is byte-identical to the original
7. Preview / thumbnail generation for supported types
8. Delete an uploaded file — hard or soft delete, and what happens to the underlying storage object
9. Replace an existing file with a new version
10. Uploaded file is correctly associated with the parent record and visible on it

### File type and content
11. Each explicitly allowed extension
12. Each explicitly blocked extension — `.exe`, `.sh`, `.bat`
13. **A file renamed to a permitted extension** — `payload.exe` renamed to `invoice.pdf`. Validation must check the actual content signature, not just the extension
14. A file with no extension at all
15. A valid extension with corrupt content — a `.pdf` that is not actually a PDF
16. An SVG containing script content, if SVG is allowed — a common XSS vector
17. A macro-enabled document, if Office formats are allowed
18. An archive containing a nested archive — zip bomb protection
19. A file whose MIME type in the request contradicts its extension

### Boundary
20. Zero-byte file
21. One-byte file
22. A file exactly at the maximum size limit — is the limit inclusive or exclusive?
23. A file one byte over the maximum
24. Maximum number of files attached to one record, and one over
25. Filename at the maximum allowed length, and one over
26. Filename of a single character

### Filename handling
27. Filename with spaces
28. Filename with unicode characters and emoji
29. Filename with special characters — quotes, ampersands, hashes, percent signs
30. **Path traversal in the filename** — `../../etc/passwd`
31. Duplicate filename uploaded twice — overwrite, reject, or auto-rename, and which is specified
32. Filename that is a reserved OS name — `CON`, `NUL`, `PRN` on Windows

### Negative and failure
33. Upload with no file selected
34. Network interruption mid-upload → clear failure, no partial record, resume if supported
35. Session expiry during a long upload
36. Storage service unavailable → does the parent record still save, or does the whole transaction fail?
37. Uploading while offline, then reconnecting
38. Server rejects the file → the UI reflects the rejection rather than showing success

### Security
39. Virus scanning, if it exists — is the file accessible before the scan completes?
40. The stored file is served through a signed or authenticated URL, not from a publicly readable location
41. **Direct URL access by an unauthenticated user, and by an authenticated user from a different tenant** — the most common real defect in this feature
42. The uploaded file is not executed or interpreted by the server
43. Stored filenames are sanitised, and the original filename is not used directly as a storage path
44. Upload rate limiting to prevent storage exhaustion

### Performance and UX
45. Large file upload time and whether the UI remains responsive
46. Concurrent uploads by many users
47. Upload on a slow mobile connection
48. Accessibility — the file input is reachable and operable by keyboard, and upload status is announced

> **Interview answer:**
> I group file upload into functional, file type and content validation, boundaries, filename handling, failure modes, security, and performance.
>
> Functionally I verify each supported type uploads, stores, lists and downloads, and specifically that the downloaded file is byte-identical to what was uploaded — corruption during storage is a real and easily missed defect. I check cancel mid-upload leaves no partial record, and that the file is correctly associated with its parent entity.
>
> Content validation is where I focus hardest. Testing that the allowed extensions work is trivial; the interesting test is renaming an executable to a permitted extension and confirming the system validates the actual content signature rather than trusting the extension. Related cases are an SVG containing script if SVG is permitted, a document with macros, an archive containing nested archives, and a request whose declared MIME type contradicts the extension.
>
> Boundaries are zero bytes, one byte, exactly the size limit — checking whether the limit is inclusive — one byte over, the maximum attachment count and one over, and maximum filename length. Filename handling deserves its own pass: spaces, unicode, quotes and ampersands, path traversal sequences, duplicate filenames, and reserved operating system names.
>
> On failure modes I check network interruption mid-upload, session expiry during a long upload, and the storage service being unavailable — that last one matters because it determines whether the parent record saves without its attachment or the whole operation fails, and both are defensible but only one is intended.
>
> Security is where the most serious defects live. The one I would check first is direct URL access: can an unauthenticated user, or an authenticated user from a different tenant, fetch the stored file by its URL? Files are very often placed in publicly readable storage and protected only by the fact that the URL is not linked anywhere, which is not protection. I would also verify the server does not execute or interpret uploaded content, that stored filenames are sanitised rather than used directly as paths, and that there is rate limiting so a single account cannot exhaust storage.

---

## Q10. "What would you do in your first 30 days as the only QA in a startup?"

**Ye leadership aur prioritisation ka sawaal hai. Galat jawab: "Main test cases likhna shuru karta hoon."** Pehle **samajhna** hai, phir **stabilise**, phir **build**.

```
Week 1 : LEARN     --> product, users, domain, team, current pain
Week 2 : ASSESS    --> risk map, quality baseline, biggest gaps
Week 3 : STABILISE --> smoke suite, defect process, quick wins on the riskiest path
Week 4 : SYSTEMISE --> automation foundation, metrics, a written strategy, next-90-day plan
```

> **Interview answer:**
> I would split it into four weeks, and I would deliberately not start by writing test cases, because effort spent before understanding the risk almost always lands in the wrong place.
>
> Week one is learning. I would use the product as a real user does, end to end, and in a construction ERP that means understanding the domain — what a purchase order actually means to a site engineer, what happens when an invoice does not match a receipt. I would sit with support or customer-facing people, because they know where the product actually hurts, and read the last few months of bug reports to find where defects cluster. I would talk to the developers about which parts of the codebase they are afraid of, which is one of the most accurate risk signals available and it costs one conversation. I would also find out what happens today when something breaks in production, because that tells me how mature the process is far better than asking.
>
> Week two is assessment. I would produce a risk map — the flows ranked by business impact and by failure probability — and a quality baseline: how many defects escape to production now, how long fixes take, what test coverage exists at any level, and what environments exist. Crucially I would identify what is missing rather than only what is weak, because a missing category, such as nobody ever verifying that a deployed page is actually reachable, is more dangerous than a weak one.
>
> Week three is stabilising, and I would pick things that deliver value inside the month rather than in six months. First, a smoke suite covering the two or three flows that must never break, automated and running on every deploy — that is the highest return per hour of work available in a startup. Second, a defect process that is minimal but real: a report template with mandatory evidence, severity and priority defined and distinguished, and a triage rhythm. Third, deep testing on the single riskiest flow identified in week two, because I need an early, visible, credible win to establish that QA is worth listening to.
>
> Week four is systemising: an automation foundation with a sensible structure rather than a pile of scripts, a small set of metrics starting with defect leakage, a short written test strategy so the approach outlives me, and a ninety-day plan agreed with the founders.
>
> Two things I would be careful about as the only QA. First, I would not try to become a manual gate for everything — one person cannot test a whole product every release, and trying makes you a bottleneck and then a scapegoat. My leverage is in prevention: asking the right questions at refinement, and automating the checks that repeat. Second, I would attack the seams rather than the components. I have seen a backend with fifty-seven passing integration tests where the end-to-end flow was completely broken, because every test constructed its own inputs and the real boundary between frontend and backend was never exercised. In a startup with no QA history, the untested space is almost always the seams and the deployment, not the individual functions — the developers have usually covered those reasonably well.

> **Cross-question: "What if the founders expect you to test everything manually before every release?"**
>
> > I would show the arithmetic rather than push back with an opinion. If the product has six core flows and a full manual pass takes three days, then at a weekly release cadence I am a permanent bottleneck and the team's velocity is capped by me — which is not what they are paying for. I would propose the alternative concretely: automate the six flows, get the pass down to twenty minutes on every deploy, and spend my human time on new features and exploratory testing where a machine cannot help. And I would ask for a specific, time-boxed investment — for example three weeks to build that foundation — rather than framing it as a general principle, because founders respond to a defined cost with a defined return.

---

# 22. Red Flags — Ye Jawab Mat Dena

> Ye section utna hi important hai jitna sahi jawab yaad karna. In sentences mein se koi bhi bol diya to interviewer ka mental model turant "junior" pe set ho jaata hai — chahe baaki interview achha gaya ho.

## 22.1 Attitude red flags

| ❌ Mat bolo | Kyun galat hai | ✅ Iske badle bolo |
|---|---|---|
| "QA ka kaam bugs dhoondhna hai" | Purely reactive, bug-hunter mentality | "My job is to give the team accurate information about product risk. Finding defects is one way I do that; preventing them at refinement is a cheaper way." |
| "Main developer ka wait karta hoon" | Passive, blocked hone pe ruk jaata hai | "While blocked I quantify the impact, look for a workaround at API level, and keep executing the unaffected cases." |
| "Ye developer ki galti thi" | Blame culture, team fit ka red flag | "It was a gap in the contract between the two services — nobody had written down which side owned that field." |
| "Maine 200 bugs find kiye" | Vanity metric, adversarial framing | "The metric I track is leakage — how many defects reached production versus how many we caught, and what technique would have caught each escape." |
| "Testing 100% guarantee deti hai ki koi bug nahi hai" | Factually wrong | "Testing reduces risk and gives information; it cannot prove the absence of defects. What I can guarantee is that the team knows what was and was not covered." |
| "Automation manual testing ko replace kar degi" | Naive; automation checks known expectations only | "Automation handles the repeatable checks so my human time goes to exploration, which is where new information comes from." |

## 22.2 Technical red flags

| ❌ Mat bolo | Kyun galat hai | ✅ Iske badle bolo |
|---|---|---|
| "Test flaky tha, maine retry laga diya" | Ye information hide karna hai, fix nahi | "I investigate before I retry. In my case a step that looked flaky was fully deterministic — it depended on the day of the month — and retrying would have permanently hidden a real defect in my locator strategy." |
| "Maine `time.sleep()` laga diya wait ke liye" | Slow + still unreliable | "I wait on a condition — the element state or a network response — never on a fixed duration." |
| "Maine `.first` use kiya kyunki multiple match aa rahe the" | Ambiguity ko chhupana | "If a locator matches more than one element, that is a signal I have not scoped it correctly. I narrow the scope rather than picking an arbitrary match." |
| "Severity aur priority basically same cheez hai" | Fundamental confusion | "Severity is technical impact and it is mine to assess; priority is business urgency and it belongs to product. That is why high-severity low-priority and low-severity high-priority both exist." |
| "100% code coverage matlab fully tested" | Coverage ≠ verification | "Coverage tells me the code executed, not that the behaviour was verified. You can hit full coverage with a suite that asserts nothing." |
| "Saare test cases pass ho gaye to product ready hai" | Weak tests bhi pass hote hain | "A green suite tells me the things we thought of are working. I have seen fifty-seven passing integration tests alongside a completely broken flow, because each test built its own inputs and the real seam was never exercised." |
| "Main sab kuch automate kar dunga" | No ROI thinking | "I automate what is repeated, stable and high-risk. Automating an unstable UI or a one-off check costs more than it returns." |

## 22.3 Process red flags

| ❌ Mat bolo | Kyun galat hai | ✅ Iske badle bolo |
|---|---|---|
| "Documentation nahi thi to main test nahi kar saka" | Blocker banna, startup mein disqualifying | "I reconstruct the specification from the developer, the code, the existing tests and the tickets, write it up as acceptance criteria, and get it confirmed." |
| "Maine bug close kar diya kyunki reproduce nahi hua" | Sabse mehnge bugs intermittent hote hain | "Non-reproducible means there is a variable I did not capture. I record the frequency, work through the variable space, and go to the logs rather than closing it." |
| "Requirement mein likha nahi tha to maine test nahi kiya" | Requirements hamesha adhoore hote hain | "A missing requirement is itself a finding. When a state has no defined exit transition, that gap is the defect." |
| "Manager ne bola isliye maine ship kar diya" | No professional voice | "I document the residual risk, propose mitigations like a feature flag and extra monitoring, and support the decision — but in writing, so it is made with full information." |
| "Estimation mein maine 5 din bola tha" | Single-point estimate, no assumptions | "I give a range with stated assumptions, so if an assumption breaks the estimate renegotiates itself instead of me looking slow." |
| "Hum Agile follow karte hain isliye documentation nahi likhte" | Agile manifesto ka misquote | "Agile says working software over comprehensive documentation, not instead of. The form changes, the decisions still have to be recorded somewhere." |

## 22.4 Interview-behaviour red flags

- **Sawaal ka jawab dene se pehle clarify na karna.** "How would you test X?" sunte hi list start kar dena junior signal hai. Pehle poocho: kaun users hain, kya constraints hain, kya risk hai. Ek clarifying question tumhe alag kar deta hai.
- **Structure ke bina bolna.** Random test ideas ki list dena. Pehle categories bolo, phir depth.
- **Trade-off na mention karna.** Har senior jawab mein ek trade-off hota hai. "I would automate this but not that, because..."
- **"Mujhe nahi pata" ki jagah bakwaas karna.** Honest "I have not worked with that, but based on the principle I would expect..." bahut better hai galat confident jawab se.
- **Apne project ka example na dena.** Har definition ke baad ek `[REAL]` example — yahi tumhara sabse bada differentiator hai.
- **Sirf definitions bolna.** Definition har candidate ko aati hai. Jo tumne actually **kiya** aur usse jo **seekha**, wo sirf tumhare paas hai.
- **Developer ke baare mein negative bolna.** Chahe sach ho. Team-fit red flag hai.

## 22.5 Do sentences jo tumhe hire karwa sakte hain

Ye tumhare **signature stories** hain. Har interview mein inhe use karo:

> **The date picker story (technical depth + root cause discipline):**
> "A step in my suite looked flaky — it stalled thirty seconds and then took a fallback path. It was not flaky, it was fully deterministic. The input is read-only because the component never passes the option that allows free text, so the fill call could never succeed. The fallback matched the day by text, but the calendar renders a six-by-seven grid padded with adjacent-month days sharing the same class, so eleven of the forty-two day numbers are duplicated, and a minimum-date rule disables past days. Taking the first match landed on a disabled out-of-month cell. Whether it failed depended on today's day of the month. A retry would have hidden all of that."

> **The 57 tests story (systems thinking + test-value judgement):**
> "The backend had fifty-seven passing integration tests while the flow was completely broken. Each test constructed its own inputs — it minted its own token and passed its own customer identifier — so every test verified the scenario its author imagined rather than the one the frontend actually produces. The real seam was never exercised. That is why I now ask one question of any test: where does its input come from? If the test controls all of its inputs, it can only confirm what I already believed."

---

# 23. Quick Revision Table

> Interview se 30 minutes pehle sirf ye table padho.

## 23.1 Core definitions — one-liners

| Concept | One-line answer |
|---|---|
| **SDLC** | Framework defining how software goes from requirement to maintenance; models differ in ordering and feedback tightness |
| **V-Model** | Every dev phase has a paired test level; test design happens on the left, execution on the right |
| **STLC 6 phases** | Requirement analysis → Test planning → Test case development → Environment setup → Execution → Closure |
| **Verification vs Validation** | Verification: are we building it right (reviews, static). Validation: are we building the right thing (execution) |
| **Test Plan vs Test Strategy** | Plan = project/release level, what and when. Strategy = organisation level, how we test generally |
| **Test Scenario vs Test Case** | Scenario = one-line what to verify. Case = detailed steps, data, expected result. 1 scenario → 3-15 cases |
| **Severity vs Priority** | Severity = technical impact, QA decides. Priority = business urgency, product decides. Independent |
| **Smoke vs Sanity** | Smoke = wide + shallow, whole build, scripted. Sanity = narrow + deep, one area, usually ad-hoc |
| **Retesting vs Regression** | Retest = the failed case on the fixed build with the same data. Regression = did anything else break |
| **Alpha vs Beta** | Alpha = internal users, our environment, controlled. Beta = external users, their environment, real world |
| **Stub vs Driver** | Stub replaces the **called** module (top-down). Driver replaces the **calling** module (bottom-up) |
| **Exploratory vs Ad-hoc** | Exploratory is charter-driven, time-boxed, documented. Ad-hoc is unstructured and unreportable |
| **Functional vs Non-functional** | What it does vs how well it does it |
| **Blocked vs Failed** | Blocked = could not run (environment/dependency). Failed = ran and behaved wrongly (quality) |

## 23.2 Formulas

| Metric | Formula |
|---|---|
| Defect Density | Defects found / Size (KLOC or function points) |
| Defect Leakage | (Production defects / Total defects) x 100 |
| Defect Removal Efficiency | (Defects found before release / Total defects) x 100 = 100 − Leakage |
| Test Case Effectiveness | (Defects found by test cases / Total defects) x 100 |
| Requirement Coverage | (Requirements with ≥1 test / Total requirements) x 100 |
| Defect Rejection Ratio | (Rejected defects / Total raised) x 100 |
| Defect Severity Index | Σ(defects at level x weight) / Total defects; S1=4, S2=3, S3=2, S4=1 |
| PERT Expected | (O + 4M + P) / 6 |
| PERT Std Deviation | (P − O) / 6 |
| Risk Score | Probability x Impact |

## 23.3 Techniques — kab kaunsi

| Input shape | Technique | Worked example to quote |
|---|---|---|
| Numeric range | EP + BVA | Age 18-60: partitions <18, 18-60, >60 + non-numeric/decimal/empty; 2-value BVA = 17, 18, 60, 61; 3-value = 17, 18, 19, 59, 60, 61 |
| Fixed option set | EP | Each option is its own partition |
| Multiple conditions → outcome | Decision table | 3 binary conditions = 8 rules; PO approval (limit / verified supplier / budget) exposes that rule 6 has no defined behaviour |
| Entity with statuses | State transition | DRAFT → SUBMITTED → ACKNOWLEDGED → RECEIVED → INVOICED; invalid: acknowledge twice, cancel after receipt, any event from CANCELLED |
| Many config parameters | Pairwise | 4 browsers x 3 OS x 3 roles x 2 currencies = 72 → ~12 cases |
| Business workflow | Use case testing | The P2P end-to-end suites |
| Final sweep | Error guessing | Double-click, back button, session expiry, `O'Brien`, trailing whitespace |

## 23.4 The 4-quadrant severity/priority matrix

| | **High Priority** | **Low Priority** |
|---|---|---|
| **High Severity** | Login broken for all users; record created with no customer attached | Crash on a browser with 0.1% traffic; data bug in a report only run at year-end |
| **Low Severity** | Company name misspelled on a customer invoice; wrong currency symbol | Label misaligned on an internal admin settings page |

## 23.5 Scenario answers — the opening line for each

| Question | Open with |
|---|---|
| Test a login feature | "Let me organise this into nine categories — functional, negative, boundary, security, API, UI, accessibility, performance, compatibility." |
| 1000 cases, 2 hours | "First, what decision does this two hours support? Then I filter by risk, change, defect history, speed, and what I can run in parallel." |
| Bug not reproducible | "Non-reproducible means there is a variable I did not capture. I record the frequency and work through the variable space before I go anywhere near closing it." |
| No documentation | "I reconstruct the spec from people, the product, the code, the existing tests and the tickets, write it as acceptance criteria, and get it confirmed." |
| "It's by design" | "First I check whether they're right. Usually the requirement doesn't cover it, which makes it a product decision rather than a QA-vs-dev dispute." |
| When is testing done | "Testing is never done, we stop. Exit criteria are defined up front: coverage, open defects, the defect discovery trend, risk areas, and business sign-off." |
| Critical bug before release | "First I verify it's real and check whether it already exists in production. Then impact, then I bring the release owner four options with trade-offs." |
| Test search | "Functional matching, no-result handling, relevance, boundary and scale, performance and UX, and security — and the bug I'd hunt hardest is the stale-response race condition." |
| Test file upload | "Functional, content validation, boundaries, filename handling, failure modes, security, performance — and the first security check is direct URL access from another tenant." |
| First 30 days as only QA | "Week 1 learn, week 2 assess and build a risk map, week 3 stabilise with a smoke suite and a defect process, week 4 systemise. I wouldn't start by writing test cases." |

## 23.6 Merlin project — cheat sheet

| Ask kya sakte hain | Bolna kya hai |
|---|---|
| **Project** | Construction ERP. Backend Kotlin/Spring Boot with MongoDB, frontend Next.js with Mantine. I own QA for the procure-to-pay flows. |
| **What you automated** | Six end-to-end suites, twelve to fifteen steps each, covering PO create → supplier acknowledge → receive → invoice across material, trade and custom item types, plus bid flows. Playwright with Python. They run against production. |
| **Hardest technical problem** | The date picker. Read-only input plus a padded six-by-seven calendar grid with eleven duplicated day numbers plus disabled past days — a first-match locator picked a disabled out-of-month cell and stalled thirty seconds. Looked flaky, was deterministic on day-of-month. |
| **Best defect you found** | Project Sales V1 — five defects, three blockers: a record created with a null customer because neither side owned the field; a customer-facing endpoint returning null totals and empty line items; an offer minted in draft that nothing ever published, so the email fell back to a legacy tokenless URL the accept gate rejected; the correct tokenized link 404ing because the page was not deployed on any portal including production; and a public unauthenticated endpoint returning 400 on a duplicate contact email while leaking the raw database query. |
| **What you learned from it** | Fifty-seven backend integration tests were passing throughout, because each one constructed its own inputs. The seam was never exercised. That changed how I judge whether a test is worth anything. |
| **Tools** | Jira, Linear, DevRev, PostHog, JMeter, Postman, Playwright with Python. |
| **What you'd improve** | A production-like staging environment so the end-to-end suites do not need to run against production, contract tests at the frontend-backend seam, and a post-deploy check that actually requests customer-facing URLs on every portal. |

---

> **Aakhri baat.** In interviews, definition har koi bol deta hai. Jo tumhe alag karta hai wo teen cheezein hain: **structure** (categories pehle, detail baad mein), **apne project ka concrete example** har concept ke saath, aur **trade-off** har jawab ke end mein. Date-picker aur 57-tests wali do stories tumhare paas hain — inhe har relevant sawaal mein le aao. Wo tumhare paas hain, kisi aur candidate ke paas nahi.
