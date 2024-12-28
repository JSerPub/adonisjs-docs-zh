---
summary: 了解 AdonisJS 为 TypeScript 、 Prettier 和 ESLint 所使用的工具配置预设。
---

# 工具配置

AdonisJS 在很大程度上依赖 TypeScript 、 Prettier 和 ESLint 来确保代码的一致性，在构建时检查错误，更重要的是，获得令人愉悦的开发体验。

我们已将所有配置选项提炼到可供所有官方包和官方入门套件使用的即用型配置预设中。

如果您想在使用TypeScript编写的Node.js应用程序中使用相同的配置预设，请继续阅读本指南。

## TSConfig（TypeScript配置）

[`@adonisjs/tsconfig`](https://github.com/adonisjs/tooling-config/tree/main/packages/typescript-config) 包包含了TypeScript项目的基础配置。我们将TypeScript模块系统设置为“NodeNext”，并使用“TS Node + SWC”进行即时编译。

可随意查看[基础配置文件](https://github.com/adonisjs/tooling-config/blob/main/packages/typescript-config/tsconfig.base.json)、[应用程序配置文件](https://github.com/adonisjs/tooling-config/blob/main/packages/typescript-config/tsconfig.app.json)以及[包开发配置文件](https://github.com/adonisjs/tooling-config/blob/main/packages/typescript-config/tsconfig.package.json)中的选项。

您可以按以下方式安装该包并使用它。

```sh
npm i -D @adonisjs/tsconfig

# 确保同时安装以下包
npm i -D typescript ts-node @swc/core
```

在创建AdonisJS应用程序时，从`tsconfig.app.json`文件进行扩展。（入门套件中已预先配置）。

```jsonc
{
  "extends": "@adonisjs/tsconfig/tsconfig.app.json",
  "compilerOptions": {
    "rootDir": "./",
    "outDir": "./build"
  }
}
```

在为AdonisJS生态系统创建包时，从`tsconfig.package.json`文件进行扩展。

```jsonc
{
  "extends": "@adonisjs/tsconfig/tsconfig.package.json",
  "compilerOptions": {
    "rootDir": "./",
    "outDir": "./build"
  }
}
```

## Prettier配置

[`@adonisjs/prettier-config`](https://github.com/adonisjs/tooling-config/tree/main/packages/prettier-config) 包包含了用于自动格式化源代码以保持样式一致的基础配置。可随意查看[`index.json`文件](https://github.com/adonisjs/tooling-config/blob/main/packages/prettier-config/index.json)中的配置选项。

您可以按以下方式安装该包并使用它。

```sh
npm i -D @adonisjs/prettier-config

# 确保同时安装Prettier
npm i -D prettier
```

在`package.json`文件中定义以下属性。

```jsonc
{
  "prettier": "@adonisjs/prettier-config"
}
```

此外，创建一个`.prettierignore`文件来忽略特定的文件和目录。

```
// title:.prettierignore
build
node_modules
```

## ESLint配置

[`@adonisjs/eslint-config`](https://github.com/adonisjs/tooling-config/tree/main/packages/eslint-config) 包包含了应用代码检查规则的基础配置。可随意查看[基础配置文件](https://github.com/adonisjs/tooling-config/blob/main/packages/eslint-config/presets/ts_base.js)、[应用程序配置文件](https://github.com/adonisjs/tooling-config/blob/main/packages/eslint-config/presets/ts_app.js)以及[包开发配置文件](https://github.com/adonisjs/tooling-config/blob/main/packages/eslint-config/presets/ts_package.js)中的选项。

您可以按以下方式安装该包并使用它。

:::note

我们的配置预设使用了 [eslint-plugin-prettier](https://github.com/prettier/eslint-plugin-prettier) 来确保ESLint和Prettier能够协同工作而互不干扰。

:::

```sh
npm i -D @adonisjs/eslint-config

# 确保同时安装ESLint
npm i -D eslint
```

在创建AdonisJS应用程序时，从`eslint-config/app`文件进行扩展。（入门套件中已预先配置）。

```json
// title: package.json
{
  "eslintConfig": {
    "extends": "@adonisjs/eslint-config/app"
  }
}
```

在为AdonisJS生态系统创建包时，从`eslint-config/package`文件进行扩展。

```json
// title: package.json
{
  "eslintConfig": {
    "extends": "@adonisjs/eslint-config/package"
  }
}
``` 