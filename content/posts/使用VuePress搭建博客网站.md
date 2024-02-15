+++
title = '使用 VuePress 搭建博客网站'
date = 2023-06-20
+++

作为一名技术人员，搭建自己的技术博客，有助于总结和沉淀知识，建设技术影响力，有利于职业发展

市面上有非常多的博客搭建工具，比如大名鼎鼎的 WordPress，也有新晋的 Hexo 等等，但是本篇博客向大家传授一个更简单的搭建博客网站的方法，使用 VuePress 搭建博客网站

## VuePress

VuePress 是 vue 为驱动的一个静态网站生成工具，可以用它来编写技术文档和搭建个人博客。它有下列特性：

- 内置 markdown 扩展，针对技术文档进行了优化
- 可以直接在 markdown 文件中使用 Vue 代码
- 包含一个默认主题，也可通过 vue 代码编写自定义主题
- 支持 PWA
- Google Analytics 集成

使用 VuePress 搭建的博客上手简单、博客界面简洁优雅、依托免费的 GitHub Pages 部署工具，所以使用 VuePress 搭建博客非常香

### 安装 VuePress

> VuePress 需要 Node.js>= 8.6

在本地创建一个博客项目

```shell
mkdir blog && cd blog
```

然后使用你喜欢的包管理器进行初始化，笔者在这里使用 `npm`

```shell
npm init # or yarn init
```

将 VuePress 安装为本地依赖

```shell
npm install -D vuepress #yarn add -D vuepress
```

在 package.json 中添加一些 scripts。为什么是 `docs:dev`,请见下文

```json
{
  "scripts": {
    "docs:dev": "vuepress dev docs",
    "docs:build": "vuepress build docs"
  }
}
```

### 添加主题

VuePress 官方提供 @vuepress/theme-blog 主题，将官方的博客主题添加到博客项目中

```shell
npm install @vuepress/theme-blog -D
# or yarn add @vuepress/theme-blog -D
```

推荐的文件目录如下

```text
├── blog
│ ├── _posts
│ │ ├── 第一篇博客.md #example
│ │ ├── ...
│ │ └── 第n篇博客.md #example
│ └── .vuepress
│ ├── `components` _(**Optional**)_
│ ├── `public` _(**Optional**)_
│ ├── `styles` _(**Optional**)_
│ │ ├── index.styl
│ │ └── palette.styl
│ ├── config.js
│ └── `enhanceApp.js` _(**Optional**)_
└── package.json
```

要求：

- `blog/.vuepress/config.js`: 配置入口文件，也可以是 `yml` 或 `toml`
- `blog/_posts`：存储您的帖子内容

可选：

如果你想配置下面的文件，你需要一些 [VuePress](https://vuepress.vuejs.org/zh/) 的知识

- `blog/.vuepress/components`：该目录下的 Vue 组件会自动注册为全局组件。
- `blog/.vuepress/styles`：存放样式相关的文件。
- `blog/.vuepress/styles/index.styl`: 自动应用的全局样式文件，生成在 CSS 文件末尾，优先级高于默认样式。
- `blog/.vuepress/styles/palette.styl`：调色板用于覆盖默认颜色常量和设置 Stylus 的颜色常量。
- `blog/.vuepress/public`: 静态资源目录。
- `blog/.vuepress/enhanceApp.js`：应用程序级别增强。

### 博客配置

下面是博客的配置说明

```js
module.exports = {
  // 博客标题
  title: "my blog",
  // 主题设置
  theme: "@vuepress/theme-blog",
  //主题配置
  themeConfig: {
    // 导航栏
    nav: [],
    footer: {
      //联系信息
      contact: "",
    },
  },
};
```

更详细的配置请见[@vuepress/theme-blog 主题配置](https://vuepress-theme-blog.billyyyyy3320.com/config/#sitemap)

### 添加文章

在 `blog` 目录下新建一个 `docs` 文件夹，在 `docs` 文件夹里创建 `_post` 文件夹，并创建`第一篇博客.md`

```md
---
title: 第一篇博客
date: 2023-01-04
author: qinghuanI
location: wuhan
---

第一篇博客内容
```

然后在本地启动服务器

```shell
npm run docs:dev # or yarn docs:dev
```

VuePress 会在 [http://localhost:8080](http://localhost:8080/) 启动一个热重载的开发服务器。页面效果如下所示

![vuepress_blog.png](/2020/vuepress_blog.png)

### 部署

项目搭建完成后，接下来就是部署博客。笔者将代码托管在 GitHub,下面使用 Github Pages 部署博客项目

在自己的 github 创建一个 githubName.github.io 名称的仓库。比如我的 github 名称是 qinghuanI,那么我创建的仓库名是 `qinghuanI.github.io`，然后提交所有代码

在项目根目录下的 `.github/workflows` 目录（没有的话，请手动创建一个）下创建一个 .yml 或者 .yaml 文件，如:blog-deploy.yml;

```textmate
name: Build and Deploy
on: [push]
jobs:
build-and-deploy:
runs-on: ubuntu-latest
steps:
- name: Checkout
uses: actions/checkout@master

- name: blog-deploy
uses: jenkey2011/vuepress-deploy@master
env:
ACCESS_TOKEN: ${{ secrets.BLOG }}
TARGET_REPO: qinghuanI/qinghuanI.github.io <--- 你的仓库名
TARGET_BRANCH: gh_pages
BUILD_SCRIPT: npm run docs:build
BUILD_DIR: blog/.vuepress/dist
CNAME: https://www.qinghuani.fun
```

配置仓库的 `ACCESS_TOKEN` 后，当代码构建成功后，就可以从 https://qinghuanI.github.io 访问了

### 自定义域名

如果您需要自定义网站的域名，可以去[万网](https://wanwang.aliyun.com/domain/)购买域名，然后进行如下的配置

```textmate
env:
ACCESS_TOKEN: ${{ secrets.BLOG }}
TARGET_REPO: qinghuanI/qinghuanI.github.io <--- your repo name
TARGET_BRANCH: gh_pages
BUILD_SCRIPT: npm run docs:build
BUILD_DIR: blog/.vuepress/dist
CNAME: https://www.qinghuani.fun <--- 你购买的域名
```

然后在项目的根目录下添加 CNAME 文件

```textmate
qinghuani.fun <--- 你购买的域名
```

GitHub Pages 部署成功后，在浏览器输入你购买的域名，就可以正常访问了，一个简单的博客网站搭建完成

## 参考链接

- [@vuepress/theme-blog](https://vuepress-theme-blog.billyyyyy3320.com/#intro)
