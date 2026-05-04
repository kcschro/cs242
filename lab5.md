# Lab 5: Observing Hash Tables

**CPTR 242 — Sequential and Parallel Data Structures and Algorithms**
Walla Walla University

---

## Overview

In this lab you will **run complete programs and observe their behavior**. There is no coding required. Your job is to read the output carefully, record what you see, and reason about what it means.

By the end you should be able to:

- Trace a hash table through a sequence of insertions and look up the expected slot for any key
- Explain how chaining handles collisions using a linked list per bucket
- Describe how linear probing finds an open slot and why deletion requires tombstones
- Compare hash table performance against a sorted array and reason about when O(1) average breaks down
- Recognize the load-factor threshold that triggers resizing and understand why rehashing is necessary

---

## Background: From Search to O(1) Lookup

Every data structure you have studied so far — arrays, linked lists, trees — locates a value by *comparing* keys. Binary search on a sorted array takes O(log n); a balanced BST takes O(log n). A hash table sidesteps comparison entirely: it computes an array index from the key and accesses that slot directly. When the hash function distributes keys uniformly and the table is not too full, every lookup, insertion, and deletion takes O(1) average time.

The central question: **where does that O(1) come from, and when does it break?**

---

## Model 1: Hash Function Behavior and Slot Distribution

Before examining a full hash table, you need to understand what a hash function actually does to a set of keys. This program hashes a fixed list of strings using a polynomial rolling hash and prints the raw hash value, the slot index (`hash % TABLE_SIZE`), and the final slot assignment for each key.

### Observation Table 1a — Hash Slot Assignments

Fill in the slot each key maps to:

| Key | Slot (0–10) |
|---|---|
| alice | 10 |
| bob | 4 |
| carol | 6 |
| dave | 3 |
| eve | 5 |
| frank | 5 |
| grace | 4 |
| heidi | 4 |
| ivan | 10 |
| judy | 3 |

### Observation Table 1b — Slot Occupancy

| Slot | Number of keys assigned |
|---|---|
| 0 | 0 |
| 1 | 0 |
| 2 | 0 |
| 3 | 2 |
| 4 | 3 |
| 5 | 2 |
| 6 | 1 |
| 7 | 0 |
| 8 | 0 |
| 9 | 0 |
| 10 | 2 |

---

### Critical Thinking Questions — Model 1

**Q1.** How many slots are empty after inserting 10 keys into an 11-slot table? How many collisions occurred? Does the distribution look roughly uniform, or are keys clumped heavily in a few slots?

> After inserting 10 keys, 6 slots are empty, and 4 slots have collisions. Overall, the distribution is not very even, and keys are in fact rather clumped.

**Q2.** The hash function multiplies by 31 before adding each character. What would happen if you simply summed the character values without multiplying? Give an example of two different strings that would collide under the summing approach but likely not under the polynomial approach.

> Both bat and tab would sum to the same value without multiplying each before summing. Since they are palindromes/anagrams, they have the same characters and each character needs to be multiplied and then added to the next to avoid collision.

**Q3.** TABLE_SIZE is chosen to be prime (11). Suppose you changed it to 10 (not prime). Would you expect more or fewer collisions? What property of prime-sized tables makes them distribute keys more evenly?

> You would expect more collisions due to the nature of modulo. If you're hashing even numbers, then they will clump in even indices, since modulo ten will only create even indices from these numbers. Prime numbers result in more varied indices since they are only divisible by themselves and one.

**Q4.** The load factor after 10 insertions is approximately 0.91. This is high. What are the consequences of a high load factor for (a) a chaining table and (b) an open-addressing table?

> High load factor increases much more quickly for open-addressing than chaining, so open-addressing often has a lower threshold to resize. Open-addressing slows down much more quickly, but chaining also increases memory consumption and has to chase pointers.

---

## Model 2: Chaining — Insertion, Lookup, and Collision Handling

Chaining resolves collisions by storing a linked list at each bucket. This program builds a chaining hash table from a sequence of key–value pairs, prints the internal state of every bucket after each insertion, and then performs several lookups showing the probe path through the chain.

### Observation Table 2a — Table State After Each Insertion

For each insertion, record which slot the key lands in and whether a collision occurred (another key was already in that slot):

