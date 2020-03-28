# bash

查看 /etc/shells 文件，可以知道 Linux 可用的 shells，Linux 预设使用 /bin/bash。

如果读取到一个 Enter 符号 (CR) ，就尝试开始执行该行 (或该串) 命令；

如果一行的内容太多，则可以使用 \【Enter】来延伸至下一行；

bash的优点：
- history功能：记忆使用过的指令；
  - 语法：`history` 查看已经执行过的历史指令，会显示出指令的编号
  - `history`显示所有历史指令
  - `history 10` 显示最近的10个指令
  - `history !178` 执行编号为178的指令
- 命令和文件补全功能：
  - 【Tab】接在一串指令的第一个字后面，则为命令补全；
  - 【Tab】接在一串指令的第二个字以后，则为文件补全；
- 命令别名设定功能：给常用指令设定别名
  - eg：`alias lm='ls -al'`，此时就可以使用`lm`代替`ls -al`指令；
  - 直接输入`alias`查看当前的命令别名；
  - eg：`unalias lm`取消别名
- 通配符（Wildcard）
  - `*`：任意多个任意字符
  - `?`：一定有一个任意字符。eg：`ll -d /etc/?????`找出 /etc/ 下文件名刚好是五个字母的文件名
  - `[]`：一定有一个在括号内的字符(非任意字符)。eg：`[abcd]`一定有a、b、c、d中的一个字符
  - `[-]`：若有减号在中括号内时，代表在编码顺序内的所有字符。eg：`[0-9]`代表 0 到 9 之间的所有数字
  - `[^]`：若中括号内的第一个字符为`^`，那表示反向选择，例如`[^abc]`代表 一定有一个字符，只要是非 a, b, c 的其他字符就接受的意思。

在命令行中：`bash`进入子程序；`exit`离开当前子程序



## 变量

`set`指令可以查看当前shell中的所有变量。

- 使用`echo`指令来取用变量，变量在被取用时，前面必须有`$`才行。

  ```bash
  echo $PATH  #或
  echo ${PATH}  #建议使用这种方式
  ```

- 变量的设定

  ```bash
  echo $myname  #这里不会输出任何信息，因为该变量未被设定
  myname=+1day
  echo $myname  #输出+1day
  ```



### 系统变量

如何查看环境变量？ 使用`env`或`export`指令即可。常见的环境变量：

| 环境变量名 | 描述                                                        |
| ---------- | ----------------------------------------------------------- |
| HOME       | 用户的家目录。`cd`指令就是取用了该变量                      |
| SHELL      | 当前环境使用的SHELL是哪个程序，Linux预设为 /bin/bash        |
| HISTSIZE   | 可被记录的历史指令的数量                                    |
| MAIL       | 使用`mail`指令在收信时，系统会读取的邮件信箱文件（mailbox） |
| PATH       | 执行文件搜寻的路径。目录与目录间用`:`分隔                   |
| LANG       | 语言数据                                                    |
| RANDOM     | 随机数的变量，bash环境下，该变量介于0~32767之间             |

使用`locale`指令可查询当前Linux支持的语系。部分重要的其他系统变量：

- `PS1`：命令提示字符
  - 当前环境下值：`PS1='[\u@\h \W]\$ '`
  - `\u`：当前账户名称
  - `\h`：仅取主机名在第一个小数点前的名字
  - `\W`：利用 basename 函数取得工作目录名称，所以仅会列出最后一个目录名。
  - `\$` ：提示字符，如果是 root 时，提示字符为 # ，否则就是 $
- `$`：本shell的PID
  - 代表当前Shell的线程代号（Process ID）
  - 使用`echo $$`查询PID号码
- `?`：上一个执行指令的返回值
  - 执行某些指令时， 这些指令都会回传一个执行后的代码。一般来说，如果成功的执行该指令，则会回传一个 0 值，如果执行过程发生错误，就会回传『错误代码』才对！一般就是以非为 0 的数值来取代。

