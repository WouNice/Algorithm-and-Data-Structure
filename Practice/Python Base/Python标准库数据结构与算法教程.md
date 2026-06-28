# Python 标准库内置数据结构与算法教程

## 内置数据结构基础

### dict — 哈希表

#### 底层逻辑

Python 的 `dict` 是基于 **开放定址哈希表** 实现的。Python 3.6+ 起引入了 **紧凑存储 (Compact Dict)**，3.7+ 正式保证插入顺序。

**核心结构**：

```text
哈希表内部由两个数组组成：
  - entries[] : 按插入顺序存储 (hash, key, value)
  - indices[] : 稀疏索引数组（大小 = 2^ceil(log₂(N))），存储 entry 的位置

查找时：
  hash(key) → mask → index → indices[index] → entry_index → entries[entry_index]
```

- **哈希冲突处理**: 采用 **随机探测 (random probing)**，使用高 5 位 hash 值作为探测步长的扰动因子
- **负载因子**: 约 2/3 时触发扩容（约 2-4 倍），扩容后 **rehash 全部元素**
- **hash 值要求**: key 必须可哈希（`__hash__()` + `__eq__()`），**可变容器不可哈希**

#### 时间复杂度

| 操作 | 平均 | 最坏 |
|------|------|------|
| 查找 `d[k]` | O(1) | O(n) — 罕见哈希冲突 |
| 插入 `d[k]=v` | O(1) amortized | O(n) — 扩容时 |
| 删除 `del d[k]` | O(1) | O(n) |
| 遍历 | O(n) | O(n) |

> 最坏情况在实际中极难触发（Python 对字符串 hash 做了随机种子）

#### 核心特点

| 特性 | 说明 |
|------|------|
| 🔑 O(1) 查找 | 最适合"存在性检查"和"值映射" |
| 📦 有序保留 | Python 3.7+ 保持插入顺序 |
| 🔄 迭代安全 | 迭代时仅允许修改值，**增删键会报错** |
| 🧩 可哈希要求 | key 必须是不可变类型（str, int, tuple, frozenset） |
| ⚡ 内存紧凑 | 相比 Python 3.5 节省 ~50% 内存 |

#### 常用操作速查

```python
d = {'a': 1, 'b': 2}

# 安全读取
d.get('c', 0)             # → 0，不抛 KeyError
d.setdefault('c', []).append(1)  # 不存在则设默认值，返回该值

# 合并
d1 = {'a': 1}; d2 = {'b': 2}
{**d1, **d2}              # 合并（Python 3.5+）
d1 | d2                   # 合并（Python 3.9+）
d1 |= d2                  # 原地合并（Python 3.9+）

# 缺失键兜底
from collections import defaultdict
dd = defaultdict(list)
dd['x'].append(1)          # 自动创建空 list

# 有序性利用
for k, v in d.items():     # 按插入顺序遍历
for k in reversed(d):      # 逆序遍历（Python 3.8+）
```

#### 算法题实战应用

**① 两数之和（LeetCode 1）**
```python
def twoSum(nums, target):
    seen = {}
    for i, v in enumerate(nums):
        complement = target - v
        if complement in seen:       # O(1) 查找
            return [seen[complement], i]
        seen[v] = i
```
> 用 dict 将 O(n²) 暴力法优化为 O(n)

**② 最长连续序列（LeetCode 128）**
```python
def longestConsecutive(nums):
    num_set = set(nums)              # O(1) 存在性检查
    longest = 0
    for n in num_set:
        if n - 1 not in num_set:     # 只从序列起点开始遍历
            length = 1
            while n + length in num_set:
                length += 1
            longest = max(longest, length)
    return longest
```
> 利用 set O(1) 查找 + 起点检查，将 O(n²) 优化为 O(n)

**③ LRU 缓存（LeetCode 146）**
```python
def __init__(self, capacity):
    self.dict = OrderedDict()
    self.capacity = capacity

def get(self, key):
    if key not in self.dict:
        return -1
    self.dict.move_to_end(key)      # O(1) 移动到末尾
    return self.dict[key]

def put(self, key, value):
    if key in self.dict:
        self.dict.move_to_end(key)
    self.dict[key] = value
    if len(self.dict) > self.capacity:
        self.dict.popitem(last=False)  # O(1) 弹出最久未用
```
> `OrderedDict` 的 `move_to_end` + `popitem` 让 LRU 实现极简

### set / frozenset — 集合

#### 底层逻辑

`set` 和 `frozenset` 的底层实现与 `dict` 完全相同 — 也是开放定址哈希表，只是 **只存储 key，不存储 value**。

- `set` 可变：支持 add/remove/discard/pop
- `frozenset` 不可变：可哈希，可作为 dict key 或 set 元素

#### 时间复杂度

| 操作 | 平均 |
|------|------|
| `x in s` | O(1) |
| `s.add(x)` | O(1) amortized |
| `s.remove(x)` | O(1) |
| `s1 \| s2` (并集) | O(len(s1) + len(s2)) |
| `s1 & s2` (交集) | O(min(len(s1), len(s2))) |
| `s1 - s2` (差集) | O(len(s1)) |

#### 核心特点

- **元素必须可哈希**（同 dict key 要求）
- **无序存储**（虽然 Python 3.7+ 有一定的 hash 顺序稳定性，但 **不保证顺序**）
- **去重自动** — 任何需要去重的场景首先考虑 set
- **集合运算**极大地简化了"共同/差异"类问题

#### 算法题实战应用

**① 数组中是否存在重复元素（LeetCode 217）**
```python
def containsDuplicate(nums):
    return len(nums) != len(set(nums))
```

**② 两个数组的交集（LeetCode 349）**
```python
def intersection(nums1, nums2):
    return list(set(nums1) & set(nums2))
```

**③ 最长不重复子串长度（LeetCode 3）— 滑动窗口 + set**
```python
def lengthOfLongestSubstring(s):
    chars = set()
    left = max_len = 0
    for right, c in enumerate(s):
        while c in chars:
            chars.remove(s[left])
            left += 1
        chars.add(c)
        max_len = max(max_len, right - left + 1)
    return max_len
```
> set 在 O(1) 时间维护窗口内字符的存在性

**④ 有效的数独（LeetCode 36）**
```python
def isValidSudoku(board):
    rows = [set() for _ in range(9)]
    cols = [set() for _ in range(9)]
    boxes = [set() for _ in range(9)]
    for i in range(9):
        for j in range(9):
            val = board[i][j]
            if val == '.': continue
            b = (i // 3) * 3 + j // 3
            # set O(1) 查重
            if val in rows[i] or val in cols[j] or val in boxes[b]:
                return False
            rows[i].add(val)
            cols[j].add(val)
            boxes[b].add(val)
    return True
```

### list — 动态数组

#### 底层逻辑

Python `list` 是 **动态数组 (dynamic array)**，内部使用 C 语言的 `PyObject*` 指针数组实现。

```text
实际容量（allocated） > 逻辑长度（ob_size）

  [ PyObj* | PyObj* | PyObj* | ... | unused | unused ]
   ├── ob_size ─┤         ↑
                      allocated
```

**扩容策略**:
- 初始空 list 容量为 0
- 首次添加时分配 4 个槽位
- 之后按 **`new_allocated = (newsize >> 3) + (newsize < 9 ? 3 : 6) + newsize`**
- 大致相当于每次扩容 ~1.125 倍（与典型的 2 倍扩容不同，Python 选用了更省内存的"过度分配公式"）

#### 时间复杂度

| 操作 | 时间复杂度 | 说明 |
|------|-----------|------|
| 索引 `l[i]` | O(1) | 指针偏移直接寻址 |
| 末尾 `append` | O(1) amortized | 均摊后 O(1) |
| 末尾 `pop()` | O(1) | 仅减少 ob_size |
| 开头 `insert(0, x)` | O(n) | 需移动所有元素 |
| 中间 `insert(i, x)` | O(n) | 需移动 i 之后元素 |
| 删除 `remove(x)` | O(n) | 查找 + 移动 |
| 查找 `x in l` | O(n) | 线性扫描 |
| 切片 `l[i:j]` | O(k) | 复制 k 个元素 |
| 排序 `l.sort()` | O(n log n) | Timsort |

