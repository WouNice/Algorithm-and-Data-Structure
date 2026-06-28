# Python 标准库内置数据结构与算法 — 补充篇

## math 模块

> **一句话定位**：`math` 模块提供了**市面上最快、最可靠**的数学函数实现。它的底层是用 C 语言写的，比你用 Python 手写的数学计算函数快 10-100 倍。凡是遇到数学计算需求，**先翻翻 math 里有没有现成的**。

### 数论函数——专门和整数打交道的工具

#### math.gcd(a, b)

**这是什么？** `gcd` 的英文全称是"Greatest Common Divisor"，中文叫**最大公约数**，就是找出两个整数 a 和 b 都能被整除的最大那个整数。

**举个具体例子**：
- `gcd(12, 18)` = 6，因为 12 和 18 的最大公约数是 6
- `gcd(7, 13)` = 1，因为 7 和 13 都是质数，没有公共因子

**为什么不用自己手写？**你自己可以用循环写一个：

```python
def my_gcd(a, b):
    for i in range(min(a, b), 0, -1):
        if a % i == 0 and b % i == 0:
            return i
```

但这样写复杂度是 O(min(a,b))，如果 a=10亿 就要循环10亿次。而 `math.gcd` 内部用的是**欧几里得算法**（辗转相除法），复杂度只有 O(log min(a,b))，**快了几个数量级**。

```python
import math

# 使用方式
math.gcd(12, 18)          # → 6
math.gcd(7, 13)           # → 1
math.gcd(100, 0)          # → 100（任何数和 0 的 gcd 是该数本身）
```

**什么场景用？**

- 约分分数（分子分母同时除以 gcd）
- 判断两个数是否互质（gcd == 1）
- 循环节检测

#### math.lcm(a, b)

**这是什么？** `lcm` 是"Least Common Multiple"，中文叫**最小公倍数**，就是 a 和 b 的最小公共倍数。

**举个具体例子**：

- `lcm(4, 6)` = 12，因为 4的倍数有4,8,12,16...；6的倍数有6,12,18...；最小公共的是12
- `lcm(3, 5)` = 15

**关系公式**：`lcm(a, b) = a * b / gcd(a, b)`，所以不用死记硬背，它和 gcd 是连在一起的。

```python
import math

math.lcm(4, 6)            # → 12
math.lcm(3, 5)            # → 15
math.lcm(0, 6)            # → 0
math.lcm(10, 15, 20)      # → 60（支持多个数！Python 3.9+）
```

**什么场景用？**

- 分数通分（找分母的最小公倍数）
- 两个周期性事件下一次同时发生的时间

#### math.comb(n, k)

**这是什么？** `comb` 是"combination"的缩写，中文叫**组合数**，数学符号为 C(n, k) 或 ⁿCₖ。它回答"从 n 个物品中选出 k 个，有多少种选法"。

**举个具体例子**：

-   从 5 个人中选 2 个人做值日，有几种选法？
-   答案是 `comb(5, 2)` = 10 种。
-   （编号1-5的人，所有可能：12,13,14,15,23,24,25,34,35,45）

```python
import math

math.comb(5, 2)           # → 10
math.comb(10, 3)          # → 120
math.comb(4, 0)           # → 1（选0个只有1种方式：什么都不选）
math.comb(4, 4)           # → 1（全选也只有1种方式）
```

**为什么不用手写？**你可以用阶乘公式：C(n,k) = n! / (k! * (n-k)!)，但 50! 是一个天文数字（约 3×10⁶⁴），会产生**巨大的中间结果**。`math.comb` 内部用了更聪明的算法（避免了中间结果溢出）。

⚠️ **注意**：在 Python 3.8 之前的版本没有 `math.comb`。

**什么场景用？**

- 网格路径计数（从 m×n 网格左上角走到右下角走法的数量 = C(m+n-2, m-1)）
- 彩票中奖概率计算
- 子集枚举的数量计算

#### math.perm(n, k)

**这是什么？** `perm` 是"permutation"的缩写，中文叫**排列数**，数学符号为 P(n, k) 或 ⁿPₖ。它回答"从 n 个物品中选出 k 个，并且考虑它们排列的顺序，有多少种方式"。

**与 comb 的关键区别**：组合不考虑顺序，排列**考虑顺序**。

**举个具体例子**：从 3 个人 (A,B,C) 中选 2 个当班长和副班长（班长→副班长有顺序区别）：

- AB (A班长B副班)、BA (B班长A副班) 是**两种不同情况**
- 所以 `perm(3, 2)` = 6
- 而 `comb(3, 2)` = 3（因为"选A和B"不区分顺序）

```python
import math

math.perm(3, 2)           # → 6 （3个人选2个排列：AB,AC,BA,BC,CA,CB）
math.perm(5, 3)           # → 60
math.perm(4, 4)           # → 24（4人全排列 = 阶乘 4!）
math.perm(5, 0)           # → 1（选0个排列只有1种方式）
```

**什么场景用？**

- 密码学中排列数量计算
- 计算"有多少种安排座位的方式？"
- 需要知道排列总数而非枚举所有排列时的数学题

#### math.factorial(n)

**这是什么？** `factorial` 中文叫**阶乘**，n! 就是从 1 乘到 n。即 `n! = 1×2×3×...×n`

```python
import math

math.factorial(5)         # → 120（1×2×3×4×5）
math.factorial(0)         # → 1（0! 定义为 1）
math.factorial(10)        # → 3628800
```

**什么场景用？**
- 组合数学中 C(n,k) 和 P(n,k) 的手工计算（不过直接用 comb/perm 更好）
- 阶乘相关的数学题（如 LeetCode 172：阶乘后的零）

#### math.isqrt(n)

**这是什么？** `isqrt` 是"integer square root"的缩写，中文叫**整数平方根**。它计算 ⌊√n⌋，即 √n **向下取整**后的整数。

```python
import math

math.isqrt(25)            # → 5（因为 √25 = 5 正好）
math.isqrt(26)            # → 5（因为 √26 ≈ 5.099，向下取整得 5）
math.isqrt(100)           # → 10
math.isqrt(0)             # → 0
```

**为什么不用 math.sqrt(n) 再转 int？**`math.sqrt(n)` 返回**浮点数**（float），浮点数有精度限制。当 n 很大时（比如 10¹⁶ 级别），浮点数会丢失精度，给出错误结果。但 `math.isqrt` **全程用整数运算**，不管 n 多大都精确无误。

```python
import math

# ❌ float 精度问题
n = 9999999999999999 ** 2
int(math.sqrt(n))         # 可能返回 n-1，因为浮点数不够精确！

# ✓ isqrt 永远精确
math.isqrt(9999999999999999 ** 2)  # → 9999999999999999（正确！）
```

**什么场景用？**
- 判断一个数是否完全平方数（`isqrt(n) ** 2 == n`）
- 质数判定（只需检查到 √n 即可）
- 质因数分解

#### math.prod(iterable)

**这是什么？** `prod` 是"product"的缩写，中文叫**连乘**。它把可迭代对象里的所有数字乘起来。

```python
import math

math.prod([2, 3, 4])      # → 24（2×3×4）
math.prod(range(1, 6))    # → 120（1×2×3×4×5，等价于 5!）
math.prod([1, 2, 3], start=10)  # → 60（10×1×2×3）
```

**什么场景用？**
- 替代手写累乘循环
- 配合 itertools 做复杂的排列组合计算

#### math.dist(p, q)

**这是什么？** `dist` 是"distance"的缩写，计算**欧几里得距离**（两点间的直线距离）。

**公式**：$dist([x₁,y₁], [x₂,y₂]) = √((x₁-x₂)² + (y₁-y₂)²)$，在三维空间中则为：$dist([x₁,y₁,z₁], [x₂,y₂,z₂]) = √((x₁-x₂)² + (y₁-y₂)² + (z₁-z₂)²)$

```python
import math

# 二维
math.dist([0, 0], [3, 4])       # → 5.0（直角三角形，3-4-5）
# 三维
math.dist([0, 0, 0], [1, 1, 1]) # → 1.732...（√3）
```

#### math.hypot(*coords)

**这是什么？** `hypot` 是"hypotenuse"的缩写，中文叫**斜边**。它计算 √(x₁² + x₂² + x₃² + ...)，也就是从原点到 n 维空间中某点的距离。

```python
import math

math.hypot(3, 4)          # → 5.0（直角三角形斜边，√(3²+4²)）
math.hypot(1, 1, 1)       # → 1.732...（√(1²+1²+1²) = √3）
```

**dist 和 hypot 的关系**：`dist([x1,y1],[x2,y2])` = `hypot(x1-x2, y1-y2)`

#### math.nextafter(x, y)

**这是什么？** "next after"的缩写，返回从 x 出发向 y 方向移动**一个**浮点数精度单位后的值。也就是说，把 x 稍微往 y 的方向挪一丁点。

```python
import math

math.nextafter(1.0, 2.0)           # → 1.0000000000000002（向 2 方向挪动）
math.nextafter(1.0, 0.0)           # → 0.9999999999999999（向 0 方向挪动）
```

**什么场景用？**
- 遍历浮点数的所有可表示值
- 避免除零错误（当除数可能为 0 时，用 nextafter 移到最小正数）

#### math.ulp(x)

**这是什么？** `ulp` 是"Unit in the Last Place"的缩写，表示 x 与其下一个可表示浮点数之间的**差距大小**。可以理解为浮点数在当前刻度下的"最小步长"。

```python
import math

math.ulp(1.0)             # → 2.220446049250313e-16（1附近的最小步长）
math.ulp(1000.0)          # → 1.1368683772161603e-13（1000附近步长变大）
math.ulp(1e-10)           # → 非常小（越靠近 0 步长越小）
```

**关键概念**：浮点数不是均匀分布的——越靠近 0 精度越高，越远离 0 精度越低。

### 浮点函数

#### math.isclose(a, b, *, rel_tol=1e-9, abs_tol=0.0)

**这是什么？** "is close"——判断两个浮点数是否"足够接近"（在允许的误差范围内相等）。

**为什么需要这个？**计算机用二进制表示小数时会有**精度误差**：

```python
0.1 + 0.2 == 0.3           # → False！❌（因为 0.1+0.2=0.30000000000000004）
```

直接用 `==` 比较浮点数不可靠。`isclose` 解决了这个问题：

