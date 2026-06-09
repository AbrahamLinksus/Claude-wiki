# Trie (Prefix Tree)

**Summary**: A Trie is a tree where each path from root to a leaf spells a complete word — enabling O(m) insert, search, and prefix operations that no hash map can match for prefix-based queries.

**Pre-Req**: [[adsa-arrays]] — arrays and indexing; [[adsa-strings]] — string fundamentals and pattern matching context

**Sources**: `1 - Strings-and-Advanced-String-Algorithms.pdf`

**Last updated**: 2026-06-09

---

## What Is a Trie?

A **Trie** (pronounced "try", from re**trie**val) is a tree-shaped data structure where:

- Every **node** represents a single character.
- Every **path from root to a node** represents a prefix of one or more stored words.
- Every **path from root to a node marked as "end of word"** represents one complete stored word.

The root node represents the empty string. Children of the root represent single characters. Children of those represent 2-character prefixes, and so on.

```
Words stored: "car", "cat", "cart", "bat", "ball"

                (root)
               /      \
              c         b
              |        / \
              a        a   (nothing else)
             / \      / \
            r   t    t   l
            |   |    |   |
           [t] [*]  [*]  l
            |             |
           [*]            [*]

Legend:
  [*] = isEndOfWord flag is TRUE here
  Letters on edges = the character stored in the child node
```

Reading the diagram:
- "car": root → c → a → r → **end** (the [*] below r without the 't' branch)
- "cat": root → c → a → t → **end**
- "cart": root → c → a → r → t → **end**
- "bat": root → b → a → t → **end**
- "ball": root → b → a → l → l → **end**

Notice that "car" and "cart" **share** the nodes for c, a, r. This prefix sharing is the defining property of a Trie.

---

## Why Not Just Use a Hash Map?

A hash map stores complete words as keys. It answers "does word X exist?" in O(m) average (computing the hash costs O(m)). It cannot efficiently answer:

- "Give me all words that start with 'pre'." — requires scanning all keys: O(total characters in dictionary).
- "Is 'pre' a valid prefix of any word?" — same problem.
- "Find the longest word in the dictionary that is a prefix of input string." — no efficient way.

A Trie answers all three in O(prefix_length) — just walk down the trie one character at a time.

```
Hash map query "all words starting with 'ca'":
  Must scan every key and check its prefix → O(total characters)

Trie query "all words starting with 'ca'":
  Walk root → c → a   (2 steps)
  Then collect all words reachable below this node
  → O(2 + number of results)
```

For large dictionaries this difference is enormous.

---

## Node Structure

Each node in a Trie needs two things:

1. **children**: an array (or map) linking to the 26 possible next characters (for lowercase English a-z).
2. **isEndOfWord**: a boolean flag marking whether a complete word ends at this node.

```
Trie Node:
┌─────────────────────────────────────────────────────┐
│  children[26]   → pointers to child nodes           │
│                   children[0] = pointer for 'a'     │
│                   children[1] = pointer for 'b'     │
│                   ...                               │
│                   children[25] = pointer for 'z'   │
│                   NULL means "no child for that letter" │
│                                                     │
│  isEndOfWord    → True if a word ends here          │
└─────────────────────────────────────────────────────┘
```

The index for a character `c` is computed as: `c - 'a'`  (so 'a'=0, 'b'=1, ..., 'z'=25).

In pseudocode:
```
class TrieNode:
    children = array of 26 NULLs
    isEndOfWord = False
```

For Unicode or mixed-character alphabets, replace the fixed array with a hash map from character to child node. The logic stays identical; the array just trades constant-time access for variable-size storage.

---

## Insert — O(m)

Insert a word of length m by walking the trie one character at a time. If a node for the current character does not exist, create it. When all characters are consumed, mark the final node as `isEndOfWord = True`.

### Step-by-Step: Insert "cat" into an empty Trie

