---
title: COMP9044 Lab01 - 简易版本控制系统：Shell脚本实现与典型错误分析
date: 2026-03-28T13:50:00+11:00
tags: [Shell, Linux, COMP9044, 版本控制, 脚本, 错误分析]
categories: 01-Courses/COMP9044-all
---

# COMP9044 Lab01 - 简易版本控制系统：Shell脚本实现与典型错误分析

> 本文记录COMP9044 Lab题目：实现一个简易的版本控制系统。通过模拟Git的基本逻辑（`save`相当于commit，`load`相当于checkout），深入掌握Shell脚本编程的核心概念和常见陷阱。

## 题目背景与目标

### 项目概述
实现一个简易的版本控制系统，包含两个脚本：

1. **`snapshot-save.sh`** - 相当于 `git commit`
   - 创建当前目录的快照
   - 自动递增版本号
   - 排除脚本自身和已有快照目录

2. **`snapshot-load.sh`** - 相当于 `git checkout`
   - 加载指定版本号的快照
   - 加载前自动创建当前状态的紧急备份
   - 恢复文件到指定版本

### 核心逻辑要求

#### 1. 自动计数机制
- 通过循环检查 `.snapshot.n` 是否存在
- 找到第一个可用的数字作为版本号
- 保证备份线性递增，不覆盖旧记录

#### 2. `snapshot-save.sh` 职责
- 创建目录：`mkdir .snapshot.$count`
- 选择性拷贝：遍历当前目录，只拷贝普通文件
- 排除项：脚本自身和已有的 `.snapshot.*` 文件夹

#### 3. `snapshot-load.sh` 职责
- 先存档后读取：调用 `snapshot-save.sh` 创建紧急备份
- 恢复文件：将目标快照文件夹内的文件拷贝回当前目录

## 完整实现代码

### `snapshot-save.sh`
```bash
#!/bin/bash
# snapshot-save.sh - 创建目录快照

# 查找可用的快照编号
count=0
while [ -e ".snapshot.$count" ]
do
    count=$((count + 1))
done

# 创建快照目录
mkdir ".snapshot.$count"
echo "Creating snapshot $count"

# 拷贝当前目录的文件到快照目录
for file in *
do
    # 排除脚本自身和已有的快照目录
    if [ "$file" != "snapshot-save.sh" ] && \
       [ "$file" != "snapshot-load.sh" ] && \
       [ "${file:0:9}" != ".snapshot." ]
    then
        # 只拷贝普通文件
        if [ -f "$file" ]
        then
            cp "$file" ".snapshot.$count/"
        fi
    fi
done

echo "Snapshot $count created successfully."
```

### `snapshot-load.sh`
```bash
#!/bin/bash
# snapshot-load.sh - 加载指定快照

# 检查参数
if [ $# -ne 1 ]
then
    echo "Usage: $0 <snapshot-number>"
    exit 1
fi

snapshot_num=$1

# 检查快照是否存在
if [ ! -d ".snapshot.$snapshot_num" ]
then
    echo "Error: Snapshot $snapshot_num does not exist"
    exit 1
fi

# 先创建当前状态的紧急备份
echo "Creating emergency backup before loading..."
./snapshot-save.sh

# 恢复文件
echo "Loading snapshot $snapshot_num..."
cp ".snapshot.$snapshot_num"/* . 2>/dev/null

echo "Snapshot $snapshot_num loaded successfully."
```

## 典型错误分析与解决方案

### ❌ 错误1：逻辑结构错误 - `if` vs `while`

#### 错误代码：
```bash
# 错误：只能处理0到1的一次递增
if [ -e .snapshot.0 ]
then
    count=1
else
    count=0
fi
```

#### 问题分析：
- 当 `.snapshot.1` 也存在时，脚本就"卡死"在1了
- 无法处理多个已存在的快照

#### 正确方案：
```bash
# 正确：使用while循环处理不确定次数的查找
count=0
while [ -e ".snapshot.$count" ]
do
    count=$((count + 1))
done
```

#### 教训：
- 处理**不确定次数**的查找时，必须使用 **`while` 循环**
- `if` 语句只适合确定性的条件判断

### ❌ 错误2：进程生命周期误解

#### 错误理解：
认为在脚本里执行了 `count=$((count+1))`，下次运行脚本时 `count` 就会从新值开始。

#### 原因分析：
- Shell变量只存在于当前运行的进程中
- 脚本结束，变量消失
- 每次运行脚本都是全新的进程

#### 正确理解：
脚本的状态必须通过**外部文件或目录结构**来判断：
```bash
# 通过检查文件夹是否存在来判断状态
while [ -e ".snapshot.$count" ]
do
    count=$((count + 1))
done
```

#### 教训：
- 不要依赖变量记忆跨进程的状态
- 使用文件系统作为持久化存储

### ❌ 错误3：路径引用问题 - `./` 的陷阱

#### 错误代码：
```bash
# 在load脚本里
./snapshot-save.sh
```

#### 问题分析：
- `autotest` 测试环境会将脚本放在父目录
- 在 `tmp` 文件夹下找不到 `./` 开头的文件
- 导致测试失败

#### 正确方案：
```bash
# 直接写脚本名，让系统通过PATH查找
snapshot-save.sh
```

#### 教训：
- 在自动化测试环境下，直接写脚本名最稳妥
- 避免使用相对路径，除非确定当前目录

### ❌ 错误4：通配符与目录拷贝 - `/*` 的缺失

#### 错误代码：
```bash
cp .snapshot.$n .
```

#### 错误信息：
```
cp: omitting directory '.snapshot.0'
```

#### 原因分析：
- `cp` 命令默认不拷贝文件夹
- 需要拷贝的是文件夹**里的内容**，不是文件夹本身

