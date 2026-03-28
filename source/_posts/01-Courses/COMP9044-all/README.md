# COMP9044 Shell编程与Python实践学习笔记

本目录专门记录COMP9044课程的学习内容，包括Shell编程、Python实践、题目练习和实验作业。

## 课程特点
COMP9044不仅包含Shell脚本编程，还涉及Python编程实践，通过两种语言的对比学习，深入理解数据处理和系统编程的核心概念。

## 当前进度

### 已完成
- ✅ **刷题01**：学生选课数据处理（Shell版）
- ✅ **刷题02**：学生选课数据处理（Python版）
- ✅ **Lab01**：简易版本控制系统（Shell脚本）

### 进行中
- ⏳ 总题目数：40题（包含Shell和Python题目）
- ⏳ 已完成：3题
- ⏳ 完成率：7.5%

## 文章索引

### 刷题系列
1. [COMP9044-刷题-01-学生选课数据处理](./COMP9044-刷题-01-学生选课数据处理.md)
   - Shell Pipeline设计与实现
   - 字典污染和窗口时机陷阱分析
   - 调试技巧和测试方法

2. [COMP9044-刷题-02-Python版学生选课数据处理](./COMP9044-刷题-02-Python版学生选课数据处理.md)
   - 从Shell到Python的思维转换
   - 字符串处理和数据结构选择
   - 自动化测试和性能考虑

### Lab系列
1. [COMP9044-Lab-01-简易版本控制系统](./COMP9044-Lab-01-简易版本控制系统.md)
   - 模拟Git基本逻辑的实现
   - Shell脚本典型错误分析
   - 防御式编程和测试验证

## 知识点分类

### Shell编程部分
#### 基础命令
- 文件操作：`ls`, `cd`, `cp`, `mv`, `rm`, `mkdir`
- 文本处理：`cat`, `grep`, `sed`, `awk`, `cut`, `sort`, `uniq`
- 权限管理：`chmod`, `chown`, `sudo`

#### Shell编程核心
- 变量：定义、使用、作用域
- 条件判断：`if`, `case`, 测试表达式
- 循环控制：`for`, `while`, `until`
- 函数：定义、参数、返回值
- 数组和关联数组

#### 高级特性
- 管道和重定向：`|`, `>`, `>>`, `<`
- 进程管理：`&`, `jobs`, `fg`, `bg`, `kill`
- 信号处理：`trap`
- 正则表达式：基本和扩展正则
- 调试技巧：`set -x`, `echo`, 错误处理

### Python实践部分
#### 数据处理
- 标准输入处理：`sys.stdin`
- 字符串解析：`split()`, `strip()`, 正则表达式
- 数据结构：字典去重、列表排序、集合操作

#### 算法实现
- 从Shell流水线到Python算法的思维转换
- 性能优化：时间复杂度、空间复杂度分析
- 测试驱动：自动化测试脚本编写

#### 跨语言对比
- Shell管道 vs Python函数链
- 文本流处理 vs 数据结构操作
- 命令组合 vs 算法设计

## 解题模板

### 数据处理Pipeline模板
```bash
# 典型的数据处理流水线
input_command | filter_command | transform_command | sort_command | output_command

# 示例：提取去重排序
cut -d'|' -f2,3 | sort -u | cut -d',' -f2 | sed 's/^[[:space:]]*//' | sort
```

### 脚本框架模板
```bash
#!/bin/bash
# 脚本说明

# 函数定义
function helper() {
    echo "Usage: $0 [options]"
    exit 1
}

# 参数检查
if [ $# -ne 1 ]; then
    helper
fi

# 主逻辑
main() {
    # 实现具体功能
}

# 错误处理
trap 'echo "Error occurred"; exit 1' ERR

# 脚本入口
main "$@"
```

## 常见错误与解决方案

### 错误1：变量作用域问题
```bash
# 错误
count=0
if [ condition ]; then
    count=1
fi
# 下次运行时count不会记住1

# 解决方案：使用文件记录状态
echo "1" > .count
count=$(cat .count)
```

### 错误2：字符串处理不健壮
```bash
# 错误：假设只有一个空格
first_name=$(echo "$name" | cut -d' ' -f1)

# 正确：处理多个空格和制表符
first_name=$(echo "$name" | awk '{print $1}')
```

### 错误3：目录拷贝问题
```bash
# 错误：尝试拷贝目录
cp directory .

# 正确：拷贝目录内容
cp directory/* .
```

## 学习资源

### 官方文档
- [Bash Reference Manual](https://www.gnu.org/software/bash/manual/)
- [Advanced Bash-Scripting Guide](https://tldp.org/LDP/abs/html/)

### 在线工具
- [Explain Shell](https://explainshell.com/) - 命令解释
- [ShellCheck](https://www.shellcheck.net/) - 代码检查
- [Regex101](https://regex101.com/) - 正则表达式测试

### 练习平台
- [HackerRank Shell Practice](https://www.hackerrank.com/domains/shell)
- [LeetCode Shell Problems](https://leetcode.com/problemset/shell/)

## 学习计划

### 短期目标（1-2周）
1. 完成基础命令练习（10题，包含Shell和Python）
2. 掌握Shell脚本基本语法和Python数据处理
3. 能够编写简单的自动化脚本和数据处理程序

### 中期目标（3-4周）
1. 完成中级题目练习（15题）
2. 掌握高级Shell特性和Python算法优化
3. 能够解决复杂的数据处理问题
4. 理解Shell和Python的适用场景和转换技巧

### 长期目标（5-8周）
1. 完成所有题目练习（40题，Shell和Python混合）
2. 掌握系统管理和自动化
3. 能够设计复杂的Shell应用和Python数据处理管道
4. 建立跨语言编程思维，灵活选择合适工具解决问题

## 质量要求

### 代码质量
1. **可读性**：清晰的注释和结构
2. **健壮性**：处理边界情况和错误
3. **效率**：考虑时间和空间复杂度
4. **可维护性**：模块化设计，易于修改

### 博客质量
1. **完整性**：覆盖问题分析、解决方案、测试验证
2. **深度**：不仅给出答案，还要分析原理
3. **实用性**：提供可复用的模板和技巧
4. **启发性**：引发思考，提出扩展问题

## 相关项目

### 学习工具
- [OpenClaw学习监督系统](../../../../.openclaw/workspace/)
- [学习进度跟踪](../../../../life-os/data/learning-progress.json)

### 技术博客
- [算法题解](../../02-LeetCode/)
- [项目实践](../../03-Projects/)

### 学习规划
- [完整学习计划](../../../../life-os/data/study-plan.md)
- [博客结构规划](../../BLOG-STRUCTURE.md)

## 更新日志

### 2026-03-28
- 创建COMP9044学习目录
- 添加3篇技术博客
- 建立知识点分类体系
- 制定学习计划和目标

### 2026-03-27
- 开始COMP9044学习
- 完成第1道题目和博客
- 建立学习监督系统

---

*本目录将持续更新，记录COMP9044学习的全过程。通过系统的整理和分享，不仅完成课程要求，更要深入理解Shell编程的精髓，为后续的技术学习打下坚实基础。*