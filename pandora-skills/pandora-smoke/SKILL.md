---
name: pandora-smoke
description: test the system
disable-model-invocation: true
---

We recently updated the pithos system, please run a smoke test to ensure all agents are behaving as expected.

1. Create a new test repo at ~/hello-pandora (including `git init`)
2. Create the scope
3. Q triage to break down some arbitrary tasks that try to use the full capabilities of the pithos graph system, including escalations, repairs and the full range of Evils as you know them
4. Ensure there is an escalation gated on the full graph so you can inspect results and discuss with the user
5. Finally clear up any created repos/scopes
