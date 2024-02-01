+++
title = '使用 VuePress 制作在线简历'
date = 2023-02-04
+++

## 前言

使用在线简历可以让简历更好看，也方便他人浏览，给 HR 和面试官耳目一新的感受

VuePress 是一款基于 `Vue` 实现的静态网站生成器。有界面简洁优雅，上手简单和扩展 `Markdown` 语法等优点。
本篇博客带大家学习如何使用 VuePress 制作自己的在线简历

## 初始化

首先需要在本地新建在线简历项目

1. 创建并进入一个新目录

```shell
mkdir online-resume && cd online-resume
```

2. 使用您喜欢的包管理器进行初始化

```shell
yarn init
# npm init or pnpm init
```

根据提示，需要填写项目名称、版本号、项目描述和作者，其他的内容使用默认值

3. 将 VuePress 安装为本地依赖

```shell
yarn add vuepress -D
```

4. 添加一些必要的文件和配置

- 在根目录创建 `docs` 文件夹，在 `docs` 中添加 `.vuepress/config.js`，内容如下

```js
module.exports = {
  base: '/online-resume/',   //<--- 使用 GitHub Pages 时会用到
  themeConfig: {
    // 禁用搜索
    search: false,
    // 不需要导航栏
    navbar: false
  }
};
```

- 在 `docs` 中添加 `.vuepress/README.md` 文件

```markdown
---
home: true
heroText: xxx
tagline: xxx开发
---

请撰写您的工作履历
::: slot footer
MIT Licensed | Copyright © 2023-present [qinghuanI](https://github.com/qinghuanI)
:::
```

5. 在 `package.json` 中添加一些 `scripts`

```json
{
  "scripts": {
    "docs:dev": "vuepress dev docs",
    "docs:build": "vuepress build docs"
  }
}
```

6. 在本地启动服务器

```shell
yarn run docs:dev
```

