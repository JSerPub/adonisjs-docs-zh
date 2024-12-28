---
summary: 了解在生产环境中部署 AdonisJS 应用程序的一般指南。
---

# Deployment

Deploying an AdonisJS application is no different than deploying a standard Node.js application. You need a server running `Node.js >= 20.6` along with `npm` to install production dependencies.

部署AdonisJS应用程序与部署标准的Node.js应用程序并无不同。你需要一台运行 `Node.js >= 20.6` 且安装了 `npm` 的服务器来安装生产依赖项。

This guide will cover the generic guidelines to deploy and run an AdonisJS application in production. 

本指南将涵盖在生产环境中部署和运行 AdonisJS 应用程序的通用准则。

## 创建生产构建

As a first step, you must create the production build of your AdonisJS application by running the `build` command.

作为第一步，你必须通过运行 `build` 命令来创建 AdonisJS 应用程序的生产构建。

See also: [TypeScript build process](../concepts/typescript_build_process.md)

另请参阅：[TypeScript构建过程](../concepts/typescript_build_process.md)

```sh
node ace build
```

The compiled output is written inside the `./build` directory. If you use Vite, its output will be written inside the `./build/public` directory.

编译后的输出会写入`./build`目录中。如果你使用 Vite，其输出将写入 `./build/public` 目录中。

Once you have created the production build, you may copy the `./build` folder to your production server. **From now on, the build folder will be the root of your application**.

一旦创建好生产构建，你可以将`./build`文件夹复制到生产服务器上。**从现在起，构建文件夹将成为你应用程序的根目录**。

### 创建 Docker 镜像

If you are using Docker to deploy your application, you may create a Docker image using the following `Dockerfile`.

如果你使用Docker来部署应用程序，可以使用以下`Dockerfile`创建Docker镜像。

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

# 
FROM base AS build
WORKDIR /app
COPY --from=deps /app/node_modules /app/node_modules
ADD . .
RUN node ace build

# 
FROM base
ENV NODE_ENV=production
WORKDIR /app
COPY --from=production-deps /app/node_modules /app/node_modules
COPY --from=build /app/build /app
EXPOSE 8080
CMD ["node", "./bin/server.js"]
```

Feel free to modify the Dockerfile to suit your needs.

可根据自身需求随意修改 `Dockerfile`。

## 配置反向代理

Node.js applications are usually [deployed behind a reverse proxy](https://medium.com/intrinsic-blog/why-should-i-use-a-reverse-proxy-if-node-js-is-production-ready-5a079408b2ca) server like Nginx. So the incoming traffic on ports `80` and `443` will be handled by Nginx first and then forwarded to your Node.js application.

Node.js 应用程序通常部署在像 Nginx 这样的反向代理服务器之后（[参考此处](https://medium.com/intrinsic-blog/why-should-i-use-a-reverse-proxy-if-node-js-is-production-ready-5a079408b2ca)）。因此，端口 `80` 和 `443` 上的传入流量将首先由 Nginx 处理，然后转发到你的 Node.js 应用程序。

Following is an example Nginx config file you may use as the starting point.

以下是一个可供你作为起始参考的 Nginx 配置文件示例。

:::warning
Make sure to replace the values inside the angle brackets `<>`.

务必替换尖括号`<>`内的值。
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

If you are deploying your application on a bare-bone server, like a DigitalOcean Droplet or an EC2 instance, you may use an `.env` file to define the environment variables. Ensure the file is stored securely and only authorized users can access it.

如果你将应用程序部署在裸机服务器上，比如 DigitalOcean Droplet 或 EC2 实例，你可以使用 `.env` 文件来定义环境变量。确保该文件安全存储，且只有授权用户能够访问。

:::note
If you are using a deployment platform like Heroku or Cleavr, you may use their control panel to define the environment variables.

如果你使用像 Heroku 或 Cleavr这 样的部署平台，可以使用它们的控制面板来定义环境变量。

:::

Assuming you have created the `.env` file in an `/etc/secrets` directory, you must start your production server as follows.

假设你已在 `/etc/secrets` 目录中创建了 `.env` 文件，你必须按如下方式启动生产服务器。

```sh
ENV_PATH=/etc/secrets node build/bin/server.js
```

The `ENV_PATH` environment variable instructs AdonisJS to look for the `.env` file inside the mentioned directory.

`ENV_PATH` 这个环境变量指示 AdonisJS 在上述提到的目录中查找`.env`文件。

## 启动生产服务器

You may start the production server by running the `node server.js` file. However, it is recommended to use a process manager like [pm2](https://pm2.keymetrics.io/docs/usage/quick-start).

- PM2 will run your application in background without blocking the current terminal session.
- It will restart the application, if your app crashes while serving requests.
- Also, PM2 makes it super simple to run your application in [cluster mode](https://nodejs.org/api/cluster.html#cluster)


你可以通过运行 `node server.js` 文件来启动生产服务器。不过，建议使用像[pm2](https://pm2.keymetrics.io/docs/usage/quick-start)这样的进程管理器。

- PM2会在后台运行你的应用程序，不会阻塞当前终端会话。
- 如果你的应用程序在处理请求时崩溃，它会重启应用程序。
- 而且，PM2使得在[集群模式](https://nodejs.org/api/cluster.html#cluster)下运行应用程序变得极其简单。

Following is an example [pm2 ecosystem file](https://pm2.keymetrics.io/docs/usage/application-declaration) you may use as the starting point.

以下是一个可供你作为起始参考的[pm2生态系统文件](https://pm2.keymetrics.io/docs/usage/application-declaration)示例。

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
// title: Start server
pm2 start ecosystem.config.js
```

