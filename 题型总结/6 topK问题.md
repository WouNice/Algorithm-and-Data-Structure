# topK问题

topK问题是指在数组中寻找第K大(小)的数、数组中寻找前k大(小)的数以及一些相关的变形题目，如数组中寻找频数前K、寻找中位数等。

## 解决思路

### 直接sort排序，再取第K个值，时间复杂度O(nlogn)

### 快速选择

与快速排序类似，在一次递归调用中，寻找一个基准值，将数组分成两部分，左边的值都小于等于基准值，右边的值都大于等于基准值，然后根据k的条件对左边或者右边进行递归调用；与快速排序不同的是只需要对一边进行递归，所以最优的情况下时间复杂度O(n)

一般为了避免极端情况的出现，将数组切分之后元素全部集中在左边或者右边，时间复杂度可能退化到O(n*n)，常常引入随机化或者选取中点作为基准值

```java
// 快速选择代码，这边选取中点作为基准值，然后双指针两边不停的跟基准值比较
// 最后j左边的值都小于基准值，i右边的值都大于基准值
private void quickSelect(int[] nums, int left, int right, int k) {
    if (left >= right) return;
    int baseVal = nums[(left + right) >> 1];
    int i = left, j = right;
    while (i <= j) {
        // 降序的话，这边是大于baseVal
        while (i <= j && nums[i] < baseVal) {
            i++;
        }
        // 降序的话，这边是小于baseVal
        while (i <= j && nums[j] > baseVal) {
            j--;
        }
        if (i <= j) {
            swap(nums, i++, j--);
        }
    }
    if (k >= i) {
        quickSelect(nums, i, right, k);
    } else if (k <= j) {
        quickSelect(nums, left, j, k);
    }
}
```

### 堆排序

手写堆排，创建大顶堆(小顶堆)，再调整堆的过程中找到第K个值

创建堆时间复杂度O(n)

k次调整堆O(k*logn)

### 优先队列

利用现有API，优先队列：约束队列中存储的个数，超过k要进行移除操作，小于k添加，使时间复杂度限制在O(logk)

每个节点遍历一次，O(n*logk)

### 二叉搜索树

BST和大根堆的思路差不多～BST的优势就是求得的前K个数字保证是有序的。

## 例题

### 数组中的第K个最大元素

>   leetcode链接：https://leetcode.cn/problems/kth-largest-element-in-an-array/description/

快速选择：

>   寻找第K大，与每次快速选择得基准值位置进行比较，确定下一次需要递归的区间或者找到直接结束

```
public int findKthLargest(int[] nums, int k) {
        // nums.length - k 升序之后的下标
        quickSelect(nums, k - 1, 0, nums.length - 1);
        return nums[k - 1];
    }

    private void quickSelect(int[] nums, int k, int left, int right) {
        if (left >= right) return;
        // 确定一个数作为基准值，然后将大于等于他的放到右边，小于等于的放到左边
        int basic = nums[(left + right) >> 1];
        int i = left, j = right;
        while (i <= j) {
            // 降序
            while (i <= j && nums[i] > basic) {
                i++;
            }
            // 降序
            while (i <= j && nums[j] < basic) {
                j--;
            }
            if (i <= j) {
                swap(nums, i++, j--);
            }
        }
        if (k >= i) {
            quickSelect(nums, k, i, right);
        } else if (k <= j) {
            quickSelect(nums, k, left, j);
        }
    }

    private void swap(int[] nums, int i, int j) {
        if (i != j) {
            int temp = nums[i];
            nums[i] = nums[j];
            nums[j] = temp;
        }
    }
```

堆排序

>   构建大顶堆，根节点即为最大的值，然后将根节点和最后一个节点交换，再重新构建大顶堆，如此需要调整K次

