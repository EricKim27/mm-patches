## RFC: Take limits per type when shrink_node()

RFC: Request for Comments; the patch wants early feedback and discussion.

[link](https://lore.kernel.org/linux-mm/de7d583e-bd9b-464f-9e28-73075c0211a4@linux.dev/T/#mdd1fc0b168c86a6a91bcdbecf6bd1fe04e378272)

### Background/Problem

shrink_node() is called either when reclaimable slabs OR unmapped page caches are above minimal number, but doesn’t take into consideration types, and enters shrink_slab() when it is below minimum amount of reclaimable slabs, or attempt to shrink unmapped page caches when the case is reversed.

### Fix

The first patch introduces variable skip_slab_reclaim as a member of scan_control, that gates whether to shrink slab or not by checking if the amount of reclaimable slabs are below minimum. Becomes true if so, and false if not. shrink_slab() is called when it is false(reclaimable slabs are above average).

The second patch introduces skip_file_reclaim as a member of scan_control. Similar to the first patch, it determines whether to skip file reclamation when reclaimable page caches are below minimum. Becomes true if so, false if not. 

It checks when shrink_lruvec is called, which checks skip_file_reclaim, and when file page cache is below minimum, checks if anon_pages are possible for reclamation, and determine whether to reclaim from anon or return early if even anon is not available. 

Additionally this check happens when it is evicting folios. When scanning, if the type of folio is file and skip_file_reclaim is true, it does not evict from that folio.

The third patch removes check for minimum patch when calling shrink_node() as it has moved into shrink_node() function to check, and is quite dangerous as it frees both even when one of those two is possible for reclamation.

The fourth patch adds additional condition when bailing out in node_reclaim(), to take consideration of anonymous pages to see if that is reclaimable and bail out if even anonymous are not available.
