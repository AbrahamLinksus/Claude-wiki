# Stacks

**Summary**: Stack ADT via dynamic arrays and linked lists — LIFO principle, O(1) push/pop, the call stack, backtracking, expression evaluation, balanced parentheses, infix-to-postfix conversion, and overflow/underflow guards.

**Pre-Req**: [[adsa-arrays]] — dynamic array reallocation and amortized analysis

**Sources**: Session-built (2026-06-04), `1 - Stack-and-Queue-Using-Arrays-and-Linked-Lists.pdf`, `2 - Stack-LIFO-Function-Calls-and-Backtracking.pdf`

**Last updated**: 2026-06-09

---

## LIFO Principle

A stack is **Last In, First Out** — the most recently pushed element is always popped first. Think of a stack of plates.

```
Push →  [ 10 | 20 | 30 | 40 ]
                         ↑ top
         ← Pop always comes from here
```

Two operations:
- **push(x)** — add to the top
- **pop()** — remove and return the top

---

## Stack using a Dynamic Array

Maintain a dynamic array + a `top` index pointing at the last occupied slot.

```
top = 3
index:  0    1    2    3
      [ 10 | 20 | 30 | 40 ]
                        ↑ top
```

**Push**: write at `arr[top+1]`, increment `top`.
- **Amortized O(1)** — occasional resize doubles the array (O(n) copy), but spread across n pushes the average cost per push stays O(1). (See [[adsa-arrays]].)

**Pop**: read `arr[top]`, decrement `top`. The slot stays in memory but is ignored.
- **O(1) strictly** — no shifting, no reallocation.

```
After pop:
top = 2
index:  0    1    2    3
      [ 10 | 20 | 30 | ?? ]
                   ↑ top
```

Space: O(n), up to 2× wasted capacity due to growth-factor doubling.

---

## Stack using a Linked List

