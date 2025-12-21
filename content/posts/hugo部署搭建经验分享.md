+++
date = '2025-12-20T16:21:35+08:00'
draft = false
title = '使用Hugo搭建个人博客并部署至GitHub Pages实现自动化发布'
+++

过去我并没有养成系统记录和输出的习惯，总以为靠脑子就能记住。

然而，随着年龄增长，我发现自己记忆的磨损速度快得惊人，常常转身就忘事。

于是，我决定搭建一个博客，用于日常的记录和整理。

这篇文档就用来记录整个博客的创建过程。

## 技术选型：Hugo + GitHub Pages

我将自己的需求告诉了Gemini，请它帮忙推荐合适的技术方案和工具。

核心需求如下：

1.  **多终端发布**：支持在电脑、手机等多种设备上发布内容。毕竟有时懒得开电脑，而灵感可能随时出现在手机上。
2.  **简洁易用**：安装和部署过程简单，博客不求华丽，但求简洁、流畅。
3.  **公网可访问**：必须能够通过公网访问，不能仅限于本地浏览。

综合考虑后，我选择了 **Hugo + GitHub Pages** 的方案，操作系统为 Windows。

## 部署步骤

### 1. 安装 Hugo

通过 Windows 包管理器 winget 安装 Hugo Extended 版本（支持 Sass/SCSS）。

```bash
winget install Hugo.Hugo.Extended
```

### 2. 安装 Git

Git 的安装过程此处省略。

### 3. 初始化 Hugo 项目

```bash
hugo new site quickstart # 文件夹名称可按喜好更改
cd quickstart
git init
# 添加 Ananke 主题作为 Git 子模块
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
echo "theme = 'ananke'" >> hugo.toml
hugo server
```

### 4. 创建文章

使用命令创建新文章：

```bash
hugo new content posts/my-first-post.md
```

文章将在 `content/posts/` 目录下生成。你也可以直接在该目录下手动创建 `.md` 文件。

**注意：内容完成后，务必将文章头部的 `draft` 字段改为 `false`，因为 Hugo 默认不会构建草稿。**

### 5. 配置网站信息

在项目根目录的 `hugo.toml` 文件中，配置网站的基本信息。

```toml
baseURL = 'https://yourname.github.io/' # 替换为你的 GitHub 用户名，必须与账户名一致
languageCode = 'en-us'
title = 'My New Hugo Site'              # 替换为你想要的网站名称
theme = 'ananke'
```

配置完成后，可以使用 `hugo server -D` 命令在本地预览网站（`-D` 参数包含草稿）。

### 6. 在 GitHub 创建仓库

在 GitHub 上创建一个名为 `yourname.github.io` 的公共仓库（`yourname` 替换为你的 GitHub 用户名）。

然后，进入仓库的 **Settings > Pages**，将 **Source** 改为 **GitHub Actions**。

### 7. 配置 GitHub Pages 与自动化工作流

在 Hugo 项目的根目录下，创建 `.github/workflows/hugo.yml` 文件，并将官网提供的配置内容复制进去。

该工作流会在代码推送到 `main` 分支时，自动构建 Hugo 站点并部署到 GitHub Pages。

你可以在 Hugo 官方文档中找到最新的工作流配置示例：<https://gohugo.io/host-and-deploy/host-on-github-pages/>

### 8. 完成部署，开始写作

完成以上配置后，你的自动化发布流水线就已就绪。今后只需在 `content/posts/` 目录下添加或更新文章，然后将更改 `push` 到 GitHub 仓库，GitHub Actions 便会自动构建并发布网站。

访问 `https://yourname.github.io` 即可查看你的博客。

### 9. 后续优化 (Todo)

*   支持在文章中插入并管理图片。

### 10. 修改主题

``git submodule add https://github.com/pseudoyu/pure themes/pure``

git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
git submodule update --init --recursive # needed when you reclone your repo (submodules may not get cloned automatically)



---

**参考链接：**
*   Hugo 快速入门指南：<https://gohugo.io/getting-started/quick-start/>
