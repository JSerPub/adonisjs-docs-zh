---
summary: AdonisJS 中渲染视图和模板的可用选项
---

# 视图和模板

AdonisJS 非常适合在 Node.js 中创建传统的服务器端渲染应用程序。如果你喜欢使用后端模板引擎来输出 HTML，而无需虚拟 DOM 和构建工具的任何开销，那么本指南非常适合你。

在 AdonisJS 中，服务器端渲染应用程序的典型工作流程如下：

- 选择一个模板引擎来动态渲染 HTML。
- 使用 [Vite](../basics/vite.md) 来捆绑 CSS 和前端 JavaScript。
- 可选地，你可以选择像 [HTMX](https://htmx.org/) 或 [Unpoly](https://unpoly.com/) 这样的库来逐步增强你的应用程序，并实现像 SPA 一样的导航。

:::note
AdonisJS 核心团队创建了一个与框架无关的模板引擎，称为 [Edge.js](https://edgejs.dev)，但不强制你使用它。你可以在 AdonisJS 应用程序中使用任何你喜欢的其他模板引擎。
:::

## 流行选项

以下是你可以在 AdonisJS 应用程序中使用的流行模板引擎列表（就像在其他 Node.js 应用程序中一样）。

- [**EdgeJS**](https://edgejs.dev) 是一个简单、现代且功能齐全的模板引擎，由 AdonisJS 核心团队为 Node.js 创建和维护。
- [**Pug**](https://pugjs.org) 是一个受 Haml 影响的模板引擎。
- [**Nunjucks**](https://mozilla.github.io/nunjucks) 是一个功能丰富的模板引擎，受 Jinja2 的启发。

## 混合应用程序

AdonisJS 也非常适合创建混合应用程序，这些应用程序在服务器端渲染 HTML，然后在客户端对 JavaScript 进行水合。这种方法在开发人员中很受欢迎，他们希望使用 `Vue`、`React`、`Svelte`、`Solid` 或其他库来构建交互式用户界面，但仍然需要一个完整的后端堆栈来处理服务器端问题。

在这种情况下，AdonisJS 提供对 [InertiaJS](./inertia.md) 的一流支持，以弥合前端和后端之间的差距。
