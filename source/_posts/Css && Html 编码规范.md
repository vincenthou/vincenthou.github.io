Css && Html Code GuideLine
 ==========================

**基于以前项目的css书写不规范导致的浏览器兼容问题和多人协作混乱问题，特此制定Css && Html Code GuideLine，所有参与项目css文件编写的成员请务必参看**

**注解：**此GuideLine基于douban和google网上公开的规范，并且加入了一些适用于多人协作的内容整理而成，项目整体css编码风格和思想借鉴了twitter's bootstrap框架,适用但不局限于IN-BIKE项目
1. 项目必须有library,即静态示范页，里面包括了项目里所有可能用到的的html结构和css，library应与实际项目最终输出的html保持一致，并作为bug重现/修复参考，如有bug，先看library和当前项目有无冲突，再联系相关人员。
2. Library负责人管理HTML结构搭建和基础CSS组建，当开发人员发现自己的页面/模块最终输出与Library不一致时，请与此负责人联系。
3. 全局CSS文件global.css、reset.css、username.css文件都位于css文件夹根目录，严禁另外创建文件夹存放css文件（防止目录混乱）
4. 如需使用插件，css文件请存放在lib/pluginname/css,与站点css分开放置
5. global.css中包含的是整站的基础模块，由一人撰写和维护（library.html 和 global.css）,其他开发人员禁止修改这两个文件，建议通过svn权限进行控制
6. 所有参与写样式的开发人员都需要在css文件夹根目录中添加一个以自己英文名命名的css文件，并且联系global.css负责人将其Import进 global.css，
   当涉及到global没有提供且与自己相关的页面/模块，就将CSS样式写在其中，但是要注意使用mod写法不要影响到他人的模块,如果是公有的，请联系负责人在global.css中添加(@import只是针对多人协作的临时方式，上线时会merge公共部分，其余会按需加载)；
7. 所有CSS样式至少兼容浏览器IE7+,Opera,firefox3.0+,Safir,Chrome，原则是渐进增强平稳退化，即标准浏览器实现完整交互和视觉效果，非标准浏览器至少保留基本功能
8. 使用html5 doctype (<!DOCTYPE html> <html lang="en"> ... </html>)
9. reset.css文件使用yahoo YUI reset
10. 所有css命名均使用中划线
11. css属性推荐使用一行式写法，便于查找，这也是google和douban推荐的写法（不习惯的同学也可以暂时选择多行式写法）
12. 全局不允许使用通配符 *；
13. 为防止兼容混乱，暂时不推荐使用css3的选择器，如子选择器>,:firstchild,nth-child,last-child,属性选择器[type="xxx"]等
14. 遵循模块化思想，模块命名遵循以下结构:

``` css
.xxx-module .ooo{property:value;}
.bread-module{property:value;}
.bread-module .hd{}//hd代表head
.bread-module .hd .operate{}
.bread-module .bd{}//bd代表body
.bread-module .ft{}//ft代表foot
```
1. 以下几个css属性慎用，经测试，会导致页面性能下降
   position:fixed
   background-position: fixed
   border-radius
   background-size
   box-shadow
   gradient
2. 每个模块前加注释，格式 /\* modulename */
3. 清除浮动使用clearfix,避免使用多余标签
4.  严禁使用css表达式（css expression)
5. 因需求变动等各种原因导致的无用的css内容/文件请及时删除
6. tips:
   另外给出几个常用的css命名及解释

``` css
.hd // header
.bd // body
.ft // foot
.operate // 用在编辑和操作
.pic //图片包裹
.info // 内容
.nav //导航
.active //激活项（用于选项卡，面包屑，其他高亮栏目）
.tab //选项卡
```

不允许如下样式出现

``` css
.tips{xxx:ooo;} //错误示例,污染全局
*{xxx:ooo;}//错误示例，全局禁用通配符
```
