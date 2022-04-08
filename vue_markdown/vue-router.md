# 起步 例子

```html
<script src="https://unpkg.com/vue/dist/vue.js"></script>
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>

<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!-- use router-link component for navigation. -->
    <!-- specify the link by passing the `to` prop. -->
    <!-- <router-link> will be rendered as an `<a>` tag by default -->
    <router-link to="/foo">Go to Foo</router-link>
    <router-link to="/bar">Go to Bar</router-link>
  </p>
  <!-- route outlet -->
  <!-- component matched by the route will render here -->
  <router-view></router-view>
</div>
```



```javascript
// 0. If using a module system, call Vue.use(VueRouter)

// 1. Define route components.
// These can be imported from other files
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }

// 2. Define some routes
// Each route should map to a component. The "component" can
// either be an actual component constructor created via
// Vue.extend(), or just a component options object.
// We'll talk about nested routes later.
const routes = [
  { path: '/foo', component: Foo },
  { path: '/bar', component: Bar }
]

// 3. Create the router instance and pass the `routes` option
// You can pass in additional options here, but let's
// keep it simple for now.
const router = new VueRouter({
  routes
})

// 4. Create and mount the root instance.
// Make sure to inject the router with the router option to make the
// whole app router-aware.
const app = new Vue({
  router
}).$mount('#app')

// Now the app has started!
```



# 动态路由匹配 例子

```javascript
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)

// The matching uses path-to-regexp, which is the matching engine used
// by express as well, so the same matching rules apply.
// For detailed rules, see https://github.com/pillarjs/path-to-regexp
const router = new VueRouter({
  mode: 'history',
  base: __dirname,
  routes: [
    { path: '/' },
    // params are denoted with a colon ":"
    { path: '/params/:foo/:bar' },
    // a param can be made optional by adding "?"
    { path: '/optional-params/:foo?' },
    // a param can be followed by a regex pattern in parens
    // this route will only be matched if :id is all numbers
    { path: '/params-with-regex/:id(\\d+)' },
    // asterisk can match anything
    { path: '/asterisk/*' },
    // make part of the path optional by wrapping with parens and add "?"
    { path: '/optional-group/(foo/)?bar' }
  ]
})

new Vue({
  router,
  template: `
    <div id="app">
      <h1>Route Matching</h1>
      <ul>
        <li><router-link to="/">/</router-link></li>
        <li><router-link to="/params/foo/bar">/params/foo/bar</router-link></li>
        <li><router-link to="/optional-params">/optional-params</router-link></li>
        <li><router-link to="/optional-params/foo">/optional-params/foo</router-link></li>
        <li><router-link to="/params-with-regex/123">/params-with-regex/123</router-link></li>
        <li><router-link to="/params-with-regex/abc">/params-with-regex/abc</router-link></li>
        <li><router-link to="/asterisk/foo">/asterisk/foo</router-link></li>
        <li><router-link to="/asterisk/foo/bar">/asterisk/foo/bar</router-link></li>
        <li><router-link to="/optional-group/bar">/optional-group/bar</router-link></li>
        <li><router-link to="/optional-group/foo/bar">/optional-group/foo/bar</router-link></li>
      </ul>
      <p>Route context</p>
      <pre>{{ JSON.stringify($route, null, 2) }}</pre>
    </div>
  `
}).$mount('#app')

```

# 命名视图 例子

```html
<script src="https://unpkg.com/vue/dist/vue.js"></script>
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>

<div id="app">
  <h1>Named Views</h1>
  <ul>
    <li>
      <router-link to="/">/</router-link>
    </li>
    <li>
      <router-link to="/other">/other</router-link>
    </li>
  </ul>
  <router-view class="view one"></router-view>
  <router-view class="view two" name="a"></router-view>
  <router-view class="view three" name="b"></router-view>
</div>
```

```javascript
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }
const Baz = { template: '<div>baz</div>' }

const router = new VueRouter({
  mode: 'history',
  routes: [
    { path: '/',
      // a single route can define multiple named components
      // which will be rendered into <router-view>s with corresponding names.
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    },
    {
      path: '/other',
      components: {
        default: Baz,
        a: Bar,
        b: Foo
      }
    }
  ]
})

new Vue({
	router,
  el: '#app'
})
```

# 重定向 例子

