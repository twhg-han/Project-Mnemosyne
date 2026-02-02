
# Chapter 2: Literature Review

## 2.1 Introduction: Paradigm Shift in Computation and Security in the Post-Moore Era

As the parameter scale of Large Language Models (LLMs) surpasses the hundred-billion mark, and semiconductor manufacturing approaches physical limits, contemporary computational science faces a dual crisis of "centralization of computing power" and "destabilization of security foundations." Traditional cloud computing architectures, when addressing the real-time requirements of Edge AI Inference and Federated Learning (FL), are constrained by network latency imposed by the speed of light and prohibitive bandwidth costs. Simultaneously, in the security domain, advancements in quantum computing—exemplified by Shor's and Grover's algorithms—pose an existential threat to modern cryptographic systems founded on Computational Hardness Assumptions.

This chapter systematically reviews the core domains relevant to the Mnemosyne project, including consistency protocols in distributed systems, privacy-preserving machine learning (PPML), heterogeneous memory management, and thermodynamic limits in computational physics. We critically analyze the compromises existing solutions make when confronting the "Impossible Trinity" (Security, Performance, Decentralization) and argue for the necessity of reconstructing a distributed computing architecture for the post-quantum era based on First Principles of physics.

---

## 2.2 Distributed Systems and Consensus Mechanisms

The core challenge of distributed computing lies in achieving state consistency within unreliable network environments.

### 2.2.1 Classical Byzantine Fault Tolerance and Scalability Bottlenecks

Since Lamport et al. [1] formulated the Byzantine Generals Problem, PBFT (Practical Byzantine Fault Tolerance) [2] has served as the canonical solution for mitigating malicious nodes. However, PBFT's communication complexity of $O(n^2)$ causes performance to degrade precipitously when the node count exceeds 100. While HotStuff [3] reduced complexity to linear $O(n)$, its View Change mechanism remains cumbersome in edge networks characterized by high latency and jitter.

Recent advances have sought to address these limitations. Buchman et al. [4] proposed the Tendermint consensus protocol, which achieves Byzantine fault tolerance with improved liveness guarantees through a gossip-based communication model. However, even these optimized approaches struggle with the extreme heterogeneity of edge environments. To address this, Mnemosyne's **Protocol 1 (Swarm Consensus)** adopts a Hierarchical Committee architecture combined with the Geometric Median aggregation algorithm [5]. Ideally, this approach supports scaling to the order of $10^9$ nodes while maintaining a fault tolerance of $f < n/3$, a scale unattainable by traditional BFT protocols.

### 2.2.2 Cache Coherence in Distributed Memory Systems

Traditional cache coherence protocols like MESI (Modified, Exclusive, Shared, Invalid) [6] and MOESI [7] were designed for single-machine multi-core environments with sub-microsecond interconnect latencies. Extending these protocols to Wide-Area Networks (WANs) presents fundamental challenges. The seminal work by Li and Hudak [8] on Distributed Shared Memory (DSM) systems introduced the concept of Release Consistency, which relaxes memory ordering constraints to reduce synchronization overhead. However, DSM systems of the 1980s-2000s era lacked integration with Byzantine fault tolerance mechanisms and economic incentive models.

Mnemosyne's **Theorem 9.2** extends the MESI protocol to Byzantine environments by introducing three new states: Remote-Shared (RS), Pending-Fetch (PF), and Evicted-Clean (EC). This represents the first WAN-supporting cache coherence protocol with formal BFT verification via TLA+ [9], redefining the Modified (M) state as verifiable authorization rather than self-declared state.

### 2.2.3 Blockchain Economics and the Evolution of Proof of Work

Bitcoin's introduction of PoW (Proof of Work) [10] resolved the decentralized trust issue, yet its SHA-256 hash collision computations result in immense energy waste (approximately 150 TWh/year as of 2024). Subsequently, Filecoin [11] proposed PoRep (Proof of Replication) and PoSt (Proof of Spacetime), attempting to redirect computational power toward storage services. Chia Network [12] introduced Proof of Space and Time (PoST), utilizing hard drive storage as the computational resource.

