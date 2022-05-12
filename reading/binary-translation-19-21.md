# reading-papers-19-21

### 2019 Runtime performance evaluation and optimization of type-2 hypervisor for MIPS64 architecture

- Journal of King Saud University – Computer and Information Sciences
- 一个软件实现的 type-2 hypervisor 用于 embedded system

> However, there are few open-source virtualization solutions available for embedded systems.

- HTTM 是一个在 MIPS6 host 的 Linux 系统上支持 MIPS64 Guest 的虚拟机

> HTTM is a MIPS6 based type-2 hypervisor running on a Linux based host system providing full system virtualization.

- 多线程：每个 core/device 都使用一个单独的线程实现

> HTTM is a multithreaded application. Each virtual core/device implementation creates a separate thread. Cores, UART, Central Interrupt Unit (CIU), Virtio, and network devices are maintained as separate threads on the host system.

- 执行流程：Block Fetch, Instruction Translation, Block Execution

> The execution cycle can simply be described in three steps, which are performed by each core.
>
> 1. Block Fetching: A set of instructions starting with current guest PC and ending with control shifting instruction (called a block) is fetched from the guest code.
> 2. Instruction Translation: The fetched block is converted into the translated block, by translating each instruction one by one. These translated assembly instructions are executed on the real hardware on behalf of the guest.
> 3. Block Execution: The current host PC is then shifted to the translated block, which then gets directly executed on the host.

- 优化：上下文切换

> 4.2 Proposed optimization - reduction in context switching
>
> HTTM keeps different types of caches. Mainly there are three of them. 1) Block Cache: contain the translatied block correspongding to the starting PC of the block, 2) Address Translation  Cache: contain the tarnslated address, i.e. Host's virtual address (HVA) corresponding to the Guest's virtual address (GVA) and 3) I/O address cache: Translated guest I/O addresses are kept in a separate cache.
>
> Context switching is avoided by executing some handwritten assembly instructions which check and retrievee the content from the cache.

- 指令集虚拟化：动态二进制翻译器

> HTTM uses Dynamic Binary Translation (DBT) based technique for ISA virtualization

- 优化动态二进制翻译器
  - 单条指令翻译 => 基本块翻译，减少访存指令

> As pointed out by the block analysis, it’s intuitive that by changing the scope of DBT from one instruction to the whole block will help in optimizing the overall performance of the hypervisor. The main idea is to reduce the redundant load/store instructions in the generated translated block.



### 2019 x86-64 Instruction Usage among C/C++ Applications [ACM SYSTOR]

- 详细分析了 x86_64 指令的使用情况

> This paper presents a study of x86-64 instruction usage across 9,337 C/C++ applications and libraries in the Ubuntu 16.04 GNU/Linux distribution



### :star: :star: 2019 cross-ISA Machine Instrumentation using Fast and Scalable Dynamic Binary Translation [VEE]

- Name: Qelt

- 优化 cross-ISA 的 DBT 性能

> In this paper we improve cross-ISA emulation and instrumentation performance through three novel techniques.

- 1.  浮点操作模拟

  - 根据观察，仅加速常见情况下的 FP 指令，覆盖 SPECFP06 的 99.18%

> First, we increase floating point (FP) emulation performance by observing that most FP operations can be correctly emulated by surrounding the use of the host FP unit with a minimal amount of non-FP code.
>
> Our approach is thus to leverage the host FPU but only for a subset of all possible FP operations
>
> We thus accelerate the common case, i.e.: the rounding is round-to-nearest-even, inexact is already set, and the operands are checked to limit the flags that the operation can raise. Otherwise, we defer to a soft-float implementation.
>
> Profiling shows that on average Qelt accelerates (i.e., does not defer to softfloat) 99.18% of FP instructions in SPECfp06 benchmarks.

- 2. 共享 Code Cache 来支持多核 Guest 的模拟

  - 基于 QEMU 的 Base Line 修改，原本为全局锁来保护，修改使得每个 vCPU 可以更快地运行
  -  **Partitioned TB cache** and **per-region binary search tree** 

> Second, we introduce the design of a translator with a shared code cache that scales for multi-core guests, even when they generate translated code in parallel at a high rate.
>
> In Qelt, we modify this baseline design, in which a single lock protects both code translation/invalidation as well as code chaining, to make each vCPU thread work—in the fast path—on uncontended cache lines.
>
> We keep the baseline's single, contiguous (in virtual memory) buffer for the code cahe. However, we partition this buffer so that each vCPU generates code into a separate region.
>
> To serve these lookups we maintain a per-region binary search tree that tracks the beginning and end host addresses of the region’s TBs.
>
> Descriptors of guest memory pagesare kept in the memory map, which is implemented as a radix tree. We modify this tree to support lock-free insertions, and rely on the fact that the tree already supports RCU for lookups

- 3. 提出了一个 ISA 无关的代码插桩层

> Third, we present an ISA-agnostic instrumentation layer that can instrument guest operations that occur outside of the DBT’s intermediate representation (IR), which are common in full-system emulators.

- 其他优化：SoftMMU 和 indirect branches cache in QEMU (IBTC)

> Cross-ISA TLB Emulation. Guest TLB emulation in crossISA full-system emulators is a large contributor to performance overhead. The “softMMU”, as it is called in QEMU [6], is a table kept in software that maps guest to host virtual addresses.
>
> Indirect branches in DBT. 
>
> An approach better suited for full-system emulators is the use of caching [44], which is demonstrated on QEMU by Hong et al. [26]. They add a small cache to each vCPU thread to track cross-page and indirect branch targets, and then modify the target code to inline cache lookups and avoid most costly exits to the dispatcher.



### :star: :star: 2019 Scalable Emulation of Heterogeneous Systems [PhD]

- Pico: Cross-ISA Machine Emulation
  - 共享 code cache 的设计
  - 原子指令的正确实现

> The two main contributions of this chapter are:
>
> - A memory-efficient design of a shared code cache for DBT engines.
> - A scalable, fully correct cross-ISA approach to emulating atomic instructions and recondciling guest-host differences in meory consistency models.

- Qelt: Cross-ISA Machine Instrumentation
  - 利用 host FPU 来支持 cross-ISA DBT 的浮点指令
  - 并行 DBT 引擎的设计：共享 code cache、TB index、等等
  - 基于一个基于 QEMU 的 Qelt 的一个 DBI 的工具

> In this chapter we make the following contributions:
>
> - We describe a technique to leverage the host FPU when performing cross-ISA DBT to achieve high emulation performance
> - We present the design of a parallel cross-ISA DBT engine
> - We present an ISA-gnostic instrumentation layer that converts a cross-ISA DBT engine into a low-overhead cross-ISA DBI tool

- Accelerator Coupling in Heterogeneous Architectures

> we present a study that compares different accelerator coupling models
>
> The study is enabled by Cargo. a machine simulator that we developed to model systems that integrate accelerators
>
> Our experiments on these accelerators induce the following observations
>
> - Private local memories (PLMs) are key to performance the energy efficiency
> - DRAM bandwith can quickly become a bottleneck
> - Cache pollution plays a minor role when comparing LLC-DMA vs. DRAM-DMA loosely-coupled accelerators
> - The run-time overhead of abstracting loosely-coupled accelerators as just SoC-like devices becomes negligible once the granularity of the acceleration becomes non-trivial.

- ROCA: Reducing the Opportunity Cost of Accelerator Integration

> we present a novel technique to expand the last-level cache of the heterogeneous system by reuing memory from otherwise unused on-chip accelerators



### 2019 SoK: Using Dynamic Binary Instrumentation for Security [AsiaCCS'19]

- 介绍了 DBI frameworks 以及 DBI 工具可能存在的两个安全问题：evade 和 escape

