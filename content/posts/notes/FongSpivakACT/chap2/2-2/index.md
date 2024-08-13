+++
tags = ["category theory","lecture"]
title = '2.2 - Symmetric monoidal preorders'
date = 2024-08-02T01:26:30-07:00
draft = false
+++

## 2.2.1 - Definition and first examples

> **Definition 2.2:** A *symmetric monoidal structure* on a preoirder $(X, \leq)$ consists of
> - (i) a *monoidal unit*, $I \in X$
> - (ii) a *monoidal product* $\otimes: X \times X \rightarrow X$
>
> And the monoidal product $\otimes(x_1,x_2) = x_1 \otimes x_2$ must also satisfy the following properties (assume all elements are in $X$)
> - (a) $x_1 \leq y_1$ and $x_2 \leq y_2 \implies x_1 \otimes x_2 \leq y_1 \otimes y_2$
> - (b) $I \otimes x = x \otimes I = x$
> - (c) associativity
> - (d) commutivity/symmetry
> 
> (a) is called *monotnoicity* and (b) is *unitality*

**Remark 2.3:** replacing $=$ with $\cong$ in definition 2.2 will give us a *weak monoidal structure*.

> **Exercise 2.5:** The preorder structure $(\mathbb{R}, \leq)$ and the multiplication operation $\times$ will not give us a symmetric monoidal order because of the simple counter example of $-2 \times -2 \nleq 1 \times 1$.

> **Example 2.6:** A *monid* is similar to a symmetric monoidal preorder in that it consists of a set $M$, a function $*: M\times M \rightarrow M$, and an elment $e \in M$ called the *monid unit*, such that for every $m,n,p \in M$,
> - $m * e = m$
> - $e * m = m$
> - associativity holds
> 
> Further, if commutivity holds (which isn't not generally true),  then it is also called *commutative*

## 2.2.2 - Introducing wiring diagrams
:warning: I am seeing the wiring diagrams, but I fail to understand why they are any different from the Hasse diagrams we've seen previously.

Essentially, wiring diagrams seem to be a way to encode information about symmetric monoidal structures. The basic rules built up so far are as follows:
- A wire without a label, or with the label of the monoidal unit, is equivalent to nothing
- Otherwise, a wire labeled with an element represents that element (:warning: this could be wrong)
- Two parallel wires represent the monoidal product of those elements
- Placing a $\leq$ block between two $x,y$ wires indicates that $x \leq y$

Thinking back to the conditions for a symmetric monoidal structure, we find that
1. Transitivity allows us to combine wiring diagrams left to right
{{< figure src="./images/transitivitypt1.png" width="400px" align="center">}}

1. Monotonicity is represented as being able to combine wiring diagrams top to bottom
{{< figure src="./images/monotonicity.png" width="400px" align="center">}}

1. A monoidal product with a monoidal unit and another element gives us the element again, because the monoidal unit is equivalent to nothing (reflexivity)

1. Associativity means we can "wiggle" around parallel wires
{{< figure src="./images/associativity.png" width="400px" align="center">}}

1. Commutivity means we can cross wires
{{< figure src="./images/commutivity.png" width="400px" align="center">}}

It is intuitive to see how these wiring diagrams can be used to prove statements. In fact, the above images are trivial proofs. Take a look at the following exercise

> **Exercise 2.20:**
> Prove--given $t \leq v+w$, $w+u \leq x+z$, and  $v+x \leq y$--that $t+u \leq y+z$.
>  
> ALgebraically, we proceed like so:
> $$\begin{align} 
    t + u &\leq (v+w) + u \\
    &\leq v + (w+u) \\
    &\leq v + (x+z) \\
    &\leq (v + x) + z \\
    &\leq y + z \\
\end{align}$$
> and the wiring diagram would look like
> {{< figure src="./images/wiringex.png" caption="The squares are the $\leq$ blocks" width="400px" align="center">}}

## 2.2.3 - Applied examples
While this section did solidify some concepts. It wasn't too important. Although, it did carry two useful examples: discarding and splitting.

With discarding, if a symmetric monoidal structure also satisfies $x \leq I$ for every $x \in X$, then it is possible to terminate any wire:
{{< figure src="./images/discard.png" width="300px" align="center">}}

And if instead have a property like $x \leq x + x$, then we can split any wire:
{{< figure src="./images/splitting.png" width="300px" align="center">}}

## 2.2.4 - Abstract examples 
Again, after a skim through, this section did not seem critical.

## 2.2.5 - Monoidal montone maps
We begin with recalling that for any preorder $(X,\leq)$ we have an induced equivalence relation $\cong$ on $X$ where two elements $x \cong x' \iff x \leq x$ and $x' \leq x$

> **Definition 2.41:** $\mathcal{P} = (P, \leq_P, I_P, \otimes_P)$ and $\mathcal{Q} = (Q, \leq_Q, I_Q, \otimes_Q)$ be monoidal preorders. A __monoidal monotone__ from $\mathcal{P}$ to $\mathcal{Q}$ is a monotone map $f: (P, \leq_P) \rightarrow (Q, \leq_Q)$ which satisfies
> - (a) $I_Q \leq_Q f(I_P)$
> - (b) $f(p_1) \otimes_Q f(p_1 \otimes_P p_2)$
> for all $p_1,p_2 \in P$.
> 
> Additionally, $f$ is a _strong monoidal monotone_ if it satisfies
> - (a') $I_Q \cong f(I_P)$
> - (b') $f(p_1) \otimes_Q f(p_1 \otimes_P p_2) \cong f(p_1) \otimes_Q f(p_2)$
> And it is called a _strict monoidal monotone_ if it satisfies
> - (a'') $I_Q = f(I_P)$
> - (b'') $f(p_1) \otimes_Q f(p_1 \otimes_P p_2) = f(p_1) \otimes_Q f(p_2)$

Monoidal monotones are said to be examples of _monoidal functors_ in category theory.

The exercises for this section seem a little easy, so I will be skipping them for now, returning to them if I get confused on the defnitions of monoidal monotones.
