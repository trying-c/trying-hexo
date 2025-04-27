---
title: 为Hexo博客添加Waline评论系统的具体操作步骤
abbrlink: 38521
date: 2025-04-28 00:37:50
categories: Hexo
series: Hexo
tags:
  - Hexo
---

最近想把博客的评论系统从无到有搭建起来，最终选择了轻量且支持数据库的 **Waline**。记录下完整的配置流程，包含服务端部署与前端集成细节。

---

## 🚧 操作环境

- 静态博客框架：Hexo
- 主题：Butterfly
- 评论系统：Waline
- 数据库服务：LeanCloud（免费套餐）

---

## 🔧 配置步骤

### 一、准备后端服务

#### 1. 注册 LeanCloud

1. 访问 [LeanCloud 国际版](https://console.leancloud.app/)
2. 创建新应用（如 `BlogComment`）
3. 进入 **设置 → 应用凭证**，记录 `AppID` 、 `AppKey`和`MasterKey`

#### 2. 部署 Waline 服务端

推荐通过 Vercel 一键部署：

1. 点击 [Waline Vercel 模板](https://github.com/walinejs/waline/tree/main/example)
2. 点击 **Deploy**，按提示完成部署
3. 在 Vercel 控制台找到 **环境变量**，添加以下配置：
   ```env
   LEAN_ID=你的AppID
   LEAN_KEY=你的AppKey
   LEAN_MASTER_KEY=你的MasterKey
   ```

> 部署完成后获得服务端地址（如 `https://xxx.vercel.app`）

---

### 二、配置 Hexo 主题

#### 1. 安装 Waline 插件

> 若使用的主题已内置 Waline, 这步可跳过

在博客根目录执行：

```bash
npm install @waline/hexo-next
```

#### 2. 修改主题配置文件

打开 `_config.<主题名>.yml`（如`_config.butterfly.yml`）：

```yaml
# Waline 评论配置
waline:
  serverURL: https://xxx.vercel.app # 替换为你的Vercel地址
  appId: 你的LeanCloud AppID
  appKey: 你的LeanCloud AppKey
  pageview: true # 开启阅读量统计
```

---

## 🚨 常见问题

### 1. 评论框不显示

- 检查 `page.comments` 变量：是否在文章 Front-matter 中设为 `true`
- 打开浏览器控制台查看是否有 JS 报错

### 2. 数据库写入失败

- 确保 LeanCloud 应用的 **数据存储** 服务已开启
- 检查 Vercel 环境变量是否包含 `LEAN_ID` 和 `LEAN_KEY`

---

## 🌟 效果验证

1. 本地启动服务：

```bash
hexo clean && hexo s
```

2. 访问任意文章页面，底部应出现评论框
3. 提交测试评论后，登录 [LeanCloud 控制台](https://console.leancloud.app/) 查看 `Comment` 表是否有新数据

---

## 💡 总结

1. **服务分离架构**：Waline 的前后端分离设计保障了静态博客的安全性
2. **免费资源利用**：通过 LeanCloud + Vercel 实现零成本部署
3. **扩展性强**：后续可增加表情包、审核功能等

{% note primary no-icon %}
小贴士：完整配置代码已同步至[博客仓库](https://github.com/trying-c/trying-c.github.io)，欢迎参考交流~
{% endnote %}
