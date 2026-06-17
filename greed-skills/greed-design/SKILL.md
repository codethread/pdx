---
name: greed-design
description: >
  Greed design-mode workflow. Use when the held Pithos task capability is `design`,
  or when you need to develop a plan, resolve uncertainty with the user, perform
  external research before HITL discussion, and produce a signed-off design brief/spec.
---

# Greed design mode

## Prerequisites

- You have claimed exactly one `design` task.
- You have inspected the graph and held task body.
- You understand relevant upstream artifacts, gates, and requested outcome.

## Knowledge

Design mode exists to turn uncertainty into an agreed plan before execution.

Your job is to understand the problem, explore viable options, resolve verifiable unknowns, and guide the user through remaining choices. Do not enqueue execute tasks directly; downstream execution routes through `triage`.

External research is required when the design depends on behaviour that cannot be verified from the repo alone.

Research whenever the design involves:

- a third-party library, SDK, or framework feature, especially one the codebase does not currently use or a version you have not recently worked with
- an external service API: REST, GraphQL, gRPC, webhooks, or similar
- a CLI or tool whose flags, subcommands, or semantics are central to the design
- a standard, protocol, or platform behaviour you are not certain about
- recent changes, deprecations, or known issues that may not be reflected in model knowledge

## Decisions

```text
design task -> inspect -> explore repo/domain -> external unknowns?
                                                |
                                                +-> yes -> research authoritative sources
                                                |
                                                +-> no  -> prepare HITL brief

prepare HITL brief -> Pandora escalate -> grill user -> draft spec -> sign-off?
                                                                  |
                                                                  +-> no -> continue grilling/revise
                                                                  |
                                                                  +-> yes -> artifact -> complete
```

## Procedures

### Explore before discussion

1. Consider the problem space but do not over-solutionize in the initial sweep.
2. Read relevant repo/domain material thoroughly and inspect prior artifacts.
3. Use subagents liberally when exploration can be parallelised.
4. If a question can be answered by exploring the codebase, explore the codebase instead of asking the user.

### Research external unknowns

1. Check the manifest, lockfile, generated code, or local tool version first so research is pinned to the version actually in use.
2. Prefer authoritative sources: official docs, source code, changelogs, or local `--help` output.
3. Frame each query as a focused question, such as "what are the options for X in version Y?" or "how does Z behave when ...?"
4. Parallelise independent research threads when useful, and keep exploring the repo while they run.
5. Read the research results before escalating to Pandora.
6. If a finding shifts a decision, surface the tradeoff explicitly in the user discussion.

### Run HITL design discussion

1. Notify Pandora when ready for the user to join. Include session/run/scope/task/topic context per the bundled prompt.
2. Align with the user on the brief: "what is the problem we are solving?"
3. Interview the user relentlessly until there is shared understanding.
4. Walk down each branch of the design tree, resolving dependencies between decisions one by one.
5. Ask questions one at a time.
6. For each question, provide your recommended answer so the user can quickly say yes.

### Capture the design

1. Draft the result in a spec or design brief according to repo guidance. Use relevant Skills when available.
2. Write to a tmp file first and let the user review artifacts before finalising when appropriate.
3. Require affirmative wording such as "the plan looks good" or "go ahead with the design brief" before treating the design as signed off.
4. Record one concise artifact once the session has concluded.
5. Complete the held design task.

## Constraints

- Do not escalate with open unknowns that external sources could have resolved.
- Do not enqueue execute tasks directly unless explicitly instructed otherwise by the bundled prompt/control plane.
- Do not treat silence or vague acknowledgement as sign-off.
- Do not copy the upstream brief into the final artifact.

## Validation

- The user has explicitly signed off or requested a concrete follow-up path.
- The final artifact records the decision/outcome/risk/next step in 2–5 sentences.
- Durable references are included: task/artifact ids, repo paths, line ranges, commits, MRs/PRs, Jira tickets, or specs as applicable.
- Workflow state is clear: ready for Toil, needs user choice, needs redesign, or blocked.
