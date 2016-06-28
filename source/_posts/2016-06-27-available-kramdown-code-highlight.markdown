---
layout: post
title: "Rdiscountからkramdownへmarkdownのparserを変更した"
date: 2016-06-27 16:01:52 +0900
comments: true
categories: 
description: "数式を扱うために前回の記事で、markdownをkramdownに変更したのですが、コードのシンタックスハイライトが効かなくなってしまいました。kramdownのparserをGFMにしてやるとうまくいきました。GFMを使わない場合は、プラグインがあるのでそちらでも対応できます。"
---

数式を扱うために[前回の記事](/blog/2016/06/27/use-math-syntax-on-jekyll/)で、markdownをkramdownに変更したのですが、コードのシンタックスハイライトが効かなくなってしまいました。

kramdownのparserをGFMにしてやるとうまくいきました。

_config.yml

```
kramdown:
  input: GFM
```

GFMを使わない場合は、プラグインがあるのでそちらでも対応できます。

[kramdown-with-pygments](https://github.com/mvdbos/kramdown-with-pygments)

使い方ですが、pluginsディレクトリにkramdown_pygments.rbを置きます。  
そして、`_config.yml`のmarkdownを以下のように修正します。

```
markdown: KramdownPygments
```


また、parserにkramdownを使う場合はRdiscountとkramdownでcodeのmarkdownのフォーマットが違うのでgenerate時にエラーがでるかもしれません。

自分は以下のワンライナーで書き換えました。

```
find source/_posts -type f -name "*.markdown" | xargs sed -i '' -e 's/\`\`\`/\
~~~/g'
```
