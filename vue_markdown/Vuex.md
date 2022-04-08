# Vuex



## 0. 开始

每一个 Vuex 应用的核心就是 store（仓库）。“store”基本上就是一个容器，它包含着你的应用中大部分的**状态 (state)**。

Vuex 和单纯的全局对象有以下两点不同：

1. Vuex 的状态存储是响应式的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。

2. 你不能直接改变 store 中的状态。改变 store 中的状态的唯一途径就是显式地**提交 (commit) mutation**。

   （这样使得我们可以方便地跟踪每一个状态的变化，从而让我们能够实现一些工具帮助我们更好地了解我们的应用。）

**例子：**

```js
import Vue from 'vue'
import Vuex from 'vuex'
Vue.use(Vuex)
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})
```

在所有子组件中，可以通过 store.state 来获取状态对象，以及通过 store.commit 方法触发状态变更

```js
// 根组件中注入 store
new Vue({
  el: '#app',
  store
})
```

```js
// 所有子组件中
methods:{
  increment(){
    this.$store.commit('increment'); 	// 提交 mutation
		console.log(this.$store.state.count);	// -> 1
  }
}
```



## 1. State

### 1) 在 Vue 组件中获得 Vuex 状态

由于 Vuex 的状态存储是响应式的，从 store 实例中读取状态最简单的方法就是在 **计算属性 **中返回某个状态

```js
// 创建一个 Counter 组件
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return this.$store.state.count
    }
  }
}
```

每当 `store.state.count` 变化的时候, 都会重新求取计算属性，并且触发更新相关联的 DOM。



### 2) `mapState` 辅助函数

当一个组件需要获取多个状态的时候，将这些状态都声明为计算属性会有些重复和冗余。

为了解决这个问题，我们可以使用 `mapState` 辅助函数帮助我们生成计算属性

mapState 函数返回一个对象

```js
import { mapState } from 'vuex'
export default{
  computed: mapState({
    count: state => state.count,
    countAlias: 'count',
    countPlusLocalState: (state) => {
      return state.count += this.localCount
    }
  })
}
```

当映射的计算属性的名称与 state 的子节点名称相同时，我们也可以给 `mapState` 传一个字符串数组。

```js
computed: mapState([
  'count'	// 映射 this.count 为 store.state.count
])
```

### 3) 对象展开运算符 ...mapState

`mapState` 函数返回的是一个对象。我们如何将它与局部计算属性混合使用呢？

通常，我们需要使用一个工具函数将多个对象合并为一个，以使我们可以将最终对象传给 `computed` 属性。

