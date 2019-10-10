# Composing Objects

## Composition

- using objects inside other objects

## Thread-safe design

- identify ***state variables*** (fields) in your class
- identify ***specifications*** (invariants)
- identify ***synchronization policy*** to maintain specifications

## Java monitor pattern

- make all instance fields `private`
- make all method `synchronized`

## Instance confinement

- embed unsafe object inside a thread-safe class
- thread-safe class controls all access to object

```java
public class SyncIntegerHashSet{

    // invariant:  an integer is in the set if and only if it was added to the set

    // mySet is unsafe but SyncIntegerHashSet is thread-safe and controls
    // all accesses to mySet. thus, mySet has been properly confined
    private final Set<Integer> mySet= new HashSet<Integer>();

    public synchronized void addInteger(int i) {
        mySet.add(i);
    }

    public synchronized boolean containsInteger(int i) {
        return mySet.contains(i);
    }
}
```

- confined object shouldn't be published or allowed to escape

## Delegation

- delegating safety obligations to thread safe object field

```java
public class ZeroCounter{

    // Invariant:  count records the number of 0s processed since the most recent
    // reset, or since the object was created, provided count never exceeds
    // MAX_VALUE

    final private BoundedCounterThreadSafecount count = new BoundedCounterThreadSafe(Integer.MAX_VALUE);

    public void processInt(int i) {
        if (i == 0) count.inc();
    }

    public int getCount() {
        return count.current();
    }

    public void reset() {
        count.reset();
    }
}
```

- instance fields are not published
- instance fields must be thread-safe
- no invariant can constrain multiple instance fields
- no method can make more than one call to any method in an instance field

## State dependency

- method can only perform action if some state condition is met
- options
  - balking: method refuses to perform action and instead raises exception, returns an error code...
  - guarded suspension: lock has to be acquired and precondition satisfied to perform action
  - optimistic retrying: copy the current state, apply operation, "commit" updated state

```java
public class BoundedCounter{

    private int value = 0;
    private int upperBound= 0;

    // INVARIANT:  in all instances 0 <= value <= upperBound

    public synchronized boolean isMaxed() {
        return(value == upperBound);
    }

    // Pre: none
    // Post: increment value if not maxed. otherwise, do nothing.
    // Exception: none

    // here we balk by ignoring if isMaxed() returns true
    public synchronized void inc() {
        if (!isMaxed()) ++value;
    }
}
```

```java
void stateDependentMethod() {

    // example of guarded suspension
    synchronized(lock) {
        while(!conditionPredicate())
            lock.wait();
        // now we can proceed. we have the lock and we've satisfied the precondition
    }
}
```

```java
// state copying method (synchronized to get the right value)
public synchronized int current() { return value; }

// commit method (synchronized to ensure atomicity of commit)
public synchronized boolean commit(int oldState, int newState) {
    if (value == oldState) {
        value = newState;
        return true;
    }
    else return false;
}

public void inc() {
    for (;;) {
        // we are in a loop to ensure that we "optimistically" retry
        int currentState = current();
        if ((currentState < upperBound) && (commit(currentState, currentState + 1)))
            break;
        else
            // ensures that another thread runs before we try to increment again
            Thread.yield();
    }
}
```

## wait()/notify()/notifyAll()

- when thread calls `obj.wait()`, thread is added to wait set for that object and gives up lock
- thread calling `obj.notify()` randomly picks a thread from wait set to wake up and schedule
- thread calling `obj.notifyAll()` wakes up and schedules all threads from the object's wait set (preferred)

```java
// See the beautiful explanation at https://stackoverflow.com/a/3186336
public synchronized void put(Object o) {
    while (buf.size() == MAX_SIZE) {
        wait();
    }
    buf.add(o);
    notifyAll();
}

public synchronized Object get() {
    while (buf.size()==0) {
        wait();
    }
    Object o = buf.remove(0);
    notifyAll();
    return o;
}
```

## Nested monitor lockout

```java
public class BoundedBufferWaitNoNull{

    private final BoundedBufferWait buffer;

    BoundedBufferWaitNoNull(int capacity) {
        buffer = new BoundedBufferWait(capacity);
    }

    public synchronized boolean put(Object elt) throws InterruptedException {
        if (elt!= null) {
            buffer.put(elt);
            return true;
        }
        else return false;
    }

    public synchronized Object take() throws InterruptedException {
        return buffer.take();
    }

    // suppose thread t1 calls the BoundedBufferWaitNoNull.take() method when buffer is empty
    // buffer is empty so t1 waits on buffer and releases inner BoundedBufferWait lock
    // BoundedBufferWaitNoNull still holds its own intrinsic lock
    // OS schedules thread t2 and t2 calls BoundedBufferWaitNoNull.put(), causing deadlock
}
```
