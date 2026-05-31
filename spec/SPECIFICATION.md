# DACS — Demos Agent Commerce Standards

**Introduction and DACS-1 through DACS-5**

> Published as DACS **v0.1** — the first publicly released version, with all five per-stage standards versioned together as a single document. See [CHANGELOG](../CHANGELOG.md) for normative change history.

## About this document

This document specifies DACS — the Demos Agent Commerce Standards — across five per-stage standards: DACS-1 (Identify), DACS-2 (Vet), DACS-3 (Negotiate), DACS-4 (Settle), and DACS-5 (Verify). Shared material (terminology, substrate capabilities, the Demos production mapping, references) is presented once in the front and back matter rather than repeated per chapter. Each per-stage chapter contains the material specific to that stage. The companion DACS Dev Tasks working document is published separately and is **not** part of the standards.
**Normative language.** This document uses the RFC 2119 / RFC 8174 keywords **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL**, interpreted as in those RFCs. Keywords are normative only when in uppercase.
**Versioning.** This document is **DACS v0.1**, the first publicly released version. Earlier drafts circulated internally under per-stage version numbers (DACS-1..5 v0.1, paper v0.7) and a brief v0.8 cut that consolidated review-pass revisions; those numbers are retired in favour of a single document-level version. From v0.1 onward, the entire DACS standard (all five per-stage standards plus the front-matter substrate-binding, the threat model, the glossary, and the conformance plan) versions together. Per-stage specs do not version independently. Future versions will be v0.2, v0.3, and so on through to v1.0 once the standard is considered ready for unsupervised production use.

## Abstract

Autonomous agents are transacting with agents they have never met. Open standards exist for fragments of what a transaction requires — identity registries, payment authorisation, HTTP-layer micropayments, capability discovery — but each addresses one slice of the problem. Nothing in widespread use composes the fragments into a working commerce lifecycle, which is why agents that need the full lifecycle today fall back to closed operator marketplaces.
**DACS — Demos Agent Commerce Standards** — is the protocol Demos uses for agent commerce. It is organised around the five stages every agent-to-agent transaction passes through: **Identify, Vet, Negotiate, Settle, Verify.** For each stage, DACS composes with the existing standards that already work and adds new standards where the open ecosystem has gaps. Each new standard names the substrate capability it depends on. DACS is built for the Demos Network, but the capability-level specification is kept clean of Demos-specific dependencies so a substrate that provides the same capabilities can host a compatible implementation.

## 1. The problem

An agent making a transaction with another agent today has three working options:
The fragments exist. The lifecycle does not. A buyer agent today can use existing open standards to discover that a seller agent exists, recognise the seller’s identity, and authorise a payment to it. What it cannot do with open standards alone:
These gaps map cleanly to four of the five stages of a transaction. They are why institutional and regulated agents still fall back to operator marketplaces. DACS is designed to provide the lifecycle on a public, permissionless substrate, composing with the open standards that already work for individual stages while filling the gaps that remain.
**Why this matters now.** Agent commerce is moving from prototype to production. Google’s AP2, donated to the FIDO Alliance in April 2026, is establishing the payment-mandate envelope. Coinbase’s x402 is settling per-request micropayments. ERC-8004 is formalising on-chain agent identity in the Ethereum Foundation pipeline. ERC-8183 adds an EVM-native escrow primitive for job-style transactions. A2A is becoming the default capability discovery pattern. Each is real, deployed, and useful. Each addresses one slice. None of them, individually or together, specifies a complete commerce lifecycle with the discipline a regulated flow requires.
Meanwhile the closed alternatives are consolidating fast: full-stack agent platforms are building their own identity, payment, reputation, and dispute systems behind operator-owned APIs. If a usable lifecycle does not emerge from the public-chain ecosystem soon, the agent economy ends up looking like the app stores, with two or three platforms collecting rents on everything that moves. DACS is the Demos contribution to keeping the lifecycle on public infrastructure.

## 2. The approach

DACS follows three principles.
**Composition.** Identity, payment, and several forms of credential attestation already have working standards with real adoption. There is no value in replacing them, and reinventing them slows everyone down. DACS uses composition, not replacement.
**Gap-filling.** Where there are real gaps, DACS specifies a new standard. New standards stay narrow in scope and are designed to compose cleanly with the rest of the stack.
**Stated substrate requirements.** Each new DACS standard names the substrate capability it depends on. The capability is the requirement, stated in the spec. Which substrate provides it is operational detail. This keeps DACS substrate-agnostic in specification while staying honest about what the new standards actually need underneath.
Three things follow:

## 3. The five stages

Every agent-to-agent transaction, whether a $5 data lookup or a $5M institutional swap, passes through five stages:
DACS is one standard per stage. Each standard either fully composes existing open standards for that stage, or specifies what is needed to close the gaps. Phase types specific to a stage (e.g. negotiate-rfq, pay-cross-chain-htlc) belong to that stage’s standard.
| Standard | Stage | Scope |
| --- | --- | --- |
| DACS-1 | Identify | Agent identity, signed and anchored service listings, discovery (.well-known/agent.json extension and off-chain catalog) |
| DACS-2 | Vet | Method-pluggable credential attestation against authoritative sources |
| DACS-3 | Negotiate | Private negotiation phases (RFQ, sealed envelope, fixed price), agreement commitment |
| DACS-4 | Settle | Payment rail registry, payment phases, delivery phases |
| DACS-5 | Verify | Session record, attestation bundle, reputation derivation |

The stages are sequential within a transaction. The standards version independently. Chapters 6–10 below specify them in detail.

## 4. Per-stage summary

A compact summary of what each stage composes, what it adds, and which substrate capabilities it depends on. Chapters 6–10 expand each row in full.
| Stage | Existing standards used | DACS additions | Substrate capabilities |
| --- | --- | --- | --- |
| Identify (DACS-1) | ERC-8004, W3C DIDs, A2A, authority and platform identifiers | Identity claim reference scheme; identity bundle schema; listing schema; .well-known extension; catalog API | SR-2; SR-1 optional |
| Vet (DACS-2) | W3C VC, TLSNotary, zkTLS | Method-pluggable recipe registry; proxy-attestation method for public-registry credentials; composite verification record | SR-2, SR-3 |
| Negotiate (DACS-3) | (none widely adopted) | Private negotiation phases; agreement commitment | SR-4 (private patterns), SR-2 |
| Settle (DACS-4) | AP2, x402, ERC-20, SPL, HTLC | Payment rail registry; payment and delivery phases (incl. Liquidity Tanks) | SR-2, SR-5 (cross-chain only) |
| Verify (DACS-5) | ERC-8004 reputation registry (publication surface) | Session record; attestation bundle; reputation derivation | SR-1, SR-2 |

Three stages compose substantially with existing standards (Identify, Vet, Settle). Two stages are gap-filled almost entirely (Negotiate, Verify). All five reference the same small set of substrate capabilities introduced in chapter 5.

## 5. Substrate capabilities

Every DACS standard names the substrate capability it depends on. The capabilities below are the complete set; each per-stage chapter cites a subset by ID.
| ID | Capability | Description | Used by |
| --- | --- | --- | --- |
| SR-1 | Cross-substrate identity aggregation | Optional. A composition primitive binding one root key to multiple sub-identities — per-substrate keys, verified Web2 identifiers, authority-issued identifiers, platform accounts — presented under a single signature. Cross-Context Identities (CCI) is the Demos implementation. | DACS-1, DACS-5 |
| SR-2 | Anchored, immutable storage | Content-addressed key-value storage with chain-anchored writes, suitable for signed documents up to a soft size limit. Storage Programs is the Demos implementation. | DACS-1, 2, 3, 4, 5 |
| SR-3 | Consensus-backed proxy attestation of HTTP responses | A substrate primitive that, given a fetch specification, returns a response signed by a consensus set of validators and anchors the attestation on chain at production throughput. DAHR (Data Agnostic HTTPS Relay) is the Demos implementation. | DACS-2 (one of several methods) |
| SR-4 | Identity-keyed private coordination channels | Private channels whose membership is bound to the substrate’s public-chain identity and whose contents stay between members. The public chain sees only commitments. L2PS (Layer-2 Privacy Subnets) is the Demos implementation. | DACS-3 (private negotiation patterns) |
| SR-5 | Multi-chain coordinated atomic settlement | Atomic settlement across substrates. May be provided by substrate-native cross-chain transactions, HTLC contracts on participating chains, or pre-funded liquidity primitives such as Liquidity Tanks on Demos. | DACS-4 (cross-chain settlement only) |

A substrate shipping all five can host a full DACS implementation. A substrate that ships some subset can host DACS partially: listings whose pipelines require unsupported capabilities are unfulfillable there, but the rest of the stack still works.
**Substrate-coupling status in v1.** SR-1, SR-2, and SR-5 are specified at the protocol level: another substrate that ships an equivalent primitive (cross-substrate identity aggregation; content-addressed anchored storage; atomic cross-chain settlement) can interoperate with DACS implementations on Demos at the artifact level (the bundles, listings, evidence records validate the same way). **SR-3 and SR-4 are specified at the trust-property level only in v1.** Two substrates each shipping their own SR-3 (consensus-backed proxy attestation) or SR-4 (identity-keyed private coordination) implementations will *not* be wire-protocol interoperable; the trust properties listed under CH-1..CH-5 for SR-4 and under §7.3.5 for SR-3 are the conformance bar v1 requires, but the underlying message formats and consensus signatures are substrate-specific. v2 of DACS-2 and DACS-3 is expected to specify wire formats for SR-3 attestation envelopes and SR-4 channel messages that enable cross-substrate interoperability; until then, a session begun on substrate A cannot be completed on substrate B if it uses any SR-3- or SR-4-dependent phase.
**Reference substrate.** The **Demos Network** is the substrate against which DACS was designed and, as of this draft, the only substrate that ships all five capabilities natively. The DACS specifications cite the substrate capabilities (SR-1 through SR-5), not the Demos primitives themselves; this separation keeps the artifact-level specification portable while staying honest about which primitives are concretely realised today and where v2 work is needed.

## 6. Demos production mapping

A mapping of which substrate primitives are live today, what extensions are needed for v1, and which dependencies are third-party. The mapping applies to every per-stage standard — DACS-1 through DACS-5 — in this paper.
**Legend.** 🟢 in production today; 🟡 Demos team to add for v1; 🔵 third-party (composed, not built by Demos). This legend describes the substrate-primitive status — what the chain ships. Per-recipe and per-rail operational status uses the normative availability field defined in §7.4.5 (recipes) and §9.4.5 (rails). The legend here is informative about substrate features; availability there is normative about specific attestation paths and settlement rails. Earlier drafts conflated the two surfaces by extending this legend to recipes and rails; that conflation has been corrected in v1.

### 6.1 SR-1 — Cross-Context Identities (CCI)

**Stor-backed credentials.** The stor-cred:<type>:<id> scheme convention is the extensibility surface for future credentials not yet promoted to native CCI contexts. **OFAC-clear is not a CCI context** — it is a per-session freshness check that lives only in DACS-2’s CompositeVerificationRecord (it is a check, not a stable identity claim).

### 6.2 SR-2 — Storage Programs

**Logical vs native addresses (applies universally).** Throughout this document, addresses of the form dacs1:…, dacs2:…, dacs3:…, dacs4:…, dacs5:… are *logical* addresses: substrate-independent, human-readable, stable identifiers the protocol reasons about. Each substrate maps them to its native addressing under a deterministic rule. On Demos the mapping is native_address := "stor-" + sha256(logical_address); the colon-containing logical string is not used directly as a StorageProgram name (Demos does not permit colons in names). Other substrates substitute their own mapping; the requirement is that the mapping is deterministic, one-to-one, and reversible by any party knowing the logical pattern. Implementations on a given substrate MUST anchor at the native address and MAY carry the logical address as descriptive metadata; consumers MUST resolve the native address from the logical pattern before reading.

### 6.3 SR-3 — DAHR (Data Agnostic HTTPS Relay)

### 6.4 SR-4 — L2PS (Layer-2 Privacy Subnets)

**DACS-3 phase types are realised as DemosWork WorkSteps.** Each negotiation pattern compiles to a sequence of WorkSteps with context: "xm" | "web2" | "native" and DACS-defined content shapes.

### 6.5 SR-5 — Native Bridges / Liquidity Tanks

**v1 cross-chain settlement scope.** pay-cross-chain-liquidity-tank is supported **only** for the rails currently live in tankAddresses.json (ETH Sepolia, Polygon Amoy; USDC; unidirectional EVM source). All other tank rails in the registry are 🟡 to-add and will unlock as Native Bridges Phase 2–4 ship. pay-cross-chain-htlc is the path the reference implementation runs today; v1 keeps both first-class.

## 7. Global terminology

Terms used in more than one per-stage chapter are defined here once. Per-stage chapters define only terms unique to that stage.

### 7.1 Claim references and identity

### 7.2 Anchoring and signing

### 7.3 Verification and evidence

### 7.4 Session, pipeline, and phases

### 7.5 Shared phase-handler types

