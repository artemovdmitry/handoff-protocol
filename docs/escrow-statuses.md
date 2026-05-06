# Escrow statuses

A Handoff Bundle describes assets, but doesn't move them. The companion concept is **escrow**: every transferable asset carries a status that says who currently controls it.

This is a UX and trust pattern, not part of the schema's logic — but the schema reserves a `status` field on every transferable object so any platform can render the same view.

## The three statuses

### `TRANSFERRED`

The asset is fully under the brand's control. Examples:

- Backlink whose target URL the brand owns and can monitor
- Disavow file uploaded under the brand's GSC account
- Vendor introduction where the brand has a direct contact
- Content brief stored in the brand's own DAM

`TRANSFERRED` is the terminal state. Once an asset is `TRANSFERRED`, the outgoing agency has no remaining control over it.

### `ESCROW`

The asset is held by a neutral third party until the rank-safety window closes. The "third party" can be:

- A platform like Handoff (the reference implementation)
- A managed service provider both parties trust
- A holding account explicitly created for the transition

`ESCROW` exists because some assets are dangerous to transfer atomically. Example: revoking the outgoing agency's GSC access on day 1 of a 30-day handover means losing the agency's ability to remediate an emergency. Holding access in escrow means: read-only for both parties, write disabled, until the rank-safety SLA closes.

### `PENDING`

The asset has been identified but not yet handed over. This is an action item for the outgoing agency. Examples:

- A backlink the agency built but hasn't documented in the bundle
- A vendor relationship that hasn't been introduced to the brand
- A content brief that exists in the agency's Notion but not in the bundle

`PENDING` is the only status that triggers a workflow on the outgoing-agency side. The bundle becomes "complete" when no asset is `PENDING`.

## State transitions

```
PENDING ──→ ESCROW ──→ TRANSFERRED
   │                       ▲
   └───────────────────────┘
```

The most common path is `PENDING → ESCROW → TRANSFERRED` (held neutrally during transition, then released). The direct `PENDING → TRANSFERRED` path is used when the rank-safety window for that asset is not relevant — e.g., a vendor introduction.

Backwards transitions (`TRANSFERRED → ESCROW`) are not allowed by design. Once an asset is fully transferred, it cannot be re-escrowed without the brand's explicit consent — which is logged as a separate decision in the bundle.

## Rank-safety window

The escrow concept is meaningful only because of a defined rank-safety window — typically 14 to 30 days — during which traffic must stay within an agreed band (e.g., "no more than a 10% week-over-week drop on tracked keywords").

The window is a contractual concept, not a schema field. The schema captures the start and end of the engagement (`outgoing_party.engagement_ended_at`, `incoming_party.engagement_started_at`); platforms like Handoff layer the rank-safety SLA on top.
