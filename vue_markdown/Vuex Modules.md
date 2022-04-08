# Vuex

​	由于使用单一状态树，应用的所有状态会集中到一个比较大的对象。当应用变得非常复杂时，store 对象就有可能变得相当臃肿。

​	为了解决以上问题，Vuex 允许我们将 store 分割成**模块（module）**。

​	每个模块拥有自己的 state、getters、mutations、actions、甚至是嵌套子模块 modules——从上至下进行同样方式的分割



## 1. 分割模块

```javascript
const moduleA = {
  state: () => ({ ... }),
  getters: { ... },
  mutations: { ... },
  actions: { ... },
}

const moduleB = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,	
    b: moduleB
  }
})

store.state.a // -> moduleA 的 state 对象
store.state.b // -> moduleB 的 state 对象

```



## 2. 模块的局部状态

### 1) 模块内部的 getter 和 mutation

对于模块内部的 getter 和 mutation ，接收的第一个参数是**模块的局部状态对象**。

```js
const moduleA = {
  state: () => ({
    count: 0
  }),
  getters: {
    doubleCount (state[, getters, rootState, rootGetters]) {
     // state是模块的局部状态对象，rootState是根节点的状态对象
      return state.count * 2
    }
  },
  mutations: {
    increment (state) {  // 这里的 `state` 对象是模块的局部状态对象
      state.count++
    }
  },
}
```



### 2) 模块内部的action

对于模块内部的 action，通过 `context.state` 访问局部状态，通过 `context.rootState` 根节点状态

```js
const moduleA = {
  // ...
  actions: {
    incrementIfOddOnRootSum ({ state, commit, rootState }) {
      if ((state.count + rootState.count) % 2 === 1) {
        commit('increment')
      }
    }
  }
}
```

####  `context` 对象

`context` 对象包含以下属性：

```js
{
  state,      // 等同于 `store.state`，若在模块中则为局部状态
  getters,    // 等同于 `store.getters`
  commit,     // 等同于 `store.commit`
  dispatch,   // 等同于 `store.dispatch`
  rootState,  // 等同于 `store.state`，只存在于模块中
  rootGetters // 等同于 `store.getters`，只存在于模块中
}
```

action 同时如果有第二个参数 `payload` 的话也能够接收。



## 3. 命名空间

默认情况下，模块内部的 getter 、 mutation 和 action 是注册在**全局命名空间**的——这样使得多个模块能够对同一 mutation 或 action 作出响应。

可以通过添加 **`namespaced: true`** 的方式使其成为带命名空间的模块。

当模块被注册后，它的所有 getter、mutation 及 action 都会自动根据模块注册的路径调整命名。

```js
const store = new Vuex.Store({
  modules: {
    account: {
      namespaced: true,

      // 模块内容（module assets）
      // 模块内的状态已经是嵌套的了，使用 `namespaced` 属性不会对其产生影响
      state: () => ({ ... }),  // -> store.state.account
      getters: {
        isAdmin () { ... }  // -> getters['account/isAdmin']
      },
      actions: {
        login () { ... }  // -> dispatch('account/login')
      },
      mutations: {
        login () { ... }  // -> commit('account/login')
      },

      // 嵌套模块
      modules: {
        // 没有 namespaced:true，所以继承父模块的命名空间(属于同一命名空间)
        myPage: {
          state: () => ({ ... }),
          getters: {
            profile () { ... } // -> getters['account/profile']
          }
        },

        // 进一步嵌套命名空间
        posts: {
          namespaced: true,

          state: () => ({ ... }),
          getters: {
            popular () { ... } // -> getters['account/posts/popular']
          }
        }
      }
    }
  }
})
```

启用了命名空间的 getter 会收到局部化的 `getter`，action 会收到局部化的 `dispatch` 和 `commit`。

换言之，你在使用模块内容（module assets）时不需要在同一模块内额外添加空间名前缀。更改 `namespaced` 属性后不需要修改模块内的代码。



## 4. 在带命名空间的模块内访问全局内容（Global Assets）

1）如果希望使用全局 state 和 getter，getter 中**`rootState`** 和 **`rootGetters`** 会作为第三和第四参数传入 ，action 中也会通过 `context` 对象的属性传入 。

2）若需要在全局命名空间内提交 mutation 或 分发 action ，将 `{ root: true }` 作为第三参数传给 `commit` 或  `dispatch` 即可。

