# Storage and loading model

## Rule

Use markdown everywhere by default.

Do not introduce JSON or JSONL unless a specific file proves genuinely awkward in markdown.

This keeps the vault:
- readable
- Obsidian-friendly
- git-friendly
- easy to inspect with shell tools
- understandable and usable for TUI-focused developers without special tooling

## Vault layout

```text
~/.pi/agent/native-memory/
  sessions/
    <slug>/
      <session-id>/
        scratchpad.md
        # checkpoints.md is reserved for later if a real need appears

  slugs/
    <slug>/
      memory.md
      inbox.md

  notes/
    20260423114819-someid.md
    20260423121504-otherid.md

  daily/
    2026-04/
      2026-04-23.md
      2026-04-24.md

  inbox/
    inbox.md
```

## What each file means

### `sessions/<slug>/<session-id>/scratchpad.md`

Private workspace of one pi session.

Contents:
- temporary notes
- open questions
- small checklists
- near-term reminders
- links to currently relevant durable notes

Properties:
- survives `pi --continue`
- survives `pi --resume` when reopening the same session
- a forked session starts with an inherited copy, then diverges independently
- survives compaction and `/tree`
- never leaks into parallel sessions

This is the main automatically loaded scope.


## `slugs/<slug>/memory.md`

Small durable memory for one cwd or repo slug.

Contents:
- repo-specific preferences
- recurring gotchas
- stable local reminders
- small conventions not worth moving into a repo-local skill yet

This overlaps with repo-local skills, but has a different role:
- slug memory is lighter and more agent-managed
- repo-local skills are more curated and explicit

A later promotion path from slug memory to repo-local skill makes sense.

## `slugs/<slug>/inbox.md`

Repo-scoped inbox.

Use it for items that probably belong to this slug, but are not yet stable enough for `memory.md`.

Examples:
- maybe this repo wants a dedicated review skill later
- recurring command should probably be documented better

## `notes/*.md`

Global durable note graph.

Use timestamp-prefixed filenames like:

```text
20260423114819-pi-native-memory.md
20260423121504-odroid-iso.md
```

Each note should have YAML frontmatter, including a stable opaque `id`.
The `id` should not be derived from the title.
Changing a title should not force link churn.

Current decision:
- use UUIDv4-derived values for the frontmatter `id`
- store them without dashes as 32 lowercase hex characters
- keep timestamp ordering in the filename instead of the ID itself

Minimum valid frontmatter:

```md
---
id: 550e8400e29b41d4a716446655440000
---
```

Everything else is optional and flow-specific.

Example with additional optional frontmatter:

```md
---
id: 550e8400e29b41d4a716446655440000
title: pi native memory
tags:
  - pi
  - memory
  - architecture
status: active
created: 2026-04-23T11:48:19Z
updated: 2026-04-23T12:32:00Z
---

# pi native memory

This note tracks the design of the custom memory extension.
```

## `daily/YYYY-MM/YYYY-MM-DD.md`

Tiny daily trace.

Daily files are not the main memory store.
They should mostly answer:
- what happened today
- which durable notes or workstreams were touched

Example:

```md
# 2026-04-23

- refined [[20260423114819-pi-native-memory]]
- decided to keep global scope out of automatic injection
```

## `inbox/inbox.md`

Global low-friction capture inbox.

This is where a future capture command or companion capture flow would append entries.

This file is intentionally not part of normal automatic context loading.

Example:

```md
# Inbox

## 2026-04-23 12:33:22
- ich muss noch milch kaufen

## 2026-04-23 12:34:01
- https://path-to-something-interesting
```

## Linking strategy

Plain relative markdown links are not the preferred note-to-note link model.
They go stale too easily when files move or are renamed.

Preferred direction:
- note-to-note references use wikilinks
- wikilinks should resolve via stable note `id`
- file paths and normal markdown links remain fine for assets, URLs, and one-off file references

Why:
- long-term stable identity matters more than raw path literals
- retrieval for agents can use exact search on note IDs
- backlinks can later be derived by searching for IDs without needing a database
- the vault stays simple and inspectable

Important constraint:
- this project is not trying to reimplement Obsidian, org-roam, or a backlink database
- if a future helper script is useful, it should remain lightweight and file-based

## Loading strategy

Pi's strength is a very small default context.
The memory system should preserve that.

Use three temperatures.

## Hot

Auto-loaded at session startup:
- private session scratchpad excerpt

Mechanism:
- plain file I/O only
- no extra LLM call
- use a small tail-style excerpt
- include file path and omitted line count

Example shape:

```md
Private scope from `.../scratchpad.md`
Showing last 12 of 41 lines.

...tail excerpt...
```

## Warm

Maybe lightly loaded:
- slug memory excerpt

Mechanism:
- same tail-style excerpt model as hot memory
- same no-LLM rule
- probably smaller than hot memory

This remains a policy choice.
Current bias: keep it opt-in or very strict.

## Cold

Never broadly auto injected:
- cold-memory notes
- inbox files
- daily logs
- full slug files

These should be accessed explicitly.

## Dynamic retrieval

Dynamic retrieval is preferred for everything that is not hot memory.

Possible mechanisms:
- read specific files directly
- use `rg`
- use a dedicated skill for global note usage

A key pattern here is cross-scope referencing:
- hot and warm memory should often keep short links or note IDs that point into colder notes
- hotter scopes are mainly for keeping relevant durable context close at hand during active work
- this should usually happen by reference, not by copying large cold-note bodies into hotter scopes

## Explicit scope helpers

A later helper command or tool may be useful for cases like:
- inject a summary of hot memory
- inject a summary of warm memory
- inject a summary of one note or one file

Important constraint:
- this is explicit, not part of normal startup loading

## Capture and inbox workflows

Cheap capture is important, but the naming is still soft.
Current placeholder names like `/in` and `/process-inbox` are not final decisions.

Desired properties for any future capture flow:
- extremely cheap
- can be used from inside pi
- can be used from shell scripts outside pi
- does not require an LLM
- writes to markdown with timestamped entries

That makes capture possible even when pi is not running.

Phase notes:
- Phase 1 does not need capture or inbox workflows implemented yet
- Phase 2 is the first likely place for them
- richer multimodal inbox workflows can come later without changing the inbox file away from markdown
- Phase 2 is also the first likely place for dedicated reader UX such as `/readmem hot`, `/readmem warm`, and maybe a picker-backed `/readmem cold`

## Session file references

Some files may store references to pi session JSONL files in frontmatter.

Example:

```yaml
session_refs:
  - host: LECTOR-MBP-JK
    path: /Users/jkuball/.pi/agent/sessions/.../session.jsonl
```

Why this matters:
- `pi --session /path/to/session.jsonl` makes these paths operationally useful
- the vault can point back to the live pi session history without copying it

Constraints:
- paths are host-specific
- references may go stale
- stale refs should be cleanable later

## Non-goals for the storage model

At least initially:
- do not mirror full pi sessions into the vault
- do not rely on JSON or databases for core state
- do not load large memory blocks automatically
- do not add special dependencies beyond what pi already ships with
- do not add sqlite indexes, backlink engines, or other heavyweight PKM infrastructure
