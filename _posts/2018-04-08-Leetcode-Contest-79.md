---
layout: post
title: "LeetCode Contest 79"
categories: LeetCode题解
tags: LeetCode Breadth-First-Search
---
LeetCode Contest 79 最后一题解法详解
<!-- more -->
#### **Bus Routes**

```
We have a list of bus routes. Each routes[i] is a bus route that the i-th bus repeats
forever. For example if routes[0] = [1, 5, 7], this means that the first bus
(0-th indexed) travels in the sequence 1->5->7->1->5->7->1->... forever.

We start at bus stop S (initially not on a bus), and we want to go to bus stop T.
Travelling by buses only, what is the least number of buses we must take to reach
our destination? Return -1 if it is not possible.

Example:
Input:
routes = [[1, 2, 7], [3, 6, 7]]
S = 1
T = 6
Output: 2
Explanation:
The best strategy is take the first bus to the bus stop 7,
then take the second bus to the bus stop 6.

Note:
    1 <= routes.length <= 500.
    1 <= routes[i].length <= 500.
    0 <= routes[i][j] < 10 ^ 6.
```
每一条路线经过一系列站点,求解起点 S 到终点 T 的最少换乘次数的路线.
很明显转化为图,每一条路线是一个点. 换乘必须在换乘车站,也就是线路的交点,判断路线与路线之间是
否可达就在于是否存在公共站点,在图中我们表示为点与点之间的边.
判断两条路线是否相交,可以用两个集合求交集或者排序后用 two-pointers 的方法来遍历求解.

那么求解思路就是:
    0. 预处理,将输入的路线信息排序,每条路线经过的站点从小到大排序
    1. 根据路线信息构建交通图, 存储的是和每条路线直接相连的其他路线
    2. 进行 BFS 遍历求解.

代码如下:
{% highlight C++ %}
class Solution {
public:
    int numBusesToDestination(vector<vector<int>>& routes, int S, int T) {
        if (S == T) return 0;
        int N = routes.size();

        unordered_map<int, unordered_set<int>> graph;
        for (int i = 0; i < N; ++i) {
            sort(routes[i].begin(), routes[i].end());
        }
        unordered_set<int> seen, targets;
        queue<pair<int,int>> que;

        // Build the graph.  Two buses are connected if
        // they share at least one bus stop.
        for (int i = 0; i < N; ++i) {
            for (int j = i+1; j < N; ++j) {
                if (intersect(routes[i], routes[j])) {
                    graph[i].insert(j);
                    graph[j].insert(i);
                }
            }
        }

        // Initialize seen, queue, targets.
        // seen represents whether a node has ever been enqueued to queue.
        // queue handles our breadth first search.
        // targets is the set of goal states we have.
        for (int i = 0; i < N; ++i) {
            if (binary_search(routes[i].begin(), routes[i].end(), S)) {
                seen.insert(i);
                que.push(make_pair(i, 0));
            }
            if (binary_search(routes[i].begin(), routes[i].end(), T))
                targets.insert(i);
        }

        while (!que.empty()) {
            auto info = que.front();
            que.pop();
            int node = info.first, depth = info.second;
            if (targets.count(node) > 0) return depth+1;
            for (int nei: graph[node]) {
                if (seen.count(nei) == 0) {
                    seen.insert(nei);
                    que.push(make_pair(nei, depth+1));
                }
            }
        }

        return -1;
    }

    bool intersect(const vector<int>& A, const vector<int>& B) {
        int i = 0, j = 0;
        while (i < A.size() && j < B.size()) {
            if (A[i] == B[j]) return true;
            if (A[i] < B[j]) i++;
            else j++;
        }
        return false;
    }
};

{% endhighlight %}
