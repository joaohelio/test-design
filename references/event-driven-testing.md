# Event-Sequence and Idempotency Testing

For code that *consumes* events or messages — queue handlers, webhook
endpoints, event-sourced projections, saga steps — the delivery contract is
part of the input domain. Under at-least-once delivery, **duplicates and
reordering are normal inputs, not error paths**: the broker is allowed to hand
the handler the same event twice, or a stale event after a newer one, and
correctness is defined over the whole *sequence*, not over one delivery.

This is where classic state-transition testing falls short. Its model has two
transition categories — valid and invalid — but a replayed event is neither:
it's a **redundant** delivery of a valid trigger, and the correct response is
a defined no-op, not a rejection. A suite that covers every state and every
transition can still miss the duplicate- and out-of-order-delivery bugs that
dominate multi-consumer architectures.

## The delivery-semantics model

Start from the state table (see
[logic-and-state-testing.md](logic-and-state-testing.md)) and classify every
state × event cell three ways instead of two:

- **Valid** — the event applies and changes state.
- **Invalid** — the event can't legitimately arrive here; must be refused.
- **Redundant** — the event's effect is already applied (replay, or an
  equivalent event won the race); must converge to the same result with **no
  repeated side effect**.

Then add the delivery axis: which orderings and multiplicities the transport
actually permits (at-least-once? per-key ordering only? competing consumers?).
The permitted degradations are coverage items.

## Coverage items

1. **Replay** — every event delivered twice in every state where it validly
   applies. Assert idempotence of *state* and of *side effects*: the second
   delivery must not emit a second outbound event, notification, or charge.
   The property form generalizes this beyond examples:
   `apply(e) ∘ apply(e) ≡ apply(e)` — a natural property-based test when a
   generator for events exists.
2. **Reorder** — for every ordering-dependent event pair, deliver the stale
   one after the newer one. Assert the stale event is detected (version,
   sequence number, timestamp guard) and ignored — silently dropping it into
   the state is the bug being hunted.
3. **Redelivery after partial failure** — simulate a crash between the
   handler's effects (state persisted but not acked, or side effect emitted
   but state not persisted), then redeliver. The sequence must converge; a
   handler that is only idempotent on the happy path double-applies here.
4. **Concurrent delivery** — where competing consumer instances can receive
   the same or related events, run deliveries concurrently and assert a
   single net effect. Only meaningful when the architecture permits it; say
   so when it doesn't.

Full coverage = every redundant cell exercised (replay), every
ordering-dependent pair reordered, and each partial-failure point redelivered
once.

## The dedup mechanism is a test object of its own

Idempotency usually rests on an explicit guard — a dedup key, a version or
sequence check, a transactional outbox. Each guard earns its own cases:
the key *matching* when it should (replay suppressed), the key *not* matching
when payloads legitimately differ (a real update isn't swallowed as a
duplicate), and the guard's storage expiring or being absent. A wrong dedup
key is symmetric: it either lets duplicates through or eats real events, and
only the second failure mode is silent.

## Multi-consumer propagation

When one event fans out to several consumers, test **each consumer's contract
against the event schema independently** (consumer-driven contract style):
each consumer's suite feeds it the producer's schema — including replays and
reorders — and asserts that consumer's behavior. Do not build a full
cross-service end-to-end matrix; it multiplies suite size without adding
coverage items the per-consumer contracts don't already carry.

## Worked example (Go)

A shipment projection consuming `Dispatched` and `Delivered` events, deduped
by event ID and guarded by a version number. Each test case is an **event
sequence**, and the assertions cover both the final state and the emitted
side effects:

```go
tests := []struct {
    name      string
    events    []Event
    wantState Status
    wantSent  int // notifications emitted — the side-effect half of idempotence
}{
    {"happy path", seq(dispatched(1), delivered(2)), Delivered, 2},
    // replay: redundant cell — same state, and no second notification
    {"delivered replayed", seq(dispatched(1), delivered(2), delivered(2)), Delivered, 2},
    {"dispatched replayed", seq(dispatched(1), dispatched(1), delivered(2)), Delivered, 2},
    // reorder: stale version arriving late must be version-guarded away
    {"stale dispatched after delivered", seq(dispatched(1), delivered(2), dispatched(1)), Delivered, 2},
    // dedup key must not swallow a genuinely new event
    {"new event, new id, same type", seq(dispatched(1), dispatched(3)), Dispatched, 2},
}
```

The redelivery-after-partial-failure item needs a fault injection point
(fail after persist, before ack; then redeliver) and typically lives in an
integration test against the real store rather than this table.

When reporting coverage, name the sequence classes: "both events replayed in
their redundant cells, the one ordering-dependent pair reordered, dedup-key
false-positive checked; partial-failure redelivery covered in integration."

---
*Delivery semantics and idempotent-receiver patterns: Hohpe & Woolf,
"Enterprise Integration Patterns" (Idempotent Receiver); Kleppmann,
"Designing Data-Intensive Applications" (exactly-once semantics, ordering);
property-based formulation follows the QuickCheck tradition (Claessen &
Hughes). ISO/IEC/IEEE 29119-4 and the ISTQB syllabi do not cover this
category — it extends the classic state-transition technique.*
