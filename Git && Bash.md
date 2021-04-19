

# Git && Bash

## Git

### 简介

Git是目前世界上最先进的分布式版本控制系统，跟github配合使用管理文件系统

### 基本使用

初始化仓库

```
git init
```

添加文件到仓库

```
git add filename 
```

添加所有

```
git add .
```

将文件提交到仓库

```
git commit -m "备注"
```

### 版本控制

显示从最近到最远的提交日志

```
git log
```

加上`--pretty=oneline`参数可以简化信息

```
git log --pretty=oneline
```

记录每一次命令

```
git reflog
```

查看文件状态

```
git status
```

查看工作区和版本库里面最新版本的区别

```
git diff HEAD -- filename
```

丢弃工作区的修改

```
git checkout -- file
```

把暂存区的修改撤销掉（unstage），重新放回工作区

```
git reset HEAD file
```

在Git中，用`HEAD`表示当前版本,上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本写100个`^`比较容易数不过来，所以写成`HEAD~100`。

回退版本使用

```
git reset --hard HEAD^
```

或者使用git log中的日志号

```
git reset --hard xxxx
```

### 远程仓库

首先在本地跟github上分别建仓库

关联两个仓库

```
git remote add origin git@github.com:github_username/github_repository.git
```

推送

```
git push -u origin master
```

删除远程库

```
git remote rm origin
```

从远程库克隆

```
git clone githuburl
```

### 分支管理

创建分支

```
git branch dev
```

切换分支

```
git checkout dev/ git switch dev
```

创建并切换分支

```
git checkout -b dev/ git switch -c dev
```

查看当期分支

```
git branch
```

将当前分支合并到master分支

```
git merge dev
```

删除分支

```
git branch -d dev
```

查看分支合并图

```
git log --graph
```

保存工作现场

```
git stash
```

查看stash内容

```
git stash list
```

恢复现场（恢复的同时删除删除stash内容）

```
git stash pop / git stash apply && git stash drop
```

复制特定提交到当前分支

```
git cherry-pick xxxx
```

分支：

- `master`分支是主分支，因此要时刻与远程同步；
- `dev`分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步；
- bug分支只用于在本地修复bug，就没必要推到远程了，除非老板要看看你每周到底修复了几个bug；
- feature分支是否推到远程，取决于你是否和你的小伙伴合作在上面开发。

多人协作的工作模式通常是这样：

1. 首先，可以试图用`git push origin <branch-name>`推送自己的修改；
2. 如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并；
3. 如果合并有冲突，则解决冲突，并在本地提交；
4. 没有冲突或者解决掉冲突后，再用`git push origin <branch-name>`推送就能成功！

### 标签

打上标签

```
git tag name
```

给某个commit打上标签

```
git tag name xxxcommit
```

查看标签

```
git tag
```

查看某个标签

```
git show tagname
```

创建带有说明的标签，用`-a`指定标签名，`-m`指定说明文字

```
git tag -a v0.1 -m "version 0.1 released" 1094adb
```

删除标签

```
git tag -d tagname
```

推送标签到远程

```
git push origin tagname
```

推送所有标签

```
git push origin --tags
```



## Bash

### Bash简介

Bash 是 Unix 系统和 Linux 系统的一种 Shell（命令行环境），是目前绝大多数 Linux 发行版的默认 Shell。

### Shell含义

Shell 这个单词的原意是“外壳”，跟 kernel（内核）相对应，比喻内核外面的一层，即用户跟内核交互的对话界面。

具体来说，Shell 这个词有多种含义。

首先，Shell 是一个程序，提供一个与用户对话的环境。这个环境只有一个命令提示符，让用户从键盘输入命令，所以又称为命令行环境（command line interface，简写为 CLI）。Shell 接收到用户输入的命令，将命令送入操作系统执行，并将结果返回给用户。本书中，除非特别指明，Shell 指的就是命令行环境。

其次，Shell 是一个命令解释器，解释用户输入的命令。它支持变量、条件判断、循环操作等语法，所以用户可以用 Shell 命令写出各种小程序，又称为脚本（script）。这些脚本都通过 Shell 的解释执行，而不通过编译。

