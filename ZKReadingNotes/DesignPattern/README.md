##读书笔记-JavasScript设计模式与开发实践

###1, call 和 apply
####1.1 call 和 apply 的区别
Function.prototype.call 和 Function.prototype.apply 都是非常常用的方法。它们的作用一模一样,区别仅在于传入参数形式的不同。
apply 接受两个参数,第一个参数指定了函数体内 this 对象的指向,第二个参数为一个带下 标的集合,这个集合可以为数组,也可以为类数组,apply 方法把这个集合中的元素作为参数传 递给被调用的函数:
```
var func = function(a, b, c) {
    alert([a, b, c]); // [1, 2, 3]
}
func.apply(null, [1, 2, 3])
```
在这段代码中,参数 1、2、3 被放在数组中一起传入 func 函数,它们分别对应 func 参数列 表中的 a、b、c。

call 传入的参数数量不固定,跟 apply 相同的是,第一个参数也是代表函数体内的 this 指向, 从第二个参数开始往后,每个参数被依次传入函数:
```
var func = function( a, b, c ){
    alert ( [ a, b, c ] ); // 输出 [ 1, 2, 3 ]
};
func.call( null, 1, 2, 3 );
```
注意: 当调用一个函数时,JavaScript 的解释器并不会计较形参和实参在数量、类型以及顺序上的 区别,JavaScript 的参数在内部就是用一个数组来表示的。从这个意义上说,apply 比 call 的使用 率更高,我们不必关心具体有多少参数被传入函数,只要用 apply 一股脑地推过去就可以了。
    call 是包装在 apply 上面的一颗语法糖,如果我们明确地知道函数接受多少个参数,而且想 一目了然地表达形参和实参的对应关系,那么也可以用 call 来传送参数。
    当使用 call 或者 apply 的时候,如果我们传入的第一个参数为 null,函数体内的 this 会指 向默认的宿主对象,在浏览器中则是 window.

####1.2 call 和 apply 的用途
(1) 改变 this 指向
代码如下, 如果不用`call`修正this指向的话, 那个`func();`运行之后, 函数中的`this`指向就是`window`, 直接报错`// 输出:undefined`
```
document.getElementById( 'div1' ).onclick = function(){ 
    var func = function(){
        alert ( this.id ); // 输出:div1
    }
    func.call( this ); 
};
```
(2) 借用其他对象的方法
函数的参数列表 arguments 是一个类数组对象,虽然它也有“下标”,但它并非真正的数组, 所以也不能像数组一样,进行排序操作或者往集合里添加一个新的元素。这种情况下,我们常常 会借用 Array.prototype 对象上的方法。比如想往 arguments 中添加一个新的元素,通常会借用 Array.prototype.push:
```
(function(){
    Array.prototype.push.call( arguments, 3 ); 
    console.log ( arguments ); // 输出[1,2,3]
})( 1, 2 );
```
想把 arguments 转成真正的数组的时候,可以借用 Array.prototype.slice 方法;想截去 arguments 列表中的头一个元素时,又可以借用 Array.prototype.shift 方法。
接下来, 我们腿短, 可以把“任意”对象传入
```
var a = {};
Array.prototype.push.call(a, 'first');
alert(a.length); // 1
alert(a[0]); // first
```
前面我们之所以把“任意”两字加了双引号,是因为可以借用 Array.prototype.push 方法的对
象还要满足以下两个条件,从 ArrayPush 函数的(1)处和(2)处也可以猜到,这个对象至少还要满足:
* 对象本身要可以存取属性; (如果上面粒子中的`a`是`number`类型, 就会报错)
* 对象的 length 属性可读写。(如果`a`是`function`类型, 也会报错)

###2, 装饰者模式__AOP的应用
类似`OC`的分类, 在需要给某个函数添加功能的时候, 很多时候我们不想去碰原函数,也许原函数是由其他同事编写的,里面的实现非常杂乱。甚 至在一个古老的项目中,这个函数的源代码被隐藏在一个我们不愿碰触的阴暗角落里。现在需要 一个办法,在不改变函数源代码的情况下,能给函数增加功能,这正是开放-封闭原则给我们指 出的光明道路。
利用`AOP`(面向切面编程), 首先给出 Function.prototype.before 方法 和Function.prototype.after 方法:
```
Function.prototype.before = function(beforefn) {
    var _self = this; // 保存原函数的引用
    return function() { // 返回包含了原函数和新函数的'代理'函数
        beforefn.apply(this, arguments); // 先执行新函数, 且保证this不被劫持, 新函数接受的参数也会被原封不动地传入原函数, 新函数在原函数之前执行.
        return _self.apply(this, arguments); // 执行原函数并返回原函数的执行结果, 并保证this不被劫持
    }
}
Function.prototype.after = function(afterfn) {
    var _self = this;
    return: function() {
        var ret = _self.apply(this, arguments);
        afterfn.apply(this, arguments);
        return ret;    
    }
}
```
下面看个例子:
```
<html>
    <button id="button"></butotn>
    <script>
    Function.prototype.before = function(beforefn) {
        var _self = this;
        return function() {
            beforefn.apply(this, arguments);
            return _self.apply(this, arguments);
        }
    }
    
    document.getElementById = document.getElementById.before(function(){
        alert('调用原函数之前先执行');
    });
    
    var button = document.getElementById('button');
    console.log(button);
    </script>
</html>
```
再来一个例子:

