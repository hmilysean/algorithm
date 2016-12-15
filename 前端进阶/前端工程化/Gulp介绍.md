## gulp实践

### gulp 简介

地址： [http://gulpjs.com/](http://gulpjs.com/)

*   它是一款nodejs应用。
*   它是打造前端工作流的利器，打包、压缩、合并、git、远程操作...，
*   简单易用
*   无快不破
*   高质量的插件
*   ....

### gulp 安装及常见问题

#### 1\. 安装gulp

```
    npm install -g gulp

```

如果报`Error: EACCES, open '/Users/xxx/xxx.lock`错误。先执行：`sudo chown -R $(whoami) $HOME/.npm`;

如果使用npm安装插件太慢（被墙），可执行 `npm install -g cnpm --registry=https://registry.npm.taobao.org`,先安装cnpm, 之后再安装插件时用cnpm安装`cnpm install gulp`

#### 2\. 安装各种插件

```
    npm install --save gulp            //本地使用gulp
    npm install --save gulp-imagemin   //压缩图片
    npm install --save gulp-minify-css //压缩css
    npm install --save gulp-ruby-sass  //sass
    npm install --save gulp-jshint     //js代码检测
    npm install --save gulp-uglify     //js压缩
    npm install --save gulp-concat     //文件合并
    npm install --save gulp-rename     //文件重命名
    npm install --save png-sprite      //png合并
    npm install --save gulp-htmlmin    //压缩html
    npm install --save gulp-clean      //清空文件夹
    npm install --save browser-sync    //文件修改浏览器自动刷新
    npm install --save gulp-shell      //执行shell命令
    npm install --save gulp-ssh        //操作远程机器
    npm install --save run-sequence    //task顺序执行

```

或者根据package.json 自动安装。把package.json拷贝到自己的工程目录下，进入目录，执行:`npm install`

### gulp 使用案例

所有代码在：[https://coding.net/u/jirengu/p/gulp/git](https://coding.net/u/jirengu/p/gulp/git)

#### 范例1:

demo1目录结构如下。把demo1中的 index.html压缩，把src里面的less编译、合并、压缩、重命名、存储到dist。src里面的图片压缩、合并存储到dist。src里面的js做代码检查，压缩，合并，存储到dist。

```

    + demo1
        + dist
            + css
                - merge.min.css
            + js
                - merge.min.js
            + imgs
                - 1.png
                - 2.png
            - index.html
        + src
            + css
                - a.css
                - b.css
            + js
                - a.js
                - b.js
            + imgs
                - 1.png
                - 2.png
            - index.html

```

gulpfile.js

```

    var gulp = require('gulp');

    // 引入组件
    var minifycss = require('gulp-minify-css'), // CSS压缩
        uglify = require('gulp-uglify'), // js压缩
        concat = require('gulp-concat'), // 合并文件
        rename = require('gulp-rename'), // 重命名
        clean = require('gulp-clean'), //清空文件夹
        minhtml = require('gulp-htmlmin'), //html压缩
        jshint = require('gulp-jshint'), //js代码规范性检查
        imagemin = require('gulp-imagemin'); //图片压缩

    gulp.task('html', function() {
      return gulp.src('src/*.html')
        .pipe(minhtml({collapseWhitespace: true}))
        .pipe(gulp.dest('dist'));
    });

    gulp.task('css', function(argument) {
        gulp.src('src/css/*.css')
            .pipe(concat('merge.css'))
            .pipe(rename({
                suffix: '.min'
            }))
            .pipe(minifycss())
            .pipe(gulp.dest('dist/css/'));
    });
    gulp.task('js', function(argument) {
        gulp.src('src/js/*.js')
            .pipe(jshint())
            .pipe(jshint.reporter('default'))
            .pipe(concat('merge.js'))
            .pipe(rename({
                suffix: '.min'
            }))
            .pipe(uglify())
            .pipe(gulp.dest('dist/js/'));
    });

    gulp.task('img', function(argument){
        gulp.src('src/imgs/*')
            .pipe(imagemin())
            .pipe(gulp.dest('dist/imgs'));
    });

    gulp.task('clear', function(){
        gulp.src('dist/*',{read: false})
            .pipe(clean());
    });

    gulp.task('build', ['html', 'css', 'js', 'img']);

```

执行：

```
    gulp build;

```

可实现src目录下的html压缩，css、js合并压缩，图片压缩，最后放入 dist目录下

#### 范例2:

监控项目文件变动，自动刷新浏览器，本地调试， 并且把本地代码同步到远程服务器

```

    var gulp = require('gulp');

    // 引入组件
    var browserSync = require('browser-sync').create();   //用于浏览器自动刷新
    var scp = require('gulp-scp2');                       //用于scp到远程机器
    var fs = require('fs');             

    gulp.task('reload', function(){
        browserSync.reload();
    });

    gulp.task('server', function() {
        browserSync.init({
            server: {
                baseDir: "./src"
            }
        });

        gulp.watch(['**/*.css', '**/*.js', '**/*.html'], ['reload', 'scp']);
    });

    gulp.task('scp', function() {
        return gulp.src('src/**/*')
            .pipe(scp({
                host: '121.40.201.213',
                username: 'root',
                privateKey: fs.readFileSync('/Users/wingo/.ssh/id_rsa'),
                dest: '/var/www/fe.jirengu.com',
                watch: function(client) {
                    client.on('write', function(o) {
                        console.log('write %s', o.destination);
                    });
                }
            }))
            .on('error', function(err) {
                console.log(err);
            });
    });

```

执行：

```
    gulp scp;  // 可把本地开发环境代码拷贝到服务器
    gulp server; //可在本地创建服务器，本地开发浏览器立刻刷新

```

#### 范例3:

监控项目文件变动，自动压缩、合并、打包、添加版本号

```
html

    </html>
    <head>

    <!-- build:css css/merge.css -->
    <link href="css/a.css" rel="stylesheet">
    <link href="css/b.css" rel="stylesheet">
    <!-- endbuild -->

    </head>
    <body>

    <p>demo1-工程化手动版</p>

    <!-- build:js js/merge.js -->
    <script type="text/javascript" src="js/a.js"></script>
    <script type="text/javascript" src="js/b.js"></script>
    <!-- endbuild -->

    </body>
    </html>

```

gulpfile.js

```
    var gulp = require('gulp');    
    var rev = require('gulp-rev');   //添加版本号
    var revReplace = require('gulp-rev-replace');  //版本号替换
    var useref = require('gulp-useref');   //解析html资源定位
    var filter = require('gulp-filter');   //过滤数据
    var uglify = require('gulp-uglify');  
    var csso = require('gulp-csso');      //css优化压缩
    var clean = require('gulp-clean');

    gulp.task("index", ['clear'], function() {
      var jsFilter = filter("**/*.js", {restore: true});
      var cssFilter = filter("**/*.css", {restore: true});

      var userefAssets = useref.assets();

      return gulp.src("src/index.html")
        .pipe(userefAssets)      // Concatenate with gulp-useref
        .pipe(jsFilter)
        .pipe(uglify())             // Minify any javascript sources
        .pipe(jsFilter.restore)
        .pipe(cssFilter)
        .pipe(csso())               // Minify any CSS sources
        .pipe(cssFilter.restore)
        .pipe(rev())                // Rename the concatenated files
        .pipe(userefAssets.restore())
        .pipe(useref())
        .pipe(revReplace())         // Substitute in new filenames
        .pipe(gulp.dest('dist'));
    });

    gulp.task('clear', function(){
        gulp.src('dist/*',{read: false})
            .pipe(clean());
    });

```

#### 范例4:

本地shell命令， 远程shell， 任务顺序执行...

```

    var gulp = require('gulp');

    var shell = require('gulp-shell');
    var runSequence = require('run-sequence');
    var fs = require('fs');
    var GulpSSH = require('gulp-ssh');

    //shell操作, 
    gulp.task('git', shell.task(['git add .', 'git commit -am "dd"', 'git push -u origin dev']));
    gulp.task('clear', shell.task(['find . -name ".DS_Store" -depth -exec rm {} \\;']));

    //操作远程主机
    var gulpSSH = new GulpSSH({
        ignoreErrors: false,
        sshConfig: {
            host: '121.40.201.213',
            port: 22,
            username: 'root',
            privateKey: fs.readFileSync('/Users/wingo/.ssh/id_rsa')
        }
    });

    gulp.task('remote', function() {
        return gulpSSH
            .shell(['cd /var/www/fe.jirengu.com', 'git pull origin dev', 'rm -rf _runtime']);
    });

    gulp.task('build', function(callback) {
        runSequence(
            'git',
            'clear',
            'remote',
            callback
        );
    });

    gulp.task('watch', function() {
        gulp.watch(['**/*.css', '**/*.js', '**/*.html', '**/*.php'], ['build']);
    });

```