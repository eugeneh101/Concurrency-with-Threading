# Coding Challenge: Using Threading Module for Concurrency

### Challenge: 
* GIVEN: a random number generator sampling numbers from a known distribution
* TASK: create a class where there exist multiple number generators (each providing a stream of numbers) while having a single writer thread storing the numbers to disk, sorted by datetime the number was generated.  

### Solution:
My code is in the Jupyter Notebook in [Concurrency_with_Threading.ipynb](https://github.com/eugeneh101/Concurrency-with-Threading/blob/master/Concurrency_with_Threading.ipynb).
I wrote my code in Python 3 using the **Threading** module. To model the arrival of numbers, I used the **exponential** distribution ($f(x) = \lambda e^{\lambda x} $) where I set rate=10 (which implies 10 arrivals/second). I believe this is a relatively high rate of arrival since that would be equivalent to 600 arrivals/minute. With number of generators set to 5, that would imply 3000 arrivals/minute. As a POC, I ran my number generators for only 4 seconds, so the expected value would be 10 \* 5 \* 4 = 200 arrivals whereas my empirical result  was 208 arrivals. That's pretty good!  
In addition, I used **inheritance** where appropriate to reduce repeating method definitions, except for instances where the methods are overridden.

### Runtime and Space Complexity
From a coding perspective, I used **deques** to for **O(1)** popleft operations and **O(1)** append operations. If the writer() method can write to disk faster than the threads of number generators yielding more numbers, then the effective space complexity will be **0(1)**, as the queues would be empty most of the time. The converse (where generators are yielding faster than the writer can handle) would result in the queues truly occupying **O(n)** space complexity and ultimately hitting a MemoryError.  
For the writer() method, it pops off the earliest number from the queues and loads it into a dictionary. The dictionary remains at **O(1)** space complexity with respect to the rate of arrivals and **O(n)** with respect to number of threads.


### Empirical Runtime
Through extensive profiling, I've determined that my multithreaded solution does not scale linearly when rate increases. A minor reason is that there is overhead with the thread acquision and release during context switching. Empirically, the major reason turns out to be pretty lame--time.sleep() for each of the exponentially-distributed wait time on Windows is really inaccurate and slow at the millisecond level due to Window's slower clock time. I conjecture with Linux's clock time, most of this inaccuracy would be alleviated and that scaling to higher arrival rates would retain high accuracy. Windoze...  

### High-Performance Scaling Proposal:
Suppose you want to massively scale (number of generators > 1000), then I propose an alternate solution. Each of the generators would write to disk immediately based on time windowing. For example, an example file name could be generator_id_4250_at_11_40pm_to_11_50pm.txt. Then, you can use the ipyparallel or multiprocessing module for true parallel computation to aggregate the 10-minute chunk from each generator and sort and save to disk. Each disjoint time window can be processed indepedently. If there exists a further need to scale horizontally, you can apply a traditional map-reduce paradigm such as AWS Elastic Map-Reduce reading from S3 bucket.  