- `export 变量名=变量值` 将 Shell 变量输出为环境变量
- `source 配置文件` 让修改后的配置信息立即生效
- 为了让 /etc/profile 的环境变量生效，需要使用`source /etc/profile`或重启



### 自定义变量

- 字符串类型的变量

变量名只能是字母、数字、下划线组成，不能以数字开头

`=`两边不能有空格符

```bash
mynae = +1day  或者 myname=+ 1day   都是错误的
```

变量内容若有空格符，可使用`""`或`''`，但

1. 双引号内的特殊字符（如`$`）可以保有原本特性；

```bash
var="lang is $LANG"  #echo $var后输出lang is zh_TW.UTF-8
```

2. 单引号内的特殊字符仅为纯文本。

声明静态变量（无法撤销）：`readonly 变量名 `

可以使用转义字符`\`将特殊字符变成一般字符

若要在变量原有基础上增加内容：

```bash
PATH="$PATH":/home/bin  #或
PATH=${PATH}:/home/bin
```

若该变量需要在其他子程序执行，需要以`export`来使变量成为环境变量

```bash
export 变量名
```

撤销变量：`unset 变量名`

---

- 如果需要非字符串类型的变量，就需要声明变量。

- `declare [-aixr] 变量名`：声明变量的类型
  - `-a` ：将后面的变量定义成为数组 (array) 类型
  - `-i` ：将后面的变量定义成为整型 (integer) 
  - `-x` ：用法与 export 一样，就是将后面的变量变成环境变量；将`-`变为`+`可以进行取消动作。
  - `-r` ：将变量设定成为`readonly`类型，该变量不可被更改内容，也不能`unset`

```shell
sum=100+30
echo ${sum}     #输出100+30，而没有进行计算
declare -i sum=100+30
echo ${sum}   #输出130
```

注意：若不指定变量类型，变量类型默认为字符串！



### 预定义变量

预定义变量就是 Shell 设计者事先定义好的变量，可以直接在Shell 脚本中使用
- `$!` 后台运行的最后一个进程的进程ID（PID）
- `$?` 最后一次执行的命令的返回状态。
  - 如果这个变量的值为 0，证明上一个命令执行正确；
  - 如果为非0（具体是哪个数字，由命令自己决定），则上一个命令执行不正确



### 位置参数变量

Shell 脚本文件在执行时可以向脚本内的程序传参吗？答案是可以的。针对参数，有一些默认的变量名：（在程序内，可以使用这些变量获取到传入的参数）

执行 Shell 脚本时，如果希望获取命令行的参数信息，可以使用位置参数变量。

- `$n` n为数字
  - `$0`是脚本程序文件名
  - `$1`-`$9`是第一到第九个参数
  - 10以上的参数要用`{}`包含，如`${10}`
- `$*` 代表命令行中所有参数。该命令将所有参数看作一个整体
- `$@` 也代表命令行中所有参数。不过把每个参数区分对待
- `$#` 代表命令行中所有参数的个数

文件：test.sh。执行时：`sh test.sh 3 "ms5cm"`

```shell
#!/bin/bash
echo "文件名：$0"  #test.sh
echo "第二个参数：$1"  #ms5cm
echo "参数个数：$#"   #2
echo "全部参数内容：$@"  #'3 ms5cm'
```



### 外部读取变量

`read [-pt] 变量名`：读取来自键盘输入的变量。

- `-p`：后面可以接提示符
- `-t`：后面可以接等待用户输入的时间，单位为秒

```shell
read -p "请输入你的名字:" -t 30 username
```



### 变量内容相关操作

- 删除部分变量内容：`${变量名#关键词}`
  - `#`：表示变量内容**从左往右**开始删除：
    - `#`：表示删除符合“关键词”最短的部分
    - `##`：表示删除“关键词”最长的部分
  - `%`：表示变量内容**从右往左**开始删除
    - `%`：表示删除符合“关键词”最短的部分
    - `$$`：表示删除“关键词”最长的部分
  - 通配符`*`代表任意长的任意字符串

