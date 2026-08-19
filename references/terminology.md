# Testing Terminology, Principles, Levels, and Static Testing

Standard software-testing vocabulary as used across the industry (and by
certification schemes such as ISTQB's). Use these meanings when answering
"what is X?" questions and keep them consistent in test documentation.

## Core Chain of Terms

- **Error (mistake)** — a human action producing a wrong result
- **Defect (fault, bug)** — the flaw the error leaves in a work product
- **Failure** — the observable wrong behavior when a defect is executed (some
  defects never fail; some fail only under specific conditions)
- **Root cause** — the underlying reason the error happened; fixing it prevents
  the *next* defect, not just this one

Derivation chain in test design: **test basis** (what tests are derived from) →
**test conditions** (testable aspects, "what to test") → **coverage items**
(units of coverage a technique defines) → **test cases** (preconditions,
inputs, expected results) → procedures/suites. **Testware** = everything these
activities produce. **Traceability** across this chain is what makes coverage
measurable and impact analysis possible.

Other frequent terms: **anomaly** (deviation from expectation — maybe a defect,
maybe not), **test charter** (goals guiding an exploratory session), **test
oracle** (the source of expected results), **verification** (built per spec)
vs **validation** (fit for actual need), **quality control** (product-oriented,
testing is one form) vs **quality assurance** (process-oriented, preventive).

## Classic Testing Principles

Long-established observations, worth citing by origin:

1. Testing shows the **presence** of defects, never their absence (Dijkstra).
2. **Exhaustive testing is impossible** for non-trivial systems — hence
   techniques, risk, and prioritization to spend the budget well.
3. **Defects are cheaper the earlier they're found** (Boehm's cost-of-change
   curve) — test early, both statically and dynamically.
4. **Defects cluster**: a few modules hold most of the bugs (Pareto); known
   clusters are prime risk-analysis input.
5. The **pesticide paradox** (Beizer): a suite run unchanged stops finding new
   bugs; vary and extend it (deliberate regression suites are the exception).
6. **Testing is context-dependent** — an embedded brake controller and a
   marketing site deserve different approaches.
7. **Zero known defects ≠ a good product**: verifying every requirement can't
   rescue software that solves the wrong problem — validate too.

## Test Levels and Test Types

**Levels** (who tests what, against which basis):
- **Component/unit** — one unit in isolation, usually by its developers
- **Component integration** — interactions between units; shaped by the
  integration strategy
- **System** — the whole system's functional and non-functional behavior
  against its specification
- **System integration** — the system talking to other systems and third
  parties, in a production-like environment
- **Acceptance** — validation by/for the intended users (user, operational,
  contractual/regulatory, alpha/beta)

**Types** (what quality is being asked about) — applicable at every level:
- **Functional** — does it do the right thing (completeness, correctness)
- **Non-functional** — how well: performance, reliability, security, usability,
  compatibility, maintainability, portability, safety (the ISO 25010
  characteristics)
- **Black-box** (specification-based) vs **white-box** (structure-based)

**After changes**: **confirmation testing** proves the specific fix works
(re-run what failed); **regression testing** proves the change broke nothing
else — automate it, and scope it by impact analysis. **Maintenance testing**
covers changes to live systems: fixes and enhancements, platform migrations and
data conversions, and retirement (archival/restore).

## Static Testing

Evaluating work products **without executing them** — code, requirements,
designs, test cases, contracts — via human **reviews** and tool-based **static
analysis**. It finds defect kinds dynamic testing can't (ambiguous or
contradictory requirements, unreachable code, standard violations, some
vulnerability patterns) and finds them earlier and cheaper. Static testing
locates defects directly; dynamic testing produces failures you then diagnose.

A structured review runs: plan → kick off → individual review → discuss and
classify findings → fix and report. Distinct roles keep it honest: author,
moderator/facilitator, scribe, reviewers, a manager who allocates the time.

Review formality is a dial:
- **Informal review** — a colleague reads it; no process, no record
- **Walkthrough** — the **author** presents and drives
- **Technical review** — qualified reviewers, moderated, aiming at consensus on
  a technical decision
- **Inspection** — fully process-driven with metrics; the author never
  moderates or scribes; maximizes anomalies found

Reviews succeed on: clear goals and exit criteria, small chunks, prepared
reviewers, findings about the *product* never the author, and management that
actually budgets time for them.

## Tooling and Automation

Tools span the whole process: test management and traceability, static
analysis, test design/data generation, execution frameworks and coverage
measurement, non-functional harnesses (load, security), CI/CD infrastructure,
and environment tooling (containers, virtualization).

Automation pays in repeatability, regression speed, objective coverage numbers,
and freeing humans for the thinking work — but it fails predictably too:
underestimated maintenance, automating what a human should explore, blind trust
in green pipelines, tool lock-in, and platform mismatch. Automate the
repetitive; keep human judgment in the loop for the rest.

---
*Further reading: ISO/IEC/IEEE 29119-1 (concepts and vocabulary), ISO 25010
(quality model), IEEE 1028 (reviews); the ISTQB glossary
(glossary.istqb.org) is a searchable industry reference for these terms.*