## 迁移数据库

If you are using a SQL database, you must run the database migrations on the production server to create the required tables.

如果你使用的是 SQL 数据库，必须在生产服务器上运行数据库迁移来创建所需的表。

If you are using Lucid, you may run the following command.

如果你使用 Lucid，可以运行以下命令。

```sh
node ace migration:run --force
```

The `--force` flag is required when running migrations in the production environment.

在生产环境中运行迁移时，需要使用`--force`标志。

### 何时运行迁移

Also, it would be best if you always run the migrations before restarting the server. Then, if the migration fails, do not restart the server.

此外，最好在重启服务器之前运行迁移。如果迁移失败，不要重启服务器。

Using a managed service like Cleavr or Heroku, they can automatically handle this use case. Otherwise, you will have to run the migration script inside a CI/CD pipeline or run it manually through SSH.

使用像 Cleavr 或 Heroku 这样的托管服务，它们可以自动处理这种情况。否则，你将不得不在持续集成/持续部署（CI/CD）管道中运行迁移脚本，或者通过SSH手动运行它。

### 不要在生产环境中回滚

Rolling back migrations in production is a risky operation. The `down` method in your migration files usually contains destructive actions like **drop the table**, or **remove a column**, and so on.

在生产环境中回滚迁移是一项有风险的操作。迁移文件中的 `down` 方法通常包含诸如 **删除表** 、 **移除列** 等破坏性操作。

It is recommended to turn off rollbacks in production inside the config/database.ts file and instead, create a new migration to fix the issue and run it on the production server.

建议在`config/database.ts`文件中关闭生产环境中的回滚功能，而是创建一个新的迁移来解决问题，并在生产服务器上运行它。

Disabling rollbacks in production will ensure that the `node ace migration:rollback` command results in an error.

在生产环境中禁用回滚功能将确保 `node ace migration:rollback` 命令会报错。

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

### 并行迁移

If you are running migrations on a server with multiple instances, you must ensure that only one instance runs the migrations.

如果你在具有多个实例的服务器上运行迁移，必须确保只有一个实例运行迁移。

For MySQL and PostgreSQL, Lucid will obtain advisory locks to ensure that concurrent migration is not allowed. However, it is better to avoid running migrations from multiple servers in the first place.

对于MySQL和PostgreSQL，Lucid将获取咨询锁以确保不允许并行迁移。不过，最好从一开始就避免从多个服务器运行迁移。

## 文件上传的持久化存储

