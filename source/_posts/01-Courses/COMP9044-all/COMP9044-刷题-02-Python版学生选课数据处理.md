---
title: COMP9044 刷题02 - Python版学生选课数据处理：从Shell到Python的思维转换
date: 2026-03-28T13:50:00+11:00
tags: [Python, Shell, COMP9044, 数据处理, 算法, 字符串处理, 课程作业]
categories: 01-Courses/COMP9044-all
---

# COMP9044 刷题02 - Python版学生选课数据处理：从Shell到Python的思维转换

> 本文记录COMP9044课程中的Python实践题目：用Python实现学生选课数据处理。这道题是COMP9044课程的重要组成部分，通过对比Shell和Python两种实现方式，深入理解数据处理的核心逻辑和跨语言编程思维差异。

## 课程背景
COMP9044不仅包含Shell脚本编程，还涉及Python编程实践。通过同一问题在不同语言中的实现，帮助学生理解：
1. 不同编程语言的思维模式和适用场景
2. 数据处理的核心算法和逻辑
3. 从命令行工具到编程语言的技能迁移

## 题目背景与目标

### 原题描述
输入格式：标准输入（stdin），每行数据以 `|` 分隔，格式为：
```
课程代码|学号|姓名
```
姓名格式通常为：`姓, 名 中间名`（例如：`Thorpe, Ian Augustus`）

### 核心逻辑要求
1. **学生去重**：同一个学生（由唯一的学号标识）可能出现在多行中（选了多门课），但在输出中该学生只计入一次
2. **姓名提取**：从全名中精准提取First Name（名），忽略姓氏和中间名
3. **重名处理**：如果两个不同的学生（学号不同）拥有相同的First Name，该名字需重复打印
4. **排序输出**：将提取出的所有First Name按字母升序排列，每行打印一个

### 运行示例
```bash
./final_q2.py < enrollments.txt
```

### 期望输出
```
Heidi
Ian
Ian
Kevin
Liam
Mary
Mary
Mary
Wei
Wei
```

## Shell流水线 vs Python实现

### Shell实现（参考）
```bash
# 对应的Shell流水线
cut -d'|' -f2,3 | sort -u -k1,1 | cut -d',' -f2 | awk '{print $1}' | sort
```

### Python实现思路
在Python中，推荐使用"单循环处理 + 字典存储"的架构，模拟Shell流水线的逻辑。

## 完整Python实现

### 方案1：基础版本
```python
#!/usr/bin/env python3
# final_q2.py - Python版学生选课数据处理

import sys

def main():
    # 使用字典存储，key为学号，value为first name
    student_data = {}
    
    # 读取标准输入
    for line in sys.stdin:
        line = line.strip()
        if not line:
            continue
        
        # 拆分字段
        parts = line.split('|')
        if len(parts) < 3:
            continue
        
        student_id = parts[1].strip()
        full_name = parts[2].strip()
        
        # 提取first name
        # 格式: "姓, 名 中间名"
        name_parts = full_name.split(',')
        if len(name_parts) < 2:
            continue
        
        # 提取"名"部分，并取第一个单词
        first_name_part = name_parts[1].strip()
        first_name = first_name_part.split()[0] if first_name_part else ""
        
        if first_name:
            # 使用学号作为key，自动去重
            student_data[student_id] = first_name
    
    # 获取所有的first name并排序
    first_names = list(student_data.values())
    first_names.sort()
    
    # 输出结果
    for name in first_names:
        print(name)

if __name__ == "__main__":
    main()
```

