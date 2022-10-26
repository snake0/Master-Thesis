

----- 转发的邮件 -----
发件人: "guokaicheng" <guokaicheng@sjtu.edu.cn>
收件人: "guokaicheng" <guokaicheng@sjtu.edu.cn>
发送时间: 星期二, 2022年 1 月 11日 下午 11:58:35
主题: vee_review

VEE'22 Paper #76 Reviews and Comments
===========================================================================
Paper #76 RmemV: A Remote Memory System for Virtualization


Review #76A
===========================================================================

Overall merit
-------------
2. Weak reject

Reviewer expertise
------------------
3. Knowledgeable

Paper summary
-------------
With the unprecedented development of network technologies, remote-memory/disaggregated memory is drawing more and more attention from both academics and the industry. The remote memory technology can improve the resources utilization of the datacenter and provide available memory to applications running on other servers. This work proposed a swap-based remote memory management system, RmemV, for the applications running on virtual machines. The authors also propose some optimizations to the swap system, e.g., cold-page-eviction and hit-rate-based-prefetching.

Comments for author
-------------------
## Strength

Authors considered a realistic usage scenarios, virtual machine. 

The proposed remote memory management system, RmemV, is transparent to existing applications. 

## Weakness

The contributions of RmemV are not clear to me. Its system design and optimizations are very similar with existing works. 

# Details

1) I can not understand the contributions of RmemV. It seems that the authors ported the existing swap-based remote memory management solutions, e.g., fastswap, to the host kernel /supervisor. What are the unique challenges of designing a swap-based remote memory data path in the virtual machine scenario?
2) The proposed cold-page-eviction and hit-rate-based-prefetching are very similar to the kernel’s existing prefetching and swap-out algorithms. For example, the kernel has a prefetcher for swap. The kernel prefetcher utilizes the hit rate of prefetched pages to adjust its prefetching length. The kernel maintains active/inactive page lists to select cold pages for swap-out. At the same time, the kernel also has a daemon thread, kswapd, to swap out cold pages in the background.  Besides, lots of prefetching algorithms and eagerly-swap-out techniques are already proposed. For example, Leap is a prefetcher published in  USENIX ATC. Both Leap and Fastsawp have their own eagerly-swap-out mechanism.   Hence, I can not understand the unique contributions of RmemV. 
3) The design of the RPC module is confusing. It seems RmemV only relies on 2-sided RDMA to do some simple communications, e.g., init the RDMA QP, handle the complete RDMA requests. RmemV never offloads computing tasks to the remote memory servers. In this situation, what’s the purpose of designing a dedicated RPC module here? 
4) The argument of the disadvantage of scale-out solutions, e.g., distributed framework and distributed shared memory, is confusing. First, the programming models of map-reduced frameworks, .e.g., Spark and Hadoop, should be simpler than single machine programming models, e.g., Java and C/C++. Second, both scale-out and scale-up frameworks can run on the remote-memory /disaggregated-memory platforms. I cannot get the conclusion that scale-out solution is a bad choice for datacenter.



Review #76B
===========================================================================

Overall merit
-------------
1. Reject

Reviewer expertise
------------------
4. Expert

Paper summary
-------------
The paper proposes remote memory through virtualization and implements it in KVM and QEMU. The paper adds two optimizations, cold page eviction and remote page prefetching, to improve performance. Experimental evaluation shows good scalability of the proposed system for both sequential and random accesses. Application performance is also improved when compared to using local swap.

Comments for author
-------------------
Memory aggregation using RDMA has been broadly studied by the research community recently. Despite some discussions in the introduction section, I cannot identify key contributions of this paper. The authors need to clarify what is new in this work in their rebuttal.

The writing needs improvement. Many details are missing. 

Abstract/Intro: Key-based authentication is mentioned in abstract and intro and never explained later. 

Sec. 2.3: Can a VM use remote memory from multiple remote nodes? If so, how do you manage?

Sec. 2.3: Please add a leading paragraph explaining the four modules.

Sec. 2.3.1: What is “free chain”? Is it the same as the global free list in Figure 2?

Sec. 2.3.2: Are cold page detection and memory swapping related? How exactly is cold page detected? 

Sec. 2.3.3:  This whole section is confusing. Reading between the lines, I guess the system needs to allocate local pages and copy the remote pages to local pages. 

Sec. 3.2: The relationship between the global free list and the per-cpu cache is unclear.

Sec. 3.3: The paragraph about “30%” and “70%” utilization thresholds is hard to follow. In the sentence “When the memory utilization reaches 70%, the EPT violation process will send the memory swapping request”, how does the EPT violation process relate to a threshold? 

Sec. 3.4: define “memory slots”.

Sec. 3.4: Many details in Alg. 1 are missing. Hoes is checkTrend implemented? What is the initial value of npage_{curr}? What is the intuition behind the choices of window size and npage_{curr}?

Sec. 4.1.2: Figure 3 -> Figure 4

Figure 5: legend is missing.

Sec. 4.2: (25%) -> (50%)



Review #76C
===========================================================================

Overall merit
-------------
2. Weak reject

Reviewer expertise
------------------
4. Expert

Paper summary
-------------
The paper describes a remote memory system for VMs in the KVM/QEMU platform. VMs can access remote memory transparently by swapping to a remote memory pool. Implementation-wise QEMU is modified to perform swapping and prefetching to/from a remote memory server upon EPT violations. An RDMA-based RPC mechanism is implemented for page transfer. While remote memory has been studied extensively, including for VMs, the paper claims to be the first to implement this system on KVM/QEMU.

Comments for author
-------------------
I appreciate the systems development effort to make this prototype work. However the novelty of the work is quite thin.  Remote memory has been studied extensively. 

Remote memory pools for virtual machines has been investigated before.

