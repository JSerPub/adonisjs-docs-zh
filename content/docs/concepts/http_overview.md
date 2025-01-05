---
summary: 了解 AdonisJS 如何启动 HTTP 服务器、处理传入请求以及 HTTP 层可用的模块。
---

# HTTP 概述

AdonisJS 主要是一个用于创建响应 HTTP 请求的应用程序的 Web 框架。在本指南中，我们将了解 AdonisJS 如何启动 HTTP 服务器、处理传入请求以及 HTTP 层可用的模块。

## HTTP 层

AdonisJS 应用程序中的 HTTP 层由以下模块组成。值得一提的是，AdonisJS 的 HTTP 层是从头开始构建的，并没有使用任何底层微框架。

<dl>

<dt>

[路由器](../basics/routing.md)

</dt>

<dd>

[路由器模块](https://github.com/adonisjs/http-server/blob/main/src/router/main.ts)负责定义应用程序的端点，这些端点被称为路由。一个路由应该定义一个处理请求的处理器，该处理器可以是一个闭包或是对控制器的引用。

</dd>

<dt>

[控制器](../basics/controllers.md)

</dt>

<dd>

控制器是 JavaScript 类，你可以将其绑定到路由以处理 HTTP 请求。控制器作为组织层，帮助你将应用程序的业务逻辑分布在不同的文件/类中。

</dd>

<dt>

[HttpContext](./http_context.md)

</dt>

<dd>

AdonisJS 会为每个传入的 HTTP 请求创建一个 [HttpContext](https://github.com/adonisjs/http-server/blob/main/src/http_context/main.ts) 类的实例。HttpContext（又称 `ctx`）携带了给定请求的信息，如请求体、头信息、认证用户等。

</dd>

<dt>

[中间件](../basics/middleware.md)

</dt>

<dd>

AdonisJS 中的中间件管道是[责任链](https://refactoring.guru/design-patterns/chain-of-responsibility)设计模式的实现。你可以使用中间件来拦截 HTTP 请求，并在它们到达路由处理器之前对它们进行响应。

</dd>

<dt>

[全局异常处理器](../basics/exception_handling.md)

</dt>

<dd>

全局异常处理器在一个中心位置处理 HTTP 请求期间抛出的异常。你可以使用全局异常处理器将异常转换为 HTTP 响应或报告给外部日志服务。

</dd>

<dt>

服务器

</dt>

<dd>

[服务器模块](https://github.com/adonisjs/http-server/blob/main/src/server/main.ts)将路由器、中间件、全局异常处理器连接起来，并导出[一个 `handle` 函数](https://github.com/adonisjs/http-server/blob/main/src/server/main.ts#L330)，你可以将其绑定到 Node.js HTTP 服务器以处理请求。

</dd>

</dl>

## AdonisJS 如何启动 HTTP 服务器
一旦你在 Server 类上调用[`boot` 方法](https://github.com/adonisjs/http-server/blob/main/src/server/main.ts#L252)，HTTP 服务器即被启动。在底层，该方法执行以下操作。

- 创建中间件管道
- 编译路由
- 导入并实例化全局异常处理器

在典型的 AdonisJS 应用程序中，`boot` 方法由 `bin/server.ts` 文件中的 [Ignitor](https://github.com/adonisjs/core/blob/main/src/ignitor/http.ts) 模块调用。

此外，必须在调用 `boot` 方法之前定义路由、中间件和全局异常处理器，AdonisJS 通过使用 `start/routes.ts` 和 `start/kernel.ts` [预加载文件](./adonisrc_file.md#preloads) 来实现这一点。

![server_boot_lifecycle](./server_boot_lifecycle.png)

## HTTP 请求生命周期
现在我们有了一个监听传入请求的 HTTP 服务器。让我们看看 AdonisJS 如何处理给定的 HTTP 请求。

:::note

**另请参阅**：\
[中间件执行流程](../basics/middleware.md#middleware-execution-flow)\
[中间件和异常处理](../basics/middleware.md#middleware-and-exception-handling)

:::

<dl>

<dt> 创建 HttpContext </dt>

<dd>

作为第一步，服务器模块创建 [HttpContext](./http_context.md) 类的实例，并将其作为引用传递给中间件、路由处理程序和全局异常处理器。

如果已启用 [AsyncLocalStorage](./async_local_storage.md#usage)，则相同的实例将作为本地存储状态共享。

</dd>

<dt> 执行服务器中间件堆栈 </dt>

<dd>

接下来，执行[服务器中间件堆栈](../basics/middleware.md#server-middleware-stack) 中的中间件。这些中间件可以在请求到达路由处理程序之前拦截并响应请求。

此外，即使你没有为给定端点定义任何路由器，每个 HTTP 请求也会经过服务器中间件堆栈。这允许服务器中间件在不依赖路由系统的情况下为应用程序添加功能。

</dd>

<dt> 查找匹配的路由 </dt>

<dd>

如果服务器中间件没有结束请求，我们将为 `req.url` 属性查找匹配的路由。如果没有匹配的路由，请求将以 `404 - 未找到` 异常中止。否则，我们将继续处理请求。

</dd>

<dt> 执行路由中间件 </dt>

<dd>

一旦找到匹配的路由，我们将执行[路由器全局中间件](../basics/middleware.md#router-middleware-stack) 和[命名中间件堆栈](../basics/middleware.md#named-middleware-collection)。同样，中间件可以在请求到达路由处理程序之前拦截请求。

</dd>

<dt> 执行路由处理程序 </dt>

<dd>

作为最后一步，请求到达路由处理程序并返回响应给客户端。

假设在此过程中任何一步引发了异常。在这种情况下，请求将被交给全局异常处理器，它负责将异常转换为响应。

</dd>

<dt> 序列化响应 </dt>

<dd>

一旦使用 `response.send` 方法或通过从路由处理程序返回值定义了响应体，我们就开始响应序列化过程并设置适当的头信息。

了解更多关于[响应体序列化](../basics/response.md#response-body-serialization) 的信息。

</dd>

</dl>