```shell
path=${PATH}	#为了防止修改错误，使用path来操作
/opt/jdk10.0.1/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/fms5cms/.local/bin:/home/fms5cms/bin
echo ${path#/*:} #删除了 /opt/jdk10.0.1/bin:
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/fms5cms/.local/bin:/home/fms5cms/bin
path=${PATH}
echo ${path##/*:} #删除了 /opt/jdk10.0.1/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/fms5cms/.local/bin:
/home/fms5cms/bin
```

```shell
echo ${path%:*bin}
```

---

- 取代部分变量内容
  - `${变量名/旧字符串/新字符串}`：从左往右第一个符合的旧字符串被替换为新字符串
  - `${变量名//旧字符串/新字符串}`：所有的符合的旧字符串被替换为新字符串

---

- 变量测试&内容替换

  - `新变量名=${旧变量名-内容}`

    - 若旧变量没有设定过，那么使用新变量取代旧变量，且新变量值为`-`后面的内容；
    - 若旧变量已被设定过（包括值为空字符串），新变量内容以旧变量来替换，旧变量内容不变。
    - 通常新、旧变量名是一样的！

  - `新变量名=${旧变量名:-内容}`

    - 若旧变量没有设定过或值为空字符串，那么使用新变量取代旧变量，且新变量值为`-`后面的内容；
    - 若旧变量已被设定过，新变量内容以旧变量来替换，旧变量内容不变。
    - 通常新、旧变量名是一样的！

  - 上面两个使用中`-`的测试并不会影响到旧变量的内容，如果想将旧变量内容也一起替换掉，使用`=`。

  - 如果旧变量不存在，整个测试显示“有错误”，此时可使用`?`

    ```shell
    unset str; var=${str?无该变量} #str不存在，测试结果为“无该变量”
    #旧变量存在的话，新变量内容以旧变量来替换，旧变量内容不变
    ```



## 数据流重导向

标准输入（standard input）：代码为0，使用`<`或`<<`；

标准输出（standard output）：代码为1，使用`>`或`>>`；（将正确的数据输出）

标准错误输出（standard error）：代码为2，使用`2>`或`2>>`。（将错误的数据输出）



- `>`输出重定向（将文件的内容覆盖） `>>`追加
  - `ls -l > 文件`：把`ls -l`显示的列表的内容写入文件（文件不存在则创建文件再覆盖内容）
  - `ls -al >> 文件`：把`ls -al`显示的列表的内容追加到文件末尾
  - `cat 文件1 > 文件2`：将文件1的内容覆盖到文件2
  - `echo "内容">>文件`：把内容追加入文件
  - `cat > 文件`：输入该命令后，就可以使用键盘来向文件中写入内容了。使用【ctrl+d】中断键盘输入。
    - 由于`>`在`cat`后，所以后面的文件会被主动创建

```shell
将正确与错误数据全部写入同一个文件：
find /home -name .bashrc > list 2> list  #错误
#两股数据同时写入一个文件，又没有使用特殊的语法， 此时两股数据可能会交叉写入该文件内，造成次序的错乱
find /home -name .bashrc > list 2>&1  #正确
find /home -name .bashrc &> list #正确
```

```shell
find /home -name .bashrc 2> /dev/null #将错误的数据丢弃。/dev/null可以吃掉任何导向该终止的信息
```

---

- `<`：将原本需要由键盘输入的数据，改由文件内容来取代

```shell
cat > catfile < test.txt
  #将test.txt的内容写入catfile中，以覆盖的形式
```

- `<<`：结束的输入字符

```shell
# 用 cat 直接将输入的讯息输出到 catfile 中， 且当由键盘输入eof时，该次输入就结束,不用输入ctrl+d
# eof并不会输入到文件中
cat > catfile << "eof"  
```



## 多指令同时执行

当我们希望可以一次执行多个指令时，可以在指令之间用`;`来分隔。

```shell
sync;sync;shutdown -h now
```

