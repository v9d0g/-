
# 博客重构计划：

本文意在记录闲暇之余学习开发的一些内容。

## 前言：

作为一个业余的开发者，第一次开发的Web系统存在诸多问题，例如：

并发性太差，请求太多就直接崩掉，垃圾请求无法过滤，也没有定时清理日志：

![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/E5316FB6-9B46-4755-AC5F-819E05D9FC381e1ff3e8df16470a.png)

后台管理功能集合太捞：

![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/0DFFD875-0B62-4bf1-A698-B8D2AA45C297ee9b72498b16a4b2.png)

各种不合理、没有任何日志保存和分析的东西，也没有定时备份、文章以markdown格式上传但图片只能依靠图床等等问题。

综上各种问题，目前刚好在了解各种框架和其他语言，可以尝试重构一下博客网站（虽然这个博客网站也才写好几个月）

感觉学Web安全，能深入了解一下Web开发对理解很多这方面的安全能有一定的帮助，业余时间可以学习学习这些技术栈，然后赋予行动。


同时，由于本博客是容器化部署的，我在考虑是否能通过站库分离等一系列操作，构建一个较为复杂的内网环境来学习一些相关知识。

`【2024-03-22更新】`

## 技术栈：

这次重构我想使用一些框架来细化各个部分的功能，暂定：

`前端-vue2、组件库-bootstrap、脚手架-vue-cli`

`数据库-mysql`

`后端-gin`

## 前端：

### 安装

#### 对应版本信息