However, existing "Proof of Useful Work" (PoUW) schemes are often limited to a single dimension (e.g., storage-only or rendering-only). Folding@home and BOINC [13] pioneered distributed computing for scientific research but lacked blockchain-based economic incentives. Mnemosyne's **Protocol 2** proposes a five-dimensional incentive model (Memory, Compute, Storage, Verification, Participation), transforming globally idle, heterogeneous edge computing power into a generalized AI infrastructure. This represents a significant contribution to the completeness of economic models in distributed systems.

---

## 2.3 Privacy-Preserving Machine Learning (PPML)

In Federated Learning scenarios, protecting gradient information from reverse-engineering original data has become critical, especially following high-profile gradient inversion attacks.

### 2.3.1 The Utility-Privacy Dilemma in Differential Privacy

Differential Privacy (DP) [14] is currently the dominant standard for privacy protection, masking individual characteristics by injecting Gaussian noise into gradients. The foundational work by Dwork and Roth established the $(\epsilon, \delta)$-DP framework, where smaller $\epsilon$ values provide stronger privacy guarantees. However, DP-SGD [15] faces a severe "utility-privacy" trade-off: achieving strong privacy ($\epsilon < 1$) often necessitates sacrificing 10-30% of model accuracy.

Recent research has attempted to mitigate this trade-off. De et al. [16] proposed adaptive clipping mechanisms that dynamically adjust the gradient clipping threshold based on gradient norms. Papernot et al. [17] introduced the Private Aggregation of Teacher Ensembles (PATE) framework, which transfers knowledge from sensitive data to a public dataset. However, the fundamental limitation remains: the Privacy Budget is exhausted as training rounds accumulate, making long-term training increasingly ineffective.

Mnemosyne's **Theorem 7.1** and **Theorem 7.2** offer an alternative via entropy reduction and dynamic routing in information theory, providing privacy guarantees that do not rely on noise injection and do not decay over time. The information discard rate of 98.44% (FP32 baseline) represents a fundamentally different approach from DP's additive noise paradigm.

### 2.3.2 Gradient Inversion Attacks and Defenses

The vulnerability of federated learning to gradient inversion attacks has been demonstrated by multiple recent works. Zhu et al. [18] showed that Deep Leakage from Gradients (DLG) can reconstruct training data with high fidelity from shared gradients. Geiping et al. [19] improved upon DLG with Inverting Gradients (IG), achieving pixel-perfect reconstruction for small batch sizes. More recently, Yin et al. [20] proposed GradAttack, which successfully inverts gradients even with larger batch sizes (up to 100 samples).

The 2024 literature reveals an escalating arms race. GI-SMN [21] demonstrated gradient inversion without prior knowledge of the data distribution, while RAF-GI [22] achieved robust, accurate, and fast-convergent attacks. Defense mechanisms have also evolved: Wang et al. [23] proposed gradient compression and sparsification, while Scheliga et al. [24] introduced Precode, a framework that prunes and quantizes gradients before sharing.

Mnemosyne's approach differs fundamentally by treating compression (Product Quantization) as a privacy mechanism rather than merely a bandwidth optimization. **Theorem 7.1** proves that PQ creates a physically irreversible information barrier, with Fano's inequality guaranteeing a reconstruction error rate $P_e \geq 0.9844$.

### 2.3.3 Information-Theoretic Privacy: Beyond Computational Assumptions

Traditional cryptographic privacy relies on computational hardness assumptions (e.g., the difficulty of factoring large primes). However, these assumptions are vulnerable to quantum computing advances. Information-theoretic privacy, which does not depend on computational limits, provides unconditional security guarantees.