### 方案2：优化版本（更健壮）
```python
#!/usr/bin/env python3
# final_q2.py - 优化版本

import sys

def extract_first_name(full_name):
    """
    从全名中提取first name
    
    参数:
        full_name: 格式为"姓, 名 中间名"的字符串
    
    返回:
        first name字符串，如果提取失败返回None
    """
    try:
        # 按逗号分割
        parts = full_name.split(',', 1)
        if len(parts) < 2:
            return None
        
        # 获取"名 中间名"部分
        given_names = parts[1].strip()
        
        # 按空格分割，取第一个单词
        first_name = given_names.split()[0]
        return first_name
    except (IndexError, AttributeError):
        return None

def process_input():
    """处理输入数据"""
    student_data = {}
    
    for line_num, line in enumerate(sys.stdin, 1):
        line = line.strip()
        if not line:
            continue
        
        # 拆分字段
        parts = line.split('|')
        if len(parts) < 3:
            # 可以添加日志或跳过
            continue
        
        student_id = parts[1].strip()
        full_name = parts[2].strip()
        
        # 提取first name
        first_name = extract_first_name(full_name)
        if first_name:
            student_data[student_id] = first_name
    
    return student_data

def main():
    # 处理输入
    student_data = process_input()
    
    # 获取并排序first names
    first_names = sorted(student_data.values())
    
    # 输出结果
    for name in first_names:
        print(name)

if __name__ == "__main__":
    main()
```

## 关键知识点与"坑"分析

### 1. Shell重定向 vs 命令行参数

#### 问题场景：
题目要求使用 `< file` 重定向，而不是命令行参数。

#### Shell思维：
```bash
# Shell中很自然
./final_q2.py < enrollments.txt
```

#### Python实现：
```python
# 必须从sys.stdin读取
import sys

for line in sys.stdin:
    # 处理每一行
    pass
```

#### 错误做法：
```python
# 错误：试图从文件参数读取
import sys

if len(sys.argv) > 1:
    with open(sys.argv[1], 'r') as f:
        # 这样不符合题目要求
```

#### 教训：
- 理解标准输入(stdin)和文件参数的区别
- 题目明确要求时，必须使用指定方式

### 2. 字符串处理的健壮性

#### 关键技巧：
```python
# split()的无参数用法会自动处理所有空白符
first_name = name_part.split()[0]
```

#### 为什么这样更健壮：
1. **处理前导/尾随空格**：`"  Ian Augustus  "` → `["Ian", "Augustus"]`
2. **处理制表符**：`"Ian\tAugustus"` → `["Ian", "Augustus"]`
3. **处理多个空格**：`"Ian  Augustus"` → `["Ian", "Augustus"]`

#### 对比其他方法：
```python
# 方法1：指定分隔符（不够健壮）
first_name = name_part.split(' ')[0]  # 只能处理单个空格

# 方法2：使用正则表达式（过度设计）
import re
first_name = re.split(r'\s+', name_part)[0]

# 方法3：最佳实践（推荐）
first_name = name_part.split()[0]
```

### 3. 字典的妙用：学号去重 vs 名字去重

#### 错误理解：
```python
# 错误：直接用set存名字
first_names = set()
for line in sys.stdin:
    first_name = extract_first_name(line)
    first_names.add(first_name)  # 这会合并重名学生！
```

#### 问题分析：
- 两个不同的学生（学号不同）可能同名
- 使用set会错误地合并他们
- 不符合题目要求"重名重复打印"

#### 正确方案：
```python
# 正确：先用学号做key过滤学生
student_data = {}  # key: 学号, value: first_name

for line in sys.stdin:
    student_id = extract_student_id(line)
    first_name = extract_first_name(line)
    
    # 同一个学号多次出现会被覆盖
    student_data[student_id] = first_name

# 然后取所有的values
first_names = list(student_data.values())
```

#### 逻辑对应Shell命令：
```bash
# 对应Shell的 sort -u -k1,1 （按第一列去重）
cut -d'|' -f2,3 | sort -u -k1,1
```

### 4. 环境声明与可执行权限

#### Shell脚本习惯：
```bash
#!/bin/bash
# 第一行声明解释器
```

#### Python脚本对应：
```python
#!/usr/bin/env python3
# 使用env查找python3，更便携
```

