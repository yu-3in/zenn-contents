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

なお、このクラスの実態は、

```css
font-variant-numeric: tabular-nums;
```

となります。この CSS プロパティについての詳細は、[MDN | font-variant-numeric](https://developer.mozilla.org/ja/docs/Web/CSS/font-variant-numeric) をご覧ください。

@[codepen](https://codepen.io/yuuumiravy/pen/oNqaZZP)
