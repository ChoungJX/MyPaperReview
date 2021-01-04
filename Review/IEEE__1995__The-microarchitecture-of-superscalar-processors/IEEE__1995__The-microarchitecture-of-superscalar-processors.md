# Linfeng Zheng (2020-12-16)

# Paper information

- Title: The Microarchitecture of Superscalar Processors
- Authors: J.E. Smith, G.S. Sohi
- Venue: 
- Keywords: Microarchitecture, Pipeline processing, Microprocessors, Parallel processing, Clocks, Reduced instruction set computing, Processor scheduling, Dynamic scheduling, Registers, Technological innovation

# Paper content

## Summary

This paper figures out what problems have superscalar processors solved and what characteristics should superscalar processors have.

---

- The key idea of the superscalar processor is ***dynamic instruction scheduling*** which can analyze data dependence, dispatch instructions to different functional units, and execute instructions in parallel.
- Two main problems are solved in the superscalar processors' implementation. 
  - Dependences
  - Parallel execution
- There are five crucial components in superscalar processors.
  - Instruction Fetching and Branch Prediction
  - Instruction Decoding, Renaming, and Dispatch
  - Instruction Issuing and Parallel Execution
  - Memory Operations
  - Committing State

---

Key mechanisms:

- ***Instruction renaming*** changes registers' names which are pointed to by instructions.

  In a real sequential execution model, processors can simply read or write registers which instructions need. However, RAW, WAW, and WAR hazards would reduce concurrent performance in *out of order* execution model. So, this mechanism introduces two types of register. One is logical registers which are origin registers that processors should have. It uses to store global information and provides precise state data when an interruption occurred. Another type of register is additional hardware implementation used to storage logical register values. Many physical registers may associate one specific logical register but each of these values corresponds to different points in time. Artificial hazards like WAW and WAR could be eliminated by *instruction renaming*.

- ***Instruction dispatching*** can dispatch different types of instructions into different functional units to be executed.

  In a scalar processor, there is only one instruction to be executing in per time cycle, which means other ALUs would be wasted. Superscalar processors can dispatch instructions into different functional unit buffers. Once the instructions' needed data are available, they would be issued. This is why superscalar processors can execute more than one instruction per time cycle.

- ***Committing state*** maintains the original sequential order and the precise state.

  There is no committing state in a real sequential execution model because processors execute instructions in sequential order and can directly update registers' value after execution. However, origin sequential order would be disordered by *instruction dispatch* and parallel execution. So, it is necessary to introduce the committing state to recover origin sequential order and update correct architecture state (logical registers' data). 

  In addition, some hardware implementation of committing state (like *re-order buffer*) can work as a buffer to offer data to functional unit buffers by bypasses. In functional unit buffers, some instructions wait for data needed to become available. They can get these data from *ROB* instead of waiting until the data are updated.

---

The superscalar processor implements a sequential execution model ostensibly but executes instructions concurrently in fact. Both performance and compatibility have driven the development of superscalar processors. But the more parallel hardware resources are introduced, the fewer returns would be gained. So, there will be some new method to increase performance like VLIW and multiprocessing.

The second is most of the implementation of superscalar processors on the hardware level. Therefore, it can speed up existed programs. However, the software level can assist by creating optimized programs. There are two ways.

- Decreasing the likelihood that an instruction has to wait for the result of previous instruction when it could have been issued.
- Increasing the likelihood that a group of instructions can be issued simultaneously.

## Strengths

The first strength of superscalar processors is that it fully utilizes the most of hardware resources. Before superscalar processors occur, there just one ALU are working per time cycle, which is inefficient. Superscalar processors try to make all of ALUs work simultaneously, which can speed up executing instructions.

The second is there are many caches introduced, which could improve performance.

- *Instruction buffer* (speed up instruction fetching)
- *Translation lookaside buffer* and *Memory hierarchy* (speed up handling memory operations)

Three examples of the superscalar processor in this paper are representative. They both cover most of the hardware implementation mentioned in this paper and two different types of ISA.

- *MIPS R1000* is a RISA processor. 

  It implements a prediction table and resume buffer for branch prediction and implements *Multiple Queue Method* and *Re-order buffer* for *dynamic instruction scheduling*.

- *Alpha 21164* is also a RISA processor.

  It just implements a simple superscalar processor, which means it forgoes the advantages of dynamic instruction scheduling in favor of a high clock rate. It only implements the *Single Queue Method* without *OoO* execution.

- *AMD K5* is a CISA (x86) processor.

  It converts the original instruction byte to *RISA-like Operations* in the decoding stage and using instruction buffer for branch prediction. As for *dynamic instruction scheduling*, it implements two reservation stations for each functional unit except *FPU*.

## Weaknesses

The first is too complex hardware. Especially when superscalar processors introduce the Out of Order execution, there must be additional physical registers to be introduced. In addition, the time cost of dependency checking logic would increase and it needs to associate with functional units. So, the performance would be decreased, and it would consume more energy.

The second is the number of instructions that can issue simultaneously would be limited by the number of ALUs.

The third is the branch prediction is inefficient. If speculative execution goes a false branch, processors may fetch and execute instructions again.

## Thought

Authors care about almost of aspects to speed up efficiency in this paper. I am not confident that could do better now. 

However, I think there are room to improve performance on speculative execution. For example, using sophisticated methods to improve the accuracy or executing all of branches instructions and putting them into a temporary buffer. But the later method would increase hardware complexity and processors would execute some unnecessary instructions. So, I am not sure if it is practical.

Furthermore, most of strengths of superscalar processors would make hardware design become more complex, which is not "beautiful". But there may be a paradox between simplifying and performance enhancing.

## Takeaways and questions

I think this paper is nice and comprehensible. Moreover, it helped me sort out some of the key points like the relationship between OoO and superscalar. 

There is a question about VLIW. Authors mentioned that some schools of thoughts thought the VLIW is the future. It seems that VLIW was failed in CPUs markets. But I think the VLIW is very similar to the SIMD and the SIMD is very useful in deep learning areas. Is it possible to make VLIW revive in machine learning areas?

