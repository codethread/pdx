---
name: greed-review
description: >
  Greed review-mode workflow. Use when the held Pithos task capability is `review`,
  or when you need to inspect evidence, prepare a stakeholder walkthrough, capture
  acceptance/rejection, and route follow-up without doing implementation work.
---

# Greed review mode

## Variables

| Variable | Value | Notes |
| --- | --- | --- |
| MODE | `review` | Held task capability |
| HITL_AGENT | Pandora | Escalation target for review session |
| EXECUTION_OWNER | Toil / War | Rework routes through Toil/War; you do not silently rewrite the chain |

## Prerequisites

- You have claimed exactly one `review` task.
- You have inspected the graph and held task body.
- You know which upstream tasks, artifacts, diffs, command output, and smoke-test evidence are in scope.

## Knowledge

Review mode exists to help the stakeholder decide whether completed work or a proposed outcome is acceptable.

Prior to this task, other agents should have checked spec compliance. Your role is to walk through the changes with the stakeholder to ensure alignment, not to perform substantial implementation.

Acceptance is evidence for downstream release; it is not automatic permission to merge unless the surrounding workflow says so.

## Decisions

```text
review task -> inspect evidence -> enough context?
                                   |
                                   +-> no -> inspect more / ask for missing evidence
                                   |
                                   +-> yes -> prepare walkthrough

prepare walkthrough -> Pandora escalate -> user outcome
                                      |
                                      +-> accepted -> artifact -> complete
                                      |
                                      +-> rejected/changes -> artifact with routing -> complete
                                      |
                                      +-> blocked -> artifact/escalation state -> complete or fail if session failed
```

## Procedures

### Inspect evidence

1. Inspect the held review task and graph context.
2. Read named upstream tasks/artifacts and relevant local/global state.
3. Inspect diffs, command output, smoke-test evidence, and any explicitly requested focus areas.
4. Run only read/check commands needed to understand review state.
5. Do not perform substantial implementation.

### Prepare walkthrough

Prepare a focused stakeholder walkthrough covering:

- what changed or was decided
- evidence that supports the outcome
- known risks and limitations
- unresolved questions
- what the user must accept, reject, or choose

### Run HITL review session

1. Enqueue a global escalation for Pandora saying the review session is ready. Include session/run/scope/task/topic context per the bundled prompt.
2. Stay in the HITL session until the user reaches an outcome here or Pandora relays one.
3. If the user asks for clarification, answer from evidence where possible.
4. If evidence is missing, say what is missing rather than inventing confidence.

### Capture outcome

1. Record whether the work was accepted, rejected, needs rework, needs redesign, or is blocked.
2. If rejected or changes are requested, record the rejected outcome and follow-up routing.
3. Complete the review task unless the review session itself failed.
4. Do not silently rewrite the task chain.

## Constraints

- Do not perform substantial implementation during review.
- Do not infer approval from silence.
- Do not treat accepted review as automatic merge approval unless the surrounding release workflow explicitly says so.
- Do not copy long upstream briefs into the artifact.

## Validation

- The artifact states the review outcome and workflow state in 2–5 sentences.
- Durable references are included: task/artifact ids, repo paths, line ranges, commits, MRs/PRs, Jira tickets, specs, or command evidence as applicable.
- Any rejection or requested change has explicit follow-up routing.
- The held review task is completed or failed with a concise reason if the review session itself failed.
