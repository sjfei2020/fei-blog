---
title: 200行不到手写一个Router库
date: 2024-05-29
updated: 2024-05-29
---

## 200行不到手写一个Router库

### 引言

为什么要写一个 Router，首先 Router 库 是我们项目中常常使用的一个库，使用过的人很多。其次 库的主要功能其实非常简单，比较适合用来学习，对后续其他源码感兴趣也可以作为入门。最后，Router库 比较适合用来结合TDD，有点复杂又不太多

通过这篇文章，应该可以学会Router原理，TDD入门，打开源码阅读大门

### 什么是 Router？

- **Router**（路由器）在软件开发中，通常是指负责处理 URL 路径和相应功能之间映射的组件。它决定了用户在访问特定 URL 时，应用程序应该展示什么内容或者执行什么操作。
- 在前端开发中，Router 主要用于单页应用（*SPA*），通过改变浏览器的 URL 来切换不同的视图，而不需要重新加载整个页面。

#### 常见的路由库

- 在单页应用（SPA）中，Router 是核心组件之一。流行的前端框架如 React、Vue 和 Angular 都有各自的 Router 库（如 React Router、Vue Router 和 Angular Router）。
- 通过 Router，开发者可以定义视图之间的导航规则，处理动态路由参数，设置路由守卫等。

### 设计一个 Router

#### 需求分析

##### 1. URL 导航

- **作用**：Router 使得应用能够根据 URL 的变化来动态加载和展示不同的内容，这对于创建用户体验流畅的单页应用尤其重要。
- **示例**：用户在浏览器地址栏中输入不同的 URL，可以导航到应用的不同页面或组件。

##### 2. 路径管理

- **作用**：Router 允许开发者定义应用的路径结构和导航规则，使得应用的 URL 更加语义化和可读。
- **示例**：通过配置路由规则，开发者可以定义 `/home` 对应首页组件，`/about` 对应关于页面组件等。

##### 3. 状态保持

- **作用**：通过使用 Router，可以在浏览器的历史记录中保存导航状态，允许用户使用浏览器的前进和后退按钮进行导航。
- **示例**：用户在页面之间导航时，可以使用浏览器的前进和后退按钮返回到之前访问的页面。

##### 4. 组件加载

- **作用**：在前端，Router 可以与组件系统结合，按需加载和渲染组件，提高应用的性能和响应速度。
- **示例**：通过路由懒加载，只在用户访问特定路径时才加载相应的组件，减少初始加载时间。

##### 5. 动态路由

- **作用**：动态路由允许在路径中包含参数，使得同一路径可以根据不同的参数展示不同的内容。
- **示例**：在电商网站中，产品详情页可以使用动态路由，例如 `/product/:id`，其中 `:id` 是产品的唯一标识符。通过读取 URL 中的 `id` 参数，可以展示不同的产品详情。
- **实现**：动态路由通常通过在路径中使用占位符来实现，Router 会解析路径并将参数传递给相应的组件或处理函数。
##### 6. 路由守卫

- **作用**：路由守卫用于在路由切换前后执行特定逻辑，如权限验证、数据预加载等，确保用户只有在满足特定条件时才能访问某些页面。
- **示例**：在需要用户登录才能访问的页面，可以使用路由守卫进行权限检查，未登录用户将被重定向到登录页面。
- **实现**：路由守卫可以在路由配置中定义钩子函数，这些钩子函数会在路由切换前后触发，允许开发者执行相应的逻辑。

### 从零开始开干

前面铺垫了那么多，其实主要是围绕路由本身，从这里开始就进入了本文的正文部分，从零开始手写一个 *Router* 库。

我日常使用  Vue-Router 比较多，我们就从他开始吧。

#### 注册 Router Plugins

Vue 中的插件是一种能为 Vue 添加全局功能的工具代码。

```js
import { createApp } from 'vue' 
const app = createApp({}) 
app.use(myPlugin, { /* 可选的选项 */ })
```

插件没有严格定义的使用范围，但是插件发挥作用的常见场景主要包括以下几种：

