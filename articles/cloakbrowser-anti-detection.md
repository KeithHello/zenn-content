---
title: "CloakBrowser —— Chromium C++レベル改造でreCAPTCHA 0.9を達成した「検出不能ブラウザ」の全貌"
emoji: "🛡️"
type: "tech"
topics: ["AI", "ブラウザ", "スクレイピング", "オープンソース", "自動化"]
published: true
---

## はじめに

ブラウザ自動化の世界には、長年解決されなかった根本的な矛盾があります。**Webサイトはボットを検出したい。開発者はボットを動かしたい。** この綱引きの中で生まれたのが、JSインジェクションや設定パッチによる場当たり的な対策でした。

**CloakBrowser**（GitHub 22Kスター）は、この問題に対するアプローチを根本から変えました。Chromiumの**C++ソースコードレベルで58箇所の修正**を加え、検出不能な状態でコンパイルしたブラウザバイナリです。

本稿では、その技術的価値とアーキテクチャ、そしてAIエージェント時代における意義を掘り下げます。

## 対象読者

- Playwright / Puppeteer でブラウザ自動化を実装しているエンジニア
- Webスクレイピングのアンチボット対策に苦戦している方
- AIエージェント（browser-use / LangChain / Crawl4AI）の開発者

## CloakBrowserが解決する問題

従来のアプローチ（`playwright-stealth`、`undetected-chromedriver`）は、**JavaScriptレイヤーでのパッチ適用**に依存していました。しかし、アンチボットシステムはJSレイヤーの改変を検出する能力を年々高めており、Chromeのアップデートのたびにパッチが破壊されるという根本的な不安定さを抱えていました。

CloakBrowserは、**ChromiumのC++ソースコードそのものを改変し、バイナリとしてコンパイル**します。これにより：

1. JSレイヤーでの検出が不可能（改変されたJSコードが存在しない）
2. Chromeのアップデートに影響されない（独自ビルドのバイナリ）
3. TLS指紋レベルでの一致性（ja3n/ja4/akamaiすべてChromeと同一）

## 本文

### 1. 技術アーキテクチャ

CloakBrowserの中核は、Chromium 146をベースにした**58箇所のC++ソースコード修正**です。

修正対象領域：

| カテゴリ | 修正内容 |
|---|---|
| **ナビゲーターAPI** | `webdriver` フラグのハードコード無効化、`plugins`/`mimeTypes` の正規化 |
| **Canvas/WebGL** | レンダラー文字列の実GPU名への置換、ノイズ追加の除去 |
| **オーディオ** | AudioContextオシレーターの正規化、`OscillatorNode` の不一致解消 |
| **フォント** | システムフォント列挙の正規化 |
| **画面パラメータ** | 解像度・色深度の一貫性確保 |
| **ネットワークタイミング** | Resource Timing APIの異常値除去 |
| **CDP（Chrome DevTools Protocol）** | デバッガー検出シグナルの無効化 |
| **TLS指紋** | JA3/JA4/Akamai指紋の標準Chromeとの一致 |

この手法の優位性は明白です。JSインジェクションでは対処できない**TLS指紋やWebGLレンダラー情報**まで完全に制御できます。また、ブラウザバイナリが固定されているため、Chromeのアップデートでパッチが壊れるリスクもありません。

### 2. reCAPTCHA 0.9の実力

CloakBrowserの実際のベンチマーク結果は、競合との差を如実に示しています：

| 検出サービス | 通常Playwright | playwright-stealth | undetected-chromedriver | Camoufox | **CloakBrowser** |
|---|---|---|---|---|---|
| reCAPTCHA v3 | 0.1 | 0.3-0.5 | 0.3-0.7 | 0.7-0.9 | **0.9** |
| Cloudflare Turnstile | 失敗 | 時々 | 時々 | 通過 | **通過** |
| FingerprintJS | 検出 | 検出 | 検出 | — | **通過** |
| BrowserScan | 検出 | — | — | — | **正常(4/4)** |
| ShieldSquare | ブロック | — | — | — | **通過** |

reCAPTCHA v3の **0.9** は人間レベルのスコアです。これはサーバーサイドで検証された実際の値であり、クライアント側の改ざんではありません。

### 3. `humanize` — 行動エミュレーションの実装

指紋検出を回避しても、**行動分析**でボット判定されるケースは少なくありません。

CloakBrowserの `humanize=True` は、3つの入力次元すべてをカバーします：

- **マウス**: ベジェ曲線パスによる自然な軌道、ターゲット手前での減速、微細なジッターと行き過ぎ
- **キーボード**: 1文字ずつの入力、文字間・単語間に人間らしい「思考休止」を挿入
- **スクロール**: 加速→定速→減速の自然なリズム

これらの実装はPlaywrightのマウス・キーボード・スクロールAPIを**完全に置き換える**形で提供され、ヒューリスティックなラッパーではありません。

### 4. AIエージェント時代のブラウザインフラ

Anthropicの2026 Agentic Coding Trends Reportが示すように、AIエージェントは急速に本番環境へ進出しています。そして、それらのエージェントが実際に「仕事をする」ための**エントリーポイントはブラウザ**です。

フライト予約、フォーム入力、データ収集、自動テスト —— すべてブラウザを経由します。しかし、現状のWebインフラはAI駆動ブラウザをボットとしてブロックする設計になっています。

CloakBrowserは以下の主要AIエージェントフレームワークと統合済みです：

- **browser-use** (70K+ stars)
- **Crawl4AI** (58K+ stars)
- **LangChain** (100K+ stars)
- **Crawlee / Scrapling / Stagehand / Selenium**

さらに、**Browser Profile Manager**により、商用アンチ検出ブラウザ（Multilogin、GoLogin、AdsPower）のセルフホスト代替としても機能します。Dockerイメージ、Docker Compose、AWS Lambda対応も完備。

## 結論

CloakBrowserの22Kスターは、単なるバズではありません。**「JSレイヤーでの対症療法」から「ソースレベルでの根本的解決」へ** —— ブラウザ自動化のパラダイムシフトを示す明確なシグナルです。

AIエージェントが現実世界のWebと対話する時代において、CloakBrowserのような「エージェントネイティブなブラウザ」の重要性は増すばかりです。現在はChromiumベースですが、将来的にはFirefoxやWebKitベースのビルドも視野に入っているとのこと。

ブラウザ自動化の未来は、**検出を回避するハック**ではなく、**最初から検出されない基盤**を構築することにあります。CloakBrowserは、その未来を今日から使える形で提供しています。

---

## 参考リンク

- [CloakBrowser GitHub](https://github.com/CloakHQ/CloakBrowser)
- [CloakBrowser 公式ドキュメント](https://cloakbrowser.com)
- [Docker Hub](https://hub.docker.com/r/cloakhq/cloakbrowser)