最后，Shell 是一个工具箱，提供了各种小工具，供用户方便地使用操作系统的功能。

Shell 有很多种，只要能给用户提供命令行环境的程序，都可以看作是 Shell。

### 常用命令

#### 基本操作

1.export

显示所有的环境变量，如果你想获取某个变量的详细信息，使用 `echo $VARIABLE_NAME`.

Example:

```bash
$ export
SHELL=/bin/zsh
AWS_HOME=/Users/adnanadnan/.aws
LANG=en_US.UTF-8
LC_CTYPE=en_US.UTF-8
LESS=-R

$ echo $SHELL
/usr/bin/zsh
```

2.whereis

whereis使用系统自动构建的数据库来搜索可执行文件，源文件和手册页面。

Example:

```bash
$ whereis php
/usr/bin/php
```

3.which

它在环境变量PATH指定的目录中搜索可执行文件。此命令将打印可执行文件的完整路径。

```bash
which program_name 
```

Example:

```bash
$ which php
/c/xampp/php/php
```

4.clear

清除窗口上的内容。

#### 文件操作

1.ls

列出您的文件。`ls`有很多选项：`-l`列出“长格式”的文件，其中包含文件的确切大小，拥有该文件的人员，有权查看该文件，以及何时进行上次修改。`-a`列出所有文件，包括隐藏文件。

Example:

```
$ ls -al
rwxr-xr-x   33 adnan  staff    1122 Mar 27 18:44 .
drwxrwxrwx  60 adnan  staff    2040 Mar 21 15:06 ..
-rw-r--r--@  1 adnan  staff   14340 Mar 23 15:05 .DS_Store
-rw-r--r--   1 adnan  staff     157 Mar 25 18:08 .bumpversion.cfg
-rw-r--r--   1 adnan  staff    6515 Mar 25 18:08 .config.ini
-rw-r--r--   1 adnan  staff    5805 Mar 27 18:44 .config.override.ini
drwxr-xr-x  17 adnan  staff     578 Mar 27 23:36 .git
-rwxr-xr-x   1 adnan  staff    2702 Mar 25 18:08 .gitignore
```

2.touch

创建或更新您的文件。

Example:

```bash
$ touch trick.md
```

3.cat

它可以在UNIX或Linux下用于以下目的。

- 在屏幕上显示文本文件
- 复制文本文件
- 合并文本文件
- 创建新的文本文件

```
cat filename
cat file1 file2 
cat file1 file2 > newcombinedfile
```

4.more

显示文件的第一部分（用空格移动并键入q以退出）。

```bash
more filename
```

5.head

输出文件的前10行。

```bash
head filename
```

6.tail

输出最后10行文件。用于-f在文件增长时输出附加数据。

```bash
tail filename
```

7.mv

将文件从一个位置移动到另一个位置。

```bash
mv filename1 filename2
```

`filename1` 文件的源路径，`filename2` 是目标路径。

8.cp

将文件从一个位置复制到另一个位置。

```bash
cp filename1 filename2
```

`filename1` 文件的源路径，`filename2` 是目标路径。

9.rm

删除文件。在目录上使用此命令会给您显示一个错误： `rm: directory: is a directory`。 为了删除目录，你必须传递`-rf`去递归删除目录中的所有内容。

```bash
rm filename
```

10.diff

比较文件，并列出他们的差异。

```bash
diff filename1 filename2
```

11.chmod

让您更改文件的读取，写入和执行权限。

使用：

```
chmod [-cfvR] [--help] [--version] mode file...
```

其中：

- u 表示该文件的拥有者，g 表示与该文件的拥有者属于同一个群体(group)者，o 表示其他以外的人，a 表示这三者皆是。
- \+ 表示增加权限、- 表示取消权限、= 表示唯一设定权限。
- r 表示可读取，w 表示可写入，x 表示可执行，X 表示只有当该文件是个子目录或者该文件已经被设定过为可执行。

其他参数说明：

- -c : 若该文件权限确实已经更改，才显示其更改动作
- -f : 若该文件权限无法被更改也不要显示错误讯息
- -v : 显示权限变更的详细资料
- -R : 对目前目录下的所有文件与子目录进行相同的权限变更(即以递归的方式逐个变更)
- --help : 显示辅助说明
- --version : 显示版本

