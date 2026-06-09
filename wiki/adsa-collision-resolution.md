# Collision Resolution

**Summary**: When two keys hash to the same index a collision occurs. Two main strategies: chaining (external linked lists) and open addressing (probing within the table).

**Pre-Req**: [[adsa-hash-functions]], [[adsa-arrays]]

**Sources**: Session-built (no raw file); `Hashing-and-Hash-Tables.pdf` (Talenciaglobal).

**Last updated**: 2026-06-08

---

## Why Collisions are Unavoidable

Pigeonhole principle: if possible keys > table slots, some keys must share a slot. Collision handling is always required regardless of hash function quality.

---

## Strategy 1: Chaining

Each bucket holds a linked list of all entries that hash to that index.

```
Table:
[0] → (key1, val1) → (key4, val4) → None
[1] → (key2, val2) → None
[2] → None
[3] → (key3, val3) → (key5, val5) → None
```

- **Insert**: hash key → append to list at bucket. O(1) amortized.
- **Lookup**: hash key → scan list for key match. O(1) average, O(n) worst (all keys in one bucket).
- **Delete**: hash key → unlink node. O(1) average.

**Pros**: Simple, handles high load factors gracefully (α can exceed 1.0), deletion is clean (just unlink the node).  
**Cons**: ~24–40 bytes pointer overhead per entry, cache-unfriendly (linked list nodes scattered in memory).

**Enterprise upgrade — Java HashMap (Java 8+)**: when a chain exceeds **8 entries**, the linked list is automatically converted to a **Red-Black Tree**. This caps worst-case bucket lookup at O(log n) instead of O(n) — critical safety net under adversarial key collisions. (source: Hashing-and-Hash-Tables.pdf)

---

## Strategy 2: Open Addressing

All entries live inside the table itself — no external lists. On collision, probe for the next empty slot according to a **probe sequence**.

```
Table (size 8):
[0] _
[1] key2
[2] key1   ← h(key1) = 2, no collision
[3] key4   ← h(key4) = 2 also, probed to slot 3
[4] _
```

### Linear Probing
```
slot(i) = (h(key) + i) % n    for i = 0, 1, 2, ...
```
Step size is 1. Simplest but causes **primary clustering** — long filled runs form, slowing future probes for any key landing in that region.

### Quadratic Probing (Perturbation)
```
slot(i) = (h(key) + i²) % n    for i = 0, 1, 2, ...
```
Offsets: +1, +4, +9, +16... The growing offset **perturbs** the probe away from the cluster rather than walking through it step-by-step. Causes **secondary clustering** (keys with the same initial hash still follow the same probe path) but much less severe than linear.

### Double Hashing
```
slot(i) = (h1(key) + i * h2(key)) % n
```
Step size is key-dependent via a second hash function. Minimizes both primary and secondary clustering. Most cache-unfriendly of the three.

---

## Comparison

| Method             | Clustering      | Cache friendly | Deletion  | Max α  | Best use case              |
|--------------------|-----------------|----------------|-----------|--------|----------------------------|
| Chaining           | None            | Poor           | Easy      | > 1.0  | Frequent deletes, caches   |
| Linear probing     | Primary         | Excellent      | Tombstone | ≤ 0.7  | Read-heavy, cache-sensitive |
| Quadratic probing  | Secondary       | Good           | Tombstone | < 1.0  | General purpose            |
| Double hashing     | Minimal         | Moderate       | Tombstone | < 1.0  | High-throughput engines    |

(source: Hashing-and-Hash-Tables.pdf)

---

## Tombstones (Deletion in Open Addressing)

You cannot simply clear a slot on delete — it would break the probe chain for keys inserted after it.

Mark deleted slots with a special **DELETED** sentinel instead:

```
EMPTY   → slot never used; stop probing here
DELETED → was used; skip over it but keep probing
```

On insert, a DELETED slot can be reused. On lookup, treat it as occupied and continue probing.

#adsa
