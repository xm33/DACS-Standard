# DACS — Conformance test plan (Chapter 14)

> Part of **DACS v0.1**. Companion reference to [CORE](CORE.md) — moved out of the Core document to keep the normative reading surface compact. Original section numbering is retained, so existing citations (e.g. §14.x) remain stable.

The conformance requirements and golden-vector test plan, per role and per module. Machine-readable fixtures live in [conformance/](../conformance/).

---

## Chapter 14 — Conformance test plan

This chapter sketches the test categories an implementer should cover to claim conformance to each DACS standard. It is a **plan, not a test suite**; the test suite itself (test vectors, expected outputs, golden files) is produced separately and tracked alongside reference implementations. Where a chapter’s conformance summary enumerates labelled rules (e.g., BP-1, LR-2, CM-3), the test plan groups them into runnable categories.

### 14.1 DACS-1 — Identify

Exercise each rule at its normative home; full text is not restated (define-once). Fixtures under `conformance/`.

| Rules | Home | Exercise (intent) | Vectors |
| --- | --- | --- | --- |
| Claim-reference parser | §6.3.1 | every scheme: valid canonical / valid non-canonical (canonicalise on read) / invalid grammar (reject) / unknown-scheme (not silently accepted) | `conformance/vectors/` |
| BP-1..BP-4 (bundle producer) | §6.3.2 | produce → canonical form → hash → domain-separated sign → anchor round-trip | `conformance/fixtures/identity/` |
| BR-1..BR-5 (bundle reader) | §6.3.2 | accept-conformant; reject unsigned / missing-required-`verifiedBy` / unverified-`presentedBy`-when-selector-set; unknown-scheme → unverified; SIWD `dacs:<hex>` Resource + session-`Nonce` match | `conformance/fixtures/identity/` |
| match() (BundleRequirement) | §6.3.3 | required missing / required failing / oneOf satisfied / oneOf unsatisfied / selector match / mismatch | `conformance/vectors/` |
| LP-1..LP-4, LR-1..LR-3 | §6.3 | publisher: sign / anchor / version-monotonicity / revocation; reader: halt-on-first-failure, revoked refusal, size-cap | `conformance/vectors/` |
| Discovery | §6.3.6 | well-known parser; catalog endpoint shape; anchor cross-check from `ListingSummary` | `conformance/vectors/` |
| IT-1..IT-3 (identity tier) | §6.3.2.1 | derive from verified-and-fresh claims only; ignore self-asserted; deterministic; institutional precedence; stale-`verifiedBy` does not elevate | `conformance/fixtures/identity/` |

### 14.2 DACS-2 — Vet

| Rules | Home | Exercise (intent) | Vectors |
| --- | --- | --- | --- |
| CM-1..CM-5 (method common) | §7.3 | per-method: input-shape; pass/fail/indeterminate; attestation anchoring; `VerifyResult` with correct method; canonical form + domain-sep signature | `conformance/fixtures/` |
| RA-1..RA-5 + resolution | §7.4 | steward-sig + domain separator; canonical anchoring; version monotonicity; supersede-on-replace; index lookup; content-hash; version pinning | `conformance/` |
| PSP-1..PSP-5 | §7.5.1 | match-predicate per format; parse-fail → error (not fail); negative-match inversion; `indeterminateOn` before match; dataMap extraction non-deciding; deterministic (no script/sub-fetch/redirect); PSP-5 completeness floor before a negative `pass` | `conformance/` |
| VP-R1..VP-R4, VP-C1..VP-C3 | §7.6.1 | transient retry / permanent no-retry / new-attestation / no-retry-on-indeterminate; reuse within effective window; maxAge tightens never widens | `conformance/` |
| Aggregation | §7.7.1 | classify_required branches; oneOf within-group precedence error>indeterminate>fail; cross-accumulator fail>error>indeterminate; VPC-4 counterparty-malformed attribution | `conformance/` |
| RAV-1..RAV-7, RAV-R1..RAV-R5 | §7.4.5, §9.4.4 | recipe availability consumer+steward behaviour; rail preflight; no disabled/failed selection; RAV-R5 authoritative signed read | `conformance/` |
| VPC-1..VPC-4, MA-1..MA-3 | §7.8, §6.3.3 | phase order / two-sided / anchor-before-return / fail-or-indeterminate; matching + `presentedBy` verification | `conformance/` |
| WN-1..WN-6 (warnings) | §7.7 | advisory-only; MUST NOT move `overallDecision`; preserved on `pass`; `suggestedRetryAfterMs` doesn't override recipe; unknown-code conservative | `conformance/fixtures/` |

