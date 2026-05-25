## 1. 编码
默认情况下，python3源码文件以UTF-8编码，所有字符串都是unicode字符串。

当然，你也可以为源码文件指定不同的编码：
```python
## -*_ coding:cp-1252 _*_
```
在上面的定义中，允许在源文件中使用Windows-1252字符集中的字符编码，对应适合语言为保加利亚语、白俄罗斯语、马其顿语、俄语、塞尔维亚语。

## 2. 标识符
### 2.1. 关于标识符
在 Python 中，标识符 (Identifier) 是程序员用来命名变量、函数、类、模块或其他对象的名称。它是代码中最基本的组成部分之一。
为了让代码能够被 Python 解释器正确识别和执行，标识符的命名必须遵循严格的规则，同时为了代码的可读性，社区也形成了一套通用的命名规范（风格指南）。

### 2.1. 标识符命名规则
> Python 3 支持 Unicode 字符，这意味着你可以使用中文、Emoji 甚至数学符号作为标识符（虽然不推荐这样做，除非有特殊需求）。
> 所以下面的规则中我们一般不考虑使用这些字符。

Python中的标识符必须遵循以下规则：
* 第一个字符必须为字母或下划线，不能为数字。
* 标识符的其他部分由字母、数字和下划线组成。
* 标识符对大小写敏感，count和Count是不同的标识符。
* 标识符的长度没有硬性要求，但建议保持简洁（一般不超过20个字符）。
* 禁止使用保留关键字，如`if`、`for`、`class`等。

**正确示例：**
```python
age = 25                # 普通变量名，最常见
user_name = "Alice"     # 用下划线连接单词，清晰易读
_total = 100            # 下划线开头通常表示“内部使用”或“私有”
MAX_SIZE = 1024         # 全大写通常表示“常量”（固定不变的值）
calculate_area()        # 函数名，动词+名词
StudentInfo             # 类名，首字母大写（驼峰命名法）
__private_var           # 双下划线开头，有特殊含义
```


**错误示例：**
```python
2nd_place = "silver"    # 错误：以数字开头
user-name = "Bob"       # 错误：包含连字符
class = "Math"          # 错误：使用关键字
$price = 9.99          # 错误：包含特殊字符
for = "loop"           # 错误：**使用关键字**
```

**特别注意：**
Python 3允许使用Unicode字符作为标识符，可以用中文作为变量名，非ASCII标识符也是允许的。
```python
姓名 = "张三"  # 合法
π = 3.14159   # 合法
```

### 3.3. 标识符命名规范
虽然以下规则不会导致报错，但违反它们会被认为是不专业的代码，降低可读性。Python 官方风格指南 PEP 8 对此有明确规定。

规范如下：
* **变量和函数名**：小写字母 + 下划线 (snake_case / 蛇形命名法)
  ```python
    user_name = "Alice"
    total_count = 0
    def calculate_area():
        pass
  ```
* **类名**：大驼峰命名法 (PascalCase / CapWords)，即每个单词首字母大写，不使用下划线。
  ```python
    class UserProfile:
        pass

    class HttpRequestHandler:
        pass  
  ```
* **常量名**:全大写字母 + 下划线。
  ```python
    MAX_CONNECTIONS = 100
    PI = 3.14159
    API_KEY = "secret_key"
  ```
  > Python 没有真正的常量机制，这只是约定俗成，告诉开发者“这个变量不应该被修改”。
* **私有成员名**：
  * `_var`单下划线开头：表示受保护或内部使用。这是一种约定，告诉其他开发者“虽然你可以访问它，但请不要在类外部直接使用它”。
    * 示例：`_internal_cache`
  * `__var`双下划线开头：表示“私有”。Python 会对其进行名称改写 (Name Mangling)，使其在类外部很难直接访问（主要用于避免子类命名冲突）。
    * 示例：`__password` (在类内部自动变为 `_ClassName__password`)。
  * `__var__`双下划线前后：这是 Python 的魔术方法 (Magic Methods) 或特殊属性，千万不要自己发明这种命名。
    * 示例：`__init__`, `__str__`,`__name__`。


### 3.4. 最佳实践建议
1. 见名知意：标识符应该清晰地描述其用途。
   * 好：days_until_expiry, calculate_total_price
   * 坏：d, func1, x (除非在极短的循环中如 for i in range(10))
2. 避免缩写：除非是众所周知的缩写（如 id, url, html），否则尽量拼写完整。
3. 保持一致性：在整个项目中坚持同一种命名风格（通常遵循 PEP 8）。
4. 慎用中文：虽然支持，但在团队协作、开源项目或部署到某些老旧系统时，中文标识符可能会引起编码问题或阅读障碍。
## 3. Python 保留关键字
### 3.1. 特性
Python 关键字（也称为保留字）是 Python 语言中具有特殊含义的单词，它们被 Python 解释器保留用于特定的语法功能。这些关键字不能用作变量名、函数名或其他标识符。

**特点:**
* 不可变性：关键字是语言规范的一部分，不能修改其含义
* 有限性：Python 的关键字数量是固定的（Python 3.8 有 35 个关键字）
* 大小写敏感：所有关键字都是小写形式
* 语法功能：每个关键字都有特定的语法作用



### 3.2. 查看所有关键字
Python 的标准库提供了一个 keyword 模块，可以输出当前版本的所有关键字：
```python
import keyword

## 打印所有关键字列表
print(keyword.kwlist)

## 检查某个字符串是否是关键字
print(keyword.iskeyword("if"))      # True
print(keyword.iskeyword("hello"))   # False
```
**Python 3.14.3 输出：**
```text
['False', 'None', 'True', 'and', 'as', 'assert', 'async', 'await', 'break', 'class', 'continue', 'def', 'del', 'elif', 'else', 'except', 'finally', 'for', 'from', 'global', 'if', 'import', 'in', 'is', 'lambda', 'nonlocal', 'not', 'or', 'pass', 'raise', 'return', 'try', 'while', 'with', 'yield']
```

### 3.3. 35个关键字详解
目前，在Python 3.14版本，共有35个关键字，本章节将概述这35个关键字的含义。

为了方便记忆和理解，我们可以按照功能将这些关键字分类。后续章节，我们将按照类型讲解这些关键字。

#### 3.3.1. 值关键字（3个）
这些关键字代表特定的值：
| 关键字 | 含义 | 示例 |
| :--- | :--- | :--- |
| `True` | 真（布尔值） | `is_valid = True` |
| `False` | 假（布尔值） | `is_error = False` |
| `None` | 空值/无对象 | `result = None` |

#### 3.3.2. 运算符关键字（5个）
用于逻辑和布尔运算：
| 关键字 | 含义 | 示例 |
| :--- | :--- | :--- |
| `and` | 逻辑与 | `if a > 0 and b > 0:` |
| `or` | 逻辑或 | `if x == 1 or x == 2:` |
| `not` | 逻辑非 | `if not is_finished:` |
| `is` | 对象标识对比 | `if x is None:` |
| `in` | 成员测试 | `if 'a' in 'apple':` |

#### 3.3.3. 控制流关键字（8个）
控制程序执行流程：

| 关键字 | 含义 | 示例 |

| :--- | :--- | :--- |
| `if` | 如果（条件开始） | `if score >= 60:` |
| `elif` | 否则如果（中间条件） | `elif score >= 90:` |
| `else` | 否则（默认分支） | `else: print("Fail")` |
| `for` | 遍历循环 | `for i in range(5):` |
| `while` | 条件循环 | `while count < 10:` |
| `break` | 跳出循环 | `if found: break` |
| `continue` | 跳过本次循环 | `if skip: continue` |
| `pass` | 空语句占位符 | `pass` |


> match 和 case 是 Python 3.10 引入的新特性，用于结构化模式匹配。

#### 3.3.4. 函数与类关键字（4个）
用于定义代码块的结构。
| 关键字 | 含义 | 示例 |
| :--- | :--- | :--- |
| `def` | 定义函数 | `def say_hello():` |
| `class` | 定义类 | `class Dog:` |
| `return` | 返回值 | `return x + y` |
| `lambda` | 匿名函数 | `f = lambda x: x*2` |

#### 3.3.5. 异常处理关键字（4个）
处理程序中的异常：

| 关键字 | 含义 | 示例 |
| :--- | :--- | :--- |
| `try` | 尝试执行代码 | `try: ...` |
| `except` | 捕获异常 | `except ValueError:` |
| `finally` | 最终执行（无论是否出错） | `finally: close_file()` |
| `raise` | 主动抛出异常 | `raise Exception("Error")` |


#### 3.3.6. 变量作用域（2个）
控制变量作用域：
| 关键字 | 含义 | 示例 |
| :--- | :--- | :--- |
| `global` | 声明全局变量 | `global count` |
| `nonlocal` | 声明非局部变量 | `nonlocal x` |

#### 3.3.7. 模块与导入（3个）
管理模块和导入：

| 关键字 | 含义 | 示例 |
| :--- | :--- | :--- |
| `import` | 导入模块 | `import math` |
| `from` | 从模块导入 | `from math import pi` |
| `as` | 起别名 | `import numpy as np` |

#### 3.3.8. 异步编程关键字（2个）
> Python 3.5新增

用于异步编程：
| 关键字 | 含义 | 示例 |
| :--- | :--- | :--- |
| `async` | 定义异步函数 | `async def fetch():` |
| `await` | 等待异步操作完成 | `await response` |

#### 3.3.9. 其他关键字（4个）
| 关键字 | 含义 | 示例 |
| :--- | :--- | :--- |
| `del`	| 删除引用	| `del my_list[0]`| 
| `with`	| 上下文管理	| `with open('file') as f:`| 
| `yield`	| 生成器返回值	| `yield x`| 
| `assert`	| 断言条件为真	| `assert x > 0`| 


## 4. 注释
Python 中的注释是用来解释代码、提高可读性或临时禁用代码行的文本。解释器在运行代码时会完全忽略注释部分。

Python 主要支持两种形式的注释：单行注释和多行注释（实际上是多行字符串）。

### 4.1. 单行注释
这是最常用的注释方式。使用`#`号开头，从`#`开始直到该行结束的所有内容都会被视为注释。

示例：
```python
## 这是一个独立的单行注释
print("Hello, World!")  # 这是行尾注释，解释这行代码的作用

## print("这行代码被注释掉了，不会执行")
```

最佳实践：
* 空格规范：# 后面通常要加一个空格，这样更易读（如 # 注释 而不是 #注释）。
* 用途：用于解释某一行或某一段代码的逻辑，或者临时调试（屏蔽代码）。

### 4.2. 多行注释（文档字符串 / 块注释）
严格来说，Python 没有专门的多行注释符号（像 C/Java 的 /* ... */ 那样）。但在实际使用中，我们可以用以下的方式来实现多行注释的效果：
* 多个`#`号。
* 使用`'''`。
* 使用`"""`。

示例：
```python
#!/usr/bin/python3
 
## 第一个注释
## 第二个注释
 
'''
第三注释
第四注释
'''
 
"""
第五注释
第六注释
"""
print ("Hello, Python!")
```


**多行`#` vs `"""`:**
| 特性 | 单行注释 (`#`) | 多行字符串 (`"""..."""`) |
| :--- | :--- | :--- |
| 语法本质 | 真正的注释语法 | 字符串字面量 (未被赋值时起到注释效果) |
| 主要用途 | 解释单行逻辑、临时禁用代码 | 函数/类文档 (Docstring)、大段说明 |
| 能否被 `help()` 读取 | 不能 | 能 (仅当位于定义首行时) |
| 性能影响 | 无 (解析阶段直接忽略) | 极微小 (理论上创建了一个未引用的对象) |
| 嵌套问题 | 不支持嵌套 | 支持在不同引号类型间嵌套 (如 `""" 包含 ' 单引号 """`) |

