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

**4. Attribute &amp; Property**
```html
<input id="the-input" type="typo" value="Name:" />
```
在上述例子中，`input.getAttribute('type')`输出为`typo`，`input.type`输出为`text`，即 *Property* 相当于人固有的部件，耳朵眼睛鼻子等；*Attribute* 相当于人附加的部件，如名字外号学历等。*Attribute* 的输出就是DOM中写入的内容，而*Property*可能与看见的不同，且都是在固定的范围内取，比如`type`只能为 *text*, *checkbox*等。

**5. DOM**

DOM是文档对象模型，可以把页面解析成一个结构树，并提供API用于和页面进行交互。

**6. 浏览器如何渲染页面**

1. 将HTML构造成DOM树
2. CSS解析，将CSS构造成CSSOM树
3. 构造渲染树：DOM树的根节点开始遍历每个可见节点，让后对每个可见节点找到适配的CSS样式规则并应用
4. layout构造，根据渲染树每个节点的信息，在网页页面进行定位
5. 渲染树绘制，浏览器对内容进行修饰



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

**15. margin塌陷**
* 垂直之间塌陷的原则是以两盒子最大的外边距为准，如上面的div margin-bottom为50px，下面的div margin-top为40px，则两者之间的margin是50px而非90px
* 嵌套时为内部元素添加margin-top，内外部元素都会向下移动

---

## JS

### 基础知识

**1. 普通函数 Vs 箭头函数**
1. 箭头函数是匿名函数，普通函数是匿名或者具名函数
2. *this* 指向不同，普通函数中 *this* 指向调用的对象；而箭头函数指向箭头函数定义时所处的对象，而不是箭头函数使用时所在的对象，默认使用父级的 *this*
3. 普通函数有 *arguments*，存储了传入的对象；箭头函数没有 *arguments*
4. 箭头函数不能使用 *new*，即不能作为构造函数
> argument和this这两个是有特殊性的

**2. == &amp; ===**
* == 是先进行类型转换，再进行判断，只会判断值，不会判断类型
* === 不进行类型转换，会判断值和类型

**3. 变量提升**

JS会自动把用`var`定义的变量声明提前(但是不会提升变量赋值, 如果一个变量先使用再赋值, 那么提升时赋值为`undefined`)
```js
// 下面文件执行时, 输出结果为 undefined, 10, 99
var a = 99
f()
console.log(a)  // 会在f()执行完成后执行, 打印全局变量99
function f(){
  console.log(a)  // 会将局部变量a的声明提前, 此时a=undefined
  var a = 10
  console.log(a)  // 打印局部变量a=10
}
```

**4. 防止自动提交和Bubble**

使用`e.preventDefault()`和`e.stopPropagation()`

**5. 原型 &amp; 原型链**

原型：每个对象都会在其内部初始化一个属性，就是 *prototype*，原型中的属性和方法是可以被所有对象使用的，他们保存在一个单独的空间中，不会因为创建新的对象而保存到不同的地方。

原型链：利用原型让一个引用类型继承另一个引用类型的属性和方法。如 *Student* &rarr; *Person* &rarr; *Object*。

**6. 类的定义**
1. 给每个不同的对象创建不同的空间，即定义的内容不是存储在`__proto__`中供所有对象使用的
```js
function U(a, b){
    const u = {};
    u.a = a;
    u.b = b;
    u.add = (a, b) => a+b;
    return u;
}
const u = U(1, 2);
u.add();    // 3
```

2. 分开定义属性和方法
```js
function U(a, b) {
    this.a = a;
    this.b = b;
};
U.prototype.add = function() {
    return this.a + this.b;
};
const u = U(1, 2);
u.add();    // 3
```

3. 使用`constructor`
```js
class U{
    constructor(a, b){
        this.a = a;
        this.b = b;
    };
    add: () => this.a + this.b;
};
const u = new U(1, 2);
u.add();    // 3
```

**7. let &amp; var**

