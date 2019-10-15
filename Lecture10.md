# Task Execution

# Tasks

- units of work to be done
- size: tasks should be small and modular
- independence: tasks shouldn't interact with one another if possible

## Metrics

- throughput: how many tasks are being completed per unit time
- responsiveness: how long does it take for tasks to complete
- graceful degradation: how does system behave as it becomes overloaded

## Implementations

```java
public class SingleThreadWebServer{
    public static void main(String[] args) throws IOException{
        ServerSocket socket = new ServerSocket(80);
        while (true) {
            Socket connection = socket.accept();
            handleRequest(connection);
        }
    }
    // sequential, blocking execution of tasks
    // if handleRequest() crashes, whole server goes down
}
```

```java
public class ThreadPerTaskWebServer{
    public static void main(String[] args) throws IOException{
        ServerSocket socket = new ServerSocket(80);
        while (true) {
            final Socket connection = socket.accept();
            Runnable task = new Runnable() {
                public void run() {
                    handleRequest(connection);
                }
            };
            new Thread(task).start();
        }
        // one thread per task has too much overhead
    }
}
```

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