```python
import math

math.isclose(0.1 + 0.2, 0.3)        # → True ✓（默认容忍 1e-9 的相对误差）

# rel_tol = 相对容忍度（基于数值大小的百分比误差）
math.isclose(1e100, 1e100 + 1, rel_tol=1e-9)  # → True（相对误差很小）

# abs_tol = 绝对容忍度（接近 0 时用绝对误差）
math.isclose(1e-100, 2e-100, abs_tol=1e-50)   # → False（绝对误差 > 1e-50）
math.isclose(1e-100, 2e-100, rel_tol=0.5)     # → True（相对误差 100%，容忍 50%）

# 当数值接近 0 时，建议设置 abs_tol
math.isclose(0.0, 1e-15, abs_tol=1e-10)       # → True
```

**什么场景用？**
- 任何涉及浮点数 == 比较的地方
- 单元测试中比较计算结果
- 几何计算中判断点是否重叠

#### math.frexp(x)

**这是什么？** "fractional exponent"的缩写。它将一个浮点数 x 分解为**尾数 (fraction)** 和 **指数 (exponent)**，使 `x = 尾数 × 2ⁿ`。

```python
import math

math.frexp(12.5)          # → (0.78125, 4)  因为 0.78125 × 2⁴ = 12.5
math.frexp(1.0)           # → (0.5, 1)      因为 0.5 × 2¹ = 1.0
math.frexp(0.5)           # → (0.5, 0)      因为 0.5 × 2⁰ = 0.5
math.frexp(0.0)           # → (0.0, 0)
```

**尾数（第一个返回值）** 的范围在 [0.5, 1) 之间（正数时）。

**什么场景用？**

- 理解浮点数的二进制表示
- 科学计算中需要分离大小量级
- 特殊数值算法

#### math.ldexp(m, e)

**这是什么？** "load exponent"的缩写。`frexp` 的逆运算——给定尾数 m 和指数 e，返回 `m × 2ᵉ`。

```python
import math

math.ldexp(0.78125, 4)   # → 12.5（frexp(12.5) 的逆运算）
math.ldexp(1, 10)        # → 1024（即 1×2¹⁰）
math.ldexp(0.5, -1)      # → 0.25
```

#### math.modf(x)

**这是什么？** "modulo fraction"的缩写。将一个数字 x 拆分成**小数部分**和**整数部分**。

```python
import math

math.modf(3.14)           # → (0.14000000000000012, 3.0)
#     小数部分 ↗           整数部分 ↗

math.modf(-2.5)           # → (-0.5, -2.0)
math.modf(5.0)            # → (0.0, 5.0)
```

⚠️ **注意**：返回值是**元组 (小数部分, 整数部分)**。`modf` 返回的整数部分不是 int 而是 float。

**什么场景用？**
- 需要分别处理小数的整数和小数部分
- 货币金额拆分为元角分

#### math.remainder(x, y)

**这是什么？** "remainer"——计算 x 除以 y 的**IEEE 754 标准余数**。它与 Python 的 `%` 运算符（模运算）**不同**。

**关键区别**：
- `x % y`：结果与 y 的符号相同，取的是"向负无穷取整"的余数
- `math.remainder(x, y)`：结果在 `[-y/2, y/2]` 之间，取的是"向最近的整数取整"的余数

```python
import math

# ÷ 3，最近的整数倍是 9（3×3）和 12（3×4）
# 离 9 更近（差 1），所以 remainder 返回 1
math.remainder(10, 3)     # → 1.0

# ÷ 3，最近的整数倍是 12（3×4），差 -1
math.remainder(11, 3)     # → -1.0

# 对比 % 运算符
11 % 3                    # → 2（% 永远返回非负数）
math.remainder(11, 3)     # → -1.0（返回"最接近"的余数）
```

**什么场景用？**
- 需要"对称"余数（结果在正负之间分布）
- 信号处理、角度规范化

#### math.fmod(x, y)

**这是什么？** "floating point modulo"——专门为浮点数设计的**模运算**。

**与 x % y 的区别**：

- `x % y`：当 x 为负数时，结果 >= 0（向负无穷取整）
- `math.fmod(x, y)`：结果的符号与 x 一致（向零取整）

```python
import math

# 当 x 为负数时结果不同
math.fmod(-10, 3)         # → -1.0（向零取整：-10 ÷ 3 = -3 余 -1）
-10 % 3                   # → 2（向负无穷取整：-10 ÷ 3 = -4 余 2）

# 正数时二者一致
math.fmod(10, 3)          # → 1.0
10 % 3                    # → 1
```

### 算法题实战：综合使用 math 模块

**示例①：分数加减运算（LeetCode 592）**

**问题**：给你一个表示分数加减表达式的字符串 `"-1/2+1/3+1/4"`，计算出结果并以最简分数形式返回。

**为什么用 math**：需要 gcd 来约分，需要 isqrt 来优化（如果需要批量处理）。

```python
import math

def fractionAddition(expression):
    """将分数加减表达式计算结果"""
    i, n = 0, len(expression)
    numerator, denominator = 0, 1  # 从 0/1 开始累加

    while i < n:
        # 判断正负号
        sign = 1
        if expression[i] in '+-':
            sign = 1 if expression[i] == '+' else -1
            i += 1

        # 读取分子
        num = 0
        while i < n and expression[i].isdigit():
            num = num * 10 + int(expression[i])
            i += 1

        i += 1  # 跳过 '/' 斜杠

        # 读取分母
        den = 0
        while i < n and expression[i].isdigit():
            den = den * 10 + int(expression[i])
            i += 1

        # 合并分数：a/b + c/d = (a*d + c*b) / (b*d)
        numerator = numerator * den + sign * num * denominator
        denominator *= den

        # 用 gcd 约分，避免分子分母过大
        g = math.gcd(abs(numerator), denominator)
        numerator //= g
        denominator //= g

    return f"{numerator}/{denominator}"

# 测试
print(fractionAddition("-1/2+1/3+1/4"))  # → "-1/12"（通分计算得 -6/12+4/12+3/12 = 1/12）
```

**示例②：判断质数（经典算法）**

```python
import math

def is_prime(n):
    """判断 n 是否为质数"""
    if n < 2:
        return False
    # 只需检查到 √n。为什么？因为如果 n=a×b，则 min(a,b) ≤ √n
    for i in range(2, math.isqrt(n) + 1):
        if n % i == 0:
            return False
    return True

def sieve_of_eratosthenes(n):
    """埃拉托色尼筛法：找出 1 到 n 之间的所有质数"""
    is_prime = [True] * (n + 1)
    is_prime[0] = is_prime[1] = False
    for i in range(2, math.isqrt(n) + 1):
        if is_prime[i]:
            # 从 i² 开始标记（因为 i×(小于i的数) 已经被更小的质数标记过了）
            for j in range(i * i, n + 1, i):
                is_prime[j] = False
    return [i for i, p in enumerate(is_prime) if p]

# 测试
print(is_prime(17))                  # → True
print(is_prime(9999991))             # → True（大数也没问题）
print(sieve_of_eratosthenes(30))     # → [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
```

**示例③：不同路径 LeetCode 980（组合数公式）**

```python
import math

def uniquePaths(m, n):
    """
    一个机器人从 m×n 网格的左上角走到右下角，
    每次只能向右或向下走，求有多少种不同的走法。

    推理：总共需要走 (m-1) 步向下 + (n-1) 步向右 = (m+n-2) 步
    我们要在这 (m+n-2) 步中选择 (m-1) 步作为向下走
    所以答案 = C(m+n-2, m-1)
    """
    return math.comb(m + n - 2, m - 1)

print(uniquePaths(3, 7))   # → 28（3行7列的网格有28种走法）
```

## random 模块

> **一句话定位**：`random` 模块让你在 Python 中做**所有"随机"相关的事**——洗牌、抽奖、随机数字、打乱顺序。它底层用的是 Mersenne Twister 算法，是一种成熟、快速的伪随机数生成器。

### 随机浮点数

#### random.random()

**这是什么？** 生成一个 [0.0, 1.0) 之间的随机浮点数。

⚠️ `[0.0, 1.0)` 表示**包含 0.0，但不包含 1.0**。

```python
import random

random.random()           # → 比如 0.734289...
random.random()           # → 比如 0.156832...
# 每次调用都返回一个不同的值
```

**什么场景用？**
- 判断概率（`random.random() < 0.5` 相当于 50% 概率）
- 生成0~1之间的随机比例

#### random.uniform(a, b)

**这是什么？** 生成 [a, b] 范围内的**随机浮点数**（包含 a 和 b）。

```python
import random

random.uniform(1.5, 3.5)  # → 比如 2.731...
random.uniform(0, 100)    # → 比如 67.342...
```

**与 random.random() 的关系**：`uniform(a, b)` = `a + (b - a) * random.random()`

### 随机整数

#### random.randint(a, b)

**这是什么？** 生成 [a, b] 范围内的**随机整数**，**包含 a 和 b 两端**。

```python
import random

random.randint(1, 6)      # → 随机得到 1,2,3,4,5,6 之一（模拟骰子）
random.randint(0, 1)      # → 随机 0 或 1（模拟硬币）
```

#### random.randrange(start, stop [, step])

**这是什么？** 从 `range(start, stop, step)` 这个整数序列中随机选一个。与 `randint` 的区别：**不包含 stop**。

```python
import random

random.randrange(1, 10)          # → 1~9 之间的随机整数（不包含 10）
random.randrange(0, 10, 2)       # → 从 [0,2,4,6,8] 中随机选一个
random.randrange(5)              # → 0~4 之间的随机整数（相当于 randrange(0,5)）
```

**什么场景用？**

- 需要按步长随机（如只取偶数）
- 需要明确 stop 是**开区间**的场合

### 从序列中随机选

#### random.choice(seq)

**这是什么？** 从**非空序列**中随机选**一个**元素。

```python
import random

cards = ['♠A', '♥K', '♦Q', '♣J']
random.choice(cards)      # → 比如 '♥K'（随机四张牌中的一张）

# 字符串也是序列
random.choice('abcdef')   # → 比如 'c'

# 但从集合中选不行（集合不支持索引）
# random.choice({1, 2, 3})  → 报错！
```

#### random.choices(population, weights=None, k=1)

**这是什么？** 从 population 中**有放回**地选 k 次（同一个元素可以被多次选中）。可以设置权重，让某些元素更容易被选中。

**关键词"有放回"**：好比从袋子里抽小球，抽一个记下来，**放回去**再抽下一个，所以可能抽到同一个球多次。

