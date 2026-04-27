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
- custom model-callable memory tools

## Required storage model

### Required paths

For persisted pi sessions, Phase 1 creates these paths when needed:

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
```

Cold note files created by the write workflow live under:

```text
~/.pi/agent/native-memory/notes/<timestamp>-<hint>.md
```

For in-memory pi sessions, such as `pi --no-session`, Phase 1 performs zero automatic file creation.

The vault root is hardcoded in Phase 1:

```text
~/.pi/agent/native-memory/
```

There is no environment variable override in Phase 1.
If configurability becomes necessary later, prefer an explicit settings file or pi settings integration.

### Reserved but not required in Phase 1

These paths are part of the broader design, but Phase 1 does not create or manage them:

```text
~/.pi/agent/native-memory/
  sessions/<slug>/<session-id>/checkpoints.md
  slugs/<slug>/inbox.md
  daily/<YYYY-MM>/<YYYY-MM-DD>.md
  inbox/inbox.md
```

## Identity model

For persisted sessions:
- derive `<slug>` from the parent directory name of `ctx.sessionManager.getSessionFile()`
- derive `<session-id>` from `ctx.sessionManager.getSessionId()`
- never compute hot-memory identity from cwd alone

For fork inheritance:
- only copy parent hot memory on `session_start` with `reason: "fork"`
- resolve the parent slug from `path.basename(path.dirname(event.previousSessionFile))`
- parse the parent session ID from the first JSONL header line in `event.previousSessionFile`
- copy the parent scratchpad to the child scratchpad only if the child scratchpad does not already contain content
- if the parent cannot be resolved confidently, touch an empty child scratchpad instead of guessing
- do not add a marker comment or mutate copied content in Phase 1

For in-memory sessions:
- do not create hot memory
- do not create warm memory
- do not create the notes directory
- inject only a short system-prompt notice that automatic hot and warm memory are disabled

## File contracts

### 1. Private scope

File:
```text
sessions/<slug>/<session-id>/scratchpad.md
```

Contract:
- one file per persisted pi session
- mutable working surface
- isolated from parallel sessions
- reused when the same session is resumed or continued
- not shared with other sessions in the same slug
- a forked session starts with an inherited copy of the parent's scratchpad, then diverges independently
- created as a zero-byte file when missing
- no required frontmatter
- no required heading or template text

This file is the only scope whose contents are automatically loaded in Phase 1.

Hot-memory write guidance:
- use the visible hot memory file directly for small session-local updates
- optimize the last twelve lines because the injected excerpt is tail-based
- append concise bullets by default
- edit stale lines when they would mislead future continuation
- if the hot file becomes a long log, create or update a short current-state section near the end

### 2. Slug scope

File:
```text
slugs/<slug>/memory.md
```

Contract:
- one file per persisted cwd or repo slug
- persistent across many sessions in the same slug
- stores small repo memory and gotchas
- is not instruction context
- must stay distinct from `AGENTS.md` and repo-local skills
- created as a zero-byte file when missing
- no required frontmatter
- no required heading or template text

Phase 1 decision:
- warm memory stays in Phase 1 as a non-authoritative draft layer for repo memory
- it is a staging area for small gotchas, references, and candidate memories that should survive sessions but are not yet curated instructions
- it is never content-loaded automatically in Phase 1
- the system prompt only injects its path and line count
- if a warm-memory item becomes a stable project rule, a later workflow may promote it to `AGENTS.md` or a repo skill

Warm memory should not become a hidden `AGENTS.md`.
If warm memory conflicts with explicit user instructions or project instruction files, the explicit instructions win.

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
Phase 1 should not add `created` or `updated` by default.
Those fields should be reconsidered after the MVP works.

A new cold note may consist of only frontmatter.
If a title is provided by the note creation script, write it as an optional H1 in the body, not as required frontmatter.

Example:

```md
---
id: 550e8400e29b41d4a716446655440000
---

