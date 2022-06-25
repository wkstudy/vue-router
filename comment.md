# 阅读笔记
<!-- 1. `this.$router`访问的是啥？ -->
2. 滚动的先不看
3. `this.$router.push()`看源码好像就是执行了下钩子函数，并同步到url上，没看到组件是这么切换的
   1. 那就是 `<router-view/>`在做的是事情了，看来vuerouter主要是要一个要一个路由配置对象+`<router-view/>`就可以了，<router-link/>其实是可有可无的


## vue-router 做了哪些事
1. 根据用户定义的路由做一套url到组件的映射
2. 封装window.history 
   1. 监听url变化，并根据映射关系加载正确的组件
   2. 暴露一些api如push、replace等方法给this.$router，让用户自在的跳转路由

## 梳理
```
const router = new VueRouter({
  mode: 'hash',
  routes: [
    {
      path: '/home',
      component:  import('../xxx/Home.vue')
    }
  ]
})
Vue.use(VueRouter)
new Vue({
  router,
})
```
1. `Vue.use(VueRouter)`做了什么
   1. 把初始化后的router
    1. 记录到`this._router`上（Vue私有属性）
    2. 记录到`this.$router`上（让用户使用），可以认为`this.$router`就是自己定义的router（被VueRouter处理过）
  2. 注册路由组件`RouterView 、RouterLink`

2. `new VueRouter()`做了什么
   主要是初始化一个VueRouter()对象,比较关键的是两个属性`this.matcher`和`this.history`
  1. `this.matcher` 主要维护 路由和组件的映射关系
  2. `this.history` 是对window.history的封装，主要是监听路由和暴露给用户操作路由的方法

3. `this.$router.push('/world')` 发生了什么（假设`mode=history`)
   1. 实际调用的是 `this.$router.history.push()`
   2. history.push
    ```
    this.transitionTo(location, route => {
          pushState(cleanPath(this.base + route.fullPath)) // 同步到url上
          handleScroll(this.router, route, fromRoute, false)
          onComplete && onComplete(route)
        }, onAbort)

    ```
 3. transitionTo
 ```
  route = this.router.match(location, this.current) 
   this.confirmTransition(
      route,
      () => {
        this.updateRoute(route)
        onComplete && onComplete(route)
        this.ensureURL()
        this.router.afterHooks.forEach(hook => {
          hook && hook(route, prev)
        }
      },
      }
  ```
  4. confirmTransition  （执行所有的钩子函数）
   