*let* 必须定义后才能使用，且不能重复定义，同时具有块作用域；
```js
for (var i = 0; i <10; i++) {  
  setTimeout(function() {  // 同步注册回调函数到 异步的 宏任务队列。
    console.log(i);        // 执行此代码时，同步代码for循环已经执行完成
  }, 0);
}
// 输出结果
// 10   共10个

for (let i = 0; i < 10; i++) { 
  setTimeout(function() {
    console.log(i);    //  i 是循环体内局部作用域，不受外界影响。
  }, 0);
}
// 输出结果：
// 0  1  2  3  4  5  6  7  8 9
```

**8. null &amp; undefined**

*null* 是代表没有定义，即不存在，*undefined* 是代表无初始值。

**9. 柯里化**

把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术。
```js
function func1(x){
    return function (y){
        return x + y;
    }
};
a = func1(1);
b = a(2);   // 3
```
优点：(1)参数复用；(2)延迟运行，即先运行需要第一个参数的内容，然后再返回一个可以执行其他参数的函数；(3)预先判断，根据第一个参数，返回不同的函数去处理后面的参数。

**10. JS数据类型**
* 基本数据类型：Number, String, undefined, null, Object, Symbol, Boolean
* 引用数据类型：Array, Object, Map, Set

**11. 立即执行函数**

1. 定义与执行方法如下
```js
// method 1
(function(){
    ...
}());
// method 2
(function(){
    ...
})()
```
2. 立即执行函数只有一个作用：创建一个独立的作用域。这个作用域里面的变量，外面访问不到。
```js
// 相当于使用在for中用let代替var
var liList = ul.getElementsByTagName('li')
for(var i=0; i<6; i++){
  (function(j){
    liList[j].onclick = function(){
      alert(j) // 0、1、2、3、4、5
    }
  })(i)
}
```
3. 立即执行函数只是函数的一种调用方式，只是声明完之后立即执行，这类函数一般都只是调用一次，调用完之后会立即销毁，不会占用内存。
4. 闭包则主要是让外部函数可以访问内部函数的作用域，也减少了全局变量的使用，保证了内部变量的安全，但因被引用的内部变量不能被销毁，增大了内存消耗，使用不当易造成内存泄露。

**12. 闭包**

闭包就是用函数访问某些函数内部保护的私密数据。
* 优点：封装；减少全局变量
* 缺点：内存泄漏
> 怎么解决内存泄漏？ &rarr; 将不需要使用的赋值为null即可
```js
function a(){
     var i=0;
     function b(){
         i++;
         alert(i);
     }
     return b;
}
// 因为i会被保存在内存中，所以可以被不断的访问
var c = a();
c();// 1
c();// 2
c();// 3
```

**13. 实现`setTimeout`与`setInterval`**
* `setTimeInterval`
```js
// 利用嵌套调用
function myInterval(fn, interval){
    let t = null;
    return {
        start: function(){
            t = setTimeout(()=>{
                fn();
                this.start();
            }, interval);
        },
        end: function(){
            clearTimeout(t);
            t = null;
        }
    }
}
```

**14. `call()`, `apply()`与`bind()`**

这些函数都是用来改变程序的上下文，即改变 *this* 的指向。

1. 如果使用 *apply* 或 *call* 方法，那么 *this* 指向他们的第一个参数，*apply* 的第二个参数是一个参数数组，*call* 的第二个及其以后的参数都是数组里面的元素。
```js
const obj1 = {
    name: "tom",
    func: function() {
        console.log(this.name);
    }
};
const obj2 = {
    name: "jerry"
};
obj1.func.apply(obj2);  // jerry
obj1.func.call(obj2);   // jerry
```
2. *bind* 与他们相似，但是<u>并不会立即执行</u>，而是进行绑定后等待调用。同时，如果多次对绑定不同的函数，只会对首次绑定的生效。(传参方式同 *call*)
```js
let max0 = Math.max.bind(Math, 1, 20, 9, 7);    // 此时并无输出
max0(); // 20
```

