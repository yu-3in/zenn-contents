---
title: "Tailwind CSSで数字の幅を揃える"
emoji: "🔨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["css"]
published: true
---

Tailwind CSS では、独自のリセット CSS の [Preflight](https://tailwindcss.com/docs/preflight) の影響で数字の幅が揃わないという現象が発生することがあります。

今回は、その一例と解決方法を紹介します。

# 例えばこのようなもの

@[codepen](https://codepen.io/yuuumiravy/pen/VwXEpmE?open_file=index.html)

上記のように、`1` と `0` の幅が異なるため、これを揃えます。

# 解決策

Tailwind CSS で用意されている `tabular-nums` というクラスを用います。

@[codepen](https://codepen.io/yuuumiravy/pen/oNqaZZP)

なお、このクラスのプロパティは、

```css
--tw-numeric-spacing: tabular-nums;
```

です。

そのため、ページ全体対して適用したい場合は、個別に設定するのではなく、 `tabular-nums` クラスを html タグや body タグに設定することもできます。
その場合のコードは、下記のようなものです。

```html
<html lang="ja" class="tabular-nums"></html>
<!-- または -->
<body class="tabular-nums"></body>
```

なお、[Tailwind CSS の公式ドキュメント](https://tailwindcss.com/docs/font-variant-numeric)には、このクラスのプロパティは、

```css
font-variant-numeric: tabular-nums;
```

であると書かれています。
しかし、実際のプロパティとは異なっているため、この記述はおそらく「`font-variant-numeric: tabular-nums;`と同様のはたらきをするクラスである」という意味合いなのでしょう。

`font-variant-numeric` についての詳細は、[MDN | font-variant-numeric](https://developer.mozilla.org/ja/docs/Web/CSS/font-variant-numeric) をご覧ください。
