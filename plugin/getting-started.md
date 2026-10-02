# Marketplace gettingStarted

Point plugin `gettingStarted` at the skill named `getting-started`.

Setup variables (secret fields) map to HTTP headers on every MCP call:

| Variable | Header |
| --- | --- |
| `META_ACCESS_TOKEN` | `Authorization: Bearer …` |
| `META_APP_ID` | `X-Meta-App-Id` |
| `META_APP_SECRET` | `X-Meta-App-Secret` |

Replace `YOUR_HOST` in `mcp.json` with the deployed Workers (or Node) HTTPS origin before publish.