1. 通过 [`app.component()`](https://cn.vuejs.org/api/application.html#app-component) 和 [`app.directive()`](https://cn.vuejs.org/api/application.html#app-directive) 注册一到多个全局组件或自定义指令。
   
2. 通过 [`app.provide()`](https://cn.vuejs.org/api/application.html#app-provide) 使一个资源[可被注入](https://cn.vuejs.org/guide/components/provide-inject.html)进整个应用。
   
3. 向 [`app.config.globalProperties`](https://cn.vuejs.org/api/application.html#app-config-globalproperties) 中添加一些全局实例属性或方法

所以我们的install 应该怎么写，已经很清晰了

```ts
class Router{
    install(app: App) {

		// 挂载对象
		app.config.globalProperties.$router = this;
		
		// 全局注入实例
		app.provide('router', this);

		// 此处也可以注册 router-view router-link 组件
		app.component('RouterLink', RouterLink);
    app.component('RouterView', RouterView);
	}
}
```

不难想象，在Vue2 中，也可以使用 mixin 混入完成.去把 beforeCreate 和 destroyed 钩子函数注入到每一个组件中。

对应的使用也很简单,main.ts文件中，实例化Vue 后，app.use(router)即可。

#### TDD方式开发

现在我们的代码应该是这个样子

```ts
class Router{
	constructor(){
	}
	
	install(app: App) {
		app.config.globalProperties.$router = this;
		app.provide('router', this);
		this.onRouteChange(); // Initialize the first route
	}
}
```

![image-20240529141600575](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss/img/image-20240529141600575.png)



按照这个想法，我们先写一个对应的test

```js
it('路由构造函数默认值', () => {
	const router = new Router();
	expect(router.mode).toBe('history');
	expect(router.routes.size).toBe(0);
});
```

编写代码
```ts
import { type App } from 'vue';

class Router {
	routes = new Map<string, any>();

	mode: string;

	constructor({ mode = 'history', routes = [] }: { mode?: string, routes?: any[] } = {}) {
		this.mode = mode;
		routes.forEach(route => {
			this.routes.set(route.path, route.component);
		});
	}
	
	install(app: App) {
		app.config.globalProperties.$router = this;
		app.provide('router', this);
	}
}

  

export default Router;
```



查看控制台的测试用例运行情况

![image-20240529142228815](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss/img/image-20240529142228815.png)

测试通过，取决于进不进入重构不是每次都需要的，觉得有脏气味再进入也不迟，毕竟过早优化是万恶之源。

为了节约文章篇幅，不然太水了，这部分后面就略了。

#### history 与 hash 模式

我们继续，mode 这里是支持 history 的，但是我们都知道还有一种叫hash模式的，作为一个Router库，肯定是这两种是必须要支持的。

原理也比较简单， history 模式主要监听的是popstate , hash模式主要监听hashchange事件，所以对应的代码应该都是这样。

```ts
init() {
	if (this.mode === 'history') {
		window.addEventListener('popstate', async () => {
			//
		});
	} else if (this.mode === 'hash') {
		window.addEventListener('hashchange', async () => {
			//
		});
	}
}
```

当然实际的源码中，为了摇树考虑，也特别做了处理，这里考虑到太稀碎了，小细节就跳过了。感兴趣可以看源码。



#### 路径管理

我们有了基础的架子后，怎么管理路径尼，参照 Vue Router ,我们定义个数据结构，

{ path: '/home', component: *HELLO* } , 当url 匹配，即可显示对应的component.

![image-20240529144445744](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss/img/image-20240529144445744.png)



#### URL导航

顾名思义，URL变化引起页面变化，通过上文的监听事件后，当URL变化的时候我们已经可以拿到事件了。

这时候想直接影响页面，有什么办法尼，直接在回调中获取DOM元素更新当然是可以的，但是太不优雅了，还记得插件里面我们可以注册全局组件吗，我们完全可以使用 *router-view* 组件来实现这个功能。

```vue 
<script setup>
import { inject, computed } from 'vue';

// 获取注入的路由器实例
const router = inject('router');

// 使用计算属性获取当前组件
const currentComponent = computed(() => router.getCurrentComponent().value);
</script>

<template>
  <component :is="currentComponent"></component>
</template>
```



在组件中获取到Router 实例，然后获取到当前组件，配合 内置的 component 组件，即可实现优雅的变化。

![image-20240529143847716](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss/img/image-20240529143847716.png)



#### 动态路由

动态路由应该是很多小伙伴写这个时候多困难点，这里我也是抛砖，

我们先看下 Vue Router 源码中是如何进行的：

![image-20240529152938986](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss/img/image-20240529152938986.png)

![image-20240529152959577](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss/img/image-20240529152959577.png)



总结来看的话，通过 `createRouter` 和 `createRouterMatcher` 函数， 实现了路由的添加、解析和导航。`createRouterMatcher` 负责管理路由匹配器，而 `createRouter` 则负责初始化路由实例和处理导航逻辑。最终，路由匹配器会根据路径或名称解析出匹配的路由记录，并执行相应的导航操作。



很显然，这套方案太麻烦了,不符合我们简单明了的宗旨，所以这边我取了个巧，用正则去处理这块，我的方案

```ts

    matchTargetUrl(path: string) {
        const routes = Array.from(this.routes.keys());
        for (const route of routes) {
            const regex = new RegExp(`^${this.convertToRegex(route)}$`);
            const match = path.match(regex);
            if (match) {
                return route;
            }
        }
        return '/404';
    }

    convertToRegex(route: string) {
        return route.replace(/:[^\s/]+/g, '([^\\s/]+)');
    }
```

我们来看看这个效果，看看我这段测试用例

```js
import Router from "../Router.ts";
import { describe, it, expect, beforeAll } from 'vitest';
const HELLO = { template: '<div>Hello World</div>' };

describe('Router', () => {
    let router;
  
    beforeAll(() => {
      const routes = [
        { path: '/home', component: HELLO },
        { path: '/users/:userId', component: HELLO },
        { path: '/products/:productId', component: HELLO },
        { path: '/categories/:categoryId/products/:productId', component: HELLO },
        { path: '/search/:query?', component: HELLO },
        { path: '/departments/:departmentId/employees/:employeeId', component: HELLO }
      ];
      router = new Router({ routes: routes, mode: 'history' });
    });
  
    it('should match basic route', () => {
      expect(router.matchTargetUrl('/home')).toBe('/home');
    });
  
    it('should match route with userId parameter', () => {
      expect(router.matchTargetUrl('/users/123')).toBe('/users/:userId');
    });
  
    it('should match route with productId parameter', () => {
      expect(router.matchTargetUrl('/products/456')).toBe('/products/:productId');
    });
  
    it('should match route with categoryId and productId parameters', () => {
      expect(router.matchTargetUrl('/categories/789/products/123')).toBe('/categories/:categoryId/products/:productId');
    });
  
    it('should match route with optional query parameter', () => {
      expect(router.matchTargetUrl('/search/something')).toBe('/search/:query?');
    });
  
    it('should return 404 for unmatched path', () => {
      expect(router.matchTargetUrl('/contact')).toBe('/404');
    });
  
    it('should match route with departmentId and employeeId parameters', () => {
      expect(router.matchTargetUrl('/departments/001/employees/002')).toBe('/departments/:departmentId/employees/:employeeId');
    });
});
```

运行的结果

![image-20240529153324534](https://cdn.jsdelivr.net/gh/knightgao/public-img-oss/img/image-20240529153324534.png)

完全没问题嘛，😊



#### 路由守卫

路由守卫这块其实比较简单，很多同学可能是没接触过所以觉得很神奇，直接上代码

```ts
beforeEach(hook: (from: string, to: string) => Promise<void>) {
        this.beforeGuards.push(hook);
    }
 async handleRouteChange(path: string) {
        const from = this.currentRoute.value;
        const to = path;

        try {
            for (const hook of this.beforeGuards) await hook(from, to);
            for (const hook of this.beforeResolveGuards) await hook(from, to);

            this.currentRoute.value = path;
            this.currentComponent.value = this.matchRouteComponent(path);

            for (const hook of this.afterGuards) await hook(from, to);

            this?.readyResolve();
        } catch (error) {
            for (const handler of this.errorHandlers) handler(error);
        }
    }
```

这样即可

### 总结

如果看完整的源码太长了，希望这个简单版本的路由可以让你了解核心，感兴趣的去阅读源码，水平有限，若有错漏，欢迎支持。

### 完整代码

```ts
import { type App, shallowRef } from 'vue';
import RouterLink from './components/RouterLink.vue';
import RouterView from './components/RouterView.vue';

class Router {
    currentRoute = shallowRef<string>('');
    routes = new Map<string, any>();
    currentComponent = shallowRef<any>(null);
    mode: string;
    beforeGuards: any[] = [];
    beforeResolveGuards: any[] = [];
    afterGuards: any[] = [];
    errorHandlers: any[] = [];
    readyPromise: Promise<void>;
    readyResolve!: () => void;

    constructor({ mode = 'history', routes = [] }: { mode?: string, routes?: any[] } = {}) {
        this.mode = mode;
        this.readyPromise = new Promise((resolve) => {
            this.readyResolve = resolve;
        });
        this.init();
        this.addRoutes(routes);
    }

    addRoutes(routes: any[]) {
        routes.forEach(route => {
            this.addRoute(route.path, route.component);
        });
    }

    addRoute(path: string, component: any) {
        if (typeof component !== 'object') {
            throw new Error('component is not a Vue component');
        }
        this.routes.set(path, component);
    }

    init() {
        if (this.mode === 'history') {
            window.addEventListener('popstate', async () => {
                await this.onRouteChange();
            });
        } else if (this.mode === 'hash') {
            window.addEventListener('hashchange', async () => {
                await this.onRouteChange();
            });
        }
        this.onRouteChange(); // Initialize the first route
    }

    

    beforeEach(hook: (from: string, to: string) => Promise<void>) {
        this.beforeGuards.push(hook);
    }

    beforeResolve(hook: (from: string, to: string) => Promise<void>) {
        this.beforeResolveGuards.push(hook);
    }

    afterEach(hook: (from: string, to: string) => Promise<void>) {
        this.afterGuards.push(hook);
    }

    onError(handler: (error: Error) => void) {
        this.errorHandlers.push(handler);
    }

    isReady(): Promise<void> {
        return this.readyPromise;
    }

    

    async push(path: string) {
        if (this.mode === 'history') {
            window.history.pushState({}, '', path);
        } else if (this.mode === 'hash') {
            window.location.hash = path || '/';
        }
        await this.onRouteChange();
    }
    
    async replace(path: string) {
        if (this.mode === 'history') {
            window.history.replaceState({}, '', path);
        } else if (this.mode === 'hash') {
            const newHash = '#' + (path || '/');
            window.location.replace(window.location.pathname + window.location.search + newHash);
        }
        await this.onRouteChange();
    }

    back() {
        window.history.back();
    }

    forward() {
        window.history.forward();
    }

    go(delta: number) {
        window.history.go(delta);
    }

    async onRouteChange() {
        const path = this.getCurrentPath();
        const targetPath = this.matchTargetUrl(path);

        await this.handleRouteChange(targetPath);
    }

    async handleRouteChange(path: string) {
        const from = this.currentRoute.value;
        const to = path;

        try {
            for (const hook of this.beforeGuards) await hook(from, to);
            for (const hook of this.beforeResolveGuards) await hook(from, to);

            this.currentRoute.value = path;
            this.currentComponent.value = this.matchRouteComponent(path);

            for (const hook of this.afterGuards) await hook(from, to);

            this?.readyResolve();
        } catch (error) {
            for (const handler of this.errorHandlers) handler(error);
        }
    }

    matchTargetUrl(path: string) {
        const routes = Array.from(this.routes.keys());
        for (const route of routes) {
            const regex = new RegExp(`^${this.convertToRegex(route)}$`);
            const match = path.match(regex);
            if (match) {
                return route;
            }
        }
        return '/404';
    }

    convertToRegex(route: string) {
        return route.replace(/:[^\s/]+/g, '([^\\s/]+)');
    }

    matchRouteComponent(path: string) {
        return this.routes.get(path) || null;
    }

    getCurrentPath(): string {
        if (this.mode === 'history') {
            const path = window.location.pathname;
            return path === '' || path === '/' ? '/' : path;
        } else if (this.mode === 'hash') {
            const hash = window.location.hash.slice(1);
            return hash === '' ? '/' : hash;
        }
        return '/';
    }

    getCurrentComponent() {
        return this.currentComponent;
    }

    install(app: App) {
        app.config.globalProperties.$router = this;
        app.provide('router', this);
        app.component('RouterLink', RouterLink);
        app.component('RouterView', RouterView);
        this.onRouteChange(); // Initialize the first route
    }
}

export default Router;
```



对应的测试文件

```js
import Router from "../Router.ts";
import { describe, it, expect, beforeEach, afterEach, vi } from 'vitest';

// 模拟一个 Vue 组件
const HELLO = { template: '<div>Hello World</div>' };

describe('路由初始化', () => {
    let addEventListenerSpy;
    let originalPathname;
    let originalHash;

    beforeEach(() => {
        originalPathname = window.location.pathname;
        originalHash = window.location.hash;
        addEventListenerSpy = vi.spyOn(window, 'addEventListener');
    });

    afterEach(() => {
        window.history.pushState({}, '', originalPathname);
        window.location.hash = originalHash;
        addEventListenerSpy.mockRestore();
    });

    it('路由构造函数默认值', () => {
        const router = new Router();
        expect(router.mode).toBe('history');
        expect(router.routes.size).toBe(0);
    });

    it('路由构造函数自定义值', () => {
        const routes = [{ path: '/home', component: HELLO }];
        const router = new Router({ mode: 'hash', routes });
        expect(router.mode).toBe('hash');
        expect(router.routes.size).toBe(1);
    });

    it('处理路由变化时调用错误处理程序', async () => {
        const router = new Router();
        const errorHandler = vi.fn();
        const error = new Error('测试错误');

        router.onError(errorHandler);
        router.beforeEach(() => { throw error; });

        await router.handleRouteChange('/home');

        expect(errorHandler).toHaveBeenCalledWith(error);
    });

    it('readyResolve 在路由变化后应只调用一次', async () => {
        const router = new Router();
        const readyResolveSpy = vi.spyOn(router, 'readyResolve');

        router.addRoute('/home', HELLO);
        await router.push('/home');

        expect(readyResolveSpy).toHaveBeenCalledTimes(1);
    });

    it('convertToRegex 应该正确转换动态路由', () => {
        const router = new Router();
        const regex = router.convertToRegex('/home/:id');
        expect(regex).toBe('/home/([^\\s/]+)');
    });

    it('getCurrentPath 在不同模式下应返回正确路径', () => {
        const routerHistory = new Router({ mode: 'history' });
        window.history.pushState({}, '', '/home');
        expect(routerHistory.getCurrentPath()).toBe('/home');

        const routerHash = new Router({ mode: 'hash' });
        window.location.hash = '#/home';
        expect(routerHash.getCurrentPath()).toBe('/home');
    });

    it('install 应该将路由器注入 Vue 应用', () => {
        const app = { config: { globalProperties: {} }, provide: vi.fn(),component: vi.fn()};
        const router = new Router();
        router.install(app);
        expect(app.config.globalProperties.$router).toBe(router);
        expect(app.provide).toHaveBeenCalledWith('router', router);
    });

    it('init 方法应设置事件监听器并调用 onRouteChange', async () => {
        const routerHistory = new Router({ mode: 'history' });
        expect(addEventListenerSpy).toHaveBeenCalledWith('popstate', expect.any(Function));

        const routerHash = new Router({ mode: 'hash' });
        expect(addEventListenerSpy).toHaveBeenCalledWith('hashchange', expect.any(Function));

        const onRouteChangeSpyHistory = vi.spyOn(routerHistory, 'onRouteChange');
        const onRouteChangeSpyHash = vi.spyOn(routerHash, 'onRouteChange');
        await routerHistory.init();
        await routerHash.init();
        expect(onRouteChangeSpyHistory).toHaveBeenCalled();
        expect(onRouteChangeSpyHash).toHaveBeenCalled();
    });

    it('matchRouteComponent 对于未匹配的路径应返回 null', () => {
        const router = new Router();
        expect(router.matchRouteComponent('/unknown')).toBeNull();
    });

    it('在路由变化时 currentComponent 应该更新', async () => {
        const router = new Router();
        router.addRoute('/home', HELLO);
        await router.push('/home');
        expect(router.currentComponent.value).toEqual(HELLO);
    });

    it('isReady 应该在路由变化后解析', async () => {
        const router = new Router();
        const readyPromise = router.isReady();
        await router.push('/home');
        await expect(readyPromise).resolves.toBeUndefined();
    });

    it('测试 navigation 方法（go, back, forward）', () => {
        const router = new Router();
        const goSpy = vi.spyOn(window.history, 'go');
        const backSpy = vi.spyOn(window.history, 'back');
        const forwardSpy = vi.spyOn(window.history, 'forward');

        router.go(1);
        expect(goSpy).toHaveBeenCalledWith(1);

        router.back();
        expect(backSpy).toHaveBeenCalled();

        router.forward();
        expect(forwardSpy).toHaveBeenCalled();
    });

    it('router.addRoute 应该成功添加路由', () => {
        const router = new Router();
        router.addRoute('/home', HELLO);
        expect(router.routes).toEqual(new Map([['/home', HELLO]]));
    });

    it('router.addRoute 接受的不是个组件', () => {
        const router = new Router();
        expect(() => {
            router.addRoute('/home', 'HELLO');
        }).toThrowError('component is not a Vue component');
    });

    it('router.push 和 replace 方法应更新路径并调用 onRouteChange', async () => {
        const router = new Router();
        router.addRoute('/home', HELLO);
        const onRouteChangeSpy = vi.spyOn(router, 'onRouteChange');

        await router.push('/home');
        expect(router.getCurrentPath()).toBe('/home');
        expect(onRouteChangeSpy).toHaveBeenCalled();

        await router.replace('/home');
        expect(router.getCurrentPath()).toBe('/home');
        expect(onRouteChangeSpy).toHaveBeenCalled();
    });

    it('router.replace 成功', async () => {
        const router = new Router();
        router.addRoute('/home', HELLO);
        await router.replace('/home');
        expect(router.getCurrentComponent().value).toEqual(HELLO);
    });

    it('钩子执行顺序应该是 beforeEach -> beforeResolve -> afterEach', async () => {
        const router = new Router();
        const callOrder = [];

        router.beforeEach(() => { callOrder.push('beforeEach'); return Promise.resolve(); });
        router.beforeResolve(() => { callOrder.push('beforeResolve'); return Promise.resolve(); });
        router.afterEach(() => { callOrder.push('afterEach'); return Promise.resolve(); });

        router.addRoute('/home', HELLO);
        await router.push('/home');

        expect(callOrder).toEqual(['beforeEach', 'beforeResolve', 'afterEach']);
    });

    it('onRouteChange 应该正确处理当前路径和未匹配路径', async () => {
        const router = new Router();
        router.addRoute('/home', HELLO);

        window.history.pushState({}, '', '/home');
        await router.onRouteChange();
        expect(router.currentRoute.value).toBe('/home');
        expect(router.currentComponent.value).toBe(HELLO);

        window.history.pushState({}, '', '/unknown');
        await router.onRouteChange();
        expect(router.currentRoute.value).toBe('/404');
    });

    it('getCurrentPath 方法在未定义模式下应返回根路径', () => {
        const router = new Router({ mode: 'invalid' });
        expect(router.getCurrentPath()).toBe('/');
    });

    it('push 和 replace 方法在 hash 模式下应更新 hash 并调用 onRouteChange', async () => {
        const router = new Router({ mode: 'hash' });
        router.addRoute('/home', HELLO);
        const onRouteChangeSpy = vi.spyOn(router, 'onRouteChange');

        await router.push('/home');
        expect(window.location.hash).toBe('#/home');
        expect(onRouteChangeSpy).toHaveBeenCalled();

        await router.replace('/home');
        expect(window.location.hash).toBe('#/home');
        expect(onRouteChangeSpy).toHaveBeenCalled();
    });

    it('push 和 replace 方法在 hash 模式下应正确处理空 hash', async () => {
        const router = new Router({ mode: 'hash' });
        const onRouteChangeSpy = vi.spyOn(router, 'onRouteChange');

        await router.push('');
        expect(window.location.hash).toBe('#/');
        expect(onRouteChangeSpy).toHaveBeenCalled();

        await router.replace('');
        expect(window.location.hash).toBe('#/');
        expect(onRouteChangeSpy).toHaveBeenCalled();
    });

    it('构造函数应该调用 addRoutes 方法', () => {
        const addRoutesSpy = vi.spyOn(Router.prototype, 'addRoutes');
        new Router({ routes: [{ path: '/home', component: HELLO }] });
        expect(addRoutesSpy).toHaveBeenCalled();
        addRoutesSpy.mockRestore();
    });

    it('调用 addRoutes 方法时应该添加所有路由', () => {
        const router = new Router();
        const routes = [{ path: '/home', component: HELLO }, { path: '/about', component: HELLO }];
        router.addRoutes(routes);
        expect(router.routes.size).toBe(2);
        expect(router.routes.get('/home')).toBe(HELLO);
        expect(router.routes.get('/about')).toBe(HELLO);
    });

    it('readyResolve 在 handleRouteChange 方法中应被调用', async () => {
        const router = new Router();
        const readyResolveSpy = vi.spyOn(router, 'readyResolve');

        router.addRoute('/home', HELLO);
        await router.push('/home');

        expect(readyResolveSpy).toHaveBeenCalled();
    });

    it('测试 beforeResolve 钩子', async () => {
        const router = new Router();
        const beforeResolveHook = vi.fn().mockResolvedValue(undefined);

        router.addRoute('/home', HELLO);
        router.beforeResolve(beforeResolveHook);

        await router.push('/home');

        expect(beforeResolveHook).toHaveBeenCalled();
    });

    it('测试 onError 钩子', async () => {
        const router = new Router();
        const errorHook = vi.fn();

        router.addRoute('/home', HELLO);
        router.onError(errorHook);

        const error = new Error('测试错误');
        router.beforeEach(() => {
            throw error;
        });

        await router.push('/home');

        expect(errorHook).toHaveBeenCalledWith(error);
    });
});
```

```js
import Router from "../Router.ts";
import { describe, it, expect, beforeAll } from 'vitest';
// 模拟一个 Vue 组件
const HELLO = { template: '<div>Hello World</div>' };

describe('Router', () => {
    let router;
  
    beforeAll(() => {
      const routes = [
        { path: '/home', component: HELLO },
        { path: '/users/:userId', component: HELLO },
        { path: '/products/:productId', component: HELLO },
        { path: '/categories/:categoryId/products/:productId', component: HELLO },
        { path: '/search/:query?', component: HELLO },
        { path: '/departments/:departmentId/employees/:employeeId', component: HELLO }
      ];
      router = new Router({ routes: routes, mode: 'history' });
    });
  
    it('should match basic route', () => {
      expect(router.matchTargetUrl('/home')).toBe('/home');
    });
  
    it('should match route with userId parameter', () => {
      expect(router.matchTargetUrl('/users/123')).toBe('/users/:userId');
    });
  
    it('should match route with productId parameter', () => {
      expect(router.matchTargetUrl('/products/456')).toBe('/products/:productId');
    });
  
    it('should match route with categoryId and productId parameters', () => {
      expect(router.matchTargetUrl('/categories/789/products/123')).toBe('/categories/:categoryId/products/:productId');
    });
  
    it('should match route with optional query parameter', () => {
      expect(router.matchTargetUrl('/search/something')).toBe('/search/:query?');
    });
  
    it('should return 404 for unmatched path', () => {
      expect(router.matchTargetUrl('/contact')).toBe('/404');
    });
  
    it('should match route with departmentId and employeeId parameters', () => {
      expect(router.matchTargetUrl('/departments/001/employees/002')).toBe('/departments/:departmentId/employees/:employeeId');
    });
});
```

git 仓库地址 https://github.com/knightgao/tiny-router