但是自从有了[对象展开运算符](https://github.com/tc39/proposal-object-rest-spread)[ ](https://github.com/tc39/proposal-object-rest-spread)[ (opens new window)](https://github.com/tc39/proposal-object-rest-spread)，我们可以极大地简化写法：

```js
computed:{
  ...mapState({
    'count'
  }),
  localComputed(){
    // ...
  }
}
```



## 2. Getters

### 1) 使用Getters

Vuex 允许我们在 store 中定义“getter”（可以认为是 store 的计算属性）。

就像计算属性一样，getter 的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。

Getter 接受 state 作为其第一个参数：

```js
const store = new Vuex.Store({
	state:{
		todos:[
			{ id: 1, name: "name1", done: true },
			{ id: 2, name: "name2", done: false }
		]
	},
	getters：{
		doneTodos: (state) => {
			return state.todos.filter(todo =>  todo.done)
		}
	}
})
```

### 2) 通过属性访问

```js
// 组件中
store.getters.doneTods  // -> [ { id: 1, name: "name1", done: true } ]
```

Getter 也可以接受 getters 作为第二个参数，以此来访问其他 getter

```js
getters:{
  // ...
  doneTodosCount: (state, getters){
    return getters.doneTodos.length;
  }
}
```

```js
// 组件中
store.getters.doneTodosCount  // -> 1
```

### 3) 通过方法访问

也可以通过让 getter 返回一个函数，来实现给 getter 传参。在你对 store 里的数组进行查询时非常有用。

```js
getters: {
  // ...
  getTodoById: (state) => (id) => {
    return state.todos.find(todo => todo.id === id)
  }
}
```

```js
// 组件中
store.getters.getTodoById(2) // -> { id: 2, name: "name2", done: false }
```

**注意，getter 在通过方法访问时，每次都会去进行调用，而不会缓存结果。**

### 4) `mapGetters` 辅助函数

`mapGetters` 辅助函数仅仅是将 store 中的 getter 映射到局部计算属性：

```js
import { mapGetters } from 'vuex'

export default {
  // ...
  computed: {
  // 使用对象展开运算符将 getter 混入 computed 对象中
    ...mapGetters([
      'doneTodosCount',
      'anotherGetter',
      'getTodoById',  // this.getTodoById(id) -> this.$store.getters.getTodoById(id)
      // ...
    ])
  }
}
```

如果你想将一个 getter 属性另取一个名字，使用对象形式：

```js
...mapGetters({
  // 把 `this.doneCount` 映射为 `this.$store.getters.doneTodosCount`
  doneCount: 'doneTodosCount'
})
```



## 3. Mutations

更改 Vuex 的 store 中的状态的唯一方法是**提交 mutation**。

Vuex 中的 mutation 非常类似于事件：每个 mutation 都有一个字符串的 **事件类型 (type)** 和 一个 **回调函数 (handler)**。

这个回调函数就是我们实际进行状态更改的地方，并且它会接受 state 作为第一个参数：

```js
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment (state) {
      // 变更状态
      state.count++
    }
  }
})
```

```js
// 组件中
this.$store.commit('increment')
```

### 1) 提交载荷

你可以向 `store.commit` 传入额外的参数，即 mutation 的 **载荷（payload）**：

```js
// store 中
mutations: {
  increment (state, n) {
    state.count += n
  }
}
```

```js
// 组件中
store.commit('increment', 10)
```

#### a. 载荷是一个对象

```js
// store 中
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
```

```js
// 组件中
store.commit('increment', {
  amount: 10
})
```

#### b. 对象风格提交

```js
// 组件中
store.commit({
	type: 'increment',
  amount: 10
}
```



### 2) Mutation 需遵守 Vue 的响应规则

既然 Vuex 的 store 中的状态是响应式的，那么当我们变更状态时，监视状态的 Vue 组件也会自动更新。

这也意味着 Vuex 中的 mutation 也需要与使用 Vue 一样遵守一些注意事项：

1. 最好提前在你的 store 中初始化好所有所需属性。
2. 当需要在对象上添加新属性时，你应该

- 使用 `Vue.set(obj, 'newProp', 123)`, 或者

- 以新对象替换老对象。

  例如，利用[对象展开运算符]我们可以这样写：`state.obj = { ...state.obj, newProp: 123}`
  
  

### 3) 使用常量替换 Mutation 事件类型

​	使用常量替代 mutation 事件类型在各种 Flux 实现中是很常见的模式。这样可以使 linter 之类的工具发挥作用，同时把这些常量放在单独的文件中可以让你的代码合作者对整个 app 包含的 mutation 一目了然

```js
// mutation-types.js
export const SOME_MUTATION = 'SOME_MUTATION'
```

```js
// store.js
import Vuex from 'vuex'
import { SOME_MUTATION } from './mutation-types'

const store = new Vuex.Store({
  state: { ... },
  mutations: {
    // 我们可以使用 ES2015 风格的计算属性命名功能来使用一个常量作为函数名
    [SOME_MUTATION] (state) {
      // mutate state
    }
  }
})
```



### 3) Mutation 必须是同步函数



### 4) 组件中提交 Mutation

可以在组件中使用 `this.$store.commit('xxx')` 提交 mutation。

或者使用 `mapMutations` 辅助函数将组件中的 methods 映射为 `store.commit` 调用（需要在根节点注入 `store`）。

```js
import { mapMutations } from 'vuex'

export default {
  // ...
  methods: {
    ...mapMutations([
      'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`

      // `mapMutations` 也支持载荷：
      'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`
    ]),
    ...mapMutations({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
    })
  }
}
```



## 4. Actions



### 1) 使用 actions

Action 类似于 mutation，不同在于：

- Action 提交的是 mutation，而不是直接变更状态。
- Action 可以包含任意异步操作。

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

Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象，

`context` 对象包含以下属性：

```js
{
  state,      // 等同于 `store.state`，若在模块中则为局部状态
  commit,     // 等同于 `store.commit`
  dispatch,   // 等同于 `store.dispatch`
  getters,    // 等同于 `store.getters`
  rootState,  // 等同于 `store.state`，只存在于模块中
  rootGetters // 等同于 `store.getters`，只存在于模块中
}
```

因此你可以调用 `context.commit` 提交一个 mutation，或者通过 `context.state` 和 `context.getters` 来获取 state 和 getters。

同时如果有第二个参数 `payload` 的话也能够接收。

可以利用参数解构简化代码如下：

```js
actions: {
  increment ({ commit }) {
    commit('increment')
  }
}
```



### 2) 分发 action

Action 通过 `store.dispatch` 方法触发：

```js
store.dispatch('increment')
```

由于 mutation 只能是同步函数，因此可以通过分发 action ，在 action 中执行异步操作，并等待异步操作执行完成后提交 mutaion

```js
actions: {
  incrementAsync ({ commit }) {
    setTimeout(() => {
      commit('increment')
    }, 1000)
  }
}
```



Actions 支持同样的载荷方式和对象方式进行分发，

在action 中接收此载荷执行异步操作或提交mutation

```js
// 以载荷形式分发
store.dispatch('incrementAsync', {
  amount: 10
})

