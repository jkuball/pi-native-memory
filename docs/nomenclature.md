# Nomenclature

This file tracks currently used project terms and their current meaning.

A term appearing here is not necessarily finalized.
Some names are placeholders that may change during design.

## Stable enough terms for now

### Private scope

Session-local working memory.

Properties:
- tied to one pi session
- survives resume, compaction, and `/tree`
- never leaks into parallel sessions

### Slug scope

Repo or cwd-scoped memory stored centrally in the memory directory.

Properties:
- lighter and more draft-like than `AGENTS.md`
- useful for gotchas and small repo context
- may later be promoted into `AGENTS.md` or repo-local skills

### Global scope

Durable Zettelkasten-style note graph.

Properties:
- cross-session
- cross-repo
- long-lived
- not automatically injected into context

### Hot, lukewarm, and cold memory

A loading metaphor.

- **hot** = automatically loaded private scope
- **lukewarm** = maybe lightly loaded slug scope
- **cold** = global notes, inbox, daily files, full checkpoints, and other explicitly accessed material

## Intentionally soft terms

### Vault

Currently used as a convenient name for the top-level memory directory.

This may change later.
It should not yet be treated as fixed branding or as a hard technical term.

### `/in`

Current placeholder name for a future cheap capture command.

Not finalized.
The capture concept is important. The command name is not settled.

### `/process-inbox`

Current placeholder name for a future interactive inbox-processing workflow.

Not finalized.

### Inbox

Currently means a cheap dumping ground for captured items before they are processed.

The concept is likely to stay.
The exact workflow and naming may change.

### Checkpoint

Currently means a cheap markdown marker in session-local storage.

It does not necessarily mean an LLM-generated summary.

### Stable note ID

A note identity concept, distinct from title and filename.

Current direction:
- each note gets an opaque frontmatter `id`
- use UUIDv4-derived values for that `id`
- store them without dashes as 32 lowercase hex characters
- titles can change without changing identity
- note-to-note links should conceptually point at the `id`

### Wikilink

Current preferred note-to-note link style.

This should be treated as an identity-oriented note reference, not merely as Obsidian-specific syntax.

## Open terminology questions

- Should `vault` remain the top-level name?
- What should the final capture command be called?
- What should the final inbox-processing command be called?
- How exactly should ID-based wikilinks be rendered in final file conventions?
