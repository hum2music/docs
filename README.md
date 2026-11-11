# Hum2Music Docs

このリポジトリは [hum2music/hum2music](https://github.com/hum2music/hum2music) の `./docs` にgit submoduleとしてマウントされる、設計・仕様ドキュメント専用リポジトリです。

- 製品要求（コンセプト・スコープ・受け入れ基準）は `../PRD.md`（メインリポジトリ直下）が正とする
- 本リポジトリは、PRDをベースにした **実装レベルの詳細仕様**（アーキテクチャ・データモデル・各エンジンの設計）を扱う
- 開発の進め方・ロードマップ・タスク管理は非公開リポジトリで行う

## 構成

| ファイル | 内容 |
|---|---|
| [specs/architecture.md](./specs/architecture.md) | モノレポ構成・パッケージ責務・依存関係 |
| [specs/data-model.md](./specs/data-model.md) | パイプライン中間データ型・JSONスキーマ方針 |
| [specs/audio-analysis.md](./specs/audio-analysis.md) | ピッチ・発音区間検出（YIN等） |
| [specs/rhythm.md](./specs/rhythm.md) | BPM推定・リズム量子化 |
| [specs/harmony.md](./specs/harmony.md) | 調性推定・コード進行探索 |
| [specs/arrangement.md](./specs/arrangement.md) | ピアノ・ベース・ギター編曲エンジン |
| [specs/drums.md](./specs/drums.md) | ドラムパターン生成・スコアリング |
| [specs/expression.md](./specs/expression.md) | Humanization / Expression Engine |
| [specs/export.md](./specs/export.md) | MIDI/JSON/WAV出力仕様 |
| [specs/cli.md](./specs/cli.md) | CLIコマンド・オプション・エラー方針 |
| [specs/ui.md](./specs/ui.md) | localhost UI仕様 |
| [specs/style-profiles.md](./specs/style-profiles.md) | スタイルプロファイルによる横断的パラメーター調整 |
| [specs/non-functional.md](./specs/non-functional.md) | 非機能要件(再現性・性能・オフライン・テスト容易性・エラー処理)の統合 |

## 仕様のステータス表記

各仕様書内の数値・アルゴリズム詳細のうち、PRD.mdに明記のないものは **提案値（要検証）** として記載する。実装・ベンチマークの結果次第で変更され得る。
