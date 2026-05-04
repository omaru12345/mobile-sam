# Acquisition Thesis: Google Acqui-hire Strategy

本ドキュメントは、本プロジェクトがGoogleによるAcqui-hire（タレント/技術買収）を引き出すための論理的根拠、ポジショニング、および露出戦略を定義する。

## 1. なぜGoogleが我々を買うのか (The "Why")
Appleの「Apple Intelligence」やVision Frameworkのオンデバイス強化に対し、GoogleはAndroidエコシステム全体（Pixelからローエンド端末まで）でエッジAIの優位性を証明する必要がある。我々の「超軽量モデルをWebGPU/TFLiteで極限まで最適化する」技術スタックと、「動画のオブジェクトをエッジでリアルタイムに追跡・分離する」IPは、Googleの主要プロダクトのボトルネック（クラウド推論コスト、通信遅延、プライバシー）を直接的に解消する。

## 2. プロダクト別 具体的ニーズ
* **Google Photos ("Video Magic Eraser" の実現):**
  静止画のMagic Eraserは成功したが、動画版は計算量の壁がある。我々のSDKは、動画内の動的オブジェクトのマスキングをエッジ側（ユーザーの端末）で完結させ、背景補完モデルへの入力マスクをゼロ・レイテンシで生成できる。
* **Google Lens (動的コンテキスト検索):**
  カメラをかざしている最中に、動いている被写体（車、動物、服）をリアルタイムに自動セグメンテーションしてハイライト表示する「選択的検索」のUXを飛躍的に向上させる。
* **YouTube Create (モバイルネイティブな動画編集):**
  グリーンバックなしでの高精度な人物/オブジェクトの自動切り抜き（Rotoscoping）。クラウドへ動画をアップロードして処理する現在のパイプラインをオンデバイス化し、サーバーコストを劇的に削減する。

## 3. ポジショニング
**「Mobile/Web Nativeの Video Magic Eraser 基盤」**
MetaのSAM 2は動画対応を果たしたが、アーキテクチャ（Memory Attention等）が重く、依然としてエッジ（特にブラウザやミドルエンドのAndroid）ではリアルタイム稼働が困難である。我々は「精度を数%犠牲にしてでも、エッジで30FPS以上で動かすエンジニアリング力」に特化し、実用性を極める。

## 4. 競合優位性（技術の堀）
1. **極限のエッジ最適化ノウハウ:**
   PyTorchモデルをただ変換するだけでなく、CoreMLのANEアーキテクチャ特有のテンソル制約回避、TFLiteのGPU Delegateのキャッシュ最適化、WebGPUのCompute Shaderメモリ管理まで、フルスタックでチューニングできる点。
2. **Tracking + Maskingのハイブリッド・パイプライン:**
   重いImage Encoderを全フレームで回すのではなく、カルマンフィルタベースの軽量トラッカー（ByteTrack）とMask Decoderを連動させることで、計算量（バッテリー消費）を1/10に抑えるアーキテクチャ設計。

## 5. 研究コミュニティでの露出戦略（バズの設計）
GoogleのAIリサーチチーム（Google DeepMind / Google Research）およびプロダクトリードの目に留まるための戦略。
* **arXiv Paper:** タイトル案 *"Mobile-SAM-Track: Real-time On-device Video Segmentation and Tracking"*。理論の新規性ではなく「エンジニアリング的ブレイクスルーと実機パフォーマンス」に焦点を当てる。
* **CVPR / ICCV などのDemo Track:** 実機（低スペックなAndroid端末を含む）でリアルタイムに動くブースを出展。
* **Twitter (X) / LinkedIn でのバイラル:**
  ブラウザ（Chrome WebGPU）上で、YouTube動画の再生中にリアルタイムでオブジェクトをクリック＆ドラッグして切り抜いている画面録画を公開。「No Cloud, Pure WebGPU」を強調する。
