# Eigenvalue and Characteristic Polynomial

A very known method to solve linear ODE is to solve the characteristic equation and we have those $e^{r_ix}$ compoenents where $r_i$ is root of the equation.

We also recall that we have similar methods for solving recursive sequences and matrix eigenvalues. So is it just a coincidence or there are something related with all these methods?

Consider a recursive sequence defined as:

$$
a_n = c_1 a_{n-1} + \cdots + c_{n-1} a_1 + c_n
$$

Then this can be represented as $\vec{a_k} = A \vec{a_{k-1}}$ where $\vec{a_k} = \{a_k, a_{k-1} \cdots a_{k-n+1}, 1\}$.
$
This matrix can be diagonalized (why?) as: $A = P^{-1} D P$ and then let $\vec{z} = P\vec{a}$.

Now we have $\vec{z_k} = D \vec{z_{k-1}}$. Notice that this simply means we have $z_{k,1} = d_1 z_{k-1, 1}$ and it's trivial to solve. Then we transfer back to $\vec{a}$.

Using roughly the same reasoning we can solve ODEs. We know that if $y' = ky$ then the general solution is $e^{kx}$, so we can similarly perform a linear transformation and get the trivial solution then transform it back to original problem. So this gives us the intuition that the general solution must be some linear combination of exponential functions.