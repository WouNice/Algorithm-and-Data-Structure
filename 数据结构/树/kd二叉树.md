# kd 二叉树：从诞生到应用的全景教程

## 一、什么是 kd 树

### 诞生背景

**kd 树**（k-dimensional tree）由 **Jon Louis Bentley** 于 1975 年在论文 *"Multidimensional binary search trees used for associative searching"*（Communications of the ACM）中首次提出。

**催生背景：**

- **1970s 数据库革命：** 关系数据库初兴，传统 B 树 / BST 仅支持一维键值索引，无法高效处理「查询平面上的点」「查找 3D 空间中距离最近的对象」等多维问题。
- **计算几何兴起：** 计算机图形学、地理信息系统（GIS）和 VLSI 布线等领域需要一种能在多维空间中快速定位和搜索的数据结构。
- **Post Office Problem 的驱动：** 著名计算机科学家 **Donald Knuth** 在其《TAOCP》中提出了"邮局问题"——给定一组城市坐标，如何为任意查询点找到最近的城市。这正是最近邻搜索（Nearest Neighbor Search）的原型，kd 树正是为解决此类问题而生的经典结构。

**历史意义：** kd 树是第一个真正意义上将二叉搜索树思想泛化到多维空间的数据结构，奠定了空间索引和计算几何的基础。

### 核心思想

- 每一层选择一个维度作为**分割轴**（Split Axis）
- 在该维度上选取**中位数**（Median）作为节点值
- 小于中位数的点在左子树，大于的在右子树
- 交替使用不同维度进行分割

这种交替分割使得：**空间中某一点附近的区域在树结构中也保持相邻**，这是 kd 树能加速搜索的根本原因。

### 与经典 BST 的对比

| 特性 | 二叉搜索树（BST） | kd 树 |
|------|-------------------|-------|
| 诞生年份 | 1960s（独立提出） | 1975（Bentley） |
| 键值 | 一维标量 | k 维向量 |
| 比较维度 | 始终同一维度 | 逐层交替 |
| 分割方式 | 数值大小划分 | 超平面分割空间 |
| 平衡方式 | 随机插入 / AVL / 红黑 | 中位数选择 |
| 典型查询 | 精确查找、范围查找 | 最近邻搜索、范围搜索 |
| 空间开销 | $O(n)$ | $O(n)$（每个节点存 k 维坐标） |

## 二、核心原理：构建算法

### 诞生背景与核心原理

**解决的问题：** 传统数组/列表存储多维点时，无法利用空间邻近性加速搜索。构建 kd 树的核心目标是以**空间分割**的方式，建立一个可用于高效搜索的"空间索引"。

**核心原理——分治（Divide & Conquer）：**

```
原始空间 ──→ 选定轴分割 ──→ 左右子空间 ──→ 递归分割
```

每个节点对应于一个**超平面**分割，左子树的所有点在该轴上坐标 ≤ 节点该轴坐标，右子树 >。这样，对于任意查询，我们总能迅速判断目标点位于哪个半空间。

### 迭代构建过程

以**二维空间**为例，给定一组点：`(2,3), (5,4), (9,6), (4,7), (8,1), (7,2)`

```
第 1 层（x 轴分割）：
  点集：{(2,3), (5,4), (9,6), (4,7), (8,1), (7,2)}
  按 x 排序：2, 4, 5, 7, 8, 9
  中位数（左开右闭取上中位数）：x=7 → (7,2) 为根
  左子树（x < 7）：{(2,3), (5,4), (4,7)}
  右子树（x ≥ 7）：{(9,6), (8,1)}

第 2 层（y 轴分割）：
  左子树按 y 排序：3, 4, 7 → (5,4) 为左子节点
  右子树按 y 排序：1, 6 → (8,1) 为右子节点

第 3 层（x 轴分割）：
  继续递归...直到所有点插入
```

**树结构：**

```
              (7,2)
             /     \
          (5,4)   (8,1)
         /     \       \
      (2,3)   (4,7)   (9,6)
```

### 空间分割可视化

```
y
↑
7  |   (4,7)
6  |                     (9,6)
5  |
4  |       (5,4)
3  | (2,3)
2  |               (7,2)
1  |                         (8,1)
0  +---|----|----|----|----|----|→ x
   0    2    4    6    8    10

分割线说明：
  根节点 (7,2) → 竖线 x=7 将平面一分为二
  (5,4)        → 横线 y=4 分割左半平面
  (8,1)        → 横线 y=1 分割右半平面
  (2,3) 和 (4,7) → 竖线分割各自子空间
```

### 完整代码实现与关键优化

```python
import math
import statistics


class KDNode:
    """kd 树节点"""
    __slots__ = ('point', 'left', 'right', 'axis', 'id')

    def __init__(self, point, left=None, right=None, node_id=None):
        self.point = point      # k 维坐标点（元组或列表）
        self.left = left        # 左子树
        self.right = right      # 右子树
        self.axis = None        # 分割轴索引
        self.id = node_id       # 可选：点的原始标识


class KDTree:
    """kd 树——空间索引结构"""

    def __init__(self, points, k=2, ids=None):
        """
        构建 kd 树

        参数：
            points: list of tuple/list, 点的坐标集合
            k: 维度
            ids: 点的原始标识（可选）
        """
        self.k = k
        if ids is None:
            data = [(p, None) for p in points]
        else:
            data = list(zip(points, ids))
        self.root = self._build(data, depth=0)

    def _build(self, data, depth):
        """
        递归构建 kd 树

        关键优化点讨论：
        1. 【排序 vs 快速选择】Python sorting O(n log n)，可用 nth_element 降至 O(n)
        2. 【axis 选择策略】Round-Robin 已足够，高方差维度更优但需额外计算
        3. 【中位数选择】左开右闭，左子树 < 轴，右子树 ≥ 轴
        4. 【递归深度】最坏 O(n)（退化），平均 O(log n)
        """
        if not data:
            return None

        # ★ 分割轴选择：按深度轮换（Round-Robin）
        # 变体策略：也可以计算每个维度方差，选方差最大的维度
        # 方差策略优势：使分割后子空间更紧凑，搜索效率更高
        # 代价：需 O(kn) 计算方差，大规模数据时值得
        axis = depth % self.k

        # ★ 按当前轴排序取中位数
        # 优化方向：当 n 很大时，将 sort 替换为 nth_element (quickselect)
        # quickselect 期望 O(n)，最坏 O(n²)，用 median-of-medians 可保证 O(n)
        data.sort(key=lambda item: item[0][axis])
        median = len(data) // 2

        # 构建当前节点
        point, pid = data[median]
        node = KDNode(point, node_id=pid)
        node.axis = axis
        node.left = self._build(data[:median], depth + 1)
        node.right = self._build(data[median + 1:], depth + 1)

        return node
```

### 算法边界与适用场景

| 维度 | 构建策略 | 适用场景 |
|------|---------|---------|
| k=1 | 退化为普通 BST | 一维有序数据（不如红黑树） |
| 2 ≤ k ≤ 10 | kd 树最优 | 地理坐标、图像特征、游戏碰撞 |
| k > 10 | 性能开始退化 | 考虑 Ball Tree 或近似方法 |
| k > 20 | 维度诅咒严重 | 改用 LSH、HNSW 或 ANN |

### 优化构建：快速选择 nth_element

