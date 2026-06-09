# Strings

**Summary**: A complete guide to strings as a data structure — how they are stored in memory, why encoding matters, and a full survey of pattern-matching algorithms from the naive O(nm) approach through KMP, Rabin-Karp, and Boyer-Moore.

**Pre-Req**: [[adsa-arrays]] — arrays and memory layout; [[adsa-hashing]] — hash functions and collision

**Sources**: `1 - Strings-and-Advanced-String-Algorithms.pdf`

**Last updated**: 2026-06-09

---

## What Is a String?

A string is a **sequence of characters**. That sounds simple, but the details of *how* those characters are stored differ significantly between languages and systems.

```
The string "hello"

index:  0    1    2    3    4
      [ h  | e  | l  | l  | o  ]
```

Every character sits at a numbered position (index). The key question any implementation must answer is: **where does the string end?**

---

## Null-Termination vs Length-Prefix

There are two dominant strategies for marking the end of a string.

### Null-Terminated Strings (C)

In C, a string is just a char array. The end is marked by a special byte with value 0, written `'\0'`. There is no separate length stored.

```
"cat" stored in C:

index:  0    1    2    3
      [ c  | a  | t  | \0 ]
                        ↑ terminator — signals "string ends here"
```

**Consequence**: to find the length of a string, you must walk the entire array counting characters until you hit `\0`. That is O(n). There is no way to know the length in O(1) without scanning.

**Risk**: if you accidentally overwrite `\0`, or forget to allocate space for it, you get a buffer overflow — reading past the array's memory.

### Length-Prefix Strings (Python, Java, Go, Rust...)

Modern languages store the length alongside the character data. Python's `str` object internally keeps a `length` field. Java's `String` object does the same.

```
Python "cat" conceptually:

[ length=3 | c | a | t ]
  ↑ stored separately
```

**Consequence**: `len("cat")` is O(1) — no scanning needed, just read the length field.

**Trade-off**: slightly more memory per string object, but the O(1) length query is almost always worth it.

---

## ASCII vs Unicode

### ASCII: The Simple Case

ASCII assigns a number 0–127 to each common English character. Each character fits in exactly 1 byte.

```
'A' = 65    'a' = 97    '0' = 48    ' ' = 32    '\n' = 10
```

With ASCII, 1 byte always = 1 character. Length in bytes = number of characters. Simple.

### Unicode: The Real World

Unicode assigns a **code point** (a number) to every character in every human language — over 140,000 characters. The code points range from U+0000 to U+10FFFF.

The challenge: **how do you encode these numbers into bytes?**

#### UTF-8 (Variable Width)

UTF-8 is the dominant encoding on the web and in most modern systems. It uses 1 to 4 bytes per character depending on the code point range.

```
Code point range        Bytes used    Examples
─────────────────────────────────────────────────────────
U+0000  – U+007F        1 byte        ASCII (A, z, 0–9)
U+0080  – U+07FF        2 bytes       Latin extended, Arabic, Hebrew
U+0800  – U+FFFF        3 bytes       Chinese, Japanese, Korean
U+10000 – U+10FFFF      4 bytes       Emoji, rare scripts
```

Example:
```
"A"     → 1 byte  (0x41)
"é"     → 2 bytes (0xC3 0xA9)
"中"    → 3 bytes (0xE4 0xB8 0xAD)
"😀"    → 4 bytes (0xF0 0x9F 0x98 0x80)
```

Critical consequence: **byte length != character count for non-ASCII text**.

```python
s = "café"
len(s)         # 4  — Python counts Unicode code points (characters)
len(s.encode('utf-8'))   # 5  — 'é' takes 2 bytes
```

#### UTF-16 and Surrogate Pairs

UTF-16 uses 2 bytes per character for the most common code points (U+0000–U+FFFF, the Basic Multilingual Plane). For rarer code points above U+FFFF, it uses a pair of 2-byte units called a **surrogate pair** — so those characters take 4 bytes total.

```
UTF-16 surrogate pair example: emoji U+1F600 "😀"
→ encoded as two 16-bit units: 0xD83D 0xDE00
```

Java's `char` type is 16-bit UTF-16. This means a Java `char` cannot represent emoji — those require two `char` values. String length in Java counts `char` units, not human-visible characters.

---

## String Immutability

In Python and Java, strings are **immutable** — once created, the characters inside cannot be changed. A new string object must be created for any modification.