`head` is the top. Each node stores a value and a pointer to the node below it (its predecessor's address).

```
head
 ↓
[40|*]→[30|*]→[20|*]→[10|NULL]
 top                    bottom
```

**Push**: allocate new node, `new.next = head`, `head = new`. **O(1)**.

**Pop**: save `head.val`, `head = head.next`. **O(1)**.

No resizing needed — memory grows one node at a time, no wasted capacity.

**Cost**: every node carries a pointer (8 bytes on 64-bit). Nodes sit at random addresses, so traversal hammers the cache. See [[adsa-queues]] for the cache-miss analysis.

---

## Stack as an ADT

An **Abstract Data Type** defines the *interface* — what operations exist and what they promise — without specifying how they are implemented. The implementation (array or linked list) is hidden behind the contract.

Stack ADT contract:

| Operation | Description | Complexity |
|---|---|---|
| `push(x)` | Add x to the top | O(1) amortized |
| `pop()` | Remove and return top | O(1) |
| `peek()` | Read top without removing | O(1) |
| `isEmpty()` | True if no elements | O(1) |
| `size()` | Number of elements | O(1) |

Any code that uses a stack only cares about this contract. Whether it runs on an array or a linked list underneath is an implementation detail — swapping one for the other doesn't break the caller.

---

## Stack Overflow and Underflow

These are the two fundamental error conditions for any stack. Both must be guarded explicitly (source: `2 - Stack-LIFO-Function-Calls-and-Backtracking.pdf`).

### Stack Overflow

**What it is**: Attempting to push onto a stack that is already full.

**When it occurs**:
- On a static (fixed-size) array stack: when `top == capacity - 1` and you call push.
- On the hardware call stack: when the chain of active function calls grows too deep, exceeding the OS-allocated stack memory segment.

**The classic trigger**: infinite recursion with no base case.

```
// This function never stops pushing frames
def infinite(n):
    return infinite(n + 1)   # no base case → stack overflow

Recursion depth limits:
  Python:  ~1,000 frames (configurable via sys.setrecursionlimit)
  Java:    ~500 frames (configurable via -Xss flag)
  C/C++:   ~1 MB stack per thread on Windows
```

**How to guard** (array stack):
```
push(x):
    if top == capacity - 1:
        raise StackOverflowError   // or resize if dynamic
    top++
    arr[top] = x
```

**How to guard** (recursion):
```
solve(state, depth):
    if depth > MAX_DEPTH:
        raise RecursionLimitError
    ...
    solve(next_state, depth + 1)
```

### Stack Underflow

**What it is**: Attempting to pop or peek from an empty stack.

**When it occurs**:
- When you dequeue faster than you enqueue — a consumer bug.
- When bracket matching pops on a closing bracket but the stack is already empty.
- When expression evaluation encounters an operator but there are not enough operands.

**How to guard**:
```
pop():
    if top == -1:          // or isEmpty()
        raise StackUnderflowError
    val = arr[top]
    top--
    return val

peek():
    if isEmpty():
        raise StackUnderflowError
    return arr[top]
```

**Summary**:

| Condition | Trigger | Guard |
|---|---|---|
| Overflow | push when full | Check `top == capacity - 1` before push |
| Underflow | pop/peek when empty | Check `isEmpty()` before pop/peek |

---

## The Call Stack (Activation Records)

The CPU maintains a hardware stack in memory called the **call stack**. Every function call pushes an **activation record** (stack frame) onto it; every return pops one off.

```
                    ┌─────────────────┐  ← Stack grows downward
                    │  main() frame   │
                    ├─────────────────┤
                    │   foo() frame   │
                    ├─────────────────┤
  SP (stack ptr) →  │   bar() frame   │  ← current frame (top)
                    └─────────────────┘
```

Each frame holds:
- **Return address** — where to jump after the function returns
- **Parameters** — arguments passed to the function
- **Local variables** — variables declared inside the function
- **Saved registers** — CPU register state to restore on return

Two CPU registers manage this:
- **SP (Stack Pointer)** — always points at the top of the stack (the current frame's top)
- **BP/FP (Base/Frame Pointer)** — anchors the bottom of the current frame so local variables can be addressed at fixed offsets

**Call**: SP decrements, new frame is written.
**Return**: SP increments (frame is "popped"), control jumps to the saved return address.

This is why **infinite recursion causes a stack overflow** — each recursive call pushes a new frame but nothing ever pops, until the stack runs into other memory.

```
main() calls f(1)
  f(1) calls f(2)
    f(2) calls f(3)
      ...
        f(n) → no base case → stack overflow
```

---

## Stacks in Backtracking

Backtracking is a strategy: explore a path, and if it leads to a dead end, **undo** the last choice and try another. The LIFO property of a stack maps directly onto this — "undo the most recent decision first."

### Explicit stack (iterative DFS / maze solving)

```
Maze:
S . # .
. # . .
. . . E

Push start S.
At each step: push unvisited neighbours.
Hit a dead end: pop (backtrack) to the last branching point.
Repeat until E is reached or stack is empty (no path).
```

```
Stack state while exploring:
[S] → [S, A] → [S, A, B] → dead end → [S, A] → [S, A, C] → ... → [E]
```

### Implicit stack (recursive DFS)

When you write a recursive backtracking function, the **call stack is the backtracking stack**. Each recursive call is a "push"; each return is a "pop" back to the previous decision point.

```
solve(state):
    if goal(state): return true
    for each choice c:
        apply(c)
        if solve(state): return true
        undo(c)          ← this is the backtrack / "pop"
    return false
```

The OS's call stack holds the chain of `apply` decisions automatically — you don't manage a stack explicitly, the hardware does it for you.

### Other backtracking use cases
- **N-Queens** — place a queen, recurse, remove it if it causes a conflict
- **Sudoku solver** — fill a cell, recurse, erase if the board becomes invalid
- **Browser back button** — each visited page is pushed; back pops to the previous

---

## Expression Evaluation: Postfix (RPN)

**Postfix notation** (also called Reverse Polish Notation, RPN) writes operators *after* their operands. There are no parentheses needed — operator precedence is encoded in position.

Example: infix `(3 + 4) * 5 - 2` becomes postfix `3 4 + 5 * 2 -`

### Algorithm

Scan tokens left to right:
- If the token is a **number**: push it onto the stack.
- If the token is an **operator**: pop two operands, apply the operator, push the result.

At the end, the stack contains exactly one value — the answer.

### Dry Run: `3 4 + 5 * 2 -`

Initial state: stack = [ ]

**Token: 3** (number → push)
```
Stack: [ 3 ]
```

**Token: 4** (number → push)
```
Stack: [ 3 | 4 ]
              ↑ top
```

**Token: +** (operator → pop 4 and 3, compute 3+4=7, push 7)
```
Pop b=4, pop a=3
Compute a + b = 3 + 4 = 7
Stack: [ 7 ]
```

**Token: 5** (number → push)
```
Stack: [ 7 | 5 ]
              ↑ top
```

**Token: \*** (operator → pop 5 and 7, compute 7*5=35, push 35)
```
Pop b=5, pop a=7
Compute a * b = 7 * 5 = 35
Stack: [ 35 ]
```

**Token: 2** (number → push)
```
Stack: [ 35 | 2 ]
               ↑ top
```

**Token: -** (operator → pop 2 and 35, compute 35-2=33, push 33)
```
Pop b=2, pop a=35
Compute a - b = 35 - 2 = 33
Stack: [ 33 ]
```

**End**: stack has one element → **result = 33**

Verify with infix: `(3 + 4) * 5 - 2 = 7 * 5 - 2 = 35 - 2 = 33`. Correct.

### Pseudocode

```
eval_postfix(tokens):
    stack = []
    for token in tokens:
        if token is a number:
            stack.push(token)
        else:  // token is an operator
            b = stack.pop()
            a = stack.pop()
            stack.push(apply(a, token, b))
    return stack.pop()
```

**Complexity**: O(n) time, O(n) space in the worst case (all numbers before operators).

**Why this matters**: Every compiler, calculator, and expression parser uses this or the equivalent two-stack algorithm. Postfix is what your CPU actually executes — infix is only for human readability (source: `1 - Stack-and-Queue-Using-Arrays-and-Linked-Lists.pdf`).

---

## Balanced Parentheses

A classic stack application: verify that brackets in a string are correctly nested and matched.

**Valid**: `{[()]}` — every opener has a matching closer in the right order.
**Invalid**: `{[(])}` — the `]` closes `[` but there is still an unclosed `(` inside.

### Algorithm

Scan left to right:
- If token is an **opener** (`(`, `[`, `{`): push it.
- If token is a **closer** (`)`, `]`, `}`): check if stack top is the matching opener. If yes: pop. If no or stack is empty: **invalid**.
- At the end: if stack is empty → **valid**. If stack is non-empty → **invalid** (unclosed openers remain).

### Dry Run: `{[()]}` (valid)

```
char    action                          stack
{       opener → push {                 [ { ]
[       opener → push [                 [ {, [ ]
(       opener → push (                 [ {, [, ( ]
)       closer → top is ( → pop (       [ {, [ ]
]       closer → top is [ → pop [       [ { ]
}       closer → top is { → pop {       [ ]
END     stack is empty → VALID
```

Result: **VALID**

### Dry Run: `{[(])}` (invalid)

```
char    action                              stack
{       opener → push {                     [ { ]
[       opener → push [                     [ {, [ ]
(       opener → push (                     [ {, [, ( ]
]       closer → top is ( — MISMATCH!
        Expected ] to match [,
        but top of stack is (.
        → INVALID immediately
```

Result: **INVALID** — the `]` doesn't match the most recently opened `(`.

### Pseudocode

```
is_balanced(s):
    stack = []
    pairs = {')':'(', ']':'[', '}':'{'}
    for c in s:
        if c in '([{':
            stack.push(c)
        elif c in ')]}':
            if stack.isEmpty() or stack.pop() != pairs[c]:
                return False
    return stack.isEmpty()
```

**Why LIFO is exactly right here**: each closing bracket must match the *most recently* opened bracket. LIFO naturally models nesting — the innermost opener is always at the top of the stack (source: `2 - Stack-LIFO-Function-Calls-and-Backtracking.pdf`).

**Complexity**: O(n) time, O(n) space in the worst case (all openers, no closers).

---

## Infix to Postfix: The Shunting-Yard Algorithm

Dijkstra's **Shunting-Yard algorithm** (1960) converts a human-readable infix expression to postfix using one stack (for operators) and one output list. It handles operator precedence and parentheses correctly.

### Operator Precedence Table

| Operator | Precedence | Associativity |
|---|---|---|
| `*`, `/` | 2 (high) | Left |
| `+`, `-` | 1 (low) | Left |
| `(` | 0 (sentinel) | N/A |

### Algorithm

For each token left to right:
- **Number**: append to output directly.
- **Operator op**: while stack top is an operator with precedence >= op, pop it to output. Then push op.
- **`(`**: push onto stack.
- **`)`**: pop operators to output until you hit `(`. Pop and discard the `(`.
- **End**: pop all remaining operators to output.

### Dry Run: `3 + 4 * 5 - 2`

Initial: output = [ ], stack = [ ]

**Token: 3** (number → output)
```
Output: [3]         Stack: []
```

**Token: +** (operator, stack empty → push)
```
Output: [3]         Stack: [+]
```

**Token: 4** (number → output)
```
Output: [3, 4]      Stack: [+]
```

**Token: \*** (operator, prec(*)=2 > prec(+)=1 → do NOT pop +, just push *)
```
Output: [3, 4]      Stack: [+, *]
                                ↑ top
```

**Token: 5** (number → output)
```
Output: [3, 4, 5]   Stack: [+, *]
```

**Token: -** (operator, prec(-)=1)
- Stack top is `*`, prec(*)=2 >= prec(-)=1 → pop `*` to output
- Stack top is now `+`, prec(+)=1 >= prec(-)=1 → pop `+` to output
- Stack is empty → push `-`
```
Output: [3, 4, 5, *, +]    Stack: [-]
```

**Token: 2** (number → output)
```
Output: [3, 4, 5, *, +, 2]    Stack: [-]
```

**End**: pop remaining operators to output
```
Output: [3, 4, 5, *, +, 2, -]    Stack: []
```

**Postfix result**: `3 4 5 * + 2 -`

**Verify**: evaluate the postfix:
- `3 4 5 *` → push 3, push 4, push 5, `*` → 4*5=20, push 20 → stack: [3, 20]
- `+` → 3+20=23 → stack: [23]
- `2 -` → push 2, `-` → 23-2=21 → **result = 21**

Check infix: `3 + 4 * 5 - 2 = 3 + 20 - 2 = 21`. Correct — multiplication binds before addition.

### Why Shunting-Yard Works

The operator stack acts as a "holding area" for operators waiting to be applied. When a lower-precedence operator arrives, it forces all higher-precedence operators out first — which is exactly the order in which they should execute in the expression (source: `1 - Stack-and-Queue-Using-Arrays-and-Linked-Lists.pdf`).

**Complexity**: O(n) time, O(n) space.

---

## Related Pages

- [[adsa-queues]] — FIFO counterpart; BFS uses a queue where DFS uses a stack
- [[adsa-circular-queue]] — queue with wrap-around, no dead slots
- [[adsa-arrays]] — the underlying dynamic array used by array-based stacks

#adsa
