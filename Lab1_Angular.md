# 高级Web技术 Lab 1：Angular 2 框架

2017.4



[TOC]

# 前言

这篇 Lab 主要包括了 Angular 2 框架的背景、技术栈和起步指导。具体学习 Angular 推荐同学们参考官方文档，不要依赖于 Lab 文档。

参考汇总：

> Angular 官方网站: https://angular.io，中文网站：https://angular.cn。
>
> TypeScript: https://www.typescriptlang.org/index.html
>
> npm: https://docs.npmjs.com/getting-started
>
> china npm: https://npm.taobao.org/
>
> Node.js: https://nodejs.org

## Angular 2

如今的 Web 前端发展迅速，迭代频率很快。同学们在 Web 应用基础课上学习的 jQuery 在五年之前还是相当前沿的js库；助教两年前在上高级 Web 技术课时，最常用的前端框架还是是 Backbone.js，那时 Angular 1 和 React 刚刚发布不久；而如今，jQuery 和 Backbone.js 都已被淘汰，Angular 2, React 和 Vue 是三个最流行的前端框架。

在PJ中，推荐同学们使用 Angular 或者 React 开发。除了`多人VR环境`选题之外，不希望不使用前端框架而只使用 jQuery 等简单的库。

> Angular 官网：https://angular.io，Angular 为 Google 出品，请自备翻墙网络。



### 一、 安装 Node 与 npm

