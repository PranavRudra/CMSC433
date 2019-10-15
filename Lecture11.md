
# Fork/Join Parallelism

## Overview

- used to parallelize divide-and-conquer algorithms (non-overlapping repeated subproblems)

## Architecture
- `ForkJoinPool`: executor class that implements `ExecutorService` and manages the thread pool
- `ForkJoinTask<V>`: task class that implements the `Future<V>` interface and is more lightweight than a thread
  - `RecursiveTask`: subclass that returns a value
  - `RecursiveAction`: subclass that doesn't return a value

## ForkJoinPool

- default size of thread pool is equal to number of CPU's available
- **work-stealing** is used to keep works busy
  - each worker thread has its own work dequeue
  - when a worker's queue is empty it takes work from another's
- work is pushed to the front of the worker deque and popped to complete (to mimic recursion)
- work-stealing then happens from the back of the worker dequeue (i.e. the oldest work is stolen)

## Methods

- `ForkJoinPool`
  - `V invoke(ForkJoinTask<V> task)`: performs the given task and returns its result on completion
- `ForkJoinTask<V>`
  - `V invoke()`: invokes the task and returns its return value
  - `ForkJoinTask<V> fork()`: submits task to `ForkJoinPool`
  - `V join()`: has the effect of `get()` on a `Future<V>` but doesn't block

```java
public class ForkJoinSum extends RecursiveTask<Long>{
	private Integer[] buffer; 
	private int start;
	private int end;
	public ForkJoinSum(Integer[] buffer, int s, int e) {
		this.buffer = buffer;
		this.start = s;
		this.end = e;
	}
	@Override
	protected Long compute() {
		long sum = 0;
		if (end - start <= 8) {
			for(int i = start; i < end; i++) {
				sum += buffer[i];
			}
			return sum;
		} else {
			int m = (start + end) /  2;
			ForkJoinSum left = new ForkJoinSum(buffer, start, m);
			ForkJoinSum right = new ForkJoinSum(buffer, m, end);
			
			left.fork();
			long t = right.compute();
			long s =  t +  left.join();
			
			return s;
		}
	}	
}
public class Driver {
	public static void main(String[] args) {
		Integer[] buffer = new Integer[16];
		for(int i = 0; i < 16; i++) {
			buffer[i] = i + 1;
		}
		long sum = 0;
		ForkJoinPool forkJoinPool = new ForkJoinPool(2);
		sum = forkJoinPool.invoke(new ForkJoinSum(buffer, 0, buffer.length));
	}
}
```