Environments like Amazon EKS, Google Kubernetes, Heroku, DigitalOcean Apps, and so on, run your application code inside [an ephemeral filesystem](https://devcenter.heroku.com/articles/dynos#ephemeral-filesystem), which means that each deployment, by default, will nuke the existing filesystem and creates a fresh one.

像亚马逊EKS、谷歌Kubernetes、Heroku、DigitalOcean Apps等环境，会在[临时文件系统](https://devcenter.heroku.com/articles/dynos#ephemeral-filesystem)中运行你的应用程序代码，这意味着默认情况下，每次部署都会清除现有的文件系统并创建一个新的文件系统。

If your application allows users to upload files, you must use a persistent storage service like Amazon S3, Google Cloud Storage, or DigitalOcean Spaces instead of relying on the local filesystem.

如果你的应用程序允许用户上传文件，必须使用像亚马逊S3、谷歌云存储或DigitalOcean Spaces这样的持久化存储服务，而不是依赖本地文件系统。

## 记录日志

AdonisJS uses the [`pino` logger](../digging_deeper/logger.md) by default, which writes logs to the console in JSON format. You can either set up an external logging service to read the logs from stdout/stderr, or forward them to a local file on the same server.

AdonisJS默认使用[`pino`日志记录器](../digging_deeper/logger.md)，它以JSON格式将日志写入控制台。你可以设置一个外部日志记录服务来从标准输出/标准错误读取日志，或者将它们转发到同一服务器上的本地文件中。

## 提供静态资源服务

Serving static assets effectively is essential for the performance of your application. Regardless of how fast your AdonisJS applications are, the delivery of static assets plays a massive role to a better user experience.

有效地提供静态资源服务对于应用程序的性能至关重要。无论你的 AdonisJS 应用程序有多快，静态资源的传输对于提升用户体验都起着巨大的作用。

### 使用 CDN

The best approach is to use a CDN (Content Delivery Network) for delivering the static assets from your AdonisJS application.

最佳方法是使用内容分发网络（CDN）来提供 AdonisJS 应用程序的静态资源。

The frontend assets compiled using [Vite](../basics/vite.md) are fingerprinted by default, which means that the file names are hashed based on their content. This allows you to cache the assets forever and serve them from a CDN.

使用[Vite](../basics/vite.md)编译的前端资源默认带有指纹识别，这意味着文件名会根据其内容进行哈希处理。这使得你可以永久缓存资源并从CDN提供服务。

Depending upon the CDN service you are using and your deployment technique, you may have to add a step to your deployment process to move the static files to the CDN server. This is how it should work in a nutshell.


1. Update the `vite.config.js` and `config/vite.ts` configuration to [use the CDN URL](../basics/vite.md#deploying-assets-to-a-cdn).

2. Run the `build` command to compile the application and the assets.

3. Copy the output of `public/assets` to your CDN server. For example, [here is a command](https://github.com/adonisjs-community/polls-app/blob/main/commands/PublishAssets.ts) we use to publish the assets to an Amazon S3 bucket.

根据你使用的CDN服务和部署技术，可能需要在部署过程中添加一个步骤，将静态文件移动到 CDN 服务器上。大致的工作流程如下：

1. 更新 `vite.config.js` 和 `config/vite.ts` 配置，以[使用CDN URL](../basics/vite.md#deploying-assets-to-a-cdn)。

2. 运行 `build` 命令来编译应用程序和资源。

3. 将 `public/assets` 的输出复制到你的CDN服务器上。例如，[这里有一个命令](https://github.com/adonisjs-community/polls-app/blob/main/commands/PublishAssets.ts)，我们用它将资源发布到亚马逊S3存储桶中。

### 使用Nginx提供静态资源服务

Another option is to offload the task of serving assets to Nginx. If you use Vite to compile the front-end assets, you must aggressively cache all the static files since they are fingerprinted.

另一个选择是将提供资源的任务交给Nginx。如果你使用Vite编译前端资源，由于资源带有指纹识别，你必须积极缓存所有静态文件。

Add the following block to your Nginx config file. **Make sure to replace the values inside the angle brackets `<>`**.

在你的 Nginx 配置文件中添加以下代码块。**务必替换尖括号`<>`内的值**。

```nginx
location ~ \.(jpg|png|css|js|gif|ico|woff|woff2) {
  root <PATH_TO_ADONISJS_APP_PUBLIC_DIRECTORY>;
  sendfile on;
  sendfile_max_chunk 2m;
  add_header Cache-Control "public";
  expires 365d;
}
```

### 使用AdonisJS内置静态文件服务器

You can also rely on the [AdonisJS inbuilt static file server](../basics/static_file_server.md) to serve the static assets from the `public` directory to keep things simple.

你也可以依靠[AdonisJS内置静态文件服务器](../basics/static_file_server.md)从 `public` 目录提供静态资源服务，这样做比较简单。

No additional configuration is required. Just deploy your AdonisJS application as usual, and the request for static assets will be served automatically.

无需额外配置。只需像往常一样部署 AdonisJS 应用程序，对静态资源的请求就会自动得到处理。

:::warn
The static file server is not recommended for production use. It is best to use a CDN or Nginx to serve static assets.

不建议在生产环境中使用静态文件服务器。最好使用 CDN 或 Nginx 来提供静态资源服务。
:::