```python
import random

def _quickselect(arr, k, key_func):
    """
    快速选择算法：找到数组中第 k 小的元素
    期望时间复杂度 O(n)，最坏 O(n²)

    典型实现方式：
    1. 选取 pivot（可随机选择避免最坏情况）
    2. 分区（类似快排）
    3. 递归搜索包含第 k 小元素的一侧
    """
    if len(arr) == 1:
        return arr[0]

    # 随机选择 pivot（关键优化：避免有序数组退化为 O(n²)）
    pivot = random.choice(arr)
    pivot_key = key_func(pivot)

    left = [x for x in arr if key_func(x) < pivot_key]
    mid = [x for x in arr if key_func(x) == pivot_key]
    right = [x for x in arr if key_func(x) > pivot_key]

    if k < len(left):
        return _quickselect(left, k, key_func)
    elif k < len(left) + len(mid):
        return mid[0]
    else:
        return _quickselect(right, k - len(left) - len(mid), key_func)


def build_optimized(points, k, depth=0):
    """
    优化构建：quickselect 替代 sort
    构建复杂度从 O(n log n) 降至期望 O(n log n)
    （但 sort 在 Python 中由 C 实现，实际 n 较小时 sort 更快）
    """
    if not points:
        return None

    axis = depth % k
    median_idx = len(points) // 2

    # 使用 quickselect 找到中位数（避免全排序）
    median_point = _quickselect(points, median_idx, key_func=lambda p: p[axis])

    # 分区
    left = [p for p in points if p[axis] < median_point[axis]]
    right = [p for p in points if p[axis] > median_point[axis]]
    mids = [p for p in points if p[axis] == median_point[axis]]

    # 中位数可能有多个相同值，取一个作为当前节点
    node = KDNode(mids[0])
    node.axis = axis
    node.left = build_optimized(left, k, depth + 1)

    # 右侧包含剩下的同轴值
    right = mids[1:] + right
    node.right = build_optimized(right, k, depth + 1)

    return node
```

## 三、核心算法 1：最近邻搜索（Nearest Neighbor Search）

### 诞生背景与核心原理

**历史渊源：** 最近邻搜索问题也称为 **"邮局问题"（Post Office Problem）**，由 Knuth 在《TAOCP》中提出：一个邮局有数千个邮箱，每个邮箱有二维坐标，用户站在某个位置，如何快速找到最近的邮箱？

**在 kd 树上的突破：** Bentley 在 1975 年的论文中展示了 kd 树可以将最近邻搜索从暴力 $O(n)$ 降至平均 $O(\log n)$，这是多维搜索领域的里程碑。

**核心原理——分支限界（Branch and Bound）：**

1. **先搜近处：** 根据目标点与分割超平面的关系，先进入"更有可能"的分支搜索
2. **检查当前：** 访问节点时更新当前最优解
3. **回溯剪枝：** 判断是否需要搜索另一侧——计算目标点到分割超平面的距离，**只有当这个距离小于当前最优距离时**，另一侧才可能有更近的点

**这个"轴距判断"是整个搜索的核心优化——它利用 kd 树的空间分割特性，在保证正确性的前提下**大量裁剪搜索空间。

### 解决的问题与适用边界

**核心问题：** 给定一个点集 $S$，对任意查询点 $q$，找到 $S$ 中距离 $q$ 最近的点。

**适用边界：**

| 条件 | 效果 |
|------|------|
| 低维（k ≤ 10） | 平均 $O(\log n)$，性能极佳 |
| 中等维度（10 < k ≤ 20） | 仍优于暴力，但优势缩小 |
| 高维（k > 20） | 维度诅咒：退化接近 $O(n)$ |
| 点集分布均匀 | 剪枝效果好，搜索更快 |
| 点集分布极端 | 树不平衡，搜索变慢 |

### 完整代码与关键优化点

```python
def euclidean_distance_sq(p1, p2):
    """
    欧几里得距离平方

    核心优化：比较距离时不开方，避免昂贵的 sqrt 运算
    只在最终返回结果时才开方
    """
    return sum((a - b) ** 2 for a, b in zip(p1, p2))


def nearest_neighbor_search(root, target):
    """
    在 kd 树中搜索最近邻

    算法流程（DFS 回溯）：
    1. 从根节点递归下降，每次进入目标所在半空间
    2. 到达叶节点后回溯，沿途检查：
       a) 当前节点是否更优
       b) 是否需要搜索另一侧（轴距剪枝）
    3. 返回全局最优

    时间复杂度：
      - 平均：O(log n)
      - 最坏：O(n)（所有点共线，每次都要搜两侧）
      - 空间复杂度：O(log n)（递归栈）
    """

    def search(node, target, best, depth):
        if node is None:
            return best

        axis = depth % len(target)

        # ★ 步骤 1：决定搜索方向
        # 核心思想：优先进入目标点所在半空间
        # 因为目标点大概率离该侧的点更近
        if target[axis] < node.point[axis]:
            near_branch = node.left
            far_branch = node.right
        else:
            near_branch = node.right
            far_branch = node.left

        # ★ 步骤 2：递归搜索优先分支
        # 一直搜索到叶节点（找到"最可能"的区域）
        best = search(near_branch, target, best, depth + 1)

        # ★ 步骤 3：检查当前节点
        # 回溯时的关键一步：当前节点可能比子树中任何点都近
        current_dist = euclidean_distance_sq(target, node.point)
        best_dist = euclidean_distance_sq(target, best.point)
        if current_dist < best_dist:
            best = node

        # ★ 步骤 4：轴距剪枝（核心优化！）
        # 计算目标点到当前节点分割超平面的距离（平方）
        # 关键洞察：如果这个距离已经 ≥ 当前最优距离，
        # 那么穿越超平面后所有点都不可能比 current_best 更近
        axis_dist = (target[axis] - node.point[axis]) ** 2
        if axis_dist < euclidean_distance_sq(target, best.point):
            # 必须搜索另一侧：因为超平面另一侧存在
            # 半径为 sqrt(best_dist) 的区域内的点
            best = search(far_branch, target, best, depth + 1)

        return best

    best_node = search(root, target, root, 0)
    dist_sq = euclidean_distance_sq(target, best_node.point)
    return best_node.point, math.sqrt(dist_sq)
```

### 剪枝条件推导（数学）

给出剪枝条件的严格数学推导：

```
设当前节点 N 的分割轴为 a，分割值为 v
目标点 T = (t₀, t₁, ..., tₐ, ..., tₖ₋₁)
当前最优距离 = d_best²

条件：T 到超平面 a = v 的距离平方 = (tₐ - v)²

如果 (tₐ - v)² ≥ d_best²：
  对于超平面另一侧的任意点 P：
  ||T - P||² ≥ (tₐ - Pₐ)² ≥ (tₐ - v)² ≥ d_best²
  ∴ P 不可能比当前最优更近 → 不需要搜索另一侧 ✓

如果 (tₐ - v)² < d_best²：
  超平面另一侧可能存在点 P 使得 ||T - P||² < d_best²
  → 必须搜索另一侧
```

**直观理解：** 以目标点 T 为圆心、$\sqrt{d_{best}}$ 为半径的圆（高维：超球体），如果这个圆与当前分割超平面**不相交**，那么整个另一半空间都可以安全跳过。

### 典型题目详解

#### 题目：LeetCode 973 — K Closest Points to Origin

**问题描述：** 给定 `points[i] = [xi, yi]`，返回距离原点 `(0,0)` 最近的 k 个点。

**四种解法对比：**

| 解法 | 时间复杂度 | 空间复杂度 | 适用场景 |
|------|-----------|-----------|---------|
| 全排序 | $O(n\log n)$ | $O(n)$ | 代码最简单 |
| 堆 | $O(n\log k)$ | $O(k)$ | k 较小 |
| 快速选择 | $O(n)$ avg | $O(\log n)$ | 数据量大 |
| **kd 树** | $O(k\log n)$ | $O(n)$ | 多次查询（建树可复用） |

**解题思路推导：**

**思路一：堆（最推荐，面试够用）**
维护大小为 k 的最大堆，遍历每个点计算到原点的距离平方，如果比堆顶小则入堆。最终堆中即为最近的 k 个点。

