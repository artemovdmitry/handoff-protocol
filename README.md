# Handoff Protocol

**A portable, vendor-neutral handover format for SEO engagements.**

Brands lose 20–40% of organic traffic every time they switch SEO agencies. There is no industry standard for transferring assets, decisions, and historical context between vendors — so every transition is a one-off, manual, lossy process.

This repo contains a working draft of the **Handoff Bundle** — a JSON-Schema'd format that describes everything an SEO project produces and consumes, in a way that any agency, in-house team, or platform can read and write.

> **TL;DR.** Think of it as the `package.json` of an SEO engagement: a single, machine-readable file that captures audit history, link inventory, decisions, redirect history, anchor strategy, vendor relationships, and 24-month baseline metrics — so a brand can fire its agency on Monday and resume work with a new one on Tuesday without losing rankings.

---

## Why a protocol, not a tool

Every all-in-one SEO platform (Semrush, Ahrefs, AgencyAnalytics, SE Ranking, SEOWORK) is structured around the assumption that **the agency, not the brand, owns the workspace**. When the agency is fired, the brand loses:

1. Every backlink the agency built (no inventory file, no anchor map, no DR tracking)
2. Every audit run and its remediation history
3. The disavow file (and the rationale for each entry)
4. Content briefs, content drafts, and the editorial calendar
5. Decisions log: *why* a given page was de-indexed, *why* a redirect was chosen, *why* a competitor was prioritised
6. Vendor relationships: link-builders, copywriters, outreach contacts
7. 24+ months of baseline metrics that any new agency needs to detect anomalies

The Handoff Protocol is the open, audited container for that data. It is deliberately platform-agnostic: a `.handoff` file is a versioned JSON archive that any vendor can ingest.

## What's in this repo

```
.
├── README.md                              ← you are here
├── spec/
│   └── handover-bundle.schema.json        ← the JSON Schema (draft v0.1)
├── examples/
│   └── north-supply-co.bundle.json        ← realistic synthetic bundle
└── docs/
    ├── 12-categories.md                   ← the 12 top-level objects, explained
    ├── design-decisions.md                ← rationale for the schema choices
    └── escrow-statuses.md                 ← TRANSFERRED / ESCROW / PENDING semantics
```

## The 12 categories

A Handoff Bundle is a single JSON document with the following top-level keys. Each is independently versioned so partial bundles are valid.

| # | Key | What it carries |
|---|---|---|
| 1 | `meta` | Brand, outgoing agency, incoming agency, bundle version, signing hashes |
| 2 | `audit_history` | Every site-audit run (timestamp, tool, raw findings, remediation status) |
| 3 | `link_inventory` | All known backlinks with DR, anchor, status, build date, builder |
| 4 | `disavow` | Active disavow list + rationale per entry |
| 5 | `content_briefs` | Briefs delivered to writers, with target keywords and SERP context |
| 6 | `decisions_log` | Append-only journal of strategic decisions (timestamp, author, rationale) |
| 7 | `redirect_history` | Every 301/302 ever shipped, source → target, ship date, reason |
| 8 | `anchor_strategy` | Anchor-text targets per cluster + observed distribution |
| 9 | `vendor_relationships` | Subcontractors (link-builders, writers, designers) with contacts and rates |
| 10 | `baseline_metrics` | 24-month timeseries: GSC clicks/impressions, GA sessions, rankings |
| 11 | `roadmap` | Active and completed task-blocks (modular roadmap protocol, separate spec) |
| 12 | `comments` | Threaded discussion attached to any object above |

See [`docs/12-categories.md`](./docs/12-categories.md) for field-level detail.

## Asset escrow

A Handoff Bundle alone does not move assets — it describes them. The companion concept is **escrow**: every asset in the bundle has a status indicating who currently controls it.

| Status | Meaning |
|---|---|
| `TRANSFERRED` | Asset is fully under the brand's control (admin login, ownership, etc.) |
| `ESCROW` | Held by a neutral third party until the rank-safety window closes |
| `PENDING` | Identified but not yet handed over (action required) |

Escrow is a UX pattern, not a part of the schema — but the schema reserves a `status` field on every transferable asset so any platform can render the same view.

## Status

This is a working draft. The schema is **not yet stable**; field names and shapes will change as we run it against real handovers.

- **v0.1** (current) — 12 top-level keys, lossy round-trip
- **v0.2** (planned) — lossless round-trip on synthetic bundles
- **v1.0** — first migration in production with a paying brand

## Who is this for

- **Brands** who want a clean exit clause in their agency contracts ("on termination, agency provides Handoff Bundle v1.x within 14 days")
- **Agencies** who want to win business by signalling that they don't lock clients in
- **Platforms** that want to support import/export and become the lingua franca

## Reference implementation

A reference implementation is being built at **[handoff-smooth-transitions.lovable.app](https://handoff-smooth-transitions.lovable.app/)** as the Handoff product. The protocol itself is intended to outlive any single tool — that's the point of putting it on GitHub.

## License

The schema and documentation are released under the MIT License (see `LICENSE`). Use it, fork it, build on it.

## Contact

Built by [Dmitry Artemov](https://www.linkedin.com/in/dmitry-artemov-62384163/) — 16 years in SEO across LG Russia, Zvuk/Sber, and a small agency I closed to build this.
