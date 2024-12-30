---
summary: "了解如何在 AdonisJS 中测试与数据库交互的代码：设置、重置和在测试期间保持数据库清洁的简单步骤。"
---

# 数据库测试

数据库测试指的是测试您的应用程序如何与数据库进行交互。这包括测试写入数据库的内容、如何在测试前运行迁移，以及如何在测试之间保持数据库清洁。

## 数据库迁移

在执行与数据库交互的测试之前，您需要先运行迁移。在 `testUtils` 服务中，我们有两个钩子可供使用，您可以在 `tests/bootstrap.ts` 文件中进行配置。

### 每个运行周期后重置数据库

第一个选项是 `testUtils.db().migrate()`。这个钩子会首先运行所有迁移，然后回滚所有内容。

```ts
// title: tests/bootstrap.ts
import testUtils from '@adonisjs/core/services/test_utils'

export const runnerHooks: Required<Pick<Config, 'setup' | 'teardown'>> = {
  setup: [
    () => testUtils.db().migrate(),
  ],
  teardown: [],
}
```

通过在此处配置钩子，将会发生以下情况：

- 在运行测试之前，将执行迁移。
- 在测试结束后，将回滚数据库。

因此，每次运行测试时，我们都会拥有一个全新且空的数据库。

### 每个运行周期后截断表

每个运行周期后重置数据库是一个不错的选择，但如果迁移很多，可能会很慢。另一个选项是在每个运行周期后截断表。这个选项会更快，因为迁移只会在第一次在全新数据库上运行测试时执行一次。

在每个运行周期结束时，表将被截断，但我们的架构将被保留。因此，下次运行测试时，我们将拥有一个空的数据库，但架构已经就位，因此无需再次运行每个迁移。

```ts
// title: tests/bootstrap.ts
import testUtils from '@adonisjs/core/services/test_utils'

export const runnerHooks: Required<Pick<Config, 'setup' | 'teardown'>> = {
  setup: [
    () => testUtils.db().truncate(),
  ],
}
```

## 数据库填充

如果您需要填充数据库，可以使用 `testUtils.db().seed()` 钩子。这个钩子将在运行测试之前运行所有填充数据。

```ts
// title: tests/bootstrap.ts
import testUtils from '@adonisjs/core/services/test_utils'

export const runnerHooks: Required<Pick<Config, 'setup' | 'teardown'>> = {
  setup: [
    () => testUtils.db().seed(),
  ],
}
```

## 在测试之间保持数据库清洁

### 全局事务

在运行测试时，您可能希望在每个测试之间保持数据库清洁。为此，您可以使用 `testUtils.db().withGlobalTransaction()` 钩子。这个钩子将在每个测试之前启动一个事务，并在测试结束时回滚。

```ts
// title: tests/unit/user.spec.ts
import { test } from '@japa/runner'
import testUtils from '@adonisjs/core/services/test_utils'

test.group('User', (group) => {
  group.each.setup(() => testUtils.db().withGlobalTransaction())
})
```

请注意，如果您在测试代码中使用了任何事务，这将不起作用，因为事务不能嵌套。在这种情况下，您可以使用 `testUtils.db().migrate()` 或 `testUtils.db().truncate()` 钩子。

### 截断表

如上所述，如果您在测试代码中使用了事务，全局事务将不起作用。在这种情况下，您可以使用 `testUtils.db().truncate()` 钩子。这个钩子将在每个测试后截断所有表。

```ts
// title: tests/unit/user.spec.ts
import { test } from '@japa/runner'

test.group('User', (group) => {
  group.each.setup(() => testUtils.db().truncate())
})
```
