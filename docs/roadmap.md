# Development Roadmap — Phase 1 to Phase 3

This document is the 12-week roadmap to the project goal (SDK complete + acquisition-grade demo published), with evaluation criteria and risk management.

## 1. Schedule and task breakdown (week by week)

### Phase 1: Model preparation and core pipeline (Weeks 1–4)
* **W1: Pick teacher / student models, lock the architecture**
  * Evaluate MobileSAM / EfficientSAM weights as the baseline.
  * **DoD:** Working PyTorch inference script (single image).
* **W2: Tracking (a stripped-down ByteTrack)**
  * Implement Kalman-filter logic that predicts the next BBox from the current BBox.
  * **DoD:** Given a manually drawn first BBox, the tracking script follows the object to the final frame on a video.
* **W3: SAM + Tracking hybrid pipeline**
  * Image Encoder on keyframes; tracker-predicted BBox + Mask Decoder for the intermediate frames.
  * **DoD:** A complete video pipeline runs in PyTorch. Baseline FPS and IoU are measurable.
* **W4: PyTorch → ONNX export with a static graph**
  * Strip dynamic tensors; produce ONNX models exportable to each target platform.
  * **DoD:** ONNX Runtime (CPU) matches PyTorch outputs within 1e-4.

### Phase 2: Edge optimization and platform integration (Weeks 5–8)
* **W5: iOS / CoreML**
  * FP16 conversion via `coremltools`, ANE optimization, AVFoundation camera input in Swift.
  * **DoD:** Camera input on a real iPhone 13 renders segmentation at ≥ 15 FPS.
* **W6: Android / TFLite**
  * PTQ INT8 quantization. CameraX integration. GPU Delegate / XNNPACK benchmarks.
  * **DoD:** Camera input on a real Pixel 6 runs at ≥ 15 FPS.
* **W7: Web / ONNX Runtime + WebGPU**
  * Browser inference setup. WASM fallback.
  * **DoD:** Chrome (WebGPU enabled) on M1 Mac runs at ≥ 30 FPS.
* **W8: Memory / latency / thermal optimization**
  * Tune Image Encoder cadence (every N-th frame).
  * **DoD:** A 5-minute continuous run does not force-quit due to overheating, nor halve the FPS (< 10 FPS).

### Phase 3: SDK packaging, evaluation, and demo launch (Weeks 9–12)
* **W9: SDK packaging**
  * iOS: Swift Package Manager (SPM). Android: AAR. Web: npm.
  * **DoD:** Each package can be imported into a fresh project and used in ≤ 10 lines.
* **W10: Quantitative evaluation on the public benchmarks**
  * (See the evaluation protocol below.)
  * **DoD:** All numbers (IoU, FPS, etc.) hit the targets — or trade-offs are documented.
* **W11: Demo apps + landing page + arXiv paper**
  * Build a TestFlight / web demo users can run on their own devices.
  * **DoD:** Paper draft complete; web demo public.
* **W12: Marketing and buzz**
  * Posts on X, Hacker News, Reddit (r/MachineLearning).
  * **DoD:** Inbound contact from Google folks, or 1,000+ GitHub stars.

## 2. Evaluation protocol
* **Datasets:**
  1. **DAVIS 2017 & YouTube-VOS 2018:** Standard video segmentation metrics.
  2. **In-house mobile dataset:** 50 clips of 10–15 s each shot on phones, including hand-shake, motion blur, backlighting, and occlusion.
* **Metrics:**
  * **J (Region Similarity):** IoU (Intersection over Union).
  * **F (Boundary Accuracy):** Boundary precision.
  * **J&F Mean:** Average of the two. Target: ≥ 80%.
  * **End-to-end latency:** Milliseconds from camera frame capture to screen rendering.

## 3. Risks and mitigations

| Risk | Severity | Mitigation |
|---|---|---|
| **License uncertainty** | High | Meta SAM is Apache 2.0 but the SA-1B training dataset is non-commercial. Verify the license of every weight we ship. If problematic, switch to a BYOM (Bring Your Own Model) shape — SDK code Apache 2.0, weights downloaded by the user — to firewall legal risk. |
| **Android GPU performance variance** | High | TFLite GPU Delegate behaves differently on Mali vs. Adreno. Add cloud test devices (e.g. Firebase Test Lab) to CI; profile GPU support at startup; auto-fall back to XNNPACK (CPU) on incompatible devices. |
| **Battery thermal throttling** | Med | Continuous run causes the SoC to downclock. Add a "Battery Saver Mode" in the SDK that dynamically increases the keyframe interval (e.g. skip Image Encoder when motion is small). |
| **INT8 accuracy degradation** | Med | INT8-quantizing the Mask Decoder sometimes destroys the output (noisy masks). Apply hybrid quantization: keep light layers like the Mask Decoder in FP16 / FP32 and quantize only the compute-heavy Image Encoder. |
