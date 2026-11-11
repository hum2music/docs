# Rhythm Analysis Engine Spec

対応：PRD §7。`packages/audio-analysis`（BPM推定）と`packages/core`（量子化オーケストレーション）に分割実装する想定。

## BPM推定

- 入力：ノート境界検出で得られたオンセット時刻列
- オンセット間隔（Inter-Onset Interval）のヒストグラム、またはオンセット強度列の自己相関からテンポ候補のピークを抽出する
- 倍テンポ・半テンポの曖昧性を考慮し、候補ピークが他候補の2倍/0.5倍に近い場合は`TempoCandidate.isHalfOrDoubleOf`で関連付けて保持する（PRD §7.1）
- 一意に決定できない場合は、信頼度付きで複数候補（例：60/120/180 BPM）を返す。デフォルト選択は最高信頼度の候補とし、`--bpm`指定時は常にユーザー指定を優先する

## 拍子

- MVPでは4/4固定（PRD §7.2）。拍子推定ロジックは実装せず、`Composition.metadata.timeSignature`に`[4, 4]`を設定する
- 3/4・6/8等への対応は将来のスコープとする

## リズム量子化

- 選択されたBPM候補ごとに、各ノートの開始時刻・長さを以下の音価グリッドに当てはめて比較する：
  - 4分音符 / 8分音符 / 16分音符 / 付点音符 / 休符
- 単純に最も近い拍へ丸めるのではなく、複数のリズム解釈（grid hypothesis）を比較し、曲全体での量子化誤差合計が最小になる解釈を選択する（提案：シンプルな貪欲法から開始し、必要であれば小節単位のDPへ拡張）
- 量子化後の`startBeat`/`durationBeats`と、量子化前の実時刻との差分を`NoteEvent.expression.timingOffsetMs`として保持する（PRD §7.3、表現情報として後段のHumanizationが参照する生データ）

## テスト観点

- 固定BPMの合成メロディーFixtureに対するTempo Estimation Error
- 意図的に走り/もたりを含むFixtureに対して、量子化後のメロディーが元の音符数・音高を保持しているか
