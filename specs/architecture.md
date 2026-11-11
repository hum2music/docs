# Architecture Spec

PRD §5「システムアーキテクチャ」の実装レベル詳細。

## モノレポ構成と責務

```text
hum2music/
├── apps/
│   ├── cli/            # コマンド実装。packages/coreのみに依存する薄い層
│   └── web/             # P1。coreを呼ぶ薄いAPI + UI
├── packages/
│   ├── core/            # パイプライン全体のオーケストレーション、公開型定義、エンジンバージョン管理
│   ├── audio-analysis/  # WAVデコード、YINピッチ推定、発音区間検出
│   ├── music-theory/    # スケール/インターバル/コード理論の純粋関数群（harmony, arrangementの共有leaf依存）
│   ├── harmony/         # 調性推定、コード候補生成・スコアリング・DP探索
│   ├── arrangement/
│   │   ├── piano/
│   │   ├── guitar/      # P1
│   │   ├── bass/
│   │   └── drums/
│   ├── expression/      # Humanizationエンジン、seeded RNG
│   ├── midi/            # Standard MIDI File 読み書き、ピッチベンド/チャンネル割当
│   └── rendering/       # P1。MIDI→WAVレンダリング
├── fixtures/
├── tests/
└── docs/                # 本submodule
```

## 依存方向

```text
apps/cli, apps/web
        │
        ▼
    packages/core
        │
   ┌────┼─────────────┬───────────┬────────────┬────────┐
   ▼    ▼             ▼           ▼            ▼        ▼
audio-  music-theory  harmony   arrangement  expression  midi/rendering
analysis     ▲            │          │
             └────────────┴──────────┘
```

- `music-theory`は`harmony`と`arrangement`の両方から参照される純粋ロジック層とし、音声解析・I/Oに関する依存を持たない。
- `core`はどのエンジン実装を使うか（アルゴリズムの差し替え）を注入可能にし、各パッケージを独立にテストできる状態を維持する（PRD §15.4）。
- `apps/cli`・`apps/web`は`core`の公開APIのみを呼び、業務ロジックを持たない。

## パイプラインオーケストレーション（`packages/core`）

`core`は以下を保持する：

- PRD §5.2のデータパイプラインを関数合成として実行するオーケストレーター
- 各工程の中間結果をJSONへシリアライズする共通フック（`--json`対応、PRD §13.2）
- `engineVersion`（各サブパッケージのバージョン or アルゴリズムID）を`Composition.metadata`に埋め込み、再現性検証（PRD §15.1）とゴールデンFixtureテストの基準にする
- 全体で使う単一の`seed`から各エンジン・各トラック向けのサブシードを決定的に導出する仕組み（詳細は[expression.md](./expression.md)）

## なぜモジュール分割するか

- 各工程を独立にテスト・差し替え可能にする（PRD §1.3 Modularity、§9.1 `InstrumentArranger`インターフェース）
- 将来のGuitar Arranger追加（Phase 5）やスタイル追加時に、他モジュールへの影響を局所化する
- `music-theory`を薄く保つことで、将来の「Music Theory Optimization Engine」（PRD §20.2）が制約ソルバーとして差し替え可能になる
