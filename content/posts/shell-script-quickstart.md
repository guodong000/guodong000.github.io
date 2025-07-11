+++
date = '2021-07-04T11:38:24+08:00'
draft = false
title = 'Shell 脚本快速入门'
showtoc = true
nosummary = true
+++

## 管道与重定向
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

## 语法

### 变量

默认情况下，所有变量都被看作字符串，并以字符串来存储。

变量名区分大小写，通过变量名前添加 `$` 符号进行访问。

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
| `$*` | 所有参数，作为一个整体 |
| `$@` | 所有参数，各参数由 `$IFS` 的第一个字符分隔开，可使用 for 循环 |
| `$#` | 传递给脚本的参数个数（环境变量） |
| `$$` | 脚本的进程号（环境变量） |
| `$?` | 上一个命令的退出状态码（环境变量） |

<small>👆 参数变量仅在有相应参数时被创建。即使没有任何参数，环境变量依然存在。</small>


对于第9个参数之后的参数，可以通过 `${pos}` 进行访问，比如 `${10}`。

### 控制结构

#### test 命令

`test` 与 `[` 等价，只是为了可读性，使用 `[` 时会使用 `]` 结尾。

```bash
if test -f foo; then echo 'exist'; fi
# 等价于
if [ -f foo ]; then echo 'exist'; fi

# 字符串比较（大于号和小于号需要转义）
str1 = str2         # 两字符串是否相同
str1 != str2        # 两字符串是否不同
str1 < str2         # str1 是否小于 str2
str1 > str2         # str1 是否大于 str2
-n str              # 字符串长度是否非0
-z str              # 字符串长度是否为0

# 算术比较
exp1 -eq exp2       # 两表达式相等
exp1 -ne exp2       # 两表达式不等
exp1 -gt exp2       # 大于
exp1 -ge exp2       # 大于等于
exp1 -lt exp2       # 小于
exp1 -le exp2       # 小于等于
! exp               # 取反

# 文件测试
-d file             # file 是否存在且为目录
-e file             # file 是否存在（-e 不可移植，通常使用 -f）
-f file             # file 是否存在且为普通文件
-s file             # file 是否存在且为非空
-r file             # file 是否存在且可读
-w file             # file 是否存在且可写
-x file             # file 是否存在且可执行
-O file             # file 是否存在且属当前用户所有
-G file             # file 是否存在且默认组与当前用户相同
-g file             # file 是否存在且 set-group-id 位置位
-u file             # file 是否存在且 set-user-id 位置位
file1 -nt file2     # file1 是否比 file2 新
file1 -ot file2     # file1 是否比 file2 旧
```

#### 双括号命令（Bash 提供）

双括号命令 (( expression )) 相较于 test 允许使用高级数学表达式。

双括号表达式中的大于号和小于号无需转义。

```bash
val++               # 后增
val--               # 后减
++val               # 先增
--val               # 先减
!                   # 逻辑取反
~                   # 位取反
**                  # 幂运算
<<                  # 左移位
>>                  # 右移位
&                   # 按位与
|                   # 按位或
&&                  # 逻辑与
||                  # 逻辑或
```

#### 双方括号（Bash 提供）

双方括号 `[[ expression ]]` 相较于 `test` 提供了针对字符串比较的高级特性（模式匹配）。

`==` 使用右侧的正则表达式对左侧进行匹配。

```bash
if [[ $USER == r* ]]; then
    echo "Hello $USER"
fi

# expression 可以为 test 命令采用的标准字符串比较
if [[ $str1 = $str2 ]]; then
    echo "$str1 equal $str2"
fi
```

#### 命令列表 AND OR

命令退出码为 0 表示成功。

`&&` 与 `||` 连接的命令执行时为短路径求值 (short circuit evaluation)。

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

#### 语句块

当要在只允许使用单个语句的地方（如 AND 或 OR 列表中）使用多条语句，可以使用 `{}` 构造一个语句块。

