---
summary: 了解如何使用 AdonisJS 哈希服务对值进行哈希处理。
---

# Hashing（哈希处理）

你可以使用 `hash` 服务在应用程序中对用户密码进行哈希处理。AdonisJS 原生支持 `bcrypt`、`scrypt` 和 `argon2` 哈希算法，并且具备 [添加自定义驱动程序](#creating-a-custom-hash-driver) 的能力。

哈希值以 [PHC 字符串格式](https://github.com/P-H-C/phc-string-format/blob/master/phc-sf-spec.md) 存储。PHC 是一种用于格式化哈希的确定性编码规范。

## 用法

`hash.make` 方法接受一个纯字符串值（用户密码输入）并返回哈希输出。

```ts
import hash from '@adonisjs/core/services/hash'

const hash = await hash.make('user_password')
// $scrypt$n=16384,r=8,p=1$iILKD1gVSx6bqualYqyLBQ$DNzIISdmTQS6sFdQ1tJ3UCZ7Uun4uGHNjj0x8FHOqB0pf2LYsu9Xaj5MFhHg21qBz8l5q/oxpeV+ZkgTAj+OzQ
```

你 [无法将哈希值转换回纯文本](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html#hashing-vs-encryption)，哈希处理是一个单向过程，一旦生成哈希，就无法检索原始值。

然而，哈希处理提供了一种验证给定纯文本值是否与现有哈希匹配的方法，你可以使用 `hash.verify` 方法执行此检查。

```ts
import hash from '@adonisjs/core/services/hash'

if (await hash.verify(existingHash, plainTextValue)) {
  // 密码正确
}
```

## 配置

哈希处理的配置存储在 `config/hash.ts` 文件中。默认驱动程序设置为 `scrypt`，因为 scrypt 使用 Node.js 原生加密模块，不需要任何第三方包。

```ts
// title: config/hash.ts
import { defineConfig, drivers } from '@adonisjs/core/hash'

export default defineConfig({
  default: 'scrypt',

  list: {
    scrypt: drivers.scrypt(),

    /**
     * 使用 argon2 时取消注释
       argon: drivers.argon2(),
     */

    /**
     * 使用 bcrypt 时取消注释
       bcrypt: drivers.bcrypt(),
     */
  }
})
```

### Argon

Argon 是推荐用于对用户密码进行哈希处理的算法。要使用 AdonisJS 哈希服务的 argon，你必须安装 [argon2](https://npmjs.com/argon2) npm 包。

```sh
npm i argon2
```

我们为 argon 驱动程序配置了安全的默认值，但你可以根据应用程序需求调整配置选项。以下是可用选项的列表。

```ts
export default defineConfig({
  // highlight-start
  // 确保将默认驱动程序更新为 argon
  default: 'argon',
  // highlight-end

  list: {
    argon: drivers.argon2({
      version: 0x13, // 19 的十六进制代码
      variant: 'id',
      iterations: 3,
      memory: 65536,
      parallelism: 4,
      saltSize: 16,
      hashLength: 32,
    })
  }
})
```

<dl>

<dt>

variant（变体）

</dt>

<dd>

要使用的 argon 哈希变体。

- `d` 更快，对 GPU 攻击具有高度抵抗力，适用于加密货币。
- `i` 较慢，对权衡攻击具有抵抗力，更适用于密码哈希和密钥派生。
- `id`*（默认）* 是上述两者的混合组合，对 GPU 和权衡攻击具有抵抗力。

</dd>

<dt>

version（版本）

</dt>

<dd>

要使用的 argon 版本。可选选项为 `0x10 (1.0)` 和 `0x13 (1.3)`。默认情况下应使用最新版本。

</dd>

<dt>

iterations（迭代次数）

</dt>

<dd>

`iterations` 计数增加了哈希强度，但计算所需时间更长。

默认值为 `3`。

</dd>

<dt>

memory（内存）

</dt>

<dd>

用于哈希处理值的内存量。每个并行线程将具有此大小的内存池。

默认值为 `65536 (KiB)`。

</dd>

<dt>

parallelism（并行性）

</dt>

<dd>

用于计算哈希的线程数。

默认值为 `4`。

</dd>

<dt>

saltSize（盐大小）

</dt>

<dd>

盐的长度（以字节为单位）。在计算哈希时，argon 会生成此大小的加密安全随机盐。

密码哈希的默认和推荐值为 `16`。

</dd>

<dt>

hashLength（哈希长度）

</dt>

<dd>

原始哈希的最大长度（以字节为单位）。输出值将比指定的哈希长度更长，因为原始哈希输出会进一步编码为 PHC 格式。

默认值为 `32`

</dd>

</dl>

### Bcrypt

要使用 AdonisJS 哈希服务的 Bcrypt，你必须安装 [bcrypt](http://npmjs.com/bcrypt) npm 包。

```sh
npm i bcrypt
```

以下是可用配置选项的列表。

```ts
export default defineConfig({
  // highlight-start
  // 确保将默认驱动程序更新为 bcrypt
  default: 'bcrypt',
  // highlight-end

  list: {
    bcrypt: drivers.bcrypt({
      rounds: 10,
      saltSize: 16,
      version: '2b'
    })
  }
})
```

<dl>

<dt>

rounds（轮次）

</dt>

<dd>

计算哈希的成本。建议阅读 Bcrypt 文档中的 [A Note on Rounds](https://github.com/kelektiv/node.bcrypt.js#a-note-on-rounds) 部分，以了解 `rounds` 值对计算哈希所需时间的影响。

默认值为 `10`。

</dd>

<dt>

saltSize（盐大小）

</dt>

<dd>

盐的长度（以字节为单位）。在计算哈希时，我们会生成此大小的加密安全随机盐。

默认值为 `16`。

</dd>

<dt>

version（版本）

</dt>

<dd>

哈希算法的版本。支持的值为 `2a` 和 `2b`。建议使用最新版本，即 `2b`。

</dd>

</dl>

### Scrypt

scrypt 驱动程序使用 Node.js 加密模块来计算密码哈希。配置选项与 [Node.js `scrypt` 方法](https://nodejs.org/dist/latest-v19.x/docs/api/crypto.html#cryptoscryptpassword-salt-keylen-options-callback) 所接受的选项相同。

```ts
export default defineConfig({
  // highlight-start
  // 确保将默认驱动程序更新为 scrypt
  default: 'scrypt',
  // highlight-end

  list: {
    scrypt: drivers.scrypt({
      cost: 16384,
      blockSize: 8,
      parallelization: 1,
      saltSize: 16,
      maxMemory: 33554432,
      keyLength: 64
    })
  }
})
```

## 使用模型钩子对密码进行哈希处理

由于你将使用 `hash` 服务对用户密码进行哈希处理，因此可能会发现将逻辑放置在 `beforeSave` 模型钩子中很有帮助。

:::note

如果你使用的是 `@adonisjs/auth` 模块，则无需在模型中哈希密码。`AuthFinder` 会自动处理密码哈希，确保你的用户凭据得到安全处理。有关此过程的更多信息，请[在此处](../authentication/verifying_user_credentials.md#hashing-user-password)了解。

:::

```ts
import { BaseModel, beforeSave } from '@adonisjs/lucid'
import hash from '@adonisjs/core/services/hash'

export default class User extends BaseModel {
  @beforeSave()
  static async hashPassword(user: User) {
    if (user.$dirty.password) {
      user.password = await hash.make(user.password)
    }
  }
}
```

## 切换驱动程序

如果你的应用程序使用多个哈希驱动程序，可以使用 `hash.use` 方法在它们之间切换。

`hash.use` 方法接受配置文件中的映射名称，并返回匹配驱动程序的实例。

```ts
import hash from '@adonisjs/core/services/hash'

// 使用配置文件中的 "list.scrypt" 映射
await hash.use('scrypt').make('secret')

// 使用配置文件中的 "list.bcrypt" 映射
await hash.use('bcrypt').make('secret')

// 使用配置文件中的 "list.argon" 映射
await hash.use('argon').make('secret')
```

## 检查密码是否需要重新哈希

建议使用最新的配置选项来确保密码的安全性，特别是在报告旧版本哈希算法存在漏洞时。

使用最新选项更新配置后，可以使用 `hash.needsReHash` 方法检查密码哈希是否使用了旧选项，并执行重新哈希。

检查必须在用户登录时执行，因为只有在此时才能访问明文密码。

```ts
import hash from '@adonisjs/core/services/hash'

if (await hash.needsReHash(user.password)) {
  user.password = await hash.make(plainTextPassword)
  await user.save()
}
```

如果使用模型钩子来计算哈希，可以为 `user.password` 分配一个明文值。

```ts
if (await hash.needsReHash(user.password)) {
  // 让模型钩子重新哈希密码
  user.password = plainTextPassword
  await user.save()
}
```

## 在测试中模拟哈希服务

哈希值通常是一个缓慢的过程，会使测试变慢。因此，你可以考虑使用 `hash.fake` 方法模拟哈希服务，以禁用密码哈希。

在以下示例中，我们使用 `UserFactory` 创建了 20 个用户。由于你使用模型钩子来哈希密码，这可能需要 5-7 秒（具体取决于配置）。

```ts
import hash from '@adonisjs/core/services/hash'

test('get users list', async ({ client }) => {
  await UserFactory().createMany(20)    
  const response = await client.get('users')
})
```

然而，一旦你模拟了哈希服务，相同的测试将以更快的数量级运行。

```ts
import hash from '@adonisjs/core/services/hash'

test('get users list', async ({ client }) => {
  // highlight-start
  hash.fake()
  // highlight-end
  
  await UserFactory().createMany(20)    
  const response = await client.get('users')

  // highlight-start
  hash.restore()
  // highlight-end
})
```

## 创建自定义哈希驱动程序

哈希驱动程序必须实现 [HashDriverContract](https://github.com/adonisjs/hash/blob/main/src/types.ts#L13) 接口。此外，官方哈希驱动程序使用 [PHC 格式](https://github.com/P-H-C/phc-string-format/blob/master/phc-sf-spec.md) 来序列化哈希输出以便存储。你可以查看现有驱动程序的实现，了解它们如何使用 [PHC 格式化程序](https://github.com/adonisjs/hash/blob/main/src/drivers/bcrypt.ts) 来生成和验证哈希。

```ts
import {
  HashDriverContract,
  ManagerDriverFactory
} from '@adonisjs/core/types/hash'

/**
 * 哈希驱动程序接受的配置
 */
export type PbkdfConfig = {
}

/**
 * 驱动程序实现
 */
export class Pbkdf2Driver implements HashDriverContract {
  constructor(public config: PbkdfConfig) {
  }

  /**
   * 检查哈希值是否按照哈希算法进行格式化。
   */
  isValidHash(value: string): boolean {
  }

  /**
   * 将原始值转换为哈希
   */
  async make(value: string): Promise<string> {
  }

  /**
   * 验证明文值是否与提供的哈希匹配
   */
  async verify(
    hashedValue: string,
    plainValue: string
  ): Promise<boolean> {
  }

  /**
   * 检查由于配置参数已更改，哈希是否需要重新哈希
   */
  needsReHash(value: string): boolean {
  }
}

/**
 * 工厂函数，用于在配置文件中引用驱动程序。
 */
export function pbkdf2Driver (config: PbkdfConfig): ManagerDriverFactory {
  return () => {
    return new Pbkdf2Driver(config)
  }
}
```

在上面的代码示例中，我们导出了以下值。

- `PbkdfConfig`：你希望接受的配置的 TypeScript 类型。

- `Pbkdf2Driver`：驱动程序的实现。它必须遵循 `HashDriverContract` 接口。

- `pbkdf2Driver`：最后，一个工厂函数，用于延迟创建驱动程序的实例。

### 使用驱动程序

实现完成后，可以使用 `pbkdf2Driver` 工厂函数在配置文件中引用驱动程序。

```ts
// title: config/hash.ts
import { defineConfig } from '@adonisjs/core/hash'
import { pbkdf2Driver } from 'my-custom-package'

export default defineConfig({
  list: {
    pbkdf2: pbkdf2Driver({
      // 配置在此处
    }),
  }
})
```
