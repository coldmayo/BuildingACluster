- Peak Performance of a processor refers to the maximum theoretical computation speed: Peak Performance$=$Clock Speed$\times$Vector Width$\times$Number of Floating Point Units$\times$Number of Cores
	- In reality, performance does not reach the theoretical peak
- Arithmetic Intensity: # of operations per word
	- If the number is too low we call the algorithm bandwidth bound, since performance is determined by memory bandwidth, rather than processor speed.
- Data reuse: Maximizing cache usage boosts performance
- Processors vs Memory Speed:
	- Processors are usually faster than memory can supply data
	- Performance Bottlenecking arises when data cannot be delivered to processors efficiently
- Denormal numbers (subnormals) slow computations as processors handle them via micro-code emulation
	- Solutions include enabling Flush-To-Zero (FTZ) mode to improve performance at the cost of accuracy
- Modern CPUs use pipelined FPUs
	- Bottlenecks occur in nonpipelinable operations, such as the inner product accumulation:
	- ```for (i=0; i<N; i++) {s+= a[i]*b[i];}```
		- The issue here is that it gets both read and written, which halts the addition pipeline
	- Loop unrolling is a good way to optimize code, it is a code optimization technique that reduces the number of times a loop is executed
		- Example: ```int x;for (x = 0; x < 100; x++){     delete(x);}``` becomes: ```int x;
		 for (x = 0; x < 100; x += 5) {
		     delete(x);
		     delete(x + 1);
		     delete(x + 2);
		     delete(x + 3);
		     delete(x + 4);
		 }```
- "on-die" cache is a cache that is physically embedded into the microprocessor, or part of the same silicon chip
- The L1, L2, and L3 caches have progressively lower bandwidth and higher latency
	- L1 is usually on die and the smallest (the fastest, closest to the CPU)
	- L2 is larger and slightly slower
	- L3 is the largest but the slowest 
	- L2 and L3 are still used since they have higher storage (time vs space tradeoff)
- Cache blocking: Process smaller blocks of data to fit in cache
- Timing Analysis: Test loop efficiency using cache-friendly access patterns. 
	- Example: Adjusting array size to fit L1 cache for faster reuse
- Direct-Mapped Cache: A scheme where each block of main memory maps to only one specific cache location
	- Direct-mapped caches can suffer collisions when repeatedly accessing specific data locations.
		- Deal with this by accessing data with varying displacement to observe time per access and assess cache associativity's impact.
- Loop order affects memory access patterns and performance
	- In C a user should optimize for column-major order (inner loop on row index)
	- In Fortran a user should optimize for row-major order
	- In OpenMP: Larger loops should be outermost for effective parallelization
- Loop Tiling: Dividing loops into blocks (tiles) to improve cache reuse
	- Improved performance in memory-bound problems like matrix operations
- Compilers cannot automatically achieve optimal performance and manual optimizations are very helpful
	- Tools like ATLAS and Spiral can optimize specific computational kernels
- Cache-Aware programming: Tailor algorithms to specific cache sizes for optimal performance
- Cache-Oblivious programming: Use divide-and-conquer approaches that naturally adapt to cache hierarchies