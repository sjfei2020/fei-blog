---
title: 你应该知道的vue router库
date: 2024-04-24
updated: 2024-04-24
---

### 🛠️ 1. 底层的实现方式

- 浏览器历史管理（History API）与路由的关系
- Hash模式与History模式的原理和区别
- 实现简单的路由库示例

#### 🌍 浏览器历史管理（History API）与路由的关系

前端路由器会使用History API将新的URL添加到浏览器的历史记录中。这样，当用户单击浏览器的“后退”按钮时，浏览器会返回到之前的URL，前端路由器会根据URL的变化重新渲染页面内容。

#### 🔀 Hash模式与History模式的原理和区别

Hash模式下，前端路由器会监听URL中hash值的变化，并根据hash值的变化来更新页面内容。这种方式的好处是不需要向服务器发送请求，因为hash值的变化不会触发浏览器向服务器发送请求，而是在浏览器端通过JavaScript实现的。

History模式是通过HTML5中的History API来实现前端路由的一种方式。History模式下，前端路由器会监听URL的变化，并根据URL的变化来更新页面内容。

| 特征      | Hash模式                                | History模式                            |
| ------- | ------------------------------------- | ------------------------------------ |
| URL格式   | 包含hash值，例如`http://example.com/#/home` | 不包含hash值，例如`http://example.com/home` |
| URL变化   | 不会触发浏览器向服务器发送请求                       | 会触发浏览器向服务器发送请求                       |
| 前进和后退功能 | 通过浏览器的历史记录实现                          | 通过HTML5中的History API实现               |
| 兼容性     | 兼容性较好，可以在大多数浏览器中使用                    | 需要浏览器支持HTML5中的History API才能使用        |

#### 💻 实现简单的路由库示例 [例子](https://codepen.io/knightgao/pen/MWRLJEo?editors=1111)

```js
class Router {  
    routes = {  
        '/': { render: () => console.log('首页') },  
        '/about': { render: () => console.log('关于') },  
        '/404': { render: () => console.log('404页面未找到') }  
    };  
    currentUrl = '';  
    beforeHooks = [];  
    afterHooks = [];  
    mode = 'history';  
  
    constructor({ mode = 'history', routes, currentUrl = '/' } = {}) {  
        this.mode = mode;  
        if (routes) this.routes = routes;  
        this.currentUrl = currentUrl;  
        this.init();  
    }  
  
    init() {  
        if (this.mode === 'history') {  
            window.addEventListener('popstate', () => this.routeChanged());  
        } else if (this.mode === 'hash') {  
            window.addEventListener('hashchange', () => this.routeChanged());  
        }  
        this.routeChanged(); // 处理初始路由  
    }  
  
    async routeChanged() {  
        const path = this.getCurrentPath();  
        await this.handleRouteChange(path);  
    }  
  
    async handleRouteChange(path) {  
        const from = this.currentUrl;  
        const to = path;  
  
        console.log("from", from)  
        console.log("to", to)  
  
        for (let hook of this.beforeHooks) await hook(from, to);  
  
        this.currentUrl = path;  
        this.handleRender();  
  
        for (let hook of this.afterHooks) await hook(to);  
    }  
  
    handleRender() {  
        const route = this.routes[this.currentUrl];  
        if (route) {  
            route.render();  
        } else {  
            this.routes['/404'].render();  
        }  
    }  
  
    getCurrentPath() {  
        if (this.mode === 'history') {  
            return window.location.pathname;  
        } else if (this.mode === 'hash') {  
            return window.location.hash.slice(1) || '/';  
        }  
    }  
  
    beforeEach(hook) {  
        this.beforeHooks.push(hook);  
    }  
  
    afterEach(hook) {  
        this.afterHooks.push(hook);  
    }  
  
    async navigate(path) {  
        if (this.mode === 'history') {  
            history.pushState({}, '', path);  
        } else if (this.mode === 'hash') {  
            window.location.hash = path;  
        }  
        await this.routeChanged();  
    }  
}
```

