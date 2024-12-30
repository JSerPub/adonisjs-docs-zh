---
summary: 学习如何使用 BodyParser 中间件解析请求体。
---

# BodyParser 中间件

请求数据使用在 `start/kernel.ts` 文件中注册的 `BodyParser` 中间件进行解析。

中间件的配置存储在 `config/bodyparser.ts` 文件中。在此文件中，您可以配置解析器以解析 **JSON 有效负载**、**带有文件上传的多部分表单** 和 **URL 编码的表单**。

另请参阅：[读取请求体](./request.md#request-body)\
另请参阅：[文件上传](./file_uploads.md)

```ts
import { defineConfig } from '@adonisjs/core/bodyparser'

export const defineConfig({
  allowedMethods: ['POST', 'PUT', 'PATCH', 'DELETE'],

  form: {
    // 解析 HTML 表单的设置
  },

  json: {
    // 解析 JSON 请求体的设置
  },

  multipart: {
    // 多部分解析器的设置
  },

  raw: {
    // 原始文本解析器的设置
  },
})
```

## 允许的方法

您可以定义一个 `allowedMethods` 数组，指定 `BodyParser` 中间件应尝试解析请求体的方法。默认情况下，配置了以下方法。但您可以随意删除或添加新方法。

```ts
{
  allowedMethods: ['POST', 'PUT', 'PATCH', 'DELETE']
}
```

## 将空字符串转换为 null

当输入字段没有值时，HTML 表单会在请求体中发送一个空字符串。HTML 表单的这种行为使得在数据库层进行数据规范化变得更加困难。

例如，如果您有一个可空的数据库列 `country`，当用户不选择国家时，您希望在该列中存储 `null` 作为值。

然而，使用 HTML 表单时，后端接收到的是一个空字符串，您可能会将空字符串插入到数据库中，而不是将该列留空为 `null`。

当配置中的 `convertEmptyStringsToNull` 标志启用时，`BodyParser` 中间件可以处理这种不一致性，将所有空字符串值转换为 `null`。

```ts
{
  form: {
    // ... 其他配置
    convertEmptyStringsToNull: true
  },

  json: {
    // ... 其他配置
    convertEmptyStringsToNull: true
  },

  multipart: {
    // ... 其他配置
    convertEmptyStringsToNull: true
  }
}
```

## JSON 解析器

JSON 解析器用于解析定义为 JSON 编码字符串的请求体，其 `Content-type` 头与预定义的 `types` 值之一匹配。

```ts
json: {
  encoding: 'utf-8',
  limit: '1mb',
  strict: true,
  types: [
    'application/json',
    'application/json-patch+json',
    'application/vnd.api+json',
    'application/csp-report',
  ],
  convertEmptyStringsToNull: true,
}
```

<dl>

<dt>

encoding

</dt>

<dd>

将请求体 Buffer 转换为字符串时使用的编码。最常用的是 `utf-8`。但是，您可以使用 [iconv-lite 包](https://www.npmjs.com/package/iconv-lite#readme) 支持的任何编码。

</dd>

<dt>

limit

</dt>

<dd>

解析器允许的最大请求体数据限制。如果请求体超过配置的限制，将返回 `413` 错误。

</dd>

<dt>

strict

</dt>

<dd>

严格解析仅允许在 JSON 编码字符串的顶层使用 `objects` 和 `arrays`。

</dd>

<dt>

types

</dt>

<dd>

`Content-type` 头应使用 JSON 解析器解析的值数组。

</dd>

</dl>

## URL 编码表单解析器

`form` 解析器用于解析 `Content-type` 头设置为 `application/x-www-form-urlencoded` 的 URL 编码字符串。换句话说，HTML 表单数据使用 `form` 解析器进行解析。

```ts
form: {
  encoding: 'utf-8',
  limit: '1mb',
  queryString: {},
  types: ['application/x-www-form-urlencoded'],
  convertEmptyStringsToNull: true,
}
```

<dl>

<dt>

encoding

</dt>

<dd>

将请求体 Buffer 转换为字符串时使用的编码。最常用的是 `utf-8`。但是，您可以使用 [iconv-lite 包](https://www.npmjs.com/package/iconv-lite#readme) 支持的任何编码。

</dd>

<dt>

limit

</dt>

<dd>

解析器允许的最大请求体数据限制。如果请求体超过配置的限制，将返回 `413` 错误。

</dd>

<dt>

queryString

</dt>

<dd>

使用 [qs 包](https://www.npmjs.com/package/qs) 解析 URL 编码的请求体。您可以使用 `queryString` 属性定义该包的选项。

```ts
  form: {
    queryString: {
      allowDots: true,
      allowSparse: true,
    },
  }
```

</dd>

</dl>

## 多部分解析器

`multipart` 解析器用于解析带有文件上传的 HTML 表单请求。

另请参阅：[文件上传](./file_uploads.md)

```ts
multipart: {
  autoProcess: true,
  processManually: [],
  encoding: 'utf-8',
  fieldsLimit: '2mb',
  limit: '20mb',
  types: ['multipart/form-data'],
  convertEmptyStringsToNull: true,
}
```

<dl>

<dt>

autoProcess

</dt>

<dd>

启用 `autoProcess` 会将所有用户上传的文件移动到操作系统的 `tmp` 目录。

之后，在控制器中，您可以验证文件并将其移动到持久位置或云服务。

如果禁用 `autoProcess` 标志，则需要手动处理流并从请求体中读取文件/字段。另请参阅：[自处理多部分流](./file_uploads.md#self-processing-multipart-stream)。

您可以定义一个路由数组，用于自动处理这些路由的文件。值 **必须是路由模式**，而不是 URL。

```ts
{
  autoProcess: [
    '/uploads',
    '/post/:id'
  ]
}
```

</dd>

<dt>

processManually

</dt>

<dd>

`processManually` 数组允许您为选定的路由关闭文件的自动处理。值 **必须是路由模式**，而不是 URL。

```ts
multipart: {
  autoProcess: true,
  processManually: [
    '/file_manager',
    '/projects/:id/assets'
  ],
}
```

</dd>

<dt>

encoding

</dt>

<dd>

将请求体 Buffer 转换为字符串时使用的编码。最常用的是 `utf-8`。但是，您可以使用 [iconv-lite 包](https://www.npmjs.com/package/iconv-lite#readme) 支持的任何编码。

</dd>

<dt>

limit

</dt>

<dd>

处理所有文件时允许的最大字节限制。您可以使用 [request.file](./file_uploads.md) 方法定义单个文件大小限制。

</dd>

<dt>

fieldsLimit

</dt>

<dd>

处理多部分请求时，允许字段（非文件）的最大字节限制。如果字段大小超过配置的限制，将返回 `413` 错误。

</dd>

</dl>