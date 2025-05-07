---
title: 从零搭建Vue3组件库：开发、文档、发布全流程指南
tags:
  - Vue3
  - Vite
  - 组件库
  - 前端工程化
abbrlink: 32970
date: 2025-05-08 01:31:59
---

本文将详细介绍如何使用 Vue3 + Vite 技术栈搭建个人组件库，包含项目架构设计、组件开发、文档编写和 npm 发布全流程。

---

## 一、项目初始化与架构设计

### 1. 创建基础项目

```bash
npm create vite@latest my-ui --template vue
cd my-ui
```

### 2. 调整目录结构

```
├── packages/       # 组件源代码
│   └── button
│       ├── src
│       │   ├── Button.vue
│       │   └── style.scss
│       └── index.js
├── docs/           # 组件文档
├── examples/       # 开发测试示例
├── build/          # 打包配置
└── package.json
```

### 3. 安装必要依赖

```bash
npm install vuepress@next -D
npm install sass sass-loader -D
```

---

## 二、组件开发规范

### 1. 编写示例组件（Button）

```javascript
// packages/button/src/Button.vue
<template>
  <button :class="['my-button', type]">
    <slot></slot>
  </button>
</template>

<script setup>
const props = defineProps({
  type: {
    type: String,
    default: 'default'
  }
})
</script>

<style scoped lang="scss">
.my-button {
  padding: 8px 16px;
  border-radius: 4px;

  &.primary {
    background: #409eff;
    color: white;
  }

  &.default {
    border: 1px solid #dcdfe6;
  }
}
</style>
```

### 2. 组件入口文件

```javascript
// packages/button/index.js
import Button from "./src/Button.vue";

Button.install = (app) => {
  app.component(Button.name, Button);
};

export default Button;
```

---

## 三、打包配置与构建

### 1. 配置 vite.config.js

```javascript
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";

export default defineConfig({
  plugins: [vue()],
  build: {
    lib: {
      entry: "packages/index.js",
      name: "MYUI",
      fileName: (format) => `my-ui.${format}.js`,
    },
    rollupOptions: {
      external: ["vue"],
      output: {
        globals: {
          vue: "Vue",
        },
      },
    },
  },
});
```

### 2. 创建全局入口文件

```javascript
// packages/index.js
import Button from "./button";

const components = [Button];

const install = (app) => {
  components.forEach((component) => {
    app.use(component);
  });
};

export default {
  install,
  Button,
};
```

---

## 四、本地开发与测试

### 1. 链接本地包

```bash
npm link
cd examples
npm link my-ui
```

### 2. 在示例项目中使用

```javascript
// examples/main.js
import { createApp } from "vue";
import MYUI from "my-ui";
import App from "./App.vue";

const app = createApp(App);
app.use(MYUI);
app.mount("#app");
```

---

## 五、使用 VuePress 编写文档

### 1. 初始化文档项目

```bash
mkdir docs && cd docs
npm init -y
npm install vuepress@next
```

### 2. 创建文档结构

```
docs/
├── .vuepress/
│   └── config.js
└── components/
    └── button.md
```

### 3. 配置文档主题

```javascript
// docs/.vuepress/config.js
module.exports = {
  title: "组件库文档",
  themeConfig: {
    nav: [{ text: "组件", link: "/components/button" }],
    sidebar: [
      {
        title: "组件列表",
        children: ["/components/button"],
      },
    ],
  },
};
```

### 4. 编写组件文档

## Button 按钮

### 基础用法

```vue
<template>
  <my-button>默认按钮</my-button>
  <my-button type="primary">主要按钮</my-button>
</template>
```

---

## 六、发布到 npm

### 1. 配置 package.json

```json
{
  "name": "my-ui",
  "version": "1.0.0",
  "main": "dist/my-ui.umd.js",
  "module": "dist/my-ui.es.js",
  "files": ["dist", "packages"],
  "scripts": {
    "build": "vite build"
  }
}
```

### 2. 发布流程

```bash
npm login
npm version patch
npm run build
npm publish
```

---

## 七、全局安装使用

```javascript
// main.js
import { createApp } from "vue";
import MYUI from "my-ui";
import App from "./App.vue";

const app = createApp(App);
app.use(MYUI);
```

---

## 总结

通过以上步骤，我们完成了：

1. 项目架构设计与初始化
2. 标准化组件开发流程
3. 组件库打包构建配置
4. 专业文档编写方案
5. npm 发布与版本管理

> 提示：实际开发中建议使用 husky+lint-staged 规范代码提交，并配置 commitlint 保证提交信息规范性。
