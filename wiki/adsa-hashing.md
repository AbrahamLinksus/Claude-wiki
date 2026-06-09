# Hashing

**Summary**: Overview of hash tables — how they achieve O(1) average lookup, the hash function, and collision resolution strategies.

**Sources**: Session-built; `Hashing-and-Hash-Tables.pdf` (Talenciaglobal).

**Last updated**: 2026-06-08

---

- [[adsa-hash-functions]] — Key-space compression, 5 characteristics, division/multiplication/Universal hashing
- [[adsa-collision-resolution]] — Chaining (+ Java RBT upgrade), linear/quadratic/double hashing, tombstones
- [[adsa-python-dict]] — O(1) vs array of pairs, bitwise AND trick, high-bit spreading, language comparison table
- [[adsa-hashing-advanced]] — Perfect hashing, Cuckoo hashing, Robin Hood, Hopscotch, concurrent maps, incremental rehashing
- [[adsa-hashing-applications]] — FinTech use cases (dedup/fraud/KYC), 8 interview patterns, security (HashDoS/passwords), architectural patterns

## Why O(1)?

A hash table maps a key directly to a bucket index via `hash(key) % table_size`. Access is a fixed-cost arithmetic step, not a traversal. Collisions degrade this, but a good hash function and low load factor keep them rare.

## Roadmap

1. Understand the hash function and its properties → [[adsa-hash-functions]]
2. Learn how collisions are handled → [[adsa-collision-resolution]]
3. See how Python applies these ideas in its dict → [[adsa-python-dict]]
4. Advanced variants and concurrent maps → [[adsa-hashing-advanced]]
5. Real applications and patterns → [[adsa-hashing-applications]]

#adsa
