Queue-Based Stock Market Order Matching Engine

A pure queue-based order matching engine that enforces strict **price-time priority**. Each price level has its own FIFO queue, so earlier orders at the same price are always matched first, and best-price access stays fast.

> **Course:** Data Structures and Algorithms
> **Institution:** Saveetha Institute of Medical and Technical Sciences (SIMATS Engineering)
> **Author:** Sudarshan S (1925260264)
> **Guide:** Dr. s .Sravani

---

## Table of Contents

- [Overview](#overview)
- [Objectives](#objectives)
- [Why Queue-Based?](#why-queue-based)
- [Design](#design)
- [Architecture](#architecture)
- [Matching Pipeline](#matching-pipeline)
- [How Drawbacks Are Addressed](#how-drawbacks-are-addressed)
- [Implementation Plan](#implementation-plan)
- [Evaluation Metrics](#evaluation-metrics)
- [Project Status](#project-status)
- [References](#references)

---

## Overview

An order matching engine pairs incoming buy and sell orders in real time. Stock markets need this to be **fair** (price-time priority) and **low-latency** (thousands of orders per second).

Common approaches such as sorted maps, heaps, or arrays often trade away FIFO fairness for speed, or the reverse. This project proposes a queue-per-price-level design that is fair, fast, and easy to reason about.

**Order matching cycle:**

1. **Receive:** incoming buy/sell order
2. **Match:** against the best-priced queue on the opposite side
3. **Execute:** trades are generated and queues updated
4. **Rest:** any unmatched portion stays in the book

## Objectives

| Objective | Description |
| --- | --- |
| Price-time priority | Strict priority using FIFO queues at each price level |
| Low-latency matching | Microsecond-scale matching using efficient queue and map structures |
| Fairness guarantee | Earlier orders at the same price are always matched first |
| Order book integrity | Consistent bid/ask book under concurrent inserts and cancels |
| Performance evaluation | Measure throughput, latency, and fairness against alternative data structures |

## Why Queue-Based?

| Existing approach | Limitation |
| --- | --- |
| TreeMap + List | Good price ordering, but list iteration during matching is slow |
| Priority queue / heap | Fast best-price access, but weak FIFO time priority |
| Array of price levels | Very fast for discrete prices, but memory-heavy and rigid |
| Skip-list book | Good average performance, but complex to implement correctly |
| Hybrid commercial engines | Highly optimized, but proprietary and hard to study |

Consequences: fairness is not always guaranteed, latency spikes appear under load, and cancellation handling is complex.

## Design

- **Price-level queues:** each distinct price owns a FIFO queue of orders, giving strict time priority.
- **Bid and ask maps:** sorted maps (or arrays) hold the active price levels, giving O(1) or O(log n) best-price access.
- **Matching logic:** an incoming order walks the opposite side's queues until it is filled or its price limit is reached.
- **Cancellation support:** orders can be removed from the middle of a queue via handles or a linked structure.
- **Order ID index:** fast cancel by order reference.
- **Concurrency control:** a lock-free or carefully locked design keeps the book consistent under high throughput.

## Architecture

```
Incoming Orders (Market / Limit / Cancel)
        │
        ▼
  Order Gateway          (validate & timestamp)
        │
        ▼
  Order Book Core        (bid queues + ask queues)
        │
        ▼
  Matching Engine        (price-time priority matcher)
        │
        ├──► Trade Generator   (partial fills & residuals)
        ├──► Book Update       (remove filled, requeue rest)
        │         │
        │         ▼
        │    Updated Order Book (consistent state for next order)
        │
        └──► Market Data Feed  (publish BBO & trades)
```

## Matching Pipeline

1. **Ingest:** receive and validate the order
2. **Match:** walk the opposite side's best-price queues
3. **Execute:** generate trades and update remaining quantities
4. **Update:** maintain book integrity (remove filled orders, requeue residuals)

## How Drawbacks Are Addressed

| Drawback in existing systems | Response in this design |
| --- | --- |
| Fairness vs. latency trade-off | A dedicated FIFO queue per price guarantees time priority without scanning |
| Cancellation complexity | Order handles and linked queues allow O(1) cancel while preserving order |
| Memory and cache pressure | Compact price maps and optional price-level pooling reduce footprint |
| Implementation complexity | Clear separation of price map and per-price queues simplifies reasoning |
| Scalability limits | Scales linearly with active price levels and is easy to profile |
| Lack of open benchmarks | Throughput, latency, and fairness measured in a controlled simulator |

## Implementation Plan

1. **Core book:** price-level maps holding FIFO queues of orders
2. **Matcher:** walk opposite-side queues and generate trades
3. **Workload:** synthetic order flow with a market/limit/cancel mix
4. **Benchmark:** compare the queue design against heap and pure-tree alternatives

## Evaluation Metrics

| Metric | Definition |
| --- | --- |
| Throughput | Orders matched per second |
| Latency | Time from order arrival to match completion |
| Fairness | Percentage of correct time-priority matches |
| Memory | Bytes per active order / price level |

Design-analysis scores (1-5) rated each structure on fairness, latency, throughput, memory, implementation ease, and cancel support. Price-level Queue and Array of Queues scored highest (3.67) and Naive Scan lowest (2.67). These are indicative scores from design analysis, not live exchange benchmarks.

## Project Status

This is currently a **design and analysis phase** project. The next step is a Python prototype to measure throughput, latency, and fairness directly.

<!-- Once code exists, add: Installation, Usage, Running benchmarks, Project structure -->

## References

1. M. O'Hara, "High frequency market microstructure," *Journal of Financial Economics*, 2015.
2. R. Cont, S. Stoikov, and R. Talreja, "A stochastic model for order book dynamics," *Operations Research*, 2010.
3. J. W. Lockwood et al., "A low-latency library in FPGA for high-frequency trading," *IEEE Hot Interconnects*, 2012.
4. C. Lee and S. Seo, "Design of a high-performance matching engine," technical reports, 2018.
5. NASDAQ OMX, "Matching engine architecture overview," technical whitepaper, 2020.
6. A. Polychronopoulos et al., "Hardware acceleration of limit order books," 2016.
7. R. Cont, "Statistical modeling of high-frequency financial data," 2011.
