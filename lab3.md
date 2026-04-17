# Lab 3: Observing Linked Lists

**CPTR 242 — Sequential and Parallel Data Structures and Algorithms**
Walla Walla University

---

## Overview

In this lab you will **run complete programs and observe their behavior**. There is no coding required. Your job is to read the output carefully, record what you see, and reason about what it means.

By the end you should be able to:

- Explain how pointer manipulation produces correct insert and remove operations on linked lists
- Describe the performance difference between linked lists and array-based lists for different operations
- Recognize O(1) and O(n) behavior in list operations from empirical data
- Explain what memory layout looks like for linked nodes versus contiguous arrays

---

## Model 1: Pointers in Motion

The most important thing to understand about a linked list is that every insert and remove is really just a careful reassignment of pointers. This program traces every pointer change so you can see exactly what happens at each step.


### Observation Table 1 — List State After Each Operation

Run the program and record the list contents printed after each labeled step:

| Step | List contents (data values in order) |
|---|---|
| Initial |  |
| After insert 30 at front | |
| After insert 10 at front | |
| After insert 20 after 10 | |
| After insert 40 after 30 | |
| After removing 20 (after head) | |
| After removing head (10) | |

---

### Critical Thinking Questions — Model 1

**Q1.** Every `new Node(val)` call allocates memory on the heap. Every `delete` call frees it. What would happen to a program that builds a large linked list, removes all nodes using `removeAfter`, but forgets to call `delete` on the removed nodes?

> A pointer that is not deleted is a memory leak. This could result in stack overflow or a program crashing, since there are dangling pointers.

---

## Model 2: Operation Costs — Linked List vs. Array

Theory says linked lists give O(1) insert at front but O(n) access by index, while arrays give O(1) access but O(n) insert at front. This program measures those claims empirically across a range of sizes.


### Observation Table 2a — Insert at Front (μs)

| n | Linked `pushFront` | Vector `insert(begin)` |
|---|---|---|
| 1,000 | 42.1 | 61.8 |
| 5,000 | 298.9 | 1161.6 |
| 10,000 | 366.9 | 1963.8 |
| 50,000 | 2476.7 | 68934.3 |
| 100,000 | 4424.5 | 308058.0 |

### Observation Table 2b — Insert at Back (μs)

| n | Linked (walk to end) | Vector `push_back` |
|---|---|---|
| 1,000 | 582.1 | 4.5 |
| 5,000 | 20,058.6 | 44.5 |
| 10,000 | 113,285.3 | 93.6 |
| 50,000 | 3,601,843.0 | 309.9 |
| 100,000 | 16,221,978.3 | 589.6 |


### Observation Table 2c — Access by Index (ns, single access at n/2)

| n | Linked `getAt(n/2)` | Vector `v[n/2]` |
|---|---|---|
| 1,000 | 816.7 | 3.163 |
| 5,000 | 4634.9 | 2.428 |
| 10,000 | 8571.4 | 2.413 |
| 50,000 | 66411.32 | 2.715 |
| 100,000 | 161765.8 | 5.343 |

---

### Critical Thinking Questions — Model 2

**Q2.** In Table 2a, how does linked list `pushFront` time change as n grows from 1,000 to 100,000 (100×)? How does vector `insert(begin)` time change? Which operation's growth rate matches O(1) and which matches O(n)?

> Linked list grows roughly linearly with some variation, while the vector time is growing expoenentially. These both match O(1) and O(n), respectively. The linked list does n number of O(1) operations resulting in O(n), while the vector does n number of up to O(n) operations, resulting in around O(n^2).

**Q3.** In Table 2b, the linked list version walks to the end of the list before each insertion, making it O(n²) total. When n grows from 1,000 to 10,000 (10×), by approximately what factor does the linked list time increase? Does this match O(n²)?

> For a O(n²), when n increases by 10x, this results in a factor 10*10=100. The linked list table grows by a factor of about 200, so this does match O(n²), just with an additional constant multiplier.

