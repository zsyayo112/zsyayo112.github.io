---
title: COMP2041 Week 03,4 — Shell Scripting 完整笔记
date: 2026-03-18
tags:
  - COMP2041
  - Shell
  - Linux
  - UNSW
categories:
  - COMP9044
---

> 本笔记整理自 COMP2041/9044 26T1 Week 04 课件、Tutorial 题目、Lab Exercise，以及实际动手练习过程中踩到的所有坑。内容包括知识点讲解、命令示例、常见错误与修正、练习题与答案。

---

## 一、Shell 执行流水线

当你在终端输入一条命令，Shell 并不是直接执行它，而是先经过 **8 个变换步骤**：

| 步骤 | 名称               | 示例                             |
| ---- | ------------------ | -------------------------------- |
| 1    | Tilde expansion    | `~/labs` → `/home/z1234567/labs` |
| 2    | 变量展开           | `$HOME` → `/home/z1234567`       |
| 3    | 算术展开           | `$((6 * 7))` → `42`              |
| 4    | 命令替换           | `$(whoami)` → `z1234567`         |
| 5    | **Word splitting** | `" xx yyy "` → 三个独立的词      |
| 6    | **Glob 展开**      | `*.c` → `main.c hello.c`         |
| 7    | I/O 重定向         | `< input.txt`                    |
| 8    | 执行命令           | 第一个词是程序名，其余是参数     |

> **重点**：步骤 5 和步骤 6 是最多 bug 的来源。理解这两步是掌握引号用法的关键。

---

## 二、变量

### 2.1 赋值与引用

```sh
# 赋值：等号两边绝对不能有空格！
x=hello          # ✅ 正确
x = hello        # ❌ 错误：shell 把 x 当成命令名

# 引用
echo $x          # 基本引用
echo ${x}        # 带花括号，明确边界
echo ${x}world   # 输出 helloworld，不是 $xworld
```

### 2.2 常见错误：赋值空格

```sh
# 实际踩坑记录
z5523839@vx24$ x = 8
bash: x: command not found    # shell 把 x 当命令！

z5523839@vx24$ now = $(date)
bash: now: command not found  # 同样的问题

# 正确写法
x=8
now=$(date)
```

### 2.3 命令替换 `$()`

```sh
# $(command) 执行命令，捕获 stdout，去掉末尾换行
now=$(date)
echo "现在是: $now"

user=$(whoami)
echo "你好, $user"

# 注意：$date 是读取变量 date，不是执行命令 date！
echo $date    # 空的（变量未定义）
echo $(date)  # 执行命令，输出时间
```

### 2.4 算术展开 `$(())`

```sh
x=8
echo $((x*x - 3*x + 2))   # 输出 42
n=5
n=$((n + 1))               # n 变成 6
echo $((3 > 2))            # 输出 1（真）
echo $((3 < 2))            # 输出 0（假）
```

> `$(())` 里面的变量不需要加 `$`，直接写变量名就行。

### 2.5 变量展开扩展语法

```sh
# ${var:-default}：var 有值用 var，没有用 default（不赋值）
unset name
echo ${name:-stranger}    # 输出 stranger
name=Alice
echo ${name:-stranger}    # 输出 Alice

# ${var:=default}：var 没有值时赋值并使用
unset count
echo ${count:=0}          # 输出 0，count 被赋值为 0
echo $count               # 输出 0

# ${var:?message}：var 没有值时报错退出
unset required
echo ${required:?'ERROR: required not set'}
# 输出：bash: required: ERROR: required not set

# 实用场景：脚本参数默认值
file=${1:-output.txt}
dir=${2:-/tmp}
echo "将写入: $dir/$file"
```

---

## 三、引号

### 3.1 三种引号的区别

|                | 无引号 | 双引号 `""` | 单引号 `''` |
| -------------- | ------ | ----------- | ----------- |
| 变量展开       | ✅      | ✅           | ❌           |
| 命令替换       | ✅      | ✅           | ❌           |
| 算术展开       | ✅      | ✅           | ❌           |
| Glob 展开      | ✅      | ❌           | ❌           |
| Word splitting | ✅      | ❌           | ❌           |

```sh
answer=42
echo "The answer is $answer."   # → The answer is 42.
echo 'The answer is $answer.'   # → The answer is $answer.
echo The answer is $answer.     # → The answer is 42.

# 单引号内什么都不展开
echo 'Hello $(date) $HOME *.txt'
# → Hello $(date) $HOME *.txt
```

### 3.2 Word Splitting 演示

```sh
w=" xx yyy zzzz "
echo $w     # → xx yyy zzzz（前后空格丢失，中间压缩）
echo "$w"   # →  xx yyy zzzz （完整保留）

# 参数个数的区别（实际踩坑）
x="hello world"
/tmp/showargs.sh $x     # 收到 2 个参数：hello 和 world
/tmp/showargs.sh "$x"   # 收到 1 个参数：hello world
```

### 3.3 文件名含空格的陷阱

```sh
touch "my file.txt"

# 危险：$(ls) 经过 word splitting，文件名断开
for f in $(ls); do echo "文件: $f"; done
# 输出：
# 文件: my
# 文件: file.txt    ← 断开了！

# 正确：用 glob，不经过 word splitting
for f in *; do echo "文件: $f"; done
# 输出：
# 文件: my file.txt  ← 正确！
```

### 3.4 Tilde 展开的陷阱

```sh
echo ~/labs      # → /home/z1234567/labs  ✅ 展开了
echo "~/labs"    # → ~/labs               ❌ 双引号内 ~ 不展开！

# Shruti 的符号链接 bug 就是这个原因
ln -s "~/friends/alice.jpg" .    # ❌ ~ 不展开，链接坏掉
ln -s ~/friends/alice.jpg .      # ✅ ~ 在引号外，正确展开
ln -s "${HOME}/friends/alice.jpg" .  # ✅ $HOME 在双引号内也能用
```

### 3.5 黄金法则

> **变量几乎永远要加双引号：`"$var"`**
> 除非你明确需要 word splitting 或 glob。

### 3.6 单引号陷阱

```sh
echo 'it's broken'
# shell 看到：'it' 关闭 + s broken' 又开了一个没关闭的单引号
# 会一直等待输入，用 Ctrl+C 退出

# 单引号内不能出现单引号
# 双引号内可以有单引号
echo "it's fine"    # → it's fine
```

---

## 四、括号总结

这是最容易混淆的地方，统一整理：

