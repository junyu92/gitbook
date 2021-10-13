# Memory Model

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