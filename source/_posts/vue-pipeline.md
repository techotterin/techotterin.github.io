---
title: Vuex框架核心流程
date: 2018-09-20 01:11:05
author: techotter
tags:
  - Vue.js
  - Vuex
  - 状态管理
categories:
  - 前端框架
---

Vuex是一个专为Vue.js应用程序开发的状态管理模式。它采用集中式存储管理应用的所有组件的状态,并以相应的规则保证状态以一种可预测的方式发生变化。本文将介绍Vuex的整体流程、常见API用法以及初始化装载与注入的过程。

<!-- more -->

## Vuex的整体流程

Vuex为Vue Components建立起了一个完整的生态圈,包括开发中的API调用一环,围绕这个生态圈,简要介绍一下各模块在核心流程中的主要功能:

- Vue Components: Vue组件,HTML页面上负责接收用户操作等交互行为,执行dispatch方法触发对应action进行回应。
- dispatch: 操作行为触发方法,是唯一能执行action的方法。
- actions: 操作行为处理模块,负责处理Vue Components接收到的所有交互行为,包含同步或异步操作,支持多个同名方法按照注册的顺序依次触发,向后台API请求的操作就在这个模块中进行,包括触发其他action以及提交mutation的操作,该模块提供了Promise的封装以支持action的链式触发。
- commit: 状态改变提交操作方法,对mutation进行提交,是唯一能执行mutation的方法。 
- mutations: 状态改变操作方法,是Vuex修改state的唯一推荐方法,其他修改方式在严格模式下将会报错,该方法只能进行同步操作,且方法名只能全局唯一,操作之中会有一些Hook暴露出来以进行state的监控等。
- state: 页面状态管理容器对象,集中存储Vue components中data对象的零散数据,全局唯一,以进行统一的状态管理。页面显示所需的数据从该对象中进行读取,利用Vue的细粒度数据响应机制来进行高效的状态更新。
- getters: state对象读取方法,Vue Components通过该方法读取全局state对象。

总结如下:

1. Vue组件接收交互行为,调用dispatch方法触发action相关处理 
2. 若页面状态需要改变,则调用commit方法提交mutation修改state
3. 通过getters获取到state新值,重新渲染Vue Components,界面随之更新

## 目录结构

- module: 提供module对象与module对象树的创建功能
- plugins: 提供开发辅助插件,如时光穿梭功能,state修改的日志记录功能等  
- helpers.js: 提供action、mutations以及getters的查找API
- index.js: 是源码主入口文件,提供store的各module构建安装
- mixin.js: 提供了store在Vue实例上的装载注入
- util.js: 提供了工具方法如find、deepCopy、forEachValue以及assert等方法

## 常见API用法

### State

Vuex通过store选项,将状态从根组件注入到每一个子组件中:

```js
new Vue({
  el: '#app',
  store, 
  components: { Counter },
  template: `<div><counter></counter></div>`  
})
```

子组件可以通过`this.$store`访问到该store的实例:

```js
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count() {
      return this.$store.state.count  
    }
  }
}  
```

### Getter

可以将其理解为store的计算属性,getter的返回值会根据它的依赖被缓存起来,且只有当它的依赖值发生了改变才会被重新计算。它接收state作为第一个参数:

```js
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true }, 
      { id: 2, text: '...', done: false }
    ]
  },
  getters: { 
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    }
  }
})
```

外部调用:

```js
store.getters.doneTodos // [{ id: 1, text: '...', done: true }]
```

Getter也可以接受其他getter作为第二个参数:

```js
getters: {
  // ...
  doneTodosCount: (state, getters) => {
    return getters.doneTodos.length 
  }
}

store.getters.doneTodosCount // 1

// 在组件中使用
computed: {  
  doneTodosCount() {
    return this.$store.getters.doneTodosCount
  }
} 
```

getter还可以返回一个函数,来实现给getter传参:

```js
getters: {
  // ... 
  getTodoById: (state) => (id) => {
    return state.todos.find(todo => todo.id === id)
  }
}

store.getters.getTodoById(2) // { id: 2, text: '...', done: false }   
```

### Mutation

更改Vuex的store中的状态的唯一方法是提交mutation,非常类似于事件。每个mutation都有一个事件类型(type)和一个回调函数(handler),回调函数就是进行状态更改的地方,并且接收state作为第一个参数:

```js
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
}) 
```

需要注意:
- 不能直接调用mutation handler,应当以相应的type调用`store.commit`方法:
  ```js
  store.commit('increment') 
  ```
- 同时可以向`store.commit`传入额外的参数,即mutation的载荷(payload):
  ```js
  // ...
  mutations: { 
    increment (state, payload) {
      state.count += payload.amount
    }
  }
  
  store.commit('increment', { 
    amount: 10
  })
  
  // 等同于
  store.commit({
    type: 'increment',
    payload: 10  
  })
  ```

#### Mutation的提交
  
mutation必须是同步函数(若是异步,则可能存在当mutation触发的时候,回调函数还没有被调用的情况)。mutation的提交可以使用`this.$store.commit('...')`。

或者可以使用`mapMutations`辅助函数将组件中的methods映射为`store.commit`调用:

