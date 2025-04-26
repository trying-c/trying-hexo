---
title: 使用 GitHub Actions 自动部署 Hexo
categories: GitHub
series: GitHub
tags:
  - 效率工具
  - GitHub
  - Hexo
abbrlink: 24967
date: 2025-04-24 23:12:37
---

- Hexo 是一款高效便捷的静态博客框架，但每次更新文章后手动生成静态文件并部署到 GitHub Pages 可能会有些繁琐。
- 借助 **GitHub Actions** 工具，我们可以实现自动化部署，解放双手。
- 部署到 Github Pages 的常用方式是 **Hexo** 官方推荐的 `hexo-deployer-git` 一键部署
- 本文则是通过编写自定义的 **GitHub Actions** 工作流配置文件（如 `deploy.yml`）去完成静态网页的生成并将其推送到分支后触发 **Github Pages** 的部署。

---

## 官方推荐的一键部署

Hexo 官方提供了 `hexo-deployer-git` 插件，只需在 Hexo 的配置文件 `_config.yml` 中添加以下配置：

```yaml
deploy:
  type: git
  repo: https://github.com/用户名/仓库名.git
  branch: gh-pages
```

然后在命令行中执行 `hexo clean && hexo deploy`，插件会自动将生成的静态文件推送到 `gh-pages` 分支。这种方式简单快捷，适合需要基础功能的用户。

---

## 自定义 GitHub Actions 工作流

另外也可以通过编写 `deploy.yml` 文件定义自定义 GitHub Actions 工作流。以下是具体实现步骤：

### 1. 创建 GitHub Actions 配置文件

在项目根目录下新建 `.github/workflows/deploy.yml` 文件，内容如下：

```yaml
name: Deploy Hexo to GitHub Pages

on:
  push:
    branches:
      - main # 当推送到 main 分支时触发

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      # 步骤1：检出仓库代码
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          submodules: false # 禁用子模块检查

      # 步骤2：设置 Node.js 环境
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: v20.18.0 # 指定 Node.js 版本 (最好和本地的Node版本一致，减少报错的机率)

      # 步骤3：安装 Hexo 及依赖
      - name: Install Dependencies
        run: npm install

      # 步骤4：安装 Hexo 部署插件
      - name: Install Hexo Git Deployer
        run: |
          npm install hexo-deployer-git --save 
          npm install hexo-cli -g

      # 步骤5：生成静态文件
      - name: Clean and Generate Static Files
        run: |
          hexo clean
          hexo generate

      # 步骤6：配置 Git 用户信息 (可以按自己喜好自定义)
      - name: Configure Git
        run: |
          git config --global user.name 'github-actions[bot]'
          git config --global user.email 'github-actions[bot]@users.noreply.github.com'

      # 步骤7：部署到 gh-pages 分支
      - name: Deploy to GitHub Pages
        env:
          GH_TOKEN: ${{ secrets.GH_TOKEN }} # 使用 Token 认证
        run: |
          cd public/
          git init
          git add -A
          git commit -m "Create by workflows"
          git remote add origin https://${{ secrets.GH_TOKEN }}@github.com/trying-c/trying-hexo.git
          git push origin HEAD:gh-pages -f
```

### 2. 配置密钥（GH_TOKEN）

1. 在 GitHub 仓库的 **Settings → Secrets → Actions** 中，新建一个名为 `GH_TOKEN` 的密钥，值为具有仓库读写权限的 [Personal Access Token](https://github.com/settings/tokens)。
2. 确保仓库的 GitHub Pages 功能已开启，并指定源分支为 `gh-pages`。
   - 进入仓库的 **Settings** → **Pages** 。
   - 在 **Branch** 选项中选择 `gh-pages` 分支，并指定根目录（默认为 `/ (root)`）。
   - 点击 **Save**，稍等片刻即可通过 https://用户名.github.io/仓库名 访问你的博客。

### 3. 触发自动化流程

将代码推送到 `main` 分支后，GitHub Actions 会自动执行以下操作：

1. `deploy.yml`: 安装 Node.js 环境 → 构建 Hexo 项目 → 生成静态文件 → 推送至 `gh-pages` 分支
2. 推送至 `gh-pages`分支后自动生成的工作流: 由 Github Pages 完成网页部署。

---

## 总结

1. 追求快速上手，官方的一键部署完全够用！！（{% post_link '两个简化Hexo操作的小脚本' '也可以编写便捷的脚本，双击文件解决繁琐的发布操作' %}）
2. 追求解放双手，自定义 GitHub Actions 工作流会是更好的选择喔~（还能实现 Github 上使用同一仓库完成源码托管和静态部署）

两种方式最终都通过 GitHub Pages 实现博客托管，读者可以根据实际需求灵活选用。
