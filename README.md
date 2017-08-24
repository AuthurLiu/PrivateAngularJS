# angularJS构建简单项目
---
## 1、目录结构
```
├── dist/
├── node_modules/
├── static/
│   ├── css/
│   ├── fonts/
│   ├── framework/
│   ├── img/
│   ├── js/
├── view/
│   ├── index/
│   │  ├── index.html
│   ├── login/
│   │  ├── login.html
├── src/
│   ├── index/
│   │  ├── indexController.js
│   │  ├── indexService.js
│   │  ├── index.html
│   ├── login/
│   │  ├── loginController.js
│   │  ├── loginService.js
│   │  ├── login.html
│   ├── httpService.js
│   ├── main.js
└── index.html
```
## 2、入口文件index
- 通过在index.html中引入angular.js ui.router及其各模块
```js
<script src="resource/anuglar/angular.js"></script>
<script src="static/framework/anuglar/angular-cookies.js"></script>
<script src="static/framework/anuglar/angular-animate.js"></script>
<script src="static/framework/anuglar/angular-ui-router.js"></script>
<script src="dist/main.build.js"></script>
```
- 在index.html <html>标签标记ng-app 将主模块引入
```js
<html ng-app="app">
```
- 通过ui-view指定路由模块显示区域
```js
<body >
    <ui-view></ui-view>
</body>
```
## 3、主模块main.js
由于原生js不支持模块的引入，这里使用webpack处理模块引入

