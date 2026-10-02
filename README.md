# Meta Ads (marketplace plugin)

**Your Meta app. Your tokens. Full control in chat.**

This plugin connects Grok Bot / Cursor to the Meta Marketing API using **bring-your-own** credentials. You keep ownership of the developer app, system user, and ad-account assignments — the connector is shared engineering, not a shared Meta app.

## Why BYO

- **Control** — you decide which accounts the token can touch and when to revoke it.
- **Clarity** — no third-party Ads middleman sitting between you and Graph.
- **Ready today** — install, complete setup fields, run `doctor` → `list_ad_accounts`.

A hosted usage tier for heavier media pipelines may arrive later; self-serve BYO works now.

## Setup

1. Install the plugin from the marketplace.
2. Open the **getting-started** skill and follow the hand-held Meta app + system-user walkthrough.
3. Paste **App ID**, **App Secret**, and **access token** into plugin setup fields only (never into chat).
4. Call `doctor`, then `list_ad_accounts`.

`mcp.json` points at `https://YOUR_HOST/mcp` until the production Worker URL is substituted at publish time.

## Creative media

- Compact stills: `upload_image_from_url` → `image_hash`.
- Production images / all video: upload in Ads Manager, Graph, or `scripts/resumable-video-upload.mjs`, then pass **image_hash** / **video_id** into `create_creative`.
- Ask the agent for `get_media_upload_guide` anytime.

## License

MIT — see LICENSE. You remain responsible for Marketing API compliance on your Meta app and ad accounts.
