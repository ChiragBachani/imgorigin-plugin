---
name: listing-photo-check
description: >
  Use ImgOrigin only on two housing-agent triggers, and ask the user to connect
  it when those tools are missing. Trigger 1: the user drops a screenshot,
  Facebook post, or broker post with no listing URL — upload_image, then
  search_image for the StreetEasy or Zillow twin with include_similar false
  (exact and near_duplicate only). Trigger 2: about to scam-check a shortlist
  or daily standouts — check_listing on that batch and read reuse_score as
  foreign-exact evidence only. Do not use on every listing. A Lens rank is
  not exact. Never invent is_scam from tool output.
---

# ImgOrigin — two connect triggers

ImgOrigin is the hosted MCP at `https://imgorigin.com/mcp`. Use it only after you name one of the two triggers below. If you cannot name one, stop. Do not connect, and do not call a tool.

Pay bar and field definitions: `REQUIREMENTS.md` in this plugin.

## Name the trigger

| Fire | Situation |
| --- | --- |
| 1. Mystery post | The user dropped a screenshot, a Facebook post, or a broker post, and the message has no listing URL you can open. |
| 2. Shortlist / standouts | You are about to scam-check a shortlist or the daily standouts, before you bother the human with that batch. |

Do not fire for an ordinary summary, a tour, a rent comparison, or a listing that already has a URL when you are not running that scam-check. One listing you happen to be reading is not trigger 2. A photo question that already includes the listing URL is not trigger 1.

## Connect, then use

Cursor UI uses the OAuth connect card. Headless clients that cannot show a card use device-code → API key (`POST /v1/device/code`). The client sends it as `X-API-Key` or `Authorization: Bearer`. Never ask the user to paste a key or token into the transcript.

Trigger 1 needs `upload_image` and `search_image`. Trigger 2 needs `check_listing`, and `upload_image` only when a photo is an attachment rather than an https URL. If a tool that this trigger needs is missing, ask the user to connect ImgOrigin and wait:

> ImgOrigin isn't connected. I need it for this photo (StreetEasy/Zillow twin, or the shortlist photo check). In Cursor, approve the ImgOrigin connect card. On a headless client, finish device-code login (`POST /v1/device/code`) so the client stores an API key. I will not paste a key into chat.

If `search_image` cannot take `include_similar` and return `match_kind`, stop trigger 1. Do not send `image_base64`. Do not rank a Google Lens result and call #1 `exact`.

## Trigger 1 — screenshot / FB / broker post, no listing URL

Goal: the StreetEasy or Zillow listing that hosts this same photo. Not a same-building lookalike.

1. Connect if the tools are missing.
2. For each photo in the post, call `upload_image` with one of: the chat attachment id, a box file path, or an https URL. Take the returned `image_id` (content-hash). Never put base64 in the tool arguments.
3. Call `search_image` with that `image_id` and `include_similar=false`. You may pass `image_url` only when the photo is already a public https URL and you did not need an upload. Leave `include_similar` false. That call returns `exact` and `near_duplicate` only.
4. Keep hits whose `domain` is StreetEasy or Zillow. Read `match_kind` off the tool. Do not assign it yourself.

| `match_kind` | What you may say |
| --- | --- |
| `exact` | This listing carries the same bytes (content-hash) or the same photo URL. This is the twin. |
| `near_duplicate` | Perceptual near-dupe. Say near-duplicate. It is not the proven same file. |
| anything else, or a Lens rank with no `match_kind` | Not a twin. Ignore it. `visually_similar` is out of this call. |

5. Report each kept hit as: `{match_kind}` on `{domain}` — `{url}` (`{title}` and `{confidence}` when the tool sent them).
6. No StreetEasy or Zillow `exact` or `near_duplicate`: say you did not find a twin. Do not fill the gap with a visually similar listing, a Lens #1, or a same-building lookalike.
7. Fee / no-fee, including when the post itself says "no fee": that is not an MCP field. Open the `exact` StreetEasy listing and read StreetEasy's fee field. State fee or no-fee only from that page. A Zillow-only hit, a `near_duplicate`, or a post that claims no-fee is not a fee reading. If you have no `exact` StreetEasy page, say the fee is unverified.

Trigger 1 stops at the twin (and the StreetEasy fee read). Do not also run `check_listing` unless trigger 2 applies.

## Trigger 2 — about to scam-check a shortlist or daily standouts

Goal: foreign-exact photo evidence for the batch, before you send it to the human. You decide what to do with it.

1. Connect if the tools are missing.
2. Call `check_listing` in batches of at most 20 listings. For each listing pass `listing_url` plus its photo URLs and/or `image_ids` (from `upload_image` when the photo is only an attachment). Send direct photo URLs, not the listing HTML page. If the live schema caps photos per listing, send the main photo first and stay inside that cap.
3. Read the fields the tool returns. Do not add fields.

| Field | Meaning |
| --- | --- |
| `reuse_score` | Count of **foreign exact** hits after the server strips known mirrors of this `listing_url` (StreetEasy ↔ Zillow ↔ the building site for the same unit). Arithmetic, not a probability and not a 0–100 risk. |
| `only_on_listing` | No foreign exact hit. |
| `exact_elsewhere` | `{url, domain}` of those foreign exact pages. |
| `cache_hit`, `usage.remaining`, `usage.reset` | On every response. A cache hit is not a new billed search. |

4. One line per listing: `{listing_url}: reuse_score={n} foreign exact. Elsewhere: {exact_elsewhere urls, or none}.`
5. Decide yourself whether to drop the listing, flag it, or still show it. Say that decision in your own words. `reuse_score` above zero is a reason to look, not a label. Agencies and cross-posts reuse photos too.
6. The tool does not return `is_scam`. Do not invent `is_scam`, "likely scam", or a risk percent from `reuse_score`, from how many sites appear, or from a Lens rank. A same-building lookalike is not an exact hit and does not raise `reuse_score`.

## Errors

Errors are `{code, retryable, message}`. Never treat a missing or partial body as success. If `retryable` is true, retry that call once. If the message says the allowance is used up, tell the user and pass along an upgrade link when the error includes one.

Uploads and cache hits are free. Do not flip `include_similar` to true to force a `visually_similar` dump.
