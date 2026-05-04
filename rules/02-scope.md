# 02 — Scope

A small team can only win by being narrower than the competition. This file defines
the hard scope. Anything outside it must be rejected, even if technically interesting.

## DO focus on

- Cross-platform parity: iOS / Android / Web all hit the same IoU within ±2%
- Real-time on commodity hardware (≥ 30 FPS on iPhone 13, ≥ 25 FPS on Pixel 6)
- Tiny model size (≤ 15 MB total)
- Stable inter-frame tracking (no mask flicker, stable IDs)
- A trivially adoptable SDK (≤ 10 lines to integrate per platform)

## DO NOT build

- **A GPU training cluster.** Distillation runs on rented spot GPUs; do not over-invest
  in MLOps infrastructure.
- **Static-image segmentation tooling.** Plenty exists already (SAM, MobileSAM, FastSAM).
  Our differentiation is **video + edge real-time**.
- **A consumer camera app.** This is an SDK. Sample apps stay minimal — they exist only
  to demonstrate the SDK.
- **Your own foundation segmentation model.** Distill from MobileSAM / EfficientSAM /
  FastSAM. Verify license compatibility before each new teacher choice.
- **3D / NeRF / Gaussian Splatting features.** Out of scope. Keep the surface area small.

## Why this matters

Every "DO NOT" item above has been considered and rejected because it would either
(a) duplicate an existing OSS project, (b) dilute the technical moat (video + edge),
or (c) push the team toward operational work that a 1–5 person crew cannot sustain.

If you believe a "DO NOT" item should be reopened, escalate to the human owner —
do not start implementing it.
