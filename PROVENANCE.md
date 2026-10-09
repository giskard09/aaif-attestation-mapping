# PROVENANCE

All quotes below were re-verified against their source in this session,
immediately before writing `MAPPING.md` — not carried over from an earlier
summary.

| Claim / quote | Source | How verified |
|---|---|---|
| Survey numbers (27 agents, 34 SDKs) and the three gap-table rows | `~/Downloads/WG - Observability & Traceability Running Notes.txt` (local file, unchanged — "Agent Behavior Trace Model: an evidence-first approach — Proposal for the WG meeting, September 30, 2026") | Read directly, this session |
| `execution-join-ref-v1` invariant 6, attempt_id/attempt_seq, invariant 3 text | `giskard09/argentum-core`, `docs/spec/execution-join-ref-v1.md`, `main` `f4d989052bb3a61691b1b156cd6dd655793fea4f` | `grep` against the file on disk, this session |
| `idempotency-ref` invariant 5 text ("different token sampling...") | `giskard09/argentum-core`, `docs/spec/idempotency-ref.md`, same commit | `grep` against the file on disk, this session |
| `revocation-ref` invariant 2 ("revocation is append-only") | `giskard09/argentum-core`, `docs/spec/revocation-ref.md`, same commit | `grep` against the file on disk, this session |
| Compensation Event / saga gap quote | `~/Downloads/IDEAS_ESTRATEGICAS.txt`, entry dated 2026-10-08 (NOT `DEUDA_INTERNA.txt` — that file has no matching entry, checked directly with `grep`) | Read directly, this session; cross-checked against `~/Downloads/BITACORA_CORRECTOVER.txt` 2026-10-08 |
| Issue #44 / #45 status, dependencies, celikkanat's unanswered 2026-09-18 comment | `aaif/wg-observability-and-traceability` issues #42, #43, #44, #45 | `gh api`, this session |
| P2 charter area empty | `aaif/wg-observability-and-traceability`, `working-documents/*` charter doc referenced in the local WG notes file | Read directly, this session |

## What this does NOT claim

- That the AAIF WG agrees with any of this mapping, or that it has been
  shown to anyone there.
- That `execution-join-ref-v1`'s field shapes are a proposed OTel
  convention — only that the underlying distinction (approved vs. effective
  preimage; decision outcome bound to action_ref; attempt counter vs.
  content-derived key) is one working answer to a named gap.
- That we have a saga/compensation layer. We don't, and the gap is
  registered as open design debt with no owner or timeline.
