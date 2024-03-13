# CSCI3390 Project 1 - Zihan Ni

## Introduction
This project introduces Spark and Scala for data processing, guiding through setup, tool installation. The report aims to summarize the hands-on Bitcoin mining simulation, which is designed to find nonces producing hashes with specific leading zeros, detailing findings from running at various difficulty levels on both a local machine and Google Cloud Platform (GCP).

## Local Machine Execution

On the local machine, the program is tested through the difficulty from 2 - 6, with different numbers of trails. Here is the result:

| Difficulty (`k`) | Number of Trials (`n`) | Nonce (`xS`)                                   | Hash Value                                                        | Time Elapsed (s) |
|------------------|------------------------|------------------------------------------------|-------------------------------------------------------------------|------------------|
| 2                | 100                    | 1625962345			             | 007c1a3c1d3ba8a452f5a4bacbbe093d6a0294644f4c2fc2c14f92bd3684ea22  | 1                |
| 3                | 10,000                 | 237772082       				     | 000e295b041a6aa0f295026851deac23cc11ac5c96329294df62590202ae7e41  | 1                |
| 4                | 100,000                | 793098413 				     | 0000c8e8ca060b522686eb4d51c3f80110dcf411636095df962ee3fc40904148  | 1                |
| 5                | 1,000,000              | 1568556591				     | 0000094d3a87a6e1ba33aeb5e1d6f31afdbc52d5d3c9ada9bc561cb7fa86a616  | 2                |
| 6                | 10,000,000             | 1578318045	    			     | 0000000cae6e248aeb158b11b091616c8f42453de7e8c471738aeb2fb671f56e  | 5                |


## GCP Execution Summary for `k = 7`

### Result Overview

- **Nonce (`xS`):** `627111892`
- **Hash Value:** `0000000d0803d4bd8a1534af876c70a8898ff9d516d0bec54ae2e7778e01061b`
- **Number of Trials (`n`):** `1,000,000,000`
- **Time Elapsed:** `331 seconds`

### Cluster Configuration

- **Region/Zone:** `us-central1`, `us-central1-c`
- **Image Version:** `2.1.42-debian11`
- **Machine Type:** `n2-standard-4` (4 vCPUs, 16 GB memory)
- **Primary Disk Size:** `500GB`
- **Scheduled Deletion:** Idle > 2 hours
- **Cluster Type:** Single Node
- **Autoscaling:** Off
- **Network:** Default
- **Project Access:** Full API access


## Modification to Nonce Generation

The nonce generation code in `src/main/scala/project_1/main.scala` was modified from a randomized approach to a sequential one to explore the efficiency of different search strategies in finding a valid nonce. The original line of code:

```scala
val nonce = sc.range(0, trials).mapPartitionsWithIndex((indx, iter) => {
  val rand = new scala.util.Random(indx + seed)
  iter.map(x => rand.nextInt(Int.MaxValue - 1) + 1)
})
```
This was changed to:
```scala
val nonce = sc.range(0, trials).mapPartitionsWithIndex((indx, iter) => {
  iter.map(x => x + 1) // Modified line
})
```
After changing the code, the run time to find Bitcoin, in the difficulty of 7, takes 3000s.

This design would have both pros and cons:
**Pros:**
- **Efficiency:** Less resource-intensive than random generation.
- **Predictability:** Guarantees each number is generated once, ensuring systematic search.

**Cons:**
- **No Randomness:** Could miss advantages of stochastic processes.
- **Load Balancing:** May create hotspots in distributed systems without careful management.

In summary, the deterministic method offers thorough coverage and predictability, ideal for smaller, controlled environments. In contrast, the randomized approach suits large, distributed systems by reducing overlap through independent exploration, albeit with efficiency dependent on luck and variance.