VuePress 会在 [http://localhost:8080](http://localhost:8080) 启动一个热重载的开发服务器。
一个简单的简历项目骨架搭建完成，如图所示

![Resume Skeleton](https://cdn.jsdelivr.net/gh/qinghuanI/blog-images@main/uPic/resume_skeleton.png)

## 内容编排

在上一个章节，我们搭建了简单的项目骨架，现在需要往 `README.md` 中添加您的工作履历。
如何组织内容可以根据自己喜好来，笔者在这里使用我的格式演示

```markdown
---
home: true
heroText: XXX
tagline: Web 前端开发
---

## 专业技能

- 熟悉 XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
- 熟悉使用 XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

## 工作经历

<div :style="{display: 'flex', justifyContent: 'space-between'}">
  <div>XXXX公司1</div>
  <div>XX工程师</div>
  <div>XXXX项目组</div>
  <div>XXXX/XX-XXXX/XX</div>
</div>

XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

<div :style="{display: 'flex', justifyContent: 'space-between'}">
  <div>XXXX公司2</div>
  <div>XX工程师</div>
  <div>XXXX项目组</div>
  <div>XXXX/XX-XXXX/XX</div>
</div>

XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

## 项目经历

**XXXXXXXXXXXXXXXXXXX 项目**

项目描述：XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

项目职责：

- XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
- XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
- XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
- XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

项目业绩：XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

**XXXXXXXXXXXXXXXXXXX 项目**

项目描述：XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

项目职责：

- XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
- XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
- XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
- XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

项目业绩：XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

## 教育经历

<div :style="{display: 'flex', justifyContent: 'space-between'}">
  <div>XXXX/XX-XXXX/XX</div>
  <div>XXXXX大学</div>
  <div>本科</div>
  <div>XX学历</div>
</div>

## 个人成就

- XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
- XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
- XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

## 自我评价

- XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
- XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
- XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

::: slot footer
MIT Licensed | Copyright © 2023-present [qinghuanI](https://github.com/qinghuanI)
:::
```

> VuePress 支持在 Markdown 文件中使用组件，扩展 Markdown 的功能

简历骨架的显示效果如下

![Online Resume Template](https://cdn.jsdelivr.net/gh/qinghuanI/blog-images@main/uPic/online_resume_template.png)

此时项目的目录结构如下：

```shell
.
├── docs
│   ├── README.md     <--- 简历内容
│   └── .vuepress
│       └── config.js <--- VuePress 配置
├── package.json
└── yarn.lock
```

## 美化

上面的步骤已经实现在线简历，但是我们还可以使用插件优化，提升 HR 或面试官的浏览体验

当简历内容比较多，一屏显示不下所有内容时，浏览人员从简历底部回到顶部需要多次滑动鼠标滚轮，体验不佳，可以添加
[回到顶部](https://vuepress.vuejs.org/zh/plugin/official/plugin-back-to-top.html) 插件快速回到顶部

1. 安装 @vuepress/plugin-back-to-top

```shell
yarn add @vuepress/plugin-back-to-top -D
```

2. 在 `config.js` 中添加 `@vuepress/plugin-back-to-top`

```js
module.exports = {
  base: '/online-resume/',   //<--- 使用 GitHub Pages 时会用到
  themeConfig: {
    // 禁用搜索
    search: false,
    // 不需要导航栏
    navbar: false,
  },
  plugins: ["@vuepress/back-to-top"],
};
```

插件成功运行后，页面右下角会出现向上的箭头，点击箭头，即可快速回到页面顶部

## 部署

VuePress 项目有很多种[部署](https://vuepress.vuejs.org/zh/guide/deploy.html#%E4%BA%91%E5%BC%80%E5%8F%91-cloudbase)方式。笔者在这里使用 `GitHub Pages` 和 `GitHub Actions` 部署和集成该项目

1. 创建 GitHub access token

在您的 GitHub 页面，按照 `Settings -> Developer settings -> Personal access tokens -> Tokens(classic)` 进入创建新 `tokens` 页面。如图所示

![Generate new Tokens](https://cdn.jsdelivr.net/gh/qinghuanI/blog-images@main/uPic/generate_new_tokens.png)

点击 `generate new tokens` 按钮，选择 `Generate new tokens(classic)` 选项，
进入 `New personal access token(classic)` 页面

![New personal access token](https://cdn.jsdelivr.net/gh/qinghuanI/blog-images@main/uPic/new_personal_access_token.png)

输入 `Note`，选择永不过期，勾选上所有的 `scopes`，最后点击 `Generate token` 按钮创建 `token`，请存好该 token

2. 创建 secret

在您的 GitHub 里，创建 `online-resume` 仓库，如图所示

![Create Online Resume Repo](https://cdn.jsdelivr.net/gh/qinghuanI/blog-images@main/uPic/create_online_resume_repo.png)

在 `online-resume` 页面，按照 `Settings -> secrets and variables -> Actions` 创建仓库的 secret

![Add secret](https://cdn.jsdelivr.net/gh/qinghuanI/blog-images@main/uPic/add_secret.png)

输入 Name 和第一步创建的 personal access token，点击 `Add secret` 按钮创建 secret，创建后的 secret 如图所示

![Secrets List](https://cdn.jsdelivr.net/gh/qinghuanI/blog-images@main/uPic/secrets_list.png)

3. 在项目根目录下的 `.github/workflows` 目录（没有的话，请手动创建一个）下创建一个 `.yml` 或者 `.yaml` 文件，如:`online-resume-deploy.yml`;

```textmate
name: Build and Deploy
on: [push]
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@master

      - name: online-resume-deploy
        uses: jenkey2011/vuepress-deploy@master
        env:
          ACCESS_TOKEN: ${{ secrets.ONLINE_RESUME }}
          TARGET_REPO: qinghuanI/online-resume
          TARGET_BRANCH: gh_pages
          BUILD_SCRIPT: yarn && yarn docs:build
          BUILD_DIR: docs/.vuepress/dist
```

4. 本地代码同步到远程 `online-resume` 仓库

将项目代码推送至 GitHub 的 `online-resume` 仓库，`GitHub Actions` 会帮助持续集成代码，每次有新提交，
GitHub 都会重新构建

5. 配置 GitHub Pages

按照 `Settings -> Pages` 指示，找到 `Branch` 配置，选择分支，然后点击 `save` 按钮保存

![GitHub Pages Configuration](https://cdn.jsdelivr.net/gh/qinghuanI/blog-images@main/uPic/github_pages_branch.png)

等构建成功后，在地址栏输入 [https://qinghuanI.github.io/online-resume/](https://qinghuanI.github.io/online-resume/) 就可以正常访问

完整的代码请见 [online-resume 仓库](https://github.com/qinghuanI/online-resume)
