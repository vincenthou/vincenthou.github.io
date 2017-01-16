![image](http://gruntjs.cn/img/grunt-logo.png)
为了保证代码的质量和自动化一些重复的工作，构建工具是不必可少的。C/C++程序通过makefile管理编译测试打包的过程，Java程序通过Maven,Ant实现项目构建管理功能，Ruby有Rake，这些工具都是为后端语言设计的。

之前如果需要一些简单的前端构建，可以[使用Ant](http://book.36ria.com/ant/index.html)来实现一些基础的比如打包js/css、压缩文件、构建目录、重命名文件，但对于前端开发来说还不够完善。随着前端开发的复杂度逐渐提高，node.js的推广，基于javascript的构建工具应运而生，这就是Grunt。Grunt可以执行像压缩，合并，模糊化代码， 单元测试,，代码检查以及打包发布等等任务。
# 开始安装使用
## 1. 安装CLI

安装CLI之前首先安装Nodejs和npm，安装成功之后

``` bash
npm install -g grunt-cli
```

**Tip:**  CLI只是grunt的引导加载器，可以使多个版本的grunt并存
## 2. 安装Grunt

``` bash
npm install -g grunt
```
## 3. 在项目根目录添加npm项目配置文件

“package.json” 是npm项目配置文件，使用命令行交互的方式创建pacakge.json文件

``` bash
npm init
```

命令行会提示输入配置信息，或者直接复制下面的内容到一个空的package.json文件中

``` json
{
  "name": "my-project-name",
  "version": "0.1.0",
  "devDependencies": {
    "grunt": "~0.4.2",
    "grunt-contrib-jshint": "~0.6.3",
    "grunt-contrib-nodeunit": "~0.2.0",
    "grunt-contrib-uglify": "~0.2.2"
  }
}
```

文件中的devDependencies选项需要通过安装才能真正生效，例如安装grunt

``` bash
npm install grunt --save-dev
```

--save-dev选项会将grunt配置信息添加到package.json文件中
## 4. 在项目根目录添加grunt配置文件

"Gruntfile.js"是专门用来配置grunt的配置文件（还可以添加Gruntfile.coffee）
"Gruntfile.js"主要由以下几部分组成
- 包装函数
- 项目和任务的配置
- 加载插件和任务
- 自定义任务
  下面一个gruntfile的例子使用uglify来压缩文件，并且添加固定格式的注释信息

``` javascript
//包装函数
module.exports = function(grunt) {

  // 项目配置
  grunt.initConfig({
    //读取npm配置文件信息
    pkg: grunt.file.readJSON('package.json'),
    uglify: {
      options: {
        banner: '/*! <%= pkg.name %> <%= grunt.template.today("yyyy-mm-dd") %> */\n'
      },
      build: {
        src: 'src/<%= pkg.name %>.js',
        dest: 'build/<%= pkg.name %>.min.js'
      }
    }
  });

  // 加载提供“uglify”任务的插件
  grunt.loadNpmTasks('grunt-contrib-uglify');

  // 设置默认任务为“uglify”
  grunt.registerTask('default', ['uglify']);

};
```

**Tip:**  需要创建一个src文件夹，并且添加一个与package.jso里面同名的js文件来进行压缩，从代码中可以看到banner的生成是要读取package.json里面信息的
## 5. 安装需要使用的插件

无论是在package.json里面，还是在Gruntfile中进行配置，这些只是告诉Grunt需要使用的插件和任务是什么，以及使用时传入的参数为何，但是真正的工作的插件还需要自行安装的，通过以下命令来安装

``` bash
npm install grunt-contrib --save-dev
```

**安装成功后，新安装的模块会被添加到node_modules中**
该命令会安装常用的官方工具库，让你能够快速的上手使用grunt（grunt官方的工具库都是以contrib为前缀），在实际开发中要根据需要选择性的下载相应的工具库。
[所有可用的grunt插件库](http://gruntjs.com/plugins)
[常用的grunt插件](https://npmjs.org/package/grunt-contrib)
## 6. 运行grunt命令，开始使用grunt

``` bash
cd project & grunt <task-name>
```

如果不指定任务名称，默认会执行default的任务
# 进一步的深入

[配置任务](http://gruntjs.com/configuring-tasks)
[Grunt API文档](http://gruntjs.com/api/grunt)
[创建自定义任务](http://gruntjs.com/creating-tasks)
**Tip:**  自定义任务放在文件夹中，通过grunt.loadTasks引入
[创建自定义插件](http://gruntjs.cn/creating-plugins/)
[创建项目模板](http://gruntjs.com/project-scaffolding)
# 实际的例子

[入门介绍](http://blog.fens.me/nodejs-grunt-intro/)
[常用插件的使用](http://www.36ria.com/6226)
[任务配置详解](http://www.36ria.com/6232)
