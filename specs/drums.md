# Drum Generation Engine Spec

対応：PRD §10。`packages/arrangement/drums`。

## 基本方針

生成AIに依存せず、テンプレート＋ルールベースの候補生成とスコアリングで決定する（PRD §10.1）。固定パターンの単純反復ではなく、曲構造・他楽器との関係を考慮する。

## 対応パーツ

- MVP：Kick / Snare / Closed Hi-hat / Open Hi-hat
- 拡張：Ride / Crash / Toms / Ghost Notes（PRD §10.2）

## 基本パターン生成

- 4/4拍子の基本バックビートテンプレートを出発点とする（PRD §10.3の例を初期テンプレートとして採用）：

  ```text
  Beat:  1   2   3   4
  Kick:  X       X
  Snare:     X       X
  Hat:   X X X X X X X X
  ```

- テンプレートライブラリ（複数のバックビートバリエーション）を持ち、スタイル・メロディーのアクセントから追加/削除を行う
- フレーズ終端（セクション境界）でのフィル挿入、メロディーのリズム密度・シンコペーション量に応じたハイハット密度の増減を行う

## パターンのスコアリング

各候補パターンを以下の評価要素で採点し、最高スコアの候補を採用する（提案：[harmony.md](./harmony.md)のコードスコアリングと同様の加重和構造）：

- 拍の強弱との整合性
- キックとベースの整合性（[arrangement.md](./arrangement.md)参照。ベース生成より先にドラムを確定させる前提）
- メロディーのアクセントとの整合性
- シンコペーションの量
- リズム密度
- パターンの反復性（繰り返しすぎない）
- フレーズ終端での変化量

初期実装はテンプレート＋ルールによる候補生成とする。評価関数の重みは提案値とし、Fixtureでの聴感評価を通じて調整する。

## Humanization連携

```ts
interface DrumHumanization {
  timingOffsetMs: number;
  velocityOffset: number;
  swingAmount: number;
  ghostNoteProbability: number;
}
```

- 無条件のランダムなズレは禁止し、拍位置・演奏パターン・フレーズに依存するルールを優先する（PRD §10.5）
- 確率的処理（ゴーストノート挿入等）はすべてseed指定により再現可能にする（[expression.md](./expression.md)参照）