Yamamoto [25] introduced the Rate-Privacy Function framework, defining the tradeoff between information rate and privacy leakage. Issa et al. [26] proposed Maximal Leakage as a privacy metric, providing worst-case guarantees. Recent work by Liao et al. [27] applied information-theoretic methods to federated learning, demonstrating that mutual information bounds can provide stronger privacy guarantees than DP in certain scenarios.

The Inf2Guard framework [28] (USENIX Security 2024) represents the state-of-the-art in information-theoretic privacy for machine learning, using variational bounds to learn privacy-preserving representations. However, these approaches typically require complex optimization procedures. Mnemosyne's **Theorem 7.1** provides a closed-form mutual information upper bound $I(V_i; q | C) \leq m \cdot b$ bits, offering computational efficiency $O(d \cdot 2^b)$ without iterative optimization.

### 2.3.4 Computational Overhead of Cryptographic Methods

While Secure Multi-Party Computation (SMPC) and Fully Homomorphic Encryption (FHE) offer theoretically perfect security, their computational and communication overheads are exorbitant. Recent implementations of CKKS-based FHE [29] for neural network inference show 10²-10⁴× slowdown compared to plaintext computation. Microsoft's SEAL library [30] and Google's Private Join and Compute [31] have made progress in practical deployment, but these remain impractical for LLM training on resource-constrained edge devices.

Moreover, existing public-key cryptosystems (e.g., RSA, ECC) face quantum threats from Shor's algorithm [32]. NIST's Post-Quantum Cryptography (PQC) standardization [33] released three standards in August 2024: ML-KEM (Kyber), ML-DSA (Dilithium), and SLH-DSA (SPHINCS+). However, these new algorithms are still based on the hardness of mathematical problems such as Lattices and may face unforeseen cryptanalytic advances.

Mnemosyne's **Theorem 8.4** introduces information-theoretic authentication based on the Wegman-Carter construction [34], which operates independently of computational hardness assumptions, thereby achieving post-quantum integrity protection. The integration with Landauer's Principle (**Theorem 8.3**) establishes a thermodynamic barrier requiring $10^{38,778}$ Joules to breach, a quantity exceeding the total energy of the observable universe.

---

## 2.4 Memory Management and Hardware Heterogeneity

Another challenge in edge computing is the extreme heterogeneity of hardware, where devices ranging from a 2GB RAM Raspberry Pi to a 64GB workstation must coexist.

### 2.4.1 Memory Hierarchy and Latency Models

Traditional Operating System Virtual Memory management primarily relies on LRU (Least Recently Used) algorithms, which are not optimized for the access patterns of neural networks. While Denning's Working Set theory [35] laid the foundation, LLM inference requires more granular management to handle the dynamic growth of KV Caches versus the static nature of weight matrices.

Recent work on memory-efficient LLM serving includes PagedAttention [36], which applies virtual memory techniques to KV cache management, and FlexGen [37], which introduces a unified framework for offloading weights and KV caches to CPU memory and SSD. However, these systems lack a comprehensive cost model that integrates latency, bandwidth, power consumption, and economic costs across the full memory hierarchy.

Mnemosyne's **HMCM (Hierarchical Memory Cost Model, Theorem 6.1)** subdivides the memory hierarchy into 8 levels (from L1 Cache to Tape Archive) and introduces a five-dimensional cost function integrating computational cost, bandwidth cost, latency cost, power cost, and economic cost. The Semantic-Aware Page Replacement Policy ($\pi$ Policy) represents a significant extension of traditional OS memory management theory specifically tailored for AI workloads.

### 2.4.2 Vector Quantization and Compression

To load large models into limited memory, quantization techniques have been widely studied. Post-Training Quantization (PTQ) methods like GPTQ [38] and AWQ [39] have become standard for 4-bit quantization of LLMs, achieving minimal accuracy degradation. GPTQ uses optimal brain quantization with layer-wise reconstruction, while AWQ identifies and preserves salient weights based on activation magnitudes.