```
Start: root is an empty node.

Character 'c':
  root.children['c'-'a'] = root.children[2]
  Is it NULL? YES → create a new node for 'c'
  Move down to node_c.

  root
   |
   c (new node)

Character 'a':
  node_c.children['a'-'a'] = node_c.children[0]
  Is it NULL? YES → create a new node for 'a'
  Move down to node_a.

  root
   |
   c
   |
   a (new node)

Character 't':
  node_a.children['t'-'a'] = node_a.children[19]
  Is it NULL? YES → create a new node for 't'
  Move down to node_t.

  root
   |
   c
   |
   a
   |
   t (new node)

End of word:
  node_t.isEndOfWord = True

  root
   |
   c
   |
   a
   |
   t  [*]
```

### Step-by-Step: Now Insert "car" into the Same Trie

```
Character 'c':
  root.children[2] → already exists (node_c)
  Just move down. No new node needed.

Character 'a':
  node_c.children[0] → already exists (node_a)
  Just move down. No new node needed.

Character 'r':
  node_a.children['r'-'a'] = node_a.children[17]
  Is it NULL? YES → create a new node for 'r'
  Move down to node_r.

End of word:
  node_r.isEndOfWord = True

  root
   |
   c
   |
   a
  / \
 r   t
[*] [*]

'c' and 'a' are shared between "car" and "cat".
```

### Pseudocode

```
function insert(root, word):
    node = root
    for each character ch in word:
        index = ch - 'a'
        if node.children[index] == NULL:
            node.children[index] = new TrieNode()
        node = node.children[index]
    node.isEndOfWord = True
```

Time: O(m) where m = length of word. One node visited or created per character.
Space per insertion: O(m) new nodes in the worst case (no shared prefix with existing words).

---

## Search — O(m)

Search for an exact word by walking the trie. If any character's child is missing (NULL), the word is not present. After consuming all characters, check `isEndOfWord` — the path existing is not enough, the word must have been explicitly inserted.

### Why Check isEndOfWord?

```
Trie contains: "cart"

Query: "car"
  Walk: root → c → a → r   (all nodes exist)
  node_r.isEndOfWord = False  ← "car" was never inserted
  → return False

Query: "cart"
  Walk: root → c → a → r → t
  node_t.isEndOfWord = True
  → return True
```

Without the `isEndOfWord` flag, "car" would falsely appear to exist because it shares a prefix path with "cart".

### Step-by-Step: Search "cat"

```
Trie contains: "car", "cat"

  root
   |
   c
   |
   a
  / \
 r   t
[*] [*]

Query: "cat"
  Step 1: ch='c', index=2
    root.children[2] exists → move to node_c

  Step 2: ch='a', index=0
    node_c.children[0] exists → move to node_a

  Step 3: ch='t', index=19
    node_a.children[19] exists → move to node_t

  End of word: node_t.isEndOfWord = True → return True  ✓
```

### Step-by-Step: Search "cab" (not inserted)

```
  Step 1: ch='c' → move to node_c  ✓
  Step 2: ch='a' → move to node_a  ✓
  Step 3: ch='b', index=1
    node_a.children[1] = NULL → return False  ✓
```

### Pseudocode

```
function search(root, word):
    node = root
    for each character ch in word:
        index = ch - 'a'
        if node.children[index] == NULL:
            return False
        node = node.children[index]
    return node.isEndOfWord
```

Time: O(m). Space: O(1) extra (just a pointer walking down).

---

## startsWith (Prefix Search) — O(m)

Check whether any inserted word starts with a given prefix. This is identical to `search` except the final check: we do NOT require `isEndOfWord`. If we successfully walk all characters of the prefix without hitting a NULL, then some word with that prefix exists.

```
Trie contains: "cart"

Query: startsWith("car")
  Walk: root → c → a → r   (all nodes exist)
  → return True  (no isEndOfWord check needed)

Query: startsWith("cab")
  Walk: root → c → a → ?  ('b' child is NULL)
  → return False
```

### Pseudocode

```
function startsWith(root, prefix):
    node = root
    for each character ch in prefix:
        index = ch - 'a'
        if node.children[index] == NULL:
            return False
        node = node.children[index]
    return True   // reached end of prefix — some word continues from here
```

Time: O(m) where m = length of prefix.

### Autocomplete: Collecting All Words with a Prefix

`startsWith` just returns True/False. For full autocomplete (return all words), walk to the prefix node, then do a depth-first traversal of all nodes below it, collecting words at each `isEndOfWord` node.

