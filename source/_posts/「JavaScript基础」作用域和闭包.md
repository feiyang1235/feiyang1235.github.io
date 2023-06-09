---
title: 「JavaScript基础」作用域和闭包
date: 2023-05-08 21:15:23
tags: JavaScript基础
---
# 作用域
## 概念
- **作用域：** 指代码当前控制着变量和函数的可见性和生命周期。作用是隔离变量，不同作用域下同名变量相同不会冲突。
- **作用域链：** 指如果在当前作用域中没有查到变量，就会向上级作用域查询，直到全局作用域。这样的查找过程就形成了一个查询链条，这样的链条被称之为作用域链。子作用域可以访问父作用域，反之则不行。<br>
作用域具体可细分为四种：**全局作用域、模块作用域、函数作用域、块级作用域**<br>
- **全局作用域**： 代码在程序的任何地方都能被访问，例如 window （浏览器），global（node环境）对象。但全局变量会污染全局命名空间，容易引起命名冲突。在没有模块开发概念的上古前端时代，命名污染是一种常见的错误。
- **模块作用域**： 早期 js 中没有模块的概念，因为脚本简单。后来随着脚本越来越复杂，就出现了模块化方案（AMD、CJS、UMD、ES6模块等）。通常一个模块就是一个文件或者脚本，这个模块拥有独立的模块作用域。
- **函数作用域**： 由函数创建形成的作用域。
- **块级作用域**： js 存在变量提升的概念，形成变量覆盖、变量污染等设计缺陷，所以 ES6 引入了块级作用域关键字来解决这类问题。最经典的案例就是 for 循环。
```
// 变量提升
function foo(){
    a = 10; // 省略声明变量，自动变量提升
}
foo();
console.log(a); // 10
```
```
变量提升
console.log(a); // undefined
var a = 10;
```
```
// var
for(var i=0; i<100; i++) {
    // do anything
}
console.log(i); 
// 运行结果
100
```
```
// let
for(let i=0; i<100; i++) {
     // do anything
}
console.log(i); 
// 运行结果
Uncaught ReferenceError: i is not defined
```
## 和Java对比（闲谈）
当然，Java肯定也有作用域的概念（内心os:废话，什么编程语言没有作用域的概念呢）。**任何程序设计语言都有作用域的概念，简单的说，作用域就是变量与函数的可访问范围，即作用域控制着变量与函数的可见性和生命周期。** 但是Java的控制似乎更加精细。Java程序入口从main函数开始，所以没有全局作用域的概念。
在Java中，我们经常看到```public```、```protected```、```private```这些修饰符。在Java中，这些修饰符可以用来限定访问作用域，这可以类比JS中的模块作用域。而函数作用域和块级作用域的概念是一致的。
### **public**
定义为```public```的```class```、```interface```可以被其他任何类访问：
```
package abc;

public class HelloWorld {
    public void sayHi() {
    }
}
```
上面的HelloWorld是public，因此，可以被其他包的类访问：
```
package xyz;

class Main {
    void foo() {
        // Main可以访问HelloWorld
        HelloWorld h = new HelloWorld();
    }
}
```
定义为```public```的```field```、```method```可以被其他类访问，前提是首先有访问```class```的权限：
```
package abc;

public class HelloWorld {
    public void sayHi() {
    }
}
```
上面的```sayHi()```方法是```public```，可以被其他类调用，前提是首先要能访问HelloWorld类：
```
package xyz;

class Main {
    void foo() {
        HelloWorld h = new HelloWorld();
        h.sayHi();
    }
}
```
### **private**
定义为```private```的```field```、```method```无法被其他类访问：
```
package abc;

public class Person {
    public void borrowMoney(int value) {
        if(value> balance()){
            System.out.println("没钱!")
        }else{
            System.out.println("借给你!")
        }
    }

    // 不能被其他类调用(个人存款，太穷了不能让别人看到)
    private int balance() {
        return 0.01
    }
}
```
确切地说，```private```访问权限被限定在```class```的内部，而且与方法声明顺序无关。推荐把```private```方法放到后面，因为```public```方法定义了类对外提供的功能，阅读代码的时候，应先关注```public```方法。
由于 Java 支持嵌套类，如果一个类内部还定义了嵌套类，那么，嵌套类拥有访问```private```的权限：
```
public class Main {
    public static void main(String[] args) {
        Inner i = new Inner();
        i.sayHi();
    }

    // private方法:
    private static void hello() {
        System.out.println("a private hello method!");
    }

    // 静态内部类:
    static class Inner {
        public void sayHi() {
            Main.hello();// 我是内部类，所以我能访问Main的私有方法
        }
    }
}
```
### **protected**
```protected```作用于继承关系。定义为```protected```的字段和方法可以被子类访问，以及子类的子类：
```
package abc;

public class Person {
    // protected方法:
    protected void sayHi() {
    }
    private void balance() {

    }
}
```
上面的```protected```方法可以被继承的类访问：
```
package xyz;

class Main extends Person {
    void foo() {
        // 可以访问protected方法:
        sayHi();
        // 发生编译错误，不能访问private方法
        balance();
    }
}
```
### **package**
最后，包作用域是指一个类允许访问同一个```package```的没有```public```、```private```修饰的```class```，以及没有```public```、```protected```、```private```修饰的字段和方法。
```
package abc;
// package权限的类:
class Person {
    // package权限的方法:
    void sayHi() {
    }
}
```
只要在同一个包，就可以访问```package```权限的```class```、```field```和```method```：
```
package abc;

class Main {
    void foo() {
        // 可以访问package权限的类:
        Person p = new Person();
        // 可以调用package权限的方法:
        p.sayHi();
    }
}
```
注意，包名必须完全一致

