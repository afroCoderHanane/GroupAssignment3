#  Assignment 10: Distributed Database Analysis
**Option 1: Distributed Databases using Replication**

**Student Name:** Hanane, Gary , Yang

**Git Repository:** 
* https://github.khoury.northeastern.edu/yangzhao/HW10
*  https://github.khoury.northeastern.edu/wangchong/CS6650-repo/tree/main/assignment10 
---

## 1. Executive Summary
This report analyzes the performance and consistency trade-offs of a distributed Key-Value store ($N=5$) under four distinct configurations: **Strong Consistency ($W=5, R=1$)**, **Quorum ($W=3, R=3$)**, **High Availability ($W=1, R=5$)**, and **Leaderless ($W=N, R=1$)**.

To simulate Wide Area Network (WAN) characteristics, the following delays were enforced:
* **Leader Write:** 200ms delay per follower.
* **Follower Write:** 100ms processing delay.
* **Follower Read:** 50ms processing delay.

**Key Findings:**
* **Strategy 2 ($W=1$)** achieved the fastest writes (**41.46ms**) by bypassing the synchronous replication delay.
* **Strategy 1 ($W=5$)** maintained extremely stable but high write latency (**~339ms**) while offering fast reads (**~34ms**).
* **Leaderless** architecture provided the fastest raw reads (**0.76ms**) but exhibited operational instability (73 failures recorded).

---

## 2. Strategy 1: Strong Consistency ($W=5, R=1$)
**Configuration:** The Leader waits for *all* 5 nodes to acknowledge a write. Reads are served locally by the Leader (or synced replica).

### Performance Data
* **Write Latency (Mean):** Consistently **337ms - 339ms**.
* **Read Latency (Mean):** Consistently **33ms - 35ms**.
* **Stability:** The variance is low. P95 and Mean are very close (e.g., 339ms Mean vs 350ms P95), indicating deterministic behavior.

### Latency Graphs
**1% Write / 99% Read:**
Graph of W5R1 1% Write
<img width="2970" height="1745" alt="interval_distribution_w5r1_1w" src="https://github.com/user-attachments/assets/5a36a9b3-4aa1-4bbb-aeae-7725a8dc45ce" />


**10% Write / 90% Read:**
Graph of W5R1 10% Write
<img width="2970" height="1745" alt="interval_distribution_w5r1_10w" src="https://github.com/user-attachments/assets/4b29569f-1427-44cc-8cf4-3633f9992eb1" />


**50% Write / 50% Read:**
Graph of W5R1 50% Write
<img width="2970" height="1745" alt="interval_distribution_w5r1_50w" src="https://github.com/user-attachments/assets/919cf7a1-8a60-4d83-95ba-4474e3248720" />


**90% Write / 10% Read:**
Graph of W5R1 90% Write
<img width="2965" height="1745" alt="interval_distribution_w5r1_90w" src="https://github.com/user-attachments/assets/ddf9349a-bbeb-42d0-aef5-a815a49fac2f" />


### Analysis
The data confirms Strategy 1 is **Read-Optimized**.
* **Why:** The read latency (~34ms) is consistently low because $R=1$ allows the node to answer immediately. The Write Latency is dominated by the "slowest follower" effect. Since we wait for all 5 nodes, the 200ms transmission delay + 100ms processing delay + overhead results in the observed ~340ms floor.

---

## 3. Strategy 3: Quorum Consensus ($W=3, R=3$)
**Configuration:** Balanced approach. Writes wait for 3 nodes; Reads query 3 nodes.

### Performance Data
* **Write Latency (Mean):** **~337ms - 338ms**.
* **Read Latency (Mean):** **~85ms - 94ms**.
* **Tail Latency:** P99 Read latency spiked to **132ms** (in the 50% test), showing higher variance than Strategy 1.

### Latency Graphs
**10% Write / 90% Read:**
Graph of W3R3 10% Write
<img width="2970" height="1745" alt="interval_distribution_w3r3_10w" src="https://github.com/user-attachments/assets/560e6fac-10cd-4f29-a4f8-3d2f66519d2c" />


**50% Write / 50% Read:**
Graph of W3R3 50% Write
<img width="2970" height="1745" alt="interval_distribution_w3r3_50w" src="https://github.com/user-attachments/assets/314165ce-fa10-49ca-83c6-31a7abcbd9d7" />


**90% Write / 10% Read:**
Graph of W3R3 90% Write
<img width="2970" height="1745" alt="interval_distribution_w3r3_90w" src="https://github.com/user-attachments/assets/44da7611-02e0-4545-81cd-fdc419cfd90d" />


