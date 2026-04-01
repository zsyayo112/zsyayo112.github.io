---
title: COMP2041 Week 01,02 — Unix Filters 完整笔记
date: 2026-04-01
tags:
  - COMP2041
  - Shell
  - Linux
  - UNSW
categories:
  - COMP9044
---

# COMP2041 Week 01-02 — Unix Filters

## 什么是 Filter？

Filter 是一种**读取输入、变换数据、写入输出**的程序。Unix 的核心哲学之一就是把多个小 filter 用管道 `|` 串联起来，解决复杂问题。

```bash
filter1 | filter2 | filter3
```

每个 filter 从 stdin 读取，写到 stdout，不需要知道前后是什么。

---

## Filter 分类总览

| 类型 | 命令 | 作用 |
|------|------|------|
| 横向切片（选行） | `cat` `head` `tail` `grep` `sed` `uniq` | 选出满足条件的行 |
| 纵向切片（选列） | `cut` `sed` | 选出特定字段/字符 |
| 替换 | `tr` `sed` | 字符/字符串替换 |
| 统计 | `wc` `uniq -c` | 统计行数/词频 |
| 组合 | `paste` `join` | 合并多个文件 |
| 排序 | `sort` | 重新排列行 |
| 文件搜索 | `find` | 搜索文件系统 |

---

## 核心命令详解

### grep — 按正则筛选行

```bash
grep 'pattern' file          # 基本匹配
grep -E 'pattern' file       # 扩展正则（推荐）
grep -i 'pattern' file       # 忽略大小写
grep -v 'pattern' file       # 反转：只输出不匹配的行
grep -c 'pattern' file       # 只输出匹配行数
grep -w 'pattern' file       # 只匹配完整单词
grep -x 'pattern' file       # 只匹配整行
```

**grep 家族：**
- `grep -F`：纯字符串匹配，不用正则，速度更快
- `grep -G`：基础正则（默认），不支持 `+` `?` `|` `()`
- `grep -E`：扩展正则，日常推荐用这个
- `grep -P`：Perl 扩展正则，需要更复杂功能时用

**踩坑记录：**

```bash
# ❌ 错误：| 是 shell 的管道，不是正则的或
grep <p>|<br> file

# ✅ 正确：加引号，转义 <
grep -E '<p>|<br>' file
```

---

### 正则表达式基础

| 语法 | 含义 | 例子 |
|------|------|------|
| `.` | 任意单个字符 | `a.c` 匹配 `abc`、`aXc` |
| `*` | 零个或多个 | `ab*c` 匹配 `ac`、`abc`、`abbc` |
| `+` | 一个或多个 | `[0-9]+` 匹配任意整数 |
| `?` | 零个或一个 | `colou?r` 匹配 `color`、`colour` |
| `^` | 行首 | `^hello` 匹配以 hello 开头的行 |
| `$` | 行尾 | `bash$` 匹配以 bash 结尾的行 |
| `[abc]` | 字符集 | `[aeiou]` 匹配任一元音 |
| `[^abc]` | 排除字符集 | `[^:]` 匹配非冒号字符 |
| `{n}` | 恰好 n 次 | `[0-9]{4}` 匹配4位数字 |
| `{n,m}` | n 到 m 次 | `[0-9]{2,4}` 匹配2到4位数字 |
| `\|` | 或（需 `-E`） | `cat\|dog` 匹配 cat 或 dog |
| `()` | 分组（需 `-E`） | `(ab)+` 匹配 ab、abab |

**重要：`^` 有两种含义！**
- `^[abc]`：行首是 a、b 或 c
- `[^abc]`：不是 a、b、c 的任意字符

**常用模式：**

```bash
[^:]*        # 匹配到下一个冒号为止的所有内容（处理冒号分隔文件常用）
[0-9]{4}     # 恰好4位数字（匹配学号、年份等）
[ \t]+$      # 行尾空白（检测 trailing whitespace）
```

---

### tr — 字符替换

`tr` 逐字符替换，**只从 stdin 读取**，不接受文件名参数。

```bash
tr 'A-Z' 'a-z' < file          # 大写转小写
tr 'a-z' 'A-Z' < file          # 小写转大写
tr -d '0-9' < file              # 删除所有数字
tr -s ' ' < file                # 压缩连续空格为一个
tr -cs 'a-zA-Z0-9' '\n' < file  # 分词：非字母数字替换为换行
```

**`-c` 和 `-s` 的含义：**
- `-c`：complement，取补集（匹配不在集合里的字符）
- `-s`：squeeze，压缩连续重复字符，只保留一个
- `-d`：delete，删除匹配的字符

**经典三步词频统计：**

```bash
tr 'A-Z' 'a-z' < file | tr -cs 'a-zA-Z0-9' '\n' | sort | uniq -c | sort -rn
```

---

### cut — 纵向切片

```bash
cut -d: -f1 file          # 取冒号分隔的第1列
cut -d: -f1,3 file        # 取第1和第3列
cut -d: -f2- file         # 取第2列到最后
cut -c1-5 file            # 取每行第1到第5个字符
```

**注意：`-d` 必须和 `-f` 一起用，单独用 `-d` 会报错。**

`cut` 的局限：
- 无法引用"最后一列"（不像 awk 的 `$NF`）
- 无法重新排列列的顺序

---

### sort — 排序

```bash
sort file                   # 默认字典序排序
sort -r file                # 倒序
sort -n file                # 数字排序（不加 -n 的话 10 < 2）
sort -rn file               # 数字倒序
sort -t: -k2 file           # 按冒号分隔的第2列排序
sort -t: -k2,2 file         # 只用第2列排序（不延伸到后面的列）
sort -t'|' -k2,2 -rn file   # 按第2列数字倒序
```

