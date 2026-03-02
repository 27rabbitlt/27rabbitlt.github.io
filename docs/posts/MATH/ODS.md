# Optimization for Data Science

## Theory of Convex Optimization

### First-order Characterization of Convexity

We have a lemma to prove a function is convex using first-order gradient.
Suppose $D(f)$ is open and that $f$ is  differentiable; in particular the gradient

$$
\nabla f(x) = \left( \frac{\partial f}{\partial x_1}, \dots, \frac{\partial f}{\partial x_d} \right)
$$

exists at every point $x \in D(f)$.

Then $f$ is convex iff $D(f)$ is convex and 

$$
f(y) \ge f(x) + \nabla f(x)^T (y - x)
$$

holds for all $x, y \in D(F)$.