```
window.onload = function() {
    alert(1);
}
window.onload = (window.onload || function(){}).after(function() {
    alert(2);
}).after(function() {
    alert(3);
})

```
经典的提交表单的例子:
```
Function.prototype.before = function(beforefn) {
    var _self = this;
    return function() {
        if (beforefn.apply(this, arguments) === false) {
            return;
        }
        return _self.apply(this, arguments);
    }
}

var validata = function() {
    if (username.value === '') {
        alert('用户名不能为空');
        return false;
    }
    if (password.value === '') {
        alert('密码不能为空');
        return false;
    }
}

var formSubmit = function() {
    var param = {
        username: username.value,
        password: password.value
    }
    ajax('http://xx.com/login', param);
}

formSubmit = formSubmit.before(validata);

submitBtn.onclick = function() {
    formSubmit();
}

```

!!值得提到的是,上面的 AOP 实现是在 Function.prototype 上添加 before 和 after 方法,但许 多人不喜欢这种污染原型的方式,那么我们可以做一些变通,把原函数和新函数都作为参数传入 before 或者 after 方法:
```
var before = function(fn, beforefn) {
    return function() {
        beforefn.apply(this, arguments);
        return fn.apply(this, arguments);
    }
}

var a = before(
    function() {alert(3)}, 
    function() {alert(2)}    
)

a = before(a, function() {alert(1)});
a();
```
####注意:
值得注意的是,因为函数通过 Function.prototype.before 或者 Function.prototype.after 被装 饰之后,返回的实际上是一个新的函数,如果在原函数上保存了一些属性,那么这些属性会丢失。 代码如下:
```
var func = function() {
    alert(1);
}
func.a = 'a';
func = func.after(function() {
    alert(2);
});

alert(func.a); //输出: undefined
```
另外,这种装饰方式也叠加了函数的作用域,如果装饰的链条过长,性能上也会受到一些 影响。

####装饰者模式与代理模式的区别
装饰者模式和代理模式的结构看起来非常相像,这两种模式都描述了怎样为对象提供 一定程度上的间接引用,它们的实现部分都保留了对另外一个对象的引用,并且向那个对象发送 请求。
代理模式和装饰者模式最重要的区别在于它们的意图和设计目的。代理模式的目的是,当直接访问本体不方便或者不符合需要时,为这个本体提供一个替代者。
本体定义了关键功能,而代理提供或拒绝对它的访问,或者在访问本体之前做一些额外的事情。
装饰者模式的作用就是为对 象动态加入行为。换句话说,代理模式强调一种关系(Proxy 与它的实体之间的关系),
这种关系 可以静态的表达,也就是说,这种关系在一开始就可以被确定。而装饰者模式用于一开始不能确 定对象的全部功能时。代理模式通常只有一层代理本体的引用,而装饰者模式经常会形成一条 长长的装饰链。
在虚拟代理实现图片预加载的例子中,本体负责设置 img 节点的 src,代理则提供了预加载 的功能,这看起来也是“加入行为”的一种方式,但这种加入行为的方式和装饰者模式的偏重点 是不一样的。
装饰者模式是实实在在的为对象增加新的职责和行为,而代理做的事情还是跟本体 一样,最终都是设置 src。但代理可以加入一些“聪明”的功能,比如在图片真正加载好之前, 先使用一张占位的 loading 图片反馈给客户。

