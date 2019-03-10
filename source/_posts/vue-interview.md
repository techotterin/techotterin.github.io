---
title: Vue常见面试问题收集
date: 2019-03-11 10:01:00
author: techotter
tags: 
  - Vue
  - Vuex
categories:
  - 面试
---

本文是 Vue 常见面试问题收集，本文将简要介绍 Vue.js 的核心概念，包括 MVVM 模式、Vue 的生命周期、组件间通信、以及路由管理。希望本文能帮助你更好地理解 Vue.js，并在你的项目中得以应用。

<!-- more -->

## 1. MVVM 模式

### 什么是 MVVM？

MVVM 是 Model-View-ViewModel 的缩写。它是一种设计模式，用于分离 UI（用户界面）的表示和业务逻辑。

- **Model** 层代表数据模型，包含数据修改和操作的业务逻辑。
- **View** 层代表 UI 组件，负责将数据模型转化为用户界面。
- **ViewModel** 层是连接 View 和 Model 的桥梁，通过双向数据绑定将两者联系起来。

在 MVVM 架构中，View 和 Model 间没有直接联系，而是通过 ViewModel 进行交互。ViewModel 的双向数据绑定功能使得 Model 数据的变化能自动反映到 View 上，反之亦然。

### MVVM 和 MVC 的区别？

虽然 MVC（Model-View-Controller）和 MVVM 都是常见的设计模式，但 MVVM 主要是通过 ViewModel 替换了 MVC 中的 Controller，从而减少了直接操作 DOM 的需要，改善了页面渲染性能和用户体验。

## 2. Vue 的优点

Vue.js 设计理念的优点包括：

- **低耦合**：View 可以独立于 Model 修改，反之亦然。
- **可重用性**：可以将视图逻辑集中在一个 ViewModel 中，以便多个视图重用。
- **独立开发**：开发人员可以专注于业务逻辑和数据开发，设计人员可以专注于页面设计。
- **可测试性**：与传统的界面测试相比，基于 ViewModel 的测试更容易实现。

## 3. Vue 生命周期

Vue 实例的生命周期包括以下几个阶段：

- **创建前/后**：在 `beforeCreate` 阶段，实例的挂载元素 `el` 和数据对象 `data` 都未初始化。
- **载入前/后**：在 `beforeMount` 阶段，实例的 `$el` 和 `data` 初始化完成，但尚未挂载。在 `mounted` 阶段，实例挂载完成，页面渲染更新。
- **更新前/后**：当数据变化时，会触发 `beforeUpdate` 和 `updated` 阶段。
- **销毁前/后**：在执行 `destroy` 阶段后，实例解除事件监听和与 DOM 的绑定，但 DOM 结构依然存在。

## 4. 组件间的传值

### 父组件向子组件传值

```vue
<template>
  <Main :obj="data"></Main>
</template>

<script>
import Main from "./Main";

export default {
  name: "Parent",
  data() {
    return {
      data: "我要向子组件传递数据"
    };
  },
  components: {
    Main
  }
}
</script>
```

### 子组件向父组件传递数据

```vue
<template>
  <div v-on:click="emitEvent">{{ data }}</div>
</template>

<script>
export default {
  name: "Child",
  props: ["data"],
  methods: {
    emitEvent() {
      this.$emit('event', this.data); // 派发事件，传递数据
    }
  }
}
</script>
```

## 5. 路由与嵌套路由

Vue-router 提供了 `router-link` 组件的 `active-class` 属性，用于设置当前激活路由的 CSS 类。

嵌套路由通过 VueRouter 的 `children` 配置实现，使得可以构建多层嵌套的组件结构。

```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/parent',
      component: Parent,
      children: [
        {
          path: 'child',
          component: Child
        }
      ]
    }
  ]
});
```

接下来，我们继续扩展关于 Vue 和 Vuex 的内容，特别是 Vuex 的使用以及如何在 Vue 应用中有效地管理状态。

### Vuex 的使用

Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。Vuex 非常适合处理多个组件共享状态的情况，如用户认证信息、应用配置等。

#### Vuex 核心概念

1. **State**：
   - Vuex 使用单一状态树 —— 一个对象包含了全部的应用层级状态。作为一个唯一的数据源存在。
   - Vuex 的状态存储是响应式的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。

2. **Getters**：
   - Vuex 允许我们定义“getters”（可以认为是 store 的计算属性）。Getters 接受 state 作为其第一个参数。
   - Getters 的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。

3. **Mutations**：
   - 更改 Vuex 的 store 中的状态的唯一方法是提交 mutation。Vuex 中的 mutations 非常类似于事件：每个 mutation 都有一个字符串的事件类型 (type) 和一个回调函数 (handler)。
   - 这个回调函数就是我们实际进行状态更改的地方，并且它会接受 state 作为第一个参数。

4. **Actions**：
   - Action 类似于 mutation，不同在于：
     - Action 提交的是 mutation，而不是直接变更状态。
     - Action 可以包含任意异步操作。
   - Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象，因此你可以调用 context.commit 提交一个 mutation，或者通过 context.state 和 context.getters 来获取 state 和 getters。

5. **Modules**：
   - 由于使用单一状态树，应用的所有状态会集中到一个比较大的对象。当应用变得非常复杂时，store 对象就可能变得相当臃肿。
   - Vuex 允许我们将 store 分割成模块（module）。每个模块拥有自己的 state、mutation、action、getter 甚至是嵌套子模块 —— 这类似于组件的局部状态。

#### 示例：在 Vue 组件中使用 Vuex

```javascript
// main.js
import Vue from 'vue';
import Vuex from 'vuex';
import App from './App.vue';

Vue.use(Vuex);

const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment(state) {
      state.count++;
    }
  },
  actions: {
    increment({ commit }) {
      commit('increment');
    }
  }
});

new Vue({
  el: '#app',
  store,
  render: h => h(App)
});
```
