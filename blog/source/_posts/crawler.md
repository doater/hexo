---
title: 爬虫简单示例
date: 2017-01-12 16:17:48
categories: node编程
tags: 爬虫
---
```
/*
 *   one:
 *  目标站点:动脑论坛
 *  目标数据:会员名
 *   two:
 *  目标站点:起点小说
 *  目标数据:玄幻小说
 *  three: 
 *  目标站点:http://www.mm131.com/
 *  目标数据:美女图片 
 *  作者:yao
 *  时间:2017.1.11
 */

var cheerio = require('cheerio');
var request = require('request');
var fs = require('fs');
var mkdirp = require('mkdirp')


// charpter one 抓取会员名, output: name.txt
request('http://101.200.129.112:4567/users/latest', function(error, response, body) {
    if (!error && response.statusCode == 200) {
        var $ = cheerio.load(body);
        var items = $('#users-container .user-info a');
        var names = [];
        items.each(function() {
            names.push($(this).text());
        })
        fs.writeFile('name.txt', names.join(','));
        console.log('会员名抓取完毕');
    }
})

// charpter two 抓取小说, output: xiaoshuo.txt
function getContent(url) {
    request(url, function(error, response, body) {
        if (!error && response.statusCode == 200) {
            var $ = cheerio.load(body);
            var str = '';
            var next = $('#j_chapterNext').attr('href');
            next = 'http:' + next;
            str += $('.read-content').text();
            fs.appendFile('xiaoshuo.txt', str);
            getContent(next);
        } else {
            console.log('抓取完毕');
        }
    })
}
var url = 'http://read.qidian.com/chapter/OOqJX-uOLbLmkXioLmMPXw2/fwup5h1VDGlMs5iq0oQwLQ2';
getContent(url);


//创建目录
mkdirp('./images', function(err) {
    if(err){
        console.log(err);
    }
});
// charpter three 抓取美女图片,output:images文件夹
function getImages(url, base) {
    request(url, function(error, response, body) {
        if (!error && response.statusCode == 200) {
            var $ = cheerio.load(body);
            var items = $('.public-box dd a img');
            var $next = $('.page-en').eq(-2);
            nextUrl = base + $next.attr('href');
            items.each(function() {
                var src = $(this).attr('src');
                if (/pic/.test(src) > 0) {
                    download(src, './images', Math.floor(Math.random() * 1000000000) + src.split('/').pop());
                    console.log('正在下载' + src);
                }
            })
            if (isNaN($next.text())) {
                getImages(nextUrl, base);
            } else {
                console.log('抓取完毕');
            }
        }
    })
}

function download(url, dir, filename) {
    request.head(url, function(err, res, body) {
        request(url).pipe(fs.createWriteStream(dir + '/' + filename));
    })
}

getImages('http://www.mm131.com/qingchun/', 'http://www.mm131.com/qingchun/');

```