### 14.3 DACS-3 — Negotiate

| Rules | Home | Exercise (intent) | Vectors |
| --- | --- | --- | --- |
| Channel envelope + failure | §8.3.3, §8.12 | channelmsg domain-sep sig; sequence monotonicity; signature scope; liveness-exceeded → channel-failed; abort round-trip | `conformance/` |
| negotiate-fixed-price | §8.4.1 | live signature path; auto-accept commitment + instance-signature path; reject pre-issued per-instance signatures | `conformance/` |
| RFQ-1..RFQ-4 | §8.4.2 | maxTurns; turn-timeout; out-of-band-terms rejection at commit-agreement | `conformance/` |
| SE-1..SE-7 | §8.4.3 | commitDeadline (chain-timestamped); reveal-window vs SR-2 anchor (SE-3); mismatch exclusion; anchored-reveal-set selection (relay-suppression); exclusion ordering (currency/non-positive before reserve); reserve floor/ceiling inclusive; tie-break (SE-5); empty-set → negotiate-failed; rule-ref content-hash binding (SE-6); bidHash domain-sep + salt floor (SE-7) | `conformance/` |
| PS-1..PS-3 | §8.8 | exactly-one negotiate phase; commit immediately follows; pattern ↔ pricing-model compatibility | `conformance/` |
| Agreement validation | §8.5.2 | price-band / rail-acceptance / deliverable / deadline / pattern checks; `priceAnchor` valid-when-present, optional | `conformance/` |
| CA-1..CA-4 | §8.6 | refuse-advance-until-ok; double-commit reject; immutability after anchor; domain-sep commitment signature | `conformance/` |

### 14.4 DACS-4 — Settle

Exercise each rule at its normative home; the full rule text is **not** restated here (define-once — same discipline as the §12.4 threat index). "Exercise" is one-line test intent; executable fixtures live under `conformance/`.

| Rules | Home | Exercise (intent) | Vectors |
| --- | --- | --- | --- |
| RD-1..RD-5 | §9.4.3 | steward-sig + domain separator; anchor; version monotonicity; railType↔asset/network consistency | `conformance/fixtures/settlement/` |
| PC-1..PC-7 | §9.5.1 | input-shape; anchored evidence; correct `attestationRef` (deferrable under PC-7); all `errorClass` values; PC-5 currency-resolution; PC-6 `settlementFinality` present-on-success/absent-on-delivery; PC-7 cross-chain anchor decoupling | `conformance/fixtures/settlement/` |
| HTLC-1..HTLC-10 | §9.5.4 | buyerSalt entropy/confidentiality/non-reuse; HKDF derivation + input-uniqueness; canonical claim order; per-chain hashlocks; timelock asymmetry on absolute expiry (pinned params, source-finality margin); HTLC-9/ST-8 asymmetric resolution; HTLC-10 free-option | `conformance/fixtures/settlement/htlc9-asymmetric.json` |
| CD-1 | §B.2 | economically-equal decimals (`"1.50"`=`"1.5"`) → identical hashes/signatures | `conformance/vectors/` (CD-1) |
| AMEND-1..AMEND-4 | §9.7.1 | `amendsEvidenceRef` resolves + jobId match; refund/partial-refund reference success-only; summed `refundAmount` ≤ `paymentAmount`; flagged-amendment not treated as valid unwind | `conformance/fixtures/settlement/` |
| PIPE-1..PIPE-5 | §9.9 | ≥1 deliver (pay-* optional, §6.3.4(8)); deterministic ordering; pay↔deliver gating; phase repetition | `conformance/vectors/` |
| Per-rail procedures | §9.5.2–§9.5.7 | erc20/spl decimal-conversion (no float) + finality wait; tank BridgeOperation lifecycle + route scope; ap2/x402 mandate-revocation + receipt-signature | `conformance/fixtures/settlement/` |
| Delivery phases | §9.6 | storage-program (normal + extended-pointer); entitlement sig/anchor/scope; attested-payload composing a DACS-2 attestation | `conformance/fixtures/` |

