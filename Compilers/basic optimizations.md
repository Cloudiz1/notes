Local value numbering is done very simply. Give each expression an ID, build an ID for the new expression, if it exists, it is redundant. It is important to consider ordering. Especially in the context of CSE, the way we group expressions may expose another redundant expression. For example, consider the following example:
```
a = 2 * 3 * 4
b = 3 * 4
```
becomes:
```
t0 = 2 * 3
a = t0 * 4
b = 3 * 4
```
this example doesn't find any redundancies, however, if we reorder a few things:
```
t0 = 3 * 4
a = 2 * t0
b = 3 * 4 // same as t0!
```
this one *does* find a redundancy. Of course, ordering is a heuristic, and we can't always find the perfect one, but compilers typically impose a global ordering such that something like `2 * 3` and `3 * 2` do become the same expression, for example.

Tree height balancing:
having a left or right associative tree for a long list of additions signals to the compiler that you have to do it in that exact order (say, left-to-right for the expression: `a+b+c+d+e+f+g+h`). However, this can be parallelized. If we have a balanced tree, we can see that we can compute `a+b` and `g+h` simultaneously, for example.

This optimization only works for operations that are commutative and associative, and only works for expressions that only contain the same operation.

Loop unrolling:
A common trick exists in the case of a nested loop. You can unroll the outer loop, which causes multiple inner loops. You can then merge the bodies through loop *fusion*. This process is called unroll-and-jam

Global Code Placement:
The basic example of this is fall-through branches on asymmetric branch costs. Imagine a `if err != nil { .. }` statement. One would expect the `true` branch would happen far less frequently than the `false` branch (at least, I'd hope so). It follows that it is best to place the `false` branch as the fall-through branch, and leave the `true` branch as a jump. 