```python
import random

# 不加权重，等概率选
random.choices(['A', 'B', 'C'], k=5)
# → 比如 ['B', 'A', 'C', 'A', 'B']（可重复，也能抽到 5 个 A）

# 加权重——概率不同
random.choices(['R', 'N', 'B'], weights=[0.5, 0.3, 0.2], k=10)
# → R 被选中的概率是 50%，N 是 30%，B 是 20%
```

#### random.sample(population, k)

**这是什么？** 从 population 中**无放回**地选 k 个元素（同一元素**不能被选中两次**）。比 `choices` 更符合日常的"抽奖"概念。

```python
import random

random.sample(['甲', '乙', '丙', '丁', '戊'], 3)
# → 比如 ['甲', '丁', '丙']（3个不同的获奖者）

# 从大范围抽，非常高效
random.sample(range(1000000), 10)  # 从 100万 个数中抽 10 个

# ⚠️ k 不能大于 population 的长度
# random.sample(['A', 'B'], 3)  → 报错！（只有2个元素却要抽3个）
```

**choices vs sample 的核心区别**：

| 特性 | choices | sample |
|------|---------|--------|
| 可否重复选到同一元素？ | ✅ 可以 | ❌ 不可以 |
| k 可以超过 len(pop)？ | ✅ 可以 | ❌ 不行 |
| 支持权重？ | ✅ 支持 | ❌ 不支持 |
| 典型场景 | 随机生成文本、蒙特卡洛模拟 | 抽奖、数据集拆分 |

#### random.shuffle(x)

**这是什么？** 将**原序列直接打乱**（**原地修改**，不返回新序列）。

```python
import random

cards = ['♠A', '♥K', '♦Q', '♣J']
random.shuffle(cards)
print(cards)              # → 比如 ['♣J', '♠A', '♥K', '♦Q']（原顺序被打乱）

# ⚠️ shuffle 不返回新列表！它修改原列表
result = random.shuffle([1, 2, 3])
print(result)             # → None（不是 [3, 1, 2]！）
```

**底层算法：Fisher-Yates 洗牌算法**

从后往前遍历，每次把一个随机位置的元素和当前位置交换。**O(n)** 时间即可完成洗牌，是最优的洗牌算法。

```python
# 等效手写版本
def fisher_yates_shuffle(x):
    for i in range(len(x) - 1, 0, -1):    # 从最后一个元素往前遍历
        j = random.randint(0, i)          # 随机选一个 ≤ i 的位置
        x[i], x[j] = x[j], x[i]           # 交换
```

#### random.seed(a)

**这是什么？** 设置随机数生成器的**种子**。种子的作用是让"随机"变得**可复现**。

**为什么需要？** 计算机无法产生真正的随机数，而是用公式"模拟"出看似随机的结果。种子就是公式的起始输入——**相同种子 → 相同的"随机"序列**。

```python
import random

random.seed(42)
random.random()           # → 0.6394...（固定的）

random.seed(42)
random.random()           # → 0.6394...（和上面完全一样！）

random.seed(42)
random.randint(1, 100)    # → 82（固定的）
random.randint(1, 100)    # → 81（固定的）
```

**什么场景用？**
- 调试时需要结果可重现
- 游戏中的地图种子
- A/B 测试需要相同随机顺序

#### random.getrandbits(k)

**这是什么？** 生成一个 k 位的非负随机整数（二进制下占 k 位）。

```python
import random

random.getrandbits(1)     # → 0 或 1（1位二进制）
random.getrandbits(8)     # → 0~255 之间的随机整数（8位，即1字节）
random.getrandbits(128)   # → 非常大的随机数（128位）
```

**什么场景用？**
- 需要生成特定位数的随机数
- 密码学中的非安全随机数

### 算法题实战

**示例①：用 rand7() 实现 rand10()（LeetCode 470）**

**问题**：给你一个只生成 1~7 的随机函数 rand7，请实现一个生成 1~10 的随机函数。

**思路**：用"拒绝采样法"——调用 rand7 两次，得到 [1,49] 之间的均匀随机数。只取前 40 个（因为 40 是 10 的倍数），然后映射到 1~10。

```python
def rand7():
    """假设这是题目提供的函数，生成 1~7 的随机整数"""
    return random.randint(1, 7)

def rand10():
    """用 rand7 实现 rand10"""
    while True:
        # 两次 rand7 得到 [1,49] 的均匀分布
        idx = (rand7() - 1) * 7 + rand7   # 范围 1~49
        if idx <= 40:                      # 取前 40 个值
            return (idx - 1) % 10 + 1      # 映射到 1~10
        # 如果 idx > 40，重新采样（约 18% 的概率被拒绝）
```

**示例②：打乱数组（LeetCode 384）**

```python
class Solution:
    def __init__(self, nums):
        self.original = nums[:]   # 复制一份原始数组
        self.nums = nums

    def reset(self):
        """重置为原始数组"""
        self.nums = self.original[:]
        return self.nums

    def shuffle(self):
        """随机打乱"""
        random.shuffle(self.nums)    # 一行搞定！O(n)
        return self.nums
```

**示例③：随机数索引（LeetCode 398）— 蓄水池抽样**

**问题**：在数组中随机选择一个等于 target 的索引，要求一次遍历且不知道数组长度。

**思路**：遍历时每遇到一个 target，有 1/count 的概率替换当前答案。这就是"蓄水池抽样"。

```python
class Solution:
    def __init__(self, nums):
        self.nums = nums

    def pick(self, target):
        """随机返回一个等于 target 的索引"""
        count = 0   # 遇到 target 的次数
        ans = 0     # 当前选中的索引
        for i, x in enumerate(self.nums):
            if x == target:
                count += 1
                # 有 1/count 的概率替换为当前索引
                if random.randint(1, count) == count:
                    ans = i
        return ans
```

## statistics 模块

> **一句话定位**：`statistics` 模块提供**常用统计数据**的计算函数——均值、中位数、标准差、方差等。大部分函数都是 O(n) 时间一次遍历。

### 集中趋势（数据的"中心"在哪里？）

#### statistics.mean(data)

**这是什么？** 计算**算术平均数**，也就是通常说的"平均值"。

**计算公式**：`mean = (x₁ + x₂ + ... + xₙ) / n`

```python
import statistics

statistics.mean([1, 2, 3, 4, 5])     # → 3（(1+2+3+4+5)/5 = 15/5 = 3）
statistics.mean([10, 20, 30])        # → 20
statistics.mean([5, 10, 15, 20])     # → 12.5
```

#### statistics.fmean(data)

**这是什么？** 和 `mean` 一样计算算术平均数，但专门针对**浮点数**（float），速度更快。Python 3.8+ 新增。

```python
import statistics

statistics.fmean([1.5, 2.5, 3.5])    # → 2.5（7.5/3）
# 和 mean 结果一样，但 fmean 更快

# 可以混用整数，但返回 float
statistics.fmean([1, 2, 3, 4])       # → 2.5
```

**什么时候用 fmean 而非 mean**：数据全是 float 且量很大时，fmean 约快 2-3 倍。

#### statistics.geometric_mean(data)

**这是什么？** 计算**几何平均数**。不太常见，但很有用。

**计算公式**：`geometric_mean = ⁿ√(x₁ × x₂ × ... × xₙ)`（所有数乘起来再开 n 次方）

**和 mean 的区别**：

- mean 适用于**加减关系**的数据（比如平均分数）
- geometric_mean 适用于**乘除关系**的数据（比如平均增长率）

**举个具体例子**：假设投资回报率——第一年涨 100%（乘 2），第二年跌 50%（乘 0.5），第三年涨 100%（乘 2）

- 算术平均 = (2+0.5+2)/3 = 1.5（看起来不错？）
- 几何平均 = ∛(2×0.5×2) = 1.0（实际没赚没赔——这才是对的！）

```python
import statistics

statistics.geometric_mean([1, 4, 16])   # → 4（∛(1×4×16) = ∛64 = 4）
statistics.geometric_mean([2, 0.5, 2])  # → 1.0（∛(2×0.5×2) = ∛2 = 1.0）
```

#### statistics.harmonic_mean(data)

**这是什么？** 计算**调和平均数**。适用于**"速率"类数据的平均**。

**计算公式**：`harmonic_mean = n / (1/x₁ + 1/x₂ + ... + 1/xₙ)`

**举个具体例子**：

假设去程车速 60 km/h，回程车速 40 km/h。平均速度是 (60+40)/2 = 50 吗？❌

不对！因为去程和回程时间不同。实际平均速度 = 调和平均 = 2/(1/60+1/40) = 48 km/h ✓

```python
import statistics

statistics.harmonic_mean([2, 3, 6])  # → 3
# / (1/2 + 1/3 + 1/6) = 3 / (3/6+2/6+1/6) = 3/(6/6) = 3/1 = 3

statistics.harmonic_mean([60, 40])   # → 48（正确计算平均速度）
```

#### statistics.median(data)

**这是什么？** 计算**中位数**——将数据从小到大排列，取中间的那个数。如果数据个数是偶数，返回中间两个数的平均值。

**中位数 vs 平均数的区别**：中位数**不受极端值影响**。

**举个具体例子**：班级考试成绩：[30, 60, 70, 80, 90]

- mean = 66（被30分拉低了）
- median = 70（中间值，不受30分影响）

```python
import statistics

statistics.median([1, 3, 2, 4])      # → 2.5（排好序 [1,2,3,4]，中间两个数 (2+3)/2 = 2.5）
statistics.median([1, 3, 2])         # → 2（排好序 [1,2,3]，中间是 2）

# 中位数对极端值不敏感
statistics.median([1, 2, 3, 100])    # → 2.5（(2+3)/2，100 这个极端值不影响）
statistics.mean([1, 2, 3, 100])      # → 26.5（被 100 严重拉高）
```

#### statistics.median_low(data) 和 median_high(data)

**这是什么？** `median` 的两种变体。当数据个数为偶数时，`median` 取中间两个数的平均，而：
- `median_low`：取中间两个数中**较小**的那个
- `median_high`：取中间两个数中**较大**的那个

```python
import statistics

statistics.median([1, 3, 2, 4])      # → 2.5（(2+3)/2）
statistics.median_low([1, 3, 2, 4])  # → 2（取较小的中间值）
statistics.median_high([1, 3, 2, 4]) # → 3（取较大的中间值）
```

#### statistics.mode(data)

**这是什么？** 计算**众数**——数据中出现**次数最多**的元素。如果多个元素出现次数相同，返回第一个遇到的。

