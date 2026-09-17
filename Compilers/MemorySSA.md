provide an SSA based form for memory, complete with def-use and use-def chains. 
memory SSA is intraprocedural, which means it is restricted to a single function or procedure at a time

`MemorySSA` contains a structure that maps `Instruction`s to `MemoryAccess`.
`MemoryAccess` can be one of 3 things:
- `MemoryDef`
- `MemoryPhi`
- `MemoryUse`

`MemoryDef`s either modify memory or introduce an ordering constraint. Examples:
- `Store`
- function calls
- volatile operations
- and more

`MemoryDef` introduces a new version of the entire memory
linked with a single `MemoryDef/MemoryPhi`. This means there is a single `Def` chain that connects to all other `Def`s. This results in (initially) every `MemoryDef` clobbers every other `MemoryDef`. LLVM uses something called The Walker to fix this.

`MemoryPhi` is a phi node for memory operations. If two (or more) definitions can flow into a basic block, the block's top `MemoryAccess` will be a `MemoryPhi`. Just the one, recall that we are interested in the entire state of memory as a single thing. Just as `Instruction` is mapped to `MemoryUse` and `MemoryDef`, a basic block is mapped to a `MemoryPhi`.

Note: Phi nodes merge may-reach definitions, rather than must-reach ones in traditional SSA

`MemoryUse` is an operation that uses but does not modify memory. Think `load` or a `readonly` function call

Since SSA needs definitions, there is a special `MemoryDef` called `liveOnEntry`. it dominates all accesses in the function that memory ssa is being run on, and implies we are at the top of the function. It just says that the memory is either undefined or defined before the function starts.


