---
title: "node-sass安装失败"
date: 2019-03-22 17:30:20
categories: 异常处理
tags: debug
---

{% blockquote %}
    有sass的项目进行npm install,结果报错如下图所示
{% endblockquote%}

{% asset_img nodeSass.png %}


# 解决方法
## 第一步,清除npm缓存

```
    npm cache clean -f
```

## 第二步,删除node_modules,运行命令
```
    npm install
```

## 第三步,重构node-sass
```
    npm rebuild node-sass

```