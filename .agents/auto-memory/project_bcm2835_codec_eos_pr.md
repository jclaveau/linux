---
name: project_bcm2835_codec_eos_pr
description: bcm2835-codec oops when V4L2 STOP comes before STREAMON (zero-frame encode); fix = hold the EOS until the input port is enabled; upstream raspberrypi/linux#7733 state + pending follow-up work
metadata:
  type: project
---

**Bug:** `vchiq_mmal_port_enable+0xd4` NULL-pointer oops from `bcm2835_codec_start_streaming`, then every codec use fails with `Mutex fail` until reboot. Upstream issue raspberrypi/linux#4920 (open since 2022, many reporters, no repro before ours).

**Repro:** `ffmpeg -f lavfi -i testsrc=size=1280x720:rate=24 -t 0 -c:v h264_v4l2m2m -b:v 2M -f null -` (any h264_v4l2m2m encode that hits EOF before its first frame; Jellyfin hits it on the last HLS segment when audio outlasts video).

**Cause:** ffmpeg sends `V4L2_ENC_CMD_STOP` before `VIDIOC_STREAMON(OUTPUT)` → `vidioc_encoder_cmd()` submits `q_data->eos_buffer` to the still-disabled MMAL input port → queued on `port->buffers` → `start_streaming()` memsets it while linked → `port_enable()` `list_del()` on NULL pointers.

**Fix (`drivers/staging/vc04_services/bcm2835-codec/bcm2835-v4l2-codec.c`):** `bcm2835_codec_submit_eos()` holds the EOS (`eos_buffer_pending`) while the input port is disabled; `start_streaming()` submits it after `port_enable()` (failure only logged, so STREAMON never fails with the port enabled); a START cancels a held EOS. Decoder cmd shares the path. Tested on Pi 4, 6.12.93+rpt-rpi-v8 (out-of-tree DKMS build), never booted on 6.18.

**Spec deviation (kept on purpose):** dev-encoder.rst says STOP on a non-streaming queue must not start a drain. Strict variant (drop the EOS) hangs jellyfin-ffmpeg 7.1 on a zero-frame encode 3/3; `v4l2_m2m_encoder_cmd()` makes the same deviation.

**Upstream PR raspberrypi/linux#7733:** against `rpi-6.18.y`, head = this fork's `bcm2835-codec-eos-before-streamon`, ONE commit `a17df8146ba7` (squashed 2026-11-02). Maintainer pelwell pushed back on review noise — see [[feedback_upstream_iteration_on_fork]]. Never merge, never post there without jean's go.

**Pending follow-up (separate PR, not #7733):**
- Normal drain loses frames on stock driver too: STREAMON → QBUF×3 → STOP yields 0–1 of 3 (EOS submitted while frames still wait in the m2m queue).
- Frames queued before STREAMON then STOP are dropped by the held EOS (variant "B" — send the EOS from `device_run()` once no source buffer is ready — fixes it, 3/3).
**Impact measured 2026-11-02: LOW.** ffmpeg 5.1 and jellyfin-ffmpeg 7.1 h264_v4l2m2m encodes of 1/2/3/4/5/8/48/240 frames (×3 each) kept ALL frames (ffprobe -count_frames). ffmpeg feeds frames lock-step, so none are left in the m2m queue at STOP; the loss needs a raw V4L2 client that queues a burst and STOPs at once. Park unless someone reports it.
Bench tooling (drain-test.c V4L2 client, variant bench script) lives in jean's nas repo `bcm2835-codec-eosfix/`.

Related: [[reference_checkpatch_action_assisted_by]], [[reference_fork_ci]]
