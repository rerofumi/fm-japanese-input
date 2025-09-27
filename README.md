# fm-japanese-input

LLM 日本語変換（ローマ字＋英語混在入力を自然な日本語へ変換するデスクトップアプリ）。Neutralinojs 上で動作し、Google Gemini API を利用して変換を行います。

> 注記: 本 README は開発者向けです。正式な起動・ビルド手順は後日記載します。

---

## 概要
- 半角英数（ローマ字）と英語が混ざったテキストを、文脈に沿って自然な日本語へ変換します。
- 英単語・英文はそのまま維持し、ローマ字部分のみを日本語（ひらがな・カタカナ・漢字）に変換します。
- Neutralinojs を用いたクロスプラットフォームなデスクトップ UI（Tailwind CSS ベース）。

## 主な機能
- 変換操作
  - 入力欄にテキストを入力し「変換」ボタン、または Ctrl+Enter で変換
- リアルタイムプレビュー（お試し）
  - トグルで ON/OFF
  - 入力の様子を 3 秒間隔で Gemini に投げ、プレビュー結果を薄い文字色で表示（確定ではありません）
- コピー／リセット
  - 変換結果のコピー、入力のリセットボタンを用意
- 変換履歴
  - 直近最大 10 件を localStorage に保存
  - リストから選ぶと、入力欄と出力へ復元
  - 「履歴を消去」ボタンで全削除
- エラーハンドリング／リトライ
  - 429/5xx 応答時は指数バックオフで最大 3 回再試行
  - 失敗時は UI にエラー表示

## 必要要件（概要）
- Google Gemini API キー（GEMINI_API_KEY）
  - 現実装のエンドポイント: `gemini-2.5-flash-preview-05-20`
- Neutralino CLI などの開発環境
  - 具体的な起動・ビルド手順は後日追記します

## 環境変数の設定（API キー）
このアプリは起動時に以下の順で API キーを探索します。
1. OS の環境変数 `GEMINI_API_KEY`
2. `.env` ファイル（アプリ本体パス（NL_PATH）直下、または実行時カレント（NL_CWD）直下）

`.env` 例:

```
GEMINI_API_KEY=your_api_key_here
```

- 先頭・末尾の引用符は不要です（付ける場合は "..." でも可）。
- `.env` をリポジトリにコミットしないよう注意してください（.gitignore 推奨）。

補足（実装上の読み込み箇所）:
- OS 環境変数: `Neutralino.os.getEnv('GEMINI_API_KEY')`
- `.env` 探索: `filesystem.getJoinedPath(NL_PATH, '.env')` と `filesystem.getJoinedPath(NL_CWD, '.env')` を順に読み込み

## 使い方（UI の流れ）
- 入力欄にローマ字混じりのテキストを入力
- Ctrl+Enter もしくは「変換」ボタンで変換を実行
- 結果は「変換結果」エリアに表示（コピー可能）
- プレビューを有効にすると、入力中も 3 秒間隔で下書き結果が表示されます（薄い文字色＝プレビュー）
- 履歴エリアから過去の入力/出力を復元可能。不要なら「履歴を消去」

## 設定とカスタマイズ（開発者向け）
- モデル／エンドポイント
  - `www/index.html` 内の `callGeminiAPI()` で使用モデルと URL を定義
  - 既定: `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent`
- システムプロンプト
  - `www/index.html` の `systemPrompt` を編集
  - 変換仕様（英語は保持、ローマ字を自然な日本語に、説明文の出力禁止 等）を制御
- プレビューの間隔
  - `PREVIEW_TICK_INTERVAL = 3000`（ミリ秒）を調整
- 履歴上限
  - `MAX_HISTORY_ITEMS = 10`
- ウィンドウ設定
  - `neutralino.config.json` の `modes.window`（タイトル、サイズ、アイコン など）

## ディレクトリ構成（抜粋）
```
.
├─ neutralino.config.json     # Neutralino 設定（タイトル/ウィンドウ/CLI など）
├─ www/
│  ├─ index.html              # UI とアプリロジック（Gemini 呼び出し含む）
│  ├─ neutralino.js           # Neutralino クライアントライブラリ
│  ├─ neutralino.d.ts         # 型定義（参考）
│  └─ icon.png                # アプリアイコン
└─ LICENSE                    # MIT ライセンス
```

## 既知の制約・注意
- API キー未設定時は変換できません。`.env` または環境変数の設定を確認してください。
- プレビューは入力ごとにリクエストが発生するため、レート制限に注意してください（必要に応じてトグルで OFF）。
- 変換ルール上、英語はそのまま維持されます。

## 起動・ビルド手順（後日記載）
- 開発用の実行方法（neu run 等）やビルド手順（neu build 等）は後日追記します。

## ライセンス
- 本プロジェクトは MIT License です。詳細はリポジトリ同梱の `LICENSE` を参照してください。

## 謝辞
- [Neutralinojs](https://neutralino.js.org/)
- Google Gemini API