**`'''` vs """`：**
| 特性 | 三个单引号 (`'''`) | 三个双引号 (`"""`) |
| :--- | :--- | :--- |
| 功能 | 完全相同 | 完全相同 |
| 社区惯例 (Docstring) | 较少见 (PEP 257 推荐双引号) | 标准推荐 (绝大多数开源项目使用) |
| 最佳适用场景 | 注释内容中包含大量双引号 (`"`) 时 | 注释内容中包含大量单引号 (`'`) 时，或作为通用标准 |
| 嵌套风险 | 如果注释内容里也有 `'''`，会提前结束注释 | 如果注释内容里也有 `"""`，会提前结束注释 |


## 5. 行与缩进
在 Python 中，行（Lines）和缩进（Indentation）不仅仅是代码风格的问题，它们是语法的核心组成部分。

### 5.1. 物理行 (Physical Line)
物理行就是你代码编辑器中看到的每一行。

Python 通常一行写一条语句。行末不需要分号 ;（虽然写了也不报错，但不推荐）
```python
x = 10          # 这是一个物理行
y = 20          # 这是另一个物理行
```

### 5.2. 逻辑行（Logical Line）
逻辑行是 Python 解释器实际执行的一个完整语句。
* 一行一语句：大多数情况下，一个物理行就是一个逻辑行。
  ```python
  x = 10          # 一行一语句
  ```
* 多行一语句：如果一个语句太长，可以分成多个物理行来写。
  * 隐式换行（推荐）：在括号`()`、`[]`、`{}`内换行，不需要任何额外符号。
    ```python
    # 隐式换行 (推荐) - 括号内自动识别为同一逻辑行
    total = (1 + 2 + 3 + 
         4 + 5 + 6)
    ```
  * 显式换行：使用反斜杠`\`作为续行符。
    ```python
    # 显式换行 (不推荐，除非必要) - 使用 \
    long_string = "这是一段非常非常长的字符串，" \
                "需要分成两行来写。"
    ```
* 一行多语句：虽然不推荐（不符合 PEP 8 规范），但可以用分号 ; 分隔多条语句。
  ```python
  x = 1; y = 2; z = 3  # 不推荐，可读性差
  ```

### 5.3. 缩进（Python的灵魂）
与 C、Java、JavaScript 等语言使用大括号 `{}` 来划分代码块不同，Python 强制使用缩进来定义代码块的层级和逻辑结构。如果缩进错误，程序将直接报错（IndentationError）或运行出完全错误的结果。
> 缩进决定了代码的从属关系。

**核心规则：**
* 同一代码块必须严格对齐：属于同一个逻辑块（如 if 内部、for 循环内部、函数内部）的所有行，必须有完全相同的缩进量。
* 层级变化通过缩进体现：
  * 增加缩进：表示进入一个新的代码块（子级）。
  * 减少缩进：表示当前代码块结束，返回上一级。
* 禁止混用：绝对不能在同一文件中混用空格（Space）和制表符（Tab）进行缩进。
  * 现代编辑器（VS Code, PyCharm）默认会将 Tab 转换为 4 个空格，请保持这个设置。

**标准规范 (PEP 8)**
* 官方推荐使用 4 个空格 作为一个缩进层级。
* 不要使用 Tab 键（除非你的编辑器配置为“按 Tab 键自动插入 4 个空格”）。

缩进的空格数是可变的，但是同一个代码块的语句必须包含相同的缩进空格数。实例如下：
```python
if True:
    print ("True")
else:
    print ("False")
```


以下代码最后一行语句缩进数的空格数不一致，会导致运行错误：
```python
if True:
    print ("Answer")
    print ("True")
else:
    print ("Answer")
  print ("False")    # 缩进不一致，会导致运行错误
```
以上程序由于缩进不一致，执行后会出现类似以下错误：
```python
 File "test.py", line 6
    print ("False")    # 缩进不一致，会导致运行错误
                                      ^
IndentationError: unindent does not match any outer indentation level

```

### 5.4. 空行
函数之间或类的方法之间用空行分隔，表示一段新的代码的开始。类和函数入口之间也用一行空行分隔，以突出函数入口的开始。

空行与代码缩进不同，空行并不是 Python 语法的一部分。书写时不插入空行，Python 解释器运行也不会出错。但是空行的作用在于分隔两段不同功能或含义的代码，便于日后代码的维护或重构。

**记住：** 空行也是程序代码的一部分。

### 5.5. 总结
| 概念 | 规则 | 建议 |
| :--- | :--- | :--- |
| 行结束 | 不需要分号 `;` | 一行一语句，太长用括号换行 |
| 缩进符号 | 空格 或 Tab (不可混用) | 强制使用 4 个空格 |
| 代码块定义 | 靠缩进量决定，而非 `{}` | `if`, `for`, `def`, `class` 后必须缩进 |
| 对齐要求 | 同级代码必须严格左对齐 | 哪怕差一个空格也会报错 |
| 空行 | 允许存在 | 用于分隔逻辑段落，提高可读性 |

## 6. Python 运算符
Python 的运算符非常丰富，下面的章节将分类讲解这些运算符。
### 6.1. 算术运算符
| 运算符 | 名称 | 示例 (`a=10, b=3`) | 结果 | 说明 |
| :--- | :--- | :--- | :--- | :--- |
| `+` | 加 | `a + b` | `13` | 数字相加，或字符串拼接 |
| `-` | 减 | `a - b` | `7` | 数字相减 |
| `*` | 乘 | `a * b` | `30` | 数字相乘，或字符串重复 (`"Hi"*2` -> `"HiHi"`) |
| `/` | 真除法 | `a / b` | `3.333...` | 结果永远是浮点数 (float)，即使能整除 |
| `//` | 地板除 | `a // b` | `3` | 向下取整，结果是整数 (int) |
| `%` | 取模 | `a % b` | `1` | 返回除法的余数 |
| `` | 幂运算 | `a  b` | `1000` | 计算 $10^3 $ ，即 10 的 3 次方 |


### 6.2. 比较运算符
| 运算符 | 含义 | 示例 | 结果 |
| :--- | :--- | :--- | :--- |
| `==` | 等于 | `10 == 10` | `True` |
| `!=` | 不等于 | `10 != 5` | `True` |
| `>` | 大于 | `10 > 5` | `True` |
| `<` | 小于 | `10 < 5` | `False` |
| `>=` | 大于等于 | `10 >= 10` | `True` |
| `<=` | 小于等于 | `10 <= 5` | `False` |

> **🚨 常见错误：**
> * 千万不要用 = 进行比较！= 是赋值，== 才是判断相等。
> * 链式比较：Python 支持数学写法，如 1 < x < 10，这等价于 1 < x and x < 10，非常优雅。
### 6.3. 赋值运算符
| 运算符 | 等价写法 | 说明 |
| :--- | :--- | :--- |
| `=` | `x = 5` | 基本赋值 |
| `+=` | `x += 3` | `x = x + 3` |
| `-=` | `x -= 3` | `x = x - 3` |
| `*=` | `x *= 3` | `x = x * 3` |
| `/=` | `x /= 3` | `x = x / 3` |
| `//=` | `x //= 3` | `x = x // 3` |
| `%=` | `x %= 3` | `x = x % 3` |
| `**=` | `x **= a` | `x = x ** 3` |

在 Python 3.8 及更高版本中，引入了一种新的语法特性，称为"海象运算符"（Walrus Operator），它使用 `:= `符号。这个运算符的主要目的是在表达式中同时进行赋值和返回赋值的值。

使用海象运算符可以在一些情况下简化代码，尤其是在需要在表达式中使用赋值结果的情况下。这对于简化循环条件或表达式中的重复计算很有用。

下面是一个简单的实例，演示了海象运算符的使用：
```python
## 传统写法
n = 10
if n > 5:
    print(n)

## 使用海象运算符
if (n := 10) > 5:
    print(n)
```

海象运算符的优点：

* 海象运算符（:=）允许在表达式内部进行赋值，这可以减少代码的重复，提高代码的可读性和简洁性。
* 在上述例子中，传统写法需要单独一行来赋值 n，然后在 if 语句中进行条件检查。而使用海象运算符的写法可以在 if 语句中直接进行赋值和条件检查。

### 6.4. 逻辑运算符

| 运算符 | 含义 | 规则 | 示例 |
| :--- | :--- | :--- | :--- |
| `and` | 与 | 全真才真，一假即假 | `True and False` → `False` |
| `or` | 或 | 一真即真，全假才假 | `True or False` → `True` |
| `not` | 非 | 取反 | `not True` → `False` |


> 短路逻辑 (Short-circuit)：
> * A and B: 如果 A 为假，直接返回 A，不再计算 B。
> * A or B: 如果 A 为真，直接返回 A，不再计算 B。
> * 应用场景：user and user.name (如果 user 存在，才去访问 name，避免报错)。
### 6.5. 位运算符
直接对整数的二进制位进行操作（底层开发或算法题常用）。
假设 a = 60 (0011 1100), b = 13 (0000 1101)

| 运算符 | 名称 | 规则 | 示例结果 |
| :--- | :--- | :--- | :--- |
| `&` | 按位与 | 两位都为 1 时结果为 1 | `a & b` = 12 (`0000 1100`) |
| `\|` | 按位或 | 有一位为 1 时结果为 1 | `a \| b` = 61 (`0011 1101`) |
| `^` | 按位异或 | 两位不同为 1，相同为 0 | `a ^ b` = 49 (`0011 0001`) |
| `~` | 按位取反 | 0 变 1，1 变 0 (包括符号位) | `~a` = -61 |
| `<<` | 左移 | 向左移动指定位数，右边补 0 | `a << 2` = 240 (相当于 `*4`) |
| `>>` | 右移 | 向右移动指定位数，左边补符号位 | `a >> 2` = 15 (相当于 `//4`) |


### 6.6. 成员运算符
用于判断某个值是否存在于序列（字符串、列表、元组等）中。
| 运算符 | 含义 | 示例 | 结果 |
| :--- | :--- | :--- | :--- |
| `in` | 存在于 | `"p" in "Python"` | `True` |
| `not in` | 不存在于 | `"z" not in "Python"` | `True` |

### 6.7. 身份运算符
这是 Python 特有的重点！ 用于判断两个变量是否指向内存中的同一个对象，而不是判断值是否相等。

| 运算符 | 含义 | 对比 `==` |
| :--- | :--- | :--- |
| `is` | 是同一个对象 | `==` 比较值 (Value) |
| `is not` | 不是同一个对象 | `is` 比较内存地址 (ID) |

> 避坑指南：
> * 判断变量是否为 None 时，必须使用 is，不要用 ==。
> * ✅ 正确：if x is None:
> * ❌ 错误：if x == None: (虽然通常能工作，但不符合规范，且效率略低)
> * 对于小整数 (-5 到 256)，Python 有缓存机制，is 可能会返回 True，但不要依赖这个特性，比较数值永远用 ==。
### 6.8. 运算符优先级
当一行代码中有多个运算符时，执行顺序如下（记不住也没关系，多用括号）：
1. `()` (括号，优先级最高)
2. `**` (幂)
3. `~`, `+`, `-` (按位取反，正负号)
4. `*`, `/`, `//`, `%` (乘除取模)
5. `+`, `-` (加减)
6. `<<`, `>>` (位移)
7. `&` (按位与)
8. `^` (按位异或)
9. `|` (按位或)
10. `==`, `!=`, `>`, `<`, ... (比较)
11. `not` (逻辑非)
12. `and` (逻辑与)
13. `or` (逻辑或)
14. `=`, `+=`, ... (赋值，优先级最低)

