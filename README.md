 HTML模板引擎

## 支持特性
1. 根据页面内嵌节点作为模板
2. 根据嵌入式代码文件作为模板
3. 从文件加载模板
4. include标签的支持(页面嵌套)


## 安装

```
  npm install dee-template
  
```

# 用法1. 根据页面内嵌节点作为模板 
模板代码 (将模板放在当前代码页面)
```html
<TEXTAREA id='tpl_1' style='display:none'>
   日期:<span>${date}</span>
   姓名:<span>${name}</span>
   照片:<img src='${body.photo}'>
   体重:<span>${body.weight}</span>
   身高:<span>${body.length}</span>
   
</TEXTAREA>

<div id='babyInformation'>
</div>
```
用法
```js
  require("dee-template");
  var data ={
      date:'2017-06-02',
      name:'Allen,Ding',
      body:{
         'weight':3.45,
         'length':50,
      }
  }
  var node = Template.makeNode('#tpl_1', data);
  $('#babyInformation').append(node);
  
  //或者
  Template.makeNodeTo('#tpl_1',data,'#babyInformation');
  
```


# 用法2. 根据嵌入式代码文件作为模板
主页面代码:
```html
<object id='obj_1' data='include.inc.html'
type='text/plain' style='disply:none'></object>
```
模板文件 include.inc.html代码:
```html
<TEXTAREA id='tpl_1'>
   日期:<span>${date}</span>
   姓名:<span>${name}</span>
   照片:<img src='${body.photo}'>
   体重:<span>${body.weight}</span>
   身高:<span>${body.length}</span>
    <% 
        for(var i =0; i < 5; i++)
        {
    %>
        ${i} is <%=i %> 
     <%
        }
    %>
</TEXTAREA>
<TEXTAREA id='tpl_2'>
.....
</TEXTAREA>
```
用法：
```js
 require("dee-template");
  var data ={
      date:'2017-06-02',
      name:'Allen,Ding',
      body:{
         'weight':3.45,
         'length':50,
      }
  }
  var maker = Template.fromEmbededObject('#obj_1');
  var html = maker.template('#tpl_1', data);
  
```


# 用法3. 从文件加载模板（node）

>（依赖 FS 组件，只适合于nodeJs环境）

主页面代码:
```html
 无
```
模板文件 include.inc.html代码:

```html
<div id='tpl_1'>
   日期:<span>${date}</span>
   姓名:<span>${name}</span>
   照片:<img src='${body.photo}'>
   体重:<span>${body.weight}</span>
   身高:<span>${body.length}</span>
    <% 
        for(var i =0; i < 5; i++)
        {
    %>
        ${i} is <%=i %> 
     <%
        }
    %>
</div>
<div id='tpl_2'>
...
</div>
```
主界面js用法：
```js
  require("dee-template");
  var data ={
      date:'2017-06-02',
      name:'Allen,Ding',
      body:{
         'weight':3.45,
         'length':50,
      }
  }
  var maker = Template.fromFile('include.inc.html');
  var html = maker.template('#tpl_1', data);
   
  
```

# 4.  include  标签的支持(html文件引用)

> 本方法适用于HTML的模块化
，本功能特别适合于大型项目，进行模块划分
include 功能的深度嵌入
 


主界面代码，
举例1
```html
<INCLUDE node='tpl_1' src='include.inc.html' script='hello.js'></INCLUDE>
```
举例2
```html
<INCLUDE module='inc_1' src='include.inc.html'></INCLUDE>
<INCLUDE module='inc_1' node='tpl_1' ></INCLUDE>
<INCLUDE module='inc_1' node='tpl_2' data='{a:1,b:2}'></INCLUDE>
```
模板文件 include.inc.html代码:
```html
<div id='tpl_1' >
   日期:<span>${date}</span>
   姓名:<span>${name}</span>
   照片:<img src='${body.photo}'>
   体重:<span>${body.weight}</span>
   身高:<span>${body.length}</span>
</div>
<div id='tpl_2'>
   ...
</div>
```
主界面JS代码
```js
 无
```
## 4.1 include标签的属性 
属性 | 说明
---|---
src | 所嵌入的html代码所在文件名
node | 所使用的子模板的id（在模板文件内的节点id）
module | include模板的实例id
data | 向模板填充的值
script| 需要执行的代码 (书写规则另见)

如果一个模板文件 有多个子模板节点被使用，
可以参考include功能的举例2，先将模板文件实例化，再分别使用各子模板


## 4.2 支持循环嵌套

include 支持循环嵌套

## 4.3 支持运行js代码

script 指向模块需要运行的内嵌代码，规则举例
```js
class MyModule  {
  constructor($) { 
       this.$=$;
       //  $选中的节点，为局部HTML模块的节点
       var name = $('#username').html(); 
       myOldCode($);
  }
  publicMethod (){
     //公开的函数
     //使用方法，见下文
  }
} //class
function myOldCode($){
    //all my old code
}
module.exports = MyModule;

```

### 4.3.1 调用公共函数
```js
//调用公共函数
$('include')[0].runtime.publicMethod();
//访问节点的类实例的函数

```
### 4.3.2 绑定自定义事件
```js
//绑定自定义事件
$('include')[0].runtime.on('myevent',function(param){
    //.....
})
```
### 4.3.3 触发自定义事件
```js
//类内部触发事件
this.trigger('myevent',123);
//或  
this.myevent.trigger(123);
//或从外部触发
$('include')[0].runtime.trigger('myevent',123);
//或
$('include')[0].runtime.myevent.trigger(123)
```

**data 属性应用：**
> 没有node值的include标签不会被实例化，
可以为该节点设置 node, data, module,甚至src 后，
使用activeInclude方法，动态激活该标签
```html
<include id='inc_1'> </include>
```
```js
$('#inc_1').attr('src','xxx.inc.html');
$('#inc_1').attr('node','tpl_10');
$('#inc_1').attr('data','{a:100,b:200}');
Template.activeInclude('#inc_1');
//或者
Template.activeInclude(
     '#inc_1',
     {node:'', src:'', data:''}
);
```

## 4.4 事件制造器 Template.EventFactory
```js
Template.EventFactory(obj, event)
```
```js
//为对象声明事件
Template.EventFactory(mainObj, 'myevent');

//触发事件
mainObj.myevent.trigger(111);

```

使用者
```js
mainObj.myevent(function(param){
 //接收到参数

});
 

```
 
 
# Thank you!

 