```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
  <meta charset="UTF-8">  
  <title>Router Navigation Example</title>  
  <script src="router.js"></script>  
</head>  
<body>  
<h1>Router Navigation Example</h1>  
<button id="home-btn">首页</button>  
<button id="about-btn">关于</button>  
<button id="not-found-btn">404页面</button>  
  
<script>  
  // 初始化路由器  
  const router = new Router({  
    mode: 'history', // 使用 history 模式  
    currentUrl: '/' // 设置初始URL为首页  
  });  
  
  // 绑定按钮点击事件到路由导航  
  document.getElementById('home-btn').addEventListener('click', () => {  
    router.navigate('/');  
  });  
  
  document.getElementById('about-btn').addEventListener('click', () => {  
    router.navigate('/about');  
  });  
  
  document.getElementById('not-found-btn').addEventListener('click', () => {  
    router.navigate('/non-route'); // 故意导航到一个不存在的路由  
  });  
</script>  
</body>  
</html>
```

### 📐 2. 路由的配置

- 静态添加路由和动态添加路由
- 路由基础配置
- 路由高级配置
- 嵌套路由的声明

#### 📌 静态添加路由和动态添加路由

静态添加路由

```js
const router = new Router(
{ mode: 'history', 
 routes: [ 
	 { path: '/', component: Home },
	 { path: '/about', component: About  } 
  ] 
  });
```
动态添加路由

```js
const router = new Router({
  mode: 'history',
  routes: [
    { path: '/', component: Home }
  ]
});

// 假设基于某些条件动态添加路由
if (userHasAccessToAboutPage) {
  router.addRoute({ path: '/about', component: () => import('./components/About.vue') });
}
```
#### 🛠️ 路由基础配置

基础的路由配置通常包含以下几个关键部分：

1. **path**：定义路由的URL路径。
2. **component**：指定该路径对应的Vue组件。
3. **name**：为路由指定一个唯一名称，可选但推荐，便于编程时使用。
4. **children**：定义嵌套的子路由，可选。
5. **meta**：提供额外的信息（如标题、是否需要登录等），可选。
6. **redirect**：设置重定向的路径，可选。

```js
const router = new Router({
  mode: 'history', // 路由模式
  routes: [
    {
      path: '/',
      component: Home,
      name: 'home'
    },
    {
      path: '/about',
      component: About,
      name: 'about',
      meta: { requiresAuth: true }
    },
    {
      path: '/user/:id',
      component: User,
      name: 'user',
      children: [
        {
          path: 'profile',
          component: () => import('./components/UserProfile.vue'), // 懒加载组件
          name: 'userProfile'
        }
      ]
    }
  ]
});
```

#### 🚀 路由高级配置

Vue Router还支持如下高级功能：

- **路由守卫**（beforeEnter）：可以在路由被解析之前进行一些操作，如验证用户是否登录。
- **懒加载**（component: () => import('...')）：允许你延迟加载路由组件，这对于大型应用是非常有用的，可以减少初始加载时间。
- **路由模式**（mode）：定义使用的路由模式，`'history'` 或 `'hash'`，其中`'history'`模式需要服务器配置支持。

### 🚦 3. 路由的使用

- 路由跳转
- 路由信息对象
- 路由视图

#### 🔄 路由跳转

##### 编程式导航

使用代码来实现路由跳转，通常用于需要动态决定导航目标的场景。

```js
const router = useRouter();
const route = useRoute();

const goToAbout = () => {
  // 编程式导航到/about页面
  router.push('/about');
};

const goToUser = (userId) => {
  // 使用路由名称和参数进行导航
  router.push({ name: 'user', params: { id: userId } });
};
```

##### 声明式导航

```html
<router-link to="/">Home</router-link>
<router-link to="/about">About</router-link>
<router-link :to="{ name: 'user', params: { id: 123 } }">User 123</router-link>
```

