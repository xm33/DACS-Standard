# SR-4 / L2PS — Implementation Brief

**Audience:** SDK/substrate dev implementing the DACS negotiation channel on Demos L2PS.
**Goal:** close the three 🟡 gaps in CORE §A.4 (SR-4) so DACS-3 `negotiate-rfq` and `negotiate-sealed-envelope` run end-to-end on a real private channel.
**Stack:** `@kynesyslabs/demosdk` (TypeScript) — `l2ps`, Identities/CCI, Storage Programs, `demoswork`.

---

## 0. Background (read once)

SR-4 is the **private negotiation channel**. In DACS, two negotiation patterns require it:

| Pattern | Needs SR-4? |
|---|---|
| `negotiate-fixed-price` | No (SR-2 only — just accept the listed terms) |
| `negotiate-rfq` | **Yes** (private offer/counter) |
| `negotiate-sealed-envelope` | **Yes** (commit-then-reveal sealed bids) |

The channel must satisfy six properties — **CH-1 … CH-6** (DACS-3 §8.3.1). These are the conformance bar; everything below exists to meet them:

- **CH-1 Identity-keyed membership** — members are a fixed list of ClaimReferences, each in a verified DACS-1 bundle.
- **CH-2 Confidentiality** — non-members can't read; the public chain sees only commitments.
- **CH-3 Authenticity** — every message signed by the author's **primary-claim key** (the same key used in DACS-2 verification).
- **CH-4 Liveness** — bounded delivery; members can detect failure and abort.
- **CH-5 Termination** — channel reaches a terminal state (agreement / abort / timeout).
- **CH-6 Per-session channelId uniqueness** — a `channelId` is never reused across sessions.

**Claim vs key (important — don't conflate these):**
- **Primary claim** = a `ClaimReference` (`<scheme>:<identifier>`) — the *name* of the identity. This is what goes in `sender`, `members[]`, and `AgreementParty.primaryClaim`. It is a label, not a key.
- **Primary key** = the key that *controls* that claim — what you actually **sign with**. The spec (§8.3.2) requires it to be "the same key that holds value on-chain." **On Demos that is the agent's Demos Ed25519 keypair** — so in this implementation, *sign with the Demos public/secret key*. (The spec says "primary claim/key" rather than "Demos key" only to stay substrate-agnostic; if a party's primary claim were instead an EVM-context CCI, the controlling key would be that EVM key. For a Demos-resident agent: primary key = Demos key.)

**The one invariant that ties it together:** the key that signs in-channel messages **must be the same key that controls the sender's primary claim** — i.e. the Demos keypair that signs the final AgreementDocument (DACS-3 §8.5.1) and holds value on-chain. That chain of custody — *in-channel signer == on-chain party* — is the whole reason DACS-3 uses a CCI-bound channel instead of a generic chat. **Do not introduce any signing path that uses a key other than the one controlling the primary claim (on Demos: the Demos key).**

---

## 1. What already works (🟢 — build on this, don't rebuild)

- `new l2ps.L2PS()` / `new l2ps.L2PS(rsaPrivateKey)` — subnet creation + membership (currently **RSA-key-keyed**).
- DemosWork orchestration — `WorkStep`, `BaseOperation`, `ConditionalOperation` (`@kynesyslabs/demosdk/demoswork`).
- Storage Programs — anchoring the agreement hash and sealed-envelope commitments (this is SR-2).

The substrate transports messages and anchors data. The gaps are **DACS identity binding, DACS message-envelope semantics, and the disclosure helper** on top.

---

## 2. Work items

Three work items, build in order (each depends on the previous).

### WI-1 — CCI-keyed membership (binding proof)

**Goal:** make channel membership map to **CCI primary claims**, so channel signatures verify against the same identity used in DACS-2 (CH-1, CH-3).

**Why:** today L2PS identifies members by RSA keys. DACS needs members identified by their CCI primary claim (a `ClaimReference`) present in their DACS-1 bundle. Until L2PS is natively CCI-keyed, DACS-3 §8.3.2 specifies an **interim binding-proof**.

**What to build:** before negotiation begins, each participant produces and anchors a signed binding that links their subnet identity to their CCI primary claim.

