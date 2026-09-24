## Refill prefilled sheaves from the barn

[link](https://lore.kernel.org/linux-mm/20260918114318.124346-1-hao.li@linux.dev/)

### Problem

When a reclaimable rcu sheaf is trying to go to its node barn to be reused, it has to be flushed to slab when full sheaves list is full.

### Solution

The proposed patch introduces partial sheaf, and refilling process refills utliizing partial and full sheaves. When refilling target sheaves(partial or empty one) from node barns, it would fill first from full sheaves list if there are no partial(so partial list needs some) and put that sheaf to partial. If there is a sheaf in the partial, if both combined reaches capacity of one sheaf, one with more objects get filled, and full sheaf goes to full list, while other one goes to partial or empty. If it doesn’t, fill it up to the capacity with sheaf from full list then put that into full list and put the used full sheaf to partial.