```sh
$var      → 读取变量值
${var}    → 读取变量值（花括号明确边界）
$(cmd)    → 执行命令，捕获输出
$((expr)) → 计算算术表达式

''        → 原样字符串，什么都不展开
""        → 展开变量和命令，但不 glob 不 word split
```

```sh
# 具体例子
x=hello
echo $x           # hello
echo ${x}world    # helloworld
echo $(date)      # 当前时间
echo $((3+5))     # 8
echo '$x'         # $x（原样）
echo "$x"         # hello（展开变量）
```

---

## 五、内置变量

```sh
$0    # 脚本自身名字：./myscript.sh
$1    # 第一个参数
$2    # 第二个参数
$#    # 参数个数（不含 $0）
"$@"  # 所有参数，每个保持独立（保护空格）
$?    # 上一条命令的退出状态（0=成功，非0=失败）
$$    # 当前 shell 的进程号（PID）
```

### 5.1 `$?` 退出状态

```sh
# 退出状态约定：0=成功，非0=失败
./script.sh Alice
echo $?    # 0（成功）

./script.sh
echo $?    # 1（失败，触发了 exit 1）

# 注意：$? 要配合 echo 用！
$?    # ❌ shell 把数字当命令执行，报 "1: command not found"
echo $?  # ✅ 正确用法
```

### 5.2 `"$@"` vs `$@`

```sh
# "$@" 把每个参数作为独立的词，保护空格
./script.sh "hello world" foo
# $1 = hello world（含空格）
# $2 = foo

# for 遍历参数必须用 "$@"
for a in "$@"; do echo "[$a]"; done
# 输出：
# [hello world]
# [foo]

# 不加引号就会被 word split
for a in $@; do echo "[$a]"; done
# 输出：
# [hello]    ← hello world 被拆开了
# [world]
# [foo]
```

---

## 六、source 与 ./ 的区别

### 6.1 核心区别

```
./script.sh  → 开子进程执行，子进程的变量/目录改变不影响父 shell
source script.sh  → 在当前 shell 里执行，变量/目录改变都生效
```

### 6.2 实际演示

```sh
# 脚本内容
cd /tmp
ex1=jpg2png
echo "脚本内部: $(pwd), ex1=$ex1"

# 用 ./ 执行
pwd                  # /home/z1234567/labs
./script.sh          # 脚本内部: /tmp, ex1=jpg2png
pwd                  # /home/z1234567/labs  ← 没变！
echo $ex1            # （空）← 没有！

# 用 source 执行
source ./script.sh   # 脚本内部: /tmp, ex1=jpg2png
pwd                  # /tmp  ← 变了！
echo $ex1            # jpg2png  ← 有了！
```

### 6.3 Tutorial Q1 解答

`start_lab04.sh` 里的 `cd` 和变量赋值不起作用，是因为脚本在子进程中运行。修复方法：

```sh
source ./start_lab04.sh
# 或者
. ./start_lab04.sh
```

---

## 七、test 命令

`test` 命令执行条件判断，成功返回 0，失败返回非 0。`[ ]` 是 `test` 的等价写法。

### 7.1 数字比较

```sh
test "$x" -eq 5    # 等于
test "$x" -ne 5    # 不等于
test "$x" -lt 5    # 小于
test "$x" -le 5    # 小于等于
test "$x" -gt 5    # 大于
test "$x" -ge 5    # 大于等于

# 注意：不能用 > < >= <= 做数字比较！
# > 在 shell 里是重定向符号
[ "$x" > 5 ]    # ❌ 这是重定向，不是比较！
[ "$x" -gt 5 ]  # ✅ 正确
```

### 7.2 字符串比较

```sh
test "$s" = "hello"    # 相等（注意是 = 不是 ==）
test "$s" != "hello"   # 不相等
```

### 7.3 文件判断

```sh
test -f file    # 是普通文件
test -d dir     # 是目录
test -e path    # 存在（文件或目录都行）
test -r file    # 可读
test -x file    # 可执行
test -s file    # 存在且非空
test -z "$var"  # 字符串为空
```

### 7.4 逻辑组合

```sh
# -a 是 AND，-o 是 OR，! 是 NOT
test "$x" -ge 10 -a "$x" -le 20    # 10 <= x <= 20

# 更安全的写法：用两个独立的 [ ] 配合 && ||
[ "$x" -ge 10 ] && [ "$x" -le 20 ]
[ "$x" -lt 0 ] || [ "$x" -gt 100 ]
```

> **为什么推荐用两个 `[]` 而不是 `-a -o`？**
> 当变量为空时，`test "$n" -gt "$max"` 里 `$max` 是空字符串，
> `-a` 不会真正短路，会解析整行，导致 `Illegal number` 错误。
> 用 `||` 连接两个独立的 `[]` 才是真正的短路。

### 7.5 [] 的注意事项

```sh
[ -e "$name" ]   # ✅ 两边有空格
[-e "$name"]     # ❌ 语法错误，[ 后面必须有空格
[ -e $name ]     # ⚠️  变量没加引号，文件名含空格会出错
```

---

## 八、if / elif / else

```sh
if test "$#" -eq 0
then
    echo "用法: $0 <名字>" 1>&2
    exit 1
elif test "$#" -eq 1
then
    echo "你好, $1"
else
    echo "参数太多了"
fi

# 也可以用 [] 写法
if [ "$#" -eq 0 ]
then
    ...
fi
```

### 8.1 && 和 || 短路执行

```sh
# && ：前面成功才执行后面
./script.sh Alice && echo "脚本成功了"
test -d tmp || mkdir tmp    # 目录不存在就创建

# 组合使用
test -d tmp || mkdir tmp && chmod 755 tmp
mkdir -p tmp && chmod 755 tmp    # 更简洁
```

---

## 九、while 循环

```sh
# 语法
while 命令
do
    循环体
done

# 例子：计数
n=1
while test "$n" -le 5
do
    echo "$n"
    n=$((n + 1))
done

# 用 [] 写法
while [ "$n" -le 5 ]
do
    echo "$n"
    n=$((n + 1))
done
```

> **无限循环陷阱**：如果忘了写 `n=$((n + 1))`，循环永远不结束。用 `Ctrl+C` 退出。

---

## 十、for 循环

```sh
# 遍历词列表
for n in 1 2 3 4 5
do
    echo "$n"
done

# 遍历参数
for a in "$@"
do
    echo "参数: $a"
done

# 遍历文件（用 glob，不用 $(ls)）
for f in *.txt
do
    echo "文件: $f"
done

# for 不像 while 那样基于退出状态，它直接遍历列表
```

### 10.1 break 和 continue