> 建议：如果你不确定顺序，直接加括号 ()。
> 例如：3 + 4 * 5 是 23，但 (3 + 4) * 5 是 35。写清楚括号能让代码更易读。
## 7. Python 基本数据类型

Python 是一种动态类型语言，变量本身没有类型，只有变量所引用的对象有类型。Python 提供了丰富的内置数据类型，用于表示各种不同形式的数据。

### 7.1. 数字类型 (Numeric Types)

Python 支持多种数字类型，包括整数、浮点数、复数等。

#### 7.1.1. 整数 (int)

Python 3 中的整数类型可以表示任意大小的整数（只受内存限制），这与 Python 2 不同（Python 2 区分 int 和 long）。

```python
## 整数示例
age = 25
count = 0
negative = -10
large_num = 9999999999999999999999999  # Python 可以处理非常大的整数

## 不同进制的整数表示
decimal = 10        # 十进制
binary = 0b1010     # 二进制，等于 10
octal = 0o12        # 八进制，等于 10
hexadecimal = 0xA   # 十六进制，等于 10

## 整数的常用操作
print(10 + 5)       # 加法：15
print(10 - 5)       # 减法：5
print(10 * 5)       # 乘法：50
print(10 // 3)      # 整除：3
print(10 % 3)       # 取模：1
print(10 ** 2)      # 幂运算：100
print(abs(-10))     # 绝对值：10
print(pow(2, 3))    # 幂运算：8
```

#### 7.1.2. 浮点数 (float)

浮点数用于表示带小数的数值。Python 的浮点数遵循 IEEE 754 双精度标准（64 位）。

```python
## 浮点数示例
price = 19.99
pi = 3.14159
scientific = 1.23e-4  # 科学计数法，等于 0.000123
negative_float = -0.001

## 浮点数运算
print(10.5 + 2.3)     # 12.8
print(10.0 / 3)       # 3.3333333333333335
print(2.5 * 4)        # 10.0

## 注意事项：浮点数精度问题
print(0.1 + 0.2)      # 0.30000000000000004 (不是精确的 0.3)
print(0.1 + 0.2 == 0.3)  # False

## 解决精度问题的方法
from decimal import Decimal
print(Decimal('0.1') + Decimal('0.2'))  # 0.3
```

**⚠️ 浮点数精度警告：**
由于计算机使用二进制表示浮点数，某些十进制小数无法精确表示。在需要高精度的场景（如金融计算），建议使用 `decimal` 模块。

#### 7.1.3. 复数 (complex)

复数由实部和虚部组成，虚部用 `j` 或 `J` 表示。

```python
## 复数示例
z1 = 3 + 4j
z2 = complex(1, 2)  # 1 + 2j

## 复数操作
print(z1.real)      # 实部：3.0
print(z1.imag)      # 虚部：4.0
print(z1.conjugate())  # 共轭复数：(3-4j)
print(abs(z1))      # 模长：5.0
```

#### 7.1.4. 布尔类型 (bool)

布尔类型是整数的子类，只有两个值：`True` 和 `False`。

```python
## 布尔值
is_active = True
is_error = False

## 布尔运算
print(True and False)   # False
print(True or False)    # True
print(not True)         # False

## 布尔值与整数的关系
print(int(True))        # 1
print(int(False))       # 0
print(True + 1)         # 2
print(False * 10)       # 0

## 真值测试 (Truth Value Testing)
## 以下值在布尔上下文中被视为 False：
## - None, False
## - 零值：0, 0.0, 0j, Decimal(0), Fraction(0, 1)
## - 空序列和集合：'', (), [], {}, set(), range(0)
## 其他值都为 True

if "hello":  # 非空字符串为 True
    print("This will print")

if []:  # 空列表为 False
    print("This won't print")
```

### 7.2. 序列类型 (Sequence Types)

序列类型是可以按索引访问元素的有序集合。

#### 7.2.1. 字符串 (str)

字符串是不可变的 Unicode 字符序列。

```python
## 字符串创建
name = "Alice"
message = 'Hello, World!'
multiline = """这是一个
多行字符串"""
empty = ""

## 字符串特性：不可变
## name[0] = 'B'  # 错误！不能修改字符串

## 字符串操作
text = "Python Programming"
print(len(text))          # 长度：18
print(text[0])            # 索引：'P'
print(text[-1])           # 负索引：'g'
print(text[0:6])          # 切片：'Python'
print(text[7:])           # 切片：'Programming'
print(text[::-1])         # 反转：'gnimmargorP nohtyP'
print(text.lower())       # 小写：'python programming'
print(text.upper())       # 大写：'PYTHON PROGRAMMING'
print(text.split())       # 分割：['Python', 'Programming']
print("Py" in text)       # 成员测试：True
print(text.find("Pro"))   # 查找：7
print(text.replace("Python", "Java"))  # 替换

## 字符串格式化
name = "Alice"
age = 25
print(f"My name is {name} and I am {age} years old.")  # f-string (推荐)
print("My name is {} and I am {} years old.".format(name, age))
print("My name is %s and I am %d years old." % (name, age))

## 转义字符
path = "C:\\Users\\Alice"  # 反斜杠
quote = 'He said, "Hello!"'  # 双引号
newline = "Line 1\nLine 2"  # 换行
raw_string = r"C:\Users\Alice"  # 原始字符串，不转义
```

#### 7.2.2. 列表 (list)

列表是可变的有序元素序列，可以包含不同类型的元素。

```python
## 列表创建
numbers = [1, 2, 3, 4, 5]
mixed = [1, "hello", 3.14, True]
nested = [[1, 2], [3, 4], [5, 6]]  # 嵌套列表
empty_list = []
list_from_range = list(range(5))  # [0, 1, 2, 3, 4]

## 列表特性：可变、有序、允许重复
numbers[0] = 10  # 修改元素
numbers.append(6)  # 添加元素

## 列表操作
nums = [1, 2, 3, 4, 5]
print(len(nums))        # 长度：5
print(nums[2])          # 索引：3
print(nums[-1])         # 负索引：5
print(nums[1:4])        # 切片：[2, 3, 4]
print(nums + [6, 7])    # 拼接：[1, 2, 3, 4, 5, 6, 7]
print(nums * 2)         # 重复：[1, 2, 3, 4, 5, 1, 2, 3, 4, 5]

## 列表方法
nums.append(6)          # 末尾添加：[1, 2, 3, 4, 5, 6]
nums.insert(0, 0)       # 插入：[0, 1, 2, 3, 4, 5, 6]
nums.extend([7, 8])     # 扩展：[0, 1, 2, 3, 4, 5, 6, 7, 8]
nums.remove(0)          # 删除第一个匹配的元素
popped = nums.pop()     # 弹出末尾元素：8
index = nums.index(3)   # 查找索引：3
count = nums.count(3)   # 计数：1
nums.sort()             # 排序
nums.reverse()          # 反转
nums.clear()            # 清空

## 列表推导式 (List Comprehension)
squares = [x**2 for x in range(10)]  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
evens = [x for x in range(10) if x % 2 == 0]  # [0, 2, 4, 6, 8]
matrix = [[i*j for j in range(3)] for i in range(3)]
## [[0, 0, 0], [0, 1, 2], [0, 2, 4]]
```

#### 7.2.3. 元组 (tuple)

元组是不可变的有序元素序列，通常用于保护数据不被修改。

```python
## 元组创建
coordinates = (10, 20)
colors = ("red", "green", "blue")
single_item = (42,)  # 单元素元组需要逗号
empty_tuple = ()
mixed = (1, "hello", 3.14)

## 元组特性：不可变、有序、允许重复
## coordinates[0] = 15  # 错误！不能修改元组

## 元组操作
tup = (1, 2, 3, 4, 5)
print(len(tup))         # 长度：5
print(tup[2])           # 索引：3
print(tup[-1])          # 负索引：5
print(tup[1:4])         # 切片：(2, 3, 4)
print(tup + (6, 7))     # 拼接：(1, 2, 3, 4, 5, 6, 7)
print(tup * 2)          # 重复：(1, 2, 3, 4, 5, 1, 2, 3, 4, 5)

## 元组方法
print(tup.count(3))     # 计数：1
print(tup.index(4))     # 查找索引：3

## 元组解包 (Tuple Unpacking)
x, y = coordinates
print(x, y)  # 10, 20

a, *b, c = (1, 2, 3, 4, 5)
print(a, b, c)  # 1, [2, 3, 4], 5

## 元组的用途
## 1. 函数返回多个值
def get_point():
    return (10, 20)

x, y = get_point()

## 2. 字典的键（因为元组是不可变的）
location = {(0, 0): "origin", (1, 1): "diagonal"}

## 3. 数据库记录
user_record = (1, "Alice", "alice@example.com")
```

#### 7.2.4. 范围 (range)

range 类型表示不可变的数字序列，通常用于循环指定次数。

```python
## range 创建
r1 = range(5)           # 0, 1, 2, 3, 4
r2 = range(2, 7)        # 2, 3, 4, 5, 6
r3 = range(0, 10, 2)    # 0, 2, 4, 6, 8 (步长为 2)
r4 = range(10, 0, -1)   # 10, 9, 8, ..., 1 (递减)

## range 特性：节省内存，只在需要时计算值
print(list(r1))         # [0, 1, 2, 3, 4]
print(r1.start)         # 起始值：0
print(r1.stop)          # 结束值：5
print(r1.step)          # 步长：1

## 常用于循环
for i in range(5):
    print(i)  # 输出 0 到 4
```

### 7.3. 映射类型 (Mapping Type)

#### 7.3.1. 字典 (dict)

字典是无序的键值对集合（Python 3.7+ 保持插入顺序）。

```python
## 字典创建
person = {
    "name": "Alice",
    "age": 25,
    "city": "New York"
}
empty_dict = {}
dict_from_pairs = dict([("name", "Bob"), ("age", 30)])
nested_dict = {
    "user1": {"name": "Alice", "age": 25},
    "user2": {"name": "Bob", "age": 30}
}

## 字典操作
print(person["name"])       # 访问：'Alice'
print(person.get("age"))    # 安全访问：25
print(person.get("email", "not provided"))  # 默认值

## 修改和添加
person["age"] = 26          # 修改
person["email"] = "alice@example.com"  # 添加新键值对

## 删除
del person["city"]          # 删除键
age = person.pop("age")     # 弹出并返回值
last_item = person.popitem()  # 弹出最后一个插入的项

## 字典方法
keys = person.keys()        # 所有键
values = person.values()    # 所有值
items = person.items()      # 所有键值对
person.update({"phone": "123456"})  # 更新
person.clear()              # 清空

## 遍历字典
for key in person:
    print(key, person[key])

for key, value in person.items():
    print(f"{key}: {value}")

## 字典推导式
squares = {x: x**2 for x in range(5)}  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
even_squares = {x: x**2 for x in range(10) if x % 2 == 0}

## 字典合并 (Python 3.9+)
dict1 = {"a": 1, "b": 2}
dict2 = {"c": 3, "d": 4}
merged = dict1 | dict2  # {'a': 1, 'b': 2, 'c': 3, 'd': 4}
```

