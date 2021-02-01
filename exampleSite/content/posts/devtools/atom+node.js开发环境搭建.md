---
title: Atom+Node.js开发环境的搭建
date: 2018-03-26 21:01:58
tags: ["环境搭建"]
categories: ["环境搭建"]
---
  &emsp;&emsp;最近可能要折腾前后端分离，先试下demo
  <!--more-->
1. node.js环境的安装

  ** windows系统推荐安装.msi版本 **
  &emsp;&emsp;安装版环境变量什么的自动配置，然后更改全局模块路径和缓存的路径
  说明：这里的环境配置主要配置的是npm安装的全局模块所在的路径，以及缓存cache的路径，之所以要配置，是因为以后在执行类似：npm install express [-g] （后面的可选参数-g，g代表global全局安装的意思）的安装语句时，会将安装的模块安装到【C:/Users/用户名/AppData/Roaming/npm】路径中，占C盘空间。
  例如：我希望将全模块所在路径和缓存路径放在我node.js安装的文件夹中，则在我安装的文件夹【D:/Program Files/nodejs】下创建两个文件夹【node_global】及【node_cache】
  * cmd窗口运行下面的语句：
    ```shell
    npm config set cache "D:/Program Files/nodejs/node_cache"
    npm config set prefix "D:/Program Files/nodejs/node_global"
    ```
2. Atom 插件的安装
  ** 必装两个插件：**
    * Script:
    * Atom ternjs

  ** 其他拓展插件请参照：**
    [前端：ATOM环境搭建 + 插件列表](https://blog.csdn.net/hangmine/article/details/78257665)

3. 运行demo
  * 服务端代码
```javascript
var http = require('http');
var fs = require('fs');
var url = require('url');
  // 创建服务器
http.createServer( function (request, response) {  
   // 解析请求，包括文件名
   var pathname = url.parse(request.url).pathname;
   // 输出请求的文件名
   console.log("Request for " + pathname + " received.");
   // 从文件系统中读取请求的文件内容
   fs.readFile(pathname.substr(1), function (err, data) {
      if (err) {
         console.log(err);
         // HTTP 状态码: 404 : NOT FOUND
         // Content Type: text/plain
         response.writeHead(404, {'Content-Type': 'text/html'});
      }else{             
         // HTTP 状态码: 200 : OK
         // Content Type: text/plain
         response.writeHead(200, {'Content-Type': 'text/html'});    

         // 响应文件内容
         response.write(data.toString());        
      }
      //  发送响应数据
      response.end();
    });   
}).listen(8080);
  // 控制台会输出以下信息
console.log('Server running at http://127.0.0.1:8080/');
```
  * 创建一个简单的页面index.html
```html
  <!DOCTYPE html>
<html>
      <head>
        <meta charset="utf-8">
        <title>一个demo页面</title>
      </head>
      <body>
        <h1>这是一个标题</h1>
        <p>这是一个段落。</p>
      </body>
</html>
```
  * 启动
快捷键（`ctrl+shift+B`）,运行的是Atom的`Script`插件，结束命令使用`ctrl+Q`
  * 访问
  地址：localhost:8080/index.html

4. 遇到的问题及解决办法
  * 运行node.js程序的时候，不正常终止程序的话，会导致端口占用：
    解决办法：cmd窗口运行`taskkill /f /t /im node.exe`命令
5. 如果是前端开发，还是推荐开发工具VS Code，Atom用来写个文档还是不错的。