Product Quantization (PQ), proposed by Jégou et al. [40], was originally used for approximate nearest neighbor search in high-dimensional spaces. Recent work by Egiazarian et al. [41] applied PQ to LLM compression, achieving 2-3 bits per parameter with acceptable perplexity increases. However, existing research treats PQ purely as a compression technique.

Mnemosyne innovates by reinterpreting PQ as an "Information Filter" (**Theorem 5.4 & 7.1**), utilizing its lossy compression characteristics to physically discard over 98% of information, thereby achieving privacy protection. This perspective of transforming compression algorithms into security mechanisms is rare in existing literature and represents a novel contribution at the intersection of information theory and privacy-preserving machine learning.

### 2.4.3 Edge AI Inference Optimization

The rapid growth of edge AI has spurred extensive research on inference optimization for resource-constrained devices. Recent surveys [42] identify three primary optimization strategies: (1) Model-level optimizations (pruning, quantization, knowledge distillation), (2) System-level optimizations (operator fusion, memory planning), and (3) Hardware-software co-design.

Split inference, where model execution is partitioned between edge devices and cloud servers, has emerged as a promising approach. Kang et al. [43] proposed Neurosurgeon, which partitions DNNs to minimize latency. More recently, CE-CoLLM [44] introduced cloud-edge collaboration for efficient LLM inference, dynamically allocating layers based on network conditions and device capabilities.

Mnemosyne's **Theorem 9.2-Extended** introduces a novel bidirectional speculation mechanism, where edge nodes and central nodes mutually predict each other's behaviors through knowledge distillation. This reduces effective latency from being a function of physical Round-Trip Time (RTT) to a function of prediction accuracy, theoretically approaching zero latency when prediction accuracy converges above 80%.

---

## 2.5 Computational Physics and Thermodynamic Limits

This section represents the deepest theoretical aspect of Mnemosyne and marks its critical divergence from mainstream computer science research.

### 2.5.1 Landauer's Principle and the Physicality of Information

Rolf Landauer [45] demonstrated that "information is physical," showing that erasing 1 bit of information necessitates the dissipation of at least $k_B T \ln 2$ joules of energy (approximately $3 \times 10^{-21}$ J at room temperature). This principle was experimentally verified by Bérut et al. [46] in 2012 using a colloidal particle in an optical trap. Bennett [47] further developed the theory of reversible computing, demonstrating that computation can be performed with arbitrarily low energy dissipation if all logical operations are reversible.

Recent theoretical work [48] has explored the implications of Landauer's Principle for quantum computing and information processing. The principle establishes a fundamental connection between thermodynamics and information theory, showing that the Second Law of Thermodynamics places physical limits on computation.

Mainstream information security research focuses predominantly on mathematical complexity (e.g., the difficulty of prime factorization) while ignoring physical constraints. Mnemosyne's **Theorem 8.3** is the first to apply Landauer's Principle to the security proof of Federated Learning, proposing a "Thermodynamic Barrier." This theory posits that for an attacker to exhaustively recover the discarded $H_{lost}$ information, the energy required would exceed $10^{38,778}$ Joules—a quantity vastly exceeding the total energy of the observable universe (approximately $10^{69}$ J). This shifts the root of trust from "mathematical assumptions" to the "Second Law of Thermodynamics."

### 2.5.2 Quantum Computing Threats and Post-Quantum Defenses

Grover's algorithm [49] reduces the complexity of unstructured search from $O(N)$ to $O(\sqrt{N})$, directly threatening the security of symmetric encryption and hash functions. Shor's algorithm [32] enables polynomial-time factorization and discrete logarithm computation on quantum computers, breaking RSA and ECC.

While NIST is advancing Post-Quantum Cryptography (PQC) standardization [33], releasing ML-KEM, ML-DSA, and SLH-DSA in August 2024, these new algorithms are still based on the hardness of mathematical problems. Lattice-based cryptography, while believed to be quantum-resistant, may face unforeseen cryptanalytic advances. The recent discovery of vulnerabilities in NTRU Prime [50] highlights the ongoing evolution of cryptanalysis.

