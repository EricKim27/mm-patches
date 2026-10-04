# __slab_free() case handling

[link](https://lore.kernel.org/linux-mm/179000339670.55983.14009373005903071416.b4-ty@b4/T/#m0258d720bdfbbb3d7a0ac1ef5864a9fa4e557652)

### Context

__slab_free() function handles freeing objects in slab, while dealing with it being partial, full, or empty.

### The problem

The current implementation of __slab_free() fails to deliver all the handling cases of slab.

### Solution

The patch aims to add handling cases for all slab cases:

 - a. partial->partial, offlist
 - b. partial->partial, onlist
 - c. partial->empty, offlist
 - d. partial->empty, onlist, exceeding min_partial
 - e. partial->empty, onlist, not exceeding min_partial
 - f. full->empty, exceeding min_partial
 - g. full->empty, not exceeding min_partial
 - h. full->partial

The revised __slab_free() checks first for full->partial case which would not get the node in case, update the freelist and return. This return path is used also when get_node() returns nothing. 
It would then test if the slab is partial(if not, empty or full), and return early if it was not also full. It would then remove partial and discard slab if it is empty and exceeds minimum. At that point, it would check if it was full before, which would have to refill the slab if so. Otherwise, it would do nothing and let the slab stay on the partial list.