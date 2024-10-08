---
title: Vue 3 SSR 全站框架指南
---

# Vue 3 SSR 全站框架指南

<vue-metadata author="swwind" time="2020-12-5" tags="Vue,javascript"></vue-metadata>

距离 Vue 3 的发布已经过去很久了，本人由于奇妙的大一年度计划被迫重新走上了前端的开发之路。个人感觉，Vue 3 相较于 Vue 2 确实改进了许多东西。其中的 Composition API 大大简化了代码的分散程度，以及 Async Component 更是对异步请求页面的服务端渲染提供了完美的支持。

由于本人折腾的时间过长，因此决定有必要总结一下目前个人将 Vue 3 网站全面实现服务端渲染（SSR）的解决方案。

**注意：本文已经过时**

## 安装依赖

`@vue/server-renderer` 是专门为 Vue 3 量身定制的 SSR 代码库，但是令人惶恐的是他只提供了一个方法 `renderToString`，并且没有提供任何文档。

与此同时在网上能找到的 Vue SSR 文档都是关于 `vue-server-renderer` 这个只能支持 Vue 2 的包的使用指南。

所以我们只能自己折腾了，不过基本的思路还是跟 Vue 2 的 SSR 类似。

## 渲染成字符串

`@vue/server-renderer` 中提供了 `renderToString` 方法，其中第一个参数接受一个用 `Vue.createSSRApp` 来创建的实例。该方法会将实例渲染成 html 代码，但是仅仅只是包括 html 代码，所以别的什么东西还是要靠自己解决。

```javascript
const App = {
  data() {
    return { msg: "hello" };
  },
  template: `<div>{{ msg }}</div>`,
};

const app = Vue.createSSRApp(App);
const html = await renderToString(app); // "<div>hello</div>"
```

这是最基本的使用方法，同时他也可以渲染 Async Component，但是需要套在一个 `<Suspense>` 里面。`<Suspense>` 是专门用来渲染 Async Component 的一个东西，直接用就可以了。

```js
const AsyncComponent = {
  template: `<div>{{ msg }}</div>`,
  async setup() {
    await new Promise((s) => setTimeout(s, 1000));
    // 这里可以发送一些异步的请求并处理结果
    return {
      msg: "Elaina is my waifu!",
    };
  },
};

// 把异步控件套在 Suspense 里面
const App = {
  template: `<Suspense><AsyncComponent/></Suspense>`,
  components: { AsyncComponent },
};

const app = Vue.createSSRApp(App);
const html = await renderToString(app); // "<div>Elaina is my waifu!</div>"
```

## 结合 vue-router

把 `<Suspense>` 结合到 `<router-view>` 里面需要用一些奇怪的写法，并且可以顺便处理一下页面加载失败的情况。

```html
<router-view v-slot="{ Component }">
  <suspense>
    <template #default>
      <component :is="Component" />
    </template>
    <template #fallback>
      <div>Loading...</div>
    </template>
  </suspense>
</router-view>
```

如此这样，我们便可以直接在 vue-router 当中使用 Async Component 了。

```js
const router = createRouter({
  routes: [
    {
      path: "/async",
      component: AsyncComponent,
    },
  ],
});
```

值得注意的是，这里似乎有一个奇怪的特性，如果你想让 vue-router 把 params 当成 props 传入 Async Component 的话必须手动指定 props 的类型。否则你将会得不到任何 props。

```js
const RegionView = {
  template: `
	<div>Region: {{ region }}</div>
	<div>Post: {{ post }}</div>
  `,
  props: {
    region: {
      type: String,
      required: true,
    },
    post: {
      type: String,
      required: true,
    },
  },
  async setup(props) {
    const { region, post } = Vue.toRefs(props);

    // region.value => string
    // post.value => string

    // 如果你不添加 props 属性
    // region => undefined
    // post => undefined
  },
};

const router = createRouter({
  routes: [
    {
      path: "/r/:region/:post",
      component: AsyncComponent,
      props: true, // 把 params 当成 props 传入
    },
  ],
});
```

## 渲染标题和其他 meta 标签

`vue-router` 可以给路由添加一些 meta 属性，这些属性可以在服务端渲染的时候被使用。

```js
const router = createRouter({
  // ...
  routes: [
    {
      path: "/async",
      component: AsyncComponent,
      meta: {
        title: "Async Page",
      },
    },
  ],
});

router.push("/async");
await router.isReady();

router.currentRoute.value.meta; // { title: 'Async Page' }
```

但是很多时候我们发现，这些属性是需要根据页面内容动态更新的。因此个人感觉 `vue-router` 的 meta 属性不适合处理这些信息。

那怎么办呢，我们可以使用 Vuex 的状态来模拟这些 meta 属性，甚至是 HTTP 响应码。

```js
// @/store/ssr.js

export default {
  // 注意状态一定要用函数形式
  state: () => ({
    title: "",
    status: 200,
  }),
  mutations: {
    [MutationTypes.HTTP_STATUS](state, payload) {
      state.status = paylaod;
    },
    [MutationTypes.SET_TITLE](state, payload) {
      state.title = paylaod;
    },
  },
};
```

这样我们就可以在 `setup` 函数里面游刃有余地控制这些 meta 属性了。