#### 核心特点与陷阱

```python
# ⚠️ 常见陷阱
# ① 浅拷贝问题
a = [[0]] * 3     # → [[0], [0], [0]]
a[0][0] = 1       # → [[1], [1], [1]]  ❌ 三个元素是同一个对象

# ② 迭代时修改
for x in lst:
    lst.remove(x) # ❌ 行为不确定，不要这样做

# ③ 列表推导 vs 显式循环（通常更快）
squares = [x**2 for x in range(1000)]        # ✓ Python 内部 C 级执行
squares = []
for x in range(1000): squares.append(x**2)   # 慢得多
```

#### 算法题优化：用索引运算代替链式结构

```python
# 用 list 模拟栈 — 比 collections.deque 更快
stack = []
stack.append(1)   # push — O(1)
stack.pop()       # pop — O(1)
stack[-1]         # peek — O(1)

# ⚠️ 不要用 list 模拟队列！
queue = []
queue.append(1)   # enqueue — O(1)
queue.pop(0)      # dequeue — O(n) ❌ 每次移动所有元素
```

**空间复用技巧** — 二维 list 的原地操作（动态规划常用）：
```python
# 用两个一维 list 代替二维矩阵（节省 O(n) 空间）
prev = [0] * (n + 1)
for i in range(1, m + 1):
    curr = [0] * (n + 1)
    for j in range(1, n + 1):
        if text1[i-1] == text2[j-1]:
            curr[j] = prev[j-1] + 1
        else:
            curr[j] = max(prev[j], curr[j-1])
    prev = curr
```

### tuple — 不可变序列

#### 底层逻辑

`tuple` 是 **不可变的定长数组**。一旦创建，不能修改其内容（不能增删改元素）。底层是简单的 `PyObject*` 数组，**没有扩容逻辑**。

```c
typedef struct {
    PyObject_VAR_HEAD
    PyObject *ob_item[1];  // 实际大小为创建时确定
} PyTupleObject;
```

#### 核心特点

| 特性 | 说明 |
|------|------|
| 🔒 不可变 | 可用作 dict key 和 set 元素 |
| ⚡ 更省内存 | 比相同元素的 list 省 ~30-50% 内存 |
| 🏎️ 创建更快 | 编译器对字面量 tuple 有优化 |
| 📦 解包 | `a, b = (1, 2)` — Python 最优雅的特性之一 |
| 🔄 可哈希 | `hash((1, 2, 3))` 没问题，但 `hash((1, [2], 3))` 不行 |

#### 算法题应用

```python
# ① 作为 dict key 表示坐标/状态 (DP 和 BFS 常用)
dp = {}
dp[(i, j, k)] = min(dp.get((i-1, j, k), INF), ...)

# ② 多值返回的约定
def min_max(lst):
    return min(lst), max(lst)
mini, maxi = min_max(data)

# ③ 具名元组代替简单类
from collections import namedtuple
Point = namedtuple('Point', ['x', 'y'])
p = Point(1, 2)
# 比 class Point 更省内存，比 dict 更快
```

## collections 模块

`collections` 模块扩展了内置的容器类型，提供了针对不同场景优化的数据结构。

### deque — 双端队列

#### 底层逻辑

`deque` 用 **双向链表式分块数组 (doubly-linked list of fixed-size blocks)** 实现。

```text
block_size = 64（每个块可存 64 个元素）

  [left block] ↔ [middle blocks...] ↔ [right block]
   ↑                                    ↑
  left指针                             right指针
```

两端操作均为 **O(1)** ：
- 从头部或尾部添加/删除只影响块指针，**不需要移动元素**
- 当某个块满时，只需分配一个新块
- 当某个块空时，回收该块

#### 时间复杂度

| 操作 | 复杂度 |
|------|--------|
| `append(x)` / `appendleft(x)` | O(1) |
| `pop()` / `popleft()` | O(1) |
| 索引 `d[i]` | O(n) — 从头或尾向目标遍历 |
| `insert(i, x)` | O(n) |
| `remove(x)` | O(n) |
| `rotate(k)` | O(k) — 高效环形旋转 |

> ⚠️ **重要**: deque **不适合随机访问**。需要频繁索引操作的场景用 list。

#### 核心 API

```python
from collections import deque

d = deque([1, 2, 3], maxlen=5)   # 固定长度，超出自动丢弃另一端的元素

d.append(4)          # 右端入队
d.appendleft(0)      # 左端入队
d.pop()              # 右端出队 → 4
d.popleft()          # 左端出队 → 0

d.extend([5, 6])     # 右端批量入队
d.extendleft([-1, 0]) # 左端批量入队（注意顺序反向）

d.rotate(2)          # 环形右移 2 步
d.rotate(-1)         # 左移 1 步

d.index(3, 0, 5)     # 在指定范围查找（O(n)）
d.count(2)           # 计数（O(n)）

d.reverse()          # 原地反转
```

#### 算法题实战应用

**① 滑动窗口最大值（LeetCode 239）**
```python
def maxSlidingWindow(nums, k):
    dq = deque()          # 存索引，保证元素单调递减
    result = []
    for i, v in enumerate(nums):
        # 移除超出窗口左端的索引
        if dq and dq[0] < i - k + 1:
            dq.popleft()
        # 维持递减：弹出所有比 v 小的（它们不可能再成为最大值）
        while dq and nums[dq[-1]] < v:
            dq.pop()
        dq.append(i)
        if i >= k - 1:
            result.append(nums[dq[0]])
    return result
```
> deque 在 O(1) 时间实现"单调队列"，将暴力 O(n·k) 优化为 O(n)

**② 二叉树 zigzag 层序遍历（LeetCode 103）**
```python
def zigzagLevelOrder(root):
    if not root: return []
    q = deque([root])
    result, left_to_right = [], True
    while q:
        level = deque()
        for _ in range(len(q)):
            node = q.popleft()
            if left_to_right:
                level.append(node.val)
            else:
                level.appendleft(node.val)
            if node.left: q.append(node.left)
            if node.right: q.append(node.right)
        result.append(list(level))
        left_to_right = not left_to_right
    return result
```

**③ 设计循环双端队列（LeetCode 641）**
```python
class MyCircularDeque:
    def __init__(self, k):
        self.d = deque(maxlen=k)  # maxlen 自动处理环形覆盖

    def insertFront(self, value):
        if len(self.d) == self.d.maxlen:
            return False
        self.d.appendleft(value)
        return True

    def insertLast(self, value):
        if len(self.d) == self.d.maxlen:
            return False
        self.d.append(value)
        return True

    def deleteFront(self):
        return bool(self.d) and bool(self.d.popleft() or True)

    def deleteLast(self):
        return bool(self.d) and bool(self.d.pop() or True)

    def getFront(self):
        return self.d[0] if self.d else -1

    def getRear(self):
        return self.d[-1] if self.d else -1

    def isEmpty(self):
        return len(self.d) == 0

    def isFull(self):
        return len(self.d) == self.d.maxlen
```

**④ 括号匹配（LeetCode 20）** — 仍用 list 作为栈，但 BFS/DFS 中的队列建议用 deque
```python
def isValid(s):
    stack = []          # 栈用 list 即可
    pairs = {')': '(', ']': '[', '}': '{'}
    for c in s:
        if c in pairs:
            if not stack or stack[-1] != pairs[c]:
                return False
            stack.pop()
        else:
            stack.append(c)
    return not stack
```

### Counter — 计数器

#### 底层逻辑

`Counter` 是 `dict` 的子类。key 为元素，value 为计数。内部纯 Python 实现，没有 C 层优化，但足够高效。

