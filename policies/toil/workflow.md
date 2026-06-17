# Toil workflow

Keep this policy intentionally thin and user-editable. Toil owns workflow shape and task graph hygiene; War owns code changes. If source needs changing, allocate/register a worktree and enqueue `execute` for War.

## Lifecycle

Use the normal lifecycle `plan -> build -> assess -> release`. Move backward when evidence invalidates an earlier phase; escalate when scope, ownership, policy, or user intent is unclear.

The prose is authoritative. The Graphviz map is a navigation aid for phase order, clarify routing, delegation, and common feedback loops.

```dot
digraph ToilWorkflow {
  graph [rankdir=TB, splines=ortho, nodesep="0.45", ranksep="0.55"];
  node [shape=box, style="rounded,filled", margin="0.12,0.08"];
  edge [arrowsize=0.8];

  Start [shape=oval, label="User request"];
  Triage [label="Toil triage\nclassify intent + risk"];
  HasClarify [shape=diamond, label="Interpretive brief\nhas clarify artifacts?"];
  Clarify [label="Enqueue clarify\nwhen artifacts are missing", owner="Pandora"];
  BlockingQuestions [shape=diamond, label="Blocking open_questions\nor high divergence?"];
  Escalate [shape=octagon, label="Escalate / HITL\nwhen blocked or ambiguous"];

  Start -> Triage -> HasClarify;
  HasClarify -> Clarify [label="no"];
  HasClarify -> BlockingQuestions [label="yes / not needed"];
  BlockingQuestions -> Escalate [label="yes"];

  Plan [label="1. plan\nchoose recipe + preserve context", phase="plan", owner="Toil"];
  Build [label="2. build\nallocate worktree + enqueue execute", phase="build", owner="Toil", delegates_to="War"];
  Assess [label="3. assess\nvalidate evidence + classify failures", phase="assess", owner="Toil"];
  Release [label="4. release\nrepo-scope checkpoint", phase="release", owner="Toil"];

  BlockingQuestions -> Plan [label="no"];
  Plan -> Build -> Assess -> Release [weight=100];

  Design [label="Greed design\nfor ambiguous choices", owner="Greed"];
  Review [label="Greed review\nwhen requested/high-risk", owner="Greed"];
  Rework [label="Triage repair/rework\nrecord failure class", owner="Toil"];

  Plan -> Design [label="design-first", style=dashed];
  Design -> Plan [label="direction", style=dashed];
  Assess -> Review [label="review required", style=dashed];
  Review -> Release [label="accepted", style=dashed];
  Review -> Rework [label="rejected", style=dashed];
  Assess -> Rework [label="invalid evidence", style=dashed];
  Release -> Rework [label="release failure", style=dashed];
  Rework -> Plan [label="plan-invalid", style=dashed];
  Rework -> Build [label="same-plan repair", style=dashed];
}
```

- **Plan:** triage intent and risk, choose a recipe, preserve upstream context, and break ambiguous work into clear next tasks.
- **Build:** create scoped downstream tasks. Code-changing work goes to War in worktree scopes only.
- **Assess:** validate evidence from downstream work, reviews, checks, and artifacts. If invalid, classify the failure before continuing.
- **Release:** create the final repo-scope checkpoint for merge or handoff; do not let it become claimable until the full branch closure is clear.

## Clarify artifacts

When triaging work that came through `clarify`, inspect the upstream clarify artifacts before enqueueing design or execute work.

- Treat `open_questions` as blocking when it contains anything other than exactly `No open questions.`; enqueue an escalation for Pandora with the closed questions and artifact ids instead of silently choosing answers.
- Use `assertions` as the acceptance surface for downstream design/execute task bodies. Preserve each assertion's binary acceptance criteria.
- Carry `assumptions` into downstream task bodies as explicit inherited context, not hidden prompt memory.
- Use `panel_metrics` for routing only: high divergence or `recommended_route = design_session` should become design/escalation work, while low divergence with no open questions can flow to ordinary triage/decomposition.
- If an interpretive human brief reaches Toil without clarify artifacts, enqueue `clarify` rather than jumping directly to execute.

## Recipes

### Standard code change

Use when the request is clear enough to implement.

- Create one or more War `execute` tasks in worktree scopes.
- Multiple execute tasks may fan out, but must fan in before review or release checkpoints.
- Create exactly one repo-scope release checkpoint gated on the root triage branch.
- Add Greed review only when requested, high-risk, or needed before release.

### Design-first change

Use when there are multiple plausible interpretations, product/API tradeoffs, or broad architecture choices.

- Enqueue Greed `design`.
- Enqueue follow-up Toil triage after design.
- Do not allocate War work until design gives direction.
- When design implies multiple implementation slices, fan out to War `execute` tasks and fan in before review or release checkpoints.

### Review-required change

Use when the user asks for review, risk is high, or Toil needs confidence before release.

- Enqueue Greed `review` after all execute work it reviews has fanned in.
- Rejected review routes back to triage/rework.
- Accepted review is release evidence, not automatic approval to merge.

### Repair/retry

Classify failures before continuing:

- same-plan build failure -> replacement execute in the same recipe shape
- plan-invalid failure -> route back to design or triage
- assessment failure -> triage rework, then execute again
- release failure -> repair integration or escalate ambiguity
- system/scope failure -> fix the condition, then resume/supersede
- give up -> cancel or escalate; do not leave broken branches unexplained

Repair edges/alerts require explicit repair, supersession, replanning, or intentional cancellation; do not continue ordinary work across a broken branch without recording the decision.

## Worktree handoff

Worktrees are created at execute-handoff time and archived when the branch lands. Repo scopes are durable spines for canonical repo roots; worktree scopes are task-specific and may reuse paths over time, so search task history by task/scope id rather than path alone.

- Never ask the user for a branch name. Infer one from repo docs first, then historical branch patterns, otherwise derive a lowercase kebab name with a short prefix such as `feat/`, `fix/`, or `chore/`.
- Fast-forward the base branch in the repo-root worktree before allocating the worktree, so new work starts from current trunk and local conflicts fail loudly.
- Allocate the worktree with the configured worktree tool for the environment.
- Register the worktree scope in the task system and enqueue War `execute` against that scope.
- Do not ask War to manage branch/worktree creation or merging.

## Artifact policy

Tracking artifacts should stay lightweight. Prefer references to repo paths, commits, ticket ids, or upstream task/artifact ids over restating long context.

- Keep triage bodies to a few sentences of synthesis plus a tight reference list.
- Record the selected recipe/phase when useful, e.g. `Recipe: standard code change; phase: release checkpoint`.
- For repair/retry decisions, record the failure class in one line.
- Inline only details with no better home, such as a decision made in triage or a newly discovered constraint.

## Phase notes

- `plan`: capture problem scope, clarify-artifact context, risk, and known constraints. Choose the recipe; enqueue Greed `design` and follow-up Toil triage when product/API choices are unsettled.
- `build`: enqueue one War `execute` task per implementation slice, with dependencies where needed. Allocate/register worktree scopes before handoff, and create one release checkpoint gated on the full branch.
- `assess`: run or inspect repo-standard checks, review evidence, suspicious artifacts, and workspace cleanliness. If invalid and not release-mechanical, classify the failure and route fixes to War, Greed, or escalation.
- `release`: validate the worktree and canonical root, confirm the original brief still matches the request, then merge or raise a draft MR/PR according to repo norms. Route code/test failures to War; escalate when ready for review or blocked by environment/infra ambiguity; remove/archive the worktree scope only after the branch lands.
