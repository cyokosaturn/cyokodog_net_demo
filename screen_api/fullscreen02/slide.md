# Fullscreen API

- [https://www.w3.org/TR/fullscreen/](https://www.w3.org/TR/fullscreen/)
- JavaScriptにて全画面表示への切り替えができるAPI
- ブラウザの全画面表示と異なり、指定した要素のみを全画面表示できる
- ユーザーの操作をトリガーに全画面表示する
  - （画面ロード後いきなり全画面表示とかできない）
- YouTubeの全画面表示で利用されてる

## 実装状況

- [Can I use... Support tables for HTML5, CSS3, etc](http://caniuse.com/#search=fullscreen)
- 全ブラウザ対応済み
  - ベンダープレフィックスは必要
  - メソッド名自体がベンダー間で異なるものも...
- モバイル環境は、Chrome for Androidのみ

## API

- requestFullscreen()
  - fullscreen表示する
  - Firefox: mozRequestFullScreen()
  - Safari/Chrome/Opera: webkitRequestFullScreen()
  - IE: msRequestFullscreen
- exitFullscreen()
  - fullscreen表示の解除
  - Firefox: mozExitFullScreen()
  - Safari/Chrome/Opera: webkit**CancelFullScreen**()
  - IE: msExitFullscreen()
- fullscreenEnabled
  - fullscreenをサポートしてるか
  - Firefox: mozFullscreenEnabled
  - Safari/Chrome/Opera: webkitFullscreenEnabled
  - IE: msFullscreenEnabled
- fullscreenElement
  - fullscreen表示されてる要素
  - Firefox: mozFullscreenElement
  - Safari/Chrome/Opera: webkitFullscreenElement
  - IE: msFullscreenElement


## Event

- fullscreenchange
  - fullscreen表示/解除時に発火
  - Safari/Chrome/Opera: webkitfullscreenchange
  - Firefox: mozfullscreenchange
  - IE: **MSFullscreenChange**
    - MSだけキャメルケース


## CSS Selector

- :fullscreen
  - fullscreen表示されてる要素に適用
  - fullscreen表示時のみ、fullscreen対象要素や内包要素に適用したいスタイルがある場合使用するとよさそう
  - Firefox: -moz-full-screen
  - Safari/Chrome/Opera: -webkit-full-screen
  - IE: -ms-fullscreen
- :fullscreen-ancestor
  - fullscreen表示されてる要素の先祖要素に適用
  - fullscreen表示時のみ、fullscreen対象要素外にある要素（メニューとか）に適用したいスタイルがある場合使用するとよさそう

## User-agent level style sheet defaults

```css
*|*:not(:root):fullscreen {
  position:fixed !important;
  top:0 !important; right:0 !important; bottom:0 !important; left:0 !important;
  margin:0 !important;
  box-sizing:border-box !important;
  min-width:0 !important;
  max-width:none !important;
  min-height:0 !important;
  max-height:none !important;
  width:100% !important;
  height:100% !important;
  transform:none !important;

  /* intentionally not !important */
  object-fit:contain;
}

iframe:fullscreen {
  border:none !important;
  padding:0 !important;
}

::backdrop {
  position:fixed;
  top:0; right:0; bottom:0; left:0;
}

*|*:not(:root):fullscreen::backdrop {
  background:black;
}
```

## 書き方

最大化

```javascript
document.querySelector('.bar').requestFullScreen();
```

解除

```javascript
document.exitFullScreen();
```

## いろんな要素をfullscreenしてみる

Chromeでテスト（他のブラウザと挙動が異なるかも）

- video、img要素の場合のみ(?)`width:100%`,`height:100%`が適用される
  - 2016/09/16時点では、chromeの場合サイズ変更されなかったけど実装が変わった模様
  - [Fullscreen API をさわったのでメモ - Qiita](http://qiita.com/cyokodog@github/items/78998b7a3dbe07065712)
- 対象要素が縦横中央にセンタリングされる
  - object-fit:contain;
- 対象要素が画面表示領域より大きい場合は見切れる（スクロール不可）

## Chromeのバグ：img要素、inline-block要素のカラム落ちが発生

- img要素、inline-block要素をfullscreenすると解除時に絡む落ちする
- fullscreenに`inline-block`であることを明示すると起きない

```
img:-webkit-full-screen{
  display: inline-block;
}
```

## fullscreen中もメニューを表示する

`position:fixed`と`z-index`でfullscreen要素より前面へ表示する

**ただし、DOM構成上、fullscreen要素より後方に配置してないと表示されないっぽい...**

```css
:-webkit-full-screen-ancestor .tools{
  position: fixed;
  right:0;
  top:0;
  z-index: 2147483647;
  & select {
    display: none;
  }
  & fullscreen {
    display: none;
  }
  & exit {
    display: inline;
  }
}
```


## 横並びの画像をfullscreenする場合

画面幅全体を使うようにすると良さそう

```css
/* 土台を広げて */
.imageGroup:-webkit-full-screen{
  width: 100%;
  position:absolute;
  left:0;
}
/* 画像幅を等分 */
.imageGroup:-webkit-full-screen img{
  width:25%;
}
```

## 見切れてしまうコンテンツをfullscreenする場合

コンテンツの先頭から表示して、スクロールで全体を読めるようにすると良さそう

```css
/* 先頭行に位置合わせしてスクロールで全体を読めるようにする */
.longText:-webkit-full-screen{
  padding: 0 10%;
  position: fixed;
  top: 0;
  left: 0;
  width: 80%;
  height: 100%;
  font-size: 200%;
  overflow: scroll;
}
```

