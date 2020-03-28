# JavaScript基础

JavaScript 可为网站添加交互功能，如游戏、动态样式、动画、按下按钮时做出的响应等。

# 1.语法

## 1.1代码位置

- 内嵌方式：把JS脚本放在`<head>`部分中，或放在`<body>`的底部：
```html
<head>
    <script type="text/javascript">

    </script>
</head>
```
- 引入外部js文件：把脚本保存在外部文件(扩展名为 .js)中，然后在网页中引用：
```html
<head>
    <script src="外部js文件"></script>
</head>
<!--注意：外部脚本不能包含<script>标签-->
```



## 1.2 JS输出

在HTML中，全局变量是 window 对象: 所有数据变量都属于 window 对象。

JS没有任何打印或输出的函数！可通过不同方式来输出数据：

- 使用 **window.alert() 弹出警告框**。
- 使用 **document.write() 方法将内容写到 HTML 文档中**。
  - 如果在文档已完成加载后执行`document.write()`，整个 HTML 页面将被覆盖。
- 使用 **innerHTML 写入到 HTML 元素**。
  - 写入元素首先要访问到该HTML元素，使用`document.getElementById(id)`可根据指定的id找到对应的元素
- 使用 **console.log() 写入到浏览器的控制台**。

```html
<body>
    <p id="demo">一个段落。</p>
    <button type="button" onclick="myFunction()">点击这里</button><br/>
    <script>
        window.alert(5+6);
        document.getElementById("demo").innerHTML = "段落已修改。";
        document.write(Date());
        console.log(5+6);<!--这里会向控制台打印数据，可通过浏览器的开发者工具查看-->
            function myFunction() {
            document.write(Date());
            <!--当点击了上面的按钮后，调用该函数，整个HTML页面都会被覆盖-->
        }
    </script>
</body>
```



## 1.3变量

JavaScript 变量均为对象。使用`var`关键字来声明变量!!

JS中使用`var`声明的变量就是局部变量；不使用`var`声明的变量就是全局变量

可以使用`typeof`运算符来获取变量的类型！

如果把值赋给尚未声明的变量，该变量将被自动作为全局变量声明。



### 数据类型

五种**基本数据类型**：
- String。字符串需要使用引号引起来。使用双引号或单引号都可以
    - 注意：引号不能嵌套：双引号里不能再放双引号，单引号里不能再放单引号。但是单引号里可以嵌套双引号
    - 在文本字符串中可以使用`\`来换行
    - 属性：

    | 属性        | 描述                       |
    | ----------- | -------------------------- |
    | constructor | 返回创建字符串属性的函数   |
    | length      | 返回字符串的长度           |
    | prototype   | 允许您向对象添加属性和方法 |
- Number
    - 最大值：`Number.MAX_VALUE`为： 1.7976931348623157e+308
    - 最小值：`Number.MIN_VALUE`为： 5e-324
    - 如果使用Number表示的变量超过了最大值，则会返回`Infinity`
        - 无穷大（正无穷）：Infinity
        - 无穷小（负无穷）：-Infinity
    - NaN：是一个特殊的数字，表示Not a Number，非数值
    - `isNaN()`:任何不能被转换为数值的值，都会让这个函数返回 true
- Boolean

    - 注意：0、-0、null、""、undefined、NaN也是false
- Null：一个只有一个值的特殊类型。表示一个空对象引用，可通过设置值为`undefined`来清空对象
- Undefined：一个没有设置值的变量，类型为`defined`

**引用类型**：
- Object:由花括号分隔。在括号内部，对象的属性以名称和值对的形式 (name : value) 来定义。属性由逗号分隔.
    - 注意：内置对象function、Array、Date、RegExp、Error等都是属于Object类型。也就是说，除了那五种基本数据类型之外，其他的，都称之为Object类型。


示例1：
```javascript
    typeof Infinity             //返回number
    typeof NaN                 //返回number
    var person = null;           // 值为 null(空), 但类型为对象
    var person = undefined;     // 值为 undefined, 类型为 undefined

    //undefined和null的区别：
    typeof undefined             // undefined
    typeof null                  // object
    null === undefined           // false
    null == undefined            // true
```



### 动态类型

JavaScript拥有动态类型：
```javascript
    var x;               // x 为 undefined
    var x = 5;           // 现在 x 为数字
    var x = "John";      // 现在 x 为字符串
```



### 类型转换

以下函数和运算符可进行类型转换：

`Number() `转换为数字， `String()`和`toString()`转换为字符串， `Boolean() `转化为布尔值，一元运算符`+`：

```javascript
    Number("3.14")    // 返回 3.14
    String(false)        // 返回 "false"
    //使用一元运算符+时,如果变量不能转换，它仍然会是一个数字，但值为 NaN (不是一个数字)
    var y = "5";      // y 是一个字符串
    var x = + y;      // x 是一个数字
```

- 使用String()函数做强制类型转换时：
    - 对于Number和Boolean而言，实际上就是调用toString()方法。
    - 对于null和undefined，就不会调用toString()方法。它会将 null 直接转换为 "null"。将 undefined 直接转换为 "undefined"。

- `parseInt()`：字符串转为整数
    - 只保留字符串最开头的数字，后面的中文自动消失；
    - 自动带有截断小数的功能：取整，不四舍五入
- `parseFloat()`：字符串转为浮点数

自动类型转换：

```javascript
    5 + null    // 返回 5         null 转换为 0
    "5" + null  // 返回"5null"   null 转换为 "null"
    "5" + 1     // 返回 "51"      1 转换为 "1"
    "5" - 1     // 返回 4         "5" 转换为 5
