# ROADMAP

PRD.md §17「開発フェーズ」を実行可能なタスク単位に分解したもの。進捗の実体（Issue化・担当・期限）は [hum2music/tasks](https://github.com/hum2music/tasks) 側で管理し、本ファイルはフェーズの定義と完了条件のみを保持する。

**現状：Phase 0・Phase 1 完了。Phase 2 未着手。**

---

## Phase 0：音声からMIDIへ

**Goal：鼻歌のメロディーをMIDIに変換できる。**

- [x] Bun workspaceの初期化（`apps/cli`, `packages/core`, `packages/audio-analysis`, `packages/midi`）
- [x] WAV/PCMデコード・モノラル化
- [x] YINによる基本周波数推定
- [x] 周波数→MIDIノート番号変換（セント偏差保持）
- [x] 発音区間（ノート境界）推定
- [x] 短時間異常ピッチ除外・ピッチ軌跡平滑化
- [x] Standard MIDI File出力（単一メロディートラック）
- [x] `hum2music transcribe input.wav -o melody.mid` CLI実装
- [x] 合成音声Fixture（C4単音、Cメジャースケール、ビブラート等）とテスト

**完了条件**：Fixtureに対してPitch Accuracy / Note Onset Error / Note Duration Errorが許容範囲内であること（具体的閾値は実装時にベンチマークして確定する）。✅ 達成（`tests/phase0/` 配下、19件のテストがpass。実装は`hum2music/hum2music`リポジトリのコミット`5b707c3..ea44aaa`）。

## Phase 1：和声の推定

**Goal：メロディーからコード進行を提案できる。**

- [x] `packages/music-theory`（スケール・インターバル・コード理論プリミティブ）
- [x] 発音時刻からのBPM候補生成（倍/半テンポ考慮）
- [x] 拍への量子化（4分/8分/16分/付点/休符の比較選択）
- [x] 調性推定（長調/短調、複数候補＋信頼度）
- [x] コード候補生成（Major/Minor/Dom7/Maj7/Min7/Sus2/Sus4）
- [x] コード進行スコアリング（M, H, V, S）とDPによる全体探索（S項はPhase 5のスタイルプロファイルまで保留）
- [x] JSON中間出力（`--json`）
- [x] `hum2music analyze input.wav --chords` CLI実装

**完了条件**：単純な既知コード進行を持つFixtureメロディーに対して、妥当なコード進行が再構成できること。✅ 達成（`tests/phase1/harmony.test.ts`のI-IV-V-I進行復元テストで確認。実装は`hum2music/hum2music`リポジトリのコミット`0dd0381..6ff5a6a`）。

## Phase 2：最小自動編曲（P0 MVP）

**Goal：1つの鼻歌から複数トラックの楽曲を生成できる。**

- [ ] `InstrumentArranger`共通インターフェース実装
- [ ] Piano Arranger（ブロックコード、滑らかな転回形選択）
- [ ] Bass Arranger（ルート音＋拍位置配置、キックとの整合）
- [ ] Drum生成（Kick/Snare/Closed Hat/Open Hat、4/4バックビート＋アクセント調整）
- [ ] マルチトラックMIDI出力（Track 0: Tempo, 1: Melody, 2: Piano, 3: Bass, 4: Drums）
- [ ] `hum2music generate input.wav -o song.mid` CLI実装
- [ ] seed指定による再現性の担保と検証テスト

**完了条件**：PRD §18「MVP受け入れ基準」をすべて満たすこと。

## Phase 3：Humanization

**Goal：機械的な楽曲に自然な演奏表現を与える。**

- [ ] `packages/expression`（Pitch/Timing/Dynamics Expression Layer）
- [ ] 表現データの保持（pitchCurve、timingOffsetMs等）とMIDI変換（ピッチベンド）
- [ ] スウィング・グルーヴオフセット・アンティシペーション
- [ ] ベロシティアクセント・クレッシェンド/デクレッシェンド・ゴーストノート
- [ ] `DrumHumanization`パラメーター実装（seed対応、無条件ランダム禁止）
- [ ] `--humanize`統合パラメーターの楽器別パラメーターへの変換

## Phase 4：localhost UI

**Goal：鼻歌を録音して、その場で楽曲を確認できる。**

- [ ] `apps/web`（coreを呼び出す薄いAPI）
- [ ] マイク録音・ファイルアップロード
- [ ] ピッチカーブ・推定音符・BPM候補・コード進行候補の表示
- [ ] 楽器ON/OFF・コード進行選択・Humanization設定・再生成
- [ ] MIDI/JSON（/WAV）ダウンロード
- [ ] `hum2music serve --port 3000` CLI実装

## Phase 5：高度な編曲

- [ ] Guitar Arranger（チューニング・運指制約・ストローク）
- [ ] 複数スタイル対応
- [ ] 高度なグルーヴ・曲構造推定
- [ ] より高度なボイシング・MIDI演奏表現の拡張
- [ ] WAV音声出力（ローカル音源/シンセサイザー）

---

## マイルストーン後のレビューポイント

各フェーズ完了時に以下を確認する（PRD §16 品質評価に対応）。

- 単体テストが主要エンジンに存在するか
- 中間データJSONが検証可能か
- 同一入力・設定・seedで出力が再現されるか
- 合成Fixtureに加え、実際の鼻歌録音での評価を行ったか