| Key inserted | Slot | Collision? (Y/N) | Chain length after insertion |
|---|---|---|---|
| alice | 6 | N | 1 |
| bob | 4 | N | 1 |
| carol | 2 | N | 1 |
| dave | 3 | N | 1 |
| eve | 6  | Y | 2 |
| frank | 1 | N | 1 |
| grace | 1 | Y | 2 |
| heidi | 6 | Y | 3 |

### Observation Table 2b — Lookup Probe Counts

| Key searched | Slot | Probes needed | Found? |
|---|---|---|---|
| alice | 6 | 3 | Yes |
| carol | 2 | 1 | Yes |
| grace | 1 | 1 | Yes |
| zara | 0 | 0 | No |

---

### Critical Thinking Questions — Model 2

**Q5.** When two keys land in the same slot, the new key is **prepended** to the chain (inserted at the front), not appended to the back. Does this affect correctness? Does it affect performance? Would you ever prefer to append instead?

> Prepended makes sense because if an item is appended, then this operation loses the speed of constant time. Any item has to walk back to be appended. You would only want to prepend in a specific case where it needs LIFO ordering.

**Q6.** The `search("zara")` call walks an entire chain and finds nothing. How many nodes did it examine? In terms of load factor λ, what is the expected number of probes for an unsuccessful search in a chaining table?

> If the search for zara walks an entire chain, the number of nodes checked depends on which bucket. If we assumine it is slot 6, then it checked 3 nodes. Since the load factor is 1.14, there is an expected 1.14 probes to execute an unsuccesful search. 

**Q7.** After all 8 insertions into a 7-slot table the load factor exceeds 1.0. Chaining still works. Open addressing would not — why not? What invariant does open addressing require that chaining does not?

> Open addressing can only place one key in every slot. If the table is full, or greater than one, it will infinitely loop, being unable to insert new keys. Open addressing **must** have available slots.

**Q8.** Look at the final table printout. Some slots have chains of length 2 or more; others are empty. Even with a good hash function, perfect uniformity is not guaranteed. What is the theoretical expected maximum chain length for n keys in a table of size n, under uniform hashing?

> In the scenario of perfect hashing, there would be 1 key in a table with n keys of size n. Under uniform hashing however, this is a more mathematically complicated concept. Upon looking it up, this should be O(logn/(log(logn))). Hypothetically, you could have all the keys in one, so a maximum chain length of n, but this is absurdly statistically improbable to occur in uniform.

---

## Model 3: Linear Probing — Insertion, Clustering, and Tombstones

Linear probing stores everything in the array itself — no linked lists, no extra allocation. On collision it steps forward by 1 until an empty slot is found. This program inserts a sequence of keys, prints the full array after each insertion (showing which slots are occupied and where keys "landed" after probing), then demonstrates the tombstone deletion problem.

### Observation Table 3a — Slot Placement After Each Insertion

Record where each key ended up (its final slot index) and how many extra probes were needed beyond the home slot:

| Key | Home slot (`hash % 11`) | Final slot | Extra probes | Caused by cluster? |
|---|---|---|---|---|
| alice | 10 | | 0 | |
| bob | 4 | | 0 | |
| carol | 6 | | 0 | |
| dave | 3 | | 0 | |
| eve | 5 | | 0 | |
| frank | 5 | 7 | 2 | Y |
| grace | 4 | 8 | 4 | Y |

### Observation Table 3b — Search Probe Paths After Tombstone Insertion

After `bob` is deleted (tombstone), record the slots visited for each search:

| Key searched | Slots visited (in order) | Probes | Result |
|---|---|---|---|
| carol (before delete) | 6 | 1 | |
| carol (after delete) | 6 | 1 | |
| alice | 10 | 1 | |
| grace | 4,5,6,7,8 | 5 | |
| zara | 7,8,9 | 2 | Not found |

---

### Critical Thinking Questions — Model 3

**Q9.** Look at the extra-probe column in Table 3a. Several keys required probing past their home slot. This is **primary clustering** — runs of occupied slots that grow longer over time. Describe in your own words why a new key hashing *anywhere into* an existing cluster makes the cluster grow longer.

> If a new key hashes into a cluster and must be moved, it will simply probe forward until it finds the next empty space. If keys keep colliding here, then the cluster just grows, as the key that needs to be moved is always essentially appended to the cluster.

