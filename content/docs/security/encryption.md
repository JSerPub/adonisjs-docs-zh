---
summary: 使用加密服务在应用程序中加密和解密值。
---

# Encryption（加密）

使用加密服务，您可以在应用程序中加密和解密值。加密基于 [aes-256-cbc 算法](https://www.n-able.com/blog/aes-256-encryption-algorithm)，并且在最终输出中附加了一个完整性哈希（HMAC）以防止值被篡改。

`encryption` 服务使用存储在 `config/app.ts` 文件中的 `appKey` 作为密钥来加密值。

- 建议确保 `appKey` 的安全性，并通过 [环境变量](../getting_started/environment_variables.md) 将其注入到应用程序中。任何拥有此密钥的人都可以解密值。

- 密钥应至少为 16 个字符长，并具有加密安全的随机值。您可以使用 `node ace generate:key` 命令生成密钥。

- 如果您以后决定更改密钥，将无法解密现有值。这将导致现有的 cookie 和用户会话失效。

## 加密值

您可以使用 `encryption.encrypt` 方法加密值。该方法接受要加密的值和一个可选的时间段，在该时间段后认为值已过期。

```ts
import encryption from '@adonisjs/core/services/encryption'

const encrypted = encryption.encrypt('hello world')
```

定义一个时间段，在该时间段后值将被认为已过期且无法解密。

```ts
const encrypted = encryption.encrypt('hello world', '2 hours')
```

## 解密值

可以使用 `encryption.decrypt` 方法解密加密的值。该方法接受加密的值作为第一个参数。

```ts
import encryption from '@adonisjs/core/services/encryption'

encryption.decrypt(encryptedValue)
```

## 支持的数据类型

传递给 `encrypt` 方法的值使用 `JSON.stringify` 序列化为字符串。因此，您可以使用以下 JavaScript 数据类型：

- string（字符串）
- number（数字）
- bigInt（大整数）
- boolean（布尔值）
- null（空值）
- object（对象）
- array（数组）

```ts
import encryption from '@adonisjs/core/services/encryption'

// Object（对象）
encryption.encrypt({
  id: 1,
  fullName: 'virk',
})

// Array（数组）
encryption.encrypt([1, 2, 3, 4])

// Boolean（布尔值）
encryption.encrypt(true)

// Number（数字）
encryption.encrypt(10)

// BigInt（大整数）
encryption.encrypt(BigInt(10))

// Data objects are converted to ISO string（数据对象被转换为 ISO 字符串）
encryption.encrypt(new Date())
```

## 使用自定义密钥

您可以直接创建 [Encryption 类的实例](https://github.com/adonisjs/encryption/blob/main/src/encryption.ts) 来使用自定义密钥。

```ts
import { Encryption } from '@adonisjs/core/encryption'

const encryption = new Encryption({
  secret: 'alongrandomsecretkey',
})
```