```python
Counter('abracadabra') == {'a': 5, 'b': 2, 'r': 2, 'c': 1, 'd': 1}
```

#### 核心 API

```python
from collections import Counter

# 任何可迭代对象
c = Counter('abracadabra')
c = Counter(['a', 'b', 'a'])
c = Counter({'a': 3, 'b': 1})
c = Counter(a=3, b=1)

# 三种访问方式
c['z']        # → 0（不抛 KeyError！因为 __missing__ 返回 0）
c.most_common(2)     # → [('a', 5), ('b', 2)]
c.total()            # → 11（Python 3.10+，所有计数之和）

# 常用方法
c.elements()         # 按计数展开的迭代器
c.update('abc')      # 增加计数
c.subtract('abc')    # 减少计数

# 集合运算！
c1 = Counter(a=3, b=1)
c2 = Counter(a=1, c=2)
c1 + c2              # → Counter({'a': 4, 'c': 2, 'b': 1})
c1 - c2              # → Counter({'a': 2, 'b': 1})  # 仅保留正数
c1 & c2              # → Counter({'a': 1})          # 交集：min
c1 | c2              # → Counter({'a': 3, 'c': 2, 'b': 1})  # 并集：max
+c1                  # 去除零和负数
-c1                  # 保留负数取绝对值后转为正数

# 迭代更新元素计数
c1 &= c2             # 原地交集
```

#### 算法题实战应用

**① 字符串中第一个唯一字符（LeetCode 387）**
```python
def firstUniqChar(s):
    count = Counter(s)                        # O(n) 计数
    for i, ch in enumerate(s):
        if count[ch] == 1:
            return i
    return -1
```

**② 同字母异序词分组（LeetCode 49）**
```python
def groupAnagrams(strs):
    # Counter 作为 key（需转为 frozenset 或 tuple）
    groups = defaultdict(list)
    for s in strs:
        key = tuple(sorted(Counter(s).items()))  # 或简单地 tuple(sorted(s))
        groups[key].append(s)
    return list(groups.values())
```
> 更优写法：直接用 `tuple(sorted(s))` 作为 key

**③ 根据字符出现频率排序（LeetCode 451）**
```python
def frequencySort(s):
    count = Counter(s)
    # 按频率降序拼接
    return ''.join(ch * cnt for ch, cnt in count.most_common())
```

**④ 总和为 K 的子数组（LeetCode 560）** — 配合前缀和 + dict
```python
def subarraySum(nums, k):
    # prefix sum + 计数
    prefix_counts = Counter({0: 1})
    prefix_sum = result = 0
    for n in nums:
        prefix_sum += n
        result += prefix_counts[prefix_sum - k]
        prefix_counts[prefix_sum] += 1
    return result
```

### defaultdict — 默认值字典

#### 底层逻辑

`defaultdict` 是 `dict` 的子类，重写了 `__missing__` 方法。当访问不存在的 key 时，调用构造函数传入的 `default_factory` 生成默认值。

```python
from collections import defaultdict

dd = defaultdict(list)
dd['a'].append(1)  # 'a' 不存在 → __missing__ → list() → []
```

> ⚠️ `default_factory` 必须为 **可调用对象** 或 `None`。设为 `None` 时行为同普通 dict。

#### 常用形式

```python
defaultdict(list)         # 自动创建空列表
defaultdict(set)          # 自动创建空集合
defaultdict(int)          # 自动创建 0
defaultdict(dict)         # 自动创建空字典
defaultdict(lambda: 0)    # 自定义工厂
defaultdict(lambda: float('inf'))  # 无穷大（图论 DP 常用）
```

#### 算法题应用

```python
# ① 图的邻接表 — 最简写法
graph = defaultdict(list)
for u, v in edges:
    graph[u].append(v)
    graph[v].append(u)    # 无向图

# ② 树的分层遍历
level_nodes = defaultdict(list)
for node, depth in traversal:
    level_nodes[depth].append(node.val)

# ③ 分组
nums = [1, 2, 3, 4, 5, 6]
groups = defaultdict(list)
for n in nums:
    groups[n % 3].append(n)   # → {1: [1,4], 2:[2,5], 0:[3,6]}

# ④ 替代手动检查
d = {}
for k, v in pairs:
    if k not in d:        # ❌ 手动检查
        d[k] = []
    d[k].append(v)

dd = defaultdict(list)     # ✓ 一行搞定
for k, v in pairs:
    dd[k].append(v)
```

### OrderedDict — 有序字典

#### 底层逻辑

Python 3.7+ 普通 `dict` 也保持插入顺序了，`OrderedDict` 还有什么优势？

关键区别在于 **额外的双向链表**：

```text
dict 只记录"插入顺序"（一个简单的顺序数组）
OrderedDict 额外维护了一个双向链表来记录顺序关系
```

这带来了两个普通 dict **没有**的能力：

| 能力 | 方法 | 复杂度 |
|------|------|--------|
| 将 key 移动到末尾 | `move_to_end(key)` | O(1) |
| 弹出首/尾项 | `popitem(last=True/False)` | O(1) |
| 开头的插入顺序 | 链表中保留了"上一个"和"下一个"指针 | O(1) |
| 相等性检查 | 考虑顺序 (`od1 == od2` 要求元素和顺序都一致) | O(n) |

> 普通 dict 的 `popitem()` 在 Python 3.7+ 之后也可以弹出最后一项，但无法弹出第一项（需要 O(n)）。

#### 算法题应用

```python
# LRU Cache — 简洁实现
class LRUCache:
    def __init__(self, capacity: int):
        self.cache = OrderedDict()
        self.cap = capacity

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
        self.cache.move_to_end(key)   # O(1) 移到末尾（最近使用）
        return self.cache[key]

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.cap:
            self.cache.popitem(last=False)  # O(1) 弹出最久未用
```

> 这是 LeetCode 146 标准解法中最简洁、最不容易出错的实现。

### namedtuple — 命名元组

#### 底层逻辑

`namedtuple` 是 **元组的子类**，在运行期动态生成新类。它比普通类更省内存（因为继承自 tuple），且支持元组的所有操作。

```python
from collections import namedtuple

Point = namedtuple('Point', ['x', 'y'])
p = Point(1, 2)

p.x          # → 1（属性访问）
p[0]         # → 1（索引访问）
x, y = p     # → 解包
```

**比普通类好的地方：**
- 内存占用约 1/3（无 `__dict__`）
- 默认实现了 `__repr__`, `__eq__`, `__hash__`
- 可作为 dict key（因为它是 tuple 子类）

**算法题应用：**

```python
# Dijkstra 优先队列中的状态表示
Node = namedtuple('Node', ['dist', 'vertex'])

import heapq
pq = [Node(0, start)]
while pq:
    d, v = heapq.heappop(pq)    # 按 dist 排序
    if d > dist[v]: continue
    for w, wgt in graph[v]:
        nd = d + wgt
        if nd < dist[w]:
            dist[w] = nd
            heapq.heappush(pq, Node(nd, w))

# BFS 状态（比 tuple 更可读）
State = namedtuple('State', ['x', 'y', 'key_mask', 'steps'])
q = deque([State(sx, sy, 0, 0)])
```

### ChainMap — 链式映射

#### 底层逻辑

`ChainMap` 将多个 dict（或映射对象）"链接"在一起，查找时按顺序搜索。

```python
from collections import ChainMap

defaults = {'theme': 'light', 'lang': 'en'}
user_prefs = {'theme': 'dark'}
config = ChainMap(user_prefs, defaults)

config['theme']       # → 'dark'（user_prefs 优先）
config['lang']        # → 'en'（从 defaults 找到）
config['nonexist']    # → KeyError —— 搜索完所有映射仍未找到
```

**特点：**
- 查找 O(m) — m 为链接的映射数，每个映射内 O(1) 哈希查找
- 写操作（修改/新增/删除）只影响 **第一个** 映射
- 底层映射 **不合并**，修改原映射会反映到 ChainMap 中