```python
import statistics

statistics.mode([1, 1, 2, 3])        # → 1（1出现2次，最多）
statistics.mode(['red', 'blue', 'red', 'green'])  # → 'red'
```

#### statistics.multimode(data)

**这是什么？** 也是找众数。但 `multimode` 返回**所有**出现次数最多的元素（列表），而不只返回一个。

```python
import statistics

statistics.multimode([1, 1, 2, 2, 3])  # → [1, 2]（1和2都出现2次，并列最多）
statistics.multimode([1, 1, 2, 3])     # → [1]（只有1最多）
```

#### statistics.quantiles(data, n=4)

**这是什么？** 计算**分位数**——把数据排序后等分成 n 份，返回 n-1 个分界点的值。默认 n=4（四分位数）。

**什么是四分位数？**把数据从小到大排列，分成 4 份：

- 第 1 个分界点（Q1）：25% 的数据在此之下
- 第 2 个分界点（Q2）：50% 的数据在此之下（= 中位数）
- 第 3 个分界点（Q3）：75% 的数据在此之下

```python
import statistics

data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
statistics.quantiles(data, n=4)    # → [3.25, 5.5, 7.75]
# 排好序，分成4等份，返回3个分界值

# n=10 表示十分位数（返回9个分界值）
statistics.quantiles(data, n=10)   # → [1.9, 2.8, ...]
```

**什么场景用？**
- 数据分析和统计报表
- 异常值检测（IQR 方法：Q3 - Q1 的差值的 1.5 倍之外算异常值）

### 离散程度（数据"分散"的程度）

#### statistics.pstdev(data) 和 statistics.pvariance(data)

**这是什么？**

- `pstdev` = population standard deviation = **总体标准差**
- `pvariance` = population variance = **总体方差**

**方差（variance）** 衡量数据点离平均值的平均距离（的平方）。

**标准差（standard deviation）** 是方差的平方根，单位与原数据一致，更容易理解。

```python
import statistics

data = [1, 2, 3, 4]

statistics.pvariance([1, 2, 3, 4])   # → 1.25（方差）
# 计算过程：平均=2.5，每个偏差的平方和 = (1-2.5)²+(2-2.5)²+(3-2.5)²+(4-2.5)² = 2.25+0.25+0.25+2.25 = 5.0
# 总体方差 = 5.0/4 = 1.25

statistics.pstdev([1, 2, 3, 4])      # → 1.118...（标准差 = √1.25）
```

#### statistics.stdev(data) 和 statistics.variance(data)

**这是什么？**

- `stdev` = sample standard deviation = **样本标准差**
- `variance` = sample variance = **样本方差**

**"样本" vs "总体"的区别**：

- 如果你**拥有全部数据**（比如一个班的成绩），用 `pvariance` / `pstdev`
- 如果你**只有一部分数据**（比如从全市抽100人做样本），用 `variance` / `stdev`

**样本方差的计算分母是 n-1 而不是 n**（这叫"贝塞尔校正"，是为了在样本数据中更准确地估计总体方差）。

```python
import statistics

data = [1, 2, 3, 4]

statistics.variance([1, 2, 3, 4])    # → 1.666...（样本方差 = 5.0/3，分母 n-1=3）
statistics.pvariance([1, 2, 3, 4])   # → 1.25（总体方差 = 5.0/4，分母 n=4）

statistics.stdev([1, 2, 3, 4])       # → 1.290...（样本标准差 = √1.667）
statistics.pstdev([1, 2, 3, 4])      # → 1.118...（总体标准差 = √1.25）
```

## decimal 与 fractions 模块

### decimal — 十进制精确算术

**为什么需要 Decimal？—— 计算机浮点数的问题**

先看一个让人困惑的现象：

```python
0.1 + 0.2                           # → 0.30000000000000004 ❌ 不是 0.3！
```

这是因为计算机用二进制存储小数。就像你用十进制无法精确表示 1/3 = 0.3333... 一样，用二进制无法精确表示 0.1（它需要无限循环的二进制）。

`Decimal` 解决了这个问题——它用**十进制**存储，就像人类手算一样精确。

```python
from decimal import Decimal, getcontext, localcontext

# 创建 Decimal：必须用字符串！
Decimal('0.1') + Decimal('0.2')     # → Decimal('0.3') ✓ 精确！

# 如果用 float 创建，精度问题已经带入 Decimal
Decimal(0.1) + Decimal(0.2)         # → 可能又不精确了 ❌ 所以最好用字符串

# Decimal 可以指定精度
getcontext().prec = 50              # 设置全局精度为 50 位
Decimal(1) / Decimal(7)             # → 0.142857142857142857142857...（50位）

# 舍入模式
from decimal import ROUND_HALF_UP
getcontext().rounding = ROUND_HALF_UP

# quantize：精确舍入到指定小数位数
Decimal('2.675').quantize(Decimal('0.01'))  # → 2.68（四舍五入到两位小数）

# 对比 float 的舍入
round(2.675, 2)                     # → 2.67 ❌ 浮点问题
```

**Decimal 支持的数学运算**：

```python
# 高精度数学运算
Decimal(2).sqrt()                   # → 1.414213562...（任意精度）
Decimal('1.2').ln()                 # → 自然对数
Decimal('10').exp()                 # → 22026.46579...

# 局部精度控制（不全局修改）
with localcontext() as ctx:
    ctx.prec = 100
    result = Decimal(1) / Decimal(7)  # 100 位精度
    # 在此之外，精度恢复默认
```

**什么场景用 Decimal？**

- **金钱计算**（不能用浮点数算钱！）
- 需要精确十进制结果
- 会计系统、金融系统

### fractions — 分数

**为什么需要 Fraction？**`Fraction` 让你可以把数字看成**精确的分数**，而不是有误差的小数。所有分数运算都保持精确。

```python
from fractions import Fraction

# 创建分数
Fraction(1, 2)                      # → 1/2
Fraction('3/4')                     # → 3/4

# 分数运算——自动保持精确
Fraction(1, 2) + Fraction(1, 3)    # → Fraction(5, 6) 即 5/6
Fraction('3/4') * Fraction('2/5')  # → Fraction(3, 10) 即 3/10

# 自动约分
Fraction(4, 6)                      # → Fraction(2, 3)（自动约分）

# 对比 Decimal 和 float
Fraction('0.3')                     # → Fraction(3, 10) ✓  用字符串
Fraction(0.3)                       # → Fraction(5404319552844595, 18014398509481984) ❌ 用 float 就不对了

# 从 float 得到"最近似的分数"
Fraction(0.3).limit_denominator(100)  # → Fraction(3, 10) ✓ 限制分母不超过100

# 混合运算
Fraction(3, 4) > Decimal('0.7')    # → True（Fraction 可以和 Decimal 比较）

# 分数转字符串
frac = Fraction(5, 6)
f"{frac.numerator}/{frac.denominator}"  # → "5/6"
```

**什么场景用 Fraction？**

- 分数运算题目（LeetCode 592 分数加减法）
- 需要精确有理数计算
- 需要自动约分的场景

## io 模块

### StringIO — 把字符串当文件用

**这是什么？** 让你像操作文件一样操作字符串——可以用 `.write()` 写入、`.seek()` 移动位置、`.read()` 读取。

**为什么需要？**
- 拼接大量字符串时，用 `+` 拼接会导致 O(n²) 的性能（每次拼接都创建新字符串）
- 有些函数只接受"文件"参数，你想用字符串代替

```python
from io import StringIO

# 创建一个"内存中的文件"
buffer = StringIO()

# 像写文件一样写字符串
buffer.write('Hello\n')
buffer.write('World\n')

# 获取所有内容（类似读取整个文件）
buffer.getvalue()           # → 'Hello\nWorld\n'

# 回到开头然后逐行读取
buffer.seek(0)              # 将"读写位置"移到开头
for line in buffer:
    print(line.strip())     # 打印 Hello → World

# 别忘了关闭
buffer.close()

# 更优雅的方式：with 语句自动关闭
with StringIO() as buf:
    buf.write('data')
    result = buf.getvalue()

# 也可以直接用字符串初始化
buf = StringIO('初始内容')
buf.read()                  # → '初始内容'
```

**性能对比——拼接一百万个数字：**

```python
# ❌ 慢！用 + 拼接字符串
result = ''
for i in range(100000):
    result += str(i)      # O(n²) —— 每次拼接都复制整个字符串

# ✓ 快！StringIO
from io import StringIO
buf = StringIO()
for i in range(100000):
    buf.write(str(i))     # O(n) —— 内部用 list 高效拼接
result = buf.getvalue()

# ✓ 更快！更简洁：''.join()
result = ''.join(str(i) for i in range(100000))  # O(n)
```

### BytesIO — 把字节流当文件用

**和 StringIO 一样**，但操作的是**二进制数据**。

```python
from io import BytesIO

buf = BytesIO()
buf.write(b'\x00\x01\x02')   # 写二进制数据
buf.seek(0)
buf.read(2)                  # → b'\x00\x01'

# 常用于处理图片、音频等二进制数据
```

**什么场景用？**
- 树的序列化/反序列化（LeetCode 297）
- 需要"类文件对象"但不想创建真实文件的场景
- 测试中模拟文件读写
- 大量字符串拼接（虽然 `''.join()` 通常已经够用）

## copy 模块

**一句话**：`copy` 模块用来**复制对象**——当你想创建一个对象的副本而不是引用时使用。

### 先理解"引用"（不然看不懂深浅拷贝）

```python
# 在 Python 中，a = b 不创建副本，只是创建了一个"别名"
original = [1, 2, 3]
alias = original           # 没有复制！alias 和 original 指向同一个对象
alias[0] = 99
print(original[0])         # → 99 ❌ original 被修改了！
```

### 浅拷贝 vs 深拷贝

#### copy.copy(x) — 浅拷贝

**这是什么？** 创建一个**新的容器对象**，但容器内的元素仍然是**原对象的引用**。

```python
import copy

original = [[1, 2], [3, 4]]
shallow = copy.copy(original)

# 外层是新对象
shallow is original             # → False（不是同一个对象）

# 但内层是共享的！
shallow[0][0] = 99
print(original[0][0])           # → 99 ❌ 内层被修改了！

# 浅拷贝只复制最外层
# 一维 list 用浅拷贝就够了
original_1d = [1, 2, 3]
shallow_1d = copy.copy(original_1d)  # 等价于 original_1d[:]
shallow_1d[0] = 99
print(original_1d[0])           # → 1 ✓ 一维元素是不可变 int，不受影响
```

