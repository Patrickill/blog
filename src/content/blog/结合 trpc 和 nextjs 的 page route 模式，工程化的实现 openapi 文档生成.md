\---

title: 工程化的实现 openapi 文档生成

date: 2024-05-28 9:17:12

tags: ['front']

summary: 结合 trpc 和 nextjs 的 page route 模式，工程化的实现 openapi 文档生成

\---

## 技术栈介绍

详细的介绍在这里不再做赘述，就谈谈我对这几个技术栈的理解

### trpc

它的出现是为了解决客户端与服务器端之间的类型安全通信，在它之前，我们想要在客户端中获得安全的类型定义，我们需要自行定义类型，虽然apifox支持一键导出功能但是接口多了难免显得繁琐和难以管理，它的出现就是使得我们在客户端调用api时能像调用本地函数那样方便和安全，保证了前后端的类型同一，但是需要注意的是只有在next或者nuxt这种全栈框架中才能使用这一特性

#### 如何使用

从项目结果出发大体描述一遍整个项目的运行逻辑

```
项目结构
├─ src
│  ├─ pages
│  │  ├─ api
│  │  │  ├─ openapi.json.ts // 提供 OpenAPI 文档的 API 端点
│  │  │  ├─ trpc
│  │  │  │  └─ [trpc].ts // 将 tRPC 路由和处理逻辑连接到 Next.js 的 API 路由上，处理 tRPC 的所有请求
│  │  │  └─ [trpc].ts // tRPC 路由转换为 OpenAPI 兼容的处理器，并处理传入的请求，作为 /api/[trpc] 路由的处理器
│  │  ├─ index.tsx // 主页面
│  │  ├─ login.tsx // 登录页面
│  │  └─ _app.tsx // 使用 trpc.withTRPC 高阶组件包裹根组件 MyApp，从而在整个应用中提供 tRPC 的支持
│  ├─ server
│  │  ├─ openapi.ts // 创建 OpenAPI 文档的逻辑
│  │  ├─ router
│  │  │  ├─ hello.ts // 具体的处理函数
│  │  │  ├─ login.ts
│  │  │  └─ _app.ts // 定义服务器端路由和处理函数
│  │  └─ trpc.ts
│  └─ utils
│     └─ trpc.ts // 创建 tRPC 客户端实例，供客户端中直接调用使用，可以设置 BaseUrl、批处理请求等配置
└─ tsconfig.json

```



#### nextjs

ext.js 是一个 React 框架，用于构建服务端渲染（SSR）和静态生成（SSG）的应用程序。它的 page route 模式是指

- 使用 `pages` 目录中的文件来自动创建路由。
- 每个文件自动映射到一个路由。

而任何放在 `src/pages/api` 文件夹下的文件都会自动成为一个 API 路由端点例如，`src/pages/api/hello.ts` 文件会被映射为 `/api/hello` 端点。



## 实现方案

我主要使用了trpc-openapi 自动生成openapi文档，使用swagger-ui生成具体的api文档展示，使用typescript脚本生成具体配置文件，并使用redocly/cli根据typescript脚本生成的openapi.json生成具体的html文档资源,最后编写shell脚本和配置package.json脚本工程化使得每次run dev和run build时自动更新和生成openapi文档，从而实现工程化自动生成

步骤如下

#### 1. 集成 tRPC 和 OpenAPI

首先，我们使用 tRPC 定义服务器端的 API 路由和处理函数。然后，利用 `trpc-openapi` 将 tRPC 路由转换为 OpenAPI 规范的文档。

#### 2. 生成和展示 API 文档

使用 `swagger-ui-react` 集成 Swagger UI，我们可以生成一个交互式的 API 文档页面，可以直接在项目中访问和查看这个文档。

在 `src/pages/api/openapi.json.ts` 中设置一个 API 端点，用于提供生成的 OpenAPI 文档。通过访问这个端点，Swagger UI 可以加载并展示最新的 API 文档。

#### 3. 配置 TypeScript 脚本

编写 TypeScript 脚本，用于生成 OpenAPI 文档的具体配置文件。在 `.\scripts\generate-openapi.ts` 中定义生成 OpenAPI 文档的逻辑。这些脚本可以自动化生成openapi文档的json文件

#### 4. 使用 redocly/cli

`redocly/cli` 是一个强大的工具，可以将 OpenAPI 文档转换为静态 HTML 文档。我们使用它根据生成的 `openapi.json` 文件生成具体的 HTML 文档资源，方便发布和分享 API 文档。

#### 5. 编写 Shell 脚本和配置 package.json

为了实现工程化自动生成 OpenAPI 文档，我们编写 Shell 脚本，将生成文档的过程集成到项目的构建和开发流程中。在 `package.json` 中配置脚本，使得每次运行 `run dev` 或 `run build` 时，都会执行生成 OpenAPI 文档的脚本。