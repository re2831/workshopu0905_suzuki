# 気仙沼市 企業誘致AIアシスタント — 開発ドキュメント

## 概要

気仙沼市の職員が企業誘致業務で使うAIアシスタント。企業名を入力すると、市のナレッジデータベースと照合して相性スコア・シナジー・アプローチメールを自動生成する。

| 項目 | 内容 |
|---|---|
| 公開URL | https://re2831.github.io/workshopu0905_suzuki/ |
| リポジトリ | https://github.com/re2831/workshopu0905_suzuki |
| 実装形式 | 素のHTML/CSS/JS（1ファイル完結）|
| データ保存 | localStorage |
| AIモデル | Claude API（Haiku / Sonnet / Opus 選択可）|

---

## 実装した機能

### 1. ダッシュボード / 検索画面
- 企業名・証券コードの入力フォーム
- 分析件数・高相性企業数・平均スコアの統計カード
- 最近の分析結果カード一覧（クリックで詳細へ）
- APIキー設定状態の表示（AI分析 / デモ分析の切り替え案内）

### 2. 分析プロセス（3ステップアニメーション）
1. 外部情報を収集中（IR・ニュース・企業Webサイト）
2. 気仙沼市ナレッジDBを照合中（RAG）
3. AIが相性を判定・提案文を生成中

### 3. 分析結果画面
- **左カラム**: 5段階スコア（★表示・色分け）、主要シナジー3件、マッチした市のアセットタグ、庁内共有用サマリー
- **右カラム**: アプローチメールのリアルタイム編集エディタ（コピー・印刷ボタン付き）

### 4. ナレッジ管理画面
- 市の強み・施策・課題の追加 / 編集 / 削除
- カテゴリ別バッジ表示（水産業・観光・再生エネ・優遇制度・地域課題・インフラ）
- デフォルトナレッジへのリセット機能

### 5. 設定画面
- Claude APIキー入力（localStorageに保存）
- 使用モデル選択（Haiku / Sonnet / Opus）
- 全データリセット

---

## 技術スタック（フロントエンドデモ版）

```
index.html（1ファイル）
├── CSS（インライン）
│   ├── CSS カスタムプロパティでカラー管理
│   ├── Flexbox / CSS Grid でレスポンシブレイアウト
│   └── @media (max-width: 720px) でモバイル対応
└── JavaScript（インライン）
    ├── IIFE（即時実行関数）でモジュール化
    ├── localStorage でデータ永続化
    ├── イベント委譲（document.addEventListener）
    └── Claude API（fetch）による AI分析
```

### ブラウザから Claude API を直接呼び出す

```javascript
fetch('https://api.anthropic.com/v1/messages', {
  method: 'POST',
  headers: {
    'x-api-key': apiKey,
    'anthropic-version': '2023-06-01',
    'anthropic-dangerous-direct-browser-access': 'true'  // CORS許可ヘッダー
  },
  body: JSON.stringify({ model, max_tokens: 3000, messages })
})
```

---

## AI分析の仕組み

### APIキーあり → Claude API分析

```
企業名
  ↓
[プロンプト生成]
  ・気仙沼市ナレッジ全件をテキスト化して挿入
  ↓
[Claude API呼び出し]
  ↓
[JSONパース]
  affinityScore / keySynergies / matchedAssets / approachLetter / executiveSummary
  ↓
[localStorage保存 → 画面表示]
```

### APIキーなし → キーワードマッチングによるデモ分析

企業名に含まれるキーワードを7パターンと照合してスコア・シナジーを決定:

| パターン | キーワード例 | スコア |
|---|---|---|
| 水産 | 水産・海・魚・養殖・漁業 | 5 |
| エネルギー | 再生可能・カーボン・ESG・脱炭素 | 4 |
| DX・IT | DX・AI・IoT・ロボット・自動化 | 4 |
| 観光 | 観光・ツーリズム・ホテル・宿泊 | 4 |
| 食品 | 食品・農業・フード・加工食品 | 4 |
| 物流 | 物流・輸送・ロジスティクス | 3 |
| 医療・福祉 | 医療・健康・介護・ヘルスケア | 3 |
| その他 | （マッチなし） | 3 |

---

## データ設計（localStorage）

| キー | 内容 | 型 |
|---|---|---|
| `kes_analyses` | 分析履歴の配列 | Array\<Analysis\> |
| `kes_knowledge` | ナレッジエントリの配列 | Array\<Knowledge\> |
| `kes_settings` | APIキー・モデル設定 | Object |

### Analysis オブジェクト

```json
{
  "id": "1725500000000",
  "companyName": "株式会社水産テクノロジー",
  "stockCode": "9999",
  "createdAt": "2026-09-05T00:00:00.000Z",
  "affinityScore": 5,
  "keySynergies": ["シナジー1", "シナジー2", "シナジー3"],
  "matchedAssets": ["水産加工集積地区施設", "企業誘致補助制度"],
  "approachLetter": "...アプローチメール本文...",
  "executiveSummary": "## サマリー...",
  "isDemo": false
}
```

### デフォルトナレッジ（6件）

| カテゴリ | タイトル |
|---|---|
| 水産業 | 水産加工集積地区施設 |
| 再生エネ | 未利用バイオマス資源活用 |
| 優遇制度 | 企業誘致補助制度 |
| 観光 | 三陸ジオパーク・インバウンド観光 |
| 地域課題 | 水産業の人手不足・高齢化 |
| インフラ | 気仙沼港湾設備・物流基盤 |

---

## 画面フロー

```
ダッシュボード
  │
  ├─ [企業名入力 → 分析開始]
  │     ↓
  │   プログレスアニメーション（3ステップ）
  │     ↓
  │   分析結果画面
  │     ├─ アプローチメール編集・コピー・印刷
  │     └─ 庁内サマリーコピー
  │
  ├─ [ナレッジ管理]
  │     ├─ ナレッジ追加（モーダル）
  │     ├─ ナレッジ編集（モーダル）
  │     └─ ナレッジ削除
  │
  └─ [設定]
        ├─ APIキー・モデル保存
        ├─ ナレッジリセット
        └─ 全データリセット
```

---

## 今後の展開（フルスタック版）

`architecture.txt` に記載された本番構成:

| レイヤー | 技術 | 役割 |
|---|---|---|
| フロントエンド | Next.js + TypeScript + Tailwind CSS | 現在のデモUIを移植 |
| バックエンドAPI | Python / FastAPI | 非同期処理・外部API連携 |
| データベース | PostgreSQL + pgvector（Supabase） | ナレッジのベクトルストア |
| 外部データ連携 | EDINET API・Google Custom Search | IR情報・ニュース自動収集 |
| AI / LLM | Claude API + text-embedding-3-large | 埋め込み生成・相性判定 |

### フェーズ1で追加予定
- EDINET APIによる有価証券報告書の自動取得（事業の状況・課題抽出）
- pgvector によるナレッジのコサイン類似度検索（本格RAG）
- `POST /api/v1/analysis/company` エンドポイント実装

---

## 開発メモ

- GitHub Pages はサブパス配下でホストされるため、絶対パス（`/style.css`）は禁止。すべて相対パス（`./`）を使用
- Claude API はブラウザから直接呼び出し可能（`anthropic-dangerous-direct-browser-access: true` ヘッダーが必要）
- APIキー未設定時もデモとして完全に動作するよう設計
- ビルドツール不使用。npm/バンドラなし。ブラウザでそのまま動く素のHTMLのみ
