# Git worktree contract

This is the local, repo-committed summary Toil should use when routing work through git worktrees. It is derived from the broader dotfiles `wktree` contract, but this file is the stable reference for Pandora's Box policy.

## Ownership

- `wktree` is the source of truth for git/worktree lifecycle transitions.
- Pithos is the source of truth for task graph and scope ids.
- Tmux is only a navigation/runtime surface; do not treat tmux state as proof of worktree existence, cleanliness, or ownership.

## Naming conventions

- Non-pooled worktree path: `<canonicalRoot>__<branch with / encoded as -->`.
- Pooled slot path: `<canonicalRoot>__featN`.
- Pooled placeholder branch: `wk-pool/featN`.
- Tmux session identity, when needed, is derived from the worktree path: `basename(worktree_path).replaceAll(".", "_")`.

Do not invent alternate path or tmux names.

## Core commands

Prefer JSON where a command returns structured state.

- `wktree root --cwd <path>` — print canonical/root worktree.
- `wktree list --cwd <path> --json` — list worktrees and pool/session metadata.
- `wktree path --cwd <path> --branch <branch>` — print expected worktree path for a branch.
- `wktree add --cwd <path> --branch <branch> --json [--base <branch>] [--slot <path>]` — create or allocate a worktree.
- `wktree remove --cwd <path> (--branch <branch> | --self <path>) --json` — remove or recycle a worktree.
- `wktree status --cwd <path>` — inspect pool status.
- `wktree recycle --cwd <path> --slot <path> [--force]` — explicitly recycle a pooled slot.

## Machine-readable outcomes

When `--json` is available, branch on the payload `kind` instead of parsing stderr.

Important kinds:

- `ready` — worktree exists and is ready.
- `pool_full` — no free pooled slot; candidates require deliberate choice.
- `blocked` — operation refused because it would be unsafe without explicit user intent.

A ready add-like outcome should identify at least:

- `worktree_path`
- `branch`
- `created_new_branch`
- optional `session.name` / `session.path`

## Base branch and branch naming

- New branches should default to the remote default branch/trunk, not the current HEAD.
- Use an explicit base only when stacked work is intended.
- If the user did not provide a branch name, infer one from repo docs or history; otherwise derive a lowercase kebab branch with a short prefix such as `feat/`, `fix/`, or `chore/`.

## Pool safety

Pooled slots are bounded reusable environments for expensive repos.

- Pool exhaustion is a blocked/recoverable state, not permission to recycle automatically.
- Do not recycle dirty slots, branches without upstreams, or branches not merged to upstream without explicit safe evidence or user intent.
- Forced recycling may discard work; escalate rather than guessing.

## Pithos handoff sequence

1. Confirm the canonical repo root and base branch.
2. Fast-forward the base branch in the repo-root worktree when appropriate.
3. Use `wktree add --json` to create or allocate the worktree.
4. If the outcome is `pool_full` or `blocked`, escalate or ask for a deliberate choice.
5. Upsert the Pithos worktree scope for `worktree_path`.
6. Enqueue War `execute` in that worktree scope with branch, path, upstream task ids, and validation expectations.

## Removal/archive sequence

After release/merge lands and the worktree is clean:

1. Use `wktree remove --json` for the branch or current worktree.
2. Archive the corresponding Pithos worktree scope only after the filesystem lifecycle succeeds or the intended cleanup state is otherwise clear.
3. Escalate dirty, ambiguous, or blocked cleanup states.