### Analysis
* **The "Quorum Anomaly":** Interestingly, our data shows that $W=3$ writes (~338ms) are **statistically identical** to $W=5$ writes (~339ms). Theoretically, waiting for 3 nodes should be faster than 5. The fact that they are identical suggests that the Leader's broadcasting mechanism (sending the 200ms-delayed messages) is likely serial or CPU-bound, masking the benefit of waiting for fewer acknowledgments.
* **The Read Penalty:** Reading is roughly **3x slower** than Strategy 1 (94ms vs 34ms). This represents the network round-trip cost of contacting 2 extra peers.

---

## 4. Strategy 2: High Availability ($W=1, R=5$)
**Configuration:** Writes return immediately (Async replication). Reads query ALL nodes to find the newest version.

### Performance Data
* **Write Latency (Mean):** **41.46ms** (Fastest of all Leader strategies).
* **Read Latency (Mean):** **98.36ms** (Slowest).
* **P99 Write:** 72.59ms.

### Latency Graph
**90% Write / 10% Read:**
Graph of W1R5 90% Write
<img width="4170" height="1765" alt="latency_distribution_90w" src="https://github.com/user-attachments/assets/97daaaac-ecd1-4f1e-bdfa-624ef799f0a2" />


### Analysis
This configuration successfully "cheated" the CAP theorem for writes.
* **Why:** By setting $W=1$, the Leader ignores the 200ms artificial delay during the client request. It commits locally and returns success in **41ms**.
* **The Trade-off:** The cost is shifted to the reader. The mean read latency of **98ms** is the highest of all groups because the reader must wait for the *slowest* of the 5 nodes to respond before it can determine which version is the latest.

---

## 5. Leaderless Architecture ($W=N, R=1$)
**Configuration:** Decentralized. Any node acts as coordinator.

### Request Statistics
Table of Leaderless Statistics
<img width="835" height="840" alt="图像" src="https://github.com/user-attachments/assets/42265a84-076c-4f93-90e1-93931b93c329" />


### Analysis
* **Speed vs. Stability:** The Leaderless reads are blazing fast (**0.76ms**), likely because the local read path bypasses the Leader logic entirely.
* **The Failure Rate:** The table shows **73 aggregated failures**. This proves that the Leaderless coordination overhead ($W=N$) under load causes timeouts or dropped messages, making it less reliable than the Leader-Follower strategies despite its raw read speed.

---

## 6. Comparative Data Summary
The following table aggregates the mean latency results collected from all test runs.

| Strategy | Workload (Write/Read) | Avg Write Latency | Avg Read Latency | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **Strong ($W=5$)** | 1% / 99% | **339.03 ms** | **34.85 ms** | Best for Read-Heavy |
| | 10% / 90% | 338.08 ms | 32.62 ms | |
| | 50% / 50% | 339.56 ms | 34.83 ms | |
| | 90% / 10% | 337.52 ms | 33.47 ms | Slowest Writes |
| **Quorum ($W=3$)** | 1% / 99% | **338.80 ms** | **87.02 ms** | |
| | 10% / 90% | 338.62 ms | 85.35 ms | |
| | 50% / 50% | 337.56 ms | 86.13 ms | Balanced / Safe |
| | 90% / 10% | 337.46 ms | 94.85 ms | |
| **High Avail ($W=1$)** | 90% / 10% | **41.46 ms** | **98.36 ms** | **Fastest Writes** / Slowest Reads |
| **Leaderless** | Aggregated | 309.00 ms | **0.76 ms** | **Fastest Reads** (but Unstable) |

---

## 7. Final Recommendations

Based on the empirical data collected above:

### 1. Best for Read-Heavy Apps (e.g., DNS, CDNs)
**Recommendation:** **Strategy 1 ($W=5, R=1$)**
* **Data Support:** It offers **34ms** reads vs 98ms for Strategy 2. The write penalty (339ms) is acceptable when writes are only 1% of traffic.

### 2. Best for Write-Heavy Apps (e.g., IoT Sensors, Clickstreams)
**Recommendation:** **Strategy 2 ($W=1, R=5$)**
* **Data Support:** This is the **only** strategy that keeps write latency under 300ms. It achieved **41ms** writes, which is an **8x improvement** over Strategies 1 and 3.

### 3. Best for Mixed/Consistency-Critical Apps (e.g., Banking)
**Recommendation:** **Strategy 3 ($W=3, R=3$)**
* **Data Support:** While Strategy 3 didn't offer a write speedup in our specific tests (due to implementation overhead), it provides the mathematical guarantee of a Quorum. It avoids the potential data loss of Strategy 2 ($W=1$) and prevents the "Single Point of Read Failure" of Strategy 1, keeping both latencies in a predictable range (338ms Write / 90ms Read).