**何时用浅拷贝？**
- 一维 list（元素都是不可变类型，如 int/str/tuple）
- 你确定不需要修改嵌套内容

#### copy.deepcopy(x) — 深拷贝

**这是什么？** 递归地复制**所有层级**。新对象和原对象**完全独立**。

```python
import copy

original = [[1, 2], [3, 4]]
deep = copy.deepcopy(original)

deep[0][0] = 99
print(original[0][0])           # → 1 ✓ 完全不受影响！

deep[0] is original[0]          # → False（内层也是新创建的对象）
```

**⚠️ 深拷贝的性能代价**：深拷贝需要递归遍历整个对象图。对巨大的嵌套结构深拷贝可能非常慢，而且会循环拷贝（遇到循环引用也会处理）。

### 算法题中的正确用法

```python
# 回溯算法：不要每步深拷贝整个状态！
# 错误做法（太慢）：
def permute_wrong(nums):
    result = []
    def backtrack(path, remaining):
        if not remaining:
            result.append(copy.deepcopy(path))  # ❌ 可以更简单！
            return
        for i, x in enumerate(remaining):
            backtrack(path + [x], remaining[:i] + remaining[i+1:])
    backtrack([], nums)
    return result

# 正确做法（回溯模式）：
def permute_correct(nums):
    result = []
    def backtrack(path):
        if len(path) == len(nums):
            result.append(path[:])      # ✓ 一维 list 浅拷贝就够了
            return
        for x in nums:
            if x in path: continue
            path.append(x)             # 修改
            backtrack(path)            # 递归
            path.pop()                 # 恢复（回溯）
    backtrack([])
    return result

# ✅ 什么时候用浅拷贝？
result.append(path[:])                 # 一维 list
result.append(list(path))              # 等价写法
result.append(tuple(path))             # 转 tuple 更安全

# ✅ 什么时候用深拷贝？
# 保存二维或更复杂的可变嵌套结构时
original_state = [[0] * n for _ in range(n)]
saved = copy.deepcopy(original_state)  # 必须深拷贝
```

## operator 模块

> **一句话**：`operator` 把 Python 的运算符（+、-、[]、.等）变成**函数**，方便传给 `sorted`、`map`、`filter` 等需要"函数参数"的地方。

### itemgetter — 按索引或键取值

**这是什么？** 从序列或字典中取出指定位置的元素。你可以把 `itemgetter(1)` 理解成"获取第二个元素的函数"。

```python
from operator import itemgetter

# 基本作用：itemgetter(n) 返回一个函数，这个函数接受一个序列并返回第 n 个元素
get_second = itemgetter(1)       # 创建一个函数，功能是"取第二个元素"
get_second([10, 20, 30])         # → 20 ✓
get_second('hello')              # → 'e'

# 最常用的场景：排序时的 key 参数
students = [('Alice', 25), ('Bob', 20), ('Charlie', 30)]

# 按年龄排序（元组的第二个元素）
sorted(students, key=itemgetter(1))    # → [('Bob',20), ('Alice',25), ('Charlie',30)]

# 对比 lambda 写法——效果一样，但 itemgetter 更快
sorted(students, key=lambda x: x[1])   # 等价

# 多个键——先按年龄，再按姓名
sorted(students, key=itemgetter(1, 0)) # 先年龄升序，再姓名升序

# 字典也适用
data = [{'name': 'Alice', 'grade': 85}, {'name': 'Bob', 'grade': 92}]
sorted(data, key=itemgetter('grade'))  # 按 grade 排序
```

**用 itemgetter 还是 lambda？**

```python
# 只是取某个元素/键 → itemgetter（更快、更简洁）
sorted(data, key=itemgetter('score'))

# 逻辑复杂（比如计算值）→ lambda（itemgetter 做不了）
sorted(data, key=lambda x: x['score'] - x['penalty'])
```

### attrgetter — 按属性取值

**和 itemgetter 一样**，但操作的是**对象属性**（用 `.` 点号访问）。

```python
from operator import attrgetter

class Point:
    def __init__(self, x, y):
        self.x, self.y = x, y
    def __repr__(self):
        return f'Point({self.x},{self.y})'

points = [Point(3, 4), Point(1, 2), Point(2, 1)]

# 按 x 坐标排序
sorted(points, key=attrgetter('x'))      # → [Point(1,2), Point(2,1), Point(3,4)]

# 多个属性
sorted(points, key=attrgetter('y', 'x')) # 先按 y 再按 x

# 不用写 lambda
sorted(points, key=lambda p: p.x)        # 等价但慢
```

### methodcaller — 调用方法

**这是什么？** 创建一个函数，调用时会对参数执行指定方法。

```python
from operator import methodcaller

# 创建一个"调用 upper() 方法"的函数
to_upper = methodcaller('upper')
to_upper('hello')              # → 'HELLO'（相当于 'hello'.upper()）

# 批量调用
data = ['abc', 'defg', 'hi']
list(map(methodcaller('upper'), data))  # → ['ABC', 'DEFG', 'HI']

# 带参数的方法
starts_with_a = methodcaller('startswith', 'a')
starts_with_a('apple')         # → True
starts_with_a('banana')        # → False
```

### 算术和比较运算函数

| 函数 | 等价操作 | 说明 |
|------|---------|------|
| `add(a, b)` | `a + b` | 加法 |
| `sub(a, b)` | `a - b` | 减法 |
| `mul(a, b)` | `a * b` | 乘法 |
| `truediv(a, b)` | `a / b` | 除法（返回 float） |
| `floordiv(a, b)` | `a // b` | 整除 |
| `mod(a, b)` | `a % b` | 取模 |
| `pow(a, b)` | `a ** b` | 幂运算 |
| `neg(a)` | `-a` | 取负 |
| `abs(a)` | abs | 绝对值 |
| `eq(a, b)` | `a == b` | 相等 |
| `lt(a, b)` | `a < b` | 小于 |
| `xor(a, b)` | `a ^ b` | 异或 |

**最常用的场景：reduce + operator**

```python
from functools import reduce
from operator import add, mul, xor

# 求和
numbers = [1, 2, 3, 4, 5]
reduce(add, numbers)                # → 15（1+2+3+4+5）

# 累乘
reduce(mul, numbers)                # → 120（1×2×3×4×5）

# 异或（找LeetCode 136：只出现一次的数字）
nums = [4, 1, 2, 1, 2]
reduce(xor, nums)                   # → 4（因为 4⊕1⊕2⊕1⊕2 = 4）
```

**为什么用 operator 而不是手写 lambda？**

```python
# 写法对比
reduce(lambda a, b: a + b, nums)    # lambda 版本
reduce(add, nums)                   # operator 版本——更简洁、更快

# operator 是 C 语言实现的，比 Python 的 lambda 快
```

## enum 模块

> **一句话**：`enum` 用来定义**枚举**——一组有名字的常量。比用普通字符串或数字更安全、更可读。

### 基础枚举：Enum

**为什么需要 enum？**

```python
# ❌ 不用枚举：用字符串表示方向
direction = 'north'          # 拼写错了 'noth' 也不会报错
if direction == 'north':     # 全靠你记住正确的名字
    ...

# ✓ 用枚举：定义明确，拼错报错
class Direction(Enum):
    NORTH = 1
    SOUTH = 2
    EAST = 3
    WEST = 4

direction = Direction.NORTH  # 不会拼错
```

**核心用法：**

```python
from enum import Enum

# 定义枚举
class Color(Enum):
    RED = 1
    GREEN = 2
    BLUE = 3

# 使用枚举
Color.RED                    # → <Color.RED: 1>
Color.RED.value              # → 1（获取数值）
Color.RED.name               # → 'RED'（获取名字）

# 反向查找：通过值找枚举成员
Color(1)                     # → <Color.RED: 1>

# 通过名字找
Color['RED']                 # → <Color.RED: 1>

# 枚举成员之间比较
Color.RED == Color.RED       # → True
Color.RED == Color.BLUE      # → False
Color.RED is Color.RED       # → True

# 遍历所有枚举成员
for c in Color:
    print(c.name, c.value)   # RED 1, GREEN 2, BLUE 3

# 获取数量
len(Color)                   # → 3
```

### 重要特性

#### auto() — 自动赋值

```python
from enum import Enum, auto

# auto() 会自动分配 1, 2, 3, ...
class Status(Enum):
    PENDING = auto()         # 1
    ACTIVE = auto()          # 2
    DONE = auto()            # 3

Status.ACTIVE.value          # → 2
```

#### @unique — 禁止重复值

```python
from enum import Enum, unique

@unique
class Status(Enum):
    PENDING = 1
    ACTIVE = 2
    # COMPLETED = 2   ← 这行取消注释会报错！因为 2 被用了
```

#### IntEnum — 枚举值可以直接当整数用

```python
from enum import IntEnum

class Priority(IntEnum):
    LOW = 1
    MEDIUM = 2
    HIGH = 3

# 可以直接比较大小
Priority.HIGH > Priority.LOW     # → True ✓
Priority.LOW + 1                 # → 2 ✓（可以当 int 运算）

# 可以用在需要整数的所有地方
[Priority.LOW, Priority.HIGH].sort()  # 可以排序
```

### Flag — 位标志（组合多个选项）

```python
from enum import Flag, auto

# Flag 允许用位运算组合多个枚举值
class Permissions(Flag):
    NONE = 0
    READ = auto()             # 1（二进制 001）
    WRITE = auto()            # 2（二进制 010）
    EXECUTE = auto()          # 4（二进制 100）
    ALL = READ | WRITE | EXECUTE  # 7（二进制 111）

# 组合权限
perm = Permissions.READ | Permissions.WRITE  # 读写权限

# 检查是否包含某权限
Permissions.READ in perm             # → True ✓
Permissions.EXECUTE in perm          # → False

# 添加权限
perm |= Permissions.EXECUTE          # 添加执行权限

# 移除权限
perm &= ~Permissions.WRITE           # 移除写权限

# Flag 的核心优势：可以在一个数字里保存多个"开/关"状态
# 等价于用多个 bool，但更紧凑、更可读
```

### 算法题实战

**示例①：机器人模拟（状态机模式）**

方向枚举 + 转向逻辑 = 避免手写 if-elif 链

