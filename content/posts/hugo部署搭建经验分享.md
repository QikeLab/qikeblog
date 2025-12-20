+++
date = '2025-12-20T16:21:35+08:00'
draft = false
title = 'Hugo搭建个人博客并部署Github Pages实现自动化公网发布'
+++

之前一直不太重视记录和输出，光凭脑子记就可以

然而，现在年纪大了，发现自己的脑子正在以惊人的速度磨损

经常有些事情转头就忘

遂打算搭建一个博客做些记录

那么就用这个文档记录创建的过程吧！

## 选择Hugo+Github Pages

把我的需求告诉Gemini，让他帮我选择使用的技术方案和工具。

需求是：

1. 可以用电脑、手机等多终端发布，毕竟有时候懒得打开电脑、有时候想法只是记在手机里
2. 安装、部署简单，不要求博客花里胡哨，简介、流畅即可
3. 要能够通过公网访问，不能只能在本地浏览

最终我选择了Hugo + Github Pages的方案，windows系统

## 部署过程

### 1. 安装hugo

winget install Hugo.Hugo.Extended

### 2. git的安装省略

### 3. 初始化hugo项目

``` bash
hugo new site quickstart //文件夹名随喜
cd quickstart
git init
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke //这里是拷贝ananke主题作为git子模块
echo "theme = 'ananke'" >> hugo.toml
hugo server
```

### 4. 添加文章

``hugo new content content/posts/my-first-post.md``

创建的文章会在content/posts目录下生成，也可以自己手动创建

Note: **内容完成后，记得将draft字段修改为false。因为hugo默认不会构建草稿**

### 5. 配置网址

在根目录的hugo.toml文件中，配置网址信息

``` toml
baseURL = 'https://example.org/'   //替换为'https://yourname.github.io/'，yourname必须和你的github账户名一致
languageCode = 'en-us'
title = 'My New Hugo Site'         //替换为你想要的网站名
theme = 'ananke'
```

配置完成后，可以使用``hugo server -D``预览

### 6. 在Github创建yourname.github.io仓库

在github仓库的Settings > Pages中，将Source改为Github Actions

### 7. 配置github pages和Actions实现自动化构建网页

    在项目根目录，创建.github/workflows/hugo.yml文件

    并替换为官网的配置：

    ``` yml
    name: Build and deploy
    on:
    push:
        branches:
        - main
    workflow_dispatch:
    permissions:
    contents: read
    pages: write
    id-token: write
    concurrency:
    group: pages
    cancel-in-progress: false
    defaults:
    run:
        shell: bash
    jobs:
    build:
        runs-on: ubuntu-latest
        env:
        DART_SASS_VERSION: 1.96.0
        GO_VERSION: 1.25.5
        HUGO_VERSION: 0.152.2
        NODE_VERSION: 24.12.0
        TZ: Europe/Oslo
        steps:
        - name: Checkout
            uses: actions/checkout@v5
            with:
            submodules: recursive
            fetch-depth: 0
        - name: Setup Go
            uses: actions/setup-go@v5
            with:
            go-version: ${{ env.GO_VERSION }}
            cache: false
        - name: Setup Node.js
            uses: actions/setup-node@v4
            with:
            node-version: ${{ env.NODE_VERSION }}
        - name: Setup Pages
            id: pages
            uses: actions/configure-pages@v5
        - name: Create directory for user-specific executable files
            run: |
            mkdir -p "${HOME}/.local"
        - name: Install Dart Sass
            run: |
            curl -sLJO "https://github.com/sass/dart-sass/releases/download/${DART_SASS_VERSION}/dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
            tar -C "${HOME}/.local" -xf "dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
            rm "dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
            echo "${HOME}/.local/dart-sass" >> "${GITHUB_PATH}"
        - name: Install Hugo
            run: |
            curl -sLJO "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz"
            mkdir "${HOME}/.local/hugo"
            tar -C "${HOME}/.local/hugo" -xf "hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz"
            rm "hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz"
            echo "${HOME}/.local/hugo" >> "${GITHUB_PATH}"
        - name: Verify installations
            run: |
            echo "Dart Sass: $(sass --version)"
            echo "Go: $(go version)"
            echo "Hugo: $(hugo version)"
            echo "Node.js: $(node --version)"
        - name: Install Node.js dependencies
            run: |
            [[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true
        - name: Configure Git
            run: |
            git config core.quotepath false
        - name: Cache restore
            id: cache-restore
            uses: actions/cache/restore@v4
            with:
            path: ${{ runner.temp }}/hugo_cache
            key: hugo-${{ github.run_id }}
            restore-keys:
                hugo-
        - name: Build the site
            run: |
            hugo \
                --gc \
                --minify \
                --baseURL "${{ steps.pages.outputs.base_url }}/" \
                --cacheDir "${{ runner.temp }}/hugo_cache"
        - name: Cache save
            id: cache-save
            uses: actions/cache/save@v4
            with:
            path: ${{ runner.temp }}/hugo_cache
            key: ${{ steps.cache-restore.outputs.cache-primary-key }}
        - name: Upload artifact
            uses: actions/upload-pages-artifact@v3
            with:
            path: ./public
    deploy:
        environment:
        name: github-pages
        url: ${{ steps.deployment.outputs.page_url }}
        runs-on: ubuntu-latest
        needs: build
        steps:
        - name: Deploy to GitHub Pages
            id: deployment
            uses: actions/deploy-pages@v4
    ```

参考官网的配置：<https://gohugo.io/host-and-deploy/host-on-github-pages/>

### 8. 配置完成，持续输出文章并通过yourname.github.io访问

目前按照这个配置，只要在posts中增加内容，然后push到git仓库中，Github会自动将内容发布到<yourname.github.io>

### 9. 支持图片todo



refs: <https://gohugo.io/getting-started/quick-start/>