```
function autocomplete(root, prefix):
    node = root
    for each ch in prefix:
        if node.children[ch - 'a'] == NULL:
            return []    // no words with this prefix
        node = node.children[ch - 'a']
    
    results = []
    dfs(node, prefix, results)
    return results

function dfs(node, currentWord, results):
    if node.isEndOfWord:
        results.append(currentWord)
    for i from 0 to 25:
        if node.children[i] != NULL:
            dfs(node.children[i], currentWord + char(i + 'a'), results)
```

Time: O(m + k) where k = total characters in all matching words.

---

## Delete — O(m)

Deletion has two levels:

**Soft delete**: just set `isEndOfWord = False` at the target node. The nodes remain in the trie. Simple and safe — other words sharing the path are unaffected.

**Hard delete with pruning**: after clearing `isEndOfWord`, walk back up and remove any nodes that are now dead (no children, not an end of another word). This reclaims memory but requires care.

### Step-by-Step Soft Delete: Remove "car" from Trie containing "car", "cart"

```
Before:
  root → c → a → r [*] → t [*]

Step 1: Walk to the node for the last character of "car" = node_r
Step 2: node_r.isEndOfWord = False

After:
  root → c → a → r [ ] → t [*]

"cart" is unaffected. "car" no longer returns True on search.
```

### Step-by-Step Hard Delete with Pruning: Remove "cart" from Trie containing only "cart"

```
Before:
  root → c → a → r → t [*]

Step 1: Walk to node_t. Set node_t.isEndOfWord = False.
Step 2: node_t has no children and isEndOfWord=False → it is dead. Delete it.
Step 3: node_r now has no children and isEndOfWord=False → dead. Delete it.
Step 4: node_a now has no children and isEndOfWord=False → dead. Delete it.
Step 5: node_c now has no children and isEndOfWord=False → dead. Delete it.

After:
  root   (empty trie)
```

### Step-by-Step Hard Delete: Remove "car" from Trie containing "car", "cart"

```
Before:
  root → c → a → r [*] → t [*]

Step 1: node_r.isEndOfWord = False.
Step 2: node_r has a child (t) → NOT dead. Stop pruning.

After:
  root → c → a → r [ ] → t [*]

"cart" is fully preserved.
```

### Pseudocode (recursive with pruning)

```
function delete(node, word, depth):
    if depth == len(word):
        node.isEndOfWord = False
        // Return True if node can be deleted (no children left)
        return not hasChildren(node)
    
    index = word[depth] - 'a'
    if node.children[index] == NULL:
        return False   // word not found
    
    shouldDelete = delete(node.children[index], word, depth + 1)
    
    if shouldDelete:
        node.children[index] = NULL
        // This node itself can be deleted if it has no other children and
        // is not the end of another word
        return not hasChildren(node) and not node.isEndOfWord
    
    return False

function hasChildren(node):
    for i from 0 to 25:
        if node.children[i] != NULL: return True
    return False
```

Time: O(m). The recursion unwinds at most m levels.

---

## Space Complexity

Each TrieNode allocates an array of 26 pointers. On a 64-bit system, one pointer is 8 bytes, so one node's children array costs:

```
26 pointers × 8 bytes = 208 bytes per node
```

Total space for a trie with N words of average length L:

```
O(ALPHABET_SIZE × average_key_length × number_of_keys)
= O(26 × L × N)
```

### Comparison with Hash Map

| Structure | Space | Lookup | Prefix Search |
|---|---|---|---|
| Hash map | O(total characters) | O(m) average | O(total) — must scan all keys |
| Trie (array children) | O(26 × total nodes) | O(m) worst | O(m + results) |
| Trie (hash map children) | O(total characters) | O(m) average | O(m + results) |

The array-children trie wastes memory on empty slots. For a dictionary of common English words (large alphabet coverage), utilization can be reasonable. For sparse alphabets or large Unicode ranges, use hash map children.

Concretely: a dictionary of 100,000 English words, average length 8:
- Hash map: ~800,000 characters stored = ~800KB
- Array-children Trie: many shared prefixes reduce node count, but 26 pointers per node means possibly 3-5MB
- Hash-map-children Trie: similar to hash map in space but O(m) prefix queries

