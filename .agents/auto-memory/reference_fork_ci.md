---
name: reference_fork_ci
description: how CI runs on this fork (jclaveau/rpi-linux, renamed from jclaveau/linux 2026-11-02) and on upstream PRs from it
metadata:
  type: reference
---

- Fork Actions only run after a one-time "enable workflows" click in the fork's Actions tab (no API for it); done 2026-11-02.
- Triggers (from `rpi-*` branches' `.github/workflows/`): kernel-build, dtoverlaycheck, kunit on PRs/pushes to `rpi-*`; checkpatch on any PR. Pushes to `main-jclaveau` (this memory branch) trigger nothing.
- Upstream PRs from a fork wait at `action_required` until a maintainer approves the run.
- Kernel build matrix = 7 defconfigs (bcm2835, bcmrpi, bcm2709, bcm2711, bcm2711_rt, bcm2712, arm64 defconfig), ~1 h.
- Default branch is `main-jclaveau` (memories only); upstream-tracking branches stay `rpi-*`.
