---
layout: post
title : JavaScript编程
---

JavaScript是一门简单但完备的语言。看上去结构就像一个 hash 表，可以给 key 绑定值或函数。

# this
this 指的当前实例对象。通过改变this，实现各种动态。

它代表函数运行时，自动生成的一个内部对象，只能在函数内部使用。比如:

```js
function test() {
  this.x = 1;
}
```

随着函数使用场合的不同，this的值会发生变化。但是有一个总的原则，那就是this指的是:

> 调用函数的那个对象。

# prototype
正是prototype体现了JavaScript的Script，其它的都是Java了。
通过prototype，可以实现类、继承进而其它高级的设计模式。

## 构造函数
和一般函数一样，只是为了便于认出首字母大写，然后通过**new**来创建新实例。

可以看出，JavaScript 的实例创造，就是给 new 方法传了一个构造函数，而不像其他真正的面向对象编程语言传的是一个类，然后类里有构造函数。

这个构造函数里的 this，即是实例的 this。

```javascript
function Person(name) {
  this.name = name;
  this.who = function() {
    console.log(this.name);
  }
}

var p1 = new Person('Jack');
var p2 = new Person('Tom');
p1.who()  // #=> Jack
```

在使用**new**来创建新实例时，会经历下面4个步骤：

* 创建一个新的对象。
* 将构造函数的作用域赋给新对象（因此this就指向了这个新对象）。
* 执行构造函数中的代码（为这个新对象添加属性）。
* 返回新对象。

由于在JavaScript中函数也是对象，因此上例中的 p1 和 p2 的 who 方法是不同的，都有自己的定义。也就是说每次实例化一个对象，都会重新定义一遍方法，显然这是不可取的。

因此会选择在构造函数的 prototype，即原型来定义实例的方法以及实例间共享的变量，有点父类的意思。为什么不直接实现父类呢？只因想简单些。

## 寄生组合式继承

# 闭包
通过闭包，JavaScript实现了命名空间的隔离。

# Node.js
前段与后端都是用JS写的，但是他们之间如何通信呢？

REST。

## ejs
使用ejs，把数据嵌进去。<%= %>之间便是要展示的数据。因为在控制层有一个render的函数用来渲染view。

这就是MVC啊，render很神奇。在Rails中，controller与view中的通过@声明的变量是相通的。而modle层就是只是方法、数据结构的定义而不是实例。

当然js原来就是前端，所以当引入库时，node.js能完成的前端也可以完成的。比如可以做一个前端加后台数据库的网站，所有逻辑写在前端，但这样你的逻辑与秘钥也就暴漏了。因为前端都是公开的。

## jade

# TIPS

数组长度用array的length访问得到：a.length。

# 参考
- http://www.ruanyifeng.com/blog/2011/06/designing_ideas_of_inheritance_mechanism_in_javascript.html