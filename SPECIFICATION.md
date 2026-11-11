# Hum2Music 完成系 技術仕様書

本書は、PRD.md（プロダクト要求定義）とROADMAP.md（フェーズ別タスク分解）を踏まえ、**Phase 0〜5がすべて完了した時点での完成系**を一つの技術仕様として統合したものである。個別の実装詳細は`specs/`配下の各ドキュメントに委ね、本書はそれらの関係と、完成系が満たすべき全体像・受け入れ基準を定義する「仕様のインデックス兼最終合意点」として機能する。

実装は本書を一度に満たすのではなく、ROADMAP.mdのフェーズ順に段階的に満たしていく。各フェーズの途中では、本書に書かれた項目のうち未実装の部分があって構わない（それは「未完成」であって「仕様違反」ではない）。

## 1. 完成系のスコープ境界

完成系とは、PRD §3.1のF-001〜F-017（P0+P1すべて）が実装され、PRD §17 Phase 0〜5がすべて完了した状態を指す。

次はスコープ外のまま据え置く（PRD §3.2）：

- 歌詞生成・歌声合成・ボーカル自動生成
- 複数人同時編集・クラウド同期
- 本格的なDAW機能・VST/AUプラグイン形式
- ポリフォニック音源からの楽器分離
- リアルタイム完全自動編曲

PRD §20「将来構想」（Musical Imperfection Engine高度化、Music Theory Optimization Engine、DAW連携、Music Research Toolkit）は、完成系のさらに先にあるビジョンであり、本書が定義する受け入れ基準には含めない。

## 2. 完成系のシステム構成

```text
hum2music/
├── apps/
│   ├── cli/
│   └── web/                      # Phase 4
├── packages/
│   ├── core/
│   ├── audio-analysis/
│   ├── music-theory/
│   ├── harmony/
│   ├── arrangement/
│   │   ├── piano/
│   │   ├── guitar/                # Phase 5
│   │   ├── bass/
│   │   └── drums/
│   ├── expression/                # Phase 3
│   ├── midi/
│   └── rendering/                 # Phase 5 (WAV)
├── fixtures/
├── tests/
└── docs/
```

詳細は[specs/architecture.md](./specs/architecture.md)。依存方向・モジュール分離方針はPhase 0から完成系まで一貫して変更しない。

## 3. 完成系のデータパイプライン

```text
Audio File
  → Audio Preprocessing
  → Pitch & Onset Detection            [specs/audio-analysis.md]
  → Melody Representation
  → Rhythm Analysis                    [specs/rhythm.md]
  → Key / Harmony Analysis             [specs/harmony.md]
  → Chord Progression Search           [specs/harmony.md]
  → Instrument Arrangement             [specs/arrangement.md, specs/drums.md]
  → Style Profile適用                  [specs/style-profiles.md]
  → Humanization Engine                [specs/expression.md]
  → MIDI Export / Audio Rendering      [specs/export.md]
```

`Style Profile適用`は完成系で追加される工程であり、Arrangement〜Humanizationの各ステップのパラメーターを横断的に調整する（新規の分岐ロジックではなく重み付けとして実装する、[specs/style-profiles.md](./specs/style-profiles.md)）。

ベースアレンジャーはドラムのキック位置を参照するため、完成系でもドラム生成→ベース生成の順に実行する（[specs/arrangement.md](./specs/arrangement.md)）。

## 4. 完成系の機能一覧とフェーズ対応

