---
title: 「JavaScript基础」原型和原型链
date: 2023-05-16 22:14:45
tags: JavaScript基础
---
# 前言
从JavaScript开始流行到今天，学习它的人都会有个疑惑——JavaScript是不是面向对象语言？其实，ECMA-262早就给出了答案，在ECMAScript的第一个版本中就明确指出，ECMAScript是一种面向对象的语言，参见如下引文：
```
ECMAScript is an object-oriented programming language for performing computations and manipulating computational objects within a host environment.
```
既然ECMAScript是面向对象的，那么JavaScript作为ECMAScript的一个分支，当然也是一种面向对象的语言。常见的C#、Java这样面向对象语言是基于类（class-based）的面向对象，而**JavaScript是基于原型（prototype-based）的面向对象。**
# 基于原型的继承
对于使用过基于类的语言 (如 Java 或 C++) 的开发者们来说，JavaScript 实在是有些令人困惑 —— JavaScript 是动态的，本身不提供一个 class 的实现。即便是在 ES2015/ES6 中引入了 class 关键字，但那也只是语法糖，JavaScript 仍然是基于原型的。
## prototype
JavaScript 对象有一个指向一个原型对象的链。当试图访问一个对象的属性时，它不仅仅在该对象上搜寻，还会搜寻该对象的原型，以及该对象的原型的原型，依次层层向上搜索，直到找到一个名字匹配的属性或到达原型链的末尾。
我们先来看几个例子
```
function Person() {

}
var person = new Person();
person.name = '二毛';
console.log(person.name) // 二毛
```
上述例子，Person被视为一个构造函数，通过new关键字实例化了对象```person```，可以看到 JavaScript 可以直接为对象构造器未声明的属性复制。
```
function Person() {

}

var person = new Person();
person.name = '二毛';
console.log(person.name) // 二毛

Person.prototype.age = 18;
var person1 = new Person();
var person2 = new Person();

console.log(person.age) // 18
console.log(person1.age) // 18
console.log(person2.age) // 18
```
每个函数都有一个 ```prototype``` 属性, 函数的 ```prototype``` 属性指向了一个对象，这个对象正是调用该构造函数而创建的实例的原型，也就是上述例子中的person, person1 和 person2 的原型。那什么是原型呢？可以这样理解：**每一个JavaScript对象(null除外)在创建的时候就会与之关联另一个对象，这个对象就是我们所说的原型，每一个对象都会从原型继承属性。**
用一张图来表示他们的关系：
{% asset_img image1.png 原型链关系图1 %}
上图显示了构造函数与实例原型之间的关系，那么实例又是访问到实例原型的呢？你需要了解实例对象的一个隐式属性 ```__proto__```
## \_\_proto\_\_
这是每一个JavaScript对象(除了 null )都具有的一个属性，叫 ```__proto__```，这个属性会指向该对象的原型。
尝试运行以下代码
```
function Person() {

}
var person = new Person();
console.log(person.__proto__ === Person.prototype); // true
```
到此，我们可以更新一下上述关系图
{% asset_img image2.png 原型链关系图2 %}
## constructor
每个原型都存在一个属性 constructor，该属性指向其关联的构造函数。
尝试运行以下代码
```
function Person() {

}
console.log(Person === Person.prototype.constructor); // true
```
我们可以把 constructor 更新到上述关系图中
{% asset_img image3.png 原型链关系图3 %}
## 原型链
原型其实也是一个对象，既然是对象，那么他的原型又是什么呢？可以运行一下以下的代码
```
function Person(){

}
const person = new Person();
console.log(Person.prototype.__proto__ === Object.prototype) //true
console.log(Object.prototype.__proto__) //null
```
可以发现 ```Person.prototype```对象的原型其实就是 ```Object``` 的原型，换句话说， ```person``` 对象的```__proto__``` 是 ```Person.prototype```，```person``` 的 ```__proto__```的```__proto___```是```Object.prototype```，而 ```Object.prototype``` 的原型 ```__proto__``` 就是 null 了。
最后补充上原型链的访问，关系图可以更新为如下：
{% asset_img image4.png 原型链关系图4 %}

