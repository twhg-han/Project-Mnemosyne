# Chapter 3.1: System Formalization

## 3.1.1 Introduction: Necessity of Formal Modeling

The design philosophy of the Mnemosyne system rests upon a rigorous mathematical foundation. To ensure behavioral predictability, verifiable correctness, and implementation consistency across platforms, we formally describe the core system components using a 6-tuple model $\mathcal{M}=(M,F,\pi,\delta,\tau,\mathcal{E})$.

This definition encompasses the entire stack, from OS-level memory mapping (mmap), file descriptor management, and page replacement strategies, to information-theoretic compression mechanisms, hardware configuration parameters, and the computational security layer. Together, these elements constitute a complete and verifiable theoretical framework.

Establishing this formal model is not merely a requirement for academic rigor but a necessary measure to address specific engineering challenges:

* **Multi-Platform Heterogeneity**: Edge devices such as Raspberry Pi, Jetson Orin, and Intel NUC exhibit significant differences in Memory Management Units (MMU), Translation Lookaside Buffer (TLB) configurations, and SIMD instruction sets. Formal definitions ensure algorithmic consistency across diverse hardware architectures.
* **Latency Predictability**: The strict P50 and P99 latency requirements for edge AI inference (target \< 20 μs, \< 100 μs) necessitate a quantitative model for low-level behaviors, including L1/L2/L3 cache hierarchies, DRAM access, and Page Fault handling.
* **Formal Verification Requirements**: This study intends to employ TLA+ and Z3 SMT Solvers in subsequent phases to verify system correctness. The 6-tuple model provides the strict semantic basis required for these verification tools.

The structure of this chapter is organized as follows:

* **§3.1.2** Defines the six core elements of the tuple $\mathcal{M}$.
* **§3.1.3** Details the hierarchical architecture and latency model of the memory regions $M$.
* **§3.1.4** Elucidates the Zero-Copy semantics of the file mapping function $F$.
* **§3.1.5** Introduces the semantic-aware page replacement strategy $\pi$ and its improved LRU algorithm.
* **§3.1.6** Explains the information-theoretic basis of the delta encoding function $\delta$ (**Revised**).
* **§3.1.7** Defines the minimum requirements for hardware configuration $\tau$.
* **§3.1.8** Defines the computational security semantics of the encryption operator $\mathcal{E}$.
* **§3.1.9** Discusses system invariants and the TLA+ specification.
* **§3.1.10** Summarizes the chapter and key formal contributions.

----

## 3.1.2 The 6-Tuple Model: Core Definition

The Mnemosyne memory system is defined as a 6-tuple:

$$\mathcal{M}=(M,F,\pi,\delta,\tau,\mathcal{E})$$

Where:

* $M$: **Set of Memory Regions**, defining the structure of the virtual address space.
* $F$: **File Descriptor Mapping**, implementing zero-copy access.
* $\pi$: **Page Replacement Policy**, optimized based on LLM access patterns.
* $\delta$: **Delta Encoding Function**, implementing information-theoretic compression.
* $\tau$: **Hardware Configuration**, defining minimum system requirements.
* $\mathcal{E}$: **Encryption Operator**, providing computational security guarantees.

----

## 3.1.3 Element One: $M$ (Memory Regions)

### Definition

$M$ is defined as a set of non-overlapping, contiguous memory regions within the Virtual Address Space (VAS). Here, each memory region $m_i$ is treated as a set of addresses; thus, its boundaries can be represented by both set notation and a half-open interval.

$$M=\{m_{1},m_{2},\dots,m_{N_m}\}\subseteq\mathcal{P}(VAS)$$

Each memory region $m_{i}$ is a half-open interval:

$$m_{i}=[base\_addr_{i},\,base\_addr_{i}+size_{i})$$

These regions satisfy the mutual exclusion condition:

$$\forall i\neq j,\;m_{i}\cap m_{j}=\emptyset$$

### Working Set Definition

Let $M(t)$ denote the set of memory regions actually mapped by the process at time $t$, satisfying $M(t)\subseteq M$.

$$WorkingSet(t)=\sum_{m_i\in M(t)} size_i$$

This represents the sum of the sizes of regions mapped at time $t$. When $M(t) = \emptyset$, we define $WorkingSet(t) = 0$. The working set satisfies the boundary condition:

$$0\le WorkingSet(t)\le C_{RAM}$$

