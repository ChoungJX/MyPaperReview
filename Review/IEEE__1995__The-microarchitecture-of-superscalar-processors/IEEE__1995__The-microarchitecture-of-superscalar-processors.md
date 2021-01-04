# Linfeng Zheng (2020-12-22)

# Paper information

- Title: The Microarchitecture of Superscalar Processors
- Authors: James E. Smith, Gurindar S. Sohi
- Venue: 
- Keywords: Microarchitecture, Pipeline processing, Microprocessors, Parallel processing, Clocks, Reduced instruction set computing, Processor scheduling, Dynamic scheduling, Registers, Technological innovation

# Paper content

## Summary

This paper figures out what problems have superscalar processors solved and what characteristics should superscalar processors have.

---

- The main problem solved by superscalar processors is converting an ostensibly sequential program into a more parallel one, which needs to overcome two dependencies:
  - Control dependency
  - Data dependency
- The key idea of the superscalar processor is ***dynamic instruction scheduling*** which can analyze data dependence, dispatch instructions to different functional units, and execute instructions in parallel.
- The major parts of the microarchitecture of superscalar processors are:
  - Instruction fetching and branch prediction
  - Instruction decoding, register renaming, and instruction dispatching
  - Instruction issuing and parallel execution
  - Memory operation analysis
  - Instruction reorder and instruction committing

---

Key mechanisms:

- ***Instruction renaming*** changes registers' names which are originally specified by instructions.

  If the microarchitecture of a processor is not pipelined or superscalar, it is unnecessary to rename registers' name. Because this processor handles the instruction strictly followed *program counter* (PC), it can simply access registers which instructions need. However, superscalar processors execute multiple instructions simultaneously although it looks like a sequential model ostensibly. Some instructions depend on values that are produced by other instructions, they must wait until these values available to be executed, which would reduce execution performance. Instruction renaming renames the logical register into a different register naming space (physical registers) in order to break the false dependencies thus achieving higher performance. Logical registers are needed by Instruction Set Architecture (ISA). It uses to store the data updated by commit state and provides precise state data when an interruption occurred. Physical registers are additional hardware structures used to store logical register values. Multiple data values are stored in multiple physical registers.  These values may associate with one specific logical register but each of these values corresponds to different points in time during a purely sequential execution process. The false dependencies refer to dependencies that can be avoided like Write after Write (WAW) which occurs when multiple instructions update the same storage location, and Write after Read (WAR) which occurs when an instruction needs to write a new value into storage location, but must wait until all preceding instructions needing to read the old value have done so. However, the true dependencies cannot be eliminated by instructions renaming like Read after Write (RAW). Because the instruction can only read the value after the another instruction has produced it.

- ***Instruction dispatching*** can dispatch different types of instructions into different functional units to be executed.

  In scalar processors, there is only one instruction to be fetched and decoded per cycle, which means there is only one instruction can be executed at the same time. However, superscalar processors have multiple parallel instruction pipelines, superscalar processors can fetch and decode multiple instructions per cycle. After superscalar processors finished decoding and dependence analyzing, they dispatch instructions into different functional unit buffers waiting for issuing. Once the instructions' needed registers are available, they would be issued to functional units to be executed. This is why superscalar processors can execute more than one instruction per cycle.

- ***Commit state*** maintains the original sequential order and the precise state.

  There is no need for a commit state in a processor whose microarchitecture is a pure sequential execution model. Because it fetches, decodes, and executes one instruction strictly followed the correct order which is specified by the PC. Registers' value can be updated directly. However, superscalar processors can fetch, decode, and execute multiple instructions simultaneously. Moreover, different types of the functional unit would spend different time to execute an instruction. Therefore, the order after parallel execution in superscalar processors would not match the order of program counter specified. It is necessary to reorder instructions that finished executing, then, commit instructions followed the correct order (Update the value of registers).

  In addition, some hardware implementation of commit state (like *re-order buffer*) can work as a buffer to offer data to functional unit buffers by bypasses which is a bus connecting with ROB and functional unit buffers. In functional unit buffers, some instructions wait for data needed to become available. They can get these data from *ROB* instead of waiting until the data are updated.