```sh
for n in 1 2 3 4 5
do
    if [ "$n" -eq 3 ]; then continue; fi  # 跳过 3
    if [ "$n" -eq 5 ]; then break; fi     # 到 5 停止
    echo "$n"
done
# 输出：1 2 4
```

---

## 十一、case 语句

```sh
case "$变量" in
模式1)
    命令
    ;;
模式2)
    命令
    ;;
*)
    默认命令
    ;;
esac
```

```sh
# 实际例子：文件类型分类
case "$file" in
*.c)   echo "C 源文件" ;;
*.h)   echo "C 头文件" ;;
*.sh)  echo "Shell 脚本" ;;
*.txt) echo "文本文件" ;;
*)     echo "未知类型" ;;
esac

# 参数个数检查
case "$#" in
0) echo "没有参数" ;;
1) filename=$1 ;;
*) echo "参数太多" ;;
esac
```

### 11.1 case 的模式 vs 正则表达式

case 使用的是 **glob**，不是正则表达式，二者的 `*` 含义完全不同：

|          | Glob               | 正则                               |
| -------- | ------------------ | ---------------------------------- |
| `*`      | 匹配任意个任意字符 | 前面那个字符重复0次或多次          |
| `*.c`    | 以 `.c` 结尾       | 任意字符后跟 `c`（`.` 是任意字符） |
| 正确写法 | `*.c`              | `.*\.c$`                           |
| 特殊字符 | `*` `?` `[]`       | `.*+?^$[]{}()\\` 等                |

---

## 十二、Shell 函数

```sh
# 语法
函数名() {
    命令
}

# 调用
函数名 参数1 参数2
```

```sh
# 例子
favourite_command() {
    local name command    # 用 local 声明局部变量！
    name=$1
    command=$2
    echo "我是 $name，我最喜欢的命令是 $command"
}

favourite_command Alice ls
favourite_command Bob grep
```

### 12.1 local 变量——最重要的概念

**不用 `local` 的后果：**

```sh
# ❌ 没有 local，函数内的 i 是全局变量
is_prime() {
    n=$1
    i=2    # 全局！会破坏外层循环的 i
    while test $i -lt $n; do
        test $((n % i)) -eq 0 && return 1
        i=$((i + 1))
    done
    return 0
}

i=2
while test $i -lt 10
do
    is_prime $i && echo "$i is prime"
    i=$((i + 1))    # i 被函数改掉了，循环混乱！
done
# 实际输出：3 is prime（无限循环）→ 需要 Ctrl+C
```

**用 `local` 修复：**

```sh
# ✅ 有 local，函数内的变量不影响外层
is_prime() {
    local n i    # 关键！
    n=$1
    i=2
    while test $i -lt $n; do
        test $((n % i)) -eq 0 && return 1
        i=$((i + 1))
    done
    return 0
}

i=2
while test $i -lt 10
do
    is_prime $i && echo "$i is prime"
    i=$((i + 1))    # i 不受函数影响，正常递增
done
# 正确输出：2 3 5 7
```

### 12.2 return vs exit

```sh
return 0    # 函数正常返回，退出状态 0
return 1    # 函数返回，退出状态 1
exit 0      # 终止整个脚本！不只是函数
exit 1      # 终止整个脚本！
```

> **注意**：在函数内使用 `exit` 会终止整个脚本，不是只退出函数！用 `return` 代替。

---

## 十三、I/O 重定向

| 语法             | 作用                         |
| ---------------- | ---------------------------- |
| `< infile`       | stdin 从文件读取             |
| `> outfile`      | stdout 写入文件（覆盖）      |
| `>> outfile`     | stdout 追加到文件            |
| `2> outfile`     | stderr 写入文件              |
| `2>> outfile`    | stderr 追加到文件            |
| `> outfile 2>&1` | stdout 和 stderr 都写入文件  |
| `1>&2`           | stdout 重定向到 stderr       |
| `/dev/null`      | 垃圾桶，写进去的内容直接消失 |

```sh
# 常见用法
echo "错误信息" 1>&2             # 错误信息走 stderr
./script.sh 2>/dev/null         # 只看正常输出，丢掉错误
./script.sh 1>/dev/null         # 只看错误，丢掉正常输出
./script.sh > output.txt        # 正常输出存文件
cat 不存在的文件 2> /tmp/err.txt # 把错误信息存文件
```

### 13.1 stdout vs stderr 的意义

```sh
#!/bin/dash
if test "$#" -eq 0
then
    echo "用法: $0 <文件名>" 1>&2    # 错误提示 → stderr
    exit 1
fi
cat "$1"    # 正常结果 → stdout
```

- 程序的**正常结果**走 stdout
- 程序的**错误提示、警告**走 stderr
- 调用者可以用 `2>/dev/null` 只看正常输出，`1>/dev/null` 只看错误

### 13.2 `wc -l` 的文件名问题

```sh
wc -l notes.txt         # 输出：  10 notes.txt  （带文件名）
wc -l < notes.txt       # 输出：  10             （只有数字）
lines=$(wc -l < "$f")   # 正确获取行数的方式
```

---

## 十四、管道与子Shell陷阱

### 14.1 管道基础

```sh
# 管道把左边的 stdout 连接到右边的 stdin
cat file.txt | grep "hello" | sort | uniq -c
```

### 14.2 管道子Shell陷阱——最重要的陷阱

```sh
# ❌ 有 bug 的版本
total=0
cat numbers.txt | while read n
do
    total=$((total + n))    # 这里修改了 total
done
echo "总和: $total"    # 输出 0！

# 原因：管道右边的 while 在子Shell里运行
# 子Shell里对 total 的修改对父Shell不可见
```

```sh
# ✅ 修复方法1：用重定向代替管道
total=0
while read -r n
do
    total=$((total + n))
done < numbers.txt
echo "总和: $total"    # 正确输出

# ✅ 修复方法2：用 awk
awk '{sum += $1} END {print "总和:", sum}' numbers.txt

# ✅ 修复方法3：用 here document
total=0
while read -r n
do
    total=$((total + n))
done << EOF
3
5
7
EOF
echo "总和: $total"
```

### 14.3 read 命令

```sh
# read 从 stdin 读一行，存进变量
read v
hello world
echo "$v"    # → hello world

# 读多个变量
read a b c
1 2 3 4 5
echo "a=$a b=$b c=$c"    # → a=1 b=2 c=3 4 5（最后一个变量接收剩余内容）

# 在脚本中逐行读取文件
while IFS= read -r line
do
    echo "$line"
done < file.txt

# -r 选项：不处理反斜杠转义（推荐默认加上）
# IFS= ：不去除行首行尾空格
```

