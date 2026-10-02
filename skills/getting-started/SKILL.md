---
name: getting-started
description: First-run onboarding for the BYO Meta Ads MCP. Use when the user installs meta-ads-mcp, asks how to connect Meta/Facebook Ads, needs App ID/Secret/token setup, or wants a hand-held walkthrough of creating their own Meta developer app and wiring it into Grok Bot.
---

# Getting started with Meta Ads MCP (BYO)

Walk any installer from zero to a working connector. Stay warm and clear. Use short paragraphs and numbered steps. Never ask them to paste App Secret or access tokens into chat. Secrets go only into Grok Bot setup fields, AddMcpServer env, or a secret-request UI.

## Why bring-your-own Meta app

This connector is **shared code**, not a shared Meta app you OAuth into.

- **You own the tokens.** App ID, App Secret, and access token live in *your* Meta developer account and *your* Grok Bot config.
- **More control.** You choose which ad accounts the token can touch, which permissions it has, and when to revoke it.
- **No third-party middleman.** You are not depending on someone else's App Review, rate limits, or outage.
- **Self-serve BYO works today.** Run this MCP yourself at no charge from us. A hosted tier for heavier media pipelines may arrive later; BYO setup is complete and ready now.

Private use on ad accounts you admin usually does **not** need Meta App Review. App Review matters when one Meta app serves many strangers.

## When to use

User phrasing: get started, set up Meta ads, connect Facebook Ads, create Meta app, paste token, AddMcpServer, doctor, list my ad accounts, first install, onboarding.

## When not to use

| User wants | Do instead |
| --- | --- |
| Pause / activate / change budget on live ads | Call `set_status` / `update_budget` after this skill succeeds |
| Create creatives or ads | Use `upload_image_from_url`, `create_creative`, `create_ad` (creates default **PAUSED**) |
| Raw Graph exploration | `run_graph_read` / `run_graph_write` after connect works |
| Debug a failing connect | Stay in this skill: Prerequisites → Common failures |

## Prerequisites checklist

Confirm before Step 1:

1. Access to [Meta for Developers](https://developers.facebook.com/) (same Facebook login you use for ads is fine).
2. Access to a [Meta Business Manager](https://business.facebook.com/) that owns or is assigned the ad account(s) you care about.
3. Admin (or equivalent) on those ad accounts so you can assign a system user.
4. Grok Bot (or another MCP host) where you can add a local MCP server and set env vars / plugin secrets.
5. This package built somewhere the host can run: `node …/meta-ads-mcp/dist/index.js`.

If they lack Business Manager access, stop and say so. They need an admin to invite them or to create the system user for them.

## Tools used in this skill

| Intent | Tool |
| --- | --- |
| Check env + token health (no secrets returned) | `doctor` |
| See which `act_*` accounts the token can reach | `list_ad_accounts` |
| Optional smoke test on one account | `get_account_summary` |
| Creative / video ingest paths | `get_media_upload_guide` |

Do **not** call write tools (`set_status`, `update_budget`, `create_*`, `run_graph_write`) during first-run onboarding unless the user explicitly asks after success.

---

## Step 0: Explain the path (say this once)

Tell the user, in your own words:

> You will create **your own** Meta developer app, add Marketing API, mint a token (preferably a Business system-user token), then paste App ID, App Secret, and token into Grok Bot setup fields. After that, this bot can list accounts and manage ads you assigned. Secrets never go in chat. v1 is self-serve BYO; any paid threshold for heavy use comes later.

Then ask them to open two tabs and keep this chat open:

- [developers.facebook.com](https://developers.facebook.com/)
- [business.facebook.com](https://business.facebook.com/)

Wait for them to say they are ready before Step 1.

---

## Step 1: Create a Meta developer app

1. Go to [Meta for Developers](https://developers.facebook.com/) → **My Apps** → **Create App**.
2. Pick a type that supports Marketing API (often **Business** or **Other**, depending on Meta's current wizard).
3. Name it something clear (example: `Grok Bot Meta Ads - YourBrand`).
4. Finish create and land on the app dashboard.

**Collect (not yet paste):** App ID from **App settings → Basic**.

Remind them: App Secret stays hidden until they click show/reveal. They will paste it only into Grok Bot setup, never into this chat.

---

## Step 2: Add the Marketing API product

1. In the app dashboard, open **Add products** (or **Products**).
2. Find **Marketing API** and click **Set up** / **Add**.
3. You do not need a public OAuth redirect for the preferred system-user flow. Token paste is enough for this MCP.

If Marketing API is missing from the catalog, their app type may be wrong. Have them create a new app with a Business-capable type and retry.

---

## Step 3 (preferred): Business system user + token

This is the durable path. Prefer it over Graph Explorer.

### 3a. Open Business settings

1. Go to [business.facebook.com](https://business.facebook.com/) → **Business settings** (gear).
2. Confirm they are in the Business that owns the ad account(s).

### 3b. Create a system user

1. **Users** → **System users** → **Add**.
2. Name it (example: `grok-bot-meta-ads`).
3. Role: **Admin** if they need broad management; **Employee** can work if ad account assets are assigned carefully. For first install, Admin is simpler.

### 3c. Assign ad accounts to the system user

1. Open the system user → **Add assets** (or **Assign assets**).
2. Choose **Ad accounts**.
3. Select every `act_*` they want this connector to see (including **T83** / their primary account if that is the label they use in Ads Manager).
4. Permission: at least **Manage campaigns** (or full control) so `ads_management` tools work.

If they skip this assignment, `list_ad_accounts` will look empty or error even with a valid token.

### 3d. Generate the token

1. On the system user, click **Generate new token**.
2. Select **the Meta app you just created** (must match App ID / Secret).
3. Enable at least:
   - `ads_read`
   - `ads_management`
   - `business_management` (recommended so business-scoped listing works)
4. Generate and **copy the token once**. Store it in a password manager until Grok Bot setup.

System-user tokens can be long-lived when generated this way. That is what we want.

---

## Step 3 (fallback): Graph API Explorer long-lived user token

Use only if they cannot create a system user yet.

1. Open [Graph API Explorer](https://developers.facebook.com/tools/explorer/).
2. Select **their** app (not "Graph API Explorer" default alone).
3. Add permissions: `ads_read`, `ads_management`, and `business_management` if available.
4. **Generate Access Token** and complete the Facebook login / permission dialogs.
5. Exchange for a long-lived token via Meta's documented token exchange (short-lived user tokens expire in hours). Point them at [Access Token docs](https://developers.facebook.com/docs/facebook-login/guides/access-tokens/get-long-lived).

Warn clearly: user tokens expire and are tied to a person. System users are better for bots. Plan to migrate when Business access is available.

---

## Step 4: Collect the three secrets (offline)

They should have:

| Secret | Where it came from |
| --- | --- |
| `META_APP_ID` | App settings → Basic |
| `META_APP_SECRET` | App settings → Basic (reveal) |
| `META_ACCESS_TOKEN` | System user generate token (preferred) or long-lived user token |

Optional: `META_API_VERSION` (default `v22.0` is fine).

**Hard rule for the agent:** Never ask them to paste these values into the chat transcript. Say:

> Paste App ID, App Secret, and access token only into Grok Bot plugin setup fields or AddMcpServer `env`. If the host offers a secret-request / secure input card, use that. Do not paste tokens here.

If they paste a token in chat anyway, tell them to revoke/regenerate it and use setup fields next time. Do not echo the secret back.

---

## Step 5: Wire into Grok Bot

### Path A: Marketplace / plugin setup fields

When installed as a marketplace plugin, `gettingStarted` opens this skill. Setup fields map to HTTP headers on the hosted MCP:

- `META_APP_ID`
- `META_APP_SECRET`
- `META_ACCESS_TOKEN`
- optional `META_API_VERSION`

On the hosted connector these become `Authorization: Bearer`, `X-Meta-App-Id`, and `X-Meta-App-Secret` on every request.

After they save, the host starts the MCP. Continue to Step 6.

### Path B: AddMcpServer / local MCP config

1. Ensure the package is built (`npm install && npm run build` in the project root).
2. Add a server named `meta-ads` (or similar) with:

```json
{
  "mcpServers": {
    "meta-ads": {
      "command": "node",
      "args": ["/home/box/mcp-wrappers/meta-ads-mcp/dist/index.js"],
      "env": {
        "META_APP_ID": "YOUR_APP_ID",
        "META_APP_SECRET": "YOUR_APP_SECRET",
        "META_ACCESS_TOKEN": "YOUR_SYSTEM_USER_OR_LONG_LIVED_TOKEN",
        "META_API_VERSION": "v22.0"
      }
    }
  }
}
```

On a laptop mirror, point `args` at that machine's `…/meta-ads-mcp/dist/index.js` instead.

3. Save and reconnect / restart the MCP host so tools appear.

Confirm tools like `doctor` and `list_ad_accounts` are visible before Step 6.

---

## Step 6: Verify with doctor

Call `doctor`.

**Success looks like:** env present, token accepted, no secret values returned (maybe suffix / lengths only).

**If doctor fails:**

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| Missing env | Setup fields empty or MCP not restarted | Re-enter secrets; reconnect MCP |
| Invalid OAuth / token error | Wrong token, expired user token, or token from a different app | Regenerate token on **this** app; prefer system user |
| App secret proof / signature errors | App Secret does not match App ID | Re-copy secret from the same app as App ID |

Do not proceed until doctor is healthy.

---

## Step 7: List ad accounts

Call `list_ad_accounts`.

**Success looks like:** one or more accounts with `act_` ids (and names). For the primary operator here, confirm **T83** or the expected `act_*` appears.

**If the list is empty or missing the expected account:**

1. System user was not assigned that ad account (Step 3c).
2. Token lacks `ads_read` / `ads_management`.
3. Wrong Business / wrong app selected when minting the token.

Have them fix assignment, regenerate token, update setup fields, reconnect, then re-run `doctor` and `list_ad_accounts`.

Optional: `get_account_summary` on one `act_*` to confirm insights/metadata read works.

---

## What success looks like

You can tell the user they are done when:

1. `doctor` reports healthy config (no secrets leaked).
2. `list_ad_accounts` returns their expected accounts (including T83 / target `act_*` when relevant).
3. Meta tools show as connected in Grok Bot like any other MCP.

From here they can pause/activate, adjust budgets, upload images, and create **PAUSED** ads safely.

---

## Common failures (quick card)

| Problem | What to say / do |
| --- | --- |
| Token expired | User tokens expire. Switch to a Business system-user token. |
| Wrong app | Token must be generated for the **same** app as `META_APP_ID` / `META_APP_SECRET`. |
| Missing ad account assignment | Assign the ad account asset to the system user in Business settings. |
| App Review scare | Private / own-account use typically does **not** need App Review. Review is for shipping one app to many external customers. |
| Creates went live | This MCP defaults creates to **PAUSED** when status is omitted. If something is ACTIVE, pause with `set_status`. |
| Secrets in chat | Revoke/regenerate; paste only into setup fields next time. |
| Empty `list_ad_accounts` | Assignment + permissions + correct Business, then new token. |

---

## Soft note on pricing

v1 is **self-serve BYO**: their Meta app, their token, this open connector code.

A polished hosted tier for heavier media may be defined later. Do not invent prices. If they ask, say heavier media hosting is on the roadmap and point them back to finishing connect — BYO already gives full campaign control.

---

## Agent checklist (copy for your turn)

1. Explain why BYO (control, no middleman, ready today).
2. Prerequisites.
3. Create app → Marketing API.
4. System user (preferred) or Graph Explorer fallback.
5. Assign ad accounts; mint token with `ads_management`.
6. Secrets into setup fields only (never chat).
7. `doctor` → `list_ad_accounts` → confirm expected `act_*`.
8. Celebrate success; offer next actions (list campaigns, pause, budget) only after they ask.