```python
s = "hello"
s[0] = 'H'    # ERROR — strings are immutable in Python
```

### Why Immutability?

- **Safety**: strings can be shared between objects without risk of one modifying what another sees.
- **Hashing**: immutable strings can be hashed once and the hash cached. Python's `str` and Java's `String` are safe to use as dictionary/hash-map keys because their hash never changes.
- **Interning**: the runtime can deduplicate identical string literals.

### The O(n²) Concatenation Trap

Because strings are immutable, naive concatenation in a loop is expensive.

```python
# BAD: O(n²) time
result = ""
for word in words:
    result = result + word   # creates a brand new string each time
```

Each `+` allocates a new string, copies the old content, and appends the new word. If you concatenate n words of average length k, the total work is:
```
k + 2k + 3k + ... + nk  =  k * n*(n+1)/2  =  O(n²k)
```

For 10,000 words this can be millions of character copies.

```
Step 1:  "hello"               (5 chars copied)
Step 2:  "hello world"         (11 chars copied — started over)
Step 3:  "hello world foo"     (15 chars copied — started over)
...
```

### The Fix: StringBuilder / bytearray / join

Use a mutable buffer that appends in-place, then convert to a string once at the end.

```python
# GOOD: O(n) time
parts = []
for word in words:
    parts.append(word)
result = "".join(parts)   # one final copy
```

`"".join(parts)` knows the total length in advance, allocates once, and copies each word exactly once. Total work: O(total characters) = O(n).

In Java, `StringBuilder` does the same thing — it maintains a mutable internal buffer that doubles in capacity as needed (exactly like a dynamic array).

---

## Common O(n) String Operations

| Operation | Description | Time | Notes |
|---|---|---|---|
| `len(s)` | Return length | O(1) | Length-prefix languages only; O(n) in C |
| `s[i]` | Access character at index | O(1) | Direct array indexing |
| `s[i:j]` | Substring / slice | O(j-i) | Must copy the characters |
| `s == t` | Compare two strings | O(min(n,m)) | Compares char-by-char until mismatch |
| `s + t` | Concatenate | O(n+m) | Allocates new string, copies both |
| `s.find(p)` | Search for pattern p | O(nm) naive | See pattern matching section below |

Substring copy is O(k) where k is the length of the extracted substring because the characters must be copied into a new object. There is no "free" slicing that avoids the copy in most standard libraries.

---

## Pattern Matching: The Problem

The **string matching problem**: given a text T of length n and a pattern P of length m, find all positions i in T where T[i..i+m-1] == P.

```
Text T:    A B A B C A B A B A B
                               n = 11
Pattern P: A B A B
                   m = 4

Matches at positions: 0, 6
```

This problem appears everywhere: Ctrl+F in text editors, grep, DNA sequence search, spam filter keyword matching, plagiarism detection.

---

## Naive Algorithm — O(nm)

The simplest approach: slide P along T one position at a time, comparing all m characters at each position.

```
Text:     A B A B C A B A B A B
Pattern:  A B A B
          ↑ try position 0

Compare:
T[0]=A == P[0]=A  ✓
T[1]=B == P[1]=B  ✓
T[2]=A == P[2]=A  ✓
T[3]=B == P[3]=B  ✓  → MATCH at position 0

Now slide to position 1:
Text:     A B A B C A B A B A B
Pattern:    A B A B
            ↑ try position 1

T[1]=B vs P[0]=A  ✗  → mismatch at first char, skip

Slide to position 2:
Text:     A B A B C A B A B A B
Pattern:      A B A B
              ↑ try position 2

T[2]=A == P[0]=A  ✓
T[3]=B == P[1]=B  ✓
T[4]=C vs P[2]=A  ✗  → mismatch, discard all work done

...
```

The problem: after a partial match of k characters followed by a mismatch, the naive approach throws away all k comparisons and starts fresh from the next position. This leads to worst-case O(nm) — for example, searching for "AAAB" in "AAAAAAAAAA".

---

## KMP Algorithm — O(n+m)

KMP (Knuth-Morris-Pratt, 1977) fixes the naive algorithm's waste. After a mismatch, instead of sliding by 1 and restarting, it uses a precomputed **prefix function** to skip ahead as far as possible without missing any match.

Full detail is in [[kmp-algorithm]]. Summary:

- **Preprocessing**: compute pi[] array (size m) in O(m). pi[i] = length of the longest proper prefix of P[0..i] that is also a suffix of P[0..i].
- **Search**: scan T with two pointers i (text) and j (pattern). On mismatch, set j = pi[j-1] — do NOT move i backward. O(n).
- **Total**: O(n+m), worst case guaranteed.

```
Pattern "ABABAC":
Index:  0  1  2  3  4  5
P:      A  B  A  B  A  C
pi:     0  0  1  2  3  0

On mismatch at j=5 (C vs something), jump to j = pi[4] = 3.
Pattern shifts so the "ABA" prefix aligns with the "ABA" suffix already matched.
```

KMP is deterministic and immune to adversarial inputs — worst case is always O(n+m).

---

## Rabin-Karp Algorithm — O(n+m) Average

Rabin-Karp (1987) takes a completely different approach: instead of comparing characters, it **compares hash values**. A match on hashes is then verified by direct character comparison to guard against false positives.

Full detail is in [[rabin-karp]]. Summary:

- **Hash the pattern** P into a single number Hp.
- **Hash the first window** T[0..m-1] into Ht.
- **Slide** the window: compute the next hash from the previous hash in O(1) using the rolling formula. No need to rehash all m characters from scratch each shift.
- **On hash match**: verify by comparing actual characters (to eliminate spurious hits / hash collisions).

Rolling hash formula:
```
H_next = (H_current - T[i] * b^(m-1)) * b + T[i+m]   (mod M)

Where b = base (e.g. 256), M = large prime (e.g. 10^9+7)
```

Average case O(n+m). Worst case O(nm) if every window causes a collision (pathological input). Excellent for searching multiple patterns simultaneously (store all pattern hashes in a set, check each window against the set in O(1)).

---

## Boyer-Moore Algorithm — O(n/m) Best Case

Boyer-Moore (1977, published same year as KMP) is often the **fastest algorithm in practice** for large alphabets and long patterns. Unlike KMP and Rabin-Karp which scan left-to-right, Boyer-Moore scans **the pattern right-to-left** at each position, allowing large skips.

### Core Idea

Align the pattern at position i in the text. Compare P[m-1] with T[i+m-1] first (rightmost character), then work left. When a mismatch is found, two heuristics tell us how far to jump.

### Heuristic 1: Bad Character Rule

When P[j] mismatches T[i+j], look at the text character T[i+j] = x.

- If x does not appear anywhere in P: slide the pattern completely past T[i+j]. Skip m positions.
- If x appears in P at position k < j: align that occurrence with T[i+j]. Skip j-k positions.

```
Text:     A B C D A B C
Pattern:  A B C

Step 1: align P at position 0, compare right-to-left
        T[2]='C' vs P[2]='C'  ✓
        T[1]='B' vs P[1]='B'  ✓
        T[0]='A' vs P[0]='A'  ✓  → MATCH at 0

Step 2: align P at position 1
        T[3]='D' vs P[2]='C'  ✗ mismatch
        Bad character: 'D' does not appear in P at all
        → skip past 'D': slide pattern 3 positions
        
Step 3: align P at position 4
        T[6]='C' vs P[2]='C'  ✓
        T[5]='B' vs P[1]='B'  ✓
        T[4]='A' vs P[0]='A'  ✓  → MATCH at 4
```

In this example, we skipped position 1, 2, and 3 entirely. With a large alphabet (e.g., English text) most characters in the text will not appear near the current position in the pattern, so large skips are common.

### Heuristic 2: Good Suffix Rule

When a suffix of length k matched before the mismatch, look for another occurrence of that suffix elsewhere in P, or for a prefix of P that matches a suffix of the matched portion. Align to the best such match.

The good suffix rule handles cases like repetitive patterns where the bad character rule alone would only shift by 1.

```
Text:     X A B A B C ...
Pattern:  A B A B C
                ↑ mismatch here

Matched suffix so far: "BABC"  (wait, we scan right-to-left, so first mismatch
is the leftmost unmatched char)

Good suffix: find the next rightmost position where "AB" occurs in the pattern
and align, allowing a larger skip than bad character alone.
```

### Boyer-Moore Complexity

