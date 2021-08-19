---
title: 在构造函数中调用虚函数
date: 2021-08-18T14:36:13+08:00
---

## 简介

虚函数是指派生类中重新定义的成员函数。如有 `A#fromJSON`，当 B 继承 A 时并重新定义了 `B#fromJSON`，这时候 `fromJSON` 就是一个虚函数了。
虚函数在实现多态时非常有用，但在构造函数中调用虚函数却会带来很多问题。
最近在这个问题中碰壁多次，决定写一篇总结。


## 动机

最开始我是希望提供两个 API：

- `new A(options)` 根据传入参数进行实例化
- `A#fromJSON(options)` 根据传入参数修改当前实例的属性

由于这两个 API 都需要检查输入，并设置当前实例的属性，于是我很自然的就在构造函数中调用了 `fromJSON` 方法。
这两个 API 最开始还能工作得比较好，但随着我的类之间的依赖变得越来越复杂，这种做法的问题暴露出越来越多的问题。

## 问题

### 访问未完成实例化的对象

先来看看下面的 typescript 示例代码：

```ts
class Color {
    r = 0;
    g = 0;
    b = 0;

    fromJSON(options: any) {
        if (options.r) this.r = options.r;
        if (options.g) this.g = options.g;
        if (options.b) this.b = options.b;
    }
}

class A {
    isA = true;

    constructor(options: any) {
        this.fromJSON(options);
    }

    fromJSON(options: any) {
        if (options.isA) this.isA = options.isA;
    }
}

class B extends A {
    color = new Color();

    fromJSON(options: any) {
        super.fromJSON(options);
        if (options.color) this.color.fromJSON(options.color);
    }
}

new B({ color: { r: 1, g: 1, b: 1 } })
```

这是很正常的写法，B 继承了 A，然后在初始化的时候去调用 `B#fromJSON` 去设置自己的属性。
然而当我们运行的时候却会报下面的错误：

```text
runtime.ts:178 TypeError: Cannot read property 'fromJSON' of undefined
    at B.fromJSON (eval at <anonymous> (runtime.ts:154), <anonymous>:35:24)
    at new A (eval at <anonymous> (runtime.ts:154), <anonymous>:20:14)
    at new B (eval at <anonymous> (runtime.ts:154), <anonymous>:29:9)
    at eval (eval at <anonymous> (runtime.ts:154), <anonymous>:38:1)
```

初看报错会一脸懵逼，但看一看 build 出来的 javascript 代码：

```js
class Color {
    // ...
}

class A {
    constructor(options) {
        this.isA = true;
        this.fromJSON(options);
    }
    fromJSON(options) {
        // ...
    }
}

class B extends A {
    constructor() {
        super(...arguments);
        this.color = new Color();
    }
    fromJSON(options) {
        super.fromJSON(options);
        if (options.color) this.color.fromJSON(options.color);
    }
}
```

可以看到，在 B 的 `constructor` 里面的 `super(...arguments)` 里面执行了 `this.fromJSON`，在这之前 B 的 `color` 属性还没有初始化，因此在调用 `B#fromJSON` 的时候就会报错了。
这是一个非常典型的错误， 在 stackoverflow 中的[一篇答案](https://stackoverflow.com/a/5230637/2331095)所说的：

> You shouldn't: calling instance method in constructor is dangerous because the object is not yet fully initialized (this applies mainly to methods than can be overridden). Also complex processing in constructor is known to have a negative impact on testability.

在实例化的过程中调用实例方法是一个非常危险的做法，因为该实例还没有完成初始化流程，在实例化时执行复杂的处理会影响实例的可测试性。


### 没有正确的调用时机

上面的例子中，我们看到了`this.fromJSON` 的调用时机不正确导致了代码报错，要解决上面例子的报错，只需要调整一下 `this.fromJSON` 的调用时机即可，也就是说在初始化 `color` 之后再调用即可。

但这种做法只是适用上面的例子而已，下次再加一个 `class C extends B` 一样会报类似的错误：

```ts
class A {
    isA = true;
    fromJSON(options: any) { /* ... */ }
}

class B extends A {
    color: Color;
    constructor(options: any) {
        super(options);
        this.color = new Color();
        this.fromJSON(options);
    }
    fromJSON(options: any) { /* ... */ }
}

class C extends B {
    color2 = new Color();
    fromJSON(options: any) {
        super.fromJSON(options);
        if (options.color2) this.color2.fromJSON(options.color2);
    }
}

new C({ color2: { r: 1, g: 0, b: 1 } });
```

```text
TypeError: Cannot read property 'fromJSON' of undefined
    at C.fromJSON (eval at <anonymous> (runtime.ts:154), <anonymous>:46:25)
    at new B (eval at <anonymous> (runtime.ts:154), <anonymous>:30:14)
    at new C (eval at <anonymous> (runtime.ts:154), <anonymous>:40:9)
    at eval (eval at <anonymous> (runtime.ts:154), <anonymous>:49:1)
```

看吧，同样的报错。


### 容易重复调用实例方法

当我们希望基类及其派生类都有同样的行为时，我们很自然的会往基类的构造函数中调用一些实例方法。但当我们一不留神的时候，很容易会重复调用：

```ts
class A {
    constructor(options: any) {
        this.fromJSON(options);
    }

    fromJSON(options: any) {
        if (options.A) console.log('Checking prop A');
        console.log('I am in A');
    }
}

class B extends A {
    fromJSON(options: any) {
        super.fromJSON(options);
        if (options.B) console.log('Checking prop B');
        console.log('I am in B');
    }
}

class C extends B {
    constructor(options: any) {
        super(options);
        this.fromJSON(options);
    }

    fromJSON(options: any) {
        super.fromJSON(options);
        if (options.C) console.log('Checking prop C');
        console.log('I am in C');
    }
}

new C({ A: 1, B: 1, C: 1 });
```

```text
[LOG]: "Checking prop A" 
[LOG]: "I am in A" 
[LOG]: "Checking prop B" 
[LOG]: "I am in B" 
[LOG]: "Checking prop C" 
[LOG]: "I am in C" 
[LOG]: "Checking prop A" 
[LOG]: "I am in A" 
[LOG]: "Checking prop B" 
[LOG]: "I am in B" 
[LOG]: "Checking prop C" 
[LOG]: "I am in C" 
```

这是由于我们在 `C` 的 `constructor` 中也调用了一次 `this.fromJSON`，换句话说，如果我们不清楚父级的 `constructor` 实现的话，就会很容易发生类似的事情。

但有没有什么优雅的做法去解决这一类问题呢？ 很遗憾，并没有。

最好的解决办法是不要在构造函数中调用虚函数。


## 参考资料

- [虚函数](https://docs.microsoft.com/zh-cn/cpp/cpp/virtual-functions?view=msvc-160)
- [Calling virtual functions inside constructors](https://stackoverflow.com/questions/962132/calling-virtual-functions-inside-constructors)
- [Can I call methods in constructor in Java](https://stackoverflow.com/questions/5230565/can-i-call-methods-in-constructor-in-java)
