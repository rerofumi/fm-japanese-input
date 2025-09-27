# fm-japanese-input

LLM 日本語変換（ローマ字＋英語混在入力を自然な日本語へ変換するデスクトップアプリ）。Neutralinojs 上で動作し、Google Gemini または OpenAI API 互換プロバイダ（OpenRouter / Ollama 等）を利用して変換を行います。

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
- いずれかの LLM プロバイダ設定
  - OpenAI API 互換（OpenRouter / Ollama 等）
    - LLM_BASE_URL（例: https://openrouter.ai/api または http://localhost:11434）
    - LLM_MODEL（例: openai/gpt-4o-mini, llama3.1 など）
    - LLM_API_KEY（任意: OpenRouter 等で必要。Ollama は通常不要）
  - もしくは Google Gemini
    - GEMINI_API_KEY
    - 任意で LLM_MODEL（Gemini モデル名の上書き）
- Neutralino CLI などの開発環境
  - 具体的な起動・ビルド手順は後日追記します

## 設定と優先順位（OpenAI 互換 / Gemini）
このアプリは起動時に以下の順で環境変数や .env を解決し、動作モードを決定します。

1) OpenAI API 互換モード（優先）
- 条件: LLM_BASE_URL と LLM_MODEL が設定されている
- LLM_API_KEY は任意（OpenRouter などでは必要、Ollama では不要）
- エンドポイント: `${LLM_BASE_URL}/v1/chat/completions`
- リクエスト: messages 形式（system + user）、temperature=0.2、非ストリーミング
- 認証: LLM_API_KEY がある場合のみ Authorization: Bearer を付与

2) Gemini モード（フォールバック）
- 条件: OpenAI 互換モードでない かつ GEMINI_API_KEY が設定されている
- モデル名: LLM_MODEL があればそれを使用、なければデフォルト `gemini-2.5-flash-preview-05-20`
- エンドポイント: `https://generativelanguage.googleapis.com/v1beta/models/{MODEL}:generateContent?key=...`

3) 未設定
- いずれの条件も満たさない場合、UI 上で案内を表示し、変換操作は無効化されます。

解決順序（共通）
- OS 環境変数を優先し、無ければ .env（NL_PATH または NL_CWD 直下）を参照します。
- .env は KEY=VALUE 形式で、コメント行（#）や空行は無視されます。

`.env` 例（OpenRouter）:
```
LLM_BASE_URL=https://openrouter.ai/api
LLM_MODEL=openai/gpt-4o-mini
LLM_API_KEY=sk-or-v1_xxx
```

`.env` 例（Ollama）:
```
LLM_BASE_URL=http://localhost:11434
LLM_MODEL=llama3.1
# LLM_API_KEY は不要
```

`.env` 例（Gemini）:
```
GEMINI_API_KEY=your_gemini_key
# 任意で Gemini のモデル名を上書き
LLM_MODEL=gemini-1.5-flash
```

- 先頭・末尾の引用符は不要です（付ける場合は "..." でも可）。
- `.env` はコミットしないでください（.gitignore 推奨）。

補足（実装上の読み込み箇所）:
- OS 環境変数: `Neutralino.os.getEnv(...)`
- `.env` 探索: `filesystem.getJoinedPath(NL_PATH, '.env')` と `filesystem.getJoinedPath(NL_CWD, '.env')` を順に読み込み

## 使い方（UI の流れ）
- 入力欄にローマ字混じりのテキストを入力
- Ctrl+Enter もしくは「変換」ボタンで変換を実行
- 結果は「変換結果」エリアに表示（コピー可能）
- プレビューを有効にすると、入力中も 3 秒間隔で下書き結果が表示されます（薄い文字色＝プレビュー）
- 履歴エリアから過去の入力/出力を復元可能。不要なら「履歴を消去」

## 設定とカスタマイズ（開発者向け）
- モデル／エンドポイント
  - OpenAI 互換: `callOpenAICompatibleAPI()` にて `${LLM_BASE_URL}/v1/chat/completions` を使用
  - Gemini: `callGeminiAPI()` にて `https://generativelanguage.googleapis.com/v1beta/models/{MODEL}:generateContent`
  - Gemini の既定モデル: `gemini-2.5-flash-preview-05-20`（LLM_MODEL で上書き可）
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