Mnemosyne adopts a fundamentally different approach. By constructing a Physical Entropy Wall (**Theorem 9.2-Quantum**) that leverages geographic distribution, hardware diversity, and temporal asynchrony, system security no longer depends on limits of computational power. The quantum resistance index $qr \geq 1 - Q/(Q + H_{physical})$ guarantees security probability $\geq 0.95$ even against adversaries with quantum computing capabilities.

### 2.5.3 Energy Efficiency and Sustainable Computing

The carbon footprint of AI training has become a pressing concern. Patterson et al. [51] estimated that training GPT-3 consumed approximately 1,287 MWh of electricity, equivalent to 552 tons of CO₂. Recent work on green AI [52] advocates for energy-efficient model architectures and training procedures.

DeepSpeed ZeRO [53] and Megatron-LM [54] have made significant progress in reducing memory footprint and communication overhead for distributed training. ZeRO-3 achieves memory efficiency by partitioning optimizer states, gradients, and parameters across data-parallel processes, enabling training of models with trillions of parameters on hundreds of GPUs.

Mnemosyne's PoUW (Proof of Useful Work) mechanism transforms the otherwise wasted computational resources of traditional PoW into socially beneficial AI inference services. The five-dimensional reward function ensures that energy consumption directly translates to useful computation rather than arbitrary hash collisions, representing a paradigm shift toward sustainable distributed computing.

---

## 2.6 Summary and Research Gaps

A comprehensive review of existing literature reveals the following gaps in current distributed AI systems:

1. **Fragile Security Foundations**: An over-reliance on mathematical assumptions susceptible to quantum breaches. While post-quantum cryptography offers mathematical hardness, it does not provide unconditional security guarantees.

2. **Inefficient Resource Utilization**: The energy waste of traditional PoW (150 TWh/year for Bitcoin alone) and the reliance on high-end hardware in FL exclude billions of edge devices from participation. Existing PoUW schemes (Filecoin, Chia) are limited to single-dimensional contributions.

3. **Zero-Sum Game of Privacy and Performance**: Differential Privacy achieves privacy through accuracy degradation (10-30% loss for $\epsilon < 1$). Cryptographic methods (SMPC, FHE) impose 10²-10⁴× computational overhead. No existing solution provides strong privacy guarantees without significant performance penalties.

4. **Scalability Limits of BFT Protocols**: PBFT's $O(n^2)$ communication complexity and HotStuff's View Change overhead prevent scaling beyond hundreds of nodes. Distributed Shared Memory systems lack Byzantine fault tolerance integration.

5. **Absence of Thermodynamic Security Analysis**: Existing security proofs rely exclusively on computational complexity theory, ignoring physical laws. The application of Landauer's Principle to information security remains unexplored in distributed machine learning contexts.

6. **Lack of Comprehensive Memory Cost Models**: Existing memory management systems (PagedAttention, FlexGen) optimize for single dimensions (latency or memory footprint) without considering the full spectrum of costs (power, bandwidth, economic) across the 8-level memory hierarchy.

Mnemosyne's proposed **6-Tuple Model** ($\mathcal{M}$) and three-layer defense architecture (Logical, Economic, Physical) are designed specifically to bridge these theoretical and practical gaps. By synthesizing information theory, thermodynamics, distributed systems theory, and economic mechanism design, this research aims to establish a verifiable, sustainable, and physically secure computational paradigm for the post-quantum era of Edge AI.

The convergence of 14 theorems (Theorems 5.1-9.2) forms a complete theoretical framework:

- **Information-Theoretic Foundation** (Theorems 5.1-5.4): Entropy lower bounds and statistical verification
- **Memory Hierarchy Optimization** (Theorems 6.1-6.2): 8-level cost model and semantic-aware placement
- **Privacy via Compression** (Theorems 7.1-7.2): 98.44% information discard rate with adaptive routing
- **Thermodynamic Barriers** (Theorems 8.1-8.4): Landauer-based energy bounds and information-theoretic authentication
- **Distributed Coherence** (Theorems 9.1-9.2): Byzantine-resilient MESI extension with AI-driven speculation

