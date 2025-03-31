# Complex Analysis

##

A function $f(z)$ is analytic in a domain $D$ if $f(z)$ is single valued and has a finite derivative $f'(z)$ for every $z \in D$. 

In fact analyticity is defined as a function that can be represented as convergent power series.

This is equivalent to holystic function, which is defined as differentiable in any neighbour within a certain domain.

Indeed, having a convergent power series is a quite strong condition. Even though we know Taylor series can always be a good estimation up to an error term $O(x^n)$, Taylor series is not always convergent, even if it's convergent, it might not convergent to the function itself.

Take $f(x) = e^{\frac{1}{-x^2}}$ as an example, and we additionally let $f(0) = 0$. Then it's easy to check $f^{(n)}(0)$ are always $0$. So Taylor series is always zero no matter how many terms are used, which means it's convergent but obviously not convergent to the function itself.
