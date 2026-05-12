# Lab 6: Observing Heaps and Heapsort

**CPTR 242 — Sequential and Parallel Data Structures and Algorithms**
Walla Walla University

## Model 1: The Array Heap — Sift-Up on Insertion

Before measuring anything, you need to see exactly what happens inside the array on every insertion. This program builds a max-heap by inserting keys one at a time and prints the full array and a swap trace after each insertion. It then performs several extractions with sift-down traces.

### Observation Table 1a — Array State After Each Insertion

Run the program and fill in the array contents (h[1] through h[8]) after each insertion.

| After inserting | h[1] | h[2] | h[3] | h[4] | h[5] | h[6] | h[7] | h[8] |
|---|---|---|---|---|---|---|---|---|
| 10 | 10 | | | | | | | |
| 4  | 10 | 4 | | | | | | |
| 15 | 15 | 4 | 10 | | | | | |
| 7  | 15 | 7 | 10 | 4 | | | | |
| 20 | 20 | 15 | 10 | 4 | 7 | | | |
| 3  | 20 | 15 | 10 | 4 | 7 | 3 | | |
| 18 | 20 | 15 | 18 | 4 | 7 | 3 | 10 | |
| 9  | 20 | 15 | 18 | 9 | 7 | 3 | 10 | 4 |

### Observation Table 1b — Extraction Sequence

| Extraction # | Value returned | Number of sift-down swaps |
|---|---|---|
| 1st | 20 | 3 |
| 2nd | 18 | 3 |
| 3rd | 15 | 2 |

---

### Critical Thinking Questions — Model 1

**Q1.** What value always sits at `h[1]` after every insertion, regardless of the order keys arrive? Why does the heap property guarantee this?

> The max value is always at h[1], regardless of insertion order. In max-heaps, every parent must be greater than its children, so the first node must always be the max.

**Q2.** When 20 is inserted, it sifts all the way to the root. Trace the swaps by hand using the formula `parent = i / 2` starting from the index where 20 lands. Do the printed swaps match your calculation?

> Following i//2, it would move from the insertion point of h[5] to h[2], and then h[1]. This matches the printed swaps.

**Q3.** `extractMax` moves the *last* element in the array to the root before sifting down. Why the last element specifically? What structural property of a complete binary tree makes this the correct choice?

> A complete binary tree must have all levels full, left to right. If you were to remove the top value and then just shift everything up, then this could break the completeness of the tree. If the last is moved and sifted down, then the tree will return to its correct complete form.

**Q4.** The heap has 8 nodes, so its height is ⌊log₂ 8⌋ = 3. From Table 1b, did any extraction require more than 3 sift-down swaps? Explain why this bound holds.

> None of the extractions executed more than 3 sift-down swaps, as expected. If the height of the tree is three, any item can only be moved up or down the height of the tree. Swaps can only happen going up or down a level.

---

## Model 2: Array Heap vs. Linked-Node Heap — Memory and Speed

An array heap requires no pointers. A linked-node heap stores explicit `left`, `right`, and `parent` pointers in every node. This program builds both heaps from identical insertions and reports the memory consumed by each representation at several values of n, then times n insertions for both.


### Observation Table 2a — Memory Usage

| n | Array heap (bytes) | Linked heap (bytes) | Ratio (linked / array) |
|---|---|---|---|
| 1,000 | 4,096 | 32,000 | 7.8x |
| 10,000 | 65,536 | 320,000 | 4.9x |
| 100,000 | 524,288 | 3,200,000 | 6.1x |
| 500,000 | 2,097,152 | 16,000,000 | 7.6x |

### Observation Table 2b — Insertion Time

| n | Array heap (ms) | Linked heap (ms) |
|---|---|---|
| 1,000 | 0.193 | 0.671 |
| 10,000 | 0.902 | 7.757 |
| 100,000 | 8.647 | 95.720 |
| 500,000 | 28.705 | 454.898 |

---

### Critical Thinking Questions — Model 2

**Q5.** The program prints `sizeof(int)` and `sizeof(Node)` at startup. Using those two numbers, compute the *theoretical* ratio of linked-to-array memory and compare it to the ratios you measured in Table 2a. Does the ratio stay constant as n grows? What explains any discrepancy at small n?

> With 4 bytes for an int and 32 bytes for a node, one would expect a 8:1 ratio of linked to array. Comparing to the observed ratios, this does mostly match. For the second two n-values it dips lower to 5 and 6, but for the first and last it is just shy of 8. The discrepancies for small n appear to come from the array heap, which doesn't grow consistently in memory usage like the linked heap. The array isn't always resizing or at capacity.

**Q6.** The linked heap's `nodeAt(idx)` navigates from the root to the target node by following pointers level by level. The array heap computes the parent in one step with `i / 2`. Both are O(log n) in theory, but with very different constants. Explain in terms of CPU cache behavior why pointer chasing is slower than index arithmetic, referencing what you observed about linked lists vs. arrays in Lab 4.

> In the context of CPU cache behavior, arrays vs pointers once again come down to blocks of continuous memory. Index arithmetic is simple, and much less likely to result inmisses since the given indices are vectorized. Pointer chasing is called that because pointers are not clumped together, and have to be repeatedly searched out to find the next value. This results in more frequent cache misses.

**Q7.** Given what Tables 2a and 2b show, propose at least one realistic scenario where you would choose the linked-node heap over the array heap despite its overhead.

