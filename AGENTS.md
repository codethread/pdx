This folder contains Pandora's Box user configuration.
Read `PANDORA.md` before changes; it is the installed configuration reference and may be refreshed by `pdx open`.
Validate rendering with `pandora-spawn preview` after changes when available.

# Workflow policy layout guidance

Use this when writing or revising workflow/policy documents such as `policies/*/workflow*.md`.

## Goal

Make workflows clear for both humans and agents without creating multiple competing sources of truth.

Use three layers:

1. **Graphviz DOT** for the macro structure.
2. **Terse ASCII scan lines** under recipe/headings for local flow shape.
3. **Prose bullets** for authoritative operational rules.

The prose remains authoritative for exact behaviour. Diagrams and scan lines exist to make structure easier to see and reduce accidental contradictions in long prose.

## DOT guidance

Use Graphviz DOT for the top-level workflow map when the policy has real structure.

Good DOT content:

- normal lifecycle path
- core phase ordering, e.g. `plan -> build -> assess -> release`
- major ambiguity/escalation gates
- key ownership/delegation boundaries
- common feedback loops
- fan-out / fan-in points when tasks can run in parallel

Avoid graphing every recipe branch, exception, or failure class unless the graph remains readable. Put those in prose.

Always add comments or nearby prose that explain graph scope and ownership. Custom DOT attributes such as `owner`, `phase`, and `delegates_to` are acceptable as metadata for agent readers, but Graphviz may not render them. Prefer visible labels or comments for anything humans must see.

## ASCII scan-line guidance

Use terse ASCII directly under each recipe or workflow heading when a compact local view helps scanning.

Example:

```text
req -> triage -> War execute[N] -> fan-in -> Greed review? -> release checkpoint -> merge/MR
```

Conventions:

- `?` means optional or conditional.
- `[N]` means one or more tasks may fan out.
- `fan-in` means all parallel work must complete before the next checkpoint.
- Use actor names when ownership matters: `War execute`, `Greed review`, `Toil triage`.
- Keep scan lines mnemonic, not exhaustive.

## Prose guidance

Use prose bullets for exact operational rules:

- what triggers the recipe
- who owns each step
- what must be enqueued or validated
- what must not happen
- how failures route
- when to escalate

If the document uses lifecycle phases and concrete task types, add a bridge sentence/table explaining the mapping.

## Maintainability checks

Before finishing a workflow-policy edit, check:

- Does the DOT graph claim to be exhaustive? If not, say what it omits.
- Do graph labels match prose terminology?
- Are ownership/delegation boundaries visible, not only hidden in DOT attributes?
- Are fan-out/fan-in points explicit where multiple tasks can run in parallel?
- Are optional steps marked with `?` in scan lines?
- Is there one authoritative source for exact behaviour? Usually prose.
- Could a human understand the rendered diagram and an agent understand the raw markdown?
