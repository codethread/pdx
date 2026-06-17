# Review-required workflow

```text
triage(plan) -> War execute[N](build) -> fan-in -> Toil assess -> Greed review(HITL) -> Toil assess/rework -> release checkpoint -> complete
```

Use when the implementation path is clear enough to build, but the user requested review, risk is high, or Toil needs HITL confidence before release. This workflow does not require Greed design unless the approach itself is uncertain.

## Shape

1. Toil inspects the triage task and graph context.
2. Toil allocates/registers worktree scopes as needed and enqueues one or more War `execute` tasks.
3. Toil creates the repo-scope release checkpoint early, gated on the full downstream branch/root so it is not claimable until execution/review drains.
4. If multiple execute tasks fan out, gate/fan them in before assessment and review.
5. Toil assesses execution evidence before review: validation, changed files, workspace cleanliness, and whether review has enough evidence.
6. Toil enqueues Greed `review` with explicit focus, upstream task ids, artifact ids, diffs/commits, and validation evidence.
7. Greed records accepted/rejected/blocked outcome as a concise artifact.
8. Toil reassesses the review outcome:
   - accepted -> release checkpoint may proceed
   - rejected or changes requested -> classify and route rework before release
   - blocked -> escalate or gather missing evidence
9. The release checkpoint performs merge/MR/handoff routing and cleanup only after assessment/review evidence is acceptable.

## Task body scaffold

Execute task:

- upstream triage task id
- target repo/worktree scope and path
- branch name and base branch when relevant
- requested change
- validation expectations
- note that review is expected after fan-in

Review task:

- upstream execute task ids and artifact ids
- review focus and acceptance question
- evidence to inspect: paths, commits/diffs, validation transcripts, screenshots, MRs/PRs, or specs
- risks or user concerns that motivated review

Pre-review assess task:

- upstream execute task ids and artifact ids
- validation evidence and changed files to inspect
- workspace cleanliness and brief-fit checks
- whether review has enough evidence to proceed
- failure classes to use for rework routing before review

Post-review assess/rework task:

- upstream review task id and outcome artifact id
- accepted/rejected/blocked review state
- rework, redesign, escalation, or release-readiness decision
- failure class when review rejected or blocked release

Release checkpoint:

- upstream post-review assessment task/artifact ids
- branch/root gate target
- accepted review outcome artifact id
- intended release path: merge, MR/PR, handoff, or no-release cleanup

## Constraints

- Do not insert Greed design unless the approach/product/API choice is uncertain.
- Do not treat accepted review as automatic merge approval unless the surrounding release workflow says so.
- Do not release on rejected/blocked review; route rework, redesign, or escalation first.
- Do not skip assessment; release only after evidence is acceptable.
- Do not skip the release checkpoint; it is Toil's final routing point.