那么当两个指令之间存在相关性时，可以使用`&&`或`||`，且具有短路作用。

```shell
cmd1 && cmd2
# cmd1 正确执行（$?=0），则开始执行cnmd2；cmd1 执行错误（$?不等于0），则cmd2不执行
```

```shell
cmd1 || cmd2
# cmd1 正确执行，则cmd2不执行；cmd1执行错误，则开始执行cmd2
```



## 管线命令

`|`：将前一个命令的处理结果输出传递给后面的命令处理

但仅能处理经由前面一个指令传来的正确信息，也就是 standard output 的信息，对于stdandard error 会忽略。



### 筛选命令

- `cut`指令：将同一行里面的数据进行分解！
- `-d`：后接分隔字符，与`-f`一起使用
  - `-f`：依据`-d`的分隔字符将一段讯息分区成为数段，用`-f`取出第几段的意思
- `-c`：以字符为单位取出固定字符区间
  

```shell
 echo "1;2;3;4;5" | cut -d ';' -f 2，4   #输出 2;4
 echo "1;2;3;4;5" | cut -c 2-5      #输出 ;2;3
 echo "1;2;3;4;5" | cut -c 2-		#输出 ;2;3;4;5
```

---

- `grep [选项] 查找内容 源文件`：过滤查找分析**一行**讯息
- `-a`：将 binary 文件以 text 文件的方式查找数据
  
- `-c`：计算找到所需数据的次数
  
- `-n`：显示匹配行及行号
  
- `-i`：忽略字母大小写
  
- `-v`：反向查找
  

```shell
last | grep 'root'   #将 last 中，出现 root 的每一行都取出来。last可得到登入主机者的身份
last | grep -v 'root' #只有没有root就将那一行取出来
```



### 排序指令

- `sort`指令，默认使用第一个数据排序
- `-f`：排序时忽略大小写差异
  - `-b`：忽略最前面的空格部分
- `-M`：以月份排序
  - `-n`：使用纯数字排序（默认以字符来排序，即从a开始向后排）
- `-r`：反向排序
  - `-u`：去重
- `-t`：分隔符，默认用【tab】分隔
  - `-k`：以某个区间排序

```shell
cat /etc/passwd | sort
```

---

- `uniq`指令，将重复的资料仅列出一个显示
- `-i`：忽略大小写
  
- `-c`：计数
  

```shell
# last 将账号列出，仅取出账号栏，进行排序后仅取出一位；
last | cut -d ' ' -f 1 | sort | uniq  
# 在上面的基础上还知道每个人的登入次数
last | cut -d ' ' -f 1 | sort | uniq -c 
```

---

- `wc`指令，统计共有多少行、字、字符
- `-l`：仅列出行
  
- `-w`：仅列出多少字（英文单字）
  
- `-m`：多少字符
  

```shell
cat /etc/man_db.conf | wc #/etc/man_db.conf有多少字、行、字符数
```



### 双向重导向

- `tee [-a] 文件`：将数据流分送到文件与屏幕
- `-a`：以追加方式将数据写入文件
  

```shell
last | tee last.list | cut -d " " -f1
#last查询到的结果，一份以覆盖方式写入last.list，一份导向cut命令
ls -l | tee test | more
#ls的数据存一份到test中，同时屏幕也会输出
```



### 字符转换命令

- `tr`指令：删除一段信息中的文字，或进行文字内容的替换
- `-d`：删除信息中的指定字符串
  - `-s`：取代掉重复的字符
  

```shell
#删除last查询结果中 ms5cm 这个字符串
last | tr -d "ms5cm"  
#将last查询结果中的小写字母用大写字母替换
last | tr "[a-Z]" "[A-Z]" 
```

---

- `col`指令：常用于将【tab】按键取代成为对等的空格键
- `-x`：将【tab】键转换成对等的空格键
  

```shell
#cat -A会显示出所有的特殊按键，此时会看到很多 ^I 也就是【tab】
cat -A /etc/man_db.conf  
#【tab】被取代为空格
cat /etc/man_db.conf | col -x | cat -A | more 
```