```bash
[ -f foo ] && {
    echo 'hello'
    echo 'world'
}
```

#### if 语句

注意： condition 通常采用 test 命令，condition 命令的退出码决定如何执行条件代码。

condition 可以为复合条件测试：`[ cond1 ] && [ cond2 ]` 和 `[ cond1 ] || [ cond2 ]`。

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

#### case 语句

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

#### for 语句

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

# for 默认使用空格划分列表的值，当在直接在 for 中写列表时需要注意空格和引号
# 对于保存在变量中的列表，一般无需关心对引号转义
for test in I don't know if this'll work "hello world"; do
    echo "word:$test"
done
# Output: 
# I
# dont know if thisll
# work
# hello world
```

环境变量 `IFS` 被称为内部字段分隔符 (internal field separator)，其定义了 Bash 用作字段分隔符的一系列字符。
默认情况下 Bash 的字段分隔符为：空格、制表符、换行符。

```bash
IFS=$'\n'       # 修改字段分隔符
for f in $(ls -l); do
    echo $f
done

# for 命令可以使用通配符遍历目录
for file in /path/*; do
    echo $file
done
```

> `$'` (Dollar-sign quote) 会将 `$'string'` 中的转义字符进行标准替换，但不会对其中的变量进行展开。
> `$"` (Dollar-sign double-quote) 用于本地化。

#### C 语言风格 for 命令

`for (( variable assignment; condition; iteration process ))`

需要注意以下几点：
* 变量赋值可以有空格
* 条件中的变量不以 `$` 开头
* 迭代过程的算式为使用 expr 命令格式

```bash
for (( i = 1; i <= 10; i++ )); do
    echo $i
done

# 使用多个变量
for (( a=1, b=10; a <= 10; a++, b-- )); do
    echo $a, $b
done
```

#### while 语句

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

# while 命令可以使用多个测试命令，每次迭代时都会执行所有测试命令，
# 并以最后一个测试命令的退出状态码做为判断依据。
# 每个测试命令出现在单独的一行上。
while echo $var
    [ $var -ge 0 ]
do
    ...
done
```

#### until 语句

与 while 类似，循环反复执行，直到条件为真。

同样的 until 命令也支持多个测试命令。

```bash
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

> 如果要对循环的输出使用管道或者重定向，可以在 `done` 命令之后添加处理命令。比如 `done > out.txt` 或 `done | sort`。

### 函数

函数在调用之前必须先定义。

当一个函数被调用时，位置参数与相关环境变量（`$*`/`$@`/`$#`/`$1`/`$2` 等）会被替换为函数的参数。当函数执行完毕后，这些参数会恢复之前的值。

函数可以通过 `return` 命令返回一个数字值，返回字符串的方式可以使用 `echo` 命令，或将返回值存在一个变量中。

如果函数没有使用 `return` 指定返回值，则默认返回函数中最后一条命令的退出码。

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

## 命令

Shell 脚本内使用的命令分为两种

* 外部命令：独立存在于 Shell 之外的命令
* 内部命令：Shell 内置的，不能作为外部程序被调用。不过按照 POSIX 标准，大多数内部命令也同时提供了独立运行的程序。

使用 `$(command)` 捕获一条命令的执行结果，是命令的输出，而非退出码。

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

### expr 命令

将其参数当作一个表达式来求值。最常见的用法就是进行简单数学运算。

在较新的脚本程序中，`expr` 通常被替换为更有效的 `$((...))` 语法。

```bash
# `` 和 $() 为命令替换
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

### 算术扩展

`expr` 命令可以处理一些简单的算数命令，但执行起来比较慢，一般建议采用 `$((...))` 扩展。

`$[...]` 与 `$((...))` 相同，前者已经被弃用，尽可能使用后者。

```bash
x=0
while [ "$x" -ne 10 ]; do
    echo $x
    x=$(($x+1))     # 累加
done
```

### 参数扩展

```bash
for i in 1 2; do
    some_process $i_tmp     # shell 试图替换变量 $i_tmp 的值
    some_process ${i}_tmp   # shell 会使用变量 i 的值替换 ${i}
done

# 常见参数扩展方法
${param:-default}       # 如果 param 为空，则为 default
${param:=default}       # 如果 param 为空，将 param 赋值为 default
${param:?msg}           # 如果 param 为空，则显示 msg 并退出脚本
${param:+default}       # 如果 param 不为空，则为 default
${#param}               # 给出 param 的长度
${param%word}           # 从 param 的尾部删除 word 的最短匹配，然后返回剩余部分
${param%%word}          # 从 param 的尾部删除 word 的最长匹配，然后返回剩余部分
${param#word}           # 从 param 的头部删除 word 的最短匹配，然后返回剩余部分
${param##word}          # 从 param 的头部删除 word 的最长匹配，然后返回剩余部分

# 如果需要在 ${} 内使用变量（展开），需要将 $ 换成 !
hello=world
world=ok
echo ${!hello}
# Output: ok
```

### 脚本调试

设置 shell 选项的方式可以在调用 shell 时加上命令行选项，或使用 `set` 命令。

| 命令行选项 | set 命令 | 说明 |
| --- | --- | --- |
| `sh -n <script>` | `set -o noexec`<br>`set -n` | 只检查语法错误，不执行命令 |
| `sh -v <script>` | `set -o verbose`<br>`set -v` | 在执行命令之前回显 |
| `sh -x <script>` | `set -o xtrace`<br>`set -x` | 在执行命令之后回显 |
| `sh -u <script>` | `set -o nounset`<br>`set -u` | 如果使用了未定义的变量，就给出出错消息 |

`set -o xtrace` 表示启用，`set +o xtrace` 表示取消设置。

`set -- args` 将当前命令行参数替换为 args。

### getopt

命令格式 `getopt optstring parameters`

optstring 定义命令行中有效的选项字母，如果选项有参数，则在其后加个冒号，比如 `ab:c`。

getopt 命令不擅长处理带空格和引号的参数值，它会将空格作为参数分隔符且无视引号。

```bash
getopt ab:cd -a -b test1 -cd test2 test3
# Output: -a -b test1 -c -d -- test2 test3
# -q 选项可以抑制 getopt 输出错误信息
```

使用示例：

```bash
set -- $(getopt -q ab:cd "$@")

while [ -n "$1" ]; do
    case "$1" in
        -a) echo "Found the -a option" ;;
        -b) param="$2"
            echo "Found the -b option, with parameter value $param"
            shift ;;
        -c) echo "Found the -c option" ;;
        --) shift
            break ;;
        *) echo "$1 is not an option";;
    esac
    shift
done

count=1
for param in "$@"; do
    echo "Parameter #$count: $param"
    count=$(($count+1))
done
```

### getopts

Bash 内建命令，一次只处理命令行上检测到的一个参数，当所有参数处理完毕后退出，且返回一个大于0的退出状态码。

命令格式 `getopts optstring variable`

* `optstring`: 选项字母序列，对于有参数的选项，在其后加个冒号。如需取消错误消息，则在 optstring 之前加个冒号。
* `variable`: 当前参数名。
* `$OPTARG`: 当前参数值。
* `$OPTIND`: 当前参数在参数列表中的位置。

使用示例：

```bash
while getopts :ab:cd opt; do
    case "$opt" in
    a) echo "Found the -a option"  ;;
    b) echo "Found the -b option, with value $OPTARG" ;;
    c) echo "Found the -c option"  ;;
    d) echo "Found the -d option"  ;;
    *) echo "Unknown option: $opt" ;;
    esac
done

shift $(($OPTIND - 1))

count=1
for param in "$@"; do
    echo "Parameter $count: $param"
    count=$(($count+1))
done
```
