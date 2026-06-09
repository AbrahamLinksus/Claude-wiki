# KMP Algorithm

**Summary**: Knuth-Morris-Pratt is a linear-time string search algorithm that avoids redundant comparisons by precomputing an LPS array that encodes how much of the pattern can be reused after a mismatch.

**Pre-Req**: basic string indexing, [[adsa-arrays]]

**Sources**: Session-built (no raw file)

**Last updated**: 2026-06-04

---

## Why KMP Exists

Naive string search is O(n × m) — on every mismatch it resets the pattern pointer to 0 and slides one character forward in the text. KMP is O(n + m) because the LPS array tells it exactly how far back to fall in the pattern without re-examining text characters it already passed.

---

## The LPS Array

**LPS = Longest Proper Prefix which is also a Suffix**

For a pattern of length `m`, `lps[i]` = the length of the longest proper prefix of `pattern[0..i]` that is also a suffix of `pattern[0..i]`.

- "Proper" means not the whole substring itself.
- `lps[0]` is always `0` — a single character has no proper prefix.

**Intuition**: `lps[i]` answers "if a mismatch happens just after position `i`, how many characters of the pattern are we guaranteed to have already matched?"

Example — pattern `"AABAAB"`:

| i | pattern[0..i] | longest prefix = suffix | lps[i] |
|---|---------------|-------------------------|--------|
| 0 | A             | (none)                  | 0      |
| 1 | AA            | "A"                     | 1      |
| 2 | AAB           | (none)                  | 0      |
| 3 | AABA          | "A"                     | 1      |
| 4 | AABAA         | "AA"                    | 2      |
| 5 | AABAAB        | "AAB"                   | 3      |

Result: `lps = [0, 1, 0, 1, 2, 3]`

---

## LPS Construction — Theory

Two pointers: `i` scans forward through the pattern, `len` tracks the length of the current candidate prefix-suffix.

**Case 1 — match**: `pattern[i] == pattern[len]`
- The current prefix-suffix just got one character longer.
- Set `lps[i] = len + 1`, advance both `i` and `len`.

**Case 2 — mismatch, len > 0**: `pattern[i] != pattern[len]`, but `len != 0`
- Fall back: `len = lps[len - 1]`
- Do NOT move `i` — try matching `pattern[i]` against the shorter prefix.
- This is the key insight: `lps[len-1]` is the longest prefix-suffix of the prefix we just abandoned, so it is still guaranteed to match.

**Case 3 — mismatch, len == 0**: `pattern[i] != pattern[0]`
- No prefix-suffix possible. Set `lps[i] = 0`, advance `i`.

---

## LPS Construction — Pseudocode

```
function compute_lps(pattern):
    m   = length(pattern)
    lps = array of zeros, size m
    len = 0
    i   = 1

    while i < m:
        if pattern[i] == pattern[len]:
            len = len + 1
            lps[i] = len
            i = i + 1
        else:
            if len != 0:
                len = lps[len - 1]     // fall back, don't move i
            else:
                lps[i] = 0
                i = i + 1

    return lps
```

---

## Dry Run — pattern `"AABAAB"`

Initial state: `lps = [0,0,0,0,0,0]`, `len = 0`, `i = 1`

| Step | i | len | pattern[i] | pattern[len] | Match? | Action                        | lps after         |
|------|---|-----|------------|--------------|--------|-------------------------------|-------------------|
| 1    | 1 | 0   | A          | A            | yes    | len=1, lps[1]=1, i=2          | [0,1,0,0,0,0]     |
| 2    | 2 | 1   | B          | A            | no     | len!=0 → len=lps[0]=0         | [0,1,0,0,0,0]     |
| 3    | 2 | 0   | B          | A            | no     | len==0 → lps[2]=0, i=3        | [0,1,0,0,0,0]     |
| 4    | 3 | 0   | A          | A            | yes    | len=1, lps[3]=1, i=4          | [0,1,0,1,0,0]     |
| 5    | 4 | 1   | A          | A            | yes    | len=2, lps[4]=2, i=5          | [0,1,0,1,2,0]     |
| 6    | 5 | 2   | B          | B            | yes    | len=3, lps[5]=3, i=6          | [0,1,0,1,2,3]     |