### Alignment Constraints

To maximize CPU Cache utilization and avoid Split Loads (accesses crossing Cache Lines), Mnemosyne enforces **16 KB Tile alignment**:

$$base\_addr_{i}\equiv0 \pmod{16384}$$

This constraint ensures that each Tile (16 KB) precisely covers 256 64-byte Cache Lines, aiming to achieve optimal access patterns on common architectures such as ARM Cortex-A72 (32 KB L1 Cache) and Intel Core i7 (32 KB L1 Cache).

### Capacity Constraints

To prevent the system from entering a state of Thrashing, the total mapped size is restricted by a safety threshold relative to physical RAM:

$$\sum_{i=1}^{N_m} size_{i}\le 0.7\,C_{RAM}$$

When mapped regions exceed 70% of RAM, the frequency of the Linux kernel's Page Replacement is expected to rise sharply, potentially degrading P99 latency from the microsecond scale ($\mu$s) to the millisecond scale (ms). When $WorkingSet > C_{RAM}$, the system enters a Thrashing state, and the $\pi$ policy activates aggressive eviction mechanisms to maintain system stability.

### Latency Model: Hierarchical Memory Levels

For any address $p\in \bigcup m_{i}$, its access latency $T_{access}(p)$ depends on the hierarchy level in which it resides:

$$T_{access}(p)=
\begin{cases}
0.5 \text{ ns} & p\in\text{L1 Cache (32 KB)}\\
7 \text{ ns} & p\in\text{L2 Cache (256 KB)}\\
100 \text{ ns} & p\in\text{DRAM}\\
T_{pagefault}+T_{disk} & \text{Load from SSD required}
\end{cases}$$

Where $T_{pagefault}\approx1\text{–}5\,\mu s$, and $T_{disk}$ is detailed by storage type:

*   NVMe SSD: 10–50 μs
*   SATA SSD: 100–500 μs
*   SD Card: 1–10 ms

### Cache Hit Rate Baseline Model

If the working set $W$ and Cache capacity $C$ satisfy:

$$HitRate(W,C)=\begin{cases} 1 & W\le C \\ \frac{C}{W} & W>C \end{cases}$$

The expected value of the average latency is:

$$T_{avg}=T_{hit}\cdot HitRate+T_{miss}\cdot (1-HitRate)$$

**Numerical Verification Example**: For LLM Embedding access ($d=4096$, FP16, 8KB per vector), if L2 Cache = 256 KB, it can accommodate 32 vectors. When the sequence length $N>32$ (e.g., $N=1000$):

$$T_{avg}=7 \text{ ns}\cdot \frac{32}{1000}+100 \text{ ns}\cdot \left(1-\frac{32}{1000}\right)\approx 0.224+96.8\approx 97 \text{ ns}$$

This theoretical calculation (approximately 97 ns) serves as the baseline for subsequent performance optimizations.

----

## 3.1.4 Element Two: $F$ (File Descriptor Mapping)

### Definition

$F$ defines the mapping relationship between the file system and memory regions $M$:

$$F:M\to \text{FilePath}\times \text{Offset}\times \text{Length}\times \text{Protection}$$

For any \( m_i \in M \), its mapping attributes are defined as a 4-tuple:

\( m_i = (\text{file\_path}_i, \text{offset}_i, \text{length}_i, \text{prot}_i) \)

Where:

* \( \text{file\_path}_i \): The file path on the disk (e.g., embeddings.mmap).

* \( \text{offset}_i \): The starting offset within the file, which must be page-aligned (based on x86-64/ARMv8 standard 4KB page size):

$$\text{offset}_i \equiv 0 \pmod{4096}$$

*   $\text{length}_i$: Mapping length (bytes), must satisfy $\text{length}_i=\text{size}_i$.
*   $\text{prot}_i\in \{\text{PROT\_READ},\text{PROT\_READ}|\text{PROT\_WRITE}\}$: Memory protection mode.

### POSIX mmap System Call Mapping

$F$ corresponds to the POSIX `mmap` system call:

```c
void* mmap(void* addr, size_t length, int prot, int flags, int fd, off_t offset);
```

Where:

*   `addr = base_addr_i` (Suggested value, determined by the kernel)
*   `length = length_i`
*   `prot = prot_i`
*   `flags = MAP_SHARED` (Shared mapping, supporting multi-process access)
*   `fd = open(file_path_i, O_RDWR)` (File descriptor)
*   `offset = offset_i`