**算法题应用：**

```python
# 作用域链模拟（编译原理、解释器实现）
current_scope = {'x': 1}
outer_scope = {'y': 2}
scopes = ChainMap(current_scope, outer_scope)

# 嵌套 dict 的扁平化搜索
nested = ChainMap(
    {'a': 1, 'b': 2},
    {'b': 3, 'c': 4}   # b 取第一个映射的值 = 2
)
```

## heapq 模块

### 堆理论

Python 的 `heapq` 实现的是 **最小堆 (min-heap)**，底层用 **列表** 存储一棵完全二叉树：

```text
             heap[0]  ← 最小值
            /       \
       heap[1]    heap[2]
       /    \      /    \
   heap[3] heap[4] heap[5] heap[6]
   ...

父子关系（索引 i 从 0 开始）：
  parent(i)     = (i - 1) // 2
  left_child(i) = 2 * i + 1
  right_child(i)= 2 * i + 2
```

**堆性质**：`heap[i] <= heap[2*i+1]` 且 `heap[i] <= heap[2*i+2]`

### 核心 API

| 函数 | 复杂度 | 说明 |
|------|--------|------|
| `heappush(heap, item)` | O(log n) | 追加元素并上浮 |
| `heappop(heap)` | O(log n) | 弹出最小值并下沉 |
| `heapify(x)` | O(n) | 将任意列表转为合法堆 |
| `heapreplace(heap, item)` | O(log n) | pop 后 push，比先后调用更高效 |
| `heappushpop(heap, item)` | O(log n) | push 后 pop |
| `nlargest(n, iter)` | O(n log k) | 见下文 |
| `nsmallest(n, iter)` | O(n log k) | 见下文 |

**为什么 heapify 是 O(n) 而不是 O(n log n)？**

```text
推导：对高度为 h 的二叉树，每个节点下沉最多需要 h 次
每个高度上的节点数与下沉次数的乘积求和：
  Σ_{h=0}^{log n} (n/2^{h+1}) * h → O(n)
```

#### 基本用法

```python
import heapq

heap = []
heapq.heappush(heap, 5)
heapq.heappush(heap, 1)
heapq.heappush(heap, 3)
heap[0]    # → 1（但不保证 heap[1], heap[2] 的顺序）
heapq.heappop(heap)  # → 1

# 从已有列表建堆
nums = [5, 2, 8, 1, 9]
heapq.heapify(nums)  # O(n) 原地堆化
nums[0]   # → 1

# 最大堆 — Python 没有原生最大堆，用取负
max_heap = []
heapq.heappush(max_heap, (-x, x))  # 存储 (负值, 原值)
heapq.heappop(max_heap)[1]         # 取最大值
```

### 堆排序

```python
def heapsort(iterable):
    heap = list(iterable)
    heapq.heapify(heap)          # O(n)
    return [heapq.heappop(heap) for _ in range(len(heap))]  # O(n log n)

# 最坏 O(n log n)，空间 O(n)
```

> **不是稳定排序**，但 Python 内置 `sorted()` 更快（Timsort, C 实现），所以一般只在需要 **外部排序** 或 **部分排序** 时自己堆排序。

### 合并多个有序序列

```python
# heapq.merge — 合并多个有序迭代器（性能极好，不一次加载全部）
import heapq
list(heapq.merge([1, 3, 5], [2, 4, 6], [0, 7, 8]))
# → [0, 1, 2, 3, 4, 5, 6, 7, 8]
```

**应用场景**：合并多个已排序文件（外部排序），或者合并多个有序链表。

**复杂度**：输出 n 个元素，输入 k 个序列 = O(n log k)

### nlargest / nsmallest

```python
import heapq

data = [5, 2, 8, 1, 9, 3]
heapq.nlargest(3, data)     # → [9, 8, 5]
heapq.nsmallest(3, data)    # → [1, 2, 3]

# 带 key 函数
heapq.nlargest(3, data, key=lambda x: -x)
heapq.nsmallest(3, data, key=lambda x: x)

# 实际应用：找出 Top K
sales = [
    {'name': 'A', 'amount': 500},
    {'name': 'B', 'amount': 800},
    {'name': 'C', 'amount': 300},
    {'name': 'D', 'amount': 900},
]
heapq.nlargest(2, sales, key=lambda x: x['amount'])
```

**底层实现差异：**

| n 大小 | 实现方式 | 复杂度 |
|--------|---------|--------|
| 小 (n < 1% 总元素) | 用大小为 n 的堆 | O(len × log n) |
| 大 | 堆化整个列表再 nlargest | O(len + n log len) |

> **选择建议**: 小 n 用 `nlargest/nsmallest`，大 n 用 `sorted(iterable)[:n]`

#### 算法题实战应用

**① 数组中的第 K 个最大元素（LeetCode 215）**
```python
def findKthLargest(nums, k):
    # 方法 1: 最小堆
    heap = nums[:k]
    heapq.heapify(heap)            # O(k)
    for x in nums[k:]:
        if x > heap[0]:            # 如果比堆顶大，替换
            heapq.heapreplace(heap, x)  # O(log k)
    return heap[0]                 # 堆顶就是第 k 大的
    # 总 O(n log k)，空间 O(k)

    # 方法 2: 直接调用（面试不建议，但可用）
    # return heapq.nlargest(k, nums)[-1]

    # 方法 3: 快速选择（最快）O(n) average
```

**② 前 K 个高频元素（LeetCode 347）**
```python
def topKFrequent(nums, k):
    freq = Counter(nums)          # O(n) 统计频率
    # 最小堆，维护频率最高的 k 个
    heap = []
    for num, cnt in freq.items():
        if len(heap) < k:
            heapq.heappush(heap, (cnt, num))
        elif cnt > heap[0][0]:    # 频率比堆顶高就替换
            heapq.heapreplace(heap, (cnt, num))
    return [num for _, num in heap]
```
> O(n log k) 时间，O(n + k) 空间

**③ 合并 K 个升序链表（LeetCode 23）**
```python
def mergeKLists(self, lists):
    heap = []
    # 将所有头节点入堆
    for i, node in enumerate(lists):
        if node:
            heapq.heappush(heap, (node.val, i, node))
    dummy = cur = ListNode(0)
    while heap:
        val, i, node = heapq.heappop(heap)    # 弹出最小
        cur.next = node
        cur = cur.next
        if node.next:
            heapq.heappush(heap, (node.next.val, i, node.next))
    return dummy.next
```

**④ 数据流中的中位数（LeetCode 295）** — 双堆技巧
```python
class MedianFinder:
    def __init__(self):
        self.small = []   # 最大堆（存负数）
        self.large = []   # 最小堆

    def addNum(self, num):
        # 先入最大堆，再平衡
        heapq.heappush(self.small, -num)
        # 保证 small 所有元素 <= large 所有元素
        if self.small and self.large and (-self.small[0]) > self.large[0]:
            val = -heapq.heappop(self.small)
            heapq.heappush(self.large, val)
        # 平衡大小
        if len(self.small) > len(self.large) + 1:
            heapq.heappush(self.large, -heapq.heappop(self.small))
        elif len(self.large) > len(self.small):
            heapq.heappush(self.small, -heapq.heappop(self.large))

    def findMedian(self):
        if len(self.small) > len(self.large):
            return -self.small[0]
        return (-self.small[0] + self.large[0]) / 2
```
> 双堆法（两个优先级队列）是最经典的在线中位数做法，见 LeetCode 295

**⑤ 最小 K 个数（剑指 Offer 40/LCR 159）**
```python
def smallestK(arr, k):
    if k == 0: return []
    # 最大堆（存负值）
    heap = [-x for x in arr[:k]]
    heapq.heapify(heap)
    for x in arr[k:]:
        if x < -heap[0]:
            heapq.heapreplace(heap, -x)
    return [-x for x in heap]
```

## bisect 模块

### 核心 API

`bisect` 模块提供 **二分查找** 和 **有序插入**，底层用 C 实现，效率极高。

