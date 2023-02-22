[Git](https://git-scm.com/book/zh/v2) 安装后，使用 `git --version` 判断是否安装成功。

```shell
git help --web log # 浏览器中查看 log 命令的使用
git merge -h # 查看 merge 命令的帮助文档
gitk # 打开当前仓库的图形化界面，每个点代表了一次 commit，会有不同的分支或标签指向这个点
# 创建并切换到新 branch 上
git checkout -b new_branch
# 基于旧的 branch(也可以是某一次 commit)创建一个新的 branch，并切换到新 branch
git checkout -b new_branch old_branch
# 比较两个 commit 间的差异有以下两种方式：
# 1.
git diff commit_1 commit_2
# 2.
# HEAD 指代的是最近一次的 commit， HEAD^1 表示当前 commit 上一次 commit，HEAD^1^1 表示上上一次 commit
# 也可以省略 1，写 HEAD^^，同样的 ^ 等同于 ~，但 HEAD^^ 则等同于 HEAD~2
git diff HEAD HEAD^1
```

# 配置

使用前的最小配置：

```shell
# 配置 user 信息
git config --global user.name ’用户名‘
git config --global user.email ‘邮箱’

# 在 Git 中把非 ASCII 字符叫做 Unusual 字符。这类字符在 Git 输出到终端的时候默认是用 8 进制转义字符输出的（以防乱码），但现在的终端多数都支持直接显示非 ASCII 字符，所以需要关闭掉这个特性！
git config --global core.quotepath off
```

上面使用 `git config` 配置时使用了 `--global` 参数，config 共有三个作用域：

```shell
git config --local # 只对某个仓库有效，缺省等同于 local
git config --global # 对当前用户的所有仓库有效
git config --system # 对系统所有登录的用户有效

# 本地连有 VPN 时，7890为本地混合配置的端口号，配置以下代理
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
```

如何显示 config 的配置？加上 `--list`，去如：`git config --list --system`

# 建仓库

- 场景一：将已有的项目代码纳入 Git 管理

```shell
cd 项目代码所在文件夹
git init
```

- 场景二：新建项目直接用 Git 管理

```shell
cd 某个文件夹
git init project_name # 会在当前路径下创建与 project_name 同名的文件夹
```

初始化仓库后，会默认生成一个主分支 master。

```shell
# add 命令后面可以跟工作文件、文件夹
git add readme # 将文件 readme 添加到暂存区(见后)
git status   # 查看仓库文件状态
git commit -m'Add readme'
# 可以查看 commit 日志
git log
```

# 工作区&暂存区

Git 中有三个区域：

1. 工作区（添加、编辑、修改文件等操作）
2. 暂存区（暂存已经修改的文件，最后统一提交到 Git 仓库）
3. Git 仓库（最终确定的文件保存到仓库，成为一个新的版本，且对他人可见）

如果要变更工作区的内容，一般是 checkout；
如果要变更暂存区的内容，一般是 reset。

案例：

```shell
# 向暂存区添加 index.html 文件及 images 文件夹
git add index.html images
git commit -m'Add index + logo'

git add styles
git commit -m'Add style.css'

git add js
git commit -m'Add js'

# TODO index.html、style.css 进行修改
# -u 将文件的修改、删除，添加到暂存区。
git add -u
git commit -m'Add refering projects'
```

# 文件重命名

- 方式一：mv、add、rm

```shell
mv readme readme.md
git status  # 提示为删除了 readme，新增了 readme.md
git add readme.md
git rm readme
git status  # 提示中可以看出 git 知道你进行的重命名文件的操作
```

- 方式二：mv

```shell
# 为了还原为之前的场景，该命令会清理掉暂存区、工作目录下的所有变更！
git reset --hard
# 上面的操作并不会破坏 git 的提交记录，可通过 git log 查看提交记录
git mv readme readme.md  # 这条命令可以替代方式一中的三个步骤
git status
```

# 删除

- 方式一：

1. 工作区的文件可以直接删除
2. 再通过 `git add/rm file_name` 来将操作更新到暂存区

- 方式二：

`git rm file_name` 即可

# 版本历史

```shell
git log # 不加任何参数时，显示当前分支下所有的 commit 历史
git log --oneline  # 简洁查看历使，每次 commit 仅显示一行信息
git log -n4 # 指定查看最近的四次 commit 历史，显示的信息很全面，n 可以省略
git log -2 --oneline # 以上两种结合显示
```

通过案例演示更多的使用方法：

```shell
git branch -v # 查看本地所有分支
git checkout -b temp # 新建并切换到 temp 分支
vim readme.md
git add readme.md
git commit -m'Add test'
git log --all # 显示所有分支的 commit 历史
git log --all --graph # 图形化显示所有分支 commit 历史，可以看到各个分支的演进过程
git log temp # 查看 temp 分支的 commit 历史
```

以上使用的参数都可以组合使用

# .git

- `cat HEAD` 会提示：`ref: refs/heads/master` 代表当前工作在 master 分支上，切换分支时，HEAD 文件的内容也会对应改变
- config 文件保存与本地仓库相关的配置信息，[core] 节点进行显示。
  - 如果当前仓库使用 `git config --local` 配置过的话，会出现 [user] 节点
  - 注：global 配置在当前用户路径下的 .gitconfig 文件中，而不会记录在某个仓库的 config 文件中
- refs 文件夹
  - heads 存放当前仓库的所有分支
  - tags 存放当前仓库的所有标签
- objects 文件夹存放所有的 Git 对象，对象哈希值前 2 位作为文件夹名称，后 38 位作为对象文件名
  - 每个对象由 40 位字符组成，前两位作为文件夹名，后 38 位作为文件内容

# 三个对象

Git 的核心对象主要是 commit、tree、blob 三种。

- 每次执行 `git commit` 都会创建出一个 commit 对象；
- 一个 commit 对应一棵 tree(对应于一个文件夹)；
- 一棵 tree 会包含多个 tree、blod
  - 其中 tree 对应文件夹，blob 对应具体的文件；
  - 任何文件的文件内容相同(无论文件名是否一致)，Git 就会将其视为唯一的一个 blob！！！

`find -.git/objects -type f` 可以查看当前仓库下由多少个对象。

```shell
git cat-file -t 08d0e0b302779 # 显示版本库对象的类型，后面的对象名不一定要写全
git cat-file -s 08d0e0b302779 # 显示版本库对象的大小
git cat-file -p 08d0e0b302779 # 显示版本库对象的内容
```

# detached HEAD

分离头指针(detached HEAD)状态本质上是指现在正工作在一个没有分支的状态下。此时可以继续开发，但是如果后续切换分支进行开发的话，Git 可能会将分离头指针状态下做的 commit 当作垃圾清理掉。所以，如果要做变更，一定要和某一分支挂钩，在这一分支的基础上对分支进行变更。

如果临时想基于某个 commit 做变更，试试新方案是否可行，就可以采用分离头指针的方式。测试后发现新方案不成熟，直接 reset 回其他分支即可。省却了建、删分支的麻烦了。

`git checkout commit的哈希值` 会出现分离头指针的情况，且是基于该 commit 的。

在分离头指针状态修改完成后，add、commit，然后使用 log 查看时，会发现此时第一条 commit 并不处于任何分支，而是显示为 `HEAD`，这就是分离头指针。

如果此时 checkout 其他分支，此时会有警告信息，并提示为之前分离头指针状态做的 commit 单独创建一个 branch，这样之后 Git 就不会将其清除掉。

注意：正常状态下，log 会显示类似 `HEAD -> master` 这样的信息，表示现在处于 master 分支。

# .gitignore

创建`.gitignore`文件，配置忽略的文件，不会推送到 git 仓库，也不会检查本地的变化。如：

```shell
### Go template
# Binaries for programs and plugins
*.exe
*.exe~
*.dll
*.so
*.dylib

# Test binary, built with `go test -c`
*.test

# Output of the go coverage tool, specifically when used with LiteIDE
*.out
```

注意：`abc` 和 `abc/` 是不同的，`abc/` 代表 abc 文件夹下的文件都被忽略，但 abc 文件则不会；而 `abc` 则代表 abc 文件 和 abc 文件夹均被忽略！

# 零散命令

```shell
git branch -av # 查看所有分支包括远程分支
git branch Xxx # 新建一个名为 Xxx 的分支（但此时仍在默认的 master 分支上）
git checkout Xxx # 切换到 Xxx 分支上。这时，就可以在这个分支上进行改动了。
git checkout -b Xxx # 新建一个 Xxx 分支，并切换到该分支。
git branch -r # 查看远程分支

git tag v1.0 # 在当前代码状态下新建一个 v1.0 的标签
git tag # 查看历史 tag 记录
git checkout v1.0 #切换到 v1.0 tag 的代码状态
```