12.gzip

压缩文件。

```bash
gzip filename
```

13.gunzip

解压缩gzip压缩的文件。

```bash
gunzip filename
```

14.gzcat

让你查看gzip压缩文件，而不需要gunzip它。

```bash
gzcat filename
```

15.lpr

打印文件。

```bash
lpr filename
```

16.lpq

查看打印机队列。

```bash
lpq
```

17.lprm

从打印队列移除某些内容。

```bash
lprm jobnumber
```

#### 文本操作

1.awk

awk是处理文本文件最有用的命令。它一行一行地在整个文件上运行。默认情况下，它使用空格分隔字段。awk命令最常用的语法是

```bash
awk '/search_pattern/ { action_to_take_if_pattern_matches; }' file_to_parse
```

让我们采取以下文件 `/etc/passwd`。以下是此文件包含的示例数据：

```
root:x:0:0:root:/root:/usr/bin/zsh
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
```

所以现在让我们从这个文件只获取用户名。 `-F` 指定在我们要基于哪个分隔字段。在我们的例子中 `:` 。`{ print $1 }` 意味着打印出第一个匹配字段。

```bash
awk -F':' '{ print $1 }' /etc/passwd
```

运行上述命令后，您将获得以下输出。

```
root
daemon
bin
sys
sync
```

2.grep

查找文件内的文本。您可以使用grep搜索与一个或多个正则表达式匹配的文本行，并仅输出匹配的行。

```bash
grep pattern filename
```

Example:

```bash
$ grep admin /etc/passwd
_kadmin_admin:*:218:-2:Kerberos Admin Service:/var/empty:/usr/bin/false
_kadmin_changepw:*:219:-2:Kerberos Change Password Service:/var/empty:/usr/bin/false
_krb_kadmin:*:231:-2:Open Directory Kerberos Admin Service:/var/empty:/usr/bin/false
```

您还可以通过使用`-i`选项强制grep忽略单词大小写。`-r`可用于搜索指定目录下的所有文件，例如：

```bash
$ grep -r admin /etc/
```

`-w` 只搜索单词。

3.wc

告诉你一个文件中有多少行，多少单词和多少字符。

```bash
wc filename
```

Example:

```bash
$ wc demo.txt
7459   15915  398400 demo.txt
```

`7459` 是行数, `15915` 是单词数， `398400` 是字符数.

4.sed

用于过滤和转换文本的流编辑器。

*example.txt*

```bash
Hello This is a Test 1 2 3 4
```

*用连字符替换所有空格*

```bash
sed 's/ /-/g' example.txt
Hello-This-is-a-Test-1-2-3-4
```

*使用"d"替换所有的数字*

```bash
sed 's/[0-9]/d/g' example.txt
Hello This is a Test d d d d
```

5.sort

排序文本文件的行

*example.txt*

```bash
f
b
c
g
a
e
d
```

*sort example.txt*

```bash
sort example.txt
a
b
c
d
e
f
g
```

*随机化一个排序的example.txt*

```bash
sort example.txt | sort -R
b
f
a
c
d
g
e
```

6.uniq

报告或省略重复的行

*example.txt*

```bash
a
a
b
a
b
c
d
c
```

*只显示example.txt的唯一行（首先你需要排序，否则看不到重叠）*

```bash
sort example.txt | uniq
a
b
c
d
```

*显示每行的唯一项，并告诉我找到了多少个实例*

```bash
sort example.txt | uniq -c
    3 a
    2 b
    2 c
    1 d
```

7.cut

从每行文件中删除部分。

*example.txt*

```bash
red riding hood went to the park to play
```

*显示第2,7和9栏的空格作为分隔符*

```bash
cut -d " " -f2,7,9 example.txt
riding park play
```

8.echo

显示一行文字

*显示 "Hello World"*

```bash
echo Hello World
Hello World
```

*用字母之间的换行显示 "Hello World"*

```bash
echo -ne "Hello\nWorld\n"
Hello
World
```

9.fmt

简单的最佳文本格式化程序

*example: example.txt (1 line)*

```bash
Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet.
```

