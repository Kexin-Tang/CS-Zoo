# 前端面试总结

本文汇总了一些常见的前端/全栈相关的面经，以供~~临时抱佛脚~~。

## HTML

**1. HTML**

HTML是 *Hyper Text Markup Language*，即超文本标记语言。

**2. `<!DOCTYPE html>`有什么用**

`<!DOCTYPE>` 声明位于文档中的最前面的位置，处于 `<html>` 标签之前。此标签可告知浏览器文档使用哪种规范。使用该方法可以避免浏览器使用自己默认的各种怪异类型去解析页面，强制所有浏览器遵守标准规范。

**3. 本地存储和Cookie**

本地存储是将一些内容存储在浏览器缓存中，只能在本地被访问，服务端一般无法访问；该缓存一般没有过期时间，除非用户手动清除浏览器缓存或通过JS清除。

Cookie既可以从服务端也可以从客户端访问数据，每次发出请求时都会发送Cookie的数据。具有一定的时效性，过期了就被删除了。

---

## CSS

**1. 盒子模型Box**

<img src="http://pic.caibaojian.com/uploads/2016/12/83659def42fb1e7291d3091e8533550a1.png">

* Border -- 定义了包含元素的最大面积。边框可以可见，也可以不可见，可以定义它的高度和宽度等。
* Padding -- 定义边框和元素之间的间距。
* Margin -- 定义边框和任何相邻元素之间的间距。
> border相当于房子外的栅栏，padding相当于屋子和栅栏的间距，margin相当于和隔壁房子的间距。

**2. BFC**

BFC(Block Formatting Context)即块级格式上下文，是页面中的一块渲染区域，有一套渲染规则，决定了其子元素如何布局，以及和其他元素之间的关系和作用。

满足以下条件即可视为BFC：
* 根元素，即HTML元素 
* float的值不为none 
* overflow的值不为visible(一般为hidden或auto) 
* display的值为inline-block、table-cell、table-caption 
* position的值为absolute或fixed

BFC的特点就是阻止左侧和上侧出现边界重叠。

**3. 选择器优先级**

!import &gt;&gt; inline &gt; id &gt; class &gt; tag &gt; *

**4. 实现三角形**
```css
.triangle-up{
    width: 0px;
    height: 0px;
    border-left: 50px solid transparent;
    border-right: 50px solid transparent;
    border-top: 0px;
    border-bottom: 100px solid red;
}
```

**5. 圣杯式布局**

所谓的圣杯布局就是左右两边大小固定不变，中间宽度自适应。我们可以用浮动、定位以及flex这三种方式来实现。

1. 浮动
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">

    <title>Document</title>
    <style type="text/css">
        .main>div {
            float: left;
        }

        .left {
            background-color: blue;
            width: 10%;
            margin-left: -90%;
        }

        .center {
            background-color: red;
            width: 80%;
            margin-left: 10%;
        }

        .right {
            background-color: yellow;
            width: 10%;
        }
    </style>
</head>

<body>
    <div class="main">
        <div class="center">中间</div>
        <div class="left">左边</div>
        <div class="right">右边</div>
    </div>

</body>

</html>
```

**6. 响应式布局**

根据浏览器大小、设备大小的不同，设计不同的网页布局，在css中使用`@media (case) {...}`

**7. 选择器的类型**
1. 普通tag：`p`, `a`
2. class：`.myClass`
3. id：`#certainID`
4. 相邻：`p + a`
5. 父子：`p > span`
6. 后代：`p span`
7. 属性选择：`form[action="/web"]`
8. 伪类选择：`img:hover`, `p:nth-child`

**8. div居中**
```css
div{
    width: 100px;
    height: 100px;
    margin: 0 auto;
}
```

**9. position的值**
* static（默认）：按照正常文档流进行排列；
* relative（相对定位）：不脱离文档流，参考自身静态位置通过 top, bottom, left, right 定位；
* absolute(绝对定位)：**脱离文档流**，参考距其最近一个不为static的父级元素通过top, bottom, left, right 定位；
* fixed(固定定位)：所固定的参照对像是可视窗口，即页面滚动，该元素也会在恒定的位置跟着滚动；

**10. 为什么要初始化CSS样式**

因为浏览器的兼容问题，不同浏览器对有些标签的默认值是不同的，如果没对CSS初始化往往会出现浏览器之间的页面显示差异

**11. display:none与visibility：hidden的区别**
* display：none 不显示对应的元素，在文档布局中不再分配空间（回流+重绘）
> I am a boy &rarr; I_am_boy
* visibility：hidden 隐藏对应元素，在文档布局中仍保留原来的空间（重绘）
> I am a boy &rarr; I_am_ _boy

**12. 伪类和伪元素**
1. 伪类使用`tag: property`，表示所有 *property* 的 *tag*，如`p:nth-child(2)`选中`p`中第2个`tag`
2. 伪元素使用`tag:: property `，表示 *tag* 内的元素满足 *property*，如`p::first-letter`大写每个`p`的首字母