```js
import { mapMutations } from 'vuex'

export default {
  // ...
  methods: {
    ...mapMutations([  
      // 将 `this.increment()` 映射为 `this.$store.commit('increment')`
      'increment', 
       
      // 将 `this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`  
      'incrementBy' 
    ]),
    ...mapMutations({
      // 将 `this.add()` 映射为 `this.$store.commit('increment')`
      add: 'increment'
    })
  }
}
```

### Action

Action提交的是mutation,而不是直接变更状态,可以包含任意异步操作。一个简单的action:

```js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      context.commit('increment') 
    }
  }
})
```

Action函数接受一个与store实例具有相同方法和属性的context对象,因此可以调用`context.commit`提交一个mutation。

#### 分发Action

Action通过`store.dispatch`方法触发:

```js
store.dispatch('increment')
```

之所以这样使用,是因为mutation必须同步执行,而action则不必如此,可以在其内部执行异步操作:

```js
actions: {
  increment({ commit }) {
    setTimeout(_ => {
      commit('incrementAsync')  
    }, 1000)
  }
}

// 同时也支持载荷方式  
store.dispatch('incrementAsync', {
  amount: 10  
})

// 等同于
store.dispatch({
  type: 'incrementAsync', 
  amount: 10
})  
```

#### 组件中分发Action

在组件中使用`this.$store.dispatch('...')`分发action,或者使用`mapActions`辅助函数将组件的methods映射为`store.dispatch`调用。

#### 组合Action

因为`store.dispatch`返回的是一个Promise对象,所以可以使用`then()`方法来进行处理,亦或是可以使用`async/await`:

```js  
// 假设 getData() 和 getOtherData() 返回的是 Promise
actions: {
  async actionA ({ commit }) {
    commit('gotData', await getData())
  },
  async actionB ({ dispatch, commit }) {
    await dispatch('actionA') // 等待 actionA 完成
    commit('gotOtherData', await getOtherData())
  }
}
```

## 初始化装载与注入  

下面我们就来看看Vuex到底是如何在项目中进行装载与注入的。首先是入口文件,先来看入口处的export函数到底导出了哪些东西,详细可见官方[vuejs/vuex](https://github.com/vuejs/vuex/blob/dev/src/index.js):

```js
export default {
  Store,  
  install,
  mapState,
  mapMutations, 
  mapGetters,
  mapActions  
}
```

### 装载与注入

我们一般在使用Vuex的时候如下所示:

```js
// store.js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

// 创建并导出store对象
export default new Vuex.Store() 


// index.js文件引入  
import Vue from 'vue'
import App from './../pages/app.vue' 
import store from './store.js'

new Vue({
  el: '#root',
  router,   
  store, // <== 这里注入 
  render: h => h(App)
})
```

除了Vue的初始化代码,只是多了一个store对象的传入。我们来看下源码中的实现方式:

```js
// store.js
// 定义局部变量Vue,用于判断是否已经装载和减少全局作用域查找
let Vue 

// 判断若处于浏览器环境下且加载过Vue,则执行install方法
if (!Vue && typeof window !== 'undefined' && window.Vue) {
  install(window.Vue)
}
```

若是首次加载,将局部Vue变量赋值为全局的Vue对象,并执行`applyMixin`方法:

```js
// store.js  
export function install (_Vue) {
  if (Vue && _Vue === Vue) {
    if (process.env.NODE_ENV !== 'production') {
      console.error(
        '[vuex] already installed. Vue.use(Vuex) should be called only once.'
      )
    }
    return
  }
  Vue = _Vue
  applyMixin(Vue) 
}
```

下面是`applyMixin`的源码,如果是2.x以上的版本,可以使用Hook的形式进行注入,即在`beforeCreate`钩子前插入初始化代码(`vuexInit`):

```js
export default function (Vue) {
  const version = Number(Vue.version.split('.')[0])
  
  if (version >= 2) {
    Vue.mixin({ beforeCreate: vuexInit })
  } else {
    // override init and inject vuex init procedure
    // for 1.x backwards compatibility.
    const _init = Vue.prototype._init
    Vue.prototype._init = function (options = {}) {
      options.init = options.init
        ? [vuexInit].concat(options.init) 
        : vuexInit
      _init.call(this, options)
    }
  }

  // Vuex init hook, injected into each instances init hooks list.
  function vuexInit () {
    const options = this.$options
    // 将初始化Vue根组件时传入的store设置到this对象的$store属性上
    // 子组件从其父组件引用$store属性,层层嵌套进行设置   
    if (options.store) {
      this.$store = typeof options.store === 'function'
        ? options.store()
        : options.store
    } else if (options.parent && options.parent.$store) {
      this.$store = options.parent.$store
    }
  }
}
```

这也就是为什么我们在Vue的组件中可以通过`this.$store.xxx`来访问到Vuex的各种数据和状态的原因了,因为在任意组件中执行`this.$store`都能找到装载的那个store对象。

希望通过本文,你能更好地理解Vuex的核心流程和常见API用法,并能在实际项目中灵活运用。在下一篇文章中,我们将探讨Vuex源码的实现细节,敬请期待。
