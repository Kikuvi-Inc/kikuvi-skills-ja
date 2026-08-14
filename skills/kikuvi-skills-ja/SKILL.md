---
name: kikuvi-skills-ja
description: >-
  Use Kikuvi to create, manage, and analyze interviews. Activate when the user
  wants to "create an interview design", "run an interview session", "analyze
  interview results", "generate a report", or is working with the Kikuvi
  platform.
metadata:
  author: Kikuvi-Inc
  version: "0.1.0"
---

# Kikuvi Skills

## Kikuvi とは
Kikuvi は AI ヒアリングエージェント。AI が対話形式でインタビュー（ヒアリング）を自動実施し、回答を収集・分析・レポート化する。

## エンティティ構造

- **Organization**（組織）: 請求・メンバー管理の最上位単位
  - **Project**（プロジェクト）: テーマ別のヒアリンググループ
    - **Design**（ヒアリング設計）: 1ヒアリングの設計書（質問・対象者を定義）
      - **Section**（セクション）: アイテムをテーマ別にグループ化
        - **質問**: 対象者に尋ねる項目（7タイプ。「質問タイプ」を参照）
        - **AIへの指示**: 実施中の AI への指示。質問文は AI がその場で組み立てる（キクゾウ(V2) のみ）
      - **Session**（セッション）: 対象者1人=1セッション（実際の会話）
        - **SessionReport**: セッション完了時に自動生成
      - **DesignReport**: 全セッション完了後に自動生成
    - **ProjectReport**: 生成toolをcallして生成

## インタビュアーの選択（V1 / V2）
ヒアリング設計ごとに、対象者と会話するインタビュアーを選ぶ。

| 値 | 名称 | 特徴 |
|---|---|---|
| `V1` | 標準 | 従来のインタビュアー。デフォルト |
| `V2` | キクゾウ | リアルタイム音声基盤によるより自然な会話。「AIへの指示」タイプを使える |

- どちらを使うかはユーザーに確認する。推測で切り替えない
- 「AIへの指示」を1つでも含む設計は `V2` を指定する。`V1` のままだとエラーになる
- 作成後の変更は `design_update_metadata`。V1 → V2 は設計中（IN_DESIGN）のみ可能で、**V2 → V1 には戻せない**

## 質問タイプ
対象者に尋ねる質問7タイプと「AIへの指示」の計8タイプ。全タイプ共通で `title`（項目名）と `shouldFollowUp`（深掘りの有無）を持つ。

| `type` | 画面表示 | 用途 | 固有の設定 |
|---|---|---|---|
| `QUALITATIVE` | AI質問（会話型） | 自由回答で深掘りする（経験・意見・理由） | `shouldFollowUp` は常に `true` |
| `TEXT` | テキスト入力 | 音声では答えにくく、正確な値を取りたい情報（メールアドレス・電話番号など） | なし |
| `SINGLE_SELECTION` | 単一選択 | 選択肢から1つ選ぶ | `choices` |
| `MULTIPLE_SELECTION` | 複数選択 | 選択肢から複数選ぶ | `choices` |
| `NUMBER_IN_RANGE_KEYBOARD` | 数値入力 | 数値をキーボードで入力する | `min` / `max` / `step` |
| `NUMBER_IN_RANGE_SLIDER` | スライダー | 数値をスライダーで選ぶ | `min` / `max` / `step` / `minLabel` / `maxLabel` |
| `RATE` | レーティング | 満足度・評価などの尺度 | `icon` / `max` |
| `AI_INSTRUCTION` | AIへの指示 | 他の回答に応じて進め方が変わる（キクゾウ(V2) のみ） | `instruction` |

- 意見・経験・理由を聞く質問は、短い回答を想定していても `QUALITATIVE`。`TEXT` は音声で正確に伝えられない値をキーボードで入力してもらうためのタイプ
- `QUALITATIVE` は深掘り前提のタイプ。`shouldFollowUp: false` は拒否されるため、深掘り不要な質問は他のタイプで表現する
- 選択・数値・スライダー・レーティングの回答はレポートの定量データとして集計される
- 選択肢の個数上限や必須項目などの制約は記憶せず、MCP のスキーマで確認する