---

## Why Trie Beats Hash Map For These Problems

### 1. Autocomplete

```
Scenario: User typed "comp". Suggest all valid words.

Hash map approach:
  Iterate all 100,000 keys, check if each starts with "comp" → O(total chars)

Trie approach:
  Walk root → c → o → m → p  (4 steps)
  DFS below that node → collect all matches → O(4 + result_size)
```

This is why every phone keyboard and search bar autocomplete uses a Trie (or a variant).

### 2. Spell Checking

A spell checker needs two operations: "is this a valid word?" (exact search) and "is this a valid prefix?" (for suggesting completions). Both are O(m) in a Trie.

Additionally, a Trie supports **fuzzy search** (words within edit distance k) by exploring multiple branches simultaneously — much harder with a hash map.

### 3. IP Routing (Longest Prefix Match)

IP routing tables store prefixes like `192.168.1.0/24`. Given an incoming packet's destination IP, the router must find the most specific (longest) matching prefix.

```
Routing table prefixes (in binary):
  0* → interface A
  00* → interface B
  001* → interface C

Destination: 00101...

Walk trie bit-by-bit:
  '0' → match 0*, record interface A
  '0' → match 00*, record interface B (more specific)
  '1' → match 001*, record interface C (even more specific)
  '0' → no more matches
  → Route to interface C (longest prefix match)
```

Modern routers use compressed tries (Patricia tries) for this, handling millions of prefixes with sub-microsecond lookup times.

---

## Compressed Trie (Patricia Trie / Radix Tree)

In a standard Trie, long chains of single-child nodes waste space and time. The word "strength" alone creates 8 nodes, each with exactly one child.

```
Standard Trie for "strength":
  root → s → t → r → e → n → g → t → h [*]
         (8 nodes, each with 1 child = wasteful)
```

A **Compressed Trie** (also called Patricia Trie or Radix Tree) merges any chain of single-child nodes into a single edge labeled with the entire substring.

```
Compressed Trie for "strength":
  root --"strength"--> [*]
         (1 node, 1 edge with label "strength")

Compressed Trie for "string" and "strength":
  root --"str"--> node_str --"ing"--> [*]  ("string")
                           |
                           +"ength"--> [*]  ("strength")
```

Each edge now carries a **string label** (a substring) rather than a single character.

### Full Example: Compressed Trie for "car", "cat", "cart"

```
Standard Trie:
  root → c → a → r [*] → t [*]
               |
               t [*]

All nodes have at most 2 children except the a-node. No single-child chains
long enough to compress here.

Add "database":
  root → c → a → r [*] → t [*]
               |
               t [*]
         d → a → t → a → b → a → s → e [*]

Compressed (merge d→a→t→a→b→a→s→e into one edge):
  root --"c"--> node_c --"a"--> node_a --"r"--> [*] --"t"--> [*]
                                         |
                                         +"t"--> [*]
       |
       +"database"--> [*]
```

### Space Savings

A Patricia Trie on n strings of total length L uses at most O(n) nodes (at most n-1 internal nodes and n leaves). This is a dramatic improvement over O(L) nodes in the standard trie for long, non-overlapping strings.

### Trade-off

Insertion and deletion in a compressed trie require **splitting edges** when a new word shares only part of an edge label. The logic is more complex than a standard trie, but the asymptotic time stays O(m).

---

## Aho-Corasick: Multi-Pattern Matching with a Trie

**Problem**: Given a dictionary of k patterns P1, P2, ..., Pk and a text T, find all occurrences of all patterns in T simultaneously.

Naive approach: run KMP for each pattern → O(k * n). For k = 10,000 patterns and n = 10^9 characters (a large log file), this is 10^13 operations — completely infeasible.

**Aho-Corasick** (1987) solves this in O(n + total_pattern_length + number_of_matches):
- Build a Trie of all patterns.
- Add **failure links** to every node: when a character mismatch occurs during search, the failure link points to the longest proper suffix of the current path that is also a prefix of some pattern in the trie. This is the same idea as KMP's prefix function, generalized to a trie.
- Scan T once. At each character, follow the trie edge (if it exists) or follow failure links (if not). Record every end-of-word node reached.