### 7.4. 集合类型 (Set Types)

#### 7.4.1. 集合 (set)

集合是无序的不重复元素集。

```python
## 集合创建
fruits = {"apple", "banana", "cherry"}
empty_set = set()  # 注意：不能用 {}，那是空字典
numbers = set([1, 2, 2, 3, 3, 3])  # {1, 2, 3}，自动去重

## 集合特性：无序、不重复、可变
## print(fruits[0])  # 错误！集合不支持索引

## 集合操作
fruits.add("orange")        # 添加元素
fruits.remove("banana")     # 删除元素（不存在会报错）
fruits.discard("grape")     # 删除元素（不存在不报错）
popped = fruits.pop()       # 随机弹出一个元素

## 集合运算
set1 = {1, 2, 3, 4, 5}
set2 = {4, 5, 6, 7, 8}

print(set1 | set2)  # 并集：{1, 2, 3, 4, 5, 6, 7, 8}
print(set1 & set2)  # 交集：{4, 5}
print(set1 - set2)  # 差集：{1, 2, 3}
print(set1 ^ set2)  # 对称差集：{1, 2, 3, 6, 7, 8}

## 集合关系
print(set1.issubset(set2))     # 子集：False
print(set1.issuperset(set2))   # 超集：False
print(set1.isdisjoint(set2))   # 是否无交集：False

## 集合推导式
unique_squares = {x**2 for x in [1, 1, 2, 2, 3]}  # {1, 4, 9}
```

#### 7.4.2. 冻结集合 (frozenset)

frozenset 是不可变的集合。

```python
## 冻结集合创建
frozen = frozenset([1, 2, 3, 4, 5])
empty_frozen = frozenset()

## 冻结集合特性：不可变、可哈希（可以作为字典的键）
## frozen.add(6)  # 错误！不能修改

## 作为字典的键
dict_with_frozen = {frozen: "immutable set"}
```

### 7.5. 特殊类型

#### 7.5.1. NoneType

NoneType 只有一个值：None，表示空值或无。

```python
## None 的使用
result = None

def no_return():
    pass

print(no_return())  # None

## 检查是否为 None
if result is None:
    print("Result is None")

## 默认参数值
def greet(name=None):
    if name is None:
        name = "Guest"
    return f"Hello, {name}!"
```

#### 7.5.2. 字节类型 (bytes, bytearray, memoryview)

用于处理二进制数据。

```python
## bytes (不可变)
b = b"hello"
byte_data = bytes([65, 66, 67])  # b'ABC'

## bytearray (可变)
ba = bytearray(b"hello")
ba[0] = 72  # 修改

## memoryview (内存视图，用于高效处理大二进制数据)
mv = memoryview(b"hello world")
sub_mv = mv[0:5]  # 不复制数据，只是视图
```

### 7.6. 类型转换

Python 提供了内置函数进行类型转换：

```python
## 转换为整数
int("123")        # 123
int(3.14)         # 3 (截断，不是四舍五入)
int("10", 2)      # 2 (二进制转十进制)

## 转换为浮点数
float("3.14")     # 3.14
float(10)         # 10.0

## 转换为字符串
str(123)          # "123"
str(3.14)         # "3.14"
str([1, 2, 3])    # "[1, 2, 3]"

## 转换为列表
list("hello")     # ['h', 'e', 'l', 'l', 'o']
list((1, 2, 3))   # [1, 2, 3]
list({1, 2, 3})   # [1, 2, 3]

## 转换为元组
tuple([1, 2, 3])  # (1, 2, 3)

## 转换为集合
set([1, 2, 2, 3])  # {1, 2, 3}

## 转换为字典
dict([("a", 1), ("b", 2)])  # {'a': 1, 'b': 2}

## 类型检查
type(123)         # <class 'int'>
isinstance(123, int)  # True
```

### 7.7. 可变与不可变类型

理解可变性对于编写正确的 Python 代码至关重要。

**不可变类型 (Immutable):**
- 数字 (int, float, complex)
- 字符串 (str)
- 元组 (tuple)
- 冻结集合 (frozenset)
- 字节 (bytes)

**可变类型 (Mutable):**
- 列表 (list)
- 字典 (dict)
- 集合 (set)
- 字节数组 (bytearray)

```python
## 不可变示例
x = 10
y = x
x = 20
print(y)  # 10 (y 不受影响)

## 可变示例
list1 = [1, 2, 3]
list2 = list1
list1.append(4)
print(list2)  # [1, 2, 3, 4] (list2 也受影响！)

## 解决方案：创建副本
list3 = list1.copy()  # 或 list(list1), list1[:]
```

### 7.8. 数据类型总结表

| 类型 | 可变性 | 有序 | 可重复 | 示例 |
| :--- | :--- | :--- | :--- | :--- |
| int | 不可变 | N/A | N/A | `42` |
| float | 不可变 | N/A | N/A | `3.14` |
| str | 不可变 | ✓ | ✓ | `"hello"` |
| list | ✓ | ✓ | ✓ | `[1, 2, 3]` |
| tuple | 不可变 | ✓ | ✓ | `(1, 2, 3)` |
| dict | ✓ | ✓* | N/A | `{"a": 1}` |
| set | ✓ | ✗ | ✗ | `{1, 2, 3}` |
| frozenset | 不可变 | ✗ | ✗ | `frozenset([1,2,3])` |
| range | 不可变 | ✓ | ✓ | `range(5)` |

*注：Python 3.7+ 字典保持插入顺序

## 8. 常量与变量

### 8.1. 变量 (Variables)

#### 8.1.1. 变量的概念

在 Python 中，变量是对对象的引用（标签），而不是存储数据的容器。变量本身没有类型，只有它引用的对象有类型。

```python
## 变量赋值
x = 10          # x 引用整数对象 10
name = "Alice"  # name 引用字符串对象 "Alice"

## 变量可以重新绑定到不同类型的对象
x = 10
x = "hello"     # 合法！x 现在引用字符串
x = [1, 2, 3]   # 合法！x 现在引用列表
```

#### 8.1.2. 变量命名规则

详见第 2 章标识符部分，这里再次强调：

**必须遵守的规则：**
- 第一个字符必须是字母或下划线 `_`
- 不能以数字开头
- 只能包含字母、数字和下划线
- 不能使用 Python 关键字
- 区分大小写

```python
## 正确示例
user_name = "Alice"
_total = 100
MAX_SIZE = 1024
_private = "internal"

## 错误示例
## 2nd_place = "silver"  # 以数字开头
## user-name = "Bob"     # 包含连字符
## class = "Math"        # 使用关键字
```

#### 8.1.3. 变量赋值

Python 支持多种赋值方式：

```python
## 基本赋值
x = 10
y = 20

## 链式赋值
a = b = c = 0  # a, b, c 都引用同一个对象 0

## 多重赋值（元组解包）
x, y = 10, 20
print(x, y)  # 10, 20

## 交换变量值
a, b = b, a  # Python 特有的优雅语法

## 解包序列
first, *middle, last = [1, 2, 3, 4, 5]
print(first)   # 1
print(middle)  # [2, 3, 4]
print(last)    # 5

## 增强赋值运算符
x = 10
x += 5   # x = x + 5
x -= 3   # x = x - 3
x *= 2   # x = x * 2
x /= 4   # x = x / 4
x //= 2  # x = x // 2
x %= 3   # x = x % 3
x **= 2  # x = x ** 2

## 海象运算符 (Python 3.8+)
if (n := len([1, 2, 3, 4])) > 3:
    print(f"列表长度为 {n}")
```

#### 8.1.4. 变量作用域

变量的作用域决定了在哪里可以访问该变量。

```python
## 局部变量 (Local Scope)
def my_function():
    local_var = 10  # 局部变量，只在函数内部有效
    print(local_var)

my_function()
## print(local_var)  # 错误！NameError

## 全局变量 (Global Scope)
global_var = 100  # 全局变量

def access_global():
    print(global_var)  # 可以访问

access_global()  # 100

## 修改全局变量
counter = 0

def increment():
    global counter  # 声明使用全局变量
    counter += 1

increment()
print(counter)  # 1

## 嵌套作用域 (Enclosing Scope)
def outer():
    x = 10
    def inner():
        nonlocal x  # 声明使用外层函数的变量
        x += 1
        print(x)
    inner()  # 11
    print(x)  # 11

outer()
```

**作用域查找顺序 (LEGB 规则):**
1. **L**ocal - 当前函数的局部作用域
2. **E**nclosing - 外层函数的作用域
3. **G**lobal - 当前模块的全局作用域
4. **B**uilt-in - Python 内置作用域

#### 8.1.5. 变量类型提示 (Type Hints)

Python 3.5+ 支持类型注解，虽然不影响运行，但有助于代码理解和静态分析。

```python
## 变量类型提示
age: int = 25
name: str = "Alice"
prices: list[float] = [19.99, 29.99]
is_valid: bool = True

## 函数类型提示
def greet(name: str, age: int) -> str:
    return f"Hello, {name}! You are {age} years old."

## 可选类型
from typing import Optional
maybe_name: Optional[str] = None  # 可以是 str 或 None

## 联合类型
from typing import Union
value: Union[int, str] = 10  # 可以是 int 或 str
```

### 8.2. 常量 (Constants)

#### 8.2.1. 常量的概念

常量是指在程序运行过程中不应该被改变的值。然而，Python 没有真正的常量机制（不像 C/C++ 的 const）。

Python 中的常量是通过命名约定来实现的：

```python
## 常量命名约定：全大写字母 + 下划线
MAX_CONNECTIONS = 100
PI = 3.14159
API_KEY = "secret_key_123"
DEFAULT_TIMEOUT = 30  # 秒
GRAVITY = 9.8  # m/s²
```

#### 8.2.2. 常量的特性

**Python 常量实际上是可变的：**

```python
## 定义"常量"
MAX_SIZE = 100

## 技术上可以修改（但不应该这样做）
MAX_SIZE = 200  # 不会报错，但违反了约定

## 不可变对象作为常量
TUPLE_COORDINATES = (10, 20)  # 元组本身不可变
## TUPLE_COORDINATES[0] = 15  # 错误！

## 可变对象作为常量（危险！）
DEFAULT_CONFIG = {"debug": False, "timeout": 30}
## 可以被修改（应该避免）
DEFAULT_CONFIG["debug"] = True  # 不推荐！
```

#### 8.2.3. 定义常量的最佳实践

**1. 模块级常量**
```python
## config.py
MAX_RETRIES = 3
TIMEOUT = 30
API_VERSION = "v1"
```

**2. 使用类来组织常量**
```python
class Config:
    DEBUG = True
    DATABASE_URL = "sqlite:///db.sqlite"
    SECRET_KEY = "your-secret-key"
    
class StatusCodes:
    OK = 200
    NOT_FOUND = 404
    SERVER_ERROR = 500
```

**3. 使用枚举 (Enum)**
```python
from enum import Enum

class Color(Enum):
    RED = "red"
    GREEN = "green"
    BLUE = "blue"

print(Color.RED.value)  # "red"
```

**4. 使用命名元组 (Named Tuple)**
```python
from collections import namedtuple

Constants = namedtuple('Constants', ['PI', 'E', 'G'])
MATH_CONSTANTS = Constants(PI=3.14159, E=2.71828, G=6.67430e-11)

print(MATH_CONSTANTS.PI)  # 3.14159
```