*将example.txt的行输出为20个字符的宽度*

```bash
cat example.txt | fmt -w 20
Lorem ipsum
dolor sit amet,
consetetur
sadipscing elitr,
sed diam nonumy
eirmod tempor
invidunt ut labore
et dolore magna
aliquyam erat, sed
diam voluptua. At
vero eos et
accusam et justo
duo dolores et ea
rebum. Stet clita
kasd gubergren,
no sea takimata
sanctus est Lorem
ipsum dolor sit
amet.
```

10.tr

翻译或删除字符

*example.txt*

```bash
Hello World Foo Bar Baz!
```

*把所有小写字母变成为大写*

```bash
cat example.txt | tr 'a-z' 'A-Z' 
HELLO WORLD FOO BAR BAZ!
```

*把所有的空格变成换行符*

```bash
cat example.txt | tr ' ' '\n'
Hello
World
Foo
Bar
Baz!
```

11.nl

显示文件的行数

*example.txt*

```bash
Lorem ipsum
dolor sit amet,
consetetur
sadipscing elitr,
sed diam nonumy
eirmod tempor
invidunt ut labore
et dolore magna
aliquyam erat, sed
diam voluptua. At
vero eos et
accusam et justo
duo dolores et ea
rebum. Stet clita
kasd gubergren,
no sea takimata
sanctus est Lorem
ipsum dolor sit
amet.
```

*带行号显示 example.txt*

```bash
nl -s". " example.txt 
     1. Lorem ipsum
     2. dolor sit amet,
     3. consetetur
     4. sadipscing elitr,
     5. sed diam nonumy
     6. eirmod tempor
     7. invidunt ut labore
     8. et dolore magna
     9. aliquyam erat, sed
    10. diam voluptua. At
    11. vero eos et
    12. accusam et justo
    13. duo dolores et ea
    14. rebum. Stet clita
    15. kasd gubergren,
    16. no sea takimata
    17. sanctus est Lorem
    18. ipsum dolor sit
    19. amet.
```

12.egrep

打印匹配模式的行 - 扩展表达式（别名为：'grep -E'）

*example.txt*

```bash
Lorem ipsum
dolor sit amet, 
consetetur
sadipscing elitr,
sed diam nonumy
eirmod tempor
invidunt ut labore
et dolore magna
aliquyam erat, sed
diam voluptua. At
vero eos et
accusam et justo
duo dolores et ea
rebum. Stet clita
kasd gubergren,
no sea takimata
sanctus est Lorem
ipsum dolor sit
amet.
```

*在其中显示“Lorem”或“dolor”的行*

```bash
egrep '(Lorem|dolor)' example.txt
or
grep -E '(Lorem|dolor)' example.txt
Lorem ipsum
dolor sit amet,
et dolore magna
duo dolores et ea
sanctus est Lorem
ipsum dolor sit
```

13.fgrep

打印匹配模式到的行 - FIXED模式匹配（别名为：'grep -F'）
*example.txt*

```bash
Lorem ipsum
dolor sit amet,
consetetur
sadipscing elitr,
sed diam nonumy
eirmod tempor
foo (Lorem|dolor) 
invidunt ut labore
et dolore magna
aliquyam erat, sed
diam voluptua. At
vero eos et
accusam et justo
duo dolores et ea
rebum. Stet clita
kasd gubergren,
no sea takimata
sanctus est Lorem
ipsum dolor sit
amet.
```

*在example.txt中找到具体的字符串'（Lorem | doloar）'*

```bash
fgrep '(Lorem|dolor)' example.txt
or
grep -F '(Lorem|dolor)' example.txt
foo (Lorem|dolor) 
```

#### 文件夹操作

1.mkdir

生成一个新的目录。

```bash
mkdir dirname
```

2.cd

执行这个，从一个目录转移到另外一个目录。

```bash
$ cd
```

将你移动到主目录。此命令接受可选的`dirname`，将你移动到该目录。

```bash
cd dirname
```

返回上一层目录

```
cd ..
```

3.pwd

告诉你你目前所在的目录。

```bash
pwd
```

#### SSH,系统信息或网络操作

1.ssh

ssh (SSH client) 是一个用来在登录到远程机器并执行的命令的程序。