到此，我们可以不妨再回头再重新理解一下这段话“**JavaScript 对象有一个指向一个原型对象的链。当试图访问一个对象的属性时，它不仅仅在该对象上搜寻，还会搜寻该对象的原型，以及该对象的原型的原型，依次层层向上搜索，直到找到一个名字匹配的属性或到达原型链的末尾。**”
# 使用不同的方式来创建对象和生成原型链
## 使用语法结构创建的对象
```
var o = {a: 1};

// o 这个对象继承了 Object.prototype 上面的所有属性
// Object.prototype 的原型为 null
// 原型链如下：
// o ---> Object.prototype ---> null

var a = ["yo", "whadup", "?"];

// 数组都继承于 Array.prototype
// (Array.prototype 中包含 indexOf, forEach 等方法)
// 原型链如下：
// a ---> Array.prototype ---> Object.prototype ---> null

function f(){
  return 2;
}

// 函数都继承于 Function.prototype
// (Function.prototype 中包含 call, bind 等方法)
// 原型链如下：
// f ---> Function.prototype ---> Object.prototype ---> null

const person={
    name: '二毛',
    __proto__: {
        age: 18,
        getMoney: ()=>{
            return "100w"
        }
    }
}
// person这个对象有自己的隐式原型对象，由__proto__主动
// 声明， 所以他的原型链如下：
// person -> { age: 18, getMoney: ()=>{ return "100w" } } -> Object.prototype -> null
```
## 使用构造器创建的对象
在 JavaScript 中，构造器其实就是一个普通的函数。当使用 new操作符 来作用这个函数时，它就可以被称为构造函数。
```
function Person() {
  this.name = '二毛';
  this.age = 18;
  this.deposits = 100
}

Person.prototype = {
  increase: function(v){
    this.deposits+=v;
  }
};

var ermao = new Person();
// ermao 是生成的对象，他的自身属性有 'name' , 'age' 和 'deposits'。
// 在 ermao 被实例化时，ermao.[[Prototype]] 指向了 Person.prototype。添加了一个 increase 方法
```
## 使用 Object.create 创建的对象
ECMAScript 5 中引入了一个新方法：```Object.create()```。可以调用这个方法来创建一个新对象。新对象的原型就是调用 create 方法时传入的第一个参数：
```
var a = {a: 1};
// a ---> Object.prototype ---> null

var b = Object.create(a);
// b ---> a ---> Object.prototype ---> null
console.log(b.a); // 1 (继承而来)

var c = Object.create(b);
// c ---> b ---> a ---> Object.prototype ---> null

var d = Object.create(null);
// d ---> null
console.log(d.hasOwnProperty); // undefined，因为 d 没有继承 Object.prototype
```
## 使用 class 关键字创建的对象
ECMAScript6 引入了一套新的关键字用来实现 class。使用基于类语言的开发人员会对这些结构感到熟悉。但它们仍然是基于 JavaScript 原型实现的，对于 JavaScript 来说属于一种语法糖。这些新的关键字包括 class, constructor，static，extends 以及 super。
```
"use strict";

class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}

class Man extends Person {
  constructor(name, age) {
    super(name, age);
  }
  get name() {
    return this.name;
  }
  set age(age) {
    this.age = age;
  }
}

var zhangsan = new Man('zhangsan', 20);

```
# 原型的性能
在原型链上查找属性比较耗时，对性能有副作用，在性能要求苛刻的情况下考虑较少的遍历会变得很重要。此外，访问不存在的属性时会遍历整个原型链。

遍历对象的属性时，原型链上的每个可枚举属性都会被枚举出来。要检查对象是否具有自己定义的属性，而不是其原型链上的某个属性，可以使用对象从 Object.prototype 继承的 hasOwnProperty方法。（或者Object.keys()方法）
```
console.log(ermao.hasOwnProperty('name')); // true
console.log(ermao.__proto__.hasOwnProperty('increase')); // true
```
有一个小点需要注意的是我们不能通过判断属性是否为 ```undefined``` 来判断其是否存在，因为在某些情况下该属性可能存在，但其值恰好被设置成了 ```undefined```。

# 结论
在使用原型继承编写复杂代码之前，理解原型继承模型是至关重要的。此外，请注意代码中原型链的长度，并在必要时将其分解，以避免可能的性能问题。此外，原生原型不应该被扩展，除非它是为了与新的 JavaScript 特性兼容。


