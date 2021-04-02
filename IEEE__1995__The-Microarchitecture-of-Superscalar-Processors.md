# Linfeng Zheng (2020-12-22)

# Paper information

* Title: The Microarchitecture of Superscalar Processors
* Authors: James E. Smith, Gurindar S. Sohi
* Venue: IEEE 1995
* Keywords: Microarchitecture, Pipeline processing, Microprocessors, Parallel processing, Clocks, Reduced instruction set computing, Processor scheduling, Dynamic scheduling, Registers, Technological innovation

# Paper content

## Summary

This paper figures out what problems superscalar processors have solved and what characteristics should superscalar processors have.

---

* The main problem solved by superscalar processors is converting an ostensibly sequential program into a more parallel one, which needs to overcome two dependencies:
  * Control dependency
  * Data dependency
* The key idea of the superscalar processor is ***dynamic instruction scheduling,*** which can analyze data dependence, dispatch instructions to different functional units, and execute instructions in parallel.
* The major parts of the microarchitecture of superscalar processors are:
  * Instruction fetching and branch prediction
  * Instruction decoding, register renaming, and instruction dispatching
  * Instruction issuing and parallel execution
  * Memory operation analysis
  * Instruction reorder and instruction committing

---

Key mechanisms:

* ***Instruction renaming*** changes registers' names which are originally specified by instructions.

  Superscalar processors can execute multiple instructions simultaneously and do not follow the order of *Program Counter* (PC) to execute instructions, which could violate the dependencies between instructions then get wrong results. In order to keep the correct result, the dependencies must be maintained. Instructions need to wait for results that have not yet completed, which would cause performance degradation. *Instruction renaming* renames the logical register into a different register naming space (physical registers) in order to break the false dependencies, thus achieving higher performance. Logical registers are needed by Instruction Set Architecture (ISA). They are used to provides precise state data in some situations like the interruption. Physical registers are additional hardware structures, which are used to store logical register values. Logical registers' values at different points in time are recorded in physical registers for instructions.
* ***Instruction dispatching*** dispatches different types of instructions into different functional units to be executed.

  After superscalar processors decoded instructions and checked dependencies analyzing, different types of instructions are dispatched into different functional unit buffers waiting for issuing. Once resources needed by instructions are available, they would be issued to functional units to be executed in parallel. *Instruction dispatching* prevents ALUs from being idle for too long.
* ***Commit state*** maintains the original sequential order and the precise state.

  If instructions update logical registers immediately after execution, it would get false results because superscalar processors do not follow original program order to execute instructions. Therefore, there is an additional *commit state* to update logical registers following original sequential order. After instructions were executed, their results are store in a buffer temporarily, then commit them by sequential order.

---

Key conclusions:

The superscalar processors execute multiple instructions by breaking false dependencies, which have both performance improvement and compatibility. However, the more instructions are executed simultaneously, the less performance improves. Therefore, there will be new execution models to get better performance easily, like *Very Long Instruction Word* (VLIW) and *multiprocessing*.

Most of the implementation of superscalar processors is on the hardware level. Therefore, it can speed up existing programs. Moreover, the software level can assist by creating optimized programs. There are two ways.

* Reducing the possibility that instruction needs to wait for values needed

  In static code, the compiler could make an instruction that produces a value stay away from other instructions that need this value.
* Enhance the possibility that a group of instructions can be issued simultaneously.

  The compiler could optimize the order of instructions according to some aspects that limit the performance of superscalar processors, like the dependence relationships between the instructions and the resources available in the microarchitecture.

## Strengths

The first strength of superscalar processors is that it fully utilizes most of the hardware resources. Before superscalar processors were invented, most functional units would be idle when instructions are executed, which is inefficient. Superscalar processors try to make all functional units work simultaneously, which can speed up executing instructions.

The second is there are many caches introduced, which could improve performance.

* *Instruction buffer* (speed up instruction fetching)
* *Translation lookaside buffer* and *Memory hierarchy* (speed up handling memory operations)

Three examples of the superscalar processor in this paper are representative of the typical superscalar processor. They cover most of the hardware implementation mentioned in this paper and two different types of ISA.

* *MIPS R1000* is a RISC processor.

  It implements a prediction table and resume buffer for branch prediction and implements *Multiple Queue Method* and *Re-order buffer* for *dynamic instruction scheduling*.
* *Alpha 21164* is also a RISC processor.

  It just implements a simple superscalar processor, which means it forgoes the advantages of dynamic instruction scheduling in favor of a high clock rate. It only implements the *Single Queue Method* without *OoO* execution.
* *AMD K5* is a CISC (x86) processor.

  It converts the original instruction byte to *RISA-like Operations* in the decoding stage and using an instruction buffer for branch prediction. As for *dynamic instruction scheduling*, it implements two reservation stations for each functional unit except *FPU*.

## Weaknesses

The first is too complex hardware. Especially when superscalar processors introduce the Out of Order execution, there must be additional physical registers to be introduced. The time cost of dependency checking logic would increase, and it needs to associate with functional units. So, the performance would be decreased, and it would consume more energy.

The second is the number of instructions that can issue simultaneously would be limited by the number of ALUs.

The third is that the branch prediction is inefficient. If speculative execution goes a false branch, processors may fetch and execute instructions again.

## Thoughts

Authors care about almost of aspects to speed up efficiency in this paper. I am not confident that I could do better now.

However, I think there is room to improve performance on speculative execution. For example, using sophisticated methods to improve the accuracy or executing all of the branches' instructions and putting them into a temporary buffer. But the latter method would increase hardware complexity, and processors would execute some unnecessary instructions. So, I am not sure if it is practical.

Furthermore, most of the strengths of superscalar processors would make hardware design more complex, which is not "beautiful." But there may be a tradeoff between simplifying and performance-enhancing.

## Takeaways and questions

I think this paper is nice and comprehensible. Moreover, it helped me sort out some of the key points, like the relationship between OoO and superscalar.

There is a question about VLIW. The authors mentioned that some schools of thoughts thought the VLIW is the future. It seems that VLIW was failed in CPUs markets. But I think the VLIW is very similar to the SIMD and the SIMD is very useful in deep learning areas. Is it possible to make VLIW revive in machine learning areas?