```python
import bisect

# arr 必须是有序的（升序）
arr = [1, 3, 5, 7, 9]

# 查找插入位置
bisect.bisect_left(arr, 5)    # → 2  （第一个 >= 5 的位置）
bisect.bisect_right(arr, 5)   # → 3  （第一个 > 5 的位置，即插入后 5 的右侧）
bisect.bisect(arr, 5)         # 同 bisect_right

# 插入元素到有序序列
bisect.insort_left(arr, 4)    # arr → [1, 3, 4, 5, 7, 9]
bisect.insort_right(arr, 6)   # arr → [1, 3, 4, 5, 6, 7, 9]
bisect.insort(arr, 6)         # 同 insort_right
```

所有函数均可传入 `lo` 和 `hi` 参数限定范围（与切片类似，但 **不创建新列表**）。

### 手写变体

在算法题中，`bisect_left` 是最重要的 — 手动实现：

```python
def lower_bound(arr, target):
    """返回第一个 >= target 的位置（同 bisect_left）"""
    lo, hi = 0, len(arr)
    while lo < hi:
        mid = (lo + hi) // 2
        if arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid
    return lo

def upper_bound(arr, target):
    """返回第一个 > target 的位置（同 bisect_right）"""
    lo, hi = 0, len(arr)
    while lo < hi:
        mid = (lo + hi) // 2
        if arr[mid] <= target:
            lo = mid + 1
        else:
            hi = mid
    return lo
```

### 算法题实战应用

**① 最长递增子序列（LeetCode 300）**
```python
def lengthOfLIS(nums):
    tails = []                    # tails[i] = 长度为 i+1 的 LIS 的最小末尾值
    for x in nums:
        i = bisect_left(tails, x)  # 找到第一个 >= x 的位置
        if i == len(tails):
            tails.append(x)       # x 比所有末尾都大，扩展 LIS
        else:
            tails[i] = x          # 替换掉该位置的值（保持 tails 递增）
    return len(tails)
```
> 这是经典的"耐心排序 (Patience Sorting)"解法，O(n log n)，bisect 让实现极其简洁

**② 搜索旋转排序数组（LeetCode 33）**
```python
def search(nums, target):
    # 核心：二分 + 判断有序区间
    lo, hi = 0, len(nums) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if nums[mid] == target:
            return mid
        # 左半段是否有序？
        if nums[lo] <= nums[mid]:
            if nums[lo] <= target < nums[mid]:   # target 在左段中
                hi = mid - 1
            else:
                lo = mid + 1
        else:  # 右半段有序
            if nums[mid] < target <= nums[hi]:
                lo = mid + 1
            else:
                hi = mid - 1
    return -1
```

**③ 在排序数组中查找元素的第一个和最后一个位置（LeetCode 34）**
```python
def searchRange(nums, target):
    left = bisect_left(nums, target)
    if left == len(nums) or nums[left] != target:
        return [-1, -1]
    right = bisect_right(nums, target) - 1
    return [left, right]
```

**④ 有序数组中的单一元素（LeetCode 540）**
```python
def singleNonDuplicate(nums):
    lo, hi = 0, len(nums) - 1
    while lo < hi:
        mid = (lo + hi) // 2
        # 保持 mid 为偶数下标
        if mid % 2 == 1:
            mid -= 1
        if nums[mid] == nums[mid + 1]:
            lo = mid + 2        # 单一元素在右半
        else:
            hi = mid            # 单一元素在左半
    return nums[lo]
```

**⑤ 查找峰值元素 II + 二维二分搜索**

二维矩阵中的二分搜索（LeetCode 74 — 搜索二维矩阵）：
```python
def searchMatrix(matrix, target):
    # 将二维坐标映射为一维坐标
    m, n = len(matrix), len(matrix[0])
    lo, hi = 0, m * n - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        val = matrix[mid // n][mid % n]
        if val == target:
            return True
        elif val < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return False
```

**⑥ num 的平方根（LeetCode 69）— 二分模板**
```python
def mySqrt(x):
    lo, hi = 0, x
    while lo <= hi:
        mid = (lo + hi) // 2
        if mid * mid <= x:
            lo = mid + 1
        else:
            hi = mid - 1
    return hi   # 返回最后一个满足 mid² ≤ x 的值
```

## 排序算法

### sorted() 与 list.sort()

| 函数 | 返回 | 原地 | 可排序类型 |
|------|------|------|-----------|
| `sorted(iterable)` | 新列表 | ✗ | 任何可迭代对象 |
| `list.sort()` | `None` | ✓ | 仅 list |

```python
sorted([3, 1, 2])                # → [1, 2, 3]
sorted({3, 1, 2})                # set 也支持
sorted("cba")                    # → ['a', 'b', 'c']
sorted(enumerate(['c','a','b'])) # 按索引排序！

# 逆序
sorted([3, 1, 2], reverse=True)

# key 参数（关键技巧）
sorted(['abc', 'a', 'ab'], key=len)          # → ['a', 'ab', 'abc']
sorted(['3', '10', '2'], key=int)            # → ['2', '3', '10']

# 多键排序
students = [
    ('Alice', 'B', 20),
    ('Bob', 'A', 20),
    ('Charlie', 'A', 22),
]
sorted(students, key=lambda x: (x[1], x[2]))  # 按年级→年龄
```

### Timsort 工作原理

Python 内置排序使用的是 **Timsort** — 一种混合稳定排序算法，由 Tim Peters 于 2002 年为 Python 设计（后来被 Java、Android 等采用）。

```text
核心思想：检测数据中的"自然有序区间 (run)"并合并

流程：
1. 从左到右扫描，找出自然有序（升序或严格降序）的子序列（run）
2. 每个 run 的长度若小于 minrun（32-64），用二分插入排序补足
3. 将 run 压入栈，维持"不变量"——保证栈中 run 的长度递减
4. 用归并排序合并相邻 run
```

**为什么 Timsort 快？**
- **部分有序数据**: 检测到自然有序段后 O(n) 即可完成
- **O(n log n) 最坏保证**
- **稳定排序** — 相等元素保持原始相对顺序
- **自适应**: 完全随机约 O(n log n)，\n完全有序约 O(n)

```python
# Timsort 的稳定性质在复合排序中很重要
data = [('A', 2), ('A', 1), ('B', 1)]
sorted(data, key=lambda x: x[0])  # 先按字母排
sorted(data, key=lambda x: x[1])  # 再按数字排 — 稳定排序保持字母顺序
# → [('A', 1), ('A', 2), ('B', 1)]
```

### 自定义排序键

#### key vs cmp

Python 3 已移除 `cmp` 参数（Python 2 的 `sorted(..., cmp=fn)` 已废弃）。

**所有自定义排序都通过 key 实现**（转换后比较，效率远高于每次比较调用 cmp）：

```python
# ❌ 错误：Python 3 不支持 cmp
# sorted(data, cmp=lambda a, b: a - b)

# ✓ 正确：用 key 转换
# 天然支持的内置 key
sorted(strings, key=str.lower)       # 忽略大小写
sorted(dates, key=lambda d: d['time'])  # 按字典字段

# 复合排序 = (primary_key, secondary_key)
sorted(points, key=lambda p: (p.x, p.y))

# 一个字段升序一个降序
sorted(points, key=lambda p: (p.x, -p.y))  # x 升序 → y 降序
```

#### functools.cmp_to_key 兼容旧接口

```python
from functools import cmp_to_key

def compare(a, b):
    # 返回 负数(<) / 0(=) / 正数(>)
    return a - b

sorted(nums, key=cmp_to_key(compare))
```

> 仅用于兼容旧代码。生产环境应尽量使用 key 函数（更快）。

### 算法题中的排序优化技巧

**① 合并区间（LeetCode 56）**
```python
def merge(intervals):
    intervals.sort(key=lambda x: x[0])  # 按起始位置排序
    merged = []
    for interval in intervals:
        if not merged or interval[0] > merged[-1][1]:
            merged.append(interval)
        else:
            merged[-1][1] = max(merged[-1][1], interval[1])
    return merged
```

