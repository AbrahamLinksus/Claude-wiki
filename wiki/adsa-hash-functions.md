# Hash Functions

**Summary**: A hash function transforms an arbitrary input into a fixed-range integer index. Five key properties separate a good one from a bad one.

**Pre-Req**: [[adsa-arrays]] — indexed storage and O(1) random access

**Sources**: Session-built (no raw file); `Hashing-and-Hash-Tables.pdf` (Talenciaglobal).

**Last updated**: 2026-06-08

---

## The Key-Space Compression Problem

A **Direct Address Table** would give perfect O(1) with zero collisions — just use the key itself as the index: `T[key] = value`. This works when keys are integers in a small range. The problem: a 16-digit card number has a universe of 10¹⁶ values — physically impossible to allocate 10¹⁶ slots.

A hash function solves this by compressing the key space:

```
Key Input → Hash Function → Compression (% m) → Bucket Array
```

A table of `m` buckets can handle a universe of any size at the cost of occasional collisions.

## What is a Hash Function?

A hash function `h(k)` takes a key `k` and produces an integer in range `[0, n-1]` where `n` is the table size:

```
index = h(key) % table_size
```

The goal: map arbitrary keys (strings, ints, objects) to array indices for direct O(1) access. (source: Hashing-and-Hash-Tables.pdf)

---

## 5 Characteristics of a Good Hash Function

### 1. Deterministic
Same input must always produce the same output. Without this, a lookup could never find what was inserted.

### 2. Uniform Distribution
Outputs should spread evenly across `[0, n-1]`. Clustering in one region causes more collisions and degrades performance.

```
Bad:  h("a")=0, h("b")=0, h("c")=1   ← clustered
Good: h("a")=0, h("b")=3, h("c")=7   ← spread
```

### 3. Fast to Compute
O(1) preferred. If hashing itself takes O(k) it defeats the purpose of the data structure.

### 4. Avalanche Effect
A small change in input should cause a large, unpredictable change in output. This prevents similar keys from landing in the same region of the table.

```
Without avalanche: h("cat")=5, h("bat")=6    ← always adjacent
With avalanche:    h("cat")=5, h("bat")=71   ← scattered
```

### 5. Low Collision Rate
No hash function avoids all collisions (pigeonhole principle — more keys than slots guarantees some collisions), but a good one minimizes them across realistic inputs.

---

## Common Hash Functions

### Division method
```
h(k) = k % n
```
Simple and fast. **Pitfall**: if `n` is a power of 2, only the lower bits of `k` are used — sequential keys like ACC-001, ACC-002 cluster badly. **Best practice**: choose `n` as a prime not close to a power of 2. (source: Hashing-and-Hash-Tables.pdf)

### Multiplication method
```
h(k) = floor(n * (k * A mod 1))    where A ≈ 0.6180339887 (golden ratio — Knuth's recommendation)
```
Works well regardless of `n`. Slightly slower than division but better distribution.

### Universal Hashing
```
h(key) = ((a × key + b) mod p) mod m
```
Where `p` is a large prime, `a ∈ [1, p-1]`, `b ∈ [0, p-1]` chosen **randomly at startup**. Provably low collision probability for any input set. Critical for security: prevents **HashDoS attacks** where an adversary crafts inputs that all hash to the same bucket, causing O(n) degradation. Java 8+ uses a per-process random seed for this reason. (source: Hashing-and-Hash-Tables.pdf)

### Polynomial rolling hash (strings)
```
h(s) = s[0]*p^(m-1) + s[1]*p^(m-2) + ... + s[m-1]   mod n
```
`p` is typically a small prime (31 or 37). "Rolling" update in O(1) when a window slides — used in [[rabin-karp]] and fraud narration scanning.

---

## Load Factor

```
α = entries in table / table size
```

- α < 0.7 → collision probability stays low
- Python dict resizes when α > 2/3 (~0.67) → see [[adsa-python-dict]]
- Java HashMap resizes at α > 0.75

A high load factor means more entries per bucket, pushing average lookup toward O(n).

#adsa
