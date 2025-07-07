---
title: 解决 Electron 安装失败问题 —— RequestError: unable to verify the first certificate：
tags:
  - Electron
  - 安装教程
  - Node
  - NPM 
date: 2025-07-07 17:39:17
---

> 记录一次 Electron 安装经历，以及最终通过 cnpm 成功解决的方案

## 问题背景

```bash
npm install electron
```

使用 npm 或 yarn 安装 Electron 时持续报错：

```bash
error RequestError: unable to verify the first certificate
    ...
```

尝试使用 yarn 安装，同样的问题：

```bash
yarn add electron
```

错误信息基本相同，核心提示都是 **"unable to verify the first certificate"**，这意味着 SSL 证书验证失败。

## 尝试解决方案

### 方法 1：切换国内镜像源

首先尝试切换到淘宝源，这是国内开发者常用的解决方案：

```bash
npm config set registry https://registry.npmmirror.com
```

再次运行 `npm install electron` - 问题依旧！

### 方法 2：跳过 SSL 证书验证

既然报错是证书问题，尝试关闭 SSL 验证：

```bash
npm config set strict-ssl false
```

再次运行 `npm install electron` - 问题依旧！

### 方法 3：清除 npm 缓存

怀疑缓存问题，尝试清除缓存后重试：

```bash
npm cache clean --force
rm -rf node_modules
rm package-lock.json
npm install
```

结果 - 没有任何改变。

### 方法 4：设置代理环境变量

尝试设置 Node.js 的代理环境变量：

```bash
set NODE_TLS_REJECT_UNAUTHORIZED=0
npm install electron
```

还是同样的错误，问题依然存在。

## 尝试 cnpm

搜索相关问题，看到用 cnpm 成功安装的案例，故尝试：

### 安装 cnpm

```bash
npm install -g cnpm --registry=https://registry.npmmirror.com
```

### 使用 cnpm 安装 Electron

```bash
cnpm install electron
```

最终成功安装了 Electron！

## 为什么 cnpm 能成功？

思考整个解决过程，虽然切换了 npm 源，但 npm 客户端本身在请求时可能仍有某些安全限制。而 cnpm 作为专门为国内环境优化的工具：

1. 内置了适合国内网络的默认配置
2. 对证书验证的处理可能更宽松
3. 使用了专门为国内优化的下载逻辑

### 推荐工作流程

对于国内开发者，建议采用以下工作流程安装 Electron：

```bash
# 1. 安装cnpm
npm install -g cnpm --registry=https://registry.npmmirror.com

# 2. 使用cnpm安装Electron
cnpm install electron

# 3. 验证安装
npx electron -v
```
