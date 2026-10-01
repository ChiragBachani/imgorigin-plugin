---
name: imgorigin-listing-photo-check
description: Check whether a listing's photos (apartment, rental, marketplace item) already appear on other websites, using the ImgOrigin connector. Use on every new listing you evaluate, before recommending it.
---
Use only the ImgOrigin connector tools. Never ask the user for an API key or put tokens in chat. If the connector is not connected, ask the user to connect it from the plugin's sign-in card.

For each new listing, call `check_listing` with up to 3 direct photo URLs (main photo first) and the listing's page URL. Direct photo URLs end in an image file or come from the site's image host, not the listing page itself. Checks of the same photos again are free, so it is fine to re-check.

Report evidence, not a verdict: "photos also appear on N other sites (examples: …)". Reused photos are a common sign of a fake listing, but not proof: agencies and cross-posting also reuse photos. Never call a listing a scam.

Use `search_image` for a single photo when the user asks where an image comes from.

If a tool returns an error with `retryable: true`, wait `retry_after_seconds` and try once more. If it says the free allowance is used up, tell the user and share the upgrade link it returns.
