# Untyped Lambda Calculus

I still doesn't understand why this is called untyped. From current understanding I guess it basically means there no types but only lambda abstractions. Everything 
is modeled as a lambda function. For example, for natural numbers, we have some expectations for them from our maths background knowledge and daily life.
As long as the definition or the model satisfies the expectation, then it is a valid description of natural numbers.

Surprisingly, the computational power of this model is Turing Complete: $e := x \mid \lambda x.e \mid e_1 e_2$.

Interestingly we have terms that can cause infinite reduction:

$$
\omega := \lambda x.x x \;\; \Omega := \omega \omega
$$

It turns out that $\Omega$ reduces to itself.

## Scott Encodings

We can encode natural numbers by lambdas. We can think of numbers as a function $n$ taking two parameters $z, s$:

```
match n with
    0 => z
    S n => s n
end
```

Here `S` means `Succ`.

Note that this is not the definition of natural numbers in lambda because this *mathcing* mechenism doesn't exist in lambda. This is just the properties that we hope natural numbers can have.
We can come up with such definition in lambda calculus that the property described above is feasible.

We let $0 := \lambda z,s . z$ and inductively $1 := \lambda z,s. s \; 0$, etc.. In general, $n+1 := \lambda z,s . s \; n$.

So in fact, natural numbers are some lambda functions. But it behaves like natural number in the way that we have $0$ and $Succ$.

## Fixpoint (Y combinator)

It's difficult to encode recursive function in lambda calculus. The reason is that functions are annoymous so you cannot call yourself by a name.

So again, we can write the property that we hope the function has and try to come up with a definition in lambda abstractions that satisfy this requirement.

Take multiplication by $2$ as an example. We hope the function has the property:

```
mul2 n := if n == 0 then 0 else Succ(Succ(mul2(n-1)))
```

Since we don't have recurssion naturally in lambda calculus, this function is not in a closed form mathematically. We want to solve this function equation where the variable is a function.

Let's think in this way, `if n == 0 then 0 else Succ(Succ(mul2(n-1)))` looks just like a normal lambda function if we take `mul2` as a parameter. So in fact:
```
mul2 := F mul2
F := lambda mul2, n. n 0 Succ(Succ(mul2 (n-1)))
```
But still this is not a closed form because `mul2` appears in both sides of the equation. Basically `mul2` is the solution (fixpoint) of this equation `a = F(a)`.
Interestingly, in lambda calculus, such fixpoint always exists and can be easily computed by Y combinator defined as:

$$
\text{fix} \; F := \lambda x. \text{fix}' \; \text{fix}' \; F \; x
$$

$$
\text{fix}' := \lambda f, F.F \; (\lambda x. f \; f \; F \; x)
$$

Or simply:

$$
Y := \lambda f.(\lambda x. f \; (x \; x))(\lambda x. f \; (x \; x))
$$

## Scott Encoding for Other Inductive Types

Scott encoding can be used for other inductive types similar to natural number. 
Each value of an inductive type can be thought of as a matching function. If the input matches the inductive construction, then we return a function applied on the inductive origin.

