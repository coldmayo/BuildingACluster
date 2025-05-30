- Von Neumann architecture: A design with an undivided memory that stores both program and data, and a processing unit that executes the instructions, operating on the data in 'fetch, execute, store cycle'
	- Nowadays CPUs have multiple cores, which allows for things to execute in parrallel
	- Also, in recent years computers have multiple FPUs 
		- Some computers have Fused Multiple-Add (FMA) units which performs fused multiply-add operations such as $ax+b$ in the same amount of time as a separate add/multiply FPU
	- Out-of-order instruction execution allows CPUs to process instructions as resources become available rather than strictly adhering to the original sequence, improving efficiency while also increasing complexity in processor design.
- CPUs have cores, each core operates on its own stream of instructions simultaneously, significantly enhancing overall performance and computational throughput.
- Floating point add and multiply units are pipelined
	- Pipelining is a technique to improve performance by executing different phases of multiple instruction operations at once, allowing several operations to be processed concurrently within a single clock cycle.
- Understanding the trade-offs between memory access times and data transfer rates is crucial for optimizing scientific computations. (check out Chapter 1.3)
- Caches serve as a high-speed data storage area that retains frequently accessed data close to the CPU, enhancing access times. There are multiple cache levels (L1, L2, L3), each with unique speed and capacity characteristics, optimizing different aspects of performance.
- Memory interleaving across banks can significantly increase data transfer rates when accessed optimally; however, improper management can lead to conflicts that may degrade performance, necessitating strategic access patterns.
- Cache Misses:
	- Compulsory Miss: First time data access
	- Capacity Miss: Data is evicted due to insufficient cache space
	- Conflict Miss: Two Addresses map to the same cache location
	- Invalidation Miss: Cache data becomes outdated due to updates by other cores
- Cache Optimization Strats
	- Blocking: Process data in chunks that fit in cache
	- Prefetching: Load data before it's needed
	- Data locality: the process of moving computation to the node where that data resides
- Latency: Time taken to fetch a memory item
- Bandwidth: Amount of data transferred per second
- Memory stall: Occurs when CPU must wait for data from memory
- Prefetching:
	- Hardware prefetching: CPU predicts memory access patterns and loads data in advance.
	- Software prefetching: Programmers manually hint the CPU to load data early.
- Shared Cache Designs:
	- L1: Private per core
	- L2: Sometimes shared
	- L3: Shared between all cores
- Current challenges of multicore computing:
	- Cache coherency: Ensuring that all cores see the same data
	- Memory Bandwidth Limitations: Shared memory access slows performance.
	- Thread Synchronization: Managing execution order of parallel threads