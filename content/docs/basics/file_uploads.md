---
summary: 了解如何使用 AdonisJS 中的 `request.file` 方法处理用户上传的文件，并使用验证器对其进行验证。
---

# 文件上传

AdonisJS 对使用 `multipart/form-data` 内容类型发送的用户上传文件提供一流的支持。文件通过 [bodyparser 中间件](../basics/body_parser.md#multipart-parser) 自动处理，并保存在操作系统的 `tmp` 目录中。

之后，在控制器中，您可以访问这些文件，对其进行验证，并将其移动到持久位置或像 S3 这样的云存储服务。

## 访问用户上传的文件

您可以使用 `request.file` 方法访问用户上传的文件。该方法接受字段名称并返回 [MultipartFile](https://github.com/adonisjs/bodyparser/blob/main/src/multipart/file.ts) 的实例。

```ts
import { HttpContext } from '@adonisjs/core/http'

export default class UserAvatarsController {
  update({ request }: HttpContext) {
    // highlight-start
    const avatar = request.file('avatar')
    console.log(avatar)
    // highlight-end
  }
}
```

如果单个输入字段用于上传多个文件，您可以使用 `request.files` 方法访问它们。该方法接受字段名称并返回 `MultipartFile` 实例的数组。

```ts
import { HttpContext } from '@adonisjs/core/http'

export default class InvoicesController {
  update({ request }: HttpContext) {
    // highlight-start
    const invoiceDocuments = request.files('documents')
    
    for (let document of invoiceDocuments) {
      console.log(document)
    }
    // highlight-end
  }
}
```

## 手动验证文件

您可以使用 [validator](#using-validator) 验证文件，或通过 `request.file` 方法定义验证规则。

在以下示例中，我们将通过 `request.file` 方法内联定义验证规则，并使用 `file.errors` 属性访问验证错误。

```ts
const avatar = request.file('avatar', {
  size: '2mb',
  extnames: ['jpg', 'png', 'jpeg']
})

if (!avatar.isValid) {
  return response.badRequest({
    errors: avatar.errors
  })
}
```

当处理文件数组时，您可以遍历文件并检查一个或多个文件是否未通过验证。

提供给 `request.files` 方法的验证选项适用于所有文件。在以下示例中，我们期望每个文件都小于 `2mb`，并且必须具有允许的文件扩展名之一。

```ts
const invoiceDocuments = request.files('documents', {
  size: '2mb',
  extnames: ['jpg', 'png', 'pdf']
})

/**
 * 创建无效文件集合
 */
let invalidDocuments = invoiceDocuments.filter((document) => {
  return !document.isValid
})

if (invalidDocuments.length) {
  /**
   * 返回文件名及其旁边的错误
   */
  return response.badRequest({
    errors: invalidDocuments.map((document) => {
      name: document.clientName,
      errors: document.errors,
    })
  })
}
```

## 使用验证器验证文件

与手动验证文件（如前一节所示）不同，您可以使用 [validator](./validation.md) 将文件验证作为验证管道的一部分。使用验证器时，无需手动检查错误；验证管道会处理这些。

```ts
// app/validators/user_validator.ts
import vine from '@vinejs/vine'

export const updateAvatarValidator = vine.compile(
  vine.object({
    // highlight-start
    avatar: vine.file({
      size: '2mb',
      extnames: ['jpg', 'png', 'pdf']
    })
    // highlight-end
  })
)
```

```ts
import { HttpContext } from '@adonisjs/core/http'
import { updateAvatarValidator } from '#validators/user_validator'

export default class UserAvatarsController {
  async update({ request }: HttpContext) {
    // highlight-start
    const { avatar } = await request.validateUsing(
      updateAvatarValidator
    )
    // highlight-end
  }
}
```

可以使用 `vine.array` 类型验证文件数组。例如：

```ts
import vine from '@vinejs/vine'

export const createInvoiceValidator = vine.compile(
  vine.object({
    // highlight-start
    documents: vine.array(
      vine.file({
        size: '2mb',
        extnames: ['jpg', 'png', 'pdf']
      })
    )
    // highlight-end
  })
)
```

## 将文件移动到持久位置

默认情况下，用户上传的文件保存在操作系统的 `tmp` 目录中，并且可能会在计算机清理 `tmp` 目录时被删除。

因此，建议将文件存储在持久位置。您可以使用 `file.move` 在同一文件系统中移动文件。该方法接受要移动文件的目录的绝对路径。

```ts
import app from '@adonisjs/core/services/app'

const avatar = request.file('avatar', {
  size: '2mb',
  extnames: ['jpg', 'png', 'jpeg']
})

// highlight-start
/**
 * 将头像移动到 "storage/uploads" 目录
 */
await avatar.move(app.makePath('storage/uploads'))
// highlight-end
```

建议为移动的文件提供一个唯一的随机名称。为此，您可以使用 `cuid` 助手。

```ts
// highlight-start
import { cuid } from '@adonisjs/core/helpers'
// highlight-end
import app from '@adonisjs/core/services/app'

await avatar.move(app.makePath('storage/uploads'), {
  // highlight-start
  name: `${cuid()}.${avatar.extname}`
  // highlight-end
})
```

文件移动后，您可以将其名称存储在数据库中以便后续引用。

```ts
await avatar.move(app.makePath('uploads'))

/**
 * 虚拟代码，将文件名作为头像保存在
 * 用户模型中，并将其持久化到
 * 数据库。
 */
auth.user!.avatar = avatar.fileName!
await auth.user.save()
```

### 文件属性

以下是您可以在 [MultipartFile](https://github.com/adonisjs/bodyparser/blob/main/src/multipart/file.ts) 实例上访问的属性列表。

| 属性       | 描述                                                                                                         |
|------------|--------------------------------------------------------------------------------------------------------------|
| `fieldName`  | HTML 输入字段的名称。                                                                                        |
| `clientName` | 用户计算机上的文件名。                                                                                       |
| `size`       | 文件的大小（以字节为单位）。                                                                                 |
| `extname`    | 文件的扩展名。                                                                                               |
| `errors`     | 与给定文件关联的错误数组。                                                                                   |
| `type`       | 文件的 [MIME 类型](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types)。             |
| `subtype`    | 文件的 [MIME 子类型](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types)。           |
| `filePath`   | `move` 操作后文件的绝对路径。                                                                                |
| `fileName`   | `move` 操作后的文件名。                                                                                      |
| `tmpPath`    | `tmp` 目录中文件的绝对路径。                                                                                 |
| `meta`       | 与文件关联的元数据，作为键值对。默认情况下，该对象为空。                                                     |
| `validated`  | 布尔值，指示文件是否已验证。                                                                                 |
| `isValid`    | 布尔值，指示文件是否通过了验证规则。                                                                         |
| `hasErrors`  | 布尔值，指示一个或多个错误是否与给定文件关联。                                                               |

## 提供文件服务

如果您将用户上传的文件与应用程序代码保存在同一文件系统中，您可以通过创建路由并使用 [`response.download`](./response.md#downloading-files) 方法来提供文件服务。

```ts
import { sep, normalize } from 'node:path'
import app from '@adonisjs/core/services/app'
import router from '@adonisjs/core/services/router'

const PATH_TRAVERSAL_REGEX = /(?:^|[\\/])\.\.(?:[\\/]|$)/

router.get('/uploads/*', ({ request, response }) => {
  const filePath = request.param('*').join(sep)
  const normalizedPath = normalize(filePath)
  
  if (PATH_TRAVERSAL_REGEX.test(normalizedPath)) {
    return response.badRequest('Malformed path')
  }

  const absolutePath = app.makePath('uploads', normalizedPath)
  return response.download(absolutePath)
})
```

- 我们使用[通配符路由参数](./routing.md#wildcard-params)获取文件路径，并将数组转换为字符串。
- 接下来，我们使用 Node.js 的 path 模块对路径进行规范化。
- 使用 `PATH_TRAVERSAL_REGEX` 保护此路由免受[路径遍历](https://owasp.org/www-community/attacks/Path_Traversal)攻击。
- 最后，我们将 `normalizedPath` 转换为 `uploads` 目录中的绝对路径，并使用 `response.download` 方法提供文件服务。

## 使用 Drive 上传和提供文件服务

Drive 是 AdonisJS 核心团队创建的文件系统抽象。您可以使用 Drive 管理用户上传的文件，并将它们存储在本地文件系统中，或将其移动到像 S3 或 GCS 这样的云存储服务。

我们建议使用 Drive 而不是手动上传和提供文件服务。Drive 处理了许多安全问题，如路径遍历，并为多个存储提供商提供了统一的 API。

[了解更多关于 Drive 的信息](../digging_deeper/drive.md)

## 高级用法 - 自处理多部分流

您可以关闭多部分请求的自动处理，并针对高级用例自行处理流。打开 `config/bodyparser.ts` 文件，并更改以下选项之一以禁用自动处理。

```ts
{
  multipart: {
    /**
     * 如果希望对所有 HTTP 请求手动自处理多部分流，
     * 请将此设置为 false。
     */
    autoProcess: false
  }
}
```

```ts
{
  multipart: {
    /**
     * 定义要手动处理多部分流的路由模式数组。
     */
    processManually: ['/assets']
  }
}
```

禁用自动处理后，您可以使用 `request.multipart` 对象处理单个文件。

在下面的示例中，我们使用 Node.js 的 `stream.pipeline` 方法处理多部分可读流，并将其写入磁盘上的文件。但是，您也可以将此文件流式传输到一些外部服务，如 `s3`。

```ts
import { createWriteStream } from 'node:fs'
import app from '@adonisjs/core/services/app'
import { pipeline } from 'node:stream/promises'
import { HttpContext } from '@adonisjs/core/http'

export default class AssetsController {
  async store({ request }: HttpContext) {
    /**
     * 步骤 1：定义文件监听器
     */
    request.multipart.onFile('*', {}, async (part, reporter) => {
      part.pause()
      part.on('data', reporter)

      const filePath = app.makePath(part.file.clientName)
      await pipeline(part, createWriteStream(filePath))
      return { filePath }
    })

    /**
     * 步骤 2：处理流
     */
    await request.multipart.process()

    /**
     * 步骤 3：访问处理后的文件
     */
    return request.allFiles()
  }
}
```

- `multipart.onFile` 方法接受您希望处理的文件的输入字段名称。您可以使用通配符 `*` 处理所有文件。

- `onFile` 监听器接收 `part`（可读流）作为第一个参数，接收 `reporter` 函数作为第二个参数。

- `reporter` 函数用于跟踪流进度，以便 AdonisJS 可以在流处理完成后为您提供对处理后的字节、文件扩展名和其他元数据的访问。

- 最后，您可以从 `onFile` 监听器返回一个属性对象，这些属性将与您使用 `request.file` 或 `request.allFiles()` 方法访问的文件对象合并。

### 错误处理

您必须监听 `part` 对象上的 `error` 事件，并手动处理错误。通常，流读取器（可写流）会在内部监听此事件，并中止写入操作。

### 验证流部分

即使您手动处理多部分流，AdonisJS 也允许您验证流部分（即文件）。如果发生错误，将在 `part` 对象上发出 `error` 事件。

`multipart.onFile` 方法接受验证选项作为第二个参数。此外，请确保监听 `data` 事件，并将 `reporter` 方法绑定到它。否则，不会进行任何验证。

```ts
request.multipart.onFile('*', {
  size: '2mb',
  extnames: ['jpg', 'png', 'jpeg']
}, async (part, reporter) => {
  /**
   * 以下两行是执行流验证所必需的
   */
  part.pause()
  part.on('data', reporter)
})
```