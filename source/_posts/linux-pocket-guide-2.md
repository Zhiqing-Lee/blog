---
title: 《Linux pocket guide》读书笔记之二：Shell特性
date: 2016-07-20
categories:
- Linux
tags:
- Linux
- Code
- Shell
---

使用Linux操作系统时，通常是在一个叫Shell的东西里输入命令然后敲入回车，从而让Linux执行相应的程序。比如在Shell中输入`who`然后敲入回车，系统将会输出有哪些用户在什么地方登录了该计算机。

- `|` 为管道符，用于链接两条命令，作用是将第一条命令的输出作为第二条命令的输入。如`who | wc -l`将会显示当前有多少用户登录了该计算机（`wc -l`命令的功能为显示输入数据的行数）。
- `echo $SHELL` 可以显示shell的类型，大多数linux发型版都默认为bash(the Bourne-Again Shell)，输出为`/bin/bash`。
- `exit` 退出当前shell

## Shell命令对应的程序

当你输入一条命令时，实际上是调用了一个linux程序或者是一条内置的命令。可以使用`type`命令显示一个命令的类型或程序的位置：

```bash
type who  #输出: who is /usr/bin/who
type cd  #输出: cd is a shell builtin
```

## bash的特点

### 通配符

通配符是用来代替一个或多个字符的符号，例如：`a*`的意思是所有以字符`a`开头的文件。如果在shell中输入`ls a*`会显示当前文件夹下所有以`a`开头的文件。和输入`ls apple applet ada`效果相同。

> ls命令并不知道你使用了通配符，通配符是由shell处理的，所以原则上每个命令都能使用通配符。

点文件：

> 点文件是指文件名以英文句点开头的文件，如：`.profile`。这种文件在很多程序中默认是不可见的，也被称为隐藏文件
> - `ls` 命令默认不显示点文件，除非加上`-a`选项
> - shell的通配符不匹配点文件

通配符：

- `*` 零个或多个字符
- `?` 单个字符
- `[set]` 给定集合中的任意字符，如:`[aeiouAEIOU]`将匹配任意一个元音字母，也可以给定一个范围，如:`[A-Z]`将匹配任意一个大写字母
- `[^set]` 匹配任意一个不在给定集合中的字符，如`[^0-9]`就爱那个匹配一个不是数字的字符
- `[!set]` 同`[^set]`

### 大括号匹配

和通配符类似，大括号里的表达式将会被分为多个参数传给命令：

`{X,YY,ZZZ}`

shell会分别把X、YY、ZZZ传给命令行，例如：

```bash
echo sand{X,YY,ZZZ}wich  
#输出：sandXwich sandYYwich sandZZZwich
```

### Shell变量

你可以定义变量并赋值：

```bash
MYVAR=3
```

通过美元符号加变量名可以取出变量的值：

```bash
echo $MYVAR  #输出： 3
```

在启动shell之前系统通常会帮你定义的一些变量：

变量 | 含义
----|----------------
DISPLAY | 显示器名称
HOME | 家目录地址，如：`/home/zhiqing`
LOGNAME | 登录的用户名，如：`zhiqing`
OLDPWD | 上一个`cd`命令执行之前的目录
PATH | shell搜索路径，用`:`分隔的目录
PWD | 当前目录
SHELL | shell的路径，如：`/bin/bash`
TERM | 终端的类型，如：`xterm`
USER | 当前登录的用户名

变量的作用域一般是默认是当前Shell，如果想让其他shell和程序也能调用该变量，需使用`export`命令：

```bash
MYVAR=3
export MYVAR  
export MYVAR2=3  #功能和上两行一样
```

这样的变量叫做`环境变量(environment variable)`,通过`printenv`命令可以输出环境变量：

```bash
printenv  #输出所有环境变量
printenv HOME  #输出单个环境变量，输出：`/home/zhiqing`
HOME=/home/tom printenv HOME  #输出:`/home/tom`
printenv HOME  #输出：`/home/zhiqing`
```

如上所示，如需临时改变环境变量的值，可在命令前加上`变量=值`。

### 搜索路径(PATH)