> A code is said to **evade** DBI-based analysis when its instruction and data flows eventually start diverging from those that would accompany it in a native execution.
>
> A code is said to **escape** DBI-based analysis when parts of its instruction and data flows get out of the DBI engine’s control and execute natively on the CPU, in the address space of the analysis process or of a different one



### :star: 2019 Optimizing data permutations in structured loads/stores translation and SIMD register maping for a cross-ISA dynamic binary translator [JSA]

Hsu

- 针对特殊的 SIMD 指令的翻译进行优化，使用 host 的 SIMD 进行加速

> More and more modern processors have been supporting non-contiguous SIMD data accesses. However, translating such instructions has been overlooked in the Dynamic Binary Translation (DBT) area.
>
> Structured loads/stores, such as VLDn/VSTn in ARM NEON, are one type of strided SIMD data access instructions.

- 设计了一种寄存器映射算法，来使得翻译的结果更优，不仅考虑到寄存器映射，还有本地硬件实现的特性

> The goal of SIMD register mapping is to determine a conﬁguration which maps guest registers to host registers and minimizes data reorganization overhead.
>
> To simplify SIMD hardware design, microprocessor vendors usually provide eﬃcient data reorganization among a few speciﬁc SIMD lanes. Data movement among the other SIMD lanes usually requires multiple hops of data reorganization, thus incurring higher cost.Hence, the register mapper should consider not only the usage of the guest SIMD registers, but also the required data reorganization cost.

- 提出了一个 Guest 访存融合的技术：GuestMemoryFusion

> GuestMemoryFusion is implemented as a LLVM optimization pass.
>
> It tries to fuse multiple load/store instructions which write/read data to/from consecutive sub-registers and guarantees no possible memory alias
>
> It operates on basic blocks.



### 2019 Non-Intrusive Hardware Acceleration for Dynamic Binary Translation in Embedded Systems [IEEE ICIT]
- 一种嵌入式系统中非侵入式的 DBT 硬件加速方案：DBTOR

> This article describes a non-intrusive hardware acceleration approach for Dynamic Binary Translation (DBT) in modern resource-constrained embedded systems, detailing its motivation, design decisions and overall architecture.
>
> This article explains the functionality extension of the DBTOR engine using external hardware support integrated as a bus sniffer, the HWSniffer, which results in a hardware-based and non-intrusive execution ﬂow technique never tried before in the state of the art.

- 似乎是一种可以监控访存的硬件手段？

> The address and access signals are monitored through the HWSniffer logic, in order to identify accesses to sensitive memory addresses and trigger the respective actions, when needed.
>
> The bus HWSniffer can be used to **trigger software or hardware components based on the source architecture memory accesses**, providing great ﬂexibility on the type of application to serve without disturbing the base DBT program ﬂow.

- 三种应用

> Three types of application to the proposed bus sniffer were presented: CC ﬂags handling, source peripherals remapping and interrupt support.

- 写得真烂，看不下去



### 2019 DBTOR: A Dynamic Binary Translation Architecture for Modern Embedded Systems [IEEE ICIT]
- 资源受限的嵌入式系统中的 DBT

> This article describes a dynamic binary translation (DBT) system specially tailored to ﬁt resource-constrained embedded systems, detailing its design decisions and architectural components.

- 结构也是一个基本的 DBT 框架



### :star: 2019 Enhancing Transactional Memory Execution via Dynamic Binary Translation [ACM SIGAPP]

Hsu

- Transactional Synchronization Extensions (TSX) 内存的两个机制
  - Hardware Lock Elision (HLE)
  - Restricted Transactional Memory (RTM)

> Transactional Synchronization Extensions (TSX) have been introduced for hardware transactional memory since the 4th generation Intel Core processors. TSX provides two software programming interfaces: Hardware Lock Elision (HLE) and Restricted Transactional Memory (RTM).
>
> HLE is easy to use and maintains backward compatibility with processors without TSX support, while RTM is more flexible and scalable.

- 很多程序用 HLE 写的，而改用 RTM 性能会更好，因此利用 DBT 来加速

> Previous researches have shown that critical sections protected by RTM with a well-designed retry mechanism as its fallback code path can often achieve better performance than HLE.
>
> More parallel programs may be programmed in HLE, however, using RTM may obtain greater performance.
>
> we present a framework built on QEMU that can dynamically transform HLE instructions in an application binary to fragments of RTM codes with adaptive tuning on the fly.
>
> When the guest and host ISAs are the same, e.g., the guest is x86 + HLE and the host is x86 + RTM in our case, DBT is used for dynamic binary optimization.
>
> Compared to HLE execution, our prototype achieves 1.56× speedup with **8 threads** on average.

- HLE `xacqiore` and `xrelease` 
- RTM `xbegin` `xend` and `xabort` 



### :star: :star: 2019 Processor-Tracing Guided Region Formation in Dynamic Binary Translation 

Hsu

- 利用硬件的分支历史信息，为 trace 的收集提供硬件支持

> We leverage the branch history information stored in the processor to reconstruct the program execution profile and effectively form high-quality regions with low cost.
>
> Furthermore, we present the designs of lightweight hardware performance monitoring sampling and the branch instruction decode cache to minimize region formation overhead

- 硬件分支历史信息包含分支指令的压缩形式，因此使用分支指令译码缓存，减少软件译码的开销

> we first provide an example that demonstrates the process of software decoding and then present the proposed branch instruction decode cache, which can effectively reduce the overhead of software decoding.



### 2019 Exploiting Vector Processing in Dynamic Binary Translation [ICPP]

Hsu

- 旧软件用不到新的 SIMD 特性，使用 DBT 来将 loop 转换为 host 的向量指令

> However, since processor architectures have kept enhancing with new features to improve vector/SIMD performance, legacy application binaries failed to fully exploit new vector/SIMD capabilities in modern architectures.
>
>  we study the fundamental issues involved in cross-ISA Dynamic Binary Translation (DBT) to convert non-vectorized loops to vector/SIMD forms to achieve greater computation throughput available in newer processor architectures

- Register Spilling 寄存器溢出：标量数据写回内存，以便空出寄存器

> Scalar variables in binaries may not always be in the form of **scalars**, but in the form of memory references because of register spilling
>
> Furthermore, it is also possible to spill the arbitrary variables from register to memory such as array base address and constant stride in binaries.

- Virtual Register Promotion：把 spilled 的变量放在虚拟寄存器中，便于向量化的 loop 分析

> We adopt vritual register promotion, which promotes spilled variables to virtual register, to simplify loop analysis and enable vectorizations.
>
> There are two types of spilled variables
>
> - Program Counter (PC) relative data
> - Stack variable

- 避免 memory aliasing：Runtime Check for Virtual Register Promotion



### 2019 Exploiting SIMD Asymmetry in ARM-to-x86 Dynamic Binary Translation [ACM Transactions on Architecture and Code Optimization]

Hsu

- (大致同上篇) 即 Guest SIMD 与 Hoset SIMD 的能力不对等，通常 Host 更大更强

> In this article, we present a novel binary translation technique called spill-aware superword level parallelism (saSLP)

- 代价模型来评估向量融合是否有效果

> We propose a novel spill-aware cost model that can determine whether combining guest registers would be profitable by weighing the extra unpacking overheads and the benefits of conserving host registers.

- 利用 SIMD gather 指令来将非规则的 loop 进行向量化

> We present the vectorization of data-irregular loops by exploiting SIMD gather instructions and propose a new algorithm to vectorize loop reductions in DBTs, where the typical methods used in static compilers do not work.



### 2019 Improving Startup Performance in Dynamic Binary Translators [PDP]

- 多线程翻译，减少 program translation 的开销，提高启动性能

> We propose and assess the potential of a new technique that parallelizes program translations on multi-core machines to reduce its evident run-time costs

- 基于 DynamoRIO DBT
- 提前发现一些即将执行但是还未执行的 guest code 来进行并行翻译

> our ﬁrst challenge to parallelize block translations is to ﬁnd guest binary code blocks before they are reached during execution.
>
> We call this speculative translation of blocks as eager translation.

