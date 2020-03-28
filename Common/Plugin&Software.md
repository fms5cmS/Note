常用插件&软件：

# Windows

## Wox

[开源项目地址](https://github.com/Wox-launcher/Wox)

要想搜索所有的文件，需要安装[everything](https://www.voidtools.com/)，并运行。

一个[中文的说明文档](http://doc.wox.one/zh/basic/)

安装插件：在搜索框中输入`wpm install 插件名`即可，[插件官网](http://www.wox.one/plugin)

删除插件：`wpm uninstall 插件名`

查看已安装插件：`wpm list`

书签搜索插件：[Bookmarks Seacher](http://www.wox.one/#plugin/182/)，每次搜索时以`b`开始

## Snipaste

屏幕截图：[Snipaste 官网](https://zh.snipaste.com/)

## f.lux

护眼软件

## QuickLook

快速查看，在 Windows 的应用商店下载。

# Atom 插件

## sync settings

1. 登录 Github 后，进入 Settings
2. 选择 Developer settings，点击 Personal access tokens
3. 点击 Generate new token，填入 token description（随意），选中 gist，选择 Generate tiken
4. 复制生成的 token，一定要现在复制，刷新后就再也看不到 token 值而只能看到 token 描述了
5. 在 GitHub 中 New gist

- 该网址是被墙了的，可以在 hosts 文件中加入：`192.30.253.118 gist.github.com`解决

6. 将上面复制的 token 值粘贴到新建的 gist 中，然后 Create secret gist（之后在 Atom 中 backup 时，配置信息会上传到这里）
7. 创建 gist 完成后，复制 URL 地址栏最后一个`/`后面的内容，不要`#`后面的内容，该内容就是你的 Gist ID
8. 在 Atom 中 sync settings 的设置中，将 token 值和 Gist ID 粘贴到相应的位置
9. 在命令行输入`Sync Settings:Backup`,再查看 Gist 会发现有配置信息写进来了
10. 其他电脑上的 Atom 想要同步配置的话，填好 toekn 和 Gist ID，然后在命令行输入`sync Settings:Restore`即可

## Atom Beautity

一个格式化代码的插件，支持 HTML, CSS, JavaScript, PHP, Python, Ruby, Java, C, C++, C#, Objective-C,CoffeeScript, TypeScript, SQL 等多种语言。

使用 Ctrl + Alt + B 快捷键格式化

## platformio-ide-terminal

命令行工具

需要安装 nodejs、python2.7

## minimap

在编辑器中看到文档的小地图。

## autocomplete-\* 系列

autocomplete-\* 系列包含各个语言的代码自动补全功能，需要什么语言的就可以下载该语言相关的插件即可。

- autocomplete-java：java 代码提示补全

# Chrome 插件

- AdBlock：拦截网页广告
- WEB 前端助手
- Clear Cache：快速清空浏览器缓存
- Translate Man 翻译侠（很棒的翻译插件）
- 哔哩哔哩助手
- Toggle table of contents（可以显示页面目录）
- Restlet Client
- Octotree：在 GitHub 左侧显示当前项目的目录结构

# IDEA 插件

- ALibaba Java Coding Guidelines 阿里巴巴的代码检测
- MybatisCodeHelperPro
- Lombok Plugin
- Markdown Navigator
- Maven Helper
- Rainbow Brackets
- Alibaba Cloud Toolkit
- .ignore

## Lombok 使用

javac 从 Java6 开始支持 JSR-269 API 规范，只要程序实现了该 API，就能在 javac 运行时得到调用。Lombok 支持了 JSR-269 API 规范。

给 IDE 安装 Lombok 插件；注意，每个项目都要引入依赖：

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.16.18</version>
    <scope>provided</scope>
</dependency>
```

注意：在类需要序列化、反序列化时详细控制字段。

使用：注解都是标注在类上。常用注解：

- `@Getter`:生成 public 的 Getter 方法

  - `@Getter(AccessLevel.PROTECTED)`:生成 protected 的 Getter 方法,还可以生成其他访问权限的方法

- `@Setter`:生成 public 的 Setter 方法

  - `@Setter(AccessLevel.PROTECTED)`:生成 protected 的 Setter 方法

- `@NoArgsConstructor`:生成无参构造器;

- `@AllArgsConstructor`:生成所有参数的构造器

- `@ToString`:生成打印所有属性的 toString()

  - `@ToString(exclude = {"column1","column2"})`:生成的 toString()排除一个或多个属性，相反的，使用 of 说明该方法中只要哪些属性
  - 直接使用`@ToString`的话，`toString()`只会打印当前类的属性而不会打印父类的属性，如果还要打印父类的属性，可以使用`@ToString(callSuper = true)`

- `@EqualsAndHashCode`:重写 equals()、hashCode()。也可以使用 exclude 排除属性，使用 of 说明该方法中只要哪些属性

- `@Slf4j`:生成 Logback 日志框架的`Logger`的实例：log，可以直接调用 log

- `@Log4j`:当项目使用 Log4j 日志框架时使用

- `@Data`:包含了`@Getter`、`@Setter`、`@ToString`、`@EqualsAndHashCode`，使用所有属性来生成！
