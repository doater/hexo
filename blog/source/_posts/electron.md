---
title: "Node构建桌面应用教程"
date: 2017-01-13 17:30:20
categories: node编程
tags: 桌面应用
---

{% blockquote %}
    Electron 提供了一个实时构建桌面应用的纯 JavaScript 环境。Electron 可以获取到你定义在 package.json 中 main 文件内容，然后执行它。通过这个文件（通常我们称之为main.js），可以创建一个应用窗口，这个应用窗口包含一个渲染好的 web 界面，还可以和系统原生的 GUI 交互。
{% endblockquote%}


{% blockquote %}
    直接启动main.js是无法显示应用窗口的,在main.js调用BrowserWindow模块才能使用窗口。每个窗口都将执行属于自己的渲染进程。渲染进程处理的是一个正真的web页面"HTML+CSS+JavaScript"。前端人员也可以用web的形式开发桌面应用啦。
{% endblockquote %}

{% asset_img tree.png electron工作示意图%}

# 开始electron编程
## 目录结构

{% blockquote %}
    * app 项目目录
        * index.js index页面的渲染进程
        * list.js  list页面的渲染进程
        * statics  静态资源目录
    * main.js 主进程
    * package.json 
{% endblockquote %}

## 在项目文件夹中建造package.json文件，内容如下
```
    {
        "name": "yao",
        "version": "0.1.0",
        "main": "./main.js",
        "scripts": {
            "start": "electron ."
        }
    }

```

## 安装electron
```
    npm install --save-dev electron

```
## 创建入口程序main.js

### 引入项目依赖，ES6写法
{% blockquote %}
   BrowserWindow管理窗口,子窗口可以向ipcMain发送事件，达到窗口间通信的效果
{% endblockquote %}

```
    const {app, BrowserWindow , ipcMain} = require('electron')
    const path = require('path')
    const url = require('url')
    //窗口全局变量
    let win

```

### 创建主窗口

```
   function createWindow () {
     // 创建窗口
     win = new BrowserWindow({width: 800, height: 600})

     // 加载本地app的html
     win.loadURL(url.format({
       pathname: path.join(__dirname, 'index.html'),
       protocol: 'file:',
       slashes: true
     }))

     // 打开控制台
     win.webContents.openDevTools()

     // 绑定函数，当窗口关闭时触发
     win.on('closed', () => {
       win = null
     })
   }

```

### 绑定事件

```
    //当app加载完即启用回调函数
    app.on('ready', createWindow)

    //当所有窗口关闭启用回调函数
    app.on('window-all-closed', () => {
      if (process.platform !== 'darwin') {
        app.quit()
      }
    })

    //当app活跃时，启动回调
    app.on('activate', () => {
      if (win === null) {
        createWindow()
      }
    })


```
## 创建页面

### index.html

```
    <!DOCTYPE html>
    <html>
      <head>
        <meta charset="UTF-8">
        <title>gulp打包</title>
        <link rel="stylesheet" type="text/css" href="statics/styles/reset.css">
        <link rel="stylesheet" type="text/css" href="statics/styles/index.css">
      </head>
      <body>
      <h1>hello word</h1>
      <button>open list</button>
      </body>
      <script>
        //加载jquery
        window.$ = window.jQuery = require('./statics/scripts/jquery-1.11.2.min.js');
      </script>
      <script>
        // 加载管理自己渲染进程的js
        require('./index.js')
      </script>
    </html>

```
### list.html

```
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>List消息列表</title>
        <link rel="stylesheet" type="text/css" href="statics/styles/reset.css">
        <link rel="stylesheet" type="text/css" href="statics/styles/list.css">
    </head>
    <body>
        <div class="list_content">
           <span></span>
        </div>
    </body>
    <script>
        //加载jquery
        window.$ = window.jQuery = require('./statics/scripts/jquery-1.11.2.min.js');
    </script>
    <script>
        //家贼管理自己渲染进程的js 
        require('./list.js')
    </script>
    </html>

```


# electron页面之间的消息传递

## index.js

``` 
    const {
        ipcRenderer
    } = require('electron');

    var msg='i am yao'

    // 点击按钮,打开List列表
    // 向主进程传递open-list-window事件
    $('button').on('click', () => {
        ipcRenderer.send('open-list-window', msg);
    })

```

## main.js添加事件监听 open-list-window事件
```
    //子窗口
    let listWindow

    //监听渲染进程发过来的事件，执行回调 ,子窗口被打开

    ipcMain.on('open-list-window', (event, arg) => {
      if (listWindow) {
        return;
      }
      listWindow = new BrowserWindow({
        height: 500,
        width: 500,
        resizable: false
      })
      listWindow.loadURL(url.format({
        pathname: path.join(__dirname, 'app/list.html'),
        protocol: 'file:',
        slashes: true
      }))
      listWindow.webContents.openDevTools();
      listWindow.on('closed', () => {
        listWindow = null;
      })
    })

```
## list.js添加事件监听 hello事件
```
    //子窗口发送消息给主进程

    const {ipcRenderer}=require('electron');

    ipcRenderer.send('hello','i am 子窗口')

```
## main.js添加代码

```
    ipcMain.on('hello', (event, arg) => {
        console.log(arg)
    })

```
{% blockquote %}
   控制台最后打印出i am 子窗口。
{% endblockquote %}

# electron展望

{% blockquote %}
   electron主进程可以执行node的exec模块，调用系统命令,可以通过web页面构建良好的用户交互体验，通过点击按钮，执行一些系统命令。

   比如: 建造一个项目， 利用localstorage去记录文件，选择项目文件，复制改文件到app中的source中，安装依赖，执行app里面的打包脚本，最后输出到文件目录中dist处。可以构建一个类似weflow的打包桌面应用。

   目前正在进行中,敬请期待。
{% endblockquote %}