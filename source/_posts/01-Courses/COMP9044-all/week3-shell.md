---
title: week3-shell
date: 2026-03-13T23:22:29+11:00
tags: shell
categories: COPM9044
---


**shell**


第一个点 变量的赋值


第二个点 要注意加入引号之后的含义



name=90

echo "my name is $name"

---

w= xx yyy zzz

echo $w

和

echo "$w"

是不一样的



---

![三种引号的区别](quote.png)



---



./test.sh 运行：shell 开一个子进程来执行脚本，子进程里的 cd 和变量赋值只影响子进程，子进程结束后父 shell 完全不知道发生了什么。source ./test.sh 运行：不开新进程，直接在当前 shell 里执行脚本里的每一行，所以 cd 和变量赋值都影响当前 shell。


![source运行和直接运行的区别](source.png)


---

atest 命令和 if 语句。这是写控制逻辑的基础。把 test.sh 改成：

#!/bin/dash
# 检查参数个数
if test "$#" -eq 0
then
    echo "用法: $0 <名字>"
    exit 1
fi

echo "你好, $1"


保存后

./test.sh           # 不给参数，看报错
./test.sh Alice     # 给参数，看输出
echo $?             # 看退出状态


$? 要配合 echo 用，不能直接敲 $?——因为 shell 会把它展开成数字，然后把那个数字当命令来执行，所以报 1: command not found。

---

看到 0 和 1 之后，再试试 && 和 || 的用法——这是 $? 最实用的场景：

./test.sh Alice && echo "脚本成功了"
./test.sh          && echo "这行不会出现"
./test.sh          || echo "脚本失败了，走这里"


&& 的意思是"前面成功才执行后面"，|| 是"前面失败才执行后面"。

---

## While 循环 和 For 循环



![while循环和for循环](whileandfor.png)



---

## case 语句。
case 是 if/elif/elif/... 的简洁替代，特别适合一个变量对应多种情况的时候。语法是这样：


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

注意三个细节：

每个分支用 ;; 结束
* 是默认分支，相当于 else
模式用的是 glob，不是正则


---


case 的 *.c 和正则的 *.c 有什么区别？

Glob（case 用的）：

* = 匹配任意个任意字符
*.c = 以 .c 结尾的任何字符串
只有 * ? [] 三种特殊符号

正则表达式（grep/sed 用的）：

* = 前面那个字符重复0次或多次
*.c = 任意个任意字符后面跟一个 c（. 在正则里是"任意单个字符"）
.*\.c$ 才是正则里"以 .c 结尾"的写法


---

## 函数和 local 变量。


    #!/bin/dash


    is_prime() {
        local n=$1
        local i=2
        while test "$i" -lt "$n"
        do
            if test $((n%i)) -eq 0
            then
                return 1
            fi
            i=$((i+1))
        done
        return 0
    }

    i=2
    while test "$i" -lt 10
    do
        if is_prime "$i"
        then
            echo "$i is prime"
        fi
        i=$((i+1))
    done


---

## I/O 重定向和管道陷阱。


    #!/bin/dash

    total=0
    echo "3
    5
    7" | while read n
    do
        echo "现在读到 $n"
        total=$((total+n))
    done
    echo "总和$total"



    echo "3
    5
    7" > /tmp/nums.txt


    total=0
    while read n 
    do
        total=$((total+n))
    done < /tmp/nums.txt

    echo "total num : $total"


---
## trap 和 mktemp

这两个配合使用，解决一个实际问题：脚本用了临时文件，但如果中途被 Ctrl+C 打断，临时文件就留在磁盘上了。
先感受问题，把 test.sh 改成：

    #!/bin/dash

    echo "开始工作..."
    sleep 10
    echo "工作完成"

跑起来然后立刻 Ctrl+C 打断：

    ./test.sh


然后进入正确写法，把 test.sh 改成：

    #!/bin/dash

    TMP=$(mktemp)
    echo "创建了临时文件: $TMP"
    trap 'rm -f "$TMP"; exit' INT TERM EXIT

    echo "开始工作..."
    echo "一些数据" > "$TMP"
    cat "$TMP"
    sleep 5
    echo "工作完成"


跑起来，Ctrl+C 打断，然后检查临时文件还在不在：

    ./test.sh
    ls /tmp/tmp*


完美！trap 起作用了——Ctrl+C 打断之后，/tmp/tmp.O6u2N211Yb 这个临时文件已经被自动删掉了，在列表里找不到它了（其他的都是别人的文件，你没权限）。
现在解释三个信号：

![trap and mktmp](trap.png)


记住这个模板，以后写用到临时文件的脚本都这样开头：

    TMP=$(mktemp)
    trap 'rm -f "$TMP"; exit' INT TERM EXIT


所以 trap 的作用就是——无论脚本怎么结束，都保证临时文件被清理掉，不留垃圾在 /tmp 里。


---


第十关：{} vs () 子Shell