```



## 1.4 循环控制

JS中的代码块，只具有分组的作用，没有其他的用途。代码块中的内容，在外部是完全可见的。

### 判断

```javascript
    if (条件表达式) {
		// 条件为真时，做的事情
	} else {
		// 条件为假时，做的事情
	}
```



### for循环

for循环有两种：
- 一种与Java的用法相同:
    ```javascript
        for (var i = 1; i <= 100; i++) {
            console.log(i);
        }
    ```
- 一种用于遍历对象的属性:
    ```javascript
        var person={fname:"John",lname:"Doe",age:25};
        for (var x in person){
            txt=txt + person[x];
        }
    ```

也可用`break、continue`来跳出循环，用法同Java，也可以跳到标签处：

```html
    cars=["BMW","Volvo","Saab","Ford"];
    list:{
        document.write(cars[0] + "<br>");
        break list;
        document.write(cars[1] + "<br>");
    }
```



### switch

```javascript
    switch(n){
        case 1:
          执行代码块 1
          break;
        case 2:
          执行代码块 2
          break;
        default:
          n 与 case 1 和 case 2 不同时执行的代码
    }
```



### while和do while

```javascript
    while (条件){
      需要执行的代码
    }

    do{
      需要执行的代码
    }
    while (条件);
```



## 1.5 JS函数

function也是一个对象！
```javascript
    typeof function(){}			//返回function
```

- 函数定义：
    - 方式一：使用`function`关键字
    ```javascript
    function 函数名(变量1，变量2){
    	//执行代码
        var x=5;
        return x;//如果需要返回值，使用return语句
        //如果return语句后不跟任何值，就相当于返回一个undefined
        //如果函数中不写return，则也会返回undefined
    }
    ```
    - 方式二：使用函数表达式
    ```javascript
    var 函数名  = function([形参1,形参2...形参N]){
    		//语句....
    	}
    ```

- 函数调用：`函数名(参数...)`



### fn() 和 fn ★

- `fn()`：调用函数。相当于获取了函数的返回值。
- `fn`：函数对象。相当于直接获取了函数对象



### 自调用函数

如果表达式后面紧跟`()`，则会自动调用。

```javascript
    (function () {
      var x = "Hello!!";      // 我将调用自己
    })();
    //这是一个 匿名(没有函数名)自我调用的函数
```



### 函数参数

JS 函数对参数的值没有进行任何的检查。

- 显式参数(Parameters)与隐式参数(Arguments)：
  - 函数显式参数在函数定义时列出。
  - 在函数中调用的参数是函数的隐式参数。

- 默认参数：
  - 如果函数在调用时未提供隐式参数，参数会默认设置为： **undefined**

- 在调用函数时，浏览器每次都会传递两个隐含参数：
    - 函数的上下文对象 this。见后面。
    - 封装实参的 arguments
        - 是一个类数组对象，它也可以通过索引来操作数据，也可以获取长度。

```javascript
    function foo() {
        console.log(arguments);
        console.log(typeof arguments);
    }
```



## 1.6 正则表达式RegExp

JS中正则表达式是放在`/`和`/`之间的！

在 JavaScript 中，正则表达式通常用于两个字符串方法 : search() 和 replace()。

**search() 方法** 用于检索字符串中指定的子字符串，或检索与正则表达式相匹配的子字符串，并返回子串的起始位置。

**replace() 方法** 用于在字符串中用一些字符替换另一些字符，或替换一个与正则表达式匹配的子串。

```javascript
var str = "Visit Openketangb!";
var n = str.search(/Openketang/);   //6
```

```javascript
var str = document.getElementById("demo").innerHTML;
var txt = str.replace(/microsoft/,"Openketang");
```

- **test()**方法，是一个正则表达式方法
  - 用于检测一个字符串是否匹配某个模式，如果字符串中含有匹配的文本，则返回 true，否则返回 false。

```javascript
var patt = /e/;
patt.test("The best things in life are free!");  //true
```

- **exec()**方法，是一个正则表达式方法
  - 用于检索字符串中的正则表达式的匹配。该函数返回一个数组，其中存放匹配的结果。如果未找到匹配，则返回值为 null。

```javascript
/e/.exec("The best things in life are free!");   //e
```



## 1.7 JavaScript错误

**try** 语句测试代码块的错误。

**catch** 语句处理错误。

```javascript
try{
    //运行代码
}catch(err){
    //处理错误
}
```

案例：

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8"/>
	<script>
		var txt="";
		function message(){
			try {
				adddlert("Welcome guest!");
			}
			catch(err) {
				txt="本页有一个错误。\n\n";
				txt+="错误描述：" + err.message + "\n\n";
				txt+="点击确定继续。\n\n";
				alert(txt);
			}
		}
	</script>
</head>
<body>
	<input type="button" value="查看消息" onclick="message()" />
</body>
</html>
```

**throw** 语句创建自定义错误。

