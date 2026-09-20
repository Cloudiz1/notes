### LVN
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

### Tree height balancing:
having a left or right associative tree for a long list of additions signals to the compiler that you have to do it in that exact order (say, left-to-right for the expression: `a+b+c+d+e+f+g+h`). However, this can be parallelized. If we have a balanced tree, we can see that we can compute `a+b` and `g+h` simultaneously, for example.

This optimization only works for operations that are commutative and associative, and only works for expressions that only contain the same operation.

### Loop unrolling:
Just to cover the basics, loop unrolling is essentially doing fewer iterations of *more* work to remove the overhead of loops (jumping, increment, conditions). A simple example is as follows:
```C
int a[32];
for (int i = 0; i < 32; i++) {
	a[i] = i;
}
```
can become:
```C
int a[32]
for (int i = 0; i < 32; i += 4) {
	a[i] = i;
	a[i + 1] = i + 1;
	a[i + 2] = i + 2;
	a[i + 3] = i + 3;
}
```

In my head (this is not in Engineering a Compiler and this really exists as a reminder to myself to go research it) a loop can be unrolled if the live ranges do not overlap, that is nothing from one iteration of the loop interferes with anything else from the loop. 

A common trick exists in the case of a nested loop. You can unroll the outer loop, which causes multiple inner loops. You can then merge the bodies through loop *fusion*. This process is called unroll-and-jam

### Global Code Placement:
The basic example of this is fall-through branches on asymmetric branch costs. Imagine a `if err != nil { .. }` statement. One would expect the `true` branch would happen far less frequently than the `false` branch (at least, I'd hope so). It follows that it is best to place the `false` branch as the fall-through branch, and leave the `true` branch as a jump. 

One way to weigh these branches, is by profiling the branches, generating weights for each branch, and then greedily constructing a *hot path* in the CFG. 

### Inline substitution:
When you inline a function!
Pay attention to the *proliferation* of local variables, that is, make sure there aren't too many of them. This can hurt the performance of the final code *and* slow down the compilation process. I believe I saw somewhere this can also mess with cache locality if done impropely. 

There are definitely lots of criteria to watch out for, including:
- Callee size: if the callee is smaller than the procedure linkage code, then we inlining the code should be beneficial
- Caller size: we may want to limit the overall size of a procedure (for aformentioned reasons)
- Constant valued actual parameters: of course, perform constant folding through functions too!
- Static call count: the number of times in the code we are calling the function. Inlining too many times can cause really large binaries (which, of course, is a metric we are optimizing for too).

We have a few metrics for the cost of procedure linkage as well:
- Parameter count: storing parameters takes time and resources!
- Calls in the proc: track the number of calls in them, it becomes trivial to find leaves in a call graph, which are usually beneficial to inline 
- Loop nesting depth: optimizing a call inside a loop is more beneficial than one outside (for the reason why loop optimizations are so important)

We can, again, use profiling to get better heuristics! We can choose to inline functions that run very very often (referred to as dynamic call count) or just get a percentage of overall execution time!

### Procedure Placement
We can also discuss a notion of optimizing the placement of procedures themselves. If a procedure $p$ calls procedure $q$, we should place these procedures adjacent to one another within the binary. Of course, this is not always possible (if $p$ calls three procedures, we can not place three procedures adjacent to $p$ in a linear, 1-d, way), so we opt for a greedy algorithm that is good *enough*. The main reason is due to paging. If we place the procedures physically close to each other, it becomes more likely that $p$ and $q$ live on the same page, and we won't have to fetch a whole page to call $q$. I believe this can also reduce the size of the final binary by optimizing for short jumps. 

Namely, "if $p$ calls $q$ and the distance from $p$ to $q$ is less than the size of the instruction cache, placement succeeds." We can take this one step further and optimize for weights (either profiled or estimated). The greedy algorithm consists of a priority queue sorted by weights!

read the chapter notes of chapter 8 for further reading!

### Frame pointer omission and stack frame omission
Briefly, defintions:
a frame pointer is a base reference for your stack frame, so, `rbp`.
omitting a frame pointer emits the prologue and epilogue for setting up `rbp`, it does the following transformation:
```asm
foo:
	push rbp
	mov rbp, rsp
	sub rsp, ... ; if you have local variables
	
	...
	
	leave
	ret
```
into:
```asm
foo:
	sub rsp, ... ; if you have local variables
	ret
```
and we do things with reference to `rsp` instead. Not only does this remove 3 instructions, it also allows us to use `rbp` as a general purpose register for math again!

omitting the `sub rsp, ...` is known as omitting the stack frame. Intuitively, this can be done if we don't use any local variables. However there is definitely more nuance to this. SystemV requires that functions are aligned to 16 bytes (this is good practice, following ABI or not). If you already need to decrement for local variables, ensure this decrement is aligned to 16 bytes, this will give you the alignment for free!