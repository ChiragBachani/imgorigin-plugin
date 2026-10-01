# ImgOrigin plugin (Cursor Marketplace / Grok Bot)

Install plus skills and docs for the hosted ImgOrigin MCP at `https://imgorigin.com/mcp`. The server is not in this repository.

The skill [`skills/listing-photo-check`](skills/listing-photo-check/SKILL.md) asks the user to connect ImgOrigin, then uses it, on two triggers only:

1. **Mystery post.** A screenshot, Facebook Marketplace post, Craigslist post, or broker post with no StreetEasy, Zillow, or other canonical rental listing URL. An FB or Craigslist link still fires. The agent uploads the photo (`upload_image`) and searches for the twin on StreetEasy, Zillow, SpareRoom, Leasebreak, or HotPads (`search_image` with `include_similar=false`). `exact` means the listing has the same image bytes or the same photo URL. A Google Lens rank is not `exact`.
2. **Shortlist or daily standouts.** Just before a scam-check of that batch. The agent calls `check_listing` (photo URLs and/or `image_ids`, at most 20 listings per call). `reuse_score` is the count of foreign exact hits after mirrors of that listing are removed. The tool does not return `is_scam`. The agent decides.

It does not run on every listing.

## Auth

- **Cursor UI / Marketplace:** OAuth. Install the plugin and approve the connect card once.
- **Headless** (no connect card): device-code → API key, `POST /v1/device/code`. The client sends that key as `X-API-Key` or `Authorization: Bearer`. This is the default for headless. Do not paste the key into chat.

## Fees

Fee and no-fee are not MCP fields. After an `exact` StreetEasy hit, the agent reads the fee field on that StreetEasy listing and only then states fee or no-fee.

## Pay bar

[REQUIREMENTS.md](REQUIREMENTS.md) is the locked v1 list: tools, `match_kind`, `reuse_score`, and what is billed.

Support: support@imgorigin.com · Privacy: https://imgorigin.com/privacy · Terms: https://imgorigin.com/terms
