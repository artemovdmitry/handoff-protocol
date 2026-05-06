# The 12 categories of a Handoff Bundle

A Handoff Bundle is a single JSON document. The top-level keys below are independently versioned, so a partial bundle (e.g., one missing `content_briefs`) is still valid — the missing object is treated as "we don't have this yet" rather than "we have nothing".

This document is human-readable rationale for the schema; the canonical definition lives in [`spec/handover-bundle.schema.json`](../spec/handover-bundle.schema.json).

---

## 1. `meta`

Bundle-level metadata: brand, outgoing party, incoming party, generation timestamp, optional cryptographic signature. Without `meta` you don't know whose bundle this is or which agency it came from.

The `signature` block is optional but encouraged. A bundle that ships with an `ed25519` signature from the outgoing agency's known key gives the brand a legal-grade audit trail: "the agency attested to the contents of this file at this moment in time".

## 2. `audit_history`

Every site-audit run delivered to the brand. Each finding carries severity, type, URL, and a remediation status with a timeline. The point is not "what's broken now" — that's a snapshot the new agency will recreate. The point is **"what was identified, what was decided, what was fixed, and what was deferred"**, so the new agency doesn't redo two years of work or re-trip on a known wontfix.

## 3. `link_inventory`

All known backlinks with the metadata required to reproduce, defend, or replace them: source URL, target URL, anchor text, anchor type, rel attributes, DR/UR ratings, first seen, last checked, and an `acquisition` block that records *how* the link was obtained (outreach, guest post, digital PR) and at what cost.

Each link carries an [escrow status](./escrow-statuses.md). This is the single most important object in the bundle for E-commerce DTC handovers — it's the difference between "the new agency has a list to defend" and "the new agency is starting from zero".

## 4. `disavow`

The active disavow list with rationale per entry. Disavow is one of the few SEO assets that is *literally invisible* on the brand side without explicit transfer — it lives only in the search engine's tooling, scoped to the account that uploaded it. A bundle that doesn't carry the disavow file effectively *re-exposes* the brand to whatever penalty risk caused the entries in the first place.

## 5. `content_briefs`

Briefs delivered to writers, with target keywords and SERP context. Captures the *why* behind every commissioned piece: target intent, target URL, the SERP at the time of brief, and the writer who delivered it.

This is what makes the difference between "the new agency rewrites everything from scratch" and "the new agency knows the priors and can iterate".

## 6. `decisions_log`

The append-only journal of strategic decisions. **The single most important object in the bundle.**

Every consequential SEO decision — "we deindexed /tag/* because…", "we 301'd /shop/* into /collections/* because…", "we declined to disavow domain X because…" — is recorded with the context, the options considered, the chosen option, and the rationale. Linked objects (audits, briefs, redirects) cross-reference back into `decisions_log`.

Without this object, the new agency reverse-engineers two years of intent from artifacts. With it, they read the journal in an afternoon.

## 7. `redirect_history`

Every 301/302 ever shipped: source, target, status code, ship date, reason, and a foreign key into `decisions_log`. Without this, the new agency can't detect broken redirect chains, can't tell which redirects were intentional vs. accidental, and can't safely retire any of them.

## 8. `anchor_strategy`

Anchor-text targets per content cluster + observed distribution. Lets the incoming agency see *target* vs. *actual* — whether the link profile is over-optimised on exact-match (manual-action risk), under-optimised on branded (visibility risk), or actually balanced.

## 9. `vendor_relationships`

The subcontractors used during the engagement: link-builders, copywriters, outreach specialists, designers, developers. Each entry has role, contact, rate, and performance notes.

This is the soft network capital that almost always gets lost on handover. The new agency may have its own network, but the brand benefits from continuity in the high-trust relationships (e.g., a journalist who already covers the brand).

## 10. `baseline_metrics`

A 24-month timeseries baseline:

- `gsc` — Google Search Console clicks/impressions/CTR/position by day
- `ga4` — sessions, engaged sessions, conversions, revenue by day
- `rank_tracking` — keyword positions across search engines, countries, devices

Required so the incoming agency can detect anomalies introduced during the migration. A 30% click drop in week 2 of a transition is meaningful only against a baseline.

## 11. `roadmap`

Active and completed task-blocks. Each task has a block type, status, owner vendor, expected outcome, actual outcome, and timestamps.

This refers out to the modular roadmap protocol (a separate spec) which defines the standard library of SEO task blocks. The point of including it in the bundle is to let the new agency see what's in flight, what was promised, and what was delivered.

## 12. `comments`

Threaded discussion attached to any object above. Preserves the soft context that lives in chat — the side comment on a brief, the dispute on a redirect decision, the reasoning behind a vendor choice.

Without `comments`, the new agency inherits the artifacts but loses the conversation. With it, they inherit the engagement.

---

## What is intentionally NOT in the bundle

- **CMS content** itself (HTML, images, drafts) — out of scope; covered by CMS export tooling.
- **Live access tokens** (GA, GSC, Ads) — handled by the Asset Escrow flow, not the bundle.
- **Source code or theme files** — a developer concern, not an SEO one.
- **Email correspondence** — privacy-sensitive; stays with the brand.

The bundle is deliberately scoped to *the SEO engagement*, not *the entire marketing operation*.
