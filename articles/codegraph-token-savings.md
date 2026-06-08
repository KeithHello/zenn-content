---
title: "CodeGraph で AI コーディングのトークン消費を 47% 削減する方法"
emoji: "🧠"
type: "tech"
topics:
  - "AI"
  - "Claude"
  - "CodeGraph"
  - "MCP"
  - "Agentic"
published: true
---

## はじめに

Claude Code や Cursor でコードの質問をするとき、Agent は毎回 `grep` → `read` → `grep` のループを繰り返しています。この「探索」だけで、**トークン消費の 50〜70%** が費やされているのをご存知でしょうか。

CodeGraph は、プロジェクトのコードを事前に知識グラフとしてインデックス化し、Agent が必要な情報を一発で取得できるようにするツールです。GitHub 43K スター、MIT ライセンス、100% ローカル動作。7 つの OSS プロジェクトでの実測では、**平均 47% のトークン削減、58% のツール呼出削減、16% のコスト削減**を達成しています。

---

## 対象読者

- Claude Code / Cursor / Codex のヘビーユーザー
- 中〜大規模プロジェクト（500+ ファイル）で AI コーディングしている開発者
- トークンコストを削減したいチームリーダー

---

## CodeGraph がもたらす 3 つのメリット

1. **トークン消費の大幅削減** — 平均 47%、最大 64%（VS Code 10,000 ファイル）
2. **ツール呼出の最小化** — 平均 58% 減、ファイル読み取りがほぼゼロに
3. **ゼロコスト運用** — ローカル SQLite、API 不要、外部通信なし

---

## CodeGraph とは？

CodeGraph は、あなたのコードベースを **知識グラフ（Knowledge Graph）** として構造化する CLI ツールです。

仕組みは 4 層：

1. **ソースコードスキャン** — プロジェクトを読み込み、`node_modules` などは自動除外
2. **tree-sitter AST 解析** — 20+ 言語の文法を理解した正確なパーサーで関数・クラス・メソッドを抽出
3. **SQLite 知識グラフ** — Nodes（シンボル）と Edges（呼出/import/継承）を `.codegraph/codegraph.db` に保存。FTS5 全文検索付き
4. **MCP サーバー** — Agent に 8 つのクエリツールを提供

grep との本質的な違いは、grep が「文字列がどこにあるか」しか教えないのに対し、CodeGraph は「**この関数が誰に呼ばれ、誰を呼び、変更するとどこに影響するか**」まで教えることです。

---

## ベンチマーク結果

Claude Opus 4.8 を使用。同一のアーキテクチャ質問を CodeGraph あり/なしで各 4 回実行、中央値を比較：

| プロジェクト | 言語 | トークン削減 | ツール呼出削減 | コスト削減 |
|-------------|------|------------|--------------|-----------|
| VS Code | TS | -64% | -81% | -18% |
| Alamofire | Swift | -64% | -58% | -40% |
| Django | Python | -60% | -77% | -8% |
| OkHttp | Java | -54% | -50% | -25% |
| Tokio | Rust | -38% | -57% | ±0% |
| Gin | Go | -23% | -44% | -19% |
| Excalidraw | TS | -25% | -40% | ±0% |

VS Code の詳細が特に印象的です：

| 指標 | なし | あり | 削減 |
|------|------|------|------|
| ファイル読み取り | 9 回 | 0 回 | -9 |
| grep | 11 回 | 0 回 | -11 |
| ツール呼出合計 | 21 回 | 4 回 | -81% |
| トークン | 179 万 | 64 万 | -64% |

---

## 導入手順

```bash
# 1. インストール（Node.js 不要）
curl -fsSL https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.sh | sh

# 2. Agent に接続（Claude Code, Cursor, Codex などを自動検出）
codegraph install

# 3. プロジェクトを初期化 + インデックス構築
cd your-project
codegraph init -i

# 4. Agent を再起動 — 完了！
```

インストール後、Agent にアーキテクチャ質問を投げてみてください。ツール呼出が `grep` から `codegraph_explore` に変わっているはずです。

---

## 適しているプロジェクト

- 500 ファイル以上のプロジェクト（コードベースが大きいほど効果が高い）
- クロス言語プロジェクト（Swift↔ObjC、React Native）— grep では追えない言語境界を横断可能
- CI/CD のテスト最適化（`codegraph affected` で変更の影響を受けるテストのみ実行）

## 不要なケース

- 50 ファイル未満の小規模プロジェクト
- ChatGPT Web のみを使用（MCP 非対応）
- シンプルな CRUD 作業のみ

---

## まとめ

CodeGraph の本質は「**探索をクエリに置き換える**」ことです。Agent が毎回ファイルを grep する代わりに、事前に構築した知識グラフにクエリを投げる — それだけでトークン消費が平均 47% 削減されます。

3 分のセットアップで、永続的なコスト削減。ランニングコストはゼロ。プライバシーの懸念もゼロ。

AI コーディングのインフラとして、ぜひ導入を検討してみてください。

---

**リンク：**
- GitHub: [colbymchenry/codegraph](https://github.com/colbymchenry/codegraph)
- ドキュメント: [colbymchenry.github.io/codegraph](https://colbymchenry.github.io/codegraph/)
