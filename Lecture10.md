# Task Execution

# Tasks

- units of work to be done
- size: tasks should be small and modular
- independence: tasks shouldn't interact with one another if possible

## Metrics

- throughput: how many tasks are being completed per unit time
- responsiveness: how long does it take for tasks to complete
- graceful degradation: how does system behave as it becomes overloaded

## Executors

- thread pool of worker threads
- queue to hold waiting tasks
- producers generate tasks while consumers (worker threads) consume them
- execution policy defines how tasks get executed (e.g. directly, serially, etc...)

```java
public interface Executor {
    void execute(Runnable command);
}
```

```java
public class TaskExecutionWebServer{
    private static final intNTHREADS  = 100; // Fixed number of threads
    private static final Executor exec = 
        Executors.newFixedThreadPool(NTHREADS);
    public static void main(String[] args) throws IOException{
        ServerSocket socket = new ServerSocket(80);
        while (true) {
            final Socket connection = socket.accept();
            Runnable task = new Runnable() {
                public void run() {
                    handleRequest(connection);
                }
            };
            exec.execute(task);
        }
    }
    private static void handleRequest(Socket connection) { // request-handling logic here }
}
```

## Thread Pools

- `static ExecutorService newCachedThreadPool()`
  - creates as many threads as needed, but will reuse previously created threads
- `static ExecutorService newSingleThreadExecutor()`
  - single worker thread gets work from an unbounded queue
- `static ScheduledExecutorService newScheduledThreadPool(int corePoolSize)`
  - schedules commands to run after delay or periodically

## ExecutorService

- interface extending `Executor` that has functionality to shut down worker threads and deal with queued work

## Executor Lifecycle

- running: executor is running and can accept new tasks
- shutdown: executor isn't accepting new tasks, and may be finishing already accepted tasks
- terminated: executor has terminated all of the worker threads and is completely finished

## ScheduledExecutorService

- `ExecutorService` that can schedule commands to run after a fixed delay, or execute periodically
- `scheduleAtFixedRate`: schedules command to occur periodically
- `scheduleWithFixedDelay`: schedules command to occur periodically after a fixed delay

## Task Submission

- `void execute(Runnable command)`: executes the command at some time in the future
- `Future<T> submit(Callable<T> task)`: submits value-returning task and returns `Future` representing pending task
- `Future<?> submit(Runnable task)`: submits a non-value-returning task and returns `Future` representing pending task

## Task Status

- created, submitted, started, completed

## Callable vs. Runnable

- callable
  - cannot be fed into `Thread` constructor
  - can return a value
  - can throw checked exceptions (wrapped inside `ExecutionException`)
- runnable
  - can be fed into `Thread` constructor
  - cannot return a value
  - cannot throw checked exceptions

## CompletionService

- interface extending `ExecutorService` with a blocking completion queue
- when submitted task finishes, a `Future` for it is put in the completion queue
- user of the completion service can perform a `take()` on the completion queue

## Thread Starvation Deadlock

- suppose we have a fixed thread pool of size N where all N threads are running
- if each thread submits a task and blocks waiting on a result then deadlock
- no threads are left to handle new tasks on which N threads are blocking

## ThreadPoolExecutor

- base class whose constructor is called by `Executors.newXXXThreadPool()`
- `corePoolSize`: target number of threads in pool, even when there are no tasks
- `maximumPoolSize`: maximum number of threads can be active at any time
- `keepAliveTime`: thread alive for this long can be killed if number of active threads is greater than `corePoolSize`

## ThreadPoolExecutor Queues

- `singleThreadExecutor()`: `LinkedBlockingQueue` that is unbounded, blocks when empty, FIFO
- `newCachedThreadPool()`: `SynchronousQueue` that has capacity 0 (task handed to thread immediately)

## Saturation Policy

- determines course of action if work queue is bounded and full and new task comes in
- `AbortPolicy` (default): `execute()` throws `RejectedExecutionException`
- `DiscardPolicy`: `execute()` silently discards newest task
- `DiscardOldestPolicy`: `execute()` discards task at head of queue and tries to resubmit current task
- `CallerRunsPolicy`: `execute()` runs the current task immediately, giving time for threads to clear out work queue
