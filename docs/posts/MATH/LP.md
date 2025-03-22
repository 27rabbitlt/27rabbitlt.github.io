# Linear Programming and Optimization

## 1 Intro to LP

### 1.1 Definitions

Linear programming asks to optimize a linear objective with constraints of inequalities and equalities. It captures the essence of a variety of practical problems. It can also serve as a different point of view to some combinatorial problems.

In general a LP is in the form of:

$$
\begin{align*}
\max / \min \; c^T  &x \\
A&x \le e \\
B&x \ge f \\
C&x = g
\end{align*}
$$

It's worth noting that all the constraints only involve **closed** constraints, i.e. no inequalities like: $Ax < e$.

To simplify it, we propose a canonical form: 

$$
\begin{align*}
\max \; c^T &x \\
A &x \le b \\
&x \ge 0
\end{align*}
$$

It's not hard to transform a general LP into canonical one. The only tricky part is that the canonical part requires non-negative solution, which can be achieved by replacing a un-constrainted $x$ with $x^+ - x^-$.

The advantage of non-negativitiy is that we can guarantee the feasible area cannot contain a straight line, which is a useful property in the future analysis.

### 1.2 Different Types

We conclude LPs into three types:

1. Finite Optimum
2. Unbounded
3. Infeasible

The definition is implied in the name.

An LP solver should return:

1. A certificate of optimality together with the optimal value. In Simplex Method we will finally come up with a formula $\text{opt} = c^Tx + d$ where $c \le 0$.
2. A certificate of unbound-ness, i.e. a half-line, basically a vector/direction $v$ with a starting point $y$, such that $y$ is feasible and $y + \lambda v$ are feasible for all $\lambda$, $c^T v \gt 0$. Notice that in canonical form $y + \lambda v$ are all feasible is equivalent to say $Av \le 0$.
3. A certificate of infeasibility. This might seem a bit obscure: how could one give a *proof* or *certificate* for infeasibility? By making use of duality, we can show that it's enough to give a certificate of unbound-ness of the dual problem.

### 1.3 Example: Shortest Path

A classical example of LP is s-t shortest path problem. It can be modeled in this way: 

$$
\begin{align*}
\max \; &d_t \\
&d_s = 0 \\
&d_v \le d_u + w_{(u, v)} &\forall \; \text{edge from} \; u \; \text{to} \; v
\end{align*}
$$

It's not straightforward to understand why we are trying to maximize the objective while we are looking for a **shortest** path.

## 2 Polyhedra and Convex Geometry

LP is optimizing on a polyhedron so the study of polyhedra and convex geomotry enables us to have a geometrical view of the problem which is usually insightful and intuitional.

### 2.1 Definitions

!!! Note "Half-space & Hyperplane"
    A hyperplane is a set of the form $\left\{x | \alpha^T x = \beta \right\}$ for $\alpha \in \mathbb{R}^n \setminus \{0\}$ and $\beta \in \mathbb{R}^n$.
    A half-space is a set of the form $\left\{x | \alpha^T x \le \beta \right\}$ for $\alpha \in \mathbb{R}^n \setminus \{0\}$ and $\beta \in \mathbb{R}^n$. 
    A polyhedron is a finite intersection of half-spaces. Moreover, a bounded polyhedron is called a polytope.

Some more defs:

The dimension of a polyhedron $P$ is the dimension of a smallest dimensional affine subspace containing $P$.

A supporting hyperplane for polyhedron $P$ is a hyperplane s.t. $P \bigcap H \neq \emptyset$ and $P$ is contained in one of the half-space defined by the hyperplane $H$, i.e. either $H^+$ or $H^-$.

Important def:

Let $P$ be a non-empty polyhedron.

1. A *face* is either P itself or intersection with a supporting hyperplane.
2. A *vertex* is a 0-dimensional face.
3. An *edge* is a 1-dimensional face.
4. An *facet* is a $(\dim(P)-1)$-dimensional face.

Some very important propositions:

Let $P = \left\{ x \in \mathbb{R}^n: Ax \le b \right\}$ be a polyhedron with $A \in \mathbb{R}^{m \times n}$, let $F \subset P$, then these statements are equivalent:
1. $F$ is a face of $P$.
2.  $\exists c \in \mathbb{R}^n$ s.t. $\delta = \max \left\{ c^Tx: x \in P \right\}$ is finite and $F = \left\{ x \in P: c^Tx = \delta \right\}$,
3. $F = \left\{x \in P: \bar{A}x = \bar{b} \right\}$ where $\bar{A}$ is an inequality subsystem.

Always try to prove it before proceeding since the proof involves some basic methods showed up frequently in other problems.

Important proposition:

Let $P = \left\{ x \in \mathbb{R}^n: Ax \le B \right\}$ be a polyhedron and $y \in P$. Then the following statements are equivalent:

1. $y$ is a vertex of $P$.
2. $y$ is the unique solution to $\bar{A}x = \bar{b}$, where $\bar{A}x \le \bar{b}$ is the subsystem tight on $y$.
3. $y$ is an extreme point.

From 1 to 3:

Since $y$ is a vertex, by previous proposition there exists $c$ s.t. $y$ is the only point that maximizes $c^T x$. Assume $y$ is not extreme point then there exists $y + \epsilon \in P$ and $y - \epsilon \in P$. Then $c^T y = (c^T (y + \epsilon) + c^T(y - \epsilon)) / 2$.  Either one is at least as large as $y$.

From 3 to 2:

Assume $y$ is not the unique solution, which means there exists $v$ s.t. $\bar{A} v = 0$.

Then consider $y \pm \epsilon v$ with $\epsilon$ small enough. We have $\bar{A} (y \pm \epsilon v) = b$.

Since $\bar{A}$ is the tight subsystem, with $\epsilon$ small enough the un-tight inequality will not be broken.

In summary, $y \pm \epsilon v \in P$, which contradicts with $y$ being extreme point.

From 2 to 1:

We immediately have $\left\{ y \right\} = \left\{ x \in P: \bar{A}x = \bar{b} \right\}$, by previous proposition $\left\{ y \right\}$ is a face. It's obvious 0-dimension so it's a vertex.

