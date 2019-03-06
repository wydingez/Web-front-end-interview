## CSS
### 1. 清除浮动，防止父级高度塌陷
* 为父级div定义伪类:after和zoom
```css
  .clearfloat:after { display: block; content: ''; clear: both;  }
```
* 结尾处添加空div标签，并设置clear：both
* 创建父级BFC
* 父级定义高度height

### 2. 块级格式化上下文（BFC）
页面中的一块渲染区域，并有一定的渲染规则，决定其子元素如何定位，以及和其他元素的关系和相互作用

具备BFC特性的元素可以看做是一个隔离的容器，容器里边的元素在布局上不会影响到外面的元素。

触发BFC特性的方式：
* body根元素
* 浮动元素：float除none以外的值
* 绝对定位元素：position为absolute、fixed
* display为inline-block、table-cell
* overflow除了visible以外的值（hideen、auto、scroll）

应用：
* 同一个BFC下外边距会发生重叠：解决方法就是将元素放在不同的BFC容器中去
* BFC可以包含浮动元素（清除浮动）
* BFC可以阻止元素被浮动元素覆盖

### 3. 居中布局
* 水平居中
  * 行内元素：text-align:center
  * 块级元素：margin: 0 auto
  * absolute + transform
  * flex + justify-content: center
* 垂直居中
  * line-height + height
  * absolute + transform
  * flex + align-items: center
  * table (vertical-align: middle)
* 水平垂直居中
  * absolute + transform
    ```css
    .inner {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%)
    }
    ```
  * flex + justify-content + align-items
    ```css
    .wrapper {
      display: flex;
      justify-content: center;
      align-items: center;
    }
    ```
### 4. css实现三角形
```css
div {
  width: 0;
  height: 0;
  border-top: 40px solid #000;
  border-right: 40px solid transparent;
  border-left: 40px solid transparent;
  border-bottom: 40px solid transparent;
}
```

## JS
### 1. 防抖和节流函数
都是为了优化高频操作，对性能有着极大的提升

* 防抖（debounce）
单位时间触发多次时，只在最后一次执行，通常用在用户表单输入校验时的操作
```js
  function debounce (fn, wait, immediate) {
    let timer = null

    return function () {
      let args = arguments
      let context = this

      if (!timer && immediate) {
        fn.apply(context, args)
      }

      if (timer) clearTimeout(timer)

      // 多次重复执行，重置计时器
      timer = setTimeout(() => {
        fn.apply(context, args)
      }, wait)
    }
  }
```

* 节流(throttle)
每隔一段时间后执行一次，也就是降低频率；使用场景：滚动事件或者resize事件
```js
  function throttle (fn, wait, immediate) {
    let timer = null
    let callNow = immediate

    return function () {
      let context = this
      let args = arguments

      if (callNow) {
        fn.apply(context, args)
        callNow = false
      }

      if (!timer) {
        timer = setTimeout(() => {
          fn.apply(context, args)
          timer = null
        }, wait)
      }
    }
        
  }
```

### 2. 模块化
模块化在现代开发中必不可少，大大的提高了项目的可维护、可扩展和可协作性。我们通常在浏览器中使用ES6的模块化支持，Nodejs中使用commonjs的模块化开发
* 分类
  * ES6：import / export
  * commonjs：require / module.exports / exports
  * amd

* require 和 import 的区别
  * require支持动态导入，import不支持
  * require是同步导入，import属于异步导入
  * require是值拷贝，导出的值不会影响导入值；import指向内存地址，导入值会谁导出值变化而变化

### 3. 继承
* 圣杯模式
```js
var inherit = (function(target, origin) {
  var F = function() {}
  return function (target, origin) {
    F.prototype = origin.prototype
    target.prototype = new F()
    target.uber = origin.prototype
    target.prototype.constructor = target
  }
})()
```
* es6的class / extends

### 4. 原型 / 构造函数 / 实例
```js
function foo () {}
var f = foo()

// foo：称为构造函数
// f：称为实例
// f.__proto__：称为原型

foo.prototype === f.__proto__   // 原型
foo.prototype.constructor === foo
f.constructor === foo

foo.prototype.__proto__ === Object.prototype

foo.__proto__ === Function.prototype

Function.prototype.__proto__ === Object.prototype
```

## http/https
### 1. 缓存
分为强缓存和协商缓存，两种缓存分别通过Http报文头部不同的字段进行控制
* Cache-control / Expires : 浏览器判断缓存是否过期，未过期时，直接使用强缓存，Cache-control的max-age优先级高于Expires
* 当缓存已经过期时，使用协商缓存：
  * 唯一标志方案：Etag（response携带） & If-None-Match（request携带，上一次返回的Etag）：服务器判断资源是否修改
  * 最后一次修改时间：Last-Modified（response） & If-Modified-Since（request，上一次放回的Last-Modified）
    * 如果一致，则直接返回304，通知浏览器使用缓存
    * 果不一致，则服务器返回新的资源

### 2. 跨域
* JSONP： 利用<script>标签不受跨域限制的特点，缺点是只能支持get请求
```js
function jsonp(url, jsonpCallback, success) {
  const script = document.createElement('script)
  script.url = url
  script.async = true
  script.type 'text/javascript'
  window[jsonpCallback] = function (data) {
    success && success(data)
  }
  document.body.appendChild(script)
}
```

* CROS： 基本思想是使用自定义的HTTP头部允许浏览器和服务器之间的交互
一般都是由服务器端开启：
  Access-Control-Allow-Origin：指定授权访问的域
  Access-Control-Allow-Methods：授权请求的方法（GET / POST / PUT / DELETE / OPTIONS 等）

* HTML5的新API：postMessage
```js
window.frames[0].postMessage(data, 'http://www.aaa.com')
window.addEventListener('message', callback)
```
