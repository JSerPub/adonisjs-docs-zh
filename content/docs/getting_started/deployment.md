---
summary: 了解在生产环境中部署 AdonisJS 应用程序的一般指南。
---

# 部署

部署 AdonisJS 应用程序与部署标准 Node.js 应用程序没有区别。您需要一台运行 `Node.js >= 20.6` 的服务器，以及 `npm` 来安装生产依赖项。

本指南将介绍在生产环境中部署和运行 AdonisJS 应用程序的一般指南。

## 创建生产构建

作为第一步，您必须通过运行 `build` 命令来创建 AdonisJS 应用程序的生产构建。

另请参阅：[TypeScript 构建过程](../concepts/typescript_build_process.md)

```sh
node ace build
```

编译输出将写入 `./build` 目录。如果您使用 Vite，其输出将写入 `./build/public` 目录。

创建生产构建后，您可以将 `./build` 文件夹复制到生产服务器。**从现在开始，构建文件夹将是您应用程序的根目录**。

### 创建 Docker 镜像

如果您使用 Docker 来部署应用程序，可以使用以下 `Dockerfile` 创建一个 Docker 镜像。

```dockerfile
FROM node:20.12.2-alpine3.18 AS base

# 所有依赖阶段
FROM base AS deps
WORKDIR /app
ADD package.json package-lock.json ./
RUN npm ci

# 仅生产依赖阶段
FROM base AS production-deps
WORKDIR /app
ADD package.json package-lock.json ./
RUN npm ci --omit=dev

# 构建阶段
FROM base AS build
WORKDIR /app
COPY --from=deps /app/node_modules /app/node_modules
ADD . .
RUN node ace build

# 生产阶段
FROM base
ENV NODE_ENV=production
WORKDIR /app
COPY --from=production-deps /app/node_modules /app/node_modules
COPY --from=build /app/build /app
EXPOSE 8080
CMD ["node", "./bin/server.js"]
```

请根据需要修改 Dockerfile。

## 配置反向代理

Node.js 应用程序通常部署在像 Nginx 这样的反向代理服务器后面。因此，端口 `80` 和 `443` 上的传入流量将由 Nginx 首先处理，然后转发到您的 Node.js 应用程序。

以下是一个 Nginx 配置文件示例，您可以将其作为基础。

:::warning
请确保替换尖括号 `<>` 内的值。
:::

```nginx
server {
  listen 80;
  listen [::]:80;

  server_name <APP_DOMAIN.COM>;

  location / {
    proxy_pass http://localhost:<ADONIS_PORT>;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_cache_bypass $http_upgrade;
  }
}
```

## 定义环境变量

如果您在裸机服务器（如 DigitalOcean Droplet 或 EC2 实例）上部署应用程序，可以使用 `.env` 文件来定义环境变量。请确保安全存储该文件，并且只有授权用户可以访问它。

:::note
如果您使用 Heroku 或 Cleavr 等部署平台，可以使用它们的控制面板来定义环境变量。
:::

假设您已在 `/etc/secrets` 目录中创建了 `.env` 文件，您必须以如下方式启动生产服务器。

```sh
ENV_PATH=/etc/secrets node build/bin/server.js
```

`ENV_PATH` 环境变量指示 AdonisJS 在指定目录中查找 `.env` 文件。

## 启动生产服务器