**② 最大数（LeetCode 179）**
```python
def largestNumber(nums):
    # 自定义排序：判断 x+y 与 y+x 的大小
    strs = list(map(str, nums))
    # 自定义比较器
    def compare(a, b):
        if a + b > b + a: return -1
        return 1 if a + b < b + a else 0
    strs.sort(key=cmp_to_key(compare))
    result = ''.join(strs).lstrip('0')
    return result or '0'
```

**③ 会议室 II（LeetCode 253）— 排序 + 最小堆（优先队列）**
```python
def minMeetingRooms(intervals):
    if not intervals: return 0
    intervals.sort(key=lambda x: x[0])  # 按开始时间排序
    heap = []                            # 存结束时间
    for start, end in intervals:
        if heap and start >= heap[0]:    # 最早结束的会议室可用
            heapq.heapreplace(heap, end) # 复用
        else:
            heapq.heappush(heap, end)    # 新开会议室
    return len(heap)
```

**④ 最接近的三数之和（LeetCode 16）— 排序 + 双指针**
```python
def threeSumClosest(nums, target):
    nums.sort()
    closest = sum(nums[:3])
    for i in range(len(nums) - 2):
        lo, hi = i + 1, len(nums) - 1
        while lo < hi:
            s = nums[i] + nums[lo] + nums[hi]
            if abs(s - target) < abs(closest - target):
                closest = s
            if s < target:
                lo += 1
            elif s > target:
                hi -= 1
            else:
                return target
    return closest
```

**⑤ 插入区间（LeetCode 57）— 二分查找 + 插入**
```python
def insert(intervals, newInterval):
    # 用 bisect 找到插入位置
    i = bisect_left(intervals, newInterval, key=lambda x: x[0])
    intervals.insert(i, newInterval)  # O(n) 插入
    # 然后执行合并逻辑
    ...
```

## queue 模块

`queue` 模块主要提供 **线程安全** 的队列实现。在单线程算法题中，更推荐用 `collections.deque`（更快），但在 **多线程** 或多进程场景中必须用 `queue.Queue`。

### Queue — FIFO 队列

```python
from queue import Queue

q = Queue(maxsize=10)    # maxsize=0 表示无上限

q.put(item)              # 入队 — 若队列已满则阻塞
q.put(item, block=False) # 不阻塞，满则抛 Full
q.put(item, timeout=1)   # 最多等 1 秒

item = q.get()           # 出队 — 若队列为空则阻塞
item = q.get(block=False)# 不阻塞，空则抛 Empty
item = q.get(timeout=1)  # 最多等 1 秒

q.qsize()                # 当前大小（近似值，多线程环境下不精确）
q.empty()                # 是否为空
q.full()                 # 是否已满
q.task_done()            # 任务完成通知
q.join()                 # 等待所有任务完成
```

**线程安全机制：**
- 内部使用 `threading.Lock` 进行互斥
- 有条件的 `threading.Condition` 实现阻塞/通知
- **算法题中不要用**（有锁开销），用 `deque` 代替

### LifoQueue — 栈

```python
from queue import LifoQueue

stack = LifoQueue()
stack.put(1)
stack.put(2)
stack.get()  # → 2（后进先出）
```

> 算法题中直接用 `list` 做栈（`append/pop` O(1)），无需用 `LifoQueue`。

### PriorityQueue — 优先队列

```python
from queue import PriorityQueue

pq = PriorityQueue()
pq.put((2, 'code'))
pq.put((1, 'eat'))
pq.put((3, 'sleep'))

pq.get()  # → (1, 'eat')
```

**底层:** 内部使用 `heapq` 实现，但用锁包装了线程安全。

> 单线程算法题中应直接用 `heapq`，`PriorityQueue` 有锁开销。

## itertools 模块

`itertools` 是算法题的 **秘密武器** — 许多看似复杂的组合/排列/迭代问题可以用它一行搞定。

### 无限迭代器

| 迭代器 | 参数 | 说明 |
|--------|------|------|
| `count(start, step)` | start, [step] | 无限等差数列 |
| `cycle(iterable)` | iterable | 无限循环 |
| `repeat(elem, [n])` | elem, [n] | 无限重复（可限制次数） |

```python
from itertools import count, cycle, repeat

# 计数器
for i in count(10, 2):  # 10, 12, 14, ...
    if i > 20: break

# 循环
for c in cycle('ABC'):  # A, B, C, A, B, C, ...
    if ...: break

# 重复
list(repeat(0, 5))  # → [0, 0, 0, 0, 0]
```

### 有限迭代器

| 迭代器 | 参数 | 返回 | 类似 |
|--------|------|------|------|
| `accumulate(iter)` | iter, [func] | 累积和/累积运算 | `reduce` 的中间结果 |
| `chain(*iters)` | 多个可迭代对象 | 串联 | `a + b` 的惰性版 |
| `chain.from_iterable(iters)` | 可迭代的可迭代对象 | 扁平化 | 嵌套展开 |
| `compress(data, selectors)` | data, selectors | 按选择器筛选 | `filter` 的布尔版 |
| `dropwhile(pred, seq)` | pred, seq | 跳过前缀 | 类似 skipwhile |
| `takewhile(pred, seq)` | pred, seq | 截取前缀 | |
| `filterfalse(pred, seq)` | pred, seq | 反过滤 | |
| `groupby(iter, key)` | iter, [key] | 相邻分组 | 见下文 |
| `islice(iter, [start,] stop, [step])` | | 切片 | |
| `pairwise(iter)` | iter | 相邻对 **(3.10+)** | |
| `starmap(func, iter)` | func, iter | `*args` 调用 | |
| `tee(iter, n=2)` | iter, n | 克隆 n 个独立迭代器 | |
| `zip_longest(*iters, fillvalue)` | | 长 zip | |

#### ⭐ groupby — 最重要的 itertools 函数之一

```python
from itertools import groupby

data = [('A', 1), ('A', 2), ('B', 1), ('B', 3)]
groups = groupby(data, key=lambda x: x[0])

for key, group in groups:
    print(key, list(group))
# A [('A', 1), ('A', 2)]
# B [('B', 1), ('B', 3)]

# ⚠️ groupby 要求输入已按 key 排序！
# 错误用法：
data = [('A', 1), ('B', 2), ('A', 3)]
groups = groupby(data, key=lambda x: x[0])
for key, group in groups:
    print(key, list(group))
# A [('A', 1)]   ← 第二个 'A' 不在这里！
# B [('B', 2)]
# A [('A', 3)]   ← 它跑到了这个分组
```

#### pairwise (Python 3.10+)

```python
from itertools import pairwise

list(pairwise([1, 2, 3, 4]))  # → [(1, 2), (2, 3), (3, 4)]
# 等价于 zip(seq, seq[1:])，但更高效（惰性）
```

### 组合生成器

| 函数 | 返回数量 | 说明 |
|------|---------|------|
| `product('ABCD', repeat=2)` | nʳ | 笛卡尔积 |
| `permutations('ABCD', 2)` | P(n, r) = n!/(n-r)! | 排列 |
| `combinations('ABCD', 2)` | C(n, r) = n!/r!(n-r)! | 组合 |
| `combinations_with_replacement('ABCD', 2)` | C(n+r-1, r) | 可重复组合 |

```python
from itertools import product, permutations, combinations

# 笛卡尔积
list(product('AB', repeat=2))   # → [('A','A'),('A','B'),('B','A'),('B','B')]
list(product('AB', '12'))       # → [('A','1'),('A','2'),('B','1'),('B','2')]

# 排列
list(permutations('ABC', 2))  # → [('A','B'),('A','C'),('B','A'),('B','C'),('C','A'),('C','B')]

# 组合
list(combinations('ABC', 2))  # → [('A','B'),('A','C'),('B','C')]

# 可重复组合
list(combinations_with_replacement('ABC', 2))
# → [('A','A'),('A','B'),('A','C'),('B','B'),('B','C'),('C','C')]
```

