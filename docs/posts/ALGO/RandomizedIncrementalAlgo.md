# Randomized Inceremental Algorithm

随机增量法在学竞赛的时候听学长讲过，可惜当时根本没听懂。只知道它很神奇，看起来三层循环但实际上期望复杂度线性；看起来道理很简单，但却能解决感觉上很复杂的问题。

在竞赛中此类算法只考察过最小圆覆盖问题。实际上还有一些其他问题也可以用类似的思想解决（refer to https://www.cs.rpi.edu/~cutler/classes/computationalgeometry/F23/lectures/07_randomized_incremental_construction.pdf）。

我们在这里先介绍最小圆覆盖，后面再施工其他的算法。

!!! note "最小圆覆盖问题"
    给定平面上 $n$ 个点，求半径最小的圆使得所有的点都在圆上或圆内。

这个问题是比较特殊的选址问题（考虑 $R^2$ 上的欧氏距离的只有一个设施的选址问题），可以很自然的推广到更高维的最小*球*覆盖。

我们在竞赛中用到的算法是 Welzl 算法 （https://www.cs.rpi.edu/~cutler/classes/computationalgeometry/F23/lectures/07_randomized_incremental_construction.pdf）原文写的非常好，有兴趣的话可以看看。同时这个问题本身也有确定性线性时间的算法，但是常数很大（refer to https://en.wikipedia.org/wiki/Smallest-circle_problem）。两个算法都有来自线性规划的 intuition。

首先我们需要两个看起来很显然的引理：

!!! note "引理"
    包含三个不共线的点的最小半径的圆是由这三个点外接所确定的圆。

用扰动的方法很简单即可证明。

!!! note "引理"
    最小覆盖圆有且仅有一个。

存在最小覆盖圆是显然的，唯一性可以通过反证法证明。如果有两个最小半径覆盖圆，那么点集一定都在两个圆的交集中。明显可以构造一个半径更小的圆覆盖这个交集，与最小半径的假设矛盾。

而后我们介绍算法流程。

算法本身极度简单，用递归来描述简洁明了：

```
algorithm Welzl
    input: Finite sets P and R of points in the plane |R| ≤ 3.
    output: Minimal disk enclosing P with R on the boundary.

    if P is empty or |R| = 3 then
        return trivial(R)
    choose p in P (randomly and uniformly)
    D := welzl(P − {p}, R)
    if p is in D then
        return D

    return welzl(P − {p}, R ∪ {p})
```

伪代码中 `P` 代表需要被覆盖的点集。`R` 代表额外的约束，我们要求 `R` 中的元素必须在圆的边界上。所以算法求的是所有覆盖 `P` 且边界包含 `R` 的圆中半径最小的那一个。

算法思路是：

给定输入 $P$ 为需要被覆盖的点集，$R$ 为已知必须在圆上的点构成的点集。每次随机挑选一个点 $p$ 排除在外，递归的求取剩下的 $n-1$ 个的点的最小圆覆盖，记为 $md(P \setminus \{p\})$。

+ 如果被排除的点 $p$ 已经包含在 $md(P \setminus \{p\})$，那么直接返回 $md(P \setminus \{p\})$；
+ 如果不包含，说明 $p$ 一定在 $md(P)$ 的圆周上，那么我们令 $R = R \cup \{p\}$. 然后递归执行 $md(P \setminus \{p\}, R)$.

如果 $R$ 已经包含三个元素，那么圆可以被直接确定。

然后我们分析期望时间复杂度。

\begin{align}
E(T(n, j)) &= 1 + E(T(n - 1, j)) + \text{Pr} \left\{ p \notin md(P \setminus \{p\}, R) \right\} \cdot E(T(n - 1, j + 1)) \\
E(T(n, 3)) &= 0
\end{align}

考虑每个点都是随机选择的，$n$ 个点中只需要 $3$ 个点就可以确定最小覆盖圆，而我们考虑剩下的 $n - 3$ 个点，他们如果是被随机去掉的那个点，一定会被包含在剩下的点的最小覆盖圆里，所以 $\text{Pr}\left\{p \notin md(P \setminus \{p\}, R)\right\} \le \frac{3}{n}$。所以简单数学归纳法即可证明 $E(T(n, j)) = O(n)$

