## GitBook 简介

- https://www.gitbook.com

- 一个可以创建、书写、阅读电子书的网站
- 支持将 md 等格式文件导入形成电子书
- 提供 Gitbook CLI（Command Line Interface）工具，基于 Node.js



## Gitbook CLI

- 基于 Node.js

### 安装 node.js

- https://nodejs.org/en/download

由于在使用 gitbook 时候出现如下报错

> gitbook init Error: TypeError [ERR_INVALID_ARG_TYPE]: The “data” argument must be

https://stackoverflow.com/questions/61538769/gitbook-init-error-typeerror-err-invalid-arg-type-the-data-argument-must-b

所以暂且选择 v12.18.1 进行使用

- https://nodejs.org/dist/v12.18.1

下载得到 node-v12.18.1-win-x64.zip 进行解压，可选进行环境变量的配置

验证 node 安装的版本

```
node -v
//输出
v12.18.1 
```
验证 node 自带 npm 安装的版本

```
npm -v
//输出
6.14.5
```

设置国内镜像

```
npm config set registry=http://registry.npm.taobao.org -g
```

### 安装 Gitbook CLI

- https://github.com/GitbookIO/gitbook


```
//-g 代表将模块安装到全局
npm install -g gitbook-cli
```

验证 gitbook 安装的版本

```
//V 要大写
gitbook -V
//输出
CLI version: 2.3.2
GitBook version: 3.2.3
```



初始化书籍

- 执行完目录里会自动生成相关配置文件

```
gitbook init
```

- 如果不存在 README.md 和 SUMMARY.md 这两个文件，会先自动生成
- README.md 撰写书籍的介绍
- SUMMARY.md 撰写书籍的目录结构



构建生成书籍页面

```
gitbook build
```



构建生成书籍页面，并部署到本地服务，供本地预览

```
//部署完服务地址 http://localhost:4000
gitbook serve
//指定端口
gitbook serve --port 4333
```



一些其他命令

```
//列出所有已发布的 gitbook 版本
gitbook ls-remote
//列出本地已安装的 gitbook 版本
gitbook ls
//下载指定版本号的 gitbook
gitbook fetch <gitbook version>
//使用指定的版本进行构建，如果指定版本没有下载到本地会先进行下载
gitbook build --gitbook=<gitbook version>
//删除指定版本号的 gitbook
gitbook uninstall <gitbook version>
```



## 安装 Typora Markdown 编辑器

- https://typora.io

用于直接编辑本地 md 文件



## 安装 SUMMARY 目录自动生成插件

- gitbook-summary https://github.com/imfly/gitbook-summary

```
npm install -g gitbook-summary
```

生成目录

```
book sm
```



目录生成 bug 

- 会出现包含 _book、node_modules 文件夹内的文件的 bug

修改  node-v12.18.1-win-x64\node_modules\gitbook-summary\lib\summary\index.js 文件

```javascript
大概 141 行
把
if (/^[\.]|'_book'|'node_modules'/.test(key)) {
改成
if (/^[\.]\w*|_book|node_modules/.test(key)) {
```



附：

如果使用高版本 node，比如 node-v14.16.1-win-x64 遇到 Installing GitBook 3.2.3 执行失败，报 graceful-fs 相关的错误

```
...
nodejs/lib/node_modules/gitbook-cli/node_modules/npm/node_modules/graceful-fs/polyfills.js:287
      if (cb) cb.apply(this, arguments)
                 ^

TypeError: cb.apply is not a function
...
```

进入 node-v14.16.1-win-x64\node_modules\gitbook-cli\node_modules\npm\node_modules 目录下执行

```
//安装升级到最新版
npm install graceful-fs@latest --save
```



npm 插件搜索

https://www.npmjs.com



GitBook 简明教程http://www.chengweiyang.cn/gitbook/

用Gitbook和Github轻松打造属于自己的出版平台http://self-publishing.ebookchain.org/

Gitbook 学习使用笔记https://zq99299.gitbooks.io/gitbook-guide/content/