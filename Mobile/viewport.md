# VIEWPORT

## 不设置viewport
在header中，不设置```meta```，这样，会让浏览器使用默认的屏幕宽度，在iphone上是980px。

## viewport的宽度为固定
``` html
<meta name="viewport" content="width=320">
```
在不同的设备上，分辨率都设置成320，这样就是相同的元素在不同机型上，会相应的放大。
> 如果写了initial-scale=1.0，那么在不同的设备上行为不一样，浏览器会取320和ideal viewport中较大的一个值。

## viewport的宽度为设备默认宽度
``` html
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no ,maximum-scale=1.0, minimum-scale=1.0">
```
设置viewport的宽度为device的宽度（ideal viewport)。iphone6是350，iphone6p是414。
在这种情况下可以全部元素使用px，就像weui那样。同样大小的元素，文字，在不同的屏幕分辨率下大小是一样的。

## viewport的宽度通过dpr来计算
``` javascript
<script>
  (function (baseFontSize, fontscale) {
      var _baseFontSize = baseFontSize || 100;
      var _fontscale = fontscale || 1;
      var win = window;
      var doc = win.document;
      var ua = navigator.userAgent;
      var matches = ua.match(/Android[\S\s]+AppleWebkit\/(\d{3})/i);
      var UCversion = ua.match(/U3\/((\d+|\.){5,})/i);
      var isUCHd = UCversion && parseInt(UCversion[1].split('.').join(''), 10) >= 80;
      var isIos = navigator.appVersion.match(/(iphone|ipad|ipod)/gi);
      var dpr = win.devicePixelRatio || 1;
      if (!isIos && !(matches && matches[1] > 534) && !isUCHd) {
        // 如果非iOS, 非Android4.3以上, 非UC内核, 就不执行高清, dpr设为1;
        dpr = 1;
      }
      var scale = 1 / dpr;

      var metaEl = doc.querySelector('meta[name="viewport"]');
      if (!metaEl) {
        metaEl = doc.createElement('meta');
        metaEl.setAttribute('name', 'viewport');
        doc.head.appendChild(metaEl);
      }
      metaEl.setAttribute('content', 'width=device-width,user-scalable=no,initial-scale=' + scale + ',maximum-scale=' + scale + ',minimum-scale=' + scale);
      doc.documentElement.style.fontSize = _baseFontSize / 2 * dpr * _fontscale + 'px';
      window.fontSizeNum = _baseFontSize / 2 * dpr * _fontscale;
      window.viewportScale = dpr;
    })();
</script>
```
在不同的机型上，visual viewport的宽度为ideal viewport ＊ dpr。在iphone6上为350 ＊ 2 ＝ 700， 在iphone6p上为414 ＊ 3 ＝ 1242。
在这种情况下，如果还继续使用px，会导致相应的px非常小。
这种情况下适合使用rem。上面的代码，在iphone6上，1rem＝100px，在iphone6p上1rem＝150px。这样可以保证相同的rem，在不同分辨率下的大小一样。
另外，这种情况下还有一个好处就是px的绝对大小是一致的。
### 媒体查询
在这种情况下，媒体查询的宽度，是乘了dpr的宽度。
在iphone4上:
``` css
@media (max-width: 640px) {
  font-size: 0.24rem;

  .details {
    font-size: 0.24rem;
  }
}
```