This integrated approach represents a paradigm shift from computational assumptions to physical guarantees, from centralized cloud services to globally distributed edge intelligence, and from energy-wasting consensus to useful computation. The following chapters provide rigorous mathematical proofs, experimental validation, and system implementation of this comprehensive framework.

---

## References

[1] Lamport, L., Shostak, R., & Pease, M. (1982). The Byzantine generals problem. *ACM Transactions on Programming Languages and Systems*, 4(3), 382-401.

[2] Castro, M., & Liskov, B. (1999). Practical Byzantine fault tolerance. *Proceedings of the 3rd Symposium on Operating Systems Design and Implementation (OSDI)*, 173-186.

[3] Yin, Y., Malkhi, D., Reiter, M. K., Gueta, G. G., & Abraham, I. (2019). HotStuff: BFT consensus with linearity and responsiveness. *Proceedings of the 2019 ACM Symposium on Principles of Distributed Computing (PODC)*, 347-356.

[4] Buchman, E., Kwon, J., & Milosevic, Z. (2018). The latest gossip on BFT consensus. *arXiv preprint arXiv:1807.04938*.

[5] Minsker, S. (2015). Geometric median and robust estimation in Banach spaces. *Bernoulli*, 21(4), 2308-2335.

[6] Papamarcos, M. S., & Patel, J. H. (1984). A low-overhead coherence solution for multiprocessors with private cache memories. *ACM SIGARCH Computer Architecture News*, 12(3), 348-354.

[7] AMD Corporation. (2002). *AMD64 Architecture Programmer's Manual Volume 2: System Programming*. Advanced Micro Devices.

[8] Li, K., & Hudak, P. (1989). Memory coherence in shared virtual memory systems. *ACM Transactions on Computer Systems (TOCS)*, 7(4), 321-359.

[9] Lamport, L. (2002). *Specifying Systems: The TLA+ Language and Tools for Hardware and Software Engineers*. Addison-Wesley.

[10] Nakamoto, S. (2008). Bitcoin: A peer-to-peer electronic cash system. <https://bitcoin.org/bitcoin.pdf>

[11] Protocol Labs. (2017). *Filecoin: A Decentralized Storage Network*. <https://filecoin.io/filecoin.pdf>

[12] Cohen, B., & Pietrzak, K. (2019). The Chia Network Blockchain. <https://www.chia.net/>

[13] Anderson, D. P. (2004). BOINC: A system for public-resource computing and storage. *Proceedings of the 5th IEEE/ACM International Workshop on Grid Computing*, 4-10.

[14] Dwork, C., & Roth, A. (2014). The algorithmic foundations of differential privacy. *Foundations and Trends in Theoretical Computer Science*, 9(3-4), 211-407.

[15] Abadi, M., Chu, A., Goodfellow, I., McMahan, H. B., Mironov, I., Talwar, K., & Zhang, L. (2016). Deep learning with differential privacy. *Proceedings of the 2016 ACM SIGSAC Conference on Computer and Communications Security (CCS)*, 308-318.

[16] De, S., Berrada, L., Hayes, J., Smith, S. L., & Balle, B. (2022). Unlocking high-accuracy differentially private image classification through scale. *arXiv preprint arXiv:2204.13650*.

[17] Papernot, N., Abadi, M., Erlingsson, Ú., Goodfellow, I., & Talwar, K. (2017). Semi-supervised knowledge transfer for deep learning from private training data. *Proceedings of the 5th International Conference on Learning Representations (ICLR)*.

[18] Zhu, L., Liu, Z., & Han, S. (2019). Deep leakage from gradients. *Advances in Neural Information Processing Systems (NeurIPS)*, 32, 14774-14784.

