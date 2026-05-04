# mobile-sam

Compress the Segment Anything Model (SAM) until **it runs on mobile and in the browser**, and package it as an SDK that tracks target objects across video frames.

---

## 🎯 Goal: Acquisition by Google

This project targets a **Jetpac-style acqui-hire by Google** (2–3 person team / technical core + demo).

**The acquisition target is the tiny model + the cross-platform SDK.** Equal accuracy and speed across browser / iOS / Android is the value.

| Likely buyer | Integration scenario |
|---|---|
| **Google Lens** | Tap an object in the camera view → get a precise mask of the entire object. |
| **Google Photos** | The video equivalent of "Magic Eraser" — track and erase moving objects. |
| **YouTube Create** | Subject extraction / background replacement for Shorts editing. |
| **Pixel Camera** | Real-time subject highlighting while recording video. |

### Technical moat we are paid for
1. **Model size < 30 MB** (further distilled from FastSAM / MobileSAM)
2. **Real-time on both iPhone and Pixel** (30 FPS @ 720p)
3. **Tracking stability** (mask IDs stay consistent across frames)
4. **A WebGPU build that runs in the browser** (PoC anyone can try in a browser)

---

## Tech stack

| Layer | Tech |
|---|---|
| Base model | Distilled from MobileSAM / EfficientSAM / FastSAM (whichever wins) |
| Quantization | INT8 / FP16, with mobile-friendly substitutions for ops like LayerNorm |
| Inference | iOS: CoreML; Android: TFLite (GPU/NNAPI delegate); Web: ONNX Runtime + WebGPU |
| Tracking | Lightweight Kalman + IoU-based ID matching (a stripped-down ByteTrack) |
| Samples | Three demos: iOS / Android / Web |

---

## Repository layout

```
mobile-sam/
├── model/                 # Distillation / quantization scripts
├── sdk/
│   ├── ios/              # CoreML + Swift package
│   └── android/          # TFLite + Kotlin AAR
├── examples/             # Per-platform demos
└── docs/                 # Benchmark results / acquisition pitch
```

---

## Three-phase plan

### Phase 1 (≤ 2 months): Model compression
- [ ] Reproduce the MobileSAM distillation pipeline
- [ ] Convert to CoreML / TFLite, validate on iPhone 15 / Pixel 8
- [ ] Hit 30 FPS @ 720p latency

### Phase 2 (≤ 4 months): SDK + tracking
- [ ] Publish the iOS Swift Package
- [ ] Publish the Android AAR
- [ ] Inter-frame tracking with the stripped-down ByteTrack

### Phase 3 (≤ 6 months): Visibility
- [ ] Ship a WebGPU demo anyone can try in a browser
- [ ] Submit a report to arXiv
- [ ] Send demos to the Google Lens / Photos teams

---

## Development commands

```bash
# Distillation / conversion
cd model && python distill.py --teacher sam_vit_b --student mobile

# iOS SDK
cd sdk/ios && swift build

# Android SDK
cd sdk/android && ./gradlew assembleRelease
```

(Implementations land when Phase 1 starts.)
