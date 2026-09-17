## SSA out of translation problems:

There are largely two concerns when generating copy or move instructions for out of SSA translation blindly. The Lost-Copy Problem and the Swap Problem. The lost copy problem is when a copy destroys a live variable before it is used. This typically results after an aggressive copy propagation pass, consider the following:

```
Block 1:
	x0 = 0
Block 2:
	x1 = Φ(x0, x2)
	y = x1
	x2 = x1 + 1;
	// loop 3 times

Block 3:
	// use y; y = 2
```

A copy propagation pass may realize that `y` can be replaced with `x2`, which results in:

```
Block 1:
	x0 = 0
Block 2:
	x1 = Φ(x0, x2)
	x2 = x1 + 1
	// loop three times

Block 3:
	// use x2; x2 = 3
```

This would make sense for a copy propagation pass. However, if we aren't careful and blindly place copy instructions, we'll use the final value than the expected penultimate value.

Another problem is the swap problem. In SSA, $\phi$-functions at the beginning of a basic block are expected to be executed in parallel. For example, imagine a loop that repeatedly swaps the values `x` and `y`:

 ```
Block 1:
	x0 = ...
	y0 = ...
	 
Block 2:
	x1 = Φ(x0, y1)
	y1 = Φ(y0, x1)
	// loop
	
Block 3:
	// use x1
	// use y1
 ```

This example could pop up after a copy propagation optimization pass. Let's place copy instructions:

```
Block 1:
	x0 = ...;
	y0 = ...;
	x1 = x0;
	y1 = y0
	
Block 2:
	x1 = y1
	y1 = x1
	// loop
	
Block 3:
	// use x1
	// use y1
```

Since the $\phi$-functions are no longer done in parallel but rather sequentially, we see that we actually have the incorrect value for `y1`, since we clobber the old value of `x1` by swapping it with `y1`.

The simple fix for the lost copy problem is to split *critical edges*, that is, edges with both multiple predecessors and multiple successors. We must split this edge and place a new block that houses all the copy instructions. However, we may not always be able to split a critical edge. In that case, we can add an extra variable to preserve the old value before terminating the loop (as was the case in the original example).

The swap problem has a very similar fix, we generate an extra temporary variable for $\phi$-functions that rely on the outputs of other $\phi$-functions, and we use those for copying instead. However, this generates twice the amount of copy instructions, and is not ideal.

In summary, blindly placing copy instructions can erroneously extend the range in which a variable is live. Alternatively, replacing the parallel $\phi$-functions with sequential copy instructions can cause values to be clobbered.