```bash
ssh user@host
```

此命令还接受`-p`可用于连接到特定端口的选项。

```bash
ssh -p port user@host
```

2.whoami

返回当前登录用户名。

3.passwd

允许当前登录的用户更改其密码。

4.quota(未在manjaro系统上成功使用)

显示您的磁盘配额。

```bash
quota -v
```

5.date

显示当前日期和时间。

6.cal

显示月份的日历。

7.uptime

显示当前的正常运行时间。

8.w

显示谁在线

9.finger(未在manjaro系统上成功使用)

Displays information about user.

```bash
finger username
```

10.uname

显示内核信息。

```bash
uname -a
```

11.man

显示指定命令的手册。

```bash
man command
```

12.df

显示磁盘使用情况。

13.du

显示文件名中文件和目录的磁盘使用情况（du -s只给出一个总数）。

```bash
du filename
```

14.last

列出您最后登录的指定用户。

```bash
last yourUsername
```

15.ps

列出您的进程。

```bash
ps -u yourusername
```

16.kill

使用您所提供的ID杀死（结束）进程。

```bash
kill PID
```

17.killall

用名称杀死所有进程。

```bash
killall processname
```

18.top

显示当前活动的进程。

19.bg

列出停止的或后台工作的Job; 恢复在后台停止的Job。

20.fg

前台化最近的Job。

21.ping

Pings主机并输出结果。

```bash
ping host
```

22.whois(未在manjaro系统上成功使用)

获取域的whois信息。

```bash
whois domain
```

23.dig(未在manjaro系统上成功使用)

获取域的DNS信息。

```bash
dig domain
```

24.wget

下载文件。

```bash
wget file
```

25.scp

在本地主机和远程主机之间或两台远程主机之间传输文件。

*从本地主机复制到远程主机*

```bash
scp source_file user@host:directory/target_file
```

*从远程主机复制到本地主机*

```bash
scp user@host:directory/source_file target_file
scp -r user@host:directory/source_folder farget_folder
```

此命令还接受`-P`选项可用于连接到特定的端口。

```bash
scp -P port user@host:directory/source_file target_file
```

### 基本语法

####  1.echo命令

`echo`命令的作用是在屏幕输出一行文本，可以将该命令的参数原样输出。

```
$ echo hello world
hello world
```

如果想要输出的是多行文本，即包括换行符。这时需要把多行文本放在引号里面。

```
$ echo "<HTML>
    <HEAD>
          <TITLE>Page Title</TITLE>
    </HEAD>
    <BODY>
          Page body.
    </BODY>
</HTML>"
```



- -n

默认情况下，`echo`输出的文本末尾会有一个回车符。`-n`参数可以取消末尾的回车符，使得下一个提示符紧跟在输出内容的后面。

```
$ echo a;echo b
a
b

$ echo -n a;echo b
ab
```



- -e

`-e`参数会解释引号（双引号和单引号）里面的特殊字符（比如换行符`\n`）。

```
$ echo "Hello\nWorld"
Hello\nWorld

# 双引号的情况
$ echo -e "Hello\nWorld"
Hello
World

# 单引号的情况
$ echo -e 'Hello\nWorld'
Hello
World
```



#### 2.命令格式

命令行环境中，主要通过使用 Shell 命令，进行各种操作。Shell 命令基本都是下面的格式。

配置参数往往分短形式和长形式，如-l 跟 -- list

```
$ command [ arg1 ... [ argN ]]
```

Bash 单个命令一般都是一行，用户按下回车键，就开始执行。有些命令比较长，写成多行会有利于阅读和编辑，这时可以在每一行的结尾加上反斜杠，Bash 就会将下一行跟当前行放在一起解释。

```
$ echo foo bar

# 等同于
$ echo foo \
bar
```



#### 3.空格

Bash 使用空格（或 Tab 键）区分不同的参数。

```
$ command foo bar
```

如果参数之间有多个空格，Bash 会自动忽略多余的空格。

```
$ echo this is a     test
this is a test	
```



#### 4.分号

分号（`;`）是命令的结束符，使得一行可以放置多个命令，上一个命令执行结束后，再执行第二个命令。

```
$ clear; ls
```

