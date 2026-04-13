---
dimension: 16:9
toplevel-attributes: enter="~duration:0" slip down="~duration:0 title-anchor" unreveal="repo-link"
---
<style>
body {
    font-size: 42px;
}
</style>

{#title}
# Polymorphic bottom-up weighted relational programming

{#authors}
{style="text-align:center"}
### Dmitri Volkov, Yafei Yang, Chung-Chieh Shan

{style="text-align:center"}
### Indiana University

<br />

{style="text-align:center"}
{#repo-link}
> [https://github.com/sporkl/semiringkanren](https://github.com/sporkl/semiringkanren)

<br />

{#title-anchor}
<br />

<br /> <br /> <br /> <br /> <br />

{slip}
{pause}
-----
## Introducing *semiringKanren*:

semiringKanren is a *relational programming language*.

{pause}
Relations take values, and either *succeed* or *fail* for those values. semiringKanren computes success/failure for all possible values.

{pause}
Relations are defined in terms of *primitive relations* and *connectives*. Collectively, we call these *goals*.

{pause}
{center}
{#coin-flip-semiringkanren}
{.example title="Coin flip - semiringKanren"}
---
Aim to write a unary relation expressing coin flip results. 

Represent heads as `(left sole)` and tails as `(right sole)`.


```
(defrel (coin-flip (c : (Sum Unit Unit)))
  (disj
    (== c (left sole))
    (== c (right sole))))
```

Here, `disj` is a connective, and `==` is a primitive relation.

This succeeds for both `(left sole)` and `(right sole)`; both "heads" and "tails" are possible results of a coin flip.

---
{pause}
{up}
{unreveal="weighted-coin-flip"}
semiringKanren variables are typed.

```
; x has type A, y has type B
(x : A)
(y : B)

; Unit type
(sole : Unit)

; Sum type
((left x) : (Sum A B))
((right y) : (Sum A B))

; Product type
((pair x y) : (Prod A B))

; example: Boolean type
(define Bool (Sum Unit Unit))
(define false (left sole))
(define true (right unit))
```

{pause}
{center}

We use `(Num n)` as shorthand for recursively constructed sum types of size n, and numbers to represent values of that type.

For example, the values of type <br /> `(Num 4) = (Sum Unit (Sum Unit (Sum Unit Unit)))` are:
- `0 = (left sole)`
- `1 = (right (left sole))`
- `2 = (right (right (left sole)))`
- `3 = (right (right (right sole)))`

{pause}
{up}
semiringKanren programs are *weighted*.

{.example title="Weighted coin flip"}
---
```
(defrel (weighted-coin-flip (coin : (Sum Unit Unit)))
  (disj
    (conj
      (factor 0.7)
      (== coin (left sole))
    (conj
      (factor 0.3)
      (== coin (right sole))
```

This represents weighted coin flip results, where "heads" has weight 0.7, and "tails" has weight 0.3.

---

{pause}
{center="semirings"}

`(factor w)` adds weight to a program branch, where `w` is drawn from a *commutative semiring*.

{#semirings}
{.definition title="Semirings"}
---
A *semiring* (or *rig*) is a set $R$ with $+$ and $*$ operations, with corresponding identity elements $0$ and $1$, such that:

```math
a * (b + c) = (a * b) + (b * c)
```

(and some additional properties).

Examples: real numbers with addition and multiplication, booleans with $\vee$ and $\wedge$.
 
---

Nonzero weights are treated as successes, and zero is treated as failure.

{up}
{pause}
semiringKanren supports recursive relations!

{.example title="Transitive closure"}
---

Represent a graph as a relation between nodes (pseudo-syntax):

![](transitive-closure-graph.jpg)

```
(defrel (graph (x : Num) (y : Num))
  (disj
    (conj (== x 0) (== y 1))
    (conj (== x 1) (== y 0))
    (conj (== x 1) (== y 2))
    (conj (== x 3) (== y 2))))
```

{pause}
{center}
We can express connectivity (or "transitive closure") in this graph as a recursive relation:

```
(defrel (connect (x : Num) (y : Num))
  (disj
    (graph x y)
    (fresh ((z : Num))
      (conj
        (connect x z)
        (connect z y)))))
```
{pause}
{center}
{#run-transitive-closure}
Note we use the `fresh` connective here, to introduce a *fresh variable*.

{pause}
The results change for different semirings. For booleans, we get reachability:

![](transitive-closure-graph.jpg){style="float:right"}

{style="float:none"}

| | **0** | **1** | **2** | **3** |
| **0**&nbsp; | $\top$ | $\top$ | $\top$ | $\bot$ |
| **1**&nbsp; | $\top$ | $\top$ | $\top$ | $\bot$ |
| **2**&nbsp; | $\bot$ | $\bot$ | $\bot$ | $\bot$ |
| **3**&nbsp; | $\bot$ | $\bot$ | $\top$ | $\bot$ |

(`x` is the row, `y` is the column)

{pause}
{center}
For the *min-tropical* semiring $(\mathbb{R}^\infty, \min, +)$, we get the shortest path length:

> | | **0** | **1** | **2** | **3** |
> | **0**&nbsp; | $2$ | $1$ | $2$ | $\infty$ |
> | **1**&nbsp; | $1$ | $2$ | $1$ | $\infty$ |
> | **2**&nbsp; | $\infty$ | $\infty$ | $\infty$ | $\infty$ |
> | **3**&nbsp; | $\infty$ | $\infty$ | $1$ | $\infty$ |

Assuming each path in the original graph is given weight 1 with `(factor 1)`.

{slip}
{pause}
-----

## Semantics and Implementation

<br />

{pause}

{#relations-are-arrays}
> # Relations compute (nd)arrays.
> {style="text-align:center"}
> [(with associated types)]{#with-associated-types}

{pause}
{#variables-are-dimensions}
# Variables are specific dimensions of relation arrays.

{pause}
{#goals-are-operations}
# Goals are operations on relation arrays.

{#variable-vectors}
----

{static="relations-are-arrays variables-are-dimensions"}
{unstatic="goals-are-operations with-associated-types"}
{up="relations-are-arrays"}

{pause}
Values are represented as 1-dimensional arrays.

{.example title="Pair of booleans"}
{up="sole-to-one"}
{pause}
---
`(pair (left sole) (right sole))` of type:

`(Prod (Sum Unit Unit) (Sum Unit Unit))`

{pause}
```math
\text{sole} \leadsto [1]
```

{pause}
{center}
```math
(\text{left}\;\text{sole}) : (\text{Sum}\;\text{Unit}\;\text{Unit}) \leadsto [1] \oplus [0] = [1, 0]
```
```math
(\text{right}\;\text{sole}) : (\text{Sum}\;\text{Unit}\;\text{Unit}) \leadsto [0] \oplus [1] = [0, 1]
```

{pause}
```math
(\text{pair}\;(\text{left}\;\text{sole})\;(\text{right}\;\text{sole})) \leadsto [1, 0] \otimes [0, 1] = [0, 1, 0, 0]
```

{pause}
{center}
All encodings for the type:
```math
(\text{pair}\;(\text{left}\;\text{sole})\;(\text{left}\;\text{sole})) \leadsto [1, 0, 0, 0]
```
```math
(\text{pair}\;(\text{left}\;\text{sole})\;(\text{right}\;\text{sole})) \leadsto [0, 1, 0, 0]
```
```math
(\text{pair}\;(\text{right}\;\text{sole})\;(\text{left}\;\text{sole})) \leadsto [0, 0, 1, 0]
```
```math
(\text{pair}\;(\text{right}\;\text{sole})\;(\text{right}\;\text{sole})) \leadsto [0, 0, 0, 1]
```

---

{pause}
{unreveal="relations-compute-arrays-2"}
{center}
Values become one-hot vectors. Type size becomes vector size.

{#relation-arrays}
----

{#relations-compute-arrays-2}
# Relations compute (nd)arrays.

{pause}
{#two-hot-vector}
{.example title="Two-hot vector"}
---
What if we have something like $[1, 0, 0, 1]$?

{pause}
{reveal="relations-compute-arrays-2"}

{pause}
`(pair (left sole) (left sole))`, `(pair (right sole) (right sole))` succeed! The other values fail.

---

{pause}
{up="relations-compute-arrays-2"}

Relations compute arrays: nonzero entries succeed, zero entries fail.

{pause}

Relations on more than one variable?

{pause}
{center}
{#variables-are-dimensions-2}
# Variables are specific dimensions of relation arrays.

{pause}
{up="variables-are-dimensions-2"}

Consider some relation on two booleans:
```
(defrel (bool-rel (x : Bool) (y : Bool))
  ...)
```

{pause}
Then `x` and `y` each get a dimension of the relation's array:

```math
\begin{matrix}
    \quad \; \; y \rightarrow \\
    x \downarrow
    \begin{bmatrix}
        [a, b], \\
        [c, d] \\
    \end{bmatrix}
\end{matrix}
```

{pause}
{down}
{.example title="Equal booleans"}
---
If we consider the relation where `x` equals `y`...

```
(defrel (equal-bools (x : Bool) (y : Bool))
  (== x y))
```

{pause}
Then `x` should be `true` when `y` is `true`, and same for `false`.

{style="float:right"}
```math
\begin{bmatrix}
[1, 0], \\
[0, 1]
\end{bmatrix}
```

{style="float:none"}
> | | `true` &nbsp;&nbsp; | `false` |
> | --- | --- | --- |
> | **`true`** &nbsp;| $1$ | $0$ |
> | **`false`** &nbsp;| $0$ | $1$ |

{pause}
{center}
{.example title="One specific boolean"}
---
If we consider a relation where only one variable is conditioned:

```
(defrel (one-specific-bool (x : Bool) (y : Bool))
  (== x true))
```

{pause}
{unreveal="goals-are-operations-2"}
Then the values for each variable are restricted/unrestricted accordingly:

{style="float:right"}
```math
\begin{bmatrix}
[1, 1], \\
[0, 0]
\end{bmatrix}
```

{style="float:none"}
> | | `true` &nbsp;&nbsp; | `false`&nbsp;&nbsp; |
> | --- | --- | --- |
> | **`true`** | $1$ | $1$ |
> | **`false`** &nbsp; | $0$ | $0$ |

{#goal-operations}
----

{#goals-are-operations-2}
# Goals are operations on relation arrays.

{center}
{pause}
How do we construct and use these relation arrays in practice?

{pause}
{reveal="goals-are-operations-2"}
{up="goals-are-operations-2"}

{pause}
There are two classes of goals:

{pause}
{#relation-goals}
> ## Primitive relations
> `==`, `=/=`, `succeed`, `fail`, `factor`, and relation application

{pause}
{#combine-goals}
> ## Connectives
> `conj`, `disj`, `fresh`.

{pause}
{static="relation-goals"}
{unstatic="combine-goals"}
{up="relation-goals"}

{pause}

[`(== x y)`]{style="float:left"}
```math
\begin{bmatrix}
1 & 0 & 0 \\
0 & \ddots & 0 \\
0 & 0 & 1
\end{bmatrix}
```

{pause}

[`(=/= x y)`]{style="float:left"}
```math
\begin{bmatrix}
0 & 1 & 1 \\
1 & \ddots & 1 \\
1 & 1 & 0
\end{bmatrix}
```

{pause}
[`(factor 120)`]{style="float:left"}
```math
\begin{bmatrix}
120 & 120 & 120 \\
120 & \ddots & 120 \\
120 & 120 & 120
\end{bmatrix}
```

{pause}
{center}
`succeed` is `(factor 1)`, `fail` is `(factor 0)`.

{pause}

Relation application unifies argument variables with precalculated relation array variables.

{pause}
{center}
{#goal-combinators-2}
> ## Connectives
> `conj`, `disj`, `fresh`.

{pause}
{up="goal-combinators-2"}

`conj` uses semiring multiplication:

```math
(\text{conj}\;
\begin{bmatrix}
2 & 2 & 2 \\
1 & 1 & 1 \\
0 & 0 & 0
\end{bmatrix}
\;
\begin{bmatrix}
0 & 1 & 2 \\
0 & 1 & 2 \\
0 & 1 & 2
\end{bmatrix})
\leadsto
\begin{bmatrix}
0 & 2 & 4 \\
0 & 1 & 2 \\
0 & 0 & 0
\end{bmatrix}
```

{pause}

`disj` uses semiring addition:
```math
(\text{disj}\;
\begin{bmatrix}
2 & 2 & 2 \\
1 & 1 & 1 \\
0 & 0 & 0
\end{bmatrix}
\;
\begin{bmatrix}
0 & 1 & 2 \\
0 & 1 & 2 \\
0 & 1 & 2
\end{bmatrix})
\leadsto
\begin{bmatrix}
2 & 3 & 4 \\
1 & 2 & 3 \\
0 & 1 & 2
\end{bmatrix}
```

{pause}
{center}
`fresh` uses summation:
```math
(\text{fresh}\;((x:\dots))\;
\begin{bmatrix}
0 & 1 & 2 \\
0 & 1 & 2 \\
0 & 1 & 2
\end{bmatrix})
\leadsto
\begin{bmatrix}
0 & 3 & 6
\end{bmatrix}
```

----

{pause}
{center}
{#what-about-recursion}
## What about recursion?

{pause}
{up="what-about-recursion"}
Assume relations *fail* until we have more information.

{pause}
Recompute once we have the information.

{pause}
Repeat until there's no new information - program execution by *fixpoint*.

{.example title="Transitive closure step-by-step"}
---

Recall transitive closure example with $(\mathbb{R}^\infty,\min,+)$: [![](transitive-closure-graph.jpg)]{style="float:right"}

```
(defrel (connect (x : Num) (y : Num))
  (disj
    (graph x y)
    (fresh ((z : Num))
      (conj
        (connect x z)
        (connect z y)))))
```

{pause}
{center}
First assume nothing: [![](transitive-closure-graph.jpg)]{style="float:right"}
<br /><br />

```math
\begin{bmatrix}
\infty & \infty & \infty & \infty \\
\infty & \infty & \infty & \infty \\
\infty & \infty & \infty & \infty \\
\infty & \infty & \infty & \infty \\
\end{bmatrix}
```

{pause}
{center}
`connect` calls fail, but `graph` calls do not:
```math
\begin{bmatrix}
\infty & 1 & \infty & \infty \\
1 & \infty & 1 & \infty \\
\infty & \infty & \infty & \infty \\
\infty & \infty & 1 & \infty \\
\end{bmatrix}
```

{pause}
{center}
Now we can consider intermediate vertices:
```math
\begin{bmatrix}
2 & 1 & 2 & \infty \\
1 & 2 & 1 & \infty \\
\infty & \infty & \infty & \infty \\
\infty & \infty & 1 & \infty \\
\end{bmatrix}
```

Re-evaluating `connect` gets the same result. All done! [![](transitive-closure-graph.jpg)]{style="float:right"}

---

{pause}
{down}
> This is "bottom-up evaluation", similar to Datalog.
>
> Note we can approximate nonterminating recursions by computing the fixpoint up to bounded precision.
> <br /> <br />

{slip}
{pause}
-----

<!--  introduce polymorphic version of the language, with example programs -->

{top}
## Polymorphic semiringKanren

semiringKanren supports *polymorphic relations*; relations which take in values of unknown types (represented with *type variables*).

{pause}
{up}
{.example title="Option map"}
{#ex-option-map}
---
```
(defrel (option-map
  (f : (Prod α β)) (x : (Sum Unit α)) (y : (Sum Unit β)))
  (disj
    (conj (== x (left sole)) (== y (left sole)))
    (fresh ((a : α) (b : β))
      (conj
        (== x (right a))
        (== f (pair a b))
        (== y (right b))))))

(defrel (main (x : (Sum Unit (Num 4))) (y : (Sum Unit (Num 4))))
  (fresh ((a : Bool) (b : Bool))
    (conj
      (=/= a b)
      (option-map (pair a b) x y))))
```

{pause}
{down="ex-option-map"}
`main` gets denoted as:

```math
\begin{bmatrix}
    1 & 0 & 0 & 0 & 0 \\
    0 & 0 & 1 & 1 & 1 \\
    0 & 1 & 0 & 1 & 1 \\
    0 & 1 & 1 & 0 & 1 \\
    0 & 1 & 1 & 1 & 0
\end{bmatrix}
```
---

{pause}
{center}
How is `option-map` denoted?

{pause}
semiringKanren compiles polymorphic programs to non-polymorphic programs.

{pause}
{up}
{.example title="Option map - compiled"}
---
{#compiled-option-map}
```
(defrel (option-map
          (f : (Prod (Num 3) (Num 3)))
          (x : (Sum Unit (Num 3)))
          (y : (Sum Unit (Num 3))))
  (disj
    (conj (== x (left sole)) (== y (left sole)))
    (fresh ((a : (Num 3)) (b : (Num 3)))
      (conj
        (== x (right a))
        (== f (pair a b))
        (== y (right b))))))
```

{pause}
Within `main`, the relation call `(option-map (pair a b) x y)` compiles to:

{pause}
{down}
{#compiled-option-map-call}
{style="font-size:38px"}
```
(fresh ((om-f : (Prod (Num 3) (Num 3)))
  (om-x : (Sum Unit (Num 3))) (om-y : (Sum Unit (Num 3))))
  (conj
    (option-map om-f om-x om-y)
    (fresh ((om-f-hd : (Num 3)) (om-x-right : (Num 3))
      (f-hd : Bool) (x-right : Bool))
      (conj
        (== (pair om-f-hd _) om-f) (== (right om-x-right) om-x)
        (== (pair f-hd _) (pair a b)) (== (right x-right) x)
        (disj
          (conj (== om-f-hd om-x-right) (== f-hd x-right))
          (conj (=/= om-f-hd om-x-right) (=/= f-hd x-right)))))
    (fresh ((om-f-tl : (Num 3)) (om-y-right : (Num 3))
      (f-tl : Bool) (y-right : Bool))
      (conj
        (== (pair _ om-f-tl) om-f) (== (right om-y-right) om-y)
        (== (pair _ f-tl) (pair a b)) (== (right y-right) y)
        (disj
          (conj (== om-f-tl om-y-right) (== f-tl y-right))
          (conj (=/= om-f-tl om-y-right) (=/= f-tl y-right)))))))
```
---

{pause}
{center}
Type variables in polymorphic relations are converted into concrete types.

{center="compiled-option-map"}

{pause}
{center}
Polymorphic relation calls extract the *equality patterns* of variable-typed values, and enforce those patterns on the arguments.

{down="compiled-option-map-call"}

{up="ex-option-map"}

{pause}
{center}
Given `α = (Num 3)` and `β = (Num 3)`, `option-map` has the denotation:

```math
\begin{bmatrix}
    1 & \begin{matrix} 0 & 0 & 0 \end{matrix} \\
    \begin{matrix} 0 \\ 0 \\ 0 \end{matrix} & \begin{matrix} \huge{f} \end{matrix}
\end{bmatrix}
```

Note this is actually a 3-dimensional array, with the third dimension corresponding to $f$. <br/>
`f : (Prod (Num 3) (Num 3))`, so the size of the third dimension is 9.

<br />

{pause}
{center}
How does semiringKanren determine the specific concrete types for type variables?

The concretized types need to be encompass any possible equality pattern.

{pause}
In the most extreme case, all occurrences of a type variable are disequal. We need a type large enough so all type variable occurrences can have different values.

So, we just need to count the maximum number of occurrences of a type variable.

{up="ex-option-map"}

{center="compiled-option-map"}

<br />

{pause}
{up}
All done?

{pause}
{.example title="Two-valued type"}
{#ex-two-valued}
---
```
(defrel (two-valued (x : α))
  (fresh ((y : α))
    (=/= x y)))
```

`α = (Num 2)`, so this always succeeds, so the denotation is $\begin{bmatrix} 1 & 1 \end{bmatrix}$.

{pause}
{center}
Then the call `(two-valued sole)` compiles to:

```
(fresh ((tv-x : (Num 2)))
  (conj
    (two-valued tv-x)
    (disj
      (conj (== tv-x tv-x) (== sole sole))
      (conj (=/= tv-x tv-x) (=/= sole sole)))))
```

which succeeds.

{pause}
{down="ex-two-valued"}

But "morally," `(two-valued sole)` should be equivalent to:
```
(fresh ((y : Unit))
    (=/= sole y)))
```

{up="ex-two-valued"}

{pause}
{down="ex-two-valued"}

```
(=/= sole sole)
```

which should fail!
---

{pause}
{center}
So we also need to generate smaller versions of the polymorphic relations.

<br />

{slip}
{pause}
-----

## Formalization (overview)

Variable types are tracked in *type environments*, notated $\Delta$.

Variable values are tracked in *value environments*, notated $\delta$.

{pause}

A value environment $\delta = x_1 \mapsto v_1, \dots, x_n \mapsto v_n$ <br />
has a corresponding type environment $\Delta = x_1 : \tau_1, \dots, x_n : \tau_n$ <br />
when $v_i : \sigma(\tau_i)$ for some type variable substitution $\sigma$.

{pause}

We write $[\![ e ]\!] (\delta)$ for "the denotation of the value of $e$ under value environment $\delta$."

{.example title="Pair of x"}
---
$[\![ \mathtt{(pair}\;x\;x\mathtt{)} ]\!] (x \mapsto \mathtt{sole}) = \mathtt{(pair\;sole\;sole)}$
---

{pause}
{center}

We say value environments $\delta_1, \delta_2$ have the same *equality pattern*, notated $\delta_1 \leftrightharpoons_\Delta \delta_2$, when:

- The "concrete" parts, or *shells*, of both environments are equal.
- Let $h_1, k_1$ be values in $\delta_1$ corresponding to variable-typed parts of $\Delta$ ("*holes*").<br/>Let $h_2, k_2$ be corresponding values in $\delta_2$. Then:
    - $h_1 = k_1$ and $h_2 = k_2$, or
    - $h_1 \ne k_1$ and $h_2 \ne k_2$.

{pause}
{center}

{.example title="Equality patterns"}
---
Consider $\Delta = x : \mathtt{(Sum\;\alpha\;\alpha), y : \alpha}$.

Does the following hold? ($\alpha = \mathtt{sole}$)
```math
x \mapsto \mathtt{(left \; sole)}, y \mapsto \mathtt{sole} \; \leftrightharpoons_\Delta \; x \mapsto \mathtt{(right \; sole), y \mapsto \mathtt{sole}}
```

{pause}

No, because the shells of $x$, $\mathtt{(left \; \dots)}$ and $\mathtt{(right \; \dots)}$ are not equal.

{pause}
{center}

Does the following hold? ($\alpha = \mathtt{Bool}$)
```math
x \mapsto \mathtt{(left \; true)}, y \mapsto \mathtt{false} \; \leftrightharpoons_\Delta \; x \mapsto \mathtt{(left \; true), y \mapsto \mathtt{true}}
```

{pause}

No, because the holes are disequal under $\delta_1$ ($\mathtt{true} \ne \mathtt{false}$), but are equal under $\delta_2$ ($\mathtt{true} = \mathtt{true}$).

{pause}
{center}

Does the following hold? ($\alpha = \mathtt{Bool}$)
```math
x \mapsto \mathtt{(left \; true)}, y \mapsto \mathtt{false} \; \leftrightharpoons_\Delta \; x \mapsto \mathtt{(left \; false), y \mapsto \mathtt{true}}
```

{pause}

Yes! The holes are disequal both under $\delta_1$ ($\mathtt{true} \ne \mathtt{false}$), and under $\delta_2$ <br />($\mathtt{false} \ne \mathtt{true}$).

{pause}
{center}

Does the following hold? ($\sigma_1(\alpha) = \mathtt{Bool}, \sigma_2(\alpha) = \mathtt{Unit}$)
```math
x \mapsto \mathtt{(left \; false)}, y \mapsto \mathtt{false} \; \leftrightharpoons_\Delta \; x \mapsto \mathtt{(left \; sole), y \mapsto \mathtt{sole}}
```

{pause}

Yes! The holes are equal both under $\delta_1$ ($\mathtt{false} = \mathtt{false}$), and under $\delta_2$ <br />($\mathtt{sole} = \mathtt{sole}$).
---

{pause}
{center}

Let $[\![g]\!](\eta;\delta)$ denote the *weight* of goal $g$ under relation environment $\eta$ and value environment $\delta$.

{pause}

{.theorem}
---
If $[\![g]\!](\eta;\delta_1) = w$ and $\delta_1 \leftrightharpoons_\Delta \delta_2$, then $[\![g]\!](\eta;\delta_2) = w$.
---

{pause}
(Proof in progress...)

{pause}
{center}

Consider the case $g = \mathtt{(fresh \; ((x:\tau)) \; g')}$.

{pause}

```math
[\![g]\!](\eta; \delta) = \sum_{i \in \tau} [\![g']\!](\eta; \delta,x \mapsto i)
```

{pause}

What if $\tau = \alpha$?

{pause}
{center}

```math
\sum_{i \in \sigma_1(\tau)} [\![g']\!](\eta; \delta_1,x \mapsto i) = w_1 + \dots + w_m
```
```math
\sum_{j \in \sigma_2(\tau)} [\![g']\!](\eta; \delta_2,x \mapsto j) = w'_1 + \dots + w'_n
```
```math
w_1 + \dots + w_m \stackrel{?}{=} w'_1 + \dots + w'_n
```

{pause}
{center}

Let $\#_\alpha \Delta$ denote the number of occurrences of values of type $\alpha$ in $\Delta$.

Assuming $\sigma(\alpha)$ is large enough, we know $i$ can be either equal or disequal to $\alpha$-holes in $\delta$.
Thus $[\![g']\!](\eta; \delta,x \mapsto i)$ inhabits at most $\#_\alpha \Delta$ possible weights.

{pause}

We can show $\delta_1,x \mapsto i \leftrightharpoons_\Delta \delta_2, x \mapsto j$.
So by the inductive hypothesis, the inhabited weight values are the same.

{pause}
{center}

Let $r_1, \dots, r_k$ be the weight values. Commutative semiring, so can rearrange:

```math
\sum_{i \in \sigma_1(\tau)} [\![g']\!](\eta; \delta_1,x \mapsto i) = (r_1 + \dots) + \dots + (r_k + \dots)
```
```math
\sum_{j \in \sigma_2(\tau)} [\![g']\!](\eta; \delta_2,x \mapsto j) = (r_1 + \dots\dots) + \dots + (r_k + \dots\dots)
```
```math
(r_1 + \dots) + \dots + (r_k + \dots) \stackrel{?}{=} (r_1 + \dots\dots) + \dots + (r_k + \dots\dots)
```

{pause}
{center}

What now?

{pause}

Also require semiring addition to be *idempotent*. ($x + x = x$)

Then:
```math
(r_1 + \dots) + \dots + (r_k + \dots) = r_1 + \dots + r_k = (r_1 + \dots\dots) + \dots + (r_k + \dots\dots)
```

{pause}
{center}

So: 
```math
\sum_{i \in \sigma_1(\tau)} [\![g']\!](\eta; \delta_1,x \mapsto i) = \sum_{j \in \sigma_2(\tau)} [\![g']\!](\eta; \delta_2,x \mapsto j)
```

And so $[\![\mathtt{(fresh \; ((x:\tau)) \; g')}]\!](\eta,\delta_1) = [\![\mathtt{(fresh \; ((x:\tau)) \; g')}]\!](\eta,\delta_2)$.

{center="~duration:5 authors"}
{reveal="repo-link"}