### **final**
Java还提供了一个```final```修饰符。```final```与访问权限存在一定的关联。

用```final```修饰的```class```可以阻止被继承：
```
package abc;

// 无法被继承:
public final class Person {
    private int n = 0;
    protected void sayHi(int t) {
        long i = t;
    }
}
```
```
public Main extends Person {}  // error!,无法继承

```
用```final```修饰```method```可以阻止被子类覆写(override)：
```
package abc;

public class Person {
    // 无法被覆写:
    protected final void sayHi() {
    }
}
```
```
package abc;

public class Main extends Person {
    // error! 无法被覆写:
    protected final void sayHi() {
    }
}
```
用```final```修饰```field```可以阻止被重新赋值：
```
package abc;

public class Person {
    private final int n = 0;
    protected void sayHi() {
        this.n = 1; // error!
    }
}
```
用```final```修饰局部变量可以阻止被重新赋值（类比js中的const）：
```
package abc;

public class Person {
    protected void sayHi(final int i) {
        i = 1; // error!
    }
}
```
### **变量提升**
  Java中没有变量提升的概念<br><br>
# 闭包
## 概念
 **闭包**：**闭包是捆绑在一起（封闭）的函数与其周围状态（词法环境）的引用的组合**。 换句话说，闭包使您可以从内部函数访问外部函数的作用域。 在 JavaScript 中，每次创建函数时都会在函数创建时创建闭包。
可以看出闭包是函数作用域下的产物，闭包会随着外层函数的执行而被同时创建。**它是一个**函数以及其捆绑的周边环境状态的引用的**组合**。换而言之，闭包是内层函数对外层函数变量的不释放。
**闭包的特征：**

- 函数中存在函数；
- 内部函数可以访问外层函数的作用域；
- 参数和变量不会被 GC，始终驻留在内存中；
- 有内存地方才有闭包。

## 词法作用域
考虑以下示例代码：

```
function init(){
   var name = "二毛"; // name 是 init 创建的局部变量
   function displayName(){
     // displayName() 是构成闭包的内部函数
     console.log(name) // 使用在父函数中声明的变量
   }
   displayName();
}
init();
```