```python
import heapq
from typing import List

class Solution:
    def kClosest(self, points: List[List[int]], k: int) -> List[List[int]]:
        """
        堆解法：维护大小为 k 的最大堆

        推导过程：
        1. 距离计算：d² = x² + y²（不开方）
        2. 最大堆存 (-d², point)：Python heap 是最小堆
           → 存负值使其行为类似最大堆
        3. 遍历过程中，堆顶总是当前"最大距离"
        4. 遇到更近的点就替换堆顶
        """
        # 技巧：Python heapq 是最小堆，用 (-dist, point) 模拟最大堆
        heap = []
        for x, y in points:
            dist = x * x + y * y
            if len(heap) < k:
                heapq.heappush(heap, (-dist, x, y))
            else:
                # 如果当前点比堆顶（最远的点）更近
                if dist < -heap[0][0]:
                    heapq.heappushpop(heap, (-dist, x, y))

        return [[x, y] for _, x, y in heap]
```

**思路二：快速选择（面试加分）**
利用快排分区思想，找第 k 小的距离，同时将前 k 个点划分到数组左侧。

```python
import random
from typing import List

class Solution:
    def kClosest(self, points: List[List[int]], k: int) -> List[List[int]]:
        """
        快速选择解法：期望 O(n)

        推导过程：
        1. 选 pivot 按距离分区
        2. left: 距离 < pivot_dist
        3. mid: 距离 == pivot_dist
        4. right: 距离 > pivot_dist
        5. 根据 k 在 left/mid/right 中的位置决定递归方向
        """
        dists = [(x * x + y * y, x, y) for x, y in points]

        def quick_select(arr, k):
            if len(arr) <= 1:
                return arr

            pivot = random.choice(arr)
            pivot_dist = pivot[0]

            left = [p for p in arr if p[0] < pivot_dist]
            mid = [p for p in arr if p[0] == pivot_dist]
            right = [p for p in arr if p[0] > pivot_dist]

            if k < len(left):
                return quick_select(left, k)
            elif k < len(left) + len(mid):
                return left + mid[:k - len(left)]
            else:
                return left + mid + quick_select(right, k - len(left) - len(mid))

        result = quick_select(dists, k)
        return [[x, y] for _, x, y in result]
```

**思路三：kd 树（适用于多次查询）**

当需要**多次**查询不同目标点（如每隔几秒查询当前位置最近的餐馆），建树一次后每次查询只需 $O(\log n)$，远快于每次重新计算。

```python
from typing import List

class Solution:
    def kClosest(self, points: List[List[int]], k: int) -> List[List[int]]:
        """kd 树解法（单次查询优势不大，多次查询优势明显）"""
        if k == len(points):
            return points

        # 建树 O(n log n)
        kdtree = KDTree(points)

        # 查询 O(k log n)
        nearest = k_nearest_neighbors(kdtree.root, (0, 0), k)
        return [list(p) for p in nearest]
```

#### 题目：LeetCode 658 — Find K Closest Elements

**问题描述：** 给定排序数组 `arr`，找到最靠近 `x` 的 k 个数，结果升序排列。

**理解与 kd 树的关联：** 这是**一维空间**的最近邻搜索问题。一维 kd 树 = 普通 BST，这里用更简洁的双指针或二分+双指针求解。

**解题思路推导：**

```python
from typing import List
import bisect

class Solution:
    def findClosestElements(self, arr: List[int], k: int, x: int) -> List[int]:
        """
        方法一：二分 + 双指针（最优）

        推导过程：
        1. 二分查找 x 在 arr 中的插入位置 idx
        2. 双指针 left = idx - 1, right = idx 向两侧扩展
        3. 比较 arr[right] - x 和 x - arr[left]，选更小的
        4. 直到找到 k 个元素

        时间复杂度 O(log n + k)
        """
        idx = bisect.bisect_left(arr, x)
        left, right = idx - 1, idx

        result = []
        while len(result) < k:
            if left < 0:
                result.append(arr[right])
                right += 1
            elif right >= len(arr):
                result.append(arr[left])
                left -= 1
            elif x - arr[left] <= arr[right] - x:
                result.append(arr[left])
                left -= 1
            else:
                result.append(arr[right])
                right += 1

        return sorted(result)
```

## 四、核心算法 2：K 近邻搜索（KNN Search）

### 诞生背景与核心原理

**历史渊源：** K 近邻算法是**模式识别**和**机器学习**中最基础的非参数方法之一，由 Fix 和 Hodges 于 1951 年提出。当需要寻找 k 个最近邻而不是 1 个时，搜索更加复杂。

**从 1-NN 到 K-NN 的挑战：** 最近邻搜索维护的只是一个"全局最优"，而 KNN 需要维护一个**长度为 k 的有序列表**——这不仅增加了状态管理的复杂度，也使剪枝条件的判定变得更精细：需要检查的是最新"第 k 近"的距离，而不仅仅是最优。

**核心原理：** KNN 在 kd 树上的实现基于**最大堆**（Max-Heap）来维护当前找到的 k 个候选点。

- 堆的大小恒为 k
- 堆顶是第 k 近的候选项（即 k 个候选中的"最远距离"）
- 剪枝条件变为：如果轴距离 < **堆顶距离**（即当前第 k 近的距离），说明远侧可能存在应被纳入候选的点

### 解决的问题与适用边界

**核心问题：** 给定点集 $S$，对查询点 $q$，找到 $S$ 中距离 $q$ 最近的 $k$ 个点（按距离升序）。

**适用边界：**

| 条件 | 效果 |
|------|------|
| $k \ll n$ | 堆维护成本低，接近 1-NN 性能 |
| $k$ 较大（如 $k \approx n/2$） | 堆操作 $O(\log k)$ 变大，剪枝效果变差 |
| 数据维度低 | 剪枝高效，接近 $O(\log n)$ |
| 需要**全序结果** | 最终需对堆排序（或弹出排序） |

### 完整代码与关键优化

```python
import heapq

def k_nearest_neighbors(root, target, k):
    """
    K 近邻搜索

    算法流程：
    1. 使用最大堆（Python 存负值实现）维护 k 个候选点
    2. 与 1-NN 类似的回溯搜索，但有两个关键变化：
       a) 堆顶距离替代单个最优距离
       b) k 个点未满时不剪枝
    3. 搜索完成后，堆转为有序列表返回

    核心优化：
    - 堆的 push/pop 操作 O(log k)，k 越大开销越大
    - 剪枝条件用堆顶距离（第 k 近的距离）
    - 如果命中点数量尚不足 k，则必须搜远侧

    时间复杂度：$O(k \log n \log k)$
      - 搜索路径 $O(\log n)$
      - 每次堆操作 $O(\log k)$
    最坏情况：$O(k n \log k$（全遍历）
    """
    heap = []

    def search(node, depth):
        if node is None:
            return

        axis = depth % len(target)
        point = node.point
        dist = euclidean_distance_sq(target, point)

        # ★ 步骤 1：决定搜索顺序（类似 1-NN）
        if target[axis] < point[axis]:
            near, far = node.left, node.right
        else:
            near, far = node.right, node.left

        # ★ 步骤 2：先搜近侧
        search(near, depth + 1)

        # ★ 步骤 3：处理当前节点
        # 堆未满 → 直接加入
        # 堆已满但当前点更近 → 替换堆顶
        if len(heap) < k:
            heapq.heappush(heap, (-dist, point))
        elif dist < -heap[0][0]:
            heapq.heappushpop(heap, (-dist, point))

        # ★ 步骤 4：轴距剪枝（使用堆顶距离）
        # 如果堆未满，必须搜远侧（因为没有候选可剪枝）
        # 如果堆已满，只有轴距 < 堆顶距离时可能找到更好的
        if len(heap) < k or (target[axis] - point[axis]) ** 2 < -heap[0][0]:
            search(far, depth + 1)

    search(root, 0)

    # 整理输出：最大堆转为升序列表
    # 过程：逐个 pop（从大到小），再反转
    result = []
    while heap:
        _, point = heapq.heappop(heap)
        result.append(point)
    result.reverse()
    return result
```

### 常见误区与调试