```js
modules: {
  foo: {
    namespaced: true,
    getters: {
      // 在这个模块的 getter 中，`getters` 被局部化了
      // 你可以使用 getter 的第四个参数来调用 `rootGetters`
      someGetter (state, getters, rootState, rootGetters) {
        getters.someOtherGetter // -> 'foo/someOtherGetter'
        rootGetters.someOtherGetter // -> 'someOtherGetter'
      },
      someOtherGetter: state => { ... }
    },

    actions: {
      // 在这个模块中， dispatch 和 commit 也被局部化了
      // 他们可以接受 `root` 属性以访问根 dispatch 或 commit
      someAction ({ dispatch, commit, getters, rootGetters }) {
        getters.someGetter // -> 'foo/someGetter'
        rootGetters.someGetter // -> 'someGetter'

        dispatch('someOtherAction') // -> 'foo/someOtherAction'
        dispatch('someOtherAction', null, { root: true }) // -> 'someOtherAction'

        commit('someMutation') // -> 'foo/someMutation'
        commit('someMutation', null, { root: true }) // -> 'someMutation'
      },
      someOtherAction (ctx, payload) { ... }
    }
  }
}
```



## 5. 在带命名空间的模块注册全局 action

若需要在带命名空间的模块注册全局 action，你可添加 `root: true`，并将这个 action 的定义放在函数 `handler` 中。

```js
{
  actions: {
    someOtherAction ({dispatch}) {
      dispatch('someAction')
    }
  },
  modules: {
    foo: {
      namespaced: true,

      actions: {
        someAction: {
          root: true,
          handler (namespacedContext, payload) { ... } // -> 'someAction'
        }
      }
    }
  }
}
```



## 6. 带命名空间的绑定函数

当使用  `mapState` ,  `mapGetters `,  `mapActions`  和  `mapMutations` 这些函数来绑定带命名空间的模块时，写起来可能比较繁琐

```js
computed: {
  ...mapState({
    a: state => state.some.nested.module.a,
    b: state => state.some.nested.module.b
  })
  ...mapGetters({
    c: 'some/nested/module/c',
    d: 'some/nested/module/d',
  })
},
methods: {
  ...mapActions([
    'some/nested/module/foo', // -> this['some/nested/module/foo']()
    'some/nested/module/bar' // -> this['some/nested/module/bar']()
  ])
}
```

### 方法一

将模块的空间名称字符串作为第一个参数传递给上述函数，这样所有绑定都会自动将该模块作为上下文。

```js
computed: {
  ...mapState('some/nested/module', {
    a: state => state.a,
    b: state => state.b
  }),
  ...mapGetters('some/nested/module', {
    'c': 'gettera',
    'd': 'gettera'
    // ['gettera','getterb']
  }),
},
methods: {
  ...mapActions('some/nested/module', [
    'foo', // -> this.foo()
    'bar' // -> this.bar()
  ])
}
```

### 方法二

可以通过使用 `createNamespacedHelpers` 创建基于某个命名空间辅助函数。

它返回一个对象，对象里有新的绑定在给定命名空间值上的组件绑定辅助函数。

```js
import { createNamespacedHelpers } from 'vuex'
const { mapState, mapActions } = createNamespacedHelpers('some/nested/module')
export default {
  computed: {
    // 在 `some/nested/module` 中查找
    ...mapState({
      a: state => state.a,
      b: state => state.b
    })
  },
  methods: {
    // 在 `some/nested/module` 中查找
    ...mapActions([
      'foo',
      'bar'
    ])
  }
}
```



## 7. 模块动态注册

在 store 创建**之后**，你可以使用 `store.registerModule` 方法注册模块：

```js
import Vuex from 'vuex'
const store = new Vuex.Store({ /* 选项 */ })
// 注册模块 `myModule`
store.registerModule('myModule', {
  // ...
})
// 注册嵌套模块 `nested/myModule`
store.registerModule(['nested', 'myModule'], {
  // ...
})
```

之后就可以通过 `store.state.myModule` 和 `store.state.nested.myModule` 访问模块的状态。

也可以使用 `store.unregisterModule(moduleName)` 来动态卸载模块。注意，你不能使用此方法卸载静态模块（即创建 store 时声明的模块）。

注意，你可以通过 `store.hasModule(moduleName)` 方法检查该模块是否已经被注册到 store

### 1) 保留 state

在注册一个新 module 时，你很有可能想保留过去的 state，例如从一个服务端渲染的应用保留 state。

你可以通过 `preserveState` 选项将其归档：`store.registerModule('a', module, { preserveState: true })`。

当你设置 `preserveState: true` 时，该模块会被注册，action、mutation 和 getter 会被添加到 store 中，但是 state 不会。这里假设 store 的 state 已经包含了这个 module 的 state 并且你不希望将其覆写。

### 2) 模块重用

如果我们使用一个纯对象来声明模块的状态，这个状态对象可能会通过引用被共享，那么会导致状态对象被修改时 store 或模块间数据互相污染的问题。

实际上这和 Vue 组件内的 `data` 是同样的问题。因此解决办法也是相同的——使用一个函数来声明模块状态（仅 2.3.0+ 支持）：

```js
const MyReusableModule = {
  state: () => ({
    foo: 'bar'
  }),
  // mutation, action 和 getter 等等...
}
```

































