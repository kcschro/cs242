# Lab 2: Observing Sorting Algorithms

**CPTR 242 — Sequential and Parallel Data Structures and Algorithms**
Walla Walla University

---

## Model 1: Counting Comparisons

Operation counts are the purest measure of algorithmic complexity — they are exact and machine-independent. This program runs all four sorting algorithms on arrays of increasing size and counts every comparison made.

### Observation Table 1 — Comparisons on Random Input

Record the program output:

| n | Selection | Insertion | Merge | Quick | n(n-1)/2 | n·log₂(n) |
|---|---|---|---|---|---|---|
| 100 | 4950 | 2976 | 539 | 652 | 4950 | 664 |
| 500 | 124750 | 65301 | 3858 | 4464 | 124750 | 4482 |
| 1,000 | 499500 | 253791 | 8701 | 11150 | 499500 | 9965 |
| 2,000 | 1999000 | 1006215 | 19427 | 24252 | 1999000 | 21931 |
| 5,000 | 12497500 | 6341489 | 55218 | 74673 | 12497500 | 61438 |

---

### Critical Thinking Questions — Model 1

**Q1.** Compare **Merge** and **Quick** to the **n·log₂(n)** column. Are they above or below it? By roughly what factor? What does this tell you about the constant hidden inside O(n log n)?

> Merge sort runs slightly below nlog(n), by a factor of about 0.9. Quicksort runs slightly above, by a factor of around 1.1 or 1.2 times the nlog(n). This would seem to show that the constant inside of O(n log n) does have an impact, but may not be as significant on a large scale.

**Q2.** When n doubles from 1,000 to 2,000, what happens to the Selection sort comparison count? What happens to the Merge sort count? Do these ratios match what you would predict from O(n²) and O(n log n)?

> As n doubles, selection sort increases by almost exactly 4x. This perfectly matches O(n²). For merge sort, it slightly more than doubles, about 2.2x. This also matches expected behavior for O(n log(n)).

**Q3.** The Insertion sort comparison count on random data should be roughly between n and n(n-1)/2. Where does it actually land? What does this suggest about its average-case behavior?

> Between n and n(n-1)/2, insertion sort appears to land at about roughly half the value of n(n-1)/2. So, as n grows, insertion grows much faster, in accordance with something that falls under the category of O(n²).

**Q4.** Quick sort's comparison count on random data is likely close to — or sometimes less than — merge sort's. Both are O(n log n). Given that quicksort has an O(n²) worst case, why might it make *fewer* comparisons than merge sort on average?

> I am observing the opposite behavior in my testing, with properly running the compile commands and compiler optimization disable flag. In my specific data, it would make sense that there could possibly be some higher concentration of poor pivot choices or similar factors, resulting in a higher quicksort upon comparison. My data aside, in the context of the question, it could make fewer comparisons with **good** pivot choices, beating merge sort on average but still having a higher worst case.

---

## Model 2: Input Shape Changes Everything

Theoretical complexity describes average behavior. But real inputs are rarely random. This program runs all four algorithms on three different input arrangements: random, already sorted, and reverse sorted.

### Observation Table 2 — Comparisons by Input Shape (n = 1,000)

| Input type | Selection | Insertion | Merge | Quick |
|---|---|---|---|---|
| Random | 499500 | 246352| 8707 | 11952 |
| Already sorted | 499500 | 999 | 5044 | 499500 |
| Reverse sorted | 499500 | 499500 | 4932 | 499500 |

---

### Critical Thinking Questions — Model 2

**Q5.** Look at the **Selection** column across all three input types. Does the comparison count change? Why or why not? What does this tell you about selection sort's relationship to input shape?

> No, the comparison count does not change. The algorithm clearly does not change based on input. It always has to check the entire "unsorted" section for each comparison.

**Q6.** Look at the **Insertion** column. Already-sorted input should give a dramatically lower count than random or reverse-sorted. By approximately what factor does it drop? What is happening inside the algorithm that explains this?

> Random is half that of reverse sorted, and already sorted goes down from random that by a factor of 250. This algorithm inserts into the already sorted section, so for already sorted, it just has to check N-1 values.

