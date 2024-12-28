---
summary: 汇编器钩子是在汇编器生命周期的特定点执行代码的一种方式。
---

# 汇编器钩子

汇编器钩子是在汇编器生命周期的特定点执行代码的一种方式。提醒一下，汇编器是 AdonisJS 的一部分，它使您能够启动开发服务器、构建应用程序和运行测试。

这些钩子对于文件生成、代码编译或注入自定义构建步骤等任务非常有帮助。

例如， `@adonisjs/vite` 包使用 `onBuildStarting` 钩子来注入一个构建前端资源的步骤。因此，当您运行 `node ace build` 时，`@adonisjs/vite` 包将在其余构建过程之前构建您的前端资源。这是一个如何使用钩子自定义构建过程的好例子。

## 添加钩子

汇编器钩子在 `adonisrc.ts` 文件中的 `hooks` 键下定义：

```ts
import { defineConfig } from '@adonisjs/core/app'

export default defineConfig({
  hooks: {
    onBuildCompleted: [
      () => import('my-package/hooks/on_build_completed')
    ],
    onBuildStarting: [
      () => import('my-package/hooks/on_build_starting')
    ],
    onDevServerStarted: [
      () => import('my-package/hooks/on_dev_server_started')
    ],
    onSourceFileChanged: [
      () => import('my-package/hooks/on_source_file_changed')
    ],
  },
})
```

可以为汇编生命周期的每个阶段定义多个钩子。每个钩子是一个要执行的函数数组。

我们建议使用动态导入来加载钩子。这确保了钩子仅在需要时才加载，而不是不必要地加载。如果您直接将钩子代码写在`adonisrc.ts`文件中，这可能会减慢应用程序的启动速度。

## 创建钩子

钩子只是一个简单的函数。让我们以一个应该执行自定义构建任务的钩子为例。

```ts
// title: hooks/on_build_starting.ts
import type { AssemblerHookHandler } from '@adonisjs/core/types/app'

const buildHook: AssemblerHookHandler = async ({ logger }) => {
  logger.info('正在生成一些文件...')

  await myCustomLogic()
}

export default buildHook
```

请注意，钩子必须默认导出。

一旦定义了此钩子，您只需将其添加到`adonisrc.ts`文件中，如下所示：

```ts
// title: adonisrc.ts
import { defineConfig } from '@adonisjs/core/app'

export default defineConfig({
  hooks: {
    onBuildStarting: [
      () => import('./hooks/on_build_starting')
    ],
  },
})
```

现在，每次您运行`node ace build`时，都会执行`onBuildStarting`钩子，并运行您定义的自定义逻辑。

## 钩子列表

以下是可用钩子的列表：

### onBuildStarting

此钩子在构建开始之前执行。它对于文件生成或注入自定义构建步骤等任务非常有帮助。

### onBuildCompleted

此钩子在构建完成后执行。它也可以用于自定义构建过程。

### onDevServerStarted

此钩子在Adonis开发服务器启动后执行。

### onSourceFileChanged

此钩子在每次修改源文件（由您的`tsconfig.json`包含）时执行。您的钩子将接收修改文件的路径作为参数。