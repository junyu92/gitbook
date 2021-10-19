# Memory Layout

## MemoryRegion

It is a tree strucutre.

```
                    MemoryRegion
                   +---------------+
                   | addr          |
                   +---------------+
                   | size          |
                   +---------------+
                   | subregions    |
                   +---------------+
```

There are multiple types of MemoryRegions:

* RAM: a range of host memory that can be made available to the guest.
* MMIO: a range of guest memory that is implemented by host callbacks;
  each read or write causes a callback to be called on the host.
* ROM: a ROM memory region works like RAM for reads, and forbids writes.
* ROM device: a ROM device memory region works like RAM for reads, but
  like MMIO for writes.
* IOMMU region: an IOMMU region translates addresses of accesses made to
  it and forwards them to some other target memory region.
* container: a container simply includes other memory regions, each at
  a different offset. Containers are useful for grouping several regions
  into one unit. For example, a PCI BAR may be composed of a RAM region
  and an MMIO region.
* alias: a subsection of another region.
* reservation region: a reservation region is primarily for debugging.

```c
struct MemoryRegion {
    Object parent_obj;

    /* private: */

    /* The following fields should fit in a cache line */
    bool romd_mode;
    bool ram;
    bool subpage;
    bool readonly; /* For RAM regions */
    bool nonvolatile;
    bool rom_device;
    bool flush_coalesced_mmio;
    uint8_t dirty_log_mask;
    bool is_iommu;
    RAMBlock *ram_block;
    Object *owner;

    const MemoryRegionOps *ops;
    void *opaque;
    MemoryRegion *container;
    Int128 size;
    hwaddr addr;
    void (*destructor)(MemoryRegion *mr);
    uint64_t align;
    bool terminates;
    bool ram_device;
    bool enabled;
    bool warning_printed; /* For reservations */
    uint8_t vga_logging_count;
    MemoryRegion *alias;
    hwaddr alias_offset;
    int32_t priority;
    QTAILQ_HEAD(, MemoryRegion) subregions;
    QTAILQ_ENTRY(MemoryRegion) subregions_link;
    QTAILQ_HEAD(, CoalescedMemoryRange) coalesced;
    const char *name;
    unsigned ioeventfd_nb;
    MemoryRegionIoeventfd *ioeventfds;
};
```

## Flat View

As its name implies, `FlatView` is a sorted memory hierarchy.

```c
/* Flattened global view of current active memory hierarchy.  Kept in sorted
 * order.
 */
struct FlatView {
    struct rcu_head rcu;
    unsigned ref;
    FlatRange *ranges;
    unsigned nr;
    unsigned nr_allocated;
    struct AddressSpaceDispatch *dispatch;
    MemoryRegion *root;
};
```

```
 FlatView
+----------+
| nr       |
+----------+
| ranges   | -> +------------------+ FlatRange[0]
+----------+    | mr               |
                +------------------+
		| offset_in_region |
                +------------------+
		| addr             |
		+------------------+
		| ...              |
		+------------------+ FlatRange[..]
```

## AddressSpace

```c
struct AddressSpace {
    /* private: */
    struct rcu_head rcu;
    char *name;
    MemoryRegion *root;

    /* Accessed via RCU.  */
    struct FlatView *current_map;

    int ioeventfd_nb;
    struct MemoryRegionIoeventfd *ioeventfds;
    QTAILQ_HEAD(, MemoryListener) listeners;
    QTAILQ_ENTRY(AddressSpace) address_spaces_link;
};
```

```
 AddressSpace

+-------------------------------+
| MemoryRegion *root            |
+-------------------------------+
| struct FlatView *current_map  |
+-------------------------------+
```

## Update memory_region

Here is a standard scheme to update MemoryRegion.

```c
void pci_bridge_update_mappings(PCIBridge *br)
{
    memory_region_transaction_begin();

    // do something

    memory_region_transaction_commit();
}
```

### memory_region_transaction_begin

```c
void memory_region_transaction_begin(void)
{
    qemu_flush_coalesced_mmio_buffer();
    ++memory_region_transaction_depth;
}
```

### memory_region_transaction_commit

```c
void memory_region_transaction_commit(void)
{
    AddressSpace *as;

    assert(memory_region_transaction_depth);
    assert(qemu_mutex_iothread_locked());

    --memory_region_transaction_depth;
    if (!memory_region_transaction_depth) {
        if (memory_region_update_pending) {
	    // re-gerenate memory topology
            flatviews_reset();

            MEMORY_LISTENER_CALL_GLOBAL(begin, Forward);

            QTAILQ_FOREACH(as, &address_spaces, address_spaces_link) {
                address_space_set_flatview(as);
                address_space_update_ioeventfds(as);
            }
            memory_region_update_pending = false;
            ioeventfd_update_pending = false;
            MEMORY_LISTENER_CALL_GLOBAL(commit, Forward);
        } else if (ioeventfd_update_pending) {
            QTAILQ_FOREACH(as, &address_spaces, address_spaces_link) {
                address_space_update_ioeventfds(as);
            }
            ioeventfd_update_pending = false;
        }
   }
}
```

```
memory_region_transaction_commit
|- flatviews_reset
   |- flatviews_init: create a global hash table to cache flatview infomation
   |- generate_memory_topology: render a root MemoryRegion's topology into a list of disjoint absolute ranges
      |- render_memory_region: recursively create MemoryRegion and insert into FlatView
      |- flatview_simplify: normalize flatview
|- address_space_set_flatview: update FlatView for AddressSpace
   |- address_space_update_topology_pass: compares old flatview and new flatview and invokes callback
```