案例：

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8"/>
</head>
<body>

	<p>请输出一个 5 到 10 之间的数字:</p>

	<input id="demo" type="text">
	<button type="button" onclick="myFunction()">测试输入</button>
	<p id="message"></p>

	<script>
	function myFunction() {
		var message, x;
		message = document.getElementById("message");
		message.innerHTML = "";
		x = document.getElementById("demo").value;
		try {
			if(x == "")  throw "值为空";
			if(isNaN(x)) throw "不是数字";
			x = Number(x);
			if(x < 5)    throw "太小";
			if(x > 10)   throw "太大";
		}
		catch(err) {
			message.innerHTML = "错误: " + err;
		}
	}
	</script>
</body>
</html>
```



## 1.8 JavaScript表单

HTML表单验证可通过JS完成。

```html
<form name="myForm" action="demo_form.php" onsubmit="return validateForm()" method="post">
    名字: <input type="text" name="fname">
    <input type="submit" value="提交">
</form>
```

JS验证：

```javascript
function validateForm() {
    var x = document.forms["myForm"]["fname"].value;
    if (x == null || x == "") {
        alert("需要输入名字。");
        return false;
    }
}
```



- JS验证API：约束验证DOM方法：

| Property            | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| checkValidity()     | 如果 input 元素中的数据是合法的返回 true，否则返回 false。   |
| setCustomValidity() | 设置 input 元素的 validationMessage 属性，用于自定义错误提示信息的方法。 |

```html
<body>
    <p>输入数字并点击验证按钮:</p>

    <input id="id1" type="number" min="100" max="300" required>
    <button onclick="myFunction()">验证</button>
    <p>如果输入的数字小于 100 或大于300，会提示错误信息。</p>
    <p id="demo"></p>

    <script>
    function myFunction() {
        var inpObj = document.getElementById("id1");
        if (inpObj.checkValidity() == false) {
            document.getElementById("demo").innerHTML = inpObj.validationMessage;
        } else {
            document.getElementById("demo").innerHTML = "输入正确";
        }
    }
    </script>
</body>
```

- 约束验证DOM属性

  | 属性              | 描述                                  |
  | ----------------- | ------------------------------------- |
  | validity          | 布尔属性值，返回 input 输入值是否合法 |
  | validationMessage | 浏览器错误提示信息                    |
  | willValidate      | 指定 input 是否需要验证               |



## 1.9 计时事件

在一个设定的时间间隔之后来执行代码，而不是在函数被调用后立即执行。我们称之为计时事件。

- `setInterval()`：间隔指定的毫秒数不停地执行指定的代码。

  ```javascript
  window.setInterval("javascript function",毫秒数); //可以不带window对象
  ```

  - `clearInterval()`方法用于停止`setInterval()`方法执行的函数代码。

    ```javascript
    window.clearInterval(intervalVariable);
    //要使用 clearInterval() 方法, 在创建计时方法时你必须使用全局变量：
    ```

  ```html
  <body>
      <p>页面上显示时钟：</p>
      <p id="demo"></p>
      <button onclick="myStopFunction()">停止时钟</button>
      <script>
          var myVar=setInterval(function(){myTimer()},1000);
          function myTimer(){
              var d=new Date();
              var t=d.toLocaleTimeString();
              document.getElementById("demo").innerHTML=t;
          }
          function myStopFunction(){
              clearInterval(myVar);
          }
      </script>
  </body>
  ```

- `setTimeout()`：暂停指定的毫秒数后执行指定的代码。会返回某个值，假如你希望取消这个`setTimeout()`，你可以使用这个变量名来指定它。

  ```javascript
  window.setTimeout("javascript 函数",毫秒数);
  ```

  - `clearTimeout()`方法用于停止执行`setTimeout()`方法的函数代码。

    ```javascript
    window.clearTimeout(timeoutVariable);
    //要使用clearTimeout() 方法, 你必须在创建超时方法中（setTimeout）使用全局变量:
    ```

  ```javascript
  var myVar;
  function myFunction(){
  	myVar=setTimeout(function(){alert("Hello")},3000);
  }

  function myStopFunction(){
  	clearTimeout(myVar);
  }
  ```



## 1.10 Cookie

创建Cookie：

```javascript
document.cookie="username=John Doe";
//为 cookie 添加一个过期时间（以 UTC 或 GMT 时间）。默认情况下，cookie 在浏览器关闭时删除：
document.cookie="username=John Doe; expires=Thu, 18 Dec 2013 12:00:00 GMT";
//使用 path 参数告诉浏览器 cookie 的路径。默认情况下，cookie 属于当前页面。
document.cookie="username=John Doe; expires=Thu, 18 Dec 2013 12:00:00 GMT; path=/";
```

读取Cookie：

```javascript
var x = document.cookie;
```

修改Cookie：类似于创建Cookie，会覆盖旧的Cookie

```javascript
document.cookie="username=John Smith; expires=Thu, 18 Dec 2013 12:00:00 GMT; path=/";
```

删除Cookie：设置 expires 参数为以前的时间即可

```javascript
document.cookie = "username=; expires=Thu, 01 Jan 1970 00:00:00 GMT";
```



```javascript
<script>
function setCookie(cname,cvalue,exdays){
	var d = new Date();
	d.setTime(d.getTime()+(exdays*24*60*60*1000));
	var expires = "expires="+d.toGMTString();
	document.cookie = cname+"="+cvalue+"; "+expires;
}
function getCookie(cname){
	var name = cname + "=";
	var ca = document.cookie.split(';');
	for(var i=0; i<ca.length; i++) {
		var c = ca[i].trim();
		if (c.indexOf(name)==0) return c.substring(name.length,c.length);
	}
	return "";
}
function checkCookie(){
	var user=getCookie("username");
	if (user!=""){
		alert("Welcome again " + user);
	}
	else {
		user = prompt("Please enter your name:","");
  		if (user!="" && user!=null){
    		setCookie("username",user,30);
    	}
	}
}
</script>
```


# 2.对象

在JS中，所有事物都是对象。

## 2.1 创建

方法一：使用`new`关键字来创建对象：
```javascript
    var obj = new Object();  //这里是调用构造函数来创建的对象
    obj.name = "bx";   //添加属性
    delete obj.name;   //删除属性
    //添加方法
    obj.方法名 = function(){
        //。。。
    };

    obj.方法名  //获取方法
    obj.方法名()   //调用方法