---

The superscalar processor implements a sequential execution model ostensibly but executes instructions concurrently in fact. Both performance and compatibility have driven the development of superscalar processors. But the more parallel hardware resources are introduced, the fewer returns would be gained. So, there will be some new method to increase performance like VLIW and multiprocessing.

Most of the implementation of superscalar processors on the hardware level. Therefore, it can speed up existing programs. However, the software level can assist by creating optimized programs. There are two ways.

- Decreasing the likelihood that an instruction has to wait for the result of previous instruction when it could have been issued.

  An instruction that produces a value for another instruction can be placed in the static code so that it is fetched and executed far in advance of the consumer instruction.

- Increasing the likelihood that a group of instructions can be issued simultaneously.

  To arrange instructions so that a group of instructions in the static program matches the parallel execution constraints of the underlying superscalar processor that need to be considered include the dependence relationships between the instructions, and the resources available in the microarchitecture.

## Strengths

The first strength of superscalar processors is that it fully utilizes most of the hardware resources. Before superscalar processors were invented, most of the functional units would be idle when instructions are executed, which is inefficient. Superscalar processors try to make all of functional units work simultaneously, which can speed up executing instructions.

The second is there are many caches introduced, which could improve performance.

- *Instruction buffer* (speed up instruction fetching)
- *Translation lookaside buffer* and *Memory hierarchy* (speed up handling memory operations)

Three examples of the superscalar processor in this paper are representative the typical superscalar processor. They cover most of the hardware implementation mentioned in this paper and two different types of ISA.

- *MIPS R1000* is a RISC processor. 

  It implements a prediction table and resume buffer for branch prediction and implements *Multiple Queue Method* and *Re-order buffer* for *dynamic instruction scheduling*.

- *Alpha 21164* is also a RISC processor.

  It just implements a simple superscalar processor, which means it forgoes the advantages of dynamic instruction scheduling in favor of a high clock rate. It only implements the *Single Queue Method* without *OoO* execution.

- *AMD K5* is a CISC (x86) processor.

  It converts the original instruction byte to *RISA-like Operations* in the decoding stage and using instruction buffer for branch prediction. As for *dynamic instruction scheduling*, it implements two reservation stations for each functional unit except *FPU*.

## Weaknesses

The first is too complex hardware. Especially when superscalar processors introduce the Out of Order execution, there must be additional physical registers to be introduced. In addition, the time cost of dependency checking logic would increase and it needs to associate with functional units. So, the performance would be decreased, and it would consume more energy.

The second is the number of instructions that can issue simultaneously would be limited by the number of ALUs.

The third is the branch prediction is inefficient. If speculative execution goes a false branch, processors may fetch and execute instructions again.

## Thoughts

Authors care about almost of aspects to speed up efficiency in this paper. I am not confident that could do better now. 

However, I think there are room to improve performance on speculative execution. For example, using sophisticated methods to improve the accuracy or executing all of branches instructions and putting them into a temporary buffer. But the later method would increase hardware complexity and processors would execute some unnecessary instructions. So, I am not sure if it is practical.

Furthermore, most of strengths of superscalar processors would make hardware design become more complex, which is not "beautiful". But there may be a tradeoff between simplifying and performance enhancing.

## Takeaways and questions

I think this paper is nice and comprehensible. Moreover, it helped me sort out some of the key points like the relationship between OoO and superscalar. 

There is a question about VLIW. Authors mentioned that some schools of thoughts thought the VLIW is the future. It seems that VLIW was failed in CPUs markets. But I think the VLIW is very similar to the SIMD and the SIMD is very useful in deep learning areas. Is it possible to make VLIW revive in machine learning areas?

