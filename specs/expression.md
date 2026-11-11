# Expression / Humanization Engine Spec

対応：PRD §11。`packages/expression`。Phase 3のスコープ。

## 目的

量子化・平均律補正で失われがちな自然な演奏表現を、ルールベースで付加する（完全な乱数ではなく「音楽的な意図」として扱う、PRD §11.1, §11.3）。

## 表現レイヤー

- **Pitch Expression**：Pitch Bend / Vibrato / Portamento / Microtonal Deviation
- **Timing Expression**：Swing / Groove Offset / Anticipation / Delayed Attack
- **Dynamics Expression**：Velocity Accent / Crescendo / Decrescendo / Ghost Note

各レイヤーは独立したルールセットとして実装し、`NoteEvent.expression`（[data-model.md](./data-model.md)）へ書き込む。

## Imperfection Parameter（`--humanize`）

- 単一の乱数係数で全体を揺らすのではなく、`--humanize`の値（0〜1想定）を楽器別・レイヤー別パラメーターへ変換するマッピングを`expression`パッケージ内に持つ
- 例（提案値・要検証）：同じ`--humanize`値でも、ドラムのタイミングジッター許容幅はピアノより広く設定する、など楽器特性に応じた非線形マッピングとする
- マッピングテーブル自体は定数として`expression`パッケージ内に定義し、将来的に外部設定化できる構造にしておく

## Seed管理

- トップレベルの単一`seed`（CLI `--seed`）から、エンジン／トラック／小節単位のサブシードを決定的に導出する（例：`hash(seed, trackName, barIndex)`のような純粋関数。具体的なハッシュ方式は実装時に確定）
- これにより「同じ入力・設定・seedから同じMIDIイベントを生成できる」（PRD §15.1、MVP受け入れ基準）を保証しつつ、トラック間の確率的処理が互いに独立することを両立する
- 無条件のランダム処理（seedに依存しない`Math.random()`等の直接利用）は禁止する

## pitchCurveとMIDI変換

- `pitchCurve`は相対時刻とセント偏差の時系列として保持し、MIDI書き出し時にのみピッチベンドイベントへ変換する（PRD §12）
- 一般的なMIDIピッチベンドはチャンネル単位でしか作用しないため、同時に独立したピッチ変化を要求する複数音（和音内の個別ベンド等）を扱う場合は、チャンネル分離またはMPEが必要になる
- MVPスコープ（メロディーは単旋律）ではこの制約に抵触しないため後回しにするが、Phase 5以降でピアノ/ギターのボイシングに個別ピッチ表現を与える場合は設計を再検討する（PRD §12、オープンな設計課題として明記）
