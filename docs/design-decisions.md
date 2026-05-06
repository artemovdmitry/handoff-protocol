# Design decisions

Notes on the trade-offs behind the schema. Useful if you're forking the spec or proposing changes.

## Why JSON Schema over OpenAPI / Protobuf / Avro

JSON Schema is the lowest-friction option for the audience. Brand-side ops people, agency PMs, and content folks need to be able to read a bundle and reason about it without tooling. JSON-with-comments is the lingua franca; Protobuf is a non-starter outside engineering teams.

We're paying the cost on the strictness side: JSON Schema's validation story is weaker than Protobuf's, and there's no built-in code generation. Trade accepted.

## Why a single document, not a multi-file archive

Early drafts had `audit-history.json`, `link-inventory.json`, etc. as separate files inside a `.handoff` zip. We collapsed this for two reasons:

1. **Tooling.** A single JSON document round-trips through `jq`, GitHub Gists, and copy/paste into a chat. A zip archive doesn't.
2. **Cross-references.** `decisions_log` IDs are referenced from `redirect_history`, `comments` are pinned to objects across the bundle. Separate files would have made cross-document referential integrity painful.

The trade-off: large bundles can get heavy. We're handling this by accepting `null`/`omitted` top-level keys, so a bundle that doesn't carry, say, a 24-month GSC timeseries can simply omit `baseline_metrics.gsc`.

## Why append-only `decisions_log`

Decisions get revisited, but they don't get *unmade*. If a brand changes its mind about a deindex strategy, the right shape is a *new* decision that supersedes the old one — with a `linked_objects` reference back. This preserves the historical record so the new agency can understand *why the old approach was tried* before the reversal.

Append-only also matters for legal-grade audit trails. A signed bundle whose `decisions_log` was edited after the fact is a tampered bundle.

## Why escrow status is on the asset, not in a separate file

Backlinks, vendor relationships, and disavow entries each have their own escrow lifecycle. Pulling status out into a separate "escrow ledger" file would have meant joining everywhere. Keeping the status on the asset means a render of the bundle is a render of the escrow state. The trade-off: bundles can't easily express "this asset transitioned from PENDING to ESCROW at time X" without a separate audit log. Future versions may add per-asset transition history if real handovers demand it.

## Why versioning is per-key, not whole-bundle

Schema evolution will happen at different rates for different objects. `audit_history` is likely stable; `roadmap` will move with the modular roadmap protocol; `baseline_metrics` may add new metric types as AI-search tracking matures.

A whole-bundle version would force everyone to upgrade in lockstep. A per-key version (planned for v0.2; not yet implemented) lets each object evolve independently.

## Why `comments` exists at all

The first draft didn't have it. We added it after realising that ~30% of the context an incoming agency actually needs lives in the *side-channel* — the Slack thread on a brief, the inline note on a redirect, the dispute over a vendor. Without `comments`, the bundle would carry the artifacts but lose the conversation.

The trade-off is privacy. Some chat content shouldn't be in a bundle. We're addressing this with `posted_at` filters and an `excluded_from_bundle` flag (planned for v0.2).

## Open questions for v0.2

1. **Lossless round-trip.** The current draft loses information on round-trip — fields that aren't in the schema get dropped. v0.2 will add a top-level `extensions` object that platforms can use to carry vendor-specific data.
2. **Per-key versions.** As above.
3. **Modular roadmap reference.** `roadmap` currently inlines tasks. If the modular roadmap protocol stabilises, we'll move to a `$ref` model.
4. **Anchor strategy expressiveness.** The current `anchor_strategy` only handles distribution percentages. Real strategies have rules ("no exact-match for branded clusters in Q4 2026") that the schema doesn't yet express.
5. **Disavow-rationale required vs. optional.** Currently required; some bundles will arrive with imported disavow files that pre-date the rationale practice. v0.2 may demote it to recommended.

PRs welcome.
