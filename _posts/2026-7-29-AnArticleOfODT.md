---
title: 珂朵莉树学习笔记
tags: 
    - OI
    - 学习笔记
mathjax: true
---

# 介绍

珂朵莉树（Chtholly Tree），又名老司机树（Old Driver Tree，简称 ODT），是一种能够维护区间推平（颜色段均摊）的数据结构，名字来源于[CF896C](https://codeforces.com/problemset/problem/896/C)。

珂朵莉树实质是一种思想，将一段元素相同的区间看做一个元素，用平衡树（`std::map`，`std::set` 等）或链表等数据结构来维护各个代表区间的元素，能很好的处理区间推平（覆盖）。

常见的储存方法是对于每个区间用 $(l, v)$ 来描述，对于下标为 $i$ 的元素，其对应的区间为 $[l_x, l_{x + 1})$，储存的值为 $v_i$。

所以为了满足以上条件，需要在首尾添加 $2$ 个哨兵。

```cpp
m[1] = 0;
m[n + 1] = 0;
```

# 主要实现

珂朵莉树有以下几种操作：

## Split（分裂）

传入一个下标 $x$，将包含 $x$ 的区间 $[l, r]$ 分裂为 $[l, x)$ 和 $[x, r]$，以便于我们之后推平区间。

```
//m 用于实现珂朵莉树的 std::map
map<int, int>::iterator split(int x) {//std::map 的第二关键字（此处为 int）可以换成对应题目需要存储的数据
	auto it = prev(m.upper_bound(x));//找到第一个 l <= x 的区间
	if (it->first == x)//已经不用分裂了
		return it;
	return m.emplace(x, it->second).first;//由于获取 it 这个区间的 r 需要访问它的下一个指针，所以可以直接将 x 加入 std::map 中
}
//此处返回的指针就是 [x, r] 的指针，便于进行操作
```

## assign（推平）

传入区间下标 $l, r$ 与推平的数值 $v$，进行区间推平操作。

```
void assign(int l, int r, int v) {
	auto itr = split(r + 1), itl = split(l);//分裂区间，使 [l, r] 在 std::map 上有完全对应的区间，这里传入 r + 1 是因为我们需要访问分裂后返回的前一个区间，保证它的右端点为 r
  //此处 itl 与 itr 的分裂顺序似乎会对迭代器有影响，详情见 oi-wiki
	while (itl != itr)
		itl = m.erase(itl);//删除
	m[l] = v;//推平成新的值
}
```


查询函数与推平函数类似，需要根据题目实现。

# 例题

## 洛谷 P2082 区间推平（加强版）

[原题面](https://www.luogu.com.cn/problem/P2082)

这是一个板子题，观察 $s_i, t_i$ 的范围较大，不支持我们进行差分，而 $N \le 10^5$，可以考虑使用珂朵莉树实现。

具体做法就是每读入一个区间就推平，最后对珂朵莉树中的所有元素统计一遍。

代码：

```cpp
#include <bits/stdc++.h>
using namespace std;
#define int long long
const int N = 1e17;

int n;
map<int, int> m;
map<int, int>::iterator split(int x) {
	auto it = prev(m.upper_bound(x));
	if (it->first == x)
		return it;
	return m.emplace(x, it->second).first;
}
void ass(int l, int r) {
	auto itl = split(l), itr = split(r + 1);
	m.erase(itl, itr);
	m[l] = 1;
}
signed main() {
	ios::sync_with_stdio(false);
	cin.tie(0);
	cout.tie(0);
	
	cin >> n;
	m[1] = 0; m[N + 1] = 0;
	for (int i = 1; i <= n; i++) {
		int s, t;
		cin >> s >> t;
		ass(s, t);
	}
	int ans = 0;
	auto itl = m.begin();
	for (auto it = ++m.begin(); it != m.end(); it++) {
		if (itl->second)
			ans += it->first - itl->first;
		itl++;
	}
	cout << ans;
	return 0;
}
```