Linux系统中的程序存放在各种各样的目录中，如`/bin`、`/usr/bin`、`/opt/java/jdk/bin`。为了方便shell能够正确找到该程序，shell提供了`PATH`环境变量：

```bash
echo $PATH  #输出：`/usr/local/bin:/bin:/usr/bin`
```

如上所示，PATH中的值是用`:`分隔的多个目录。shell执行程序时会在这些目录中一个一个地查找，找到了便运行该程序，否则提示`bash: who: command not found`。

你也可以添加自己的程序目录到环境变量：

```bash
PATH=$PATH:/home/zhiqing/prog
echo $PATH  
#输出:`/usr/local/bin:/bin:/usr/bin:/homae/zhiqing/prog`
```

这样只会影响当前shell，在`~/.bash_profile`文件中设置PATH变量可使以后每次开启Shell自动设置。

### 别名(Aliases)

可以使用内置的`alias`命令为长命令创建别名以方便记忆和输入：

```bash
alias ll='ls -lG'
ll  #此时`ll`与`ls -lG`的输出相同
```

在`~/.bash_profile`中定义别名，可以使以后登录后直接使用你定义的别名。

### 输入/输出重定向

Shell可以把`srandard input(标准输入)`、`srandard output(标准输出)`、`srandard error(标准错误输出)`重定向到文件或从文件重定向：

```bash
command < infile  #重定向文件到标准输入
command > outfile  #重定向标准输出到一个文件（创建新文件/覆盖就文件）
command >> outfile  #重定向标准输出到一个文件（追加到文件之后）
command 2> errorfile  #重定向标准错误输出到文件
command > outfile 2> errorfile  
#分别重定向标准输出和标准错误输出到不同的文件
command >& outfile
command &> outfile
#以上两个均为重定向标准输出和标准错误输出到一个共同的文件
```

### 管道(Pipes)

使用管道(`|`)你可以重定向一条命令的标准输出到另一条命令的标准输入：

```bash
ls | wc -l  #输出当前目录下的文件个数
ls | wc -l | cowsay  #让一只母牛说出当前目录下的文件个数
```

### 子过程

与管道类似，使用子过程`<(command)`可以把一条命令的输出以文件的形式重定向到另一条命令：

```bash
cat <(ls | wc -l | cowsay)
#让一只母牛说出当前目录下的文件个数，但`cat`命令接收的参数为文件
```

### 执行多条命令

用`;`分隔一行内的多条命令，可以使这些命令按次序依次执行：

```bash
command1 ; command2 ; command3
#依次执行command1, command2, command3
```

用`&&`分隔一行内的多条命令，可以使前面的所有命令均执行成功再执行后面的命令：

```bash
command1 && command2 && command3
#先执行command1，如成功则执行command2，否则退出
#command1和command2均执行成功才会执行command3
```

用`||`分隔一行内的多条命令，可以使前面的所有命令中只要有一条执行成功则执行后面的命令：

```bash
command1 || command2 || command3
#先执行command1，如成功则执行command2，否则退出
#command1和command2只要有一条执行成功会执行command3
```

### 引用

通常一条命令的参数中如过有空格，则该参数会被分为多个参数传递给命令。如需传递包含空格的参数，需使用单引号`‘`或双引号`"`引起来。双引号中的字符串会被解析而单引号不会：

```bash
echo 'HOME变量的值为$HOME'
#输出：HOME变量的值为$HOME
echo "HOME变量的值为$HOME"
#输出：HOME变量的值为/home/zhiqing
```

