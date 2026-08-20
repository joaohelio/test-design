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

Practical loop in Go: design the black-box cases first, then
`go test -coverprofile=c.out ./... && go tool cover -html=c.out`, then add a
case per meaningful uncovered branch.

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
ISO/IEC/IEEE 29119-4 (structural coverage); the ISTQB CTFL syllabus
(istqb.org).*
