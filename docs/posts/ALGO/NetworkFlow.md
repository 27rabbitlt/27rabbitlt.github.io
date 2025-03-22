# Network Flow

Suppose at first the graph has only intergral capacity.

The Ford-Folkerson algorithm runs in this intuitive way:

For currenet $f-$flow (if it's the very first step, then the flow is just empty), we try to augment it by finding a path from $s$ to $t$ in residual graph.

If we can find such a path, then we *add* this path to the current $f-$flow and we get a new flow with at least 1 more value.

And the other direction is always true: if we can't find an augmentation path, then it means $s$ and $t$ are already seperated into two connected component in residual graph, thus we can get a $s-t$ cut $C = \text{component}(s)$.

Now we want to prove this cut has the same value of the flow.

We know that the value of a flow:

$$
\nu(f) = f(\delta^+(C)) - f(\delta^-(C))
$$

While since $C$ is a connected component in residual graph, so $u_r(\delta^+(C)) = 0$, which means $f(\delta^+(C)) = u(\delta^+(C))$.

Again, for a similar reason, $u_r(\delta^-(C)^R) = 0$, which means $f(\delta^-(C)) = 0$.

So in conclusion,

\begin{align*}
\nu(f) &= f(\delta^+(C)) - f(\delta^-(C)) \\
       &= u(\delta^+(C)) - 0 \\
       &= u(\delta^+(C)) \\
       &= \text{cut}(C)
\end{align*}

This indeed proves the Ford-Folkerson algorithm's correctness.

If the algorithm terminates, then it finds a maximum flow; if it didn't terminate yet, then we can at least improve the flow by 1.

So the time complexity bound of this algorithm is $O(|E| f)$ where $|E|$ is the number of edges and $f$ is the value of maximum flow.

A variation of FF algorithm is the Edmonds-Karp algorithm with time complexity bound $O(|V||E|^2)$. Another variation is Dinic/Dinitz algorithm, which is published earlier with time complexity of $O(|V|^2|E|)$.

The idea of EK algorithm is also simple. We still implement the FF algorithm but each time we pick the shortest augment path (short in the sense of number of edges). We can always find such path using BFS.

Denote the edges in BFS tree (a tree formed by the edges used in BFS procedure) as $E_L$.

It's obvious that each iteration of augmenting cost $O(|E|)$ time so now we need to determine the upper bound of the number of iterations.

We denote the edges in the BFS tree of $G_f$ to be $E_L$.

We notice that using BFS to find augment path, we always have at least one bottleneck edge that will get saturated. This edge will be removed and the reverse will be added to the graph.

Say edge $e = (u, v)$ is a bottleneck edge with $u \in L_i, v \in L_{i+1}$ where $L_i = \{x | \text{dist}(s, x) = i\}$. Then removing $e$ and adding $e^R$ will never decrease the distance between $s$ and $t$; and the level of $u$ increase by at least 2 or the edge $e$ is completely removed from the BFS tree, the level of $v$ doesn't decrease. If an edge is removed from the BFS tree, it cannot be added back (why? think about this).

This means an edge can at most appear $\frac{n}{2}$ times because the BFS tree is at most $n$ levels deep. We have $2|E|$ edges including the reversed ones, so the number of iterations would be $O(|V||E|), leading a total runtime of $O(|V||E|^2)$.

Another variation Dinic algorithm also implements FF algorithm but in yet another way finding a new augment path.

It's indeed not only a path but actually a *blocking flow*.

In Dinic algorithm, each iteration we find a blocking flow and add it to current flow.

A blocking flow is definied in this way:

!!! Definition
    A blocking flow in $G_f$ is a flow $f'$ such that every shortest path (short in the sense of number of edges) consists of at least one saturated edge in $f'$. Or in other words, there isn't a path from $s$ to $t$ only using the edges $E_L \setminus \{e \;|\; f'(e) = u_f(e)\}$.

Blocking flow isn't necessarily a max flow.

We firstly prove in this way we only need $O(n)$ iterations.

Like how we reason in EK algorithm, we would like to show each iteraion the distance in $G_L = (V, E_L)$ between $s$ and $t$ increase at least by one.

