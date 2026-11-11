# Data Model Spec

PRD §12の型を実装レベルに拡張したもの。パイプラインの各段階は「解析前（raw）」と「正規化後（quantized/selected）」を明確に分離して保持する（PRD §6.4, §7.3）。

## PRD由来の確定型

```ts
interface NoteEvent {
  pitch: number;
  startBeat: number;
  durationBeats: number;
  velocity: number;
  expression?: {
    pitchCurve?: number[];
    timingOffsetMs?: number;
  };
}

interface ChordEvent {
  symbol: string;
  startBeat: number;
  durationBeats: number;
  confidence: number;
}

interface InstrumentTrack {
  instrument: string;
  notes: NoteEvent[];
}

interface Composition {
  metadata: {
    bpm: number;
    timeSignature: [number, number];
    key?: string;
  };
  melody: NoteEvent[];
  chords: ChordEvent[];
  tracks: InstrumentTrack[];
}
```

## 追加提案型（提案値・実装時に確定）

中間工程での「複数候補＋信頼度を保持する」という原則（PRD §6, §7, §8）をコードレベルで表現するための型。

```ts
// audio-analysis: フレーム単位の生ピッチ情報
interface PitchFrame {
  timeMs: number;
  f0Hz: number | null;       // 無声/無音はnull
  confidence: number;        // 0-1
  voiced: boolean;
}

// audio-analysis: 量子化前のノート候補（境界検出結果）
interface NoteCandidate {
  startMs: number;
  durationMs: number;
  rawPitch: number;          // 四捨五入前のMIDIノート相当値
  pitchTrajectory: PitchFrame[];
  onsetConfidence: number;
}

// rhythm: BPM候補
interface TempoCandidate {
  bpm: number;
  confidence: number;
  isHalfOrDoubleOf?: number; // 倍/半テンポ関係にある候補のbpmを指す
}

// harmony: 調性候補
interface KeyCandidate {
  tonic: string;             // 例: "C"
  mode: "major" | "minor";
  confidence: number;
}

// harmony: コード候補（選択前、スコア内訳つき）
interface ChordCandidate {
  symbol: string;
  startBeat: number;
  durationBeats: number;
  scores: { melodyFit: number; harmonicNaturalness: number; voiceLeading: number; styleFit: number };
  totalScore: number;
}

// arrangement: 各Arrangerへ渡すコンテキスト
interface ArrangementContext {
  composition: Composition;
  instrument: string;
  style?: string;
  seed: number;
  range?: { minPitch: number; maxPitch: number };
}
```

## JSONエクスポート方針

- `Composition`全体ダンプとは別に、工程ごとの中間JSON（`PitchFrame[]`, `NoteCandidate[]`, `TempoCandidate[]`, `KeyCandidate[]`, `ChordCandidate[]`）を `hum2music analyze --json` で出力できるようにする（PRD §13.2の用途：デバッグ・理論検証・アルゴリズム比較・再利用）。
- 将来の形式変更に備え、出力JSONには `schemaVersion` フィールドを持たせる。Fixtureテストはこのバージョンに固定して比較する。
- `pitchCurve`は最終的に「相対時刻＋セント偏差の時系列」へ拡張し、MIDI書き出し時にのみピッチベンドへ変換する（生データとMIDI表現を分離。PRD §12）。