```

方法二：
```javascript
    var person={firstname:"John", lastname:"Doe", id:5566};
    //或
    var person={
      firstname : "John",
      lastname  : "Doe",
      id        :  5566
      方法名 : function() {
          //方法体
      }
    };
    //设置属性：
    person.id=23;
    //获取对象的属性：
    name=person.lastname;
    name=person["lastname"];
    //调用方法：
    name = person.方法名();
```

方法三：先创建构造函数，再调用其来创建对象
```javascript
    // 创建一个构造函数
    function Student(name) {
        this.name = name;
        this.sayHi = function () {
            console.log(this.name + "厉害了");
        }
    }

    //利用构造函数自定义对象
    var stu1 = new Student("smyh");
    console.log(stu1);
    stu1.sayHi();
```



## 2.2 零散

### in运算符

通过该运算符可以检查一个对象中是否含有指定的属性。如果有则返回true，没有则返回false。
```javasvript
    console.log("name" in obj);  //检查obj中是否含有name属性
```
可以通过 for in 来枚举对象中的属性，见for循环部分。



### 函数和方法

如果一个函数作为一个对象的属性保存，那么我们称这个函数是这个对象的方法。调用这个函数就说调用对象的方法（method）。但是它只是名称上的区别，没有其他的区别。



### 作用域

- 全局作用域
    - 直接编写在script标签中的JS代码，都在全局作用域。
    - 全局作用域在页面打开时创建，在页面关闭时销毁。
    - 在全局作用域中有一个全局对象window，它代表的是一个浏览器的窗口，它由浏览器创建我们可以直接使用。
    - 在全局作用域中，创建的变量都会作为window对象的属性保存。
    - 在全局作用域中，创建的函数都会作为window对象的方法保存。
- 函数作用域
    - 调用函数时创建函数作用域，函数执行完毕以后，函数作用域销毁。



### this★

解析器在调用函数每次都会向函数内部传递进一个隐含的参数，这个隐含的参数就是this，this指向的是一个对象，这个对象我们称为函数执行的 上下文对象。

- 根据函数的调用方式的不同，this会指向不同的对象：【重要】
    - 以函数的形式调用时，this永远都是window。比如fun();相当于window.fun();
    - 以方法的形式调用时，this是调用方法的那个对象
    - 以构造函数的形式调用时，this是新创建的那个对象
    - 使用call和apply调用时，this是指定的那个对象



## 2.3 数组

Array：一个数组中可以有不同变量类型
```javascript
    var cars=new Array();
    cars[0]="Saab";
    cars[1]="Volvo";
    cars[2]="BMW";
    //或
    var cars=new Array("Saab","Volvo","BMW");
    //或
    var cars=["Saab","Volvo","BMW"];
```

- 数组的类型是Object：

```javascript
    typeof [1,2,3,4]             // 返回 object
```



### 常用方法：

```javascript
    Array.isArray(被检测的值)  //判断是否为数组
    数组.toString()   //数组转为字符串，使用逗号分隔
    数组.join()   //将数组中的元素用符号连接成字符串。用参数指定连接符号，默认使用逗号连接
    数组1.concat(数组2)  //把参数(数组2)拼接到当前数组（原数组不会被修改）
    原数组.slice(开始位置index，结束位置index) //从当前数组中截取一个新的数组（不影响原来的数组。左开右闭区间）
    数组1.splice(起始索引index，需要操作的个数，弥补的值)  //删除当前数组的某些元素（原数组会被改变）
