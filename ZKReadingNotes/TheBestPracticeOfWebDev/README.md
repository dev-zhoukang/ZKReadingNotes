##读书笔记之 Web前端开发最佳实践 ( 作者: 党建 )

###一: 编写高维护性的JS代码

####&1 养成良好的编程习惯

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

####&2 事件处理和业务处理逻辑分离

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

####&3 配置数据和代码逻辑分离

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

####&4 逻辑与结构样式分离

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
注: 也可以用 ***<template>*** 标签, 但是只有高版本的浏览器支持.  