```python
# 误区 1：堆距离比较用了 sqrt
# ❌ 错误做法
dist = math.sqrt(squared_dist)  # 不必要的开销
heapq.heappush(heap, (-dist, point))

# ✅ 正确做法：全程使用平方距离，只在最终输出时开方
dist = squared_dist
heapq.heappush(heap, (-dist, point))

# 误区 2：推入堆但未验证是否已有更近点
# ✅ 正确做法：每次推入前先检查堆容量和堆顶距离
```

## 五、核心算法 3：范围搜索（Range Search）

### 诞生背景与核心原理

**历史渊源：** 范围搜索（Range Search / Window Query）是空间数据库的基础操作，由 Bentley 和 Friedman 于 1979 年系统研究。"给定一个矩形区域，找出其中所有点"——这在地理信息系统（GIS）、碰撞检测、数据库范围查询中无处不在。

**核心原理：** kd 树的空间分割特性天然适合范围搜索：

1. **自上而下剪枝：** 从根开始，如果当前节点的分割轴与查询区域不相交，则可以完全跳过整个子树
2. **递归包含性：** 当某个节点的整个子树空间完全被查询区域覆盖时，整个子树都可返回（本实现为了简化未做此优化）
3. **分治合并：** 左右子树的结果最终合并

### 解决的问题与适用边界

**核心问题：** 给定一个 $k$ 维轴对齐超矩形（Axis-Aligned Hyperrectangle），找出其中所有的点。

**适用边界：**

| 条件 | 效果 |
|------|------|
| 查询区域小 | 剪枝高效，快速跳过大量区域 |
| 查询区域大（接近全空间） | 接近全遍历 $O(n)$ |
| 数据分布均匀 | 每层都能有效剪枝 |
| 数据呈条带状 | 剪枝效果退化 |

### 完整代码与关键优化

```python
def range_search(root, query_region, depth=0):
    """
    范围搜索：找出 kd 树中落在指定区域内的所有点

    参数：
        root: KDNode, kd 树根节点
        query_region: list of (min, max), 每个维度的范围
        depth: 当前深度

    返回：
        list of points, 区域内所有点

    算法流程：
    1. 检查当前点是否在区域内
    2. 判断左子树空间是否与区域相交：如果区域最小值 ≤ 当前节点分割值
    3. 判断右子树空间是否与区域相交：如果区域最大值 ≥ 当前节点分割值
    4. 递归进行

    时间复杂度：$O(n^{1-1/k} + m)$
      其中 m 为结果数量，k 为维度

    优化技巧：
    - 如果当前子树完全被区域包含，整棵子树可直接返回
    - 但判断"完全包含"需要跟踪当前子树的空间范围
    """

    def search(node, depth):
        if node is None:
            return

        k = len(node.point)
        axis = depth % k
        point = node.point

        # ★ 检查当前点是否在区域内
        in_region = all(
            lo <= point[i] <= hi
            for i, (lo, hi) in enumerate(query_region)
        )
        if in_region:
            result.append(point)

        # ★ 左子树优化剪枝
        # 如果区域在当前轴上的最小值 ≤ 当前节点的分割值
        # 则左子树（坐标 ≤ 分割值）可能包含区域内的点
        if query_region[axis][0] <= point[axis]:
            search(node.left, depth + 1)

        # ★ 右子树优化剪枝
        if query_region[axis][1] >= point[axis]:
            search(node.right, depth + 1)

    result = []
    search(root, depth)
    return result
```

### 进阶优化：完全包含判断

```python
def range_search_optimized(root, query_region):
    """
    带完全包含优化的范围搜索

    核心优化：跟踪当前子树的空间边界，
    如果子树整个被查询区域包含，则直接返回整棵子树的所有点（无剪枝遍历）。
    这在大区域查询时能大幅减少递归次数。
    """
    result = []

    def subtree_points(node):
        """收集子树所有点（完全包含时使用）"""
        if node is None:
            return
        result.append(node.point)
        subtree_points(node.left)
        subtree_points(node.right)

    def search(node, region_bounds, depth):
        """
        region_bounds: list of (min, max), 当前子树在整个空间中的边界
        例如根节点的 region_bounds 是整个坐标空间
        """
        if node is None:
            return

        k = len(node.point)
        axis = depth % k
        point = node.point

        # ★ 优化：检查子树是否完全被查询区域包含
        subtree_contained = all(
            query_lo <= bound_lo and bound_hi <= query_hi
            for (query_lo, query_hi), (bound_lo, bound_hi) in zip(query_region, region_bounds)
        )
        if subtree_contained:
            subtree_points(node)
            return

        # 检查当前点
        if all(
            lo <= point[i] <= hi
            for i, (lo, hi) in enumerate(query_region)
        ):
            result.append(point)

        # 递归子空间
        lo, hi = region_bounds[axis]
        if query_region[axis][0] <= point[axis]:
            new_bounds = region_bounds.copy()
            new_bounds[axis] = (lo, point[axis])  # 左子空间上界为当前分割值
            search(node.left, new_bounds, depth + 1)
        if query_region[axis][1] >= point[axis]:
            new_bounds = region_bounds.copy()
            new_bounds[axis] = (point[axis], hi)  # 右子空间下界为当前分割值
            search(node.right, new_bounds, depth + 1)

    # 初始空间边界：设得足够宽（实际应用应传入真实边界）
    initial_bounds = [(float('-inf'), float('inf')) for _ in range(len(query_region))]
    search(root, initial_bounds, 0)
    return result
```

### 典型题目详解

#### 题目：统计矩形区域内的点数

**问题描述：** 二维平面上有 n 个点，多次给定矩形 `(x1, y1, x2, y2)`，查询矩形内的点数。

**解题思路推导：**

```python
def count_in_rect(root, x1, y1, x2, y2):
    """
    统计矩形 [x1, x2] × [y1, y2] 内的点数

    与 kd 树范围搜索结合：预处理建树，每次查询 O(√n + m)

    典型应用场景：
    - 地图上查找某区域内的 POI 数量
    - 图像处理中统计某个特征区间的像素
    """
    region = [(x1, x2), (y1, y2)]
    points = range_search(root, region)
    return len(points)
```

## 六、核心算法 4：范围计数（Range Count）

### 核心原理

**与范围搜索的区别：** 范围计数只需要**数量**不需要具体点，因此可以进一步剪枝。

**核心原理：** 在递归过程中，如果当前节点的整个子树空间**全部在查询区域内**，可以直接返回该子树的节点总数（预计算），无需逐点检查。

### 完整代码

```python
def range_count(root, query_region, depth=0):
    """
    范围计数：统计区域内点的个数

    优化：比 range_search 更快，因为不需要收集具体点，
    遇到完全包含的子树可以直接返回大小而无需遍历。
    """
    if root is None:
        return 0

    k = len(root.point)
    axis = depth % k
    point = root.point

    count = 1 if all(
        lo <= point[i] <= hi
        for i, (lo, hi) in enumerate(query_region)
    ) else 0

    # 左子树：如果区域的最小值 ≤ 分割轴值
    if query_region[axis][0] <= point[axis]:
        count += range_count(root.left, query_region, depth + 1)
    # 右子树：如果区域的最大值 ≥ 分割轴值
    if query_region[axis][1] >= point[axis]:
        count += range_count(root.right, query_region, depth + 1)

    return count
```

## 七、核心算法 5：插入与删除（动态维护）

### 诞生背景与核心原理

**为什么需要动态操作？** 现实场景中，点集通常是**动态变化**的：
- 实时导航：新的 POI 不断加入，旧 POI 可能被移除
- 传感器网络：设备上线/下线
- 游戏场景：对象创建/销毁

**插入原理：** 类似 BST 插入，根据分割轴比较决定左/右子树。但 kd 树的插入**不一定保持平衡**——按顺序插入会导致退化。

**删除原理：** 删除比 BST 复杂得多。BST 删除用前驱或后继替换，但 kd 树的后继取决于**分割轴**，不能用简单的"右子树最左节点"替代。

**核心算法——FindMin（关键）**：为了找到正确的替换节点，需要：