首先，我们需要安装 [Node.js](https://nodejs.org/zh-cn/) 和 [npm](https://www.npmjs.com/)。

For Windows: 安装Lab目录中的 node-v6.10.2-x64.msi，或前往官网下载安装。

For Mac: 安装Lab目录中的 node-v6.10.2.pkg，或前往官网下载安装。

npm 会随着 Node.js 一同安装。

#### Node.js

Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行时，可以近似理解为脱离浏览器运行 JavaScript 代码的平台。常见用途是开发后端服务器，同学们可以在 PJ 中使用 Node.js 开发后端。

在 Angular 板块中我们不需要用到 Node，我们需要用到的是 Node 的包管理工具 npm。

#### npm

npm = Node Package Manager，绝大部分的现代前端项目中都使用 npm 作为包管理工具。在 lab1 中我们就会使用npm来安装Angular，TypeScript等包依赖。

> （重要）原生 npm 在中国的访问速度特别慢，如果没有稳定的翻墙环境，推荐使用阿里的国内 npm 镜像：https://npm.taobao.org/



### 二、 新建 Angular 项目

> 新建 Angular 项目会用 npm 安装许多依赖，使用教学楼网络可能会消耗很长时间来下载所有文件。
>
> 如果没有翻墙网络，务必使用阿里的国内 npm 镜像，否则有很大可能下载失败。
>
> 同学们也可以通过 Angular 的 CLI 来创建新的项目，具体参考官方文档。

#### Step 1: Clone official quickstart seed

从头新建 Angular 项目是非常繁琐的，我们直接从 GitHub 上 clone Angular 的起始项目种子。在命令行中运行：

```shell
git clone https://github.com/angular/quickstart.git angular
```

#### Step 2: 使用 npm 安装依赖

首先切换到项目目录：

```shell
cd angular
```

你可以从 `package.json` 文件中查看到项目所用的依赖包。

使用 npm 安装依赖：

```shell
npm install
```

可以观察到， 在安装依赖之前，项目只有 1.4 MB。安装完依赖后，项目大小增加到了 100 MB 以上。

#### Step 3: 运行 Angular 项目

运行 Angular 项目本身是非常复杂的，不过我们 clone 的起始种子已经配置好了 Angular 的运行脚本。

```shell
# 在命令行中运行：
npm start
```

编译成功后，会在本机的 3000 端口运行一个 Node.js 后端部署我们的 Angular 项目。在浏览器中访问 localhost:3000 即可。

#### Step 4: 使用 npm 添加 Material UI 库

> Material UI 是 Google 推出的一套 UI 设计方案，提供了许多 UI 组件，类似于像 BootStrap 之类的 CSS 库。
>
> 使用 UI 库可以简单方便地编写界面美观的前端项目，大大优于手写 CSS。
>
> 这部分的目的是告诉同学们如何通过npm添加自定义的库，可以跳过。
>
> 参考网址：https://material.angular.io/

```shell
npm install --save @angular/material
npm install --save @angular/animations
```

安装之后，我们可以注意到 package.json 中多了两行：

```
......
  "@angular/animations": "^4.0.2",
......
  "@angular/material": "^2.0.0-beta.3",
......
```

新添加的库已经被记录到`package.json`中。

我们还需要在 `src/systemjs.config.js` 中的 angular bundles 里添加以下四行：

```javascript
'@angular/material': 'npm:@angular/material/bundles/material.umd.js',
'@angular/animations': 'npm:@angular/animations/bundles/animations.umd.js',
'@angular/animations/browser': 'npm:@angular/animations/bundles/animations-browser.umd.js',
'@angular/platform-browser/animations': 'npm:@angular/platform-browser/bundles/platform-browser-animations.umd.js',
```



### 三、Angular 2 起步

#### 1. TypeScript

Angular 2 项目一般使用 TypeScript 来代替 JavaScript。直接写 JavaScript 是合法的，但是不推荐。

TypeScript 是一种编译到 JavaScript 的编程语言，弥补了一些 JavaScript 语言上的一些缺点，比 JavaScript 更加强大好用。

> TypeScript 学习：https://www.typescriptlang.org/index.html

#### 2. Angular 代码结构

我们 clone 的 quickstart seed 中有许多文件，其中大部分在我们 Lab 中不需要关注。

首先我们来看 `src/index.html` ，这是 Angular 项目的起始 HTML 页面，也是唯一的 HTML 页面。我们其他的 HTML、CSS、TypeScript 等代码都会被动态加载到这个 HTML 文件中。在页面切换的时候，Angular 不会跳转到另外一个页面，而是修改这个页面。这样的设计结构也被叫做单页面应用（[Single-page application](https://en.wikipedia.org/wiki/Single-page_application)， SPA），大部分现代前端框架都采用了这种设计。

尽管网页内容都会被加载到 `src/index.html` 中，但一般我们不需要对这个文件作修改，网页内容都会动态地加载进来：

```html
<script>
	System.import('main.js').catch(function(err){ console.error(err); });
</script>
```

在这里，`index.html` 导入了 `main.js` 。`main.js` 是从 `main.ts` 编译过来的，如果你已经成功运行过 `npm start`，就能看到 `main.js` 文件。

`main.ts` 做了两件事，一是指定我们的运行平台是浏览器（Angular 还支持其他平台），二是导入了 `src/app/app.module` 文件。而 `src/app/app.module` 导入了 `src/app/app.component.ts` 文件。

我们这次的 Lab 就从`src/app/app.module` 和  `src/app/app.component.ts` 文件开始，暂时不需要理会之前的几个文件。

#### 3. 理解起始工程

下面让我们来理解一下工程的起始页面是如何工作的。

`src/app/app.module` 的主要作用是导入需要用到的库（如 BrowserModule），并声明项目中包含的组件（如 AppComponent）。

`src/app/app.component.ts` 中的内容就是我们运行项目之后看到的 `Hello Angular`。

其中，`template` 的值 `<h1>Hello {{name}}.</h1>` 就是页面上的`Hello Angular`。

`name` 变量在 `AppComponent` 类中被定义为了 `‘Angular’` ，修改 `name` 变量的值就可以修改对应的 HTML 代码，这个特性被称为`数据绑定`。

`selector: 'my-app'` 对应的是 `index.html` 中的`<my-app></my-app>`，表示当前组件的代码会被插入到 `<my-app>` 标签中。

页面的 CSS 效果是因为 `index.html` 中链接了 `style.css` ，像这样直接配置全局 CSS 在 Angular 中不是一个好的方式，后面会介绍应当如何在 Angular 工程中写 CSS 代码。

#### 4. 修改起始代码，改成计时器

首先，我们导入刚刚安装的 `Material UI` 组件到 `src/app/app.module` 中：

```typescript
import {NgModule}      from '@angular/core';
import {BrowserModule} from '@angular/platform-browser';
import {BrowserAnimationsModule} from '@angular/platform-browser/animations';
import {MdButtonModule} from '@angular/material';

import {AppComponent}  from './app.component';

@NgModule({
  imports: [BrowserModule, MdButtonModule, BrowserAnimationsModule],
  exports: [MdButtonModule],
  declarations: [AppComponent],
  bootstrap: [AppComponent]
})
export class AppModule {
}
```

这里只导入了 `Material UI` 中的 `MdButtonModule`， 即按钮组件的样式。使用组件则需要导入对应的组件模块。

然后修改 `src/app/app.component.ts` 为：

```typescript
import {Component} from '@angular/core';

@Component({
  selector: 'my-app',
  styles: ['h1 {color: #00bcd4;}', 'button {background-color: #00bcd4; color: white}'],
  template: `<h1>Time flows: {{time}}s.</h1>
			 <button md-raised-button (click)="addOneSecond()">+1s</button>`,
})
export class AppComponent {
  time = 0;

  ngOnInit():void {
    let self = this;
    setInterval(function () {
      self.time += 1;
    }, 1000);
  }

  addOneSecond():void {
    this.time += 1;
  }
}
```

 `src/app/app.component.ts` 中目前包含了整个页面的 HTML，CSS 和 TypeScript 代码。

- styles 数组即为当前页面的 CSS 设置。
  - 这种类似内联的方式也不够好，推荐使用外联：修改这行为`styleUrls: ['./app.component.css']`，并将 CSS 代码写在 `app.component.css` 文件中。
  - 在 Component 内的 CSS 代码是局部的，不是全局的，不同 Component 中的 CSS 不会相互影响。不仅如此，你甚至可以在不同 Component 中定义 id 或 class 相同的元素，它们的 CSS 也不会相互影响。
- template 中是页面的 HTML 代码
  - {{time}} 对应 AppComponent 中的 time 变量
  - `md-raised-button` 是 Material UI 里的一种按钮样式（md = material design）
  - `(click)="addOneSecond()"` 表示按钮点击时调用 AppComponent 中的 addOneSecond 方法
  - 与 CSS 相同，将此行改为 `templateUrl: 'app.component.html'` 即可使用外联的方式写 HTML 代码
- AppComponent 中定义了一个 time 变量。
  - ngOnInit 方法会在 AppComponent 首次被加载时调用，里面代码的作用是每秒给 time 变量加一
  - addOneSecond 方法可以直接为 time 变量加一

运行后界面如图： ![Screen Shot 2017-04-14 at 1.27.57 AM](https://cloud.githubusercontent.com/assets/6532225/25094182/82575e16-23c8-11e7-8844-e5cf96127f9d.png)

同学们可以在此基础上自己尝试做些修改。

### 四、继续学习 Angular

到这里，Lab 文档已经介绍了许多 Angular 相关的知识，但是没有讲的知识更多，无法一一讲解。

继续学习 Angular 请阅读官方 Tutorial。

相关参考：

> Angular 官方网站: https://angular.io，中文网站：https://angular.cn。
>
> TypeScript: https://www.typescriptlang.org/index.html
>
> npm: https://docs.npmjs.com/getting-started
>
> china npm: https://npm.taobao.org/
>
> Node.js: https://nodejs.org



