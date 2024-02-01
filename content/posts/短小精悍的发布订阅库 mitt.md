+++
title = '短小精悍的发布订阅库 mitt'
date = 2023-02-13
+++

## 介绍

`mitt` 是一个小而美的发布-订阅库，短短的几十行代码，小于 `200b` 的体积，提供三个重要的 API。然而麻雀虽小，五脏俱全。

## 发布-订阅模式

发布-订阅模式定义了一种一对多的依赖关系，让多个订阅者对象同时监听某一个主题对象。这个主题对象在自身状态变化时，
会通知所有订阅者对象，使它们能够自动更新自己的状态。

## 用法

有这样的一个需求，小明想买 100w 以内的二手房，小华想买 150w 左右的二手房，但是房产中介告诉他俩，暂时没有他们想要的房源，
中介的工作人员留下他俩的联系方式，一旦有合适的房源就通过电话通知。这就是一个经典使用发布-订阅的模式的场景。请看下面的代码。

```js
const mitt = require("mitt");

const houseAgents = mitt();

houseAgents.on("xiaoming", () => {
  console.log("有 100w 以内的房源了");
});

houseAgents.on("xiaohua", () => {
  console.log("有 150w 左右的房源了");
});
```

过了一段时间，中介收到 150w 左右的房源，立马通知 xiaohua。

```js
houseAgents.emit("xiaohua");
```

## 源码分析

mitt 通过几十行代码实现了发布-订阅机制。我们来剖析一下 [mitt 源码](https://github.com/developit/mitt/blob/master/src/index.js)。

```js
function mitt(all) {
  all = all || Object.create(null);

  return {
    on: function () {
      /* some code*/
    },
    emit: function () {
      /* some code*/
    },
    off: function () {
      /* some code*/
    },
  };
}
```

mitt 库的源码中只有一个 mitt 函数。它接收 all 参数，返回一个对象，该对象包含 on、emit、off 三个方法。

```js
all = all || Object.create(null);
```

mitt 方法接收 `all` 参数，`all` 用来存储监听的事件。当传入的 `all` 的值是 `undefined`、`null`、`""`
、`false`、`NaN` 等值时，all 的值默认为 `Object.create(null)`， 它是一个没有 `__proto__` 属性的对象。

实际上,这里缺少类型处理。比如 mitt(true), 就会导致错误。

```js
const mitt = require("mitt");
const ob = mitt(true);

ob.on("a", () => {});
ob.emit("a", "dd");
```

建议 all 的类型是对象类型或者不传。推荐传入空对象或数组。

```js
const mitt = require("mitt");
const ob = mitt();
/* 
  or
  const ob = mitt([]);
  const ob = mitt({});

*/
```

接下来，我们来看看 mitt 里非常重要的三个方法。

```js
// ...
on: function on(type, handler) {
  (all[type] || (all[type] = [])).push(handler);
}

// ...
```

`on` 接收两个参数，`type` 类型和 `handler`` 事件处理方法。方法体内仅有一行非常优雅的代码。
当该类型存在时，就将其事件处理追加到数组后面，当该类型不存在时，初始化一个空数组，用来
存储该类型的事件处理方法。

```js
emit: function emit(type, evt) {
  (all[type] || []).slice().map(function (handler) {
    handler(evt);
  });
  (all["*"] || []).slice().map(function (handler) {
    handler(type, evt);
  });
}
```

订阅事件使用 `on` 方法，发布事件使用 `emit` 方法。`emit` 方法里就做了一件事，根据事件类型，
将该类型订阅的所有事件遍历调用。

## 不足

通过分析 `mitt` 的源码，我们会发现，`mitt` 没有考虑匿名函数情况，在使用 `on` 方法时，传入的第二个参数必须是具名函数。

## 手动实现发布订阅库

```js
class PubSub {
  constructor() {
    this.listeners = [];
  }

  sub(type, handler, always) {
    console.log(this.listeners[type] || []);
    if (!this.listeners[type]) {
      this.listeners[type] = [];
    }
    this.listeners[type].push({ handler, always });
  }

  on(type, handler, always = true) {
    this.sub(type, handler, always);
  }

  once(type, handler, always = false) {
    this.sub(type, handler, always);
  }

  emit(type, evt) {
    if (this.listeners[type]) {
      this.listeners[type].forEach((listener) => {
        listener.handler(evt);
      });

      this.listeners[type] = this.listeners[type].slice().filter((listener) => Boolean(listener.always));
    }
  }

  off(type, handler) {
    if (this.listeners[type]) {
      this.listeners[type] = this.listeners[type]
        .slice()
        .filter((listener) => listener.handler.toString() !== handler.toString());
    }
  }
}
```