###3, 节流函数的实现
throttle 函数的原理是,将即将被执行的函数用 setTimeout 延迟一段时间执行。如果该次延迟执行还没有完成,则忽略接下来调用该函数的请求。 throttle 函数接受 2 个参数,第一个参数为需要被延迟执行的函数,第二个参数为延迟执行的时 间。具体实现代码如下:
```
var throttle = function(fn, interval) {
    var _self = fn, // 保存需要延迟执行的函数引用
    timer, 
    firstTime = true; // 是否第一次调用
    
    return function() {
        var args = arguments,
        _me = this;
        if (firstTime) { // 如果是第一次调用, 不需要延迟执行
            _self.apply(_me, args);
            return firstTime = false;
        }
        if (timer) { // 如果定时器还在, 说明前一次延迟执行还没有完成
            return false;
        }
        timer = setTimeout(function() {
            clearTimeout(timer);
            timer = null;
            _self.apply(_me, args);
        }, interval || 500);
    }
};

window.onresize = throttle(function() {
    console.log(1);
}, 600);
```
###4, 懒加载函数
比如我们需要一个在各个浏览器中能够通用的事件绑定函数 addEvent 函数, 最常见的也是不好的写法如下:
```
var addEvent = function(elem, type, handler) {
    if (window.addEventListener) {
        return elem.addEventListener(type, handler, false);
    }
    if (window.attachEvent) {
        return elem.attachEvent('on' + type, hadler);
    }
};
```
这个函数的缺点是,当它每次被调用的时候都会执行里面的 if 条件分支,虽然执行这些 if
分支的开销不算大,但也许有一些方法可以让程序避免这些重复的执行过程。下面介绍最佳实践方式:
```
var addEvent = function(elem, type, handler) {
    if (window.addEventListener) {
        addEvent = function(elem, type, handler) {
            elem.addEventListener(type, handler, false);
        }
    }
    else if (window.attachEvent) {
        addEvent = function(elem, type, handler) {
            elem.attachEvent('on' + type, handler);
        }
    }
    addEvent(elem, type, handler);
}
```
上面懒加载函数的原理就是
此时 addEvent 依然被声明为一个普通函 数,在函数里依然有一些分支判断。但是在第一次进入条件分支之后,在函数内部会重写这个函 数,重写之后的函数就是我们期望的 addEvent 函数,
在下一次进入 addEvent 函数的时候,addEvent 函数里不再存在条件分支语句.

###5, 单例模式
```
var getSingle = function(fn) {
    var result;
    return function() {
        return result || (result = fn.apply(this, arguments));
    }
};
```
上面代码中, result变量因为身在闭包中, 他永远不会被销毁. 在将来的请求中, 如果result已经被赋值, 那么他将返回这个值.

下面举个单例登录框的例子:
```
var createLoginLayer = function() {
    var div = document.createElement('div');
    div.innerHTML = '假装我是登录浮窗';
    div.style.display = 'none';
    document.body.appendChild(div);
    return div;
}
var createSingleLoginLayer = getSingle(createLoginLayer);

document.getElementById('login-btn').onclick = function() {
    var loginLayer = createSingleLoginLayer();
    loginLayer.style.display = 'block';
}

```

###策略模式
定义: 定义一系列的算法, 把他们一个一个封装起来, 并且可以使他们可以相互替换.
将不变的部分与变化的部分分隔开是每个设计模式的主题, 策略模式也不例外, 策略模式的目的就是将算法的使用和算法的实现分离开来.
先来一个经典的使用场景: 年终奖的计算: 绩效为S的人年终奖是4倍工资, 绩效A的是3倍工资, 绩效B的是2倍工资.
最佳实现方式就是用策略模式进行代码的封装:
```
var strategies = {
    'S': function(salary) {
        return salary * 4;
    },
    'A': function() {
        return salary * 3;
    },
    'B': function() {
        return salary * 2;
    }
};

var calculateBonus = function(level, salary) {
    return strategies[level](salary);
};

console.log(calculateBonus('A', 20000)); // 60000
```
再来个经典的例子, 验证表单的输入合法性:
```
/* 策略对象 */
var strategies = {
    isNonEmpty: function(value, errorMsg) {
        if (value === '') {
            return errorMsg;
        }
    },
    minLength: function(value, length, errorMsg) {
        if (value.length < length) {
            return errorMsg;
        }
    },
    isMobile: function() {
        if (!/(^1[3|5|8][0-9]$)/.test(value)) {
            return errorMsg;
        }
    }
}
/* Validator类 */
var Validator = function() {
  this.cache = [];
};

Validator.prototype.add = function(dom, rules) {
    var self = this;
    for (var i = 0, rule; rule = rules[i ++]; ) {
        (function(rule) {
            var strategyAry = rule.strategy.split(':');
            var errorMsg = rule.errorMsg;
            
            self.cache.push(function() {
                var strategy = strategyAry.shift();
                strategyAry.unshift(dom.value);
                strategyAry.push(errorMsg);
                return strategies[strategy].apply(dom, strategyAry);
            });
        })(rule)
    }
};

Validator.proototype.start = function() {
    for (var i = 0, validatorFunc; validatorFunc = this.cache[i ++]; ) {
        var errorMsg = validatorFunc();
        if (errorMsg) {
            return errorMsg;
        }
    }
};

/* 客户端调用 */

var registerForm = document.getElementById('register-form');

var validataFunc = function() {
    var validator = new Validator();
    validator.add(registerForm.userName, [ 
        {
            stratrgy: 'isNonEmpty',
            errorMsg: '用户名不能为空'
        }, 
        {
            stratrgy: 'minLength: 10',
            errorMsg: '用户名长度不能小于10位'
        }
    ]);
    validator.add(registerForm.password, [{
        stratrgy: 'minLength: 6',
        errorMsg: '用户名长度不能小于6位'
    }]);
    
    var errorMsg = validator.start();
    return errorMsg;
}

registerForm.onsubmit = function() {
    var errorMsg = validataFunc();
    if (errorMsg) {
        alert(errorMsg);
        return false;
    }
}
```