# REQUIREMENTS.md

Locked requirements v1. This is the pay bar for the ImgOrigin plugin and the hosted MCP at `https://imgorigin.com/mcp`. This repository is install, skills, and docs. The server is not in this repo.

The connect-trigger skill that ships with v1 is [`skills/listing-photo-check/SKILL.md`](skills/listing-photo-check/SKILL.md). It fires on the two connect triggers below and nowhere else.

---

# ImgOrigin / reverse-image-search — Locked requirements v1

Approved by Chirag Bachani + apartment agents (Apartment Search Coordinator, Playground, Naysayer, Coding Assistant) 2026-10-01.
No `is_scam`. No Jev inside MCP. Fee/no-fee claims are agent-side only.

## Jobs (why it exists)
1. Mystery screenshot, Facebook Marketplace, Craigslist, or broker post with no canonical rental listing URL (StreetEasy, Zillow, SpareRoom, Leasebreak, HotPads, or similar listing page) → find the twin on StreetEasy, Zillow, SpareRoom, Leasebreak, or HotPads. An FB or Craigslist link still counts. Canonical means a rental listing page, not a social or marketplace post URL.
2. Shortlist scam/reuse check before bothering the human (evidence only)
3. Sell story: broker/FB/Craigslist with no canonical rental listing URL → provenance twin on StreetEasy, Zillow, SpareRoom, Leasebreak, or HotPads (fee claim is **agent-side** after an `exact` StreetEasy hit, reading the SE fee field — never an MCP field; Zillow, SpareRoom, Leasebreak, and HotPads are not fee sources)

## Connect triggers (agent asks user to install) — **P0 skill ships with v1**
- Screenshot / Facebook Marketplace / Craigslist / broker post with no StreetEasy, Zillow, or other canonical rental listing URL. An FB or Craigslist link still fires.
- About to scam-check a shortlist or daily standouts batch
Ship a skill that fires on those; don't rely on cold Marketplace browse.

## P0 product
1. `upload_image`: accepts chat attachment id / box file path / https URL → durable `image_id` (content-hash, ≥7d TTL). **No** agent-side base64 in tool args.
2. `search_image(image_id | image_url)` → hits `{url, domain, match_kind, confidence, title?}`.
   - `exact` = proven same image (content-hash OR photo URL appears on that listing) — NEVER "Lens ranked #1"
   - `near_duplicate` = perceptual near-dupe
   - `visually_similar` = rest
   - Default `include_similar=false` (exact + near_duplicate only)
3. `check_listing`: `listing_url` + photo URLs and/or `image_ids`, batch ≤20 listings/call → `{reuse_score, only_on_listing, exact_elsewhere[{url,domain}], cache_hit, usage}`.
   - `reuse_score` = count of **foreign exact** hits after stripping known mirrors of *this* listing_url (SE↔Zillow↔building site for same unit). Documented arithmetic, not an ML scam score.
   - Never return `is_scam` or a mystery 0–100 risk.
4. Errors always `{code, retryable, message}` — never silent partial.
5. Every response: `usage{remaining, reset}` + `cache_hit`.
6. Auth: **device-code → API key** default for headless; OAuth OK for Cursor UI / Marketplace.

## P1 (after P0)
7. Domain allow/deny + optional geo/query hint
8. Join/fingerprint across multi-photo searches (same listing cluster)

## Out of v1
Unit-number inference, crop/focus, building-vs-unit model, Jev inside MCP (optional consumer Choice only with tiny enum; rules defaults first).

## Acceptance
- Planted SE photo → `match_kind=exact` only if that listing carries the same bytes/URL (not Lens #1 / same-building lookalike)
- FB or Craigslist screenshot works via upload → `image_id`, including when the only link is the FB or Craigslist post (no canonical rental listing URL)
- Batch 20 `check_listing` without browser
- Broker/FB/Craigslist with no canonical rental listing URL → `exact` is the real twin on StreetEasy, Zillow, SpareRoom, Leasebreak, or HotPads, not a same-building lookalike. Fee/no-fee is read only from an `exact` StreetEasy page.

## Billing
Meter the **agent key**.
- Bill: finished cold `search_image`, finished `check_listing` batch
- Free: cache hits, upload→`image_id`, retryable failures; do not bill a `visually_similar` dump when caller asked exact-only
- **Price cold search > batch check**
- Monthly = prepaid credits, not a separate product
