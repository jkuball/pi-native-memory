# Philosophy

This project is for developers driving an agent, not only for the LLM.

## Core beliefs

- user experience matters more than magic
- performance matters
- simple files matter
- explicit behavior matters
- shell-friendly operation matters
- the driver and the LLM should work hand in hand

## What this implies

### Keep the default context tiny

Pi excels because the default context is very small.
This project should preserve that.

### Prefer plain files over hidden machinery

The memory system should stay understandable in markdown files and normal shell tools.
Humans should be able to browse it directly.

### Avoid cleverness unless it earns its keep

The system should not become a fake reimplementation of:
- Obsidian
- org-roam
- a backlink engine
- a sqlite-backed personal knowledge management (PKM) system

### Housekeeping should be automatable without an LLM

Anything that can be automated programmatically should not require a model call.
If cleanup can run through a shell script, cron job, launchd service, or systemd unit, that is a feature.

Important distinction:
- structural housekeeping like stale link cleanup, stale session-ref cleanup, or simple file hygiene should prefer programmatic non-LLM tooling
- prose-oriented housekeeping, summarization, rewriting, or note cleanup may still benefit from LLM help when explicitly requested

LLMs are useful. They just should not be mandatory for the boring deterministic maintenance work.

### The vault should work with the user's tools

The project should not force one editor or one external dependency.
It should remain easy to use with pi, Neovim, Obsidian, and basic shell tools.

### Small local models and dumb tooling should still be useful

The data layout should stay simple enough that weaker local models and basic tools can still traverse it.

## Anti-goals

- do not optimize only for the LLM
- do not hide everything behind agent-only workflows
- do not add magic layers just because they seem clever
- do not let the project become slow, bloated, or opaque

## Short version

Keep it simple.
Keep it fast.
Keep it readable.
Make it pleasant for the driver.