- 离线翻译方案：先运行一次 test 产生 block list，再次运行时在初始化阶段将其翻译好

> First, we run each benchmark program (with the test input data-set) with a DynamoRIO conﬁguration that outputs the list of block addresses that are reached during execution.
>
> The later measured runs employ a DynamoRIO setup that uses this list of block addresses to populate the compiler queue during initialization.



### 2019 Multi-objective Exploration for Practical Optimization Decisions in Binary Translation

- 使用机器学习来建立可行的优化决策方案（翻译成中文怎么就这么别扭）

> we investigate an opportunity to build practical optimization decision models for DBT by using machine learning techniques.
>
> we identify **the best classification algorithm** for our infrastructure with consideration for both prediction accuracy and cost
>
> 74.5% of prediction accuracy for the optimal unroll factor and realizes an average 20.9% reduction in dynamic instruction count during the steady-state translated code execution

- 以 loop unrolling 为例使用 ML 来进行决策， 选择 unroll 的最佳参数

> We show how machine learning techniques can improve loop unrolling decisions for dynamic binary translation on the mobile processor.



### 2019 Hybrid-DBT: Hardware/Software Dynamic Binary Translation Targeting VLIW
- 开源的基于 VLIW 的硬件加速支持的 DBT

> In this paper, we present Hybrid-DBT, an open-source, hardware accelerated DBT system targeting VLIW cores.
>
> Three different hardware accelerators have been designed to speed-up critical steps of the translation process

- 软硬件结合的设计，将 RISC-V 翻译到 VLIW

> Hybrid-DBT system is a hardware software co-designed DBT framework. It translates RISC-V binaries into VLIW binaries and uses dedicated hardware accelerators to reduce the cost of certain steps of the translation/optimization process.

- 硬件加速器1：First-Pass Translator

> In the system, the accelerator called First-Pass Translator is used in the first optimization level to perform a first translation that needs to be as cheap as possible.

- 硬件加速器2,3：IR Generator and IR Scheduler

> The IR Generator is then used at the next optimization level to generate a higher level representation of the translated binaries, which is used by the IR Scheduler to generate VLIW binaries by performing instruction scheduling and register allocation.

- 三层的翻译结构

> Hybrid-DBT is organized as a three-step translation and optimization flow.
>
> 1) **Translation of instructions** is the optimization level 0. When executing cold-code, the DBT framework tries to spend as little time/energy as possible to translate binaries. In the framework, we first perform a naive translation of each source instruction into one or several instructions from the VLIW ISA. This optimization step does not try to exploit ILP at all.
> 2) **Block building and scheduling** is the optimization level 1. When a block has more than a certain number of instructions, it is considered as a candidate for optimization level 1: an IR of the block is built and used for performing instruction scheduling. This optimization level is triggered aggressively, without any profiling information because of the use of hardware accelerators that reduces its cost. Scheduled blocks are then profiled to trigger further optimizations.
> 3) **Interblock optimization** are performed at optimization level 2. When a given block has been executed enough, it triggers the interblock optimization. This optimization level analyzes the control flow (place and destination of each jump instruction) to build a control-flow graph of the function (i.e., all blocks that are accessed through direct branches except calls). Then interblock optimizations are performed on this function to extract more ILP.



### 2019 Implementation of a binary translation for improving ECU performance

3 pages ......

- 看不懂：ECU？IVN？



### 2019 Dynamic Translation Optimization Method Based on Static Pre-Translation [IEEE Access]

数学工程与先进计算国家重点实验室，郑州

- 针对 DBT ，使用静态翻译来做优化。基于 Sunway 处理器（神威）

> An optimization method of dynamic binary translation with static pre-translation was proposed in this paper. By pre-translating the source program and using more in-depth code optimization method to optimize the translated code, most of the code translation time can be cut down and the translated code will be more efficient.

- 静态提前翻译的两个问题：静态获取代码 and 动态查询静态的翻译结果

> There are two key problems need to be solved to realize the mechanism of static pre-translation.
>
> The first is the availability of static translation code. It needs the code obtained by static translator to have the same function of the code obtained by dynamic translation.
>
> The second is to achieve the use of static translation results by perfecting the query and translation module. Since not all code can be translated by static translator, the query and execution process need to distinguish weather the source code has been pre-translated and not.

- 使用相同的翻译模式和存储方式，使得 static 和 dynamic 的 translation result 一致

> The result of the transformation is independent of the translation method—no matter it’s dynamic or static binary translation, When the matching pattern of instruction mapping and the state mapping are the same.

- 基于地址信息的查询

> The correctness of the instruction transformation has been ensured. So just execute the results of both dynamic and static translation with the order maintained.
>
> The instruction’s loading address can be used as the unique identification of the instruction, as the value in the instruction pointer register is the loading address of the following instruction in a determinate execution process. 说白了就是 PC
>
> The sequence of source instructions for static translation is usually in code segment that are loaded in read-only mode, so it does not take the self-modifying code into account. 无视自修改代码

- 起名：SDQEMU，针对 x86 到 sunway 平台

> The optimization method of using the static pre-translation in dynamic binary translation is mainly to improve the migration of the binary executable program from x86/Linux platform to the Sunway/Linux platform.
>
> And the translator SDQEMU (static pre-translation dynamic quick emulator) is equipped on the Sunway processor.



### 2019 HyperBench: A Benchmark Suite for Virtualization Capabilities [ACM Meas. Anal. Comput. Syst]

- 针对虚拟化平台的 benchmark

> In this paper, we present HyperBench, a benchmark suite that focuses on the capabilities of different virtualization platforms. Currently, we design 15 hypervisor benchmarks covering CPU, memory, and I/O.

- 设计了一个用于虚拟化处理的代价模型（衡量虚拟化的开销）

> We propose a cost model which formulates the time spent in handling hypervisor-level events.

- baseline 是 benchmark 在 host 的运行时间

> The cost model uses benchmark’s execution time natively on the host as a baseline

- guest 的执行时间包括必要的时间（direct）和虚拟机介入的时间（virt）

> The execution time of a guest application includes the necessary time taken on the host machine and additional time consumed by hypervisor-level events.
>
> T direct represents the time spent on instructions that can be executed directly on the host machine.
>
> T virt represents the runtime of instructions that need to be specially handled in a virtualized environment.

- 虚拟机介入的时间（virt）包括三部分

> he breakdown of T virt is shown in equation 3. T cpu , T memory , and T io represent the time for handling CPU, memory and I/O virtualization respectively.
>
> ​                                            T vir t = T cpu + T memory + T io + η
>
> T cpu represents the CPU virtualization overheads. The cost of CPU virtualization consists of the
> sensitive instruction cost (T sen) plus the cost caused by virtualization extension instructions (T ext)
>
> ​                                           T cpu = C sen ⊛ T sen + C ex t ⊛ T ex t
>
> T memory represents the cost of memory virtualization.
>
> - switching of page table root (T switch)
> - synchronization between the guest page tables and the page tables in the hypervisor(T sync)
> - the construction of an extra set of page table entries (T cons)
> - The indirect mapping eliminates the synchronization but introduces 2-D page walk (T two)
>
> ​             T memory = C swit ch ⊛ T swit ch + C sync ⊛ T sync + C cons ⊛ T cons + C two ⊛ T two
>
> T io represents the cost of I/O virtualization. The latency added to virtual I/O attributes to notifications in the two directions—from the I/O driver in the VM to the I/O device in the hypervisor (T out), and the opposite direction (T in).
>
> ​                                           T io = C in ⊛ T in + C out ⊛ T out

- 整体结构

> The cost model described in section 3 requires HyperBench benchmarks to run in both native and virtualized environments.
>
> HyperBench kernel is an ELF file in multiboot format, which allows grub to boot it directly.
>
> In the native environment, HyperBench kernel runs directly on the hardware.
>
> In the virtualized environment, HyperBench kernel runs as a test VM.



