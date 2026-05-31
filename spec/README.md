# DACS Specification — v0.1

This directory holds the authoritative DACS standard.

| File | Description |
|------|-------------|
| [`SPECIFICATION.md`](./SPECIFICATION.md) | **Browsable edition** — the full standard rendered as Markdown so it reads, links, and diffs directly on GitHub. This is the recommended way to read the spec. |
| [`DACS-Specification-v0.1.docx`](./DACS-Specification-v0.1.docx) | **Formatting-faithful edition** — the same content as a Word document (comment-free). Use if you need the original layout. |

Both editions carry identical normative content: Introduction and DACS-1 through
DACS-5, with global terminology, substrate-capability definitions (SR-1…SR-5), the
Demos production mapping, conformance summaries, threat model, and conformance test
plan.

**Normative language.** The specification uses the RFC 2119 / RFC 8174 keywords
(MUST, SHOULD, MAY, …), normative only when uppercase.

**Version.** Published as DACS **v0.1** — the first publicly released version. All
five per-stage standards version together as a single document. See the
[CHANGELOG](../CHANGELOG.md) for normative change history and a migration list for
anyone who implemented against an earlier draft.

> The Markdown edition is auto-derived from the source document. If you spot a
> rendering artifact (a table that didn't convert cleanly, a mis-levelled heading),
> please [open an issue](https://github.com/DACS-Agent-commerce/DACS-Standard/issues)
> — the `.docx` is the tie-breaker for content.