```
def findMin(node, target_axis):
    if node 的分割轴 == target_axis:
        → 最小值一定在左子树（或自己）
    else:
        → 两侧都可能，递归找最小值
```

### 插入代码与边界分析

```python
def insert(root, point, depth=0):
    """
    向 kd 树中插入新点

    注意事项：
    1. kd 树不是自平衡的——顺序插入会导致线性退化！
    2. 大量插入后应考虑重建
    3. 与 BST 不同，当 point[axis] == node.point[axis] 时，
       统一归入右子树（或左子树），保持一致性即可

    退化示例：
    按 (1,1), (2,2), (3,3), (4,4)... 顺序插入
    → 每层都在同一侧，退化为链表 O(n)
    """
    if root is None:
        node = KDNode(point)
        node.axis = depth % len(point)
        return node

    axis = depth % len(point)
    if point[axis] < root.point[axis]:
        root.left = insert(root.left, point, depth + 1)
    else:
        root.right = insert(root.right, point, depth + 1)

    return root
```

### 删除原理深度解析

```python
def find_min(node, target_axis, depth=0):
    """
    在 kd 树中找指定轴 target_axis 上的"最小值节点"

    为什么要这样做？

    场景：删除节点 N，其分割轴为 a
    需要在 N 的子树中找一个节点 M 来替代 N
    M 必须满足：对轴 a 而言，是子树中最小值

    算法逻辑：
    - 如果当前节点分割轴 == target_axis：
      → 最小值一定在左子树（因为左子树所有点在该轴上更小）
        如果左子树为空，当前节点就是最小值
    - 如果当前节点分割轴 != target_axis：
      → 无法判断最小值在左还是右
      → 必须同时搜索两侧和当前节点
      → 取三者中最小值
    """
    if node is None:
        return None

    current_axis = depth % len(node.point)

    if current_axis == target_axis:
        # 目标轴方向：最小值只能在左子树或当前节点
        if node.left is None:
            return node
        return find_min(node.left, target_axis, depth + 1)

    # 非目标轴：两侧都需要搜索
    left_min = find_min(node.left, target_axis, depth + 1)
    right_min = find_min(node.right, target_axis, depth + 1)

    candidates = [n for n in [left_min, right_min] if n is not None]
    candidates.append(node)

    # 在 target_axis 上取最小值
    return min(candidates, key=lambda n: n.point[target_axis])


def delete(root, point, depth=0):
    """
    从 kd 树中删除指定点

    删除策略（三种情况）：

    情况 1：叶节点 → 直接删除

    情况 2：右子树非空
      → 从右子树中找 target_axis 上的最小值节点 M
      → 用 M 替换当前节点
      → 递归删除 M（原位置）

    情况 3：右子树为空，左子树非空
      → 从左子树中找 target_axis 上的最小值节点 M
      → 用 M 替换当前节点
      → 左子树替换右子树位置
      → 递归删除 M

    为什么情况 3 需要"右子树 = 左子树；左子树 = None"？
    因为 kd 树的分割规则：左子树 ≤ 节点，右子树 > 节点。
    替换后 M 的值（最小值）一定 ≤ 右子树的所有值，
    所以把原左子树作为新右子树。
    """
    if root is None:
        return None

    axis = depth % len(point)

    # 找到要删除的节点
    if root.point == point:
        if root.right is not None:
            # 找右子树中当前轴上的最小值
            min_node = find_min(root.right, axis, depth + 1)
            root.point = min_node.point
            root.right = delete(root.right, min_node.point, depth + 1)
        elif root.left is not None:
            min_node = find_min(root.left, axis, depth + 1)
            root.point = min_node.point
            # 关键：将左子树移到右侧
            root.right = delete(root.left, min_node.point, depth + 1)
            root.left = None
        else:
            return None  # 叶节点，直接删除
    elif point[axis] < root.point[axis]:
        root.left = delete(root.left, point, depth + 1)
    else:
        root.right = delete(root.right, point, depth + 1)

    return root
```

### 动态操作的典型问题

**问题：** 分析以下操作序列对 kd 树的影响：

```
插入: (3,3), (1,5), (4,2), (0,6), (5,1)
删除: (4,2)
插入: (2,4)
```

**分析退化情况：**
```
按 x 升序插入：(1,1), (2,2), (3,3), (4,4), (5,5)

第 1 层（x 轴）：(1,1) 为根 → (2,2) 入右子树
第 2 层（y 轴）：(2,2) 与 (1,1) 比较 y → 入右子树
第 3 层（x 轴）：(3,3) 入右子树...
结果：所有节点都在右子树，退化为链表！
```

**解决方案：**
- 批量重建：当插入量达到阈值时，用平衡方式重建整棵树
- Bkd 树：维护多棵 kd 树副本，定期合并
- 懒惰删除：标记删除而非真正删除，定期重建

## 八、综合应用算法

### 带标签的最近邻搜索

**问题场景：** 在点集中找到离目标最近且标签匹配的点。例如：在地图上找最近的"咖啡店"而不是"加油站"。

**核心思路：** 在 kd 树节点中存储标签信息，搜索过程中只考虑符合标签条件的点。

```python
def nearest_with_label(root, target, target_label):
    """
    带标签过滤的最近邻搜索

    算法变化：
    1. 每个节点存储 label
    2. 检查当前点时，只考虑 label 匹配
    3. 使用全局变量跟踪最优解（因需同时记录距离和标签）
    """
    best = [None, float('inf')]

    def search(node, depth):
        if node is None:
            return

        axis = depth % len(target)
        if node.label == target_label:
            dist = euclidean_distance_sq(target, node.point)
            if dist < best[1]:
                best[0] = node.point
                best[1] = dist

        # 决定搜索方向
        if target[axis] < node.point[axis]:
            near, far = node.left, node.right
        else:
            near, far = node.right, node.left

        search(near, depth + 1)

        # 轴距剪枝
        axis_dist = (target[axis] - node.point[axis]) ** 2
        if axis_dist < best[1]:
            search(far, depth + 1)

    search(root, 0)
    return best[0], math.sqrt(best[1])
```

### 图像特征检索

**问题场景：** 基于图像特征向量（如 SIFT、CNN 特征）的最相似图像检索。

```python
def image_similarity_search(feature_db, query_feature, top_k=10):
    """
    基于 kd 树的图像相似性检索

    流程：
    1. 离线：用所有图像的特征向量构建 kd 树
    2. 在线：对查询图像提取特征向量
    3. 搜索 kd 树得到最相似的 top_k 图像

    维度注意事项：
    - SIFT 特征通常是 128 维 → kd 树不再适用！
    - 对于高维特征，应使用：
      * 降维后再用 kd 树（PCA）
      * 或用近似算法（HNSW、FAISS）

    适用场景（低维特征）：
    - 颜色直方图（3D: RGB）
    - 纹理特征（6-8D）
    - 位置特征（2D: 经纬度）
    """
    points = [feat for feat, _ in feature_db]
    kdtree = KDTree(points)

    nearest = k_nearest_neighbors(kdtree.root, query_feature, top_k)

    # 映射回图像 ID
    feat_to_id = {tuple(f): img_id for f, img_id in feature_db}
    return [feat_to_id[tuple(p)] for p in nearest]
```

## 九、算法复杂度与对比分析

### 完整复杂度矩阵

| 操作 | 平均时间复杂度 | 最坏时间复杂度 | 空间复杂度 | 备注 |
|------|:------------:|:------------:|:---------:|------|
| **构建** | $O(n\log n)$ | $O(n\log n)$ | $O(n)$ | 每层排序 |
| **构建（优化）** | $O(n\log n)$ | $O(n^2)$ | $O(n)$ | quickselect |
| **最近邻搜索** | $O(\log n)$ | $O(n)$ | $O(\log n)$ | 栈深度 |
| **KNN 搜索** | $O(k\log n\log k)$ | $O(kn\log k)$ | $O(k+\log n)$ | 堆+栈 |
| **范围搜索（2D）** | $O(\sqrt{n}+m)$ | $O(n)$ | $O(\log n)$ | m=结果数 |
| **范围计数** | $O(n^{1-1/k})$ | $O(n)$ | $O(\log n)$ | k=维度 |
| **插入** | $O(\log n)$ | $O(n)$ | $O(1)$ | 不平衡！ |
| **删除** | $O(\log n)$ | $O(n)$ | $O(\log n)$ | 含 find_min |

