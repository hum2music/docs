# CLI Spec

対応：PRD §4.2、§14.1。`apps/cli`。CLIは`packages/core`の公開APIを呼ぶ薄い層とする。

## コマンド一覧

| コマンド | 説明 |
|---|---|
| `hum2music transcribe <input.wav> -o <melody.mid>` | メロディー抽出とMIDI出力のみ |
| `hum2music analyze <input.wav> [--chords] [--json]` | 解析結果（BPM候補・調性・コード進行候補等）の表示/出力 |
| `hum2music generate <input.wav> [options] -o <song.mid>` | マルチトラック自動編曲（MVP） |
| `hum2music serve --port <port>` | localhost UI起動 |

## オプション（抜粋・PRD §4.2, §8.4, §11.3準拠）

| オプション | 説明 | 優先度 |
|---|---|---|
| `--bpm <number>` | BPMを明示指定し、自動推定より優先する | P0 |
| `--style <name>` | 楽曲スタイル指定（コード/ドラムのスコアリング重みに影響） | P1 |
| `--seed <number>` | 確率的処理の再現性を固定する | P0 |
| `--instruments <list>` | 生成する楽器をカンマ区切りで選択（例：`piano,bass,drums`） | P0 |
| `--key <name>` | 調性を明示指定し、自動推定より優先する | P1 |
| `--chords <progression>` | コード進行を明示指定し、自動推定より優先する | P1 |
| `--json` | 中間解析結果をJSONで出力する | P0 |
| `--humanize <0-1>` | 演奏表現の統合パラメーター（楽器別パラメーターへ変換される） | P1 |
| `--port <number>` | `serve`コマンドのポート指定 | P1 |

初期CLIではファイル入力を必須とし、マイクからの直接録音対応はlocalhost UI以降とする（PRD §4.2）。

## エラー処理方針（PRD §15.5）

以下の状態を検出した場合、原因と利用可能な代替操作をユーザーに提示する。確定困難な値を確定値として扱わない（例：BPM推定に失敗した場合は複数候補を提示し、強制的に単一値へfallbackしない）。

- 音声ファイルの破損・対応外フォーマット
- 有効な音高が検出できない
- テンポが推定できない
- コード進行の信頼度が低い
