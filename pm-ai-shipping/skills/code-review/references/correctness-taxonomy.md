# Correctness taxonomy — twelve lenses, with detection tells

*Reference for the correctness sub-case — the core of the `code-review` skill. The performance and
security sub-cases have their own files alongside this one.*

Overlapping diagnostic lenses, not a classification scheme and not a quota. Each entry says **how you
detect it**, because a class name alone changes nothing about what a reviewer looks at.

Lenses 1 and 2 are the ones strong reviewers — human and machine — miss most often, and the only two
that warrant an explicit forced probe rather than a read-through. Both are failures of *looking*, not
of judgement: the code reads sensibly at each participant, and the defect exists only in the
relationship between them.

---

### 1. Authority reconciliation
Follow a proposed value through validation, normalisation, negotiation or commit. Identify downstream
state still derived from the **proposal** when the authority can legitimately return something
different. Check that reconciliation happens *before* the first consequential use.

> **A requested value is not an applied value.** Anywhere a system can say "I heard you, and here is
> what I actually did", the reply is the authority — and the request is not.

**Probe:** construct an execution where the effective answer differs from the requested one, then ask
which stored state, which UI, which later call and which log line still carry the request.

### 2. Identity and correlation
Trace how an operation's result finds its originating entity. Establish the correlation key's
**uniqueness, stability and lifetime** under overlapping operations, reordering, removal and reuse. A
label, an index, a position or arrival order is suspicious exactly when one of those can fail.

**Probe:** run two operations concurrently and complete them out of order. Then remove one mid-flight
and reuse its slot.

### 3. Freshness and generations
Mark every value captured before a yield, await, callback, timer or lock release. Determine what can
change before it is used, and whether the operation still targets the intended entity *and version*.
Check what the code actually does with a stale result — ignore, apply, or apply silently.

### 4. Lifecycle and derived state
Compare every reachable construction, replacement, restoration, reset, failure and termination path.
Look for derived fields or cached decisions correctly re-established on one path and wrongly retained
on another. Two paths that should end in equivalent state are an agreement like any other.

### 5. Atomicity and partial failure
Split multi-effect operations at each failure and cancellation point. Is partial state permitted,
recoverable and accurately reported? Look for success reported before the required effects are
durable.

### 6. Replay and effect cardinality
Follow retries, duplicate delivery, repeated callbacks and re-entry into effects. Compare the actual
delivery guarantee against the required effect count — especially where an effect costs money, sends
a message or mutates a shared total. Inspect deduplication scope, lifetime, and behaviour after
partial success.

### 7. Composition and precedence
Trace independently produced pieces through merge, reduction, ordering and dispatch. Compare the real
overwrite and selection rules against the intended authority or priority — including transformations
applied *after* the merge that quietly re-order or re-key it.

### 8. Bounds, units and accounting
Follow counts, lengths, offsets, capacities and totals through every transformation. Check empty and
boundary cases, overflow, rounding, and whether measurement and consumption use the same unit and
representation.

### 9. Framing and incremental processing
Compare logical item boundaries against actual read, write, iterator and callback boundaries. Test
split items, combined items, partial writes and early termination. Inspect buffering, flush and
finalisation — especially the last item.

### 10. Representation and information loss
Compare the values a producer can emit against the distinctions a consumer relies on: absent versus
empty, zero versus missing, signed versus unsigned, canonical forms, precision, encoding, equality
semantics. Trace round-trips and derived outputs for distinctions that disappear.

### 11. Ownership, completion and progress
Trace who may mutate, release, cancel and complete an operation or resource. Inspect exceptional
exits and competing terminal paths for premature release, missing completion, deadlock, or use after
ownership changed hands.

### 12. Decisions and dispatch
Enumerate the meaningful states and inputs for consequential predicates and dispatch tables. Compare
branches against supported expectations: overlapping conditions, inverted tests, missing cases, and
what the fallback actually does.

---

## Using these

- They overlap on purpose. One defect can be lens 1 and lens 3 at once; report the root cause once.
- Absence of findings under a lens is a valid result. Do not manufacture one to fill the table.
- Finding nothing under lenses 1 and 2 is worth a second look *only* if you never constructed their
  probes — a read-through reliably returns nothing here, which is exactly the failure mode.
- None of these is language- or framework-specific. If a lens seems inapplicable, say which property
  of the system makes it so.