### kd 树 vs 其他空间数据结构

| 结构 | 构建 | 最近邻 | 范围搜索 | 动态性 | 维度适应性 |
|------|:----:|:------:|:--------:|:-----:|:---------:|
| **kd 树** | $O(n\log n)$ | $O(\log n)$ avg | $O(n^{1-1/k}+m)$ | 差 | 低维（≤10） |
| **Ball Tree** | $O(n\log n)$ | $O(\log n)$ avg | $O(\log n+m)$ | 中 | 中等（≤20） |
| **R 树** | $O(n\log n)$ | $O(\log n)$ avg | $O(\log n+m)$ | 好 | 低维（≤10） |
| **VP Tree** | $O(n\log n)$ | $O(\log n)$ avg | 好 | 中 | 任意度量空间 |
| **Octree** | $O(n)$ | $O(\log n)$ avg | $O(n)$ | 好 | 三维离散 |
| **LSH** | $O(n)$ | 近似 | — | 好 | 高维（优秀） |
| **HNSW** | $O(n\log n)$ | $O(\log n)$ | — | 好 | 高维（优秀） |

### 维度诅咒的数学解释

假设 k 维空间中，点均匀分布在 $[0,1]^k$ 超立方体内。

1. **最近邻距离期望** $E[d_{nn}(k)] \approx \left(\frac{1}{n}\right)^{1/k}$
   - 当 $k=20$，$n=10^6$ 时：$E[d_{nn}] \approx (10^{-6})^{1/20} \approx 10^{-0.3} \approx 0.5$
   - 最近邻距离几乎等于空间半宽！

2. **剪枝失效原因**：
   - 目标到超平面距离的期望值随 k 增大而增大
   - 当最近邻距离 ≈ 空间半宽时，几乎所有轴距都 < 最近距离
   - 必须搜索几乎每一棵子树 → 退化为 $O(n)$

3. **经验法则**：
   - $n$ 固定时，kd 树的搜索复杂度随 $k$ 指数增长
   - 需要 $n \gg 2^k$ 才能保证 kd 树的有效性

## 十、常见算法题目详解

### 题目 1：二维平面最近点对（经典分治问题）

> **问题描述：** 给定 $n$ 个二维坐标点，找到欧几里得距离最近的一对点。

**解法一：kd 树***

```python
import math

def closest_pair(points):
    """
    使用 kd 树求解最近点对

    思路：
    1. 构建 kd 树
    2. 对每个点，找它的最近邻（排除自身）
    3. 取所有点中最近邻距离最小的对

    时间复杂度：O(n log n) 建树 + O(n log n) 查询 = O(n log n)
    空间复杂度：O(n)

    注意：如果有很多点距离为 0（重合），需要使用点 id 来排除自身
    """
    if len(points) < 2:
        return None, float('inf')

    # 给点加上 id
    points_with_id = [(p, i) for i, p in enumerate(points)]

    # 重写 KDTree 以支持 id
    class KDTreeWithId(KDTree):
        def _build(self, data, depth):
            if not data:
                return None
            axis = depth % self.k
            data.sort(key=lambda item: item[0][axis])
            median = len(data) // 2
            point, pid = data[median]
            node = KDNode(point, node_id=pid)
            node.axis = axis
            node.left = self._build(data[:median], depth + 1)
            node.right = self._build(data[median + 1:], depth + 1)
            return node

    # 建树
    tree = KDTreeWithId(points, k=2)

    min_dist = float('inf')
    best_pair = None

    # 遍历每个点，搜索最近邻（排除自身）
    for point, pid in points_with_id:
        # 特殊修改：排除 id 相同的点
        def nearest_except_self():
            best_node = tree.root
            best_dist = float('inf')

            def search(node, depth):
                nonlocal best_node, best_dist
                if node is None:
                    return

                axis = depth % 2
                # 决定搜索方向
                if point[axis] < node.point[axis]:
                    near, far = node.left, node.right
                else:
                    near, far = node.right, node.left

                search(near, depth + 1)

                # 排除自身
                if node.id != pid:
                    dist = euclidean_distance_sq(point, node.point)
                    if dist < best_dist:
                        best_dist = dist
                        best_node = node

                # 轴距剪枝
                axis_dist = (point[axis] - node.point[axis]) ** 2
                if axis_dist < best_dist:
                    search(far, depth + 1)

            search(tree.root, 0)
            return best_node.point, math.sqrt(best_dist)

        neighbor, dist = nearest_except_self()
        if dist < min_dist and dist > 0:
            min_dist = dist
            best_pair = (point, neighbor)

    return best_pair, min_dist
```

**解法二：分治法（最经典，无需 kd 树）**

```python
def closest_pair_divide_conquer(points):
    """
    分治法求最近点对

    算法流程：
    1. 按 x 坐标排序
    2. 递归分治：左右两边各求内部最近点对
    3. 合并：检查跨左右两边、距离 ≤ min(d_left, d_right) 的点对

    合并步骤详解：
    a) 取中间线 mid_x，筛选出 x ∈ [mid_x-d, mid_x+d] 的点
    b) 按 y 排序这些点
    c) 对每个点，检查其后最多 6 个点（几何证明保证 ≤ 6）

    时间复杂度：O(n log n)
    空间复杂度：O(n)
    """
    points = sorted(points, key=lambda p: p[0])

    def distance(p1, p2):
        return math.sqrt((p1[0] - p2[0]) ** 2 + (p1[1] - p2[1]) ** 2)

    def solve(l, r):
        """返回 [l, r) 区间内的最近点对"""
        if r - l <= 1:
            return float('inf'), None, None
        if r - l == 2:
            d = distance(points[l], points[l + 1])
            return d, points[l], points[l + 1]

        mid = (l + r) // 2
        mid_x = points[mid][0]

        d_left, p1_left, p2_left = solve(l, mid)
        d_right, p1_right, p2_right = solve(mid, r)
        d = min(d_left, d_right)
        best_pair = (p1_left, p2_left) if d_left < d_right else (p1_right, p2_right)

        # ★ 合并：筛选条形区域内的点
        strip = [p for p in points[l:r] if abs(p[0] - mid_x) < d]
        strip.sort(key=lambda p: p[1])

        # ★ 对每个点检查后续最多 6 个点
        for i in range(len(strip)):
            for j in range(i + 1, len(strip)):
                if strip[j][1] - strip[i][1] >= d:
                    break
                dist = distance(strip[i], strip[j])
                if dist < d:
                    d = dist
                    best_pair = (strip[i], strip[j])

        return d, best_pair[0], best_pair[1]

    _, p1, p2 = solve(0, len(points))
    return (p1, p2), distance(p1, p2)
```

### 题目 2：LeetCode 973 — K Closest Points to Origin

（见第三章 3.5 节，已有完整推导，此处补充面试追问）

**面试追问与回答：**

> **Q：** 如果 points 有 10⁷ 个，k = 10，哪种方法最优？
> **A：** **堆解法 $O(n\log k) = O(10⁷\log 10)$**，约 3000 万次操作。快速选择 $O(n)$ 理论上更快，但 quickselect 需要全量分区，内存访问模式不如 heap 友好。实践中 heap 因为常数小通常更快。

> **Q：** 如果要求"在线算法"——流式输入 points，每次只看到 1 个点？
> **A：** 只能用**堆解法**。kd 树和快速选择都要求全量数据。

> **Q：** 能否用 kd 树优化到 $O(\log n)$？
> **A：** 单次查询可以，但建树需 $O(n\log n)$。当：① 有多次查询 ② 点集相对稳定 时，建树成本摊薄后 kd 树更有优势。