---

## 十五、here document

```sh
# << EOF ... EOF 把多行字符串作为命令输入
cat << EOF
Hello $name
How are you
Good bye
EOF

# 变量会展开（类似双引号）
name=Alice
tr a-z A-Z << EOF
hello $name
EOF
# → HELLO ALICE

# 不展开变量（类似单引号）：给 EOF 加引号
cat << 'EOF'
$PATH is not expanded here
EOF
# → $PATH is not expanded here
```

---

## 十六、trap 与 mktemp

### 16.1 问题背景

脚本如果中途被 `Ctrl+C` 打断，创建的临时文件会留在磁盘上，造成垃圾。

### 16.2 信号类型

| 信号   | 触发方式          | 含义                 |
| ------ | ----------------- | -------------------- |
| `INT`  | Ctrl+C            | 用户中断             |
| `TERM` | `kill PID`        | 系统终止请求         |
| `EXIT` | 脚本正常/异常结束 | 退出时执行（最重要） |
| `HUP`  | 终端关闭          | 挂断                 |

### 16.3 标准模板

```sh
#!/bin/dash

# 创建临时文件
TMP=$(mktemp)
echo "临时文件: $TMP"

# 设置 trap：无论怎么退出都删除临时文件
trap 'rm -f "$TMP"; exit' INT TERM EXIT

# 做工作...
echo "一些数据" > "$TMP"
cat "$TMP"
sleep 5
echo "工作完成"

# 脚本结束时 EXIT 信号触发，$TMP 自动删除
```

```sh
# 临时目录版本
TMP_DIR=$(mktemp -d)
trap 'rm -rf "$TMP_DIR"; exit' INT TERM EXIT
cd "$TMP_DIR" || exit 1
# 在临时目录里做工作，退出时整个目录删除
```

---

## 十七、{} vs () 子Shell

```sh
x=123

# () 子Shell：变量和目录改变不影响外部
( x=abc; cd /tmp )
echo "() 之后: x=$x, pwd=$(pwd)"
# → () 之后: x=123, pwd=/home/z1234567（没变）

# {} 当前Shell：变量和目录改变立刻生效
{ x=abc; cd /tmp; }    # 注意：最后一条命令后面必须有分号，} 前面有空格
echo "{} 之后: x=$x, pwd=$(pwd)"
# → {} 之后: x=abc, pwd=/tmp（都变了）
```

|          | `()`          | `{}`               |
| -------- | ------------- | ------------------ |
| 执行环境 | 子Shell       | 当前Shell          |
| 变量改变 | 不传出        | 生效               |
| 目录改变 | 不传出        | 生效               |
| 类似于   | `./script.sh` | `source script.sh` |

> **`{}` 的语法要求**：
> - `{` 后面必须有空格
> - 最后一条命令后面必须有 `;`
> - `}` 前面必须有空格

---

## 十八、PATH 搜索机制

```sh
echo $PATH
# → /usr/local/bin:/usr/bin:/bin:/home/z1234567/bin

# 输入 cat 时，shell 依次检查：
# /usr/local/bin/cat
# /usr/bin/cat  ← 找到了！执行它
# /bin/cat

# 所以必须用 ./script.sh 而不是 script.sh
# 因为当前目录 . 不在 PATH 里（默认）
```

### 18.1 安全警告

```sh
# 危险：. 在 PATH 开头
PATH=.:/usr/bin:/bin    # 当前目录优先，安全风险！

# 攻击场景：在当前目录放一个叫 ls 的恶意脚本
# 用户输入 ls 时运行的是恶意脚本而不是 /bin/ls

# 脚本里固定 PATH（推荐）
PATH=/usr/local/bin:/usr/bin:/bin
```

### 18.2 Tutorial Q12 — which.sh 实现

```sh
#!/bin/dash
# 在 PATH 里找程序，模拟系统的 which 命令

program=$1

for dir in $(echo $PATH | tr ':' ' ')
do
    if test -f "${dir}/${program}" -a -x "${dir}/${program}"
    then
        ls -l "${dir}/${program}"
        exit 0
    fi
done

echo "$program not found"
exit 1
```

---

## 十九、shellcheck

```sh
# 静态分析工具，不运行脚本就能发现 bug
shellcheck your_script.sh

# 强烈推荐！能发现：
# - 变量没加引号
# - [ ] 用法错误
# - 常见 bash/dash 兼容问题
# 网页版：https://www.shellcheck.net/
```

---

## 二十、Tutorial 题目完整解答

### Q1：start_lab04.sh 不起作用

**问题**：脚本里 `cd ~/labs/04` 和变量赋值，但运行后父Shell的 `pwd` 没变，`$ex1` 是空的。

**原因**：`./start_lab04.sh` 在子进程里运行，子进程的改变不影响父Shell。

**修复**：
```sh
source ./start_lab04.sh
# 或
. ./start_lab04.sh
```

---

### Q2：is_business_hours

```sh
#!/bin/dash
# 提取小时数（date 输出：Fri 13 Mar 2026 15:29:25 AEDT，第5字段是时间）
hour=$(date | cut -d' ' -f5 | cut -d':' -f1)

# 可选：周末判断
week=$(date | cut -d' ' -f1)
if test "$week" = "Sat" -o "$week" = "Sun"
then
    exit 1
fi

if test "$hour" -ge 9 -a "$hour" -lt 17
then
    exit 0
else
    exit 1
fi
```

**知识点**：
- `cut -d' ' -f5`：以空格为分隔符，取第5字段
- `cut -d':' -f1`：以冒号为分隔符，取第1字段（小时）
- 退出状态 0=工作时间，1=非工作时间

---

### Q3：Shruti 的符号链接

**问题代码**：
```sh
for image_file in $(ls ~/friends); do
    ln -s "~/friends/$image_file" .
done
```

**两个 bug**：
1. `"~/friends/..."` 双引号内 `~` 不展开，链接指向字面字符串
2. `$(ls ~/friends)` 不安全，文件名含空格会断开

**修复**：
```sh
# 方法1：用 $HOME 代替 ~
for image_file in $(ls ~/friends); do
    ln -s "${HOME}/friends/$image_file" .
done

# 方法2：更好，用 glob
for image_file in ~/friends/*
do
    ln -s "$image_file" .
done
```

---

### Q4：update_course_code.sh