| Case | Complexity | When |
|---|---|---|
| Best case | O(n/m) | Large alphabet, rare matches, many full-pattern skips |
| Average case | O(n/m) sublinear | English text, typical patterns |
| Worst case | O(nm) | Small alphabet, many partial matches (e.g. "aaa" in "aaaa...") |

Boyer-Moore is not used when worst-case guarantees are needed (use KMP then). But for practical text searching — grep, find/replace in editors — it is often the algorithm of choice because on English text it frequently skips 3-5 characters per step.

### Preprocessing Cost

- Bad character table: O(m + sigma) where sigma = alphabet size. For ASCII, sigma = 256.
- Good suffix table: O(m).

Space: O(m + sigma).

---

## String Hashing (Polynomial Rolling Hash)

String hashing converts a string into a single integer, allowing O(1) comparison of string windows instead of O(m) character-by-character comparison.

### The Polynomial Hash

Treat each character as a number (its ASCII/Unicode code point). Compute:

```
hash(s) = s[0]*b^(m-1) + s[1]*b^(m-2) + ... + s[m-1]*b^0   (mod M)

Where:
  b = base, typically 256 (covers all ASCII) or 31 (lowercase letters only)
  M = large prime, typically 10^9 + 7
```

Example with b=10, M=1000, string "cat":
```
'c' = 99,  'a' = 97,  't' = 116

hash("cat") = 99*10^2 + 97*10^1 + 116*10^0
            = 9900 + 970 + 116
            = 10986
            = 10986 mod 1000
            = 986
```

### Rolling (Sliding Window) Hash

The key insight for Rabin-Karp: you don't need to recompute the hash of each new window from scratch. Given the hash of window T[i..i+m-1], compute the hash of T[i+1..i+m] in O(1):

```
H_new = (H_old - T[i] * b^(m-1)) * b + T[i+m]   (mod M)

Step-by-step:
1. Remove the contribution of the outgoing character T[i]:   subtract T[i] * b^(m-1)
2. Shift everything left by one power of b:                  multiply by b
3. Add the incoming character T[i+m]:                        add T[i+m]
```

```
Window 0: A B C   hash0 = A*b^2 + B*b + C
Window 1: B C D   hash1 = (hash0 - A*b^2) * b + D
                        = B*b^2 + C*b + D   ✓
```

### Collision Probability

Two different strings can produce the same hash value — this is a **hash collision** (also called a **spurious hit** in Rabin-Karp). After a hash match you must verify the actual characters.

Collision probability for a single window with a random prime M: approximately 1/M. With M = 10^9+7, this is about 1 in a billion — very unlikely in practice.

For security-sensitive applications (or adversarial input), use **double hashing** with two independent large primes. The collision probability drops to ~1/(M1*M2), which is negligible.

See [[rabin-karp]] for the full treatment.

---

## Tries: When Hash Maps Are Not Enough

A hash map can answer "does string X exist in this set?" in O(m) average (hashing costs O(m)). But hash maps cannot efficiently answer **prefix queries** like "give me all words that start with 'pre'".

A **Trie** (prefix tree) organizes strings so that shared prefixes share nodes. Every path from the root to a leaf spells one complete string. This makes prefix queries O(prefix_length) — as fast as a single lookup.

```
Trie for: "car", "cat", "cart", "bat", "ball"

        (root)
        /    \
       c      b
       |      |
       a      a
      / \    / \
     r   t  t   l
     |   |      |
    (t) (*)    l
     |          |
    (*)         (*)

(*) = isEndOfWord marker
```

Tries excel at:
- **Autocomplete**: find all words with a given prefix
- **Spell checking**: check if a prefix is a valid dictionary prefix
- **IP routing**: longest prefix match in routing tables

See [[adsa-trie]] for full coverage including insert, search, delete, space analysis, compressed tries, and Aho-Corasick multi-pattern matching.

---

## Algorithm Comparison Table

This table summarizes all major string pattern-matching algorithms covered here and in the linked pages.

| Algorithm | Preprocessing | Search Time | Space | Best Use Case |
|---|---|---|---|---|
| Naive | None — O(1) | O(nm) worst | O(1) | Very short patterns (m < 20), simple code |
| KMP | O(m) | O(n) | O(m) | Guaranteed linear time, adversarial inputs, streaming |
| Rabin-Karp | O(m) | Avg O(n+m), worst O(nm) | O(1) | Multiple patterns, plagiarism detection, 2D matching |
| Boyer-Moore | O(m + sigma) | Avg O(n/m), worst O(nm) | O(m+sigma) | Long patterns, large alphabet (English text), grep |
| Aho-Corasick | O(total pattern len) | O(n + matches) | O(total) | Fixed dictionary of patterns, intrusion detection |

