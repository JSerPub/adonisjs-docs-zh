# 文档模板

我们在 AdonisJS 项目中使用的模板仓库，用于创建文档网站。该模板允许最大程度的自定义，而不会过于冗长。

## 为什么不使用类似 VitePress 的工具？

我从未成为前端优先工具渲染 Markdown 文件为静态 HTML 的忠实粉丝。我还记得 Gridsome 和 Gatsby 的时代，当时使用 GraphQL 构建静态网站被认为是正常的 😇。

话虽如此，[围绕渲染 Markdown 的功能集](https://vitepress.dev/guide/markdown)感觉现代且令人耳目一新，它使用了前端工具。但是，底层库并不局限于前端生态系统，您可以在任何 JavaScript 项目中使用它们。

因此，如果我有所有可用的工具，为什么不构建和使用一些简单的东西，而这个东西不会随着前端生态系统的新一波创新而改变呢？

## 工作流程

文档模板基于以下工作流程要求构建。

- 创建一个高度可定制的 Markdown 渲染管道。我需要控制每个 Markdown 元素的渲染，并根据我的需求调整其 HTML 输出。这由 [@dimerapp/markdown](https://github.com/dimerapp/markdown) 和 [@dimerapp/edge](https://github.com/dimerapp/edge) 包提供支持。

- 使用 [Shiki](https://github.com/shikijs/shiki) 为代码块设置样式。Shiki 使用 VSCode 主题和语法进行语法高亮，并且不需要前端代码。

- 使用一个 [基础 HTML 和 CSS 主题](https://github.com/dimerapp/docs-theme) 来避免每次都要从头开始构建文档网站。但仍然允许自定义以给每个网站添加个性。

- 使用一个简单的 JSON 文件来渲染文档侧边栏（JSON 数据库文件）。扫描文件和文件夹并按某种约定对它们进行排序会使重构变得更加困难。

- 允许链接到 Markdown 文件，并在渲染为 HTML 时自动解析其 URL。

- 允许将图像和视频放在 Markdown 内容旁边，并在渲染为 HTML 时自动解析其 URL。

## 文件夹结构

```
.
├── assets
│  ├── app.css
│  └── app.js
├── bin
│  ├── build.ts
│  └── serve.ts
├── content
│  ├── docs
│  └── config.json
├── src
│  ├── bootstrap.ts
│  └── collections.ts
├── templates
│  ├── elements
│  ├── layouts
│  ├── partials
│  └── docs.edge
├── vscode_grammars
│  ├── dotenv.tmLanguage.json
│  └── main.ts
├── package-lock.json
├── package.json
├── README.md
├── tsconfig.json
└── vite.config.js
```

### assets 目录

`assets` 目录包含 CSS 和前端 JavaScript 入口点文件。我们主要在这些文件中导入其他包和 [基础主题](https://github.com/dimerapp/docs-theme)。但是，您可以自由调整这些文件以创建更具个性化的网站。

### bin 目录

`bin` 目录包含两个脚本文件，用于启动开发服务器并将文档导出为静态 HTML 文件。这些脚本在底层启动 AdonisJS 框架。

### content 目录

`content` 目录包含 Markdown 和数据库 JSON 文件。我们将 Markdown 文件组织成集合，每个集合都有自己的数据库文件。

您可以将集合视为网站上不同的文档区域。例如：您可以为 **文档** 创建一个集合，为 **API** 参考创建一个集合，以及为 **配置参考** 创建一个集合。

另请参阅：[创建新集合](#creating-new-collections)

### src 目录

`src` 目录包含一个 `bootstrap` 文件，用于将所有内容连接在一起。我们不将引导过程隐藏在某个包中。这是因为我们希望最终项目能够完全控制配置、引入额外包或移除未使用的功能。

`collections.ts` 文件用于定义一个或多个集合。

### templates 目录

`templates` 目录包含用于渲染 HTML 的 Edge 模板。

- `docs` 模板渲染一个常规的文档布局，包含标题、侧边栏、内容和目录。您可以在多个集合中使用相同的模板。
- Logo 作为 SVG 保存在 `partials/logo.edge` 和 `partials/logo_mobile.edge` 文件中。
- 基础 HTML 片段是 `layouts/main.edge` 文件的一部分。您可以在此文件中添加自定义元标签或脚本/字体。

### vscode_grammars 目录

`vscode_grammars` 目录包含您希望在项目中使用的自定义 VSCode 语言集合。

另请参阅：[使用自定义 VSCode 语法](#using-custom-vscode-grammars)

## 使用方法

从 GitHub 克隆仓库。我们建议使用 [degit](https://www.npmjs.com/package/degit)，它会在不保留 git 历史的情况下下载仓库。

```sh
npx degit dimerapp/docs-boilerplate <my-website>
```

安装依赖项

```sh
cd <my-website>
npm i
```

运行开发服务器。

```sh
npm run dev
```

然后访问 [http://localhost:3333/docs/introduction](http://localhost:3333/docs/introduction) URL 在浏览器中查看网站。

## 添加内容

默认情况下，我们创建一个包含 `introduction.md` 文件的 `docs` 集合。

作为第一步，您应该打开 `content/docs/db.json` 文件，并为您的文档添加所有条目。最初手动定义条目可能会感到乏味，但这将允许在未来进行更轻松的自定义。

一个典型的数据库条目具有以下属性。

```json
{
  "permalink": "introduction",
  "title": "Introduction",
  "contentPath": "./http_overview.md",
  "category": "Guides"
}
```

- `permalink`: 文档的唯一 URL。集合前缀将自动应用于永久链接。请参阅 `src/collection.ts` 文件中的集合前缀。
- `title`: 在侧边栏中显示的标题。
- `contentPath`: 指向 Markdown 文件的相对路径。
- `category`: 文档的分组类别。

定义完所有条目后，创建 Markdown 文件并编写一些实际内容。

## 更改网站配置

我们使用一个非常简洁的配置文件来更新网站的某些部分。配置存储在 `content/config.json` 文件中。

```json
{
  "links": {
    "home": {
      "title": "Your project name",
      "href": "/"
    },
    "github": {
      "title": "Your project on GitHub",
      "href": "https://github.com/dimerapp"
    }
  },
  "fileEditBaseUrl": "https://github.com/dimerapp/docs-boilerplate/blob/develop",
  "copyright": "Your project legal name"
}
```

- `links`: 对象包含两个固定链接。主页和 GitHub 项目 URL。

- `fileEditBaseUrl`: GitHub 上文件的基础 URL。这用于内容页脚以显示 **在 GitHub 上编辑** 链接。

- `copyright`: 在版权页脚中显示的名称。

- `menu`: 可选地，您可以定义一个头部菜单作为对象数组。
  ```json
  {
    "menu": [
      {
        "href": "/docs/introduction",
        "title": "Docs",
      },
      {
        "href": "https://blog.project.com",
        "title": "Blog",
      },
      {
        "href": "https://github.com/project/releases",
        "title": "Releases",
      }
    ]
  }
  ```

- `search`: 可选地，您可以为 Algolia 搜索定义配置。
  ```json
  {
    "search": {
      "appId": "",
      "indexName": "",
      "apiKey": ""
    }
  }
  ```

## 创建新集合

您可以通过在 `src/collections.ts` 文件中定义它们来创建多个集合。

集合使用 `Collection` 类定义。该类接受数据库文件的 URL。配置好集合后，调用 `collection.boot`。

```ts
// Docs
const docs = new Collection()
  .db(new URL('../content/docs/db.json', import.meta.url))
  .useRenderer(renderer)
  .urlPrefix('/docs')

await docs.boot()

// API reference
const apiReference = new Collection()
  .db(new URL('../content/api_reference/db.json', import.meta.url))
  .useRenderer(renderer)
  .urlPrefix('/api')

await apiReference.boot()

export const collections = [docs, apiReference]
```

## 使用自定义 VSCode 语法

您可以通过在 `vscode_grammars/main.ts` 文件中定义它们来添加对自定义 VSCode 语言的支持。每个条目都必须符合 [Shiki](https://github.com/shikijs/shiki/blob/main/docs/languages.md) 中的 `ILanguageRegistration` 接口。

## 更改 Markdown 代码块主题

代码块主题使用在 `src/bootstrap.ts` 文件中创建的 Markdown 渲染器实例定义。您可以使用 [预定义主题或自定义主题](https://github.com/dimerapp/shiki/tree/next#using-different-themes)。

```ts
export const renderer = new Renderer(view, pipeline)
  .codeBlocksTheme('material-theme-palenight')
```

## 自定义 CSS

[基础文档主题](https://github.com/dimerapp/docs-theme) 大量使用 CSS 变量，因此您可以通过定义一组新的变量来调整大多数样式。

如果要更改颜色，我们建议使用 [Radix Colors](https://www.radix-ui.com/docs/colors/getting-started/installation)，因为我们使用的是默认样式。

## 自定义 HTML

由于我们不是为全世界创建一个通用的文档生成器，因此 HTML 输出不是 100% 可定制的。该模板旨在在约束条件下使用。

但是，您仍然可以控制布局，因为页面的所有部分都作为 Edge 组件导出，并且您可以在 DOM 中的任何位置放置它们。请检查 `templates/docs.edge` 文件以了解如何使用一切。

### 页眉插槽

您可以将以下组件插槽传递给网站页眉。

- `logo (必需)`: 在桌面视口上显示的 Logo 内容。

- `logoMobile (必需)`: 在移动视口上显示的 Logo 内容。

- `popupMenu (可选)`: 为弹出菜单触发器定义自定义标记。触发器仅在移动视图中显示。
  ```edge
  @component('docs::header', contentConfig)
    @slots('popMenu')
      <span> Open popup menu </span>
    @end
  @end
  ```

- `themeSwitcher (可选)`: 为主题切换器按钮定义自定义标记。
  ```edge
  @component('docs::header', contentConfig)
    @slots('themeSwitcher')
      <span x-if="store.darkMode.enabled"> Dark </span>
      <span x-if="!store.darkMode.enabled"> Light </span>
    @end
  @end
  ```

- `github (可选)`: 为页眉中的 GitHub 链接定义自定义标记。
  ```edge
  @component('docs::header', contentConfig)
    @slots('github')
      <span> GitHub (11K+ Stars) </span>
    @end
  @end
  ```
  