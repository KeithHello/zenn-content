---
title: "Understand-Anything：コードベースを教える知識グラフに変える45.9kスターのツール"
emoji: "🧠"
type: "tech"
topics: ["AI", "Claude", "エージェント", "開発ツール", "知識グラフ"]
published: false
---

## はじめに

新しい職場の初日に渡された20万行のコードベース。マネージャーは「今週は慣れるのに使って」と言うけれど、grep と IDE タブの海で溺れるだけ——そんな経験はありませんか。

ほとんどのコード可視化ツールは「見た目が良い」グラフを作ることに集中してきました。派手なデモには映えるけれど、実際に理解しようとすると圧倒されるだけです。

Georgia Tech で LLM マルチエージェント協調を研究する Lum1104 氏は、この問題に正面から取り組みました。彼のプロジェクトのタグラインはこうです：

> **「教えるグラフ ＞ 魅せるグラフ」**

## 対象読者

- 新規プロジェクトのコードベースを迅速に把握したい開発者
- 複数リポジトリを横断して作業するシニアエンジニア
- チームオンボーディングの効率化を目指すテックリード
- Claude Code、Codex、Cursor など AI コーディングツールのヘビーユーザー

## Understand-Anything のコア概念

Understand-Anything は、あらゆるコードベースを**対話型知識グラフ**に変換する Claude Code ネイティブプラグインです。17 のコーディングプラットフォームに対応し、現在 **45.9k Stars** を獲得しています。

### 設計哲学：「一度スキャン、何度も再利用」

従来のコード理解プロセスは毎回 LLM にソースを再読込させる「リアクティブ」なモデルでした。Understand-Anything はこれを反転させ、プロジェクト全体を単一の JSON 知識グラフにあらかじめ圧縮します。

この「事前計算コンテキスト」アプローチは類似プロジェクトで **コードレビュー：6.8 倍、日常コーディング：49 倍**のトークン節約を達成したと報告されています。

## アーキテクチャ：決定論と意味理解の融合

### Tree-sitter：構造の土台

ソースコードを具象構文木にパースし、インポート、エクスポート、関数・クラス定義、コールサイト、継承関係を抽出。同一入力には常に同一出力。フィンガープリントによる変更検出で差分更新も実現。

### LLM：意味のレイヤー

パースされた構造×元ソースから、平易な英語サマリ、タグ、アーキテクチャ層割り当て、ビジネスドメインマッピングを生成。

このハイブリッド設計の魅力は再現性と柔軟性の両立にあります。構造は決定的、意味は適応的。

### 7 つの専門エージェント

| エージェント | 責務 |
|--------------|------|
| `project-scanner` | ファイル発見、言語・フレームワーク検出 |
| `file-analyzer` | 機能抽出、グラフノード・エッジ構築（最大5並列・20-30ファイル/バッチ） |
| `architecture-analyzer` | API/Service/Data/UI/Utility の層識別 |
| `tour-builder` | 依存順の学習パス自動生成 |
| `graph-reviewer` | 完全性・参照整合性の検証 |
| `domain-analyzer` | ビジネス領域・フロー抽出 |
| `article-analyzer` | ナレッジ記事からのエンティティ抽出 |

## 主要機能

### 1. `/understand-onboard` — ガイド付きツアー

依存関係順にソートしたアーキテクチャ全体のウォークスルーを自動生成。新メンバーにコードの「地図」だけでなく「通り道」を提供します。

### 2. `/understand-diff` — 影響範囲マッピング

git diff の変更箇所を知識グラフ上にマッピングし、変更が影響するシステム全体を可視化。コードレビュー前のリスク確認に。

### 3. `/understand-domain` — ビジネス視点

コードをビジネスプロセスとして読み解く機能。API エンドポイントが請求処理のどのステップにあたるか——技術とビジネスの翻訳レイヤー。

### 4. ファジー＆セマンティック検索

名前検索だけでなく、「認証周りを処理しているのはどこ？」といった意味検索にも対応。

## インストール

```bash
# Claude Code
/plugin marketplace add Lum1104/Understand-Anything
/plugin install understand-anything

# ワンライナー（Codex / OpenCode / Gemini CLI 他）
curl -fsSL https://raw.githubusercontent.com/Lum1104/Understand-Anything/main/install.sh | bash
```

対応プラットフォーム：Claude Code, Cursor, VS Code + GH Copilot, Copilot CLI, Codex, OpenCode, OpenClaw, Antigravity, Gemini CLI, Pi Agent, Vibe CLI, Hermes, Cline, KIMI CLI, Trae（全 17）

## このツールの「向かない」領域

Understand-Anything はコード生成ツールでも、自動リファクタリングツールでもありません。コードを理解するためのツールです。以下のようなシーンで最も価値を発揮します：

- 新規参画時のコードベース把握
- レビュー前の影響範囲確認
- チーム全体で共有可能な「コードの地図」の構築

## まとめ

「一度スキャンして何度も再利用する」——この哲学は、AI コーディングが標準化する時代において、コードを「書く」より「理解する」ことの方がボトルネックになる未来を先取りしています。45.9k Stars という数字は、その証左です。

---

**GitHub:** [github.com/Lum1104/Understand-Anything](https://github.com/Lum1104/Understand-Anything)  
**ライセンス:** MIT | **最新:** v2.7.3（2026年5月） | **作者:** Yuxiang Lin（Georgia Tech）
