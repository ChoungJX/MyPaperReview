# Linfeng Zheng (2021-01-14)

# Paper information

- Title: InvisiSpec: Making Speculative Execution Invisible in the Cache Hierarchy
- Authors: Mengjia Yan, Jiho Choi, Dimitrios Skarlatos, Adam Morrison, Christopher Fletcher, Josep Torrellas
- Venue: MICRO 2018
- Keywords: hardware security, speculation, side channel, memory hierarchy

# Paper content

## summary

This paper proposes *InvisiSpec* to block microarchitecture covert and side channels due to *speculative execution* and *Out of Order execution*.

---

The cache hierarchy is a hardware design, which is used to accelerate data access from the main memory. This design makes most instructions including speculative instructions have to access the main memory through the cache. Therefore, even if *speculative execution* does not change the architectural state, the cache hierarchy state would be changed thus the side channels are created. This paper's **key idea** is preventing speculative instructions to change the cache hierarchy state until they cannot create side channels.

---

Key mechanisms:

- **Spec-GetS transactions** get copies of cache lines without changing the cache hierarchy state.

  In multicore processors, cores have both independent cache and shared cache. In order to guarantee that there is only a unique copy of a specific address in these caches, cache coherence transactions are introduced to maintain coherency. If a core wants to get a value that belongs to another core, the cache coherence state of the cache line of this value would be changed (e.g., in MESI protocol, this cache line's state will change to "S" from "E"). To avoid the cache coherence state changed due to accessing the cache by speculative Loads, *InvisiSpec* introduces a new transaction called *Spec-GetS*. *Spec-GetS* helps speculative Loads to requests a copy of a cache line from the owner that owns this cache line repeatedly until it gets the copy or speculative Loads are no longer speculative then switch to the standard transaction.
- **Unsafe/safe instructions & visible point**

  In this paper, *Load* instructions in speculative execution are called *Unsafe Speculative Loads* (USLs). *InvisiSpec* copies values required by USLs into a specific buffer called *Speculative Buffer* (SB) from the cache without changing the cache hierarchy state (make USLs invisible). After USLs become safe (they cannot create any side channel), *InvisiSpec* moves values from SB to cache hierarchy (make USLs visible). There are different *visible points* for USL in different attack models.
  - For *Spectre* attack model, USLs can become visible when their branch is resolved.
  - For *Futuristic* attack model, USLs can become visible when
    1. they reached the head of the ROB

    or 
    
    2. they did not reach the head of the ROB but their previous instructions cannot let them get squashed.

- **Validation or Exposure of a USL** makes a USL visible and maintains memory consistency.

  USLs cannot change the coherence state until they become safe. If a cache line loaded by a USL received an invalidation, only the value in cache hierarchy will be changed, in SB will not. If this USL using an older value in SB instead of the value updated in cache hierarchy after this cache line got an invalidation, it is equivalent to reordering some load instructions. However, this is disallowed in some memory consistency models. To avoid this violation, InvisiSpec will **validate** whether the value in SB is equal with the copy from the cache. If not equal, all successive instructions will be squashed.

  if there is not any Load in front of a USL in ROB, this USL will have no chance to violate memory consistency model. So, it does not need Validation. *InvisiSpec* will move its SB's value to the cache hierarchy without comparison, which is called **Exposure**.

  ---

  The Key conclusion:

  *InvisiSpec* can defend against *Spectre* attacks and *Futuristic* attacks with some performance decrease. However, its performance is faster than using fences.

## Strengths

1. InvisiSpec proposes two solutions to defend against the Spectre attack and Futuristic attack which have security and performance tradeoff. People can choose a suitable solution for different application backgrounds.
2. Some mechanisms in *InvisiSpec* have extra performance overhead, but it alleviates these slowdown using some other mechanisms:
   - **reusing the Speculative Buffer**

     For taking advantage of spatial locality, a USL loads a cache line into SB instead of a value. If the next USL needs the same cache line, it can wait for the first USL to finish the copy then get this cache line from the first USL's SB to avoid copying this cache line from the cache again.
   - **Overlapping Validations and Exposures**

     *Exposures* and *validations* will access the cache/main memory at least one more time than normal Load instructions. InvisiSpec tries to "visible" them as many as possible simultaneously to reduce overall time spent. (with some restrictions to grantee the correctness)
   - **Per-Core Last Level Cache Speculative Buffer (LLC-SB)**

     When USLs move cache lines needed into SB, cache lines may not resident in the cache. USLs must bring them from the main memory. To prevent these USLs access main memory again invalidations, InvisiSpec will copy these cache lines into LLC-SB to speed up main memory access.

## Weaknesses

1. Unnecessary operation

   *InvisiSpec* will allocate an LQ and SB entry for **every** issued load instruction. If a load is not speculative, its SB entry's state will be set to "N" (not necessary). But it is unnecessary to allocate an LQ and SB entry for every load. It is enough to only allocate them for USL so that it can reduce some operation and reduce the "state" hardware design thus reducing overheads.
2. Only blocking side channels through the cache

   *InvisiSpec* disallows USLs to change any cache state thus blocking adversarial code to reveal information from cache. However, some attack models do not create side channels through cache (e.g., some attacks reveal information based on branch predictor state), which means *InvisiSpec* cannot defend against these models well.

## Thoughts

I think *InvisiSpec* is a complete solution, but there is room for improvement in some mechanisms.

- Changing the logic in allocating LQ and SB entries.

  *InvisiSpec* will allocate an SB entry for **every** load instruction issued, then set 'N' flag in Status bit even a load instruction is not speculative. The better way is determining that if a load instruction is speculative first, then allocating LQ and SB entries for USLs.
- *LLC-SB* seems that not necessary.

  *LLC-SB* is used to speed up obtaining data when USLs in *Validation*. Why not add some bits in cache hierarchy instead of it? For example, adding one bit to announce that this cache line is used for *Validation* and prohibiting any instruction accessing it.

## Takeaways and questions

*InvisiSpec*'s key idea is straightforward: because the cache hierarchy creates side channels, there will be no problem after hiding the cache hierarchy state. However, there are many details to be worked out in implementation. *InvisiSpec* considered many aspects, like the memory consistency model, the cache coherence model, delay the interruption and TLB access, etc. Through this paper, I learned the feature of the speculative attack model and details about memory consistency.

**Question**:

- Why *Speculative Buffer* copies a cache line and sets an address mask, not store the value only?
- When a USL wants a cache line requested by other entries but does not reach SB, this USL will modify *MSHR*, which is complex. Why not wait for the value to reach SB then copy it?