#### 设置可执行权限：
```bash
# 使脚本可执行
chmod +x final_q2.py

# 现在可以直接运行
./final_q2.py < enrollments.txt
```

#### 如果不加shebang：
```bash
# 需要显式指定解释器
python3 final_q2.py < enrollments.txt
```

## 测试与验证

### 测试数据准备
```python
# test_data.py - 生成测试数据
test_data = [
    "COMP1917|3360379|Costner, Kevin Augustus |13978/1|M",
    "COMP1917|3364562|Carey, Mary |13711/1|M",
    "COMP3311|3383025|Thorpe, Ian Augustus |13978/3|M",
    "COMP2920|3860448|Steenburgen, Mary Nell |13978/3|F",
    "COMP1927|3360582|Neeson, Liam |13711/2|M",
    "COMP3411|3863711|Klum, Heidi June Anne |13978/3|F",
    "COMP3141|3383025|Thorpe, Ian Augustus |13978/3|M",  # 重复学号
    "COMP3331|5122456|Wang, Wei |13978/2|M",
    "COMP3331|5456732|Wang, Wei |13648/3|M",  # 不同学号，同名
]

with open('test_enrollments.txt', 'w') as f:
    for line in test_data:
        f.write(line + '\n')
```

### 自动化测试脚本
```python
#!/usr/bin/env python3
# test_final_q2.py - 自动化测试

import subprocess
import sys

def run_test():
    """运行测试"""
    # 准备期望输出
    expected_output = [
        "Heidi",
        "Ian",      # 注意：Ian只出现一次（学号去重）
        "Kevin",
        "Liam",
        "Mary",
        "Mary",     # 注意：Mary出现两次（不同学生）
        "Wei",
        "Wei",      # 注意：Wei出现两次（不同学生）
    ]
    
    # 运行被测试脚本
    try:
        result = subprocess.run(
            ['./final_q2.py'],
            stdin=open('test_enrollments.txt', 'r'),
            capture_output=True,
            text=True,
            timeout=5
        )
    except FileNotFoundError:
        print("错误: 找不到final_q2.py")
        return False
    except subprocess.TimeoutExpired:
        print("错误: 脚本运行超时")
        return False
    
    # 检查输出
    actual_output = result.stdout.strip().split('\n')
    
    # 比较结果
    if actual_output == expected_output:
        print("✅ 测试通过!")
        return True
    else:
        print("❌ 测试失败")
        print(f"期望输出: {expected_output}")
        print(f"实际输出: {actual_output}")
        return False

if __name__ == "__main__":
    # 生成测试数据
    with open('test_enrollments.txt', 'w') as f:
        f.write("""COMP1917|3360379|Costner, Kevin Augustus |13978/1|M
COMP1917|3364562|Carey, Mary |13711/1|M
COMP3311|3383025|Thorpe, Ian Augustus |13978/3|M
COMP2920|3860448|Steenburgen, Mary Nell |13978/3|F
COMP1927|3360582|Neeson, Liam |13711/2|M
COMP3411|3863711|Klum, Heidi June Anne |13978/3|F
COMP3141|3383025|Thorpe, Ian Augustus |13978/3|M
COMP3331|5122456|Wang, Wei |13978/2|M
COMP3331|5456732|Wang, Wei |13648/3|M
""")
    
    # 运行测试
    success = run_test()
    
    # 清理
    import os
    if os.path.exists('test_enrollments.txt'):
        os.remove('test_enrollments.txt')
    
    sys.exit(0 if success else 1)
```

### 边界条件测试

#### 测试1：空输入
```bash
# 应该输出空（无错误）
echo "" | ./final_q2.py
```

#### 测试2：格式错误的数据
```bash
# 缺少字段的行应该被跳过
echo "COMP1917|3360379" | ./final_q2.py  # 缺少姓名字段
echo "Invalid line" | ./final_q2.py      # 格式完全错误
```

