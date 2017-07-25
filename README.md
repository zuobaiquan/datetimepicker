---
title: angularjs自定义指令directive
date: 2017-04-09 16:31:49
tags: AngularJS
---

```javascript
var app = angular.module('myApp',[],['$compileProvider',function($compileProvider){
    console.log($compileProvider);//这里面包含directive方法
    //-->指令的名字
    //指令的名字不要使用ng为指令名称，这样可能会和angular内置指令冲突
    //如果指令的名字为 xxx-yyy 在设置指令的名字时应为xxxYyy驼峰式命名
    $compileProvider.directive('customTag',function(){
        return {
            restrict:''
        }
    })
}]);
```

### restrict属性

restrict：指令在模板中的使用方式

可以是4种风格任意组合，如果忽略 restrict ，默认为 A

如果打算支持 IE8 ，就使用基于属性和样式类的指令

|  字母  |  风格  |                实例                 |         说明         |
| :--: | :--: | :-------------------------------: | :----------------: |
|  E   |  元素  |         <my-dir></my-dir>         |    可以当做html标签使用    |
|  C   | 样式类  | <span class="my-dir:exp;"></span> | 把自定义指令名称写在class类里边 |
|  A   |  属性  |    <span my-dir="exp"></span>     |   把自定义指令名称写在属性里边   |
|  M   |  注释  |   <!--directive:my-dir exp -->    |         --         |

```html
<body ng-app="myApp">
    E:<custom-tags></custom-tags>  <br />
    C:<div class="custom-tags"></div>  <br />
    A:<div custom-tags></div>  <br />
    M:<!-- directive:custom-tags -->  <br />
</body>
```

```javascript
var app = angular.module('myApp',[],['$compileProvider',function($compileProvider){
    console.log($compileProvider);//这里面包含directive方法
    //-->指令的名字
    //指令的名字不要使用ng为指令名称，这样可能会和angular内置指令冲突
    //如果指令的名字为 xxx-yyy 在设置指令的名字时应为xxxYyy驼峰式命名
    // template：模板的内容，这个内容会根据replace参数的设置替换节点或只替换节点内容
    // replace :如果此配置为true，则替换指令所在的元素，如果为false或者不指定，则把当前指令追加到所在元素的内部
    //    对于restrict为元素(E)在最终效果是多余的，所有的replace通常设置为true
    $compileProvider.directive('customTags',function(){
        return {
            restrict:'ECAM',
            template:'<div>我是自定义指令</div>',
            replace:true,
            //这里可以通过comppile,如果console.log四次，表示都渲染成功
            compile:function(){
                console.log(1);
            }
        }
    })
}]);
```
### templateUrl属性

1.templateUrl加载模板所要使用 的url

  可以在templateUrl模板中使用scope里面的变量

```
<!--常见的报错信息： Template for directive 'comstomTags' must have exactly one root element-->
<!--如果replace为true时，必须在模板中加上最外层的标签（根）-->
<!--同理：指定template属性时，也应该加上最外层的标签（根）-->
<!--templateUrl可以使用控制器里面的变量-->
```

2.可以加载当前模板内对应的text/ng-template script id

```html
<body ng-app="myApp">
<script type="text/ng-template" id="comstomTags2">
    <div>hello {{name}}</div>
</script>
<div ng-controller="firstController">
    <comstom-tags2></comstom-tags2>
</div>
</body>
```

```javascript
var app = angular.module('myApp',[])
.directive("comstomTags2",function(){
    return {
        restrict:'ECAM',
        templateUrl:'comstomTags2',
        replace:true
    }
})
.controller('firstController', function ($scope) {
    $scope.name='zuobaiquan';
});
```

###  transclude属性

自定义指令的属性 transclude：为true时，允许把html中新定义的指令中原来的dom运用到该指令的template中。

```html
<body ng-app="myApp">
	<div ng-controller="firstController">
       <custom-tags>我是原来的数据</custom-tags>
	</div>
</body>
```

```javascript
//自定义指令的属性 transclude：为true时，
// 允许把html中新定义的指令中原来的dom运用到该指令的template中。
.directive('customTags',function(){
    return {
        restrict:'ECAM',
        template:'<div>我是新数据<br/><span ng-transclude></span></div>',
        replace:true,
        transclude:true
    }
})
```

### priority、terminal属性

自定义指令的属性priority 设置该指令的优先级，优先级大的先执行，默认指令的优先级是0（但ng-repeat指令的优先级默认是1000）。

 属性terminal:为true时，指示优先级小于当前指令的指令都不执行，仅执行到本指令。

```html
<body ng-app="myApp">
    <div ng-controller="firstController">
        <h3>一个div上同时使用 custom-tags2 指令 和 custom-tags3 指令：</h3>
        <div custom-tags2 custom-tags3></div>
    </div>
</body>
```

```javascript
angular.module('myApp',[])
//定义第二个指令：customTags2
.directive('customTags2',function(){
    return {
        restrict:'ECAM',
        template:'<div>222</div>',
        replace:true,
        priority:-1  //指示指令的优先级，优先级大的先执行，默认指令们的优先级都是0，但ng-repeat指令的优先级是1000
    }
})
//定义第三个指令：customTags3
.directive('customTags3',function(){
    return {
        restrict:'ECAM',
        template:'<div>333</div>',
        replace:true,
        priority:0,
        //小于0的directive都不会执行
        terminal:true  //为true时，指示优先级小于本指令的优先级的directive都不再执行
        //为false的时候，执行会报错，不能有多个template（多个directive，只能有一个template或者templateUrl）
    }
})
.controller('firstController',['$scope',function($scope){

}]);
```

