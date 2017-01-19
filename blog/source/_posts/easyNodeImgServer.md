---
title: Node图片服务器的简易实现
date: 2017-01-12 16:18:49
categories: node编程
tags: 服务器
---
# http商业服务器遵循的原则
高扩展,低耦合
{% asset_img http.jpg 商业服务器 %}

[demo下载](https://github.com/yaotiancheng123456/demo)

## Quick Start

```
"安装formidable模块"

npm install formidable --save-dev

```
### 创建一个入口文件 index.js

```
var server=require('./server.js');

server.start();  

```
### 创建文件 server.js,创键服务器

```
var http = require('http');
var router=require('./router.js');

function start(){
    function onRequest(request,response){
        router.route(request,response);
    }

    http.createServer(function(request,response){
        onRequest(request,response);
    }).listen(8080,function(){
        console.log('server start on port 8080');
    });
}

exports.start=start;

```

### 创键文件route.js，负责路由分发

```
    var requestHandler = require('./requestHandler.js');
    var url = require('url');
    var handle = requestHandler.handle;

    function route(request, response) {
        var pathname = url.parse(request.url).pathname;
        if (typeof(handle[pathname]) === 'function') {
            console.log('recieve a handle ' + pathname);
            handle[pathname](request, response);
        } else {
            response.writeHead(404, {
                'content-type': 'text/plain'
            });
            response.write('Not Found');
            response.end();
        }
    }
    exports.route = route;

```
### 创键文件requestHandler.js,集中管理相关操作

```
var startHandler=require('./startHandler');
var uploadHandler=require('./uploadHandler');
var showHandler=require('./showHandler');
var handle={};
handle['/']=startHandler.start;
handle['/upload']=uploadHandler.upload;
handle['/show']=showHandler.show;
exports.handle=handle;

```

### 创键startHandler.js,具体负责start路由下，服务器的操作

```
var fs=require('fs');

function start(request,response){
    var body=fs.readFileSync('./post.html');
    response.writeHead(200,{
        'content-type':'text/html'
    });
    response.write(body);
    response.end();
}

exports.start=start;

```

### 创键uploadHandler.js,具体负责上传路由,服务器的操作

```
var formidable = require('formidable');
var fs=require('fs');

function upload(request, response) {
    var form = new formidable.IncomingForm();
    form.uploadDir='pubilc/upload/';
    form.parse(request, function(err, fields, files) {
        fs.renameSync(files.myFile.path,'tmp/test.png');
        response.writeHead(200, {
            'content-type': 'text/html'
        });
        response.write('received upload:<br/>');
        response.write('<img src="/show"/>');
        response.end();
    });
}

exports.upload = upload;

```
### 创键showHandler.js,负责显示图片

```
var fs = require('fs');

function show(request, response) {
    fs.readFile('tmp/test.png', 'binary', function(error, file) {
        if (error) {
            response.writeHead(500, {
                'content-type': 'text/plain'
            });
            console.log(error);
            response.write('500 服务器内部错误');
            response.end();
        } else {
            response.writeHead(200, {
                'content-type': 'image/png'
            });
            response.write(file, "binary");
            response.end();
        }
    })
}
exports.show = show;

```
