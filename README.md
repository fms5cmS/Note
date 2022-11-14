个人学习笔记。

记录大部分语言静态代码检查工具：https://github.com/analysis-tools-dev/static-analysis

Windows 下安装 GCC：在 [链接](http://tdm-gcc.tdragon.net/download) 中下载后安装，然后将安装目录/bin 添加到环境变量中

- 在建立 MySQL 连接时，url 的参数之间使用 & 分开，示例：
  - 可以参照：<https://blog.csdn.net/booloot/article/details/76223004> 

```properties
url=jdbc:mysql://localhost:3306/mybatis?serverTimezone=UTC&useSSL=false
```

顺序 IO 与随机 IO：

磁盘中 block 是有多个相邻扇区组成的，block 是操作系统读取的最小单元，如果信息能以 block 的形式聚集在一起，则只需一次 IO 读取即可，这样就能极大减少磁盘 IO 时间，这就是顺序 IO。而如果信息再一个磁道中分散在各个扇区中，或者分布在不同磁道的扇区上，就会造成随机 IO，影响性能（寻道时间是随机 IO 的主要瓶颈）。

随机 IO 和顺序 IO 大概相差百倍 (随机 IO：10 ms/ page, 顺序 IO 0.1ms / page)。

# 备注

在代码块中前面带 `$` 的为输入的命令，不带的为输出结果！
