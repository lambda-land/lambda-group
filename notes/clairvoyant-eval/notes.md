# *Call-by-Need Is Clairvoyant Call-by-Value* (Hackett and Hutton)

We spent much of the reading group discussing the presentation of the "natural semantics" for the untyped λ-calculus (Fig. 1 in the paper). "Natural semantics" in this context just refers to a big-step semantics, but the way variables are handled is a bit interesting.

The crucial thing about the rules is that:

- We don't substitute arbitrary terms for variables, only new variables (enforced by the fact that only a variable can be on the right-hand side of an application in the syntax);
- We *never remove variables created by let-bindings from the heap*. Instead, we use renaming to prevent name conflicts.

The images in this directory provide a few examples of how these rules come into play. For example, suppose $z_1, z_2$ are values. The expression

$$(\textbf{let } y = z_1 \textbf{ in } \lambda x. y) \space z_2$$

will first evaluate the **let**-expression and become

$$(\lambda x. y) \space z_2$$

with $y$ bound to $z_1$ in the heap. Note that (following the LET rule) **let** doesn't substitute the newly-bound variable into its RHS, it just creates a new binding, evaluates the RHS, and then returns it unmodified. This means that the RHS can potentially contain references to the new variable (as in this case), which is why the binding needs to be kept in the heap.

I also want to contextualize the $\hat z$ notation in the VAR rule. The paper doesn't explain this beyond saying that $z$ is "alpha-renamed so as to avoid unwanted name capture," which I found a little vague.

What's actually going on is that we are renaming *all bound variables in $z$ to be fresh with respect to those already bound in $\Delta$ and $x$*. This ensures that when we ultimately evaluate the body of $z$ and generate new bindings, these bindings will not conflict with those in the heap. We don't have to rename the $z$ that we store back into $\Delta$ because any subsequent accesses to $x$ will also be renamed.

The [attached image](IMG_3142.jpeg) contains an example of an expression that "goes wrong" without this rule. Let $z$ be a variable referring to a value $z_1$. Then we can reduce the expression as follows:

$$
\begin{aligned}
z \mapsto z_1 &: \textbf{let }x=(\lambda y. \textbf{ let } a = y \textbf { in } a z)\textbf{ in } x x
& (1) \\
z \mapsto z_1 ,\space x \mapsto (\lambda y. \textbf{ let } a = y \textbf { in } a z) &: x x
& (2) \\
z \mapsto z_1 ,\space x \mapsto (\lambda y. \textbf{ let } a = y \textbf { in } a z) &: (\lambda y. \textbf{ let } a = y \textbf { in } a z) x
& (3) \\
z \mapsto z_1 ,\space x \mapsto (\lambda y. \textbf{ let } a = y \textbf { in } a z) &: \textbf{let } a = x \textbf { in } a z
& (4) \\
z \mapsto z_1 ,\space x \mapsto (\lambda y. \textbf{ let } a = y \textbf { in } a z), \space a \mapsto x &: a z
& (5) \\
z \mapsto z_1 ,\space x \mapsto (\dots), \space a \mapsto (\lambda y. \textbf{ let } a = y \textbf { in } a z) &: (\lambda y. \textbf{ let } a = y \textbf { in } a z) z
& (6) \\
z \mapsto z_1 ,\space x \mapsto (\dots), \space a \mapsto (\lambda y. \textbf{ let } a = y \textbf { in } a z) &: \textbf{let } a = z \textbf { in } a z
& (7) \\
z \mapsto z_1 ,\space x \mapsto (\dots), \space {\color{red}a} \mapsto (\lambda y. \textbf{ let } a = y \textbf { in } a z), \space {\color{red}a} \mapsto z &: {\color{red}a} z
& (8)
\end{aligned}
$$

Now there is a problem, because we have two bindings for the variable $a$ in the heap and no way to choose between them. (A policy of "always choose the latest binding" works here, but breaks the principle that the heap is unordered, and we can come up with other examples where it is necessary to choose an earlier binding from the heap.)

Instead, we should have renamed the result of evaluating $a$ after step 5.

$$
\begin{aligned}
\cdots \\
z \mapsto z_1 ,\space x \mapsto (\lambda y. \textbf{ let } a = y \textbf { in } a z), \space a \mapsto x &: a z
& (5) \\
z \mapsto z_1 ,\space x \mapsto (\dots), \space a \mapsto (\lambda y. \textbf{ let } a = y \textbf { in } a z) &: (\lambda y. \textbf{ let } {\color{blue}b} = y \textbf { in } {\color{blue}b} z) z
& (6) \\
z \mapsto z_1 ,\space x \mapsto (\dots), \space a \mapsto (\lambda y. \textbf{ let } a = y \textbf { in } a z) &: \textbf{let } {\color{blue}b} = z \textbf { in } {\color{blue}b} z
& (7) \\
z \mapsto z_1 ,\space x \mapsto (\dots), \space a \mapsto (\lambda y. \textbf{ let } a = y \textbf { in } a z), \space {\color{blue}b} \mapsto z &: {\color{blue}b} z
& (8) \\
z \mapsto z_1 ,\space x \mapsto (\dots), \space a \mapsto (\lambda y. \textbf{ let } a = y \textbf { in } a z), \space {\color{blue}b} \mapsto z_1 &: z_1 z
& (9) \\
\end{aligned}
$$

And the evaluation is successful.