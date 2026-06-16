# Wiki Operation Log

Append-only record of all wiki operations.

---

## 2026-06-16 — Adjacency List page added

**Source**: Session-built (no raw file)

**Tag**: #adsa

**Pages created** (1):
- `adsa-adjacency-list.md` — Definition covering both unweighted (`graph[u] = [v1, v2]`) and weighted (`graph[u] = [(v1, w1), ...]`) forms, directed vs undirected storage convention, key properties (O(V+E) space, O(degree) neighbor iteration), advantages/disadvantages vs matrix, creation pseudocode for both forms, use cases (sparse graphs, BFS/DFS, Dijkstra/Bellman-Ford), examples

**Pages updated** (1):
- `adsa-graphs.md` — Topic map now links `adsa-adjacency-list`

**Index updated**: yes

---

## 2026-06-16 — Self-loops folded into existing pages

**Source**: Q&A discussion

**Tag**: #adsa

**Pages updated** (2):
- `adsa-cyclic-acyclic-graphs.md` — Added "Self-Loops: The Smallest Cycle" subsection under Definition: self-loop as smallest possible cycle, multigraph/pseudograph distinction from simple graphs, degree counting (+2 in undirected), traversal/visited-tracking implication
- `adsa-adjacency-matrix.md` — Expanded the self-loops bullet in Key Properties to note the degree-2 convention for undirected self-loops, linked to `adsa-cyclic-acyclic-graphs`

**Index updated**: no (no new pages)

---

## 2026-06-16 — Adjacency Matrix page added

**Source**: Session-built (no raw file)

**Tag**: #adsa

**Pages created** (1):
- `adsa-adjacency-matrix.md` — Definition (V×V grid, weighted vs unweighted cell values), key properties (symmetric for undirected, diagonal = self-loops, O(1) lookup, O(V) neighbor scan, O(V²) space), advantages/disadvantages, creation pseudocode, use cases (dense graphs, Floyd-Warshall, complete graphs), examples

**Pages updated** (1):
- `adsa-graphs.md` — Topic map now links `adsa-adjacency-matrix`

**Index updated**: yes

---

## 2026-06-16 — Explicit vs Implicit Graphs page added

**Source**: Session-built (no raw file)

**Tag**: #adsa

**Pages created** (1):
- `adsa-explicit-implicit-graphs.md` — Definition (pre-stored adjacency structure vs lazily-generated neighbor function), advantages/disadvantages, representation comparison table, explicit lookup vs implicit generator pseudocode, generic search pseudocode showing lazy generation + visited-by-value tracking, use cases (puzzle solving, game tree search, AI state-space planning), examples (road map vs 8-puzzle vs chess engine)

**Pages updated** (1):
- `adsa-graphs.md` — Topic map now links `adsa-explicit-implicit-graphs`

**Index updated**: yes

---

## 2026-06-16 — DAG page added; all graph pages expanded with full template

**Source**: Session-built (no raw file)

**Tag**: #adsa

