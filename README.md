# ProcerERP agent plugin

Connects your coding agent to the ProcerERP MCP server at `https://distribution-mcp.procergroup.com` — the
product's own data, under your own permissions rather than a service account's.

**This repository is generated.** Every file in it is build output from the ProcerERP monorepo. Edits
here are overwritten by the next publish; open a change against the source instead.

## Install

Claude Code:

```
/plugin marketplace add Procer-Group/procer-distribution-plugin
/plugin install procer-distribution@procer-distribution-plugins
```

Codex:

```
codex plugin marketplace add Procer-Group/procer-distribution-plugin
codex plugin add procer-distribution@procer-distribution-plugins
```

Cursor, Gemini CLI, OpenCode, OpenClaw, Hermes — run this in your project:

```
npx github:Procer-Group/procer-distribution-plugin install
```

It detects which of those harnesses you have, writes the skill and the server declaration, and
prints anything it will not write for you.

Any other MCP client: add `https://distribution-mcp.procergroup.com/mcp` and set the OAuth client id to
`procer-public-client`.

## After installing

Sign in when prompted, then ask your agent to call the `server_info` tool. It answers with the
deployment name and the tools it publishes, and it touches nothing else — so it is the one call
worth making first when something is not working.

Access is granted on ProcerERP's own roles screen, per tenant: `mcp:read` and `mcp:write`.
Reads working while writes answer 403 means the write permission has not been granted, or the
token predates the grant.

Version 0.1.0+f720048f0a9f.
