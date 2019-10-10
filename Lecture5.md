# Synchronization Deadlock

## Deadlock

- two threads block forever because they each hold a lock that is required by the other thread to continue
- set of threads deadlocked if each is waiting on an event that only another thread in the set can cause

```java
Object A = new Object();
Object B = new Object();

// Thread #1
synchronized(A) {
    synchronized(B) {

    }
}

// Thread #2
synchronized(B) {
    synchronized(A) {

    }
}

// If T1 finishes before T2 we have no problem
// If T2 finishes before T1 we have no problem
// Deadlock scenario:
// Time 1: T1 grabs lock A
// Time 2: T2 grabs lock B
// Time 3: T1 blocks for lock B
// Time 4: T2 blocks for lock A
```

## Open Call

- calling a method without holding a lock yourself

## Wait Graphs

- deadlock occurs when there is a cycle in a wait graph
- locks are squares
- threads are circles
- -> indicates thread wants that lock, <- means thread has lock

## Livelock

```java
// Thread #1
while (counter.get() < 10) {
    counter.inc();
}

// Thread #2
while (counter.inc() > 0) {
    counter.dec();
}

// If execution alternates between T1 and T2, counter will stay the same forever
// This is livelock because T1 and T2 aren't blocked, but no progress is being made
```

## Starvation

- process is continually denied resources (e.g. runnable process indefinitely overlooked by scheduler)
