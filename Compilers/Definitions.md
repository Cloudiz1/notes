## Dominance
$$\text{DOM}(n) = \{n\} \text{ } \cup \left( \bigcap_{m \in preds(n)} \text{DOM} (m)\right)$$
Node $n$ dominates node $m$ if and only if every path from the entry node $n_0$ to $m$ contains $n$. We define $\text{DOM} (n_{0}) = \{n_{0}\}$ where $n_{0}$ is the entry node. There will be an $m \in \text{DOM}(n)$ where $m \ne n$ that is closer to $n$ than any other node. We call this the immediate dominator of $n$.

Iterative data-flow analysis tells us that the $\text{DOM}$ sets given by this iterative algorithm has a fixed point and that fixed point is unique. Since the solution is unique, the order of traversal is actually not important. We pick specifically a post order traversal since they are optimal for forward data-flow problems (because we need $n$'s predecessor to get information about $n$, data flows *forward* through the edges of the CFG). 

We can also define a handy data structure using immediate dominance relationships. The *dominator tree* has a node $n$ for every block in the CFG. Edges encode immediate dominance. That is, if $m = \text{IDOM}(n)$, then $m$ is a child of $n$. 
### Dominance Frontiers

In general, a definition $a$ in some block $B_{i}$ will need a $\phi$-function in any node that lies one CFG edge beyond the region that $B_{i}$ dominates. The dominance frontier of $B_{i}$, $\text{DF}(B_{i})$, is the set of all such nodes. In other words $q \in \text{DF}(p)$ if along some path $q$ lies on edge beyond the region that $p$ dominates. two results follow.
$$preds(q) \in \text{DOM}(p)$$
$$p \not\in (\text{DOM}(q) - q)$$
We can use dominator trees to compute a dominance frontier by iterating over the join point $n$'s CFG predecessors $p$ and inserting $n$ into $\text{DF}(p)$ as needed by the following rules:
- If $p = \text{IDOM(n)}$, then $n \not\in \text{DF}(p)$ and $n \not\in \text{DF}(m), \forall m \in preds(p)$.
- If $p \neq \text{IDOM}(n)$, then $n \in \text{DF}(p)$ and $n \in \text{DF}(q)$ where $q \in \text{DOM}(p)$ and $q \not\in (\text{DOM}(n) - n)$
## Live-Variable Analysis

$$\text{LiveOut}(n)=\bigcup_{m \in succ(n)} (\text{UEVar}(m)) \text{ } \cup \text{ } (\text{LiveOut}(m) \text{ } \cap \text{ } \overline{\text{VarKill}(m)})$$
where $\text{UEVar}(m)$ is the upward-exposed variables of $m$, the variables that are used in block $m$ before any redefinition of $m$. "$\text{VarKill}(m)$ contains all variables that are defined in $m$ and the overline on $\text{VarKill}(m)$ indicates its logical complement, the set of all variables not defined in $m$." You can interpret this equation in two parts, the left side of the union, and the right side of the union. A variable is live if one of two things are true. It can be referenced in $m$ before it is redefined in $m$, in which case $v \in \text{UEVar}(m)$. Or it can be live on exit of $m$ and not be defined in $m$, in which case $v \in \text{LiveOut}(m) \text{ } \cup \overline{\text{VarKill}(m)})$. 

Finding potentially undefined variables is actually quite easy once we've computed $\text{LiveOut}$ sets for all blocks. If we assume the entry block $b_{0}$ has no definitions, yet there exists $v \in \text{LiveOut}(b_{0})$, then we know there exists a path from $b_0$ to $v$ where $v$ is undefined. Intuitively, a variable can not be live before it is defined, so $v \in \text{LiveOut}(b_{0})$ is nonsensical.

There are unfortunately a few scenarios that break this:
- Defining $v$ through a name that is different from $v$, for example, a pointer to $v$
- If $v$ is defined before our entry block. This breaks in the scenario of static variables and functions, for example.
- The path it finds might be unreachable, for example:
```C
int i, n, s;
scanf("%d", &n); 
i = 1;
while (i <= n) {
	if (i == 1) 
		s = 0;
	s = s + i++;
}
```
`s` is always defined before we reach the usage, but because of the `if` statement, the compiler may believe the line `s = 0` will not always run.

