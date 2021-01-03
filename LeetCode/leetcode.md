# LeetCode 总结

## LC 03 没有重复字符的最长字符子串

字符串 + 滑动窗口

```cpp
/*
滑动窗口时，一个指针快速移动，另一个指针需要保证窗口的合法性
*/
```

[题目](https://leetcode.com/problems/longest-substring-without-repeating-characters/)


[分析](https://www.youtube.com/watch?v=LupZFfCCbAU)


## LC 05 最长回文子串

字符串 + 回文子串 + DP

- DP

```cpp
/*
dp[i][j] : s[i] ~ s[j] 是否是回文串
*/
```

- 从中间开始找

```cpp
/*
分两种情况: s[index] 和 s[index] ~ s[index + 1]
s[index - step] ~ s[index + step]:  
s[index - step] ~ s[index + 1 + step]
*/
```

[题目](https://leetcode.com/problems/longest-palindromic-substring/)

[分析](https://www.bilibili.com/video/BV18J411j7Pb?from=search&seid=13947515736860821639)


## LC 22 生成括号

递归

```cpp
/*
由于始终在末尾增加字符，所以当中间变量出现 )( 时则是不合法的
*/
```

[题目](https://leetcode.com/problems/generate-parentheses/)

[分析](https://www.bilibili.com/video/BV1hb411i7t7?from=search&seid=11147594321615163316)


## LC 26 原地移除数组重复元素

双指针

[题目](https://leetcode.com/problems/remove-duplicates-from-sorted-array/)

## LC 32 括号字符串的最长合法性子串

DP + 栈

```cpp
/*
括号题主要是用栈
*/
```

[题目](https://leetcode.com/problems/longest-valid-parentheses/)


## LC 41 最小未出现的正整数

数组

```cpp
自定义 hash，使得各元素归位
```

[题目](https://leetcode.com/problems/first-missing-positive/submissions/)


## LC 44 字符串匹配

DP

```cpp
/*
dp[i][j] 表示 s 中前 i 个字符组成的子串和 p 中前 j 个字符组成的子串是否能匹配

dp[0][0] = true

当 s 为空，p 为连续的星号时，由于星号是可以代表空串的，所以只要 s 为空，那么连续的星号的位置都应该为 true，所以先将连续星号的位置都赋为 true

若 p 中第 j 个字符是星号，由于星号可以匹配空串，所以如果 p 中的前 j-1 个字符跟 s 中前 i 个字符匹配成功了的话（dp[i][j - 1]），则 dp[i][j] 也能为 true；由于型号可以匹配任意字符串，若 p 中的前 j 个字符跟 s 中的前 i-1 个字符匹配成功了的话（dp[i - 1][j]），则 dp[i][j] 也能为 true
*/
```

[题目](https://leetcode.com/problems/wildcard-matching/)


## LC 46 47 组合相关

回溯

[题目](https://leetcode.com/problems/permutations/)

[题目](https://leetcode.com/problems/permutations-ii/)


## LC 55 数组跳跃

贪心

[题目](https://leetcode.com/problems/jump-game/)


## LC 72 编辑距离

DP + 递归

```cpp
/*
dp[i][j] : word1[0 ~ i - 1] to word2[0 ~ j - 1]

dp[i][j] = 
            i if j == 0
            j if i == 0
            dp[i - 1][j - 1] if word1[i - 1] == word2[j - 1]
            min(dp[i - 1][j], dp[i][j - 1], dp[i - 1][j - 1]) + 1
*/
```

[题目](https://leetcode.com/problems/edit-distance/)

[分析](https://www.youtube.com/watch?v=Q4i_rqON2-E)


## LC 85 最大子矩形

DP

[题目](https://leetcode.com/problems/maximal-rectangle/)

[分析](https://zxi.mytechroad.com/blog/dynamic-programming/leetcode-85-maximal-rectangle/)

## LC 95 生成BST

二叉树 + DP + 递归

```cpp
/*
划分左右子树，递归构造

将区间  [1, n] 当作一个整体，然后需要将其中的每个数字都当作根结点，其划分开了左右两个子区间，然后分别调用递归函数，会得到两个结点数组，接下来要做的就是从这两个数组中每次各取一个结点，当作当前根结点的左右子结点


*/
```
[题目](https://leetcode.com/problems/unique-binary-search-trees-ii/)


## LC 123 188 数组最大 m 段和

DP

[题目](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/)

[题目](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/)

[分析](https://mp.weixin.qq.com/s/lQEj_K1lUY83QtIzqTikGA)


## LC 127 字典搜索

BFS + 双向 BFS

[题目](https://leetcode.com/problems/word-ladder/)


## LC 128 最长连续子序列

哈希

```cpp
/*
建立哈希表 hash(key, len)，存储以 key 为端点得子序列长度，每次更新只需更细连续区间端点的值
*/
```

[题目](https://leetcode.com/problems/longest-consecutive-sequence/)

[分析](https://www.youtube.com/watch?v=rc2QdQ7U78I)


## LC 146 设计 LRU

LRU

[题目](https://leetcode.com/problems/lru-cache/)

[分析](https://www.youtube.com/watch?v=q1Njd3NWvlY)

## LC 221 最大子矩形

DP

[题目](https://leetcode.com/problems/maximal-square/)

[分析](https://www.youtube.com/watch?v=vkFUB--OYy0)


## LC 207 210 课程安排

有向图 + 拓扑排序（用于判断单向图是否有环）

[题目](https://leetcode.com/problems/course-schedule/)

[分析](https://www.youtube.com/watch?v=M6SBePBMznU)


## LC 221 最大正方形面积

DP

```cpp
/*
dp[i][j] : 以 matrix[i][j] 为右下角的最大面积正方形边长
*/
```

[题目](https://leetcode.com/problems/maximal-square/)

[分析](https://www.youtube.com/watch?v=vkFUB--OYy0)


## LC 236 最低公共祖节点

DFS + 递归

```cpp
/*
首先看当前结点是否为空，若为空则直接返回空；若为 p 或 q 中的任意一个，也直接返回当前结点，否则的话就对其左右子结点分别调用递归函数

若 p 和 q 分别位于左右子树中，那么对左右子结点调用递归函数，会分别返回 p 和 q 结点的位置，而当前结点正好就是 p 和 q 的最小共同父结点，直接返回当前结点即可
*/
```

[题目](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/)

[分析](https://www.cnblogs.com/grandyang/p/4641968.html)


## LC 239 窗口内的最大值

BST + 单调队列


[题目](https://leetcode.com/problems/sliding-window-maximum/)

[分析](https://www.youtube.com/watch?v=2SXqBsTR6a8)


## LC 241 添加括号后所有计算结果

记忆化递归

```CPP
/*
在操作符的位置对前后的运算进行分割
*/
```

[题目](https://leetcode.com/problems/different-ways-to-add-parentheses/)


## LC 287 寻找重复元素

二分

```cpp
/*
在区间 [1, n] 中搜索，首先求出中点 mid，然后遍历整个数组，统计所有小于等于 mid 的数的个数，如果个数小于等于 mid，则说明重复值在 [mid+1, n] 之间，反之，重复值应在 [1, mid-1] 之间
*/
```

[题目](https://leetcode.com/problems/find-the-duplicate-number/)

## LC 312 打气球

DP

```cpp
/*
dp[i][j] : maxCoins(nums[i] ~ nums[j])
*/
```

[题目](https://leetcode.com/problems/burst-balloons/)

[分析](https://www.youtube.com/watch?v=z3hu2Be92UA&t=700s)

## LC 329 最长递增路径

DFS + DP

[题目](https://leetcode.com/problems/longest-increasing-path-in-a-matrix/)

[分析](https://www.bilibili.com/video/BV1mW411d7q8?from=search&seid=5099018830887943293)


## LC 337 树状结构的房屋抢劫

二叉树 + 递归

[题目](https://leetcode.com/problems/house-robber-iii/)


## LC 368 最大可整除子集合划分

DP

[题目](https://leetcode.com/problems/largest-divisible-subset/)


## LC 377 任意组合得到指定和

DP

[题目](https://leetcode.com/problems/combination-sum-iv/)


## LC 547 朋友圈

UnionFind

[题目](https://leetcode.com/problems/friend-circles/)

[分析](https://www.youtube.com/watch?v=HHiHno66j40)


## LC 410 分割数组得到子数组的最小和

DP

```cpp
/*
dp[i][j] : nums[0] ~ nums[j] split i groups

dp[1][j] = sum(nums[0] ~ nums[j])
dp[i][j] = min(max(dp[i - 1][k], sum(nums[j] ~ nums[k + 1])))
*/
```

[题目](https://leetcode.com/problems/split-array-largest-sum/)

[分析](https://www.youtube.com/watch?v=_k-Jb4b7b_0&t=668s)


## LC 416 划分数组子集合使集合和相等

DP

```cpp
/*
背包问题一般都可以进行状态压缩，压缩后需要反向更新状态

dp[i] : sum(subset(...)) == i
*/
```

[题目](https://leetcode.com/problems/partition-equal-subset-sum/)

[分析](https://www.youtube.com/watch?v=r6I-ikllNDM&t=382s)


## LC 474 子集合有指定的元素数目

DP

```cpp
/*
背包问题一般都可以进行状态压缩，压缩后需要反向更新状态

dp[i][j] : i -- 0, j -- 1
*/
```

[题目](https://leetcode.com/problems/ones-and-zeroes/)


## LC 494 添加运算符得到指定计算结果

DP + DFS

```cpp
/*
DFS 就是暴力搜索，会超时

dp[i][j] : sum(nums[0 ~ i]) == j cnt
dp[i][j] = dp[i - 1][j + nums[i]] + dp[i - 1][j - nums[i]]
可以进行状态压缩
*/
```

[题目](https://leetcode.com/problems/target-sum/)

[分析1](https://www.youtube.com/watch?v=r6Wz4W1TbuI)

[分析2](https://www.youtube.com/watch?v=zks6mN06xdQ&t=1100s)


## LC 496 下一个比它大的值

单调栈

[题目](https://leetcode.com/problems/next-greater-element-i/)


## LC 518 硬币组合得到指定数额

DP

```cpp
/*
dp[i][j] : coins[0 ~ i] make up j
 
dp[i][j] = dp[i- 1][j] + dp[i][j - coins[i]]
*/
```

[题目](https://leetcode.com/problems/coin-change-2/)


## LC 560 子数组和

前缀和

```cpp
/*
sum[i] = nums[0] + ... + nums[i]
sum[i ~ j] =sum[j] - sum[i - 1]
使用哈希表统计前缀和出现的次数
*/
```

[题目](https://leetcode.com/problems/subarray-sum-equals-k/)

[分析](https://www.youtube.com/watch?v=mKXIH9GnhgU)

## LC 647 计算回纹子字符串数目

DP + 回文串

[题目](https://leetcode.com/problems/palindromic-substrings/)


## LC 813 分割数组使子区间平均数最大

DP

```cpp
/*
dp[i][j] : nums[0 ~ j] splite i group

dp[i][j] = max(dp[i - 1][k] + sum(j, k) / (j - k)) k : i - 1 ~ j - 1
*/
```

[题目](https://leetcode.com/problems/palindromic-substrings/)


## LC 1095 山峰数组找目标值

二分

[题目]https://leetcode.com/problems/find-in-mountain-array/)

[分析](https://www.bilibili.com/video/BV1m5411V7x7)


## LC 1143 最长公共子序列

DP

```cpp
/*
dp[i][j] : text[0 ~ i] text[0 ~j] LCS

if text[i] == text[j] then dp[i][j] = dp[i - 1][j - 1] + 1
else dp[i][j] = max(dp[i - 1][j - 1], dp[i - 1][j], dp[i][j - 1])
*/
```

[题目](https://leetcode.com/problems/longest-common-subsequence/)


## LC 1218 定差最长子序列

DP + hash

```cpp
/*
unordered_map<int, int> dp
*/
```

[题目](https://leetcode.com/problems/longest-arithmetic-subsequence-of-given-difference/)

[分析](https://www.youtube.com/watch?v=1THU4Aa9akQ)


## LC 1278 做最少操作使字符有指定个回纹子字符串

DP

```cpp
/*
dp[i][j] : s[0 ~ i - 1] splite j group

dp[i][j] = min(dp[k][j - 1] + cast(k, i - 1)) j - 2 < k < i - 2
*/
```

[题目](https://leetcode.com/problems/palindrome-partitioning-iii/)

[分析](https://www.youtube.com/watch?v=kD6ShM6jr3g)


## LC 1335 工作安排

DP

```cpp
/*
dp[i][j] : jd[0 ~ i + 1] splite j days

dp[i][j] = min(dp[k][j - 1] + max(jd[k ~ i - 1]))
*/
```

[题目](https://leetcode.com/problems/minimum-difficulty-of-a-job-schedule/)

[分析](https://www.youtube.com/watch?v=eRBpfoWujQM&t=223s)

## LC 1349 分配考场座位

DP + 位运算

```cpp
/*
利用行状态做 DP
dp[i][s]：第 i 行状态为 s 时的最多人数
dp[i + 1][s] = max(dp[i][t] + row(i + 1, t))

取一个数的二进制子集
generate a subset of v
x = v
while x:
    x = (x - 1) & v
*/
```

[题目](https://leetcode.com/problems/maximum-students-taking-exam/)

[分析](https://www.youtube.com/watch?v=QJvCYx1eGxE)

## LC 1416 重新划分字符串

DP + 递归

```cpp
/*
递归形式：
int helper(string s) {
    if(s.empty() == 0) return 1;
    if(s[0] == '0') return 0;
    int ans = 0;
    for(int i = 1; i < s.length(); ++i) {
        int num = stoi(s.substr(0, i));
        if(num > K) break;
        ans += (ans + helper(s.substr(i))) % kmod;
    }
    return ans;
}
使用 hash 记忆化后仍超时

dp[i] : s[i ~ n] cnt
dp[i] = sum(dp[j]) j : i ~ n
*/
```

[题目](https://leetcode.com/problems/restore-the-array//)

[分析](https://www.youtube.com/watch?v=mdUTRI2FMtU)


## LC 1425 指定范围内的最大子序列和

DP + 滑动窗口 + BST + 单调队列

[题目](https://leetcode.com/problems/constrained-subsequence-sum/)

[分析](https://www.youtube.com/watch?v=B5fa989qz4U&t=8s)


## LC 1438 元素差小于指定值的子数组

滑动窗口 + 单调队列 + BST

[题目](https://leetcode.com/problems/longest-continuous-subarray-with-absolute-diff-less-than-or-equal-to-limit/)

[分析](https://www.youtube.com/watch?v=p8-f0_CwWLk&t=687s)

## LC 1441 涂装方式

DP

```cpp
/*
利用行的状态做 DP
一行格子涂装总共有 27 中方式
dp[i][p] 表示第 i 行用 p 这种方式涂装的次数
dp[i + 1][q] = sum(dp[i][p]) 其中 q 与 p 不冲突
*/
```

[题目](https://leetcode.com/problems/number-of-ways-to-paint-n-3-grid/)

[分析](https://www.youtube.com/watch?v=LwD9UIDIvHE)

## LC 1449 组合最大数

DP

[题目](https://leetcode.com/problems/form-largest-integer-with-digits-that-add-up-to-target/submissions/)

[分析](https://www.youtube.com/watch?v=J43N1o1XhqE)

## LC 1589 区间次数统计

区间统计

```cpp
/*
使用差分数组统计区间次数，利用差分数组统计频率变化导数

diff[i] 表示第 i 个元素的查询频次与第 i - 1 个元素查询频次差值

对于 [start, end]，diff[start] += 1，diff[end + 1] -= 1;

对于元素频率 f[i] = f[i - 1] + diff[i]
*/
```

[题目](https://leetcode.com/problems/maximum-sum-obtained-of-any-permutation/)

[分析](https://www.youtube.com/watch?v=9BJkTC3iTGs)


## LC 1643 可达最远楼梯

延迟决策

[题目](https://leetcode.com/problems/furthest-building-you-can-reach/)

[分析](https://www.youtube.com/watch?v=FowBaF5hYcY)

## LC 1653 字符串平衡

DP

```cpp
/*
dp[i][j]: 0 ~ i 的子字符串平衡的最少操作，j == 0 表示以 a 结尾，j == 1 表示以 b 结尾
*/
```

[题目](https://leetcode.com/problems/minimum-deletions-to-make-string-balanced/)

## LC 1665 分配问题

Bitmask + subsets + DP

```cpp
/*
dp[mask][i]: 选择客户为 mask 时，能否用前 i 组数满足

dp[mask][i] = any(
    sums(subset) <= freq[i] and dp[mask^subset][i - 1]
)
*/
```

[题目](https://leetcode.com/problems/distribute-repeating-integers/)

[分析](https://www.bilibili.com/video/BV1qt4y1a7Lm)


## LC 1696 跳跃游戏

DP + 单调队列

[题目](https://leetcode.com/problems/jump-game-vi/)