**Q7.** Look at the **Merge** column. Does it change meaningfully across input types? What property of merge sort explains this consistency?

> For merge sort, comparisons are slightly higher on random, but not by any particularly large amount. Sorted and reverse sorted have negligible differences. Merge sorts strategy of dividing and sorting individual halves always results in O(n log(n)).

**Q8.** Look at the **Quick** column on already-sorted input. It is likely much higher than on random input. Explain why — trace what happens when the last element is chosen as pivot and the array is already sorted.

> If an array is already sorted, choosing a pivot at the end creates the worst case runtime for quicksort. If the input is already sorted, then this particular pivot is higher than everything else, and the algorithm will compare every data point with the pivot. It incorporates recursion, making comparisons rise even higher.

**Q9.** Based on this table, which algorithm would you choose for a dataset you *know* is almost always already sorted (e.g., a log file that is mostly in time order with a few late arrivals)? Which would you avoid? Justify your choices.

> The best choice would be insertion sort, yielding the lowest comparisons for already sorted input. I would use mergesort due to its very high efficiency. The only limitation is of course, large data sets on space limited machines. I would avoid quick and selection with their extremely high worst cases.

---

## Model 3: Time, Space, and Stability

Comparison counts tell us about the algorithm's logic. But programmers also care about wall-clock time and how much memory an algorithm uses — and whether it preserves the original order of equal elements (stability). This program measures all three.

### Observation Table 3 — Runtime and Memory

| n | Sel (ms) | Sel mem | Ins (ms) | Ins mem | Merge (ms) | Merge mem | Quick (ms) | Quick mem |
|---|---|---|---|---|---|---|---|---|
| 1,000 | 7.12 | 4,000 | 3.80 | 4,000 | 1.15 | 81,540 | 0.22 | 4,000 |
| 5,000 | 120.27 | 20,000 | 68.23 | 20,000 | 3.98 | 778,756 | 0.96 | 20,000 |
| 10,000 | 382.79 | 40,000 | 212.63 | 40,000 | 5.96 | 1,688,580 | 1.82 | 40,000 |

### Observation Table 3b — Stability

| Algorithm | Stable? |
|---|---|
| Selection sort | YES |
| Insertion sort | YES |
| Merge sort | YES |
| Quicksort | NO | 


---

### Critical Thinking Questions — Model 3

**Q10.** Look at the **Sel mem** and **Ins mem** and **Quick mem** columns. They should be near zero (just the overhead of passing the vector by value). The **Merge mem** column should show values proportional to n. Approximately how many bytes does merge sort allocate per element? Does this match the O(n) space complexity?

> Bytes per element are ranging from about 80 to 170. The memory allocation is roughly linear, since for higher n, as input doubles, memoryy also roughly doubles.

**Q11.** When n grows from 1,000 to 10,000 (10×), how does selection sort's runtime change? How does merge sort's runtime change? Do these match the predicted scaling from O(n²) and O(n log n)?

> Selection runtime increases by a factor of aobut 55x. Merge sort runtime increases by 6x. Though not exact at a first glance, selection has a much steeper rate, matching O(n²), and merge is slightly faster than O(n log n).

**Q12.** At n = 10,000, which algorithm is fastest on random data? Which is slowest? Is the fastest algorithm the one with the best Big-O, or are there other factors at play?

> Quicksort runs fastest, and selection is slowest. There are certainly various factors at play, creating variations in runtime, but selectoin and quick are O(n²) and O(n log n), respectively. This seems to match experimental runtime.

**Q13.** You are building a leaderboard that sorts players by score. When two players have the same score, they should remain in the order they originally achieved it (i.e., first-to-reach-score is ranked higher). Based on the stability results, which sorting algorithms are safe to use? Which are not?

> Any of the four except for quicksort are safe to use. Quicksort is unstable, making swaps that can disrupt original order.

**Q14.** Merge sort uses O(n) extra memory. On a machine with 8 GB of RAM, sorting a dataset of 1 billion integers (each 4 bytes = 4 GB of data), merge sort would need approximately how much *additional* memory? Is this a concern?

