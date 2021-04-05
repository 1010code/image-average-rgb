# 透過 JavaScript 取得照片中的主題色
## 前言
本篇文章提供兩種方法透過 JavaScript 讀取 HTML 中的 `<img/>` 影像並取得影像中的平均色作為背景顏色。成果如下：

![](./screenshot/img01.png)

## 方法一
此方法瀏覽器必須要支援 `canvas`。我們透過讀取前端網頁中的 `<img/>` 標籤，並將影像複製到一個 `canvas` 上。接著將三個 channel 的 RGB 計算平均的 Pixel 數值。其中我們可以設定 `blockSize` 代表每多少個 Pixel 取一次數值做累加。

```js
function getAverageRGB(imgEl) {
    var blockSize = 5, // only visit every 5 pixels
        defaultRGB = {r:0,g:0,b:0}, // for non-supporting envs
        canvas = document.createElement('canvas'),
        context = canvas.getContext && canvas.getContext('2d'),
        data, width, height,
        i = -4,
        length,
        rgb = {r:0,g:0,b:0},
        count = 0;

    if (!context) {
        return defaultRGB;
    }
    height = canvas.height = imgEl.naturalHeight || imgEl.offsetHeight || imgEl.height;
    width = canvas.width = imgEl.naturalWidth || imgEl.offsetWidth || imgEl.width;
    context.drawImage(imgEl, 0, 0);

    try {
        data = context.getImageData(0, 0, width, height);
    } catch(e) {
        /* security error, img on diff domain */
        return defaultRGB;
    }
    length = data.data.length;

    while ( (i += blockSize * 4) < length ) {
        ++count;
        rgb.r += data.data[i];
        rgb.g += data.data[i+1];
        rgb.b += data.data[i+2];
    }
    // ~~ used to floor values
    rgb.r = ~~(rgb.r/count);
    rgb.g = ~~(rgb.g/count);
    rgb.b = ~~(rgb.b/count);

    return rgb;
}
```

使用方式是讀取前端一張照片後，直接呼叫 `getAverageRGB()` 即可得到 RGB 色彩。

```js
const img = document.querySelector('img');
const rgb = getAverageRGB(img);
document.body.style.backgroundColor = 'rgb('+rgb.r+','+rgb.g+','+rgb.b+')';
```

## 方法二 (採第三方套件)
這裡我們使用開源的 [COLOR THIEF](https://lokeshdhakar.com/projects/color-thief/) 函式庫，在 GitHub 有提供 CDN 與 NPM 下載方式。

### CDN
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/color-thief/2.3.0/color-thief.umd.js"></script>
```

js 程式碼
```js
const colorThief = new ColorThief();
const img = document.querySelector('img');

// Make sure image is finished loading
if (img.complete) {
    colorThief.getColor(img);
} else {
    image.addEventListener('load', function() {
    colorThief.getColor(img);
    });
}
```


## Reference 
- [stackoverflow](https://stackoverflow.com/questions/2541481/get-average-color-of-image-via-javascript)
- [COLOR THIEF](https://lokeshdhakar.com/projects/color-thief/)