**`-k2` vs `-k2,2` 的区别：**
- `-k2`：从第2列排到行尾
- `-k2,2`：只用第2列排序，更精确

**口诀：先 `sort` 再 `uniq`，先 `sort` 再 `join`**

---

### uniq — 去重/统计

```bash
uniq file           # 删除相邻重复行
uniq -c file        # 统计每行出现次数
uniq -d file        # 只输出重复的行
uniq -u file        # 只输出不重复的行
```

**关键：`uniq` 只处理相邻重复行，所以几乎总是先 `sort`！**

```bash
sort file | uniq -c | sort -rn    # 词频统计标准写法
```

---

### sed — 流编辑器

```bash
sed 's/old/new/' file       # 替换每行第一个匹配
sed 's/old/new/g' file      # 替换每行所有匹配
sed -n '/pattern/p' file    # 只打印匹配行（等价于 grep）
sed '/pattern/d' file       # 删除匹配行
sed -n '1,10p' file         # 打印第1到10行
sed -n '81,100p' file       # 打印第81到100行
```

**`-n` 和 `p` 的配合：**
- `sed` 默认打印每一行
- `-n` 抑制默认输出
- `p` 命令显式打印
- 两者配合 = 只打印匹配的行

**处理含 `/` 的 pattern（比如路径）：**

```bash
# ❌ 冲突：/ 和 sed 分隔符冲突
sed -n '/bin/bash/p' file

# ✅ 方法1：转义
sed -n '/\/bin\/bash/p' file

# ✅ 方法2：换分隔符（更清晰）
sed -n '\|/bin/bash|p' file
```

**捕获组（`-E` 才能用）：**

```bash
# 对调前两列
sed -E 's/([^:]*):([^:]*)/\2:\1/' file
```

要点：
1. 用 `()` 括起来才能捕获
2. `\1` 对应第一个括号，`\2` 对应第二个
3. pattern 要用 `[^:]*` 而不是 `[a-zA-Z]*`，否则会漏掉特殊字符

---

### join — 按 key 合并文件

```bash
join file1 file2                    # 按第1列 join（默认空格分隔）
join -t'|' file1 file2              # 指定分隔符
join -t'|' -a1 -a2 file1 file2     # 显示所有行（含不匹配的）
join -t'|' -a1 -a2 -e '--' -o 0,1.2,2.2 file1 file2  # 缺失字段用 -- 填充
```

**`-o` 输出格式：**
- `0`：key 字段
- `1.2`：第1个文件的第2列
- `2.2`：第2个文件的第2列

**口诀：永远先 `sort` 再 `join`**

```bash
sort file1 > sorted1.txt
sort file2 > sorted2.txt
join -t'|' sorted1.txt sorted2.txt
```

---

## 管道使用注意事项

### 重定向位置很关键

```bash
# ❌ 错误：< 作用在 cut 上，grep 没有输入
grep pattern | cut -d: -f1 < file

# ✅ 正确：文件给第一个命令
grep pattern file | cut -d: -f1

# ✅ 也正确：< 放在最前面
grep pattern < file | cut -d: -f1
```

**规则：`<` 只作用于它紧跟的那个命令，不影响整条管道。**

### 不要同时读写同一个文件

```bash
# ❌ 危险：shell 先截断 story.txt，sed 读到空文件
sed 's/[aeiou]//g' story.txt > story.txt

# ✅ 正确：先写临时文件再覆盖
sed 's/[aeiou]//g' story.txt > story.txt.new && mv story.txt.new story.txt

# ✅ 或用 -i（部分 sed 支持）
sed -i 's/[aeiou]//g' story.txt
```

---

## 实战练习总结

### 经典管道组合

**统计词频（不区分大小写）：**
```bash
tr 'A-Z' 'a-z' < file | tr -cs 'a-zA-Z0-9' '\n' | sort | uniq -c | sort -rn
```

**统计每种 shell 的用户数：**
```bash
cut -d: -f7 passwd | sort | uniq -c | sort -rn
```

**找出高于平均分的学生（成绩 ≥ 70）：**
```bash
grep -E '\|[7-9][0-9]$' marks.txt | sort -t'|' -k2,2 -rn
```

**找出两门课都选了的学生：**
```bash
sort marks1.txt > s1.txt && sort marks2.txt > s2.txt
join -t'|' s1.txt s2.txt
```

**显示所有学生，缺课用 `--` 填充：**
```bash
join -t'|' -a1 -a2 -e '--' -o 0,1.2,2.2 s1.txt s2.txt
```

---

## 踩坑总结

| 坑 | 原因 | 解法 |
|---|---|---|
| `grep -E "[a-zA-Z]{2}[0-9]{4}"` 匹配到不该匹配的行 | 没有用 `^` 锚定行首 | 加 `^`：`grep -E "^(cs\|se)[0-9]{4}:"` |
| `cut -d':'` 报错 | `-d` 必须配合 `-f` 使用 | `cut -d: -f1` |
| `sort` 后数字顺序不对（10 在 2 前面） | 默认字典序，不是数字序 | 加 `-n`：`sort -n` |
| `uniq` 没有去掉所有重复 | `uniq` 只处理相邻重复行 | 先 `sort` 再 `uniq` |
| `join` 报 not sorted 错误 | `join` 要求输入已排序 | 先 `sort` 再 `join` |
| `sed '/bin/bash/p'` 报错 | `/` 和 sed 分隔符冲突 | 用 `\|/bin/bash|p` 或转义 `\/` |
| 捕获组 `\2` 报 invalid reference | pattern 里没有用 `()` | 加括号：`([^:]*)` |
| 重定向位置错 `cmd1 \| cmd2 < file` | `<` 作用在 cmd2 而非 cmd1 | `cmd1 file \| cmd2` 或 `cmd1 < file \| cmd2` |