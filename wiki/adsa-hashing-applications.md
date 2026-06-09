# Hashing — Applications & Patterns

**Summary**: Real-world use cases (FinTech scale), algorithmic interview patterns, security considerations, and architectural patterns built on hash tables.

**Pre-Req**: [[adsa-hashing]], [[adsa-hash-functions]], [[adsa-collision-resolution]]

**Sources**: `Hashing-and-Hash-Tables.pdf` (Talenciaglobal).

**Last updated**: 2026-06-08

---

## Why Hash Tables at Scale

At 50K TPS (transactions per second), the difference between O(1) and O(log n) is the difference between 200μs and 2μs per lookup — a 100× gap. Binary search on a 200M-entry sorted fraud list requires ~27 comparisons and cannot meet 100ms authorization SLAs under concurrent load. O(n) linked list traversal is simply non-viable. (source: Hashing-and-Hash-Tables.pdf)

---

## FinTech Use Cases

### Use Case 1 — Transaction Deduplication (UPI/IMPS)

India's UPI processes 500M+ daily transactions. Network retries can send the same payment twice within milliseconds. Processing it twice = double debit = critical RBI regulatory violation.

**Architecture**:
```
Payment Gateway
  → Deduplication Layer (ConcurrentHashMap<String, Long>)
      Key: SHA-256(sender_id | receiver_id | amount | idempotency_token)
      TTL: 5 minutes (sliding window)
  → Core Banking System
```

**Flow**:
1. Compute dedup key from transaction fields
2. If key EXISTS in HashSet → REJECT as duplicate
3. If NOT found → INSERT with TTL, proceed to processing

**Outcome**: duplicate rate < 0.001%, meets RBI idempotency mandate, O(1) lookup handles 500M+ daily volume. (source: Hashing-and-Hash-Tables.pdf)

---

### Use Case 2 — In-Memory Fraud Screening

A card network maintains 200M+ compromised card numbers. Every authorization (~10K TPS) must be checked within 50ms.

**Two-tier architecture**:
1. **Bloom Filter** (probabilistic pre-check) — ultra-fast, reduces hash table lookups by 40%
2. **Hash Table** (deterministic confirmed check) — HashSet<Long>, 200M entries, ~3.2GB RAM, 0.5μs lookup

Card numbers stored as SHA-256 hashes — never plaintext — satisfying PCI-DSS requirement of not storing raw PANs.

**Key metrics**: 200M entries, 3.2GB RAM, 0.5μs lookup latency. (source: Hashing-and-Hash-Tables.pdf)

---

### Use Case 3 — LRU Cache for KYC Verification

KYC verification via UIDAI/CKYC APIs costs ~₹2–5 per call. At 100K loan applications/day, redundant calls waste ₹2–5 lakh daily and add 2–3 seconds latency per application.

**LRU Cache design — HashMap + Doubly Linked List** (O(1) for both GET and PUT):

```
HashMap<AadhaarHash, DLL Node pointer>   → O(1) lookup
Doubly Linked List                        → maintains recency order
    HEAD = most recently used
    TAIL = least recently used (eviction candidate)
```

- **GET**: HashMap lookup O(1) → move node to HEAD
- **PUT**: Insert at HEAD → if at capacity, evict TAIL node. Both O(1).
- Cache entries carry a **24-hour TTL** — KYC status can change and must be periodically re-verified.

**Outcomes**: 65% cache hit rate, ₹1.5L+ daily savings, 50ms cached latency vs 3 seconds for uncached API call. (source: Hashing-and-Hash-Tables.pdf)

---

## Algorithmic Patterns

| Pattern              | Structure                        | Complexity         | Example problem                          |
|----------------------|----------------------------------|--------------------|------------------------------------------|
| Value-to-Index Map   | `HashMap<V, idx>`                | O(n)               | Two Sum, finding settlement pairs        |
| Frequency Map        | `HashMap<K, int>`                | O(n) time, O(k) space | Top-K merchants, MCC distribution    |
| Seen Set             | `HashSet<K>`                     | O(n)               | First duplicate transaction, cycle detection |
| Canonical Key        | Transform input → canonical form | O(n)               | Group anagrams, normalize IFSC codes     |
| Prefix Sum + Map     | `HashMap<sum, count>`            | O(n)               | Subarrays with target sum (batch reconciliation) |
| Sliding Window + Map | `HashMap<K, int>` in window      | O(n)               | Count distinct merchants in 30-day window |
| Two Maps             | One map per direction            | O(n)               | Isomorphic strings, word pattern matching |
| Design with HashMap  | HashMap as core of a structure   | O(1) per op        | LRU Cache, LFU Cache, Insert-Delete-GetRandom O(1) |