一对反引号(\`)中可以包含一条命令，该处的内容会被命令的输出所代替：

```bash
date +%Y  #输出当前年：2016
echo This year is `date +%Y`  #输出：This year is 2016-07
```

`$(command)`有同样的功能，但是使用`$(command)`更好,因为它可以嵌套：

```bash
echo This year is $(date +%Y)  #输出：This year is 2016
echo This year is $(expr $(date +%Y) + 1)
#输出：This year is 2017
```

### 转义

有时候你想在参数中包含已被Shell使用的特殊符号，如`*`、`$`，这时可以使用转义符`\`：

```bash
echo a*  #输出：apple applet app
echo a\*  #输出：a*
echo "I live in $HOME"  #输出：I live in /home/zhiqing
echo "I live in \$HOME"  #输出：I live in $HOME
```

还可以使用`^V`(Ctrl + v)来转义控制字符，如在输入制表符(`tab`)之前按下`^V`，即可以把制表符输入参数中。

> `tab`键的功能将在后面提到

### Shell历史(History)

通过Shell历史你可以重新执行你之前执行过的命令，常见的一些命令如下：

命令 | 含义
----|---------
`history` | 输出执行命令的历史记录
`history N` | 输出最近N条记录
`history -c` | 清除历史记录
`!!` | 重新执行上一条命令
`!N` | 重新执行在历史记录中的第N条命令
`!-N` | 重新执行最近的第N条记录
`!$` | 将上一条命令的最后一个参数作为当前命令的参数，如先执行`ls z*`，然后执行`rm !$`，此时的`rm !$`等同于`rm z*`
`!*` | 将上一条命令的所有参数作为当前命令的参数

### 代码补全

如果你只记得命令或文件名的前几个字符，你可以通过按`tab`键来进行自动补全：

```bash
cd /usr/bin
ls un<Tab><Tab>  
#系统将会显示出/usr/bin目录下所有以un开头的文件
```

## Sehll工作控制

- `jobs` 列出工作列表
- `&` 在后台运行一个工作
- `^Z` 暂停当前前台工作
- `suspend` 暂停一个shell
- `fg` 恢复一个工作并将其显示到前台
- `bg` 让一个已暂停的工作在后台运行

### jobs

这是一个内置命令，用于显示当前shell上的工作列表：

```bash
jobs
#输出：
#[1]-  Running       emacs myfile &
#[2]+  Stopping      shh exaple.com
```

左边方括号中的整数为工作编号，旁边的加号和减号用于区别是前台工作还是后台工作。

### &

将`&`符号放在命令行之后，即可以让该命令在后台运行：

```bash
emacs myfile &
#输出 [2] 28090
```

返回的结果包括工作编号(2)和命令的进程号(28090)。


### ^Z

当一个工作在前台运行时，按下`^Z`会使工作停止暂停，但是状态不变（前台/后台）：

```bash
sleep 10  #等待10秒
^Z
#输出：[1]+ Stopped        sleep 10
```

现在你可以输入`bg`使该工作在后台运行，或者输入`fg`使该工作恢复到前台运行。当然，你也可以不管该命令而继续执行其他命令。

### suspend

`suspend`是一个内置命令，用于暂停当前shell，就像在当前shell中按`^Z`。比如当你用`sudo`命令启动了一个超级用户shell，而你又想返回原shell：

```bash
whoami
#zhiqing
sudo bash
#Password: *******
whoami
#root
suspend
#[1]+  Stopped        sudo bash
whoami
#zhiqing
```

### bg

`bg`命令可以使一个已暂停的命令在后台继续运行。如果没有参数则运行最近暂停的一个工作。如果需要指定一项明确的工作，则在工作编号前加上百分号传给`bg`命令作为参数即可：

```bash
bg %2
```

一些交互型的程序不支持在后台运行，你可以使用`fg`命令使它们在前台运行。

### fg

`fg`命令可以使一个已暂停的命令在前台继续运行。如果没有参数则通常运行最近暂停的一个工作或在在后台运行的工作。如果需要指定一项明确的工作，则在工作编号前加上百分号传给`bg`命令作为参数即可：

```bash
fg %2
```

## 结束一条命令

可以使用`^C`结束一条正在shell前台运行的命令.如果你想结束一条在后台运行的工作，可以使用`fg`命令使它在前台运行，然后按`^C`结束该工作。

> 另一种选择是使用`kill`命令。

## 退出Shell

可以使用`exit`命令或者按`^D`键退出当前shell。

## 定制Shell

你可以定制你自己的shell一边更好的使用或工作。你可以在你的家目录找到并编辑`.bash_profile`或`.bashrc`来达到定制shell。它们可以定义变量或别名、运行程序、打印你的星座运势或者是任何你想做的事。
