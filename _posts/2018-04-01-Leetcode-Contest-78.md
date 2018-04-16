---
layout: post
title: "LeetCode Contest 78"
categories: LeetCode题解
tags: LeetCode Dynamic-Programming
---
LeetCode Contest 78 最后两题解法详解
<!-- more -->

#### **Soup Serving**   

```
 There are two types of soup: type A and type B. Initially we have N ml of each type of soup. There are four kinds of operations:
 - Serve 100 ml of soup A and 0 ml of soup B
 - Serve 75 ml of soup A and 25 ml of soup B
 - Serve 50 ml of soup A and 50 ml of soup B
 - Serve 25 ml of soup A and 75 ml of soup B  
When we serve some soup, we give it to someone and we no longer have it.  Each turn, we will choose from the four operations with *equal probability 0.25*. If the remaining volume of soup is not enough to complete the operation, we will serve as much as we can.  We stop once we no longer have some quantity of both types of soup.   
Note that we do not have the operation where all 100 ml's of soup B are used first.  
Return the probability that soup A will be empty first, plus half the probability that A and B become empty at the same time.  
Notes:  
0 <= N <= 10^9.   
Answers within 10^-6 of the true value will be accepted as correct.
```   

题目要求A汤先用光的概率加上一半的A，B汤同时用光的概率。每一次有四种选择，A，B汤初始的量也是相同的，
第一步可以简化问题，基本单位是25ml，所以我们可以把Nml简化为 （N+24)/25 个单位，这样四个选择就变成 （A-4， B），（A-3， B-1），（A-2，B-2），（A-1，B-3）。
接下来就是DP来解了，但是因为N可能很大，所以naive DP肯定无法满足要求。  
if N == 4800, the result = 0.999994994426  
if N == 4801, the result = 0.0.999995382315
需要注意的一点是，因为误差范围小于10^-6就可以，所以当N > 4800 时，概率就无限接近于1了。（证明待加）  
{% highlight C++ %}
double memo[200][200];
double soupServings(int N) {
    return N >= 4800 ? 1.0 : helper((N+24)/25, (N+24)/25);
}
double helper(int a, int b) {
    if(a <= 0 && b <= 0) return 0.5;
    if(a <= 0) return 1;
    if(b <= 0) return 0;
    if(memo[a][b] > 0) return memo[a][b];
    memo[a][b] = 0.25 * ( helper(a-4, b) + helper(a-3, b-1) + helper(a-2, b-2) + helper(a-1, b-3));
    return memo[a][b];
}
{% endhighlight %}   

------


#### **Chalkboard XOR Game**

```
We are given non-negative integers nums[i] which are written on a chalkboard.  Alice and Bob take turns erasing exactly one
number from the chalkboard, with Alice starting first.  If erasing a number causes the bitwise XOR of all the elements of the
chalkboard to become 0, then that player loses.  (Also, we'll say the bitwise XOR of one element is that element itself, and the
bitwise XOR of no elements is 0.)  
Also, if any player starts their turn with the bitwise XOR of all the elements of the chalkboard equal to 0, then that player wins.  

 Return True if and only if Alice wins the game, assuming both players play optimally.   

Notes:  
 1 <= N <= 1000
 0 <= nums[i] <= 2^16
```   



分析： 这道题难点在于仔细分析题意，alice先手拿走一个数（随意选择），如果剩下的数做异或运算后结果为0，那么alice输了。  

也就是说：

+ 在取数之前，如果数组中剩下的所有元素异或运算结果为0，则当前选手获胜。    

+ 如果数组元素个数为偶数，且每个选手取完数后异或操作得到的结果不为0，则因为最后一个数的取数操作不是alice，所以alice能获胜。

考虑元素个数为偶数，start condition是XOR ！= 0，这意味着至少有两个数是不一样的，（如果所有数一样，则异或得到的结果为0）所以alice每一次的取数只要和当前的XOR不一样就能保证取完数不为0，所以最后一定能获胜。

元素个数为奇数，则第二个取数的Bob面对的是偶数个数的数组，所以alice必输。

这其实是一道数学题，需要仔细分析他的逻辑。

代码如下：

{% highlight C++ %}
bool xorGame(vector<int>& nums) {
    int xor = 0;
    for (int i: nums) xor ^= i;
    return xor == 0 || nums.size() % 2 == 0;
}
{% endhighlight %}