```
/*
    基于堆排序进行选择
    每次构建完大顶堆的时候，堆顶根节点的值即为最大值
    那只要调整k次大顶堆，即第k大的数就在队顶
     */
public int findKthLargest(int[] nums, int k) {
    int size = nums.length;
    buildMaxHeap(nums, size);
    // 只要调整k次大顶堆，即第k大的数就在队顶
    for (int i = size - 1, j = 1; i > 0 && j < k; i--, j++) {
        swap(nums, 0, i);
        size--;
        handleHeap(nums, 0, size);
    }
    // 堆顶即为最大元素
    return nums[0];
}

private void buildMaxHeap(int[] nums, int length) {
    // 第一个非叶子节点 为 length/2-1
    for (int i = length / 2 - 1; i >= 0; i--) {
        // 处理堆，是的每一个非叶子节点都大于等于他的左右节点
        handleHeap(nums, i, length);
    }
}

private void handleHeap(int[] nums, int i, int length) {
    int left = i * 2 + 1, right = i * 2 + 2;
    int maxIndex = i;
    if (left < length && nums[maxIndex] < nums[left]) {
        maxIndex = left;
    }
    if (right < length && nums[maxIndex] < nums[right]) {
        maxIndex = right;
    }
    // 需要交换
    if (i != maxIndex) {
        swap(nums, i, maxIndex);
        // 交换之后可能会导致子节点作为父节点的时候 破坏了大顶堆的特性，需要继续处理maxIndex的顶堆特性
        handleHeap(nums, maxIndex, length);
    }
}

private void swap(int[] nums, int i, int j) {
    if (i != j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```

### 前 K 个高频元素

>   leetcode链接：https://leetcode.cn/problems/top-k-frequent-elements/description/

稍做转换即确定【出现次数】数组得topK问题

适用于选择前k个元素，但是不要求他们的输出顺序

优先队列：优先队列在topK中的解法，主要就是约束队列中存储的个数，超过k要进行移除操作，小于k添加

```java
/*
    topK问题：类似于LeetCode215，在数组中寻找前K大的数
    这边转换下就是在 【出现次数数组】中寻找TopK
     */
public int[] topKFrequent(int[] nums, int k) {
    int[] ans = new int[k];
    Map<Integer, Integer> map = new HashMap<>();
    for (int num : nums) {
        map.put(num, map.getOrDefault(num, 0) + 1);
    }
    // 使用堆排序，使用现有的API优先队列实现
    // 优先队列add、poll的时间复杂度 O(logn)，涉及到调整堆，处理堆的高度次
    // 所以在涉及到优先队列的时候最好是做约束，堆中的个数，降低复杂度；明确堆中个数超过多少时就进行poll...
    PriorityQueue<int[]> queue = new PriorityQueue<>(((o1, o2) -> o1[1] - o2[1]));
    for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
        // 长度小于k的时候直接入队
        if (queue.size() < k) {
            queue.add(new int[]{entry.getKey(), entry.getValue()});
        } else {
            // 长度大于等于k，判断当前是否大于队头元素（最小元素）
            if (entry.getValue() > queue.peek()[1]) {
                queue.poll();
                queue.add(new int[]{entry.getKey(), entry.getValue()});
            }
        }
    }
    // 出队
    for (int i = 0; i < k; i++) {
        ans[i] = queue.poll()[0];
    }
    return ans;
}
```

快速选择：快速选择前k个元素，与选择第K个类似，在最后数组中的前k个即为答案

```
public int[] topKFrequent(int[] nums, int k) {
        int[] ans = new int[k];
        Map<Integer, Integer> map = new HashMap<>();
        for (int num : nums) {
            map.put(num, map.getOrDefault(num, 0) + 1);
        }
        int[][] numAndCount = new int[map.size()][2];
        int i = 0;
        for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
            numAndCount[i][0] = entry.getKey();
            numAndCount[i][1] = entry.getValue();
            i++;
        }
        quickSelect(numAndCount, 0, numAndCount.length - 1, k - 1);
        for (int m = 0; m < k; m++) {
            ans[m] = numAndCount[m][0];
        }
        return ans;
    }

    private void quickSelect(int[][] numAndCount, int left, int right, int k) {
        if (left >= right) return;
        int baseVal = numAndCount[(left + right) >> 1][1];
        int i = left, j = right;
        while (i <= j) {
            while (i <= j && numAndCount[i][1] > baseVal) {
                i++;
            }
            while (i <= j && numAndCount[j][1] < baseVal) {
                j--;
            }
            if (i <= j) {
                swap(numAndCount, i++, j--);
            }
        }
        if (k >= i) {
            quickSelect(numAndCount, i, right, k);
        } else if (k <= j) {
            quickSelect(numAndCount, left, j, k);
        }
    }

    private void swap(int[][] numAndCount, int i, int j) {
        int[] temp = numAndCount[i];
        numAndCount[i] = numAndCount[j];
        numAndCount[j] = temp;
    }
```