### AIへの指示
質問文の代わりに、ヒアリング中の AI への指示を `instruction` に文章で書くアイテム。対象者に実際に聞く質問は、その時点までの回答をもとに AI が組み立てる。

使うのは、他のアイテムへの回答に依存して進め方が変わる場合のみ:
- 条件分岐（例: 「満足度を3未満と答えた人にだけ、何が不足していたかを聞いて」）
- 直前の回答ごとの繰り返し（例: 「選ばれたツールそれぞれについて使い方を聞いて」）
- 回答次第で深掘りの方向が決まる場合

書き方:
- 条件・各分岐で聞くこと・スキップする条件を文章で明記する
- 他のアイテムはIDではなく内容で参照する（実施中の AI はIDを持たない）
- `title` は一覧に表示される内部用のラベル。対象者には表示されない
- 深掘りは `instruction` 内で指示する（`shouldFollowUp` は `false`）
- 1セクションあたり1〜2個まで。会話中に間が生じるため、条件分岐が不要なら `QUALITATIVE` を使う

典型パターン: 特定の選択肢だけ深掘りする
定量質問の `shouldFollowUp` はどの選択肢を選んでも深掘りするため、条件付きの深掘りは2アイテムに分けて表現する。

1. 定量質問（例: `SINGLE_SELECTION`）を `shouldFollowUp: false` で置く
2. その直後に `AI_INSTRUCTION` を置き、深掘りする条件・そこで聞くこと・条件に合わない場合はスキップすることを `instruction` に書く

`instruction` の例: 「前の質問で『不満』または『やや不満』を選んだ場合のみ、どの場面で不満を感じたかを具体的に聞いてください。それ以外の選択肢を選んだ場合は、この項目はスキップしてください。」

## MCP が使えない場合
MCP が設定されていない場合は、ユーザーに即座に伝えて MCP のみのセットアップを案内すること。

**1. APM の存在チェック**
`apm --version` を Bash で実行する。コマンドが見つからない場合のみ、以下のインストール手順をユーザーに案内する：

macOS / Linux
```bash
curl -sSL https://aka.ms/apm-unix | sh
```

Windows
```
irm https://aka.ms/apm-windows | iex
```

**2. Kikuvi MCP サーバーをインストール**
```bash
apm install --mcp kikuvi --transport http --url https://api.kikuvi.com/mcp
```
初回接続時にブラウザが開き、Kikuvi アカウントへの OAuth サインインが完了する。

**3. 使用しているクライアントに反映する**
```bash
apm compile --target <client>
```
`<client>` は使用している AI クライアントに合わせて指定する（例: `claude`, `codex`, `cursor`, `copilot`, etc.）

## Critical Rule: Kikuvi の API 仕様はモデルの記憶を信じない
Kikuvi の API は継続的に進化している。モデルの学習データは作成された時点で古くなっており、存在しないツールを呼び出したり廃止された引数を渡したりする原因になる。
- ツール名・引数・スキーマを記憶から決め打ちしない
- Tool を実行する前に必ず MCP でスキーマを確認する
- MCP からエラーが返ったときも、ツール名や引数を決め打ちせず MCP のスキーマを再確認してから修正する

## Guideline: 必要な情報はユーザーに質問して収集する
操作の目的・背景・対象者などTool Callに必要な情報を推測するのではなく、必要な情報はすべて対話形式でユーザーに質問して収集する。

## Checklist: 操作前チェックリスト
- 内部IDはユーザーに見せない — 常に名前で表示する
- Tool callに必要なID（designId, sessionId など）は list 系ツールで取得し、内部で保持する

## ヒアリング設計作成

### どちらのアプローチを使うか
| 状況 | アプローチ |
|------|-----------|
| デフォルト（基本的にはこちらを採用） | 対話的に設計 |
| ユーザーが「自動生成」「すぐ作って」「任せる」など自動生成を明示した場合のみ | AI 自動生成 |

### 対話的に設計
**MANDATORY: `references/interview-design-creation.md` を読みその内容に従うこと。** 

### AI 自動生成
ユーザーが明示的に自動生成を要求した場合のみ使用する。who / what / how / why などの情報を渡し、`design_generate` で AI がセクション・質問を自動生成する。
