# jQuery

```javascript
console.log($.fn.jquery);  //打印 jQuery 库的版本号
```

# 1.基础

## 1.1了解

jQuery是一款JS库，可以简化原生JS的操作，最主要的用途是用来做查询。[中文API](http://tool.oschina.net/apidocs/apidoc?api=jquery)

- 为什么使用jQuery：

  - 强大的选择器：方便快速查找DOM元素；

  - **链式编程**：比如`.show()`和`.html()`可以连写成`.show().html()`；

  - **隐式迭代**：在方法的内部会为匹配到的所有元素进行循环遍历，执行相应的方法；简化操作，方便调用；

  - 读写合一：读数据和写数据使用同一个函数；

  - 事件处理；

  - DOM操作

  - 样式操作；

  - 动画；

  - 丰富的插件支持；

  - 浏览器兼容。

    - 1.x：兼容ie678，但相对其它版本文件较大，官方只做BUG维护，功能不再新增；

      2.x：不兼容ie678，相对1.x文件较小，官方只做BUG维护，功能不再新增；

      3.x：不兼容ie678，只支持最新的浏览器，很多老的jQuery插件不支持这个版本，相对1.x文件较小，提供不包含Ajax/动画API版本。


使用：

1. 在[官网](https://code.jquery.com/)下载jQuery库(下载uncompressed的版本，源代码便于阅读)

2. 引入jQuery库+入口函数

   ```html
   <head>
   	<!--引入jQuery库-->
       <script src="jquery-1.12.4.js"></script>
       <script>
           //入口函数
           $(document).ready(function () {
               //功能实现代码
               $("#btn").click(function () {
                   alert("jQuery事件处理函数");
               });
           });
       </script>
   </head>
   ```




## 1.2 jQuery入口函数

- 原声JS的入口函数:

  ```javascript
  //原生 js 的入口函数。页面上所有内容加载完毕，才执行。
  //不仅要等文本加载完毕，而且要等图片也要加载完毕，才执行函数。
  window.onload = function () {
      alert(1);
  }
  ```

- jQuery的入口函数有多种写法:

  - 方法一:

    ```javascript
    //1.文档加载完毕，图片不加载的时候，就可以执行这个函数。
    $(document).ready(function () {
        alert(1);
    })
    ```

  - 方法二:(写法一的简洁版)

    ```javascript
    //2.文档加载完毕，图片不加载的时候，就可以执行这个函数。
    $(function () {
        alert(1);
    });
    ```

  - 方法三:

    ```javascript
    //3.文档加载完毕，图片也加载完毕的时候，在执行这个函数。
    $(window).ready(function () {
        alert(1);
    })
    ```

- jQUery与JS加载模式对比：

  - JS中的多个`window.onload`只会执行官一次，后面的会覆盖前面的

    ```html
    <script>
        window.onload = function () {
            alert("hello lnj1"); // 不会显示
        }
        window.onload = function () {
            alert("hello lnj2"); // 会显示
        }
    </script>
    ```

  - jQuery中的多个`$(document).ready()`会执行多次，不存在覆盖问题

    ```html
    <script>
        $(document).ready(function () {
            alert("hello lnj1"); //会显示
        });
        $(document).ready(function () {
            alert("hello lnj2"); // 会显示
        });
    </script>
    ```

  - 对比：

    |          | window.onload                                                | $(document).ready()                                  |
    | -------- | ------------------------------------------------------------ | ---------------------------------------------------- |
    | 执行时机 | 必须等待网页全部加载完毕(包括 图片、外部的js文件等),然后再执行包裹代码 | 只需要等待网页中的DOM结构加载完毕,就能执行包裹的代码 |
    | 执行次数 | 只能执行一次,如果第二次,那么 第一次的执行会被覆盖            | 可以执行多次,第N次都不会被上 一次覆盖                |
    | 简写     | 无                                                           | $(function(){})                                      |




## 1.3 jQuery的$符号

`$`符就是`jQuery`：`window.jQuery`=`window.$`=`jQuery`

```html
   <script src="jquery-1.12.4.js"></script>
    <script>
        console.log($==jQuery)    //true
    </script>
```





## 1.4 JS的DOM对象和jQuery对象

jQuery对象的本质是一个伪数组，如：

```javascript
var obj = {0:"abc" ,1:"33" ,2:"male" ,length:3}
```



- JS的DOM对象和jQuery对象的区别:

  - 通过 jQuery 获取的元素是一个**数组**，数组中包含着原生JS中的DOM对象。如:

    ```javascript
    //这些方式获取的都是元素节点数组
    var jqBox1 = $("#box");
    var jqBox2 = $(".box");
    var jqBox3 = $("div");
    ```

  - 原生JS获取元素：

    ```javascript
    var myBox = document.getElementById("box");           //通过 id 获取单个元素
    var boxArr = document.getElementsByClassName("box");  //通过 class 获取的是数组
    var divArr = document.getElementsByTagName("div");    //通过标签获取的是数组
    ```

  - 总结：jQuery 就是把 DOM 对象重新包装了一下，让其具有了 jQuery 方法。

- JS的DOM对象和jQuery对象的相互转换：

  - DOM对象转为jQuery对象：`$(js对象)`

    ```javascript
    jqBox1 = $(myBox);
    jqBox2 = $(boxArr);
    jqBox3 = $(divArr);
    ```

  - jQuery对象转为DOM对象：`jquery对象[index]`（推荐！）或`jquery对象.get(index)`

    ```javascript
    var $box = $("#box");
    var box = $box[0];
    ```


案例：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="jquery-1.12.4.js"></script>
    <script>
        $(function () {
            var lis = $("li");
            for(var i=0;i<lis.length;i++){
                if(i % 2 == 0)
                    lis[i].style.background = "red";
                else
                    lis[i].style.background = "blue";
            }
        });
    </script>
</head>
<body>
    <ul>
        <li>NY</li>
        <li>fS</li>
        <li>Clannad</li>
        <li>比宇宙更远的地方</li>
    </ul>
</body>
</html>
```



## 1.5 核心函数

`$()`就代表调用jQuery的核心函数。

jQuery核心函数一共三大类。

- 1.接收一个函数`jQuery(callback)`：当DOM加载完成后执行传入的回调函数

  ```html
  <script>
      $(function () {
      });
  </script>
  ```

- 2.接收一个字符串

  - 2.1接收一个字符串选择器`jQuery()`

    ```html
    <script>
        $(function () {
            var div = $("div");
        });
    </script>
    ```

  - 2.2接受一个代码片段

    ```html
    <script>
        $(function () {
            var $p = $("<p>段落</p>");
        });
    </script>
    ```

- 3.接受一个DOM元素

  ```html
  <script>
      $(function () {
          var span = document.getElementByTagName("span")[0]; //获取一个DOM元素
          
          var $span = $(span);
      });
  </script>
  ```



## 1.6 静态方法

可以直接使用`$.方法名`调用。

- `$.holdReady()`：暂停或恢复`$.ready()`方法，传入参数：true或false；

- `$.each(object,function(index,element){})`：遍历对象或数组。参数：

  - object:要遍历的对象

  - index：表示当前元素在所有匹配元素中的索引号

  - element：参数二表示当前元素（是js 中的DOM对象，而不是jQuery对象）

    ```javascript
    var arr = {1:2,3:4,5:6,7:8,len:9};
    $.each(arr,function(index,value){
        console.log(index,value);
    })
    ```

- `$.map(object,callback)`：遍历对象或数组，将回调函数的返回值组成一个新的数组返回

  ```javascript
   var $res = $.map(obj, function (value, key) {
       console.log(key, value);
       return key + value;
   });
  ```

- `$.trim(str)`：去掉字符串起始和结尾的空格

- `$.isArray(obj)`：判断是否为数组

- `$.isFunction(obj)`：判断是否为函数

- `$.isWindow(obj)`：判断是否为window对象



## 1.7 选择器

### 基本选择器

| 语法                                     | 说明                                         |
| ---------------------------------------- | -------------------------------------------- |
| `#id`                                    | id选择器                                     |
| `.class`<br/>eg:`$("p.intro")`           | 类选择器<br/>选择所有class="intro"的\<p>元素 |
| `element`                                | 元素选择器                                   |
| `*`                                      | 选择所有元素(少用)                           |
| `$("selection1,selector2,...selectorN")` | 并集选择器                                   |



### 层级选择器

| 符号 | 说明                                                       |
| ---- | ---------------------------------------------------------- |
| 空格 | 后代选择器，选择指定元素的后代元素（儿子、孙子。。。都要）eg：`$("ul li")`选取ul的所有li后代元素 |
| `>`  | 子元素选择器，选择指定元素的**直接**子元素（只要亲儿子）eg：`$("ul>li")`选取ul的直接子代li元素 |
| `+`     | eg：`$("p+strong")`选取p元素后面紧跟的strong元素 |
| `~`      | eg：`$("p~strong")`选取p元素后面所有的strong元素 |



### 基本过滤选择器

| 选择器 | 示例 | 选取 |
| ------ | ---- | ---- |
|`:first`	|`$("p:first")`|	第一个 \<p> 元素|
|`:last`	|`$("p:last")`	|最后一个\<p> 元素|
|`:even`	|`$("tr:even")`	|所有偶数 \<tr> 元素|
|`:odd`	|`$("tr:odd")`	|所有奇数\<tr> 元素|
|`:eq(index)`	|`$("ul li:eq(3)")`	|列表中的第四个元素（index 从 0 开始）|
|`:gt(no)`	|`$("ul li:gt(3)")`	|列出 index 大于 3 的元素|
|`:lt(no)`	|`$("ul li:lt(3)")`	|列出 index 小于 3 的元素|
|`:not(selector)`	|`$("input:not(:empty)")`	|所有不为空的 input 元素|



### 内容过滤器

| 选择器            | 描述                                 |
| ----------------- | ------------------------------------ |
| `:empty`          | 选取不包含子元素或文本为空的元素集合 |
| `:parent`         | 选取含子元素或文本的元素集合         |
| `:contains(text)` | 选取含有文本内容为 text 的元素集合   |
| `:has(selector)`  | 选取含有选择器所匹配的元素集合       |



### 属性选择器

|选择器|示例|选取|
|---------|-------|-----|
|`[attribute]`	|`$("[href]")`	|所有带有 href 属性的元素|
|`[attribute=value]`	|`$("[href='#']")`	|所有 href 属性的值等于 "#" 的元素|
|`[attribute!=value]`	|`$("[href!='#']")`	|所有 href 属性的值不等于 "#" 的元素|
|`[attribute$=value]`	|`$("[href$='.jpg']")`	|所有 href 属性的值包含以 ".jpg" 结尾的元素|



### 筛选选择器

| 选择器         | 示例                            | 选取                               |
| -------------- | ------------------------------- | ---------------------------------- |
| `find(选择器)` | `find("li")`                    | 指定元素的所有后代元素（子子孙孙） |
| `children()`   | `children("li")`                | 指定元素的直接子元素（亲儿子元素） |
| `siblings()`   | `$("ul").children().siblings()` | 所有兄弟元素（不包括自己）         |
| `parent()`     | `元素.parent()`                 | 父元素（亲的）                     |
| `next()`       | `元素.next()`                   | 当前元素的下一个兄弟元素           |
| `prev()`       | `元素.prev()`                   | 当前元素的上一个兄弟元素           |



## 1.8 示例

鼠标悬停时，弹出下拉菜单：

```javascript
<script src="jquery-1.12.4.js"></script>
<script>
    //入口函数
    $(document).ready(function () {
    //需求：鼠标放入一级li中，让他里面的ul显示。移开隐藏。
    var jqli = $(".wrap>ul>li");

    //绑定事件
    jqli.mouseenter(function () {
    	// 注意：jquery对象绑定的事件中，this指js中的dom对象。【重要】
        //让this中的ul显示出来。
        $(this).children("ul").show();
    });

    //绑定事件：鼠标移开时，隐藏下拉菜单
    jqli.mouseleave(function () {
        $(this).children("ul").hide();
    });
});
</script>
```



# 2.动画

jQuery提供的一组网页中常见的动画效果，这些动画是标准的、有规律的效果；同时还提供给我们了自定义动画的功能。



## 2.1 显示动画

如何显示动画：

```javascript
//无参数，表示让指定的元素直接显示出来。底层是通过display: block;实现的
$("div").show();
```

```javascript
//通过控制元素的宽高、透明度、display属性，逐渐显示，2秒后显示完毕
$("div").show(2000);
```

```javascript
//通过控制元素的宽高、透明度、display属性，逐渐显示
//slow:慢,600ms ; 
//normal:正常,400ms;
//fast:快,200ms
$("div").show("slow");
```

```javascript
// show(毫秒值，回调函数):动画执行完后，立即执行回调函数
$("div").show(5000,function () {
        alert("动画执行完毕！");
    });
```



## 2.2 隐藏动画

参照上面的`show()`方法使用：

```javascript
$(selector).hide();
$(selector).hide(1000);
$(selector).hide("slow");
$(selector).hide(1000, function(){});
```



## 2.3 切换显示和隐藏

参照上面的`show()`方法使用：

```javascript
$(selector).toggle(); //也有四种方式
```



## 2.4 滑入&滑出

以下方法中，如果省略参数或者传入不合法的字符串，那么则使用默认值：400毫秒。

- 滑入：

  ```javascript
  $(selector).slideDown(毫秒数, 回调函数);//下拉动画，显示元素.
  ```

- 滑出：

  ```javascript
  $(selector).slideUp(毫秒数, 回调函数);//上拉动画，隐藏元素
  ```

- 滑入、滑出切换：

  ```javascript
  $(selector).slideToggle(毫秒数, 回调函数) //也有四种方式
  ```


## 2.5 淡入&淡出

通过改变透明度，切换匹配元素的显示或隐藏状态。参数含义同`show()`方法！

- 淡入：

  ```javascript
  $(selector).fadeIn(speed, callback)
  ```

- 淡出：

  ```javascript
  $(selector).fadeOut(1000)
  ```

- 淡入淡出切换：

  ```javascript
  $(selector).fadeToggle('fast', callback)
  ```



## 2.6 自定义动画

执行一组CSS属性的自定义动画。

- 语法：`$(selector).animate({params}, speed, callback)`，属性：
  - 第一个参数表示：要执行动画的CSS属性（必选）
  - 第二个参数表示：执行动画时长（可选）
  - 第三个参数表示：动画执行完后，立即执行的回调函数（可选）



## 2.7 停止动画

- 语法：`$(selector).stop(true, false)`，属性：默认两个都是false
  - 第一个参数：后续动画是否执行
  - 第二个参数：true表示立即执行完成当前动画；false表示立即停止当前动画

当调用`stop()`方法后，队列里面的下一个动画将会立即开始。 但是，如果参数clearQueue被设置为true，那么队列面剩余的动画就被删除了，并且永远也不会执行。

如果参数jumpToEnd被设置为true，那么当前动画会停止，但是参与动画的每一个CSS属性将被立即设置为它们的目标值。比如：slideUp()方法，那么元素会立即隐藏掉。如果存在回调函数，那么回调函数也会立即执行。

注意：如果元素动画还没有执行完，此时调用sotp()方法，那么动画将会停止。并且动画没有执行完成，那么回调函数也不会被执行。



# 3.操作DOM

- 操作的样式非常少，那么可以通过`.css()`实现。
- 操作的样式很多，建议通过使用类 class 的方式来操作。
- 如果考虑以后维护方便（把CSS从js中分离出来）的话，推荐使用类的方式来操作。



## 3.1 CSS操作

### 样式操作

- 操作CSS样式：`css()`的四种用法

  ```javascript
  $("p").css("color")   //获取第一个p元素的color值
  $("p").css(         //设置所有o元素的字体颜色为红色且背景为蓝色
      {
          color: "red",
          backgroud: "blue"
      }
  )
  $("p").css("color","red")  //设置所有段落字体为红色
  //主键增加div的大小
  $("div").click(function() {
      $(this).css({
          width: function(index, value) {
              return parseFloat(value) * 1.2;
          }, 
          height: function(index, value) {
              return parseFloat(value) * 1.2;
          }
      });
  });
  ```

其他方法见[中文API](http://tool.oschina.net/apidocs/apidoc?api=jquery)



### 尺寸操作

```javascript
$(selector).height();     //不带参数表示获取高度
$(selector).height(200);  //带参数表示设置高度

$(selector).width();     //不带参数表示获取宽度
$(selector).width(200);  //带参数表示设置高宽度
```

Q：jQuery的`css()`获取高度，和jQuery的height获取高度，二者的区别？

```javascript
$("div").css();     //返回的是string类型，例如：30px
$("div").height();  //返回得失number类型，例如：30。常用于数学计算。
```



### 位置操作

- `offset()`：获取或设置元素相对于 document 文档的位置。参数：
  - 无参数：表示获取。返回值为：`{left:num, top:num}`。返回值是相对于document的位置。
  - 有参数：表示设置。参数建议使用 number 数值类型。
  - 设置offset后，如果元素没有定位(默认值：static)，则被修改为relative

```javascript
$(selector).offset()
$(selector).offset({left:100, top: 150})
```

- `position()`：获取相对于其最近的**带有定位**的父元素的位置。返回值为对象：`{left:num, top:num}`

```javascript
$(selector).position()  //只能获取，不能设置
```

- `scrollTop()`：获取匹配元素相对滚动条顶部的偏移。参数：
  - 无参数：表示获取偏移
  - 有参数：表示设置偏移，参数为数值类型

```javascript
scrollTop();
$(selector).scrollTop(100);
```

- `scrollLeft()`：获取或者设置匹配元素相对滚动条左侧的偏移。参数：
  - 无参数：表示获取偏移
  - 有参数：表示设置偏移，参数为数值类型

```javascript
scrollLeft();
$(selector).scrollLeft(100);
```





## 3.2类操作

- 添加类样式：

  ```javascript
$(selector).addClass("类名")  //为指定元素添加类
  ```

- 移除类样式：

  ```javascript
  $(selector).removeClass("类名");  //为指定元素移除类
  $(selector).removeClass();          //不指定参数，表示移除被选中元素的所有类
  ```

- 判断有没有指定类样式：

  ```javascript
  $(selector).hasClass("类名");   //判断指定元素是否包含类
  ```

- 切换类样式：

  ```javascript
  $(selector).toggleClass(“类名”); //为指定元素切换类，该元素有类则移除，没有指定类则添加
  ```



## 3.3 节点操作

### 动态创建元素

注意：创建的是jQuery对象！

方式一：

```javascript
var $spanNode1 = $("<span>我是一个span元素</span>");  // 返回的是 jQuery对象
//类似于原生js中的document.createElement("标签名")
```

方式二（推荐！！！）：

```javascript
var node = $("#box").html("<li>我是li</li>");
//类似原生js中的innerHTML
```



### 添加元素

jQuery添加元素的方法很多，最重要的是`append()`，在盒子里的末尾添加元素

```javascript
//方式一：在$(selector)中追加$node。参数是jQuery对象
$(selector).append($node)

//方式二：在$(selector)中追加div元素。参数是htmlString
$(selector).append('<div></div>')
```

- 如果是页面中存在的元素，那调用`append()`后，会把这个元素放到相应的目标元素里面去；但是，原来的这个元素，就不存在了。
- 如果是给多个目标追加元素，那么方法的内部会复制多份这个元素，然后追加到多个目标里面去。

示例：

```javascript
$("button").click(function () {
    //创建一个新的jquery对象li
    var jqNewLi = $("<li>我是jquery创建出来的li。用的是append方法添加</li>");

    //append();  在盒子里的最末尾添加与严肃
    $("ul").append(jqNewLi);    //把新创建的 li 塞进已知的 ul 中
});
```



- 其他添加元素的方法：
  - `$(selector).appendTO(node)`：将`$(selector)`添加到node中
  - `$(selector).prepend(node)`：在元素的第一个子元素前面追加内容或节点
  - `$(selector).after(node)`：在被选元素之后，作为**兄弟元素**插入内容或节点
  - `$(selector).before(node)`：在被选元素之前，作为**兄弟元素**插入内容或节点



### 清空&复制元素

- 清空元素：

```javascript
//清空指定元素的所有子元素
$(selector).empty();
$(selector).html("")   //推荐！！！！
//把自己以及所有的内部元素从文档中删除掉
$(selector).remove()
```

- 复制元素：

```javascript
复制的新元素 = $(selector).clone()  //深层复制
```



## 3.4 示例

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        select {
            width: 170px;
            height: 240px;
            font-size: 18px;
            background-color: #a4ff34;
        }
    </style>
    <script src="jquery-1.12.4.js"></script>
    <script>
        $(function () {

            //步骤：
            //1.将左侧的子标签全部移动到右侧。
            $("button").eq(0).click(function () {
                //右侧标签.append(左侧标签);
                $("#sel2").append($("#sel1 option"));
            });

            //2.将右侧的子标签全部移动到左侧。
            $("button").eq(1).click(function () {
                //左侧标签.append(右侧标签);
                $("#sel1").append($("#sel2 option"));
            });

            //第二步：获取子元素的时候要注意，获取的必须是，被选中的元素。
            //技术点：怎么获取被选中的子元素呢？？？答案：option:selected;
            //1.将左侧被选中的子标签移动到右侧
            $("button").eq(2).click(function () {
                //右侧标签.append(左侧标签);
                $("#sel2").append($("#sel1 option:selected"));
            });

            //2.将右侧被选中的子标签移动到左侧
            $("button").eq(3).click(function () {
                //右侧标签.append(左侧标签);
                $("#sel1").append($("#sel2 option:selected"));
            });

        })
    </script>
</head>
<body>
    <select id="sel1" size="10" multiple>
        <option value="0">香蕉</option>
        <option value="1">苹果</option>
        <option value="2">鸭梨</option>
        <option value="3">葡萄</option>
    </select>
    <button>>>></button>
    <button><<<</button>
    <button>></button>
    <button><</button>
    <select id="sel2" size="10" multiple>

    </select>
</body>
</html>
```



## 3.5 属性操作

jQuery 无法直接操作节点的属性和src等，我们需要借助`attr()`方法。

`attr()`方法：两个参数是给属性赋值，单个参数是获取属性值

- 设置属性：

```javascript
$(selector).attr("title", "生命壹号")
//第一个参数表示：要设置的属性名称
//第二个参数表示：该属性名称对应的值
```

- 获取属性：

```javascript
$(selector).attr("title")
```



- 移除属性：

```javascript
$(selector).removeAttr("要移除的属性名")
```

- form表单中的 `prop()`方法：
  - 针对`checked、selected、disabled`属性，要使用 `prop()`方法，而不是其他的方法。
  - prop方法通常用来影响DOM元素的动态状态，而不是改变的HTML属性。例如：input和button的disabled特性，以及checkbox的checked特性。



## 3.6 其他

- `$(selector).val()`：设置或返回 form 表单元素的value值
- `$(selector).text()`：设置或获取匹配元素的文本内容。不带参数表示，会把所有匹配到的元素内容拼接为一个**字符串**，不同于其他获取操作。
- `$(selector).text("我是内容")`：设置的内容包含html标签，那么text()方法会把他们当作**纯文本**内容输出

注意：

- `text()`不识别标签
- `html()`识别标签



# 4.事件机制

- 常见的事件绑定：
  - `click(handler) `单击事件。
  - `blur(handler)` 失去焦点事件。
  - `mouseenter(handler)` 鼠标进入事件。
  - `mouseleave(handler)`	鼠标离开事件。
  - `dbclick(handler)` 双击事件。
  - `change(handler)` 改变事件，如：文本框值改变，下拉列表值改变等。
  - `focus(handler)` 获得焦点事件。
  - `keydown(handler) `键盘按下事件。



## 4.1 事件绑定

- jQuery中事件绑定有两种方式：

  - `eventName(function()P{})`绑定对应事件名的监听，如：

    ```javascript
    $("#div").click(function(){   })
    ```

  - on方式绑定，如：

    ```javascript
    $("#div").on("click",function(){   })
    ```

    ```javascript
    $(document).on("click mouseenter", ".box", {"name": 111}, function (event) {
        console.log(event.data);      //event.data获取的就是第三个参数这个json。
        console.log(event.data.name); //event.data.name获取的是name的值。
    });
    ```

    - 参数：
      - 第一个参数：events，绑定事件的名称可以是由空格分隔的多个事件（标准事件或者自定义事件）。上方代码绑定的是单击事件和鼠标进入事件。
      - 第二个参数：selector, 执行事件的后代元素。
      - 第三个参数：data，传递给事件处理函数的数据，事件触发的时候通过event.data来使用（也就是说，可以通过event拿到data）
      - 第四个参数：handler，事件处理函数。



## 4.3 off方式解绑

- `$(selector).off()`：解绑匹配元素的所有事件
- `$(selector).off("click")`：解绑匹配元素的所有click事件
- `$(selector).off( “click”, "**" )`：解绑所有代理的click事件，元素本身的事件不会被解绑



## 4.4 事件对象

- jQuery的事件对象：
  - `event.data` 传递给事件处理程序的额外数据
  - `event.currentTarget` 等同于this，当前DOM对象
  - `event.pageX` 鼠标相对于文档左部边缘的位置
  - `event.target` 触发事件源，不一定===this
  - `event.stopPropagation()`； 阻止事件冒泡
  - `event.preventDefault()`; 阻止默认行为
  - `event.type` 事件类型：click，dbclick…
  - `event.which` 鼠标的按键类型：左1 中2 右3
  - `event.keyCode` 键盘按键代码



# 5.零散

## 5.1 多库共存

**多库共存**指的是：jQuery占用了 `$` 和 `jQuery` 这两个变量。当在同一个页面中引用了 jQuery 库以及其他的库（或者其他版本的jQuery库），恰好其他的库中也用到了 `$` 或者`jQuery`变量.那么，要保证每个库都能正常使用，就产生了多库共存的问题。

解决方案：

- 方法一：让 jQuery 放弃对 `$` 的使用权：`$.noConflict()`，释放后就只能使用`jQuery`

- 方法二：同时放弃放弃两个符号的使用权，并定义一个新的使用权（如果有三个库时，可以这样用）

  ```javascript
  var fS=$.noConflict(true);   //此时 fS 就是新的符号了
  ```



## 5.2 插件

jQuery 是通过插件的方式，来扩展它的功能：

- 当你需要某个插件的时候，你可以“安装”到jQuery上面，然后使用。
- 当你不再需要这个插件，那你就可以从jQuery上“卸载”它。

```html
<script src="jquery-1.12.4.js"></script>
<!--加入插件，注意：放在jQuery后面-->
<script src="jquery.color.js"></script>
```



懒加载：当打开一个网页时，只有当我看到某个部分，再加载那个部分；而不是一下子全部加载完毕。这样可以优化打开的速度。

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
    <style>
        div {
            height: 3000px;
            background-color: pink;
        }
    </style>
    <script src="jquery-1.12.4.js"></script>
    <!--懒加载的使用。第一步：导包(必须在jquery库的下方）-->
    <script src="jquery.lazyload.js"></script>
    <script>
        $(function () {
            //第二步：调用懒加载的方法实现功能。参数的不同，功能也不同。
            $("img.lazy").lazyload();
        })
    </script>
</head>
<body>
    <div></div>
    <!--需要实现将图片设置为懒加载模式-->
    <img class="lazy" data-original="images/01.jpg" width="640" height="480">
</body>
</html>
```



# 6.Ajax

AJAX = 异步 JavaScript 和 XML（Asynchronous JavaScript and XML），可以在不重载整个网页的情况下，AJAX 通过后台加载数据，并在网页上进行显示。

jQuery 提供多个与 AJAX 有关的方法。

通过 jQuery AJAX 方法，您能够使用 HTTP Get 和 HTTP Post 从远程服务器上请求文本、HTML、XML 或 JSON ，同时能够把这些外部数据直接载入网页的被选元素中。



## 6.1 jQuery加载

- jQuery `load()`方法：从服务器加载数据，并把返回的数据放入被选元素中

  ```javascript
  $(selector).load(URL,data,callback);
  ```

  - 必需的URL参数规定您希望加载的 URL。
  - 可选的data参数规定与请求一同发送的查询字符串键/值对集合。
  - 可选的callback参数是`load()`方法完成后所执行的函数，回调函数可以设置不同的参数：
    - responseTxt：包含调用成功时的结果内容
    - statusTXT：包含调用的状态
    - xhr：包含 XMLHttpRequest 对象

示例:

```javascript
 //把文件 "demo_test.txt" 的内容加载到指定的 <div> 元素中
$("#div1").load("demo_test.txt") 

//把 "demo_test.txt" 文件中 id="p1" 的元素的内容，加载到指定的 <div> 元素中
$("#div1").load("/example/jquery/demo_test.txt #p1") 
```



```javascript
$("button").click(function(){
  $("#div1").load("demo_test.txt",function(responseTxt,statusTxt,xhr){
    if(statusTxt=="success")
      alert("外部内容加载成功！");
    if(statusTxt=="error")
      alert("Error: "+xhr.status+": "+xhr.statusText);
  });
});
```



## 6.2 get()和post()方法

- `$.get()`：通过 HTTP GET 请求从服务器上请求数据，语法：

  ```javascript
  $.get(URL,callback)
  //必需的 URL 参数规定您希望请求的 URL。
  //可选的 callback 参数是请求成功后所执行的函数。
  ```

  - 示例：

  ```javascript
  $("button").click(function(){
      $.get("demo_test.asp",function(data,status){
          alert("Data: " + data + "\nStatus: " + status);
      });
  });
  ```


- `$.post()`：通过 HTTP POST 请求从服务器上请求数据。语法：

  ```javascript
  $.post(URL,data,callback)
  //必需的 URL 参数规定您希望请求的 URL。
  //可选的 data 参数规定连同请求发送的数据。
  //可选的 callback 参数是请求成功后所执行的函数名。
  ```

  - 示例：

  ```javascript
  $("button").click(function(){
      $.post("demo_test_post.asp",{
          name:"Donald Duck",
          city:"Duckburg"
      	},
             function(data,status){
          	//第一个回调参数存有被请求页面的内容
          	//第二个参数存有请求的状态
          alert("Data: " + data + "\nStatus: " + status);
      });
  });
  ```



## 6.3 $.ajax

提交表单的数据：

1. 给表单的提交按钮绑定点击事件

   ```html
   <a onclick="dologin()" > 登录</a>
   ```

2. 在事件驱动程序中使用Ajax提交数据：

   ```javascript
   //注意：layer是layer弹层插件提供的对象
   <script>
       function dologin() {
           //表单数据的非空校验
           var loginacct = $("#loginacct").val();  //获取表单对应的数据
           //表单元素的value不可能为null，如果不输入，则value为空字符串！！！
           if(loginacct == ""){
               layer.msg("用户的账号不能为空，请输入！",{time:1000, icon:5, shift:6},function () {
   
               });
               return;
           }
   
           var userpwd = $("#userpwd").val();
           if(userpwd == ""){
               layer.msg("用户的密码不能为空，请输入！",{time:1000, icon:5, shift:6},function () {
                   
               });
               return;
           }
           //提交表单，以前是$("#loginForm").submit();
           //使用Ajax提交数据
           var loadingIndex = null;
           $.ajax({
               type: "POST",
               url: "doAjaxLogin",
               data: {
                   loginacct: loginacct,
                   userpwd: userpwd
               },
               //发送数据前显示一个“处理中”，提升用户体验
               beforeSend: function () {
                   loadingIndex = layer.msg('处理中', {icon: 16});
               },
               //处理完成后，关闭“处理中”的显示
               success: function (result) {
                   layer.close(loadingIndex);
                   if(result.success){
                       window.location.href = "main";//跳转的url为：main
                   }else {
                       layer.msg("登录账号或密码错误，请重新登录",{time:1000, icon:5, shift:6},function () {
   
                       });
                   }
               }
           });
       }
   </script>
   ```

`$.ajax()`函数通常提供JSON数据：

```js
{
    type: "POST",  //提交方式
    url: "doAjaxLogin",  //提交的URL
    data: {   //提交的数据
        loginacct: loginacct,
        userpwd: userpwd
    },
    beforeSend: function () { }  //发送数据前显示一个“处理中”，提升用户体验
    success: function (result) {  } //服务器处理完成返回给浏览器结果后执行的操作
}
```



## 6.4 jQuery Ajax函数

| 函数 | 描述 |
| ---- | ---- |
|`jQuery.ajax()`	|执行异步 HTTP (Ajax) 请求|
|`.ajaxComplete()`	|当 Ajax 请求完成时注册要调用的处理程序。这是一个 Ajax 事件|
|`.ajaxError()`	|当 Ajax 请求完成且出现错误时注册要调用的处理程序。这是一个 Ajax 事件|
|`.ajaxSend()`	|在 Ajax 请求发送之前显示一条消息|
|`jQuery.ajaxSetup()`	|设置将来的 Ajax 请求的默认值|
|`.ajaxStart()`	|当首个 Ajax 请求完成开始时注册要调用的处理程序。这是一个 Ajax 事件|
|`.ajaxStop()`	|当所有 Ajax 请求完成时注册要调用的处理程序。这是一个 Ajax 事件|
|`.ajaxSuccess()`	|当 Ajax 请求成功完成时显示一条消息|
|`jQuery.get()`	|使用 HTTP GET 请求从服务器加载数据|
|`jQuery.getJSON()`	|使用 HTTP GET 请求从服务器加载 JSON 编码数据|
|`jQuery.getScript()`	|使用 HTTP GET 请求从服务器加载 JavaScript 文件，然后执行该文件|
|`.load()`	|从服务器加载数据，然后把返回到 HTML 放入匹配元素|
|`jQuery.param()`	|创建数组或对象的序列化表示，适合在 URL 查询字符串或 Ajax 请求中使用|
|`jQuery.post()`	|使用 HTTP POST 请求从服务器加载数据|
|`.serialize()`	|将表单内容序列化为字符串|
|`.serializeArray()`	|序列化表单元素，返回 JSON 数据结构数据|



# 7.插件

## 7.1 layer

[layer弹层组件](http://layer.layui.com/)。基本用法：下载完成后，将文件夹放入项目中，在页面引入layer.js即可。需先引入jQuery！

```javascript
/*提示操作:
	time是弹出的信息显示多长事件；
	icon是弹出信息时的图标
	shift是弹出信息的效果*/
layer.msg(提示信息, {time:1000, icon:5, shift:6}, 回调方法);
layer.alert(提示信息, function(index){
    // 回调方法
    layer.close(index);
});

//询问操作：
layer.confirm("询问信息",  {icon: 3, title:'提示'}, function(cindex){
    layer.close(cindex);
}, function(cindex){
    layer.close(cindex);
});

//加载操作：
var loadingIndex = layer.msg('处理中', {icon: 16});
...
layer.close(loadingIndex);

var index = layer.load(2, {time: 10*1000});
layer.close(index);
```



## 7.2 展示数据

在程序开发中，展示数据有很多种方式，可以采用表格，图表，树形结构等不同的方式，具体采用哪一种，取决于我们想要展示数据的哪一种特性。

如果想要展示**数据的先后顺序**，那么就采用**表格**；

如果想要展示**数据的对比关系**，那么就采用**图表**；

如果想要展示**数据的层次（上下级）关系**，那么就采用**树形结构**。



### 树形结构 zTree

树形结构的数据，应该遵循从上至下，从父到子进行操作的原则。也就是说通过父节点数据操作子节点数据。一个基本的树形结构的数据，根节点数据只能有一个，但是分支节点，叶子节点可以存在多个。

网页设计中，如果想将数据以树形结构的方式展示，一般采用`<ul>` `<li>`标签的嵌套使用方式实现。基本代码为：

```html
    <ul>
        <li> 父节点
            <ul>
                <li>子节点</li>
            </ul>
        </li>
    </ul>
```

如果所有的`HTML`代码都是我们来设计和开发，会比较麻烦，而且容易出现错误，所以，一般在开发中，我们都采用第三方树形组件实现，常用的树形结构插件：

| 组件            | 优点                                                         | 缺点                                                   |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------ |
| JSTree          | 兼容多个浏览器                                               | 代码风格不是OOP                                        |
| TreeView        | 基于Jquery的轻量性、可扩展性强的树控件                       | 动态操作的功能较少                                     |
| zTree           | 优异的性能、灵活的配置、多种功能的组合                       | 右键菜单无法直接集成。代码封装比较完整，扩展功能不容易 |
| ExtJS TreePanel | ExtJS自身使用的树控件, 支持自由拖放, 有非常完善的API和开发文档 | 商业项目是需要购买license                              |



[zTree](http://www.treejs.cn/v3/main.php#_zTreeInfo)是一个依靠 jQuery 实现的多功能 “树插件”，最大优点是优异的性能、灵活的配置、多种功能的组合。

- 1.引入组件的样式和脚本：必须先引入jQuery

  ```html
  <link rel="stylesheet" href="ztree/zTreeStyle.css">
  <script src="ztree/jquery.ztree.all-3.5.min.js"></script>
  ```

- 2/渲染数据：

  ```html
  <!--2-1 在页面中增加容器标签，用于容纳生成后的树形结构内容-->
  <ul id="treeDemo" class="ztree">
      
  <!--2-2 调用组件方法在页面中渲染数据-->
  <script type="text/javascript">
      var setting = {};  //见后面
      var jsonData = [   //见后面
          {"id":1,"name":"系统权限菜单","open":true,
              "children":[
                  {"id":2,"name":"控制面板","open":true,"checked":false,
                      "children":[]
                  }
              ]
          }
      ];
      $.fn.zTree.init($("#treeDemo"), setting, jsonData);
  </script>
  ```



- Setting对象表示组件渲染数据的配置信息，需要在渲染数据之前准备好，其中包含很多的属性，如：

  | 属性     | 名称     | 例子                                                         |
  | -------- | -------- | ------------------------------------------------------------ |
  | 事件     | callback | callback: { <br/>	onClick : function(event, treeId, json) { 	}<br/> } |
  | 异步加载 | async    | async: { 	<br/>        enable: true, <br/>	url:"xxxxx", <br/>	autoParam:["id", "name=n", "level=lv"] <br/>} |
  | 复选框   | check    | check : {<br/>     enable : true  <br/>}                     |

- jsonData的含义：

  ```json
  var node = {
      name :  ‘xxxx’ , // 节点名称
      open: true,	 // 父节点是否展开
      icon : ‘xxx’,	// 图标图片的 url 可以是相对路径也可以是绝对路径
      checked:true,	// 节点的 checkBox / radio 的 勾选状态
      children:[{}, {}] 	//节点的子节点数据集合
      level:1		//记录节点的层级(不能由后台生成，是ztree组件自动生成的)
  }
  ```

- 基本操作：

  | 操作                                               | 代码                                                         |
  | -------------------------------------------------- | ------------------------------------------------------------ |
  | 读取当前树对象                                     | `var treeObj = $.fn.zTree.getZTreeObj("treeDemo");`          |
  | 刷新当前树对象的数据(异步，树结构数据必须异步加载) | `treeObj.reAsyncChildNodes(null, "refresh");`                |
  | 查询节点                                           | `var nodes = treeObj.getSelectedNodes(); //获取选择的节点`<br/> `var node  = treeObj.getNodeByParam("id", 1, null);//根据参数获取节点`<br/> `var nodes = treeObj.getCheckedNodes(true); //获取被选中的节点` |
  | 增加新节点                                         | `var newNodes = [{name:"newNode1"}, {name:"newNode2"}, {name:"newNode3"}]; newNodes = treeObj.addNodes(node, newNodes);` |
  | 修改数据                                           | var treeObj = $.fn.zTree.getZTreeObj("tree"); <br/>var nodes = treeObj.getNodes(); <br/>if (nodes.length>0) {<br/> 	nodes[0].name = "test"; <br/>	treeObj.updateNode(nodes[0]);<br/> } |
  | 删除节点                                           | treeObj.removeChildNodes(parentNode); <br/>treeObj.removeNode(currentNode); |




在数据库中可以使用一张表来表示父子节点(1对多)：让pid关联id

| id   | name     | pid(父节点id) |
| ---- | -------- | ------------- |
| 1    | 系统菜单 | 0             |
| 2    | 控制面板 | 1             |
| 3    | 权限管理 | 1             |
| 4    | 用户维护 | 3             |
| 5    | 角色维护 | 3             |
| 6    | 许可维护 | 3             |

当从数据库读取节点数据来展示时，可使用递归来读取，但效率不高。



### 图标结构 ECharts

[ECharts](http://www.echartsjs.com/index.html)是一个使用 JavaScript 实现的开源可视化库，提供了常规的[折线图](http://www.echartsjs.com/option.html#series-line)、[柱状图](http://www.echartsjs.com/option.html#series-line)、[散点图](http://www.echartsjs.com/option.html#series-scatter)、[饼图](http://www.echartsjs.com/option.html#series-pie)、[K线图](http://www.echartsjs.com/option.html#series-candlestick)，用于统计的[盒形图](http://www.echartsjs.com/option.html#series-boxplot)，用于地理数据可视化的[地图](http://www.echartsjs.com/option.html#series-map)、[热力图](http://www.echartsjs.com/option.html#series-heatmap)、[线图](http://www.echartsjs.com/option.html#series-lines)，用于关系数据可视化的[关系图](http://www.echartsjs.com/option.html#series-graph)、[treemap](http://www.echartsjs.com/option.html#series-treemap)、[旭日图](http://www.echartsjs.com/option.html#series-sunburst)，多维数据可视化的[平行坐标](http://www.echartsjs.com/option.html#series-parallel)，还有用于 BI 的[漏斗图](http://www.echartsjs.com/option.html#series-funnel)，[仪表盘](http://www.echartsjs.com/option.html#series-gauge)，并且支持图与图之间的混搭。

使用：

- 引入组件：下载见官网

  ```html
  <script src="js/echarts.js"></script>
  ```

- 绘制一个简单的图表：

  ```html
  <!DOCTYPE html>
  <html>
  	<head>
  		<meta charset="utf-8" />
  		<!-- 引入资源文件 -->
  		<script src="js/echarts.js"></script>
  	</head>
  	<body>
  		<!-- 定义图标显示区域 -->
  		<div id="main" style="widows: 600px;height: 400px"></div>
  		
  		<script type="text/jscript">
  			<!-- 初始化Echarts对象 -->
  			var myChart = echarts.init(document.getElementById("main"));
  			<!-- 指定相关配置项(数据、样式) -->
  			var option = {
  				// 图表标题
  				title:{
  					text:"Echarts入门示例"
  				},
  				//图例
  				legend:{
  					data:["销量","成本"]
  				},
  				//x轴数据
  				xAxis:{
  					data:["衬衫","羊毛衫","裤子","上衣","袜子","鞋子"]
  				},
  				//y轴数据
  				yAxis:{
  					
  				},
  				//根据相应系列进行配置
  				series:[
  					{
  						name:"销量",
  						type:"bar",
  						data:[120, 200, 150, 80, 70, 110, 130]
  						
  					},
  					{
  						name:"成本",
  						type:"bar",
  						data:[12, 20, 15, 8, 7, 11, 13]
  					}
  				]
  			}
  			<!-- 使用刚指定的配置项和数据渲染图表 -->
  			myChart.setOption(option);
  		</script>
  	</body>
  </html>
  ```


