#  Assignment 10: Distributed Database Analysis
**Option 1: Distributed Databases using Replication**

**Student Name:** Hanane, Gary , Yang

**Git Repository:** 
* https://github.khoury.northeastern.edu/yangzhao/HW10
*  https://github.khoury.northeastern.edu/wangchong/CS6650-repo/tree/main/assignment10 
---


## 1. Executive Summary
This analysis characterizes the performance boundaries of **Leader-Follower** and **Leaderless** database architectures under varying simulated network constraints.

Our load testing reveals a distinct hierarchy in performance efficiency driven by the specific consistency requirements of each configuration. **The `W=5, R=1` Leader-Follower configuration emerged as the most performant architecture** for this test harness, delivering **3x faster read latencies** (~34ms) compared to the Quorum `W=3, R=3` setup (~90ms), without incurring a write penalty.

While the **Leaderless** architecture demonstrated sub-millisecond read capabilities ($\approx$ 0.76ms), its application is limited by the trade-off of eventual consistency and high write-coordination costs.

---

## 2. Experimental Constraints & Methodology
To isolate the impact of architectural choices, artificial delays were injected to simulate real-world processing and geographic latency:
* **Leader Write Processing:** 200ms (simulating replication broadcast overhead).
* **Follower Write Processing:** 100ms (simulating disk I/O or commit logic).
* **Follower Read Processing:** 50ms (simulating local lookup latency).
* **Leader Read Processing:** 0ms (simulating cached/optimized hot-path).

---

## 3. Workload Analysis: 1% Write Ratio (Read Heavy)

In this scenario, the system is almost exclusively serving reads. The `W3R3` configuration suffers significantly here due to the mandatory follower sleep delay on every read request.

### Leader-Follower (W=5, R=1)
<div style="border: 2px solid #cccccc; padding: 10px; margin-bottom: 10px; text-align: center; background-color: #f9f9f9;">
  <strong>[GRAPH: Latency Distribution]</strong><br>
  <img width="4163" height="1765" alt="latency_distribution_w5r1_1w" src="https://github.com/user-attachments/assets/82af2d52-d513-4847-b4ae-95c744da396c" />
<br>
  <em>File: latency_distribution_w5r1_1w.png</em>
</div>

<div style="border: 2px solid #cccccc; padding: 10px; margin-bottom: 10px; text-align: center; background-color: #f9f9f9;">
  <strong>[GRAPH: Interval Distribution]</strong><br>
<img width="2970" height="1745" alt="interval_distribution_w5r1_1w" src="https://github.com/user-attachments/assets/9684a2f5-97eb-45c4-8d63-50c69de82b6d" />
  <br>
  <em>File: interval_distribution_w5r1_1w.png</em>
</div>

### Leader-Follower (W=3, R=3)
<div style="border: 2px solid #cccccc; padding: 10px; margin-bottom: 10px; text-align: center; background-color: #f9f9f9;">
  <strong>[GRAPH: Latency Distribution]</strong><br>
 <img width="4170" height="1765" alt="latency_distribution_w3r3_1w" src="https://github.com/user-attachments/assets/d0e8560a-f4b5-4db2-9fa8-f5c191fc5b56" />
</br>
  <em>File: latency_distribution_w3r3_1w.png</em>
</div>

<div style="border: 2px solid #cccccc; padding: 10px; margin-bottom: 10px; text-align: center; background-color: #f9f9f9;">
  <strong>[GRAPH: Interval Distribution]</strong><br>
 <img width="2970" height="1745" alt="interval_distribution_w3r3_1w" src="https://github.com/user-attachments/assets/ce70fa7e-4698-4dc5-b5d2-96c792d27ce1" />
  <br>
  <em>File: interval_distribution_w3r3_1w.png</em>
</div>

> **Observation:** Note the "long tail" in the W3R3 Interval distribution compared to W5R1. The architecture itself acts as a rate limiter.

---

## 4. Workload Analysis: 10% Write Ratio

As writes increase slightly, the read latency advantage of W5R1 continues to dominate the aggregate metrics, keeping intervals short.

### Leader-Follower (W=5, R=1)
<div style="border: 2px solid #cccccc; padding: 10px; margin-bottom: 10px; text-align: center; background-color: #f9f9f9;">
  <strong>[GRAPH: Latency Distribution]</strong><br>
 <img width="4170" height="1765" alt="latency_distribution_w5r1_10w" src="https://github.com/user-attachments/assets/9a358187-e284-48e5-8bc5-18223d87880c" />

  <br>
  <em>File: latency_distribution_w5r1_10w.png</em>
</div>

<div style="border: 2px solid #cccccc; padding: 10px; margin-bottom: 10px; text-align: center; background-color: #f9f9f9;">
  <strong>[GRAPH: Interval Distribution]</strong><br>
