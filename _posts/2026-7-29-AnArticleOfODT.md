---
title: 珂朵莉树学习笔记
tags: 
    - OI
    - 学习笔记
mathjax: true
---

珂朵莉树（Chtholly Tree），又名老司机树（Old Driver Tree，简称 ODT），是一种能够维护区间推平（颜色段均摊）的数据结构，名字来源于[CF896C](https://codeforces.com/problemset/problem/896/C)。

珂朵莉树实质是一种思想，将一段元素相同的区间看做一个元素，用平衡树（`std::map`，`std::set` 等）或链表等数据结构来维护各个代表区间的元素，能很好的处理区间推平（覆盖）。

常见的储存方法是对于每个区间用 $(l, v)$ 来描述设当前元素的下一个元素的 $(x + 1)_l$ 为 $r$，那么这个元素代表的区间就是 $[l, r)$。

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
//注意到以 assign 为函数名可能与 c++ 中自带的函数冲突，所以建议使用其他函数名
void assign(int l, int r, int v) {
	auto itr = split(r + 1), itl = split(l);//分裂区间，使 [l, r] 在 std::map 上有完全对应的区间，这里传入 r + 1 是因为我们需要访问分裂后返回的前一个区间，保证它的右端点为 r
  //此处 itl 与 itr 的分裂顺序似乎会对迭代器有影响，详情见 oi-wiki
	while (itl != itr)
		itl = m.erase(itl);//删除
	m[l] = v;//推平成新的值
}
```
