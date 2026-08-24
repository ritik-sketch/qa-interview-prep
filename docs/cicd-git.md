# 04 — CI/CD & Git (Senior SDET Interview Prep)

> **Kaise use karein:** Har concept ka structure fixed hai — **Kya hai → Kyun matter karta hai → Kab use hota hai → Interview answer (English) → Cross-question**.
> Explanation Hinglish mein hai taaki concept dimaag mein baithe. Lekin jo blockquote mein `> **Interview answer:**` likha hai — **wahi bolna hai, English mein, waise ka waisa**.
>
> `> **[REAL]**` wale boxes tumhare Merlin AI project ke actual examples hain. Interview mein sabse zyada weight inhi ka hai.
>
> **Is document ka mizaaj:** Tumne khud bola ki CI/CD tumhara biggest gap hai, aur ye bhi ki *"agar QA ko CI/CD nahi aata to log maan lete hain ki wo random kaam kar raha hai"*. Ye baat **bilkul sahi** hai, aur reason ye hai:
>
> Ek QA jo sirf local machine pe test chalata hai, wo **ek insaan** hai jo testing karta hai. Ek QA jo pipeline mein test wire kar deta hai, wo ek **system** ban jaata hai jo har commit pe testing karta hai. Pehle wale ko chhutti pe bhejo to testing ruk jaati hai. Doosre wale ko chhutti pe bhejo to testing chalti rehti hai. Senior SDET role ka poora matlab yahi hai — **tum ek testing system banate ho, tum khud testing "resource" nahi ho.**
>
> Isliye is document mein sirf theory nahi hai — **step-by-step setup instructions** hain jo tum literally follow karke apne Merlin repo mein pipeline khada kar sakte ho. Interview mein "maine kiya hai" bolna aur "maine padha hai" bolna — zameen aasman ka farq hai.

---

## Table of Contents

### Part A — CI/CD

| # | Section |
|---|---|
| 1 | [CI kya actually solve karta hai — Integration Hell](#1-ci-kya-actually-solve-karta-hai--integration-hell) |
| 2 | [CI vs Continuous Delivery vs Continuous Deployment](#2-ci-vs-continuous-delivery-vs-continuous-deployment) |
| 3 | [Trunk-Based Development vs GitFlow](#3-trunk-based-development-vs-gitflow) |
| 4 | [Build Once, Deploy Many](#4-build-once-deploy-many--sabse-under-rated-principle) |
| 5 | [Pipeline Anatomy — Har Stage](#5-pipeline-anatomy--har-stage-detail-mein) |
| 6 | [QA ka Role Har Stage Pe](#6-qa-ka-role-har-stage-pe) |
| 7 | [Quality Gates](#7-quality-gates) |
| 8 | [Deployment Strategies](#8-deployment-strategies) |
| 9 | [GitHub Actions — Complete + Setup Steps](#9-github-actions--complete-guide--setup-steps) |
| 10 | [Jenkins Basics](#10-jenkins-basics) |
| 11 | [GitLab CI](#11-gitlab-ci-brief) |
| 12 | [Docker for QA + Setup Steps](#12-docker-for-qa--setup-steps) |
| 13 | [Test Execution in CI](#13-test-execution-in-ci) |
| 14 | [Secrets Management](#14-secrets-management) |
| 15 | [Pipeline Performance](#15-pipeline-performance--slow-pipeline-diagnose-karna) |
| 16 | [CI/CD Senior Scenario Questions](#16-cicd-senior-scenario-questions) |

### Part B — Git

| # | Section |
|---|---|
| 17 | [Git ka Mental Model](#17-git-ka-mental-model) |
| 18 | [Repo Setup — clone, init, remote](#18-repo-setup--clone-init-remote) |
| 19 | [fetch vs pull vs push](#19-fetch-vs-pull-vs-push) |
| 20 | [Branch, checkout vs switch, restore](#20-branch-checkout-vs-switch-restore) |
| 21 | [add, commit, amend](#21-add-commit-amend) |
| 22 | [Merge vs Rebase](#22-merge-vs-rebase) |
| 23 | [Conflict Resolution — Worked Example](#23-conflict-resolution--step-by-step-worked-example) |
| 24 | [cherry-pick](#24-cherry-pick) |
| 25 | [stash](#25-stash) |
| 26 | [reset vs revert vs checkout](#26-reset-vs-revert-vs-checkout) |
| 27 | [reflog — Lost Commits Recover Karna](#27-reflog--lost-commits-recover-karna) |
| 28 | [log, diff, show, blame, bisect](#28-log-diff-show-blame-bisect) |
| 29 | [Tags & Releases](#29-tags--releases) |
| 30 | [.gitignore & .gitattributes](#30-gitignore--gitattributes) |
| 31 | [Branching Strategies](#31-branching-strategies-comparison) |
| 32 | [PR Workflow — QA ka Nazariya](#32-pr-workflow--qa-ka-nazariya) |
| 33 | [Git Hooks & pre-commit](#33-git-hooks--pre-commit-framework) |
| 34 | [Monorepo vs Polyrepo](#34-monorepo-vs-polyrepo) |
| 35 | [Git Senior Scenarios](#35-git-senior-scenarios) |

### Part C — Closing

| # | Section |
|---|---|
| 36 | [Red Flags — Ye Jawab Mat Dena](#36-red-flags--ye-jawab-mat-dena) |
| 37 | [Command Cheatsheet](#37-command-cheatsheet) |
| 38 | [Quick Revision Table](#38-quick-revision-table) |

---
---

# PART A — CI/CD

---

# 1. CI kya actually solve karta hai — Integration Hell

## 1.1 Kya hai

**Continuous Integration (CI)** = har developer apna kaam **din mein kam se kam ek baar** shared mainline (`main` branch) mein merge karta hai, aur **har merge pe automatically** build + tests chalte hain.

Do words important hain:
- **Continuous** — thodi thodi der mein, chhote chhote chunks mein. Hafte mein ek baar nahi.
- **Integration** — sabka code ek jagah milaana.

Log samajhte hain ki CI ka matlab "Jenkins/GitHub Actions setup karna" hai. **Galat.** Tool CI nahi hai. CI ek **practice** hai: chhote commits, roz merge, har merge pe automated verification. Tool sirf us practice ko enforce karta hai.

## 1.2 Kyun — "Integration Hell" kya hoti hai

CI se pehle ki duniya socho. 5 developers hain, sab apni-apni feature branch pe 3 hafte kaam kar rahe hain. Har koi apne hisse mein khush hai — "mere paas to chal raha hai".

Ab release ke pehle sab merge karte hain. Ab kya hota hai:

```
 Day 0                                          Day 21
   |                                              |
   |--- Dev A: 40 commits, refactored auth -------|
   |--- Dev B: 35 commits, changed User model ----|   <-- A aur B dono
   |--- Dev C: 50 commits, new PO module ---------|       ne User touch kiya
   |--- Dev D: 20 commits, upgraded library ------|
   |--- Dev E: 60 commits, DB schema change ------|
   |                                              |
                                                  v
                                    ┌───────────────────────────┐
                                    │   MERGE DAY = INTEGRATION │
                                    │          HELL             │
                                    │  - 200 conflicts          │
                                    │  - 3 din merge karne mein │
                                    │  - kisko pata nahi kaunsa │
                                    │    resolution sahi hai    │
                                    │  - QA ko 21 din ka code   │
                                    │    ek saath milta hai     │
                                    └───────────────────────────┘
```

Integration hell ka **asli** nuksaan conflicts nahi hai. Asli nuksaan ye hai:

1. **Debugging surface bahut bada ho jaata hai.** Agar test fail hua, to 205 commits mein se kaunsa? Ab bisect karo, 3 ghante jaayenge.
2. **QA ko 21 din ka change ek saath milta hai.** Tum regression suite chalate ho, 40 tests fail hote hain, aur tum ye distinguish hi nahi kar paate ki 40 alag-alag bugs hain ya ek root cause hai.
3. **Feedback loop 21 din ka ho gaya.** Dev A ne jo galti day 2 pe ki, use day 23 ko pata chali. Tab tak wo us code ka context bhool chuka hai, aur uske upar 38 aur commits ban chuke hain.
4. **Release date guess ban jaati hai.** "Integration mein kitna time lagega?" — koi nahi bata sakta, kyunki wo estimate hi nahi ho sakta.

CI in sab ko ek simple mechanism se maarta hai: **agar integration dard deta hai, to use itni baar karo ki wo dard hi na de.** Chhota merge = chhota conflict = chhota debugging surface.

Ek line mein: **CI feedback loop ko 21 din se 10 minute pe le aata hai.**

## 1.3 Kab

CI **hamesha**. Ek-developer project mein bhi. Kyunki:
- Ek developer bhi "works on my machine" ka shikaar hota hai (uske laptop pe env var set hai, CI pe nahi).
- Ek developer bhi purana code tod deta hai.

CI ka minimum viable version bhi (checkout + install + unit tests) zero se infinitely behtar hai.

## 1.4 Interview answer

> **Interview answer:**
> "Continuous Integration is the practice of merging every developer's work into a shared mainline at least once a day, with an automated build and test run on every merge. The tool — Jenkins, GitHub Actions — is just the enforcement mechanism; CI itself is the discipline of small, frequent integrations.
>
> The problem it solves is integration hell. Without CI, teams work on long-lived branches for weeks, and all the conflicts, incompatible assumptions and broken interfaces surface at once at the end. The real cost isn't the merge conflicts — it's that the debugging surface becomes enormous. If a test fails after merging twenty branches, you have hundreds of candidate commits and no cheap way to isolate the cause.
>
> CI attacks that by shrinking the integration unit. Small merges mean small conflicts, and a failure is attributable to one commit. From a QA point of view that's the whole value: CI converts a three-week feedback loop into a ten-minute one, and it changes my job from 'find bugs in a big pile of changes at the end' to 'catch a regression the moment it's introduced, while the author still has the context in their head'.
>
> A useful sanity check for whether a team really does CI is: are branches merged within a day or two, and is the main branch green almost all the time? If branches live for weeks, they have a CI *server*, not CI."

**Cross-question: "Agar developers roz merge karenge to half-finished features production mein chali jaayengi na?"**

> **Interview answer:**
> "That's the standard objection, and the answer is that you decouple *deploying* code from *releasing* a feature. You merge small, complete, safe increments of code, but the feature stays invisible to users until it's ready.
>
> There are three standard techniques. Feature flags — the code ships but is switched off, so it's dead code in production until the flag is enabled. Branch by abstraction — you introduce an interface, build the new implementation behind it, and only flip the wiring once the new path is proven. And keyhole or incremental delivery — you merge the backend and data layer first, which nothing calls yet, and merge the UI entry point last.
>
> As a QA this actually helps me, because I can test the new path in production behind a flag with the flag enabled only for internal accounts, before any real customer sees it. What it does add is a testing obligation: I have to test *both* states of the flag, and I have to make sure flag combinations are covered for the risky ones. Flags are technical debt with an expiry date — a team that never removes old flags ends up with an untestable combinatorial mess."

**Cross-question: "Tumhare project mein CI hai?"** — Ye tumse zaroor poochhenge. Jhooth mat bolo, structure ye rakho:

> **Interview answer:**
> "Partially, and I'm honest about the gap. The backend is Kotlin with Gradle and has a build that runs unit tests, with integration tests excluded by default behind a `-PrunIntegrationTests=true` flag, so they only run when explicitly asked for. My Playwright suite is currently triggered manually through a runner script that executes the suite and posts a summary to Slack.
>
> So the pieces exist — the suite is headless-capable, it's already reporting to Slack, it's already parameterised by environment — but the trigger is a human, not an event. What I've designed and am moving to is: lint plus unit tests on every pull request, a smoke subset on merge to main, and a sharded full regression on a nightly schedule with the same Slack notification I already produce. The suite was built to be CI-ready before it was put in CI, which is the right order — putting a flaky, environment-coupled suite into CI just teaches the team to ignore red builds."

---

# 2. CI vs Continuous Delivery vs Continuous Deployment

## 2.1 Kya hai

Teeno alag cheezein hain aur log inhe ghalat-malat use karte hain. "CD" ka matlab dono ho sakta hai — isliye interview mein hamesha **clarify** karo ki tum kis CD ki baat kar rahe ho. Ye clarification khud ek senior signal hai.

```
┌──────────────────────────────────────────────────────────────────────┐
│  CONTINUOUS INTEGRATION                                              │
│  commit ──> build ──> unit tests ──> merge to main                   │
│  Output: "code compiles aur tests pass hote hain"                    │
└──────────────────────────────────────────────────────────────────────┘
                              +
┌──────────────────────────────────────────────────────────────────────┐
│  CONTINUOUS DELIVERY                                                 │
│  ... ──> artifact banao ──> staging deploy ──> full test suite       │
│      ──> ARTIFACT PRODUCTION-READY HAI, BUTTON DABANE KA INTEZAAR    │
│  Output: "har commit release ho sakta hai" (human decide karta hai)  │
└──────────────────────────────────────────────────────────────────────┘
                              +
┌──────────────────────────────────────────────────────────────────────┐
│  CONTINUOUS DEPLOYMENT                                               │
│  ... ──> saare gates green ──> AUTOMATICALLY PRODUCTION              │
│  Output: "har green commit production mein chala jaata hai"          │
│  Koi human button nahi. Test suite HI release gate hai.              │
└──────────────────────────────────────────────────────────────────────┘
```

## 2.2 Comparison table

| Dimension | Continuous Integration | Continuous **Delivery** | Continuous **Deployment** |
|---|---|---|---|
| Kahan tak automate hai | Build + test | Build + test + deploy-to-staging + release-ready artifact | Build + test + **production deploy** |
| Production pe kaun bhejta hai | Koi nahi (scope se bahar) | **Insaan** — button dabata hai | **Pipeline** — automatic |
| Manual approval gate | N/A | Haan, ek deliberate gate | **Nahi** |
| Deploy frequency typical | N/A | Hafte mein / din mein, on-demand | Din mein 10-100 baar |
| Rollback ki importance | Low | Medium | **Critical** — automated hona chahiye |
| Test suite ki role | Safety net | Confidence builder | **RELEASE GATE — suite hi final judge hai** |
| Flaky test ka asar | Annoying | Deploy delay | **Ya to release ruk gaya, ya bug production mein** |
| Feature flags | Optional | Recommended | **Mandatory** |
| Monitoring/observability | Optional | Recommended | **Mandatory** — post-deploy verification hi safety hai |
| Kiske liye sahi | Sab | Regulated industries, enterprise, ERP jaisa domain | High-velocity SaaS, mature teams |

## 2.3 Kyun — QA ke liye ye farq kyun matter karta hai

Ye **sabse important paragraph** hai is section ka, aur interview mein yahi cheez tumhe alag karegi.

**Continuous Delivery mein tumhara test suite ek advisory hai.** Suite red hai? Ek insaan dekhta hai, sochta hai, "arre ye to flaky test hai", aur override karke deploy kar deta hai. Tumhari suite ka kaam hai *information dena*. Agar tumhari suite mein 5% flakiness hai to duniya nahi rukti — koi banda dekh lega.

**Continuous Deployment mein tumhara test suite HI release gate hai.** Koi insaan beech mein nahi hai. Iska seedha matlab:

1. **Tumhari suite ka coverage = tumhara production risk.** Jo path suite mein nahi hai, wo untested production mein jaayega. Coverage gap ab theoretical nahi, direct customer-facing risk hai.
2. **Flakiness ab organisational poison hai.** 200 tests, har ek 99.5% reliable — poori suite ka pass rate `0.995^200 = 37%`. Matlab 63% deployments jhoothe reason se blocked. Team 2 hafte mein hi "just re-run it" culture mein chali jaayegi, aur phir tumhari suite ka koi matlab nahi bacha.
3. **Test speed = deploy frequency.** Suite 2 ghante leti hai to din mein max 4 deploys. Suite 10 minute leti hai to 40. QA ki suite ki speed literally business ki velocity ban jaati hai.
4. **QA ka accountability badal jaata hai.** Continuous delivery mein bug nikla to "QA ne miss kiya, par release manager ne approve bhi kiya". Continuous deployment mein bug nikla to gate sirf tumhara tha. Isliye continuous deployment ke saath **automated rollback + strong post-deploy monitoring** non-negotiable hai — kyunki suite kabhi 100% nahi hogi, aur detection ka doosra layer chahiye.

> **[REAL]** Merlin AI ek **construction ERP** hai — purchase orders, bids, material/trade/custom line items. Ye domain continuous *deployment* ke liye natural fit nahi hai, aur interview mein ye bolna maturity dikhata hai. Reasons: financial data hai (PO amounts, vendor commitments), ERP customers ke apne internal approval cycles hote hain, aur ek galat schema migration purchase orders corrupt kar sakti hai jo silently discover ho — customer ko 3 hafte baad invoice mismatch pe pata chalega. Yahan **continuous delivery + explicit approval gate** sahi model hai: artifact har commit pe ready ho, par production push ek conscious decision ho.

## 2.4 Interview answer

> **Interview answer:**
> "They're three different things and people use 'CD' for two of them, so I'll define all three.
>
> Continuous Integration stops at merge: every commit triggers a build and a test run, and the outcome is 'the code integrates and the tests pass'. Continuous Delivery extends that to producing a deployable, verified artifact and deploying it automatically to lower environments — but the production push is a human decision. Continuous Deployment removes that human: if every gate is green, the change goes to production automatically.
>
> The distinction matters enormously to QA, because it changes what my test suite *is*. In continuous delivery my suite is advisory — a human looks at a red build, decides whether it's a real failure or a flaky test, and can override. In continuous deployment my suite *is* the release gate. There is no human judgement layer. That has three consequences I'd design around: coverage gaps become direct production risk rather than theoretical risk; flakiness becomes organisationally toxic, because two hundred tests at 99.5% reliability each gives you a 37% suite pass rate and the team will very quickly learn to ignore red; and suite runtime becomes the ceiling on deployment frequency, so test speed turns into a business metric, not a QA convenience.
>
> The other thing I'd say is that continuous deployment is only safe when it's paired with automated rollback, feature flags and real post-deploy monitoring. No suite is perfect, so you need a second detection layer and a fast, boring way to undo. On my current product — a construction ERP handling purchase orders and financial commitments — I'd argue for continuous delivery with an explicit approval gate rather than full continuous deployment, because a bad data migration there can corrupt financial records in a way that's silent for weeks."

**Cross-question: "To kya continuous deployment mein manual testing ki koi jagah nahi hai?"**

> **Interview answer:**
> "There's no place for manual testing *in the deployment path* — nothing can block the pipeline waiting for a person, by definition. But there's absolutely a place for manual testing around it, and it actually becomes more valuable, not less.
>
> It moves to three places. Before: exploratory testing on a feature-flagged build or a preview environment, before the flag is enabled for real users. Alongside: risk-based exploratory sessions on areas the automation can't judge — usability, visual correctness, weird real-world data, workflows that span days. And after: production verification with the flag enabled for internal accounts only, which is often the most realistic testing you can do.
>
> What genuinely disappears is manual *regression* testing, because you can't hand-check five hundred cases on every deploy when you deploy thirty times a day. That work has to be automated or dropped. So my job shifts from executing regression to designing the automated gate and spending my human time where a human is actually better than a script."

**Cross-question: "Ek company kaise decide kare ki wo delivery pe rahe ya deployment pe jaaye?"**

> **Interview answer:**
> "I'd look at four things. First, blast radius: how bad is a bad deploy, and how quickly is it detectable? A B2C feed app can auto-deploy; a payments ledger or an ERP that writes financial records probably shouldn't. Second, the maturity of the safety net: do you have automated rollback, feature flags, meaningful monitoring and alerting that would catch a problem in minutes rather than a customer reporting it in weeks? Third, suite health: flake rate under about one percent and a runtime short enough that people don't route around it. Fourth, regulatory and contractual constraints — some customers contractually require change notification windows, which alone rules out auto-deploy.
>
> And I'd point out you don't have to choose globally. A very reasonable pattern is continuous deployment for low-risk services — the frontend, internal tools, content — and continuous delivery with an approval gate for the services that touch money or the schema. The pipeline can be the same; only the final stage differs."

---

# 3. Trunk-Based Development vs GitFlow

## 3.1 Kya hai

Ye **branching strategy** ka sawaal hai, aur ye CI/CD se seedha juda hua hai — kyunki tumhari branching strategy decide karti hai ki CI possible bhi hai ya nahi.

### GitFlow

Vincent Driessen ne 2010 mein propose kiya. 5 tarah ki branches:

```
main      ────●────────────────────●────────────────●──────   (production, tagged)
               \                  /                /
hotfix          \            ●───/               ●/           (emergency fixes)
                 \          /                   /
release           \    ●───●──────────────     /              (stabilisation)
                   \  /                   \   /
develop   ──●───●───●─────●─────●──────●───●─●────────────    (integration branch)
             \     /       \   /        \ /
feature       ●───●         ●─●          ●                     (feature branches)
```

- `main` — sirf released code
- `develop` — integration branch
- `feature/*` — har feature ki apni branch, `develop` se katti hai
- `release/*` — release stabilise karne ke liye
- `hotfix/*` — production emergency, `main` se katti hai

### Trunk-Based Development (TBD)

Ek hi long-lived branch: `main` (trunk). Feature branches allowed hain par **1-2 din se zyada nahi jeeti**.

```
main   ──●──●──●──●──●──●──●──●──●──●──●──●──●──●──●──   (hamesha releasable)
          \ /    \ /       \ /          \ /
           ●      ●         ●            ●               (short-lived, <2 days)
```

Release branch banti hai par sirf release ke waqt, aur usme sirf cherry-picked fixes jaate hain.

## 3.2 Comparison table

| Dimension | GitFlow | Trunk-Based |
|---|---|---|
| Long-lived branches | `main`, `develop`, + release branches | Sirf `main` |
| Feature branch life | Hafte / mahine | **Ghante / max 2 din** |
| Merge conflicts | Bade aur dardnaak | Chhote aur trivial |
| Integration frequency | Feature complete hone pe | Din mein kai baar |
| CI ke saath compatibility | **Kharab** — "CI" sirf branch pe chalta hai, integration late hoti hai | **Excellent** — CI ka poora point hi yahi hai |
| Release cadence suit karta hai | Scheduled releases (quarterly, monthly), versioned products, mobile apps with app-store review | Continuous / on-demand release, SaaS |
| Feature flags ki zaroorat | Kam | **Zyada** — incomplete work trunk pe hai |
| Team discipline required | Kam | **Zyada** — main hamesha green rakhna padta hai |
| Kitni branch strategy overhead | Zyada (5 branch types, rules yaad rakho) | Bahut kam |
| Hotfix path | Dedicated hotfix branch | `main` se fix, phir release branch pe cherry-pick |
| DORA research verdict | Correlates with **lower** delivery performance | Correlates with **higher** delivery performance |

## 3.3 Kab kaunsa

**GitFlow tab sahi hai jab:**
- Tum **versioned software** ship karte ho jahan multiple versions simultaneously supported hain (v2.1 aur v3.0 dono maintain ho rahe hain)
- Mobile app hai jahan app-store review cycle hai, to release branch pe stabilisation ka time chahiye
- Regulated environment hai jahan release branch ka formal sign-off record chahiye

**Trunk-based tab sahi hai (aur ye default hona chahiye) jab:**
- Web app / SaaS hai, ek hi version live hai
- Tum CI/CD chahte ho (kyunki GitFlow mein feature branches long-lived hain, matlab integration continuous nahi hai — definition se)
- Team disciplined hai aur test coverage decent hai

**Practical reality:** Zyadatar teams **GitHub Flow** use karti hain — ye beech ka raasta hai. `main` + short-lived feature branches + PR review + merge. Ye technically trunk-based ka ek relaxed version hai. Section 31 mein detail hai.

> **[REAL]** Tumhare Merlin repo mein automation suite `main` pe hai aur tum short-lived branches use karte ho — ye effectively GitHub Flow hai. Interview mein bolne layak baat: **test automation repos ke liye trunk-based lagbhag hamesha sahi hai**, kyunki test code ka koi "version" ship nahi hota — hamesha latest hi chalta hai. Test suite ke liye GitFlow overhead pure waste hai.

## 3.4 Interview answer

> **Interview answer:**
> "GitFlow uses several long-lived branches — main, develop, plus release and hotfix branches — with feature branches that can live for weeks. Trunk-based development uses a single long-lived branch, main, and feature branches that live hours to at most a couple of days.
>
> The reason I care as a QA is that branching strategy determines whether CI is actually possible. If a feature branch lives for three weeks, you can run a build on that branch, but you are not doing continuous *integration* — integration is still deferred to the end, and so is the discovery of every conflict and every broken assumption. GitFlow with long-lived branches gives you a CI server without CI.
>
> Trunk-based is what the DORA research consistently associates with higher delivery performance, and it's what I'd default to for a web product. It does demand more discipline — main has to stay green, so you need a reliable pre-merge gate, and you need feature flags because incomplete work is sitting on the trunk. That's a QA responsibility: if I'm advocating trunk-based, I own making the pre-merge gate fast and trustworthy, otherwise the team just stops respecting it.
>
> GitFlow still earns its place where you genuinely maintain multiple released versions at once, or where there's an external release gate like an app-store review that needs a stabilisation branch. For a single-version SaaS product, it's mostly ceremony.
>
> For test automation repositories specifically I'd always use trunk-based, because test code has no released versions — you always want the latest suite running."

**Cross-question: "Trunk-based mein agar koi banda main tod de to poori team block ho jaayegi. Ye kaise handle karoge?"**

> **Interview answer:**
> "That risk is real and it's exactly why trunk-based needs stronger automation than GitFlow, not weaker.
>
> Four mitigations. First, a required pre-merge gate: branch protection so nothing merges without a green build, and a merge queue so the build runs against main-plus-your-change, not against a stale base — that closes the semantic-conflict gap where two PRs are each green but break when combined. Second, keep the pre-merge gate fast, under ten minutes, so people don't try to route around it. Third, an explicit 'stop the line' norm: a broken main is the team's top priority, and the default fix is to revert immediately and re-land properly rather than to fix forward under pressure. Fourth, keep changes small — a small change that breaks main is trivially revertible; a two-thousand-line change is not.
>
> And I'd note the comparison isn't 'trunk-based breaks main versus GitFlow doesn't'. GitFlow doesn't remove breakage, it defers and batches it into merge day, where it's harder to attribute and harder to revert."

---

# 4. Build Once, Deploy Many — sabse under-rated principle

## 4.1 Kya hai

**Ek hi artifact banao, aur wahi exact artifact har environment mein promote karo.**

```
                    ┌──────────────┐
   git commit ─────>│    BUILD     │──> artifact: myapp-a3f9c21.jar
   a3f9c21          │  (ek baar)   │    (immutable, checksummed)
                    └──────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        v                   v                   v
   ┌─────────┐        ┌──────────┐        ┌────────────┐
   │   DEV   │        │ STAGING  │        │ PRODUCTION │
   │ a3f9c21 │───────>│ a3f9c21  │───────>│  a3f9c21   │
   └─────────┘        └──────────┘        └────────────┘
   config: dev.env    config: stg.env     config: prod.env
        ^                   ^                   ^
        └───────────────────┴───────────────────┘
              SIRF CONFIG BADALTA HAI, BINARY NAHI
```

Anti-pattern (jo bahut common hai):

```
   git ──> build for dev  ──> deploy dev
   git ──> build for stg  ──> deploy staging      <-- 3 alag builds
   git ──> build for prod ──> deploy production
```

## 4.2 Kyun — aur ye QA ke liye kyun life-or-death hai

Agar tum har environment ke liye alag build karte ho, to **QA ne jo cheez test ki wo production mein jaane wali cheez hai hi nahi.** Tumne artifact X test kiya, production mein artifact Y gaya. X aur Y "same source se" bane hain — par same nahi hain.

Kya-kya alag ho sakta hai do builds mein, same commit se bhi:

| Kya badal sakta hai | Kaise |
|---|---|
| Transitive dependency versions | `^1.2.0` ne aaj `1.2.0` resolve kiya, kal `1.3.7`. Lockfile na ho to guaranteed drift. |
| Base image | `FROM python:3.11` — image kal rebuild hui, upstream patch aa gaya |
| Build-time env vars | `NODE_ENV=production` build mein minification/dead-code-elimination on, dev build mein off — **alag code path** |
| Compiler / toolchain version | Runner image update ho gaya, ab alag JDK minor |
| Timestamps / build metadata baked in | Cache keys, ETags badal jaate hain |
| Conditional compilation flags | Kotlin/Java mein `if (BuildConfig.DEBUG)` blocks |

Aur ye **sabse zyada khatarnak** isliye hai ki inme se koi bhi difference tumhare test se pakda **nahi** jaayega — kyunki tumne test kis pe kiya? Purane artifact pe.

Ek line mein: **agar tum per-environment rebuild karte ho, to tumhari saari QA activity technically invalid hai.** Tum kisi aisi cheez ke baare mein statement de rahe ho jo ship nahi ho rahi.

## 4.3 Kab / kaise implement karte hain

1. **Build stage sirf ek baar chalta hai** pipeline mein, sabse pehle.
2. Artifact **immutable** ho aur **commit SHA se tag** ho — `myapp:a3f9c21`, `myapp:latest` nahi. `latest` mutable hai, matlab tum nahi jaante kya deploy hua.
3. Artifact ek **registry / artifact store** mein jaaye (Docker registry, S3, Nexus, GitHub Packages).
4. Har deploy stage **wahi artifact download** kare, rebuild na kare.
5. **Config artifact ke bahar** ho — environment variables, config maps, secrets injected at runtime. Config artifact ke andar bake mat karo.
6. Pipeline mein **verify** karo: deployed version endpoint expose karo (`/version` → commit SHA), aur smoke test us SHA ko assert kare.

```yaml
# Simple version check — post-deploy verification ka sabse sasta form
- name: Verify deployed build is the one we tested
  run: |
    DEPLOYED=$(curl -sf https://api.merlinai.co/version | jq -r .commit)
    if [ "$DEPLOYED" != "${{ github.sha }}" ]; then
      echo "::error::Expected ${{ github.sha }} but ${DEPLOYED} is live"
      exit 1
    fi
```

> **[REAL]** Tumhare paas iska ek **perfect real-world example** hai jo isi principle ka cousin hai — aur interview mein ye story sunane layak hai.
>
> Tumhe `staging-api.merlinai.co` mila ek config file mein. Wo host **dead legacy** hai, sab kuchh 503 karta hai. Asli staging backend `staging-eks.merlinai.co` hai, aur wo sirf **deployed frontend ke JS bundle** se discover hua.
>
> Lesson exactly wahi hai jo "build once deploy many" sikhata hai: **config jo system ko describe karti hai, wo system ke sach se drift kar jaati hai. Sach hamesha deployed artifact mein hota hai.** Isliye verification hamesha deployed cheez ke against hoti hai — us document ke against nahi jo usko describe karta hai. Ye ek QA-specific insight hai aur interviewer ise pasand karega.

## 4.4 Interview answer

> **Interview answer:**
> "Build once, deploy many means you produce one immutable artifact from a commit, and promote that exact artifact through dev, staging and production. Only configuration changes between environments — it's injected at runtime, never baked into the build.
>
> The reason I care about this as a QA is blunt: if you rebuild per environment, everything I tested is invalid. I signed off on artifact A; production runs artifact B. They came from the same source, but they are not the same binary. Transitive dependencies can resolve differently, the base image can have moved, build-time flags like a production NODE_ENV change which code paths even exist after dead-code elimination. And none of those differences can be caught by my tests, because my tests ran against the artifact that isn't shipping.
>
> Practically I'd want four things: a single build stage early in the pipeline, artifacts tagged with the commit SHA rather than a mutable tag like `latest`, artifacts stored in a registry and pulled by each deploy stage rather than rebuilt, and a version endpoint that my smoke test asserts against, so the pipeline proves the thing running in production is the thing that passed the gates.
>
> I've been bitten by the general form of this problem. On my current project a config file referenced a staging API host that had been dead for a long time — it 503'd everything — and the real host was only discoverable by reading the deployed frontend's JavaScript bundle. The lesson generalises: configuration that describes a system drifts from the system. Always verify against the deployed artifact, not against the document that claims to describe it."

**Cross-question: "Agar config artifact ke bahar hai, to config ko kaun test karta hai?"**

> **Interview answer:**
> "That's the right follow-up, because moving config out of the artifact moves the risk rather than removing it — the classic outage is 'the build was fine, someone typed the wrong database URL in the production config'.
>
> I'd cover it three ways. First, config validation at startup: the application fails fast and loudly if a required variable is missing or malformed, rather than starting and failing on first request. That turns a config error into a failed deploy instead of a broken production. Second, config in version control — as code, reviewed in a PR like anything else, with secrets stored as references to a secret manager rather than values. Third, and most importantly, post-deploy smoke tests that exercise the config: hit a health endpoint that actually checks database and third-party connectivity, plus one real end-to-end request. That is the only thing that proves the config is right in that environment, because config is by definition environment-specific and can't be validated earlier.
>
> The general principle is that anything that varies per environment must be verified per environment, after deploy."

---

# 5. Pipeline Anatomy — har stage detail mein

## 5.1 Full pipeline diagram

Ye poora picture hai. Har team ke paas saare stages nahi hote, par **vocabulary yeh hai** — interview mein tumse expect kiya jaata hai ki tum yeh naam jaante ho aur bata sako ki kaun sa stage kis cheez ko rok raha hai.

```
                                  ┌─────────────────┐
                                  │   1. TRIGGER    │  push / PR / cron / manual
                                  └────────┬────────┘
                                           v
                                  ┌─────────────────┐
                                  │   2. CHECKOUT   │  git clone (shallow, depth=1)
                                  └────────┬────────┘
                                           v
                                  ┌─────────────────┐
                                  │  3. DEP CACHE   │  pip / npm / gradle restore
                                  │   + INSTALL     │  ~30s vs ~3min
                                  └────────┬────────┘
                                           v
        ┌──────────────────────────────────┴──────────────────────────────┐
        │            4. STATIC ANALYSIS  (parallel, ~1-2 min)             │
        │   lint (ruff)  │  format (black --check)  │  types (mypy)       │
        │   secret scan (trufflehog/gitleaks)  │  SAST (bandit/semgrep)   │
        └──────────────────────────────────┬──────────────────────────────┘
                                           v
                                  ┌─────────────────┐
                                  │    5. BUILD     │  compile / bundle
                                  │                 │  EK BAAR. SIRF EK BAAR.
                                  └────────┬────────┘
                                           v
                                  ┌─────────────────┐
                                  │ 6. ARTIFACT     │  docker build + push
                                  │    + STORE      │  tag = commit SHA
                                  └────────┬────────┘
                                           v
                                  ┌─────────────────┐
                                  │ 7. UNIT TESTS   │  <5 min, no I/O
                                  │   + COVERAGE    │  GATE: coverage >= X%
                                  └────────┬────────┘
                                           v
                                  ┌─────────────────┐
                                  │ 8. DEPLOY TO    │  same artifact,
                                  │    STAGING      │  staging config
                                  └────────┬────────┘
                                           v
                                  ┌─────────────────┐
                                  │  9. SMOKE       │  <3 min, ~10 tests
                                  │  (post-deploy)  │  "kya app zinda hai?"
                                  └────────┬────────┘
                                           v
                     ┌─────────────────────┴─────────────────────┐
                     │        10. INTEGRATION / API TESTS        │  5-15 min
                     │        (real DB, real services)           │
                     └─────────────────────┬─────────────────────┘
                                           v
                     ┌─────────────────────┴─────────────────────┐
                     │   11. UI REGRESSION  (sharded, parallel)  │  15-40 min
                     │   shard1 | shard2 | shard3 | shard4       │
                     └─────────────────────┬─────────────────────┘
                                           v
              ┌────────────────────────────┴────────────────────────────┐
              │      12. SECURITY SCAN        │   13. PERFORMANCE       │
              │  DAST / dep audit / licence   │   k6 / locust baseline  │
              └────────────────────────────┬────────────────────────────┘
                                           v
                                  ┌─────────────────┐
                                  │ 14. APPROVAL    │  <-- HUMAN GATE
                                  │     GATE        │      (continuous delivery)
                                  └────────┬────────┘
                                           v
                                  ┌─────────────────┐
                                  │ 15. PRODUCTION  │  blue-green / canary /
                                  │     DEPLOY      │  rolling
                                  └────────┬────────┘
                                           v
                                  ┌─────────────────┐
                                  │ 16. POST-DEPLOY │  smoke on prod
                                  │  VERIFICATION   │  + /version SHA check
                                  └────────┬────────┘
                                           v
                                  ┌─────────────────┐
                                  │ 17. MONITORING  │  error rate, latency,
                                  │  + AUTO-ROLLBACK│  business metrics
                                  └─────────────────┘
                                     |  agar SLO break --> automatic rollback
                                     v
                             ┌───────────────┐
                             │   ROLLBACK    │
                             └───────────────┘
```

## 5.2 Har stage — kya, kyun, kitna time

### 1. Trigger

**Kya:** Pipeline kis event pe chalu hoga.

| Trigger | Kab | Typical use |
|---|---|---|
| `push` to branch | Har commit | Build + fast tests |
| `pull_request` | PR khulne/update hone pe | **Pre-merge gate** — sabse important QA gate |
| `schedule` (cron) | Fixed time | Nightly full regression, security scan |
| `workflow_dispatch` | Manual button | On-demand run, env chuno |
| `repository_dispatch` / webhook | External event | Deploy hone pe test trigger |
| `tag push` | Release tag | Production release pipeline |

**QA angle:** Sabse common galti — poori suite har push pe chalana. Nateeja: pipeline 40 min ka, developers frustrated, wo PR khole bina direct merge karne ke tareeke dhoondhne lagte hain. Trigger design hi pipeline design ka aadha hissa hai.

### 2. Checkout

**Kya:** Repo code runner pe laana.

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 1        # shallow clone — sirf latest commit. Fast.
    # fetch-depth: 0      # poori history — bisect/blame/changed-file detection ke liye chahiye
```

**Gotcha:** Agar tumhe "changed files since main" nikalna hai (path-based test selection ke liye) to `fetch-depth: 0` chahiye, warna base commit hi nahi milega.

### 3. Dependency cache + install

**Kya:** `pip install`, `npm ci`, Gradle dependencies. Ye stage bina cache ke pipeline ka sabse bada silent time-waster hai.

**Numbers jo yaad rakhne layak hain:** Playwright browsers download ≈ 300-400MB, 60-120 seconds. Cache hit pe ≈ 10-15 seconds. 20 runs/day pe ye ~30 min/day bacha deta hai.

### 4. Static analysis

**Kya:** Code ko run kiye bina check karna. Char cheezein:

| Check | Tool (Python) | Kya pakadta hai | Time |
|---|---|---|---|
| Lint | `ruff` / `flake8` / `pylint` | Unused imports, bugs-prone patterns, complexity | 5-20s |
| Format | `black --check` / `ruff format --check` | Style inconsistency | 3-5s |
| Type check | `mypy` / `pyright` | Type errors, `None` misuse | 20-60s |
| Secret scan | `gitleaks` / `trufflehog` | **Committed credentials** | 10-30s |
| SAST | `bandit` / `semgrep` | Insecure patterns (SQL injection, weak crypto) | 30-90s |

**Kyun ye sabse pehle:** Sabse sasta feedback. Agar `black` fail ho raha hai to build karke 20 min waste karne ka koi matlab nahi.

> **[REAL]** Ye stage seedha tumhari **security finding** se judta hai. Tumne Merlin backend ke `src/test/resources/application-test.properties` mein **MongoDB Atlas credentials aur QuickBooks client secret plaintext mein committed** paaye. Ek `gitleaks` step pipeline mein hota to wo commit **PR pe hi block** ho jaata — mahino baad manual audit se pakadne ki zaroorat hi nahi padti. Ye tumhari sabse strong interview story hai kyunki isme **finding + preventive control ka design** dono hain. Detail Section 14 mein.

### 5. Build

**Kya:** Source ko runnable form mein convert karna — compile, bundle, minify.

**Iron rule:** **Pipeline mein sirf ek baar.** Section 4 dekho.

### 6. Artifact creation + storage

**Kya:** Build output ko immutable package banake store karna — Docker image, JAR, wheel, tarball.

```bash
# Tag = commit SHA. NEVER just `latest`.
docker build -t registry.merlinai.co/api:${GIT_SHA} .
docker push  registry.merlinai.co/api:${GIT_SHA}
# `latest` ko additionally point kar sakte ho, par deploy hamesha SHA se
docker tag registry.merlinai.co/api:${GIT_SHA} registry.merlinai.co/api:latest
```

### 7. Unit tests + coverage

**Kya:** Sabse tez tests. Koi network nahi, koi DB nahi, koi file I/O nahi. Sirf pure logic.

**Time budget: 5 minute se kam.** Agar zyada hai to wo unit tests nahi hain — kahin na kahin I/O ghusa hua hai.

**Gate:** Coverage threshold. Par dhyan se — Section 7 mein detail hai ki coverage gate ka sahi form kya hai.

### 8. Deploy to environment

**Kya:** Wahi artifact staging pe.

**QA angle:** Deploy stage khud ek test hai. Agar deploy fail hota hai to wo bhi ek bug hai — infrastructure bug. Aur deploy step **idempotent** hona chahiye: dobara chalao to wahi result mile.

### 9. Smoke tests (post-deploy)

**Kya:** 5-15 tests jo bas ye batate hain ki **app zinda hai aur basic flows toote nahi**.

Smoke ka scope:
- Health endpoint 200 deta hai
- Login ho jaata hai
- Har major module ka landing page load hota hai
- Ek create + ek read operation kaam karta hai
- `/version` endpoint deployed SHA return karta hai

**Time budget: 3 minute se kam.**

**Rule:** Smoke fail = **turant rollback**, aage ke stages chalane ka koi matlab nahi. Agar login hi nahi ho raha to 400 UI tests chalane se sirf 400 red results milenge, information zero.

> **[REAL]** Tumhare 6 E2E flows mein se **smoke subset** aise banega: har flow ka pehla 3-4 step. Login → dashboard load → PO create page khulta hai → ek draft PO save hota hai. Ye ~3 min mein ho jaayega vs poora suite jo 6 flows × 12-15 steps hai. Interview mein bolo: *"I designed smoke as the first 3-4 steps of each of my six flows, so it exercises auth, navigation and one write per module — it's a breadth check, not a depth check."*

### 10. Integration / API tests

**Kya:** Components ke beech ka contract check. Real DB, real service calls, par UI ke bina.

**Kyun ye UI se pehle:** Ye **tez** hain (koi browser nahi) aur **stable** hain (koi selector/timing flakiness nahi). Agar API level pe hi PO creation toot raha hai, to UI test chalane ka koi fayda nahi — wo bhi fail hoga par tumhe extra 20 min ke baad pata chalega, aur failure message bhi kam useful hoga ("button click ke baad element nahi mila" vs "POST /purchase-orders returned 500").

> **[REAL]** Merlin backend Kotlin/Gradle hai aur integration tests **default se excluded** hain, `-PrunIntegrationTests=true` se chalte hain. Pipeline design: PR pe unit tests only (fast), merge-to-main pe integration tests bhi. Ye pattern bilkul standard hai aur tumhe iska rationale bolna aana chahiye — *"they're excluded by default because they need real infrastructure and would slow the developer's local loop; the pipeline opts in explicitly at the stage where that infrastructure exists."*

### 11. UI regression

**Kya:** Poori browser-based E2E suite. Sabse slow, sabse flaky, sabse zyada business-value-confirming.

**Time budget:** Sharding ke baad 15-20 min. Bina sharding ke jitna bhi lage.

### 12. Security scan

| Type | Kya | Tool |
|---|---|---|
| SCA / dependency audit | Known CVEs in dependencies | `pip-audit`, Dependabot, Snyk |
| DAST | Running app pe attack simulation | OWASP ZAP |
| Container scan | Base image vulnerabilities | `trivy`, `grype` |
| Licence scan | GPL jaisi licence ghus gayi? | `pip-licenses`, FOSSA |

### 13. Performance

**Kya:** Load/latency baseline ke against comparison.

**Ye gate kaise banaao (important nuance):** Absolute threshold ("p95 < 500ms") environment noise ki wajah se flaky hota hai. Better: **regression detection** — "p95 pichhle baseline se 20% se zyada nahi bigda hona chahiye". Aur load test ko dedicated environment pe chalao, shared staging pe nahi.

### 14. Approval gate

**Kya:** Human button. Continuous delivery ki definition ka core.

Isme jo dikhna chahiye: kya deploy ho raha hai (SHA + changelog), kaun se gates pass hue, kaun se skip/override hue, rollback plan kya hai.

### 15. Production deploy

Section 8 (deployment strategies) mein detail.

### 16. Post-deploy verification

**Kya:** Production pe smoke. Ye **alag** hai staging smoke se.

Production smoke design karne ke rules:
- **Read-only prefer karo** jahan possible ho
- Agar write karna hai to **dedicated test account** use karo jo business reports se excluded ho
- Cleanup guarantee karo (`finally` block)
- **Version assert karo** — `/version` endpoint se SHA match

> **[REAL]** Tumhari suite **production (app.merlinai.co) ke against** chalti hai. Ye interview mein zaroor discuss hoga, aur ise **defend karne ka tareeka** ye hai — apologetically mat bolo, thoughtfully bolo:
>
> *"My suite currently runs against production. That wasn't the ideal choice, it was the available one — staging was unreliable and at one point I lost time to a staging host that was documented in config but had been dead for months. Running against production has one genuine advantage: it's the only environment whose data and integrations are real, so it catches config and integration problems no other environment can. But it forces discipline: dedicated test accounts, data that's clearly marked as test data and cleaned up, no destructive operations against real records, and awareness that my test traffic shows up in analytics. The direction I'm pushing is smoke against production for post-deploy verification, and the full regression against a properly maintained staging."*

### 17. Monitoring + auto-rollback

**Kya:** Deploy ke baad system metrics dekhna aur SLO break hone pe automatically pichhle version pe wapas jaana.

Metrics jo dekhne chahiye: error rate (5xx), latency p95/p99, throughput, aur **business metrics** — POs created per hour, login success rate. Business metric sabse zyada valuable hai kyunki wo woh cheezein pakadta hai jo technically "healthy" dikhti hain par actually toot chuki hain (button dikh raha hai, click pe kuch nahi hota — error rate normal, PO creation zero).

**Ye QA ka kaam kyun hai:** Monitoring **production mein testing** hai. Ek senior SDET ye argue karta hai ki alerts bhi test cases hain, aur unka bhi coverage hona chahiye.

---

# 6. QA ka role har stage pe

## 6.1 The table

| # | Stage | Kya test chalta hai | Time budget | Fail hone pe kya hota hai | Kaun theek karta hai |
|---|---|---|---|---|---|
| 1 | Pre-commit (local) | Lint, format, fast unit subset | **< 10s** | Commit hi nahi banta | Developer, turant |
| 2 | Static analysis | Lint, types, **secret scan**, SAST | **< 2 min** | PR blocked, merge nahi | Developer |
| 3 | Unit tests | Pure logic, mocked boundaries | **< 5 min** | PR blocked | Developer |
| 4 | Build | Compile / bundle | < 5 min | PR blocked | Developer |
| 5 | Contract / API tests | Provider-consumer contracts | < 5 min | PR blocked | Dev + QA |
| 6 | Deploy to staging | (deploy khud ek test hai) | < 5 min | Deploy fail, rollback | DevOps |
| 7 | **Smoke** | 5-15 critical-path tests | **< 3 min** | **Turant rollback**, aage kuch nahi chalta | QA triage, dev fix |
| 8 | Integration tests | Real DB, real services | < 15 min | Deploy hold, investigate | QA + dev |
| 9 | **UI regression** | Poori E2E suite (sharded) | < 20 min | Release blocked (gate ke hisaab se) | **QA triage — flaky vs real** |
| 10 | Security scan | DAST, dep audit, container scan | < 10 min | Severity pe depend — critical = block | Security + dev |
| 11 | Performance | Load test vs baseline | < 15 min | Regression > threshold = block | Perf/QA |
| 12 | Approval gate | Human review of evidence | Human time | Deploy nahi hota | Release manager |
| 13 | Post-deploy smoke | Production critical path | **< 5 min** | **Auto-rollback trigger** | On-call |
| 14 | Monitoring | Alerts, SLO, business metrics | Continuous | Auto-rollback ya page on-call | On-call + QA |

## 6.2 Fast stages ko fast rakhna kyun zaroori hai

Ye ek **behavioural** baat hai, technical nahi — aur isiliye senior-level answer hai.

Test pyramid ka asli reason "correctness" nahi hai. Asli reason **feedback latency** hai. Feedback latency badhne pe kya hota hai, sequence dekho:

```
Pipeline 5 min hai:
  Dev PR kholta hai --> chai peeta hai --> result dekhta hai --> fix karta hai
  Context poora dimaag mein hai. Fix 2 min mein.

Pipeline 45 min hai:
  Dev PR kholta hai --> doosre kaam pe switch karta hai --> 45 min baad notification
  --> context switch back --> "maine kya kiya tha isme?" --> 20 min lagte hain
  --> ya wo notification hi ignore kar deta hai
```

Aur phir stage 3 aata hai, jo sabse khatarnak hai:

```
Pipeline 45 min hai aur 30% baar flaky fail hota hai:
  Dev seekh jaata hai ki red build ka matlab kuchh nahi
  --> "just re-run it" default response ban jaata hai
  --> ab red build ki koi information value nahi bachi
  --> gate technically maujood hai, practically dead hai
```

Iska naam **"routing around the gate"** hai. Developers system ko chakma dene ki koshish nahi kar rahe — wo apna kaam karne ki koshish kar rahe hain aur gate unke raaste mein hai. Agar tum unhe slow gate doge to wo:
- Chhote PRs ki jagah bade PRs banayenge (kam pipeline runs)
- `[skip ci]` use karenge
- Admin merge maangenge
- Ya bas red build ignore karna seekh jaayenge

**Isliye pipeline speed ek QA responsibility hai, DevOps ki nahi.** Ye line interview mein bolna — ye tumhe alag kar degi.

Practical rule of thumb jo bolne layak hai:

| Gate | Target | Absolute max |
|---|---|---|
| Pre-commit hook | 5s | 15s |
| PR gate (total) | 5 min | 10 min |
| Merge-to-main gate | 15 min | 20 min |
| Nightly full regression | 45 min | 90 min |

## 6.3 Interview answer

> **Interview answer:**
> "I think about QA's role as being spread across the whole pipeline rather than being one stage at the end, and each stage has a different job with a different time budget.
>
> The earliest stages — pre-commit hooks and static analysis — are about cheap, deterministic feedback in seconds: lint, formatting, type checks, and a secret scan, which I'd argue is non-negotiable. Then unit tests under five minutes as the pull-request gate. After deploy, smoke tests under three minutes answer one question: is the application alive? If smoke fails, you roll back immediately and don't run anything else, because running four hundred UI tests against a broken login gives you four hundred red results and zero information. Then integration and API tests, which are fast and stable because there's no browser, and only then the UI regression suite, sharded to keep it under twenty minutes. Finally post-deploy verification against production and continuous monitoring, which I treat as testing too — alerts are test cases that run forever.
>
> The principle I'd emphasise is that the early stages must stay fast, and that's a QA responsibility, not just a DevOps one. If a pull-request gate takes forty-five minutes, developers don't wait — they context-switch, and the feedback arrives when the change is no longer in their head. Worse, if a slow gate is also flaky, people learn that red doesn't mean broken, and the default response becomes 'just re-run it'. At that point the gate still exists on paper but has no information value. So when I add a test to a gate I ask not only 'does this catch bugs' but 'what does this cost every developer, every day, and will they route around it'."

**Cross-question: "Agar test slow hai par valuable hai to?"**

> **Interview answer:**
> "Then it belongs in the pipeline, just not in the fast gate. Value and placement are separate decisions.
>
> My options in order: move it later — run it on merge to main or nightly rather than on every PR, so it still gates the release but not every push. Run it selectively — trigger the expensive suite only when files in the relevant area changed. Shard it — many slow suites are slow because they're serial, not because any test is slow; splitting across parallel runners is the cheapest big win. Or push it down the pyramid — if a slow end-to-end test is really verifying a calculation, that calculation deserves a unit test, and the end-to-end test can shrink to verifying the workflow rather than the arithmetic.
>
> And I'd be willing to say the unpopular thing: some slow tests aren't worth their cost. If a fifteen-minute suite has never caught a real bug in a year, it's a tax, not a safety net. I'd measure that — track which tests have ever failed for a real reason — and delete the ones that haven't earned their place."

---

# 7. Quality Gates

## 7.1 Kya hai

**Quality gate** = ek automated pass/fail criterion jo pipeline ko aage badhne se rok deta hai jab tak condition meet na ho.

Gate ke 3 hisse hote hain:
1. **Metric** — kya maapa jaa raha hai (test pass rate, coverage, vulnerability count)
2. **Threshold** — kis value pe fail
3. **Action** — fail hone pe kya (block, warn, notify)

## 7.2 Kyun

Bina gate ke, "quality" ek opinion hai — "mujhe lagta hai ye theek hai". Gate ke saath, quality ek **precondition** hai. Iska sabse bada fayda political hai: gate impersonal hai. Tumhe kisi ko manually rokna nahi padta ("main is release ko approve nahi karunga") — system rokta hai, aur bahas criteria pe hoti hai, insaan pe nahi.

## 7.3 Typical gates with thresholds

| Gate | Metric | Typical threshold | Kahan | Block ya warn |
|---|---|---|---|---|
| Lint | Errors | 0 | PR | Block |
| Format | Diff | 0 | PR | Block |
| Type check | Errors | 0 (naye code pe) | PR | Block |
| **Secret scan** | Secrets found | **0** | PR | **Block, hamesha** |
| Unit tests | Pass rate | 100% | PR | Block |
| **Coverage — new code** | Line coverage of diff | **>= 80%** | PR | Block |
| Coverage — overall | Total line coverage | **Ghata nahi hona chahiye** | PR | Block |
| Smoke | Pass rate | 100% | Post-deploy | Block + rollback |
| Critical-path E2E | Pass rate (P0 tests) | 100% | Pre-release | Block |
| Full regression | Pass rate | >= 98% + koi P0 fail nahi | Pre-release | Block |
| **Flake rate** | Flaky tests / total | < 1% | Weekly trend | Warn + track |
| Dependency CVEs | Critical/High count | 0 critical | PR | Block critical, warn high |
| Container scan | Critical CVEs | 0 | Build | Block |
| Performance | p95 latency vs baseline | < +20% regression | Pre-release | Block |
| Bundle size | Size delta | < +5% | PR | Warn |
| Accessibility | Critical a11y violations | 0 new | PR | Block new, warn existing |

### Coverage gate ka sahi design — ye nuance senior signal hai

**Galat gate:** "Overall coverage >= 80%".

Problem: legacy codebase 45% pe hai. Gate din 1 se red hai. Team gate disable kar deti hai. Ya phir log getters/setters ke liye meaningless tests likhte hain sirf number badhane ko.

**Sahi gate — do parts:**
1. **Diff coverage / patch coverage: naye ya badle hue lines ka >= 80% covered ho.** Ye achievable hai, fair hai, aur codebase ko dheere-dheere upar le jaata hai.
2. **Ratchet: overall coverage kabhi ghatna nahi chahiye.** Agar 45% hai to 44.9% pe fail.

Ye combination legacy code ko punish nahi karta par naya untested code aane nahi deta. Interview mein ye bolna bahut achha impression banata hai.

```yaml
# diff-cover example — sirf PR mein badle hue lines ka coverage check
- name: Diff coverage gate
  run: |
    pytest --cov=src --cov-report=xml
    diff-cover coverage.xml --compare-branch=origin/main --fail-under=80
```

## 7.4 Break-glass override policy

**Kya:** Ek documented, audited tareeka gate ko bypass karne ka — jab genuinely zaroori ho.

**Kyun ye chahiye:** Kyunki emergencies hoti hain. Production down hai, ek-line fix hai, aur regression suite 40 min leti hai. Agar koi legitimate override path nahi hoga to log **illegitimate** path banayenge — admin merge, gate hataana, force push. Aur wo untracked hoga.

**Achhi break-glass policy ke 6 elements:**

| Element | Kya |
|---|---|
| **Authorisation** | Kaun override kar sakta hai — naam se defined (eng lead, on-call). "Koi bhi" nahi. |
| **Justification** | Likhit reason mandatory. Free text, par mandatory. |
| **Audit trail** | Automatic — kaun, kab, kaunsa gate, kya reason. Immutable log. |
| **Notification** | Team channel pe automatic post. Chupke se nahi ho sakta. |
| **Time limit** | Override us ek deploy ke liye hai, permanent nahi. |
| **Follow-up ticket** | Automatically banta hai. Skipped gate 24-48h mein satisfy hona chahiye. |

```yaml
# GitHub Actions mein break-glass — workflow_dispatch input se
on:
  workflow_dispatch:
    inputs:
      skip_regression:
        description: 'EMERGENCY ONLY: skip full regression suite'
        type: boolean
        default: false
      justification:
        description: 'Required: why are you skipping the gate?'
        required: true
        type: string

jobs:
  regression:
    if: ${{ !inputs.skip_regression }}
    runs-on: ubuntu-latest
    steps: [ ... ]

  audit_override:
    if: ${{ inputs.skip_regression }}
    runs-on: ubuntu-latest
    steps:
      - name: Record and announce the override
        run: |
          echo "::warning::REGRESSION GATE SKIPPED by ${{ github.actor }}"
          echo "Reason: ${{ inputs.justification }}"
      - name: Notify team
        run: |
          curl -X POST -H 'Content-type: application/json' \
            --data "{\"text\":\":rotating_light: Regression gate SKIPPED by ${{ github.actor }} on ${{ github.sha }}. Reason: ${{ inputs.justification }}\"}" \
            ${{ secrets.SLACK_WEBHOOK_URL }}
      - name: Open follow-up issue
        run: |
          gh issue create \
            --title "Follow-up: regression skipped on ${{ github.sha }}" \
            --body "Gate bypassed by ${{ github.actor }}. Reason: ${{ inputs.justification }}. Run full regression within 24h." \
            --label "gate-override"
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## 7.5 "Jis gate ko sab ignore karte hain, wo koi gate na hone se bura hai"

Ye is section ki sabse important baat hai.

Ek ignored gate **koi gate na hone se bura** hai, teen reasons se:

1. **Wo jhoothi safety deti hai.** Management sochti hai "hamare paas coverage gate hai, quality controlled hai". Reality: sab log override karte hain. Ab risk **invisible** ho gaya — pehle sabko pata tha ki gate nahi hai, ab sabko lagta hai hai.
2. **Wo baaki saare gates ki credibility kha jaati hai.** Ek baar team ne seekh liya ki "red build ka matlab kuchh nahi", wo discrimination nahi karti — asli failure bhi ignore hoti hai. Ek flaky gate poore quality system ka signal-to-noise ratio girata hai.
3. **Wo time kha rahi hai bina value diye.** 15 min × 20 runs/day = 5 ghante compute daily, aur har developer ka wait time, ek aisi cheez ke liye jiska result koi nahi dekhta.

**To kya karo jab gate consistently ignore ho rahi hai?** Do options hain — teesra nahi hai:
- **Gate ko trustworthy banao** (flakiness theek karo, threshold realistic karo, tez karo), ya
- **Gate hata do** aur risk ko openly acknowledge karo.

Beech ka raasta — "gate rehne do, log override karte rahenge" — sabse bura outcome hai.

## 7.6 Interview answer

> **Interview answer:**
> "A quality gate is an automated pass/fail criterion that stops the pipeline unless a condition is met. It has three parts: a metric, a threshold, and an action — block, or warn.
>
> The gates I'd consider baseline are: zero lint and type errors, zero secrets detected, a hundred percent unit test pass rate, and diff coverage above eighty percent on changed lines. Then post-deploy, a hundred percent smoke pass rate with automatic rollback on failure, and for the release gate, all P0 end-to-end tests passing.
>
> The design detail I care most about is coverage. Gating on total coverage is usually a mistake — on a legacy codebase it's red from day one, so the team either disables it or writes meaningless tests for getters to pump the number. What works is a two-part gate: diff coverage on new and changed lines above a threshold, plus a ratchet so total coverage can never decrease. That's fair to legacy code but stops new untested code entering.
>
> Every gate needs a documented break-glass path, because emergencies happen and if there's no legitimate override people invent illegitimate ones like admin-merging or disabling the check. A good override is named-authorised, requires a written justification, is automatically logged and announced in the team channel so it can't be quiet, and creates a follow-up ticket so the skipped verification actually gets done within a day.
>
> And the thing I'd stress most: a gate that everyone ignores is worse than no gate. It creates false confidence at the management level while providing no protection, and it erodes trust in every other gate — once people learn that red doesn't mean broken, they stop distinguishing real failures too. So when I see a gate being routinely overridden, there are only two honest responses: fix it so it's trustworthy, or remove it and be explicit about the risk. Leaving it there to be ignored is the worst option."

**Cross-question: "Coverage 100% hone se kya code bug-free ho jaata hai?"**

> **Interview answer:**
> "No, and that's the main reason I'm careful about how coverage is used. Coverage measures which lines *executed* during the test run, not whether anything was meaningfully asserted. You can get a hundred percent line coverage with tests that have no assertions at all — the code ran, so it counts.
>
> It also can't see what isn't there. Coverage says nothing about missing error handling, missing validation, or a requirement that was never implemented. And line coverage in particular misses branch and condition combinations — a compound `if` can be fully line-covered by a single case.
>
> So I treat coverage as a *negative* signal, not a positive one. Low coverage reliably tells me something is untested and I should look. High coverage doesn't tell me the code is well tested. If I wanted a real confidence signal I'd look at mutation testing — deliberately introducing faults and checking whether tests catch them — which measures assertion quality rather than execution. It's expensive, so I'd run it on the highest-risk modules, not everywhere. On an ERP that would be the pricing and totals calculations, where a wrong number is worse than a crash because nothing alerts on it."

**Cross-question: "Tumhare pipeline mein regression suite red hai release ke din. Tum release rokoge?"**

> **Interview answer:**
> "My answer depends on triage, not on the colour. Red is a prompt to investigate, not an automatic decision, so the first thing I do is spend fifteen minutes classifying the failures rather than debating the release.
>
> I'd sort them into three buckets. Real regressions in critical paths — for my product that's anything touching purchase order or bid creation and totals. Real but low-impact issues in secondary flows. And test-side failures — flakiness, stale test data, environment problems.
>
> If there's a genuine P0 regression, I block, and I'd hold that line: shipping a known broken checkout path to avoid a date is a bad trade every time. If the failures are all test-side, I say so explicitly, give the evidence — a passing re-run plus the specific reason, not just 'it's flaky' — and let the release proceed, and I raise a ticket to fix those tests, because unexplained flakiness is a debt that compounds.
>
> The important part is what I communicate. I'd never say 'the suite is red so we can't ship' and I'd never say 'it's probably flaky, ship it'. I'd say: 'Twelve failures. Nine are a known selector issue in the vendor picker, here's the passing re-run. Two are stale test data. One is real — trade-item purchase orders are rounding the tax line incorrectly, and it affects the total the customer sees. That last one is my recommendation to block on.' That gives the decision-maker the actual risk, and they can make an informed call. My job is to make the risk legible, not to be the person who says no."

---

# 8. Deployment Strategies

## 8.1 Kyun ye QA ka sawaal hai

Log sochte hain deployment strategy DevOps ka topic hai. Galat. **Deployment strategy decide karti hai ki tum kya test kar sakte ho, kab test kar sakte ho, aur kaunsa failure mode possible hai.**

Rolling deployment mein ek aisa bug possible hai jo blue-green mein possible hi nahi (mixed-version). Canary mein tumhe production mein testing karni hi padegi. Feature flags mein tumhara test matrix double ho jaata hai. Isliye ye QA ka core topic hai.

## 8.2 Comparison table (pehle overview, phir detail)

| Strategy | Kaise kaam karta hai | Rollback speed | Extra infra cost | Mixed versions live? | QA ka main focus |
|---|---|---|---|---|---|
| **Recreate** | Purana band, naya chalu | Slow (redeploy) | None | Nahi | Downtime window |
| **Rolling** | Instance-by-instance replace | Medium (roll back through) | None | **HAAN** | **Backward compatibility** |
| **Blue-Green** | Do full envs, traffic switch | **Seconds** (switch back) | **2x** | Nahi (switch instant) | Pre-switch full verification |
| **Canary** | Chhote % traffic naye version pe | Fast (traffic 0% karo) | Small | **HAAN** | **Metric comparison, prod monitoring** |
| **Feature flag** | Code deployed, feature off | **Instant** (flag off) | None | Nahi (ek binary) | **Dono flag states test karo** |
| **Shadow / dark** | Prod traffic duplicate, response discard | N/A (user-facing nahi) | ~2x compute | N/A | **Response diffing, side-effect safety** |

---

## 8.3 Recreate (baseline)

**Kaise:** Saare purane instances band karo, naye chalu karo.

```
  v1 v1 v1        (down)        v2 v2 v2
  ─────────>  ────────────>  ─────────>
                DOWNTIME
```

**Kab:** Sirf tab jab downtime acceptable ho — internal tools, maintenance windows, ya jab schema change itna breaking ho ki mixed versions possible hi nahi.

**QA:** Maintenance window ke andar full smoke. Aur specifically **startup** test karo — recreate mein sab kuch ek saath boot hota hai, to cold-start / connection-pool-storm problems yahin dikhte hain.

---

## 8.4 Rolling deployment

**Kaise:** Ek-ek (ya batch-batch) instance ko naye version se replace karo. Load balancer purane instance ko drain karta hai, naya health check pass karta hai, phir agla.

```
  Step 0:   [v1] [v1] [v1] [v1]
  Step 1:   [v2] [v1] [v1] [v1]    <-- MIXED VERSION WINDOW starts
  Step 2:   [v2] [v2] [v1] [v1]    <-- dono versions live traffic le rahe hain
  Step 3:   [v2] [v2] [v2] [v1]
  Step 4:   [v2] [v2] [v2] [v2]    <-- window ends
```

**Rollback speed:** Medium. Rollback bhi rolling hota hai, matlab utna hi time. Aur agar aadhe raaste pe fail hua to tum mixed state mein hi phanse ho.

### Mixed-version window — ye rolling ka SABSE important QA concept hai

Deployment ke dauraan (jo minutes se lekar ghanto tak ho sakta hai) **v1 aur v2 dono simultaneously live traffic serve kar rahe hain.** Iska matlab:

1. **Ek hi user ka session dono versions ko hit kar sakta hai.** User ne v2 se page load kiya (naya JS bundle mila), phir uska next API call v1 pe chala gaya. Agar v2 ke frontend ne aisa field bheja jo v1 ka backend nahi jaanta — 400 error.
2. **Database dono versions ke liye compatible honi chahiye.** Agar v2 ne column rename kar diya to v1 crash karega — aur v1 abhi bhi live hai.
3. **Cache mein dono versions ke objects hain.** v1 ne cache mein purana shape likha, v2 ne naya shape padha — deserialisation error.
4. **Background jobs / queues** — v1 ne message queue mein purana format daala, v2 consume kar raha hai.

### Expand-Contract (Parallel Change) pattern — mixed-version ka solution

Ye pattern **har senior SDET ko aana chahiye**. Ek breaking schema change ko **teen non-breaking deployments** mein todo.

Example: `purchase_orders` table mein `vendor_name` (string) ko `vendor_id` (FK) banana hai.

```
┌──────────────────────────────────────────────────────────────────────┐
│  PHASE 1 — EXPAND (additive only, backward compatible)               │
│                                                                       │
│  DB:   ALTER TABLE purchase_orders ADD COLUMN vendor_id BIGINT NULL; │
│  Code: naya code DONO likhta hai — vendor_name AUR vendor_id         │
│        padhta abhi bhi vendor_name se hai                            │
│                                                                       │
│  v1 (purana) still works -> vendor_name maujood hai      ✅          │
│  v2 (naya)   works       -> dono columns likhta hai      ✅          │
│  MIXED VERSION SAFE                                                   │
└──────────────────────────────────────────────────────────────────────┘
                                  v
┌──────────────────────────────────────────────────────────────────────┐
│  PHASE 2 — MIGRATE + SWITCH READS                                    │
│                                                                       │
│  Backfill: purane rows ka vendor_id populate karo (batched job)      │
│  Code: ab padhna vendor_id se, likhna abhi bhi dono mein             │
│                                                                       │
│  v1 still works -> vendor_name abhi bhi likha jaa raha hai ✅        │
│  MIXED VERSION SAFE                                                   │
└──────────────────────────────────────────────────────────────────────┘
                                  v
┌──────────────────────────────────────────────────────────────────────┐
│  PHASE 3 — CONTRACT (cleanup, only after v1 fully gone)              │
│                                                                       │
│  Code: vendor_name likhna band                                       │
│  DB:   ALTER TABLE purchase_orders DROP COLUMN vendor_name;          │
│                                                                       │
│  Ye tabhi safe hai jab koi bhi purana version live NAHI hai          │
└──────────────────────────────────────────────────────────────────────┘
```

**QA ki responsibility har phase mein:**

| Phase | QA kya test karta hai |
|---|---|
| Expand | Dual-write actually dono jagah likh raha hai? Purana read path abhi bhi kaam kar raha hai? |
| Migrate | Backfill **complete** hai? Koi NULL `vendor_id` bacha? Backfill ke dauraan naye rows bhi handle hue? Reads naye column se sahi data de rahe hain? |
| Contract | Koi code path bacha jo purana column reference karta ho? (grep + integration test) |
| **Har phase** | **Rollback test — agar is phase ke baad rollback karna pada to system kaam karega?** |

**Sabse zyada miss kiya jaane wala test:** Phase 2 mein backfill ke dauraan naye rows aate rehte hain. Agar backfill job sirf "existing rows" leti hai aur naye code ne abhi tak dual-write shuru nahi kiya (deploy ordering galat), to gap ban jaata hai. Ye classic production bug hai.

### QA rolling deployment mein kya alag karta hai

- **Mixed-version compatibility test:** deliberately v1 client + v2 backend, aur v2 client + v1 backend — dono combinations. Ye ek explicit test suite honi chahiye, assumption nahi.
- **API backward compatibility check:** naya API version purane consumers ko tod to nahi raha. Contract testing (Pact) yahan valuable hai.
- **Migration rollback test:** har migration ka `down` script actually chalta hai? Staging pe test karo.
- **Deployment ke dauraan smoke:** deploy khatam hone ka intezaar mat karo — deploy chalne ke dauraan smoke chalao, kyunki mixed window hi risky window hai.

> **[REAL]** Merlin ERP mein ye khaas taur pe relevant hai. Purchase order aur bid dono ke paas material/trade/custom line item types hain. Agar kabhi line-item schema badla (jaise custom items ka pricing model), to expand-contract lazmi hai — kyunki ek half-migrated PO ka matlab hai **galat total customer ko dikh raha hai**, aur wo crash nahi karta, wo silently galat hota hai. Interview mein bolne layak: *"In an ERP the dangerous failure isn't a 500, it's a wrong number that nobody alerts on."*

---

## 8.5 Blue-Green deployment

**Kaise:** Do identical production environments — Blue (live) aur Green (idle). Naya version Green pe deploy karo, wahan poori tarah test karo, phir load balancer/DNS se saara traffic ek saath Green pe switch kar do. Blue idle ho jaata hai (rollback ke liye ready).

```
   BEFORE:                              AFTER SWITCH:
   ┌──────────┐                          ┌──────────┐
   │    LB    │                          │    LB    │
   └────┬─────┘                          └────┬─────┘
        │ 100%                                │ 100%
        v                                     v
   ┌─────────┐    ┌──────────┐           ┌─────────┐    ┌──────────┐
   │  BLUE   │    │  GREEN   │           │  BLUE   │    │  GREEN   │
   │   v1    │    │   v2     │           │   v1    │    │   v2     │
   │  LIVE   │    │  IDLE    │           │  IDLE   │    │  LIVE    │
   └─────────┘    └──────────┘           └─────────┘    └──────────┘
                   ^                       ^
                   QA yahan test karta hai  rollback = switch wapas (seconds)
```

**Rollback speed:** **Sabse fast** — traffic wapas Blue pe, seconds mein. Yahi blue-green ka main selling point hai.

**Cost:** 2x infrastructure (kam se kam switch ke dauraan).

**Sabse bada gotcha: DATABASE.** Blue aur Green usually **same database** share karte hain (do databases rakhna matlab data sync ka nightmare). Iska matlab:
- Schema change dono versions ke liye compatible honi chahiye — **expand-contract yahan bhi lagta hai**
- Rollback code ka to instant hai, **par data ka nahi**. Agar v2 ne naya data likh diya jise v1 padh nahi sakta, to rollback ke baad wo data broken hai.

**Doosra gotcha: in-flight sessions.** Switch ke waqt jo requests Blue pe chal rahi hain, unka kya? Connection draining chahiye. Aur user sessions — agar session server-side hai to Green ko wahi session store dekhna chahiye.

### QA blue-green mein kya alag karta hai

- **Green pe full production-equivalent test suite** chalao switch se pehle. Ye blue-green ka sabse bada QA fayda hai — tumhe production-identical environment milta hai jisme koi real user nahi hai. Yahan tum **destructive tests bhi chala sakte ho** (limits ke andar).
- **Green ko production data ke against test karo** (shared DB) — matlab realistic data volume, realistic edge cases.
- **Switch ke turant baad smoke** — kyunki switch khud fail ho sakta hai (DNS TTL, LB config).
- **Rollback rehearse karo.** Blue-green ka poora point fast rollback hai — agar tumne wo path kabhi test nahi kiya to tumhe pata nahi wo kaam karta hai. Ye QA ka kaam hai: **rollback bhi ek test case hai.**

---

## 8.6 Canary deployment

**Kaise:** Naya version chhote % traffic ko serve karta hai. Metrics dekho. Achha lag raha hai to % badhao. Kharab lag raha hai to 0% karo.

```
  Stage 1:  v2 gets  1% traffic  ──> 10 min observe ──> metrics OK?
  Stage 2:  v2 gets  5% traffic  ──> 10 min observe ──> metrics OK?
  Stage 3:  v2 gets 25% traffic  ──> 20 min observe ──> metrics OK?
  Stage 4:  v2 gets 50% traffic  ──> 30 min observe ──> metrics OK?
  Stage 5:  v2 gets 100% traffic ──> DONE

  Kisi bhi stage pe metrics kharab --> v2 ko 0% --> rollback complete
```

**Rollback speed:** Fast — traffic weight 0 kar do. Aur **blast radius chhota hai** — agar 1% pe bug mila to sirf 1% users affected hue.

**Canary ka asli superpower: automated analysis.** Tools (Argo Rollouts, Flagger, Spinnaker) canary ke metrics ko baseline se **statistically compare** karte hain:

| Metric | Baseline (v1) | Canary (v2) | Verdict |
|---|---|---|---|
| Error rate | 0.12% | 0.13% | OK |
| p95 latency | 240ms | 251ms | OK |
| p99 latency | 800ms | 2100ms | **FAIL — rollback** |
| PO creation success | 98.1% | 97.9% | OK |

### QA canary mein kya alag karta hai

Ye sabse bada mental shift hai: **canary mein tumhara "test" ab ek assertion nahi, ek metric comparison hai.**

- **Tumhe define karna padta hai ki "healthy" kya hai** — kaun se metrics, kya threshold, kitni der observe. Ye directly QA ka kaam hai kyunki tum jaante ho kaunsa business behaviour matter karta hai.
- **Business metrics technical metrics se zyada important hain.** Error rate normal ho sakti hai jabki PO creation rate gir gayi ho (button click pe silently kuchh nahi hota). Tumhe wo metric define karni padegi.
- **Sample size ka dhyan** — 1% traffic pe agar sirf 50 requests aayi to statistical confidence zero hai. Kam traffic wale endpoints ke liye canary time badhana padta hai.
- **Sticky sessions zaroori hain** — ek user ko baar-baar v1/v2 ke beech bounce nahi karna chahiye, warna uske liye app randomly toota hua lagega.
- **Canary khud mixed-version hai** — matlab backward compatibility ki saari requirements rolling wali yahan bhi lagti hain.

---

## 8.7 Feature flags (feature toggles)

**Kaise:** Code deploy ho gaya par feature runtime config se off hai. On karna = deployment nahi, config change.

```python
# Simplest form
if flags.is_enabled("new_po_pricing_engine", user=current_user):
    total = new_pricing_engine.calculate(line_items)
else:
    total = legacy_pricing.calculate(line_items)
```

**Rollback speed:** **Instant, aur deployment ki zaroorat nahi.** Ye sabse fast rollback mechanism hai jo exist karta hai.

**Flag types (interview mein ye distinction achhi lagti hai):**

| Type | Life | Purpose |
|---|---|---|
| Release toggle | Din/hafte | Incomplete feature chhupana (trunk-based ke liye) |
| Experiment toggle | Hafte | A/B test |
| Ops toggle | Permanent | Load pe expensive feature band karna (kill switch) |
| Permission toggle | Permanent | Plan/role ke hisaab se feature |

### QA feature flags mein kya alag karta hai — aur ye combinatorial problem hai

**Test matrix double ho jaata hai.** Ek flag = 2 states. Do flags = 4 combinations. Das flags = 1024.

Practically kya karo:

1. **Har active release toggle ke dono states test karo** — ON aur OFF. OFF path bhoolna sabse common galti hai; log naya path test karte hain aur bhool jaate hain ki purana path abhi bhi production mein zinda hai.
2. **Combinations sirf jab interaction ho.** 10 flags ke 1024 combos test karna impossible hai. Risk-based: sirf un flags ke pairs test karo jo same module ko touch karte hain.
3. **Default state test karo.** Agar flag service down ho jaaye to code ka fallback kya hai? Ye test hona chahiye — flag service outage ne poore products girae hain.
4. **Flag ko test-controllable banao.** Test setup mein flag set karne ka API hona chahiye, warna tum flags test hi nahi kar paoge.
5. **Stale flags par SLA lagao.** Release toggle 30-60 din se zyada nahi jeena chahiye. QA ko ye actively raise karna chahiye — ye technical debt hai jo test matrix ko exponentially bloat karti hai.

```python
# conftest.py — flag override fixture
import pytest

@pytest.fixture
def with_flag(page):
    """Set a feature flag override for the test session via cookie/localStorage."""
    def _set(flag_name: str, enabled: bool):
        page.add_init_script(
            f"window.localStorage.setItem('ff_override_{flag_name}', '{str(enabled).lower()}')"
        )
    return _set


@pytest.mark.parametrize("enabled", [True, False])
def test_po_total_correct_under_both_pricing_engines(page, with_flag, enabled):
    with_flag("new_po_pricing_engine", enabled)
    # ... same assertions, both code paths
```

---

## 8.8 Shadow / dark launch (traffic mirroring)

**Kaise:** Real production traffic ko **duplicate** karke naye version pe bhi bheja jaata hai, par uska **response discard** kar diya jaata hai. User ko hamesha purana version ka response milta hai.

```
                     ┌────────────┐
   User request ────>│   Proxy    │────> v1 (LIVE)  ────> response ────> User
                     │  (mirror)  │
                     └─────┬──────┘
                           │ duplicate copy
                           v
                        v2 (SHADOW)  ────> response ──> DISCARDED
                                              │
                                              v
                                        compare with v1 response
                                        (diffing / logging)
```

**Kab use karta hai:** Jab risk bahut zyada ho aur tum production-realistic load aur data chahte ho bina kisi user ko expose kiye. Classic use cases:
- Bade rewrite (purana service naye service se replace)
- Performance validation real traffic pattern pe
- Pricing/calculation engine replacement — **exactly tumhare ERP jaisa case**

**Rollback:** Concept hi nahi — shadow user-facing hai hi nahi.

### QA shadow mein kya alag karta hai — aur sabse bada khatra

**Sabse bada khatra: SIDE EFFECTS.** Shadow service real traffic process kar rahi hai. Agar wo:
- Database mein likh de → duplicate records
- Email bhej de → user ko do emails
- Payment gateway call kar de → **double charge**
- Third-party API call kare (QuickBooks!) → duplicate entries

Isliye shadow environment ko **side-effect-free** banana padta hai: writes ko separate DB/schema mein, external calls ko stub/sandbox pe, emails ko blackhole pe.

**QA ka main kaam yahan: response diffing.**

```python
# Shadow comparison — conceptual
def compare_responses(primary: dict, shadow: dict) -> list[str]:
    """Return list of meaningful differences, ignoring known-volatile fields."""
    IGNORE = {"request_id", "generated_at", "server", "trace_id"}
    diffs = []
    for key in set(primary) | set(shadow):
        if key in IGNORE:
            continue
        if primary.get(key) != shadow.get(key):
            diffs.append(f"{key}: primary={primary.get(key)!r} shadow={shadow.get(key)!r}")
    return diffs
```

Diffing mein QA ka judgement chahiye — kaunsa difference "expected improvement" hai aur kaunsa "regression"? Wo tool nahi bata sakta, wo domain knowledge hai.

> **[REAL]** Merlin ke liye shadow launch ka perfect use case: agar kabhi **PO/bid total calculation engine** replace karni ho. Tum shadow chalao, har real PO ka total dono engines se calculate karo, aur diff karo. Ek hafte mein tumhe production ke real data pe har edge case mil jaayega — jo tumne manually kabhi nahi socha hota (weird tax rates, zero-quantity custom items, negative adjustments). Ye interview mein bolna: *"For a pricing engine rewrite in an ERP I'd argue strongly for shadow traffic with response diffing, because real production data contains edge cases no test designer invents — and in an ERP a silently wrong total is worse than an outage."*

## 8.9 Interview answer

> **Interview answer:**
> "There are five strategies I'd consider, and I think about them in terms of rollback speed and what they change about testing.
>
> Rolling replaces instances gradually. It needs no extra infrastructure, but it creates a mixed-version window where old and new run simultaneously, so backward compatibility is mandatory. Blue-green runs two full environments and switches traffic at once — rollback is seconds, which is its main advantage, but it costs double infrastructure and the database is usually shared, so schema changes still have to be compatible both ways. Canary sends a small percentage of traffic to the new version and increases it while watching metrics — small blast radius, fast rollback, but it needs real observability to be meaningful. Feature flags decouple deploy from release entirely: rollback is a config change, instant, no deploy needed. Shadow launch mirrors real traffic to the new version and discards the response, so you get production-realistic validation with zero user exposure.
>
> What changes for QA is different in each. In blue-green, I get a production-identical environment with no real users, so I can run the full suite — including destructive cases — before the switch, and my key deliverable is verifying the rollback path actually works, because an untested rollback is not a rollback. In canary, my 'test' stops being an assertion and becomes a metric comparison, so I have to define what healthy means — and I'd insist on business metrics, not just error rate and latency, because the dangerous failure is the one where errors look normal but purchase order creation quietly drops. In feature flags, my test matrix doubles and I have to cover both flag states, plus the default when the flag service is unavailable. And in shadow, my job is response diffing and — critically — making sure the shadow path has no side effects, because it's processing real traffic and could double-charge or duplicate third-party records.
>
> Rolling deserves the most attention because it's the default in Kubernetes and people forget the mixed-version window. During that window a user can load the new frontend and have their next API call served by the old backend. The pattern that solves it is expand-contract: split a breaking change into three deploys. Expand — add the new column and dual-write, keeping the old read path. Migrate — backfill and switch reads to the new column, still dual-writing. Contract — stop writing the old field and drop it, only once no old version is running. Each of those three deploys is individually backward-compatible, so any of them can be rolled back safely.
>
> On my product that matters a lot, because it's a construction ERP. A half-migrated purchase order doesn't crash — it shows the customer a wrong total, and nothing alerts on a wrong number. So for anything touching pricing or line items I'd insist on expand-contract plus explicit mixed-version tests: old client against new backend and new client against old backend, as real test cases rather than an assumption."

**Cross-question: "Blue-green mein database kaise handle karoge?"**

> **Interview answer:**
> "The database is where blue-green gets misunderstood. People assume you flip everything, but you almost never duplicate the database — two databases means you'd have to keep them in sync bidirectionally during the switch, which is harder than the problem you're solving. So blue and green share one database.
>
> That means the fast rollback blue-green is famous for only applies to the *code*. Data changes aren't rolled back by flipping traffic. If the new version wrote records in a new format and you switch back, the old version now has to read data it doesn't understand.
>
> So the schema has to be compatible with both versions at the moment of switch — which is the same expand-contract discipline rolling deployments need. Blue-green doesn't exempt you from it; it just makes people think it does.
>
> Concretely I'd require: migrations are additive only in the deploy that switches traffic, destructive changes like dropping a column happen in a later, separate deploy once the old version is definitely gone, and my pre-switch test plan includes running the *old* version against the *new* schema, because that's exactly the state a rollback puts you in. That test is the one people skip, and it's the one that matters."

**Cross-question: "Canary ke liye kaunse metrics choose karoge?"**

> **Interview answer:**
> "I'd choose in three layers, and I'd argue QA should own defining them because it's essentially test oracle design.
>
> Layer one is technical health: error rate — 5xx and unhandled exceptions — plus latency at p95 and p99, and resource saturation like CPU and memory. These catch crashes and performance regressions, but they only catch loud failures.
>
> Layer two is what I care about more: business outcome metrics. For a construction ERP that's purchase orders created per hour, bid submissions completed, login success rate, and PDF or export generation success. These catch the silent failures — the case where a button renders, the request returns 200, and nothing actually gets saved. Error rate looks perfect and the product is broken.
>
> Layer three is comparative and it's the part people get wrong: every metric must be compared against the baseline running at the same time, not against a fixed threshold. Traffic patterns vary by hour and day, so an absolute threshold gives you false alarms at 9am and false confidence at 3am. Comparing canary to control removes that whole class of noise.
>
> Two practical caveats. Sample size — at one percent traffic on a low-volume endpoint you might see fifty requests, which supports no conclusion, so for low-traffic paths I'd either extend the observation window or start the canary at a higher percentage. And sticky routing — a given user should stay on one version, otherwise their experience is randomly inconsistent and your diagnosis gets very confusing."

---

# 9. GitHub Actions — Complete Guide + Setup Steps

## 9.1 Setup — Step by step (ye literally follow karo)

### Step 1: Directory banao

GitHub Actions ek **fixed path** dekhta hai. Repo root mein:

```bash
cd /path/to/your/repo
mkdir -p .github/workflows
```

Path exactly `.github/workflows/` hona chahiye. `.github/workflow/` (singular) kaam nahi karega — ye sabse common beginner mistake hai.

### Step 2: Pehli workflow file banao

```bash
touch .github/workflows/ci.yml
```

File ka naam kuchh bhi ho sakta hai (`ci.yml`, `tests.yml`, `nightly.yml`), extension `.yml` ya `.yaml`.

### Step 3: Minimum working workflow likho

```yaml
name: CI

on: [push]

jobs:
  hello:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Pipeline chal gaya"
```

### Step 4: Commit aur push

```bash
git add .github/workflows/ci.yml
git commit -m "ci: add first workflow"
git push
```

### Step 5: Result dekho

GitHub pe repo kholo → **Actions** tab → workflow run dikhega. Click karke logs dekho.

**Bas. CI setup ho gaya.** Ab isko build karte hain.

> **Important:** Workflow file **jis branch pe hai wahin se run hoti hai**. Agar tum feature branch pe workflow add karte ho aur `on: push` hai, to wo feature branch pe chalegi. `pull_request` trigger ke liye PR banana padega.

---

## 9.2 YAML anatomy — har keyword

```yaml
# ─────────────────────────────────────────────────────────────
name: CI Pipeline              # Actions tab mein dikhne wala naam. Optional.

on:                            # TRIGGER — kab chalega
  push:
    branches: [main]

env:                           # Workflow-level env vars (saare jobs ko milte hain)
  PYTHON_VERSION: "3.11"

jobs:                          # Ek ya zyada jobs. Default: PARALLEL chalte hain.
  test:                        # <-- job ID (tumhara chuna hua, unique)
    name: Run tests            # <-- UI mein dikhne wala naam. Optional.
    runs-on: ubuntu-latest     # RUNNER — kis machine pe chalega
    timeout-minutes: 30        # Job ka max time. HAMESHA set karo.

    steps:                     # Steps SEQUENTIAL chalte hain, same machine pe
      - name: Checkout code
        uses: actions/checkout@v4       # `uses` = koi banaayi hui action chalao

      - name: Set up Python
        uses: actions/setup-python@v5
        with:                            # `with` = action ko input do
          python-version: ${{ env.PYTHON_VERSION }}

      - name: Install dependencies
        run: pip install -r requirements.txt    # `run` = shell command chalao

      - name: Run tests
        run: pytest -v
        env:                             # Step-level env var
          BASE_URL: https://staging-app.merlinai.co
# ─────────────────────────────────────────────────────────────
```

| Keyword | Kya karta hai |
|---|---|
| `name` | Display name |
| `on` | Trigger events |
| `env` | Environment variables (workflow / job / step level) |
| `jobs` | Jobs ka collection — **default parallel** |
| `runs-on` | Runner OS: `ubuntu-latest`, `windows-latest`, `macos-latest`, ya `self-hosted` |
| `steps` | Job ke andar sequential steps |
| `uses` | Marketplace/repo se action import karo |
| `run` | Shell command |
| `with` | `uses` wale action ke inputs |
| `needs` | Job dependency |
| `if` | Conditional execution |
| `strategy` | Matrix / parallelism |
| `timeout-minutes` | Kill after N minutes |
| `continue-on-error` | Fail hone pe bhi pipeline chalti rahe |
| `defaults` | Default shell/working-directory |
| `permissions` | GITHUB_TOKEN ki permissions (least privilege) |
| `concurrency` | Purane runs cancel karo jab naya aaye |

**Runner kya hai:** Ek fresh VM jo GitHub provide karta hai. Har job ko **apna** fresh runner milta hai — matlab do jobs ke beech file share nahi hoti (artifacts ya cache ke bina). Ye samajhna zaroori hai, ye common confusion hai.

---

## 9.3 Triggers — poora coverage

### push

```yaml
on:
  push:
    branches:
      - main
      - 'release/**'          # wildcard
    branches-ignore:
      - 'experiment/**'
    paths:                     # sirf tab chale jab ye files badli hon
      - 'tests/**'
      - 'requirements.txt'
    paths-ignore:
      - '**.md'                # docs change pe pipeline mat chalao
    tags:
      - 'v*'                   # release tags
```

### pull_request

```yaml
on:
  pull_request:
    branches: [main]           # PRs TARGETING main
    types:
      - opened
      - synchronize            # naye commits push hue (DEFAULT)
      - reopened
      - ready_for_review       # draft se ready hua
```

**Important nuance:** `pull_request` **merge commit** pe chalta hai (PR ka code + target branch ka code milaakar), na ki sirf PR ke code pe. Isliye PR check zyada realistic hota hai. `pull_request_target` alag hai — wo base branch ke context mein chalta hai aur secrets access deta hai, isiliye fork PRs ke liye **security risk** hai. Interview mein ye nuance bolna strong hai.

### schedule (cron)

```yaml
on:
  schedule:
    # POSIX cron. Ye hamesha UTC mein hota hai — local timezone nahi.
    - cron: '30 20 * * *'      # roz 20:30 UTC = 02:00 IST
    - cron: '0 6 * * 1'        # har Monday 06:00 UTC
```

Cron format: `minute hour day-of-month month day-of-week`

```
 ┌───── minute (0-59)
 │ ┌───── hour (0-23)
 │ │ ┌───── day of month (1-31)
 │ │ │ ┌───── month (1-12)
 │ │ │ │ ┌───── day of week (0-6, Sunday=0)
 │ │ │ │ │
 30 20 * * *
```

**Gotchas:**
- Cron **UTC** hai. IST = UTC + 5:30. Nightly 2 AM IST chahiye → `30 20 * * *`.
- Scheduled workflows **default branch** se chalti hain, chahe tumne kisi aur branch pe likhi ho.
- GitHub scheduled runs ko high-load ke waqt **delay** kar deta hai (kabhi-kabhi 15-30 min). Exact timing pe depend mat karo.
- Public repo mein 60 din inactivity ke baad scheduled workflows **automatically disable** ho jaati hain.

### workflow_dispatch (manual button, with inputs)

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
      test_suite:
        description: 'Which suite to run'
        required: true
        default: 'smoke'
        type: choice
        options: [smoke, regression, full]
      shard_count:
        description: 'Number of parallel shards'
        required: false
        default: '4'
        type: string
      headed:
        description: 'Run browser in headed mode (debug)'
        type: boolean
        default: false
```

Use karne ka tareeka: `${{ inputs.environment }}` (ya purana syntax `${{ github.event.inputs.environment }}`).

UI mein: Actions tab → left sidebar mein workflow choose karo → **"Run workflow"** button → dropdown mein inputs bharo.

**Ye QA ke liye sabse useful trigger hai** — tum interview mein bolo: *"I expose a workflow_dispatch with environment and suite inputs so anyone on the team — including a PM — can run a targeted regression against staging without asking me and without a local setup."*

### Baaki useful triggers

```yaml
on:
  workflow_run:                       # doosri workflow khatam hone pe
    workflows: ["Deploy"]
    types: [completed]
    branches: [main]

  repository_dispatch:                # external system se API call
    types: [deploy-finished]

  issue_comment:                      # PR comment pe (e.g. "/run-regression")
    types: [created]
```

`repository_dispatch` trigger karna:

```bash
curl -X POST \
  -H "Authorization: Bearer $GH_TOKEN" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/OWNER/REPO/dispatches \
  -d '{"event_type":"deploy-finished","client_payload":{"env":"staging","sha":"a3f9c21"}}'
```

---

## 9.4 Jobs, needs, if, continue-on-error

### Jobs default mein PARALLEL hote hain

```yaml
jobs:
  lint:      # ┐
    ...      # │ ye teeno EK SAATH chalte hain
  unit:      # │ alag-alag runners pe
    ...      # │
  typecheck: # ┘
    ...
```

### needs — dependency banao

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps: [...]

  unit:
    runs-on: ubuntu-latest
    steps: [...]

  build:
    needs: [lint, unit]        # dono pass hone ke baad hi chalega
    runs-on: ubuntu-latest
    steps: [...]

  e2e:
    needs: build               # build ke baad
    runs-on: ubuntu-latest
    steps: [...]
```

Ye graph banta hai:

```
   lint ──┐
          ├──> build ──> e2e
   unit ──┘
```

### Job ke beech data pass karna — outputs

```yaml
jobs:
  setup:
    runs-on: ubuntu-latest
    outputs:
      image_tag: ${{ steps.meta.outputs.tag }}
    steps:
      - id: meta
        run: echo "tag=${GITHUB_SHA::7}" >> "$GITHUB_OUTPUT"

  deploy:
    needs: setup
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying image ${{ needs.setup.outputs.image_tag }}"
```

### if — conditional execution

```yaml
jobs:
  deploy:
    # sirf main branch pe, sirf push event pe
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - name: This runs only if previous steps succeeded (default)
        run: echo "ok"

      - name: This runs even if an earlier step failed
        if: failure()
        run: echo "something broke"

      - name: This ALWAYS runs — success, failure, or cancelled
        if: always()
        run: echo "cleanup"

      - name: Only on success
        if: success()
        run: echo "yay"

      - name: Only if the job was cancelled
        if: cancelled()
        run: echo "cancelled"
```

Common `if` expressions jo yaad rakhne layak hain:

| Expression | Matlab |
|---|---|
| `github.ref == 'refs/heads/main'` | main branch |
| `github.event_name == 'pull_request'` | PR event |
| `github.actor != 'dependabot[bot]'` | Dependabot ke liye skip |
| `contains(github.event.head_commit.message, '[skip-e2e]')` | Commit message flag |
| `github.event.pull_request.draft == false` | Draft PR pe mat chalao |
| `always()` | Hamesha |
| `failure()` | Koi pichhla step fail hua |
| `startsWith(github.ref, 'refs/tags/v')` | Version tag |

### continue-on-error

```yaml
      - name: Optional lighthouse audit
        run: npx lighthouse-ci autorun
        continue-on-error: true       # fail hone pe step red dikhega par job pass rahega
```

Job level pe bhi lag sakta hai:

```yaml
  performance:
    continue-on-error: true    # ye job pipeline nahi rokega
```

**Kab use karo:** Naye check ko "warn mode" mein introduce karne ke liye. Pehle 2 hafte `continue-on-error: true` rakho, data dekho, phir hata do. Ye ek achha rollout technique hai aur interview mein bolne layak hai — *"I introduce new gates in warn mode first, so I can measure the false-positive rate before I make it blocking. A gate introduced as blocking on day one that turns out to be noisy destroys trust immediately."*

### concurrency — purane runs cancel karo

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

Matlab: same branch pe naya push aaya to purana run cancel. Ye **compute minutes bachata hai** aur queue clear rakhta hai. PR workflows mein hamesha lagao. **Production deploy workflow mein mat lagao** — beech mein cancelled deploy se bura kuchh nahi.

---

## 9.5 Matrix strategy aur test sharding

### Basic matrix

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false            # ek combo fail ho to baaki bhi chalein
      max-parallel: 6             # ek saath max 6
      matrix:
        os: [ubuntu-latest, windows-latest]
        python: ["3.10", "3.11", "3.12"]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python }}
      - run: pytest
```

Ye **2 × 3 = 6 jobs** banata hai, sab parallel.

**`fail-fast: false` bahut important hai QA ke liye.** Default `true` hai, matlab ek shard fail hote hi baaki cancel ho jaate hain — aur tumhe adhoori information milti hai. Tum jaanna chahte ho ki 1 shard fail hua ya 4, kyunki wo bilkul alag diagnosis hai.

### include / exclude

```yaml
    strategy:
      matrix:
        browser: [chromium, firefox, webkit]
        include:
          - browser: chromium         # is combo ko extra property do
            record_video: true
        exclude:
          - browser: webkit           # ubuntu pe webkit skip
            os: ubuntu-latest
```

### Sharding — bade suite ko todna

Ye **THE technique** hai slow pipeline ke liye. Do tareeke:

**Tareeka 1: `pytest-split`** (recommended — duration-based, balanced shards banata hai)

```bash
pip install pytest-split
```

```yaml
jobs:
  e2e:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        shard: [1, 2, 3, 4]
    steps:
      - uses: actions/checkout@v4
      # ... setup ...
      - name: Run shard ${{ matrix.shard }} of 4
        run: |
          pytest tests/e2e \
            --splits 4 \
            --group ${{ matrix.shard }} \
            --splitting-algorithm least_duration \
            --durations-path .test_durations \
            --junitxml=results-${{ matrix.shard }}.xml
```

`.test_durations` file commit karo repo mein — isi se pytest-split jaanta hai ki kaunsa test kitna slow hai aur shards balance karta hai. Generate karne ke liye:

```bash
pytest tests/e2e --store-durations --durations-path .test_durations
git add .test_durations && git commit -m "ci: record test durations for sharding"
```

**Bina duration data ke shards imbalanced honge** — ek shard 20 min, doosra 3 min. Tumhara total time slowest shard hai, to imbalance = wasted parallelism. Ye nuance interview mein bolna.

**Tareeka 2: Manual split by marker/path** (simple, control zyada)

```yaml
      matrix:
        include:
          - name: po-flows
            args: "-m po"
          - name: bid-flows
            args: "-m bid"
          - name: smoke
            args: "-m smoke"
    steps:
      - run: pytest ${{ matrix.args }}
```

> **[REAL]** Tumhare paas **6 E2E flows** hain (PO material/trade/custom + bid material/trade/custom), har ek 12-15 steps. Sharding ka sabse natural split: **6 shards, ek flow per shard.** Serial mein agar poori suite 30 min leti hai to 6 shards mein ~5-6 min. Ye interview mein bolne ke liye concrete number hai. Aur agar flows unequal length ke hain to `pytest-split` ka `least_duration` algorithm use karo.

---

## 9.6 Caching — pip, node, Playwright browsers

Caching pipeline speed ka **sabse sasta big win** hai.

### Python / pip

Sabse simple — `setup-python` mein built-in:

```yaml
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
          cache: 'pip'                          # automatic pip cache
          cache-dependency-path: requirements.txt
```

Manual control chahiye to:

```yaml
      - name: Cache pip
        uses: actions/cache@v4
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements*.txt') }}
          restore-keys: |
            ${{ runner.os }}-pip-
```

**Cache key ka logic samajh lo — ye interview mein poochha jaata hai:**
- `key` exact match dhoondhta hai. Match mila = cache hit, restore ho gaya.
- `hashFiles(...)` requirements ka hash deta hai. requirements badle → naya key → cache miss → fresh install → naya cache save.
- `restore-keys` **prefix fallback** hai. Exact key na mile to sabse recent matching prefix wala cache restore hota hai — matlab partial benefit milta hai (zyadatar packages already cached hain, sirf naye download honge).

### Node / npm

```yaml
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
```

### Playwright browsers — ye sabse bada win hai

Playwright browsers ~350MB hain aur download mein 60-120 seconds lagte hain. Har run pe. Cache karo:

```yaml
      - name: Get Playwright version
        id: pw
        run: |
          # requirements se exact playwright version nikalo — cache key ke liye
          PW_VERSION=$(pip show playwright 2>/dev/null | awk '/^Version:/{print $2}')
          echo "version=${PW_VERSION}" >> "$GITHUB_OUTPUT"

      - name: Cache Playwright browsers
        id: pw-cache
        uses: actions/cache@v4
        with:
          path: ~/.cache/ms-playwright
          key: ${{ runner.os }}-playwright-${{ steps.pw.outputs.version }}

      - name: Install Playwright browsers
        if: steps.pw-cache.outputs.cache-hit != 'true'
        run: playwright install --with-deps chromium

      - name: Install OS deps only (cache hit path)
        if: steps.pw-cache.outputs.cache-hit == 'true'
        run: playwright install-deps chromium
```

**Critical detail jo log miss karte hain:** cache hit hone pe bhi `install-deps` chalana padta hai. Browsers cache mein hain, par unke **system libraries** (fonts, libnss, etc.) runner pe install karne padte hain — wo `~/.cache` mein nahi hain, wo `/usr/lib` mein hain. Ye bolna interview mein depth dikhata hai.

**Cache key mein Playwright version zaroori kyun:** Playwright version upgrade hone pe browser binaries badalte hain. Version key mein na ho to purane browsers naye Playwright ke saath use honge → cryptic errors.

### Caching ke general rules

| Rule | Kyun |
|---|---|
| Key mein lockfile ka hash daalo | Dependencies badle to cache invalidate ho |
| `restore-keys` hamesha do | Partial hit poori miss se behtar hai |
| Build **output** cache mat karo | Stale artifact ka risk. Dependencies cache karo, build results nahi. |
| Cache size dekho | GitHub ka limit 10GB per repo. Full hone pe LRU eviction hoti hai. |
| Cache branch-scoped hai | Feature branch base branch ka cache padh sakti hai, ulta nahi |

---

## 9.7 Secrets — setup se lekar use tak

### Step-by-step: secret add karna

1. GitHub pe repo kholo
2. **Settings** tab (repo ki settings, account ki nahi)
3. Left sidebar → **Secrets and variables** → **Actions**
4. **New repository secret** button
5. **Name**: `SLACK_BOT_TOKEN` (UPPER_SNAKE_CASE convention)
6. **Secret**: value paste karo
7. **Add secret**

**Ek baar save hone ke baad tum use dobara padh nahi sakte.** Sirf overwrite kar sakte ho. Ye by design hai.

### Secret ke levels

| Level | Scope | Kab |
|---|---|---|
| **Repository secret** | Ek repo ki saari workflows | Default |
| **Environment secret** | Sirf jab job us environment ko target kare | **Production credentials — sabse safe** |
| **Organization secret** | Multiple repos | Shared credentials (registry, Slack) |

**Environment secrets sabse important hain QA ke liye** — production DB password sirf `production` environment mein ho, aur us environment pe required-reviewer protection ho. Matlab koi random PR workflow us secret ko chhu nahi sakti.

### Reference karna

```yaml
      - name: Run tests
        run: pytest tests/e2e
        env:
          BASE_URL:        ${{ vars.BASE_URL }}              # non-secret variable
          TEST_USER:       ${{ secrets.TEST_USER_EMAIL }}
          TEST_PASSWORD:   ${{ secrets.TEST_USER_PASSWORD }}
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```

**`vars` vs `secrets`:** `vars` non-sensitive config ke liye hai (BASE_URL, TIMEOUT) — ye logs mein visible rehte hain, jo debugging ke liye achha hai. `secrets` masked hote hain.

### Masking — kaise kaam karta hai aur kaise fail hota hai

GitHub logs mein secret ki exact string ko `***` se replace kar deta hai.

**Par masking bypass ho sakti hai:**

| Bypass | Example |
|---|---|
| Base64 encode | `echo $TOKEN \| base64` → masked nahi (alag string hai) |
| Substring | Secret ka aadha hissa print karna |
| JSON mein embedded | `curl -v` request body dikha de |
| Transformed | `echo ${TOKEN:0:10}` |
| Error message | Library exception mein URL-with-password print kar de |

Isliye:
- `set -x` / `bash -x` **kabhi mat** use karo secrets wale steps mein
- `curl -v` ki jagah `curl -sf` use karo
- Custom masking add karo agar tum secret derive kar rahe ho:

```yaml
      - name: Derive and mask a value
        run: |
          DERIVED=$(some-command)
          echo "::add-mask::$DERIVED"        # ab ye bhi masked hai
          echo "value=$DERIVED" >> "$GITHUB_OUTPUT"
```

### Fork PRs — critical security detail

**Secrets fork se aayi PRs mein available NAHI hote.** Ye deliberate security feature hai — warna koi bhi tumhare repo ko fork karke, workflow modify karke, secrets exfiltrate kar leta.

Iska practical matlab: agar tumhara PR workflow secrets pe depend karta hai (jaise E2E tests jinhe login chahiye), to wo fork PRs pe fail hoga. Solutions:
- E2E ko `pull_request` se hata ke merge-to-main pe le jao
- `pull_request_target` use karo — **par bahut savdhani se**, kyunki wo base branch ke context mein chalta hai aur untrusted code ke saath combine karna dangerous hai
- Maintainer approval ke baad hi run karo (`required approval for first-time contributors` setting)

---

## 9.8 Artifacts — upload, download, aur `if: always()`

### Kya hai

Artifact = pipeline run se nikli koi file jo tum baad mein download kar sako. QA ke liye ye **lifeline** hai — screenshots, videos, traces, HTML reports, logs.

### Upload

```yaml
      - name: Upload test results
        if: always()                       # <-- CRITICAL. Neeche explain hai.
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report-shard-${{ matrix.shard }}
          path: |
            reports/
            screenshots/
            test-results/
            *.xml
          retention-days: 14               # default 90, cost bachao
          if-no-files-found: warn          # error | warn | ignore
          compression-level: 6             # 0-9
```

### `if: always()` — ye kyun sabse important line hai

Default behaviour: agar koi step fail hota hai, to baaki saare steps **skip** ho jaate hain.

Ab socho:

```yaml
      - run: pytest tests/e2e          # <-- FAIL hua
      - uses: actions/upload-artifact@v4   # <-- SKIP ho gaya
        with:
          path: screenshots/
```

**Nateeja: test fail hua, aur screenshots upload hi nahi hue.** Matlab tumhare paas debug karne ke liye kuchh nahi hai — bilkul us waqt jab tumhe sabse zyada zaroorat thi.

Ye **sabse common CI mistake hai jo QA log karte hain.** Artifacts hamesha tab chahiye jab test fail ho. To:

```yaml
      - name: Upload artifacts
        if: always()          # success ho ya failure, upload karo
```

Ya aur precise:

```yaml
      - name: Upload failure evidence
        if: failure()         # sirf fail hone pe (storage bachao)
```

**Rule yaad rakho: har diagnostic artifact step pe `if: always()` ya `if: failure()` hona chahiye. Bina condition ke wo sirf successful runs pe chalega, jahan uski koi zaroorat nahi.**

> **[REAL]** Tumhare `conftest.py` mein `pytest_runtest_makereport` hook se screenshot-on-failure hai. Wo screenshot local disk pe save hoti hai. **CI mein wo file runner ke saath delete ho jaayegi** jab tak tum upload-artifact + `if: always()` na lagao. Ye tumhare existing setup ka missing CI piece hai, aur interview mein bolne layak:
>
> *"My conftest already captures a screenshot on failure using the pytest_runtest_makereport hook. Moving that to CI needed one more piece: an upload-artifact step with `if: always()`. Without that condition the upload is skipped precisely when the tests failed, which is the only time you need the screenshots. It's a small detail but it's the difference between a CI failure you can debug and one you can only re-run and hope."*

### Download (doosre job mein)

```yaml
  merge-reports:
    needs: e2e
    if: always()
    runs-on: ubuntu-latest
    steps:
      - name: Download all shard reports
        uses: actions/download-artifact@v4
        with:
          pattern: playwright-report-shard-*     # saare shards
          path: all-reports/
          merge-multiple: true

      - name: Combine into one report
        run: |
          pip install junitparser
          junitparser merge all-reports/*.xml combined.xml
```

### Artifact vs Cache — farq (interview question)

| | Artifact | Cache |
|---|---|---|
| Purpose | **Output** save karna (reports, builds) | **Input** reuse karna (dependencies) |
| Kaun padhta hai | Insaan (download) + doosre jobs | Sirf pipeline |
| Miss hone pe | Data lost | Bas dheere hoga |
| Retention | Configurable, default 90 din | 7 din unused ke baad evict |
| Immutable? | Haan | Nahi (overwrite ho sakta hai) |

---

## 9.9 Environments aur protection rules

### Setup — step by step

1. Repo → **Settings** → **Environments** → **New environment**
2. Naam do: `staging` ya `production`
3. Protection rules configure karo:

| Rule | Kya karta hai |
|---|---|
| **Required reviewers** | Deploy se pehle named log approve karein (max 6) |
| **Wait timer** | N minute rukо deploy se pehle (cancel karne ka mauka) |
| **Deployment branches** | Sirf `main` ya sirf tags se deploy allowed |
| **Environment secrets** | Secrets jo sirf is environment mein available hain |
| **Environment variables** | Non-secret config |

### Workflow mein use

```yaml
jobs:
  deploy-production:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://app.merlinai.co       # Actions UI mein clickable link
    steps:
      - name: Deploy
        run: ./deploy.sh
        env:
          # ye secret SIRF production environment mein exist karta hai
          DEPLOY_KEY: ${{ secrets.PROD_DEPLOY_KEY }}
```

Jab ye job chalne wali hoti hai, agar `required reviewers` set hai to job **pause** ho jaata hai aur reviewers ko notification jaati hai. Approve hone pe chalega.

**Yahi tumhara "approval gate" hai** (Section 5, stage 14). Interview mein bolo: *"In GitHub Actions the approval gate is implemented as an environment with required reviewers — the job pauses, notifies the approvers, and the production secrets are scoped to that environment so no other workflow can reach them."*

---

## 9.10 Reusable workflows aur composite actions

Jab tumhare paas 5 workflows hain aur sabme wahi 8 setup steps hain — duplication. Do solutions:

### Reusable workflow (poora job reuse karo)

`.github/workflows/reusable-e2e.yml`:

```yaml
name: Reusable E2E

on:
  workflow_call:                     # <-- ye ise reusable banata hai
    inputs:
      environment:
        required: true
        type: string
      shard_total:
        required: false
        type: number
        default: 4
      pytest_args:
        required: false
        type: string
        default: ""
    secrets:
      TEST_USER_PASSWORD:
        required: true
      SLACK_BOT_TOKEN:
        required: false
    outputs:
      report_url:
        value: ${{ jobs.run.outputs.report_url }}

jobs:
  run:
    runs-on: ubuntu-latest
    outputs:
      report_url: ${{ steps.publish.outputs.url }}
    strategy:
      fail-fast: false
      matrix:
        shard: [1, 2, 3, 4]
    steps:
      - uses: actions/checkout@v4
      # ... setup, cache, run ...
      - id: publish
        run: echo "url=https://..." >> "$GITHUB_OUTPUT"
```

Caller:

```yaml
jobs:
  nightly:
    uses: ./.github/workflows/reusable-e2e.yml
    with:
      environment: staging
      pytest_args: "-m regression"
    secrets:
      TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
      SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```

Doosre repo se bhi call kar sakte ho: `uses: my-org/ci-workflows/.github/workflows/e2e.yml@v1`

### Composite action (steps ka group reuse karo)

`.github/actions/setup-python-playwright/action.yml`:

```yaml
name: 'Setup Python + Playwright'
description: 'Checkout, Python, cached deps, cached browsers — one step'

inputs:
  python-version:
    description: 'Python version'
    required: false
    default: '3.11'
  browsers:
    description: 'Space-separated browsers to install'
    required: false
    default: 'chromium'

runs:
  using: "composite"
  steps:
    - uses: actions/setup-python@v5
      with:
        python-version: ${{ inputs.python-version }}
        cache: 'pip'
        cache-dependency-path: requirements.txt

    - name: Install Python deps
      shell: bash                       # composite mein `shell` MANDATORY hai
      run: pip install -r requirements.txt

    - name: Resolve Playwright version
      id: pw
      shell: bash
      run: echo "v=$(pip show playwright | awk '/^Version:/{print $2}')" >> "$GITHUB_OUTPUT"

    - name: Cache browsers
      id: pw-cache
      uses: actions/cache@v4
      with:
        path: ~/.cache/ms-playwright
        key: ${{ runner.os }}-pw-${{ steps.pw.outputs.v }}-${{ inputs.browsers }}

    - name: Install browsers
      if: steps.pw-cache.outputs.cache-hit != 'true'
      shell: bash
      run: playwright install --with-deps ${{ inputs.browsers }}

    - name: Install system deps only
      if: steps.pw-cache.outputs.cache-hit == 'true'
      shell: bash
      run: playwright install-deps ${{ inputs.browsers }}
```

Use:

```yaml
      - uses: actions/checkout@v4
      - uses: ./.github/actions/setup-python-playwright
        with:
          python-version: '3.11'
          browsers: 'chromium firefox'
```

| | Reusable workflow | Composite action |
|---|---|---|
| Reuse ka unit | Poora job(s) | Steps ka group |
| Apna runner? | Haan, alag job hai | Nahi, caller ke runner pe |
| Secrets | Explicitly pass karne padte hain | Caller ke context mein hain |
| Matrix bana sakta hai? | Haan | Nahi |
| Nesting depth | 4 levels | 10 levels |

---

## 9.11 Self-hosted runners

### Kya hai

GitHub ke hosted runners ki jagah **apni machine** pe runner agent chalana.

### Setup — step by step

1. Repo/Org → **Settings** → **Actions** → **Runners** → **New self-hosted runner**
2. OS aur architecture chuno
3. Jo commands dikhte hain wo apni machine pe chalao:

```bash
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64.tar.gz -L \
  https://github.com/actions/runner/releases/download/v2.xxx.x/actions-runner-linux-x64-2.xxx.x.tar.gz
tar xzf ./actions-runner-linux-x64.tar.gz
./config.sh --url https://github.com/OWNER/REPO --token AAAAA...
# service ke roop mein chalao (boot pe auto-start)
sudo ./svc.sh install
sudo ./svc.sh start
```

4. Workflow mein:

```yaml
    runs-on: self-hosted
    # ya labels se target karo:
    runs-on: [self-hosted, linux, gpu, playwright]
```

### Kab use karo

| Reason | Detail |
|---|---|
| **Private network access** | Test target internal VPN pe hai, public internet se reachable nahi |
| **Cost** | Bahut zyada minutes use ho rahe hain; apna hardware sasta pad sakta hai |
| **Speed** | Warm caches, browsers pre-installed, faster disk — 2-3x tez ho sakta hai |
| **Special hardware** | GPU, specific OS version, real mobile devices attached |
| **Compliance** | Data organisation ke network se bahar nahi jaa sakta |
| **Long jobs** | Hosted runner ka 6-hour limit |

### Kab MAT use karo — aur trade-offs

| Downside | Detail |
|---|---|
| **Tum maintain karte ho** | OS patches, disk space, runner version upgrades — ye ek ongoing kaam hai |
| **State leak** | Hosted runner har run pe fresh hota hai. Self-hosted **nahi**. Pichhle run ka bacha hua file/process/env agla run affect kar sakta hai — ye **flakiness ka bada source** hai |
| **Security** | **Public repo pe self-hosted runner kabhi mat lagao.** Koi bhi PR bhej ke tumhari machine pe arbitrary code chala sakta hai |
| **Scaling** | Concurrency tumhare hardware tak limited hai |

**State leak ka mitigation** (ye QA-relevant hai):

```yaml
    steps:
      - name: Clean workspace before starting
        run: |
          rm -rf "${GITHUB_WORKSPACE}"/* || true
          pkill -f chromium || true          # zombie browsers maaro
          docker system prune -af || true
```

Ya ephemeral runners use karo (`--ephemeral` flag) — har job ke baad runner khud ko de-register kar deta hai, aur ek naya container spawn hota hai.

> **Interview answer:**
> "I'd use self-hosted runners for three reasons mainly: when the system under test is only reachable from a private network, when the cost of hosted minutes gets significant, or when I need warm state — pre-installed browsers and warm caches can cut a Playwright job's setup from two minutes to almost nothing.
>
> The trade-off I'd flag is that self-hosted runners are not clean. GitHub-hosted runners give you a fresh VM per job, which is a strong flakiness guarantee that people don't appreciate until they lose it. On a self-hosted runner, leftover files, zombie browser processes and stale environment variables from a previous run can leak into the next one, and that produces the worst kind of flakiness — the kind that depends on what ran before. So I'd either run jobs inside a container on the runner, or use ephemeral runners that deregister after each job, and I'd have an explicit cleanup step.
>
> And one hard rule: never attach a self-hosted runner to a public repository, because anyone can open a pull request that executes arbitrary code on your machine."

---

## 9.12 FULL production-grade workflow for the Merlin suite

Ye complete file hai. Fully commented. Tum ise apne repo mein daal sakte ho.

**Design:**
- **PR pe:** lint + format + type check + secret scan + unit tests. Target: **under 5 minutes.** Koi browser nahi.
- **Merge to main pe:** smoke suite staging ke against. Target: **under 5 minutes.**
- **Nightly (2 AM IST):** poori regression, 6 shards mein.
- **Manual:** kisi bhi environment pe koi bhi suite, workflow_dispatch se.
- **Hamesha:** artifacts upload + Slack notification.

### File 1 — `.github/workflows/pr-checks.yml`

```yaml
# ═══════════════════════════════════════════════════════════════════════════
# PR GATE — fast feedback. Target: under 5 minutes total.
# No browsers here. Anything needing a browser goes to a later stage.
# ═══════════════════════════════════════════════════════════════════════════
name: PR Checks

on:
  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened, ready_for_review]

# Same PR pe naya push aane pe purana run cancel karo — minutes bachao
concurrency:
  group: pr-${{ github.event.pull_request.number }}
  cancel-in-progress: true

# Least privilege: workflow ko sirf padhne ki permission chahiye
permissions:
  contents: read
  pull-requests: write        # PR pe comment karne ke liye

env:
  PYTHON_VERSION: "3.11"

jobs:
  # ─────────────────────────────────────────────────────────────────────
  # Job 1: Static analysis. Sabse sasta feedback, sabse pehle.
  # ─────────────────────────────────────────────────────────────────────
  static:
    name: Lint / Format / Types
    runs-on: ubuntu-latest
    timeout-minutes: 10
    # Draft PRs pe mat chalao — dev abhi kaam kar raha hai
    if: github.event.pull_request.draft == false
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: 'pip'
          cache-dependency-path: |
            requirements.txt
            requirements-dev.txt

      - name: Install dev dependencies
        run: pip install -r requirements-dev.txt

      # continue-on-error: false (default) — ye BLOCKING gates hain
      - name: Lint (ruff)
        run: ruff check . --output-format=github

      - name: Format check (ruff format)
        run: ruff format --check .

      - name: Type check (mypy)
        run: mypy tests/ pages/ utils/ --ignore-missing-imports

  # ─────────────────────────────────────────────────────────────────────
  # Job 2: Secret scanning. NON-NEGOTIABLE gate.
  # Ye wahi gate hai jo application-test.properties wali problem rokta.
  # ─────────────────────────────────────────────────────────────────────
  secrets-scan:
    name: Secret scan
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
        with:
          # fetch-depth 0 chahiye taaki gitleaks POORI history scan kar sake,
          # sirf latest commit nahi
          fetch-depth: 0

      - name: Run gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        # NOTE: ye deliberately blocking hai. Committed secret ek P0 hai,
        # chahe wo "sirf test config" ho.

  # ─────────────────────────────────────────────────────────────────────
  # Job 3: Unit tests + diff coverage gate
  # ─────────────────────────────────────────────────────────────────────
  unit:
    name: Unit tests
    runs-on: ubuntu-latest
    timeout-minutes: 15
    if: github.event.pull_request.draft == false
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0        # diff-cover ko base branch chahiye

      - uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: 'pip'
          cache-dependency-path: requirements.txt

      - name: Install dependencies
        run: pip install -r requirements-dev.txt

      - name: Run unit tests with coverage
        run: |
          pytest tests/unit \
            -v \
            --cov=. \
            --cov-report=xml \
            --cov-report=term-missing \
            --junitxml=unit-results.xml \
            -p no:cacheprovider

      # Diff coverage gate: naye/badle hue lines ka 80% covered hona chahiye.
      # Total coverage pe gate NAHI lagate — wo legacy code ko punish karta hai.
      - name: Diff coverage gate
        run: |
          pip install diff-cover
          git fetch origin main --depth=50
          diff-cover coverage.xml \
            --compare-branch=origin/main \
            --fail-under=80 \
            --html-report diff-coverage.html

      # if: always() — CRITICAL. Warna fail hone pe report upload hi nahi hoti.
      - name: Upload coverage reports
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: unit-coverage
          path: |
            coverage.xml
            diff-coverage.html
            unit-results.xml
          retention-days: 7

  # ─────────────────────────────────────────────────────────────────────
  # Job 4: Suite ki apni health — collect-only.
  # Ye pakadta hai: import errors, syntax errors, duplicate test IDs,
  # broken fixtures. Browser chalaye bina, 20 second mein.
  # ─────────────────────────────────────────────────────────────────────
  suite-health:
    name: E2E suite collects cleanly
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: 'pip'
      - run: pip install -r requirements.txt

      # --collect-only test chalata nahi, sirf discover karta hai.
      # Bahut tez, aur poori suite ke syntax/import ko validate karta hai.
      - name: Collect E2E tests without running them
        run: pytest tests/e2e --collect-only -q

      # Marker discipline: har E2E test pe suite marker hona chahiye
      - name: Verify every E2E test has a suite marker
        run: |
          UNMARKED=$(pytest tests/e2e --collect-only -q \
            -m "not smoke and not regression" 2>/dev/null | head -n -2)
          if [ -n "$UNMARKED" ]; then
            echo "::error::These tests have no smoke/regression marker:"
            echo "$UNMARKED"
            exit 1
          fi
```

### File 2 — `.github/workflows/smoke-on-merge.yml`

```yaml
# ═══════════════════════════════════════════════════════════════════════════
# POST-MERGE SMOKE — main pe kuch bhi merge hone pe.
# Sawaal ek hi hai: "kya app abhi bhi zinda hai?"
# Target: under 5 minutes. 6 flows ke pehle 3-4 steps.
# ═══════════════════════════════════════════════════════════════════════════
name: Smoke (post-merge)

on:
  push:
    branches: [main]
    paths-ignore:
      - '**.md'
      - 'docs/**'
  # Deploy system ke baad trigger karne ke liye (agar deploy alag pipeline hai)
  repository_dispatch:
    types: [deploy-completed]

# Production deploy verification hai — cancel-in-progress NAHI.
concurrency:
  group: smoke-${{ github.ref }}
  cancel-in-progress: false

permissions:
  contents: read

jobs:
  smoke:
    name: Smoke suite
    runs-on: ubuntu-latest
    timeout-minutes: 15
    environment:
      name: staging
      url: https://staging-app.merlinai.co
    steps:
      - uses: actions/checkout@v4

      # Composite action — saara setup ek step mein (Section 9.10)
      - uses: ./.github/actions/setup-python-playwright
        with:
          python-version: '3.11'
          browsers: 'chromium'

      # BASE_URL environment variable se aata hai, config file se NAHI.
      # Config file mein staging-api.merlinai.co jaisa dead host reh jaata hai.
      - name: Verify target is actually reachable before running tests
        run: |
          echo "Target: ${{ vars.BASE_URL }}"
          curl -sfI --max-time 10 "${{ vars.BASE_URL }}" > /dev/null \
            || { echo "::error::${{ vars.BASE_URL }} is not reachable"; exit 1; }

      - name: Run smoke suite
        run: |
          pytest tests/e2e \
            -m smoke \
            -v \
            --browser chromium \
            --tracing retain-on-failure \
            --screenshot only-on-failure \
            --video retain-on-failure \
            --junitxml=smoke-results.xml \
            --html=smoke-report.html --self-contained-html
        env:
          BASE_URL:      ${{ vars.BASE_URL }}
          API_URL:       ${{ vars.API_URL }}
          TEST_USER:     ${{ secrets.TEST_USER_EMAIL }}
          TEST_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
          CI: "true"

      # if: always() — screenshots aur traces ki zaroorat SIRF failure pe hai,
      # aur failure pe hi ye step skip ho jaata agar condition na hoti.
      - name: Upload smoke evidence
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: smoke-evidence-${{ github.run_number }}
          path: |
            smoke-report.html
            smoke-results.xml
            test-results/
            screenshots/
          retention-days: 14
          if-no-files-found: warn

      # Failure pe hi Slack — success spam se log notification mute kar dete hain
      - name: Notify Slack on failure
        if: failure()
        run: |
          curl -sS -X POST https://slack.com/api/chat.postMessage \
            -H "Authorization: Bearer ${{ secrets.SLACK_BOT_TOKEN }}" \
            -H "Content-type: application/json; charset=utf-8" \
            --data @- <<EOF
          {
            "channel": "${{ vars.SLACK_QA_CHANNEL }}",
            "text": ":rotating_light: Smoke FAILED on main",
            "blocks": [
              {
                "type": "header",
                "text": {"type": "plain_text", "text": ":rotating_light: Smoke suite FAILED"}
              },
              {
                "type": "section",
                "fields": [
                  {"type": "mrkdwn", "text": "*Commit:*\n\`${GITHUB_SHA:0:7}\`"},
                  {"type": "mrkdwn", "text": "*Author:*\n${{ github.actor }}"},
                  {"type": "mrkdwn", "text": "*Env:*\nstaging"},
                  {"type": "mrkdwn", "text": "*Run:*\n#${{ github.run_number }}"}
                ]
              },
              {
                "type": "section",
                "text": {
                  "type": "mrkdwn",
                  "text": "Smoke covers the first steps of all six PO/bid flows. A failure here means something fundamental is broken — treat as P0.\n<${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}|View logs and screenshots>"
                }
              }
            ]
          }
          EOF
```

### File 3 — `.github/workflows/nightly-regression.yml`

```yaml
# ═══════════════════════════════════════════════════════════════════════════
# NIGHTLY REGRESSION — poori suite, sharded.
# 6 E2E flows (PO material/trade/custom + bid material/trade/custom).
# Sharded so wall-clock time is the slowest shard, not the sum.
# ═══════════════════════════════════════════════════════════════════════════
name: Nightly Regression

on:
  schedule:
    # 20:30 UTC = 02:00 IST. Cron ALWAYS UTC — local time nahi.
    - cron: '30 20 * * *'

  # Manual trigger with inputs — koi bhi (PM bhi) targeted run chala sake
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        default: 'staging'
        type: choice
        options: [staging, production]
      suite:
        description: 'Test selection'
        required: true
        default: 'regression'
        type: choice
        options: [smoke, regression, all]
      shards:
        description: 'Number of parallel shards'
        required: false
        default: '6'
        type: string
      browser:
        description: 'Browser'
        required: false
        default: 'chromium'
        type: choice
        options: [chromium, firefox, webkit]

permissions:
  contents: read

jobs:
  # ─────────────────────────────────────────────────────────────────────
  # Job 1: Shard matrix dynamically banao (input se shard count aata hai)
  # ─────────────────────────────────────────────────────────────────────
  prepare:
    name: Prepare shard matrix
    runs-on: ubuntu-latest
    outputs:
      matrix:       ${{ steps.gen.outputs.matrix }}
      shard_total:  ${{ steps.gen.outputs.total }}
      marker:       ${{ steps.gen.outputs.marker }}
    steps:
      - id: gen
        run: |
          TOTAL="${{ inputs.shards || '6' }}"
          # [1,2,...,N] ka JSON array banao — matrix isi se banega
          MATRIX=$(seq -s, 1 "$TOTAL" | sed 's/^/[/; s/$/]/')
          echo "matrix=$MATRIX"  >> "$GITHUB_OUTPUT"
          echo "total=$TOTAL"    >> "$GITHUB_OUTPUT"

          # 'all' ka matlab koi marker filter nahi
          SUITE="${{ inputs.suite || 'regression' }}"
          if [ "$SUITE" = "all" ]; then
            echo "marker=" >> "$GITHUB_OUTPUT"
          else
            echo "marker=-m $SUITE" >> "$GITHUB_OUTPUT"
          fi

  # ─────────────────────────────────────────────────────────────────────
  # Job 2: Sharded execution
  # ─────────────────────────────────────────────────────────────────────
  regression:
    name: Shard ${{ matrix.shard }}
    needs: prepare
    runs-on: ubuntu-latest
    timeout-minutes: 45
    strategy:
      # fail-fast: false — CRITICAL for QA.
      # Default true hota hai, matlab ek shard fail hote hi baaki cancel.
      # Tab tumhe pata hi nahi chalta ki 1 test toota ya 100.
      fail-fast: false
      max-parallel: 6
      matrix:
        shard: ${{ fromJSON(needs.prepare.outputs.matrix) }}

    steps:
      - uses: actions/checkout@v4

      - uses: ./.github/actions/setup-python-playwright
        with:
          python-version: '3.11'
          browsers: ${{ inputs.browser || 'chromium' }}

      - name: Pre-flight — target reachable?
        run: |
          URL="${{ inputs.environment == 'production' && vars.PROD_BASE_URL || vars.BASE_URL }}"
          echo "Target: $URL"
          curl -sfI --max-time 10 "$URL" > /dev/null \
            || { echo "::error::$URL unreachable — aborting before wasting 40 min"; exit 1; }

      - name: Run regression shard ${{ matrix.shard }}/${{ needs.prepare.outputs.shard_total }}
        run: |
          pytest tests/e2e \
            ${{ needs.prepare.outputs.marker }} \
            --splits ${{ needs.prepare.outputs.shard_total }} \
            --group ${{ matrix.shard }} \
            --splitting-algorithm least_duration \
            --durations-path .test_durations \
            --browser ${{ inputs.browser || 'chromium' }} \
            --tracing retain-on-failure \
            --screenshot only-on-failure \
            --video retain-on-failure \
            --junitxml=results-${{ matrix.shard }}.xml \
            --html=report-${{ matrix.shard }}.html --self-contained-html \
            -v
        env:
          BASE_URL:      ${{ inputs.environment == 'production' && vars.PROD_BASE_URL || vars.BASE_URL }}
          API_URL:       ${{ inputs.environment == 'production' && vars.PROD_API_URL  || vars.API_URL }}
          TEST_USER:     ${{ secrets.TEST_USER_EMAIL }}
          TEST_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
          CI: "true"

      - name: Upload shard evidence
        if: always()          # <-- without this, failures upload nothing
        uses: actions/upload-artifact@v4
        with:
          name: regression-shard-${{ matrix.shard }}
          path: |
            results-${{ matrix.shard }}.xml
            report-${{ matrix.shard }}.html
            test-results/
            screenshots/
          retention-days: 14
          if-no-files-found: warn

  # ─────────────────────────────────────────────────────────────────────
  # Job 3: Reports merge karo + ek Slack summary bhejo
  # if: always() — kyunki hum FAILURE ki report bhi chahte hain
  # ─────────────────────────────────────────────────────────────────────
  report:
    name: Merge reports and notify
    needs: [prepare, regression]
    if: always()
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Download every shard's results
        uses: actions/download-artifact@v4
        with:
          pattern: regression-shard-*
          path: shards/
          merge-multiple: true

      - name: Parse results into a summary
        id: summary
        run: |
          pip install junitparser
          python - <<'PY' >> "$GITHUB_OUTPUT"
          import glob
          from junitparser import JUnitXml

          total = failed = errored = skipped = 0
          failures = []

          for path in glob.glob("shards/results-*.xml"):
              xml = JUnitXml.fromfile(path)
              for suite in xml:
                  total   += suite.tests
                  failed  += suite.failures
                  errored += suite.errors
                  skipped += suite.skipped
                  for case in suite:
                      if case.result and case.result[0].__class__.__name__ in ("Failure", "Error"):
                          failures.append(case.name)

          passed = total - failed - errored - skipped
          rate = (passed / total * 100) if total else 0.0

          print(f"total={total}")
          print(f"passed={passed}")
          print(f"failed={failed + errored}")
          print(f"skipped={skipped}")
          print(f"rate={rate:.1f}")
          # Multi-line output ke liye heredoc syntax
          print("failures<<EOF")
          for name in failures[:15]:
              print(f"- {name}")
          if len(failures) > 15:
              print(f"...and {len(failures) - 15} more")
          print("EOF")
          PY

      - name: Publish combined report as an artifact
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: regression-combined-${{ github.run_number }}
          path: shards/
          retention-days: 30

      # Nightly ke liye success bhi post karo — team ko roz health signal chahiye
      - name: Post summary to Slack
        run: |
          if [ "${{ needs.regression.result }}" = "success" ]; then
            EMOJI=":white_check_mark:"; STATUS="PASSED"
          else
            EMOJI=":x:"; STATUS="FAILED"
          fi

          curl -sS -X POST https://slack.com/api/chat.postMessage \
            -H "Authorization: Bearer ${{ secrets.SLACK_BOT_TOKEN }}" \
            -H "Content-type: application/json; charset=utf-8" \
            --data @- <<EOF
          {
            "channel": "${{ vars.SLACK_QA_CHANNEL }}",
            "text": "${EMOJI} Nightly regression ${STATUS}",
            "blocks": [
              {"type":"header","text":{"type":"plain_text","text":"${EMOJI} Nightly Regression — ${STATUS}"}},
              {"type":"section","fields":[
                {"type":"mrkdwn","text":"*Environment:*\n${{ inputs.environment || 'staging' }}"},
                {"type":"mrkdwn","text":"*Pass rate:*\n${{ steps.summary.outputs.rate }}%"},
                {"type":"mrkdwn","text":"*Passed:*\n${{ steps.summary.outputs.passed }}/${{ steps.summary.outputs.total }}"},
                {"type":"mrkdwn","text":"*Failed:*\n${{ steps.summary.outputs.failed }}"}
              ]},
              {"type":"section","text":{"type":"mrkdwn","text":"*Failing tests:*\n\`\`\`${{ steps.summary.outputs.failures }}\`\`\`"}},
              {"type":"actions","elements":[
                {"type":"button","text":{"type":"plain_text","text":"View run + screenshots"},
                 "url":"${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"}
              ]}
            ]
          }
          EOF

      # Nightly regression fail hui to workflow ko red dikhao
      - name: Fail the workflow if any shard failed
        if: needs.regression.result != 'success'
        run: exit 1
```

### Setup checklist — ye workflows chalane ke liye kya-kya chahiye

```
[ ] .github/workflows/ directory bani
[ ] Repo Settings > Secrets and variables > Actions > Secrets:
      TEST_USER_EMAIL
      TEST_USER_PASSWORD
      SLACK_BOT_TOKEN
[ ] Repo Settings > Secrets and variables > Actions > Variables:
      BASE_URL           = https://staging-app.merlinai.co
      API_URL            = https://staging-eks.merlinai.co      <-- NOT staging-api (dead)
      PROD_BASE_URL      = https://app.merlinai.co
      PROD_API_URL       = https://api.merlinai.co
      SLACK_QA_CHANNEL   = C0XXXXXXX
[ ] requirements-dev.txt mein: ruff, mypy, pytest-cov, diff-cover,
    pytest-split, pytest-html, junitparser
[ ] pytest.ini / pyproject.toml mein markers registered:
      markers =
        smoke: fast critical-path subset
        regression: full suite
        po: purchase order flows
        bid: bid flows
[ ] .test_durations file generate karke commit ki (sharding balance ke liye)
[ ] .github/actions/setup-python-playwright/action.yml banayi
[ ] Repo Settings > Environments: `staging` aur `production` banaye,
    production pe required reviewers set
[ ] Repo Settings > Branches > branch protection on `main`:
      - Require PR before merging
      - Require status checks: static, secrets-scan, unit, suite-health
      - Require branches up to date before merging
```

---

# 10. Jenkins Basics

## 10.1 Kyun abhi bhi padhna hai

Jenkins 2011 ka hai aur log kehte hain "purana ho gaya". Par reality:
- Enterprise mein **abhi bhi sabse zyada deployed** CI tool hai
- Banks, insurance, telecom, healthcare — jahan on-prem requirement hai, wahan Jenkins hi hai
- **Interview mein poochha jaata hai** — chahe company GitHub Actions use karti ho, interviewer Jenkins ka sawaal daal deta hai kyunki wo classic hai

Tumhe Jenkins expert banne ki zaroorat nahi. Tumhe **declarative Jenkinsfile padhna aur likhna** aana chahiye, aur GitHub Actions se mapping pata honi chahiye. Bas.

## 10.2 Core concepts

| Concept | Kya hai | GH Actions equivalent |
|---|---|---|
| **Controller (master)** | Jenkins server, orchestration karta hai | GitHub ka backend |
| **Agent (node/slave)** | Machine jahan actual kaam hota hai | Runner |
| **Job / Pipeline** | Ek automation unit | Workflow |
| **Jenkinsfile** | Pipeline as code, repo mein rehta hai | `.github/workflows/*.yml` |
| **Stage** | Pipeline ka logical hissa | Job |
| **Step** | Ek command/action | Step |
| **Plugin** | Functionality extension (1800+ hain) | Marketplace action |
| **Credentials** | Secret store | Secrets |
| **Multibranch pipeline** | Har branch ka apna pipeline auto-discover | Automatic |

**Do syntax hain:**
- **Declarative** — structured, `pipeline { }` block. **Ye seekho.** Modern, readable, most common.
- **Scripted** — Groovy code, `node { }` block. Purana, powerful, par messy.

## 10.3 Declarative Jenkinsfile anatomy

```groovy
pipeline {
    // AGENT — kahan chalega
    agent any                          // koi bhi available agent
    // agent { label 'linux && playwright' }     // specific label
    // agent { docker { image 'python:3.11' } }  // container mein
    // agent none                                // stage-level agents use karo

    // OPTIONS — pipeline behaviour
    options {
        timeout(time: 60, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '30'))
        disableConcurrentBuilds()
        timestamps()                   // har log line pe timestamp
        ansiColor('xterm')             // coloured output
    }

    // PARAMETERS — workflow_dispatch inputs ka equivalent
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['staging', 'production'],
               description: 'Target environment')
        choice(name: 'SUITE', choices: ['smoke', 'regression', 'all'],
               description: 'Test selection')
        string(name: 'SHARDS', defaultValue: '6', description: 'Parallel shards')
        booleanParam(name: 'SKIP_REGRESSION', defaultValue: false,
                     description: 'EMERGENCY ONLY')
    }

    // TRIGGERS
    triggers {
        cron('30 20 * * *')            // nightly
        pollSCM('H/5 * * * *')         // har 5 min git check (webhook better hai)
        // upstream(upstreamProjects: 'deploy-job', threshold: hudson.model.Result.SUCCESS)
    }

    // ENVIRONMENT — env vars
    environment {
        PYTHONUNBUFFERED = '1'
        CI = 'true'
        // credentials() helper — Jenkins credential store se uthata hai
        SLACK_BOT_TOKEN = credentials('slack-bot-token')
        // username/password credential do vars banata hai:
        //   TEST_CREDS_USR aur TEST_CREDS_PSW
        TEST_CREDS = credentials('merlin-test-user')
    }

    // TOOLS — Jenkins-managed installations
    tools {
        // jdk 'jdk17'
        // gradle 'gradle8'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        // ... aur stages
    }

    // POST — stages ke BAAD chalta hai, HAMESHA
    post {
        always   { echo 'Runs no matter what' }
        success  { echo 'Only if everything passed' }
        failure  { echo 'Only if something failed' }
        unstable { echo 'Tests failed but build succeeded' }
        aborted  { echo 'Cancelled or timed out' }
        changed  { echo 'Result differs from previous build' }
        cleanup  { echo 'Runs last, after all other post conditions' }
    }
}
```

**`post` block GitHub Actions ke `if: always()` ka equivalent hai** — aur wahi importance rakhta hai. Artifact archiving aur report publishing hamesha `post { always { } }` mein hona chahiye, warna failure pe skip ho jaayega.

**`unstable` ka concept GH Actions mein nahi hai** — Jenkins mein build "failed" (build hi toota) aur "unstable" (build theek, tests fail) alag hain. Ye actually useful distinction hai.

## 10.4 Full Jenkinsfile — GH Actions workflow ka equivalent

```groovy
// ═══════════════════════════════════════════════════════════════════════
// Merlin AI — Playwright/pytest E2E pipeline
// Section 9.12 wale GitHub Actions setup ke barabar
// ═══════════════════════════════════════════════════════════════════════
pipeline {
    agent none                          // har stage apna agent chunega

    options {
        timeout(time: 90, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '30', artifactNumToKeepStr: '10'))
        disableConcurrentBuilds()
        timestamps()
        skipDefaultCheckout(true)       // hum manually checkout karenge
    }

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['staging', 'production'],
               description: 'Which environment to test against')
        choice(name: 'SUITE', choices: ['smoke', 'regression', 'all'],
               description: 'Which tests to run')
        string(name: 'SHARDS', defaultValue: '6',
               description: 'Number of parallel shards')
    }

    triggers {
        cron('30 20 * * *')             // 20:30 UTC = 02:00 IST
    }

    environment {
        PYTHONUNBUFFERED = '1'
        CI               = 'true'
        // Credential IDs Jenkins > Manage Jenkins > Credentials mein bane hain
        TEST_CREDS       = credentials('merlin-test-user')   // _USR aur _PSW
        SLACK_BOT_TOKEN  = credentials('slack-bot-token')
        BASE_URL         = "${params.ENVIRONMENT == 'production' ? 'https://app.merlinai.co' : 'https://staging-app.merlinai.co'}"
        // NOTE: staging-api.merlinai.co DEAD hai. Asli host staging-eks hai.
        API_URL          = "${params.ENVIRONMENT == 'production' ? 'https://api.merlinai.co' : 'https://staging-eks.merlinai.co'}"
    }

    stages {

        // ─────────────────────────────────────────────────────────
        // Fast checks — sab parallel, ek saath
        // ─────────────────────────────────────────────────────────
        stage('Static analysis') {
            agent { docker { image 'python:3.11-slim'; reuseNode true } }
            steps {
                checkout scm
                sh 'pip install --no-cache-dir -r requirements-dev.txt'
            }
            // NESTED PARALLEL — teeno ek saath
            stages {
                stage('Parallel static checks') {
                    parallel {
                        stage('Lint')       { steps { sh 'ruff check . --output-format=github' } }
                        stage('Format')     { steps { sh 'ruff format --check .' } }
                        stage('Types')      { steps { sh 'mypy tests/ pages/ utils/ --ignore-missing-imports' } }
                        stage('Secrets')    {
                            steps {
                                // Committed credentials pakadne ke liye — ye
                                // wahi gate hai jo plaintext Mongo/QuickBooks
                                // secrets ko PR pe hi rok deti.
                                sh '''
                                    docker run --rm -v "$PWD:/repo" \
                                      zricethezav/gitleaks:latest detect \
                                      --source=/repo --no-git -v
                                '''
                            }
                        }
                    }
                }
            }
        }

        // ─────────────────────────────────────────────────────────
        // Unit tests + coverage gate
        // ─────────────────────────────────────────────────────────
        stage('Unit tests') {
            agent { docker { image 'python:3.11-slim'; reuseNode true } }
            steps {
                sh '''
                    pip install --no-cache-dir -r requirements-dev.txt
                    pytest tests/unit \
                      --cov=. --cov-report=xml \
                      --junitxml=unit-results.xml -v
                '''
                sh 'diff-cover coverage.xml --compare-branch=origin/main --fail-under=80'
            }
            post {
                always {
                    // junit step build ko UNSTABLE marks karta hai agar tests fail hue
                    junit testResults: 'unit-results.xml', allowEmptyResults: false
                    publishCoverage adapters: [coberturaAdapter('coverage.xml')]
                }
            }
        }

        // ─────────────────────────────────────────────────────────
        // Smoke — pehle. Fail hua to regression chalane ka fayda nahi.
        // ─────────────────────────────────────────────────────────
        stage('Smoke') {
            agent {
                docker {
                    // Official Playwright Python image — browsers pre-installed
                    image 'mcr.microsoft.com/playwright/python:v1.47.0-jammy'
                    args  '--ipc=host'     // Chromium ke shared memory crashes rokta hai
                    reuseNode true
                }
            }
            steps {
                sh '''
                    pip install --no-cache-dir -r requirements.txt
                    pytest tests/e2e -m smoke \
                      --browser chromium \
                      --screenshot only-on-failure \
                      --tracing retain-on-failure \
                      --junitxml=smoke-results.xml \
                      --html=smoke-report.html --self-contained-html \
                      -v
                '''
            }
            post {
                always {
                    junit testResults: 'smoke-results.xml', allowEmptyResults: true
                    archiveArtifacts artifacts: 'smoke-report.html, test-results/**, screenshots/**',
                                     allowEmptyArchive: true, fingerprint: true
                }
            }
        }

        // ─────────────────────────────────────────────────────────
        // Sharded regression — DYNAMIC parallel stages
        // Ye Groovy code hai jo runtime pe N parallel branches banata hai
        // ─────────────────────────────────────────────────────────
        stage('Regression (sharded)') {
            when {
                // Sirf tab jab suite smoke nahi hai
                expression { params.SUITE != 'smoke' }
            }
            steps {
                script {
                    def total  = params.SHARDS.toInteger()
                    def marker = params.SUITE == 'all' ? '' : "-m ${params.SUITE}"
                    def branches = [:]

                    for (int i = 1; i <= total; i++) {
                        def shard = i                    // closure capture ke liye local var
                        branches["shard-${shard}"] = {
                            node('linux') {
                                docker.image('mcr.microsoft.com/playwright/python:v1.47.0-jammy')
                                      .inside('--ipc=host') {
                                    checkout scm
                                    sh """
                                        pip install --no-cache-dir -r requirements.txt
                                        pytest tests/e2e ${marker} \
                                          --splits ${total} --group ${shard} \
                                          --splitting-algorithm least_duration \
                                          --durations-path .test_durations \
                                          --browser chromium \
                                          --screenshot only-on-failure \
                                          --tracing retain-on-failure \
                                          --junitxml=results-${shard}.xml \
                                          --html=report-${shard}.html --self-contained-html \
                                          -v
                                    """
                                    junit testResults: "results-${shard}.xml", allowEmptyResults: true
                                    archiveArtifacts artifacts: "report-${shard}.html, test-results/**",
                                                     allowEmptyArchive: true
                                }
                            }
                        }
                    }
                    // failFast: false — ek shard fail hone pe baaki chalte rahen.
                    // GH Actions ke `fail-fast: false` ke barabar. QA ke liye critical.
                    branches.failFast = false
                    parallel branches
                }
            }
        }

        // ─────────────────────────────────────────────────────────
        // Approval gate — continuous delivery ka human gate
        // ─────────────────────────────────────────────────────────
        stage('Approve production deploy') {
            when {
                allOf {
                    branch 'main'
                    expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
                }
            }
            steps {
                timeout(time: 4, unit: 'HOURS') {
                    input message: 'Deploy this build to production?',
                          ok: 'Deploy',
                          submitter: 'release-managers',       // sirf ye group
                          parameters: [
                              text(name: 'RELEASE_NOTES', defaultValue: '',
                                   description: 'What is in this release?')
                          ]
                }
            }
        }
    }

    // ─────────────────────────────────────────────────────────────
    // POST — GH Actions ke `if: always()` ka equivalent
    // ─────────────────────────────────────────────────────────────
    post {
        always {
            node('linux') {
                script {
                    def status = currentBuild.currentResult      // SUCCESS/UNSTABLE/FAILURE
                    def emoji  = [SUCCESS: ':white_check_mark:',
                                  UNSTABLE: ':warning:',
                                  FAILURE: ':x:',
                                  ABORTED: ':black_square_for_stop:'][status] ?: ':grey_question:'
                    sh """
                        curl -sS -X POST https://slack.com/api/chat.postMessage \
                          -H "Authorization: Bearer \$SLACK_BOT_TOKEN" \
                          -H 'Content-type: application/json' \
                          -d '{
                            "channel": "#qa-automation",
                            "text": "${emoji} Merlin E2E — ${status} — ${params.ENVIRONMENT} — build ${env.BUILD_NUMBER}\\n${env.BUILD_URL}"
                          }'
                    """
                }
            }
        }
        unstable {
            echo 'Tests failed but the build itself was fine — triage required.'
        }
        cleanup {
            node('linux') { cleanWs() }        // workspace saaf karo
        }
    }
}
```

## 10.5 Jenkins ke important patterns

### Credentials binding

```groovy
// Method 1: environment block (simplest)
environment {
    API_KEY = credentials('my-api-key')
}

// Method 2: withCredentials (scoped, zyada control)
steps {
    withCredentials([
        usernamePassword(credentialsId: 'merlin-test-user',
                         usernameVariable: 'TEST_USER',
                         passwordVariable: 'TEST_PASSWORD'),
        string(credentialsId: 'slack-bot-token', variable: 'SLACK_TOKEN'),
        file(credentialsId: 'gcp-service-account', variable: 'GOOGLE_CREDS')
    ]) {
        sh 'pytest tests/e2e'
    }
    // credentials sirf is block ke andar available hain
}
```

Jenkins bhi logs mein credentials mask karta hai. Par wahi bypasses hain — `set -x` mat use karo.

### when conditions

```groovy
stage('Deploy') {
    when {
        allOf {
            branch 'main'
            not { changeRequest() }                       // PR nahi hai
            environment name: 'DEPLOY_ENABLED', value: 'true'
            expression { params.SUITE == 'regression' }
            changeset "src/**"                            // in files mein change hua
        }
    }
    steps { sh './deploy.sh' }
}
```

### Shared libraries (reusable workflow ka equivalent)

`vars/runPlaywrightSuite.groovy` ek shared library repo mein:

```groovy
def call(Map config = [:]) {
    def suite   = config.suite   ?: 'smoke'
    def shards  = config.shards  ?: 4
    def browser = config.browser ?: 'chromium'

    docker.image('mcr.microsoft.com/playwright/python:v1.47.0-jammy')
          .inside('--ipc=host') {
        sh """
            pip install -r requirements.txt
            pytest tests/e2e -m ${suite} --browser ${browser} \
              --junitxml=results.xml -v
        """
    }
}
```

Jenkinsfile mein:

```groovy
@Library('merlin-ci-lib@v1') _

pipeline {
    agent any
    stages {
        stage('E2E') {
            steps { runPlaywrightSuite(suite: 'regression', shards: 6) }
        }
    }
}
```

## 10.6 Jenkins vs GitHub Actions — comparison table

| Dimension | Jenkins | GitHub Actions |
|---|---|---|
| **Hosting** | Self-hosted (tum server chalate ho) | SaaS (self-hosted runners optional) |
| **Setup effort** | Zyada — server, plugins, agents, backups | Bahut kam — file commit karo, ho gaya |
| **Config language** | Groovy (Jenkinsfile) | YAML |
| **Learning curve** | Steep — Groovy + plugin ecosystem | Gentle |
| **Maintenance** | **Tumhara kaam** — upgrades, plugin conflicts, disk, security patches | GitHub ka kaam |
| **Cost model** | Infra ka kharcha + engineer ka time | Free tier + per-minute billing |
| **Ecosystem** | 1800+ plugins, kuch bhi mil jaata hai | 20k+ marketplace actions |
| **Plugin quality** | **Variable** — kuch abandoned hain, conflicts hote hain | Variable, par versioning saaf |
| **VCS support** | Kuchh bhi — Git, SVN, Perforce, Mercurial | GitHub-centric (dusre VCS awkward) |
| **On-prem / air-gapped** | **Excellent** — poori tarah offline chal sakta hai | Enterprise Server chahiye |
| **Secrets** | Credentials plugin, Vault integration | Built-in secrets + environments |
| **Parallelism** | `parallel {}` block, dynamic Groovy se | `strategy.matrix`, declarative |
| **Dynamic pipelines** | **Strong** — Groovy runtime pe kuch bhi bana sakta hai | Limited — `fromJSON` se workaround |
| **UI** | Purana, cluttered (Blue Ocean try karo) | Modern, clean |
| **Debugging** | SSH into agent, workspace inspect karo | Logs + `tmate` action hack |
| **PR integration** | Plugin chahiye, config effort | **Native, zero config** |
| **Kab choose karo** | On-prem mandatory, non-GitHub VCS, complex dynamic pipelines, already invested | GitHub pe ho, team chhoti-medium, maintenance nahi chahte |

## 10.7 Interview answer

> **Interview answer:**
> "I've worked with GitHub Actions as my primary CI, and I can read and write a declarative Jenkinsfile — the concepts map cleanly, so moving between them is mostly syntax.
>
> The mapping is: a Jenkins stage is roughly a GitHub Actions job, a step is a step, an agent is a runner, and the `post` block is the equivalent of `if: always()`. Jenkins credentials binding, either through the `environment` block or `withCredentials`, is the equivalent of the secrets store, and both mask values in logs with the same caveat — masking is string matching, so anything that transforms the secret, like base64 or a verbose curl, defeats it.
>
> Architecturally the difference is that Jenkins is a server you own and Actions is a service you consume. Jenkins gives you more power — dynamic pipeline generation in Groovy is genuinely more capable than a YAML matrix, and it runs fully air-gapped, which some regulated environments require. The cost is that you own the upgrades, the plugin conflicts, the disk space and the security patching, and in most teams that ends up being someone's part-time job.
>
> One Jenkins concept I actually miss in Actions is the distinction between a failed build and an unstable one. In Jenkins, 'failed' means the build itself broke, and 'unstable' means the build was fine but tests failed. That's a genuinely useful distinction for QA — it separates 'the pipeline is broken' from 'the product has a regression', which are different problems for different people. In Actions everything is just red, so I end up encoding that distinction in the notification instead."

**Cross-question: "Agar tumhe aaj se ek naye project pe CI choose karna ho to kaunsa loge?"**

> **Interview answer:**
> "For a team already on GitHub, I'd pick GitHub Actions almost without hesitation, and the reason is maintenance cost rather than features. CI is infrastructure, and infrastructure that a QA has to babysit is time not spent on testing. With Actions the pipeline is a file in the repo, it's versioned with the code, the PR integration is native, and nobody has to patch a server.
>
> I'd choose Jenkins in three cases. If the environment must be air-gapped or on-premise for compliance. If the team already has significant Jenkins investment — shared libraries, established agents — because migration cost is real and rarely worth it just for aesthetics. Or if the pipelines need genuinely dynamic generation that YAML can't express well.
>
> What I'd care about more than the tool is the properties: the pipeline is defined as code in the repo, it's reproducible locally in some form, secrets are in a proper store, and the fast gate stays under ten minutes. Those matter far more than which tool renders the logs."

---

# 11. GitLab CI (brief)

Agar company GitLab pe hai to `.gitlab-ci.yml` repo root mein hoti hai (`.github/workflows/` jaisa subdirectory nahi).

```yaml
# .gitlab-ci.yml — Merlin suite ka equivalent

stages:                              # execution order — stages sequential hain
  - static
  - test
  - e2e
  - notify

# Default settings jo saare jobs ko milte hain
default:
  image: python:3.11-slim
  before_script:
    - pip install --no-cache-dir -r requirements.txt
  interruptible: true                # naya pipeline aane pe cancel

variables:
  PIP_CACHE_DIR: "$CI_PROJECT_DIR/.cache/pip"
  PYTHONUNBUFFERED: "1"
  CI: "true"

# YAML anchors — reusable blocks (composite action jaisa)
.python_cache: &python_cache
  cache:
    key:
      files: [requirements.txt]      # requirements badle to cache invalidate
    paths:
      - .cache/pip
      - .venv/

# ─────────────────────────────────────────────────────────
lint:
  stage: static
  <<: *python_cache
  script:
    - pip install ruff mypy
    - ruff check .
    - ruff format --check .
    - mypy tests/ pages/ --ignore-missing-imports
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

secret-scan:
  stage: static
  image:
    name: zricethezav/gitleaks:latest
    entrypoint: [""]
  before_script: []                  # default before_script override
  script:
    - gitleaks detect --source . -v
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

# ─────────────────────────────────────────────────────────
unit:
  stage: test
  <<: *python_cache
  script:
    - pytest tests/unit --cov=. --cov-report=xml --junitxml=unit.xml
  coverage: '/TOTAL.*\s+(\d+%)$/'    # regex se coverage % nikaal ke MR pe dikhata hai
  artifacts:
    when: always                     # <-- GH Actions ke if: always() ka equivalent
    reports:
      junit: unit.xml                # GitLab test results ko natively parse karta hai
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml
    expire_in: 1 week

# ─────────────────────────────────────────────────────────
e2e:
  stage: e2e
  image: mcr.microsoft.com/playwright/python:v1.47.0-jammy
  parallel: 6                        # <-- sharding. 6 jobs banata hai.
                                     # CI_NODE_INDEX (1..6) aur CI_NODE_TOTAL milte hain
  script:
    - |
      pytest tests/e2e -m regression \
        --splits "$CI_NODE_TOTAL" \
        --group "$CI_NODE_INDEX" \
        --splitting-algorithm least_duration \
        --browser chromium \
        --screenshot only-on-failure \
        --tracing retain-on-failure \
        --junitxml="results-${CI_NODE_INDEX}.xml"
  variables:
    BASE_URL: "https://staging-app.merlinai.co"
    API_URL:  "https://staging-eks.merlinai.co"
    # TEST_USER / TEST_PASSWORD: Settings > CI/CD > Variables mein (masked+protected)
  artifacts:
    when: always
    paths:
      - test-results/
      - screenshots/
    reports:
      junit: "results-*.xml"
    expire_in: 2 weeks
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
    - if: $CI_PIPELINE_SOURCE == "schedule"
  retry:
    max: 1
    when: [runner_system_failure, stuck_or_timeout_failure]   # infra failure pe retry,
                                                              # test failure pe NAHI

# ─────────────────────────────────────────────────────────
notify:
  stage: notify
  when: always                       # pipeline fail hone pe bhi chalega
  script:
    - |
      curl -X POST https://slack.com/api/chat.postMessage \
        -H "Authorization: Bearer $SLACK_BOT_TOKEN" \
        -H 'Content-type: application/json' \
        -d "{\"channel\":\"#qa-automation\",\"text\":\"Pipeline $CI_PIPELINE_STATUS — $CI_PIPELINE_URL\"}"

# ─────────────────────────────────────────────────────────
deploy-production:
  stage: notify
  environment:
    name: production
    url: https://app.merlinai.co
  when: manual                       # <-- approval gate
  only:
    - main
  script:
    - ./deploy.sh
```

**Concept mapping — teeno tools:**

| Concept | GitHub Actions | Jenkins | GitLab CI |
|---|---|---|---|
| File location | `.github/workflows/*.yml` | `Jenkinsfile` (root) | `.gitlab-ci.yml` (root) |
| Unit of work | job | stage | job |
| Ordering | `needs:` | stage order | `stages:` + `needs:` |
| Parallelism | `strategy.matrix` | `parallel {}` | `parallel: N` |
| Always-run | `if: always()` | `post { always {} }` | `when: always` |
| Secrets | `secrets.X` | `credentials('id')` | CI/CD Variables (masked) |
| Manual gate | `environment` + reviewers | `input` step | `when: manual` |
| Reuse | reusable workflow / composite | shared library | `include:` / anchors |
| Artifacts | `upload-artifact` | `archiveArtifacts` | `artifacts:` |
| Manual trigger | `workflow_dispatch` | `parameters` | `when: manual` + variables |

---

# 12. Docker for QA + Setup Steps

## 12.1 Container kya hai — aur "works on my machine" kaise fix karta hai

**Kya hai:** Container ek isolated process hai jo apne saath **poora filesystem** leke chalta hai — OS libraries, language runtime, dependencies, tumhara code, sab kuchh. Wo host machine ka **kernel** share karta hai, par baaki sab apna hai.

**VM se farq (interview classic):**

```
   VIRTUAL MACHINES                     CONTAINERS
  ┌──────┬──────┬──────┐              ┌──────┬──────┬──────┐
  │ App  │ App  │ App  │              │ App  │ App  │ App  │
  ├──────┼──────┼──────┤              ├──────┼──────┼──────┤
  │ Libs │ Libs │ Libs │              │ Libs │ Libs │ Libs │
  ├──────┼──────┼──────┤              └──────┴──────┴──────┘
  │Guest │Guest │Guest │              ┌────────────────────┐
  │  OS  │  OS  │  OS  │  <-- heavy   │  Container Engine  │
  ├──────┴──────┴──────┤   (GBs,      ├────────────────────┤
  │     Hypervisor     │    minutes)  │      Host OS       │  <-- shared kernel
  ├────────────────────┤              ├────────────────────┤     (MBs, seconds)
  │      Host OS       │              │      Hardware      │
  ├────────────────────┤              └────────────────────┘
  │      Hardware      │
  └────────────────────┘
```

VM apna poora OS chalata hai (GBs, boot mein minutes). Container host ka kernel share karta hai (MBs, start mein milliseconds).

**"Works on my machine" kaise fix hota hai:**

Ye problem hoti kyun hai? Kyunki tumhare laptop pe aur CI runner pe ye cheezein alag hain:

| Difference | Example |
|---|---|
| OS / distro | tum macOS pe, CI Ubuntu pe |
| Python version | tum 3.12, CI 3.11 |
| System libraries | tumhare paas `libnss3` hai, CI pe nahi → Chromium crash |
| Fonts | tumhare paas system fonts hain, CI pe nahi → **screenshot diffs, text wrapping alag** |
| Locale / timezone | tum IST, CI UTC → **date assertions fail** |
| Env vars | tumhare shell profile mein set hain, CI pe nahi |
| Installed tools | tumne globally kuchh install kiya tha 6 mahine pehle, bhool gaye |

Container ye saari differences **image mein freeze** kar deta hai. Tum aur CI **byte-for-byte same environment** chalate ho.

**QA ke liye ye khaas kyun matter karta hai:** Fonts aur locale. Ye do cheezein UI test flakiness ke silent killers hain. Alag fonts pe text alag width leta hai → element position shift → click galat jagah. Alag timezone pe date formatting alag → assertion fail. Container inhe deterministic bana deta hai.

> **Interview answer:**
> "A container is an isolated process that ships with its own filesystem — the OS libraries, the language runtime, the dependencies and the application — while sharing the host kernel. That's the difference from a VM: a VM boots a whole guest operating system, which is gigabytes and minutes; a container starts in milliseconds and is measured in megabytes.
>
> For QA the value is determinism. 'Works on my machine' happens because the machine differs in ways nobody has written down — a different Python patch version, a missing system library that Chromium needs, a different timezone, and most insidiously, different fonts. Fonts and locale are underrated sources of UI test flakiness: different font metrics change text width, which changes element positions, which changes where a click lands, and a different timezone breaks date assertions in ways that look random.
>
> Putting the test run in a container freezes all of that into the image, so my laptop and the CI runner execute in byte-identical environments. That converts a whole class of environment-dependent failures into a class that either always happens or never happens — which is exactly what you want, because a deterministic failure is debuggable and an intermittent one isn't."

---

## 12.2 Setup — Docker install karna aur pehla container chalana

### Step 1: Docker install

macOS:
```bash
brew install --cask docker
# Docker Desktop app kholo, wo daemon start karega
```

Linux (Ubuntu):
```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker "$USER"     # sudo ke bina chalane ke liye
newgrp docker                        # ya logout/login
```

### Step 2: Verify

```bash
docker --version
docker run --rm hello-world
```

### Step 3: Playwright Python image try karo

```bash
docker run --rm -it mcr.microsoft.com/playwright/python:v1.47.0-jammy python -c "
from playwright.sync_api import sync_playwright
with sync_playwright() as p:
    b = p.chromium.launch()
    page = b.new_page()
    page.goto('https://example.com')
    print('Title:', page.title())
    b.close()
"
```

Agar ye chal gaya to **tumhare paas ek working, reproducible test environment hai** — bina kuchh install kiye.

### Step 4: Apna suite container mein chalao (bina Dockerfile ke)

```bash
docker run --rm -it \
  -v "$PWD":/work \
  -w /work \
  --ipc=host \
  -e BASE_URL="https://staging-app.merlinai.co" \
  -e TEST_USER="$TEST_USER" \
  -e TEST_PASSWORD="$TEST_PASSWORD" \
  mcr.microsoft.com/playwright/python:v1.47.0-jammy \
  bash -c "pip install -r requirements.txt && pytest tests/e2e -m smoke -v"
```

Flags samjho:

| Flag | Kya karta hai |
|---|---|
| `--rm` | Container exit hone pe delete kar do (disk bharne se bachao) |
| `-it` | Interactive + TTY (output theek dikhta hai, Ctrl-C kaam karta hai) |
| `-v "$PWD":/work` | Current directory ko container ke `/work` pe mount karo |
| `-w /work` | Working directory set karo |
| `--ipc=host` | **Chromium ke liye zaroori.** Neeche detail. |
| `-e KEY=value` | Environment variable pass karo |

**`--ipc=host` kyun zaroori hai (interview mein poochha jaata hai):** Chromium `/dev/shm` (shared memory) use karta hai. Docker default mein use sirf **64MB** deta hai. Chromium bade pages pe usse zyada chahta hai, aur shm full hone pe browser **crash** ho jaata hai — aur ye crash random tabs pe hota hai, matlab **flaky tests jinka koi pattern nahi**. `--ipc=host` host ka shared memory use karne deta hai. Alternative: `--shm-size=2gb`.

Ye ek classic "container mein Playwright flaky hai" ka root cause hai aur ise jaanna interview mein depth dikhata hai.

---

## 12.3 Dockerfile — line by line

```dockerfile
# ═══════════════════════════════════════════════════════════════════
# Dockerfile — Merlin Playwright/pytest suite
# ═══════════════════════════════════════════════════════════════════

# FROM — base image. Har Dockerfile isi se shuru hota hai.
# Official Playwright Python image: Python + Playwright + saare browsers
# + saari system dependencies pre-installed. Version PIN karo — `latest` nahi,
# warna kal image badal jaayegi aur tumhara build reproducible nahi rahega.
FROM mcr.microsoft.com/playwright/python:v1.47.0-jammy

# LABEL — metadata. Ownership track karne ke liye useful.
LABEL maintainer="qa@merlinai.co"
LABEL description="Merlin AI E2E test suite"

# ENV — image mein baked environment variables.
# PYTHONUNBUFFERED=1: Python output ko buffer mat karo — warna CI logs mein
#   output tab tak nahi dikhta jab tak process khatam na ho. Debugging ke liye
#   ye critical hai.
# PYTHONDONTWRITEBYTECODE=1: .pyc files mat banao, image chhoti rehti hai
# TZ: timezone FIX karo — warna date assertions machine-dependent ho jaayengi
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1 \
    TZ=Asia/Kolkata

# WORKDIR — is directory mein saare aage ke commands chalenge.
# `RUN cd /app` se better hai kyunki wo persist nahi karta.
WORKDIR /app

# ─── LAYER CACHING KA SABSE IMPORTANT TRICK ───────────────────────
# Sirf requirements.txt copy karo, poora code nahi.
# Kyun: Docker har instruction ko ek LAYER banata hai aur cache karta hai.
# Agar requirements.txt nahi badla, to next RUN layer cache se aayega —
# pip install skip ho jaayega.
# Agar tumne poora code pehle copy kiya hota, to har code change pe
# pip install dobara chalta. 2 minute vs 2 second ka farq.
COPY requirements.txt .

# RUN — image build ke waqt command chalao. Result layer mein save hota hai.
# `&&` se chain karo taaki ek hi layer bane (kam layers = chhoti image).
RUN pip install --no-cache-dir -r requirements.txt

# Ab code copy karo. Ye layer har commit pe invalidate hoti hai — theek hai,
# kyunki ye sasti hai. Mehngi (pip install) layer upar hai aur cached rehti hai.
COPY . .

# Non-root user — security best practice. Playwright image mein `pwuser`
# already bana hua hai.
# Root ke roop mein chalane pe: agar test compromised ho to container mein
# root access mil jaata hai, aur mounted volumes pe root-owned files ban jaati
# hain jo host pe delete karna mushkil hota hai.
RUN chown -R pwuser:pwuser /app
USER pwuser

# HEALTHCHECK — optional, par docker-compose mein `depends_on: condition:
# service_healthy` ke liye zaroori
# HEALTHCHECK --interval=30s --timeout=3s CMD python -c "import playwright" || exit 1

# ENTRYPOINT vs CMD:
#   ENTRYPOINT = fixed command, override karna mushkil
#   CMD        = default arguments, easily override ho jaate hain
# Dono milke: `docker run image` chalata hai `pytest tests/e2e -v`
#             `docker run image -m smoke` chalata hai `pytest -m smoke`
ENTRYPOINT ["pytest"]
CMD ["tests/e2e", "-v"]
```

### Build aur run

```bash
# Build
docker build -t merlin-e2e:local .

# Default CMD ke saath chalao
docker run --rm --ipc=host \
  -e BASE_URL="https://staging-app.merlinai.co" \
  -e TEST_USER -e TEST_PASSWORD \
  merlin-e2e:local

# CMD override karke smoke chalao
docker run --rm --ipc=host \
  -e BASE_URL="https://staging-app.merlinai.co" \
  merlin-e2e:local tests/e2e -m smoke -v

# Reports host pe nikalne ke liye volume mount karo
docker run --rm --ipc=host \
  -v "$PWD/reports":/app/reports \
  -e BASE_URL="https://staging-app.merlinai.co" \
  merlin-e2e:local tests/e2e --html=reports/report.html --self-contained-html
```

### Layer caching — kya invalidate karta hai kya

```
┌─────────────────────────────────────────────────────────────────┐
│ Layer 1: FROM mcr.../playwright/python:v1.47.0-jammy            │
│   Invalidate hota hai: kabhi nahi (version pinned hai)          │
│   CACHED ✅                                                      │
├─────────────────────────────────────────────────────────────────┤
│ Layer 2: ENV ...                                                 │
│   Invalidate: Dockerfile mein ENV line badle to                 │
│   CACHED ✅                                                      │
├─────────────────────────────────────────────────────────────────┤
│ Layer 3: COPY requirements.txt .                                 │
│   Invalidate: requirements.txt ka CONTENT badle to               │
│   CACHED ✅ (kyunki requirements roz nahi badalti)              │
├─────────────────────────────────────────────────────────────────┤
│ Layer 4: RUN pip install -r requirements.txt   <-- MEHNGI (2min) │
│   Invalidate: layer 3 invalidate hone pe                         │
│   CACHED ✅  <-- YAHI hai poore trick ka point                  │
├─────────────────────────────────────────────────────────────────┤
│ Layer 5: COPY . .                                                │
│   Invalidate: KOI BHI file badle to (har commit pe)             │
│   REBUILT 🔄  <-- sasti hai, koi baat nahi                      │
└─────────────────────────────────────────────────────────────────┘
```

**Golden rule: Dockerfile mein cheezein "kam badalne wali" se "zyada badalne wali" ke order mein rakho.**

**`.dockerignore` bhi zaroori hai** (warna `COPY . .` sab kuch bhej dega — `.git`, `venv`, reports):

```
.git
.gitignore
__pycache__/
*.pyc
.venv/
venv/
.pytest_cache/
node_modules/
reports/
test-results/
screenshots/
*.md
.env
```

`.dockerignore` na hone pe do problems: build context bada hota hai (slow), aur `COPY . .` layer har baar invalidate hoti hai kyunki `.pytest_cache` badalta rehta hai.

---

## 12.4 docker-compose — app + db + tests ek saath

Ye **sabse powerful QA pattern** hai: ek command se poora stack khada karo, tests chalao, saaf karo.

`docker-compose.test.yml`:

```yaml
# ═══════════════════════════════════════════════════════════════════
# Full stack test environment: database + app + tests
# Chalane ka tareeka:
#   docker compose -f docker-compose.test.yml up \
#     --build --abort-on-container-exit --exit-code-from tests
# ═══════════════════════════════════════════════════════════════════

services:

  # ─────────────────────────────────────────────────────────────
  # DATABASE — har run pe bilkul saaf, tmpfs ki wajah se
  # ─────────────────────────────────────────────────────────────
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: merlin_test
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
      # Fsync off — test DB hai, durability ki zaroorat nahi.
      # Ye postgres ko 2-3x tez kar deta hai.
      POSTGRES_INITDB_ARGS: "--no-sync"
    command: >
      postgres
      -c fsync=off
      -c synchronous_commit=off
      -c full_page_writes=off
    # ─── tmpfs — DB ko RAM mein rakho ─────────────────────────
    # Do fayde:
    #   1. TEZ — disk I/O bilkul nahi. Test DB ke liye 3-5x speedup.
    #   2. SAAF — container band, data gone. Har run pe guaranteed
    #      pristine database. Koi leftover state nahi, koi "pichhle
    #      run ka data" wali flakiness nahi.
    tmpfs:
      - /var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U test -d merlin_test"]
      interval: 3s
      timeout: 3s
      retries: 20
      start_period: 5s

  # ─────────────────────────────────────────────────────────────
  # APPLICATION under test
  # ─────────────────────────────────────────────────────────────
  app:
    build:
      context: ./backend
      dockerfile: Dockerfile
    environment:
      DATABASE_URL: postgresql://test:test@db:5432/merlin_test
      # Service name (`db`) hi hostname hai — compose ka internal DNS
      SPRING_PROFILES_ACTIVE: test
      # Third-party integrations stub pe point karo — asli QuickBooks
      # ko test se kabhi hit mat karo
      QUICKBOOKS_BASE_URL: http://wiremock:8080
    depends_on:
      db:
        condition: service_healthy      # DB ready hone ka INTEZAAR karo
      wiremock:
        condition: service_started
    ports:
      - "8080:8080"                     # host se debug karne ke liye
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 5s
      timeout: 3s
      retries: 30
      start_period: 30s                 # JVM boot slow hota hai

  # ─────────────────────────────────────────────────────────────
  # THIRD-PARTY STUBS — QuickBooks etc.
  # ─────────────────────────────────────────────────────────────
  wiremock:
    image: wiremock/wiremock:3.9.1
    command: ["--verbose", "--global-response-templating"]
    volumes:
      - ./test-stubs:/home/wiremock:ro
    ports:
      - "8081:8080"

  # ─────────────────────────────────────────────────────────────
  # TESTS — ye service exit karega aur uska exit code hi result hai
  # ─────────────────────────────────────────────────────────────
  tests:
    build:
      context: .
      dockerfile: Dockerfile
    ipc: host                           # Chromium /dev/shm crash fix
    depends_on:
      app:
        condition: service_healthy      # app ready hone ka intezaar
    environment:
      BASE_URL: http://app:8080
      API_URL:  http://app:8080/api
      TEST_USER: seeded-test@merlinai.co
      TEST_PASSWORD: ${TEST_PASSWORD}   # host ke env se aata hai
      CI: "true"
    volumes:
      # Reports host pe likho taaki container delete hone ke baad bhi bachein
      - ./reports:/app/reports
      - ./test-results:/app/test-results
    command: >
      pytest tests/e2e
        -m smoke
        --browser chromium
        --screenshot only-on-failure
        --tracing retain-on-failure
        --junitxml=/app/reports/results.xml
        --html=/app/reports/report.html --self-contained-html
        -v
```

### Chalane ka tareeka — aur wo do critical flags

```bash
docker compose -f docker-compose.test.yml up \
  --build \
  --abort-on-container-exit \
  --exit-code-from tests
```

| Flag | Kya karta hai | Kyun zaroori |
|---|---|---|
| `--build` | Chalane se pehle images rebuild karo | Warna purani image use hogi, code changes nahi dikhenge |
| `--abort-on-container-exit` | Koi bhi container exit kare to sab band karo | **Warna `tests` khatam ho jaayega par `db` aur `app` chalte rahenge aur command kabhi return nahi karegi — CI job hang ho jaayega aur timeout pe fail hoga** |
| `--exit-code-from tests` | Poore compose command ka exit code `tests` service ka exit code ho | **Warna compose hamesha 0 return karega aur CI ko lagega tests pass ho gaye, chahe wo fail hue hon.** Ye silent-green ka classic source hai. |

**Ye do flags interview mein poochhe jaate hain aur inka answer yaad rakhna hai.** Bina inke pipeline ya to hang hoga ya jhoothi green degi — dono se bura kya ho sakta hai.

### Cleanup

```bash
# Sab band karo aur volumes bhi hata do
docker compose -f docker-compose.test.yml down -v --remove-orphans

# CI mein hamesha cleanup karo, chahe kuch bhi ho
```

GitHub Actions mein:

```yaml
      - name: Run full stack tests
        run: |
          docker compose -f docker-compose.test.yml up \
            --build --abort-on-container-exit --exit-code-from tests

      - name: Dump app logs on failure
        if: failure()
        run: docker compose -f docker-compose.test.yml logs app db

      - name: Upload reports
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: compose-test-reports
          path: reports/

      - name: Tear down
        if: always()
        run: docker compose -f docker-compose.test.yml down -v --remove-orphans
```

**`if: failure()` pe app logs dump karna** — ye ek chhoti si cheez hai jo debugging time ghanton se minutes mein le aati hai. Test fail hua? Ab tumhare paas test ka output bhi hai aur application ka server-side log bhi. Interview mein ye bolna practical maturity dikhata hai.

## 12.5 Docker image caching CI mein

Build ko tez rakhne ke liye GitHub Actions mein:

```yaml
      - name: Set up Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build with layer cache
        uses: docker/build-push-action@v6
        with:
          context: .
          tags: merlin-e2e:${{ github.sha }}
          load: true
          # GitHub Actions cache backend — layers run ke beech persist hote hain
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

`mode=max` saari intermediate layers cache karta hai (sirf final nahi) — matlab agar sirf code badla to `pip install` layer cache se aayegi.

## 12.6 Interview answer

> **Interview answer:**
> "I use Docker in three ways as a QA.
>
> First, as a reproducible test environment. I run the suite in the official Playwright Python image, pinned to a version, so my laptop and CI execute in byte-identical environments — same OS libraries, same browser build, same fonts, same timezone. That eliminates a whole class of environment-dependent flakiness, particularly fonts and locale, which quietly break UI tests by changing text metrics and date formatting.
>
> Second, as a full-stack test environment with docker-compose: database, application, third-party stubs and the test runner brought up together with one command. Two details matter there. I run the database on tmpfs, which puts it in RAM — that's faster, but more importantly it guarantees a pristine database on every run, so no test can be polluted by leftover state from the previous one. And I run the compose command with `--abort-on-container-exit` and `--exit-code-from tests`. Without the first, the test container finishes but the database keeps running and the CI job hangs until it times out. Without the second, compose always exits zero, so CI reports green even when every test failed. That second one is dangerous precisely because it's silent.
>
> Third, for build speed via layer caching. The Dockerfile is ordered from least-changing to most-changing: copy the requirements file and install dependencies first, then copy the source. That way a code change invalidates only the cheap final layer, and the expensive pip install comes from cache.
>
> One Playwright-specific detail I'd flag: containers need `--ipc=host` or a larger shm-size. Docker gives `/dev/shm` only 64 megabytes by default, Chromium needs more, and when it runs out the browser crashes on random pages. That produces flakiness with no reproducible pattern, and it's a very common cause of 'Playwright is unreliable in Docker'."

**Cross-question: "Docker mein test chalane ka koi nuksaan hai?"**

> **Interview answer:**
> "A few, and I'd be honest about them.
>
> Debugging is harder. You can't just watch the browser — you need headed mode with an X server or VNC, or you rely on traces and videos instead. In practice I lean on Playwright traces heavily, which is fine, but the feedback loop when writing a new test is slower than running locally.
>
> There's a startup cost. Bringing up a compose stack and waiting for health checks adds thirty to sixty seconds per run, which is negligible in CI but annoying for the local write-a-test loop. So my rule is: local development runs against a local or shared environment for speed, and CI runs in containers for determinism.
>
> Performance timing is less representative. Containers have resource limits and share the host, so absolute performance numbers from inside a container aren't reliable — I wouldn't run load tests there without dedicated infrastructure.
>
> And there's a maintenance surface. The compose file, the Dockerfile and the stub configuration are all code that can rot. If the app's real dependencies change and the compose file doesn't, you end up testing a fiction. So the compose environment has to be treated as part of the product's configuration, and ideally generated from or validated against the same definitions the real deployment uses."

---

# 13. Test Execution in CI

## 13.1 Parallelisation — do bilkul alag cheezein

Log "parallel" bol dete hain par do alag mechanisms hain aur interview mein farq poochha jaata hai.

```
  PROCESS-LEVEL PARALLELISM (pytest-xdist)          SHARDING (matrix / pytest-split)
  ek machine, kai processes                          kai machines, ek-ek process (ya kai)

  ┌──────────── Runner 1 ──────────────┐            ┌─ Runner 1 ─┐ ┌─ Runner 2 ─┐
  │  proc0  proc1  proc2  proc3        │            │  tests     │ │  tests     │
  │   ▓▓     ▓▓     ▓▓     ▓▓          │            │  1-50      │ │  51-100    │
  │  (shared CPU, RAM, browser cache)  │            └────────────┘ └────────────┘
  └────────────────────────────────────┘            ┌─ Runner 3 ─┐ ┌─ Runner 4 ─┐
                                                     │  101-150   │ │  151-200   │
                                                     └────────────┘ └────────────┘
```

Dono ko **saath mein** use karte hain: 4 shards × 4 workers = 16 parallel tests.

### pytest-xdist

```bash
pip install pytest-xdist
```

```bash
# N workers
pytest -n 4

# CPU count ke hisaab se automatic
pytest -n auto

# Logical cores (hyperthreading included)
pytest -n logical

# Distribution mode — YE IMPORTANT HAI
pytest -n 4 --dist load        # default: jo worker free ho use agla test do
pytest -n 4 --dist loadscope   # same CLASS/MODULE ke tests same worker pe
pytest -n 4 --dist loadfile    # same FILE ke tests same worker pe
pytest -n 4 --dist loadgroup   # @pytest.mark.xdist_group ke hisaab se
```

**`--dist loadfile` UI tests ke liye aksar best hai.** Kyun: agar ek file ke tests session state share karte hain (login ek baar, phir 5 tests), to `load` mode unhe alag workers pe bikher dega aur har worker ko dobara login karna padega — ya worse, state assumptions toot jaayengi.

**xdist ke saath jo cheezein tootti hain:**

| Problem | Kyun | Fix |
|---|---|---|
| Shared test data collision | Do workers same record edit kar rahe hain | Har test apna unique data banaye (`uuid4()` suffix) |
| Session-scoped fixtures | Har worker ko apna copy milta hai, "session" per-worker hai | `tmp_path_factory` + file lock, ya per-worker resource |
| DB truncate between tests | Ek worker truncate karega, doosre ka data ud jaayega | Per-worker schema/database, ya transactional rollback |
| Test ordering dependency | `load` mode order guarantee nahi karta | **Ye actually achha hai** — ye tumhare hidden dependencies expose karta hai |
| Output interleaving | Sab workers ek saath print kar rahe hain | `-n 4 --tx` ke saath `-p no:randomly`, ya reports pe rely karo |

Per-worker isolation ka pattern:

```python
# conftest.py
import os
import pytest

@pytest.fixture(scope="session")
def worker_id():
    """xdist worker id: 'gw0', 'gw1', ... ya 'master' agar -n nahi diya."""
    return os.environ.get("PYTEST_XDIST_WORKER", "master")


@pytest.fixture(scope="session")
def test_user(worker_id):
    """Har xdist worker ko apna dedicated login account.
    Warna do workers ek hi account se login karenge aur session
    invalidation ek doosre ko flaky bana degi."""
    accounts = {
        "master": "qa-auto-0@merlinai.co",
        "gw0":    "qa-auto-0@merlinai.co",
        "gw1":    "qa-auto-1@merlinai.co",
        "gw2":    "qa-auto-2@merlinai.co",
        "gw3":    "qa-auto-3@merlinai.co",
    }
    return accounts.get(worker_id, "qa-auto-0@merlinai.co")
```

> **[REAL]** Tumhare 6 flows PO aur bid banate hain. Agar tum `-n 6` chalao aur sab same test account use karein, to do risks hain: (a) same account pe concurrent sessions ek doosre ko logout kar sakte hain, (b) list pages pe assertions ("PO count should be N") ek doosre ke created records se poison ho jaayengi. **Fix: per-worker account + har PO ka name unique (`PO-{uuid4().hex[:8]}`) + assertions apne banaye record pe, global count pe nahi.** Ye interview mein bahut concrete answer hai.

### pytest-split (sharding)

```bash
pip install pytest-split

# Step 1: ek baar durations record karo (serial run)
pytest tests/e2e --store-durations --durations-path .test_durations
git add .test_durations && git commit -m "ci: record test durations"

# Step 2: CI mein har shard apna group chalata hai
pytest tests/e2e --splits 6 --group 3 \
  --splitting-algorithm least_duration \
  --durations-path .test_durations
```

**Algorithms:**

| Algorithm | Kaise baantta hai | Kab |
|---|---|---|
| `duration_based_chunks` | Consecutive chunks, duration ke hisaab se | Order maintain karna ho |
| `least_duration` | Greedy bin-packing — sabse balanced | **Default choice** |

**Bina durations file ke kya hota hai:** pytest-split test count ke hisaab se baant deta hai. Agar tumhare tests unequal hain (ek 30 sec, ek 4 min) to shards imbalanced honge:

```
  Bina durations:            Durations ke saath:
  shard1: ████████████ 22m   shard1: ██████ 8m
  shard2: ██ 4m              shard2: ██████ 8m
  shard3: ███ 5m             shard3: ██████ 7m
  shard4: █ 2m               shard4: ██████ 8m

  Wall clock = 22 min        Wall clock = 8 min
  (parallelism WASTED)       (parallelism USED)
```

**Ye interview mein bolne layak insight hai:** *"Total wall-clock time is the slowest shard, not the average. So unbalanced sharding wastes most of the parallelism you paid for. Recording durations and using a least-duration split is what turns four runners into an actual 4x, rather than a 1.5x."*

`.test_durations` ko periodically refresh karo (nightly job se), warna naye tests ke liye estimate nahi hoga aur balance dheere-dheere bigadta jaayega.

---

## 13.2 Test selection by changed paths

**Kya hai:** Sirf wo tests chalao jo badle hue code se related hain.

### Level 1: workflow-level path filters (sabse simple)

```yaml
on:
  pull_request:
    paths:
      - 'src/purchase-orders/**'
      - 'tests/e2e/po/**'
```

### Level 2: dynamic — changed files se markers derive karo

```yaml
jobs:
  detect:
    runs-on: ubuntu-latest
    outputs:
      markers: ${{ steps.pick.outputs.markers }}
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }

      - id: pick
        run: |
          BASE="origin/${{ github.base_ref || 'main' }}"
          git fetch origin "${{ github.base_ref || 'main' }}" --depth=50
          CHANGED=$(git diff --name-only "$BASE"...HEAD)
          echo "Changed files:"; echo "$CHANGED"

          MARKERS=""
          echo "$CHANGED" | grep -q '^src/purchase-order'  && MARKERS="${MARKERS} or po"
          echo "$CHANGED" | grep -q '^src/bid'             && MARKERS="${MARKERS} or bid"
          echo "$CHANGED" | grep -q '^src/pricing'         && MARKERS="${MARKERS} or po or bid"
          # Shared/core code badla to SAB kuch chalao — conservative raho
          echo "$CHANGED" | grep -qE '^(src/core|src/auth|pom.xml|build.gradle)' \
            && MARKERS=" or po or bid or admin or reports"

          MARKERS="${MARKERS# or }"       # leading ' or ' hatao
          [ -z "$MARKERS" ] && MARKERS="smoke"   # kuch match nahi hua -> smoke
          echo "markers=$MARKERS" >> "$GITHUB_OUTPUT"

  test:
    needs: detect
    runs-on: ubuntu-latest
    steps:
      - run: pytest tests/e2e -m "${{ needs.detect.outputs.markers }}"
```

### Level 3: `pytest-testmon` (coverage-based, sabse precise)

```bash
pip install pytest-testmon
pytest --testmon         # sirf wo tests jinka covered code badla hai
```

Testmon coverage data se track karta hai ki kaunsa test kaunsi line execute karta hai. Bahut precise, par `.testmondata` database maintain karna padta hai aur E2E tests ke liye kam practical hai.

### Path-based selection ka KHATRA — ye zaroor bolna

**Path-based selection ek risk trade hai, free lunch nahi.** Ye ye maanti hai ki tumhara dependency map sahi hai. Aur wo aksar galat hota hai:

- Shared utility badli, par map mein sirf uska direct consumer tha
- Config file badli jo runtime pe sab kuch affect karti hai
- Database migration — wo kis "path" mein aati hai?
- Dependency version bump — koi source file nahi badli

Isliye rule ye hona chahiye:

| Kahan | Selection |
|---|---|
| PR gate | **Selective** — tez feedback, thoda risk acceptable |
| Merge to main | **Selective + smoke** |
| Nightly | **Poori suite, hamesha** — ye safety net hai jo selection ki galtiyaan pakadta hai |
| Pre-release | **Poori suite** |

> **Interview answer:**
> "Path-based test selection is one of the biggest wins on pipeline time, but it's a risk trade rather than a free optimisation, and I'd be explicit about that.
>
> The implementation is straightforward: diff against the merge base, map changed paths to test markers, and run only the matching subset. The part that needs care is the fallback. If a change touches shared code — core modules, authentication, build files, dependency locks — the mapping should widen to everything rather than narrow, because a shared utility change can break anything. And if nothing matches the map at all, I'd default to running smoke rather than running nothing, because 'no tests matched' should never silently produce a green pipeline.
>
> The honest limitation is that the dependency map is a human artifact and it drifts. A config change or a database migration doesn't fit neatly into a path mapping. So I only use selection on the fast PR gate, where the value of quick feedback is highest, and I always run the complete suite nightly and before release. The nightly full run is what catches the cases where my selection logic was wrong — without it, selection is just hoping."

---

## 13.3 Retry policy — aur retries bugs kaise chhupate hain

### Retries kaise lagate hain

```bash
pip install pytest-rerunfailures

pytest --reruns 2 --reruns-delay 1
# sirf specific errors pe retry
pytest --reruns 2 --only-rerun "TimeoutError" --only-rerun "ConnectionError"
```

Per-test:

```python
@pytest.mark.flaky(reruns=2, reruns_delay=1)
def test_known_flaky_third_party_widget(page):
    ...
```

### Kyun retries khatarnak hain

Retry ka matlab hai: **"test fail hua, par hum us failure ko ignore kar rahe hain."** Aur ye teen problems banata hai:

**1. Real intermittent bugs chhup jaate hain.**
Ye sabse important point hai. Race condition ek **product bug** hai, test bug nahi. Wo production mein 1% users ko affect karega. Retry us signal ko exactly delete kar deta hai. Tumhara test **sahi** kaam kar raha tha — usne ek asli intermittent bug pakada — aur tumne retry laga ke use silence kar diya.

**2. Flakiness ki asli lagat chhup jaati hai.**
Retry ke saath suite "99% pass" dikhti hai. Bina retry ke wo 85% hai. Ab koi flakiness fix nahi karega kyunki number achha dikh raha hai. Retry ek painkiller hai — dard band ho gaya, bimari nahi.

**3. Time chupke se badhta hai.**
Har retry poora test dobara chalata hai (setup + login + navigation). 30 flaky tests × 2 retries × 90 sec = 90 minute chhupa hua time.

### Retry policy jo main recommend karunga

```
┌─────────────────────────────────────────────────────────────────────┐
│ RETRY KARO (infrastructure-level, product ka bug nahi):             │
│   - Runner spawn failure / OOM kill                                 │
│   - Docker image pull timeout                                       │
│   - Network unreachable to the environment                          │
│   - Explicitly known third-party outage (payment sandbox down)      │
│   Ye JOB-level retry hai, test-level nahi.                          │
├─────────────────────────────────────────────────────────────────────┤
│ RETRY MAT KARO (assertion failures):                                │
│   - Koi bhi assertion fail                                          │
│   - Element not found                                               │
│   - Timeout waiting for something                                   │
│   Ye ya to product bug hai ya test bug — dono investigate hone chahiye│
├─────────────────────────────────────────────────────────────────────┤
│ AGAR RETRY KARNA HI HAI:                                            │
│   - Max 1 retry, 2 nahi                                             │
│   - Retry hone wala test PASS nahi, "FLAKY" mark ho                 │
│   - Har flaky-pass metrics mein RECORD ho aur trend dikhe           │
│   - Flake rate ke upar apna alag gate ho (>1% = investigate)        │
└─────────────────────────────────────────────────────────────────────┘
```

CI mein job-level retry (test-level nahi):

```yaml
      - name: Run tests (retry only on infrastructure failure)
        uses: nick-fields/retry@v3
        with:
          timeout_minutes: 30
          max_attempts: 2
          retry_on: error          # exit code error pe, test failure pe nahi
          command: pytest tests/e2e -m smoke
```

GitLab mein ye zyada elegant hai:

```yaml
  retry:
    max: 1
    when:
      - runner_system_failure
      - stuck_or_timeout_failure
      - api_failure
    # NOTE: `script_failure` deliberately shamil NAHI hai — matlab
    # test failure pe retry nahi hoga, sirf infra failure pe hoga
```

> **Interview answer:**
> "My position is that retries are a diagnostic tool, not a fix, and the default should be no automatic retries on assertion failures.
>
> The reason is that a retry deletes a signal. If a test fails intermittently because of a race condition in the product, that's a real bug that will hit a small percentage of real users. The test did its job — it caught a genuine intermittent defect — and adding a retry silences exactly the signal you needed. You can't distinguish 'flaky test' from 'flaky product' from the outside, and retrying assumes it's always the former.
>
> The second problem is that retries hide the cost. With two retries a suite might report ninety-nine percent pass; without them it's eighty-five. Nobody funds fixing an eighty-five percent problem that displays as ninety-nine, so the flakiness compounds. And the time cost is invisible — thirty flaky tests retried twice at ninety seconds each is ninety minutes of hidden pipeline time.
>
> What I do allow is job-level retry for genuine infrastructure failures: runner crashes, image pull timeouts, network unreachable. Those aren't test results at all, they're the harness failing. GitLab expresses this well — you can retry on `runner_system_failure` while excluding `script_failure`.
>
> If a team insists on test-level retries, I'd accept at most one, and I'd insist that a test which passes on retry is reported as FLAKY rather than PASSED, and that the flake rate is tracked as its own metric with its own threshold. The moment a retried pass looks identical to a clean pass in the report, you've stopped measuring the thing that matters."

---

## 13.4 Flaky test detection, quarantine, and an SLA

### Flaky kya hai

Ek test jo **same code pe** kabhi pass kabhi fail hota hai. Non-deterministic.

Common causes (ye list interview mein bolne layak hai):

| Cause | Example | Fix |
|---|---|---|
| Implicit waits / sleeps | `time.sleep(2)` — kabhi kam pad jaata hai | Web-first assertions, `expect(locator).to_be_visible()` |
| Race conditions | Test click karta hai, API abhi pending hai | `page.wait_for_response()`, network idle |
| Test interdependence | Test B, test A ke bane data pe depend karta hai | Har test apna data banaye |
| Shared mutable state | Do tests same record edit karte hain | Unique data per test |
| Time/date | Midnight pe, month-end pe fail | Time freeze, ya relative dates |
| Animation | Element move ho raha hai jab click hua | `prefers-reduced-motion`, animation disable |
| Third-party | Ad script slow load ho raha hai | Route intercept se block karo |
| Order dependence | Serial mein pass, parallel mein fail | `pytest -p randomly` se expose karo |
| Environment | Slow CI runner, kam RAM | Timeouts, resources |

### Detection — kaise pakdo

**Tareeka 1: Same commit pe repeat runs**

```yaml
# Nightly flake detector — main ke same commit pe suite 3 baar chalao.
# Koi bhi test jo results mein inconsistent ho, wo flaky hai.
name: Flake Detector
on:
  schedule:
    - cron: '0 22 * * *'

jobs:
  detect:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        run: [1, 2, 3]
    steps:
      - uses: actions/checkout@v4
      - uses: ./.github/actions/setup-python-playwright
      - run: pytest tests/e2e --junitxml=run-${{ matrix.run }}.xml -v
        continue-on-error: true       # hum failures chahte hain, block nahi
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: flake-run-${{ matrix.run }}
          path: run-${{ matrix.run }}.xml

  analyse:
    needs: detect
    if: always()
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with: { pattern: flake-run-*, path: runs/, merge-multiple: true }
      - name: Find tests with inconsistent results
        run: |
          pip install junitparser
          python - <<'PY'
          import glob
          from collections import defaultdict
          from junitparser import JUnitXml

          results = defaultdict(set)
          for path in glob.glob("runs/run-*.xml"):
              xml = JUnitXml.fromfile(path)
              for suite in xml:
                  for case in suite:
                      outcome = "fail" if case.result else "pass"
                      results[f"{case.classname}::{case.name}"].add(outcome)

          flaky = [name for name, outcomes in results.items() if len(outcomes) > 1]
          if flaky:
              print("FLAKY TESTS DETECTED (inconsistent across identical runs):")
              for name in flaky:
                  print(f"  - {name}")
          else:
              print("No flakiness detected across 3 runs.")
          PY
```

**Tareeka 2: Historical tracking** — har run ka JUnit XML database mein daalo, per-test pass rate trend banao. `pass_rate < 100% and pass_rate > 0%` over last 30 runs = flaky.

**Tareeka 3: Retry pe pass** — agar tum retries use karte ho, to har "passed on retry" ek flake signal hai. Wo record karo.

### Quarantine — SLA ke saath

**Quarantine kya hai:** Flaky test ko main gate se hata do (blocking nahi rahega) par delete mat karo — wo alag "quarantine" job mein chalta rahe.

```python
# pytest.ini / pyproject.toml
[tool.pytest.ini_options]
markers = [
    "quarantine: flaky test, does not block the pipeline. Must have an owner and a due date.",
]
```

```python
@pytest.mark.quarantine
# QUARANTINED 2026-08-20 by ritik — owner: @ritik — due: 2026-09-03
# Reason: vendor dropdown intermittently renders empty when the
# /vendors call resolves after the modal animation completes.
# Suspected product race condition, not a test issue. Ticket: MER-4821
def test_po_material_with_new_vendor(page):
    ...
```

Pipeline mein:

```yaml
      # Main gate — quarantined tests EXCLUDE
      - name: Regression (blocking)
        run: pytest tests/e2e -m "regression and not quarantine"

      # Quarantine job — chalta hai, par block nahi karta
      - name: Quarantined tests (non-blocking)
        continue-on-error: true
        run: pytest tests/e2e -m quarantine --junitxml=quarantine.xml
```

### Quarantine SLA — ye rules honi chahiye

| Rule | Value | Kyun |
|---|---|---|
| Har quarantined test ka **named owner** ho | Mandatory | Bina owner ke koi fix nahi karega |
| Har quarantined test ki **due date** ho | 14 din | Open-ended quarantine = permanent |
| Har quarantined test ka **ticket** ho | Mandatory | Sprint planning mein visible ho |
| **Quarantine cap** | Total suite ka max 2% | Cap hit ho to naya quarantine tab tak nahi jab tak purana clear na ho |
| Due date miss hone pe | **Test delete kar do** | Kadwa par sahi — neeche explain |
| Quarantine ka **weekly review** | Standing agenda item | Visibility se hi cheezein hoti hain |

**"Due date miss to delete" kyun:** Kyunki ek quarantined test jo kabhi fix nahi hoga, wo **zero value aur non-zero cost** hai. Wo compute kha raha hai, report mein noise bana raha hai, aur sabse bura — wo team ko jhoothi tasalli de raha hai ki "wo scenario covered hai". Wo covered nahi hai. Delete karna kam se kam **imaandaar** hai — ab coverage gap dikh raha hai aur uspe decision liya jaa sakta hai.

Ye line interview mein bolo, ye bahut strong hai.

### Quarantine ke misuse se bachna

Quarantine ka abuse ye hai ki wo "dustbin" ban jaata hai jahan har inconvenient test daal diya jaata hai. Isse bachne ke liye:
- **P0 / critical-path tests kabhi quarantine nahi ho sakte.** Agar login test flaky hai to wo emergency hai, quarantine candidate nahi. Us case mein suite rukni chahiye jab tak fix na ho.
- Quarantine **automatically expire** ho (marker mein date parse karke CI fail kar de agar overdue hai):

```python
# conftest.py — overdue quarantine ko fail karo
import re, inspect, datetime, pytest

QUARANTINE_RE = re.compile(r"due:\s*(\d{4}-\d{2}-\d{2})")

def pytest_collection_modifyitems(config, items):
    today = datetime.date.today()
    overdue = []
    for item in items:
        if item.get_closest_marker("quarantine"):
            doc = (item.function.__doc__ or "") + (inspect.getcomments(item.function) or "")
            m = QUARANTINE_RE.search(doc)
            if not m:
                overdue.append(f"{item.nodeid}: quarantine has no due date")
            elif datetime.date.fromisoformat(m.group(1)) < today:
                overdue.append(f"{item.nodeid}: quarantine expired {m.group(1)}")
    if overdue:
        raise pytest.UsageError(
            "Quarantine SLA breached:\n" + "\n".join(overdue)
        )
```

> **Interview answer:**
> "I treat flakiness as a first-class defect category with its own process, not as background noise.
>
> Detection first: I run the full suite three times against an identical commit on a nightly schedule, and any test with inconsistent outcomes across those runs is flaky by definition — the code didn't change, so the test is non-deterministic. I also track per-test pass rate historically, so I can see a test degrading before it becomes everyone's problem.
>
> Then triage, and this is the step people skip: I try to determine whether it's a flaky test or a flaky product. A race condition in the application is a real bug that affects a small percentage of users, and it presents identically to a badly written test. If I can't distinguish, I escalate rather than quarantine, because quarantining a real product race condition is the worst possible outcome.
>
> For genuine test-side flakiness I use quarantine with a hard SLA. The test is excluded from the blocking gate but still runs in a separate non-blocking job, so I keep the data. Every quarantined test needs a named owner, a ticket, and a due date fourteen days out. There's a cap of about two percent of the suite — if we hit the cap, nothing new can be quarantined until something is fixed, which forces the conversation. And if the due date passes without a fix, I delete the test.
>
> That last rule sounds harsh but I'd defend it. A permanently quarantined test has zero value and non-zero cost: it consumes compute, adds noise to reports, and — worst of all — it lets the team believe a scenario is covered when it isn't. Deleting it is at least honest: now the coverage gap is visible and someone can make a real decision about it.
>
> One exception: critical-path tests are never quarantine candidates. If the login test is flaky, that's an emergency, not a candidate for the parking lot."

---

## 13.5 Reporting formats

| Format | Kaun padhta hai | Kaise generate | Strength |
|---|---|---|---|
| **JUnit XML** | **Machines** — CI, dashboards, aggregators | `--junitxml=results.xml` | Universal. Har CI tool isse parse karta hai. **Ye hamesha generate karo.** |
| **pytest-html** | Insaan, quick look | `--html=report.html --self-contained-html` | Ek file, kahin bhi kholo, artifact ke liye perfect |
| **Allure** | Insaan, stakeholder-facing | `--alluredir=allure-results` + `allure generate` | Sabse rich — steps, attachments, history, trends, categories |
| **Playwright trace** | **Debugging** | `--tracing retain-on-failure` | Time-travel debugging — DOM snapshots, network, console |
| **Coverage XML (Cobertura)** | Machines, coverage gates | `--cov-report=xml` | Diff-cover, Codecov, SonarQube isse padhte hain |
| **Terminal** | Developer, local | default | Fastest feedback |

### Sabhi ek saath

```bash
pytest tests/e2e \
  -m regression \
  --junitxml=reports/results.xml \
  --html=reports/report.html --self-contained-html \
  --alluredir=reports/allure-results \
  --tracing retain-on-failure \
  --screenshot only-on-failure \
  --video retain-on-failure \
  -v
```

### Playwright trace — QA ka sabse powerful debugging artifact

```bash
# Trace kholo
playwright show-trace test-results/.../trace.zip
```

Trace mein kya hota hai: har action ka **before/after DOM snapshot**, network requests, console logs, source code line, aur ek timeline. CI failure debug karne ke liye ye screenshot se **kaafi behtar** hai — screenshot batata hai *kya* dikha, trace batata hai *kya hua*.

Interview mein bolne layak: *"For CI failures I rely on traces more than screenshots. A screenshot tells me the end state; a trace lets me step backwards through every action with the DOM at that moment, plus the network and console. It turns 'I can't reproduce it locally' into a solvable problem."*

### GitHub Actions mein results publish karna

```yaml
      - name: Publish test results to the PR
        if: always()
        uses: EnricoMi/publish-unit-test-result-action@v2
        with:
          files: "**/results-*.xml"
          check_name: "E2E Results"
          comment_mode: always

      - name: Job summary
        if: always()
        run: |
          {
            echo "## E2E Results"
            echo ""
            echo "| Metric | Value |"
            echo "|---|---|"
            echo "| Total | ${{ steps.summary.outputs.total }} |"
            echo "| Passed | ${{ steps.summary.outputs.passed }} |"
            echo "| Failed | ${{ steps.summary.outputs.failed }} |"
          } >> "$GITHUB_STEP_SUMMARY"
```

`$GITHUB_STEP_SUMMARY` — ye markdown seedha run page pe render hota hai. Bahut under-used feature hai; isse tumhara CI run **padhne layak** ban jaata hai bina logs khole.

---

## 13.6 Notifications — Slack/Teams

> **[REAL]** Tumhare paas already `scripts/run_and_report.py` hai aur Slack bot token configured hai. Matlab tumne notification ka **sabse mushkil hissa already kar liya hai** — ab bas usse CI se call karna hai. Interview mein ye bolna: *"The reporting layer already existed as a script with a Slack bot integration; moving to CI was mostly about invoking it from a workflow step and passing the environment through variables instead of local config."*

### Notification design — kya galat hota hai

| Anti-pattern | Kyun bura | Fix |
|---|---|---|
| Har run pe notify (pass bhi) | Log channel mute kar dete hain | Failure pe notify; nightly summary daily |
| Sirf "FAILED" bhejna | Koi actionable info nahi | Failing test names + link + owner |
| Sabko @channel | Alert fatigue | Relevant log ko target karo |
| Log link do jo login maange | Log kholte hi nahi | Direct deep link + inline summary |
| Same failure har ghante | Noise | Dedupe / thread mein reply |

### Achha Slack message

```python
# scripts/notify_slack.py — CI se callable
import os, json, urllib.request

def post_summary(channel: str, results: dict, run_url: str) -> None:
    passed  = results["passed"]
    total   = results["total"]
    failed  = results["failed"]
    rate    = (passed / total * 100) if total else 0
    ok      = failed == 0
    emoji   = ":white_check_mark:" if ok else ":x:"

    blocks = [
        {"type": "header",
         "text": {"type": "plain_text",
                  "text": f"{emoji} Merlin E2E — {'PASSED' if ok else 'FAILED'}"}},
        {"type": "section", "fields": [
            {"type": "mrkdwn", "text": f"*Environment:*\n{os.environ['TEST_ENV']}"},
            {"type": "mrkdwn", "text": f"*Pass rate:*\n{rate:.1f}%"},
            {"type": "mrkdwn", "text": f"*Passed:*\n{passed}/{total}"},
            {"type": "mrkdwn", "text": f"*Duration:*\n{results['duration_min']:.1f} min"},
        ]},
    ]

    if failed:
        names = "\n".join(f"• {n}" for n in results["failed_tests"][:10])
        extra = f"\n_...and {failed - 10} more_" if failed > 10 else ""
        blocks.append({
            "type": "section",
            "text": {"type": "mrkdwn", "text": f"*Failing:*\n{names}{extra}"}
        })

    blocks.append({
        "type": "actions",
        "elements": [
            {"type": "button",
             "text": {"type": "plain_text", "text": "Run + screenshots"},
             "url": run_url},
        ]
    })

    req = urllib.request.Request(
        "https://slack.com/api/chat.postMessage",
        data=json.dumps({"channel": channel, "text": f"{emoji} E2E {'passed' if ok else 'FAILED'}",
                         "blocks": blocks}).encode(),
        headers={"Authorization": f"Bearer {os.environ['SLACK_BOT_TOKEN']}",
                 "Content-Type": "application/json; charset=utf-8"},
    )
    with urllib.request.urlopen(req) as resp:
        body = json.loads(resp.read())
        if not body.get("ok"):
            raise RuntimeError(f"Slack API error: {body.get('error')}")
```

**Notification philosophy jo bolni hai:**

> **Interview answer:**
> "My rule for notifications is that every message must be actionable, and the volume must stay low enough that people still read them. Those two constraints drive everything else.
>
> Concretely: I notify on failure, not on every pass, because a channel that pings green twenty times a day gets muted within a week — and then it's also muted when it goes red. Nightly gets a summary either way, because a daily health signal is genuinely useful. The message includes the actual failing test names, the environment, the pass rate and a direct link to the run with screenshots and traces attached, so someone can triage from their phone without opening a laptop. And I'd route by ownership rather than broadcasting — a failure in the purchase order flow goes to the team that owns purchase orders.
>
> On my project the reporting side already existed — I had a runner script producing a summary and posting to Slack via a bot token — so moving into CI was mostly about invoking it from a workflow step and sourcing the environment from CI variables rather than local config. The lesson I'd take from that is that reporting is worth building before the pipeline, because a pipeline whose output nobody reads is just a way to spend compute."

---

# 14. Secrets Management

## 14.1 Kya hai aur kyun ye QA ka topic hai

**Secret** = koi bhi value jo compromise hone pe nuksaan kare: DB password, API key, OAuth client secret, private key, session signing key, test account ka password.

Log sochte hain ye security team ka kaam hai. Par QA ka isse **direct** rishta hai, teen wajah se:

1. **Test code sabse zyada secrets touch karta hai.** Test ko login karna hai, DB seed karni hai, third-party sandbox call karni hai. Application code ek secret use karta hai; test suite paanch.
2. **Test config sabse kam review hoti hai.** Koi `application-test.properties` ko dhyan se nahi padhta. "Test config hai na, kya farq padta hai." — yahi soch leak ki jadd hai.
3. **QA hi wo banda hai jo poori system ko end-to-end dekhta hai.** Dev apne module mein hai, DevOps infra mein hai. Config drift aur leaked credentials aksar QA hi pakadta hai.

> **[REAL]** **Ye tumhari sabse strong interview story hai. Isse yaad karo aur confidently sunao.**
>
> Tumne Merlin backend mein `src/test/resources/application-test.properties` file mein **MongoDB Atlas connection credentials aur QuickBooks client secret plaintext mein committed** paaye.
>
> Ye finding itni strong kyun hai:
> - **Ye ek QA ne pakda, security scan ne nahi** — matlab tumne apne scope se bahar dekha
> - **Ye "sirf test config" hai** — aur yahi sabse khatarnak assumption hai. MongoDB Atlas ek **cloud-hosted** DB hai; wo credentials internet se reachable hain. QuickBooks client secret ek **financial integration** ka hai.
> - **Ye git history mein hai** — matlab file delete karne se problem solve nahi hoti
> - Tumne sirf report nahi kiya, tum **preventive control** bhi bata sakte ho (pipeline mein secret scanning gate)

## 14.2 Rule 1 — Secrets kabhi code mein nahi

**Kabhi nahi. Kisi bhi form mein. Kisi bhi environment ke liye.**

Wo excuses jo log dete hain, aur unka jawab:

| Excuse | Reality |
|---|---|
| "Ye sirf test environment hai" | Test env aksar production data ka copy rakhta hai. Aur test DB se production ke baare mein schema, structure, integration endpoints sab pata chalta hai. |
| "Repo private hai" | Private repos leak hote hain — laptop chori, ex-employee ka access, fork, third-party CI integration, accidental public toggle. |
| "Ye read-only credentials hain" | Aaj read-only hain. Kal kisi ne permission badha di. Aur read-only bhi PII leak kar sakta hai. |
| "Encoded hai (base64)" | Base64 encoding hai, encryption nahi. `base64 -d` ek command hai. |
| "Purana hai, ab use nahi hota" | Rotate nahi hua to abhi bhi valid hai. Aur agar rotate ho gaya to code se hata kyun nahi? |

### `.gitignore` — pehli line of defence

```gitignore
# ─── Secrets ──────────────────────────────────────
.env
.env.*
!.env.example              # example file COMMIT karo (values ke bina)
*.pem
*.key
*.p12
*.pfx
credentials.json
service-account*.json
secrets.yaml
secrets.yml
**/application-local.properties
**/application-secret.properties

# ─── Python ───────────────────────────────────────
__pycache__/
*.pyc
.venv/
venv/
.pytest_cache/
.coverage
coverage.xml
htmlcov/

# ─── Test output ──────────────────────────────────
test-results/
screenshots/
videos/
reports/
allure-results/
playwright-report/
.test_durations.local

# ─── IDE ──────────────────────────────────────────
.idea/
.vscode/
*.swp
.DS_Store
```

**`.env.example` commit karo** — ye documentation hai:

```bash
# .env.example — copy karke .env banao aur real values daalo
BASE_URL=https://staging-app.merlinai.co
API_URL=https://staging-eks.merlinai.co
TEST_USER=your-test-account@merlinai.co
TEST_PASSWORD=
SLACK_BOT_TOKEN=
```

**`.gitignore` ki bahut badi limitation — ye interview mein poochha jaata hai:**

`.gitignore` sirf **untracked** files ko ignore karta hai. Agar file **pehle se tracked** hai, to `.gitignore` mein daalne se kuch nahi hoga — wo track hoti rahegi.

```bash
# Galat samajh: "maine .gitignore mein daal diya, ab safe hai"
echo ".env" >> .gitignore
git status                    # .env abhi bhi modified dikh raha hai!

# Sahi: pehle index se hatao
git rm --cached .env          # file disk pe rahegi, git se hat jaayegi
git commit -m "chore: stop tracking .env"

# Ab .gitignore kaam karega — LEKIN purani history mein wo abhi bhi hai
```

Aur ye doosra critical point: **`.gitignore` history ko affect nahi karta.** Agar secret ek baar commit ho gaya, to wo history mein hamesha hai jab tak tum history rewrite na karo. Section 35.2 mein poora procedure hai.

## 14.3 Rule 2 — Secret scanning, pipeline mein

### Tools

| Tool | Kya karta hai | Kab use |
|---|---|---|
| **gitleaks** | Regex + entropy, poori git history scan | CI gate + pre-commit. **Sabse popular.** |
| **trufflehog** | Detection + **verification** (key actually valid hai kya, live check karke) | Deep audit, incident response |
| **git-secrets** (AWS) | Pre-commit hook, AWS patterns pe focus | Local prevention |
| **detect-secrets** (Yelp) | Baseline file ka concept — existing secrets whitelist karo, naye block karo | Legacy repo jahan already secrets hain |
| **GitHub Secret Scanning** | Native, push-protection, partner ko auto-notify | GitHub pe free (public repos), Advanced Security (private) |

**trufflehog ka `--only-verified` flag khaas hai** — wo actually key ko provider ke against test karta hai. Matlab false positives lagbhag zero. Interview mein ye bolna: *"trufflehog's verification mode actually calls the provider to check whether the credential is live, which turns a noisy regex problem into a high-signal one — a verified finding is not a maybe, it's an active leak."*

### Setup — gitleaks locally

```bash
# Install
brew install gitleaks              # macOS
# ya: docker run --rm -v "$PWD:/repo" zricethezav/gitleaks:latest detect --source=/repo

# Working directory scan (uncommitted files bhi)
gitleaks detect --source . --no-git -v

# Poori git HISTORY scan — ye wo hai jo purane commits pakadta hai
gitleaks detect --source . -v

# Report file
gitleaks detect --source . --report-format json --report-path gitleaks-report.json
```

`.gitleaks.toml` — custom rules aur allowlist:

```toml
title = "Merlin AI secret scanning config"

[extend]
useDefault = true                 # gitleaks ke built-in rules bhi chalao

# Custom rule — Merlin-specific patterns
[[rules]]
id = "merlin-internal-api-key"
description = "Merlin internal API key"
regex = '''(?i)merlin[_-]?api[_-]?key['"\s:=]{1,10}([a-zA-Z0-9]{32,})'''
tags = ["key", "merlin"]

[[rules]]
id = "mongodb-atlas-uri"
description = "MongoDB Atlas connection string with embedded credentials"
regex = '''mongodb(\+srv)?://[^:]+:[^@]+@[^/\s]+\.mongodb\.net'''
tags = ["database", "credentials"]

[allowlist]
description = "Known-safe placeholders"
regexes = [
  '''EXAMPLE_KEY''',
  '''your-password-here''',
  '''xxxxxxxx''',
]
paths = [
  '''\.gitleaks\.toml''',
  '''docs/security-examples\.md''',
  '''\.env\.example''',
]
```

### CI gate

```yaml
  secrets-scan:
    name: Secret scan (blocking)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0        # POORI history chahiye, warna sirf latest commit scan hoga

      - name: gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      # Verified-only pass — high signal, zero noise
      - name: trufflehog (verified secrets only)
        uses: trufflesecurity/trufflehog@main
        with:
          extra_args: --only-verified
```

### Pre-commit hook — CI se pehle rok do

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.4
    hooks:
      - id: gitleaks

  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.5.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
```

```bash
pip install pre-commit
pre-commit install               # .git/hooks/pre-commit install ho gaya
pre-commit run --all-files       # ek baar poore repo pe chalao
```

**Defence in depth — teen layers:**

```
  Layer 1: PRE-COMMIT HOOK        <- developer ke laptop pe. Fastest.
           (bypass ho sakta hai --no-verify se)
                  |
  Layer 2: CI GATE                <- bypass nahi ho sakta (branch protection ke saath)
           (blocking, poori history)
                  |
  Layer 3: PUSH PROTECTION        <- GitHub server-side, push hi reject
           + platform scanning       (GitHub Advanced Security)
```

Ek layer kaafi nahi hai. Pre-commit bypass ho sakta hai (`git commit --no-verify`). CI gate reliable hai par tab tak secret laptop se nikal chuka hai. Push protection sabse strong hai par har platform pe nahi hota.

## 14.4 Rule 3 — Secrets ko sahi jagah rakho

### Options, weakest se strongest

| Storage | Security | Rotation | Audit | Kab |
|---|---|---|---|---|
| Code mein hardcoded | **Zero** | Impossible | None | **Kabhi nahi** |
| Local `.env` (gitignored) | Low | Manual | None | Local dev only |
| CI secret store (GH/GitLab/Jenkins) | Medium | Manual | Basic | **Baseline for CI** |
| Cloud secret manager (AWS Secrets Manager, GCP Secret Manager, Azure Key Vault) | High | **Automatic** | Full | Production |
| **HashiCorp Vault** | **Highest** | Automatic + **dynamic** | Full + leases | Enterprise, multi-cloud |

### Vault — kya khaas hai

Vault ki sabse important cheez **dynamic secrets** hai, aur ye interview mein bolne layak hai.

Static secret: ek password jo 6 mahine se same hai, 40 logon ke paas hai, aur leak hone pe pata bhi nahi chalega.

**Dynamic secret:** Vault **request pe** ek naya credential banata hai, TTL ke saath.

```bash
# CI job Vault se DB credential maangta hai
vault read database/creds/qa-readonly
# Output:
#   lease_id       database/creds/qa-readonly/abc123
#   lease_duration 1h
#   username       v-token-qa-read-x7k2m9
#   password       A1a-8sjdh2kJHDs9

# 1 ghante baad Vault khud ye user DB se DELETE kar deta hai
```

Iska matlab:
- Har CI run ka apna credential hai
- Wo 1 ghante mein khud mar jaata hai
- Leak hua to blast radius = 1 ghanta, 1 job
- Audit log mein exactly dikhta hai kaunse job ne kaunsa credential liya

CI se Vault access karne ka modern tareeka — **OIDC**, matlab koi long-lived token bhi nahi:

```yaml
    permissions:
      id-token: write            # OIDC token maangne ke liye
      contents: read
    steps:
      - name: Get secrets from Vault via OIDC
        uses: hashicorp/vault-action@v3
        with:
          url: https://vault.merlinai.co
          method: jwt
          role: github-actions-qa
          secrets: |
            secret/data/qa/test-user password | TEST_PASSWORD ;
            secret/data/qa/test-user username | TEST_USER
```

Yahan GitHub apna signed identity token deta hai, Vault use verify karta hai, aur short-lived secret return karta hai. **Kahin bhi koi long-lived credential store nahi hua.** Ye "keyless" pattern aaj ka best practice hai.

## 14.5 Rule 4 — Least-privilege test accounts

Test account production admin nahi hona chahiye. Ye obvious lagta hai par bahut common galti hai.

| Principle | Implementation |
|---|---|
| **Dedicated accounts** | `qa-automation-1@merlinai.co` — kisi insaan ka account nahi |
| **Minimum role** | Test ko jo chahiye bas wahi. PO create karna hai? Admin role mat do. |
| **Environment-scoped** | Staging ka credential production pe kaam na kare — alag identity |
| **Marked as test data** | Account/records pe flag ho taaki analytics aur billing se exclude ho |
| **Per-worker accounts** | Parallel runs ke liye alag accounts (Section 13.1) |
| **No production write** | Production pe read-only, ya strictly scoped test tenant |
| **Rate-limit exempt but monitored** | Test traffic block na ho par visible ho |

> **[REAL]** Tumhari suite production pe chalti hai. Ye interview mein aayega. Answer ka shape:
>
> *"Because my suite runs against production, account scoping matters more, not less. What I'd insist on: a dedicated automation account that isn't a person's login, the minimum role needed to create a purchase order rather than an admin role, records created with an identifiable prefix so they can be excluded from reports and cleaned up, and no destructive operations against real customer records. The risk of testing in production isn't hypothetical — it's that your test data pollutes real business reporting, and that a badly scoped account can do real damage. Those are controllable with account design, and they're the first thing I'd fix if they weren't already in place."*

## 14.6 Rule 5 — Rotation

**Rotation** = credential ko periodically naye se replace karna.

| Trigger | Timeline |
|---|---|
| Scheduled | 90 din (typical), 30 din (high-value) |
| Employee offboarding | **Turant** |
| Suspected exposure | **Turant** |
| **Confirmed leak (commit mein mila)** | **Turant, sab kuch se pehle** |
| Vendor breach notification | Turant |

**Rotation tabhi practical hai jab wo automated ho.** Agar rotation ka matlab hai "8 jagah manually value badalna" to koi rotate nahi karega. Isliye:
- Secret ek hi jagah store ho (secret manager), aur sab wahin se padhein
- Application secret ko **runtime pe** padhe, startup pe cache karke bhool na jaaye — ya rotation pe reload kare
- Rotation ke waqt **do valid credentials ka overlap window** ho (old + new dono kaam karein), warna rotate karte hi outage

## 14.7 Interview answer — poora, tumhari finding ke saath

> **Interview answer:**
> "My baseline is that secrets never live in the repository — not in application code, not in test configuration, not base64-encoded, and not 'just for the test environment'. Then I'd layer defences: `.gitignore` plus a committed `.env.example` so the required variables are documented without values; a pre-commit hook running gitleaks so the developer is stopped locally; a blocking gitleaks job in CI with full history fetched, because scanning only the latest commit misses everything already in history; and ideally platform push protection, which rejects the push server-side. Any one of those layers can be bypassed — a pre-commit hook is defeated by `--no-verify` — so the CI gate is the one that has to be non-negotiable and enforced through branch protection.
>
> For storage I'd use the CI secret store as a minimum and a proper secret manager or Vault where it's available. The pattern I find most compelling is Vault's dynamic secrets combined with OIDC: the CI job presents its signed identity, Vault issues a database credential with a one-hour lease, and Vault deletes that user afterwards. There is no long-lived credential stored anywhere, and a leak has a one-hour blast radius on a single job.
>
> I've dealt with the real version of this. On my current project I found MongoDB Atlas connection credentials and a QuickBooks client secret committed in plaintext in the backend's `src/test/resources/application-test.properties`. The reasoning that put them there was 'it's only test config', and that's exactly the assumption that makes it dangerous — MongoDB Atlas is cloud-hosted, so those credentials are reachable from the internet, and QuickBooks is a financial integration.
>
> The sequence I'd follow, and the ordering matters: rotate first, always. Deleting the file or rewriting history does nothing about a credential that's already valid — assume it's compromised the moment it's in a repository, because you cannot prove who cloned it. So: revoke and reissue the Atlas user and the QuickBooks secret, then check the access logs for use from unexpected addresses, then purge from history with git-filter-repo or BFG and coordinate the force-push with everyone who has a clone, and finally add the gitleaks gate so the class of problem can't recur.
>
> That last step is the part I care about most. Reporting a finding is table stakes; the value a QA adds is converting a one-off discovery into a control that runs on every commit forever."

**Cross-question: "Agar secret already commit ho gaya to file delete karne se kaam nahi chalega?"**

> **Interview answer:**
> "No. Deleting the file creates a new commit that removes it from the current tree, but every previous commit still contains it. Anyone can run `git log -p -- path/to/file` or check out an older commit and read it. If the repository was ever pushed, forked, cloned, or mirrored to a CI cache, that content exists in places you don't control.
>
> So the response order is: rotate the credential first, because that's the only step that actually removes the risk and it works regardless of who already has a copy. Then investigate — check provider access logs for use from unexpected sources, because a leaked credential in a repo may already have been harvested by scanners; there are bots that scan public GitHub pushes within seconds.
>
> Then rewrite history with git-filter-repo, or BFG for large repositories, to purge the blob. That's cleanup and hygiene, not remediation. And I'd be honest about its cost: rewriting history changes every commit SHA after the touched commit, so everyone has to re-clone or hard-reset, open pull requests may need rebasing, and any tags referencing old SHAs break. That's a coordinated team activity, not something you do quietly on a Friday evening.
>
> Finally, add the scanning gate so it can't happen again, and if the platform supports it, enable push protection so the rejection happens before the commit ever reaches the server."

---

# 15. Pipeline Performance — slow pipeline diagnose karna

## 15.1 Rule #1 — pehle MEASURE karo, guess mat karo

Ye sabse important cheez hai aur 90% log ise skip karte hain. Wo seedha "parallel kar dete hain" ya "cache add kar dete hain" — bina jaane ki time ja kahan raha hai.

**Ek pipeline optimise karne ka pehla step hamesha ek stage-wise breakdown banana hai.**

```
Pipeline: 47 minutes total

  Stage                        Time      %      Cumulative
  ────────────────────────────────────────────────────────
  Checkout                     0:20      0.7%   0:20
  Install dependencies         3:40      7.8%   4:00     <- cacheable
  Install Playwright browsers  2:10      4.6%   6:10     <- cacheable
  Lint + typecheck             1:15      2.7%   7:25
  Build                        4:30      9.6%   11:55
  Unit tests                   2:05      4.4%   14:00
  Deploy to staging            3:00      6.4%   17:00
  Wait for health check        4:00      8.5%   21:00    <- polling badly?
  Smoke                        2:30      5.3%   23:30
  UI regression (serial)      21:00     44.7%   44:30    <- THE PROBLEM
  Report + notify              2:30      5.3%   47:00
```

Ab decision data-driven hai. UI regression 45% hai — wahi attack karo. Dependencies 12% hain — cache se lagbhag zero ho jaayega. Baaki sab noise hai.

**Agar tumne measure kiye bina "let's parallelise the unit tests" bola, to tumne 4% ko attack kiya aur 45% ko chhod diya.**

### Measure kaise karo

GitHub Actions timing:

```bash
# API se har job aur step ka duration
gh api repos/OWNER/REPO/actions/runs --jq '.workflow_runs[0].id'

gh api repos/OWNER/REPO/actions/runs/RUN_ID/jobs \
  --jq '.jobs[] | {
    name: .name,
    started: .started_at,
    completed: .completed_at,
    steps: [.steps[] | {name: .name, started: .started_at, completed: .completed_at}]
  }'
```

Simple stage timing workflow mein:

```yaml
      - name: Timed step
        run: |
          START=$(date +%s)
          pytest tests/e2e -m smoke
          END=$(date +%s)
          echo "::notice::Smoke took $((END-START))s"
          echo "- Smoke: $((END-START))s" >> "$GITHUB_STEP_SUMMARY"
```

pytest ke andar kaunsa test slow hai:

```bash
pytest --durations=25              # sabse slow 25 tests
pytest --durations=0 -vv           # sab, setup/teardown ke saath alag
```

Output aisa aata hai:

```
========================= slowest 25 durations =========================
185.20s call     tests/e2e/test_po_custom.py::test_create_po_with_custom_items
 92.10s setup    tests/e2e/test_bid_trade.py::test_bid_trade_flow
 88.40s call     tests/e2e/test_po_material.py::test_create_po_material
 ...
```

`setup` line dekhna important hai — agar setup 92 sec le raha hai to problem test mein nahi, fixture mein hai (shayad har test pe fresh login ho raha hai).

## 15.2 Optimisation techniques — impact ke order mein

| # | Technique | Typical saving | Effort | Risk |
|---|---|---|---|---|
| 1 | **Sharding / parallelism** | 60-80% of test time | Low | Low |
| 2 | **Dependency + browser caching** | 2-5 min per run | **Very low** | None |
| 3 | **Test selection by path (PR only)** | 50-90% on PRs | Medium | **Medium** |
| 4 | **Shift tests down the pyramid** | Varies, biggest long-term | High | Low |
| 5 | Remove redundant/dead tests | 10-30% | Medium | Medium |
| 6 | Fix slow fixtures (session-scoped auth) | 20-40% of E2E time | Low | Low |
| 7 | Stub third-party calls | 10-25% | Medium | Medium |
| 8 | Faster runners (more CPU) | 20-40% | Low (costs money) | None |
| 9 | Docker layer caching | 1-4 min on build | Low | None |
| 10 | Reduce artifact size / retention | Upload time, cost | Very low | None |
| 11 | Fail fast — cheap gates first | Avg time on failures | Low | None |
| 12 | Skip on doc-only changes | 100% on those PRs | Very low | Low |

### 1. Sharding — biggest lever

Section 13.1 mein detail hai. Key insight dobara: **wall-clock = slowest shard**, isliye balance matter karta hai.

```
21 min serial  ──6 balanced shards──>  ~4 min
21 min serial  ──6 UNBALANCED shards──> ~12 min   (parallelism wasted)
```

### 2. Caching

Section 9.6 mein detail. Sabse sasta win — 10 minute ka kaam, har run pe 3-5 min bachao.

### 3. Test selection

Section 13.2. **PR pe use karo, nightly pe nahi.**

### 4. Shift down the pyramid — sabse valuable long-term

Ye ek mindset shift hai. Har E2E test ke liye poochho: **"Is test ka assertion kya hai, aur kya wo assertion neeche kisi level pe ho sakti hai?"**

| E2E test kya check kar raha hai | Kahan hona chahiye | Time saving |
|---|---|---|
| Tax calculation ka number sahi hai | **Unit test** | 90 sec → 5 ms |
| Required field validation ka message | **Component test** | 60 sec → 50 ms |
| API 400 return karta hai invalid payload pe | **API test** | 90 sec → 200 ms |
| Vendor dropdown ka data API se aa raha hai | **API + component** | 90 sec → 250 ms |
| **Poora PO creation workflow end to end** | **E2E — yahi rehna chahiye** | — |

E2E test ka kaam **workflow verify karna** hai, arithmetic verify karna nahi. Agar tumhare paas 6 E2E tests hain jo sirf alag-alag tax rates check karte hain, to 5 ko unit test banao aur 1 E2E rakho jo workflow prove kare.

> **[REAL]** Tumhare 6 flows × 12-15 steps hain. Ye actually **achha ratio hai** — 6 tests, har ek ek distinct business workflow. Ye bloated E2E suite nahi hai. Interview mein ye bolna confidently: *"My E2E suite is deliberately small — six flows covering the distinct business workflows, not one test per input variation. Input variations belong at the unit and API level. That's why my suite is minutes, not hours."* Ye tumhari suite ki **design quality** ka signal hai, aur interviewer ise notice karega.

### 6. Slow fixtures — aksar sabse bada hidden cost

```python
# ❌ SLOW — har test 40-60 sec login mein
@pytest.fixture
def logged_in_page(page):
    page.goto(f"{BASE_URL}/login")
    page.fill("#email", TEST_USER)
    page.fill("#password", TEST_PASSWORD)
    page.click("button[type=submit]")
    page.wait_for_url("**/dashboard")
    return page
```

```python
# ✅ FAST — ek baar login, storage state reuse
# conftest.py
import pytest
from pathlib import Path

STATE_FILE = Path(".auth/state.json")

@pytest.fixture(scope="session")
def auth_state(browser, worker_id):
    """Login once per xdist worker, save the storage state, reuse everywhere."""
    state_path = Path(f".auth/state-{worker_id}.json")
    state_path.parent.mkdir(exist_ok=True)

    if state_path.exists():
        return str(state_path)

    context = browser.new_context()
    page = context.new_page()
    page.goto(f"{BASE_URL}/login")
    page.fill("#email", TEST_USER)
    page.fill("#password", TEST_PASSWORD)
    page.click("button[type=submit]")
    page.wait_for_url("**/dashboard")
    context.storage_state(path=str(state_path))
    context.close()
    return str(state_path)


@pytest.fixture
def page(browser, auth_state):
    """Every test starts already logged in — no UI login, no wait."""
    context = browser.new_context(storage_state=auth_state)
    page = context.new_page()
    yield page
    context.close()
```

**Saving:** 6 tests × 45 sec login = 4.5 minute → ~5 second. Ye 30-40% saving hai bina koi test badle.

Aur ek step aage: login **API se** karo, UI se nahi:

```python
@pytest.fixture(scope="session")
def auth_state(playwright, worker_id):
    """Authenticate via the API and inject the token — no browser needed at all."""
    request_ctx = playwright.request.new_context(base_url=API_URL)
    resp = request_ctx.post("/auth/login",
                            data={"email": TEST_USER, "password": TEST_PASSWORD})
    assert resp.ok, f"API login failed: {resp.status} {resp.text()}"
    token = resp.json()["accessToken"]
    state_path = Path(f".auth/state-{worker_id}.json")
    state_path.parent.mkdir(exist_ok=True)
    state_path.write_text(json.dumps({
        "cookies": [],
        "origins": [{
            "origin": BASE_URL,
            "localStorage": [{"name": "accessToken", "value": token}],
        }],
    }))
    request_ctx.dispose()
    return str(state_path)
```

**Caveat jo bolna zaroori hai:** Agar tum har jagah API login use karoge to **login flow khud kabhi test nahi hoga**. To ek dedicated UI login test rakho jo asli login flow verify kare, aur baaki sab tests API se authenticate karein. Ye trade-off explicitly bolna interview mein achha lagta hai.

### 11. Fail fast — sasta gate pehle

```yaml
jobs:
  cheap-gates:            # 90 seconds
    runs-on: ubuntu-latest
    steps: [lint, format, typecheck, secret-scan]

  expensive:              # 25 minutes
    needs: cheap-gates    # <-- lint fail hua to ye chalega hi nahi
    runs-on: ubuntu-latest
    steps: [e2e]
```

Ye average time bachata hai, best-case nahi. Agar 20% PRs lint pe fail hote hain, to un 20% ka 25 min bacha.

### 12. Skip on doc-only changes

```yaml
on:
  pull_request:
    paths-ignore:
      - '**.md'
      - 'docs/**'
      - 'LICENSE'
      - '.github/ISSUE_TEMPLATE/**'
```

### 10. Artifact size

```yaml
      - uses: actions/upload-artifact@v4
        if: failure()               # sirf failure pe — success pe video kaun dekhta hai
        with:
          name: evidence
          path: test-results/
          retention-days: 7         # default 90 — cost aur upload time dono
          compression-level: 9
```

Videos sabse bhaari hote hain. `--video retain-on-failure` use karo, `--video on` nahi. Ek 15-step flow ka video 20-40MB ho sakta hai; 6 flows × har run = ~200MB upload, har run pe 30-60 sec.

## 15.3 Interview answer

> **Interview answer:**
> "My first move is always to measure, not to guess. I'd produce a stage-by-stage breakdown of where the wall-clock time goes, because the intuitive answer is usually wrong. I've seen people parallelise unit tests that were four percent of the pipeline while ignoring a serial UI suite that was forty-five percent. Without the breakdown you optimise what's visible rather than what's expensive.
>
> Once I have the numbers I work in order of impact per unit of effort. Caching is first because it's nearly free — dependency and Playwright browser caching typically saves two to five minutes per run for about ten minutes of setup work, with essentially no risk. Sharding is next and it's usually the biggest single lever: it turns test time into wall-clock time divided by the number of runners. The detail that matters there is balance, because total time is the slowest shard, not the average — so I record test durations and use a least-duration split, otherwise four runners give you a 1.5x speedup rather than 4x.
>
> Then I look at fixtures, which are often a large hidden cost. Logging in through the UI in every test is a classic one: caching the authenticated storage state and reusing it, or authenticating through the API, can cut thirty to forty percent off an end-to-end suite without touching a single test. The caveat I'd state explicitly is that you then need one dedicated test that exercises the real UI login, otherwise you've optimised away coverage of the login flow itself.
>
> Then test selection on the PR gate — running only tests related to changed paths — with the discipline that the full suite always runs nightly, so the selection logic's mistakes get caught.
>
> The highest-value change long term is shifting tests down the pyramid. If an end-to-end test exists to verify a tax calculation, that assertion belongs in a unit test where it runs in five milliseconds instead of ninety seconds. End-to-end tests should prove workflows, not arithmetic. That's slower to do than adding runners, but it's the only approach where the suite doesn't get slower again in six months."

**Cross-question: "Tumne saara kuch kar liya aur pipeline abhi bhi slow hai. Ab?"**

> **Interview answer:**
> "Then I'd stop treating it as one pipeline and split it by purpose, because at that point the constraint isn't efficiency, it's that I'm asking one pipeline to answer two different questions.
>
> The pull-request gate answers 'is this change safe to merge' and needs to be minutes. The release gate answers 'is this build safe to ship' and can take much longer, because it runs far less often. So I'd move the expensive-but-valuable work — full cross-browser regression, performance, security scanning — off the PR gate and onto merge-to-main, nightly, and pre-release. The total compute goes up slightly; the developer-facing latency drops a lot, and that's the number that changes behaviour.
>
> If it's still too slow after that, I'd start deleting. I'd measure which tests have ever caught a real regression — not failed, but caught something that was a genuine product defect — and be willing to say that a test which has never earned its keep in a year is a tax, not a safety net. That's an uncomfortable conversation, but a suite nobody can afford to run is worth less than a smaller suite everybody runs.
>
> And I'd check whether the slowness is even in the tests. I've seen pipelines where the largest single block was waiting for a health check that polled every thirty seconds when the service was ready in four, or an artifact upload of gigabytes of videos captured on passing runs. Those are minutes recovered for almost no effort, and they're invisible unless you actually look at the timeline."

---

# 16. CI/CD Senior Scenario Questions

> Ye section sabse zyada important hai. Definitions har koi ratta maar leta hai — **scenario answers** se senior aur mid-level ka farq pata chalta hai. Har answer ka structure: **clarify → structure → prioritise → trade-off acknowledge → measure**.

---

## 16.1 "Tumhare paas 5000 automation tests hain aur CI pipeline 2 ghante leti hai. Kaise kam karoge?"

**Ye THE headline question hai.** Isse interview mein bahut baar poochha jaata hai. Iska answer **structured aur numbers ke saath** hona chahiye — "I'd parallelise it" bolne se tum mid-level lagoge.

### Pehle: clarify karo (ye khud ek senior signal hai)

Answer dene se pehle 3 sawaal poochho:
1. Ye 2 ghante **kis stage** pe hain — har PR pe, ya nightly? (PR pe 2 ghante emergency hai; nightly pe acceptable ho sakta hai)
2. 5000 tests ka **breakdown** kya hai — kitne unit, kitne API, kitne UI?
3. Wo 2 ghante **wall-clock** hain ya **total compute**? (agar wall-clock hai to already parallel ho sakta hai)

### Structured answer with expected savings

| # | Technique | Kaise | Expected saving | Effort | Risk | Naya total |
|---|---|---|---|---|---|---|
| 0 | **Measure first** | Stage-wise breakdown, `--durations=50` | 0 (par sab kuch isi pe depend hai) | 1 day | None | 120 min |
| 1 | **Dependency + browser caching** | `actions/cache`, Playwright browser cache | **3-6 min** | 2 hours | None | ~115 min |
| 2 | **Sharding across runners** | 10 balanced shards, duration-based split | **~85%** of test time | 1 day | Low | **~20 min** |
| 3 | **In-shard parallelism (xdist)** | `-n 4` inside each shard | **additional 40-60%** | 2 days (isolation fix) | Medium | **~10 min** |
| 4 | **Fix slow fixtures** | Session-scoped auth via storage state / API login | **20-40%** of E2E time | 1 day | Low | ~8 min |
| 5 | **Path-based selection (PR only)** | Changed paths → markers, nightly full | **50-80% on PRs** | 3 days | Medium | ~4 min typical PR |
| 6 | **Shift tests down the pyramid** | E2E asserting logic → unit/API | **30-50%** over a quarter | Weeks | Low | Structural |
| 7 | **Delete dead tests** | Tests that never caught a real bug | 10-20% | 1 week analysis | Medium | Structural |
| 8 | **Split the pipeline by purpose** | PR gate vs release gate vs nightly | Latency, not compute | 1 day | Low | PR gate < 10 min |

### Bolne wala answer

> **Interview answer:**
> "Before I optimise anything I'd ask three clarifying questions, because they change the answer completely. Is that two hours on every pull request or on a nightly run? What's the composition of the five thousand — how many are unit versus API versus UI? And is that two hours wall-clock or total compute, because if it's compute across parallel runners the problem is different.
>
> Assuming it's two hours of wall-clock on the PR gate, which is the bad case, here's how I'd sequence it.
>
> Step zero is measurement. I'd produce a stage-by-stage breakdown and run pytest with `--durations` to find the slowest tests and, importantly, the slowest *setup* — because a slow setup line usually means an expensive fixture running per test rather than per session, and that's a much cheaper fix than optimising a test. Skipping measurement is how people spend a week parallelising something that was four percent of the runtime.
>
> Then, in order of impact per unit of effort:
>
> **Caching** first, because it's nearly free. Dependency caching and Playwright browser caching typically recover three to six minutes per run for a couple of hours of setup work, with no correctness risk.
>
> **Sharding** is the biggest single lever. If the suite is currently serial, splitting it across ten runners takes test execution to roughly a tenth. The detail that matters is balance: total wall-clock is the slowest shard, not the average, so I'd record test durations and use a least-duration split. Without duration data you split by test count, shards come out lopsided, and ten runners buy you a 3x speedup instead of close to 10x. That alone should take two hours to around twenty minutes.
>
> **Process-level parallelism inside each shard** with pytest-xdist stacks on top of sharding. That's where the work is, though — running tests concurrently exposes every shared-state assumption in the suite. Each test needs its own data with unique identifiers, and for UI tests each worker needs its own login account, otherwise concurrent sessions invalidate each other. Budget real time for the isolation work; the parallelism itself is one flag.
>
> **Fixture optimisation**: logging in through the UI in every test is the single most common waste in an end-to-end suite. Authenticate once per worker, save the storage state and reuse it — or authenticate through the API entirely. That's often twenty to forty percent of end-to-end time recovered without touching a test. The caveat is you then need one dedicated test that exercises the real UI login, or you've optimised away coverage of it.
>
> **Path-based test selection** on the PR gate: map changed paths to test markers and run the relevant subset, always widening to everything when shared or core code changes. That takes typical PRs down further. The discipline is that the full suite still runs nightly, so the selection logic's mistakes are caught within a day.
>
> Then the two structural changes that take longer but are the only ones that stop the problem returning. **Shifting tests down the pyramid** — if an end-to-end test exists to verify a tax calculation, that assertion belongs in a unit test running in five milliseconds rather than ninety seconds. End-to-end tests should prove workflows, not arithmetic. And **deleting tests that don't earn their place** — I'd measure which tests have ever caught a genuine product regression, and be willing to say that one which hasn't in a year is a tax, not a safety net.
>
> Finally I'd **split the pipeline by purpose**. The pull-request gate answers 'is this safe to merge' and should be under ten minutes. The release gate answers 'is this safe to ship' and can take forty. Cross-browser matrices, performance and security scanning belong on merge and nightly, not on every push. Total compute goes up slightly; developer-facing latency drops a lot, and latency is the number that changes behaviour.
>
> Realistically I'd expect two hours to under fifteen minutes within two weeks from caching, sharding and fixtures alone, with the structural work bringing the typical PR under five over a quarter.
>
> The thing I'd add is that I wouldn't do this once. Pipeline duration is a metric I'd track over time with an alert, because suites grow back. Without a standing number, you're back at two hours in a year and the whole exercise repeats."

**Cross-question: "Sharding se compute cost badh jaayegi na?"**

> **Interview answer:**
> "Compute cost stays roughly the same — you're running the same tests, just concurrently. Total machine-minutes are similar, sometimes slightly higher because each shard pays its own setup cost for checkout and dependency installation. That's a real overhead: ten shards means ten times the setup, which is exactly why caching matters more once you shard.
>
> What changes is that you're now paying for concurrency, which on GitHub-hosted runners means hitting concurrency limits rather than a bigger bill on the free tier. So there's a practical ceiling.
>
> But I'd reframe the question, because the comparison isn't 'money versus nothing'. If ten engineers wait an extra ninety minutes on every pull request, several times a day, that's the expensive resource — and it's worse than the arithmetic suggests, because a ninety-minute wait doesn't cost ninety minutes, it costs a context switch. And there's a second-order cost: slow pipelines change behaviour. People batch changes into bigger pull requests to avoid the wait, which makes every review harder and every failure harder to attribute. Runner minutes are cheap compared to that."

---

## 16.2 "Ek test local pe pass hota hai par CI mein fail hota hai. Kaise debug karoge?"

> **Interview answer:**
> "My mental model is that this is always an environment difference, and my job is to find which one. The test isn't behaving randomly — something differs, and the differences fall into a small number of categories, so I work through them systematically rather than re-running and hoping.
>
> First I gather evidence rather than theorise. In CI I want the Playwright trace, a screenshot, the video if it's captured, and the application-side logs from the same window. The trace is the most valuable of those, because a screenshot shows me the end state whereas a trace lets me step backwards through every action with the DOM, network and console at that moment. That alone resolves a large share of these.
>
> Then I go through the categories in order of likelihood.
>
> **Timing.** CI runners are usually slower and more contended than a laptop, so anything relying on an implicit assumption about speed breaks there first. Fixed sleeps and hard-coded timeouts are the usual culprits. The fix isn't a longer timeout, it's a web-first assertion that waits for the actual condition.
>
> **Test isolation and ordering.** CI often runs in a different order, or in parallel, or on fresh state. If the test passes locally only because a previous run left data behind, it fails on a clean environment. A quick check is to run it locally in isolation with `pytest path::test -p no:randomly` on a clean database, and to run the suite locally with random ordering to see if it breaks.
>
> **Environment configuration.** Different base URL, different feature flags, different seeded data, different third-party credentials pointing at a sandbox rather than a live service. I'd print the resolved configuration at the start of the CI run, because assuming what the config is has burned me before — I once lost time to a documented API host that had been dead for months while the real one was only discoverable from the deployed frontend bundle. Now I log what the run actually targets, and I add a pre-flight reachability check so a wrong or dead host fails in ten seconds with a clear message rather than forty minutes later as a wall of confusing test failures.
>
> **Machine differences.** Headless versus headed changes viewport, and viewport changes layout, which changes what's visible or clickable. Fonts differ, and different font metrics shift element positions. Timezone and locale differ, which breaks date assertions — that one is especially nasty because it's often correct locally in IST and wrong on a UTC runner. And in containers specifically, Chromium crashes when `/dev/shm` is the Docker default of 64 megabytes, which produces failures with no reproducible pattern.
>
> **Resources.** Constrained CPU and memory change behaviour under parallelism, and a runner that's swapping produces timeouts that look like application bugs.
>
> If none of that resolves it, I reproduce CI locally rather than continuing to guess — run the test in the same container image the pipeline uses, with the same environment variables, headless, with the same viewport and timezone. That usually converts the problem into a locally reproducible one, and at that point it's just debugging.
>
> The outcome I aim for isn't only fixing the test. If the cause was an environment difference, I'd try to eliminate the difference — pin the timezone in the image, run locally in the same container, set an explicit viewport — so the whole class of problem goes away rather than this one instance."

**Cross-question: "Aur agar ye CI mein bhi sirf kabhi-kabhi fail ho raha hai?"**

> **Interview answer:**
> "Then it's a flakiness problem rather than an environment difference, and the first question I'd ask is one people skip: is this a flaky *test* or a flaky *product*? A race condition in the application presents identically to a badly written test from the outside, and it's a real bug that will affect a small percentage of real users.
>
> To distinguish them I'd look at what the failure actually is. If the test times out waiting for an element that eventually appears, that's usually test-side — a missing wait. If the application returns a 500, or the data is occasionally wrong, or the failure correlates with concurrency, that's product-side and it's more important than the test.
>
> To get data I'd run the test in a loop against a fixed commit — say fifty iterations — and record the failure rate and the failure modes. A consistent ten percent failure with the same signature is very different from three different failure modes appearing once each. I'd also check whether it correlates with parallelism by running it serially versus with xdist, and whether it correlates with time of day, which points at shared environment contention or scheduled jobs.
>
> What I would not do is add a retry as the first response. A retry deletes the signal — if there's a genuine intermittent product defect, the test was doing exactly its job and retrying silences it. If I need to unblock the team in the meantime I'd quarantine it with an owner, a ticket and a fourteen-day due date, so it stops blocking without becoming invisible."

---

## 16.3 "Pipeline mein flaky tests kaise handle karoge?"

> **Interview answer:**
> "I treat flakiness as a defect category with its own detection, triage and SLA, rather than as background noise — because the failure mode of ignoring it is that the team stops trusting every red build, and at that point the entire suite has no information value.
>
> **Detection.** I run the full suite three times against an identical commit on a nightly schedule. Any test with inconsistent outcomes across those runs is flaky by definition — the code didn't change, so the test is non-deterministic. I also track per-test pass rate historically from the JUnit XML, so I can see a test degrading before it becomes everyone's problem.
>
> **Triage, which is the step people skip.** I try to determine whether it's a flaky test or a flaky product. If I can't tell, I escalate rather than quarantine, because quarantining a genuine product race condition is the worst possible outcome — you've taken a real bug and made it invisible.
>
> **Fixing the common causes.** Most test-side flakiness comes from a small set: fixed sleeps instead of condition-based waits, tests sharing mutable data, order dependence, animations moving elements mid-click, unstubbed third parties, and date or timezone assumptions. Each has a standard fix — web-first assertions, unique data per test, route interception, frozen time.
>
> **Quarantine with a hard SLA** for the ones I can't fix immediately. The test is excluded from the blocking gate but still runs in a separate non-blocking job so I keep the data. Every quarantined test needs a named owner, a ticket, and a due date fourteen days out. There's a cap of about two percent of the suite — hit the cap and nothing new can be quarantined until something is fixed, which forces the conversation rather than letting it drift. If the due date passes without a fix, I delete the test.
>
> That last rule sounds harsh, and I'd defend it directly. A permanently quarantined test has zero value and non-zero cost: it consumes compute, adds noise, and lets the team believe a scenario is covered when it isn't. Deleting it is at least honest — now the coverage gap is visible and someone can make a real decision about it.
>
> One exception: critical-path tests are never quarantine candidates. If the login test is flaky, that's an emergency, not a parking-lot candidate.
>
> **Measurement.** I'd track flake rate as a first-class metric with its own threshold — under one percent — and report it alongside pass rate. If flake rate isn't visible, nobody funds fixing it, because the retried suite looks green."

---

## 16.4 "Kya automated tests ko deployment block karna chahiye? Kab haan, kab nahi?"

> **Interview answer:**
> "Yes, but selectively — and I'd argue that deciding *which* tests block is more important than deciding whether tests block at all.
>
> **Tests that should always block.** Smoke tests, without exception: if login is broken or the application doesn't start, deploying further is not a judgement call. Critical-path end-to-end tests for the flows the business genuinely can't operate without — on a construction ERP that's purchase order and bid creation. Security gates: a detected secret or a critical CVE blocks, full stop. And build and unit test failures, because if it doesn't compile or its own unit tests fail, there's nothing to discuss.
>
> **Tests that should not block, but should be loud.** Anything with a known false-positive rate — visual regression comparisons, performance measurements on shared infrastructure, accessibility scans that flag pre-existing issues. Those should report, create tickets and trend over time, but they shouldn't stop a deploy, because a gate with a meaningful false-positive rate teaches people to override, and once overriding is normal it applies to the real gates too.
>
> Tests for non-critical secondary flows I'd also treat as reporting rather than blocking, at least on the deploy path — an admin report rendering slightly wrong shouldn't hold a security fix.
>
> **The principle I'd state**: a gate should block when the cost of shipping the failure exceeds the cost of the delay, and when the signal is trustworthy enough that people won't route around it. Both halves matter. A perfectly trustworthy gate on a trivial issue is still a bad gate, and a gate on a critical issue that's wrong thirty percent of the time will simply be ignored.
>
> **When I'd override even a blocking gate**: a production incident where the fix is small and well-understood and the delay costs more than the risk. But that needs a real break-glass process — named authorisation, a written justification, automatic logging and announcement in the team channel so it can't be quiet, and a follow-up ticket so the skipped verification actually gets done within a day. If there's no legitimate override path, people invent illegitimate ones like admin-merging or disabling the check, and those are untracked.
>
> And the failure mode I'd watch for: a gate everyone ignores is worse than no gate, because it creates false confidence at the management level while providing no actual protection. When I see routine overrides, there are only two honest responses — make it trustworthy, or remove it and be explicit about the risk."

---

## 16.5 "Pipeline ko khud kaise test karoge?"

> **Interview answer:**
> "This is a question I like, because pipeline code is production code for the engineering team — it's just rarely treated that way. If the pipeline is wrong, every quality signal downstream of it is wrong, and the failure is usually silent.
>
> **The failure mode I care about most is the false green.** A pipeline that reports success while doing nothing is far more dangerous than one that fails, because a failure gets investigated and a false green gets trusted. Concretely: a docker-compose run without `--exit-code-from` always exits zero regardless of test results. A pytest invocation with a marker expression that matches nothing collects zero tests and exits zero. A conditional that silently skips a job. A test file with an import error that gets swallowed. Every one of those produces a green tick and no testing.
>
> So my first control is **assertions about the pipeline's own behaviour**. I assert a minimum collected test count — if a run collects fewer tests than expected, it fails, because 'zero tests ran' should never be green. I use `--strict-markers` so a typo'd marker is an error rather than a silent no-op. I have a job that runs `pytest --collect-only` on every PR, which catches import errors, syntax errors and broken fixtures in seconds without running anything. And I assert the deployed version — a `/version` endpoint returning the commit SHA that the smoke test checks — so the pipeline proves it tested the thing it deployed.
>
> **Deliberate failure injection.** I periodically verify the gates actually gate: open a pull request with a deliberately failing test and confirm the merge is blocked; commit a fake credential in a test branch and confirm the secret scanner catches it. If a gate has never been observed to fail, you don't know it works — you only know it hasn't complained.
>
> **Treat pipeline changes like code changes.** Workflow files go through pull request review. Actions are pinned to a version, ideally a SHA, so an upstream change can't alter behaviour silently. I test pipeline changes on a branch before merging, and tools like `act` let you run workflows locally for fast iteration.
>
> **Monitor the pipeline as a system.** Duration trends — a suite that grows five percent a month is a problem you can only see in a trend. Failure rate and, separately, flake rate. Cache hit rate, because a silently broken cache key shows up as a slowly slowing pipeline rather than an error. And queue time, which is invisible in the run duration but very visible to the developer waiting.
>
> **Observability in the pipeline itself.** Every run should log its resolved configuration — which environment, which URLs, which commit, how many tests collected. When something goes wrong six weeks later, that log is the difference between a five-minute diagnosis and an afternoon.
>
> The general principle: anything that can silently succeed while doing nothing needs an explicit assertion that it did something."

---

## 16.6 "Ek team ke paas CI bilkul nahi hai. Tum kaise set up karoge?"

> **Interview answer:**
> "I'd start deliberately small, because the biggest risk isn't technical — it's that the team stops trusting the pipeline in the first month and routes around it forever. Getting adoption is harder than getting it working.
>
> **Week one: make it exist and make it green.** Checkout, install dependencies, run whatever tests already exist, on every push. That's it. Even if it's five unit tests, the point is to establish that a pipeline exists, it runs automatically, and it's green. Nothing blocking yet — I want people to see it working before it ever tells them no.
>
> **Week two: add cheap, unarguable gates.** Lint, formatting and a secret scan. These are fast, deterministic, and nobody argues about whether they're valid — which is exactly why they're the right first blocking gates. A secret scanner in particular pays for itself the first time it fires, and I'd push for it early regardless of what else is in place.
>
> **Week three: make it blocking, and make it fast.** Turn on branch protection so pull requests can't merge without a green build. This is the moment the culture shifts from 'CI exists' to 'CI matters', and it only works if the gate is under ten minutes and reliable. If either of those isn't true, I'd fix that before making it blocking, because a slow or flaky gate introduced as blocking on day one destroys trust immediately and you don't get a second chance.
>
> **Month two: deploy and smoke.** Automated deployment to a staging environment on merge, followed by a smoke suite of five to fifteen tests answering one question — is the application alive. This is where the pipeline starts delivering value to people other than developers, because now it's catching integration and configuration problems, not just code problems.
>
> **Month three: the full regression, nightly and sharded**, with results in Slack. Nightly rather than per-commit at first, because I want to see its stability before anyone depends on it.
>
> **Ongoing:** flake detection, quarantine SLA, duration tracking, and gradually shifting work earlier as the fast layers earn trust.
>
> Two things I'd do in parallel with all of it. First, **make the pipeline visible** — results in the channel people already read, a link that goes straight to the failure with screenshots attached. A pipeline nobody sees is a pipeline nobody trusts. Second, **be the person who fixes it when it breaks**, at least early on. The fastest way to lose a team is for the new gate to block them on a Friday and for the person who introduced it to be unavailable.
>
> And I'd introduce every new gate in warning mode first, for a week or two, so I can measure its false-positive rate on real traffic before it blocks anything. A gate introduced as blocking that turns out to be noisy is very hard to recover from politically, even after you fix it."

---

## 16.7 "Pipeline environments ke across test data kaise manage karoge?"

> **Interview answer:**
> "Test data is usually the thing that makes a pipeline unreliable long before the tests do, and my central principle is that **each test creates the data it needs and cleans up after itself**. Shared, pre-seeded fixtures that many tests depend on are the single largest source of cross-test coupling I've seen — one test mutates a record, twelve unrelated tests fail, and the failure looks random.
>
> **The strategies, from most to least preferred.**
>
> *Create on demand via API.* The test sets up its own data through the application's API in setup — faster and more reliable than building it through the UI — asserts on the records it created, and deletes them in teardown. Every record gets a unique identifier, typically a UUID fragment in the name, so parallel runs can't collide and so test data is identifiable in the database later.
>
> *Ephemeral database per run.* In containerised environments, bring up the database fresh for each pipeline run. I run it on tmpfs, in RAM — faster, and more importantly it guarantees a pristine state every time, so no test can be polluted by leftovers. This is the strongest isolation available and it's why I like the compose-based approach.
>
> *Seeded baseline plus per-test additions.* Where a full reset isn't possible, seed a small read-only reference set — currencies, unit types, a standard vendor list — that tests may read but never mutate, and have each test create its own mutable data on top. The rule has to be enforced in review: nothing modifies the shared baseline.
>
> *Transactional rollback* for API and integration tests where the framework supports it — run the test in a transaction and roll back. Very fast and perfectly isolated, but it doesn't work for end-to-end tests through a real HTTP boundary.
>
> **Across environments** the data must not be identical, and pretending otherwise causes problems. Staging can hold anonymised production-shaped data, which is valuable because real data contains edge cases nobody invents — odd characters in vendor names, zero-quantity line items, historical records with fields that no longer exist. Production, where I currently run, needs different discipline entirely: a dedicated automation account rather than a person's login, records created with an identifiable prefix so they can be excluded from business reporting and cleaned up, no destructive operations against real customer records, and awareness that test traffic appears in analytics.
>
> **Cleanup has to be guaranteed, not best-effort.** Teardown runs in a fixture finalizer so it happens even when the test fails, and I'd back that with a scheduled sweeper job that deletes anything matching the test-data prefix older than a day — because teardown will eventually fail when a run is cancelled or a runner is killed mid-test, and without a sweeper the environment degrades slowly until someone has to clean it manually.
>
> **What I'd avoid:** tests that depend on a specific record existing with a specific ID, assertions on global counts like 'the list should contain fifty items' — assert on the record you created instead — and any test that depends on another test having run first. Running the suite in random order is a cheap way to find those; if a suite only passes in one order, it has hidden dependencies that will surface as flakiness the moment you parallelise it.
>
> On a financial domain like an ERP there's one more consideration: test data has to be visibly test data. A purchase order created by automation must never be mistakable for a real commitment in a report someone bases a decision on. That's a naming and flagging convention, and it's worth agreeing explicitly with the product side rather than assuming."

---
---

# PART B — GIT

---

# 17. Git ka Mental Model

## 17.1 Char jagah — ye diagram dimaag mein fix kar lo

Git ke saare commands sirf **char areas ke beech cheezein move karte hain**. Agar ye picture clear hai to koi bhi command confuse nahi karega.

```
 ┌────────────────┐   ┌────────────────┐   ┌────────────────┐   ┌────────────────┐
 │    WORKING     │   │    STAGING     │   │  LOCAL REPO    │   │     REMOTE     │
 │   DIRECTORY    │   │  AREA (INDEX)  │   │   (.git dir)   │   │    (origin)    │
 │                │   │                │   │                │   │                │
 │  tumhari files │   │  agle commit   │   │  commits ka    │   │  GitHub /      │
 │  jaisi disk pe │   │  ke liye chuni │   │  poora graph   │   │  GitLab pe     │
 │  abhi hain     │   │  hui changes   │   │                │   │                │
 └────────────────┘   └────────────────┘   └────────────────┘   └────────────────┘
         │                     │                    │                    │
         │   git add           │   git commit       │   git push         │
         ├────────────────────>├───────────────────>├───────────────────>│
         │                     │                    │                    │
         │   git restore       │  git reset --soft  │   git fetch        │
         │<────────────────────┤<───────────────────┤<───────────────────┤
         │      --staged       │                    │                    │
         │                     │                    │                    │
         │<────────────────────────────────────────-┤                    │
         │        git checkout <commit> -- <file>   │                    │
         │                     │                    │                    │
         │<─────────────────────────────────────────────────────────────-┤
         │              git pull  (= fetch + merge/rebase)                │
         │                     │                    │                    │
         │<────────────────────┴────────────────────┤                    │
         │            git reset --hard              │                    │
         │       (DONO ko commit ke barabar kar do) │                    │
```

| Area | Kya hai | Kaise dekho |
|---|---|---|
| **Working directory** | Tumhari actual files, jo abhi disk pe hain | `ls`, editor mein |
| **Staging area (index)** | Ek "draft" of the next commit. Tumne jo `git add` kiya | `git diff --staged` |
| **Local repository** | `.git/` folder — saare commits, branches, tags | `git log` |
| **Remote** | GitHub/GitLab pe copy | `git remote -v`, `git log origin/main` |

**Staging area kyun exist karta hai (interview question):**

Ye Git ka ek distinctive feature hai. Iska point ye hai ki tum **commit ko curate** kar sako. Tumne 3 alag cheezein fix ki hain ek session mein — tum unhe 3 alag commits bana sakte ho, chahe wo ek hi file mein hon:

```bash
git add -p              # interactive: har hunk ke liye y/n poochhega
# sirf pehle change ke hunks stage karo
git commit -m "fix: correct tax rounding on trade line items"
# ab baaki
git add -p
git commit -m "refactor: extract vendor lookup into a helper"
```

Ye "atomic commits" enable karta hai — aur atomic commits hi `git bisect`, `git revert` aur code review ko useful banate hain.

## 17.2 Commits snapshots hain, diffs nahi — ye sabse important concept hai

**Ye galat mental model hai (SVN se aaya hua):** "Har commit ek change record karta hai — jo lines add/remove hui."

**Ye sahi hai:** Har commit poore project ka ek **complete snapshot** hai us waqt.

```
   COMMIT A                COMMIT B                COMMIT C
  ┌──────────┐            ┌──────────┐            ┌──────────┐
  │ tree ────┼──> {       │ tree ────┼──> {       │ tree ────┼──> {
  │          │      a.py  │          │      a.py  │          │      a.py'  <- badli
  │ parent:  │      b.py  │ parent:A │      b.py' │ parent:B │      b.py'
  │  (none)  │      c.py  │          │      c.py  │          │      c.py
  │          │    }       │          │    }       │          │      d.py   <- nayi
  │ author   │            │ author   │            │          │    }
  │ message  │            │ message  │            │ author   │
  └──────────┘            └──────────┘            │ message  │
   SHA: a1b2c3             SHA: d4e5f6            └──────────┘
                                                   SHA: 7g8h9i
```

Har commit mein hota hai:
- **tree** — poore project ka file structure us waqt (snapshot)
- **parent** — pichhla commit (ya merge mein do parents)
- **author / committer** — kaun, kab
- **message**
- **SHA-1 hash** — upar ki saari cheezon ka hash

**Efficiency ka sawaal — "har commit poora snapshot hai to repo huge nahi ho jaayega?"**

Nahi, kyunki Git **content-addressable storage** use karta hai. Agar `c.py` commit A se C tak nahi badli, to teeno commits ka tree **same blob object** ko point karta hai. File ek hi baar store hoti hai. Aur Git periodically packfiles banata hai jo delta compression use karte hain — **par wo storage optimisation hai, data model nahi.**

**Ye distinction kyun matter karta hai — practical implications:**

| Kyunki commits snapshots hain... | Isliye |
|---|---|
| Har commit independently checkout ho sakta hai | `git checkout <sha>` instant hai, replay nahi karna padta |
| Branch sirf ek **pointer** hai ek commit pe | Branch banana O(1) hai — ek 41-byte file likhna |
| Rebase = **naye commits banana** | Purane commits delete nahi hote, orphan ho jaate hain — reflog se recoverable |
| SHA content ka hash hai | Ek commit badalne pe uske baad ke **saare** SHAs badal jaate hain (isliye history rewrite ke baad force-push chahiye) |
| Diff **compute** hota hai, store nahi | `git diff A B` do trees compare karta hai on the fly |

**Interview mein ye bolna:**

> **Interview answer:**
> "Git's model is four areas: the working directory, the staging area or index, the local repository, and the remote. Every command is really just moving content between those, and once that's clear the command set stops feeling arbitrary.
>
> The staging area is Git's distinctive piece. It exists so you can curate a commit rather than being forced to commit everything you changed — `git add -p` lets you stage individual hunks, so three unrelated fixes made in one session become three atomic commits. That matters practically, because atomic commits are what make `git bisect`, `git revert` and code review actually work.
>
> The concept people most often have wrong is that commits store diffs. They don't — each commit is a complete snapshot of the project tree, plus a pointer to its parent, plus metadata, and its SHA is the hash of all of that. Diffs are computed between two snapshots on demand, not stored.
>
> That's not academic. It explains why branching is essentially free — a branch is a 41-byte file containing a commit SHA, so creating one is O(1) regardless of repository size. It explains why checking out an old commit is instant rather than replaying history. And it explains why rewriting history requires a force-push: since a commit's SHA is a hash of its content including its parent, changing any commit changes every SHA after it, so the rewritten branch is no longer a fast-forward from what's on the remote.
>
> It also explains recovery. A rebase doesn't destroy the old commits, it creates new ones and moves the branch pointer — the originals become unreferenced but still exist, which is exactly why reflog can recover them."

**Cross-question: "Agar har commit poora snapshot hai to storage kaise manageable rehta hai?"**

> **Interview answer:**
> "Because Git stores content, not files, and it deduplicates by content hash. If a file is unchanged between two commits, both commits' trees reference the identical blob object — it's stored once. So a thousand commits that each change one file don't store a thousand copies of the whole project; they store a thousand trees, which are tiny, plus one new blob each.
>
> On top of that Git periodically repacks loose objects into packfiles, which do use delta compression between similar objects. But that's a storage-layer optimisation applied after the fact — the logical data model is still snapshots. That distinction matters, because it means the delta chains are chosen for compression efficiency, not forced to follow parent-child history, and the model you reason about stays simple."

---

# 18. Repo Setup — clone, init, remote

## 18.1 init — naya repo

```bash
# Current directory ko git repo banao
git init

# Ya nayi directory ke saath
git init my-project

# Default branch ka naam set karo (modern default 'main' hai)
git init -b main

# Globally set karo taaki har naye repo mein 'main' bane
git config --global init.defaultBranch main
```

Kya hua: ek `.git/` directory bani. Bas. Wahi poora repository hai — delete kar do to Git history khatam.

## 18.2 clone — existing repo copy karo

```bash
# Basic
git clone https://github.com/merlinai/qa-automation.git

# Alag folder name ke saath
git clone https://github.com/merlinai/qa-automation.git merlin-tests

# SSH se (recommended — har baar password nahi maangega)
git clone git@github.com:merlinai/qa-automation.git

# Sirf ek branch
git clone --branch develop --single-branch <url>

# Shallow — sirf latest commit. CI mein bahut fast.
git clone --depth 1 <url>

# Shallow par saari branches ke refs ke saath
git clone --depth 1 --no-single-branch <url>
```

**Shallow clone ka gotcha (CI mein relevant):** `--depth 1` ke baad `git log`, `git blame`, `git bisect`, aur "changed files since main" — sab toot jaate hain, kyunki history hai hi nahi. GitHub Actions ka `actions/checkout@v4` default `fetch-depth: 1` karta hai. Agar tumhe history chahiye:

```yaml
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0        # poori history
```

Ya baad mein deepen karo:

```bash
git fetch --unshallow
```

## 18.3 remote — remotes manage karo

```bash
# Dekho kaunse remotes configured hain
git remote -v
# origin  git@github.com:merlinai/qa-automation.git (fetch)
# origin  git@github.com:merlinai/qa-automation.git (push)

# Naya remote add karo
git remote add origin git@github.com:merlinai/qa-automation.git

# URL badlo (HTTPS se SSH pe switch karna common hai)
git remote set-url origin git@github.com:merlinai/qa-automation.git

# Doosra remote — fork workflow mein
git remote add upstream git@github.com:original-org/repo.git

# Remote ki details
git remote show origin

# Hatao
git remote remove upstream

# Rename
git remote rename origin github
```

**Fork workflow (open source aur kabhi-kabhi internal mein bhi):**

```
   upstream (original repo)  <---- yahan se fetch karke apna fork sync karte ho
        │
        │ fork
        v
   origin (tumhara fork)     <---- yahan push karte ho, phir PR upstream pe
        │
        │ clone
        v
   local
```

```bash
git remote add upstream git@github.com:original/repo.git
git fetch upstream
git checkout main
git merge upstream/main          # ya: git rebase upstream/main
git push origin main             # apna fork sync ho gaya
```

## 18.4 Zaroori initial config

```bash
# Identity — commits mein yahi naam/email jaayega
git config --global user.name  "Ritik Chaturvedi"
git config --global user.email "ritik.chaturvedi@merlinai.co"

# Ek specific repo ke liye alag email
git config user.email "ritik@personal.com"

# Editor
git config --global core.editor "vim"       # ya "code --wait"

# Line endings — cross-platform team mein IMPORTANT
git config --global core.autocrlf input     # macOS/Linux
# git config --global core.autocrlf true    # Windows

# Pull ka default behaviour — ye zaroor set karo
git config --global pull.rebase true        # pull = fetch + rebase (cleaner)
# ya:
git config --global pull.ff only            # sirf fast-forward, warna error

# Push ka default — sirf current branch push karo
git config --global push.default current

# Naye branch pe push karne pe automatically upstream set ho
git config --global push.autoSetupRemote true

# Colours
git config --global color.ui auto

# Saara config dekho (aur kahan se aa raha hai)
git config --list --show-origin
```

**Useful aliases:**

```bash
git config --global alias.st  "status -sb"
git config --global alias.lg  "log --oneline --graph --decorate --all"
git config --global alias.last "log -1 HEAD --stat"
git config --global alias.unstage "restore --staged"
git config --global alias.amend "commit --amend --no-edit"
git config --global alias.wip "commit -am 'wip: checkpoint' --no-verify"
```

---

# 19. fetch vs pull vs push

## 19.1 Ye interview mein PAKKA poochha jaata hai

```
   git fetch                              git pull
   ─────────                              ────────
   remote se data laao,                   fetch + merge (ya rebase)
   working directory ko HAATH MAT LAGAO   working directory UPDATE kar do

   ┌────────────────────────┐             ┌────────────────────────┐
   │  origin/main  ●──●──●  │  <- updated │  origin/main  ●──●──●  │
   │                        │             │                     ↓  │
   │  main  ●──●            │  <- SAME    │  main  ●──●──●──●──●   │ <- merged
   │        ↑ tumhara kaam  │             │                        │
   │          untouched     │             │  working dir BADAL     │
   └────────────────────────┘             │  gayi                  │
                                          └────────────────────────┘
```

## 19.2 fetch

```bash
# Sab remotes se sab branches
git fetch --all

# Sirf origin
git fetch origin

# Specific branch
git fetch origin main

# Fetch + delete stale remote-tracking branches
# (remote pe branch delete ho gayi to local reference bhi hata do)
git fetch --prune
git fetch -p
```

Fetch ke baad **kuch nahi badla** tumhari working directory mein. Sirf `origin/main` jaise remote-tracking references update hue.

**Fetch ke baad kya karo — ye workflow QA ke liye khaas useful hai:**

```bash
git fetch origin

# Dekho remote pe kya naya aaya, MERGE karne se pehle
git log HEAD..origin/main --oneline

# Kaunsi files badli?
git diff HEAD origin/main --stat

# Poora diff
git diff HEAD origin/main

# Ab decide karo
git merge origin/main      # ya
git rebase origin/main
```

**Ye "look before you leap" pattern hai.** Interview mein bolna: *"I fetch and inspect before I integrate. `git log HEAD..origin/main` tells me exactly what's incoming, and `git diff --stat` tells me which files it touches — which matters when I'm about to rebase a branch with a lot of local work, because I can see the conflict surface before I create it."*

## 19.3 pull

```bash
# Default: fetch + merge
git pull

# fetch + rebase (cleaner history, no merge commit)
git pull --rebase

# Sirf fast-forward — agar diverge hua to error do
git pull --ff-only

# Specific
git pull origin main
```

**`git pull` = `git fetch` + `git merge FETCH_HEAD`** (ya `rebase` agar configured hai).

**Pull ka default khatarnak kyun ho sakta hai:** Agar tumhare local commits hain aur remote pe bhi naye commits hain, to `git pull` chupke se ek **merge commit** bana deta hai. History mein "Merge branch 'main' of github.com:..." jaise useless commits bhar jaate hain.

Isliye ye config recommended hai:

```bash
git config --global pull.rebase true
# ya safest:
git config --global pull.ff only
```

`pull.ff only` ke saath, agar diverge hua to Git error dega aur tumhe **explicitly** decide karna padega merge ya rebase — jo behtar hai.

## 19.4 push

```bash
# Current branch ko uske upstream pe
git push

# Pehli baar — upstream set karo
git push -u origin feature/po-custom-items
git push --set-upstream origin feature/po-custom-items    # same thing

# Specific
git push origin main

# Tags push karo (default mein NAHI jaate)
git push --tags
git push origin v1.2.0

# Branch delete karo remote pe
git push origin --delete feature/old-branch
git push origin :feature/old-branch          # purana syntax, same cheez

# FORCE — history rewrite ke baad
git push --force-with-lease      # <-- YE use karo
git push --force                 # <-- ye AVOID karo
```

### `--force` vs `--force-with-lease` — ye important hai

| | `--force` | `--force-with-lease` |
|---|---|---|
| Kya karta hai | Remote branch ko unconditionally overwrite | Overwrite tabhi jab remote wahi hai jo tumne aakhri baar dekha tha |
| Agar kisi aur ne push kiya beech mein | **Uska kaam delete ho gaya, chup-chaap** | **Reject ho jaayega** — tum bach gaye |
| Kab use karo | Kabhi nahi, practically | Rebase/amend ke baad hamesha |

```bash
# --force-with-lease safe hai kyunki wo check karta hai:
#   "kya origin/feature abhi bhi wahi commit hai jo mere local
#    remote-tracking ref mein hai?"
# Agar haan -> push. Agar nahi -> reject, kyunki kisi aur ne push kiya hai.

git push --force-with-lease origin feature/po-custom-items
```

**Gotcha:** Agar tumne `git fetch` kiya (jisse tumhara remote-tracking ref update ho gaya) par uske changes dekhe nahi, to `--force-with-lease` phir bhi allow kar dega. Isliye:

```bash
git push --force-with-lease=feature/po-custom-items:<expected-sha>
```

Ya simply: force-push se pehle fetch mat karo, ya fetch karke dekho kya aaya.

## 19.5 Interview answer

> **Interview answer:**
> "`git fetch` downloads new objects and updates the remote-tracking references like `origin/main`, but it doesn't touch your working directory or your local branch at all. `git pull` is `git fetch` followed immediately by a `git merge` — or a rebase, if you've configured it that way — so it does change your working state.
>
> The practical difference is control. Fetch lets me look before I leap. After fetching I can run `git log HEAD..origin/main` to see exactly what's incoming, and `git diff HEAD origin/main --stat` to see which files it touches. That matters when I'm about to integrate into a branch with substantial local work — I can see the conflict surface before I create it, and decide whether to merge or rebase based on what's actually there.
>
> I'd also mention a configuration point, because the default pull behaviour causes real problems. If you have local commits and the remote has moved on, a plain `git pull` silently creates a merge commit, and history fills with 'Merge branch main of github.com…' commits that carry no information. I set `pull.rebase true`, or on shared repositories `pull.ff only`, which makes Git refuse and force an explicit decision when the branches have diverged. Being forced to decide is better than a silent default.
>
> On push, the thing I'd emphasise is `--force-with-lease` over `--force`. Plain force overwrites the remote unconditionally, so if a teammate pushed while you were rebasing, their commits are gone silently. `--force-with-lease` only overwrites if the remote is still at the commit your remote-tracking reference expects — so if someone else pushed, the push is rejected and you get to find out rather than destroying their work. After any history rewrite, that's the only force I'd use."

---

# 20. Branch, checkout vs switch, restore

## 20.1 Branch kya hai — sabse simple definition

**Ek branch ek 41-byte file hai jisme ek commit ka SHA likha hai. Bas.**

```bash
cat .git/refs/heads/main
# a3f9c21e8b4d5f6a7c8e9d0b1a2c3d4e5f6a7b8c
```

Isliye branch banana instant hai — chahe repo mein 10 lakh commits hon.

**HEAD** ek aur pointer hai — wo batata hai ki tum abhi kahan ho:

```bash
cat .git/HEAD
# ref: refs/heads/main        <- normal state: HEAD ek branch ko point karta hai

# Detached HEAD state mein:
# a3f9c21e8b4d5f6a7c8e9d0b1a2c3d4e5f6a7b8c    <- seedha commit pe
```

```
        ┌──── HEAD
        │
        v
      main
        │
        v
  A ──> B ──> C
              ^
              │
           feature
```

## 20.2 Branch commands

```bash
# List local branches (* = current)
git branch

# Local + remote
git branch -a

# Sirf remote
git branch -r

# Zyada info — last commit, upstream, ahead/behind
git branch -vv

# Nayi branch banao (switch NAHI karo)
git branch feature/po-custom-items

# Banao aur switch karo
git switch -c feature/po-custom-items
git checkout -b feature/po-custom-items       # purana syntax

# Specific commit se branch banao
git switch -c hotfix/tax-rounding a3f9c21

# Remote branch se local banao aur track karo
git switch -c feature/x origin/feature/x
git switch feature/x                          # modern git ye auto-detect karta hai

# Rename current branch
git branch -m feature/new-name

# Rename doosri branch
git branch -m old-name new-name

# Delete (safe — agar merge nahi hui to refuse karega)
git branch -d feature/done

# Force delete (merge nahi hui to bhi)
git branch -D feature/abandoned

# Remote branch delete
git push origin --delete feature/done

# Wo branches jo main mein merge ho chuki hain (cleanup ke liye)
git branch --merged main
git branch --no-merged main

# Merged branches bulk delete
git branch --merged main | grep -v '\*\|main\|develop' | xargs -n 1 git branch -d
```

## 20.3 checkout vs switch vs restore — kyun teen commands hain

`git checkout` **overloaded** tha — wo do bilkul alag kaam karta tha:
1. Branch badalna
2. File ko restore karna

Ye confusing aur khatarnak tha (`git checkout file.py` chup-chaap tumhare changes delete kar deta tha). Git 2.23 (2019) mein isse do commands mein toda gaya:

| Purana | Naya | Kaam |
|---|---|---|
| `git checkout <branch>` | **`git switch <branch>`** | Branch badlo |
| `git checkout -b <new>` | **`git switch -c <new>`** | Nayi branch banao + switch |
| `git checkout <sha>` | **`git switch --detach <sha>`** | Detached HEAD |
| `git checkout -- <file>` | **`git restore <file>`** | File ke changes discard karo |
| `git checkout <sha> -- <file>` | **`git restore --source=<sha> <file>`** | File ko purane version se laao |
| `git reset HEAD <file>` | **`git restore --staged <file>`** | Unstage karo |

`git checkout` abhi bhi kaam karta hai (backward compatible), par **naya code `switch`/`restore` use kare.** Interview mein ye bolna modern awareness dikhata hai.

## 20.4 switch

```bash
# Branch badlo
git switch main

# Banao aur switch
git switch -c feature/bid-custom-items

# Pichhli branch pe wapas (cd - jaisa)
git switch -

# Detached HEAD — specific commit pe jao
git switch --detach a3f9c21

# Uncommitted changes ke saath forcefully switch (changes CARRY hote hain)
git switch -f main       # ya --discard-changes se changes chhod do
```

**Detached HEAD kya hai:** Jab HEAD kisi branch ki jagah seedha ek commit ko point kare. Tum yahan commit kar sakte ho, par wo commits kisi branch pe nahi hain — switch karte hi orphan ho jaayenge (reflog se recoverable, par easily kho jaate hain).

```bash
git switch --detach a3f9c21
# ... kuch experiment kiya, commit kiya ...
# Ab agar tum switch karoge to ye commits khо jaayenge

# Bachane ke liye branch bana do:
git switch -c experiment/my-idea
```

## 20.5 restore

```bash
# Working directory ke changes discard karo (file ko HEAD wale version pe wapas)
git restore tests/e2e/test_po.py

# Saari files
git restore .

# Unstage karo (staging se hatao, changes working dir mein rakho)
git restore --staged tests/e2e/test_po.py

# Dono: unstage + discard
git restore --staged --worktree tests/e2e/test_po.py

# Kisi specific commit ka version laao
git restore --source=a3f9c21 tests/e2e/test_po.py
git restore --source=HEAD~3 conftest.py

# Doosri branch se file laao
git restore --source=main conftest.py
```

**`git restore <file>` DESTRUCTIVE hai** — un changes ko wapas nahi laaya jaa sakta jo commit nahi hue the. Ye un chhoti commands mein se hai jinse log data khote hain.

## 20.6 Interview answer

> **Interview answer:**
> "A branch in Git is just a movable pointer — literally a small file containing a commit SHA. That's why branching is effectively free regardless of repository size, and it's why Git encourages branching where older systems discouraged it. HEAD is a second pointer that says where you currently are, normally pointing at a branch rather than directly at a commit.
>
> On `checkout` versus `switch` and `restore`: `git checkout` was historically overloaded — it changed branches *and* restored files, which is two unrelated operations sharing one command. That was genuinely dangerous, because `git checkout somefile.py` silently discarded uncommitted work with no confirmation and no undo. Git 2.23 split it: `switch` changes branches, `restore` changes file contents. I use those in anything I write or teach, because the intent is explicit and the destructive operation is clearly named.
>
> The related concept I'd mention is detached HEAD, because it confuses people. It means HEAD points directly at a commit rather than at a branch. You can commit there, but those commits belong to no branch, so switching away leaves them unreferenced — recoverable through reflog, but easy to lose. If I've done work in a detached HEAD I want to keep, the fix is one command: `git switch -c some-branch` to give it a name before moving."

---

# 21. add, commit, amend

## 21.1 add — staging area mein daalo

```bash
# Specific file
git add tests/e2e/test_po_material.py

# Directory
git add tests/

# Sab kuch (tracked + untracked, gitignore respect karke)
git add .

# Sab kuch repo root se, chahe tum kisi subdirectory mein ho
git add -A
git add --all

# Sirf tracked files ke changes (naye untracked files nahi)
git add -u

# INTERACTIVE — hunk by hunk. Ye seekho, ye powerful hai.
git add -p

# Sirf ek file ke hunks
git add -p conftest.py
```

**`git add -p` ka interactive menu:**

```
Stage this hunk [y,n,q,a,d,s,e,?]?
  y - haan, ye hunk stage karo
  n - nahi, skip
  q - quit, aur kuch stage mat karo
  a - is file ke ye aur baaki saare hunks stage karo
  d - is file ke ye aur baaki hunks skip karo
  s - is hunk ko chhote hunks mein SPLIT karo
  e - hunk ko manually EDIT karo
  ? - help
```

Ye **atomic commits** banane ka main tool hai. Ek session mein 3 alag cheezein fix ki? `add -p` se unhe 3 commits mein baanto.

## 21.2 commit

```bash
# Editor kholega message ke liye
git commit

# Inline message
git commit -m "fix: correct tax rounding on trade line items"

# Multi-line
git commit -m "fix: correct tax rounding on trade line items" \
           -m "Trade items were rounding at the line level instead of the order level, producing totals that differed by up to 2 paise from the invoice. Rounding now happens once, at the order total."

# add + commit ek saath (sirf TRACKED files — naye files nahi)
git commit -am "test: add bid custom item flow"

# Empty commit (CI trigger karne ke liye useful)
git commit --allow-empty -m "ci: retrigger pipeline"

# Hooks skip karo (savdhani se!)
git commit --no-verify -m "wip"
```

### Achha commit message — Conventional Commits

```
<type>(<scope>): <short summary, imperative mood, <= 72 chars>
                                 <- blank line
<body: WHY, not what. Kya problem thi, kya approach li, kya trade-off.>
                                 <- blank line
<footer: Refs MER-4821, BREAKING CHANGE: ...>
```

| Type | Kab |
|---|---|
| `feat` | Nayi feature |
| `fix` | Bug fix |
| `test` | Test add/fix (**QA ka sabse common**) |
| `ci` | Pipeline changes |
| `refactor` | Behaviour nahi badla, structure badla |
| `docs` | Documentation |
| `chore` | Maintenance, deps |
| `perf` | Performance |

**Achhe examples (tumhare project se):**

```
test(po): add end-to-end flow for custom line items

Covers the PO custom-item path which had no automated coverage. The
flow exercises 14 steps from creation through vendor assignment to
submission, and asserts the computed total rather than a fixed value,
so it stays valid when tax rates change.

Refs MER-4821
```

```
ci: shard the regression suite across six runners

Serial regression was 30 minutes, which was long enough that the team
had started merging without waiting for it. Splitting by recorded test
duration brings wall-clock to about 6 minutes.

Durations are committed in .test_durations and refreshed nightly;
without them pytest-split falls back to test count and the shards come
out badly unbalanced.
```

**Bure examples:**

```
fix                          <- kya fix?
updated files                <- kaunsi, kyun?
asdf                         <- ...
Fixed the thing that was broken as discussed in the meeting yesterday   <- kaunsi meeting?
```

**Body mein WHY likho, WHAT nahi.** *What* to diff mein dikh raha hai. *Why* sirf tumhare dimaag mein hai, aur 6 mahine baad wo bhi nahi hoga.

## 21.3 amend — pichhla commit badlo

```bash
# Sirf message badlo
git commit --amend -m "test(po): add end-to-end flow for custom line items"

# Editor kholo message badalne ke liye
git commit --amend

# Bhooli hui file add karo, message wahi rakho
git add tests/e2e/test_po_custom.py
git commit --amend --no-edit

# Author badlo (galat email se commit ho gaya)
git commit --amend --author="Ritik Chaturvedi <ritik.chaturvedi@merlinai.co>"

# Commit date badlo
git commit --amend --date="2026-08-21T10:00:00"
```

### Amend ke baare mein sabse important baat

**Amend commit ko "edit" nahi karta — wo ek BILKUL NAYA commit banata hai aur branch pointer usko move kar deta hai.**

```
  BEFORE:                          AFTER amend:

  A ── B ── C                      A ── B ── C        <- orphan (reflog mein hai)
            ^                                │
            │                                └── C'   <- naya commit, naya SHA
          main                                    ^
                                                  │
                                                main
```

Iske do implications:

1. **Purana commit gaya nahi hai** — reflog mein hai. `git reflog` se recover ho sakta hai.
2. **SHA badal gaya** — matlab agar tumne wo commit already push kiya tha, to ab **force-push chahiye**.

### GOLDEN RULE — amend

**Sirf un commits ko amend karo jo tumne push NAHI kiye hain.**

Agar push ho chuka hai aur tum amend karke force-push karte ho, to jisne bhi wo commit pull kiya hai uski history tumhari se diverge kar gayi hai — aur jab wo pull karega to ajeeb conflicts aayenge.

Exception: tumhari apni feature branch jispe koi aur kaam nahi kar raha. Wahan amend + `--force-with-lease` normal aur acceptable hai (PR review feedback address karne ka standard tareeka).

```bash
# Feature branch pe, PR review ke baad
git add .
git commit --amend --no-edit
git push --force-with-lease
```

## 21.4 Interview answer

> **Interview answer:**
> "`git add` moves changes into the staging area, and the tool I'd highlight is `git add -p`, which stages hunk by hunk. That's how you get atomic commits when you've fixed three unrelated things in one session — and atomic commits are what make `git bisect`, `git revert` and code review actually work. If a commit does five things, you can't revert one of them and you can't attribute a bisect result to a specific change.
>
> For messages I follow Conventional Commits — a type and scope, a short imperative summary, then a body that explains *why*. The *what* is visible in the diff; the *why* only exists in the author's head and won't exist there in six months either. As a QA I write a lot of `test:` and `ci:` commits, and I try to record the reasoning — why this scenario was worth automating, or what problem a pipeline change solves — because that's the context that makes the change reviewable.
>
> `git commit --amend` replaces the previous commit, and the important detail is that it doesn't edit it — it creates a new commit with a new SHA and moves the branch pointer. The original becomes unreferenced but survives in reflog, so it's recoverable.
>
> The golden rule is that amend is safe on commits you haven't pushed. Because the SHA changes, an amended commit that was already pushed requires a force-push, and anyone who pulled the original now has diverged history that will produce confusing conflicts. The exception I'd allow is your own feature branch that nobody else is working on — amending and force-pushing with `--force-with-lease` is the standard way to address review feedback without cluttering history with 'fix review comment' commits."

---

# 22. Merge vs Rebase

## 22.1 Setup — dono cases ka starting point

```
                    D ── E              <- feature
                   /
   A ── B ── C ── F ── G                <- main
```

Tumne `feature` ko commit C se kaata. Us ke baad `main` pe F aur G aa gaye. Ab tumhe main ke changes chahiye (ya feature ko main mein daalna hai).

---

## 22.2 MERGE

```bash
git switch feature
git merge main
```

**AFTER:**

```
                    D ── E ─────── M    <- feature (M = merge commit)
                   /              /
   A ── B ── C ── F ── G ─────────      <- main
```

`M` ek **merge commit** hai — uske **do parents** hain (E aur G). Ye Git ka wo ek case hai jahan commit ke do parents hote hain.

**Merge ki khaasiyat:**
- Kuch bhi rewrite nahi hota — D aur E waise ke waise hain, same SHAs
- History **sach** dikhati hai — ye actually kaise hua tha
- **Non-destructive** — safe on shared branches

### Fast-forward merge — special case

Agar `main` pe commit C ke baad kuch nahi aaya:

```
   BEFORE:                              AFTER (fast-forward):

                D ── E   <- feature                  D ── E   <- feature, main
               /                                    /
   A ── B ── C            <- main       A ── B ── C
```

Git ko koi merge commit banane ki zaroorat nahi — wo bas `main` pointer ko aage sarka deta hai. Isliye "fast-forward".

```bash
# Fast-forward allow karo (default)
git merge feature

# Fast-forward possible ho tab bhi merge commit BANAO
git merge --no-ff feature

# SIRF fast-forward — agar diverge hua to fail karo
git merge --ff-only feature
```

**`--no-ff` kab useful hai:** Jab tum chahte ho ki history mein dikhe ki "ye ek feature branch thi jo yahan merge hui". GitFlow isi ko prefer karta hai. Iska fayda: `git revert -m 1 <merge-commit>` se poori feature ek command mein revert ho jaati hai.

### Three-way merge

Jab dono branches aage badhi hain, Git **teen** commits dekhta hai:
1. **Merge base** — common ancestor (yahan C)
2. Tumhari branch ka tip (E)
3. Doosri branch ka tip (G)

```
        merge base
             │
             v
   A ── B ── C ── F ── G     <- "theirs"
             │         ^
             │         └──── incoming
             │
              ╲ D ── E       <- "ours"
                     ^
                     └────── current
```

Git base se dono taraf ke changes compare karta hai:
- Sirf ek taraf badla → wo change le lo
- Dono taraf badla, **alag lines** → dono le lo
- Dono taraf badla, **same lines** → **CONFLICT**

---

## 22.3 REBASE

```bash
git switch feature
git rebase main
```

**AFTER:**

```
                          D' ── E'      <- feature (NAYE commits, naye SHAs)
                         /
   A ── B ── C ── F ── G                <- main
```

Rebase kya karta hai, step by step:
1. `feature` ke commits (D, E) ko dhoondho jo `main` mein nahi hain
2. Unko temporary jagah save karo (patches ke roop mein)
3. `feature` ko `main` ke tip (G) pe reset karo
4. Har saved commit ko **ek-ek karke replay** karo, naye commits banate hue

**Rebase ki khaasiyat:**
- History **linear** ho gayi — koi merge commit nahi
- D aur E **naye commits** hain (D' aur E') — **naye SHAs**
- Original D aur E orphan ho gaye (reflog mein hain)
- History ab "clean" hai par ye **jhooth** bol rahi hai — aisa lag raha hai ki tumne G ke baad kaam shuru kiya, jabki tumne C ke baad kiya tha

### Rebase commands

```bash
# Basic
git rebase main

# Conflict aane pe:
#   theek karo, phir:
git add <resolved-files>
git rebase --continue

#   ya ye commit skip karo:
git rebase --skip

#   ya sab cancel, waapas original state:
git rebase --abort

# INTERACTIVE — history edit karo
git rebase -i HEAD~4
git rebase -i main

# Pull ke saath
git pull --rebase

# Rebase ke dauraan conflicts automatically resolve karne ke liye
# purane resolutions yaad rakho
git config --global rerere.enabled true
```

### Interactive rebase — history clean karna

```bash
git rebase -i HEAD~4
```

Editor khulega:

```
pick a1b2c3d test(po): add custom item flow skeleton
pick d4e5f6a wip
pick 7g8h9i0 fix typo
pick b1c2d3e test(po): assert computed total

# Commands:
# p, pick   = commit waise hi rakho
# r, reword = commit rakho, message badlo
# e, edit   = ruk jao yahan, commit amend karne do
# s, squash = pichhle commit mein milao, message combine karo
# f, fixup  = squash jaisa, par is commit ka message DROP kar do
# d, drop   = commit hata do
# x, exec   = shell command chalao
# b, break  = yahan ruk jao
#
# LINES KA ORDER BADAL SAKTE HO — commits reorder ho jaayenge
```

Practical use — 4 messy commits ko 1 clean commit banao:

```
pick a1b2c3d test(po): add custom item flow skeleton
f    d4e5f6a wip
f    7g8h9i0 fix typo
f    b1c2d3e test(po): assert computed total
```

Result: ek commit, `a1b2c3d` ka message, saare changes ke saath.

**`exec` ka QA-relevant use — har commit pe test chalao:**

```bash
git rebase -i main --exec "pytest tests/unit -q"
```

Ye har replayed commit ke baad tests chalayega. Agar koi commit tests todta hai to rebase wahin ruk jaayega. Ye **bisect-friendly history** banane ka tareeka hai — interview mein bolne layak.

---

## 22.4 Merge vs Rebase — comparison

| Dimension | Merge | Rebase |
|---|---|---|
| History shape | Branched, graph | **Linear** |
| Commit SHAs | **Unchanged** | **Sab naye** |
| Merge commit banta hai | Haan (jab tak ff na ho) | Nahi |
| Existing history rewrite | **Nahi** | **Haan** |
| Shared branch pe safe | **Haan** | **NAHI** |
| Conflicts kab aate hain | Ek baar, merge ke waqt | **Har commit pe alag** ho sakte hain |
| Conflict resolution effort | Ek baar | Potentially N baar |
| `git log` padhne mein | Complex par sach | Simple par sanitised |
| `git bisect` | Kaam karta hai | **Behtar kaam karta hai** (linear) |
| `git revert` a feature | Ek merge commit revert karo | Har commit alag revert |
| "Kab merge hua" ka record | **Preserved** | **Lost** |
| Force-push chahiye | Nahi | **Haan** (agar push ho chuka tha) |

## 22.5 THE GOLDEN RULE OF REBASE

> ## **Kabhi bhi aisi branch ko rebase mat karo jo kisi aur ke paas bhi ho.**

Kyun — concretely kya hota hai:

```
  Tum aur Priya dono `feature` pe kaam kar rahe ho.

  Remote:   A ── B ── C ── D ── E        <- origin/feature
  Priya ka local: same, aur usne F bana liya (D ke upar)

  Tum rebase karte ho:
  Tumhara local: A ── B ── C' ── D' ── E'    (naye SHAs)
  git push --force

  Remote ab:  A ── B ── C' ── D' ── E'

  Priya pull karti hai:
  Uska Git dekhta hai C, D, E (uske local mein) aur C', D', E' (remote pe).
  Content same hai par SHAs alag hain. Git inhe ALAG commits samajhta hai.
  Nateeja: Priya ko har commit DO baar dikhta hai, ya usse aisa merge
  karna padta hai jo duplicate history banata hai, ya wo apna kaam kho deti hai.
```

**Safe rebase targets:**
- Tumhari local branch jo push nahi hui
- Tumhari feature branch jo push hui hai par **koi aur uspe kaam nahi kar raha** (PR branch)

**Never rebase:**
- `main`, `develop`, `release/*` — koi bhi shared branch
- Koi bhi branch jisse kisi aur ne branch kaati hai

## 22.6 Kab kya use karo — practical policy

| Situation | Use | Kyun |
|---|---|---|
| Feature branch ko main ke latest ke saath update karna | **Rebase** | Linear history, koi noise merge commit nahi |
| Feature branch ko main mein daalna | **Merge (squash ya --no-ff)** | Main ki history preserve, revert easy |
| Shared branch jispe teammate bhi hai | **Merge** | Rebase unki history todega |
| PR ke commits clean karna merge se pehle | **Interactive rebase** | Reviewer ko readable history mile |
| `main` ko `main` mein integrate karna (pull) | **`pull --rebase`** | Merge commit noise se bachao |
| Hotfix ko release branch pe le jaana | **cherry-pick** | Sirf ek commit chahiye |
| Long-lived branch jo bahut behind hai | **Merge** | Rebase mein har commit pe conflict aayega |

> **[REAL]** Tumhare Merlin automation repo ke liye recommended policy jo interview mein bolna:
>
> *"On my automation repo I rebase my feature branch onto main to keep it current — that keeps the branch's own history linear and makes review straightforward — and then use a squash merge into main. So main gets one commit per change with a meaningful message, which makes it genuinely bisectable. I only rebase branches nobody else is working on, and I use `--force-with-lease` rather than `--force` when pushing after a rebase."*

## 22.7 Interview answer

> **Interview answer:**
> "Both integrate changes from one branch into another, but they do it very differently.
>
> Merge creates a new commit with two parents, joining the two histories. Nothing existing is rewritten — every commit keeps its SHA — and the resulting graph is an accurate record of what actually happened, including when the integration occurred.
>
> Rebase takes your branch's commits, sets your branch to the tip of the target, and replays each commit on top one at a time. The result is linear, but every replayed commit is a *new* commit with a new SHA. The originals become unreferenced — recoverable through reflog, but no longer part of the branch.
>
> The trade-off is honesty versus readability. Merge history is true but noisy; rebased history is clean but presents a sequence of events that never happened — it looks as though you started your work after everyone else's, when you didn't. Both are defensible, and I'd care more about the team having one consistent policy than about which one it picks.
>
> The golden rule is: never rebase a branch that anyone else has. Because rebasing changes SHAs, a collaborator's Git sees the rewritten commits as entirely new ones even though the content is identical. When they pull, they get duplicated history or a confusing merge, and it's easy for someone to lose work in the cleanup. Rebasing my own unpushed work, or my own PR branch that nobody else is on, is fine — that's routine.
>
> One practical point on conflicts: with merge you resolve once, at the merge. With rebase you can hit conflicts on every replayed commit, because each is applied in sequence against a moving base. For a long-lived branch that's badly out of date, that's genuinely painful, and merge is usually the pragmatic choice there. Turning on `rerere` helps, since Git then remembers how you resolved a given conflict and reapplies it automatically.
>
> My own default is: rebase my feature branch onto main to stay current, then squash-merge into main. Main ends up with one meaningful commit per change, which makes `git bisect` and `git revert` genuinely useful — and as a QA, bisect is a tool I rely on to find which change broke a test, so I care about history being bisectable."

**Cross-question: "Squash merge ke nuksaan kya hain?"**

> **Interview answer:**
> "Three, and they're real.
>
> First, you lose the granular history. If a feature was thirty thoughtful commits, main now has one, so `git blame` on any line in that feature points at the squash commit and tells you nothing about the specific reasoning. On a large feature that's a genuine loss of context.
>
> Second, it complicates repeated merges from the same branch. Because the squash commit isn't a merge commit, Git doesn't record that the feature branch was integrated. If someone keeps working on that branch and merges again, Git can present changes it has effectively already seen. Deleting the branch after merge avoids this, which is why squash-merge workflows usually pair with auto-delete.
>
> Third, cherry-picking between long-lived branches gets harder, because the individual commits no longer exist to pick.
>
> Where I'd still choose it: repositories where most branches are small, and where main's readability matters more than branch archaeology — which is most application and test repositories. Where I'd avoid it: long-lived release branches that need selective cherry-picks, and large features where the internal commit sequence carries real reasoning. A reasonable middle ground is to require that the branch's own commits are cleaned up with an interactive rebase before merge, then use `--no-ff` — you get both a readable main and the preserved detail."

---

# 23. Conflict Resolution — step by step worked example

## 23.1 Conflict kab hota hai

Conflict tab hota hai jab **dono branches ne SAME file ki SAME lines** badli hain, aur Git decide nahi kar sakta kaunsi rakhni hai.

Git ye cheezein **automatically** handle kar leta hai (conflict nahi hota):
- Alag files badli
- Same file, **alag lines** badli
- Ek taraf change, doosri taraf koi change nahi

Conflict tab hi hota hai jab overlap ho.

## 23.2 Worked example — tumhare project se

### Setup

`main` branch pe `conftest.py` hai:

```python
# conftest.py (commit C — dono branches ka common ancestor)
import pytest

BASE_URL = "https://app.merlinai.co"
TIMEOUT = 30000


@pytest.fixture
def base_url():
    return BASE_URL
```

**Tum** apni branch `feature/staging-support` pe kaam kar rahe ho. Tumne staging support add kiya:

```python
# conftest.py (tumhari branch)
import os
import pytest

BASE_URL = os.environ.get("BASE_URL", "https://staging-app.merlinai.co")
TIMEOUT = 30000


@pytest.fixture
def base_url():
    return BASE_URL
```

**Priya** ne `main` pe timeout ka issue fix kiya:

```python
# conftest.py (main)
import pytest

BASE_URL = "https://app.merlinai.co"
TIMEOUT = 60000        # CI runners are slower — bumped from 30s


@pytest.fixture
def base_url():
    return BASE_URL
```

### Ab tum rebase karte ho

```bash
git switch feature/staging-support
git fetch origin
git rebase origin/main
```

Output:

```
Auto-merging conftest.py
CONFLICT (content): Merge conflict in conftest.py
error: could not apply a1b2c3d... feat: read base URL from environment
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
Could not apply a1b2c3d... feat: read base URL from environment
```

### Step 1 — Situation samjho

```bash
git status
```

```
interactive rebase in progress; onto 7g8h9i0
Last command done (1 command done):
   pick a1b2c3d feat: read base URL from environment
No commands remaining.
You are currently rebasing branch 'feature/staging-support' on '7g8h9i0'.

Unmerged paths:
  (use "git restore --staged <file>..." to unstage)
  (use "git add <file>..." to mark resolution)
	both modified:   conftest.py
```

```bash
# Sirf conflicted files list karo
git diff --name-only --diff-filter=U
# conftest.py
```

### Step 2 — File kholo, markers samjho

```python
import os
import pytest

<<<<<<< HEAD
BASE_URL = "https://app.merlinai.co"
TIMEOUT = 60000        # CI runners are slower — bumped from 30s
=======
BASE_URL = os.environ.get("BASE_URL", "https://staging-app.merlinai.co")
TIMEOUT = 30000
>>>>>>> a1b2c3d (feat: read base URL from environment)


@pytest.fixture
def base_url():
    return BASE_URL
```

### Markers ka matlab — ye EXACTLY samajh lo

```
<<<<<<< HEAD
    ^-- Yahan se "OURS" shuru — jispe tum apply kar rahe ho
        REBASE mein: ye TARGET branch hai (main) — confusing!
        MERGE  mein: ye CURRENT branch hai (jahan tum ho)

=======
    ^-- Separator. Upar "ours", neeche "theirs".

>>>>>>> a1b2c3d (feat: read base URL from environment)
    ^-- "THEIRS" khatam. SHA aur message batate hain ye kaunsa commit hai.
        REBASE mein: ye TUMHARA commit hai jo replay ho raha hai
        MERGE  mein: ye INCOMING branch hai
```

> ### **Rebase mein "ours" aur "theirs" ULTE ho jaate hain — ye interview mein poochha jaata hai**
>
> Rebase ke dauraan tum apne commits ko target branch ke **upar** replay kar rahe ho. To Git ke perspective se:
> - **"ours" = target branch (main)** — kyunki wo base hai jispe apply ho raha hai
> - **"theirs" = tumhara commit** — kyunki wo "aa raha hai"
>
> Merge mein ye seedha hai: "ours" = tumhari branch, "theirs" = jo merge ho rahi hai.
>
> Ye ulta-pan bahut logon ko galat resolution karwa deta hai. **Marker ke naam pe bharosa mat karo — content padho aur SHA dekho.**

Agar 3-way markers on kar do to aur clear ho jaata hai:

```bash
git config --global merge.conflictStyle zdiff3
```

Phir conflict aisa dikhta hai:

```python
<<<<<<< HEAD
BASE_URL = "https://app.merlinai.co"
TIMEOUT = 60000
||||||| a3f9c21                              <-- COMMON ANCESTOR (base)
BASE_URL = "https://app.merlinai.co"
TIMEOUT = 30000
=======
BASE_URL = os.environ.get("BASE_URL", "https://staging-app.merlinai.co")
TIMEOUT = 30000
>>>>>>> a1b2c3d
```

**Ab bilkul clear hai:** base mein `TIMEOUT = 30000` tha. Priya ne use 60000 kiya. Tumne use nahi chhua (30000 hi hai), tumne sirf BASE_URL badla. **`zdiff3` ON karo — ye single best git config hai conflict resolution ke liye.** Interview mein ye bolna practical experience dikhata hai.

### Step 3 — Resolve karo

Ab decision clear hai: **dono changes chahiye.** Priya ka timeout fix + tumhara environment support.

```python
import os
import pytest

BASE_URL = os.environ.get("BASE_URL", "https://staging-app.merlinai.co")
TIMEOUT = 60000        # CI runners are slower — bumped from 30s


@pytest.fixture
def base_url():
    return BASE_URL
```

**Saare markers (`<<<<<<<`, `=======`, `|||||||`, `>>>>>>>`) hata do.** Wo code nahi hain.

### Step 4 — Verify karo ki koi marker bacha nahi

```bash
# Ye search karo — koi result nahi aana chahiye
grep -rn '^<<<<<<<\|^=======\|^>>>>>>>' .

# Python file hai to syntax check karo
python -m py_compile conftest.py

# Aur behtar: tests collect ho rahe hain?
pytest --collect-only -q
```

**Ye step skip mat karo.** Committed conflict markers ek classic embarrassing bug hain — file syntactically invalid ho jaati hai aur poori suite fail hoti hai.

### Step 5 — Resolved mark karo

```bash
git add conftest.py

# Verify
git status
# interactive rebase in progress; onto 7g8h9i0
# Changes to be committed:
#	modified:   conftest.py
```

**`git add` ka matlab yahan "stage karo" nahi hai — iska matlab "maine ye conflict resolve kar diya" hai.** Ye Git ko batata hai ki file ab conflicted state mein nahi hai.

### Step 6 — Continue karo

```bash
git rebase --continue
```

Git commit message ke liye editor kholega (rebase mein). Save karo.

```
Successfully rebased and updated refs/heads/feature/staging-support.
```

### Step 7 — Verify aur push

```bash
# History dekho
git log --oneline --graph -5

# Tests chalao — resolution ne kuch toda to nahi
pytest tests/unit -q

# Rebase ke baad force-push chahiye (SHAs badal gaye)
git push --force-with-lease
```

---

## 23.3 Escape hatches

```bash
# Sab cancel karo, rebase se pehle wali state pe wapas
git rebase --abort

# Merge ke case mein
git merge --abort

# Ye commit skip kar do (uska change chhod do)
git rebase --skip

# Ek file ke liye poori tarah ek side choose karo
git checkout --ours conftest.py       # rebase mein: TARGET branch ka version
git checkout --theirs conftest.py     # rebase mein: TUMHARA commit ka version
git add conftest.py

# Conflict ko dobara se shuru karo (resolution galat ho gayi)
git checkout --merge conftest.py      # markers wapas laao
```

## 23.4 Tools

```bash
# Configured merge tool kholo (vimdiff, meld, kdiff3, VS Code)
git mergetool

# VS Code ko merge tool banao
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'

# Backup .orig files na banaye
git config --global mergetool.keepBackup false

# rerere — "reuse recorded resolution"
# Ek baar resolve kiya to Git yaad rakhega aur agli baar auto-apply karega
git config --global rerere.enabled true
```

**`rerere` khaas kab bachaata hai:** Long-lived branch jise tum baar-baar rebase kar rahe ho — har rebase pe wahi conflict aata hai. `rerere` ke saath tum pehli baar resolve karte ho, phir Git khud kar leta hai. Ye rebase ke sabse bade dard ka ilaaj hai.

## 23.5 Conflict kam karne ki strategy

| Strategy | Kaise |
|---|---|
| **Chhote, short-lived branches** | Sabse effective. 2 din ki branch mein 2 hafte se kam conflict. |
| **Roz main se integrate karo** | Chhote conflicts baar-baar > bada conflict ek baar |
| **Files ko chhota rakho** | Ek 2000-line `conftest.py` conflict magnet hai. Modular fixtures. |
| **Team boundaries agree karo** | Do log same file na chhuen, ya coordinate karein |
| **Auto-generated files commit mat karo** | Lockfiles, `.test_durations` — inpe conflicts constant hote hain |
| **`.gitattributes` merge strategies** | Kuch files ke liye custom merge driver |
| **`rerere` on rakho** | Repeated conflicts automatic |

## 23.6 Interview answer

> **Interview answer:**
> "A conflict happens when two branches change the same lines of the same file and Git can't determine which version is correct. Different files, or different regions of the same file, merge automatically — it's only genuine overlap that conflicts.
>
> My process is: run `git status` to see which files are unmerged, open each one, and read the markers. Everything between `<<<<<<<` and `=======` is 'ours' and everything from `=======` to `>>>>>>>` is 'theirs'. The trap worth knowing is that during a rebase those are inverted relative to intuition — because you're replaying your commits onto the target, 'ours' is the target branch and 'theirs' is your own commit. That inversion causes a lot of wrong resolutions, so I read the content and the commit SHA on the marker rather than trusting the labels.
>
> The single configuration change I'd recommend to anyone is `merge.conflictStyle = zdiff3`. It adds the common ancestor into the conflict block, so instead of guessing what each side intended you can see what the code looked like before either change. That turns most conflicts from a judgement call into an obvious one — you can see immediately that one side changed the timeout and the other changed the URL, so you want both.
>
> Then I resolve by understanding both intentions rather than picking a side. Usually both changes are wanted and the answer is a combination, not a winner. I remove every marker, verify with a grep for leftover markers and a syntax check, and — importantly — run the tests, because a syntactically valid resolution can still be semantically wrong. Then `git add` to mark it resolved, which in a conflict means 'I've dealt with this' rather than 'stage this', and `git rebase --continue`.
>
> If I'm out of my depth, `git rebase --abort` returns everything to the pre-rebase state, and I'd rather abort and think than resolve badly under pressure.
>
> The strategic point I'd make is that conflict resolution is mostly a prevention problem. The best predictor of conflict pain is branch age — a two-day branch rarely conflicts badly, a three-week branch reliably does. So short-lived branches and frequent integration matter more than any resolution technique. And I'd turn on `rerere`, which records how you resolved a conflict and reapplies it automatically the next time the same conflict appears — that's what makes repeatedly rebasing a longer-lived branch bearable."

---

# 24. cherry-pick

## 24.1 Kya hai

Kisi bhi branch se **ek specific commit** utha ke apni current branch pe apply karo.

```
   BEFORE:                              AFTER cherry-pick F:

   main     A ── B ── C                 main     A ── B ── C
                       \                                    \
   hotfix               D ── E ── F     hotfix               D ── E ── F
                                                    │
   release  A ── B ── X ── Y            release  A ─┴ B ── X ── Y ── F'
                                                                     ^
                                                          naya commit, naya SHA,
                                                          same content
```

## 24.2 Commands

```bash
# Ek commit
git cherry-pick a1b2c3d

# Kai commits
git cherry-pick a1b2c3d d4e5f6a 7g8h9i0

# Range (a1b2c3d EXCLUSIVE, 7g8h9i0 inclusive)
git cherry-pick a1b2c3d..7g8h9i0

# Range (a1b2c3d INCLUSIVE)
git cherry-pick a1b2c3d^..7g8h9i0

# Apply karo par commit mat karo (staging mein chhod do)
git cherry-pick -n a1b2c3d
git cherry-pick --no-commit a1b2c3d

# Original commit ka reference message mein add karo — TRACEABILITY ke liye
git cherry-pick -x a1b2c3d
# Message mein add hoga: "(cherry picked from commit a1b2c3d...)"

# Merge commit cherry-pick karo (parent batana padta hai)
git cherry-pick -m 1 <merge-commit-sha>

# Conflict aane pe
git cherry-pick --continue
git cherry-pick --skip
git cherry-pick --abort
```

**`-x` flag hamesha use karo cross-branch picks ke liye.** Wo message mein original SHA daal deta hai, jisse 6 mahine baad koi trace kar sake ki ye commit kahan se aaya. Ye chhota sa detail interview mein bolne layak hai.

## 24.3 Kab cherry-pick sahi tool hai

| Use case | Kyun sahi |
|---|---|
| **Hotfix ko multiple release branches pe** | Sabse classic use. Fix `main` pe bani, use `release/2.4` aur `release/2.5` pe bhi chahiye. |
| Ek useful commit abandoned branch se | Branch chhod di, par ek commit kaam ka tha |
| Feature branch se ek independent fix nikalna | Feature abhi ready nahi, par usme ek bug fix hai jo abhi chahiye |
| Galat branch pe commit ho gaya | Sahi branch pe cherry-pick, galat se hatao (Section 35.3) |

**Classic hotfix pattern:**

```bash
# Fix main pe banao
git switch main
git switch -c hotfix/tax-rounding
# ... fix karo ...
git commit -m "fix: round tax at order level, not line level"
git switch main
git merge hotfix/tax-rounding
git push

# Ab same fix supported release branches pe le jao
FIX_SHA=$(git rev-parse main)

git switch release/2.4
git cherry-pick -x "$FIX_SHA"
git push origin release/2.4

git switch release/2.5
git cherry-pick -x "$FIX_SHA"
git push origin release/2.5
```

## 24.4 Cherry-pick ke khatre

| Khatra | Detail |
|---|---|
| **Duplicate commits** | Agar baad mein branches merge hue to same change do baar dikhega (content-wise Git aksar handle kar leta hai, par history confusing hoti hai) |
| **Missing dependencies** | Commit F, commit E pe depend karta hai. Sirf F pick kiya to code toot sakta hai — **aur ye compile ho sakta hai par runtime pe toot sakta hai** |
| **Overuse = branching strategy toot chuki hai** | Agar tum roz cherry-pick kar rahe ho to problem branching model mein hai, cherry-pick mein nahi |
| **Traceability lost** | `-x` ke bina pata nahi chalta ye commit kahan se aaya |

## 24.5 Interview answer

> **Interview answer:**
> "Cherry-pick applies the changes introduced by a specific commit onto your current branch, creating a new commit with a new SHA but the same content.
>
> The case where it's clearly the right tool is a hotfix that needs to reach several long-lived branches. You fix it once on main, then cherry-pick that commit onto each supported release branch. Merging main into a release branch would drag in everything else on main, which is exactly what you don't want on a stabilised release. Cherry-pick takes only the change you need.
>
> I always use `-x`, which appends the original commit SHA to the message. Six months later, when someone is trying to work out why the same fix appears on three branches, that line is the difference between a five-second answer and an archaeology exercise.
>
> The main risk is missing dependencies. A commit is a diff against a specific parent, so if the commit you're picking relies on an earlier commit that isn't on the target branch, the pick can apply cleanly and still be broken — it compiles, and it fails at runtime. So I always run the relevant tests on the target branch after a pick rather than trusting a clean application.
>
> The other thing I'd say is that heavy reliance on cherry-picking is usually a symptom rather than a technique. If a team cherry-picks constantly, the branching model is fighting them — normally long-lived divergent branches that should have been shorter-lived. So I'd treat frequent cherry-picking as a signal to look at the branching strategy rather than as a workflow to optimise."

---

# 25. stash

## 25.1 Kya hai

Uncommitted changes ko temporarily "shelf" pe rakh do, working directory clean ho jaaye, baad mein wapas le lo.

**Kab zaroorat padti hai:** Tum feature ke beech mein ho, kuch bhi commit karne layak nahi hai, aur tumhe **turant** branch switch karni hai (production bug aa gaya).

## 25.2 Commands

```bash
# Stash karo (tracked files ke changes)
git stash
git stash push                          # same, modern syntax

# Message ke saath (ZAROOR do — warna 5 stashes ke baad kuch yaad nahi rahega)
git stash push -m "wip: po custom items form validation"

# UNTRACKED files bhi include karo — ye BAHUT important hai
git stash -u
git stash --include-untracked

# Ignored files bhi (rarely needed)
git stash -a
git stash --all

# Sirf specific files
git stash push -m "just conftest" conftest.py tests/e2e/test_po.py

# Interactive — hunk by hunk chuno kya stash karna hai
git stash push -p

# Staged changes ko rakho, sirf unstaged stash karo
git stash --keep-index
```

```bash
# List
git stash list
# stash@{0}: On feature/po-custom: wip: po custom items form validation
# stash@{1}: On main: WIP on main: a1b2c3d test: add bid flow

# Dekho kya hai andar
git stash show                    # summary
git stash show -p                 # poora diff
git stash show -p stash@{1}       # specific stash ka diff

# Wapas laao AUR stash list se hata do
git stash pop
git stash pop stash@{1}

# Wapas laao par stash list mein RAKHO
git stash apply
git stash apply stash@{1}

# Delete
git stash drop stash@{0}
git stash clear                   # SAB delete — savdhani se

# Stash se ek nayi branch banao (agar base badal gaya hai)
git stash branch feature/recovered-work stash@{0}
```

## 25.3 `-u` flag — sabse common galti

**`git stash` bina `-u` ke UNTRACKED files ko chhod deta hai.**

Scenario: tumne ek naya test file `tests/e2e/test_po_custom.py` banaya (abhi `git add` nahi kiya). `git stash` chalaya. Branch switch ki.

**Wo file abhi bhi wahin hai** — kyunki wo untracked thi, stash usse nahi utha. Ab tum doosri branch pe ho aur wahan wo file bhi dikh rahi hai. Confusing, aur kabhi-kabhi wo galti se commit ho jaati hai galat branch pe.

**Isliye habit banao: `git stash -u` hamesha.**

## 25.4 `pop` vs `apply`

| | `pop` | `apply` |
|---|---|---|
| Kya karta hai | Restore + stash delete | Restore, stash rehne do |
| Conflict aaye to | **Stash delete NAHI hota** (Git safe hai) | Stash rehta hai |
| Kab use karo | Normal case | Jab stash ko multiple branches pe apply karna ho, ya risk ho |

**Safe habit:** `apply` use karo, verify karo ki sab theek hai, phir `drop` karo.

```bash
git stash apply
# ... verify, tests chalao ...
git stash drop
```

## 25.5 Stash ke khatre

| Khatra | Detail |
|---|---|
| **Bhool jaana** | Stashes invisible hain — `git status` mein nahi dikhte. Log mahino purane stashes bhool jaate hain. |
| **Message na hona** | `git stash list` mein "WIP on main: a1b2c3d" — kuch pata nahi chalta |
| **`clear` se sab udd jaana** | `git stash clear` irreversible-ish hai (reflog se recover ho sakta hai par mushkil) |
| **Untracked chhoot jaana** | `-u` ke bina |
| **Merge conflict on pop** | Base badal gaya to pop pe conflict aa sakta hai |

**Better alternative aksar: WIP commit.**

```bash
# Stash ke bajaye
git add -A
git commit -m "wip: checkpoint before hotfix" --no-verify

# ... hotfix karo ...

# Wapas aake
git switch feature/po-custom-items
git reset --soft HEAD~1     # commit undo, changes staging mein wapas
```

**WIP commit ke fayde:** visible hai (`git log` mein dikhta hai), branch se attached hai, push ho sakta hai (backup), aur reflog protection zyada strong hai. **Interview mein ye preference bolna maturity dikhata hai.**

## 25.6 Interview answer

> **Interview answer:**
> "Stash shelves uncommitted changes so the working directory goes clean, and you can restore them later. The typical use is being mid-feature when something urgent arrives and needing to switch branches with nothing in a committable state.
>
> Two things I'd flag. First, plain `git stash` doesn't include untracked files — so a new test file you haven't added yet stays in the working directory and follows you onto the other branch, which is confusing and occasionally results in committing it somewhere wrong. I always use `git stash -u`. Second, always give it a message with `-m`, because `git stash list` without one shows 'WIP on main' and a SHA, which tells you nothing once you have three of them.
>
> On `pop` versus `apply`: pop restores and deletes the stash, apply restores and keeps it. I tend to apply, verify the state is what I expected, and then drop — because if the restore goes wrong, pop has already removed my safety net. Git is careful here and won't drop the stash if the pop conflicts, but I'd rather not depend on that.
>
> Honestly, though, for anything beyond a few minutes I prefer a WIP commit over a stash. A commit is visible in `git log`, it's attached to the branch so you can't confuse which work belongs where, it can be pushed as a backup, and when you come back `git reset --soft HEAD~1` puts you exactly where you were. Stashes are invisible — they don't show in `git status` — and I've seen people carry months-old stashes they've completely forgotten about. Stash is for minutes; a WIP commit is for anything longer."

---

# 26. reset vs revert vs checkout

## 26.1 Ye teeno alag cheezein hain — pehle ye clear karo

| Command | Kya move karta hai | History rewrite? | Shared branch pe safe? |
|---|---|---|---|
| `git reset` | **Branch pointer** (aur optionally index/working dir) | **Haan** | **NAHI** |
| `git revert` | Kuch nahi move karta — **naya commit banata hai** jo undo karta hai | **Nahi** | **HAAN** |
| `git checkout` / `restore` | Sirf **working directory** ki files | Nahi | Haan (par local changes kho sakte hain) |

## 26.2 reset — teen modes ka diagram

```
   Starting state:

   A ── B ── C ── D           HEAD -> main -> D
                  ^
              main, HEAD

   Working dir:  D ke files + kuch uncommitted changes
   Staging area: kuch staged changes

   ══════════════════════════════════════════════════════════════════

   git reset --soft B

   A ── B ── C ── D          (C aur D orphan, reflog mein hain)
        ^
    main, HEAD

   ┌──────────────────────────────────────────────────────────┐
   │ HEAD/branch:    B pe move ho gaya       ✅ BADLA         │
   │ Staging area:   C aur D ke changes AB STAGED hain  ✅    │
   │ Working dir:    KUCH NAHI BADLA         ✅ SAFE          │
   └──────────────────────────────────────────────────────────┘
   Use: "commits ko ek commit mein milana hai" / "commit message galat tha"

   ══════════════════════════════════════════════════════════════════

   git reset --mixed B      (ya sirf: git reset B — YE DEFAULT HAI)

   A ── B ── C ── D
        ^
    main, HEAD

   ┌──────────────────────────────────────────────────────────┐
   │ HEAD/branch:    B pe move ho gaya       ✅ BADLA         │
   │ Staging area:   CLEAR ho gaya           ⚠️  RESET        │
   │ Working dir:    KUCH NAHI BADLA         ✅ SAFE          │
   └──────────────────────────────────────────────────────────┘
   Changes working directory mein hain, unstaged. Tum dobara curate kar sakte ho.
   Use: "commits undo karne hain, changes rakhne hain, dobara stage karunga"

   ══════════════════════════════════════════════════════════════════

   git reset --hard B

   A ── B ── C ── D
        ^
    main, HEAD

   ┌──────────────────────────────────────────────────────────┐
   │ HEAD/branch:    B pe move ho gaya       ✅ BADLA         │
   │ Staging area:   CLEAR                   ⚠️  RESET        │
   │ Working dir:    B ke barabar            ☠️  DESTROYED    │
   └──────────────────────────────────────────────────────────┘
   Uncommitted changes PERMANENTLY GAYE. Reflog se bhi wapas nahi aayenge
   (kyunki wo kabhi commit hi nahi hue the).
   Use: "sab kuch fek do, clean state chahiye"
```

## 26.3 reset ki table

| Mode | HEAD/branch | Staging area (index) | Working directory | Uncommitted work |
|---|---|---|---|---|
| `--soft` | Move | **Preserved** (changes staged ho jaate hain) | Untouched | **Safe** |
| `--mixed` (default) | Move | Reset | Untouched | **Safe** (unstaged ho jaata hai) |
| `--hard` | Move | Reset | **Reset** | **☠️ DESTROYED** |

**Sabse important line: `--soft` aur `--mixed` tumhara uncommitted kaam nahi khaate. `--hard` khaata hai, aur wo recoverable nahi hai.**

## 26.4 reset ke practical uses

```bash
# Pichhla commit undo karo, changes staged rakho (message theek karna hai)
git reset --soft HEAD~1
git commit -m "test(po): add custom item flow with computed total assertion"

# Pichhle 3 commits ko ek mein milao
git reset --soft HEAD~3
git commit -m "test(po): complete custom item flow"

# Commit undo karo, changes wapas working dir mein (dobara curate karna hai)
git reset HEAD~1
git add -p                  # ab selectively stage karo
git commit -m "..."

# Unstage karo (modern: git restore --staged)
git reset HEAD conftest.py

# SAB kuch fek do, remote ke barabar ho jao
git fetch origin
git reset --hard origin/main

# Ek specific commit pe wapas jao (sab kuch fek ke)
git reset --hard a1b2c3d
```

**`git reset --hard` se pehle hamesha:**

```bash
git status              # kya uncommitted hai?
git stash -u            # ya backup le lo
```

## 26.5 revert — safe undo

```bash
# Ek commit undo karo (naya commit banake)
git revert a1b2c3d

# Kai commits (newest se oldest ke order mein karo)
git revert d4e5f6a a1b2c3d

# Range
git revert a1b2c3d..7g8h9i0

# Apply karo par commit mat karo (kai reverts ek commit mein milane ke liye)
git revert -n a1b2c3d
git revert -n d4e5f6a
git commit -m "revert: roll back the trade pricing change"

# MERGE COMMIT revert karo — -m se batao kaunsa parent "mainline" hai
git revert -m 1 <merge-commit-sha>
```

**Revert kya karta hai:**

```
   BEFORE:                        AFTER git revert C:

   A ── B ── C ── D               A ── B ── C ── D ── C⁻¹
                  ^                                   ^
                main                                main

   C⁻¹ ek NAYA commit hai jisme C ka bilkul ulta diff hai.
   C history mein ABHI BHI hai. Kuch delete nahi hua.
```

### `-m 1` merge commit ke liye — ye interview mein poochha jaata hai

Merge commit ke **do parents** hote hain. Git ko batana padta hai ki "undo" kis parent ke relative karna hai.

```
              D ── E              <- feature (parent 2)
             /       \
   A ── B ── C ── F ── M          <- main (M = merge commit)
                       ^
                  parent 1 = G (main ka side)
                  parent 2 = E (feature ka side)
```

```bash
git revert -m 1 M     # feature ke changes undo karo, mainline (main) rakho
git revert -m 2 M     # main ke changes undo karo (bahut rare)
```

**`-m 1` ka matlab: "parent 1 ko mainline maano, aur baaki sab undo kar do".** 99% cases mein `-m 1` hi chahiye.

**Ek gotcha jo senior-level hai:** Agar tumne merge commit revert kiya aur baad mein **wahi branch dobara merge** karna chaho, to Git us branch ke commits ko "already merged" samajhta hai aur kuch nahi laata. Tumhe **revert ka revert** karna padta hai:

```bash
git revert <the-revert-commit>     # revert ko revert karo
git merge feature                  # ab dobara merge karo
```

Ye classic problem hai aur ise jaanna interview mein depth dikhata hai.

## 26.6 reset vs revert — kaun kab

```
  ┌─────────────────────────────────────────────────────────────┐
  │  Kya commit PUSH ho chuka hai / shared branch pe hai?       │
  └──────────────────┬──────────────────────┬───────────────────┘
                     │                      │
                    NO                     YES
                     │                      │
                     v                      v
            ┌────────────────┐     ┌────────────────────┐
            │  git reset     │     │  git revert        │
            │  (rewrite ok)  │     │  (naya commit)     │
            └────────────────┘     └────────────────────┘
```

| Situation | Command |
|---|---|
| Local commit galat message ke saath | `git reset --soft HEAD~1` + recommit |
| Local commits ko squash karna | `git reset --soft HEAD~N` + commit |
| Local branch ko remote ke barabar karna | `git reset --hard origin/main` |
| **Pushed commit jo bug laaya** | **`git revert <sha>`** |
| **Production pe bura release** | **`git revert`** — audit trail rehta hai |
| Uncommitted changes fek do | `git restore .` ya `git reset --hard` |

## 26.7 Interview answer

> **Interview answer:**
> "They solve different problems and the distinction that matters is whether history gets rewritten.
>
> `git reset` moves the branch pointer to a different commit, and the mode controls what else moves. `--soft` moves only the pointer and leaves the changes staged, which is how I squash local commits or fix a bad commit message. `--mixed`, the default, also clears the staging area but leaves the working directory alone, so the changes are there unstaged and I can re-curate them. `--hard` resets everything including the working directory — and that's the one that destroys uncommitted work irrecoverably, because those changes were never in a commit for reflog to find. Before any `--hard` I check `git status` or stash first.
>
> `git revert` doesn't move anything. It creates a *new* commit containing the inverse of a previous commit. The original stays in history, so nothing is rewritten and no force-push is needed.
>
> The rule that follows: reset for local, unpushed work; revert for anything already pushed or on a shared branch. Resetting a shared branch and force-pushing removes commits other people have, and their next pull produces confusing divergence or lost work.
>
> On a production rollback I'd always use revert, and not only for safety. The revert commit is an audit record — it shows what was rolled back and when, which matters for incident review. Rewriting history to pretend the bad release never happened is exactly the wrong thing during an incident.
>
> One detail worth knowing: reverting a merge commit needs `-m 1` to tell Git which parent is the mainline. And there's a trap after that — because the merge is still in history, Git considers that branch already merged, so re-merging it later brings in nothing. You have to revert the revert first. That surprises people, and it's a good thing to know before you're doing it under pressure."

---

# 27. reflog — lost commits recover karna

## 27.1 Kya hai

**Reflog** ek local log hai jo record karta hai ki **HEAD aur branch pointers kahan-kahan gaye**. Har baar jab tum commit, checkout, reset, rebase, merge, ya pull karte ho — reflog mein ek entry banti hai.

**Ye Git ka undo button hai.** Aur ye Git ka sabse under-used feature hai.

**Critical insight:** Git **kabhi bhi commits turant delete nahi karta**. Jab tum reset ya rebase karte ho, purane commits sirf **unreferenced** ho jaate hain — koi branch unhe point nahi karta. Par wo object database mein maujood hain, aur reflog unka SHA yaad rakhta hai.

Wo tab tak zinda rehte hain jab tak garbage collection na chale — default **90 din** reachable objects ke liye, **30 din** unreachable ke liye.

## 27.2 Commands

```bash
# HEAD ka reflog
git reflog
git reflog show HEAD

# Specific branch ka reflog
git reflog show main
git reflog show feature/po-custom-items

# Timestamps ke saath (zyada readable)
git reflog --date=iso
git reflog --date=relative

# Time-based references
git show HEAD@{2}                    # 2 moves pehle HEAD kahan tha
git show main@{yesterday}
git show main@{2.days.ago}
git show main@{"2026-08-20 14:00"}
```

Output aisa dikhta hai:

```
a1b2c3d HEAD@{0}: reset: moving to HEAD~3
7g8h9i0 HEAD@{1}: commit: test(po): assert computed total
b1c2d3e HEAD@{2}: commit: test(po): add vendor assignment step
f4e5d6c HEAD@{3}: commit: test(po): add custom item flow skeleton
a1b2c3d HEAD@{4}: checkout: moving from main to feature/po-custom-items
a1b2c3d HEAD@{5}: pull --rebase: Fast-forward
```

Padhne ka tareeka: **`HEAD@{0}` sabse recent hai.** Har line batati hai ki us operation ke **baad** HEAD kahan tha.

## 27.3 Recovery scenarios

### Scenario 1: `git reset --hard` galti se kar diya

```bash
# Oops
git reset --hard HEAD~3
# 3 commits "chale gaye"

# Reflog dekho
git reflog
# a1b2c3d HEAD@{0}: reset: moving to HEAD~3
# 7g8h9i0 HEAD@{1}: commit: test(po): assert computed total     <-- YAHAN jaana hai

# Verify karo ki yahi hai
git show 7g8h9i0

# Wapas jao
git reset --hard 7g8h9i0
# ya
git reset --hard HEAD@{1}
```

### Scenario 2: Branch galti se delete kar di

```bash
git branch -D feature/po-custom-items
# Deleted branch feature/po-custom-items (was 7g8h9i0).
#                                              ^^^^^^^ Git SHA batata hai!

# Recreate
git switch -c feature/po-custom-items 7g8h9i0

# Agar SHA note nahi kiya:
git reflog | grep "po-custom"
# ya sab unreachable commits dekho:
git fsck --lost-found
```

### Scenario 3: Rebase kharab ho gaya

```bash
# Rebase ke pehle wali state dhoondho
git reflog
# c1d2e3f HEAD@{0}: rebase (finish): returning to refs/heads/feature/x
# b2c3d4e HEAD@{1}: rebase (pick): test: third commit
# a3b4c5d HEAD@{2}: rebase (pick): test: second commit
# 9a8b7c6 HEAD@{3}: rebase (start): checkout main
# 5f6e7d8 HEAD@{4}: commit: test: third commit          <-- rebase se PEHLE
#                                                            yahan the

git reset --hard HEAD@{4}
# Ya branch-specific reflog use karo (zyada reliable):
git reflog show feature/x
git reset --hard feature/x@{1}
```

**`ORIG_HEAD` ek shortcut hai** — Git dangerous operations (reset, rebase, merge, pull) se pehle HEAD ko `ORIG_HEAD` mein save karta hai:

```bash
git reset --hard ORIG_HEAD      # aakhri dangerous operation se pehle wali state
```

### Scenario 4: Detached HEAD mein commits kar diye, phir switch kar liya

```bash
git switch --detach a1b2c3d
# ... commits kiye ...
git switch main
# warning: you are leaving 2 commits behind, not connected to any of your branches:
#   7g8h9i0 experiment: try API-based auth fixture

# Recover
git switch -c experiment/api-auth 7g8h9i0
```

### Scenario 5: Stash galti se drop kar diya

Stashes reflog mein nahi hain (wo `refs/stash` ka apna reflog hai), par unreachable objects hain:

```bash
git fsck --unreachable | grep commit
# unreachable commit 7g8h9i0...

git show 7g8h9i0      # verify karo yahi stash hai

git stash apply 7g8h9i0
```

## 27.4 Reflog ki limitations

| Limitation | Detail |
|---|---|
| **Purely LOCAL** | Reflog push/clone nahi hota. Doosre machine pe recover nahi kar sakte. |
| **Expiry** | Reachable: 90 din. Unreachable: 30 din. `gc.reflogExpire` se configurable. |
| **Fresh clone mein khaali** | Naye clone ka reflog empty hota hai |
| **Uncommitted changes recover NAHI hote** | Reflog commits track karta hai. Jo kabhi commit hua hi nahi wo `--hard` se hamesha ke liye gaya. |

**Wo aakhri point sabse important hai aur interview mein bolne layak hai:** reflog `git reset --hard` se **committed** work bachaata hai. **Uncommitted** work nahi. Isliye "commit early, commit often" sirf ek style preference nahi — wo tumhara safety net hai.

## 27.5 Interview answer

> **Interview answer:**
> "Reflog is a local log of everywhere HEAD and your branch pointers have been — every commit, checkout, reset, rebase, merge and pull creates an entry. It's effectively Git's undo history, and it's the answer to almost every 'I destroyed my work' situation.
>
> It works because Git doesn't delete commits when you reset or rebase. It just stops referencing them — no branch points at them any more — but the objects are still in the database, and reflog remembers their SHAs. They survive until garbage collection, which by default is thirty days for unreachable objects.
>
> So if I do a `git reset --hard` I regret, I run `git reflog`, find the entry from before the reset, and `git reset --hard HEAD@{1}`. If I delete a branch, the deletion message itself prints the SHA, and I can recreate it with `git switch -c name <sha>`. If a rebase goes wrong, I'd use the branch-specific reflog — `git reflog show my-branch` — which is cleaner than the HEAD reflog because it isn't cluttered with every intermediate rebase step. And `ORIG_HEAD` is a useful shortcut: Git saves HEAD there before any dangerous operation, so `git reset --hard ORIG_HEAD` undoes the last reset, rebase, merge or pull.
>
> The limitation I'd be clear about is that reflog only recovers *committed* work. It tracks where references pointed, so anything that was never committed — uncommitted changes wiped by `reset --hard` — is genuinely gone. That's the practical reason 'commit early, commit often' matters: it isn't a style preference, it's what makes your work recoverable. A WIP commit you can reset away later costs nothing and puts your work inside Git's safety net.
>
> The other limitation is that reflog is purely local. It isn't pushed and it isn't in a fresh clone, so it can't help a colleague recover something you lost."

---

# 28. log, diff, show, blame, bisect

## 28.1 log

```bash
# Basic
git log

# Compact
git log --oneline

# GRAPH — sabse useful. Branch structure dikhata hai.
git log --oneline --graph --decorate --all

# Alias bana lo (Section 18.4)
git lg

# Last N
git log -5

# File changes ke saath
git log --stat
git log -p                       # poora diff

# Ek FILE ki history
git log --follow -- tests/e2e/test_po_material.py
# --follow = rename ke aar-paar bhi track karo

# Ek FUNCTION ki history (bahut powerful)
git log -L :test_create_po_material:tests/e2e/test_po_material.py

# Author se filter
git log --author="Ritik"

# Date se
git log --since="2026-08-01" --until="2026-08-21"
git log --since="2 weeks ago"

# Commit MESSAGE mein search
git log --grep="tax rounding"
git log --grep="MER-4821"

# CODE CONTENT mein search — "pickaxe". Ye interview mein poochha jaata hai.
git log -S "calculate_tax"       # jab ye string ADD ya REMOVE hui
git log -G "calculate_tax.*rate" # regex se, har change jo match kare

# Do branches ka farq
git log main..feature            # feature mein hai, main mein nahi
git log feature..main            # main mein hai, feature mein nahi
git log main...feature           # dono mein alag-alag (symmetric difference)

# Custom format
git log --pretty=format:"%h %ad %an: %s" --date=short
```

**`git log -S` (pickaxe) QA ke liye khaas useful hai.** Sawaal: *"Ye `wait_for_timeout(5000)` kab aur kis commit mein aaya?"*

```bash
git log -S "wait_for_timeout(5000)" --oneline -- tests/
```

Ye seedha wo commit dikhayega jisne ye line add ki. Interview mein bolna: *"When I'm investigating why a test became flaky, `git log -S` on the suspicious line takes me directly to the commit that introduced it, along with its message and author — which is usually faster than reading the whole file history."*

## 28.2 diff

```bash
# Working directory vs staging area
git diff

# Staging area vs last commit (jo commit hone wala hai)
git diff --staged
git diff --cached                # same thing

# Working directory vs last commit (sab kuch)
git diff HEAD

# Do commits
git diff a1b2c3d 7g8h9i0

# Do branches
git diff main feature
git diff main...feature          # merge base se — PR jaisa diff

# Ek file
git diff main -- conftest.py

# Sirf file names
git diff --name-only main
git diff --name-status main      # A/M/D ke saath

# Stats
git diff --stat main

# Word-level diff (prose ke liye behtar)
git diff --word-diff

# Whitespace ignore karo
git diff -w
git diff --ignore-all-space
```

**`git diff main...feature` (teen dots) vs `git diff main feature` (do dots) — ye interview mein poochha jaata hai:**

| | `main feature` (2 dots) | `main...feature` (3 dots) |
|---|---|---|
| Compare karta hai | main ka tip vs feature ka tip | **merge base** vs feature ka tip |
| Dikhata hai | Dono ke beech ka poora farq | **Sirf feature ne kya kiya** |
| PR review ke liye | Confusing (main ke changes bhi ulte dikhte hain) | **Sahi — GitHub yahi dikhata hai** |

## 28.3 show

```bash
# Ek commit ki details + diff
git show a1b2c3d

# Sirf file names
git show --stat a1b2c3d
git show --name-only a1b2c3d

# Kisi commit pe ek file ka CONTENT (diff nahi)
git show a1b2c3d:conftest.py
git show main:tests/e2e/test_po.py
git show HEAD~3:requirements.txt

# Tag dekho
git show v1.2.0

# Merge commit ka combined diff
git show -m <merge-sha>
```

**`git show <commit>:<file>` bahut useful hai** — bina checkout kiye purana version padho. QA ke liye: *"ye test 3 commits pehle kaisa dikhta tha?"*

```bash
git show HEAD~3:tests/e2e/test_po_material.py > /tmp/old_version.py
diff /tmp/old_version.py tests/e2e/test_po_material.py
```

## 28.4 blame

```bash
# Har line kaunse commit se aayi
git blame conftest.py

# Sirf specific lines
git blame -L 40,60 conftest.py

# Whitespace changes ignore karo (reformatting noise hatao)
git blame -w conftest.py

# Moved/copied code bhi track karo — POWERFUL
git blame -w -C -C -C conftest.py

# Ek commit se pehle ka blame (reformatting ke pehle)
git blame a1b2c3d~1 -- conftest.py

# Bulk reformat commits ignore karo
git blame --ignore-rev <reformat-commit-sha> conftest.py
```

**Bulk-reformat problem aur uska fix:** Agar kisi ne poore repo pe `black` chalaya, to `git blame` har line pe wahi ek commit dikhayega — useless. Fix:

```bash
# .git-blame-ignore-revs file banao
echo "a1b2c3d4e5f6  # style: apply black formatting across the repo" >> .git-blame-ignore-revs
git config blame.ignoreRevsFile .git-blame-ignore-revs
```

GitHub bhi is file ko respect karta hai. Ye ek chhoti si cheez hai jo interview mein bolne pe achhi lagti hai.

**Blame ka sahi use — ye important framing hai:**

> Blame ka naam bura hai. Uska point kisi ko doshi thehraana nahi hai — **context dhoondhna** hai. "Ye ajeeb sa wait kyun hai?" → blame → commit message → ticket → pata chala ki ek specific race condition ke liye tha. Ab tum use safely hata sakte ho ya nahi, ye decide kar sakte ho.

## 28.5 bisect — WORKED EXAMPLE

**Ye QA ka sabse powerful Git tool hai.** Binary search se wo commit dhoondho jisne kuch toda.

### Problem

`test_po_trade_items` pass ho raha tha 2 hafte pehle. Aaj fail ho raha hai. Beech mein **200 commits** hain. Manually check karoge to 200 checkouts.

Bisect **log₂(200) ≈ 8 checkouts** mein answer de dega.

### Step-by-step

```bash
# ─── Step 1: Bisect shuru karo ─────────────────────────────
git bisect start

# ─── Step 2: Current (broken) commit ko BAD mark karo ──────
git bisect bad
# ya specific: git bisect bad HEAD

# ─── Step 3: Ek jaana-maana GOOD commit batao ──────────────
# Wo tag / commit jahan test pass ho raha tha
git bisect good v1.4.0
# ya SHA se: git bisect good a1b2c3d
# ya date se:
#   git bisect good $(git rev-list -1 --before="2026-08-07" main)
```

Git jawab deta hai:

```
Bisecting: 99 revisions left to test after this (roughly 7 steps)
[7g8h9i0abc] feat: add bulk vendor import to trade items
```

Git ne automatically **beech ka commit** checkout kar diya.

```bash
# ─── Step 4: Test karo ─────────────────────────────────────
pytest tests/e2e/test_po_trade.py::test_po_trade_items -q

# ─── Step 5: Result batao ──────────────────────────────────
git bisect good      # agar PASS hua
# ya
git bisect bad       # agar FAIL hua
# ya
git bisect skip      # agar test chal hi nahi saka (build toota tha)
```

Har jawab ke baad Git search space aadha kar deta hai:

```
Bisecting: 49 revisions left to test after this (roughly 6 steps)
[b1c2d3e] refactor: extract line item pricing into a service
```

Repeat karte raho. Aakhir mein:

```
b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0 is the first bad commit
commit b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0
Author: Priya Sharma <priya@merlinai.co>
Date:   Wed Aug 13 11:24:31 2026 +0530

    refactor: extract line item pricing into a service

 src/pricing/LineItemPricingService.kt | 142 ++++++++++++++
 src/po/PurchaseOrderService.kt        |  38 +---
 2 files changed, 148 insertions(+), 32 deletions(-)
```

```bash
# ─── Step 6: Bisect khatam karo, wapas apni branch pe ──────
git bisect reset
```

### Automated bisect — ye asli superpower hai

Manual bisect mein 8 baar test chalana padta hai. **`git bisect run` sab automatic kar deta hai:**

```bash
git bisect start
git bisect bad HEAD
git bisect good v1.4.0

# Ek script do jo exit code 0 (good) ya non-zero (bad) return kare
git bisect run pytest tests/e2e/test_po_trade.py::test_po_trade_items -q

# ... Git khud sab checkouts karke test chalayega ...
# b1c2d3e is the first bad commit

git bisect reset
```

**Ek chai ke time mein 200 commits mein se culprit mil gaya.**

Complex case ke liye wrapper script:

```bash
#!/usr/bin/env bash
# bisect-check.sh
# Exit codes:
#   0   = good
#   1-124 = bad
#   125 = skip (is commit pe test chal hi nahi sakta)

set -e

# Dependencies install karo (har commit pe alag ho sakti hain)
pip install -q -r requirements.txt || exit 125

# Agar app build hi nahi hoti to SKIP karo, bad mat kaho
./gradlew build -x test -q || exit 125

# Asli test
pytest tests/e2e/test_po_trade.py::test_po_trade_items -q
```

```bash
chmod +x bisect-check.sh
git bisect run ./bisect-check.sh
```

**`exit 125` ka matlab "skip" hai** — ye important hai. Agar us commit pe build hi tooti thi to wo "bad" nahi hai (test ka bug wahan tha hi nahi), wo bas untestable hai. `125` batata hai Git ko ki isse chhod do.

### Bisect ke aur commands

```bash
# Ab tak ka bisect log dekho
git bisect log

# Log save karke replay karo (galti ho gayi to)
git bisect log > bisect.log
git bisect reset
git bisect replay bisect.log

# Custom terms (agar "good/bad" fit nahi baithta)
git bisect start --term-old=fast --term-new=slow

# Sirf ek path pe bisect karo (unrelated commits skip)
git bisect start -- src/pricing/
```

### Bisect achhe se kaam kare, iske liye kya chahiye

| Requirement | Kyun |
|---|---|
| **Har commit buildable ho** | Warna bahut saare `skip` — search inefficient ho jaata hai |
| **Atomic commits** | Ek commit = ek change. Agar commit 5 cheezein karta hai to "yahi commit hai" kaafi useful nahi. |
| **Deterministic test** | **Flaky test bisect ko poori tarah tod deta hai** — galat answer milega |
| **Reasonably linear history** | Squash-merge wali history bisect ke liye ideal hai |

**Flaky test wali baat sabse important hai.** Agar test 20% baar randomly fail hota hai, to bisect ka har "bad" verdict 20% chance se jhootha hai, aur binary search galat direction mein chali jaayegi. **Bisect se pehle confirm karo ki failure deterministic hai** — test ko 5 baar chalao current commit pe.

> **[REAL]** Ye tumhare project mein directly applicable hai. Agar `test_po_trade_items` achanak fail hone lage, to bisect exactly wo tool hai. Aur ye bhi bolna: *"This is also why I care about the merge strategy. I squash-merge into main, so main has one meaningful commit per change — which makes bisect give a genuinely actionable answer rather than pointing at a merge commit containing thirty changes."*

## 28.6 Interview answer

> **Interview answer:**
> "These are my investigation tools, and I use them differently.
>
> `git log` with `--oneline --graph --decorate --all` gives me the branch structure at a glance. The variant I use most as a QA is the pickaxe, `git log -S`, which finds commits where a specific string was added or removed. When I'm investigating why a test became flaky, searching for the suspicious line — a hard-coded wait, say — takes me straight to the commit that introduced it, with its message and author, which is much faster than reading the whole file history.
>
> On `diff`, the distinction I'd highlight is two dots versus three. `git diff main feature` compares the two tips, which mixes in changes that happened on main. `git diff main...feature` compares against the merge base, showing only what the feature branch did — that's what GitHub shows in a pull request, and it's what you actually want when reviewing.
>
> `git blame` is for context, not attribution. When I find something odd in a test — an unexplained wait, a strange selector — blame gives me the commit, the message and usually a ticket reference, which tells me whether it was a deliberate workaround for a specific race condition or just leftover. That determines whether I can safely remove it. I always run it with `-w` to ignore whitespace, and I'd set up a `.git-blame-ignore-revs` file so bulk reformatting commits don't mask the real history.
>
> `git bisect` is the one I'd call out as the most valuable for QA. It binary-searches history to find the commit that introduced a regression — two hundred commits becomes about eight checkouts. And it automates: `git bisect run pytest path::test` will do the whole search unattended while you get coffee. In a wrapper script, exit code 125 means 'skip', which matters for commits where the build was broken — those aren't bad, they're untestable, and marking them bad sends the search the wrong way.
>
> The prerequisite I'd stress is that bisect is only as reliable as the test is deterministic. If the test is flaky, every verdict has a chance of being wrong and the binary search confidently converges on an innocent commit. So before bisecting I confirm the failure is deterministic by running it several times at the current commit. And it's a concrete reason I care about commit hygiene: atomic, individually-buildable commits are what make bisect produce an actionable answer instead of pointing at a merge containing thirty unrelated changes."

---

# 29. Tags & Releases

## 29.1 Do tarah ke tags

```bash
# LIGHTWEIGHT — bas ek pointer, koi metadata nahi
git tag v1.2.0

# ANNOTATED — poora git object: tagger, date, message, optionally signed
git tag -a v1.2.0 -m "Release 1.2.0 — trade item pricing rework"

# GPG signed
git tag -s v1.2.0 -m "Release 1.2.0"
```

**Releases ke liye HAMESHA annotated tags use karo.** Kyun: unme kaun, kab, kyun ka record hota hai; `git describe` unhe use karta hai; aur wo verify ho sakte hain.

## 29.2 Commands

```bash
# List
git tag
git tag -l "v1.*"

# Tag ki details
git show v1.2.0

# Purane commit pe tag lagao
git tag -a v1.1.9 a1b2c3d -m "Backfill missing release tag"

# Push — tags DEFAULT MEIN NAHI JAATE
git push origin v1.2.0
git push --tags                 # saare
git push --follow-tags          # sirf annotated tags jo reachable hain (safest)

# Delete
git tag -d v1.2.0                        # local
git push origin --delete v1.2.0          # remote

# Tag pe checkout (detached HEAD)
git switch --detach v1.2.0

# Tag se branch banao (hotfix ke liye)
git switch -c hotfix/1.2.1 v1.2.0

# Nearest tag se current commit describe karo
git describe --tags
# v1.2.0-14-ga1b2c3d
#   ^      ^     ^
#   |      |     └── current commit ka short SHA
#   |      └──────── tag ke baad 14 commits
#   └─────────────── nearest tag
```

**`git describe --tags` build versioning ke liye perfect hai:**

```yaml
      - name: Compute version
        run: |
          VERSION=$(git describe --tags --always --dirty)
          echo "VERSION=$VERSION" >> "$GITHUB_ENV"
          echo "Building version $VERSION"
```

**Yaad rakhne wali baat: `git push` tags nahi bhejta.** Ye ek classic "maine tag banaya par CI trigger nahi hua" ka reason hai.

## 29.3 Semantic Versioning

```
   MAJOR . MINOR . PATCH
     │       │       │
     │       │       └── Backward-compatible bug fixes
     │       └────────── Backward-compatible new features
     └────────────────── BREAKING changes

   v2.4.1  ->  v2.4.2   bug fix
   v2.4.1  ->  v2.5.0   nayi feature, purana code abhi bhi chalega
   v2.4.1  ->  v3.0.0   breaking change, consumers ko update karna padega
```

**QA ka role yahan:** Version number ek **promise** hai. Agar koi `2.4.1` se `2.4.2` pe ja raha hai to wo expect karta hai ki kuch nahi tootega. **Ye testable hai** — backward compatibility tests. Interview mein bolna: *"A patch version bump is a contract that says nothing breaks. That's a testable claim, and API contract tests are how you verify it before shipping."*

## 29.4 Tag-triggered release pipeline

```yaml
name: Release

on:
  push:
    tags:
      - 'v*.*.*'

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0        # changelog generate karne ke liye history chahiye

      - name: Generate changelog since last tag
        id: changelog
        run: |
          PREV=$(git describe --tags --abbrev=0 HEAD^ 2>/dev/null || echo "")
          if [ -n "$PREV" ]; then
            RANGE="${PREV}..HEAD"
          else
            RANGE="HEAD"
          fi
          {
            echo "log<<CHANGELOG_EOF"
            git log "$RANGE" --pretty=format:"- %s (%h)" --no-merges
            echo ""
            echo "CHANGELOG_EOF"
          } >> "$GITHUB_OUTPUT"

      - name: Create GitHub release
        run: |
          gh release create "${{ github.ref_name }}" \
            --title "Release ${{ github.ref_name }}" \
            --notes "${{ steps.changelog.outputs.log }}"
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

# 30. .gitignore & .gitattributes

## 30.1 .gitignore

```gitignore
# Pattern syntax
*.log                # koi bhi .log file, kahin bhi
/build               # SIRF root ka build (leading slash = root-anchored)
build/               # koi bhi build DIRECTORY, kahin bhi
!important.log       # NEGATION — is file ko ignore mat karo
doc/**/*.pdf         # doc ke andar kisi bhi depth pe .pdf
temp?.txt            # temp1.txt, tempA.txt (? = ek character)
[Bb]uild/            # Build/ ya build/
```

**Precedence rules (interview mein poochha jaata hai):**
1. Baad wala pattern pehle wale ko override karta hai
2. Agar parent **directory** ignored hai, to uske andar ki file ko `!` se un-ignore **nahi** kar sakte

```gitignore
# ❌ Ye KAAM NAHI karega
reports/
!reports/keep-this.md         # reports/ directory hi ignored hai, Git usme jaata hi nahi

# ✅ Ye kaam karega
reports/*
!reports/keep-this.md
```

**Multiple .gitignore files:** Har directory ka apna ho sakta hai. Nested wala parent ko override karta hai. Plus global:

```bash
git config --global core.excludesfile ~/.gitignore_global
# Isme apne OS/editor ki cheezein daalo: .DS_Store, .idea/, *.swp
# Repo ke .gitignore mein editor-specific cheezein daalna team ko annoy karta hai
```

**Debug karo ki file kyun ignored hai:**

```bash
git check-ignore -v reports/report.html
# .gitignore:23:reports/    reports/report.html
#      ^file    ^line ^pattern
```

**Already-tracked files ka problem** (Section 14.2 mein bhi tha, itna important hai):

```bash
# .gitignore tracked files pe kaam NAHI karta
git rm --cached path/to/file          # ek file
git rm -r --cached .                  # sab kuch untrack, phir dobara add
git add .
git commit -m "chore: apply gitignore to previously tracked files"
```

## 30.2 .gitattributes

Ye kam jaana jaata hai par interview mein bolne pe achha lagta hai.

```gitattributes
# ─── Line endings ─────────────────────────────────────────
* text=auto                    # Git khud detect kare, repo mein LF store kare
*.sh   text eol=lf             # shell scripts HAMESHA LF
*.bat  text eol=crlf           # windows batch HAMESHA CRLF
*.py   text eol=lf

# ─── Binary files (diff mat karo) ─────────────────────────
*.png  binary
*.jpg  binary
*.pdf  binary
*.zip  binary

# ─── Merge strategies ─────────────────────────────────────
# Lockfiles: merge mat karo, hamesha "ours" lo aur regenerate karo
package-lock.json merge=ours
poetry.lock       merge=ours

# Changelog: dono taraf ke entries rakho
CHANGELOG.md merge=union

# ─── Diff behaviour ───────────────────────────────────────
*.py diff=python               # Python-aware hunk headers (function names dikhte hain)
*.md diff=markdown

# ─── Export ignore (archive/tarball mein na jaayein) ──────
tests/          export-ignore
.github/        export-ignore
.gitattributes  export-ignore

# ─── Git LFS (bade files) ─────────────────────────────────
*.mp4  filter=lfs diff=lfs merge=lfs -text
*.psd  filter=lfs diff=lfs merge=lfs -text

# ─── Linguist (GitHub ki language stats) ──────────────────
tests/fixtures/*  linguist-generated=true
docs/*            linguist-documentation=true
```

**Line endings ka problem QA ke liye kyun matter karta hai:** Agar koi Windows pe kaam kar raha hai aur CRLF commit ho jaate hain, to shell scripts CI (Linux) pe fail hote hain — `bad interpreter: /bin/bash^M`. Ye ek classic "works on my machine" hai jise `.gitattributes` permanently fix kar deta hai.

```gitattributes
* text=auto
*.sh text eol=lf
```

Interview mein bolna: *"A `.gitattributes` with `* text=auto` and `*.sh text eol=lf` prevents an entire class of cross-platform CI failure — a Windows developer committing CRLF line endings, and the shell script then failing on the Linux runner with 'bad interpreter'. It's one line that removes a recurring, confusing failure mode."*

---

# 31. Branching Strategies Comparison

## 31.1 Teen main strategies

### GitFlow

```
main      ───●──────────────────────●────────────────●────  (production, tagged)
              \                    /                /
hotfix         \              ●───/               ●/
                \            /                   /
release          \      ●───●────────────       /
                  \    /              \        /
develop   ──●──●───●──●────●─────●─────●──────●─────────
             \    /         \   /       \    /
feature       ●──●           ●─●         ●──●
```

### GitHub Flow

```
main   ──●──●──●──●──●──●──●──●──●──●──●──●──   (hamesha deployable)
          \  /    \  /       \  /
           ●●      ●●         ●●               (short-lived, PR se merge)
```

### Trunk-Based Development

```
main   ──●─●─●─●─●─●─●─●─●─●─●─●─●─●─●─●─●──   (hamesha green, hamesha releasable)
          \/    \/     \/        \/
          ●     ●      ●         ●             (<2 days, ya seedha commit)

release/1.4  ────────●──●                      (sirf release ke waqt kati)
```

## 31.2 Comparison table

| Dimension | GitFlow | GitHub Flow | Trunk-Based |
|---|---|---|---|
| Long-lived branches | `main`, `develop`, `release/*` | `main` | `main` |
| Feature branch life | Hafte–mahine | Din–hafta | **Ghante–2 din** |
| Merge frequency | Feature complete pe | PR ready pe | **Din mein kai baar** |
| Complexity | **Zyada** (5 branch types, rules) | Kam | **Sabse kam** |
| CI compatibility | Kharab (deferred integration) | Achhi | **Excellent** |
| CD compatibility | Kharab | Achhi | **Excellent** |
| Feature flags zaroori | Kam | Kabhi-kabhi | **Haan, mandatory** |
| Merge conflict pain | **Zyada** | Medium | **Kam** |
| Multiple versions support | **Haan** — iska main fayda | Nahi | Release branches se |
| Team discipline required | Medium | Medium | **Zyada** |
| Code review | PR pe | **PR pe (core practice)** | PR ya pair programming |
| Rollback | Revert / hotfix branch | Revert | Revert + feature flag |
| DORA performance correlation | Lower | Higher | **Highest** |
| Kiske liye | Versioned/installed software, mobile, regulated | **Most web teams** | High-velocity SaaS, mature teams |

## 31.3 Kaunsi CI/CD ke liye best hai

**Trunk-based > GitHub Flow > GitFlow**, aur reason simple hai:

CI ki **definition** hai "har developer roz mainline mein integrate kare". GitFlow ki **definition** hai "feature branches lambi chalti hain aur develop mein baad mein merge hoti hain". Ye do cheezein **directly contradict** karti hain.

GitFlow ke saath tumhare paas ek CI **server** hota hai. CI **practice** nahi hoti.

```
   Feature branch life vs Integration risk

   Risk
    ^
    │                                        ╱
    │                                    ╱
    │                              ╱
    │                        ╱
    │                  ╱
    │            ╱
    │      ╱
    │  ╱
    └──────────────────────────────────────────>  Branch age
      1d   2d   1w        2w        4w
      ↑         ↑                    ↑
   trunk-based  github flow       gitflow
```

## 31.4 QA ke liye har strategy mein kya badalta hai

| Strategy | QA kya alag karta hai |
|---|---|
| **GitFlow** | Release branch pe **dedicated regression window** milta hai — testing ke liye zyada time, par feedback bahut late. Hotfix branches ka apna test cycle. QA ka kaam "release testing" jaisa lagta hai. |
| **GitHub Flow** | PR pe testing (preview environments useful hain), merge pe smoke. QA ka kaam "continuous" ho jaata hai. |
| **Trunk-Based** | **Pre-merge gate hi sab kuch hai.** QA ki main responsibility gate ko fast aur trustworthy rakhna. Plus feature-flag testing (dono states). Production monitoring zyada important. |

## 31.5 Interview answer

> **Interview answer:**
> "There are three that come up. GitFlow uses several long-lived branches — main, develop, plus release and hotfix branches — with feature branches that can live for weeks. GitHub Flow is main plus short-lived feature branches merged through pull requests. Trunk-based development is a single long-lived branch with feature branches measured in hours to a couple of days, or direct commits with pair review.
>
> For CI/CD the ordering is clear: trunk-based, then GitHub Flow, then GitFlow — and the reason is definitional rather than stylistic. Continuous integration means every developer integrates into the mainline at least daily. GitFlow's model is that feature branches live long and merge later. Those two things contradict each other, so a team doing GitFlow has a CI *server* but isn't doing CI. All the integration risk still lands at the end, batched.
>
> The variable that actually predicts pain is branch age. A two-day branch rarely conflicts badly; a three-week branch reliably does, and the failure is harder to attribute because so much changed at once.
>
> GitFlow still earns its place where you genuinely support multiple released versions simultaneously — installed software, mobile apps with app-store review cycles, or regulated environments needing a formal stabilisation branch with sign-off. For a single-version SaaS product, most of it is ceremony.
>
> What changes for me as a QA is where my effort goes. Under GitFlow I get a dedicated regression window on the release branch — more time to test, but very late feedback, and my role looks like release testing. Under trunk-based, the pre-merge gate is essentially the whole quality system, so my responsibility is making that gate fast and trustworthy, plus testing both states of every feature flag, plus caring much more about production monitoring. It shifts from being a phase to being a property of the pipeline.
>
> For a test automation repository specifically I'd always use trunk-based, because test code has no released versions — you always run the latest suite, so there's nothing for long-lived branches to be for."

---

# 32. PR Workflow — QA ka Nazariya

## 32.1 PR lifecycle

```
   1. Branch banao          git switch -c feature/po-custom-items
              │
   2. Kaam karo, commit     git commit -m "test(po): ..."
              │
   3. Push karo             git push -u origin feature/po-custom-items
              │
   4. DRAFT PR kholo        gh pr create --draft
              │             (CI chalta hai, review nahi maanga jaata)
              │
   5. Ready karo            gh pr ready
              │
   6. Review + CI           reviewers + status checks
              │
   7. Feedback address      commits ya amend + force-with-lease
              │
   8. Approve               required approvals
              │
   9. Merge                 squash / merge / rebase
              │
  10. Branch delete         auto-delete on merge
```

## 32.2 Draft PRs — under-used, bolne layak

```bash
# CLI se
gh pr create --draft --title "test(po): custom item flow" --body "WIP — CI validation"

# Ready hone pe
gh pr ready
```

**Kyun useful hai:**
- CI chalti hai, par reviewers ko notification nahi jaati — matlab tum apna kaam CI pe validate kar sakte ho bina kisi ka time liye
- Work-in-progress visible hai — duplicate kaam nahi hota
- Early feedback maanga jaa sakta hai bina "approve karo" wale pressure ke

## 32.3 QA ek PR review karte waqt kya dekhta hai

Ye **bahut important section hai interview ke liye** — "as a QA, what do you look for when reviewing a PR?" ek common senior question hai.

### 1. Testability

- [ ] Kya ye change automated test se verify ho sakta hai? Agar nahi, to kyun?
- [ ] Naye UI elements pe **stable selectors** hain (`data-testid`)? Ya sirf CSS classes hain jo kal badal jaayengi?
- [ ] Kya naye async operations ka koi **observable completion signal** hai? (loading state, event, response) — ya test ko `sleep` karna padega?
- [ ] Feature flag hai? To kya test se toggle ho sakta hai?

### 2. Test coverage of the change

- [ ] **Naye code ke saath test aaya hai?** Nahi to kyun?
- [ ] Tests actual behaviour assert kar rahe hain ya sirf "no exception thrown"?
- [ ] **Edge cases** cover hue — empty, null, zero, negative, max length, unicode?
- [ ] **Error paths** test hue, ya sirf happy path?
- [ ] Kya koi **existing test delete ya skip** kiya gaya hai? Ye red flag hai — reason poochho.

### 3. Risk aur blast radius

- [ ] Kaunse existing flows affect ho sakte hain? (shared code, shared components)
- [ ] **Database migration** hai? To backward-compatible hai? (Section 8.4 — expand-contract)
- [ ] **API contract** badla? Consumers toot jaayenge?
- [ ] **Config change** hai? Har environment mein wo config maujood hai?
- [ ] Third-party integration touch hui? (QuickBooks!)

### 4. Observability

- [ ] Failure hone pe **debug karne layak log** hai? Ya silent failure?
- [ ] Error messages actionable hain? (`"Error"` vs `"Vendor 4821 not found for PO 9931"`)
- [ ] Naya metric/alert chahiye is change ke liye?

### 5. Rollback

- [ ] Agar ye galat nikla to **revert karna safe hai**? Ya migration ne ek-tarfa change kar diya?
- [ ] Feature flag hai to instant rollback possible hai?

### 6. Security

- [ ] **Koi secret, key, password, token?** (Section 14)
- [ ] Naya endpoint hai to authorisation check hai?
- [ ] User input validate/sanitise ho raha hai?
- [ ] Logs mein PII ya credentials to nahi jaa rahe?

### 7. PR hygiene

- [ ] PR **chhota** hai? (400 lines se bada = review quality gir jaati hai — ye research-backed hai)
- [ ] Commits atomic hain? (bisect/revert ke liye)
- [ ] Description mein **kya aur kyun** hai? Ticket link hai?
- [ ] Ek PR = ek concern? Ya 3 alag cheezein ghusi hui hain?

### Review comment ka tone — ye bhi bolne layak hai

```
❌ "Ye galat hai."
✅ "If the vendor list is empty here, does this throw? I couldn't see a guard —
    worth a test for that case if it's reachable."

❌ "Test kahan hai?"
✅ "Is the trade-item rounding path covered anywhere? If it's hard to test at
    this level I'm happy to add an E2E for it instead."

❌ "Isko refactor karo."
✅ "Non-blocking: this function is doing lookup and calculation together, which
    will make it harder to unit test later. Fine to leave for now if you'd rather."
```

**Comments ko categorise karo** — ye ek professional practice hai:
- **[blocking]** — merge se pehle theek hona chahiye
- **[non-blocking]** — suggestion, tum decide karo
- **[question]** — main samajhna chahta hoon, criticism nahi
- **[nit]** — trivial, ignore kar sakte ho

## 32.4 Squash vs Merge vs Rebase merge

| | Squash and merge | Merge commit (`--no-ff`) | Rebase and merge |
|---|---|---|---|
| Main pe kitne commits | **1** | N + 1 (merge commit) | N |
| History shape | Linear | Branched | Linear |
| Branch commits preserved | **Nahi** | Haan | Haan (naye SHAs ke saath) |
| Revert poori feature | **Easy** (1 commit) | **Easy** (`revert -m 1`) | Mushkil (N reverts) |
| `git bisect` quality | **Excellent** | Achhi | Achhi |
| `git blame` detail | **Kam** (sab ek commit pe) | **Poori** | **Poori** |
| "Kab merge hua" record | Nahi | **Haan** | Nahi |
| Kab use karo | **Default for most teams** | Bade features, GitFlow | Clean commit history wali teams |

**Recommendation jo bolni hai:**

> **Interview answer:**
> "I default to squash-merge for most repositories, and the reason is bisect and revert quality. Main ends up with one meaningful commit per change, so `git bisect` points at something actionable rather than a merge containing thirty commits, and reverting a feature is one command.
>
> The cost is real though — you lose the granular history, so `git blame` on any line inside that feature points at the squash commit and tells you nothing about the specific reasoning. On a large feature where the internal commit sequence carries genuine thinking, that's a loss.
>
> So the nuance I'd apply is: squash by default, but for large features require the author to clean their branch history with an interactive rebase and then use `--no-ff`. You get a readable main *and* the preserved detail. Either way I'd enforce it as a repository setting rather than leaving it to each person, because mixed strategies produce a history nobody can reason about.
>
> One thing that pairs with squash-merge: auto-delete the branch on merge. Because a squash commit isn't a merge commit, Git doesn't record that the branch was integrated — so if someone keeps working on that branch and merges again, Git can re-present changes it has effectively already seen. Deleting the branch avoids that entirely."

## 32.5 Branch protection setup

Repo → Settings → Branches → Add rule:

```
[✓] Require a pull request before merging
    [✓] Require approvals: 1
    [✓] Dismiss stale approvals when new commits are pushed
    [✓] Require review from Code Owners

[✓] Require status checks to pass before merging
    [✓] Require branches to be up to date before merging
    Required checks:
      - static
      - secrets-scan
      - unit
      - suite-health

[✓] Require conversation resolution before merging
[✓] Require linear history            (squash/rebase merge enforce karta hai)
[✓] Do not allow bypassing the above settings
[ ] Allow force pushes                <- OFF
[ ] Allow deletions                   <- OFF
```

**`CODEOWNERS` file** — ye QA ke liye khaas useful hai:

```
# .github/CODEOWNERS

# Test automation — QA ki review chahiye
/tests/                 @ritik
/conftest.py            @ritik
/scripts/               @ritik

# CI/CD — QA + DevOps dono
/.github/workflows/     @ritik @devops-team
/Dockerfile             @ritik @devops-team
/docker-compose*.yml    @ritik @devops-team

# Security-sensitive
/src/test/resources/    @ritik @security-team
**/application*.properties  @security-team
```

**Interview mein bolne layak:** *"I'd own CODEOWNERS entries for the test directory, the CI workflows and — importantly — any properties or resource files, so that a change to test configuration automatically requires review. That's a structural control: it means the class of problem I found once, credentials in a test properties file, gets a second pair of eyes automatically rather than depending on someone remembering."*

---

# 33. Git Hooks & pre-commit framework

## 33.1 Hooks kya hain

Scripts jo Git specific events pe automatically chalti hain. `.git/hooks/` mein rehti hain.

**Critical limitation: `.git/hooks/` version control mein NAHI hoti.** Matlab hooks share nahi hote — har developer ko khud install karna padta hai. Isliye `pre-commit` framework use hota hai (Section 33.4).

## 33.2 Useful hooks

| Hook | Kab chalta hai | Fail hone pe | Use |
|---|---|---|---|
| `pre-commit` | `git commit` se pehle, message se pehle | Commit rukta hai | Lint, format, secret scan, **branch check** |
| `prepare-commit-msg` | Message editor khulne se pehle | — | Template, ticket ID auto-insert |
| `commit-msg` | Message likhne ke baad | Commit rukta hai | Message format validate |
| `post-commit` | Commit ke baad | Kuch nahi (notification only) | Notify, log |
| `pre-push` | `git push` se pehle | Push rukta hai | Tests chalao, protected branch check |
| `pre-rebase` | Rebase se pehle | Rebase rukta hai | Published branch rebase rokna |
| `post-checkout` | Branch switch ke baad | — | Dependencies install |
| `post-merge` | Merge ke baad | — | Dependencies install |

## 33.3 Real example — pre-commit hook jo `main` pe commit rokta hai

```bash
#!/usr/bin/env bash
# .git/hooks/pre-commit
# Install: cp this to .git/hooks/pre-commit && chmod +x .git/hooks/pre-commit

set -euo pipefail

# ═══════════════════════════════════════════════════════════════
# 1. PROTECTED BRANCH CHECK — main/develop pe direct commit rok do
# ═══════════════════════════════════════════════════════════════
PROTECTED_BRANCHES="main master develop release"
CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)

for branch in $PROTECTED_BRANCHES; do
  if [ "$CURRENT_BRANCH" = "$branch" ]; then
    cat <<EOF

  ╔══════════════════════════════════════════════════════════════╗
  ║  COMMIT BLOCKED — you are on a protected branch: ${CURRENT_BRANCH}
  ╚══════════════════════════════════════════════════════════════╝

  Direct commits to '${CURRENT_BRANCH}' are not allowed.
  Create a branch and open a pull request instead:

      git switch -c feature/your-change
      git commit ...
      git push -u origin feature/your-change

  If this is genuinely an emergency, bypass with:
      git commit --no-verify
  (…and expect to explain it in the PR.)

EOF
    exit 1
  fi
done

# ═══════════════════════════════════════════════════════════════
# 2. SECRET SCAN — committed credentials rok do
# ═══════════════════════════════════════════════════════════════
if command -v gitleaks >/dev/null 2>&1; then
  echo "→ Scanning staged changes for secrets…"
  if ! gitleaks protect --staged --redact --no-banner; then
    cat <<'EOF'

  ╔══════════════════════════════════════════════════════════════╗
  ║  COMMIT BLOCKED — potential secret detected                  ║
  ╚══════════════════════════════════════════════════════════════╝

  Remove the credential and use an environment variable instead.
  If this is a false positive, add it to .gitleaks.toml allowlist.

  Do NOT bypass this one with --no-verify.

EOF
    exit 1
  fi
else
  echo "⚠  gitleaks not installed — skipping secret scan (brew install gitleaks)"
fi

# ═══════════════════════════════════════════════════════════════
# 3. CONFLICT MARKERS — half-resolved conflicts rok do
# ═══════════════════════════════════════════════════════════════
STAGED=$(git diff --cached --name-only --diff-filter=ACM)
if [ -n "$STAGED" ]; then
  if echo "$STAGED" | xargs grep -lE '^(<<<<<<<|=======|>>>>>>>)' 2>/dev/null; then
    echo ""
    echo "  COMMIT BLOCKED — conflict markers found in the files above."
    echo "  Finish resolving the conflict before committing."
    echo ""
    exit 1
  fi
fi

# ═══════════════════════════════════════════════════════════════
# 4. DEBUG STATEMENTS — breakpoints aur focused tests rok do
# ═══════════════════════════════════════════════════════════════
PY_STAGED=$(echo "$STAGED" | grep '\.py$' || true)
if [ -n "$PY_STAGED" ]; then
  if echo "$PY_STAGED" | xargs grep -nE 'breakpoint\(\)|import pdb|pdb\.set_trace|\.only\(' 2>/dev/null; then
    echo ""
    echo "  COMMIT BLOCKED — debug statements found above."
    echo ""
    exit 1
  fi

  # pytest.mark.skip bina reason ke
  if echo "$PY_STAGED" | xargs grep -nE '@pytest\.mark\.skip\s*$|@pytest\.mark\.skip\(\s*\)' 2>/dev/null; then
    echo ""
    echo "  COMMIT BLOCKED — skipped test without a reason."
    echo "  Use: @pytest.mark.skip(reason='MER-1234: waiting on backend fix')"
    echo ""
    exit 1
  fi
fi

# ═══════════════════════════════════════════════════════════════
# 5. LINT + FORMAT — sirf staged Python files pe
# ═══════════════════════════════════════════════════════════════
if [ -n "$PY_STAGED" ] && command -v ruff >/dev/null 2>&1; then
  echo "→ Linting staged Python files…"
  echo "$PY_STAGED" | xargs ruff check --quiet || {
    echo "  COMMIT BLOCKED — lint errors. Fix, or run: ruff check --fix"
    exit 1
  }
  echo "$PY_STAGED" | xargs ruff format --check --quiet || {
    echo "  COMMIT BLOCKED — formatting. Run: ruff format ."
    exit 1
  }
fi

echo "✓ Pre-commit checks passed"
exit 0
```

Install:

```bash
cp scripts/hooks/pre-commit .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit

# Test karo
git switch main
echo "x" >> README.md
git add README.md
git commit -m "test"       # <-- BLOCKED hona chahiye
```

## 33.4 commit-msg hook

```bash
#!/usr/bin/env bash
# .git/hooks/commit-msg
# $1 = message file ka path

MSG_FILE="$1"
MSG=$(head -n1 "$MSG_FILE")

# Merge/revert commits skip karo
case "$MSG" in
  Merge*|Revert*|fixup!*|squash!*) exit 0 ;;
esac

# Conventional Commits pattern
PATTERN='^(feat|fix|test|ci|docs|style|refactor|perf|chore|build|revert)(\([a-z0-9/-]+\))?!?: .{10,72}$'

if ! echo "$MSG" | grep -qE "$PATTERN"; then
  cat <<EOF

  ╔══════════════════════════════════════════════════════════════╗
  ║  COMMIT BLOCKED — message does not follow Conventional Commits
  ╚══════════════════════════════════════════════════════════════╝

  Your message:
      ${MSG}

  Expected:
      <type>(<scope>): <summary between 10 and 72 chars>

  Types: feat fix test ci docs style refactor perf chore build revert

  Examples:
      test(po): add end-to-end flow for custom line items
      ci: shard the regression suite across six runners
      fix(pricing): round tax at order level, not line level

EOF
  exit 1
fi

exit 0
```

## 33.5 pre-commit framework — production-grade solution

**Problem:** `.git/hooks/` version control mein nahi hai, to team ke saath share nahi hota.

**Solution:** `pre-commit` framework — config repo mein commit hoti hai, hooks automatically install/update hote hain.

### Setup — step by step

```bash
# Step 1: Install
pip install pre-commit

# Step 2: Config file banao (repo root)
```

`.pre-commit-config.yaml`:

```yaml
# ═══════════════════════════════════════════════════════════════
# pre-commit configuration — Merlin QA automation
# Install:  pip install pre-commit && pre-commit install
# Run all:  pre-commit run --all-files
# Update:   pre-commit autoupdate
# ═══════════════════════════════════════════════════════════════

default_stages: [pre-commit]
fail_fast: false          # saare hooks chalao, pehle fail pe ruko mat

repos:
  # ─── Generic hygiene ───────────────────────────────────────
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: check-merge-conflict       # conflict markers
      - id: check-added-large-files    # bade files galti se
        args: ['--maxkb=1000']
      - id: check-yaml                 # workflow YAML valid hai?
      - id: check-toml
      - id: check-json
      - id: end-of-file-fixer
      - id: trailing-whitespace
      - id: mixed-line-ending
        args: ['--fix=lf']
      - id: detect-private-key         # SSH/TLS private keys
      - id: no-commit-to-branch        # <-- PROTECTED BRANCH BLOCK
        args: ['--branch', 'main', '--branch', 'develop', '--branch', 'master']

  # ─── Secrets ───────────────────────────────────────────────
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.4
    hooks:
      - id: gitleaks

  # ─── Python: lint + format ─────────────────────────────────
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.6.9
    hooks:
      - id: ruff
        args: [--fix, --exit-non-zero-on-fix]
      - id: ruff-format

  # ─── Python: types ─────────────────────────────────────────
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.11.2
    hooks:
      - id: mypy
        additional_dependencies: [types-requests, pytest]
        args: [--ignore-missing-imports]

  # ─── Commit message format ─────────────────────────────────
  - repo: https://github.com/compilerla/conventional-pre-commit
    rev: v3.4.0
    hooks:
      - id: conventional-pre-commit
        stages: [commit-msg]
        args: [feat, fix, test, ci, docs, style, refactor, perf, chore, build, revert]

  # ─── Local project-specific hooks ──────────────────────────
  - repo: local
    hooks:
      # Test suite collect hoti hai? (import errors pakadta hai, 2 second mein)
      - id: pytest-collect
        name: pytest collects cleanly
        entry: pytest --collect-only -q
        language: system
        pass_filenames: false
        types: [python]

      # Debug statements
      - id: no-debug-statements
        name: no breakpoints or pdb
        entry: '(breakpoint\(\)|import pdb|pdb\.set_trace)'
        language: pygrep
        types: [python]

      # Skip without reason
      - id: no-bare-skip
        name: pytest skip must have a reason
        entry: '@pytest\.mark\.skip\s*(\(\s*\))?\s*$'
        language: pygrep
        types: [python]
```

```bash
# Step 3: Hooks install karo
pre-commit install                            # pre-commit hook
pre-commit install --hook-type commit-msg     # commit-msg hook bhi

# Step 4: Ek baar poore repo pe chalao (baseline)
pre-commit run --all-files

# Step 5: Commit karo config
git add .pre-commit-config.yaml
git commit -m "chore: add pre-commit hooks for lint, format and secret scanning"
```

Ab har team member ko sirf ye karna hai:

```bash
pip install pre-commit && pre-commit install
```

Aur CI mein bhi same hooks:

```yaml
      - name: Run pre-commit on all files
        uses: pre-commit/action@v3.0.1
```

**Ye pattern important kyun hai:** Same checks local pe aur CI mein chalte hain. Matlab "local pe pass hua par CI mein fail" wali problem is category ke liye khatam. Interview mein ye bolna: *"Running the identical pre-commit configuration locally and in CI means the fast checks can't diverge — a developer never gets surprised by a lint failure in CI that their local hook didn't catch."*

## 33.6 Hooks ki limitations — ye zaroor bolna

| Limitation | Detail |
|---|---|
| **`--no-verify` se bypass** | `git commit --no-verify` saare hooks skip kar deta hai. Hooks **prevention** hain, **enforcement** nahi. |
| **Client-side** | Har developer ko install karna padta hai (framework isse aasaan karta hai, par optional rehta hai) |
| **Fast hona chahiye** | 10 second se zyada = log `--no-verify` use karne lagenge |
| **Server-side hooks alag hain** | `pre-receive` server pe chalta hai aur bypass **nahi** ho sakta — par self-hosted Git chahiye |

**Isliye architecture ye hona chahiye:**

```
  Hooks (local)          = FAST FEEDBACK. Bypass ho sakta hai. Convenience.
       │
  CI gate (server)       = ENFORCEMENT. Bypass nahi ho sakta (branch protection).
       │
  Push protection        = PREVENTION. Server reject karta hai.
```

**Same check dono jagah honi chahiye.** Hook developer ka time bachaata hai; CI gate guarantee deta hai. Interview mein ye distinction bolna maturity dikhata hai.

## 33.7 Interview answer

> **Interview answer:**
> "Git hooks are scripts that run automatically on Git events. The ones I care about are pre-commit for lint, formatting, secret scanning and blocking commits to protected branches; commit-msg for enforcing a message convention; and pre-push for running a fast test subset.
>
> The practical problem is that `.git/hooks` isn't version-controlled, so hooks can't be shared — each person has to install them, and they drift. That's why I use the `pre-commit` framework instead: the configuration is a YAML file committed to the repository, and a new joiner runs two commands to get exactly the same checks everyone else has. It also handles tool installation and versioning, so nobody is running a different version of the linter.
>
> A specific hook I'd always include is `no-commit-to-branch` for main and develop. It's a small thing but it prevents a genuinely common accident — committing to main because you forgot to branch — and it fails with a clear message telling you what to do instead.
>
> The limitation I'd be explicit about is that hooks are convenience, not enforcement. Any developer can run `git commit --no-verify` and skip all of them, and there are legitimate reasons to. So a hook can never be the only place a check lives. My architecture is: the same check runs as a local hook for fast feedback and as a CI gate for enforcement, backed by branch protection so the CI gate can't be bypassed. The hook saves the developer a round-trip; the gate provides the guarantee.
>
> Running the identical pre-commit configuration in CI as well — there's a GitHub action for exactly this — has the useful side effect that the local and CI versions can't diverge, so nobody gets surprised by a lint failure in CI that their hook didn't catch.
>
> And I'd keep hooks under about ten seconds. Anything slower and people start using `--no-verify` habitually, at which point you've lost the hook entirely."

---

# 34. Monorepo vs Polyrepo

## 34.1 Comparison

| Dimension | Monorepo | Polyrepo |
|---|---|---|
| Structure | Sab kuch ek repo mein | Har service/library ka apna repo |
| Atomic cross-service change | **Haan** — ek PR mein API + consumer dono | Nahi — coordinated PRs |
| Dependency versioning | Ek version, sab consistent | Har repo apni versions |
| CI complexity | **Zyada** — change detection zaroori hai | Kam — har repo ka apna simple pipeline |
| CI cost | Zyada agar sab kuch chale | Kam |
| Tooling requirement | **Bazel, Nx, Turborepo, Pants** | Standard tools kaafi |
| Clone size | Bada (sparse checkout se manage) | Chhota |
| Access control | Mushkil (path-based, limited) | **Easy** — repo-level |
| Code discovery / reuse | **Easy** — sab kuch dikhta hai | Mushkil |
| Refactoring across services | **Easy** — ek commit | Painful |
| Team autonomy | Kam | **Zyada** |
| Release independence | Extra kaam chahiye | **Natural** |
| Examples | Google, Meta, Uber, Twitter | Amazon, Netflix (mostly) |

## 34.2 QA ke liye kya matter karta hai

**Monorepo mein:**
- **Test selection critical hai.** Bina change detection ke har PR poori suite chalayegi — jo unaffordable hai. Tools (Nx, Turborepo, Bazel) dependency graph se affected targets nikaalte hain.
- **Integration testing aasaan hai** — saare services ek jagah hain, compose se poora stack khada karo.
- **Cross-service contract breaks compile time pe pakde jaate hain** — ye bada fayda hai.
- Test code aur product code **saath rehte hain**, matlab drift kam.

**Polyrepo mein:**
- **Contract testing zaroori ho jaati hai** (Pact) — kyunki tum consumer aur provider ko ek saath test nahi kar sakte.
- **Version compatibility matrix** manage karna padta hai — kaunsa service version kaunse ke saath kaam karta hai?
- E2E tests kahan rahenge? Ek alag "e2e-tests" repo? Phir wo product changes se **drift** karta hai — ye classic problem hai.
- Har repo ka apna pipeline — simple, par duplication zyada.

> **[REAL]** Tumhare paas effectively polyrepo-ish setup hai — backend Kotlin/Gradle, aur automation suite alag. Interview mein bolne layak: *"Our automation lives separately from the backend, which is the common polyrepo pattern and it has a specific failure mode I watch for: the test suite drifts from the product because a backend change doesn't force a corresponding test change in the same PR. My mitigations are contract-level API tests that fail fast when a response shape changes, and a nightly full run against the deployed environment so drift surfaces within a day rather than at release."*

## 34.3 Interview answer

> **Interview answer:**
> "A monorepo puts all projects in one repository; polyrepo gives each service its own. Both are used successfully at very large scale, so it's a trade-off rather than a correct answer.
>
> Monorepo's main advantage is atomic cross-cutting change — you can modify an API and every consumer in a single commit, which means the repository is never in a state where the pieces are inconsistent. It makes large refactors and code reuse much easier. The cost is that CI becomes a harder problem: you can't run everything on every change, so you need dependency-graph-aware test selection, and that usually means adopting tooling like Bazel, Nx or Turborepo.
>
> Polyrepo gives teams autonomy and independent release cycles, and the pipelines are simpler. The cost is coordination — a breaking API change now spans multiple pull requests across multiple repositories, and there's a window where things are inconsistent.
>
> From a QA point of view the difference I care about most is where the risk moves. In a monorepo, cross-service breakage tends to surface at build time, because everything compiles together. In a polyrepo it surfaces at runtime in an integration environment, which is later and more expensive — so contract testing becomes essential rather than optional.
>
> There's also a specific polyrepo failure mode I watch for, and it's the situation I'm in: when the test suite lives in its own repository, it drifts from the product, because a product change doesn't force a corresponding test change in the same pull request. Nothing structurally prevents the divergence. My mitigations are contract-level API tests that fail quickly when a response shape changes, and a nightly full run against the deployed environment so drift shows up within a day rather than at release time."

---

# 35. Git Senior Scenarios

---

## 35.1 "Feature ke beech mein ho, production bug fix aa gaya — aur tumhari branch mein conflicts hain. Kya karoge?"

### Situation

```
   main       A ── B ── C ── D ── E        <- team ne yahan kaam kiya
                    \
   feature/po        F ── G ── [uncommitted changes]
                                ^
                             tum yahan ho
```

Production down hai. Fix chahiye **abhi**. Tumhare paas half-done kaam hai.

### Exact command sequence

```bash
# ═══════════════════════════════════════════════════════════════
# STEP 1: Apna kaam BACHAO
# ═══════════════════════════════════════════════════════════════

# Pehle dekho kya-kya uncommitted hai
git status

# OPTION A (recommended): WIP commit — visible, safe, pushable
git add -A
git commit -m "wip: checkpoint before production hotfix" --no-verify
git push origin feature/po-custom-items       # backup (optional par recommended)

# OPTION B: Stash — chhote interruption ke liye
git stash push -u -m "wip: po custom items form validation"
#              ^^ -u ZAROORI hai, warna untracked files chhoot jaayengi

# ═══════════════════════════════════════════════════════════════
# STEP 2: HOTFIX BANAO — main ke LATEST se, apni feature branch se NAHI
# ═══════════════════════════════════════════════════════════════

git switch main
git pull origin main                # latest lo — ye step SKIP MAT KARO
git switch -c hotfix/tax-rounding-p0

# ... fix karo ...

git add src/pricing/LineItemPricingService.kt
git commit -m "fix(pricing): round tax at order level, not line level

Trade line items were rounding tax per line, so a PO with many lines
accumulated up to 2 paise of drift against the invoice total. Rounding
now happens once, on the order total.

Refs MER-4901"

# Test karo — hotfix hai, par test to karna hi hai
pytest tests/e2e -m smoke

git push -u origin hotfix/tax-rounding-p0

# PR kholo, expedited review, merge
gh pr create --title "fix(pricing): round tax at order level" \
             --body "P0 — see MER-4901. Smoke suite green." \
             --label "hotfix,p0"

# ═══════════════════════════════════════════════════════════════
# STEP 3: WAPAS apne kaam pe
# ═══════════════════════════════════════════════════════════════

git switch feature/po-custom-items

# OPTION A tha to: WIP commit undo karo, changes wapas working dir mein
git reset --soft HEAD~1

# OPTION B tha to: stash wapas laao
git stash pop
# ya safer:
git stash apply
# ... verify ...
git stash drop

# ═══════════════════════════════════════════════════════════════
# STEP 4: HOTFIX KO APNI BRANCH MEIN LAAO (rebase)
# ═══════════════════════════════════════════════════════════════

# Pehle uncommitted kaam commit karo — rebase dirty tree pe nahi chalega
git add -A
git commit -m "wip: checkpoint"

git fetch origin

# Dekho kya aa raha hai, rebase se PEHLE
git log HEAD..origin/main --oneline
git diff HEAD origin/main --stat        # conflict surface ka andaza

# Rebase
git rebase origin/main

# ═══════════════════════════════════════════════════════════════
# STEP 5: CONFLICTS resolve karo
# ═══════════════════════════════════════════════════════════════

# CONFLICT (content): Merge conflict in src/pricing/LineItemPricingService.kt

git status                                    # kaunsi files
git diff --name-only --diff-filter=U          # sirf conflicted files

# Resolve karo (Section 23 ka poora process)
# zdiff3 style ON hona chahiye — base dikhta hai
git config merge.conflictStyle zdiff3

# ... file edit karo, markers hatao ...

# Verify koi marker bacha nahi
grep -rn '^<<<<<<<\|^=======\|^>>>>>>>' . || echo "clean"

git add src/pricing/LineItemPricingService.kt
git rebase --continue

# Agla conflict aaye to repeat. Har commit pe alag conflict ho sakta hai.

# Agar galat ho jaaye:
git rebase --abort                            # sab cancel, pehle jaisi state

# ═══════════════════════════════════════════════════════════════
# STEP 6: VERIFY aur push
# ═══════════════════════════════════════════════════════════════

# History check karo
git log --oneline --graph -10

# TESTS chalao — resolution ne kuch toda to nahi
pytest tests/unit -q
pytest tests/e2e -m smoke

# WIP commit undo karo (agar tha)
git reset --soft HEAD~1

# Rebase ke baad force-push chahiye (SHAs badal gaye)
git push --force-with-lease
```

### Interview answer

> **Interview answer:**
> "My first priority is not losing the in-progress work, and my second is that the hotfix must branch from the current main, not from my feature branch — otherwise I'd ship my half-finished work along with the fix.
>
> So: save the work first. For anything beyond a few minutes I prefer a WIP commit over a stash — it's visible in the log, it's attached to the branch, and I can push it as a backup. If I do stash, I use `-u` so untracked files come with it; without that, new files stay in the working directory and follow me onto the other branch, which is confusing and occasionally results in committing them somewhere wrong.
>
> Then switch to main, pull to get the actual current state — skipping that pull is a common mistake that produces a hotfix based on stale code — and branch from there. Fix, and importantly still test: 'it's a hotfix' is not a reason to skip smoke, and a bad hotfix during an incident is much worse than a slightly slower one. Push, open the PR with an expedited review, merge.
>
> Then back to the feature branch, restore the work, and rebase onto the updated main so I have the fix and I'm not going to conflict with it later. Rebase needs a clean tree, so I commit the WIP first.
>
> On the conflicts: I fetch and inspect before rebasing — `git log HEAD..origin/main` shows exactly what's incoming and `git diff --stat` shows which files it touches, so I know the conflict surface before I create it. I have `merge.conflictStyle` set to `zdiff3`, which shows the common ancestor inside the conflict block, so I can see what the code looked like before either change rather than guessing at intent. That turns most conflicts from a judgement call into an obvious one.
>
> During a rebase, remember that 'ours' and 'theirs' are inverted relative to intuition — 'ours' is the branch you're replaying onto. I read the content and the commit SHA rather than trusting the labels.
>
> After resolving I always run the tests before continuing, because a syntactically valid resolution can still be semantically wrong — that's the failure mode of conflict resolution, and it's silent. Then force-push with `--force-with-lease`, never plain `--force`, so if a teammate pushed to that branch while I was working, the push is rejected rather than silently destroying their commits.
>
> And if the rebase goes badly, `git rebase --abort` puts everything back exactly as it was. I'd rather abort and think than resolve badly under time pressure during an incident."

---

## 35.2 "Tumne secret commit kar diya. Ab kya?"

> **[REAL]** Ye tumhara actual scenario hai — `src/test/resources/application-test.properties` mein MongoDB Atlas credentials aur QuickBooks client secret. **Ye answer ratta maar lo.** Ordering hi poora answer hai.

### THE ORDER MATTERS — ye sabse important cheez hai

```
   ┌────────────────────────────────────────────────────────────────┐
   │  1. ROTATE / REVOKE          <- SABSE PEHLE. Har haal mein.    │
   │     (ye akela step hai jo risk ko ACTUALLY khatam karta hai)   │
   ├────────────────────────────────────────────────────────────────┤
   │  2. INVESTIGATE               <- kya kisi ne use kiya?          │
   ├────────────────────────────────────────────────────────────────┤
   │  3. NOTIFY                    <- security team, affected log    │
   ├────────────────────────────────────────────────────────────────┤
   │  4. REWRITE HISTORY           <- cleanup (remediation NAHI)     │
   ├────────────────────────────────────────────────────────────────┤
   │  5. PREVENT                   <- scanning gate lagao            │
   └────────────────────────────────────────────────────────────────┘
```

**Log ulta karte hain** — pehle history rewrite, phir rotate. Wo galat hai, kyunki jab tak tum history rewrite kar rahe ho, credential valid hai aur koi bhi jo already clone kar chuka hai wo use kar sakta hai.

### Step 1 — ROTATE (turant)

```
MongoDB Atlas:
  1. Atlas console > Database Access
  2. Purana user DELETE karo (disable nahi — DELETE)
  3. Naya user banao, minimum required role ke saath
  4. Naya connection string secret manager mein daalo
  5. Application config update karo (env var / secret ref se)
  6. Network Access list bhi review karo — 0.0.0.0/0 to nahi hai?

QuickBooks:
  1. Intuit Developer portal > App > Keys & credentials
  2. Client secret REGENERATE karo
  3. Naya secret secret manager mein
  4. OAuth tokens invalidate karo jo purane secret se bane the
```

**"Purana credential ab valid nahi hai" — jab tak ye sach na ho, baaki sab kuch theatre hai.**

### Step 2 — INVESTIGATE

```bash
# Kab commit hua, kisne kiya?
git log --all --oneline -- src/test/resources/application-test.properties
git log -p --all -S "mongodb+srv://" -- src/test/resources/

# Repository ka exposure kitna tha?
#   - Public tha? -> assume FULLY compromised. Bots seconds mein scan karte hain.
#   - Private tha? -> kitne logon ke paas access tha, forks the?
#   - Kabhi public se private hua? -> forks abhi bhi public ho sakte hain
```

Provider logs check karo:
- MongoDB Atlas → Project Access Log, unexpected IPs
- QuickBooks → API usage logs, unexpected app activity
- **Timeframe:** commit ki date se lekar rotation tak

### Step 3 — NOTIFY

- Security/eng lead ko turant
- Team ko, kyunki history rewrite coordinate karni hai
- Agar customer data exposed ho sakta hai to incident process follow karo

### Step 4 — REWRITE HISTORY

**Ye cleanup hai, remediation nahi.** Credential already rotate ho chuka hai, to ab ye hygiene ka kaam hai — taaki naye clones mein secret na jaaye aur audit clean rahe.

#### Option A: git-filter-repo (recommended — modern, tez)

```bash
# Install
pip install git-filter-repo
# ya: brew install git-filter-repo

# ZAROORI: fresh mirror clone pe kaam karo
cd /tmp
git clone --mirror git@github.com:merlinai/backend.git backend-clean
cd backend-clean

# METHOD 1: Poori file history se hata do
git filter-repo --invert-paths --path src/test/resources/application-test.properties

# METHOD 2: File rakho, sirf secret VALUES replace karo (behtar — file useful hai)
cat > /tmp/replacements.txt <<'EOF'
mongodb+srv://admin:R3alP4ssw0rd@cluster0.abc123.mongodb.net==>mongodb+srv://REDACTED
quickbooks.client.secret=Xy9KlmN0pQrStUvWxYz==>quickbooks.client.secret=REDACTED
EOF

git filter-repo --replace-text /tmp/replacements.txt

# Verify — kuch bhi nahi milna chahiye
git log --all -p | grep -i "mongodb+srv://admin" && echo "STILL PRESENT" || echo "clean"
gitleaks detect --source . -v

# Push karo (filter-repo remote hata deta hai, safety ke liye)
git remote add origin git@github.com:merlinai/backend.git
git push --force --all
git push --force --tags
```

#### Option B: BFG Repo-Cleaner (bade repos pe tez)

```bash
# Download: https://rtyley.github.io/bfg-repo-cleaner/
brew install bfg

git clone --mirror git@github.com:merlinai/backend.git
cd backend.git

# File hata do
bfg --delete-files application-test.properties

# Ya values replace karo
cat > /tmp/passwords.txt <<'EOF'
R3alP4ssw0rd
Xy9KlmN0pQrStUvWxYz
EOF
bfg --replace-text /tmp/passwords.txt

# Cleanup + push
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push --force
```

### Step 4b — FORCE-PUSH ke implications (ye zaroor bolna)

| Implication | Detail | Mitigation |
|---|---|---|
| **Saare SHAs badal gaye** | Har commit ka hash naya | Team ko batao, sab re-clone karein |
| **Sabki local clones invalid** | Unka `main` remote se diverge kar gaya | `git fetch origin && git reset --hard origin/main` |
| **Open PRs toot sakte hain** | Base commits exist nahi karte | PRs rebase ya reopen karne padenge |
| **Tags purane SHAs pe** | Broken references | Tags re-create karo |
| **Fork mein secret abhi bhi hai** | Tumhara rewrite forks pe nahi lagta | Fork owners ko contact karo, ya GitHub support se cached views purge karwao |
| **GitHub cached commit views** | Purane commits URL se abhi bhi accessible ho sakte hain | GitHub Support se contact karo |
| **CI caches** | Purane checkouts cached ho sakte hain | Caches invalidate karo |

**Team ko bhejne wala message:**

```
History rewrite on backend repo — action required

We removed committed credentials from git history. All commit SHAs have changed.

Before doing anything else, please run:

    cd /path/to/backend
    git fetch origin
    git checkout main
    git reset --hard origin/main

If you have local work on a branch:
    git fetch origin
    git rebase --onto origin/main <old-base> <your-branch>

Do NOT push from a stale clone — it will reintroduce the old history.
Open PRs will need rebasing; I'll handle the ones on main.

The credentials were rotated first, so there is no live exposure.
```

**Ye aakhri line important hai** — wo batati hai ki tumne sahi order follow kiya.

### Step 5 — PREVENT

```yaml
# CI gate (Section 14.3)
  secrets-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }
      - uses: gitleaks/gitleaks-action@v2
```

```yaml
# Pre-commit (Section 33.5)
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.4
    hooks:
      - id: gitleaks
```

Plus: GitHub push protection enable karo, aur `CODEOWNERS` mein `**/application*.properties` pe security team ki review required karo.

### Interview answer

> **Interview answer:**
> "The ordering is the whole answer, and most people get it backwards.
>
> **Rotate first, immediately.** Deleting the file or rewriting history does nothing about a credential that's currently valid. The moment a secret is in a repository you have to assume it's compromised, because you cannot prove who has cloned, forked, or mirrored it. If the repository was ever public, assume full compromise — there are bots scanning public pushes within seconds. So step one is revoke and reissue: delete the database user and create a new one with minimum privileges, regenerate the OAuth client secret, and invalidate any tokens derived from it.
>
> **Then investigate.** Check the provider's access logs from the commit date to the rotation for use from unexpected addresses, and work out the exposure surface — was the repo public, how many people had access, were there forks.
>
> **Then notify** — the security lead, and the team, because the history rewrite needs coordinating.
>
> **Then rewrite history**, and I'd be clear that this is hygiene, not remediation — the remediation already happened in step one. I'd use git-filter-repo on a fresh mirror clone. There are two approaches: remove the file entirely with `--invert-paths --path`, or keep the file and replace just the secret values using `--replace-text`. I usually prefer the second, because the file itself is often legitimately needed and only the values were wrong. BFG is the alternative and it's faster on very large repositories. Then verify with a gitleaks scan on the rewritten history before pushing.
>
> **The force-push implications need stating**, because this is a coordinated team activity, not something you do quietly. Every commit SHA after the touched commit changes, so everyone's clone diverges and they must reset hard to origin rather than pulling — pulling from a stale clone reintroduces the old history. Open pull requests may need rebasing. Tags pointing at old SHAs break. And critically, the rewrite doesn't touch forks, and GitHub can keep old commits accessible by URL, so for anything serious you also contact the platform support to purge cached views.
>
> **Finally, prevent recurrence** — and this is the part I care about most, because it's what turns a one-off finding into a permanent control. A blocking gitleaks job in CI with full history fetched, the same check as a local pre-commit hook, platform push protection so the rejection happens server-side, and a CODEOWNERS entry so any change to a properties or resources file requires a second reviewer.
>
> I've dealt with the real version of this. On my current project I found MongoDB Atlas connection credentials and a QuickBooks client secret committed in plaintext in the backend's test resources. The justification for it being there was 'it's only test config', which is exactly the assumption that makes it dangerous — Atlas is cloud-hosted so those credentials are internet-reachable, and QuickBooks is a financial integration. Reporting the finding was the easy part; the valuable part was designing the gate so the class of problem can't recur."

---

## 35.3 "Tumne galat branch pe commit kar diya."

### Case A: Commit abhi push nahi hua

```
   main       A ── B ── C ── D          <- D yahan nahi hona chahiye tha
                              ^
                            HEAD, main

   Chahiye:
   main       A ── B ── C
                         \
   feature                D
```

```bash
# ═══ Method 1: Branch banao yahin se, phir main ko peeche karo (safest) ═══

# Step 1: Current state se nayi branch banao (D ab is branch pe hai)
git switch -c feature/po-custom-items

# Step 2: main pe wapas jao aur usse peeche karo
git switch main
git reset --hard HEAD~1        # ya --soft agar changes staging mein chahiye

# Verify
git log --oneline -3 main
git log --oneline -3 feature/po-custom-items


# ═══ Method 2: Cherry-pick (agar feature branch already exist karti hai) ═══

# Commit ka SHA note karo
git log --oneline -1              # D ka SHA: d4e5f6a

# Sahi branch pe le jao
git switch feature/po-custom-items
git cherry-pick d4e5f6a

# main se hatao
git switch main
git reset --hard HEAD~1


# ═══ Method 3: Kai commits galat branch pe ═══

# main pe 3 commits galti se
git switch -c feature/po-custom-items      # teeno bach gaye
git switch main
git reset --hard HEAD~3


# ═══ Method 4: Commits mixed hain (kuch sahi, kuch galat) ═══

git switch -c feature/po-custom-items
git switch main
git reset --hard HEAD~3

# Ab feature branch pe interactive rebase se galat commits hatao
git switch feature/po-custom-items
git rebase -i HEAD~3
# jo commits main pe chahiye the unhe `drop` karo,
# phir unhe main pe cherry-pick karo
```

### Case B: Commit PUSH ho chuka hai

Ab tum reset + force-push nahi kar sakte (shared branch hai).

```bash
# Step 1: Commit ko sahi branch pe le jao
git switch feature/po-custom-items
git cherry-pick d4e5f6a
git push

# Step 2: main pe usse REVERT karo (reset NAHI)
git switch main
git pull
git revert d4e5f6a
git push

# History ab:
#   main:  A ── B ── C ── D ── D⁻¹      (D aur uska revert dono dikhte hain)
#   feature:              D'            (cherry-picked copy)
```

**Ek gotcha:** Agar baad mein feature branch main mein merge hui, to Git dekhega ki D revert ho chuka hai. Depending on Git ki analysis, changes wapas aa bhi sakte hain ya nahi. Safe approach: merge se pehle revert ko revert kar do, ya feature branch pe commit ko rebase karke naya SHA de do (cherry-pick already naya SHA deta hai, to usually theek hai).

### Interview answer

> **Interview answer:**
> "It depends entirely on whether the commit has been pushed, because that determines whether rewriting is safe.
>
> If it's local only, the cleanest sequence is: create the correct branch from where I am, which captures the commit, then switch back and reset the wrong branch. `git switch -c feature/whatever`, then `git switch main`, then `git reset --hard HEAD~1`. I like this order because the commit is preserved on the new branch *before* I remove it from the wrong one — there's never a moment where it exists nowhere. If the feature branch already exists, I'd cherry-pick onto it instead and then reset main.
>
> If it's already been pushed to a shared branch, resetting and force-pushing is not an option, because that rewrites history other people have. So I cherry-pick it onto the correct branch and then `git revert` it on the wrong one. The revert creates a new commit that undoes it, so history stays intact and nobody's clone breaks. Main ends up showing both the mistake and its reversal, which is slightly noisy but honest, and it's an audit record.
>
> Either way, before I do anything I'd note the commit SHA. Once you start resetting, finding it again means going through reflog — which works, but it's easier to just write it down first."

---

## 35.4 "Ek pushed commit ko undo karna hai."

### Decision tree

```
  ┌──────────────────────────────────────────────────────────────┐
  │  Kya branch SHARED hai? (main, develop, ya koi aur uspe hai)  │
  └────────────────┬───────────────────────┬─────────────────────┘
                   │                       │
                  YES                     NO (sirf tumhari PR branch)
                   │                       │
                   v                       v
        ┌────────────────────┐   ┌──────────────────────────┐
        │   git revert       │   │ git reset + push         │
        │   (naya commit)    │   │      --force-with-lease  │
        │   ✅ HAMESHA SAFE  │   │ (history rewrite)        │
        └────────────────────┘   └──────────────────────────┘
```

### Shared branch — revert

```bash
# Ek commit
git switch main
git pull origin main
git revert a1b2c3d
# editor khulega — message theek karo:
#   "revert: roll back trade item pricing change
#
#    This introduced a rounding error on multi-line POs (MER-4901).
#    Reverting while we fix it properly."
git push origin main

# Kai commits — NEWEST se OLDEST ke order mein
git revert d4e5f6a a1b2c3d

# Range (a1b2c3d exclusive)
git revert a1b2c3d..d4e5f6a

# Ek hi commit mein sab revert
git revert -n a1b2c3d
git revert -n d4e5f6a
git commit -m "revert: roll back the trade pricing rework (MER-4901)"

# MERGE commit revert
git revert -m 1 <merge-sha>
#           ^^^^ parent 1 = mainline. Ye 99% cases mein sahi hai.
```

### Apni PR branch — reset + force-with-lease

```bash
# Aakhri commit hata do
git reset --hard HEAD~1
git push --force-with-lease

# Ya commit rakho par changes wapas working dir mein
git reset HEAD~1
# ... theek karo ...
git add -A
git commit -m "..."
git push --force-with-lease
```

### Production emergency — sabse fast rollback

```bash
# Sabse fast git-based rollback: last known good pe revert
git revert --no-edit HEAD
git push origin main
# CI/CD automatically deploy karega

# Ya agar deployment artifact-based hai to git rollback slow hai —
# pipeline se pichhla artifact redeploy karo. Ye seconds mein hota hai.
```

### Interview answer

> **Interview answer:**
> "The rule is: if the branch is shared, use revert; if it's my own branch that nobody else is on, reset and force-push with lease.
>
> On a shared branch, `git revert` creates a new commit containing the inverse of the bad one. Nothing is rewritten, so nobody's clone breaks and no force-push is needed. On my own pull request branch, `git reset --hard HEAD~1` followed by `git push --force-with-lease` is cleaner, because the bad commit never has to appear in main's history at all.
>
> I'd always use `--force-with-lease` rather than plain `--force`. Lease checks that the remote is still where my remote-tracking reference expects before overwriting — so if a colleague pushed to that branch while I was working, the push is rejected instead of silently deleting their commits.
>
> For a merge commit, revert needs `-m 1` to say which parent is the mainline. And there's a follow-on trap worth knowing: because the merge is still in history, Git treats that branch as already merged, so re-merging it later brings in nothing. You have to revert the revert first. That surprises people at exactly the wrong moment.
>
> One thing I'd add about production rollbacks: reverting in Git and waiting for the pipeline is often not the fastest path. If deployment is artifact-based, redeploying the previous known-good artifact takes seconds while a git revert takes a full pipeline cycle. So during an incident I'd redeploy the previous artifact first to stop the bleeding, then do the git revert to keep the repository state honest. Fixing production and fixing the repository are two separate actions, and the order matters when users are affected."

---

## 35.5 "Ek test fail hone laga aur pata nahi kaunse commit ne toda." — bisect worked

### Full worked sequence

```bash
# ═══════════════════════════════════════════════════════════════
# STEP 0: Confirm karo ki failure DETERMINISTIC hai
# Ye step SKIP MAT KARO — flaky test bisect ko poori tarah tod deta hai
# ═══════════════════════════════════════════════════════════════
for i in 1 2 3 4 5; do
  pytest tests/e2e/test_po_trade.py::test_po_trade_items -q || echo "FAIL run $i"
done
# 5/5 fail hona chahiye. Agar 3/5 fail hai to pehle flakiness fix karo.

# ═══════════════════════════════════════════════════════════════
# STEP 1: Ek known-good point dhoondho
# ═══════════════════════════════════════════════════════════════

# Option A: last release tag
git tag --sort=-creatordate | head -5
# v1.4.0

# Option B: date se
git rev-list -1 --before="2026-08-07" main
# a1b2c3d

# Verify karo ki wahan test pass hota hai
git switch --detach v1.4.0
pytest tests/e2e/test_po_trade.py::test_po_trade_items -q
# PASSED  ✅  <- confirm ho gaya
git switch -

# ═══════════════════════════════════════════════════════════════
# STEP 2: Bisect shuru karo
# ═══════════════════════════════════════════════════════════════
git bisect start
git bisect bad HEAD              # aaj toota hua
git bisect good v1.4.0           # yahan theek tha

# Bisecting: 99 revisions left to test after this (roughly 7 steps)
# [7g8h9i0] feat: add bulk vendor import to trade items

# ═══════════════════════════════════════════════════════════════
# STEP 3a: MANUAL bisect
# ═══════════════════════════════════════════════════════════════
pytest tests/e2e/test_po_trade.py::test_po_trade_items -q
# PASSED
git bisect good

# Bisecting: 49 revisions left to test after this (roughly 6 steps)
# [b1c2d3e] refactor: extract line item pricing into a service

pytest tests/e2e/test_po_trade.py::test_po_trade_items -q
# FAILED
git bisect bad

# ... 6 aur baar repeat ...

# ═══════════════════════════════════════════════════════════════
# STEP 3b: AUTOMATED bisect — ye asli tareeka hai
# ═══════════════════════════════════════════════════════════════
git bisect reset
git bisect start
git bisect bad HEAD
git bisect good v1.4.0

git bisect run pytest tests/e2e/test_po_trade.py::test_po_trade_items -q

# Git khud sab checkouts + test runs karega
# ...
# b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0 is the first bad commit
# commit b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0
# Author: Priya Sharma <priya@merlinai.co>
# Date:   Wed Aug 13 11:24:31 2026 +0530
#
#     refactor: extract line item pricing into a service

# ═══════════════════════════════════════════════════════════════
# STEP 4: Reset karo aur culprit ko analyse karo
# ═══════════════════════════════════════════════════════════════
git bisect reset

git show b1c2d3e
git show --stat b1c2d3e
git log -1 --format="%an %ae %ad%n%n%B" b1c2d3e
```

### Complex case ke liye wrapper script

```bash
#!/usr/bin/env bash
# scripts/bisect-check.sh
#
# Exit codes bisect ke liye:
#   0     = GOOD
#   1-124 = BAD
#   125   = SKIP (is commit pe test chal hi nahi sakta)
#   128+  = abort bisect

set -uo pipefail

echo "=== Testing commit $(git rev-parse --short HEAD) ==="

# Dependencies har commit pe alag ho sakti hain
pip install -q -r requirements.txt || {
  echo "Dependency install failed — SKIP"
  exit 125
}

# Agar backend build hi nahi hoti to SKIP karo, BAD mat kaho
if [ -f build.gradle.kts ]; then
  ./gradlew build -x test -q || {
    echo "Build failed — SKIP (not a test regression)"
    exit 125
  }
fi

# App start karo aur ready hone ka intezaar karo
docker compose -f docker-compose.test.yml up -d --wait || exit 125
trap 'docker compose -f docker-compose.test.yml down -v' EXIT

# Asli test — 3 baar chalao taaki residual flakiness se galat verdict na aaye
FAILS=0
for i in 1 2 3; do
  pytest tests/e2e/test_po_trade.py::test_po_trade_items -q || FAILS=$((FAILS+1))
done

if [ "$FAILS" -ge 2 ]; then
  echo "FAILED $FAILS/3 — BAD"
  exit 1
else
  echo "PASSED — GOOD"
  exit 0
fi
```

```bash
chmod +x scripts/bisect-check.sh
git bisect run ./scripts/bisect-check.sh
```

### Interview answer

> **Interview answer:**
> "`git bisect` — it's a binary search over history, so two hundred commits becomes about eight checkouts.
>
> But there's a step before bisecting that I'd insist on: confirm the failure is deterministic. I run the test five times at the current commit. If it fails five out of five, bisect is valid. If it fails three out of five, bisect is worse than useless — every verdict has a chance of being wrong, and the binary search will confidently converge on an innocent commit. So a flaky test has to be stabilised, or at least characterised, before bisecting.
>
> Then I find a known-good reference — usually the last release tag — and verify the test actually passes there rather than assuming it. Then `git bisect start`, `git bisect bad HEAD`, `git bisect good v1.4.0`, and Git checks out the midpoint.
>
> The version I actually use is `git bisect run`, which automates the whole thing: you give it a command whose exit code decides good or bad, and it runs the entire search unattended. For anything non-trivial I wrap it in a script, and the detail that matters there is exit code 125, which means 'skip'. If a commit doesn't build, it isn't bad — the regression isn't observable there — and marking it bad sends the search in the wrong direction. My wrapper returns 125 for install or build failures and only returns a real verdict when the test genuinely ran. I also run the test a few times inside the script and take a majority, which protects against residual flakiness.
>
> Two things make bisect work well, and both are commit hygiene arguments. Every commit needs to be individually buildable, otherwise you get a lot of skips and the search degrades. And commits need to be atomic — if a commit does five things, 'this is the bad commit' is much less useful than if it does one. That's a concrete reason I care about squash-merging into main: one meaningful commit per change makes bisect produce an actionable answer rather than pointing at a merge containing thirty commits.
>
> And once bisect names the commit, that's the start of the investigation, not the end. I'd read the diff and the message — sometimes the 'first bad commit' is a refactor that merely exposed a latent bug rather than introducing one, and the real fix is elsewhere."

---

## 35.6 "Tumhara rebase kharab ho gaya."

### Case A: Rebase abhi CHAL RAHA hai

```bash
# Sabse simple — sab cancel karo
git rebase --abort
# Poori tarah pehle wali state pe wapas. Kuch nahi khoya.
```

### Case B: Rebase COMPLETE ho gaya, par result galat hai

```bash
# ═══════════════════════════════════════════════════════════════
# Method 1: ORIG_HEAD (sabse fast)
# Git dangerous operations se pehle HEAD ko yahan save karta hai
# ═══════════════════════════════════════════════════════════════
git reset --hard ORIG_HEAD

# ═══════════════════════════════════════════════════════════════
# Method 2: Branch-specific reflog (sabse reliable)
# ═══════════════════════════════════════════════════════════════
git reflog show feature/po-custom-items

# feature/po-custom-items@{0}: rebase (finish): refs/heads/feature/... onto 7g8h9i0
# feature/po-custom-items@{1}: commit: test(po): assert computed total   <-- rebase se PEHLE
# feature/po-custom-items@{2}: commit: test(po): add vendor step

# Verify karo pehle
git log --oneline feature/po-custom-items@{1} -5

# Wapas jao
git reset --hard feature/po-custom-items@{1}

# ═══════════════════════════════════════════════════════════════
# Method 3: HEAD reflog (agar branch reflog clear nahi hai)
# ═══════════════════════════════════════════════════════════════
git reflog
# c1d2e3f HEAD@{0}: rebase (finish): returning to refs/heads/feature/x
# b2c3d4e HEAD@{1}: rebase (pick): test: third
# a3b4c5d HEAD@{2}: rebase (pick): test: second
# 9a8b7c6 HEAD@{3}: rebase (start): checkout origin/main
# 5f6e7d8 HEAD@{4}: commit: test: third            <-- rebase se PEHLE ka tip

git reset --hard HEAD@{4}
```

**`rebase (start)` line dhoondho reflog mein** — uske **theek pehle** wali entry rebase se pehle ki state hai. Ye ek practical trick hai jo bolne layak hai.

### Case C: Rebase ke baad force-push bhi kar diya

```bash
# Local reflog abhi bhi hai
git reflog show feature/po-custom-items
git reset --hard feature/po-custom-items@{N}

# Dobara force-push karke sahi state restore karo
git push --force-with-lease

# Agar tumhara local bhi kharab hai par remote pe purani state thi:
git fetch origin
git reset --hard origin/feature/po-custom-items
```

### Case D: Commits bilkul kho gaye lag rahe hain

```bash
# Saare unreachable commits dhoondho
git fsck --lost-found
# dangling commit 7g8h9i0abc...

# Har ek dekho
git show 7g8h9i0

# Sab dangling commits ek saath, message ke saath
git fsck --lost-found 2>/dev/null | grep "dangling commit" | awk '{print $3}' | \
  while read sha; do
    echo "=== $sha ==="
    git log -1 --format="%ad %an: %s" "$sha"
  done

# Mil gaya to branch bana lo
git switch -c recovered-work 7g8h9i0
```

### Rebase problems se bachne ke liye

```bash
# 1. Rebase se PEHLE ek backup branch bana lo — 1 second ka kaam
git branch backup/before-rebase-$(date +%Y%m%d-%H%M)

# 2. rerere on rakho — repeated conflicts automatic resolve
git config --global rerere.enabled true

# 3. zdiff3 conflict style — base dikhta hai
git config --global merge.conflictStyle zdiff3

# 4. Har rebased commit pe tests chalao
git rebase -i origin/main --exec "pytest tests/unit -q"

# 5. Rebase se pehle dekho kya aa raha hai
git log HEAD..origin/main --oneline
git diff HEAD origin/main --stat
```

### Interview answer

> **Interview answer:**
> "It depends on whether the rebase is still in progress.
>
> If it's mid-rebase, `git rebase --abort` returns everything exactly to the pre-rebase state. Nothing is lost. And I'd rather abort early than resolve conflicts badly under pressure — I can always restart with a better approach.
>
> If the rebase completed and the result is wrong, the recovery is reflog. The quickest path is `git reset --hard ORIG_HEAD`, because Git saves HEAD there before any dangerous operation — reset, rebase, merge or pull. If that isn't right, I use the branch-specific reflog, `git reflog show my-branch`, which is cleaner than the HEAD reflog because it isn't cluttered with every intermediate pick. I look for the `rebase (start)` entry; the entry immediately before it is the pre-rebase tip. Then reset hard to that.
>
> This works because rebase doesn't destroy commits — it creates new ones and moves the branch pointer. The originals become unreferenced but stay in the object database for weeks, and reflog remembers their SHAs. If reflog somehow isn't enough, `git fsck --lost-found` lists every dangling commit, and I can inspect them and branch off the right one.
>
> Prevention is cheaper than recovery, though. Before any non-trivial rebase I create a backup branch — `git branch backup/before-rebase` is one second and turns the whole problem into a one-command fix. I keep `rerere` enabled so repeated conflicts on a long-lived branch resolve automatically after the first time. And I use `merge.conflictStyle zdiff3`, which shows the common ancestor in the conflict block, because most bad rebases are actually bad conflict resolutions where someone guessed at intent.
>
> The other habit is `git rebase -i --exec 'pytest tests/unit -q'`, which runs the tests after each replayed commit. It's slower, but it catches a broken intermediate state at the commit that caused it rather than at the end, and it leaves you with history where every commit is green — which is what makes bisect work later."

---
---

# PART C — CLOSING

---

# 36. Red Flags — Ye Jawab Mat Dena

> Ye wo jawab hain jo tumhe **turant mid-level ya junior** dikha denge. Har ek ke saath likha hai ki interviewer kya sunta hai, aur uski jagah kya bolna hai.

## 36.1 CI/CD red flags

| ❌ Ye MAT bolna | Interviewer kya sunta hai | ✅ Iski jagah ye bolo |
|---|---|---|
| "CI/CD DevOps ka kaam hai, main sirf tests likhta hoon" | Ye banda apne scope se bahar nahi sochta. Senior nahi hai. | "The pipeline is where my tests deliver value. I own how they're wired in, how fast the gate is, and whether the output is trustworthy." |
| "CI matlab Jenkins" | Tool ko practice samajh raha hai | "CI is the practice of integrating daily with automated verification. Jenkins or Actions is just the enforcement mechanism." |
| "Hum har commit pe poori regression chalate hain" | Pipeline design nahi aati; team frustrated hogi | "I stage it — fast checks on PR, smoke on merge, full sharded regression nightly. Slow gates get routed around." |
| "Flaky tests ke liye retry laga dete hain" | Signal delete kar raha hai, root cause nahi dhoondhta | "Retries hide real intermittent product bugs. I detect flakiness with repeat runs, triage test-vs-product, and quarantine with a 14-day SLA." |
| "Coverage 80% se upar hona chahiye" (bas itna) | Metric ratta maara hai, samjha nahi | "I gate on *diff* coverage of changed lines plus a ratchet on total, because a total-coverage gate on legacy code just gets disabled." |
| "100% coverage matlab no bugs" | Fundamental misunderstanding | "Coverage measures execution, not assertion quality. It's a negative signal — low coverage tells me something's untested; high coverage tells me very little." |
| "Test hamesha deployment block karna chahiye" | Nuance nahi hai | "Smoke and P0 paths block. Visual, performance and a11y checks report but don't block, because gates with false positives teach people to override." |
| "Secrets .env file mein rakhte hain" (aur wo commit hai) | Security awareness zero | "Never in the repo. Secret store or Vault, with a blocking gitleaks gate and a pre-commit hook. I've found committed credentials in a test properties file and driven the fix." |
| "Pipeline slow hai, aur runners add kar do" | Measure kiye bina optimise kar raha hai | "First I'd measure stage-by-stage. Then caching, then balanced sharding, then fixtures — in that order, because the intuitive answer is usually not the expensive stage." |
| "Docker sirf deployment ke liye hai" | Container ka QA value nahi samjha | "I run the suite in a pinned Playwright image so my laptop and CI are byte-identical — it eliminates font, locale and library differences that cause flakiness." |
| "Maine CI/CD ke baare mein padha hai" | Hands-on nahi hai | "Here's what I've built…" — aur concrete detail do (`if: always()`, sharding, secret gate) |
| "Staging pe test karte hain, production pe nahi" (jab tumhari suite production pe chalti hai) | Jhooth, ya apne setup ka awareness nahi | Honestly explain karo (Section 5.2 stage 16 ka answer) |

## 36.2 Git red flags

| ❌ Ye MAT bolna | Interviewer kya sunta hai | ✅ Iski jagah ye bolo |
|---|---|---|
| "Commits diffs store karte hain" | Fundamental model galat hai | "Commits are full snapshots of the tree with a parent pointer. Diffs are computed between snapshots, not stored." |
| "Rebase merge se better hai" (bina nuance) | Cargo cult | "Rebase gives linear history; merge preserves what actually happened. The golden rule is never rebase a branch anyone else has." |
| "Main hamesha `git push --force` karta hoon" | **Bada red flag** — data loss risk | "`--force-with-lease` always. It rejects the push if someone else pushed, instead of silently destroying their commits." |
| "Conflict aaye to `--ours` le lete hain" | Resolve nahi kar raha, side chun raha hai | "I read both intentions — usually both changes are wanted. I use `zdiff3` conflict style so I can see the common ancestor rather than guessing." |
| "`git reset --hard` se saaf kar dete hain" (routinely) | Uncommitted work kho raha hoga | "`--hard` destroys uncommitted work irrecoverably. I check status or stash first. On shared branches I revert, not reset." |
| "Commits kho gaye to kuch nahi ho sakta" | Reflog nahi jaanta | "Reflog. Git doesn't delete commits on reset or rebase — it just dereferences them. `git reset --hard ORIG_HEAD` or reflog gets them back." |
| "`git pull` bas chala dete hain" | Merge commit noise, control nahi | "I fetch and inspect first — `git log HEAD..origin/main` shows what's incoming. And I set `pull.rebase` or `pull.ff only` so the default doesn't silently create merge commits." |
| "Secret commit ho gaya to file delete kar do" | **Bada red flag** — security samajh nahi | "Rotate first — that's the only step that removes risk. History rewrite is cleanup, not remediation." |
| "Main GitFlow use karta hoon kyunki wo standard hai" | Trade-offs nahi sochta | "GitFlow suits multi-version products. For single-version SaaS with CI/CD, long-lived branches defeat the point of continuous integration." |
| "Ek bade commit mein saara kaam" | Bisect/revert/review sab kharab | "Atomic commits — I use `git add -p` to split unrelated changes. It's what makes bisect and revert actually useful." |
| "Commit message: 'fix', 'update'" | Care nahi karta | Conventional Commits, body mein **why** |
| "Bisect kabhi use nahi kiya" | Powerful tool nahi jaanta | Worked example do (Section 35.5) |

## 36.3 General interview red flags

| ❌ | ✅ |
|---|---|
| Sirf definition bolna | Definition → **apne project ka example** → trade-off |
| "Pata nahi" ke baad chup ho jaana | "I haven't used X directly, but the closest thing I've done is Y, and I'd approach X by…" |
| Har cheez ka absolute jawab | "It depends on…" aur phir **kis pe depend karta hai wo batao** |
| Apni suite ke problems chhupana | Problems openly bolo + **kya kar rahe ho unke liye** |
| Interviewer se sawaal na poochhna | Clarifying questions poochho — ye senior signal hai |
| Bahut lamba, structure ke bina | Structure: "Three things — first… second… third…" |
| Bahut chhota, sirf haan/na | Har answer mein ek concrete example |
| Blame ("dev ne bura code likha") | Ownership ("here's the gate I'd add so it can't reach main") |

## 36.4 Sabse important framing — ye yaad rakho

Tumne khud kaha: *"agar QA ko CI/CD nahi aata to log maan lete hain ki wo random kaam kar raha hai."*

Isliye **interview mein tumhara core message ye hona chahiye:**

> **"I don't just write tests. I build the system that runs them."**

Har CI/CD answer ko is direction mein le jao:
- "I wrote a test" → **"I wired it into a gate that runs on every PR"**
- "I found a bug" → **"I found it, and I added the control that catches that class of bug automatically"**
- "The suite is slow" → **"I measured it, sharded it, and I track duration as a metric"**
- "Tests are flaky" → **"I have a detection job, a triage process and a quarantine SLA"**

Ye difference hai QA aur SDET mein.

---

# 37. Command Cheatsheet

## 37.1 Git — daily

```bash
# ─── Setup ────────────────────────────────────────────────────
git init -b main                          # naya repo
git clone <url>                           # copy
git clone --depth 1 <url>                 # shallow (CI)
git remote -v                             # remotes dekho
git remote add origin <url>
git config --global user.email "you@co.com"

# ─── Status aur inspection ────────────────────────────────────
git status -sb                            # compact status
git log --oneline --graph --decorate --all
git log -5 --stat
git log -S "search_string"                # pickaxe: kab ye string aayi
git log --grep="MER-4821"                 # message mein search
git log main..feature                     # feature mein hai, main mein nahi
git diff                                  # working vs staged
git diff --staged                         # staged vs HEAD
git diff main...feature                   # PR jaisa diff (merge base se)
git show <sha>                            # commit details
git show <sha>:path/to/file               # us commit pe file ka content
git blame -w file.py                      # line-by-line authorship

# ─── Branching ────────────────────────────────────────────────
git branch -vv                            # branches + upstream + ahead/behind
git switch <branch>                       # switch
git switch -c <new-branch>                # banao + switch
git switch -                              # pichhli branch
git branch -d <branch>                    # safe delete
git branch -D <branch>                    # force delete
git push origin --delete <branch>         # remote delete
git branch --merged main                  # merged branches (cleanup ke liye)

# ─── Staging aur committing ───────────────────────────────────
git add -p                                # interactive, hunk by hunk
git add -A                                # sab kuch
git restore <file>                        # working dir changes discard
git restore --staged <file>               # unstage
git commit -m "feat(scope): summary"
git commit --amend --no-edit              # pichhle commit mein add karo
git commit --allow-empty -m "ci: retrigger"

# ─── Syncing ──────────────────────────────────────────────────
git fetch origin --prune                  # laao + stale refs hatao
git log HEAD..origin/main --oneline       # kya incoming hai
git pull --rebase                         # fetch + rebase
git push -u origin <branch>               # pehli baar
git push --force-with-lease               # rewrite ke baad (NEVER plain --force)
git push --follow-tags                    # commits + annotated tags

# ─── Integration ──────────────────────────────────────────────
git merge <branch>
git merge --no-ff <branch>                # merge commit force karo
git rebase origin/main
git rebase -i HEAD~4                      # interactive: squash/reword/drop
git rebase -i main --exec "pytest -q"     # har commit pe test
git rebase --continue | --skip | --abort
git cherry-pick -x <sha>                  # -x = original SHA note karo

# ─── Undo ─────────────────────────────────────────────────────
git reset --soft HEAD~1                   # commit undo, changes staged
git reset HEAD~1                          # commit undo, changes unstaged
git reset --hard HEAD~1                   # ☠️ sab kuch fek do
git reset --hard origin/main              # remote ke barabar ho jao
git revert <sha>                          # ✅ shared branch pe SAFE
git revert -m 1 <merge-sha>               # merge commit revert
git reset --hard ORIG_HEAD                # aakhri dangerous op undo

# ─── Recovery ─────────────────────────────────────────────────
git reflog                                # HEAD kahan-kahan gaya
git reflog show <branch>                  # branch-specific (cleaner)
git reset --hard HEAD@{3}                 # us point pe wapas
git fsck --lost-found                     # dangling commits
git switch -c recovered <sha>             # lost commits bachao

# ─── Stash ────────────────────────────────────────────────────
git stash push -u -m "message"            # -u ZAROORI (untracked)
git stash list
git stash show -p stash@{0}
git stash apply                           # restore, stash rakho
git stash pop                             # restore + drop
git stash branch <name> stash@{0}

# ─── Bisect ───────────────────────────────────────────────────
git bisect start
git bisect bad HEAD
git bisect good v1.4.0
git bisect run pytest path/to/test.py::test_name -q
git bisect reset

# ─── Tags ─────────────────────────────────────────────────────
git tag -a v1.2.0 -m "Release 1.2.0"      # annotated (releases ke liye)
git push origin v1.2.0                    # tags default mein NAHI jaate
git describe --tags --always --dirty      # build versioning

# ─── Conflicts ────────────────────────────────────────────────
git diff --name-only --diff-filter=U      # conflicted files
git checkout --ours <file>                # ek side lo
git checkout --theirs <file>
git add <file>                            # = "resolved"
git rebase --continue                     # ya: git merge --continue
git merge --abort | git rebase --abort

# ─── Config jo ZAROOR set karni chahiye ───────────────────────
git config --global merge.conflictStyle zdiff3     # base dikhao conflicts mein
git config --global rerere.enabled true            # resolutions yaad rakho
git config --global pull.rebase true
git config --global push.autoSetupRemote true
git config --global init.defaultBranch main
```

## 37.2 CI/CD — daily

```bash
# ─── GitHub CLI ───────────────────────────────────────────────
gh workflow list
gh workflow run nightly-regression.yml \
   -f environment=staging -f suite=regression -f shards=6
gh run list --workflow=nightly-regression.yml --limit 10
gh run view <run-id> --log
gh run view <run-id> --log-failed         # sirf failed steps
gh run watch                              # live follow
gh run rerun <run-id> --failed            # sirf failed jobs
gh run download <run-id>                  # artifacts
gh pr create --draft --title "..." --body "..."
gh pr checks                              # PR ke CI status
gh pr ready

# ─── pytest ───────────────────────────────────────────────────
pytest -m smoke -v
pytest --collect-only -q                  # discover, run mat karo
pytest --durations=25                     # slowest tests
pytest -n auto --dist loadfile            # xdist parallel
pytest --splits 6 --group 3 \
       --splitting-algorithm least_duration \
       --durations-path .test_durations    # sharding
pytest --store-durations                  # durations record karo
pytest --junitxml=results.xml \
       --html=report.html --self-contained-html
pytest --tracing retain-on-failure \
       --screenshot only-on-failure \
       --video retain-on-failure
pytest --lf                               # last failed
pytest --sw                               # stepwise (pehle fail pe ruko)
pytest -p no:randomly                     # random ordering off

# ─── Docker ───────────────────────────────────────────────────
docker build -t merlin-e2e:local .
docker run --rm -it --ipc=host \
  -v "$PWD":/work -w /work \
  -e BASE_URL="https://staging-app.merlinai.co" \
  mcr.microsoft.com/playwright/python:v1.47.0-jammy \
  bash -c "pip install -r requirements.txt && pytest -m smoke"

docker compose -f docker-compose.test.yml up \
  --build --abort-on-container-exit --exit-code-from tests
docker compose -f docker-compose.test.yml logs app
docker compose -f docker-compose.test.yml down -v --remove-orphans

docker system prune -af                   # disk saaf karo

# ─── Secret scanning ──────────────────────────────────────────
gitleaks detect --source . -v             # poori history
gitleaks detect --source . --no-git -v    # sirf working dir
gitleaks protect --staged --redact        # pre-commit ke liye
trufflehog git file://. --only-verified   # sirf verified (live) secrets

# ─── pre-commit ───────────────────────────────────────────────
pre-commit install
pre-commit install --hook-type commit-msg
pre-commit run --all-files
pre-commit autoupdate

# ─── History rewriting (secret removal) ───────────────────────
git clone --mirror <url> repo-clean && cd repo-clean
git filter-repo --replace-text /tmp/replacements.txt
git filter-repo --invert-paths --path path/to/secret-file
git push --force --all && git push --force --tags
```

---

# 38. Quick Revision Table

## 38.1 CI/CD concepts

| Concept | Ek line mein |
|---|---|
| **CI** | Roz mainline mein merge + automated build/test. Integration hell ko marta hai by shrinking the integration unit. |
| **Continuous Delivery** | Har commit release-ready artifact banata hai. **Human** production pe bhejta hai. |
| **Continuous Deployment** | Green commit **automatically** production. **Test suite HI release gate hai.** |
| **Trunk-based** | Ek long-lived branch, feature branches <2 din. CI ke liye best. DORA-backed. |
| **GitFlow** | 5 branch types, long-lived features. Multi-version products ke liye. CI ke against kaam karta hai. |
| **Build once, deploy many** | Ek immutable artifact, sirf config badle. **Per-env rebuild QA ko invalid kar deta hai.** |
| **Smoke test** | 5-15 tests, <3 min, "app zinda hai?". Fail = turant rollback, aage kuch mat chalao. |
| **Quality gate** | Metric + threshold + action. **Jise sab ignore karte hain wo koi gate na hone se bura hai.** |
| **Diff coverage gate** | Naye lines pe 80% + total pe ratchet. Total-coverage gate legacy pe disable ho jaata hai. |
| **Break-glass** | Named auth + written reason + auto-log + auto-announce + follow-up ticket. |
| **Blue-green** | Do envs, traffic switch. Rollback **seconds**. DB shared hai — schema dono ke liye compatible chahiye. |
| **Canary** | Chhote % traffic, metrics compare. QA ka kaam: **kya "healthy" hai** define karna. |
| **Rolling** | Instance-by-instance. **Mixed-version window** — backward compatibility mandatory. |
| **Expand-contract** | Breaking schema change ko 3 backward-compatible deploys mein todo. |
| **Feature flag** | Deploy ≠ release. Rollback instant. **Dono states test karo** + default-when-flag-service-down. |
| **Shadow launch** | Traffic mirror, response discard. **Side effects sabse bada khatra.** Response diffing. |
| **Sharding** | Suite ko N runners pe. **Wall-clock = slowest shard**, isliye balance zaroori (`least_duration`). |
| **pytest-xdist** | Ek machine, N processes. `--dist loadfile` UI tests ke liye. Per-worker accounts chahiye. |
| **Retry policy** | Infra failure pe retry, **assertion failure pe nahi**. Retry real intermittent bugs delete kar deta hai. |
| **Quarantine SLA** | Owner + ticket + 14-day due date + 2% cap. Overdue = **delete**. P0 kabhi quarantine nahi. |
| **`if: always()`** | Bina iske artifact upload **failure pe skip** ho jaata hai — jab uski sabse zyada zaroorat hai. |
| **`--exit-code-from tests`** | Bina iske compose **hamesha 0** return karega — silent false green. |
| **`--ipc=host`** | Docker ka 64MB `/dev/shm` Chromium ko crash karta hai. Pattern-less flakiness ka classic cause. |
| **Layer caching** | Dockerfile mein kam-badalne wali cheezein pehle. `COPY requirements.txt` phir `pip install` phir `COPY . .` |
| **Secret rotation** | **Rotate FIRST.** History rewrite cleanup hai, remediation nahi. |
| **Vault dynamic secrets** | Per-request credential, 1-hour lease, auto-deleted. Leak ka blast radius = 1 ghanta. |
| **Pipeline perf order** | Measure → cache → shard → fixtures → selection → shift down pyramid → split by purpose |

## 38.2 Git concepts

| Concept | Ek line mein |
|---|---|
| **Four areas** | Working dir → staging (index) → local repo → remote. Har command inke beech move karta hai. |
| **Commits** | **Snapshots**, diffs nahi. Tree + parent + metadata, sab ka SHA. |
| **Branch** | 41-byte file with a commit SHA. Isliye branching O(1) hai. |
| **fetch vs pull** | fetch = laao, kuch mat badlo. pull = fetch + merge/rebase. **Fetch, inspect, phir integrate.** |
| **merge** | Naya commit, do parents. Kuch rewrite nahi. **Shared branch pe safe.** |
| **rebase** | Commits replay, **naye SHAs**. Linear history. **Golden rule: shared branch kabhi rebase mat karo.** |
| **Rebase mein ours/theirs** | **ULTE hote hain** — "ours" = target branch. Content padho, label pe bharosa mat karo. |
| **zdiff3** | Conflict mein **common ancestor** dikhata hai. Single best conflict config. |
| **`git add` in conflict** | Matlab "resolved", "stage" nahi. |
| **reset --soft** | Pointer move, **changes staged**. Squash/reword ke liye. |
| **reset --mixed** | Pointer move, changes **unstaged**. Re-curate ke liye. |
| **reset --hard** | ☠️ Uncommitted work **permanently gaya**. Reflog bhi nahi bachaata. |
| **revert** | **Naya** commit jo undo karta hai. History intact. **Shared branch pe yahi use karo.** |
| **`revert -m 1`** | Merge commit revert. Trap: baad mein re-merge kuch nahi laayega — revert ko revert karo. |
| **reflog** | HEAD/branch kahan-kahan gaya ka local log. **Committed** work bachaata hai, uncommitted nahi. |
| **ORIG_HEAD** | Git dangerous op se pehle HEAD yahan save karta hai. `git reset --hard ORIG_HEAD` |
| **cherry-pick -x** | Ek commit doosri branch pe. `-x` original SHA note karta hai — traceability. |
| **stash -u** | `-u` ke bina **untracked files chhoot jaati hain**. Zyada der ke liye WIP commit better. |
| **bisect** | Binary search for the breaking commit. **Deterministic failure chahiye** — flaky test bisect ko todta hai. |
| **bisect exit 125** | "Skip" — commit untestable hai (build toota). Bad mark karna search ko galat direction mein bhejta hai. |
| **`git log -S`** | Pickaxe — kab ye string add/remove hui. Flaky test investigation ke liye best. |
| **`diff main...feature`** | 3 dots = merge base se. **PR jaisa diff.** 2 dots confusing hota hai. |
| **`--force-with-lease`** | Remote expected state pe hai to hi overwrite. Plain `--force` teammate ka kaam chup-chaap kha jaata hai. |
| **.gitignore limitation** | **Tracked files pe kaam nahi karta** aur **history affect nahi karta**. `git rm --cached` chahiye. |
| **.gitattributes** | `* text=auto` + `*.sh text eol=lf` — CRLF wali cross-platform CI failure permanently khatam. |
| **Squash merge** | Main pe 1 commit — bisect/revert excellent. Cost: blame detail kho jaati hai. |
| **pre-commit hooks** | Convenience, **enforcement nahi** — `--no-verify` se bypass. Same check CI mein bhi honi chahiye. |
| **Secret committed** | **1. Rotate. 2. Investigate. 3. Notify. 4. Rewrite. 5. Prevent.** Order hi answer hai. |

## 38.3 Numbers jo yaad rakhne hain

| Metric | Target |
|---|---|
| Pre-commit hook | < 10 sec (max 15) |
| PR gate total | < 5 min (max 10) |
| Merge-to-main gate | < 15 min (max 20) |
| Smoke suite | < 3 min, 5-15 tests |
| Nightly full regression | < 45 min (max 90) |
| Unit tests | < 5 min (zyada = I/O ghusa hai) |
| Diff coverage threshold | >= 80% on changed lines |
| Flake rate threshold | < 1% |
| Quarantine cap | 2% of suite |
| Quarantine SLA | 14 days, phir delete |
| PR size for good review | < 400 lines |
| Feature branch life (trunk-based) | < 2 days |
| Suite reliability math | 200 tests × 99.5% each = **37% suite pass rate** |
| Playwright browser download | ~350MB, 60-120s (cached: ~15s) |
| Docker default `/dev/shm` | 64MB (Chromium ke liye kam) |
| Secret rotation | 90 days scheduled; **immediate** on exposure |
| Bisect efficiency | 200 commits → ~8 checkouts (log₂) |

---

## 38.4 Aakhri baat

Interview se ek raat pehle **sirf ye padho:**

1. **Section 2.3** — continuous delivery vs deployment mein QA ka role kyun badalta hai
2. **Section 6.2** — fast gates ko fast rakhna QA ki responsibility kyun hai
3. **Section 8.4** — expand-contract pattern
4. **Section 16.1** — 5000 tests / 2 hours ka structured answer
5. **Section 22.5** — golden rule of rebase
6. **Section 35.2** — rotate FIRST (tumhari real finding)
7. **Section 36.4** — "I don't just write tests. I build the system that runs them."

Aur ek baat: **tumhare paas already achhi cheezein hain jo bahut QA logon ke paas nahi hoti** —
- Ek suite jo deliberately chhoti hai (6 flows, workflow-level, input-variation-level nahi)
- Screenshot-on-failure ek proper pytest hook se
- Slack reporting already wired
- Ek real security finding jo tumne khud pakdi
- Ek real config-drift story (dead `staging-api` host) jismein ek genuine QA lesson hai

Ye sab **already senior-level material hai.** CI/CD gap sirf ye tha ki ye cheezein ek pipeline se trigger nahi hoti thi. Ab tumhare paas wo bhi hai — aur usse zyada important, **uske peechhe ki soch** hai.

Interview mein confidence isi se aayegi: tum kuch **bana** chuke ho, sirf padha nahi hai.
