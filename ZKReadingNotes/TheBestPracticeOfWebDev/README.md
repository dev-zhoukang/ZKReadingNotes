##读书笔记之 Web前端开发最佳实践 ( 作者: 党建 )

注: 已经阅读到 P121 (及时更新)

###一: 关于 HTML

####&1.1 给空标签添加隐藏文字, 用于说明标签的实际功能.  

可以通过以下代码将文字隐藏掉

```css
.test li {
  text-indent: -999px;
}
```

####&1.2 `<script>` 标签的属性 (defer 和 async)  

* 按照规范, 这两个属性只有在设置了 src 属性之后才会起作用.
* defer 是在 HTML4.01 规范中定义的, async 是在 HTML5 规范中定义的.
* defer 和 async 的区别在于:
    * defer 是让脚本后置加载和执行, 相当于把脚本放置在页面最后面加载和执行  
      async 是让脚本异步加载和执行.  
    * 设置 async 之后, 不能保证脚本按照顺序加载和执行, 脚本加载完成后立即执行.   
      而设置 defer 的脚本还是会按照原有的顺序执行.  
    * 因此, 如果脚本执行之间有依赖关系, 则不使用 async 属性;     
      如果页面中有内联的脚本依赖于加载的脚本, 则不使用 defer 属性.
      
###二: 关于 CSS

####&2.1 使用高效的 CSS 选择器

* 选择器匹配步骤:
 以 `.refrences p.list div {}` 为例
 CSS 选择器的匹配原理和我们习惯的匹配过程是相反的, 它是从右向左的.
 首先查找所有的 `<div>` 标签元素, 再查找元素是否具有 `list` 类的父元素,  
 然后查找这些父元素是否为 `<p>` 标签元素, 在已匹配的这些父元素中继续向上查找其父元素是否带有 `refrences` 类.  
* 高效的 CSS 选择器的最佳实践如下: 
    * 避免使用通配符 `*`
    * 避免使用标签选择器及单个属性选择器作为关键选择器.
        > 在一个选择器中, 最右边的选择器为关键选择器, 关键选择器决定着浏览器初始匹配的元素数量, 它是整个选择符整体匹配次数的最主要决定者.
        避免如下选择器  
        ```css
        .refrences p.list div {}
        .refrences p.list [data-link="#red"] {}  
        ```
    * 尽量不要在选择符中定义过多的层级, 最好不要超过三层.
        > 层级越深, 浏览器顺着 Dom 树查找匹配选择器的次数就越多.
        
####&2.2 CSS 相关的图片处理

下面的是关于图片的最佳实践

