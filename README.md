# EECE-5811 HW6

## 1

I was curious to see what ChatGPT/Claude would generate for this problem, so I asked both of them to solve it. Below are their respective solutions.

#### Prompts for ChatGPT-5 and Claude Opus 4.1:
```
Do some research and see if the following numbers have been improved since 2009. Show me results in a table with 4 columns? First column is name, second is then, third is now, fourth is improvement. Keep the same order:

Numbers Everyone Should Know
L1 cache reference 0.5 ns
Branch mispredict 5 ns
L2 cache reference 7 ns
Mutex lock/unlock 25 ns
Main memory reference 100 ns
Compress 1K bytes with Zippy 3,000 ns
Send 2K bytes over 1 Gbps network 20,000 ns
Read 1 MB sequentially from memory 250,000 ns
Round trip within same datacenter 500,000 ns
Disk seek 10,000,000 ns
Read 1 MB sequentially from disk 20,000,000 ns
Send packet CA->Netherlands->CA 150,000,000 ns
```

```
Give me the table code in markdown
```

### ChatGPT's Response:

| Name | Then (2009) | Now (2020–2025 Approx.) | Improvement |
|---|---|---|---|
| L1 cache reference | 0.5 ns | 0.5–1 ns | ~Same |
| Branch mispredict | 5 ns | 3–5 ns | Slight improvement |
| L2 cache reference | 7 ns | 4–7 ns | Slight improvement |
| Mutex lock/unlock | 25 ns | ~17 ns | ~30% faster |
| Main memory reference | 100 ns | 90–100 ns | ~Same |
| Compress 1K bytes with Zippy | 3,000 ns | ~2,000 ns | ~1.5× faster |
| Send 2K bytes over 1 Gbps | 20,000 ns | ~20,000 ns | ~Same (bandwidth-bound) |
| Read 1 MB sequentially from memory | 250,000 ns | 20,000–60,000 ns | 4–10× faster |
| Same-datacenter round trip | 500,000 ns | 300,000–500,000 ns | Slight improvement |
| Disk seek (HDD) | 10,000,000 ns | 8,000,000–10,000,000 ns | ~Same (HDD physics-limited) |
| Read 1 MB sequentially from disk (HDD) | 20,000,000 ns | 5,000,000–20,000,000 ns | Up to ~4× faster |
| Packet CA → Netherlands → CA | 150,000,000 ns | 130,000,000–160,000,000 ns | ~Same (speed-of-light limited) |

#### Claude Response:

| **Operation** | **2009 (Original)** | **2024 (Current)** | **Improvement** |
|---|---|---|---|
| L1 cache reference | 0.5 ns | 1-2 ns | Worse (2-4x slower) |
| Branch mispredict | 5 ns | ~3-5 ns | Similar/Slight improvement |
| L2 cache reference | 7 ns | 4-5 ns | Better (~1.5x faster) |
| Mutex lock/unlock | 25 ns | 13 ns | Better (~2x faster) |
| Main memory reference | 100 ns | 60-100 ns | Similar/Slight improvement |
| Compress 1K bytes with Zippy | 3,000 ns (3 μs) | ~2,000 ns (2 μs) | Better (~1.5x faster) |
| Send 2K bytes over 1 Gbps network | 20,000 ns (20 μs) | ~20 μs for 1Gbps | Similar |
| Read 1 MB sequentially from memory | 250,000 ns (250 μs) | ~200,000 ns (200 μs) | Slight improvement |
| Round trip within same datacenter | 500,000 ns (500 μs) | 100-300 μs | Better (2-5x faster) |
| Disk seek | 10,000,000 ns (10 ms) | 250 μs for NVMe SSD | **Massively better (40x faster)** |
| Read 1 MB sequentially from disk | 20,000,000 ns (20 ms) | ~300 μs for NVMe SSD | **Massively better (65x faster)** |
| Send packet CA→Netherlands→CA | 150,000,000 ns (150 ms) | ~140-150 ms | Similar |

### My Analysis

I did tested my own hardware to verify these numbers. Here are my findings:

| **Operation** |  **Latency** | **Improvement** |
|---|---|---|---|
| L1 cache reference | ~0.6 ns | Similar |
| Branch mispredict | ~2.1 ns | Improvement, same order of magnitude |
| L2 cache reference | ~1.3 ns | Improvement, same order of magnitude |
| Mutex lock/unlock | ~10.0 ns | Improvement |
| Main memory reference (random) | ~8 ns | 10x improvement |
| Compress 1K bytes with Zippy | ~300 ns | 10x improvement |
| Send 2K bytes over 1 Gbps network | untested | untested |
| Read 1 MB sequentially from memory | ~40,000.0 ns | ~6x improvement |
| Round trip within same datacenter | untested | untested |
| Disk seek | untested | untested |
| Read 1 MB sequentially from disk | untested | untested |
| Send packet CA→Netherlands→CA | untested | untested |

### Conclusion

After researching and testing on my own hardware, the first 3 numbers have barely changed since 2009. Mutex/Unlock number is within the same order of magnitude but has slightly improved, though I found that this number can vary from kernel to kernel. The rest of the numbers (aside from network related ones) have seen significant improvements because of SSDs and faster memory technologies. Interestingly, the AI models found that most of these numbers have noteworthy improvements (Claude saying L1 cache reference have actually gotten worse....) when realistically many of them are within the same order of magnitude.

#### Sources:
- https://www.cs.cornell.edu/projects/ladis2009/talks/dean-keynote-ladis2009.pdf
- https://colin-scott.github.io/personal_website/research/interactive_latency.html
- https://github.com/sirupsen/napkin-math
- https://kyle.cascade.family/posts/latency-numbers-every-programmer-should-know-revisited/
- https://samwho.dev/numbers/
- https://claude.ai
- https://chat.openai.com

## 2

### Q1

Copy-on-write (COW) is an optimization strategy used to efficiently manage memory and resource usage. In fork(), COW delays the copying of physical page tables from a parent process to a child process until it is actually modified. On a fork() call, a page table is created for the child process with entries that reference the same memory location used as the parent process. Only when one of the references attempts to modify the object does the system write a separate physical copy for that reference.

### Q2

Pseudocode for fork() with COW:

- fork() called
- kernel creates a new process structure for child with its own page table
- for each page table entry (PTE) in the parent's page table:
    - copy the PTE to the child's page table
    - increment the reference count for the physical page
    - if the PTE is marked writable:
        - mark the PTE as read-only
        - mark the PTE with a COW flag
- when either process tries to write to a page:
    - a page fault occurs due to write attempt on read-only page
    - kernel handles the page fault:
        - if the page is marked as COW:
            - allocate a new physical page with the writeable permission
            - increment the reference count for the new physical page
            - copy the contents from the original page to the new page
            - update the page table entry for the writing process (whether its the parent or child) to point to the new page
            - remove the COW flag from the page table entry
            - decrement the reference count for the original physical page
            - if the reference count reaches zero:
                - free the original physical page
            - resume the process that caused the page fault
        - else:
            - handle as a normal page fault (e.g., segmentation fault)


#### Sources:
- https://www.rose-hulman.edu/class/csse/csse332/2324b/labs/cow/
- https://www.cs.swarthmore.edu/~kwebb/cs31/s15/bucs/fork_and_exec.html
- https://xiayingp.gitbook.io/build_a_os/labs/lab-5-copy-on-write-fork-for-xv6
- https://pdos.csail.mit.edu/6.828/2019/labs/cow.html
- https://man7.org/linux/man-pages/man2/fork.2.html 