#### 8.2.4. 常量 vs 变量

| 特性 | 变量 | 常量 |
| :--- | :--- | :--- |
| 命名规范 | snake_case (小写 + 下划线) | UPPER_CASE (全大写 + 下划线) |
| 可变性 | 可以随时修改 | 不应该修改（约定） |
| 作用域 | 局部、全局、嵌套 | 通常是模块级或类级 |
| 类型检查 | 动态类型 | 动态类型（但建议类型注解） |
| 用途 | 存储变化的数据 | 存储固定配置、魔法数字 |

#### 8.2.5. 实际应用场景

**1. 配置文件中的常量**
```python
## settings.py
DATABASE_HOST = "localhost"
DATABASE_PORT = 5432
DATABASE_NAME = "myapp"

REDIS_HOST = "127.0.0.1"
REDIS_PORT = 6379

LOG_LEVEL = "INFO"
DEBUG_MODE = False
```

**2. 数学和科学计算常量**
```python
import math

print(math.pi)        # 圆周率
print(math.e)         # 自然常数
print(math.inf)       # 无穷大
print(math.nan)       # NaN
```

**3. 状态常量**
```python
## 订单状态
ORDER_STATUS_PENDING = "pending"
ORDER_STATUS_PAID = "paid"
ORDER_STATUS_SHIPPED = "shipped"
ORDER_STATUS_COMPLETED = "completed"
ORDER_STATUS_CANCELLED = "cancelled"

## 使用枚举更好
from enum import Enum

class OrderStatus(Enum):
    PENDING = "pending"
    PAID = "paid"
    SHIPPED = "shipped"
    COMPLETED = "completed"
    CANCELLED = "cancelled"
```

### 8.3. 变量和常量的内存管理

#### 8.3.1. 引用计数

Python 使用引用计数来管理内存：

```python
import sys

x = 10
print(sys.getrefcount(x))  # 显示引用计数（实际值可能大于预期）

y = x  # 引用计数 +1
z = x  # 引用计数再 +1

del y  # 引用计数 -1
```

#### 8.3.2. 小整数缓存

Python 会缓存小整数（通常是 -5 到 256），这些整数在内存中只有一个实例：

```python
a = 256
b = 256
print(a is b)  # True (同一个对象)

c = 257
d = 257
print(c is d)  # False (不同对象，但在某些实现中可能是 True)
```

#### 8.3.3. 字符串驻留

Python 会驻留（intern）某些字符串以节省内存：

```python
a = "hello"
b = "hello"
print(a is b)  # True (同一个对象)

c = "hello world"
d = "hello world"
print(c is d)  # 可能是 False（取决于实现）
```

### 8.4. 最佳实践

#### 8.4.1. 变量使用建议

1. **见名知意**：使用描述性的名称
   ```python
   # 好
   user_age = 25
   total_price = 100.50
   
   # 不好
   a = 25
   x = 100.50
   ```

2. **避免单字母变量**（除非在短循环中）
   ```python
   # 好的短循环
   for i in range(10):
       print(i)
   
   # 不好的长函数
   def process_data(d):
       # d 是什么？不清楚！
       pass
   ```

3. **使用类型注解**
   ```python
   def calculate_total(prices: list[float]) -> float:
       return sum(prices)
   ```

4. **及时清理不再使用的变量**
   ```python
   sensitive_data = "password123"
   # 使用后删除
   del sensitive_data
   ```

#### 8.4.2. 常量使用建议

1. **避免魔法数字**
   ```python
   # 不好
   if score >= 60:
       print("及格")
   
   # 好
   PASSING_SCORE = 60
   if score >= PASSING_SCORE:
       print("及格")
   ```

2. **集中管理常量**
   ```python
   # constants.py
   MAX_USERS = 1000
   MIN_PASSWORD_LENGTH = 8
   API_ENDPOINT = "https://api.example.com"
   ```

3. **使用枚举代替相关常量**
   ```python
   from enum import Enum
   
   class Weekday(Enum):
       MONDAY = 1
       TUESDAY = 2
       WEDNESDAY = 3
       THURSDAY = 4
       FRIDAY = 5
   ```

4. **文档化常量**
   ```python
   # 最大重试次数（超过此值后会向用户报告错误）
   MAX_RETRIES = 3
   
   # API 请求超时时间（秒）
   REQUEST_TIMEOUT = 30
   ```

### 8.5. 常见错误和陷阱

#### 8.5.1. 可变默认参数

```python
## 错误示例
def add_item(item, items=[]):  # 危险！
    items.append(item)
    return items

## 第一次调用
print(add_item(1))  # [1]
## 第二次调用
print(add_item(2))  # [1, 2] (!!)

## 正确做法
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

#### 8.5.2. 全局变量滥用

```python
## 不推荐
counter = 0

def increment():
    global counter
    counter += 1

## 更好的做法：使用类
class Counter:
    def __init__(self):
        self.count = 0
    
    def increment(self):
        self.count += 1
```

#### 8.5.3. 混淆可变和不可变类型

```python
## 列表作为参数
def modify_list(lst):
    lst.append(4)

my_list = [1, 2, 3]
modify_list(my_list)
print(my_list)  # [1, 2, 3, 4] (原列表被修改！)

## 如果不想修改原列表
def modify_list_safe(lst):
    new_list = lst.copy()
    new_list.append(4)
    return new_list
```

#### 8.5.4. 错误地使用常量

```python
## 常量被意外修改
CONFIG = {"debug": False}
CONFIG["debug"] = True  # 违反了常量不应该修改的原则

## 使用 frozendict 或其他方式保护
from types import MappingProxyType
_CONFIG = {"debug": False}
CONFIG = MappingProxyType(_CONFIG)  # 只读视图
## CONFIG["debug"] = True  # 会报错
```

### 8.6. 总结

**变量要点：**
- 变量是对对象的引用，不是容器
- 遵循命名规范（snake_case）
- 理解作用域规则（LEGB）
- 使用类型注解提高可读性

**常量要点：**
- Python 没有真正的常量，靠约定维护
- 使用全大写命名（UPPER_CASE）
- 避免魔法数字
- 考虑使用枚举组织相关常量

**最佳实践：**
- 选择有意义的名称
- 保持代码一致性
- 文档化重要变量和常量
- 谨慎使用全局变量

## 9. Python 源代码文件基本结构

Python 源代码文件（.py 文件）有着相对固定和规范的结构。遵循良好的文件组织结构可以提高代码的可读性、可维护性和可复用性。

### 9.1. Python 源文件的标准组成部分

一个规范的 Python 模块文件通常包含以下部分（按顺序）：

```python
#!/usr/bin/env python3
## -*- coding: utf-8 -*-

"""
模块文档字符串（Docstring）

这里是模块的详细描述，说明这个文件的用途、功能、作者等信息。
可以包含多行，使用三个双引号包裹。
"""

## 1. 导入标准库模块
import os
import sys
from pathlib import Path
from typing import List, Dict, Optional

## 2. 导入第三方库
import requests
from flask import Flask, request

## 3. 导入本地模块
from .utils import helper_function
from .models import User, Product
from config import Config

## 4. 模块级常量
__version__ = "1.0.0"
__author__ = "Your Name"
MAX_CONNECTIONS = 100
DEFAULT_TIMEOUT = 30

## 5. 模块级变量（如果有）
_cache: Dict[str, any] = {}
_initialized: bool = False

## 6. 类定义
class MyClass:
    """类的文档字符串"""
    
    def __init__(self):
        """初始化方法"""
        pass
    
    def public_method(self):
        """公共方法"""
        pass
    
    def _private_method(self):
        """私有方法（约定俗成）"""
        pass

## 7. 函数定义
def function_one(param1: str, param2: int = 10) -> bool:
    """
    函数的文档字符串
    
    Args:
        param1: 参数 1 的描述
        param2: 参数 2 的描述，默认值为 10
    
    Returns:
        返回值的描述
    
    Raises:
        ValueError: 什么情况下会抛出
    """
    return True

def function_two():
    """另一个函数的文档字符串"""
    pass

## 8. 主程序入口
if __name__ == "__main__":
    # 测试代码或命令行入口
    print(f"Version: {__version__}")
    result = function_one("test", 20)
    print(f"Result: {result}")
