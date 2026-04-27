# Memory vs Agents

This file defines the boundary between retrievable memory and agent-facing instruction artifacts.

This boundary is important because slug memory can easily start to look like `AGENTS.md` or repo-local skills if the project does not stay disciplined.

## The core distinction

### Memory

Memory is retrievable context.

Properties:
- it is stored because it may be useful later
- it is not necessarily loaded automatically by pi
- it may be incomplete, draft-like, or provisional
- it may be organized around scopes such as private, slug, and global

Examples:
- a repo gotcha in slug memory
- a session-local open question in a private scratchpad
- a long-lived global note about a real-world workstream

### Agent artifacts

Agent artifacts are instruction surfaces.

Properties:
- they are meant to influence agent behavior directly
- they are more curated and stable
- they should be phrased like instructions, conventions, or workflows
- pi may load them automatically through existing conventions

Examples:
- `AGENTS.md`
- repo-local skills
- other explicit instruction files meant for the agent runtime

## Why slug memory is not `AGENTS.md`

Slug memory is a lightweight draft layer.

It is for:
- small repo-specific reminders
- gotchas
- recurring local context
- things that may matter again later

It is not automatically the same thing as published instruction context.

`AGENTS.md` should remain more intentional.
It is the place for instructions that should shape agent behavior by default.

## Why slug memory is not a repo-local skill

Repo-local skills are curated workflow artifacts.

They are useful when:
- a task has repeatable structure
- the instructions are stable enough to reuse
- the artifact should be discoverable as a skill

Slug memory is earlier than that.
It is the place where loose recurring facts and gotchas accumulate before they become formalized.

## Promotion paths

Useful long-term promotion paths may look like:
- hot memory -> warm memory
- hot memory -> cold-memory note
- warm memory -> `AGENTS.md`
- warm memory -> repo-local skill
- cold-memory note -> distilled instruction artifact

Promotion should usually be explicit or at least reviewable.
The promotion workflow itself may later be packaged as a skill.

## What should auto-load versus stay retrievable

### Auto-loaded or instruction-like

These belong closer to pi's instruction surfaces:
- `AGENTS.md`
- repo-local skills
- other explicit curated agent-facing artifacts

### Retrievable, not broadly auto-loaded

These belong in memory surfaces:
- warm memory
- cold-memory notes
- inbox content
- daily traces
- old checkpoints

### Session hot memory

Hot session memory is the special case.

It is still memory, not instruction, but a tiny excerpt may be auto-loaded because it represents the current local working surface.

## Design rule

When in doubt, ask:

- Is this meant to shape the agent by default?
  - then it probably belongs in `AGENTS.md` or a skill
- Is this meant to be remembered and retrieved later?
  - then it probably belongs in memory