---

- `join [选项] file1 file2`：将两个文件中有相同数据的一行合并在一起
- `-t`：指定分隔符。`join`默认以空格符分隔数据，并对比第一个字段的数据，如果两个文件相同，则将两笔数据联成一行，且第一个字段放在第一个
  - `-i`：忽略大小写
  - `-1`：代表第一个文件要用哪个字段分析
  - `-2`：代表第二个文件 要用哪个字段分析
  - 注意：使用`join`前，需要处理的文件应先经过排序处理！！
  

```shell
join -t ':' -1 4 /etc/passwd -2 3 /etc/group | head -n 3
#以':'来分隔数据，第一个文件/etc/passwd使用第四个字段分析，第二个文件/etc/group使用第三个字段分析
```

---

- `paste [选项] file1 file2`：将两个文件的两行贴在一起，且中间以【tab】分隔
- `-d`：指定分隔符，默认以【tab】分隔
  - `-`：如果 file 部分写成`-`，表示该位置的内容来自 standard input 的资料的意思
  

```shell
cat catfile | paste last.list -
#将last.list与catfile的每一行合并，中间用【tab】分隔。
```

- `expand [选项] file`：将【tab】转换为空格键。
- `-t`：定义一个【tab】用多少空格代替。一般为8个



### 分区命令

- `split`指令：将一个文件依据文件大侠或行数来分区。`split [选项] file PREFIX`，选项：

  - `-b`：指定分区成的文件大小，可接单位，如：b、k、m等
  - `-l`：指定每几行分为一个文件
  - `PREFIX`：代表前导符的意思，可作为分区文件的前导文字


```shell
split -b 300k /etc/services abc
#将/etc/services文件分成300k一个文件，且分区后的文件名以abc开始
cat abc* >> servicesback  #将上面的几个小文件合并为一个文件，文件名servicesback
```



### 参数代换

很多指令并不支持管线命令，可以通过`xargs`来提供该指令引用 standard input 之用。

- `xargs`指令：产生某个指令的参数。`xargs [选项] command`，选项：

  - `-0`：如果输入的 standard input 含特殊字符，该参数可将其还原为一般字符
  - `-e`：后接一个字符串，当`xargs`分析到该字符串时，就会停止继续工作
  - `-p`：执行每个指令的参数时，都会询问用户
  - `-n`：后接次数，每次command指令执行时，要使用几个参数
  - 当后面没有接任何指令时，默认以`echo`输出


```shell
id root #id指令可以查询用户的uid、gid等信息，但仅能接受一个参数！
cut -d ':' -f1 /etc/passwd | head -n 3 | xargs -n 1 id
#透过xargs指令，使的id指令每次仅接受一个参数，但是会执行3次（因为前面仅引出了passwd的前三行数据即前三个用户）
cut -d ':' -f1 /etc/passwd | head -n 3 | xargs -p -n 1 id
#每次执行id时，都会询问使用者
```



### 减号的用途

`-`可以用来替代 standard input 和 standard output。使用见字符转换命令的`paste`指令。



# Shell script

shell脚本用在系统管理上是很好的一项工具，但用在处理大量数值运算上，性能就不够了，因为它的速度较慢，且使用的CPU资源较多，造成主机资源的分配不良。

- 注释方式：


```shell
#  单行注释
:<<!
	多行注释
!
```

- 第一行必须是：`#!/bin/bash`。该文件内的语法用bash的语法，执行时，就可以加载bash的相关环境配置文件。
- 返回值：利用`exit`指令来中断程序，并返回一个数值给系统。
- 需要有可执行权限：eg：`chmod 744 hello.sh`

示例：

```shell
#!/bin/bash
echo "Hello World!"
exit 0
#执行完当前脚本后，输入echo $? 可以得到返回值0
```

在bash中`=`和`==`的结果是相同的，通常`=`代表变量的设定，`==`代表逻辑判断



