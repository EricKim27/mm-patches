## Fix workingset refaults

[link](https://lore.kernel.org/linux-mm/20260821093606.2231216-1-alex@ghiti.fr/T/#me80aa7df7d624ba3710ce9a7cf72f00063866d32)

### Background

For swap, swap cache is amongst the first place to look at when looking for pages that have been marked as swapped out. There are two cases of looking this up; when target page is not present, or being reclaimed by other process. The first case allocates folio to get data from swap and associates with the swap cache. The second case is occured when first case is already taking place. It gets that folio and wait until data is availablwe.

There are in-memory caches in linux system, like dentry, inode, DMA buffer, zswaps and more. These objects are to be swapped out when memory is under pressure. Shrinker keeps track of objects that it can ask subsystem to free, to make more free space in memory in this case.

### Problem

- when anonymous folio is called and swapped out, workingset_eviction() stores shadow at swap slot(eviction cookie) for cases where ram requires that page later, which would activate swapped out folio, if refault(swapped out memory being required in a short amount of time) distance is short enough.
- zswap writeback(writing zswap ram pool data to swap) breaks this in two independent ways.
    - First, when shrinker allocates buffer folio in swap cache, the allocation path counts that as a refault.
    - Second, adding that buffer to swap cache overwrites that slot’s shadow, and original eviction cookie gets lost. When buffer folio reclaimed, inaccurate cookie is at place.

### Proposed fix

- the fix keeps shadow within zswap itself, no swap table, and the shadows are inside the zswap tree instead of erasing the freed entry.
- refault evalution moves out of swap cache allocator and into the swap in caller, meaning the refault would be determined when the page is actually reclaimed. This would rule out allocation of writeback buffer from being considered a refault.
