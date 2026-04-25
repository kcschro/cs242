# Lab 4: Observing Stacks and Queues

**CPTR 242 — Sequential and Parallel Data Structures and Algorithms**
Walla Walla University

---

## Model 1: Stack State Tracing

Before measuring anything, you need to be able to predict and verify what a stack contains after a sequence of operations. This program executes a fixed sequence of push and pop operations on both a linked-list stack and an array-based stack, printing the full state after every step.

### Observation Table 1 — Stack State After Each Operation

Fill in both columns after running the program. Record the contents from top to bottom.

| Operation | Linked stack (top → bottom) | Array stack (top → bottom) |
|---|---|---|
| Initial | *(empty)* | *(empty)* |
| push(10) | 10 | 10 |
| push(20) | 20, 10 | 20, 10 |
| push(30) | 30, 20, 10 | 30, 20, 10 |
| peek | 30, 20, 10 | 30, 20, 10 |
| pop | 20, 10 | 20, 10 |
| push(40) | 40, 20, 10 | 40, 20, 10 |
| push(50) | 50, 40, 20, 10 | 50, 40, 20, 10 |
| pop | 40, 20, 10 | 40, 20, 10 |
| pop | 20, 10 | 20, 10 |
| pop | 10 | 10 |

---

### Critical Thinking Questions — Model 1

**Q1.** After every operation, the linked stack and array stack print the same contents in the same order. They are implemented completely differently internally. What does this tell you about the relationship between an ADT and its implementation?

> An ADT has the capacity to behave exactly the same through various different implementations, as seen here with a linked list based and array based stack. Assuming they are implemented correctly, there can be many variations on internals that produce the same results.

**Q2.** The linked stack prints from `head` forward. The array stack prints from `top` downward (index `top` to 0). Both show elements in top-to-bottom order. What does the array `top` index physically represent — and how does it change on push versus pop?

> On an array based stack, top represents the index of the last value, so it can push and pop without shifting all of the values in the stack to match the new indices. Push increases this index by one, pop decrements by one.

**Q3.** The program does `push(10), push(20), push(30), pop, push(40), push(50), pop, pop, pop`. Before running the program, predict the return value of each `pop` in order. Check your prediction against the output.

> (30, 50, 40, 20). This matches the output of the stack trace program.

**Q4.** The array stack has a compile-time capacity of 16. The linked stack has no capacity limit. If you pushed 17 items onto the array stack what would happen? What would happen to the linked stack? What is the practical consequence of choosing each implementation for an application where maximum stack depth is unknown?

> The array stack would give a stack overflow error for trying to add more values than the array's size, unless there was an implementation to increase this automatically. The linked stack simply has to add a pointer, so it can continue to increase size until the program runs out of memory once the heap grows too large. If the maximum stack depth is entirely unknown, the linked stack is much safer. This just gives the tradeoff of possibly increasing time and space complexity due to dealing with non-contiguous memory blocks. The array stack does not have these issues with memory issues, but is unsafe in this scenario.

---

## Model 2: A Stack Solving a Real Problem

Bracket matching is a classic stack application — and a good test of whether you understand LIFO behavior at a deeper level than just "last in, first out." This program runs a bracket checker on several strings and prints its internal state after each character is processed.

### Observation Table 2 — Bracket Matching Results

Record the final RESULT printed for each input string:

| Input | Result | Reason (in your words) |
|---|---|---|
| `({[]})` | VALID | Every type has it's matching close paren in the correct order. |
| `({[}])` | INVALID | The curly brace and brackets are mismatched. |
| `((())` | INVALID | There is a missing close paren at the end. |
| `hello(world[!])` | VALID | Program skips letters, the parentheses have their matching close characters in the right order. |
| *(empty string)* | VALID | Program has nothing to check, and skips anything that isn't a ([{}]). No mismatches can occur. |

---

### Critical Thinking Questions — Model 2

**Q5.** For input `({[]})`, trace the stack contents at each step in the table below before checking your answer against the program output:

| Char | Action | Stack after (top → bottom) |
|---|---|---|
| `(` | push | ( |
| `{` | push | ({ |
| `[` | push | ({[ |
| `]` | pop/match | ({ |
| `}` | pop/match | ( |
| `)` | pop/match | *empty* |

**Q6.** For input `({[}])` the program reports INVALID. At which character does it fail, and what exactly is wrong? Why is a queue not a useful data structure for bracket matching — what ordering property of the stack makes the matching work correctly?

> It fails once the first closing character is entered, }. The end of the open characters was a bracket, so the close curly causes it to fail immediately. A queue is not useful for matching these, since it removes from what is effectively the other end in this scenario. The stack works because the top can be removed, so comparisons are made immediately and validated.

**Q7.** The input `hello(world[!])` contains letters, `!`, and brackets mixed together. The program still reports VALID. What does the program do with non-bracket characters? Why is it correct to ignore them?

> It simply ignores all non bracket characters. It is only checking if brackets match, so empty space or anything else is discarded. This behavior is mandatory, since parentheses often contain characters in between, all that matters is if each open character has a matching close charcter..

**Q8.** The empty string input reports VALID. Is this the right answer? Justify why an empty string is considered balanced.

> This case is the same principle as the previous question. The only logic needed is to check if ( has a matching ), and so on. An empty string cannot have any mismatches, and should not be considered invalid just because it is empty.;

---

## Model 3: Queue State Tracing and the Circular Array

A queue has two active ends — front and back — which makes its state harder to visualize than a stack. This program traces both a linked-list queue and a circular-array queue through the same sequence of operations, printing front, back, and contents after each step.

### Observation Table 3a — Queue Contents After Each Operation

Record front-to-back contents and dequeue return values:

| Operation | Queue contents (front → back) | Dequeue returned |
|---|---|---|
| Initial | *(empty)* | — |
| enqueue(10) | 10 | — |
| enqueue(20) | 10, 20 | — |
| enqueue(30) | 10, 20, 30 | — |
| dequeue | 20, 30 | 10 |
| enqueue(40) | 20, 30, 40 | — |
| dequeue | 30, 40 | 20 |
| dequeue | 40 | 30 |
| enqueue(50) | 40, 50 | — |
| enqueue(60) | 40, 50, 60 | — |
| enqueue(70) | 40, 50, 60, 70 | — |
| dequeue | 50, 60, 70 | 40 |
| dequeue | 60, 70 | 50 |
| dequeue | 70 | 60 |
| dequeue | *empty* | 70 |

### Observation Table 3b — Circular Array Internal State

Record the `front` index, `back` index, and `cnt` printed for the array queue after each operation:

| Operation | `front` index | `back` index | `cnt` |
|---|---|---|---|
| Initial | 0 | 0 | 0 |
| enqueue(10) | 0 | 1 | 1 |
| enqueue(20) | 0 | 2 | 2 |
| enqueue(30) | 0 | 3 | 3 |
| dequeue | 1 | 3 | 2 |
| enqueue(40) | 1 | 4 | 3 |
| dequeue | 2 | 4 | 2 |
| dequeue | 3 | 4 | 1 |
| enqueue(50) | 3 | 5 | 2 |
| enqueue(60) | 3 | 6 | 3 |
| enqueue(70) | 3 | 7 | 4 |
| dequeue | 4 | 7 | 3 |
| dequeue | 5 | 7 | 2 |
| dequeue | 6 | 7 | 1 |
| dequeue | 7 | 7 | 0 |

---

### Critical Thinking Questions — Model 3

**Q9.** Look at Table 3a. After every operation, do the linked queue and array queue always contain the same elements in the same order? What does this confirm about the two implementations?

> Yes, both implementations always contain the same elements in the same order. This confirms that they behave exactly the same as expected, they just have different internal workings.

**Q10.** Look at Table 3b. After the three initial enqueues and one dequeue, the `front` index is 1 (not 0). The element at array index 0 is logically gone — but the dequeue operation never moved any data. What did it do instead? What does this tell you about how "removal" works in a circular array queue?

> The dequeue operation removed the beginning value, and now the front index is shifted forward one index. This means that the array can dynamically change size through a removal, but the index is still available for later.

**Q11.** The circular array queue has a `cnt` field tracking the number of elements. An alternative design uses only `front` and `back` indices and considers the queue empty when `front == back`. What ambiguity arises in that design? Why is the `cnt` field (or an `isFull` flag) needed?

> Front can be equivalent to back both when the queue is full, but also when it is empty, because the two pointers are both at the "start" in this situation. A count field or isFull is necessary to actually distinguish the two scenarios.

---

## Model 4: Performance — Stack and Queue Operations at Scale

All four operations — push, pop, enqueue, dequeue — are O(1). But the *constant* hidden inside O(1) differs between a linked implementation and an array implementation. This program measures how long it takes to push/enqueue and pop/dequeue a large number of elements on both implementations.

### Observation Table 4a — Stack Performance (μs)

| n | Linked push+pop | Vector push+pop | Ratio |
|---|---|---|---|
| 100,000 | 11,806 | 3,489 | 3.38x |
| 500,000 | 48,189 | 12,923 | 3.73x |
| 1,000,000 | 76,113 | 25,146 | 3.03x |
| 5,000,000 | 383,650 | 129,902 | 2.95x |

### Observation Table 4b — Queue Performance (μs)

| n | Linked enq+deq | Array enq+deq | Ratio |
|---|---|---|---|
| 100,000 | 9,323 | 14,616 | 0.64x |
| 500,000 | 42,143 | 17,979 | 2.34x |
| 1,000,000 | 71,973 | 27,990 | 2.57x |
| 5,000,000 | 551,687 | 92,500 | 5.96x |

---

### Critical Thinking Questions — Model 4

**Q12.** Both linked and array implementations are O(1) per operation, so doubling n should double total time for both. When n grows from 100,000 to 5,000,000 (50×), by approximately what factor does each implementation's time grow? Is this consistent with O(n) total work?

> For the stack, linked list grows by 32 times, and vector grows by 37. For the queue, linked grows by 60 times, array grows by 6. These seem to be consistent with O(n), as they are linearly proportional. The last array one is vastly more quick, but is likely the result of something else.

**Q13.** The array/vector implementation is faster than the linked implementation by a consistent ratio. Both do the same logical work. What accounts for the difference? Name at least one concrete reason.

> Array style implementation benefits from contiguous memory blocks, with much better performance. On the other hand, a linked list has to chase and maintain pointers, increasing overhead and time complexity.

**Q14.** The vector stack calls `reserve(n)` before pushing. Remove the `reserve` call mentally and predict whether the timing would change significantly. What does `reserve` prevent, and what cost does it avoid?

> The timing would be greatly increased, as it would need to keep dynamically increasing the array size. If reserve was to be removed and the array size was exceeded, the array has to be copied over into a larger array to support more values. This is the  main cost avoided, memory overhead from constantly resizing, copying and expanding.

**Q15.** Based on your observations across all four models, state a concrete rule for when you would prefer a linked-list implementation of a stack or queue over an array-based one. Your rule should reference at least one specific situation where the linked implementation's properties are genuinely advantageous.

> I know that a linked-list would be vastly preferable in a situation where the size is unknown and will have to be dynamically allocated, consistently. In this scenario, an array based stack or queue would suffer from resizing and copying data into new arrays.

---


## Extra Credit Questions


**A1.** The call stack in your computer is literally a stack. Every function call pushes a frame; every return pops one. What happens when a recursive function has no base case? Connect what you observed about the array stack's overflow behavior to the term "stack overflow."

> In the case of a recursive function with no base case, function calls continue to build up and never returning values to unwind and remove frames. This results in a stack overflow, as the program runs out of memory at runtime.

**A2.** The linked queue's `dequeue` function has a single special-case line: `if (!head) tail = nullptr`. This line is easy to forget. Given what you traced in Model 3, describe exactly what breaks in a subsequent `enqueue` if this line is missing.

> This line is needed to know if the dequeue is empty. If the tail is not updated in this scenario, it could have a dangling pointer and cause the container to break when new values are added back in. The first value added into an empty array must have a pointer to null as it's tail, and this would not occur in this scenario, resulting in any sort of undefined behavior.

**A3.** `std::stack` and `std::queue` in C++ wrap `std::deque` internally rather than a raw array or linked list. Based on what you observed in Model 4, why might `std::deque` be a better default backing container than either of your two implementations?

> Queue has functions that interact with both ends of the list, and the stack is the same but more restricted. Deque has established, defined behavior, and does well being limited into the forms of stack and queue. Upon looking it up, deque combines the aspects of arrays and linked list implementations. This allows it to have better memory performance, and still behave like linked list pointers. 

---