// 以对象形式分发
store.dispatch({
  type: 'incrementAsync',
  amount: 10
})
```



**例子：**

```js
actions: {
  checkout ({ commit, state }, products) {
    // 把当前购物车的物品备份起来
    const savedCartItems = [...state.cart.added]
    // 提交 mutation 结账请求，清空购物车
    commit(types.CHECKOUT_REQUEST)
    // 购物 API ，接受一个成功回调和一个失败回调
    shop.buyProducts(
      products,
      // 成功操作
      () => commit(types.CHECKOUT_SUCCESS),
      // 失败操作
      () => commit(types.CHECKOUT_FAILURE, savedCartItems)
    )
  }
}
```

进行一系列的异步操作，并且通过提交 mutation 来记录 action 产生的副作用（即状态变更）。



### 3) 在组件中分发 Action

可以在组件中使用 `this.$store.dispatch('xxx')` 分发 action，

或者使用 `mapActions` 辅助函数将组件的 methods 映射为 `store.dispatch` 调用（需要先在根节点注入 `store`）：

```js
import { mapActions } from 'vuex'

export default {
  // ...
  methods: {
    ...mapActions([
      'increment', // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`

      // `mapActions` 也支持载荷：
      'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.dispatch('incrementBy', amount)`
    ]),
    ...mapActions({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
    })
  }
}
```



### 4) 组合 Action

Action 通常是异步的，那么如何知道 action 什么时候结束呢？更重要的是，我们如何才能组合多个 action，以处理更加复杂的异步流程？

首先，被触发的 action 处理函数可以返回 Promise， `store.dispatch` 可以处理这个返回的 Peomise ，并且 `store.dispatch` 也会返回 Promise：

```js
actions: {
  actionA ({ commit }) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        commit('someMutation')
        resolve()
      }, 1000)
    })
  }
}
```

现在你可以：

```js
store.dispatch('actionA').then(() => {
  // ...
})
```

在另外一个 action 中也可以：

```js
actions: {
  // ...
  actionB ({ dispatch, commit }) {
    return dispatch('actionA').then(() => {
      commit('someOtherMutation')
    })
  }
}
```

最后，如果我们利用 async / await

我们可以如下组合 action：

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

> 一个 `store.dispatch` 在不同模块中可以触发多个 action 函数。在这种情况下，只有当所有触发函数完成后，返回的 Promise 才会执行。







