- 执行方式：
  - 直接执行：hello.sh 文件必须有 rx 的权限
    - 绝对路径：使用`/home/zzk/shell/hello.sh`来执行；
    - 相对路径：进入文件坐在目录后，使用`./hello.sh`来执行；
    - 变量`PATH`功能：将 hello.sh 放在`PATH`指定的目录中，如：`~/bin/`，此时直接输入`hello.sh`执行。
  - 以 bash 程序执行：hello.sh 文件有 r 权限即可
    - 使用：`bash hello.sh`或`sh hello.sh`来执行。
  - 执行程序时，是在父程序 bash 中输入以上任一命令，然后系统会给予一支新的bash 来执行 hello.sh 中的指令，因此 hello.sh 中的变量其实是在新的 bash 即子程序 bash 中执行的，当 hello.sh 执行完后，子程序 bash 内的所有数据便被移除，无法在父程序中通过`${}`来获取 hello.sh 中的变量。
  - 使用`source`来执行程序，`source hello.sh`，此时系统并不会新开 bash 来执行程序中的指令，而是直接在父程序中执行，所以此时仍可以在父程序中通过`${}`来获取 hello.sh 中的变量。



## 运算符

- \`\`反引号(esc键下面的键)可以运行里面的命令，并把结果返回给变量


```shell
result=`ls -l`
echo ${result}
```

- `$((运算式))`或`$[运算式]` 推荐使用后者

- `expr m + n` 注意：expr 运算符间要有空格

- `expr m - n`

- `expr \*，/，%` 乘、除、取余


```shell
#计算 （3+2）*4   
#方式一：	
RESULT=$(((3+2)*4))
#方式二：
RESULT=$[(3+2)*4]
echo "result=$RESULT"  #20

#使用 expr
TEMP=`expr 2 + 3`
RESULT=`expr $TEMP \* 4`
echo "result=$RESULT"   #20

#写出命令行的两个参数之和
SUM=$[$1+$2]
echo "SUM=$SUM"  #30
```



##  案例

案例一：创建三个文件，文件名为：自定义基名+三天内的日期

```shell
#!/bin/bash
#author:fms5cmS
#date:2018-10-17
echo -e "I will use 'touch' command to create 3 files. " #纯粹显示信息
read -p "please input your filename's prefix:" fileuser #提示用户输入文件基名

filename=${fileuser:-"filename"} #判断是否有配置文件基名，如果没有基名为filename

