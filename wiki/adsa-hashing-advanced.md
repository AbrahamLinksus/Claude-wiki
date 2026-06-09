# Advanced Hashing Techniques

**Summary**: Beyond basic chaining and linear probing — perfect hashing, cuckoo hashing, Robin Hood and hopscotch variants, concurrent hash maps, and how rehashing becomes a hidden production cost.

**Pre-Req**: [[adsa-hash-functions]], [[adsa-collision-resolution]]

**Sources**: `Hashing-and-Hash-Tables.pdf` (Talenciaglobal).

**Last updated**: 2026-06-08

---

## Perfect Hashing

A hash function with **zero collisions** for a known, static key set. Uses two levels:

- **Level 1**: Primary hash → `m` buckets (some collisions allowed)
- **Level 2**: Each bucket that has collisions gets its own secondary hash table sized `nᵢ²` where `nᵢ` = number of keys in that bucket. This quadratic sizing guarantees zero collisions within the bucket.

```
Total space: O(n)    Lookup: O(1) worst case (guaranteed)
```

Building it is expensive (O(n²) in the worst case at level 2), but once built it never changes.

**Use case**: static lookup tables that never change — ISO 4217 currency codes, country codes, fixed regulatory rule sets. Once built, guaranteed O(1) with no collision overhead ever. (source: Hashing-and-Hash-Tables.pdf)

---

## Cuckoo Hashing

Uses **two hash functions** and **two tables** (`T₁`, `T₂`). Every key has exactly two possible positions.

**Insert procedure**:
1. Compute `h₁(K)` and `h₂(K)` — two candidate positions
2. If either `T₁[h₁(K)]` or `T₂[h₂(K)]` is empty → place K there, done
3. If both occupied → **evict** the existing key X from one slot; X then tries *its* alternate location (cuckoo eviction)
4. If a cycle is detected during eviction → rehash with new functions

**Lookup**: check exactly 2 locations → **O(1) worst case** (not just average).

```
Lookup cost:  O(1) worst case — check T₁[h₁(key)] and T₂[h₂(key)] only
Insert cost:  O(1) amortized, but eviction cascade can be long in rare cases
```

**Use case**: hardware-accelerated network switches for payment routing — where guaranteed O(1) lookup is a hard requirement. (source: Hashing-and-Hash-Tables.pdf)

---

## Robin Hood Hashing

A refinement of linear probing that reduces variance in probe distances. The goal: every key's probe distance should be roughly equal.

**Rule**: when inserting key K at probe position `i`, if K's probe distance > the probe distance of the key already there (X), **swap** them. K takes the slot, X continues probing from where it got displaced.

> "Steal from the rich (keys close to ideal), give to the poor (keys far from ideal)."

**Result**: variance in probe distances is minimized → very consistent lookup times → predictable P99 latency.

**FinTech advantage**: latency-sensitive payment authorization paths with strict P99 SLAs (e.g., <5ms at 99th percentile). Standard linear probing has outlier lookups that blow P99; Robin Hood keeps them bounded. (source: Hashing-and-Hash-Tables.pdf)

---

## Hopscotch Hashing

Each bucket has a **neighborhood** — a bitmap of size H indicating which of the nearby H slots contain entries that hash to this bucket. Entries are always within H slots of their home bucket.

```
Neighborhood size H = 4
Bucket 5's neighborhood: slots 5, 6, 7, 8
Bitmap: [1, 0, 1, 0] → entries at slots 5 and 7 belong to bucket 5
Lookup: check only H slots → O(H) = O(1)
```

**Advantages**:
- Cache-friendly — all entries for a bucket fit in a bounded, nearby memory range
- Easy concurrent access — only the neighbourhood needs locking
- Particularly effective on modern CPUs with 64-byte cache lines

(source: Hashing-and-Hash-Tables.pdf)

---

## Concurrent Hash Maps

Standard hash tables are not thread-safe. Four approaches to concurrency:

### Global Lock
One mutex for the entire table. Simplest, but every operation blocks all others. Kills throughput at scale. Not recommended for production.

### Lock Striping
Divide the table into segments; each segment has its own lock. Java's `ConcurrentHashMap` pre-Java 8 used 16 segments. Good for moderate concurrency.

### CAS Lock-Free (Compare-And-Swap)
Use atomic CPU instructions instead of locks. Writers only retry on contention — no blocking. Used in ultra-low-latency trading systems. Java 8+ `ConcurrentHashMap` uses CAS for individual bucket operations.

### Read-Copy-Update (RCU)
Readers never block — they always see a consistent snapshot. Writers copy the affected portion, modify, then atomically swap the pointer. Ideal for read-heavy tables like fraud check blacklists.

**Production pattern**: for a payment gateway at 50K TPS with 32 worker threads, partition the hash table into 64 independent segments. Lock contention drops from O(threads) to O(threads/64). (source: Hashing-and-Hash-Tables.pdf)

---

## Rehashing — The Hidden Cost

Rehashing costs O(n) — all entries must be re-inserted into the new table. Amortized across n inserts, it's O(1) per insert. But the **instantaneous spike** is real.

**Problem**: rehashing a 100M-entry table takes seconds. During this time the table is locked or severely degraded — unacceptable on a payment authorization path.

### Incremental Rehashing (Redis-style)
Maintain **two tables** simultaneously during transition. Each operation migrates a few entries from the old table to the new one. The O(n) cost is spread across many requests, eliminating the spike.

### Pre-Sizing
If cardinality is known upfront, initialize at `expected_count / load_factor`:
```
100K-entry fraud list at α=0.75 → new HashMap<>(134_000)
```
This eliminates rehashing entirely. Alert in production if load factor exceeds 0.85 — a resize event will cause a measurable P99 latency spike. (source: Hashing-and-Hash-Tables.pdf)

---

## History

| Year | Milestone |
|------|-----------|
| 1953 | Hans Peter Luhn (IBM) — first described hashing with chaining |
| 1973 | Carter & Wegman — Universal hashing: theoretical foundation for randomized hash families |
| 1981 | Fredman et al. — Perfect Hashing: O(1) worst case with O(n) space |
| 2001 | Pagh & Rodler — Cuckoo Hashing: worst-case O(1) lookup with open addressing |
| 2018+ | Google Abseil Swiss Table — SIMD-accelerated flat hash map, now industry standard |
| 2024+ | Current state: AES-NI hardware acceleration, BLAKE3 integrity, Rust's hashbrown |

#adsa
