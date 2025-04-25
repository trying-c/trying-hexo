---
title: 一次失败的域名托管实践：Cloudns + Cloudflare + GitHub Pages
categories: GitHub
tags:
  - GitHub
  - 失败案例
series:
  - GitHub
abbrlink: 24432
date: 2025-04-25 19:23:58
---

今天尝试将 Cloudns 的免费域名通过 Cloudflare 托管并 GitHub Pages 中使用，却遭遇了持续失败。不过这个踩坑过程让我进一步理解了 DNS、域名等相关的东西，还是做个记录。

---

## 🚧 操作环境

- 域名注册商：Cloudns（免费套餐）
- DNS 托管：Cloudflare
- 静态托管：GitHub Pages
- 目标：通过自定义域名访问 GitHub Pages

## 🔄 操作步骤

1. **Cloudns 免费域名获取**
   - 注册 Cloudns 账号
   - 创建带免费域名的 DNS 空间
2. **Cloudflare 配置**
   - 选择免费计划
   - 获取对应域名的 DNS 记录
   - 按 Cloudflare 指引完成 ns 服务器配置
3. **GitHub Pages 验证**
   - 在仓库设置 Custom domain
   - 等待 DNS 传播

**最终现象**：

- ❗ GitHub 最终提示域名不可用
- ❗ 访问域名始终报 502

---

## 🔍 失败原因分析

### 1. Cloudns 免费套餐的限制

查阅[官方文档](https://www.cloudns.net/wiki/article/42/)发现：

- 免费域名**无法修改默认 NS 服务器**
- 只能使用 Cloudns 提供的 4 组 NS 地址（如`ns1.cloudns.net`等）

### 2. Cloudflare 的工作机制

- 要求域名**必须使用其指定的 NS 服务器**（如`cora.ns.cloudflare.com`）
- 通过 NS 接管的方式管理 DNS 解析

### 3. 根本矛盾

{% mermaid %}
graph LR
A[Cloudns免费域名] -->|锁定NS| B(必须使用cloudns.net的NS)
C[Cloudflare] -->|要求| D(必须改用cloudflare的NS)
B --> E((NS冲突))
D --> E
{% endmermaid %}

---

## 💡 经验总结

### 技术层面

- NS 服务器是域名解析的"总开关"
- 免费域名服务常存在隐性限制
- GitHub Pages 验证依赖 NS 最终控制权

### 解决方案

1. **更换域名提供商**
   使用支持修改 NS 的免费服务（如 Freenom）

   > 但好像从 2023 年开始，就已经注册不了了。┭┮﹏┭┮

2. **付费升级 Cloudns 套餐**
   [Premium 套餐](https://www.cloudns.net/pricing/)支持自定义 NS

   > 穷鬼一个不考虑了 ┭┮﹏┭┮

3. **直接使用 GitHub 子域名**
   放弃自定义域名，使用`xxx.github.io`

   > 还是乖乖用官方域名
