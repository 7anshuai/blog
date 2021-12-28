---

title: 通过 Nginx 动态设置 React App 环境变量
date: "2021-12-24T21:00:00.000Z"

---

V2EX 上有V友发帖讨论 [React 在 Docker 部署时，如何动态的读取特定环境下的环境变量](https://v2ex.com/t/824120)。在查阅了 Create React App 关于添加自定义环境变量的文档后，结合 nginx docker image（1.19版本及以上）提供的在 nginx config 中使用环境变量方法。可以通过 nginx 来动态注入环境变量到页面里，达到 React App
运行时动态读取环境变量的效果。

## 自定义环境变量

通过 `create-react-app` 创建的 React App 可以在 HTML 及 JavaScript 文件中使用环境变量。默认可以使用 `NODE_ENV`、`PUBLIC_URL`等[内置环境变量][1]。自定义的变量使用 `REACT_APP_` 开头。在 JavaScript 中使用 `process.env` 来访问内置及自定义的环境变量。HTML 文件中使用 `<title>%REACT_APP_WEBSITE_NAME%</title>` 语法来使用环境变量。

### 从服务器注入页面数据

**环境变量是在编译过程中嵌入到 React 应用中的**。由于编译后生成的是静态 HTML/CSS/JS 文件，它们是无法在运行时读取到环境变量的。要在运行时读取它们，您需要将 HTML 加载到服务器上的内存中并在运行时替换占位符，如[此处][2]所述。例如：

```html
<!doctype html>
<html lang="en">
  <head>
    <script>
      window.SERVER_DATA = __SERVER_DATA__;
    </script>

```

然后，您可以在发送响应之前将 `__SERVER_DATA__` 替换为真实数据的 JSON。然后客户端代码可以读取 `window.SERVER_DATA` 来使用它。**确保在将 JSON 发送到客户端之前对其进行[序列化][3]，因为它会使您的应用程序容易受到 XSS 攻击。**

### 通过 `.env` 文件添加环境变量

在日常的开发中，一般会使用 `.env` 文件来定义环境变量，在项目根目录创建默认的 `.env` 文件：

```bash
REACT_APP_BASE_URL=http://localhost
REACT_APP_API_URL=http://api.localhost
```

可以根据不同环境创建相应的[设置][4]：

- `.env`: 默认
- `.env.local`: 本地覆盖（除测试环境之外都会加载该文件）
- `.env.development`、`.env.test`、`.env.production`: 特定环境的配置
- `.env.development.local`、`.env.test.local`、`.env.production.local`：特定环境配置的本地覆盖

左侧的文件比右侧的文件具有更高的优先级：

- `npm start`: `.env.development.local`, `.env.local`, `.env.development`, `.env`
- `npm run build`: `.env.production.local`, `.env.local`, `.env.production`, `.env`
- `npm test`: `.env.test.local`, `.env.test`, `.env` （注意没有 `.env.local`）

通过不同的 `.env` 文件，可以为开发，测试，生产环境构建不同的 Bundle。如果您使用 Docker Image 来分发，部署 React 应用的话，需要针对不同的环境构建不同的镜像。

## [在 Nginx config 中使用环境变量][5]

Nginx 默认是不支持在绝大多数配置块里使用环境变量的。Docker 官方 Nginx 镜像（1.19 版本及以上）提供了一个功能，会在 nginx 启动前提取环境变量，并写入配置中（具体脚本见 [docker-entrypoint.sh](https://github.com/nginxinc/docker-nginx/blob/master/entrypoint/docker-entrypoint.sh) 及 [20-envsubst-on-templates.sh](https://github.com/nginxinc/docker-nginx/blob/master/entrypoint/20-envsubst-on-templates.sh)）。

以 docker-compose.yml 为例：

```dockerfile
web:
  image: nginx
  volumes:
   - ./templates:/etc/nginx/templates
  ports:
   - "8080:80"
  environment:
   - NGINX_HOST=foobar.com
   - NGINX_PORT=80
```

自动配置脚本默认读取 `/etc/nginx/templates/*.template` 中的模板文件，并将执行 `envsubst` 命令的结果输出到 `/etc/nginx/conf.d` 目录里。

因此，如果在项目添加 `templates/default.conf.template` 文件，其中包含这样的环境变量引用：

```bash
listen ${NGINX_PORT};
```

输出到 `/etc/nginx/conf.d/default.conf` 则像这样：

```bash
listen 80;
```

## Build once, run anywhere

现在，我们有了一个方案来实现 React App 运行时动态加载环境变量：通过 nginx 做一个简易的服务端渲染（SSR），使用 `sub_filter` 指令注入环境变量到 `index.html` 文件中。实现 React App 构建一次，运行在不同环境下。

基本实现步骤如下：

- 在项目根目录添加 `.env.development`

```bash
REACT_APP_API_URL=http://api.localhost
```

- 在 `public/index.html` 文件中添加环境变量占位符，例如：

```html
<!doctype html>
<html lang="en">
  <head>
    <script>
      window.app = {
        apiUrl: "%REACT_APP_API_URL%"
      };
    </script>

```

- 在 React 应用中使用 `window.app.apiUrl`
- 开发完成后，使用 Docker 构建镜像, `Dockerfile` 示例如下:

```Dockerfile
FROM node:14-alpine AS builder
ENV NODE_ENV production
# Add a work directory
WORKDIR /app
# Cache and Install dependencies
COPY package.json .
COPY yarn.lock .
RUN yarn install --production
# Copy app files
COPY . .
# Build the app
RUN yarn build

# Bundle static assets with nginx
FROM nginx:1.21.0-alpine as production
ENV NODE_ENV production
# Copy built assets from builder
COPY --from=builder /app/build /usr/share/nginx/html
# Add your nginx config template
COPY default.conf.template /etc/nginx/templates/default.conf.template
# Expose port
EXPOSE 80
# Start nginx
CMD ["nginx", "-g", "daemon off;"]
```

Nginx config 模板文件 `default.conf.template` 示例：

```nginx
server {
  listen 80;

  location / {
    root /usr/share/nginx/html/;
    include /etc/nginx/mime.types;
    try_files $uri $uri/ /index.html;
    sub_filter '%REACT_APP_API_URL%' '${REACT_APP_API_URL}';
  }
}
```

- 部署时，将相应的环境变量传递给 React App 容器

```bash
docker run -e REACT_APP_API_URL=https://api.example.com --name your-react-app -d -p 8080:80 your-react-app-image
```

- 检查页面：`curl -i http://localhost:8080/`

```bash
$ curl -i http://localhost:8080/
HTTP/1.1 200 OK
Server: nginx/1.21.4
Date: Sat, 25 Dec 2021 10:28:15 GMT
Content-Type: text/html
Transfer-Encoding: chunked
Connection: keep-alive

<!doctype html><html lang="en"><head><meta charset="utf-8"/><link rel="icon" href="/favicon.ico"/><meta name="viewport" content="width=device-width,initial-scale=1"/><meta name="theme-color" content="#000000"/><meta name="description" content="Web site created using create-react-app"/><link rel="apple-touch-icon" href="/logo192.png"/><link rel="manifest" href="/manifest.json"/><title>React App</title><script>window.app={apiUrl:"https://api.example.com"}</script><script defer="defer" src="/static/js/main.b115a3e9.js"></script><link href="/static/css/main.073c9b0a.css" rel="stylesheet"></head><body><noscript>You need to enable JavaScript to run this app.</noscript><div id="root"></div></body></html>
```

**Done!** [GitHub Repo][6]


[1]: https://create-react-app.dev/docs/advanced-configuration "内置环境变量"
[2]: https://create-react-app.dev/docs/title-and-meta-tags#injecting-data-from-the-server-into-the-page "服务器注入数据到页面"
[3]: https://medium.com/node-security/the-most-common-xss-vulnerability-in-react-js-applications-2bdffbcc1fa0 "JSON 序列化"
[4]: https://create-react-app.dev/docs/adding-custom-environment-variables/#what-other-env-files-can-be-used "设置"
[5]: https://hub.docker.com/_/nginx "在 nginx config 中使用环境变量"
[6]: https://github.com/7anshuai/react-app-env-vars-example "查看示例代码"
