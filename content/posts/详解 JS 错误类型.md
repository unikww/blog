+++
title = '详解 JS 错误类型'
date = 2020-05-26
+++

## 前言

bug 是应用程序的重要组成部分，编码过程也是发现 bug，然后解决 bug 的过程。认识和掌握 JavaScript 执行过程中抛出的错误类型，有助于快速定位 bug、解决 bug，写出一个健壮的 JavaScript 程序。

## JS 引擎解析流程

JS 引擎解析 JavaScript 代码的过程，分为三个阶段。

### 词法语法分析

- 词法分析

JS 引擎先把 javascript 代码的字符流(网络传输的字节流经过解码后)按照 ECMAScript 标准转换为 `Tokens`。

使用 `esprima` 分析库对下面这行代码进行解析

```js
const num = 10;
```

`const num = 10;` 这段字符串转换为对象数组格式的 Tokens 流，如下:

```js
[
  { type: "Keyword", value: "const" },
  { type: "Identifier", value: "num" },
  { type: "Punctuator", value: "=" },
  { type: "Numeric", value: "10" },
  { type: "Punctuator", value: ";" },
];
```

- 语法分析

JS 引擎在经过词法分析后，将 Tokens 流按照 ECMAScript 语法标准把词法分析所产生的记号生成语法树。通俗地说就是把从程序中收集的信息存储到数据结构中，每取一个 Token，就送入语法分析器进行分析。

解析看看 `esprima` 分析库对下面这行代码进行解析

```js
Script {
  type: 'Program',
  body: [
    VariableDeclaration {
      type: 'VariableDeclaration',
      declarations: [Array],
      kind: 'const'
    }
  ],
  sourceType: 'script'
}
```

分析该 js 脚本代码块的语法是否正确，如果出现不正确会向外抛出一个 SyntaxError(语法错误),并且停止 js 代码的执行。反之，继续分析之后的代码块，分析完毕，进入到预编译阶段。

### 预编译阶段

js 代码块通过语法分析阶段后，语法正确则进入预编译阶段。变量声明及函数声明提升发生在预编译阶段。

### 执行阶段

执行阶段涉及事件循环机制。下文讲到的异常(五种类型的错误)发生在预编译阶段或执行阶段。

> 总结:在浏览器环境下，JS 引擎首先按照顺序加载 `<script>` 标签分割的 js 代码块，加载 js 代码块完毕后，立刻进入以上三个阶段，然后再按顺序查找下一个代码块，再继续执行以上三个阶段，无论是外部脚本文件(不异步加载)还是内部脚本代码块，都是一样的原理，并且都在同一个全局作用域中。

## 语法错误

JS 引擎解析 js 代码块时，先进行词法语法分析，若发现不符合语法规范的 token 或 token 顺序时抛出 SyntaxError(语法错误)，会导致整个 js 文件无法执行。

在浏览器环境下。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Error</title>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
  </head>
  <body>
    <script>
      const author = "qinghuanI";

      console.log([); // Uncaught SyntaxError: Unexpected token ')'
    </script>
    <script>
      console.log(123456); // 123456
    </script>
  </body>
</html>
```

JS 引擎加载完第一个`<script>` 里的代码片段，立马进入上述说的三个阶段，先进行词法语法解析，若没有语法错误则进入预解析阶段，直至最后执行完毕。若遇到语法错误，停止解析，并向外抛出语法错误，然后跳出该`<script/>` 代码片段，加载第二个`<script/>` 代码片段，进入那三个阶段。

导致语法错误的代码有很多。随便写几个。

```js
var = 1; // SyntaxError: Unexpected token '='


{ // SyntaxError: Unexpected end of input
```

## 异常

异常包含很多种错误。属于异常的错误会导致在错误出现的那一行之后的代码无法执行，但在那一行之前的代码不会受到影响。主要发生在预解析和执行阶段。

1. Uncaught ReferenceError(引用错误)

引用一个不存在的变量时发生的错误。将一个值分配给无法分配的对象，比如对函数的运行结果或者函数赋值。

```js
// 使用未声明的变量
console.log(author); // ReferenceError: author is not defined

// 给函数调用赋值
const foo = () => {};

foo() = 123; // ReferenceError: Invalid left-hand side in assignment
```

2. RangeError(范围错误)

RangeError 是当一个超出有效范围时发生的错误。主要的有几种情况，第一是数组长度为负数，第二是 Number 对象的方法参数超出范围，以及函数堆栈超过最大值。

```js
// 数组长度为负数
[].length = -1; // RangeError: Invalid array length

// Number 对象的方法参数超出范围
const num = new Number(12.34);
console.log(num.toFixed(-1)); // Uncaught RangeError: toFixed() digits argument must be between 0 and 20 at Number.toFixed

// 函数堆栈超过最大值
const foo = () => foo();
foo(); // RangeError: Maximum call stack size exceeded
```

3. TypeError(类型错误)

值的类型或参数不是预期类型时发生的错误。比如使用 new 字符串、布尔值等原始类型和调用对象不存在的方法就会抛出这种错误，因为 new 命令的参数应该是一个构造函数。

```js
// 调用不存在的方法
const person = {};
person.run(); // TypeError: person.run is not a function

// new 操作符后面不是构造函数
const instance = new 2(); // TypeError: 2 is not a constructor
```

4. URIError(URL 错误)

使用全局 URI 处理函数而产生的错误。

```js
decodeURI("%"); // URIError: URI malformed

decodeURIComponent("%"); // URIError: URI malformed
```

URI 相关参数不正确时抛出的错误，主要涉及 `decodeURI` 和 `decodeURIComponent` 两个函数。

5. EvalError(Eval 错误)

<strong>ES3</strong> 中有 `EvalError` 错误类型，然而没有被列入 <strong>ES5</strong> 以后的规范，但目前基于相容性，`EvalError` 有保留
下来，不过现在没有标准 API 会抛出 `EvalError` 。所以暂且不用详细谈论 `EvalError`。

## try...catch

来自 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/try...catch)的解释。

> `try...catch` 语句标记要尝试的语句块，并指定一个出现异常时抛出的响应。

能被 `try...catch` 捕捉到的异常，必须是在报错的时候，JS 引擎执行已经进入 `try...catch` 代码块，且处在 `try...catch` 里面，这个时候才能被捕捉到。

- 无法捕捉语法错误

在 **JS 引擎解析流程** 讲到，JS 引擎首先对加载完的代码进行词法语法分析，语法分析阶段若有错误，直接抛出错误，并停止解析，执行流并未进入 `try...catch`，自然不会捕捉错误。

- 无法捕捉异步错误

```js
try {
  new Promise((resolve, reject) => {reject(new Error('error'))})
} catch (err) {
  console.error(err);
}}

// UnhandledPromiseRejectionWarning: Error: error
```

因为异步错误并不在 `try...catch` 代码块中，所以无法捕捉异步错误。因此，对于 Promise 等异步错误，使用 `catch` 方法捕捉。

```js
new Promise((resolve, reject) => {
  reject(new Error("error"));
}).catch((err) => console.error(err)); // Error: error
```

## 参考链接

- [详解 JavaScript 中的六种错误类型](https://www.jb51.net/article/124210.htm)
- [面试官：请用一句话描述 try catch 能捕获到哪些 JS 异常](https://juejin.im/post/6844904143891464200)
- [js 引擎的执行过程](https://heyingye.github.io/2018/03/26/js引擎的执行过程（二）/)