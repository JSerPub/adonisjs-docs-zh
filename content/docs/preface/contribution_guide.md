---
summary: 为 AdonisJS 项目做出贡献是回馈社区的好方法。本指南概述了如何为任何 AdonisJS 项目做出贡献的一般步骤。
---

# Contributing
这是所有 [AdonisJS](https://github.com/adonisjs) 仓库的通用贡献指南。在贡献任何仓库之前，请仔细阅读本指南 🙏

代码并不是贡献的唯一方式。以下也是一些贡献并成为社区一员的方式：

- 修正文档中的错别字
- 改进现有文档
- 撰写食谱或博客文章以教育社区中的其他人
- 筛选问题
- 就现有问题发表意见
- 在 Discord 和讨论论坛中帮助社区

## 报告错误
在开源项目中报告的许多问题通常是报告者方面的问题或配置错误。因此，我们强烈建议您在报告问题之前先自行排查。

如果您要报告错误，请提供尽可能多的信息以及您编写的代码示例。良好问题和不良问题的范围如下：

- **完美问题**：您隔离了底层错误。在仓库中创建一个失败的测试，并围绕它打开一个 GitHub 问题。
- **良好问题**：您隔离了底层错误，并提供了一个最小的 GitHub 仓库复现。Antfu 写了一篇关于[为什么需要复现](https://antfu.me/posts/why-reproductions-are-required) 的好文章。
- **不错的问题**：您准确陈述了问题。分享最初产生问题的代码。同时，包括相关的配置文件和您使用的包版本。

  最后但同样重要的是，按照 [GitHub Markdown 语法指南](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax) 正确格式化每个代码块。

- **差劲的问题**：您抛出问题，希望其他人会问相关问题并帮助您。这类问题会自动关闭，且没有任何解释。

## 进行讨论
您可能经常想要讨论某个话题或分享一些想法。在这种情况下，请在讨论论坛的 **💡Ideas** 类别下创建一个讨论。

## 教育他人
教育他人是贡献于任何社区并获得认可的最佳方式之一。

您可以使用我们讨论论坛上的 **📚 Cookbooks** 类别与他人分享文章。食谱部分并不严格受控，但分享的知识应与项目相关。

## 创建拉取请求
在投入大量时间和精力编写代码后，拉取请求被拒绝绝不是一种好的体验。因此，我们强烈建议您在开始任何新工作之前[先开始讨论](https://github.com/orgs/adonisjs/discussions)。

只需开始讨论并解释您计划贡献什么？

- **您是否试图创建拉取请求以修复错误**：一旦错误得到确认，修复错误的拉取请求通常会被接受。
- **您是否计划添加新功能**：请详细解释为什么需要此功能，并分享我们可以阅读的学习材料链接以进行自我教育。

  例如：如果您正在为 Japa 或 AdonisJS 添加快照测试支持。那么请分享我可以用来了解快照测试的一般信息的链接。

> 注意：您还应准备好为记录贡献的功能或改进打开额外的拉取请求。

## 仓库设置

1. 首先在本地机器上克隆仓库。

    ```sh
    git clone <REPO_URL>
    ```

2. 在本地安装依赖项。请不要在功能请求中更新任何依赖项。如果发现依赖项过时，请创建单独的拉取请求来更新它们。

   我们使用 `npm` 来管理依赖项，因此请不要使用 `yarn` 或任何其他工具。

    ```sh
    npm install
    ```

3. 通过执行以下命令运行测试。

    ```sh
    npm test
    ```

## 使用的工具
以下是正在使用的工具列表。

| 工具                   | 用法                                                                                                                                                                                                                                                                 |
|------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| TypeScript             | 所有仓库都用 TypeScript 编写。编译后的 JavaScript 和类型定义发布在 npm 上。                                                                                                                                                                                        |
| TS Node                | 我们使用 [ts-node](https://typestrong.org/ts-node/) 运行测试或脚本，而无需编译 TypeScript。ts-node 的主要目标是在开发过程中实现更快的反馈循环。                                                                                                                   |
| SWC                    | [SWC](https://swc.rs/) 是一个基于 Rust 的 TypeScript 编译器。TS Node 提供了一流的支持，可以使用 SWC 而不是 TypeScript 官方编译器。使用 SWC 的主要原因是速度提升。                                                                                 |
| Release-It             | 我们使用 [release-it](https://github.com/release-it/release-it) 在 npm 上发布我们的包。它负责创建发布的所有繁重工作，并将其发布在 npm 和 GitHub 上。其配置在 `package.json` 文件中定义。                                                                                |
| ESLint                 | ESLint 帮助我们在所有仓库中强制执行一致的编码风格，这些仓库有多个贡献者。我们所有的 ESLint 规则都发布在 [eslint-plugin-adonis](https://github.com/adonisjs-community/eslint-plugin-adonis) 包下。                                                                                   |
| Prettier               | 我们使用 prettier 来格式化代码库，以获得一致的视觉输出。如果您对为什么我们同时使用 ESLint 和 Prettier 感到困惑，请阅读 Prettier 网站上的 [Prettier vs. Linters](https://prettier.io/docs/en/comparison.html) 文档。            |
| EditorConfig           | 每个项目根目录中的 `.editorconfig` 文件配置您的代码编辑器使用一组规则来进行缩进和空白管理。再次强调，Prettier 用于后期格式化您的代码，而 Editorconfig 用于提前配置编辑器。                                                                                     |
| Conventional Changelog | 所有仓库中的所有提交都使用 [commitlint](https://github.com/conventional-changelog/commitlint/#what-is-commitlint) 来强制执行一致的提交信息。                                                                                                                           |
| Husky                  | 我们使用 [husky](https://typicode.github.io/husky/#/) 在提交代码时强制执行提交规范。Husky 是一个用 Node 编写的 git 钩子系统。                                                                                                                                    |

## 命令

| 命令               | 描述                                     |
|-------|--------|
| `npm run test`     | 使用 `ts-node` 运行项目测试               |
| `npm run compile`  | 将 TypeScript 项目编译为 JavaScript。编译后的输出写入 `build` 目录 |
| `npm run release`  | 使用 `np` 开始发布过程                    |
| `npm run lint`     | 使用 ESLint 检查代码库                    |
| `npm run format`   | 使用 Prettier 格式化代码库                |
| `npm run sync-labels` | 将 `.github/labels.json` 文件中定义的标签与 GitHub 同步。此命令仅供项目管理员使用。 |

## 编码风格
我们的所有项目都用 TypeScript 编写，并正在转向纯 ESM。

- 您可以在此处了解[我的编码风格](https://github.com/thetutlage/meta/discussions/3)
- 查看我遵循的 [ESM 和 TypeScript 设置](https://github.com/thetutlage/meta/discussions/2)

此外，在推送代码之前，请确保运行以下命令。

```sh
# 使用 prettier 格式化
npm run format

# 使用 ESLint 检查
npm run lint
```

## 成为认可的贡献者

我们依赖 GitHub 在仓库右侧面板中列出所有仓库贡献者。以下是一个示例。

此外，我们使用 GitHub 的[自动生成发行说明](https://docs.github.com/en/repositories/releasing-projects-on-github/automatically-generated-release-notes#about-automatically-generated-release-notes) 功能，该功能会在发行说明中添加对贡献者个人资料的引用。