其中```init()``` 创建一个名为 ```name``` 的局部变量和一个名为 ```displayName()``` 的函数。 ```displayName()``` 函数是在 ```init()``` 内部定义的内部函数，仅在 ```init()``` 函数体内可用。 请注意，```displayName()``` 函数没有自己的局部变量。 但是，由于内部函数可以访问外部函数的变量，```displayName()``` 可以访问在父函数 ```init()``` 中声明的变量名。

故```displayName()``` 函数中的 ```console.log()``` 语句成功显示了名称变量的值，该变量在其父函数中声明。 这就是一个词法作用域的例子，它描述了解析器如何在函数嵌套时解析变量名。 词法一词指的是词法作用域使用源代码中声明变量的位置来确定该变量可用的位置。 **嵌套函数可以访问在其外部作用域中声明的变量**。

在此示例中，作用域是函数作用域，因为该变量是可访问的，并且只能在声明它的函数体内访问。

## 用 let 和 const 限定作用域
在 ES6 之前，JavaScript 只有两种作用域：函数作用域和全局作用域。 用 var 声明的变量要么是函数作用域的，要么是全局作用域的，这取决于它们是在函数内还是在函数外声明。 因为带有花括号的块不会创建作用域，有些事情看起可能会很麻烦：
```
if (Math.random() > 0.5) {
   var x = 1;
} else {
   var x = 2;
}
console.log(x);
```

对于使用块级创建作用域的其他语言（例如 C、Java）的人来说，上面的代码应该在 ```console.log``` 行上抛出错误，因为 ```x``` 在两个语法块的范围外。 但是，因为块不为 var 创建作用域，所以这里的 var 语句实际上创建了一个全局变量。 下面还介绍了一个实际示例，说明这与闭包结合使用时如何导致实际错误。

在 ES6 中，JavaScript 引入了 let 和 const 声明，其中包括TDZ（Temporal dead zone）等特性，允许您创建块作用域变量。
```
if (Math.random() > 0.5) {
   const x = 1;
} else {
   const x = 2;
}
console.log(x) // ReferenceError: x is not defined
```
本质上，块最终在 ES6 中被视为作用域，但前提是您使用 let 或 const 声明变量。 此外，ES6 引入了模块，引入了另一种作用域(模块作用域)。 闭包能够捕获所有这些范围内的变量，我们将在后面介绍。

## 闭包
考虑以下代码示例：
```
function func() {
   const name = "二毛";
   function displayName() {
     console.log(name);
   }
   return displayName;
}

const myFunc = func();
myFunc();
```
此代码与上面 ```init()``` 函数的前一个示例具有完全相同的效果。 不同的是 ```displayName()``` 内部函数在执行之前从外部函数返回。

乍一看，这段代码仍然有效似乎不符合直觉。 在某些编程语言中，函数内的局部变量仅在函数执行期间存在。 一旦 ```func()``` 完成执行，您可能希望 ```name``` 变量将不再可访问。 然而，代码仍然按预期工作，这显然在 JavaScript 中是不同的情况。

原因是 JavaScript 中的函数形成了闭包。 **闭包是函数和声明该函数的词法环境的组合。** 此环境由创建闭包时范围内的任何局部变量组成。 在这种情况下，```myFunc``` 是对运行 ```func``` 时创建的函数 ```displayName``` 实例的引用。 ```displayName``` 的实例维护对其**词法环境**的引用，其中存在变量名称。 因此，当调用 myFunc 时，变量名仍然可用，并且“二毛”被传递到 ```console.log```。

另一个例子 productAdder 函数：
```
function productAdder(x) {
   return function(y){
     return x + y;
   };
}

const add10 = productAdder(10);
const add20 = productAdder(20);

console.log(add10(1)); // 11
console.log(add20(1)); // 21
```
在这个例子中，我们定义了一个函数 ```productAdder(x)```，它接受一个参数 ```x```，并返回一个新函数。 它返回的函数接收单个参数 ```y```，并返回 ```x``` 和 ```y``` 的总和。```productAdder(x)``` 是一个函数工厂。 它创建可以为其参数添加特定值的函数。 在上面的示例中，函数工厂创建了两个新函数——一个将参数加 10，另一个将参数加 20。

