# Synchronizers

## Synchronizer

- object that coordinates control flow of threads based on its state

## ReentrantLock

- `newCondition()` instead of `wait()/notify()/notifyAll()`
- waiting on `Condition` done with `await()`
- awaken with `signal()/signalAll()`
- must explicitly lock and unlock

```java
public class ArrayBoundedBuffer{
    
    private final ArrayList<Object> items;
    private final int capacity;
    private final Lock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();  // waiting for not full
    private final Condition notEmpty = lock.newCondition(); // waiting for not empty

    public void put(Object x) throws InterruptedException{
        lock.lock();
        try {
            while (items.size() == capacity)
                notFull.await();
            items.add(x); 
            notEmpty.signal();
        } finally { lock.unlock(); }
    }
    
    public Object take(Object x) throws InterruptedException{
        lock.lock();
        try {
            while (items.size() == 0)
                notEmpty.await();
            Object obj = items.get(0); 
            notFull.signal();
            return obj;
        } finally { lock.unlock(); }
    }
}
```

## Latches

- delay starting of threads until an initial condition is satisfied

```java
public class LatchExample{

    public static void main(String[] args) {
    
        final int numThreads= 25;
        final int numIterations= 1000000;
        final CountDownLatch startGate = new CountDownLatch(1);
        final CountDownLatch endGate = new CountDownLatch(numThreads);
        
        for (int i= 0; i < numThreads; i++) {
            Thread t = new Thread() {
                public void run () {
                    try { 
                        startGate.await();
                    } catch (InterruptedException e) {}
                    // empty work - do nothing
                    for (int j = 0; j < numIterations; j++) {}
                    System.out.println("Thread " + getName() + " finishes.");
                    endGate.countDown();
                }
            };
            t.start();
        }
        startGate.countDown();  // all of the threads will start now
        try { 
            endGate.await();    // performs a multiway join on all threads
        } catch (InterruptedException e) {}
}
```

## FutureTask

- enable asynchronous work

```java
public class FutureTaskTest{

    public static void main(String[] args) {
    
        Callable<String> c = new Callable<String>() {
            public String call() {
                return "Foo";
            }
        };
        
        FutureTask<String> future = new FutureTask<String>(c);
        
        new Thread(future).start();     // “invokes” future
        
        /* can do something else here */
        
        try { 
            // blocks until future.get() returns
            System.out.println(future.get());
        } catch (InterruptedException e) {}
          catch (ExecutionException e) {}
          finally { System.out.println("Done"); }
    }
}
```

## Counting Semaphores

- act like bounded counters
  - initially given a positive value
  - each `acquire()` decrements value
  - each `release()` increments value
  - semaphore blocks on `acquire()` when value is `0`

```java
public class BoundedHashSet<T> {

    private final Set<T> set;
    private final Semaphore spaceAvailable;
    
    public BoundedHashSet(int capacity) {
        this.set = Collections.synchronizedSet(new HashSet<T>());
        spaceAvailable = new Semaphore(capacity);
    }
    
    public boolean add(T o) throws InterruptedException{
        spaceAvailable.acquire();
        boolean wasAdded = false;
        try {
            wasAdded = set.add(o);
            return(wasAdded);
        } finally { 
            if (!wasAdded) 
                spaceAvailable.release();
        }
    }
    
    public boolean remove(T o) {
        boolean wasRemoved = set.remove(o);
        if (wasRemoved) 
            spaceAvailable.release();
        return wasRemoved;
    }
}
```