```sh
#!/bin/dash
# COMP2041 → COMP2042，COMP9044 → COMP9042

for arg in "$@"
do
    if test -d "$arg"
    then
        # 是目录，递归处理
        find "$arg" -type f | while read -r file
        do
            sed -i 's/COMP2041/COMP2042/g; s/COMP9044/COMP9042/g' "$file"
            echo "Updated: $file"
        done
    elif test -f "$arg"
    then
        sed -i 's/COMP2041/COMP2042/g; s/COMP9044/COMP9042/g' "$arg"
        echo "Updated: $arg"
    else
        echo "$0: '$arg' not found" 1>&2
    fi
done
```

**知识点**：
- `sed -i`：in-place，直接修改文件
- `s/旧/新/g`：全部替换
- `find -type f`：递归找普通文件

---

### Q6：is_prime.sh + primes.sh

**is_prime.sh**：
```sh
#!/bin/dash
n=$1

if test "$n" -lt 2
then
    echo "$n is not prime"
    exit 1
fi

i=2
while test "$i" -lt "$n"
do
    if test $((n % i)) -eq 0
    then
        echo "$n is not prime"
        exit 1
    fi
    i=$((i + 1))
done

echo "$n is prime"
exit 0
```

**primes.sh**：
```sh
#!/bin/dash
limit=$1
n=2

while test "$n" -lt "$limit"
do
    if ./is_prime.sh "$n" > /dev/null
    then
        echo "$n"
    fi
    n=$((n + 1))
done
```

**关键点**：`> /dev/null` 丢掉 is_prime.sh 的文字输出，只用退出状态判断。

---

### Q8：recompile.sh（stat 版）

```sh
#!/bin/dash
src=$1
last_mtime=""

while true
do
    mtime=$(stat -c '%Y' "$src")
    if test "$mtime" != "$last_mtime"
    then
        last_mtime=$mtime
        echo "检测到变化，重新编译..."
        if gcc "$src" -o a.out
        then
            ./a.out
        fi
    fi
    sleep 1
done
```

**知识点**：
- `stat -c '%Y' file`：获取文件最后修改时间（Unix 时间戳，秒数）
- 每秒轮询一次，浪费 CPU（inotifywait 版更高效）

---

### Q9：recompile.sh（inotifywait 版）

```sh
#!/bin/dash
src=$1

# 首次编译
if gcc "$src" -o a.out
then
    ./a.out
fi

# 循环等待文件变化
while inotifywait -q -e close_write "$src"
do
    echo "检测到变化，重新编译..."
    if gcc "$src" -o a.out
    then
        ./a.out
    else
        echo "编译失败" 1>&2
    fi
done
```

**知识点**：
- `inotifywait -e close_write`：等待文件被写入并关闭（即保存）
- 操作系统通知触发，不浪费 CPU
- `while inotifywait ...`：每次检测到变化后循环继续等待

---

### Q12：which.sh

```sh
#!/bin/dash
program=$1

for dir in $(echo $PATH | tr ':' ' ')
do
    if test -f "${dir}/${program}" -a -x "${dir}/${program}"
    then
        ls -l "${dir}/${program}"
        exit 0
    fi
done

echo "$program not found"
exit 1
```

---

## 二十一、常见错误汇总

### 错误1：赋值时有空格

```sh
x = 8         # ❌ bash: x: command not found
now = $(date) # ❌ bash: now: command not found
x=8           # ✅
now=$(date)   # ✅
```

### 错误2：变量没加双引号

```sh
name="hello world"
test -f $name    # ❌ 展开成 test -f hello world，三个参数
test -f "$name"  # ✅
```

### 错误3：`~` 在双引号内不展开

```sh
ln -s "~/friends/file" .    # ❌ ~ 不展开
ln -s ~/friends/file .      # ✅
ln -s "${HOME}/friends/file" . # ✅ $HOME 在双引号内可以用
```

### 错误4：`$?` 不加 echo

```sh
$?          # ❌ bash: 0: command not found
echo $?     # ✅
```

### 错误5：管道子Shell陷阱

```sh
total=0
cat file | while read n; do total=$((total+n)); done
echo $total  # ❌ 输出 0

# 修复
while read n; do total=$((total+n)); done < file
echo $total  # ✅ 正确
```

### 错误6：`local` 在函数外使用

```sh
local x=5    # ❌ 只能在函数内用 local
x=5          # ✅ 在脚本主体直接赋值
```

### 错误7：`{}` 语法错误

```sh
{ x=abc; cd/tmp }   # ❌ cd/tmp 没空格，} 前无分号
{ x=abc; cd /tmp; } # ✅ 注意分号和空格
```

### 错误8：`$(ls)` 不安全

```sh
for f in $(ls)  # ❌ 文件名含空格会断开
for f in *      # ✅ glob 安全处理含空格文件名
```

### 错误9：`test` 和 `[]` 混用的逻辑问题

```sh
# -o 不能真正短路，变量为空时报错
test -z "$max" -o "$n" -gt "$max"   # ❌ max 为空时 Illegal number

# 用两个独立的 [] 才能真正短路
[ -z "$max" ] || [ "$n" -gt "$max" ]  # ✅
```

### 错误10：`{$1}` 和 `${1}` 写反

```sh
echo "{$1}"   # ❌ 花括号在 $ 外面，输出字面字符串 {参数值}
echo "${1}"   # ✅ 花括号在 $ 里面包住变量名
echo "$1"     # ✅ 简写，和 ${1} 等价
```

---

## 二十二、边边角角的知识点

### Shebang 行

```sh
#!/bin/dash    # 用 dash（POSIX 标准，本课程要求）
#!/bin/bash    # 用 bash（更多功能但不是 POSIX 标准）
#!/usr/bin/env dash  # 更可移植的写法
```

> 本课程要求用 `#!/bin/dash`，不能用 bash 的扩展功能。

### chmod +x

```sh
chmod +x script.sh    # 给脚本加执行权限
chmod 755 script.sh   # rwxr-xr-x
```

### 调试技巧

```sh
# 方法1：打印变量值
echo "DEBUG: x=$x" 1>&2

# 方法2：set -x，显示每条执行的命令
set -x
# 或者
/bin/dash -x script.sh

# 方法3：shellcheck 静态分析
shellcheck script.sh
```

### /dev/null

```sh
# /dev/null 是垃圾桶，写进去的内容直接消失
./script.sh > /dev/null      # 丢掉 stdout
./script.sh 2> /dev/null     # 丢掉 stderr
./script.sh > /dev/null 2>&1 # 两者都丢掉

# 常见用法：只用退出状态，不要输出
if ./is_prime.sh "$n" > /dev/null 2>&1
then
    echo "$n is prime"
fi
```

### printf vs echo

```sh
echo "hello"           # 简单输出，自动加换行
printf "hello\n"       # 更精确的格式控制
printf "%d items\n" 5  # 格式化输出
printf "%s " 1 2 3     # 输出：1 2 3（不换行）
```

