## Discourse to remove dirty_folio()

[Link](https://lore.kernel.org/linux-mm/20260826050733.GB15122@lst.de/T/#t)

The original purpose of dirty_folio() is to write back data in memory to its block device when it is modified. The most ideal process goes like writing to a write-protected page, page faults, write it and mark dirty if write is allowed, and write back later.

This would work when memory access is only done by kernel. The assumption breaks down when DMA happens. DMA uses GUP to pin/get pages for dma usage, but does not page faults(as dma could take indefinite amount of time). Modern filesystems allocate blocks at the very last moment(delayed), but external devices doing DMA is not aware of that. Initially GUP makes filesystems to reserve blocks for DMA writeback, but since DMA could take long time, it cannot account for writeback blocks’ integrity.

During DMA, jobs like truncation could alter the file structure and the block to writeback may be invalid. Since GUP does not really enforce blocks and memory, any job can happen during writeback, like checksum being calculated and write job occurs, which would corrupt.
