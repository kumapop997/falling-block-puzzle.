# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## リポジトリ概要

Claude Code 学習用のリポジトリ。中身は単一 HTML ファイルの落ちモノパズルゲーム `game/index.html` のみ(日本語 UI)。

- ビルド・依存パッケージ・テスト・lint は存在しない。`game/index.html` をブラウザで直接開いて動作確認する。
- devcontainer (ubuntu24.04 base) には **node / python3 が無い**(jq はある)。ブラウザも無いため、変更後の検証は「HTML の id と JS の `getElementById` の対応を grep で確認」「`{` `}` の数の一致確認」程度に留め、ブラウザでの確認はユーザーに依頼する。
- Git 管理(ブランチ `main`、2026-09-25 開始)。それ以前の変更経緯は Claude のメモリ(`falling-puzzle-history`)にある。
- `game/instructions.md` はユーザーが改良指示を書くファイル(「@game/instructions.md通りに改良して」と頼まれる)。
- ルートの `conversation-*.txt` は過去セッションのエクスポートで、コードではない。

## 作業時の注意

作業前には、必ず、Git commitしてください。

## game/index.html の構造

1 ファイル内に `<style>` → HTML → `<script>` の順で全て入っている。外部ライブラリなし、Canvas + バニラ JS。

- **盤面**: `COLS=10, ROWS=20, CELL=30`。`grid[y][x]` は `null` か色文字列。`SHAPES[type][rotation]` がセル座標配列、`COLORS[type]` が色。
- **ゲーム状態**: スクリプト上部の `let grid, bag, current, nextType, score, lines, level, dropInterval, ... combo, maxCombo;` にまとめて宣言し、`resetGame()` で初期化する。新しい状態を追加したら両方に書く。
- **落下ループ**: `requestAnimationFrame(loop)` で `dropTimer` を加算し `dropInterval` を超えたら 1 段落とす。
- **固定〜消去の流れ**: `lockPiece()` → `clearLines()`(得点・ライン・レベル・落下速度・コンボ処理)→ `spawnNext()`(7-bag)。スポーン位置で衝突すると `triggerGameOver()`。
  - 得点: ライン消去 `[0,100,300,500,800][cleared] * level`、コンボボーナス `50 * combo * level`(combo は -1 始まり、2 連続目で 1)、ソフトドロップ +1/段、ハードドロップ +2/段。
  - レベル: `floor(lines / 10) + 1`。落下間隔: `max(100, 1000 / 1.1^(level-1))`。
- **表示更新**: スコア類は `updateStats()` に集約。
- **画面遷移**: `.board-wrap` 内のオーバーレイ(`#startOverlay` タイトル画面 / `#gameOverOverlay` / `#pauseOverlay`)を `hidden` で切り替える。`startGame()`、`showTitle()`、`togglePause()` を参照。
- **ハイスコア**: localStorage キー `fallingPuzzleHighScore`。読み書きは try/catch で囲む(`loadHighScore` / `saveHighScore`)。
- **操作**: `keydown` ハンドラ(←→↓、↑/X 右回転、Z 左回転、Space ハードドロップ、P 一時停止、Enter で開始)と、狭い画面で表示される `.touch-controls` ボタン。
- **テーマ**: ライトテーマ。色は `:root` の CSS 変数。

## ユーザーの好み・経緯

- ユーザーとのやり取りは日本語。
- ユーザーはClaude Codeの学習のために作業している