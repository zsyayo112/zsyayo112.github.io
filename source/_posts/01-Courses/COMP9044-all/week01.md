---
title: week01
date: 2026-03-06T10:34:29+11:00
tags:
categories: COMP9044

---

# WEEK1 基础概念

---

## Grep的使用 Grep 是 Unix/Linux 系统中最常用的文本搜索工具，名字来自 Global Regular Expression Print。


基本语法： grep [选项] 模式 文件


### Grep的家族成员

grep -F(Fixed): 最快, 不解析正则， 只找死字符

grep -E(Extended): 最常用,支持 +, ?, |, ()

grep -P(Perl): 最强,支持"断言"(Lookahead) 等高级用法



练习1： 创建了一个test.c 文件

```C
#include <stdio.h>
#define MAX 100

int main() {
    int a = 10; // some comment
    printf("hello world\n");
    return 0;
}

```
问题1:  匹配所有 # 开头的行:

```shell
grep -E "#" test.c

```

问题2: 匹配"空行"
```shell
grep -E "^$" test.c
```
又或者是

```shell
grep -E '^\s*$' test.c

```



练习2: 匹配IP地址

任务：在 Tutorial 中，匹配 129.94.172.1 到 129.94.172.25

理解逻辑：
    前三段固定: 129\.94\.172\. 
    最后一段： 可以是1-9， 或者是10-19， 或者是20-25
    组合: ([1-9]|1[0-9]|2[0-5])

最终指令 grep -E '129\.92\.172\.([0-9]|1[0-9]|2[0-5])'



### 深度理解（避坑指南）

1. 贪婪匹配 vs. 非贪婪匹配

笔记： .* 会匹配尽可能多的字符
例子： 匹配HTML 标签 <a> test </a>

如果使用 <.*> 他会匹配第一个 < 到 > 的所有字符

解决方案: 如果使用[^>]*。 意思是匹配除了 > 以外的所有字符， 知道遇到第一个 > w 为止

正确练习 ： <[^>]*>

2. shell的特殊字符

为什么 grep <p> test.html 会报错：

 < > 是shell里面的重定向符号
 准则： 永远用单引号扩起你的正则表达式

 错误：grep <p>|<br> file

 正确：grep -E '<p>|<br>' file















