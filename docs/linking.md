# Linking and note identity

## Decision direction

Current preferred direction:
- each note gets a stable opaque frontmatter `id`
- the `id` is not derived from the title
- filenames remain timestamp-prefixed and human-friendly
- note-to-note links use wikilinks that conceptually point at note identity, not at file paths
- normal markdown links remain valid for assets, URLs, and one-off file references

This optimizes for long-term stable identity and graph maintenance.

## Why not plain relative markdown links

Plain relative markdown links have too many downsides for note-to-note references:
- they go stale quickly when files move
- they couple identity to storage path
- they create churn during refactors
- they are awkward for LLM maintenance

They are still useful for assets and one-off file references.
They are just not the preferred graph-link model between notes.

## Why stable opaque IDs

The note identity should survive title changes.

That means:
- no title-derived IDs
- no slugified-title-as-identity model
- title and filename can evolve without forcing the identity to change

Current decision:
- use UUIDv4-derived values for note identity
- store them without dashes as 32 lowercase hex characters
- keep time ordering in the filename, not in the ID

Why this split is good:
- UUIDv4 is boring, widely understood, easy to generate, and effectively collision-free
- the dashless form is visually cleaner and friendlier in editors like Vim
- the filename already carries timestamp ordering
- the `id` only needs to represent stable opaque identity

## Why wikilinks on top of IDs

This gives a good balance:
- pleasant to read and write
- Obsidian-friendly without special configuration
- easy for agents to search exactly by ID
- future backlinks can be derived by exact-ID search

Example conceptual reference:

```md
[[550e8400e29b41d4a716446655440000]]
```

Or with display text:

```md
[[550e8400e29b41d4a716446655440000|pi native memory]]
```

The exact surface syntax may still need refinement.
The important part is the model:
- links refer to stable note identity
- not to fragile relative paths

## Tradeoffs and known downsides

This choice has downsides and they should be acknowledged.

### Downsides

- generic editors do not automatically navigate wikilinks as well as standard markdown links
- some shell and editor tooling will need lightweight conventions or helper scripts
- there is a temptation to keep adding indexing layers once identity exists

### Why these downsides are still acceptable

- the project optimizes for stable identity and graph maintenance over path literal convenience
- the vault should still be usable in plain editors because the files remain simple markdown
- retrieval for agents and tools can rely on exact ID search instead of a database

## What this project should not do

This project is not a reimplementation of:
- Obsidian
- org-roam
- a backlink database
- a sqlite-backed PKM engine

If a helper script is ever useful, it should stay lightweight, file-based, and optional.

## Driver UX and performance

This choice should support:
- readable notes in Neovim and pi
- acceptable Obsidian usage without special configuration
- shell-level traversal with simple tools
- performance that does not depend on maintaining heavy indexes

The guiding philosophy is:
- avoid magic
- optimize for user experience and long-term operability
- keep the graph simple enough for humans, shell tools, and small local models
