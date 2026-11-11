# Export Spec

対応：PRD §13。`packages/midi`、`packages/rendering`（P1）。

## MIDI

- Standard MIDI File (SMF Format 1) を出力する
- トラック分離（PRD §13.1）：

  ```text
  Track 0: Tempo / Metadata
  Track 1: Melody
  Track 2: Piano
  Track 3: Bass
  Track 4: Drums
  ```

  ギター対応後は追加トラックとして末尾に追加する。

- General MIDIプログラム番号の初期割当（提案値・要検証）：
  - Piano → Acoustic Grand Piano (#1)
  - Bass → Acoustic/Electric Bass (#33/#34)
  - Drums → GM Percussion Map固定（チャンネル10）
  - Guitar（将来）→ Acoustic/Electric Guitar系（スタイルに応じて選択）
- ピッチベンドはチャンネル単位で作用するため、MVP（メロディー単旋律）では専用チャンネルでの単純なベンド適用に留める。複数音への独立ピッチ適用はMPE検討課題として[expression.md](./expression.md)に記載

## JSON

- `Composition`全体、および各工程の中間データ（[data-model.md](./data-model.md)参照）をJSON出力できるようにする（`hum2music analyze --json`）
- 用途：デバッグ、音楽理論検証、アルゴリズム比較、後続処理での再利用（PRD §13.2）
- 出力JSONには`schemaVersion`を含め、Fixtureテストの比較基準を固定する

## WAV（P1）

- MIDIからのレンダリングにローカル音源またはシンセサイザーを使用する（外部APIなし、PRD §13.3）
- 初期実装では高品質な楽器音の再現を必須条件とせず、「プレビューとして聴感的に確認できる」レベルを目標とする
- 候補実装方式（要検討・未確定）：SoundFont(SF2)ベースの簡易シンセサイザー、または軽量な内蔵波形合成
