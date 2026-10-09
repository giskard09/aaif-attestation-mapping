# Mapping our specs against AAIF's empirical observability gaps

This is our own read, built for internal review before any outreach. It has
not been shown to, reviewed by, or endorsed by anyone in the AAIF
Observability & Traceability WG. Nothing here has been posted to
`aaif/wg-observability-and-traceability` or anywhere else.

## Source of the gaps (verified against the WG's own document, not from memory)

"Agent Behavior Trace Model: an evidence-first approach — Proposal for the
WG meeting, September 30, 2026" (`~/Downloads/WG - Observability &
Traceability Running Notes.txt`, local copy, unchanged this session).
Survey: 27 coding agents, 34 SDKs/frameworks (each language counted
separately).

| Gap (verbatim) | Evidence (verbatim) | Where the WG says it goes |
|---|---|---|
| "Approvals aren't recorded in a standard way, and denied actions often look like they ran" | "No agent uses a standard form; at least 5 of 19 record denied calls as tool executions" | Existing OTel work: `semantic-conventions-genai` #95, #535 |
| "Retries aren't visible as attempts of one logical call" | "3 of 19 record an attempt number" | "No existing discussion yet; a candidate new issue" |
| "The target of an external effect (file, command, resource) isn't recorded apart from the full tool arguments" | "None of 19" | "No existing discussion yet; a candidate new issue" |

Two of these three gaps have **no destination yet** per the WG's own
document — they are candidates for a new issue in
`open-telemetry/semantic-conventions-genai`, not an existing thread, and not
necessarily this WG's own repo at all.

## What we have, mapped against each gap (our claim, not theirs)

### Gap: "target of an external effect isn't recorded apart from the full tool arguments" (None of 19)

[`execution-join-ref-v1`](https://github.com/giskard09/argentum-core/blob/f4d989052bb3a61691b1b156cd6dd655793fea4f/docs/spec/execution-join-ref-v1.md),
invariant 6 (`effective_call_binding`). The spec separates the *approved*
preimage from the *effective* (actually-dispatched) preimage; a verifier
checks whether a fresh decision covers the effective one, independent of
whether the full argument blob matches. We built and ran this against a
real case: a file-write worked example
(`probityai/agent-evidence-observer`'s `REMORA-EFFECT-BRIDGE.md`) where the
*target path* changed before dispatch — caught as
`EFFECTIVE_CALL_REBINDING_FAILED` without needing to diff the entire
argument object. See
[`giskard09/execution-join-ref-remora-bridge`](https://github.com/giskard09/execution-join-ref-remora-bridge).

**What this does not establish:** that `execution-join-ref-v1`'s `scope`
field (string, implementation-defined shape) is the right OTel attribute
name or wire format for "target of an external effect" generally — only
that separating "approved preimage" from "effective preimage" as distinct,
independently hashed values is one working way to make the target
comparable without re-diffing full arguments.

### Gap: "retries aren't visible as attempts of one logical call" (3 of 19)

Two of our specs address different halves of this:

- [`execution-join-ref-v1`](https://github.com/giskard09/argentum-core/blob/f4d989052bb3a61691b1b156cd6dd655793fea4f/docs/spec/execution-join-ref-v1.md)'s
  `attempt_id` carries an explicit `attempt_seq`; invariant 3
  (`no_duplicate_attempt`) rejects a retry that reuses the same `attempt_id`
  as the original — "a retry is a new attempt_id over a new preimage (with
  incremented attempt_seq)."
- [`idempotency-ref-v1`](https://github.com/giskard09/argentum-core/blob/f4d989052bb3a61691b1b156cd6dd655793fea4f/docs/spec/idempotency-ref.md),
  invariant 5, covers a sharper failure mode the counter alone does not:
  when a retry re-enters *through the model* (not through the original
  caller), the regenerated tool-call arguments are not guaranteed
  byte-identical even when intent is identical — "different token sampling,
  a rephrased justification field, reordered list items." An attempt
  counter derived from hashing those regenerated arguments silently
  produces **two distinct committed effects** instead of a detected
  duplicate — a false negative, not a missed increment. `idempotency_key`
  MUST derive from durable, caller-fixed inputs instead.

**What this does not establish:** an attempt counter alone (what "3 of 19"
measures) answers "is this attempt N" but not "is attempt N the same
logical action as attempt 1 even if its regenerated arguments differ" —
that second question is what our invariant 5 gap write-up exists for, and
it is a sharper claim than what the survey's own "attempt number" metric
checked for.

### Gap: "approvals aren't recorded in a standard way, and denied actions often look like they ran" (at least 5 of 19)

[`execution-join-ref-v1`](https://github.com/giskard09/argentum-core/blob/f4d989052bb3a61691b1b156cd6dd655793fea4f/docs/spec/execution-join-ref-v1.md)'s
`decision_id` carries an explicit `outcome` ∈ `permit | deny | defer` as
part of its hashed preimage, bound to a specific `action_ref` — and
`attempt_id`/`result_id` exist **only** if a dispatch was actually attempted.
A denial that produced no attempt/result chain cannot be reported as
having executed without that absence being independently checkable — the
structure itself, not a convention on top of it, is what prevents a denied
action from looking like it ran. [`revocation-ref-v1`](https://github.com/giskard09/argentum-core/blob/f4d989052bb3a61691b1b156cd6dd655793fea4f/docs/spec/revocation-ref.md)
is the complementary piece for authorization that was valid and later
invalidated: it is append-only (invariant 2) — a revocation never mutates
or deletes the original decision record, it adds a new one.

**What this does not establish:** a standard wire-level attribute scheme
for OTel spans (the gap's own framing). Our specs are content-addressed
trail records, not OTel span/attribute conventions — bridging the two is
exactly the kind of mapping work neither side has done yet, and we are not
claiming it here.

## What we do NOT have (declared honest, not glossed over)

We have no equivalent to the WG's "Compensation Event" concept — a step
declared compensable or terminal *in the effect's contract*, with a
lint checked at workflow-definition time that rejects a graph where a
non-compensable step isn't terminal on every path. This gap was found and
registered by us before this mapping, independently of AAIF, in a
different context: Correctover's `signed-receipt-reference` design
(crewAI#5802) does exactly this for a charge-then-confirm case spanning two
systems. Our own write-up, verbatim:

> "Nuestro idempotency-ref resuelve reintento/duplicación de UNA acción; no
> resuelve una secuencia de acciones irreversibles entre sistemas distintos
> sin transacción compartida (ej. cobrar y después fallar en entregar) —
> dominio distinto, gap real, no forzar mapeo."

(`~/Downloads/IDEAS_ESTRATEGICAS.txt`, entry dated 2026-10-08; full thread in
`~/Downloads/BITACORA_CORRECTOVER.txt`, same date. No design proposed yet —
registered as open design debt, owner undecided between a sibling spec to
`idempotency-ref` or a different domain entirely.)

The WG's own Phase 1 "Tool Intent, Outcome, and Side-Effect Attribution"
deliverable lists "Compensation Event" as one of its key concepts
alongside "Retry Attempt" — so this is a real, named gap on both sides
independently, not something we are retrofitting to look aligned.

## Context, not a request to act on it

Two open issues in the WG's own execution plan read close to this work:
[#44](https://github.com/aaif/wg-observability-and-traceability/issues/44)
("Connect calls, approvals, and external effects") and
[#45](https://github.com/aaif/wg-observability-and-traceability/issues/45)
("Check independent interpretation"). Both are unowned and #44 depends on
two other open, unowned tasks (#42, #43). A contributor (celikkanat)
already offered a fixture for #44 on 2026-09-18 with no response since.
We are not commenting on either issue. These links are here only so this
document doesn't read as if we found the match ourselves without noticing
the WG had already named it.

## P2: Agent Identity, Trust & Verifiability

Confirmed empty in the WG's charter document — a header with no content
under it, unlike P0 and P1 which both have named key concepts. No current
activity to react to here; noted only because it exists as an unclaimed
slot, not because there is anything concrete to propose into it yet.