#### 正确方案：
```bash
# 使用通配符展开文件夹内容
cp ".snapshot.$snapshot_num"/* .
```

#### 重要细节：
```bash
# 错误：引号会阻止星号展开
cp ".snapshot.$n/*" .

# 正确：星号在引号外，Shell会先展开
cp ".snapshot.$n"/* .
```

#### 教训：
- 理解Shell通配符展开的时机
- 区分拷贝文件夹和拷贝文件夹内容

## 关键编程技巧

### 1. 防御式文件检查
```bash
# 在循环中检查文件类型
for file in *
do
    if [ -f "$file" ]  # 只处理普通文件
    then
        # 处理文件
    fi
done
```

### 2. 字符串模式匹配
```bash
# 检查文件名是否以.snapshot.开头
if [ "${file:0:9}" != ".snapshot." ]
then
    # 处理非快照文件
fi
```

### 3. 错误处理与用户反馈
```bash
# 检查参数数量
if [ $# -ne 1 ]
then
    echo "Usage: $0 <snapshot-number>"
    exit 1
fi

# 检查目录是否存在
if [ ! -d ".snapshot.$snapshot_num" ]
then
    echo "Error: Snapshot $snapshot_num does not exist"
    exit 1
fi
```

### 4. 静默错误处理
```bash
# 忽略cp命令的某些错误
cp ".snapshot.$snapshot_num"/* . 2>/dev/null
```

## 测试与验证

### 测试用例设计

#### 测试1：基本功能测试
```bash
# 1. 创建一些测试文件
echo "Hello" > file1.txt
echo "World" > file2.txt

# 2. 创建快照
./snapshot-save.sh

# 3. 修改文件
echo "Modified" > file1.txt

# 4. 创建第二个快照
./snapshot-save.sh

# 5. 加载第一个快照
./snapshot-load.sh 0

# 6. 验证文件内容
cat file1.txt  # 应该显示"Hello"
```

#### 测试2：边界条件测试
```bash
# 1. 创建多个快照
for i in {1..5}; do ./snapshot-save.sh; done

# 2. 检查目录结构
ls -la .snapshot.*

# 3. 测试不存在的快照
./snapshot-load.sh 99  # 应该报错
```

#### 测试3：排除规则测试
```bash
# 1. 创建应该被排除的文件
mkdir .snapshot.old
touch snapshot-save.sh  # 同名但不是脚本

# 2. 运行快照
./snapshot-save.sh

# 3. 检查快照内容
ls -la .snapshot.0/  # 不应该包含排除的文件
```

### 使用ShellCheck进行代码检查
```bash
# 安装ShellCheck
# Ubuntu/Debian: sudo apt-get install shellcheck
# macOS: brew install shellcheck

# 检查脚本
shellcheck snapshot-save.sh
shellcheck snapshot-load.sh
```

#### 常见ShellCheck警告：
1. **SC2034**: 变量未使用
2. **SC2086**: 双引号缺失可能导致单词分割
3. **SC2044**: for循环中的通配符匹配
4. **SC2162**: read命令缺少-r参数

## 扩展练习

### 练习1：添加版本信息
修改脚本，在每个快照目录中创建 `version.info` 文件，包含：
- 创建时间戳
- 创建者（当前用户）
- 文件数量统计

### 练习2：实现差异比较
添加 `snapshot-diff.sh` 脚本，比较两个快照之间的差异：
```bash
#!/bin/bash
# snapshot-diff.sh <snapshot1> <snapshot2>
# 输出两个快照之间的文件差异
```

### 练习3：添加压缩功能
修改保存脚本，将快照目录压缩为 `.tar.gz` 文件，节省磁盘空间。

### 练习4：实现标签系统
添加标签功能，允许给特定的快照打标签：
```bash
./snapshot-tag.sh <snapshot-number> <tag-name>
./snapshot-load-by-tag.sh <tag-name>
```

## 学习总结

### 掌握的核心概念：
1. **Shell脚本状态管理**：使用文件系统而非变量
2. **循环控制**：`while` vs `if` 的正确使用场景
3. **文件操作**：拷贝、检查、排除的完整流程
4. **错误处理**：参数检查、目录存在性验证
5. **路径处理**：相对路径与绝对路径的选择

### 编程习惯养成：
1. **始终使用ShellCheck**：在提交前检查代码质量
2. **防御式编程**：检查所有外部输入和条件
3. **清晰的错误信息**：帮助用户理解问题
4. **模块化设计**：每个脚本职责单一
5. **充分的测试**：覆盖正常和边界情况

### 版本控制概念映射：
- `snapshot-save.sh` ≈ `git commit`
- `snapshot-load.sh` ≈ `git checkout`
- `.snapshot.*` 目录 ≈ `.git` 目录
- 自动编号 ≈ Git的哈希值

## 相关资源

### 官方文档：
- [Bash Reference Manual](https://www.gnu.org/software/bash/manual/)
- [ShellCheck Wiki](https://github.com/koalaman/shellcheck/wiki)

### 在线练习：
- [Explain Shell](https://explainshell.com/) - 命令解释
- [ShellCheck Online](https://www.shellcheck.net/) - 在线代码检查
- [Bash Academy](https://www.bash.academy/) - 交互式学习

### 进阶学习：
1. **高级Bash脚本编程**：函数、数组、信号处理
2. **Makefile编写**：自动化构建和测试
3. **系统管理脚本**：日志轮转、备份、监控
4. **DevOps工具链**：与CI/CD管道集成

---

*本文记录于2026年3月28日，基于COMP9044 Lab实际题目总结。通过实现简易版本控制系统，不仅掌握了Shell脚本编程技巧，更重要的是理解了版本控制的基本原理和软件工程中的状态管理思想。*