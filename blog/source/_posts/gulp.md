---
title: gulp打包之解耦任务
date: 2017-01-13 16:35:43
categories: 打包
tags:  gulp
---

{% blockquote %}
    如果项目太大,需要拆分任务，可能会导致gulpfile.js体积过大,难以维护。
    为了解决这个问题，应将任务拆分到不同的文件中，即解耦。
{% endblockquote%}

# 安装gulp4.0

{% blockquote %}
    利用gulp.series控制串行任务
    利用gulp.parallel控制并行任务
    更好的利用node高并发的特点，更快的，明确的，进行构建
{% endblockquote%}

```
    npm install gulpjs/gulp#4.0 -g
    npm install gulpjs/gulp#4.0 --save-dev

```

# 创建gulpfile.js

```
    //__dirname为当前目录
    //deep可配置,控制递归的次数
    //这段代码 会去自动加载本目录下的_tasks的index.js,传递gulp
    //执行gulp --tasks可列出所有任务列表

   var gulp=require('gulp');
   var fs=require('fs');
   var path=require('path');

   var deep=3;
   run_tasks('_tasks');
   function run_tasks(tasks_path){
       if (--deep < 0) {
           throw new Error('something wrong in require tasks!');
           return;
       }
       tasks_path=path.resolve(__dirname,tasks_path);
       if (fs.existsSync(tasks_path)) {
           require(tasks_path)(gulp)
       } else {
           run_tasks(tasks_path);
       }
   } 

```

# 创建_tasks文件夹,并在该文件夹中创建index.js

```
    //index.js
    //gulpfile执行即加载了该文件,并执行了改文件
    //当前目录下所有符合命名规则(见正则)会被加载
    //即gulpfile.js依赖了这个文件夹下所有符合命名规则的文件
    var fs = require('fs');
    var path = require('path');

    module.exports = function (gulp) {
        fs.readdirSync(__dirname).filter(function (file) {
            return (file.indexOf(".") !== 0) && (file.indexOf('Task') === 0);
        }).forEach(function (file) {
            var registerTask = require(path.join(__dirname, file));
            registerTask(gulp);
        });
    };


```

# 建立TaskBuildDev.js(示例)
```
    const gulp = require('gulp');
    const spritesmith = require('gulp.spritesmith');
    const sass = require('gulp-sass');
    const moment = require('moment');
    const del = require('del');
    const merge = require('merge-stream');
    const paths = {
        src: {
            dir: './',
            images: ['./web/statics/images/icons/*.png', './web/statics/images/large/*.png', './web/statics/images/login/*.png', './web/statics/images/saas_console/icons/*.png'],
            sass: ['./web/statics/sass/**/*.scss'],
            source: './web/**/*'
        },
        dev: {
            dir: './',
            images: './web/statics/images/',
            sass: './web/statics/sass',
            css: './web/statics/styles',
            sprite: './web/statics/images/icons-*',
        }
    };
    const imageName=formatName('icons.png');
    //格式化名称 版本号
    function formatName(name) {
        var version = moment().format('MMDDHHmm');
        var tmp = name.split('.');
        return tmp[0] + '-' + version + '.' + tmp[1];
    }
    module.exports=function(gulp){
        // 删除老版本雪碧图
        function delSprite() {
            return del(paths.dev.sprite);
        }
        // 编译saas
        function compileSass() {
            return gulp.src(paths.src.sass)
                .pipe(sass().on('error', sass.logError))
                .pipe(gulp.dest(paths.dev.css));
        }
        // 生成雪碧图
        function sprite() {
            var spriteData = gulp.src(paths.src.images)
                .pipe(spritesmith({
                    imgName:imageName,
                    cssName: '_sprite.scss',
                    cssSpritesheetName: 'icon',
                    padding: 5,
                    imgPath:'../images/'+imageName
                }));
            var imgStream = spriteData.img.pipe(gulp.dest(paths.dev.images));
            var cssStream = spriteData.css.pipe(gulp.dest(paths.dev.sass));
            return merge(imgStream, cssStream);
        }
        // 重新生成雪碧图
        gulp.task('icons', gulp.series(
            delSprite,
            sprite
        ));
        // 编译sass
        gulp.task('sass',gulp.series(compileSass));
        // 生成雪碧图 并生成 saas
        gulp.task('build_dev',gulp.series(
            delSprite,
            sprite,
            compileSass
        ));
        // 监听sass
        gulp.task('dev', () => {
            gulp.watch(paths.src.sass, gulp.series(compileSass));
        });
    }
```
#建立TaskBuildWeb.js (示例)