此处```add10``` 和 ```add20``` 都形成闭包。 它们共享相同的函数体定义，但存储不同的词法环境。 在 ```add10``` 的词法环境中，x 是 10，而在 ```add20``` 的词法环境中，x 是 20。

## 闭包作用域链 （Closure scope chain）
每个闭包都有三个作用域：
- 本地作用域
- 封闭作用域（可以是块、函数或模块范围）
- 全局作用域
```
// global scope
const e = 10;
function sum(a) {
    // outer functions scope
  return function (b) {
    // outer functions scope
    return function (c) {
      // outer functions scope
      return function (d) {
        // local scope
        return a + b + c + d + e;
      };
    };
  };
}

console.log(sum(1)(2)(3)(4)); // 20
```
也可以显示指定函数名
```
// global scope
const e = 10;
function sum(a) {
    // outer functions scope
  return function func1(b) {
    // outer functions scope
    return function func2(c) {
      // outer functions scope
      return function func3(d) {
        // local scope
        return a + b + c + d + e;
      };
    };
  };
}

const sum2 = sum(1);
const sum3 = sum2(2);
const sum4 = sum3(3);
const result = sum4(4);
console.log(result); // 20
```
在上面的示例中，有一系列嵌套函数，所有这些函数都可以访问外部函数的范围。 在这种情况下，我们可以说闭包可以访问所有外部函数范围。```local scope->outer functions scope->global scope```便形成一个作用域链。

闭包也可以关闭导入的值，这被视为实时绑定，因为当原始值更改时，导入的值也会相应更改。
```
// moduleX.js
export let x = 1;
export const setX = (value) => {
  x = value;
};
```
```
// closureModule.js
import { x } from "./moduleX.js";

export const getX = () => x; // 实时绑定
```
```
import { getX } from "./closureModule.js";
import { setX } from "./moduleX.js";

console.log(getX()); // 1
setX(2);
console.log(getX()); // 2
```
## 性能
如上所述，每个函数实例管理自己的作用域和闭包。 因此，如果特定任务不需要闭包，则在其他函数中不必要地创建函数是不明智的，因为闭包会在处理速度和内存消耗方面对脚本性能产生负面影响。

例如，当创建一个新的对象/类时，方法通常应该关联到对象的原型而不是定义到对象构造函数中。 原因是无论何时调用构造函数，都会重新分配方法（即，每个对象创建单独的方法）。
举例：
```
function Person(name, age) {
  this.name = name.toString();
  this.age = age.toString();
  this.getName = function () {
    return this.name;
  };

  this.getAge = function () {
    return this.age;
  };
}
```
这种情况其实在多个实例的创建中会在性能上产生负面影响，是不必要的闭包逻辑。改为如下例子：
```
function Person(name, age) {
  this.name = name.toString();
  this.age = age.toString();
}
Person.prototype.getName = function () {
  return this.name;
};
Person.prototype.getAge = function () {
  return this.age;
};
```
继承的原型可以由所有对象共享，并且方法定义不需要在每个对象创建时都出现。
## 闭包使用场景
闭包很有用，因为它们可以让您将数据（词法环境）与操作该数据的函数相关联。 这与面向对象的编程有明显的相似之处，其中对象允许您将数据（对象的属性）与一个或多个方法相关联。

因此，您可以在通常只使用一个方法的对象的任何地方使用闭包。
### 异步场景
```
// demo1 输出 3 3 3
for(var i = 0; i < 3; i++) {
    setTimeout(function() {
        console.log(i);
    }, 1000);
} 
```
```
// demo2 输出 0 1 2
for(let i = 0; i < 3; i++) {
    setTimeout(function() {
        console.log(i);
    }, 1000);
}
```
demo1 输出结果为:3 3 3，因为 ```var``` 定义的变量i会发生变量提升，所以实际i是一个全局作用域的变量，同时 ```setTimeout``` 是一个异步回调。因此，当执行 ```setTimeout``` 时， ```i``` 的值已经是 3，导致 3 3 3 被打印到控制台。 demo2 输出结果为：0 1 2， ```let``` 关键字在 for-loop 中形成块级作用域，为循环的每次迭代创建一个新绑定，因此每个 ```setTimeout``` 都可以访问 ```i``` 的正确值。
### 用闭包模拟私有方法
Java 等语言允许您将方法声明为私有的，这意味着它们只能由同一类中的其他方法调用。

