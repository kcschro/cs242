# Lab 1: Observing Algorithm Efficiency

**CPTR 242 — Sequential and Parallel Data Structures and Algorithms**
Walla Walla University

---

### Observation Table 1 — Record Your Output

Run the program and fill in the table:

| n | Lin Best | Lin Worst | Lin Avg | Bin Best | Bin Worst | Bin Avg |
|---|---|---|---|---|---|---|
| 10 | 1 | 10 | 5 | 1 | 8 | 4 |
| 100 | 1 | 100 | 50 | 1 | 14 | 10 |
| 1,000 | 1 | 1,000 | 498 | 1 | 20 | 17 |
| 10,000 | 1 | 10,000 | 4976 | 1 | 28 | 23 |
| 100,000 | 1 | 100,000 | 49751 | 1 | 34 | 30 |

---

### Critical Thinking Questions — Model 1

**Q1.** Look at the **Lin Worst** column. Each time `n` grows by 10×, what happens to the number of comparisons? Write a formula relating Lin Worst to `n`.

> Lin worst increases directly in proportion with n. (Lin worst = n).

**Q2.** Look at the **Bin Worst** column. Each time `n` grows by 10×, what happens? Is the growth the same as linear search? Describe the pattern in words.

> Each time n grows by 10 times, bin worst increases by six. (Except for the one time that it doesn't, going from 20 to 28 on 1,000 to 10,000). This is not the same growth as the linear search, since Lin worst is exactly the same as n.

**Q3.** The **Bin Best** column should show `1` for all values of `n`. Why? What is the best-case scenario for binary search, and why does the array size not affect it?

> The best-case scenario for binary search is that the value is in the very middle of the array. An array could be any size, and the value could happen to be in the middle, the first one checked.

**Q4.** Compare **Lin Avg** to **Lin Worst**. What is the approximate ratio between them across all values of `n`? What does this tell you about where a typical search ends up?

> The ratio of Lin Avg to Lin Worst is approximately 1:2. This implies that a typical search ends up at the median of the dataset.

**Q5.** For `n = 100,000`, binary search makes far fewer comparisons than linear search in the worst case. But for `n = 10`, the difference is small. Does this mean binary search is *always* the right choice? What would make you prefer linear search?

> This can vary case by case, but binary search is not always the right choice. For example, binary search needs a sorted array, so if an array was expected to have multiple insertions (still with a small n value), it might not be worth the additional time needed to re-sort the array and then use binary search. A linear search would be sufficient.

---

## Model 2: Measuring Real Time

Operation counts are exact and machine-independent. But programmers also care about **wall-clock time** — how long the user actually waits.

---

### Observation Table 2a — Worst-Case Runtime

| n | Linear (μs) | Binary (μs) | Speedup |
|---|---|---|---|
| 1,000 | 5.33 | 0.09 | 62.3x |
| 10,000 | 65.08 | 0.13 | 503.0x |
| 100,000 | 574.22 | 0.14 | 4,043.8x |
| 1,000,000 | 5,201.66 | 0.18 | 28,362.4x |
| 10,000,000 | 52,506.90 | 0.21 | 249,794.9x |

### Observation Table 2b — Best-Case Runtime

| n | Linear (μs) | Binary (μs) |
|---|---|---|
| 1,000 | 0.01 | 0.15 | 
| 10,000 | 0.01 | 0.16 |
| 100,000 | 0.01 | 0.20 |
| 1,000,000 | 0.01 | 0.24 |
| 10,000,000 | 0.01 | 0.32 |

---

### Critical Thinking Questions — Model 2

**Q6.** In Table 2a, when `n` grows from 1,000 to 10,000 (10×), approximately how much does linear search time increase? Does this match what you predicted from your answer to Q1?

> It increased by a factor of approximately 13, which does not match my prediction of being directly proportional (equal to n).

**Q7.** Look at the **Speedup** column in Table 2a. Does the speedup stay constant, grow, or shrink as `n` increases? Explain why, using the operation counts from Model 1 as evidence.

> Speedup does not stay constant, increasing at a relatively similar rate as the input, in accordance with 10x n each time. I believe speedup is actually a ratio of our two algorithms, which grow at different rates. So we should expect increase at a rate that combines those two functions.

**Q8.** In Table 2b (best case), what happens to linear search time as `n` grows? What about binary search? Why are these results so different from the worst case?

> In linear search, best case will always be first, so the time will not change. However, worst case is equal to the size of the data set. Binary best case takes a bit longer, since it still needs to parse that many numbers in the program, taking the time to find the middle value. It requires more memory manipulation, therefore more chances to diverge from the consistent runtime of linear best case. Overall, binary best and worst case seem to vary randomly, and I can't predict that much with my current knowledge of memory and how it affects this behavior.

---

## Model 3: Memory Usage

Runtime is only half the picture. Algorithms also consume **memory**. This program measures how much heap memory each approach allocates as `n` grows.


### Observation Table 3a — Array Memory

| n | Heap bytes | Bytes per element |
|---|---|---|
| 1,000 | 4,000 | 4.0 |
| 10,000 | 40,000 | 4.0 |
| 100,000 | 400,000 | 4.0 |
| 1,000,000 | 4,000,000 | 4.0 |

### Observation Table 3b — Call Stack Depth

| n | Iterative frames | Recursive frames (worst) |
|---|---|---|
| 1,000 | 1 | 11 |
| 10,000 | 1 | 15 |
| 100,000 | 1 | 18 |
| 1,000,000 | 1 | 21 |

---

### Critical Thinking Questions — Model 3


**Q9.** When `n` increases by 10x, what happens to total heap memory? What complexity class describes the memory usage of storing an array of `n` integers?

> When n increases by 10x, total heap memory also increases by 10x. This shows O(n) behavior, a linear relationship.

**Q10.** Look at Table 3b. The iterative version always uses 1 stack frame. The recursive version uses more. How does the number of recursive frames grow as `n` grows? Does this match O(log n)?

> The number of of recursive frames grows logarithmically as n increases 10x. It is almost perfectly log2(n), just off by a small constant.


**Q11.** Both the iterative and recursive binary search are O(log n) in *time* complexity. But their *space* complexity is different. State the space complexity of each and explain why they differ.

> Iterative uses a single frame and simply replaces it each time. It does not need to keep creating new variables. There for it is O(1). However, recursive is base 2 O(log(n)), since it keeps creating new frames, allocating more and more memory on the stack.

---

## Model 4: Putting It Together — The Full Picture

This final program runs an O(n²) algorithm alongside the O(n) and O(log n) algorithms, so you can see all three growth rates in the same table.


### Observation Table 4a — Three Growth Rates Side by Side

| n | O(n²) sort (ms) | O(n) scan (ms) | O(log n) search (ms) |
|---|---|---|---|
| 500 | 0.982 | 0.001 | 0.000 |
| 1,000 | 4.103 | 0.001 | 0.000 |
| 2,000 | 16.456 | 0.003 | 0.000 |
| 4,000 | 59.964 | 0.005 | 0.000 |
| 8,000 | 219.700 | 0.019 | 0.000 |

### Observation Table 4b — O(n²) Scaling

| n | Time (ms) | Ratio vs. prev |
|---|---|---|
| 500 | 0.831 | — |
| 1,000 | 3.669 | 4.41x |
| 2,000 | 28.946 | 7.88x |
| 4,000 | 46.525 | 1.60x |
| 8,000 | 224.546 | 4.82x |

---

### Critical Thinking Questions — Model 4

**Q12.** In Table 4a, the O(log n) search times are probably so small they appear as `0.000` ms. This doesn't mean the algorithm takes zero time — it means our millisecond timer isn't precise enough. What does this tell you about using wall-clock time for benchmarking very fast algorithms?

> This shows that wall-clock time is always not accurate enough, or sometimes even irrelevant to try and benchmark such algorithms. Other methods would be necessary to accurately gauge the speed of this algorithm.

**Q13.** At `n = 8,000`, how much longer does the O(n²) sort take compared to the O(n) scan? Now imagine `n = 1,000,000`. Using the complexity ratios, estimate how much longer the O(n²) algorithm would take relative to O(n) at that scale. Show your reasoning.

> O(n²) takes over 219 ms longer, over 11,000 times how long O(n) takes. 

**Q14.** The O(n) scan uses `volatile` on the accumulator variable. Remove the `volatile` keyword, recompile with `-O2` instead of `-O0`, and run again. What happens to the O(n) scan time? Why? What does this reveal about the relationship between benchmarking and compiler optimizations?

> O(n) scan time drops off significantly. At 500 it behaves the same as the binary search, with measured times too small for the clock to count. It remains very close to 0.001 up through 8,000. This behavior shows that compiler optimizations can vary significantly, and drastically interfere with the benchmarking of clock times.

**Q15.** Suppose you are building an app that needs to sort a list of items every time a user opens it. The list is currently 1,000 items and your O(n²) sort takes 1 ms — fast enough. Your product grows and the list becomes 100,000 items. Using your observed scaling ratio, estimate the new sort time. Is it still acceptable? What would you do?

> If every doubling causes 4x overall (2 squared), and 8,000 to 100,000 is roughly 3.5 doublings, then this would result in a time of about 3,000 ms, which is a rather high launch time for an app on every single open. I would try for some sort of mergesort or a similar algorithm, which produces an O(n log(n)) time complexity.

---