# Some title
```

## Required behaviors

### Private scope behavior

Phase 1 must support:
- creating `scratchpad.md` if missing for persisted sessions
- reading `scratchpad.md`
- updating `scratchpad.md`
- isolating state by exact session ID
- restoring the same session-local file on session resume or continue
- inheriting the parent scratchpad on fork when the parent can be resolved confidently

### Slug scope behavior

Phase 1 must support:
- creating `memory.md` if missing for persisted sessions
- reading `memory.md`
- updating `memory.md`
- sharing this file across sessions in the same slug
- keeping it separate from hot memory
- keeping it separate from instruction context

### Global note behavior

Phase 1 must support:
- creating a new note file under `notes/` through the `memory-write` workflow script
- writing minimum frontmatter correctly
- reading an existing note by path
- updating an existing note by path

Phase 1 does not need note-opening by ID or full link-resolution workflows.
Simple file-based creation and reading is enough.

### Empty file behavior

For hot and warm files:
- missing files are created for persisted sessions, then treated as empty
- zero-byte files are empty
- whitespace-only files are empty
- blank-line-only files are empty
- files containing comments are not empty
- files containing frontmatter are not empty, although hot and warm memory do not require frontmatter

Implementation can use normalized text plus `trim()` or an equivalent regex check.

## Automatic loading contract

### What auto-loads

Only hot memory contents auto-load in Phase 1.

The Phase 1 extension injects the memory block during `before_agent_start` for each agent turn.
The excerpt is recreated each turn using pure file I/O.
No LLM call is used for memory loading.

### What does not auto-load

Phase 1 must not auto-load:
- warm memory contents
- cold-memory notes
- daily files
- inbox files
- reserved checkpoint files

Warm memory gets a presence signal only: path plus line count.
Cold memory is gated behind the `memory-read` skill and explicit file access.

### Hot-memory loading format

Automatic loading must:
- use pure file I/O only
- perform no LLM summarization
- read a tail-style excerpt from the hot-memory `scratchpad.md`
- include the hot-memory file path
- include omitted-line information when line truncation happens
- preserve the selected lines exactly except for line-ending normalization

Do not:
- sanitize markdown
- strip links
- parse markdown
- collapse whitespace
- remove blank lines inside the excerpt
- apply a character cap in Phase 1

### Phase 1 loading budget

Default hot-memory auto-load budget:
- last **12 logical lines**
- no character cap

Line handling:
- normalize `\r\n` and `\r` to `\n`
- count logical lines after normalization
- ignore one final empty split artifact caused by a trailing newline
- preserve the exact selected lines

If the file is empty or whitespace-only, inject an empty-state notice instead of an excerpt.

If the file has twelve lines or fewer, load all lines.
If the file has more than twelve lines, load only the last twelve lines.

### Persisted-session prompt block

Initial Phase 1 system-prompt block for persisted sessions:

```md
## Memory

You have access to pi-native-memory.

Use the visible hot memory file directly for small session-local updates.
Use `memory-read` when you need to inspect memory beyond the excerpt or read warm or cold memory.
Use `memory-write` when the user asks you to remember something, when you need help choosing the right memory scope, or when durable repo or global memory should be recorded. Do not write warm or cold memory for routine task progress; use the private hot memory file for that.

Private hot memory file: `/path/to/scratchpad.md`.
Repo warm memory file: `/path/to/memory.md` (23 lines, not loaded).

Private hot memory excerpt:
Showing last 12 of 41 lines.

<excerpt>
```

Empty hot-memory variant:

```md
## Memory

You have access to pi-native-memory.

Use the visible hot memory file directly for small session-local updates.
Use `memory-read` when you need to inspect memory beyond the excerpt or read warm or cold memory.
Use `memory-write` when the user asks you to remember something, when you need help choosing the right memory scope, or when durable repo or global memory should be recorded. Do not write warm or cold memory for routine task progress; use the private hot memory file for that.

Private hot memory file: `/path/to/scratchpad.md`.
Repo warm memory file: `/path/to/memory.md` (empty, not loaded).

