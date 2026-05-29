# Contributing to lnurl/luds

LUDs (LNURL Documents) are small, focused protocol extensions. This document
describes a minimal process to help contributors and reviewers work together
effectively. The goal is clarity, not bureaucracy.

---

## Two paths to a new LUD

**Path 1 — Organic:** Someone starts doing something, others copy it, and the
pattern gets codified into a LUD after the fact. This is how many LUDs came to
exist.

**Path 2 — Proposal:** Someone has an idea, writes it up, submits it here, and
the community gives feedback until there is reasonable agreement.

Both paths are valid. What matters is that a LUD reflects something real and
useful, not just a theoretical design.

---

## Acceptance criteria

A LUD should meet all of the following before being merged:

1. **Implemented** — at least one wallet and one service run it in production.
   The more independent implementations, the stronger the case.
2. **Optional and backward-compatible** — implementations that do not support
   the LUD must continue to work normally. The only exception is when the spec
   itself defines explicit rejection behavior (e.g. a service that sets a
   requirement must reject requests that do not meet it).
3. **One way to do the thing** — there should be no existing LUD that already
   solves the same problem.
4. **Makes sense** — the problem it solves is real, the solution is proportionate,
   and the design does not create unnecessary complexity.

---

## Submitting a PR

1. **Open an issue first** for new concepts. Describe the problem and your
   intended approach. This prevents wasted effort and surfaces early objections.
2. **Use `LUD-XX` as a placeholder** in your PR. Do not self-assign a number.
   A maintainer assigns the final number at merge time.
3. **Name your file `XX.md`** — it will be renamed when merged.
4. **Extend an existing LUD** where applicable, and reference it explicitly.
5. **Follow the document format** used in existing LUDs:
   - Title: `LUD-XX: short description`
   - Author tag
   - Motivation section
   - Mermaid sequence diagram for interaction flows
   - TypeScript-style JSON definitions
   - Numbered interaction steps
   - `MUST` / `SHOULD` / `MAY` language (RFC 2119)

---

## Review and merge

- PRs should receive at least **2 approvals** from community members before merge.
- Maintainers make the final merge decision and assign the LUD number.
- Technical objections block a merge; stylistic or philosophical disagreements
  that have been addressed in discussion do not.
- A PR that has been open a long time with approvals and production implementations
  but no decision should be re-raised directly with the maintainers.

---

## Small contributions

Typo fixes, clarifications to existing LUDs, and wallet/service additions to
the README do not need an issue first — open a PR directly.