### 算法题实战技巧

**① 子集枚举（回溯 vs itertools）— LeetCode 78**
```python
from itertools import combinations

def subsets(nums):
    res = []
    for r in range(len(nums) + 1):
        for combo in combinations(nums, r):  # 一行搞定子集
            res.append(list(combo))
    return res
```

**② 全排列（LeetCode 46）**
```python
from itertools import permutations

def permute(nums):
    return [list(p) for p in permutations(nums)]
```

**③ 电话号码字母组合（LeetCode 17）**
```python
def letterCombinations(digits):
    if not digits: return []
    mapping = {
        '2': 'abc', '3': 'def', '4': 'ghi', '5': 'jkl',
        '6': 'mno', '7': 'pqrs', '8': 'tuv', '9': 'wxyz'
    }
    # product(*[mapping[d] for d in digits]) 笛卡尔积
    return [''.join(combo) for combo in product(*(mapping[d] for d in digits))]
```

**④ 四数之和 / N 数之和的超暴力暴力法（LeetCode 18）** — 组合 + 求和
```python
def fourSum(nums, target):
    if len(nums) < 4: return []
    # 仅用于理解问题（实际用排序+双指针更优）
    # 但 itertools 在 N 很小时代码极短
    nums.sort()
    return set(
        tuple(sorted(combo))
        for combo in combinations(nums, 4)
        if sum(combo) == target
    )
    # 转 set 去重后再转 list
```

**⑤ 最长湍流子数组（LeetCode 978）— pairwise + groupby**
```python
def maxTurbulenceSize(arr):
    if len(arr) < 2: return len(arr)
    # 将相邻元素的差值符号转为标识：1 升, -1 降, 0 平
    diffs = []
    for a, b in pairwise(arr):
        if a < b: diffs.append(1)
        elif a > b: diffs.append(-1)
        else: diffs.append(0)
    # 找最长的交替符号序列
    max_len = 1
    cur = 1
    for a, b in pairwise(diffs):
        if a * b < 0:  # 一正一负
            cur += 1
            max_len = max(max_len, cur)
        else:
            cur = 1
    return max_len + 1  # 差值个数+1=数组长度
```

**⑥ 用 accumulate 实现前缀和 / 扫描线**
```python
from itertools import accumulate

# 前缀和 — O(n) 一行代码
nums = [1, 2, 3, 4]
list(accumulate(nums))  # → [1, 3, 6, 10]

# 自定义累积
list(accumulate([1, 2, 3, 4], lambda a, b: a * b))  # → [1, 2, 6, 24]

# 扫描线技巧：用 accumulate 模拟时间区间叠加
changes = {
    1: +1,   # 区间开始
    3: -1,   # 区间结束
    5: +1,
    7: -1,
}
sorted_times = sorted(changes.items())
counts = [count for _, count in sorted_times]
list(accumulate(counts))  # → [1, 0, 1, 0] 每个时间点的区间数
```

**⑦ 扁平化嵌套列表**
```python
from itertools import chain

nested = [[1, 2], [3, [4, 5]], [6]]
flat = list(chain.from_iterable(nested))
# → [1, 2, 3, [4, 5], 6]  （只展开一层）

# 完全扁平化需要递归
def deep_flatten(lst):
    for item in lst:
        if isinstance(item, (list, tuple)):
            yield from deep_flatten(item)
        else:
            yield item
```

## functools 模块

### lru_cache — 记忆化搜索

`@functools.lru_cache` 是解决 **重叠子问题** 的最强工具，可以 **零改动** 将指数级暴力搜索优化为 O(n) DP。

#### 底层逻辑

```text
实现原理：
  - 内部使用 dict 缓存 (args, kwargs) → return_value 的映射
  - 使用 LRU 淘汰策略（删除最久未访问的条目）
  - 默认 maxsize=128，设为 None 则无限缓存

关键细节：
  - 参数必须可哈希（list 不能直接用，需转 tuple）
  - 函数必须没有副作用（纯函数）
  - 缓存基于调用参数，不影响外部状态
```

#### 用法

```python
from functools import lru_cache

@lru_cache(maxsize=None)  # 无限缓存
def fib(n):
    if n < 2: return n
    return fib(n - 1) + fib(n - 2)

# 调用后，缓存自动生效
fib(30)   # 832040 — 瞬间返回
fib(100)  # 354224848179261915075 — 递归深度问题（默认 1000）

# 查看缓存信息
fib.cache_info()  # CacheInfo(hits=..., misses=..., maxsize=None, currsize=...)

# 清除缓存
fib.cache_clear()
```

#### 算法题实战

**① 斐波那契数列（LeetCode 509）**
```python
@lru_cache(maxsize=None)
def fib(n):
    if n < 2: return n
    return fib(n - 1) + fib(n - 2)
```
> 比迭代写法更短，且复杂度同为 O(n)

**② 爬楼梯（LeetCode 70）**
```python
@lru_cache(maxsize=None)
def climbStairs(n):
    if n <= 2: return n
    return climbStairs(n - 1) + climbStairs(n - 2)
```

**③ 爬楼梯最小花费（LeetCode 746）**
```python
@lru_cache(maxsize=None)
def minCostClimbingStairs(cost):
    n = len(cost)
    @lru_cache(maxsize=None)
    def dp(i):
        if i < 2: return cost[i]
        return cost[i] + min(dp(i - 1), dp(i - 2))
    return min(dp(n - 1), dp(n - 2))
```

**④ 不同的二叉搜索树（LeetCode 96）**
```python
@lru_cache(maxsize=None)
def numTrees(n):
    if n <= 1: return 1
    total = 0
    for i in range(1, n + 1):
        left = numTrees(i - 1)
        right = numTrees(n - i)
        total += left * right
    return total
```
> 自顶向下 DP，比自底向上更直观

**⑤ 编辑距离（LeetCode 72）— 二维记忆化搜索**
```python
def minDistance(word1, word2):
    @lru_cache(maxsize=None)
    def dp(i, j):
        if i == -1: return j + 1       # 插入所有剩余字符
        if j == -1: return i + 1       # 删除所有剩余字符
        if word1[i] == word2[j]:
            return dp(i - 1, j - 1)    # 相等，跳过
        return 1 + min(
            dp(i - 1, j),              # 删除 word1[i]
            dp(i, j - 1),              # 插入 word2[j]
            dp(i - 1, j - 1)           # 替换
        )
    return dp(len(word1) - 1, len(word2) - 1)
```
> 通过 `lru_cache` 自动 DP，无需手动构造二维表

**⑥ 最长回文子序列（LeetCode 516）**
```python
def longestPalindromeSubseq(s):
    @lru_cache(maxsize=None)
    def dp(i, j):
        if i > j: return 0
        if i == j: return 1
        if s[i] == s[j]:
            return dp(i + 1, j - 1) + 2
        return max(dp(i + 1, j), dp(i, j - 1))
    return dp(0, len(s) - 1)
```

**⚠️ lru_cache 的局限与替代：**
- 递归深度限制（默认 1000），深层递归需 `sys.setrecursionlimit()`
- 装填参数时 tuple 化有开销，大状态时可能不如自底向上
- Python 3.9+ 提供 `@functools.cache` — 等价于 `@lru_cache(maxsize=None)`，更简洁

### cmp_to_key — 自定义比较

```python
from functools import cmp_to_key

# 示例：按多种规则排序
def complex_cmp(a, b):
    # 返回负数表示 a < b，正数 a > b，0 相等
    if a['priority'] != b['priority']:
        return a['priority'] - b['priority']  # 优先级升序
    if a['score'] != b['score']:
        return b['score'] - a['score']        # 分数降序
    return 0

sorted(items, key=cmp_to_key(complex_cmp))
```