### 题目 3：LeetCode 447 — Number of Boomerangs

> **问题描述：** 给定 $n$ 个点，求所有满足 $dist(p_i, p_j) = dist(p_i, p_k)$ 的三元组 $(i, j, k)$ 的数量（$i,j,k$ 互不相同，$n \leq 500$）。

**解题思路推导：**

```
核心洞察：
对于每个点 i，距离相等的 j 和 k 形成回旋镖。
如果有 m 个点到 i 距离相同，这些点可以构成 P(m, 2) = m × (m-1) 个回旋镖。

例如：点 i 有 3 个距离为 d 的邻居
→ 从 3 个点中选 2 个有序排列 → 3 × 2 = 6 个三元组
→ 如果还有 2 个距离为 e 的邻居 → 2 × 1 = 2 个三元组
→ 总计 8 个以 i 为中心的回旋镖
```

```python
from collections import defaultdict
from typing import List

class Solution:
    def numberOfBoomerangs(self, points: List[List[int]]) -> int:
        """
        时间复杂度：O(n²) — n ≤ 500，完全可接受

        逐点建距离映射：
        对每个点 i，
        1. 建立所有其他点到 i 的距离 → 计数
        2. 对每个距离，ans += cnt * (cnt - 1)
        """
        ans = 0
        for x1, y1 in points:
            freq = defaultdict(int)
            for x2, y2 in points:
                dx, dy = x1 - x2, y1 - y2
                freq[dx * dx + dy * dy] += 1
            for cnt in freq.values():
                ans += cnt * (cnt - 1)  # P(cnt, 2) = cnt!/(cnt-2)!
        return ans
```

**进阶思考（kd 树如何加速？）**
当 n 很大（如 $10^5$）时，$O(n^2)$ 不可行。此时可以：
1. 对每个点，用 kd 树做 KNN 搜索，减少候选集大小
2. 但距离哈希仍然需要精确相等的点 → 近似方法（距离容忍度）

### 题目 4：LeetCode 149 — Max Points on a Line

> **问题描述：** 给定 $n$ 个二维点，求最多有多少个点共线。

**解题思路推导：**

```
核心洞察：
从每个点 i 出发，计算到其他点的斜率。
斜率相同 → 共线（与 i 在同一条直线上）。

斜率表示的关键问题：
1. 垂直线的斜率无穷大 → 特殊处理 dx = 0
2. 浮点数精度 → 用简化分数 (dx/g, dy/g) 而不是浮点数
3. 重合点 → 需要单独统计
```

```python
from collections import defaultdict
from math import gcd

class Solution:
    def maxPoints(self, points: List[List[int]]) -> int:
        """
        逐点统计斜率频率

        对每个点 i：
        1. 统计与 i 重合的点数（same）
        2. 对其他点，计简化斜率 (dx//g, dy//g)
        3. 取该点上的最大共线数 = same + max(各斜率计数)
        4. 全局最大值

        时间复杂度：O(n²)
        """
        n = len(points)
        if n < 3:
            return n

        max_count = 0
        for i in range(n):
            xi, yi = points[i]
            slopes = defaultdict(int)
            same = 1

            for j in range(i + 1, n):
                xj, yj = points[j]
                dx = xj - xi
                dy = yj - yi

                if dx == 0 and dy == 0:
                    same += 1
                    continue

                # ★ 简化分数，避免浮点数
                g = gcd(dx, dy)
                dx //= g
                dy //= g

                # ★ 统一方向：确保 (dx, dy) 的表示唯一
                # 比如 (-1, 2) 和 (1, -2) 是同一斜率
                if dx < 0 or (dx == 0 and dy < 0):
                    dx, dy = -dx, -dy

                slopes[(dx, dy)] += 1

            # 该点的最大共线数
            local_max = same
            for cnt in slopes.values():
                local_max = max(local_max, cnt + same)
            max_count = max(max_count, local_max)

        return max_count
```

### 题目 5：LeetCode 1030 — Matrix Cells in Distance Order

> **问题描述：** 给定 $R \times C$ 矩阵和起点 $(r0, c0)$，返回所有单元格按曼哈顿距离排序。

**解题思路推导：**

```
核心洞察：
曼哈顿距离 d = |r - r0| + |c - c0|
最大距离 = max(离起点的最远行 + 最远列)

最优解是桶排序（BFS 亦可）：
因为距离值域是整数且范围确定（0 到 R+C-2），
桶排序 O(R×C) 是最优的。
```

```python
from typing import List

class Solution:
    def allCellsDistOrder(self, R: int, C: int,
                          r0: int, c0: int) -> List[List[int]]:
        """
        桶排序解法

        推理过程：
        1. 曼哈顿距离范围：[0, R+C-2]
        2. 创建 (R+C-1) 个桶
        3. 遍历每个格，计算距离，放入对应桶
        4. 按距离顺序拼接桶

        时间复杂度：O(R×C) — 必须访问所有格
        空间复杂度：O(R×C) — 存储结果
        """
        max_dist = (R - 1 - r0) + (C - 1 - c0)
        buckets = [[] for _ in range(max_dist + 1)]

        for r in range(R):
            for c in range(C):
                dist = abs(r - r0) + abs(c - c0)
                buckets[dist].append([r, c])

        return [cell for bucket in buckets for cell in bucket]
```

### 题目 6：LeetCode 1760 — Minimum Limit of Balls in a Bag（二分+分割思想）

> **问题描述：** 给定数组 `nums`，每次操作将任意一个袋子拆成两袋，使两袋球数之和等于原袋。最多操作 `maxOperations` 次，求最小化袋中最大球数。

**与 kd 树的联系：** 此题体现了**空间分割的二分思想**——正如 kd 树递归分割空间来平衡点的分布，此题通过递归分割袋子来平衡球数。

**解题思路推导：**

```
二分答案的思考过程：

1. 问题具有单调性：
   如果允许袋中最多有 X 个球，合法 → 允许 X+1 个球也合法
   如果允许 X 个球不合法 → X-1 个球也不合法

2. 判断函数 can(X)：
   对袋子中球数 > X 的袋子，需要拆分
   一个装有 n 个球的袋子 → 拆成球数 ≤ X 需要 (n-1)//X 次操作
   总操作数 ≤ maxOperations → 可行

3. 二分范围：
   下界：1（不能为空）
   上界：max(nums)（不拆分时的最大值）
```

```python
from typing import List

class Solution:
    def minimumSize(self, nums: List[int], maxOperations: int) -> int:
        """
        二分答案法

        闭区间二分：
        left = 1, right = max(nums)
        while left < right:
            mid = (left + right) // 2
            if can(mid):
                right = mid
            else:
                left = mid + 1

        时间复杂度：O(n log max(nums))
        空间复杂度：O(1)
        """

        def can(max_balls: int) -> bool:
            """判断是否可以在 max_balls 限制下完成拆分"""
            ops = 0
            for n in nums:
                # 核心公式：将 n 拆成 ≤ max_balls 需要 (n-1)//max_balls 次
                ops += (n - 1) // max_balls
                if ops > maxOperations:  # 提前退出，优化
                    return False
            return True

        left, right = 1, max(nums)
        while left < right:
            mid = (left + right) // 2
            if can(mid):
                right = mid
            else:
                left = mid + 1

        return left
```

**与 kd 树的类比：**

| kd 树的构建 | 本题的二分 |
|------------|-----------|
| 递归分割空间 | 二分搜索可行解 |
| 中位数作为分割点 | mid 作为候选答案 |
| 左子树继续分割 | 缩小上界 right = mid |
| 右子树继续分割 | 增大下界 left = mid + 1 |
| 平衡树 → 快搜索 | 二分 → 避免全量判断 |

## 十一、完整测试与调试框架