### compile&link属性

angular指令编译三阶段

1.标准浏览器API转化：将html转化成dom，所以自定义的html标签必须符合html的格式；

2.angualr compile：搜索匹配directive，按照priority排序，并执行directive上的directive的compile的方法；

3.angualr link：执行directive上的link方法，进行scope绑定及事件绑定。

```html
<body ng-app="myApp">
<div ng-controller="firstController">
    <h3>一个dom上使用两个指令：custom-tags 和 custom-tags2</h3>
    <!--
        step1. div先转化成dom结构
        step2. 再按指令的优先级依次执行相应指令的compile方法
    -->
    <ul>
        <li ng-repeat="user in users" custom-tags custom-tags2></li>
    </ul>
</div>
</body>
```

```javascript
var i=0;
angular.module('myApp',[])
//定义第一个指令：customTags
.directive('customTags',function(){
    return {
        restrict:'ECAM',
        template:'<div>{{ user.name }}</div>',
        replace:true,
        //定义了compile就不需定义link,当compile返回一个方法这个方法就是link
        //tElement 正在执行该指令的当前dom元素的jquery对象
        //tAttrs   正在执行该指令的当前dom元素的属性
        compile:function(tElement,tAttrs,transclude){
            //第一个指令的编译阶段...
            console.log('customTags compile 编译阶段...');

            //若要改变dom元素,应在compile中做，此处在当前dom元素中添加一个<span>
            tElement.append(angular.element('<span> {{user.count}}</span>'));
            return {
                //pre:编译阶段之后执行
                pre:function preLink(scope,iElement,iAttrs,controller){
                    console.log('customTags preLink..');
                },
                //post:所有子元素指令的post都执行后执行，此处设置了dom元素的点击事件
                post:function postLink(scope,iElement,iAttrs,controller){
                    iElement.on('click',function(){
                        scope.$apply(function(){
                            scope.user.name=' click after';
                            scope.user.count= ++i;
                        });
                    });
                    console.log('customTags post end.');
                }
            };
            //compile也可直接返回一个方法，这就是 postLink，也就是上面的post
            //return function (){
            //    console.log('compile return this function')
            //}
        },
        //进行scope绑定及事件绑定
        link:function(scope,iElement,iAttrs,bookListController){
            //link不会再执行了，因为这里定义的就是postLink
        }
    }
})

//定义第二个指令：customTags2
.directive('customTags2',function(){
    return {
        restrict:'ECAM',
        replace:true,
        compile:function(tElement,tAttrs,transclude){
            console.log('customTags2 compile 编译阶段...');
            return {
                pre:function preLink(){
                    console.log('customTags2 preLink..');
                },
                post:function postLink(){
                    console.log('customTags2 post end.');
                }
            };
        }
    }
})
.controller('firstController',['$scope',function($scope){
    $scope.users=[
        {id:10,name:'张三',count:0}
    ]
}]);
```

运行结果

![](http://files.cnblogs.com/files/zuobaiquan01/QQ%E6%88%AA%E5%9B%BE20170531214637.bmp)

改变控制器内的初始数据为：

```javascript
$scope.users=[
    {id:10,name:'张三',count:0},{id:20,name:'李四',count:0}
]
```

![](http://files.cnblogs.com/files/zuobaiquan01/QQ%E6%88%AA%E5%9B%BE20170531215119.bmp)

 示例中，在一个 li 上同时使用了两个 angualr自定义指令，custom-tags和custom-tags2。当一个dom上同时使用两个指令时，只能在其中一个指令中定义template，否则会出错。



 关注一下两个指令的控制台输出，表现出的两个指令内部代码的执行顺序，可以看出，针对每一个应用多个指令的dom，先依次执行指令的编译阶段，再依次执行指令的preLink代码，而post代码则是等待自己后面的指令的post都执行结束后再执行（多个指令的post执行有种栈的感觉）。



总结：

#### compile

1.compile:function(tElement,tAttrs,transclude)

2.compile函数用来对模板自身进行转换，仅仅在编译阶段运行一次

3.compile中直接返回的函数是postLink，表示link参数需要执行的函数，也可以返回一个对象里面包含preLink和postLink

4.当定义compile参数时，将无视link参数，因为compile里返回的就是该指令需要执行的link函数



#### link

1.link:function(scope,iElement,iAttrs,controller)

2.link参数代表的是compile返回的postLink

3.preLink表示在编译阶段之后，指令连接到子元素之前运行

4.postLink表示会在所有子元素指令都连接之后才运行

5.link函数负责在模型和视图之间进行动态关联，对于每个指令的每个实例，link函数都会执行一次。



### controller-controllerAs属性

```
1.controller它会暴露一个API，利用这个api可以在多个指令之间通过依赖注入进行通信；

2.controller($scope,$element,$attrs ,$transclude)

3.controllerAs是给controller起的别名，方便使用

4.require可以将其他指令传递给自己

```