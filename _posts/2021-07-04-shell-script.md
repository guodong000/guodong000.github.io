---
layout: post
title: Shell 脚本简单速查
---

> 一份不全面的速查笔记，仅用于临时抱佛脚

- [参数变量 `$# $@ $1 $2 ...`](#variable)
- [控制结构](#controlstruct)
- [命令执行 `$(...)`](#command)
- [算术扩展 `$((...))`](#expr)
- [参数扩展 `${...}`](#paramexpand)

# 管道与重定向

```bash
ls -l > out.txt     # 重定向输出（覆盖）
ls -l >> out.txt    # 重定向输出（追加）

# 0: 标准输入
# 1: 标准输出
# 2: 标准错误输出
kill -HUP 123 >out.txt 2>err.txt    # 标准输出与错误输出分别重定向

# >& 操作符用于结合输出
kill -1 123 >out.txt 2>&1           # 标准输出与错误输出重定向到同一个文件

more < out.txt      # 重定向输入

# 管道连接的进程可以同时运行，并随着数据流自动的进行协调。
ps -xo comm | sort | uniq | grep -v sh | more

# here 文档
cat <<!EOF!     # << 为标签重定向符，标签是一个特殊序列不能出现在文档中
hello
world
!EOF!
```

# 语法

## <a id="variable"></a> 变量

默认情况下，所有变量都被看作字符串，并以字符串来存储。

变量名区分大小写，通过变量名前添加 `$` 符号进行访问。

```bash
hello=World     # 注意 = 左右不能有空格
echo $hello

read userinput  # read 命令将用户输入赋值给一个变量
echo $userinput

"$hello"        # 这里的变量会展开为其值
'$hello'        # 不会展开变量
```

| 参数/环境变量 | 说明 |
| --- | --- |
| `$0` | 脚本名称（环境变量） |
| `$1` `$2` | 脚本程序的参数 |
| `$*` | 所有参数，各参数由 `$IFS` 的第一个字符分隔开 |
| `$@` | 所有参数，不依赖 `$IFS` 环境变量 |
| `$#` | 传递给脚本的参数个数（环境变量） |
| `$$` | 脚本的进程号（环境变量） |

<small>👆 参数变量仅在有相应参数时被创建。即使没有任何参数，环境变量依然存在。</small>

## <a id="controlstruct"></a> 控制结构

### test 命令

`test` 与 `[` 等价，只是为了可读性，使用 `[` 时会使用 `]` 结尾。

```bash
if test -f foo; then echo 'exist'; fi
# 等价于
if [ -f foo ]; then echo 'exist'; fi

# 字符串比较
str1 = str2         # 两字符串相同则为真
str1 != str2        # 两字符串不同则为真
-n str              # 字符串不为空则为真
-z str              # 字符串为 null 则为真

# 算术比较
exp1 -eq exp2       # 两表达式相等
exp1 -ne exp2       # 两表达式不等
exp1 -gt exp2       # 大于
exp1 -ge exp2       # 大于等于
exp1 -lt exp2       # 小于
exp1 -le exp2       # 小于等于
! exp               # 取反

# 文件测试
-d file             # 文件是目录则为真
-e file             # 文件存在则为真（-e 不可移植，通常使用 -f）
-f file             # 普通文件则为真
-s file             # 文件大小不为 0 则为真
-r file             # 文件可读则为真
-w file             # 文件可写则为真
-x file             # 文件可执行则为真
-g file             # 文件 set-group-id 位被设置则为真
-u file             # 文件 set-user-id 位被设置则为真
```

### 命令列表 AND OR

**命令退出码为 0 表示成功**。

`&&` 与 `||` 连接的命令执行时为短路径求值 (short circuit evaluation)。

```bash
# AND 列表
statement1 && statement2 && statement3 && ...

# 当 foo 文件存在时才执行 echo 'hello' 命令，即短路与
if [ -f foo ] && echo 'hello'; then
    echo 'True'
fi

# OR 列表
statement1 || statement2 || statement3 || ...

# 当 foo 文件不存在时才执行 echo 'hello' 命令，即短路或
if [ -f foo ] || echo 'hello'; then
    echo 'True'
fi
```

### 语句块

当要在只允许使用单个语句的地方（如 AND 或 OR 列表中）使用多条语句，可以使用 `{}` 构造一个语句块。

```bash
[ -f foo ] && {
    echo 'hello'
    echo 'world'
}
```

### if 语句

**注意：** condition 通常采用 test 命令，condition 命令的退出码决定如何执行条件代码。

```bash
##### 语法结构 ##########
if condition
then
    statements
else
    statements
fi
########################

# 给变量加上引号，用于防止空变量导致的错误
if [ "$var" = 'hello' ]; then
    echo 'hello'
elif [ "$var" = 'world' ]; then
    echo 'world'
else
    echo 'other'
fi
```

### for 语句

```bash
##### 语法结构 ##########
for variable in values
do
    statements
done
########################

for foo in bar fud 43; do
    echo $foo
done

for file in $(ls f*.sh); do
    echo $file
done
```

### while 语句

当条件为真时反复执行。

```bash
##### 语法结构 ##########
while condition 
do
    statements
done
########################


while [ "$input" != "secret" ]; do
    echo "Sorry, try again"
    read input
done
```

### until 语句

与 `while` 类似，循环反复执行，直到条件为真。

``` bash
##### 语法结构 ##########
until condition
do
    statements
done
########################

# 当某个特定用户登录时输出提示
until who | grep "$1" > /dev/null; do
    sleep 60
done
echo "$1 has just logged in!"
```

### case 语句

将变量的内容与模式进行匹配，然后根据匹配的模式去执行不同的代码。

```bash
##### 语法结构 ##########
case variable in
    pattern [ | pattern] ...) statements;;
    pattern [ | pattern] ...) statements;;
    ...
esac
########################

read input
case "$input" in
    yes | y | Yes | YES )   echo "Yes";;    # 单行写法
    [nN]* )                                 # 多行命令写法
        echo "No"
        ;;  # ;; 相当于 break
    * )                                     # 默认条件
        echo "Other"
        exit 1
        ;;
esac
```

## 函数

函数在调用之前必须先定义。

当一个函数被调用时，位置参数与相关环境变量（`$*` `$@` `$#` `$1` `$2` 等）会被替换为函数的参数。当函数执行完毕后，这些参数会恢复之前的值。

函数可以通过 `return` 命令返回一个数字值，返回字符串的方式可以使用 `echo` 命令，或将返回值存在一个变量中。

如果函数没有使用 `return` 指定返回值，则默认返回函数中最后一条命令的退出码。

```bash
##### 语法结构 ##########
function_name() {
    statements
}
########################

foo() { 
    local text='Hello'  # 声明局部变量，仅在函数的作用域内有效
    echo $text
}
foo                     # 调用函数
res = $(foo)            # 捕获函数返回的字符串

foo1() {
    echo "param is $1"              # 获取参数
    if [ "$1" = "true" ]; then
        return 0                    # 返回 0 表示 true
    else
        return 1                    # 返回非 0 表示 false
    fi
}
param="true"
if foo1 "$param"; then              # 带参数调用
    echo "True"
else
    echo "False"
fi
```

# <a id="command"></a> 命令

Shell 脚本内使用的命令分为两种

* 外部命令：独立存在于 Shell 之外的命令
* 内部命令：Shell 内置的，不能作为外部程序被调用。不过按照 POSIX 标准，大多数内部命令也同时提供了独立运行的程序。

使用 `$(command)` 捕获一条命令的执行结果，是命令的输出，而非退出码。

```bash
# break 命令
# ---------------------
break number        # number 指示跳出的循环层数，不指定默认只跳出一层循环

# continue 命令
# ---------------------
continue number     # number 指示继续执行的循环嵌套层数，一般不使用

# : 命令是一个空命令
# ---------------------
# 偶尔被用于简化逻辑，此时相当于 true 的别名
while : 
    do
    ...
done
: ${var:=value}     # 如果没有 : ，shell 将试图把 $var 当作一条命令来处理

# . 命令
# ---------------------
# 通常，当一个脚本执行一条外部命令或脚本程序时，会创建一个新环境（子shell），
# 命令将在这个新环境中执行，执行完毕后新环境被丢弃，退出码返回给父 shell
# . 命令/source 在执行命令时使用的是当前 shell
. ./script          # 在当前 shell 中运行 ./script，使得脚本程序可以改变当前 shell 中的环境设置

# echo 命令
# ---------------------
echo -n "hello"     # 输出并去掉自带换行符
echo -e "hello \c"  # 启用转义并去掉自带换行符

# eval 命令
# ---------------------
# 对参数进行求值
foo=10; x=foo; y='$'$x; echo $y         # 输出 $foo
foo=10; x=foo; eval y='$'$x; echo $y    # 输出 10

# exec 命令
# ---------------------
exec wall "Hello"   # 将当前 shell 替换为另一个程序
exec 3< ./file      # 打开文件操作符 3 以便从 ./file 中读取数据

# exit 命令
# ---------------------
exit n      # 使脚本程序以退出码 n 结束运行，0 表示成功，1-125 为可使用代码，其余数字系统保留
[ -f file ] && exit 0 || exit 1

# export 命令
# ---------------------
# 默认在一个 shell 中创建的变量在子 shell 中不可用
# export 将作为其参数的变量导出到子 shell 中
# 更确切的说，被导出的变量构成从该 shell 衍生的任何子进程的环境变量
export hello="world"

# printf 命令
# ---------------------
# X/Open 规范建议使用 printf 代替 echo
# 与 C 语言中 printf 类似，最大的不同为不支持浮点数
printf "format string" param1 param2 ...

printf "%s %d\t%s" "Hello" 10 world

# return 命令
# ---------------------
# 使函数返回，指定的参数被看作函数的返回值，不指定则默认返回最后一条命令的退出码
return n

# set 命令
# ---------------------
# 为 shell 设置参数变量
set $(date)     # 将命令输出设置为参数变量
echo "The month is $2"

# 控制 shell 的执行方式
set -x          # 开启显示当前执行的命令

# unset 命令
# ---------------------
# 从环境中删除变量或函数，不能删除 shell 本身定义的只读变量（如 IFS）
foo = "Hello"
unset foo       # 删除变量 foo

# shift 命令
# ---------------------
# 将所有参数变量左移一个位置，原来 $1 的值被丢弃，$0 保持不变，相应的 $* $@ $# 等也会进行相关变动
shift n     # 如果指定数值参数，则表示左移次数，不指定默认为 1

while [ "$1" != "" ]; do
    echo "$1"   # 依次输出所有位置参数
    shift
done

# trap 命令
# ---------------------
# 用于指定在接收到信号后要采取的行动
trap command signal

# X/Open 规定的能被捕获的一些比较重要的信号
# HUP(1)        挂起，通常因终端掉线或用户退出引发
# INT(2)        中断，通常因按下 Ctrl+C 引发
# QUIT(3)       退出，通常因按下 Ctrl+\ 引发
# ABRT(6)       中止，通常因某些严重的执行错误引发
# ALRM(14)      报警，通常用来处理超时
# TERM(15)      终止，通常在系统关机时发送

trap - signal       # 重置 signal 的处理方式到默认值
trap signal         # 忽略 signal

date > /tmp/tmp_file_$$
trap 'rm -f /tmp/tmp_file_$$' INT   # 在要保护的代码钱指定 trap 命令
while [ -f /tmp/tmp_file_$$ ]; do
    echo "wait interrupt (CTRL-C)"  # Ctrl-C 触发中断，执行相应操作
    sleep 1
done
echo "File no longer exists"

```

## <a id="expr"></a> expr 命令

将其参数当作一个表达式来求值。最常见的用法就是进行简单数学运算。

在较新的脚本程序中，`expr` 通常被替换为更有效的 `$((...))` 语法。

```bash
x=`expr $x + 1`     # `` 用于给变量取值
x=$(expr $x + 1)    # 等价于上边

# expr 常用的求值计算
expr1 & expr2       # 任一个表达式为 0 则为 0，否则为 expr1
expr1 | expr2       # expr1 非 0 则为 expr1，否则为 expr2
expr1 = expr2       # 等于
expr1 > expr2       # 大于
expr1 >= expr2      # 大于等于
expr1 < expr2       # 小于
expr1 <= expr2      # 小于等于
expr1 != expr2      # 不等于
expr1 + expr2       # 加
expr1 - expr2       # 减
expr1 * expr2       # 乘
expr1 / expr2       # 除
expr1 % expr2       # 取余
```

## 算术扩展

`expr` 命令可以处理一些简单的算数命令，但执行起来比较慢，一般建议采用 `$((...))` 扩展。

```bash
x=0
while [ "$x" -ne 10 ]; do
    echo $x
    x=$(($x+1))     # 累加
done
```

## <a id="paramexpand"></a> 参数扩展

```bash
for i in 1 2; do
    some_process $i_tmp     # shell 试图替换变量 $i_tmp 的值
    some_process ${i}_tmp   # shell 会使用变量 i 的值替换 ${i}
done

# 常见参数扩展方法
${param:-default}       # 如果 param 为空，则为 default
${param:=default}       # 如果 param 为空，将 param 赋值为 default
${param:?msg}           # 如果 param 为空，则显示 msg 并退出脚本
${param:+default}           # 如果 param 不为空，则为 default
${#param}               # 给出 param 的长度
${param%word}           # 从 param 的尾部删除 word 的最短匹配，然后返回剩余部分
${param%%word}          # 从 param 的尾部删除 word 的最长匹配，然后返回剩余部分
${param#word}           # 从 param 的头部删除 word 的最短匹配，然后返回剩余部分
${param##word}          # 从 param 的头部删除 word 的最长匹配，然后返回剩余部分
```

## 脚本调试

设置 shell 选项的方式可以在调用 shell 时加上命令行选项，或使用 `set` 命令。

| 命令行选项 | set 命令 | 说明 |
| --- | --- | --- |
| `sh -n <script>` | `set -o noexec`<br>`set -n` | 只检查语法错误，不执行命令 |
| `sh -v <script>` | `set -o verbose`<br>`set -v` | 在执行命令之前回显 |
| `sh -x <script>` | `set -o xtrace`<br>`set -x` | 在执行命令之后回显 |
| `sh -u <script>` | `set -o nounset`<br>`set -u` | 如果使用了未定义的变量，就给出出错消息 |

`set -o xtrace` 表示启用，`set +o xtrace` 表示取消设置。
