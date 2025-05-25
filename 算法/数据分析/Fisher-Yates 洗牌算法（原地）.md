# Fisher-Yates 洗牌算法（原地）

## 应用场景

随机打乱一个已存在于内存中的数组。

## 算法步骤

- 不妨设有待乱序的长度为 n 的数组 L ；

- 遍历 n 次，在第 i 次循环中（0 ≤ i < n）：

    - 在 [i,n) 中随机抽取一个下标 j；

    - 将第 i 个元素与第 j 个元素交换。

其中数组 L 中的第 k ( k ≥ i ) 个元素为待乱序的数组，其长度为 n − i ；数组 L中的第 l   ( l < i ) 个元素为已乱序的数组，其长度为 i。n 次遍历完成后，数组
L已被乱序。

## 算法证明

结论：原数组L中的任意位置的数，被移动到乱序后数组的任意位置的概率是相同的。

证明：根据算法步骤，显然有，原数组中第 i 个位置的元素（0 ≤ i < n）移动到乱序后数组第 j 个位置（0 ≤ j < n） 的概率为

$P(i,j)=\frac{n-1}{n}\times\frac{n-2}{n-1}\times\cdots\times\frac{n-i}{n-i+1}\times\frac{1}{n-i}=\frac{1}{n}$

得证。

## 复杂度分析

- 时间复杂度：O(n)，其中 n 为待乱序数组长度。
- 空间复杂度：O(1)，原地乱序 nums 数组，仅使用 i、j 及调换 nums[i] 和 nums[j] 时使用的共 3 个中间变量。

## 代码实现

```python
import random
from typing import Any, List, NoReturn


def fisher_yates(nums: List[Any]) -> NoReturn:
    """Fisher-Yates 洗牌算法，原地乱序 nums 数组

    Parameters
    ----------
    nums : List[Any]
        待乱序数组
    """
    for i in range(len(nums)):
        j = random.randrange(i, len(nums))
        nums[i], nums[j] = nums[j], nums[i]
```

测试用例：

```
>>> nums = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
>>> fisher_yates(nums)
>>> nums
[3, 10, 8, 2, 1, 4, 7, 9, 6, 5]
>>> fisher_yates(nums)
>>> nums
[2, 10, 8, 1, 4, 7, 9, 3, 5, 6]
```