#### 测试3：姓名格式变体
```python
# 测试不同的姓名格式
test_cases = [
    "COMP1917|3360379|Costner, Kevin",          # 无中间名
    "COMP1917|3360379|Costner,Kevin",           # 无空格
    "COMP1917|3360379|Costner,  Kevin  ",       # 多余空格
    "COMP1917|3360379|Costner,Kevin Augustus",  # 有中间名
]
```

## 性能优化考虑

### 大数据集处理
```python
# 对于非常大的文件，考虑逐行处理，不一次性加载到内存
def process_large_file():
    student_data = {}
    
    # 逐行读取，内存友好
    for line in sys.stdin:
        # 处理逻辑...
        pass
    
    # 最后一次性排序输出
    first_names = sorted(student_data.values())
    for name in first_names:
        print(name)
```

### 使用生成器（高级）
```python
def read_lines():
    """生成器逐行读取"""
    for line in sys.stdin:
        yield line.strip()

def process_stream():
    """流式处理"""
    student_data = {}
    
    for line in read_lines():
        if not line:
            continue
        
        parts = line.split('|')
        if len(parts) >= 3:
            student_id = parts[1].strip()
            full_name = parts[2].strip()
            
            # 提取first name
            name_parts = full_name.split(',')
            if len(name_parts) >= 2:
                first_name = name_parts[1].strip().split()[0]
                if first_name:
                    student_data[student_id] = first_name
    
    return student_data
```

## 从Shell到Python的思维转换

### 思维对比表

| Shell思维 | Python思维 | 关键差异 |
|-----------|------------|----------|
| 管道连接命令 | 函数调用和数据传递 | Shell是进程间通信，Python是内存操作 |
| 文本流处理 | 数据结构操作 | Shell处理文本行，Python处理对象 |
| 命令组合 | 算法设计 | Shell是命令式，Python更声明式 |
| 进程开销 | 内存开销 | Shell多进程，Python单进程多数据结构 |

### 等效逻辑映射

```bash
# Shell流水线
cut -d'|' -f2,3          # → Python: line.split('|')[1:3]
sort -u -k1,1            # → Python: dict去重 (key=学号)
cut -d',' -f2            # → Python: name.split(',')[1]
awk '{print $1}'         # → Python: part.split()[0]
sort                    # → Python: sorted(list)
```

### 学习建议
1. **先理解Shell逻辑**：Shell流水线更直观展示数据处理流程
2. **再实现Python版本**：将流水线步骤映射到Python操作
3. **对比两种实现**：理解各自的优缺点和适用场景
4. **掌握核心算法**：去重、提取、排序是通用数据处理模式

## 扩展练习

### 练习1：添加统计功能
修改脚本，在输出名字的同时，添加统计信息：
```
Total students: 10
Unique first names: 8
Most common name: Mary (3 times)
```

### 练习2：实现多种输出格式
添加命令行参数支持不同输出格式：
```bash
./final_q2.py --format=csv < enrollments.txt
# 输出: student_id,first_name
```

### 练习3：性能对比
编写脚本对比Shell和Python版本的性能：
```python
import time
import subprocess

# 测试大数据集下的性能差异
```

### 练习4：错误处理增强
添加更完善的错误处理和日志：
- 记录跳过的行数和原因
- 支持不同的编码格式
- 添加进度指示器

## 学习总结

### 掌握的核心技能：
1. **标准输入处理**：`sys.stdin`的正确使用
2. **字符串解析**：健壮的姓名提取算法
3. **数据结构选择**：字典去重 vs 集合去重
4. **排序算法应用**：理解排序的稳定性和复杂度
5. **测试驱动开发**：自动化测试脚本编写

### 编程习惯养成：
1. **始终添加shebang**：`#!/usr/bin/env python3`
2. **模块化设计**：将逻辑拆分为函数
3. **错误处理**：考虑所有边界情况
4. **测试覆盖**：编写全面的测试用例
5. **性能意识**：考虑大数据集下的表现

### 跨语言思维：
通过这道题，我们学习了：
-