```python
import random
import math
import time


def test_kdtree():
    """完整的 kd 树功能测试"""
    print("=" * 60)
    print("kd 树功能测试")
    print("=" * 60)

    # 生成测试数据
    n = 1000
    points = [(random.uniform(0, 100), random.uniform(0, 100))
              for _ in range(n)]
    target = (50, 50)

# 构建测试
    print(f"\n[1] 构建 kd 树 (n={n})")
    start = time.time()
    kdtree = KDTree(points, k=2)
    build_time = time.time() - start
    print(f"    构建耗时: {build_time:.4f}s")

# 最近邻测试
    print(f"\n[2] 最近邻搜索 (target={target})")
    start = time.time()
    nn_point, nn_dist = nearest_neighbor_search(kdtree.root, target)
    nn_time = time.time() - start
    print(f"    最近邻: {nn_point}, 距离: {nn_dist:.4f}")
    print(f"    查询耗时: {nn_time:.6f}s")

    # 暴力验证
    brute_force = min(points, key=lambda p: (p[0] - 50) ** 2 + (p[1] - 50) ** 2)
    print(f"    暴力验证: {brute_force}")

# KNN 测试
    k = 5
    print(f"\n[3] KNN 搜索 (k={k})")
    start = time.time()
    knn_results = k_nearest_neighbors(kdtree.root, target, k)
    knn_time = time.time() - start
    print(f"    Top-{k}: {knn_results}")
    print(f"    查询耗时: {knn_time:.6f}s")

    # 暴力验证
    brute_knn = sorted(points,
                       key=lambda p: (p[0] - 50) ** 2 + (p[1] - 50) ** 2)[:k]
    print(f"    暴力验证: {brute_knn}")

# 范围搜索测试
    region = [(40, 60), (40, 60)]
    print(f"\n[4] 范围搜索 (region={region})")
    start = time.time()
    range_results = range_search(kdtree.root, region)
    range_time = time.time() - start
    print(f"    区域内的点数: {len(range_results)}")
    print(f"    查询耗时: {range_time:.6f}s")

    # 暴力验证
    brute_range = [p for p in points
                   if 40 <= p[0] <= 60 and 40 <= p[1] <= 60]
    print(f"    暴力验证点数: {len(brute_range)}")

# 插入测试
    print(f"\n[5] 插入操作")
    new_point = (75, 25)
    insert(kdtree.root, new_point)
    nn_after, dist_after = nearest_neighbor_search(kdtree.root, (74, 24))
    print(f"    在 (74,24) 搜索最近邻: {nn_after} (期望接近 (75,25))")

# 性能对比
    print(f"\n[6] 性能对比 (100 次查询)")
    # kd 树查询
    start = time.time()
    for _ in range(100):
        q = (random.uniform(0, 100), random.uniform(0, 100))
        nearest_neighbor_search(kdtree.root, q)
    kdtree_total = time.time() - start

    # 暴力查询
    start = time.time()
    for _ in range(100):
        q = (random.uniform(0, 100), random.uniform(0, 100))
        min(points, key=lambda p: (p[0] - q[0]) ** 2 + (p[1] - q[1]) ** 2)
    brute_total = time.time() - start

    print(f"    kd 树: {kdtree_total:.4f}s (含建树: {build_time:.4f}s)")
    print(f"    暴力:  {brute_total:.4f}s")
    if kdtree_total < brute_total:
        print(f"    ✅ kd 树更快 (快 {brute_total / kdtree_total:.1f}x)")
    else:
        print(f"    ❌ kd 树更慢 (数据量小或维度问题)")

    print("\n" + "=" * 60)
    print("测试完成")
    print("=" * 60)


if __name__ == "__main__":
    test_kdtree()
```

## 十二、面试要点速记

### 最近邻搜索模板（记忆口诀：选搜检剪）

```python
def search(node, target, best, depth):
    if not node: return best

    axis = depth % k
    near, far = (node.left, node.right) if target[axis] < node.point[axis] \
                 else (node.right, node.left)

    best = search(near, target, best, depth + 1)  # ① 先搜近侧
    best = update(best, node)                       # ② 检查当前点
    if axis_dist < best_dist:                       # ③ 轴距剪枝
        best = search(far, target, best, depth + 1)

    return best
```

### 高频考点速查表

| 考点 | 关键回答 | 追问 |
|------|---------|------|
| 分割轴选择策略 | Round-Robin 或方差最大维度 | 方差策略优势：分割更紧凑，但需 O(kn) 计算 |
| 为什么取中位数 | 保证左右子树平衡，树高 O(log n) | 如果不取中位数？→ 可能退化为链表 |
| 回溯条件 | 轴距² < 当前最优距离² | 为什么是平方？→ 全程不开方，比较即可 |
| 维度诅咒的临界值 | k > 20 时退化 | 临界值为什么是 20？→ 需 n ≫ 2^k |
| 删除的替换策略 | 找右/左子树中target_axis上的最小值 | 为什么不用前驱（BST方式）？→ kd 树前驱不在子树中 |
| 与 R 树的区别 | 超平面 vs 最小外接矩形 | 谁更精确？→ 超平面更严格但更碎片化 |
| 动态维护 | 插入不保证平衡，定期重建 | Bkd 树：多棵 kd 树多版本合并 |
| 工程优化 | nth_element > sort，不开方 | 还有？→ 完全包含剪枝、懒惰删除 |

### LeetCode 题目索引

| 题号 | 题目 | 核心考点 | kd 树关联度 | 难度 |
|------|------|---------|:----------:|:----:|
| 973 | K Closest Points to Origin | 距离排序 + 堆/快速选择 | ⭐⭐⭐ | 🟢 |
| 658 | Find K Closest Elements | 一维最近邻 + 双指针 | ⭐⭐ | 🟢 |
| 447 | Number of Boomerangs | 距离频率 + 排列组合 | ⭐ | 🟢 |
| 149 | Max Points on a Line | 斜率 + 分数化简 | ⭐ | 🔴 |
| 1030 | Matrix Cells in Dist Order | 曼哈顿距离 + 桶排序 | ⭐⭐ | 🟢 |
| 1760 | Min Limit of Balls in Bag | 二分答案 + 分割思想 | ⭐⭐ | 🟡 |
| 2191 | (变体) 带标签最近邻 | kd 树标签过滤 | ⭐⭐⭐⭐ | 🟡 |

## 十三、总结

### 知识体系图谱

```
kd 树 ── 核心思想：交替轴分治空间
  ├── 构建算法：中位数分治（平衡树）
  │     └── 优化：quickselect / nth_element
  ├── 搜索算法家族
  │     ├── 最近邻搜索（1-NN）← 最核心
  │     ├── K 近邻搜索（KNN）← 最大堆维护
  │     ├── 范围搜索 ← 空间剪枝
  │     └── 范围计数 ← 完全包含优化
  ├── 动态算法
  │     ├── 插入 ← 标准 BST 插入（可能失衡）
  │     └── 删除 ← FindMin 替换（最复杂）
  └── 应用
        ├── 空间数据库（GIS）
        ├── 图像特征检索
        ├── 物理引擎碰撞检测
        └── 机器学习 KNN 加速
```

### 一句话总结

**kd 树是在多维空间中对二叉搜索树的自然推广，通过交替轴分割和回溯剪枝机制，将最近邻搜索从 $O(n)$ 暴力降低到平均 $O(\log n)$，是低维空间索引的经典基石结构。**

### 参考资料

- **Bentley, J. L. (1975).** "Multidimensional binary search trees used for associative searching." *Communications of the ACM*, 18(9), 509-517.
- **Friedman, J. H., Bentley, J. L., & Finkel, R. A. (1977).** "An algorithm for finding best matches in logarithmic expected time." *ACM Transactions on Mathematical Software*, 3(3), 209-226.
- **Wikipedia:** k-d tree
- **Scikit-learn:** `sklearn.neighbors.KDTree` — 工业级实现参考
- **FAISS (Facebook):** 大规模向量相似性搜索库
- **CLRS:** *Introduction to Algorithms* (第 3 版)，第 33 章"计算几何"
