# Some Usefull Theorems

##

If $U, V$ are two linear subspaces, then $\dim(U + V) = \dim(U) + \dim(V) - \dim(U \bigcap V)$.

This can be proved by considering the basis of $U \bigcap V$ first and then extends the basis to $U$ and $V$ respectively.

##

The intersection of affine subspace $U$ of dimension $m < n$ and hyperplane $H$ (affine subspace of dimension $n - 1$) has three cases:

1. They don't intersect.
2. $H$ contains $U$. The dimension of intersection is $m$.
3. The dimension of intersection is $m - 1$.

This can be proved in this way: 

Suppose $U$ and $H$ intersects, then we can assume they are not *affine* but linear instead (by shifting the center point to the intersection set).

Then $\dim (U + H) = n = \dim(U) + \dim(H) - \dim(U \bigcap H) = m + n - 1 - \dim(U \bigcap H)$. 


##

