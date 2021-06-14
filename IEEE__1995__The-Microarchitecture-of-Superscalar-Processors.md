# Linfeng Zheng (2020-12-22)

# Paper information
- Title: The Microarchitecture of Superscalar Processors
- Authors: James E. Smith, Gurindar S. Sohi
- Venue: IEEE 1995
- Keywords: Microarchitecture, Pipeline processing, Microprocessors, Parallel processing, Clocks, Reduced instruction set computing, Processor scheduling, Dynamic scheduling, Registers, Technological innovation

# Paper content
## Summary
### Trying to solve
This paper figures out what problems superscalar processors should solve and what characteristics microarchitecture of superscalar processors should have.

### Key Ideas
- The main problem solved by superscalar processors is converting an ostensibly sequential program into a more parallel one, which needs to overcome two dependences:
  - Control dependence
  - Data dependence

- The key idea of the superscalar processor is *dynamic instruction scheduling*, which can analyze data dependence, dispatch instructions to different functional units, and execute instructions in parallel.

### Key mechanisms
- Register renaming
  In static programs, instructions' storage elements are *architectural storage elements* that include *logical registers* and *main memory locations*. *Logical registers* may be reused in some sequence of instructions thus creating some unnecessary data dependences that are called *false* data dependence or *artificial* data dependence (e.g., *write-after-write*, and *write-after-read*). These data dependences can be overcome by mapping *logical registers* to *physical registers*. One single *logical register* can be mapped to multiple *physical registers* and these *physical registers* record values of this *logical register* at different points in time, which will be used by some sequent instructions. When instructions are executed later, hardware will load/store values into mapped *physical registers* directly instead of original *logical registers*.

- Instruction dispatch & Parallel execution
  After instructions' registers are renamed, different types of instructions are dispatched into different functional unit buffers to wait for issuing. Once instructions' needed resources are available, they would be issued to functional units to be executed in parallel.

- Committing state
  Because of parallel execution, instructions were not completed in the original program order. Therefore, superscalar processors need a buffer to store execution results temporarily and submit results in the original program order. When instructions in this buffer, they are in committing state.

### Key conclusions
1. The superscalar processors execute multiple instructions by breaking false dependencies, which have both performance improvement and compatibility. However, the more instructions are executed simultaneously, the less performance improves due to conditional branches and control complexity.

2. The software level can optimize programs to get better performance in superscalar processors from two aspects:
   - Alleviating the impact of true data dependences (*read-after-write*).
     In static code, the compiler could make an instruction that produces a value stay away from other instructions that need this value.
   - Enhancing the possibility that a sequence of instructions can be issued simultaneously.
     The compiler could optimize the order of instructions according to some aspects that limit the performance of superscalar processors, like the dependence relationships between the instructions and the resources available in the microarchitecture.

## Strengths
1. Superscalar processors fully utilize most hardware resources. Before superscalar processors were invented, most functional units would be idle when instructions are executed, which is inefficient. Superscalar processors try to make all functional units work simultaneously, which can speed up executing instructions.

2. There are many caches introduced, which could improve performance and alleviate bottlenecks.
   - Instruction buffer (speed up instruction fetching)
   - Translation lookaside buffer and memory cache hierarchy (speed up handling memory operations)

## Weaknesses
1. More complex hardware design and implementation
   Typical microarchitecture of superscalar processors introduced register naming, instruction dispatch, and committing state, which requires more design and implementation costs. Moreover, one limitation of parallel execution is the number of ALUs. To support executing more instructions simultaneously, more ALUs are integrated into superscalar processors, which require more hardware implementation. Finally, the multi-level cache (cache hierarchy) is introduced to speed up main memory access. This design needs to consider cache coherency, which is a complex design.

2. More energy consuming
   When instructions are executed in superscalar processors, there are more processes than scalar processors like renaming registers, dispatching instructions, committing instructions, and replacing cache. These processes may consume significant energy.

3. Bottlenecks restrict the further performance improvement
   - The cache hierarchy 
     Speed of cache access cannot catch up with instruction execution even if it is used to speed up main memory access. If the cache miss occurs, the speed is worse.
   - Speculative execution
     Modern superscalar processors are deep pipelined architecture for better performance. This design has a worse penalty if false branches are taken.

## Thoughts
I think there is room to improve performance on speculative execution. For example, using sophisticated methods to improve the accuracy or executing all of the branches' instructions and putting them into a temporary buffer. But the latter method would increase hardware complexity, and processors would execute some unnecessary instructions. So, I am not sure if it is practical.

Furthermore, most of the strengths of superscalar processors would make hardware design more complex, which is not grace. But this is a tradeoff between simplifying and performance-enhancing.

## Takeaways and questions
I think this paper is nice and comprehensible. Moreover, it helped me sort out some of the key points, like the relationship between OoO and superscalar.

There is a question about VLIW. The authors mentioned that some schools of thought thought the VLIW is the future. It seems that VLIW has failed in CPUs markets. But I think the VLIW is very similar to the SIMD and the SIMD is very useful in deep learning areas. Is it possible to make VLIW revive in machine learning areas?