<img width="2970" height="1745" alt="interval_distribution_w5r1_10w" src="https://github.com/user-attachments/assets/bcd6848c-695b-4859-87a2-e3184a2084c8" />

  <br>
  <em>File: interval_distribution_w5r1_10w.png</em>
</div>

### Leader-Follower (W=3, R=3)
<div style="border: 2px solid #cccccc; padding: 10px; margin-bottom: 10px; text-align: center; background-color: #f9f9f9;">
  <strong>[GRAPH: Latency Distribution]</strong><br>
  <img width="4170" height="1765" alt="latency_distribution_w3r3_10w" src="https://github.com/user-attachments/assets/8273256a-3351-4c1e-8268-fe3a8c8574ce" />

  <br>
  <em>File: latency_distribution_w3r3_10w.png</em>
</div>

<div style="border: 2px solid #cccccc; padding: 10px; margin-bottom: 10px; text-align: center; background-color: #f9f9f9;">
  <strong>[GRAPH: Interval Distribution]</strong><br>
<img width="2970" height="1745" alt="interval_distribution_w3r3_10w" src="https://github.com/user-attachments/assets/c2e2f180-aae4-4cf6-a173-6c52bca6f5b1" />

  <br>
  <em>File: interval_distribution_w3r3_10w.png</em>
</div>

---

## 5. Workload Analysis: 50% Write Ratio (Balanced)

Here we observe the equalization of write latencies between the Leader-Follower configurations. We also see the Leaderless architecture's interval distribution for the first time.

### Leader-Follower (W=5, R=1)
<div style="border: 2px solid #cccccc; padding: 10px; margin-bottom: 10px; text-align: center; background-color: #f9f9f9;">
  <strong>[GRAPH: Latency Distribution]</strong><br>

  <img width="4170" height="1765" alt="latency_distribution_w5r1_50w" src="https://github.com/user-attachments/assets/64e72585-f9d9-4e3f-b0af-51646026e631" />

  <br>
  <em>File: latency_distribution_w5r1_50w.png</em>
</div>

<div style="border: 2px solid #cccccc; padding: 10px; margin-bottom: 10px; text-align: center; background-color: #f9f9f9;">
  <strong>[GRAPH: Interval Distribution]</strong><br>
  <img width="2970" height="1745" alt="interval_distribution_w5r1_50w" src="https://github.com/user-attachments/assets/e32a3475-3a51-47a4-8b29-5a8fb75077bb" />

  <br>
  <em>File: interval_distribution_w5r1_50w.png</em>
</div>

### Leader-Follower (W=3, R=3)
<div style="border: 2px solid #cccccc; padding: 10px; margin-bottom: 10px; text-align: center; background-color: #f9f9f9;">
  <strong>[GRAPH: Latency Distribution]</strong><br>
<img width="4170" height="1765" alt="latency_distribution_w3r3_50w" src="https://github.com/user-attachments/assets/f43c0bad-20e3-45f7-a703-08fcaa2e44e7" />

  <br>
  <em>File: latency_distribution_w3r3_50w.png</em>
</div>

<div style="border: 2px solid #cccccc; padding: 10px; margin-bottom: 10px; text-align: center; background-color: #f9f9f9;">
  <strong>[GRAPH: Interval Distribution]</strong><br>
<img width="2970" height="1745" alt="interval_distribution_w3r3_50w" src="https://github.com/user-attachments/assets/72103b9a-6223-462f-b1b4-18c0d1c5d8ed" />

  <br>
  <em>File: interval_distribution_w3r3_50w.png</em>
</div>

### Leaderless Architecture
*Note: Only Interval distribution data was captured for this workload.*
<div style="border: 2px solid #cccccc; padding: 10px; margin-bottom: 10px; text-align: center; background-color: #f9f9f9;">
  <strong>[GRAPH: Interval Distribution]</strong><br>

  <img width="2970" height="1745" alt="interval_distribution_50w" src="https://github.com/user-attachments/assets/467fc2bc-e352-4a60-a60f-69af42ac7e15" />

  <br>
  <em>File: interval_distribution_50w.png</em>
</div>

---

## 6. Workload Analysis: 90% Write Ratio (Write Intensive)

This tier provides the most comprehensive comparison, containing complete latency and interval data for all three architectures.

### Leader-Follower (W=5, R=1)
<div style="border: 2px solid #cccccc; padding: 10px; margin-bottom: 10px; text-align: center; background-color: #f9f9f9;">
  <strong>[GRAPH: Latency Distribution]</strong><br>
  <img src="latency_distribution_w5r1_90w.png" alt="Latency W5R1 90%" width="600"/><br>
  <em>File: latency_distribution_w5r1_90w.png</em>
</div>

<div style="border: 2px solid #cccccc; padding: 10px; margin-bottom: 10px; text-align: center; background-color: #f9f9f9;">
  <strong>[GRAPH: Interval Distribution]</strong><br>
  <img src="interval_distribution_w5r1_90w.png" alt="Interval W5R1 90%" width="600"/><br>
  <em>File: interval_distribution_w5r1_90w.png</em>