### 14.5 DACS-5 — Verify

| Rules | Home | Exercise (intent) | Vectors |
| --- | --- | --- | --- |
| ST-1..ST-8 (state machine) | §10.3.1 | every `(from→to)` legal-only; illegal-pair reject (ST-1); abort from any `*-pending` (ST-3); rate branch + non-fatal (ST-4/5); ST-7 pause→resume / →failed-substrate; ST-8 `settle-asymmetric` forward-resolution (→completed on final source-claim, →failed-counterparty on expiry, →paused on SR-2 outage), non-terminal; terminal→`outcome` map (ST-6) | `conformance/fixtures/settlement/` |
| Bundle production | §10.4 | two-sided anchoring at role addresses; `anchoredByRole` ↔ address (mismatch rejected); canonical-equality happy path (excludes anchoredByRole/signatures); `dacs-bundle:v1:` sig; extended-pointer | `conformance/fixtures/` |
| Bundle consumption | §10.4.3 | two-sided lookup; one-sided → aborted-by-self; divergence = `outcome`/`phaseSummary` contradiction (advisory skew is NOT divergence); per-party policy; "disputed" is a consumer verdict, not an outcome value | `conformance/fixtures/` |
| Reputation derivation | §10.5.1 | all outcome partitions; party-fault denominator excl. failed-substrate; null vs zero; empty-input totality; two-sided reconciliation + `perspective_flip`; reconciliation guards; rating de-duplication | `conformance/fixtures/reputation/` |
| Determinism receipt | §10.5.3 | `bundleRefs` = `reconciled` set, ascending-`contentHash` order; re-derive byte-identical `metrics`/`bundleCount` under recorded `windowingBasis`; omitting basis or mis-ordering is non-conforming | `conformance/` |
| Category-scoped derivation | §10.5.4 | prefix filter before §10.5.1; non-resolving `agreementRef` excluded; exact-or-`category+"."` prefix; hint accuracy | `conformance/` |
| RT-1, RT-2 (rate phase) | §10.6.1 | run-after-settle; one-record-per-direction; rating domain-sep sig; RT-1 producer-reject out-of-range/over-length; RT-2 deriver-exclude non-conforming; `dimensions` opaque | `conformance/` |
| ERC-8004 publication (optional) | §10.7 | token-owner-signed entry; bundle-anchor pointer; rate-limit | `conformance/` |

### 14.6 Universal signature scheme & canonical form (SIG-1..SIG-5, CF-1..CF-4, CD-1)

A cross-cutting test category that every conforming implementation runs once:

