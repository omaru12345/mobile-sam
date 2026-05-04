# mobile-sam

Segment Anything Model (SAM) を **モバイル / ブラウザで動くまで圧縮** し、
動画フレーム間で対象オブジェクトを追跡できる SDK としてパッケージング。

---

## 🎯 Goal: Googleに買収されること

このプロジェクトは「Google による Jetpac 型 acqui-hire（2〜3人 / 技術コア + デモ）」をエグジット目標とする。

**買収対象は軽量モデル + マルチプラットフォーム SDK。** ブラウザ / iOS / Android で同じ精度・速度が出ることが価値。

| 想定買収先 | 統合シナリオ |
|---|---|
| **Google Lens** | スマホカメラで指定オブジェクトをタップ → 物体ごと精密マスク |
| **Google Photos** | 動画編集 "Magic Eraser" の動的版（動く物体の追跡消去）|
| **YouTube Create** | Shorts 編集での被写体抜き出し / 背景差し替え |
| **Pixel Camera** | 動画撮影中のリアルタイム被写体強調 |

### 評価される技術の堀（moat）
1. **モデルサイズ < 30MB**（FastSAM / MobileSAM をさらに蒸留）
2. **iPhone / Pixel どちらでもリアルタイム**（30 FPS @ 720p）
3. **追跡の安定性**（フレーム間でマスク ID がブレない）
4. **WebGPU 版でブラウザでも動く**（PoC をブラウザで配れる）

---

## 技術スタック

| レイヤー | 技術 |
|---|---|
| ベースモデル | MobileSAM / EfficientSAM / FastSAM のいずれかを蒸留 |
| 量子化 | INT8 / FP16、LayerNorm 等の演算をモバイル向けに置換 |
| 推論 | iOS: CoreML、Android: TFLite（GPU/NNAPI delegate）、Web: ONNX Runtime + WebGPU |
| トラッキング | 軽量 Kalman + IoU ベースの ID マッチング（ByteTrack 簡易版） |
| サンプル | iOS / Android / Web の 3 つのデモ |

---

## ディレクトリ構成

```
mobile-sam/
├── model/                 # 蒸留 / 量子化スクリプト
├── sdk/
│   ├── ios/              # CoreML + Swift パッケージ
│   └── android/          # TFLite + Kotlin AAR
├── examples/             # 各プラットフォームのデモ
└── docs/                 # ベンチ結果 / 買収ピッチ
```

---

## 3 フェーズ計画

### Phase 1（〜2か月）: モデル圧縮
- [ ] MobileSAM の蒸留パイプラインを再現
- [ ] CoreML / TFLite に変換、iPhone 15 / Pixel 8 で動作確認
- [ ] 30 FPS @ 720p のレイテンシを達成

### Phase 2（〜4か月）: SDK 化 + トラッキング
- [ ] iOS Swift パッケージ公開
- [ ] Android AAR 公開
- [ ] ByteTrack 簡易版でフレーム間追跡

### Phase 3（〜6か月）: 露出
- [ ] WebGPU 版デモを公開（誰でもブラウザで触れる）
- [ ] arXiv にレポート
- [ ] Google Lens / Photos チームへデモ送付

---

## 開発コマンド

```bash
# モデル蒸留・変換
cd model && python distill.py --teacher sam_vit_b --student mobile

# iOS SDK
cd sdk/ios && swift build

# Android SDK
cd sdk/android && ./gradlew assembleRelease
```

（実装は Phase 1 着手時に追加）