### When to Use Which

```
Single pattern search
│
├── Worst-case guarantee needed?
│   └── YES → KMP  (O(n+m) guaranteed, O(m) space)
│
├── Large alphabet, long patterns, average speed matters?
│   └── YES → Boyer-Moore  (often sublinear in practice)
│
└── Short pattern (m < 20)?
    └── YES → Naive or built-in library function

Multiple pattern search
│
├── Fixed set of patterns (known upfront)?
│   └── YES → Aho-Corasick  (single pass over text)
│
└── Dynamic set of patterns (added at runtime)?
    └── YES → Rabin-Karp with hash set of pattern hashes
```

---

## Real-World Applications

| Application | Algorithm Used | Why |
|---|---|---|
| grep / text editor find | Boyer-Moore + KMP hybrid | Boyer-Moore for long patterns (fast average), KMP for short (safe worst case) |
| DNA sequence search | KMP preferred | Small alphabet (A,C,G,T) → many hash collisions for Rabin-Karp |
| Plagiarism detection | Rabin-Karp shingling | Compute rolling hashes of all k-length substrings; compare hash sets |
| Intrusion detection | Aho-Corasick | Match thousands of attack signatures in a single pass over network traffic |
| Autocomplete / spell check | Trie | Prefix queries are O(prefix length) |
| IP routing | Compressed Trie | Longest prefix match over routing table prefixes |

---

## Common Pitfalls

**1. Off-by-one in pattern index (KMP)**
KMP uses 0-based indexing. The prefix array has size m. pi[0] is always 0. Confusing 0-based and 1-based indexing produces wrong shifts and missed matches. Test on "AB" and "ABA" first.

**2. Forgetting substring verification (Rabin-Karp)**
Never report a match on a hash equality alone. Always compare actual characters at the position. Skipping this step causes false positives (spurious hits).

**3. Modulo on negative numbers**
In the rolling hash, after subtracting the outgoing character's contribution, the intermediate value can be negative. In Python `%` always returns non-negative, but in C/Java/C++ `%` with a negative operand returns a negative result.
Fix: `hash = ((hash - char*power) % MOD + MOD) % MOD`

**4. Too-small modulus**
Using MOD = 101 or other small primes causes many collisions. Every collision triggers an O(m) verification, degrading performance to O(nm). Use MOD = 10^9+7 (or two independent primes for double hashing).

**5. Unicode characters in naive implementations**
If your code assumes `s[i]` is always 1 byte (ASCII), it breaks on Unicode input. Either normalize to UTF-8 bytes, or use code-point-aware indexing. Python's `str` handles this correctly; C does not.

---

## Summary: The Progression of String Matching

```
1960s   Naive O(nm)
          ↓
1970    Morris-Pratt precursor (early linear idea)
          ↓
1977    KMP — first guaranteed linear O(n+m) worst case
        Boyer-Moore — sublinear average O(n/m) in practice
          ↓
1981    Rabin-Karp — rolling hash, extends to 2D and multi-pattern
          ↓
1987    Aho-Corasick — multi-pattern, trie + failure links
          ↓
1990s+  Hardware (SIMD/GPU) acceleration for gigabit intrusion detection
```

The deeper you go into string algorithms, the more you see a recurring theme: **precompute structure about the pattern to avoid redundant work during the search**. KMP precomputes the prefix array. Rabin-Karp precomputes the base power. Boyer-Moore precomputes shift tables. Aho-Corasick precomputes a trie with failure links. The text is always scanned once (or close to it); all the cleverness is in the preprocessing.

---

## Related Pages

- [[kmp-algorithm]] — full KMP: prefix function construction, step-by-step search, overlapping matches
- [[rabin-karp]] — full Rabin-Karp: rolling hash, spurious hits, double hashing, multi-pattern
- [[adsa-trie]] — Trie data structure: insert, search, prefix queries, compressed trie, Aho-Corasick
- [[adsa-hashing]] — hash functions, collision resolution, load factor
- [[adsa-arrays]] — dynamic arrays, memory layout, amortized analysis

#adsa