### 2019 Raising Binaries to LLVM IR with MCTOLL [ACM SIGPLAN]

- 静态翻译器，将二进制翻译成 LLVM IR

> This paper presents a static binary raiser that translates binaries to LLVM IR.
>
> Native binaries for a new ISA are generated from the raised LLVM IR using the LLVM compiler backend.

- 本身作为一个 LLVM tool 来进行翻译

> This paper describes an LLVM tool that raises legacy binaries to LLVM IR.

- [llvm-mctoll](https://github.com/Microsoft/llvm-mctoll)



### 2019 Translating AArch64 Floating-Point Instruction Set to the x86-64 Platform [ICPP]

- 扩展了 mc2llvm 来支持 aarch64 的浮点指令

>We extend mc2llvm, which is an LLVM-based retargetable 32-bit binary translator developed in our lab in the past several years, to support 64-bit ARM instruction set.
>
>In this paper, we report the translation of AArch64 floating-point instructions in our mc2llvm.
>
>Our work focuses on translating AArch64 ISA [3] with floating-point instructions statically and dynamically to x86-64 machine code.

- LLVM 并不支持浮点，因此添加了若干浮点相关的支持（软浮点的方式来支持各种浮点例外）

> For floating-point instructions, due to the lack of floating-point support in LLVM [13, 14], we add support for the flush-to-zero mode, not-a-number processing, floating-point exceptions, and various rounding modes.
>
> We implement software emulation to provide a complete floating-point environment that supports all floating-point exceptions as well as the flush-to-zero mode, NaN processing, and various rounding modes.

- 三种方式来提高性能

> In order to improve the performance of the binary translator, we implement three optimization techniques: block chaining, direct returns from function calls, and memory alias analysis.
>
> - Block chaining is used to reduce the number of context switches
> - The return address of a function call, which is an address in the source binary, is replaced with the target address
>
> - repeated memory accesses can be eliminated by memory alias analysis.



### :star: 2019 The Janus Triad: Exploiting Parallelism through Dynamic Binary Modification [VEE'19]

- 设计了一个二进制修改器，来挖掘程序的并行性

> We present a unified approach for exploiting thread-level, data-level, and memory-level parallelism through a same-ISA dynamic binary modifier guided by static binary analysis.
>
> We demonstrate this framework by exploiting three different kinds of parallelism to perform :
>
> - automatic vectorisation,        2.7×
>
> - software prefetching,             1.2×
> - automatic parallelisation                                           3.8× intotal

- 首先用 static 分析得到一系列 rules 用于动态优化

> A static binary analyser first examines an executable and determines the operations required to extract parallelism at runtime, encoding them as a series of rewrite rules that a dynamic binary modifier uses to perform binary transformation.
>
> Rules  / Rewrite Schedule             :   {TBA, RULE1}, {TBA, RULE2}, {TBC, RULE1}
>
> Original Executable                       :   TB-A -> TB-B -> TB-C
>
> Rewrite Schedule Interpreter      :    TB-A => RULE1, RULE2 => Modified TB-A
>
> ​                                                               TB-C => RULE1 => Modified TB-C
>
> ​                                                               Modified-TBA -> Copied TB-B -> Modified TB-C

- 基于 Janus 框架，一种最初定位于自动并行化的二进制修改框架

> Janus is a same-ISA binary modification framework that was initially developed for automatic parallelisation. It combines static binary analysis and dynamic binary modification, controlled through a domain-specific rewrite schedule.

- 软件预取：基于已有的一个分析方法，获取 loop 中的访存模式和需要计算地址的指令（来预取）

> The prefetched address is a prediction of the dynamic address that will be accessed n iterations in the future, where n is the prefetch distance.
>
> We use the Janus static analyser to detect regions that can be optimised through prefetching within the input binary. We implement the detection analysis in Janus’ static analyser based on the algorithm by Ainsworth and Jones [1] and implemented in their LLVM pass.
>
> - This detection method finds patterns of indirect memory accesses within loops and identifies all instructions that are required for address calculation for each iteration.

- 向量化

> **Scalar to SIMD Translation** The VECT_CONVERT rule is associated with every scalar machine instruction that needs to be translated to a SIMD counterpart.
>
> **Loop Unrolling and Peeling** Loop unrolling is directed by rule VECT_LOOP_UNROLL, which is associated with instructions that update the induction variable in a loop.
>
> **Initialisation and Reduction** The VECT_BROADCAST rule tags a scalar register, memory operand, or existing SIMD operand to be expanded to a full SIMD register
>
> **Runtime Bound Check** Once again, the unmodified loop body is required, but this time to provide the original loop in the case that it is deemed unsafe to vectorise at runtime. A VECT_BOUND_CHECK check can be performed on a register’s value using the condition of it being equal to a provided value, or on it being a positive value.
>
> **Runtime Symbolic Resolution** The VECT_DEP_CHECK is inserted whenever two memory accesses are identified as łmay-aliasž due to ambiguities caused by inputs or function arguments.



### 2020 Reducing Overheads of Binary Instrumentation on X86_64: Techniques and Applications [PhD]

- 降低二进制插桩的开销

> I present two approaches which can reduce the binary instrumentation overheads and thus, can help bring some of these tooling online and more widely adopted.

- 开关插桩：运行时开启/关闭插桩的能力

>  The first approach is **on-demand instrumentation**, which is the ability to enable and disable, aka toggle, instrumentation at run-time. Current instrumentation frameworks lack the primitives for cheap run-time instrumentation toggling. I present several novel instruction patching primitives, Wordpatch , Callpatch and Instruction Punning , which in combination, can be used to implement cheap, rapidly togglable instrumentation. I demonstrate the application of these primitives by developing a lightweight latency profiler.

- 插桩消除：基静态分析在安全的代码区域消除 shadow stack 的插桩代码，以降低开销

> The second approach is **instrumentation elision**, which skips inserting instrumentation at certain program locations using a particular instrumentation policy. Using shadow stacks as a case study, I demonstrate how we can elide shadow stack instrumentation on certain safe code regions as determined by binary static analysis. Finally, I present how this application of instrumentation elision leads to significant overhead reductions in shadow stacks.

- 作者发表了若干篇文章，有兴趣可一读 [link](https://www.researchgate.net/profile/Buddhika-Chamith)
  - 2020 [ShadowGuard : Optimizing the Policy and Mechanism of Shadow Stack Instrumentation using Binary Static Analysis](https://www.researchgate.net/publication/339350202_ShadowGuard_Optimizing_the_Policy_and_Mechanism_of_Shadow_Stack_Instrumentation_using_Binary_Static_Analysis)
  - 2017 [Compiling Tree Transforms to Operate on Packed Representations](https://www.researchgate.net/publication/353555930_Compiling_Tree_Transforms_to_Operate_on_Packed_Representations)
  - 2017 [Instruction punning: lightweight instrumentation for x86-64](https://www.researchgate.net/publication/317598355_Instruction_punning_lightweight_instrumentation_for_x86-64)
  - 2016 [Living on the edge: rapid-toggling probes with cross-modification on x86](https://www.researchgate.net/publication/303773670_Living_on_the_edge_rapid-toggling_probes_with_cross-modification_on_x86)



### :star: 2020 Robust Practical Binary Optimization at Run-time using LLVM [Workshop on LLVM-HPC]

Alexis Engelke, Martin Schulz

- 运行时的动态二进制优化库

> we describe BinOpt, a novel and robust library for performing application-driven binary optimization and specialization using LLVM.

- 将 function 转到 LLVM-IR 并使用运行时信息进行优化，生成新版本，之后和原本的代码结合到一起

> A machine code function is lifted directly to LLVM-IR, optimized in LLVM while making use of application-speciﬁed information, used to generate a new specialization for a function, and integrated back to the original code.

- 先前已有类似的工作：DBrew

> This approach of application guided optimization has been previously explored, e.g., by DBrew [1], which is a library for binary optimization at run-time. It provides a C API for applications, where functions can be specialized for known parameters and associated memory regions.

- 本文的方案直接将 native machine code 转化为 LLVM-IR 而不需要中间步骤

> Based on these previous experiences, we propose a new approach that directly transforms native machine code to LLVM-IR without the need for an intermediate step, like it was necessary in DBrew.

- 基于 Rellume 来将 x86_64 指令转化到 LLVM-IR

> For the actual lifting of machine code semantics, we use Rellume [3], a library designed for producing performant LLVM-IR from x86-64 machine code, providing almost complete coverage of architectural x86-64 instructions including SSE/SSE2.

- 基于 LLVM 提供的 JIT 将优化过的 LLVM-IR 转化为 machine-executable function

> After optimization, the LLVM-IR code is compiled to a new machine-executable function using the MCJIT [6] compiler provided by LLVM.



### :star: :star: 2020 More with Less – Deriving More Translation Rules with Less Training Data for DBTs Using Parameterization

Wenwen Wang

- 在此前工作基础（learn-based）上，通过参数化方案，尽可能地提取出更多的翻译规则

> In this paper, we propose a novel parameterization approach to take advantage of the regularity and the well-structured format in most modern ISAs. It allows us to extend the learned translation rules to include instructions or instruction sequences of similar structures or characteristics that are not covered in the training set.

- 对于类似的指令，生成的 rule 在一定程度上可以通用

> Assume we have learned a translation rule for the add instruction after the veriﬁcation process, as shown in the left box. We can derive a similar translation rule for the eor instruction with the same addressing mode as shown on the right, even if it is not included in the training set.

- 依据数据类型对指令进行分类（通常不同类似，比如整数和浮点，其操作完全不同，因此 rule 无法通用）

> We thus classify instructions ﬁrst into different subsets according to their data types.

- 在同一类型的指令中，进一步进行分类

> For instructions of the same data type, we further classify/divide them according to two guidelines.
>
> - The ﬁrst is that the instructions in the same subgroup must have the same encoding format, e.g. they should have the same length for X86 or the same R-type for MIPS.
> - The second guideline is that the instructions in the same subgroup must perform similar types of operations, e.g. arithmetic and logic, data transfer, or branch.

- 从两个维度来扩展翻译规则：操作码和地址模式

> After the instructions are classiﬁed into subgroups, a parameterization process is used to derive more translation rules from the learned rules. As mentioned earlier, our parameterization is done along two dimensions, namely, opcodes and addressing modes.
>
> - Their addressing modes can be register, memory or immediate.



### 2020 Unlocking the Full Potential of Heterogeneous Accelerators by Using a Hybrid Multi-Target Binary Translator

- 嵌入式平台有诸多加速器，但是使用这些加速器是个麻烦，因此二进制翻译可以动态的进行适应。本文的 HMTBT 可以把多个加速器使用起来，并进行动态调度

> Our HMTBT is capable of transparently translating code to different accelerators: a CGRA and a NEON engine, and automatically dispatching the translation to the most well-suited one, according to the type of the available parallelism at the moment.

- 支持 CGRA 以提高 ILP 和 ARM NEON 以提高 DLP

> The HMTBT supports code optimization to a CGRA, which exploits instruction-level parallelism, and to ARM NEON engine, which is responsible for exploiting data-level parallelism.
>
> A translation built from HMTBT to the NEON is a sequence of SIMD Instructions, while the translation built to the CGRA is named as a Configuration.
>
> A Translation Cache is responsible for storing SIMD instructions and CGRA configurations built by the HMTBT;



### 2020 Executing ARMv8 Loop Traces on Reconfigurable Accelerator via Binary Translation Framework [[FPL]](https://ieeexplore.ieee.org/document/9221508)

Nuno Paulino，另有一篇 2021 年的介绍 hardware automatic generation

1 page ......

- 根据二进制翻译的 traces 来自动生成加速器的电路，减少手动设计的麻烦

> Performance and power efﬁciency in edge and embedded systems can beneﬁt from specialized hardware.
>
> To avoid the effort of manual hardware design, we explore the generation of accelerator circuits from binary instruction traces for several Instruction Set Architectures

- 此前针对 loop traces 的加速工作
  -  [2018 Dynamic Partial Reconfiguration of Customized Single-Row Accelerators](https://ieeexplore.ieee.org/document/8502926) IEEE VLSI

- 重新设计的二进制翻译框架，从而扩展到其他 ISA 上，并可以挖掘更多的加速方法

> Using a redesigned binary translation framework, we are currently expanding the applicability of our approach to other Instruction Set Architectures (ISAs), and exploring additional accelerator architecture, such as custom instruction units and nested loop accelerators.



### :star: :star: 2020 A Retargetable System-level DBT Hypervisor [ACM Trans Comput Syst]

- Tom Spink 大神。Captive。
  - 2016 [Hardware-Accelerated Cross-Architecture Full-System Virtualization](https://dl.acm.org/doi/10.1145/2996798)

> In this article, we present Captive, our novel system-level DBT hypervisor, where users are relieved of low-level implementation effort for retargeting.
>
> Instead, users provide high-level architecture specifications similar to those provided by processor vendors in their ISA manuals.

- 从 guest machine 描述语言来生成相关代码，提高 DBT 的 retargetability

> In this article, we develop a novel, retargetable DBT hypervisor, which includes guest-specific modules generated from high-level guest machine specifications.

- 优化工作：离线/在线优化，以及充分利用 bare-metal 环境来加速

> We achieve this by combining offline and online optimizations and exploiting the freedom of a Just-in-time (JIT) compiler operating in a bare-metal environment provided by a Virtual Machine (VM) hypervisor.

- 激进的代码优化：offline + online

> Captive applies aggressive optimizations: It combines the offline optimizations of the architecture model with online optimizations performed within the generated JIT compiler, thus reducing the compilation overheads while providing high code quality.

- 利用裸机带来的特权资源

> Furthermore, Captive operates in a virtual bare-metal environment provided by a VM hypervisor, which enables us to fully exploit the underlying host architecture, especially its system-related and -privileged features not accessible to other DBT systems operating as user processes.

- 比 QEMU 快两倍

> We generate a DBT hypervisor outperforming Qemu by a factor of 2.21× for SPEC CPU2006 integer applications, and up to 6.49× for floating-point workloads.



### 2020 PerfDBT: Efficient Performance Regression Testing of Dynamic Binary Translation [ICCD]

Wenwen Wang

- 构建针对 DBT 性能的回归测试

> Due to the large scale and complexity of a DBT system, a minor code change may lead to unexpected impact on the performance. Therefore, it is necessary to conduct performance regression testing for DBT systems.

- 从现有的大程序中自动生成小测试程序

> we propose PerfDBT, which employs a novel approach to automatically generate test programs from existing long-running benchmarks.

- 方法很简单

> Proﬁling Block Hotness
>
> - PerfDBT employs a **hotness counter** for every executed basic block. At the end of the execution, PerfDBT ranks the blocks based on the hotness counters. PerfDBT is capable of capturing the hottest guest instructions, which can then be used to evaluate the performance of a DBT system.
>
> Analyzing Blocks for Data Preparation
>
> - PerfDBT sequentially scans memory access.
> - PerfDBT calculate the size of each memory region based on the three parameters: index × scale + of f set.
>
> Test Program Generation
>
> - Besides some supportive parts, PerfDBT typically generates several functions in a test program: e.g., init, test, fini, and main
> - The test function includes the blocks extracted from the original program
> - Firstly memory objects consumed by instructions are allocated by init



### 2020 Memory Error Detection Based on Dynamic Binary Translation [IEEE ICCT]

数学工程与先进计算国家重点实验室，郑州

- 基于 DBT 的一个动态访存错误检测

> We proposed a dynamic memory error detection method based on dynamic binary translation.
>
> It realized the detection by combining a **call stack tracing** method and an **IR language level memory error detection** method.

- 在 IR 上插桩很容易/方便

> This method takes advantage of the flexibility of the IR (Intermediate Representation) language in the dynamic binary translation technology. The flexibility of the IR language brings convenience to instrumentation, and the instrumentation at the IR language level can prevent the program under test to detect the abnormality of the running environment.

- 检测 stack frame 访存

> Stack Frame Memory Error Detection
>
> 就是 call 的时候存下 esp 之后 return 的时候对比一下是否一致，不一致则可能出现 stack 错误

- 被动的堆内存错误检测

> when allocating heap memory, set a left red zone adjacent to the lower memory in the range of the allocated heap chunk, and a right red zone adjacent to the higher.
>
> 在申请的 heap 区域两边设置 red zone

- 主动的堆内存错误检测

> we insert instrumentations into memory addressing IR language instructions.
>
> 在 IR 层面上插桩代码进行动态检查



### :star: 2019 DQEMU: A Scalable Emulator with Retargetable DBT on Distributed Platforms

Wenwen Wang

- 利用多个 host 的多个 node 来运行 DBT（分布式DBT）

> In this paper, we present a distributed DBT framework, called DQEMU, that goes beyond a single-node multicore processor and can be scaled up to a cluster of multi-node servers.
>
> One way to continue scaling the DBT performance is to go beyond one node and allows more cores to be available
>
> In this paper, we discussed the design issues in building such a distributed DBT system.

- 利用多种机制提高性能、降低开销

> In such a distributed DBT system, we integrate
>
> 1. a page-level directory-based data coherence protocol,
> 2. a hierarchical locking mechanism,
> 3. a delegation scheme for system calls, and
> 4. a remote thread migration approach that are effective in reducing its overheads.
>
> We also proposed several performance optimization strategies that include
>
> 1. page splitting to mitigate false data sharing among nodes,
> 2. data forwarding for latency hiding, and 
> 3. a hint-based locality-aware scheduling scheme.

- 应该是用户态的，Evaluation 中并没有提到 guest os

> In all of our experiments, we take ARM as the guest ISA and x86_64 as the host. The benchmark programs are compiled to ARM binaries with all libraries statically linked.

- 两种并行的编程模型

> In general, a guest parallel program can be written in two different programming models:
>
> 1. the shared-memory model
>
> 2. the distributed-memory model
>
>    - In the distributed-memory programming model, a parallel program is partitioned explicitly by the programmer into multiple independent processes.
>
>    - Each process has its own address space and is not visible to other processes.

- 对于共享内存的并行程序就比较麻烦：数据一致性协议

> However, for guest parallel programs using the shared-memory model, we need to maintain a virtual shared-memory multiprocessor system across multiple physical nodes
>
> To emulate a shared-memory multiprocessor on such a distributed-memory system, a data coherence protocol is maintained across all nodes.
>
> Those protocols can generally be classified into 
>
> 1. centralized protocols and 
> 2. decentralized protocols.
>
> In our implementation of the distributed DBT, a centralized page-level directory-based MSI protocol is employed.
>
> 本文：中心化的基于目录的页级 MSI 协议
>
> Because of the potential false data sharing when we enforce data coherence at the page level, we propose an adap- tive scheme to adjust the granularity of the coherence en- forcement at runtime.
>
> 为了解决假共享问题，提出了一个动态调整粒度的方案

- 内存一致性：memory consistency

> We use a two-level approach in DQEMU.
>
> 1. At the first level, the current QEMU in each node has incorporated a memory consistency enforcement scheme similar to the one used in PICO 单个node内部
> 2. At the second level, to enforce memory consistency across nodes, we use **sequential consistency** because our data coherence scheme is enforced at the page-level through explicit network communication 多个 node 之间保持顺序一致性

- 同步：原子指令的模拟

> We have also avoided the ABA problem by emulating the LL/SC across nodes with a global LL/SC hash table. 

- 线程创建与调度

> When a guest thread needs to be created by an existing guest thread in DQEMU, the creation event is trapped by instrumenting fork(), clone() and vfork() in Linux.
>
> 【？】fork 的话就是多个进程了吧，多个 node 上如何共享内存的？大概是 master node 把某个页传送给

- 系统调用的转发

> As mentioned earlier, the system state in DQEMU is maintained in the master node. Each thread accesses the system state with the help of a delegation mechanism on the master node.
>
> 主节点维护整个系统的状态，子节点需要访问时要向主节点请求
>
> The guest thread on the slave node traps its system calls and send the necessary information to the master node that includes guest CPU context, syscall number and the parameters.



### :star: 2021 Arbitrary and Variable Precision Floating-Point Arithmetic Support in Dynamic Binary Translation [ASPDAC]

- 需要比 IEEE 754 更高精度的浮点运算

> Floating-point hardware support has more or less been settled 35 years ago by the adoption of the IEEE 754 standard. However, many scientific applications require higher accuracy than what can be represented on 64 bits, and to that end make use of dedicated arbitrary precision software libraries.

- 而为了性能，软件会使用不同的精度，因此运算的时候需要更加加以区分

> To reach a good performance/accuracy trade-off, developers use variable precision, requiring e.g. more accuracy as the computation progresses. 

- 硬件上尚未有这种支持（不同精度、更高精度）

> Hardware accelerators for this kind of computations do not exist yet, and independently of the actual quality of the underlying arithmetic computations, defining the right instruction set architecture, memory representations, etc, for them is a challenging task.

- 本文在二进制翻译器上实现了这么一种框架，来看看到底该如何加速

> We investigate in this paper the support for arbitrary and variable precision arithmetic in a dynamic binary translator, to help gain an insight of what such an accelerator could provide as an interface to compilers, and thus programmers. We detail our design and present an implementation in QEMU using the MPRF library for the RISC-V processor.

- AI低精度足以，Sci高精度才行：本文关注于高精度

>On the low end side, it is driven by inference in neural network for which representations on a small number of bits are mandatory [8]. On the high end side, scientific applications in nuclear physics, astronomy, geoscience, etc, are seeking kernels able to solve their equations deterministically, with high precision and numerical stability.

- RISC-V 的 Guest 和 MPFR 的本地库

> We base our work on the QEMU [1] dynamic binary translator, using the RISC-V [24] target architecture, and demonstrate how to handle large number and variable precision by using as backend the MPFR [9] library.

- 扩展了 Guest 的指令集（RISC-V），来支持任意精度浮点

> To demonstrate the practicality of the approach, we first define an arbitrary precision ISA that extends the existing RISC-V ISA with new AP registers and AP instructions.

- MPFR 浮点库 [MPFR: A Multiple-Precision Binary Floating-Point Library With Correct Rounding](https://dl.acm.org/doi/pdf/10.1145/1236463.1236468)



### 2021 Efficient LLVM-Based Dynamic Binary Translation [VEE]

Alexis Engelke, Dominik Okwieka, Martin Schulz

- 基于该作者2020年发表的一个binary instrument工具Instrew，将其改造成一个RISC-V到AArch64的用户级动态二进制翻译器，直接将guest翻译为LLVM-IR并实现一些target-independent优化，相比QEMU和HQEMU有更高的性能提升。

> In this short paper, we generalize Instrew to support different guest and host architectures by refactoring the lifter and by implementing target-independent optimizations to re-use host hardware features for emulated code.

- 本文：一个可以将二进制转换到 LLVM-IR 的库

> A generalized performance-oriented library for lifting machine code to LLVM-IR that can be easily extended for other architectures.

- 本文：利用 host 的 RAS 对 call 和 return 进行优化
  - 在 call 之后继续翻译而不会停下（反正也要回来），但是会导致翻译耗时增加（TB变大）

> An approach to exploit the hardware return address stack for the emulated program by re-using host instructions for function calls and returns.
>
> 1. function calls in the original code are lifted to an LLVM-IR function call to the dispatcher
> 2. After that LLVM-IR call, the new program counter has to be verified to have the expected value so that the code after the call can be safely executed, otherwise the dispatcher needs to be called **again**.
> 3. Function returns are lifted to LLVM-IR return instructions
> 4. indirect jumps are realized as tail calls to the dispatcher
>
> The difference to other approaches utilizing host call/return instructions [11, 15] lies in the increased translation granularity by continuing decoding after call instructions.



### :star: 2021 BTMMU: An Efficient and Versatile Cross-ISA Memory Virtualization

- TLB MID + `setmem` 扩展了地址空间的数量
- 利用 x86 CR3 实现 `multiple shadow page table` FOR `processes in guest OS` 



### :star: 2021 Enhancing Dynamic Binary Translation in Mobile Computing by Leveraging Polyhedral Optimization [WCMC - Hindawi]

数学工程与先进计算国家重点实验室，郑州

- LLPEMU：顺序程序并行化（在翻译的时候将顺序程序转换成并行的）

> In this work, we focus on leveraging ubiquitous multicore processors to improve DBT performance by parallelizing sequential applications during translation.
>
> For that, we propose LLPEMU, a DBT framework that combines binary translation with polyhedral optimization.

- 分析 loop 的 CFG 来将循环拆分，以达到并行的目的

> Our goal is to enable a DBT system to parallelize loop nests in the sequential guest code without incurring high runtime overhead.
>
> **To parallelize the binaries, we must perform complex analysis to recover the CFG and extract the loops.**
>
> It is crucial to design the system architecture such that it can extract loops and perform **polyhedral optimization** without introducing high runtime overhead.
>
> The second issue is how to parallelize loops extracted from guest binaries.
>
> 1. Many polyhedral optimizers have been developed, such as PLUTO [8] and Polly [5].
> 2. On the basis of these considerations, we choose Polly as our parallelizer. Polly, which has been developed on the basis of LLVM infrastructure [9], analyzes and parallelizes loops at the IR level.

- 静态阶段：分析 loop 转换成并行的 host 代码。

> In the static stage, loops are extracted from the guest application and then translated into the parallelized host machine code.

- 动静结合的方式：将静态结果 `-O3 -shared -PIC` 成动态库，每个 target loop 对应一个 funciton
  - 使用 POSIX API `dlopen` `dlsym` 来获取对应 function 的地址
  - 在 code cache 中设置对应的 TB 将代码重定位到 static generated codes 中（shared lib）

> Related information ﬁles are also generated to enable the dynamic binary translator to leverage the static analysis results and load the parallelized code.
>
> When translating the target loops, DBT will redirect the control ﬂow from the original translation-execution loop to the execution of the parallelized code generated in the static stage.
>
> Code fragments in the code cache are not the translated code from the basic block but a prologue to redirect the execution from the code cache to the parallelized code.

- 针对 TCG 造成的 TB 重叠的问题（导致 CFG 变得复杂难以分析）：插入跳转指令，明确指出边界

> To solve this problem, we insert a jump instruction to the succeeding instruction as the end of the basic block when we ﬁnd that the next instruction is the entry of another basic block based on CFG.

- 利用 LLVM Pass 来优化 IR，同时使其适配 Polly 从而进行并行优化

> The optimizer is developed on the basis of prior knowledge of DBT and implemented as LLVM passes to bridge the gap between the translator and Polly. Then, IR optimized by the optimizer will be taken into the parallelizer
>
> Polly is a polyhedral optimization tool based on LLVM infrastructure.
>
> 1. It is implemented as a set of LLVM passes, to perform analysis and transformations on IR
> 2. Polly is designed to optimize IR generated by Clang, and some passes in Polly rely on results of general pass such as the loop pass and SCEV pass

- Polly 无法直接处理 translated IR，因此需要将其转换一下

> However, translated IR is far more complicated than such unoptimized IR; hence, Polly always **fails** by directly taking the translated IR as input.
>
> The key strategy is to recover the loop induction variables and memory access symbolic description. Based on recovered loop structure information, the translated IR is then transformed into an equivalent version with GEP instructions, which is suitable for polyhedral optimization on Polly.



### 2021 A Binary Translation Framework for Automated Hardware Generation

Nuno Paulino

- 边缘计算 → 能效比 → 异构 → 特定应用加速器 → 适配工作负载（本文）

> As applications move to the edge, efﬁciency in computing power and power/energy consumption is required.
>
> Heterogeneous computing promises to meet these requirements through application-speciﬁc hardware accelerators.
>
> **Runtime adaptivity** might be of paramount importance to realize the potential of hardware specialization, but further study is required on workload retargeting and ofﬂoading to reconﬁgurable hardware.
>
> This article presents our framework for the exploration of both ofﬂoading and hardware generation techniques.
>
> The framework is currently able to process instruction sequences from MicroBlaze, ARMv8, and riscv32imaf binaries, and to represent them as Control and Dataﬂow Graphs for transformation to implementations of hardware modules.

- 基于 DBT 的加速技术，将应用自适应的运行在可配置的平台上

> We summarize the technology trends underlying the emergence of heterogeneous platforms and provide an overview of a framework for exploring binary translation-based acceleration techniques applicable to future self-adaptive systems that shift work to available reconﬁgurable resources at runtime.

- 整体框架：decode → block, loops → CDFG → BSG and AST → HDL → Circuits

> These are the instruction stream extraction, segment detection, control and dataﬂow graph (CDFG) generation, and hardware generation stages.



### 2021 Helper Function Inlining in Dynamic Binary Translation [ACM SIGPLAN Compiler Construction CC‘21]

Wenwen Wang

- 把 helper 给内联了

> To mitigate the engineering effort, helper functions are frequently employed during the development of a DBT system.
>
> To solve this problem, this paper presents a novel approach to inline helper functions in DBT systems.
>
> As a result, the performance overhead introduced by helper function calls can be reduced, and meanwhile, the benefits of helper functions for DBT development are not lost.

- 把 host binary 转换个形式，以方便的放入 code cache

> The approach automatically transforms the natively-compiled host code of a helper function into a form that is feasible to be smoothly inlined into the translated host binary code and executed safely in the code cache.
>
> To solve this issue, our approach automatically scans the host assembly code to identify potentially problematic instructions and transform them into instructions that can be correctly executed in the code cache. 比如 PC 相关的指令，扫描一遍然后改掉。。。relocate！
>
> we need to insert extra assembly instructions before and after each call instruction to fulfill the duty of context switches. 对于 helper 中的 call 插入 context switch 代码进行调用
>
> we transform each return instruction into a direct branch instruction 对于 return 指令改为直接跳转即可

- 步骤：（1）拷贝代码（2）重定位

> There are two steps to inline a helper function. First, we copy the host binary code of the helper function from the loaded hfi file to the code cache. Second, we relocate the copied binary code based on the relocation records specified in the section of this helper function.

- 关键 `helper_lookup_tb_ptr` 



### :star: :star: 2021 Enhancing Atomic Instruction Emulation for Cross-ISA Dynamic Binary Translation

Wenwen Wang

- 原子指令模拟的 ABA 问题

- 软件方案1：哈希表 Hash Table-Based Store Test (HST)
  - 利用事务性内存的方案：HTM
  - 根据观察弱化的方案：HST-WEAK（多线程用LL/SC来修改，而只有持有锁的线程会用普通的store指令）

> The hash table-based store test scheme (HST) is a software-based scheme. In order to correctly emulate the LL and SC instructions, the violation of their atomicity must be detected.
>
> HST can be further optimized if there is additional hardware support, such as hardware transactional memory (**HTM**), to implement the critical section for the SC emulation.
>
> Based on the analysis, the shared data can be modified by multiple threads using atomic instructions such as LL/SC during lock contention, and only be updated by the lock-owner thread with normal stores.

- 软件方案2：页保护 Page Protection-Based Store Test (PST)

> To monitor the changes to the atomic variable of LL/SC by the store instructions in non-transactional code regions, we can also employ the page protection mechanism in OS. 
>
> When emulating the LL instruction, in addition to access the hash table entry, it will set the page that contains its atomic variable to ”read-only”.



### 2019 Heuristics to predict and eagerly translate code in DBTs [PhD]

University of Kansas School of Engineering 堪萨斯大学工程学院

- 加快翻译过程，提高 DBT 的性能：多线程加快翻译

> We explore the possibility of improving the performance of a DBT named DynamoRIO.
>
> The basic idea is to improve its performance by speeding-up the process of guest code translationg, through multiple threads translating multiple pieces of code concurrently.

- 用 test 测试集测试 translation 的 overhead

> The test-inputs with shorter run-times are more signifificantly impacted by the DynamoRIO overhead.
>
> Based on that reasoning and the supporting experimental data, we deviced to focus on test-inputs and to run further experiments without traces.

- 提前翻译（不是很有趣/新鲜的想法）

> The idea is to translate basic blocks ahead of time, such that the required target code is ready to run when needed. For that to happen, we opted parallelization of the compilation process through multiple compiler threads.
>
> We call it eager translation.
>
> The design to implement our eager translation entails capturing direct addresses and populating them in a list. That list of addresses would be processed by multiple compiler threads ahead of time, with appropriate synchronization among all threads. 感觉没说清楚：到底是（1）动态的生成这个 list 来被多个翻译线程处理，还是（2）提前扫描程序发现这个 list 之后运行时被多线程拿去处理
>
> Based on the collected data, we found that the percentage of useful eager translations is quite low. 发现有效的提前翻译很低。。。

- 借助数据挖掘技术来启发式产生翻译需求

> We opted to expolre data-mining techniques to geenrate heuristics/rule-sets that could be applied at run-time during application run under DynamoRIO.
>
> The idea is to have a set of simple rules/heuristics as they would have to be applied at run-time without incurring much overhead.
>
> Once the input training data was available from **ealier** benchmark runs, next thing to do was to generate the rule-sets.



### 2019 Runtime Software Monitoring Based on Binary Code Translation for Real-Time Software [JIPS Journal of Information Processing Systems]

软件测试：静态测试 / 运行时测试 → 自动化：自动生成测试样例 / 自动进行运行测试

- 本文：提出一种可以自动将实时软件转换成可监控的软件，方便进行测试

> Traditional runtime software testing based on handwork is very inefficient and time consuming. Hence, test automation methodologies in runtime is demanding.
>
> In this paper, we introduce a software testing system that translates a real-time software into a monitorable real-time software.
>
> The monitorable real-time software means the software provides the monitoring information in runtime.

- 根据 ELF 和需要监控的 function 使用二进制翻译来生成 monitorable ELF

> The inputs of the system are the executable linkable format (ELF) binary code and the symbol of target function to be monitored.
>
> The output of the system is the monitorable binary code.



### 2019 一种高效解决间接转移的反馈式静态二进制翻译方法 [计研发]

An Efficient Feedback Static Binary Translator for Solving Indirect Branch

数学工程与先进计算国家重点实验室，郑州

- 课题组已有 SQEMU 的静态翻译器：逐行翻译，线性遍历
- FD-SQEMU 反馈式：先静态翻译，运行过程中调用翻译器，翻译间接跳转的目标代码
  - 二级地址表快速定位：直接用 guest 的 address 作为 idx（需要一个 base）来快速获取 TB 地址
  - 不需要 hash 而是直接用 guest address 与 base 的差值本身：空间换时间



### 2019 二进制翻译中动静结合的寄存器分配优化方法 [计研发]

A Dynamic and Static Combined Register Mapping Method in Binary Translation

数学工程与先进计算国家重点实验室，郑州

- QEMU TCG 的分配策略
  - FMB First-Mean-Busy 线性扫描静态固定分配
  - 寄存器溢出：无空闲时，将其中一个写回内存，以释放出寄存器
  - 缺陷：缺乏指令间、循环等信息，造成冗余

- 本文：全局静态分配 + 局部动态分配
  - 首先依据程序的静态统计特征，实现静态的全局寄存器直接映射【可以理解为寄存器映射，从而不必写回内存】
  - 然后借助程序 CFG 减少不必要的寄存器溢出
    - 统计基本块内使用寄存器的次数，优先对更多次使用的进行映射
    - 分析循环体，可以将全局映射溢出，以提高局部循环的性能
- host 有足够的寄存器即可全部映射了。。。



### 2019 二进制翻译正确性及优化方法的形式化模型

Formal Model of Correctness and Optimization on Binary Translation

数学工程与先进计算国家重点实验室，郑州

- 全是形式化描述，读不动...... TODO ?



### 2019 块链优化技术在动态二进制翻译中的应用研究

- 不就是代码块链接吗



### 2020 基于 LLVM 的多目标高性能动态二进制翻译框架

- 一种基于 LLVM 的动态翻译框架，基于开源的 Skyeye，称之为 Dyncom
  - 混合执行：解释 + 翻译
  - 超块构造：trace。两遍扫描：（1）识别基本块和超级块（2）翻译
  - 优化：跳转、寄存器映射



### 2019 基于 QEMU 的动态二进制插桩技术 [计研发]

- 全系统仿真，基于中间代码的插桩、跟踪和记录程序执行流



### 2019 基于 QEMU 翻译系统 SIMD 指令翻译优化方法 [信息工程大学学报]

以申威处理器为 host 平台优化 QEMU 的 SIMD 指令翻译

1. 修改已有的 helper 函数
2. 引入新的向量 IR 来提高翻译效率



### 2021 基于中间表示规则替换的二进制翻译中间代码优化方法 [国防科技大学学报]

仍然是申威处理器

- 在 IR 层面上以一种基于规则的复杂的方式实现了寄存器映射。。。。。。



### 2019 基于动态二进制翻译和插桩的函数调用跟踪 [计研发]

信息系统安全技术国家重点实验室

（摘要第二句话没有主语...）

> 动态函数调用跟踪技术是调试 Linux 内核的重要手段。
>
> 针对现有动态跟踪工具存在支持平台有限、运行效率低的问题，基于二进制翻译，设计并实现支持多种指令集的动态函数调用跟踪工具。

- 现有的函数调用动态分析工具

> gprof 仅支持启动用户态可执行程序，不能用于内核分析；
>
> ftrace 需要编译内核时打开编译选项，造成多种编译优化手段无法进行；
>
> systemtap 与 dtrace 需要操作系统运行时提供接口，操作系统启动阶段无法使用

- 基于模拟器的函数跟踪技术直接分析二进制镜像
- 在中间代码阶段，针对特殊的函数调用与返回指令进行插桩（调一个 helper 收集信息）



### 2019 基于译码制导技术的动态二进制翻译优化研究 [电脑知识]

这特么题目是个啥。。。不好好说人话

- 译码：收集信息（EFLAGS的使用情况、是否触发异常）
- 翻译：根据信息生成优化的代码（根本没说如何识别异常）



### 2021 轻量级嵌入式软件动态二进制插桩算法 [理论研究]

- 利用断点调试协议实现插桩调试功能

> 为了跟踪动态执行过程和获取运行时信息，设置了远程监控进程对断点的触发进行监控。
>
> 为确定断点插入的位置，设计了静态分析算法以实现对控制流图和桩点集合的计算。



### 2021 基于申威CPU的RISC-V指令系统实时仿真研究 [电子科大硕士]

从 RISC-V 到 申威 的二进制翻译器的设计与实现（能跑简单的程序）

- 本文的主要工作

> 1. 研究 RISC-V 和申威两个指令集
> 2. 实现二进制翻译器
> 3. 编写测试
