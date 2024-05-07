如果不记笔记好像没有动力复习考试，而且看过一遍都会忘掉，尝试写一点概要。

当我们想要证明数量的下界的时候，例如 $S = \{x_1, \cdots, x_k\}$ 可以考虑：

1. 如果数量太少，一定可以找到一个次数不超过 $d$ 的 *vanishing* 非平凡多项式 $p$，也就是说 $p(x_i) = 0$ 且 $p \neq 0$.
2. 通过 $S$ 的特殊结构推出矛盾，一般可以尝试证明如果 $S$ 中的元素取值都为 $0$，那么推出还有其他更多元素的取值也为 $0$，最终推出多项式为零多项式，或者是引发其他矛盾。

目前这个方法用在两个地方：

+ 有限域 Kakeya Set 不能太小
+ 一条线上的 joint 不能太多

第二个应用并不是直接证明某个数量太少，而是证明相对数量不能太少，也即另一个的数量不能太多。

我们以 joint 数量为例：

!!! note "Joint Number"
    TBD

### Restricted Set Intersection

$L$-interesting 不要求集合自身不在 $L$ 里，但是 $L$ mod $p$-interesting 要求集合自身大小不在 $L$ 里。因为证明过程中不取模的话可以通过排序的方法保证 $f_j(v_j) \neq 0$，而模意义下大小顺序没有意义，取模后大的可能变小。

### 概率构造

有的时候我们不能给出 explicit 构造，但是可以通过概率证明这样的东西是存在的。

+ 考虑一个图的最大割（把图分成 A, B 两个部分，最大化两个部分互相连边的数量），如果我们独立同分布的把每个节点等概率分到 A 或 B，那么每条边在割里的概率也是 1/2，虽然边们并不是独立的，但是求期望不影响，期望是总边数/2，所以至少存在一个分割方案使得割边数量大于 1/2 。

+ 