```

- 添加和删除
    - `数组.push(元素)`：在数组最后面插入项，返回新数组的长度
    - `数组.pop()`：取出数组中的最后一个元素，返回被删除的元素
    - `数组.unshift(元素)`：在数组最前面插入项，返回新数组的长度
    - `数组.shift()`：取出数组中的第一个元素，返回被删除的元素

- 反转和排序
    - `reverse()`：反转数组（返回值是反转后的数组，而且原数组也已经被反转了）
    - `sort()`：给数组排序，返回排序后的数组（排序的规则看参数）
        - 无参：按照数组元素的首字符对应的Unicode编码值，从小到大排列数组元素;
        - 带参：必须为函数（回调函数：callback）。这个回调函数中带有两个参数，代表数组中的前后元素
            - 如果返回值（a-b）为负数，a排b前面。
            - 如果返回值（a-b）等于0，不动。
            - 如果返回值（a-b）为正数，a排b后面。
    ```javascript
    从小到大排序后的数组 = 数组.sort(function(a,b){
    	                                  return a-b;
    	});
    ```



### 迭代方法

数组迭代方法包括：`every()`、`filter()`、`forEach()`、`map()`、`some()`。这几个方法不会修改原数组。

- `every()`：对数组中每一项运行回调函数，如果都返回true，every就返回true；如果有一项返回false，则停止遍历，此方法返回false。
    - `every()`方法的返回值是boolean值，参数是回调函数！
- `some()`：对数组中每一项运行回调函数，只要有一项返回true，则停止遍历，此方法返回true
- `filter()`：对数组中每一项运行回调函数，返回结果是true的项组成新的数组（返回值就是这个新的数组）
- `forEach()`：遍历数组。无返回值
- `map()`：对数组中每一项运行回调函数，返回该函数的结果，组成的新数组（返回值就是这个新的数组）



### 清空数组

```javascript
    array.splice(0);      //方式1：删除数组中所有项目
	array.length = 0;     //方式1：length属性可以赋值，在其它语言中length是只读
	array = [];           //方式3：推荐
```



## 2.4 内置对象

### Date
- Date的几种声明
    - `var date1 = new Date()`
    - `var date2 = new Date("2017/09/06 09:00:00")`
    - `var date4 = new Date(2017, 1, 27)`

- 方法：
    - getDate() 获取日 1-31
    - getDay() 获取星期 0-6（0代表周日）
    - getMonth() 获取月 0-11（1月从0开始）
    - getFullYear() 获取完整年份（浏览器都支持）
    - getHours() 获取小时 0-23
    - getMinutes() 获取分钟 0-59
    - getSeconds() 获取秒 0-59
    - getMilliseconds() 获取毫秒 （1s = 1000ms）
    - getTime () 返回累计毫秒数(从1970/1/1午夜)


内置对象Array、Date无法通过`typeof`来判断其类型，因为返回都是object，可以使用`constructor`属性来判断：
```javascript
    function isArray(myArray) {
        return myArray.constructor.toString().indexOf("Array") > -1;
    }
    function isDate(myDate) {
        return myDate.constructor.toString().indexOf("Date") > -1;
    }
```



### String

简单数据类型string是无法绑定属性和方法的，String是可以的。

- 字符串的部分方法
    - `slice(索引1，索引2)`：左闭右开区间。
        - 如果只有一个参数，且参数大于0，则从该索引截取到最后
        - 如果只有一个参数，且参数小于0，则从倒数第几个截取到最后
        - 索引1>索引2，返回值为空
    - `substr(索引值, 长度)`
    - `trim()`：去除字符串前后的空白。
    - `replace()`：替换。
    - `split()`：字符串变数组



### Math

常见方法：
- Math.abs(); 取绝对值
- Math.floor(); 向下取整（向小取）
- Math.ceil(); 向上取整（向大取）
- Math.round(); 四舍五入取整（正数四舍五入，负数五舍六入）
- Math.random(); 随机数0-1



# 3.DOM

DOM(Document Object Model)文档对象模型。操作网页上的元素的API。比如让盒子移动、变色、轮播图等。

## 3.1 事件

- 事件的三要素：
    - 事件源：引发后续事件的html标签
    - 事件：可以是浏览器行为，也可以是用户行为。如：鼠标单击等
    - 事件驱动程序：事件发生后的响应：对样式和html的操作
    - 如：点击广告右上角的x，广告关闭。事件源为x，事件为鼠标点击，事件驱动程序是广告关闭了。

- 代码书写步骤：
    1. 获取事件源
    2. 绑定事件
    3. 书写事件驱动程序



### 获取事件源

获取事件源的常用方式：

- `var x=document.getElementById("intro")`：通过id查找。不存在则返回null。该方法仅用于document对象
- `getElementsByTagName("p")`：通过标签名查找，返回一个节点集合。
    ```javascript
    var x=document.getElementById("main");
    var y=x.getElementsByTagName("p");
    //先找到id=main的元素，再找到该元素中所有的p元素
    ```
- `getElementsByClassName("intro")`：通过类名查找，返回一个节点集合。



### 事件+驱动程序

| 常见事件     | 描述                         |
| ----------- | ---------------------------- |
| onchange    | HTML 元素改变                |
| onclick     | 用户点击 HTML 元素           |
| onmouseover | 用户在一个HTML元素上移动鼠标 |
| onmouseout  | 用户从一个HTML元素上移开鼠标 |
| onkeydown   | 用户按下键盘按键             |
| onload      | 浏览器已完成页面的加载       |

- 绑定事件的方式：函数即为事件驱动程序
    - 方式一：绑定匿名函数
    ```javascript
    <div id="box1" ></div>

    <script type="text/javascript">
        var div1 = document.getElementById("box1");
        div1.onclick = function () {
            alert("我是弹出的内容");
        }
    </script>
    ```
    - 方式二：先单独定义函数，再绑定
    ```javascript
     <div id="box1" ></div>

    <script type="text/javascript">
        var div1 = document.getElementById("box1");
        div1.onclick = fn;   //注意，这里是fn，不是fn()。fn()指的是返回值。
        //单独定义函数
        function fn() {
            alert("我是弹出的内容");
        }
    </script>
    ```
    - 方式三：行内绑定
    ```javascript
    <div id="box1" onclick="fn()"></div>

    <script type="text/javascript">
        function fn() {
            alert("我是弹出的内容");
        }
    </script>
    ```



### onload事件

这里单独说一下onload事件。当页面加载（文本和图片）完毕的时候，触发onload事件。

js的加载是和html同步加载的。因此，如果使用元素在定义元素之前，容易报错。这个时候，onload事件就能派上用场了，我们可以把使用元素的代码放在onload里，就能保证这段代码是最后执行。

建议是：整个页面上所有元素加载完毕在执行js内容。所以，window.onload可以预防使用标签在定义标签之前。



## 3.2 DOM

在HTML中，一切都是节点！！！

- 节点分类：
    - 元素节点：HTML标签
    - 属性节点：标签的属性
    - 文本节点:标签中的文字（比如标签之间的空格、换行）

- DOM可以做什么
    - 找对象（元素节点）
    - 设置元素的属性值
    - 设置元素的样式
    - 动态创建和删除元素
    - 事件的触发响应：事件源、事件、事件的驱动程序



### DOM节点的获取

DOM节点的获取方式其实就是获取事件源的方式。
- `var x=document.getElementById("intro")`：通过id查找。不存在则返回null。该方法仅用于document对象
- `getElementsByTagName("p")`：通过标签名查找，返回一个节点集合。
    ```javascript
    var x=document.getElementById("main");
    var y=x.getElementsByTagName("p");
    //先找到id=main的元素，再找到该元素中所有的p元素
    ```
- `getElementsByClassName("intro")`：通过类名查找，返回一个节点集合。

后两种方式获得的是标签数组，通常先遍历后使用。
```javascript
    document.getElementsByTagName("div1")[0];    //取数组中的第一个元素
	document.getElementsByClassName("hehe")[0];  //取数组中的第一个元素