**Q4.** In Table 2c, the vector access time should be nearly the same for all values of n. The linked list access time grows with n. At n = 100,000 accessing the middle element, roughly how many times slower is the linked list than the vector? What does this tell you about why `std::vector` is preferred for random-access workloads?

> The linked list is over 30,000 times slower in thsi scenario. It is very clear that a std:vector is appeared for random access, since it keeps an index value, while a linked list has to travel over all of n/2 to get to the middle element.

**Q5.** The linked list in Table 2b is slow because it has no tail pointer — it walks the whole list on every back-insertion. If you added a `tail` pointer (as in `std::list`), back insertion would be O(1). Would that make the linked list competitive with `vector::push_back` for building a list from scratch? What other factor still disadvantages the linked list?

> Hypothetically, it would make it competitive, since both should be around the same O(1) operation time for this scenario. A linked list is disadvantaged since it still has to manage pointers, which are not all grouped in memory. A vector provides faster access with less memory overhead.

**Q6.** Based on Tables 2a, 2b, and 2c together, describe a real application or data structure where a linked list would be clearly the better choice over a vector, and justify your answer using the specific operation costs you observed.

> The most clearly advantageous situation for a linked list would be consistent and frequent front insertions. If the use case does not need random access and focuses on front insertion, the linked list is a better choice. A std::vector's greatest weakness in this area is the need to shift values for every front insertion.

---

## Model 3: Memory Layout and the Cost of Cache Misses

One reason vector outperforms linked lists so dramatically on access is **cache locality** — the CPU loads nearby memory in bulk. Linked list nodes can be scattered anywhere in the heap. This program makes that visible by printing node addresses and measuring how memory layout affects traversal speed.


### Observation Table 3b — Traversal Speed

| n | Vector sum (μs) | Linked list sum (μs) | Slowdown |
|---|---|---|---|
| 100,000 | 233.06 | 1206.08 | 5.2x |
| 500,000 | 889.20 | 35960.85 | 40.4x |
| 1,000,000 | 1605.14 | 114825.73 | 71.5x |

### Observation Table 3c — Node Sizes

| Type | `sizeof` (bytes) | Notes |
|---|---|---|
| `int` | | |
| Pointer (`Node*`) | | |
| `Node` (singly) | | |
| `DNode` (doubly) | | |
| 10,000 singly-linked nodes | | |
| 10,000 doubly-linked nodes | | |

---

### Critical Thinking Questions — Model 3

**Q7.** Look at the address gaps in Table 3a. Are the nodes contiguous (gap ≈ `sizeof(Node)`)? Or are they spread out? What does this tell you about how `new` allocates memory and why linked lists are "pointer-chasing" structures?

> N/A

**Q8.** In Table 3b, both the vector and the linked list perform the same total number of additions (n additions). Yet the linked list is significantly slower. The work is identical — why is the time different? What is the CPU doing between each addition in the linked list case that it doesn't have to do for the vector?

> The cpu is having to seek different memory constantly, which is not always a perfect operation. Even without the possibliity of misses, so many different cache lookups will always be dramatically slower than the contiguous memory blocks of a vector.

**Q9.** Look at Table 3c. A doubly-linked node stores one extra pointer compared to a singly-linked node. How many extra bytes does that add per node? For 10,000 nodes, how many extra bytes total does that cost? Is this a meaningful difference in practice?

> N/A

**Q10.** The traversal slowdown in Table 3b is consistently larger than 1× — linked list traversal is slower than vector traversal even though both do O(n) work with the same loop structure. As n grows from 1,000 to 10,000, does the slowdown factor stay roughly constant, grow, or shrink? What does the trend suggest about how cache effects scale with list size?

> (Note: my code output all of table 3b increased by a 100x, so this is observed from 100,000 to 1,000,000.) The slowdown increases by by an additional 30 each n increase. This suggests that as scale increases, cache access has the capacity to continually perform worse. This makes sense, because more frequent and larger memory traversal always increases the odds of mistakes, and can just be a costly operation in general, depending on layout of the relevant blocks.

----