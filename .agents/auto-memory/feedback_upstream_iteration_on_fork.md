---
name: feedback_upstream_iteration_on_fork
description: once an upstream PR is open, iterate (review rounds, benches, fixups) on this fork's draft PR; upstream only gets the final single commit + a short summary
metadata:
  type: feedback
---

Iterate on a draft PR in this fork (e.g. jclaveau/rpi-linux#1, branch `bcm2835-codec-eos-wip`, base `rpi-6.18.y`); push to the upstream PR's branch only the final, squashed, checkpatch-clean commit, and say it in one or two sentences.

**Why:** 2026-11-02 on raspberrypi/linux#7733 a long self-review comment + a fixup commit drew pelwell's "I, for one, would appreciate not being part of the debug loop while you and your AI work knock the PR into shape. Try and get your ducks in a row, and a nice concise explanation (the first post in this thread was good) before submitting."

**How to apply:**
- Any push to the upstream PR's head branch shows on the upstream PR: use a different branch for WIP.
- A commit object can be built with the GitHub git-data API (`POST /git/commits` with tree + parent) without putting it on a branch, so nothing is announced until the final push.
- Kernel style: one commit per PR, message lines ≤ 75 chars, oops excerpts without the `virtual address 0000…` line (checkpatch reads it as a sha).
- No self-review comments upstream; answer maintainers' questions only.
