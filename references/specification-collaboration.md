# Testable Specifications: Stories, Acceptance Criteria, Test-First

Detection techniques find defects that already exist; collaborative,
test-first practices prevent a class of them from being written at all. The
common thread: make the specification precise enough to test *before*
implementation starts, and let that precision surface ambiguities while they're
cheap.

## User Stories Worth Testing

A story is a promise of a conversation, not a spec. The classic framing
(Jeffries) is **Card** (the token), **Conversation** (how it will actually be
used), **Confirmation** (the acceptance criteria that settle "done"). The
common format — *as a ⟨role⟩ I want ⟨capability⟩ so that ⟨value⟩* — matters
less than who writes it: business, development, and testing perspectives
together, because each sees different holes.

Wake's **INVEST** checklist marks a healthy story: Independent, Negotiable,
Valuable, Estimable, Small, **Testable**. The testability check is the
tester's lever — "how would we test this?" is the fastest way to expose a
story that's vague, valueless, or too big.

## Acceptance Criteria

Acceptance criteria are the story's test conditions: the observable behaviors
that must hold for stakeholders to accept it. Writing them forces scope
decisions, records consensus, and — critically — should cover **negative
scenarios**, not just the happy path.

Two dominant formats, freely mixable:
- **Scenario form** — Given/When/Then, one scenario per behavior (the BDD
  style; automatable with cucumber-like frameworks).
- **Rule form** — a verification checklist or an input→outcome table (which
  maps directly onto a table-driven test).

Any format works if a stranger could implement tests from it without asking
questions.

## Test-First Approaches

- **TDD** — the developer's inner loop: write a failing test, make it pass,
  refactor. Tests drive the design of units.
- **ATDD** — before implementing a story, the team turns its acceptance
  criteria into concrete test cases in a specification workshop. Ambiguities
  die in the workshop instead of in code review. Ordering that works: positive
  cases first, then negative, then non-functional concerns. Each test should
  trace to a criterion, cover nothing beyond the story, and no two tests should
  assert the same thing. Automated, they become executable requirements.
- **BDD** — expresses the behavior in Given/When/Then language shared with
  stakeholders and wires it to automation.

All three implement the same principle: testing moved to the earliest possible
moment (**shift left**) finds the cheapest version of every defect. The
input-domain, decision-table, and state techniques apply unchanged when
deriving the concrete cases.

---
*Further reading: Adzic, "Specification by Example"; Beck, "Test-Driven
Development: By Example"; Wake's INVEST criteria; the ISTQB CTFL syllabus
(istqb.org) covers collaboration-based approaches at foundation level.*