您可以通过运行 `node server.js` 文件来启动生产服务器。但是，建议使用像 [pm2](https://pm2.keymetrics.io/docs/usage/quick-start) 这样的进程管理器。

- PM2 将在后台运行您的应用程序，而不会阻塞当前终端会话。
- 如果您的应用程序在处理请求时崩溃，它将重新启动应用程序。
- 此外，PM2 使在 [集群模式](https://nodejs.org/api/cluster.html#cluster) 下运行您的应用程序变得非常简单。

以下是一个示例 [pm2 生态系统文件](https://pm2.keymetrics.io/docs/usage/application-declaration)，您可以将其作为基础。

```js
// title: ecosystem.config.js
module.exports = {
  apps: [
    {
      name: 'web-app',
      script: './server.js',
      instances: 'max',
      exec_mode: 'cluster',
      autorestart: true,
    },
  ],
}
```

```sh
// title: 启动服务器
pm2 start ecosystem.config.js
```

## 迁移数据库

如果您使用 SQL 数据库，必须在生产服务器上运行数据库迁移以创建所需的表。

如果您使用 Lucid，可以运行以下命令。

```sh
node ace migration:run --force
```

在生产环境中运行迁移时，需要 `--force` 标志。

### 何时运行迁移

此外，最好在重新启动服务器之前运行迁移。如果迁移失败，请不要重新启动服务器。

使用像 Cleavr 或 Heroku 这样的托管服务时，它们可以自动处理这种情况。否则，您需要在 CI/CD 管道中运行迁移脚本，或者通过 SSH 手动运行它。

### 不要在生产环境中回滚

在生产环境中回滚迁移是一项风险操作。迁移文件中的 `down` 方法通常包含破坏性操作，如 **删除表** 或 **删除列** 等。

建议在 `config/database.ts` 文件中禁用生产环境中的回滚，而是创建一个新的迁移来解决问题并在生产服务器上运行它。

在生产环境中禁用回滚将确保 `node ace migration:rollback` 命令会导致错误。

```js
{
  pg: {
    client: 'pg',
    migrations: {
      disableRollbacksInProduction: true,
    }
  }
}
```

### 并发迁移

如果您在多实例服务器上运行迁移，必须确保只有一个实例运行迁移。

对于 MySQL 和 PostgreSQL，Lucid 将获得咨询锁以确保不允许并发迁移。但是，最好从一开始就避免从多个服务器运行迁移。

## 文件上传的持久存储

像 Amazon EKS、Google Kubernetes、Heroku、DigitalOcean Apps 等环境在 [临时文件系统](https://devcenter.heroku.com/articles/dynos#ephemeral-filesystem) 中运行您的应用程序代码，这意味着每次部署时，默认情况下会销毁现有文件系统并创建一个新的。

如果您的应用程序允许用户上传文件，必须使用像 Amazon S3、Google Cloud Storage 或 DigitalOcean Spaces 这样的持久存储服务，而不是依赖本地文件系统。

## 写入日志

AdonisJS 默认使用 [`pino` 日志器](../digging_deeper/logger.md)，以 JSON 格式将日志写入控制台。您可以设置一个外部日志服务从 stdout/stderr 读取日志，或者将它们转发到同一服务器上的本地文件。

## 提供静态资源

有效地提供静态资源对于应用程序的性能至关重要。无论您的 AdonisJS 应用程序速度有多快，静态资源的交付对于提升用户体验都起着至关重要的作用。

### 使用 CDN

提供静态资源的最佳方法是使用 CDN（内容分发网络）。

使用 [Vite](../basics/vite.md) 编译的前端资源默认会带有版本哈希，这意味着文件名会根据其内容进行哈希处理。这允许您永久缓存资源并从 CDN 提供它们。

根据您使用的 CDN 服务和部署技术，您可能需要在部署过程中添加一个步骤，将静态文件移动到 CDN 服务器。这是大致的工作流程。

1. 更新 `vite.config.js` 和 `config/vite.ts` 配置以 [使用 CDN URL](../basics/vite.md#deploying-assets-to-a-cdn)。

2. 运行 `build` 命令以编译应用程序和资产。

3. 将 `public/assets` 的输出复制到 CDN 服务器。例如，[这是我们用于将资产发布到 Amazon S3 存储桶的命令](https://github.com/adonisjs-community/polls-app/blob/main/commands/PublishAssets.ts)。

### 使用 Nginx 提供静态资源

另一个选择是将提供资产的任务卸载给 Nginx。如果您使用 Vite 编译前端资产，必须积极缓存所有静态文件，因为它们都带有版本哈希。

在 Nginx 配置文件中添加以下块。**请确保替换尖括号 `<>` 内的值**。

```nginx
location ~ \.(jpg|png|css|js|gif|ico|woff|woff2) {
  root <PATH_TO_ADONISJS_APP_PUBLIC_DIRECTORY>;
  sendfile on;
  sendfile_max_chunk 2m;
  add_header Cache-Control "public";
  expires 365d;
}
```

### 使用 AdonisJS 内置静态文件服务器

您也可以依靠 [AdonisJS 内置的静态文件服务器](../basics/static_file_server.md) 从 `public` 目录提供静态资源，以保持简洁。

无需进行额外配置。只需像往常一样部署您的 AdonisJS 应用程序，静态资源的请求就会自动得到处理。

:::warning
不建议在生产环境中使用静态文件服务器。最好使用 CDN 或 Nginx 来提供静态资源。
:::