### 最接近原点的 K 个点

>   leetcode链接：https://leetcode.cn/problems/k-closest-points-to-origin/description/

优先队列：大顶堆，平分和大的在队头，当添加到队列的元素长度超过k，判断是否再入队

```
public int[][] kClosest02(int[][] points, int k) {
    int[][] ans = new int[k][2];
    PriorityQueue<int[]> queue = new PriorityQueue<>((o1, o2) -> (o2[0] * o2[0] + o2[1] * o2[1]) - (o1[0] * o1[0] + o1[1] * o1[1]));
    for (int[] point : points) {
        if (queue.size() < k) {
            queue.add(point);
        } else {
            int[] peek = queue.peek();
            int peekVal = peek[0] * peek[0] + peek[1] * peek[1];
            if (point[0] * point[0] + point[1] * point[1] < peekVal) {
                queue.add(point);
                queue.poll();
            }
        }
    }
    int i = 0;
    while (!queue.isEmpty()) {
        int[] point = queue.poll();
        ans[i][0] = point[0];
        ans[i++][1] = point[1];
    }
    return ans;
}
```

快速选择

```
/*
    利用快排的思想寻找topK
    快速选择前k个数，比较基准值base的位置和k的大小，确定边界值，进行递归调用
     */
    public int[][] kClosest(int[][] points, int k) {
        quickSelect(points, 0, points.length - 1, k - 1);
        int[][] ans = new int[k][2];
        for (int i = 0; i < k; i++) {
            ans[i][0] = points[i][0];
            ans[i][1] = points[i][1];
        }
        return ans;
    }

    private void quickSelect(int[][] points, int left, int right, int k) {
        if (left >= right) return;
        int baseVal = getDist(points[(left + right) >> 1]);
        int i = left, j = right;
        // 双指针尽量将基准值落在中间位置
        while (i <= j) {
            while (i <= j && getDist(points[i]) < baseVal) {
                i++;
            }
            while (i <= j && getDist(points[j]) > baseVal) {
                j--;
            }
            if (i <= j) {
                swap(points, i++, j--);
            }
        }
        // 最后j>i，i右边的数都大于左边
        if (k <= j) {
            // 最后比base值小的都在j和j的右边
            quickSelect(points, left, j, k);
        } else if (k >= i) {
            // 最后比base值大的都在i和i的右边
            quickSelect(points, i, right, k);
        }
    }

    private void swap(int[][] points, int i, int j) {
        int[] temp = points[i];
        points[i] = points[j];
        points[j] = temp;
    }

    private int getDist(int[] point) {
        return point[0] * point[0] + point[1] * point[1];
    }
```

### 找出数组中的第 K 大整数

>   leetcode链接：https://leetcode.cn/problems/find-the-kth-largest-integer-in-the-array/description/

快速选择：基本与寻找第k大正数一样，只不过这里需要自定义string的比较器

```
public String kthLargestNumber(String[] nums, int k) {
        quickSelect(nums, 0, nums.length - 1, k - 1);
        return nums[k - 1];
    }

    private void quickSelect(String[] nums, int left, int right, int k) {
        if (left >= right) return;
        String baseVal = nums[(left + right) >> 1];
        int i = left, j = right;
        while (i <= j) {
            while (i <= j && compare(nums[i], baseVal) > 0) {
                i++;
            }
            while (i <= j && compare(nums[j], baseVal) < 0) {
                j--;
            }
            if (i <= j) {
                swap(nums, i++, j--);
            }
        }
        if (k >= i) {
            quickSelect(nums, i, right, k);
        } else if (k <= j) {
            quickSelect(nums, left, j, k);
        }
    }

    private void swap(String[] nums, int i, int j) {
        String temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }

    private int compare(String num1, String num2) {
        int len1 = num1.length(), len2 = num2.length();
        if (len1 != len2) {
            return len1 - len2;
        } else {
            return num1.compareTo(num2);
        }
    }
```