### stat 命令

```sh
stat -c '%Y' file.c    # 获取文件最后修改时间（Unix 时间戳）
# 输出：1773031183（从1970年1月1日到现在的秒数）
```

### find 命令

```sh
find /dir -type f              # 找所有普通文件（递归）
find /dir -type d              # 找所有目录
find /dir -name "*.sh"         # 找所有 .sh 文件
find /dir -maxdepth 1 -type f  # 只找当前目录，不递归
find /dir -type f | while read -r file  # 安全遍历
```

### cut 命令

```sh
# -d 指定分隔符，-f 指定字段号
echo "a:b:c" | cut -d':' -f2    # → b
date | cut -d' ' -f5             # 取第5字段（时间部分）
date | cut -d' ' -f5 | cut -d':' -f1  # 取小时
```

### tr 命令

```sh
echo "HELLO" | tr 'A-Z' 'a-z'   # 转小写
echo "a:b:c" | tr ':' ' '       # 把冒号换成空格
echo "a:b:c" | tr ':' '\n'      # 把冒号换成换行
tr -d ' '                        # 删除所有空格
```

### sed 命令

```sh
sed 's/old/new/g' file          # 替换所有匹配
sed -i 's/old/new/g' file       # 原地修改文件
sed 's/foo/bar/; s/baz/qux/' f  # 多个替换用分号
```

### inotifywait

```sh
inotifywait -q -e close_write file.c   # 等待文件被写入并关闭
# -q：安静模式，减少输出
# -e close_write：只监听写入并关闭事件（即保存操作）
# 文件被修改后退出状态 0，while 循环继续
```

### mktemp

```sh
TMP=$(mktemp)         # 创建临时文件，返回路径
TMP_DIR=$(mktemp -d)  # 创建临时目录
# 配合 trap 使用确保清理
trap 'rm -f "$TMP"; exit' INT TERM EXIT
```

---

## 二十三、练习题与答案

### Part 1：变量与引号

**Exercise 1.1 — greet.sh**

```sh
#!/bin/dash
name=${1:-World}
echo "Hello, $name!"
```

**Exercise 1.2 — 含空格文件名的安全遍历**

```sh
#!/bin/dash
touch "hello world.txt"
for f in *.txt
do
    echo "$f"
done
```

**Exercise 1.3 — info.sh**

```sh
#!/bin/dash
echo "当前用户: $(whoami)"
echo "当前目录: $(pwd)"
echo "当前时间: $(date)"
echo "文件数量: $(ls | wc -l)"
```

**Exercise 1.4 — default.sh**

```sh
#!/bin/dash
file=${1:-output.txt}
dir=${2:-/tmp}
echo "将写入: $dir/$file"
```

---

### Part 2：条件判断

**Exercise 2.1 — check_args.sh**

```sh
#!/bin/dash
if [ "$#" -ne 2 ]
then
    echo "Usage: $0 <num1> <num2>" 1>&2
    exit 1
fi
echo $(($1 + $2))
```

**Exercise 2.2 — file_check.sh**

```sh
#!/bin/dash
if [ "$#" -ne 1 ]
then
    echo "Usage: $0 <file>" 1>&2
    exit 1
fi

f=$1

[ -e "$f" ] && echo "$f: 存在" || { echo "$f: 不存在"; exit 1; }
[ -f "$f" ] && echo "$f: 是普通文件"
[ -d "$f" ] && echo "$f: 是目录"
[ -r "$f" ] && echo "$f: 可读"
[ -x "$f" ] && echo "$f: 可执行"
```

**Exercise 2.3 — grade.sh**

```sh
#!/bin/dash
if [ "$#" -ne 1 ]
then
    echo "Usage: $0 <score>" 1>&2
    exit 1
fi

score=$1

if [ "$score" -lt 0 ] || [ "$score" -gt 100 ]
then
    echo "Error: score must be 0-100" 1>&2
    exit 1
fi

if [ "$score" -ge 85 ]; then echo "HD"
elif [ "$score" -ge 75 ]; then echo "D"
elif [ "$score" -ge 65 ]; then echo "CR"
elif [ "$score" -ge 50 ]; then echo "P"
else echo "F"
fi
```

---

### Part 3：循环

**Exercise 3.1 — countdown.sh**

```sh
#!/bin/dash
if [ "$#" -ne 1 ] || [ "$1" -le 0 ] 2>/dev/null
then
    echo "Usage: $0 <positive integer>" 1>&2
    exit 1
fi

n=$1
while [ "$n" -gt 0 ]
do
    echo "$n"
    n=$((n - 1))
done
echo "Go!"
```

**Exercise 3.2 — biggest.sh**

```sh
#!/bin/dash
if [ "$#" -eq 0 ]
then
    echo "Usage: $0 <num1> <num2> ..." 1>&2
    exit 1
fi

biggest=$1
for n in "$@"
do
    if [ "$n" -gt "$biggest" ]
    then
        biggest=$n
    fi
done
echo "$biggest"
```

**Exercise 3.3 — rename_lower.sh**

```sh
#!/bin/dash
for f in *
do
    newf=$(echo "$f" | tr 'A-Z' 'a-z')
    if test "$f" = "$newf"
    then
        echo "Skipped: $f (unchanged)"
        continue
    fi
    if test -e "$newf"
    then
        echo "Warning: $newf already exists, skipping $f" 1>&2
        continue
    fi
    mv "$f" "$newf"
    echo "Renamed: $f -> $newf"
done
```

**Exercise 3.4 — stats.sh**

```sh
#!/bin/dash
total=0
max=""
min=""

while read -r n
do
    total=$((total + n))
    if [ -z "$max" ] || [ "$n" -gt "$max" ]; then max=$n; fi
    if [ -z "$min" ] || [ "$n" -lt "$min" ]; then min=$n; fi
done

echo "最大值: $max"
echo "最小值: $min"
echo "总和: $total"
```

用法：`./stats.sh < /tmp/nums.txt`

---

### Part 4：函数

**Exercise 4.1 — math.sh**

```sh
#!/bin/dash

add() {
    local a b
    a=$1; b=$2
    echo $((a + b))
}

multiply() {
    local a b
    a=$1; b=$2
    echo $((a * b))
}

factorial() {
    local n result
    n=$1; result=1
    while [ "$n" -gt 1 ]
    do
        result=$((result * n))
        n=$((n - 1))
    done
    echo "$result"
}

echo "3 + 4 = $(add 3 4)"
echo "3 * 4 = $(multiply 3 4)"
echo "5! = $(factorial 5)"
```