```js
import { useStore } from "vuex";

export default {
  async setup() {
    const store = useStore();
    // fetch some data
    const result = await fetch("https://example.com");
    if (result.status === 200) {
      store.commit(MutationTypes.HTTP_STATUS, 200);
    } else {
      store.commit(MutationTypes.HTTP_STATUS, 404);
    }
    // ...
  },
};
```

同理还可以添加更多的 meta 属性提供给 SSR。

## 渲染状态

服务端渲染的时候我们可以将当前的状态保存，并且在客户端加载的时候直接使用。

```js
// 服务端代码
const store = createStore({ ... });
app.use(store);
// ...
const html = await renderToString(app);
// ...
const stateStr = JSON.stringify(store.state);

// 将 stateStr 插入到 html 中
responseHTML = responseHTML.replace('</head>', `<script>window.__INITIAL_STATE__=${stateStr};</script></head>`);
```

```js
// 客户端代码
if (window.__INITIAL_STATE__) {
  store.replaceState(window.__INITIAL_STATE__);
}
```

## 一些小细节

当我们访问服务器的时候可能带有一些例如 Cookie 的属性，而服务端渲染的时候默认不会使用这些 Cookie，导致渲染出来的页面可能和客户端渲染出来的页面不同。因此我们需要在服务端渲染的时候添加上这些特殊的身份识别属性。

```js
axios.defaults.headers.common["Cookie"] = ctx.get("Cookie");
```

但是这样会有一些问题，不同的请求可能同时到达服务器，直接设置 axios 的全局默认值可能导致 Cookie 的覆盖，从而导致很严重的安全问题。

这个问题有两种解决方案，一种是对于每一个实例都单独新建一个 axios 的对象，还有一种就是使用一种类似于同步锁之类的东西。

这里给出同步锁的示例代码

```js
let locked = false;
const pendings = [];
const updateQueue = () => {
  if (locked) {
    return;
  }
  const top = pendings.shift();
  if (!top) {
    return;
  }
  locked = true;
  top();
};

export const lock = () => {
  return new Promise((resolve) => {
    pendings.push(resolve);
    updateQueue();
  });
};

export const unlock = () => {
  locked = false;
  updateQueue();
};

// 使用时
await lock(); // 上锁
// 设置 axios 全局 cookie
// 发送所有请求
const html = await renderToString(app);
// 清空 axios 的全局 cookie
unlock(); // 解锁
```

这个同步锁可以保证 `await lock()` 和 `unlock()` 之间的代码同一时间只能有一个线程在执行，因此可以保证不会产生请求的冲突。

## 打包

我们需要打两份包：一份给客户端用，另外一份给服务端 SSR 用。

由于折腾 webpack 过于复杂，这里直接偷懒用 `vue-cli-service build`。需要添加一份 `vue.config.js` 来区别对待这两份构建。

```js
const nodeExternals = require("webpack-node-externals");
const path = require("path");

exports.chainWebpack = (webpackConfig) => {
  if (process.env.SSR) {
    webpackConfig.entry("app").clear().add("./src/main.server.js");
    webpackConfig.target("node");
    webpackConfig.output.libraryTarget("commonjs2");
    webpackConfig.externals(
      nodeExternals({ allowlist: [/\.(css|vue)$/, /@babel/] }),
    );
    webpackConfig.optimization.splitChunks(false).minimize(false);
    webpackConfig.plugins.delete("hmr");
    webpackConfig.plugins.delete("preload");
    webpackConfig.plugins.delete("prefetch");
    webpackConfig.plugins.delete("progress");
    webpackConfig.plugins.delete("friendly-errors");
  } else {
    // 使用 cdn 加载这三份依赖
    webpackConfig.externals({
      vue: "Vue",
      vuex: "Vuex",
      "vue-router": "VueRouter",
    });
  }
};

exports.productionSourceMap = false;

if (process.env.SSR) {
  exports.filenameHashing = false;
  exports.outputDir = path.resolve(__dirname, "build", "ssr");
}
```

```bash
vue-cli-service build          # 构建客户端
SSR=true vue-cli-service build # 构建 SSR
```

如此，客户端的代码都将放入 `dist` 文件夹中，SSR 的代码都将放入 `build/ssr` 中。

可以直接使用 `require('./build/ssr/app.js')` 来获取 SSR 部分的代码，并且拿 `dist/index.html` 来当作服务端渲染的模板。

总体来说就是这样：

```js
const template = await fs.readFile("dist/index.html", "utf-8");
const gentemp = (title, meta, html, statestr) => {
  const metastr =
    `<title>${title}</title>` +
    Object.keys(meta)
      .map((key) => {
        return `<meta name="${key}" content="${meta[key]}">`;
      })
      .join("");

  return template
    .replace('<meta charset="utf-8">', '<meta charset="utf-8">' + metastr)
    .replace('<div id="app"></div>', `<div id="app">${html}</div>`)
    .replace(
      "</head>",
      `<script>window.__INITIAL_STATE__=${statestr};</script></head>`,
    );
};
```

至此 Vue 3 SSR 框架搭建完毕，并且能够正常工作（大概。

## 总结

这个解决方案可能目前还不是很优美，但 Vue 3 仍在开发阶段，期待以后能有更加简便的解决方案。

[end]
