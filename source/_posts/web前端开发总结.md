---
title: web前端开发总结
date: 2020-06-13 18:37:52
categories: web前端开发
tags:
---
这些都是日常开发中使用频率较高的，因此封装成方法以便随时能调用，比如判断是否是微信环境、判断当前设备是哪种设备、localStorage和sessionStorage的封装等等。
#### 1、判断是否为微信浏览器：
```javascript
function isWeiXin() {
  var ua = window.navigator.userAgent.toLowerCase();
  if (ua.match(/MicroMessenger/i) == 'micromessenger') {
      return true;
  } else {
      return false;
  }
}
```
#### 2、判断当前设备是哪种设备，如ios、安卓、移动端等等。
<!-- more -->
```javascript
var browser = {
  versions: function () {
    var u = navigator.userAgent, app = navigator.appVersion;
    return {
      trident: u.indexOf('Trident') > -1, //IE内核                
      presto: u.indexOf('Presto') > -1, //opera内核                
      webKit: u.indexOf('AppleWebKit') > -1, //苹果、谷歌内核                
      gecko: u.indexOf('Gecko') > -1 && u.indexOf('KHTML') == -1, //火狐内核                
      mobile: !!u.match(/AppleWebKit.*Mobile.*/) || !!u.match(/AppleWebKit/), //是否为移动终端                
      ios: !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/), //ios终端                
      android: u.indexOf('Android') > -1 || u.indexOf('Linux') > -1, //android终端或者uc浏览器                
      iPhone: u.indexOf('iPhone') > -1 || u.indexOf('Mac') > -1, //是否为iPhone或者QQHD浏览器                
      iPad: u.indexOf('iPad') > -1, //是否iPad                
      webApp: u.indexOf('Safari') == -1//是否web应该程序，没有头部与底部            
    };
  }()
}
```
如判断是否是ios,只需要这样使用：browser.versions.ios。如为true则为ios，否则不为ios。

#### 3、图片压缩并将base64的图片url数据转换为Blob。
```javascript
const imageUtils = {
    /**
     * 图片压缩（利用canvas）
     * @param  path     图片路径
     * @param  obj      压缩配置width,height,quality，不传则按比例压缩
     * @param  callback  回调函数
     */
    dealImage(path, obj, callback) {
        var img = new Image();
        img.src = path;
        img.setAttribute('crossorigin', 'anonymous')
        img.onload = function() {
            var that = this;
            // 默认按比例压缩
            var w = that.width,
                h = that.height,
                scale = w / h;
            w = obj.width || w;
            h = obj.height || (w / scale);
            //生成canvas
            var canvas = document.createElement('canvas'),
                ctx = canvas.getContext('2d');
            canvas.width = w;
            canvas.height = h;
            ctx.drawImage(that, 0, 0, w, h);
            // 默认图片质量为0.7
            var quality = 0.7;
            if (obj.quality && obj.quality <= 1 && obj.quality > 0) {
                quality = obj.quality;
            }
            // 回调函数返回base64的值
            var base64 = canvas.toDataURL('image/jpeg', quality);
            callback(base64);
        }
    },
    /* 将以base64的图片url数据转换为Blob */
    processData(dataUrl, type) {
        var binaryString = window.atob(dataUrl.split(',')[1]),
            arrayBuffer = new ArrayBuffer(binaryString.length),
            intArray = new Uint8Array(arrayBuffer);
        for (var i = 0, j = binaryString.length; i < j; i++) {
            intArray[i] = binaryString.charCodeAt(i);
        }
        var data = [intArray],
            blob;
        try {
            blob = new Blob(data);
        } catch (e) {
            window.BlobBuilder = window.BlobBuilder || window.WebKitBlobBuilder || window.MozBlobBuilder || window.MSBlobBuilder;
            if (e.name === 'TypeError' && window.BlobBuilder) {
                var builder = new BlobBuilder();
                builder.append(arrayBuffer);
                blob = builder.getBlob(type);
            } else {
                console.log('版本过低，不支持图片压缩上传');
            }
        }
        return blob;
    },
    /* 将以base64的图片url数据转换为Blob */
    dataURLtoBlob(dataurl) {
        // base64转blob
        var arr = dataurl.split(',');
        var mime = arr[0].match(/:(.*?);/)[1];
        var bstr = atob(arr[1]);
        var n = bstr.length;
        var u8arr = new Uint8Array(n);
        while (n--) {
            u8arr[n] = bstr.charCodeAt(n);
        }
        return new Blob([u8arr], { type: mime });
    }
}
export default imageUtils
```