```

### 9.2. 各部分详细说明

#### 9.2.1. Shebang 行（仅 Unix/Linux/Mac）

```python
#!/usr/bin/env python3
```

**作用：**
- 在 Unix/Linux/Mac 系统上，告诉操作系统用哪个解释器执行此脚本
- Windows 系统通常忽略此行

**常见写法：**
```python
#!/usr/bin/env python3      # 推荐：从环境变量查找 python3
#!/usr/bin/python3         # 指定路径，不够灵活
#!/usr/bin/env python      # 可能指向 Python 2（不推荐）
```

**何时需要：**
- ✅ 需要在 Linux/Mac 上直接执行的脚本（如 `./script.py`）
- ❌ 普通模块文件（被导入的文件）不需要

#### 9.2.2. 编码声明

```python
## -*- coding: utf-8 -*-
```

**作用：**
- 指定源文件的字符编码
- Python 3 默认使用 UTF-8，通常不需要显式声明
- 如果文件中包含非 ASCII 字符（如中文注释），建议保留

**现代实践：**
- Python 3.8+ 默认 UTF-8，可以不写
- 但保留此声明是良好的兼容性实践
- 如果使用其他编码（如 GBK），必须声明

#### 9.2.3. 模块文档字符串（Module Docstring）

```python
"""
这是一个用户管理模块

提供用户创建、查询、更新、删除等功能。
支持本地存储和数据库存储两种方式。

示例用法:
    >>> from user_module import User
    >>> user = User.create("Alice", 25)
    >>> print(user.name)
    Alice

作者: 张三
日期: 2024-01-15
版本: 1.0.0
"""
```

**规范：**
- 必须是文件的第一条语句（在 shebang 和编码声明之后）
- 使用三个双引号 `"""`
- 第一行简明扼要，空行后详细描述
- 可以包含示例、作者、版本等信息

#### 9.2.4. 导入语句（Imports）

**导入分组顺序（PEP 8 规范）：**

```python
## 第一组：标准库（按字母排序）
import asyncio
import json
import os
import sys
from collections import defaultdict
from typing import List, Dict

## 第二组：第三方库（按字母排序）
import numpy as np
import pandas as pd
import requests
from flask import Flask

## 第三组：本地模块/项目内导入
from . import utils
from .models import User
from config import settings
from myapp.services import UserService
```

**分组规则：**
1. 每组之间用一空行分隔
2. 组内按字母顺序排列
3. 标准库优先，第三方库次之，本地模块最后

**导入最佳实践：**

```python
## ✅ 推荐：清晰明确的导入
from datetime import datetime, timedelta
import requests

## ✅ 推荐：避免命名冲突
import numpy as np
import pandas as pd

## ⚠️ 谨慎：通配符导入（污染命名空间）
from module import *  # 不推荐，除非明确需要

## ❌ 避免：循环导入
## file_a.py: from file_b import func_b
## file_b.py: from file_a import func_a
## 解决方案：在函数内部导入或重构代码
```

#### 9.2.5. 模块级常量和变量

**模块元信息：**
```python
__version__ = "1.0.0"
__author__ = "Your Name"
__email__ = "your.email@example.com"
__license__ = "MIT"
__copyright__ = "Copyright 2024, Your Name"
```

**业务常量：**
```python
## 配置相关
MAX_RETRIES = 3
TIMEOUT_SECONDS = 30
API_BASE_URL = "https://api.example.com"

## 业务规则
MIN_AGE = 18
MAX_FILE_SIZE_MB = 10
ALLOWED_EXTENSIONS = {'.jpg', '.png', '.gif'}
```

**模块状态变量：**
```python
## 缓存
_cache: Dict[str, any] = {}

## 初始化标志
_initialized: bool = False

## 计数器
_request_count: int = 0
```

**命名注意：**
- 常量：全大写 + 下划线
- 模块私有变量：单下划线开头 `_var`
- 类型注解：建议使用，提高可读性

#### 9.2.6. 类定义

**标准类结构：**
```python
class ClassName:
    """类的文档字符串"""
    
    # 类属性
    class_var = "shared by all instances"
    _private_class_var = "internal use"
    
    def __init__(self, param1: str, param2: int = 0):
        """
        初始化方法
        
        Args:
            param1: 描述
            param2: 描述
        """
        # 实例属性
        self.param1 = param1
        self.param2 = param2
        self._internal_state = None  # 私有属性（约定）
    
    def public_method(self, arg: str) -> bool:
        """
        公共方法的文档字符串
        
        Args:
            arg: 参数描述
        
        Returns:
            返回值描述
        """
        return True
    
    def _private_method(self):
        """
        私有方法（约定俗成，不强制）
        
        不应该在类外部直接调用
        """
        pass
    
    @classmethod
    def class_method(cls):
        """类方法"""
        return cls.class_var
    
    @staticmethod
    def static_method(x, y):
        """静态方法"""
        return x + y
    
    @property
    def computed_property(self):
        """属性装饰器"""
        return self.param1.upper()
    
    def __str__(self):
        """字符串表示"""
        return f"ClassName({self.param1})"
    
    def __repr__(self):
        """开发者友好的表示"""
        return f"ClassName('{self.param1}', {self.param2})"
```

**类组织顺序：**
1. 类文档字符串
2. 类属性
3. `__init__` 方法
4. 其他公共方法
5. 私有方法
6. 类方法、静态方法
7. 属性装饰器
8. 魔术方法（`__str__`, `__repr__` 等）

#### 9.2.7. 函数定义

**函数结构：**
```python
def function_name(param1: type, param2: type = default) -> return_type:
    """
    函数的文档字符串（遵循 Google 或 NumPy 风格）
    
    Args:
        param1: 参数 1 的描述
        param2: 参数 2 的描述，默认值为 xxx
    
    Returns:
        返回值的描述
    
    Raises:
        ValueError: 当参数不合法时
        TypeError: 当参数类型错误时
    
    Examples:
        >>> function_name("test", 10)
        True
        
    See Also:
        related_function: 相关函数说明
    """
    # 函数体
    # 1. 参数验证
    if not param1:
        raise ValueError("param1 cannot be empty")
    
    # 2. 主要逻辑
    result = some_operation(param1, param2)
    
    # 3. 返回结果
    return result
```

**函数组织原则：**
- 单一职责：一个函数只做一件事
- 长度控制：尽量不超过 50 行
- 参数数量：不超过 5 个参数（过多考虑封装）
- 副作用最小化：尽量避免修改全局状态

#### 9.2.8. 主程序入口

**测试入口：**
```python
if __name__ == "__main__":
    # 简单测试
    print("Running tests...")
    assert function_one("test") == True
    print("All tests passed!")
```

**命令行工具入口：**
```python
if __name__ == "__main__":
    import argparse
    
    parser = argparse.ArgumentParser(description="工具描述")
    parser.add_argument("-v", "--version", action="store_true", help="显示版本")
    parser.add_argument("-f", "--file", type=str, required=True, help="输入文件")
    parser.add_argument("-o", "--output", type=str, default="output.txt", help="输出文件")
    
    args = parser.parse_args()
    
    if args.version:
        print(f"Version: {__version__}")
        sys.exit(0)
    
    # 主逻辑
    result = process_file(args.file)
    save_result(result, args.output)
    print(f"Done! Output saved to {args.output}")
```

**为什么需要 `if __name__ == "__main__"`？**
```python
## module.py
def hello():
    print("Hello from module!")

print("This runs on import")

if __name__ == "__main__":
    print("This only runs when executed directly")
    hello()

## 场景 1: 直接运行 python module.py
## 输出:
## This runs on import
## This only runs when executed directly
## Hello from module!

## 场景 2: 被导入 from module import hello
## 输出:
## This runs on import
## (不会执行 if 块中的内容)
```

### 9.3. 完整的示例文件

#### 示例 1：工具模块

```python
#!/usr/bin/env python3
## -*- coding: utf-8 -*-

"""
字符串处理工具模块

提供常用的字符串操作函数，包括格式化、清理、转换等功能。

示例用法:
    >>> from string_utils import capitalize_words
    >>> capitalize_words("hello world")
    'Hello World'

作者: Python Team
版本: 2.1.0
"""

import re
from typing import List, Optional


## 常量
__version__ = "2.1.0"
DEFAULT_SEPARATOR = "-"
MAX_STRING_LENGTH = 10000


def capitalize_words(text: str) -> str:
    """
    将每个单词的首字母大写
    
    Args:
        text: 输入字符串
    
    Returns:
        首字母大写的字符串
    
    Examples:
        >>> capitalize_words("hello world")
        'Hello World'
        >>> capitalize_words("PYTHON programming")
        'Python Programming'
    """
    if not isinstance(text, str):
        raise TypeError("Input must be a string")
    
    if len(text) > MAX_STRING_LENGTH:
        raise ValueError(f"String too long (max {MAX_STRING_LENGTH} chars)")
    
    return text.title()


def slugify(text: str, separator: str = DEFAULT_SEPARATOR) -> str:
    """
    将文本转换为 URL 友好的 slug 格式
    
    Args:
        text: 输入文本
        separator: 分隔符，默认为 '-'
    
    Returns:
        slug 格式的字符串
    
    Examples:
        >>> slugify("Hello World!")
        'hello-world'
        >>> slugify("Python 编程", separator='_')
        'python_编程'
    """
    # 转小写
    text = text.lower()
    
    # 替换非字母数字字符为分隔符
    text = re.sub(r'[^\w\s]', '', text)
    text = re.sub(r'[\s_]+', separator, text)
    
    # 去除首尾分隔符
    return text.strip(separator)


def truncate(text: str, max_length: int = 100, suffix: str = "...") -> str:
    """
    截断文本到指定长度
    
    Args:
        text: 输入文本
        max_length: 最大长度
        suffix: 超出时添加的后缀
    
    Returns:
        截断后的文本
    
    Examples:
        >>> truncate("这是一段很长的文本", 5)
        '这是...'
    """
    if len(text) <= max_length:
        return text
    
    return text[:max_length - len(suffix)] + suffix


## 测试代码
if __name__ == "__main__":
    print(f"String Utils v{__version__}")
    print("=" * 40)
    
    # 测试 capitalize_words
    test_text = "hello world from python"
    print(f"capitalize_words('{test_text}') = '{capitalize_words(test_text)}'")
    
    # 测试 slugify
    title = "Hello World! This is a Test."
    print(f"slugify('{title}') = '{slugify(title)}'")
    
    # 测试 truncate
    long_text = "这是一段非常长的文本，需要被截断"
    print(f"truncate('{long_text}', 10) = '{truncate(long_text, 10)}'")
```

#### 示例 2：类模块

```python
#!/usr/bin/env python3
## -*- coding: utf-8 -*-

"""
用户模型模块

