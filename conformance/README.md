# DACS v0.1 — conformance vectors (golden primitives)

A set of **24 golden conformance vectors** for the DACS v0.1 §14 conformance test plan: byte-stable **and** accepted by an independent reference verifier, across the deterministic primitive surface where two conformant implementations cannot disagree.

> **Proposed / non-normative.** The steward owns all normative and namespace calls — this is a contributor artifact, not part of the standard. MIT.

## What's here

| Area | § | Vectors |
|------|---|---------|
| Canonicalization (JCS) | §7.1, §7.2 | object key ordering, escaping, integer handling, signed-scope exclusion |
| Canonical decimals (CD-1) | §14.4, §9.3 | trailing-zero / normal-form normalization, exponent rejection, economic equality, positivity |
| Domain-separated signing | §7.7 | roundtrip, tamper, SIG-2 cross-domain, closed 16-separator registry, SIG-4 DACS-X disjointness |
| DACS-1 identity / listing | §6.3 | CCI claim matching, tier-laundering guard, intake-only vs pay-phase rails, validity window, native-address shape |

- `MANIFEST.json` — each case: `{ id, area, spec (§), summary, status: "golden", want }`. `want` is the expected output (value, or `"throws"` for required rejections).
- `vectors/golden.json` — the shared fixture inputs (seeds, documents, addresses) the vectors are computed from.

## How to use

Point your DACS implementation at the same fixture inputs and diff your outputs against each vector's `want`. The executable reference runner that produced + checks these lives in the reference verifier: **github.com/mj-deving/dacs-verify** (`bun conformance/run.ts`).

## Scope note

This PR contributes only the **golden** primitives. A further **18 candidate** vectors exercising the proposed DACS-X dispute (§11.2.1) + disclosure (§8.7) flow are single-impl today; they are held in the cross-implementation loop on **issue #99** (pending the shared full-AttestationBundle fixture, DACS-VERIFY-0004) rather than landed here, so nothing in this directory is anything but byte-stable, reference-accepted golden.
