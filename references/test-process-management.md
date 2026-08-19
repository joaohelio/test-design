# Planning, Prioritizing, and Reporting Test Work

## Test Plans

A test plan is the thinking, written down: what's in scope, what "done" means,
and how the effort fits the constraints. Whether it's a document or a section
of an RFC, the load-bearing parts are:

- Scope, objectives, and what the tests are derived from (the test basis)
- Assumptions, constraints, and known risks (product and project)
- The approach: which levels (unit → integration → system → acceptance), which
  types (functional, performance, security, …), which design techniques, what
  coverage is targeted, what data and environments are needed
- Entry/exit criteria, schedule, and who owns what

### Entry and Exit Criteria

- **Entry criteria** — what must be true to start without thrashing:
  environment up, test data available, the build passes smoke tests, the
  stories are actually testable. Agile teams call the story-level version
  **Definition of Ready**.
- **Exit criteria** — what must be true to call the activity finished:
  target coverage reached, no open blockers above severity X, planned cases
  executed, regression suite green. The Agile counterpart is **Definition of
  Done**. Running out of time or budget is also a legitimate exit — *if* the
  remaining risk is stated and stakeholders accept it explicitly.

### Estimating Test Effort

Four standard approaches, combinable:
- **Historical ratios** — derive test effort from development effort using your
  own past projects' ratio.
- **Extrapolation** — measure the first iterations, project the rest; the
  natural fit for iterative work.
- **Estimation poker / Wideband Delphi** — experts estimate independently,
  discuss outliers, converge over rounds.
- **Three-point (PERT)** — optimistic `a`, most likely `m`, pessimistic `b`:
  estimate `(a + 4m + b) / 6` with spread `(b − a) / 6`. E.g. a=4, m=8, b=24
  gives 10 with a spread of ±3.3 — the width is the message.

### Prioritizing Test Cases

- **Risk-based** — cases covering the scariest failures run first (the usual
  default).
- **Coverage-based** — order by coverage contribution; the greedy variant picks
  whichever case adds the most *new* coverage next.
- **Requirements-based** — order by stakeholder priority of the requirement
  each case traces to.

Dependencies trump priority (prerequisites run first), and environment/tool
availability constrains everything.

### Test Pyramid and Quadrants

- **Pyramid** (Cohn): many small, fast, isolated tests at the bottom; few
  slow, broad end-to-end tests at the top. Its message is about *cost*:
  push each check to the lowest layer that can express it.
- **Quadrants** (Marick; popularized by Crispin & Gregory): classify tests on
  two axes — technology- vs business-facing, and guiding development vs
  critiquing the product. Unit/component tests (tech, guiding), story/API
  tests against acceptance criteria (business, guiding), exploratory and UAT
  (business, critiquing), performance/security/smoke (tech, critiquing). Use it
  to notice which quadrant a plan silently ignores.

## Risk-Based Testing

Risk = likelihood × impact. Distinguish:
- **Project risks** — threaten the plan (staffing, suppliers, scope creep,
  estimates).
- **Product risks** — threaten users (wrong calculations, data loss, security
  holes, unacceptable latency); consequences run from annoyance to legal
  liability.

Product risk analysis (identify → assess → prioritize) should drive test
depth: the riskier the area, the more rigorous the technique and the higher
the coverage target — three-point boundaries and all-transitions coverage for
the money path, a happy-path check for the settings page. Mitigations beyond
more testing: reviews, static analysis, more experienced owners, higher test
independence. Then monitor whether mitigations actually moved the risk.

## Monitoring and Reporting

Track what supports decisions: cases run/passed/failed, coverage achieved,
defect counts and severities, residual risk. Two report shapes:
- **Progress reports** (recurring, informal): period, plan-vs-actual, blockers
  and workarounds, new risks, what's next.
- **Completion report** (once per milestone): summary, evaluation against the
  exit criteria, deviations, unresolved defects and unmitigated risks, lessons
  learned.

Match formality to audience — a chat message for the team, a structured report
for a release decision.

## Configuration Management for Testing

Every test result should be reproducible: version the testware together with
the code it tests, identify test environments and data unambiguously, and make
baselines revertible. In practice this means tests live in the repo, pipelines
pin their dependencies, and "which version failed?" always has an answer.

## Defect Reports

A defect report has one job: let someone else reproduce, understand, and
prioritize the problem without a conversation. That takes:

- A one-line summary that names the actual anomaly
- Where and when: test object, version, environment, date
- **Reproduction steps** with the data used, plus logs/screenshots/dumps
- **Expected vs actual** behavior, stated separately
- Severity (impact) and priority (urgency) — related, not the same thing
- Lifecycle status (open → in fix → awaiting confirmation → closed /
  rejected / duplicate / deferred) and links to the test case or requirement

Anomalies aren't always defects — some turn out to be test bugs, environment
noise, or disguised change requests; triage decides, and the report should
survive that scrutiny.

---
*Further reading: Kaner, Bach & Pettichord, "Lessons Learned in Software
Testing"; Crispin & Gregory, "Agile Testing"; ISO/IEC/IEEE 29119-3 (test
documentation templates); the ISTQB CTFL syllabus (istqb.org).*
