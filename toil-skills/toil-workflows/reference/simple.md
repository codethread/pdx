# Simple workflow

```text
triage(plan) -> War execute(build) -> fan-in -> Toil assess -> release checkpoint -> complete
```

Use for small, clear implementation work that does not need HITL design or review.

## Shape

1. Toil inspects the triage task and graph context.
2. Toil chooses the narrowest appropriate repo/worktree scope.
3. If source changes are needed, Toil allocates/registers the worktree scope before handoff.
4. Toil enqueues one War `execute` task, or the smallest useful fan-out of War `execute` tasks.
5. Toil creates the repo-scope release checkpoint early, gated on the execution branch/root so it is not claimable until downstream work drains.
6. War completes or fails with changed files and validation evidence.
7. Toil assesses evidence before release: validation, changed files, workspace cleanliness, and whether the original brief still matches the outcome.
8. If assessment fails, classify the failure and route rework before release.
9. The release checkpoint performs merge/MR/handoff routing and cleanup only after assessment evidence is acceptable.

## Task body scaffold

Execute task:

- upstream triage task id
- target repo/worktree scope and path
- branch name and base branch when relevant
- requested change
- relevant files or commands if known
- expected validation

Assess task:

- upstream execute task ids and artifact ids
- validation evidence to inspect
- changed files, workspace cleanliness, and brief-fit checks
- failure classes to use for rework routing
- release readiness decision

Release checkpoint:

- upstream assessment task/artifact ids
- branch/root gate target
- intended release path: merge, MR/PR, handoff, or no-release cleanup
- worktree path/scope to clean up after landing

## Constraints

- Avoid design/review tasks unless the user asked for them or risk requires them.
- Keep execution scope narrow and concrete.
- Do not skip assessment; release only after evidence is acceptable.
- Do not skip the release checkpoint; it is Toil's final routing point.
