# Wiki Index

Personal knowledge base — maintained by Claude Code.

---

## Rust — OS Development Track

- [[rust-overview]] — Full Rust topic map, why Rust exists, comparison with C/C++/Go
- [[rust-learning-roadmap]] — 2-day sprint plan to learn Rust core before starting OS dev
- [[rust-variables-types]] — Variables, mutability, shadowing, constants, scalar and compound types
- [[rust-functions-control-flow]] — Function syntax, statements vs expressions, `if`, `loop`, `while`, `for`
- [[rust-ownership]] — Ownership rules, move semantics, `Copy` trait, stack vs heap
- [[rust-borrowing-references]] — References, borrowing rules, mutable references, no dangling pointers
- [[rust-structs]] — Struct definition, field init shorthand, update syntax, tuple structs, `impl` blocks
- [[rust-enums-pattern-matching]] — Enums, `Option<T>`, `match`, exhaustive patterns, `if let`
- [[rust-error-handling]] — `Result<T, E>`, `?` operator, `unwrap`, `expect`, `panic!`
- [[rust-traits-generics]] — Traits, default implementations, generics, trait bounds, monomorphization
- [[rust-macros]] — Metaprogramming in Rust: how macros work, `println!` explained, common macros
- [[rust-strings]] — `&str` vs `String`, UTF-8 rules, why integer indexing is banned, slicing, iteration, common methods

---

## C++ — Learning from Scratch

