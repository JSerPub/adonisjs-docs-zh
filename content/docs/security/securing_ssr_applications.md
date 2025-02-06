---
summary: 了解如何使用 @adonisjs/shield 包保护你的服务器端渲染应用程序。
---

# 保护服务器端渲染应用程序

如果你正在使用 AdonisJS 创建服务器端渲染的应用程序，那么你必须使用 `@adonisjs/shield` 包来保护你的应用程序免受常见的网络攻击，如 **CSRF**、**XSS**、**内容嗅探**等。

该包在 **web 入门套件** 中预配置。但是，你可以按以下方式手动安装和配置该包。

:::note
`@adonisjs/shield` 包对 `@adonisjs/session` 包有同等依赖，因此请确保首先 [配置 session 包](../basics/session.md)。
:::

```sh
node ace add @adonisjs/shield
```

:::disclosure{title="查看 add 命令执行的步骤"}

1. 使用检测到的包管理器安装 `@adonisjs/shield` 包。

2. 在 `adonisrc.ts` 文件中注册以下服务提供者。

   ```ts
   {
     providers: [
       // ...其他提供者
       () => import('@adonisjs/shield/shield_provider'),
     ]
   }
   ```

3. 创建 `config/shield.ts` 文件。

4. 在 `start/kernel.ts` 文件中注册以下中间件。

   ```ts
   router.use([() => import('@adonisjs/shield/shield_middleware')])
   ```

:::

## CSRF 保护