- 定义主模块，指定依赖模块
```js
var app = angular.module('app', ['ui.router','ngSanitize','ngCookies']);
```
- 引入service，并将其注册到主模块中，这里service用于处理页面接口的请求
```js
var httpService = require('./httpService');
var indexService = require('./indexService');

app
    .constant('httpservice',httpService)
    .factory('indexService',indexService)
```
- 引入controller，controller用于处理页面数据逻辑
```js
var loginController = require('./login/loginController');
```
- 引入template，template中静态模板为HTML文件，在静态模板中使用angularJS提供的directive则为动态模板
- 配置路由，使controller与template形成强耦合，可设置默认路由，传参方式以及父子路由。另：当路由为父路由时，必须使其为抽象
```js
app.config(function ($stateProvider, $urlRouterProvider ) {
    // 当找不到路由状态时自动跳转first状态
    $urlRouterProvider.otherwise("/index/first");
    $urlRouterProvider.when("",'/login');
    //设置路由地址
    $stateProvider
        //主页面
        .state('main', {
            url: "/main",
            templateUrl: "main.html",
            controller: mainController
        })
        //菜单登陆页
        .state('login', {
            url: "/login",
            templateUrl: "views/login/login.html",
            controllerAs: "loginCon",
            controller: loginController
        })
});
```
## 4.页面的构建
UI.router通过将controller与template建立强耦合来构建页面展示逻辑，即将controller指定到ui-view上，并将template的内容替换ui-view标签里的内容。而service用于接口的调用及多页面数据的交互，通过在controller注入service的方式来使用service，service与controller耦合度不高
### controller（控制器）
controller有独立的作用域$scope（作用域可以视作模板、模型和控制器协同工作的粘接器）。通过将变量和方法挂载其上，指定为此controller的标签内元素即可通过angular提供的directive去渲染$scope中的变量，或将方法挂载到某个标签上。
尽管控制器看起来并没有起到什么控制的作用，但是它在这里起到了至关重要的作用。通过给定我们数据模型的语境，控制器允许我们建立模型和视图之间的数据绑定。
```
module.exports = function ($scope,loginService) {
    function init(){
        $scope.userFilter = ' ';
        $scope.allNum = 15;
        $scope.phones = [
            { name: "Nexus S",
             snippet: "Fast just got faster with Nexus S." },
            { name: "Motorola XOOM™ with Wi-Fi",
             snippet: "The Next, Next Generation tablet." },
            { name: "MOTOROLA XOOM™",
             snippet: "The Next, Next Generation tablet." }
        ];
        $scope.showList = $scope.phones
        $scope.tip = '<div>未找到相关项</div>'
    }
    
    $scope.handleFilter = function(){
        $scope.showList = $scope.phone.filter(function(item){
            item.name.indexOf($scope.userFilter)
        })
    }
    
}
```
### template(模板)&&angularJS directive（指令）
#### 渲染定义到$scope上的数据
这样的指令包括{{}} ng-bind ng-html ng-repeat
```
<section>
    <div>{{allNum}}</div>
    <div ng-bind='allNum'></div>
    <div ng-html='tip'></div>
    <ul>
        <li ng-repeat='item in showList'>{{item.name}}</li>
    </ul>
</section>
```
#### 将定义在$scope上的方法挂载到元素上
```
<button ng-click='handleFilter()'></button>
```
#### 将输入框等输入字段绑定到$scope形成双向绑定
```
<input type="text" ng-modal='userFilter'>
```
#### 控制元素class和显示
```
<ul ng-show='allNum'>
    <li ng-repeat='item in showList'>{{item.name}}</li>
</ul>
<ul ng-class='{'strike': deleted, 'bold': important, 'has-error': error}'>
    <li ng-repeat='item in showList'>{{item.name}}</li>
</ul>
```
### service（服务）
服务是定义在模块级的函数或对象，在同模块控制器中，只需依赖注入就可使用。因此可以用来实现多个控制器间的通讯、定义util方法以及全局常量
用来和接口通讯
```
module.exports = function ($q, $http, httpService) {
  var loginService = {
    getData: function () {
      var deferred = $q.defer();
      $http({
        method: 'GET',
        url: httpService + 'account',
        params: {}
      })
      .then(function (res) {
        deferred.resolve(res.data);
      })
      .catch(function (response) {
        console.error('error', response.status, response.data.errno);
        deferred.reject(res.data);
      })
      return deferred.promise
    }
  }
  return loginService
}
```
util方法
```
module.exports = function($rootScope){
    this.closeLoading = function () {
        $rootScope.fadeShow = false;
        $rootScope.loadingShow = false;
        $('html,body').removeClass('ovfHiden');
    };
    this.openLoading = function () {
        $rootScope.fadeShow = true;
        $rootScope.loadingShow = true;
        $('html,body').addClass('ovfHiden');
    };
    return this;
};
```
定义常量 httpservice
```
var urlWebHttp = 'smxi/api/v1/';

module.exports = {
    devloper1:'http://192.168.0.1/' + urlWebHttp,
    devloper2:'http://192.168.0.2/' + urlWebHttp,
    devloper3:'http://192.168.0.3/' + urlWebHttp,
};
```
服务的挂载方法，如果提供的是工厂函数，使用factory挂载，如果是对象，则使用constant挂载
```
app
    .constant('httpservice',httpService)
    .factory('indexService',indexService)
```
### 自定义directive(指令)
angularJS没有组件的概念，directive可以实现类似组件的概念
#### showListTemplate.js
```
<ul >
	<li 
	    ng-repeat='(item,index) in srcList' 
	    ng-click='handleItem(index)'
	>
		{{item.name}}
	</li>
</ul>
```
#### showListDirective.js
```
module.exports = function($parse,$rootScope){
    return {
        restrict: 'E',
        scope:{
            list:@,
            handleItem:&
        }
        templateUrl: './showList.html',
        replace: true,
        link: function($scope) {
            $scope.srcList = Json.parse(list)
        }
    };

};
```
## 5.angularJS与React
### 1.dom更新方式
#### AngularJS 
采用“脏值检测”的方式，数据发生变更后，对于所有的数据和视图的绑定关系进行一次检测，识别是否有数据发生了改变，有变化进行处理，可能进一步引发其他数据的改变，所以这个过程可能会循环几次，一直到不再有数据变化发生后，将变更的数据发送到视图，更新页面展现。如果是手动对 ViewModel 的数据进行变更，为确保变更同步到视图，需要手动触发一次“脏值检测”。
### React
通过手动调用setState进行数据更新，默认调用setState即会重新渲染dom，可通过shouldComponentUpdate（）控制渲染。加入Virtual DOM提升性能（浏览器处理js速度远大于渲染dom）。
### 2.模板形式
angular采用的是在静态html文件中内嵌ng-directive形成的一种模板结构，其模板展现需谨遵angular提供的形式。react采用的是jsx，易于调试，且充分利用js的全部功能，避免学习的开销。
### 3.逻辑书写形式
angular中，页面逻辑由强耦合的controller、template共同完成，造成了逻辑的分散，不易排查问题，但易于分工。React不强制分离模板与逻辑，通过jsx混合写在组件中，易于调试。

 

