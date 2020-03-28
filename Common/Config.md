# yaml

## YAML 语言

YAML（YAML Ain't Markup Language）以数据为中心，比 json、xml 等更适合做配置文件

YAML 配置例子：

```yaml
server: port:8081
```

XML 配置例子：

```xml
<server>
    <port>8081</port>
</server>
```

参考[语法规范](http://www.yaml.org/)

## YAML 语法

- 使用缩进表示层级关系 ，缩进时不允许使用 Tab 键，**只允许使用空格**

- 所进的空格数目不重要，只要相同层级的元素左侧对齐即可

- 大小写敏感

- k: v  表示一对键值对（冒号和 v 之间必须有一个空格） 即：**属性和值之间一定要有空格**！

  ```yaml
  server:
    port: 8081
    path: /hello
  ```

## 数据结构

- 对象、Map：键值对（属性=值）的集合
  - `k: v` 对象还是`k: v`的方式，在下一行来写对象的属性和值的关系，注意缩进

```yaml
friends:
  lastName: zhangsan
  age: 20

#行内写法：
friends: {lastName: zhangsan,age: 20}
```

- 数组（List、Set）：一组按次序排列的值
  - 用`- 值`表示数组中的一个元素

```yaml
pets:
  - cat
  - dog

#行内写法：
pets: [cat,dog]
```

- 字面量：普通的值（数字、字符串、布尔、日期）
  - `k: v`字面量直接来写，**字符串默认不用加上任何引号**；
  - **`""`双引号可以识别转义字符**
    - `name: "zhangsan \n lisi"`  输出为 zhangsan 换行 lisi
  - **`''`单引号将转义字符作为普通字符串处理**
    - `name: 'zhangsan \n lisi'`   输出为 zhangsan \n lisi
  - 字符串可以写成多行，从第二行开始，必须有一个单空格缩进。换行符会被转为空格

```yaml
person:
  # 写成lastName和 提示的last-name都可以
  last-name: NY
  age: 17
  boss: false
  birth: 1984/07/12
  maps: { fS: 主唱, LL: 哈拉修 }
  lists:
    - fS
    - LoveLive!
    - BiBi
    - 个人
  cat:
    name: Robin
    age: 15
```
