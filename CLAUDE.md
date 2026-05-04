# CLAUDE.md — mobile-sam

> 🚨 **MUST READ FIRST**: 作業を始める前に [`rules/README.md`](rules/README.md) を起点に `rules/` 配下の全ファイルを読むこと。
> `rules/` の内容は本ファイルより**優先**される（矛盾した場合は rules/ が勝つ）。
> ルールが不明確、もしくはユーザーの依頼と矛盾する場合は、実装前に必ずユーザーに確認すること。

## プロジェクト概要

モバイル / ブラウザで動く軽量 SAM ベースの動的オブジェクト追跡 SDK。
**コア技術 = モデル圧縮 + マルチプラットフォーム推論ラッパー**。
ゴール: Google（Lens / Photos / YouTube）による acqui-hire。詳細は README 参照。

## 設計原則

- **3 プラットフォーム同精度**: iOS / Android / Web で同じマスク品質が出るまで妥協しない
- **モデルは公開、SDK は薄く**: モデル品質で勝つ
- **リアルタイム性絶対**: バッチ処理用途は対象外
- **ライセンスを汚さない**: 教師モデル（SAM 系）のライセンスを必ず確認してから蒸留

## ディレクトリ構成

`model/` がモデル開発、`sdk/` が iOS/Android ラッパー、`examples/` がデモ。
詳細は README 参照。

## ブランチ命名

```
feature-{name}-{description}
bug-{name}-{description}
refactoring-{name}-{description}
```

メインブランチは `main`、PR は `main` に向ける。

## コミット前チェック

- モデル変更: 評価データセットでの IoU が劣化していないこと
- iOS: Xcode build が通ること
- Android: `./gradlew assembleRelease` が通ること
- モデル変換: CoreML / TFLite どちらも変換成功

## スコープ（やる / やらない）

[`rules/02-scope.md`](rules/02-scope.md) を正とする。CLAUDE.md には重複記載しない。