> According to the data, merge sort used much more than 4 bytes for each, upwards of 100 bytes per data. Such consumption would definitely be a concern, greatly exceeding the memory of a machine with 8 GB.

---

## Model 4: Recursion Unrolled

Merge sort and quicksort are recursive. This program makes the recursion visible by printing each call as it happens, so you can see the divide-and-conquer structure directly.

### Observation: Merge Sort Trace

Run the program and study the merge sort trace output. Answer the questions below based on what you see printed.

**Q15.** How many levels of indentation does the merge sort trace show for n = 7? What does each level of indentation correspond to in terms of the recursion? How does the number of levels relate to log₂(7)?

> Merge sort for n = 7 shows 3 levels of indentation. Each indentation corresopnds to another level of recursion. The value of log₂(7) ~ 2.8, which matches the expected value of 3.

**Q16.** At the deepest level of indentation, each `base` line shows a single element. After the two `base` lines for the same parent, a `merge` line appears. What is always true about the output of a `merge` step — regardless of what the input subarrays looked like?

> Merge is always combining two sections, regardless of what particular base case or partially sorted array was completed. It takes two, and outputs one, merging every time.

**Q17.** Count the total number of `merge` operations printed. For n = 7, there should be 6. For n elements in general, how many merge operations does merge sort perform? Express this as a function of n.

> There are in fact 6 merge operations. For n elements, merge sort performs f(n-1) merges.

### Observation: Quicksort Trace — Random vs. Sorted Input

**Q18.** In the random-input quicksort trace, look at the partition steps. Does the pivot tend to split the array into roughly equal halves, or very unequal ones? Now look at the sorted-input trace. What do you observe about the split sizes? How many total partition steps appear in each trace?

> For random-input, the chosen pivots are rather sporadic. About half of the traces create a roughly evenly partitioned array, and the other half is very unbalanced. For sorted-input, the algorithm places a pivot recursively at n-1. (It starts at 6, then counts down to the end of the array.) Random creates 4 partitions, sorted creates 6.

**Q19.** In the sorted-input quicksort trace, each partition step's "left" side is empty `[]` and the "right" side has all remaining elements. If this happens at every level on an array of n = 1,000 elements, how many total partition steps would there be? What complexity class is this?

> This creates 999 partitions steps, since it follows (n-1). Partition steps is O(n), but creates a time complexity of O(n²).

---

## Extra Credit Questions

**A1.** Radix sort was not in today's programs — it sorts without making element-to-element comparisons at all. Given that all four algorithms we studied today are comparison-based, what fundamental limit do they all share? (Hint: think about what O(n log n) represents for comparison-based sorting.)

> O(n log(n)) represents the lower bound, which radix sort can achieve better performance than, not being limited by comparisons.

---

## Summary

| Algorithm | Comparisons (random) | Best case | Worst case | Space | Stable | Adaptive |
|---|---|---|---|---|---|---|
| Selection sort | n(n-1)/2 exactly | O(n²) | O(n²) | O(1) | No | No |
| Insertion sort | ~n²/4 | O(n) | O(n²) | O(1) | Yes | Yes |
| Merge sort | ~n·log₂n | O(n log n) | O(n log n) | O(n) | Yes | No |
| Quicksort | ~1.39·n·log₂n | O(n log n) | O(n²) | O(log n) | No | No |

**Key takeaways:**

- Selection sort makes the same number of comparisons regardless of input — completely non-adaptive.
- Insertion sort is uniquely fast on nearly-sorted data — O(n) best case. This is a real, exploitable property.
- Merge sort is the reliability choice: O(n log n) always, stable, but costs O(n) extra memory.
- Quicksort is fast in practice but fragile on sorted input with a naive pivot strategy.
- Input shape matters as much as algorithm choice. Knowing your data is as important as knowing your algorithm.
- Stability matters when equal elements carry secondary information that must be preserved.

---

*Next lab: implementing linked lists from scratch — a data structure that changes what algorithms are even possible.*