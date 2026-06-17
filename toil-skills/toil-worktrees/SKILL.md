---
name: toil-worktrees
description: >
  Toil git-flow and worktree usage guide. Use when Toil needs to plan or route
  project work through git worktrees, choose repo/worktree scopes, or explain
  how to use the local wktree workflow.
---

# toil-worktrees

## Variables

| Variable | Value | Notes |
| --- | --- | --- |
| CONTRACT | reference/git-worktrees.md | Repo-local worktree lifecycle contract |
| ENGINE | wktree | Source of truth for worktree lifecycle |

## Prerequisites

- Read CONTRACT before giving detailed CLI guidance.
- Work is tied to a git repository or planned worktree scope.

## Knowledge

`wktree` owns git/worktree lifecycle transitions. Pithos owns task graph and scope ids. Tmux is a navigation/runtime surface, not durable truth.

Important conventions from CONTRACT:

- Non-pooled path: `<canonicalRoot>__<branch encoded with / as -->`.
- Pooled slot path: `<canonicalRoot>__featN`.
- Pooled placeholder branch: `wk-pool/featN`.
- Tmux session name, when needed: `basename(worktree_path).replaceAll(".", "_")`.
- New branches default to origin's default branch/trunk unless an explicit base is chosen.
- Machine consumers should prefer `--json` and branch on outcome `kind`.

Core commands:

- `wktree root --cwd <path>`
- `wktree list --cwd <path> --json`
- `wktree path --cwd <path> --branch <branch>`
- `wktree add --cwd <path> --branch <branch> --json [--base <branch>] [--slot <path>]`
- `wktree remove --cwd <path> (--branch <branch> | --self <path>) --json`
- `wktree status --cwd <path>`
- `wktree recycle --cwd <path> --slot <path> [--force]`

## Procedures

1. Read CONTRACT for exact command semantics, naming, pool safety, and handoff sequencing.
2. Decide whether the task should run in the current repo scope or a new/existing worktree scope.
3. Use `wktree` to create, list, or locate the target worktree.
4. Treat `pool_full` or `blocked` JSON outcomes as deliberate-choice states; escalate rather than guessing.
5. Upsert the Pithos repo/worktree scope for the selected path before enqueueing execution.
6. Put branch, worktree path, upstream task ids, and validation expectations in downstream task bodies.

## Constraints

- Do not invent tmux session names; derive them from worktree path.
- Do not treat tmux state as proof of worktree existence or cleanliness.
- Do not recycle pooled slots without explicit safe evidence or user intent.
- Do not silently stack branches; choose base explicitly when stacking is intended.
- Do not bypass `wktree` with ad hoc `git worktree` lifecycle commands unless `wktree` is unavailable and you escalate or record why.

## Validation

- [ ] CONTRACT was read before detailed worktree guidance.
- [ ] Downstream work names the repo/worktree path and branch.
- [ ] Worktree lifecycle decisions use `wktree`, not ad hoc git commands.
- [ ] Unsafe pool/recycle states are escalated or blocked.