#### 📄 路由信息对象

```js
const route = useRoute(); 
// 访问路由参数 
const userId = computed(() => route.params.id);
```

#### 🖼️ 路由视图(router-view)

##### 名称视图 [文档](https://router.vuejs.org/guide/essentials/named-views)

```vue
<template>
  <div>
    <router-view name="header"></router-view>
    <router-view></router-view> <!-- 默认无名视图 -->
    <router-view name="footer"></router-view>
  </div>
</template>
```

配置中可以这样指定各个视图的组件

```js
const routes = [
  {
    path: '/',
    components: {
      default: Home,
      header: Header,
      footer: Footer
    }
  }
];
```

##### 路由组件的属性传递 [文档](https://router.vuejs.org/guide/essentials/passing-props.html)

路由组件需要根据URL中的参数来获取数据。你可以通过设置`props`在路由配置中直接将URL参数、查询参数等传递给组件，而无需在组件内部通过`$route`来访问.

当 `props` 设置为 `true` 时，将 `route.params` 设置为组件 props

```js
const routes = [ { path: '/user/:id', component: User, props: true } ]
```

或者返回个对象

```js
const routes = [
  {
    path: '/user/:id',
    component: User,
    props: route => ({ id: route.params.id, query: route.query })
  }
];
```

在组件中可以直接使用

```vue
<script>
export default {
  props: {
    id: String
  }
}
</script>

<template>
  <div>
    User {{ id }}
  </div>
</template>
```
### 🔄 4. 动态路由

- 动态路由的定义与应用场景
- 参数传递与接收的方式
- 动态路由的匹配规则

#### 📌 动态路由的定义与应用场景 

##### 定义

动态路由可以通过在路由路径中使用“参数”(params)来定义。这些参数以冒号`:`开头

```js
const routes = [ { path: '/user/:id', component: User } ];
```

其中  **:id** 是个动态参数，vue 会解析为 $route.params.id 的值，在组件内可以直接访问这个值

##### 应用场景

动态路由主要包括详情页面，文章的页面，根据日期显示内容的页面，可以避免url参数传参，也可以结合一起使用，会更加的简短

#### 🔄 参数传递与接收的方式

```vue
<script setup>
import { useRoute } from 'vue-router';

const route = useRoute();
const userId = route.params.id;
</script>

<template>
  <div>User ID: {{ userId }}</div>
</template>
```

例子写的很明白了，直接获取即可

#### 🔄 动态路由的匹配规则

Vue Router在匹配路由时，会按照路由定义的顺序进行匹配，直到找到一个匹配的路由为止。如果有多个路由规则匹配同一个路径，第一个定义的路由将会被使用。这就意味着我们需要注意路由定义的顺序。

对于动态路由，如果有多个相似的路径需要区分，如`/user/:id`和`/user/new`，应该将最具体的路由（即没有动态段的路由）放在含有动态段的路由之前：

```js
const routes = [
  { path: '/user/new', component: CreateUser },
  { path: '/user/:id', component: User }
];
```

这样访问 */user/new* 的时候，会匹配到CreateUser。

### 🌐 5. 扩展内容

- **微前端的路由**
    - 微前端概念简介
    - 微前端下的路由管理策略
- **Nginx的配置**
    - Nginx作为前端路由的应用场景
    - 常见的Nginx路由配置案例
- **路由匹配**
    - 路由优先级和匹配机制
    - 路由重定向和别名的配置
- **路由参数**
    - 查询参数和路径参数的使用
    - 参数的响应式获取方法
- **路由守卫**
    - 路由守卫的作用和分类
    - 全局守卫、路由独享守卫和组件内守卫的应用
- **路由懒加载**
    - 懒加载的原理和好处
    - 实现路由懒加载的方法
- **路由history**
    - History模式的深入理解
    - 页面刷新和路由状态的保持

---