```javascript
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)

const Home = { template: '<router-view></router-view>' }
const Default = { template: '<div>default</div>' }
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }
const Baz = { template: '<div>baz</div>' }
const WithParams = { template: '<div>{{ $route.params.id }}</div>' }
const Foobar = { template: '<div>foobar</div>' }
const FooBar = { template: '<div>FooBar</div>' }

const router = new VueRouter({
  mode: 'history',
  base: __dirname,
  routes: [
    { path: '/', component: Home,
      children: [
        { path: '', component: Default },
        { path: 'foo', component: Foo },
        { path: 'bar', component: Bar },
        { path: 'baz', name: 'baz', component: Baz },
        { path: 'with-params/:id', component: WithParams },
        // relative redirect to a sibling route
        { path: 'relative-redirect', redirect: 'foo' }
      ]
    },
    // absolute redirect
    { path: '/absolute-redirect', redirect: '/bar' },
    // dynamic redirect, note that the target route `to` is available for the redirect function
    { path: '/dynamic-redirect/:id?',
      redirect: to => {
        const { hash, params, query } = to
        if (query.to === 'foo') {
          return { path: '/foo', query: null }
        }
        if (hash === '#baz') {
          return { name: 'baz', hash: '' }
        }
        if (params.id) {
          return '/with-params/:id'
        } else {
          return '/bar'
        }
      }
    },
    // named redirect
    { path: '/named-redirect', redirect: { name: 'baz' }},

    // redirect with params
    { path: '/redirect-with-params/:id', redirect: '/with-params/:id' },

    // redirect with caseSensitive
    { path: '/foobar', component: Foobar, caseSensitive: true },

    // redirect with pathToRegexpOptions
    { path: '/FooBar', component: FooBar, pathToRegexpOptions: { sensitive: true }},

    // catch all redirect
    { path: '*', redirect: '/' }
  ]
})

new Vue({
  router,
  template: `
    <div id="app">
      <h1>Redirect</h1>
      <ul>
        <li><router-link to="/relative-redirect">
          /relative-redirect (redirects to /foo)
        </router-link></li>
        <li><router-link to="/relative-redirect?foo=bar">
          /relative-redirect?foo=bar (redirects to /foo?foo=bar)
        </router-link></li>
        <li><router-link to="/absolute-redirect">
          /absolute-redirect (redirects to /bar)
        </router-link></li>
        <li><router-link to="/dynamic-redirect">
          /dynamic-redirect (redirects to /bar)
        </router-link></li>
        <li><router-link to="/dynamic-redirect/123">
          /dynamic-redirect/123 (redirects to /with-params/123)
        </router-link></li>
        <li><router-link to="/dynamic-redirect?to=foo">
          /dynamic-redirect?to=foo (redirects to /foo)
        </router-link></li>
        <li><router-link to="/dynamic-redirect#baz">
          /dynamic-redirect#baz (redirects to /baz)
        </router-link></li>
        <li><router-link to="/named-redirect">
          /named-redirect (redirects to /baz)
        </router-link></li>
        <li><router-link to="/redirect-with-params/123">
          /redirect-with-params/123 (redirects to /with-params/123)
        </router-link></li>
        <li><router-link to="/foobar">
          /foobar
        </router-link></li>
        <li><router-link to="/FooBar">
          /FooBar
        </router-link></li>
        <li><router-link to="/not-found">
          /not-found (redirects to /)
        </router-link></li>
      </ul>
      <router-view class="view"></router-view>
    </div>
  `
}).$mount('#app')
```

# 别名 例子

```javascript
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)

const Root = { template: '<div>root</div>' }
const Home = { template: '<div><h1>Home</h1><router-view></router-view></div>' }
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }
const Baz = { template: '<div>baz</div>' }
const Default = { template: '<div>default</div>' }
const Nested = { template: '<router-view/>' }
const NestedFoo = { template: '<div>nested foo</div>' }

const router = new VueRouter({
  mode: 'history',
  base: __dirname,
  routes: [
    { path: '/root', component: Root, alias: '/root-alias' },
    { path: '/home', component: Home,
      children: [
        // absolute alias
        { path: 'foo', component: Foo, alias: '/foo' },
        // relative alias (alias to /home/bar-alias)
        { path: 'bar', component: Bar, alias: 'bar-alias' },
        // multiple aliases
        { path: 'baz', component: Baz, alias: ['/baz', 'baz-alias'] },
        // default child route with empty string as alias.
        { path: 'default', component: Default, alias: '' },
        // nested alias
        { path: 'nested', component: Nested, alias: 'nested-alias',
          children: [
            { path: 'foo', component: NestedFoo }
          ]
        }
      ]
    }
  ]
})

new Vue({
  router,
  template: `
    <div id="app">
      <h1>Route Alias</h1>
      <ul>
        <li><router-link to="/root-alias">
          /root-alias (renders /root)
        </router-link></li>
        <li><router-link to="/foo">
          /foo (renders /home/foo)
        </router-link></li>
        <li><router-link to="/home/bar-alias">
          /home/bar-alias (renders /home/bar)
        </router-link></li>
        <li><router-link to="/baz">
          /baz (renders /home/baz)
        </router-link></li>
        <li><router-link to="/home/baz-alias">
          /home/baz-alias (renders /home/baz)
        </router-link></li>
        <li><router-link to="/home">
          /home (renders /home/default)
        </router-link></li>
        <li><router-link to="/home/nested-alias/foo">
          /home/nested-alias/foo (renders /home/nested/foo)
        </router-link></li>
      </ul>
      <router-view class="view"></router-view>
    </div>
  `
}).$mount('#app')
```

