# Capture and inbox

This file isolates capture and inbox ideas from the core memory model.

The current design direction is that capture is important, but it should not distort the Phase 1 core of pi-native-memory.

## Why this needs its own file

There are two related but distinct concerns:

1. core memory behavior inside pi
2. capture and ingestion workflows around pi

The core memory behavior is currently:
- private session scratchpad
- slug memory
- global cold-storage notes
- tiny automatic loading
- explicit retrieval for everything else

Capture is adjacent to that.
It may write into the memory system, but it is not the same thing as the memory system.

## Current direction

The inbox concept is still useful.
But it should be treated as a companion workflow, not as a defining pillar of the core architecture.

That means:
- keep inbox and capture ideas in the broader roadmap
- avoid letting them bloat the Phase 1 core
- avoid turning pi-native-memory into a raw-source warehouse or multimodal archive

## What inbox is for

The inbox is a cheap dumping ground for items that are worth keeping around briefly before they are processed.

Examples:
- a reminder to remember globally later
- a possible future note
- a small captured command or idea
- a URL worth revisiting
- a repo-specific thought that may become slug memory

The inbox is not necessarily for:
- long-term storage of raw multimodal artifacts
- permanent retention of heavy source material
- becoming a second note system next to the actual note graph

## Core design constraint

A capture system should prefer referential capture over heavyweight retention.

Preferred shapes:
- text snippets
- URLs
- local file paths
- repo paths
- small markdown entries

Non-goal by default:
- storing large binary assets forever inside the memory store

## Possible future split

A useful long-term split may be:

- **pi-native-memory**
  - owns private, slug, and global memory surfaces
  - owns durable markdown memory artifacts
  - stays small, inspectable, and pi-native

- **pi-capture** or similar companion tooling
  - performs cheap capture from inside or outside pi
  - appends to inbox files
  - maybe helps process or route captured items later
  - may eventually integrate with external clipping or ingestion workflows

This would let capture exist without forcing the main extension to become an ingestion product.

## Inbox and global reminders

Inbox is still relevant even if raw-source ingestion stays out of scope.

A simple example:
- the agent notices a global reminder or follow-up worth preserving
- it appends a short entry to a global inbox
- later, the user and agent process that inbox item into a durable note, slug memory, daily trace, or discard it

This is useful and does not require a large ingestion subsystem.

## Relationship to cold storage

The inbox is adjacent to cold storage, but it is not itself the durable knowledge graph.

A good mental model is:
- inbox items are candidate material
- notes are durable distilled artifacts
- processing moves items from candidate state into durable memory, or deletes them

## Open questions

- Should inbox remain inside pi-native-memory Phase 2, or move mostly into companion tooling?
- Should there be one global inbox, slug inboxes, or both?
- What should the final capture command be called?
- Should processing stay fully manual, or have reviewable agent assistance?
- Which inbox artifacts are worth keeping after processing, if any?