Every phase handler in the stack consumes a SessionContext and returns a PhaseHandlerResult. The full TypeScript declarations:
type SessionContext = {jobId: stringlistingRef: { listingId: string; version: number; contentHash: string }recipeRegistryVersion: number // DACS-2 registry pinned at session startrailRegistryVersion: number // DACS-4 registry pinned at session startparties: SessionParty[]priorPhaseOutputs: Record<string, unknown> // accumulated contextDelta from completed phasessigner: SubstrateSigner // substrate-specific signing capabilitystartedAt: number // unix ms}type PhaseHandlerResult = {ok: booleanreason?: string // when !oktxRefs?: ChainTxRef[] // chain references produced by this invocationexplorerUrls?: string[] // human-readable handles, parallel to txRefscontextDelta?: Record<string, unknown>// merged into SessionRecord.PhaseEntry.contextDeltaattestationRef?: AttestationRef // anchored evidence referenceerrorClass?: "permanent" | "transient" | "counterparty" | "substrate" | "settlement-atomicity"}
Conformance: phase handlers MUST accept a SessionContext and return a PhaseHandlerResult. On ok: true the orchestrator merges contextDelta into the corresponding PhaseEntry and records txRefs in the session event log; on ok: false the orchestrator classifies the failure per errorClass and applies the retry policy in chapter 10.

### 7.6 Closed registries — v1 scope

The v1 set of identity schemes (DACS-1), verification methods (DACS-2), negotiation patterns (DACS-3), payment phases (DACS-4), and delivery phases (DACS-4) are **closed**. New entries are added in subsequent minor versions of the relevant standard via the governance process in chapter 11. Implementations MAY support pre-standard "experimental" entries prefixed x-; these MUST be treated as unknown by conforming readers unless out-of-band agreement exists.

### 7.7 Universal signature scheme — domain-separated signing

Every signature in DACS — across DACS-1 (listings, revocations), DACS-2 (VerifyResults, composite records, recipes), DACS-3 (channel messages, agreements, commitments), DACS-4 (settlement evidence, amendments, rails, entitlements), and DACS-5 (bundles, ratings) — MUST be computed over a domain-separated payload. The domain separator prevents cross-protocol signature replay: a signature produced under one artifact kind MUST NOT validate as a signature under any other artifact kind, even when the underlying hash bytes coincide.
The canonical payload to be signed is:
signed_bytes := domain_separator || artifact_hashdomain_separator := "dacs-" || artifact_kind || ":v" || version_tag || ":"artifact_hash:= the sha256 hex of the RFC 8785 canonical form of thesigned document, with the signature field(s) omitted
The v1 registry of domain separators is closed:
| Artifact | Domain separator | Defined in |
| --- | --- | --- |
| DACS-1 listing | "dacs-listing:v1:" | §6.3.4 |
| DACS-1 listing revocation marker | "dacs-revocation:v1:" | §6.3.4 |
| DACS-1 identity bundle presentation | "dacs-bundle-presentation:v1:" | §6.3.2 |
| DACS-2 VerifyResult | "dacs-verifyresult:v1:" | §7.5 |
| DACS-2 composite verification record | "dacs-composite:v1:" | §7.7 |
| DACS-2 recipe | "dacs-recipe:v1:" | §7.4 |
| DACS-3 channel message | "dacs-channelmsg:v1:" | §8.3.3 |
| DACS-3 agreement | "dacs-agreement:v1:" | §8.5 |
| DACS-3 commitment record | "dacs-commitment:v1:" | §8.6 |
| DACS-3 channel transcript | "dacs-transcript:v1:" | §8.7 |
| DACS-4 settlement evidence | "dacs-evidence:v1:" | §9.7 |
| DACS-4 settlement amendment | "dacs-amendment:v1:" | §9.7.1 |
| DACS-4 rail definition | "dacs-rail:v1:" | §9.4 |
| DACS-4 entitlement record | "dacs-entitlement:v1:" | §9.6.2 |
| DACS-5 attestation bundle | "dacs-bundle:v1:" | §10.4.1 |
| DACS-5 rating record | "dacs-rating:v1:" | §10.6 |

**Conformance.** (SIG-1) Every signature in DACS v1 MUST be computed over the appropriate domain-separated payload from the table above. (SIG-2) Verifiers MUST reconstruct the domain separator and artifact hash independently and MUST NOT trust either supplied as-is by a counterparty. (SIG-3) Signatures whose payload computation cannot be reproduced exactly MUST be rejected. (SIG-4) An artifact kind not in the v1 table MUST use a domain separator of the form "dacs-x-" || kind || ":v" || version || ":" until accepted into a future version of the registry.
**Algorithm.** The signing algorithm itself (Ed25519, ECDSA-secp256k1, or sr1-aggregate) is independent of the domain-separation rule. The domain separator is prepended to the signed bytes regardless of algorithm. Implementations MUST NOT compute a signature over the artifact hash without the separator and MUST NOT compute a signature over the canonical form directly (always over the prepended-separator-then-hash payload).

## 8. Composed open standards

DACS composes with the following open standards. Each per-stage chapter cites the relevant entries by name; backwards-compatibility implications are stated per-stage.
| Standard | Composed by | Touchpoint |
| --- | --- | --- |
| ERC-8004 Trustless Agents | DACS-1, DACS-5 | Identity scheme; optional reputation publication surface |
| ERC-8183 Job Escrow (proposed) | DACS-4 | Future rail (v1.1) |
| W3C DIDs v1.0 | DACS-1 | Identity scheme |
| W3C Verifiable Credentials Data Model 2.0 | DACS-2 | verifiable-credential method |
| AP2 Agent Payments Protocol | DACS-4 | pay-ap2 rail envelope (FIDO Alliance custodian from April 2026) |
| x402 HTTP 402 revival | DACS-4 | pay-x402 rail envelope |
| A2A .well-known/agent.json | DACS-1 | Discovery extension |
| TLSNotary (PSE rebuild, 2024) | DACS-2 | tlsnotary method (distinct from native cci-tlsn context) |
| Reclaim Protocol / Pluto zkTLS | DACS-2 | zktls method |
| HTLC contracts (generic) | DACS-4 | pay-cross-chain-htlc; used by the reference implementation today |
| ERC-20 / SPL | DACS-4 | pay-evm-erc20 / pay-solana-spl |
| ACME / RFC 8555 | DACS-2 | domain-tls-control method |

## Chapter 6 — DACS-1: Identify
**Stage:** Identify (1st of 5). **Status:** Draft (part of DACS v0.1). **Depends on:** SR-1 (optional), SR-2 (required); composes with ERC-8004, W3C DIDs, A2A. **Used by:** DACS-2..5.

### 6.1 Abstract

DACS-1 specifies how an agent is identified, what it offers, and how it is found. It defines three artifacts and a discovery extension:
Identity is treated as a bundle of independently-verified claims rather than a single rooted identifier. This lets the same standard cover micropayments (a signing key is enough) and regulated trades (LEI + KYB + FINRA + OFAC) without changing structure. The substrate MUST provide anchored storage for listings and bundles (SR-2); single-signature bundle convenience (SR-1) is OPTIONAL and supplements but does not replace per-claim verification.

### 6.2 Motivation

The Identify stage answers three questions a buyer must resolve before any transaction proceeds: *who is the counterparty?* (cryptographically, at a level of confidence appropriate to the stakes); *what do they offer, on what terms?*; *how are they found in the first place when only the offering is known?*
Existing open standards address fragments. ERC-8004 provides an EVM-native agent identity (an NFT). W3C DIDs provide a methodology for self-sovereign identifiers. A2A’s .well-known/agent.json provides capability advertisement. Authority-issued identifiers (LEI, FINRA CRD, SAM.gov UEI) and platform identifiers (verified domains, OAuth-backed accounts) exist outside the standards space but carry real commercial weight. None of these — alone or combined — provide: a way to *declare* which set of claims a transaction requires; a way to *present* a matching set with per-claim verification references; a signed, anchored *listing* document that is itself the binding contract for the transaction; a *commercial discovery* surface distinct from capability advertisement.
DACS-1 fills these gaps with the smallest possible additions, composing with each existing standard where it fits. The stakes-graduated nature of agent commerce is the reason a single-rooted-identity model does not work: identity for a sub-cent API micropayment is a signing key; identity for a $500 B2B SaaS purchase is a signing key plus a small bundle of platform identities plus reputation; identity for a $50k institutional trade is an LEI as the identifier-of-record, with FINRA registration, OFAC clearance, and possibly a deal-specific KYB attestation. The same standard must work across all three.
The unification mechanism is **claims, not roots**. A listing requires a bundle of claims; a counterparty presents one. The bundle has a presentation signature (which MAY be a single SR-1 root signature, per-claim signatures, or a presentation signature by a session key). The rest of the stack consumes the bundle uniformly.

### 6.3 Specification

#### 6.3.1 Identity claim reference scheme

A claim reference identifies a fact about a party that can in principle be verified against an external system.
**Grammar**
A claim reference MUST conform to:
ClaimReference := Scheme ":" Identifier [ "?" Parameters ]Scheme := scheme-start ( scheme-cont )*scheme-start := lowercase-asciischeme-cont:= lowercase-ascii | digit | "-"Identifier := scheme-specific, non-empty, NFC-normalized Unicode (printable ASCII recommended)Parameters := key1=value1 [ "&" key2=value2 ]*
A Scheme MUST start with a lowercase ASCII letter and MAY include lowercase ASCII letters, digits, and hyphens thereafter. Underscores are reserved for future use and MUST NOT appear in v1 scheme names. Parsers MUST treat Scheme case-insensitively on read and SHOULD emit lowercase on write. Identifier is treated per-scheme; the per-scheme rules below specify canonicalisation. The ?<parameters> suffix carries scheme-specific qualifiers (e.g. cci-xm:evm:mainnet:0x…?jurisdiction=US). Unknown parameters MUST be ignored by readers, MUST NOT cause rejection, and MUST NOT be silently stripped when forwarding the reference.
**Registered schemes (v1) — two-axis registry**
The v1 scheme registry is organised along two axes: (a) **CCI-native** schemes — one per Demos CCI context, with the identifier directly addressing the relevant slot in GCRMain.identities; (b) **Stor-backed credential** schemes — schemes whose verification result is anchored as a Storage Program written by a DACS-2 attestation.
**CCI-native schemes** — map to Demos CCI contexts (8 in production today + 6 to-add for DACS-1 v1):
| Scheme | CCI context | Identifier shape | Status |
| --- | --- | --- | --- |
| cci-xm:<chain>:<subchain>:<address> | xm | per chain (EVM hex, Solana base58, …) | Done |
| cci-web2:<platform>:<username> | web2 | twitter / github / discord / telegram | Done |
| cci-pqc:<algorithm>:<pubkey> | pqc | falcon / ml-dsa | Done |
| cci-ud:<domain> | ud | Unstoppable Domain | Done |
| cci-nomis:<address> | nomis | Nomis wallet score subject | Done |
| cci-humanpassport:<id> | humanpassport | humanity proof id | Done |
| cci-ethos:<id> | ethos | Ethos profile id | Done |
| cci-tlsn:<proof-hash> | tlsn | TLSNotary proof commitment | Done — DACS-2 MUST treat as a CCI claim, NOT as an external tlsnotary method |
| cci-lei:<20-char> | lei (NEW) | uppercase LEI | GCR routine to build |
| cci-finra-crd:<digits> | finra-crd (NEW) | digits only, no leading zeros | GCR routine to build |
| cci-sam-uei:<12-char> | sam-uei (NEW) | uppercase UEI | GCR routine to build |
| cci-fedramp:<id> | fedramp (NEW) | as-issued | GCR routine to build |
| cci-naics:<6-digit> | naics (NEW) | digits only | GCR routine to build |
| cci-cmmc:<cert-id> | cmmc (NEW) | as-issued | GCR routine to build |

The six new contexts (lei, finra-crd, sam-uei, fedramp, naics, cmmc) extend the existing 8-context CCI model with the regulatory identity claims DACS-1 v1 needs. Each follows the same pattern as the existing 8 contexts: per-context GCR routine for validation; verified payload stored in GCRMain.identities; readable via the existing wallet/SDK identity surface.
**The DACS-1 / DACS-2 boundary for these claims.** DACS-1 is **registered identity** (what the party stably holds — LEI, FINRA registration, etc., kept in CCI). DACS-2 is **freshness check** (per-session re-verification that the registered claim is still valid right now). The DACS-2 DAHR call against the authority produces the verified result; that result is written into the relevant CCI context (DACS-1 surface) AND referenced from the DACS-2 CompositeVerificationRecord for that session.
**Stor-backed credential schemes (extensibility surface)**
For credentials Demos has not yet promoted to a native CCI context — future regulatory regimes, jurisdiction-specific identifiers, industry-specific certifications, ad-hoc one-off attestations — DACS-1 allows a Stor-backed scheme of the form:
stor-cred:<credential-type>:<identifier>
The Storage Program at stor-{sha256(subject_cci + ":" + credential-type + ":" + identifier)} holds the latest DACS-2 VerifyResult for that (subject, credential) tuple. This is the extensibility mechanism: when a new credential type is needed and there is no native CCI context for it, listings can require a stor-cred:*scheme without waiting for Demos to add a context. When a stor-cred:* scheme sees broad enough use, it SHOULD graduate to a native CCI context per the v2 scheme-addition process.
**Composition and low-stakes schemes**
| Scheme | Identifier shape | Use |
| --- | --- | --- |
| did:… | per W3C DID method | external decentralised identifier; resolution per method |
| erc8004:<chainId>:<contract>:<tokenId> | lowercase contract; decimal tokenId | external EVM agent identity NFT; verified via DACS-2 evm-rpc |
| domain:<dns> | lowercase, IDNA-encoded | DNS / TLS control proof via DACS-2 domain-tls-control |
| key:<hex-pubkey> | lowercase, no 0x | self-signed; lowest tier; signing-key only |

**Unknown-scheme handling**
A reader encountering an unknown scheme MUST: preserve the reference verbatim when forwarding; treat the reference as **unverified** for evaluation purposes; NOT silently accept the reference as satisfying a bundle requirement; log or surface the unknown scheme to the calling agent. A reader MAY decline to engage with a bundle that contains an unknown scheme in a required position.
**Adding new schemes (v2 and beyond)**
The v1 scheme registry is closed. New schemes are added in subsequent versions of DACS-1 by: submitting a scheme definition (name, identifier grammar, canonical form, authority, default DACS-2 verification recipe); demonstrating a working DACS-2 recipe; acceptance by the registry steward per the process in chapter 11. Implementations MAY support pre-standard "experimental" schemes prefixed x- (e.g., x-myorg-internal-id); these MUST be treated as unknown by conforming readers unless out-of-band agreement exists.

#### 6.3.2 Identity bundle

An identity bundle is an ordered set of claims a party presents about itself, with verification metadata, plus a presentation signature.
**Schema**
type IdentityBundle = {bundleVersion: "1"presentedBy: ClaimReference// primary identity claim within `claims`presentedAt: number// unix millisecondsclaims: BundleClaim[]// non-empty; order is meaningfulpresentation: PresentationSignature}type BundleClaim = {ref: ClaimReferenceverifiedBy?: VerifyResultRef // DACS-2 result referenceissuedAt?: number// unix ms when the verification was performedexpiresAt?: number // unix ms when verifiedBy becomes stalemetadata?: Record<string, unknown> // scheme-specific}type PresentationSignature =| { kind: "siwd"; message: string; signature: string; address: string }| { kind: "per-claim"; signatures: { ref: ClaimReference; signature: string }[] }| { kind: "session-key"; key: string; signature: string; rootBinding?: string }| { kind: "sr1-root"; rootClaim: ClaimReference; aggregateSignature: string }
SIWD is the preferred presentation. The siwd shape matches the return of provider.request({ method: "wallet_signIn", params: […] }) on the Demos wallet — { message, signature, address } — and is the same EIP-4361-style envelope. Verifiers MUST validate the SIWD signature against the bundle’s primary claim’s key.
**sr1-root presentation.** When SR-1 cross-substrate identity aggregation is available, a single root key may co-sign every claim in the bundle under an SR-1 aggregate signature, producing one signature that covers the whole bundle. rootClaim names the SR-1 root identity (a CCI primary claim on Demos); aggregateSignature is the SR-1 aggregate signature over the domain-separated payload (§6.3.2 below). Verifiers MUST resolve the root key via SR-1 and verify the aggregate signature against the bundle hash. sr1-root is the natural presentation for a party self-binding a single document (a seller signing their own listing, an orchestrator binding multiple per-substrate addresses under one identity) because it avoids the per-claim signature overhead and produces one cryptographic artifact that the rest of the stack can reason about.
**Domain-separated payload.** All four presentation kinds sign the same canonical payload:
signed_bytes := "dacs-bundle-presentation:v1:" || bundle_hash
For the siwd kind, the SIWD message string MUST include the canonical hex-encoded signed_bytes value as a "Resource" line per EIP-4361 so the SIWD signature transitively binds to the same payload (the message is the SIWD envelope, the resource carries the bundle binding). For the per-claim kind, each per-claim signature signs signed_bytes (not the raw bundle hash). For the session-key kind, the session key signs signed_bytes; if rootBinding is set, the root key’s binding signature signs "dacs-session-binding:v1:" || session_key || bundle_hash. For the sr1-root kind, the SR-1 aggregate signature signs signed_bytes; verifiers reconstruct the SR-1 aggregate from the rootClaim’s sub-identity set and verify against signed_bytes.
**Session nonce binding (RECOMMENDED).** Bundle presentations SHOULD include a presentedAt timestamp and a session-binding nonce when presented in the context of a specific session. The nonce is conveyed via the SIWD message’s Nonce field (per EIP-4361) or, for per-claim and session-key presentations, via a "sessionNonce" metadata field on the bundle. A verifier in a session context MUST check that the bundle’s sessionNonce (or SIWD Nonce) matches the session’s expected nonce. Bundles presented without session-nonce binding are usable only outside session contexts (e.g., listing publication where the bundle is the seller’s own self-binding to the listing).
**Canonical serialisation**
A bundle’s canonical form is the RFC 8785 JCS serialisation of the bundle with the presentation field omitted. The bundle hash is sha256(canonical_form), hex-encoded. The domain-separated signed_bytes (above) is what the presentation signature actually signs. Verifiers MUST recompute both the canonical form and the domain-separated payload when validating.
**presentedBy selection rule**
presentedBy MUST be one of the claim references appearing in claims (matching by canonical scheme and identifier). If the listing’s BundleRequirement.primaryClaimSelector is set, the presenter SHOULD select the highest-tier claim of the matching scheme. If no selector is set, the presenter SHOULD select the highest-tier claim available, where tiers (highest to lowest) are: authority-issued (lei, finra-crd, sam-uei, fedramp, cmmc, naics); ERC-8004 / W3C DID with verifiable proof; platform identifiers; plain signing keys. Readers MUST accept any presentedBy value that resolves to a claim in claims; a reader MAY prefer a higher-tier alternative for display or reputation lookup but MUST NOT reject a bundle solely because presentedBy is not the highest-tier claim.
**Verification reference resolution**
For a BundleClaim with verifiedBy present: the reader fetches the VerifyResultRef.anchor.locator from the indicated kind; hashes the fetched bytes and compares to VerifyResultRef.contentHash (mismatch MUST cause rejection); parses as a DACS-2 VerifyResult and verifies it matches the recipe at recipeVersion; checks that the VerifyResult.identifier matches the BundleClaim.ref identifier component canonically; checks that VerifyResult.decision == "pass". If any step fails, the claim is treated as unverified for evaluation against bundle requirements.
**Staleness**
A verifiedBy reference is stale when now > BundleClaim.expiresAt, or, when expiresAt is absent, when now − issuedAt > recipe.defaultMaxAgeSec. A stale verification MUST be refreshed during the Vet stage (DACS-2) for any claim that is required by the listing’s BundleRequirement.
**Conformance — bundles**
A conforming bundle producer MUST: (BP-1) produce JCS-canonical serialisation for hashing and signing; (BP-2) include at least one claim; (BP-3) provide presentedBy that resolves to a claim; (BP-4) provide a presentation signature that verifies against the bundle hash.
A conforming bundle reader MUST: (BR-1) recompute the bundle hash from canonical form before signature check; (BR-2) reject a bundle whose presentation signature does not verify; (BR-3) reject a bundle in which a required (per listing) claim has a missing or invalid verifiedBy when verificationRequired = true; (BR-4) treat claims with unknown schemes as unverified.

#### 6.3.3 Bundle requirement schema

A listing declares which bundles it will accept.
type BundleRequirement = {requirementVersion: "1"required: ClaimRequirement[] // all MUST be satisfiedoneOf?: ClaimRequirement[][] // at least one inner group MUST be satisfiedpreferredPresentation?: "sr1-root" | "per-claim" | "session-key" | "any"primaryClaimSelector?: string// scheme whose identifier MUST be `presentedBy`}type ClaimRequirement = {scheme: string // e.g. "lei"verificationRequired: booleanmaxAge?: number// seconds; overrides recipe defaultparameters?: Record<string, unknown> // scheme-specific}
**Matching algorithm**
A reader MUST evaluate a candidate IdentityBundle against a BundleRequirement using the following deterministic algorithm:
match(bundle, requirement):1. For each cr in requirement.required:if NOT find_claim(bundle, cr): return REJECT("missing required: <cr.scheme>")2. For each group in (requirement.oneOf or []):any_satisfied := falsefor each cr in group:if find_claim(bundle, cr): any_satisfied := true; breakif NOT any_satisfied: return REJECT("oneOf group unsatisfied")3. If requirement.primaryClaimSelector is set:if bundle.presentedBy.scheme != requirement.primaryClaimSelector: return REJECT4. If requirement.preferredPresentation is set AND != "any":if bundle.presentation.kind != requirement.preferredPresentation: return WARN, accept5. return ACCEPTfind_claim(bundle, cr):for each c in bundle.claims:if c.ref.scheme != cr.scheme: continueif cr.verificationRequired AND (c.verifiedBy missing OR resolution fails OR decision != "pass"): continueif cr.maxAge AND (now - c.issuedAt > cr.maxAge): continueif cr.parameters AND NOT scheme_specific_match(c, cr.parameters): continuereturn creturn null
scheme_specific_match is defined per scheme in DACS-2 recipes. Where parameters are unrecognised, readers MUST treat the requirement as unmatched (not silently passed).
**Failure mode and selector semantics**
A BundleRequirement that does not match MUST cause the buyer or seller to refuse to advance the transaction past the Vet stage. v1 specifies no downgrade or renegotiation path. The primaryClaimSelector controls which claim’s identifier is used as the reputation key in DACS-5 and the counterparty identifier of record for audit purposes. Listings that handle regulated flows SHOULD set primaryClaimSelector to an authority-issued scheme (e.g., lei) to ensure reputation accumulates against a stable, externally-verifiable identifier rather than a session key.

#### 6.3.4 Service listing

The listing is the canonical contract for a transaction.
**Schema**
type Listing = {// VersioningdacsVersion: "1"listingVersion: number // monotonic per listingId, starts at 1listingId: string// unique per seller; URL-safe ASCII; max 128 charsrequiredCapabilities?: SubstrateRequirement[]// Sellerseller: {identity: IdentityBundle // seller's own bundledisplayName: string// max 200 charspublicEndpoint?: string// optional HTTPS endpoint}// Offeringoffering: {title: string// max 200 charsdescription: string// max 2000 charscategory: string // dot-delimited (e.g. "data.finance.fx")tags: string[] // max 16 tags, max 32 chars eachdeliverable: DeliverableSpec // per DACS-4extendedDescriptionUrl?: stringextendedDescriptionHash?: string}// Buyer requirementbuyerRequirement: BundleRequirement// Pipeline of phases to execute, per DACS-3/4/5pipeline: PhaseStep[]// non-empty, ordered// Pricing and accepted rails, per DACS-4pricing: PricingSpecacceptedRails?: PaymentRailRef[]// OPTIONAL: required and non-empty IF pipeline contains any pay-* phase// Termsterms: ListingTerms// Validity windowvalidity: {notBefore: number// unix msnotAfter?: number// unix ms; absent => no expiry}// Listing-level signaturesignature: ListingSignature}type SubstrateRequirement =| "SR-1" | "SR-2" | "SR-3" | "SR-4" | "SR-5"type ListingTerms = {termsOfServiceUrl?: stringtermsOfServiceHash?: stringjurisdictions?: string[] // ISO 3166-1 alpha-2 codesconflictOfLawsRule?: "buyer-jurisdiction" | "seller-jurisdiction" | "rule-ref:<uri>"deadlineSecAfterCommit?: numbercancellationPolicy?: "none" | "pre-commit" | "with-fee"retentionYears?: numbertranscriptDisclosurePolicy?: "none" | "encrypted-anchored-recommended" | "encrypted-anchored-required"}type ListingSignature = {algorithm: "ed25519" | "ecdsa-secp256k1" | "sr1-aggregate"signer: ClaimReference // MUST appear in seller.identity.claimsvalue: string// signature over the canonical bundle hash}
DeliverableSpec, PricingSpec, and PaymentRailRef are normatively defined in chapter 9 (DACS-4). PhaseStep is defined below. A listing MUST use types that conform to the cited specs.
**PhaseStep schema**
type PhaseStep = {kind: PhaseType // closed v1 set belowparameters?: Record<string, unknown>// per-`kind` shape defined in the owning spec}type PhaseType =// DACS-2| "vet-credentials"// DACS-3| "negotiate-fixed-price" | "negotiate-rfq" | "negotiate-sealed-envelope" | "commit-agreement"// DACS-4| "pay-evm-erc20" | "pay-solana-spl"| "pay-cross-chain-htlc" | "pay-cross-chain-liquidity-tank"| "pay-ap2" | "pay-x402"| "deliver-storage-program" | "deliver-entitlement" | "deliver-attested-payload"// DACS-5| "rate"
Per-kind parameter shapes are normative in the owning chapter: vet-credentials — no parameters; negotiate-fixed-price — no parameters; negotiate-rfq — {maxTurns, timeoutSec, channelSubnet?} per chapter 8; negotiate-sealed-envelope — {commitDeadline, revealWindow, selectionRule, channelSubnet?} per chapter 8; commit-agreement — no parameters; pay-*— {rail: string} (railId) per chapter 9; deliver-* — no parameters (details come from the listing’s DeliverableSpec); rate — optional {required?: boolean} per chapter 10.
**Canonical serialisation and signature**
A listing’s canonical form is the RFC 8785 JCS serialisation with the signature field omitted. The listing hash is sha256(canonical_form), hex-encoded. The signature.value is computed over the domain-separated payload per chapter 7§7.7:
signed_bytes := "dacs-listing:v1:" || listing_hash
Verifiers MUST: recompute the canonical form, listing hash, and domain-separated signed bytes; resolve signature.signer to the corresponding key (via seller.identity.claims, then via DACS-2 verification if a verifiable identifier); verify the signature against signed_bytes. If signature.algorithm is sr1-aggregate, the signer’s IdentityBundle.presentation MUST be of kind sr1-root and the signature is the SR-1 root signature over signed_bytes (the SR-1 aggregate signature scheme applies to the same domain-separated payload, not directly to the listing hash).
**Anchoring and size limits**
A listing MUST be anchored using SR-2.
**Logical address vs native address.** DACS specifies a *logical* address pattern for each artifact kind. The logical pattern for a listing is dacs1:{sellerPrimaryClaim}:{listingId}:v{listingVersion}. The logical pattern is a stable, substrate-independent identifier the protocol reasons about; it is not necessarily the literal string the substrate accepts as an address. Each substrate-binding section specifies how the logical pattern maps to the substrate’s native addressing.
**Demos binding.** On Demos, the substrate’s StorageProgram addressing requires colon-free names and resolves writes to a sha256-derived handle of the form stor-<hex>. The Demos binding for a DACS listing therefore is:
logical_address := "dacs1:" + sellerPrimaryClaim + ":" + listingId + ":v" + listingVersionnative_address:= "stor-" + sha256(logical_address)
Implementations on Demos MUST anchor at native_address and MAY carry logical_address as descriptive metadata on the anchored record. Consumers resolve a listing by computing native_address from logical_address and reading the StorageProgram at native_address. Other substrates substitute their own native-address mapping; the requirement is that the mapping be deterministic, one-to-one, and reversible by any party knowing the logical pattern. The anchor transaction (the on-chain write) is the canonical pointer; the substrate’s native address is the addressable handle.
Substrates MAY use equivalent addressing schemes; the requirement is that any party with substrate access can dereference an anchor reference to the canonical content and verify the content hash.
**Size cap.** The canonical JSON form of a listing MUST NOT exceed 16,384 bytes (16 KB). Listings exceeding the cap MUST use the extendedDescriptionUrl + extendedDescriptionHash pattern to host verbose offering descriptions externally with content-hash binding. The cap applies after canonicalisation; the actual on-chain payload size may differ slightly due to substrate encoding. On substrates whose SR-2 implementation has a smaller per-record cap, the substrate cap governs (the lesser of 16 KB and the substrate cap). Implementations MUST reject listings exceeding the applicable cap at the validation step (LR-2).
**Versioning, immutability, revocation**
Each listingVersion is independently anchored. Prior versions MUST remain readable. A new version supersedes prior versions for new sessions; sessions already past commit-agreement (DACS-3) MUST continue against their pinned version. listingVersion MUST be monotonically increasing per listingId. Versions MUST NOT be skipped.
A seller MAY revoke a listing version by anchoring a revocation marker at the address dacs1-revoked:{sellerPrimaryClaim}:{listingId}:v{listingVersion} with value {revokedAt, reason?, signature} signed by the same key that signed the listing. Readers MUST check for the revocation marker before initiating a new session. Sessions already past commit-agreement MUST NOT be invalidated by revocation.
**Validation order for readers**
Readers MUST validate listings in the following order, halting on the first failure: (1) schema conformance; (2) dacsVersion supported; (3) validity.notBefore ≤ now ≤ validity.notAfter (if set); (4) canonical form well-formed and signature verifies; (5) revocation marker absent; (6) seller.identity bundle conformant per §6.3.2; (7) pipeline references valid phase types per DACS-3/4/5; (8) if pipeline contains any pay-*phase, acceptedRails MUST be present and non-empty and MUST reference resolvable payment rails per DACS-4; if pipeline contains no pay-* phase, acceptedRails MAY be absent (this is the intake-only listing pattern — RFP intake, reverse auctions where the bid is the commitment, free services gated by reputation, sealed-bid procurements settled out-of-band); (9) signer resolves to a key controllable by the seller.
**Conformance — listing publishers and readers**
A conforming publisher MUST: (LP-1) anchor each listingVersion via SR-2 before referencing it from a listing index; (LP-2) sign the listing with a key referenced by a claim in seller.identity.claims; (LP-3) use monotonic listingVersion values per listingId; (LP-4) publish revocation markers when withdrawing a listing.
A conforming reader MUST: (LR-1) pin the (listingId, listingVersion, contentHash) tuple into any session record derived from the listing; (LR-2) reject listings failing any step in the validation order; (LR-3) refuse new sessions against revoked listings.

#### 6.3.5 Discovery — .well-known/agent.json extension

The .well-known/agent.json document published at the agent’s domain is extended with a dacs block:
{// ... existing A2A agent-card fields ..."dacs": {"dacsVersion": "1","listings": {"indexUrl": "https://example.com/.well-known/dacs/listings.json","indexHash": "sha256-...","anchor": {"kind": "storage-program","address": "dacs1-index:..."}},"identityClaims": ["lei:984500ABCDEF12345678","domain:example.com","erc8004:1:0x...:42"]}}
**Listing index file (listings.json)**
type ListingIndex = {indexVersion: "1"generatedAt: numberseller: ClaimReferencelistings: ListingIndexEntry[]}type ListingIndexEntry = {listingId: stringversion: numbercontentHash: stringanchor: { kind: string; locator: string }summary: {title: stringcategory: stringtags: string[]priceHint?: string}status: "active" | "revoked"}
The index MAY itself be anchored via SR-2; if so, the well-known block’s anchor field MUST point to it. The indexHash field in the well-known block enables clients to detect stale caches. Clients MUST cross-check each ListingIndexEntry.anchor independently before engaging with a listing; the index is for discovery convenience, not a source of truth.
**Interoperability with A2A; update and revocation**
The dacs block is additive. A2A-only clients ignore the dacs field. DACS-aware clients use the dacs field for listing discovery; absence of the field MUST be interpreted as "this agent does not publish DACS listings via well-known" (the agent MAY still have listings discoverable via a catalog API). Sellers update by re-publishing listings.json with new entries and updated generatedAt; the well-known indexHash MUST be updated to match. Revocation removes the entry from the index AND publishes the on-chain revocation marker.

#### 6.3.6 Discovery — catalog API

A DACS catalog is an off-chain index aggregating listings across many sellers, providing search, filtering, and discovery.
**Endpoints**
GET /api/dacs/listingsQuery parameters:category=<dot-delimited prefix>tag=<repeatable>credential=<scheme># listings whose buyerRequirement requires this schemeprimaryClaim=<scheme># listings whose seller.identity.presentedBy uses this schemerail=<railId># listings accepting this railpriceMax=<decimal> # advisory; uses summary.priceHintcursor=<opaque># paginationlimit=<int, default 50, max 200>Response:{ "listings": ListingSummary[], "cursor": <opaque>?, "total"?: <int> }GET /api/dacs/listings/{listingId}/{version}Response: Listing (canonical JSON)GET /api/dacs/sellers/{primaryClaimRef}Response: {"listings": ListingSummary[],"identity": IdentityBundle (catalog-cached, last-seen),"reputation": ReputationSummary (per DACS-5)}
primaryClaimRef is URL-encoded canonical form (e.g., lei%3A984500ABCDEF12345678).
**ListingSummary, caching, authentication, cross-reference**
type ListingSummary = {listingId: stringversion: numbercontentHash: stringanchor: { kind: string; locator: string }seller: { primaryClaim: ClaimReference; displayName: string }offering: { title: string; category: string; tags: string[] }pricing: { priceHint?: string; currency?: string }status: "active" | "revoked"catalogObservedAt: number}
Catalogs MAY return cached ListingSummary records. Clients MUST dereference the anchor to obtain the canonical Listing before engaging. The catalog provides discovery; the chain provides binding. Catalogs SHOULD verify each indexed listing’s anchor at least every 24 hours; the catalogObservedAt timestamp surfaces the catalog’s confidence.
Read endpoints MUST NOT require authentication. Write/registration semantics are out of scope for v1; the canonical source of truth is always the substrate-anchored listing, not the catalog entry. For every ListingSummary returned, a DACS-aware client MUST resolve the anchor to the on-chain content and validate the contentHash. The catalog’s role is to surface candidates; binding decisions MUST follow the substrate.

#### 6.3.7 Conformance summary

| Role | Requirements |
| --- | --- |
| Listing publisher | LP-1 anchor; LP-2 sign; LP-3 monotonic versions; LP-4 publish revocation markers |
| Listing reader | LR-1 pin tuple; LR-2 validate per validation order; LR-3 refuse revoked |
| Bundle producer | BP-1 JCS canonical; BP-2 non-empty claims; BP-3 valid presentedBy; BP-4 valid presentation signature |
| Bundle reader | BR-1 recompute hash; BR-2 reject invalid signature; BR-3 reject missing required verifiedBy; BR-4 treat unknown schemes as unverified |
| Well-known publisher | Publish dacs block; keep indexHash current |
| Catalog operator | Open read endpoints; honour caching constraint; decline write endpoints by spec discretion |
| Catalog client | Dereference anchors before binding |

### 6.4 Rationale

**Identity-as-bundle vs single-rooted identifier.** A single-root model forces every listing to a single identity primitive. Either the primitive is weak enough for the lowest-stakes use case (a signing key, useless for institutional flows), or strong enough for the highest-stakes (an LEI, infeasible for $0.01 micropayments). The bundle model lets each listing declare its own minimum, and lets each counterparty present whatever set of claims they hold. Reputation is keyed against the *primary* claim of the bundle so that the same party accumulates separate reputation per tier — necessary to prevent a great signing-key reputation laundering into a brand-new LEI presentation.
**Closed scheme registry in v1 vs open.** An open registry from v1 produces fragmentation: parsers cannot validate bundles without runtime-loaded recipes; conformance becomes untestable. v1 ships a fixed set of schemes covering the high-volume cases (LEI, FINRA, SAM, OFAC, FedRAMP, plus self-sovereign and platform identifiers). New schemes ship with subsequent DACS-1 minor versions, via the steward’s acceptance process. x- experimental prefixes provide an escape valve for out-of-band agreement without diluting conformance.
**Listing as full JSON in anchored storage vs hash-only with off-chain content.** Full JSON anchoring ensures any party with substrate access can retrieve and verify the binding contract without dependency on off-chain availability. The trade-off is on-chain size; the normative 16 KB cap (§6.3.4) is chosen to keep anchoring cheap on every substrate while permitting verbose offering descriptions via the extendedDescriptionUrl + extendedDescriptionHash pattern. Listings whose essential terms cannot fit in 16 KB — multi-currency price tables, very large pipelines, complex deliverable specifications — are a v2 concern; v1 treats the cap as a forcing function toward listing simplicity.
**.well-known/agent.json extension vs separate discovery surface.** Extension preserves A2A interoperability and uses an existing pattern operators already deploy. A separate surface would require duplicate publishing and create discovery ambiguity. The dacs block is purely additive.
**Catalog API off-chain vs on-chain registry.** An on-chain catalog would centralise discovery as much as an operator catalog while being slower and more expensive. The chain holds the listings (source of truth); off-chain catalogs index for performance. Multiple competing catalogs can exist; clients dereference to chain for binding regardless of catalog choice.
**SR-1 optional vs required.** Requiring SR-1 would prevent DACS-1 implementations on substrates without cross-substrate identity aggregation, including most EVM chains. Optional SR-1 lets DACS-1 ship anywhere with anchored storage; SR-1 supplements with single-signature convenience where the substrate supports it.
**Per-claim verification references (verifiedBy).** A claim without a verification reference is a self-assertion; useful for low-stakes flows but not load-bearing for high-stakes ones. The verifiedBy reference is the load-bearing link: the rest of the stack (DACS-3 negotiation, DACS-5 audit) references *verifications*, not raw claims, when stakes matter.
**Cost model for anchored storage.** DACS-1 assumes the substrate’s anchored storage (SR-2) is economically viable for documents up to the soft size limit. On low-cost substrates (Demos, L2 rollups, IPFS+L1-anchored-hash) this is trivially true; on Ethereum L1 it would dominate listing economics. Implementations on high-cost substrates SHOULD use the extendedDescriptionUrl + extendedDescriptionHash pattern aggressively, anchor only the essential listing fields, and consider L2 / off-chain-with-hash patterns.

### 6.5 Backwards compatibility

**ERC-8004.** A DACS-1 listing’s seller.identity.claims MAY include a claim of scheme erc8004 referencing an Ethereum identity registry token. Verification follows the evm-rpc recipe in chapter 7 (DACS-2) — a proxy-attested EVM call confirming the token’s owner. The token’s reputation registry entries MAY additionally surface DACS-5 reputation derivations for EVM-side consumers, but DACS-1 does not require this.
**W3C Decentralized Identifiers (DIDs).** Claims of scheme did are W3C DIDs. Resolution follows the W3C DID method specification for the relevant method. The verification recipe varies by method: methods with cryptographic key material in the DID document resolve to a self-signed verification against that material; methods bound to verifiable credentials use verifiable-credential.
**A2A .well-known/agent.json.** The DACS extension is additive. A2A-only clients ignore the dacs block. A DACS-aware client that fetches .well-known/agent.json and finds no dacs block MUST NOT infer that the agent has no listings; it MAY fall back to a catalog API search.
**W3C Verifiable Credentials.** Bundle claims MAY have a verifiedBy reference whose backing DACS-2 method is verifiable-credential. The verification material is a W3C VC presented to the verifier; the verifier checks the VC signature, issuer, and freshness per the DACS-2 recipe.
**Future identity standards.** New identity schemes are added via the DACS-1 version process. DACS-1 is structured so adding a scheme requires only registry updates, not changes to the bundle, listing, or discovery schemas.

### 6.6 Security considerations

**Forged listings.** *Threat:* an attacker publishes a listing impersonating a known seller. *Mitigation:* listings are signed; the signer MUST be a key referenced in seller.identity.claims, and the bundle itself MUST verify. A reader following the validation order detects the impersonation at the signature step or the bundle-conformance step.
**Bundle replay across sessions.** *Threat:* an attacker captures a bundle from one session and replays it in another. *Mitigation:* the presentation signature is over the domain-separated payload "dacs-bundle-presentation:v1:" || bundle_hash, which the presenter generates fresh per session and which includes the session-binding nonce when presented in a session context (§6.3.2). Verifiers in a session context MUST validate the nonce; bundles missing the nonce in a session context MUST be rejected. Replay of an unverified bundle outside a session context is the equivalent of an unverified self-assertion and offers no advantage to the attacker.
**Catalog poisoning.** *Threat:* a catalog returns false listings or omits real ones. *Mitigation:* ListingSummary includes the anchor and contentHash; clients dereference and verify. A poisoned catalog causes UX confusion (a listing that does not exist on chain, or a missing listing) but cannot produce a verifiable false transaction.
**Claim-scheme spoofing.** *Threat:* a bundle includes a claim with a scheme the reader does not understand. *Mitigation:* unknown schemes MUST be treated as unverified. The reader cannot accept the claim as satisfying a required-and-verified bundle requirement.
**Identity-claim substitution between bundle presentation and Vet.** *Threat:* a counterparty presents bundle A in negotiation and bundle B at Vet time. *Mitigation:* the bundle hash is pinned into the session record at presentation time; DACS-2’s Vet stage operates on the pinned bundle. Substitution is detected by hash mismatch.
**Reading a listing after revocation.** *Threat:* a reader has cached a listing and engages without checking for revocation. *Mitigation:* readers MUST check the revocation marker before initiating a new session. Sessions already past commit-agreement are not invalidated by revocation, preserving in-flight obligations.
**Stale bundles in active sessions.** *Threat:* a session runs long enough that a verifiedBy reference becomes stale. *Mitigation:* DACS-2 specifies refresh semantics for required claims. For long-running entitlement sessions, listings SHOULD declare a refresh interval; v1 does not standardise this, deferring to DACS-2’s per-recipe defaults.
**Index integrity in .well-known.** *Threat:* a compromised web server publishes a falsified listings.json. *Mitigation:* the indexHash in the well-known block is signed only by the TLS certificate, not by the seller’s identity. Clients SHOULD prefer the index’s anchor (substrate-anchored copy) when available; in any case, individual listings MUST be dereferenced and validated independently.
**Private endpoints and impersonation.** *Threat:* seller.publicEndpoint claims a URL the seller does not control. *Mitigation:* this is a self-claim; readers MUST NOT treat the endpoint as authoritative for any cryptographic purpose. Endpoints are conveniences for off-chain reads, not trust anchors.
**Key lifecycle.** Every spec assumes a primary key exists per ClaimReference. Implementations MUST hold primary keys in a key-management system that does not retain plaintext at rest (HSM, TEE-backed enclave, or equivalent); support rotation (the relationship between a ClaimReference and its current key may change over time; the DACS-2 recipe for a scheme defines how key-current-ness is resolved); propagate revocation (publish a revocation marker for any listings the key signed, update bundle presentations to use a new key going forward); treat signatures produced by a key after its revocation timestamp as invalid for new sessions; sessions already past commit-agreement using the prior key remain bound (the obligation already exists).
## Chapter 7 — DACS-2: Vet
**Stage:** Vet (2nd of 5). **Status:** Draft (part of DACS v0.1). **Depends on:** SR-2 (required), SR-3 (required for consensus-backed-proxy and evm-rpc methods); composes with W3C VC, TLSNotary, zkTLS / Reclaim. **Used by:** DACS-1 (claim verification), DACS-3 (pre-negotiation gate), DACS-5 (audit references).

### 7.1 Abstract

DACS-2 specifies how a party’s claimed credentials are verified against authoritative sources during the Vet stage of a transaction. It defines:
Vet does three different jobs and produces one output. The output is the composite verification record. The rest of the transaction — and a future auditor, regulator, or arbitrator — references it.

### 7.2 Motivation

The Vet stage answers two questions a party must resolve before progressing past vet-credentials: *is the counterparty’s identity bundle valid right now?* (a bundle assembled months ago may contain claims whose underlying registrations have lapsed, sanctions lists have updated, or certifications have expired); *is the bundle sufficient for this specific deal?* (the listing may require claims the bundle does not contain because they are deal-specific and were not pre-attested).
A naive design would treat Vet as a single one-shot credential check at session start. That fails for three reasons: **freshness** (existing standards produce attestations at issuance time; Vet must re-check at session start, against current authority state, and produce a current attestation that supersedes the stale one in the bundle); **supplementary signals** (reputation, completion history, dispute rates, and prior counterparty ratings inform whether to proceed even when every required claim is technically valid — these are not credentials in the formal sense but materially affect the decision); **deal-specific claims** (some claims exist only for this transaction — insurance binding for a particular shipment; clearance for a particular project — and are not pre-attested because they are not reused).
A second design failure: treating each verification method as its own protocol. Without a uniform consuming shape, every downstream component needs per-method handling. DACS-2 introduces the VerifyResult as the lingua franca: methods produce it, the rest of the stack consumes it.
A third design failure: forcing one method to handle all credentials. Authority-issued credentials in public registries (LEI, FINRA, SAM.gov, OFAC) do not cooperate with W3C VC issuance; their verification fits a consensus-backed proxy attestation pattern (SR-3). Private-data credentials (KYC tier, balance proofs) fit TLSNotary / zkTLS. Cooperative-issuer credentials fit W3C VC. The recipe registry routes each claim scheme to the appropriate method, and the rest of the stack stays method-agnostic.

### 7.3 Verification methods (v1 closed registry)

The v1 method set is closed. New methods are added in subsequent versions of DACS-2 by the governance process in chapter 11.

#### 7.3.1 Common contract

Every method MUST: (CM-1) accept inputs as specified in its sub-section; (CM-2) anchor its attestation via SR-2 at an address derived from the session id, claim scheme, and identifier (dacs2:{jobId}:{scheme}:{identifier}:v{recipeVersion} or substrate-equivalent); (CM-3) produce a VerifyResult conforming to §7.5; (CM-4) classify its outcome as exactly one of pass, fail, indeterminate, or error per the semantics in §7.5.1; (CM-5) set VerifyResult.method to its own kind.

#### 7.3.2 verifiable-credential

type VCMethodInput = {recipe: Recipeidentifier: string // claim identifier (must match VC subject)presentation: VerifiablePresentationissuerAllowList?: ClaimReference[]}
**Procedure.** Verifier parses the presentation; verifies the VC signature against the issuer key (resolved per VC method); if issuerAllowList is set, MUST reject if the VC issuer is not in the list; verifies the VC has not expired and is not revoked (via status list, if present); verifies the VC’s subject identifier matches the claim’s identifier canonically; anchors the VC (or its hash, if VC is private) via SR-2; extracts structured data per recipe.parserRules; returns VerifyResult with pass if all steps succeed, fail on signature/expiry/revocation failures, indeterminate on parser errors. **Trust model:** issuer; W3C VC spec; key resolution method (e.g. did:web). **Substrate:** SR-2.

#### 7.3.3 tlsnotary

type TLSNotaryMethodInput = {recipe: Recipeidentifier: stringproof: TLSNotaryProof// MPC-derived TLS session commitment + notary signaturesessionTemplate?: string}
**Procedure.** Validates the TLSNotary proof per the TLSNotary specification (current PSE rebuild); verifies the notary signature against a known notary public key registry; if sessionTemplate is set, verifies the proof targets that endpoint and protocol; anchors the proof commitment via SR-2; applies parser rules to any disclosed segments; returns VerifyResult per outcome. **Trust model:** TLS PKI; notary honesty; MPC computational assumptions. **Substrate:** SR-2.

#### 7.3.4 zktls

type ZKTLSMethodInput = {recipe: Recipeidentifier: stringprovider: "reclaim" | "pluto" | stringprogramId: stringproof: ZKTLSProof}
**Procedure.** Loads the verifier circuit for provider:programId; verifies the zk proof per the circuit’s verification algorithm; verifies any public inputs match expected values; anchors the proof + public-input hash via SR-2; applies parser rules to public-input disclosed data; returns VerifyResult per outcome. **Trust model:** zk soundness; provider’s circuit correctness; TLS PKI; proxy honesty (provider-dependent). **Substrate:** SR-2.

#### 7.3.5 consensus-backed-proxy

type ConsensusProxyMethodInput = {recipe: Recipeidentifier: stringendpoint: {method: "GET" | "POST"urlTemplate: string// e.g. "https://api.gleif.org/api/v1/lei-records/{identifier}"headers?: Record<string, string>body?: string}}
**Procedure.** Renders the URL by substituting {identifier} and other recipe parameters into urlTemplate; submits the fetch specification to the substrate’s SR-3 primitive (on Demos: dahr.startProxy({url, method, options})); the substrate returns the response body inline plus chain-anchored commitment hashes (responseHash, responseHeadersHash) via a one-tx web2Request. Anchors the commitment record via SR-2; applies parser rules to the response body; if recipe.negativeMatch is true (OFAC pattern): decision = "pass" when the parser finds no match, "fail" when a match is found; otherwise pass when the parser matches expected success criteria.
**Trust model in v1 — consensus-anchored hash commitment.** The substrate validator set collectively signs the on-chain transaction that asserts "we fetched URL X at time T and obtained a response with hash H." The full response body is **not** independently signed by the validator set in v1; the chain-level guarantee is "this hash came from this URL at this time," not "this body content is validator-attested." Verifiers consuming the body MUST verify the body’s hash matches the on-chain commitment.
**Trust caveats v1 implementations MUST surface to consumers.** (a) A validator-set majority that colludes can sign a commitment to a forged response; the body the consumer reads will hash to the committed value, so consumers cannot detect this from the commitment alone. (b) A single validator fetching the response and the rest signing the resulting hash is operationally indistinguishable from a full-fanout fetch under current SR-3 specs; recipes used for high-stakes verification SHOULD set multi-method alternatives (see §7.12). (c) The TLS connection between substrate validators and the authority is the trust floor for response authenticity at fetch time; an attacker who can MITM the authority’s TLS endpoint can cause all validators to commit to forged content.
**v1.1 strengthening (planned).** A future minor version is expected to specify a "validator-body-signed" mode in which each validator independently signs the response body bytes and the aggregate signature is anchored. When this mode ships, recipes for high-stakes schemes (lei, finra-crd, ofac-clear, sam-uei) SHOULD migrate to require it. v1 recipes that already declare alternatives are forward-compatible.
**Substrate:** SR-3 (proxy attestation primitive); SR-2 (anchoring).
**Note on the tlsn CCI context.** TLSNotary proofs are a native CCI context on Demos (cci-tlsn:<proof-hash>), validated by Demos’s GCR routines. When the proof is already registered as a cci-tlsn claim in the counterparty’s bundle, Vet treats it as a CCI-native verification (the tlsn GCR routine has already validated it) and does NOT re-run the external tlsnotary method. The external tlsnotary method applies only when an *unregistered* proof is presented at session time.

#### 7.3.6 oauth-attested

type OAuthAttestedMethodInput = {recipe: Recipeidentifier: stringprovider: string // e.g. "google", "github"scopes: string[]maxTokenAgeSec: number // recipe-requiredattestation: OAuthAttestationEnvelope}
**Procedure.** Validates the attestation envelope’s signature; verifies the attestation references an OAuth flow that resolved to the claimed identifier (e.g. the sub claim from a Google ID token matches); verifies the granted scopes include those required by the recipe; anchors the attestation via SR-2; returns VerifyResult. **Trust model:** OAuth provider; attestation service honesty; TLS PKI. **Substrate:** SR-2 (SR-3 if the attestation service is the substrate’s API Verification primitive).

#### 7.3.7 evm-rpc

type EVMRPCMethodInput = {recipe: Recipeidentifier: stringchainId: numbercontract: string // 0x addressmethod: string // contract method name or selectorargs?: unknown[] // ABI-encoded arguments}
**Procedure.** Submits a proxy-attested EVM call via SR-3 (typically wrapped as a JSON-RPC fetch); substrate validator set executes the call and signs the result; anchors the signed result via SR-2; decodes the return value per recipe parser rules; compares the decoded value to the claim’s identifier (e.g., owner address of an ERC-8004 token); returns VerifyResult. **Trust model:** substrate consensus; EVM chain finality. **Substrate:** SR-3; SR-2.

#### 7.3.8 domain-tls-control

type DomainTLSControlMethodInput = {recipe: Recipeidentifier: string // the domainchallengeType: "http-01" | "dns-01" | "tls-alpn-01"challenge: Challengeresponse: ChallengeResponse}
**Procedure.** Validates the challenge/response per the ACME-style challenge specification; confirms response was retrievable from the claimed domain at the time of the challenge; anchors the challenge/response transcript via SR-2; returns VerifyResult (pass on valid response, fail on invalid, indeterminate on retrieval failure). **Trust model:** DNS / TLS PKI; ACME challenge integrity. **Substrate:** SR-2.

#### 7.3.9 self-signed

type SelfSignedMethodInput = {recipe: Recipeidentifier: string // hex public keysignature: string// signature over the claim assertionassertion: string// canonical bytes signed}
**Procedure.** Validates the signature against the key in identifier; anchors the signed assertion via SR-2; returns VerifyResult with pass on valid signature, fail on invalid. This method provides minimal trust — it proves possession of the key, nothing more. Recipes targeting authority-issued schemes MUST NOT use self-signed. **Trust model:** cryptographic signature. **Substrate:** SR-2.

### 7.4 Recipe registry

A recipe binds a DACS-1 claim scheme to a verification method (or set of acceptable methods) plus parsing rules and defaults.

#### 7.4.1 Recipe schema

type Recipe = {recipeVersion: number // monotonic per scheme; starts at 1scheme: stringdefaultMethod: VerificationMethodalternatives?: VerificationMethod[]defaultMaxAgeSec: number// when a verifiedBy reference becomes staleparserRules: ParserSpecnegativeMatch?: boolean // true => presence in source means failretryClass: "transient" | "permanent"availability: RecipeAvailability// operational status (see §7.4.5)governance: {proposedBy: ClaimReferenceacceptedAt: numbersupersedes?: number}signature: RecipeSignature// steward's signature (see §7.4.4)}type RecipeAvailability =| "live"// authority endpoint reachable, attestation path runs end-to-end| "operator_gated"// requires per-operator credential, key, or whitelisting| "closed_data" // authority data not publicly accessible (e.g., paid feed, internal source)| "bilateral" // requires per-relationship agreement between parties| "mocked"// attestation path stubbed; not a production verification| "disabled"// recipe present but the steward has marked it not-for-use| "failed"// recipe's underlying authority is currently broken or unreachabletype VerificationMethod =| { kind: "verifiable-credential"; issuerAllowList?: ClaimReference[]; schemaUrl?: string }| { kind: "tlsnotary"; endpoint: string; sessionTemplate?: string }| { kind: "zktls"; provider: "reclaim" | "pluto" | string; programId: string }| { kind: "consensus-backed-proxy"; endpoint: { method: "GET" | "POST"; urlTemplate: string; headers?: Record<string, string>; body?: string } }| { kind: "oauth-attested"; provider: string; scopes: string[] }| { kind: "evm-rpc"; chainId: number; contract: string; method: string; args?: unknown[] }| { kind: "domain-tls-control"; challengeType: "http-01" | "dns-01" | "tls-alpn-01" }| { kind: "self-signed" }type ParserSpec =| { format: "json"; successJsonPath: string; dataMap?: Record<string, string> }| { format: "html"; successSelector: string; dataMap?: Record<string, string> }| { format: "xml"; successXPath: string; dataMap?: Record<string, string> }| { format: "raw"; matcher: string }

#### 7.4.2 v1 recipe registry contents

The v1 registry contains one recipe per scheme registered in chapter 6. Each recipe is anchored via SR-2 at a steward-controlled address. Implementations MUST resolve recipes from the canonical addresses listed in the recipe-registry index document (dacs2:registry:v1). Default methods per scheme:
| Scheme | Default method | Notes |
| --- | --- | --- |
| key | self-signed | Lowest trust; session continuity only |
| did | verifiable-credential (DID-bound) or self-signed (key-bound) | Per DID method |
| erc8004 | evm-rpc | Reads token owner |
| lei | consensus-backed-proxy against api.gleif.org/api/v1/lei-records/{identifier} | JSON parser; GLEIF JSON-API |
| finra-crd | consensus-backed-proxy against api.brokercheck.finra.org/search/individual/{identifier} | JSON; FINRA BrokerCheck JSON API (not the HTML site) |
| sam-uei | consensus-backed-proxy against api.sam.gov/entity-information/v3/entities?ueiSAM={identifier} | JSON; SAM.gov Entity API |
| ofac-clear | consensus-backed-proxy against sanctionslistservice.ofac.treas.gov/api/PublicationPreview/exports/SDN.XML | XML; full SDN List download; negativeMatch: true; parser searches for {identifier} entity |
| fedramp | consensus-backed-proxy against marketplace.fedramp.gov/api/v1/products/{identifier} | JSON; FedRAMP Marketplace API |
| naics | consensus-backed-proxy against api.census.gov/data/2017/cbp?get=NAICS2017&NAICS2017={identifier} | JSON; US Census API for NAICS validation |
| cmmc | verifiable-credential preferred; consensus-backed-proxy fallback against cmmcab.org public registry endpoint | CMMC AB publishes both VC issuance and a registry |
| domain | domain-tls-control | ACME-style |
| platform:<provider> | oauth-attested | Provider-specific |
| stripe-connect | oauth-attested (provider="stripe") | Stripe-specific scopes |

**Authority API stability.** The endpoints above are the canonical structured-data endpoints offered by each authority as of v1 publication. Authority APIs change; recipes MUST be re-anchored when endpoint URLs or response formats change materially. See §7.4.4 for the recipe-track governance that makes this operational.

#### 7.4.3 Recipe authoring and resolution

A conforming recipe author MUST: (RA-1) sign the recipe with the registry steward’s signing key over the domain-separated payload "dacs-recipe:v1:" || recipe_hash per chapter 7§7.7; (RA-2) anchor the recipe via SR-2 at the canonical address; (RA-3) specify recipeVersion as monotonically increasing per scheme; (RA-4) specify supersedes when replacing a prior recipe for the same scheme; (RA-5) provide at least one alternative method only if the scheme’s underlying authority supports multiple equivalent attestation paths.
A verifier MUST resolve a recipe by: reading the recipe-registry index from dacs2:registry:v1; looking up the entry for the claim’s scheme; fetching the recipe at the indicated anchor and verifying its content hash and domain-separated signature; if a listing pins a specific recipeVersion, MUST use that version, otherwise MUST use the latest at session start, pinned into the session.

#### 7.4.4 Recipe-track lifecycle and current steward

**Recipes are operational artifacts on a different lifecycle than the DACS-2 standard itself.** Authority API endpoints change, response formats evolve, sanctions lists update, OAuth provider scopes shift. Treating every recipe revision as a DACS-2 minor version would force the standard onto an impractical release cadence. v1 separates the two tracks:
**Current steward (v1).** The DACS-2 recipe registry is currently maintained by **KyneSys Labs** as the v1 steward. This is a single-signer arrangement (phase PA-2 per the progressive-anchoring scheme below). Wider governance — working-group constitution, multi-signature schemes, sub-authority delegation by domain (sanctions lists, financial regulation, etc.) — is open work for v1.1+ and depends on the eventual constitution of a multi-party body. v1 implementations and consumers reason about the registry under single-steward semantics: one signing key, one anchoring authority, full transparency about both.
**Emergency recipe updates.** When an authority endpoint becomes unavailable or returns materially-incompatible data, the steward MAY publish an emergency recipe revision. Emergency revisions MUST: be signed normally; include an emergency: true field in the governance block; cite the failure observation (URL of authority change announcement, observed response format diff). Emergency revisions take effect at next session start; in-flight sessions continue against pinned recipeVersion.
**Recipe deprecation.** A recipe MAY be marked deprecated by publishing a new revision with deprecated: true and a deprecationReason. Verifiers MUST NOT initiate new sessions using deprecated recipes for required claims; in-flight sessions continue. A deprecated recipe with no replacement leaves the scheme un-verifiable; this is a v1.1 strengthening target for any scheme that hits this condition.
**Progressive anchoring phases.** Recipe anchoring proceeds through three phases. (PA-1) **Bootstrap phase.** Implementations MAY ship recipes as in-code constants or static configuration. Recipes in this phase MUST be marked anchoring: "in-code" and MUST NOT be presented as canonically anchored. (PA-2) **Single-steward phase.** The steward (currently KyneSys Labs) anchors recipes at the canonical address under a single signature, marked anchoring: "single-signer" and disclosing the steward’s identity. This is the current operating phase for v1. (PA-3) **Constituted phase.** If and when a multi-party governance body is constituted, recipes anchor under that body’s multi-signature scheme; prior single-signer recipes are re-anchored under the constituted signature. Implementations MUST clearly disclose which phase they operate in; consumers reading a VerifyResult MUST verify the recipe’s anchoring phase against their own trust requirements.

#### 7.4.5 Recipe availability (normative)

Every Recipe MUST declare an availability value. The value names the recipe’s current operational status — what a verifier should actually expect when it tries to run this attestation path. The values are deliberately discrete and disjoint so consumers can compose them programmatically without flattening operational reality into a single boolean.
| Value | Meaning |
| --- | --- |
| live | Authority endpoint reachable today; the attestation path runs end-to-end against a public or near-public source. The normal operating state for a recipe consumers expect to "just work". |
| operator_gated | Recipe technically functions but requires per-operator setup before it can run: API credentials, IP whitelisting, paid subscription, registered relationship with the authority. Consumers can’t rely on it working without operator-side configuration. |
| closed_data | Authority data not publicly accessible — paid feed, proprietary database, regulator-only API. Recipe shape is defined for forward compatibility but the path cannot run from an open verifier. |
| bilateral | Recipe runs only between parties with a pre-existing bilateral agreement (custom trust anchor, shared secret, contracted attestation service). The shape exists so DACS-2 can describe such verifications uniformly; the recipe doesn’t function for general open-network use. |
| mocked | Attestation path is stubbed for development, testing, or honest-scope visibility. MUST NOT be presented as a production verification path. Implementations MUST surface this state to consumers (e.g., via the "dahr-stub:" prefix marker on attestations). |
| disabled | Recipe exists but the steward has marked it not-for-use, typically because a successor recipe exists or the underlying scheme is being retired. Verifiers MUST NOT initiate new sessions using disabled recipes; in-flight sessions continue. |
| failed | Recipe’s underlying authority is currently broken (endpoint down, response format changed in a way the parser cannot consume, certificate expired). Operationally indistinguishable from "live but unreachable" until the steward publishes an emergency revision or marks the recipe disabled. |

**Consumer obligations.** Verifiers MUST inspect availability before running the recipe. (RAV-1) A verifier MUST NOT silently treat operator_gated, closed_data, bilateral, mocked, disabled, or failed as live. (RAV-2) A verifier presented with a VerifyResult produced under a non-live availability MUST surface the availability value to the verifier’s consumer; a UI flattening seven states into "verified" / "not verified" without disclosing availability is non-conformant. (RAV-3) Aggregation under §7.7.1 treats VerifyResults from disabled or failed recipes as decision = "error" regardless of the underlying authority response (the recipe is non-operational; any output is unreliable). (RAV-4) The availability of an alternative method does not override the availability of the default; consumers selecting an alternative MUST honour that alternative’s own availability.
**Steward obligations.** The current steward (§7.4.4 and §11.1.1) MUST keep availability values current. (RAV-5) Discovery that an authority endpoint has gone down requires either an emergency revision (§7.4.4) or a recipe revision setting availability to failed within a reasonable window. (RAV-6) Transitions from live → failed and failed → live are themselves recipe revisions and MUST be signed and anchored normally. (RAV-7) Availability is per-recipe-version, not per-scheme; a v3 recipe MAY be live while a v2 recipe is disabled for the same scheme.
**Why this is normative, not informative.** Earlier drafts carried this distinction in the front-matter production-mapping legend as informative iconography (🟢 / 🟡 / 🔵). That framing flattened distinct operational states (live, operator-gated, closed-data, mocked, etc.) into a single "wired" status and pushed disambiguation onto every implementer’s UI surface. Promoting availability to a normative field on the recipe itself moves the disambiguation into the protocol layer, where verifiers can reason about it programmatically and consumers can rely on conformant disclosure. This change was proposed by PATH-OS Labs (third-party reviewer; §11.3) and accepted into v1.

### 7.5 VerifyResult

The uniform record every method produces.
type VerifyResult = {resultVersion: "1"scheme: string// DACS-1 claim schemeidentifier: string// canonical identifier verifiedrecipeVersion: numbermethod: VerificationMethod["kind"]decision: "pass" | "fail" | "indeterminate" | "error"reason: string// briefattestation: AttestationRefdata?: Record<string, unknown>// structured extraction per recipe.parserRulesfetchedAt: number // unix ms when the authority was queriedverifiedAt: number// unix ms when the result was finalisedvalidUntil?: numbersignature: VerifyResultSignature}type AttestationRef = {anchor: { kind: "storage-program" | "ipfs" | "https"; locator: string }contentHash: stringsigner?: ClaimReference // for VC: issuer; for proxy: substrate validator-set claim (see below)}
Canonical form is RFC 8785 JCS of the VerifyResult with the signature field omitted. The VerifyResult hash is sha256(canonical_form), hex-encoded. The signature is computed over the domain-separated payload per chapter 7§7.7:
signed_bytes := "dacs-verifyresult:v1:" || verifyresult_hash
**Validator-set claim references.** When AttestationRef.signer designates the producer of a consensus-backed-proxy or evm-rpc attestation, the ClaimReference MUST use the substrate-validator-set scheme: substrate-validator-set:<substrateId>:<epochOrSetId>. <substrateId> is a registered substrate identifier (v1 registry: "demos-mainnet", "demos-testnet"; future substrates added by the registry steward). <epochOrSetId> identifies the specific validator set that signed the attestation (Demos uses epoch numbers; substrates using rotating sets use whatever identifier the substrate exposes). Consumers MUST resolve the validator-set reference to the substrate’s published validator-set roster for that epoch and verify the attestation signature against the aggregate of those validators’ keys per the substrate’s consensus protocol. Substrates whose validator-set rosters are not publicly resolvable MUST NOT be used as the signer of a VerifyResult intended for cross-substrate consumption.

#### 7.5.1 Indeterminate semantics

#### 7.5.1 Decision values and semantics

The four decision values are not interchangeable. Each has distinct semantics that govern retry behaviour, aggregation, and consumer interpretation:
**Retry semantics.** (a) error MUST be treated as transient by default; the verifier MAY retry per recipe.retryClass (§7.7 below). (b) indeterminate MUST NOT be retried unless the recipe explicitly marks the method as retry-on-indeterminate (rare; reserved for authorities whose "pending" responses become conclusive on re-fetch). The authority’s indeterminate answer is the answer; re-asking does not change it. (c) pass and fail are terminal; no retry.
**Aggregation semantics.** A required claim with overall result error or indeterminate after retry budget exhaustion MUST cause vet-credentials to fail the phase. Consumers MUST NOT treat any of indeterminate, error, or fail as pass under any circumstances. Aggregation logic (§7.7.1) distinguishes the three non-pass outcomes in its failure reasons so downstream consumers (dispute, audit, debugging) can determine whether verification reached the authority at all.
**Why distinguish indeterminate from error.** Both produce non-pass outcomes, but the diagnostic value differs significantly. error means "we should try again or change verification path." indeterminate means "the authority answered, and the answer is not yes or no — escalate to a different authority or accept the ambiguity." Collapsing them loses information that consumers need.

#### 7.5.2 Attestation resolution algorithm

A consumer of a VerifyResult MUST validate the attestation by: fetching the anchor at AttestationRef.anchor.locator; hashing the fetched bytes and comparing to AttestationRef.contentHash (mismatch MUST cause rejection); for methods with signer, validating the attestation signature against the signer’s known key (mismatch MUST cause rejection); optionally parsing the attestation to independently re-derive the structured data (for high-stakes verifications).

### 7.6 Verification procedure

The verifier MUST execute each claim verification by: (1) resolving the recipe and pinning recipeVersion to the session record; (2) rendering method inputs by substituting the claim’s identifier into the method’s template; (3) invoking the method (calling the appropriate substrate primitive for SR-3 methods, or external service); (4) receiving the attestation (raw bytes or a reference if too large for inline transport); (5) anchoring the attestation via SR-2 at the derived address; (6) parsing the response by applying recipe.parserRules to extract structured data into VerifyResult.data; (7) applying parameters — checking the listing’s ClaimRequirement.parameters against extracted data (failure to match MUST set decision = "fail"); (8) signing and emitting the VerifyResult.

#### 7.6.1 Retry and caching semantics

(VP-R1) On decision = "error" with recipe.retryClass == "transient", the verifier MAY retry up to a recipe-defined retry budget (default: 3 attempts, exponential backoff). (VP-R2) A retry MUST produce a new attestation; reusing the prior attestation is not a retry. (VP-R3) On recipe.retryClass == "permanent", the verifier MUST NOT retry within the same session; the failure is final for that session. (VP-R4) On decision = "indeterminate", the verifier MUST NOT retry unless recipe.retryOnIndeterminate is explicitly true (default false). The authority’s indeterminate answer is itself the answer; re-asking does not change it. The retryOnIndeterminate flag is reserved for authorities whose "pending" or "queued" responses become conclusive on re-fetch.
(VP-C1) A VerifyResult for (scheme, identifier, recipeVersion) MAY be reused within the recipe’s defaultMaxAgeSec (or the result’s validUntil). (VP-C2) Reuse MUST update the consuming session’s record to reference the cached VerifyResult. (VP-C3) Reuse MUST NOT bypass freshness requirements declared by the listing’s ClaimRequirement.maxAge.

### 7.7 Composite verification record

The document the vet-credentials phase produces.
type CompositeVerificationRecord = {recordVersion: "1"jobId: string // DACS-5 session idevaluatedParty: ClaimReference// counterparty's primary identity claimbundleHash: string// sha256 of the IdentityBundle this Vet ran againstrequirementHash: string // sha256 of the listing's BundleRequirementfreshness: VerifyResultRef[]// re-verifications of pre-attested claimssupplementary: SupplementarySignal[]dealSpecific: VerifyResultRef[]overallDecision: "pass" | "fail" | "indeterminate" | "error"generatedAt: numbersignature: ComponentSignature // signed by the verifier}type VerifyResultRef = {anchor: { kind: "storage-program" | "ipfs" | "https"; locator: string }contentHash: stringrecipeVersion: number}type SupplementarySignal = {source: "dacs-5" | "cci-nomis" | "cci-ethos" | "cci-humanpassport" | "external" | stringsignalType: string// e.g. "completion-rate", "dispute-rate", "rating-avg"value: number | stringobservedAt: numberattestation?: AttestationRef// required for "external" sources}
CCI-native reputation signals (cci-nomis, cci-ethos, cci-humanpassport) are first-class supplementary signal sources: they are read from the counterparty’s CCI without needing a separate attestation, because the underlying CCI context’s GCR routine has already validated them.

#### 7.7.1 Aggregation algorithm

A verifier MUST compute overallDecision per the following algorithm. The algorithm distinguishes four cases for each required claim: passing, indeterminate (authority answered ambiguously), errored (verifier could not reach the authority), and failing/absent. Precedence among non-pass outcomes is failures > errors > indeterminates so that the strongest evidence dominates aggregation.
aggregate(record, requirement):failures := []errors := []indeterminates := []# All required claims must have a passing VerifyResultfor cr in requirement.required:classify_required(record, cr, failures, errors, indeterminates)# oneOf groups must each contain at least one passingfor group in requirement.oneOf:if not any(find_passing(record, cr.scheme) for cr in group):# Distinguish indeterminate vs error vs hard miss for the groupif any(find_indeterminate(record, cr.scheme) for cr in group):indeterminates.append("oneOf group: at least one claim indeterminate")else if any(find_error(record, cr.scheme) for cr in group):errors.append("oneOf group: at least one claim errored")else:failures.append("oneOf group: no claim satisfied")# Precedence: failures > errors > indeterminatesif failures: return "fail", failuresif errors: return "error", errorsif indeterminates: return "indeterminate", indeterminatesreturn "pass", []classify_required(record, cr, failures, errors, indeterminates):results := find_all_results(record, cr.scheme) // freshness ++ dealSpecificif results is empty:failures.append("required not present: " + cr.scheme)returnif any(r.decision == "pass" for r in results):return// claim satisfiedif any(r.decision == "fail" for r in results):failures.append("required failing: " + cr.scheme)returnif any(r.decision == "indeterminate" for r in results):indeterminates.append("required indeterminate: " + cr.scheme)return// all results are "error"errors.append("required errored: " + cr.scheme)
Supplementary signals MUST NOT change overallDecision from pass to fail automatically; they are informational. A listing MAY declare in terms that specific signals are gating (e.g. minimum reputation score); when so declared, the gating check is treated as a deal-specific claim and runs through the same aggregation. The four classifications carry distinct diagnostic value: "required not present" (no VerifyResult at all), "required failing" (authority said no), "required indeterminate" (authority answered ambiguously), "required errored" (verifier could not reach authority). Consumers debugging or auditing a failed session can read the failure reasons to determine which class the failure belongs to.

#### 7.7.2 Anchoring and signature

The composite record MUST be anchored via SR-2 at address dacs2:composite:{jobId}:{evaluatedParty} (or substrate equivalent). The anchor reference is recorded in the DACS-5 session record. The composite record’s signature MUST be produced by the verifier (the party running Vet on the counterparty) over the domain-separated payload per chapter 7§7.7:
signed_bytes := "dacs-composite:v1:" || composite_hash
In v1, the composite record carries a single verifier signature. Multi-party composition (e.g., two-sided independent Vet records cross-referenced into one) is deferred to v2.

### 7.8 The vet-credentials phase

type VetCredentialsInput = {jobId: stringactor: "buyer" | "seller"bundleToVet: IdentityBundlerequirement: BundleRequirementverifierIdentity: IdentityBundlesessionContext: SessionContextrecipeRegistryVersion: numberattempt: number}type VetCredentialsOutput = PhaseHandlerResult & {contextDelta: {"vet-credentials": {compositeRecord: AttestationRefoverallDecision: "pass" | "fail" | "indeterminate"}}}

#### 7.8.1 Phase contract

The orchestrator MUST: (VPC-1) invoke vet-credentials after a successful Identify stage and before any Negotiate phase requiring a verified bundle; (VPC-2) run Vet on both the buyer’s bundle (against listing-side requirements on buyers) and the seller’s bundle (against buyer-side requirements on sellers) before Negotiate; (VPC-3) anchor the composite record before returning the phase result; (VPC-4) on overallDecision != "pass" (after permitted retries), MUST fail the phase with errorClass derived from the overall decision: "fail" → counterparty errorClass; "indeterminate" or "error" → transient or permanent per recipe.retryClass.

#### 7.8.2 Error classification and idempotency

| Error class | Cause | Retry? |
| --- | --- | --- |
| transient | Substrate temporarily unavailable; authority HTTP 5xx | Yes (per VP-R1) |
| permanent | Required claim fails; bundle malformed | No |
| counterparty | Counterparty fails to present required deal-specific claim | No (Vet fails; counterparty marked at-fault) |
| substrate | SR-2 or SR-3 unavailable for sustained period | Pause session per DACS-5 |

Re-running vet-credentials with the same inputs MUST produce the same composite-record content (modulo timestamps in supplementary signals refreshed against current state). The orchestrator MUST NOT double-anchor; on retry, the existing anchor MUST be reused if its content has not changed.

### 7.9 Conformance summary

| Role | Requirements |
| --- | --- |
| Method implementer | CM-1 through CM-5 |
| Recipe author | RA-1 through RA-5 |
| Verifier (orchestrator) | VP-R1 through VP-R3; VP-C1 through VP-C3; VPC-1 through VPC-4 |
| VerifyResult consumer | §7.5.2 attestation resolution; recipe-version pinning |
| Composite record reader | §7.7.1 aggregation; signature validation |

### 7.10 Rationale

**Method-pluggable registry vs single method.** A single-method design would force every credential through one verification approach. No single approach fits every credential: cooperative-issuer credentials want W3C VC; private-data credentials want zkTLS; public-registry credentials want consensus-backed proxy attestation. The pluggable registry routes by credential type, and the rest of the stack consumes a uniform VerifyResult.
**Closed v1 method set vs open from day one.** An open method registry from v1 makes conformance untestable: a verifier could declare an arbitrary "my-custom-method" and produce results the rest of the stack cannot validate. v1 ships eight methods covering the established attestation patterns. New methods are added through the steward’s acceptance process, the same way new schemes are added in DACS-1.
**Recipe-per-scheme vs general-purpose verification protocol.** A general-purpose protocol (e.g., a single configurable HTTP-fetch endpoint per credential) would lose the structured parser rules, the success-criterion semantics, and the negative-match pattern that recipes encode. Recipes are small and per-scheme; they capture the messy reality of each authority’s response format.
**Composite record vs separate per-claim records.** The rest of the stack needs to reference *one* artifact for Vet, not N. Separate per-claim records would force every downstream consumer to walk a list and compose. The composite record does the composition once, signs once, anchors once.
**SR-3 dependency for consensus-backed-proxy.** A substrate-agnostic version would require trusting a single proxy (Reclaim-style) or running MPC (TLSNotary-style). Both are valid options — and are present as separate methods in the registry — but they have higher per-verification cost and lower throughput. For high-volume public-registry verification (the bulk of institutional Vet workload), consensus-backed proxy at chain rate is the right tool. The substrate dependency is stated explicitly and is opt-in per recipe.
**Single-verifier signature on composite record vs multi-party signed.** In a typical transaction, the buyer runs Vet on the seller and vice versa; each produces its own composite record. v1 carries one signature per record. Multi-party composition (a single record signed by both sides, attesting to mutual Vet) is more complex and is deferred.
**Reputation as a supplementary signal vs a hard credential gate.** Reputation is not a credential — it has no authoritative source — and treating it as one would let a low-reputation counterparty fail Vet outright, eliminating any ability for new sellers to enter the market. As a supplementary signal, reputation is surfaced to the verifier (and to a listing’s optional gating) but does not by default block engagement.

### 7.11 Backwards compatibility

**W3C Verifiable Credentials.** A DACS-2 recipe for any scheme MAY declare verifiable-credential as its default method. The DACS-2 verifier accepts a W3C VP (Verifiable Presentation) containing the VC; verification follows W3C VC data model 2.0. Recipe authors SHOULD set issuerAllowList for schemes where only specific issuers should be trusted (e.g., a recipe for kyc-tier would allow-list specific KYC providers).
**TLSNotary.** A DACS-2 recipe MAY declare tlsnotary as its method. The proof envelope is a TLSNotary commitment; the notary signature is anchored as part of AttestationRef. Compatible with TLSNotary versions implementing the PSE 2024 rebuild and later.
**zkTLS / Reclaim Protocol / Pluto.** A DACS-2 recipe MAY declare zktls and select a provider; the provider’s circuit/verification program id determines the verifier’s check. Compatible with Reclaim Protocol’s production verifier contracts and any equivalent zkTLS provider that ships a circuit description.
**ERC-8004.** Recipes for the erc8004 scheme use the evm-rpc method to read token ownership via proxy-attested EVM call. DACS-2 results may additionally be published to the ERC-8004 reputation / validation registries via DACS-5; DACS-2 itself does not write there.
**ACME / Let’s Encrypt domain-control challenges.** The domain-tls-control method follows ACME’s challenge/response patterns (RFC 8555). Implementations MAY reuse existing ACME libraries.

### 7.12 Security considerations

**Method substitution.** *Threat:* a verifier uses a weaker method than the recipe declares. *Mitigation:* the VerifyResult.method field MUST be the method actually executed; consumers compare to the recipe’s defaultMethod and alternatives and reject results that used an unaccepted method. Recipes SHOULD list only equivalent alternatives.
**Recipe poisoning.** *Threat:* a compromised recipe registry returns incorrect parsing rules, causing every verification using that recipe to mis-classify outcomes. *Mitigation:* recipes are signed by the registry steward (currently KyneSys Labs, per §7.4.4); consumers MUST verify the signature. Recipe recipeVersion is monotonic and pinned per session; an attacker compromising the registry tomorrow does not retroactively change recipes used in already-pinned sessions. Steward-key compromise is the principal residual risk under the current PA-2 single-signer phase; multi-signature governance (PA-3) is the v1.1+ mitigation pathway.
**Replay of VerifyResult across sessions.** *Threat:* a verifier reuses a stale VerifyResult from a different session or a different counterparty. *Mitigation:* the VerifyResult.identifier MUST match the claim under verification canonically; consumers verify the match. The composite record’s bundleHash and evaluatedParty bind the verification to a specific bundle. Cross-session reuse within validUntil is explicitly permitted and is safe because the result still verifies against the same identifier.
**TOCTOU: authority state changes between fetch and use.** *Threat:* a counterparty’s authority status changes between Vet and the actual transaction execution. *Mitigation:* listings handling time-sensitive flows SHOULD set ClaimRequirement.maxAge aggressively (e.g., 60 seconds for OFAC clearance on a real-money trade). Sessions with long latency between Vet and Settle SHOULD re-run Vet at Settle time for the most stake-sensitive claims.
**Substrate validator capture (SR-3).** *Threat:* a majority-corrupt substrate validator set forges responses, causing consensus-backed-proxy and evm-rpc methods to attest to false facts. *Mitigation:* the substrate’s consensus model is the trust floor; DACS-2 inherits whatever security properties the substrate provides. For credentials where this risk is unacceptable, recipes SHOULD declare multi-method alternatives — e.g., consensus-backed-proxy AND tlsnotary AND zktls, with the recipe requiring all (or a majority) to pass. The composite verification record’s aggregation algorithm supports this via multiple VerifyResults for the same claim; readers MUST honour the recipe’s requireAll flag when set.
**W3C VC issuer compromise.** *Threat:* the signing key of a VC issuer is compromised, causing all VCs signed by that issuer to be untrustworthy. *Mitigation:* recipes SHOULD set issuerAllowList and check issuer revocation registries (where available, e.g., W3C VC Status List 2021). Issuers SHOULD rotate keys regularly. Consumers of historical VerifyResults SHOULD check whether the issuer was known-compromised at the time of verification.
**TLSNotary notary collusion.** *Threat:* the notary colludes with the prover to attest to false TLS sessions. *Mitigation:* TLSNotary’s MPC ensures the notary cannot see plaintext but does require honesty about the commitment. Recipes using tlsnotary SHOULD specify a known-good notary public key in the recipe; multi-notary patterns are a future extension.
**zkTLS proxy compromise.** *Threat:* the zkTLS provider’s proxy attests to false TLS responses. *Mitigation:* recipes select specific providers and program IDs. Compromise of a provider’s proxy invalidates all results from that provider; consumers SHOULD treat results from a known-compromised provider as invalid retroactively.
**Supplementary signal poisoning.** *Threat:* false reputation data is injected (e.g. a Sybil network creating fake successful completions). *Mitigation:* DACS-5 specifies the reputation derivation; supplementary signals from DACS-5 inherit DACS-5’s anti-Sybil properties. Supplementary signals from external sources MUST carry an AttestationRef; consumers MAY decline to weigh signals from sources they do not trust.
**Identifier canonicalisation gaps.** *Threat:* the same logical identifier in two different canonical forms produces two different VerifyResult lookups, allowing an attacker to substitute. *Mitigation:* per-scheme canonicalisation rules are normative. Verifiers MUST canonicalise before issuing a VerifyResult and consumers MUST canonicalise before comparing.
**Composite record forgery.** *Threat:* an attacker constructs a composite record with overallDecision = "pass" and false VerifyResultRefs. *Mitigation:* the composite record is signed by the verifier; consumers verify the signature. The bundleHash and requirementHash fields bind the record to specific inputs; consumers verify these against the inputs they actually used. Each VerifyResultRef MUST be dereferenced and content-hash-validated before the composite record is accepted.
**Indeterminate-decision exploitation.** *Threat:* an attacker arranges for a required claim’s verification to fail in a way that returns indeterminate rather than fail, hoping consumers treat the result as pass. *Mitigation:* indeterminate is not pass. The aggregation algorithm treats indeterminate in a required position as overall indeterminate, which MUST fail the phase.
## Chapter 8 — DACS-3: Negotiate
**Stage:** Negotiate (3rd of 5). **Status:** Draft (part of DACS v0.1). **Depends on:** SR-2 (required for public commitments), SR-4 (required for genuinely private negotiation patterns); references DACS-1 listings and DACS-2 verified bundles. **Used by:** DACS-4 (pricing + rail input to settlement), DACS-5 (agreement reference in session bundle).

### 8.1 Abstract

DACS-3 specifies how transacting parties arrive at agreed terms and bind themselves cryptographically to the outcome. It defines:
Negotiation contents stay between the participants. The public chain receives only what is needed to bind them to the outcome.

### 8.2 Motivation

Negotiation is the stage in which commerce most consistently breaks open standards. Identity, payment, and capability discovery can run on public infrastructure with little privacy cost. Pricing, term-sheet drafts, sealed-bid submissions, and RFQ counter-offers cannot — they involve material non-public information, competitive pricing, or simply discussions whose contents would harm the participants if exposed.
A buyer agent today can issue an RFQ to three sellers via public channels, but the public visibility of the request, the counters, and the timing telegraphs market information that institutional desks specifically pay to keep private. Sealed-bid government procurement cannot run on a public mempool. Pre-trade negotiation in regulated markets is bound by MNPI rules that public-chain visibility violates.
DACS-3 closes this gap by separating two concerns that have historically been fused:
This separation is what makes institutional and regulated flows possible on a public-permissionless substrate. It is also what most distinguishes DACS from existing open standards: payment authorisation standards (AP2, x402), identity registries (ERC-8004), and credential attestation standards (W3C VC, zkTLS) all assume public negotiation or no negotiation. DACS-3 takes the negotiation primitive seriously.
A third concern: not every transaction needs private negotiation. A $0.01 API micropayment with a fixed price has nothing to negotiate; the listing’s posted terms are the agreement, and "acceptance is the negotiation." DACS-3 makes this case trivial via negotiate-fixed-price, which requires only SR-2 (anchoring) and works on any substrate. The substrate-locked patterns (negotiate-rfq, negotiate-sealed-envelope) are opt-in per listing.

### 8.3 Negotiation channel model

A negotiation channel is a coordination surface with the following properties.

#### 8.3.1 Required properties

(CH-1) **Identity-keyed membership.** The channel’s member set is a list of ClaimReferences. Each member’s primary claim MUST appear in their verified DACS-1 bundle. Membership changes MUST be signed by an existing member (per the channel’s admission policy) and MUST be observable to all current members.
(CH-2) **Confidentiality.** Non-members MUST NOT be able to read channel contents. The public chain MUST see only commitments (envelope commitments, agreement hash) and never raw offer/counter/reveal payloads.
(CH-3) **Authenticity.** Every message in the channel MUST be signed by its author’s primary key (the key associated with the author’s primary claim). Verifiers MUST be able to validate signatures using the same keys used in DACS-2 verification.
(CH-4) **Liveness.** The channel MUST deliver messages to all members within a bounded delay. Members MUST be able to detect channel-level failure (partition, censorship by the channel operator) and abort.
(CH-5) **Termination.** The channel MUST produce a terminal state. Terminal states are: (a) a signed AgreementDocument; (b) an abort signed by any party; (c) timeout. The terminal state is referenced by commit-agreement (if agreement) or recorded as a failed Negotiate stage (otherwise).

#### 8.3.2 SR-4 realisation

On Demos, L2PS (Layer-2 Privacy Subnets) is the SR-4 implementation. Channel sessions are subnets; messages stay between subnet members; the public chain stores only commitment hashes and the final agreement hash (as Storage Programs). For v1, subnet membership MUST be bindable to the participants’ CCI primary claims so that channel-message signatures verify against the same key that holds value on-chain, and the commit-agreement anchor’s parties match the channel members. Until CCI-keyed membership ships, implementations MAY use a binding-proof step: each participant signs an "L2PS subnet X membership = CCI Y" attestation with their CCI primary key, anchored as a Storage Program before negotiation begins.
Other substrates MAY implement SR-4 via TEE-based confidential channels, zk-based privacy circuits, or permissioned-overlay networks bound to public-chain identity, provided they satisfy CH-1 through CH-5. DACS-3 does not standardise the wire protocol or the cryptographic envelope — those are SR-4 implementation choices — but does standardise the messages’ semantic shape.

#### 8.3.3 Message envelope (substrate-independent)

type ChannelMessage = {channelId: string// substrate-derived; opaque to DACS-3sequence: number // monotonic per channel, starts at 1sender: ClaimReference // author's primary claimsentAt: number // unix mstype: "offer" | "counter" | "accept" | "reject"| "sealed-envelope-commit" | "sealed-envelope-reveal"| "abort" | "membership-change"body: unknown// type-specific (defined per pattern)refs?: { repliesTo?: number }signature: ChannelMessageSignature // see below}
The envelope’s canonical form is the RFC 8785 JCS serialisation with the signature field omitted. The envelope hash is sha256(canonical_form). The signature is computed over the domain-separated payload per chapter 7§7.7:
signed_bytes := "dacs-channelmsg:v1:" || envelope_hash
Implementations MAY add transport-level fields (routing, framing) outside the signed envelope; signed envelope contents MUST NOT change between sender and receiver.

#### 8.3.4 Channel failure detection and abort

A member MUST treat the channel as failed when: a message they sent is not acknowledged by a quorum of members within the channel’s liveness bound; a member they expect to respond does not respond within a per-pattern timeout; they observe contradictory views of the channel state from different sources (channel-operator forking).
On detected failure, the member MAY send an abort message (best-effort), abandon the channel, and record the failure in the session record (DACS-5) with classification counterparty or substrate as appropriate. An abort terminates the channel and the Negotiate phase. The abort message’s signed envelope MAY be anchored via SR-2 as an audit artifact. The phase returns PhaseHandlerResult with ok: false and an error class.

### 8.4 Negotiation patterns

The v1 closed set. Each is a DACS-3 phase type with a phase-handler contract.

#### 8.4.1 negotiate-fixed-price

Acceptance of the listing’s posted terms. No private channel required.
type NegotiateFixedPriceInput = {jobId: stringlistingHash: string// pinned listing's content hashlistingRef: { listingId: string; version: number }buyerBundle: IdentityBundle// post-VetsellerBundle: IdentityBundle // post-VetbuyerVetRef: AttestationRef// from DACS-2sellerVetRef: AttestationRef // from DACS-2sessionContext: SessionContext}type NegotiateFixedPriceOutput = PhaseHandlerResult & {contextDelta: {"negotiate-fixed-price": {agreementHash: stringagreementRef: AttestationRef}}}
**Procedure.** The orchestrator (or buyer agent, depending on actor) MUST: construct an AgreementDocument with derivedFromPattern: "fixed-price", copying terms directly from the listing’s pricing, acceptedRails (using the buyer’s selected rail), deliverable, and deadline (computed as now + listing.terms.deadlineSecAfterCommit); collect buyer signature; collect seller co-signature; anchor the agreement document via SR-2; return agreementHash and agreementRef.
**Seller-side auto-accept (optional)**
A listing MAY declare terms.acceptanceModel: "auto-accept", in which case the seller pre-issues a **template acceptance commitment** alongside the listing rather than a per-session signature. The mechanism:
Listings using auto-accept MUST publish the AutoAcceptCommitment alongside the listing, and the buyer’s orchestrator MUST verify the commitment before relying on auto-accept. A pre-issued per-instance signature (signing a placeholder agreement hash) MUST NOT be used; the per-instance signature binds to a specific agreement hash. Sellers operating auto-accept MUST hold the auto-signing key in a system that produces live instance signatures on demand (HSM, TEE, hot wallet with rate-limiting).
**Substrate:** SR-2 only.

#### 8.4.2 negotiate-rfq

Bounded multi-turn offer-and-counter exchange in a private channel.
type NegotiateRfqInput = {jobId: stringlistingHash: stringlistingRef: { listingId: string; version: number }buyerBundle: IdentityBundlesellerBundle: IdentityBundlebuyerVetRef: AttestationRefsellerVetRef: AttestationRefparameters: {maxTurns: number // hard cap; default 6; MUST be >= 2timeoutSec: number // per-turn timeoutchannelSubnet?: string // SR-4 channel id; substrate-specific}sessionContext: SessionContext}type NegotiateRfqOutput = PhaseHandlerResult & {contextDelta: {"negotiate-rfq": {agreementHash: stringagreementRef: AttestationRefturnCount: numberchannelTranscriptRef?: AttestationRef// optional; member-only-decryptable}}}
**Procedure.** The orchestrator (driving the buyer-side flow) MUST: (1) establish an SR-4 channel between buyerBundle.presentedBy and sellerBundle.presentedBy; (2) send an initial offer — buyer (or seller, depending on listing’s rfqInitiator field) sends a turn of type offer with proposed terms; (3) iterate — each side MAY respond with counter, accept, or reject; iteration continues until accept is received (proceed), reject is received (terminate; counterparty class), maxTurns is reached without accept (terminate; counterparty class), or timeoutSec elapses without a response (terminate; counterparty or substrate class); (4) construct the AgreementDocument with derivedFromPattern: "rfq" and the agreed terms, sign and send as a final message; (5) collect co-signatures from all parties; (6) anchor the agreement via SR-2; (7) optionally, if all parties consent, anchor the encrypted transcript via SR-2 with a channelTranscriptRef. Consent MUST be explicit; default is no transcript anchoring.
**Conformance.** (RFQ-1) maxTurns MUST be ≥ 2. (RFQ-2) Each turn MUST conform to the channel message envelope. (RFQ-3) Final terms MUST conform to the listing’s pricing band — counters proposing terms outside the band MUST be rejected client-side; signed agreements with out-of-band terms MUST be rejected by commit-agreement. (RFQ-4) Implementations MUST enforce the turn timeout; missed-timeout abandonment MUST be treated as channel failure. **Substrate:** SR-2 + SR-4.

#### 8.4.3 negotiate-sealed-envelope

Sealed-bid procurement: all bidders submit hash-committed bids before a deadline; bids are revealed after the deadline; winner is selected per the listing’s selection criterion.
type NegotiateSealedEnvelopeInput = {jobId: stringlistingHash: stringlistingRef: { listingId: string; version: number }buyerBundles: IdentityBundle[] // all bidders' bundlessellerBundle: IdentityBundle // listing publisherbuyerVetRefs: AttestationRef[]sellerVetRef: AttestationRefparameters: {commitDeadline: number // unix ms; MUST be > nowrevealWindow: number // seconds after commitDeadline; MUST be >= 60selectionRule: "lowest-price" | "highest-price" | "first-acceptable" | "rule-ref:<uri>"channelSubnet?: string}sessionContext: SessionContext}type NegotiateSealedEnvelopeOutput = PhaseHandlerResult & {contextDelta: {"negotiate-sealed-envelope": {agreementHash: stringagreementRef: AttestationRefwinningBidderClaim: ClaimReferencerevealedBidRefs: AttestationRef[]losingBidderClaims: ClaimReference[]}}}
**Procedure.** The orchestrator MUST: (1) establish SR-4 channels between the seller and each bidder; (2) **bidder commit phase** (before commitDeadline) — each bidder constructs a bid, computes bidHash = sha256(canonical_JCS(bid) || bidder_salt), and sends a sealed-envelope-commit message {bidHash, bidderClaim, commitTimestamp}; the commit message’s bidHash MUST also be anchored via SR-2; (3) at commitDeadline, no further commits are accepted; the orchestrator records the set of received commits; (4) **bidder reveal phase** (within revealWindow) — each bidder sends a sealed-envelope-reveal message {bid, salt} matching their prior bidHash; orchestrator verifies sha256(canonical_JCS(bid) || salt) == bidHash; mismatches cause exclusion; (5) **selection** — the orchestrator applies parameters.selectionRule; ties resolved by earliest commit timestamp; (6) construct the AgreementDocument from the winning bid with derivedFromPattern: "sealed-envelope" (losing bidders listed as bidder-non-winning parties; their signatures are not required); (7) anchor the agreement and reveal records via SR-2.
**Selection rules and the rule-ref binding requirement**
parameters.selectionRule is one of: "lowest-price"; "highest-price"; "first-acceptable" (per listing-defined acceptance criteria); "rule-ref:<contentHash>:<uri>". For rule-ref, the rule MUST be anchored as a Storage Program (or fetched from an HTTPS URI and content-hash-bound). The URI is purely informational; the <contentHash> in the selection rule string is the authoritative binding. Orchestrators MUST fetch the rule at <uri> (or the substrate anchor), compute sha256 of the canonical form, and verify it matches <contentHash>. Mismatch MUST exclude the rule and fail the selection step with errorClass: permanent. This prevents a seller from changing the selection algorithm after bids have been submitted by changing the content served at <uri>.
**Conformance.** (SE-1) commitDeadline MUST be at least 60 seconds in the future at session start. (SE-2) Every bidder commit MUST be anchored before commitDeadline; commits whose anchor timestamp is after commitDeadline MUST be excluded. (SE-3) Reveals MUST occur within revealWindow; late reveals MUST be excluded. (SE-4) Bidders failing reveal MUST be excluded from selection and MAY be marked with a failure-to-reveal reputation event (DACS-5). (SE-5) The selection rule MUST be deterministic; ties MUST resolve consistently. (SE-6) rule-ref selection rules MUST be content-hash-bound and the rule content MUST itself be deterministic given the bid set. **Substrate:** SR-2 + SR-4.

### 8.5 Agreement document

The canonical output of any negotiation pattern.
type AgreementDocument = {agreementVersion: "1"jobId: stringlistingRef: {listingId: stringversion: numbercontentHash: string// pinned listing content hash}parties: AgreementParty[]terms: {deliverable: DeliverableRef// DACS-4 referenceprice: PriceTerm // DACS-4 referencerail: PaymentRailRef // DACS-4 reference (must appear in listing.acceptedRails)deadline: number // unix ms; settle-by deadlineadditionalTerms?: Record<string, unknown>}derivedFromPattern: "fixed-price" | "rfq" | "sealed-envelope"derivedFromChannel?: {subnet: stringlastMessageHash: string}generatedAt: numbersignatures: AgreementSignature[]}type AgreementParty = {role: "buyer" | "seller" | "bidder-non-winning"bundleHash: string // sha256 of the post-Vet IdentityBundleprimaryClaim: ClaimReference // pulled from bundle.presentedByvetRecordRef: AttestationRef // DACS-2 composite verification record}// Optional: competitive context for best-execution audit.type CompetitiveContext = {pattern: "rfq" | "sealed-envelope"receivedQuotes: Array<{fromParty: ClaimReferencequoteHash: string// hash of the losing quote contentsquoteRef?: AttestationRef}>}// AgreementDocument.terms.additionalTerms MAY include "competitiveContext: CompetitiveContext".type AgreementSignature = {party: ClaimReferencealgorithm: "ed25519" | "ecdsa-secp256k1" | "sr1-aggregate"value: string// signature over agreement hash}

#### 8.5.1 Canonical serialisation and signature

The agreement’s canonical form is the RFC 8785 JCS serialisation with the signatures field omitted. The agreement hash is sha256(canonical_form), hex-encoded. Each AgreementSignature.value is computed over the domain-separated payload per chapter 7§7.7:
signed_bytes := "dacs-agreement:v1:" || agreement_hash
Verifiers MUST recompute the canonical form, agreement hash, and domain-separated payload, and for each required party, resolve the primary claim’s key (per DACS-2 verification) and verify the signature. Required signers: negotiate-fixed-price — buyer + seller (seller signature may be an auto-accept instance signature per §8.4.1); negotiate-rfq — buyer + seller; negotiate-sealed-envelope — seller + winning bidder (non-winning bidders’ signatures are not required).

#### 8.5.2 Listing conformance validation

A verifier MUST validate the agreement against its referenced listing: terms.price MUST lie within the listing’s pricing band (if pricing is negotiable, within the declared negotiable.minPct / negotiable.maxPct band; if fixed, equal to the listed price); terms.rail MUST appear in listing.acceptedRails; terms.deliverable MUST conform to the listing’s offering.deliverable (deliverable hash, schema reference, etc.); terms.deadline MUST be ≤ generatedAt + listing.terms.deadlineSecAfterCommit; derivedFromPattern MUST match the listing’s pipeline-declared negotiation pattern. Agreements failing any check MUST be rejected by commit-agreement.

### 8.6 Commitment phase (commit-agreement)

The DACS-3 phase that anchors the agreement hash on the public chain.
type CommitAgreementInput = {jobId: stringagreement: AgreementDocumentlistingRef: { listingId: string; version: number; contentHash: string }sessionContext: SessionContext}type CommitAgreementOutput = PhaseHandlerResult & {contextDelta: {"commit-agreement": {agreementHash: stringanchorTxRef: TxRefcommittedAt: number}}}
**Procedure.** The orchestrator MUST: (1) compute agreementHash = sha256(canonical_JCS(agreement)) with signatures omitted; (2) verify all required signatures are present and valid; (3) validate the agreement against the listing — any validation failure MUST cause the phase to fail with class permanent; (4) construct the on-chain commitment record:
type CommitmentRecord = {dacsVersion: "1"jobId: stringagreementHash: stringlistingRef: { listingId: string; version: number; contentHash: string }parties: ClaimReference[]// primary claims of signing partiespattern: "fixed-price" | "rfq" | "sealed-envelope"committedAt: number}
(5) anchor the commitment record via SR-2 at address dacs3:commit:{jobId} (or substrate-equivalent), with the orchestrator signature over the domain-separated payload "dacs-commitment:v1:" || sha256(canonical_JCS(commitmentRecord_without_signature)); (6) return agreementHash and anchorTxRef.
**Conformance.** (CA-1) The orchestrator MUST NOT advance to DACS-4 (Settle) until commit-agreement returns ok: true. (CA-2) Commitment records MUST be anchored on the public chain (not in a private channel). (CA-3) Once anchored, the commitment is immutable. Re-commitments for the same jobId MUST be rejected. (CA-4) The agreement document itself MAY be anchored separately (publicly or privately). For institutional flows, the agreement document is typically NOT anchored on the public chain — only its hash is. Parties retain the agreement document off-chain (or encrypted-anchored).

### 8.7 Channel transcript and disclosure

Negotiation channels produce a transcript: the ordered sequence of signed messages between participants. The transcript is private to channel members. When a transcript is anchored (see disclosure policies below), its signature is computed over the domain-separated payload "dacs-transcript:v1:" || sha256(canonical_JCS(transcript_without_signatures)) per chapter 7§7.7.
type ChannelTranscript = {transcriptVersion: "1"channelId: stringmembers: ClaimReference[]messages: ChannelMessage[]generatedAt: numbersignatures: TranscriptSignature[]}
**Default disclosure: none.** By default, the transcript is not anchored on the public chain. Only the agreement hash (via commit-agreement) is public. The DACS-1 listing’s terms.transcriptDisclosurePolicy controls this per-listing:
If all channel members consent, the transcript MAY be encrypted to the member set and anchored via SR-2. The AgreementDocument.derivedFromChannel.lastMessageHash provides a verifiable hook from the public agreement to the (private) transcript. A future DACS standard (proposed DACS-X dispute) MAY require selective transcript disclosure under signed party agreement or arbitrator order. v1 does not specify dispute resolution; parties intending to support dispute SHOULD anchor encrypted transcripts at agreement time so disclosure is technically possible later.

### 8.8 Pattern selection by listing

A DACS-1 listing’s pipeline declares which negotiation pattern is used. Each PhaseStep of kind negotiate-* specifies the pattern and its parameters.
**Validation.** (PS-1) A pipeline MUST contain exactly one negotiate-*phase. (PS-2) A pipeline MUST contain exactly one commit-agreement phase, immediately following the negotiate-* phase. (PS-3) The listing’s pricing model MUST be compatible with the chosen pattern: negotiate-fixed-price MUST be fixed or negotiable (in which case fixed-price uses the band’s centre); negotiate-rfq MUST be negotiable; negotiate-sealed-envelope MUST be auction.
**Fallback to fixed-price.** A listing offering negotiate-rfq MAY declare fixedPriceFallback: true in the pipeline step. When true, a buyer that does not wish to negotiate MAY signal acceptance of the listed centre-price via negotiate-fixed-price. The orchestrator selects which pattern runs based on buyer signal. The fallback path produces a normal AgreementDocument with derivedFromPattern: "fixed-price".
**Multi-quote RFQ (deferred to v1.1).** The v1 negotiate-rfq phase is bilateral (one buyer, one seller). Real institutional RFQ is often one-to-many — a buyer queries N liquidity providers, collects quotes, picks one. v1 does not support multi-quote RFQ directly; the closest pattern is negotiate-sealed-envelope with selectionRule: first-acceptable or lowest-price. A first-class negotiate-multi-quote phase is anticipated for v1.1.

### 8.9 Conformance summary

| Role | Requirements |
| --- | --- |
| Channel implementation | CH-1 through CH-5; message envelope; failure detection |
| negotiate-fixed-price | §8.4.1 procedure; signature collection; SR-2 anchoring |
| negotiate-rfq | §8.4.2 procedure; RFQ-1 through RFQ-4; channel turn timeouts |
| negotiate-sealed-envelope | §8.4.3 procedure; SE-1 through SE-5; deterministic selection |
| commit-agreement | CA-1 through CA-4; signature and conformance validation |
| Listing publisher | PS-1 through PS-3 |
| Substrate without SR-4 | MUST support negotiate-fixed-price; MUST refuse negotiate-rfq and negotiate-sealed-envelope with a clear substrate-capability-missing error |

### 8.10 Rationale

**Three patterns vs more, fewer, or open registry.** Three is the smallest set that covers the demonstrated commerce surface: micropayments and SaaS use fixed-price; institutional bilateral negotiation uses RFQ; sealed-bid procurement uses sealed-envelope. Open registries lose conformance testability. More patterns (auction-english, auction-dutch, multi-round-RFQ-with-deltas) are deferred to v2 once v1’s set ships with real users.
**Closed pattern set in v1 vs open from day one.** The pattern set determines what a generic orchestrator must implement. A closed set lets every conforming orchestrator handle every conforming listing. An open set means listings can declare patterns no orchestrator supports — fragmentation by design.
**Single AgreementDocument shape across patterns vs pattern-specific shapes.** The same downstream stages (Settle, Verify) consume agreements regardless of how they were negotiated. A uniform shape lets DACS-4 and DACS-5 stay pattern-agnostic. Pattern-specific data lives in additionalTerms and in optional fields (e.g., derivedFromChannel).
**Channel transcript private by default vs anchored-with-encryption.** Anchoring transcripts by default would be expensive (transcripts can be large) and politically fraught (operators would not adopt a standard that anchors their negotiation history, even encrypted). Default-private with opt-in anchoring matches institutional practice: keep the negotiation private, anchor the binding outcome. Parties who need transcript audit (regulated flows) opt in.
**commit-agreement as a separate phase vs implicit in the negotiation phase output.** A separate phase makes the public-chain commitment visible in the pipeline, lets the orchestrator validate signature and conformance before the agreement becomes binding, and provides a clear hook for downstream Settle/Verify. Implicit commitment hides the binding moment and complicates failure recovery.
**SR-4 as required for private vs alternative substrate primitives.** SR-4 is an abstract capability; substrates can realise it via private subnets (Demos), TEEs (with documented trust trade-offs), permissioned channels (where appropriate), or zk-based confidential channels. DACS-3 does not pick a winner among realisations; it specifies the abstract capability and the per-pattern requirements.
**Sealed-envelope: commit anchored, reveal in channel.** The commit hash on the public chain prevents back-dating and repudiation; the reveal stays in the channel to avoid leaking losing bids publicly. This matches government sealed-bid practice: bids are sealed in advance, revealed only to the procurement officer at opening time, and losing bid amounts are typically not disclosed.

### 8.11 Backwards compatibility

**Institutional RFQ workflows.** A Demos-hosted RFQ run via negotiate-rfq maps to existing institutional bilateral RFQ in the same way a Bloomberg chat RFQ maps to a Symphony RFQ: same semantic shape, different transport. The transport is the SR-4 channel; the semantic shape is the DACS-3 phase. Existing RFQ desks can adopt DACS-3 RFQ without changing their negotiation logic — they wrap their existing logic as a DACS-3 phase implementation.
**Sealed-bid government procurement.** FAR Part 14 (sealed bidding) and FAR Part 15 (contracting by negotiation) describe the US federal patterns. DACS-3’s negotiate-sealed-envelope covers FAR Part 14’s commit-then-reveal pattern with the addition of cryptographic commitment (the FAR pattern uses physical sealed envelopes). The selection-rule abstraction (lowest-price, first-acceptable, rule-ref) covers FAR’s "lowest responsive responsible bidder" and "best value" criteria. International equivalents (EU procurement directives, UK Crown Commercial Service) map similarly.
**Off-chain negotiation systems (existing).** A negotiation system that already exists (an institutional desk’s RFQ system, a procurement portal, a B2B contract negotiation tool) MAY function as the SR-4 channel for DACS-3 purposes provided it satisfies CH-1 through CH-5. The public-chain binding (commit-agreement) and the agreement document shape are the DACS-3 additions; the negotiation transport can be existing infrastructure.
**ERC-8183 escrow.** ERC-8183 introduces an EVM-native escrow primitive for job-style transactions. A DACS-3 agreement whose terms.rail is an EVM rail MAY reference an ERC-8183 escrow as the settlement vehicle; the rail definition (DACS-4) carries the ERC-8183 contract address.
**Future negotiation patterns.** New patterns (auctions, multi-round delta-RFQ) are added via the DACS-3 version process. Adding a pattern requires registering its phase-handler contract, parameters, and substrate requirements.

### 8.12 Security considerations

**Channel-operator censorship.** *Threat:* the SR-4 channel operator drops messages, preventing a party from responding within the timeout. *Mitigation:* CH-4 mandates liveness detection. Members observing missed deliveries (no acknowledgement from a quorum) MUST treat the channel as failed. On Demos, Private Negotiation provides per-message acknowledgements; equivalent SR-4 implementations on other substrates SHOULD do the same.
**Channel-operator forking.** *Threat:* the channel operator shows different views to different members, creating mutual misunderstanding. *Mitigation:* channel message envelopes carry monotonic sequence numbers and signatures; members SHOULD periodically exchange "current state" attestations and detect forks. SR-4 implementations are expected to provide a tamper-evident message log.
**Replay of offers across sessions.** *Threat:* an attacker captures a signed offer from session A and replays it in session B. *Mitigation:* the channel message envelope includes channelId (substrate-unique per session) and sequence (per-channel monotonic). An offer message replayed into a different channel fails signature verification because the channel id differs; replayed in the same channel it duplicates a sequence number and is rejected.
**Signature stripping or rebinding between channel and agreement.** *Threat:* an attacker takes a signature produced inside the channel and reuses it on a different agreement document. *Mitigation:* channel-message signatures are over the message envelope (including channelId); agreement-document signatures are over the agreement hash (which includes jobId, listingRef, and all terms). The two scopes are non-overlapping; a channel signature does not validate as an agreement signature.
**Sealed-envelope front-running.** *Threat:* a bidder learns competitors’ bids before reveal. *Mitigation:* bids stay encrypted in the channel until reveal; only the bid hash is public. The channel’s confidentiality ensures non-members cannot read pre-reveal bids; the cryptographic commitment ensures the bidder cannot change their bid after observing competitors at reveal time. Operators SHOULD use SR-4 implementations with member-exclusive encryption.
**Sealed-envelope post-deadline submission.** *Threat:* a bidder submits a commit after commitDeadline, claiming clock skew. *Mitigation:* SE-2 mandates the commit’s public-chain anchor timestamp (objective, substrate-determined) be ≤ commitDeadline. Clock skew at the bidder is irrelevant; the chain decides the timestamp.
**Agreement-listing mismatch.** *Threat:* a signed agreement contains terms outside the listing’s pricing band or with an unaccepted rail. *Mitigation:* validation rules; commit-agreement must reject. Both sides also SHOULD validate before signing.
**Multi-party signing race.** *Threat:* one party signs an agreement; before the other co-signs, the first party publicly commits and locks the other in. *Mitigation:* commit-agreement requires all required signatures present. A unilaterally-signed agreement fails CA. A future minor version MAY add pending-co-signature semantics for asynchronous flows; v1 requires synchronous signature collection.
**Public-chain timing analysis.** *Threat:* the pattern of commitment timestamps on the public chain reveals negotiation patterns. *Mitigation:* this is a fundamental property of any commit-on-chain protocol. Parties concerned with timing leak SHOULD use SR-4 channels with timing-padded delivery, anchor commitments at random intervals within a window, or settle through privacy-preserving rails. DACS-3 does not standardise timing obfuscation.
**Identity substitution between Vet and agreement signature.** *Threat:* a party’s bundle is verified in DACS-2 but they sign the agreement with a different key. *Mitigation:* AgreementSignature.party references the primary claim from the bundle. The signature key MUST be the one bound to that claim. Mismatches cause commit-agreement to fail.
**Channel-membership exfiltration.** *Threat:* the channel operator (or a compromised member) leaks the negotiation transcript publicly. *Mitigation:* DACS-3 cannot prevent this technically — once a member sees the transcript, they can leak it. Listings handling sensitive flows SHOULD restrict membership to known counterparties; the leak risk reduces to counterparty-trust risk, which DACS-2 verification helps quantify.
**Late-revealing bidder denial-of-service.** *Threat:* in sealed-envelope, a bidder commits and then deliberately fails to reveal, hoping to disrupt the auction. *Mitigation:* SE-4 excludes non-revealing bidders from selection and marks them with a reputation event. Repeated failures damage their DACS-5 reputation. Listings MAY require a stake from bidders (escrowed at commit, returned on reveal) to make denial-of-service costly; v1 does not standardise stake.
## Chapter 9 — DACS-4: Settle
**Stage:** Settle (4th of 5). **Status:** Draft (part of DACS v0.1). **Depends on:** SR-2 (required), SR-5 (required for cross-chain rails only); composes with AP2, x402, ERC-20, SPL, HTLC contracts, and substrate-native bridges (Liquidity Tanks on Demos). **Used by:** DACS-5 (settlement evidence in session bundle).

### 9.1 Abstract

DACS-4 specifies how value is exchanged and the deliverable provided once a DACS-3 agreement is committed. It defines:
Payment and delivery are decoupled: a listing’s pipeline composes one or more payment phases with one or more delivery phases, in any order the seller deems safe. The DACS-3 agreement document carries the chosen rail and deliverable references; DACS-4 phases consume them and produce evidence DACS-5 anchors.

### 9.2 Motivation

Settlement is the stage where the most working open standards exist. Stablecoin transfers, ERC-20 / SPL token movements, HTLC swaps, AP2 payment mandates, and x402-style HTTP micropayments all ship in production today. None of them, individually, is sufficient for the agent commerce lifecycle, because each addresses a single slice:
DACS-4 composes these standards into a uniform settlement layer. The payment rail registry routes each rail to its appropriate phase handler; the SettlementEvidence shape lets DACS-5 anchor the result regardless of which rail was used; the cross-chain coordination layer extends to settlements that span chains.
A second motivation is **scope discipline**: DACS-4 does not specify new payment cryptography. It composes existing protocols, adds the registry and evidence schema, and provides cross-chain coordination via substrate primitives (SR-5). The new bytes-on-the-wire are limited to the rail registry, the SettlementEvidence shape, and the phase-handler contracts.

### 9.3 Shared types

These types are referenced by DACS-1 (listings), DACS-3 (agreements), DACS-4 (this chapter), and DACS-5 (session record).
type PaymentRailRef = {railId: string // e.g. "evm-erc20:1:USDC" or "demos:cross-chain-tank:USDC"railVersion?: number // pinned at session start if setparameters?: Record<string, unknown>}type PricingSpec =| { kind: "fixed"; price: PriceTerm }| { kind: "negotiable"; bandCenter: PriceTerm; minPct: number; maxPct: number }| { kind: "auction"; reservePrice?: PriceTerm; selectionRule: "lowest-price" | "highest-price" | string }type PriceTerm = {amount: string // decimal as string, full precisioncurrency: string // ISO 4217 fiat OR asset id (e.g. "usd-stablecoin", "USDC", "SOL")unit?: string// optional unit qualifier (e.g. "per-call")}type DeliverableSpec =| { kind: "storage-program"; schemaUrl?: string; expectedSizeBytes?: number }| { kind: "entitlement"; durationSec: number; renewable: boolean }| { kind: "attested-payload"; payloadFormat: string; expectedSizeBytes?: number }| { kind: "external"; description: string; verificationMethod?: VerificationMethod }type DeliverableRef = {deliverableType: DeliverableSpec["kind"]hash: string // sha256 of the deliverable specschemaUrl?: string}// On-chain transaction reference; discriminated union.type TxRef = ChainTxReftype ChainTxRef =| { kind: "evm"; chainId: number; txHash: string }| { kind: "solana"; cluster: "mainnet" | "devnet" | "testnet"; signature: string }| { kind: "demos"; txHash: string }| { kind: "storage-program"; address: string; writeTxHash: string }| { kind: "ap2"; mandateId: string; providerRef: string }| { kind: "x402"; httpResource: string; paymentReceiptHash: string }| { kind: "htlc-lock"; chainId: number; contractAddress: string; lockTxHash: string }| { kind: "htlc-reveal"; chainId: number; contractAddress: string; revealTxHash: string }| { kind: "liquidity-tank"; bridgeId: string; sourceChainId: number; destChainId: number; lockTxHash: string; releaseTxHash?: string }

### 9.4 Payment rail registry

A versioned, anchored set of payment rails. Each rail entry describes one settlement path.

#### 9.4.1 Rail schema

type RailDefinition = {registryVersion: numberrailId: string // canonical id; lowercase ASCII; max 64 charsrailType: "evm-erc20" | "solana-spl" | "cross-chain-htlc" | "cross-chain-liquidity-tank" | "ap2" | "x402"asset: AssetSpec // what is being transferrednetwork: NetworkSpec // where it livesphaseHandler: PhaseType// which pay-* phase handles itparameters: Record<string, unknown>// rail-type-specificavailability: RailAvailability // operational status (see §9.4.5)governance: { proposedBy: ClaimReference; acceptedAt: number; supersedes?: number }signature: RailSignature // steward's signature (see §9.4.4)}type RailAvailability =| "live"// settlement path runs end-to-end against the network today| "operator_gated"// requires per-operator credential, key, registration, or licensed-agent setup| "closed_data" // network or asset access not publicly available (e.g., permissioned chain)| "bilateral" // requires per-relationship agreement between counterparties| "mocked"// settlement path stubbed; not a production rail| "disabled"// rail present but the steward has marked it not-for-use| "failed"// rail's underlying network or asset path is currently brokentype AssetSpec =| { kind: "erc20"; chainId: number; contract: string; symbol: string; decimals: number }| { kind: "spl"; cluster: "mainnet" | "devnet" | "testnet"; mint: string; symbol: string; decimals: number }| { kind: "native-evm"; chainId: number; symbol: string; decimals: number }| { kind: "native-solana"; cluster: "mainnet" | "devnet" | "testnet"; symbol: "SOL"; decimals: 9 }| { kind: "fiat-via-ap2"; isoCurrency: string; provider: string }| { kind: "stablecoin-cross-chain"; canonicalSymbol: string; routes: CrossChainRoute[] }type NetworkSpec =| { kind: "evm"; chainId: number; rpcAttestation: "consensus-backed-proxy" | "evm-rpc" }| { kind: "solana"; cluster: "mainnet" | "devnet" | "testnet" }| { kind: "ap2-provider"; providerEndpoint: string }| { kind: "x402-resource"; resourceBaseUrl: string }| { kind: "cross-chain"; mechanism: "htlc" | "liquidity-tank" | "substrate-native" }type CrossChainRoute = {sourceChainId: number | stringdestChainId: number | stringhtlcContracts?: { source: string; dest: string }liquidityTankIds?: string[]}

#### 9.4.2 v1 registry contents

The v1 registry contains rail entries for the most-used settlement paths in production. Implementations MUST resolve rails from the canonical addresses listed in the rail-registry index document (dacs4:registry:v1).
| Rail ID | Phase handler | Notes |
| --- | --- | --- |
| evm-erc20:1:USDC | pay-evm-erc20 | Ethereum mainnet USDC |
| evm-erc20:8453:USDC | pay-evm-erc20 | Base mainnet USDC |
| evm-erc20:42161:USDC | pay-evm-erc20 | Arbitrum One USDC |
| evm-erc20:137:USDC | pay-evm-erc20 | Polygon mainnet USDC |
| solana-spl:mainnet:USDC | pay-solana-spl | Solana mainnet USDC |
| solana-spl:mainnet:USDT | pay-solana-spl | Solana mainnet USDT |
| cross-chain-htlc:USDC | pay-cross-chain-htlc | Atomic swap across EVM ↔ Solana for USDC |
| cross-chain-liquidity-tank:USDC | pay-cross-chain-liquidity-tank | Substrate-coordinated atomic settlement; on Demos: Liquidity Tanks. v1 supported routes: ETH Sepolia → Polygon Amoy unidirectional only |
| ap2:visa-direct | pay-ap2 | AP2 mandate to Visa Direct |
| ap2:mastercard-send | pay-ap2 | AP2 mandate to Mastercard Send |
| ap2:stripe-paymentintents | pay-ap2 | AP2 mandate to Stripe PaymentIntents |
| x402:default | pay-x402 | Generic x402 HTTP 402 micropayment |

**v1 cross-chain settlement scope.** pay-cross-chain-liquidity-tank is supported **only** for the live tank routes (ETH Sepolia ↔ Polygon Amoy testnet; USDC; EVM-source unidirectional). All other tank rails are 🟡 to-add and will unlock as Native Bridges Phase 2–4 ship (Solana tanks, bidirectional, additional EVM rails, mainnet deployments, non-USDC stablecoins). pay-cross-chain-htlc is the path the reference implementation runs today (929 LOC reference implementation: Solana Anchor program + Base Sepolia EVM HTLC contract, lock/reveal/refund implemented end-to-end). v1 keeps both first-class.

#### 9.4.3 Rail authoring and resolution

A conforming rail author MUST: (RD-1) sign the rail with the registry steward’s signing key over the domain-separated payload "dacs-rail:v1:" || rail_hash per chapter 7§7.7; (RD-2) anchor the rail via SR-2 at the canonical address; (RD-3) specify registryVersion as monotonically increasing per railId; (RD-4) specify supersedes when replacing a prior rail with the same railId; (RD-5) ensure the railType matches the asset and network kinds (an evm-erc20 rail with a Solana asset MUST be rejected).
A consumer MUST resolve a rail by: reading the rail-registry index from dacs4:registry:v1; looking up the entry for the agreement’s terms.rail.railId; fetching the rail at the indicated anchor and verifying its content hash and signature; if the agreement pins a specific railVersion, MUST use that version; otherwise MUST use the latest at session start, pinned into the session.
**Progressive anchoring for early deployments.** The rail registry follows the same progressive anchoring pattern as the DACS-2 recipe registry (§7.4.4). Phase PA-1 (bootstrap): rails shipped as in-code constants. Phase PA-2 (current): rails anchored by the steward (currently KyneSys Labs) under a single signature. Phase PA-3 (future): rails anchored under multi-signature governance if and when a constituted body is established. Implementations MUST disclose which phase they operate in; consumers MUST verify the rail’s anchoring phase against their own trust requirements.

#### 9.4.5 Rail availability (normative)

Every RailDefinition MUST declare an availability value, with the same value set and semantics as recipe availability (§7.4.5). The value names the rail’s current operational status — what an orchestrator should actually expect when it tries to settle through this rail. Mapping is direct:

**Orchestrator obligations.** (RAV-R1) An orchestrator MUST inspect rail availability before selecting a rail for a session. (RAV-R2) An orchestrator MUST NOT select rails with availability values disabled or failed. (RAV-R3) An orchestrator MAY select rails with availability values operator_gated, closed_data, or bilateral only if the relevant operator-side configuration is in place; this is a runtime preflight check, not a static property of the rail. (RAV-R4) When a rail in an in-flight session transitions from live to failed mid-session, the orchestrator MUST fail the session with errorClass = substrate (rail is non-operational) rather than counterparty.
**Steward obligations.** Same as recipe availability (RAV-5, RAV-6, RAV-7 in §7.4.5) applied to rails. The steward maintains availability values current; transitions are signed and anchored revisions; availability is per-rail-version.

### 9.5 Payment phases

The v1 closed set. Each is a PhaseType from chapter 6’s closed enumeration, with a phase-handler contract conforming to chapter 5’s SessionContext / PhaseHandlerResult.

#### 9.5.1 Common contract

Every pay-* phase handler MUST: (PC-1) accept a PaymentPhaseInput conforming to the shape below; (PC-2) produce SettlementEvidence anchored via SR-2 at dacs4:payment:{jobId}:{railId} (or substrate equivalent); (PC-3) return a PhaseHandlerResult with attestationRef pointing to the evidence; (PC-4) classify outcomes as exactly one of ok: true (payment confirmed at the chain’s finality semantics), ok: false with errorClass: "permanent" (refused by chain, insufficient balance, invalid signature), errorClass: "transient" (RPC failure, mempool congestion), errorClass: "counterparty" (AP2 mandate revoked, x402 server refused), errorClass: "substrate" (SR-2 unavailable), or errorClass: "settlement-atomicity" (cross-chain only; lock succeeded on one side, the other side timed out).
type PaymentPhaseInput = {jobId: stringagreement: AgreementDocument // pinned at commit-agreementrail: RailDefinition // pinned at session startpayer: {bundleHash: stringprimaryClaim: ClaimReferencepayingKey: ClaimReference// MUST appear in payer's bundle.claims}payee: {bundleHash: stringprimaryClaim: ClaimReferencepayeeAddress: string // rail-specific destination}amount: PriceTerm// from agreement.terms.price; rail-validatedsessionContext: SessionContext}

#### 9.5.2 pay-evm-erc20

Single-chain ERC-20 token transfer.
**Procedure.** Resolves rail and verifies asset.kind == "erc20" and network.kind == "evm"; verifies amount.currency matches rail.asset.symbol; computes on-chain amount = amount.amount * 10^rail.asset.decimals (string-decimal multiplication, no float); constructs an ERC-20 transfer transaction: contract.transfer(payee.payeeAddress, amount); submits via the payer’s wallet (or via SR-3 proxy attestation when the payer’s wallet runs server-side); waits for chain finality per rail.parameters.finalityBlocks (default 1 for L2s, 12 for Ethereum mainnet); constructs SettlementEvidence with txRef of kind evm; anchors via SR-2; returns success.
**Failure modes.** payer balance insufficient → permanent; transfer reverts (contract restrictions, paused token) → permanent; chain unavailable → transient; payer-side wallet rejects → counterparty.

#### 9.5.3 pay-solana-spl

SPL token transfer on Solana.
**Procedure.** Resolves rail and verifies asset.kind == "spl"; constructs an SPL Transfer instruction (or TransferChecked for decimal safety); the payee’s token account is the destination (must exist or be created); submits via the payer’s wallet; waits for confirmation per rail.parameters.commitmentLevel (default "confirmed"); constructs SettlementEvidence with txRef of kind solana; anchors via SR-2; returns success.
**Failure modes.** Insufficient balance → permanent; token account does not exist and create-if-missing not allowed → counterparty (payee setup issue); cluster congestion / timeout → transient.

#### 9.5.4 pay-cross-chain-htlc

Atomic cross-chain settlement using HTLC contracts on source and destination chains.
**Procedure.** Resolves rail and verifies asset.kind == "stablecoin-cross-chain" and network.kind == "cross-chain" with mechanism: "htlc"; selects route from asset.routes matching (sourceChainId, destChainId); derives the preimage: preimage = HKDF(jobId || agreementHash || buyerSalt); computes per-chain hashlocks hashlock_source = H_source(preimage), hashlock_dest = H_dest(preimage) where H_source and H_dest are the native hash functions of the source and destination chains respectively (keccak256 on EVM chains, sha256 on Solana and Bitcoin family chains, blake2b on Cosmos family chains); locks payment on source chain by calling htlcContracts.source.lock(payeeAddr, amount, hashlock_source, timelock); waits for source-side finality; reveals on destination chain by the payee calling htlcContracts.dest.claim(preimage); the preimage being revealed on dest allows source-side claim against hashlock_source; payer (or seller, depending on flow) claims on source using the revealed preimage; both txRefs (htlc-lock on source, htlc-reveal on dest) are collected; constructs SettlementEvidence with both txRefs; anchors via SR-2; returns success.
**buyerSalt entropy and lifecycle (normative).** (HTLC-1) buyerSalt MUST be generated from a cryptographically-secure random source with at least 128 bits of entropy. (HTLC-2) buyerSalt MUST NOT be revealed before the source-side lock transaction has reached finality (the preimage is what releases funds; a leaked salt before lock-finality allows an adversary to compute the preimage and claim a pre-funded source position if the contract is mis-configured to accept claims pre-finality). (HTLC-3) buyerSalt MUST NOT be reused across sessions; each jobId uses a freshly-generated salt. (HTLC-4) buyerSalt MUST be retained by the payer until the destination-side claim transaction has reached finality (otherwise refund via timelock is still safe, but in-flight tracking and dispute become harder). (HTLC-5) **Preimage derivation** MUST use HKDF per RFC 5869 with sha256 as the KDF hash function; the IKM is buyerSalt, the salt is jobId, the info is agreementHash. Implementations MUST NOT use weaker preimage derivations. (HTLC-6) **Hashlock computation** is the chain-native hash function applied to the preimage. The source and destination chains MAY use different hash functions (e.g., keccak256 on EVM, sha256 on Solana). Implementations MUST NOT require both chains to share a hash function. The preimage is the only cross-chain-shared value; each side’s hashlock binds against its own native hash of that preimage. The preimage revealed on the destination chain is bit-identical to the preimage that produces hashlock_source under the source chain’s hash function.
**Reference-implementation status.** The DACS reference implementation (agent-commerce-demo, ~929 LOC) currently uses HTLC for fx-rfq cross-chain settlement: a real Solana Anchor program + Base Sepolia EVM HTLC contract, with lock / reveal / refund implemented end-to-end. This predates Native Bridges Phase 1 deployment; the reference will migrate to pay-cross-chain-liquidity-tank as Phase 1 stabilises.
**Failure modes.** Source-side lock fails → permanent (no funds at risk yet); destination-side timeout (payee never claims) → settlement-atomicity, refund path runs (payer reclaims after timelock expires); preimage revealed but source-side claim fails → settlement-atomicity (requires off-chain dispute or manual intervention; SR-5 timelock makes funds eventually recoverable).

#### 9.5.5 pay-cross-chain-liquidity-tank

Substrate-coordinated atomic settlement using pre-funded liquidity primitives. On Demos: Liquidity Tanks.
**Procedure.** Resolves rail and verifies asset.kind == "stablecoin-cross-chain" and network.kind == "cross-chain" with mechanism: "liquidity-tank"; selects liquidityTankIds matching (sourceChainId, destChainId); validates the route is in v1 supported scope (today: ETH Sepolia → Polygon Amoy, USDC, unidirectional); calls the substrate’s native bridge API — on Demos, constructs a BridgeOperation conforming to kynesyslabs/sdks/src/bridge/nativeBridgeTypes.ts with originChainType, destinationChainType, originAddress, destinationAddress, originAmount, originAsset, destinationAsset; submits via demos.bridge.submitBridgeOperation(…); the substrate’s validator shard executes lock-on-source and release-on-dest atomically (within the substrate’s consensus epoch); records the bridge_id (the 16-char hash that is the canonical end-to-end tracking handle); waits for BridgeOperation.status to transition "empty" → "pending" → "completed"; constructs SettlementEvidence with txRef of kind liquidity-tank including bridgeId, both lock and release tx hashes; anchors via SR-2; returns success.
**Trust model.** On Demos, Liquidity Tanks are operated by a rotating Demos validator shard under 2/3 BFT multisig with a 15-day deployer emergency-recovery path. This is "the operator is the substrate itself", not "no operator". The tank contracts (LiquidityTank.sol) are audited (600+ lines) and deployed to ETH Sepolia (0x7AE3A8B899BE0D9E9de51b81a9912C0CEE128d88) and Polygon Amoy (0x57cA16EeE7fbeC69BFD46E4806B5d91e173dd600). Other substrates implementing SR-5 via different mechanisms inherit their own substrate trust model; recipes referencing this rail MUST be evaluated against the relevant substrate’s security profile.
**Failure modes.** Tank insufficiency on dest → transient (retry after re-balancing); source-lock succeeds but substrate epoch interruption prevents dest-release → settlement-atomicity (substrate-native recovery applies per SR-5 implementation; on Demos the 15-day emergency recovery is the backstop); BridgeOperation.status == "failed" → permanent (deterministic rejection by tank shard).

#### 9.5.6 pay-ap2

Payment via an AP2 mandate to a card network or banking provider.
**Procedure.** Resolves rail and verifies asset.kind == "fiat-via-ap2"; constructs an AP2 PaymentMandate conforming to FIDO Alliance AP2 spec (April 2026 onwards); the payer’s AP2-compatible wallet authorises the mandate; the mandate is submitted to the rail.network.providerEndpoint; receives a payment receipt and provider-side reference (e.g., Visa Direct payment id, Stripe PaymentIntent id); constructs SettlementEvidence with txRef of kind ap2 carrying mandateId and providerRef; anchors via SR-2; returns success.
**Failure modes.** Payer’s mandate authorisation refused → counterparty; provider declines the underlying payment (insufficient funds, fraud check, regulatory hold) → permanent; provider endpoint unavailable → transient; mandate revoked between authorisation and submission → counterparty.

#### 9.5.7 pay-x402

Payment via x402 HTTP 402 micropayment to an HTTP resource.
**Procedure.** Resolves rail and verifies network.kind == "x402-resource"; constructs an x402 payment payload (signed authorisation per x402 spec); submits the GET request to the resource with x402 headers; receives the resource response and an x402 receipt; constructs SettlementEvidence with txRef of kind x402 carrying httpResource and paymentReceiptHash (sha256 of the receipt); anchors via SR-2; returns success.
**What pay-x402 adds beyond bare x402.** A direct x402 transaction produces a receipt the client and server hold off-chain; there is no anchored audit trail and the transaction is not bound to a DACS session. pay-x402 binds the x402 transaction into a DACS session by: (a) producing a SettlementEvidence record referencing the x402 receipt hash, anchored via SR-2 — the receipt itself remains off-chain but its hash becomes part of the on-chain bundle; (b) tying the x402 transaction to a specific DACS-3 AgreementDocument via the session’s jobId; (c) making the x402 transaction available to DACS-5 reputation derivation and ERC-8004 publication. For pure HTTP-402 use cases that do not need a session bundle, bare x402 is appropriate; pay-x402 is the right wrapper when the x402 transaction participates in a multi-stage agent commerce lifecycle.
**Failure modes.** Server-side x402 endpoint rejects (insufficient payment, unsupported scheme) → counterparty; HTTP error after payment submitted → transient (retry with idempotency key); payment-receipt signature invalid → permanent.

### 9.6 Delivery phases

The v1 closed set. Each consumes the agreement’s DeliverableRef and produces SettlementEvidence.

#### 9.6.1 deliver-storage-program

Seller writes a Storage Program (SR-2) containing the deliverable payload. Address derived from jobId.
**Procedure.** Validates agreement.terms.deliverable.deliverableType == "storage-program"; seller constructs the deliverable payload conforming to deliverable.schemaUrl (if specified); writes a Storage Program at address dacs4:deliverable:{jobId} with the payload as value; computes contentHash = sha256(canonical_payload); constructs SettlementEvidence with deliverableContentHash = contentHash, deliverableAnchor = {kind: "storage-program", locator: …}; anchors via SR-2; returns success.
**Soft limit.** Storage Programs have a 128 KB cap. Larger payloads MUST use the extended-pointer pattern: the Storage Program at the canonical address contains a pointer record { externalUrl, externalContentHash, segmentRefs[]? }; the actual payload is hosted externally; the externalContentHash binds it. The buyer fetches the pointer, then fetches the payload, then verifies the hash.

#### 9.6.2 deliver-entitlement

Seller issues an EntitlementRecord granting the buyer time-bound access to a service.
**Procedure.** Validates agreement.terms.deliverable.deliverableType == "entitlement"; seller constructs the EntitlementRecord:
type EntitlementRecord = {entitlementVersion: "1"jobId: stringgrantee: ClaimReference// buyer primary claimgrantor: ClaimReference // seller primary claimstartsAt: number // unix msendsAt: number // unix ms; computed from agreement.terms.deliverable.durationSecscope: { service: string; tier?: string; quotas?: Record<string, number> }renewable: booleansignature: ComponentSignature}
Seller signs the EntitlementRecord over the domain-separated payload "dacs-entitlement:v1:" || sha256(canonical_JCS(record_without_signature)) per chapter 7§7.7; anchors the EntitlementRecord via SR-2 at dacs4:entitlement:{jobId}; constructs SettlementEvidence; returns success. Buyer presents the EntitlementRecord (or its hash + anchor) at the service endpoint to access the entitled service. The service endpoint verifies the signature and anchor, checks now is within [startsAt, endsAt], and serves accordingly.
**Renewal.** If renewable: true and the buyer re-pays before endsAt, the seller MAY issue a new EntitlementRecord with extended endsAt and the same jobId. Renewal records MUST be anchored separately.

#### 9.6.3 deliver-attested-payload

Seller delivers a payload whose authenticity is attested via DACS-2 (e.g., the payload is a TLS-attested data fetch).
**Procedure.** Validates agreement.terms.deliverable.deliverableType == "attested-payload"; seller performs the underlying fetch / computation; produces a DACS-2 attestation over the result (using the recipe specified in deliverable.verificationMethod); writes the payload + attestation reference into a Storage Program at dacs4:deliverable:{jobId}; constructs SettlementEvidence carrying deliverableContentHash, deliverableAnchor, and attestationRef pointing at the DACS-2 attestation; anchors via SR-2; returns success.

### 9.7 Settlement evidence

The uniform record produced by every payment and delivery phase. Anchored on the substrate; referenced by DACS-5.
type SettlementEvidence = {evidenceVersion: "1"jobId: stringphase: PaymentPhaseType | DeliveryPhaseTypeoutcome: "success" | "failure"reason?: string// when outcome == "failure"// Payment evidencepaymentTxRefs?: ChainTxRef[]paymentAmount?: PriceTerm// actual settled amountpaymentFee?: PriceTerm // chain or provider fee// Delivery evidencedeliverableContentHash?: stringdeliverableAnchor?: { kind: string; locator: string }attestationRef?: AttestationRef// for deliver-attested-payload// Optional cross-referencesamendmentRefs?: AttestationRef[] // refunds / partial refunds linked hereobservedAt: number // unix mssignature: ComponentSignature// signer is the phase orchestrator}type PaymentPhaseType = "pay-evm-erc20" | "pay-solana-spl"| "pay-cross-chain-htlc" | "pay-cross-chain-liquidity-tank"| "pay-ap2" | "pay-x402"type DeliveryPhaseType = "deliver-storage-program" | "deliver-entitlement" | "deliver-attested-payload"
Canonical form is RFC 8785 JCS of the SettlementEvidence with the signature field omitted. Evidence hash is sha256(canonical_form), hex-encoded. The signature is computed over the domain-separated payload per chapter 7§7.7:
signed_bytes := "dacs-evidence:v1:" || evidence_hash

#### 9.7.1 Refunds and partial refunds

Refunds are not a separate phase type in v1. A refund is modelled as a SettlementAmendment record anchored after the original SettlementEvidence:
type SettlementAmendment = {amendmentVersion: "1"jobId: stringamendsEvidenceRef: AttestationRef// points to the SettlementEvidence being amendedamendmentType: "refund" | "partial-refund" | "correction"refundAmount?: PriceTermrefundTxRefs?: ChainTxRef[]reason: stringobservedAt: numbersignature: ComponentSignature// signed by the refunding party (typically seller)}
SettlementAmendment is anchored via SR-2 at dacs4:amendment:{jobId}:{evidenceHash}:{amendmentIndex}. The amendment signature is computed over the domain-separated payload "dacs-amendment:v1:" || sha256(canonical_JCS(amendment_without_signature)) per chapter 7§7.7. The DACS-5 session record includes amendments in the bundle if they arrive before bundle finalisation.

### 9.8 Cross-chain atomic settlement (SR-5)

Atomic settlement across chains requires SR-5: either substrate-native cross-chain transactions, HTLC contracts on participating chains, or pre-funded liquidity primitives (Liquidity Tanks on Demos).
**Atomicity guarantee.** SR-5 implementations MUST ensure: payment on chain A and value-receipt on chain B succeed together; or both refund / never-take-effect within a bounded time. "Bounded time" is realisation-specific: HTLC — the timelock parameter; Liquidity Tank — the substrate’s consensus epoch (Demos: typically seconds; emergency-recovery backstop: 15 days); substrate-native cross-chain — atomically within the substrate transaction.
**Cross-chain messaging vs settlement.** Cross-chain messaging protocols (Wormhole, LayerZero, Hyperlane, CCIP, Axelar, IBC) carry arbitrary message payloads between chains; they are message-passing primitives. SR-5 is settlement-atomicity: the property that a payment on chain A and a value-receipt on chain B happen together or not at all. A messaging protocol CAN be composed with SR-5 (e.g., a Liquidity Tank implementation may use a messaging protocol internally), but it is NOT itself SR-5. DACS-4 does not register messaging protocols as first-class rails for this reason — the rail surface is settlement, and a "message delivered" outcome does not imply a "value settled" outcome. Substrates whose SR-5 implementation depends on a specific messaging protocol MUST disclose this in the rail definition; the trust model then inherits both the messaging protocol’s and the SR-5 mechanism’s assumptions.
**Choosing a rail.** Cost: HTLC pays gas on two chains; Liquidity Tank typically pays gas only on dest (source-side lock is operator-paid in tank schemes that subsidise gas, including Demos’s current model). Latency: HTLC — source finality + dest finality + claim round-trip (minutes typically); Liquidity Tank — substrate epoch (seconds on Demos). Trust: HTLC — cryptographic / chain consensus only; Liquidity Tank — substrate operator. Listings selecting cross-chain rails SHOULD declare the trust expectations in terms.additionalTerms.

### 9.9 Pipeline composition

A listing’s pipeline declares the order of payment and delivery phases. Common patterns:
**Conformance.** (PIPE-1) A pipeline MUST contain at least one pay-*phase and at least one deliver-* phase. (PIPE-2) Phase ordering MUST be deterministic; the listing’s declared order is normative. (PIPE-3) If a pay-*phase is followed by a deliver-* phase, the deliver-*phase MUST NOT execute until the pay-* phase returns ok: true. (PIPE-4) If a deliver-*phase is followed by a pay-* phase, the pay-*phase MUST NOT execute until the deliver-* phase returns ok: true. (PIPE-5) Pipelines MAY repeat phases (e.g., two pay-* phases for buyer+platform-fee split); each invocation produces independent SettlementEvidence.

### 9.10 Conformance summary

| Role | Requirements |
| --- | --- |
| Rail author | RD-1 through RD-5 |
| Payment phase handler | PC-1 through PC-4; phase-specific procedure |
| Delivery phase handler | §9.6 per-kind procedure; SettlementEvidence emission |
| Pipeline executor | PIPE-1 through PIPE-5 |
| SettlementEvidence consumer | Canonical hash recomputation; signature validation; amendment chain following |

### 9.11 Rationale

**Closed rail registry vs open.** Open rail registries make conformance untestable: a listing could declare a rail no orchestrator implements. The closed v1 set covers the dominant settlement paths in production; new rails ship via the DACS-4 version process under the registry steward (currently KyneSys Labs).
**Uniform SettlementEvidence vs rail-specific evidence shapes.** Rail-specific shapes would force every downstream consumer (DACS-5, auditors, analytics) to handle N shapes. The uniform shape with a discriminated txRefs union keeps consumption simple while preserving per-rail detail.
**Payment and delivery as separate phases vs combined.** Decoupling lets listings compose risk however the seller deems safe: pay-then-deliver for trusted sellers, escrow for distrusted, streamed-entitlement for subscriptions. A combined phase would force every listing into one risk model.
**HTLC and Liquidity Tank as parallel first-class rails.** HTLC is the only fully trust-minimised cross-chain primitive that ships today across heterogeneous chains; the reference implementation already runs on it. Liquidity Tanks are faster and cheaper but trust the substrate operator. Both have legitimate use cases; v1 ships both rather than picking a winner.
**AP2 and x402 as rails vs separate stages.** AP2 and x402 are payment protocols; they fit naturally as rails with their own phase handlers. Modelling them as separate stages would duplicate everything DACS-4 already does (evidence, conformance, error classification) for no gain.
**Refunds as amendments vs separate phases.** Refunds happen post-settlement and may arrive long after the original phase has completed. Modelling them as out-of-band amendments anchored after the original evidence lets sessions close normally even when refunds straggle in. The amendment is included in the bundle if present at bundle time.
**Native bridge / Liquidity Tank trust model disclosure.** Honest disclosure of "operated by a rotating Demos validator shard under 2/3 BFT multisig with 15-day emergency recovery" is the right default. Users picking this rail are choosing speed + cost over trust-minimisation, and the recipe makes that trade-off explicit. Substrates with different SR-5 realisations inherit their own trust models and MUST disclose them similarly.

### 9.12 Backwards compatibility

**ERC-20.** pay-evm-erc20 uses the standard ERC-20 transfer interface; any compliant ERC-20 token works. The rail registry pins specific tokens (e.g. USDC) per chain to avoid scam-token substitution.
**SPL.** pay-solana-spl uses the standard SPL TransferChecked instruction; any compliant SPL token works. The rail registry pins specific mints per cluster.
**HTLC contracts.** Generic HTLC pattern; reference HTLC contracts in the DACS reference implementation are deployed on Base Sepolia and Solana devnet. Other deployments are compatible if they implement lock/claim/refund with the same hashlock-and-timelock semantics.
**AP2.** Compatible with AP2 spec as donated to the FIDO Alliance in April 2026 and subsequent FIDO Alliance versions. Per-provider rail entries (ap2:visa-direct, ap2:mastercard-send, ap2:stripe-paymentintents) pin provider-specific parameters.
**x402.** Compatible with x402 spec as published by Coinbase / Cloudflare / Anthropic. The pay-x402 rail handler is provider-agnostic; specific x402 servers may add parameters via the generic parameters field.
**ERC-8183 escrow (future).** ERC-8183 introduces an EVM-native escrow primitive for job-style transactions. A future v1.1 rail (pay-evm-erc8183) will compose ERC-8183 escrow with DACS-4 evidence; v1 does not include it.
**Substrate-native bridges (Demos Liquidity Tanks).** pay-cross-chain-liquidity-tank on Demos uses Liquidity Tanks per the SDK shape at kynesyslabs/sdks/src/bridge/nativeBridgeTypes.ts. Other substrates implementing SR-5 via native cross-chain transactions (e.g. Polkadot XCM) MAY add their own rails under the cross-chain-liquidity-tank rail type with substrate-specific parameters.

### 9.13 Security considerations

**Re-entrancy on EVM rails.** *Threat:* a malicious ERC-20 hook re-enters the orchestrator during pay-evm-erc20 settlement. *Mitigation:* phase handlers MUST be re-entrancy-safe; the SettlementEvidence MUST be anchored only after the chain transaction is confirmed at finality.
**MEV / front-running on payment txs.** *Threat:* a public-mempool payment can be front-run by MEV bots. *Mitigation:* rail.parameters MAY specify Flashbots-style private mempools or rate-limited public submission. Payment phases SHOULD support submitting via private mempools when available. For high-stakes settlements, cross-chain-liquidity-tank avoids public-mempool exposure entirely.
**Cross-chain settlement-atomicity failure.** *Threat:* HTLC source-lock succeeds but dest-claim never happens; payer’s funds are locked. *Mitigation:* HTLC timelocks let payer reclaim after expiry. Phase handlers MUST track lock expiry and invoke refund automatically. settlement-atomicity error class flags this for DACS-5 reputation logic.
**Liquidity Tank operator compromise.** *Threat:* the substrate validator shard operating Liquidity Tanks is compromised. *Mitigation:* the substrate’s security model (2/3 BFT multisig on Demos, 15-day emergency-recovery on Demos) is the floor. Listings handling high-stakes flows over Liquidity Tanks SHOULD evaluate the substrate’s validator-shard security; for the highest stakes, HTLC is recommended.
**AP2 mandate replay.** *Threat:* an old AP2 mandate is replayed against the provider. *Mitigation:* AP2 mandates include a nonce and an expiry; the provider rejects replays. DACS-4 inherits AP2’s anti-replay properties.
**x402 payment-receipt forgery.** *Threat:* a server claims payment was received when it was not. *Mitigation:* x402 receipts are signed by the payment processor; verifiers MUST validate the signature. Buyer-side x402 wallets SHOULD maintain a local record of submitted payments to detect contested cases.
**Delivery non-delivery.** *Threat:* seller signals payment received, never delivers. *Mitigation:* outside DACS-4’s remit; this is a DACS-3 / DACS-5 issue (the deliver-* phase MUST return ok: false on missing deliverable; DACS-5 records the failure; reputation impact accrues). Listings handling expensive non-recoverable deliveries SHOULD use escrow pipelines (pay-cross-chain-htlc) where the seller’s payment is contingent on demonstrable delivery.
**Refund laundering.** *Threat:* sellers issue refunds to clean up failed deliveries without recording failure. *Mitigation:* SettlementAmendment is anchored and signed; DACS-5 bundles include amendments; the audit trail shows both the original (apparently successful) payment and the (later) refund. Reputation derivation in DACS-5 MUST treat refunded sessions appropriately.
**Decimal-overflow in cross-decimal-system pay paths.** *Threat:* converting a fiat-denominated amount.amount to on-chain integer units overflows or rounds incorrectly. *Mitigation:* PC-1 mandates string-decimal arithmetic with no float. Rail authors MUST specify decimals exactly; phase handlers MUST validate amount.amount precision against rail.asset.decimals (excess precision is an error).
**Pinned-rail vs latest-rail at settle time.** *Threat:* the rail registry changes between agreement commit and settle execution. *Mitigation:* the rail is pinned at session start (per railRegistryVersion in SessionContext). Settle MUST use the pinned rail definition, even if the registry has since superseded it.

### 9.14 Phase parameters reference card

A single-table summary of phase types, their parameters (from listing PhaseStep), and the SettlementEvidence they produce, for implementers.
| Phase type | Parameters (PhaseStep) | Evidence txRef kind |
| --- | --- | --- |
| pay-evm-erc20 | {rail: railId}; rail.parameters.finalityBlocks optional | evm |
| pay-solana-spl | {rail: railId}; rail.parameters.commitmentLevel optional | solana |
| pay-cross-chain-htlc | {rail: railId}; rail.parameters.timelockSec required | htlc-lock + htlc-reveal |
| pay-cross-chain-liquidity-tank | {rail: railId} | liquidity-tank |
| pay-ap2 | {rail: railId}; rail.parameters.providerEndpoint required | ap2 |
| pay-x402 | {rail: railId} | x402 |
| deliver-storage-program | none (driven by listing.offering.deliverable) | n/a (deliverableContentHash + deliverableAnchor instead) |
| deliver-entitlement | none (driven by listing.offering.deliverable) | n/a |
| deliver-attested-payload | none (driven by listing.offering.deliverable) | n/a + attestationRef |

## Chapter 10 — DACS-5: Verify
**Stage:** Verify (5th of 5). **Status:** Draft (part of DACS v0.1). **Depends on:** SR-1 (preferred for cross-substrate primary-claim keying), SR-2 (required for bundle anchoring); composes with ERC-8004 reputation registry as an OPTIONAL publication surface. **Used by:** all subsequent DACS-1 sessions (reputation lookups), external auditors and regulators.

### 10.1 Abstract

DACS-5 specifies how a completed session is anchored, signed, and converted into a reputation signal. It defines:
Reputation is keyed against the **primary identity claim** of the bundle, not against a wallet, signing key, or session pubkey. This prevents low-tier reputation from laundering into high-tier presentations.

### 10.2 Motivation

The Verify stage answers three questions that no other stage answers: *did this transaction actually happen the way the parties say it did?* (a cryptographic, anchored audit trail anyone can inspect); *what did each party think of the other?* (a structured rating from one party about another); *how does this transaction feed into a counterparty’s future reputation?* (a deterministic update rule applied to the bundle of metrics keyed against the party’s primary claim).
Existing open standards address none of these end-to-end. ERC-8004 specifies on-chain reputation entries but says nothing about how the underlying transactions are evidenced. Off-chain rating systems (review-style ratings on marketplaces) exist but are operator-controlled and not portable across platforms. Audit-log standards (e.g., RFC 5424 syslog, OpenTelemetry traces) handle observability but not cryptographic non-repudiation across counterparties.
DACS-5 fills these gaps with three layered artifacts: the session record (working state, the orchestrator’s book), the attestation bundle (the closed audit unit, anchored), and the reputation derivation (the public-facing summary). Each layer is anchored, signed, and verifiable against the earlier stages’ outputs.
A separate concern: **reputation keying**. A party that holds a $0.01 micropayment-tier signing key and accumulates a great reputation should not be able to laundered that into an institutional LEI presentation. DACS-5 keys reputation against the bundle’s primary claim, with separate accumulations per primary-claim tier. The same wallet may hold three primary claims (key:…, did:…, lei:…); each accumulates its own reputation. The DACS-5 derivation algorithm partitions by primary claim and produces tier-distinct metrics.

### 10.3 Session record

The live, mutable state document an orchestrator maintains during a session.
type SessionRecord = {recordVersion: "1"jobId: string// ULID or substrate-equivalentstate: SessionStatelistingRef: { listingId: string; version: number; contentHash: string }parties: SessionParty[]// buyer + seller (+ optionally orchestrator)pipeline: PhaseStep[]phaseResults: PhaseEntry[] // one per executed phasestartedAt: number// unix mslastUpdatedAt: numberendedAt?: number // set on terminal staterecipeRegistryVersion: number// DACS-2 registry pinned at session startrailRegistryVersion: number// DACS-4 registry pinned at session startamendments?: AttestationRef[]// refunds and other amendments}type SessionState =| "draft"| "vet-pending" | "vet-completed" | "vet-failed"| "negotiate-pending" | "negotiate-completed" | "negotiate-failed"| "commit-pending" | "commit-completed"| "settle-pending" | "settle-completed" | "settle-failed"| "rate-pending" | "rate-completed"| "finalised"| "aborted-by-self" | "aborted-by-counterparty"| "substrate-failure-paused"type SessionParty = {role: "buyer" | "seller" | "orchestrator"bundleHash: string // sha256 of the verified IdentityBundleprimaryClaim: ClaimReference // bundle.presentedByvetRecordRef?: AttestationRef// post-Vet}type PhaseEntry = {index: number// position in pipelinestep: PhaseStepinvokedAt: numberresult: PhaseHandlerResultcontextDelta: Record<string, unknown>// merged into running context}

#### 10.3.1 State transitions

Transitions are deterministic. The orchestrator MUST: advance state only when the corresponding phase returns ok: true; on phase ok: false, transition to the appropriate failed-* state and classify per the phase’s errorClass; never reverse state direction within a session (failures end the session; new sessions get new jobIds).
**Substrate-failure pause.** On errorClass: "substrate" (SR-2 or SR-3 unavailable, etc.), the orchestrator MAY transition to substrate-failure-paused and retry per a backoff schedule. Pauses MUST be time-bounded; after a per-listing maximum pause (default 3600 seconds), the session MUST transition to failed-substrate.

#### 10.3.2 Persistence and visibility

SessionRecord is off-chain by default. The orchestrator persists it locally. Counterparties MAY exchange partial views (e.g., the buyer needs to see the seller’s VerifyResultRef from Vet) but each side maintains its own canonical SessionRecord. On bundle production (end of session), the bundle’s contents are derived from the SessionRecord; the SessionRecord itself is not anchored on chain.

### 10.4 Attestation bundle

The frozen end-of-session artifact. Signed by all parties; anchored via SR-2.
type AttestationBundle = {bundleVersion: "1"jobId: stringoutcome: "completed" | "failed-perm" | "failed-counterparty" | "failed-substrate" | "aborted-by-self" | "aborted-by-other"listingRef: { listingId: string; version: number; contentHash: string }agreementRef?: AttestationRef // present iff outcome != "aborted-by-*" before commit-agreementparties: BundleParty[]phaseSummary: BundlePhaseEntry[]vetRecords: AttestationRef[]// composite verification recordssettlementEvidence: AttestationRef[]amendments?: AttestationRef[]ratingRefs?: AttestationRef[] // when the rate phase ranrecipeRegistryVersion: number // DACS-2 registry pinned at session startrailRegistryVersion: number // DACS-4 registry pinned at session startfinalisedAt: numbersignatures: BundleSignature[] // both buyer and seller (and orchestrator if separate)}type BundleParty = {role: "buyer" | "seller" | "orchestrator"bundleHash: stringprimaryClaim: ClaimReference}type BundlePhaseEntry = {index: numberkind: PhaseTypeoutcome: "ok" | "fail"errorClass?: "permanent" | "transient" | "counterparty" | "substrate" | "settlement-atomicity"txRefs?: ChainTxRef[]attestationRef?: AttestationRef}type BundleSignature = {party: ClaimReference // primary claim of the signeralgorithm: "ed25519" | "ecdsa-secp256k1" | "sr1-aggregate"value: string // signature over bundle hash}

#### 10.4.1 Canonical serialisation, hash, and domain-separated signature

Canonical form is RFC 8785 JCS of the bundle with signatures field omitted. Bundle hash is sha256(canonical_form), hex-encoded. Each BundleSignature.value MUST be computed over a domain-separated payload:
signed_bytes = "dacs-bundle:v1:" || bundleHash.value
The "dacs-bundle:v1:" string prefix prevents cross-protocol signature confusion: an attacker capturing a bundle signature MUST NOT be able to replay it as a listing signature, agreement signature, or any other DACS signature even if the hash bytes collide. Verifiers MUST recompute the canonical form, the bundle hash, the prefixed signed_bytes, and verify each signature against the appropriate party’s primary-claim key. Required signers: buyer + seller. If the orchestrator is a distinct party (not buyer or seller), the orchestrator signature is also REQUIRED. Bundles missing any required signature MUST be rejected by consumers.

#### 10.4.2 Anchoring

The bundle MUST be anchored via SR-2. **Two-sided anchoring scheme.** Each signing party (buyer, seller, and orchestrator if distinct) anchors its own bundle at a party-specific address: stor-{sha256(jobId + "-bundle-" + role)} where role is "buyer", "seller", or "orchestrator". In the happy case both sides’ bundles are canonically equal and consumers can read either; in the divergence case both sides are independently retrievable for dispute purposes (see §10.4.3).
Bundles MUST fit within the substrate’s storage-cap soft limit (128 KB on Demos Storage Programs).
**Extended-pointer pattern for large sessions.** Sessions with extensive evidence (large transcripts, attestation chains, multi-party verifications, e.g. a sealed-envelope auction with 50 bidders’ commits and reveals) MAY exceed the size cap. In that case the bundle at the canonical address contains a pointer record:
type BundleExtendedPointer = {bundleVersion: "1"pointerKind: "extended"fullBundleUrl: stringfullBundleContentHash: stringsegmentRefs?: AttestationRef[]// optional segmented anchoringsignature: ComponentSignature}
and the full bundle is hosted externally; fullBundleContentHash binds it. Consumers MUST verify the external bundle’s hash against the on-chain pointer before treating it as authoritative.

#### 10.4.3 Bundle production rules

A bundle MUST be produced when the session reaches a terminal state. The bundle MUST include references to all DACS-2 composite verification records, DACS-3 agreement (if any), DACS-4 settlement evidence (one entry per executed phase), DACS-4 amendments (refunds), DACS-5 ratings (if the rate phase ran). The bundle MUST NOT include references to any record outside the session’s scope.
**For sessions terminating before commit-agreement** (aborted-by-self/other in Vet or Negotiate), the bundle MUST include the available vetRecords and a phaseSummary marking the failed phase; agreementRef is omitted.
**For sessions terminating with failed-substrate**, the bundle’s outcome captures the substrate failure; the failure does not count as either party’s fault in DACS-5 reputation derivation.
Two parties producing independent bundles for the same session MUST converge on identical bundle content (by canonical-form equality) or MUST surface the divergence as a dispute. Each side anchors its own bundle at its own derived address; a consumer looking up "the bundle(s) for session X" MUST query both sides’ expected addresses. The canonical addresses for the two sides are stor-{sha256(jobId + "-bundle-buyer")} and stor-{sha256(jobId + "-bundle-seller")} (or substrate-equivalent two-sided addressing). Consumers MUST: (a) fetch both addresses; (b) if exactly one bundle is present, classify the missing side as aborted-by-self per the bundle-suppression rule in §10.11; (c) if both are present and canonically equal, treat as the unified session bundle; (d) if both are present and canonically diverge, treat the session as disputed — each bundle stands on its own signatures and consumers must decide policy (e.g., trust the buyer’s bundle for buyer-reputation, the seller’s for seller-reputation; or flag for human review). v1 does not specify a dispute resolution path; divergence is handled out-of-band. A future minor version (DACS-X, dispute) may specify selective transcript disclosure under signed party agreement or arbitrator order.

### 10.5 Reputation derivation

A deterministic function from a set of attestation bundles to a small set of headline reputation metrics, keyed by primary claim.
type ReputationDerivation = {derivationVersion: "1"partyPrimaryClaim: ClaimReference// the party being scoredwindowStart: number// unix mswindowEnd: number// unix msbundleCount: numbermetrics: {completionRate: number | null// null when bundleCount == 0counterpartyDisputeRate: number | nullaverageBuyerRating: number | nullaverageSellerRating: number | nullobservedTransactionalVolume: PriceTerm[] // sum of agreement.terms.price, by currency}computedAt: numberbundleRefs: AttestationRef[] // the bundles aggregated}

#### 10.5.1 Derivation algorithm

derive(party, bundles, windowStart, windowEnd):scoped := [b for b in bundleswhere party in b.parties.primaryClaimAND windowStart <= b.finalisedAt <= windowEnd]if scoped is empty:return ReputationDerivation with bundleCount=0, all metrics nullcompleted := [b for b in scoped where b.outcome == "completed"]failed_perm := [b for b in scoped where b.outcome == "failed-perm"]failed_counterparty := [b for b in scoped where b.outcome == "failed-counterparty"]failed_substrate := [b for b in scoped where b.outcome == "failed-substrate"]aborted_by_self := [b for b in scoped where b.outcome == "aborted-by-self"AND aborter == party]aborted_by_other := [b for b in scoped where b.outcome == "aborted-by-other"AND counterparty_aborted_against == party]party_fault_count := |aborted_by_self| + |failed_perm where party_at_fault|counterparty_fault_count := |aborted_by_other| + |failed_counterparty against party|party_fault_denom := |scoped| − |failed_substrate|completionRate := |completed| / party_fault_denom when party_fault_denom > 0 else nullcounterpartyDisputeRate := counterparty_fault_count / party_fault_denomsame gate# Collect ratings by fetching each bundle's referenced rating recordsratings_targeting_party_as_seller := []ratings_targeting_party_as_buyer := []for b in scoped:for ratingRef in (b.ratingRefs or []):r := fetch_and_verify_rating(ratingRef) // RatingRecordif r.target == party AND r.targetRole == "seller":ratings_targeting_party_as_seller.append(r.value)if r.target == party AND r.targetRole == "buyer":ratings_targeting_party_as_buyer.append(r.value)averageSellerRating := mean(ratings_targeting_party_as_seller)when ratings_targeting_party_as_seller else nullaverageBuyerRating:= mean(ratings_targeting_party_as_buyer)when ratings_targeting_party_as_buyer else nullvolume := groupSumByCurrency(b.agreementRef.terms.pricefor b in scoped where agreementRef present)return ReputationDerivation with computed metrics
"party_at_fault" is recorded in the bundle’s phaseSummary errorClass (counterparty implies the other party; permanent on a non-cross-chain rail with no settlement-atomicity flag and a successful pre-pay state generally implies the local party at fault). The classification rules are spelled out in the per-phase errorClass tables in chapters 7 and 9. **failed-substrate sessions** are excluded from the party-fault denominator: party_fault_denom = |scoped| − |failed_substrate|. This ensures substrate-induced failures do not damage either party’s reputation. Metrics with denominator > 0 produce numeric values; metrics with denominator == 0 (e.g., bundleCount=0, or all sessions failed-substrate) produce null — distinct from zero, signalling "no signal" rather than "zero signal". The averageBuyerRating / averageSellerRating metrics are computed by walking each bundle’s ratingRefs, fetching the referenced RatingRecord, verifying its signature, and aggregating the values whose target matches the scored party; the metric is null when no qualifying ratings exist.

#### 10.5.2 Per-primary-claim keying

The same wallet may hold multiple primary claims (key:…, did:…, lei:…). DACS-5 reputation is computed *per primary claim*. A great reputation against key:0xabc... does NOT inherit into a brand-new lei:984500ABCDEF… presentation, even though the same wallet may control both. Consumers querying reputation MUST query with the specific primary claim used in the current bundle’s presentedBy, not a wallet identifier or session pubkey. SR-1 (cross-substrate identity aggregation) is the substrate primitive that makes the wallet ↔ multi-primary-claim relationship explicit, allowing consumers to optionally surface "this party also has reputation under primary claim X" — informationally, NOT as inheritance.

#### 10.5.3 Computation surfaces

Derivation MAY be computed: (a) lazily by a querying party (over a set of bundles they fetched themselves — highest trust); (b) by a DACS-5 catalog operator (similar to a DACS-1 catalog — indexed for performance, but consumers MUST verify against the underlying bundles for high-stakes decisions); (c) on chain via an ERC-8004 reputation registry write per §10.7. Each surface is a different point on the trust / performance trade-off; the algorithm is the same.

### 10.6 The rate phase (optional)

A DACS-5 phase that produces structured ratings between parties at session end.
type RatingRecord = {ratingVersion: "1"jobId: stringrater: ClaimReference// primary claim of the rating partytarget: ClaimReference // primary claim of the rated partytargetRole: "buyer" | "seller"value: number// 1..5 inclusive integerfreeText?: string// optional; max 1000 charsdimensions?: Record<string, number>// optional per-dimension scores (timeliness, communication, etc.)ratedAt: numbersignature: ComponentSignature}

#### 10.6.1 Phase contract

rate is OPTIONAL in a pipeline. When present, the phase MUST: run after all settle-* phases complete with ok: true; produce one RatingRecord per direction (buyer→seller, seller→buyer); sign each RatingRecord over the domain-separated payload "dacs-rating:v1:" || sha256(canonical_JCS(record_without_signature)) per chapter 7§7.7; anchor each RatingRecord via SR-2 at dacs5:rating:{jobId}:{raterClaim}; include both ratingRefs in the bundle. Sellers and buyers MAY decline to rate; absence of a rating does not block bundle production. The pipeline step parameters MAY specify { required: true | false } per side.

### 10.7 ERC-8004 publication surface

DACS-5 bundles can OPTIONALLY be reflected to the Ethereum ERC-8004 reputation / validation registries for EVM-side consumers.

#### 10.7.1 Mapping

When a party holds an erc8004 claim in their bundle, the publisher MAY write a reputation/validation registry entry referencing the bundle anchor. The publisher MUST: include in the registry entry the bundleAnchorLocator and bundleContentHash; sign the registry write with the key that owns the ERC-8004 token; rate-limit registry writes to avoid spam (suggested: at most one write per session per direction).

#### 10.7.2 Consumption

EVM-side consumers MAY read ERC-8004 entries as a discovery surface for DACS-5 bundles. They MUST fetch the referenced bundle and validate it independently. The ERC-8004 entry is a pointer, not a substitute for the bundle.
**Substrate decoupling.** Publication to ERC-8004 is OPTIONAL and is a Demos-to-Ethereum cross-pollination convenience. Other substrates MAY define equivalent publication surfaces (e.g., a Solana reputation program, a Bitcoin OP_RETURN scheme). DACS-5 does not require any particular publication surface; the bundle is the canonical artifact.

### 10.8 Conformance summary

| Role | Requirements |
| --- | --- |
| Orchestrator | Maintain SessionRecord per §10.3; transition states deterministically; produce bundle on terminal state |
| Bundle producer | Sign per §10.4.1; anchor per §10.4.2; include all required references per §10.4.3 |
| Bundle consumer | Recompute canonical hash; verify domain-separated signatures; dereference and validate every contained AttestationRef |
| Reputation deriver | Apply algorithm in §10.5.1 verbatim; partition by primary claim; treat failed-substrate per the denominator rule; return null for zero-denominator metrics |
| Rate phase handler | One RatingRecord per direction; anchor each; include in bundle |
| ERC-8004 publisher (optional) | §10.7.1 mapping; rate-limit writes; sign with token-owner key |

### 10.9 Rationale

**Session record off-chain by default vs anchored.** Anchoring the live session record every state transition would dominate session economics for no audit benefit (the bundle captures what auditors need; the intermediate state is operational noise). Off-chain SessionRecord, on-chain bundle, is the right split.
**Bundle as the unit of audit vs individual phase records.** Each phase already anchors its evidence; the bundle is the unifying envelope. Auditors verifying a session need ONE artifact to start from; from there they walk the references. Without a bundle, every consumer would have to reconstruct the session graph from disparate anchors.
**Domain-separated bundle signature.** A bundle signature must not be confused with a listing signature, agreement signature, or any other DACS signature even when hash bytes collide. The "dacs-bundle:v1:" prefix prevents cross-protocol replay at zero cost. The bundle separator is part of the universal DACS signature scheme defined in chapter 7§7.7; every DACS v1 artifact uses its own separator from the same registry.
**Per-primary-claim reputation vs wallet-keyed.** Wallet-keyed reputation lets a great key:0xabc... reputation laundered into a brand-new lei:… presentation. Per-primary-claim keying prevents this. The same wallet honestly holding multiple claims accumulates separate reputations per tier; consumers can surface the cross-claim relationship (via SR-1) informationally without inheritance.
**Substrate-failure exclusion from party-fault denominators.** If a session fails because the substrate was down, neither party did anything wrong; counting it as either party’s fault would create incentives for parties to avoid sessions during substrate strain. Excluding failed-substrate from the denominator keeps the metric honest.
**Null metrics vs zero metrics.** A new party with bundleCount=0 has no signal, not a zero signal. Zero would imply "completed 0% of sessions" — misleading. Null forces consumers to handle "no data" deliberately rather than treating new parties as worst-rated.
**Optional rate phase vs required.** Forcing every session to produce ratings would create rating noise (parties post 5-stars to avoid friction) and would expose parties to rating retaliation. Optional rating, with both sides able to decline, matches institutional practice and platform-marketplace norms.
**ERC-8004 as optional publication vs required.** ERC-8004 is the dominant EVM reputation registry, but DACS-5 ships on substrates without an Ethereum-mainnet write path. Optional publication is the right scope.
**Extended-pointer pattern for oversized bundles.** Real sessions occasionally produce bundles larger than the substrate’s storage-program cap (multi-party auctions, sessions with large attestation chains). Hard-failing such sessions would force a different audit format; the extended-pointer pattern keeps the canonical artifact at the on-chain address and ferries the rest off-chain with content-hash binding.

### 10.10 Backwards compatibility

**ERC-8004 reputation / validation registries.** §10.7 specifies the publication surface. DACS-5 inherits ERC-8004’s read semantics for EVM consumers; ERC-8004 itself is unchanged.
**Existing operator-marketplace ratings.** A marketplace migrating to DACS-5 can import its historical rating data as a one-time backfill of RatingRecord-equivalent entries signed by the marketplace operator. New ratings produced under DACS-5 stand alone; historical ratings stand under the marketplace’s signature and are clearly distinguishable.
**Audit-log standards.** A consumer that wants to feed a DACS-5 bundle into an existing audit-log pipeline (RFC 5424, OpenTelemetry) can convert the bundle to those formats at read time. DACS-5 itself defines only the bundle.
**Future DACS-X (dispute).** A future standard covering dispute resolution is anticipated. It will compose with DACS-5 by referencing bundles and producing dispute records; v1 of DACS-5 is forward-compatible with this addition (the bundle’s phaseSummary and outcome fields are sufficient for dispute proceedings to consume).

### 10.11 Security considerations

**Bundle forgery.** *Threat:* an attacker produces a fake bundle claiming a session that did not happen, hoping to influence reputation. *Mitigation:* the bundle must be co-signed by both parties; signatures use domain-separated payloads; consumers verify both signatures against the parties’ verified primary claims. A unilateral bundle cannot influence the counterparty’s reputation.
**Bundle suppression.** *Threat:* a party who performed badly in a session refuses to sign the bundle, hoping to prevent its publication. *Mitigation:* the non-signing party’s outcome (aborted-by-self) is recorded in the counterparty’s bundle attempt; consumers seeing a bundle with only one signature MUST classify the session as aborted-by-self for the non-signer and aborted-by-other for the signer. The non-signer’s reputation takes the appropriate hit even if they refuse to sign. (Implementation note: a one-sided bundle MUST follow exactly the same canonical form and signing rules; the absence of the counterparty’s signature is what flags the outcome.)
**Sybil reputation farming.** *Threat:* an attacker creates many cheap primary claims (key:…) and farms self-deal reputation between them. *Mitigation:* DACS-5 metrics are partitioned by primary claim and do not inherit; Sybil farming over key:… claims accumulates reputation only against those claims, not against higher-tier presentations. The DACS-2 supplementary signals (counterparty being a known Sybil cluster) feed back into Vet for any party who cares.
**Replay across sessions.** *Threat:* an attacker captures a signed bundle and replays it as a different session’s bundle. *Mitigation:* the bundle includes jobId; the signature payload includes the bundle hash which includes jobId. Replay against a different jobId fails verification.
**Cross-protocol signature confusion.** *Threat:* a bundle signature is replayed as some other DACS signature (listing, agreement) where the underlying hash bytes happen to align. *Mitigation:* the universal signature scheme in chapter 7§7.7 defines per-artifact domain separators across the entire DACS v1 stack; the bundle domain separator is "dacs-bundle:v1:" and other artifact kinds use their own separators per the table in §7.7. A signature produced under any artifact kind cannot validate as a signature under any other kind, even when the hash bytes coincide.
**Reputation poisoning via collusion.** *Threat:* two colluding parties run many fake sessions to inflate each other’s reputation. *Mitigation:* this is fundamentally hard to prevent at the protocol level. DACS-5 mitigates by per-primary-claim keying (collusion inflates only one tier of reputation), by transactional-volume reporting (consumers can see if a party’s reputation comes from many tiny sessions vs few large ones), and by composability with external signal sources. Consumers handling stakes worth the cost of collusion SHOULD weigh DACS-5 metrics against external signals.
**Orchestrator misclassification of errorClass.** *Threat:* the orchestrator classifies a counterparty failure as a substrate failure (or vice versa) to bias reputation. *Mitigation:* the bundle phaseSummary carries the errorClass; both parties sign the bundle; a party that disagrees with the classification refuses to sign, producing aborted-by-other. The honest party’s independent bundle (with their own classification) is the source of truth for their reputation. Persistent classification disputes are a DACS-X concern.
**Bundle anchor unavailability.** *Threat:* the SR-2 anchor becomes unreadable after the session ends (e.g. storage program purged, IPFS unpinned). *Mitigation:* on-substrate anchoring (Demos Storage Programs) provides indefinite availability under substrate operation. Off-substrate anchoring (IPFS, HTTPS) is best-effort. Listings concerned with long-term auditability SHOULD use on-substrate anchoring for bundles regardless of which surface the rest of the session uses.
**Time-bound reputation windows.** *Threat:* an old, no-longer-representative reputation is presented as current. *Mitigation:* derivations are window-bounded; consumers querying reputation MUST specify a window and SHOULD weight recent windows more heavily. The algorithm does not specify weighting (consumers choose); it does require explicit window bounds in every derivation.
**ERC-8004 write spamming.** *Threat:* an attacker writes many fake ERC-8004 entries pointing at fabricated bundles. *Mitigation:* ERC-8004 entries are pointers; consumers MUST fetch and validate the bundle. Fake bundles fail at validation. The cost of writing many ERC-8004 entries (gas) is a natural rate limit; DACS-5 publishers SHOULD additionally enforce per-session rate limits.
## Chapter 11 — Stewardship, versioning, follow-on

### 11.1 Stewardship and versioning

#### 11.1.1 Current steward

DACS v1 is stewarded by **KyneSys Labs**. This means: the registry signing key currently used to sign recipes (DACS-2) and rail definitions (DACS-4) is held by KyneSys Labs; the canonical anchored addresses for those registries are written by KyneSys Labs; spec changes between minor versions are reviewed and merged by KyneSys Labs. This is a single-steward arrangement — phase PA-2 in the progressive-anchoring scheme defined in §7.4.4. It is **not** the long-term governance target; it is the honest description of where v1 sits at time of publication.
Multi-party governance — a constituted working group, formal multi-signature schemes for the registries, sub-authority delegation by domain (sanctions lists, financial regulation, settlement rails) — is open work. v1 ships under single-steward semantics so the standard can move forward; transitioning to a multi-party arrangement is anticipated as the ecosystem of implementers, reviewers, and operators grows. The PA-2 → PA-3 transition (§7.4.4) is the formal anchor point for that change.
Implementations consuming the registries MUST disclose to their users which signing key they treat as authoritative and MUST NOT misrepresent the current steward as a constituted multi-party body. Third-party implementations (such as PATH-OS Labs’ reference) MAY operate against the same canonical registries; the steward arrangement governs who writes the registries, not who reads them.

#### 11.1.2 Versioning

DACS uses a single document-level version. The entire standard — all five per-stage standards, front-matter substrate-binding, threat model, glossary, conformance plan — versions together. This document is **DACS v0.1**, the first publicly released version. Major versions (v1, v2, …) break compatibility; minor versions (v0.2, v0.3, …) add capabilities while preserving forward-readable shapes. v0.1 freezes the registries (claim schemes in DACS-1, methods/recipes in DACS-2, patterns in DACS-3, rails in DACS-4); additions happen via minor-version registry updates released by the current steward. v1 is the version at which the standard is considered ready for unsupervised production use.

#### 11.1.3 Conformance philosophy

Each spec’s conformance section enumerates the requirements an implementation must satisfy to claim conformance to that spec. Cross-spec conformance (a full DACS-1…DACS-5 implementation) is the conjunction of per-spec conformance for every spec the implementation covers. Implementations MAY cover a strict subset (e.g., DACS-1 + DACS-4 only, for a payment-rail aggregator that does not negotiate or rate); conformance is then to the implemented subset.

#### 11.1.4 Substrate stance

DACS does not standardise the substrate. The substrate-capability statements (SR-1 through SR-5) are the abstract contract. Any substrate that provides them can host a compatible implementation. Demos is the substrate against which DACS was designed and ships all five capabilities natively; other substrates (Ethereum L1+L2 stack with bridges, Polkadot, Cosmos with privacy zones) MAY satisfy varying subsets and host correspondingly varying DACS subsets. SR-1, SR-2, and SR-5 are protocol-specified; SR-3 and SR-4 are trust-property specified in v1, with wire-protocol harmonisation expected in v2.

#### 11.1.5 Composition stance

DACS composes with the existing open ecosystem and does not seek to replace standards that already work. Where existing standards have gaps relevant to agent commerce (negotiation patterns, end-to-end audit), DACS specifies new standards as narrowly as possible, with explicit substrate dependencies. The composed-standards table in chapter 8 (front matter) is the comprehensive list of touchpoints; when an underlying standard updates, the corresponding DACS standard’s registry entry updates in the next minor version.

### 11.2 Follow-on topics

Five areas are deliberately out of scope for v1 and intended for subsequent standards.

#### 11.2.1 Dispute resolution (DACS-X, anticipated)

v1 produces signed, anchored bundles. v1 does not specify what happens when parties disagree about a bundle’s contents, contest a settlement amendment, or wish to invoke an arbitrator. A follow-on standard (working name DACS-X) is anticipated to: specify a dispute initiation phase referencing one or more bundles; specify selective transcript disclosure protocols (revealing channel transcripts to a named arbitrator under signed party agreement); specify arbitrator credentialing patterns (likely composing DACS-1 + DACS-2 — arbitrators are agents with verified credentials); specify dispute outcome bundles that supersede or annotate the original session bundles.

#### 11.2.2 Open phase set

v1’s phase types are closed across DACS-2/3/4/5. v2 may relax this to permit ecosystem-defined phases under the steward’s oversight. Until then, x- experimental phases provide an escape valve for out-of-band agreement.

#### 11.2.3 Multi-party transactions beyond bilateral

v1 negotiation is bilateral (except sealed-envelope, which is one seller / many bidders). True multi-party transactions — syndicated trades, multi-seller bundles, escrow-with-arbitrator three-party flows — are out of scope. DACS-3 v1.1 will likely add a negotiate-multi-quote pattern; truly multi-party flows are likely v2 territory.

#### 11.2.4 Streaming / continuous-flow rails

v1 rails are discrete-transaction. Streaming payment rails (Sablier-style, payment per second of usage) and continuous-delivery rails (per-second compute, per-byte data feed) are out of scope. A future DACS-4 v1.1 entry (rail type continuous) is anticipated.

#### 11.2.5 Cross-DACS-version compatibility

v1 specifies forward-compatibility within each per-stage standard (a v1.1 reader handles v1.0 bundles). Cross-version compatibility (a DACS-1 v2 listing pipelined against a DACS-3 v1 negotiator) is deferred; pipelines MUST currently use a coherent set of per-stage versions.

#### 11.2.6 Multi-party governance and registry stewardship

The transition from single-steward (PA-2) to multi-party constituted governance (PA-3) for the recipe and rail registries is itself follow-on work. v1 does not specify the constitution mechanism, multi-signature thresholds, sub-authority delegation, or transition procedure. These are open questions for the working group that the ecosystem chooses to constitute. Until that body exists, the current steward operates under the disclosure rules in §11.1.1.

### 11.3 Closing

Agent commerce is moving from prototype to production. DACS is a contribution toward keeping the lifecycle on public infrastructure: a stack that composes with the existing open standards where they work, fills the gaps where they don’t, and makes substrate dependencies explicit. A reference implementation runs the lifecycle end-to-end on the Demos substrate; an independent third-party reference implementation (PATH-OS Labs’ pathos-dacs-ref) implements the DACS-1 + DACS-2-GLEIF + DACS-5 verifier subset against the same spec.
What this document is **not**: a finished standard ready for unsupervised production at every scale. The honest list of remaining work — beyond the per-stage follow-on topics in §11.2 — includes: protocol-level wire specifications for SR-3 and SR-4 (currently trust-property specified only); expansion of independent reference-implementation coverage beyond the current third-party verifier; engagement with the maintainers of every composed standard (ERC-8004, AP2 via FIDO Alliance, W3C VC, A2A) to convert "DACS composes with X" from a unilateral claim into a documented cross-maintainer conversation; a unified threat-model audit (§12) reviewed by parties outside the current stewardship; constitution of multi-party governance (§11.2.6); and conformance test suites (§14) ready for implementers to run against.
Some of these will reveal gaps that need new work, not just refinement. The intent of v1 is to ship a coherent baseline that the next 6–12 months of implementation experience and ecosystem engagement can sharpen. It is not the final word on agent commerce.
## Chapter 12 — Unified threat model
This chapter collects, partitions, and rationalises the per-chapter security considerations into a unified threat model. It is the artifact a security review would start from. Where this chapter restates per-chapter threats, the per-chapter mitigation is normative; this chapter’s framing is informative.

### 12.1 Scope and non-goals

DACS’s security goals are: (a) cryptographic non-repudiation of every per-session artifact (listings, bundles, agreements, evidence) by the parties that produced them; (b) tamper-evident audit trail — any modification of an anchored artifact is detectable by content-hash comparison; (c) limited-trust substrate dependency — the substrate is trusted for liveness and consensus per its own security model, not for application-layer semantics; (d) prevention of cross-protocol signature confusion via the universal domain-separation scheme in §7.7; (e) prevention of replay across sessions via per-session jobIds, nonces, and content hashes; (f) substrate-failure isolation in reputation derivation so substrate outages do not damage party reputations.
Non-goals: DACS does **not** prevent collusion between buyer and seller (two parties who jointly fabricate a session produce a valid session; the audit trail records what they say happened, not what objectively happened). DACS does **not** prevent denial-of-service by a counterparty or by the substrate (it produces evidence of the failure for reputation purposes, but does not guarantee progress). DACS does **not** prevent regulatory non-compliance — it produces artifacts useful for compliance audit but does not enforce any specific regulatory regime. DACS does **not** provide unconditional privacy — the SR-4 channel contents stay between members, but member identity and timing of commitments are visible on the public chain by design.

### 12.2 Adversary model

The threat model assumes adversaries with the following capabilities; per-threat mitigations specify which class is being defended against.
| Adversary class | Capabilities | Assumed not capable of |
| --- | --- | --- |
| Network observer | Reads all public-chain traffic; can perform timing analysis on commitments; cannot read private-channel contents. | Breaking standard cryptographic primitives (sha256, Ed25519, ECDSA-secp256k1 under standard assumptions). |
| Network active attacker | Can drop, delay, reorder, or inject messages on transport links; can MITM TLS sessions if PKI is compromised. | Forging signatures by valid private keys; producing sha256 preimages. |
| Malicious counterparty | Operates a fully-conformant DACS implementation but maximises self-interest within the protocol; signs everything they’re willing to be held to and refuses to sign anything else. | Forging the other party’s signatures; controlling validator-set consensus on the substrate. |
| Compromised authority | A registry authority (GLEIF, FINRA, OFAC, etc.) returns false data, either by deliberate compromise or by API corruption. | Forging substrate-validator signatures over the false response (validators sign the fetch result, not the data’s factuality). |
| Substrate validator-set minority | A minority of substrate validators is compromised; the consensus protocol’s normal Byzantine bound holds. | Producing valid consensus signatures on falsehoods (assuming the BFT assumption holds). |
| Substrate validator-set majority | A majority of substrate validators is compromised. | (none — above the substrate’s security floor; DACS inherits the failure.) |
| Substrate operator (rotating shard) | For Liquidity Tank-style SR-5: the rotating validator shard operating tanks is partially compromised within its multisig threshold. | Bypassing the 2/3 BFT multisig or the deployer emergency-recovery path. |
| Channel operator (SR-4) | Operates the private negotiation channel; can drop messages, fork views, observe all member messages. | Forging member signatures inside the channel; observing messages encrypted to a member subset (with realisation-appropriate encryption). |
| Recipe-registry attacker | Compromises the recipe registry signing key, attempts to push poisoned recipes. | Backdating recipe-version pinning; affecting already-pinned sessions. |
| Sybil attacker | Generates unlimited low-tier (key:…) identities and farms self-deal reputation. | Generating authority-issued claims (lei, finra-crd, etc.) without owning the underlying registrations. |

### 12.3 Trust boundaries

A reader following the audit trail from a DACS-5 bundle backwards through the lifecycle crosses these trust boundaries; each boundary has its own assumptions.

### 12.4 Threat catalogue

Every per-chapter security threat, indexed by adversary class and mitigation status. The threats are stated normatively in the per-chapter sections; this is the cross-reference.
| Threat | Primary adversary | Where mitigated | Status |
| --- | --- | --- | --- |
| Forged listing | malicious counterparty | §6.6 + §7.7 (signatures + domain separator) | mitigated |
| Bundle replay | network observer | §6.3.2 (session nonce) + §6.6 | mitigated |
| Catalog poisoning | malicious infrastructure | §6.3.6 (clients dereference anchors) | mitigated |
| Identity-claim substitution | malicious counterparty | §6.6 (pinned bundle hash) | mitigated |
| Method substitution | malicious verifier | §7.12 (method field comparison) | mitigated |
| Recipe poisoning | recipe-registry attacker | §7.12 (signed recipes + pinned recipeVersion) | mitigated |
| Substrate validator capture (SR-3) | substrate validator-set majority | §7.12 (multi-method alternatives) | partial — v1.1 strengthening planned |
| VerifyResult replay | malicious verifier | §7.12 (identifier + bundle hash binding) | mitigated |
| TOCTOU authority change | time | §7.12 (maxAge tightening) | parameter-driven |
| Indeterminate exploitation | malicious counterparty | §7.5.1 + §7.7.1 (aggregation) | mitigated |
| Channel-operator censorship | channel operator | §8.12 (CH-4 liveness detection) | mitigated (substrate-dependent) |
| Channel-operator forking | channel operator | §8.12 (monotonic sequence + cross-check) | mitigated |
| Offer replay across sessions | network observer | §8.12 (channelId in envelope) | mitigated |
| Cross-artifact signature replay | malicious counterparty | §7.7 universal domain separators | mitigated |
| Sealed-envelope front-running | competing bidder | §8.12 (hash commitment + private channel) | mitigated |
| Sealed-envelope post-deadline submission | malicious counterparty | §8.4.3 (chain-timestamp anchoring) | mitigated |
| Agreement-listing mismatch | malicious counterparty | §8.5.2 (validation in commit-agreement) | mitigated |
| Re-entrancy on EVM rails | malicious counterparty | §9.13 (phase-handler ordering) | implementation-dependent |
| MEV front-running on payments | public-mempool observer | §9.13 (private mempool option) | parameter-driven |
| Cross-chain atomicity failure | time / chain operator | §9.13 (HTLC timelocks) | mitigated for HTLC; SR-5 implementation-dependent for tanks |
| Liquidity-tank operator compromise | substrate operator | §9.13 (substrate consensus + 15-day recovery) | partial — substrate-trust-floor |
| AP2 mandate replay | network observer | §9.13 (AP2 nonce/expiry) | inherited from AP2 |
| x402 receipt forgery | malicious server | §9.13 (signature verification) | inherited from x402 |
| Refund laundering | malicious seller | §9.13 (anchored amendments) | mitigated |
| Decimal-overflow on cross-decimal pay | implementation bug | §9.13 (string-decimal arithmetic) | mitigated |
| Bundle forgery | malicious counterparty | §10.11 (co-signature requirement) | mitigated |
| Bundle suppression | malicious counterparty | §10.11 (one-sided bundle classification) | mitigated |
| Sybil reputation farming | sybil attacker | §10.11 (per-primary-claim keying) | mitigated for cross-tier; not for same-tier |
| Reputation collusion | two colluding counterparties | §10.11 (volume disclosure + external signals) | partial — protocol cannot prevent |
| Orchestrator error-class misclassification | malicious orchestrator | §10.11 (party-disagreement → aborted-by-other) | mitigated |
| Bundle anchor unavailability | storage operator | §10.11 (on-substrate anchoring) | mitigated (substrate-dependent) |
| Stale reputation windows | time | §10.11 (explicit window bounds) | consumer-driven |
| ERC-8004 write spamming | spam adversary | §10.11 (gas cost + per-session rate limit) | mitigated |

### 12.5 Composite trust property

A DACS-5 bundle that validates against all per-chapter conformance rules and whose contained references all dereference and validate provides the following composite trust property to a consumer: "Two or more parties identified by the named primary claims (with the trust profile each claim’s scheme implies) participated in a session against the named listing version, agreed to the named terms, exchanged the named settlements, and produced this audit record. The substrate operator did not collude with the parties to forge the record. The recipe registry was not compromised at the time of the verifications. The composed external standards (W3C VC, TLSNotary, ACME, etc.) behaved per their own security models." This is the composite security claim of DACS v1. Each clause has explicit mitigation in the per-chapter sections; each has explicit residual risk in this chapter’s adversary model.
## Chapter 13 — Glossary
A single alphabetical glossary across all five per-stage standards, the front matter, and the back matter. Terms defined in multiple chapters are cross-referenced. This glossary is informative; per-chapter definitions are normative.
## Chapter 14 — Conformance test plan
This chapter sketches the test categories an implementer should cover to claim conformance to each DACS standard. It is a **plan, not a test suite**; the test suite itself (test vectors, expected outputs, golden files) is produced separately and tracked alongside reference implementations. Where a chapter’s conformance summary enumerates labelled rules (e.g., BP-1, LR-2, CM-3), the test plan groups them into runnable categories.

### 14.1 DACS-1 — Identify

### 14.2 DACS-2 — Vet

### 14.3 DACS-3 — Negotiate

### 14.4 DACS-4 — Settle

### 14.5 DACS-5 — Verify

### 14.6 Universal signature scheme (SIG-1..SIG-4)

A cross-cutting test category that every conforming implementation runs once:

### 14.7 Substrate-capability tests

For substrates other than Demos that claim conformance, additional capability tests apply:

### 14.8 Out of scope for v1 conformance

The following are not part of v1 conformance and SHOULD NOT be tested as such:

## References

Cross-stage references for DACS-1 through DACS-5. Per-stage chapters may cite additional substrate-specific or standard-specific material inline.
**Normative — RFCs**
**Companion DACS specifications**
**Ethereum ecosystem**
**W3C and related**
**Payment standards**
**Agent communication**
**Verification and attestation**
**Demos / Kynesys**
**Identifiers and utility**
**Procurement frameworks**