The private hot memory file is currently empty.
```

Warm memory signal rules:
- always include the warm file path for persisted sessions
- include line count only
- never include warm file contents in Phase 1
- report whitespace-only warm files as empty
- if the warm file cannot be read, report that the line count is unavailable

### In-memory prompt block

For in-memory sessions, such as `pi --no-session`, inject only:

```md
## Memory

You have access to pi-native-memory.

This pi session is in-memory, so automatic hot and warm memory are disabled for this session.
Use `memory-read` and `memory-write` only for explicitly requested cold-memory work.
```

## Agent interface required

Phase 1 must provide two mandatory skills through `resources_discover`:

```text
memory-read
memory-write
```

The extension package owns and exposes these skills.
The user may later provide alternative read or write skills that use different tooling, but the Phase 1 package must work on its own.

Recommended skill descriptions:

```yaml
---
name: memory-read
description: Inspect pi-native-memory files. Use when you need to read hot session memory, warm repo memory, or explicitly requested cold notes. Do not use for ordinary repository files.
---
```

```yaml
---
name: memory-write
description: Write pi-native-memory files. Use when the user asks to remember something, when session-local state should survive resume, or when durable repo or cold memory is clearly warranted. Do not use for routine task logs.
---
```

The skills must tell the agent how to:
- inspect hot memory
- inspect warm memory
- inspect a cold-memory note by file path
- create a new global note manually through the provided script
- pin a link or note ID from cold memory into hot or warm memory when that helps the current work
- keep warm memory non-authoritative and distinct from `AGENTS.md`

`memory-read` warm-memory policy:
- read warm memory when the task is repo-specific and the prompt shows the warm file is non-empty
- read warm memory when the user asks to inspect repo memory
- do not read warm memory for every task
- if warm memory is empty, do not read it unless needed to confirm or create the file
- treat warm memory as repo-specific notes and gotchas, not instruction context

`memory-write` hot-memory policy:
- for hot memory, optimize the last twelve lines
- the injected excerpt is tail-based, so put the current continuation state near the end
- append concise bullets by default
- edit stale lines when they would mislead future continuation

`memory-write` warm-memory policy:
- warm memory is a draft layer
- do not use it for authoritative project instructions
- if a warm-memory item becomes a stable rule, suggest promoting it to `AGENTS.md` or a repo skill in a later workflow

`memory-write` cold-note script:
- provide a small script such as `skills/memory-write/scripts/new-note.sh`
- create filenames as `YYYYMMDDHHMMSS-<hint>.md` using local time
- generate UUIDv4-derived dashless lowercase `id` frontmatter
- use `untitled` when no hint exists
- optionally write a body H1 when a title is provided
- do not add `created`, `updated`, `title`, `tags`, or other optional frontmatter by default

Phase 1 may use existing `read`, `write`, `edit`, and shell capabilities for file operations.
Phase 1 must not register custom model-callable memory tools.

## Diagnostic command

Phase 1 should provide one read-only diagnostic slash command:

```text
/memory
```

The command should print active memory paths and status for persisted sessions.

Suggested shape:

```text
pi-native-memory

Vault:   ~/.pi/agent/native-memory
Slug:    <slug>
Session: <session-id>

Hot:     ~/.pi/agent/native-memory/sessions/<slug>/<session-id>/scratchpad.md
Warm:    ~/.pi/agent/native-memory/slugs/<slug>/memory.md
Notes:   ~/.pi/agent/native-memory/notes/

Hot status: empty
Warm status: 23 lines
```

For in-memory sessions, `/memory` should report that automatic hot and warm memory are disabled and should not create files.

Phase 1 does not need:
- capture commands
- inbox processing commands
- promotion commands
- link-resolution commands
- `/readmem`, `/writemem`, `/remember`, or picker-backed memory commands

## Extension surface

Phase 1 extension surface:
- `resources_discover` to expose mandatory skills
- `session_start` to initialize persisted-session files and handle fork inheritance
- `before_agent_start` to inject the `## Memory` system-prompt block
- `/memory` diagnostic command

