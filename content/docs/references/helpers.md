---
summary: AdonisJS 将其实用工具打包到 `helpers` 模块中，并使它们可在你的应用程序代码中使用。
---

# 实用工具参考

AdonisJS 将其实用工具打包到 `helpers` 模块中，并使它们可在你的应用程序代码中使用。由于这些实用工具已由框架安装和使用，因此 `helpers` 模块不会给你的 `node_modules` 增加任何额外的负担。

实用工具方法从以下模块中导出。

```ts
import is from '@adonisjs/core/helpers/is'
import * as helpers from '@adonisjs/core/helpers'
import string from '@adonisjs/core/helpers/string'
```

## escapeHTML

对字符串值中的 HTML 实体进行转义。在底层，我们使用 [he](https://www.npmjs.com/package/he#heescapetext) 包。

```ts
import string from '@adonisjs/core/helpers/string'

string.escapeHTML('<p> foo © bar </p>')
// &lt;p&gt; foo © bar &lt;/p&gt;
```

可选地，你可以使用 `encodeSymbols` 选项对非 ASCII 符号进行编码。

```ts
import string from '@adonisjs/core/helpers/string'

string.escapeHTML('<p> foo © bar </p>', {
  encodeSymbols: true,
})
// &lt;p&gt; foo &#xA9; bar &lt;/p&gt;
```

## encodeSymbols

你可以使用 `encodeSymbols` 实用工具对字符串值中的非 ASCII 符号进行编码。在底层，我们使用 [he.encode](https://www.npmjs.com/package/he#heencodetext-options) 方法。

```ts
import string from '@adonisjs/core/helpers/string'

string.encodeSymbols('foo © bar ≠ baz 𝌆 qux')
// 'foo &#xA9; bar &#x2260; baz &#x1D306; qux'
```

## prettyHrTime

美化打印 [process.hrtime](https://nodejs.org/api/process.html#processhrtimetime) 方法的差值。

```ts
import { hrtime } from 'node:process'
import string from '@adonisjs/core/helpers/string'

const startTime = hrtime()
await someOperation()
const endTime = hrtime(startTime)

console.log(string.prettyHrTime(endTime))
```

## isEmpty

检查字符串值是否为空。

```ts
import string from '@adonisjs/core/helpers/string'

string.isEmpty('') // true
string.isEmpty('      ') // true
```

## truncate

在给定字符数处截断字符串。

```ts
import string from '@adonisjs/core/helpers/string'

string.truncate('This is a very long, maybe not that long title', 12)
// 输出: This is a ve...
```

默认情况下，字符串在给定索引处被精确截断。但是，你可以指示该方法等待单词完成。

```ts
string.truncate('This is a very long, maybe not that long title', 12, {
  completeWords: true,
})
// 输出: This is a very...
```

你可以使用 `suffix` 选项自定义后缀。

```ts
string.truncate('This is a very long, maybe not that long title', 12, {
  completeWords: true,
  suffix: '... <a href="/1"> Read more </a>',
})
// 输出: This is a very... <a href="/1"> Read more </a>
```

## excerpt

`excerpt` 方法与 `truncate` 方法相同。但是，它会从字符串中去除 HTML 标签。

```ts
import string from '@adonisjs/core/helpers/string'

string.excerpt('<p>This is a <strong>very long</strong>, maybe not that long title</p>', 12, {
  completeWords: true,
})
// 输出: This is a very...
```

## slug

为字符串值生成 slug。该方法从 [slugify package](https://www.npmjs.com/package/slugify) 中导出；因此，请参阅其文档以了解可用选项。

```ts
import string from '@adonisjs/core/helpers/string'

console.log(string.slug('hello ♥ world'))
// hello-love-world
```

你可以如下为 Unicode 值添加自定义替换。

```ts
string.slug.extend({ '☢': 'radioactive' })

console.log(string.slug('unicode ♥ is ☢'))
// unicode-love-is-radioactive
```

## interpolate

在字符串中插入变量。变量必须位于双大括号内。

```ts
import string from '@adonisjs/core/helpers/string'

string.interpolate('hello {{ user.username }}', {
  user: {
    username: 'virk'
  }
})
// hello virk
```

可以使用 `\\` 前缀转义大括号。

```ts
string.interpolate('hello \\{{ users.0 }}', {})
// hello {{ users.0 }}
```

## plural

将单词转换为复数形式。该方法从 [pluralize package](https://www.npmjs.com/package/pluralize) 中导出。

```ts
import string from '@adonisjs/core/helpers/string'

string.plural('test')
// tests
```

## isPlural

判断单词是否已经是复数形式。

```ts
import string from '@adonisjs/core/helpers/string'

string.isPlural('tests') // true
```

## pluralize

此方法结合了 `singular` 和 `plural` 方法，并根据数量使用其中之一。例如：

```ts
import string from '@adonisjs/core/helpers/string'

string.pluralize('box', 1) // box
string.pluralize('box', 2) // boxes
string.pluralize('box', 0) // boxes

string.pluralize('boxes', 1) // box
string.pluralize('boxes', 2) // boxes
string.pluralize('boxes', 0) // boxes
```

`pluralize` 属性导出 [additional methods](https://www.npmjs.com/package/pluralize) 以注册自定义的不可数、不规则、复数和单数规则。

```ts
import string from '@adonisjs/core/helpers/string'

string.pluralize.addUncountableRule('paper')
string.pluralize.addSingularRule(/singles$/i, 'singular')
```

## singular

将单词转换为单数形式。该方法从 [pluralize package](https://www.npmjs.com/package/pluralize) 中导出。

```ts
import string from '@adonisjs/core/helpers/string'

string.singular('tests')
// test
```

## isSingular

判断单词是否已经是单数形式。

```ts
import string from '@adonisjs/core/helpers/string'

string.isSingular('test') // true
```

## camelCase

将字符串值转换为驼峰式命名。

```ts
import string from '@adonisjs/core/helpers/string'

string.camelCase('user_name') // userName
```

以下是一些转换示例。

| 输入            | 输出        |
| ---------------- | ------------- |
| 'test'           | 'test'        |
| 'test string'    | 'testString'  |
| 'Test String'    | 'testString'  |
| 'TestV2'         | 'testV2'      |
| '_foo_bar_'      | 'fooBar'      |
| 'version 1.2.10' | 'version1210' |
| 'version 1.21.0' | 'version1210' |

## capitalCase

将字符串值转换为首字母大写形式。

```ts
import string from '@adonisjs/core/helpers/string'

string.capitalCase('helloWorld') // Hello World
```

以下是一些转换示例。

| 输入             | 输出           |
| ---------------- | -------------- |
| 'test'           | 'test'         |
| 'test string'    | 'test-string'  |
| 'Test String'    | 'test-string'  |
| 'Test V2'        | 'test-v2'      |
| 'TestV2'         | 'test-v-2'     |
| 'version 1.2.10' | 'version-1210' |
| 'version 1.21.0' | 'version-1210' |

## dotCase

将字符串值转换为点式命名法。

```ts
import string from '@adonisjs/core/helpers/string'

string.dotCase('helloWorld') // hello.World
```

可选地，你可以将所有单词的首字母转换为小写。

```ts
string.dotCase('helloWorld', { lowerCase: true }) // hello.world
```

以下是一些转换示例。

| Input            | Output         |
| ---------------- | -------------- |
| 'test'           | 'test'         |
| 'test string'    | 'test.string'  |
| 'Test String'    | 'Test.String'  |
| 'dot.case'       | 'dot.case'     |
| 'path/case'      | 'path.case'    |
| 'TestV2'         | 'Test.V.2'     |
| 'version 1.2.10' | 'version.1210' |
| 'version 1.21.0' | 'version.1210' |

## noCase

从字符串值中移除所有形式的大小写。

```ts
import string from '@adonisjs/core/helpers/string'

string.noCase('helloWorld') // hello world
```

以下是一些转换示例。

| Input                  | Output                 |
| ---------------------- | ---------------------- |
| 'test'                 | 'test'                 |
| 'TEST'                 | 'test'                 |
| 'testString'           | 'test string'          |
| 'testString123'        | 'test string123'       |
| 'testString_1_2_3'     | 'test string 1 2 3'    |
| 'ID123String'          | 'id123 string'         |
| 'foo bar123'           | 'foo bar123'           |
| 'a1bStar'              | 'a1b star'             |
| 'CONSTANT_CASE '       | 'constant case'        |
| 'CONST123_FOO'         | 'const123 foo'         |
| 'FOO_bar'              | 'foo bar'              |
| 'XMLHttpRequest'       | 'xml http request'     |
| 'IQueryAArgs'          | 'i query a args'       |
| 'dot\.case'            | 'dot case'             |
| 'path/case'            | 'path case'            |
| 'snake_case'           | 'snake case'           |
| 'snake_case123'        | 'snake case123'        |
| 'snake_case_123'       | 'snake case 123'       |
| '"quotes"'             | 'quotes'               |
| 'version 0.45.0'       | 'version 0 45 0'       |
| 'version 0..78..9'     | 'version 0 78 9'       |
| 'version 4_99/4'       | 'version 4 99 4'       |
| ' test '               | 'test'                 |
| 'something_2014_other' | 'something 2014 other' |
| 'amazon s3 data'       | 'amazon s3 data'       |
| 'foo_13_bar'           | 'foo 13 bar'           |

## pascalCase

将字符串值转换为帕斯卡命名法。非常适合生成 JavaScript 类名。

```ts
import string from '@adonisjs/core/helpers/string'

string.pascalCase('user team') // UserTeam
```

以下是一些转换示例。

| Input            | Output        |
| ---------------- | ------------- |
| 'test'           | 'Test'        |
| 'test string'    | 'TestString'  |
| 'Test String'    | 'TestString'  |
| 'TestV2'         | 'TestV2'      |
| 'version 1.2.10' | 'Version1210' |
| 'version 1.21.0' | 'Version1210' |

## sentenceCase

将值转换为句子。

```ts
import string from '@adonisjs/core/helpers/string'

string.sentenceCase('getting_started-with-adonisjs')
// Getting started with adonisjs
```

以下是一些转换示例。

| Input            | Output           |
| ---------------- | ---------------- |
| 'test'           | 'Test'           |
| 'test string'    | 'Test string'    |
| 'Test String'    | 'Test string'    |
| 'TestV2'         | 'Test v2'        |
| 'version 1.2.10' | 'Version 1 2 10' |
| 'version 1.21.0' | 'Version 1 21 0' |

## snakeCase

将值转换为蛇形命名法。

```ts
import string from '@adonisjs/core/helpers/string'

string.snakeCase('user team') // user_team
```

以下是一些转换示例。

| Input            | Output         |
| ---------------- | -------------- |
| '\_id'           | 'id'           |
| 'test'           | 'test'         |
| 'test string'    | 'test_string'  |
| 'Test String'    | 'test_string'  |
| 'Test V2'        | 'test_v2'      |
| 'TestV2'         | 'test_v_2'     |
| 'version 1.2.10' | 'version_1210' |
| 'version 1.21.0' | 'version_1210' |

## titleCase

将字符串值转换为标题大小写。

```ts
import string from '@adonisjs/core/helpers/string'

string.titleCase('small word ends on')
// Small Word Ends On
```

以下是一些转换示例。

| Input                              | Output                             |
| ---------------------------------- | ---------------------------------- |
| 'one. two.'                        | 'One. Two.'                        |
| 'a small word starts'              | 'A Small Word Starts'              |
| 'small word ends on'               | 'Small Word Ends On'               |
| 'we keep NASA capitalized'         | 'We Keep NASA Capitalized'         |
| 'pass camelCase through'           | 'Pass camelCase Through'           |
| 'follow step-by-step instructions' | 'Follow Step-by-Step Instructions' |
| 'this vs. that'                    | 'This vs. That'                    |
| 'this vs that'                     | 'This vs That'                     |
| 'newcastle upon tyne'              | 'Newcastle upon Tyne'              |
| 'newcastle \*upon\* tyne'          | 'Newcastle \*upon\* Tyne'          |

## random

生成给定长度的加密安全随机字符串。输出值是 URL 安全的 base64 编码字符串。

```ts
import string from '@adonisjs/core/helpers/string'

string.random(32)
// 8mejfWWbXbry8Rh7u8MW3o-6dxd80Thk
```

## sentence

将单词数组转换为逗号分隔的句子。

```ts
import string from '@adonisjs/core/helpers/string'

string.sentence(['routes', 'controllers', 'middleware'])
// routes, controllers, and middleware
```

你可以通过指定 `options.lastSeparator` 属性将 `and` 替换为 `or`。

```ts
string.sentence(['routes', 'controllers', 'middleware'], {
  lastSeparator: ', or ',
})
```

在以下示例中，两个单词使用 `and` 分隔符而不是逗号（通常在英语中提倡使用）。但是，你可以为一对单词使用自定义分隔符。

```ts
string.sentence(['routes', 'controllers'])
// routes and controllers

string.sentence(['routes', 'controllers'], {
  pairSeparator: ', and ',
})
// routes, and controllers
```
## condenseWhitespace

从字符串中移除多个空格，只保留一个空格。

```ts
import string from '@adonisjs/core/helpers/string'

string.condenseWhitespace('hello  world')
// hello world

string.condenseWhitespace('  hello  world  ')
// hello world
```

## seconds

将基于字符串的时间表达式解析为秒。

```ts
import string from '@adonisjs/core/helpers/string'

string.seconds.parse('10h') // 36000
string.seconds.parse('1 day') // 86400
```

将数值传递给 `parse` 方法时，将原样返回，假设该值已经是秒。

```ts
string.seconds.parse(180) // 180
```

使用 `format` 方法，可以将秒格式化为易读的字符串。

```ts
string.seconds.format(36000) // 10h
string.seconds.format(36000, true) // 10 hours
```

## milliseconds

将基于字符串的时间表达式解析为毫秒。

```ts
import string from '@adonisjs/core/helpers/string'

string.milliseconds.parse('1 h') // 3.6e6
string.milliseconds.parse('1 day') // 8.64e7
```

将数值传递给 `parse` 方法时，将原样返回，假设该值已经是毫秒。

```ts
string.milliseconds.parse(180) // 180
```

使用 `format` 方法，可以将毫秒格式化为易读的字符串。

```ts
string.milliseconds.format(3.6e6) // 1h
string.milliseconds.format(3.6e6, true) // 1 hour
```

## bytes

将基于字符串的单位表达式解析为字节。

```ts
import string from '@adonisjs/core/helpers/string'

string.bytes.parse('1KB') // 1024
string.bytes.parse('1MB') // 1048576
```

将数值传递给 `parse` 方法时，将原样返回，假设该值已经是字节。

```ts
string.bytes.parse(1024) // 1024
```

使用 `format` 方法，可以将字节格式化为易读的字符串。该方法直接从 [bytes](https://www.npmjs.com/package/bytes) 包中导出。有关可用选项，请参考该包的 README。

```ts
string.bytes.format(1048576) // 1MB
string.bytes.format(1024 * 1024 * 1000) // 1000MB
string.bytes.format(1024 * 1024 * 1000, { thousandsSeparator: ',' }) // 1,000MB
```

## ordinal

获取给定数字的序数词。

```ts
import string from '@adonisjs/core/helpers/string'

string.ordinal(1) // 1st
string.ordinal(2) // '2nd'
string.ordinal(3) // '3rd'
string.ordinal(4) // '4th'

string.ordinal(23) // '23rd'
string.ordinal(24) // '24th'
```

## safeEqual

检查两个缓冲区或字符串值是否相同。此方法不会泄露任何时间信息，从而防止 [timing attack](https://javascript.plainenglish.io/what-are-timing-attacks-and-how-to-prevent-them-using-nodejs-158cc7e2d70c)。

在底层，此方法使用 Node.js 的 [crypto.timeSafeEqual](https://nodejs.org/api/crypto.html#cryptotimingsafeequala-b) 方法，并支持比较字符串值。（crypto.timeSafeEqual 不支持字符串比较）

```ts
import { safeEqual } from '@adonisjs/core/helpers'

/**
 * 可信值，可能保存在数据库中
 */
const trustedValue = 'hello world'

/**
 * 不可信的用户输入
 */
const userInput = 'hello'

if (safeEqual(trustedValue, userInput)) {
  // 两者相同
} else {
  // 值不匹配
}
```

## cuid

创建一个安全、防冲突的 ID，优化用于水平扩展和性能。此方法在底层使用 [@paralleldrive/cuid2](https://github.com/paralleldrive/cuid2) 包。

```ts
import { cuid } from '@adonisjs/core/helpers'

const id = cuid()
// tz4a98xxat96iws9zmbrgj3a
```

可以使用 `isCuid` 方法检查一个值是否是有效的 CUID。

```ts
import { cuid, isCuid } from '@adonisjs/core/helpers'

const id = cuid()
isCuid(id) // true
```

## compose

`compose` 助手允许你使用更简洁的 API 使用 TypeScript 类混入。以下是没有 `compose` 助手的混入用法示例。

```ts
class User extends UserWithAttributes(UserWithAge(UserWithPassword(UserWithEmail(BaseModel)))) {}
```

以下是使用 `compose` 助手的示例。

- 没有嵌套。
- 混入的顺序是从（左到右/上到下）。而之前是从内到外。

```ts
import { compose } from '@adonisjs/core/helpers'

class User extends compose(
  BaseModel,
  UserWithEmail,
  UserWithPassword,
  UserWithAge,
  UserWithAttributes
) {}
```

## base64

用于 base64 编码和解码值的实用方法。

```ts
import { base64 } from '@adonisjs/core/helpers'

base64.encode('hello world')
// aGVsbG8gd29ybGQ=
```

与 `encode` 方法类似，你可以使用 `urlEncode` 生成可在 URL 中安全传递的 base64 字符串。

`urlEncode` 方法执行以下替换。

- 将 `+` 替换为 `-`。
- 将 `/` 替换为 `_`。
- 并从字符串末尾移除 `=` 符号。

```ts
base64.urlEncode('hello world')
// aGVsbG8gd29ybGQ
```

你可以使用 `decode` 和 `urlDecode` 方法解码之前编码的 base64 字符串。

```ts
base64.decode(base64.encode('hello world'))
// hello world

base64.urlDecode(base64.urlEncode('hello world'))
// hello world
```

当输入值是无效的 base64 字符串时，`decode` 和 `urlDecode` 方法返回 `null`。你可以启用 `strict` 模式以抛出异常。

```ts
base64.decode('hello world') // null
base64.decode('hello world', 'utf-8', true) // 抛出异常
```

## fsReadAll

从目录中获取所有文件的列表。该方法递归地从主目录和子文件夹中获取文件。隐式忽略点文件。

```ts
import { fsReadAll } from '@adonisjs/core/helpers'

const files = await fsReadAll(new URL('./config', import.meta.url), { pathType: 'url' })
await Promise.all(files.map((file) => import(file)))
```

你还可以将选项作为第二个参数与目录路径一起传递。

```ts
type Options = {
  ignoreMissingRoot?: boolean
  filter?: (filePath: string, index: number) => boolean
  sort?: (current: string, next: string) => number
  pathType?: 'relative' | 'unixRelative' | 'absolute' | 'unixAbsolute' | 'url'
}

const options: Partial<Options> = {}
await fsReadAll(location, options)
```

| 参数 | 描述 |
|------------|------------|
| `ignoreMissingRoot` | 默认情况下，当根目录缺失时会抛出异常。将 `ignoreMissingRoot` 设置为 true 不会导致错误，并返回一个空数组。 |
| `filter` | 定义过滤器以忽略某些路径。该方法在最终文件列表上调用。 |
| `sort` | 定义自定义方法来排序文件路径。默认情况下，文件使用自然排序。 |
| `pathType` | 定义如何返回收集的路径。默认情况下，返回特定于操作系统的相对路径。如果要导入收集的文件，必须设置 `pathType = 'url'` |

## fsImportAll
`fsImportAll` 方法递归地从给定目录导入所有文件，并将每个模块的导出值设置在一个对象上。

```ts
import { fsImportAll } from '@adonisjs/core/helpers'

const collection = await fsImportAll(new URL('./config', import.meta.url))
console.log(collection)
```

- Collection 是一个包含键值对树的对象。
- 键是从文件路径创建的嵌套对象。
- 值是模块的导出值。如果模块同时有 `default` 和 `named` 导出，则只使用默认导出。

第二个参数是用于自定义导入行为的选项。

```ts
type Options = {
  ignoreMissingRoot?: boolean
  filter?: (filePath: string, index: number) => boolean
  sort?: (current: string, next: string) => number
  transformKeys? (keys: string[]) => string[]
}

const options: Partial<Options> = {}
await fsImportAll(location, options)
```

| 参数 | 描述 |
|------------|------------|
| `ignoreMissingRoot` | 默认情况下，当根目录缺失时会抛出异常。将 `ignoreMissingRoot` 设置为 true 不会导致错误，并会返回一个空对象。 |
| `filter` | 定义过滤器以忽略某些路径。默认情况下，只导入以 `.js`、`.ts`、`.json`、`.cjs` 和 `.mjs` 结尾的文件。 |
| `sort` | 定义自定义方法来对文件路径进行排序。默认情况下，文件使用自然排序。 |
| `transformKeys` | 定义回调方法来转换最终对象的键。该方法接收一个嵌套键数组，并必须返回一个数组。 |

## 字符串构建器

`StringBuilder` 类提供了一个流畅的 API 来对字符串值进行转换。你可以使用 `string.create` 方法获取字符串构建器的实例。

```ts
import string from '@adonisjs/core/helpers/string'

const value = string
  .create('userController')
  .removeSuffix('controller') // user
  .plural() // users
  .snakeCase() // users
  .suffix('_controller') // users_controller
  .ext('ts') // users_controller.ts
  .toString()
```

## 消息构建器

`MessageBuilder` 类提供了一个 API 来序列化带有过期时间和用途的 JavaScript 数据类型。你可以将序列化输出存储在安全存储中（如应用程序数据库），或对其进行加密（以防止篡改）并公开共享。

在下面的示例中，我们对一个包含 `token` 属性的对象进行序列化，并将其过期时间设置为 `1 hour`。

```ts
import { MessageBuilder } from '@adonisjs/core/helpers'

const builder = new MessageBuilder()
const encoded = builder.build(
  {
    token: string.random(32),
  },
  '1 hour',
  'email_verification'
)

/**
 * {
 *   "message": {
 *    "token":"GZhbeG5TvgA-7JCg5y4wOBB1qHIRtX6q"
 *   },
 *   "purpose":"email_verification",
 *   "expiryDate":"2022-10-03T04:07:13.860Z"
 * }
 */
```

一旦你获得了带有过期时间和用途的 JSON 字符串，你可以对其进行加密（以防止篡改）并与客户端共享。

在令牌验证期间，你可以解密之前加密的值，并使用 `MessageBuilder` 来验证有效载荷并将其转换为 JavaScript 对象。

```ts
import { MessageBuilder } from '@adonisjs/core/helpers'

const builder = new MessageBuilder()
const decoded = builder.verify(value, 'email_verification')
if (!decoded) {
  return 'Invalid payload'
}

console.log(decoded.token)
```

## 密钥

`Secret` 类允许你在应用程序中持有敏感值，而不会意外地在日志和控制台语句中泄露它们。

例如，`config/app.ts` 文件中定义的 `appKey` 值是 `Secret` 类的一个实例。如果你尝试将此值记录到控制台，你将看到 `[redacted]` 而不是原始值。

为了演示，让我们启动一个 REPL 会话并尝试一下。

```sh
node ace repl
```

```sh
> (js) config = await import('./config/app.js')

# [Module: null prototype] {
  // highlight-start
#   appKey: [redacted],
  // highlight-end
#   http: {
#   }
# }
```

```sh
> (js) console.log(config.appKey)

# [redacted]
```

你可以调用 `config.appKey.release` 方法来读取原始值。`Secret` 类的目的并不是阻止你的代码访问原始值。相反，它提供了一个安全网，以防止在日志中暴露敏感数据。

### 使用 Secret 类

你可以如下将自定义值包装在 `Secret` 类中。

```ts
import { Secret } from '@adonisjs/core/helpers'
const value = new Secret('some-secret-value')

console.log(value) // [redacted]
console.log(value.release()) // some-secret-value
```

## 类型检测

我们从 `helpers/is` 导入路径导出了 [@sindresorhus/is](https://github.com/sindresorhus/is) 模块，你可以使用它在你的应用程序中进行类型检测。

```ts
import is from '@adonisjs/core/helpers/is'

is.object({}) // true
is.object(null) // false
```