定义用户数据结构和相关操作。
"""

from datetime import datetime
from typing import Optional, List
import hashlib


class User:
    """
    用户类
    
    表示系统中的一个用户，包含基本信息和操作方法。
    
    Attributes:
        id: 用户 ID
        username: 用户名
        email: 邮箱地址
        created_at: 创建时间
    """
    
    # 类常量
    MIN_USERNAME_LENGTH = 3
    MAX_USERNAME_LENGTH = 50
    SYSTEM_PREFIX = "sys_"
    
    def __init__(self, user_id: int, username: str, email: str):
        """
        初始化用户
        
        Args:
            user_id: 用户 ID
            username: 用户名
            email: 邮箱地址
        
        Raises:
            ValueError: 当用户名或邮箱格式不正确时
        """
        self.id = user_id
        self.username = self._validate_username(username)
        self.email = self._validate_email(email)
        self._password_hash: Optional[str] = None
        self.created_at = datetime.now()
        self.is_active = True
        self._login_count = 0
    
    def _validate_username(self, username: str) -> str:
        """验证用户名合法性"""
        if not username:
            raise ValueError("Username cannot be empty")
        if len(username) < self.MIN_USERNAME_LENGTH:
            raise ValueError(f"Username too short (min {self.MIN_USERNAME_LENGTH})")
        if len(username) > self.MAX_USERNAME_LENGTH:
            raise ValueError(f"Username too long (max {self.MAX_USERNAME_LENGTH})")
        return username
    
    def _validate_email(self, email: str) -> str:
        """验证邮箱格式"""
        if not email or '@' not in email:
            raise ValueError("Invalid email format")
        return email.lower()
    
    def set_password(self, password: str) -> None:
        """设置密码"""
        self._password_hash = hashlib.sha256(password.encode()).hexdigest()
    
    def check_password(self, password: str) -> bool:
        """验证密码"""
        if self._password_hash is None:
            return False
        return self._password_hash == hashlib.sha256(password.encode()).hexdigest()
    
    def login(self) -> None:
        """记录登录"""
        self._login_count += 1
    
    @property
    def login_count(self) -> int:
        """获取登录次数"""
        return self._login_count
    
    @property
    def display_name(self) -> str:
        """获取显示名称"""
        return self.username.capitalize()
    
    def is_system_user(self) -> bool:
        """检查是否为系统用户"""
        return self.username.startswith(self.SYSTEM_PREFIX)
    
    def __str__(self) -> str:
        return f"User({self.username})"
    
    def __repr__(self) -> str:
        return f"User(id={self.id}, username='{self.username}')"
    
    def to_dict(self) -> dict:
        """转换为字典"""
        return {
            'id': self.id,
            'username': self.username,
            'email': self.email,
            'created_at': self.created_at.isoformat(),
            'is_active': self.is_active
        }


## 测试
if __name__ == "__main__":
    # 创建用户
    user = User(1, "alice", "alice@example.com")
    print(user)
    
    # 设置密码
    user.set_password("secret123")
    print(f"Password set: {user.check_password('secret123')}")
    
    # 登录测试
    user.login()
    user.login()
    print(f"Login count: {user.login_count}")
    
    # 显示信息
    print(f"Display name: {user.display_name}")
    print(f"Is system user: {user.is_system_user()}")
    print(f"User data: {user.to_dict()}")
```

### 9.4. 代码组织和风格指南

#### 9.4.1. 空行使用规范

```python
## 模块级定义之间：2 个空行
def function_one():
    pass


def function_two():
    pass


## 类定义与前后内容：2 个空行
class MyClass:
    pass


## 方法之间：1 个空行
class MyClass:
    def method_one(self):
        pass
    
    def method_two(self):
        pass


## 函数内部逻辑段落：1 个空行
def complex_function():
    # 第一部分：初始化
    data = []
    
    # 第二部分：处理
    for i in range(10):
        data.append(i)
    
    # 第三部分：返回
    return data
```

#### 9.4.2. 导入语句的最佳实践

```python
## ✅ 好的导入
from collections import defaultdict, Counter
import logging
from typing import List, Dict, Optional

## ⚠️ 避免循环导入的解决方案

## 方案 1：在函数内部导入
## file_a.py
def func_a():
    from file_b import func_b  # 延迟导入
    return func_b()

## 方案 2：重构代码结构
## 将共同依赖提取到单独模块
## common.py
def shared_utility():
    pass

## file_a.py 和 file_b.py 都导入 common
from common import shared_utility
```

#### 9.4.3. 命名一致性和可见性

```python
## 公开 API（无下划线前缀）
def public_function():
    """可以被外部模块使用"""
    pass

class PublicClass:
    """公开类"""
    pass

## 内部使用（单下划线前缀）
def _internal_helper():
    """约定：不应该在外部使用"""
    pass

_internal_var = "internal"

## 名称改写（双下划线前缀）
class MyClass:
    def __private_method(self):
        """会被改写为 _MyClass__private_method"""
        pass
    
    __private_attr = "private"

## 魔术方法（双下划线前后）
class MyClass:
    def __init__(self):
        pass
    
    def __str__(self):
        return "MyClass"
```

### 9.5. 常见错误和注意事项

#### 9.5.1. 导入相关错误

```python
## ❌ 错误：导入顺序混乱
from .models import User
import os
import requests
from .utils import helper

## ✅ 正确：按标准库、第三方、本地排序
import os
import requests
from .models import User
from .utils import helper

## ❌ 错误：通配符导入导致命名污染
from module import *

## ✅ 正确：明确导入
from module import specific_function, AnotherClass

## ❌ 错误：循环导入
## module_a.py
from module_b import func_b
def func_a(): pass

## module_b.py
from module_a import func_a  # 循环依赖！
def func_b(): pass

## ✅ 解决方案：重构或使用局部导入
```

#### 9.5.2. 文档字符串缺失

```python
## ❌ 不好：没有文档
def calculate(a, b):
    return a + b

## ✅ 好：完整文档
def calculate(a: float, b: float) -> float:
    """
    计算两个数的和
    
    Args:
        a: 第一个数
        b: 第二个数
    
    Returns:
        两数之和
    """
    return a + b
```

#### 9.5.3. 主入口使用不当

```python
## ❌ 错误：测试代码污染全局
def main_function():
    pass

print("Testing...")  # 导入时就会执行
main_function()

## ✅ 正确：使用 if __name__ == "__main__"
def main_function():
    pass

if __name__ == "__main__":
    print("Testing...")
    main_function()
```

### 9.6. 总结

**Python 源文件标准结构：**
1. Shebang 行（Unix 可执行脚本）
2. 编码声明
3. 模块文档字符串
4. 导入语句（分组排序）
5. 模块级常量和变量
6. 类定义
7. 函数定义
8. 主程序入口

**关键要点：**
- 遵循 PEP 8 风格指南
- 保持清晰的层次和顺序
- 编写完整的文档字符串
- 使用 `if __name__ == "__main__"` 隔离测试代码
- 合理的空行分隔提高可读性

**最佳实践：**
- 一个文件只做一件事
- 保持文件简洁（建议<500 行）
- 及时重构过大的模块
- 为公开 API 编写详细文档
- 使用类型注解提高可读性

---

## 10. Python 项目基本文件目录结构

合理的项目目录结构对于 Python 项目的可维护性、可扩展性和团队协作至关重要。本节介绍几种常见的项目组织模式。

### 10.1. 简单项目结构

适用于小型脚本、工具程序或学习项目。

```
my_project/
├── main.py              # 主程序入口
├── utils.py             # 工具函数
├── config.py            # 配置文件
├── requirements.txt     # 依赖列表
└── README.md            # 项目说明
```

**示例：数据爬取脚本**
```
web_scraper/
├── scraper.py           # 爬虫主逻辑
├── parsers.py           # 数据解析
├── storage.py           # 数据存储
├── config.json          # 配置（目标网站、字段等）
├── requirements.txt     # requests, beautifulsoup4
└── README.md
```

**特点：**
- ✅ 结构简单，一目了然
- ✅ 适合单人开发或小项目
- ❌ 不适合大型项目
- ❌ 模块组织不够清晰

### 10.2. 标准包结构（推荐）

适用于中型项目、可发布的 Python 包。

```
my_project/
├── my_package/              # 主包目录
│   ├── __init__.py          # 包初始化文件
│   ├── core.py              # 核心逻辑
│   ├── utils.py             # 工具函数
│   ├── models.py            # 数据模型
│   ├── config.py            # 配置
│   └── exceptions.py        # 自定义异常
│
├── tests/                   # 测试目录
│   ├── __init__.py
│   ├── test_core.py
│   ├── test_utils.py
│   └── test_models.py
│
├── docs/                    # 文档目录
│   ├── index.md
│   ├── api.md
│   └── tutorial.md
│
├── examples/                # 示例代码
│   ├── basic_usage.py
│   └── advanced_example.py
│
├── scripts/                 # 辅助脚本
│   ├── setup_db.py
│   └── deploy.sh
│
├── data/                    # 数据文件（如有）
│   ├── input/
│   └── output/
│
├── .gitignore               # Git 忽略配置
├── .env                     # 环境变量（不提交到 Git）
├── .env.example             # 环境变量示例
├── requirements.txt         # 生产环境依赖
├── requirements-dev.txt     # 开发环境依赖
├── setup.py                 # 包安装配置（传统方式）
├── pyproject.toml           # 项目配置（现代方式）
├── README.md                # 项目说明
├── LICENSE                  # 许可证
└── CHANGELOG.md             # 变更日志
```

#### 10.2.1. 各目录详细说明

**主包目录 (`my_package/`)**
```python
## my_package/__init__.py
"""
My Package - 一个有用的 Python 包