使用分号时，第二个命令总是接着第一个命令执行，不管第一个命令执行成功或失败。



#### 5.命令的组合符号&& 和 ||

Bash 还提供两个命令组合符`&&`和`||`，允许更好地控制多个命令之间的继发关系。

```
Command1 && Command2
```

如果`Command1`命令运行成功，则继续运行`Command2`命令。

```
Command1 || Command2
```

如果`Command1`命令运行失败，则继续运行`Command2`命令。



#### 6.type命令

`type`命令用来判断命令的来源。

```
$ type echo
echo is a shell builtin
$ type ls
ls is hashed (/bin/ls)
```

从上面的代码可以看出，echo`是内部命令，`ls`是外部程序（`/bin/ls`）。

如果要查看一个命令的所有定义，可以使用`type`命令的`-a`参数。

```
$ type -a echo
echo is shell builtin
echo is /usr/bin/echo
echo is /bin/echo
```

上面代码表示，`echo`命令即是内置命令，也有对应的外部程序。

`type`命令的`-t`参数，可以返回一个命令的类型：别名（alias），关键词（keyword），函数（function），内置命令（builtin）和文件（file）。

```
$ type -t bash
file
$ type -t if
keyword
```



#### 7.快捷键

下面是一些最常用的快捷键

- `Ctrl + L`：清除屏幕并将当前行移到页面顶部。
- `Ctrl + C`：中止当前正在执行的命令。
- `Shift + PageUp`：向上滚动。
- `Shift + PageDown`：向下滚动。
- `Ctrl + U`：从光标位置删除到行首。
- `Ctrl + K`：从光标位置删除到行尾。
- `Ctrl + D`：关闭 Shell 会话。
- `↑`，`↓`：浏览已执行命令的历史记录。

除了上面的快捷键，Bash 还具有自动补全功能。命令输入到一半的时候，可以按下 Tab 键，Bash 会自动完成剩下的部分。比如，输入`pw`，然后按一下 Tab 键，Bash 会自动补上`d`。

除了命令的自动补全，Bash 还支持路径的自动补全。有时，需要输入很长的路径，这时只需要输入前面的部分，然后按下 Tab 键，就会自动补全后面的部分。如果有多个可能的选择，按两次 Tab 键，Bash 会显示所有选项，让你选择。



### 模式扩展

#### 简介

Shell 接收到用户输入的命令以后，会根据空格将用户的输入，拆分成一个个词元（token）。然后，Shell 会扩展词元里面的特殊字符，扩展完成后才会调用相应的命令。

这种特殊字符的扩展，称为模式扩展（globbing）。其中有些用到通配符，又称为通配符扩展（wildcard expansion）。Bash 一共提供八种扩展。

- 波浪线扩展
- `?` 字符扩展
- `*` 字符扩展
- 方括号扩展
- 大括号扩展
- 变量扩展
- 子命令扩展
- 算术扩展

Bash 允许用户关闭扩展。

```
$ set -o noglob
# 或者
$ set -f
```

下面的命令可以重新打开扩展。

```
$ set +o noglob
# 或者
$ set +f
```

#### 1.波浪线扩展

波浪线`~`会自动扩展成当前用户的主目录。

```
$ echo ~
/home/me
```

`~/dir`表示扩展成主目录的某个子目录，`dir`是主目录里面的一个子目录名。

```
# 进入 /home/me/foo 目录
$ cd ~/foo
```

`~user`表示扩展成用户`user`的主目录。

```
$ echo ~foo
/home/foo

$ echo ~root
/root
```

`~+`会扩展成当前所在的目录，等同于`pwd`命令。

```
$ cd ~/foo
$ echo ~+
/home/me/foo
```



#### 2.？字符扩展

`?`字符代表文件路径里面的任意单个字符，不包括空字符。比如，`Data???`匹配所有`Data`后面跟着三个字符的文件名。

```
# 存在文件 a.txt 和 b.txt
$ ls ?.txt
a.txt b.txt
```

如果匹配多个字符，就需要多个`?`连用。

```
# 存在文件 a.txt、b.txt 和 ab.txt
$ ls ??.txt
ab.txt
```



#### 3.*字符扩展

`*`字符代表文件路径里面的任意数量的任意字符，包括零个字符。

```
# 存在文件 a.txt、b.txt 和 ab.txt
$ ls *.txt
a.txt b.txt ab.txt
```

`*`可以匹配空字符，下面是一个例子。

```
# 存在文件 a.txt、b.txt 和 ab.txt
$ ls a*.txt
a.txt ab.txt

