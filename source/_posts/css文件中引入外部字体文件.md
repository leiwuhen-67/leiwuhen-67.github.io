---
title: css文件中引入外部字体文件
date: 2021-11-17 13:21:56
tags:
---
在项目中经常会遇到需要引入外部字体文件的问题。。这里有个奇怪的问题，先记录下，问题解决了，但是不知道为啥会这样:
在vue-cli3.0中像这样引入完全正常。
```language
@font-face {
    font-family: myFont;
    src: url("./DINOffcPro-Black.ttf");
}

```
但是在vue-cli2.0中像上面那样引入的话，如果是引入.otf文件的话是正常的，如果是引入.ttf文件的话会报错，这里，如果引入.ttf文件的话，需要像如下方式引入：
```language
@font-face {
    font-family: myFont;
    src: url-loader("./DINOffcPro-Black.ttf");
}
```