```
    const gulp = require('gulp');
    const spritesmith = require('gulp.spritesmith');
    const sass = require('gulp-sass');
    const moment = require('moment');
    const del = require('del');
    const replace = require('gulp-replace');
    const merge = require('merge-stream');
    const gulpif = require('gulp-if');
    const uglifyjs = require('uglify-js');
    const minifier = require('gulp-uglify/minifier')
    const cleanCSS = require('gulp-clean-css');
    const rename = require('gulp-rename');
    require('gulp-grunt')(gulp);
    const version = moment().format('MMDDHHmm');
    const paths = {
        src: {
            dir: './',
            images: ['./web/statics/images/icons/*.png', './web/statics/images/large/*.png', './web/statics/images/login/*.png', './web/statics/images/saas_console/icons/*.png'],
            source: ['./web/**/*', '!./web/plugins/**/*'],
            modules: './web/modules/**/*',
            sprite: './web/statics/sass/_sprite.scss'
        },
        dist: {
            dir: './dist',
            tpl: ['./dist/web/modules/**/*.tpl', '!./dist/web/modules/page/account/**/*.tpl'],
            cleanmodules: './dist/web/modules/**/*',
            modules: './dist/web/modules',
            base: './dist/web/modules/base/**/*.js',
            css: ['./dist/web/statics/styles/**/*.css', '!./dist/web/statics/**/*.min.css'],
            html: ['./dist/web/*.html'],
            statics: [
                './dist/web/statics/images/icons',
                './dist/web/statics/images/large',
                './dist/web/statics/images/login',
                './dist/web/statics/sass',
                './dist/web/tmp',
                './dist/web/modules/view'
            ],
            others: ['./dist/web/statics/scripts/sha1.js', './dist/web/statics/scripts/sha1_worker.js'],
            gif: './dist/web/statics/sass/_sprite_gif.scss',
            sass: ['./dist/web/statics/sass/**/*.scss'],
            styles: './dist/web/statics/styles'
        },
        tmp: {
            dir: './dist/web/tmp',
            base: './dist/web/tmp/modules/base',
            modules: './dist/web/tmp/modules/**/*'
        }
    };
    module.exports = function(gulp) {
        // 复制modules
        function copySource() {
            return gulp.src(paths.src.source, {
                base: paths.src.dir
            }).pipe(gulp.dest(paths.dist.dir));
        }

        function copyBase() {
            return gulp.src(paths.dist.base)
                .pipe(gulp.dest(paths.tmp.base));
        }


        function replaceTpl() {
            return gulp.src(paths.dist.tpl, {
                    base: paths.dist.dir
                }).pipe(replace(/\\/g, '\\\\'))
                .pipe(gulp.dest(paths.dist.dir))
        }
        // 清除dist文件夹中的文件
        function cleanDist() {
            return del([paths.dist.dir]);
        }
        // 压缩js
        function uglifyJS() {
            return gulp.src(paths.tmp.modules)
                .pipe(minifier({
                    output:{
                        quote_keys:true
                    },
                    compress:{
                        screw_ie8:false
                    }
                }, uglifyjs))
                .pipe(gulp.dest(paths.dist.modules));
        }
        // 压缩其他
        function uglifyOthers() {
            return gulp.src(paths.dist.others, {
                    base: paths.dist.dir
                })
                .pipe(minifier({
                    output:{
                        quote_keys:true
                    },
                    compress:{
                        screw_ie8:false
                    }
                }, uglifyjs))
                .pipe(gulp.dest(paths.dist.dir))
        }
        // 压缩css
        function uglifyCss() {
            return gulp.src(paths.dist.css, {
                    base: paths.dist.dir
                })
                .pipe(cleanCSS())
                .pipe(gulp.dest(paths.dist.dir));
        }
        // 删除dist中没有压缩合并的modules
        function cleanModules() {
            return del([paths.dist.cleanmodules]);
        }
        //删除无用的静态资源
        function cleanStatics() {
            return del(paths.dist.statics);
        }
        //添加版本号
        function addVersion() {
            return gulp.src(paths.dist.html, {
                    base: paths.dist.dir
                })
                .pipe(replace(/@@version/g, version))
                .pipe(gulp.dest(paths.dist.dir));
        }
        // 支持IE
        function createGif() {
            return gulp.src(paths.src.sprite)
                .pipe(replace(/.png/g, '.gif'))
                .pipe(rename('web/statics/sass/_sprite_gif.scss'))
                .pipe(gulp.dest(paths.dist.dir));
        }
        //删除
        function cleanGif() {
            return del(paths.dist.gif);
        }

        function compileSass() {
            return gulp.src(paths.dist.sass)
                .pipe(sass().on('error', sass.logError))
                .pipe(gulp.dest(paths.dist.styles));
        }
        gulp.task('build_web', gulp.series(
            cleanDist,
            gulp.parallel('icons'),
            copySource,
            replaceTpl,
            gulp.parallel('grunt-imagemagick-convert'),
            gulp.parallel('grunt-transport:dist'),
            gulp.parallel('grunt-concat'),
            copyBase,
            cleanModules,
            cleanGif,
            createGif,
            compileSass,
            uglifyJS,
            uglifyOthers,
            uglifyCss,
            cleanStatics,
            addVersion
        ));
        gulp.task('build', gulp.series(
            gulp.parallel('build_web'),
            gulp.parallel('copyWap', 'copyMobile')
        ));
    }

```
{% blockquote %}
    可以将不同功能的任务拆分成多个文件。每个任务都享有自己的私有变量。
    更容易扩展和维护。
{% endblockquote%}