提供 XXX、YYY、ZZZ 等功能。
"""

__version__ = "1.0.0"
__author__ = "Your Name"

from .core import main_function
from .utils import helper_function
from .models import DataModel

## 定义包的公开 API
__all__ = [
    'main_function',
    'helper_function',
    'DataModel',
]
```

**测试目录 (`tests/`)**
```python
## tests/test_core.py
"""核心模块的单元测试"""

import unittest
from my_package.core import main_function

class TestCoreFunction(unittest.TestCase):
    
    def test_normal_case(self):
        """测试正常情况"""
        result = main_function("input")
        self.assertEqual(result, "expected")
    
    def test_edge_case(self):
        """测试边界情况"""
        with self.assertRaises(ValueError):
            main_function("")
    
    def test_performance(self):
        """测试性能"""
        import time
        start = time.time()
        for _ in range(1000):
            main_function("test")
        end = time.time()
        self.assertLess(end - start, 1.0)  # 1 秒内完成

if __name__ == "__main__":
    unittest.main()
```

**文档目录 (`docs/`)**
```markdown
## docs/index.md
## My Package 文档

欢迎使用 My Package！

### 快速开始

```python
from my_package import main_function
result = main_function("input")
```

### 目录

- [API 文档](api.md)
- [教程](tutorial.md)
- [常见问题](faq.md)

```

**示例目录 (`examples/`)**
```python
## examples/basic_usage.py
"""基础使用示例"""

from my_package import main_function, DataModel

## 示例 1：简单调用
result = main_function("test input")
print(f"Result: {result}")

## 示例 2：使用数据模型
data = DataModel(name="example", value=42)
processed = main_function(data)
print(f"Processed: {processed}")
```

#### 10.2.2. 配置文件详解

**requirements.txt** - 生产环境依赖
```txt
## 核心依赖
requests>=2.28.0
pandas>=1.5.0
numpy>=1.23.0

## Web 框架
flask>=2.2.0

## 数据库
sqlalchemy>=2.0.0
psycopg2-binary>=2.9.0

## 工具库
python-dotenv>=1.0.0
pydantic>=2.0.0
```

**requirements-dev.txt** - 开发环境依赖
```txt
## 包含生产环境依赖
-r requirements.txt

## 测试框架
pytest>=7.0.0
pytest-cov>=4.0.0
pytest-asyncio>=0.21.0

## 代码质量
flake8>=6.0.0
black>=23.0.0
isort>=5.12.0
mypy>=1.0.0

## 文档生成
sphinx>=6.0.0
sphinx-rtd-theme>=1.2.0

## 开发工具
ipython>=8.0.0
jupyter>=1.0.0
```

**.gitignore** - Git 忽略配置
```gitignore
## Byte-compiled / optimized / DLL files
__pycache__/
*.py[cod]
*$py.class

## C extensions
*.so

## Distribution / packaging
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg

## PyInstaller
*.manifest
*.spec

## Installer logs
pip-log.txt
pip-delete-this-directory.txt

## Unit test / coverage reports
htmlcov/
.tox/
.nox/
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
*.py,cover
.hypothesis/
.pytest_cache/

## Translations
*.mo
*.pot

## Environments
.env
.venv
env/
venv/
ENV/
env.bak/
venv.bak/

## IDE
.idea/
.vscode/
*.swp
*.swo
*~

## Project specific
data/output/
*.log
.DS_Store
```

**.env.example** - 环境变量示例
```bash
## 数据库配置
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
REDIS_URL=redis://localhost:6379/0

## API 密钥
API_KEY=your-api-key-here
SECRET_KEY=your-secret-key-here

## 应用配置
DEBUG=False
LOG_LEVEL=INFO
PORT=8000
```

#### 10.2.3. 项目配置文件

**setup.py** - 传统包安装配置
```python
from setuptools import setup, find_packages
from pathlib import Path

## 读取 README
this_directory = Path(__file__).parent
long_description = (this_directory / "README.md").read_text(encoding='utf-8')

## 读取 requirements
requirements = []
with open('requirements.txt', 'r', encoding='utf-8') as f:
    requirements = [line.strip() for line in f if line.strip() and not line.startswith('#')]

setup(
    name='my-package',
    version='1.0.0',
    author='Your Name',
    author_email='your.email@example.com',
    description='一个有用的 Python 包',
    long_description=long_description,
    long_description_content_type='text/markdown',
    url='https://github.com/yourusername/my-package',
    packages=find_packages(exclude=['tests', 'examples']),
    classifiers=[
        'Development Status :: 4 - Beta',
        'Intended Audience :: Developers',
        'License :: OSI Approved :: MIT License',
        'Programming Language :: Python :: 3',
        'Programming Language :: Python :: 3.8',
        'Programming Language :: Python :: 3.9',
        'Programming Language :: Python :: 3.10',
        'Programming Language :: Python :: 3.11',
    ],
    python_requires='>=3.8',
    install_requires=requirements,
    extras_require={
        'dev': [
            'pytest',
            'black',
            'flake8',
        ]
    },
    entry_points={
        'console_scripts': [
            'my-command=my_package.cli:main',
        ],
    },
)
```

**pyproject.toml** - 现代项目配置（推荐）
```toml
[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "my-package"
version = "1.0.0"
description = "一个有用的 Python 包"
readme = "README.md"
requires-python = ">=3.8"
license = {text = "MIT"}
authors = [
    {name = "Your Name", email = "your.email@example.com"}
]
keywords = ["package", "useful", "python"]
classifiers = [
    "Development Status :: 4 - Beta",
    "Intended Audience :: Developers",
    "License :: OSI Approved :: MIT License",
    "Programming Language :: Python :: 3",
]
dependencies = [
    "requests>=2.28.0",
    "pandas>=1.5.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0.0",
    "black>=23.0.0",
    "flake8>=6.0.0",
]

[project.urls]
Homepage = "https://github.com/yourusername/my-package"
Documentation = "https://my-package.readthedocs.io"
Repository = "https://github.com/yourusername/my-package.git"

[tool.setuptools.packages.find]
include = ["my_package*"]
exclude = ["tests", "examples"]

[tool.black]
line-length = 88
target-version = ['py38', 'py39', 'py310']

[tool.isort]
profile = "black"
line_length = 88

[tool.mypy]
python_version = "3.8"
warn_return_any = true
warn_unused_configs = true

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = "test_*.py"
addopts = "-v --cov=my_package"
```

### 10.3. Web 应用项目结构

适用于 Flask、Django 等 Web 框架项目。

```
web_app/
├── app/                       # 应用主目录
│   ├── __init__.py            # 应用工厂
│   ├── main.py                # Flask/Django 应用实例
│   ├── config.py              # 配置管理
│   │
│   ├── models/                # 数据模型
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── product.py
│   │
│   ├── routes/                # 路由处理
│   │   ├── __init__.py
│   │   ├── auth.py            # 认证路由
│   │   ├── api.py             # API 路由
│   │   └── views.py           # 页面视图
│   │
│   ├── services/              # 业务逻辑层
│   │   ├── __init__.py
│   │   ├── user_service.py
│   │   └── payment_service.py
│   │
│   ├── schemas/               # 数据验证（Pydantic）
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── product.py
│   │
│   ├── utils/                 # 工具函数
│   │   ├── __init__.py
│   │   ├── decorators.py
│   │   └── helpers.py
│   │
│   ├── templates/             # HTML 模板
│   │   ├── base.html
│   │   ├── index.html
│   │   └── login.html
│   │
│   ├── static/                # 静态资源
│   │   ├── css/
│   │   ├── js/
│   │   └── images/
│   │
│   └── middleware/            # 中间件
│       ├── __init__.py
│       └── authentication.py
│
├── migrations/                # 数据库迁移
│   ├── versions/
│   └── env.py
│
├── tests/                     # 测试
│   ├── __init__.py
│   ├── conftest.py            # pytest 配置
│   ├── test_auth.py
│   └── test_api.py
│
├── scripts/                   # 运维脚本
│   ├── deploy.sh
│   └── backup_db.py
│
├── logs/                      # 日志目录
│
├── .env                       # 环境变量
├── .gitignore
├── requirements.txt
├── requirements-dev.txt
├── gunicorn.conf.py           # Gunicorn 配置
├── Dockerfile                 # Docker 配置
├── docker-compose.yml         # Docker Compose
├── README.md
└── wsgi.py                    # WSGI 入口
```

#### 10.3.1. Flask 应用示例

```python
## app/__init__.py
"""Flask 应用工厂"""

from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
from flask_login import LoginManager

db = SQLAlchemy()
migrate = Migrate()
login_manager = LoginManager()

def create_app(config_name='default'):
    """应用工厂函数"""
    app = Flask(__name__)
    
    # 加载配置
    from .config import config
    app.config.from_object(config[config_name])
    
    # 初始化扩展
    db.init_app(app)
    migrate.init_app(app, db)
    login_manager.init_app(app)
    
    # 注册蓝图
    from .routes.auth import auth_bp
    from .routes.api import api_bp
    from .routes.views import views_bp
    
    app.register_blueprint(auth_bp, url_prefix='/auth')
    app.register_blueprint(api_bp, url_prefix='/api')
    app.register_blueprint(views_bp)
    
    return app
```

```python
## app/routes/auth.py
"""认证路由"""

from flask import Blueprint, request, jsonify, render_template
from flask_login import login_user, logout_user, login_required
from ..models.user import User
from ..services.user_service import authenticate_user

auth_bp = Blueprint('auth', __name__)

@auth_bp.route('/login', methods=['GET', 'POST'])
def login():
    """登录页面"""
    if request.method == 'POST':
        username = request.form.get('username')
        password = request.form.get('password')
        user = authenticate_user(username, password)
        if user:
            login_user(user)
            return jsonify({'success': True})
        return jsonify({'success': False, 'error': 'Invalid credentials'}), 401
    return render_template('login.html')

@auth_bp.route('/logout')
@login_required
def logout():
    """登出"""
    logout_user()
    return jsonify({'success': True})
```

### 10.4. Django 项目结构

Django 有自己约定的项目结构。

```
django_project/
├── manage.py                  # Django 管理脚本
│
├── django_project/            # 项目配置目录
│   ├── __init__.py
│   ├── settings.py            # 项目设置
│   ├── urls.py                # 主 URL 配置
│   ├── wsgi.py                # WSGI 入口
│   └── asgi.py                # ASGI 入口
│
├── apps/                      # 应用目录
│   ├── blog/                  # 博客应用
│   │   ├── __init__.py
│   │   ├── admin.py           # Admin 配置
│   │   ├── apps.py            # 应用配置
│   │   ├── models.py          # 数据模型
│   │   ├── views.py           # 视图函数
│   │   ├── urls.py            # URL 路由
│   │   ├── forms.py           # 表单类
│   │   ├── serializers.py     # DRF 序列化
│   │   ├── tests.py           # 测试
│   │   ├── templates/         # 模板
│   │   ├── static/            # 静态文件
│   │   └── migrations/        # 数据库迁移
│   │
│   └── accounts/              # 用户应用
│       └── ...
│
├── static/                    # 全局静态文件
│   ├── css/
│   ├── js/
│   └── images/
│
├── media/                     # 用户上传文件
│
├── templates/                 # 全局模板
│   └── base.html
│
├── requirements.txt
├── .env
├── .gitignore
└── README.md
```

### 10.5. 数据科学项目结构

```
data_science_project/
├── data/
│   ├── raw/                   # 原始数据（不修改）
│   ├── processed/             # 处理后的数据
│   └── external/              # 外部数据源
│
├── notebooks/                 # Jupyter Notebooks
│   ├── 01_data_exploration.ipynb
│   ├── 02_feature_engineering.ipynb
│   └── 03_model_training.ipynb
│
├── src/                       # 源代码
│   ├── __init__.py
│   ├── data_preprocessing.py  # 数据预处理
│   ├── feature_engineering.py # 特征工程
│   ├── model.py               # 模型定义
│   ├── training.py            # 训练逻辑
│   └── evaluation.py          # 评估指标
│
├── models/                    # 训练好的模型
│   ├── model_v1.pkl
│   └── model_v2.pkl
│
├── results/                   # 结果输出
│   ├── predictions.csv
│   └── metrics.json
│
├── configs/                   # 配置文件
│   ├── data_config.yaml
│   └── model_config.yaml
│
├── scripts/                   # 运行脚本
│   ├── train_model.sh
│   └── evaluate.sh
│
├── tests/                     # 测试
│   ├── test_preprocessing.py
│   └── test_model.py
│
├── requirements.txt
├── environment.yml            # Conda 环境
└── README.md
```

### 10.6. 包发布结构

如果要发布到 PyPI，需要更严格的组织。

```
my_package/
├── my_package/                # 包源码
│   ├── __init__.py
│   └── ...
│
├── tests/                     # 测试
│   └── ...
│
├── docs/                      # 文档
│   ├── source/
│   │   ├── conf.py
│   │   ├── index.rst
│   │   └── modules.rst
│   └── Makefile
│
├── .github/                   # GitHub 配置
│   └── workflows/
│       ├── ci.yml             # CI/CD
│       └── release.yml        # 发布流程
│
├── .gitignore
├── .editorconfig              # 编辑器配置
├── .pre-commit-config.yaml    # pre-commit 钩子
├── MANIFEST.in                # 打包文件清单
├── pyproject.toml             # 项目配置
├── setup.py                   # 安装脚本
├── tox.ini                    # 多环境测试
├── pytest.ini                 # pytest 配置
├── README.md
├── LICENSE
├── AUTHORS.rst                # 作者列表
├── CHANGELOG.md               # 变更日志
├── CONTRIBUTING.md            # 贡献指南
└── CODE_OF_CONDUCT.md         # 行为准则
```

### 10.7. 虚拟环境和依赖管理

#### 10.7.1. 创建虚拟环境

```bash
## 使用 venv（Python 3.3+ 内置）
python -m venv venv

## 激活虚拟环境
## Windows
venv\Scripts\activate
## macOS/Linux
source venv/bin/activate

## 退出虚拟环境
deactivate
```

#### 10.7.2. 使用 poetry（现代推荐）

```bash
## 安装 poetry
pip install poetry

## 创建新项目
poetry new my-project

## 添加依赖
poetry add requests pandas
poetry add --group dev pytest black

## 安装所有依赖
poetry install

## 运行命令
poetry run python main.py

## 进入虚拟环境
poetry shell
```

### 10.8. 项目结构最佳实践

#### 10.8.1. 通用原则

1. **清晰的层次结构**
   - 按功能模块组织
   - 避免过深的嵌套（建议不超过 3 层）

2. **命名一致性**
   - 全部使用小写字母和下划线
   - 避免使用空格和特殊字符

3. **分离关注点**
   - 代码、测试、文档分开
   - 源码、数据、配置分开

4. **便于扩展**
   - 预留扩展空间
   - 遵循开闭原则

#### 10.8.2. 不同规模项目的选择

| 项目类型 | 推荐结构 | 特点 |
| :--- | :--- | :--- |
| **脚本/工具** | 简单结构 | 单个文件或少量模块 |
| **小型项目** | 标准包结构 | 清晰的模块划分 |
| **Web 应用** | Web 应用结构 | MVC/MTV 模式 |
| **数据科学** | DS 项目结构 | Notebook + 源码分离 |
| **开源包** | 包发布结构 | 完整的 CI/CD、文档 |

#### 10.8.3. 常见反模式

```
❌ 糟糕的结构
project/
├── final_code.py
├── final_code_v2.py
├── final_code_v3_really_final.py
├── test.py
├── test2.py
├── utils_copy.py
├── 新建文件夹/
└── 未命名文件夹/

✅ 推荐的结构
project/
├── src/
│   ├── __init__.py
│   ├── main.py
│   └── utils.py
├── tests/
│   ├── __init__.py
│   └── test_main.py
├── docs/
├── requirements.txt
└── README.md
```

### 10.9. 实际案例

#### 案例 1：Requests 库结构
```
requests/
├── requests/
│   ├── __init__.py
│   ├── api.py
│   ├── sessions.py
│   ├── models.py
│   ├── adapters.py
│   └── utils.py
├── tests/
├── docs/
└── setup.py
```

#### 案例 2：Flask 结构
```
flask/
├── src/flask/
│   ├── __init__.py
│   ├── app.py
│   ├── blueprints.py
│   ├── config.py
│   └── templating.py
├── tests/
├── examples/
├── docs/
└── setup.py
```

### 10.10. 总结

**选择合适的结构：**
- 根据项目规模和类型选择
- 不要过度设计小项目
- 也不要让大项目过于简单

**关键要素：**
1. 清晰的模块划分
2. 测试和源码分离
3. 完善的文档
4. 明确的依赖管理
5. 统一的命名规范

**持续改进：**
- 定期重构代码结构
- 收集团队反馈
- 学习优秀开源项目
- 遵循社区最佳实践

**记住：** 最好的结构是适合你的团队和项目的结构！