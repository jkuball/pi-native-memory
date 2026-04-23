# To be discussed

## Memory vs Agents

Write a dedicated plan file for the distinction between memory and agent instruction artifacts.

Why this matters:
- the difference is becoming clear and important
- slug memory overlaps with `AGENTS.md` and repo-local skills, but is not the same thing
- the project needs a clean explanation of:
  - what belongs in private memory
  - what belongs in slug memory
  - what belongs in global notes
  - what belongs in `AGENTS.md`
  - what belongs in repo-local skills
- the project should likely describe promotion paths such as:
  - slug memory -> `AGENTS.md`
  - slug memory -> repo-local skill
  - global note -> distilled instruction artifact

Open questions for that future plan file:
- when does memory become instruction?
- what should pi auto-load as instruction versus what should stay retrievable memory?
- how much of slug memory should ever be promoted automatically, if any?
- what makes a fact or gotcha stable enough to become an agent-facing artifact?

## Links

The note-linking model is still unresolved and is important enough to deserve a focused discussion.
Potential options:
- relative markdown links
- wikilinks
- frontmatter IDs
- hybrid approaches
