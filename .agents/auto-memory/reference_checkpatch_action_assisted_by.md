---
name: reference_checkpatch_action_assisted_by
description: "Advisory checkpatch review" CI fails on Assisted-by trailers — false positive from the action's bundled old checkpatch.pl
metadata:
  type: reference
---

`.github/workflows/checkpatch.yml` runs `webispy/checkpatch-action@v9`, which ships its own older `checkpatch.pl`: it reports `WARNING: Non-standard signature: Assisted-by:` + `ERROR: Unrecognized email address: 'Claude:<model>'`. The tree's own `scripts/checkpatch.pl` (rpi-6.18.y) lists `Assisted-by:` as standard and only checks the `AGENT_NAME:MODEL_VERSION` format. The check is advisory, not required. Verify locally with the tree's script: `perl scripts/checkpatch.pl --strict <patch>`. Keep the trailer (jean's rule).
