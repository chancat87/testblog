---
title: 基于 Deno 的全栈框架 Slow 简介
---

# 基于 Deno 的全栈框架 Slow 简介

<vue-metadata author="swwind" time="2023-6-16" outdate></vue-metadata>

Slow 是一款全新的全栈框架，主要基于 Deno 运行时。

## 项目结构说明

### /app 目录

| 文件/目录       | 说明             |
| --------------- | ---------------- |
| `/app/root.tsx` | 整个前端的根结构 |
| `/app/routes/`  | 存放路由文件     |

其他文件都不管，只要你 `import` 了就会打包进去。

## 路由

每个目录可以匹配一层请求的路径，可以使用 `()` 小括号来忽略该层路径的匹配，也可以使用 `[]` 方括号来匹配动态的路由参数。

如果需要匹配全部子路径，那么可以使用 `[...]` 目录。

| 文件                   | 匹配的路径       | 路由参数              |
| ---------------------- | ---------------- | --------------------- |
| `/index.tsx`           | `/`              | `{}`                  |
| `/foo/index.tsx`       | `/foo/`          | `{}`                  |
| `/(foo)/bar/index.tsx` | `/bar/`          | `{}`                  |
| `/foo/[bar]/index.tsx` | `/foo/anything/` | `{"bar":"anything"}`  |
| `/foo/[...]/index.tsx` | `/foo/bar/baz/`  | `{"$":["bar","baz"]}` |

上面的表格中使用 `index.tsx` 用来表示对应页面的前端代码文件，你可以使用其他后缀名，但是开头必须是 index。

下面是一个路由目录下支持的文件和使用例。

### index.tsx

用于存放前端代码，只需要使用 `export default` 即可。

例如对于 React 框架，你可以直接如下书写。

```jsx
// index.tsx
export default function Page() {
  return <h1>Page</h1>;
}
```

### loader.xxx.ts

用于存放获取数据方面的代码，可以提供给前端进行渲染。

名称前面的 xxx 可以自由定义，也可以没有，后缀也可以自选。

一个文件中只能使用一个 `default export`，你需要使用帮助函数 `defineLoader` 嵌套你的整个 loader 函数来获取前后端分离的支持。

```js
// loader.something.ts
import { defineLoader } from "stale";

export default defineLoader(async (req: Request) => {
  // ask database...
  return { anything: ["JSON serializable"] };
});
```

```jsx
// index.tsx
import useSomething from "./loader.something.ts";

export default function Render() {
  const something = useSomething();
  return <h1>{something.anything[0]}</h1>;
}
```

### action.xxx.ts

用于存放一般的 POST 请求接口，可以提供给前端作为响应。

用法和上面的 loader 一样。

```js
// action.create.ts
import { defineAction } from "stale";

export default defineAction(async (req: Request) => {
  const formData = await req.formData();
  const name = formData.get("name");
  const password = formData.get("password");
  // do something...
  return { ok: true };
});
```

使用的时候可能需要使用表单提交。

```jsx
// index.tsx
import { Form } from "stale";
import useCreateUser from "./action.create.ts";

export default function Render() {
  const createUser = useCreateUser();

  return (
    <Form action={createUser}>
      <input type="text" name="name" />
      <input type="password" name="password" />
      <button>submit</button>
    </Form>
  );
}
```

### layout.xxx.tsx

layout 文件用于表示嵌套的 UI 模型。

例如，如果你想要加载 `/app/routes/foo/index.tsx` 页面，那么整个页面的嵌套顺序可能是这样的：

- `/app/root.tsx`
- `/app/routes/layout.tsx`
- `/app/routes/foo/layout.tsx`
- `/app/routes/foo/index.tsx`

如果你要对某个单独的页面使用特殊的 layout 模板，那么你可以将后缀改成 `.xxx.tsx` 来确定。

例如，如果你想要加载 `/foo/index.custom.tsx`，那么整个页面的嵌套顺序可能是这样的：

- `/app/root.tsx`
- `/app/routes/layout.custom.tsx`
- `/app/routes/foo/layout.custom.tsx`
- `/app/routes/foo/index.custom.tsx`

注意如果以上列举的任何一个 layout 文件不存在，则不会嵌套在页面中。即对于指定 namespace 的 index 文件，不会加载任何默认 namespace 的 layout 文件，除非你显式创建了该 namespace 的 layout 文件并且 export 了默认的 layout 文件。

```jsx
// layout.tsx
export default function (props) {
  return <div>{props.children}</div>;
}
```

Layout 中可以正常使用 loader 和 action 的功能。

## 数据获取

上面已经简单介绍了 loader 和 action 的功能，下面是一些更加详细的例子，介绍如何进行错误处理。

```js
// /[name]/loader.ts
export default defineLoader(async (req, params) => {
  const animeName = params.name;

  const anime = await kv.get(["anime", animeName]);
  // 用 throw 来抛出异常，默认会返回 500 请求
  if (!anime.value) {
    throw new Error("Not found!");
  }
  // 用 throw URL 来重定向网页
  if (!anime.value) {
    throw new URL("/path", req.url);
  }
  // 用 throw Response 来抛出可处理的异常
  if (!anime.value) {
    throw new Response("Not found!", { status: 404 });
  }

  return { anime: ["oshi no ko"] };
});
```

在其他页面里面也可以引用这个 loader 的数据，但是必须是该目录下面的子页面。

```jsx
// /[name]/very/deep/nested/index.ts
import useLoaderData from "../../../loader.ts";

export default function () {
  const loaderData = useLoaderData();
  return <div>{loaderData.anime[0]}</div>;
}
```

## 其他事情

没有！