**算法题应用—最大数（LeetCode 179）** 见 _排序算法_ 章节中的自定义排序键部分。

### reduce — 归约操作

```python
from functools import reduce

# 等价于: (((1+2)+3)+4)
reduce(lambda a, b: a + b, [1, 2, 3, 4])  # → 10

# 带初始值
reduce(lambda a, b: a * b, [1, 2, 3], 10) # → 60

# 可迭代对象为空时带初始值不报错
reduce(max, [], 0)     # → 0
reduce(max, [1, 2, 3]) # → 3
```

**算法题应用：**

```python
# ① 从 int 数组还原数字
reduce(lambda a, b: a * 10 + b, [1, 2, 3, 4])  # → 1234

# ② 字典合并
reduce(lambda d, k: d | {k: k**2}, [1, 2, 3], {})  # → {1:1, 2:4, 3:9}

# ③ XOR 查找只出现一次的数字（LeetCode 136）
reduce(lambda a, b: a ^ b, [4, 1, 2, 1, 2])  # → 4
```

### total_ordering — 自动补全比较运算

```python
from functools import total_ordering

@total_ordering
class Point:
    def __init__(self, x, y):
        self.x, self.y = x, y
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y
    def __lt__(self, other):
        return (self.x, self.y) < (other.x, other.y)
    # __le__, __gt__, __ge__ 自动生成

p1, p2 = Point(1, 2), Point(2, 1)
p1 < p2   # → True
p1 >= p2  # → False（自动生成！）
```

## array / struct / memoryview

### array — 类型化数组

`array` 是 **固定类型** 的动态数组，比 list 更省内存。

```python
from array import array

# 'i' = signed int, 'f' = float, 'd' = double 等
arr = array('i', [1, 2, 3, 4])
arr.append(5)
arr[0] = 10

# 类型化数组只有 list 约 1/4 的内存（C 级连续存储）
# 但 Python 界很少用——大多数场景 list 已经够快

# 唯一不可替代的场景：需要 C 级二进制兼容的数据
array('B', b'hello')  # 字节数组
```

**算法题中通常不需要**，仅在内存极其受限时考虑（竞赛中的大数组场景）。

### struct — 字节打包/解包

```python
import struct

# 打包 Python 对象 → 字节串
packed = struct.pack('>I4sh', 7, b'spam', 8)
# > = Big-endian, I = unsigned int, 4s = 4字节字符串, h = short

# 解包 字节串 → Python 对象
struct.unpack('>I4sh', packed)
# → (7, b'spam', 8)

# 计算打包后的字节大小
struct.calcsize('>I4sh')  # → 10（4+4+2）
```

**应用场景**：处理二进制协议、文件格式解析（BMP/PNG 等）、网络包解析。

### memoryview — 零拷贝内存视图

```python
data = bytearray(b'hello world')
view = memoryview(data)

# 切片不复制！
slice = view[6:11]      # 零拷贝
slice[0] = ord(b'W')    # 修改视图会反映到原 buffer
data                    # → bytearray(b'hello World')

# 与 struct 配合解析二进制
import struct
buf = memoryview(data)
struct.unpack_from('>5s', buf, 0)  # 不复制字节
```

## 综合实战：算法题优化路线

### 选择数据结构的决策树

```text
问题 → 需要什么操作？

1. 查找/映射 → dict (O(1) key look up)
   ├── 需要有序 → OrderedDict / dict 3.7+
   ├── 需要默认值 → defaultdict
   ├── 需要计数 → Counter
   ├── key = 多个值组合 → tuple 作为 key
   └── 需要存在性检查 → set

2. 序列操作 → list vs deque vs heapq
   ├── 仅末尾操作（栈） → list
   ├── 两端操作（队列/BFS） → deque
   ├── 需要优先级 → heapq（或用 PriorityQueue 多线程）
   ├── 需要排序顺序 → sorted + bisect
   ├── 需要有序插入 → bisect.insort
   └── 需要去重 → set

3. 迭代/枚举 → itertools
   ├── 子集 → combinations
   ├── 排列 → permutations
   ├── 笛卡尔积 → product
   ├── 相邻分组 → groupby + sorted
   └── 累积 → accumulate

4. 递归重叠子问题 → lru_cache / @cache

5. 有序性查询/搜索 → bisect
   ├── 精确查找 (== target) → bisect_left + 值检查
   ├── 第一个 >= target → bisect_left
   ├── 第一个 > target  → bisect_right
   └── 范围查找 → 两个 bisect
```

### 经典题型与最佳模块组合

| 题型 | 核心数据结构 | 辅助模块 | 典型 LeetCode |
|------|-------------|---------|---------------|
| 两数之和类 | `dict` | `enumerate` | 1, 167, 170 |
| 前缀和 | `dict` / `Counter` | `accumulate` | 560, 525, 523 |
| 滑动窗口 | `deque` / `dict` | `bisect` | 239, 3, 76 |
| 堆/优先队列 | `heapq` | `Counter` | 215, 347, 23, 295 |
| 图论 BFS | `deque` | `defaultdict` | 127, 126, 200 |
| 图论 Dijkstra | `heapq` | `defaultdict`, `namedtuple` | 743, 787, 1514 |
| 二分查找 | `bisect` | — | 33, 34, 300, 540, 74 |
| 回溯/DFS | list / 递归 | `lru_cache` | 46, 78, 51, 22 |
| 排序优化 | `list.sort` / `sorted` | `cmp_to_key` | 56, 179, 253 |
| 动态规划 | `lru_cache` | `accumulate` | 72, 96, 516, 5 |
| 双指针 | list / str | `bisect` | 11, 15, 16, 42 |
| 树/递归 | deque / list | `defaultdict` | 103, 94, 105 |
| 区间问题 | list | `bisect`, `heapq` | 56, 57, 253 |
| 字符串 | `Counter` | `defaultdict` | 49, 387, 451, 76 |

### 总结：用标准库而非自己造轮子

许多算法面试者犯的最大错误是 **自己实现本已在标准库中的数据结构**。两个原则：

1. **标准库的 C 实现比你手写的 Python 快 10-100 倍**
2. **标准库的 API 经过充分测试，边界条件已处理妥当**

```python
# ❌ 不要自己实现
class MyBinarySearch:
    def search(self, arr, target):
        lo, hi = 0, len(arr) - 1
        while lo <= hi:
            mid = (lo + hi) // 2
            ...

# ✓ 直接用标准库
i = bisect_left(arr, target)
if i < len(arr) and arr[i] == target:
    # 找到
```

```python
# ❌ 不要自己实现堆
class MyPriorityQueue:
    def __init__(self):
        self.heap = []
    def push(self, x):
        self.heap.append(x)
        self._sift_up(len(self.heap) - 1)
    ...

# ✓ 直接用 heapq
heapq.heappush(heap, x)
heapq.heappop(heap)
```

**尤其不要自己实现的是：**
- 哈希表 (dict/set)
- 动态数组 (list)
- 二叉堆 (heapq)
- 二分查找 (bisect)
- 排序 (sorted / list.sort)
- 记忆化搜索 (lru_cache)

**需要根据情况选择的是：**
- 队列 (deque vs Queue vs list)
- 字典变体 (defaultdict vs Counter vs OrderedDict vs dict)
- 组合/排列生成 (自己实现回溯 vs itertools)

**永远自己实现的是：**
- 二叉树、树的遍历
- 图的 DFS/BFS 具体逻辑
- 动态规划状态转移（但可以用 lru_cache 缓存）
- 链表操作（标准库没有链表）
- 数学算法（gcd, 快速幂等，可用 math 部分代替）

> **最后建议**：刷题前通读一遍 `collections`、`heapq`、`bisect`、`itertools`、`functools` 的官方文档。知道"有这个东西"比"能背出来"更重要 — 你只需要记得存在这个工具，用的时候查文档即可。Python 的标准库是一把组装好的瑞士军刀，它的力量在于**你不需要每次都从打铁开始**。