```ts
// Each member signs this with the key controlling their primary claim —
// on Demos, the DEMOS Ed25519 keypair (NOT the RSA subnet key):
type L2PSMembershipBinding = {
  bindingVersion: "1"
  channelId: string            // the L2PS subnet / session channel id
  subnetMemberId: string       // the member's L2PS/RSA identity within the subnet
  cciPrimaryClaim: ClaimReference  // "<scheme>:<identifier>" — must be in the member's DACS-1 bundle
  boundAt: number              // unix ms
  signature: string            // CCI-primary-key signature over canonical(binding minus signature)
}
```

- Anchor each binding via a **Storage Program (SR-2)** *before* negotiation starts.
- Provide a verifier: `resolveMember(channelId, subnetMemberId) -> ClaimReference` that reads the anchored binding and returns the bound CCI claim, or rejects if no valid binding exists.
- Membership is **fixed for the channel lifetime** (CH-1) — no add/remove mid-channel. Reject any message from a `subnetMemberId` without a valid binding.

**Acceptance:**
- [ ] For an N-member subnet, each member has an anchored, CCI-primary-key-signed binding.
- [ ] A third party can verify `subnet member ↔ CCI primary claim ↔ DACS-1 bundle` for every member.
- [ ] A message from an unbound subnet identity is rejected.

**Gotcha:** the binding MUST be signed by the key controlling the primary claim (**on Demos, the Demos Ed25519 key** — the one that holds value on-chain), **not** the RSA subnet key — otherwise it proves nothing about on-chain identity.

---

### WI-2 — Channel message envelope API

**Goal:** wrap L2PS messages in the DACS `ChannelMessage` envelope with monotonic sequencing, CCI-key signing, and transcript export (CH-3, CH-6; §8.3.3).

**The envelope (verbatim from §8.3.3):**

```ts
type ChannelMessage = {
  channelId: string            // substrate-derived; MUST be unique per session (CH-6)
  sequence: number             // monotonic per channel, starts at 1
  sender: ClaimReference       // author's CCI primary claim (from WI-1)
  sentAt: number               // unix ms
  type: "offer" | "counter" | "accept" | "reject"
      | "sealed-envelope-commit" | "sealed-envelope-reveal" | "abort"
  body: unknown                // type-specific (sealed-envelope bodies per §8.4.3; rfq bodies impl-defined)
  refs?: { repliesTo?: number }
  signature: ChannelMessageSignature
}
```

**Signing (exact):** canonical form = RFC 8785 JCS of the envelope **with `signature` omitted**; envelope hash = `sha256(canonical_form)` (hex). The signature is over the **domain-separated payload**:

```
signed_bytes := "dacs-channelmsg:v1:" || envelope_hash
```

…produced with the key controlling the sender's primary claim — **on Demos, the sender's Demos Ed25519 key**. Transport-level fields (routing/framing) may wrap the envelope but **must not change the signed bytes**.

**Three things to implement:**