[CSRF（跨站请求伪造）](https://owasp.org/www-community/attacks/csrf) 是一种攻击，恶意网站诱骗你的网页应用用户在没有明确同意的情况下执行表单提交。

为了防范 CSRF 攻击，你应该定义一个隐藏的输入字段来保存 CSRF 令牌值，该值只有你的网站可以生成和验证。因此，由恶意网站触发的表单提交将失败。

### 保护表单

一旦配置了 `@adonisjs/shield` 包，所有没有 CSRF 令牌的表单提交将自动失败。因此，你必须使用 `csrfField` edge 辅助函数来定义一个包含 CSRF 令牌的隐藏输入字段。

:::caption{for="info"}
**edge 助手**
:::

```edge
<form method="POST" action="/">
  // highlight-start
  {{ csrfField() }}
  // highlight-end
  <input type="name" name="name" placeholder="Enter your name">
  <button type="submit"> Submit </button>
</form>
```

:::caption{for="info"}
**输出的 HTML**
:::

```html

<form method="POST" action="/">
    // highlight-start
    <input type="hidden" name="_csrf" value="Q9ghWSf0-3FD9eCiu5YxvKaxLEZ6F_K4DL8o"/>
    // highlight-end
    <input type="name" name="name" placeholder="Enter your name"/>
    <button type="submit">Submit</button>
</form>
```

在表单提交期间，`shield_middleware` 将自动验证 `_csrf` 令牌，仅允许带有有效 CSRF 令牌的表单提交。

### 处理异常

当 CSRF 令牌缺失或无效时，Shield 会引发 `E_BAD_CSRF_TOKEN` 异常。默认情况下，AdonisJS 会捕获该异常并将用户重定向回表单，并附带错误闪存消息。

你可以在边缘模板中按以下方式访问闪存消息。

```edge
// highlight-start
@error('E_BAD_CSRF_TOKEN')
  <p> {{ $message }} </p>
@end
// highlight-end

<form method="POST" action="/">
  {{ csrfField() }}
  <input type="name" name="name" placeholder="Enter your name">
  <button type="submit"> Submit </button>
</form>
```

你还可以在 [全局异常处理程序](../basics/exception_handling.md#handling-exceptions) 中自行处理 `E_BAD_CSRF_TOKEN` 异常，如下所示。

```ts
import app from '@adonisjs/core/services/app'
import { errors } from '@adonisjs/shield'
import { HttpContext, ExceptionHandler } from '@adonisjs/core/http'

export default class HttpExceptionHandler extends ExceptionHandler {
  async handle(error: unknown, ctx: HttpContext) {
    // highlight-start
    if (error instanceof errors.E_BAD_CSRF_TOKEN) {
      return ctx.response
        .status(error.status)
        .send('Page has expired')
    }
    // highlight-end
    return super.handle(error, ctx)
  }
}
```

### 配置参考

CSRF 防护的配置存储在 `config/shield.ts` 文件中。

```ts
import { defineConfig } from '@adonisjs/shield'

const shieldConfig = defineConfig({
  csrf: {
    enabled: true,
    exceptRoutes: [],
    enableXsrfCookie: true,
    methods: ['POST', 'PUT', 'PATCH', 'DELETE'],
  },
})

export default shieldConfig
```

<dl>

<dt>

enabled

</dt>

<dd>

开启或关闭 CSRF 防护。

</dd>

<dt>

exceptRoutes

</dt>

<dd>

免于 CSRF 保护的路由模式数组。如果你的应用程序有通过 API 接受表单提交的路由，你可能希望免除它们。

对于更高级的使用场景，你可以注册一个函数来动态免除特定路由。

```ts
{
  exceptRoutes: (ctx) => {
    // 免除所有以 /api/ 开头的路由
    return ctx.request.url().includes('/api/')
  }
}
```

</dd>

<dt>

enableXsrfCookie

</dt>

<dd>

启用后，Shield 将 CSRF 令牌存储在一个名为 `XSRF-TOKEN` 的加密 cookie 中，前端 JavaScript 代码可以读取该 cookie。

这允许前端请求库（如 Axios）自动读取 `XSRF-TOKEN` 并在发起 Ajax 请求时将其作为 `X-XSRF-TOKEN` 标头设置，而无需服务器渲染的表单。

如果你不是通过编程方式触发 Ajax 请求，则必须禁用 `enableXsrfCookie`。

</dd>

<dt>

methods

</dt>

<dd>

要保护的 HTTP 方法数组。上述方法的所有传入请求都必须提供有效的 CSRF 令牌。

</dd>

<dt>

cookieOptions

</dt>

<dd>

`XSRF-TOKEN` cookie 的配置。[查看 cookie 配置](../basics/cookies.md#configuration) 以了解可用选项。

</dd>

</dl>

## 定义 CSP 策略

[CSP (Content security policy)](https://web.dev/csp/) 通过定义加载 JavaScript、CSS、字体、图像等的可信来源，保护你的应用程序免受 XSS 攻击。

CSP 防护默认是禁用的。然而，我们建议你启用它，并在 `config/shield.ts` 文件中配置策略指令。

```ts
import { defineConfig } from '@adonisjs/shield'

const shieldConfig = defineConfig({
  csp: {
    enabled: true,
    directives: {
      // 在此处添加策略指令
    },
    reportOnly: false,
  },
})

export default shieldConfig
```

<dl>

<dt>

enabled

</dt>

<dd>

开启或关闭 CSP 防护。

</dd>

<dt>

directives

</dt>

<dd>

配置 CSP 指令。你可以在 [https://content-security-policy.com/](https://content-security-policy.com/#directive) 查看可用指令的列表。

```ts
const shieldConfig = defineConfig({
  csp: {
    enabled: true,
    // highlight-start
    directives: {
      defaultSrc: [`'self'`],
      scriptSrc: [`'self'`, 'https://cdnjs.cloudflare.com'],
      fontSrc: [`'self'`, 'https://fonts.googleapis.com']
    },
    // highlight-end
    reportOnly: false,
  },
})

export default shieldConfig
```

</dd>

<dt>

reportOnly

</dt>

<dd>

当 `reportOnly` 标志启用时，CSP 策略不会阻止资源。相反，它会在使用 `reportUri` 指令配置的端点上报告违规行为。

```ts
const shieldConfig = defineConfig({
  csp: {
    enabled: true,
    directives: {
      defaultSrc: [`'self'`],
      // highlight-start
      reportUri: ['/csp-report']
      // highlight-end
    },
    // highlight-start
    reportOnly: true,
    // highlight-end
  },
})
```

同时，注册 `csp-report` 端点以收集违规报告。

```ts
router.post('/csp-report', async ({ request }) => {
  const report = request.input('csp-report')
})
```
</dd>

</dl>

### 使用 Nonce

你可以通过在 `script` 和 `style` 标签上定义 [nonce 属性](https://content-security-policy.com/nonce/) 来允许内联标签。在 Edge 模板中，可以使用 `cspNonce` 属性访问 nonce 属性的值。

```edge
<script nonce="{{ cspNonce }}">
  // 内联 JavaScript
</script>
<style nonce="{{ cspNonce }}">
  /* 内联 CSS */
</style>
```

同时，在指令配置中使用 `@nonce` 关键字以允许基于 nonce 的内联脚本和样式。

```ts
const shieldConfig = defineConfig({
  csp: {
    directives: {
      defaultSrc: [`'self'`, '@nonce'],
    },
  },
})
```

### 从 Vite Dev 服务器加载资源

如果你正在使用 [Vite 集成](../basics/vite.md)，可以使用以下 CSP 关键字来允许由 Vite Dev 服务器提供的资源。

- `@viteDevUrl` 将 Vite dev 服务器 URL 添加到允许列表中。
- `@viteHmrUrl` 将 Vite HMR websocket 服务器 URL 添加到允许列表中。

```ts
const shieldConfig = defineConfig({
  csp: {
    directives: {
      defaultSrc: [`'self'`, '@viteDevUrl'],
      connectSrc: ['@viteHmrUrl']
    },
  },
})
```

如果你将 Vite 打包的输出部署到 CDN 服务器，则必须用 `@viteUrl` 关键字替换 `@viteDevUrl`，以允许来自开发服务器和 CDN 服务器的资源。

```ts
directives: {
  // delete-start
  defaultSrc: [`'self'`, '@viteDevUrl'],
  // delete-end
  // insert-start
  defaultSrc: [`'self'`, '@viteUrl'],
  // insert-end
  connectSrc: ['@viteHmrUrl']
},
```

### 为 Vite 注入的样式添加 Nonce
目前，Vite 不允许为它在 DOM 中注入的 `style` 标签定义 `nonce` 属性。有一个 [开放的 PR](https://github.com/vitejs/vite/pull/11864) 关于此问题，我们希望它能尽快得到解决。

## 配置 HSTS
[**Strict-Transport-Security (HSTS)**](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security) 响应头指示浏览器始终使用 HTTPS 加载网站。

你可以使用 `config/shield.ts` 文件配置头指令。

```ts
import { defineConfig } from '@adonisjs/shield'

const shieldConfig = defineConfig({
  hsts: {
    enabled: true,
    maxAge: '180 days',
    includeSubDomains: true,
  },
})
```

<dl>

<dt>

enabled

</dt>

<dd>

开启或关闭 hsts 防护。

</dd>

<dt>

maxAge

</dt>

<dd>

定义 `max-age` 属性。值应为秒数的数字或基于字符串的时间表达式。

```ts
{
  // 记住 10 秒
  maxAge: 10,
}
```

```ts
{
  // 记住 2 天
  maxAge: '2 days',
}
```

</dd>

<dt>

includeSubDomains

</dt>

<dd>

定义 `includeSubDomains` 指令以将设置应用于子域。

</dd>

</dl>

## 配置 X-Frame 保护
[**X-Frame-Options**](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options) 头用于指示浏览器是否允许在 `iframe`、`frame`、`embed` 或 `object` 标签中嵌入网站。

:::note

如果你已配置 CSP，则可以使用 [frame-ancestors](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors) 指令，并禁用 `xFrame` 防护。

:::

你可以使用 `config/shield.ts` 文件配置头指令。

```ts
import { defineConfig } from '@adonisjs/shield'

const shieldConfig = defineConfig({
  xFrame: {
    enabled: true,
    action: 'DENY'
  },
})
```

<dl>

<dt>

enabled

</dt>

<dd>

开启或关闭 xFrame 防护。

</dd>

<dt>

action

</dt>

<dd>

`action` 属性定义头值。可以是 `DENY`、`SAMEORIGIN` 或 `ALLOW-FROM`。

```ts
{
  action: 'DENY'
}
```

在 `ALLOW-FROM` 的情况下，你还必须定义 `domain` 属性。

```ts
{
  action: 'ALLOW-FROM',
  domain: 'https://foo.com',
}
```

</dd>

</dl>

## 禁用 MIME 类型嗅探
[**X-Content-Type-Options**](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options) 头指示浏览器遵循 `content-type` 头，而不通过检查 HTTP 响应的内容来执行 MIME 类型嗅探。

一旦启用此防护，Shield 将为所有 HTTP 响应定义 `X-Content-Type-Options: nosniff` 头。

```ts
import { defineConfig } from '@adonisjs/shield'

const shieldConfig = defineConfig({
  contentTypeSniffing: {
    enabled: true,
  },
})
```