**Zero-Copy Semantics**: Through `mmap`, reading from or writing to $m_i$ directly manipulates the file system buffer, eliminating data copying between user space and kernel space, thereby reducing access latency to O(1).

### Mapping State Transition

We define the mapping state function $\sigma: M \to \{\text{Unmapped}, \text{Mapped}\}$:

$$\sigma(m_i, t) = \begin{cases}
\text{Mapped} & \exists fd: F(m_i) = (*, *, *, *) \land \text{mmap}(fd) \text{ has been called} \\
\text{Unmapped} & \text{otherwise}
\end{cases}$$

When $\sigma(m_i, t) = \text{Mapped}$, the probability of a Page Fault triggered by accessing $p \in m_i$ is 0 (assuming no Swapping occurs).

----

## 3.1.5 Element Three: $\pi$ (Page Replacement Policy)

### Definition

$\pi$ is defined as a Semantic-Aware Page Replacement Policy, which optimizes the LRU algorithm based on the locality of LLM Token sequences.

$$\pi: \mathcal{P} \times \mathbb{N} \to \mathcal{P}$$

Where $\mathcal{P}$ is the set of pages (4KB per page), and $\mathbb{N}$ is the access timestamp.

### Improved LRU Algorithm

The standard LRU policy evicts pages based on the most recent access time:

$$\pi(P, t) = \arg\min_{p \in P} \text{last_access\_time}(p)$$

Mnemosyne introduces **semantic weights** $w(p)$ to adjust eviction priority based on Token importance (e.g., Attention Score):

$$\pi(P, t) = \arg\min_{p \in P} \left[ \text{last_access\_time}(p) - \lambda \cdot w(p) \right]$$

Where $\lambda > 0$ is a weight adjustment parameter (default 0.5).

**Semantic Weight Calculation**: For the Token sequence $\{tok_j\}$ covered by page $p$:

$$w(p) = \frac{1}{|p|} \sum_{tok_j \in p} \text{attention\_score}(tok_j)$$

This improvement ensures that pages containing high-attention tokens are prioritized for retention in memory.

### Thrashing Defense

When $WorkingSet(t) > 0.7 C_{RAM}$, the system initiates an aggressive mode:

$$\pi(P, t) = \text{select k least weighted pages}, \quad k = \lceil (WorkingSet(t) - 0.7 C_{RAM}) / 4096 \rceil$$

Selected pages are released via `munmap` to restore system stability.

----

## 3.1.6 Element Four: $\delta$ (Delta Encoding Function)

### Definition

$\delta$ is defined as the Delta Encoding Function, used for information-theoretic compression of LLM Embedding sequences.

$$\delta: \mathbb{R}^d \times \mathbb{R}^d \to \mathbb{R}^d$$

It is specifically defined as a vector difference operation:

$$\delta(V_i, V_{i-1}) = V_i - V_{i-1}, \quad i \in [1, n]$$

Where $V_i \in \mathbb{R}^d$ is the $i$-th embedding vector, and $d$ is the embedding dimension (for LLaMA-2-7B, $d=4096$).

### Information-Theoretic Basis

According to **Theorem 5.1 (Chapter 3.2)**, when the Pearson correlation coefficient of adjacent vectors satisfies $\rho > 0.5$, the differential entropy of the difference vector satisfies:

$$h(\delta(V_i, V_{i-1})) = h(V_i) + \frac{d}{2} \log_2[2(1-\rho)] < h(V_i)$$

**Theorem 5.2 (Chapter 3.3)** verifies that the joint Gaussian assumption and the condition $\rho > 0.5$ hold.

**Theorem 5.3 (Chapter 3.4)** proves the complete invertibility of $\delta$:

$$V_i = V_0 + \sum_{j=1}^{i} \delta(V_j, V_{j-1})$$

It also provides bounds for floating-point numerical stability (relative error \< 0.024%).

### Compression Loop

The complete compression pipeline is as follows:

1.  Calculate the difference sequence $\{\Delta_i = \delta(V_i, V_{i-1})\}_{i=1}^n$.
2.  Quantize $\Delta_i$ to INT8 (based on the low-variance distribution of $\Delta_i$).
3.  Store $V_0$ + quantized $\{\Delta_i\}$.