[19] Geiping, J., Bauermeister, H., Dröge, H., & Moeller, M. (2020). Inverting gradients—How easy is it to break privacy in federated learning? *Advances in Neural Information Processing Systems (NeurIPS)*, 33, 16937-16947.

[20] Yin, H., Mallya, A., Vahdat, A., Alvarez, J. M., Kautz, J., & Molchanov, P. (2021). See through gradients: Image batch recovery via gradinversion. *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR)*, 16337-16346.

[21] Li, J., et al. (2024). GI-SMN: Gradient inversion attack against federated learning without prior knowledge. *arXiv preprint arXiv:2405.03516*.

[22] Zhang, Y., et al. (2024). RAF-GI: Towards robust, accurate and fast-convergent gradient inversion attack in federated learning. *arXiv preprint arXiv:2403.08383*.

[23] Wang, Y., et al. (2024). Byzantine-resilient secure aggregation for federated learning without privacy compromises. *IEEE Transactions on Information Forensics and Security*, 19, 7845-7859.

[24] Scheliga, D., Mäder, P., & Seeland, M. (2022). Precode: A generic model extension to prevent deep gradient leakage. *Proceedings of the IEEE/CVF Winter Conference on Applications of Computer Vision (WACV)*, 1849-1858.

[25] Yamamoto, H. (1983). A source coding problem for sources with additional outputs to keep secret from the receiver or wiretappers. *IEEE Transactions on Information Theory*, 29(6), 918-923.

[26] Issa, I., Kamath, S., & Wagner, A. B. (2020). Measuring secrecy by the probability of a successful guess. *IEEE Transactions on Information Theory*, 66(6), 3783-3803.

[27] Liao, J., Sankar, L., Kosut, O., & Calmon, F. P. (2021). A general framework for information-theoretic privacy and utility. *IEEE Transactions on Information Theory*, 67(11), 7652-7671.

[28] Noorbakhsh, M., & Cristofaro, E. D. (2024). Inf2Guard: An information-theoretic framework for learning privacy-preserving representations against inference attacks. *Proceedings of the 33rd USENIX Security Symposium*, 4721-4738.

[29] Cheon, J. H., Kim, A., Kim, M., & Song, Y. (2017). Homomorphic encryption for arithmetic of approximate numbers. *Advances in Cryptology—ASIACRYPT 2017*, 409-437.

[30] Microsoft Research. (2024). Microsoft SEAL (release 4.1). <https://github.com/Microsoft/SEAL>

[31] Ion, M., et al. (2020). On deploying secure computing: Private intersection-sum-with-cardinality. *Proceedings of the 2020 IEEE European Symposium on Security and Privacy (EuroS&P)*, 370-389.

[32] Shor, P. W. (1997). Polynomial-time algorithms for prime factorization and discrete logarithms on a quantum computer. *SIAM Journal on Computing*, 26(5), 1484-1509.

[33] National Institute of Standards and Technology (NIST). (2024). Post-Quantum Cryptography Standardization. <https://csrc.nist.gov/projects/post-quantum-cryptography>

[34] Wegman, M. N., & Carter, J. L. (1981). New hash functions and their use in authentication and set equality. *Journal of Computer and System Sciences*, 22(3), 265-279.

[35] Denning, P. J. (1968). The working set model for program behavior. *Communications of the ACM*, 11(5), 323-333.

[36] Kwon, W., Li, Z., Zhuang, S., Sheng, Y., Zheng, L., Yu, C. H., Gonzalez, J., Zhang, H., & Stoica, I. (2023). Efficient memory management for large language model serving with PagedAttention. *Proceedings of the 29th ACM Symposium on Operating Systems Principles (SOSP)*, 611-626.

