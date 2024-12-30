---
summary: Prompts 是用于用户输入的终端小部件，使用 @poppinss/prompts 包。它们支持各种类型，如输入、密码和选择，并且设计为易于测试集成。
---

# Prompts

Prompts 是交互式终端小部件，您可以使用它们来接受用户输入。Ace prompts 由 [@poppinss/prompts](https://github.com/poppinss/prompts) 包提供支持，该包支持以下提示类型。

- input
- list
- password
- confirm
- toggle
- select
- multi-select
- autocomplete

Ace prompts 在设计时考虑了测试。在编写测试时，您可以捕获提示并以编程方式响应它们。

另请参阅：[Testing ace commands](../testing/console_tests.md)

## 显示提示

您可以使用所有 Ace 命令上可用的 `this.prompt` 属性来显示提示。

```ts
import { BaseCommand } from '@adonisjs/core/ace'

export default class GreetCommand extends BaseCommand {
  async run() {
    const modelName = await this.prompt.ask('Enter the model name')
    
    console.log(modelName)
  }
}
```

## 提示选项

以下是提示接受的选项列表。您可以将此表作为唯一可靠的信息来源。

<table>
<tr>
<td width="110px">选项</td>
<td width="120px">接受者</td>
<td>描述</td>
</tr>
<tr>
<td>

`default` (字符串) 

</td>

<td>

所有提示

</td>

<td>

未输入值时使用的默认值。在 `select`、`multiselect` 和 `autocomplete` 提示中，该值必须是选择数组索引。

</td>
</tr>

<tr>
<td>

`name` (字符串)

</td>

<td>

所有提示

</td>

<td>

提示的唯一名称

</td>
</tr>

<tr>
<td>

`hint` (字符串)

</td>

<td>

所有提示

</td>

<td>

在提示旁边显示的提示文本

</td>
</tr>
<tr>
<td>

`result` (函数)

</td>

<td>所有提示</td>
<td>

转换提示返回值。`result` 方法的输入值取决于提示。例如，`multiselect` 提示值将是所选选择的数组。

```ts
{
  result(value) {
    return value.toUpperCase()
  }
}
```

</td>
</tr>

<tr>
<td>

`format` (函数)

</td>

<td>所有提示</td>

<td>

在用户输入时实时格式化输入值。格式化仅应用于 CLI 输出，不应用于返回值。

```ts
{
  format(value) {
    return value.toUpperCase()
  }
}
```

</td>
</tr>

<tr>
<td>

`validate` (函数)

</td>

<td>所有提示</td>

<td>

验证用户输入。从方法返回 `true` 将通过验证。返回 `false` 或错误消息字符串将被视为失败。

```ts
{
  format(value) {
    return value.length > 6
    ? true
    : 'Model name must be 6 characters long'
  }
}
```

</td>
</tr>

<tr>
<td>

`limit` (数字)

</td>

<td>

`autocomplete`

</td>

<td>

限制显示的选项数量。您将需要滚动查看其余选项。

</td>
</tr>
</table>

## 文本输入

您可以使用 `prompt.ask` 方法呈现提示以接受文本输入。该方法接受提示消息作为第一个参数，并将 [选项对象](#prompt-options) 作为第二个参数。

```ts
await this.prompt.ask('Enter the model name')
```

```ts
// 验证输入
await this.prompt.ask('Enter the model name', {
  validate(value) {
    return value.length > 0
  }
})
```

```ts
// 默认值
await this.prompt.ask('Enter the model name', {
  default: 'User'
})
```

## 掩码输入

顾名思义，掩码输入提示会在终端中屏蔽用户输入。您可以使用 `prompt.secure` 方法显示掩码提示。

该方法接受提示消息作为第一个参数，并将 [选项对象](#prompt-options) 作为第二个参数。

```ts
await this.prompt.secure('Enter account password')
```

```ts
await this.prompt.secure('Enter account password', {
  validate(value) {
    return value.length < 6
      ? 'Password must be 6 characters long'
      : true
  }
})
```

## 选择列表

您可以使用 `prompt.choice` 方法显示单个选择的选择列表。该方法接受以下参数。

1. 提示消息。
2. 选择数组。
3. 可选的 [选项对象](#prompt-options)。

```ts
await this.prompt.choice('Select package manager', [
  'npm',
  'yarn',
  'pnpm'
])
```

若要指定不同的显示值，可以将选项定义为对象。`name` 属性作为提示结果返回，`message` 属性在终端中显示。

```ts
await this.prompt.choice('Select database driver', [
  {
    name: 'sqlite',
    message: 'SQLite'
  },
  {
    name: 'mysql',
    message: 'MySQL'
  },
  {
    name: 'pg',
    message: 'PostgreSQL'
  }
])
```

## 多选

您可以使用 `prompt.multiple` 方法允许在选择列表中进行多项选择。接受的参数与 `choice` 提示相同。

```ts
await this.prompt.multiple('Select database drivers', [
  {
    name: 'sqlite',
    message: 'SQLite'
  },
  {
    name: 'mysql',
    message: 'MySQL'
  },
  {
    name: 'pg',
    message: 'PostgreSQL'
  }
])
```

## 确认操作

您可以使用 `prompt.confirm` 方法显示带有 `Yes/No` 选项的确认提示。该方法接受提示消息作为第一个参数，并将 [选项对象](#prompt-options) 作为第二个参数。

`confirm` 提示返回一个布尔值。

```ts
const deleteFiles = await this.prompt.confirm(
  'Want to delete all files?'
)

if (deleteFiles) {
}
```

若要自定义 `Yes/No` 选项的显示值，可以使用 `prompt.toggle` 方法。

```ts
const deleteFiles = await this.prompt.toggle(
  'Want to delete all files?',
  ['Yup', 'Nope']
)

if (deleteFiles) {
}
```

## 自动完成

`autocomplete` 提示是 select 和 multi-select 提示的组合，但具有模糊搜索选择的能力。

```ts
const selectedCity = await this.prompt.autocomplete(
  'Select your city',
  await getCitiesList()
)
```
