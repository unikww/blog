+++
title = '深入浅出 Server-sent events 技术'
date = 2023-02-07
+++

## 前言

实时获取服务端的数据，大家第一时间想到的是轮询和 WebSocket 两种方案，其实还有一种新方案 Server-sent events
下文简称（SSE）。SSE 中的数据只能由服务端推向客户端

SSE 是基于 http 协议的服务器推送技术，数据只能从服务端到客户端。服务端把序列化后的数据发送给客户端，
整个过程持续不断直至连接关闭

## WebSocket vs 轮询 vs SSE

下面是 WebSocket、轮询和 SSE 的功能对比

- SSE 和轮询使用 HTTP 协议，现有的服务器软件都支持。WebSocket 是一个独立协议
- SSE 属于轻量级的 WebSocket，使用简单；WebSocket 使用相对复杂，轮询使用简单
- SSE 默认支持断线重连，WebSocket 需要自己实现断线重连
- SSE 一般只用来传送文本，二进制数据需要编码后传送，WebSocket 默认支持传送二进制数据
- SSE 支持自定义发送的消息类型
- WebSocket 支持双向推送消息，SSE 是单向的
- 轮询性能开销大、轮询时间久导致客户端及时更新数据

## 使用场景

基于服务端单向的向客户端推送信息的特性，SSE 使用场景主要有

- Sass 平台的消息通知
- 信息流网站实时更新数据

## 使用方式

下面讲解如何在客户端使用 SSE

1. 创建一个 `EventSource` 实例，向服务器发起连接

```js
const evtSource = new EventSource();
```

2. 自定义事件

对于自定义事件，服务端和客户端一定要保持事件名一致。服务端通过自定义事件发送数据，
就会触发自定义事件。SSE 默认支持 `message` 事件，下面以 `message` 事件为例

```js
evtSource.addEventListener("message", (event) => {
  let payload;

  try {
    payload = JSON.parse(event.data); // <--- event.data 需要反序列化
    console.log("receiving data...", payload);
  } catch (error) {
    console.error("failed to parse payload from server", error);
  }
});
```

自定义事件的回调函数接收 `event` 对象，`event.data` 存着服务端发给客户端的数据但是需要反序列化

可以通过 Chrome Devtool 工具查看 `eventsource` 通信情况，如图所示