* 不要设置图片的尺寸
* 使用 CSS 精灵图 (CSS Sprite) 技术
    > 减少了网络请求的次数, 提高图片整体的加载速度.  
    > 缺点: 开发过程繁琐, 维护过程复杂, 使用不当会导致性能问题.
    下面是精灵图的最佳实践
    * 项目后期再应用 CSS Sprite 技术
    * 合理组织精灵图.
    * 控制其尺寸和大小.
        > 推荐尺寸长度和宽度相乘不要超过 2500, 且图片大小控制在 200k 之内.
    * 借助工具
        * 生成器: [CSS Sprite Generator](http://spritegen.website-performance.org)
        上传一个包含多个背景图的压缩包, 即可生成精灵图.
        * 根据精灵图生成 CSS 代码, 可以使用 [Sprite Cow](http://www.spritecow.com)
        * 如果网页已经开发完毕, 可以使用 [SpriteMe](http://spriteme.org) 通过网页分析, 来产生精灵图和代码. 

###三: 关于 JavaScript

####&3.1 养成良好的编程习惯

#####避免定义全局变量和函数
最佳实践如下:
把全局的变量包含在一个局部作用域中, 然后在这个作用域中完成这些变量的定义和使用.  
并且可以将需要公开的接口 `return` 出去.
```js
var testFunc = (function () {
  var length = 0;
  function init() {  };
  function action() {  };
  
  return {
    init: init
  }
})();
```
这样, 外部代码可以通过 `testFunc.init` 调用 `init` 函数了. 此方案既做到了代码逻辑的封装, 
又公开了外部需要访问的接口, 是代码模块化的最佳实践之一.

#####使用简化的编程方式(字面量)
#####使用 `===` 而不是 '=='
#####避免使用 `with` ( 无法确定代码按照期望运行, 同时也有设计缺陷 )
#####避免使用 `eval` ( 导致代码可读性差, 也有安全问题 )
#####不要编写监测浏览器的代码

####&3.2 事件处理和业务处理逻辑分离

一个典型的例子: 处理鼠标拖动页面元素, 事件处理函数的代码如下:
```js
var move_while_dnd = function(e) {
  var lb = scheduler._get_lightbox(); // 取得元素
  
  lb.style.top = e.clientY + 'px';
  lb.style.left = e.clientX + 'px';
};
```
上面代码是不好的写法, 原因是业务逻辑和事件处理逻辑耦合紧密, 不利于代码复用. 
好的做法是: 分离开来, 业务逻辑代码中不需要调用任何 event 对象的信息或阻止冒泡或者默认行为等, 调整如下:
```js
var setLightBoxPosition = function(top, left) {
  var lb = scheduler._get_lightbox();
  lb.style.top = top + 'px';
  lb.style.left = left + 'px';
}

var move_while_dnd = function (e) {
  setLightBoxPosition(e.clientY, e.clientX);
}
```
上面代码, 函数中的逻辑和事件处理没有任何的关联, 也不依赖特定的事件处理, 自然提高了代码的可维护性和复用性, 也有利于代码的自动化测试. 
测试代码不用模拟事件的触发, 可以直接调用业务处理函数来测试业务逻辑是否正确.  

####&3.3 配置数据和代码逻辑分离

代码中一些无逻辑的数据如 URL, 显示在页面上的提示信息等值统称为 ***配置数据***, 可变的配置数据应该从代码中抽离出来.
如下: 抽离之前: 
```js
// 抽离之前
var sm = startHours * 60 + startMinutes;
var em = (endHours * 60 + endMinutes) || (24 * 60);
var top = (sm * 60 * 1000 - 0 * 60 * 60 * 1000) * 42;
```
抽离的原则是: 数据在代码中是写死的, 并且在后期有可能会变更的数据. ( 注意: 固定不变的数据就没必要抽离出来啦 )  
抽离之后:
```js
// 抽离之后
this.config = {
  first_hour: 0,
  last_hour: 24,
  hour_size_px: 42,
  min_event_height: 40,
}

var sm = startHours * 60 + startMinutes;
var em = (endHours * 60 + endMinutes) || (this.config.last_hour * 60);
var top = (sm * 60 * 1000 - this.config.first_hour * 60 * 60 * 1000) * this.config.hour_size_px;
```

####&3.4 逻辑与结构样式分离

编写JS时, 让JS值关注逻辑行为, 尽量不要越权做本该 HTML 或者 CSS 完成的工作. 否则可维护性和可读性就会非常糟糕.  
尽量避免用JS代码来创建标签, 因为可读性比较差, 也比较容易出错.  
解决方案如下:
* 从服务器动态获取 HTML 代码
```js
$('#test_container').load('contents/template/store.html');
```
这样就减少了页面初始的 HTML 代码量, 加快页面的传输之间, 减少页面的解析时间.  
* 通过客户端动态生成页面结构  
在客户端, 为了不把动态加载的 HTML 代码或模板内嵌在JS代码中, 可以把这部分代码或模板放置在页面的HTML结构中, 
如果纯粹是HTML代码, 则直接隐藏在页面中即可, 通过JS控制其显示隐藏. 而模板文件推荐放置在 `script` 标签中, 并且为了不让浏览器解析成JS代码需要设置其 `type` 属性为其他值. 如下:  
```html
<script id="main_info" type="text/x-tmpl">
  <li>
    <b>${name}</b>
    ($(class))
  </li>
</script>
```
当需要取得模板代码时, 通过 innerHTML 属性即可
```js
var infoTemplate = document.getElementById('main_info').innerHTML;
```
注: 也可以用 ***`<template>`*** 标签, 但是只有高版本的浏览器支持.  

####&3.5 模板的使用

模板引擎有很多, 这里以 Underscore 中的模板引擎为例.
使用引擎记住以下几点:  
* 尽量不要在模板中滥用逻辑块
* 不要构建太复杂的模板
* 使用预编译模板  
  如下代码是 Underscore 中的模板预编译情况
  ```js
  // 取得模板并进行预编译
  var template = _.template($('template').html());
  // 在需要模板的地方, 可以直接使用预编译后的模板
  template(templateData);
  ```
  下面代码是缓存预编译后的模板方案
   ```js
   TemplateCache = {
     get: function(selector) {
       if (!this.templates) {
          this.templates = {};
       }
       var template = this.templates[selector];
       if (!template) {
         template = _.template(template);
         this.templates[selector] = template;
       }
       
       return template;
     }
   }
   ```
   在上述代码中, 可以使用 `TemplateCache.get(test)` 来取得模板了
   当然, 也可以使用 `Grunt` 插件来预先把模板文件转换成执行速度更快的函数. 性能更好一些.
   
####&3.6 JS模块化开发

* 如果前端模块较少, 可以通过自执行函数来设计模块  

  ```js
  var modole = (function() {
    var length = 0;
    var init = function() {};
    var action = function() {};
    return {
      init: init
    }
  })();
  ```
  为了最大量保持模块的独立性, 模块与模块之间最好通过各自的公开接口来通信, 如果模块之间存在很紧的依赖关系, 
  则在模块内部最好不要直接访问所依赖的外部模块, 而是通过参数的方式传入模块, 代码:   
  ```js
  var module1 = (function ($, module2) {
    // ...
  })(jQuery, module2);
  ```
* 如果前端模块过多, 需要动态加载, 并且模块之间的依赖关系复杂, 则需要更好的方式来管理模块的加载和之间的依赖关系.
  node 后端推荐 CommonJS, 浏览器端推荐使用 AMD.
  
  
####&3.7 高效的 DOM 操作

文档对象模型(DOM)是一个独立于特定语言的应用程序接口, 在浏览器中, DOM 接口是以 JS 语言实现的, 通过 JS 来操作浏览器页面中的元素,  
这使得 DOM 成为了 JS 中重要的组成部分. 在富客户端网页应用中, 页面上 UI 的更改都是通过 DOM 来实现的, 并不是传统的刷新页面实现的.  
尽管 DOM 提供了丰富的接口供外部调用, 但 DOM 的操作的代价很高, 页面前端代码的性能瓶颈也大多集中在 DOM 操作上, 所以, DOM 操作优化的总体原则是减少 DOM 操作.

为什么 DOM 操作会影响性能呢? 原因在于, DOM 的实现和 ECMAScript 的实现是分离的. 例如, 在 IE 中, ECMAScript 的实现在 jscript.dll 中, 而 DOM 的实现实在 mshtml.dll 中;  
在 Chrom 中, 使用 WebKit 中的 WebCore 来处理 DOM 和 渲染, 但 ECMAScript 是在 V8 引擎中实现的, 其他浏览器的情况类似.  
通过 JS 代码调用 DOM 接口, 相当于两个独立模块之间的交互, 相比较在同一模块中的调用, 这种跨模块的调用的性能损耗是很高的. 但 DOM 操作对性能最大的影响其实还是因为他导致了浏览器的重绘(repaint) 和 重排(reflow).   

为什么重绘和重排对性能影响很严重呢, 先简述一下浏览器渲染原理, 从渲染文档到渲染页面的过程中, 浏览器会通过解析 HTML 文档来构建 DOM 树, 解析 CSS 产生 CSS 规则树, JS 代码在解析过程中, 可能会修改生成的 DOM 树 和 CSS 规则树,   
根据 DOM 树和 CSS 规则树构建渲染树, 在这个过程中, CSS 会根据选择器匹配 HTML 元素. 渲染树包了每个元素的大小, 边距等样式属性, 渲染树中不包含隐藏元素及 head 等不可见元素, 最后浏览器根据浏览器的坐标和大小来计算每个元素的位置, 并绘制这些元素到页面上.  
重绘 是页面的某些部分需要重新绘制, 比如颜色或背景的修改, 元素的位置和尺寸并没有改变; 重排则是元素的位置或尺寸发生了变化, 浏览器需要重新计算渲染树, 导致渲染树的一部分或者全部发生变化.  
渲染树重新建立之后, 浏览器会重新绘制页面上受影响的元素. 重排的代价比重绘的代价高很多, 因为重绘会影响部分元素, 而重排可能会影响全部的元素.
  
现代浏览器会针对重绘和重排做性能优化, 如把 DOM 操作积累一批后统一做一次重排或重绘. 但以下情况会立即执行重排或重绘, 请求以下的 DOM 元素布局信息: offsetTop/Left/Width/Height, scrillTop/Left/Width/Height, clientTop/Left/Width/Height, getComputedStyle() 或者 currentStyle. 
因为这些值都是动态计算的, 所以浏览器需要尽快完成页面的绘制, 然后计算返回值, 从而打乱了重排和重绘的优化.  


