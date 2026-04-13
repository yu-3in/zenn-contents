---
title: "Claude Code プラグインを作って、開発しながら語学学習できるようにした"
emoji: "🌍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["claudecode", "plugin", "english", "ai", "productivity"]
published: false
---

## はじめに

こんにちは、[yuuumin](https://x.com/yu_3in) です。

英語が必要なのはわかっている。アプリを入れる。3 ヶ月で辞める。年に一回「今年こそ」と思う。また辞める。このループに覚えがある人は多いんじゃないでしょうか。

自分もずっとそうでした。仕事が終わってから Duolingo を開いても、開発中のコンテキストとは全然違う例文を練習していて、正直あまり頭に入ってこない。かといって英語の技術記事を読んだり PR を英語で書いたりしても、それが正しいのかフィードバックをもらえる機会がない。

ほしかったのは英語教室じゃなくて、隣にネイティブの同僚がいて、自分の書いた文章をさりげなく直してくれる、そんな環境でした。

一方で、普段の開発では Claude Code に対して日本語で指示を出したり、たまに英語でコミットメッセージや PR の description を書いたりしている。ある日ふと、「この毎日打っているテキストをそのまま添削してもらえたら、ネイティブスピーカーと一緒に仕事しているみたいに自然と英語が身につくのでは？」と思いました。

そこで、Claude Code のプラグインを作りました。

https://github.com/yu-3in/claude-lang-coach

**lang-coach** というプラグインで、インストールすると Claude Code の応答の末尾に語学フィードバックが自動で付くようになります。

![Lang Coach の動作画面](/images/claude-lang-coach/autodetect-demo.png)
*日本語で指示を出すと、英語ではこう表現する、というフィードバックが自動で付く*

英語で書けば文法ミスを指摘してくれるし、日本語で書けば英語でどう言うかを教えてくれる。追加の操作は何も要りません。

## できること

### 自動添削モード

プラグインをインストールして言語ペアを設定すると、あとは普段通り Claude Code を使うだけです。

たとえば、こんな日本語で指示を出したとします：

```
私はITエンジニアです。日常的に語学学習に取り組もうと考えています。
おすすめの方法をいくつか紹介してください。
```

Claude はいつも通りタスクを完了します。そのあと、応答の末尾にこういうセクションが付きます：

```
✎ Lang Coach
| 日常的に語学学習に取り組もうと考えています
  → I'm thinking of making language learning part of my daily routine.
| おすすめの方法をいくつか紹介してください
  → Could you suggest some good approaches?
```

自分が書いた日本語が、そのまま英語の教材になります。

英語で指示を出した場合は、文法ミスがあれば指摘してくれます：

```
Please review this PR. I think the main changes is improving the performance.
```

```
✎ Lang Coach | ▲ `changes is` → `changes are` — 主語と動詞の一致（複数形）
```

間違いがなければ `✎ Lang Coach ✓` だけ。✓ が返ってくると地味に嬉しいです。

### 手動モード

自動添削は 1-2 行の簡潔なフィードバックですが、もう少し深く学びたいときは `/lang-coach:learn` コマンドが使えます。

```
/lang-coach:learn I goed to the store yesterday
```

```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃  正確さ      55  ███████████░░░░░░░░░  ┃
┃  流暢さ      65  █████████████░░░░░░░  ┃
┃  自然さ      70  ██████████████░░░░░░  ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃  ★ 総合      60  ████████████░░░░░░░░  ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

| | 原文 → 修正 | 理由 |
|:---:|-------------|------|
| ▲ | `goed` → `went` | go は不規則動詞 (go-went-gone) |

代替表現:
1. I stopped by the store yesterday — 「ちょっと立ち寄った」ニュアンス
2. I visited the store yesterday — ややフォーマル
```

4 軸のスコア、エラーの解説、代替表現まで返してくれます。日本語を入力すると翻訳モードに切り替わり、カジュアル・フォーマル・上級の 3 パターンで訳を出してくれます。

### 履歴の振り返り

```
/lang-coach:history
```

過去の添削をスコア付きで振り返れます。よくあるエラーのパターンや弱点の傾向分析もしてくれるので、自分の成長が見えるようになっています。

## 対応言語

英語に限らず、ISO 639-1 の任意の言語ペアに対応しています。

```
/lang-coach:setup ja en    # 日本語 → 英語
/lang-coach:setup ko en    # 韓国語 → 英語
/lang-coach:setup en ja    # 英語 → 日本語
/lang-coach:setup es fr    # スペイン語 → フランス語
```

## セットアップ

```bash
claude plugin marketplace add yu-3in/claude-lang-coach
claude plugin install lang-coach@claude-lang-coach
```

Claude Code を再起動して、言語ペアを設定します。

```
/lang-coach:setup ja en
```

これだけで、次の入力から自動添削が有効になります。

## 仕組み

Claude Code の[プラグインシステム](https://code.claude.com/docs/en/plugins)を使っています。専用のフレームワークやランタイムを追加インストールする必要はなく、Markdown と Shell Script（+ 軽量な Python3 スクリプト）で構成されています。

自動添削は `UserPromptSubmit` Hook で実現しています。ユーザーの入力が Claude に渡される前に、Shell Script が添削指示をコンテキストに注入します。

```bash
# autodetect.sh — Claude が受け取る指示を出力する
echo "MANDATORY: After your main response, append a Lang Coach section.
- en text: correct errors + suggest more natural phrasing
- ja text: translate naturally to en
..."
```

Claude はメインのタスクを処理した後、この指示に従って応答の末尾に添削を付けます。

なお、現在プラグインの `hooks.json` で定義した `UserPromptSubmit` の出力が Claude に注入されない[制限](https://github.com/anthropics/claude-code/issues/20659)があるため、`SessionStart` Hook でグローバル設定に自動登録する回避策を入れています。

:::message
プラグインの構成やアーキテクチャの詳細は [CONTRIBUTING.md](https://github.com/yu-3in/claude-lang-coach/blob/main/CONTRIBUTING.md) にまとめています。
:::

## おわりに

自分がこのループから抜けられたきっかけは、「勉強する時間を別に取る」のをやめたことでした。普段の開発中にネイティブの同僚が隣にいて、自分の書いた文章をその場で直してくれる。lang-coach はそんな感覚を Claude Code のプラグインで再現したものです。

使い始めてから変わったことがいくつかあります。PR の description を書くたびに「この英語で合ってるのかな」とずっと不安だったのが、毎回フィードバックが返ってくるようになって、少しずつ書くことへの抵抗が減ってきました。日本語で指示を出すたびに「英語だとこう言うのか」という気づきが自然に蓄積されていく感覚もあります。

何より大きいのは、「英語の勉強をしている」という意識がないことです。Claude Code を開いている時間がそのまま学習時間になっているので、サボるとか続かないとかいう概念がなくなりました。

もちろんまだ開発中で、改善したい点もたくさんあります。自動添削の結果を履歴に保存する機能（[#1](https://github.com/yu-3in/claude-lang-coach/issues/1)）や、プラグイン Hook の制限への対応など、やりたいことは尽きません。

OSS として公開しているので、Issue でも PR でもどしどしお待ちしています。「こういう機能がほしい」「このユースケースで使ってみた」など、フィードバックをいただけると嬉しいです。

https://github.com/yu-3in/claude-lang-coach

最後までお読みいただきありがとうございました！

https://x.com/yu_3in