**13. CSS引入的方法有哪些？都有什么区别？**
1. 外链式`<link href="/text.css" rel="stylesheet" />` -- *方便多个网页同时使用一个样式表*
2. 内嵌式`<style type="text/css"></style>` -- *速度快，可以减少HTTP请求，但是改版麻烦*
3. 导入式`<style>@import url(text.css)</style>` -- *不建议，因为页面首先加载，然后导入样式表，页面在1s~2s内会有没有样式表的情况，然后突然有了样式*
4. 行内式`<div style="width:200px;background:red;"></div>`

**14. alt &amp; title**

*alt* 是图片加载不出来时代替的文本；*title* 是鼠标悬浮图片上时的提示

**15. 

---

## JS

### 基础知识

### 看代码说结果

### 编程题
1. 使用下列语句，实现首先输出hello，3s后输出world,再间隔3s,再输出; `u.console("hello").settimeout(3000).console("world").settimeout(3000).console('!')` 
```js
function U(){
    this.time = 0;
    this.totalTime = 0;
    this.console = function(text){
        this.totalTime += this.time;
        setTimeout(function(){
            console.log(text);
        }, this.totalTime);
        return this;
    };
    this.settimeout = function(time){
        this.time = time || 0;
        return this;
    };
}

const u = new U();
u.console("hello").settimeout(3000).console('world').settimeout(3000).console('!');
```

---

## 网络

**1. DNS**
* DNS是用来将 URL &rarr; IP Address
* 使用UDP协议实现
* 先对本机DNS服务器查询，如果不在本机DNS服务器中，则分级查询，比如对于`www.hust.edu/eic`，根域名 &rarr; edu域名 &rarr; hust.edu域名 &rarr; eic域名。

**2. Cookie Vs Session**

* Cookie在客户端，Session在服务器端
* Cookie只能存储较少的ASCII数据，Session可以存任意数据。
* Cookie有效期比较⻓，在浏览器中一般⻓期保存。Session有效期比较短，在用户会话结束之后立刻失效。
* Cookie的安全性比较差，因此存放在用户端。

**3. 为什么需要Cookie&amp;Session?**

由于HTTP协议是无状态的协议，所以服务端需要记录用户的状态时，就需要用某种机制来识具体的用户，这个机制就是Session。通常，在服务器第一次响应某个页面时，会让浏览器使用Cookie记录一些信息，这样之后的HTTP请求就能让服务端和客户端匹配上。

**4. 请求(Request)**

包含四个内容：请求行(request line)、请求头部(request header)、空行以及请求内容(request body)

**5. 响应(Response)**

由四个部分组成，分别是：状态行、消息报头、空行和响应正文。

**6. 状态码**
> 分类 | 描述
> :---: | :---:
> 1xx | 请求继续执行
> 2xx | 成功
> 3xx | 重定向
> 4xx | 客户端出错
> 5xx | 服务器出错

**7. HTTP VS HTTPS**
> 可以把HTTPS理解为HTTP+SSL，即带有**加密传输**和**可靠性检测**的HTTP协议

**8. `POST` vs `GET`**
> 1. 浏览器用`GET`请求来获取一个html页面/图片/css/js等资源；用`POST`来提交一个`<form>`表单，并得到一个结果的网页
> 2. `GET`通过TCP直接把header+body组合成一个TCP包一起发出；而`POST`先发送header，服务器返回100后再发送body，即有两个TCP包
> 3. 使用`GET`会将各种参数包含在url中，如`www.google.com/search?q=xxx`;而`POST`会将参数包含在表单里，更加安全，如`www.google.com`
> 4. `GET`可以反复执行，而`POST`会再次提交表单（浏览器可能弹出alert('刷新后需要重新提交表单')) 
> 5. `GET`可以bookmark，而`POST`不行，因为每次`POST`都是不幂等的
> 6. `GET`请求参数会被完整保留在浏览器历史记录里，而`POST`中的参数不会被保留
> 7. `GET`请求在URL中传送的参数是有长度限制的，而`POST`没有
> 8. `GET`请求会被浏览器主动cache，而`POST`不会
> 9. `GET`请求只能进行url编码，而`POST`支持多种编码方式

**9. 一次HTTP请求的全过程**
   1. 根据URL，进行DNS解析
   2. 得到IP地址，建立TCP连接
   3. 发起HTTP请求
   4. 服务器处理
   5. 服务器返回响应
   6. 关闭TCP连接
   7. 浏览器对收到的数据进行渲染

**10. HTTP 1.0 Vs 1.1 Vs 2.0**
* 在HTTP1.1中规定，一个TCP连接中可以传输多个HTTP请求，这是1.1的新特性
* 在HTTP2.0中规定，一个TCP连接中可以将多个HTTP请求合在一起发送/接收，这是2.0的新特性

**11. OSI七层模型**

**12. TCP Vs UDP**

**13. TCP三次握手，四次挥手**

**14. HTTP常用的请求**
* Get
* Post
* Put
* Delete

**15. Post常用的数据格式**
* application/x-www-form-urlencoded
* application/json
* multipart/form-data

**16. 为什么利用多个域名来存储网站资源会更有效？**
1. CDN，表示让用户从离自己最近的下载点下载资源。
2. 突破服务器的带宽限制。
3. 节约主域名的连接数，提升并发
4. 更加安全，比如静态资源服务器上面，不能运行任何代码的。