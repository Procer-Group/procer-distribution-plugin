# The tools, in full

Every tool publishes its own input schema. Read it there rather than guessing at field names —
this file carries what the schema cannot say: which permission each tool demands, whether it
changes anything, and what it costs to call.

| Tool | Permission | Changes state | Reaches |
| --- | --- | --- | --- |
| `server_info` | `mcp:read` | No | Nothing. Answers from the process itself |
| `list_items` | `mcp:read` | No | In-memory list in this pod |
| `create_item` | `mcp:write` | **Yes** | In-memory list in this pod |
| `demonstrate_refusal` | `mcp:read` | No | Nothing |
| `competitive_landscape_health` | `mcp:read` | No | CompetitiveLandscape, over HTTP |
| `ledger_health` | `mcp:read` | No | Ledger, over HTTP |
| `storefront_health` | `mcp:read` | No | Storefront, over HTTP |
| `tape_health` | `mcp:read` | No | Tape, over HTTP |

None of them is anonymous. Every one requires a signed-in caller holding the role its policy names.

## Arguments

**`server_info`** — none. Returns the deployment name, the route it serves and the tool names it
publishes. When its tool list disagrees with a skill or a document, the tool list is right.

**`list_items`** — `limit`, an integer between 1 and 100, defaulting to 20 when omitted. Returns
the items ordered by name with the id, name and note of each.

**`create_item`** — `name`, 1 to 80 characters, and optional `note` free text. The name must be
free across the whole list: calling twice with the same name fails the second time and names the
argument at fault. Returns the created item.

**`demonstrate_refusal`** — `mode`, one of `invalid_argument`, `not_found`, `conflict`,
`forbidden`, `upstream_unavailable`, `misconfigured`. Returns the matching refusal. An unknown mode
is itself refused as an invalid argument. Reads nothing, writes nothing, affects no other call.

**`competitive_landscape_health`** — none. Reports whether this server reached CompetitiveLandscape and
was accepted by it, and the address it used. A failure says whether the problem is reachability or
credentials, which is worth knowing before blaming any other tool.
**`ledger_health`** — none. Reports whether this server reached Ledger and
was accepted by it, and the address it used. A failure says whether the problem is reachability or
credentials, which is worth knowing before blaming any other tool.
**`storefront_health`** — none. Reports whether this server reached Storefront and
was accepted by it, and the address it used. A failure says whether the problem is reachability or
credentials, which is worth knowing before blaming any other tool.
**`tape_health`** — none. Reports whether this server reached Tape and
was accepted by it, and the address it used. A failure says whether the problem is reachability or
credentials, which is worth knowing before blaming any other tool.

## Where the two permissions come from

`mcp:read` and `mcp:write` are this server's own permissions, held as client
roles in the realm. They gate whether a tool may be *called*.

## The second enforcement point

For any tool that reaches another service, that service then enforces **its own** permissions on
the forwarded token when the call arrives. So a caller needs both, and being refused by the
downstream rather than by this server is the ordinary, correct outcome for somebody who may use
this server and may not use that data. A refusal that names a downstream is not a bug in this
server.

## What is scaffolding

`list_items`, `create_item` and `demonstrate_refusal` are the samples the server was generated
with. `list_items` and `create_item` share one in-memory dictionary that empties whenever the pod
restarts, so an empty list is the normal state after a deploy, not a data loss.

They are being replaced by tools against the product's real surface. Until they are, do not build
anything on the item list, and do not describe it to a user as ProcerERP data.