#### 4、localStorage封装：
```javascript
let isLocalStorageAva = undefined
/**
 * this is a polyfill of localstore
 * 使用Cookie存储的Polyfill
 * @type {Storage}
 */
const localstore = {
  // 测试localStory是否可用
  checkLocalStory () {
    if (isLocalStorageAva === undefined) {
      try {
        window.localStorage.setItem('__test_local__', 'test')
        window.localStorage.removeItem('__test_local__')
        isLocalStorageAva = true
        return true
      } catch (error) {
        isLocalStorageAva = false
        return false
      }
    }
    return isLocalStorageAva
  },
  getItem (key) {
    if (this.checkLocalStory()) {
      return window.localStorage.getItem(key)
    }
    // 不行就从Cookie中读取
    let all = document.cookie
    let allKv = all.split(';')
    let objs = allKv.map(item => {
      let kv = item.split('=')
      return {
        key: decodeURIComponent(kv[0].trim()),
        value: kv.length > 1 ? decodeURIComponent(kv[1].trim()): undefined
      }
    })
    let finded = objs.findIndex(item => item.key === key)
    return objs[finded] ? objs[finded].value : null
  },
  setItem (key, value) {
    if (this.checkLocalStory()) {
      return window.localStorage.setItem(key, value)
    }
    let date = new Date()
    date.setDate(date.getDate() + 30)
    document.cookie = encodeURIComponent(key) + '=' + encodeURIComponent(value) + ';expires=' + date.toUTCString() + ';path=/'
  },
  clear () {
    if (this.checkLocalStory()) {
      return window.localStorage.clear()
    }
  },
  removeItem (key) {
    if (this.checkLocalStory()) {
      return window.localStorage.removeItem(key)
    }
    let date = new Date()
    date.setDate(date.getDate() - 30)
    document.cookie = encodeURIComponent(key) + '=1' + ';expires=' + date.toUTCString() + ';path=/'
  },
  key (index) {
    if (this.checkLocalStory()) {
      return window.localStorage.key(index)
    }
    // 不行就从Cookie中读取
    let all = document.cookie
    let allKv = all.split(';')
    let objs = allKv.map(item => {
      let kv = item.split('=')
      return {
        key: decodeURIComponent(kv[0].trim()),
        value: kv.length > 1 ? decodeURIComponent(kv[1].trim()): undefined
      }
    })
    if (index < 0 || index > objs.length) {
      throw 'index out of bound'
    }
    return objs[index].key
  }
}
// 会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性
Object.defineProperty(localstore, 'length', {
  get () {
    if (this.checkLocalStory()) {
      return window.localStorage.length
    }
    // 不行就从Cookie中读取
    let all = document.cookie
    let allKv = all.split(';')
    let objs = allKv.map(item => {
      let kv = item.split('=')
      return {
        key: decodeURIComponent(kv[0].trim()),
        value: kv.length > 1 ? decodeURIComponent(kv[1].trim()): undefined
      }
    })
    return objs.length
  }
})
export default localstore
```

#### 5、sessionStorage封装：
```javascript
let isSessionStorage = undefined
/**
 * this is a polyfill of sessionStorage
 * 使用Cookie存储的Polyfill
 * @type {Storage}
 */
const sessionstore = {
  // 测试localStory是否可用
  checkLocalStory () {
    if (isSessionStorage === undefined) {
      try {
        window.sessionStorage.setItem('__test_local__', 'test')
        window.sessionStorage.removeItem('__test_local__')
        isSessionStorage = true
        return true
      } catch (error) {
        isSessionStorage = false
        return false
      }
    }
    return isSessionStorage
  },
  getItem (key) {
    if (this.checkLocalStory()) {
      return window.sessionStorage.getItem(key)
    }
    // 不行就从Cookie中读取
    let all = document.cookie
    let allKv = all.split(';')
    let objs = allKv.map(item => {
      let kv = item.split('=')
      return {
        key: decodeURIComponent(kv[0].trim()),
        value: kv.length > 1 ? decodeURIComponent(kv[1].trim()): undefined
      }
    })
    let finded = objs.findIndex(item => item.key === key)
    return objs[finded] ? objs[finded].value : null
  },
  setItem (key, value) {
    if (this.checkLocalStory()) {
      return window.sessionStorage.setItem(key, value)
    }
    document.cookie = encodeURIComponent(key) + '=' + encodeURIComponent(value) + ';path=/'
  },
  clear () {
    if (this.checkLocalStory()) {
      return window.sessionStorage.clear()
    }
  },
  removeItem (key) {
    if (this.checkLocalStory()) {
      return window.sessionStorage.removeItem(key)
    }
    document.cookie = encodeURIComponent(key) + '=1' + ';expires = 0 ;path=/'
  },
  key (index) {
    if (this.checkLocalStory()) {
      return window.sessionStorage.key(index)
    }
    // 不行就从Cookie中读取
    let all = document.cookie
    let allKv = all.split(';')
    let objs = allKv.map(item => {
      let kv = item.split('=')
      return {
        key: decodeURIComponent(kv[0].trim()),
        value: kv.length > 1 ? decodeURIComponent(kv[1].trim()): undefined
      }
    })
    if (index < 0 || index > objs.length) {
      throw 'index out of bound'
    }
    return objs[index].key
  }
}
Object.defineProperty(sessionstore, 'length', {
  get () {
    if (this.checkLocalStory()) {
      return window.sessionStorage.length
    }
    // 不行就从Cookie中读取
    let all = document.cookie
    let allKv = all.split(';')
    let objs = allKv.map(item => {
      let kv = item.split('=')
      return {
        key: decodeURIComponent(kv[0].trim()),
        value: kv.length > 1 ? decodeURIComponent(kv[1].trim()): undefined
      }
    })
    return objs.length
  }
})
export default sessionstore
```