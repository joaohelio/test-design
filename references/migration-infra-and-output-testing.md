# Testing Migrations, Infrastructure-as-Code, and Rendered Output

Three artifact classes where the logic-oriented techniques (partitions,
decision tables, state transitions, event sequences) are the wrong primary
tool. Forcing them produces theater — a decision table over a Terraform apply
tests nothing. Each class has its own established discipline; the logic
techniques still *nest inside* where noted. When a task falls purely in one of
these classes, say which discipline applies instead of bending a technique to
fit.

## Data migrations and backfills

The test object is the **relationship between the old data and the new**, not
a function. Four correctness questions, each its own coverage item:

1. **Parity** — after migration, does the target agree with the source?
   Layered checks, cheapest first: row/entity counts per table and per
   partition (by date, by tenant); aggregate checksums over key columns;
   field-level diffs on a risk-weighted sample. A count match with a checksum
   mismatch localizes the defect to content, not volume.
2. **Re-run idempotency** — run the migration twice; the second run must
   converge (no duplicated rows, no re-applied transformations). Backfills get
   interrupted and re-run in practice, so a single-run-only migration is
   untested for its real execution mode.
3. **Cutover correctness** — writes arriving *during* the migration: are they
   captured, double-applied, or lost? Test the chosen strategy (dual-write,
   change-data-capture catch-up, write freeze) at its boundaries — the last
   record before the cut and the first after.
4. **Rollback** — reverse the migration on a copy and verify the source state
   is recovered. A rollback script that has never run is a hope, not a plan.

Where classic techniques nest inside: **equivalence partitioning over data
shapes** picks the migration's test records — nulls and empty strings, legacy
encodings, orphaned foreign keys, extremal lengths, rows touched by previous
migrations. A migration verified only on well-formed rows is verified on the
partition least likely to break it.

## Infrastructure-as-code

There is no function to call; the testable artifacts are the plan, the
policies, and the recovery path:

- **Plan diff as the test artifact** — review `terraform plan` (or
  equivalent) output against the *intended* change set: every expected
  resource action present, and — the half people skip — **no unexpected
  destroy/replace**. A replace on a stateful resource (database, volume) in a
  routine diff is a found defect.
- **Policy-as-code assertions** — machine-checkable invariants (encryption
  on, no public ingress, mandatory tags, deletion protection) enforced in CI
  with OPA/conftest-style tools. Anything a reviewer must "remember to check"
  on every diff belongs here, not in review.
- **Restore drills as executable tests** — a backup that has never been
  restored is untested. Periodically restore into a scratch environment and
  assert data integrity and time-to-recover; the drill, not the backup job's
  green light, is the test.
- **Drift detection** — scheduled plan against live state; any diff means
  reality and code disagree, and either one may be the defect.

## Rendered output (documents, PDFs, layout)

Layout defects — truncated fields, broken pagination, overlapping elements —
are invisible to logic tests because the computation is right and the
*presentation* is wrong. The discipline is **golden-file (snapshot)
testing**: render, compare against an approved reference, and review every
diff deliberately. A snapshot suite whose diffs get approved unread converges
to testing nothing — the review step is the assertion.

What makes golden files find bugs is the *content* fed to them, and that's
where partitioning re-enters: partition the data that drives layout, not the
business rules. Overflow-length names and addresses, the item count that
forces a page break (the boundary is "first row of page two" — a classic
second-page defect class), zero-item and single-item documents, locales with
long translations, right-to-left or multi-byte text. One golden file per
layout-driving partition.

For pixel-level UI output, visual-regression tools (screenshot diffing with a
perceptual threshold) apply the same pattern; keep the threshold tight enough
that a shifted element fails.

---
*Migration and deployment testing practice: Humble & Farley, "Continuous
Delivery"; Sadalage & Fowler, "Refactoring Databases" (migration
discipline). Golden-file/approval testing: approvaltests.org (Falco).
ISO/IEC/IEEE 29119 treats these under test types and environments rather than
test-design techniques — this file extends the skill beyond the classic
technique catalog.*
