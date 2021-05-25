# Linfeng Zheng (2021-03-21)

# Paper information
- Title: Speculative Taint Tracking (STT): A Comprehensive Protection for Speculatively Accessed Data
- Authors: Jiyong Yu, Mengjia Yan, Artem Khyzha, Adam Morrison, Josep Torrellas, Christopher W. Fletcher
- Venue: MICRO 2019
- Keywords: speculative instructions, speculative execution attacks, malicious speculation, side channel attacks, hardware mechanism, high performance

# Paper content
## Summary
### Trying to solve
This paper proposes _Speculative Taint Tracking_ (STT) to block covert channels in speculative execution.

### Key ideas
1. According to different mechanisms due to leakage, this paper divides covert channels into five types.
2. STT allows processors to execute instructions that are able to read the __secret__ that they could not have read but delays executing instructions that can reveal the __secret__ from previous instructions.

### Key mechanisms
- __Explicit/implicit channel__
  This paper defines two categories of leakage in speculative execution.
  1. _Explicit_ channel: instructions _directly_ access and reveal the __secret__.
  2. _implicit_ channel: executing instructions changes some resources and these changes _indirectly_ cause the __secret__ to be revealed.
  
- __Unsafe/safe access instructions, transmit instructions, and visible point__
  This paper figures out the general feature of leakage in speculative execution. That is, _access instructions_ are able to read the __secret__ into registers. After that, _transmit instructions_ can reveal the __secret__ by creating operand-dependent resources related to _access instructions_.
  
  _Visible point_ is the point that we can know if _access instructions_ will be actually executed. Before _access instructions_ reach the _visible point_, it is _unsafe_, otherwise, it is _safe_.
  
- __Taint and untaint operation__
  In STT, instructions are labeled with monotonically increasing numbers to present their position in the Reorder Buffer (ROB). The later instructions arrive at the ROB, the larger indexes they have. Every register in the rename table has a new field called __Youngest Root of Taint__ (YRoT) to store indexes for the _taint_ and _untaint_ operation. 
  
  When STT does the _taint_ operation, it stores indexes into their corresponding _unsafe access instructions'_ output registers' YRoT. Sequent instructions whose input registers are _tainted_ (which means their input registers are these _unsafe access instructions'_ output registers), their output registers will also be _tainted_. _Transmit instructions_ whose input registers are _tainted_ will be delayed executing (because they can reveal the __secret__).
  
  When STT does the _untaint_ operation, it broadcasts indexes of _access instructions_ that reach the _visible point_. Those delayed _transmit instructions_ can be executed if indexes in their input registers' YRoT are smaller or equivalent to the maximum index of these access instructions.

### Key conclusions
STT can block leakage from all covert channels in modern speculative out-of-order processors with better performance than other solutions.

## Strengths
1. This paper finds out new types of leakage in _implicit_ channels and gives their definitions. Moreover, STT can block leakage from all of these _implicit_ channels.
2. This paper only delays executing _transmit instructions_ that have risks to reveal the __secret__. Compared with other solutions that preventing processors from executing all speculative instructions, STT has an extremely better performance.
3. Tracking Youngest Root of Taint is simple. It only tracks _unsafe access instructions_ that reach the _visible point_. Moreover, it is also efficient. It can make multiple _access instructions_ be safe in the same cycle: broadcasting the maximum YRoT of them to reservation stations.

## Weaknesses
1. Too much hardware needs to be modified for SST's strategy.
   - The rename table needs to insert two fields (YRoT and AccessInstrIdx) whose size depends on the size of the ROB.
   - The branch units, Load-Store units, and all data-dependent arithmetic and logic units (ALUs), etc. Their reservation stations need to insert a new field (YRoT) and some logic needs to modify to judge whether an instruction can be executed.
2. STT is not "universal". STT needs to modify some ALUs and optimize for some hardware optimization mechanisms. But different microarchitectures of processors have their specific implementations and optimizations. So, implementations of STT in different microarchitectures are different and the effect of performance may be different.

## Thoughts
Inserting a new field (YRoT) into ALUs' reservation stations whose instructions need to be tracked would create some unnecessary space. It can get the YRoT directly from the rename table when judging if instructions can be issued, which can save space.

## Takeaways and questions
Taint and Untaint propagation mechanism is very brilliant. It only delays executing instructions that may reveal the __secret__ and it only tracks _unsafe access instructions_ in the _visible point_, which makes _taint_ and _untaint_ operation can do efficiently. Moreover, the abstractions of new types of implicit channels are elaborate. According to this paper, I learned about some attacks that exploit implicit channel leakage and some hardware optimization mechanisms.

### Questions
1. What will happen if the YRoT achieved the maximum? e.g. When a YRoT achieves the maximum but its _access instruction_ does not reach the _visible point_. At the same time, there is a new _unsafe access instruction's_ register need to assign a new YRoT. What number should be assigned to this new YRoT? Should STT broadcast to the rename table and reservation station to change YRoT for every register? This paper does not mention these details.
2. About the field _AccessInstrIdx_ in the rename table. Its size is equal to YRoT (log2(ROB size) bits). I guess this setting is preventing the ROB size from exceeding the maximum value of the index. But if a register does not belong to an _access instruction_, its _AccessInstrId_ is -1. If _AccessInstrIdx_ needs to present minus, I think its size should be log2(ROB size)+1 bits. I did not get these details from this paper, so I leave this as a question.