```python
from enum import Enum

class Direction(Enum):
    NORTH = 0
    EAST = 1
    SOUTH = 2
    WEST = 3

    def turn_right(self):
        """右转"""
        return Direction((self.value + 1) % 4)

    def turn_left(self):
        """左转"""
        return Direction((self.value - 1) % 4)

    def move(self, x, y):
        """向当前方向移动一步"""
        steps = {
            Direction.NORTH: (0, 1),
            Direction.EAST:  (1, 0),
            Direction.SOUTH: (0, -1),
            Direction.WEST:  (-1, 0),
        }
        dx, dy = steps[self]
        return x + dx, y + dy

# 使用示例
d = Direction.NORTH
d = d.turn_right()           # → EAST
d.move(0, 0)                 # → (1, 0)
```

**示例②：图的遍历状态（IntEnum）**

检测有向图是否有环（拓扑排序/LeetCode 207）

```python
from enum import IntEnum

class Visited(IntEnum):
    WHITE = 0    # 未访问
    GRAY = 1     # 正在访问中（如果再次遇到，说明有环！）
    BLACK = 2    # 访问完毕

def canFinish(numCourses, prerequisites):
    """判断是否能学完所有课程（有向图是否有环）"""
    graph = [[] for _ in range(numCourses)]
    for course, pre in prerequisites:
        graph[course].append(pre)

    state = [Visited.WHITE] * numCourses

    def has_cycle(v):
        if state[v] == Visited.GRAY:    # 正在访问又遇到→有环
            return True
        if state[v] == Visited.BLACK:   # 已经检查过，无环
            return False

        state[v] = Visited.GRAY         # 标记为"正在访问"
        for neighbor in graph[v]:
            if has_cycle(neighbor):
                return True
        state[v] = Visited.BLACK        # 标记为"已访问"
        return False

    for v in range(numCourses):
        if has_cycle(v):
            return False
    return True
```

## dataclasses 模块

> **一句话**：`@dataclass` 让你**一步定义数据类**——自动帮你写 `__init__`、`__repr__`、`__eq__` 等方法。比手写 class 省 80% 的模板代码。

### 基础用法——对比手写

```python
# ❌ 手写一个简单的数据类
class PointManual:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    def __repr__(self):
        return f'PointManual(x={self.x}, y={self.y})'
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

# ✓ dataclass 自动生成
from dataclasses import dataclass

@dataclass
class Point:
    x: float      # 类型注解——声明字段
    y: float

# 自动拥有了构造方法
p = Point(1.0, 2.0)             # 自动生成 __init__

# 自动拥有了字符串表示
print(p)                        # → Point(x=1.0, y=2.0) 自动生成 __repr__

# 自动拥有了相等比较
Point(1, 2) == Point(1, 2)      # → True 自动生成 __eq__
Point(1, 2) == Point(3, 4)      # → False
```

### 常用配置

#### 默认值

```python
@dataclass
class TreeNode:
    val: int = 0                        # 有默认值
    left: 'TreeNode' = None             # 不可变类型可直接赋值
    right: 'TreeNode' = None
    # ⚠️ children: list = []           # ← 这是错的！所有实例共享同一个 list
    children: list = None               # ← 正确的做法

# 如果要可变默认值，用 field(default_factory=)
from dataclasses import field

@dataclass
class TreeNode2:
    val: int = 0
    children: list = field(default_factory=list)  # 每次创建实例都生成新的空list
```

#### 可排序数据类

```python
# order=True 自动生成 __lt__、__le__、__gt__、__ge__
@dataclass(order=True)
class PriorityItem:
    priority: int
    name: str = field(compare=False)    # name 不参与比较

# 现在可以排序了！
items = [PriorityItem(2, 'B'), PriorityItem(1, 'A'), PriorityItem(3, 'C')]
sorted(items)                           # → 按 priority 排序
PriorityItem(1, 'A') < PriorityItem(2, 'B')  # → True
```

#### 不可变数据类

```python
# frozen=True —— 创建后不可修改（类似 tuple，但更灵活）
@dataclass(frozen=True)
class Point:
    x: int
    y: int

p = Point(1, 2)
# p.x = 3       ← 报错！frozen 不能修改
# 但 frozen 的好处是可以哈希，作为 dict key 或 set 元素
hash(p)                     # → 可哈希
{Point(1,2): 'origin'}      # → 可以当字典 key
```

#### 初始化后处理

```python
@dataclass
class Person:
    first_name: str
    last_name: str
    full_name: str = field(init=False)   # 不在 __init__ 参数中

    def __post_init__(self):
        """初始化后自动调用，可用来计算衍生字段"""
        self.full_name = f"{self.first_name} {self.last_name}"
        # self.full_name = field(init=False) 的字段在这里赋值

p = Person('John', 'Doe')
p.full_name                     # → 'John Doe'
```

### dataclasses 与 namedtuple 的对比

```python
from collections import namedtuple
from dataclasses import dataclass

# namedtuple——不可变、轻量
PointNT = namedtuple('Point', ['x', 'y'])
p1 = PointNT(1, 2)
p1.x                            # → 1
# p1.x = 3                     # 报错！不可修改

# dataclass——可变、更灵活
@dataclass
class PointDC:
    x: int
    y: int
p2 = PointDC(1, 2)
p2.x = 3                        # ✓ 可以修改

# 什么时候用哪个？
# namedtuple：简单的传值对象、需要哈希、内存敏感
# dataclass：需要类型注解、需要额外逻辑（__post_init__）、需要可变
```

### 算法题实战

**示例①：Dijkstra 优先队列的最清晰写法**

```python
import heapq

@dataclass(order=True)
class State:
    distance: float              # 用于排序（主键）
    vertex: int = field(compare=False)  # 顶点编号（不参与比较）

def dijkstra(graph, start):
    """最短路径"""
    n = len(graph)
    dist = [float('inf')] * n
    dist[start] = 0
    pq = [State(0, start)]

    while pq:
        d, v = heapq.heappop(pq)
        if d > dist[v]:
            continue
        for to, w in graph[v]:
            nd = d + w
            if nd < dist[to]:
                dist[to] = nd
                heapq.heappush(pq, State(nd, to))
    return dist
```

**示例②：BFS 中的路径记录**

```python
@dataclass
class BFSPath:
    x: int
    y: int
    path: list = field(default_factory=list, repr=False)  # 隐藏长路径

    def __post_init__(self):
        """自动记录当前位置"""
        self.path = self.path + [(self.x, self.y)]
```

## types 模块

### SimpleNamespace — 瞬间创建"对象"

**这是什么？** 一句话创建一个可以**动态添加属性**的对象，不需要提前定义类。

```python
from types import SimpleNamespace

# 创建对象——像字典一样灵活，但用 . 访问
obj = SimpleNamespace(x=1, y=2, name='test')
obj.x                           # → 1（属性访问）
obj.z = 3                       # 随时添加新属性

# 对比字典
d = {'x': 1, 'y': 2}
d['x']                          # → 1（字典用 [] 访问）

# SimpleNamespace 的好处：比字典更简洁，比类更快创建
```

**什么场景用？**
- 快速创建小对象传值
- 替代 `dict` 但想用 `.` 点号访问
- 不想专门 `class XXX:` 定义一个类

### MappingProxyType — 只读字典

**这是什么？** 把普通字典变成"只读"——可以看到内容，但不能修改。任何修改尝试都会报错。

```python
from types import MappingProxyType

original = {'a': 1, 'b': 2}
read_only = MappingProxyType(original)

read_only['a']                  # → 1（可以读）
# read_only['c'] = 3            # ← 报错！TypeError: 不支持修改

# 但！原字典变了，只读视图也会跟着变（因为它不是副本，是包装）
original['c'] = 3
read_only['c']                  # → 3（原字典变了，视图也能看到）
```

**什么场景用？**
- 函数需要返回"只读"配置，防止调用方篡改
- 只要你不想让别人意外修改你的数据

## typing 模块

> **一句话**：`typing` 是 Python 的**类型注解**系统——它**不影响运行**（Python 是动态类型的），但让你和 IDE 知道"这个参数应该是 int 还是 str"。

### 为什么要写类型注解？

```python
# 没注解——看代码只能猜
def add(x, y):
    return x + y        # x 和 y 可以是 int、float、list、str？
                        # 不确定

# 有注解——一目了然
def add(x: int, y: int) -> int:
    return x + y        # 明确：输入两个整数，返回整数
```

### 常用类型

```python
from typing import List, Dict, Set, Tuple, Optional, Union, Any, Callable, Deque
from collections import deque

# List — 列表
def process(nums: List[int]) -> int:       # 参数是 int 列表
    return sum(nums)

# Dict — 字典
def lookup(table: Dict[str, int], key: str) -> Optional[int]:
    return table.get(key)                   # → 可能返回 None

# Set — 集合
def unique(elements: Set[str]) -> int:
    return len(elements)

# Tuple — 元组
def split(point: Tuple[int, int]) -> Tuple[int, int]:
    return point[0], point[1]

# Optional — 可能为 None
def find(data: List[int], target: int) -> Optional[int]:
    """返回索引，不存在则返回 None"""
    if target in data:
        return data.index(target)
    return None          # 返回值类型是 int | None
# Optional[int] 等价于 Union[int, None]

# Union — 多种类型之一
def handle(value: Union[int, str]) -> str:
    return str(value)

# Callable — 函数类型
def sort_by(data: List[int], key_fn: Callable[[int], int]) -> List[int]:
    return sorted(data, key=key_fn)
# Callable[[参数类型], 返回类型]

# Deque — 双端队列类型
from collections import deque
def bfs(grid, start) -> int:
    q: Deque[Tuple[int, int]] = deque([start])  # 明确队列内容
    ...

# Any — 任何类型
def debug(value: Any) -> None:
    print(value)
```

### 进阶用法

#### 类型别名

```python
from typing import TypeAlias  # Python 3.10+

# 复杂的类型可以取别名
Grid: TypeAlias = List[List[int]]
Path: TypeAlias = List[Tuple[int, int]]

def shortest_path(grid: Grid) -> Optional[Path]:
    ...
```

#### 自引用的类型（如树节点）

```python
from typing import Optional

class TreeNode:
    def __init__(self, val: int = 0,
                 left: Optional['TreeNode'] = None,   # 字符串形式引用自身
                 right: Optional['TreeNode'] = None):
        self.val = val
        self.left = left
        self.right = right
```

## re 模块

> **一句话**：`re` 是**正则表达式**（Regular Expression）的模块，用"模式字符串"来快速搜索、替换、分割文本。

### 正则表达式基础

**什么是正则表达式？** 一种"模式语言"——你写一个模式，计算机用它去匹配字符串。