**Exercise 4.2 — is_even.sh**

```sh
#!/bin/dash

is_even() {
    local n
    n=$1
    [ $((n % 2)) -eq 0 ]
}

n=1
while [ "$n" -le 20 ]
do
    if is_even "$n"; then printf "%d " "$n"; fi
    n=$((n + 1))
done
echo ""
```

---

### Part 5：I/O 重定向

**Exercise 5.1 — logger.sh**

```sh
#!/bin/dash
if [ "$#" -eq 0 ]
then
    echo "Usage: $0 <message>" 1>&2
    exit 1
fi
echo "[$(date)] $1" >> ~/shell_practice/log.txt
```

**Exercise 5.3 — 管道陷阱修复**

```sh
#!/bin/dash
# bug：管道右边的 while 在子Shell里，total 改变不传出
# 修复：用重定向代替管道

total=0
while read -r mark
do
    total=$((total + mark))
done < marks.txt
echo "Total: $total"
```

---

### Part 6：trap + mktemp

**Exercise 6.1 — cleanup.sh**

题目：创建临时文件，往里写数据，用 `trap` 确保脚本结束时（包括 Ctrl+C）临时文件被删除。

```sh
#!/bin/dash
TMP=$(mktemp)
echo "临时文件: $TMP"
trap 'rm -f "$TMP"; exit' INT TERM EXIT

echo "处理中..."
echo "hello world" > "$TMP"
echo "完成！结果: $(cat $TMP)"

# 验证删除：脚本结束后 ls /tmp/tmp.* 找不到 $TMP
```

**测试方法**：
```sh
chmod +x cleanup.sh
./cleanup.sh           # 正常结束，检查临时文件是否消失
./cleanup.sh           # 运行后立刻 Ctrl+C，检查临时文件是否消失
ls /tmp/tmp.XXXXXXXX   # 应该报 No such file or directory
```

**知识点回顾**：
- `mktemp` 生成随机唯一的临时文件名，避免文件名冲突
- `trap '命令' INT TERM EXIT` 设置信号处理器
- `EXIT` 信号在脚本任何方式退出时都触发（最重要）
- `INT` 对应 Ctrl+C，`TERM` 对应 `kill PID`

---

**Exercise 6.2 — safe_sort.sh**

题目：把若干文件内容合并、排序、去重后输出，用临时文件存中间结果，任何退出都清理。

```sh
#!/bin/dash
if [ "$#" -eq 0 ]
then
    echo "Usage: $0 <file1> <file2> ..." 1>&2
    exit 1
fi

TMP=$(mktemp)
trap 'rm -f "$TMP"; exit' INT TERM EXIT

cat "$@" | sort -u > "$TMP"
cat "$TMP"
```

**测试方法**：
```sh
echo "banana
apple
cherry" > /tmp/fruits1.txt
echo "apple
date
banana" > /tmp/fruits2.txt

chmod +x safe_sort.sh
./safe_sort.sh /tmp/fruits1.txt /tmp/fruits2.txt
# 期望输出：
# apple
# banana
# cherry
# date

./safe_sort.sh              # 测试无参数报错
./safe_sort.sh 不存在.txt   # 测试文件不存在
```

**常见错误**：
```sh
# ❌ 管道子Shell陷阱：sort 结果在子Shell里，写入 $TMP 没问题
# 但如果想在管道后面用变量就会出问题
# 这里直接写文件所以没问题

# ✅ 正确：cat "$@" 把所有文件内容合并输出，| sort -u 排序去重
```

---

### Part 7：综合题

**Exercise 7.1 — word_freq.sh**

题目：统计文件中每个单词出现次数，按频率从高到低输出前10个。

```sh
#!/bin/dash
if [ "$#" -eq 0 ]
then
    echo "Usage: $0 <file1> [file2 ...]" 1>&2
    exit 1
fi

# 验证所有文件存在
for f in "$@"
do
    if [ ! -r "$f" ]
    then
        echo "$0: cannot read '$f'" 1>&2
        exit 1
    fi
done

cat "$@" |
    tr 'A-Z' 'a-z' |           # 转小写
    tr -cs 'a-z' '\n' |        # 非字母字符替换成换行（-c 取补集，-s 压缩连续）
    grep -v '^$' |             # 去掉空行
    sort |                     # 排序，让相同词相邻
    uniq -c |                  # 统计每个词出现次数
    sort -rn |                 # 按数字倒序（频率高的在前）
    head -10                   # 只取前10
```

**测试方法**：
```sh
chmod +x word_freq.sh
./word_freq.sh notes.txt
./word_freq.sh notes.txt hosts.txt    # 多文件合并统计
./word_freq.sh                        # 无参数报错
```

**管道逐步解析**：
```sh
# 假设文件内容：Hello world hello SHELL
echo "Hello world hello SHELL" | tr 'A-Z' 'a-z'
# → hello world hello shell

echo "Hello world hello SHELL" | tr 'A-Z' 'a-z' | tr -cs 'a-z' '\n'
# → hello
#   world
#   hello
#   shell

echo "Hello world hello SHELL" | tr 'A-Z' 'a-z' | tr -cs 'a-z' '\n' | sort | uniq -c
#    2 hello
#    1 shell
#    1 world

# 再 sort -rn | head -10 就得到最终结果
```

**知识点**：
- `tr -cs 'a-z' '\n'`：`-c` 取补集（非a-z的字符），`-s` 压缩连续，全部换成换行
- `uniq -c`：统计连续相同行的次数，必须先 `sort` 才能正确统计
- `sort -rn`：`-r` 倒序，`-n` 数字排序（不加 `-n` 会按字符串排序，10 < 9）

---

**Exercise 7.2 — backup.sh**

题目：把目录里所有 `.txt` 文件备份到 `/tmp/backup_<日期>/`，文件名加日期后缀。

```sh
#!/bin/dash
if [ "$#" -ne 1 ]
then
    echo "Usage: $0 <directory>" 1>&2
    exit 1
fi

if [ ! -d "$1" ]
then
    echo "$0: '$1' is not a directory" 1>&2
    exit 1
fi

dir=$1
date_str=$(date +%Y%m%d)
backup_dir="/tmp/backup_${date_str}"

mkdir -p "$backup_dir"

count=0
for f in "$dir"/*.txt
do
    # glob 没有匹配时 f 是字面字符串 "$dir/*.txt"，用 -f 过滤
    [ -f "$f" ] || continue

    base=$(basename "$f" .txt)
    dest="${backup_dir}/${base}_${date_str}.txt"
    cp "$f" "$dest"
    echo "  $(basename $dest)"
    count=$((count + 1))
done

if [ "$count" -eq 0 ]
then
    echo "没有找到 .txt 文件" 1>&2
    rmdir "$backup_dir"    # 删掉空的备份目录
    exit 1
fi

echo "备份到: $backup_dir/"
echo "备份完成，共 $count 个文件"
```

