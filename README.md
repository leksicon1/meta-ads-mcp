# Meta Ads

Connect your Meta ad account and run it from chat.

Campaigns, budgets, creatives, and insights in Grok Bot or Cursor. New ads start off until you turn spend on.

## Pricing

Checked on calendar-month ad spend. Flat fee, not a cut of the budget.

| Monthly ad spend | Price |
| --- | --- |
| Under $500 | Free |
| $500 to $5,000 | $49 / month |
| Over $5,000 | $199 / month |

Drop back under a line and the next month follows the lower price. Billing is not enforced yet. The prices are the offer.

## Setup

1. Install the plugin from the marketplace.
2. Open the **getting-started** skill and connect your ad account.
3. Put credentials in plugin setup fields only, never in chat.
4. Run a health check, then list your ad accounts.

The hosted endpoint is `https://mcp.meta-ads.technology83.com/mcp`.

## Creative media

- Compact stills: `upload_image_from_url` → `image_hash`.
- Production images and all video: upload in Ads Manager or Graph, then pass **image_hash** / **video_id** into `create_creative`.
- Ask the agent for `get_media_upload_guide` anytime.

## License

MIT — see LICENSE. You remain responsible for Marketing API compliance on your ad accounts.
