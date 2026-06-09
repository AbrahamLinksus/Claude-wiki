# Python Dict Internals

**Summary**: How Python's dict achieves O(1) average-case operations, why it beats a dynamic array of pairs at every operation, and how power-of-2 sizing replaces modulo with a single bitwise AND.

**Pre-Req**: [[adsa-hash-functions]], [[adsa-collision-resolution]], [[adsa-arrays]]

**Sources**: Session-built (no raw file); `Hashing-and-Hash-Tables.pdf` (Talenciaglobal).

**Last updated**: 2026-06-08

---

## Dict vs Dynamic Array of Pairs

Suppose you store key-value data as a list of pairs: `[(k1,v1), (k2,v2), ...]`.

| Operation | Unsorted list      | Sorted list          | Hash table (dict)   |
|-----------|--------------------|----------------------|---------------------|
| Lookup    | O(n) linear scan   | O(log n) binary search | **O(1) avg**       |
| Insert    | O(1) amortized     | O(n) shift to maintain order | **O(1) avg** |
| Delete    | O(n) scan + shift  | O(n) shift           | **O(1) avg**        |

**Unsorted list lookup** — scan every pair until the key matches. No shortcut.

**Sorted list lookup** — binary search gives O(log n), but keeping sorted order costs O(n) per insert due to element shifting.

**Dict lookup** — `hash(key)` computes the bucket index in O(1). One arithmetic step, one memory read. No traversal, no comparison loop.

---

## Why "Almost" O(1)?

Worst case is O(n) if all keys land in the same bucket. Python's design prevents this in practice:

1. **Load factor cap** — dict resizes when more than 2/3 full, keeping collision probability low.
2. **Strong hash functions** — Python's built-in `hash()` has good avalanche properties.
3. **Compact layout (Python 3.6+)** — entries are stored in insertion order in a dense array; the hash table holds integer indices into it. The hot path is cache-friendly.

---

## The Bitwise Modulo Trick

To map a hash to a bucket index:
```
index = hash(key) % table_size
```

Integer division/modulo takes several CPU cycles. Python avoids it by always keeping `table_size` as a **power of 2**.

When `n` is a power of 2, modulo collapses to a bitwise AND:
```
hash % n  ==  hash & (n - 1)
```

**Why this works** — powers of 2 in binary are a single 1-bit followed by zeros. Subtracting 1 flips all lower bits to 1:
```
n   = 8  →  0b00001000
n-1 = 7  →  0b00000111
```
ANDing any number with `0b0111` keeps only the last 3 bits — exactly what `% 8` computes. Bitwise AND is a single-cycle instruction on every CPU architecture.

**Concrete example**:
```
hash(key) = 181  →  0b10110101

n   = 8   →  0b00001000
n-1 = 7   →  0b00000111

181 & 7  =  0b00000101  =  5
181 % 8                 =  5  ✓
```

**Trade-off**: Power-of-2 sizes mean the table jumps 8 → 16 → 32 → 64... You may over-allocate up to ~50% of slots. Python accepts this memory cost for the speed gain.

---

## Resize Behavior

Python dict doubles in size (always staying a power of 2) when load factor exceeds 2/3:

```
Initial size 8  → resize at 6 entries   → new size 16
Size 16         → resize at 11 entries  → new size 32
Size 32         → resize at 22 entries  → new size 64
```

After resize, every entry is **rehashed** into the new table — O(n) one-time cost, amortized O(1) per insert. Same analysis as [[adsa-arrays]] dynamic array doubling.

---

## High-Bit Spreading

Power-of-2 tables have a subtlety: `hash & (n-1)` only uses the *lower* bits. If hash values differ only in high bits, they collide. Python applies a mixing step before masking:

```python
h ^= (h >> 16)    # XOR high bits down into low bits
index = h & (capacity - 1)
```

This spreads information from upper bits into the lower region used for indexing — critical for sequential keys like account IDs. (source: Hashing-and-Hash-Tables.pdf)

---

## Language Comparison

| Feature              | Java HashMap                     | Python dict             | C++ unordered_map        |
|----------------------|----------------------------------|-------------------------|--------------------------|
| Collision strategy   | Chaining (list → RBT at 8 entries) | Open addressing + perturbation | Chaining (linked list) |
| Default load factor  | 0.75                             | ~0.67 (2/3)             | 1.0                      |
| Resize factor        | 2×                               | 4× (small) or 2× (large) | ~2×                     |
| Ordering             | None                             | Insertion order (3.7+)  | None                     |
| Thread safe          | No (use ConcurrentHashMap)       | No (GIL helps, not safe) | No                      |
| Null/None keys       | 1 null allowed                   | None as key is valid    | N/A                      |

**Java pitfall**: if you override `equals()` without overriding `hashCode()`, two objects that are logically equal will hash to different buckets — one of the most common production bugs in Java systems. Always override both. (source: Hashing-and-Hash-Tables.pdf)

#adsa