**Pages created** (1):
- `adsa-dag.md` — Dedicated DAG page: definition (directed + acyclic), advantages/disadvantages, adjacency list + in-degree array representation, cycle-prevention insert pseudocode, topological sort (Kahn's algorithm) pseudocode, use cases (build systems, scheduling, prerequisites, package managers), examples (Makefile, course prereqs, npm/pip, git history)

**Pages updated** (4):
- `adsa-graphs.md` — Added Advantages, Disadvantages, Representation (adjacency list vs matrix table + diagram), Pseudocode for Creation, Use Cases sections; topic map now links `adsa-dag`
- `adsa-directed-undirected-graphs.md` — Added Advantages/Disadvantages (directed vs undirected), Representation table, creation pseudocode, Use Cases section
- `adsa-cyclic-acyclic-graphs.md` — Added Advantages/Disadvantages, Representation (cycle-detection approach), DFS cycle-detection pseudocode (directed), Use Cases section; links to new `adsa-dag` page
- `adsa-weighted-unweighted-graphs.md` — Added Advantages/Disadvantages, Representation table, weighted-edge creation pseudocode, Use Cases section; links to new `adsa-dag` page

**Index updated**: yes

---

## 2026-06-16 — Graphs split into sub-pages; weighted/unweighted added

**Source**: Session-built (no raw file)

**Tag**: #adsa

**Pages updated** (1):
- `adsa-graphs.md` — Converted to a summary/topic-map page (binary-trees pattern): what a graph is, tree as a special case, real-world examples, links to the 3 new sub-pages

**Pages created** (3):
- `adsa-directed-undirected-graphs.md` — One-way vs two-way edges, Twitter follow vs Facebook friendship examples, why direction matters for traversal/topological sort/SCCs
- `adsa-cyclic-acyclic-graphs.md` — Directed vs undirected cycle diagrams, DAG definition and 4 real-world uses (scheduling/build systems/git/spreadsheets), combined cyclic×directed classification table
- `adsa-weighted-unweighted-graphs.md` — Hop count vs total edge weight, Dijkstra/Bellman-Ford motivation, negative weights and negative-weight cycles, real-world examples table

**Index updated**: yes

---

## 2026-06-15 — Trees → Compiler Optimization cross-reference page added

**Source**: Q&A discussion

**Pages created** (1):
- `trees-compiler-optimization.md` — Maps each ADSA tree page to its compiler counterpart: AST node structure (binary trees), post/pre-order traversal for code generation, RBT for symbol tables, arena allocation for IR layout, B+ tree for SQL query optimizers; notes dominator trees as the missing next page

**Index updated**: yes

---

## 2026-06-15 — B+ Trees pages added

**Source**: Session-built (no raw file)

**Tag**: #adsa

**Pages created** (2):
- `adsa-b-plus-trees.md` — Summary page: what makes B+ trees different (data only in leaves, internal nodes as pure routing index, linked leaf chain), structure diagram for internal vs leaf nodes, separator key convention, B-tree vs B+ tree full comparison table, linked leaf layer and range query O(log n + k) explanation, order m parameters, real-world uses (MySQL InnoDB, PostgreSQL, SQLite, NTFS), topic map
- `adsa-b-plus-tree-operations.md` — Search algorithm; two split types (leaf split: copy-up first right key vs internal split: push-up middle key) with before/after diagrams; insert algorithm; insert dry run inserting 10 keys (2,3,5,7,11,17,19,23,29,31) showing leaf splits through tree height 1 then internal split at root creating height 2; delete dry run deleting 11 (underflow → merge with right sibling, separator removed from parent, stale-separator note); range query algorithm O(log n + k); leaf vs internal split summary table

**Index updated**: yes

---

## 2026-06-15 — B-Trees pages added

**Source**: Session-built (no raw file)

**Tag**: #adsa

**Pages created** (2):
- `adsa-b-trees.md` — Summary page: motivation (disk I/O cost vs RAM), minimum degree t vs order m terminology, all 6 B-tree properties, height formula O(log_t n), B-tree vs BST/AVL/RBT comparison table, real-world uses (MySQL InnoDB, PostgreSQL, NTFS), isomorphism with RBT via 2-3-4 tree encoding, topic map
- `adsa-b-tree-operations.md` — Search algorithm, insert with proactive top-down splitting (split pseudocode + insert pseudocode), insert dry run inserting 7 keys showing root split and child split; delete with 3 cases (leaf direct, internal node 2a/2b/2c, descend with underflow 3a/3b), delete dry run showing Case 3a borrow-from-sibling and Case 3b merge; strategy comparison table; height growth/shrink rules

**Index updated**: yes

---

## 2026-06-15 — Red-Black Trees pages added

**Source**: Session-built (no raw file)

**Tag**: #adsa

**Pages created** (3):
- `adsa-red-black-trees.md` — Summary page: what RBTs are, all 5 properties with explanations, black-height definition, 2 log₂(n+1) height proof (2-step: minimum nodes from BH + BH ≥ h/2), AVL vs RBT full comparison table, 2-3-4 tree isomorphism, topic map and roadmap
- `adsa-rbt-insert.md` — 3 insert fix-up cases (Case 1: Uncle Red recolor+propagate; Case 2: Triangle rotate; Case 3: Line rotate+recolor) with ASCII diagrams; full 6-key dry run inserting 41,38,31,12,19,8 triggering Case 3, Case 1, Case 2→3, Case 1; black-height verification table; complexity (≤2 rotations); Python insert code
- `adsa-rbt-delete.md` — Double-black concept; BST delete 3 sub-cases; when fix-up is skipped (deleted Red, replacement Red); 4 fix-up cases (Case 1: w Red rotate; Case 2: w Black both-children-Black recolor; Case 3: w Black near-Red far-Black rotate; Case 4: w Black far-Red rotate+recolor) with ASCII diagrams; Dry Run 1 (delete 5 from 10,5,15,20 tree → Case 4); Dry Run 2 (delete from 10,5,15 tree → Case 2 propagate to root); complexity (≤3 rotations); full Python implementation

**Index updated**: yes

---

## 2026-06-15 — AVL Trees pages added

**Source**: Session-built (no raw file)

**Tag**: #adsa

**Pages created** (3):
- `adsa-avl-trees.md` — Summary page: why AVL exists (BST degeneracy), balance factor definition, 1.44 log₂ n height guarantee (Fibonacci AVL worst case), AVL vs Red-Black Tree comparison table, topic map and roadmap
- `adsa-avl-rotations.md` — All 4 rotation cases with ASCII diagrams; detection logic (same-sign BF = single, opposite-sign = double); pseudocode for right_rotate and left_rotate; LL/RR single rotations; LR/RL double rotations step-by-step; key invariants after rotation; O(1) complexity
- `adsa-avl-operations.md` — Height/BF utility functions; insert algorithm (BST + retrace + at most 1 rotation); full 5-step insert dry run (30→20→10→25→27) triggering LL and LR cases; delete algorithm (3 BST sub-cases + multi-rotation retrace); delete dry run (remove 10, triggers RR, BF=0 special case); two-children delete dry run; complexity table; full Python implementation

**Index updated**: yes

---

## 2026-06-11 — Binary Trees pages added

**Source**: Session-built (no raw file)

**Tag**: #adsa

**Pages created** (3):
- `adsa-binary-trees.md` — Summary page: what a binary tree is, DOM as a real-world example, type taxonomy (full/complete/perfect/heap), roadmap to BSTs/heaps/balanced trees
- `adsa-binary-tree-representation.md` — Pointer (linked) vs array representation; index formula (2i+1 / 2i+2 / (i-1)//2); why sparse trees waste memory in arrays; heaps as the array-based sweet spot (complete BT + left-aligned last level = dense array); full trade-off table
- `adsa-tree-traversal.md` — Core insight (frontier DS determines order); BFS with queue (FIFO/oldest-first, level-by-level, dry-run table); DFS with stack (LIFO/newest-first, branch-by-branch); three DFS orderings (pre/in/post) with use cases; queue vs stack side-by-side comparison; random exploration (pros: escape dead-ends, MCTS; cons: non-reproducible, no shortest-path guarantee)

**Index updated**: yes

---

## 2026-06-08 — Hashing source ingested: Hashing-and-Hash-Tables.pdf

**Source**: `raw/Hashing-and-Hash-Tables.pdf` (Talenciaglobal, 52 pages — Intermediate→Advanced, FinTech-focused)

**Tag**: #adsa

**Pages created** (2):
- `adsa-hashing-advanced.md` — Perfect hashing (2-level, O(1) worst case), Cuckoo hashing (evict-cascade, O(1) worst lookup), Robin Hood (probe distance variance reduction, P99 guarantees), Hopscotch (neighbourhood bitmap, cache-friendly concurrent), Concurrent maps (global lock / lock striping / CAS / RCU), incremental rehashing (Redis-style dual-table), pre-sizing formula, history timeline 1953–2024
- `adsa-hashing-applications.md` — 3 FinTech use cases (UPI dedup with SHA-256+ConcurrentHashMap, fraud blacklist 200M entries Bloom+HashSet, KYC LRU cache HashMap+DLL), 8 algorithmic interview patterns table, 4 security considerations (HashDoS, timing side-channel, memory dump, password storage bcrypt/Argon2id), 4 architectural patterns (Cache-Aside, Idempotency Key, Token Bucket, Consistent Hashing), Hash Table vs BST/Skip List/Trie/Bloom Filter comparison, 5 common pitfalls

**Pages updated** (4):
- `adsa-hash-functions.md` — Added Direct Address Table framing, Universal Hashing formula with HashDoS context, division method pitfall (power-of-2 m), updated source citation
- `adsa-collision-resolution.md` — Added Java HashMap chain→RBT upgrade at 8 entries, expanded comparison table with max α and FinTech fit columns, updated source citation
- `adsa-python-dict.md` — Added high-bit spreading (h ^= h >> 16), language comparison table (Java/Python/C++), equals/hashCode pitfall, updated source citation
- `adsa-hashing.md` — Updated summary links to include 2 new pages, expanded roadmap to 5 steps

**Index updated**: yes

---

## 2026-06-08 — Hashing pages added

**Source**: Session-built (no raw file)

**Tag**: #adsa

**Pages created** (4):
- `adsa-hashing.md` — Summary/roadmap page: why O(1), links to all three sub-topics
- `adsa-hash-functions.md` — 5 characteristics (deterministic, uniform, fast, avalanche, low collision), load factor, division/multiplication/polynomial rolling hash methods
- `adsa-collision-resolution.md` — Chaining (LL per bucket), open addressing (linear/quadratic probing/double hashing), perturbation, tombstone deletion
- `adsa-python-dict.md` — O(1) vs unsorted/sorted array of pairs, Python compact layout (3.6+), power-of-2 sizing, bitwise AND replaces modulo, resize thresholds

**Index updated**: yes

---

## 2026-06-05 — Circular LL and Deque pages added; efficient deque internals

**Source**: Session-built (no raw file)

**Tag**: #adsa

**Pages created** (2):
- `adsa-circular-ll.md` — Circular LL definition, tail-pointer trick, singly vs doubly circular, traversal do-while stopping condition, operations table, use cases (round-robin, ring buffer, playlists), gotchas
- `adsa-deque.md` — Deque ADT (4 ops), DLL impl, circular buffer impl, unrolled linked list (node-with-array) showing why O(n/k) pointer chasing still occurs on indexed access, chunked array (dynamic array of fixed-chunk pointers) achieving true O(1) random access with only O(n/k) pointer copying on resize; full comparison table; when to use each

**Pages updated** (3): `adsa-overview.md`, `wiki/index.md`, `wiki/log.md`

---

## 2026-06-04 — Stacks page expanded: ADT, call stack, backtracking

**Pages updated**: `adsa-stacks.md`, `adsa-overview.md`
**Added**: Stack ADT operation table, call stack / activation record anatomy (SP, BP, frame layout), stack overflow from infinite recursion, backtracking with explicit stack (maze), implicit call-stack recursion (N-Queens, Sudoku), browser back button

---

## 2026-06-04 — Stacks and Queues pages added

**Source**: Session-built (no raw file)
**Pages created**: `adsa-stacks.md`, `adsa-queues.md`
**Pages updated**: `adsa-overview.md`, `index.md`
**Coverage**: LIFO/FIFO principles, dynamic-array and linked-list implementations of both, amortized O(1) analysis, dead-slot memory problem in array queues, cache-locality (1:7 vs 1:0 hit ratio) comparison

---

## 2026-06-04 — Rabin-Karp page created + KMP search dry run added

**Source**: Session-built (no raw file)

**Tag**: #adsa

**Pages created** (1):
- `rabin-karp.md` — Rolling hash theory, sliding window formula, spurious hits, pseudocode, full dry run table for "AABAAB"/"AAB" with manual rolling hash steps, Python code, KMP vs Rabin-Karp comparison table

**Pages updated** (1):
- `kmp-algorithm.md` — Added search dry run (16-step table for "AABAACDAABAAB"/"AABAAB" showing mismatch fallback cascade), fixed output comment bug ([0,6] → [0,3,6])

**Index updated**: yes

---

## 2026-06-04 — KMP Algorithm page created

**Source**: Session-built (no raw file)

**Tag**: #adsa

**Pages created** (1):
- `kmp-algorithm.md` — Why KMP exists, LPS theory (proper prefix = suffix), construction algorithm (3 cases), pseudocode, full dry run table for "AABAAB", Python code for compute_lps + kmp_search, complexity table

**Index updated**: yes

---

## 2026-06-04 — XOR Linked List page created

**Source**: Session-built (no raw file)

**Tag**: #adsa

**Pages created** (1):
- `xor-linked-list.md` — npx = id(prev) ^ id(next) theory, node structure with ctypes, insert at head, forward traversal, backward traversal, full Python example, complexity table, GC limitations

**Index updated**: yes

---

## 2026-06-03 — ADSA arrays: Kadane's algorithm added

**Tag**: #adsa

**Pages updated** (1):
- `adsa-arrays.md` — Added Kadane's algorithm: max subarray, greedy local decision, pseudocode, all-negative edge case, O(n)/O(1) complexity

---

## 2026-06-03 — ADSA arrays: prefix sum + difference array added

**Tag**: #adsa

**Pages updated** (1):
- `adsa-arrays.md` — Added: prefix sum array (O(1) range query, construction, constraint), difference array (O(1) range update, reconstruction, cost comparison table), updated summary table to cover all four array types

**Index updated**: no (page already indexed)

---

## 2026-06-03 — ADSA arrays page created

**Source**: Session-built (no raw file)

**Tag**: #adsa

**Pages created** (1):
- `adsa-arrays.md` — Static arrays, dynamic array reallocation, amortized O(1) append (accounting method), growth factor trade-off table, operation complexity table

**Pages updated** (1):
- `adsa-overview.md` — Added Arrays section link

**Index updated**: yes

---

## 2026-06-03 — ADSA concept initialized

**Source**: Session-built (no raw file)

**Tag**: #adsa

**Pages created** (1):
- `adsa-overview.md` — Shell page; will grow as Q&A sessions populate it

**Index updated**: yes

---

## 2026-05-12 — Unit 3 revision created + Unit 2 revision completed

**Source**: `Unit III - Data Science.pptx.pdf`

**Tag**: #DS

**Pages created** (1):
- `unit3-revision.md` — 18 sections: static vs interactive, Matplotlib components, bar chart, subplots, annotations/savefig, Seaborn scatter (day vs tip), histogram+KDE, boxplot, pairplot, univariate numerical, univariate categorical, compare-groups (6 chart types), distribution charts (4 types), multi-plot dashboard, 3D surface, anomaly detection visualization, Seaborn vs Matplotlib comparison table, quick reference card

**Pages updated** (1):
- `unit2-revision.md` — Sections 6–11 rewritten in unit1 format with verified outputs: transformation techniques (6 types with code), binning (pd.cut/pd.qcut with outputs), normalization vs standardization (formulas + output table), string manipulation (basic + pandas vectorised), summarizing (centrality/dispersion/describe), outlier detection (IQR + Z-score with fence values and output)

**Index updated**: yes

---

## 2026-05-12 — Unit 2 revision + exam roadmap created

**Source**: `Unit II - Data Science.pptx.pdf` (113 pages), `Question Bank_Data Science.docx`

**Tag**: #DS

**Pages created** (2):
- `unit2-revision.md` — Progressive study notes with full code: data wrangling 6-step cycle, clean→transform→merge pipeline, combining datasets (merge/concat/combine_first/join), reshaping (pivot/melt/stack/unstack), missing data handling, 6 transformation techniques, binning (pd.cut/pd.qcut), normalization vs standardization, string manipulation, summarizing (centrality/dispersion/distribution), outlier detection (IQR + Z-score)
- `unit2-exam-roadmap.md` — QB-mapped priority tiers: coverage map table for all 16 Unit 2 QB questions, Tier 1 (11 must-know cold with full code), Tier 2 (6 know-well with code), Tier 3 (short recall), quick reference card

**Index updated**: yes

---

## 2026-05-11 — Unit 1 revision page created

**Page created**: `unit1-revision.md`

**Content**: Progressive study notes built during learning session. Covers argsort, lexsort (strings + complex arrays), transpose, reshape, fillna with mean/mode, rank.

**Index updated**: yes

---

## 2026-05-11 — Data Science question bank expansion

**Source**: `Question Bank_Data Science.docx`

**Tag**: #DS

**Pages updated** (5 existing pages expanded):
- `numpy.md` — added: transpose, argsort, lexsort (multi-column sort), complex array sort, first/last name sort, array equality (array_equal vs allclose), NumPy vs Pandas comparison table
- `pandas.md` — added: CSV read + describe/info, multiple condition selection (&/|/~), Web API→DataFrame, BeautifulSoup web scraping, open data vs API comparison, MultiIndex hierarchy, pivot_table, groupby agg, business dates (BusinessDay/BusinessMonthEnd)
- `missing-data-handling.md` — added: full IQR outlier detection code, full Z-score detection code, IQR vs Z-score comparison table, impact of improper outlier handling
- `data-transformation.md` — added: standardization vs normalization clear distinction table, binning with frequency analysis (pd.cut/qcut + value_counts), why transformation is critical before modeling, full data cleaning code example
- `seaborn-visualization.md` — added: static vs interactive, customized bar chart, histogram+KDE, boxplot by group, day vs tip scatter, plot styling (annotations/fonts/colors), univariate guide (numerical + categorical), bivariate guide, compare-groups charts (6 types), distribution charts (4 types), multi-plot dashboard, anomaly detection with visualization

**Index updated**: no (pages already indexed)

---

## 2026-05-11 — Data Science course ingestion (21CSS303T)

**Sources**: `Unit I- Data Science.pptx.pdf` (161 pages), `Unit II - Data Science.pptx.pdf` (113 pages), `Unit III - Data Science.pptx.pdf` (91 pages), `Question Bank_Data Science.docx`

**Tag**: #DS

**Pages created** (11):
- `data-science-overview.md` — Summary page: topic map + learning roadmap
- `data-science-process.md` — Big Data 3 V's, 6-step DS process, statsmodels OLS, facets of data
- `data-storage-hierarchy.md` — Database/DM/DWH/Data Lake hierarchy, comparisons, open data providers
- `data-preparation.md` — Cleansing, transformation, combining; the data prep pipeline
- `missing-data-handling.md` — Detection, listwise/pairwise deletion, 6 imputation methods, outlier types, boxplot/IQR/Z-score
- `numpy.md` — ndarray, 10+ creation functions, identity vs eye, indexing/slicing, copying (3 methods), nditer, reshape, flatten
- `pandas.md` — Series and DataFrame creation/indexing/ops, alignment, NaN handling, groupby, rank (4 methods), sort
- `data-wrangling.md` — 6-step wrangling cycle, large data libraries, merge/concat/join/combine_first, pivot/melt/stack/unstack
- `data-transformation.md` — 6 transformation types, string ops, summarizing (centrality/dispersion/distribution), binning (3 types), Z-score/Min-Max standardization
- `matplotlib.md` — Component model, subplots, axes/ticks/labels/legends, annotations, savefig, full example
- `seaborn-visualization.md` — Scatter/line/histogram/boxplot/pairplot/wordcloud, 3D scatter/line/surface with mpl_toolkits

**Index updated**: yes

---

## 2026-05-10 — Biometrics course ingestion (6 source files)

**Sources**: Unit1_Biometrics(Part-1).pptx, Updated Unit1_Biometrics(Part-2).pptx, Unit2_Biometrics(1).pptx, Unit3_Biometrics(1).pptx, Unit4_Slides.pdf, Unit5_Slides.pdf

**Tag**: #biometrics

**Pages created** (17):
- `biometrics-overview.md` — Summary page: topic map + recommended learning order
- `biometric-fundamentals.md` — Pipeline, verification vs identification, gallery vs probe, error rates
- `biometric-traits.md` — 7 factors, 5 trait categories, physiological vs behavioral
- `biometric-system-modules.md` — 5 modules, enrollment vs authentication flow
- `image-processing-basics.md` — 10 steps, image types, point operations, interpolation, thresholding
- `image-enhancement.md` — Negatives, contrast stretching, CDR, gray-level slicing, histogram equalization
- `image-segmentation.md` — Threshold, Sobel/Canny, region growing, split/merge, k-means, ANN
- `fingerprint-sensors.md` — FTIR, capacitance, ultrasound, temperature, piezoelectric; plain/rolled/sweep
- `fingerprint-feature-extraction.md` — 4-step algorithm: orientation, ridge extraction, Poincaré index, binarization
- `fingerprint-recognition.md` — L1/L2/L3 levels, pattern types, minutiae matching, AFIS, latent vs live-scan
- `pattern-recognition.md` — Full pipeline, feature vectors, classifiers, Bayes, decision theory
- `dimensionality-reduction.md` — PCA vs LDA, Fisher's criterion, worked PCA example, discriminant function
- `multibiometrics.md` — 5 evidence sources, 3 acquisition modes, multi-algorithmic/instance/sensorial
- `biometric-fusion.md` — All 5 fusion levels with formulas; score-level taxonomy; Borda count
- `biometric-security.md` — 4 threats, 5 vulnerability areas, spoofing/MITM/replay/hill-climbing
- `template-protection.md` — 3 properties, 4 approaches, Table 7.1, key-binding vs key-generating
- `biometric-applications.md` — Access control, airport, Aadhaar, banking, AFIS, embedded, match-on-card

**Index updated**: yes

---

## 2026-04-16 — Session start: C++ from scratch

**Source**: https://en.cppreference.com/w/cpp (devdocs.io/cpp mirrors this)

**Tag**: #program

**Pages created**:
- `cpp-introduction.md` — What C++ is, why it matters for systems dev, compilation pipeline, learning roadmap
- `cpp-program-structure.md` — Translation units, main(), scope, name lookup, storage duration, linkage, ODR, header guards
- `cpp-fundamental-types.md` — All built-in types: integers (signed/unsigned, fixed-width), floats (IEEE-754), bool, char variants, void, nullptr
- `cpp-declarations-definitions.md` — Declaration vs definition, ODR, const/volatile qualifiers, auto/static/extern/thread_local/mutable storage class specifiers

**Index updated**: yes

---

## 2026-04-16 — Rust OS development learning roadmap

**Source**: https://doc.rust-lang.org/book/, https://os.phil-opp.com/

**Pages created**:
- `rust-learning-roadmap.md` — 2-day sprint plan covering ownership, traits, unsafe Rust, no_std, and the phil-opp OS tutorial

**Index updated**: yes

---

## 2026-04-16 — Rust Macros

**Source**: `oWiki/Rust from start.md`

**Pages created**:
- `rust-macros.md` — What macros are, why they exist, how `println!` works, common macros reference table

**Index updated**: yes

---

## 2026-04-16 — Full Rust Knowledge Base

**Source**: https://doc.rust-lang.org/book/ (official Rust Book, chapters 3–10)

**Tag**: #rust

**Pages created**:
- `rust-overview.md` — Summary page: topic map, why Rust exists, comparison table with C/C++/Go, recommended learning order
- `rust-variables-types.md` — Variables, mutability, shadowing, constants, all scalar types (integers, floats, bool, char), compound types (tuples, arrays), overflow behavior
- `rust-functions-control-flow.md` — Function syntax, snake_case convention, parameters, statements vs expressions, implicit returns, `if` as expression, `loop`/`while`/`for`, loop labels, range iteration
- `rust-ownership.md` — Stack vs heap, three ownership rules, move semantics, clone, `Copy` trait, scope and drop, ownership through functions
- `rust-borrowing-references.md` — References (`&T`, `&mut T`), borrowing rules, NLL (non-lexical lifetimes), dangling reference prevention, dereferencing
- `rust-structs.md` — Struct definition, field init shorthand, struct update syntax, tuple structs, unit-like structs, `impl` blocks, methods, associated functions
- `rust-enums-pattern-matching.md` — Enum definition, variants with data, methods on enums, `Option<T>` replacing null, `match` expressions, exhaustive matching, catch-all patterns, `if let`
- `rust-error-handling.md` — `Result<T, E>`, `match` on Result, `unwrap`, `expect`, `?` operator, error propagation, `panic!`, when to use each
- `rust-traits-generics.md` — Generic functions/structs/enums/methods, monomorphization (zero-cost), trait definition, implementing traits, orphan rule, default implementations, trait bounds (`+`, `where`), `impl Trait`, blanket implementations, common stdlib traits

**Index updated**: yes

---

## 2026-04-18 — Rust Strings

**Source**: https://doc.rust-lang.org/book/ch08-02-strings.html

**Tag**: #rust

**Pages created**:
- `rust-strings.md` — `&str` vs `String`, UTF-8 encoding, why integer indexing is disallowed, slicing, `.chars()` vs `.bytes()`, common methods, coercion rules

**Index updated**: yes

---

## 2026-04-18 — OS Development Knowledge Cluster

**Source**: `raw/Expanded Main Page - OSDev Wiki.md`

**Tag**: #os

**Pages created**:
- `os-overview.md` — Summary/topic map: learning order, all concept links, key decision points (language, kernel model, bootloader)
- `os-toolchain.md` — GCC cross-compiler (x86_64-elf target), NASM/GAS/FASM, LD linker scripts, QEMU/Bochs emulator setup and debugging
- `os-x86-modes.md` — Real Mode, Protected Mode (rings, GDT), Long Mode (x86-64), mode transition code, CPU register reference table
- `os-boot-sequence.md` — BIOS vs UEFI, MBR/GPT, GRUB/Limine protocols, GDT/TSS setup, A20 line, early kernel init checklist
- `os-interrupts.md` — IDT, gate types, exception vectors (PF, GPF, DF), PIC remapping, APIC/LAPIC/IO-APIC, NMI, ISR rules, syscall (int 0x80 vs SYSCALL MSR)
- `os-kernel-models.md` — Monolithic vs microkernel vs exokernel vs modular, comparison table, task models (mono/multi/RT)
- `os-memory-management.md` — Segmentation, 4-level paging, TLB, page frame allocators (bitmap/stack/buddy), heap allocators (bump/free-list/slab), MMU registers, virtual address layout
- `os-scheduling.md` — Process vs thread, PCB fields, context switch mechanics (iretq), FCFS/round-robin/priority/MLFQ/CFS algorithms, SMP load balancing, blocking/wait queues
- `os-ipc.md` — Message passing (pipes/queues/sockets), shared memory, signals, RPC, spinlock/mutex/semaphore/condvar, deadlock (Coffman conditions)
- `os-hardware.md` — I/O ports vs MMIO, PIT/APIC timer/HPET, ATA/AHCI/NVMe storage, VGA text mode/linear framebuffer, PS/2 keyboard+mouse, PCI enumeration, ACPI tables (RSDP/MADT/MCFG)

**Index updated**: yes

---

## 2026-04-25 — GitHub Copilot Certification (GH-300)

**Source**: https://learn.microsoft.com/credentials/certifications/resources/study-guides/gh-300 (web, no raw file)

**Pages created**:
- `github-copilot-certification.md` — Summary page: exam details, domain weights, recommended study order, official resources
- `copilot-responsible-ai.md` — Risks and limitations of GenAI, ethical usage, harm mitigation, validating AI output
- `copilot-features.md` — Copilot in IDE, CLI, Agent Mode, Edit Mode, MCP, Sub-Agents, Spaces, Spark, PR summaries, org-wide management
- `copilot-data-architecture.md` — Data flow and sharing, prompt construction, proxy filtering, post-processing, suggestion lifecycle, LLM limits
- `copilot-prompt-engineering.md` — Prompt structure, context determination, zero-shot/few-shot, best practices, chat history, prompt file reuse
- `copilot-productivity.md` — Code gen, refactoring, documentation, sample data, legacy modernization, unit/integration tests, security, performance
- `copilot-privacy-safeguards.md` — Content exclusions, editor settings, output ownership, duplication detection, security warnings, troubleshooting

**Index updated**: yes

---

## 2026-04-18 — Agentic AI knowledge cluster (session start)

**Source**: Session-built (no raw file yet)

**Tag**: #agentic

**Pages created**:
- `agentic-ai-overview.md` — Summary/topic map page: all concept links, recommended reading order, roadmap
- `agentic-agent-loop.md` — The core agent loop pattern, ReAct/Reflexion/Plan-execute variants, failure modes
- `agentic-multi-agent.md` — Full pipeline: goal → planner → parallel workers → reviewer → final output, failure modes table
- `agentic-langchain.md` — LangChain & LangGraph overview: chains, agents, tools, memory, state machine orchestration, vs rolling your own

**Index updated**: yes

## 2026-06-16 — Indegree/Outdegree (ADSA graphs)

**Source**: Session-built (no raw file)

**Tag**: #adsa

**Pages created**:
- `adsa-indegree-outdegree.md` — Definition, cost of computing from adjacency list vs matrix, handshake identity, Kahn's algorithm tie-in

**Pages updated**:
- `adsa-graphs.md` — Added to topic map and roadmap

**Index updated**: yes

## 2026-06-16 — BFS/DFS on graphs (ADSA)

**Source**: Session-built (no raw file)

**Tag**: #adsa

**Pages created**:
- `adsa-bfs.md` — Definition, illustration (dry run), advantages, disadvantages, example, use cases
- `adsa-dfs.md` — Definition, illustration (dry run), advantages, disadvantages, example, use cases

**Pages updated**:
- `adsa-graphs.md` — Added both to topic map; roadmap step 4 now links to both instead of "_next topic_"

**Index updated**: yes

## 2026-06-16 — Topological Sort (ADSA)

**Source**: Session-built (no raw file)

**Tag**: #adsa

**Pages created**:
- `adsa-toposort.md` — Definition (DAG-only ordering), Kahn's algorithm (BFS, in-degree based) with dry run, DFS-based method (finish-time stack reversed) with dry run, comparison table, use cases

**Pages updated**:
- `adsa-graphs.md` — Added to topic map and roadmap (step 4a)
- `adsa-dag.md` — Linked to dedicated toposort page from its existing Kahn's algorithm section

**Index updated**: yes

## 2026-06-16 — Flood Fill (ADSA)

**Source**: Session-built (no raw file)

**Tag**: #adsa

**Pages created**:
- `adsa-flood-fill.md` — Definition (implicit grid graph), grid dry run, advantages, disadvantages, example, use cases

**Pages updated**:
- `adsa-bfs.md` — Added flood fill as a use case
- `adsa-dfs.md` — Added flood fill as a use case

**Index updated**: yes
