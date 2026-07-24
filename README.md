# cloud-itonami-lei-52990063s23n13hu4e98

> **Independent third-party archive/analysis. Not affiliated with, endorsed by, or sponsored by Deutsche Bahn AG.**

This repository archives the publicly published Beförderungsbedingungen (Conditions of
Carriage) of **Deutsche Bahn AG**, with source-url and retrieval-date provenance, per
[ADR-2607110300](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607110300-cloud-itonami-lei-corporate-tos-catalog.edn)
(`cloud-itonami-lei-corporate-tos-catalog`, `com-junkawasaki/root`). The archive itself
(`blueprint.edn`, `NOTICE`, `80-data/public/tos.journal.edn`) is unchanged by anything
below — it stays a read-only reference archive; this repo does not act, propose, or
execute anything on the company's behalf using the archive data itself.

**As of 2026-07-24 this repo ALSO carries a governed actor layered on top of that
archive** (`src/tosmonitor/*`, ToSMonitor-LLM ⊣ ToSArchiveGovernor), added as part of a
second 10-repo validation batch extending the pilot on
`cloud-itonami-lei-2572ibtt8cczw6au4141` (P&G)
([ADR-2607241900](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607241900-cloud-itonami-lei-tos-monitor-actor-pilot.edn),
[ADR-2607242300](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607242300-cloud-itonami-lei-tos-monitor-actor-batch10-round2.edn),
repo-local record: [`docs/adr/0001-tos-monitor-actor.md`](docs/adr/0001-tos-monitor-actor.md)).
This does NOT change the archive's own read-only nature, and it does NOT extend to
every other `cloud-itonami-lei-*` repository — most repos in this family remain plain
archives exactly as ADR-2607110300 describes. See `## Governed actor` below.

## Company identity

- **Legal name**: Deutsche Bahn AG
- **LEI (ISO 17442)**: [52990063S23N13HU4E98](https://search.gleif.org/#/record/52990063S23N13HU4E98) (GLEIF-verified)
- **Jurisdiction**: DE
- **Website**: https://www.bahn.de
- **Ticker**: none (wholly state-owned, not separately listed)

## Contents

- `80-data/public/tos.journal.edn` — EDN quad-log of archived Conditions-of-Carriage
  documents, each entry carrying `:tos/full-text`, `:tos/source-url`, `:tos/retrieved-at`,
  `:tos/sha256`, `:tos/doc-type`, and a `:tos/supersedes` chain for future revisions.
- `NOTICE` — copyright/attribution statement for the archived third-party text.
- `blueprint.edn` — machine-readable company identity record.

## Governed actor

**ToSMonitor-LLM ⊣ ToSArchiveGovernor**, built on this workspace's
[`langgraph-clj`](https://github.com/com-junkawasaki/langgraph-clj) StateGraph runtime,
modeled directly on
[`cloud-itonami-commitment-ledger`](https://github.com/cloud-itonami/cloud-itonami-commitment-ledger)'s
Store/Registry/Advisor/Governor/Phase/Operation/Sim shape (byte-identical code to the
pilot and every other repo in this batch except `tosmonitor.store`'s company/baseline
data).

Given a candidate ToS/legal-document snapshot, the actor proposes whether it materially
diverges from the archived baseline above, and drafts a change summary — but **it never
writes to the archive itself.** `tosmonitor.store`'s `commit-record!` only writes to
this actor's own Store (an in-process ledger), never to `80-data/public/
tos.journal.edn`. A human archivist decides whether to fold a proposal into the real
archive.

**Single actuation, `:tos/change-proposal`, never autonomous.** Every run — whether the
candidate matches the baseline or diverges from it — carries `:stake :actuation/
archive-update` and is permanently excluded from every rollout phase's auto-commit set.
Six HARD governor checks, each independently re-verified rather than taken on the
advisor's self-report:

| Check | Guards against |
|---|---|
| `grounding-violations` | a cited excerpt not actually present in the candidate's own text (hallucinated quotes) |
| `provenance-incomplete-violations` | a candidate missing full-text/source-url/retrieved-at/sha256 |
| `sha256-mismatch-violations` | a candidate's self-reported sha256 not matching its own content (ground-truth recompute, `:clj`-only in V1) |
| `retrieved-at-not-advancing-violations` | proposing a change from input staler than what is already archived |
| `doc-type-unknown-violations` | a doc-type outside this archive family's own observed vocabulary (this repo's own baseline doc-type is `:terms-of-carriage`, the first use of this vocabulary value in this actor family) |
| `source-domain-mismatch-violations` | this actor's own distinctive check — a source-url that doesn't belong to the archived company (misattribution risk specific to an LEI-keyed independent archive) |

```bash
clojure -M:dev:run     # clean lifecycle + all six HARD-hold checks + a phase-0 hold + a MemStore->DatomicStore swap
clojure -M:dev:test    # governor contract · phase invariants · store parity · advisor smoke
clojure -M:lint        # clj-kondo (errors fail; CI mirrors this)
```

`clojure -M:dev:run` and the test suite always use the deterministic mock-advisor — no
live fetch of the company's current ToS page and no live `kotoba-server`/CACAO publish
happen anywhere in this actor. `tosmonitor.advisor/llm-advisor` exists as a written,
swappable seam but is not invoked. See
[`docs/adr/0001-tos-monitor-actor.md`](docs/adr/0001-tos-monitor-actor.md) for the full
design record.

## Design rationale

See ADR-2607110300 in `com-junkawasaki/root` (`90-docs/adr/`) for why this repo exists,
why it is keyed by LEI rather than GTIN or ticker, and why full-text archival (with
provenance) was chosen over excerpt-only storage.
