# Parallelization

## Loops

- method #1: iterations must be independent
- method #2: chunk iterations into groups

```java
// method #1

// sequential
for (Element e : collection)
    process(e);
    
// parallelized
for (Element e : collection) {
    exec.execute (new Runnable() {
        public void() run {
            process(e);
        }
    });
}
```

```java
# method #2

// sequential
sum = 0;
for (int i=0; i < a.length; i++)
    sum += a[i];
    
// parallelized
int sum[] = new int[NUMTASKS];                          // ensure initialization to 0
int size = a.length / NUMTASKS;
for (int i=0; i < NUMTASKS; i++) {
    exec.execute (new Runnable() {
        public void run() {
            for (j = i*size; j < (i+1)*size; j++)
                sum[i] += a[j];
        }
    });
}
```

## Recursion

- tasks should be independent
- best if algo tail-recursive

```java
// sequential DFS
public<T> void sequentialRecursive(List<Node<T>> nodes, Collection<T> results) {
    for (Node<T> n : nodes) {
        results.add(n.compute());
        sequentialRecursive(n.getChildren(), results);
    }
}

// parallelized DFS
public <T> Collection<T> getParallelResults(List<Node<T>> nodes) throws InterruptedException {
    ExecutorService exec = Executors.newCachedThreadPool();
    Queue<T> resultQueue = new ConcurrentLinkedQueue<T>();
    parallelRecursive(exec, nodes, resultQueue);
    exec.shutdown();
    exec.awaitTermination(…);
    return resultQueue;
}

public <T> void parallelRecursive(final Executor exec, List<Node<T>> nodes, final Collection<T> results) {
    for (final Node<T> n : nodes) {
         exec.execute(new Runnable() {
            public void run() { 
                results.add(n.compute()); 
            }
        });
        parallelRecursive(exec, n.getChildren(), results);
    }
}
```

- must have a waiting mechanism if tasks are dependent
  - completion service: if you know what all the tasks are
  - shutdown executor: if you know when tasks will stop being submitted
  - maintain a count of unfinished tasks: need a counting mechanism like a latch

```java
// sequential quicksort
public static void quickSortSegment (int[] elts, int first, int size) {
    if (size == 2) {
        if (elts[first] > elts[first+1])
            swap (elts, first, first+1);
    } else if (size > 2) {
        int pivotPosition = partitionSegment(elts, first, size);
        quickSortSegment (elts, first, pivotPosition-first);
        quickSortSegment (elts, pivotPosition+1, first+size-1-pivotPosition);
    }
}

// parallelized quicksort
public void sort(int[] elts) {
    int NUMTHREADS = …;
    exec = Executors.newFixedThreadPool(NUMTHREADS);
    tasks = new BasicCountingLatch(1);
    exec.execute (new PQSTask (elts,0,elts.length));
    tasks.await();// Wait for tasks to finish.
    exec.shutdown();
}

private class PQSTask implements Runnable {
    private int elts[];
    private int first;
    private int size;
    public PQSTask (…) { … }
    public void run () {
        parallelQuickSortSegment (elts, first, size);
        tasks.countDown();
    }
}

public void parallelQuickSortSegment (int[] elts, int first, int size) {
    if (size == 2) {
        if (elts[first] > elts[first+1]) 
            IntArraySortUtils.swap (elts, first, first+1);
    } else if (size > 2) {
        int pivotPosition = IntArraySortUtils.partitionSegment(elts, first, size);
        
        // create new sorting tasks and increment task count
        PQSTask task1 = new PQSTask(elts, first, pivotPosition-first);
        PQSTask task2 = new PQSTask(elts, pivotPosition+1, …));
        tasks.countUp(2);
        
        exec.execute(task1);
        task2.run();
        // run second task in existing worker thread
}
```

## Tuning

- naive parallelized quicksort creates ~k/2 tasks for sorting k elements (way too many)
- N_threads = N_CPU * Util_CPU * (1 + Wait/Compute)
- N_threads = N_CPU + 1 (Util_CPU, Wait/Compute are very low for sorting, +1 to avoid deadlock)
- set the size of the sequential task limit to (k / N_threads) if k is the number of elements that we are sorting

```java
public void parallelQuickSortSegment (int[] elts, int first, int size) {
    // improving performance
    if (size <= THRESHOLD) {
        if (elts[first] > elts[first+1]) 
            IntArraySortUtils.quickSortSegment(elts, first, size);
    } else if (size > THRESHOLD) {...}
```

- mergesort is not tail-recursive so we must use an unbounded thread pool to avoid thread starvation deadlock
- thread starvation deadlock happens when every thread in (fixed size) thread pool blocks on other tasks
  - but more tasks cannot be executed since all threads in the thread pool are occupied

```java
    // no more deadlock issues, unbounded thread pool
    Executors.newCachedThreadPool();
```


