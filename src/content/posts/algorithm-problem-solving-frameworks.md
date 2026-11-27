---
title: 数据结构与算法刷题笔记（从双指针到动态规划）
published: 2026-11-27
description: 按题目线索选择算法，整理双指针、链表、滑动窗口、二叉树、回溯、BFS、动态规划、贪心和分治的解题步骤与 Java 模板。
tags: [算法, 数据结构, 刷题笔记, Java]
category: 技术笔记
draft: false
slug: algorithm-problem-solving-frameworks
---

拿到一道题，先想清楚**如果时间不限，我会枚举什么**，再找能减少重复工作的信息。有序数组可以直接排除一段候选，滑动窗口可以沿用上一个区间的统计，动态规划可以把相同子问题的答案保存下来。这些框架都能从一个说得清楚的朴素解法推出来。

这篇笔记以 labuladong 第零章为参考，按做题时的判断顺序整理。代码统一用 Java。刷题时，把方法和它依赖的辅助方法放进 `class Solution`，有 `import` 的模板把导入语句放到类声明上方。题号均为 LeetCode 题号。[第零章导读](https://labuladong.online/zh/algo/intro/core-intro/) · [框架思维](https://labuladong.online/zh/algo/essential-technique/algorithm-summary/)

## 拿到题目，先选搜索范围

| 题目线索 | 优先考虑 | 写代码前先想清楚 |
| --- | --- | --- |
| 有序数组里找一对数，或从两端处理 | 左右双指针 | 移动某一端时排除了哪些候选 |
| 原地删除、去重、移动元素 | 读写双指针 | 已经写好的前缀是什么 |
| 单链表中点、环、倒数第 `k` 个位置 | 快慢指针 | 两指针的速度或间距是什么 |
| 连续子串或子数组，区间增删可更新统计 | 滑动窗口 | 何时收缩，何时记答案 |
| 树的路径、子树大小或高度 | DFS / 递归 | 需要父节点带下来的信息，还是子节点返回的结果 |
| 列出所有合法的排列、组合或方案 | 回溯 | 当前路径、可选项和终止条件是什么 |
| 每步代价相同，求最少几步到达 | BFS | 怎样表示一个状态，怎样生成它的邻居 |
| 求最优值、方案数或可行性，且子问题重复 | 动态规划 | 状态能否描述后续决策所需的全部信息 |
| 局部选择可以被证明兼容全局最优 | 贪心 | 为什么可以舍弃其他选择 |
| 能拆成独立子问题，再高效合并 | 分治 | 子问题怎么缩小，合并需要多少时间 |

先用一个小输入手算，记下每轮必须成立的条件。比如，原地筛选数组时，读指针之前的元素已经处理过，写指针之前的元素是应当保留的结果。这个在每轮都成立的条件叫**循环不变量**，它比记住指针的名字更有用。

## 数组双指针

有序数组求两数之和时，把指针放到两端。当前和偏小，把左端固定后，与区间内其他候选相加只会得到更小或相同的和，因此可以丢掉左端。当前和偏大，同理可以丢掉右端。模板返回零基下标，找不到时返回 `null`。练习第 167 题时，需要按题目要求返回从 1 开始的下标。

```java
public int[] findSortedPair(int[] nums, long target) {
    int left = 0;
    int right = nums.length - 1;

    while (left < right) {
        long sum = (long) nums[left] + nums[right];
        if (sum == target) {
            return new int[] {left, right};
        }
        if (sum < target) {
            left++;
        } else {
            right--;
        }
    }
    return null;
}
```

原地修改数组时，给两个下标不同的职责。读指针负责读完数组，写指针始终指着下一个可写位置。下面用这个模式移除指定值，函数返回有效前缀的长度。

```java
public int keepOtherValues(int[] nums, int unwanted) {
    int write = 0;
    for (int value : nums) {
        if (value != unwanted) {
            nums[write] = value;
            write++;
        }
    }
    return write;
}
```

两种写法都是 `O(n)` 时间、`O(1)` 额外空间。有序去重时，把保留条件改为“当前值和上一个已写入的值不同”。回文判断可以从两端向内比较，找回文子串则可以从每个中心向外扩，分别处理单字符和双字符中心。有序查找还可以用二分，每轮依据中点把候选区间减半。

**练习题** 167 有序数组的两数之和 → 27 移除元素 → 26 有序数组去重 → 5 最长回文子串。[数组双指针原文](https://labuladong.online/zh/algo/essential-technique/array-two-pointers-summary/)

## 单链表的节点位置关系

链表只能沿着 `next` 往前走。要找倒数第 `k` 个节点，可以先用快指针走 `k` 步，再让两个指针同步前进。快指针离开链尾时，慢指针正好指着目标节点。下面的模板使用 LeetCode 提供的 `ListNode`。当 `k` 小于 1 或链表长度不足时，返回 `null`。

```java
public ListNode kthFromEnd(ListNode head, int k) {
    if (k < 1) {
        return null;
    }

    ListNode ahead = head;
    for (int step = 0; step < k; step++) {
        if (ahead == null) {
            return null;
        }
        ahead = ahead.next;
    }

    ListNode behind = head;
    while (ahead != null) {
        ahead = ahead.next;
        behind = behind.next;
    }
    return behind;
}

public ListNode middleNode(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow; // 偶数个节点时得到后一个中点
}
```

两倍速度同样能用于判环。若有环，快指针进入环后每轮相对慢指针多走一步，最终会追上它。相遇后，把其中一个指针放回表头，再让两者每次各走一步。假设入环前走了 `a` 步，相遇点距离入口 `b` 步，那么 `a + b` 是环长的整数倍，所以两个指针会在入口再次相遇。

```java
public ListNode cycleEntry(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) {
            ListNode fromHead = head;
            while (fromHead != slow) {
                fromHead = fromHead.next;
                slow = slow.next;
            }
            return fromHead;
        }
    }
    return null;
}
```

| 常见任务 | 该维护什么 |
| --- | --- |
| 删除倒数第 `k` 个节点 | 用虚拟头节点统一删除头节点的情况，再找目标的前驱 |
| 合并两个有序链表 | 两个头指针各指向待处理的最小节点，把较小者接到结果尾部 |
| 按阈值拆分链表 | 分别维护两条链的尾指针，保留各组内的顺序，接起两段前把后一段的尾部置空 |
| 合并 `k` 个有序链表 | 用最小堆保存每条链的当前头节点，也可以两两分治合并 |
| 寻找两条无环链表的交点 | 走到 `null` 就换到另一条链的起点，双方走过的总长度对齐。比较节点引用 |

单链表快慢指针通常只用 `O(1)` 额外空间。处理两条链相交时，两个指针都走完两条链仍未相交，就会同时到达 `null`。

**练习题** 21 合并有序链表 → 19 删除倒数第 N 个节点 → 876 链表中点 → 142 环入口 → 160 链表相交。[链表双指针原文](https://labuladong.online/zh/algo/essential-technique/linked-list-skills-summary/)

## 滑动窗口的收缩和更新时机

滑动窗口适合处理**连续**区间。把当前区间记为 `[left, right)`，右端加入元素，左端移除元素，计数表、区间和或其他指标随之更新。能一路向右走的前提是，你能证明丢掉的左端以后也用不着，或者所有更长的同起点区间都不会改善答案。

| 目标 | 收缩时机 | 更新答案的位置 |
| --- | --- | --- |
| 固定长度 `k` | 扩张后超过 `k` 就缩 | 长度等于 `k` 时更新 |
| 最长合法区间 | 当前区间不合法时缩 | 缩到合法后更新 |
| 最短合法区间 | 当前区间仍然合法时缩 | 每次缩之前更新 |

例如，求最多包含 `limit` 个零的最长区间，其中 `limit` 是非负整数。窗口中的 `zeros` 可以随增删及时更新，超过上限时，继续往右扩也不会让零变少，所以可以把左端一路挪到窗口重新合法。

```java
public int longestWithAtMostKZeros(int[] nums, int limit) {
    int left = 0;
    int right = 0;
    int zeros = 0;
    int best = 0;

    while (right < nums.length) {
        if (nums[right] == 0) {
            zeros++;
        }
        right++;

        while (zeros > limit) {
            if (nums[left] == 0) {
                zeros--;
            }
            left++;
        }
        best = Math.max(best, right - left);
    }
    return best;
}
```

最短区间的更新点正好在收缩之前。以非负数组中求和至少为正数 `target` 的最短区间为例，固定右端后，逐个删除左端元素。窗口仍达标时就更新最短长度，直到当前窗口的和小于 `target`。

```java
public int shortestAtLeast(int[] nums, long target) {
    int left = 0;
    long sum = 0;
    int best = Integer.MAX_VALUE;

    for (int right = 0; right < nums.length; right++) {
        sum += nums[right];
        while (sum >= target) {
            best = Math.min(best, right - left + 1);
            sum -= nums[left];
            left++;
        }
    }
    return best == Integer.MAX_VALUE ? 0 : best;
}
```

第二段代码的 `right` 是刚加入的元素下标，对应 `[left, right + 1)`。两种写法中，每个元素至多进入和离开一次，统计更新为 `O(1)` 时，总时间就是 `O(n)`。做最小覆盖子串时，可以把区间和换成字符频次，达标以后同样先记答案再缩。遇到含负数的区间求和题，移除左端可能让和变大，先检查收缩依据。例如“和恰好为 `k` 的子数组数量”可以用前缀和加哈希表。

**练习题** 1004 最大连续 1 的个数 III → 3 无重复字符的最长子串 → 209 长度最小的子数组 → 76 最小覆盖子串。[滑动窗口原文](https://labuladong.online/zh/algo/essential-technique/sliding-window-framework/)

## 二叉树的两种递归思路

把递归调用画成树，每一层只负责定义当前问题和它的下一步。写函数前先定下**参数含义、返回值、终止条件**，然后把递归调用当成已经算好的子问题使用。例如最大深度，可以用两种写法练习同一个定义。

- **遍历思路**。进入节点时就知道从根走到了第几层，于是把 `depth` 作为参数传给孩子。类似场景还有根到叶子的路径和、收集所有路径。
- **分解思路**。约定 `depthFrom(node)` 返回以 `node` 为根的树的最大深度。拿到左、右子树的结果后，加上当前节点即可。类似场景还有计算子树大小、检查平衡、计算树的直径。

两个模板都使用 LeetCode 提供的 `TreeNode`。遍历写法用长度为 1 的数组在递归调用之间共享本次遍历的最大深度。

```java
public int maxDepthFromRoot(TreeNode root) {
    int[] deepest = {0};
    visitDepth(root, 1, deepest);
    return deepest[0];
}

private void visitDepth(TreeNode node, int depth, int[] deepest) {
    if (node == null) {
        return;
    }
    deepest[0] = Math.max(deepest[0], depth);
    visitDepth(node.left, depth + 1, deepest);
    visitDepth(node.right, depth + 1, deepest);
}

public int depthFrom(TreeNode node) {
    if (node == null) {
        return 0;
    }
    int left = depthFrom(node.left);
    int right = depthFrom(node.right);
    return 1 + Math.max(left, right);
}
```

前序位置在访问两个孩子之前，适合处理从父节点传下来的深度、剩余路径和。中序位置在两个孩子之间，二叉搜索树的中序遍历可以按从小到大访问节点。后序位置在两个孩子都返回之后，适合汇总子树信息。例如树的直径，可以在每个节点用左右子树的高度之和更新全局答案，再把该节点的高度返回给父节点。

上面两种最大深度写法都是 `O(n)` 时间、`O(h)` 调用栈，`h` 是树高。链状树的 `h` 可以达到 `n`，输入很深时可以考虑显式栈。把二叉树换成多叉决策树，同样可以用 DFS 探索路径。如果问题定义只依赖少量状态，而且很多调用落到同一个状态，就有机会用动态规划复用结果。

**练习题** 104 最大深度 → 112 路径总和 → 94 中序遍历 → 543 二叉树直径。[二叉树纲领](https://labuladong.online/zh/algo/essential-technique/binary-tree-summary/) · [递归思维](https://labuladong.online/zh/algo/essential-technique/understand-recursion/)

## 回溯中的选择、记录和撤销

要输出所有可行方案时，可以把每一步选择画成决策树。进入一条分支前，先把选择写进当前路径。回来时撤销这次选择，下一条分支就能从同一状态出发。能提前证明某条路径没有可行后续时，直接剪掉它。

下面以从不同数字中选 `k` 个为例。`start` 限制下一个下标，确保同一组数字只产生一次。可选元素少于剩余名额时，可以停止在该层继续尝试。保存答案时复制 `path`，否则后续撤销会把已保存的答案也改掉。

```java
import java.util.ArrayList;
import java.util.List;

public List<List<Integer>> chooseK(int[] values, int k) {
    List<List<Integer>> result = new ArrayList<>();
    if (k >= 0 && k <= values.length) {
        collectCombinations(values, k, 0, new ArrayList<>(), result);
    }
    return result;
}

private void collectCombinations(
        int[] values, int k, int start, List<Integer> path, List<List<Integer>> result) {
    int remaining = k - path.size();
    if (remaining == 0) {
        result.add(new ArrayList<>(path));
        return;
    }

    for (int i = start; i <= values.length - remaining; i++) {
        path.add(values[i]);
        collectCombinations(values, k, i + 1, path, result);
        path.remove(path.size() - 1);
    }
}
```

| 变体 | 怎么改框架 |
| --- | --- |
| 子集 | 进入每个节点时都保存当前路径，包括空路径，然后继续向后选 |
| 排列 | 每层从头枚举，用 `used[i]` 记住当前路径里已使用的下标 |
| 输入有重复值的组合或子集 | 先排序，同一层中若 `i > start` 且与前一个值相同就跳过 |
| 输入有重复值的排列 | 先排序，若当前值与前一个值相同且前一个下标尚未在当前路径使用，就跳过当前下标 |
| 元素可重复使用的组合 | 递归继续传 `i`，并用固定长度或逐步下降的正数余额保证终止 |

复杂度要把输出算进去。输出 `C(n, k)` 个长度为 `k` 的组合，光复制结果就需要 `O(k × C(n, k))` 时间和结果空间，此外还有搜索过程和递归栈。

**练习题** 77 组合 → 78 子集 → 46 全排列 → 47 全排列 II → 39 组合总和。[回溯原文](https://labuladong.online/zh/algo/essential-technique/backtrack-framework/) · [排列、组合和子集](https://labuladong.online/zh/algo/essential-technique/permutation-combination-subset-all-in-one/)

## 用 BFS 求最少步数

迷宫里的格子、拼图的摆法、游戏角色的当前位置和剩余次数，都可以是状态。一次合法操作对应一条边。当每条边都代表一步时，从起点按层向外扩展，第一次到达目标的层数就是最少步数。

下面把“如何找邻居”留给参数 `neighbors`，同一个框架可用于普通图和题目中临时生成的状态图。例如 `graph` 是邻接表时，可以传入 `node -> graph.get(node)`。用非空的编号或字符串标识状态。将新状态标为已访问的时机是**入队时**，这样同一层的多个节点就不会重复把它加入队列。

```java
import java.util.ArrayDeque;
import java.util.HashMap;
import java.util.Map;
import java.util.Queue;
import java.util.function.Function;

public <T> int shortestSteps(T start, T target, Function<T, Iterable<T>> neighbors) {
    Queue<T> queue = new ArrayDeque<>();
    Map<T, Integer> distance = new HashMap<>();
    queue.offer(start);
    distance.put(start, 0);

    while (!queue.isEmpty()) {
        T state = queue.poll();
        int steps = distance.get(state);
        if (state.equals(target)) {
            return steps;
        }

        for (T next : neighbors.apply(state)) {
            if (!distance.containsKey(next)) {
                distance.put(next, steps + 1);
                queue.offer(next);
            }
        }
    }
    return -1;
}
```

显式邻接表上的复杂度是 `O(V + E)` 时间、`O(V)` 额外空间。若同一个位置在“剩余一次破墙机会”和“机会用完”时能做的操作不同，就要把剩余次数也编码进状态。需要还原路径时，可以在入队时另存前驱。步间代价变化时，要根据边权选择对应算法，例如非负边权可考虑 Dijkstra。

**练习题** 102 二叉树层序遍历 → 752 打开转盘锁 → 773 滑动谜题。[BFS 原文](https://labuladong.online/zh/algo/essential-technique/bfs-framework/)

## 动态规划的状态与转移

动态规划常见于最优值、计数和可行性问题。先尝试把答案表达成一些更小状态的答案。若不同选择会落到同一状态，保存状态的结果就可以省去重复求解。

以求最少硬币数为例，假设面额为正整数、每种硬币都可无限使用。

1. **状态**。`best[a]` 表示恰好凑出金额 `a` 所需的最少硬币数。
2. **选择与转移**。最后一枚硬币若选 `coin`，前面必须已经凑出 `a - coin`，代价是 `best[a - coin] + 1`。在所有可选面额中取最小值。
3. **初始化**。金额 `0` 需要 `0` 枚硬币。任何可达金额需要的硬币数都不超过 `amount`，因此可以用 `amount + 1` 标记不可达。
4. **计算顺序与答案**。从小金额算到大金额，使读到的依赖值都已计算。最后返回 `best[amount]`，不可达时返回 `-1`。

```java
import java.util.Arrays;

public int minimumCoins(int[] coins, int amount) {
    int unreachable = amount + 1;
    int[] best = new int[amount + 1];
    Arrays.fill(best, unreachable);
    best[0] = 0;

    for (int sum = 1; sum <= amount; sum++) {
        for (int coin : coins) {
            if (coin <= sum && best[sum - coin] != unreachable) {
                best[sum] = Math.min(best[sum], best[sum - coin] + 1);
            }
        }
    }
    return best[amount] == unreachable ? -1 : best[amount];
}
```

也可以自顶向下地写 `solve(a)`，递归枚举最后一枚硬币，用备忘录保存每个金额第一次算出的结果，包括不可达的状态。以上迭代版耗时 `O(amount × coins.length)`，额外空间为 `O(amount)`。

状态要随题意变化。硬币数量有限时，仅记录金额无法区分哪些硬币还能用。把问题改成“有多少种组合”时，可以外层遍历面额、内层从小到大遍历金额，让每个组合按固定的面额顺序累积。循环次序会影响计算的是组合还是不同顺序的序列。

**练习题** 70 爬楼梯 → 198 打家劫舍 → 322 零钱兑换 → 518 零钱兑换 II → 1143 最长公共子序列。[动态规划原文](https://labuladong.online/zh/algo/essential-technique/dynamic-programming-framework/)

## 贪心选择的正确性

面对区间选择、资源分配或覆盖范围等题目，先找一个局部选择，再证明至少存在一个全局最优解包含这个选择。**交换论证**是常见的证明方式。取一个最优解，把其中的一个选择换成贪心选择，证明可行性和解的质量都能保住。于是剩下的问题可以用同一个规则继续处理。

例如，从若干 `start < end` 的活动区间中选出尽量多的互不重叠活动，两个区间的端点可以相接。每次选结束时间最早、且能接在已选活动后面的活动。任取一个最优安排，把第一个活动换成结束最早的活动，后面的活动依然都能开始，所选数量也没变。

```java
import java.util.Arrays;
import java.util.Comparator;

public int maxNonOverlappingIntervals(int[][] intervals) {
    int[][] ordered = intervals.clone();
    Arrays.sort(ordered, Comparator.comparingInt(interval -> interval[1]));
    int count = 0;
    int previousEnd = Integer.MIN_VALUE;

    for (int[] interval : ordered) {
        if (interval[0] >= previousEnd) {
            count++;
            previousEnd = interval[1];
        }
    }
    return count;
}
```

这一题耗时主要在排序，为 `O(n log n)`。遇到“看起来每次选最大的就行”的规则，可以先手算一个反例。例如用 `1、3、4` 凑 `6`，先拿 `4` 需要三枚硬币，而两枚 `3` 就够了。

**练习题** 435 无重叠区间 → 55 跳跃游戏 → 452 用最少数量的箭引爆气球。[贪心原文](https://labuladong.online/zh/algo/essential-technique/greedy/)

## 分治的拆分和合并

分治适合把问题拆成规模更小、能够分别求解的部分。写代码时先确定最小可直接处理的规模，再确定拆分边界和合并规则。例如归并排序先递归排好左右半段，再用两个下标取较小值，一次扫描合成有序区间。下面返回排序后的副本，原数组可以继续使用。

```java
public int[] mergeSort(int[] values) {
    int[] sorted = values.clone();
    int[] buffer = new int[sorted.length];
    sortRange(sorted, buffer, 0, sorted.length);
    return sorted;
}

private void sortRange(int[] sorted, int[] buffer, int start, int end) {
    if (end - start < 2) {
        return;
    }

    int mid = start + (end - start) / 2;
    sortRange(sorted, buffer, start, mid);
    sortRange(sorted, buffer, mid, end);

    int left = start;
    int right = mid;
    for (int write = start; write < end; write++) {
        if (left < mid && (right == end || sorted[left] <= sorted[right])) {
            buffer[write] = sorted[left++];
        } else {
            buffer[write] = sorted[right++];
        }
    }
    System.arraycopy(buffer, start, sorted, start, end - start);
}
```

每层合并总共扫描 `O(n)` 个元素，递归深度是 `O(log n)`，总时间 `O(n log n)`，额外空间为 `O(n)`。统计逆序对也能用同一个合并过程。当右半段的当前元素小于左半段的当前元素，左半段剩余的每个元素都与它构成逆序对。

**练习题** 912 排序数组，尝试归并排序 → 23 合并 K 个升序链表，尝试两两归并。[分治原文](https://labuladong.online/zh/algo/essential-technique/divide-and-conquer/)

## 复杂度和边界检查

| 写法 | 时间怎么算 | 空间算哪些 |
| --- | --- | --- |
| 双指针、滑动窗口 | 数指针总共移动多少次，再乘每次更新的成本 | 频次表等额外状态 |
| 树递归、回溯 | 数访问的调用或分支，加上每次处理和复制答案的成本 | 最大调用深度、路径状态、输出 |
| 图搜索 | 数访问的状态和检查的边 | 已访问集合、队列或栈 |
| 动态规划 | 状态数量乘每个状态评估的转移数量 | 状态表，记忆化递归还要算调用栈 |
| 分治 | 分开计算各层处理量，再加总 | 合并缓冲区、递归栈 |

开始写代码前，用输入上限筛掉明显过慢的方案。写完再手算空输入、单元素、重复值、无解和刚好卡在边界上的例子。单独检查实现中的操作成本，例如反复调用 `ArrayList.remove(0)` 会移动后面的元素。BFS 模板用 `ArrayDeque` 在队尾插入、队头取出，入队和出队的摊还时间是 `O(1)`。[复杂度分析原文](https://labuladong.online/zh/algo/essential-technique/complexity-analysis/)

复习可以按 `167 → 19 → 3 → 104 → 77 → 752 → 322 → 435 → 912` 的顺序各练一个框架。每道题做完留三行笔记，分别写下**选这个框架的依据、保证没有漏解的不变量、最容易写错的边界**。下次遇到变体，先尝试重新写出这三行，再打开代码对照。
