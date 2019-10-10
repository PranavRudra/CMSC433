# Introduction to Concurrency

## Clarifications

- concurrency: number of control flows is unrelated to number of processors
- parallelism: number of control flows less than or equal to number of processors

## Processes vs. Threads

- processes possess their own virtual address space and communicate via IPC
- threads are contained in one process, share heap, but don't share stack
  - operating system swaps out threads via context switching

## Instantiation

- valid constructors are `Thread()`, `Thread(Runnable r, String name)`, `Thread(Runnable r)`
- pass any data needed in the thread's `run()` method into the constructor of the class

```java
// Method #1: Extending the Thread class
public class MyThread extends Thread {
    @Override
    public void run() {

    }
}

Thread myThread = new MyThread();
myThread.start();
// NOT myThread.run(), otherwise you get single threaded behavior
```

```java
// Method #2: Implementing the Runnable interface
public class Worker implements Runnable {
    public void run() {

    }
}

Runnable worker = new Worker();
new Thread(worker).start();
```

```java
// Method #3: Anonymous class implementing Runnable interface
Thread thread = new Thread(new Runnable(){
    public void run() {

    }
});

thread.start();
```

## States

- NEW: thread has just been instantiated but not started
- TERMINATED: thread has completely exited
- RUNNABLE: thread has been started and is executing
- BLOCKED: thread is blocked waiting for a monitor lock
- WAITING: thread is waiting indefinitely for another thread to do some work
- TIMED_WAITING: thread is waiting for another thread to do some work, up to a specified time

## State Methods

```java
thread.start();     // Can ONLY call when thread is in the NEW state (i.e. only once)

thread.isAlive();   // Similar to (thread.getState() != NEW) && (thread.getState() != TERMINATED)

thread.join();      // Similar to while (thread.getState() != TERMINATED);
                    // Blocks until thread it's called on is TERMINATED
```

## User Threads vs. Daemon Threads

- all threads are user threads by default
- JVM exits when all user threads exit
- daemon threads should be used for background work
- user thread can only be made a daemon thread via `setDaemon()` when its state is NEW

## Exceptions

- InterruptedException: raised when a thread is interrupted while sleeping
