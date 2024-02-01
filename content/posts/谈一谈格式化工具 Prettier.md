+++
title = '谈一谈格式化工具 Prettier'
date = 2020-05-07
+++

## 前言

工欲善其事必先利其器，一个优秀的工具可以显著提高生产力。前端领域的工具并不多，常用的有 ESLint、stylelint、Prettier 等等。本篇博客简单谈一谈 Prettier。

## Prettier 是什么？

Prettier 是一个强约束的代码格式化工具，将原始格式的源代码按照设定的规则进行格式化，
然后输出格式化后的代码。支持常见的 `js`、`jsx`、`html`、`json` 等等编程语言或数据格式。

## Prettier 解决什么问题？

Prettier 是前端领域的一个代码格式化工具，它主要解决前端人员编码过程中存在的代码格式化问题，具体为:

- 保持团队协作中代码风格统一
- 自动化格式化代码，开发人员的精力放在业务上

## 自定义配置

Prettier 支持开发人员自定义 Prettier 提供的规则。推荐使用自定义配置文件。

1. 在 `package.json` 里添加 `prettier` 字段。

```js
{
  "name": "xxx-product",
  "version": "0.1.0",
  "description": "",
  "license": "UNLICENSED",
  // ....

  "prettier": {
    "tabWidth": 2,
  }
}
```

2. 在项目的根目录下添加 `.prettierrc` 或 `.prettierrc.js` 或 `.prettier.config.js` 和 `.prettierrc.toml` 文件。

JSON:

```json
{
  "trailingComma": "es5",
  "tabWidth": 2,
  "semi": false,
  "singleQuote": true
}
```

JS:

```js
// prettier.config.js or .prettierrc.js
module.exports = {
  trailingComma: "es5",
  tabWidth: 2,
  semi: false,
  singleQuote: true,
};
```

YAML:

```yaml
# .prettierrc or .prettierrc.yaml
trailingComma: "es5"
tabWidth: 2
semi: false
singleQuote: true
```

TOML:

```toml
# .prettierrc.toml
trailingComma = "es5"
tabWidth = 2
semi = false
singleQuote = true
```

## 具体用法

下面以 Visual Studio Code 为开发工具，讲讲如何使用 Prettier。

1. vscode 使用 prettier 扩展，结合 `.prettierrc` 配置文件格式化代码

Visual Studio Code 用户需要到插件市场里下载安装 [Prettier - Code formatter
](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)此时，但凡使用 vscode 打开某一个项目, 按下 command + S(mac 上), Prettier 会以默认的配置格式化所有的项目。(全局性的格式化)

对于不需要格式化的文件，可以使用 `.prettierignore` 文件进行忽略。比如:

```sh
node_modules
dist
```

在 **自定义配置** 这一节讲到 Prettier 允许开发者自定义选项。推荐在项目根目录下添加
`.prettierrc` 文件，对具体的选项自定义。推荐的自定义配置如下

```json
{
  "printWidth": 80,
  "tabWidth": 2,
  "semi": true,
  "singleQuote": false,
  "tabs": false,
  "trailingComma": "none"
}
```

2. 在全局环境中安装 `prettier`。即使用 CLI 方式来格式化代码。

```sh
npm install prettier -g
```

然后在终端里输入 `prettier`, 将会看到下列内容。


![Prettier](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/5/7/171ee02773eb2e87~tplv-t2oaga2asx-image.image)

使用 Prettier CLI 的格式为

```sh
prettier [options] [file/dir/glob ...]
```

CLI 提供了输出选项和格式化选项。来看个例子。

```js
// index.js

const author = 'qinghuanI';
```

在终端里执行 `prettier --single-quote false --write index.js`。执行成功后，会在 index.js 里看到。

```js
// index.js

const author = "qinghuanI";
```

3. 在浏览器环境或 Node.js 环境下，使用 API 格式化代码。

浏览器环境下:

```html
<script src="https://unpkg.com/prettier@2.0.4/standalone.js"></script>
<script src="https://unpkg.com/prettier@2.0.4/parser-graphql.js"></script>
<script>
  prettier.format("foo()", {
    semi: true,
    parser: "babel",
  });

  // "foo();"
</script>
```

Node.js 环境下:

```js
const prettier = require("prettier");

prettier.format("foo()", { semi: true, parser: "babel" }); // "foo();"
```

## 可配置的选项

Prettier 附带了一些可定制的格式选项，可在 CLI 和 API 中使用。

* **printWidth** 每一行代码允许的字符数，默认 80,超过设定的字符数，会换行
* **tabWidth** 指定每行缩进的空格数
* **tabs** `true` 使用制表符(tab键)缩进行,  `false` 使用空格缩进行
* **semicolons** `true` 在语句末尾添加分号, `false` 语句末尾不添加分号
* **quotes** `true` 使用单引号, `false` 使用双引号
* **quoteProps** `as-needed` 只有在对象属性需要引号时，为其添加双引号， `consistent` 当对象的所有属性中存在一个属性必须添加引号，则将其所有属性添加引号，`preserve` 对象属性声明时加了引号，格式化后就有引号
* **jsxQuotes** `true` 在 JSX 文件里使用单引号，`false` 在 JSX 文件里使用双引号
* **trailingCommas** `es5` 遵循 es5 语法中定义的尾逗号， `none` 无尾逗号,  `all` 尽可能在结尾处加上尾逗号
* **bracketSpacing** `true` 对象字面量两边有空格，`false` 对象字面量两边没有空格
* **jsxBrackets** `true` JSX 文件里组件里 `>` 换行, `false` JSX 文件里组件里 `>` 不换行, 
* **arrowParens** `always` 始终保留括号，`avoid` 不保留括号
* **rangeStart** 表示从那一行开始格式化
* **rangeEnd** 表示从那一行结束格式化
* **filepath** 详见 [Parser](https://prettier.io/docs/en/options.html#parser)
* **requirePragma** 是否启用注解格式化，即配置注解的格式化，不配置的不格式化，默认值：false（类似 @prettier 
* **insertPragma** `true` 当格式化时，会在文件头添加 `/** @format */`,  `false` 不会添加 `/** @format */` 
* **proseWrap** 与 markdown 有关系 默认值 `preserve`
* **htmlWhitespaceSensitivity** 指定 HTML 文件的全局空白区域敏感度,默认值 css
* **endOfLine** 结尾类型，默认值为 auto。


## Prettier 与 ESLint

**Prettier** 用来格式化代码，保持代码中分号,单双引号等等格式统一。

**ESLint** 主要用来校验 JavaScript 代码语法错误，也能起到规范代码格式的作用。

在日常开发中，我们既要使用 Prettier, 也要使用 ESLint。用 ESLint 检查代码中可能存在的语法错误，
用 Prettier 调整代码格式。进而提高开发效率。

## 参考链接

- [Prettier](https://prettier.io/)
