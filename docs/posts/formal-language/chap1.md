# Simply Typed Lambda Calculus

## Operational Semantics

To reason about the program we need semantics. The semantics describe how runtime terms are evaluated. 
We have *big-step, structural* and *contextual semantics* which are equivalent but expressing how terms are evaluated in distinct ways.

We already have beta-reduction then why do we bother to study these semantics? In real programs, we have lots of operations that are difficult or impossible to 
desribe purely based on beta-reduction. Also semantics can restrict the order of evaluation.

### Big-step Semantics

In this natural semantics, $e \downarrow v$ defines when a term $e$ evaluates to a value $v$.

### Structural Semantics 

Here we define a step-by-step process whereby the term is slowly reduced further and further. 
They are needed when considering infinite executions or to model concurrency where the individual steps of multiple computations can interleave.

We define a structural *call-by-value, weak, right-to-left* small-step operational semantics on our runtime terms: $e \succ e'$ means that $e$ steps to $e'$.
This is commonly used.

Call-by-value means we firstly evaluate the argument before we substitute. 

Weak means we do not allow reduction below $\lambda$ abstractions. For example, in a strong semantics the reduce $\lambda x. (\lambda y.y) x$ could be further reduced into $\lambda x.x$. 
We donot further simplify the expressions inside a $\lambda$ abstraction.

Right-to-left means a term like $e_1 + e_2$ would be evaluated firstly into $e_1 + e_2'$ before we begin evaluating $e_1$.

### Contextual Semantics

The problem of small step semantics are that it's very tedious to prove something since we have to discuss all cases for all possible operations even though the idea is exactly the same except that
we are using a different operation.

So in contextual semantics operations are generalised by a *hole*.