![EventSource](https://cdn.jsdelivr.net/gh/qinghuanI/blog-images@main/uPic/debugger_eventsource.png)

- **`1`** - 自定义事件名，服务端和客户端需要保持一致
- **`2`** - EventStream Tab，数据都在这里
- **`3`** - 服务端推送给客户端的数据

3. 错误处理

如果连接发生错误，就会触发 `error` 事件

```js
evtSource.addEventListener("error", (err) => {
  console.error("EventSource failed:", err);
});
```

4. 关闭连接

SSE 提供 `close` 方法，用来关闭 SSE 连接

```js
evtSource.close();
```

## 浏览器兼容性

通过 [caniuse](https://caniuse.com/?search=server-sent%20events) 查看 SSE 浏览器兼容性，如图所示

![Server-sent events](https://cdn.jsdelivr.net/gh/qinghuanI/blog-images@main/uPic/Server-sent_events_browser_compatible.png)

除了 IE 浏览器不支持，其它现代浏览器都支持，所以放心大胆在项目中使用 SSE

## 简单封装

在平常的工作中，每次写 SSE 的事件监听和错误处理会很麻烦。多个业务场景需要使用 SSE 时，就需要对 SSE 进行封装。接下来我们尝试封装一个简单的 SSE SDK，方便在项目中使用

当我们决定写 SSE 的 SDK 时，首先想到使用面向对象（OOP）进行封装。根据 SSE 的特性，那么库需要实现 `subscribe` 和 `unsubscribe`
两个方法。通过确定 `SSE` 库使用方式，根据使用方式确定 SDK 的实现。我们可以在代码中这样使用，如下所示

```js
// SSESdk 实例化
const SSE = new SSESdk(url, options);

// 订阅来自服务端的消息
SSE.subscribe("message", (data) => {
  console.log("receive message from server", data);
});

// 取消订阅
SSE.unsuscribe();
```

我们要封装的库对外仅仅提供 `subscribe` 和 `unsubscribe` 两个 Api，非常方便开发人员使用。
`subscribe` 用来订阅来自服务端的消息， `unsubscribe` 用来取消订阅，关闭 SSE 连接，通过使用形式可以看出，使用 ES6 中的类语法。接下来我们先确定 SSE SDK 的大体结构

```js
class SSEClient {
  constructor() {}

  subscribe(type, handler) {}

  unsubscribe() {}
}
```

在 `SSEClient` 类中有三个方法需要实现，通过 constructor 接受可配置的参数，比如 SSE 建立连接失败后的重试次数和重试时间。
`subscribe` 接收一个与后端保持一致的事件名和一个回调函数。`unsubscribe` 不需要传递任何参数，调用 `unsubscribe` 方法关闭
SSE 连接

```js
// SSE-client.js

class SSEClient {
  constructor(url) {
    this.url = url;
    this.es = null;
  }

  subscribe(type, handler) {
    this.es = new EventSource(url);

    this.es.addEventListener("open", () => {
      console.log("server sent event connect created");
    });

    this.es.addEventListener(type, (event) => {
      let payload;

      try {
        payload = JSON.parse(event.data);
        console.log("receiving data...", payload);
      } catch (error) {
        console.error("failed to parse payload from server", error);
      }

      if (typeof handler === "function") {
        handler(payload);
      }
    });

    this.es.addEventListener("error", () => {
      console.error("EventSource connection failed for subscribe.Retry");
    });
  }

  unsubscribe() {
    if (this.es) {
      this.es.close();
    }
  }
}
```

就这样实现了一个简单的 `SSE` SDK。首先根据 url 参数创建一个 `SSEClient` 实例，当调用 `subscribe` 方法时，才会根据传入的 url 建立 `SSE` 连接，然后监听对应的事件，一旦
连接建立成功，后端向客户端发送数据，就可以从 `handler` 方法中拿到数据

这个库仅仅实现了非常基本的功能，代码封装上存在很多问题。比如 `es` 的事件全部杂糅在 `subscribe` 方法中、缺少 `SSE` 连接建立失败的重试等等功能。接下来我们对刚刚实现的 `SSEClient` SDK 进行优化

```js
const defaultOptions = {
  retry: 5,
  interval: 3 * 1000,
};

class SSEClient {
  constructor(url, options = defaultOptions) {
    this.url = url;
    this.es = null;
    this.options = options;
    this.retry = options.retry;
    this.timer = null;
  }

  _onOpen() {
    console.log("server sent event connect created");
  }

  _onMessage(handler) {
    return (event) => {
      this.retry = options.retry;
      let payload;

      try {
        payload = JSON.parse(event.data);
        console.log("receiving data...", payload);
      } catch (error) {
        console.error("failed to parse payload from server", error);
      }

      if (typeof handler === "function") {
        handler(payload);
      }
    };
  }

  _onError(type, handler) {
    return () => {
      console.error("EventSource connection failed for subscribe.Retry");
      if (this.es) {
        this._removeAllEvent(type, handler);
        this.unsubscribe();
      }

      if (this.retry > 0) {
        this.timer = setTimeout(() => {
          this.subscribe(type, handler);
        }, this.options.interval);
      } else {
        this.retry--;
      }
    };
  }

  _removeAllEvent(type, handler) {
    this.es.removeEventListener("open", this._onOpen);
    this.es.removeEventListener(type, this._onMessage(handler));
    this.es.removeEventListener("error", this._onError(type, handler));
  }

  subscribe(type, handler) {
    this.es = new EventSource(url);

    this.es.addEventListener("open", this._onOpen);
    this.es.addEventListener(type, this._onMessage(handler));
    this.es.addEventListener("error", this._onError(type, handler));
  }

  unsunbscribe() {
    if (this.es) {
      this.es.close();
      this.es = null;
    }
    if (this.timer) {
      clearTimeout(this.timer);
    }
  }
}
```

我们将 `SSEClient` 中的三个事件方法分别提取为三个私有方法，`_onOpen` 方法在 event 触发 open 时调用，向控制台输出链接已经创建。
`_onMessage` 方法在后端向前端发送数据时触发，负责解析数据，并调用 `handler` 方法。`_onError` 方法在 SSE 发生错误时触发，
会在控制台输出错误的提示，根据开发者传入的重试次数，先关闭上一次的 SSE 链接，取消所有的事件监听，关闭定时器，
再开启递归调用 `subscribe` 方法进行重连， 一旦重连成功，重试次数恢复为设定的重试次数，如果超过重试次数依旧没有连接成功，那么 `SSE` 会彻底终止。需要开发人员排查具体原因

一个可以用在项目上的简单 `SSE` SDK 封装完

## 第三方库

`SSE` 虽然很好，但是也有它先天不足，主要问题是不能通过 `headers` 传递 `Authorization token`。虽然可以把 token 放在 url 上
解决不能传 `token` 的问题，但是又会引发 `token` 安全隐患。所以社区里有使用 `xhr` 和 `fetch` 模拟原生 `Server-sent events` 的功能，解决不能
通过 `headers` 传递 `Authorization token` 的问题。主要有两个第三方库，分别是 `eventsource` 和 `event-source-polyfill`，
下面笔者详细讲述这两个库的使用

### eventsource

此库是 EventSource 客户端的纯 JavaScript 实现。使用方式很简单。在项目中安装依赖

```shell
yarn add eventsource
# Or npm install eventsource
```

然后从 `eventsource` 中导出 `EventSource` 类，然后实例化得到 `es` 实例

```js
import EventSource from "eventsource";

const eventSourceInitDict = { headers: { authorization: "Bearer token" } };
const es = new EventSource(url, eventSourceInitDict);

es.addEventListener("message", (event) => {
  console.log("receiving data from server:", JSON.parse(event.data));
});
```

`eventsource` 的实现用到了一些 `node` 标准库。分别是 `https` 和 `http`。
笔者将 `eventsource` 的部分源码列在下面。

```js
// eventsource.js 源码如下

const https = require("https");
const http = require("http");
```

然而，浏览器环境并不支持 `https` 和 `http` 标准库。所以当我们在浏览器环境中使用 `eventsource` 时，需要做一些额外的工作。下面以 webpack5 为例子讲解解决办法

- 需要在 `webpack` 配置文件中添加 `node-polyfill-webpack-plugin` 插件

```shell
yarn add node-polyfill-webpack-plugin -D
```

然后在 `webpack` 配置文件使用该插件

```js
// 项目中的 webpack 配置文件，比如 webpack.config.js

const NodePolyfillPlugin = require("node-polyfill-webpack-plugin");

module.exports = {
  // Other rules...
  plugins: [new NodePolyfillPlugin()],
};
```

- 或者在 `webpack` 的 `callback` 中对使用的库进行单独的配置

```js
module.exports = {
  // other configuration ...
  resolve: {
    fallback: {
      https: false,
      http: false,
    },
  },
};
```

做完上面的步骤后，`eventsource` 可以在浏览器中正常运行

如果不想改动 `webpack` 的配置，那么可以试试 `event-source-polyfill` 这个库

### event-source-polyfill

event-source-polyfill 的使用非常简单，使用 `EventSourcePolyfill` 替换原生的 `EventSource`

```js
import { EventSourcePolyfill } from "event-source-polyfill";

var es = new EventSourcePolyfill(url, {
  headers: {
    authorization: "Bearer token",
  },
});

es.addEventListener("message", (event) => {
  console.log("receiving data from server:", JSON.parse(event.data));
});
```

### 不足之处

`eventsource` 和 `event-source-polyfill` 只是在一定的程度上解决了 `Authorization token` 的问题，但它们也存在问题。
这两个库提供的 `close` 方法只能关闭处于 `pending` 状态的 SSE 连接，因为 fetch 一旦从 `pending` 变为 `resolved`
或 `reject`, 其结果无法改变。当频繁的断开 SSE 连接和建立新 SSE 连接时，旧的 SSE 连接实际上并没有关闭，系统里会存在多个
SSE 连接，这样会带来很大的性能开销

## FAQ

1. SSE 不能向服务端发送数据？

可以将数据放入 `url` 中，断开当前的 SSE 连接，根据新 url 重新建立 SSE 连接

## 总结

本篇文章讲述一种服务端向客户端推送信息的技术、它比 `WebSocket` 更简单更轻量化，比轮询性能好。
简单介绍 `Server-sent events` 的技术原理和使用场景，并进行简单的封装，方便日常在项目中使用。推荐使用 `eventsource` 和 `event-source-polyfill` 第三方库解决不能通过 `headers` 传递 `Authorization token` 的问题。

## 参考链接

- [Server-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events)