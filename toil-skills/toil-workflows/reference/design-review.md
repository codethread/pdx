# Design-led workflow

```text
triage(plan) -> Greed design(HITL) -> Toil triage(plan) -> War execute[N](build) -> fan-in -> Toil assess -> Greed review? -> release checkpoint -> complete
```

Use when the work needs explicit planning, tradeoff discussion, architecture/product choice, or pre-execution user sign-off. Review may be added after execution, but design is the reason to choose this reference.

## Shape

1. Toil enqueues `design` for Greed when the approach needs HITL planning.
2. Greed records the signed-off design outcome as a concise artifact with durable references.
3. Toil enqueues or resumes a follow-up triage after design; Toil, not Greed, turns the design into graph shape and worktree handoff.
4. Toil allocates/registers worktree scopes as needed and enqueues one or more War `execute` tasks with the signed-off design context.
5. Toil creates the repo-scope release checkpoint early, gated on the full downstream branch/root so it is not claimable until execution/review drains.
6. If multiple execute tasks fan out, gate/fan them in before assessment, review, or release.
7. Toil assesses execution evidence before release: validation, changed files, workspace cleanliness, and design fit.
8. Enqueue `review` for Greed only when requested, high-risk, or when acceptance/sign-off after execution is required.
9. If assessment or review fails, route rework/redesign before release.
10. The release checkpoint performs merge/MR/handoff routing and cleanup only after assessment/review evidence is acceptable.

## Task body scaffold

Design task:

- upstream triage task id
- problem statement
- constraints/non-goals
- decisions needed
- expected output artifact or brief
- clarify artifact ids when present

Post-design Toil triage:

- upstream design task id and artifact ids
- signed-off approach reference
- open constraints or non-goals
- suggested implementation slices, if known

Execute task:

- upstream design/triage task ids
- approved approach summary or artifact reference
- target repo/worktree scope and path
- branch name and base branch when relevant
- validation expectations

Review task:

- upstream execute task ids and artifact ids
- review focus
- evidence to inspect
- acceptance question

Assess task:

- upstream execute task ids and artifact ids
- signed-off design reference
- validation evidence and changed files to inspect
- design fit and workspace cleanliness checks
- failure classes to use for rework routing
- readiness decision: release directly, enqueue review, or route rework

Release checkpoint:

- upstream assessment task/artifact ids
- branch/root gate target
- accepted review outcome artifact id if review was required
- intended release path: merge, MR/PR, handoff, or no-release cleanup

## Constraints

- Do not assume design approval from silence.
- Do not let Greed silently take over graph routing after sign-off; route back through Toil triage.
- Do not make review automatic for every execution task.
- Keep gates and dependencies explicit when multiple execution tasks fan out.
- Do not skip assessment; release only after evidence is acceptable.
- Do not skip the release checkpoint; it is Toil's final routing point.