Theoretical Compression Rate: Based on Theorem 5.1 and 5.2, we estimate a 10.88% gain (relative to FP16).

----

## 3.1.7 Element Five: $\tau$ (Hardware Configuration)

### Definition

$\tau$ is defined as the Hardware Configuration, specifying the minimum requirements for system operation.

$$\tau = (C_{CPU}, C_{RAM}, C_{STORAGE}, B_{NETWORK})$$

Where:

*   $C_{CPU}$: At least a 4-core ARM Cortex-A76 or equivalent x86 CPU, supporting NEON/AVX2 SIMD.
*   $C_{RAM}$: At least 8 GB LPDDR4, frequency ≥ 3200 MHz.
*   $C_{STORAGE}$: At least 16 GB NVMe SSD or eMMC 5.1, read speed ≥ 500 MB/s.
*   $B_{NETWORK}$: At least 1 Gbps Ethernet or Wi-Fi 6.

### Theoretical Calculation Basis

Based on the LLaMA-2-7B inference workload:

*   KV Cache (KV-Cache): \~1 GB
*   Model Weights (4-bit): \~3.5 GB
*   **Total Working Set**: \~4.5 GB

Minimum RAM required:

$$C_{RAM}\ge \frac{4.5 \text{ GB}}{0.7}\approx 6.4 \text{ GB}$$

Therefore, the recommended configuration of 8 GB is a conservative estimate consistent with theoretical calculations.

----

## 3.1.8 Element Six: $\mathcal{E}$ (Encryption Operator)

### Definition

$\mathcal{E}$ is defined as the set of encryption and decryption operators for the Computational Security Layer, designed to provide IND-CPA (Indistinguishability under Chosen Plaintext Attack) security guarantees.

$$\mathcal{E} = (E, D, \mathcal{K}, \mathcal{IV})$$

Where:

*   $E: \mathcal{K} \times \mathcal{IV} \times \mathcal{M} \to \mathcal{C} \times \mathcal{T}$ is the encryption function.
*   $D: \mathcal{K} \times \mathcal{IV} \times \mathcal{C} \times \mathcal{T} \to \mathcal{M} \cup \{\bot\}$ is the decryption function.
*   $\mathcal{K}$: 256-bit Key Space.
*   $\mathcal{IV}$: 96-bit Initialization Vector Space.
*   $\mathcal{M}$: Plaintext Space (compressed bitstream).
*   $\mathcal{C}$: Ciphertext Space.
*   $\mathcal{T}$: 128-bit Authentication Tag.

### Implementation Specifications

Mnemosyne adopts **AES-GCM-256** as the concrete implementation of $\mathcal{E}$ to satisfy requirements for high throughput and Authenticated Encryption with Associated Data (AEAD).

*   **Encryption Process**: $C, T = \text{AES-GCM-Enc}(K, IV, M, AAD)$
*   **Decryption Process**: $M = \text{AES-GCM-Dec}(K, IV, C, T, AAD)$

Here, $AAD$ (Associated Data) binds the Token ID and sequence number to prevent replay attacks.

### Integration with $\delta$

The difference sequence can be encrypted before storage:
$$\text{Ciphertext} = E_K(\delta(V_i, V_{i-1}))$$

The difference vector is recovered after decryption:
$$\Delta_i = D_K(\text{Ciphertext})$$

This combination provides dual protection: "Information-Theoretic Privacy (entropy reduction via $\delta$) + Computational Security (encryption via $\mathcal{E}$)."

----

## 3.1.9 System Invariants and TLA+ Specification

### Invariants

**Inv\_1 (Memory Bound Invariant)**:

$$\text{WorkingSet}(t)\le0.7C_{\mathrm{RAM}}
   \;\wedge\;
   (\forall m\in M,\ \text{size}(m)\le MAX_REGION_SIZE)$$

**Inv\_2 (Page Alignment Invariant)**:

$$\forall m\in M,\ \text{base\_addr}(m)\equiv0 \pmod{16384}$$

**Inv\_3 (No Data Race Invariant)**:

$$\forall t_i\neq t_j,\;
(WriteSet_i\cap WriteSet_j=\emptyset)\;\vee\;
(LockSet_i\cap LockSet_j\neq\emptyset)$$

**Inv\_4 (Delta Encoding Invertibility Invariant)**:

