# Structural Coverage and Experience-Based Heuristics

## Structural (White-Box) Coverage

Specification-based tests can't tell you how much of the *code* they touch.
Structural coverage closes that loop: measure, inspect what's untouched, decide
whether each gap deserves a test or is dead code worth deleting.

- **Statement coverage** — fraction of executable statements run by the suite.
  100% means every statement executed at least once, which is necessary but
  weak: executing a line doesn't prove it correct (a division that only fails
  on zero passes happily on other data), and it can leave decision outcomes
  untested (an `if` without an `else` reaches 100% statements without ever
  taking the false path).
- **Branch coverage** — fraction of control-flow branches taken: both outcomes
  of every `if` and loop condition, every `switch` arm. **100% branch implies
  100% statement**, never the reverse — so when you aim for one number, aim
  for branch. It's still input-blind: path- and data-dependent defects can
  survive it.

The blind spot of all structural techniques: they measure only code that
exists. A missing requirement produces no uncovered lines — **defects of
omission are invisible to coverage**, which is why structure complements, and
never replaces, specification-based design.

Practical loop, with whatever the toolchain offers (in Go,
`go test -coverprofile=c.out ./... && go tool cover -html=c.out`): design the
black-box cases first, measure, then add a case per meaningful uncovered
branch.

## Assertion Quality

Coverage decides *which cases exist* and says nothing about *what each case
checks*. A suite with perfect partitions asserting `err == nil` on every row
satisfies every criterion in this skill and catches nothing.

- **Assert the behavior the coverage item names.** The boundary case asserts
  the result *at* the limit, not that the call returned; the invalid-partition
  case asserts the specific refusal, not that some error came back. If the
  assertion would still pass with that behavior deleted from the code, the case
  is decorative.
- **Side effects are half the assertion.** Final state *and* emissions — rows
  written, events published, notifications sent, money moved — plus, where it
  matters, that nothing else happened. This is what makes an idempotency test
  real (see [event-driven-testing.md](event-driven-testing.md)).
- **Know where the expected value comes from** — the *test oracle*: the
  specification, an independently computed value, a parallel run of the system
  being replaced, or an invariant that must hold. The failure mode is asserting
  the implementation back at itself — an expected value computed by the code
  under test, or a snapshot approved without being read (see
  [migration-infra-and-output-testing.md](migration-infra-and-output-testing.md)).
  Such a test pins current behavior in place, bugs included, and then reports
  that as coverage.

**Mutation testing** measures assertion strength directly, and is the honest
answer to coverage having become a target instead of a metric: mutate the code
— flip a comparison, drop a statement, change a constant — re-run the suite,
and count the mutants it kills. A mutant surviving on a *covered* line is an
assertion that isn't checking anything, which no coverage number can reveal.
It's expensive over a whole codebase: point it at the modules where correctness
matters and treat the survivors as a worklist, not a percentage to maximize.

## Error Guessing

Deliberately hunt where defects historically live. It works as a *sweep* after
systematic design, driven by three knowledge sources: how this system failed
before, what mistakes this team/language tends to make, and how similar
software fails everywhere.

A general checklist by fault type:
- **Input handling** — valid input refused, missing/extra parameters, absurd
  sizes, wrong encodings
- **Output** — right computation, wrong format; truncation; wrong rounding
- **Logic** — missing case, inverted or off-by-one comparison, wrong operator
- **Computation** — wrong operand, integer overflow, float where money needs
  fixed-point
- **Interfaces** — argument order swapped, unit mismatch (cents vs euros,
  seconds vs millis), nil where a value is assumed
- **Data & state** — uninitialized values, stale caches, concurrent mutation,
  timezone/DST edges, leap days
- **Third-party integrations** — timeouts, malformed or partial responses,
  silent schema drift in the provider's payload, ambiguous outcomes (charged
  but errored — was the money moved?), rate limiting and retry storms

The formalized version — building a catalog of likely faults and writing one
test per entry to provoke each — is called a **fault attack**.

## Exploratory Testing

Design, execute, and learn simultaneously: probe the running system, let each
observation shape the next probe. Strongest when specs are thin, time is short,
or you suspect the spec and the software disagree. Give it structure with
**time-boxed sessions**, each guided by a short **charter** ("explore refund
edge cases around currency conversion"), and record what was tried and found so
discoveries can become durable automated tests. Other techniques nest inside
naturally — partition the inputs you're exploring, sketch the state model as
you learn it.

## Checklist-Based Testing

Keep a curated list of questions worth asking of every change in a domain
("does the endpoint enforce idempotency?", "is the amount validated before the
currency conversion?"). Good checklist hygiene:
- Items are separately and directly checkable — not vague qualities.
- Anything a machine can check belongs in CI, not on the list.
- Feed the list from defect analysis: new painful bug → new entry; entries the
  team has internalized → retire them. A checklist that only grows stops being
  read.

Checklists trade repeatability for breadth: two runs won't be identical, but a
maintained list encodes hard-won team knowledge that no generic technique
produces.

---
*Further reading: Whittaker, "How to Break Software" (fault attacks); Kaner,
Bach & Pettichord, "Lessons Learned in Software Testing" (exploratory testing);
DeMillo, Lipton & Sayward, "Hints on Test Data Selection" and Jia & Harman's
mutation-testing survey (mutation testing); ISO/IEC/IEEE 29119-4 (structural
coverage); the ISTQB CTFL syllabus
(istqb.org).*
