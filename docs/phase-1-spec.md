# Phase 1 spec

## Purpose

Phase 1 proves the core shape of pi-native-memory.

It should be the smallest implementation that demonstrates:
- hot session memory
- warm repo memory
- cold memory as markdown notes
- tiny automatic loading
- zero LLM dependency for core loading behavior

Phase 1 is not a full workflow system.
It is the smallest useful core.

## Non-goals

Phase 1 is not trying to solve:
- inbox workflows
- capture UX
- promotion flows
- compaction integration
- backlink management
- search infrastructure
- Obsidian feature parity
- repo-local skills or `AGENTS.md` generation

## Required storage model

### Required paths

Phase 1 only requires these paths to exist when needed:

```text
~/.pi/agent/native-memory/
  sessions/
    <slug>/
      <session-id>/
        scratchpad.md

  slugs/
    <slug>/
      memory.md

  notes/
    <timestamp>-<hint>.md
```

### Reserved but not required in Phase 1

These paths are part of the broader design, but Phase 1 does not need to create or manage them:

```text
~/.pi/agent/native-memory/
  sessions/<slug>/<session-id>/checkpoints.md
  slugs/<slug>/inbox.md
  daily/<YYYY-MM>/<YYYY-MM-DD>.md
  inbox/inbox.md
```

## File contracts

### 1. Private scope

File:
```text
sessions/<slug>/<session-id>/scratchpad.md
```

Contract:
- one file per pi session
- mutable working surface
- isolated from parallel sessions
- reused when the same session is resumed or continued
- not shared with other sessions in the same slug
- a forked session starts with an inherited copy of the parent's scratchpad, then diverges independently

This file is the only scope automatically loaded in Phase 1.

### 2. Slug scope

File:
```text
slugs/<slug>/memory.md
```

Contract:
- one file per cwd or repo slug
- persistent across many sessions in the same slug
- stores small repo memory and gotchas
- is not instruction context
- must stay distinct from `AGENTS.md` and repo-local skills

Phase 1 decision:
- this file is manually inspected in Phase 1
- it is not automatically loaded by default
- lightweight presence signaling can be revisited after Phase 1 if it proves useful

### 3. Cold memory

Files:
```text
notes/<timestamp>-<hint>.md
```

Contract:
- each note is a standalone markdown file
- each note has YAML frontmatter
- each note has a stable opaque `id`
- the `id` is UUIDv4-derived, lowercase, and dashless
- the `id` is not derived from the title
- the filename provides timestamp ordering and a small human hint
- the timestamp format should be `YYYYMMDDHHMMSS`
- the timestamp should use local time
- if no better hint exists, `untitled` is fine

### Required note frontmatter

Minimum required frontmatter for Phase 1:

```yaml
---
id: 550e8400e29b41d4a716446655440000
---
```

A minimally valid note only requires `id`.
Everything else is optional and flow-specific.

Additional frontmatter such as `title`, `tags`, `status`, `session_refs`, `created`, or `updated` is allowed, but not required by Phase 1.

## Required behaviors

### Private scope behavior

Phase 1 must support:
- creating `scratchpad.md` if missing
- reading `scratchpad.md`
- updating `scratchpad.md`
- isolating state by exact session ID
- restoring the same session-local file on session resume

### Slug scope behavior

Phase 1 must support:
- creating `memory.md` if missing
- reading `memory.md`
- updating `memory.md`
- sharing this file across sessions in the same slug
- keeping it separate from hot memory

### Global note behavior

Phase 1 must support:
- creating a new note file under `notes/`
- writing minimum frontmatter correctly
- reading an existing note by path
- updating an existing note by path

A new note may consist of only frontmatter.
An H1 matching the title is allowed, but not required.

Phase 1 does not need note-opening by ID or full link-resolution workflows.
Simple file-based creation and reading is enough.

## Automatic loading contract

### What auto-loads

Only hot memory auto-loads in Phase 1.
The Phase 1 auto-load happens at session startup.

### What does not auto-load

Phase 1 must not auto-load:
- warm memory
- cold-memory notes
- daily files
- inbox files
- reserved checkpoint files

### Hot-memory loading format

Automatic loading must:
- use pure file I/O only
- perform no LLM summarization
- read a tail-style excerpt from the hot-memory `scratchpad.md`
- include the file path
- include omitted-line information when truncation happens

### Phase 1 loading budget

Default starting budget for hot-memory auto-load:
- last **12 lines**
- up to **1200 characters**

If the file is shorter, load the whole file.
If the file is longer, prepend a short notice such as:

```md
Private scope from `/path/to/scratchpad.md`
Showing last 12 of 41 lines.
```

These are the initial Phase 1 defaults.
If implementation experience shows that slightly different limits work better, they can be adjusted intentionally.

## Agent interface required

Phase 1 does not need new memory-specific primitives if the agent already has generic file and shell tools.
Behavior matters more than naming in Phase 1.

Phase 1 must provide a documented workflow, such as through skills, that tells the agent how to:
- inspect hot memory
- inspect warm memory
- inspect a cold-memory note by file path
- create a new global note manually
- pin a link or note ID from cold memory into hot or warm memory when that helps the current work

This may be implemented entirely with existing read, write, edit, and shell capabilities.

Phase 1 does not need:
- capture commands
- inbox processing commands
- promotion commands
- link-resolution commands
- dedicated memory-specific slash commands

## Explicitly out of scope

### Out of scope for Phase 1

- inbox processing workflows
- capture command naming and implementation
- shell capture tooling
- checkpoints as a real workflow feature
- automatic checkpointing around compaction
- promotion from slug memory into `AGENTS.md` or repo-local skills
- automatic slug-scope loading by default
- note-opening by ID
- backlink derivation workflows
- multimodal capture flows beyond normal markdown links or file paths

### Out of scope by philosophy unless they clearly earn their keep later

- sqlite or database indexing layers
- backlink engines
- fake PKM application features
- hidden heavy compatibility layers for external tools
- any dependency beyond what pi already ships with, plus whatever external tools the user independently prefers

## Acceptance tests

### Test 1: session isolation

Given two parallel pi sessions in the same repo,
when each session writes different private notes,
then each session must only see its own `scratchpad.md`.

### Test 2: session resume

Given a session with an existing `scratchpad.md`,
when the same session is resumed or continued,
then its hot memory must be restored from that same file.

### Test 3: slug persistence

Given two different sessions in the same slug,
when one session updates `slugs/<slug>/memory.md`,
then the other session must be able to inspect the same slug file.

### Test 4: global note creation

Given an empty `notes/` directory,
when a new global note is created,
then the note file must:
- have a timestamp-prefixed filename
- contain minimally valid frontmatter
- contain a UUIDv4-derived dashless lowercase `id`
- be readable as plain markdown

### Test 5: tiny default context

Given a long hot-memory scratchpad,
when Phase 1 auto-load runs,
then only the configured hot-memory tail excerpt must be injected.
Warm and cold memory must remain unloaded by default.

### Test 6: shell and editor readability

Given the resulting filesystem layout,
then a user should be able to inspect it sanely with:
- shell tools
- Neovim
- Obsidian

No special database or index must be required.

## Open points still allowed in Phase 1

Only these points are still intentionally soft:
- final skill and command names
- whether warm memory later gets optional tiny auto-loading behind a flag or a presence-only signal
- exact note template text beyond the minimum required frontmatter