MemX: Virtualization of Cluster-Wide Memory
https://ieeexplore.ieee.org/abstract/document/5599239

Disaggregated Cloud Memory with Elastic Block Management
https://ieeexplore.ieee.org/abstract/document/5558156

Swapping over RDMA has also been studied before.

Swapping to Remote Memory over InfiniBand: An Approach using a High Performance Network Block Device
https://ieeexplore.ieee.org/abstract/document/4154093

There are several other closely related works that the paper itself cites. The only novelty I see is that this system has been implemented in KVM/QEMU. 

The paper really needs a dedicated and systematic related work section, given the extensive prior work in this area.

The introduction mentions "cold page detection" and "dynamic prefetching" as contributions. The former not described in any depth. I had to guess from the context that either the hypervisor or QEMU looks at the Accessed bit in EPT to decide whether or not to to swap out a page. 

Prefetching is a straightforward dynamic window-based algorithm, similar to MemX above. Algorithm 1 is presented without much description. For instance, what does checkTrend() do and how? What constitutes an access pattern? 

The description of RPC over RDMA, which I was hoping would be a key novelty, was disappointingly brief and generic. There are no evaluations to show much does using RDMA contribute to performance versus not using it.

Figures 3 & 4 : Please include units in Y-axis

Figure 5: What do the colors in bars represent?

Algorithm 1 title: Dynamci --> Dynamic



Review #76D
===========================================================================

Overall merit
-------------
2. Weak reject

Reviewer expertise
------------------
4. Expert

Paper summary
-------------
The paper describes how KVM/qemu can be modified to transparently support
swapping of VM memory onto remote memory through RDMA. Communication
protocol, basic reclamation policy and prefetching are implemented as
an optimization.

Comments for author
-------------------
Thanks for your submission.

While there is nothing fundamental wrong in this work, I think that this
work raises more questions than it answers. Bit portions of this work
``reinvent the wheel'' without justification or comparison with existing
alternatives.

But before we get to these parts, let's discuss the motivation. The
transparency that the proposed solution provides can come with considerable
performance cost due to the semantic gap. The hypervisor, for instance, might
writeback to the remote memory data that is no longer needed by the VM, or fault in
a free-page that the VM is about to rewrite. Double-paging (aka double-swapping)
is another possible problem of such a system. a discussion of the
drawbacks relatively to the non-transparent alternative is needed, if not
evaluation. Personally, I am not convinced that in most cases the benefits
of the transparent solution outweigh its shortcomings.

Going back to my main criticism of this work---I do not understand the
benefit of this work over a hypervisor in which the host uses some academic
native remote-memory swapping solution (e.g, infiniswap) or existing one
(e.g., host swap on NBD). What in this work is any different just because
the swapped out memory is memory that is used by the VM?

To be more concrete to Linux/KVM, VM memory can already be swapped out by the
host and one can use existing/academic block-device/frontswap to swap it
to remote nodes. Changes to access-bit in the EPT are already propagated
back to the host (through kvm_mmu_notifier_test_young()) so KVM does not
have to be modified. Linux reclamation policy and prefetch mechanisms are
being used for VM memory when it is swapped out---just as
they are used for the host memory.

Now, it might be that in your implementation you short-circuit
the swap mechanism and get better performance. Yet, without a proper
explanation, this would just be a hack, as it might break abstraction
layers.

In this regard, the paper makes implicitly a contradicting
argument: first you circumvent the exiting kernel reclamation/prefetch
mechanisms, and this circumvention, as much as I understand, is the motivation
not to use the alternative existing aforementioned solutions that go
through the existing memory management mechanisms. Yet then you add 
reclamation/prefetch policy, which is very very similar to the existing
kernel policies. Nothing in this policies is virtualization-oriented,
as far as I understand.

As I mentioned before, I do not think that even if the performance of
the solution that this paper presents is better than using existing
alternatives on the host, this is by itself a justification for the
proposed design. Yet, to make matters worse, the paper does not even
present the performance of any alternative. 


Nits/minor:

* Please put a space before the citatiton.

* Many bibliography entries are missing a key and therefore show first the year.

* I am not sure how the "failure of Moore's Law" applies here. In addition, so far node sizes do get smaller.

* The goal of many paragraphs is only revealed at the end of the paragraph (e.g., the paragraph that starts with "The MapReduce-based"). Try to state the purpose of the paragraph in the first 2 sentences.

* "Dynamci"

* Please include a legend in Figure 5



Review #76E
===========================================================================

Overall merit
-------------
1. Reject

Reviewer expertise
------------------
4. Expert

Paper summary
-------------
The paper presents RmemV, a system that uses RDMA to use memory on remote nodes
as secondary storage for VMs. This use of "remote memory as swap" is implemented
in KVM, with the express goals of (1) giving VMs the illusion that a lot of
physical memory is available, and (2) doing so in a way that judiciously handles
what pages are available.

Comments for author
-------------------
After reading this paper, I find it has many critical problems in both the work
and its presentation (shown in no particular order):

* You do not properly motivate why a KVM-based solution is needed, when you
  could simply use existing solutions on either the guest or host system (e.g.,
  Infiniswap).

* You do not properly explain why you cannot simply use/extend the host Linux
  (kswapd, etc) to use remote memory for swapping. Or more precisely, you do not
  motivate why your eviction and prefetching policies are better or even
  sufficient compared to the ones already present in Linux.

* The design and implementation are severely lacking in detail and reasoning to
  back your choices.

* The evaluation lacks a comparison against Infiniswap (e.g., running on the
  guest VM), and existing Linux using a local tmpfs as a swapping device with an
  added delay to roughly emulate the network (seen in the remote memory
  literature). The former is a necessary piece of related work. The latter is a
  necessary sanity check of why your approach is needed.