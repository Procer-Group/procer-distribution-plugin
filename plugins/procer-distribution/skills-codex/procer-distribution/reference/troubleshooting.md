# When it does not work

Run the checks in order. The first two are cheap and settle most cases.

| # | Check | Proves |
| --- | --- | --- |
| 1 | Call `server_info` | Separates "unreachable or not signed in" from "signed in, call refused" |
| 2 | Open `https://distribution-mcp.procergroup.com/health` in a browser | The deployment is up. It answers `{"status":"ok"}` anonymously |
| 3 | Call `competitive_landscape_health` | This server can reach CompetitiveLandscape and be accepted by it |
| 3 | Call `ledger_health` | This server can reach Ledger and be accepted by it |
| 3 | Call `storefront_health` | This server can reach Storefront and be accepted by it |
| 3 | Call `tape_health` | This server can reach Tape and be accepted by it |
| 4 | Call `demonstrate_refusal` with `forbidden` | The refusal path renders correctly in this harness |

## Symptom index

| Symptom | Cause | Fix |
| --- | --- | --- |
| Browser shows a redirect-uri mismatch during sign-in | The callback port in this harness's config is not one the realm registers | Use port `8125`, or register the port you used on `procer-public-client` |
| Sign-in never opens a browser, or fails naming client registration | The harness is trying dynamic client registration, which this realm refuses | The harness must be told to use client id `procer-public-client`. A harness with no field for it cannot sign in to this server |
| Every tool answers 401 | No token, or an expired one | Sign in again. If it recurs immediately, the token's audience does not name this server |
| Every tool answers 403 | The account holds neither permission | Grant the client role carrying `mcp:read`, then sign in again |
| Reads work, `create_item` answers 403 | The account has `mcp:read` but not `mcp:write` | Grant the write permission, then sign in again — the old token does not gain it |
| A permission was granted but the call still answers 403 | The token was minted before the grant | Sign out fully and back in. Refreshing is not enough |
| `list_items` returns an empty list | The pod restarted. The sample list lives in memory | Working as designed. It is scaffolding, not a catalogue |
| `competitive_landscape_health` refuses, naming an address | CompetitiveLandscape holds nothing at that path | A server-side routing fault. Report the address it named |
| `ledger_health` refuses, naming an address | Ledger holds nothing at that path | A server-side routing fault. Report the address it named |
| `storefront_health` refuses, naming an address | Storefront holds nothing at that path | A server-side routing fault. Report the address it named |
| `tape_health` refuses, naming an address | Tape holds nothing at that path | A server-side routing fault. Report the address it named |
| `server_info` answers but names a different deployment | The harness is pointed at another host | Check the URL in this harness's MCP config |
| Tools are missing entirely from the session | The MCP server is configured but not loaded | Reload the harness. Most do not pick up a new server mid-session |


## What this cannot tell you

If `/health` answers and `server_info` still fails for every account in every harness, the problem
is in the deployment rather than in any client — the realm, the ingress route, or the token
audience. That is a change in `ProcerERP/src/Distribution/MCP/`, not here.