```



### DOM访问关系获取

DOM的节点并不是孤立的，因此可以通过DOM节点之间的相对关系对它们进行访问。

节点的访问关系都是属性。

- 获取父节点：`节点.parentNode`。一个节点只有一个父节点。
- 获取兄弟节点：
    - `节点.nextSibling`
    - `节点.nextElementSibling`
    - `节点.previousSibling`
    - `节点.previousElementSibling`
- 获取单个子节点：
    - `节点.firstChild`
    - `节点.firstElementChild`
    - `节点.lastChild`
    - `节点.lastElementChild`
- 获取全部子节点：
    - `节点.childNodes`,标准属性。返回的是指定元素的子节点的集合（包括元素节点、所有属性、文本节点）
    - `节点.children`,非标准属性。返回的是指定元素的子元素节点的集合。【重要】



### DOM节点的操作

节点的操作都是函数。

- 创建节点：`新的标签(元素节点) = document.createElement("标签名")`
- 插入节点：
    - `父节点.appendChild(新的子节点)`，在父节点内容的最后插入一个新的子节点
    - `父节点.insertBefore(新的子节点,参考节点)`，在参考节点前插入一个新的节点。如果参考节点为null，那么他将在父节点里面的最后插入一个子节点
- 删除节点：`父节点.removeChild(子节点)`，用父节点删除子节点。必须要指定是删除哪个子节点。
```javascript
    node1.parentNode.removeChild(node1);//删除节点自身
```
- 复制节点：
    - `要复制的节点.cloneNode()`，不带参数/带参数false：只复制节点本身，不复制子节点
    - `要复制的节点.cloneNode(true)`，带参数true：既复制节点本身，也复制其所有的子节点



### 设置节点的属性

- 获取节点的属性值
    - 方式一：`元素节点.属性`或`元素节点[属性]`
    - 方式二：`元素节点.getAttribute("属性名称")`。推荐！
- 设置节点的属性值
    - 方式一：如`myNode.src = "images/2.jpg"`修改src的属性值
    - 方式二：`元素节点.setAttribute(属性名, 新的属性值)`
- 删除节点的属性：`元素节点.removeAttribute(属性名)`



### DOM对象的属性

DOM对象的属性和HTML的标签属性几乎是一致的。例如：src、title、className、href等。

- nodeType：返回一个代表给定节点类型的整数
    - 1为元素节点
    - 2为属性节点
    - 3为文本节点
- nodeName：给定节点的名字。
    - 元素节点或属性节点的nodeName属性将返回这个元素的名字；
    - 文本节点的nodeName 属性将返回内容为 #text 的字符串。
- nodeValue：给定节点的当前值：
    - 属性节点，返回值是这个属性的值
    - 文本节点，返回值是这个文本节点的内容
    - 元素节点，返回值是null



### innerHTML和innerText

- innerHTML和innerText的区别！！
    - value：标签的value属性。
    - innerHTML：修改双闭合标签里面的内容（识别标签）。
    - innerText：修改双闭合标签里面的内容（不识别标签）。（老版本的火狐用textContent）
    ```javascript
    // p1标签中插入一级标题：新文本!
    document.getElementById("p1").innerHTML="<h1>新文本!</h1>";

    // p1标签中插入内容：<h1>新文本!</h1>。浏览器显示时<h1>也会显示
    document.getElementById("p1").innerText="<h1>新文本!</h1>";
    ```

## 3.3 EventListener

除了在3.1中事件中添加事件的方式，还有一种添加事件的方式：使用`addEventListener()`。

- 语法：`element.addEventListener(event, function, useCapture);`
    - event:事件的类型,注意，这里不要使用on前缀，如使用"click"而不是"onclick"
    - function：事件触发后调用的函数
    - useCapture:布尔值用于描述事件是冒泡还是捕获。该参数是可选的。默认值为 false, 即冒泡传递，当值为 true 时, 事件使用捕获传递。

```javascript
    var btn = document.getElementsByTagName("button")[0];

    //addEventListener: 事件监听器。 原事件被执行的时候，后面绑定的事件照样被执行
    //这种事件绑定的方法不会出现层叠。（更适合团队开发）
    btn.addEventListener("click", fn1);
    btn.addEventListener("click", fn2);

    function fn1() {
        console.log("事件1");
    }

    function fn2() {
        console.log("事件2");
    }