date1=$(date --date='2 days ago' +%Y%m%d) #前两天的日期
date2=$(date --date='1 days ago' +%Y%m%d) #昨天的日期
date3=$(date +%Y%m%d) #今天的日期
#生成文件名：自定义前缀+日期
file1=${filename}${date1}
file2=${filename}${date2}
file3=${filename}${date3}
#根据文件名创建文件
touch "${file1}"
touch "${file2}"
touch "${file3}"
```



案例二：程序计算两个数字（必须为整数）的乘积：

- 可以使用`$((计算式))`来进行数值运算，不过仅支持整型。
  - 如果想要保存结果的话，可以使用一个变量接收：`var=$(( 2 * 3))`；
  - 就像上面的例子所示，这种方式在计算式的位置可以有空格；
  - 如果要计算含小数的数据，可借助`bc`这个指令的帮助：`echo "123.123*55.9" | bc`，如果不加bc指令，输出的是字符串。

```shell
#!/bin/bash
#author:fms5cmS
#date:2018-10-17
echo -e "You should input 2 numbers,I will multiplying them! \n"
# echo -e 可以对要输出内容的反斜杠转义的字符进行解释
read -p "first number:" firstnu
read -p "second number:" secnu
total=$((${firstnu}*${secnu}))
echo -e "\nThe result of ${firstnu} x ${secnu} is ==> ${total}"
```



案例三：计算pi

用户输入计算结果pi的小数点后面的位数

```shell
#!/bin/bash
#author:fms5cmS
#date:2018-10-17
echo -e "This program will calculate the value of pi.\n"
echo -e "You should input a float number to calculate pi.\n"
read -p "This scale number (10~10000)?" checking
num=${checking:-"10"} #判断是否有输入值
echo -e "Starting calculate pi. Be patient."
time echo "scale=${num};4*a(1)" | bc -lq
```

上面的`4*a(1)`是 bc 主动提供的一个计算 pi 的函数，使用该函数必须要使用`bc -l`来呼叫。



## 判断式

- `test`指令，该指令不会返回信息，但可以通过其他方式来获取结果：

```shell
test -e /dmtsai && echo "exist" || echo "Not exist"
```

- `[]`判断符号：

  - 使用时中，括号内的每个组件都需要有空格符来分隔！！！
  - 中括号内的变量，最好都以`""`括起来；
  
  - 如：`[ "${HOME}" == "${MAIL}" ]`相当于：`test${HOME}==${MAIL}`

- 以上两种判断式，非空返回 true，0为 true，>1 为false


常用的判断条件

  - 两个整数的比较：

| 符号 |     ==     | -lt  |   -le    | -eq  | -gt  |   -ge    |  -ne   |
| :--: | :--------: | :--: | :------: | :--: | :--: | :------: | :----: |
| 含义 | 字符串比较 | 小于 | 小于等于 | 等于 | 大于 | 大于等于 | 不等于 |

  - 按照文件权限进行判断：

| 符号 |     -r     |     -w     |      -x      |
| :--: | :--------: | :--------: | :----------: |
| 含义 | 有读的权限 | 有写的权限 | 有执行的权限 |

  - 按照文件类型进行判断：

| 符号 |             -f             |    -e    |          -d          |
| :--: | :------------------------: | :------: | :------------------: |
| 含义 | 文件存在，且是一个常规文件 | 文件存在 | 文件存在且是一个目录 |

案例：

```shell
#判断 “ok”是否等于“ok”
if [ "ok" == "ok" ]; then 
	echo "equal"
fi

#判断 23是否大于等于22
if [ 23 -ge 22 ]; then
	echo "yes"
fi

#判断目录中的文件是否存在
if [ -e /home/zzk/ok.txt ]; then 
	echo "存在"
fi
```



## 流程控制

- if 判断，推荐使用第二种方式

```sh
if [ 条件判断 ]; then
条件判断成立时，执行程序...
else  #else及下面的程序 可以没有
条件判断不成立时，执行程序...
fi #代表结束if
```

```sh
if [ 条件判断1 ]; then
程序
elif [ 条件判断2 ]; then
程序
fi
```

案例：如果输入的参数大于等于60 ，输出“及格”，否则输出“不及格”

```sh
#!/bin/bash
read -p "Please input your number:" number
if [ ${number} -ge 60 ]; then
	echo "及格"
else
	echo "不及格"
fi
```

---

- case 语句

```sh
case $变量名 in
"值1")
	程序1（如果变量值等于值1，则执行程序1）
	;;
"值2")
	程序2（如果变量值等于值1，则执行程序2）
	;;
*)	# *代表其他
	程序（如果变量值不等于上面任何一个，则执行此程序）
	;;
esac
```

案例：当命令行参数是 1 时，输出“周一”，2时输出“周二”，其他情况输出“other”

```sh
#!/bin/bash
case $1 in
"1")
	echo "周一"
	;;
"2")
	echo "周二"
	;;
*)
	echo "other"
	;;
esac
```



## 循环

- for 循环

```sh
for 变量 in 值1 值2 值3 ...
do
	程序
done
```

```sh
for ((初始值;循环控制条件;变量变化))
do
	程序
done
```

案例：

```sh
#打印命令行输入的参数
#!/bin/bash
for i in "$*"       #这里也可以用 "$@"
do
	echo "the num is $i"
done
```

  ```sh
#从1加到100的值输出显示
#!/bin/bash
#定义一个变量
SUM=0
for((i=1;i<=100;i++))
do
	SUM=$[$SUM+$i]
