---
summary: 如何创建和配置一个新的 AdonisJS 应用程序。
---

# 安装

在创建新应用程序之前，请确保您的计算机上已安装 Node.js 和 npm。AdonisJS 需要 `Node.js >= 20.6`。

您可以使用 [官方安装程序](https://nodejs.org/en/download/) 或 [Volta](https://docs.volta.sh/guide/getting-started) 安装 Node.js。Volta 是一个跨平台包管理器，可在您的计算机上安装和运行多个 Node.js 版本。

```sh
// title: 验证 Node.js 版本
node -v
# v22.0.0
```

:::tip
**您是视觉学习者吗？** - 查看我们朋友在 Adocasts 上的 [Let's Learn AdonisJS 6](https://adocasts.com/series/lets-learn-adonisjs-6) 免费屏幕录制系列。
:::

## 创建新应用程序

您可以使用 [npm init](https://docs.npmjs.com/cli/v7/commands/npm-init) 创建一个新项目。这些命令将下载 [create-adonisjs](http://npmjs.com/create-adonisjs) 初始化程序包并开始安装过程。

您可以使用以下 CLI 标志自定义初始项目输出。

- `--kit`: 选择项目的 [启动套件](#starter-kits)。您可以选择 **web**、**api**、**slim** 或 **inertia**。

- `--db`: 指定您选择的数据库方言。您可以选择 **sqlite**、**postgres**、**mysql** 或 **mssql**。

- `--git-init`: 初始化 git 存储库。默认为 `false`。

- `--auth-guard`: 指定您选择的身份验证防护。您可以选择 **session**、**access_tokens** 或 **basic_auth**。

:::codegroup

```sh
// title: npm
npm init adonisjs@latest hello-world
```

:::

在使用 `npm init` 命令传递 CLI 标志时，请确保使用 [双破折号两次](https://stackoverflow.com/questions/43046885/what-does-do-when-running-an-npm-command)。否则，`npm init` 不会将标志传递给 `create-adonisjs` 初始化程序包。例如：

```sh
# 创建一个项目并提示输入所有选项
npm init adonisjs@latest hello-world

# 创建一个使用 MySQL 的项目
npm init adonisjs@latest hello-world -- --db=mysql

# 创建一个使用 PostgreSQL 和 API 启动套件的项目
npm init adonisjs@latest hello-world -- --db=postgres --kit=api

# 创建一个使用 API 启动套件和访问令牌防护的项目
npm init adonisjs@latest hello-world -- --kit=api --auth-guard=access_tokens
```

## 启动套件

启动套件是使用 AdonisJS 创建应用程序的起点。它们具有 [有主见的文件夹结构](./folder_structure.md)、预配置的 AdonisJS 包以及开发所需的必要工具。

:::note

官方启动套件使用 ES 模块和 TypeScript。这种组合允许您使用现代 JavaScript 构造并利用静态类型安全。

:::

### Web 启动套件

Web 启动套件专为创建传统的服务器端渲染 Web 应用程序而设计。不要让关键词 **"传统"** 阻止您。如果您制作一个前端交互性有限的 Web 应用程序，我们推荐这个启动套件。

使用 [Edge.js](https://edgejs.dev) 在服务器上渲染 HTML 的简单性将提高您的工作效率，因为您无需处理复杂的构建系统来渲染一些 HTML。

稍后，您可以使用 [Hotwire](https://hotwired.dev)、[HTMX](http://htmx.org) 或 [Unpoly](http://unpoly.com) 使您的应用程序像 SPA 一样导航，并使用 [Alpine.js](http://alpinejs.dev) 创建交互式小部件，如下拉菜单或模态框。

```sh
npm init adonisjs@latest -- -K=web

# 切换数据库方言
npm init adonisjs@latest -- -K=web --db=mysql
```

Web 启动套件包含以下包。

<table>
<thead>
<tr>
<th width="180px">包</th>
<th>描述</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>@adonisjs/core</code></td>
<td>框架的核心提供了创建后端应用程序时可能需要的基础功能。</td>
</tr>
<tr>
<td><code>edge.js</code></td>
<td><a href="https://edgejs.dev">edge</a> 模板引擎，用于构建 HTML 页面。</td>
</tr>
<tr>
<td><code>@vinejs/vine</code></td>
<td><a href="https://vinejs.dev">VineJS</a> 是 Node.js 生态系统中速度最快的验证库之一。</td>
</tr>
<tr>
<td><code>@adonisjs/lucid</code></td>
<td>Lucid 是由 AdonisJS 核心团队维护的 SQL ORM。</td>
</tr>
<tr>
<td><code>@adonisjs/auth</code></td>
<td>框架的身份验证层。它配置为使用会话。</td>
</tr>
<tr>
<td><code>@adonisjs/shield</code></td>
<td>一组安全原语，可保护您的 Web 应用程序免受 **CSRF** 和 **XSS** 等攻击。</td>
</tr>
<tr>
<td><code>@adonisjs/static</code></td>
<td>中间件，用于从应用程序的 <code>/public</code> 目录提供静态资源。</td>
</tr>
<tr>
<td><code>vite</code></td>
<td><a href="https://vitejs.dev/">Vite</a> 用于编译前端资源。</td>
</tr>
</tbody>
</table>

---

### API 启动套件

API 启动套件专为创建 JSON API 服务器而设计。它是 `web` 启动套件的精简版。如果您计划使用 React 或 Vue 构建前端应用程序，可以使用 API 启动套件创建 AdonisJS 后端。

```sh
npm init adonisjs@latest -- -K=api

# 切换数据库方言
npm init adonisjs@latest -- -K=api --db=mysql
```

在此启动套件中：

- 我们移除了对提供静态文件的支持。
- 未配置视图层和 vite。
- 禁用了 XSS 和 CSRF 保护，并启用了 CORS 保护。
- 使用 ContentNegotiation 中间件以 JSON 格式发送 HTTP 响应。

API 启动套件配置为使用基于会话的身份验证。但是，如果您希望使用基于令牌的身份验证，可以使用 `--auth-guard` 标志。

另请参阅：[我应该使用哪种身份验证防护？](../authentication/introduction.md#choosing-an-auth-guard)

```sh
npm init adonisjs@latest -- -K=api --auth-guard=access_tokens
```

---

### Slim 启动套件

对于极简主义者，我们创建了一个 `slim` 启动套件。它仅包含框架的核心和默认文件夹结构。当您不希望使用 AdonisJS 的任何附加功能时，可以使用它。

```sh
npm init adonisjs@latest -- -K=slim

# 切换数据库方言
npm init adonisjs@latest -- -K=slim --db=mysql
```

---

### Inertia 启动套件

[Inertia](https://inertiajs.com/) 是一种构建服务器端驱动的单页应用程序的方法。您可以使用您喜爱的前端框架（React、Vue、Solid、Svelte）构建应用程序的前端。

您可以使用 `--adapter` 标志选择要使用的前端框架。可用选项为 `react`、`vue`、`solid` 和 `svelte`。

您还可以使用 `--ssr` 和 `--no-ssr` 标志打开或关闭服务器端渲染。

```sh
# 使用服务器端渲染的 React
npm init adonisjs@latest -- -K=inertia --adapter=react --ssr

# 不使用服务器端渲染的 Vue
npm init adonisjs@latest -- -K=inertia --adapter=vue --no-ssr
```

---

### 带来您自己的启动套件

启动套件是托管在 GitHub、Bitbucket 或 Gitlab 等 Git 存储库提供商上的预构建项目。您也可以创建自己的启动套件并按如下方式下载它们。

```sh
npm init adonisjs@latest -- -K="github_user/repo"

# 从 GitLab 下载
npm init adonisjs@latest -- -K="gitlab:user/repo"

# 从 BitBucket 下载
npm init adonisjs@latest -- -K="bitbucket:user/repo"
```

您可以使用 Git+SSH 身份验证下载私有存储库，使用 `git` 模式。

```sh
npm init adonisjs@latest -- -K="user/repo" --mode=git
```

最后，您可以指定一个标签、分支或提交。

```sh
# 分支
npm init adonisjs@latest -- -K="user/repo#develop"

# 标签
npm init adonisjs@latest -- -K="user/repo#v2.1.0"
```

## 启动开发服务器

创建 AdonisJS 应用程序后，您可以通过运行 `node ace serve` 命令启动开发服务器。

Ace 是捆绑在框架核心中的命令行框架。`--hmr` 标志监视文件系统，并对代码库中的某些部分执行 [热模块替换 (HMR)](../concepts/hmr.md)。

```sh
node ace serve --hmr
```

开发服务器运行后，您可以访问 [http://localhost:3333](http://localhost:3333) 在浏览器中查看您的应用程序。

## 为生产构建

由于 AdonisJS 应用程序是用 TypeScript 编写的，因此必须在生产环境中编译为 JavaScript。

您可以使用 `node ace build` 命令创建 JavaScript 输出。JavaScript 输出写入 `build` 目录。

当配置 Vite 时，此命令还会使用 Vite 编译前端资源，并将输出写入 `build/public` 文件夹。

另请参阅：[TypeScript 构建过程](../concepts/typescript_build_process.md)。

```sh
node ace build
```

## 配置开发环境

虽然 AdonisJS 负责构建最终用户应用程序，但您可能需要其他工具来享受开发过程并保持编码风格的一致性。

我们强烈建议您使用 **[ESLint](https://eslint.org/)** 来检查代码，并使用 **[Prettier](https://prettier.io)** 来重新格式化代码以保持一致性。

官方启动套件已预先配置 ESLint 和 Prettier，并使用 AdonisJS 核心团队的有主见预设。您可以在文档的 [Tooling config](../concepts/tooling_config.md) 部分中了解更多信息。

最后，我们建议您为代码编辑器安装 ESLint 和 Prettier 插件，以便在应用程序开发过程中获得更紧密的反馈循环。此外，您可以使用以下命令从命令行 `lint` 和 `format` 您的代码。

```sh
# 运行 ESLint
npm run lint

# 运行 ESLint 并自动修复问题
npm run lint -- --fix

# 运行 Prettier
npm run format
```

## VSCode 扩展

您可以在任何支持 TypeScript 的代码编辑器上开发 AdonisJS 应用程序。然而，我们为 VSCode 开发了几个扩展，以进一步提升开发体验。

- [**AdonisJS**](https://marketplace.visualstudio.com/items?itemName=jripouteau.adonis-vscode-extension) - 在代码编辑器中查看应用程序路由、运行 ace 命令、迁移数据库并直接阅读文档。

- [**Edge**](https://marketplace.visualstudio.com/items?itemName=AdonisJS.vscode-edge) - 通过语法高亮、自动补全和代码片段支持，提升您的开发工作流程。

- [**Japa**](https://marketplace.visualstudio.com/items?itemName=jripouteau.japa-vscode) - 使用键盘快捷键在代码编辑器中运行测试，或从活动侧边栏直接运行测试，而无需离开编辑器。