$$\forall i \in [1,n],\; V_i = V_0 + \sum_{j=1}^{i} \delta(V_j, V_{j-1})$$

This invariant ensures that the original vector sequence can be fully recovered via $\delta^{-1}$ under any system state.

### TLA+ Specification Summary

```tla
---- MODULE MnemosyneInvariants ----
EXTENDS Naturals, Reals, Sequences

CONSTANTS Regions, PHYSICAL_RAM, MAX_REGION_SIZE, THREADS, VECTOR_DIM

VARIABLES size, base_addr, write_set, locks, V_sequence, Delta_sequence

RECURSIVE Sum(_)
Sum(seq) == IF seq = <<>> THEN 0 ELSE Head(seq) + Sum(Tail(seq))

MemoryBound ==
    /\\ \A i \in DOMAIN size: size[i] <= MAX_REGION_SIZE
    /\\ Sum(size) <= PHYSICAL_RAM * 0.7

PageAlignment ==
    \A i \in DOMAIN base_addr: base_addr[i] % 16384 = 0

NoDataRace ==
    \A i, j \in THREADS: i # j =>
        (write_set[i] \cap write_set[j] = {}) \\/
        (locks[i] \cap locks[j] # {})

DeltaEncodingInvertibility ==
    \A i \in 1..Len(V_sequence)-1:
        V_sequence[i] = V_sequence[^0] + Sum(SubSeq(Delta_sequence, 1, i))

SystemInvariant ==
    /\\ MemoryBound
    /\\ PageAlignment
    /\\ NoDataRace
    /\\ DeltaEncodingInvertibility

=========================================
```

----

## 3.1.10 Summary

This chapter has established the complete formal definition of the Mnemosyne system. The 6-tuple model $\mathcal{M}=(M,F,\pi,\delta,\tau,\mathcal{E})$ covers OS-level memory management ($M,F$), intelligent page replacement ($\pi$), information-theoretic compression ($\delta$), hardware configuration ($\tau$), and the computational security layer ($\mathcal{E}$), constituting a rigorous and verifiable theoretical framework.

### Key Formal Contributions

1.  **Hierarchical Latency Quantification**: Established theoretical latency models for L1/L2/DRAM/SSD as benchmarks for performance estimation.
2.  **Zero-Copy Semantic Definition**: Formalized mmap behavior, distinguishing between logical mapping and physical loading.
3.  **Delta Encoding Closed Loop** [Key Revision Focus]:
 *   Concretized the $\delta$ function definition: $\delta(V_i, V_{i-1}) = V_i - V_{i-1}$
 *   Cited **Theorem 5.1** as the theoretical basis for entropy reduction.
 *   Cited **Theorem 5.2** as validation for statistical assumptions.
 *   Cited **Theorem 5.3** as a guarantee for invertibility and numerical stability.
4.  **Computational Security Closed Loop**: Introduced the encryption operator $\mathcal{E}$, completing the security theory for wartime modes.
5.  **Safety Invariants**: Defined four system-level invariants (adding Inv\_4), laying the foundation for TLA+ verification.

### Relationship with Subsequent Chapters

*   **Chapter 3.2 (Theorem 5.1)**: Proves the entropy reduction theory of $\delta$.
*   **Chapter 3.3 (Theorem 5.2)**: Verifies the statistical prerequisites for $\delta$.
*   **Chapter 3.4 (Theorem 5.3)**: Proves the invertibility and error bounds of $\delta$.
*   **Chapter 3.5 (Theorem 5.4)**: Extends to lossy compression (PQ).

This chapter provides a unified formal semantic framework for the entire Mnemosyne system, ensuring consistency between theoretical and implementation levels across all elements.

----

## References

1.  Denning, P. J. (1968). The working set model for program behavior. *Communications of the ACM*, *11*(5), 323–333.
2.  Kerrisk, M. (2010). *The Linux Programming Interface*. No Starch Press.
3.  Cover, T. M., & Thomas, J. A. (2006). *Elements of Information Theory* (2nd ed.). Wiley-Interscience.
4.  Lamport, L. (2002). *Specifying Systems: The TLA+ Language and Tools*. Addison-Wesley.
5.  Gorman, M. (2004). *Understanding the Linux Virtual Memory Manager*. Prentice Hall.
6.  Daemen, J., & Rijmen, V. (2002). *The Design of Rijndael: AES - The Advanced Encryption Standard*. Springer.
