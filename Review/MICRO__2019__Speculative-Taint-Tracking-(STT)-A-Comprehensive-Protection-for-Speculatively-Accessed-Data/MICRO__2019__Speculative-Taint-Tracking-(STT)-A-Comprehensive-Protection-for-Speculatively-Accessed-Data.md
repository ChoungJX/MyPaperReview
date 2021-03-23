# Linfeng Zheng (2021-03-21)

- Title: Speculative Taint Tracking (STT): A Comprehensive Protection for Speculatively Accessed Data
- Authors: Jiyong Yu, Mengjia Yan, Artem Khyzha, Adam Morrison, Josep Torrellas, Christopher W. Fletcher
- Venue: MICRO 2019
- Keywords: speculative instructions, speculative execution attacks, malicious speculation, side channel attacks, hardware mechanism, high performance

# Paper content

## Summary

This paper proposes *Speculative Taint Tracking* (STT) to block covert channels in speculative execution.

---

There are two key ideas in this paper:

1. According to different mechanisms due to leakage, this paper divides covert channels into five types.
2. STT allows processors to execute instructions that are able to read the **secret** that they could have not read but delay executing instructions that can reveal the **secret** from previous instructions.

---

Key mechanisms:

- **unsafe/safe access instructions, transmit instructions, and visible point**

  This paper figures out the general feature of leakage in speculative execution that there are instructions called *access instruction* are able to read **secret** into registers then there are instructions called *transmit instruction* reveal the **secret** by creating operand-dependent resources related to *access instructions*.

  Visible point is the point that we can know whether *access instructions* will be actually taken. Before *access instructions reach the visible point*, it is unsafe, otherwise, it is *safe*.
- **Taint and untaint propagation**

  STT taints the output register of unsafe *access instructions* and taints registers of sequent instructions that depend on *access instructions'* tainted register. It is ok to execute instructions that depend on tainted register excepted *transmit instructions* because *transmit instructions* can reveal taint registers' values that could be **secrets**.
- **Tracking Youngest Root of Taint** (YRoT)

  In STT, instructions are labeled with monotonically increasing numbers to present their position in Reorder Buffer (ROB). The later an instruction reached ROB, the bigger the index of it is (the younger it is). Every register in rename table has a new field called **Youngest Root of Taint** (YRoT). If an instruction's registers depend on an *access instruction's* output register, its registers' YRoT would record the index of this *access instruction* in ROB. When an *access instruction* reaches the visible point, STT will broadcast its index to sequent instructions in reservation stations, then these instructions whose registers' YRoTs are smaller or equivalent this *access instruction*'s index can become visible together.

---

Key conclusions:

STT can block leakage from both explicit and implicit covert channels in modern speculative out-of-order processors with better performance than other solutions.

## Strengths

1. This paper finds out new types of leakage in implicit channels and gives their definitions. Moreover, SST can block leakage from all of these implicit channels.
2. This paper only delays executing instructions (*transmit instructions*) that have a risk to reveal the **secret**. Compared with other solutions which prevent processor executing all of speculative instructions, STT has an extremely better performance.
3. Tracking Youngest Root of Taint is simple but is efficient enough. It only tracks *access instructions* that reached visible point, then all sequent instructions that depend on its or retired *access instructions*' output registers can become visible together. Moreover, it can make multiple *access instructions* visible in the same cycle: just broadcast the maximum YRoT of them to ROB.

## Weaknesses

1. Too much hardware needs to modify for SST.
   * The rename table needs to insert two fields (YRoT and AccessInstrIdx) whose size depends on the size of ROB.
   * The branch units, Load-Store units, and all data-dependent arithmetic and logic units (ALUs), etc. Their reservation stations need to insert a new field (YRoT) and some logic needs to modify to judge whether an instruction can be executed.
2. STT is not "universal". STT needs to modify some ALUs and optimize for some hardware optimization mechanisms. But every microarchitecture of processors has its specific implementation and optimization. So, Implementations of STT in different microarchitectures are different and the effect of performance may be different.

## Thoughts

Inserting a new field (YRoT) into ALUs' reservation stations whose instructions need to be tracked would create some unnecessary space. It can get the YRoT directly from rename table when judging if instructions can be issued, which can save space.

## Takeaways and questions

Taint and Untaint propagation mechanism is very brilliant. It can only delay executing instructions that may reveal the **secret**. Moreover, Tracking the Youngest Root of Taint is fast. Its design is simple but can solve problems well. The abstractions of new types of implicit channels are novel. According to the paper, I learned about some attacks that exploit implicit channel leakage and some hardware optimization mechanisms.

**Questions**:

1. What will happen if the YRoT achieved the maximum? e.g. When a YRoT achieved the maximum but its *access instruction* has not reached visible point, then there is a new unsafe *access instruction*'s register need to assign a new YRoT, what number should be assigned to this new YRoT. Should STT broadcast to rename table and reservation station to change YRoT for every register? This paper does not mention these details.
2. About the field *AccessInstrIdx* in rename table. Its size is equal to YRoT (log2(ROB size) bits). I guess this setting is preventing the ROB size exceeds the maximum value of the index. But if a register is not from an *access instruction*, its *AccessInstrId* is -1. If *AccessInstrIdx* needs to present minus, I think its size should be log2(ROB size)+1 bits. I did not get these details from this paper, so I leave this as a question.