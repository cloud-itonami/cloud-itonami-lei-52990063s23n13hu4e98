# ADR-0001: ToSMonitor-LLM ⊣ ToSArchiveGovernor -- a governed actor layered on this archive

- Status: Accepted (2026-07-24)
- Related: [`com-junkawasaki/root` ADR-2607110300](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607110300-cloud-itonami-lei-corporate-tos-catalog.edn)
  (the archive-only design this repo was created under -- unchanged by this
  ADR); [`com-junkawasaki/root` ADR-2607241900](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607241900-cloud-itonami-lei-tos-monitor-actor-pilot.edn)
  (the original 1-repo pilot); [`com-junkawasaki/root` ADR-2607242300](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607242300-cloud-itonami-lei-tos-monitor-actor-batch10-round2.edn)
  (the round-2 10-repo validation batch this repo is part of -- this repo
  is the first `:terms-of-carriage` doc-type company in this actor family).

## Context

This repository archives the publicly published Beforderungsbedingungen
(Conditions of Carriage) of Deutsche Bahn AG, per ADR-2607110300 -- a
read-only reference archive. As part of a second 10-repo validation batch
extending the `cloud-itonami-lei-2572ibtt8cczw6au4141` pilot (see
ADR-2607241900/ADR-2607242300 for the full fleet-level design rationale),
this repo gains a governed actor layer on top of the unchanged archive.

## Decision

Identical design and code to the pilot and every other repo in this batch
(`src/tosmonitor/{governor,phase,operation,registry,advisor}.cljc` are
byte-for-byte identical across all of them) -- see ADR-2607241900 for the
full rationale of each of the six HARD governor checks, the single
always-escalate `:tos/change-proposal` actuation, and the mock-advisor-only
scope. Only `tosmonitor.store`'s company/baseline demo data is specific to
this repo:

- **Company**: Deutsche Bahn AG, LEI 52990063S23N13HU4E98, website
  `https://www.bahn.de`.
- **Baseline provenance** (real, from this repo's own `80-data/public/
  tos.journal.edn`): source-url `https://www.bahn.de/agb`, retrieved-at
  `2026-07-10T06:26:33Z`, doc-type `:terms-of-carriage` -- the first
  company in this actor family to use this doc-type value (already
  present in `tosmonitor.registry/known-doc-types` after that vocabulary's
  round-1 correction, ADR-2607242000, so it correctly passes rather than
  false-positive holding).
- **Baseline full text**: a short, hand-written representative German
  excerpt (not the real archived page), with a self-consistent SHA-256
  computed from that excerpt itself -- matching the pilot's own convention
  (ADR-2607241900).

The archive-of-record (`80-data/public/tos.journal.edn`) is never touched;
`commit-record!` only writes to this actor's own Store.

## Consequences

Same as the pilot (ADR-2607241900) and the batch (ADR-2607242300), plus:
first real-world exercise of the `:terms-of-carriage` doc-type and of a
non-English baseline excerpt within this batch.

## Run

```bash
clojure -M:dev:run     # walk a clean lifecycle + all six HARD-hold checks + a phase-0 hold + a backend swap
clojure -M:dev:test    # governor contract · phase invariants · store parity · advisor smoke
clojure -M:lint        # clj-kondo (errors fail; CI mirrors this)
```
