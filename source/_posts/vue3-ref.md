---
title: Vue3学习：Ref全家桶
tags:
  - Vue3
  - 前端开发
  - 响应式原理
abbrlink: 35100
date: 2025-05-10 02:17:49
---

## 初识 Ref 家族

在 Vue3 的响应式系统中，Ref 家族成员扮演着重要角色。它们帮助我们处理基本类型和复杂类型的响应式需求，下面让我们逐个认识这些实用的 API。

### ref：响应式数据声明

```javascript
import { ref } from "vue";

// 创建响应式数据
const count = ref(0);

// 修改值需要.value
function increment() {
  count.value++;
}

// 模板中使用不需要.value
// <div>{{ count }}</div>
```

**核心特点：**

- 支持所有数据类型
- 通过`.value`访问/修改值
- 深度响应（嵌套对象也会被代理）

**使用场景：**

- 基本类型数据
- 需要保持引用的对象
- 需要显式跟踪变化的场景（**比如用在模板中的数据**）

### isRef：判断是否为 Ref 数据对象

```javascript
import { ref, isRef } from "vue";

const num = ref(42);
const num2 = 43;
console.log(isRef(num)); // true
console.log(isRef(num2)); // false
```

**作用：**

- 判断变量是否为 ref 对象
- 类型检查时特别有用

### shallowRef：浅层响应式数据

```javascript
import { shallowRef } from "vue";

const user = shallowRef({
  name: "小明",
  address: {
    city: "北京",
  },
});

// 直接赋值触发响应
user.value = { name: "小红" };

// 修改深层属性不会触发
user.value.address.city = "上海";
```

**特点：**

- 只监听顶层变化
- 适合大型对象优化
- 深层响应的情况需要配合`triggerRef`使用

### triggerRef：强制触发更新

```javascript
import { shallowRef, triggerRef } from "vue";

const data = shallowRef({ count: 0 });

// 修改深层属性后手动触发
data.value.count++;
triggerRef(data);
```
> PS： ref为深层响应，shallowRef为浅层响应。ref更新时底层逻辑默认调用triggerRef，故ref和shallowRef不能用在同一个地方，不然shallowRef会受到ref的影响强制更新深层数据。

### customRef：定制版ref

```javascript
import { customRef } from "vue";

function useDebouncedRef(value, delay = 200) {
  let timeout;
  return customRef((track, trigger) => {
    return {
      get() {
        track();
        return value;
      },
      set(newValue) {
        clearTimeout(timeout);
        timeout = setTimeout(() => {
          value = newValue;
          trigger();
        }, delay);
      },
    };
  });
}

// 使用示例
const searchText = useDebouncedRef("");
```

customRef相当于把ref的响应逻辑交给开发人员控制，适合有定制需求的场景下使用。

## 成员特性对比

| 特性         | ref  | shallowRef | customRef |
| ------------ | ---- | ---------- | --------- |
| 响应深度     | 深   | 浅         | 自定义    |
| 自动触发更新 | ✔️   | ❌         | 自定义    |
| 性能消耗     | 较高 | 较低       | 灵活      |
| 使用频率     | 高   | 中         | 特殊场景  |

## 使用小贴士

1. **避免.value 陷阱**：在模板中会自动解包，但 JS 中操作必须使用.value
2. **性能优先原则**：深层对象建议先尝试 shallowRef
3. **组合使用技巧**：shallowRef+triggerRef 可替代部分 watch 场景
4. **解构注意事项**：解构 ref 对象会失去响应性，推荐使用 toRefs
5. **类型安全方案**：搭配 TypeScript 使用更安心

## 如何选择？

- 普通场景 ➡️ ref
- 大对象优化 ➡️ shallowRef
- 特殊需求 ➡️ customRef
- 类型判断 ➡️ isRef
- 强制更新 ➡️ triggerRef

实际开发中要根据数据结构和性能需求灵活选择，没有绝对的最佳实践。建议从简单场景开始尝试，逐步体会不同 API 的特性差异。