</div>

### Leader-Follower (W=3, R=3)
<div style="border: 2px solid #cccccc; padding: 10px; margin-bottom: 10px; text-align: center; background-color: #f9f9f9;">
  <strong>[GRAPH: Latency Distribution]</strong><br>
  <img src="latency_distribution_w3r3_90w.png" alt="Latency W3R3 90%" width="600"/><br>
  <em>File: latency_distribution_w3r3_90w.png</em>
</div>

<div style="border: 2px solid #cccccc; padding: 10px; margin-bottom: 10px; text-align: center; background-color: #f9f9f9;">
  <strong>[GRAPH: Interval Distribution]</strong><br>
  <img src="interval_distribution_w3r3_90w.png" alt="Interval W3R3 90%" width="600"/><br>
  <em>File: interval_distribution_w3r3_90w.png</em>
</div>

### Leaderless Architecture
<div style="border: 2px solid #cccccc; padding: 10px; margin-bottom: 10px; text-align: center; background-color: #f9f9f9;">
  <strong>[GRAPH: Latency Distribution]</strong><br>
  <img src="latency_distribution_90w.png" alt="Latency Leaderless 90%" width="600"/><br>
  <em>File: latency_distribution_90w.png</em>
</div>

<div style="border: 2px solid #cccccc; padding: 10px; margin-bottom: 10px; text-align: center; background-color: #f9f9f9;">
  <strong>[GRAPH: Interval Distribution]</strong><br>
  <img src="interval_distribution_90w.png" alt="Interval Leaderless 90%" width="600"/><br>
  <em>File: interval_distribution_90w.png</em>
</div>

---

## 7. Deep Dive: Leaderless Architecture Statistics

The Leaderless configuration exhibits the most extreme variance in performance characteristics. Below are the aggregated statistics capturing the request/response behavior.

<div style="border: 2px solid #cccccc; padding: 10px; margin-bottom: 10px; text-align: center; background-color: #f9f9f9;">
  <strong>[TABLE: Request Statistics]</strong><br>
<img width="835" height="840" alt="图像" src="https://github.com/user-attachments/assets/ec9c5862-1fbd-4384-8045-a001fb651f3c" />

  
  <br>
  <em>File: 图像.png</em>
</div>

| Metric | Value | Implication |
| :--- | :--- | :--- |
| **Read Latency (Mean)** | **0.76ms** | Near-instant access; bypasses network/sleeps. |
| **Write Latency (Mean)** | **309ms** | Coordination cost is high; practically identical to Leader-based systems. |
| **Staleness Risk** | High | $R=1$ in a leaderless system creates a massive inconsistency window. |

> **Architectural Observation:** The Leaderless setup effectively trades **Consistency** for **Read Availability**. It is the only architecture that could serve a read request if the network partitioned the client from the majority of nodes.

---

## 8. Strategic Recommendations & Conclusion

Based on the full suite of latency distributions, we classify the architectures by their optimal production roles:

### 8.1. The "Golden Path": Leader-Follower (W=5, R=1)
* **Performance Profile:** High-speed reads ($\approx 34ms$), moderate writes.
* **Why it wins:** This configuration effectively masks the cost of replication by handling writes in parallel while optimizing the "hot path" (reads) to hit the single fastest node (the Leader).
* **Operational Risk:** The $W=5$ setting is technically "brittle." In a real cluster, requiring *all* nodes to acknowledge a write means a single node failure causes total write unavailability. 
* **Recommendation:** In production, this would likely be deployed as **$W=3, R=1$** (Async replication) to maintain the read speed while removing the fragility of synchronous replication to all nodes.

### 8.2. The "High Availability" Safety Net: Leader-Follower (W=3, R=3)
* **Performance Profile:** Slow reads ($\approx 90ms$), moderate writes.
* **Why use it:** This is the standard "Quorum" configuration used by systems like Cassandra or DynamoDB when data integrity is paramount. It guarantees that even if the Leader dies and a partition occurs, you can still read/write as long as a majority exists.
* **Trade-off:** You pay a **200-300% latency tax** on every read operation to buy this insurance.
* **Recommendation:** Use only for mission-critical data (e.g., billing, identity management) where `Consistency > Latency`.

### 8.3. The "Speed Demon": Leaderless Architecture
* **Performance Profile:** Instant reads ($<1ms$), slow writes.
* **Why use it:** It bypasses the network entirely for reads, serving directly from local memory.
* **Trade-off:** It abandons Strong Consistency. A user reading from Node A might see data seconds older than a user reading from Node B.
* **Recommendation:** Ideal for "Write-Once, Read-Many" workloads where freshness is negotiable but speed is not. Examples include **social media feeds, product recommendation engines, and IoT sensor logging**.