| 機能領域 | 完成系の内容 | 対応Phase | 詳細仕様 |
|---|---|---|---|
| 音声入力・ピッチ推定 | WAV入力、YIN、セント偏差保持、信頼度付きノート境界検出 | 0 | [audio-analysis.md](./specs/audio-analysis.md) |
| MIDI変換（単旋律） | メロディートラックのSMF出力 | 0 | [export.md](./specs/export.md) |
| BPM・拍子・量子化 | 複数BPM候補、4/4固定、複数音価グリッド比較 | 1 | [rhythm.md](./specs/rhythm.md) |
| 調性・コード推定 | 複数調性候補、7種のコードタイプ、DPによる進行探索 | 1 | [harmony.md](./specs/harmony.md) |
| ピアノ/ベース編曲 | ボイスリーディング最適化、キック同期ベース | 2 | [arrangement.md](./specs/arrangement.md) |
| ドラム生成 | テンプレート＋スコアリング、4パーツ（MVP）+Ride/Crash/Toms/Ghost | 2/5 | [drums.md](./specs/drums.md) |
| マルチトラックMIDI出力 | Track 0-4（+Guitar） | 2/5 | [export.md](./specs/export.md) |
| Humanization | Pitch/Timing/Dynamics Expression、seed対応 | 3 | [expression.md](./specs/expression.md) |
| localhost UI | 録音・解析可視化・編曲調整・エクスポート | 4 | [ui.md](./specs/ui.md) |
| ギター編曲 | チューニング/運指制約つきストローク・アルペジオ | 5 | [arrangement.md](./specs/arrangement.md) |
| 複数スタイル | スタイルプロファイルによる横断的パラメーター調整 | 5 | [style-profiles.md](./specs/style-profiles.md) |
| WAV出力 | ローカル音源/シンセサイザーによるレンダリング | 5(P1) | [export.md](./specs/export.md) |
| CLI全体 | transcribe/analyze/generate/serve全コマンド | 0-4 | [cli.md](./specs/cli.md) |
| 非機能要件 | 再現性・性能・オフライン・テスト容易性・エラー処理 | 全フェーズ | [non-functional.md](./specs/non-functional.md) |

## 5. 完成系の受け入れ基準

PRD §18（P0 MVP受け入れ基準）に加え、完成系では以下を満たすこと。

### P0（MVP、PRD §18そのまま）

- [ ] 外部のAI APIを使わずに動作する
- [ ] WAVファイルから単旋律を抽出できる
- [ ] 有効な音符をMIDIとして出力できる
- [ ] テンポ候補を推定できる
- [ ] メロディーに適合したコード進行候補を生成できる
- [ ] ピアノ伴奏・ベースライン・4/4ドラムパターンを生成できる
- [ ] メロディー/ピアノ/ベース/ドラムが別トラックで出力される
- [ ] CLIから一連の処理を実行できる
- [ ] JSONで中間結果を検証できる
- [ ] 同じ入力・設定・seedから同じMIDIイベントを生成できる
- [ ] 主要エンジンに自動テストが存在する

### P1 / 完成系追加項目

- [ ] 入力音声のピッチ変化情報（ビブラート等）がMIDIピッチベンドとして反映される
- [ ] `--bpm` / `--style` / `--key` / `--chords`によるユーザー指定が自動推定より優先される
- [ ] ギターアレンジャーが演奏不可能な運指を生成しない（物理制約の自動検証テストを含む）
- [ ] localhost UIから録音→解析→編曲調整→エクスポートまでを完結できる
- [ ] WAV出力がMIDIと一貫した内容でレンダリングされる
- [ ] 複数のスタイルプロファイルが、各エンジンのスコアリング/テンプレート選択に反映され、出力に可聴な差を生む
- [ ] `--humanize`が楽器別パラメーターへ適切に変換され、無条件の乱数ではなくルール依存で演奏表現が生成される
- [ ] 非機能要件（[non-functional.md](./specs/non-functional.md)）すべてに対応するテスト/ベンチマークが存在する

## 6. 本書の位置付けと更新方針

- PRD.mdは製品要求として不変に近い（変更時は明示的な改訂）。本書と`specs/`は実装の進行・ベンチマーク結果に応じて更新してよい
- `specs/`内の「提案値・要検証」は、実装・計測の結果が出た時点で確定値に更新し、本書の該当箇所もあわせて見直す
- 新しいフェーズに着手する際は、まず本書の該当セクションと対応する`specs/*.md`を読み、矛盾があればPRD.mdを優先して解決する
