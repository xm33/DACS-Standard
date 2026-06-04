# DACS v0.1 — Spec Size Analysis (analyze-first, no changes made)

/ Produced 2026-06-04. Non-normative working analysis. Nothing in the spec was changed. /

## 1. The measured problem

- **52k words / 3,986 lines.** ~10–15× any single peer (ERC-8004 ~3–4k, ERC-8183 ~2–3k, Virtuals ACP core lean).
- Lifecycle chapters 6–10 = **82%** of the doc; each 575–749 lines.
- Prose-density fingerprint of 9 review rounds: **1,911 parentheticals** (1 per 27 words), **540 em-dashes**, **50 sentences >60 words**, single paragraphs of **1,032** and **701** words.
- ~30 rule families.

Two things are true at once:
- **Breadth is essential** — DACS is a 5-stage lifecycle stack; the fair peer is *8004 + 8183 + x402 + AP2 + KYB/sanctions + portable reputation combined*, not a single EIP.
- **But a real chunk is accidental bloat** — rationale woven into normative text, defenses restated 3× (chapter prose + §14 conformance + §12 threat model), mega-sentences. Normative core ≈ 55–60% of words.

## 2. Worked example — compression is behavior-preserving

The §9.5.3 `buyerSalt entropy and lifecycle` paragraph (the worst offender, 1,032 words, one paragraph) compressed to **394 words (−62%)**; the normative-only reading dropped to **263 words (−75%)**. **Every rule HTLC-1..HTLC-10 was preserved** — only inline rationale was extracted to one non-normative Rationale box (the EIP `Specification` vs `Rationale` split we currently don't follow).

This is the lowest-risk, highest-readability lever and works on the ~12 mega-paragraphs + the 50 long sentences.

### Whole-spec estimate (Lever 1: editorial compression) — CORRECTED by PoC

The earlier ~30–35% estimate was too optimistic. **Empirical result from a full Lever-1 pass on Chapter 9** (branch `spec-compression`, behavior-preserving, 35/35 rule IDs kept, all validators + 35 tests green):

- buyerSalt paragraph (worst offender): 1,032 → 394 words (**−62%**).
- **Chapter total: 10,402 → 9,112 words (−12.4% all-in; −14.5% prose-only).** Lines went *up* (+21) because bulletizing trades line-count for scannability.

Why far below 30%: ~25% of the chapter is incompressible type-definition code; the genuinely-redundant defensive prose is only ~15% of words; the rest is reader-helpful non-normative prose (motivation, rationale, choosing-a-rail) that shouldn't just be cut.

- **Realistic whole-spec Lever-1 projection: ~52k → ~45k words, ~3,986 → ~3,500 lines (≈ −12–15%).** A useful trim and a real *scannability* gain (rules-at-a-glance, rationale separated) — but the document stays "big."
- **To actually reach "manageable like 8004" you need Lever 2 (modularize)** — put the core reader in front of ~1,100 lines — or Lever 3 (trim scope). Editorial alone cannot get there. This is the PoC's main finding.
- Behavior-preserving discipline confirmed viable: rule-ID-set diff + validator gate caught nothing dropped.

## 3. Lever 2 — Modularize: core + composable modules

Mirrors why 8004 stays small (a registry others compose). A reader/implementer meets a **~1,100-line core**, not a 3,986-line monolith.

| Spec | Contents | ~lines (post-compress) |
|---|---|---|
| **DACS-Core** | §1–5, data model (IdentityBundle / AgreementDocument / SettlementEvidence / AttestationBundle / ReputationDerivation / PhaseHandlerResult / SessionContext), universal signature + canonical form (SIG/CF/CD), 5-stage skeleton + phase-handler contract + session state machine (ST), SR-1..5, governance/versioning, threat-model frame, conformance frame | **~1,100** |
| Module: **DACS-2 Vet methods** | the 8 methods, recipe schema, PSP/RA/RAV/VP-* | ~450 |
| Module: **DACS-4 Payment rails** | PC-1..7 contract + per-rail (erc20, spl, **HTLC topology**, liquidity-tank, x402, AP2) | ~550 (HTLC ~250) |
| Module: **DACS-3 Negotiation patterns** | RFQ, sealed-envelope, channels (CH/SE/RFQ) | ~400 |
| Module: **DACS-5 Reputation detail** | `derive()`, two-sided reconciliation, determinism receipt | ~250 |
| Module: **DACS-X Dispute** | (already a separate design seam, #98/#99) | n/a |

Cost: cross-references become cross-document; a release must version core+modules coherently. Benefit: each artifact is independently readable, reviewable, and adoptable.

## 4. Lever 3 — Trim v0.1 scope (product decision, not editorial)

Candidates to defer to a later minor/module, ranked by defer-safety:

1. **AP2 fiat rail** — depends on external FIDO AP2 spec (April 2026); low coupling. Safe defer.
2. **Liquidity-tank rail** — Demos-Phase-1 specific; ref-impl notes Phase 1 not yet stable. Safe defer to a Demos module.
3. **Second negotiation pattern** — ship ONE of RFQ / sealed-envelope in v0.1-core; defer the other. Medium.
4. **Heavy vet methods** — ship key/DID + one attestation method + OFAC; defer zktls/tlsnotary/VC to the Vet module. Medium.
5. **Cross-chain HTLC** — single biggest chunk, but a flagship differentiator vs 8183/8004. Trimming it is a *positioning* call, not a cleanup — do NOT defer lightly.

## 5. Recommendation

- **Do Lever 1 regardless** (−30–35%, zero normative change, pure readability win). Sequence it as a behavior-preserving pass with the impact-evaluator gate, one chapter at a time, measuring before/after.
- **Lever 2 is the real structural answer** to "why are they manageable and we aren't" — but it's a bigger restructure; worth a dedicated decision once Lever 1 shows the compressed core size.
- **Lever 3** only if positioning calls for a leaner v0.1; it's reversible (deferred items become modules).

Suggested order: Lever 1 → re-measure → decide Lever 2 against the real compressed core → Lever 3 only if still too broad.