$ ls *b*
b.txt ab.txt
```

注意，`*`不会匹配隐藏文件（以`.`开头的文件），即`ls *`不会输出隐藏文件。

如果要匹配隐藏文件，需要写成`.*`。

```
# 显示所有隐藏文件
$ echo .*
```

如果要匹配隐藏文件，同时要排除`.`和`..`这两个特殊的隐藏文件，可以与方括号扩展结合使用，写成`.[!.]*`。

```
$ echo .[!.]*
```

`*`只匹配当前目录，不会匹配子目录。

```
# 子目录有一个 a.txt
# 无效的写法
$ ls *.txt

# 有效的写法
$ ls */*.txt
```



#### 4.方括号扩展

方括号扩展的形式是`[...]`，只有文件确实存在的前提下才会扩展。如果文件不存在，就会原样输出。括号之中的任意一个字符。比如，`[aeiou]`可以匹配五个元音字母中的任意一个。

```
# 存在文件 a.txt 和 b.txt
$ ls [ab].txt
a.txt b.txt

# 只存在文件 a.txt
$ ls [ab].txt
a.txt
```

方括号扩展还有两种变体：`[^...]`和`[!...]`。它们表示匹配不在方括号里面的字符，这两种写法是等价的。比如，`[^abc]`或`[!abc]`表示匹配除了`a`、`b`、`c`以外的字符。

```
# 存在 aaa、bbb、aba 三个文件
$ ls ?[!a]?
aba bbb
```

上面命令中，`[!a]`表示文件名第二个字符不是`a`的文件名，所以返回了`aba`和`bbb`两个文件。

注意，如果需要匹配`[`字符，可以放在方括号内，比如`[[aeiou]`。如果需要匹配连字号`-`，只能放在方括号内部的开头或结尾，比如`[-aeiou]`或`[aeiou-]`。



#### 5.[start-end] 扩展

方括号扩展有一个简写形式`[start-end]`，表示匹配一个连续的范围。比如，`[a-c]`等同于`[abc]`，`[0-9]`匹配`[0123456789]`。

```
# 存在文件 a.txt、b.txt 和 c.txt
$ ls [a-c].txt
a.txt
b.txt
c.txt

# 存在文件 report1.txt、report2.txt 和 report3.txt
$ ls report[0-9].txt
report1.txt
report2.txt
report3.txt
...
```

下面是一些常用简写的例子。

- `[a-z]`：所有小写字母。
- `[a-zA-Z]`：所有小写字母与大写字母。
- `[a-zA-Z0-9]`：所有小写字母、大写字母与数字。
- `[abc]*`：所有以`a`、`b`、`c`字符之一开头的文件名。
- `program.[co]`：文件`program.c`与文件`program.o`。
- `BACKUP.[0-9][0-9][0-9]`：所有以`BACKUP.`开头，后面是三个数字的文件名。



#### 6.大括号扩展

大括号扩展`{...}`表示分别扩展成大括号里面的所有值，各个值之间使用逗号分隔。比如，`{1,2,3}`扩展成`1 2 3`。

```
$ echo {1,2,3}
1 2 3

$ echo d{a,e,i,u,o}g
dag deg dig dug dog

$ echo Front-{A,B,C}-Back
Front-A-Back Front-B-Back Front-C-Back
```

注意，大括号扩展不是文件名扩展。它会扩展成所有给定的值，而不管是否有对应的文件存在。

另一个需要注意的地方是，大括号内部的逗号前后不能有空格。否则，大括号扩展会失效。

```
$ echo {1 , 2}
{1 , 2}
```

上面例子中，逗号前后有空格，Bash 就会认为这不是大括号扩展，而是三个独立的参数。

逗号前面可以没有值，表示扩展的第一项为空。

```
$ cp a.log{,.bak}

# 等同于
# cp a.log a.log.bak
```

大括号可以嵌套;大括号也可以与其他模式联用，并且总是先于其他模式进行扩展。



#### 7.{start..end} 扩展

大括号扩展有一个简写形式`{start..end}`，表示扩展成一个连续序列。比如，`{a..z}`可以扩展成26个小写英文字母。



#### 8.变量扩展

Bash 将美元符号`$`开头的词元视为变量，将其扩展成变量值。

```
$ echo $SHELL
/bin/bash
```

变量名除了放在美元符号后面，也可以放在`${}`里面。

```
$ echo ${SHELL}
/bin/bash
```



#### 9.子命令扩展

`$(...)`可以扩展成另一个命令的运行结果，该命令的所有输出都会作为返回值。

```
$ echo $(date)
Tue Jan 28 00:01:13 CST 2020
```



#### 10.算数扩展

`$((...))`可以扩展成整数运算的结果

```
$ echo $((2 + 2))
4
```



#### 11.字符类

`[[:class:]]`表示一个字符类，扩展成某一类特定字符之中的一个。常用的字符类如下。

- `[[:alnum:]]`：匹配任意英文字母与数字
- `[[:alpha:]]`：匹配任意英文字母
- `[[:blank:]]`：空格和 Tab 键。
- `[[:cntrl:]]`：ASCII 码 0-31 的不可打印字符。
- `[[:digit:]]`：匹配任意数字 0-9。
- `[[:graph:]]`：A-Z、a-z、0-9 和标点符号。
- `[[:lower:]]`：匹配任意小写字母 a-z。
- `[[:print:]]`：ASCII 码 32-127 的可打印字符。
- `[[:punct:]]`：标点符号（除了 A-Z、a-z、0-9 的可打印字符）。
- `[[:space:]]`：空格、Tab、LF（10）、VT（11）、FF（12）、CR（13）。
- `[[:upper:]]`：匹配任意大写字母 A-Z。
- `[[:xdigit:]]`：16进制字符（A-F、a-f、0-9）。



#### 12.使用注意点

1. 通配符是先解释，再执行。

2. 文件名扩展在不匹配时，会原样输出。

3. 只适用于单层路径。

   所有文件名扩展只匹配单层路径，不能跨目录匹配，即无法匹配子目录里面的文件。或者说，`?`或`*`这样的通配符，不能匹配路径分隔符（`/`）。

4. 文件名可以使用通配符。

   Bash 允许文件名使用通配符，即文件名包括特殊字符。这时引用文件名，需要把文件名放在单引号里面。



### 引号和转义

#### 1.转义

某些字符在 Bash 里面有特殊含义（比如`$`、`&`、`*`）。如果想要原样输出这些特殊字符，就必须在它们前面加上反斜杠，使其变成普通字符。这就叫做“转义”（escape）。

```
$ echo \$date
$date
```

反斜杠除了用于转义，还可以表示一些不可打印的字符。

- `\a`：响铃
- `\b`：退格
- `\n`：换行
- `\r`：回车
- `\t`：制表符

如果想要在命令行使用这些不可打印的字符，可以把它们放在引号里面，然后使用`echo`命令的`-e`参数。

```
$ echo a\tb
atb

$ echo -e "a\tb"
a        b
```



#### 2.单引号

Bash 允许字符串放在单引号或双引号之中，加以引用。

单引号用于保留字符的字面含义，各种特殊字符在单引号里面，都会变为普通字符，比如星号（`*`）、美元符号（`$`）、反斜杠（`\`）等。

```
$ echo '*'
*

$ echo '$USER'
$USER

$ echo '$((2+2))'
$((2+2))

$ echo '$(echo foo)'
$(echo foo)
```



#### 3.双引号

双引号比单引号宽松，大部分特殊字符在双引号里面，都会失去特殊含义，变成普通字符。

三个特殊字符除外：美元符号（`$`）、反引号（```）和反斜杠（`\`）。这三个字符在双引号之中，依然有特殊含义，会被 Bash 自动扩展。

```
$ echo "$SHELL"
/bin/bash

$ echo "`date`"
Mon Jan 27 13:33:18 CST 2020
```



#### 

