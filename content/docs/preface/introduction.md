---
summary: "AdonisJS 是一个以 TypeScript 为优先选的 Node.js Web 框架。您可以使用它来创建全栈Web应用程序或 JSON API 服务器。"
---

# 介绍

::include{template="partials/introduction_cards"}

## AdonisJS 是什么?

AdonisJS `/əˈdəʊnɪs/JS` 是一个以 TypeScript 为优先选的 Node.js Web 框架。您可以使用它来创建全栈Web应用程序或 JSON API 服务器。

在基础层面，AdonisJS [为您的应用程序提供结构](../getting_started/folder_structure.md)，配置了[无缝的 TypeScript 开发环境](../concepts/typescript_build_process.md)，为后端代码配置了热模块替换（[HMR](../concepts/hmr.md)），并提供了一系列已维护且文档详尽的丰富包集合。

我们设想，使用 AdonisJS 的团队将**花费更少的时间**在琐碎的决定上，比如为每个小功能精挑细选npm包、编写粘合代码、争论完美的文件夹结构，而**将更多的时间**投入到实现满足业务需求的实际功能上。

### 前端无关

AdonisJS 专注于后端，并允许您选择自己喜欢的前端技术栈。

如果您喜欢保持简洁，可以将 `AdonisJS` 与[传统模板引擎](../views-and-templates/introduction.md)结合使用，在服务器上生成静态 `HTML`，为您的前端 `Vue/React` 应用创建一个 `JSON API`，或者使用 `Inertia` 让您喜欢的前端框架完美协同工作。

AdonisJS 旨在为您提供一套完整的工具，让您能够从头开始创建一个强大的后端应用。无论是发送电子邮件、验证用户输入、执行 CRUD 操作，还是用户身份验证，我们都已为您妥善处理。


### 现代和类型安全

AdonisJS 建立在现代 JavaScript 基础之上。我们使用 ES 模块、Node.js 子路径导入别名、SWC 来执行 TypeScript 源码，以及 Vite 来打包资源。

此外，在设计框架的 API 时，TypeScript 扮演了举足轻重的角色。例如，AdonisJS 具有：

- [类型安全的事件发射器](../digging_deeper/emitter.md#making-events-type-safe)
- [类型安全的环境变量](../getting_started/environment_variables.md)
- [类型安全的验证库](../basics/validation.md)


### 拥抱 MVC

AdonisJS 采用了经典的 `MVC` 设计模式。您首先使用功能性 JavaScript API 定义路由，将控制器绑定到这些路由上，并在控制器内编写处理 HTTP 请求的逻辑。

```ts
// title: start/routes.ts
import router from '@adonisjs/core/services/router'
const PostsController = () => import('#controllers/posts_controller')

router.get('posts', [PostsController, 'index'])
```

控制器可以使用模型从数据库中获取数据，并呈现一个视图（也称为模板）作为响应。

```ts
// title: app/controllers/posts_controller.ts
import Post from '#models/post'
import type { HttpContext } from '@adonisjs/core/http'

export default class PostsController {
  async index({ view }: HttpContext) {
    const posts = await Post.all()
    return view.render('pages/posts/list', { posts })
  }
}
```

如果您正在构建 `API` 服务器，您可以用 `JSON` 响应来替换视图层。但是，处理和响应 `HTTP` 请求的流程保持不变。

```ts
// title: app/controllers/posts_controller.ts
import Post from '#models/post'
import type { HttpContext } from '@adonisjs/core/http'

export default class PostsController {
  async index({ view }: HttpContext) {
    const posts = await Post.all()
    // delete-start
    return view.render('pages/posts/list', { posts })
    // delete-end
    // insert-start
    /**
     * Posts array will be serialized to JSON
     * automatically.
     */
    return posts
    // insert-end
  }
}
```

## 指南设想

AdonisJS 的文档被编写为参考指南，涵盖了由核心团队维护的多个包和模块的使用方法和 API。

**本指南不会教你如何从头开始构建一个应用程序。**如果你正在寻找教程，我们建议你从 [Adocasts](https://adocasts.com/) 开始学习之旅。Tom（Adocasts 的创始人）制作了一些高质量的视频，帮助你迈出使用 AdonisJS 的第一步。

话虽如此，该文档仍广泛涵盖了现有模块的使用方法和框架的内部工作原理。

## 最近发布

以下是最近发布的版本列表。[点击这里](./releases.md)查看所有发布版本。

::include{template="partials/recent_releases"}

## 赞助商

::include{template="partials/sponsors"}