(source: Hashing-and-Hash-Tables.pdf)

---

## Security Considerations

### HashDoS Attack
An adversary crafts inputs that all hash to the same bucket → every lookup becomes O(n) → CPU exhaustion. **Mitigation**: use Universal Hashing or SipHash with a random per-process seed. Java 8+ and Python 3.3+ both randomize hash seeds at startup. (source: Hashing-and-Hash-Tables.pdf)

### Timing Side-Channel
An attacker infers whether a key exists by measuring response time variation — standard comparison exits early on mismatch. **Mitigation**: use constant-time comparison for security-sensitive lookups (API key validation, HMAC verification).

### Memory Dump Exposure
Plaintext keys and values are visible in heap dumps or core dumps. **Mitigation**: encrypt sensitive values in-memory; use off-heap storage; wipe on deallocation.

### Password Storage — Done Right

Plain `SHA-256("password")` is wrong — a GPU can try **10 billion hashes per second**, making rainbow table attacks trivial. Adding a salt helps but is still insufficient without computational cost.

```python
# Wrong — too fast
stored = SHA256("user_password")
stored = SHA256(salt + "user_password")

# Correct
stored = bcrypt("user_password", cost=12)        # ~250ms per hash
stored = Argon2id("user_password", time=3, memory=64MB)  # memory-hard, defeats GPU parallelism
```

Key properties of correct password hashing:
- **Designed to be slow** (~250ms) — makes brute force computationally infeasible
- **Memory-hard** (Argon2id) — requires large RAM per attempt, defeats GPU parallelism
- **Per-user random salt** — each password has a unique salt, defeats precomputed rainbow tables

(source: Hashing-and-Hash-Tables.pdf)

---

## Architectural Patterns

### Cache-Aside
Application checks HashMap/Redis first → on miss, query DB → populate cache. Reduces DB load 60–80% for hot account data in trading engines.

### Idempotency Key
Store request ID in HashSet → reject duplicates. Mandatory for RBI-compliant payment aggregators. Enables exactly-once semantics across network retries.

### Token Bucket Rate Limiting
`HashMap<client_id, {tokens, last_refill}>` — O(1) per request. Enforces API rate limits without database overhead.

### Consistent Hashing
Distribute keys across nodes using a hash ring. When nodes are added or removed, only a fraction of keys need to be remapped (vs full rehash). Used in Redis Cluster and Memcached for near-zero disruption during scaling. (source: Hashing-and-Hash-Tables.pdf)

---

## Hash Tables vs Alternatives

| Feature           | Hash Table  | Balanced BST   | Skip List   | Trie          | Bloom Filter   |
|-------------------|-------------|----------------|-------------|---------------|----------------|
| Avg Lookup        | O(1)        | O(log n)       | O(log n)    | O(k) key len  | O(k) hash count |
| Worst Lookup      | O(n)        | O(log n) ✓     | O(n) rare   | O(k)          | O(k)           |
| Ordered Iteration | No          | Yes            | Yes         | Prefix only   | No             |
| Range Queries     | No          | Efficient      | Efficient   | Prefix only   | No             |
| False Positives   | None        | None           | None        | None          | Yes (by design) |
| Best FinTech fit  | Dedup, caches, lookups | Sorted data, rate schedules | Concurrent sorted sets | IFSC autocomplete | Pre-filter blacklists |

**Decision rule**: need fast lookup with no ordering → Hash Table. Need guaranteed worst-case O(log n) → BST. Need prefix search → Trie. Need 10× less memory with acceptable false positive rate → Bloom Filter. (source: Hashing-and-Hash-Tables.pdf)

---

## Common Pitfalls

| Pitfall              | Problem                                                    | Fix                                                       |
|----------------------|------------------------------------------------------------|-----------------------------------------------------------|
| Mutable keys         | Key changes after insert → hash changes → entry is "lost" | Always use immutable keys: strings, tuples, frozen objects |
| equals/hashCode mismatch | Logically equal objects hash to different buckets     | In Java: always override both together                    |
| Integer overflow     | Large polynomial hash overflows int → negative indices     | Use modular arithmetic; `Math.floorMod()` in Java         |
| Resize in hot path   | Unexpected latency spike during resize event               | Pre-size at init: `new HashMap<>(expected / load_factor)` |
| Float as key         | Floating-point precision → "equal" values hash differently | Round to fixed precision or use integer (₹ amounts → paise as long) |

(source: Hashing-and-Hash-Tables.pdf)

#adsa