**测试方法**：
```sh
mkdir /tmp/test_backup
echo "内容1" > /tmp/test_backup/notes.txt
echo "内容2" > /tmp/test_backup/hosts.txt

chmod +x backup.sh
./backup.sh /tmp/test_backup
ls /tmp/backup_$(date +%Y%m%d)/

./backup.sh              # 无参数报错
./backup.sh /不存在      # 目录不存在报错
mkdir /tmp/empty_dir
./backup.sh /tmp/empty_dir  # 没有 .txt 文件报错
```

**知识点**：
- `date +%Y%m%d`：格式化日期，`%Y` 四位年，`%m` 月，`%d` 日
- `basename "$f" .txt`：取文件名去掉路径，第二个参数去掉后缀
- `[ -f "$f" ] || continue`：当 glob 没有匹配时跳过字面字符串

---

**Exercise 7.3 — BOSS 题：monitor.sh**

题目：监控目录，每隔5秒检查一次，新文件出现打印名字和行数，文件被删除打印警告，退出时清理临时文件。

```sh
#!/bin/dash
if [ "$#" -ne 1 ]
then
    echo "Usage: $0 <directory>" 1>&2
    exit 1
fi

if [ ! -d "$1" ]
then
    echo "$0: '$1' is not a directory" 1>&2
    exit 1
fi

dir=$1
TMP_OLD=$(mktemp)
TMP_NEW=$(mktemp)

# trap 处理退出：清理临时文件，打印结束信息
trap 'rm -f "$TMP_OLD" "$TMP_NEW"; echo "监控结束"; exit' INT TERM EXIT

echo "监控目录: $dir"

# 初始化：记录当前文件列表
find "$dir" -maxdepth 1 -type f | sort > "$TMP_OLD"

while true
do
    sleep 5

    # 获取最新文件列表
    find "$dir" -maxdepth 1 -type f | sort > "$TMP_NEW"

    timestamp=$(date +%H:%M:%S)

    # 找新增文件（在新列表里但不在旧列表里）
    # diff 输出：< 是只在旧文件，> 是只在新文件
    diff "$TMP_OLD" "$TMP_NEW" | grep '^>' | cut -c3- |
    while read -r f
    do
        lines=$(wc -l < "$f")
        echo "[$timestamp] 新文件: $(basename "$f") ($lines 行)"
    done

    # 找删除的文件（在旧列表里但不在新列表里）
    diff "$TMP_OLD" "$TMP_NEW" | grep '^<' | cut -c3- |
    while read -r f
    do
        echo "[$timestamp] 文件删除: $(basename "$f")"
    done

    # 更新旧列表
    cp "$TMP_NEW" "$TMP_OLD"
done
```

**测试方法**：
```sh
mkdir /tmp/test_monitor
chmod +x monitor.sh

# 终端1：启动监控
./monitor.sh /tmp/test_monitor

# 终端2：操作文件，观察终端1的输出
echo "hello" > /tmp/test_monitor/a.txt
echo -e "line1\nline2\nline3" > /tmp/test_monitor/b.txt
rm /tmp/test_monitor/a.txt

# 终端1 Ctrl+C，观察"监控结束"是否打印，临时文件是否清理
```

**关键设计决策**：

```sh
# 1. 为什么用 find 而不是 ls？
find "$dir" -maxdepth 1 -type f    # 只找普通文件，不包括目录
ls "$dir"                           # 会包括目录，而且不安全

# 2. 为什么用 sort？
find ... | sort    # 确保文件列表有序，diff 才能正确比较

# 3. diff 输出格式
diff old.txt new.txt
# < 只在旧文件里（被删除的）
# > 只在新文件里（新增的）
# cut -c3- 去掉前两个字符（"> " 或 "< "）

# 4. 管道子Shell陷阱说明
# diff ... | while read -r f; do echo ...; done
# 这里 echo 在子Shell里，没有外部变量要传出，所以没有问题
# 如果需要统计新增文件数量，就必须用重定向代替管道

# 5. trap 里 echo "监控结束"
# EXIT 信号触发时执行，Ctrl+C 也会触发（INT → EXIT 链）
```

**扩展挑战**：如果要统计总共监控到多少次变化，应该怎么修改？（提示：管道子Shell陷阱）

```sh
# 有问题的版本
changes=0
diff "$TMP_OLD" "$TMP_NEW" | grep '^>' | while read -r f
do
    changes=$((changes + 1))    # 子Shell里，changes 传不出来
done
echo "总变化: $changes"    # 永远是 0

# 正确版本：用临时文件记录计数
COUNTER=$(mktemp)
echo 0 > "$COUNTER"
trap 'rm -f "$TMP_OLD" "$TMP_NEW" "$COUNTER"; echo "监控结束"; exit' INT TERM EXIT

# 更新计数
count=$(cat "$COUNTER")
count=$((count + 1))
echo "$count" > "$COUNTER"
```

---

## 二十四、快速参考卡

```sh
# 变量
x=value                  # 赋值（等号两边无空格）
echo "$x"                # 读取（永远加双引号）
echo "${x}suffix"        # 明确边界
echo $((x + 1))          # 算术
echo $(command)          # 命令替换
echo ${x:-default}       # 默认值

# 引号
"$var"    # 展开变量，不 glob，不 word split
'literal' # 原样，什么都不展开

# 内置变量
$0 $1 $2 "$@" $# $? $$

# 条件
[ "$x" -eq 5 ]     # 数字等于
[ "$s" = "str" ]   # 字符串等于
[ -f "$f" ]        # 是文件
[ -d "$d" ]        # 是目录
[ -x "$f" ]        # 可执行
[ -z "$s" ]        # 字符串为空

# 循环
while [ cond ]; do ...; done
for var in list; do ...; done
for var in "$@"; do ...; done
for f in *.txt; do ...; done

# 函数
func() { local x; x=$1; echo "$x"; }

# 重定向
> file   >> file   < file   2> file   1>&2

# 清理
TMP=$(mktemp)
trap 'rm -f "$TMP"; exit' INT TERM EXIT
```

---

*笔记生成于 2026-03-18，基于 COMP2041/9044 26T1 Week 04 课件与 Tutorial 实践。*