[nvm](https://github.com/coreybutler/nvm-windows/releases)

```sh
# 版本
nvm -v
1.1.12
```

[nodejs](https://nodejs.org/en) 这个可以通过nvm下载，不需要官网下载。

```sh
# 版本
node -v
10.14.2
# 切换版本
nvm use 10.14.2
```

[yarn](https://blog.csdn.net/jiaoqi6132/article/details/130146199)

```sh
# 版本
yarn -v
1.22.22
```

[vue/cli](https://blog.csdn.net/qq_45156819/article/details/129928007)

```sh
# 下载
yarn global add @vue/cli

# 版本
vue --version
@vue/cli 4.5.17
```

**由于开发环境是Windows，所以需要配置各种环境变量。**

### 创建项目

由于可能会出现版本报错

```sh
error magic-string@0.30.8: The engine "node" is incompatible with this module. Expected version ">=12". Got "10.14.2" info Visit https://yarnpkg.com/en/docs/cli/install for documentation about this command. error Found incompatible module.  ERROR  command
```

可以用下面的指令忽略

```sh
yarn config set ignore-engines true
```

**这里出现了一个问题，各种环境变量配置好后，在cmd里面是可以执行的，但是在Vscode里面无法识别。但由于影响不大，所以并没有解决。**

同时，这里解决了一个我的历史遗留问题：[Vscode下载后更换目录导致无法更新](https://blog.csdn.net/yimenren/article/details/125923232)

### 引入组件库

[BootstrapVue](https://code.z01.com/bootstrap-vue/docs/)

```sh
yarn add bootstrap-vue bootstrap@4.5.3
```

这里可能会存在报错的情况，可以通过降低vue版本来解决

```sh
yarn cache clean #清除缓存
yarn add vue@2.7.14
```

由于是第一次使用vue，它的报错十分奇怪。不符合代码规范就会报错，无法正常显示，现在主要出现的报错是关于空行和分号的错误。这个直接在vscode内部修改格式化的相关信息，统一使用两格作为缩进。

---
`已弃用`

代码高亮
[highlightjs.js](https://highlightjs.org/#usage)

```sh
yarn add highlight.js highlight.js@11.9.0
```

用于前端表单验证

```sh
 yarn add vuelidate
```

用于api请求

```sh
yarn add axios
yarn add vue-axios
```

图标组件
[echarts]()

```sh
npm install echarts
npm i echarts-wordcloud
npm i -S echarts-wordcloud

npm install echarts-wordcloud
```

报错解决

`To install it, you can run: npm install --save core-js/modules/es.array.push.js`

```sh
npm install --save core-js
```

图片放大fancybox
```sh
yarn add @fancyapps/ui
```

---

#### markdown文本处理

最开始想法是后端上传编辑markdown->后端转html->前端显示
后来发现这种方式
`优点`:样式自定义化程度高 可以做一些效果很好的ui显示
`缺点`:只能使用不安全的v-html来渲染前端 并且调整ui会增加前端的渲染压力

markdown前端显示
```sh
yarn add markdown-it-vue
```
然后发现这个组件其实集成了echarts和highlight.js (草了 那我前面花这么多时间 写自定义ui组件在干嘛)

虽然这个组件集成了很多实用的组件 fancybox(图片放大) echarts(图表) highlight(代码高亮) latex(数学公式富文本)等等

但根据作者的issue中提到的 无法自定义代码高亮的风格 而且对markdown渲染的文本没有办法更进一步自定义

不过好在通过引入highlight的样式和修改一下代码高亮的style可以实现对应效果
```vue
import 'highlight.js/styles/atom-one-dark.css'; // 引入highlight.js的atom-one-dark样式

.markdown-body .highlight pre,
.markdown-body pre {
  padding: 16px;
  overflow: auto;
  font-size: 85%;
  line-height: 1.45;
  background-color: #282c34;
  border-radius: 3px;
}

/* 字体高亮 */
code {
  font-size: 100%;
  color: #ffffff;
  word-wrap: break-word;
}

.markdown-body code {
  padding: .2em .4em;
  margin: 0;
  font-size: 85%;
  background-color: rgb(15 45 74);
  border-radius: 3px;
}

/* 链接 */
.markdown-body a {
  color: #53b151;
  text-decoration: none;
}
```

这个库中对目录列表的生成存在问题 原计划是使用`[TOC]`来指定markdown中生成目录结构 但是该组件不能正常的生成目录结构 于是需要自己设计算法



## 后端

### 配置

设置GOROOT，GOROOT为GO下载的目录，设置完后的关键参数。

```sh
set GO111MODULE=on
set GOROOT=S:\Cyber Security\2024\go
```

`GO111MODULE`开启时，不需要`GOPATH`来引入依赖。

模块初始化

```sh
go mod init v9d0g/go
```

安装Go语言环境一笔带过，遇到的问题主要有Gin下载时报错

```sh
go get -u github.com/gin-gonic/gin
go: downloading github.com/gin-gonic/gin v1.9.1
go: github.com/gin-gonic/gin@v1.9.1: verifying module: missing GOSUMDB
```

解决方案是

```sh
go env -w GONOSUMDB="*"
```

安装数据库相关依赖

```go
go get -u github.com/jinzhu/gorm
go get github.com/go-sql-driver/mysql
```

windows环境下的vscode中，设置环境变量可能无法生效，需要在vscode终端中输入

```sh
$env:GOROOT="S:\Cyber Security\2024\go"
```

这里不知道为什么我每次打开vscode都得重新设置一遍。

【解决方案】

vscode通过一个GO ENV文件来指定环境变量 一般在左下角 文件路径可以通过 cmd中GO ENV来查看

在此设置GOROOT便不会出现该问题

解决跨域
[GO CORS](https://www.bilibili.com/video/BV1CE411H7bQ?p=12&spm_id_from=pageDriver&vd_source=5a8a81ceda2c7e4a39c105be7125bb84)


# 目前进度：

## `【2024-04-04更新】`

前端的布局大致准备就这样了，但是由于对vue的不熟悉（小看前端开发了）。很多组件之间数据的交互并不规范，有待学习。

![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/660e4f079f345e8d038d13c2.png)



![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/660e4ef49f345e8d038c895e.png)



------

## `【2024-04-10更新】`

大致风格基本定下来了，现在需要完成的就是对路由的规划了。目前的主页会一次性加载所有样式，但是不显示。这种方式不知道会不会导致访问速度变慢，后续可能会更改。还有就是后台登录的页面需要优化，以及现在可以开始着手后端的编写了。

![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/66168d1968eb935713e30afd.png)

![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/66168d3c68eb935713e33536.png)

![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/66168d4f68eb935713e38205.png)

![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/66168d7b68eb935713e3b5cc.png)


------

## `【2024-05-13更新】`

这次把移动端的适配效果做的差不多了，然后把布局整理了一下，以及一些组件的增删。目前记录的所有文章将不会在现在这个[博客](http://v9d0g.fun)里面上传，后续会统一上传至新博客。

![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/6641dcc60ea9cb1403ea433f.png)


![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/6641dcd60ea9cb1403ea5856.png)



---

## `【2024-10-24更新】`

从七月份开始就一直很忙
只有断断续续的改改代码和提进一下开发进度

重新调整了一下主页面的UI和功能 并且初步写好了前端的请求接口
![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/Pasted%20image%2020241028083843.png)

![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/Pasted%20image%2020241028083928.png)

通过文章内容提取目录以及文章内图片的适应大小暂未完成
![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/Pasted%20image%2020241028083949.png)

简单的设计了后端的页面和组件
![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/Pasted%20image%2020241028084154.png)

![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/Pasted%20image%2020241028084208.png)

![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/Pasted%20image%2020241028084216.png)

![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/Pasted%20image%2020241028084224.png)

![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/Pasted%20image%2020241028084232.png)

![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/Pasted%20image%2020241028084238.png)

![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/Pasted%20image%2020241028084244.png)

---

## 【2024-11-13更新】

借用某知名博客的图床图片测试

![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/Pasted%20image%2020241113160442.png)

![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/Pasted%20image%2020241113160515.png)

博客文章中的样式基本完成了

移动端适配 

![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/Pasted%20image%2020241113161046.png)

![](https://raw.githubusercontent.com/v9d0g/my-blog-development-logs-/refs/heads/main/images/Pasted%20image%2020241113161147.png)

TODO:
 - 设计评论组件和预留接口
 - 封装axios接口和获取数据异常弹窗组件
 - 后台