Note that every shortest path between $s$ and $t$ contains a saturated edge (because of the definition of blocking flow). So suppose before the iteration the flow is $f$, and during iteration we find the blocking flow $f'$, and the graph $G_L$ is constructed under $G_f$ and $G'_L$ is constructed under $G_{f + f'}$.

Now if we have a shortest path $p'$ from $s$ to $t$ in $G'_L$ with the same distance as the shorteset path $p$ from $s$ to $t$ in $G_L$.

If $p'$ consists of a reversed edge (reversed in $G'_L$ and un-reversed in $G_L$), then there exists a shorter path in $G_L$, leading to contradiction; if $p'$ doesn't contain any reversed edge, then all edges also appear in $G_L$. However by definition of blocking flow, every shortest path contains at least one saturated edge.

So we proved by contradition that the distance on $G_L$ increase by at least 1 after each iteration.

The distance can increase no more than $n$ times since the upper bound of the distance is $n$.

Then it comes to the question: how to find such *blocking flow*?

We can give a simple algorithm which does DFS on $G_L$ repeatedly until no DFS path can be found.

Each time we run DFS from $s$ to find a path to $t$. During DFS, if we notice the subtree of $v$ didn't return a valid path to $t$ then we remove the edges to $v$; and after find such a path, we remove the edges that are saturated.

The pseudocode looks like this:

```C++
f = 0;
H = E_L;

while (true) {
       P = dfs(s, t, H);
       if (P.empty()) {
              return f;
       } else {
              f' = the flow saturates the path P;
              f = f + f';
       }
}

Path dfs(node u, node t, EdgeSet H) {
       for ((u, v) in H) {
              P = dfs(v, t, H);
              if (P.empty()) {
                     H.remove((u, v));
              } else {
                     return (u, v) + P;
              }
       }
       return emptyPath;
}
```

Then what's the runtime bound for one iteration?

Consider that each edge can be only delted for once, so it takes at most $O(m)$ time; each edge can be saturated only once, so the `while()` loop can have at most $m$ iterations; and despite of the time of removing edges in `dfs` (since we already took them into consideration), the runtime of each `dfs()` is $O(n). So the total runtime would be $O(m + nm) = O(nm)$.

Now we are done, we have at most $O(n)$ iterations and each iteration costs $O(nm)$, so the time complexity of Dinic is $O(n^2m)$.

In special cases we can have better bounds for the number of iterations.

If it's a unit capacity graph then we have at most $\min(m^{1/2}, n^{2/3})$ iterations.

We can prove the two upper bounds one by one.

For the first one, consider after $k$ iterations we have a flow $f$ and a residual graph $G_f$. Suppose the maximal flow on $G_f$ is $f'$, which means $f + f'$ is the max flow for graph $G$.

Since it's already after $k$ iterations, the min distance between $s$ and $t$ is at least $k$.

Then the capacities of all edges in $f'$ is at least $\nu(f) \cdot k$ (consider path decomposition). However we have in total $m$ edges so $\nu(f) \cdot k \le m$, thus $\nu(f) \le \frac{m}{k}$. So we can improve the current flow by at most $\frac{m}{k}$.

We know that each iteration increase the flow by at least 1 so the remaining iterations are at most $\frac{m}{k}$.

So the number of iterations is bounded by $k + \frac{m}{k}$ which minimize at $k = \sqrt{m}$.

For the second bound, consider the BFS tree.

There are at least $k$ levels in BFS tree, and $n-1$ vertices in total (ignore the source $s$). Ignore the levels deeper than $k$. (Here $k$ is the distance between $s$ and $t$)

There are more than $k/2$ levels with less than $2n/k$ vertices, otherwise the sum exceeds $n-1$. 

So at least two such levels are adjacent. There are at most $(2n/k)^2$ edges connected between these two levels, so we can form a $s-t$ cut with value at most $(2n/k)^2$. Again, we know that each iteration increase the flow by at least 1 so the remaining iterations are at most $\frac{4n^2}{k^2}$.

Now the number of iterations is bounded by $k + \frac{4n^2}{k^2}$ which minimize at $k=2n^{2/3}$ with min value $O(n^{2/3})$.

And for such graph, the blocking flow can be found in $O(m)$ time.

----

Try to prove the upper bound of iterations for bipartite graph matching flow: $O(\sqrt{n})$.

----