1. **Sequence numbering** — assign/enforce `sequence` monotonic per channel, starting at 1. Reject out-of-order or duplicate sequences. (This + CH-6 is the anti-replay defence, §8.12: a captured message replayed into a *different* channel fails because `channelId` differs; replayed into the *same* channel duplicates a `sequence` and is rejected.)
2. **channelId uniqueness (CH-6)** — the substrate derives a per-session-unique `channelId`; reject a session that reuses a prior `channelId`. Without this, sequence-based replay defence is vacuous.
3. **Signature export** — `sign(envelope) -> ChannelMessage` (CCI key, `dacs-channelmsg:v1:` payload) and `verify(msg) -> bool` (checks signature against the sender's bound CCI key from WI-1, checks `sender ∈ members`, checks sequence). Export the raw signed envelope so any auditor can re-verify independently.
4. **Transcript export** — assemble the ordered signed messages into a `ChannelTranscript` (consumed by WI-3):

```ts
type ChannelTranscript = {
  transcriptVersion: "1"
  channelId: string
  members: ClaimReference[]
  messages: ChannelMessage[]   // in sequence order
  generatedAt: number
  signatures: TranscriptSignature[]
}
```

**Acceptance:**
- [ ] Round-trip: construct → sign (primary-claim key, i.e. the Demos key) → send → receiver verifies signature + `sender ∈ members` + monotonic sequence.
- [ ] A reused `channelId` is rejected (CH-6).
- [ ] An out-of-order or duplicate `sequence` is rejected.
- [ ] Full ordered transcript exports and can be re-verified message-by-message by a non-member given the public keys.

**Gotcha:** `sender` is the **ClaimReference** (the identity's *name*, resolved via WI-1), not the RSA subnet id. Verification = the signature is valid under the key controlling that member's primary claim — i.e. the **Demos key** bound to that member via WI-1.

---

### WI-3 — Encrypted transcript anchoring helper

**Goal:** support `terms.transcriptDisclosurePolicy` (§8.7), specifically `encrypted-anchored-required` and `encrypted-anchored-recommended`.

**Policy values (§8.7):**
- `none` (default) — transcript stays in the channel; nothing anchored.
- `encrypted-anchored-recommended` — SHOULD anchor; consent must be explicit.
- `encrypted-anchored-required` — **MUST** anchor an encrypted transcript; **absence of the anchor fails the phase.**

**What to build:** `anchorEncryptedTranscript(transcript, members) -> AttestationRef` that:
1. **Encrypts** the `ChannelTranscript` to the **member set** (only members can decrypt) — CH-2.
2. **Anchors the ciphertext via SR-2** (Storage Program), keeping the **content hash public** for tamper-evidence.
3. **Signs** over the domain-separated payload:
   ```
   signed_bytes := "dacs-transcript:v1:" || sha256(canonical_JCS(transcript_without_signatures))
   ```
4. Returns a **`channelTranscriptRef: AttestationRef`** (`{ anchor, contentHash }`) — the `negotiate-rfq` output carries this as an optional field.

**Acceptance:**
- [ ] `required` policy: produces a member-decryptable, publicly-hash-verifiable anchor; if absent, the phase fails.
- [ ] `recommended` policy: anchors only on explicit consent; default is no anchor.
- [ ] A non-member cannot decrypt; anyone can verify the content hash against the anchor (tamper-evidence).

---

## 3. Build order & dependencies

```
WI-1 (membership binding)  →  WI-2 (envelope API)  →  WI-3 (transcript helper)
   defines "who is a member,    needs WI-1 for sender      consumes WI-2's
   keyed to which CCI claim"     + CCI-key signing          exported transcript
```

## 4. Definition of done (overall)

A `negotiate-rfq` or `negotiate-sealed-envelope` session can:
1. Establish an L2PS subnet whose members are bound to CCI primary claims (WI-1).
2. Exchange CCI-signed `ChannelMessage`s with enforced monotonic sequence + unique `channelId` (WI-2).
3. On `encrypted-anchored-required`, produce a member-decryptable, hash-verifiable anchored transcript (WI-3).
4. The party who signed in-channel is the **same CCI identity** that co-signs the committed AgreementDocument (the invariant in §0).

This moves CH-1..CH-6 enforcement **out of the application layer and into the SR-4 substrate** (the roadmap item: *"CCI-keyed L2PS membership + channel-envelope API + transcript export — Substrate-live; SDK backlog"*).

## 5. References

**Spec (DACS-3 / `spec/DACS-3-NEGOTIATE.md`):**
- §8.3.1 — CH-1..CH-6 (the conformance bar)
- §8.3.2 — SR-4 realisation + the binding-proof recipe (WI-1)
- §8.3.3 — `ChannelMessage` envelope + `dacs-channelmsg:v1:` (WI-2)
- §8.7 — `ChannelTranscript`, disclosure policy + `dacs-transcript:v1:` (WI-3)
- §8.12 — replay / forking / censorship threats these defend against
- CORE §A.4 — SR-4 status (the 🟢/🟡 items)

**SDK (`@kynesyslabs/demosdk`):** `l2ps`, Identities/CCI (primary-claim keys + binding), Storage Programs (SR-2 anchoring), `demoswork`. See the `demos-sdk:demosskill` reference for L2PS/CCI/anchoring patterns.

**Canonical form (applies to every signature above):** RFC 8785 JCS with the `signature` field omitted; hash = `sha256(canonical_form)` hex; all signatures are domain-separated (the `dacs-*:v1:` prefixes), per CORE §B.2 / §B.7.