> A heap could hypothetically be advantageous if you didn't want resizing. Though insertion time may be slower and memory overhead greater, if you have a very large array, then you wouldn't deal with the time and space complexity of resizing and allocating more memory.

---

## Model 3: Heapsort — Heapify and Sort-Down

Heapsort sorts an array in two phases. **Phase 1 (heapify):** rearrange the whole array into a max-heap in O(n) time by applying sift-down from the bottom of the tree upward. **Phase 2 (sort-down):** repeatedly swap the root (the current maximum) to the end of the unsorted region and sift the new root down. This program prints the full array after every step of both phases so you can trace the sort exactly.

### Observation Table 3a — Array After Each Heapify Step

| siftDown from index | Array contents after step |
|---|---|
| (initial) | 5, 3, 8, 1, 9, 2, 7, 4, 6 |
| 3 | 5,  3,  8,  6,  9,  2,  7,  4,  1 |
| 2 | 5,  3,  8,  6,  9,  2,  7,  4,  1 |
| 1 | 5,  9,  8,  6,  3,  2,  7,  4,  1 |
| 0 | 9,  6,  8,  5,  3,  2,  7,  4,  1 |

### Observation Table 3b — Sort-Down Phase

Record the value moved into the sorted region and the remaining unsorted size after each step.

| Step | Value placed in sorted region | Unsorted size remaining |
|---|---|---|
| 1 | 9 | 8 |
| 2 | 8 | 7 |
| 3 | 7 | 6 |
| 4 | 6 | 5 |
| 5 | 5 | 4 |
| 6 | 4 | 3 |
| 7 | 3 | 2 |
| 8 | 2 | 0 |

---

### Critical Thinking Questions — Model 3

**Q8.** The heapify phase starts at index `n/2 - 1` and works *downward* to 0. Why not start at `n - 1`? What kind of nodes occupy indices `n/2` through `n - 1`, and why do they need no sifting?

> The nodes between n/2 and n-1 are all already leavbes. Due to the organization of a heap, they cannot have child nodes. Everything in the first half does, so you have to work from there.

**Q9.** After Phase 1, the maximum is at index 0. Phase 2 swaps it to the last position, temporarily placing a small value at the root and breaking the heap property. Why is it still correct to call `siftDown(a, 0, end)` with the sorted tail excluded? What guarantee does sift-down restore?

> If the tail is already sorted, then sift-down will fix the heap property, moving everything down through the list until it is fully sorted. Sift-down guarantees the correct logic of a larger parent vs child.

---

## Model 4: Heapsort vs. Mergesort vs. std::sort — Time and Space

Heapsort's O(n log n) guarantee comes with a practical cost: non-sequential memory access causes frequent cache misses. Mergesort accesses memory sequentially but requires O(n) auxiliary space. `std::sort` uses a hybrid strategy that attempts to get the best of both worlds. This program benchmarks all three on random, already-sorted, and reverse-sorted inputs, and separately reports the auxiliary memory each algorithm requires.

### Observation Table 4a — Sorting Time (ms)

**n = 10,000**

| Input order | heapsort (ms) | mergesort (ms) | std::sort (ms) |
|---|---|---|---|
| random  | 5.59 | 6.14 | 1.58 |
| sorted  | 6.28 | 2.03 | 0.55 |
| reverse | 4.39 | 5.33 | 1.18 |

**n = 100,000**

| Input order | heapsort (ms) | mergesort (ms) | std::sort (ms) |
|---|---|---|---|
| random  | 83.12 | 28.61 | 23.17 |
| sorted  | 66.19 | 12.22 | 5.87 |
| reverse | 68.28 | 12.67 | 12.55 |

**n = 500,000**

| Input order | heapsort (ms) | mergesort (ms) | std::sort (ms) |
|---|---|---|---|
| random  | 376.55 | 143.38 | 98.28 |
| sorted  | 272.11 | 54.89 | 31.42 |
| reverse | 290.63 | 56.34 | 17.95 |

### Observation Table 4b — Auxiliary Memory at n = 500,000

| Algorithm | Space complexity | Auxiliary bytes |
|---|---|---|
| heapsort  | O(1)     | 4 |
| mergesort | O(n)     | 2,000,000 |
| std::sort | O(log n) | *(stack, not measured)* |

---

### Critical Thinking Questions — Model 4

**Q10.** Across all three input orders and all three values of n, which algorithm is consistently fastest? Which is consistently slowest? Does input order affect runtime meaningfully? Explain why or why not this would be the case, referencing heapsort's algorithm nature.

> std::sort is the fastest across all input orders and values of n. Heapsort is the slowest, although less clear for small values of n. Random consistently has the largest impact out of any input orders, but it is never too far off from the others. It makes sense that input order would not majorly affect heapsort, since it always leads by creating the max heap first, and then executing sift-down to sort. 

**Q11.** `std::sort` uses **introsort**: it starts with quicksort but falls back to heapsort when recursion depth exceeds 2·⌊log₂ n⌋, preventing O(n²) worst-case behavior. Based on Table 4a, does `std::sort` appear to behave more like heapsort or something significantly faster? What does that tell you about how often the heapsort fallback is actually triggered in practice?

> As can be seen in n = 10,000 std::sort behaves significantly faster than heapsort. Looking at higher values of n, the heapsort fallback is probably triggered later, as std::sort continues to remain consistently faster than heapsort. 

---
