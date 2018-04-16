---
layout: post
title: "LeetCode Contest 80"
categories: LeetCode题解
tags: LeetCode Dynamic-Programming Breadth-First-Search
---

LeetCode Contest 80 最后一题解法详解
<!-- more -->

#### **Race Car**


>Your car starts at position 0 and speed +1 on an infinite number line.  
(Your car can go into negative positions.)
Your car drives automatically according to a sequence of instructions A (accelerate) and R (reverse).
When you get an instruction "A", your car does the following: position += speed, speed *= 2.
When you get an instruction "R", your car does the following: if your speed is positive then speed = -1 , otherwise speed = 1.  (Your position stays the same.)
For example, after commands "AAR", your car goes to positions 0->1->3->3, and your speed goes to 1->2->4->-1.
Now for some target position, say the length of the shortest sequence of instructions to get there.

>Example 1:
Input:
target = 3
Output: 2
Explanation:
The shortest instruction sequence is "AA".
Your position goes from 0->1->3.

>Example 2:
Input:
target = 6
Output: 5
Explanation:
The shortest instruction sequence is "AAARA".
Your position goes from 0->1->3->7->7->6.

>Note:
    1 <= target <= 10000.


题意就是给定一个目标点,起始位置为0,起始初速度为1. 有两种移动方式:
- 'A'代表按当前方向继续前进,每移动一次,位置增加当前速度值,速度翻倍
- 'R'代表掉头,位置保持不变,前进方向改变为相反方向,速度恢复为初始速度,即为1
求解到达目标位置的最短移动路径.

观察可以发现,如果一直'A'向前, 移动距离为 `2^k-1`, k 为当前方向移动的次数, 'R'不改变位置, 所以
任意的 target 都可以最终表示为 `A^k1 - A^k2 + A^k3 - ....` 这样的形式.
起始位置为0, target >= 1,所以肯定先一直向前行驶,最开始调头毫无意义.
简单的一个情况是: 如果 target = 2^k-1, 则解为 k, 向前移动 k 次即可.
由此,我们可以发现, `2^(k-1) <= target < 2^k`.
所以可以很容易发现 `k1 > k3 > k5 > ...`, `k2 > k4 > ...`.
在需要选择调头操作时,有两种操作, 第一种我们已经驶过了 target 点,所以需要调头往回开; 第二种尚未驶过 target 点,但是继续前行会驶过 target 点,所以原地调头,往回开一段距离再掉头继续前行.


**动态规划**   
运用动态规划来解比较容易想到. 每一次掉头之后, 速度变为1, 可以理解为再次从0出发,初速度为1进行行驶,这样就找到了重复的子问题. 所以我们可以用 dp[pos] 记录下到达 pos 点的最短移动路径长度.
状态转移方程可以表示为:
`2^(k-1) <= target < 2^k`
不行驶过 target 点:
`dp[target] = min(dp[target], dp[target - 2^(k-1) + 2^j] + k-1 + j + 2)`( 0 <= j < k-1)
行驶过 target 点:
`dp[target] = min(dp[target], dp[2^k-1 - target] + k + 1)`

代码如下:
{% highlight C++ %}
int racecar(int target) {
    vector<int> dp = vector<int>(target + 1, INT_MAX);
    dp[0] = 0; dp[1] = 1; dp[2] = 4;
    for (int t = 3; t <= target; ++t) {
        int k = 1, m = t;
        while(m/2 > 0) { m /= 2; ++k; }
        if (t == (1<<k) - 1) {
            dp[t] = k;
            continue;
        }
        for (int j = 0; j < k-1; ++j)
            dp[t] = min(dp[t], dp[t - (1<<(k-1)) + (1<<j)] + k-1 + j + 2);
        if ((1<<k) - 1 - t < t)
            dp[t] = min(dp[t], dp[(1<<k) - 1 - t] + k + 1);
    }
    return dp[target];
}
{% endhighlight %}

分析下时间复杂度, 每一个 T 点耗时 log(T), 所以总耗时 Tlog(T)

**BFS**   
按照分析, BFS 同样适用,最快包含 target 的层即为我们的最短路径.
{% highlight C++ %}
int racecar(int target) {
    unordered_set<string> visited;
    vis.insert("0,1");
    queue<pair<int, int>> que;
    que.push({0, 1});
    int depth = 0;
    while (!que.empty()) {
        int num = que.size();
        while(num > 0){
            auto cur = que.front();
            que.pop();
            --num;
            if (cur.first == target) {
                return depth;
            }
            string s1 = to_string(cur.first) + "," + to_string(cur.second*2);
            string s2 = to_string(cur.first) + "," + to_string(cur.second > 0 ? -1 : 1);
            if (visited.find(s1) == visited.end() && cur.first + cur.second < 2 * target) {
                visited.insert(s1);
                que.push({cur.first+cur.second, cur.second*2});
            }
            if (visited.find(s2) == visited.end() && cur.first >= target /2) {
                visited.insert(s2);
                que.push({cur.first, cur.second > 0 ? -1 : 1});
            }
        }
        depth++;
    }
    return -1;
}
{% endhighlight %}
