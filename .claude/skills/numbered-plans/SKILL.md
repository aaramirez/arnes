---
name: numbered-plans
description: Turns an approved design/spec into a single written implementation plan saved as a sequentially numbered file in the current project's planes/ directory (planes/0001-<slug>.md, planes/0002-<slug>.md, ...), instead of the default docs/superpowers/specs location. Use this whenever the user asks to "write the plan", "generate a plan", "save this as a numbered plan", refers to a project's planes/ folder for review, or asks to create/use a "planning skill". Typically invoked right after a brainstorming/design discussion has been approved, as the hand-off step before (or instead of) superpowers:writing-plans's default output location. The plan is written for human review and approval — this skill never starts implementation itself.
---

# Numbered Plans

Turn an approved design into one self-contained plan document, numbered and
filed for review — the way a person reviews a pull request, not a wiki page
they have to hunt for. This skill only writes the file. It never implements
anything from it; that is a separate, later step the human explicitly starts.

## When this applies

Use this after a design has been through the normal brainstorming/design
process (see superpowers:brainstorming) and the human has approved it in
chat. This skill replaces where that approved work gets written down: one
combined spec+plan file in `<project-root>/planes/`, instead of a spec in
`docs/superpowers/specs/` plus a separately-located plan.

If there's no `planes/` directory yet, create it — its presence is what
signals a project has opted into this convention.

## Numbering

1. List `planes/*.md` in the project root.
2. Find the highest existing `NNNN` prefix (4 digits, zero-padded). If none
   exist, start at `0001`.
3. Slugify the plan's title (lowercase, hyphens, no stopwords-obsession —
   just make it readable in a file listing).
4. Write to `planes/NNNN-<slug>.md`. Never overwrite an existing numbered
   file — if asked to revise a plan, either edit that same file in place
   (preferred, while it's still under review) or create a new number and
   note in it which plan it supersedes.

## Plan document structure

Use this template. Keep sections proportional to the decision they cover —
a one-file bugfix plan should be short; a new subsystem plan can be long.
Omit a section entirely rather than filling it with filler when it doesn't
apply.

```markdown
---
status: proposal
date: YYYY-MM-DD
---

# <Title>

## Problem / Goal
What this plan accomplishes and why, in a few sentences. Link back to the
conversation's context if useful, not a transcript of it.

## Approach
The chosen approach and, briefly, what alternatives were considered and
why they lost. This is a record of decisions, not a persuasive essay.

## Architecture / Components
The pieces being built or changed and how they relate. Use a table or a
short list of components with one line each when there are several; prose
when there's really just one thing.

## Data flow
How information moves between the components, if that's non-obvious.
Skip this section if there's nothing to say beyond "it's a function call."

## Configuration
Any new config files, schemas, or settings this introduces, with the
shape of each (fields and what they mean) — enough that someone could
write a valid config file from this section alone.

## Implementation steps
A numbered, ordered list of concrete steps. Each step should be small
enough to review and verify independently. This is the part
superpowers:writing-plans would normally produce — reuse its judgment on
step granularity, just write the result into this section instead of a
separate file.

## Error handling
What happens when a step fails partially — not exhaustive edge-case
hunting, just the failure modes that would otherwise surprise whoever
implements this.

## Testing / validation
How whoever implements this will know it works. Manual steps are fine for
small projects; say so plainly rather than inventing a test suite the
project doesn't have.

## Out of scope
What was explicitly discussed and deliberately excluded, so it doesn't
get silently re-litigated or silently assumed-included later.

## Open questions
Anything left for the human to decide before or during implementation.
Empty is fine — don't invent questions to fill this section.
```

## After writing the file

Tell the human the file path and stop. Do not start implementing, do not
run `writing-plans`'s own file-writing step afterward, and do not create
any code, subagents, or skills the plan describes — that only happens if
and when the human comes back and asks for it, plan in hand.

## Relationship to superpowers:writing-plans

This skill doesn't replace the *thinking* in writing-plans — step
granularity, sequencing, and verification framing there are still good
judgment to borrow. It replaces the *filing convention*: one number, one
file, in `planes/`, combining what writing-plans would otherwise split into
a spec doc plus a separate plan doc. If a project has no `planes/`
directory and no signal the human wants this convention, prefer
writing-plans's default behavior instead.
