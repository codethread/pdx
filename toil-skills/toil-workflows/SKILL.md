---
name: toil-workflows
description: >
  Toil workflow graph-shape index. Use when Toil needs to choose or explain the
  Pithos task graph shape for simple work, design-led work, or review-required
  chains.
---

# toil-workflows

## Variables

| Variable | Value | Notes |
| --- | --- | --- |
| SIMPLE | reference/simple.md | Simple execute + assess + release-checkpoint scaffold |
| DESIGN_REVIEW | reference/design-review.md | Design-led execute/review + assess + release-checkpoint scaffold |
| REVIEW_REQUIRED | reference/review-required.md | Clear-build workflow with required HITL review before release |

## Knowledge

This skill is intentionally an index. Read the reference file matching the task shape before enqueueing.

Available references:

- `reference/simple.md` — straightforward triage to execution with Toil assessment and release checkpoint.
- `reference/design-review.md` — design-led workflow with post-design Toil triage, optional HITL review, assessment, and release checkpoint.
- `reference/review-required.md` — implementation path is clear, but review is required before release.

## Procedures

1. Classify the task shape:
   - simple implementation → read SIMPLE.
   - approach/product/API choice is uncertain → read DESIGN_REVIEW.
   - implementation path is clear but user/risk requires review → read REVIEW_REQUIRED.
2. Use the selected reference as the graph-shape scaffold; preserve assessment, the release checkpoint, and any required post-design Toil triage.
3. Keep task bodies concise and name upstream task ids, scopes, branches, and existing artifacts when relevant.

## Constraints

- Do not create review tasks by default; only when requested or when the selected workflow requires HITL acceptance.
- Do not create design tasks just because review is required; design is for uncertain approach/product/API choices.
- Do not fan out more tasks than needed to move the chain forward.
- Do not skip assessment before release.
- Do not skip the repo-scope release checkpoint; it is Toil's final routing point.
- Do not duplicate reference details in this index; put details under `reference/`.

## Validation

- [ ] Selected graph shape matches task complexity.
- [ ] Enqueued tasks have clear capabilities and scopes.
- [ ] Assessment and release checkpoint are visible.
- [ ] Any review/design gate is intentional and visible.
