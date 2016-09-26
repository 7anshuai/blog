# 通过 Nginx 给本地应用取个漂亮域名
- pubdate: 2016-09-19
- tags: nginx, shell

-------

简单记录下之前看到并实践的一篇文章 [Serving Apps Locally with Nginx and Pretty Domains](http://zaiste.net/2013/03/serving_apps_locally_with_nginx_and_pretty_domains/)。在 Mac OS X 上通过配置 Nginx 实现本地应用可以通过漂亮的域名来访问，比如 `http://anapp.dev/`。类似的解决方案有 [pow](http://pow.cx) - Mac OS X 上的零配置 Rake Server。

## Nginx

首先需要关掉 Apache 进程（Mac OS X 上默认启动 Apache 监听 `80` 端口）：
```bash
sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist
```

使用 Homebrew 安装 Nginx ：
```bash
brew install nginx
```

Nginx 监听 `80`（或任何小于 `1024` 的）端口需要使用 `sudo` 命令，否则会启动失败。对于大于 `1024` 的端口，如下直接为启动脚本建立一个符号链接：
```bash
ln -sfv /usr/local/opt/nginx/*.plist ~/Library/LaunchAgents
launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.nginx.plist
```

对于 HTTP 默认端口，需要在 `/usr/local/etc/nginx.conf` 中将 `listen` 的值从 `8080` 修改为 `80`。
```
server {
     …
     listen 80;
     server_name localhost;
     …
}
```

小于 `1024` 的端口不能为启动脚本建立符号链接，必须将脚本拷贝到 `/Library/LaunchAgents`。
```bash
sudo cp /usr/local/opt/nginx/homebrew.mxcl.nginx.plist /Library/LaunchAgents
```

在 `homebrew.mxcl.nginx.plist` 中，需要将 `UserName` 项修改为 `root`。为了方便，还可以将 `Label` 项修改为 `nginx`，这样就可以使用
```bash
launchctl start nginx
```

代替
```bash
launchctl start homebrew.mxcl.nginx
```

## 本地 DNS
接下来是设置一个本地的 DNS。因为不能在 `/etc/hosts` 文件中使用通配符，无法实现类似功能：
```
127.0.0.1      *.dev.
```

为了解决这个问题，需要安装一个叫做 [DNSMasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) 的 DNS 代理：
```bash
brew install dnsmasq
```

配置文件存储在 `/usr/local/etc/` 下的 `dnsmasq.conf` ：
```bash
touch /usr/local/etc/dnsmasq.conf
```

在文件中写入：
```
address=/.dev/127.0.0.1
```

这样所有 `*.dev` 的站点会被重定向到本地 IP，即 `127.0.0.1`。
类似 Nginx 进程，`dnsmasq` 需要 `root` 权限：
```bash
sudo cp -fv /usr/local/opt/dnsmasq/*.plist /Library/LaunchDaemons
sudo launchctl load /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist
```

然后，需要配置 OSX 使用本地系统作为首要 DNS 服务器。进入系统设置 -> 网络，在 DNS 配置中将回环 IP (即 `127.0.0.1`)作为第一行，然后是惯例的 DNS IP：
```
127.0.0.1
8.8.8.8
8.8.4.4
```

现在，试着 `ping` 任何以 `.dev` 结尾的地址，应该返回的 IP 地址是 `127.0.0.1`：
```bash
$ ping example.dev 
PING example.dev (127.0.0.1): 56 data bytes
```

## 虚拟主机
关于虚拟主机配置，按照惯例在 `/usr/local/etc/nginx/` 下创建两个目录 `sites-enabled` 和 `sites-available`：
```bash
cd /usr/local/etc/nginx
mkdir sites-available
mkdir sites-enabled
```

在 `nginx.conf` 中的 `http` 部分，需要加入以下行：
```
include sites-enabled/*.dev;
```

### 后端参与的项目配置
现在可以指定每一个 app 的配置了。看一下配置模板文件：
```
upstream NAME {
    server 127.0.0.1:3000;
}

server {
    listen 80;
    server_name NAME.dev;
    root PATH_TO_PUBLIC;

    try_files $uri/index.html $uri.html $uri @app;

    location @app {
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host $http_host;
      proxy_redirect off;

      proxy_pass http://NAME;
    }
}
```

为了使它工作起来，至少需要修改两个地方，即 `NAME` 和 `PATH_TO_PUBLIC`。 `NAME` 可以是应用程序名称。 `PATH_TO_PUBLIC` 则指定项目静态资源目录，例如在 Express 中的路径为 `public`。
配置文件需要放在 `sites-available` 下，然后需要链接到 `sites-enabled`，例如：
```bash
ln -s /usr/local/etc/nginx/sites-available/anapp.dev \
  /usr/local/etc/nginx/sites-enabled/anapp.dev
```

建立链接后，需要重启 Nginx：
```bash
sudo launchctl stop nginx
sudo launchctl start nginx
```

### 无后端参与的项目配置
以上的配置文件对于纯静态的项目来说是不必要的。可以通过位于 `/usr/local/etc/nginx/nginx.conf` 中默认的 `server` 指令设置一个动态的应用程序分发。Nginx 会在定义的基础路径中查找匹配被请求的域名目录。如在以下例子中， `appname.dev` 会匹配 `/Users/zaiste/dev` 下一个叫做 `appname` 的目录：
```
server {
    listen       80;
    server_name  app localhost .dev;

    set $basepath "/Users/zaiste/dev";

    set $domain $host;
    if ($domain ~ "^(.*)\.dev$") {
        set $domain $1;
    }
    set $rootpath "${domain}";
    if (-d $basepath/$domain/public) {
        set $rootpath "${domain}/public";
    }
    if (-f $basepath/$domain/index.html) {
        set $rootpath $domain;
    }

    root $basepath/$rootpath;

    # redirect server error pages to the static page /50x.html
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
```
现在，只需要在 `/Users/zaiste/dev` 创建一个新的目录及相应的 HTML 文件，剩下的事情就交给 Nginx 了。

参考链接：
- [Serving Apps Locally with Nginx and Pretty Domains](http://zaiste.net/2013/03/serving_apps_locally_with_nginx_and_pretty_domains/)