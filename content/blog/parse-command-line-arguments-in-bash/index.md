---
title: 在 Bash 中解析命令行参数
date: "2016-04-22T00:00:00.000Z"
---

最近的一个前端小项目是智能 Wi-Fi 音箱 [Sugr Cube](http://sugrsugr.com) 中的 Web 上传歌曲界面，使用了 Require.js 组织代码，文件上传部分基于 [jQuery File Upload](https://github.com/blueimp/jQuery-File-Upload)。Web page 编写完后需要使用 r.js 打包处理下，并将生成的文件上传到硬件设备里的 [Nweb](https://github.com/ankushagarwal/nweb) 目录下。

## 项目结构
```
|--node_modules
|  |-- blueimp-file-upload-node
|  |  本地文件上传服务器
|  |-- requirejs
|  |  requirejs optimizer (r.js)
|  |-- inliner
|  |  Node utility to inline images, CSS and JavaScript for a web page
|--public
|  静态文件（css，js，images等）
|-- package.json
|  npm 项目配置
|-- index.html
```

## npm scripts
编写 [npm scripts](https://docs.npmjs.com/misc/scripts) 用来运行相关脚本：
```json
{
  ...,
  "scripts": {
    "build": "r.js -o baseUrl=public/js paths.requireLib=require paths.jquery=jquery name=app include=requireLib out=public/js/app-built.js",
    "inliner": "inliner -vs index.html > www/index.html",
    "server": "node node_modules/blueimp-file-upload-node/server.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  ...
}
```

OK，页面开发完后运行相应的命令 `npm run build`，`npm run inliner` 之后，再将生成好的单一 HTML 文件（www/index.html）scp 到硬件设备的文件 www 目录下。那么问题来了，调试过程中需要频繁的重复这一过程，而且硬件设备的局域网 IP 地址也常会发生变化，我需要个一键部署的 shell 脚本，比如这样：
```
./publish --user root --host 192.168.1.99
```

## Bash script
如何编写一个能接受命令行参数的 Bash 脚本？在 stackoverflow 上找到大家推荐的方法：使用没有 getopt[s] 的 straight bash。

### 空格分离的 Straight Bash

使用方法：`./myscript.sh -e conf -s /etc -l /usr/lib /etc/hosts`
```bash
#!/bin/bash
# Use -gt 1 to consume two arguments per pass in the loop (e.g. each
# argument has a corresponding value to go with it).
# Use -gt 0 to consume one or more arguments per pass in the loop (e.g.
# some arguments don't have a corresponding value to go with it such
# as in the --default example).
# note: if this is set to -gt 0 the /etc/hosts part is not recognized ( may be a bug )
while [[ $# -gt 1 ]]
do
key="$1"

case $key in
    -e|--extension)
    EXTENSION="$2"
    shift # past argument
    ;;
    -s|--searchpath)
    SEARCHPATH="$2"
    shift # past argument
    ;;
    -l|--lib)
    LIBPATH="$2"
    shift # past argument
    ;;
    --default)
    DEFAULT=YES
    ;;
    *)
            # unknown option
    ;;
esac
shift # past argument or value
done
echo FILE EXTENSION  = "${EXTENSION}"
echo SEARCH PATH     = "${SEARCHPATH}"
echo LIBRARY PATH    = "${LIBPATH}"
echo "Number files in SEARCH PATH with EXTENSION:" $(ls -1 "${SEARCHPATH}"/*."${EXTENSION}" | wc -l)
if [[ -n $1 ]]; then
    echo "Last line of file specified as non-opt/last argument:"
    tail -1 $1
fi
```

### 等号分离的 Straight Bash
使用方法：`./myscript.sh -e=conf -s=/etc -l=/usr/lib /etc/hosts`
```bash
#!/bin/bash

for i in "$@"
do
case $i in
    -e=*|--extension=*)
    EXTENSION="${i#*=}"
    shift # past argument=value
    ;;
    -s=*|--searchpath=*)
    SEARCHPATH="${i#*=}"
    shift # past argument=value
    ;;
    -l=*|--lib=*)
    LIBPATH="${i#*=}"
    shift # past argument=value
    ;;
    --default)
    DEFAULT=YES
    shift # past argument with no value
    ;;
    *)
            # unknown option
    ;;
esac
done
echo "FILE EXTENSION  = ${EXTENSION}"
echo "SEARCH PATH     = ${SEARCHPATH}"
echo "LIBRARY PATH    = ${LIBPATH}"
echo "Number files in SEARCH PATH with EXTENSION:" $(ls -1 "${SEARCHPATH}"/*."${EXTENSION}" | wc -l)
if [[ -n $1 ]]; then
    echo "Last line of file specified as non-opt/last argument:"
    tail -1 $1
fi
```

为了更好的理解 `${i#*=}` 可在[这篇指南](http://tldp.org/LDP/abs/html/string-manipulation.html)中查找 "Substring Removal"。它的功能等同于 `sed 's/[^=]*=//' <<< "$i"`（调用了一个不必要的子进程）或者 `echo "$i" | sed 's/[^=]*=//'`（调用了两个不必要的子进程）。

参考链接：
- [A quick guide to writing scripts using the bash shell](http://www.panix.com/~elflord/unix/bash-tute.html)
- [How do I parse command line arguments in bash?](http://stackoverflow.com/questions/192249/how-do-i-parse-command-line-arguments-in-bash)
