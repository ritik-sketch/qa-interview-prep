# QA Engineering Notes

Working notes on test automation, Python, and the engineering fundamentals underneath both.

I write these while learning rather than after — so they are structured the way a thing
actually becomes clear, not the way a finished tutorial presents it. Wherever there is code,
every line carries an explanation of *why it is written that way*, because that is the part
that transfers and the part most material skips.

---

## Where to start

**New to a topic?** Start with [QA Fundamentals](qa-fundamentals.md), then
[Python](python.md). Those two carry most of the vocabulary the rest assumes.

**Preparing for interviews?** [DSA Patterns](dsa-patterns.md) is organised around
recognising which pattern a problem belongs to, rather than around memorising solutions —
which is the thing that actually survives an unseen problem.

**Debugging something right now?** [Debugging & RCA](debugging-rca.md) is the shortest
document here and the one I reread most.

---

## The notes

### Foundations
- **[QA Fundamentals](qa-fundamentals.md)** — test design techniques, defect lifecycle,
  severity versus priority, risk-based testing, and how test strategy is actually decided.
- **[Debugging & Root Cause Analysis](debugging-rca.md)** — finding a bug's cause rather
  than its symptom. Bisection, the silent-fallback pattern, and writing a report a developer
  will actually read.

### Programming
- **[Python — Zero to Advanced](python.md)** — from how the interpreter runs your file
  through to decorators, generators, context managers, concurrency and the memory model.
  Examples are drawn from test automation rather than invented.
- **[DSA Patterns & Problems](dsa-patterns.md)** — brute force first, then the insight,
  then the optimal solution, each with a dry run. Includes the automation-flavoured questions
  that appear in QA interviews instead of textbook algorithms.
- **[Algorithms Deep Dive](algorithms.md)** — every sorting algorithm derived and traced,
  plus searching, recursion, graphs, dynamic programming, greedy and string algorithms.
  Nothing asserted without showing why it holds.

### Testing craft
- **[API Testing](api-testing.md)** — HTTP internals, REST versus GraphQL, status codes,
  authentication and the ways JWTs get forged, schema validation, and contract testing.
- **[Database & SQL](database-sql.md)** — joins, indexes and query plans, transactions and
  isolation levels, MongoDB's aggregation pipeline, and test-data strategy at scale.
- **[Playwright with Python](playwright-python.md)** — auto-waiting and actionability,
  Locator versus ElementHandle, strict mode, contexts and storage state, network
  interception, tracing.

### Engineering
- **[CI/CD & Git](cicd-git.md)** — pipeline anatomy, quality gates, deployment strategies,
  sharding and flaky-test quarantine; then Git from the object model up to bisect and reflog.
- **[OOP, Architecture & System Design](oop-architecture.md)** — OOP through SOLID and
  design patterns, framework architecture, then designing test platforms at scale.
- **[AI & GenAI Testing](ai-genai-testing.md)** — how LLMs work, testing non-deterministic
  output, RAG pipelines and retrieval metrics, prompt injection, and agent loops.

---

## A note on how these are written

Explanations are in Hinglish — Hindi and English mixed, in roman script — because that is
how the idea lands for me first. Anything written to be *spoken* is in English.

If you find an error, I would genuinely like to know.