**15. 手写bind**
* 简单版本，不能支持 *new*
```js
Function.prototype.myBind = function(myThis, ...args1){
    const fn = this;    // fn即为调用该函数的对象，如fn.myBind()
    return function(...args2){
        return fn.apply(myThis, [...args1, ...args2]);
    }
}
```
* 高级版本，支持 *new*
```js
Function.prototype.myBind = function(myArgs){
    const slice = Array.prototype.slice;
    const args1 = slice.call(arguments, 1);    // 除了第一个代表函数的参数外的其他参数
    const fn = this;
    if(typeof fn !== 'function'){
        throw new Error("Must pass a function");
    }
    function temp(){
        const args2 = slice.call(arguments, 0);
        return fn.apply(temp.prototype.isPrototypeOf(this)? this: myArgs
        , args1.concat(args2));
    }
    temp.prototype = fn.prototype;

    return temp;
}
```

**16. new的过程**
```js
// new fn(args)
const temp = {}
temp.__proto__ = fn.prototype
fn.apply(temp, [...args]) // fn.call(temp, ...args)
return temp
```

**17. ES6**

ES6新加特性，典型的如下：
   1. 加入 *let*, *const*
   2. 加入了 *Promise*
   3. 加入了 *async/await*
   4. 增加了反引号(`)
   5. 增加了 `for ... of ...`
   6. 增加了 *class*
   7. 增加了 *Symbol*

**18. Promise**

*Promise* 的构造函数是同步的，但是 *then* 和 *catch* 是异步的。

*Promise* 有三种情形：*Pending*, *Resolved*, *Rejected*。

*Promise* 中的 *rejected* 是 *Promise* 的方法，而 *catch* 是实例对象的方法。同时，*rejected* 用于抛出异常，*catch* 用于捕获异常。

**19. 事件传播**

在捕获过程中，由外到内，查找具体是哪个部件有事；响应是由内到外，被称为冒泡，不断的扩散。可以使用`e.stopPropagation`来终止冒泡。


### 看代码说结果

**题目一**
```js
var A=function(){}

A.prototype.n=1

var b=new A()

A.prototype={
     n:2,
     m:3
}

var c=new A()

console.log(b.n,b.m,c.n,c.m)    // 1, undefined, 2, 3
```
b在继承A的时候，n=1，m不存在；c在继承A的时候，n=2，m=3。

**题目二**
```js
var F=function(){};

Object.prototype.a=function(){
     console.log('a()')
};

Function.prototype.b=function(){
     console.log('b()')
}

var f=new F();

f.a()// a()
f.b()// 找不到函数
F.a()// a()
F.b()// b()
```

**题目三**
```js
var length = 10;
 
function fn() {
    return this.length + 1;
}
 
var obj = {
    length: 5,
    test1: function () {
        return fn();
    }
};
 
obj.test2 = fn;
 
console.log(obj.test1())    // 11，相当于直接调用fn
console.log(obj.test2())    // 6，相当于在obj中定义了一个返回length+1的函数
```
该题目中相当于定义
```js
var obj = {
    length: 5,
    test1: function() {
        return fn();
    },
    test2: function() {
        return this.length + 1;
    }
}
```

**题目四**
```js
console.log('event start')
setTimeout(function () {
    console.log('setTimeout');
});

new Promise(function(resolve,reject){
    console.log('promise start')
    resolve()
}).then(function(){
    console.log('promise end')
})
console.log('event end')
// event start
// promise start
// promise end
// event end
// setTimeout
```
该问题涉及到宏任务(macro-task)和微任务(micro-task)的概念。宏任务包括`setTimeout`和`setInterval`，微任务包括`Promise`。<u>当前执行栈执行完毕时会立刻先处理所有微任务队列中的事件，然后再去宏任务队列中取出一个事件。同一次事件循环中，微任务永远在宏任务之前执行。</u>

**题目五**
```js
console.log(0.1 + 0.2 == 0.3);  // false
```
因为无法精确表示小数0.1，0.2等，所以会有舍入误差，比如0.1+0.2=0.3000004，所以上述式子不等。

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

2. 使用 *Set* 去重
```js
let nums = [1,1,2,3,2];
let sets = [... new Set(nums)];
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