```
Patterns: "he", "she", "his", "hers"

Trie:
  root → h → e [*"he"] → r → s [*"hers"]
       → s → h → e [*"she"]
       → h → i → s [*"his"]

Failure links (conceptual):
  node for "she" → node for "he"  (suffix "he" of "she" is a prefix of "he")
  node for "hers" → node for "s"  (no shared prefix longer than "s")
  ...

Scanning text "ushers":
  u: no match, stay at root
  s: follow edge s from root
  h: follow edge h from s-node  (now at "sh" state)
  e: follow edge e from sh-node → "she" match! Also follow failure to "he" → "he" match!
  r: follow edge r from she-node
  s: follow edge s from sher-node → "hers" match!
```

A single pass over "ushers" found all four patterns. This is why Aho-Corasick is used in intrusion detection systems to scan network traffic for thousands of attack signatures simultaneously.

Full Aho-Corasick implementation is an advanced topic; the key takeaway is that a Trie + failure links generalizes KMP to handle arbitrarily many patterns at no extra cost per pattern during the text scan.

---

## Complexity Summary

| Operation | Time | Space |
|---|---|---|
| Insert word of length m | O(m) | O(m) new nodes worst case |
| Search exact word of length m | O(m) | O(1) extra |
| startsWith prefix of length m | O(m) | O(1) extra |
| Delete word of length m | O(m) | O(m) freed nodes worst case |
| Autocomplete (prefix len p, r results) | O(p + r) | O(r) for result list |
| Build trie from N words total length L | O(L) | O(26 × nodes) |
| Aho-Corasick build (total pattern len K) | O(K × 26) | O(K × 26) |
| Aho-Corasick search (text len n, total matches R) | O(n + R) | O(1) extra |

---

## Implementation Notes

### Python Sketch

```python
class TrieNode:
    def __init__(self):
        self.children = {}      # hash map: char → TrieNode (handles any alphabet)
        self.is_end = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end = True

    def search(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:
                return False
            node = node.children[ch]
        return node.is_end

    def starts_with(self, prefix):
        node = self.root
        for ch in prefix:
            if ch not in node.children:
                return False
            node = node.children[ch]
        return True
```

Using a hash map for children handles any character (including Unicode) without the 26-slot restriction. The trade-off: hash map children are slightly slower due to hashing overhead, but O(m) in all operations still holds.

### Fixed-Array Version (Lowercase a-z Only)

```python
ALPHABET = 26

class TrieNode:
    def __init__(self):
        self.children = [None] * ALPHABET
        self.is_end = False

def insert(root, word):
    node = root
    for ch in word:
        idx = ord(ch) - ord('a')
        if node.children[idx] is None:
            node.children[idx] = TrieNode()
        node = node.children[idx]
    node.is_end = True
```

This version is faster per operation (array index vs hash lookup) but restricted to lowercase English and wastes 208 bytes per node for the array.

---

## Applications Summary

| Application | How Trie Helps |
|---|---|
| Autocomplete (keyboard, search bar) | Walk to prefix node, DFS to collect all words |
| Spell checker | Exact search for valid words; prefix search for suggestions |
| IP routing (longest prefix match) | Walk bits of IP address, track deepest match seen |
| DNS prefix lookup | Same as IP routing, over domain name labels |
| Multi-pattern string search | Aho-Corasick: trie + failure links, single-pass scan |
| Word games (Boggle, Scrabble) | Prune search early when no word starts with current path |
| Contact/phone book search | Type letters, immediately filter all matching contacts |
| Command-line tab completion | Shell completions are trie lookups over available commands |

---

## Related Pages

- [[adsa-strings]] — string fundamentals, pattern matching overview, algorithm comparison table
- [[kmp-algorithm]] — KMP prefix function (the 1D analogue of Aho-Corasick failure links)
- [[rabin-karp]] — rolling hash multi-pattern variant
- [[adsa-hashing]] — hash maps, collision handling (relevant for hash-map-children trie)
- [[adsa-arrays]] — dynamic array mechanics underlying many trie node implementations

#adsa