Final: `lps = [0, 1, 0, 1, 2, 3]`

---

## KMP Search — Dry Run

Using `lps = [0, 1, 0, 1, 2, 3]` from above.
text = `"AABAACDAABAAB"`, pattern = `"AABAAB"`

`i` is the text pointer, `j` is the pattern pointer.

| Step | i  | j | text[i] | pattern[j] | Match? | Action                              |
|------|----|---|---------|------------|--------|-------------------------------------|
| 1    | 0  | 0 | A       | A          | yes    | i=1, j=1                            |
| 2    | 1  | 1 | A       | A          | yes    | i=2, j=2                            |
| 3    | 2  | 2 | B       | B          | yes    | i=3, j=3                            |
| 4    | 3  | 3 | A       | A          | yes    | i=4, j=4                            |
| 5    | 4  | 4 | A       | A          | yes    | i=5, j=5                            |
| 6    | 5  | 5 | **C**   | B          | **no** | j!=0 → j=lps[4]=**2**               |
| 7    | 5  | 2 | C       | B          | no     | j!=0 → j=lps[1]=**1**               |
| 8    | 5  | 1 | C       | A          | no     | j!=0 → j=lps[0]=**0**               |
| 9    | 5  | 0 | C       | A          | no     | j==0 → i=6                          |
| 10   | 6  | 0 | D       | A          | no     | j==0 → i=7                          |
| 11   | 7  | 0 | A       | A          | yes    | i=8, j=1                            |
| 12   | 8  | 1 | A       | A          | yes    | i=9, j=2                            |
| 13   | 9  | 2 | B       | B          | yes    | i=10, j=3                           |
| 14   | 10 | 3 | A       | A          | yes    | i=11, j=4                           |
| 15   | 11 | 4 | A       | A          | yes    | i=12, j=5                           |
| 16   | 12 | 5 | B       | B          | yes    | i=13, j=6 → **MATCH at index 7**    |

Key moment: at step 6, five characters were already matched (AABAA). Instead of resetting `j` to 0, KMP falls back to `j=2` — it knows the first two characters of the pattern ("AA") still hold. The text pointer `i` never moves backward.

---

## Python Code

```python
def compute_lps(pattern):
    m = len(pattern)
    lps = [0] * m
    length = 0
    i = 1
    while i < m:
        if pattern[i] == pattern[length]:
            length += 1
            lps[i] = length
            i += 1
        else:
            if length != 0:
                length = lps[length - 1]   # fall back without moving i
            else:
                lps[i] = 0
                i += 1
    return lps


def kmp_search(text, pattern):
    n, m = len(text), len(pattern)
    lps = compute_lps(pattern)
    matches = []
    i = 0   # text pointer
    j = 0   # pattern pointer
    while i < n:
        if text[i] == pattern[j]:
            i += 1
            j += 1
        if j == m:                         # full match found
            matches.append(i - j)
            j = lps[j - 1]                 # slide pattern using LPS
        elif i < n and text[i] != pattern[j]:
            if j != 0:
                j = lps[j - 1]             # fall back pattern pointer
            else:
                i += 1                     # no fallback possible, advance text
    return matches


# Example
text    = "AABAABAABAAB"
pattern = "AABAAB"
print(compute_lps(pattern))      # [0, 1, 0, 1, 2, 3]
print(kmp_search(text, pattern)) # [0, 3, 6]
```

---

## Complexity

| Phase          | Time   | Space |
|----------------|--------|-------|
| LPS build      | O(m)   | O(m)  |
| Search         | O(n)   | O(1)  |
| **Total**      | O(n+m) | O(m)  |

Neither pointer ever moves backward — `i` in the LPS build and the text pointer in the search only increase, which is what guarantees linear time.

#adsa