No custom memory tools are registered in Phase 1.

## Explicitly out of scope

### Out of scope for Phase 1

- inbox processing workflows
- capture command naming and implementation
- shell capture tooling
- checkpoints as a real workflow feature
- automatic checkpointing around compaction
- promotion from slug memory into `AGENTS.md` or repo-local skills
- automatic warm-memory content loading
- note-opening by ID
- backlink derivation workflows
- multimodal capture flows beyond normal markdown links or file paths
- custom memory tools
- environment-variable vault configuration

### Out of scope by philosophy unless they clearly earn their keep later

- sqlite or database indexing layers
- backlink engines
- fake PKM application features
- hidden heavy compatibility layers for external tools
- any dependency beyond what pi already ships with, plus whatever external tools the user independently prefers

## Developer tooling preferences

Phase 1 should include lightweight developer tooling for the extension implementation.

Preferred shape:
- use a Nix flake
- structure the flake with `flake-parts`
- provide a dev shell for implementation and tests
- wire `git-hooks.nix` through `prek`, matching the user's usual project blueprint

Once Phase 1 exists, reusable tooling preferences like the flake plus hooks blueprint are good candidates for cold memory.

## Testing approach

Prefer pure function tests for extension-owned behavior.
Avoid brittle snapshots of pi's full system prompt.

Implementation should isolate deterministic helpers where practical, for example:
- memory path resolution
- file initialization
- fork inheritance
- line counting
- hot tail extraction
- prompt block generation
- cold note filename and ID generation

Pure tests should cover:
- persisted session with empty hot and empty warm memory
- persisted session with hot memory shorter than twelve lines
- persisted session with hot memory longer than twelve lines
- warm line count shown while warm contents are not included
- whitespace-only files treated as empty
- in-memory sessions inject the disabled notice and create no files
- fork inheritance copies parent hot content when the parent can be resolved
- fork inheritance does not overwrite an existing child scratchpad

Integration tests can be added after the MVP works.
NixOS VM tests are plausible later, especially once the dev shell is in place, but they are not required for the initial MVP.

## Acceptance tests

### Test 1: session isolation

Given two parallel pi sessions in the same repo,
when each session writes different private notes,
then each session must only see its own `scratchpad.md`.

### Test 2: session resume

Given a session with an existing `scratchpad.md`,
when the same session is resumed or continued,
then its hot memory must be restored from that same file.

### Test 3: fork inheritance

Given a session with an existing `scratchpad.md`,
when the session is forked,
then the child session must start with a copied scratchpad and diverge independently.

### Test 4: slug persistence

Given two different persisted sessions in the same slug,
when one session updates `slugs/<slug>/memory.md`,
then the other session must be able to inspect the same slug file.

### Test 5: global note creation

Given an empty `notes/` directory,
when a new global note is created through the write workflow script,
then the note file must:
- have a timestamp-prefixed filename
- contain minimally valid frontmatter
- contain a UUIDv4-derived dashless lowercase `id`
- be readable as plain markdown

### Test 6: tiny default context

Given a long hot-memory scratchpad,
when Phase 1 prompt injection runs,
then only the configured hot-memory tail excerpt must be injected.
Warm memory contents and cold notes must remain unloaded by default.

### Test 7: warm presence signal

Given a non-empty warm memory file,
when Phase 1 prompt injection runs,
then the prompt must include the warm memory path and line count, but not its contents.

### Test 8: in-memory session isolation

Given an in-memory pi session,
when Phase 1 starts and injects the memory prompt,
then it must not create hot memory, warm memory, or notes directories automatically.

### Test 9: shell and editor readability

Given the resulting filesystem layout,
then a user should be able to inspect it sanely with:
- shell tools
- Neovim
- Obsidian

No special database or index must be required.

## Open points still allowed in Phase 1

Only these points are still intentionally soft:
- exact final prompt wording after testing with recent models
- exact script interface for `memory-write/scripts/new-note.sh`
- exact formatting of the `/memory` diagnostic command output
