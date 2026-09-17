---
name: procer-distribution
description: Connects to and drives the ProcerERP MCP server at distribution-mcp.procergroup.com — the product's own data, under the caller's own permissions. Use when working with ProcerERP data, when a ProcerERP tool refuses a call, or when connecting a harness to the ProcerERP server.
---

# ProcerERP

ProcerERP's MCP server fronts the product's CompetitiveLandscape, Ledger, Storefront and Tape services. It holds no database of its
own: every tool call is an HTTP call to a service that already enforces the product's own rules,
made **as the person using GPT** rather than as a service account.

That is the fact everything else follows from. The server never sees more than the caller does, so
a tool that refuses is usually reporting the caller's own access rather than a fault.

## Connecting

The server ships with this plugin. Enabling the plugin registers it — there is nothing to add by
hand. If tool calls answer 401, run `codex mcp login procer-distribution` to sign in again.

Sign-in is OAuth against the Keycloak realm at `keycloak.quantarcane.io`, using the public client
`procer-public-client` and the loopback callback on port `8125`. Both halves are fixed: a different
port is a redirect-uri mismatch, not a preference.

**Call `server_info` first when anything is wrong.** It answers without touching any downstream
service, so it separates "the server is unreachable or I am not signed in" from "the server is fine
and the call was refused".

## What access means here

Two permissions gate everything:

| Permission | Covers |
| --- | --- |
| `mcp:read` | Every tool that only reads |
| `mcp:write` | Every tool that changes something |

They are Keycloak client roles on `procer-public-client`, granted to a user or to a group in the realm.
A session where reads succeed and writes answer 403 is a role problem, not a connection problem —
and re-authenticating does not fix it, because the token has to be minted again *after* the role is
granted.

## The tools

| Tool | One line |
| --- | --- |
| `server_info` | What this deployment is and what it can reach. Touches nothing else |
| `list_items` | Reads the item list |
| `create_item` | Adds one item |
| `competitive_landscape_health` | Proves this server can reach CompetitiveLandscape and be accepted by it |
| `ledger_health` | Proves this server can reach Ledger and be accepted by it |
| `storefront_health` | Proves this server can reach Storefront and be accepted by it |
| `tape_health` | Proves this server can reach Tape and be accepted by it |

Read [reference/tools.md](reference/tools.md) before the first call in a session — it carries the
argument shapes and which tools change state.

> **These are the tools the server was generated with, not ProcerERP's product surface.** They are
> scaffolding, they are being replaced as the product's real capabilities are wired up, and
> `list_items` in particular reads an in-memory list that empties whenever the pod restarts. If the
> table above disagrees with what `server_info` reports, `server_info` is right and this file is
> stale — say so rather than working around it.

## Reading a refusal

A refused call names the arguments at fault and says whether retrying is worth anything. Read it
rather than resending the same call.

- **The refusal names an argument** — fix the argument and retry once.
- **The refusal is a 403** — a permission or tenant problem. Retrying cannot fix it. Tell the user
  which permission is missing.
- **The refusal is a 401** — the token expired or was never minted. Sign in again.
- **The refusal says a downstream holds nothing at that address** — the server reached the
  downstream and got a 404. That is a server-side routing fault, not a bad argument.

A refusal that fits none of these is worth a question rather than a second attempt.
STOP and use Codex's structured user-input tool when available; if it is unavailable, ask directly in chat to clarify.

`demonstrate_refusal` fails on purpose and changes nothing, so it is safe to call when you want to
see the refusal shape before relying on it.

For symptoms that survive a retry, read
[reference/troubleshooting.md](reference/troubleshooting.md).

## Rules that are easy to get wrong

**Do not cache what `list_items` returns across turns.** It reads live state that another person
with the same access can change between calls.

**A write is not confirmed until the tool says so.** `create_item` returns the created item; if the
call refused, nothing was written, and reporting otherwise to the user is worse than reporting the
failure.
