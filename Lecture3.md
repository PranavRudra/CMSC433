# Thread Safety

```java
public class Example extends Thread {

    private static int cnt = 0;         // shared state

    public void run() {
        int y = cnt;
        cnt = y + 1;
    }

    public static void main(String[] args) {
        Thread t1 = new Example();
        Thread t2 = new Example();
        t1.start();
        t2.start();
    }

    // Possible data race
    // Time 1: t1 reads cnt into y (y = 0)
    // Time 2: t2 reads cnt into y (y = 0)
    // Time 3: t1 increments y (y = 1)
    // Time 4: t2 increments y (y = 1)
    // Time 5: t1 writes to cnt (cnt = 1)
    // Time 6: t1 writes to cnt (cnt = 1)
}
```

## Atomicity

- operations A, B atomic if when executing one operation other has not occurred or completely occurred
- i.e. atomic operations will not have any thread interference since they cannot be interrupted

## Locking

- every `Object` in Java has an intrinsic lock
- intrinsic locks are reentrant
- `synchronized(obj){}`
  - acquires lock before statements in braces executed
  - releases lock after statements in braces executed

```java
public class Example extends Thread {

    private static int cnt = 0;         // shared state

    static Object lock = new Object();  // shared lock

    public void run() {
        synchronized(lock) {
            int y = cnt;
            cnt = y + 1;
        }
    }

    public static void main(String[] args) {
        Thread t1 = new Example();
        Thread t2 = new Example();
        t1.start();
        t2.start();
    }

    // No data race this time since each run operation is atomic
}
```

```java
public synchronized boolean isMaxed() {
    return (value == upperBound);
}

public synchronized void inc() {
    if (!isMaxed()) ++inc;
}

// Reentrant locks can be "re-acquired" many times by the same thread
// If lock wasn't reentrant every call to isMaxed() would block forever
```

## Synchronization

- using `static synchronized` locks on the class object
  - threads are unable to access all other `static synchronized` code blocks
  - HOWEVER, threads can still "unsafely" access blocks that aren't `static synchronized`
- using `synchronized` on instance methods only blocks other threads accessing `synchronized` sections on that instance

## Thread Safe vs. Thread Compatible

- thread safe: stateful object synchronizes itself with `synchronized`
- thread compatible: callers perform synchronization before calling

```java
// Thread safe since State class is internally synchronized
public class State {

    private int count = 0;

    public int synchronized incCount(int x) {
        count += x;
    }

    public int synchronized getCount() {
        return count;
    }
}

public class MyThread extends Thread {

    State s;

    public MyThread(State s) {
        this.s = s;
    }

    public void run() {
        s.incCount(1);
    }


    public void main(String args[]) {
        State s = new State();
        MyThread thread1 = new MyThread(s);
        MyThread thread2 = new MyThread(s);
        thread1.start();
        thread2.start();
    }
}
```

```java
public class Worker extends Thread {

    // Thread-compatible since synchronization occurs in callers/users of lst
    static List lst = new ArrayList();

    String s;

    void add(String s) {
        synchronized (lst) {
            lst.add(s);
        }
    }

    boolean check(String s) {
        synchronized (lst) {
            return lst.contains(s);
        }
    }

    // Race condition exists however
    // Time 1: t1 sees that "hello" doesn't exist in lst
    // Time 2: t2 sees that "hello" doesn't exist in lst
    // Time 3: t3 sees that "goodbye" doesn't exist in lst
    // Time 4: t1 adds "hello" to lst
    // Time 5: t2 adds "hello" to lst
    // Time 6: t3 adds "goodbye" to lst
    // Result is that lst has duplicate "hello" which should have been prevented
    // Issue can be fixed by making the whole compound operation synchronized
    public void run() {
        if (!check(s))
            add(s);
    }

    public void main(String args[]) {
        Set t1 = new Worker (“hello”);
        Set t2 = new Worker(“hello”);
        Set t3 = new Worker(“goodbye”);
        t1.start();
        t2.start();
        t3.start();
    }
}
```
