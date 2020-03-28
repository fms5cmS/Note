[AngularJS](http://www.angularjs.net.cn/)是一款优秀的前端JS结构化框架，已经被用于Google的多款产品当中。AngularJS有着诸多特性，最为核心的是：MVC、模块化、自动化双向数据绑定、依赖注入等等。

- 与 jQuery 对比：
  - jQuery 是一个 JS 函数库，封装简化 DOM 操作
  - AngularJS 是一个 JS 结构化框架，主体不再是 DOM，而是页面中的动态数据
- AngularJS 可以构建单页面(SPA：Single Page Application) Web应用或 Web App 应用
  - 单页面应用特点：
    - 所有活动局限于一个页面
    - 当页面中有部分数据发生了变化不再刷新整个页面而是局部刷新
    - 利用的是 Ajax 技术，路由

# 特征

- MVC模式
  - Angular遵循软件工程的MVC模式，并鼓励展现，数据，和逻辑组件之间的松耦合.通过依赖注入（dependency injection），Angular为客户端的Web应用带来了传统服务端的服务，例如独立于视图的控制。 因此，后端减少了许多负担，产生了更轻的Web应用。
    - Model:数据，其实就是angular变量(`$scope.XX`)；
    - View: 数据的呈现，Html+Directive(指令)；
    - Controller:操作数据，就是function，数据的增删改查。
- 双向绑定
  - 声明式编程应该用于构建用户界面以及编写软件构建，而指令式编程非常适合来表示业务逻辑。
  - 框架采用并扩展了传统HTML，通过双向的数据绑定来适应动态内容，双向的数据绑定允许模型和视图之间的自动同步。因此，AngularJS使得对DOM的操作不再重要并提升了可测试性。
- 声明式依赖注入



# HelloWorld

```html
<!doctype html>
<!--当加载该页时，标记ng-app告诉AngularJS处理整个HTML页并引导应用-->
<html ng-app>
    <head>
        <!--引入AngularJS的头文件-->
        <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.5.8/angular.min.js"></script>
    </head>
    <body>
        <div>
            <label>Name:</label>
            <!--指令ng-model将<input>输入的值绑定到了变量yourname-->
            <input type="text" ng-model="yourName" placeholder="Enter a name here">
            <hr>
            <h1>Hello {{yourName}}!</h1>
        </div>
    </body>
</html>
```

Angularjs表达式用双括号`{{ }}`形式表示，他会对包裹的`yourname`变量进行解析。指令`ng-model`一将`<input>`输入的值绑定到了变量`yourname`，`{{yourname}}`就解析出来了。这个过程是同步的，而且是双向的。

- 变量初始化：`ng-init="myname='zzk'"`，myname变量的初始值就是zzk
- 