- Sign every artifact kind in §B.7 with a known key; verify with the same key against the domain-separated payload; reject if the verifier reconstructs without the separator.
- Cross-artifact replay test: take a valid signature on artifact kind A, attempt to verify it as a signature on artifact kind B with the same hash bytes; verification MUST fail.
- Unknown-artifact x-* prefix test: implementations encountering an unknown domain separator MUST reject; experimental x- separators MUST be accepted only with out-of-band agreement.
- **CF-1 (NFC).** A document carrying a non-ASCII identifier supplied in NFD form MUST hash and verify identically to the same document supplied in NFC form; a verifier MUST normalise before recomputing the canonical form.
- **CF-2/CF-3 (ClaimReference canonical form & identity).** `CCI-LEI:…` and `cci-lei:…` MUST produce identical content hashes when embedded in a signed document (scheme case-folded). Two references differing only in parameter order MUST produce identical canonical bytes; two references differing only in the presence/value of parameters MUST resolve to the same reputation key (parameters excluded from identity).
- **CF-4 (logical-address encoding).** A logical address built from a multi-colon primary claim MUST round-trip: assemble → derive native address → split the logical address back into `{sellerPrimaryClaim, listingId, listingVersion}` and percent-decode each to the exact originals. Likewise, per address kind:
  - the DACS-4 payment-evidence address `dacs4:payment:{jobId}:{railId}:{phaseIndex}` MUST round-trip a multi-colon `railId` (e.g. `evm-erc20:1:USDC`) — the encoded railId splits back to its exact original while `phaseIndex`/`resolved` remain unescaped fixed segments;
  - the DACS-2 addresses `dacs2:{jobId}:{scheme}:{identifier}:v{recipeVersion}` and `dacs2:composite:{jobId}:{evaluatedParty}` MUST round-trip a multi-colon `{identifier}` / `{evaluatedParty}` (e.g. `evm:mainnet:0x1234`) while `{scheme}`/`v{recipeVersion}` remain unescaped;
  - the DACS-5 rating address `dacs5:rating:{jobId}:{rater}` MUST round-trip a multi-colon `{rater}`.

  An address whose variable segments are left raw (unescaped) MUST be rejected as malformed.
- **CD-1 (canonical decimal).** `"1.50"` and `"1.5"` as `PriceTerm.amount` MUST produce identical agreement hashes and signatures.
- **SIG-5 (preserve-unknown).** A verifier built against schema vN MUST successfully verify the signature on a document produced under vN+1 that adds an unknown field, by hashing the document as received (unknown field included); a verifier that strips the unknown field before hashing (and thus rejects) FAILS this test.

### 14.7 Governance (GOV-1..GOV-3)

- **GOV-1 steward disclosure.** A registry consumer surfaces which signing key it treats as authoritative and does not present the single steward as a constituted multi-party body.
- **GOV-2 anchoring-phase disclosure.** An implementation discloses its operating phase (in-code / single-signer / multisig).
- **GOV-3 anchoring-phase verification.** A consumer reads the resolved recipe's `governance.anchoring` and evaluates each pinned recipeVersion against the phase recorded at pin time; a recipe marked `in-code` is not treated as canonically anchored.

### 14.8 Substrate-capability tests

For substrates other than Demos that claim conformance, additional capability tests apply:

- **SR-1.** Sub-identity binding test: a root key binds N sub-identities, presents under a single SR-1 signature, verifier resolves each to its claim scheme.
- **SR-2.** Anchor-write → retrieve → content-hash check round-trip; size-cap enforcement.
- **SR-3.** Fetch-specification → consensus-signed commitment → anchor; body-hash verification by independent consumer. (v0.1 conformance bar is trust-property; v2 will add wire-protocol tests.)
- **SR-4.** Channel-establish → member-only-message-delivery → non-member-cannot-read; CH-1..CH-6 each as a test (CH-6: channelId unique per session — cross-session offer-replay rejected). (v0.1 trust-property; v2 wire-protocol.)
- **SR-5.** Cross-chain lock → release with bounded-time atomicity; refund path on counterparty timeout.

### 14.9 Out of scope for v0.1 conformance

The following are not part of v0.1 conformance and SHOULD NOT be tested as such:

- Cross-substrate interoperability for SR-3- or SR-4-dependent phases (deferred to v2).
- Multi-party transactions beyond bilateral plus sealed-envelope (deferred).
- Streaming / continuous-flow rails (deferred).
- Cross-DACS-version pipelines (deferred).
- Dispute *resolution* flows (DACS-X, anticipated). Divergence *detection* — the two-sided lookup plus canonical-divergence classification and per-party policy of §10.4.3(d) — **is** in scope for v0.1 conformance; only the resolution layer is deferred.