```python
import re

# 最简单的正则：精确匹配字符
re.search(r'hello', 'hello world')   # 匹配成功→返回 Match 对象
re.search(r'hello', 'goodbye')       # 匹配失败→返回 None
```

**r 前缀是什么意思？** `r'...'` 表示"原始字符串"——里面的反斜杠就是反斜杠，不需要转义。写正则时**永远用 `r'...'`**。

### 常用特殊字符

```python
# .  — 匹配任意字符（除了换行符）
re.search(r'he.lo', 'heXlo')   # → 匹配（. 匹配 X）
re.search(r'he.lo', 'hello')   # → 匹配（. 匹配 l）

# \d — 匹配数字（0-9）
re.search(r'\d+', 'abc123')    # → 匹配 '123'

# \w — 匹配字母、数字、下划线
re.search(r'\w+', 'hello!')    # → 匹配 'hello'

# \s — 匹配空白字符（空格、制表符等）

# *  — 前面的字符出现 0 次或多次
# +  — 前面的字符出现 1 次或多次
# ?  — 前面的字符出现 0 次或 1 次
# {n}— 前面的字符正好出现 n 次
# {n,m}— 前面的字符出现 n 到 m 次

# 例子
re.search(r'\d{3}', 'abc123def')  # → 匹配 '123'（正好3位数字）
re.search(r'a+', 'aaab')          # → 匹配 'aaa'（1个或多个a）
re.search(r'colou?r', 'color')    # → 匹配（u出现0次）
re.search(r'colou?r', 'colour')   # → 匹配（u出现1次）

# ^  — 行开头
# $  — 行末尾
re.search(r'^Hello', 'Hello world')  # → 匹配（开头）
re.search(r'^Hello', 'Hi Hello')     # → 不匹配（不是开头）

# [] — 字符集合
re.search(r'[aeiou]', 'hello')   # → 匹配 'e'（任意一个元音字母）
re.search(r'[0-9]', 'abc')       # → 不匹配（没有数字）
re.search(r'[a-z]', 'ABC')       # → 不匹配（只有大写）

# () — 分组（提取部分匹配内容）
m = re.search(r'(\d+)-(\d+)', '12-34')
m.group(0)                       # → '12-34'（完整匹配）
m.group(1)                       # → '12'（第一个括号）
m.group(2)                       # → '34'（第二个括号）
m.groups()                       # → ('12', '34')（所有分组）
```

### 核心函数

#### re.compile(pattern) —— 编译正则（推荐先编译）

```python
# 编译后可以反复使用，效率更高
pattern = re.compile(r'\d+')     # 编译一次

# 使用编译后的对象
pattern.findall('a1b2c3')        # → ['1', '2', '3']
pattern.search('a1b2c3')         # → 第一个匹配
pattern.match('123abc')          # → 从开头匹配
pattern.fullmatch('123')         # → 整个字符串必须全部匹配
```

#### findall —— 找到所有匹配

```python
re.findall(r'\d+', 'a1b2c3')     # → ['1', '2', '3']
# 返回所有匹配到的子串列表
```

#### search —— 查找第一个匹配

```python
m = re.search(r'(\d+)', 'abc123def456')
m.group(1)                       # → '123'（只返回第一个）
```

#### match vs search

- `match`：从字符串**开头**匹配
- `search`：在字符串**任意位置**匹配

```python
re.match(r'\d+', 'abc123')       # → None（开头不是数字）
re.search(r'\d+', 'abc123')      # → '123'（任意位置）
re.match(r'\d+', '123abc')       # → '123'（开头是数字）
```

#### sub —— 替换

```python
re.sub(r'\d+', '#', 'a1b2c3')    # → 'a#b#c#'（所有数字替换为 #）
re.subn(r'\d+', '#', 'a1b2c3')   # → ('a#b#c#', 3)（额外返回替换次数）

# 用函数替换
re.sub(r'\d+', lambda m: str(int(m.group(0)) * 2), 'a1b2c3')  # → 'a2b4c6'
```

#### split —— 分割

```python
re.split(r'\s+', 'a  b   c')     # → ['a', 'b', 'c']（按空白分割）

# 保留分隔符（用括号）
re.split(r'(\s+)', 'a  b')       # → ['a', '  ', 'b']
```

### 算法题实战

**示例①：验证 IP 地址（LeetCode 468）**

```python
import re

def validIPAddress(queryIP):
    """验证 IP 是 IPv4 还是 IPv6"""
    ipv4 = re.compile(r'^(\d{1,3}\.){3}\d{1,3}$')
    ipv6 = re.compile(r'^([0-9a-fA-F]{1,4}:){7}[0-9a-fA-F]{1,4}$')

    if ipv4.match(queryIP):
        parts = queryIP.split('.')
        for p in parts:
            # 检查前导零（如 "01" 不行）
            if (len(p) > 1 and p[0] == '0'):
                return "Neither"
            # 检查范围
            if int(p) > 255:
                return "Neither"
        return "IPv4"

    if ipv6.match(queryIP):
        return "IPv6"
    return "Neither"
```

**示例②：字符串解码（LeetCode 394）**

```python
def decodeString(s):
    """将 '3[a2[c]]' 解码为 'accaccacc'"""
    import re
    # 反复替换最内层的数字+方括号
    while '[' in s:
        s = re.sub(r'(\d+)\[([a-z]*)\]',
                   lambda m: int(m.group(1)) * m.group(2),  # 数字×内容
                   s)
    return s

# 解析过程：
# "3[a2[c]]" → 先替换 "2[c]" → "3[acc]" → 再替换 "3[acc]" → "accaccacc"
```

**⚠️ 性能陷阱**：正则嵌套过多可能导致"灾难性回溯"

```python
# ❌ 下面的模式处理长字符串时会极慢
re.match(r'(a+)+b', 'a' * 30 + 'c')   # 呈指数级变慢！

# ✓ 避免嵌套量词
re.match(r'a+b', 'a' * 30 + 'c')      # 线性时间
```

## weakref 模块

> **一句话**：不阻止对象被回收的"弱引用"。当没有其他人再用这个对象时，弱引用会自动变成 None——非常适合做**自动清理的缓存**。

### 先理解"引用"和"垃圾回收"

```python
# 正常情况下，只要还有变量引用一个对象，它就不会被回收
obj = [1, 2, 3]          # 引用计数 = 1
another = obj            # 引用计数 = 2
del obj                  # 引用计数 = 1（还没被回收）
del another              # 引用计数 = 0 → 回收！
```

### weakref.ref —— 弱引用

```python
import weakref

class BigData:
    def __init__(self, key):
        self.key = key
        self.data = "很大很大的数据"

obj = BigData('cache_key')
ref = weakref.ref(obj)       # 创建弱引用——不增加引用计数！

ref()                        # → BigData 对象（还存在）
print(ref().data)            # → '很大很大的数据'

del obj                      # 删除唯一的强引用
ref()                        # → None（对象已被回收，弱引用自动变为 None）
```

### WeakValueDictionary —— 值自动消失的字典

**最常用的 weakref 工具**。字典中的值如果没人再用了，自动从字典中消失。

```python
import weakref

class ExpensiveData:
    def __init__(self, key):
        self.key = key
        self.data = f"大块数据 #{key}"

cache = weakref.WeakValueDictionary()

obj = ExpensiveData(1)
cache[1] = obj                 # 存入缓存

cache[1]                       # → 还在！有 obj 引用着
print(cache[1].data)           # → '大块数据 #1'

del obj                        # 删除唯一的外部引用
cache.get(1)                   # → None（缓存自动清理了！）

# 对比普通字典——不会自动清理
normal_cache = {}
obj2 = ExpensiveData(2)
normal_cache[2] = obj2
del obj2
normal_cache[2]                # → 还留着！（浪费内存）
```

**什么场景用？**
- 缓存：存的时候随便存，用的人不用了自动清掉
- 无需手动管理缓存大小

## pprint 模块

> **一句话**：`pprint`（pretty-print）让 Python 对象**打印得更漂亮**——嵌套结构自动换行、缩进，一眼能看懂。

### 对比 print 和 pprint

```python
from pprint import pprint

data = {
    'config': {
        'name': 'test',
        'deep': {
            'a': 1,
            'b': 2,
            'c': [1, 2, 3, 4, 5, 6]
        }
    }
}

# print 把所有东西塞在一行
print(data)
# → {'config': {'name': 'test', 'deep': {'a': 1, 'b': 2, 'c': [1, 2, 3, 4, 5, 6]}}}

# pprint 自动格式化
pprint(data)
# {'config': {'deep': {'a': 1, 'b': 2, 'c': [1, 2, 3, 4, 5, 6]},
#             'name': 'test'}}
```

### 常用参数

```python
# depth — 限制显示深度
pprint(data, depth=2)        # 只展开 2 层，再深就用 ...

# indent — 缩进空格数
pprint(data, indent=4)

# width — 每行最大宽度（超过就换行）
pprint(data, width=40)

# sort_dicts — 按 key 排序（Python 3.8+）
pprint(data, sort_dicts=True)

# 不打印，直接获取字符串
from pprint import pformat
text = pformat(data)          # 把格式化结果存为字符串
```

## timeit 模块

> **一句话**：精确测量一小段 Python 代码的执行时间。比 `time.time()` 手动算更准确。

### 基础用法

```python
import timeit

# 方法1：直接传字符串
timeit.timeit('"-".join(str(n) for n in range(100))', number=10000)
# number: 重复执行 10000 次，返回总时间（秒）

# 方法2：传 lambda 函数
timeit.timeit(lambda: '-'.join(str(n) for n in range(100)), number=10000)

# 方法3：需要导入时用 setup
timeit.timeit(
    'bisect_left(lst, 50)',                    # 要测量的代码
    setup='from bisect import bisect_left; lst = list(range(100))',  # 准备代码
    number=100000
)
```

### 实际应用：对比数据结构性能

```python
import timeit

# 对比 list.pop(0) 和 deque.popleft()
t_list = timeit.timeit(
    'lst.pop(0)',
    'lst = list(range(1000))',
    number=10000
)

t_deque = timeit.timeit(
    'dq.popleft()',
    'from collections import deque; dq = deque(range(1000))',
    number=10000
)

print(f"list.pop(0): {t_list:.4f}s")
print(f"deque.popleft(): {t_deque:.4f}s")
# 结果：deque.popleft() 比 list.pop(0) 快几百倍
# 因为 list.pop(0) 是 O(n)，deque.popleft() 是 O(1)
```

### 最佳实践

