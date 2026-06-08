---
title: "Claude Codeユーザーのための Understand Anything vs CodeGraph 選び方ガイド"
emoji: "🧠"
type: "tech"
topics: ["AI", "Claude", "CodeGraph", "開発ツール", "エージェント"]
published: false
---

## はじめに

30万行のプロジェクトでClaude Codeに機能追加を依頼したら、grepを3回、globを2回、ファイルを6つ読んだ挙句「ロジックの場所がわかりません」と言われた。これがClaude Codeの根本的な弱点です——コードベースの構造を「知らない」のです。

この問題に対処するUnderstand AnythingとCodeGraph、2つのツールを比較します。

## 対象読者

- Claude Codeで大規模コードベースを扱うエンジニア
- AIエージェントのトークン消費を抑えたい方
- コード理解ツールの導入を検討しているチーム

## 2つのツールの本質的な違い

| | Understand Anything | CodeGraph |
|---|---|---|
| **誰のため** | 人間 | AIエージェント |
| **何をするか** | ブラウザで操作する可視化マップ | MCP経由でClaudeが裏で呼び出す索引 |
| **ネット接続** | 必要（LLM API） | 不要（ローカルSQLite） |
| **Claudeの認識** | なし（手動操作） | あり（自動認識） |

**要するに**：Understand Anythingは「人間が全体像を掴む」ツール。CodeGraphは「Claudeがgrepを減らす」ツール。競合しないので両方入れられます。

## Understand Anything ── 人間のためのコード地図

5〜7個の並列エージェントでコードベースを解析し、以下を生成：

- **アーキテクチャ図**：依存関係グラフ（クリック＆ドラッグ可能）
- **ドメインビュー**：コード→ビジネスフローのマッピング
- **学習パス**：新人向け自動ガイド
- **差分分析**：変更影響範囲の可視化
- **ナレッジベース分析**：Karpathy式Wikiの解析

**制約**：毎回LLM APIコスト発生。ファイル監視なし。Claude Codeからは独立している。

**適した場面**：オンボーディング、アーキテクチャレビュー、非エンジニア向け説明。

## CodeGraph ── Claude Codeに構造認識を

tree-sitterでAST解析→シンボル抽出→SQLite保存→MCP経由でClaudeが直接クエリ。

8つのMCPツール（`codegraph_explore`、`_search`、`_callers`、`_callees`、`_impact`、`_affected`、`_node`、`_status`）。

**CodeGraphだけの強み**：

1. **CI連携** — `codegraph affected`で変更ファイルに依存するテストだけ実行
2. **クロスランゲージ** — Swift/ObjC/JS/Native Module間の呼び出し追跡
3. **ゼロ設定** — `codegraph init -i`一発。OSネイティブファイル監視で自動同期

**ベンチマーク**：ツール呼び出し-58%、トークン-47%、コスト約-16%。

## 5つの実践シナリオ

1. **新人オンボーディング** → Understand Anything（15分で全体像把握）
2. **バグ修正のたびにClaudeが10ファイル読む** → CodeGraph（一発で探索終了）
3. **コアモジュールのリファクタリング** → 精度重視ならCodeGraph、プレゼン重視ならUnderstand Anything
4. **非エンジニアへの説明** → Understand Anything（ドメインビューが使える）
5. **React Nativeネイティブモジュールのバグ** → CodeGraph（クロスランゲージ追跡が唯一可能）

## インストール

### CodeGraph

```bash
# Windows
irm https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.ps1 | iex

# Mac/Linux
curl -fsSL https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.sh | sh

codegraph install && cd your-project && codegraph init -i
```

### Understand Anything

Claude Codeプラグインマーケットプレイスからインストール後、`/understand`を実行。

## まとめ

CodeGraphは常駐インフラとして全プロジェクトに入れる。Understand Anythingはアーキテクチャレビューやオンボーディングの前にオンデマンド実行。ひとつは財布を守り、もうひとつは脳を守る——両方使うのが合理的です。
