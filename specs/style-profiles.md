# Style Profiles Spec（完成系）

PRD全体で触れられる「スタイル」（`--style`オプション、PRD §4.2・§8.3のS項・§10.4・§11.3）を、完成系でどう実装するかを一つの仕様としてまとめる。MVP（Phase 2）ではニュートラルな単一プロファイルのみ実装し、本書はPhase 5までの完成形を定義する。

## スタイルの役割

スタイルは新しい生成アルゴリズムを追加するものではなく、既存のスコアリング・テンプレート選択・パラメーターに対する**重み付けプロファイル**として実装する。これにより、各エンジンの決定的性質（PRD §15.1）を壊さずにスタイルごとの違いを表現する。

## プロファイルが影響する項目

| 項目 | 影響するエンジン | 内容 |
|---|---|---|
| コード進行スコアリングの重み (`wM, wH, wV, wS`のうち`wS`の具体的な内訳) | harmony | テンション使用頻度、機能和声 vs モーダルな進行の許容度（[harmony.md](./harmony.md)） |
| ピアノ伴奏パターン選択の重み | arrangement/piano | ブロックコード／アルペジオ／リズミック伴奏の出現比率（[arrangement.md](./arrangement.md)） |
| ベースのアプローチノート使用頻度 | arrangement/bass | 経過音・半音アプローチの出現率 |
| ギターのストローク/アルペジオ比率、ミュート頻度 | arrangement/guitar | （Phase 5） |
| ドラムテンプレート群の選択範囲とシンコペーション量の既定値 | arrangement/drums | テンプレートライブラリのサブセット選択（[drums.md](./drums.md)） |
| Humanizeパラメーターの既定スケール | expression | `--humanize`の楽器別マッピングのベースライン（[expression.md](./expression.md)） |
| GM音色プログラム番号の既定選択 | midi/export | 同一楽器でもスタイルにより音色候補が変わる（例：ギターのアコースティック/エレキ）（[export.md](./export.md)） |

## プロファイル定義の構造（提案・要検証）

```ts
interface StyleProfile {
  name: string; // 例: "pop", "ballad", "jazz" など。MVP後に具体的なプリセットを確定する
  harmonyWeights: { tensionBias: number; modalBias: number };
  pianoPatternMix: { block: number; arpeggio: number; rhythmic: number };
  bassApproachFrequency: number;
  guitarMix: { strum: number; arpeggio: number; muteFrequency: number };
  drumTemplatePoolId: string;
  humanizeBaseline: Record<string, number>; // 楽器名 -> 既定humanize寄与度
  defaultProgramNumbers: Record<string, number>;
}
```

- 各フィールドは0-1の相対重みとし、既存のスコアリング関数・テンプレート選択ロジックへの乗算係数として使う（新しい分岐ロジックを増やさない）
- プロファイル自体は`packages/core`または`packages/arrangement`内の定数テーブルとして持ち、JSONとしてもエクスポートしてUIから参照できるようにする

## 具体的なプリセット

MVP完成時点ではプリセットを確定しない。少なくとも1つのニュートラルプロファイル（すべての重みが中立値）を既定として提供し、Phase 5で追加プリセット（例：pop / ballad 等）を具体的な数値とともに定義する。本書はプリセットの**実装の型**を定義するものであり、各プリセットの具体的な数値はFixtureベースの聴感評価を経て確定する（PRD §16.3 主観評価の方針に従う）。
