# 113 vue-路由懒加载

当打包构建应用时，JavaScript 包会变得非常大，影响页面加载。 如果我们能把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应组件，这样就更加高效了。

结合 Vue 的异步组件和 Webpack 的代码分割功能，轻松实现路由组件的懒加载。

```javascript
const router = new VueRouter({
  routes: [
    { path: '/foo', component: () => import('./Foo.vue') }
  ]
})
```

## 把组件按组分块

有时候我们想把某个路由下的所有组件都打包在同个异步块 \(chunk\) 中。只需要使用 命名 chunk，一个特殊的注释语法来提供 chunk name **\(需要 Webpack &gt; 2.4\)**。

```javascript
const Foo = () => import(/* webpackChunkName: "group-foo" */ './Foo.vue')
const Bar = () => import(/* webpackChunkName: "group-foo" */ './Bar.vue')
const Baz = () => import(/* webpackChunkName: "group-foo" */ './Baz.vue')
```

Webpack 会将任何一个异步模块与相同的块名称组合到相同的异步块中。

## 参考链接

[https://router.vuejs.org/zh/guide/advanced/lazy-loading.html\#%E8%B7%AF%E7%94%B1%E6%87%92%E5%8A%A0%E8%BD%BD](https://router.vuejs.org/zh/guide/advanced/lazy-loading.html#%E8%B7%AF%E7%94%B1%E6%87%92%E5%8A%A0%E8%BD%BD)

