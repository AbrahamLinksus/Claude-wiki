# Rabin-Karp Algorithm

**Summary**: String search algorithm that uses a rolling hash to compare the pattern against each text window in O(1) per step, achieving O(n+m) average-case performance without an LPS array.

**Pre-Req**: [[kmp-algorithm]], basic modular arithmetic

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-04

---

## The Core Idea

[[kmp-algorithm]] avoids redundant comparisons by tracking how much of the pattern survives a mismatch. Rabin-Karp takes a different route: instead of comparing characters one by one at each text position, it computes a hash of the current window and compares it to the pattern hash. A hash comparison is O(1), so the algorithm can scan all n windows cheaply.

---

## Rolling Hash

A polynomial hash for a string `s[0..m-1]`:

```
hash = (s[0]*d^(m-1) + s[1]*d^(m-2) + ... + s[m-1]) % q
```

- `d` = base (26 for lowercase alphabet, 256 for full ASCII)
- `q` = a large prime (reduces hash collisions)

Computing this from scratch at each position costs O(m) — still O(n·m) total. The trick is the **rolling step**.

### Sliding the Window One Character Right

To move from window `text[i..i+m-1]` to `text[i+1..i+m]`:

1. Remove the leading character's contribution: subtract `text[i] * d^(m-1)`
2. Shift everything left: multiply by `d`
3. Add the incoming character: add `text[i+m]`

```
new_hash = (d * (old_hash - text[i] * h) + text[i+m]) % q
```

where `h = d^(m-1) % q` is precomputed once — O(1) per slide.

---

## Spurious Hits

A **spurious hit** is when `text_hash == pat_hash` but the strings differ (a hash collision). When hashes match, a full O(m) character comparison is done to confirm. If all n windows collide spuriously, complexity degrades to O(n·m). Choosing a large prime `q` (e.g. 101, 1000003) keeps spurious hits rare in practice.

---

## Pseudocode

```
function rabin_karp(text, pattern, d, q):
    n, m = length(text), length(pattern)
    h    = d^(m-1) mod q

    compute pat_hash  for pattern[0..m-1]
    compute text_hash for text[0..m-1]

    for i from 0 to n-m:
        if text_hash == pat_hash:
            if text[i..i+m-1] == pattern:      // guard against spurious hits
                record match at i

        if i < n-m:
            text_hash = (d * (text_hash - text[i]*h) + text[i+m]) mod q
            if text_hash < 0:
                text_hash = text_hash + q      // keep hash positive

    return matches
```

---

## Dry Run

pattern = `"AAB"`, text = `"AABAAB"`
d = 26, q = 11, using A=0, B=1 (offset from `'A'`)

**Precompute:**
```
h            = 26^(3-1) mod 11 = 4^2 mod 11 = 16 mod 11 = 5
pattern_hash = (0*676 + 0*26 + 1) mod 11    = 1
text_hash[0] = (0*676 + 0*26 + 1) mod 11    = 1   ← window "AAB"
```

**Slide and compare:**

| i | Window | text_hash | pat_hash | Hash match? | Char check     | Result    |
|---|--------|-----------|----------|-------------|----------------|-----------|
| 0 | AAB    | 1         | 1        | yes         | "AAB"=="AAB"   | **MATCH** |
| 1 | ABA    | (26*(1 - 0·5) + 0) % 11 = 26 % 11 = **4** | 1 | no | — | skip |
| 2 | BAA    | (26*(4 - 0·5) + 0) % 11 = 104 % 11 = **5** | 1 | no | — | skip |
| 3 | AAB    | (26*(5 - 1·5) + 1) % 11 = 1 | 1 | yes | "AAB"=="AAB" | **MATCH** |

Rolling step at i=3 in detail:
```
text[2] = 'B' = 1,  text[5] = 'B' = 1
new_hash = (26 * (5 - 1*5) + 1) % 11
         = (26 * 0 + 1) % 11
         = 1
```

Matches: `[0, 3]`

---

## Python Code

```python
def rabin_karp(text, pattern, d=26, q=101):
    n, m = len(text), len(pattern)
    if m > n:
        return []

    h         = pow(d, m - 1, q)   # d^(m-1) mod q
    pat_hash  = 0
    text_hash = 0

    for i in range(m):
        pat_hash  = (d * pat_hash  + ord(pattern[i])) % q
        text_hash = (d * text_hash + ord(text[i]))    % q

    matches = []
    for i in range(n - m + 1):
        if text_hash == pat_hash:
            if text[i:i + m] == pattern:   # confirm: guards against spurious hits
                matches.append(i)

        if i < n - m:
            text_hash = (d * (text_hash - ord(text[i]) * h) + ord(text[i + m])) % q
            if text_hash < 0:
                text_hash += q

    return matches


# Example
text    = "AABAABAABAAB"
pattern = "AAB"
print(rabin_karp(text, pattern))   # [0, 3, 6, 9]
```

---

## KMP vs Rabin-Karp

| Property            | KMP                        | Rabin-Karp                         |
|---------------------|----------------------------|------------------------------------|
| Preprocessing       | LPS array — O(m)           | Initial hash — O(m)                |
| Search              | O(n)                       | O(n) average                       |
| Worst case          | O(n+m) guaranteed          | O(nm) if all windows are spurious  |
| Extra space         | O(m) for LPS               | O(1)                               |
| Multiple patterns   | Needs Aho-Corasick         | Hash all patterns, one pass        |
| Best used when      | Single pattern, must guarantee linear time | Multiple patterns or pattern changes frequently |

---

## Complexity

| Phase         | Time            | Space |
|---------------|-----------------|-------|
| Preprocessing | O(m)            | O(1)  |
| Search        | O(n) average    | O(1)  |
| **Total**     | O(n+m) average  | O(1)  |

#adsa