```python
import timeit

# ① number 要足够大，让总计时 > 0.2 秒（排除系统噪声）
# ② 用 repeat 取最小值（排除 GC 等干扰）
min(timeit.repeat(
    'code',
    'setup code',
    repeat=5,           # 重复 5 轮
    number=10000        # 每轮 10000 次
))
# 取最小值——最小值更接近真实运行时间
```

## sys 模块

> **一句话**：`sys` 提供 Python**解释器相关的系统功能**——递归深度、大输入读取、版本等信息。

### 最核心的几个功能

#### sys.setrecursionlimit(n) / sys.getrecursionlimit()

**这是什么？** 设置/获取 Python 允许的**递归调用最大深度**。默认只有 **1000**。

**为什么需要改？** 二叉树、DFS、回溯等递归算法很容易超过 1000 层。

```python
import sys

# 查看当前递归限制
sys.getrecursionlimit()          # → 1000（默认）

# 设大一点
sys.setrecursionlimit(100000)    # 设为 10 万

# 例子：二叉树的递归遍历，树高可能超过 1000
def max_depth(root):
    if not root:
        return 0
    return 1 + max(max_depth(root.left), max_depth(root.right))
```

#### sys.stdin.buffer —— 快速读取输入

**这是什么？** 竞赛/算法题中快速读取大量输入的方法。

```python
# ❌ 慢！一行一行读
n = int(input())
arr = list(map(int, input().split()))

# ✓ 快！一次性读取所有输入
import sys
data = sys.stdin.buffer.read().split()
# data 是字节列表 [b'5', b'1', b'2', ...]
it = iter(data)
n = int(next(it))
arr = [int(next(it)) for _ in range(n)]
```

#### sys.intern —— 字符串驻留

**这是什么？** 把字符串放到一个"驻留池"中——相同的字符串只存一份，后续比较直接用指针地址比较。

```python
import sys

# 普通字符串：比较的是内容（O(n) 字符比较）
a = 'hello world!'
b = 'hello world!'   # 注意：长字符串不一定自动驻留
a is b               # → 可能是 False（不同对象）

# 驻留后
a_intern = sys.intern(a)
b_intern = sys.intern(b)
a_intern is b_intern # → True（同一个对象！比较只要 O(1)）

# 实际价值：大量重复字符串时提升性能
tokens = [sys.intern(t) for t in huge_word_list]  # 后续比较极快
```

#### sys.maxsize

**这是什么？** 当前系统能表示的最大整数（通常是 2⁶³-1 = 9223372036854775807）。

```python
import sys

sys.maxsize                     # → 9223372036854775807
# Python 3 的 int 是任意精度，所以这不会限制你的整数大小
# 但 list 索引不能超过这个值
```

#### sys.getsizeof(obj)

**这是什么？** 返回对象的**内存大小**（字节）。

```python
import sys

sys.getsizeof(42)               # → 28（一个 int 占 28 字节）
sys.getsizeof([1, 2, 3])        # → 88（不包含元素本身）
sys.getsizeof('hello')          # → 54
sys.getsizeof(True)             # → 28

# 注意：只返回对象"本身"的大小，不包含引用的其他对象
```

## textwrap 模块

> **一句话**：处理文本的排版——自动换行、缩进、去除缩进。

### 核心函数

#### textwrap.wrap(text, width)

**这是什么？** 把长文本**按指定宽度换行**，返回字符串列表。

```python
import textwrap

text = "This is a very long line of text that needs to be wrapped."
lines = textwrap.wrap(text, width=30)
print(lines)
# → ['This is a very long line of', 'text that needs to be', 'wrapped.']
```

#### textwrap.fill(text, width)

**和 wrap 一样**，但直接返回换行后的完整字符串（而不是列表）。

```python
import textwrap

text = "This is a very long line of text."
print(textwrap.fill(text, width=20))
# This is a very long
# line of text.
```

#### textwrap.indent(text, prefix)

**这是什么？** 给每行文本前面加前缀（比如缩进、注释符号）。

```python
import textwrap

text = "line1\nline2\nline3"
textwrap.indent(text, '>>> ')
# → '>>> line1\n>>> line2\n>>> line3'
```

#### textwrap.dedent(text)

**这是什么？** 去除多行字符串中**公共的缩进**。常用于处理 `"""..."""` 多行字符串。

```python
import textwrap

# 普通的 """ 字符串会保留缩进
code = """
    def hello():
        print('world')
"""
print(repr(code))
# → '\n    def hello():\n        print(\'world\')\n'

# dedent 去除公共缩进
clean = textwrap.dedent(code)
print(clean)
# → def hello():
#       print('world')
```

#### textwrap.shorten(text, width)

**这是什么？** 截断文本到指定宽度，末尾用 `...` 替代。

```python
import textwrap

textwrap.shorten("Hello world, this is a long text", width=20)
# → 'Hello world,...'
```

## 综合：全模块决策树与最佳实践

### 遇到问题时的"查表指南"

```text
问题性质 → 找哪个模块？

数值计算 → math
├── 求最大公约数？ → math.gcd(a, b)
├── 求最小公倍数？ → math.lcm(a, b)
├── 求组合数 C(n,k)？ → math.comb(n, k)
├── 求排列数 P(n,k)？ → math.perm(n, k)
├── 求整数平方根？ → math.isqrt(n)
├── 判断浮点数相等？ → math.isclose(a, b)
└── 角度转弧度？ → math.radians(x)

随机相关 → random
├── 随机整数 (1~6)？ → random.randint(1, 6)
├── 随机选一个？ → random.choice(list)
├── 重量随机选多个？ → random.choices(list, weights=[...], k=n)
├── 无放回抽奖？ → random.sample(list, k)
├── 打乱顺序？ → random.shuffle(list)
└── 可复现的随机？ → random.seed(42)

统计数据 → statistics
├── 平均值？ → mean(data)
├── 中位数（不受极端值影响）？ → median(data)
├── 出现最多的元素？ → mode(data) / multimode(data)
├── 数据分散程度？ → stdev(data) / variance(data)
└── 四分位数？ → quantiles(data)

精确数值 → decimal 或 fractions
├── 算钱（不能有浮点误差）？ → Decimal('0.1')
└── 分数运算（自动约分）？ → Fraction(1, 2)

内存字符串 → io.StringIO
├── 大量字符串拼接？ → StringIO().write(...)
└── 需要"类文件对象"？ → StringIO()

对象复制 → copy
├── 复制一层？ → copy.copy(x)
└── 完全独立复制？ → copy.deepcopy(x)

排序键函数 → operator
├── 按元组第2个元素排序？ → sorted(..., key=itemgetter(1))
├── 按对象属性排序？ → sorted(..., key=attrgetter('name'))
├── 累加求和/求积？ → reduce(add, list)
└── 异或找只出现一次的数？ → reduce(xor, list)

枚举常量 → enum
├── 方向、状态等有限常量？ → class MyEnum(Enum): ...
├── 需要比较大小？ → class MyEnum(IntEnum): ...
└── 需要组合多个选项？ → class MyFlag(Flag): ...

快速数据模型 → dataclasses
├── 定义一个数据持有类？ → @dataclass class Point: ...
├── 需要排序的类？ → @dataclass(order=True)
└── 不可变类？ → @dataclass(frozen=True)

文本解析 → re
├── 查找数字？ → re.findall(r'\d+', text)
├── 替换所有空格？ → re.sub(r'\s+', ' ', text)
├── 按逗号或空格分割？ → re.split(r'[,\s]+', text)
└── 验证邮箱/电话/IP格式？ → re.fullmatch(pattern, text)

弱引用缓存 → weakref
├── 缓存值自动释放？ → WeakValueDictionary()
└── 缓存键自动释放？ → WeakKeyDictionary()

打印调试 → pprint
├── 打印嵌套结构？ → pprint(data)
└── 获取格式化字符串？ → pformat(data)

性能测试 → timeit
├── 比较两种写法哪个快？ → timeit.timeit(...)
└── 获得稳定结果？ → min(timeit.repeat(...))

系统控制 → sys
├── 递归超过1000层报错？ → sys.setrecursionlimit(100000)
├── 快速读输入？ → sys.stdin.buffer.read()
├── 重复字符串比较优化？ → sys.intern(str)
└── 对象占多大内存？ → sys.getsizeof(obj)

文本排版 → textwrap
├── 自动换行？ → textwrap.wrap(text, width=60)
├── 批量缩进？ → textwrap.indent(text, '> ')
└── 去除多行字符串缩进？ → textwrap.dedent(text)
```

### 模块总表

| 模块 | 一句话记住 | 最常用函数 |
|------|-----------|-----------|
| `math` | C 级数学函数，又快又稳 | gcd / isqrt / comb / isclose |
| `random` | 各种随机：抽奖、洗牌、模拟 | randint / choice / shuffle / sample |
| `statistics` | 统计指标：均值、中位数、标准差 | mean / median / stdev / mode |
| `decimal` | 精确算钱，无浮点误差 | Decimal(str) |
| `fractions` | 精确分数，自动约分 | Fraction(a, b) |
| `io.StringIO` | 把字符串当文件操作 | write / getvalue |
| `copy` | 复制对象：浅/深拷贝 | copy / deepcopy |
| `operator` | 把运算符当函数用 | itemgetter / attrgetter / add / xor |
| `enum` | 有名字的常量，比字符串安全 | Enum / IntEnum / Flag / auto |
| `dataclasses` | 自动生成 init/repr/eq 的类 | @dataclass |
| `types` | 快速创建特定类型 | SimpleNamespace / MappingProxyType |
| `typing` | 类型注解，让代码更可读 | List / Optional / Deque |
| `re` | 正则：文本搜索/替换/分割 | search / findall / sub / split |
| `weakref` | 不阻回收的引用，缓存自动清理 | WeakValueDictionary |
| `pprint` | 漂亮打印数据结构 | pprint / pformat |
| `timeit` | 测量代码运行时间 | timeit / repeat |
| `sys` | 解释器系统功能 | setrecursionlimit / stdin.buffer / intern |
| `textwrap` | 文本排版：换行、缩进、截断 | wrap / fill / dedent |

> **最后的话**：本文共介绍了 **18 个标准库模块**，加上主教程的 8 个模块，基本覆盖了 Python 标准库中所有与算法/数据结构相关的内容。不需要一次背完——**记住"哪个模块能解决什么问题"** 比背 API 重要得多。需要用时，翻一翻这篇文档的目录，找到对应的模块再看细节就好。