- [[cpp-introduction]] — C++ overview and learning roadmap (#program)
- [[cpp-program-structure]] — How a C++ program is structured and compiled (#program)
- [[cpp-fundamental-types]] — All built-in types: integers, floats, bool, char, void (#program)
- [[cpp-declarations-definitions]] — Declarations vs definitions, qualifiers, storage classes (#program)

---

---

## Agentic AI

- [[agentic-ai-overview]] — Full topic map for agentic AI: agent loop, tools, planning, memory, multi-agent, eval, safety (#agentic)
- [[agentic-agent-loop]] — The observe → think → act cycle; loop variants, stopping conditions, failure modes (#agentic)
- [[agentic-multi-agent]] — Pipeline pattern: planner → parallel workers → reviewer → aggregated output (#agentic)
- [[agentic-langchain]] — LangChain & LangGraph: framework abstractions for chains, tools, memory, and multi-agent state machines (#agentic)

---

## OS Development

- [[os-overview]] — Full topic map and recommended learning order for building an OS from scratch (#os)
- [[os-toolchain]] — GCC cross-compiler, NASM, LD linker scripts, QEMU/Bochs emulators (#os)
- [[os-x86-modes]] — Real Mode → Protected Mode → Long Mode (x86-64), CPU registers reference (#os)
- [[os-boot-sequence]] — BIOS/UEFI → GRUB/Limine → kernel entry, GDT, TSS, A20 line (#os)
- [[os-interrupts]] — IDT, ISRs, exceptions, PIC, APIC, NMI, timer, syscall interface (#os)
- [[os-kernel-models]] — Monolithic vs microkernel vs exokernel vs modular; task models (#os)
- [[os-memory-management]] — Segmentation, paging, page frame allocator, heap, MMU, virtual address layout (#os)
- [[os-scheduling]] — Processes vs threads, PCB, context switching, scheduling algorithms, SMP (#os)
- [[os-ipc]] — Message passing, shared memory, signals, RPC, semaphores, mutexes, deadlock (#os)
- [[os-hardware]] — I/O ports, MMIO, storage (ATA/AHCI/NVMe), video, input, PCI, ACPI (#os)

---

---

## GitHub Copilot Certification (GH-300)

- [[github-copilot-certification]] — Exam overview, domain weights, study order, and official resources
- [[copilot-responsible-ai]] — Risks of generative AI, ethical usage, harm mitigation, validating output (15–20%)
- [[copilot-features]] — IDE, CLI, Agent Mode, Edit Mode, MCP, Spaces, Spark, PR summaries, org policies (25–30%)
- [[copilot-data-architecture]] — Data flow, prompt building, proxy filtering, suggestion lifecycle, LLM limits (10–15%)
- [[copilot-prompt-engineering]] — Prompt structure, zero-shot/few-shot, context crafting, chat history (10–15%)
- [[copilot-productivity]] — Code gen, refactoring, docs, testing, security, legacy modernization (10–15%)
- [[copilot-privacy-safeguards]] — Content exclusions, duplication detection, security warnings, org controls (10–15%)

---

---

## Biometrics

- [[biometrics-overview]] — Full topic map and recommended learning order for the biometrics course (#biometrics)
- [[biometric-fundamentals]] — System pipeline, verification vs identification (1:1 / 1:N), gallery vs probe, error rates (#biometrics)
- [[biometric-traits]] — 7 trait factors (Universality → Circumvention), 5 trait taxonomy categories (#biometrics)
- [[biometric-system-modules]] — 5 modules: sensor, quality assessment, feature extraction, matching, database (#biometrics)
- [[image-processing-basics]] — 10 fundamental steps, image types, point operations, interpolation, thresholding (#biometrics)
- [[image-enhancement]] — Negatives, contrast stretching, CDR, gray-level slicing, histogram equalization (#biometrics)
- [[image-segmentation]] — Threshold, edge-based (Sobel/Canny), region-based (split/merge), clustering, ANN (#biometrics)
- [[fingerprint-sensors]] — FTIR, capacitance, ultrasound, temperature differential, piezoelectric; plain/rolled/sweep (#biometrics)
- [[fingerprint-feature-extraction]] — 4-step algorithm: orientation, ridge extraction, Poincaré index, binarization (#biometrics)
- [[fingerprint-recognition]] — Feature levels L1/L2/L3, pattern types, minutiae matching, AFIS (#biometrics)
- [[pattern-recognition]] — Full pipeline, feature vectors, classifiers, Bayes, normative vs descriptive decision theory (#biometrics)
- [[dimensionality-reduction]] — PCA (unsupervised, max variance) vs LDA (supervised, Fisher's criterion), worked example (#biometrics)
- [[multibiometrics]] — 5 evidence sources, 3 acquisition modes, multi-algorithmic/instance/sensorial types (#biometrics)
- [[biometric-fusion]] — Fusion at sensor/feature/score/rank/decision level; sum/product/min/max; Borda count (#biometrics)
- [[biometric-security]] — 4 threats, 5 vulnerability areas (A–E), spoofing/MITM/replay/hill-climbing attacks (#biometrics)
- [[template-protection]] — 3 required properties, 4 approaches, Table 7.1 full comparison (#biometrics)
- [[biometric-applications]] — Access control, airport/immigration, Aadhaar, banking, AFIS, match-on-card (#biometrics)

---

## Data Science (21CSS303T)

- [[data-science-overview]] — Full topic map and learning roadmap for the SRM Data Science course (#DS)
- [[data-science-process]] — Big Data 3 V's, 6-step DS process, model execution with statsmodels (#DS)
- [[data-storage-hierarchy]] — Database → Data Mart → DWH → Data Lake; comparisons; open data providers (#DS)
- [[data-preparation]] — Data cleansing, transformation, combining; the full prep pipeline (#DS)
- [[missing-data-handling]] — Detection, deletion (listwise/pairwise), imputation, outlier types, IQR/Z-score (#DS)
- [[numpy]] — ndarray, creation functions, indexing, copying, iterating, identity/eye, reshape, flatten (#DS)
- [[pandas]] — Series, DataFrame, indexing (loc/iloc/at), rank, sort, alignment, groupby (#DS)
- [[data-wrangling]] — 6-step wrangling cycle, large data libs, merge/concat/join/combine_first, pivot/melt/stack (#DS)
- [[data-transformation]] — Cleaning steps, smoothing/generalization/aggregation, binning, Z-score/Min-Max (#DS)
- [[matplotlib]] — Components, subplots, axes/ticks/labels/legends, annotations, savefig (#DS)
- [[seaborn-visualization]] — Scatter/line/boxplot/pairplot/wordcloud, 3D scatter/line/surface plots (#DS)
- [[unit1-revision]] — Progressive revision notes: argsort, lexsort, transpose, reshape, fillna, rank (#DS)
- [[unit1-exam-roadmap]] — QB-mapped priority tiers for Unit 1: NumPy sorting, Pandas ops, DS process (#DS)
- [[unit2-revision]] — Progressive revision notes: wrangling, clean/merge/reshape, binning, outliers, strings (#DS)
- [[unit2-exam-roadmap]] — QB-mapped priority tiers for Unit 2: merge, pivot, IQR, Z-score, normalization (#DS)
- [[unit3-revision]] — Progressive revision notes: Matplotlib, Seaborn, subplots, distributions, 3D, dashboards (#DS)

---

---

## Advanced Data Structures and Algorithms

- [[adsa-overview]] — Overview page; populated via Q&A (#adsa)
- [[adsa-arrays]] — Static arrays, dynamic reallocation, amortized O(1) append (#adsa)
- [[adsa-stacks]] — LIFO, array-backed vs LL-backed, amortized O(1) push, O(1) pop (#adsa)
- [[adsa-queues]] — FIFO, array-backed dead-slot problem, LL head+tail, cache-miss analysis (#adsa)
- [[adsa-circular-ll]] — Circular LL, tail-pointer trick, singly vs doubly circular, traversal gotchas (#adsa)
- [[adsa-deque]] — Deque ADT, circular buffer, unrolled LL, chunked array with O(1) random access (#adsa)
- [[xor-linked-list]] — Memory-efficient doubly linked list using XOR of addresses as a single npx pointer (#adsa)
- [[kmp-algorithm]] — LPS array construction, pseudocode, dry run, search dry run with fallback, O(n+m) implementation (#adsa)
- [[rabin-karp]] — Rolling hash theory, spurious hits, pseudocode, dry run, KMP vs Rabin-Karp comparison (#adsa)
- [[adsa-hashing]] — Hash table overview and roadmap: O(1) access, hash functions, collision resolution (#adsa)
- [[adsa-hash-functions]] — 5 characteristics: deterministic, uniform, fast, avalanche, low collision; load factor (#adsa)
- [[adsa-collision-resolution]] — Chaining vs open addressing; linear, quadratic (perturbation), double hashing; tombstones (#adsa)
- [[adsa-python-dict]] — O(1) vs array of pairs; Python compact layout; bitwise AND replaces modulo for power-of-2 tables (#adsa)
- [[adsa-hashing-advanced]] — Perfect hashing, Cuckoo hashing, Robin Hood, Hopscotch, concurrent maps (CAS/RCU), incremental rehashing (#adsa)
- [[adsa-hashing-applications]] — UPI dedup, fraud blacklist, KYC LRU cache; 8 interview patterns; HashDoS/password security; Cache-Aside/Consistent Hashing (#adsa)
- [[adsa-binary-trees]] — Binary tree overview: DOM example, types (full/complete/heap), topic map for trees (#adsa)
- [[adsa-binary-tree-representation]] — Pointer vs array storage, index formula, heap shape guarantee, memory trade-off table (#adsa)
- [[adsa-tree-traversal]] — BFS (queue/oldest-first), DFS (stack/newest-first), three DFS orderings, random exploration (#adsa)
- [[adsa-avl-trees]] — AVL tree overview: balance factor, 1.44 log n height guarantee, AVL vs Red-Black comparison (#adsa)
- [[adsa-avl-rotations]] — All 4 rotation cases (LL/RR single, LR/RL double) with ASCII diagrams and pseudocode (#adsa)
- [[adsa-avl-operations]] — Insert and delete algorithms, two full dry runs (LR case + delete with rebalance), Python implementation (#adsa)
- [[adsa-red-black-trees]] — RBT overview: 5 properties, black-height, 2 log n height proof, 2-3-4 isomorphism, AVL vs RBT (#adsa)
- [[adsa-rbt-insert]] — 3 insert fix-up cases (Uncle Red, Triangle, Line) with diagrams, full 6-key dry run showing all cases, Python code (#adsa)
- [[adsa-rbt-delete]] — Double-black concept, 4 delete fix-up cases, two dry runs (Case 4 + Case 2), complete Python implementation (#adsa)
- [[adsa-b-trees]] — B-tree overview: disk I/O motivation, min-degree t, 6 properties, height formula, comparison with BST/AVL/RBT, isomorphism with 2-3-4/RBT (#adsa)
- [[adsa-b-tree-operations]] — Search, insert (proactive split), delete (proactive fill/merge); two full dry runs with t=2 (#adsa)

---

**Last updated**: 2026-06-15