**Q10.** After `bob` is deleted, the search for `carol` must cross the tombstone. What would happen if the tombstone were replaced with an EMPTY marker instead — would the search for `carol` succeed or fail? Explain why.

> If there was an empty marker at this position, the search would fail instead of finding the key for carol. The search starts at the expected corresponding hashed slot, and probes until an EMPTY slot is found. The tombstone allows probing to the end of the cluster.

**Q11.** Tombstones accumulate over time. A slot marked DELETED counts as "occupied" for searches (you must keep probing past it) but as "available" for insertions (you can place a new key there). What happens to average probe length as the fraction of tombstone slots grows? How does rehashing fix this?

> Probe length should increase on average as tombstones also increase. There are gaps in the cluster, and tombstones allow this searchable area to continue growing. Rehashing would redistribute the keys without the tombstones, and while there still may be a cluster, it would be shorter with a lack of tombstones in the middle.

**Q12.** Linear probing requires λ < 1 (the table can never be completely full). Chaining from Model 2 allowed λ > 1. Why does open addressing impose this strict limit, while chaining does not?

> If λ reaches a value of one, then the table is full. Linear probing will always fail, as there are no more slots to find. The slowdown as λ approaches one is also significant. Chaining does not mandate this limit, since the "buckets" can hold multiple keys with linked-lists.

---

## Model 4: Collision Strategy Comparison and Load Factor Effects

Linear probing degrades quickly as the table fills. Double hashing avoids clustering by using a second hash function to compute the step size. This program inserts the same keys into three tables — chaining, linear probing, and double hashing — and reports the average probe count for both successful and unsuccessful searches at several load factors.

### Observation Table 4a — Measured Average Probe Counts

| Load factor (λ) | Chaining | Linear probing | Double hashing |
|---|---|---|---|
| ~0.10 | 1 | 1 | 1 |
| ~0.25 | 1 | 1 | 1 |
| ~0.50 | 1.14 | 3.52 | 1.28 |
| ~0.69 | 1.271 | 5.957 | 1.814 |
| ~0.89 | 1.356 | 8.756 | 2.322 |

### Observation Table 4b — Theoretical Linear vs. Double Hashing Predictions

| Load factor (λ) | Linear (theory) | Double (theory) |
|---|---|---|
| 0.10 | 1.056 | 1.054 |
| 0.25 | 1.167  | 1.151 |
| 0.50 | 1.500 | 1.386  |
| 0.69 | 2.113 | 1.697 |
| 0.89 | 5.045 | 2.480 |

---

### Critical Thinking Questions — Model 4

**Q13.** At λ = 0.50 (half full), linear probing already takes noticeably more probes than double hashing. At λ ≈ 0.89 the gap widens dramatically. In your own words, explain *why* double hashing avoids the primary clustering that hurts linear probing.

> Double hashing avoids primary clustering by using a second hash function to introduce a step size for probes. Linear probing requires walking down the list until an empty slot is found to insert, whereas double hashing jumps forward, resulting in more even distribution.

**Q14.** Chaining's probe count grows slowly and almost linearly with λ — and it still works above λ = 1.0. Open-addressing schemes blow up as λ → 1. From the table, at what load factor does linear probing exceed 3 probes on average? What does that tell you about a safe upper bound for λ in a linearly-probed table?

> For open addressing, open-addressing breaks 3 probes on average at a load factor of 0.5. This seems to suggest that an upper bound for lambda could reasonably be around 0.5 to 0.7, as shown in the table.

**Q15.** Both theoretical formulas (linear and double hashing) are derived assuming **uniform random hashing** — every key equally likely in any slot. Your measured values are based on a specific deterministic hash function on synthetic keys. Are the measured values close to the theoretical predictions? What might cause discrepancies?

> The measured values in table 4a are relatively the same as the theoreticals in 4b. Double hashing measured and theory are nearly identical, while linear shoots off a bit faster in the measured table. Perhaps synthetic keys were not themselves random enough to represent the general predicted case. The nature of synthetic keys being present could cause variations.

---


*Next lab: trees — where we trade O(1) average for O(log n) guaranteed and gain the ability to iterate keys in sorted order.*