在类出现之前，JavaScript 没有声明私有方法的本机方式，但可以使用闭包模拟私有方法。 私有方法不仅仅用于限制对代码的访问。 它们还提供了一种管理全局名称空间的强大方法。

下面的代码说明了如何使用闭包来定义可以访问私有函数和变量的公共函数。
```
const counter = (function () {
  let _counter = 0;
  function change(value) {
    _counter += value;
  }

  return {
    increment() {
      change(1);
    },

    decrement() {
      change(-1);
    },

    value() {
      return _counter;
    },
  };
})();

console.log(counter.value()); // 0.
counter.increment();
counter.increment();
console.log(counter.value()); // 2.
counter.decrement();
console.log(counter.value()); // 1.
```
在之前的示例中，每个闭包都有自己的词法环境。 不过，这里有一个由三个函数共享的词法环境：counter.increment、counter.decrement 和 counter.value。

共享词法环境是在匿名函数主体中创建的，该函数在定义后立即执行（也称为 IIFE）。 词法环境包含两个私有项：一个为 ```_counter``` 的变量和一个称为 ```change``` 的函数。 您不能从匿名函数外部访问这些私有成员。 代替的是，您可以使用从匿名包装器返回的三个公共函数来访问它们。

这三个公共函数形成了共享相同词法环境的闭包。 由于 JavaScript 的词法作用域，它们都可以访问 ```_counter``` 变量和 ```change``` 函数。<br>
我们也可以通过改写IIFE的格式，来形成多实例各自共享词法环境。
```
const productCounter = function () {
  let _counter = 0;
  function change(value) {
    _counter += value;
  }
  return {
    increment() {
      change(1);
    },

    decrement() {
      change(-1);
    },

    value() {
      return _counter;
    },
  };
};

const counter1 = productCounter();
const counter2 = productCounter();

console.log(counter1.value()); // 0.
counter1.increment();
counter1.increment();
console.log(counter1.value()); // 2.
console.log(counter2.value()); // 0.

counter1.decrement();
console.log(counter1.value()); // 1.
console.log(counter2.value()); // 0.
```
counter1 和 counter2 形成两个独立的闭包，他们共享独立的词法环境，在一个闭包中更改变量值不会影响另一个闭包中的值。

## 和Java对比（闲谈）
Java 没有真正可以类比的闭包概念， groovy倒是有Closure的语法。不过都是基于JVM的语言，真正还是生成了匿名内部类。Java8的Lambda表达式可能和闭包写法相同。
Java8之前
```
String[] oldWay = "Improving code with Lambda expressions in Java 8".split(" ");
Arrays.sort(oldWay, new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        // 忽略大小写排序:
        return s1.toLowerCase().compareTo(s2.toLowerCase());
    }
});
System.out.println(String.join(", ", oldWay));
```
对于只有一个方法的接口，在Java 8中，现在可以把它视为一个函数，用lambda表示式简化如下：
```
String[] newWay = "Improving code with Lambda expressions in Java 8".split(" ");
Arrays.sort(newWay, (s1, s2) -> {
    return s1.toLowerCase().compareTo(s2.toLowerCase());
});
System.out.println(String.join(", ", newWay));
```
实际上，lambda表达式最终也被编译为一个实现类，不过语法上做了简化。
# 总结
作用域和闭包是 JavaScript 中基础且非常重要的概念，作用域决定变量与函数的可见性和生命周期。这对理解内存模型，内存回收，以及内存泄漏相关的问题很有帮助。而闭包是函数作用域下的产物，闭包会随着外层函数的执行而被同时创建，它是一个函数以及其捆绑的周边环境状态的引用的组合。换而言之，闭包是内层函数对外层函数变量的不释放，合理的使用闭包可以写出更为优雅的代码。