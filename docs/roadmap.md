# Development Roadmap: Phase 1 to Phase 3

本ドキュメントは、プロジェクトのゴール（SDK完成およびAcqui-hireに向けたデモ公開）までの12週間の開発ロードマップと、評価基準・リスク管理を定義する。

## 1. スケジュールとタスク分解 (週単位)

### Phase 1: モデル準備とコアパイプライン構築 (Week 1-4)
* **W1: 教師モデル・生徒モデルの選定とアーキテクチャ確定**
  * MobileSAM / EfficientSAMの重みを評価し、ベースラインとして組み込む。
  * **DoD:** PyTorch環境での推論スクリプト完成（静止画ベース）。
* **W2: トラッキング（ByteTrack簡易版）の実装**
  * BBoxから次のBBoxを予測するKalman Filterロジックの実装。
  * **DoD:** 動画データにおいて、初回のBBoxを手動入力し、最終フレームまでBBoxが追従するスクリプト完成。
* **W3: SAM + Tracking のハイブリッドパイプライン統合**
  * キーフレームでImage Encoder実行、中間フレームはTracker予測BBox + Mask Decoderのみ実行。
  * **DoD:** PyTorch上で一連の動画処理パイプラインが動作し、FPSとIoUのベースラインが計測可能になる。
* **W4: PyTorchからONNXへのエクスポートと静的グラフ化**
  * 動的テンソルを排除し、各プラットフォームへエクスポート可能なONNXモデルを生成。
  * **DoD:** ONNX Runtime (CPU) にて、PyTorchと同等の推論結果（誤差1e-4以内）を得る。

### Phase 2: エッジ最適化とプラットフォーム実装 (Week 5-8)
* **W5: iOS / CoreML 実装**
  * `coremltools`によるFP16変換とANE最適化。SwiftでのAVFoundationカメラ入力連携。
  * **DoD:** iPhone 13実機でカメラ入力に対し15 FPS以上でセグメンテーション描画が可能。
* **W6: Android / TFLite 実装**
  * PTQによるINT8量子化。CameraX APIとの連携。GPU DelegateおよびXNNPACKでのベンチマーク。
  * **DoD:** Pixel 6実機でカメラ入力に対し15 FPS以上で動作。
* **W7: Web / ONNX Runtime + WebGPU 実装**
  * ブラウザ上での推論環境構築。WASMフォールバック実装。
  * **DoD:** Chrome (WebGPU enabled) にて、M1 Mac上で30 FPS以上で動作。
* **W8: メモリ・レイテンシ・熱の最適化 (サーマルスロットリング対策)**
  * Image Encoderの実行頻度のチューニング（Nフレームに1回）。
  * **DoD:** 5分間の連続実行テストで、端末の過熱による強制終了やFPSの半減（<10 FPS）が発生しないこと。

### Phase 3: SDK化・評価・デモ公開 (Week 9-12)
* **W9: SDKパッケージング**
  * iOS: Swift Package Manager (SPM) 化。Android: AARライブラリ化。Web: NPMパッケージ化。
  * **DoD:** 各パッケージを空の新規プロジェクトにインポートし、10行以内のコードで初期化・実行できること。
* **W10: 評価データセットを用いた定量評価**
  * (後述の評価プロトコルに基づく計測)
  * **DoD:** 評価結果の数値（IoU, FPS等）が「目標数値」をクリア、またはトレードオフの理由が説明可能であること。
* **W11: デモアプリおよびLP (Landing Page) / arXiv 論文執筆**
  * ユーザーが自身の端末で試せるTestFlight/Webデモの構築。
  * **DoD:** 論文ドラフト完成、Webデモのパブリック公開。
* **W12: マーケティングとバズメイキング**
  * Twitter、Hacker News、Reddit (r/MachineLearning) への投稿。
  * **DoD:** ターゲット（Google関係者）からのコンタクト獲得、またはGitHubスター1,000以上獲得。

## 2. 評価プロトコル
* **評価データセット:**
  1. **DAVIS 2017 & YouTube-VOS 2018:** 標準的な動画セグメンテーション指標計測用。
  2. **自作モバイル動画 (In-house Mobile Dataset):** スマホ特有の手ブレ、激しいモーションブラー、逆光、オクルージョンを含む10〜15秒のクリップ50本。
* **評価指標:**
  * **J (Region Similarity):** IoU (Intersection over Union)。
  * **F (Boundary Accuracy):** 輪郭の正確さ。
  * **J&F Mean:** 上記2つの平均値。目標は80%以上。
  * **End-to-End Latency:** カメラフレーム取得から画面描画までのミリ秒。

## 3. 想定リスクと回避策

| リスク (Risk) | 影響度 | 回避策 (Mitigation Strategy) |
|---|---|---|
| **ライセンスの不確実性** | 高 | MetaのSAMはApache 2.0だが、学習用データセット(SA-1B)は非商用。既存の軽量モデルのウェイトライセンスを確認し、問題があれば「SDKコード自体はApache 2.0で公開し、ウェイトはユーザー自身にダウンロードさせる」BYOM (Bring Your Own Model) 形式で法的リスクを遮断する。 |
| **Android端末のGPU性能差** | 高 | TFLite GPU DelegateはMali/Adrenoで挙動や精度が異なる。CIに複数端末のクラウドテストデバイス (Firebase Test Lab等) を組み込み、初期化時にGPU対応可否をプロファイリングし、非互換の場合は自動でXNNPACK (CPU) へフォールバックするロジックを実装する。 |
| **バッテリー過熱 (Thermal)** | 中 | 連続稼働でSoCがダウンクロックする。Image Encoderを走らせるキーフレームの間隔を動的に調整する「Battery Saver Mode」をSDKに設ける（例: オブジェクトの移動量が少ない場合はキーフレーム更新をスキップする）。 |
| **INT8量子化による精度劣化** | 中 | Mask DecoderのINT8化は出力が著しく破綻（ノイズだらけのマスクになる等）する事例がある。Mask Decoderのような軽量な部分はFP16またはFP32のまま据え置き、計算比率の重いImage EncoderのみINT8化するハイブリッド量子化を適用する。 |
