---
name: procer-distribution-update
description: Moves the installed ProcerERP plugin to the version published on the public mirror, and reports what changed. Use when asked to update, upgrade or refresh ProcerERP, the ProcerERP plugin or the ProcerERP skills, when a ProcerERP tool is missing or behaves unlike its documentation, or when the tools the server reports disagree with what the skills describe.
---

# Updating ProcerERP

This plugin is published from ProcerERP's own repository to a public mirror, and every harness installs
it from there. Nothing arrives on its own unless the harness was told to look, so a skill that
disagrees with the server is almost always a plugin that was never refreshed.

Run this whole file top to bottom. It is one act, not a menu.

## 1. Find out whether there is anything to do

The published version is the `version` field of the manifest on the mirror's default branch:

```bash
# One HTTPS GET, no credentials, no gh CLI. Answers something like 0.1.0+e50c0e722efd.
curl -fsSL https://raw.githubusercontent.com/Procer-Group/procer-distribution-plugin/main/plugins/procer-distribution/.claude-plugin/plugin.json | grep '"version"'
```

The installed version is in this plugin's own manifest, inside the Codex plugin cache. Find it with
`codex plugin list`, which prints the installed version beside the plugin name.

The two strings are compared for equality, not ordered — the part after `+` is the source commit the
build was made from, so any difference at all means the installed copy is behind.

**Equal strings mean stop.** Say so and do nothing else. Reinstalling an identical version churns the
harness's cache and proves nothing.

## 2. Install the published version

Two commands, in this order. Codex has no auto-update, so this is how every new version arrives.

```bash
# Refresh the configured marketplace snapshots, then reinstall onto what they now hold.
codex plugin marketplace upgrade
codex plugin add procer-distribution@procer-distribution-plugins
```

Start a new Codex thread afterwards. The skills loaded in this one are the old ones.

## 3. Say what actually happened

Report the version it moved from and to, and name the reload the user still owes. An update that is
installed but not reloaded looks exactly like an update that did not happen, and the user is the only
one who can tell the difference from the outside.

If a command failed, give its output rather than a summary of it. The two failures worth naming:

- **The marketplace is not registered.** Nothing to update; the plugin was installed some other way,
  or under another marketplace name. Install it as the mirror's README describes rather than
  guessing at a name.
- **The refresh succeeded and the version did not move.** The publish has not finished.
  ProcerERP's workflow mirrors within about a minute of a push; wait and run this again.

## What this does not do

It does not touch the ProcerERP server or any of its data — it only replaces the files this
plugin installed. `server_info` is the check for whether the *server* is reachable and current, and
it answers with the deployment that responded. When the tool list it reports disagrees with what any
skill says, the tool list is right and this file is what needs running.
