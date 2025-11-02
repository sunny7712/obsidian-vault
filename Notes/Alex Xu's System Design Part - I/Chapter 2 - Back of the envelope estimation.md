#system-design #books #notes
# Back of the envelope estimation

* Back of the envelope estimates are created using a combination of thought experiments and common performance numbers, to get a good feel for which designs will meet your requirements.

## 1) Power of two

* One ASCII character is 1 byte (8 bits)

| Power | Approximate Value | Size (Name) |
| :--- | :--- | :--- |
| $2^{10}$ | ~1000 ($10^3$) | 1 KB (Kilobyte) |
| $2^{20}$ | 1 Million ($10^6$) | 1 MB (Megabyte) |
| $2^{30}$ | 1 Billion ($10^9$) | 1 GB (Gigabyte) |
| $2^{40}$ | 1 Trillion ($10^{12}$) | 1 TB (Terabyte) |
| $2^{50}$ | 1 Quadrillion ($10^{15}$) | 1 PB (Petabyte) |

# Types of Cache in memory

## L1 Cache

The smallest and quickest type of cache memory. It is directly embedded into the CPU, allowing it to operate at the same speed as CPU. It's divided into two parts:

* (i) For storing instructions (L1i)
* (ii) For storing data (L1d)
* Size typically ranges from 2KB to 64KB.

## (ii) L2 Cache

* Slightly larger but slower than L1.
* Closer to the CPU than main memory.
* May be located in or outside CPU.
* Can be shared across multiple cores or be exclusive to one core depending on the architecture.
* 256KB to 512KB.

## (iii) L3 Cache

* Larger than L1 and L2, but slower than them.
* Outside the CPU, shared by all the cores.
* 1MB to 8MB.

---

# Cache Mapping Techniques

* Decides how memory blocks are loaded into the cache.

## (i) Direct Mapping

* A specific block of main memory can only go into one specific line in cache. The address for the cache line is determined by:
    `(Main memory block address) % (Number of cache lines)`
* **Pros:** Very simple, fast. No searching required.
* **Cons:** High chance of cache miss if two or more "hot" blocks of memory map to the same cache line. (Conflict miss)

## (ii) Fully Associative Mapping

* Block of memory can be placed anywhere into any available line of cache.
* Must decide on an eviction policy (LRU).
* **Pros:** Best cache hit rate and completely avoids conflict misses.
* **Cons:** Searching is expensive. Complex to build.

## (iii) Set-Associative Mapping

* The cache is first divided into a number of groups called 'sets'. A block from main memory first maps to one of the groups (Direct mapping), but it can then be placed in any available line in that set (Fully Associative).
* Number of lines per set is called "Associativity".
* In practice, this technique is used in L1, L2, and L3 caches.
* **Pros:** Solves the conflict miss problem without the extreme cost of fully associative.
* **Cons:** "Conflict misses" can still happen (if you need more hot blocks that map to the same set than there are lines in that set). More complicated than Direct Mapping.

---

# Availability numbers

* High availability is the ability of the system to be operational for a desirably long period of time.
* 100% availability = zero downtime.
* **SLA** - Service Level Agreement formally defines the uptime of the service.

| Availability | Downtime per day | Downtime per year |
| :----------- | :--------------- | :---------------- |
| 99%          | 14.40 minutes    | 3.65 days         |
| 99.9%        | 1.44 minutes     | 8.77 hours        |
| 99.99%       | 8.64 seconds     | 52.60 minutes     |
| 99.999%      | 864 millisec     | 5.26 min          |
| 99.9999%     | 86.4 millisec    | 31.56 seconds     |

---

# Latencies every programmer should know

- https://highscalability.com/google-pro-tip-use-back-of-the-envelope-calculations-to-choo/
- https://colin-scott.github.io/personal_website/research/interactive_latency.html