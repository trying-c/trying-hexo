---
title: 用GitHub Pages搭网站，GitHub Actions自动化真香！
date: 2025-04-27 02:33:49
categories: GitHub
series: GitHub
tags: GitHub
abbrlink: 6603
---

前面写了一篇关于 GitHub Actions 自动部署 Hexo 博客的文，今天来具体介绍一下 Github Pages 配合 GitHub Actions 部署展示静态页面的使用方法。

## GitHub Pages：免费的静态页面托管

### GitHub Pages 是什么？

简单说就是 GitHub 自带的静态网站托管服务，支持 HTML/CSS/JS，自带 CDN 和 HTTPS。我常用它来托管：

- 个人博客（配合 Hexo/Jekyll）
- 项目演示页
- 技术文档
- 简历展示页

### 极简四步上手

1. **新建特殊仓库**  
   仓库名必须为`<用户名>.github.io`，比如我的就是`trying-c.github.io`，那么对应的访问地址是`https://trying-c.github.io`

   > 自定义仓库名也是可以的，等于是普通仓库，对应的访问地址就变成`https://<用户名>.github.io/<仓库名>`

2. **提交一个 index.html 试试水**

   ```html
   <!DOCTYPE html>
   <html>
     <body>
       <h1>Hello GitHub!</h1>
       <p>这是我的第一个GitHub Pages页面</p>
     </body>
   </html>
   ```

3. **推送到 main 分支**
   等个 1-2 分钟，访问`https://<用户名>.github.io`就能看到页面

4. **绑定自定义域名（可选）**
   在仓库 Settings -> Pages 里填域名，再到域名服务商添加 CNAME 记录即可

### 注意事项

- 遇到 404 先检查：仓库是否公开/分支是否正确/文件名是否小写
- 想用 Vue/React 等框架时，记得先 build 生成静态文件
- 每月有流量限制（100GB），但个人博客完全够用

## GitHub Actions：让部署自动化飞起来

以前每次更新内容都要手动：

{% mermaid %}
graph LR
A[本地生成静态文件] --> B[推送到 GitHub 仓库] --> C[刷新页面等待部署]
{% endmermaid %}

现在用 Actions 可以做到：

{% mermaid %}
graph LR
trigger[代码 Push 事件] --> D[自动安装依赖]
D --> E[构建静态文件]
E --> F[部署到 GitHub Pages]
{% endmermaid %}

**下面来介绍一下如何自动化部署基于 Vue3 + Vite 框架构建的项目吧**

### 创建基础工作流文件

首先在项目根目录创建`.github/workflows/deploy.yml`文件，写入基础结构：

```yaml
name: Deploy Blog # 工作流名称
on: [push] # 触发条件：默认监听所有分支的push事件
```

修改触发条件，只监听 main 分支的 push：

```yaml
name: Deploy Blog
on:
  push:
    branches: [main] # 只监控main分支的推送
```

添加 jobs 配置块，指定运行环境和任务名称：

```yaml
name: Deploy Blog
on:
  push:
    branches: [main]

jobs:
  build-and-deploy: # 任务ID（可自定义）
    runs-on: ubuntu-latest # 使用GitHub托管的Ubuntu虚拟机
    steps: [] # 步骤占位符
```

### 代码检出（步骤 1）

添加代码拉取步骤：

```yaml
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          fetch-depth: 0 # 获取完整提交历史
```

### 配置 Node 环境（步骤 2）

设置 Node.js 运行环境：

```yaml
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
      ...

    - name: Setup Node
      uses: actions/setup-node@v3
      with:
        node-version: 18
        cache: 'npm'  # 启用npm缓存加速
```

### 安装依赖（步骤 3）

安装项目依赖（Vue 项目不需要全局安装 CLI）：

```yaml
    - name: Setup Node
      uses: actions/setup-node@v3
      ...

    - name: Install dependencies
      run: npm ci  # 推荐使用ci命令保证依赖一致性
```

### 构建生产包（步骤 4）

执行 Vite 构建命令：

```yaml
    - name: Install dependencies
      run: npm ci
      ...

    - name: Build project
      run: npm run build
```

### 自动部署（步骤 5）

部署到 GitHub Pages：

```yaml
    - name: Build project
      run: npm run build
      ...

    - name: Deploy to GitHub Pages
      uses: peaceiris/actions-gh-pages@v3
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./dist  # Vite默认输出目录
```

### 完整配置文件

最终完整的工作流配置：

```yaml
name: Deploy Vue Site
on:
  push:
    branches: [main]
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Build project
        run: npm run build

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
          keep_files: false # 每次部署清空旧文件
```

> **TIP**: 类似`${{ secrets.GITHUB_TOKEN }}`的变量配置可以预先在仓库的 **Settings** -> **Secrets** 中填入。

### GitHub Pages 分支设置指南

> 关键操作步骤（需手动配置一次）

1. **进入仓库 Settings**  
   点击仓库顶部导航栏的 `Settings` -> 左侧菜单选择 `Pages`

2. **配置部署分支**
   ```markdown
   Build and deployment -> Source
   ↓
   Branch: [gh-pages] ← 选择部署分支
   ↓
   Folder: / (root) ← 选择部署目录（即 gh-pages 分支根目录）
   ↓
   Save
   ```

### 工作流示意图

{% mermaid %}
graph TD
A[代码推送到 main 分支] --> B[拉取最新代码]
B --> C[配置 Node.js 环境]
C --> D[安装项目依赖]
D --> E[构建生产包]
E --> F[部署到 gh-pages 分支]
F --> G[触发 GitHub Pages 更新]
{% endmermaid %}

**注意事项**：

1. 部署后 404 错误：检查 vite.config.js 中的 base 路径配置
2. 环境变量失效：需在 GitHub 仓库 **Settings** -> **Secrets** 中配置对应变量
3. 文件更新延迟：GitHub Pages 有 1-5 分钟的缓存时间
4. 权限问题：确保 GITHUB_TOKEN 具有仓库写入权限

## 结语

从手动 上传到全自动部署，GitHub 这套组合拳帮我省下了无数喝咖啡的时间。如果你还在手动部署，不妨今天就用 GitHub Pages + GitHub Actions 改造你的工作流。有什么实战心得欢迎评论区交流~