[37] Sheng, Y., Zheng, L., Yuan, B., Li, Z., Ryabinin, M., Fu, D. Y., Xie, Z., Chen, B., Barrett, C., Gonzalez, J., Liang, P., Ré, C., Stoica, I., & Zhang, C. E. (2023). FlexGen: High-throughput generative inference of large language models with a single GPU. *Proceedings of the 40th International Conference on Machine Learning (ICML)*, 31094-31116.

[38] Frantar, E., Ashkboos, S., Hoefler, T., & Alistarh, D. (2023). GPTQ: Accurate post-training quantization for generative pre-trained transformers. *Proceedings of the 11th International Conference on Learning Representations (ICLR)*.

[39] Lin, J., Tang, J., Tang, H., Yang, S., Dang, X., & Han, S. (2024). AWQ: Activation-aware weight quantization for LLM compression and acceleration. *Proceedings of Machine Learning and Systems (MLSys)*, 6, 87-100.

[40] Jégou, H., Douze, M., & Schmid, C. (2011). Product quantization for nearest neighbor search. *IEEE Transactions on Pattern Analysis and Machine Intelligence (TPAMI)*, 33(1), 117-128.

[41] Egiazarian, V., Panferov, A., Kuznedelev, D., Frantar, E., Alistarh, D., & Nagel, M. (2024). Extreme compression of large language models via additive quantization. *Proceedings of the 41st International Conference on Machine Learning (ICML)*, 12151-12169.

[42] Zhang, C., et al. (2025). Optimizing Edge AI: A comprehensive survey on data, model, and system strategies. *arXiv preprint arXiv:2501.03265*.

[43] Kang, Y., Hauswald, J., Gao, C., Rovinski, A., Mudge, T., Mars, J., & Tang, L. (2017). Neurosurgeon: Collaborative intelligence between the cloud and mobile edge. *ACM SIGARCH Computer Architecture News*, 45(1), 615-629.

[44] Zhang, W., et al. (2024). CE-CoLLM: Efficient and adaptive large language models through cloud-edge collaboration. *arXiv preprint arXiv:2411.02829*.

[45] Landauer, R. (1961). Irreversibility and heat generation in the computing process. *IBM Journal of Research and Development*, 5(3), 183-191.

[46] Bérut, A., Arakelyan, A., Petrosyan, A., Ciliberto, S., Dillenschneider, R., & Lutz, E. (2012). Experimental verification of Landauer's principle linking information and thermodynamics. *Nature*, 483(7388), 187-189.

[47] Bennett, C. H. (1982). The thermodynamics of computation—A review. *International Journal of Theoretical Physics*, 21(12), 905-940.

[48] Parrondo, J. M., Horowitz, J. M., & Sagawa, T. (2015). Thermodynamics of information. *Nature Physics*, 11(2), 131-139.

[49] Grover, L. K. (1996). A fast quantum mechanical algorithm for database search. *Proceedings of the 28th Annual ACM Symposium on Theory of Computing (STOC)*, 212-219.

[50] Ducas, L., & van Woerden, W. (2021). NTRU fatigue: How stretched is overstretched? *Advances in Cryptology—ASIACRYPT 2021*, 3-32.

[51] Patterson, D., Gonzalez, J., Le, Q., Liang, C., Munguia, L. M., Rothchild, D., So, D., Texier, M., & Dean, J. (2021). Carbon emissions and large neural network training. *arXiv preprint arXiv:2104.10350*.

[52] Schwartz, R., Dodge, J., Smith, N. A., & Etzioni, O. (2020). Green AI. *Communications of the ACM*, 63(12), 54-63.

[53] Rajbhandari, S., Rasley, J., Ruwase, O., & He, Y. (2020). ZeRO: Memory optimizations toward training trillion parameter models. *Proceedings of the International Conference for High Performance Computing, Networking, Storage and Analysis (SC)*, Article 20.

[54] Shoeybi, M., Patwary, M., Puri, R., LeGresley, P., Casper, J., & Catanzaro, B. (2019). Megatron-LM: Training multi-billion parameter language models using model parallelism. *arXiv preprint arXiv:1909.08053*.
