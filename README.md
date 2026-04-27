# pi-native-memory

A (planned) custom memory extension for pi.
This repo starts as design-first documentation and will be iterated on in my free time.

There are many memory systems, but this one is mine. The goal is "Memory, the pi way".
Small by default, explicit when needed, and aligned with the same strengths that pi excels at.

## Why this exists

There are many memory systems and extensions. They all get many things right but nothing really clicked for me. This sounds like an xkcd 927 type of situation, but this is not designed as a standard. This is designed to be helpful for me first.

My workflow with pi is very session-hoppy.
I often close and reopen pi quickly with `pi --{continue,resume,fork}`.

I am building something that plays to pi's strengths instead:
- very small default context
- strong session model
- explicit session resume
- compaction hooks
- dead-simple markdown-first storage
- adhd-friendly low-friction capture during active work

## Project status

The project is currently in Phase 0, exploration and conception.

The current design direction has:
- differently scoped memory for hot, warm, and cold memory. (**hot memory** for one pi session, **warm memory** for one cwd or repo slug, **cold memory** as a Zettelkasten-style note graph)
- a simple markdown-first cold-memory model that is easy to inspect for any TUI-focused developer
- tiny automatic loading of hot memory at session startup, with dynamic retrieval for everything else
- hot and warm memory used mainly for short working references into colder notes
- capture and inbox ideas as important adjacent features, but not necessarily part of the first core implementation

The storage model is intentionally Obsidian-friendly and git-friendly.
This section should evolve gradually as the project becomes more concrete.

The main open planning task now is to define a clean boundary between Phase 1 and Phase 2.
Current bias:
- Phase 1 should prove the hot-memory shape, warm-memory shape, cold-memory shape, tiny startup-only loading, and skills-based agent workflow
- Phase 2 should make the system pleasant to use, including better capture, promotion, and driver-facing workflows such as `/readmem`

## Related projects and inspirations

### `pi-memory`

Source: <https://github.com/jayzeng/Pi-Memory>

Things I like from it:
- durable markdown memory files
- useful memory read and write primitives
- optional qmd integration
- clear attempt to make memory practical in pi

Things that do not fit this project as well:
- the shutdown-oriented summarization model does not match a workflow with frequent close and reopen cycles
- daily-log centered organization is not a great fit for long-lived, low-priority workstreams in my workflow
- this project wants to lean harder into pi's tiny default context and explicit retrieval model

This project was initially inspired by `pi-memory`, but aims at a different workflow shape.

### Karpathy's "LLM Wiki"

Source: <https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f>

Things I like from it:
- markdown as the primary durable artifact
- Obsidian-friendly vault thinking
- persistent, compounding knowledge instead of re-deriving everything from raw sources every time
- shell-tool and search-tool friendliness

Where this project differs:
- this project is built around pi sessions and pi's tiny default context
- it includes hot session memory and warm repo memory, not only a cold global wiki layer
- it also cares about driver UX, shell tooling, and long-term operability without assuming an LLM is always in the loop

## Docs

- `docs/plan.md`
- `docs/phase-1-spec.md`
- `docs/storage-and-loading.md`
- `docs/linking.md`
- `docs/memory-vs-agents.md`
- `docs/philosophy.md`
- `docs/nomenclature.md`
- `docs/capture.md`
- `docs/to-be-discussed.md`
