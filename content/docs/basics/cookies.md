---
summary: 了解如何在 AdonisJS 中读取、写入和清除 cookie。
---

# Cookies

在 HTTP 请求期间，请求 cookie 会自动被解析。你可以使用 [request](./request.md) 对象读取 cookie，并使用 [response](./response.md) 对象设置/清除 cookie。

```ts
// title: 读取 cookie
import router from '@adonisjs/core/services/router'

router.get('cart', async ({ request }) => {
  // highlight-start
  const cartItems = request.cookie('cart_items', [])
  // highlight-end
  console.log(cartItems)
})
```

```ts
// title: 设置 cookie
import router from '@adonisjs/core/services/router'

router.post('cart', async ({ request, response }) => {
  const id = request.input('product_id')
  // highlight-start
  response.cookie('cart_items', [{ id }])
  // highlight-end
})
```

```ts
// title: 清除 cookie
import router from '@adonisjs/core/services/router'

router.delete('cart', async ({ request, response }) => {
  // highlight-start
  response.clearCookie('cart_items')
  // highlight-end
})
```

## 配置

设置 cookie 的默认配置在 `config/app.ts` 文件中定义。你可以根据应用程序的需求调整这些选项。

```ts
http: {
  cookie: {
    domain: '',
    path: '/',
    maxAge: '2h',
    httpOnly: true,
    secure: true,
    sameSite: 'lax',
    /**
     * 实验性属性
     * https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#partitioned
     */
    partitioned: false,
    priority: 'medium',
  }
}
```

这些选项会被转换为 [set-cookie 属性](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#attributes)。`maxAge` 属性接受基于字符串的时间表达式，并且其值会被转换为秒。

在设置 cookie 时，可以覆盖同一组选项。

```ts
response.cookie('key', value, {
  domain: '',
  path: '/',
  maxAge: '2h',
  httpOnly: true,
  secure: true,
  sameSite: 'lax',
})
```

## 支持的数据类型

cookie 值使用 `JSON.stringify` 序列化为字符串；因此，你可以将以下 JavaScript 数据类型用作 cookie 值。

- string
- number
- bigInt
- boolean
- null
- object
- array 

```ts
// Object
response.cookie('user', {
  id: 1,
  fullName: 'virk',
})

// Array
response.cookie('product_ids', [1, 2, 3, 4])

// Boolean
response.cookie('is_logged_in', true)

// Number
response.cookie('visits', 10)

// BigInt
response.cookie('visits', BigInt(10))

// Date 对象会被转换为 ISO 字符串
response.cookie('visits', new Date())
```

## 签名 cookie

使用 `response.cookie` 方法设置的 cookie 是签名的。签名 cookie 防篡改，意味着更改其内容将使签名无效，并且 cookie 将被忽略。

cookie 使用 `config/app.ts` 文件中定义的 `appKey` 进行签名。

:::note

签名 cookie 仍然可以通过 Base64 解码来读取。如果你希望 cookie 值不可读，可以使用加密 cookie。

:::

```ts
import router from '@adonisjs/core/services/router'

router.get('/', async ({ request, response }) => {
  // 设置签名 cookie
  response.cookie('user_id', 1)

  // 读取签名 cookie
  request.cookie('user_id')
})
```

## 加密 cookie

与签名 cookie 不同，加密 cookie 的值无法解码为明文。因此，你可以使用加密 cookie 来存储包含敏感信息的值，这些值不应被任何人读取。

加密 cookie 使用 `response.encryptedCookie` 方法设置，并使用 `request.encryptedCookie` 方法读取。在底层，这些方法使用 [Encryption 模块](../security/encryption.md)。

```ts
import router from '@adonisjs/core/services/router'

router.get('/', async ({ request, response }) => {
  // 设置加密 cookie
  response.encryptedCookie('user_id', 1)

  // 读取加密 cookie
  request.encryptedCookie('user_id')
})
```

## 普通 cookie

当你希望前端代码使用 `document.cookie` 值访问 cookie 时，可以使用普通 cookie。

默认情况下，我们会对原始值调用 `JSON.stringify` 并将其转换为 base64 编码字符串。这样做是为了使你能够使用 `objects` 和 `arrays` 作为 cookie 值。然而，可以关闭编码。

普通 cookie 使用 `response.plainCookie` 方法定义，并使用 `request.plainCookie` 方法读取。

```ts
import router from '@adonisjs/core/services/router'

router.get('/', async ({ request, response }) => {
  // 设置普通 cookie
  response.plainCookie('user', { id: 1 }, {
    httpOnly: true
  })

  // 读取普通 cookie
  request.plainCookie('user')
})
```

要关闭编码，请在选项对象中设置 `encode: false`。

```ts
response.plainCookie('token', tokenValue, {
  httpOnly: true,
  encode: false,
})

// 读取关闭编码的普通 cookie
request.plainCookie('token', {
  encoded: false
})
```

## 测试期间设置 cookie

以下指南涵盖了编写测试时使用 cookie 的方法。

- 使用 [Japa API 客户端](../testing/http_tests.md#readingwriting-cookies) 定义 cookie。
- 使用 [Japa 浏览器客户端](../testing/browser_tests.md#readingwriting-cookies) 定义 cookie。