```

`removeEventListener()`方法来移除事件的监听。



### DOM事件流

事件传播的三个阶段是：事件捕获、事件冒泡和目标。

- 事件捕获：事件从最上一级标签开始往下查找，直到捕获到事件目标 target。（从祖先元素往子元素查找，DOM树结构）。在这个过程中，事件相应的监听函数是不会被触发的。
    - 捕获阶段，事件依次传递的顺序是：window --> document --> html--> body --> 父元素、子元素、目标元素。
- 事件冒泡：事件从事件目标 target 开始，往上冒泡直到页面的最上一级标签。（从子元素到祖先元素冒泡）
    - 子元素的事件被触发时，父盒子的同样的事件也会被触发。取消冒泡就是取消这种机制
    - 以下事件不冒泡：blur、focus、load、unload、onmouseenter、onmouseleave。即事件不会往父元素那里传递。
- 事件目标：当到达目标元素之后，执行目标元素该事件相应的处理函数。如果没有绑定监听函数，那就不执行。

- 在JS中：
    - 如果想获取 html节点，方法是`document.documentElement`
    - 如果想获取 body 节点，方法是`document.body`



# 4.BOM

BOM(Browser Object Model)：浏览器对象模型，操作浏览器部分功能的API。比如让浏览器自动滚动。

window对象是BOM的顶层(核心)对象，所有对象都是通过它延伸出来的，也可以称为window的子对象。

- window对象
    - window对象是JavaScript中的顶级对象；
    - 全局变量、自定义函数也是window对象的属性和方法；
    - window对象下的属性和方法调用时，可以省略window



## 4.1 弹窗

JS中可以创建三种消息框：警告框、确认框、提示框。

- 警告框：常用于确保用户可以得到某些信息。
  - 当警告框出现后，用户需要点击确定按钮才能继续进行操作。

    ```javascript
    window.alert("内容");  //可以直接使用alert方法而不带window对象
    ```

- 确认框：常用于验证是否接受用户操作。
  - 当确认卡弹出时，用户可以点击 "确认" 或者 "取消" 来确定用户操作。
  - 当你点击 "确认", 确认框返回 true， 如果点击 "取消", 确认框返回 false。

    ```javascript
    window.confirm("sometext");//可以直接使用confirm方法而不带window对象
    ```

- 提示框：常用于提示用户在进入页面前输入某个值。
  - 当提示框出现后，用户需要输入某个值，然后点击确认或取消按钮才能继续操纵。
  - 如果用户点击确认，那么返回值为输入的值。如果用户点击取消，那么返回值为 null。

    ```javascript
    window.prompt("sometext","defaultvalue");//可以直接使用prompt方法而不带window对象
    ```



## 4.2开/关窗口

- 打开窗口：`window.open(url,target,param)`
    - url：要打开的地址
    - target：新窗口的位置。可以是：_blank 、_self、 _parent 父框架
    - param：新窗口的一些设置
    - 返回值：新窗口的句柄
- 关闭窗口：`window.close()`

# 5.JSON

JSON(JavaScript Object Notation)是用于存储和传输数据的格式，通常用于服务端向网页传递数据 。

JSON 格式在语法上与创建 JavaScript 对象代码是相同的。如:

```json
{"sites":[
    {"name":"Openketang", "url":"openketang.com"},
    {"name":"Google", "url":"www.google.com"},
    {"name":"Taobao", "url":"www.taobao.com"}
]}
//sites是一个数组，包含了三个对象每个对象为站点信息(网站名、网址)
```

- JSON的语法规则：
  - 数据为 键/值 对；
  - 数据由逗号分隔；
  - 大括号保存对象；
  - 方括号保存数组。

通常我们从服务器中读取 JSON 数据，并在网页中显示数据。

JSON不能存储`Date`对象，如果你需要存储`Date`对象，需要将其转换为字符串，之后再将字符串转换为 Date 对象。



## 5.1JSON与JS对象转化

JSON字符串转为JS对象：

1. 创建 JavaScript 字符串，保存JSON 格式的数据；

   ```json
   var text = '{ "sites" : [' +
   '{ "name":"Openketang" , "url":"openketang.com" },' +
   '{ "name":"Google" , "url":"www.google.com" },' +
   '{ "name":"Taobao" , "url":"www.taobao.com" } ]}';
   ```

2. 使用 JavaScript 内置函数`JSON.parse()`将字符串转换为 JavaScript 对象:

   ```javascript
   var obj = JSON.parse(text);
   ```



使用`JSON.stringify()`可将JavaScript 值转换为 JSON 字符串。

```javascript
JSON.stringify(value[, replacer[, space]])
//value:必需， 一个有效的 JSON 字符串。
//replacer:可选。用于转换结果的函数或数组。
//space:可选，文本添加缩进、空格和换行符，如果 space 是一个数字，则返回值文本在每个级别缩进指定数目的空格，如果 space 大于 10，则文本缩进 10 个空格。space 有可以使用非数字，如：\t。
```

JSON 不允许包含函数，JSON.stringify() 会删除 JavaScript 对象的函数，包括 key 和 value。



## 5.2对象

JSON对象在`{}`之间。

使用`.`或`[]`访问对象值：

```json
var myObj, x;
myObj = { "name":"edualiyun", "alexa":10000, "site":null };
x = myObj.name;
x = myObj["name"];
```

使用`delete`关键字删除对象属性：

```json
delete myObj.sites.site1;
delete myObj.sites["site1"]
```

# 6.注意

## 6.1 浮点型使用

JavaScript 中的所有数据都是以 64 位**浮点型数据(float)** 来存储。

所有的编程语言，包括 JavaScript，对浮点型数据的精确度都很难确定：

```javascript
var x = 0.1;
var y = 0.2;
var z = x + y            // z 的结果为 0.3
if (z == 0.3)            // 返回 false
```



## 6.2 提升现象

**函数声明**和**变量声明**总是会被解释器悄悄地被"提升"到方法体的最顶部。

下面的两个实例会得到相同结果：

```javascript
x = 5; // 变量 x 设置为 5
elem = document.getElementById("demo"); // 查找元素
elem.innerHTML = x;                     // 在元素中显示 x
var x; // 声明 x
```

```javascript
var x; // 声明 x
x = 5; // 变量 x 设置为 5
elem = document.getElementById("demo"); // 查找元素
elem.innerHTML = x;
```

JS中只有声明的变量会提升，初始化则不会：

```javascript
var x = 5; // 初始化 x
var y = 7; // 初始化 y
elem = document.getElementById("demo"); // 查找元素
elem.innerHTML = x + " " + y;           // 显示 x 和 y
```

```javascript
var x = 5; // 初始化 x
elem = document.getElementById("demo"); // 查找元素
elem.innerHTML = x + " " + y;           // 显示 x 和 y ,这里y=undefined
var y = 7; // 初始化 y
```



## 6.3 严格模式

在JS代码开头输入`"use strict"`，指定代码在严格条件下执行。值运行在脚本或函数开头。

- 严格模式的限制：
  - 不能使用未声明的变量(对象也是一个变量)

    ```javascript
    "use strict";
    x = 3.14;                // 报错 (x 未定义)
    ```

  - 不允许删除变量或对象

    ```javascript
    "use strict";
    var x = 3.14;
    delete x;                // 报错
    ```

  - 不允许删除变量或对象。

    ```javascript
    "use strict";
    var x = 3.14;
    delete x;                // 报错
    ```

  - 不允许删除函数。

    ```javascript
    "use strict";
    function x(p1, p2) {};
    delete x;                // 错报
    ```

  - 不允许变量重名:

    ```javascript
    "use strict";
    function x(p1, p1) {};   // 报错
    ```

  - 不允许使用八进制:

    ```javascript
    "use strict";
    var x = 010;             // 报错
    ```

  - 不允许使用转义字符:

    ```javascript
    "use strict";
    var x = \010;            // 报错
    ```

  - 不允许对只读属性赋值:

    ```javascript
    "use strict";
    var obj = {};
    Object.defineProperty(obj, "x", {value:0, writable:false});

    obj.x = 3.14;            // 报错
    ```

  - 不允许对一个使用getter方法读取的属性进行赋值

    ```javascript
    "use strict";
    var obj = {get x() {return 0} };

    obj.x = 3.14;            // 报错
    ```

  - 不允许删除一个不允许删除的属性：

    ```javascript
    "use strict";
    delete Object.prototype; // 报错
    ```

  - 变量名不能使用 "eval" 字符串:

    ```javascript
    "use strict";
    var eval = 3.14;         // 报错
    ```

  - 变量名不能使用 "arguments" 字符串:

    ```javascript
    "use strict";
    var arguments = 3.14;    // 报错
    ```

  - 不允许使用以下这种语句:

    ```javascript
    "use strict";
    with (Math){x = cos(2)}; // 报错
    ```

  - 由于一些安全原因，在作用域 eval() 创建的变量不能被调用：

    ```javascript
    "use strict";
    eval ("var x = 2");
    alert (x);               // 报错
    ```

  - 禁止this关键字指向全局对象。

    ```javascript
    function f(){
    	return !this;
    }
    // 返回false，因为"this"指向全局对象，"!this"就是false

    function f(){
    	"use strict";
    	return !this;
    }
     // 返回true，因为严格模式下，this的值为undefined，所以"!this"为true。
    ```

  - 因此，使用构造函数时，如果忘了加new，this不再指向全局对象，而是报错。

    ```javascript
    function f(){
    	"use strict";
    	this.a = 1;
    };
    f();// 报错，this未定义
    ```