done
echo "sum=$SUM"
  ```

---

- while 循环：只要满足条件就循环

```sh
while [ 条件判断 ]
do 
	程序
done
```

案例：从命令行输入一个数字 n，统计从1+...+n的值是多少

```sh
#!/bin/bash
SUM=0
i=1
while [ $i -le $1 ]
do
  SUM=$[$SUM+$i]
  i=$[$i+1]
done
echo "sum=$SUM"
```

---

- util 循环：当满足条件时终止循环


```shell
util [ 条件判断 ]
do
	程序
done
```



##  函数

系统函数：

- `basename [pathname]` 显示文件名（含后缀）

  ```shell
basename /home/zzk/ok.txt                #输出：ok.txt
  ```
  
- `basename [string][suffix]` 仅显示文件名（不含后缀）

  ```shell
  basename /home/zzk/ok.txt .txt           #输出：ok
  ```

- `dirname 文件绝对路径`：返回路径（目录的部分）

  ```shell
	dirname /home/zzk/ok.txt				#输出：/home/zzk
  ```

---


自定义函数

- Shell script 的执行方式是由上而下，由左而右，因此函数的设定一定要在程序最前面！

- function也有内建变量，函数名称为`$0`，后续接的变量也是以`$1`、`$2`...来取代

- 语法：

  ```sh
  function funname(){
      Action
      [return int;]
  }
  ```

- 调用：直接写函数名 `funname 传入的参数`

案例：计算输入两个参数的和

```sh
#!/bin/bash
read -p "请输入计算的数值一：" NUM1
read -p "请输入计算的数值二：" NUM2
function getSum(){
  SUM=$[$NUM1+$NUM2]
  echo "和是：$SUM"
}
getSum $NUM1 $NUM2
```



## 追踪与debug

- `sh [选项] hello.sh`
  - `-n`：不执行脚本，仅查询语法的问题
  - `-v`：在执行脚本前，先将脚本的内容输出在屏幕上
  - `-x`：将用到的脚本内容显示在屏幕上！！！！



## 综合案例

需求分析：

1. 每天凌晨 2:10 备份数据库 zzkDB 到 /data/backup/db
2. 备份开始和备份结束能够给出相应的提示信息
3. 备份后的文件要求以备份时间为文件名，并打包成 .tar.gz 形式，如：2018-03-12_130201.tar.gz
4. 在备份的同时，检查是否有十天前备份的数据库文件，如果有就将其删除

脚本 mysql_db_backup.sh 完成后，设置到 crond 中执行

1. 编写脚本


```shell
#!/bin/bash
#备份的路径
BACKUP=/data/backup/db
#获取当前时间作为文件名
DATETIME=$(date +$Y_%m_%d_%H%M%S)

echo "===================开始备份======================"
echo "===================备份路径是：$BACKUP/$DATETIME.tar.gz"

#主机
HOST=localhost
#用户名
DB_USER=root
#密码
DB_PWD=root

#备份数据库名
DATABASE=zzkDB
#创建备份的路径
#如果备份的路径文件夹存在，就使用，否则就创建
[ ! -d "$BACKUP/$DATETIME" ] && mkdir -p "$BACKUP/$DATETIME"
#执行MySQL的备份数据库的指令(将其压缩到目录下，并起名作为一个临时文件)
mysqldump -u${DB_USER} -p${DB_PWD} --host=$HOST $DATABASE | gzip > $BACKUP/$DATETIME/$DATETIME.sql.gz 
#打包备份文件
cd $BACKUP
tar -zcvf $DATETIME.tar.gz $DATETIME
#删除临时目录
rm -rf $BACKUP/$DATETIME

#删除10天前的备份文件
#在指定目录下按时间找10天前的文件，然后对找到的文件执行删除命令
find $BACKUP -mtime +10 -name "*.tar.gz" -exec rm -rf {} \;
echo "=================备份文件成功==================="
```

2. 给脚本可执行权限
3. 在任务调度中执行：

`crontab -e`

`10 2 * * * /.../mysql_db_backup.sh`













